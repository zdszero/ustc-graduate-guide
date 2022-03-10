---
title: cmu 15445 并发控制 part2
date: 2022-03-08 16:43:35
tags: ["cmu-15445"]
draft: true
---

## 项目描述

在task1的基础上添加deadlock prevention，在task2这里需要实现的死锁避免策略为wound-wait。

## 思路

![wound-wait](img/wound-wait.png)

死锁避免的实现方式有两种：

* wait-die ("Old Waits for Young")
  → If requesting txn has higher priority than holding txn, then requesting txn waits for holding txn
  → Else requesting txn is aborted
* wound-wait ("Yound Waits for Young")
  → If requesting txn has higher priority than holding txn, then holding txn is aborted
  → Else requesting txn waits

bustub中用事务的编号表示事务的优先级，编号越低的优先级越高。在向请求队列中插入请求时，首先扫描队列，找到比当前请求优先级低的请求，abort对应的事务，然后插入请求。

## 实现

只需改变向请求队列中插入请求的代码即可。

```cpp
void LockManager::LockRequestQueue::Insert(Transaction *txn, const RID &rid, LockMode mode, std::unique_lock<std::mutex> &lck) {
  txn_id_t txn_id = txn->GetTransactionId();
  // deadlock prevention -- wound wait
  for (auto iter = request_queue_.begin(); iter != request_queue_.end();) {
    if (iter->txn_id_ > txn_id) {
      Transaction *prev_txn = TransactionManager::GetTransaction(iter->txn_id_);
      prev_txn->SetState(TransactionState::ABORTED);
      iter = request_queue_.erase(iter);
    } else {
      iter++;
    }
  }
  bool can_grant = CanGrant(mode);
  auto &last = request_queue_.emplace_back(txn_id, mode, can_grant);
  if (!can_grant) {
    lck.unlock();
    last.Wait();
  }
  if (mode == LockMode::SHARED) {
    txn->GetSharedLockSet()->insert(rid);
  } else {
    txn->GetExclusiveLockSet()->insert(rid);
  }
}
```
