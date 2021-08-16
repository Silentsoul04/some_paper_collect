> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QFShdjSkg6d6NZJ66-dLCA)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **103** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/21

靶机难度：初级（4.2/10）

靶机发布日期：2017 年 10 月 13 日

靶机描述：

Haircut is a fairly simple machine, however it does touch on several useful attack vectors. Most notably, this machine demonstrates the risk of user-specified CURL arguments, which still impacts many active services today.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/XrBsia6eKtTFtr4vwm8FVt5frF8ojc6Xtp0ChSOwic1tRYkxthCoB1v1SekZZzcuvLGhDnRCDt8IVxpHV9flfc9A/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PXWEicxy2xC9I5LF9rFMqYCphbH3cCW3vuSv6ic9WhOhzpoJibNkQLKy2DAWiazzOcIg1RLfgZiauLaLG8ucgHOJGdw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjKicXVVIe8mTSZNTwPyfySNfrxXYefrTqWX6uoanjtoBqmI0rE47h45w/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.24....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjoN6QJvNTiaicyPNy29DDlicDJqnXqIpTPLZ704aSHcrPqRWFvDgPtY3dw/640?wx_fmt=png)

nmap 发现开放了 ssh 和 http 服务...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjCN4CXQqjpZF8ptzQc4hCWGgbbBuF4uEgdcYTgEMlcEond6vfcg0Ttw/640?wx_fmt=png)

页面无有用信息... 爆破

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjt9JSsZ9atoPmSIiapCDJeib9PIibq3hBmen0mt4JDneB1ibPGffARU1utQ/640?wx_fmt=png)

发现存在 exposed.php 页面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjjryju4vAmgWJYmVvBvic0g22R1fPBadxicTSeONs9FXEA6icQCn5NRTDQ/640?wx_fmt=png)

访问页面发现 curl 正在服务器中运行着，利用 burpsuit 分析...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pj7SaYqqVSTFOiatcYmSESxjFuC9wRCPedhu13pFUicuFEGbj8YiatgWtgQ/640?wx_fmt=png)

这是个简单的页面... 直接上传 shell 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjQJRiaRjU0w5yt5sZw9CHTDvghrO1tDnlMNwgMWeQhqWh2fGhEplyxHw/640?wx_fmt=png)

简单一句话 php 获得了 shell 外壳...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjh9l6RsaaRshKt5dVzibv58AluK0C3lbqQOrpMePX2A9U8lsjTXdWoeg/640?wx_fmt=png)

成功获得了 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjib20v2SibgB0CHAN8Xab1gk9FfxdDZYyX9GJriamibZ9bM4AerSmBoQTAA/640?wx_fmt=png)

通过查看 SUID，发现存在 screen 4.5.0，这是挺老的版本容易受到 bashscript 攻击...

查看 EXP 可利用 41154.sh...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjPHS30xDTYLkKx1BOWuJaEiau9YvIgLfIfTTk8Pwzx4XibYGTkxppnEAg/640?wx_fmt=png)查看发现 EXP 存在三个部分，这漏洞在最前面的章节，Vulnhub 就展示过了... 直接按照 exp 提示的操作即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN77GrWeEQYGpRibYtELs1pjWLHx8KGt8CUa3XnIt16Ubt0fibILodaYf9CccrlzoTkn7Xa4FXqPWibw/640?wx_fmt=png)

  

通过 gcc 反编译，然后上传执行提权，成功获得 root 信息... 获得了 root.txt 的 flag。

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台初级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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