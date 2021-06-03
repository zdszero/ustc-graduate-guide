---
title: 中科大2021年复试机试题详解
date: 2021-05-21 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

求$\sum_{k=1}^{n} \frac{1}{k!}$，其中n从标准输入读取，将结果输出到标准输出，保留到小数点后4位。

### 解题思路

注意为了避免阶乘溢出，不要先计算$n!$再计算$\frac{1}{n!}$，用$\frac{1}{n!} = \frac{1}{(n-1)!} * \frac{1}{n}$的方式计算。

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int n;
  cin >> n;
  double sum = 0, factor = 1;
  for (int i = 1; i <= n; i++) {
    factor /= i;
    sum += factor;
  }
  printf("%.4f", sum);
  return 0;
}
```

## 第二题

### 题目描述

从标准输入读取数字a和b（64位），求a和b之间的海明距离（二进制表示中不同位的个数）。

### 实现思路

这题看着简单，但千万不要将a和b都分别转化为二进制后再计算不同的位数。直接`a^b`即可保存a和b中不同的二进制位。

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  long long a, b, c;
  cin >> a >> b;
  c = a ^ b;
  int ans = 0;
  for (int i = 0; i < 64; i++) {
    ans += (c & 1);
    c >>= 1;
  }
  cout << ans;
  return 0;
}
```

## 第三题

### 题目描述

从标准输入中读取四个正整数，判断只通过加法和乘法是否能够得到24点？如果可以话在标准输出中输出"YES"，否则输出"NO"。

```
输入示例:
2 2 1 5
输出示例:
NO // (2+2) * (1+5) == 24 可以满足，但不允许使用括号

输入示例:
3 4 1 12
输出示例:
YES // 3*4 + 1*12
```

### 实现思路

本题是24点算法的简单版本，关于完整24点的算法讲解可以参考[LeetCode 679题](https://leetcode-cn.com/problems/24-game/submissions/)，建议在LeetCode上完成24点的原题再来做这题。本题对条件进行了限制，支持的运算符只有`+`和`*`，且不可以使用括号。

24点的算法的本质是使用`dfs`和`回溯`从逻辑上构造（并没有真的构造）不同的表达式树，并采用后序遍历的方式对给定的四个数字进行计算，表达树可以是以下图为例的形式，其中NUM结点可以是任何数字，OP可以是任何算符。

表达式树可以是如下形式：

![tree1](../img/tree1.png)

![tree2](../img/tree2.png)

该题做出了一些限制，我们主要从下面两个方面进行考虑：
+ 运算符只包括`+`、`*`，这两个运算符都具有对称性，满足`opnd1 (+|*) opnd2 == opnd2 (+|*) opnd1`，我们在代码实现中无需对这种重复的情况进行计算。
+ 由于不允许使用括号，所以在低层的OP结点为`+`的情况下，高层的结点不可以是`*`，我们在代码实现中要考虑该情况。

### 代码

```cpp
#include <iostream>
#include <vector>
using namespace std;

// bool表示当前数字是否通过加法计算得到
#define VEC vector<pair<int, bool>>

bool solve(VEC &nums) {
  int size = nums.size();
  if (size == 1) return (nums[0].first == 24);
  VEC tmp;
  // 选举nums中的第i个和第j个数字根据加法或乘法组成新的数字
  for (int i = 0; i < size - 1; i++) {
    for (int j = i + 1; j < size; j++) {
      // 将未被选择数字加入tmp中
      for (int k = 0; k < size; k++)
        if (k != i && k != j) tmp.push_back(nums[k]);
      tmp.push_back(make_pair(nums[i].first + nums[j].first, true));
      if (solve(tmp)) return true;
      tmp.pop_back();
      // 只有当选取的两个数字都不是通过加法计算得到时才能调用乘法对这两个数字进行计算
      if (!nums[i].second && !nums[j].second) {
        tmp.push_back(make_pair(nums[i].first * nums[j].first, false));
        if (solve(tmp)) return true;
      }
      tmp.clear();
    }
  }
  return false;
}

int main(int argc, char *argv[]) {
  VEC nums;
  int num;
  for (int i = 0; i < 4; i++) {
    cin >> num;
    nums.push_back(make_pair(num, false));
  }
  cout << (solve(nums) ? "YES" : "NO");
  return 0;
}
```

## 第四题

### 题目描述

从标准输入读取一个十进制正整数，将该整数转化为七进制输出到标准输出。

### 代码

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main(int argc, char *argv[]) {
  long long num;
  vector<int> seven;
  cin >> num;
  while (num) {
    seven.push_back(num % 7);
    num /= 7;
  }
  for (int i = seven.size() - 1; i >= 0; i--)
    cout << seven[i];
  return 0;
}
```
