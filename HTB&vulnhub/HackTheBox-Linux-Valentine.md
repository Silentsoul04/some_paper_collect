> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fChikBMv-Gfvghr4DnPx5A)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **124** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/TaLibElTFqBjNpYLEVorsaLMeHScCZR2CQcXF4QQuCmtUwOYTolRMZkXEOKJKKHnrJNjWo2g0h75l4aweJQQwAQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4wW5pibnKd0rib9mFlhgF2CWl6ibhJn9yNL1gIWU97JjNekJGTgEZ9wjNgOhiaqibaxPYfJnUKVHnMM9JHthC6kj1Ew/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/127

靶机难度：中级（4.7/10）

靶机发布日期：2018 年 7 月 28 日

靶机描述：

Valentine is a very unique medium difficulty machine which focuses on the Heartbleed vulnerability, which had devastating impact on systems across the globe.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/Clq0o4fE5u6X5A1maTmqcvtEibdrsDO41kZPibRCHsX3Koj69GFK2qOyPwdcrgcDkHklrdJzBCiaQPuMVe11oSYHA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CvwBwuJ5iaIuKRdhotaw2e3n2t3XzbjKYkLh6zErKz9DaftB4csSXxGQ/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.79....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CS0VdmIr0FiaUKt6ws6OeEULXBO8weuSuUvdHF3U4HEezKhtaADrDtDg/640?wx_fmt=png)

Nmap 发现了靶机同时运行着 HTTP 和 HTTPS 的 OpenSSH 和 Apache.... 还有 ssh...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CQicDUicvp0GLfPxwHMy6ickDDU2292ViaMgMR5AiaHvCqq7C7s2nsANibvGw/640?wx_fmt=png)

该图提示了 heartbleed vulnerability.....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CmEty7PpugYJ6ia2r54BZuaX69IvPMyxOVRxpCiaVvmCL8ibiczUSudR4XA/640?wx_fmt=png)

再次使用 kali 自带的 heartbleed 枚举脚本运行 nmap，确认服务器确实容易受到 heartbleed 的攻击....cve-2014-0160

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3COibXYahatGnjrdDco3g1zB10dGKSibAkImcfyibmKcVaiakg47CIBl7xAw/640?wx_fmt=png)

```
[cve-2014-0160](https://github.com/sensepost/heartbleed-poc)
```

直接利用 POC...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3Cx8enzHJowDB23MxPc3D15EIOTT8ABTiaUjsxAIMnBTMb0qRI6ictiaUpA/640?wx_fmt=png)

获取到了 base64 值密码信息... 按照 ssh，应该 ssh+rsa + 密码登录.... 还缺乏 rsa....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CwQAYcWQNC5RVksHET3eiazqwnlmhJRrd3ZOXjUAibCqtFfyia14TYdiaIg/640?wx_fmt=png)

利用 Dirbuster 爆破发现了 dev 和 encode 两个有用的目录信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CqNMxUUKiaf280OmVT3kqLjolicNPib2TSicDnEXAXJhlrjr950QCPRo8ibw/640?wx_fmt=png)

在 dev 目录下发现了两个文件....key 应该就是我想要找的 rsa 吧....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CkvuTe3oq7oA62zfkSgVwMxdbtyHcib0BSTyxx16JZ4RdoVfhe4KAVHA/640?wx_fmt=png)

该文件是一堆十六进制字节....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CjLRuPaiahD2iaNNkR2yRp9l3pjb8KtEuxyNOiaz8HgdS7EWLSnQG7JR3A/640?wx_fmt=png)

这里也阐述了 decode 是编译器....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3Cx3FCibugJ1Mo7NPI7Y34WHzTEjNOMB0TFcwaNJC7MByA8LgPHVOIDXg/640?wx_fmt=png)

```
cat hype_key | xxd -r -p
```

直接利用 xxd 做一个十六进制转储.... 获得了 rsa 登录密匙凭证....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3C7B4DVlJqyTmCfXVVfibsicQWXPO8TU4v2FrUlI6F091MrRKr0Y8KXczA/640?wx_fmt=png)

通过 ssh 成功登录获得了 user_flag 信息....

方法 1：

![](https://mmbiz.qpic.cn/mmbiz_png/xm4F2hYetPjEQESs45UQXRGBy3wswtHDWMZz77ibhszjBEbNjYqjTeF5Oiabq6YwXD7bWyT7xPAPcTPnasSXbSkA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ZkKbicS7g7ZHYZnHJIaEIGnCFUcfEpoZzbNbicBMkmZsoicIR4wRS4gabRwDEkG2qXlDxM2mJPI62cpq1pM3Alm5w/640?wx_fmt=png)

直接运行 ps aux 发现正在以 root 用户身份运行着 tmux 会话...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CHHam9zPhrMribRNI8JQHicicRHdqq5J0QAzcznLkicYkrxOX3aBHj1fq4Q/640?wx_fmt=png)

直接利用该 tmux 跳转到该会话即可获得 root 权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CjI9ygoMsQb1OXsaeKg9q4iaQvOSDG829jicv6BRa4iaANN6MUkL8DQdibw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3ClibfmIXibcMVAwsk7FFlqYOLqiaP3g1JmeUc7FuoVSb0JMFG5N9aCujBg/640?wx_fmt=png)

通过 tmux 直接跳转到了 root 会话中获得了 root 权限，并获得了 root_flag 信息....

方法 2：

![](https://mmbiz.qpic.cn/mmbiz_png/xm4F2hYetPjEQESs45UQXRGBy3wswtHDWMZz77ibhszjBEbNjYqjTeF5Oiabq6YwXD7bWyT7xPAPcTPnasSXbSkA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ZkKbicS7g7ZHYZnHJIaEIGnCFUcfEpoZzbNbicBMkmZsoicIR4wRS4gabRwDEkG2qXlDxM2mJPI62cpq1pM3Alm5w/640?wx_fmt=png)

利用内核漏洞提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3C6hKnmUnUxl7dLHNfWfr4reCMUIu079ReRUq4FhKTUiacicuO9ZXxYh8g/640?wx_fmt=png)

查看版本版本，发现容易受到脏牛攻击... 这方法试过很多次了... 开始

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3CnQQ4J1KiaHOxN4oDacicLkiawWkoX4NQxaOmiaXEn7oYkPRibHlFuYMliacA/640?wx_fmt=png)

这是容易受到脏牛的内核版本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNC4IKV7vjEkfcicNbLOLP3C2Xl4wM7x7YvLF805cd7Uc8d01bQN56dKrsbZ6MSoZGfdxO1lmZMWVA/640?wx_fmt=png)

直接利用脏牛提权即可.... 成功获得 root_flag.....

![](https://mmbiz.qpic.cn/mmbiz_png/TaLibElTFqBjNpYLEVorsaLMeHScCZR2CQcXF4QQuCmtUwOYTolRMZkXEOKJKKHnrJNjWo2g0h75l4aweJQQwAQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4wW5pibnKd0rib9mFlhgF2CWl6ibhJn9yNL1gIWU97JjNekJGTgEZ9wjNgOhiaqibaxPYfJnUKVHnMM9JHthC6kj1Ew/640?wx_fmt=png)

这里我不多说了，希望大家能跟着我的节奏学习，如果突然看到这篇文章的，可以自行 google 搜索脏牛提权的方式方法，很简单....

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台中级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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