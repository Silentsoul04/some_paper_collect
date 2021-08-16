> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IsPkLVd63ZfuCaRvRG-4Fw)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **57** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/RKmmCHT73fdQQ2nv9rDeddIlJk71QWHcslefZEPQxvuVzXNn9ZlY6dicKOiaJQBXNFYkbHtUsOw0duN5FIUuItSA/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/186

靶机难度：容易（4.5/10）

靶机发布日期：2019 年 5 月 20 日

靶机描述：

Bastion is an Easy level WIndows box which contains a VHD (Virtual Hard Disk) image from which credentials can be extracted. After logging in, the software MRemoteNG is found to be installed which stores passwords insecurely, and from which credentials can be extracted.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/xVzbJNmGHSCH5d0fX1bHZYbyKoFLsiapvaq5K6Oo80wFkVAmt04DEn4DSiagPY4oL5QTcTlFhJZsA5mbTUZTJFYQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzNor5PVoku2LicibGstAxegxRUmich8fDIqDZRlgysC9oTN7KzHd8JvpJg/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.134，windows 系统的靶机...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzG6qmciawX8iajwkcl3W4cUGkhxgv2Rs14mTsibEVickY7whCSdZlTZ176A/640?wx_fmt=png)

```
1.可以看到SSH和Windows服务135,139,445处于活动状态...
2.这里最大的攻击面就是SMB，ssh具有身份验证或者匿名访问，可以看到445中的microsoft-ds通过ssh匿名访问smb....
3.服务器还报告了明显的时钟偏移，这表明可能是时间源配置错误即可....
4.查看smb发现在smb-security-mode中保留了对SMB的guest访问....通过SMB在我们的堡垒机器上检查公共文件共享...
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz7uCgd9YvSQrTt7Ezx8YP64j5iabk3StZtHlQBxAnqc1M31ib5CDpqg9g/640?wx_fmt=png)

```
nmap -sT -p 1-65535 -oN fullscan_tcp 10.10.10.134
```

另外我对 TCP 所有端口都扫了一遍... 还发现 Powershell 远程访问的两个端口 5985 和 47001，对于这两者都需要凭据，因此在获得访问权限之前，需要进行一些其他枚举...

根据上面得到的信息，先对 445 进行查看下 smb...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz3OKeNndpNh3vxy1k3rQdB5ZXz98Xf1LZqNGbibjzFo7YZA9jCZHOvdQ/640?wx_fmt=png)

```
smbclient -L \\10.10.10.134 -N
```

看到唯一可以访问的共享是 Backups...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzcX7qAJ9CFDvMILLzbO1zQVK916Pj3BJXMLIEFMATZYSwCSa29BYELA/640?wx_fmt=png)

```
smbclient -N \\\\10.10.10.134\\Backups
cd "Backup 2019-02-22 124351\"
```

直接访问 Backups 用户，发现 note.txt 文件，使用 “get 命令将 note.txt 文件下载到本地...

sysadmin 会发出警告：系统管理员建议，由于 vpn 连接速度慢，我们不应该在本地传输整个备份文件。让我们浏览备份文件...

经过以上收集的信息可看出：

1. 发现计算机备份是在备份文件夹中进行的... 并且有两个 VHD 备份文件系统，大概一个用于引导分区，另一个用于 C：卷....

2. 需要将文件从 SMB 中取出并挂载，因为无法通过在文件夹头中键入存储系统的 IP 地址来访问这些设备.... 例如在 Windows 中，需要将此设备像磁盘一样安装在我们的计算机上....

3. 考虑到 sysadmin 发出的警告，则获取这些文件可能会花费很长时间，应该是下载这两个 VHD 文件需要很长时间，估计很大...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzZ88libWLSUBLibKQovn3nMXRicJIdlpiaZD62Bbd9u4kPyqsbTdpYLcHAg/640?wx_fmt=png)

```
mount -t cifs //10.10.10.134/Backups dayusmb
```

我利用 mount 挂载 backups 到本地 smb 目录... 这里我使用了 cifs，也可以用 smbfs....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz2HRdDvofSkadYOtrMUDnibibQBlibQqzck81v72pyuMUvPZHxnH1BoibgQ/640?wx_fmt=png)

```
du -hs *
```

果然文件很大，由于靶机在国外服务器，我访问都很慢....

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

挂载

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

  

  

方法 1：

  

  

前面通过 mount 挂在到 dayusmb 目录下后，可以读取到 VHD...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzdLXMQyaiclx7qDuQsEBG7ULHoVJSrVv23DnvETpB3Ds7R0bH2BSKANw/640?wx_fmt=png)

利用 vhdimount 将文件挂载到 / mnt /dayutest（这里文件是从前面 mount 挂在到 dayusmb 里面提出来的，不然就得自行下载 5G 文件）

从输出中可以看到，在 / mnt/dayutest / 中生成了一个图像文件 vhdi1，由于需要挂载该文件，以便可以访问文件系统，所以在映像文件上运行 fdisk -l 查看更多的信息...

fdisk 发现了文件系统的起点和扇区大小：128*512=65536.... 可以分割出分区并将其安装...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzZvOccsNLqd3fnuPyZNvIziavDS2USBEjlMReHpwsCcuaRjSHbpSSibUw/640?wx_fmt=png)

```
mount -o ro,noload,offset=65536 vhdi1 /mnt/dayujust/
```

利用 mount 安装此分区的偏移量的起点和扇区大小后，发现搞错了，安装了 c3 的 VHD....（这里搞错了，但是我还是写出来了，记录错误）

重新挂载下 C4 的 VHD...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzOLoAkGhynduqev7HPZSUHVHqPrjRFoZk3nTP00P7FmJ6raibicDM9Y8Q/640?wx_fmt=png)

重新对 C4 操作了一遍... 就不解释了...

这里可以看到 14.9G 的文件值... 起点和扇区大小还是和 C3 一样... 直接 mount 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzHBEMekoYHkicc4Qmj2ibVG8TMrbJcF6jQrcoibnSRhZdqJrwUryViaz1Ww/640?wx_fmt=png)

在 windows 中的 config 目录下可以看到 SAM 文件... 里面肯定含有哈希值....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz0icGZU20peV4dubMV16k3hpY4aeD4xgics1JLXWhCia8mQOP7h3q2pFibw/640?wx_fmt=png)

```
pwdump SYSTEM SAM
```

利用 pwdump 或者 samdump2 可以读取 windows 登陆系统 SAM 文件里的 hash 值...

发现三个用户，1000，直接利用 L4mpje 用户即可...

  

  

方法 2：

  

  

利用 guestmount 工具将文件挂载到 / mnt/dayutest4....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzjnzATMn82qoOIH4mSpNRpkHS9Lpic8egZ42fZQXelyic9yLz95ssmcDg/640?wx_fmt=png)

```
guestmount -a "9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd"  --ro /mnt/dayutest4 -i -v （需要知道字母什么意思的去-help）
```

已经成功挂载过去...

利用 pwdump 或者 samdump2 读取 windows 登陆系统 SAM 文件里的 hash 值... 和方法 1 一样...

[guestmount 链接](https://linux.die.net/man/1/guestmount) （学习）

  

  

方法 3：

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzicCeGhOhQia5sI7H1fThUoo97tj2QrXkJWbFhej7tgiaicxucCQQIQeynQ/640?wx_fmt=png)

```
modprobe nbd
```

 如果利用 qumu，必须得在本地加载 nbd 模块....  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz5fZzULXwJicogXdFw0mA3nLD5qNxXIESKoMlBwju2yTX88fBont87tg/640?wx_fmt=png)

```
qemu-nbd -r -c /dev/nbd0 "dayusmb/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd"
```

利用 qemu-nbd 工具，qemu-nbd 是 qemu 里面其中一个用户态工具，依赖于 nbd.ko

qemu-nbd 可以将 C4.VHD 挂载到本地...qemu-nbd -h 查看命令介绍，使用 - r 和 - c 即可....

然后 mount “挂在的目录”  “本地目录”

最后利用 pwdump 或者 samdump2 读取 SAM 文件哈希即可....

  

  

方法 4：

  

  

https://www.7-zip.org/download.html

利用 7-zip 软件将文件拷出来挂在....

方法非常多，只是介绍了几种... 方法 1 中还可以用 rsync，最好用的还是方法 1 和 2...

```
解码HTML哈希值：26112010952d963c8dc4217daec986d9
```

推荐地址：

https://crackstation.net/

https://hashes.org/search.php

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzbkQRBXJfIv0s0zC5WUu0Y1Hh4IrGXz9whl8T02ez0rWRXDfPYh1LZw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzvYAJAY2jOCXjLgekbz8A32W81yjHqQUPJjk2xnOXrSynj5GTctpAhA/640?wx_fmt=png)

```
26112010952d963c8dc4217daec986d9:bureaulampje
l4mpje：bureaulampje
```

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

SSH 登陆 l4mpje....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz6pbhibHdpHVLuXXlrV1UiaEicFweOjldpD54OnGHpTq6Q9FOGVanUcAIA/640?wx_fmt=png)

```
ssh l4mpje@10.10.10.134
```

成功登陆...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz8iaQDyCxZjomedB0hAia3sE5HdFes0k1Md8GcIxCPng90b5pJF48vYTw/640?wx_fmt=png)

成功查看到 **user.txt**...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz8b3H3Uicbak54SGFU1qUu8MaWdenkjxcuvNHzfIdhCiamicziazX4b3dJw/640?wx_fmt=png)

利用 mremoteng_decrypt.py 脚本可以查找管理员帐户的密码 

```
[链接](https://github.com/haseebT/mRemoteNG-Decrypt/blob/master/mremoteng_decrypt.py)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzXFD5cp4ngMYdwok8F2j7fs0opuO0NYCHWK7yOkzssVlbeG5maxER9g/640?wx_fmt=png)

打开 confCons.xml 来检索加密的密码发现存在：

```
aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
yhgmiu5bbuamU3qMUKc/uYDdmbMrJZ/JvR1kYe4Bhiu8bXybLxVnO0U9fKRylI7NcB9QuRsZVvla8esB
```

这是 XML 的 BASE64 值，需要解密...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nz3ib51wOJxjyboyliccsloEdceHBapWWwJurzzFgAM8vuUrMNQ1rhA46Q/640?wx_fmt=png)

```
scp l4mpje@10.10.10.134:/Users/L4mpje/AppData/Roaming/mRemoteNG/confCons.xml .
```

或者下载到本地自行玩...  

有很多种方法可以解析 xml 的 base64 值...

```
（[链接](http://hackersvanguard.com/mremoteng-insecure-password-storage/)）
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzYuMeK2aRK039Nm41uLxjId9IS4kCUJ76Cm2GnZd4I8iaOtn3qBkNCVg/640?wx_fmt=png)

我这里在网上搜索之后... 在谷歌发现 github mremoteng_decrypt.py 可以利用... 非常便捷

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzsml8tCHUXiabYmlwKUfKJzibWOmInTRMahicmKJWIyBUaPHd88MUgDOIQ/640?wx_fmt=png)

我利用 python3 执行 mremoteng_decrypt.py 脚本发现不存在 Cryptodome 模块... 以为没安装 pycrypto，发现装在了 python2.7 上了...

（先卸载 crypto 和 pycrypto 即 sudo pip uninstall crypto 和 sudo pip uninstall pycrypto，在安装 crypto 即 sudo pip install pycrypto--- 另外一种方法）

运行 python 执行获得：

```
Password: bureaulampje   ---这是L4用户的密码...
Password: thXLHM96BeKL0ER2   ---adminidsrator用户的密码...
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhosibUavjRYNCrUOtNU8nzwNLaacdbhq46icRLibI8OABeTsoibPFicUaGUzPNdF6EXRBU2d10n9mbrQ/640?wx_fmt=png)

成功使用密码登陆 administrator，获得 root.txt...

![](https://mmbiz.qpic.cn/mmbiz_png/RKmmCHT73fdQQ2nv9rDeddIlJk71QWHcslefZEPQxvuVzXNn9ZlY6dicKOiaJQBXNFYkbHtUsOw0duN5FIUuItSA/640?wx_fmt=png)

非常不错的一台靶机... 学习到了很多东西...

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