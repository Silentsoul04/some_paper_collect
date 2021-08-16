> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/e6yr9xYc-ztEpgPxqaF8lQ)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **35** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/spydersec-challenge,128/

靶机难度：中级（CTF）

靶机发布日期：2015 年 9 月 4 日

靶机描述：

该虚拟机采用 OVA 格式，并且是通用的 32 位 CentOS Linux 构建，其中包含挑战所在的单个可用服务（HTTP）。随时启用桥接网络，以为 VM 自动分配一个 DHCP 地址。此 VM 已在 VMware Workstation 12 Player 中测试（如果需要，请选择 “重试”）和 VirtualBox 4.3。

目标：找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicj4D0lPGW0NToZd71NAvMcTyJouTDiasSsEAWiab9Qz8V9K5npq4qW0oLw/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjGu1lxzlUWQhzjcqcnZblHDzqN9pUT0GtBYvjibics69FuUSvjWO8jjIw/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：

```
192.168.56.131
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjQuvHgmUfIyTico8o7wzoJicvqpiaxHd9yfo10eknys2MnmUbx9uaUHiccA/640?wx_fmt=png)

nmap 扫到了 22 和 80 端口是开放的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjM3XSKlhv8Lo9AUeDoK2vVM6mqibLnx841uiapcrGJ1W15cHnScPvFLBw/640?wx_fmt=png)

查看图片内容...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjrNys00KIB18jUoQFWicOx0lpCK0nGPcE4JojNVG94ExGVDWgc44CH4Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjplNlG2327x3QADYmUq5wzDS59eCHScsvNbUOUkqFqp7Xv8NdibrgoicA/640?wx_fmt=png)

发现了十六进制的值... 转换看看...

```
[转换链接](https://www.rapidtables.com/convert/number/hex-to-ascii.html)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjS5uaLlTwZg03wTFticSYib6ibbzfDGicmDCBRL8RAOc2KnfQxLfMohjrFA/640?wx_fmt=png)

转换后：

```
51:53:46:57:64:58:35:71:64:45:67:6a:4e:7a:49:35:63:30:78:42:4f:32:67:30:4a:51:3d:3d
```

继续转换...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjticsWT2C6NdTN03w9wxKYjY3hY9MPHp5rlxUYPFFWsE0aMFiaUQRKicWg/640?wx_fmt=png)

```
QSFWdX5qdEgjNzI5c0xBO2g0JQ==
```

转换后是 base64 值...  

继续转...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjH6OfmbjESbicAKzVB3ibmXnBgBqRNjLWpfcfPScA98dyh12NM5vEib3bw/640?wx_fmt=png)

```
A!Vu~jtH#729sLA;h4%
```

这像一个密码，先留着，另外一张图没什么有用的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjVP3jWwxbONacAs4oW0PC77EGIckGV7HI62THaqcbwSVG9WOfIIh4Kw/640?wx_fmt=png)

在谷歌上搜索了 eval 函数，发现是 JavaScript 它可以在其后隐藏代码.. 使用 javascript 解压缩器对其进行解压缩...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjQD06WIDzeU8FMvYqic4P6TLwdib5avG9dj8iabXmwlGzHEibNFHYlm6IPw/640?wx_fmt=png)

这边使用 javascript unpacker 进行转化，得到：

```
61 : 6c: 65 : 72 : 74 : 28 : 27 : 6d: 75 : 6c: 64 : 65 : 72 : 2e: 66 : 62 : 69 : 27 : 29 : 3b
```

不用看，这还是和前面一样，继续转 ASICC 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicj4z8b7YKVoGAXR2VrmAjqJOg52U5SST296tPlzy3JUkE44iaia2e6jYGQ/640?wx_fmt=png)

转换后：

```
alert('mulder.fbi');
```

这边按照思路走下去，目前信息没什么用啊，感觉... 先不管留着吧  

查看下 web 还有别的信息吗...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjwrv83G9Pq1XWEsjoE6zibfsn8ZRfZhjppCt9vfFaRKlVDb0bf8sByhQ/640?wx_fmt=png)

```
Cookie: URI=%2Fv%2F81JHPbvyEQ8729161jd6aKQ0N4%2F
```

发现了 Cookie 值，URL，转码看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjexHw9Shu638VwXd1lwSr8CUM6NX0ib6feqKqUUliciaialpXM3GTZpFic5w/640?wx_fmt=png)

```
/v/81JHPbvyEQ8729161jd6aKQ0N4/
```

转码发现一个目录？？？？双斜杠 //...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjuYBhBOWTXwZibjjZz0j8Jq4vDZag0y2z4ugDuxqibKVxs89CRlicfRg9w/640?wx_fmt=png)

还真是一个目录，这是表示网页存在，但是没权限查看...

这里搞了好久，发现了前面 alert('mulder.fbi'); 找到的信息... 添加看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjuo7TGbIysdDTWV272ciaFibmrAOUkhwZh348xaZ7nXxLhLa5PSs3ibKJg/640?wx_fmt=png)

下载...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjE9iaCibaAR2nkbvZagZiaz1Kha07k1Cb8nU4fYia1ldC1UuVFCicTLagG3w/640?wx_fmt=png)

这是一个 MP4 视频... 查看完听完后没啥问题...

我从别的途径了解信息... 发现没路了，我觉得还是视频有问题...

查询谷歌，有篇文章告知了我答案...

```
[链接](http://oskarhane.com/hide-encrypted-files-inside-videos/)
```

隐藏视频中的加密文件... 用代码查看下...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjnJl1AUowmH0hXEQKVXZOgVQEiciaPRzG4zUt3sslFN2fa93SdfLiar6XA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjiaZCHzicHiclB9AUqSxys8hKzNWqNCLMRibWkQPr6T1DnibZuuACpStIovg/640?wx_fmt=png)

果然是一个 TrueCrypt 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjQt4UO3Yl3ATW4BKYceVYUx3UIicBwDF31ic7bteLnlWsj8odp8YdlV7A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjgCIMhaAUBvniavichibZ7oXWAuepHH7Urag3Wj58Vz7Cnmk4uickxopmLQ/640?wx_fmt=png)

这边我在虚拟机 windows10 系统上安装 TrueCrypt 程序..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicjuInXg6FyoZIWLZbGMOyufOytJaVIfAAns9CkuuJ1gF18CsuTVbVFjg/640?wx_fmt=png)

需要输入密码.... 尝试最初的这个看看：

```
A!Vu~jtH#729sLA;h4%
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicj4bqzOxicZKRa0biaByFL4b9iag2kBf6YDI2VM6NnpicNg5tUrPiaprLqSfQ/640?wx_fmt=png)

可以看到驱动器将已被解锁，打开...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgJ28tk8mvhrG2IqsmsYicju3KGU7FgytbBYm2wNibqw8jFGMc0ESelzewsoJbBPz465VLFbvTcXSg/640?wx_fmt=png)

可以看到已经找到了 flag.txt 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

这台靶机说难不难，其实很简单... 但是我还是花了好几个小时...

中间找了很多资料，对 web 进行了 dirb 和 nikto 爆破，以及 ssh 都没用到... 这边我硬是靠着收集信息来完成了...

十六进制转 ASICC，以及一环扣一环的信息，缺一不可... 最后 TrueCrypt 视频隐藏驱动等等... 新知识...

我觉得存在 ssh，就应该还有别的方法能看到 flag.txt，我可能还坚信 flag 别的地方也有... 我还没这个能力，如果有小伙伴知道别的方法，请告知我，感谢！！

由于我们已经成功找到了 flag，因此完成了这台靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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