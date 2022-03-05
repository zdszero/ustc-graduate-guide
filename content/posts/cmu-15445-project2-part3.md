---
title: cmu 15445 可扩展哈希表实现 - part3
date: 2022-02-08 09:21:43
tags: ["cmu-15445"]
draft: true
---

## 项目说明

项目的第三部分是为已经实现的哈希表加入并发控制，从而保证多个线程同时执行插入、删除、查找操作时的正确性。

## 思路

### 为目录和桶加锁

为了保证并发访问的正确性，需要对目录和桶加锁，对目录加锁直接使用hash table中的`table_latch_`即可，对桶加锁需要调用Page类中的加锁和解锁函数。目录和桶使用的锁均为读者写者锁。

为了方便加锁和解锁过程，在hash table类中封装如下函数：

```cpp
void HASH_TABLE_TYPE::LockBucket(page_id_t page_id, bool exclusive) {
  Page *p = buffer_pool_manager_->FetchPage(page_id);
  if (exclusive) {
    p->WLatch();
  } else {
    p->RLatch();
  }
  buffer_pool_manager_->UnpinPage(page_id, exclusive);
}

void HASH_TABLE_TYPE::UnlockBucket(page_id_t page_id, bool exclusive) {
  Page *p = buffer_pool_manager_->FetchPage(page_id);
  if (exclusive) {
    p->WUnlatch();
  } else {
    p->RUnlatch();
  }
  buffer_pool_manager_->UnpinPage(page_id, exclusive);
}

void HASH_TABLE_TYPE::LockDirectory(bool exclusive) {
  if (exclusive) {
    table_latch_.WLock();
  } else {
    table_latch_.RLock();
  }
}

void HASH_TABLE_TYPE::UnlockDirectory(bool exclusive) {
  if (exclusive) {
    table_latch_.WUnlock();
  } else {
    table_latch_.RUnlock();
  }
}
```

### 加锁的粒度

为了保证并发性能，需要在代码中使得加锁的粒度尽量小。比如在插入过程中，考虑如下两种方案

* 方案一

在整个插入过程中对目录和桶施加写者锁。

* 方案二

对目录施加读者锁，对桶施加写者锁，如果插入后需要进行SplitInsert操作的话，则再重新对目录施加写者锁。

方案二显然比方案一具有更高的并发量，考虑多个线程分别对不同桶中插入键值对，采用方案一的话该过程无法并发执行，只能一个一个地插入，采用方案二的话可以使该过程并发执行。

## 实现

具体代码请参考我的仓库中的代码。
