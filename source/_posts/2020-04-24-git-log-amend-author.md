---
title: Git 修改历史提交的作者信息 
date: 2020-04-24 11:27:09
categories:
	git	
tags:
	- git
	- git rebase
---

对于最近一次的提交我们可以使用 `git commit --amend --author="name <your@email>" --no-edit`
来进行修改。

那么对于更早之前的提交我们该怎么办呢？

<!--more-->

我们可以通过 `git rebase` 来实现。上例子。


我们现在有一份 git log 看起来是这样的：
```
commit 8d180449cb0f295de18958cb4353c0b42c3f1dcf (HEAD -> master)
Author: Zhao Yujiu <yujiuzhao@gmail.com>
Date:   Fri Apr 24 13:39:59 2020 +0800

    commit 3

commit 7a98ea783e3b169d6a9a703cf2ceb0f570e6a2dd
Author: zyj <zyj@example.com>
Date:   Fri Apr 24 13:37:06 2020 +0800

    commit 2

commit 0772372634e4653af7bee9b24f23c722068b682d
Author: z <z@example.com>
Date:   Fri Apr 24 13:36:26 2020 +0800

    commit 1
(END)

```

可以看到三次提交的作者都不一样，对于 Github 来说，它只认识 commit 3 的邮箱，
也就只有第三次的提交记录能看到我们帅帅的头像，前两次都是黑户，这怎么能忍呢。

那我们的目标就是改掉前两次提交的作者信息然后推送到远程库。

---

这里我们需要使用 `git rebase` 来改写我们的提交。`-i` 指定要从哪个 commit 之后操作。

比如我要修改 `7a98ea783e3b169d6a9a703cf2ceb0f570e6a2dd` `-i` 要指向 `0772372634e4653af7bee9b24f23c722068b682d`

那对于第一次的提交怎么办呢，毕竟它之前没有记录了。对于它就要特殊处理了，加上 `--root` 标记，就可以对全部提交操作了。

```
git rebase -i --root
```

### 1. 指定修改项
```
$ git rebase -i --root
```

进入如下界面，前提需要配置一下编辑器 `git config --global core.editor vim`

```
  1 pick 0772372 commit 1
  2 pick 7a98ea7 commit 2
  3 pick 8d18044 commit 3
  4
  5 # 变基 8d18044 到 c7794cc（3 个提交）
  6 #
  7 # 命令:
  8 # p, pick <提交> = 使用提交
  9 # r, reword <提交> = 使用提交，但修改提交说明
 10 # e, edit <提交> = 使用提交，进入 shell 以便进行提交修补
 11 # s, squash <提交> = 使用提交，但融合到前一个提交
 12 # f, fixup <提交> = 类似于 "squash"，但丢弃提交说明日志
 13 # x, exec <命令> = 使用 shell 运行命令（此行剩余部分）
 14 # b, break = 在此处停止（使用 'git rebase --continue' 继续变基）
 15 # d, drop <提交> = 删除提交
 16 # l, label <label> = 为当前 HEAD 打上标记
 17 # t, reset <label> = 重置 HEAD 到该标记
 18 # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
 19 # .       创建一个合并提交，并使用原始的合并提交说明（如果没有指定
 20 # .       原始提交，使用注释部分的 oneline 作为提交说明）。使用
 21 # .       -c <提交> 可以编辑提交说明。
 22 #
 23 # 可以对这些行重新排序，将从上至下执行。
 24 #
 25 # 如果您在这里删除一行，对应的提交将会丢失。
 26 #
 27 # 然而，如果您删除全部内容，变基操作将会终止。
 28 #

```
把 commit 1、2 前面的 pick 改为 edit 即可，commit 3 我们保留。
```
  1 e 0772372 commit 1
  2 e 7a98ea7 commit 2
  3 pick 8d18044 commit 3
```
然后保存退出。
### 2. 修改作者信息

```
停止在 0772372... commit 1
您现在可以修补这个提交，使用

  git commit --amend

当您对变更感到满意，执行

  git rebase --continue

```
这里 rebase 停下来提示我们可以为所欲为了～

改它！
```
$ git commit --amend --author="Zhao Yujiu <yujiuzhao@gmail.com>" --no-edit
[分离头指针 c96e143] commit 1
 Author: Zhao Yujiu <yujiuzhao@gmail.com>
 Date: Fri Apr 24 13:36:26 2020 +0800
 1 file changed, 1 insertion(+)
 create mode 100644 log

```
这里 --no-edit 表示我们只想改作者不想进编辑器再修改 log 内容了。反之就去掉这个选项。

然后我们让 rebase 继续。
```
$ git rebase --continue
```
如果我们有多个需要 edit 的 commit，rebase 每个都会停下来，我们挨个
执行 `git commit --amend; git rebase --continue` 就行了。

```
$ git rebase --continue
停止在 7a98ea7... commit 2
您现在可以修补这个提交，使用

  git commit --amend

当您对变更感到满意，执行

  git rebase --continue

$ git commit --amend --author="Zhao Yujiu <yujiuzhao@gmail.com>" --no-edit
[分离头指针 e2546e9] commit 2
 Author: Zhao Yujiu <yujiuzhao@gmail.com>
 Date: Fri Apr 24 13:37:06 2020 +0800
 1 file changed, 1 insertion(+)
$ git rebase --continue
Successfully rebased and updated detached HEAD.
```

### 3. 推送远端
检查一下我们的修改。
```
$ git log
commit fa1765984ad3623c39f4b3fea62fe897af6f30a5 (HEAD)
Author: Zhao Yujiu <yujiuzhao@gmail.com>
Date:   Fri Apr 24 13:39:59 2020 +0800

    commit 3

commit e2546e9342cb4a9688dd545dbae59c2e97160aad
Author: Zhao Yujiu <yujiuzhao@gmail.com>
Date:   Fri Apr 24 13:37:06 2020 +0800

    commit 2

commit c96e1437efe3e5a3bc9fe1cda49db03682524f01
Author: Zhao Yujiu <yujiuzhao@gmail.com>
Date:   Fri Apr 24 13:36:26 2020 +0800

    commit 1
(END)
```
`git push origin master` 推送远端仓库搞定。

