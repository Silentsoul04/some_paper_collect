> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/d6f8vhFi3OjnvhsiaklLlQ)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **100** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/17

靶机难度：初级（3.5/10）

靶机发布日期：2017 年 10 月 17 日

靶机描述：

Brainfuck, while not having any one step that is too difficult, requires many different steps and exploits to complete. A wide range of services, vulnerabilities and techniques are touched on, making this machine a great learning experience for many.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ibqicBWEiaFhaUkEs3YhX1fKvaBcTc3V7YooNTGXoXQEGE8V3BGstZA0g9OpLlWicaefuM0zBUvxG3mPIlLdP7vnYw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/1prMbIpCa3humOrLAChJmsjMl4Kxia7vzrQE59ny2bGibWz5Cr8YzNvia9NXzt8O2jiclnVwHYxubpFU1Q6dX9FRCQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/5PYA1G5YGGkjAN1M3sw2tjaT2EzjYhfiax6biaK6IUQxeAFY5cgZQtGqXrMp1oRbNic8EDqpxsg5BjArxBhibLM5XQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRDpH5lGmtyEVpkXuGibtib9MH1XlxNqyS7DkTQXAgVPxVLmC5MGWRxEuA/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.17....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRQSBx2u5HFllveVAgnX6HR1L43M7g8yvBICat5BtL139H8lIibtiaTGaw/640?wx_fmt=png)

Nmap 发现了几个开放服务以及通过 SSL 证书枚举的几个主机名，需要将所有的 DNS 主机名添加到 / etc/hosts...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRBDBhEnmHW6BdwpciaRdFgljFRNzoksuicVxFC6LDmbba0N6Eicy2hn6uQ/640?wx_fmt=png)

访问这是个 nginx 页面.. 前面 nmap 就知道存在证书信息... 看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRsDMDmzkJM0ICKoTaAljB8ic6mkWwkSibiaRBStzNuoe731MUIH5TltNiaQ/640?wx_fmt=png)

在证书中，我发现了一封电子邮件地址...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRWiazrmmTTSdawgwlWn6PXQD0kShYl9l3bIvDib9MxyWVsempq9WSWOkA/640?wx_fmt=png)

看到 brainfuck.htb 页面正在运行 WordPress...（这里又可以用 wpscan 了）... 还提示了邮件是通过 SMTP 进行的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRYFRywD684l5YaiaBfKfeGibk2icw8rYeLfP29qFHrp33t4S6oWsmmoUibw/640?wx_fmt=png)

sup3rs3cr3t.brainfuck.htb 页面是一种类似论坛的发言面板...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRmmcrBxzClrZ0Kn3XbHcITFpHxc5liaiagLriaicfGmbQn29ymQQMW0Ojibw/640?wx_fmt=png)

```
wpscan --url https://brainfuck.htb --disable-tls-checks
```

直接利用 wpscan 收集信息... 运行该工具后，该站点似乎正在运行过期扩展名 wp-support-plus-responsive-ticket-system...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRuvaH0uL3siblYa3SviaHShg3gKibERibHH0MK6eVG2emC0XctAv30oLDzQ/640?wx_fmt=png)

searchsploit 查询中，可利用的有特权升级和 SQL 注入利用... 这里利用了 41006...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRH1HCeM5V2y290JcH4YYTlwrUibXHAhFPZbRwBAfTq4TLeZI7icI1YFgw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRrHoZanhJxT3r3l5icicBqZdW59w6IvQSXdVXKUaicoHlwdVbsOPbKNL8A/640?wx_fmt=png)

利用该 exp 简单修改填入信息... 利用即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRicJralw39bfN6Eo6Zk5czvocace54EicL21yHrst5wEd6f6ZTV8kCf9w/640?wx_fmt=png)

开启本地 80 监听... 通过访问本地... 执行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRt16OEAhdjVCQVQuicgsyZsaYSLqgAPRITibibv7aVpjo0RnRbRkJgQDWg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRyKSaDWHQcLOVk2tXfpLfYqyw6m3qAEwcw77JgBCumicY7T4DwxibICYQ/640?wx_fmt=png)

在右上方可以看到该漏洞已起作用，现在已经以管理员身份登录 WordPress 网站，并可以访问管理门户内容...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRgpicmZOUicgf3vVCIecibjZ7ibBphHLg0dWxRLibBWiakxzXNLQiavIFeL1JA/640?wx_fmt=png)

根据前面 SMTP 的信息提示... 直接找到最下方 SMTP 的模块，通过前端源码查看到了用户名和密码....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydR0tmK7rH8Qibvv4pfN2bzwHAsmjC35GJLvlov67YOUZ9Eh62eKpxSicww/640?wx_fmt=png)

```
telnet 10.10.10.17 110
```

通过登陆邮箱，发现了两封邮件... 在最后一封邮件发现了论坛的登录名和密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRmyaNZA3AGiaVhn8gxjgj8yv19zNmOibiaicJCWU8ztczale7WdJRViaHBkg/640?wx_fmt=png)

成功利用登陆后... 发现 key 和 ssh，应该之间通话存在加密行为...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRDMYVclDiaaYUVlkH5OMDbUCTPc19CUAicnwcvYbdaSibe4jPu2xSNmHRA/640?wx_fmt=png)

通过检查发现两者之间的通信已加密.. 可看到 Orestis 使用的特殊签名会在 SSH 访问中找到... 可以对加密文本内容在网站进行解密...

```
http://rumkin.com/tools/cipher/vigenere-keyed.php
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRERxgVYAfg0nh1Q1R1D01wn0D62t6XK8ibbkLj4N0ggeByxhs31VLXhA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydR2TicO58hKibQePiaYlhQT82icibqzjP4zxh4q5HSuo9eq9iag4tNuAFARM1g/640?wx_fmt=png)

通过 key 和 ssh 对话互相解密后... 找到了底层存在的 id_rsa 密匙凭证...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRQEyMP1IqfvEPTF9HicaUPYN2g9xfsKWLD3w7Cp9UhYiceR6lISEym3TA/640?wx_fmt=png)

下载下来查看，这里直接开始解密即可...（熟悉的密匙)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRNIpiaR8icQyBGN4bJicQMfHkJX36xbuwX5JelTTRf1bzjVpdepfia0wuzQ/640?wx_fmt=png)

前几章也讲过类似的环境了... 直接通过 john 解密出了密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydREdBN16tCFpuuEnzu90LRMfiaJRBOvkcPlXbRVYvngbtxticHvZZy44Cg/640?wx_fmt=png)

通过授权，靶机开放的 ssh，成功登陆了 orestis 用户... 获得了 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRgcBq68wRDZAxsG2ic2oib4pOuqTbic0ZVfPJUWxcpGRKXoHaBuqRIyuIw/640?wx_fmt=png)

可以看到在目录底层还存在另外的密匙文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRG4z7DtdHibicgQKVpxJsWQ9UBvoD4tIt9lBDC83Bwiaich1SyESgLticn7w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRmgIWc5B2PVFTAh3NtcsvWLqicMsuLiaYcE8NM4kRQjMfV2BjlSEavuOw/640?wx_fmt=png)

看不懂文件意思... 直接复制 google 搜索，这是转换 rsa 密匙....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRiatCyXOlvic5iaMicokR2OmxXNfAdp3QCV6atErIMePOicKlzKZkV3n0xFA/640?wx_fmt=png)

通过 google 搜索 rsa q p e 解密代码... 利用即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRibutJxhJcib1DCXevgKPSlfPH5mTWopRUHu5gqVFuticicdo4wKadrYRrw/640?wx_fmt=png)

回来查看该 crypto.sage 脚本意思是可使用 debug.txt 中的调试值读取文件，该脚本可以输入 p、q 和 e 的值来读取该 root.txt 信息....

将密匙填入即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO7DtCvOXs54QvhBhaOeydRO3e8gkHotbwlhAHrr6kWVgOYX3TnAnzSh9H61ibwAoF9T7XRNQD3Ruw/640?wx_fmt=png)

因为取了 of 的最后两个值 pt 并对其进行解码，使得出十进制值即为 root.txt 的内容，需要将十进制转换为十六进制... 成功获得了 root.txt 信息...

  

这是一台密码学类型的靶机... 也有几台类似次类型的靶机... 比较耗时间不知道解密的页面或者类型是过不去的...

建议多多利用 google 搜索，不懂的就复制黏贴问 google!!!

由于我们已经成功得到 orestis 权限查看 user 和 root.txt，因此完成这台密码类型的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

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