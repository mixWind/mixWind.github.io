---
title: MobaXterm 配置
date: 2020-04-27 18:44:02
categories:
- 工具
tags:
- windows
- tools
---





Win 下的 SSH 客户端我迁移到了 MobaXterm。



本文主要安利下这个软件，顺便分享下我的一些配置。

<!-- more -->

## MobaXterm

下面 MobaXterm 都以 mxt 简称。

mxt 的一些[特性在这](https://mobaxterm.mobatek.net/features.html)，我也懒得吹了自己看吧，整不明白的网上的安利贴也不少自己搜吧。我所谓的安利就是让你知道到关键词，有这么个东西可以一试，合不合适自己用了才知道。



当然我看中的功能还是简单提一嘴：

- 协议支持全，功能集成多，基本需要的都集成了
- 自带两分屏和四分屏，省去 tmux 了
- 图形化 SSH 隧道管理
- 自身就是个 shell，在 windows 下支持了一票 linux 命令
- 最多 12 个会话
- 支持录制宏，最多4个
- 密码管理好用
- 可以加[插件](https://mobaxterm.mobatek.net/plugins.html)增强自己



mxt 是免费和开源的，有家庭版（免费）和专业版（收费）。免费的功能有限制，但是我感觉基本够用，唯一感到蛋疼的限制是它的录制宏免费版只支持 4 个。关于它的宏我们后面再吐嘈。当然充值那就没事了。



mxt 给我们提供了一个 Windows 下的 shell 这个真的是个惊喜了啊。
默认的命令行是 bash 还有 zsh 和 win 下的一些命令行可选。

这里我推荐还是 bash 吧，zsh 体验略差，不过可以关注下。

官方支持很多 Linux 命令，可以在 bash 下 `help` 查看。




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



## 2. 踩坑记录

### 2.1 vim

既然命令行有了，那我们自然想要个 vim 了，vim 自带就有，Emacs 可以安装插件支持。其实本文就是在 mxt 下用 vim 写的，当然体验不是太好，能用吧。



我把自己的 .vimrc 搞了一份到家目录，亲测基本能支持，当然把那些插件都去掉了哈，只留下单纯的 vim 配置，本来也没必要搞那些，我们的定位就是顺带改个东西、看个东西用。



这里面有个坑是语法高亮 `set syntx on` 会报错。我还稍微调查了一下，Linux 下 vim 的语法高亮其实是在安装目录下有个文件夹存放了各种文件类型对应的语法高亮方案，这个目录下还有一些相应的脚本，我们使能语法高亮vim就会执行脚本导入这些方案。所以我怀疑是打包的这个vim没有这个文件夹或路径不对。



当然我不打算深究下去了，因为我发现注释掉高亮，依然可以有高亮支持，但是我不确定这个高亮是 vim 的还是 mxt 全局支持的。



### 2.2 输入法

公司的电脑是 Win7，mxt 下用搜狗输入法或者小狼毫都会有个问题就是按 ** Backspace ** 在输入法框内不能正确的删除字符，但是实际上是删掉了，也就是说显示上有错位。这个问题不影响继续输入但是会有点别扭。



我开始怀疑是 backspace 的配置问题但是修改之后问题还是存在，我家里电脑用的 Win10 自带的输入法没有这个问题。精神洁癖我也不想在我电脑上装搜狗来对比了。所以我目前的方案是输错了就 Esc 清掉重输，顺便练下打字正确率。



唉，弗难人拼音难呐。



### 2.3 宏

mxt 这个宏必须吐嘈，官方既然把这个功能限制到 4 个那也从侧面表示了他们看重这个功能，觉着是个好东西。



录制没啥问题。问题在于我人打字就这么快，结果就是录出来的宏慢的一匹，当让 mxt 也给了解决的办法可以编辑这个宏，那我就可以修改下sleep的时间，改时间的话有些交互命令执行时间其实也不好把握。



既然可以编辑宏，那问题就来了，能不能直接开放编辑，非得先去录个宏？咋想的呢。



所以这么设计就把这功能局限在避免重复输入这一个点上了。



但其实用到这种功能大部分还是应付交互式的输入场景。单纯嫌麻烦不想敲大串命令我直接远端弄个脚本不就完了嘛。



所以真正缺的是 XShell 的脚本功能，虽然它那脚本用python写咋能正确退出我至今没闹明白，但它 vbs 起码能用啊。



## 3. Cygwin

翻了下 mxt 的源码目录，以及它的 apt 源，我猜这玩意是基于 [Cygwin 项目](https://www.cygwin.com/) 搞的。这个项目好像还蛮好玩的，意外有点小惊喜。



就像一本好书会推荐给你一堆好书，好的项目也是。



我就 BB 这么多，enjoy ~
