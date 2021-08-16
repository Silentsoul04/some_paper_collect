> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GceeUFbwsaHUwKQpY6VFkw)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

**所有话题标签：**

[#Web 安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808926912513&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#漏洞复现](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808859803651&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#工具使用](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1556485811410419713&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#权限提升](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559100355605544960&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#权限维持](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1554692262662619137&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#防护绕过](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553424967114014720&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#内网安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102220258885633&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#实战案例](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553386251775492098&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#其他笔记](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102973052567553&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#资源分享](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [#MSF](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1570778197200322561&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

 |

**0x01 前言**

朋友发来的是一个支持外链的 MSSQL，未做站库分离处理，且可以直接通过 xp_cmdshell 执行命令，但是由于目标系统为 Windows 2019，自带的有微软的 Windows Defender 防病毒，他在执行 PowerShell 攻击命令时被拦截后不知道要怎么绕过，所以让我帮着给看下。  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0ulaXCUibjbQxT5tJsQdtQKA5b2kiagkQibiaJEpBhcfRuuqWoURV1ttX5g/640?wx_fmt=png)

  

**0x02 信息搜集**

****目标机器基本信息：****  

```
目标系统：Windows Server 2019（10.0.17763 暂缺 Build 17763）
数据库版本：Microsoft SQL Server 2014 - 12.0.2000.8 (X64) 
当前权限：nt service\mssqlserver
开放端口：80、135、445、1433、2383（ssas）、3389......
进程名称：Ssms.exe、sqlwriter.exe、sqlservr.exe、MsMpEng.exe、chrome.exe......
```

有实战经验的老哥在看到 “此脚本包含恶意内容，已被你的防病毒软件阻止。” 提示时就知道是 Windows Defender 拦截的，我们拿目标机器上的进程列表到 “Windows 杀软在线对比辅助” 上对比了一下发现确实存在。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw07oSgf8qm8q3Z00l5p0RYDKahKv0ffPGmoVKuod5KbjkFrKc8yL2ibIg/640?wx_fmt=png)

****0x03 实战提权过程****

现在已经确定存在 Windows Defender，那么我们就先来绕过它获取一个 MSF 会话，经过测试发现 MSF 的 web_delivery 模块直接就可以绕过了，但是不能在 SSMS 和 Navicat Premium 中执行这个 Payload，会出现 “开头的 标识符 太长。最大长度为 128” 的报错。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0zWjGChPggntVwibuXbQ3PrHyCd3QkBB5II1PILfpgPe2pINQFkDvfWg/640?wx_fmt=png)

因为这是一个支持外链的 MSSQL，所以我们还可以换其他的 MSSQL 连接工具试一下，这里我直接用的 MSF 中的 mssql_exec 模块执行的 Payload，可以看到已经成功获取到目标机器会话。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0A7l3gmGGibjEvk5TYL1qcLcNUAQiczQVZFLtWicvRVn3oia6hdjGld2tKg/640?wx_fmt=png)

这里我们先尝试了利用 MS16-075 模块来进行权限提升，但是由于目标机器上的 Windows Defender 防病毒软件原因利用失败了，而且会话还掉了几次，如果有免杀 EXP 可以试一下。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0ibc0wh3uAtrnplXiby4flHZyeI9eJoBSCCGOCJJ4pG3LLjibSx6jV8tdw/640?wx_fmt=png)

MS16-075 模块利用失败了，但是我们在前期的信息搜集中有发现目标机器上还运行着一个 Ssms.exe 进程，这个进程是 MSSQL 客户端连接工具，全称为 SQL Server Management Studio。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0oRZQb0UEQbfQQibfltKYHsln06VMqkiaaU3zVMXN1ES376pdgdBoTQgQ/640?wx_fmt=png)

我们在[《站库分离常规渗透思路总结》](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484281&idx=1&sn=4d9fdae999907b222b0890fccb25bbcc&chksm=cfa6a76af8d12e7c366e0d9c4f256ec6ee6322d900d14732b6499e7df1c13435f14238a19b25&scene=21#wechat_redirect)这篇文章中也有提到过这个提权方法，当目标机器有在使用 Windows 身份验证连接 MSSQL 数据库时就会保留当前登录用户的令牌，所以可以直接利用 MSF 中的 Incognito 扩展来进行权限提升，可以看到已经成功提升至 Administrator 权限。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0jdcicleVP0MLjmKUtwlkGuawmibV9A2nd5Sc4AqDGo55sfzHP4T2OHAg/640?wx_fmt=png)

**注：**为什么拿到 Administrator 后不直接进入 cmdshell 添加管理员用户呢？因为动静太大了，在进行远程桌面连接时会产生大量日志，这样容易被管理员发现，所以个人建议不到万不得已时不要直接去添加管理员用户和连接远程桌面，尽可能的都在命令行下执行相关操作。

虽然已经得到目标机器的 Administrator 权限，但是 Windows Defender 还拦截了一些命令的执行，如：执行 getsystem 会一直卡着不动，所以目前无法得到 SYSTEM 以及 HASH 和明文。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0ju1PUMCUMlOTNZNrGoGRt4Dta7fPF0LiaJv6T3BOXlTV7ibB4D4J5YJg/640?wx_fmt=png)  

经过测试发现可以使用 migrate 命令将当前会话进程迁移至其他 SYSTEM 权限运行的进程中去，这里我迁移的是 PID 为 7008 的 GoogleCrashHandler.exe 进程，但还是不能使用 hashdump 命令抓取 hash，kiwi 也抓取不到明文密码，以前没有测试过 Windows 2019，不知道能不能通过修改 WDigest 注册表的方式来抓取明文密码。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0t9exoXnsBT84C9qGz7JGEZdmV0bNWYkNajzamScbraTwxgaVLiaicxAg/640?wx_fmt=png)

因为我们现在已经是 SYSTEM 最高权限了，它具备 SAM 注册表的完全控制权限，所以现在是可以直接利用 post/windows/gather/hashdump 模块抓取 HASH，但是解密不了。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcvrBTrB38aCZ4BNGMCoPw0dtYUGA0ThkledpyT8dJKcbnm0OD5cVxfLibSrosTNdX2IynbTuuBM4g/640?wx_fmt=png)

**思路小结：**

*   1. 利用 mssql_exec 模块执行 powershell payload 绕过 Windows Defender 获取 MSF 会话；
    
*   2. 利用 kiwi 扩展 “通过 MSSQL 客户端连接工具的 Ssms.exe 进程” 模拟 Administrator 令牌；
    
*   3. 利用 migrate 进程迁移命令绕过 Windows Defender 得到 SYSTEM 以及 HASH 和明文密码；
    

**0x04 其他绕过思路**

当目标机器存在 Windows Defender 防病毒软件时，即使已经拿到了 Administrator 会话后仍然无法执行 getsystem、hashdump、list_tokens 等命令和一些后渗透模块，除了上边已测试的 migrate 进程迁移方法外还可以尝试以下三个思路。尽可能的拿到目标机器的 SYSTEM 以及 HASH 和明文密码，在内网环境中可能会有其他用途，这里仅为大家扩展几个绕过思路，就不截图了！

#### (1) 直接添加管理员用户

使用 shell 命令进入 cmdshell 后直接利用 net 命令来添加一个管理员用户，然后远程桌面连接进去关闭 Windows Defender 防病毒软件的实时保护，最后尝试抓取目标机器 HASH 和明文密码。

```
net user test xxxasec!@#!23 /add
net localgroup administrators test /add
```

#### (2) 修改 SAM 注册表权限

使用 regini 命令修改 SAM 注册表权限，然后利用 post/windows/gather/hashdump 模块抓取目标机器 HASH，最后再利用 135/445 等支持哈希传递的工具来执行命令。

```
echo HKLM\SAM\SAM [1 17]>C:\ProgramData\sam.ini
regini C:\ProgramData\sam.ini
```

#### (3) 关闭杀毒软件实时保护

使用 Windows Defender 防病毒软件中自带的 MpCmdRun.exe 程序来关闭它的实时保护，然后再利用 hashdump 命令或模块抓取目标机器 HASH。MSF 中的 rollback_defender_signatures 模块也可以用来关闭实时保护，但是需要 SYSTEM 权限才能执行。

```
C:\PROGRA~1\WINDOW~1>MpCmdRun.exe -RemoveDefinitions -all
MpCmdRun.exe -RemoveDefinitions -all

Service Version: 4.18.1812.3
Engine Version: 1.1.17600.5
AntiSpyware Signature Version: 1.327.2026.0
AntiVirus Signature Version: 1.327.2026.0
NRI Engine Version: 1.1.17600.5
NRI Signature Version: 1.327.2026.0

Starting engine and signature rollback to none...
Done!
```

**0x05 注意事项**

记得前几年在测试 Windows Defender 时好像几乎所有获取 MSF 会话的方式都是会被拦截的，但是不知道为什么在这个案例中就没有拦截 web_delivery 模块中的 Powershell，hta_server 模块是会被拦截的，MSF 或 Windows Defender 版本原因吗？这里我也没有再去深究这个问题，所以大家在实战测试中还是得自己多去尝试，说不定哪种方法就成功了呢！！！

只需在公众号回复 “9527” 即可领取一套 HTB 靶场学习文档和视频，“1120” 领取安全参考等安全杂志 PDF 电子版，“1208” 领取一份常用高效爆破字典，还在等什么？

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfSyD5Wo2fTiaYRzt5iaWg1GJk2Cx54PBIoc0Ia3z1yIfeyfUV61mn3skB5bGP3QHicHudVjMEGhqH4A/640?wx_fmt=png)