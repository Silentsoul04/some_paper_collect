> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GXvz4jXc8IPVQcXkB6L6_w) ![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5snruuNFhPCFY915Oiaj5VBjTTLPo95NKquoO8AnoiatHialznd6eQkia0tA/640?wx_fmt=png)

前言
--

远程桌面在内网渗透中可以说是再常见不过了，在渗透测试中，拿下一台主机后有时候会选择开 3389 进远程桌面查看一下对方主机内有无一些有价值的东西可以利用。对远程桌面的利用姿势有很多，本篇文章中我们来学习一下 RDP 会话劫持的相关利用姿势。

RDP 劫持的原理
---------

> 系统管理员和用户通常可以通过 RDP 远程桌面登录指定服务器 3389 远程桌面，而攻击者可以通过可以特权提升至 SYSTEM 权限的用户，可以在不知道其他用户登录凭据的情况下，用来劫持其他用户的 RDP 会话，该漏洞在 2017 年由以色列安全研究员 Alexander Korznikov 在个人博客中披露。利用条件只需要获取机器 SYSTEM 权限执行 tscon 命令。

对于开启远程桌面服务的 Windows 系统，当有多个用户登录该系统时，会产生多个会话，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sLIw6icEpGFDSXIp51dtr74KKNcaPXT5oeUNyNww9ib4zdzupRbTWNEjQ/640?wx_fmt=png)image-20210523173030619

其中，管理员用户 Administrator 为本地登录，用户 bunny 为通过远程桌面服务（RDP）连接 3389 端口的远程桌面登录。接下来，如果用户 Administrator 想要切换至用户 bunny 的远程桌面，可通过右键—> 连接（Connect）进行连接，接着输入密码即可切换到 bunny 用户：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sAKILTpUWOFKYhKfEN1sX3SbBR0LibWksEVrmxh0LeWchVEeHLHEZnYg/640?wx_fmt=png)image-20210523173120796![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sXsyeUp9TMibAq5o5Yhvd3J4FTSH5B3ZOibC34FndjIE9s73sib8RYo87g/640?wx_fmt=png)image-20210523173145771

点击确定后，如下图所示，成功切换到了 bunny 用户的远程桌面：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sZhgZjHqe17jFl9icmgtEw8lDKIRjuAX0Yd8523xr3YblrHO6ZibJk1ng/640?wx_fmt=png)image-20210523172737212

而且，在 Windows 中有一个 tscon 命令，是命令行下使用的工具，也可以实现与上述相同的功能。

首先执行如下命令获取用户对应的会话 ID：

```
query user
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sdFicRFiaqJTibJCxSQyF8axF22WgLOSEqXmzl7PFjjvEDTYAicV0dU3FpQ/640?wx_fmt=png)image-20210523173320885

可以看到用户 bunny 对应的会话 ID 为 2，然后通过执行 tscon 命令即可成功切换至用户 bunny 的远程桌面，命令如下：

```
tscon 2 /PASSWORD:Bunny2021
```

•/PASSWORD：bunny 用户的密码

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sZhgZjHqe17jFl9icmgtEw8lDKIRjuAX0Yd8523xr3YblrHO6ZibJk1ng/640?wx_fmt=png)image-20210523172737212

可见，tscon 命令提供了一个切换用户会话的功能，并且，在正常情况下，切换会话时需要提供目标用户的登录密码。但这并不能完全确保会话安全，攻击者通过特殊的利用方法完全能够绕过验证，不输入密码即可切换到目标会话，从而实现目标用户的未授权登录。

而这里所讲的特殊的利用方法便是在 SYSTEM 权限下直接执行 tscon 会话切换命令：

```
tscon ID
```

此时攻击者可以在不提供其他用户登录凭据的情况下自由切换会话桌面，实现劫持其他用户的 RDP 会话。

RDP 会话劫持在特定情况下可以大显身手，比如对于较新的 Windows 系统，默认情况下是无法通过 Mimikatz 导出用户明文口令的，此时我们通过常规方法无法切换至另一用户的桌面，那么我们便可以借助上文提到的方法，先提权至 SYSTEM 权限，再劫持目标用户的 RDP 并切换过去。

特别注意的是，即使远程连接的用户关闭了远程连接窗口，也不会劫持该回话，只是在后台显示 “已断开连接”（Disconnected）：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sIPlZh31s2cIUmnl9xA9cGpFbuBECSRHxy5cicuKqMZNOG9QJpoz5WIw/640?wx_fmt=png)image-20210523181120642

此时，仍能在 SYSTEM 权限下通过 `tscon` 实现未授权连接。

高权限用户劫持低权限用户的 RDP
-----------------

高权限用户劫持低权限用户的 RDP 会话利用起来比较简单，由于具有管理员权限，可以直接通过创建服务等方式获取 SYSTEM 权限。

创建劫持用户会话的服务：

```
sc create rdp binpath= "cmd.exe /k tscon 2 /dest:console"sc start rdp
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s24Hz9RGPwEMROlalqq0TvW2Tfn3A1AKNAib447FvVIYAI6zicMGrut7Q/640?wx_fmt=png)image-20210523174510831

执行 `sc start rdp` 后，我们创建的劫持会话的服务将会启动，由于 Windows 是以 SYSTEM 权限运行服务的，所以我们 `tscon 2` 命令也会以 SYSTEM 权限运行，此时便可以在不提供目标用户密码的情况下成功劫持目标用户的会话：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s366icpRwnGd8z5bIqQfo5nMpttAjF2wH5yytEv2LGXfRNVEIBJpN4qQ/640?wx_fmt=png)image-20210523174727394

其实也可以使用 Psexec 来获得一个 SYSTEM 权限的 cmd（Psexec 获得的 shell 是 SYSTEM 权限的），然后再这个 SYSTEM 权限的 cmd 中直接执行 `tscon 2` 劫持命令：

```
psexec -s -i cmd    # 获得一个 SYSTEM 权限的 cmdquser user    # 在新获得的 SYSTEM 权限的 cmd 中执行劫持命令tscon 2 /dest:console
```

低权限用户劫持高权限用户的 RDP
-----------------

低权限用户劫持高权限用户的 RDP 会话利用起来没有前者那么简单，因为权限太低，所以无法执行创建服务，执行 Psexec 等高权限的命令。所以如果低权限用户想要劫持高权限用户的 RDP 的话需要想办法提权，即将自己的权限提升至 SYSTEM。

实验环境如下：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s3N12nicHoUY84F0V4NfthySmZFFAQ3icOIAzkibST22VxU4Dj4XFHR6icg/640?wx_fmt=png)image-20210523180237610

假设有这么一种情况，有一台 Windows Server 2012 系统的服务器，其本地登录着一个普通域用户 bunny：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sG2MFds5hK9v26r46VMcy5aA64NxjK08ugFMPYagSh9cXxJwAr8t5Gw/640?wx_fmt=png)image-20210523170231958

我们通过某种方式获得了这个 bunny 用户的登录密码，并使用这个 bunny 用户成功进行远程登录：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s9ibDGEDic7hlo107v8RdapqvDjUX3xHEmF4TiaraJwTa9pTIOZPIFF02w/640?wx_fmt=png)image-20210523170429634

此时，登录后查看任务管理器发现后台还存在管理员用户 Administrator 的会话：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s6iaJQmMKbvTCvqoxPkl1YMJDqIfib7Egw2J5ibnoFqMJcdla6icEyiaXiaBw/640?wx_fmt=png)image-20210523171040725

并且使用 `query user` 命令查看其会话 ID 为 1。接下来我们尝试劫持这个管理员用户的远程会话。

首先使目标主机上线一个 bunny 用户权限的 MSF，然后通过各种系统漏洞获得了目标机的 System 权限：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5suUlYCrqic8XqlpK0sEvaB2Q7iaDhQhicKTITkUgLxdcDKCAN4c4zvGIYg/640?wx_fmt=png)

然后进入 shell 中执行 tscon 命令进行劫持即可：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sdYWXqCsBmicocOwYc52ENadWe00n7hn5DyY8uvXjzhl4IHiaSD8pW4Uw/640?wx_fmt=png)image-20210523171756440

如上图所示，成功劫持并切换到了 Administrator 用户的远程桌面。

配合远程桌面辅助功能后门的利用
---------------

相信你一定知道 Windows 粘滞键后门，如果你在电脑上连按五次 shift 键，你就会发现电脑屏幕上弹出了一个叫做 “粘滞键” 的程序，即使没有登录进系统：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5stHhOG66m559X4wpPg6eeWwSyw3DO4OGZpFVUeRicBF19GJmXuMsyqWQ/640?wx_fmt=png)image-20210524103630570

这个粘滞键程序名称为 “sethc.exe”，其路径为 “c:\windows\system32\sethc.exe”。利用粘滞键做后门是一种比较常见的持续控制方法。其基本流程就是找到 sethc.exe 将其删除或改名为 sethc.exe.bak，接着将 cmd.exe 程序复制一个副本，并命名为 “sethc.exe”：

```
cd c:\windows\system32  move sethc.exe sethc.exe.bak   // 将sethc.exe重命名copy cmd.exe sethc.exe       // 将一个cmd.exe副本保存伪装成sethc.exe
```

最后，重启计算机再次按下 5 次 Shift 键时，就会弹出 CMD 界面，后门制作成功。

除了粘滞键 sethc 外，在 Windows 登录界面上还有很多辅助功能，比如屏幕键盘，放大镜，屏幕阅读等，这些辅助功能都可以像粘滞键 sethc 一样被攻击者用来制作一个后门。

Metasploit 中的 `post/windows/manage/sticky_keys` 模块可实现自动化地利用沾滞键的权限维持技术。该模块将用 cmd.exe 替换那些辅助功能的二进制文件（sethc、osk、disp、utilman）：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sQ3ibYiaLCiaGVwibRWhVIkC2RI4QMscnrSICdyEtJWm8kkqibpvwLialYmBA/640?wx_fmt=png)image-20210524121055853

使用方法如下：

```
use post/windows/manage/sticky_keysset session 6set target UTILMANexploit
```

执行成功后，我们开启目标主机的远程桌面，当我们点击左下角的辅助功能按钮后，成功弹出了 CMD 窗口，并且为 SYSTEM 权限的：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5scQtJqhBe9STDGbphoUuWgGnpMSGls9tPL8J4jiaaVspzzpcHzkHYialg/640?wx_fmt=png)image-20210524121931421

由于此时获得的 CMD 是 SYSTEM 权限的，所以我们这里可以直接配合 RDP 劫持进去目标系统。如下图所示，发现目标主机上有三个用户的会话，那我们便可以通过 `tscon` 进行随意的劫持与切换：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sHwRRAka24LYQYuAqAW2joIOvGI5rxeZWKGvckHfroaZ1aEos9OFaqw/640?wx_fmt=png)image-20210524110134327

执行 `tscon 1` 命令后，如下图所示，成功劫持并切换到了 administrator 用户的会话：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sjVw6tv2Bu8XPv4UFEsIpYY1574aiaic33oXNw6FvicUxlNmkICEWmHsjA/640?wx_fmt=png)image-20210524110255414

实战案例
----

千辛万苦拿到的域控权限竟然被上线的管理员发现，不仅修改了我们已知域用户的密码，而且把我们新创建的隐藏后门用户也删了，远程桌面进不去了：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sFZyhVIL7JS0boO0CZR3icAyao8Ycd8M9vd1FGrodkxdfBxrw37N4Qqw/640?wx_fmt=png)image-20210613010754191

但如果此时由于某些需求你必须进入远程桌面的话该怎么办呢？你可以选择不停地改密码、创建用户，但这也不是长久之计。在这种情况下我们还可以利用 Windows 登录桌面的辅助功能配合 RDP 劫持，无需任何用户凭据即可进入目标系统桌面。

直接使用 Metasploit 自带的粘滞键后门模块创建 Shift 后门：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5suLNCJYiaqN6BXRZadGYnXDEvRQ3iaWM3DvqAnaz7Mlibqz1uGs6TFOEbA/640?wx_fmt=png)image-20210613010007275

成功创建后，打开远程桌面登录界面，按下五次 Shift 键后弹出 CMD 窗口，执行 query user 命令可以看到目标主机上的会话，此时虽然会话是断开了的，但是我们仍能在 SYSTEM 权限下通过 `tscon` 命令进行 RDP 劫持实现未授权连接：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5snruuNFhPCFY915Oiaj5VBjTTLPo95NKquoO8AnoiatHialznd6eQkia0tA/640?wx_fmt=png)image-20210613011023770

如下图所示，执行 `tscon 3` 后成功进入目标主机的远程桌面：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s4C8wuFM9GoEKszsJQxaRYausRm5shsPsXzaggiawnHNuehYDN0Yg8QA/640?wx_fmt=png)image-20210613011529092

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动

[![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibHHibNbEpsAMia19jkGuuz9tTIfiauo7fjdWicOTGhPibiat3Kt90m1icJc9VoX8KbdFsB6plzmBCTjGDibQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496998&idx=1&sn=da047300e19463fc88fcd3e76fda4203&chksm=ec1ca019db6b290f06c736843c2713464a65e6b6dbeac9699abf0b0a34d5ef442de4654d8308&scene=21#wechat_redirect)

**点赞，转发，在看**

投稿作者：WHOAMI

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)