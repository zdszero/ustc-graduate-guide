---
title: 中科大2019年复试机试题详解
date: 2021-05-19 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

一个合法的身份证号码前17个数字加1位校验码组成。校验码的计算规则如下：

首先对前17位数字加权求和，权重分配为：{7，9，10，5，8，4，2，1，6，3，7，9，10，5，8，4，2}；然后将计算的和对11取模得到值`Z`；最后按照以下关系对应`Z`值与校验码`M`的值：

```
Z：0 1 2 3 4 5 6 7 8 9 10
M：1 0 X 9 8 7 6 5 4 3 2
```

现给定一个身份证号，从标准输入读取，判断该身份证是否合理，如果合理输出"YES"，如果不合理输出"NO"。

```
输入示例:
37070419881216001X
输出示例:
YES
```

### 解题思路

注意`char`和`int`类型的转换即可，关键在于理解题意。

### 代码

```cpp
#include <iostream>
using namespace std;

int w[17] = {7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2};
char m[11] = {'1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2'};

bool isValid(string &s) {
  int sum = 0;
  for (int i = 0; i < 17; i++) {
    if (!isdigit(s[i])) {
      return false;
    }
    sum += w[i] * (s[i] - '0');
  }
  return (m[sum % 11] == s[17]);
}

int main(int argc, char *argv[]) {
  string s;
  cin >> s;
  cout << (isValid(s) ? "YES" : "NO");
  return 0;
}
```

## 第二题

### 题目描述

判断给定的数字是否可以拆分为两个2的幂的和的形式，如果可以输出拆分方案，不能输出"NO"。

```
输入示例:
5
输出示例:
5 = 2^0 + 2^1

输入示例:
7
输出示例:
NO
```

### 实现思路

判断该数字对应二进制位是否有两个1即可。

### 代码

```cpp
#include <iostream>
#include <bitset>
using namespace std;

int main(int argc, char *argv[]) {
  int num;
  cin >> num;
  bitset<32> bits(num);
  if (bits.count() != 2) {
    cout << "NO";
  } else {
    int a[2], idx = 0, tmp = num;
    for (int i = 0; i < 32; i++) {
      if (tmp & 1)
        a[idx++] = i;
      tmp >>= 1;
    }
    printf("%d = 2^%d + 2^%d\n", num, a[0], a[1]);
  }
  return 0;
}
```

## 第三题

### 题目描述

给出前缀式，只有加减乘除，求结果。前缀式从文件`pre.in`中读取，将结果输出到标准输出中，题目保证输入的前缀式有效。

```
输入示例:
- + 1 * + 2 3 4 5
输出示例:
16
```

### 解题思路

笔者在复习做到这道题目时直接忘了前缀式如何处理，因为平时只处理过中缀表达式和后缀表达式，忘记了的同学可以看看[这篇文章](https://baike.baidu.com/item/%E5%89%8D%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F)。其实和后缀表达式求值原理一样，只不过是从右向左扫描的。

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <stack>
using namespace std;

int calc(int lhs, int rhs, char op) {
  if (op == '+')
    return lhs + rhs;
  else if (op == '-')
    return lhs - rhs;
  else if (op == '*')
    return lhs * rhs;
  else
    return lhs / rhs;
}

int main(int argc, char *argv[]) {
  ifstream ifs("./pre.in");
  string s;
  getline(ifs, s);
  stack<int> numStk;
  for (int i = s.size() - 1; i >= 0; i--) {
    char c = s[i];
    if (isspace(c)) continue;
    if (isdigit(c)) {
      numStk.push(c - '0');
      continue;
    }
    int lhs = numStk.top();
    numStk.pop();
    int rhs = numStk.top();
    numStk.pop();
    numStk.push(calc(lhs, rhs, c));
  }
  cout << numStk.top();
  return 0;
}
```

## 第四题

### 题目描述

求集合的所有划分，集合从文件`set.in`中读取，结果输出到标准输出中。

```
输入示例:
a b c
输出示例:
{{a,b,c}}
{{a,b},{c}}
{{a,c},{b}}
{{a},{b,c}}
{{a},{b},{c}}
```

**注**：输入 `a b c` 代表集合`{a,b,c}`

### 解题思路

采用`dfs`和`回溯`进行解答，用`vector<vector<char>>`来存储所有的子集，对于一个新元素，可以选择将其加入一个已经存在的子集，也可以将其加入一个新的子集。

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <vector>
using namespace std;

void print(vector<vector<char>> &subsets) {
  cout << '{';
  int flag1 = 0;
  for (auto &a : subsets) {
    if (flag1) cout << ',';
    cout << '{';
    int flag2 = 0;
    for (auto &b : a) {
      if (flag2) cout << ',';
      cout << b;
      flag2 = 1;
    }
    cout << '}';
    flag1 = 1;
  }
  cout << '}';
  cout << endl;
}

void solve(vector<char> &v, vector<vector<char>> &subsets, int idx) {
  if (idx == v.size()) {
    print(subsets);
    return;
  }
  for (int i = 0; i < subsets.size(); i++) {
    subsets[i].push_back(v[idx]);
    solve(v, subsets, idx + 1);
    subsets[i].pop_back();
  }
  subsets.push_back({v[idx]});
  solve(v, subsets, idx + 1);
  subsets.pop_back();
}

int main(int argc, char *argv[]) {
  ifstream ifs("./set.in");
  char c;
  vector<char> v;
  while (ifs >> c) {
    v.push_back(c);
  }
  vector<vector<char>> subsets;
  solve(v, subsets, 0);
  return 0;
}
```

## 第五题

### 题目描述

给出一个二叉排序树的层次遍历，求从它的一个叶子结点到另一个叶子结点的路径，要求路径上经过结点的数值之和最大。二叉树的层次遍历序列从文件`tree.in`中读取，结点数值大于0，将结果输出到标准输出中。

```
输入示例:
25 15 40 5 20 30 50 10 35
输出示例:
165
```

### 解题思路

根据上述输入可以构建如下二叉树，最长路径为 `20, 15, 25, 40, 30, 35`，和为165，这里注意最长路径不一定经过树的根节点。

```
     25
   /    \ 
  15     40
 /  \   /  \
5   20 30  50
 \      \
  10    35
```

首先根据输入建立二叉排序树，然后直接暴力递归即可，按照如下代码

```cpp
#include <fstream>
#include <iostream>
#include <cassert>

using namespace std;

struct TreeNode {
  TreeNode *left{nullptr};
  TreeNode *right{nullptr};
  int val;
  TreeNode(int val): val{val} {}
};

TreeNode *root = nullptr;

void insertNode(int val) {
  if (!root) {
    root = new TreeNode(val);
    return;
  }
  TreeNode *cur = root;
  TreeNode *prev = nullptr;
  while (cur) {
    prev = cur;
    if (val < cur->val) {
      cur = cur->left;
    } else if (val > cur->val) {
      cur = cur->right;
    } else {
      assert(false);
    }
  }
  assert(prev);
  if (val < prev->val) {
    prev->left = new TreeNode(val);
  } else {
    prev->right = new TreeNode(val);
  }
}

#define MAX(a, b, c) max(a, max(b, c))

// 求从根节点到叶结点的最短路径
int getMaxPath(TreeNode *root) {
  if (!root) {
    return 0;
  }
  int ans = root->val + max(getMaxPath(root->left), getMaxPath(root->right));
  return ans;
}


// 求以root为根节点的树中，两个叶结点之间的最长路径，有三种可能性：
// 1. 最长路径经过该根节点
// 2. 最长路径在左子树
// 3. 最长路径在右子树
int getMaxDist(TreeNode *root) {
  if (!root) {
    return 0;
  }
  return MAX(getMaxDist(root->left), getMaxDist(root->right),
      root->val + getMaxPath(root->left) + getMaxPath(root->right));
}

int main() {
  ifstream ifs("tree.in");
  int val;
  while (ifs >> val) {
    insertNode(val);
  }
  cout << getMaxDist(root) << endl;
  return 0;
}
```

以上代码对于 `getMaxPath` 的重复计算较多，可以使用如下方式优化：

```cpp
struct TreeNode {
  TreeNode *left{nullptr};
  TreeNode *right{nullptr};
  int val;
  // 从以该结点为根节点的子树到叶结点的最长路径
  int maxpath{-1};
  TreeNode(int val): val{val} {}
};

int getMaxPath(TreeNode *root) {
  if (!root) {
    return 0;
  }
  if (root->maxpath != -1) {
    return root->maxpath;
  }
  int ans = root->val + max(getMaxPath(root->left), getMaxPath(root->right));
  root->maxpath = ans;
  return ans;
}

int getMaxDist(TreeNode *root) {
  if (!root) {
    return 0;
  }
  return MAX(getMaxDist(root->left), getMaxDist(root->right),
      root->val + getMaxPath(root->left) + getMaxPath(root->right));
}
```
