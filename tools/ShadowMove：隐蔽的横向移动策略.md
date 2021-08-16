\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzA5ODA0NDE2MA==&mid=2649732789&idx=1&sn=16cde6178e78a4b92c623f449b29efc8&chksm=888c86dabffb0fccf64fa8161063a1caa006963531c00cc407d51aaf3a4001025996b40c33f6&mpshare=1&scene=1&srcid=1013jmFT1I6VaVpGoal8inJh&sharer\_sharetime=1602550458988&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=8c9049d0f83009fedb4e63529b4d5d66eda698dfcfbdbcc6be3f33a101a6bd435dcd63a088b856dd1116c2338ccacf724f90ee83ecb4c2745256404f18e55721ca55444a1678291fa3674b338439568d18621957f76f1ce401a8da00af68135f4ee817631d6e8b3c1c0f69db27f9fd0606e8273e9993e7933ec7a55a36f54fde&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=AVvzFI7Olr9eJqEqHUS0QiU%3D&pass\_ticket=2G6SwO4uyYCX4aTiQDJvW1D1IrAJXn1CnpH%2BbX1rykSOMZNKPaotYwa2vyHnTBud&wx\_header=0)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAvVicpVmcCgwIZTXOWpSjqhVPgJDSP7MNSzHMr1KKpncgXktyOHHITew/640?wx_fmt=jpeg)

高级持续威胁（APT）攻击使用各种策略和技术在进入奖励环境中横向移动。但是，现有的策略和技术具有局限性，例如需要更高的权限，创建新的连接，执行新的身份验证或需要进行过程注入。基于这些特征，已经提出了许多基于主机和基于网络的解决方案来防止或检测这种横向移动尝试。在本文中提出了一种新颖的隐蔽横向移动策略 ShadowMove，其中仅将企业网络中系统之间已建立的连接误用于横向移动。它具有一组独特的功能，例如不需要提升的特权，不需要新的连接，不需要额外的身份验证以及不需要进行进程注入，这使它绕过了最新的检测机制。ShadowMove 通过新颖的套接字复制方法启用，该方法允许恶意进程以静默方式滥用良性进程建立的 TCP 连接。

本文为当前的 Windows 和 Linux 操作系统设计和实现 ShadowMove，为了验证 ShadowMove 的可行性，构建了多个原型，这些原型成功劫持了三种企业协议 FTP，Microsoft SQL 和 Window Remote Management，以执行横向移动操作，例如将恶意软件复制到下一台目标计算机并在目标计算机上启动恶意软件。还确认现有的基于主机和网络的解决方案无法检测到本研究的原型，例如五种顶级防病毒产品（McAfee，Norton，Webroot，Bitdefender 和 Windows Defender），四个 IDS（Snort，OSSEC，Osquery 和 Wazuh）以及两个终端检测响应系统（CrowdStrike Falcon Prevent 和 Cisco AMP）。

  

  

0x01 Introduction

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHASCSibIbfWkiacuFrsaibGwAcxjZXRnDLEJzfeIicVheR4ibgINaSRLciaVrg/640?wx_fmt=png)

高级持久威胁（APT）是复杂的精心计划，并且是针对诸如政府机构或大型企业等知名目标的多步骤网络攻击。这种攻击是由资源丰富的知识渊博的攻击者（如 Lazarus 或 APT38）进行的，每年给公司和政府机构造成数十亿美元的财务损失。

APT 攻击者通常使用鱼叉攻击或水坑攻击在目标网络中找到立足点。一旦进入目标网络，他们会谨慎地使用承诺的系统作为进入其他系统的跳板，直到他们能够访问关键系统（例如包含机密文件的文件服务器）后，这些关键系统都被藏在网络内部。朝向关键系统的这种移动称为横向移动（lateral movement）。

横向移动可以通过多种方式实现。攻击者可以利用 SMB 或 RDP 等网络服务中的漏洞在网络中横向移动。但是，由于防御机制的进步，找到此类漏洞并成功将其利用而不被发现就变得越来越困难。或者，攻击者可以从受感染的系统中获取用户凭据，然后重复使用这些凭据来执行横向移动（例如，凭据转储 credential dumping，哈希传递 pass-the-hash 或票证传递 pass-the-ticket）。但是，这种方法需要创建新的网络连接，因此如果新连接偏离合法系统之间的正常通信模式，则可以通过网络级防御进行检测。攻击者可以使用另一种方法，利用劫持攻击来修改合法客户端，以便将其连接重新用于横向移动（例如，通过修补 SSH 客户端以与 SSH 服务器通信而不知道密码）。但是，此类攻击是特定于应用程序和协议的，需要进行过程注入。由于现有的基于主机的防御解决方案（例如 Windows Defender ATP）认识到各种过程注入技术，因此它们难以实施且易于检测。

在本文中提出了一种新颖的横向移动策略，称为 ShadowMove，它使 APT 攻击者可以在企业网络中的系统之间平稳移动，而不会被现有的主机级和网络级防御机制所发现。攻击者希望避免在操作过程中利用远程服务中的漏洞，以减少被入侵检测系统（IDS）暴露的机会。在这种攻击情况下，攻击者被动地观察受感染系统的通信动态，以逐步构建其在目标网络中正常行为的模型，并利用该模型选择下一个受害者系统。此外，为了使攻击更加隐秘，攻击者将自己限制为仅重用已建立的连接。

WinRM（Windows 远程管理）和 FTP 等许多应用程序协议允许用户在远程服务器上执行某些操作。攻击者将自己的命令注入此类协议的命令流中，以实现其目标。例如，攻击者可以通过在建立的 WinRM 会话中注入命令来远程执行程序），或者可以通过在建立的 FTP 连接中注入 FTP 命令来检查远程系统上的文件系统。

ShadowMove 在良性客户端过程中不使用任何代码来注入伪造的命令。取而代之的是，它采用了一种新颖的技术来秘密复制合法客户端所拥有的套接字，并通过这种被盗套接字注入命令。这样，将不会创建新的连接，也不会执行新的身份验证，因为在已建立的会话的上下文中解释了所注入的命令；这意味着攻击者无需通过任何身份验证。

在本文中，展示了攻击者如何在典型的企业网络上实施这种攻击。为此开发了一个原型系统，该系统可以劫持在同一客户端下运行的 FTP 客户端，Microsoft SQL 客户端和 WinRM 客户端建立的现有 TCP 连接。用户帐户作为本研究的原型，没有任何提升的特权。还提供了一个基于 Prolog 的计划程序，攻击者可以利用该计划程序通过劫持可用的连接来系统地计划横向移动。这样，攻击者可以比现有攻击方案更秘密地访问关键系统。本文讨论了有关攻击者如何注入其数据包的技术挑战，这些数据包符合在已建立的 TCP 连接上运行的协议，并且在连接的另一端服务器可以接受。

  

  

0x02 ShadowMove

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHASCSibIbfWkiacuFrsaibGwAcxjZXRnDLEJzfeIicVheR4ibgINaSRLciaVrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAGGN2PBl1D1tDHMOBUjdQ98SRJYt9gfzIgOQFyR09OHLpS8uJobIPFw/640?wx_fmt=png)

ShadowMove 的基本思想是重用已建立的合法连接，以在受感染的网络内横向移动。如上图所示，ShadowMove 的工作分为三个主要步骤：首先，它默默地复制合法客户端应用程序用来与服务器应用程序通信的套接字。其次，它使用复制的套接字在客户端和服务器之间的现有 TCP 会话中注入数据包。第三，服务器处理注入的数据包，并无意中保存和 / 或启动 ShadowMove 的新实例。这些步骤的结果是，攻击者会从客户端计算机秘密移动到服务器计算机。  

由于 ShadowMove 会限制自己重用已建立的连接到相邻系统，因此它可以确保入侵检测系统对意外连接发出警报，而不会检测到它的运行。此外，通过这样做，攻击可以绕过建立新连接所需的身份验证阶段。从主机安全角度和网络安全角度来看，ShadowMove 攻击都是值得注意的：在主机级别，ShadowMove 滥用受害进程拥有的资源（即已建立并经过身份验证的网络连接）；另一方面，由于 ShadowMove 滥用的是套接字，通过将恶意网络流量与良性网络流量混合，其攻击行为扩展到了网络级别。

### **1）ShadowMove 利用的基本弱点**

ShadowMove 攻击是现有计算机环境中的两个基本弱点。在诸如 GNU Linux 和 Microsoft Windows 之类的商用操作系统中，这两个相互矛盾但必不可少的要求（即进程隔离和资源共享）的第一个漏洞。下一个弱点是由于许多现有的网络协议缺少适当的内置消息源完整性验证机制，这使它们容易受到消息注入攻击。

进程隔离和进程（资源）共享是相互矛盾的要求。进程具有虚拟地址空间，系统对象的打开句柄和其他属性。出于可靠性和安全性的考虑，必须保护操作系统中的所有进程免受彼此的活动影响。现代 OS 的保护机制将进程之间对不同种类资源（例如 CPU，内存和 I / O 设备）的访问隔离开来。例如，内存隔离会将每个进程放入其自己的 “地址空间”。

另一方面，现代 OS 支持在进程之间共享，因为共享数据 / 资源可能很有用。以套接字共享为例，一个进程首先创建套接字并建立连接，然后将那些套接字移交给其他进程，这些进程将负责通过这些套接字进行信息交换。但是，进程之间的共享存在风险，因此必须对其进行仔细控制。现代 OS 假设共享资源的进程通过设置适当的安全策略来控制对共享对象的访问，从而确保这种共享的安全性，从而彼此信任。

不幸的是，商用 OS 的默认访问控制策略遭受关于过程信任关系的错误假设。例如，内置的 Windows 安全策略允许同一用户的进程将其开放句柄共享给资源，而内置的 Linux 策略则允许父进程通过 ptrace 访问子进程的内存。这些默认允许策略假定同一用户的进程之间或父进程与子进程之间建立信任关系，这在当今的计算环境中是不现实的。结果，这种默认的允许策略可能会被攻击者滥用。在本文中提供了一个具体的示例，套接字复制攻击，它使恶意进程能够在与网络上的外部实体进行交互时模仿合法进程。

启用 ShadowMove 的另一个潜在问题是，在许多应用程序协议（例如 FTP 和 TDS（用于 MS SQL））中缺少适当的消息源完整性检查。结果，端点无法验证消息的来源，以确保恶意行为者不会交错消息。复制套接字的攻击者可以在客户端的请求之间插入一个请求，并误导服务器以为原始客户端发送了该请求，从而处理了该请求。关于加强消息来源完整性，可以将应用协议分为三类：

• 没有源完整性强制实施：这样的协议没有使服务器能够检查接收到的消息的来源完整性的任何内置机制，因此服务器接受符合协议的任何适当消息。它们容易受到 ShadowMove 攻击，一种典型的协议是 FTP。

• 原始完整性执行不充分：在这些协议中，服务器生成一个随机的随机数供客户端与其请求一起使用，并且服务器使用此随机数来验证接收到的请求的来源。不幸的是，这些协议对 ShadowMove 并不安全，因为攻击者可以等待客户端创建新连接并听取服务器的响应以立即学习。一种典型的协议是 WinRM。

• 充分执行原始完整性：在这些协议中，验证起源完整性所需的部分信息是由客户端而不是服务器生成的。在这种情况下，攻击者无法通过侦听服务器响应来学习该信息。这些协议不受 ShadowMove 的影响，其中一个代表性协议是 SSL。

### **2）威胁模型**

假设攻击者已经在普通用户的特权下建立了受害者系统，并且希望向关键资产进行横向移动。攻击者必须运行恶意软件才能实现此目的。假定将要劫持其 TCP 连接的受害者进程不知道恶意软件进程。

演示场景—以公司的员工自助服务应用程序为例。这是典型的多层企业应用程序，可以从浏览器进行访问。下面是这种系统的组件的描述：

• 运行 Web 客户端的员工台式计算机，一些员工同时是 IT 人员，他们有时需要将内容推送到应用程序服务器，因此他们的计算机上安装了文件复制工具（例如 FTP）。

• 应用程序服务器，运行许多应用程序，例如工资，股票，健康保险，退休计划和差旅。

• 数据库服务器，用于存储人员信息，例如 DOB，SSN，联系信息和薪水，并由应用程序服务器访问。

在此示例中，攻击者通过鱼叉式网络登陆到员工的桌面上，而该员工恰好是 IT 人员。攻击者所追求的关键资产是存储在数据库服务器中的员工信息。因此，攻击者需要从桌面移动到应用程序服务器，然后再移动到数据库服务器。此外，他们需要在数据库服务器上保留一些工具，以便获取有关员工记录更新的每日报告。

为了从桌面移动到应用程序服务器，攻击者可以利用 FTP 连接（请参阅第 4.2 节）将一段恶意软件复制到应用程序服务器，然后等待该恶意软件被执行。例如，应用服务器通常可以在配置文件中指定的路径中运行外部程序（例如，用 C 实现的数据处理应用）。配置文件可能包含 “命令名 = C：\\ users \\ alluser \\ appdata \\ updater \\ dpanalyzer.exe”，并在此基础上，一旦触发了某些相关事件，应用服务器将执行 dpanalyzer.exe。为了使应用程序服务器保持最新，授权 IT 人员将文件复制到应用程序服务器以更新 dpanalyzer.exe。在这种情况下，攻击者可以利用 FTP 连接将一段恶意软件复制到应用程序服务器，以替换合法的 dpanalyzer.exe，然后等待该应用程序服务器执行该恶意软件。攻击者可以通过相同的 FTP 连接获取配置文件的内容。

当恶意软件在应用程序服务器上启动时（例如 dpanalyzer.exe），它可以利用应用程序服务器和数据库服务器之间的数据库连接（Microsoft SQL）来进一步复制和启动。数据库服务器上的恶意软件。

  

  

0x03 ShadowMove Architecture and Design

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHASCSibIbfWkiacuFrsaibGwAcxjZXRnDLEJzfeIicVheR4ibgINaSRLciaVrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAqaCthXOMgGooKerCB1w5QSdG5YDVOL8akhFov1DPEwDrzB4CAcEfpQ/640?wx_fmt=png)

上图描述了 ShadowMove 的总体体系结构，它由六个主要模块组成：连接检测器，套接字复制器，对等处理器，网络视图管理器，横向移动计划器和计划执行器。

ShadowMove 设计的核心是网络视图的概念，它表示受害环境中正常网络通信模式的模型，由在不同受害系统上运行的 ShadowMove 实例共同维护。每个 ShadowMove 实例都维护两个视图：本地视图基于本地系统中的当前连接，而全局视图是通过在 ShadowMove 实例之间交换和传播信息来构造的。

连接检测器模块负责检测可用于横向移动的新建立的 TCP 连接，并请求套接字复制器复制相应的套接字。它还可以检测到 TCP 连接的中断，并通知网络视图管理器。

套接字复制器复制目标进程拥有的套接字，并将其与其他上下文信息（如所有者进程的 PID）一起传递给其调用方。

对等处理器与相邻的 ShadowMove 实例进行通信，以同步其对受感染网络的视图。一方面，它使用从其对等方（例如，新发现的主机）中学到的信息来更新 Net View Manager。另一方面，它将本地 ShadowMove 实例的网络视图发送到其远程对等方。  
网络视图管理器基于来自连接检测器和对等处理程序的通知，结合了几种方法来维护受害网络的全局视图。它还确定每个重复的套接字支持的服务类型，并保持重复的套接字的活动性。

横向移动计划器会根据当前网络视图和重复的套接字所支持的功能定期创建横向移动计划，该计划指定必须使用的套接字，必须执行的操作类型以及有效载荷。

最后，计划执行器器通过向给定套接字发送数据包和 / 或从给定套接字接收数据包，在横向移动计划中执行各个步骤，例如将文件传输到远程服务器。

### **1）ShadowMove 连接检测器**

存在两种检测和跟踪 TCP 连接的方法。首先，可以定期轮询形成的 TCP 连接，并将返回的信息与先前调用的结果进行比较。Windows 上的 TCPView 等工具使用此方法。第二种方法是事件驱动的，注册一个事件处理程序以用于连接的创建或拆除。在 Windows 操作系统中，可以通过创建 WMI（Windows 管理规范）过滤器并注册 WMI 事件使用者来获得有关连接状态更改的信息。但是，注册 WMIevent 使用者需要管理特权。

结果，本研究选择第一种方法。通过在 Windows 上调用 GetTcpTable2 和 GetTcp6Table2，或在 Linux 上运行 netstat -ntp 命令，连接检测器可以获得有关 TCP 连接的基本信息，例如连接状态，本地 IP 地址，本地端口，远程 IP 地址 ，远程端口和所有者进程的 ID。从进程 ID，它可以进一步获取进程名称。当连接检测器观察到连接状态从未建立到已建立时，它将调用套接字复制器有关新的 TCP 连接的信息，然后通知网络视图管理器将复制的套接字添加到池中。另一方面，当它观察到连接状态从 ESTABLISHED 变为 non-ESTABLISHED 时，它会通知 Network View Manager 从池中删除重复的套接字，因为关联的 TCP 连接变得不可用。通知消息包含 TCP 连接的基本信息和所有者进程名称。

在 Windows 上，连接检测器在通知套接字复制器或网络视图管理器之前，会对 TCP 连接进行一些简单的过滤。具体来说，它检查 ShadowMove 进程是否具有足够的权限来打开具有 PRO CESS\_DUP\_HANDLE 访问标志的 TCP 连接的所有者进程，并且跳过那些 ShadowMove 进程没有足够权限的连接。

### **2）对等处理器**

通过对等处理器模块，ShadowMove 实例可以与其相邻的 ShadowMove 实例共享其对受感染网络的看法。使用共享信息的每个实例通过已经受到威胁的系统来构建可访问系统的全局视图，对等处理器模块在单独的工作线程中执行。

执行后，对等处理器将尝试在 I 的工作目录中找到一个配置文件。此文件包含有关用于将 I 移至当前系统的 TCP 连接的信息。然后，ShadowMove 确定先前的 ShadowMove 实例滥用的相应服务器进程和套接字。它通过调用套接字复制器模块来复制此套接字，然后连续侦听复制套接字的传入流量。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHADceUYUp3NW7uoofvR0QzMyEIiaSribngfk6NhRaaqVpb2HhmXTbricpiaA/640?wx_fmt=png)

如上图所示，前驱 ShadowMove 定期挂起客户端进程，然后向远程服务器发送特殊请求。收到此 “信号” 消息后，后继 ShadowMove 将挂起服务器进程。然后，这两个 ShadowMove 实例可以使用类似于距离矢量路由协议的协议来同步其关于网络的知识。  

### **3）网络视图管理器**

此模块根据从连接检测器和对等处理程序接收的信息维护受害网络的全局视图。它管理重复套接字池，并为该池中的每个套接字保留一个元组 <连接状态，本地 IP 地址，本地端口，远程 IP 地址，远程端口，服务类型，所有者 PID，所有者进程名>。除服务类型（或协议）外，连接检测器会传递这些字段中的大多数字段，服务类型（或协议）是通过组合几种方法在称为第 7 层协议检测器的子模块中确定的。首先，它从目标端口进行猜测，因为许多服务都运行在众所周知的默认端口之后，例如，FTP 的默认端口号为 21。其次，它从所有者进程中猜测它们是否是用于以下操作的知名客户端工具一些服务，例如 ssms.exe 或 Microsoft SQL Server Management Studio 是 SQL Server 的客户端。最后，如果端口号和所有者进程信息不足以进行可靠的猜测，它会通过在每个套接字上调用 recv API 并设置 MSG\_PEEK 标志来被动地嗅探网络流量。然后，它利用现有的协议分析技术（例如，Suricata 中的自动协议检测功能）对接收到的有效负载进行分析，以识别应用程序级协议。

网络视图管理器基于重复套接字池，计算一个本地视图，该视图可以由多个谓词表示：系统谓词定义主机的 IP 地址，而连接谓词定义两个系统之间的连接。当它从对等处理程序接收到通知时，这些通知是邻居共享的系统谓词和连接谓词，它通过将谓词合并到其本地视图中来更新其全局视图。

值得注意的是，在 Windows 中，关闭套接字并不总是需要 TCP 连接终止握手。仅当最后一个套接字描述符关闭时，才会发生终止握手。结果，即使所有者进程关闭了其套接字，连接也将保持打开状态。但是，由于多种原因（例如网络故障，远程进程崩溃或连接不活动超时），TCP 连接可能不可用。为防止发生连接不活动超时，网络视图管理器使用 setsockopt API 函数为所有重复的套接字设置 SO\_KEEPALIVE 标志。这样，将通过这些连接自动发送保持活动的数据包。

### **4）ShadowMove 套接字复制器**

套接字复制器在收到来自连接检测器或对等处理程序的请求时，将复制与给定 TCP 连接关联的套接字。方法的基本思想是在目标进程内部复制套接字，并使用生成的套接字秘密访问已建立的 TCP 连接。

**(a)Windows 上的套接字复制**

在 Windows 上，可以调用 DuplicateHandle API 从远程进程复制不同类型的句柄。但是，如 DuplicateHandle 文档所述，此函数不能用于复制套接字。

尽管 Windows 提供了一个名为 WSADuplicateSocket 的 API 来复制套接字，但是不能直接使用此函数，因为它需要进程之间的合作。使用此功能的典型场景如下，源进程创建一个套接字，并希望与目标进程共享它。首先，源进程调用 WSADuplicateSocket 以获取特殊的 WSAPROTOCOL\_INFO 结构。该信息结构通过进程间通信（IPC）机制提供给目标进程。目标进程将信息结构传递给 WSASocket 以在其一侧重建套接字。这种方法的主要挑战（即使用 WSADuplicateSocket）是两个进程必须相互协作才能复制套接字，而在本文的方案中，攻击者想要从一个粗心的受害者进程中复制套接字，情况并非如此。解决此问题的一种方法是将代码注入受害者进程中，以实现由于缺乏合作而缺少的步骤。但是，现有的防御机制（例如 WindowsDefender ATP）标记了通用进程注入技术的使用，这使得解决方案的吸引力降低了。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAeSrsg7eN7picnW5Q8SsCarSvrmXNtRCicbPJR3JypIBNp8HlXajNnZ1w/640?wx_fmt=png)

本文以一种非常规的方式使用 Windows API 设计了一种新颖的技术，该技术使攻击者进程可以从目标进程复制套接字，而无需其协作。上表描述了攻击者进程执行的从目标进程复制套接字的步骤，假设它知道了目标的进程 ID，这要归功于实时连接检测。首先，它通过使用 OpenProcess 枚举目标中所有打开的句柄来打开目标进程。攻击者进程仅寻找名称为 \\ device \\ afd 的文件句柄（步骤 3-5，而 afd 代表辅助功能驱动程序）。在此操作期间，攻击者进程将复制所有文件句柄，这是读取句柄名称所必需的。研究发现，攻击者进程可以将这些重复的 afd 进程视为套接字。为了找到与 TCP 连接相对应的确切套接字，攻击者进程会获取套接字 afd 句柄所连接的远程 IP 地址和远程端口（通过调用 getpeername），并将它们与连接检测器传递的信息进行比较。如果存在匹配项，则攻击者进程会将 afd 句柄传递给 WSADuplicateSocketW，以获取复制原始套接字所需的信息。获取协议信息结构后，攻击者进程将调用 WSASocketW 函数来复制套接字。然后将此套接字与上下文信息（例如所有者 PID，所有者进程名称，本地 IP 地址，本地端口，远程 IP 地址和远程端口）一起保存在重复套接字池中。

还值得注意的是，在 Windows 上，用于 IPv4 / 6 的 TCP 连接表仅包含有关原始套接字描述符的信息，而不包含有关重复的套接字描述符的信息，并且即使在所有者进程终止后，套接字描述符的所有者 PID 也不会改变。这意味着依赖于 Windows API 来检索 TCP 连接表的常规工具（如 netstat）不能用于检测连接是否重复以及其复制器。

**(b) 深入探讨 Windows 上的套接字复制**

要了解 ShadowMove 的套接字复制为何起作用，必须首先了解套接字上下文。Thewinsock2 库在不同层的许多数据结构中为每个套接字句柄维护套接字上下文（下图）。在 WS2\_32.dll 中，有一个名为 sm\_context\_table 的哈希表，该哈希表将套接字句柄映射到 DSOCKET 对象，该对象存储有关套接字的信息，例如进程和服务提供者。在下一层，mswsock.dll（服务提供商），还有另一个称为 SockContextTable 的哈希表，该哈希表将套接字句柄映射到 SOCKET\_INFORMATION 对象，该对象存储诸如套接字状态，引用计数，本地地址和远程地址之类的信息。套接字上的每个用户级操作（例如 connect，send 和 recv）都必须引用并可以更改套接字上下文（例如，远程地址和引用计数）。此外，针对每个过程维护包括哈希表的这种上下文信息。套接字功能的内核端是辅助功能驱动程序或 AFD.sys，它还维护套接字上下文信息（例如，本地地址和远程地址），这对于内核驱动程序最终构造网络数据包是必需的。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAUoZrhZVHKALkYGzHE48ZuzWfkDq0nQzoDcdl7jnR8QEhG3CZrnuGWg/640?wx_fmt=png)

通过 WSADuplicateSocket 在普通套接字共享期间会发生什么：Win dows 上的普通套接字共享涉及三个步骤，如上图所示。当源进程调用 WSASocket 创建新的套接字时，它会做三件事：（1）调用 NtCreateFile 获取套接字句柄（例如，句柄 1），（2）为句柄 1 创建新的 SOCKET\_INFORMATION 对象，以及（3）调用 NtDeviceIoControlFile 来设置句柄 1 的内核侧上下文信息。接下来，当源进程调用 WSADuplicateSocket 与以下对象共享句柄 1 时在目标进程中，它首先创建句柄 1 的副本（例如，句柄 2），然后将句柄 2 放入 WSAPROTOCOL\_INFO 结构的 dwProviderReserved 字段中，以便与目标进程共享。当目标进程使用 WSAPROTOCOL\_INFO 结构作为一个参数调用 WSASocket 时，WSASocket 从 dwProviderReserved 字段中提取句柄 2，并使用它来调用 NtDeviceIoControlFile 以获得内核端上下文信息；完成此操作后，它将使用获得的信息为句柄 2 构造一个 SOCKET\_INFORMATION 对象，这使句柄 2 成为功能性套接字句柄。

ShadowMove 的套接字劫持过程中会发生什么：使用上面相同的场景，ShadowMove 攻击可以与句柄 1 秘密共享套接字，而无需源进程的配合。Shad owMove 还使用了 WSADuplicateSocket 和 WSASocket 的组合，但它又做了一个准备工作：它首先通过调用 NtDuplicateObject 创建 Handle 1 的副本。之所以需要这样做，是因为句柄 1 位于源进程的地址空间中，因此 Shadow Move 无法直接对其进行操作，但是 ShadowMove 可以直接使用重复的句柄（例如，句柄 1），因为它是在 ShadowMove 的上下文中创建的。接下来，Shad owMove 调用 WSADuplicateSocket 与自己共享 Handle1。结果，创建了句柄 2，并将其放入 WSAPROTOCOL\_INFO 结构的 dwProviderReserved 字段中。最后，ShadowMove 以 WSAPROTOCOL\_INFO 结构作为一个参数调用 WSASocket，以使 Handle 2 成为功能性的套接字句柄。由于 WSADuplicateSocket 和 WSASocket 是在同一进程（即 ShadowMove）中调用的，因此无需在进程之间传递 WSAPROTOCOL\_INFO 结构。

**(c)Linux 上的套接字复制**

在 Linux（或 \* NIX）上的套接字复制设计与 Windows 上的套接字复制不同。由于更严格的进程隔离，即使另一个进程由同一用户拥有，也无法直接从另一个进程复制套接字。但是，Linux 上支持套接字共享，但是这需要两个进程之间的合作。由于 ShadowMove 假定受害者应用程序不合作，因此解决方案是通过将代码注入到其地址空间中来强制受害者应用程序进行合作，以建立与 ShadowMove process 的套接字共享。要将代码注入受害者应用程序，创建了一个启动器，它将启动受害者应用程序作为子进程，然后利用 ptrace 以共享库的形式注入代码。最后，在命令搜索路径中将启动器版本放在原始受害者应用程序的前面，以便用户在运行受害者应用程序时会调用启动器。

应该注意，与 Windows 上的 ShadowMove 相比，使用进程注入可以减少 Linux 上 ShadowMove 攻击的隐蔽性。但是，Linux 设计仍然有很大的机会规避最先进的防御措施。将对评估进行详细讨论。

Linux 上的套接字共享：为了共享一个套接字，两个进程首先通过 Unix 域套接字连接，然后发送方进程调用 sendmsg 并在输入参数中传递套接字描述符，而接收方调用 recvmsg 并从中检索一个（可能不同的）套接字描述符。输出参数。当以这种方式传递套接字描述符时，底层的 Linux 内核会在接收进程的地址空间中创建一个新的描述符，该地址指向内核中与结束进程发送的描述符相同的文件表条目。

更具体地说，在 Linux 上有四个 ShadowMove 攻击组件，分别是目标进程，共享库，启动器和 ShadowMove（下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAqH1t1xcpAomyLuXpSLgMd8IR9afuR2qEBgxiafEMmhNGKptBEADvQ6A/640?wx_fmt=png)

启动程序通过使用 ptrace 将共享库注入目标进程，该进程必须首先附加到目标进程。当前的 Linux 系统对 ptrace 施加了严格的控制。具体来说，默认情况下，Yama Linux 安全模块（LSM）仅允许具有 sudo 特权的进程或从父进程到子进程的 ptrace。使用第二个选项，因此不需要特权升级。启动器将目标应用程序作为子进程运行，然后使用 ptrace 附加到目标进程。之后，它调用\_\_libc\_dlopen\_mode 将共享库加载到目标进程中。  

本研究开发了共享库的原型，该共享库的构造函数（在加载库时自动执行）枚举目标进程中的打开套接字。对于每个打开的套接字，它使用 dup 方法创建该套接字的副本，通过 Unix 域套接字连接到 ShadowMove 进程，并使用该通道共享重复的套接字。如果没有打开的套接字，它将休眠一段时间并尝试再次查找打开的套接字。为了避免阻塞目标进程的主线程，创建了一个专用于套接字复制的新线程。

为了使受害用户在打算运行目标应用程序时无意中运行启动器，给启动器起一个与目标应用程序相同的名称，并确保启动器在命令搜索路径中位于目标应用程序的前面，可以通过更改 PATH 环境变量来完成。为了使攻击更加隐秘，如果当前命令搜索路径上的任何位置是（1）受害用户可写的并且（2）在目标应用程序的位置之前，则可以避免更改 PATH 环境变量。情况下，只需要在该可写位置复制启动器即可。否则，将创建一个看起来是良性的文件夹（例如，可被名为 npm 的良性应用程序使用的 / home/alice/.npm-packages/bin），将启动器复制到此处，然后添加通过将 export PATH = / path / of / the / launcher：$ PATH 添加到受害者用户的. bashrc 中，将新文件夹位置添加到 PATH 环境变量中。

例如，如果 ftp 是目标应用程序，则启动器将命名为 ftp。当用户尝试运行 FTP 时，启动器将被执行，它将作为子进程运行原始 FTP 应用程序

**(d) 良性应用程序与攻击之间的竞争**

应该注意，在提出的攻击中，套接字在原始客户端和攻击者之间共享，这可能导致从远程端点接收和发送数据时出现竞争状况。首先调用 recv 函数的人将从输入缓冲区中获取数据，而首先调用 send 函数的人会将数据发送至服务器。这可能导致从服务器读取部分响应或向服务器发送乱码请求。为了避免这种可能性，攻击者可以在客户端正在从服务器发送 / 接收数据时，暂时将客户端进程暂停，然后再恢复客户端进程。要暂停客户端进程，攻击者可以通过调用 SuspendThread 暂停其所有线程，而要恢复客户端进程，攻击者可以使用 ResumeThread 恢复所有线程。

### **5）横向移动计划器（LMP）**

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHA03RmibYlnbOxeJnHaTm28YZtf7rdI7ibCvIfNicqPiakvibQ6tgLDDtOLwQ/640?wx_fmt=png)

横向移动计划器（LMP）可以使敌方协调多个受害者系统上的攻击行动，从而优化攻击效率和隐蔽性。例如，假设上图中的攻击者已承诺将主机 A 和 B 都连接到主机 C，但是它们各自的连接不足以进行横向移动（例如，A 的连接只能复制恶意软件，而 B 的连接只能执行恶意软件）。在这种情况下，同时涉及 A 和 B 的协调计划（例如，将恶意软件复制到 C，然后 B 远程在 C 上启动恶意软件）将允许横向迁移到 C，从而使攻击更加有效。对于另一个示例，如果存在到目标系统的多条路径，则协调计划将允许攻击者使用最短路径将有效载荷发送到目标 / 从目标接收到数据，从而使攻击更加隐秘。假设攻击者寻找一组特定的目标，这些目标可以在达到目标时被识别。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAMq3YqPofnwZIU11GibVmKn2V3TnbJEdt8tbicq8mu3ue9TjnI3a1Q8fg/640?wx_fmt=png)

在 Prolog 中制定了攻击计划问题：使用上表中的谓词指定受感染网络的当前状态：system 和 connected 指定可访问系统及其互连，并且 commit 定义了 ShadowMove 实例已在系统上执行的操作。对于每个协议，还使用能力谓词来指定攻击者劫持相应 TCP 连接时可以执行的操作。  

前图展示了系统 B（IP 地址为 10.10.10.50）ShadowMove 知识库的快照，该知识库由一组事实组成，这些事实代表具有三个受感染系统和一个目标的网络。此知识库是从所有 Shadow Move 实例之间共享的全局视图构造的。LMP 使用以下规则来确定是否可以在给定系统 X 的远程系统 Y 上执行特定操作。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAYOqNeCSblTgoXptHwJWjdoPjCAafI5ZtlZibtchVzXFbWAEN1aRkXqg/640?wx_fmt=png)

通过使用 remoteOperation，ShadowMove 实例可以检查两个系统之间是否存在路径，以允许它们执行特定的操作，例如执行或上载文件。例如，攻击者可以执行以下查询：  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAaiax6amTw7rDWNAoZiaXSTRcK5lgPm68f6e9YpibtXJPKorGWRaloDeicQ/640?wx_fmt=png)

它返回 \[010.10.10.100，0 10.10.10.300，0 10.10.10.1000\]。此结果意味着，到达 10.10.10.10 并已移至 10.10.10.30 的攻击者可以通过 ShadowMove 执行器之一将恶意软件从 10.10.10.30 复制到 10.10.10.100。可以使用 remoteOperation 谓词来构造更复杂的谓词，例如 commitExecuteOperation：  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAxxMSmym6PmDIPmZQVBfJ1azicR9v1HY6Mv9pDTdm73wLT0u9n6uy2Xg/640?wx_fmt=png)

为了从受威胁的系统上在目标系统上运行 ShadowMove，不仅这两个系统之间必须存在允许 ShadowMove 立场执行执行操作的连接，而且文件还必须通过以下方式上载到该目标系统：执行操作之前的阴影移动实例之一。例如，在前图中，当且仅当（1）存在允许系统 B 在系统 C 上执行文件的连接时，系统 B 才能在系统 C（目标）上启动 ShadowMove：  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAI9VBeuZxOVUfZUraSf6ylia9CjNBfMlbnibFY5LoulfPzIMKc9nKoic2g/640?wx_fmt=png)

（2）ShadowMove 二进制文件已上传到系统 C：  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAxmtyq4z3Iricxic7KIG5akjKZkPt5IL5Wb5fibQTc86ZpgNib2dic4iczuOg/640?wx_fmt=png)

如果基于其当前的知识库，没有任何 ShadowMove 实例将文件上传到目标上，则系统 B 必须等待直到 ShadowMove 实例之一（例如系统 A 上的一个）提交了上载操作。系统 B 可以在其上启动 ShadowMove 的目标系统，系统 B 上的 ShadowMove 实例可以执行以下查询：  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAbas0qZ4fWY5ZZQoS4sMOVuOvtAYuF6flfBI36Wib9RKYd8N2McMgq4A/640?wx_fmt=png)

如果返回的 ExecuteList 不为空（例如 \[‘10 .10.10.100’\]），则可以在新的目标系统（例如 10.10.10.100）上启动 ShadowMove 的实例。这是横向移动的说明，它要求在不同路径之间进行协调，只有在可以全面了解受感染网络的情况下才有可能。  

### **6）横向移动执行器**

横向移动执行器（LMA）是一个模块管理器，其中包含多个执行模块。这些模块中的每一个都负责处理一种协议，例如 TDS。LMA 可以采取被动和主动行动。在被动模式下，模块仅绕过 MSG\_PEEK 标志从套接字读取，以进行 Recv API 调用。这样，不会清空输入缓冲区，因此原始进程可以读取内容。在活动模式下，模块不通过 MSG\_PEEK 标志就从套接字读取数据；因此 recvcall 会消耗输入缓冲区中的数据。在这种状态下，模块还写入套接字输出缓冲区以发送经过精心设计的消息。

在某些协议中需要学习一些秘密，然后才能制作有效的消息（例如，WinRM 的 shellID）。在这些情况下，执行器模块以被动模式启动，嗅探接收消息以了解此类秘密值。在学习了所有这些必需的数据元素之后，执行器模块可以将其自身切换到活动模式并开始与远程端点进行通信。值得注意的是，LMA 模块只能读取传入的消息。它无法读取传出的消息，没有这样的 API 允许人们从套接字输出缓冲区中读取消息。在当前的原型中，LMA 具有用于 FTP，MS SQL 和 WinRM 协议的三个驱动模块。但是，可以通过实现称为 IPModule 的接口向 LMA 添加新协议。

  

  

0x04 Prototypes for ShadowMove Actuators

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHASCSibIbfWkiacuFrsaibGwAcxjZXRnDLEJzfeIicVheR4ibgINaSRLciaVrg/640?wx_fmt=png)

用 2,501 行 C / C ++ 代码在 Windows 上实现了 ShadowMove 设计的原型。横向移动计划器基于 SWI-Prolog，它是 Prolog 编程语言的免费实现。原型展示了一些常见功能，例如连接检测，套接字复制，网络视图同步和横向移动计划；它还克服了操作上的挑战，即如何使注入的数据包与各自的协议保持一致，并且对横向移动（例如上传恶意软件和启动恶意软件）有用，这特定于各个应用程序协议。

在本节中介绍三个利用 FTP，MS SQL 和 WinRM 的 ShadowMove 执行器。正如在之前中讨论的那样，选择这些协议的标准是它们缺乏对消息来源完整性的支持。具体来说，FTP 和 Microsoft SQL 没有原始完整性强制，而 WinRM 没有足够的原始完整性强制。

### **1）ShadowMove 实例化**

对于每个实验，首先准备一个包含受害者应用程序的目标环境，例如，一台运行 FTP 客户端的计算机和另一台运行 FTP 服务器的计算机。对应用程序进行配置，以使它们不会按预期目的运行，在受害客户端计算机上启动 ShadowMove PoC。观察到，一旦建立候选 PoC，PoC 就会定期检测候选 TCP 连接（受害客户端应用程序不必在 PoC 之前启动），复制相应的套接字，并确定在 TCP 连接上运行的协议（例如 FTP） 。

PoC 定期查询横向移动计划器模块（通过显示其当前网络视图），并在计划程序返回下一个移动目标时执行执行器逻辑（例如，使用 FTP 连接将 PoC 复制到 FTP 服务器） 。在服务器计算机上启动 PoC 时，看到它检测到活动的 TCP 连接（包括与客户端计算机的 TCP 连接）并复制了相应的套接字，进一步观察到，服务器上的 PoC 与客户端上的 PoC 成功交换了 “信号” 消息，然后它们交换了当前的网络视图。这样，两台计算机上的网络视图都会更新。一段时间后，再次查询横向移动计划器模块，以基于新的网络视图做出下一个决定。

上述情况对于所有三个执行器都是通用的。因此，在单个执行器的描述中省略了这些细节。利用 FTP 的 ShadowMove PoC 演示视频可在（http://54.36.162.222/ShadowMoveDemo/ShadowmovePrototypeDemo.mp4，请点击文末 “阅读全文” 查看链接 ）中找到上述情况的案例。在本演示中，将 ShadowMove PoC 移至 FTP 服务器后手动启动它，但是可以通过 WinRM 自动启动 PoC。

### **2）FTPShadowMove：劫持 FTP 会话**

本研究开发了可以劫持 Windows 10 和 Ubuntu 18.04 上已建立的 FTP 连接的原型系统。它们在 ftp 的默认安装下工作，不需要任何提升的特权。它们使攻击者无需身份验证即可将文件下载和上传到远程 FTP 服务器。

在 FTP 协议中，客户端使用一个 TCP 连接将命令发送到服务器并从服务器接收相应的响应。此连接称为命令通道。客户端还使用另一个 TCP 连接来发送或接收数据，例如文件内容。该连接称为数据通道。客户端可以为给定的命令通道打开多个数据通道。仅在建立命令通道时才需要身份验证，这意味着客户端无需重新身份验证即可创建新的数据通道。劫持命令通道的攻击者可以向服务器发送请求以打开一个自己的新数据通道，从而避免与现有数据通道上正在传输的客户端内容发生任何冲突。但是，攻击者仍应采取策略来防止共享命令通道中的条件竞争。请注意，由于合法客户端也可能会打开新数据通道，因此无法仅通过监视新数据通道的创建来检测攻击。

FTP 客户端可以通过两种方式请求创建新的数据通道：主动 FTP 和被动 FTP。在活动的 FTP 中，客户端向服务器发送端口命令，以指定服务器需要重新建立连接的端口。在被动 FTP 中，客户端将 PASV 命令发送到服务器，要求服务器侦听客户端可以连接的端口以创建新的数据通道。简而言之，这两种模式之间的区别在于谁发起了新的 TCP 连接：主动模式下的服务器和被动模式下的客户端应该分别连接到客户端和服务器指定的端口。在原型中，实现了被动 FTP 进行演示。但是，主动 FTP 也可以轻松实现。

在被动 FTP 中，客户端将 PASV 命令发送到服务器，服务器通过提供有关客户端必须连接到的端点（包括 IP 地址和端口）的信息来进行响应，以创建新的数据通道。PASV 在 RFC-959 中记录。

实验设置：在互联网上托管的基于 Linux 的虚拟专用服务器上部署了 vsftpd 服务器。对于合法客户端，使用 ftp 命令和 Windows 资源管理器连接到已配置的服务器。服务器上的匿名登录已被阻止，因此客户端需要发送有效的用户名和密码才能连接到该用户。从下图上半部分的演示视频中可以看到，客户端与服务器交换了几条消息，以便登录到服务器。之后，以与 ftp 客户端相同的用户帐户启动 FTPShadowMove。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAKSMsqiaicA1DNnlUne3z0IKXlNPRosSA0y1DLSibrtu5jpmhzrqIJfBrQ/640?wx_fmt=png)

FTPShadowMove PoC 首先通过复制相应的套接字来劫持 FTP 连接，然后发送若干命令以将二进制文件上传到服务器上的特定目录。上图的下半部分显示了特定的命令（例如 CWD / files /）和服务器响应。具体来说，可以看到服务器在端口 45307 上响应 54.36.162.222（即 176\*256 + 251）。然后，FTP ShadowMove 请求在服务器上上载名为 PoC2.txt 的文件。从服务器收到响应代码 150 后，FTPShadowMove 打开了到指定远程端点的 TCP 连接，并将文件的内容发送到打开的连接。服务器将文件解释为二进制内容，并将其存储在服务器上的 / files/PoC2.txt 中。

在 Ubuntu 18.04 上的原型使用与上述相同的 FTP 命令，有关其工作原理的视频片段，请参见 http://54.36.162.222/ShadowMoveDemo/LinuxShadowMove.gif （请点击文末 “阅读全文” 查看链接 ）。

在原型系统中，仅使用了一些 FTP 命令。但是，攻击者还可以使用许多其他 FTP 命令。具体来说，FTP SITE 命令允许用户通过主机上的 FTP 服务器执行有限数量的命令。无需进一步的身份验证即可执行该命令。可能执行的命令因系统而异，一些有用的命令包括 EXEC 和 CHMOD。EXEC 命令在服务器上执行提供的可执行文件，可用于启动恶意软件。幸运的是，在许多系统上都未实现 SITE 命令，如果可能，还建议在 FTP 服务器上禁用 SITE 命令。

### **3）SQLShadowMove：劫持 Microsoft SQL 会话**

已经确认，可以（1）劫持 Microsoft SQL 连接以将恶意软件可执行文件从 SQL 客户端计算机上载到 SQL Server，以及（2）在 SQL Server 上执行恶意软件。

实验设置：使用 Microsoft SQL Server Management Studio 17 作为合法的 SQL 客户端，并使用 Microsoft SQL Server 14.0.1000.169 版作为服务器。在 SQL 服务器上配置一个可以创建数据库和表的用户。首先启动 SQL 客户端并登录到服务器。然后，运行概念验证 SQLShadowMove。确认 PoC 在 Microsoft SQL 的默认安装和常规应用程序设置下有效。

SQL 劫持方案需要满足一些先决条件才能成功工作：（1）流量未加密，（2）SQL Server 上有一个可由 SQL Server 进程写入的文件夹，（3）SQL 客户端已成功通过 SQL 身份验证服务器（4）SQL 客户端承担着允许在 SQL 服务器上创建表的角色。

通常可以满足上述前提条件。默认情况下，Microsoft SQL 通信未加密，并且％TEMP％文件夹始终可由 SQL Server 上的任何进程写入。而且，SQL Server 几乎是无状态的。客户端和服务器使用 TDS（表格数据流）协议进行通信。尽管 TDS 标头中的几个字段是为维护某些状态而设计的，但它们是可选的或当前实现未使用。例如，TDS 数据包头中的 SPID 字段是服务器上与当前连接相对应的进程 ID。如果严格检查此 ID，则攻击者必须在构造恶意数据包之前以某种方式学习它。不幸的是，此字段不是必需的，服务器可接受的值为 0x0000。同样，还定义了两个字段，但将其忽略：PacketID 和 Window。

TDS 数据包有几种类型，与本研究的攻击最相关的类型是 “批处理客户端请求” 类型，其有效载荷可以是任何 SQL 语句的 Unicode 编码，并且数据包头中没有校验和。这使得捕获真实的批处理客户端请求数据包然后将其用作模板以通过将负载替换为新的 Unicode 字符串来创建新的恶意请求变得很简单。在例子中，这些字符串对应于一系列 SQL 语句。

SQLShadowMove 首先检测由 SQL 客户端进程创建的 TCP 连接，然后复制相应的套接字。然后，它使用复制的套接字将一系列批处理客户端请求数据包发送到 SQL Server，并从服务器接收任何响应数据包。这些批处理客户端请求数据包的有效负载由 SQL 脚本组成，这些脚本将可执行文件上载到 SQL Server 并执行该文件。

具体来说，SQL 脚本首先在 SQL Server 上创建一个表，然后将可执行文件中的字节块插入表中。最后，它们调用 bcp 命令将表的内容导出到服务器上的常规文件，从而还原原始的可执行文件。SQL 脚本的伪代码如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAsAP8ac3acYsj0enZYhFlR4bbKjJRNXwgqnGcERHC0r03GzzUPu0XMw/640?wx_fmt=png)

使用 SQL 服务器上的可执行文件，原型可以通过 SQL 语句进一步运行它。为了通过实验确认 SQLShadow Move 的可行性，开发了一个简单的 Windows 应用程序（名为 notepad.exe）来表示一部分 “恶意软件”。此应用程序在与应用程序可执行文件相同的文件夹中创建一个文件（名为 notepad.txt），并将当前日期和时间写入该文件。然后，生成 SQL 脚本，以将简单的“恶意软件” 上载到 SQL Server 上的％T EMP％\\ notepad.exe 并运行它。运行 SQLShadow Move 的概念验证后，可以直观地确认 SQL Server 上首先出现 notepad.exe，然后出现 notepad.txt，并且其内容与 SQL Server 上的时间和日期匹配。有关 SQLShadowMove 的工作原理的视频剪辑，请参见 http://54.36.162.222/ShadowMoveDemo/SQLShadowMove.gif （请点击文末 “阅读全文” 查看链接）。

请注意，为了运行 bcp 命令或可执行文件，必须在 SQL Server 上启用 xp\_cmdshell。但是，这对于原型来说并不是障碍，因为 SQL 脚本在使用 xp\_cmdshell 之前先启用它。

### **4）WinRMShadowMove：基于 WinRM 的远程执行**

Windows 远程管理（WinRM）是 Windows 的一项功能，允许管理员远程运行管理脚本。已经确认，可以劫持 WinRM 会话以在远程计算机上运行恶意软件。假设远程计算机正在运行 WinRM 服务，并且恶意软件已上传到远程计算机，并且只需要启动它即可。

**(a)WinRM 协议简介**

WinRM 协议使用 HTTP 与远程服务器进行通信。要使用远程计算机进行身份验证，WinRM 具有六个身份验证机制：基本，摘要，Kerberos，协商，证书和 CredSSP。默认情况下，它使用协商。WinRM 客户端首先通过 WinRM 服务器进行身份验证。验证后，WinRM 客户端从服务器接收一个 shellID，该 ID 将在以后的通信中使用。除了 shellID 之外，每个请求消息中还有一些其他 ID。messageID 用于将响应消息与相应的请求消息配对，并且在响应消息中，请求消息 ID 显示为 “RelatesTo” 字段。下图说明了 WinRM 会话期间的消息交换。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHAqXMkMUYnVeCSSzgV5SicVQrQXniarjTsVOefZibdzY5rSnPB0oS1ZKRVA/640?wx_fmt=png)

**(b) 实验设置**

为了准备 WinRM 劫持的环境，首先在 Windows 10 上为正常应用程序场景设置 WinRM，其中包括在服务器和客户端上同时启用 WinRM，并将服务器添加为客户端计算机上的受信任主机。然后，可以使用客户端计算机上的命令行工具 winrs 在服务器上运行命令。

但是，ShadowMove 在上述默认设置下不起作用，因为 WinRM 通信量已默认加密。为了使 WinRMShadowMove PoC 能够正常工作，管理员必须将 WinRM 服务器配置为具有基本身份验证，并允许传输未加密的数据。应该注意，这种配置并不罕见，因为它可以使 WinRM 快速工作，并且某些第三方 WinRM 客户端和库需要未加密的有效负载才能与 WinRM 服务器通信。

**(c) 劫持 WinRM**

为了演示 WinRMShadowMove 的工作原理，在客户端计算机上，运行命令行 winrs -un -r：http：// host\_ip：5985 -u：user -p：pass cmd，这将创建一个新的 winrs 进程并打开命令外壳到远程计算机。-un 标志指定将不对请求和响应消息进行加密。同时在另一个终端中，运行 WinRMShadowMove。

当 winrs 进程开始执行时，它将建立与 WinRM 服务器的 TCP 连接，并由连接检测器捕获。结果，连接检测器通知套接字复制器，后者在 winrs 进程中查找并复制该套接字。WinRMShadowMove 首先以被动模式运行（即，通过重复的套接字窥探传入的网络数据包），以便从服务器获取 shellID；然后切换到活动模式。

因为 WinRM 服务器支持未加密的有效负载，所以可以构造纯文本 HTTP 有效负载并将其通过 TCP 套接字发送到服务器。为了使该方案起作用，构造的有效负载必须对服务器合法。在使用 Wireshark 分析 HTTP 请求和响应数据包之后，发现 MessageID 对于每个有效负载都是唯一的，并且实际上是 UUID。

因此，使用 UUID 生成器生成 messageID。此外，从身份验证响应消息中获取 shellID。使用这两个 ID，可以构造有效负载以在远程 WinRM 服务器上执行可执行文件。为了学习如何构造有效负载，利用了一个称为 winrm4j 的开源 WinRM 客户端与远程 WinRM 服务器进行通信，并且将 winrm4j 生成的请求数据包用作有效负载的模板。下图显示了示例 WinRM 请求的有效负载。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHARXsusGuAnWx6icuCF7XyD4SLqiaGNr8CkyERGfZAEVDUssBCEyosHyJA/640?wx_fmt=png)

在使用劫持的 TCP 套接字将有效负载发送到远程计算机之前，WinRMShadowMove 会挂起合法的进程，以防止其从 WinRM 服务器获取响应消息。从 WinRM 服务器获得响应后，它将恢复合法客户端。暂停和恢复之间的时间间隔非常短，因此合法客户可能不会注意到它。上图显示了攻击消息与合法 WinRM 消息的交换。  

  

  

0x05 Evaluation of ShadowMove Proof-of-concepts

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHASCSibIbfWkiacuFrsaibGwAcxjZXRnDLEJzfeIicVheR4ibgINaSRLciaVrg/640?wx_fmt=png)

### **1）理论评估**

正如前文中演示的那样，当前最先进的横向移动检测器无法检测到 ShadowMove。在本节中，讨论了导致现有解决方案在检测 ShadowMove 横向移动方面无效的根本原因。

在主机级别，为了执行横向移动，在 Windows 上进行的 ShadowMove 设计依赖于其他良性进程也常用的一些 API 函数。例如 Windows 上的许多进程都使用带有 PROCESS\_ALL\_ACCESS 访问标志的 OpenProcess 进行调用，这实际上是在要求对目标进程的所有可能的许可，包括复制其进程的许可。此外，ShadowMove 调用 WSADuplicateSocket，它也具有合法的用例，例如将套接字卸载到子进程。其次，很难从套接字描述符追溯到有权访问它的所有进程，因为套接字所有者中仅记录所有者的进程 ID。

当前在 Linux 上进行的 ShadowMove 设计需要对攻击者有更强的假设，因为它依赖进程注入来强制受害者应用程序进行协作，这使其不如 Windows 同类程序隐蔽（例如，通过监视良性代码段的运行时完整性）在应用中，人们可以检测到代码注入的效果。此外，由于设计可能会修改系统的配置（例如 PATH 环境变量和. bashrc），因此可以通过监视此类更改来检测到它。

具体来说，检测 Linux 上的 Shad owMove 攻击存在实际挑战。当前的 Linux 发行版不支持对应用程序进行运行时代码完整性监视，并且已知的监视工具需要系统管理程序或特殊的硬件。监视配置更改以检测 ShadowMove 也不是一件容易的事，因为许多良性应用程序（例如 npm）也对 PATH 环境变量和. bashrc 进行了更改；因此，监视工具必须检查精确的条件（最有可能是特定于应用程序的），以避免错误警报。将启动器隐藏在看似良性的路径下（例如 / home/alice/.npm-packages/bin），这进一步提高了检测的标准。在 Linux 上流行的几种基于主机的基于 IDS 的经验中证实了这一点：OSSEC ，Osquery 和 Wazuh，它们无法使用现有规则检测 ShadowMove。当然，可以添加新规则来检测 ShadowMove 的特定实例，但是这并非易事。

在网络级别，ShadowMove 通过两端良性进程建立的现有连接来隧道传送其消息。换句话说，它会将其消息注入由良性客户端发送到远程服务的良性消息流中。因此，检测异常的新连接的基于异常的解决方案对于 ShadowMove 来说是无关紧要的。此外，在客户端和远程服务器执行所需的身份验证步骤后，ShadowMove 开始横向移动。这意味着 ShadowMove 操作不需要任何其他身份验证尝试。结果，那些将用户登录活动与网络连接活动相关联的异常检测解决方案是无效的。

### **2）实验评估**

在本节中将在企业环境中通常存在的基于主机和基于网络的防御机制下，对 ShadowMove 进行广泛评估。更具体地说，针对新兴的终端检测响应（EDR）系统，一流的防病毒产品，基于主机的 IDS 和基于网络的 IDS 对 ShadowMove 进行了测试。

在新兴的终端检测响应（EDR）系统（即 CrowdStrike Falcon Prevent 和 Cisco AMP）存在的情况下评估 ShadowMove。EDR 与评估有关，因为某些 EDR（例如 CrowdStrike Falcon）旨在检测横向移动。还会在存在基于主机的 tivirus 产品的情况下对 ShadowMove 进行评估：选择排名前 50 的前四名防病毒产品进行评估（McAfee，Norton，Web root 和 Bitdefender）；还选择 Windows Defender，因为它是 Windows 系统上的默认 AV。此外，选择 Snort IDS 来针对基于网络的解决方案评估 ShadowMove（使用了 Snort 规则 V2.9.12）。最后，对于 Linux 上的 ShadowMove 设计，使用三种流行的基于主机的 IDS（OSSEC，Osquery 和 Wazuh）对其进行评估。

反制 EDR 和 IDS 解决方案：实验确认，ShadowMove PoC 可以逃避对 Strike Falcon Prevent，Cisco AMP，OSSEC，Osquery，Wazuh 和 Snort（Windows 和 Linux）的检测。详细结果如下表所示。在评估期间，使用了此类工具提供的默认检测规则。研究还手动检查这些默认规则，以了解为什么它们无法检测到 ShadowMove。例如，默认的 Osquery 规则根本不提及 ptrace 或进程注入。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHA8NVb4m2HyRa3kFqUZM2G943BH7l3uoKO96fhLol4FkrgegnIXKSmfQ/640?wx_fmt=png)

绕过基于主机的防病毒产品：还通过实验证实，ShadowMove PoC 可以逃避 Windows 10 上上述五个 AV 的最新版本的检测（这些 AV 没有 Linux 版本），总体结果见上表。

供应商反馈：联系了 Microsoft 安全响应中心（MSRC），并为报告的问题总结了一个案例（编号 46036）。2018 年 6 月 21 日，MSRC 忽略了报告的漏洞漏洞，指出 “此行为是设计使然…… 因为从系统安全的角度出发，如果没有完整的进程，则无法复制进程中的句柄。控制它，到那时还有许多其他攻击可能。” 微软工程团队的反馈证实，应对此攻击并非易事，因为要完全解决该攻击，将需要重新设计 Windows 中句柄的访问控制机制。这也意味着像 ShadowMove 这样的技术将在可预见的将来继续帮助 Windows 上的攻击者。

  

  

0x06 Discussions and Future Work

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHASCSibIbfWkiacuFrsaibGwAcxjZXRnDLEJzfeIicVheR4ibgINaSRLciaVrg/640?wx_fmt=png)

ShadowMove 的可能缓解方法。通过解决现有计算环境中的两个基本弱点，可以减轻 ShadowMove 的负担。一种想法是更好地将合法进程与潜在的攻击者进程隔离开，以防止套接字窃取。例如，可以将合法进程设置为 Protected（在 Vista 中引入）或 Protected Process Light（在 Windows 8.1 中引入）进程，这样不受保护的进程就无法使用 PROCESS\_DUP\_HANDLE 打开合法进程。但是，这种方法有局限性，例如不能保护具有 GUI 的进程，并且程序文件必须由 Microsoft 签名。另一个想法是像 SSL 一样，在常见的企业计算协议中引入强大的源完整性机制。但是，这可能会破坏许多旧版应用程序。

当前 ShadowMove 原型的局限性：首先，它必须找到未加密的 TCP 通道，因为它是用户级别的攻击，无法在受害者进程内部获取机密。由于此限制，ShadowMove 无法劫持对其有效负载应用了用户级加密的连接。劫持加密连接的一种已知方法是将代码注入受害者进程，该进程将能够访问纯文本消息。不幸的是，过程注入会使 ShadowMove 对现有的检测工具（例如 Windows Defender ATP）更可见。此外，加密的存在对于 Shad owMove 可能并不总是一个障碍：有人提议在内核空间中实现加密服务（例如 TLS），这将使 TLS 会话容易受到 ShadowMove 的攻击，因为发送了未加密的有效负载部署此类内核级服务的系统中的套接字接口接收或接收。其次，如果客户端首先使用缓冲区，则 Shad owmMove 可能无法从接收缓冲区中获取 shellID 之类的信息。但是，攻击者可以简单地重试，并且只需成功一次即可实现横向移动。第三，在 Linux 上进行的 ShadowMove 设计将代码注入到目标进程的地址空间中，以劫持其控制流，与 Windows 相比，这危害了 ShadowMove 的隐蔽性。

通过套接字复制启用的其他攻击：机器内部应用程序（例如浏览器和后端密码管理器）之间的 TCP 通信并不完全安全。因此，本文的套接字复制技术可用于拦截和窃取此类应用程序中的敏感数据。此外，在本研究中，尝试主要利用客户端套接字（尽管利用了服务器端套接字来同步网络视图）。但是，可以使用相同的技术来利用服务器应用程序。例如，通过复制服务器应用程序使用的套接字，可以注入恶意数据以对客户端计算机发起网络钓鱼攻击。

  

  

0x07 Conclusion

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb7EBicjybYH8vOzDfMN2zoHASCSibIbfWkiacuFrsaibGwAcxjZXRnDLEJzfeIicVheR4ibgINaSRLciaVrg/640?wx_fmt=png)

本文提出了 ShadowMove 策略，该策略允许 APT 攻击者在企业网络内进行隐蔽横向移动。ShadowMove 以新颖的套接字复制技术为基础，利用了现有的良性网络连接，并且不需要任何提升的特权，新的连接，额外的身份验证或进程注入。因此，它能够逃避主机和网络级别防御机制的检测。为了确认方法的可行性，开发了 ShadowMovefor Windows 和 Linux OS 的现代版本的原型，该原型成功利用了三个通用企业协议（即 FTP，Microsoft SQL 和 WinRM）进行横向移动，例如将恶意软件上传到下一个目标计算机，并在下一个目标计算机上开始执行恶意软件。

文章描述了 ShadowMove 中的技术挑战，例如如何生成适合现有网络连接环境的网络数据包。还通过实验确认原型实现无法被最新的防病毒产品、IDS（例如 Snort）以及端点检测响应系统检测到。本研究为企业环境中的横向移动检测提高了标准，并呼吁采用创新的解决方案。

**译文声明**  

译文仅供参考，具体内容表达以及含义原文为准。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)

**戳 “阅读原文” 查看更多内容**