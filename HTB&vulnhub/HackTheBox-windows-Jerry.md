> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MZTnSODRB6PumIzC4cWcvw)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **63** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/2ichQqW6XvPlgohk6kjVu8GYOQ2Oco557j1bibkVCOsbLrO28pO7Lws1oVXcvS90GtYFe9Va2cepbqXjuziaDrnibg/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/144

靶机难度：初级（4.0/10）

靶机发布日期：2018 年 11 月 11 日

靶机描述：

Although Jerry is one of the easier machines on Hack The Box, it is realistic as Apache Tomcat is often found exposed and configured with common or weak credentials.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/70aCp38I3nX6dfnC3RPrQfDeuwyvRCkVZ5NrvqgrPsUd76ALjnYzdoWubzsdbaGpIBU9LdWWaN6eK2jaDkibicFA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaicHUKVLMp2vK1qtPdpGSbdadeHWDtXOTAP2972AjPw559ADXrTJyapsWiabLgdMOibHrJkY8K58CibQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBF5evRSscZjYjvGHqGGc2FTXTPKQiafro6l8zrUKjJI682rC2n7Zk4ib4w/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.95.....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFSuC0IXNkNQaHicxtsckhkU3fZ6PibAx8SYenz4F2SQbyu15T0Co46jPg/640?wx_fmt=png)

nmap 扫描发现 8080 端口开放着，并且正在运行 Apache Tomcat / Coyote JSP engine 1.1 的 http 服务....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFGghcYdLTZouhMvkRJ65iaUfDcqANha5q8tXC5RoZxJhRWm7icCHQmAEg/640?wx_fmt=png)

可以看到，这就是一个正常的 Apache Tomcat 服务器页面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFibe3ZDnibu8W883qq88icVKLs22qtt1YE90TJOibC7jj3veib0L4IfIUtug/640?wx_fmt=png)

尝试挖掘信息，发现 mannager app...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFOhVncH1ib4iagibbN55ZH6sle08HBibCicxibvep3FhDAQicYO1hoCuiaZLg3g/640?wx_fmt=png)

尝试使用默认密码不对，cancel 发现重定向到了另外页面...

在页面发现用户名密码...tomcat/s3cret...（这里由于保存了 cookie 导致无法返回重定向页面没法截图...）

然后利用账号密码登录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFo2yrfC6sgMSJE4SPQD4p5VSLmf6pqX9QEd3Fk1lS5864X4FGKnYTZQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFTVrDaojne84kAOlYF16edEh0b8ibUmKDib5KiapdPibASv8lXPpI4hSGwQ/640?wx_fmt=png)

往下翻看到，典型的文件上传漏洞.... 好吧，做过太多次了，根据 WAR file to deploy 直接上传 WAR 文件即可...

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaicHUKVLMp2vK1qtPdpGSbdadeHWDtXOTAP2972AjPw559ADXrTJyapsWiabLgdMOibHrJkY8K58CibQ/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaicHUKVLMp2vK1qtPdpGSbdiaMSAhshIKEfHM67EItzicnqSabTlmLh8MLGU5PVy3Lyc3cIPJE0Kjnw/640?wx_fmt=png)

方法 1：

直接暴力利用 MSF 提权...GO

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFoMzjG1dz1JhdYeVS6eLTPOJHSR9LDHHxib76w4GqsG31X3R9unPJ39g/640?wx_fmt=png)

生成 WAR 反向 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBF0rvdDm5OTBQaf8RHNUeZauksYCeDA0xmSbdjibWSfYOdcdwffiayMBrA/640?wx_fmt=png)

获得反向外壳... 可以看到直接获得了 system 最高权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFUkx20Ma1LFghKbS7Dd1vZAnyelnEfib92axuaLicSz11nvojibGVT21vw/640?wx_fmt=png)

成功获得 flag... 这文件里面包含了 user 和 root.txt 文件...

这里还可以利用 jave 得 MSF 生成 shell，直接提权即可，不需要查看 jsp... 方法一样！

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaicHUKVLMp2vK1qtPdpGSbdiaMSAhshIKEfHM67EItzicnqSabTlmLh8MLGU5PVy3Lyc3cIPJE0Kjnw/640?wx_fmt=png)

方法 2：

利用 CMD 脚本进行页面渗透...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFEibEichaRd29iciaQT4oDtC761JYKsAfh8Wic7cdT0ic1q60XlODuSgNFVkg/640?wx_fmt=png)

```
git clone https://github.com/SecurityRiskAdvisors/cmd.jsp
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBF8y2roibHdWXGL8ibJF15HPzCuMvpPtUFMWcKSza6s3QJBUIkQibRK8hnw/640?wx_fmt=png)

添加反向地址...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFuxkfANClMYSkUlR6ZLNxPcnrhdzpPbOgaQiaHtEic2RgaOial4lHLJ3Zw/640?wx_fmt=png)

将原有的 cmd.war 删除，生成新的 WAR，因地址更换了... 然后上传...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFPUHRzuvnYAEsicK18245WLIdmibrID34WAn81FC9CSj4YD8kTXvmhibnA/640?wx_fmt=png)

开启 80，成功链接本地得 shell 库...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFzUUDQQgcQonVQvu6sF8jMwQDDOK2uDLo8Epw9P0CZo4TjxZX802yVA/640?wx_fmt=png)

可以看到成功获得了 shell... 执行命令即可获得 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOftMFAmT5Cz24HZLqwcnBFKibhJy3diaUUQdbKoQiaUNHNoXUOuLQJFRrxQh5LeB1Zuz4I5o0OY22UA/640?wx_fmt=png)

```
cmd /c dir \Users\Administrator\Desktop\flags
```

![](https://mmbiz.qpic.cn/mmbiz_png/2ichQqW6XvPlgohk6kjVu8GYOQ2Oco557j1bibkVCOsbLrO28pO7Lws1oVXcvS90GtYFe9Va2cepbqXjuziaDrnibg/640?wx_fmt=png)

成功获得 flag... 执行 cmd /c type \Users\Administrator\Desktop\flags\"2 for the price of 1.txt" 即可获得 root....

另外还有各种方法能提权... 例如 https://github.com/byt3bl33d3r/SILENTTRINITY/tree/legacy，搭建一个服务，链接靶机，然后提权...

这台靶机中规中矩，大家在学习的同时，一定要做好笔记，也是告诫我自己，加油！

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/70aCp38I3nX6dfnC3RPrQfDeuwyvRCkVZ5NrvqgrPsUd76ALjnYzdoWubzsdbaGpIBU9LdWWaN6eK2jaDkibicFA/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

**2021.6.9~2021.6.16 号开启收徒模式，实战教学，有想法的私聊！**  

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)