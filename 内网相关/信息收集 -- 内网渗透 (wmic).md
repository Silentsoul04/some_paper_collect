> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/8ljbXmCOXJuysqo4cUZkfQ)

昨天晚上和舍友聊天，彼此聊着自己的前女友，聊到最后，我们同时对彼此说了一句，你真贱。。。

----  网易云热评

一、WMIC 简介

WMIC 是扩展 WMI（Windows Management Instrumentation，Windows 管理规范），提供了从命令行接口和批命令脚本执行系统管理的支持。在 WMIC 出现之前，如果要管理 WMI 系统，必须使用一些专门的 WMI 应用，比如 SMS，或者使用 WMI 的脚本编程 API，或者使用象 CIM Studio 之类的工具。如果不熟悉 C++ 之类的 编程语言或 VBScript 之类的 脚本语言，或者不掌握 WMI 名称空间的基本知识，要使用 WMI 管理系统是很困难的。WMIC 改变了这种情况，为 WMI 名称空间提供了一个强大的、友好的命令行接口。

二、使用方法

1、第一次执行 WMIC 命令时，Windows 首先要安装 WMIC，然后显示出 WMIC 的命令行提示符。在 WMIC 命令行提示符上，命令以交互的方式执行

2、进入 wmic 并查看帮助信息

wmic

/?

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqV5cNexv8qV93YVTaEv6IpcnmPTt84K6IHb23jK6EpUmicjMiaibP5ibD2rA/640?wx_fmt=png)

3、中文翻译：

/NAMESPACE 别名操作的命名空间的路径。 

/ROLE 包含别名定义的角色的路径。 

/NODE 别名将对其进行操作的服务器。 

/IMPLEVEL 客户端模拟级别。 

/AUTHLEVEL 客户端身份验证级别。 

/LOCALE 客户端应使用的语言 ID。 

/PRIVILEGES 启用或禁用所有权限。 

/TRACE 将调试信息输出到 stderr。 

/RECORD 记录所有输入命令和输出。

 /INTERACTIVE 设置或重置交互模式。

/FAILFAST 设置或重置 FailFast 模式。

 /USER 在会话期间使用的用户。 

/PASSWORD 用于会话登录的密码。 

/OUTPUT 指定输出重定向的模式。 

/APPEND 指定输出重定向的模式。 

/AGGREGATE 设置或重置聚合模式。 

/AUTHORITY 指定连接的 <authority type>。 

4、查看进程管理的帮助

process /?

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqV8RR4MhSB5yXP6pic5ebd909kSk9BGqXicVnyRfpjziaNLYRib6sPFYrkmQ/640?wx_fmt=png)

5、查看进程的详细信息

process where list full

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqV0NYRJVw1BpCHFfsZexI9nVicBM2icpTLzWqA2UE9CxEoNTMG17wa0d1w/640?wx_fmt=png)

6、获取当前正在运行的进程、进程 id，进程路径

process get name,processid,executablepath

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqVqxghQXRaCT0RJLl0f33SCGegicD3tnuuEq6nqp45DFCAWBsPzAHDtxQ/640?wx_fmt=png)

7、获取进程的核心信息

process list brief

8、结束 svchost.exe 进程, 路径为非 C:\WINDOWS\system32\svchost.exe 的

process where "name='svchost.exe'and ExecutablePath<>'C:\\WINDOWS\\system32\\svchost.exe'" call Terminate

9、打开计算器进程

process call create calc

10、获取运行状态的服务，进程 ID、路径

service where (state="running") get name,processid,executablepath

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqV4BicYULQZa46mxPyMJ019PN2eaqshvw1OyYqJ3eoCcDKNRVc6E1Oia4A/640?wx_fmt=png)

11、运行 spooler 服务

SERVICE where call startservice

12、停止 spooler 服务

SERVICE where call stopservice

13、暂停 spooler 服务

SERVICE where call PauseService

14、更改 spooler 服务启动类型 [auto|Disabled|Manual] [自动 | 禁用 | 手动]

SERVICE where

15、删除服务

SERVICE where call delete

16、获取安装的软件

product get name

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqVfNgMmDaS31Dean9xbSDUibRlsgsHjk7rJSQicQ5rLqGvbKqegv1zMDFg/640?wx_fmt=png)

安装包在 C:\WINDOWS\Installer 目录下

卸载. msi 安装包

wmic PRODUCT where " call Uninstall

修复. msi 安装包

wmic PRODUCT where " call Reinstall

禁止非法，后果自负

欢迎关注公众号：web 安全工具库

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqVw7LYvmSZgoCicJIQwzgBiavmmgTYUibSHiaNdbgTvMsEW1gpFjFjmrQGkQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibuyH6RHX1nvZd9eVcy2BvqVhqpeY3ic8Ixx6bG0JzxyBs5eoL9wsZfLWLv5skxE1cEVQMDCSI5pVjg/640?wx_fmt=jpeg)