> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hNlpIi28O8PLE0RdDuOMoA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **33** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/web-developer-1,288/

靶机难度：初级（CTF）

靶机发布日期：2018 年 11 月 5 日

靶机描述：

Zico 试图建立自己的网站，但在选择要使用的 CMS 时遇到了一些麻烦。在尝试了一些受欢迎的方法后，他决定建立自己的方法。那是个好主意吗？

提示：枚举，枚举和枚举！

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn90BudSia3UAT9rnRxEhRLOVr288vhVSNp6KJa1408BvVNTAX6Z4GhnQ/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznj6TPicibplibEj9iaY7ED4RXhOp1p0WmaoZqHZIXCcHxp2dLHtNErJvIzA/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：

```
192.168.56.129
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznficlVB86yStHNqWbygfc9zLbo1gssZwDHS10fvyNHVYDjG1dEljiczmg/640?wx_fmt=png)

nmap 看出开了 22 和 80 端口... 还是 wordpress 框架...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznIpicrWVZTFB6gueVnZpV3mmoDHUUSRnVibg0dbPoND6NFo1I5v4EjHyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznicVcI8fyiaGK2mvIlGDdjZAiabMWS6bfn45B97hzOApFlpwFIHLPGYLIw/640?wx_fmt=png)

前面章节的靶机 web 也是这个界面... 估计是同一个作者...（回看发现是 oscp 教材视频里的一个靶机... 快速做了...）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn1QuD6INAo98ibdaMykOwKomxkRgtQaKFEsjMLw2icS07MXXx34QdpS1g/640?wx_fmt=png)

发现了几个目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznzDCOhh1CD2pD2XKQ27WIMgGGZHjQfBUdpSM8LI70H8d6njV90uLkXg/640?wx_fmt=png)

这是个数据包，下载用 Wireshark 打开...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznibEyXtOPspNTCeiboSLrP2MEbR6HYURjQFoWUY5uDcxDqVEwtkFtaNicQ/640?wx_fmt=png)

在 180 位置发现了 log 和 pwd 用户密码...

webdeveloper 和 Te5eQg&4sBS!Yr$)wf%(DcAd

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn10eOtbYK4wMxgFYX1t26uAPdeV5lgGOQOSjpdfHicI8srGaJx8Ksmew/640?wx_fmt=png)

成功登录... 发现可以上传文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznd3mQpPuZ7KwROxyWKXXWLQlZ3LtibHtZ9FtZUT5LW22Gge4gy4DcqdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznzS1InjWhicGrWu8icpPskxwKK5mbV6F5YSJz1F2RU71kEcIwoByjzXpw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznJKDV5ldiaZdwh66OVntlef1mDNMwzwteVpxv4wiaF02RwyodNicA4TmzA/640?wx_fmt=png)

本想写段木马进去，提示无法写入，只能上传 PHP...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznlAowCcyWJyzo5X4k49a1picJPdfT4LSpdAD2JP1DvPtcX6hpx5V395A/640?wx_fmt=png)

切换到 twenty sixteen 后，发现可以写入了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznzcAWtbFIMKrRLDh3kF58icaFl4kWicwPrI3IUROfCvEKtaKmw3UIzRKg/640?wx_fmt=png)

成功获取低权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn9eFVLiagEH7QAo78FrczJktExP1UrVUaBlaOavsACjNHPYibodcRQatw/640?wx_fmt=png)

或者在这里添加插件进行提权... 应该很多种方法...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznb0bicVGVc5cibW1gPmTR0t2AkwlshLejZKpr8jTRYZibGEQibJr79HD4LQ/640?wx_fmt=png)

不允许...sudo 和进入 tty...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznj4BD3NAOBXibzqonEFJ29plBEvblX9Yibb4jgCz24X7MPiasHib0L9gjHQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzniczk3xVKcA2kKOuRTSEbyYzMkSR1xIvhibIGmoCf5ODF5MaCLiaB5icEqQ/640?wx_fmt=png)

webdeveloper 和 MasterOfTheUniverse

登陆...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzntuEAwGOcDrvhYc3cyfUraPeWVDPic7lbUx7iayljuQGSpXJiaMpUQH1qA/640?wx_fmt=png)

前面章节介绍过怎么使用 tcpdump 提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn6OUPvoX6DSuug9rLQP4DHSN4bzc8YlvlHLFIDLMPa2Y4SsHLoLWicvw/640?wx_fmt=png)

本想用 nc -e 去提权... 哪晓得这靶机装的 nc 版本太低了... 不支持 - e...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn4WibUnj07FsUoDkvDzKQ86rXANNZeGibbm8hsS53zib2qnAHCMzZLsEAQ/640?wx_fmt=png)

```
[参考](https://github.com/xapax/security/blob/master/privilege_escalation_-_linux.md)
```

这边方法还是一样的... 只不过调用了原来的 404.php 文件进行提权...

这边将网站 404.php 写入本地 shell，然后命令和以前一样执行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznXmpWXZqhpYLwSVPGKia6wA8m5Y739CWOtx4V10UQj4wzzKY6fLPgdKg/640?wx_fmt=png)

记得本地开启 nc....

成功获取 root 和 flag 文件... 很熟悉这台靶机，我记得在哪里做过类似的...

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

由于我们已经成功得到 root 权限，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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