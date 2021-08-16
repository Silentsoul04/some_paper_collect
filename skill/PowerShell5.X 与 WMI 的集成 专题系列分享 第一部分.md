> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NZeFX7gM59NuUtDCey2ycw)

> 本文作者：****贝多芬不忧伤****（Ms08067 内网安全小组成员）

贝多芬不忧伤微信（欢迎骚扰交流）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaiclSAAMIhNerQBdTuIugAkzydQ3Dnu2ziaTlVDqNk6NTcpX23SAEvsXl80kzfPKsibCpnvX7Jv6AgaA/640?wx_fmt=jpeg)

    众所周知，在 windows10 以及 Windows Server2016 的平台当中，PowerShell5.x 已经能够去获取到系 统当中大部分的信息，但有时候仍有一些信息需要借助于调用 WMI 的类来完成。通过本文，你可以 了解到 WMI 的基本架构和组件，包括 WMI 的数据库，WMI 的 provider，以及在 PowerShell 调用 WMI 的 时候提供的 module 和相关的命令。接下来我们就能通过 powershell 的命令去完成 WMI 的查询操作， 去获取到系统当中 WMI 的实例。然后我们还可通过实例的属性查看到系统当中不同的信息，同时的 话去调用实例当中为我们提供的不同的方法，去修改系统信息的配置。

**1. WMI 简介**

    在这一节当中，将为大家介绍 WindowsPowerShell 与 WMI 的集成，通过这一节的介绍，首先我们就 能够了解到在 windows 的运行平台当中什么是 WMI，WMI 的基本定义以及 WMI 与 PowerShell 的基本关 系。接下来即可在演示的环境当中去完成一些基本的演示。通过这些演示，我们就可以了解到在一 些特定的场景当中，powershell 必须依赖 WMI 才能去完成自己的任务。  

**1.1 What is WMI  
**

    在了解 WMI 的基本定义之前，我们需要了解四个不同的名词，分别为  

DMTF\CIM\WBEM(WSMAN)\WMI(WinRM)。  

**DMTF**  

*   DMTF 是一个标准化组织的缩写（Distribusted Management Task Force）  
    
*   该组织所制定的标准最主要的作用是方便 IT 人员或者是软件开发人员能够更加方便的去管理任 何的 IT 运行环境（Sets standards to enable effective management of any IT enviroment）  
    
*   比如说，假设当前我们有一台服务器，而这台服务器当中安装了操作系统和应用程序。通过 DMTF 制定的标准，软件开发人员或者 IT 运维人员就可以使用同样的方法去获取到这一台服务 器它的品牌和型号，以及操作系统的类型和应用程序的信息。这样我们就屏蔽底层系统或者硬 件的差异 ，而通过同一种方式去获取到这些目标的基本信息。  
    
*   那么要去实现这一目标的时候，DMTF 这个标准化的组织就为我们去制定了两个非常关键的标 准。首先第一个标准就是 CIM。  
    

**CIM（Common Information Model）**  

*   这个标准就定义了系统当中软件硬件所有的信息发布的时候的基本元素，那么所有软件或者硬 件的厂商在发布新产品的时候都会来兼容 CIM 这个标准（DMTF standard that defines the managed elements）  
    
*   在这个 CIM 的标准当中，我们就能够去找到这样的一些基本信息。那有了 CIM 这个基本的标准 之后，IT 人员又是如果去获取这些信息的呢? 其实 DMTF 还为我们制定了另外一套标准，即 WBEM / WS-MAN。  
    

**WBEM / WS-MAN （Web-Base Enterprise Management）**  

*   这个标准其实就是为我们提供了一个访问的接口，那么通过这个访问的接口，我们就能够去访 问到 CIM 这个通用模型当中相应的信息，从而我们就可以了解到一个系统当中它的服务器信 息，操作系统的信息，以及软件的信息。（Web Services Management）
    

**WMI（Windows Management Instrumentation）**  

*   WMI 就是在 windows 的平台当中的 CIM 的模型，即 Microsoft's CIM-Server for Windows  
    

**WinRM（Windwos Remote Management）**  

*   WinRM 就是在 windows 的平台当中的 WS-MAN 的实现，即 Implements WS-Man in Windows
    

    了解了以上基本信息之后，我们便知道了。在 windows 平台中，我们有了 WMI 这个基本组件之后， 我们就可以编写脚本或者编写相应的代码去获取到系统当中我们想去获取的任何信息（包括操作系 统的信息、软件、硬件、网卡、磁盘以及应用程序的信息等等）。因此我们可以将 WMI 看作一个通 用服务或者模型，通过这个模型就可以获取到自己想要的信息。  

**1.2** **WMI 与 PowerShell**

    众所周知，对于 PowerShell 来说，它为我们提供了不同的 PowerShell Module。在这一系列的 PowerShell Module 当中就包括了很多的 powershell 命令 (cmdlet)，我们就可通过这些命令(cmdlet) 去 获得相应的信息。比如在活动目录当中，我们可以通过 Active Directory Module 去获取到相应的活动 目录当中的用户、计算机、安全组等信息，当然我们也可以去创建用户、计算机、安全组。除此之 外，在 hyperV 的运行环境当中，PowerShell 也为我们提供了 hyperV 的 Module，那我们就能够了解到 虚拟机的基本信息，虚拟机的运行状态，同时也能更改虚拟机的所有配置。 

    WMI 在 windows 中首次出现的时候是在 NT 时代，至今已有二十多年的发展史。反观 PowerShell 是在 2008 年出现的，所以至今也就十余年历史。WMI 经过了二十多年的发展，所以 WMI 获取信息的能力 以及对软硬件兼容的支持程度其实是更优秀的。在一些情况中，如果 powershell 本身提供的命令能 够去获取相应的信息，那也可以使用 powershell 的方式来完成相应的操作，但是如果 powershell 对 某些操作没有相关的命令支持，这时便可以通过 powershell 调用 WMI 的方法去获取相关的信息。

**版本查看：  
**

```
// 查看PowerShell版本
PS C:\> $PSVersionTable.PSVersion

Major Minor Build Revision
----- ----- ----- --------
5     1     18362 1171      //查看命令的结果，其中"Major"既是当前已安装PowerShell的版本号
```

**PowerShell 基本演示：  
**

**Get-ADComputer**  

*   Module : [ActiveDirectory]  
    
*   Gets one or more Active Directory computers.  
    
*   https://docs.microsoft.com/en-us/powershell/module/addsadministration/Get-ADComputer?vi ew=win10-ps
    

```
PS C:\> Get-ADComputer -Filter *

DistinguishedName : CN=DC,OU=Domain Controllers,DC=testLab,DC=com
DNSHostName : DC.testLab.com
Enabled : True
Name : DC
ObjectClass : computer
ObjectGUID : 77dde2fb-7692-4dc9-bcdf-526aa53bf77d
SamAccountName : DC$
SID : S-1-5-21-2941787077-2348910004-141610113-1001
UserPrincipalName :
DistinguishedName : CN=DC3,OU=Domain Controllers,DC=testLab,DC=com
DNSHostName : DC3.testLab.com
Enabled : True
Name : DC3
ObjectClass : computer
ObjectGUID : a61cc09a-a4ec-4a96-8371-219b5ccd7dd8
SamAccountName : DC3$
SID : S-1-5-21-2941787077-2348910004-141610113-5765
UserPrincipalName :
-----------------------------------------------------------------------------
-----------
-----------------------------------------------------------------------------
-----------
PS C:\> Get-ADComputer -Filter * -Properties * | Select-Object
DNSHostName,IPv4Addres,OperatingSystem

DNSHostName           IPv4Addres     OperatingSystem
----------- ---------- ---------------
DC.testLab.com        172.30.8.150   Windows Server 2012 R2 Standard
DC3.testLab.com       172.30.8.151   Windows Server 2012 R2 Standard
zhangsan.testLab.com  172.30.8.152   Windows Server 2016 Standard
guanyu.testLab.com    172.30.8.160   Windows 7 专业版
```

**Get-LocalUser** 

*   Module : [Microsoft.PowerShell.LocalAccounts] 
    
*   Gets local user accounts. 
    
*   https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/getlocaluser?view=powershell-5.1  
    

```
PS C:\> Get-LocalUser
Name Enabled Description
---- ------- -----------
Administrator True 管理计算机(域)的内置帐户
DefaultAccount False 系统管理的用户帐户。
Guest False 供来宾访问计算机或访问域的内置帐户
WDAGUtilityAccount False 系统为 Windows Defender 应用程序防护方案管理和使用的用户帐户。
```

**Invoke-Command**  

*   Module : [Microsoft.PowerShell.Core]  
    
*   Runs commands on local and remote computers.  
    
*   https://docs.microsoft.com/en-us/powershell/module/Microsoft.PowerShell.Core/Invoke-Com mand?view=powershell-5.1
    

```
PS C:\> Invoke-Command -ComputerName zhangsan.testLab.com -ScriptBlock {GetLocalUser}

Name Enabled Description PSComputerName
---- ------- ----------- ---------------
Administrator True 管理计算机(域)的内置帐户
zhangsan.testLab.com
DefaultAccount False 系统管理的用户帐户。
zhangsan.testLab.com
Guest False 供来宾访问计算机或访问域的内置帐户
zhangsan.testLab.com
-----------------------------------------------------------------------------
-----------
-----------------------------------------------------------------------------
-----------
PS C:\> Invoke-Command -ComputerName guanyu.testLab.com -ScriptBlock {GetLocalUser}
无法将“Get-LocalUser”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。请检查名称的拼
写，如果包括路径，请确保路径正确，然后再试一次。
所在位置 行:1 字符: 1
+ CategoryInfo : ObjectNotFound: (Get-LocalUser:String)
[],CommandNotFoundException
+ FullyQualifiedErrorId : CommandNotFoundException
+ PSComputerName : guanyu.testLab.com
```

**WMI 基本演示：** 

**Get-WmiObject** 

*   Module : [Microsoft.PowerShell.Management] 
    
*   Gets instances of Windows Management Instrumentation (WMI) classes or information about the available classes. 
    
*   https://docs.microsoft.com/en-us/powershell/module/Microsoft.PowerShell.Management/GetWmiObject?view=powershell-5.1
    

```
PS C:\> Get-WmiObject -Class Win32_UserAccount
AccountType : 512
Caption : R90LT26V\Administrator
Domain : R90LT26V
SID : S-1-5-21-416148207-2992492597-1317475389-500
FullName :
Name : Administrator
AccountType : 512
Caption : R90LT26V\DefaultAccount
Domain : R90LT26V
SID : S-1-5-21-416148207-2992492597-1317475389-503
FullName :
Name : DefaultAccount
AccountType : 512
Caption : R90LT26V\Guest
Domain : R90LT26V
SID : S-1-5-21-416148207-2992492597-1317475389-501
FullName :
Name : Guest
```

通过以上的基本演示，我们就能够了解到 PowerShell 当中可能有的命令是没有的，在不同的系统当 中，它还在进行一个演进的过程。而 WMI 相对更加成熟，功能更加完善。  

**未完待续，下一次我将为大家讲解 WMI 架构的相关知识 ....  
**

**扫描二维码，****加入内网小组，共同成长！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9l9FNcvFsJZ3BvAGHhTVOzKWkTtDU8iaQePhu5BbEibDolILr4Qrh9qa4f0xibBc0b9814Uiaq604kUQ/640?wx_fmt=png)

**扫描下方****二维码学习更多安全知识！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

 ![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)

**目前 30000 + 人已关注加入我们**

![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)