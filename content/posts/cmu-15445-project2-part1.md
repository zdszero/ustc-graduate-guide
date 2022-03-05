---
title: cmu 15445 可扩展哈希表实现 - part1
date: 2022-02-06 15:25:05
tags: ["cmu-15445"]
draft: true
---

## 项目说明

project2要求实现一个extendible hash table。

项目的第一个部分要求实现`hash_table_directory_page`和`hash_table_bucket_page`。具体要求请查阅官网[project specification](https://15445.courses.cs.cmu.edu/fall2021/project2/#hash-table-bucket-page)。

## 思路

### bucket page和directory page的内存布局

bucket page和directory page的大小均为4096B，它们实际在运行过程中对应于Page类中的`char data_[PAGE_SIZE]`，在使用的过程中通过`reinterpre_cast`获取：

```
Page *p = buffer_pool_manager_->FetchPage();
HASH_TABLE_BUCKET_TYPE *bucket_page = reinterpre_cast<HASH_TABLE_BUCKET_TYPE *>(p->GetData());
```

### BUCKET_ARRAY_SIZE

`BUCKET_ARRAY_SIZE`定义在另一个头文件中，表示桶中最多可以存放的键值对个数。

桶中需要`array_`数组用于存放所有的键值对集合，以及用于记录键值对信息的`occupied_`和`readable_`数组。

其中occupied表示该位置是否存放过键值对，readable表示该位置当前是否仍然有键值对。（楼主觉得`occupied_`数组是多余的，但是实验代码给了，就先这么实现吧）。

接下来考虑`BUCKET_ARRAY_SIZE`是怎么得到的：

每一个键值对的存储空间为`sizeof(MappingType)`，同时分别需要用`occupied_`和`readable_`数组的一个比特来记录当前状态，所以每个键值对需要占用的总空间为`sizeof(Mapping) + 1/4` Byte，最大键值对个数为：

$$\frac{page\ size}{sizeof(MappingType) + \frac{1}{4}} = \frac{4 * page\ size}{4 * sizeof(MappingType) + 1}$$

`occupied_`数组长度定义为`(BUCKET_ARRAY_SIZE - 1) / 8 + 1`，即`BUCKET_ARRAY_SIZE / 8`的上界。

## 桶实现

### 键值对信息查询和修改

主要是一些位操作，请读者自行思考。

注意`BUCKET_ARRAY_SIZE`不一定恰好是8的整数倍，所以在实现`IsFull`和`IsEmpty`的过程中需要对`readable_`数组中的最后一个元素进行特殊判断。

下面给出部分代码的实现：

```cpp
bool HASH_TABLE_BUCKET_TYPE::IsReadable(uint32_t bucket_idx) const {
  size_t idx = bucket_idx >> 3;
  size_t offset = bucket_idx & 7;
  return (readable_[idx] & (1 << offset)) != 0;
}

void HASH_TABLE_BUCKET_TYPE::SetReadable(uint32_t bucket_idx) {
  size_t idx = bucket_idx >> 3;
  size_t offset = bucket_idx & 7;
  readable_[idx] |= (1 << offset);
}

void HASH_TABLE_BUCKET_TYPE::RemoveAt(uint32_t bucket_idx) {
  // set the bit to 0
  size_t idx = bucket_idx >> 3;
  size_t offset = bucket_idx & 7;
  readable_[idx] &= (~(1 << offset));
}

bool HASH_TABLE_BUCKET_TYPE::IsFull() {
  size_t size = BUCKET_ARRAY_SIZE >> 3;
  for (size_t i = 0; i < size; i++) {
    if (static_cast<uint8_t>(readable_[i]) != 0xff) {
      return false;
    }
  }
  size_t remainder = (BUCKET_ARRAY_SIZE & 7);
  if (remainder > 0) {
    return readable_[size] == ((1 << remainder) - 1);
  }
  return true;
}
```

### 插入

找到第一个`!IsReadable`的位置标记为可插入位置，注意必须扫描所有的键值对来查询带插入键值对是否已经存在。

```cpp
bool HASH_TABLE_BUCKET_TYPE::Insert(KeyType key, ValueType value, KeyComparator cmp) {
  // find the first place readable == 0
  int available_idx = -1;
  for (size_t bucket_idx = 0; bucket_idx < BUCKET_ARRAY_SIZE; bucket_idx++) {
    if (!IsReadable(bucket_idx)) {
      if (available_idx == -1) {
        available_idx = bucket_idx;
      }
      continue;
    }
    // readable
    if (cmp(KeyAt(bucket_idx), key) == 0 && ValueAt(bucket_idx) == value) {
      return false;
    }
  }
  if (available_idx != -1) {
    array_[available_idx].first = key;
    array_[available_idx].second = value;
    SetOccupied(available_idx);
    SetReadable(available_idx);
    return true;
  }
  return false;
}
```

### 删除

```cpp
bool HASH_TABLE_BUCKET_TYPE::Remove(KeyType key, ValueType value, KeyComparator cmp) {
  bool ret = false;
  for (size_t bucket_idx = 0; bucket_idx < BUCKET_ARRAY_SIZE; bucket_idx++) {
    if (!IsOccupied(bucket_idx)) {
      break;
    }
    if (IsReadable(bucket_idx) && cmp(key, KeyAt(bucket_idx)) == 0 && value == ValueAt(bucket_idx)) {
      ret = true;
      RemoveAt(bucket_idx);
      break;
    }
  }
  return ret;
}
```

### 查询

```cpp
bool HASH_TABLE_BUCKET_TYPE::GetValue(KeyType key, KeyComparator cmp, std::vector<ValueType> *result) {
  bool ret = false;
  for (size_t bucket_idx = 0; bucket_idx < BUCKET_ARRAY_SIZE; bucket_idx++) {
    if (!IsOccupied(bucket_idx)) {
      break;
    }
    if (IsReadable(bucket_idx) && cmp(KeyAt(bucket_idx), key) == 0) {
      ret = true;
      result->push_back(ValueAt(bucket_idx));
    }
  }
  return ret;
}
```

## 目录实现

绝大多数函数都是很好实现的，下面列出一部分稍微复杂的函数实现。

```cpp
uint32_t HashTableDirectoryPage::GetLocalDepthMask(uint32_t bucket_idx) {
  return ((1 << local_depths_[bucket_idx]) - 1);
}

uint32_t HashTableDirectoryPage::Size() { return (1 << global_depth_); }

bool HashTableDirectoryPage::CanShrink() {
  for (uint32_t i = 0; i < Size(); i++) {
    assert(GetLocalDepth(i) <= global_depth_);
    if (GetLocalDepth(i) == global_depth_) {
      return false;
    }
  }
  return true;
}

uint32_t HashTableDirectoryPage::GetLocalHighBit(uint32_t bucket_idx) { return 1 << local_depths_[bucket_idx]; }
```
