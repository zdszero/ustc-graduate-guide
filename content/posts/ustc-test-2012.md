---
title: 中科大2012年复试机试题详解
date: 2021-05-12 12:00:00
tags: ["中科大复试"]
draft: true
---

## 第一题

### 题目描述

字符串处理：从`string.in`文件里读入两个字符串，字符串除了数字还可能包括'-'、'E'、'e'、'．'，相加之后输出到文件`string.out`中，如果是浮点型，要求用科学计数法表示（最多包含10个有效数字）。 

```
Sample Input:                       
34.56                                
2.45e2    

Sample Output: 
2.7956e2 
```

### 解题思路

本题用C++的输入输出比较难处理，用C语言中的`scanf`、`printf`进行处理，注意按照题目要求，去除指数表示中的底数部分小数点后多余的0。

### 代码

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  double a, b;
  char buf[128];
  FILE *infile = fopen("./string.in", "r");
  FILE *outfile = fopen("./string.out", "w");
  if (!infile || !outfile) exit(1);
  fscanf(infile, "%lf", &a);
  fscanf(infile, "%lf", &b);
  sprintf(buf, "%.10e", a+b);
  int i, j;
  for (i = 0; buf[i] != '\0'; i++)
    if (buf[i] == 'e')
      break;
  for (j = i-1; buf[j] == '0'; j--);
  for (int k = 0; k <= j; k++)
    fputc(buf[k], outfile);
  fputc('e', outfile);
  for (j = i+1; buf[j] != '\0'; j++)
    fputc(buf[j], outfile);
  return 0;
}
```


## 第二题

### 题目描述

 
最大公约数：从`number.in`文件中读入n个数，求出这n个数的最小值、最大值以及它们两的最大公约数，输出到文件`number.out`中。number.in 中第一行为n，接下来为n个大于零的整数。

```
Sample Input:
3
4 8 6

Sample Output:
4 8 4
```

### 代码实现

```cpp
#include <fstream>
#include <vector>
using namespace std;

int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }

int main(int argc, char *argv[]) {
  ifstream ifs("./number.in");
  ofstream ofs("./number.out");
  int n;
  ifs >> n;
  vector<int> a(n);
  for (int i = 0; i < n; i++)
    ifs >> a[i];
  int minVal, maxVal;
  minVal = maxVal = a[0];
  for (int i = 1; i < n; i++) {
    if (a[i] < minVal)
      minVal = a[i];
    if (a[i] > maxVal)
      maxVal = a[i];
  }
  ofs << minVal << " " << maxVal << " " << gcd(maxVal, minVal);
  return 0;
}
```

## 第三题

### 题目描述

任务调度：从`task.in`文件中读入任务调度序列，输出n个任务适合的一种调度方式到`task.out`中。每行第一个表示前序任务，括号中的任务为后序任务，表示只有在前序任务完成的情况下，后序任务才能开始。若后序为NULL则表示无后序任务。

```
Sample Input:
Task0(Task1,Task2)
Task1(Task3)
Task2(NULL)
Task3(NULL)

Sample Output:
Task0 Task1 Task3 Task2
```

### 解题思路

本题考察的是`拓朴排序`的代码实现，对于该算法不熟悉的请参考[这篇文章](https://www.geeksforgeeks.org/topological-sorting/)。在这里用`dfs`和`visit数组`来实现拓朴排序。

本题另外一个难点在于处理给定的文件形式，我们用多个`unordered_map`来存储某个任务名字与对应结点号之间的映射关系。

### 代码

```cpp
#include <fstream>
#include <unordered_map>
#include <vector>
using namespace std;

unordered_map<string, int> idxMap;
unordered_map<int, string> nameMap;
unordered_map<int, vector<int>> adjMap;
vector<int> q;
int curIdx = 0;

void parseLine(const string &line) {
  string tmp;
  int i, j;
  for (i = 0; i < line.length(); i++)
    if (line[i] == '(')
      break;
  tmp = line.substr(0, i);
  if (!idxMap.count(tmp)) {
    nameMap[curIdx] = tmp;
    idxMap[tmp] = curIdx++;
  }
  int src = idxMap[tmp];
  for (j = ++i; j <= line.length(); j++) {
    if (line[j] == ',' || line[j] == ')') {
      tmp = line.substr(i, j - i);
      if (!idxMap.count(tmp)) {
        nameMap[curIdx] = tmp;
        idxMap[tmp] = curIdx++;
      }
      adjMap[src].push_back(idxMap[tmp]);
      i = ++j;
    }
  }
}

void dfs(vector<int> &vis, int src) {
  vis[src] = 1;
  for (int next : adjMap[src]) {
    if (next == -1)
      break;
    if (vis[next])
      continue;
    dfs(vis, next);
  }
  q.push_back(src);
}

int main(int argc, char *argv[]) {
  ifstream ifs("./task.in");
  ofstream ofs("./task.out");
  idxMap["NULL"] = -1;
  string line;
  while (getline(ifs, line)) {
    parseLine(line);
  }
  int size = curIdx;
  vector<int> vis(size, 0);
  for (int i = 0; i < size; i++) {
    if (vis[i])
      continue;
    dfs(vis, i);
  }
  for (int i = q.size() - 1; i >= 0; i--) {
    if (i != q.size()-1)
      ofs << " ";
    ofs << nameMap[q[i]];
  }
  return 0;
}
```

## 第四题

### 题目描述

火车票订购：火车经过X站，火车的最大载客人数为m，有n个订票请求，请求订购从a站到b站的k张票，若能满足订购要求则输出1，否则输出0。数据从`ticket.in`中输入，第一行有两个数字，分别是n，m。接下来有n行，每行三个数分别是a、b、k。结果输出到文件`ticket.out`中。

```
Sample Input:
5 10
4 10 9
8 12 2
8 12 1
14 20 8
30 300 15

Sample Output:
1
0
1
1
0
```

### 解题思路

用一个大数组来存储每一站的车上人数，在每一次运送乘客时先尝试是否有足够空位，如果有空位则将flag置为1，否则将flag置为0，如果尝试之后flag=1则可以搭载这波乘客。

### 代码

```cpp
#include <fstream>
#include <map>
#include <memory.h>
using namespace std;

int main(int argc, char *argv[]) {
  ifstream ifs("./ticket.in");
  ofstream ofs("./ticket.out");
  int nums[10000];
  memset(nums, 0, 10000);
  int n, m;
  ifs >> n >> m;
  int a, b, c, flag;
  for (int i = 0; i < n; i++) {
    ifs >> a >> b >> c;
    flag = 1;
    for (int j = a; j < b; j++) {
      if (nums[j] + c > m) {
        flag = 0;
        break;
      }
    }
    if (flag) {
      for (int j = a; j < b; j++) {
        nums[j] += c;
      }
    }
    ofs << flag << endl;
  }
  return 0;
}
```

## 第五题

## 题目描述

最短路径：有n个城市m条道路(n &lt; 1000, m &lt; 10000),每条道路有个长度,请找到从起点s到终点t的最短距离,并且输出经过的城市的名,如果有多条,输出字典序最小的那条;若从 s 到 t 没有路径,则输出“can't arrive”。从 road.in 中读入数据,第一行有四个数,分别为 n,m,s,t。接下来 m 行,每行三个数,分别为两个城市名和距离。输出结果到 road.out 中。

```
Sample Input:
3 3 1 3
1 3 3
1 2 1
2 3 1

Sample Output:
2
1 2 3
```

### 解题思路

使用`floyed-warshall`算法。

### 代码

```cpp
#include <fstream>
#include <vector>
using namespace std;

#define INF 0x7fffffff

int main(int argc, char *argv[]) {
  ifstream ifs("./road.in");
  ofstream ofs("./road.out");
  int cityCnt, roadCnt, src, dst;
  ifs >> cityCnt >> roadCnt >> src >> dst;
  vector<vector<int>> road(cityCnt+1, vector<int> (cityCnt+1, INF));
  vector<vector<int>> next(cityCnt+1, vector<int> (cityCnt+1, -1));
  for (int i = 0; i < roadCnt; i++) {
    int a, b, c;
    ifs >> a >> b >> c;
    road[a][b] = c;
    next[a][b] = b;
  }
  // floyed-warshall
  for (int k = 1; k <= cityCnt; k++) {
    for (int i = 1; i <= cityCnt; i++) {
      for (int j = 1; j <= cityCnt; j++) {
        if (road[i][k] != INF && road[k][j] != INF && road[i][k] + road[k][j] < road[i][j]) {
          road[i][j] = road[i][k] + road[k][j];
          next[i][j] = k;
        }
      }
    }
  }
  ofs << road[src][dst] << endl;
  int i = src;
  while (i != dst) {
    ofs << i << " ";
    i = next[i][dst];
  }
  ofs << dst;
  return 0;
}
```
