---
title: 中科大2013年复试机试题详解
date: 2021-05-13 12:00:00
draft: true
---

前三题太简单，只给出题目。

## 第一题

### 题目描述

在`1.in`中有几个正整数，把这几个正整数中的最大值输出到`1.out`.

## 第二题

### 题目描述

在`2.in`中有几个正整数，把这几个正整数的和输出到`2.out`.

## 第三题

### 题目描述

在`3.in`中有一个正整数，把这个正整数的二进制形式输出到`3.out`.

## 第四题

### 题目描述

在`4.in`中有一个正整数，把这个正整数分解成两个质数的和的方案数输出到`4.out`.

```
Sample input:10
Sample output:2(5+5=10  3+7=10   7+3和3+7是一个方案 )
```

### 代码实现

```cpp
#include <fstream>
#include <cmath>
using namespace std;

bool isOdd(int num) {
  for (int i = 2; i < sqrt(num); i++)
    if (num % i == 0)
      return false;
  return true;
}

int main(int argc, char *argv[]) {
  ifstream ifs("./4.in");
  ofstream ofs("./4.out");
  int num, ans = 0;
  ifs >> num;
  for (int i = 2; i <= num / 2; i++)
    if (isOdd(i) && isOdd(num - i))
      ans++;
  ofs << ans;
  return 0;
}
```

## 第五题

### 题目描述

n皇后问题：n×n格的棋盘上放置彼此不受攻击的n个皇后。按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。n后问题等价于再n×n的棋盘上放置n个皇后，任何2个皇后不妨在同一行或同一列或同一斜线上。在`5.in`中给定棋盘的大小n,在`5.out`中输出放置方法的个数。

```
Sample input：4
Sample output:2
```

### 实现思路

经典的回溯算法，网上关于n皇后的算法原理有很多讲解，这里不赘述。以下代码使用了C++中的一部分函数式编程特性，定义了函数闭包和函数变量，减少了代码篇幅，当然也可以不这么写。

### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./5.in");
  ofstream ofs("./5.out");
  int n;
  ifs >> n;
  int cnt = 0;
  vector<vector<int>> b(n, vector<int> (n, 0));
  vector<pair<int, int>> loc;
  loc.reserve(n);
  auto dfs = [&](auto &&self, int i) -> void {
    auto isValid = [&](int i, int j) {
      for (auto &p : loc)
        if (p.first == i || p.second == j || abs(p.first - i) == abs(p.second - j))
          return false;
      return true;
    };
    if (i == n) {
      cnt++;
      return;
    }
    for (int j = 0; j < n; j++) {
      if (!isValid(i, j))
        continue;
      b[i][j] = 1;
      loc.push_back(make_pair(i, j));
      self(self, i+1);
      b[i][j] = 0;
      loc.pop_back();
    }
  };
  dfs(dfs, 0);
  ofs << cnt << endl;
  return 0;
}
```

## 第六题

### 题目描述

字符串匹配问题：在`6.in`中有两个字符串，在`6.out`输出第二个字符串在第一个字符串中的起始和终止位置，如果没有则输出0。

```
Sample input:abcdefgdrefege
cdef
Sample output:3 6
```

### 实现思路

使用`kmp`算法

### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

int kmp(string &main, string &sub) {
  int len1 = main.length(), len2 = sub.length();
  // getNext
  vector<int> next(len2, 0);
  for (int i = 1; i < len2; i++) {
    if (sub[i] == sub[next[i-1]])
      next[i] = next[i-1]+1;
    else if (sub[i] == sub[0])
      next[i] = 1;
    else
      next[i] = 0;
  }
  for (int i = len2-1; i >= 1; i--) {
    next[i] = next[i-1];
  }
  next[0] = -1;
  int i = 0, j = 0;
  while (i < len1 && j < len2) {
    if (main[i] == sub[j]) {
      i++;
      j++;
    } else {
      if (next[j] == -1)
        i++;
      else
        j = next[j];
    }
  }
  if (j == len2)
    return i-j;
  return -1;
}

int main(int argc, char *argv[]) {
  ifstream ifs("./6.in");
  ofstream ofs("./6.out");
  string s1, s2;
  getline(ifs, s1);
  getline(ifs, s2);
  int ans = kmp(s1, s2);
  if (ans == -1) {
    ofs << 0 << endl;
  } else {
    ofs << ans+1 << " " << ans+s2.length() << endl;
  }
  return 0;
}
```

## 第七题

### 题目描述

哈弗曼树问题：在`7.in`中有几个数，和为1，进行哈弗曼编码，并把编码结果输出到`7.out`中。

```
Sample input: 0.1 0.15 0.2 0.25 0.3
Sample output:
000
001
01
10
11
```

### 实现

充分利用C++中的优先队列，作为实现huffman算法的基本数据结构。每次选取两个值最小的结点来构成一个新的结点，将其插入优先队列，循环这一过程直到队列中只有一个结点。

### 代码

```cpp
#include <fstream>
#include <queue>
#include <unordered_map>
#include <vector>
using namespace std;

struct TreeNode {
  TreeNode *lchild, *rchild;
  float val;
  TreeNode(float val) : lchild(nullptr), rchild(nullptr), val(val) {}
};

struct Cmp {
  bool operator()(TreeNode *l, TreeNode *r) {
    return l->val > r->val;
  }
};

unordered_map<TreeNode *, int> idxMap;
vector<string> ans;
string tmp;

void preOrder(TreeNode *root) {
  if (!root)
    return;
  if (!root->lchild && !root->rchild) {
    ans[idxMap[root]] = tmp;
    return;
  }
  tmp += "0";
  preOrder(root->lchild);
  tmp.pop_back();
  tmp += "1";
  preOrder(root->rchild);
  tmp.pop_back();
}

int main(int argc, char *argv[]) {
  ifstream ifs("./7.in");
  ofstream ofs("./7.out");
  float num;
  priority_queue<TreeNode *, vector<TreeNode *>, Cmp> pq;
  int idx = 0;
  while (ifs >> num) {
    TreeNode *t = new TreeNode(num);
    pq.push(t);
    idxMap[t] = idx++;
  }
  ans.resize(pq.size());
  while (pq.size() != 1) {
    TreeNode *t1 = pq.top();
    pq.pop();
    TreeNode *t2 = pq.top();
    pq.pop();
    TreeNode *t = new TreeNode(t1->val + t2->val);
    t->lchild = t1;
    t->rchild = t2;
    pq.push(t);
  }
  TreeNode *root = pq.top();
  preOrder(root);
  for (string &s : ans) {
    ofs << s << endl;
  }
  return 0;
}
```
