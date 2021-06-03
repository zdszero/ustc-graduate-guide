---
title: 中科大2015年复试机试题详解
date: 2021-05-15 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

输出 abcdefghij 任取 5 个字符所有组合，不能有重复，输出到控制台
```
格式：
第 1 种组合：a,b,c,d,e
第 n 种组合：....
......
```

### 代码

```cpp
#include <iostream>
using namespace std;

string s = "abcdefg";
int cnt = 1;
int size = 0;
char comb[5];

void solve(int idx) {
  if (size == 5) {
    printf("第%d种组合: %c, %c, %c, %c, %c\n", cnt++, comb[0], comb[1], comb[2],
           comb[3], comb[4]);
    return;
  }
  for (int i = idx; i < s.length(); i++) {
    comb[size++] = s[i];
    solve(i + 1);
    size--;
  }
}

int main(int argc, char *argv[]) {
  solve(0);
  return 0;
}
```

## 第二题

### 题目描述

设计一个模拟测试系统。输出 0-50 以内 2 个随机数的加或减。从键盘输入答案，每题有2次机会，第一次答对得10分，第二次答对得5分，总共10题，最终输出总得分。

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int score = 0;
  for (int i = 0; i < 10; i++) {
    int a = rand() % 51, b = rand() % 51, c = rand() % 2;
    int guess, ans;
    char op;
    if (c == 0) {
      op = '+';
      ans = a + b;
    } else {
      op = '-';
      ans = a - b;
    }
    for (int j = 0; j < 2; j++) {
      printf("%d %c %d = ", a, op, b);
      scanf("%d", &guess);
      if (guess == ans) {
        cout << "right" << endl;
        score += (10 - j*5);
        break;
      } else {
        cout << "wrong" << endl;
      }
    }
  }
  printf("score = %d\n", score);
  return 0;
}
```

## 第三题

### 题目描述

输出树的层次遍历的奇数层的所有结点。从`input_3.txt`输入，输出到`output_3.txt`。

```
输入格式：
A B C
B E
C F G

树
    A
   / \
  B   C
 /   / \
E   F   G

输出格式：
第 1 层结点：A
第 3 层结点：E,F,G
```

### 解题思路

+ 如何用给定的输入方式构建实际的树？用`map<char, TreeNode *>`来建立字符到结点的映射。
+ 写一个层次遍历来输出奇数层的结点。

### 代码

```cpp
#include <fstream>
#include <queue>
#include <sstream>
#include <unordered_map>
using namespace std;

struct TreeNode {
  TreeNode *lchlid, *rchild;
  char val;
  TreeNode(char val) : lchlid(nullptr), rchild(nullptr), val(val) {}
};

int main(int argc, char *argv[]) {
  ifstream ifs("./input_3.txt");
  ofstream ofs("./output_3.txt");
  unordered_map<char, TreeNode *> nodeMap;
  char a, b, c;
  string line;
  TreeNode *root, *t, *l, *r;
  ifs >> a;
  root = new TreeNode(a);
  nodeMap[a] = root;
  ifs.putback(a);
  while (getline(ifs, line)) {
    istringstream iss(line);
    iss >> a >> b;
    if (!nodeMap[a]) {
      nodeMap[a] = new TreeNode(a);
    }
    if (!nodeMap[b]) {
      nodeMap[b] = new TreeNode(b);
    }
    t = nodeMap[a];
    l = nodeMap[b];
    t->lchlid = l;
    if (iss >> c && !nodeMap[c]) {
      r = new TreeNode(c);
      nodeMap[c] = r;
      t->rchild = r;
    }
  }
  int size = 1, level = 1;
  queue<TreeNode *> q;
  q.push(root);
  while (!q.empty()) {
    int newSize = 0;
    int flag = level % 2;
    if (flag) ofs << level << ": ";
    for (int i = 0; i < size; i++) {
      if (flag && i != 0) ofs << " ";
      t = q.front();
      q.pop();
      if (flag) ofs << t->val;
      if (t->lchlid) {
        q.push(t->lchlid);
        newSize++;
      }
      if (t->rchild) {
        q.push(t->rchild);
        newSize++;
      }
    }
    if (flag) ofs << endl;
    size = newSize;
    level++;
  }
  return 0;
}
```

## 第四题

### 题目描述

从文件`input_4.txt`输入一个图，要求输出从1经过k到n的最短路径，可以有环，输出到文件`output_4.txt`。

```
输入：
n=5
k=3
1 2 10 5 30
2 3 20
3 4 60 5 10
4 5 20
5
输出：
1 2 3 5
```

![graph](../img/graph.png)

### 解题思路

本题考察最短路径，1经过k到n的最短路径即1到k的最短路径加上k到n的最短路径。我们用`next`数组来记录两点之间的最短路径的中间点，用`g`数组来保存两点之间的路径。建议在遇到最短路径问题时直接用`floyed-warshall`算法，简单粗暴，便于记忆。考场上写`djkstra`还是不稳。

### 代码

```cpp
#include <vector>
#include <fstream>
#include <sstream>
#include <climits>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./input_4.txt");
  ofstream ofs("./output_4.txt");
  int n, k;
  ifs.ignore(2);
  ifs >> n;
  ifs.ignore(3);
  ifs >> k;
  ifs.ignore(1);
  vector<vector<int>> g(n+1, vector<int>(n+1, INT_MAX));
  vector<vector<int>> next(n+1, vector<int>(n+1, 0));
  for (int i = 0; i < n; i++) {
    g[i][i] = 0;
    next[i][i] = i;
  }
  string line;
  while (getline(ifs, line)) {
    istringstream iss(line);
    int src, dst, distance;
    iss >> src;
    while (iss >> dst >> distance) {
      g[src][dst] = distance;
      next[src][dst] = dst;
    }
  }
  for (int x = 0; x < n; x++) {
    for (int i = 0; i < n; i++) {
      for (int j = 0; j < n; j++) {
        if (g[i][x] == INT_MAX || g[x][j] == INT_MAX)
          continue;
        int sum = g[i][x] + g[x][j];
        if (sum < g[i][j]) {
          g[i][j] = sum;
          next[i][j] = x;
        }
      }
    }
  }
  int cur = 1;
  ofs << cur;
  while (cur != k) {
    cur = next[cur][k];
    ofs << " " << cur;
  }
  while (cur != n) {
    cur = next[cur][n];
    ofs << " " << cur;
  }
  return 0;
}
```
