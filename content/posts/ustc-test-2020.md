---
title: 中科大2020年复试机试题详解
date: 2021-05-20 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

从标准输入读取两个正整数，求这两个数的最大公约数。

```
输入示例:
35 49
输出示例:
7
```

### 代码

```cpp
#include <iostream>
using namespace std;

int gcd(int a, int b) {
  return (b == 0 ? a : gcd(b, a % b));
}

int main(int argc, char *argv[]) {
  int a, b;
  cin >> a >> b;
  if (b > a) swap(a, b);
  cout << gcd(a, b);
  return 0;
}
```

## 第二题

### 题目描述

求长度为n的无序数组中第k大的数。数组从文件`array.in`中读取，首先给出n和k，然后给出数组的n个元素。

```
输入示例:
10 7
6 2 20 99 37 100 33 28 78 44
输出示例:
44
```

### 解题思路

本题**千万不要**先排序完再获取第k个元素，这样估计分得被扣完。该题有两种比较好的思路：
+ 基于堆排序，找到第k大的元素即停止排序。
+ 基于快速排序，因为快速排序每一趟可以确定一个元素的位置，所以递归时只需要向一个方向递归即可。

下述代码采用基于快速排序的方式。

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <vector>
using namespace std;

void partition(vector<int> &a, int l, int r, int k) {
  if (l >= r) return;
  int base = a[l];
  int i = l, j = r;
  while (i < j) {
    while (i < j && a[j] >= base) j--;
    a[i] = a[j];
    while (i < j && a[i] <= base) i++;
    a[j] = a[i];
  }
  a[i] = base;
  if (k == i) return;
  else if (i < k) partition(a, i + 1, r, k);
  else partition(a, l, i - 1, k);
}

int main(int argc, char *argv[]) {
  int n, k;
  ifstream ifs("./array.in");
  ifs >> n >> k;
  vector<int> a(n);
  for (int i = 0; i < n; i++) ifs >> a[i];
  partition(a, 0, n - 1, k - 1);
  cout << a[k - 1];
  return 0;
}
```

## 第三题

### 题目描述

从文件`number.in`中读取若干个正整数，求出它们最大的组合数，输出到标准输出中。

```
输入示例:
123 456 78 782 789
输出示例:
78978782456123
```

### 解题思路

将正整数当作字符串读入，然后对这些字符串进行排序，依次对字符串的高位数字进行比较，将高位数字较大的字符串排在前面。注意对长度不等的字符串的处理，比如`78`和`782`，应该组合为`78782`而不是`78278`。

### 代码

```cpp
#include <algorithm>
#include <fstream>
#include <iostream>
#include <vector>
using namespace std;

bool cmp(const string &l, const string &r) {
  int lsize = l.size();
  int rsize = r.size();
  int maxv = max(l.size(), r.size());
  for (int i = 0; i < maxv; i++) {
    if (l[i % lsize] == r[i % rsize])
      continue;
    return l[i % lsize] > r[i % rsize];
  }
  return true;
};

int main(int argc, char *argv[]) {
  ifstream ifs("./number.in");
  vector<string> v;
  string s;
  while (ifs >> s)
    v.push_back(s);
  sort(v.begin(), v.end(), cmp);
  for (const auto &s : v) cout << s;
  return 0;
}
```
