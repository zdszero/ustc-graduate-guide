---
title: 中科大2011年复试机试题详解
date: 2021-05-11 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

给两个十进制数，先异或，然后输出其二进制形式。

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int a, b, c;
  cin >> a >> b;
  c = a ^ b;
  int bits[32];
  int N = 0;
  if (c == 0)
    cout << 0;
  while (c) {
    bits[N++] = c & 1;
    c >>= 1;
  }
  for (int i = N-1; i >= 0; i--)
    cout << bits[i];
  return 0;
}
```

## 第二题

### 题目描述

一共有十二个球，其颜色有红、黄、黑三种，红黄黑分别有 x，y，k 个，现在从其中取出八个球，共有多少种取法，输出到文件中？（x，y，k 从键盘输入，同种颜色的球不区分）

```
输出格式;
1.  红球， 黄球， 黑球;
2.  红球， 黄球， 黑球;
3.
....
```

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int x, y, k;
  cin >> x >> y >> k;
  int cnt = 0;
  for (int i = 0; i <= 8 && i <= x; i++) {
    for (int j = 0; j <= 8 - i && j <= y; j++) {
      if (8 - i - j <= k) {
        printf("%2d. %d红球，%d黄球，%d黑球\n", ++cnt, i, j, 8 - i - j);
      }
    }
  }
  return 0;
}
```

## 第三题

### 题目描述

在文件`3.txt`中查看是否有模式 abcde。若有，输出`找到abc*d?e匹配`；若无，则输出`没有找到abc*d?e匹配`，将结果输出到控制台。

### 实现思路

人工实现一个识别特定正则表达式的有限自动机

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <sstream>
using namespace std;

string readFile(string filename) {
  ifstream ifs(filename);
  ostringstream oss;
  char c;
  while (ifs >> c) {
    oss.put(c);
  }
  return oss.str();
}

bool check(const string &s, int idx) {
  if (s[idx++] != 'a') return false;
  while (s[idx++] == 'b');
  if (s[idx] == 'e') return true;
  if (s[idx++] != 'd') return false;
  if (s[idx] != 'e') return false;
  return true;
}

int main(int argc, char *argv[]) {
  string s = readFile("./input.txt");
  int flag = 0;
  for (int i = 0; i < s.length(); i++) {
    if (check(s, i)) {
      flag = 1;
      break;
    }
  }
  if (flag) {
    cout << "找到abd*d?e匹配" << endl;
  } else {
    cout << "没找到abd*d?e匹配" << endl;
  }
  return 0;
}
```

## 第四题

### 题目描述

从文件`4.txt`中读入一个二叉树，然后后序遍历该二叉树，将结果输出到文件`4.out`。

```
输入格式：
4 // 表示结点个数
1 2 4 // 当前结点编号 左孩子编号 右孩子编号
2 0 3
3 0 0
4 0 0

输出格式：
3 2 4 1
```

### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

int flag = 0;
ifstream ifs("./tree.in");
ofstream ofs("./tree.out");

struct TreeNode {
  TreeNode *lchild, *rchild;
  int val;
  TreeNode(int val): val(val) {}
};

void postOrder(TreeNode *t) {
  if (!t)
    return;
  postOrder(t->lchild);
  postOrder(t->rchild);
  if (flag)
    ofs << " ";
  flag = 1;
  ofs << t->val;
}

int main(int argc, char *argv[]) {
  int n, rootIdx;
  int a, b, c;
  ifs >> n >> rootIdx;
  vector<TreeNode *> nodes(n+1, nullptr);
  ifs.putback(rootIdx+'0');
  while (ifs >> a >> b >> c) {
    if (!nodes[a]) nodes[a] = new TreeNode(a);
    if (!nodes[b] && b != 0) nodes[b] = new TreeNode(b);
    if (!nodes[c] && c != 0) nodes[c] = new TreeNode(c);
    nodes[a]->lchild = nodes[b];
    nodes[a]->rchild = nodes[c];
  }
  TreeNode *root = nodes[rootIdx];
  postOrder(root);
  return 0;
}
```
