---
title: bash：你真的了解这两个尖尖的东西吗
date: 2020-06-23 11:48:41
updte: 2020-06-23 11:48:41
categories:
- shell
tags:
- linux
- shell
---
先来看一个问题。
假设有一个文件`echo 111 > test`，那么：
- `cat <test >test` 的输出是什么？

运行一下看看结果和你想的是否一样。如果你明确的知道为什么，那么就不必再看了。

<!--more-->

很神奇是不是，读出来竟然是空的，不止输出是空，连文件本身都被清空了。

那么这到底是为什么呢？

作为一个硬核博客我们自然是要追本溯源的，跟我来吧~

`strace` 是深挖细节的不二之选。但是 strace 直接抓上面的命令，shell 会把重定向符归属到 strace 命令本身，所以我们需要利用一点 strace 的小技巧：
```
strace -fp pid 
```
来跟踪一个进程。-p 选项指定进程，-f 选项跟踪子进程。

具体操作就是
- 打开两个 shell，我们分别记作 A, B。
- 在 A 中 `echo $$` 获取 A 的 pid
- B 中运行 `strace -fp pid `
- 然后我们在 A 中运行测试命令，B 中观察 strace

接下来就开始我们的探案之旅吧。

A 中运行 `cat <test >test`，我们会发现 B 中出现巨量的打印，这里我把关键部分摘出来：

```
[pid 12326] open("test", O_RDONLY|O_LARGEFILE) = 3
[pid 12326] dup2(3, 0)                  = 0
[pid 12326] close(3)                    = 0
[pid 12326] open("test", O_WRONLY|O_CREAT|O_TRUNC|O_LARGEFILE, 0666) = 3
[pid 12326] dup2(3, 1)                  = 1
[pid 12326] close(3)                    = 0
[pid 12326] execve("/bin/cat", ["cat"], [/* 28 vars */]) = 0
```

这一部分是在做什么呢，我来解读一下。

### 重定向的原理
首先我们知道每个进程的0、1、2号文件描述符分别对应 stdin, stdout, stderr，也就是标准输入、标准输出、标准错误。

进程每打开一个文件都对应一个文件描述符(fd, file descriptor)，所谓**重定向**就是打开一个文件然后将标准输入/输出的fd与之绑定。

它的实现依赖的是系统调用dup2()来完成的，我们 `man` 一下 dup2

```
SYNOPSIS
       #include <unistd.h>

       int dup(int fildes);
       int dup2(int fildes, int fildes2);

DESCRIPTION
       The  dup()  function  provides an alternative interface to the service provided by fcntl() using the F_DUPFD command. The
       call dup(fildes) shall be equivalent to:

           fcntl(fildes, F_DUPFD, 0);

       The dup2() function shall cause the file descriptor fildes2 to refer to the same open file description as  the  file  de‐
       scriptor fildes and to share any locks, and shall return fildes2.  If fildes2 is already a valid open file descriptor, it
       shall be closed first, unless fildes is equal to fildes2 in which case dup2() shall return fildes2 without closing it. If
       the  close  operation  fails to close fildes2, dup2() shall return −1 without changing the open file description to which
       fildes2 refers. If fildes is not a valid file descriptor, dup2() shall return −1 and shall not close fildes2.  If fildes2
       is less than 0 or greater than or equal to {OPEN_MAX}, dup2() shall return −1 with errno set to [EBADF].
```
我们结合上面 strace 抓到的流程，可见 bash 会在执行命令前先把重定向准备好。

`dup2(3,0)`对应的就是 `<` 输入重定向。`dup2(3,1)` 则是对应 `>` 输出重定向。而这里我们重点关注open的参数，在输出重定向对的open中：
`O_WRONLY|O_CREAT|O_TRUNC|O_LARGEFILE` 其中 O_TRUNC 就是清空文件的原因所在了。我们 `man 2 open` 可以搜到

```
O_TRUNC
       If the file already exists and is a regular file and the access mode allows writing (i.e., is O_RDWR or  O_WRONLY)
       it  will  be  truncated  to length 0.  If the file is a FIFO or terminal device file, the O_TRUNC flag is ignored.
       Otherwise, the effect of O_TRUNC is unspecified.

```
也就是说该选项将文件长度清0了，而根据我们 strace 看到的情况所有的重定向工作都在 cat 命令执行之前准备完成，所以当 cat 去读文件时文件已经被清空了。


### cat < test >>test
那不妨再猜猜 `cat <test >>test` 会怎么样？当然前提和上面一样 test 中有一行内容。给你一分钟想一想试一试。

你猜对了吗，这条命令会无限套娃根本停不下来，CTRL+C 退出后你可以 `cat test` 感受一下。

那到底发生了什么呢？

首先我们要知道 cat 命令是可以从 stdin 和文件中获取输入的，你可以试试在命令行只运行`cat` 会发生什么，CTRL+D 退出哦。

接下来照例我们 strace 抓一下，我依然截取部分。
```
[pid 12399] open("test", O_RDONLY|O_LARGEFILE) = 3
[pid 12399] dup2(3, 0)                  = 0
[pid 12399] close(3)                    = 0
[pid 12399] open("test", O_WRONLY|O_CREAT|O_APPEND|O_LARGEFILE, 0666) = 3
[pid 12399] dup2(3, 1)                  = 1
[pid 12399] close(3)                    = 0
[pid 12399] execve("/bin/cat", ["cat"], [/* 28 vars */]) = 0
```
与上面相比的话差异主要在于输出重定向时打开文件方式是 `O_APPEND` 而不是 `O_TRUNC`，所以文件没有被清空。

此时文件内容作为 stdin 被 cat 读入然后写入文件尾部，cat 没有接到 EOF(CTRL+D)信号则会继续这个动作，套娃开始。

那到底有没有办法能不请空文件又不套娃呢？

### <> 操作符
试试 `cat <>test`，哈哈终于正常了。

我们看一下 [GUN bash menual](https://www.gnu.org/software/bash/manual/html_node/Redirections.html) 给出的解释：
> 3.6.10 Opening File Descriptors for Reading and Writing
> The redirection operator
> 
> [n]<>word
> causes the file whose name is the expansion of word to be opened for both reading and writing on file descriptor n, or on file descriptor 0 if n is not specified. If the file does not exist, it is created.
> 

`<>` 是以读写的方式将文件冲重向到 0 中。这里就不再抓系统调用了，相信你也猜到了就是以读写方式打开文件，然后 dup2() 绑定 0。

### 小结

