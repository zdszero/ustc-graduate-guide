---
title: cmu-15445-project0
date: 2022-03-08 16:29:54
tags: ["cmu-15445"]
draft: true
---

## 项目说明

完成`src/include/primer/p0_starter.h`中矩阵类和矩阵操作。

## 思路

这个是对C++能力的基本测试，实现非常简单。

## 实现

```cpp
template <typename T>
class Matrix {
 protected:
  Matrix(int rows, int cols): rows_(rows), cols_(cols) {
    linear_ = new T[static_cast<size_t>(rows * cols)];
  }

  int rows_;
  int cols_;

  T *linear_;

 public:
  virtual int GetRowCount() const = 0;

  virtual int GetColumnCount() const = 0;

  virtual T GetElement(int i, int j) const = 0;

  virtual void SetElement(int i, int j, T val) = 0;

  virtual void FillFrom(const std::vector<T> &source) = 0;

  virtual ~Matrix() {
    delete [] linear_;
  }
};

template <typename T>
class RowMatrix : public Matrix<T> {
 public:
  RowMatrix(int rows, int cols) : Matrix<T>(rows, cols) {
    data_ = new T*[rows];
    for (int i = 0; i < rows; i++) {
      data_[i] = Matrix<T>::linear_ + i * cols;
    }
  }

  int GetRowCount() const override { return Matrix<T>::rows_; }

  int GetColumnCount() const override { return Matrix<T>::cols_; }

  T GetElement(int i, int j) const override {
    if (i < 0 || i >= GetRowCount() || j < 0 || j >= GetColumnCount()) {
      throw Exception(ExceptionType::OUT_OF_RANGE, "GetElement() out of range");
    }
    return data_[i][j];
  }

  void SetElement(int i, int j, T val) override {
    if (i < 0 || i >= GetRowCount() || j < 0 || j >= GetColumnCount()) {
      throw Exception(ExceptionType::OUT_OF_RANGE, "SetElement() out of range");
    }
    data_[i][j] = val;
  }

  void FillFrom(const std::vector<T> &source) override {
    size_t sz = static_cast<size_t>(GetRowCount() * GetColumnCount());
    if (source.size() != sz) {
      throw Exception(ExceptionType::OUT_OF_RANGE, "FillFrom() out of range");
    }
    for (size_t i = 0; i < source.size(); i++) {
      Matrix<T>::linear_[i] = source[i];
    }
  }

  ~RowMatrix() override {
    delete [] data_;
  }

 private:
  T **data_;
};

template <typename T>
class RowMatrixOperations {
 public:
  static std::unique_ptr<RowMatrix<T>> Add(const RowMatrix<T> *matrixA, const RowMatrix<T> *matrixB) {
    int rows = matrixA->GetRowCount();
    int cols = matrixA->GetColumnCount();
    auto res = std::make_unique<RowMatrix<T>>(rows, cols);
    for (int r = 0; r < rows; r++) {
      for (int c = 0; c < cols; c++) {
        res->SetElement(r, c, matrixA->GetElement(r, c) + matrixB->GetElement(r, c));
      }
    }
    return res;
  }

  static std::unique_ptr<RowMatrix<T>> Multiply(const RowMatrix<T> *matrixA, const RowMatrix<T> *matrixB) {
    assert(matrixA->GetColumnCount() == matrixB->GetRowCount());
    int rows = matrixA->GetRowCount();
    int cols = matrixB->GetColumnCount();
    int tmp = matrixA->GetColumnCount();
    auto res = std::make_unique<RowMatrix<T>>(rows, cols);
    for (int r = 0; r < rows; r++) {
      for (int c = 0; c < cols; c++) {
        T elem = 0;
        for (int x = 0; x < tmp; x++) {
          elem += matrixA->GetElement(r, x) * matrixB->GetElement(x, c);
        }
        res->SetElement(r, c, elem);
      }
    }
    return res;
  }

  static std::unique_ptr<RowMatrix<T>> GEMM(const RowMatrix<T> *matrixA, const RowMatrix<T> *matrixB,
                                            const RowMatrix<T> *matrixC) {
    return Add(Multiply(matrixA, matrixB), matrixC);
  }
};
```
