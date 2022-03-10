<!-- --- -->
<!-- title: implement a simple c++ unique_ptr -->
<!-- date: 2022-03-09 23:21:09 -->
<!-- tags: ["cpp"] -->
<!-- draft: true -->
<!-- --- -->

## intro

`unique_ptr` is a class which owns and manages a traditional cpp pointer, and delete the managed pointer automatically when going out of scope.

## enlightment

* When the destructor of `unique_ptr` is called, the managed pointer is deleted.
* Just as the name of `unique_ptr` suggests, there cannot be duplicate copies of `unique_ptr` object. So the copy constructor and copy assigment operator should be deleted.
* We can use `std::exchange` and `std::swap` to simplify our code.

## implementation

### first version

```cpp
namespace smart_ptr {

template <typename T>
struct DefaultDeleter {
  void operator()(T *ptr) noexcept {
    if (ptr) delete ptr;
  }
};

template <typename T, typename Deleter = DefaultDeleter<T>>
class unique_ptr {
 public:
  unique_ptr() = default;
  unique_ptr(T *ptr): ptr_(ptr) {}
  ~unique_ptr() { del_(ptr_); }

  unique_ptr &operator=(const unique_ptr &) = delete;
  unique_ptr(const unique_ptr &) = delete;

  unique_ptr(unique_ptr &&rhs) : ptr_(rhs.release()) {}
  unique_ptr &operator=(unique_ptr &&rhs) {
    reset(rhs.release());
    return *this;
  }

  T *get() { return ptr_; }
  T *release() { return std::exchange(ptr_, nullptr); }
  void reset(T *ptr) { del_(std::exchange(ptr_, ptr)); }
  void swap(unique_ptr &rhs) { std::swap(ptr_, rhs.ptr_); }

  T *operator->() { return ptr_; }
  T &operator*() { return *ptr_; }

  operator bool() { return static_cast<bool>(ptr_); }

 private:
  T *ptr_{nullptr};
  Deleter del_;
};

};  // namespace smart_ptr
```

### second version

The second version add more specifiers to functions in `unique_ptr` class

* **noexcept**:
* **explicit**:
* **constexpr**:
* **const**: 

```cpp
template <typename T, typename Deleter = DefaultDeleter<T>>
class unique_ptr {
 public:
  constexpr unique_ptr() noexcept = default;
  constexpr unique_ptr(std::nullptr_t) noexcept : unique_ptr() {}
  explicit unique_ptr(T *ptr) noexcept : ptr_(ptr) {}
  ~unique_ptr() noexcept { del_(ptr_); }

  unique_ptr &operator=(const unique_ptr &) = delete;
  unique_ptr(const unique_ptr &) = delete;
  unique_ptr &operator=(std::nullptr_t) {
    reset(nullptr);
    return *this;
  }

  unique_ptr(unique_ptr &&rhs) noexcept : ptr_(rhs.release()) {}
  unique_ptr &operator=(unique_ptr &&rhs) noexcept {
    reset(rhs.release());
    return *this;
  }

  T *get() const noexcept { return ptr_; }
  T *release() noexcept { return std::exchange(ptr_, nullptr); }
  void reset(T *ptr) noexcept { del_(std::exchange(ptr_, ptr)); }
  void swap(unique_ptr &rhs) noexcept { std::swap(ptr_, rhs.ptr_); }

  T *operator->() const noexcept { return ptr_; }
  T &operator*() const { return *ptr_; }

  explicit operator bool() const noexcept { return static_cast<bool>(ptr_); }

 private:
  T *ptr_{nullptr};
  Deleter del_;
};
```

### make_unique
