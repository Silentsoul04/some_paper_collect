> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/O6jaLb2ITcP9850ZDqaPXA)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzLicKwibCOrj4LSdPHyjzIeCKlSvwNBOBxyUO2TxqtBXsZa3lXycRjicD81lr7wpjXkKO37ETn00Sow/640?wx_fmt=png)

  

1

  

**安全应急**

**1、应急响应目标**

在第一时间采取响应的措施，恢复业务到正常，调查安全事件发生的原因，避免同类事件发生，提供数字证据

**2、应急响应范围**

邮件钓鱼，黑客入侵，APT 攻击，漏洞利用，网络攻击，数据外泄，时间通报，攻击溯源，网络异常，网站被黑，网站挂马，网站暗链，网站篡改

**3、应急响应的目的**

判断这次应急是否是被成功入侵的安全事件

找到攻击者入口点

提取恶意样本

帮助客服梳理攻击者的攻击路线，提供漏洞修复方案

2

  

**Linux**

**1、Linux 系统基础知识**

Etc/sysconfig/network-script/ 网卡配置

目录结构

`1/bin 放置系统命令目录，/bin目录可以在维护模式下还可以被操作  
2/boot 放置开机使用的文件，包括linux核心文件以及开机选单与开机所需设定文件  
3/dev 放置装置与周边设备都是以文件的形态，存在于这个目录当中  
4/etc 系统主要的设定文件几乎都放置在这个目录内，列如人员的账户密码文件，各种服务的起始文件等等  
5/home 系统预设的使用者家目录(home,directory) 你新增的用户家目录都会规范进来  
6/lib和/lib64 系统的函试库非常的多，而.lib或者/lib64放置常用的函数  
7/opt 存放第三方软件和位置目录  
8/root root用户的家目录  
9/sbin linux有非常多指令是用来设定系统环境的，这些指令只有root才能够利用来设置系统，里面包括了开机，修复，还原系统所需要的指令  
10/tmp 任何人都可以读写的目录，重新启动目录中存放的文件会被删除  
`

用户组

`1所有者，所在组，其它组  
`

文件普通权限

`1Rwx r=4 w=2 x=1  
`

文件特殊权限

`1SUID：s出现在文件所有者的X权限上  
2SGID：s出现在文件所属群组的X权限上  
3SBIT：t出现在文件其他用户的x权限上  
`

**2、Linux 常用命令**

Stat 跟文件

会显示文件 UID，文件名，文件属性，文件访问时间，文件内容修改时间，文件元数据变化时间

`1Access time 访问时间：文件中的内容最后被访问的最后时间  
2Modified time 修改时间：文件内容被修改的最后时间  
3Change time变化时间：文件的元数据发生变化，写入文件，更改所有者，权限修改  
`

Ls 命令 列出文件

`1-a 显示隐藏文件，-L 详细显示，-R 递归显示  
`

Netstat 命令

`1-a 显示所有链接中的soket   -n 使用ip，而不是域名显示 -t 显示TCP链接  
2-p 显示每个网络链接对应的进程和用户 -l 显示处于监听中的socket  
3-e 显示拓展信息，inode等信息  
4-u 显示UDP连接  
`

Lsof 命令 列出来当前系统打开文件

`1Lsof -c sshd 显示sshd进程现在打开的文件  
2Lsof -p pid 显示进程号为pid的进程情况  
3Lsof +d /tmp 显示目录下被进程打开的文件  
4Lsof +D /tmp 递归显示显示目录下被进程打开的文件  
5Lsof -i:80 查看端口为80的tcp或者udp进程  
`

Ps 命令 常用进程

`1Ps -a 显示当前终端下的进程  
2Ps -u 以用户为主的显示方式  
3Ps -x 显示所有进程  
`

Grep 命令 检索命令

`1Grep -I 忽略大小写  
2Grep -v 不包含特殊字符  
`

Tcpdump 命令 抓流量

`1Tcpdump -I eth0 抓取网卡为eth0的流量  
2Tcpdump tcp   抓取tcp流量  
3Tcpdump port 53 抓取端口为53的流量，源端口或者目的端口  
4Tcpdump host 1.1.1.1 抓取和主机1.1.1.1 有关的流量  
5Tcpdump -w wireshark。Pcap 流量保存为图形化分享软件可识别的数据包  
`

Find 命令 目录文件列出来，也可以查找文件

`1Find path -name filename 在path路径下查找文件名为filename的文件  
2Find path -perm 777 在path路径下查找文件权限为777的文件  
3Find path -perm -700 在path路径下查找文件权限为700以及700以上权限的文件  
`

Md5 计算文件 Md5

Rz 接收文件

Sz 传输文件

Strings 字符串显示文件

Linux 常规检查项 - 关键文件，关键目录等

文件检测 - 历史命令

Root 用户执行过的历史命令

`1/root/.bash_history  
`

对应 username 执行过的历史命令

`1/home/{username}/.bash_history  
`

当前用户执行过的历史命令

`1History  
`

文件检测 - 系统关键文件

`1/etc/passwd 包含系统用户和用户的主要信息  
2/etc/shadow 用于储存系统中用户的密码，又称为影子文件  
3/etc/group  记录组ID和组名的对应文件  
`

写到 profile 里面的命令都是会被调用执行的

`1/etc/profile 此文件涉及系统的环境，变量，会从中调用shell变量  
2/root/.bash_profile 变量  
3/home/{username}/.bash_profile 变量  
4/root/.bashrc  此文件为系统的每个用户设置的环境信息，用户第一次登陆时，该文件被执行  
5/home/{username}/.bashrc  
6/root/.bash_logout  用户退出的时候的操作  
7/home/{username}/.bash_logout  
8/root/.ssh/authorized_keys  存放ssh公私钥的地方  
9/home/{username}/.ssh/authorized_keys  
`

文件检测—系统关键目录

`1/root/  
2/home/{username}  
3/tmp  
4/var/tmp  
5/dev/shm  
`

命令检测，防止恶意命令替换

`1/usr/bin/ps  
2/usr/bin/netatat  
3/usr/bin/ps  
4/usr/sbin/lsof  
5/usr/sbin/ss  
6/usr/bin/stat  
`

检测系统安装包

列出系统所有按照的 rpm 包

Rpm -qa

校验系统所有 rpm 包

Rpm -V -a

校验特定文件或者命令

Rpm -V -f /etc/sysconfig

Which ps 查看 ps 命令的变量路径

**3、Linux 应急计划任务**

Crontab 计划任务：系统自带的定时执行脚本或者命令的系统服务

Systemctl status crond  查看 crond 服务状态

Systemctl start crond   启动 crond 服务

Systemctl stop crond 停止 crond 服务

Systemctl enable crond    开机启动 crond 服务

Systemctl disable crond  关闭开机启动 crond 服务

Crontab

`1-l 列出crontab  
2-u 指定用户用户  
3-e 编辑crontab  
`

从左到右依次为：

[分钟] [小时] [每月的某一天] [每年的某一月] [每周的某一天] [执行的命令]

/etc/cron.deny    /etc/cron.allow

Crontab 的限制文件，用户名存在 cron.deny，且 cron.allow 不存在，或者用户名不存在 cron.allow 的时候，不允许此用户创建 crontab

Cron.allow 优先级高于 cron.deny

系统默认不存在 cron.allow

系统默认 crontac 相关配置文件目录

`1/etc/cron.d/  
2/etc/cron.hourly/  
3/etc/cron.daily/  
4/etc/cron.monthly/  
5/etc/cron.weekly/  
6/etc/crontab  
7/var/spool/cron/  
8/var/log/cron  crontab日志  
9/etc/systemd/system/multi-user.target.wants/crond.service  
`

**4、Linux - 系统日志**

配置文件

`1/etc/rsyslog.conf  
2/etc/rsyslog.d/*  
`

登陆相关日志

`1/var/log/下  
2Secure 记录于安全相关的信息  
3Lastlog 当前登陆的用户日志  
4Wtmp 永久记录每个用户登陆，注销及系统的启动，停机的事件，last命令查看  
5Btmp 尝试登陆且失败日志  
`

其他日志

`1Messages 各种系统守护进程，用户程序和内核相关信息  
2Cron   c rontab日志  
3Audit/*   audit日志，监控系统调用  
4Boot.log  启动信息相关日志  
5Yum.log  通过yum安装rpm相关日志  
6Httpd/*   httpd服务访问日志和错误日志  
7Firewalld  防火墙相关日志  
8Mail      邮件相关日志  
9Dmesg    核心启动日志  
`

Windows 常见命令

`1Regedit 查看策略表  
2Msconfig  查看系统配置  
3Taskmgr   启动任务管理器  
4Eventvwr,msc  打开日志的命令  
5Gpedit.msc   打开本地组策略  
6Compmgmt.msc  计算机管理  
7Lusrmgr.msc   打开用户与组  
8Taskschd   打开计划任务  
9Net user xxx /add 添加用户  
10Net localgroup administrators xxxx /add 把某用户放到管理员组里面  
11Net session 查询当前会话  
12Net start 查看当前运行的服务  
13Net use 查看当前共享连接  
14Net share 查看共享映射的盘符，连接状态  
15Net share xxx /del 删除共享的链接  
16查看隐藏用户可以去，用户管理  
17Findstr /s /I “hellow” **   
18查询包含hellow 的关键字  
19Wmic pross  
20Attrib 查看文件属性  
21Attrid 1.txt  
22Attrid -R  
`

系统日志收集工具

`1Sglad_ir  
2Gather_log  
`

3

  

**windows**

系统变量敏感文件路径

`1%WINDIR%     C盘 windows  
2%WINDIR%\system32\%  c盘windows system32  
3%TEMP% 临时目录  
4%APPDATA% 软件程序  
5%LOCALAPPDATA% 软件程序  
`

Windows 系统日志

`1C:\Windows\System32\winevt\Logs\system.evtx  
`

Windows 系统安全日志

`1C:\Windows\System32\winevt\Logs\ Security.evtx  
24624id 是登陆成功的id  
`

Windows 应用程序日志

`1C:\Windows\System32\winevt\Logs\ Application.Evtx  
2主要关注安全日志，里面记录账户登陆，注销，等等的日志  
`

系统日志中的 id

`1Id 12 系统启动  
26005ID 事件日志服务启动  
36004ID 事件日志服务停止  
4Id 13 系统关闭  
`

安全日志中的 id

`14732 添加用户启动安全性的本地组中  
24722  启动用户的id  
34720  创建用户  
44624 登陆成功  
54625 失败登陆  
64726  删除用户  
74634 注销  
84776  成功/失败的账户认证  
91102 清理日志  
`

安全日志中的 登陆日志类型

`12 交互登陆  
23 网络登陆(通过net use，访问共享网络）  
34 批处理(为批处理程序保留)  
45 服务器启动  
56 不支持  
67 解锁（带密码保护的屏幕保护程序）  
78 网络明文，iis服务器登陆验证  
810 远程交互(终端服务，远程桌面，远程辅助)  
911 缓存域证书登陆  
`

常用的抓包工具

`1wireshark  
2Tcp.port eq 25  
3查询tcp端口为25的  
4Ip.addr == 127.0.0.1 包含127.0.0.1的  
5Linux 抓包用的比较多  
6Tcpdump  
7Tcpdump host ip  
8抓这个ip  
9Tcpdump host ip1 and ip2  
10抓  
11Tcpdump -I eth0 监听这个网卡  
12Tcpdump tcp port 445 and src host ip  
`

源 ip 端口为 445 的

这个 windows 用的比较多，比较简单好用，用 Micrsosoft Network Monitor

安全分析工具

PChunter

Autoruns

Process explorer

https://www.anquanke.com/post/id/182858

Web 日志分析

`1Apache  
2在httpd.conf里面记录log的访问日志的路径  
3/etc/httpd/logs  
4Access.log  
5Nginx  
`

现在主要的日志格式是 NCSA 拓展格式

`1访问主机(remotehost)  
2日期时间(date)  
3请求(request)  
4请求类型(METHOD)  
5请求资源(RESOURCE)  
6协议版本号(PROTOCOL)  
7状态码(status)  
8传输字节数(bytes)  
9来源页面(referrer)  
10浏览器信息(agent)  
`

![](https://mmbiz.qpic.cn/mmbiz_gif/rCukMxCXicnYUtJSDs80JIhNguHxevVR6uLBIBf4U8hibYuWgicImvpWJ6mA2bXcF7mVuGwQN6ZIJEO7rPmgxBLJA/640?wx_fmt=gif)

文：Jie

转自：https://www.hackjie.com/1402.html

如有侵权请联系删除

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzLicKwibCOrj4LSdPHyjzIeCec4cT7TKYicpltRA9sjls9gnl2G8aQ2xxbEMDPklOXS9Qq1PiaWicxcjA/640?wx_fmt=png)

我就知道你 “在看”

![](https://mmbiz.qpic.cn/mmbiz_gif/rCukMxCXicnaJbqicEeFlobznozfm72D79VrDP7Z5o6icc8SVia8haOeSC8wakd8Wo4LboXV8DFgJP5Xf0fcPD1BHA/640?wx_fmt=gif)