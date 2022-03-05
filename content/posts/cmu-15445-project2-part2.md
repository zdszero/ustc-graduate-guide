---
title: cmu 15445 可扩展哈希表实现 - part2
date: 2022-02-07 22:32:46
tags: ["cmu-15445"]
draft: true
---

## 项目说明

在项目的第一个部分我们实现了`hash_table_directory_page`和`hash_table_bucket_page`。在这一部分我们需要实现单线程的完整的可扩展哈希表，完成文件`extendible_hash_table.h`和`extendible_hash_table.cpp`。

## 思路

### 哈希表的内存布局

`extendible_hash_table`文件中存储有目录的page id，在获取每个桶页面之前，需要先获取目录页面。目录页面中存储有指向桶的page id，以及该桶的local depth，以及global depth。

### 不变量的测试

不变量（invariant）指程序运行过程中保持不变的一些属性，可以用不变量对程序的正确性进行测试，在程序中加入assert语句，用于检验所编写的代码是否正确。哈希表的一些不变量如下所示：

```
hash_table.Size() = 2^(global_depth_)
bucket number <= hash_table.size()

global_depth >= any(local_depth)
global_depth = max(local_depth)

bucket_reference_count = 2^(global_depth - local_depth)

for all the indexes that point to the same bucket with local_depth(ld):
	the ld LSB are the same
	index interval = (1 << ld)
```

## 实现

### 查找

先根据Hash(key)找到目的桶，再从桶中找到对应键值对即可。

```cpp
inline uint32_t HASH_TABLE_TYPE::KeyToPageId(KeyType key, HashTableDirectoryPage *dir_page) {
  page_id_t page_id = dir_page->GetBucketPageId(KeyToDirectoryIndex(key, dir_page));
  assert(page_id != INVALID_PAGE_ID);
  return page_id;
}

HashTableDirectoryPage *HASH_TABLE_TYPE::FetchDirectoryPage() {
  Page *p = buffer_pool_manager_->FetchPage(directory_page_id_, nullptr);
  return reinterpret_cast<HashTableDirectoryPage *>(p->GetData());
}

bool HASH_TABLE_TYPE::GetValue(Transaction *transaction, const KeyType &key, std::vector<ValueType> *result) {
  HashTableDirectoryPage *directory_page = FetchDirectoryPage();
  page_id_t bucket_pid = KeyToPageId(key, directory_page);
  HASH_TABLE_BUCKET_TYPE *bucket_page = FetchBucketPage(bucket_pid);
  bucket_page->GetValue(key, comparator_, result);
  buffer_pool_manager_->UnpinPage(bucket_pid, false);
  buffer_pool_manager_->UnpinPage(directory_page_id_, false);
  return !result->empty();
}
```

### 插入

插入部分比较复杂，在插入后如果桶未满的话，直接返回即可，否则进行如下操作：

* 增加global depth，相当于对hash table扩容。
* 需要将桶进行分割，根据HighBit将原桶中的键值对分配到原桶或新桶中。
* 更新hash table中的local_depth_数组和bucket_page_ids_数组，使其与指向的桶对应。

```cpp
bool HASH_TABLE_TYPE::Insert(Transaction *transaction, const KeyType &key, const ValueType &value) {
  HashTableDirectoryPage *directory_page = FetchDirectoryPage();
  page_id_t bucket_pid = KeyToPageId(key, directory_page);
  buffer_pool_manager_->UnpinPage(directory_page_id_, false);
  HASH_TABLE_BUCKET_TYPE *bucket_page = FetchBucketPage(bucket_pid);
  if (bucket_page->Insert(key, value, comparator_)) {
    buffer_pool_manager_->UnpinPage(bucket_pid, true);
    return true;
  }
  if (bucket_page->IsFull()) {
    buffer_pool_manager_->UnpinPage(bucket_pid, true);
    return SplitInsert(transaction, key, value);
  }
  buffer_pool_manager_->UnpinPage(bucket_pid, true);
  return false;
}

bool HASH_TABLE_TYPE::SplitInsert(Transaction *transaction, const KeyType &key, const ValueType &value) {
  bool ret = false;
  HashTableDirectoryPage *directory_page = FetchDirectoryPage();
  uint32_t bucket_idx = KeyToDirectoryIndex(key, directory_page);
  page_id_t bucket_pid = KeyToPageId(key, directory_page);
  HASH_TABLE_BUCKET_TYPE *bucket_page = FetchBucketPage(bucket_pid);
  while (true) {
    if (bucket_page->Insert(key, value, comparator_)) {
      ret = true;
      break;
    }
    // full or duplicate <k, v>
    if (!bucket_page->IsFull()) {
      // if duplicate
      break;
    }
    uint32_t high_bit = directory_page->GetLocalHighBit(bucket_idx);
    directory_page->IncrLocalDepth(bucket_idx);
    uint8_t local_depth = directory_page->GetLocalDepth(bucket_idx);
    if (directory_page->GetLocalDepth(bucket_idx) > directory_page->GetGlobalDepth()) {
      // populate bucket array
      uint32_t size = directory_page->Size();
      assert(size < DIRECTORY_ARRAY_SIZE);
      for (uint32_t i = size; i < 2 * size; i++) {
        directory_page->SetBucketPageId(i, directory_page->GetBucketPageId(i - size));
        directory_page->SetLocalDepth(i, directory_page->GetLocalDepth(i - size));
      }
      directory_page->IncrGlobalDepth();
      assert(directory_page->GetGlobalDepth() == directory_page->GetLocalDepth(bucket_idx));
    }
    // create new page and redistribute key,values
    page_id_t new_bucket_pid;
    Page *p = buffer_pool_manager_->NewPage(&new_bucket_pid);
    HASH_TABLE_BUCKET_TYPE *new_bucket_page = reinterpret_cast<HASH_TABLE_BUCKET_TYPE *>(p->GetData());
    for (size_t i = 0; i < BUCKET_ARRAY_SIZE; i++) {
      if ((Hash(bucket_page->KeyAt(i)) & high_bit) != 0) {
        bucket_page->RemoveAt(i);
        new_bucket_page->Insert(bucket_page->KeyAt(i), bucket_page->ValueAt(i), comparator_);
      }
    }
    // point to new buckets
    for (size_t i = 0; i < directory_page->Size(); i++) {
      if (directory_page->GetBucketPageId(i) == bucket_pid) {
        directory_page->SetLocalDepth(i, local_depth);
        if ((i & high_bit) != 0) {
          directory_page->SetBucketPageId(i, new_bucket_pid);
        }
      }
    }
    buffer_pool_manager_->UnpinPage(bucket_pid, true);
    buffer_pool_manager_->UnpinPage(new_bucket_pid, true);
    // next loop
    bucket_idx = KeyToDirectoryIndex(key, directory_page);
    bucket_pid = KeyToPageId(key, directory_page);
    bucket_page = FetchBucketPage(bucket_pid);
  }
  buffer_pool_manager_->UnpinPage(bucket_pid, true);
  buffer_pool_manager_->UnpinPage(directory_page_id_, true);
  return ret;
}
```

### 删除

删除键值对后判断桶是否为空，如果桶不为空，则需要进行merge操作：找到一个与当前桶具有相同local depth的split image，然后合并两个桶，并且更新hash table中的local_depth_数组和bucket_page_ids_数组，使其与指向的桶对应，如果目录可以缩小的话，减小global depth的值。

merge函数运行直到如下条件成立：
* 桶不为空
* 当前桶的local depth为0
* 桶的local depth与它所有的split image的local depth都不相同

```cpp
bool HASH_TABLE_TYPE::Remove(Transaction *transaction, const KeyType &key, const ValueType &value) {
  HashTableDirectoryPage *directory_page = FetchDirectoryPage();
  page_id_t bucket_pid = KeyToPageId(key, directory_page);
  HASH_TABLE_BUCKET_TYPE *bucket_page = FetchBucketPage(bucket_pid);
  bool ret = bucket_page->Remove(key, value, comparator_);
  if (bucket_page->IsEmpty()) {
    Merge(transaction, key, value);
  }
  buffer_pool_manager_->UnpinPage(bucket_pid, true);
  buffer_pool_manager_->UnpinPage(directory_page_id_, false);
  return ret;
}

void HASH_TABLE_TYPE::Merge(Transaction *transaction, const KeyType &key, const ValueType &value) {
  // set the page id to it's split image's page id
  HashTableDirectoryPage *directory_page = FetchDirectoryPage();
  while (true) {
    uint32_t bucket_idx = KeyToDirectoryIndex(key, directory_page);
    page_id_t bucket_pid = KeyToPageId(key, directory_page);
    assert(bucket_pid != INVALID_PAGE_ID);
    HASH_TABLE_BUCKET_TYPE *bucket_page = FetchBucketPage(bucket_pid);
    // no longer empty
    if (!bucket_page->IsEmpty()) {
      buffer_pool_manager_->UnpinPage(bucket_pid, false);
      break;
    }
    buffer_pool_manager_->UnpinPage(bucket_pid, false);
    uint8_t local_depth = directory_page->GetLocalDepth(bucket_idx);
    // only one bucket left
    if (local_depth == 0) {
      break;
    }
    uint32_t interval = (1 << (local_depth - 1));
    page_id_t merged_page_id = INVALID_PAGE_ID;
    for (size_t i = (bucket_idx + interval) % directory_page->Size(); i != bucket_idx;
         i = (i + interval) % directory_page->Size()) {
      page_id_t cur_pid = directory_page->GetBucketPageId(i);
      if (cur_pid == bucket_pid) {
        continue;
      }
      if (directory_page->GetLocalDepth(i) == local_depth) {
        merged_page_id = cur_pid;
        break;
      }
    }
    // ld doesn't match its split image's ld
    if (merged_page_id == INVALID_PAGE_ID) {
      break;
    }
    directory_page->SetBucketPageId(bucket_idx, merged_page_id);
    directory_page->DecrLocalDepth(bucket_idx);
    for (size_t i = (bucket_idx + interval) % directory_page->Size(); i != bucket_idx;
         i = (i + interval) % directory_page->Size()) {
      directory_page->DecrLocalDepth(i);
      if (directory_page->GetBucketPageId(i) == bucket_pid) {
        directory_page->SetBucketPageId(i, merged_page_id);
      }
    }
    buffer_pool_manager_->DeletePage(bucket_pid);
    while (directory_page->CanShrink()) {
      directory_page->DecrGlobalDepth();
    }
  }
  buffer_pool_manager_->UnpinPage(directory_page_id_, true);
}
```
