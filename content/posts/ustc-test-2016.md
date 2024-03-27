---
title: 中科大2016年复试机试题详解
date: 2021-05-16 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

设计程序，讲用户输入的十进制整数转换为十六进制数，并在屏幕上输出。

### 代码

```cpp
#include <iostream>
using namespace std;

char hexchar[] = {'0', '1',  '2', '3', '4', '5', '6', '7',
                  '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

int main(int argc, char *argv[]) {
  int num;
  cin >> num;
  int st[32];
  int i = 0;
  while (num) {
    st[i++] = num % 16;
    num /= 16;
  }
  while (--i >= 0) {
    cout << hexchar[st[i]];
  }
  return 0;
}
```

## 第二题

### 题目描述

编程实现一个简单的我微信红包分配算法。满足以下要求即可：

1. 键盘输入红包金额X元，以及红包个数N。X是正数，例如10.88，表示10.88元。N是正整数。
2. 每个红包的金额至少是一分钱。如果总金额X无法满足这一条件，则提示重新输入。
3. 在红包金额固定的条件下，单个红包的金额随机生成。
4. 在屏幕上输出红包分配结果。

```
输入示例
100 5
输出示例
10.1  20  30.2  9.9  29.8
```

### 解题思路

+ 为了方便我们将红包金额转换为以分为单位的整数，比如10.88元被转换为1088分钱。然后问题就转换为了如何将一个整数X随机分为N份，我们可以将整数X类看作长度为X的带子，在带子中间随机地切N-1刀，就可以获得N个随机长度的带子了。
+ 我们用map来存储切割的位置，因为其键值具有有序性，方便输出结果。

### 代码

```cpp
#include <iostream>
#include <map>
using namespace std;

int main(int argc, char *argv[]) {
  float X;
  int N;
  cin >> X >> N;
  int Y = X * 100;
  map<int, int> occ;
  occ[0] = occ[Y] = 1;
  for (int i = 0; i < N - 1; i++) {
    int div;
    do {
      div = rand() % (Y - 1) + 1;
    } while (occ.count(div));
    occ[div] = 1;
  }
  auto prev = occ.begin();
  for (auto it = next(occ.begin()); it != occ.end(); it++) {
    printf("%.2f ", (float)(it->first - prev->first) / 100);
    prev++;
  }
  return 0;
}
```
感觉用map没有必要，写了一个没有map的解法
```cpp
//
// Created by liu'bo'yan on 2024/3/22.
//
#include <bits/stdc++.h>

using namespace std;
int main(){
    double x;
    int n;
    scanf("%lf",&x);
    int x_100=100*x;
    cin>>n;
    if(x_100<n){
        cout<<"please reenter!";
        return 0;
    }
    int v[n];
    for (int i = 0; i < n-1; ++i) {
        int r=0;
        while (r==0){
            r=rand()%(x_100-1);
        }
        v[i]=r;
        x_100-=r;
    }
    v[n-1]=x_100;
    for (int i = 0; i < n; ++i){
        printf("%.2f ",v[i]/100.0);
    }
}
```
## 第三题

### 题目描述

求多项式的根`2x^11 - 3x^8 - 5x^3 - 1 = 0`，根的精度为0.00000001

### 解题思路

用`二分法`求解，令`f(x) = 2x^11 - 3x^8 - 5x^3 - 1`观察到`f(0) < 0`，`f(2) > 0`，可选初始结果范围为`(0, 2)`

### 代码

```cpp
#include <iostream>
#include <cmath>
using namespace std;

double func(double x) {
  return 2 * pow(x, 11) - 3 * pow(x, 8) - 5 * pow(x, 3) - 1;
}

double solve(double (*func) (double), double l, double r) {
  if (r - l < 0.00000001)
    return l;
  double mid = (r + l) / 2;
  double tmp = func(mid);
  if (tmp > 0) return solve(func, l, mid);
  else if (tmp < 0) return solve(func, mid, r);
  else return mid;
}

int main(int argc, char *argv[]) {
  printf("%.9f\n", solve(func, 1, 2));
  return 0;
}
```
将二分写在函数内的写法
```cpp
//
// Created by liu'bo'yan on 2024/3/22.
//
#include <bits/stdc++.h>

using namespace std;
double eps=0.00000001;
double f(double x){
    return 2*pow(x,11)-3*pow(x,8)-5*pow(x,3)-1;
}
int main(){
    double l=0.0,r=2.0,mid;
    while (r-l>eps){
        mid=(l+r)/2.0;
        if(f(mid)>0){
            r=mid;
        }else{
            l=mid;
        }
    }
    printf("%f",l);
}
```
## 第四题

### 题目描述

给定一个二叉树的前序遍历和后序遍历，给出一种可能的中序遍历结果。
输入从文件`4.in`中给定。其中第一行是二叉树结点的个数，第二行是二叉树的前序遍历序列，第三行是后序遍历序列。二叉树种的结点名称以大写字母表示，最多26个结点。
将结果输出到文件`4.out`，输出一种可能的中序遍历结果。

```
输入示例
5
A B D C E
D B E C A
输出示例
D B A E C
```

### 解题思路

见[LeetCode第889题](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

ifstream ifs("./4.in");
ofstream ofs("./4.out");

struct TreeNode {
  TreeNode *lchild, *rchild;
  char val;
  TreeNode(char val) : lchild(nullptr), rchild(nullptr), val(val) {}
};

void inOrder(TreeNode *t) {
  if (!t) return;
  inOrder(t->lchild);
  ofs << t->val << " ";
  inOrder(t->rchild);
}

TreeNode *buildTree(vector<char> &pre, vector<char> &post) {
  auto make = [&](auto &&self, int l1, int r1, int l2, int r2) -> TreeNode * {
    if (l1 > r1) return nullptr;
    TreeNode *t = new TreeNode(pre[l1]);
    if (l1 != r1) {
      int i;
      for (i = l2; i <= r2; i++)
        if (pre[l1 + 1] == post[i]) break;
      t->lchild = self(self, l1 + 1, l1 + 1 + i - l2, l2, i);
      t->rchild = self(self, l1 + 2 + i - l2, r1, i + 1, r2 - 1);
    }
    return t;
  };
  return make(make, 0, pre.size() - 1, 0, post.size() - 1);
}

int main(int argc, char *argv[]) {
  int n;
  ifs >> n;
  vector<char> pre(n), post(n);
  for (int i = 0; i < n; i++) {
    ifs >> pre[i];
  }
  for (int i = 0; i < n; i++) {
    ifs >> post[i];
  }
  TreeNode *root = buildTree(pre, post);
  inOrder(root);
  return 0;
}
```
