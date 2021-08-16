> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5qAfYxDNpKbT3fy786fo4w)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **94** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

靶机地址：https://www.hackthebox.eu/home/machines/profile/1

靶机难度：初级（4.3/10）

靶机发布日期：2017 年 10 月 9 日

靶机描述：

Lame is a beginner level machine, requiring only one exploit to obtain root access. It was the first machine published on Hack The Box and was often the first machine for new users prior to its retirement.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibTI8P0OCvaU2icfPvr6ngOUrG4x8apyuFDgF1ttCo0xsUCLUTIPv5zokZMnTuq7Fn27raOTY9w1fGot0Tq2D01g/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBpKSrnWvyzBa3oCDxVHAIn2NV4YUVcokLqB2hbibJTSIcozjy1RMlOlmw/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.3....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBpbLdZ57SflDID6wpn41gwWzHA00Lz1GoHHPRLDAQruxvpTQwghKBnHA/640?wx_fmt=png)

Nmap 发现 FTP：vsftpd 2.3.4，OpenSSH 和 Samba（3.0.20）都开放着... 这里的版本很低，都存在很多漏洞...

利用这些漏洞试试吧...

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ibTI8P0OCvaU2icfPvr6ngOUrG4x8apyuFDgF1ttCo0xsUCLUTIPv5zokZMnTuq7Fn27raOTY9w1fGot0Tq2D01g/640?wx_fmt=png)

二、提权 

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBpIjUdZcIwyEvKJH2icpxVD4Feic2rWzAYZDswHlialyXZldGu3bibwOA1Lg/640?wx_fmt=png)

直接 kali 搜索版本存在可利用的 EXP 即可...（这里可以直接修改 EXP 内容，直接执行即可提权...）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBpB182zfEG2ork5vArZFhTsYicN1T86aJOrAR6BZBdweibjBZjA28M6U5w/640?wx_fmt=png)

渗透久了，直接利用 MSF 几分钟就搞定了... 开始

可以看到利用 MSF 找到了可利用的 exploit...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBpxicHpqS5Y7nibWou6Z3PGd09s8HjGlib87A3u1KCicJOvPibBT07QkKdVbA/640?wx_fmt=png)

这里没有获得 session... 查了下对于该漏洞在网上的解释...

根据 google 搜索发现，此后门已在 2011 年 6 月 30 日至 2011 年 7 月 1 日之间引入了 vsftpd-2.3.4.tar.gz 中，该后门程序于 2011 年 7 月 3 日已经被删除无法利用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBpvjfGtJFUJ6tJAkebH0tJVZSBpquvjTonIicWOLAWJXwGkibskML3jwIQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBp6FoJTdHzZoo5l6lvtGEZC47FQsRrhI3KXKUb9mQUGLXPzzHKqYtZyA/640?wx_fmt=png)

这里直接利用 mulit 目录的 EXP 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBproKTwO2rLC5FUIwFNiaqXpibnBGTIV6CRpNqE10xwTP056NaqFgFkeGw/640?wx_fmt=png)

很简单，获得了 root 权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPbX8iagjAJHm0ltAHfNgSBp8K8bUNMUmpWzAb62p97CiaUjfXQwDtAP75mpRIyQjjxA7yunicmR1OZQ/640?wx_fmt=png)

轻松获得了 user 和 root 信息....

很简单的一台靶机... 我初学的时候就是在这环境下理解了很久，当时是 vulnhub 的靶机....

现在操作起来，行云流水，也理解了很多原理... 感谢以前自己的坚持....

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台简单的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/nLSlp1hFMwsA5K49ZxLYXtEibosgIZkIlYqhXqx03XbFqNrVBAm5axMu7OLyRv4RXEQdpzTBFs5wfhaLhvqw41Q/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

随缘收徒中~~ **随缘收徒中~~** **随缘收徒中~~**

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)