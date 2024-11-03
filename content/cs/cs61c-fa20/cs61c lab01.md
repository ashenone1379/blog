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
❯ gcc -o {program_name} src_file_1.c src_file_2.c ...
```
# Ex01 See What You Can C
主要是让你理解C语言的分支控制语句, 太基础了, 跳过.
# Ex02 GDB/CGDB 
## GDB & CGDB
[GDB](https://sourceware.org/gdb/)是GNU 项目的一部分, 是一个强大的调试工具. 

[CGDB](https://cgdb.github.io/)与之类似, 但是添加了一个显示代码的区域, 在调试过程中可以显示程序运行于代码的具体位置. 通过`<ESC>`可以将光标移动到代码区, `i`回到命令区.

> gdb/cgdb的基本用法可以参考[gdb refcard](attachments/gdb5-refcard.pdf)

为生成可供调试的程序, 需要在编译时添加`-g`选项.
```bash
❯ gcc -g -o {program_name} src_file_1.c src_file_2.c ...
```

## Questions
>[!question] Q1 设置传入参数
>While you’re in a gdb session, how do you **set the arguments** that will be passed to the program when it’s run?

```gdb
set args <arg1> arg2> ...
```

>[!question] Q2 设置断点
> How do you **create a breakpoint**?

```gdb
b <line number>
```

 >[!question] Q3 逐过程调试
 > How do you execute the next line of C code in the program after stopping at a breakpoint?
 
```gdb
n
```

>[!question] Q4 逐行调试
>If the next line of code is a function call, you’ll execute the whole function call at once if you use your answer to #3. (If not, consider a different command for #3!) How do you tell GDB that you **want to debug the code inside the function** (i.e. step into the function) instead? (If you changed your answer to #3, then that answer is most likely now applicable here.)

```gdb
s
```

>[!question] Q5 从断点继续
>How do you **continue the program after stopping** at a breakpoint?

```gdb
c
```

>[!question] Q6 求值
>How can you **print the value of a variable** (or even an expression like 1+2) in gdb?

```gdb
p <expression>
```

>[!question] Q7 添加监视
>How do you configure gdb so it **displays the value of a variable after every step**?

```gdb
display <val name>
```

>[!question] Q8 查看当前作用域变量值
>How do you **show a list of all variables and their values** in the current function?

```gdb
info local
```

>[!question] Q9 退出
>How do you **quit** out of gdb?

```gdb
q
```
# Ex03 Input Redirection
# Ex04 Valgrind
## Intro
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

## Bugs
下面来编译, 运行lab中提供的两个示例程序`segfault`和`no_segfault`.
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

而第二个则是*Heisenbug*, 出错的地方可就没有那么明显了. 你能找到其中的错误吗?

## Valgrind Memcheck
接下来使用valgrind分析两个程序.
### `segfault`
> 使用命令`> ./valgrind segfault`


 ```
==3460== Memcheck, a memory error detector
==3460== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==3460== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==3460== Command: ./segfault
==3460== 
==3460== Invalid write of size 4
==3460==    at 0x10914F: main (segfault_ex.c:4)
==3460==  Address 0x1fff001000 is not stack'd, malloc'd or (recently) free'd
==3460== 
==3460== 
==3460== Process terminating with default action of signal 11 (SIGSEGV)
==3460==  Access not within mapped region at address 0x1FFF001000
==3460==    at 0x10914F: main (segfault_ex.c:4)
==3460==  If you believe this happened as a result of a stack
==3460==  overflow in your program's main thread (unlikely but
==3460==  possible), you can try to increase the size of the
==3460==  main thread stack using the --main-stacksize= flag.
==3460==  The main thread stack size used in this run was 8388608.
==3460== 
==3460== HEAP SUMMARY:
==3460==     in use at exit: 0 bytes in 0 blocks
==3460==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
==3460== 
==3460== All heap blocks were freed -- no leaks are possible
==3460== 
==3460== For lists of detected and suppressed errors, rerun with: -s
==3460== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
[1]    3460 segmentation fault (core dumped)  valgrind ./segfault
 ```

> 或是直接使用IDE调用valgrind, 获得了更加可读的结果, 错误的代码行位置也被一并指出了


![valgrind_segfault01](attachments/valgrind_segfault01.png)
![valgrind_segfault02](attachments/valgrind_segfault02.png)
第一个例子的警告是很清晰的: 向非栈中, 或未被`malloc`, 或并非最近被`free`掉的区域, 非法写值, 结果就是程序发生segmentation fault崩溃.
### `nosegfault`
![valgrind_nosegfault01](attachments/valgrind_nosegfault01.png)
第二个立即的报错信息就不是那么直观了, 但是方向是对的: 访问了未初始化的值. 这导致了输出的结果每次都不同.
## Questions
>[!question] Why is Valgrind important and how is it useful?

Valgrind 能够分析内存错误, 帮助我们找到*Heisenbug*.

> [!question] How do you run a program in Valgrind?

使用命令`valgrind {executable}`

> [!question] How do you interpret the error messages? 

- 例1: `InvalidWrite` $\to$ 给未分配的内存写入值, 由数组下标越界导致;
- 例2: `UninitValue` $\to$ 使用了未初始化的值, 这里同样也是数组下标越界导致.


> [!question] Why might uninitialized variables result in “heisenbugs”?

未初始化的变量存储着没有意义且不确定的垃圾值.
## Further Questions
>[!question] Why **didn’t** the `no_segfault_ex` program segfault?

`no_segfault`只越界访问了数组后的一小部分区域, 这一部分的内存是被分配给教程了的.

>[!question] Why does the `no_segfault_ex` produce inconsistent outputs?

`no_segfault`在循环第5次后, 开始读取紧接着数组后的内存的值, 这部分的没有被初始化, 里面的值都是不确定的垃圾值, 所以每次运行结果都不同.

>[!question] Why is `sizeof` incorrect? How could you still use `sizeof` but make the code correct?

`sizeof`返回对象所占内存的大小, 在大多数的现代机器上, `int`的大小为4 bytes, 所以一个存储5个`int`变量的数组的大小应该是$5\times 4=20$ bytes.  所以循环实际上会进行20次, 而非预期的5次. 

正确的写法为
```c
    for (int j = 0; j < sizeof(a) / sizeof(a[0]); j++) {
        total += a[j];
    }
```
`sizeof(a) / sizeof(a[0])`将正确返回数组的元素个数.