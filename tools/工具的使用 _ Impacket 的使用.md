> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485314&idx=1&sn=67d022bf61471796288edcd03f5bc1ad&chksm=eaad87bfddda0ea935d38273c132afbb1451d560948645d1100bef79c1411cff28ba0db2cd2b&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFhLz1H5qAkgh9wkAnWtKQNJd5gpJXE7XFR5qAuM2JpmdfLVUoDkug3r0BJF0TiaMK5vyiaYCEzwqeag/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MzA3NzM2MjAzMg==&mid=2657228904&idx=1&sn=aa0d7a52864f19cbd6245a46ce162a1f&scene=21#wechat_redirect)

作者：谢公子

  

CSDN 安全博客专家，擅长渗透测试、Web 安全攻防、红蓝对抗。其自有公众号：谢公子学安全

Impacket

Impacket 是用于处理网络协议的 Python 类的集合，用于对 SMB1-3 或 IPv4 / IPv6 上的 TCP、UDP、ICMP、IGMP，ARP，IPv4，IPv6，SMB，MSRPC，NTLM，Kerberos，WMI，LDAP 等协议进行低级编程访问。数据包可以从头开始构建，也可以从原始数据中解析，而面向对象的 API 使处理协议的深层次结构变得简单。

**项目地址：**https://github.com/SecureAuthCorp/impacket

**关于工具的说明：**https://www.secureauth.com/labs/open-source-tools/impacket

![图片](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**Impacket 的安装**

```
git clone https://github.com/CoreSecurity/impacket.git
cd impacket/
python setup.py install
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicibicA4BuVuAJUr72f6rK9FvrZJymDJopDPyh5WJRaB5ibTEAQy37W7MWA/640?wx_fmt=png)

  

安装完成后，进入 examples 目录，查看有哪些脚本

```
cd impacket/examples
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicicV9DEOwLBWButVCk8oiavXDhnF0Xt3Nib6eQukTrd4fqoRh01CHDanuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**Impacket 中包含以下协议**  

*   以太网，Linux “Cooked” 数据包捕获  
    
*   IP，TCP，UDP，ICMP，IGMP，ARP
    
*   支持 IPv4 和 IPv6
    
*   NMB 和 SMB1，SMB2 和 SMB3（高级实现）
    
*   MSRPC 版本 5，通过不同的传输协议：TCP，SMB / TCP，SMB/NetBIOS 和 HTTP
    
*   使用 密码 / 哈希 / 票据 / 密钥 进行简单的 NTLM 和 Kerberos 身份验证
    
*   部分或完全实现以下 MSRPC 接口：EPM，DTYPES，LSAD，LSAT，NRPC，RRP，SAMR，SRVS，WKST，SCMR，BKRP，DHCPM，EVEN6，MGMT，SASEC，TSCH，DCOM，WMI
    
*   部分 TDS（MSSQL）和 LDAP 协议实现。
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

Impacket 中的脚本

**远程执行**

*   **psexec.py：**类似 psexec 的功能示例，使用 remcomsvc（https://github.com/kavika13/remcom）
    
*   **smbexec.py：**与使用 remcomsvc 的 psexec 类似的方法。这里描述了该技术。我们的实现更进一步，实例化本地 smbserver 以接收命令的输出。这在目标计算机没有可写共享可用的情况下很有用。
    
*   **atexec.py：**此示例通过 Task Scheduler 服务在目标计算机上执行命令，并返回已执行命令的输出。
    
*   **wmiexec.py：**通过 Windows Management Instrumentation 使用的半交互式 shell，它不需要在目标服务器上安装任何服务 / 代理，以管理员身份运行，非常隐蔽。
    
*   **dcomexec.py：**类似于 wmiexec.py 的半交互式 shell，但使用不同的 DCOM 端点。目前支持 MMC20.Application，ShellWindows 和 ShellBrowserWindow 对象。
    
*   **GetTGT.py：**指定密码，哈希或 aesKey，此脚本将请求 TGT 并将其保存为 ccache
    
*   **GetST.py：**指定 ccache 中的密码，哈希，aesKey 或 TGT，此脚本将请求服务票证并将其保存为 ccache。如果该帐户具有约束委派（具有协议转换）权限，您将能够使用 - impersonate 参数代表另一个用户请求该票证。
    
*   **GetPac.py：**此脚本将获得指定目标用户的 PAC（权限属性证书）结构，该结构仅具有正常的经过身份验证的用户凭据。它通过混合使用 [MS-SFU] 的 S4USelf + 用户到用户 Kerberos 身份验证组合来实现的。
    
*   **GetUserSPNs.py：**此示例将尝试查找和获取与普通用户帐户关联的服务主体名称。
    
*   **GetNPUsers.py：**此示例将尝试为那些设置了属性 “不需要 Kerberos 预身份验证” 的用户获取 TGT（UF_DONT_REQUIRE_PREAUTH). 输出与 JTR 兼容 
    
*   **ticketer.py：**此脚本将从头开始或基于模板（根据 KDC 的合法请求）创建金 / 银票据，允许您在 PAC_LOGON_INFO 结构中自定义设置的一些参数，特别是组、外接程序、持续时间等。 
    
*   **raiseChild.py：**此脚本通过（ab）使用 Golden Tickets 和 ExtraSids 的基础来实现子域到林权限的升级。
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**Windows Secrets**  

*   **secretsdump.py：**执行各种技术从远程机器转储 Secrets，而不在那里执行任何代理。对于 SAM 和 LSA Secrets（包括缓存的凭据），然后将 hives 保存在目标系统（％SYSTEMROOT％\ Temp 目录）中，并从中读取其余数据。对于 DIT 文件，我们使用 dl_drsgetncchanges（）方法转储 NTLM 哈希值、纯文本凭据（如果可用）和 Kerberos 密钥。它还可以通过使用 smbexec/wmiexec 方法执行的 vssadmin 来转储 NTDS.dit. 如果脚本不可用，脚本将启动其运行所需的服务（例如，远程注册表，即使它已被禁用）。运行完成后，将恢复到原始状态。
    
*   **mimikatz.py：**用于控制 @gentilkiwi 开发的远程 mimikatz RPC 服务器的迷你 shell
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**服务器工具 / MiTM 攻击**

*   **ntlmrelayx.py：**此脚本执行 NTLM 中继攻击，设置 SMB 和 HTTP 服务器并将凭据中继到许多不同的协议（SMB，HTTP，MSSQL，LDAP，IMAP，POP3 等）。该脚本可以与预定义的攻击一起使用，这些攻击可以在中继连接时触发（例如，通过 LDAP 创建用户），也可以在 SOCKS 模式下执行。在此模式下，对于每个中继的连接，稍后可以通过 SOCKS 代理多次使用它  
    
*   **karmaSMB.py：**无论指定的 SMB 共享和路径名如何，都会响应特定文件内容的 SMB 服务器
    
*   **smbserver.py：**SMB 服务器的 Python 实现，允许快速设置共享和用户帐户。
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**WMI**  

*   **wmiquery.py：**它允许发出 WQL 查询并在目标系统上获取 WMI 对象的描述（例如，从 win32_account 中选择名称）
    
*   **wmipersist.py：**此脚本创建、删除 WMI 事件使用者、筛选器，并在两者之间建立链接，以基于指定的 wql 筛选器或计时器执行 Visual Basic
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**已知的漏洞**  

*   **goldenPac.py：** 利用 MS14-068。保存 Golden Ticket 并在目标位置启动 PSExec 会话
    
*   **sambaPipe.py：**该脚本将利用 CVE-2017-7494，通过 - so 参数上传和执行用户指定的共享库。 
    
*   **smbrelayx.py：**利用 SMB 中继攻击漏洞 CVE-2015-0005。如果目标系统正在执行签名并且提供了计算机帐户，则模块将尝试通过 NETLOGON 收集 SMB 会话密钥。 利用 SMB 中继攻击漏洞 CVE-2015-0005
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**SMB / MSRPC**  

*   **smbclient.py：**一个通用的 SMB 客户端，可以允许您列出共享和文件名，重命名，上传和下载文件，以及创建和删除目录，所有这些都是使用用户名和密码或用户名和哈希组合。这是一个很好的例子，可以了解到如何在实际中使用 impacket.smb
    
*   **getArch.py：**此脚本将与目标主机连接，并使用文档化的 msrpc 功能收集由（ab）安装的操作系统体系结构类型。 
    
*   **rpcdump.py：**此脚本将转储目标上注册的 RPC 端点和字符串绑定列表。它还将尝试将它们与已知端点列表进行匹配。
    
*   **ifmap.py：**此脚本将绑定到目标的管理接口，以获取接口 ID 列表。它将在另一个界面 UUID 列表上使用这个列表，尝试绑定到每个接口并报告接口是否已列出或正在侦听
    
*   **opdump.py：**这将绑定到给定的 hostname:port 和 msrpc 接口。然后，它尝试依次调用前 256 个操作号中的每一个，并报告每个调用的结果。 
    
*   **samrdump.py：**从 MSRPC 套件与安全帐户管理器远程接口通信的应用程序中。它列出了通过此服务导出的系统用户帐户、可用资源共享和其他敏感信息  
    
*   **services.py：**此脚本可用于通过 [MS-SCMR] MSRPC 接口操作 Windows 服务。它支持启动，停止，删除，状态，配置，列表，创建和更改。 
    
*   **netview.py：**获取在远程主机上打开的会话列表，并跟踪这些会话在找到的主机上循环，并跟踪从远程服务器登录 / 退出的用户 
    
*   **reg.py：**通过 [ms-rrp]msrpc 接口远程注册表操作工具。其想法是提供与 reg.exe Windows 实用程序类似的功能。=
    
*   **lookupsid.py：**通过 [MS-LSAT] MSRPC 接口的 Windows SID 暴力破解程序示例，旨在查找远程用户和组
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**MSSQL / TDS**  

*   **mssqlinstance.py：**从目标主机中检索 MSSQL 实例名称。 
    
*   **mssqlclient.py：**MSSQL 客户端，支持 SQL 和 Windows 身份验证（哈希）。它还支持 TLS。
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**文件格式**  

*   **esentutl.py：**Extensibe 存储引擎格式实现。它允许转储 ESE 数据库的目录，页面和表（例如 NTDS.dit） 
    
*   **ntfs-read.py：**NTFS 格式实现。此脚本提供了一个用于浏览和提取 NTFS 卷的功能小的反弹 shell，包括隐藏 / 锁定的内容
    
*   **registry-read.py：**Windows 注册表文件格式实现。它允许解析脱机注册表配置单元
    

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFjWh40vdydJOCqT0rKf86hicjolibfVWO3TkzHYX4ia7llfGuagSMVWX2CPs5mN0Vg2jH6Uknbr3sI9Q/640?wx_fmt=gif)

**其他**  

*   **GetADUsers.py：**此脚本将收集有关域用户及其相应电子邮件地址的数据。它还将包括有关上次登录和上次密码设置属性的一些额外信息。  
    
*   **mqtt_check.py：**简单的 MQTT 示例，旨在使用不同的登录选项。可以很容易地转换成帐户 / 密码暴力工具。 
    
*   **rdp_check.py：**[MS-RDPBCGR]和 [MS-CREDSSP] 部分实现只是为了达到 CredSSP 身份验证。此示例测试帐户在目标主机上是否有效。
    
*   **sniff.py：**简单的数据包嗅探器，使用 pcapy 库来监听在指定接口上传输的包。
    
*   **sniffer.py：**简单的数据包嗅探器，它使用原始套接字来侦听与指定协议相对应的传输中的数据包。
    
*   **ping.py：**简单的 ICMP ping，它使用 ICMP echo 和 echo-reply 数据包来检查主机的状态。如果远程主机已启动，则应使用 echo-reply 数据包响应 echo 探针。 
    
*   **ping6.py：**简单的 IPv6 ICMP ping，它使用 ICMP echo 和 echo-reply 数据包来检查主机的状态。
    

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFjWI9QibTmpF13L33cHIh2bSMLAI4tW7sTgTkzh4lRcZ6JR7SrOibCTYUEsg8ZsmyKnUBm7h4J5klZw/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MzA3NzM2MjAzMg==&mid=2657228904&idx=1&sn=aa0d7a52864f19cbd6245a46ce162a1f&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFjGibCQezQKY4NzE1WGn6FBCbq3pQVl0oONnYXT354mlVw0edib6X6flYib9JRTic4DTibgib15WZC7sDUA/640?wx_fmt=png)

[内网渗透（十三） | WinRM 远程管理工具的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247492427&idx=1&sn=af3a862d78184e93b6e9377f12bce354&chksm=fc7be796cb0c6e80a057dff2a7d67e3483c33e8da2d3a7acb84d04fdd997f28cb89f1fd617fd&scene=21#wechat_redirect)  

[内网渗透（十二） | 利用委派打造隐蔽后门 (权限维持)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247491363&idx=1&sn=e5d6670b0f76299d92110d7b679ad70b&chksm=fc781bfecb0f92e8aacaa6f4f7788ed48577e25f943d92073b1b26e68bfbc8f505b2dd2fa4d8&scene=21#wechat_redirect)  

[内网渗透（十一） | 哈希传递攻击 (Pass-the-Hash,PtH)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490908&idx=1&sn=97594fbbef40346d07b5a6e5185ce77e&chksm=fc781981cb0f9097d18f4b32ff39f59b3512cedd35f0810ad5f61b661e631153f8c4e157d875&scene=21#wechat_redirect)  

[技术干货 | 工具：Social engineering tookit 钓鱼网站](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490513&idx=2&sn=10afb29a20f37df05ebb12ea4d540e1f&chksm=fc781f0ccb0f961a85e646dd54e977dbcaeb5569be6701db4c29b9e204d964bab3ded6bf1999&scene=21#wechat_redirect)

[技术干货 | 工具的使用：CobaltStrike 上线 Linux 主机 (CrossC2)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490608&idx=1&sn=f2b2ea93b109447aa8cc2c872aa87c52&chksm=fc7818edcb0f91fbf85fa53f71e9967fc29fc93f6a783eed154707ca2dec24ca7f419fde5705&scene=21#wechat_redirect)

[内网渗透（十） | 票据传递攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490376&idx=2&sn=c070dd4c761b49d3fabd573cc9c96b5a&chksm=fc781f95cb0f9683b0f6c64f5db5823973c1b10e87b1452192bbed6c1159eccf6e8f2fd0290b&scene=21#wechat_redirect)  

[内网渗透（九） | Windows 域的管理](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490197&idx=1&sn=4682065ddcab00b584918bc267e33f53&chksm=fc781e48cb0f975eddc44d77698fbb466d0eac7d745a6e5bbaf131560b3d4f9e22c1a359d241&scene=21#wechat_redirect)  

[内网渗透（八） | 内网转发工具的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490042&idx=1&sn=136d4057044a7d6f6cb5b57d20f7954a&chksm=fc781d27cb0f9431ec590662ab4e6bcd31b303e7caa20a2b116fd9a9b97e9e3be0bc34408490&scene=21#wechat_redirect)  

[内网渗透 | 域内用户枚举和密码喷洒攻击 (Password Spraying)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489985&idx=1&sn=0b7bce093e501b9817f263c24e0ed5b8&chksm=fc781d1ccb0f940aad0c9b2b06b68c7a58b0b4c513fe45f7da6e6438cac76d4778e61122faf8&scene=21#wechat_redirect)  

[内网渗透（七） | 内网转发及隐蔽隧道：网络层隧道技术之 ICMP 隧道 (pingTunnel/IcmpTunnel)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489736&idx=2&sn=0cb551ee520860878c2c33108033c00c&chksm=fc781c15cb0f9503f672aa0bd18cb13fef4c60124ba5978ab947c34272b2d8a28c584a99219d&scene=21#wechat_redirect)  

[内网渗透（六） | 工作组和域的区别](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489205&idx=1&sn=24f9a2e0e6b92a167f3082bb6e09c734&chksm=fc781268cb0f9b7e3c11d19a9fb41567124055eb0e8dd526cbbaf1e9393ff707f9fa9d10c32b&scene=21#wechat_redirect)  

[内网渗透（五） | AS-REP Roasting 攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489128&idx=1&sn=dac676323e81307e18dd7f6c8998bde7&chksm=fc7812b5cb0f9ba3a63c447468b7e1bdf3250ed0a6217b07a22819c816a8da1fdf16c164fce2&scene=21#wechat_redirect)

[内网渗透 | 内网穿透工具 FRP 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489057&idx=3&sn=f81ef113f1f136c2289c8bca24c5deb1&chksm=fc7812fccb0f9beaa65e5e9cf40cf9797d207627ae30cb8c7d42d8c12a2cb0765700860dab84&scene=21#wechat_redirect)  

[内网渗透（四） | 域渗透之 Kerberoast 攻击_Python](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488972&idx=1&sn=87a6d987de72a03a2710f162170cd3a0&chksm=fc781111cb0f98070f74377f8348c529699a5eea8497fd40d254cf37a1f54f96632da6a96d83&scene=21#wechat_redirect)  

[内网渗透（三） | 域渗透之 SPN 服务主体名称](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)  

[内网渗透（二） | MSF 和 CobaltStrike 联动](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[内网渗透 | 域内认证之 Kerberos 协议详解](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)  

[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)