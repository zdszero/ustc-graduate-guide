---
title: 中科大2014年复试机试题详解
date: 2021-05-14 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

从标准输入读取一个字符串，然后统计字符串中有多少个字母、数字、空格和其他字符。

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  string s;
  cin >> s;
  int digitCnt = 0, alphaCnt = 0, spaceCnt = 0, elseCnt = 0;
  for (char c : s) {
    if (isdigit(c))
      digitCnt++;
    else if (isalpha(c))
      alphaCnt++;
    else if (c == ' ')
      spaceCnt++;
    else
      elseCnt++;
  }
  printf("数字: %d\n", digitCnt);
  printf("字母: %d\n", alphaCnt);
  printf("空格: %d\n", spaceCnt);
  printf("其他字符: %d\n", elseCnt);
  return 0;
}
```

## 第二题

### 题目描述

给你一个十进制数，输出相应的八进制数。经典的进制转换，考过几次了。

## 第三题

### 题目描述

给你一个数，要求你求出这个数与其反序数的和相加多少次才可以得到回文数。

```
给你 1568 
 
1568+8651=10219，不是回文字，继续， 
 
10219+91201=101410，不是回文字，继续， 
 
101410+014101=115511，是回文字，结束，输出 3； 
```

### 解题思路

这题需要处理的点在于如何将数字反序并相加，一种比较简单的方式就是用`string`类型判断是否是回文数，用`int`类型将数字相加，我们可以用`stoi(value)`和`string(number)`在字符串和整数之间横跳。

笔者在这里采用的方式实现字符串表示数字的加法，统一用字符串处理。

### 代码

```cpp
#include <iostream>
using namespace std;

bool isPalindrome(const string &s) {
  int i = 0, j = s.size()-1;
  while (i < j)
    if (s[i++] != s[j--])
      return false;
  return true;
}

string add(const string &s1, const string &s2) {
  int maxlen = max(s1.length(), s2.length());
  string ans(maxlen, '0');
  int i = s1.length()-1, j = s2.length()-1, k = maxlen-1;
  int carry = 0;
  while (i >= 0 && j >= 0) {
    int sum = s1[i--] - '0' + s2[j--] - '0' + carry;
    carry = sum / 10;
    ans[k--] = (sum % 10) + '0';
  }
  while (i >= 0) {
    int sum = s1[i] - '0' + carry;
    carry = sum / 10;
    ans[i--] = (sum % 10) + '0';
  }
  while (j >= 0) {
    int sum = s2[j] - '0' + carry;
    carry = sum / 10;
    ans[j--] = (sum % 10) + '0';
  }
  if (carry == 1)
    ans = "1" + ans;
  return ans;
}

string reverse(const string &s) {
  return string(s.rbegin(), s.rend());
}

int main(int argc, char *argv[]) {
  string s;
  cin >> s;
  int cnt = 0;
  while (!isPalindrome(s)) {
    s = add(s, reverse(s));
    cnt++;
  }
  cout << cnt;
  return 0;
}
```

## 第四题

### 题目描述

 
读取一个文件`graph.in`，做无向图的广度遍历输出，文件的第一行是图的结点个数，后面是边的信息，`0 0`表示结束，结点的编号从1开始，将广度遍历序列输出到控制台。

```
输入示例：
4
1 2
1 4
2 3
2 4
3 4
0 0

输出示例：
1 2 4 3
```

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./graph.in");
  int n;
  ifs >> n;
  vector<vector<int>> g(n + 1, vector<int>(n + 1, 0));
  vector<int> vis(n + 1, 0);
  queue<int> q;
  int s, d;
  while (ifs >> s >> d && s != 0 && d != 0) {
    g[s][d] = g[d][s] = 1;
  }
  q.push(1);
  vis[1] = 1;
  int flag = 0;
  while (!q.empty()) {
    int tmp = q.front();
    q.pop();
    if (flag) cout << " ";
    flag = 1;
    cout << tmp;
    for (int i = 1; i <= n; i++) {
      if (g[tmp][i] && !vis[i]) {
        q.push(i);
        vis[i] = 1;
      }
    }
  }
  return 0;
}
```
邻接表表示的图的写法
```cpp
//
// Created by liu'bo'yan on 2024/3/22.
//
#include <bits/stdc++.h>

using namespace std;
int main(){
    ifstream ifs("graph.in");
    int n,x,y;
    ifs>>n;
    vector<vector<int>> g(n+1,vector<int>());
    while (ifs>>x>>y){
        if(x==0||y==0)break;
        g[x].emplace_back(y);
        g[y].emplace_back(x);
    }
    queue<int> q;
    int vis[n+1];
    memset(vis,0, sizeof(vis));
    q.push(1);
    vis[1]=1;
    while (!q.empty()){
        int i=q.front();
        q.pop();
        cout<<i<<" ";
        vis[i]=1;
        for(int j:g[i]){
            if(!vis[j]){
                q.push(j);
                vis[j]=1;
            }
        }
    }
}

```
