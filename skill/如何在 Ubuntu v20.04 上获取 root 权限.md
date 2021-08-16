\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/wE1haq\_UZQ2DXtkt4Rg2tg)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fqlALPKqMmvfoofjDUs2cgZBwiaibPuv9FlgAYDycIahnLX6LpsrwibLTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

写在前面的话

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

我个人非常喜欢 Ubuntu，所以我也想尽自己所能让它变得更加安全。这段时间，我已知在寻找 Ubuntu 系统服务中得安全漏洞，我也发现并报告了一些问题，但大多数漏洞的严重性都很低。Ubuntu 是开源的，这意味着已经有很多前辈已经看过它的源代码了，似乎所有的漏洞都已经被发现了。不过，我还是发现了一个问题，并打算在这篇文章中跟大家详细介绍一下。

在这篇文章中，我将跟大家介绍一种在 Ubuntu 上实现权限提升的方法。只需要在命令行终端中输入几个简单的命令，再点击几次鼠标，普通权限的用户就可以为自己创建一个管理员账户了。在文章中，我还提供了一个演示视频以供大家学习参考。

注意，这个漏洞只会影响使用图形化系统桌面的 Ubuntu 用户。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

漏洞利用步骤

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

首先，我们需要打开一个命令行窗口，然后创建一个指向 home 目录的符号链接：

ln -s /dev/zero .pam\_environment

如果这个命令运行不了的话，可能是因为目标目录下已经存在一个名为. pam\_environment 的文件了，我们可以把这个旧文件重命名一下，等需要用到的时候再恢复就行了。

接下来，在系统设置中打开 “区域 & 语言” 选项，然后尝试修改语言。此时对话框将会卡住，这里大家不用管，此时切换回终端窗口。这个时候，一个名为 accounts-daemon 的程序将会占用 100% 的 CPU 资源，此时你的电脑将会变得非常卡。

不要慌，在命令行中删除这个符号链接，否则你可能会把自己锁定在自己的账号里：

crm .pam\_environment

下一步，给 accounts-daemon 发送一个 SIGSTOP 信号，让它停止占用 CPU 资源。但是这里，我们首先需要知道 accounts-daemon 的进程标识符（PID）。在演示视频中，我使用的是 top 工具（用于监控运行进程），因为 accounts-daemon 已经陷入了一个死循环中，这样可以让它立刻排到列表顶部。另一个寻找 PID 的方法就是使用 pidof 工具：

$ pidof accounts-daemon  
597

拿到 accounts-daemon 的进程标识符（PID）之后，我们就可以使用 kill 命令来发送 SIGSTOP 信号了：

kill -SIGSTOP 597

现在，你的电脑应该能 “喘过气” 了吧。

接下来就是关键步骤了！现在，注销你的账号，但你得先设置一个定时器来在你注销账号之后重启 accounts-daemon 进程。否则，你将被锁定无法登录，那么漏洞利用也就失败了。设置定时器的代码如下：

nohup bash -c "sleep 30s; kill -SIGSEGV 597; kill -SIGCONT 597"

nohup 工具可以让我们注销之后仍然保持脚本的运行状态，这个命令将运行一个 bash 脚本，这个脚本会做三件事情：

1、休眠 30 秒；

2、给 accounts-daemon 发送一个 SIGSEGV 信号，来让程序崩溃亏；

3、给 accounts-daemon 发送一个 SIGCONT 信号来停用之前发送的 SIGSTOP。SIGSEGV 不会生效，直到接收到了 SIGCONT 为止；

完成之后，注销账号，然后等几秒种让 SIGSEGV 执行。如果漏洞利用成功，你将会看到一大堆的对话框弹出来，然后帮助你设置一个新的用户账号，而这个新账号，就是管理员账号。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fciarKqCg5pznep4hVWwPRj0s1iah4zibOmxibicQmanH6GnAxiczl076b12g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

漏洞运行机制

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

不用担心，就算你之前对 Ubuntu（准确的说应该是 GNOME）不是非常了解，我也会耐心跟大家介绍这个漏洞的。实际上，这里涉及到两个漏洞，第一个漏洞存在于 accountsservice 中，这个服务负责管理计算机上的用户账号。第二个漏洞存在于 GNOME Display Manager （gdm3）中，它负责管理登录界面。

### **accountsservice 拒绝服务漏洞（GHSL-2020-187、GHSL-2020-188/CVE-2020-16126、CVE-2020-16127）**

accountsservice 守护进程（accounts-daemon）是一个负责管理设备上用户账号的系统服务。它可以创建新的用户账号，或修改用户密码。除此之外，它还能修改用户的头像或语言偏好。守护进程能够在系统后台运行，而且不需要拥有用户接口。但是，系统设置对话框能够通过 D-Bus 这个消息系统来跟 accounts-daemon 交互：

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fg964jpyxgkHtYVgwzFkdoms8Vugic4yibs6rYsnMWKyIjmS9m3HcgMNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fbxQzXUe33gW67ubS64bmboaasRiaoYMnYtrSZpYyztVWh6EkwOMQMSw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

关于 D-Bus

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

D-Bus 实质上一个适用于桌面应用的进程间的通讯机制，即所谓的 IPC（inter-process communication）机制。它最初产生于 Linux 平台，是做为 freedesktop.org 项目的一部分来开发的。现在已经深入地渗透到 Linux 桌面之中。在 Qt4，GNOME，Windows 以及 Maemo 中都已实现。在 KDE4 中已经取代了著名的 DCOP，在 GNOME 取代笨重的 Bonobo。在嵌入式系统中常用来实现 C/S 结构。

D-Bus 作为应用程序间通信的消息总线系统, 用于进程之间的通信。它是个 3 层架构的 IPC 系统，包括：

1、函数库 libdbus，用于两个应用程序互相联系和交互消息。

2、一个基于 libdbus 构造的消息总线守护进程，可同时与多个应用程序相连，并能把来自一个应用程序的消息路由到 0 或者多个其他程序。

3、基于特定应用程序框架的封装库或捆绑（wrapper libraries or bindings ）。

例如，libdbus-glib 和 libdbus-qt，还有绑定在其他语言，例如 Python 的。大多数开发者都是使用这些封装库的 API，因为它们简化了 D-Bus 编程细节。libdbus 被有意设计成为更高层次绑定的底层后端（low-level backend ）。大部分 libdbus 的 API 仅仅是为了用来实现绑定。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

继续我们的漏洞利用分析

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fvq6zR6TUibiakxZ2CEDMCrWpH9S1icdM9uUT8fcbiatCmWHR92f5icPP3yA/640?wx_fmt=png)

在漏洞利用过程中，我使用了系统设置对话框来修改语言。标准用户可以自己修改这个设置，而无需管理员权限。在后台，系统服务对话框此时会通过 D-Bus 向 accounts-daemon 发送 org.freedesktop.Accounts.User.SetLanguage 命令。  
实际上，Ubuntu 使用的是一个修改版的 accountsservice，其中包含了 freedesktop 上游版本中未包含的额外代码。Ubuntu 添加了一个名为 is\_in\_pam\_environment 的函数，它会寻找并读取用户 home 目录下一个名为. pam\_environment 的文件。此时，我们只需要制作一个. pam\_environment 指向 / dev/zero 的符号链接，即可触发这个拒绝服务漏洞。/dev/zero 是一个特殊的文件，它实际上并不存在于磁盘中。它由操作系统提供，全部由零字节组成。当 is\_in\_pam\_environment 尝试读取. pam\_environment 时，它会被符号链接重定向到 / dev/zero，然后陷入无限循环中。

### **accounts-daemon 的无响应触发的 gdm3 提权漏洞（GHSL-2020-202 / CVE-2020-16125）**

GNOME Display Manager（gdm3）是 Ubuntu 用户接口中的一个基础组件，它负责处理用户在登录和注销过程中的类似用户会话启动和关闭之类的任务。除此之外，它还负责管理系统的登录界面。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fFQPacEQLkNibLcCGwMficxoL5utrpBzBrDs03RW9V8xytr5f11pl8kGw/640?wx_fmt=png)

gdm3 负责处理的另一个东西就是计算机在安装完新系统之后的初始化配置，当你在一台新的设备上安装好 Ubuntu 之后，第一件要做的事情就是创建一个新的用户账号。初始用户账号需要是一个管理员账号，这样我们才能继续完成设备的配置，比如说配置 WiFi 和安装应用程序等等。下面给出的截图显示的就是设备的初始化安装界面：

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb45Xz7YvYbDsFicic1zAF1I6fRTluic9XNZrlrHQicJtRcZTqLC1ibJypmFvJTayqT0G5mk36ARHYAtXtA/640?wx_fmt=png)

大家在上图中看到的对话框就是一个单独的应用程序，这个应用程序名叫 gnome-initial-setup，当系统中不存在用户账号时，gdm3 便会触发它的执行，也就是在系统初始化配置的时候会执行。

它会使用 D-Bus 来询问 accounts-daemon：现在系统中已经有多少个用户账号了？但是由于 accounts-daemon 已经无响应了，那么 D-Bus 的调用也就因超时（大约 20 秒超时时间）而失败了。由于超时情况的出现，代码并不会设置 priv->have\_existing\_user\_accounts 的值。不幸的是，priv->have\_existing\_user\_accounts 的默认值为 false，而不是 true。那么现在，gdm3 将认为系统中不存在任何用户账号，于是它便会启动 gnome-initial-setup。

没错，就是这么简单！

**译文声明**  

译文仅供参考，具体内容表达以及含义原文为准。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

  

\- End -  

精彩推荐

[微软工程师薅微软羊毛 1000 万美金，竟拿同事当替罪羊](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649734698&idx=1&sn=a4e6816bd444aba7199f4edb9919ffd5&chksm=888c8e45bffb0753ae13438cdd14153f63038156876f47d09273221d63b5b6a2b4269ec5c2b7&scene=21#wechat_redirect)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[双十一大回血：送书 + 送钱！](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649734525&idx=1&sn=ad3ec4a6cb3d489fee573e0afae65d6b&chksm=888c8d12bffb04043eccad4c888e3da8de44de792c393533c90e50a16552bbd89177f0c8dc66&scene=21#wechat_redirect)
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[美国联邦贸易委员会曝光 Zoom 就端到端加密问题已欺骗用户多年](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649734695&idx=1&sn=6653bab47b6647ebc30c68d60d2b477a&chksm=888c8e48bffb075e6d2b33af4f3e016fc237c94287b1f6bdb7284c5d9c5c1ffd9c0f96eaa76c&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[2020 湖湘杯 PWN WriteUp](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649734687&idx=1&sn=8d60a3d964f4a1002df98dad8232cdee&chksm=888c8e70bffb07664441101eaa598fb9e7cb97e055f15cd6747f54bc1561cecda94254b167c2&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)  

------------------------------------------------------------------------------------------------------------------------------------------------

  

**戳 “阅读原文” 查看更多内容**