---
title: cmu 15445 课程学习指南
date: 2022-03-08 15:18:08
tags: ["cmu-15445"]
draft: true
---

## 课程学习

[2021课程主页](https://15445.courses.cs.cmu.edu/fall2021/)

课程视频可以在youtube上找到，cmu-db组将课程公开了供大家学习，笔者观看的是2018秋季的课程，Andy Pavlo老师主讲，课程非常精彩。

**注意**：Andy老师上课特别强调了不要将自己的代码上传到网上，很多中国学生喜欢这样做，正在学习这门课的同学注意一下。并且在github上创建自己的`bustub-private`时记得要将权限设置为private而不是public。

## 课后实验

15445课后实验是这门课的精髓之一，学生需要在课程学习过程中完成`bustub`项目中的各个模块，`bustub`是一个C++实现的类似于sqlite的数据库。

### 克隆代码仓库

```
git clone https://github.com/cmu-db/bustub
git checkout -b your-code
```

### 编译代码

```
$ mkdir build
$ cd build
$ cmake -DCMAKE_BUILD_TYPE=Debug ..
$ make
```

### 代码逻辑结构

```
├── build_support   # format, lint的一些脚本
├── src             # bustub各个模块的代码
│   ├── buffer
│   ├── catalog
│   ├── CMakeLists.txt
│   ├── common
│   ├── concurrency
│   ├── container
│   ├── execution
│   ├── include
│   ├── recovery
│   ├── storage
│   └── type
├── test            # 对于各个模块的测试
├── third_party
├── CMakeLists.txt
```

`bustub`并不提供一个完整微型数据库的功能，`CMakeLists.txt`中指定了将`src`中的所有文件编译为`bustub_shared`动态库，在编译测试程序时将测试程序与这个动态库链接。学生需要完成源代码中不完整的各个模块，然后通过本地和远程测试。

### 编译并且运行测试

本地测试文件在源代码的`test`目录中，对于每一个测试文件，测试文件的`CMakeLists.txt`指定了`EXCLUDE_FROM_ALL`选项，用户需要用`make test_name`单独编译某个测试，比如对于测试文件`test/starter/primer_test.cpp`，可以在`build`目录中用`make primer_test`来编译该测试，然后用`./test/primer_test`可以运行该测试。

学生在通过本地测试后可以将对应模块的代码上传到[gradescope](https://www.gradescope.com/)上，非CMU的学生也可以创建帐号并且提交代码进行评分，gradescope上具有数量更多且更加复杂的测试。
