> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)

作者：谢公子

  

CSDN 安全博客专家，擅长渗透测试、Web 安全攻防、红蓝对抗。其自有公众号：谢公子学安全

免责声明：本公众号发布的文章均转载自互联网或经作者投稿授权的原创，文末已注明出处，其内容和图片版权归原网站或作者本人所有，并不代表安全 + 的观点，若有无意侵权或转载不当之处请联系我们处理，谢谢合作！

**欢迎各位添加微信号：qinchang_198231** 

**加入安全 + 交流群 和大佬们一起交流安全技术**

**SPN**

**SPN**(ServicePrincipal Names)服务主体名称，是服务实例 (比如：HTTP、SMB、MySQL 等服务) 的唯一标识符。

Kerberos 认证过程使用 SPN 将服务实例与服务登录账户相关联，如果想使用 Kerberos 协议来认证服务，那么必须正确配置 SPN。如果在整个林或域中的计算机上安装多个服务实例，则每个实例都必须具有自己的 SPN。如果客户端可能使用多个名称进行身份验证，则给定服务实例可以具有多个 SPN。SPN 始终包含运行服务实例的主机的名称，因此服务实例可以为其主机的每个名称或别名注册 SPN。一个用户账户下可以有多个 SPN，但一个 SPN 只能注册到一个账户。在内网中，SPN 扫描通过查询向域控服务器执行服务发现。这对于红队而言，可以帮助他们识别正在运行重要服务的主机，如终端，交换机等。SPN 的识别是 kerberoasting 攻击的第一步。

♬..♩~ ♫. ♪ ~ ♬..♩..♩~ ♫. ♪ ~ ♬..♩..♩~ ♫. ♪ ~ ♬..♩..♩~ ♫. ♪ ~ ♬..♩

♫. ♪ ~ ♬..♩~ ♫. ♪..♩~ ♫. ♪ ~ ♬..♩..♩~ ♫. ♪ ~ ♬..♩..♩~ ♫. ♪ ~ ♬..♩

**下面通过一个例子来说明 SPN 的作用：**

当某用户需要访问 MySQL 服务时，系统会以当前用户的身份向域控查询 SPN 为 MySQL 的记录。当找到该 SPN 记录后，用户会再次与 KDC 通信，将 KDC 发放的 TGT 作为身份凭据发送给 KDC，并将需要访问的 SPN 发送给 KDC。KDC 中的 TGS 服务对 TGT 进行解密。确认无误后，由 TGS 将一张允许访问该 SPN 所对应的服务的 ST 服务票据和该 SPN 所对应的服务的地址发送给用户，用户使用该票据即可访问 MySQL 服务。

SPN 分为两种类型：

  

1. 一种是注册在活动目录的机器帐户 (Computers) 下，当一个服务的权限为 Local System 或 Network Service，则 SPN 注册在机器帐户 (Computers) 下。域中的每个机器都会有注册两个 2.SPN：HOST / 主机名  和  HOST / 主机名. xie.com

另一种是注册在活动目录的域用户帐户 (Users) 下，当一个服务的权限为一个域用户，则 SPN 注册在域用户帐户 (Users) 下。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwXo7ibwdXR5O49CjcJ6YP8wErw1RzhdbSErgX7LhlVXJvwDqvaOzicm6Q/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

这里以 SQLServer 服务注册为例：

  

SQLServer 在每次启动的时候，都会去尝试用自己的启动账号注册 SPN。但是在 Windows 域里，默认普通机器账号有权注册 SPN，但是普通域用户账号是没有权注册 SPN 的。这就会导致这样一个现象，SQL Server 如果使用 “Local System account” 来启动，Kerberos 就能够成功，因为 SQL Server 这时可以在 DC 上注册 SPN。如果用一个域用户来启动，Kerberos 就不能成功，因为这时 SPN 注册不上去。

**解决办法：**

*   可以使用工具 SetSPN -S 来手动注册 SPN。但是这不是一个最好的方法，毕竟手工注册不是长久之计。如果 SPN 下次丢了，又要再次手动注册。
    
*   所以比较好的方法，是让 SQL Server 当前启动域账号有注册 SPN 的权力。要在 DC 上为域账号赋予 “Read servicePrincipalName” 和 “Write serverPrincipalName” 的权限即可。
    

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

**SPN 的配置**

微软官方文档：_**https://docs.microsoft.com/zh-cn/windows-server/networking/sdn/security/kerberos-with-spn**_

在 SPN 的语法中存在四种元素，两个必须元素和两个额外元素，其中 <service class> 和 <host> 为必须元素：

```
SPN格式：<service class>/<host>:<port>/<service name>

<service class>：标识服务类的字符串，可以理解为服务的名称，常见的有WWW、MySQL、SMTP、MSSQL等；必须元素
<host>：服务所在主机名，host有两种形式，FQDN(win7.xie.com)和NetBIOS(win7)名；必须元素
<port>：服务端口,如果服务运行在默认端口上,则端口号(port)可以省略；额外元素
<service name>：服务名称,可以省略；额外元素

一些服务的SPN示例：
#Exchange服务
exchangeMDB/ex01.xie.com
#RDP服务
TERMSERV/te01.xie.com
#WSMan/WinRM/PSRemoting服务
WSMAN/ws01.xie.com
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Owz0YdjvNRqyh68zKEGjkd6kar5kicn39KDU4jGpK0WX3qLMoVJeChAsg/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

**使用 SetSPN 注册 SPN**

SetSPN 是一个本地 Windows 二进制文件，可用于检索用户帐户和服务之间的映射。该实用程序可以添加，删除或查看 SPN 注册。

*   主机：win7.xie.com
    
*   域控：win2008.xie.com
    
*   当前用户：xie/test
    

**注：注册 SPN 需要域管理员权限，普通域成员注册会提示权限不够！**

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwOrwzdMaWgJhPre28cOVBpwyI9XvxuVnFVn2z2Pn8lUhfibRJjWGrzxQ/640?wx_fmt=png)

**以 test 用户的身份进行 SPN 服务的注册**

```
setspn -S SQLServer/win7.xie.com:1433 test    #test必须是当前的用户
或
setspn -U -A SQLServer/win7.xie.com:1433 test    #test必须是当前的用户
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwEEq2xDH9aYVDd4YwOZSrUY82Fme3VtPnoQvcCvo4h2BDYfxPRwH2Ag/640?wx_fmt=png)

**以 WIN7 主机的身份在 DC(win2008.xie.com)上进行 SPN 服务 (SQLServer) 的注册**

```
setspn -S SQLServer/win7.xie.com:1533/MSSQL win7    #win7必须是当前的主机名
```

这里由于之前用 test 用户注册过，所以会提示重复，我们可以将端口修改为其他端口，则不是重复的 SPN 了

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow7FfKsmicmEh7kGbo1v7VFIibzU56rxE5xTzUebFh8YAt47f4VHJibibm7A/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

**SPN 的发现**

由于每台服务器都需要注册用于 Kerberos 身份验证服务的 SPN，因此这为在不进行大规模端口扫描的情况下收集有关内网域环境的信息提供了一个更加隐蔽的方法。

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

使用 SetSPN 查询：

  

windows 系统自带的 setspn 可以查询域内的 SPN。

```
查看当前域内所有的SPN：setspn  -Q  */*
查看指定域xie.com注册的SPN：setspn -T xie.com -Q */*      如果指定域不存在，则默认切换到查找本域的SPN
查找本域内重复的SPN：setspn -X
删除指定SPN：setspn -D MySQL/win7.xie.com:1433/MSSQL hack
查找指定用户/主机名注册的SPN：setspn -L username/hostname
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwicRuWsgdU3TcYv7Mw5Jiatntvq1c3JKgNrpCiaX9kibic91Z7J6f7ySUNbA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Owtb5xia0iaev0Azs0hjVdHOSMDXiaUnJbyDXuee9yChiaH6KTZ6bOBxKpEg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwMiax327OQWCCZ7sbjbjI1f11pmZTFhbbib7VquvgPSJtNJmlP3IXAmdg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwB4qF0ibNyf193UONicL0u7agicqiaDsL3mBMAdNsZVaMNeN8zMjEWf7NLA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

PowerShell-AD-Recon：

  

该工具包提供了一些探测指定 SPN 的脚本，例如 Exchange，Microsoft SQLServer，Terminal 等

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwJiaYpfDslgJPtEXgEZqPOUS3dL7cA7p8yicgJsEaNHHOaShFbNnLmCuw/640?wx_fmt=png)

```
#Discover-PSMSSQLServers.ps1的使用,扫描MSSQL服务
Import-Module .\Discover-PSMSSQLServers.ps1;Discover-PSMSSQLServers

#Discover-PSMSExchangeServers.ps1的使用，扫描Exchange服务
Import-Module .\Discover-PSMSExchangeServers.ps1;Discover-PSMSExchangeServers

#扫描域中所有的SPN信息
Import-Module .\Discover-PSInterestingServices.ps1;Discover-PSInterestingServices
```

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

GetUserSPNs.ps1：

  

GetUserSPNs 是 Kerberoast 工具集中的一个 powershell 脚本，用来查询域内用户注册的 SPN。

```
Import-Module .\GetUserSPNs.ps1
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwYg3JvMvBQ4AtDw8nG1CbkSLdEmXib5kZFTpmnGEgE4uiajtYJV6qt41A/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

GetUserSPNs.vbs：

  

GetUserSPNs 是 Kerberoast 工具集中的一个 vbs 脚本，用来查询域内用户注册的 SPN。

```
cscript .\GetUserSPNs.vbs
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwZJ1N9IH7iauxJlXl8ovJI4ASUcAK0rduue0deyajWoCNub6XG9UibxVA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

PowerView.ps1：

  

PowerView 是 PowerSpolit 中 Recon 目录下的一个 powershell 脚本，PowerView 相对于上面几种是根据不同用户的 objectsid 来返回，返回的信息更加详细。

```
Import-Module .\PowerView.ps1
Get-NetUser -SPN
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwZ9bxbRAjN6ooq9eVErynyFfYjVMcMQPkuCicHEQqyqI6Y9BBgG3DC9w/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

PowerShellery：

  

PowerShellery 下有各种各样针对服务 SPN 探测的脚本。其中一些需要 PowerShell v2.0 的环境，还有一些则需要 PowerShell v3.0 环境。

```
#Powershellery/Stable-ish/Get-SPN/ 下Get-SPN.psm1脚本的使用,需要powershell3.0及以上版本才能使用
Import-Module .\Get-SPN.psm1
Get-SPN -type service -search "*"
Get-SPN -type service -search "*" -List yes | Format-Table

#Powershellery/Stable-ish/ADS/ 下Get-DomainSpn.psm1脚本的使用
Import-Module .\Get-DomainSpn.psm1
Get-DomainSpn
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwICBHJTEA3lic0FXQ8VtLLZj4wEKmICiaEiaYF205lZjb0oXhJdZmiaRh2A/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42Ow5IErX3JmYwwL1r5SSX9Vj7LYJic3b5DnlkdBdPNukkCLhdqvDxuIaibA/640?wx_fmt=gif)

  

RiskySPN 中的 Find-PotentiallyCrackableAccounts.ps1：

  

该脚本可以帮助我们自动识别弱服务票据，主要作用是对属于用户的可用服务票据执行审计，并根据用户帐户和密码过期时限来查找最容易包含弱密码的票据。

```
Import-Module .\Find-PotentiallyCrackableAccounts.ps1;Find-PotentiallyCrackableAccounts -FullData -Verbose
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwU9B5GnoRZiac8ldX6DvN5Kric5KefC9Yh4npuZOVVJS2nrficY1Z1aDNQ/640?wx_fmt=png)

该脚本将提供比 klist 和 Mimikatz 更详细的输出，包括组信息，密码有效期和破解窗口。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwfD6pyl47wkX6tX49GZO5MxE6nwpGz0EfSJd3XtTdibWAuvqTaXDPDbg/640?wx_fmt=png)

使用 domain 参数，将返回所有具有关联服务主体名称的用户帐户，也就是将返回所有 SPN 注册在域用户下的用户。

```
Import-Module .\Find-PotentiallyCrackableAccounts.ps1;Find-PotentiallyCrackableAccounts -Domain "xie.com"
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwzOI0OxP3J6Ir4LN5INvKWpHd6sBmhib95SItsyVDQMgajc5HQLGsAAw/640?wx_fmt=png)

**关注公众号：谢公子学安全，回复** **SPN** **即可获取下载链接**

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwLrDM2wibABBoHl68xiclRupO8FjzPDr5I6TTqDq9Vpt0tU3bB90DobRg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwsicKq0aoQQQZ6EohxEHfOgBW1zEsoic3aeG74NGcC3c7yXibVEdgetjJA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgeslPbe0qkuKAwYU3q42OwibuUWlOkCvHRFHkkXc67gRRNQXTGyhEUliaDPjclvhS0xGkMXVvpB8hQ/640?wx_fmt=png)

[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)  

[内网渗透（二） | MSF 和 CobaltStrike 联动](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[内网渗透 | 域内认证之 Kerberos 协议详解](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)