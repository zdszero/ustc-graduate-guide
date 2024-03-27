---
title: 中科大2017年复试机试题详解
date: 2021-05-17 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

文本文件`input1.txt`中含有一行字符串（字符串长度不大于1000），由若干个英文单词构成，单词之间以一个或多个空格分割，请将该字符串中单词之间多余的空格删除，即单词之间只保留一个空格，字符串首尾的空格也删除，并将所有单词的首字母改成大写之后输出到屏幕。
```
输入
i  am  very nervous today      since i   will attend the C   programming
输出
I Am Very Nervous Today Since I Will Attend The C Programming
```

### 代码

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main(int argc, char *argv[]) {
  string tmp;
  int flag = 0;
  ifstream ifs("./input1.txt");
  while (ifs >> tmp) {
    if (flag) {
      cout << " ";
    }
    flag = 1;
    tmp[0] = toupper(tmp[0]);
    cout << tmp;
  }
  return 0;
}
```

## 第二题

### 题目描述

一个数如果等于它所有的因子（除了自己本身）之和，比如`6=3+2+1`，则称其为“完数”；若因子之和大于该数，则称其为“盈数”。在屏幕上输出2到1000之间的所有“盈书”和“完数”。

### 解题思路

将判断是否是素数的代码稍作修改，即可得到本题的答案，注意数字本身不能作为自己的因子。

```cpp
/* 判断是否是素数 */
bool isOdd(int num) {
  for (int i = 2; i <= sqrt(num); i++)
    if (num % i == 0)
      return true;
  return false;
}
```

### 代码

```cpp
#include <iostream>
#include <vector>
#include <cmath>
using namespace std;

bool isOk(int num) {
  int sum = 1, tmp = num, i = 2;
  while (i <= sqrt(tmp)) {
    if (tmp % i != 0) {
      i++;
      continue;
    }
    tmp /= i;
    sum += i;
    i = 2;
  }
  if (tmp != num)
    sum += tmp;
  return sum >= num;
}

int main(int argc, char *argv[]) {
  int flag = 0;
  for (int i = 2; i <= 1000; i++) {
    if (isOk(i)) {
      if (flag) cout << " ";
      cout << i;
      flag = 1;
    }
  }
  cout << endl;
  return 0;
}
```

## 第三题

### 题目描述

文本文件`input3.txt`中有一个有序的正整数序列，以空格作为间隔符。计算该序列中包含的最长等差子序列的长度，并输入，例如

```
输入：1 2 3 5 7 8 14 20 26
最长等差子序列：2 8 14 20 26
输出：5
```

### 解题思路

本题是一个动态规划的问题，有暴力法和dp数组优化两种方法。

暴力法使用一个三重循环得到答案，时间复杂为`O(n^3)`，这里不过多介绍，相信大家都可以写出来。

该题符合动态规划的两个特征：`overlapping subproblem`和`optimal substructure`，所以可以使用动态规划解决问题。动态规划一般是使用一个一维数组
或二维数组来保存子问题的状态，但是由于本题等差子数列的差未知，并且题目没有给出差的范围，所以为了写出一个一般解，我们用`map<int, vector<int>> dp`
来替代一般的二维数组，为了更好地说明`dp`的含义，我们举个实际的例子。比如`dp[6][8] = 4`指的是差为6，并且以第8个元素（在上述输入中为3）为结尾的
等差子序列的长度为4。

总结：在没有做过该题的情况下，考场上能想出来动态规划还是挺难的，建议直接暴力法，也可以得到不少分。另外看到这篇文章但对动态规划不太熟悉的可以
看看`geeksforgeeks`网站中的这个系列[dynamic programming](https://www.geeksforgeeks.org/dynamic-programming/)，三哥写的教程，质量很高。

### 代码

```cpp
#include <iostream>
#include <fstream>
#include <vector>
#include <map>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./input3.txt");
  int num;
  vector<int> v(1, 0);
  while (ifs >> num)
    v.push_back(num);
  int n = v.size() - 1;
  int maxv = 0;
  map<int, vector<int>> dp;
  for (int i = 1; i <= n; i++) {
    for (int j = 0; j < i; j++) {
      int diff = v[i] - v[j];
      if (!dp.count(diff))
        dp[diff] = vector<int>(n+1, 1);
      int tmp = dp[diff][j] + 1;
      dp[diff][i] = tmp;
      if (tmp > maxv)
        maxv = tmp;
    }
  }
  cout << maxv << endl;
  return 0;
}
```
写了一个比较刁钻的dp,状态转移的方程的核心，就是能否与之前的构成等差序列，或者从他这里再开一个新的等差序列。不知道是否是对的，能过他给的样例，但不知道是否具有泛化性。
```cpp
//
// Created by liu'bo'yan on 2024/3/21.
//
#include <bits/stdc++.h>

using namespace std;
int main(){
    int n=0;
    int a[1000];
    ifstream ifs("2017input3.txt");
    while (ifs>>a[n++]);
    int dp[n+1];
    memset(dp,0, sizeof(dp));
    dp[0]=1;
    dp[1]=2;
    for (int i = 2; i <n; ++i){
        dp[i+1]=max(dp[i]+((a[i-1]-a[i-2])==(a[i]-a[i-1])),2);
    }
    cout<<dp[n];
}
```
## 第四题

### 题目描述

在文本文件`input4.txt`中给定无向连通图G，判断图G是否存在这样一个顶点V，当V被删除时，该图的其他部分不再连通。如果存在，只需要找出一个这样的顶点，并输出顶点的编号，
如果不存在，则输出"not exist"。

```
输入：
5   // 表示顶点个数
1 5 // 节点1和5之间有一条边
2 3
2 4
3 5
4 5
输入数据对应的图如下所示：
       4
      / \
1 -- 5   2
      \ /
       3
输出：
5
```

### 解题思路

1. 一般做题时为了处理方便，图都尽量用邻接矩阵存储，首先我们读取输入建立矩阵，这个过程我们不赘述。
2. 接下来的关键就在于找出方法判断`去除某个顶点后图是否仍然连通`，我们不如将这个问题先简化，想一想如何判断`某个图是否连通`，
这个算法应该比较容易想到：用`visit`数组记录每个顶点是否访问过，然后用一个`dfs`遍历无向图，如果所有顶点都访问过，即`visit`
数组中的所有元素都是`1`，则说明图为连通图，代码如下所示。接下来我们再考虑一下删除某个顶点`idx`后如何判断图是否仍然连通，首先我们
将`visit[idx] = 1`表示该顶点已经访问，然后再从这个顶点的下一个执行`dfs`遍历图，再判断所有顶点是否都访问过。
3. 注意在代码中如果多个定义全局函数来解决问题的话，可能要传递比较多的参数，所以我们在代码中用`lambda表达式`在主函数中建立函数闭包，
解题方法更容易编写，如果对于C++这个特性不太熟悉的同学可以去google一下。

```cpp
// 判断图是否连通
bool isConnected(vector<vector<int>> &g) {
  vector<int> vis(0, g.size());
  dfs(g, vis, 0);
  for (int i = 1; i <= n; i++)
    if (!vis[i])
      return false;
  return true;
}

void dfs(vector<vector<int>> &g, vector<int> &vis, int idx) {
  vis[idx] = 1;
  for (int i = 1; i <= n; i++)
    if (g[idx][i] && !vis[i])
      dfs(g, vis, i);
}
```

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <vector>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./input4.txt");
  int n;
  ifs >> n;
  vector<vector<int>> g(n + 1, vector<int>(n + 1, 0));
  int a, b;
  while (ifs >> a >> b) {
    g[a][b] = g[b][a] = 1;
  }
  auto keyVertex = [&]() -> int {
    auto isConnected = [&](int idx) -> bool {
      vector<int> vis(n + 1, 0);
      vis[idx] = 1;
      auto dfs = [&](auto &&self, int idx) -> void {
        vis[idx] = 1;
        for (int i = 1; i <= n; i++)
          if (g[idx][i] && !vis[i])
            self(self, i);
      };
      dfs(dfs, idx == n ? 1 : idx + 1);
      for (int i = 1; i <= n; i++)
        if (!vis[i])
          return false;
      return true;
    };
    for (int i = 1; i <= n; i++)
      if (!isConnected(i))
        return i;
    return 0;
  };
  int key = keyVertex();
  if (key)
    cout << key;
  else
    cout << "not exist";
  return 0;
}
```
