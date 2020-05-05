---
title: 分享我的 MobaXterm 配置
date: 2020-04-27 18:44:02
categories:
- 工具
tags:
- windos
- tools
---

## MobaXterm
下面 MobaXterm 都以 mxt 简称。

mxt 给我们提供了一个 Windows 下的 shell 这个真的是个惊喜了啊。
默认的命令行是 bash 还有 zsh 和 win 下的一些命令行可选。

这里我推荐还是 bash 吧，zsh 体验略差，不过可以关注下。

官方支持很多 Linux 命令在，可以在 bash 下 `help` 查看。


## 1. 基础配置
点击上方 Settings -> Configuration 进入配置
![setting](/post_images/2020-04-27-mobaxterm/setting.jpg)

### General
前面我们说了 mxt 实现了类 Unix shell，所以它也构建了一套 rootfs。默认是在 C 盘创建的。好奇的宝宝们 `df -h` 自行观察挂载点哈。

当然这种方案我们肯定不同意啊，改造之：
这里我是在 D 盘下创建目录 AmobaxtermRoot 。

名字你们随意，我个人是建议以数字或'A'开头，因为这个目录会被高频访问。置顶方便找它，直接放 D 盘下也是这个考虑。

至于我为啥不以数字开头，那啥职业病哈。

![create dirs](/post_images/2020-04-27-mobaxterm/createDir.jpg)
在 AmobaxtermRoot 下再创建三个目录：
- home  我们的家目录，主要的工作区所以单拿出来
- slash 根目录，home 外的都在这里啦
- logs  放 mxt 会话日志的地方

然后我们配置一下挂载点
![general settings](/post_images/2020-04-27-mobaxterm/generalSetting.jpg)

重启 mxt 就可以在 我们的 home 目录下看到东西啦。当然我们这里先不重启,还要接着配置。

瞄一下家目录，还是很清爽的。
![home](/post_images/2020-04-27-mobaxterm/homeDir.jpg)

Desktop 是 windows 桌面
LauncherFolder 是 mxt 的安装目录，mxt 的插件都是要放到这里的，所以这么一个链接也是很贴心的。
MyDocuments 我的文档

不说了，真的良心。

### Terminal
接下来我们配一下终端的默认设置，你肯定不想每个会话都单独设置一遍吧。
![terminal settings](/post_images/2020-04-27-mobaxterm/terminalSetting.jpg)

log 功能不常用就不要勾了会话里需要再开也行。还记得我们上面创建的三个文件夹吗，logs 就是为这里准备的。

呐，一家人就要整整齐齐~

### SSH
![ssh settings](/post_images/2020-04-27-mobaxterm/sshSetting.jpg)
主要是勾选 keepalive 断线还是蛮影响心情的，其他随意。

那个 Remote-monitoring 会在会话下方列出远端的一些资源使用情况，这个单独配就好了，不用默认勾选。我一般连虚拟机会开，服务器就看心情了，毕竟服务器卡了我这暴脾气早就 top free iostat 开整了。

### Display
这里没啥好说的，看个人品味了。

我只说一句，半透明真香。
![ssh settings](/post_images/2020-04-27-mobaxterm/displaySetting.jpg)

### Misc
![ssh settings](/post_images/2020-04-27-mobaxterm/miscSetting.jpg)

## 2. 进阶


vim syntx highline
实测 


## 3. Cygwin
