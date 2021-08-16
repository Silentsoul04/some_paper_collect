> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Yv3LX3S4CeEr-JDjP2hwrA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **54** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/2ichQqW6XvPlgohk6kjVu8GYOQ2Oco557j1bibkVCOsbLrO28pO7Lws1oVXcvS90GtYFe9Va2cepbqXjuziaDrnibg/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/hacknos-os-hacknos-3,410/

靶机难度：容易 + 中级（CTF）

靶机发布日期：2019 年 12 月 14 日

靶机描述：

Difficulty: Intermediate

Flag: 2 Flag first user And the second root

Learning: Web Application | Enumeration | Privilege Escalation

Web-site: www.hacknos.com

Contact-us : @rahul_gehlaut

This may work better with VirtualBox rather than VMware

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/70aCp38I3nX6dfnC3RPrQfDeuwyvRCkVZ5NrvqgrPsUd76ALjnYzdoWubzsdbaGpIBU9LdWWaN6eK2jaDkibicFA/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uibBj3IVvoT25eIZczwxWj006uwEumqVY4UJKhmicpA3FQoIwFVvE7peg/640?wx_fmt=png)我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2ucS4ibLe0yaoDlKicUlJ2rC2n4JibuSNby459gFVC62kIqI5wmZhJOicCNQ/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.145

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2ut9kDfxkMKYG4oGLH9SbzXia7yPvziassWE5Arv9kMeSXIAVyjTtfP0sw/640?wx_fmt=png)

nmap 发现了 22 和 80 端口是开放的... 可以看到 nmap 里发现了 80 端口存在 websec...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uaQibm97RFgz9q5iagf0iaLtYSUYl3nwGyPbld7z1CGHvRDEvabUboNuibw/640?wx_fmt=png)

访问 websec...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uCZC5ic4sqZbTZ63LbQYekg6sgSOlf0BvaxQCIxxJrzmxY5bh9EHUNeA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uYnlI3jZyYgia8m8stVW9zgu9JAGV9KQEfF8O7CUosmwnX0of2mkFjkQ/640?wx_fmt=png)

这里没发现别的插件可以利用的... 只存在用邮箱和联系方式...

```
contact@hacknos.com
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uF6a7hHzJ5E559YZXibDZwvmJjvNZicNUTuzLZYYkQJYBLEzJZgMyOKNw/640?wx_fmt=png)

发现了听过的，在 admin 发现了登陆页面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2u2Hc1XoSp7PLtRDaMjLciabPpNS68NAG4Ym1ZDLib0OeeDCEjlIC6dN4g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uG7HTk7LIShAvDF3VxRfTRnWf4cOdX3o7EktIx1E20kNcia5xd3sSdxg/640?wx_fmt=png)

```
cewl http://192.168.56.145/websec -d 2 -w dayu.txt
```

利用 cewl 单词列表生成器创建一个简短的单词列表... 然后进行爆破...

这里利用 burpsuite 和 hardy 爆破出现小问题... 这边有 waf 做了限制，一分钟只能试一次...

我手动测试后发现密码是：

```
Securityx
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2u3rBp9MZfeVl2usMa7d9RosqW18q8flGmKq29xxMMxxVzXr1uic8PZ0w/640?wx_fmt=png)

成功登陆...

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uEvcVgBEUoBDFsjYPET85Rk8rF8TTy1vXU4Jc2SDPwbU7yEl0K5RX9A/640?wx_fmt=png)

找到了可写入的反向 shell 地方...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uZxff8QKibLmvABkl8DIPkXsCUL8oEtFo3VibWJuOQq8GTsZ7BdFCcibKA/640?wx_fmt=png)

将反向 shell 复制进去，然后 save 保存，在点下 save 即可获得低权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uMorWh5e5Bnh6x0OkmctQ3H8qk95IzmBCLwNUNszVZhe6x6fr53pSLg/640?wx_fmt=png)获得第一个 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uf2ibIBiavQf8BqqsfTFdmazHsqZoHMXicZlgfBVDibnYxJhJmQNcQkyhzw/640?wx_fmt=png)

分析下... 使用脚本：

```
[LinEnum.sh](https://github.com/rebootuser/LinEnum)或者[linuxprivchecker.py](https://github.com/sleventyeleven/linuxprivchecker)
```

试了几种方法都没成功... 跳过...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2u5m8GC5Mo4KGvMg4rDibRLdmNN6RqzHB49olYY5XQlbyiag9E95gpuX8w/640?wx_fmt=png)

在 local 目录下发现数据库文件...fackespreadsheet 这是说上面是电子表格编码...

```
[谷歌搜索...](http://www.spammimic.com/spreadsheet.php)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2u2GAeTzf16ghOFfHib7JMv1gSibqiaWLNXbTmQ2yfpGvZLd8niaHDJ1Ao6A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uw7kTvRIkVLAHoUImcnwibmapMInxSd1SibyaND9phV5Zpfp3ILRoFogw/640?wx_fmt=png)

进来后选择 fackespreadsheet 解码即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2ufHGPYcay1wJdHzjicCl8aibEdd3ib7N6kiaDzO77JWqWVW5zEutM4vQvpA/640?wx_fmt=png)

```
Security@x@
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2uXic2trr0XIk1vLNGQdiagYFVMAeAm0egLPVicLqIjWIgic8hYx6qMmwoqg/640?wx_fmt=png)

成功利用电子解码获得的密码登陆到 blackdevil 用户...

发现很多命令都可以提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqTxmpcPCD7h0BlPpO2W2udo4aXvqmibuRF8WhzVyHGsOF3IgVV22j7otd1wrKQNMZwTkicGichLZTQ/640?wx_fmt=png)

直接最简单的提权，获得了 root 权限并查看了第二个 flag...

肯定还有很多提权的方法... 听说还可以使用 cpulimit 提权... 还可以用

```
docker run -v /:/hostOS -i -t rootplease
```

提权... 或者

```
[参考](https://www.cnblogs.com/cocowool/p/make_your_own_base_docker_image.html)
```

![](https://mmbiz.qpic.cn/mmbiz_png/2ichQqW6XvPlgohk6kjVu8GYOQ2Oco557j1bibkVCOsbLrO28pO7Lws1oVXcvS90GtYFe9Va2cepbqXjuziaDrnibg/640?wx_fmt=png)

除了这些... 是不是在 web 渗透还有别的没发现的信息呢？？？

由于我们已经成功得到 root 权限查看 flag，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/70aCp38I3nX6dfnC3RPrQfDeuwyvRCkVZ5NrvqgrPsUd76ALjnYzdoWubzsdbaGpIBU9LdWWaN6eK2jaDkibicFA/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)