> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Y24wHy3tgCXoJDxCNFS9cg)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **115** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/AhUATAqa6tibYa4zTrlvc4l1rFIH7HV8c7ibcicw1jgibbwVW2zia9JeVCleEKLjkT0RO7sJS34DVSzMJ9sGsIAn5Fg/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/85

靶机难度：中级（4.5/10）

靶机发布日期：2017 年 10 月 18 日

靶机描述：

SolidState is a medium difficulty machine that requires chaining of multiple attack vectors in order to get a privileged shell. As a note, in some cases the exploit may fail to trigger more than once and a machine reset is required.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/oQI1m5hhwD5Gicl7xUf6kh3ISTH6iacM05s8G12QVAykGzh7S5Po8EgeS5XJvZbiacbS8AuRQJ1VaRic18jlToOhVQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0fb0Y1M6icJJia7t9xsBuUuxZQgOLeWHYicicRpfEiahMz3mlpK0icx8qLpfMLDojhD7IwSE2IalXVBBFs9E1Z88Ka3Q/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/PRVgXdHra5CzBfuOaOX4dpiaoOia6WZfdos1RiaJEZJG7nrnxTkXBoianpRmkQTmqkmW3zkbaQqjAu6WwBYAmyGibiaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVce0hb6Oc1TXWL0aEDHKgEXNlzQKEBxrAzkL2ktibkQTR4W6l28fDfS8zA/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.51....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceiaeWAn5ibVVn8ZiaPKY2hjGPEtn9KVQAOUdtuVJhpA1PyzMq2icAm8IzXw/640?wx_fmt=png)

Nmap 扫描发现开放了 OpenSSH，Apache，一个 SMTP 服务器以及 Apache James POP 和管理服务器...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceiaibbFIH5MBkgmubKLfEia521Wy9Oz804h4pxCicV0IFM9ePKPMhjS9fTg/640?wx_fmt=png)

访问 apache 未发现有用信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceqBdwsNCH6peIgxMnV9A9y9xzibqcTE8UJXJ5kZp8tNVHPBo7GNdJHiag/640?wx_fmt=png)

直接对邮件服务器 james 查找漏洞，发现存在 EXP 利用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVce58uMiaF0xbjjgyMu1UC2JLb9vLgvYyibgeZcIolNyFqBc3ltEw0XQdAA/640?wx_fmt=png)

检查 EXP 默认密码是 root... 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVcek74IQkibusfe3cBice6FtFBsjHC5BqzWhpTHSZwljrG9d1Kvt5veOYZw/640?wx_fmt=png)

成功登录... 帮助查看可以修改用户密码... 修改

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVce66NnbZHpQw5xx07AYLYpWMtvRTvFiaAYt9bMubcCOay66iagDDPpoLBQ/640?wx_fmt=png)

修改后登录 POP3，查看到了两封邮件，一封是很高兴进来胜任工程师....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVcesnSKoPsCqd9IVDUCmcNSyAeWWpACibtArPzzFq3foxrKKW4ibULusc3Q/640?wx_fmt=png)

另外一封给了 ssh 的用户名密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceQk4Dl1yIgNOtHVWbW4spq5bXIqOtqlckPwXjrpJw9Qlda1YFcrVSRw/640?wx_fmt=png)

通过 ssh 成功登录，获得了 user_flag 信息...

但是这里 - rbash，会限制非常多命令... 需要绕过 rbash，不然没法枚举...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceZL2Liaa7dQZZePD8O11icicrVbXwoT9u0ibJIh2fhRicJ6zQOMYuL3hF8BA/640?wx_fmt=png)

```
https://speakerdeck.com/knaps/escape-from-shellcatraz-breaking-out-of-restricted-unix-shells?slide=9
```

google 找到了绕过的方法...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceRS4VbvzDv5ITtNIH3sQGYpXVy51KS9ibzvkMS4OBuia7Bbl8lfgloyzA/640?wx_fmt=png)

成功登录... 查看正常...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceTcFIyLkmicKxmOBslZEwmuDdEZfViclmVKKlxZ5xSFfGUCh2Re75I7ibg/640?wx_fmt=png)

上传 LinEnum 枚举...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceMJVTeDgZ4J5RNaX20GbNd0DlmePE4skJBsTovnsNSKFy0G1ooPF63Q/640?wx_fmt=png)

枚举发现 tmp.py 文件存在 root 权限执行 / bin/bash... 去看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceFKiaRImRDcb5y7suTa9LzAwaJoo21EesoNEjQqJpibWJibU0lILicm3iaAA/640?wx_fmt=png)

发现还是有限制...SSH 登录有限制，打算利用 EXP 获得 shell 外壳....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVcef1icq7bHe0dGAym7RzHia1qsLEOibsibFzPuRic4aYTvpv0jX14Ic4ycUHA/640?wx_fmt=png)

写入 35513.py 的 EXP，简单一句话 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceZZbyvxdQ152BJ3IhHShCnEYaRO8LLlQSibCJbDT58vgzd1q7FkmSbEg/640?wx_fmt=png)

运行 EXP，成功植入 shellcode...

然后本地开启监听，访问 SSH 即可获得反向外壳....

通过 EXP 的提供的 shell 外壳，成功查看到了内容....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceloUzQ4eV5r47sUl3Z9B8WjmicqV4hQnAibFaOx1Gv1RRnfz4t36TmU9w/640?wx_fmt=png)

这里修改的时候，还是存在通道字符异常现象，需要在优化 TTY，stty raw -echo 即可...

修改 nc 提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMz2hA9OsSlHuVy7T4QicVceQj874mic3yX0Ufava311my8ibKoywbjH6aViacbibcLlTibrQN6onQ1Sp6w/640?wx_fmt=png)

等待 2 分钟后，成功获得 root 权限... 获得 root_flag 信息....

![](https://mmbiz.qpic.cn/mmbiz_png/9qXnTkZPuxe8H1QicBcbrQQVKOeKw2PsaPtbkhed7icVWmmGk0o3VgYFqKdtNwPFicT2aW803Yp7DqjdiaoFRYVX3A/640?wx_fmt=png)

james 漏洞 + POP3 信息收集 + SSH 登录绕过 rbash + 补全 TTY + 内核提权....

可能在绕过方面耽误点时间... 应该算简单靶机...

这里写得很简单，我不会和前 NO.100 写得很详细，很多内容都一笔带过，希望看我笔记文章的能从前面开始...

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台中级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/gUVKXuw5icTuicMe1TSd3CYPJzxFcxUnzpBLmOY2lYosbSmH5Ro01bJbqOVUwZ97d098kTPyiaWWicblornticcLu9w/640?wx_fmt=png)

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