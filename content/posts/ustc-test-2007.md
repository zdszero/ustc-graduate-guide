---
title: 中科大2007年复试机试题详解
date: 2021-05-07 12:00:00
tags: ["中科大复试"]
draft: true
---

## 题目一

### 题目描述

编写程序，判断给定数字是否是回文数，如输入：123，输出：Y。输入：234，输出:N。

### 代码

```cpp
#include <iostream>
using namespace std;

bool isPalindrome(const string &s) {
  int i = 0, j = s.length()-1;
  while (i < j) {
    if (s[i++] != s[j--])
      return false;
  }
  return true;
}

int main(int argc, char *argv[]) {
  string tmp;
  while (cin >> tmp) {
    if (isPalindrome(tmp))
      cout << "Y" << endl;
    else
      cout << "N" << endl;
  }
  return 0;
}
```

## 题目二 

### 题目描述

队列的循环报数问题：设有n个人站成一排，从左往右的编号分别为1~n，现在从左往右报数“1,2,1,2....”，数到“1”的人出列，数到“2”的人立即站到队伍的最右端。报数过程反复进行，直到n个人都出列为止。要求给出它们的出列顺序。

n从键盘输入，出列顺序输出到控制台。

```
输入示例：
10
输出示例：
1 3 5 7 9 2 6 10 8 4
```

### 代码

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main(int argc, char *argv[]) {
  int n;
  cin >> n;
  queue<int> q;
  for (int i = 1; i <= n; i++) q.push(i);
  int cnt = 1, flag = 0;
  while (!q.empty()) {
    if (cnt == 1) {
      if (flag) cout << " ";
      flag = 1;
      cout << q.front();
      q.pop();
      cnt = 2;
    } else {
      q.push(q.front());
      q.pop();
      cnt = 1;
    }
  }
  return 0;
}
```

## 题目三

### 题目描述

无向图的最小生成树：输入在文件`3.in`中给出，描述了图的形状。首先给出图中结点的总数n，结点编号从0到n-1，然后接下来每一行给出边的信息，每行包含三个数字，分别是两个顶点的编号以及边长。要求出无向图对应的最小生成树，将结果输出到文件`3.out`中。

```
输入示例:
9
7      6    1 
8      2    2 
6      5    2 
0      1    4 
2      5    4 
8      6    6 
2      3    7 
7      8    7 
0      7    8 
1      2    8 
3      4    9 
5      4    10
1      7    11
3      5    14

输出示例:
7 -- 6 : 1
6 -- 5 : 2
8 -- 2 : 2
2 -- 5 : 4
0 -- 1 : 4
2 -- 3 : 7
1 -- 2 : 8
3 -- 4 : 9
```

### 解题思路

求无向图的最小生成树常有两种算法：`prim`和`kruskal`，建议熟练掌握`kruskal`算法的实现，相比`prim`算法思路更为简单，在考场上更加容易实现。

`kruskal`算法的基本过程是从无向图中依次选取边长最短的边加入最小生成树，如果会导致连通的话就放弃这条边。如果无向图的顶点数为n，那么选择n-1条符合条件的边即停止。这里的难点在于判断加入一条新的边后是否会导致连通，我们用subset[n]数组来表示每个边所在的子集序号，`subset[i] == 0`表示顶点i不在任何子集中。如果新加入的边的两个顶点序号为i、j，那么会出现如下几种情况：
+ subset[i] == subset[j] == 0，说明两个顶点不在其他子集中，加入新的子集
+ subset[i] != 0 && subset[j] != 0，说明两个顶点都存在于子集中
    + subset[i] == subset[j]，两个顶点存在于相同的子集中，加入的边会导致连通，应该舍弃
    + subset[i] != subset[j]，两个顶点存在与不同的子集中，合并两个不同的子集
+ ((subset[i] != 0 && subset[j] == 0) || (subset[i] == 0 && subset[j] != 0))，将不在任何子集中的顶点加入另一个顶点的子集

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct Edge {
  int src, dst, weight;
  Edge(int src, int dst, int weight) : src(src), dst(dst), weight(weight) {}
};

struct Cmp {
  bool operator()(const Edge &lhs, const Edge &rhs) {
    return lhs.weight > rhs.weight;
  }
};

int main(int argc, char *argv[]) {
  ifstream ifs("./3.in");
  int n;
  ifs >> n;
  vector<int> subset(n+1, 0);
  int MaxIdx = 1;
  priority_queue<Edge, vector<Edge>, Cmp> edges;
  vector<Edge> mst;
  int a, b, c;
  while (ifs >> a >> b >> c) {
    edges.push(Edge(a, b, c));
  }
  while (!edges.empty() && mst.size() < n-1) {
    Edge e = edges.top();
    edges.pop();
    int set1 = subset[e.src], set2 = subset[e.dst];
    if (!set1 && !set2) {
      subset[e.src] = subset[e.dst] = MaxIdx++;
    } else if (set1 && set2) {
      if (set1 == set2)
        continue;
      for (auto &s : subset)
        if (s == set2)
          s = set1;
    } else {
      if (!set1) subset[e.src] = set2;
      if (!set2) subset[e.dst] = set1;
    }
    mst.push_back(e);
  }
  for (auto &e : mst) {
    printf("%d -- %d : %d\n", e.src, e.dst, e.weight);
  }
  return 0;
}
```

## 题目四 

### 题目描述

中序后序得前序：输入在文件`4.in`中给出，首先给出二叉树中的顶点个数`n`，然后在接下来两行给出中序和后序序列，要求根据中序和后序序列构建二叉树，并且将二叉树的前序遍历序列输出到文件`4.out`中。也可以思考中序前序得后序。

```
输入示例:
5
D B A C E
D B E C A

输出示例:
D B A C E
```


### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

int flag = 0;

struct TreeNode {
  TreeNode *lchild, *rchild;
  char val;
  TreeNode(char val) : val(val) {}
};

TreeNode *buildTree(vector<char> &in, vector<char> &post) {
  auto make = [&](auto &&self, int l1, int r1, int l2, int r2) -> TreeNode *{
    if (l1 > r1)
      return nullptr;
    TreeNode *t = new TreeNode(post[r2]);
    int i;
    for (i = l1; i <= r1; i++)
      if (in[i] == post[r2])
        break;
    t->lchild = self(self, l1, i-1, l2, l2+i-1-l1);
    t->rchild = self(self, i+1, r1, l2+i-l1, r2-1);
    return t;
  };
  return make(make, 0, in.size()-1, 0, post.size()-1);
}

void inOrder(TreeNode *t, ostream &os) {
  if (!t)
    return;
  inOrder(t->lchild, os);
  if (flag) os << " ";
  os << t->val;
  flag = 1;
  inOrder(t->rchild, os);
}

int main(int argc, char *argv[]) {
  ifstream ifs("./4.in");
  ofstream ofs("./4.out");
  int n;
  ifs >> n;
  vector<char> in(n);
  vector<char> post(n);
  for (int i = 0; i < n; i++) ifs >> in[i];
  for (int i = 0; i < n; i++) ifs >> post[i];
  TreeNode *root = buildTree(in, post);
  inOrder(root, ofs);
  return 0;
}
```
