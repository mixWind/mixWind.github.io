---
title: mobaxterm
date: 2020-04-27 18:44:02
categories:
tags:
---

## MobaXterm
下面 MobaXterm 都以 mxt 简称。

mxt 给我们提供了一个 Windows 下的 shell 这个真的是个惊喜了啊。
默认的命令行是 bash 还有 zsh 和 win 下的一些命令行可选。

这里我推荐还是 bash 吧，zsh 体验略差，不过可以关注下。

官方支持很多 Linux 命令在，可以在 bash 下 `help` 查看。


## 1. 基础配置
点击上方 Settings -> Configuration 进入配置
![setting](*/setting.jpg)

### General
前面我们说了 mxt 实现了类 Unix shell，所以它也构建了一套 rootfs
默认是在 C 盘创建的。好奇的宝宝们 `df -h` 自行观察挂载点哈。

当然这种方案我们肯定不同意啊，改造之：
这里我是在 D 盘下创建目录 AmobaxtermRoot 。
名字你们随意，我个人是建议以数字或'A'开头，因为这个目录会被高频访问。
置顶方便找它，直接放 D 盘下也是这个考虑。

至于我为啥不以数字开头，那啥职业病哈。

![create dirs](/post_images/2020-04-27-mobaxterm/createDir.jpg)
在 AmobaxtermRoot 下再创建三个目录：
- home  我们的家目录，主要的工作区所以单拿出来
- slash 根目录，home 外的都在这里啦
- logs  放 mxt 会话日志的地方

然后我们配置一下挂载点
![general settings](/post_images/2020-04-27-mobaxterm/generalSetting.jpg)

重启 mxt 就可以在 我们的 home 目录下看到东西啦。当然我们这里先不重启,还要接着配置。
![home](/post_images/2020-04-27-mobaxterm/homeDir.jpg)

### Terminal
### SSH
### Display
### Misc





## 2. 进阶


vim syntx highline
实测 


## 3. Cygwin
