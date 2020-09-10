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

## GCC 优化等级
首先我们先看看 GCC 都有哪些优化等级。下表出自[gcc -o / -O option flags](https://www.rapidtables.com/code/linux/gcc/gcc-o.html)
![gcc opt levels](/post_images/2020-06-17-change-linux-gcc-optimalize-level/gcc-opt-level.jpg)

简单来说
- -O0 优化编译时间
- -O(-O1) 优化代码大小和执行效率
- -O2 比 -O1 跟进一步的优化代码大小和执行效率
- -O3 同上更进一步优化
- -Os 代码大小的进一步优化

同时我们也注意到优化等级越高编译耗时越长，执行效率的提升代价是内存空间的消耗。

回到我们的正题优化很重要的一部分就是把函数转为内联(inline)。

根据[这份gcc文档](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)我们可以看看每个 level 都做了哪些事情。本文我们主要关注 inline 相关的一些优化，这部分内容我摘录在下面方便你理解，更多的选项感兴趣可自行查看。
```
-O1 级别有一项：
-finline-functions-called-once

-O2 -O3 -Os 都有两项：
-finline-functions 
-finline-small-functions
```
具体解释如下：
> <big>-finline-functions-called-once</big>
> Consider all static functions called once for inlining into their caller even if they are not marked inline. If a call to a given function is integrated, then the function is not output as assembler code in its own right.
> 
> Enabled at levels -O1, -O2, -O3 and -Os, but not -Og.
> 
> <big>-finline-functions</big>
> Consider all functions for inlining, even if they are not declared inline. The compiler heuristically decides which functions are worth integrating in this way.
> 
> If all calls to a given function are integrated, and the function is declared static, then the function is normally not output as assembler code in its own right.
> 
> Enabled at levels -O2, -O3, -Os. Also enabled by -fprofile-use and -fauto-profile.
> 
> <big>-finline-small-functions</big>
> Integrate functions into their callers when their body is smaller than expected function call code (so overall size of program gets smaller). The compiler heuristically decides which functions are simple enough to be worth integrating in this way. This inlining applies to all functions, even those not declared inline.
> 
> Enabled at levels -O2, -O3, -Os. 

所以困扰我的“static 函数被优化的问题”其实是这一条 **-finline-functions-called-once**, 只要定义为 static 且只有一处调用的函数会被内联，优化等级 -O1 起就有此项了。此外一些所谓的小函数在 -O2 -O3 -Os 也会被优化掉。

而内核的优化等级主要是 -O2 和 -Os，最新的内核还会有 -O3。所以要达到我们的目的需要把优化等级降到 -O0 或者 -Og。

-------
## 去优化
原理我们现在清楚了，那如何去避免这些优化呢？

### noinline
最简单的，如果我们只需要少数几个函数不被优化，那就在目标函数前声明 `noinline` 就好了。

那么更大范围呢，比如一个模块。最粗暴的我们可以 sed + 正则 扫过去。但是这样有可能覆盖不全或者错误添加，毕竟正则嘛，其次改动太多。或许你想起来了可以在内核的顶层目录修改优化等级，但是很遗憾全局范围的 -O0 是编不过去的。

### CFLAGS_REMOVE_
内核中提供了方法。

对于单个文件，可在 Makefile 中定义 CFLAGS_REMOVE_xxx.o
```
# 比如目标文件为 target.c 则在对应的 Makefile 中添加
CFLAGS_REMOVE_target.o = -O2
```
我们分析一下它的原理。grep 搜索这个变量前缀可以发现，他是被用在 scripts/Makefile.lib
```
$ ag CFLAGS_REMOVE  scripts/Makefile.lib
106:_c_flags       = $(filter-out $(CFLAGS_REMOVE_$(basetarget).o), $(orig_c_flags))
```
分析这个路径 scripts/Makefile.lib 可以猜到这是内核编译的公共 makefile。`filter-out` 是 make 的内置函数，作用就是将 `CFLAGS_REMOVE_$(basetarget).o` 中的字符串从 `orig_c_flags` 中剔除。我们再看一下这个 orig_c_flags。
```
$ ag -A2 "orig_c_flags.*="  scripts/Makefile.lib
104:orig_c_flags   = $(KBUILD_CPPFLAGS) $(KBUILD_CFLAGS) $(KBUILD_SUBDIR_CCFLAGS) \
105-                 $(ccflags-y) $(CFLAGS_$(basetarget).o)
106-_c_flags       = $(filter-out $(CFLAGS_REMOVE_$(basetarget).o), $(orig_c_flags))
```
看到这里基本上我们大概知道了就是先加载总的 cflags 配置，根据 CFLAGS_REMOVE_ 来裁剪某个文件具体的配置。当然你还可以搜一下 `_c_flags`，这里就不展开了。

### KBUILD_CFLAGS
KBUILD_CFLAGS 用来进行目录级的修改。至于原理经过我们上面的分析你应该也明白了，通过操作 KBUILD_CFLAGS 来影响 orig_c_flags 从而作用于整个目录。

用法：
```
KBUILD_CFLAGS := $(filter-out -O2, $(KBUILD_CFLAGS))
```



