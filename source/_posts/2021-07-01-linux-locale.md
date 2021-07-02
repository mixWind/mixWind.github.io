---
title: Linux 语言环境 
date: 2021-07-01 19:02:09
categories:
	linux
tags:
	- linux
---

zh_CN.UTF-8,  en_US.UTF-8 到底是什么含义呢？有啥区别？

<!--more-->

locale 的命名格式：

```
language[_territory][.codeset][@modifier]
```

where *language* is an [ISO 639 language code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes), *territory* is an [ISO 3166 country code](https://en.wikipedia.org/wiki/ISO_3166-1#Current_codes), and *codeset* is a [character set](https://en.wikipedia.org/wiki/Character_encoding) or encoding identifier like [ISO-8859-1](https://en.wikipedia.org/wiki/ISO/IEC_8859-1) or [UTF-8](https://en.wikipedia.org/wiki/UTF-8). See [setlocale(3)](https://man.archlinux.org/man/setlocale.3).



UTF-8 编码大家当然都是一样的，en_US.UTF-8 自然也能正确显示汉字，主要区别在于语言习惯的不同。

例如，美元$ ，元￥，还有日期的格式等等。



`locale` 查看当前的语言环境

`locale -a` 查看本机安装的语言环境

```
> locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```



## 字段解释

**LANG** 是默认的语言环境，它工作的前提是 **LC_ALL** 没有设置。上面 `locale` 输出的配置中如果 **LC_ALL** 设置了那么其他的字段都是无效的，以 LC_ALL 为准。



其余的 **LC_*** 就是具体作用的字段了，含义如下：

- **LC_CTYPE**， Character classification and case conversion

- **LC_COLLATE** ，Collation (sort) order

- **LC_TIME** ，Date and time formats
- **LC_NUMERIC** ，Non-monetary numeric formats
- **LC_MONETARY** ，Monetary formats
- **LC_MESSAGES** ，Formats of informative and diagnostic messages, and of interactive responses
- **LC_PAPER** ，Paper size
- **LC_NAME** ，Name formats
- **LC_ADDRESS** ，Address formats and location information
- **LC_TELEPHONE** ，Telephone number formats
- **LC_MEASUREMENT** ，Measurement units (Metric or Other)
- **LC_IDENTIFICATION** ，Metadata about the locale information

`locale -k 字段` 可以查看具体的 key-value 



## 对比

俺还是比较好奇具体有哪些差异，所以来做个对比吧。



可用下面这个脚本收集 zh_CN.UTF-8,  en_US.UTF-8  具体的配置：

```bash
CATS="LC_CTYPE LC_COLLATE LC_TIME LC_NUMERIC LC_MONETARY LC_MESSAGES LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT LC_IDENTIFICATION"

LC_ALL=en_US.UTF-8
locale -k $CATS > en.out

LC_ALL=zh_CN.UTF-8
locale -k $CATS > zh.out

```

然后我们可以 `diff en.out zh.out` 对比一下两者的区别：

```diff
❯ diff en.out zh.out
1c1
< ctype-class-names="upper";"lower";"alpha";"digit";"xdigit";"space";"print";"graph";"blank";"cntrl";"punct";"alnum";"combining";"combining_level3"
---
> ctype-class-names="upper";"lower";"alpha";"digit";"xdigit";"space";"print";"graph";"blank";"cntrl";"punct";"alnum";"combining";"combining_level3";"hanzi"
7c7
< ctype-map-offset=86
---
> ctype-map-offset=87
50,58c50,58
< abday="Sun;Mon;Tue;Wed;Thu;Fri;Sat"
< day="Sunday;Monday;Tuesday;Wednesday;Thursday;Friday;Saturday"
< abmon="Jan;Feb;Mar;Apr;May;Jun;Jul;Aug;Sep;Oct;Nov;Dec"
< mon="January;February;March;April;May;June;July;August;September;October;November;December"
< am_pm="AM;PM"
< d_t_fmt="%a %d %b %Y %r %Z"
< d_fmt="%m/%d/%Y"
< t_fmt="%r"
< t_fmt_ampm="%I:%M:%S %p"
---
> abday="日;一;二;三;四;五;六"
> day="星期日;星期一;星期二;星期三;星期四;星期五;星期六"
> abmon="1月;2月;3月;4月;5月;6月;7月;8月;9月;10月;11月;12月"
> mon="一月;二月;三月;四月;五月;六月;七月;八月;九月;十月;十一月;十二月"
> am_pm="上午;下午"
> d_t_fmt="%Y年%m月%d日 %A %H时%M分%S秒"
> d_fmt="%Y年%m月%d日"
> t_fmt="%H时%M分%S秒"
> t_fmt_ampm="%p %I时%M分%S秒"
66c66
< time-era-entries="S"
---
> time-era-entries="▒e"
70c70
< first_weekday=1
---
> first_weekday=2
74c74
< date_fmt="%a %d %b %Y %r %Z"
---
> date_fmt="%Y年 %m月 %d日 %A %H:%M:%S %Z"
76,77c76,77
< alt_mon="January;February;March;April;May;June;July;August;September;October;November;December"
< ab_alt_mon="Jan;Feb;Mar;Apr;May;Jun;Jul;Aug;Sep;Oct;Nov;Dec"
---
> alt_mon="一月;二月;三月;四月;五月;六月;七月;八月;九月;十月;十一月;十二月"
> ab_alt_mon="1月;2月;3月;4月;5月;6月;7月;8月;9月;10月;11月;12月"
80c80
< grouping=3;3
---
> grouping=3
84,85c84,85
< int_curr_symbol="USD "
< currency_symbol="$"
---
> int_curr_symbol="CNY "
> currency_symbol="￥"
88c88
< mon_grouping=3;3
---
> mon_grouping=3
97,99c97,99
< p_sign_posn=1
< n_sign_posn=1
< crncystr="-$"
---
> p_sign_posn=4
> n_sign_posn=4
> crncystr="-￥"
101c101
< int_p_sep_by_space=1
---
> int_p_sep_by_space=0
103c103
< int_n_sep_by_space=1
---
> int_n_sep_by_space=0
106,107c106,107
< duo_int_curr_symbol="USD "
< duo_currency_symbol="$"
---
> duo_int_curr_symbol="CNY "
> duo_currency_symbol="￥"
115c115
< duo_int_p_sep_by_space=1
---
> duo_int_p_sep_by_space=0
117,119c117,119
< duo_int_n_sep_by_space=1
< duo_p_sign_posn=1
< duo_n_sign_posn=1
---
> duo_int_n_sep_by_space=0
> duo_p_sign_posn=4
> duo_n_sign_posn=4
130,133c130,133
< yesexpr="^[+1yY]"
< noexpr="^[-0nN]"
< yesstr="yes"
< nostr="no"
---
> yesexpr="^[+1yYｙＹ是]"
> noexpr="^[-0nNｎＮ不否]"
> yesstr="是"
> nostr="不是"
135,136c135,136
< height=279
< width=216
---
> height=297
> width=210
138c138
< name_fmt="%d%t%g%t%m%t%f"
---
> name_fmt="%f%t%g%t%d"
140,143c140,143
< name_mr="Mr."
< name_mrs="Mrs."
< name_miss="Miss."
< name_ms="Ms."
---
> name_mr="先生"
> name_mrs="太太"
> name_miss="小姐"
> name_ms="女士"
145,156c145,156
< postal_fmt="%a%N%f%N%d%N%b%N%h %s %e %r%N%T, %S %z%N%c%N"
< country_name="United States"
< country_post="USA"
< country_ab2="US"
< country_ab3="USA"
< country_car="USA"
< country_num=840
< country_isbn="0"
< lang_name="English"
< lang_ab="en"
< lang_term="eng"
< lang_lib="eng"
---
> postal_fmt="%c%N%T%N%s %h %e %r%N%b%N%d%N%f%N%a%N"
> country_name="中华人民共和国"
> country_post=""
> country_ab2="CN"
> country_ab3="CHN"
> country_car="CHN"
> country_num=156
> country_isbn="7"
> lang_name="中文"
> lang_ab="zh"
> lang_term="zho"
> lang_lib="chi"
158,161c158,161
< tel_int_fmt="+%c (%a) %l"
< tel_dom_fmt="(%a) %l"
< int_select="11"
< int_prefix="1"
---
> tel_int_fmt="+%c %a %l"
> tel_dom_fmt="0%a %l"
> int_select="00"
> int_prefix="86"
163c163
< measurement=2
---
> measurement=1
165,167c165,167
< title="English locale for the USA"
< source="Free Software Foundation, Inc."
< address="https://www.gnu.org/software/libc/"
---
> title="Chinese locale for Peoples Republic of China"
> source=""
> address=""
172,173c172,173
< language="American English"
< territory="United States"
---
> language="Chinese"
> territory="China"
177,178c177,178
< revision="1.0"
< date="2000-06-24"
---
> revision="0.1"
> date="2000-07-25"
```



 ## 参考

https://wiki.archlinux.org/title/locale

https://www.baeldung.com/linux/locale-environment-variables

https://pubs.opengroup.org/onlinepubs/7908799/xbd/envvar.html

https://docs.oracle.com/cd/E23824_01/html/E26033/glmbx.html
