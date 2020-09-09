---
title: Linux 内核如何避免编译器优化
date: 2020-06-17 22:02:53
updte: 2020-06-17 22:02:53
categories:
- Linux Kernel
tags:
- Linux
- Gcc
- Makefile
---

最近调试内核使用 ftrace 的时候总是由于内核编译器的优化导致一些函数(主要是部分 static void 定义的函数)不能被追踪。去看 nm 和反汇编就会发现此时这些函数都已经不是一个独立的函数了。

那么问题来了，为什么它们不配拥有姓名呢？


会被优化的主要是这几类函数：
- static 并且只有一处调用
- 一些小函数


<!--more-->

首先我们先看看 GCC 都有哪些优化等级。下表出自[gcc -o / -O option flags](https://www.rapidtables.com/code/linux/gcc/gcc-o.html)
![gcc opt levels](/post_images/post_images/2020-06-17-change-linux-gcc-optimalize-level/gcc-opt-level.jpg)

简单来说
- -O0 优化编译时间
- -O(-O1) 优化代码大小和执行效率
- -O2 比 -O1 跟进一步的优化代码大小和执行效率
- -O3 同上更进一步优化
- -Os 代码大小的进一步优化

同时我们也注意到优化等级越高编译耗时越长，执行效率的提升代价是内存空间的消耗。

回到我们的正题优化很重要的一部分就是把函数转为内联(inline)。

根据[这份gcc文档](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)我们可以看看每个 level 都做了哪些事情。本文我们主要关注
inline 相关的一些优化，这部分内容我摘录在下面方便你理解，更多的选项感兴趣可自行查看。
```
-O1 级别有一项：
-finline-functions-called-once

-O2 -O3 -Os 都有两项：
-finline-functions 
-finline-small-functions

具体解释如下：
-finline-functions-called-once
Consider all static functions called once for inlining into their caller even if they are not marked inline. If a call to a given function is integrated, then the function is not output as assembler code in its own right.

Enabled at levels -O1, -O2, -O3 and -Os, but not -Og.

-finline-functions
Consider all functions for inlining, even if they are not declared inline. The compiler heuristically decides which functions are worth integrating in this way.

If all calls to a given function are integrated, and the function is declared static, then the function is normally not output as assembler code in its own right.

Enabled at levels -O2, -O3, -Os. Also enabled by -fprofile-use and -fauto-profile.

-finline-small-functions
Integrate functions into their callers when their body is smaller than expected function call code (so overall size of program gets smaller). The compiler heuristically decides which functions are simple enough to be worth integrating in this way. This inlining applies to all functions, even those not declared inline.

Enabled at levels -O2, -O3, -Os. 

```
所以困扰我的“static 函数被优化的问题”其实是这一条 **-finline-functions-called-once**, 只要定义为 static 且只有一处调用的函数会被内联，优化等级 -O1 起就有此项了。

而内核的优化等级主要是 -O2 和 -Os，最新的内核还会有 -O3。因此呢
