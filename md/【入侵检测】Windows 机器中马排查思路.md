> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Qibi3DO806fbqtsI5vZAlQ)

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

通用排查思路

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

1.1 **前提 & 思路**
---------------

问清楚谁在什么时间发现的主机异常情况，异常的现象是什么，受害用户做了什么样的紧急处理。问清楚主机异常情况后，需要动脑考虑为什么会产生某种异常，从现象反推可能的入侵思路，再考虑会在 Windows 主机上可能留下的痕迹，最后才是排除各种可能，确定入侵的过程。

1.2 **应急事件分类**
--------------

Windows 系统的应急事件，按照处理的方式，可分为下面几种类别：

病毒、木马、蠕虫事件

1.  Web 服务器入侵事件或第三方服务入侵事件

2. 系统入侵事件，如利用 Windows 的漏洞攻击入侵系统、利用弱口令入侵、利用其他服务的漏洞入侵，跟 Web 入侵有所区别，Web 入侵需要对 Web 日志进行分析，系统入侵只能查看 Windows 的事件日志。

3.  网络攻击事件（DDoS、ARP、DNS 劫持）等

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

文件排查

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

2.1 **P****ower****shell** **执行日志**
-----------------------------------

```
C:\Users\XX\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ ConsoleHost_history.txt Win10
```

 查看 PowerShell 的日志

```
Microsoft->Windows->PowerShell->OPtions
```

2.2 **开机启动有无异常文件**
------------------

```
【开始】➜【运行】➜【msconfig】➜【启动】
```

2.3 **temp 目录 &Recent 文件夹**
---------------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2HdeXfkKTZwpicwe5riaGpM6GQ52aA3Xguic1iaiakS5WCYu2mj57racBicPlbrH21Pz7p0Vyx1SU74MfVhkqhBGonTg/640?wx_fmt=png)

1、各个盘下的 temp(tmp) 相关目录下查看有无异常文件 ：Windows 产生的临时文件

2、Recent 是系统文件夹，里面存放着你最近使用的文档的快捷方式，查看用户 recent 相关文件，通过分析最近打开分析可疑文件：

【开始】➜【运行】➜【%UserProfile%\Recent】

2.4 **浏览器痕迹 - 文件 -cookie**
--------------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2HdeXfkKTZwpicwe5riaGpM6GQ52aA3Xguic1iaiakS5WCYu2mj57racBicPlbrH21Pz7p0Vyx1SU74MfVhkqhBGonTg/640?wx_fmt=png)

浏览器浏览痕迹、浏览器下载文件、浏览器 cookie 信息，根据不同浏览器进行排查；例如：IE 浏览器，打开 IE 浏览器点击【查看】➜【浏览器栏】➜【历史记录】（快捷键是 Ctrl+Shift+H）

2.5 **查找可疑文件**
--------------

1、需注意的文件
--------

```
下载目录
回收站文件
程序临时文件
历史文件记录
应用程序打开历史
搜索历史
快捷方式（LNK）
驱动
driverquery
进程 DLL 的关联查询
tasklist -M
共享文件
最近的文件（%UserProfile%\Recent）
文件更新
已安装文件
hklm:\software\Microsoft\Windows\CurrentVersion\Uninstall\
异常现象之前创建的文件
```

2、D 盾查杀 -19.4

3、时间线索

        查看文件时间，创建时间、修改时间、访问时间，黑客通过菜刀类工具改变的是修改时间。所以如果修改时间在创建时间之前明显是可疑文件

4、findstr 查找

例如：PHP webshell

```
findstr /m /i /s “eval” *.php  (注意字符串编码格式)
findstr    Window系统自带的命令，查找指定的一个或多个文件
参数说明：
/m 如果文件包含匹配项，则仅打印该文件名
/i      指定搜索不区分大小写
/s      在当前目录和所有子目录中搜索匹配的文件
/b      如果位于行的开头则匹配模式
/e      如果位于行的末尾则匹配模式
/x      打印完全匹配的行
/v      只打印不包含匹配的行
```

 Webshell 文件内容中常见的恶意函数：

```
PHP
Eval、System、assert、……
JSP
getRunTime、 FileOutputStream、……
ASP
eval、execute、 ExecuteGlobal、……
```

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

端口 & 进程排查

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

3.1 **网络连接**
------------

 netstat -anob 查看目前的网络连接，定位可疑的 ESTABLISHED
------------------------------------------

```
netstat 显示网络连接、路由表和网络接口信息；
参数说明：
-a      显示所有网络连接、路由表和网络接口信息
-n     以数字形式显示地址和端口号
-o      显示与每个连接相关的所属进程 ID
-r 显示路由表
-s 显示按协议统计信息、默认地、显示IP
-b ：显示在创建每个连接或侦听端口时涉及的可执行程序。在某些情况下，已知可执行程序承载多个独立的组件，这些情况下，显示创建连接或侦听端口时涉及的组件序列。在此情况下，可执行程序的名称位于底部 [] 中，它调用的组件位于顶部，直至达到 TCP/IP。注意，此选项可能很耗时，并且在你没有足够权限时可能失败。
```

常见的状态说明：

```
LISTENING      侦听状态
ESTABLISHED    建立连接
CLOSE_WAIT     对方主动关闭连接或网络异常导致连接中断
```

3.2 **进程排查**
------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2HdeXfkKTZwpicwe5riaGpM6GQ52aA3Xguic1iaiakS5WCYu2mj57racBicPlbrH21Pz7p0Vyx1SU74MfVhkqhBGonTg/640?wx_fmt=png)

1. tasklist | findstr 10744 

tasklist /svc 显示运行在本地或远程计算机上的所有进程；

根据 netstat 定位出的 pid，再通过 tasklist 命令进行进程定位

2. wmic process | findstr "chrome.exe"

使用 wmic 命令获取进程信息

wmic process | find "pid" > proc.csv

根据 wmic  process 获取进程的全路径 [任务管理器也可以定位到进程路径]

3.3 **PowerShell 相关**
---------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2HdeXfkKTZwpicwe5riaGpM6GQ52aA3Xguic1iaiakS5WCYu2mj57racBicPlbrH21Pz7p0Vyx1SU74MfVhkqhBGonTg/640?wx_fmt=png)

Get-WmiObject -Class Win32_Process

Get-WmiObject -Query  "select * from win32_service where -ComputerName Server01, Server02 | Format-List -Property PSComputerName, Name, ExitCode, Name, ProcessID, StartMode, State, Status

Get-Process

Get-NetTCPConnection

Get-NetTCPConnection -State Established

4 **系统信息排查**
============

4.1 **环境变量**
------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2HdeXfkKTZwpicwe5riaGpM6GQ52aA3Xguic1iaiakS5WCYu2mj57racBicPlbrH21Pz7p0Vyx1SU74MfVhkqhBGonTg/640?wx_fmt=png)

【我的电脑】➜【属性】➜【高级系统设置】➜【高级】➜【环境变量】

排查内容：temp 变量的所在位置的内容；后缀映射 PATHEXT 是否包含有非 windows 的后缀；有没有增加其他的路径到 PATH 变量中 (对用户变量和系统变量都要进行排查)；

4.2 **计算机详细信息**
---------------

```
msinfo32
```

4.3 **计算机管理**
-------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2HdeXfkKTZwpicwe5riaGpM6GQ52aA3Xguic1iaiakS5WCYu2mj57racBicPlbrH21Pz7p0Vyx1SU74MfVhkqhBGonTg/640?wx_fmt=png)

【开始】➜【运行】➜【compmgmt.msc】 - 计划 / 事件 / 用户 / 共享 / 性能 / 设备 / 存储磁盘 / 服务综合排查

4.4 **计划任务**
------------

【程序】➜【附件】➜【系统工具】➜【任务计划程序】

存放计划任务的文件

```
C:\Windows\System32\Tasks\
C:\Windows\SysWOW64\Tasks\
C:\Windows\tasks\
*.job（指文件）
```

使用命令查看计划任务 cmd 运行 schtasks

4.5 **Windows 用户**
------------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2HdeXfkKTZwpicwe5riaGpM6GQ52aA3Xguic1iaiakS5WCYu2mj57racBicPlbrH21Pz7p0Vyx1SU74MfVhkqhBGonTg/640?wx_fmt=png)

【开始】➜【运行】➜【lusrmgr.msc】 (用户名以 $ 结尾的为隐藏用户，如：admin$)

1、net user admin$

2、检查注册表

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList，HKLM\SAM\Domains\Account\
```

（默认是 SYSTEM）权限，需要配置成管理员权限查看。  

3、wmic UserAccount get

query user 查看当前系统得会话，查看是否有人使用远程终端登录服务器，logoff 提出该用户

4.6 **事件查看器**
-------------

```
【开始】➜【运行】➜【eventvwr.msc】
```

4.7 **systeminfo 系统版本以及补丁信息**
-----------------------------

对比漏洞补丁

```
https://github.com/neargle/win-powerup-exp-index
```

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

应急检测表

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

<table cellspacing="0"><tbody><tr><td width="87" valign="center"><p>检测项目</p></td><td width="417" valign="center"><p>检&nbsp;&nbsp;&nbsp;测 &nbsp;&nbsp;细&nbsp;&nbsp;&nbsp;节</p></td><td width="66" valign="center"><br></td></tr><tr><td width="87" valign="center"><p>服务器&nbsp;&nbsp;&nbsp;基本信息</p></td><td width="417" valign="center"><p><strong>记录服务器系统、</strong><strong>W</strong><strong>eb 开发语言、中间件（容器、通用系统（框架）、数据库、对外开发的服务等信息</strong></p></td><td width="66" valign="center"><br></td></tr><tr><td width="87" valign="center" rowspan="7"><p>Web 服务功能检测</p></td><td width="417" valign="center"><p>1.&nbsp;检查 Web 目录最近 1-30 天新增的文件</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>是否发现可疑文件</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p><strong>记录可疑文件新增、修改、访问时间</strong></p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>搜集存在时长最久的可疑文件，往该文件时间前后检查 1-30 天的新增文件，并记录</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>2.&nbsp;检查 Web 后台是否对外访问</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>3.&nbsp;后台管理员帐号是否使用弱口令</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>是否利用后台上传可疑文件</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="87" valign="center" rowspan="16"><p>服务器检测</p></td><td width="417" valign="center"><p>&nbsp;&nbsp;1.&nbsp;检查服务器日志（Web 日志、数据库日志、服务器日志、安全设备日志等）</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p><strong>日志是否保留超过 6 个月</strong></p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p><strong>记录服务器日志</strong></p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>&nbsp;&nbsp;2.&nbsp;检查服务项</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>是否发现可疑进程</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>3.&nbsp;检查网络连接</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p><strong>是否发现可疑端口、可疑 IP、PID 及程序进程</strong></p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>4.&nbsp;分析系统信息</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>记录开机启动项、系统变量、计划任务</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>5.&nbsp;检查服务器是否被提权</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>6.&nbsp;检查用户相关分析</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>分析管理组与远程连接组用户</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>记录可疑用户</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>记录用户最后一次登录时间</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>记录用户最近登录信息</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>记录能够登陆的帐号</p></td><td width="66" valign="center"><br></td></tr><tr><td width="87" valign="center" rowspan="6"><p>数据库检测</p></td><td width="417" valign="center"><p>1.&nbsp;数据库日志是否开启</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>2.&nbsp;<strong>数据库是否被脱库</strong></p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>&nbsp;&nbsp;3.&nbsp;检查数据库是否能任意写入文件</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>4.&nbsp;检查数据库是否被利用（sa、UDF、MOF）</p></td><td width="66" valign="center"><p>□</p></td></tr><tr><td width="417" valign="center"><p>5&nbsp;记录当前数据库使用账户</p></td><td width="66" valign="center"><br></td></tr><tr><td width="417" valign="center"><p>&nbsp;&nbsp;6.&nbsp;记录当前数据库使用版本</p></td><td width="66" valign="center"><br></td></tr><tr><td width="571" valign="top" colspan="3"><br></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

工具

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

Ø PC Hunter 是一个 Windows 系统信息查看软件

```
下载地址：http://www.xuetr.com/
```

Ø ProcessExplorer, 一款 Windows 系统和应用程序监视工具, 超级任务管理器

```
下载链接：http://www.baidu.com
```

Ø 卡巴斯基、赛门铁克，不多说，下载了开杀就行

Ø Webshell 查杀工具、D 盾 / 河马等直接扫描

Ø Log Parser (Lizard) 快速日志分析工具

        Log Parser 是一个用于日志文件分析的使用简单又强大的工具，多用于分析 IIS web 服务器的日志，支持基于文本的日志文件、XML 文件、CSV（逗号分隔符）文件以及注册表、文件系统等内容

```
下载地址：
https://www.microsoft.com/en-us/download/confirmation.aspx?id=24659
使用：
LogParser.exe "select top 10 time, c-ip,cs-uri-stem, sc-status, time-taken from C:\Users\liuhao02\Desktop\排查\样本\iis.log" -o:datagrid
```

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/nMQkaGYuOibDavXvuud5F09Tjl7NMvU8Yzhia63knJ4QJFvO4WBfd6KQazjtuPC7uqNBt5gE06ia7GjOVn2RFOicNA/640?wx_fmt=jpeg)

扫取二维码获取

更多精彩

![](https://mmbiz.qpic.cn/mmbiz_png/TlgiajQKAFPtOYY6tXbF7PrWicaKzENbNF71FLc4vO5nrH2oxBYwErfAHKg2fD520niaCfYbRnPU6teczcpiaH5DKA/640?wx_fmt=png)

Qingy 之安全  

![](https://mmbiz.qpic.cn/mmbiz_png/Y8TRQVNlpCW6icC4vu5Pl5JWXPyWdYvGAyfVstVJJvibaT4gWn3Mc0yqMQtWpmzrxibqciazAr5Yuibwib5wILBINfuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3pKe8enqDsSibzOy1GzZBhppv9xkibfYXeOiaiaA8qRV6QNITSsAebXibwSVQnwRib6a2T4M8Xfn3MTwTv1PNnsWKoaw/640?wx_fmt=png)

点个在看你最好看