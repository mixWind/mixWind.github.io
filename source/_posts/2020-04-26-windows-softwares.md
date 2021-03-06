---
title: 我的 Windos 必备软件分享
date: 2020-04-26 17:53:46
categories:
- 工具
tags:
- windows
- tools
---



最近把电脑玩坏了，修复引导也没救回来，目测是分区表弄坏了。索性系统全部重装。



由于电脑比较老了，以前是 MBR 的双系统。这次乘机切换到 UEFI，当然整个盘全部格了，干干净净的好处是可以重新梳理下自己的东西，挺好的。以前看过一篇文章作者说每年会重装一次系统，突然有点理解这个行为了。



由于整个重装了，而且手头也没有一份安装软件的 checklist，所以我能想起来的都是缺了就不习惯的。这篇单纯整理下 windows 下我必定会安装的软件，通用软件，工作和个人爱好强相关的略掉。

<!--more-->

个人的偏好是 商业免费软件 = 不贵的付费软件 > 开源软件 > 昂贵的付费软件，对破解软件有中度洁癖。



## 1. [f.lux](https://justgetflux.com/)

调节色温的软件，休仙党必备。



色温和亮度不是一个概念，这里简单科普下吧，非专业不负责讲解。

所谓色温，字面意思颜色的温度，黑体从绝对零度加热会对应不同的颜色，由黑变红变黄变白变蓝，蓝光的能量最高。通过调节色温可以让屏幕整体偏红或偏蓝。色温的单位是 K （开尔文）。



显示器的色温一般偏高，我们调节色温的目的是减少蓝光。

- 一是保护眼睛，手机的护眼模式屏幕会偏黄就是这个原因。
- 二则晚上蓝光会抑制裉黑素的分泌导致不想睡觉（是你晚上玩手机导致睡不着的元凶之一），裉黑素这个话题感兴趣可以自行了解。
- 三嘛，加班时给自己一个温暖舒适的色调，心情会好不少嗷。



白天自然光色温偏高，黄昏偏低。正午的日光色温在 5000K 左右。所以我白天色温会设置到大概 4600K ~ 4800K。晚上看心情，但是不要太红，过低的色温也是不好的。



## 2. [Snipaste](https://zh.snipaste.com/)

截图 + 贴图神器。这个就厉害了，基本见我用过的同事都会打听这是啥。



什么是贴图？资料老是要在多个窗口切换，或者要翻页回去看图表，等等这类场景太熟悉有木有。那贴图就是针对这个啦，用 Snipaste  截的图可以选择贴到屏幕上，贴图就是将你截的图置顶到所有窗口的前面，而且可以自由拖动和缩放。而且不限数量哦。



它的截图功能也很强大，自动选择窗口，各种标注、添加文字、打马赛克，甚至还能取色，一看就是老码农开发的啦。嗯，软件完全免费，是个人开发者业余时间搞的，据说老兄吭哧吭哧打磨了好几年，可惜代码没有开源，不过能够理解。



自从用了 Snipaste ，Bug 群甩锅再也没输过 :D。



有两点遗憾：

- 不支持滚动截图
- Linux 还要等亿小会



## 3. [Ditto](https://ditto-cp.sourceforge.io/)

剪切板管理器，可以浏览和快速粘贴历史复制内容。谁用谁知道。



## 4. [Everything](https://www.voidtools.com/zh-cn/)

就和它的名字一样可以快速搜到任何文件，再也不用打开文件夹一个一个翻了。支持按类型搜索、支持通配符和正则、可以在窗口内操作文件、甚至还能搜文件内容。



## 5. [Bandizip](https://www.bandizip.com/)

一款全能的解压缩软件，装这一款就够了，支持所有格式的压缩包，包括各种 tar 包哦。



## 6. [MouseInc](https://shuax.com/project/mouseinc/)

全局的鼠标手势，鼠标手势这东西吧用惯了缺了就别扭。当然它的功能远不止鼠标手势，具体顺着链接自己看吧。



我确实不是这个的重症用户，主要 win7 下的滚轮穿透，还有就是画 C 召唤计算器哈哈哈哈。它这个也有贴图功能，当然为了贴合鼠标手势的概念强行用鼠标画方框去截图的设计在我看来属实智障。



有的杀软会报毒，这个软件我自己用的也不多，就不推荐了，列出来做个参考吧。



## 7. [Chrome](https://www.google.com/intl/zh-CN/chrome/)

老实说我装好系统的第一件事是翻墙用 Edge 下了个 Chrome，然后开同步。



这年头浏览器都能当操作系统了，Firefox 实在懒得折腾，当然我知道有专门折腾好了给大家开箱即用的项目，Google 也有这这那那的问题，只是 Chrome 用习惯了，总比国内的那些垃圾好吧。



Chrome 和 Firefox 都有所谓的国内版，请远离他们，去真正的官网下载。



用 Chrome 当然是为了用插件啦，列一下我常用的：

### [SwithyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif)

自动切换科学上网，标配了吧。

### [Adblock Plus](https://chrome.google.com/webstore/detail/adblock-plus-free-ad-bloc/cfhdojbkjhnklbpkdaibdccddilifddb)

去广告

### [The Great Suspender](https://chrome.google.com/webstore/detail/the-great-suspender/klbibkeccnjlkjkiokjodocebajanakg)

Chrome 可是吃内存大户，而我动不动会打开二三十个网页，这个插件会冻结那些非活动的页面，节省内存。

### [OneTab](https://chrome.google.com/webstore/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall)

把网页都收集到一个页面列出，也是省内存的利器。此外充当一个临时书签。

### [crxMouse Chrome](https://chrome.google.com/webstore/detail/crxmouse-chrome-gestures/jlgkpaicikihijadgifklkbpdajbkhjo)

鼠标手势，这个才是我真正常用的鼠标手势。

### [Tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)

油猴大名不用说了吧，脚本就不列了。

### [WasteNoTime](https://chrome.google.com/webstore/detail/wastenotime/enebomhlllfaccbelnjhfgblnalofhch)

统计花在各个网站的时间。Waste no time，对自己要诚实哦。

