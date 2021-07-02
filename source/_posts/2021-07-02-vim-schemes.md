---
title: vim 修改配色方案
date: 2021-07-02 17:45:46
categories:
- vim
tags:
- vim
---

vimdiff 默认的配色伤害太高，我们来换掉它吧。

<!--more-->

## 配置颜色

通过 `colorscheme` 命令可以临时修改配色

```
:colorscheme [配色方案]
```

想要查看所有的配色方案可以：

```
:colorscheme [space] [Ctrl+d]
```

选一个自己喜欢的就行


我们只想针对 vimdiff 做出配色修改，可以在 `vimrc` 中配置：

```
if &diff
	colorscheme murphy
endif
```



## 更多配色

网络上有很多制作好的配色方案，[GitHub](https://github.com/)， [vimcolors](https://vimcolors.com/).

下好的配置请放到 `~/.vim/colors` ，没有就新建一个目录。



## 自定义方案

想要自己设计的话主要靠 `highlight ` 命令，语法如下：

```
:highlight [Group] [key=value]
```

**Group** 指示后面的 k-v 影响哪一类字

**key=value** 指明配色

http://vimdoc.sourceforge.net/htmldoc/syntax.html

**Group** 的取值举例：

- Normal (normal text)
- NonText (characters that don’t exist in the text)
- Cursor (the character under the cursor)
- ErrorMsg (error messages)



**Key** 主要包括前景色、后景色和其他属性，又因 terminal 和 GUI 有区别：

- **ctermfg** (for setting the foreground)
- **ctermbg** (for setting the background)
- **cterm** (for additional properties) 



- **guifg** (for setting the foreground)
- **guibg** (for setting the background)
- **gui** (for additional properties)



fg，bg 的 value 都是颜色，可用的值有 standard color names, their prescribed numbers or hex values (only in the GUI).

第三项 cterm，gui 的取值是：*bold*, *italic*, *underline*, *reverse*, and *none*. 主要就是一些控制字形的信息。



### 例子

```
:hi Normal ctermfg=Red ctermbg=Black
```



## 参考

https://phoenixnap.com/kb/vim-color-schemes

https://stackmirror.com/questions/2019281

