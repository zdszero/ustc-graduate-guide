---
title: 中科大2010年复试机试题详解
date: 2021-05-10 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

`input.txt`文件里面有一段文章，由字符串和一些特殊字符构成。先把 input 文件中的内容读入缓冲区，在从缓冲区中取字符，进行如下操作:
1. 如果是`字符，空格`，输出
2. 如果是`!`，删除前面一个字符
3. 如果是`*`，删除前面 1 行字符串
4. 如果是`>`，讲前面一个单词的首字符，进行大小写转化
5. 如果是`数字`，则不作任何操作

```
输入示例:
this is some > text!
one plus one eqs 2
this line should be deleted
prev line sh*ould be deleted

输出示例:
this is Some  tex
one plus one eqs 
prev line should be deleted
```

### 代码

```cpp
#include <iostream>
#include <fstream>
#include <list>
using namespace std;

int main(int argc, char *argv[]) {
  list<string> lines;
  string tmp;
  ifstream ifs("./input.txt");
  while (getline(ifs, tmp))
    lines.push_back(tmp);
  for (auto it = lines.begin(); it != lines.end(); it++) {
    for (int i = 0; i < it->length(); i++) {
      if (it->at(i) == '!' && i > 0)
        (*it)[i-1] = '0';
      else if (it->at(i) == '*')
        lines.erase(prev(it));
      else if (it->at(i) == '>') {
        int j;
        for (j = i-1; j >= 0 && it->at(j) != ' '; j--);
        for (; j >= 0 && it->at(j) == ' '; j--);
        if (j < 0)
          continue;
        for (; j >= 0 && it->at(j) != ' '; j--);
        (*it)[j+1] = toupper((*it)[j+1]);
      }
    }
  }
  for (auto &s : lines) {
    for (auto c : s) {
      if (isalpha(c) || c == ' ')
        cout << c;
    }
    cout << endl;
  }
  return 0;
}
```

## 第二题

### 题目描述

将两个矩阵相乘并输出。

## 第三题

### 题目描述

已知二叉排序树用二叉链表存储，结点的关键字为正整数。从键盘输入结点的关键字（以0表示结束）建立一棵二叉排序树，并输出其后序遍历序列。

```
输入示例：
40 20 60 70 0

输出示例：
20 70 60 40
```

### 代码

```cpp
#include <iostream>
using namespace std;

struct TreeNode {
  TreeNode *lchild, *rchild;
  int val;
  TreeNode(int val) : lchild(nullptr), rchild(nullptr), val(val) {}
};

TreeNode *root = nullptr;
int flag = 0;

void insertNode(int val) {
  if (!root) {
    root = new TreeNode(val);
    return;
  }
  TreeNode *t = root;
  TreeNode *pre;
  while (t) {
    pre = t;
    if (val >= t->val)
      t = t->rchild;
    else
      t = t->lchild;
  }
  if (val >= pre->val)
    pre->rchild = new TreeNode(val);
  else
    pre->lchild = new TreeNode(val);
}

void postOrder(TreeNode *t) {
  if (!t) return;
  postOrder(t->lchild);
  postOrder(t->rchild);
  if (flag) cout << " ";
  cout << t->val;
  flag = 1;
}

int main(int argc, char *argv[]) {
  int num;
  while (cin >> num) {
    if (num == 0) break;
    insertNode(num);
  }
  postOrder(root);
  return 0;
}
```
