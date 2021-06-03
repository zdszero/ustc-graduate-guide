---
title: 中科大2008年复试机试题详解
date: 2021-05-08 12:00:00
tags: ["中科大复试"]
draft: true
---

## 题目一

### 题目描述

一个十进制正整数转换成二进制有多少个1

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int num, cnt = 0;
  cin >> num;
  for (int i = 0; i < 32; i++) {
    cnt += (num & 1);
    num >>= 1;
  }
  cout << cnt;
  return 0;
}
```

## 题目二

### 题目描述

约瑟夫环问题，可以参考[这篇文章](https://www.cxyxiaowu.com/1159.html)

## 题目三

### 题目描述

求矩阵的转置。

## 题目四

### 题目描述

字符串问题。从文件`4.in`中读入几行英文句子。输出每个单词出现的个数，并且按照字典索引输出到控制台。

```
输入示例：
I am
a student from china
china
am student

输出示例：
I 1
a 1
am 2
china 2
from 1
student 2
```

### 代码

```cpp
#include <fstream>
#include <iostream>
#include <map>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./4.in");
  map<string, int> occMap;
  string tmp;
  while (ifs >> tmp) {
    occMap[tmp]++;
  }
  for (auto it : occMap) {
    cout << it.first << " " << it.second << endl;
  }
  return 0;
}
```
