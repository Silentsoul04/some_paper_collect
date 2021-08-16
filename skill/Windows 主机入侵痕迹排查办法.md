> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hBdQstyTUKC-0RsGyvklww)

**一、**排查思路
==========

在攻防演练保障期间，一线工程师在实施主机入侵痕迹排查服务时可能面临时间紧、任务急、需要排查的主机数量众多情况。为了确保实施人员在有限的时间范围内，可以高效且保证质量的前提下完成主机入侵痕迹排查工作，本人总结了自己的一些经验，下面的内容特此分享主机入侵痕迹排查服务中重点、关键的排查项，仅作为参考使用。

### **1.1** 初步筛选排查资产

一般情况下，客户资产都比较多，想要对所有的资产主机进行入侵痕迹排查基本不太现实，等你全部都排查完了，攻击者该做的事早就做完了，想要的目的也早就达到了。那么针对客户资产量大的情况，我们应该怎么处理？

首先，在排查前，作为项目经理，应该与客户沟通好，取得授权，确认排查范围和排查方案和办法，客户若是没有授意或者同意，那么下面的操作都是违规操作，甚至有的还违法。

取得客户同意后，我们再从资产面临的风险等级、资产的重要程度、攻击者的攻击思路、手法及目标选择倾向几个方面去初步筛选出排查资产。这里建议从以下资产范围选取：

①曾失陷资产：在以前的红蓝对抗、攻防演练、或者真实的黑客攻击事件中被攻陷的主机，曾失陷资产应作为排查的重点对象。

②互联网暴露脆弱资产：从互联网暴露资产中筛选出使用了高危漏洞频发的组件 / 应用（组件如 Weblogic、JBoss、Fastjson、Shiro、Struts2 等）。还有一个点需要注意，就是客户是否具有有效的资产管理，是否能够清晰明确识别出哪些资产用了什么组件，如果不能的话，只能通过之前的渗透测试结果来筛选出脆弱资产。

③关键资产：如域控等可以导致大量主机失陷的集权类资产。

### **1.2 确定排查资产**

主机入侵痕迹排查工作建议在一周内对数量控制在 20 台以内的主机进行排查。经过初步筛选的资产数量如果远远大于 20 台主机，需要从资产里面进行二次筛选，如果存在曾失陷资产，排查主机范围可以定为曾失陷资产；如果不存在曾失陷资产，排查主机范围可以定为脆弱资产，具体可以根据客户自身实际情况调整。

需要注意是，如果排查资产中包含曾失陷资产的话，需要向客户索要历史攻防演练 / 应急等报告，在排查时需结合历史报告和指导手册内容一起进行排查，需要特别留意历史报告中攻击者的入侵痕迹是否已经完全清理。

### **1.3 入侵痕迹排查**

在实际情况下，攻击者在进行攻击时使用的攻击手法、攻击思路、行为等各有差异，无论是考虑实现成本还是效率问题，都难以通过很精细很全面的排查项去实施主机入侵痕迹排查，但是我们可以从攻击中可能会产生的一些比较共性的行为特征、关键的项进行排查。

对于主机的入侵痕迹排查，主要从网络连接、进程信息、后门账号、计划任务、登录日志、自启动项、文件等方面进行排查。比如，如果存在存活后门，主机可能会向 C2 发起网络连接，因此可以从网络连接排查入手，如果存在异常的网络连接，则必然说明存在恶意的进程正在运行，则可以通过网络连接定位到对应进程，再根据进程定位到恶意文件。如果攻击者企图维持主机控制权限的话，则可能会通过添加后门账号、修改自启动项，或者添加计划任务等方式来维持权限，对应的我们可以通过排查账号、自启动项、计划任务来发现相应的入侵痕迹。

**二、**排查内容
==========

### **2.1windows 主机**

攻击者一般使用 attrib <程序> +s +h 命令隐藏恶意程序，故在排查痕迹前需打开 “工具—文件夹选项—查看”。按照下图中的设置，即可显示所有文件。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERsexTicYFWAk28ibFZPeLlIzXghZyGDbhTGiczYCeOaBaaEZQbfyydVnOFpw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseugdEhSwfJhvblXWUwHG7czjVQKf65elK9eKNRpb4eud0ydbuyQrJ0A/640?wx_fmt=jpeg)

**2.1.1 网络连接**

排查步骤:

在 CMD 中执行 netstat -ano 查看目前的网络连接。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERsejsbianmAiaVicYfw3iciccrNbddXTJzg30kpZPovBpxCWUZc3KZ8F39rQcQ/640?wx_fmt=jpeg)

这种情况一般都比较正常，只有 80 和 443 端口，一般都是正常业务开放端口。

**分析方法：**

如果网络连接出现以下情况，则当前主机可能已经失陷：

1、主机存在对内网网段大量主机的某些端口（常见如 22，445，3389，6379 等端口）或者全端口发起网络连接尝试，这种情况一般是当前主机被攻击者当作跳板机对内网实施端口扫描或者口令暴力破解等攻击。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseyPCkQe1Woiaq1yp0V3j0RgnpQnI2KXd3WIia9Ur4TgmL930NjAu76Myg/640?wx_fmt=jpeg)

2、主机和外网 IP 已经建立连接（ESTABLISHED 状态）或者尝试建立连接（SYN_SENT 状态），可以先查询 IP 所属地，如果 IP 为国外 IP 或者归属各种云厂商，则需要重点关注。进一步可以通过威胁情报（https://x.threatbook.cn / 等）查询 IP 是否已经被标注为恶意 IP。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERset9IFuMzyPx7h2Nt54AT9ibQsHuGgNLRojtKDzAkYGDtTOicavELPcWSg/640?wx_fmt=jpeg)

3、如果无法直接从网络连接情况判断是否为异常连接，可以根据网络连接找到对应的进程 ID，判断进程是否异常。如果不能从进程判断，可以进一步找到进程对应文件，将对应文件上传至 virustotal（https://www.virustotal.com）进行检测。如上面截图中对内网扫描的进程 ID 是 2144，在任务管理器中发现对应的文件是 svchost.exe。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseZDTr7A0fdUibJaEJC4pmgovtaNEVqzNBoic6Lgeaaic7yLduUBQvH7tkA/640?wx_fmt=jpeg)

上传至 virustotal 检测的结果为恶意文件。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseSwt9wvUYSjibia4l9tP38TleEqZFuAxfVk3icYHTmFjy3wGL56icfCKw8A/640?wx_fmt=jpeg)

若在排查网络连接中，任务管理器只能看到有命令行工具（如 powershell、cmd）powershell 进程与外联 IP 建立会话，无法看到进程对应的运行参数。此时可借助 Process Explorer 进一步观察 powershell 的运行参数。如下在 Process Explorer 中发现 powershell 执行了 cobalt strike 脚本的痕迹。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseAyIkw2ia8rwb7SlKTgbBJ8R8wfJGw0NvibgPtaDj90d5Bcz6W8pn7AwQ/640?wx_fmt=jpeg)

**2.1.2 敏感目录**

**排查步骤：**

查看攻击方常喜欢上传的目录是否有可疑文件。

**分析方法：**

1、各个盘符下的临时目录，如 C:\TEMP、C:\Windows\Temp 等。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseMa1VqWmOUyuJN2ibmqygc9JzU3VKJgib1Oe1UJ2v7xSxcSG8n2jqN9iag/640?wx_fmt=jpeg)

2、%APPDATA%，在文件夹窗口地址栏输入 %APPDATA%，回车即可打开当前用户的 appdata 目录。![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseCP0nEWhXklTwadgf21pGjibCv39dGa5fMXnuRI43vpJUwzicNnPjcFIA/640?wx_fmt=jpeg)

如 Administrator 用户对应的 %APPDATA% 目录 C:\Users\Administrator\AppData\Roaming。可以按照修改日期排序筛选出比较临近时间有变更的文件。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERse5IoJvHYHE0DIhftCvOm8D4eg9iaw0WWftMiaq8G3eHLFDHEl58FHaIkQ/640?wx_fmt=jpeg)

3、浏览器的下载目录

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseS9c0ibfqwc5Y1Zoeyk3huhQicLR1ocAWBkVypRCodkBewGJCwd6ica2Qw/640?wx_fmt=jpeg)

4、用户最近文件 %UserProfile%\Recent，如 Administrator 对应的目录为 C:\Users\Administrator\Recent

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseoVI5Zw9TycLGUvmlyBWfLwpC7Dg5HwWswd54xAY15kJOMteRPnr73A/640?wx_fmt=jpeg)

5、回收站，如 C 盘下回收站 C:$Recycle.Bin

对于脚本文件可直接查看内容判定是否为恶意，若是遇到 exe 可执行文件，可将对应文件上传至 virustotal（https://www.virustotal.com）进行检测。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERse15lZSc7V5XZK49NoYGPUTVjV7vA3heW1kHYc6ib1ia29PxPapWiausNXg/640?wx_fmt=jpeg)

**2.1.3 后门文件**

**排查步骤：**

> 查看粘滞键 exe；
> 
> 查看注册表中映像的键值。

**分析方法：**

1、查看粘滞键 exe

查看 C:\Windows\System32 \ 下的 sethc.exe 文件的创建、修改时间是否正常，如下图，一般情况下，系统文件的创建时间与修改时间应相同，sethc 的创建时间与修改时间不同，可确定 sethc 已被替换成后门文件。由于攻击者可修改文件时间，上述简单粗暴的判断方式可能不靠谱，可将 sethc 拷贝出来、上传至 VT 检测危害。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseASNt4yOsSNK7WXZQMzoroqXSWO05hs922CThYKMXotxpVIQhstocoA/640?wx_fmt=jpeg)

2、查看注册表中映像的键值

检查注册表 “HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options” 下所有 exe 项中是否有 debugger 键，若有 debugger 键，将其键值对应的程序上传至 VT 检测。如下图，攻击者利用该映像劫持的攻击者方式，在 sethc.exe 项中新建 debugger 键值指向 artifact.exe，攻击效果为当连续按 5 下 shift 键后，不会执行 sethc.exe，而是转而执行劫持后的 artifact.exe 文件。于是在排查中发现有 debugger 键值，均可认为指定的文件为后门文件，待上传 VT 后确认其危害。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERse1xB0DrjUWlpss2FkE0XmBP93WAENOw8MmurJQyTRE3unicfaT4Bpvnw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERsevdDZFoRlorI0AwTSzIhHueQs72hFydo0acLkkWiatTJlYUEDv5zzpBg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseuQM0HVz6CfLg3CUSxFd30UzeUqhhGZVFQPFnXPCQVIaJo3YlDrjUsg/640?wx_fmt=jpeg)

这里没有 debugger 键，下面的图是有的:

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERsebw9HDpgCoaoJhoVNjiaruM89YvfWaMWXoudX6ULmB55o32MocFbuRSA/640?wx_fmt=jpeg)

**2.1.4 后门账号**

**排查步骤：**

> 打开 regedit 查看注册表中的账号；
> 
> 查看 administrators 组中是否存在赋权异常的账号。

**分析方法：**

查看注册表中 HKLM\SAM\SAM\Domains\Account\Users\Names 中是否有多余的账号（可询问客户运维人员以确定账号存在的必要性）。正常情况下，上述路径的 SAM 权限仅 system 用户可查看，需要给 administrator 用户授权才能打开完整路径。对 SAM 右键、给 administator 用户添加完全控制权限（下图的权限操作方法适用于 win7 及以上操作系统）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERse8ibB5NLMEpl4dZ8woDNrcwHP3NwDKMglmgjqEicN2V6pqtCo4iajVwKzQ/640?wx_fmt=jpeg)

win2003、XP 等低版本系统的操作方法请使用下图的流程给 administrators 组添加权限。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERse4kXmRZh0GfHLdb9rPibFmXwQkPdffa6NOKPvXUctibfevvnDdZVrb7ug/640?wx_fmt=jpeg)

带有 $ 符号的账号特指隐藏账号（如 aaaa$），正常业务中不需要创建隐藏账号，可判断带有 $ 符号的均为后门账号。然后在客户运维的协助下排查其他的异常账号。

如下图中，除了 aaaa$ 可直接判断外，root 账号为高度关注对象。（注：aaaa$ 中的键值 0x3ea 表示该账号与 Users 表中相应数值的表相对应，在删除账号时需一起删除）

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseAticTiaynRSQZMepMFUoy5s0iaJ44fxTiaYWTJDF0bqtAgekCqL7EkE1dA/640?wx_fmt=jpeg)

**注：**异常账号删除后需要将之前授权的 administrator 移除 SAM 权限。

查看 administrators 组中是否存在赋权异常的账号。比如正常情况下 guest 用户处于禁用状态、普通应用账户 (weblogic、apache、mysql) 不需要在 administrators 组中。如下图，执行命令 net user guest 查看 guest 账号的信息，如果 guest 账号被启用，且在管理员组成员中有 guest 用户，需要询问客户运维人员该 guest 账户启用的必要性以及加入管理组是否有必要，否则可认为攻击者将系统自带用户 guest 启用并提权至管理员组后作为后门账号使用。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERseTUiab7qtewqbhMkdLcY0ichZQk1x1fU6RUpWIiahibfsic33lzJTTSGyHTg/640?wx_fmt=jpeg)

执行 net localgroup Administrators 关注管理员组别是否存在异常账号：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERsen1GEUEKT75FlibhviafwYMnCgqKbIGEwIHltfMDcsIicGPyH7mgib8MZsA/640?wx_fmt=jpeg)

**2.1.5 自启动项**

**排查步骤：**

> 使用 Autoruns 工具查看自启动项
> 
> 查看组策略中的脚本
> 
> 查看注册表中的脚本、程序等
> 
> 查看各账号自启目录下的脚本、程序等
> 
> 查看 Windows 服务中的可执行文件路径

**分析方法：**

1、使用 Autoruns：

使用工具能较全面地查看系统中的自启动项。在得到客户授权，能够在可能失陷的主机上传排查工具时，可使用 Autoruns 工具进行详细的自启动项排查。排查中主要关注粉色条目，建议与客户运维人员一同查看，以及时排除业务所需的正常自启项。如下图，在 Everything 栏中，查看粉色的条目中发现常见的 sethc 被劫持为 cmd，Command Processor 键值（默认为空）关联到名为 windowsupdate.exe（效果为启动 cmd 时，被关联的程序会静默运行）。sethc 的劫持可确认为入侵痕迹，Command Processor 键值的关联程序需要找客户进一步确认是否业务所需，或将 windowsupdate.exe 上传 VT 检测。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERsebuaKWWjX8bvsXDkl3PfGU82lMkhgSiavNnoSnuYzhVH2icQpJCjcdqRw/640?wx_fmt=jpeg)

另外，这个工具很好用 ，特别小，可以直接上传文件到 VT 进行检测。

2、查看组策略：

在无法使用工具、只能手工排查的情况下，可查看常见的自启项是否有异常文件。打开 gpedit.msc—计算机配置 / 用户配置—Windows 设置—脚本，在此处可设置服务器启动 / 关机或者用户登录 / 注销时执行的脚本。下图 1、2 两处的脚本均需要查看是否添加有脚本。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ictApuppQSdibzVdic8C7ERse1oGd9l21X62Gm0I0x0SA5A0uHKe1RbibsYvx4UxLrnIXxA3W96auMjw/640?wx_fmt=jpeg)

我这里没有脚本。

**2.1.6 日志**

工程师基本都会看日志，windows 日志也就那些内容，比较简单，我就不细述，主要写一下几个比较重要的点，基本上就可以排查出是否有异常登录了。

**排查步骤：**

> 查看登录日志中暴力破解痕迹；
> 
> 查看账号管理日志中账号的新增、修改痕迹；
> 
> 查看远程桌面登录日志中的登录痕迹。

三、总结一下
------

技能都是需要动手的，然后思路需要清晰，操作要细致，跟客户要保持沟通，切记不能闷着擅自操作！

作者：白衣不再少年，文章来源：FreeBuf

****扫描关注乌雲安全****  

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavz34wLFhdnrWgsQZPkEyKged4nfofK5RI5s6ibiaho43F432YZT9cU9e79aOCgoNStjmiaL7p29S5wdg/640?wx_fmt=jpeg)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**