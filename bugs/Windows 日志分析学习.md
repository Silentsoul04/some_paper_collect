> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sah3GAVlOALP4hx7vk8eJA)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/1brjUjbpg5zGqnux86icY3iaSADXAreXiaJEIOOPug2W5Rich6Cst24vCeB65NNkoxowMh4uZIcwoSUKENv1mFW3ww/640?wx_fmt=png)

点击上方 “信安前线” 可订阅  

0x01 Windows 事件日志简介

Windows 系统日志是记录系统中硬件、软件和系统问题的信息，同时还可以监视系统中发生的事件。用户可以通过它来检查错误发生的原因，或者寻找受到攻击时攻击者留下的痕迹。

Windows 日志类别包括以下在早期版本的 Windows 中可用的日志：应用程序、安全和系统日志。此外还包括两个新的日志：安装程序日志和 ForwardedEvents 日志。Windows 日志用于存储来自旧版应用程序的事件以及适用于整个系统的事件。

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7ia5eUepia9MMTq4kPoPbxTBItPqpzgibqgNCDyticj2UJwticf2TECcRgjw/640?wx_fmt=png)

**系统日志**

```
 系统日志包含 Windows 系统组件记录的事件。例如，在启动过程中加载驱动程序或其他系统组件失败将记录在系统日志中。系统组件所记录的事件类型由 Windows 预先确定。
 
 默认位置：%SystemRoot%\System32\Winevt\Logs\System.evtx
```

**应用程序日志**

```
 应用程序日志包含由应用程序或程序记录的事件。例如，数据库程序可在应用程序日志中记录文件错误。程序开发人员决定记录哪些事件。
 
 默认位置：%SystemRoot%\System32\Winevt\Logs\Application.evtx
```

**安全日志**

```
 安全日志包含诸如有效和无效的登录尝试等事件，以及与资源使用相关的事件，如创建、打开或删除文件或其他对象。管理员可以指定在安全日志中记录什么事件。例如，如果已启用登录审核，则对系统的登录尝试将记录在安全日志中。
 
 默认位置：%SystemRoot%\System32\Winevt\Logs\Security.evtx
```

系统和应用程序日志存储着故障排除信息，对于系统管理员更为有用。安全日志记录着事件审计信息，包括用户验证（登录、远程访问等）和特定用户在认证后对系统做了什么，对于调查人员而言，更有帮助。

### 0X02 审核策略与事件查看器

Windows Server 2008 R2 系统的审核功能在默认状态下并没有启用 ，建议开启审核策略，若日后系统出现故障、安全事故则可以查看系统的日志文件，排除故障，追查入侵者的信息等。

PS：默认状态下，也会记录一些简单的日志，日志默认大小 20M

**设置 1**："**Win+R**"→"**secpol.msc**"→" **本地安全策略** "→" **本地策略** "→" **审核策略** "，参考配置操作：

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7oFAudMUFK2jy0f4Ricic41uSqgfDBmOoZ5mZEznyIaTOlzA5iaALgxHLQ/640?wx_fmt=png)

**设置 2**：设置合理的日志属性，即日志最大大小、事件覆盖阀值等：  

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7yYia3aoDjGKSENcZExuRTTN86X1rUXia6Xo21Xe3KA6hAFBRDBGXN8TQ/640?wx_fmt=png)

**查看系统日志方法：**  

1.  按 "**Window+R**"，输入 ”**eventvwr.msc**“
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7npftaV0Sq6IAU0wSCEQXlUWiaXEIplYbYbJW3QTlAegO7LAZV1CzO2A/640?wx_fmt=png)

### 0x03 事件日志基础知识  

对于 Windows 事件日志而言，首先是学习事件日志的属性。Windows **事件日志属性**如下：

<table width="800"><thead><tr cid="n28" mdtype="table_row"><th>属性名</th><th>描述</th></tr></thead><tbody><tr cid="n31" mdtype="table_row"><td>事件 ID</td><td>标识特定事件类型的编号。描述的第一行通常包含事件类型的名称。例如，6005 是在启动事件日志服务时所发生事件的 ID。此类事件的描述的第一行是 “事件日志服务已启动”。产品支持代表可以使用事件 ID 和来源来解决系统问题。</td></tr><tr cid="n34" mdtype="table_row"><td>来源</td><td>记录事件的软件，可以是程序名（如 “SQL Server”），也可以是系统或大型程序的组件（如驱动程序名）。例如，“Elnkii” 表示 EtherLink II 驱动程序。</td></tr><tr cid="n37" mdtype="table_row"><td>级别</td><td>事件严重性的分类，以下事件严重性级别可能出现在系统和应用程序日志中：<br><strong>信息：</strong>指明应用程序或组件发生了更改，如操作成功完成、已创建了资源，或已启动了服务。<br><strong>警告：</strong>指明出现的问题可能会影响服务器或导致更严重的问题（如果未采取措施）。<br><strong>错误：</strong>指明出现了问题，这可能会影响触发事件的应用程序或组件外部的功能。<br><strong>关键：</strong>指明出现了故障，导致触发事件的应用程序或组件可能无法自动恢复。以下事件严重性级别可能出现在安全日志中：<br><strong>审核成功 ：</strong>指明用户权限练习成功。<br><strong>审核失败：</strong>指明用户权限练习失败。在事件查看器的正常列表视图中，这些分类都由符号表示。</td></tr><tr cid="n40" mdtype="table_row"><td>用户</td><td>事件发生所代表的用户的名称。如果事件实际上是由服务器进程所引起的，则此名称为客户端 ID；如果没有发生模仿的情况，则为主 ID。如果适用，安全日志项同时包含主 ID 和模仿 ID。当服务器允许一个进程采用另一个进程的安全属性时就会发生模拟的情况</td></tr><tr cid="n43" mdtype="table_row"><td>操作代码</td><td>包含标识活动或应用程序引起事件时正在执行的活动中的点的数字值。例如，初始化或关闭</td></tr><tr cid="n46" mdtype="table_row"><td>日志</td><td>已记录事件的日志的名称</td></tr><tr cid="n49" mdtype="table_row"><td>任务类别</td><td>用于表示事件发行者的子组件或活动。</td></tr><tr cid="n52" mdtype="table_row"><td>关键字</td><td>可用于筛选或搜索事件的一组类别或标记。示例包括 “网络”、“安全” 或“未找到资源”</td></tr><tr cid="n55" mdtype="table_row"><td>计算机</td><td>发生事件的计算机的名称。该计算机名称通常为本地计算机的名称，但是它可能是已转发事件的计算机的名称，或者可能是名称更改之前的本地计算机的名称</td></tr><tr cid="n58" mdtype="table_row"><td>日期和时间</td><td>记录事件的日期和时间</td></tr></tbody></table>

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7qmIffsVvXT4Vp97cU53hiaE58VCzX7kCZjB2DPVhpWAEEubFibmRLYibQ/640?wx_fmt=png)

以下重点讲述**事件 ID** 值，Windows 的日志以事件 id 来标识具体发生的动作行为，可通过下列网站查询具体 id 对应的操作：  

<table width="800"><thead><tr cid="n64" mdtype="table_row"><th>事件 ID</th><th>说明</th></tr></thead><tbody><tr cid="n67" mdtype="table_row"><td>1102</td><td>清理审计日志</td></tr><tr cid="n70" mdtype="table_row"><td>4624</td><td>账号登录成功</td></tr><tr cid="n73" mdtype="table_row"><td>4625</td><td>账号登录失败</td></tr><tr cid="n76" mdtype="table_row"><td>4634</td><td>账号注销成功</td></tr><tr cid="n79" mdtype="table_row"><td>4647</td><td>用户启动的注销</td></tr><tr cid="n82" mdtype="table_row"><td>4672</td><td>使用超级用户（如管理员）进行登录</td></tr><tr cid="n85" mdtype="table_row"><td>4720</td><td>创建用户</td></tr><tr cid="n88" mdtype="table_row"><td>4726</td><td>删除用户</td></tr><tr cid="n91" mdtype="table_row"><td>4732</td><td>将成员添加到启用安全的本地组中</td></tr><tr cid="n94" mdtype="table_row"><td>4733</td><td>将成员从启用安全的本地组中移除</td></tr><tr cid="n97" mdtype="table_row"><td>4688</td><td>创建新进程</td></tr><tr cid="n100" mdtype="table_row"><td>4689</td><td>结束进程</td></tr><tr cid="n170" mdtype="table_row"><td>……</td><td>……</td></tr></tbody></table>

每个成功登录的事件都会标记一个登录类型，不同登录类型代表不同的方式：

<table width="800"><thead><tr cid="n204" mdtype="table_row"><th>登录类型</th><th>描述</th><th>说明</th></tr></thead><tbody><tr cid="n208" mdtype="table_row"><td>2</td><td>交互式登录（Interactive）</td><td>用户在本地进行登录。</td></tr><tr cid="n212" mdtype="table_row"><td>3</td><td>网络（Network）</td><td>最常见的情况就是连接到共享文件夹或共享打印机时。</td></tr><tr cid="n216" mdtype="table_row"><td>4</td><td>批处理（Batch）</td><td>通常表明某计划任务启动。</td></tr><tr cid="n220" mdtype="table_row"><td>5</td><td>服务（Service）</td><td>每种服务都被配置在某个特定的用户账号下运行。</td></tr><tr cid="n224" mdtype="table_row"><td>7</td><td>解锁（Unlock）</td><td>屏保解锁。</td></tr><tr cid="n228" mdtype="table_row"><td>8</td><td>网络明文（NetworkCleartext）</td><td>登录的密码在网络上是通过明文传输的，如 FTP。</td></tr><tr cid="n232" mdtype="table_row"><td>9</td><td>新凭证（NewCredentials）</td><td>使用带 / Netonly 参数的 RUNAS 命令运行一个程序。</td></tr><tr cid="n236" mdtype="table_row"><td>10</td><td>远程交互，（RemoteInteractive）</td><td>通过终端服务、远程桌面或远程协助访问计算机。</td></tr><tr cid="n240" mdtype="table_row"><td>11</td><td>缓存交互（CachedInteractive）</td><td>以一个域用户登录而又没有域控制器可用</td></tr></tbody></table>

关于更多 EVENT ID，详见以下说明链接：

> 参考链接 ：https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx?i=j

### 0x04 日志分析案例

#### 1、RDP 登陆爆破定位

##### 1.1、自带筛选器分析

首先确认账号登陆失败事件 ID 值为 **4625**，选择 "Windows 日志" → " 安全 **"→"** 筛选器 " ，在筛选器中输入 ID 值 。

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7KhJZVxk9ol03JUsibzcr3LBZVW0gT3vlLUI8dKWibEiblicbWquXLMDR5g/640?wx_fmt=png)

筛选出有 96 个登陆失败事件，仔细查看这 96 个事件发现有 ip 在 2021/2/3 上午 11:24 对服务器大量登陆且登陆失败，推测处在进行 rdp 爆破

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7vvODicvtq4fwKb7tS1hzqPWxPK6SHVos7RibPGxF7AtRUaw7HAiaFS63A/640?wx_fmt=png)

双击任一审核失败事件进行查看  

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7yKMODd6l3qzmDSq2NKshY6GhqDWfNAFwqau6cnibSEkiaxAibiapu9GzgA/640?wx_fmt=png)

查看多个审核失败事件发现客户端地址为 10.0.0.1 对服务器进行爆破，接下来搜索 **4624** 审核成功的事件  

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7GFE82pjAUYth5oXf3c05ZOibe5Osop5HqU7HLF6Rl92iaEgzKqnZxnhg/640?wx_fmt=png)

再查找关键词 "10.0.0.1"  

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7WtGUQPjM8g0YqK3s9snrvGhDb5amkd8KIfZQVB8L4nXeZfuWR84CVA/640?wx_fmt=png)

发现 2021/2/3 上午 11:26 客户段 IP 成功登陆服务器。由此推断 IP 地址为 10.0.0.1 的 hacker 在 2021/2/3 上午 11:24 对服务器 RDP 远程桌面进行爆破在 11:32 爆破成功！！！  

##### 1.2、使用 Log Parser 分析

Log Parser 是一款功能强大的多功能工具，可提供对基于文本的数据（例如日志文件，XML 文件和 CSV 文件）以及 Windows 操作系统上的关键数据源（例如事件日志，注册表， 文件系统和 ActiveDirectory）的查询以及输出。

**基本查询结构**

```
 Logparser.exe –i:EVT –o:DATAGRID "SELECT * FROM c:\xx.evtx"
```

**使用 Log Parser 分析日志**

1、查询登录失败的事件

```
 登录失败的所有事件：
 LogParser.exe -i:EVT –o:DATAGRID "SELECT * FROM c:\Security.evtx where EventID=4625"
 
 提取登录失败用户名进行聚合统计：
 LogParser.exe -i:EVT "SELECT EXTRACT_TOKEN(Message,13,' ') as EventType,EXTRACT_TOKEN(Message,19,' ') as user,count(EXTRACT_TOKEN(Message,19,' ')) as Times,EXTRACT_TOKEN(Message,39,' ') as Loginip FROM c:\Security.evtx where EventID=4625 GROUP BY Message"
 
```

登陆失败事件查看

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7897JRibWZWsrfU7gvt0k4TcXNw9jPZCsxLTBSwooCroZMDBzU0JaxIQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7iceSm4285J6MmnFk01icicP1Eur8Tic9jQllFhceQnSbqvGvyD6hMzibtmQ/640?wx_fmt=png)

发现有大量 10.0.0.1IP 地址登陆失败，可推出 10.0.0.1 的 IP 地址一直在对服务器进行 rdp 爆破登陆  

2、查询登录成功的事件

```
 登录成功的所有事件
 LogParser.exe -i:EVT –o:DATAGRID "SELECT * FROM c:\Security.evtx where EventID=4624"
 
 指定登录时间范围的事件：
 LogParser.exe -i:EVT –o:DATAGRID "SELECT * FROM c:\Security.evtx where TimeGenerated>'2018-06-19 23:32:11' and TimeGenerated<'2018-06-20 23:34:00' and EventID=4624"
 
 提取登录成功的用户名和IP：
 LogParser.exe -i:EVT –o:DATAGRID "SELECT EXTRACT_TOKEN(Message,13,' ') as EventType,TimeGenerated as LoginTime,EXTRACT_TOKEN(Strings,5,'|') as Username,EXTRACT_TOKEN(Message,38,' ') as Loginip FROM c:\Security.evtx where EventID=4624"
 
```

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7JJoo94icQU8E2ZgicKbiagjtibzerqtaOqDm7hDQocPv02icfxBmyypIHIg/640?wx_fmt=png)

发现 2021/2/3 上午 11:26 客户段 IP 成功登陆服务器。

#### 2、本地权限提升

**前提预知：系统管理员以普通用户 admin 权限运行 apahce 中间件**

1、hacker 在获取到 webshell 后，通常思维为查看当前权限再对此服务器进行信息收集

```
 whoami
 systeminfo
 net user
 net1 user
```

Windows 日志中可以开启审核进程跟踪对执行的命令进行追踪，其中创建新进程事件 ID 为 **4688**

```
 C:\Program Files\Log Parser 2.2>LogParser.exe -i:EVT "SELECT TimeGenerated,EventID,EXTRACT_TOKEN(Strings,1,'|') as UserName,EXTRACT_TOKEN(Strings,5,'|') as ProcessName FROM c:\security.evtx where EventID=4688" > 11.txt
```

查询结果

```
 TimeGenerated       EventID UserName         ProcessName
 ------------------- ------- ---------------- ----------------------------------------
 2021-02-03 11:26:03 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\winlogon.exe
 2021-02-03 11:26:03 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\LogonUI.exe
 2021-02-03 11:26:03 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\TSTheme.exe
 2021-02-03 11:26:04 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\wbem\WMIADAP.exe
 2021-02-03 11:26:38 4688   Administrator   C:\Windows\System32\verclsid.exe
 2021-02-03 11:26:39 4688   Administrator   C:\Program Files\Notepad++\notepad++.exe
 2021-02-03 11:26:44 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\wuauclt.exe
 2021-02-03 11:27:16 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:27:16 4688    admin            C:\Windows\System32\whoami.exe
 2021-02-03 11:27:19 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:27:19 4688    admin            C:\Windows\System32\whoami.exe
 2021-02-03 11:27:31 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\svchost.exe
 2021-02-03 11:27:38 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:38 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:38 4688   admin           C:\Windows\System32\whoami.exe
 2021-02-03 11:27:45 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:45 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:45 4688   admin           C:\Windows\System32\net.exe
 2021-02-03 11:27:45 4688   admin           C:\Windows\System32\net1.exe
 2021-02-03 11:27:50 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:50 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:50 4688   admin           C:\Windows\System32\net.exe
 2021-02-03 11:27:50 4688   admin           C:\Windows\System32\net1.exe
 2021-02-03 11:27:58 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:58 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:27:58 4688   admin           C:\Windows\System32\systeminfo.exe
 2021-02-03 11:27:58 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\wbem\WmiPrvSE.exe
 2021-02-03 11:27:58 4688    WIN-4HO1USO2OUA$ C:\Windows\servicing\TrustedInstaller.exe
 2021-02-03 11:29:57 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:29:57 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:29:59 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:29:59 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:09 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:09 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:09 4688   admin           C:\phpStudy\ms16-032.exe
 2021-02-03 11:30:15 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:15 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:15 4688   admin           C:\phpStudy\ms16-032.exe
 2021-02-03 11:30:15 4688   WIN-4HO1USO2OUA$ C:\phpStudy\ms16-032.exe
 2021-02-03 11:30:15 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\whoami.exe
 2021-02-03 11:30:35 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:35 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:35 4688   admin           C:\phpStudy\ms16-032.exe
 2021-02-03 11:30:35 4688   WIN-4HO1USO2OUA$ C:\phpStudy\ms16-032.exe
 2021-02-03 11:30:35 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\net.exe
 2021-02-03 11:30:35 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\net1.exe
 2021-02-03 11:30:50 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:30:50 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:30:50 4688    admin            C:\phpStudy\ms16-032.exe
 2021-02-03 11:30:50 4688    WIN-4HO1USO2OUA$ C:\phpStudy\ms16-032.exe
 2021-02-03 11:30:50 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\net.exe
 2021-02-03 11:30:50 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\net1.exe
 2021-02-03 11:30:56 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:57 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:30:57 4688   admin           C:\Windows\System32\net.exe
 2021-02-03 11:30:57 4688   admin           C:\Windows\System32\net1.exe
 2021-02-03 11:31:08 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:31:08 4688   admin           C:\Windows\System32\cmd.exe
 2021-02-03 11:31:08 4688   admin           C:\phpStudy\ms16-032.exe
 2021-02-03 11:31:08 4688   WIN-4HO1USO2OUA$ C:\phpStudy\ms16-032.exe
 2021-02-03 11:31:08 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\net.exe
 2021-02-03 11:31:08 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\net1.exe
 2021-02-03 11:31:16 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:31:16 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:31:16 4688    admin            C:\Windows\System32\net.exe
 2021-02-03 11:31:16 4688    admin            C:\Windows\System32\net1.exe
 2021-02-03 11:31:21 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:31:21 4688    admin            C:\Windows\System32\cmd.exe
 2021-02-03 11:31:21 4688    admin            C:\Windows\System32\net.exe
 2021-02-03 11:31:21 4688    admin            C:\Windows\System32\net1.exe
 2021-02-03 11:31:46 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\smss.exe
 2021-02-03 11:31:46 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\csrss.exe
 2021-02-03 11:31:46 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\winlogon.exe
 2021-02-03 11:31:46 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\LogonUI.exe
 2021-02-03 11:31:47 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\wsqmcons.exe
 2021-02-03 11:31:47 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\schtasks.exe
 2021-02-03 11:32:11 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\dllhost.exe
 2021-02-03 11:32:11 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\TSTheme.exe
 2021-02-03 11:32:11 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\taskeng.exe
 2021-02-03 11:32:11 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\rdpclip.exe
 2021-02-03 11:32:11 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\userinit.exe
 2021-02-03 11:32:11 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\dwm.exe
 2021-02-03 11:32:11 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\taskeng.exe
 2021-02-03 11:32:11 4688   admin$           C:\Windows\System32\ServerManagerLauncher.exe
 2021-02-03 11:32:11 4688    admin$           C:\Windows\explorer.exe
 2021-02-03 11:32:11 4688   admin$           C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:11 4688    admin$           C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:11 4688   admin$           C:\Windows\System32\ie4uinit.exe
 2021-02-03 11:32:12 4688    admin$           C:\Windows\System32\regsvr32.exe
 2021-02-03 11:32:12 4688   admin$           C:\Windows\System32\regsvr32.exe
 2021-02-03 11:32:12 4688    admin$   C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:12 4688   admin$   C:\Windows\System32\rundll32.exe
 2021-02-03 11:32:12 4688    admin$   C:\Windows\System32\ie4uinit.exe
 2021-02-03 11:32:12 4688   admin$   C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:12 4688    admin$   C:\Windows\System32\rundll32.exe
 2021-02-03 11:32:12 4688   admin$   C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:12 4688    admin$   C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:12 4688   admin$   C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:13 4688    admin$   C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:13 4688   admin$   C:\Windows\System32\vm3dservice.exe
 2021-02-03 11:32:13 4688    admin$           C:\Program Files\VMware\VMware Tools\vmtoolsd.exe
 2021-02-03 11:32:13 4688   admin$           C:\Program Files\Common Files\Java\Java Update\jusched.exe
 2021-02-03 11:32:13 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\dllhost.exe
 2021-02-03 11:32:13 4688   admin$           C:\Windows\System32\verclsid.exe
 2021-02-03 11:32:18 4688    admin$           C:\Windows\System32\mmc.exe
 2021-02-03 11:32:19 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\TSTheme.exe
 2021-02-03 11:32:34 4688    WIN-4HO1USO2OUA$ C:\Windows\System32\LogonUI.exe
 2021-02-03 11:32:35 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\LogonUI.exe
 2021-02-03 11:32:35 4688    WIN-4HO1USO2OUA$ C:\Program Files\VMware\VMware Tools\VMwareResolutionSet.exe
 2021-02-07 23:43:45 4688   WIN-4HO1USO2OUA$ C:\Windows\System32\svchost.exe
 
```

查看到在 2021-02-03 11:27:16 时间 admin 权限执行了 whoami.exe、systeminfo.exe 等命令

2、根据系统信息进行权限提升

```
 C:\phpStudy\ms16-032.exe net.exe
 C:\phpStudy\ms16-032.exe net1.exe
 
```

Windows 常识查看用户安全标识符（SID）

```
 wmic useraccount get name,sid
 
```

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7beeIIicm7adibGPpPvQB7DbgQsZDDxg2zL74AsWHbDiaCl5edPUbIn3dQ/640?wx_fmt=png)

查看创建用户动作事件 ID 4720，将成员添加到启用安全的本地组中动作事件 ID 4733  

```
 登录成功的所有事件
 LogParser.exe -i:EVT –o:DATAGRID "SELECT * FROM c:\Security.evtx where EventID=4720"
 LogParser.exe -i:EVT –o:DATAGRID "SELECT * FROM c:\Security.evtx where EventID=4732"
```

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn77O0w2RcTNGicic3kPeIQje57iaL90oxL6qiaeD2Gk0JGbqCBT2KPH4J3UA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7cEh8w7iadDyPO8vyOXZT1fC4UpbmknOv8KUjiaWu1CTCkIzHCsRicYjTA/640?wx_fmt=png)

其中通过进程创建账户 admin$ 影藏账户，在添加账户的同时该账户自动加入 users 组

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7vshib49HlmPw3xic8ekEOUPEwRHN1x4Qry0OicPicg81ghibIdH3o9aGm8Q/640?wx_fmt=png)

再将 admin$ 用户组加入 Administrators 管理员组中  

3、使用影藏账户登陆服务器

```
 提取登录成功的用户名和IP：
 LogParser.exe -i:EVT –o:DATAGRID "SELECT EXTRACT_TOKEN(Message,13,' ') as EventType,TimeGenerated as LoginTime,EXTRACT_TOKEN(Strings,5,'|') as Username,EXTRACT_TOKEN(Message,38,' ') as Loginip FROM c:\Security.evtx where EventID=4624"
 
```

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTcRqriblV01qnuAH2xgnpn7ShWu7O2e7zqAcJBDicSonh8rpiaPcibvzpbGo5msSXAzQRm6YCY0WBM8Q/640?wx_fmt=png)

发现 hacker 在 2021-02-03 11:32 登陆系统！！！当然 hacker 已经获得系统 system 权限完全可以伪造、删除日志等操作，但在等保 2.0 的要求下都要求有日志审计系统的……  

 PS：比较基础，对学习 Windows 日志分析还是很有帮组的……

更多精彩

*   [Web 渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [安全基线加固课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486688&idx=2&sn=80a4c340a84948a14807e60d2341051a&chksm=fcfc31b6cb8bb8a099ebf4d9daf3136e394bd0ac8225b3603a9989dab4f984429e032d4ee143&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486688&idx=2&sn=80a4c340a84948a14807e60d2341051a&chksm=fcfc31b6cb8bb8a099ebf4d9daf3136e394bd0ac8225b3603a9989dab4f984429e032d4ee143&scene=21#wechat_redirect)[商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSd9wDlUiar0tUpHCYAzrZfTzOvS2SEw9cia9j7d1HKP2bWArPLCegs1XoejVUPu0GkSuZh7Wia7aExA/640?wx_fmt=png)

**如果感觉文章不错，分享让更多的人知道吧！**