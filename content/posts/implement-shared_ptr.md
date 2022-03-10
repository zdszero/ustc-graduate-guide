---
title: implement-shared_ptr
date: 2022-03-10 15:42:05
tags: ["cpp"]
draft: true
---

## haha

```cpp
namespace smart_ptr {

class ref_count {
 public:
  unsigned inc_ref() { return ++count_; }
  unsigned dec_ref() { return --count_; }
  unsigned use_count() { return count_; }

 private:
  unsigned count_{1};
};

template <typename T>
struct DefaultDeleter {
  void operator()(T *ptr) {
    if (ptr) delete ptr;
  }
};

template <typename T, typename Deleter = DefaultDeleter<T>>
class shared_ptr {
 public:
  // CONSTRUCTOR
  explicit shared_ptr(T *ptr = nullptr) : ptr_(ptr) {
    if (ptr != nullptr) {
      rep_ = new ref_count();
    }
  }

  // COPY CONSTRUCTOR AND ASSIGNMENT
  shared_ptr(const shared_ptr &rhs) noexcept : ptr_(rhs.ptr_), rep_(rhs.rep_) {
    if (rep_) {
      rep_->inc_ref();
    }
  }
  shared_ptr &operator=(const shared_ptr &rhs) noexcept {
    shared_ptr{rhs}.swap(*this);
    return *this;
  }

  // MOVE CONSTRUCTOR AND ASSIGNMENT
  shared_ptr(shared_ptr &&rhs) noexcept : ptr_(rhs.ptr_), rep_(rhs.rep_) {
    rhs.ptr_ = nullptr;
    rhs.rep_ = nullptr;
  }
  shared_ptr &operator=(shared_ptr &&rhs) noexcept {
    shared_ptr{std::move(rhs)}.swap(*this);
    return *this;
  }

  // DESTRUCTOR
  ~shared_ptr() {
    if (rep_ && rep_->dec_ref() == 0) {
      del_(ptr_);
      delete rep_;
    }
  }

  T *get() const noexcept { return ptr_; }
  void reset(T *ptr) noexcept { shared_ptr{ptr}.swap(*this); }
  void swap(shared_ptr &rhs) noexcept {
    std::swap(ptr_, rhs.ptr_);
    std::swap(rep_, rhs.rep_);
  }
  unsigned use_count() const noexcept { return rep_ ? rep_->use_count() : 0; }

  T *operator->() const noexcept { return ptr_; }
  T &operator*() const { return *ptr_; }

  explicit operator bool() const noexcept { return static_cast<bool>(ptr_); }

 private:
  T *ptr_{nullptr};
  Deleter del_;
  ref_count *rep_{nullptr};
};

template <typename T, typename... Args>
auto make_shared(Args &&...args) {
  return shared_ptr<T>(new T(std::forward<Args>(args)...));
}
```
