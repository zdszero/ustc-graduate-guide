---
title: 中科大2009年复试机试题详解
date: 2021-05-09 12:00:00
tags: ["中科大复试"]
draft: true
---

## 题目一

### 题目描述

输入0~65535之间的十进制数。进行如下处理：比如输入4，转化为16位二进制数0000 0000 0000 0100，4个一组，相异或，变为0001，然后把0001转化为十进制的1然后输出。

```
输入：4
输出：1
```

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int16_t num;
  cin >> num;
  num = (num ^ (num >> 4) ^ (num >> 8) ^ (num >> 12)) & 0x000f;
  cout << num << endl;
  return 0;
}
```

## 题目二

### 题目描述

处理：将n个数由小到大排序，如果n是奇数，输出正中间的数；如果n是偶数，输出正中间的两个数。

### 代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main(int argc, char *argv[]) {
  int n;
  cin >> n;
  vector<int> a(n);
  for (int i = 0; i < n; i++) {
    cin >> a[i];
  }
  sort(a.begin(), a.end());
  cout << a[n/2] << endl;
  return 0;
}
```

## 题目三

### 题目描述

读入文件`tree.in`中的二叉树先序序列，0表示叶子。中序输出深度&le;depth/2的结点，其中depth是你所建立的树的深度。

```
输入示例：
ABC00DE0000
输出示例：
B A
```

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <vector>
using namespace std;

int depth;
int flag = 0;

struct TreeNode {
  TreeNode *lchild, *rchild;
  char val;
  TreeNode(char val) : val(val) {}
};

TreeNode *buildTree(vector<char> &v, int &idx) {
  if (v[idx] == '0') return nullptr;
  TreeNode *t = new TreeNode(v[idx]);
  t->lchild = buildTree(v, ++idx);
  t->rchild = buildTree(v, ++idx);
  return t;
}

int getDepth(TreeNode *t) {
  if (!t) return 0;
  return max(getDepth(t->lchild), getDepth(t->rchild)) + 1;
}

void postOrder(TreeNode *t, int dep) {
  if (!t) return;
  postOrder(t->lchild, dep + 1);
  if (dep <= depth / 2) {
    if (flag) cout << " ";
    flag = 1;
    cout << t->val;
  }
  postOrder(t->rchild, dep + 1);
}

int main(int argc, char *argv[]) {
  ifstream ifs("./tree.in");
  char c;
  vector<char> v;
  while (ifs >> c) v.push_back(c);
  int idx = 0;
  TreeNode *root = buildTree(v, idx);
  depth = getDepth(root);
  postOrder(root, 1);
  return 0;
}
```
