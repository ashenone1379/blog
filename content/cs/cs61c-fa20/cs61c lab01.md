---
title: "Lab 01: Number Rep, C and CGDB"
draft: false
tags:
  - gdb
  - valgrind
  - c
  - dev-tool
---
> [点此阅读Lab01](https://www.learncs.site/docs/curriculum-resource/cs61c/labs/lab01)
# Ex00 Command Line Basics
使用`gcc`编译程序的基本方法如下.

紧跟`-o`选项的字段为目标程序名, 接下来是程序的各个源文件.
```bash
❯ gcc -o <program_name> src_file_1.c src_file_2.c ...
```
# Ex01 See What You Can C
主要是让你理解C语言的分支控制语句, 太基础了, 跳过.
# Ex02 GDB/CGDB 
# Ex03 Input Redirection
# Ex04 Valgrind
> [!INFO] Heisenbugs & Bohrbugs
> *Heisenbugs*是指那些如同[海森堡测不准原理](https://zh.wikipedia.org/zh-cn/不确定性原理)一样难以测定, 难以复现的bug;
> 
> *Bohrbugs*则与之相反, 如同[玻尔模型](https://zh.wikipedia.org/wiki/玻尔模型)中的原子, 行为可以预测, 容易复现和解决的bug.
> 

[Valgrind](https://valgrind.org/)是一个用于内存调试、内存泄漏检测以及性能分析的开发工具. 

很多时候, *Heisenbugs*是由内存管理问题导致的, 因此**Valgrind**非常适合用来捕获这种类型的*Heisenbugs*.

> [!caution] Valgrind 平台兼容性
> Valgrind 并**不兼容Windows平台**, 因此UCB给他们的学生提供了Hive Machine服务器, 学生可以通过SSH连接, 使用上面配有Valgrind的开发环境.
> 
> 作为非本校的, 白嫖课程的学生当然没资格享受这个服务了, 所以另一种解决方法是: 使用[WSL](https://learn.microsoft.com/zh-cn/windows/wsl/about)~~(其实也就相当于SSH连接到本地的Linux服务器)~~

下面来看编译, 运行lab中提供的两个示例程序`segfault`和`no_segfault`.
> `segfault`
```c
//segfault.c

int main() {
    int a[20];
    for (int i = 0; ; i++) {
        a[i] = i;
    }
}
```
`segfault`程序直接崩溃了, 报错称遇到了segmentation fault.
```bash
❯ ./segfault
[1]    18021 segmentation fault (core dumped)
```

> `no_segfault`
```c
//no_segfault.c

#include <stdio.h>
int main() {
    int a[5] = {1, 2, 3, 4, 5};
    unsigned total = 0;
    for (int j = 0; j < sizeof(a); j++) {
        total += a[j];
    }
    printf("sum of array is %d\n", total);
}

```
`no_segfault`程序则成功运行, 但是输出结果不正确且每次都不同.
```bash
❯ ./no_segfault
sum of array is -631332874
❯ ./no_segfault
sum of array is -2022076669
❯ ./no_segfault
sum of array is 263147542

```
显然第一个程序出现的bug为*Bohrbug*, 属于一眼就能找到错误的类型: 无限循环导致的数组下标越界导致的分段错误. 

而第二个则是*Heisenbug*, 出错的地方可就没有那么明显了.

接下来使用valgrind分析两个程序.
> TODO: run valgrind

