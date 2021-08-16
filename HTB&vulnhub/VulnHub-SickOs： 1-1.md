> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/peFQC7vcpqs--dx6z0FRFw)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **17** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/9Ku2t1uaSwDSnPM80libzbs8ofzicXbQesVN9mGMnqZPxqCS8gUoHLkVWJkEPByShv4ul050UUYX4Phfnnc5lLJQ/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/sickos-11,132/

靶机难度：中级（CTF）

靶机发布日期：2015 年 12 月 11 日

靶机描述：这个 CTF 明确地比喻了如何在网络上执行黑客策略，以在安全的环境中危害网络。这个虚拟机与我在 OSCP 中遇到的实验室非常相似。目的是破坏网络 / 计算机并在其上获得管理 / 根目录特权。

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iaRSfuVibrT80ovdTMiaZM0gZOFHmtS8KYVtVHtFNaz9XcENaibWibgmw8JIn1niaCurDOrBCjUbQ8az7fdzNMu8kTrw/640?wx_fmt=png)

  

  

一、信息收集

  

  

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4slSlQlQNhBwBoKeZovdat1ticQ503GuX5A1zgwIicvHPiaFwvl8YZV69w/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：

```
192.168.56.109
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4Mz86D21h9OYeOX2dZ5paedDiaUhW886IfXZxUPhlLORicNhPz7XONsyQ/640?wx_fmt=png)

命令：

```
nmap -sS -sV -T5 -A -p- 192.168.56.109
```

这边开放了 22、3128、8080 端口

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa455MhiavW4IQYuDrR4KlIoytUx0gibmp7lWuNUticqxoYrXHK0wQkAyKZQ/640?wx_fmt=png)

用 nikto 扫描出很多信息... 透露了 / cgi-bin/status 位置（容易受到 shellshock 攻击），这边还存在 robots.txt，有一条思路是直接用 BP 拦截通过 / cgi-bin/status 发送反弹 shell 直接获得权限... 知识量不足... 这边先查看 robots.txt 吧

这边先用开启代理

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4dDT01KS4rib5UxsJ5Iu7PrHtvKP8DBlFSDXz2k5x5TxgVOId9uJicz1g/640?wx_fmt=png)

然后去访问他的 80 端口

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4DlF1N4vTxr0yBice3LjnuuVHcjYJmlxSy1tXBNo8QS8GJZjByIHPFWg/640?wx_fmt=png)

答对了~~ 查看源代码没啥信息... 这边是用代理来访问的 80，如果不用代理 80 是没打开的，打开 robots

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4BPf7uRTpBdicic7ZKERtWYym5xSS1hONvGnRfLvzROolXp8xOKeuaRkg/640?wx_fmt=png)

找到 / wolfcms 的信息，有可能此网站是在 Wolf CMS 中建立的，也有可能就是个目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4rQWdMylGKRumxo3AS10aJib2j4YX36ecJia7mKCohsyibSDl1jaGR9t3A/640?wx_fmt=png)

果然是 Wolf cms 程序（PHP 编写的轻量级 cms 程序），这边需要找到登录界面，网上找了下别人登录 cms 的链接

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4PzksOz64kFJKuwiasLjyonQpMMqKe5EIEr8V3AJItHSz7zSRlrJJfYA/640?wx_fmt=png)

访问下 /?/admin/login，这是通病... 遇到 Wolf cms 直接能使用这个目录进入登录界面

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4KRJasu7icsscjoq9UnjP3yAXn8YesTIxWn8QibpTkna9S1qdQZoJkqCg/640?wx_fmt=png)

尝试 admin/admin 轻松登录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4G712I9U8kTeYrb4dj4V46P5ibGGING04Ft6GkYbQ3PianWXT5cpFU3XA/640?wx_fmt=png)

成功登录后，发现能上传文件，这边要使用 msfvenom 做 PHP（muma）文件上传进行 shell 提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4LWqvIFyzGeT4joRh5j6vGTGWRKddTL8Yric1SYv23Crj4tiaib0Ljpzvg/640?wx_fmt=png)

这边解释下原理，图上可以看到，通过 IP 和端口写 php 马！！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4UxP0lAzz7d25micOCljvIib1tENdKVZEQZhs6WqL3nT7WuK5wvytBACA/640?wx_fmt=png)

这边命令就不复制了，多敲打就熟悉了...

可以看到 payload 成功创建，这边把创建个 PHP 文件把代码放进去即可

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4JlNynibE0F4Iv6sniaicBLqgZ1fJgSiar7fzcVMDWonyqibteN0XTcIBDOQ/640?wx_fmt=png)

然后我们去 msfconsole 去执行

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4yV1wQQjj4Ur9LwMdibU5Nj8BAjYKqRkrxeA9UQRMGbYl3REPqoW9qlQ/640?wx_fmt=png)

执行 exploit，然后去 cms 上传 shell.php 即可

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4e71ALdz7xt2UdibDgux3g8aMnNTaO6ic0eYjm4YqlLjKR23H33N7o9vA/640?wx_fmt=png)

上传成功，出现目录：public / 访问

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa47r0meeFPJFmOcSDric4ibXVxIhKF5gWFfko6MXJckavHPvq39p8gibTaA/640?wx_fmt=png)

可以看出是刚上传的，这边点击下 shell.php 运行它！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4lywZVe3qbfnTophsZPCa4IibCvEKMudHPhNYWUlpTqZkqHbGbMyUiaqA/640?wx_fmt=png)

OK... 经典的文件上传提权成功，继续提 root 权限

不多介绍，直接 python 进入 tty

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4ej5OEGibsoFyzpibiahwSb0aL0l7bHEJvoWrsYoPkGMib5hVlKiaW31Ypng/640?wx_fmt=png)

直接去找 config.php 文件，里面包含了用户名密码！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4vmKib0HHC7baXm4jfOsAgVUGS6p9bUNpz6UhH01E1Otelia38cSmxl7g/640?wx_fmt=png)

找到了，继续查看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa443icp2hS9sv7YZjkfNf2vc1pTyaXrWFQibbjiamUoCelPuUa5icZCWsFbw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4cibGISVSnkvodStnz7y1zTmIJf4dTopickeJ4I4BZwotibNibEE8X0IcUg/640?wx_fmt=png)

发现直接 root 进不去... 回看后发现

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4nVlMEEsdSb6vOiahg13ibDFoibqJA0y7DnK3j24qfZBE67ner9KCmqcCg/640?wx_fmt=png)

```
sickos:x:1000:1000:sickos,,,:/home/sickos:/bin/bash
```

意思是 sickos 是第一用户，那直接访问

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4LsW2RFa6k2fYTjFm6rsdKE1XE0HLz91qAyYNDI0h9bTdpzV9NOW1dQ/640?wx_fmt=png)

继续提权，这权限不对...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOfScGXoTuYLfxWatPzLNa4hOZibg9vb5reuTsQADyTccibcS8mVdYKkCRvhUoPHH5UKeS1HYw1vu4Q/640?wx_fmt=png)

成功提权...Thanks for Trying！！！

![](https://mmbiz.qpic.cn/mmbiz_png/9Ku2t1uaSwDSnPM80libzbs8ofzicXbQesVN9mGMnqZPxqCS8gUoHLkVWJkEPByShv4ul050UUYX4Phfnnc5lLJQ/640?wx_fmt=png)

文件上传，vulnhub 的逆向工程、提权等等，还是不错的一台靶机！

由于我们已经成功得到 root 权限 & 找到 flag.txt，因此完成了 CTF 靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iaRSfuVibrT80ovdTMiaZM0gZOFHmtS8KYVtVHtFNaz9XcENaibWibgmw8JIn1niaCurDOrBCjUbQ8az7fdzNMu8kTrw/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)