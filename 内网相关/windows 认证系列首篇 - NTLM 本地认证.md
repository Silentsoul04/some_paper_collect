> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_ZeYSruAG2OQhrji9A3NJw)

本篇文章将给大家介绍 Windows NTLM 认证的相关内容，包括 NTLM  的概念与相关安全问题的介绍，后续还会出包括 kerbros 认证的整个认证系列篇，慢慢帮助不太关注理论的师傅扎好马步，丝毫不夸张的说，对认证系列的学习是 windows 内网渗透最重要一环。工具五花八门永远在变，原理是永远不变的。

步入正题：

01

Windows 主要采用另一种认证协议——NTLM（NT Lan Manager）。NTLM 使用在 Windows NT 和 Windows 2000  Server（or later）工作组环境中（Kerberos 用在域模式下）。在 AD 域环境中，如果需要认证 Windows  NT 系统，也必须采用 NTLM。较之 Kerberos，基于 NTLM 的认证过程要简单很多。NTLM 采用一种质询 / 应答（Challenge/Response）消息交换模式。

02

看完了上面的概念，我们来讲讲当用户在 Windows 本地登录时，密码文件存放在了哪里![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFjp6ianAkZzsUJBicUItVybm66x4ptYZ3hzu0dpGJ5OAIlAgZ4VDfD8cUFEddpuTU0e9frWU9PnMz2w/640?wx_fmt=png)？我们用户的密码存储在本地计算机的 SAM 这个文件里。

SAM 文件的路径为：

```
C:\Windows\System32\config
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFia46zBNh5WrZiavZKzkpIIFrnUO4QmMMIkbiaRcVQBe5BaKx9Oxd9hIrnaxayJu7vjTYWjzVVHrVYSQ/640?wx_fmt=png)

在 Windows 中是不会保存明文密码的，只会保存密码的哈希值。其中本机用户的密码哈希是放在 本地的 SAM 文件 里面。

03

这个文件怎么搞出来呢？

我们实战中的通用方式便是通过 SAM 数据库获得本地用户 HASH。

sam 文件：是用来存储本地用户账号密码的文件的数据库  
system 文件：里面有对 sam 文件进行加密和加密的密钥

**利用方式：(前提是高权限用户才能导）**

**导出 sam 和 system：**

```
reg save hklm\sam sam.hiv
```

****读取`HKLM\SYSTEM`，获得`syskey`：****

**syskey 的由来：**读取注册表项`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa`下的键值`JD`、`Skew1`、`GBG`和`Data  
`

```
reg save hklm\system system.hiv
```

**syskey 的作用：**Syskey 中的加密的是账号数据库，也就是位于`%SystemRoot%\system32\config`的 SAM 文件

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFjp6ianAkZzsUJBicUItVybm6rSBHefEGN4ykcpLtDHBDmkx2v7ibz5lsK9osbY0iaZFclgJF5Zz8EVzg/640?wx_fmt=png)

可以看到两个 hiv 后缀的文件被导出，实战中可以拷回 web 路径下载下来，或者各种方式把他压缩后弄出来就行。  

****解密工具 mimikatz：****

lsadump::sam /sam:sam.hiv /system:system.hiv

文件拉回本地进入 ****mimikatz**** 解码这两个文件，当然解密方式不止一种。

老规矩给 mimkatz 提上权限后执行

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFjp6ianAkZzsUJBicUItVybm6WcraJLEwG1picbNnfWdAFwPJJodEMzNt1l2EkgumVNV7lqS8ibqBsVJA/640?wx_fmt=png)  

04

关于 lsass.exe：

lsass.exe 是一个系统进程，用于微软 Windows 系统的安全机制。它用于本地安全和登陆策略，这个进程中会存一份明文密码，将明文密码加密成 NTLM Hash，对 SAM 数据库比较认证。

当用户输入密码进行本地认证的过程中，用户输入的密码将为被转化为 NTLM Hash，然后与 SAM 中的 NTLM  Hash 进行比较。当用户注销、重启、锁屏后，操作系统会让 winlogon.exe 显示登录界面（输入框）。

```
winlogon.exe -> 接收用户输入 -> lsass.exe -> (认证)
```

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" "exit" > lsass.txt  //解密
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFia46zBNh5WrZiavZKzkpIIFr4fJo1NpaH9FzA244djIiaChA3uuaCSDQSfwkPUHL1ibqrd2pURhS3EdQ/640?wx_fmt=png)

当 winlogon.exe  接收输入后，会将密码交给 lsass 进程。

说一说转储 lsass.exe 的常用技巧：  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFia46zBNh5WrZiavZKzkpIIFrkmpLeYcAcicicqr7jwmS4yqPcewxzv0ntcAYbUwBDugPXSGZbRXYFSrw/640?wx_fmt=png)

当然他也是可以手动进行转储的，直接右击转储即可。

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFia46zBNh5WrZiavZKzkpIIFr8VGl0eKMI3oYZ5JO6eZyloc4h9ouKzkyDy1xfQv0RHuqeib5f75969w/640?wx_fmt=png)

不过实战中没人会这么无聊上 3389 转储这个玩意，一般的思路都是 procdump.exe 做进程转储。

05

工具转储：procdump.exe

实战中用 procdump.exe 来 dump 对方主机的 lsass 内存文件，然后用 mimikatz 等工具进行处理。这种方式的好处是可以避免被查杀  

Procdump 是一个轻量级的 Sysinternal 团队开发的命令行工具, 它的主要目的是监控应用程序的 CPU 异常动向,  并在此异常时生成 crash dump 文件, 供研发人员和管理员确定问题发生的原因. 你还可以把它作为生成 dump 的工具使用在其他的脚本中，简单的说，_ProcDump_ 是一款 dump 抓取和分析工具。绝大多是情况下都是可以正常实用，但很多杀软都做起来防范，例如卡巴斯基等。

lsass 进程转储获取密码：

procdump.exe -accepteula -ma lsass.exe lsass.dmp    // 进程转储

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFjp6ianAkZzsUJBicUItVybm6DXS9lFmn2BicnaMYSAb9kyta66D92Ehh0zibXBvatan43mxXIiafcSq0w/640?wx_fmt=png)

执行后便生成了 lsass.dmp，依旧是 mimkatz 解密即可。

```
Administrator:500:AAD3B435B51404EEAAD3B435B51404EE:31D6CFE0D16AE931B73C59D7E0C089C0:::
AAD3B435B51404EEAAD3B435B51404EE是LM Hash
31D6CFE0D16AE931B73C59D7E0C089C0是NTLM Hash
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFjp6ianAkZzsUJBicUItVybm6h2Hf1DxicVjkX6IcceHYbbQvib19M2xxx5VGH2SvXCbgXUP1kPbcWwdw/640?wx_fmt=png)

over!

06

在渗透测试中，通常可从 Windows 系统中的 SAM 文件和域控的 NTDS.dit 文件中导出所有用户的 Hash。导出来的哈希经常会看到这样的格式：

```
小贴士：从Windows Vista 和 Windows Server 2008开始，默认情况下只存储 NTLM Hash，LM Hash 将不再存在
```

LM Hash：

LM Hash 的全称为 LAN Manager Hash，这是 Windows 中最早用的加密算法。

LM Hash 计算方式:

> 1. 用户的密码转换为大写，密码转换为 16 进制字符串，不足 14 字节将会用 0 来再后面补全。
> 
> 2. 密码的 16 进制字符串被分成两个 7byte 部分。每部分转换成比特流，并且长度位 56bit，长度不足使用 0 在左边补齐长度
> 
> 3. 再分 7bit 为一组, 每组末尾加 0，再组成一组
> 
> 4. 上步骤得到的二组，分别作为 key 为 “`KGS!@#$%`“进行 DES 加密。
> 
> 5. 将加密后的两组拼接在一起，得到最终 LM HASH 值。

```
小贴士：从Windows Vista 和 Windows Server 2008开始，默认情况下只存储 NTLM Hash，LM Hash 将不再存在
```

AAD3B435B51404EEAAD3B435B51404EE。如果空密码或者不储蓄 LM Hash 的话，我们抓到的 LM Hash 是`AAD3B435B51404EEAAD3B435B51404EE`，这里的 LM Hash 已经没有任何价值了。

### NTLM Hash:

### NTLM Hash 是微软弥补 LM Hash 不安全性的产物。

NT Hash 生成的算法：

> 1. 先将用户密码转换为十六进制格式。
> 
> 2. 将十六进制格式的密码进行 Unicode 编码。
> 
> 3. 使用 MD4 摘要算法对 Unicode 编码数据进行 Hash 计算

关于 Net-NTLM Hash 是一个非常有意思的东西，后面我们讲 HASH 中继会为各位详谈。

07

windows 本地认证十分简单，大体概念为：用户输入密码，系统收到密码后将用户输入的密码计算成 NTLMHash，然后与 sam 数据库（%SystemRoot%\system32\config\sam）中该用户的哈希比对，匹配则登陆成功，不匹配则登陆失败。

08

总结：

windows 认证理念这个东西，时而回头看看，蹲蹲马步，可以更好理解 windows 域渗透，为游走内网打好基础，毕竟域内渗透都在和 windows 机器打交道，最后的目的就是摸到域控，重要的理论知识也不可不抓。

未完待续 --------  

有啥事欢迎私下骚扰

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/ibHceSKEmlFjp6ianAkZzsUJBicUItVybm6vpZZgEPZDiczD5ufHqTRdzFR2rzV0icJZSk4QVHXt3W9IQ5ekgvvMdOA/640?wx_fmt=jpeg)

微信号 | attack_tubai  

分享收藏点赞在看