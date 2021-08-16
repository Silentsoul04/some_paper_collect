> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7mzsql7sD08sZJkAb_GK5A)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **77** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_gif/Wic8BDGMRUojcP7P5MXxWPyW6sEXJUX24bQWIAqJHib0vdDp2dbEfveEaicTwLTba2CqibcMSGMpb6p6aopkkxhFjw/640?wx_fmt=gif)

靶机地址：https://www.hackthebox.eu/home/machines/profile/151

靶机难度：中级（4.5/10）

靶机发布日期：2019 年 1 月 14 日

靶机描述：

SecNotes is a medium difficulty machine, which highlights the risks associated with weak password change mechanisms, lack of CSRF protection and insufficient validation of user input. It also teaches about Windows Subsystem for Linux enumeration.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_gif/b6beYwicB58hl0HVHYtRMBLZVmQhMLkWUBrDeZwjA5hIS9DgspzICqhF1IONibMQ3Il0nuicMZibGbRqCqzkQ8IYvw/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/KjuGCOqDiaaNFtib6VJTMcniaaxmveDTOv0gxK5cXQIzqEMz4nucT2GzXGfSJfX6EoUEkmBoLvFFiaM0hRKjyFbpjQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_gif/SmicJrKFtALIWrz54cxp3O2ebjtPdQTG273RZmfK4JBv5drfB8QicSuSEDJBdrNh3FXWiceqaEhdqB2vjRED5JnBw/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvO6uLgZHk3YsNg7yu4U2o8Sj1Cq0DPtOIKaRz79S7P44vKTJmcnOCPA/640?wx_fmt=png)可以看到靶机的 IP 是 10.10.10.97....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvMVTP24C56O8DXBM7DficDDT8bUkgSRKIXapYTBDLS8mwqlbsicCPCCVQ/640?wx_fmt=png)

Nmap 看到 80 和 8808 端口都安装了 IIS 10.0... 端口 445 也打开着，Windows 文件共享服务（SMB）也可访问....

在 80 端口上有个 login.php，8080 端口上 microsoft/windows...

这里思路是，先找到 smbcilent 的登录密码... 才能共享对方靶机查看信息...

开始找密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvBdPkbxKJqQQTL9K1nzIxkNaK6202hS8H6pqlLRZuSk9J2GnLlzQ5PA/640?wx_fmt=png)

sign uo now... 注册一个新用户...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRv8F19h5awy1NZFibvztJrKd4wO8vTCE23URyy0icSg6HshLJ6oKVJdb9Q/640?wx_fmt=png)

可以看到进来后... 在提示我们可以注入... 开启 sql...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvm8KJQqmWCDXSyOtAYHbfib6nQCBltOa0Ypia1mDOKhYZApyQPxx45DAA/640?wx_fmt=png)

按照提示，进行了账号注释... 利用 OR 1 OR 注入发现了新站点信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRv5WPtfaicgL0k9u5ZA0WOCicR3gXU7IpiaYx3ibj41LRUyGQt99I0PbFBuw/640?wx_fmt=png)

在站点内，发现了用户和密码.. 以及域名...

```
\\secnotes.htb\new-site tyler / 92g!mA8BGjOirkL%OG*&
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvV09iapJ8ibNiaKowxbEJ0k13Sm9raxnZmmmMiavrbYlw8R7EYA2UW3aVVA/640?wx_fmt=png)

```
smbclient //10.10.10.97/new-site -U "tyler"
```

可以看到成功登陆... 并切存储了 IIS 上使用的文件...

那上传个恶意程序上去即可提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvraLfhL85pjsL4Y8mHrTCibmfwJVbwFQ0YugxPZCaHiaMjicgguKUU2Libg/640?wx_fmt=png)

利用一句话恶意代码，成功上传...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvxClfAWFL6j90s4OGw82EGBCh6rh6RgOMLaa9fsRXBtR1FyzWYYaUcQ/640?wx_fmt=png)

成功提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvO2u3oTqLwXuUP1BblrShTUYhGmiaL1E2fm92Lt7f9Fq5Fed8dMwT3oA/640?wx_fmt=png)

这里先拿到了 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvRqyOtYiaXMia8v9LSCTUxnjxyrINbPVgFxa4iblWWfbpSVhozWLSXxRWA/640?wx_fmt=png)

找了一会，发现在 C\distros 目录下存在乌班图界面系统？？？试试利用

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvDEP7ibuDonPJF0AZd5kwvUp2jmgA8DQfHDmrsdbzTmU5siby92bA76Ww/640?wx_fmt=png)

这里使用小技巧，利用 windows 的 bash.exe 利用乌班图二进制编码进入到乌班图系统内...

看到是 root 权限了...

这里是 linux 系统了... 可以开始提 TTY 了..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvPf2WBZpEgH207ckib6IxTpEolBtgUMB45ESE05IceEXl4HUPQ2RhjIw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvPf2WBZpEgH207ckib6IxTpEolBtgUMB45ESE05IceEXl4HUPQ2RhjIw/640?wx_fmt=png)

在根目录历史记录中，可以找到用户如何尝试连接到 SMB 共享的信息，可以看到了管理员的登陆密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvRWnEicuq5gI8THLYp29NoBdiaB635BiadYiaSzFRDBH4FgibWxriaRbtAI4A/640?wx_fmt=png)

可以看到成功获得 root 信息...

这里想获得管理员的最高权限可以利用 impacket 里的 psexec.py 进行 shell 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMJricqiaFoyhobkWNp8NQFRvWpbht8ckZ1jLTB7ia3kQFUZDmRZjyVBmerCQR9T1GLzgibMu4V3FiayZw/640?wx_fmt=png)

成功获得 shellcode 最高权限... 这里可以开始看任何内容做任何事情了... 不提倡！！！

![](https://mmbiz.qpic.cn/mmbiz_png/zYxEsibHhhqHFXvQKic55dUSltLhKZhWS26N6nZiaz7TZhriaodk3GvvC5cnnSRwZR5f8TztGuKSBM7d2JMSl5iafcw/640?wx_fmt=png)

这里很多玩法，开始的账号密码可以很多简单的 sql 注入就能获得站点... 然后提权方面想怎么提都行... 最后 smb 的登陆...

还是很流畅的结束了... 加油

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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