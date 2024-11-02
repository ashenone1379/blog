---
title: "Lab 01: Number Rep, C and CGDB"
draft: false
tags:
  - gdb
  - valgrind
  - c
  - dev-tool
---
# Valgrind
> [!INFO] Heisenbugs & Bohrbugs
> *Heisenbugs* 是指那些如同[海森堡测不准原理](https://zh.wikipedia.org/zh-cn/不确定性原理)一样难以测定, 难以复现的bug;
> 
> *Bohrbugs* 则与之相反, 如同[玻尔模型](https://zh.wikipedia.org/wiki/玻尔模型)中的原子, 行为可以预测, 容易复现和解决的bug.
> 

[Valgrind](https://valgrind.org/)是一个用于内存调试、内存泄漏检测以及性能分析的开发工具. 

很多时候, *Heisenbugs* 是由内存管理问题导致的, 因此**Valgrind**非常适合用来发现*Heisenbugs*.

> [!caution] Valgrind 平台兼容性
> Valgrind 并**不兼容Windows平台**, 因此UCB给他们的学生提供了Hive Machine服务器, 学生可以通过SSH连接, 使用上面配有Valgrind的开发环境.
> 
> 作为校外白嫖课程的学生当然没资格享受这个服务了, 所以另一种解决方法是: 使用[WSL](https://learn.microsoft.com/zh-cn/windows/wsl/about)~~(其实也就相当于SSH连接到本地的Linux服务器)~~

下面来看lab中提供的两个示例程序`segfault`和`no_segfault`, 以及两者的运行结果.
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
可以看到`segfault`程序直接崩溃了, 报错称遇到了segmentation fault.
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
而`no_segfault`程序则成功运行, 但是显然结果不正确, 且每次都不同.
```bash
❯ ./no_segfault
sum of array is -631332874
❯ ./no_segfault
sum of array is -2022076669
❯ ./no_segfault
sum of array is 263147542

```
显然第一个程序出现的bug属于*Heisenbug*, 而第二个则是*Bohrbug*.
> TODO run valgrind

