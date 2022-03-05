---
title: cmu 15445 查询处理器实现
date: 2022-02-13 19:41:58
tags: ["cmu-15445"]
draft: true
---

## 项目说明

实现bustub中query execution的部分，阅读代码，完成execution目录中所有的`*_executor.cpp`文件。

## 思路

project3的难点在于理解bustub中的数据存储以及查询处理器的模型，需要认真阅读多个模块的代码。

### 存储模型

物理模型从小到大分别由类Value、Tuple、TablePage、TableHeap存储。

逻辑模型由从小到大Column、Schema存储。

### 查询处理器模型

查询处理一般包含如下步骤：

* 语法分析
* 初始逻辑查询计划生成
* 查询重写
* 物理查询计划生成
* 物理查询计划评价
* 执行物理查询计划

bustub代码中未添加sql语句语法解析的部分，逻辑查询计划在测试文件中手动编写生成，并且实验中只包含了查询计划生成以及执行查询计划的部分，完成该部分的代码即可。

## 实现

### seq scan

注意以下几点：

* `select ... from ... where condition`中的condtion以comparison expression的形式存储在plan_的数据成员中，在进行线性查找时需要进行判断。

```cpp
SeqScanExecutor::SeqScanExecutor(ExecutorContext *exec_ctx, const SeqScanPlanNode *plan)
    : AbstractExecutor(exec_ctx),
      plan_(plan),
      table_heap_(exec_ctx->GetCatalog()->GetTable(plan_->GetTableOid())->table_.get()),
      iter_(table_heap_->Begin(exec_ctx->GetTransaction())) {}

void SeqScanExecutor::Init() {
  iter_ = table_heap_->Begin(exec_ctx_->GetTransaction());
}

bool SeqScanExecutor::Next(Tuple *tuple, RID *rid) {
  if (iter_ == table_heap_->End()) {
    return false;
  }
  // get the rid and target columns
  std::vector<Value> vals;
  const Schema *output_schema = plan_->OutputSchema();
  Schema table_schema = exec_ctx_->GetCatalog()->GetTable(plan_->GetTableOid())->schema_;
  uint32_t col_cnt = output_schema->GetColumnCount();
  vals.reserve(col_cnt);
  for (uint32_t i = 0; i < col_cnt; i++) {
    const AbstractExpression *expr = output_schema->GetColumn(i).GetExpr();
    vals.push_back(expr->Evaluate(&(*iter_), &table_schema));
  }
  Tuple temp_tuple(vals, output_schema);
  RID temp_rid = iter_->GetRid();
  ++iter_;
  //
  const AbstractExpression *predicate = plan_->GetPredicate();
  if (predicate == nullptr || predicate->Evaluate(&temp_tuple, output_schema).GetAs<bool>()) {
    *tuple = temp_tuple;
    *rid = temp_rid;
    return true;
  }
  return Next(tuple, rid);
}
```

### insert

注意如下几点：

* 插入分为两种情况：直接指定插入值即raw values以及插入子查询subquery的结果，可以通过`plan_->IsRawInsert()`判断是否是直接插入值。
* 在插入时同时要向索引中插入记录。
* 添加数据成员table_info_、table_heap_方便代码的编写。

```cpp
InsertExecutor::InsertExecutor(ExecutorContext *exec_ctx, const InsertPlanNode *plan,
                               std::unique_ptr<AbstractExecutor> &&child_executor)
    : AbstractExecutor(exec_ctx),
      plan_(plan),
      child_executor_(std::move(child_executor)),
      table_info_(exec_ctx->GetCatalog()->GetTable(plan->TableOid())),
      table_heap_(table_info_->table_.get()) {}

void InsertExecutor::Init() {}

bool InsertExecutor::Next([[maybe_unused]] Tuple *tuple, RID *rid) {
  // if there is no child plans, execute it directly
  if (plan_->IsRawInsert()) {
    for (auto &vals : plan_->RawValues()) {
      InsertIntoTableIndexes(Tuple(vals, &(table_info_->schema_)));
    }
    return false;
  }
  // else first execute child plans, then insert child result set to table
  child_executor_->Init();
  std::vector<Tuple> child_tuples;
  Tuple child_tuple;
  RID child_rid;
  while (child_executor_->Next(&child_tuple, &child_rid)) {
    child_tuples.push_back(child_tuple);
  }

  for (auto &child_tuple : child_tuples) {
    InsertIntoTableIndexes(child_tuple);
  }
  return false;
}

void InsertExecutor::InsertIntoTableIndexes(const Tuple &cur_tuple) {
  RID rid;
  // insert tuple
  if (!table_heap_->InsertTuple(cur_tuple, &rid, exec_ctx_->GetTransaction())) {
    throw Exception(ExceptionType::OUT_OF_MEMORY, "InsertExecutor: no enough space to insert a tuple");
  }
  // update indexes
  for (IndexInfo *index_info : exec_ctx_->GetCatalog()->GetTableIndexes(table_info_->name_)) {
    auto key_tuple =
        cur_tuple.KeyFromTuple(table_info_->schema_, index_info->key_schema_, index_info->index_->GetKeyAttrs());
    index_info->index_->InsertEntry(key_tuple, rid, exec_ctx_->GetTransaction());
  }
}
```

### update

child executor为SeqScanExecutor，利用child executor依次查询到需要更新的tuple，用GenerateUpdatedTuple函数生成更新后的tuple并且更新。

```cpp
UpdateExecutor::UpdateExecutor(ExecutorContext *exec_ctx, const UpdatePlanNode *plan,
                               std::unique_ptr<AbstractExecutor> &&child_executor)
    : AbstractExecutor(exec_ctx), plan_(plan), child_executor_(std::move(child_executor)) {}

void UpdateExecutor::Init() {
  child_executor_->Init();
  table_info_ = exec_ctx_->GetCatalog()->GetTable(plan_->TableOid());
}

bool UpdateExecutor::Next([[maybe_unused]] Tuple *tuple, RID *rid) {
  Tuple old_tuple;
  Tuple new_tuple;
  RID tuple_rid;
  while (true) {
    if (!child_executor_->Next(&old_tuple, &tuple_rid)) {
      break;
    }
    new_tuple = GenerateUpdatedTuple(old_tuple);
    TableHeap *table_heap = table_info_->table_.get();
    table_heap->UpdateTuple(new_tuple, tuple_rid, exec_ctx_->GetTransaction());
  }
  return false;
}

Tuple UpdateExecutor::GenerateUpdatedTuple(const Tuple &src_tuple) {
  const auto &update_attrs = plan_->GetUpdateAttr();
  Schema schema = table_info_->schema_;
  uint32_t col_count = schema.GetColumnCount();
  std::vector<Value> values;
  for (uint32_t idx = 0; idx < col_count; idx++) {
    if (update_attrs.find(idx) == update_attrs.cend()) {
      values.emplace_back(src_tuple.GetValue(&schema, idx));
    } else {
      const UpdateInfo info = update_attrs.at(idx);
      Value val = src_tuple.GetValue(&schema, idx);
      switch (info.type_) {
        case UpdateType::Add:
          values.emplace_back(val.Add(ValueFactory::GetIntegerValue(info.update_val_)));
          break;
        case UpdateType::Set:
          values.emplace_back(ValueFactory::GetIntegerValue(info.update_val_));
          break;
      }
    }
  }
  return Tuple{values, &schema};
}
```

### delete

与插入子查询的过程类似，不过执行的是删除过程。

```cpp
DeleteExecutor::DeleteExecutor(ExecutorContext *exec_ctx, const DeletePlanNode *plan,
                               std::unique_ptr<AbstractExecutor> &&child_executor)
    : AbstractExecutor(exec_ctx), plan_(plan), child_executor_(std::move(child_executor)) {}

void DeleteExecutor::Init() {
  table_info_ = exec_ctx_->GetCatalog()->GetTable(plan_->TableOid());
}

bool DeleteExecutor::Next([[maybe_unused]] Tuple *tuple, RID *rid) {
  Tuple del_tuple;
  RID del_rid;
  while (true) {
    try {
      if (!child_executor_->Next(&del_tuple, &del_rid)) {
        break;
      }
    } catch (Exception *e) {
      throw Exception(ExceptionType::UNKNOWN_TYPE, "DeleteExecutor: child executor error");
    }
    DeleteTuple(del_tuple, del_rid);
  }
  return false;
}

void DeleteExecutor::DeleteTuple(const Tuple &del_tuple, const RID &del_rid) {
  TableHeap *table_heap = table_info_->table_.get();
  table_heap->MarkDelete(del_rid, exec_ctx_->GetTransaction());
  for (IndexInfo *index_info : exec_ctx_->GetCatalog()->GetTableIndexes(table_info_->name_)) {
    auto key_tuple =
        del_tuple.KeyFromTuple(table_info_->schema_, index_info->key_schema_, index_info->index_->GetKeyAttrs());
    index_info->index_->DeleteEntry(key_tuple, del_rid, exec_ctx_->GetTransaction());
  }
}
```

### nested loop

用left exectutor和right executor分别对进行join的两张表进行线性查询。查询计划plan中的predicate为ComparisonExpression的实例，用于判断连接条件是否成立。为了方便Next过程，使用`std::vector<Tuple> result_`数据成员来存储Init过程中得到的所有连接后的tuple，然后在Next过程中依次返回连接结果。

```cpp
NestedLoopJoinExecutor::NestedLoopJoinExecutor(ExecutorContext *exec_ctx, const NestedLoopJoinPlanNode *plan,
                                               std::unique_ptr<AbstractExecutor> &&left_executor,
                                               std::unique_ptr<AbstractExecutor> &&right_executor)
    : AbstractExecutor(exec_ctx),
      plan_(plan),
      left_executor_(std::move(left_executor)),
      right_executor_(std::move(right_executor)),
      now_idx_(0) {}

void NestedLoopJoinExecutor::Init() {
  Tuple left_tuple;
  Tuple right_tuple;
  RID left_rid;
  RID right_rid;
  left_executor_->Init();
  const Schema *left_schema = left_executor_->GetOutputSchema();
  const Schema *right_schema = right_executor_->GetOutputSchema();
  while (left_executor_->Next(&left_tuple, &left_rid)) {
    right_executor_->Init();
    while (right_executor_->Next(&right_tuple, &right_rid)) {
      if (plan_->Predicate() == nullptr ||
          plan_->Predicate()->EvaluateJoin(&left_tuple, left_schema, &right_tuple, right_schema).GetAs<bool>()) {
        std::vector<Value> output;
        for (const Column &col : GetOutputSchema()->GetColumns()) {
          output.push_back(col.GetExpr()->EvaluateJoin(&left_tuple, left_schema, &right_tuple, right_schema));
        }
        result_.emplace_back(Tuple(output, GetOutputSchema()));
      }
    }
  }
}

bool NestedLoopJoinExecutor::Next(Tuple *tuple, RID *rid) {
  if (now_idx_ < result_.size()) {
    *tuple = result_[now_idx_];
    now_idx_++;
    return true;
  }
  return false;
}
```

### hash join

假设内存中可以容纳所有的中间结果，hash join的基本过程是：

* 建立哈希表，读取左表中的所有key tuple，计算哈希值，将`<HashKey, key>`加入哈希表中。
* 探测过程，读取右表中的所有key tuple，计算哈希值，判断是否在哈希表中存在，如果存在，则连接对应元组，加入结果集中。


#### hash join key的实现

可以用`HashJoinKey`类对join key其进行封装，为了进行连接操作时的比较操作，应该重载类中的比较比较操作符，为了可以将join key作为unordered_map的键值，应该实现`std::hash`对于join key的方法。

```cpp
namespace bustub {

  struct HashJoinKey {
    Value column_val_;

    bool operator==(const HashJoinKey &other) const {
      return column_val_.CompareEquals(other.column_val_) == CmpBool::CmpTrue;
    }
  };

}

namespace std {

/** Implements std::hash on AggregateKey */
template <>
struct hash<bustub::HashJoinKey> {
  std::size_t operator()(const bustub::HashJoinKey &agg_key) const {
    size_t curr_hash = 0;
    if (!agg_key.column_val_.IsNull()) {
      curr_hash = bustub::HashUtil::CombineHashes(curr_hash, bustub::HashUtil::HashValue(&agg_key.column_val_));
    }
    return curr_hash;
  }
};
}  // namespace std
```

#### hash join executor的实现
```cpp
HashJoinExecutor::HashJoinExecutor(ExecutorContext *exec_ctx, const HashJoinPlanNode *plan,
                                   std::unique_ptr<AbstractExecutor> &&left_child,
                                   std::unique_ptr<AbstractExecutor> &&right_child)
    : AbstractExecutor(exec_ctx),
      plan_(plan),
      left_child_(std::move(left_child)),
      right_child_(std::move(right_child)) {}

void HashJoinExecutor::Init() {
  left_child_->Init();
  right_child_->Init();
  Tuple left_tuple;
  RID left_rid;
  // build hash table
  while (left_child_->Next(&left_tuple, &left_rid)) {
    HashJoinKey dis_key;
    dis_key.column_val_ = plan_->LeftJoinKeyExpression()->Evaluate(&left_tuple, left_child_->GetOutputSchema());
    if (map_.count(dis_key) != 0) {
      map_[dis_key].emplace_back(left_tuple);
    } else {
      map_[dis_key] = std::vector{left_tuple};
    }
  }
  Tuple right_tuple;
  RID right_rid;
  // probing
  while (right_child_->Next(&right_tuple, &right_rid)) {
    HashJoinKey dis_key;
    dis_key.column_val_ = plan_->RightJoinKeyExpression()->Evaluate(&right_tuple, right_child_->GetOutputSchema());
    if (map_.count(dis_key) != 0) {
      for (const Tuple &tmp_tuple : map_.find(dis_key)->second) {
        std::vector<Value> output;
        for (const Column &col : GetOutputSchema()->GetColumns()) {
          output.push_back(col.GetExpr()->EvaluateJoin(&tmp_tuple, left_child_->GetOutputSchema(), &right_tuple,
                                                       right_child_->GetOutputSchema()));
        }
        result_.emplace_back(Tuple(output, GetOutputSchema()));
      }
    }
  }
}

bool HashJoinExecutor::Next(Tuple *tuple, RID *rid) {
  if (now_id_ < result_.size()) {
    *tuple = result_[now_id_];
    *rid = tuple->GetRid();
    now_id_++;
    return true;
  }
  return false;
}
```

### aggregation

aggregation key以及aggregation hash table的实现已经给出，下面给出aggregation executor的实现。

```cpp
AggregationExecutor::AggregationExecutor(ExecutorContext *exec_ctx, const AggregationPlanNode *plan,
                                         std::unique_ptr<AbstractExecutor> &&child)
    : AbstractExecutor(exec_ctx),
      plan_(plan),
      child_(std::move(child)),
      aht_(plan->GetAggregates(), plan->GetAggregateTypes()),
      aht_iterator_(aht_.Begin()) {}

void AggregationExecutor::Init() {
  child_->Init();
  Tuple tuple;
  RID rid;
  while (child_->Next(&tuple, &rid)) {
    aht_.InsertCombine(MakeAggregateKey(&tuple), MakeAggregateValue(&tuple));
  }
  aht_iterator_ = aht_.Begin();
}

bool AggregationExecutor::Next(Tuple *tuple, RID *rid) {
  if (aht_iterator_ == aht_.End()) {
    return false;
  }
  const AggregateKey &agg_key = aht_iterator_.Key();
  const AggregateValue &agg_val = aht_iterator_.Val();
  ++aht_iterator_;
  if (plan_->GetHaving() == nullptr ||
      plan_->GetHaving()->EvaluateAggregate(agg_key.group_bys_, agg_val.aggregates_).GetAs<bool>()) {
    std::vector<Value> ret;
    for (const Column &col : plan_->OutputSchema()->GetColumns()) {
      ret.push_back(col.GetExpr()->EvaluateAggregate(agg_key.group_bys_, agg_val.aggregates_));
    }
    *tuple = Tuple(ret, plan_->OutputSchema());
    return true;
  }
  return false;
}
```

### limit

调用seq scan executor读取limit条元组即可。

```cpp
LimitExecutor::LimitExecutor(ExecutorContext *exec_ctx, const LimitPlanNode *plan,
                             std::unique_ptr<AbstractExecutor> &&child_executor)
    : AbstractExecutor(exec_ctx),
      plan_(plan),
      child_executor_(std::move(child_executor)) {}

void LimitExecutor::Init() {
  child_executor_->Init();
  count_ = 0;
}

bool LimitExecutor::Next(Tuple *tuple, RID *rid) {
  if (count_ == plan_->GetLimit()) {
    return false;
  }
  while (!child_executor_->Next(tuple, rid)) {}
  count_++;
  return true;
}
```

### distinct

模仿`AggregateKey`实现`DistinctKey`。

```cpp
namespace bustub {
  struct DistinctKey {
    std::vector<Value> distincts_;

    bool operator==(const DistinctKey &other) const {
      for (size_t i = 0; i < other.distincts_.size(); i++) {
        if (distincts_[i].CompareEquals(other.distincts_[i]) != CmpBool::CmpTrue) {
          return false;
        }
      }
      return true;
    }
  };
} // namespace bustub

namespace std {
template <>
struct hash<bustub::DistinctKey> {
  std::size_t operator()(const bustub::DistinctKey &agg_key) const {
    size_t curr_hash = 0;
    for (const auto &key : agg_key.distincts_) {
      if (!key.IsNull()) {
        curr_hash = bustub::HashUtil::CombineHashes(curr_hash, bustub::HashUtil::HashValue(&key));
      }
    }
    return curr_hash;
  }
};
} // namespace std
```

用一个哈希表记录已经读取过的键值。

```cpp
void DistinctExecutor::Init() {
  child_executor_->Init();
}

bool DistinctExecutor::Next(Tuple *tuple, RID *rid) {
  Tuple temp_tuple;
  RID temp_rid;
  if (child_executor_->Next(&temp_tuple, &temp_rid)) {
    DistinctKey distinct_key;
    uint32_t col_count = plan_->OutputSchema()->GetColumnCount();
    distinct_key.distincts_.reserve(col_count);
    for (uint32_t i = 0; i < col_count; i++) {
      distinct_key.distincts_.push_back(temp_tuple.GetValue(plan_->OutputSchema(), i));
    }
    if (set_.count(distinct_key) > 0) {
      return Next(tuple, rid);
    }
    set_.insert(distinct_key);
    *tuple = temp_tuple;
    *rid = temp_rid;
    return true;
  }
  return false;
}
```
