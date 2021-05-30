---
title: 中科大2018复试机试题详解
date: 2021-05-18 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

2000年1月1日是星期六，现在给你任意一个天数n，代表其距离2000年1月1日有n天，让你求该天是几年几月几日，星期几。比如n=2，输出“2000-1-3 Monday”。（记得注意闰年有366天，而且二月有29天）

### 代码

```cpp
#include <iostream>
using namespace std;

int mcnt[] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
int ycnt[] = {366, 365, 365, 365};
string dname[] = {"Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Firaday"};

int main(int argc, char *argv[]) {
  int n;
  cin >> n;
  int tmp = n % 7;
  int year = 2000, month = 1, date = 1;
  int i = 0;
  while (n >= ycnt[i%4]) {
    n -= ycnt[i%4];
    year++;
    i++;
  }
  i = 0;
  if (year % 4 == 0)
    mcnt[1] = 29;
  while (n >= mcnt[i]) {
    n -= mcnt[i];
    month++;
    i++;
  }
  date += n;
  printf("%d-%d-%d %s", year, month, date, dname[tmp].c_str());
  return 0;
}
```

## 第二题

### 题目描述

从文件input.txt中读入一个m，表示有m个签到记录。记录每个人的上机时间，求出上机时间最长的人。若签退时间小于签到时间，则说明跨天。且一人可以多次签到，但多次签到的时间没有重合。输入包括学号 签到时间 签退时间

```
input:
3
SA0010001 13:00 16:39
SA0010101 07:22 22:01
SA0010111 12:00 11:56
```

### 解题思路

+ 将每天的时间按照按照分钟计算，从`0`到`24*60`，方便计算
+ 考虑到每个人每天可以多次签到，我们用`map`来记录每个人一天的上机时间

### 代码

```cpp
#include <iostream>
#include <unordered_map>
using namespace std;

int getGap(int a, int b, int c, int d) { return a * 60 + b - c * 60 - d; }

int main(int argc, char *argv[]) {
  int n;
  cin >> n;
  int longest = 0;
  unordered_map<string, int> timeTrack;
  string id, ans;
  /* start hour, start minite, end hour, end minute */
  int sh, sm, eh, em;
  for (int i = 0; i < n; i++) {
    cin >> id;
    scanf("%d:%d", &sh, &sm);
    scanf("%d:%d", &eh, &em);
    int tmp;
    // the next day
    if (eh <= sh && em <= sm) {
      tmp = 24 * 60 - getGap(eh, em, sh, sm);
    } else {
      tmp = getGap(sh, sm, eh, em);
    }
    timeTrack[id] += tmp;
  }
  for (auto it = timeTrack.begin(); it != timeTrack.end(); it++) {
    if (it->second > longest) {
      longest = it->second;
      ans = it->first;
    }
  }
  cout << ans << endl;
  return 0;
}
```

## 第三题

### 题目描述

给一个m，n，接下来给出n个正整数表示邮票的面值，问可凑够面值m所需的最小邮票数，无法满足则输出0

```
input：
10 3
1 2 4
output：
3
```

### 解题思路

用一个dfs来遍历所有的选择，注意应该优先从数值较大的邮票开始选择（这样可以使凑够目标面值所需的邮票数较少），如果遍历完所有的情况都不能满足则返回-1

### 代码

```cpp
#include <algorithm>
#include <fstream>
#include <iostream>
#include <vector>
using namespace std;

/* 返回-1时表示当前的邮票选择不能满足要求，不断递归直到得到不是-1的结果或者遍历完所有组合返回-1 */
int solve(vector<int> &v, int total, int idx) {
  if (total < 0) return -1;
  if (total == 0) return 0;
  for (int i = idx; i >= 0; i--) {
    int tmp = solve(v, total - v[i], i);
    if (tmp == -1) continue;
    return tmp + 1;
  }
  return -1;
}

int main(int argc, char *argv[]) {
  ifstream ifs("./input3.txt");
  int n, total;
  ifs >> total >> n;
  vector<int> v(n);
  for (int i = 0; i < n; i++) ifs >> v[i];
  sort(v.begin(), v.end());
  int tmp = solve(v, total, v.size() - 1);
  cout << (tmp == -1 ? 0 : tmp);
  return 0;
}
```


## 第四题

### 题目描述

求两个字符串的最长公共子序列的长度

```
input：
abcfac abcdfd
programming contest
abc mmp
output:
4
2
0
```

### 解题思路

经典的动态规划问题，这里不进行详细讲解，具体思路可以看看[这篇文章](https://www.geeksforgeeks.org/longest-increasing-subsequence-dp-3/)，以下给出递归法和迭代法两种解法。

### 代码

+ 递归法

```cpp
#include <fstream>
#include <iostream>
using namespace std;

int lic(const string &s1, const string &s2, int i, int j) {
  if (i == 0 || j == 0) return 0;
  if (s1[i - 1] == s2[j - 1]) return lic(s1, s2, i - 1, j - 1) + 1;
  return max(lic(s1, s2, i - 1, j), lic(s1, s2, i, j - 1));
}

int main(int argc, char *argv[]) {
  ifstream ifs("./input4.txt");
  string s1, s2;
  while (ifs >> s1 >> s2) {
    cout << lic(s1, s2, s1.length() - 1, s2.length() - 1) << endl;
  }
  return 0;
}
```

+ 迭代法

```cpp
#include <iostream>
#include <fstream>
#include <vector>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./input4.txt");
  string s1, s2;
  int dp[1000][1000];
  while (ifs >> s1 >> s2) {
    int m = s1.length(), n = s2.length();
    for (int i = 1; i <= m; i++) {
      for (int j = 1; j <= n; j++) {
        if (s1[i-1] == s2[j-1])
          dp[i][j] = dp[i-1][j-1] + 1;
        else
          dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
      }
    }
    cout << dp[m][n] << endl;
  }
  return 0;
}
```
