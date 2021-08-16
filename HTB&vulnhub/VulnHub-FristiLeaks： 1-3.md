> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KfVRfnFlWwuoG4OisZlWFA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **21** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

  

  
![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/fristileaks-13,133/

靶机难度：中级（CTF）

靶机发布日期：2015 年 12 月 14 日

靶机描述：用于荷兰非正式黑客聚会的小型 VM，称为 Fristileaks。打算在几个小时内打破，而无需调试器，逆向工程等。

目标：得到 root 权限 & 找到 ** 四个 **flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tKCLz9jOJKBzxRGzHTFe2HkicVRpicMCG7oxs83gNSt7fHcxXjGFLK2lg/640?wx_fmt=png)

这里靶机已经显示了主机 IP，因为这是 vulnhub 内容要求的，VMware 用户将需要手动将 VM 的 MAC 地址编辑为：

```
08：00：27：A5：A6：76
```

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tIlR1OSSdSVO5RQE4JqMA0V62Hug5HqGhpC0HClZVlqCHZ10eqqBBGA/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.182.130

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tGZAIAl7ib9cJ34svHcoz10ibm9TZEcBfCQib9Qv4Ygbwq24ibQgXKvqpmg/640?wx_fmt=png)

只开了 80 端口

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t4icC569UPbtv9iatn84JfNCwzUPiaPphEoqRZwXHnDz2ZffcFvuIvkHKQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tUmKu8GHdTt5RbruU4BlwI69Wry0Q6mvOdXIxpvcfWL2F0ialqMU9XQQ/640?wx_fmt=png)

查看前段源代码没发现有用信息... 就是让我们 4 小时完成....

爆破下目录试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tftmHdlxK9HIfniaFg9BWeeYmFgLxREe8ZphxubGLz78tkAf6Qrmiaqlg/640?wx_fmt=png)

发现三个目录

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tbDs5o3D8yPyg4yc9aAb9QKf7y31O8u5mwBVvf5MhuqHBwicPB3x6A2A/640?wx_fmt=png)

分别访问进去都是同一内容

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t7K6hdytyibLaBy3ckWB6bdGF6fKV8Ll7gWW8aHRIEicGnKd31HjedocA/640?wx_fmt=png)

没啥有用的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tYTqonfS3sRh3fzmZ6ibahsDgkmJTAFCJEiaujzP6IOHPpyeun0GXeLBA/640?wx_fmt=png)

这里就是个存图片的目录，就 80 端口粉色图和人图... 没啥有用信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tqlyMBEFDcSIibdtpmyHfUesHtsibibtM2CPgm0a7wKvdl0eTU8icgZ9wuA/640?wx_fmt=png)

这里收集到使用的版本和一些信息...

这里耽误了点时间，回看主页面的信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t2wy9VoKUde90rZoM0YqyDyWKRsdaQdnTib27HkJHUZoibepXdH4iaQAdA/640?wx_fmt=png)

这里的 fristi 和主题 FristiLeaks 相对应，我尝试访问目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t1qsKy71dFxCnReRkckI5PGMx50YMB9B1wtIbLTDE4vFooourDaOBFQ/640?wx_fmt=png)

admin 尝试登录失败... 查看前端代码看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t6X7GZfCk2iclhibj2BXiaesZm3DS2Tyo6DXXhZEETEUUSja7DWPu02m7A/640?wx_fmt=png)

这里发现应该是用户名...eezeepz

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tPuqWSj1E5jJFvURLBYWJGSXQuTr013Ox5pSEsicTC5wthx0YmYCm1icA/640?wx_fmt=png)

发现 base64 编码的字符串... 解码解码

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8taW6bGDv6BRH5zaPvmria19lkibZS999LChquPian6qjef90DEZEIy0UOA/640?wx_fmt=png)

这是 png 格式，修改下

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tOOuMVzo2kEEFibpdeWNUM6dv8LuibLiaOh4ia6csuBqY2qQ26SFic66KImg/640?wx_fmt=png)

```
keKkeKKeKKeKkEkkEk....
用户名：eezeepz
密码：keKkeKKeKKeKkEkkEk
```

二、提权

登录

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tZqg4Zn8EWkmoYeheD8shEuBBK5tIImQ8GvmJHGVc19nU9w36Awm7bw/640?wx_fmt=png)

可以上传 shell，弄个反弹 shell 提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tjsWTjLVkVIGFoXAvf06YH4176iaFibJRH0vUyVibAoicYXyfJVWKDGCicHg/640?wx_fmt=png)

这边我随意上传一个文件测试，这边只能用这三种格式的木马才能提权...

这边教大家制作木马了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t3CsxPNKibms5M2oZGIgZwiaT1HIcSFm0RiatRRefQtLiaBoqe8iaa5Vfptw/640?wx_fmt=png)

这边先用 msfvenom 生成木马文件...

msfvenom 生成的 php 木马，但是服务器限制了三种类型上传，这边给 php 加个外壳，然后上传即可！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t22DfcYkRnh6RBia1YYDQf0Cibjh7TibSSs6CFMVdgZaTPPgMBicjAad4gg/640?wx_fmt=png)

上传成功后启动它（使用 web 进行访问即可）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tEHDFfWSBVZyKNUiaIZLPhPKRoib8hqtpFrIXDp4hOiaC6N47ib1Hharnfg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tuqgA3M2QMiasds39BbPfsAK1CSchJCGpsHGES1ZnMM6rluPv77RD7Zg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t6S52bzyrDtAMF3MfnrbZOKNkM5zVMRLfwrUQCxDdByuLRY7r2cpOTA/640?wx_fmt=png)

成功提得 apache 低权限....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tOxiaA4IOC0RWYpXWZOS9gvwpjNk6lmFibLVElbicz0tZgFtia9gXchK5kg/640?wx_fmt=png)

发现有三个用户能登录服务器... 一个一个看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tdTAdz7OCKn3qnPGfcTm86NWLgib0LLUaEAuLNz8wiahysHBlAib50NVDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tG2kQxBHPoGHRxf0G7YWITF9ZpzQjPSVPiaKATat7Dmve7xPM1pVZgqQ/640?wx_fmt=png)

admin 无权限...eezeepz 发现个 txt 文件...fristigod 目前无权限... 下一步查看下 txt 看看（反正在这个用户下）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t1DDeK8b9BOc3SJ9RnO6x6rDWpGRzEJmAY0AuKK8DYBGSZNC6HKHiaYg/640?wx_fmt=png)

意思是说如果正在运行脚本，则该脚本将在 / tmp 目录中以 admin 身份执行任何命令（如果位于名为 runthis 的文件中），所以只需要执行一个命令，使用 / tmp/runthis 文件技巧就可以访问 / admin / 文件（还有每个文件一分钟后才生效...)

尝试的发出 chmod，通过回 chmod 777 /home/admin 至 / tmp/runthis，等了大约一分钟后...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tg8OXwEhSl2od51mLlxNIQU1IFZGaGW5BoDG8V6FWLic2GiacQcymVP4A/640?wx_fmt=png)

```
echo "/home/admin/chmod 777 /home/admin" > /tmp/runthis
```

成功进入 admin 查看其中目录文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tTfHOtsVlfV8xeLicdLubxicotephqjpWhlxDYbNchQP5VkZJLdZN8JQA/640?wx_fmt=png)

```
cat cryptedpass.txt
mVGZ3O3omkJLmy2pcuTq
cat whoisyourgodnow.txt
=RFn0AKnlMHMPIzpyuTI0ITG
```

发现了两个 base64 值和一个解码当前 base64 的 python 脚本...

都是套路啊... 通过此脚本加密了两个 txt 文件，反向修改下就可以读取到真正需要的值...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t9Icic8vbjwHUssocgeh68VnnaLBCB2ay7VcsThibP7zF00bzhhWu4tUQ/640?wx_fmt=png)

这边我特地直观的截图更好的理解... 这是一种方法...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tE4SWR84xzjCkjYhArxPHTc2pRjAJVYsiahIKGfdWfS5sib7SumcaiaA4Q/640?wx_fmt=png)

或者这么改也行，意思类似

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tdzfl3iaB4icUMSdqbweZiaZt27BxFEoudic2n6qBxVTictlsQ6iaqMMLOEEw/640?wx_fmt=png)

输出结果是一样的 LetThereBeFristi!、thisisalsopw123

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tkvRYAMvGVeObib6O7jAYMcDico6hS4AM5HyG7fWyK7TiaMuuiao1AHlGSg/640?wx_fmt=png)

先登录了 admin，发现此用户没权限在本地执行 sudo... 换 fristigod 用户...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tPnHjXyaUdLKictRe8e7siaJpnR9hVaByMQmiaHicfWVj8yJRqe24iaszhLA/640?wx_fmt=png)

可以看到 doCom 是二进制文件... 权限在目录下 / var/fristigod/.secret_admin_stuff 执行... 进入试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tT2aickkuc2kMJozHIqzwBbkqJhtqOzRicQkRI2IJpdaoCaCBLwoIvEbQ/640?wx_fmt=png)

果然，这是拥有 root 权限的... 直接将用户 sudo 用做 fristi 来访问 / var/fristigod/.secret_admin_stuff/doCom

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tqS3aY7yfYl1dSuG0ALan0iccnahwROqLRH0qYXoTzH4MF3TSIYAKd3g/640?wx_fmt=png)

```
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom su -
```

成功拿到 root 权限和 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

这边本想在尝试用内核版本强行提权的，发现三个用户都限制了接手 shell 的权限... 先这样吧，这边有挺多新的知识点，写完印象更深了... 加油！！！

由于我们已经成功得到 root 权限 & 找到 flag.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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