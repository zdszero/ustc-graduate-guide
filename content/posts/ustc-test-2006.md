---
title: 中科大2006年复试机试题详解
date: 2021-05-06 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

求矩阵的转置。

## 第二题

### 题目描述

写一个程序，判断C语言中的变量命名是否合法。

### 代码

```cpp
#include <iostream>
using namespace std;

// [_a-zA-Z][_a-zA-Z0-9]*
bool isValid(const string &s) {
  int i = 0;
  if (!(isalpha(s[i]) || s[i] == '_'))
    return false;
  for (++i; i < s.length(); i++) {
    if (!(isalpha(s[i]) || isdigit(s[i]) || s[i] == '_'))
      return false;
  }
  return true;
}

int main(int argc, char *argv[]) { 
  string s;
  cin >> s;
  cout << isValid(s) << endl;
  return 0;
}
```

## 第三题

### 题目描述

编写一程序，将中缀表达式转化为后缀表达式（含括号情况）。从`expr.in`文件中读取输入，将结果输出到`expr.out`文件中。

```
输入示例：
a-b*(c+d)
输出示例：
abcd+*-
```

### 解题思路

即将中缀表达式转换为后缀表达式。只使用一个栈来存储算符，并且用一个表来存储算符的优先级。当遇到运算数时直接输出，遇到算符时根据规则压入栈或者输出。

### 代码

```cpp
#include <fstream>
#include <stack>
#include <unordered_map>
#include <vector>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./expr.in");
  ofstream ofs("./expr.out");
  vector<char> v;
  stack<char> opStk;
  unordered_map<char, int> priority;
  priority['('] = 0;
  priority['+'] = priority['-'] = 1;
  priority['*'] = priority['/'] = 2;
  char c;
  while (ifs >> c)
    v.push_back(c);
  for (char c : v) {
    if (isalpha(c)) {
      ofs << c;
      continue;
    }
    if (c == '(') {
      opStk.push('(');
      continue;
    }
    if (c == ')') {
      char ch;
      while (1) {
        ch = opStk.top();
        opStk.pop();
        if (ch == '(')
          break;
        ofs << ch;
      };
      continue;
    }
    while (!opStk.empty() && priority[c] < priority[opStk.top()]) {
      char op = opStk.top();
      opStk.pop();
      ofs << op;
    }
    opStk.push(c);
  }
  while (!opStk.empty()) {
    char ch = opStk.top();
    opStk.pop();
    ofs << ch;
  }
  return 0;
}
```

## 第四题

### 题目描述

给定两个数m和n，实现如下功能：如m=3，n=4时，输出

```
1 2 3
1 2 4
1 3 4
2 3 4
```

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int m, n;
  cin >> m >> n;
  for (int i = 1; i <= n-2; i++) {
    for (int j = i+1; j <= n-1; j++) {
      for (int k = j+1; k <= n; k++) {
        printf("%d %d %d\n", i, j, k);
      }
    }
  }
  return 0;
}
```

## 第五题

### 题目描述

给定一无向图的矩阵存储，求其最大连通分量。图的信息存储在文件`graph.in`中，文件的第一行给无向图的顶点数量n和边数k，顶点序号从0到n-1，接下来分别给出k条边的两个顶点号。输出无向图中的最大连通分量的所有顶点号，输出到文件`graph.out`中。

```
输入示例:
10 6
0 1
0 2
4 5
4 6
5 7
8 9

输出示例:
4 5 7 6

```
### 解题思路

设置`visit`数组表标记顶点是否访问过，然后按照顶点序号从小到大的顺序用`dfs`来遍历所有的连通分量，并且保存最大的连通分量中的所有顶点。

### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

int dfs(vector<vector<int>> &g, vector<int> &vis, vector<int> &tmp, int idx,
        int n) {
  if (vis[idx])
    return 0;
  vis[idx] = 1;
  tmp.push_back(idx);
  int ans = 1;
  for (int i = 0; i < n; i++) {
    if (g[idx][i]) {
      ans += dfs(g, vis, tmp, i, n);
    }
  }
  return ans;
}

int main(int argc, char *argv[]) {
  ifstream ifs("./graph.in");
  ofstream ofs("./graph.out");
  int n, m;
  ifs >> n >> m;
  vector<vector<int>> g(n, vector<int>(n, 0));
  vector<int> vis(n, 0);
  vector<int> ans, tmp;
  int maxval = 0;
  for (int i = 0; i < m; i++) {
    int a, b;
    ifs >> a >> b;
    g[a][b] = g[b][a] = 1;
  }
  for (int i = 0; i < n; i++) {
    if (vis[i])
      continue;
    int tmpval = dfs(g, vis, tmp, i, n);
    if (tmpval > maxval) {
      maxval = tmpval;
      ans = tmp;
    }
    tmp.clear();
  }
  for (int i = 0; i < ans.size(); i++) {
    if (i)
      ofs << " ";
    ofs << ans[i];
  }
  return 0;
}
```

## 第六题

### 题目描述

求连续子数组的最大和，比如[-2,1,-3,4,-1,2,1,-5,4]中的连续子数组[4,-1,2,1]的和最大，为6。数组中的数字保存在文件`number.in`中，求出结果输出到文件`number.out`中。

```
输入示例:
-2 1 -3 4 -1 2 1 -5 4

输出示例:
6
```

### 解题思路

这是一道动态规划的题目，具体解析可以看[这篇文章](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)。

设数组为`nums`，动态规划列表为`dp`，则`dp[i]`表示**以第i个元素结尾的子数组**的最大和，`dp`满足如下递归公式：

<div>$$
dp[i] = \begin{cases}
max\{dp[i-1] + nums[i], 0\},\ i \gt 0 \\
nums[i],\ i = 0
\end{cases}
$$</div>

为了进一步降低空间复杂度，我们可以直接用nums数组代替dp数组，以下的代码实现就采用了这种方式。

### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./number.in");
  ofstream ofs("./number.out");
  vector<int> nums;
  int num;
  while (ifs >> num) nums.push_back(num);
  int ans = nums[0];
  for (int i = 1; i < nums.size(); i++) {
    nums[i] += max(nums[i-1], 0);
    ans = max(ans, nums[i]);
  }
  ofs << ans;
}
```
