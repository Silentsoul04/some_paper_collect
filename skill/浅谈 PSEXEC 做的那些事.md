\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/HOck5KYFGrVvCxrRCtrnHw)

这是 **酒仙桥六号部队** 的第 **108** 篇文章。

全文共计 3058 个字，预计阅读时长 9 分钟。

  

===

**前言**

在某个游戏的夜晚，兄弟找我问个工具，顺手聊到 PsExec 的工具，之前没用过，看到兄弟用的时候出现了点问题，那就试用用，顺便分析一下它做了什么。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL15xoib2XNrukDO7FwCkibs61TIibP4u9y4hQ7kRianFdqliaveXpibgG2DITg/640?wx_fmt=png)

**PsExec 简介**

PsExec 是由 Mark Russinovich 创建的 Sysinternals Suite 中包含的工具，是一种命令行工具。最初，它旨在作为系统管理员的便利工具，以便他们可以通过在远程主机上运行命令来执行维护任务。PsExec 是一个轻量级的 telnet 替代工具，它使您无需手动安装客户端软件即可执行其他系统上的进程，并且可以获得与命令控制台几乎相同的实时交互性。PsExec 最强大的功能就是在远程系统和远程支持工具（如 ipconfig、whoami）中启动交互式命令提示窗口，以便显示无法通过其他方式显示的有关远程系统的信息。

*   PsExec 特点
    

1.  psexec 远程运行需要远程计算机启用文件和打印共享且默认的 Admin$ 共享映射到 C:\\windows 目录。
    
2.  psexec 建立连接之后目标机器上会被安装一个 “PSEXESVC” 服务。但是 psexec 安全退出之后这个服务会自动删除（在命令行下使用 exit 命令退出）。
    

**工作原理**

*   PsExec 详细运行过程简介
    
    正式开展测试，启用 net sharAdmin $ 共享。拒绝访问？这是要出师未捷身先死？
    

1.  TCP 三次握手，通过 SMB 会话进行身份验证。
    
2.  连接 admin$ 共享，通过 SMB 访问默认共享文件夹 ADMIN$，写入 PSEXESVC.exe 文件；
    
3.  利用 ipc 命名管道调用 svcctl 服务
    
4.  利用 svcctl 服务开启 psexesvc 服务
    
5.  生成 4 个命名管道以供使用。一个 psexesvc 管道用于服务本身，另外的管道 stdin（输入）、stdout（输出）、stderr（输出）用于重定向进程。
    

正式开展测试，启用 net sharAdmin $ 共享。拒绝访问？这是要出师未捷身先死？

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1OyDmJrJBLUYjegAn3YPiaVEKjgY2zAkkIC4IbNibW3iaGBDoKRQF41TPw/640?wx_fmt=png)

稳住，先别慌，抓包看看，目测是 admin$ 无法访问导致的。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1y4Jic377lqYZDhBFApjMHmUoJA82JNCobOOx5eO9pARd4OtDdMthtDA/640?wx_fmt=png)

检查 admin $、IPC$，已经开启共享。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1Q1yZSvoO9oQu1iaSyicicRdqSKbdkJPpqiaamicrFGE29st8wKc6bg0M7UA/640?wx_fmt=png)

尝试访问一下，果然是 admin$ 访问不了，咋办呢（陷入沉思~~）

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1ZYic6LUhkDF1GCVqSoIV4AdhdhX4bdPaqRlmbyESXRQaEW4A6ddQh1g/640?wx_fmt=png)

本地策略原因限制了访问？打来看看 “网络访问、拒绝本地登陆、拒绝从网络远程访问这台计算机” 的策略，没异常啊。不是策略，机制么？remote UAC？很大可能呀，不管，关了！

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL16f7iaibhEOfY3aiarGPmgRAzftHCRxicvVFHz30EQxUS3rstLKMEVWO2QA/640?wx_fmt=png)

再运行 psexec：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1Xk50EvsCHQDwMFMBJ4Riau6ticeDyzVzXhMCQ95gvIBgA0JJkY1T7vWA/640?wx_fmt=png)

哦豁，可以了，目标服务器被添加 “PSEXESVC” 服务。为什么关了 remote UAC 就可以了？（陷入了反思~）

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1wl23t3aGunI8VI8IH1yVHv6NRfKAoIwa0d0ou7DfPYsyDhaTaeiayKA/640?wx_fmt=png)

UAC 是什么？UAC 是微软在 Windows Vista 以后版本引入的一种安全机制，可以阻止未经授权的应用程序自动进行安装，并防止无意中更改系统设置。那么对于防御是不是不改 UAC，保持默认或更高就可以了？并不是，可以改注册表的嘛。

方法二：

HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System 添加新 DWORD 值，键值：LocalAccountTokenFilterPolicy 为 1。  

**进一步分析**

条件具备，软件正常，开始抓包分析。psexec 刚开始运行就做了三件事，第一：通过 TCP3 次握手连接目标 445 端口；第二：SMB 协商使用 SMBv2 协议通信；第三：进行 NTML 认证。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1n9Xovjj8hWv1NRRlwf7aep8hmiaW0sdiaPql2gsq6Y3NzRDXhTsl0bIg/640?wx_fmt=png)

三次握手略过，直接分析 SMB 协议。SMB(全称是 Server MessageBlock) 是一个协议名，可用于在计算机间共享文件、打印机、串口等，电脑上的网上邻居就是靠它实现的，SMB 工作原理如下：攻击机向目标机器发送一个 SMB negotiate protocol request 请求数据包，并列出它所支持的所有 SMB 协议版本（其中 Dialect 带有一串 16 进制的 code 对应着 SMB 的不同版本以此分辨准确版本），若无可使用的版本返回 0XFFFFH 结束通信。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1UwJVBWJP554MOp9HWSu17JFHIy6vJHmOK8GC1hbL912sr5C0uxr8sA/640?wx_fmt=png)

目标机器返回 NEGTIATE ResponseDialect 数据包协商确定使用 SMB2.1，至此 SMB 协商使用 SMBv2 协议通信过程结束。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1uSkRfv7ZXx4LrG3BQI1cJ6gGy3jIB4904et9WkiarhlQ8MNibordia7icA/640?wx_fmt=png)

NTML 认证开始，攻击机向目标机器发送 SESSION\_SETUP\_ANDX 协商请求，以完成攻击机与目标机器之间的身份验证，该请求包含用户名密码。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1d04e4eMlGB5fMKm1OIj05KcfXdzaOexGMkFeMHJn2rhacSnuXWMuQA/640?wx_fmt=png)

认证结束，psexec 就能正常使用了么？肯定不是，接着进入 PsExec 运行的重点分析过程。首先，攻击机向目标机器发送 Tree connect rerquest SMB 数据包，并列出想访问网络资源的名称 ipc$、admin$，目标机器返回 tree connect response 响应数据包表示此次连接是否被接受或拒绝。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1DzooNtYqRL033ULiae9eBO8xib8PzcZjWbmZePia5khvfV66eW7zMekOg/640?wx_fmt=png)

连接到相应资源后，通过 SMB 访问默认共享文件夹 ADMIN$，写入 PSEXESVC.exe 文件。（4d5a 是 PE 文件即可移植的可执行的文件的 MZ 文件头）

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1Arqw7Gjr3SRpuuslVsH0jiazicQfnFZ3kAeHHXQlwLNicZbIfLz84HdBQ/640?wx_fmt=png)

close request and response 数据包表示 PSEXESVC.exe 文件完成写入。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL15a6cdEf8wpDSMZKUEntgzW2Abl2D8EWL6v1JfSCAvA27kiaNIyWxpNA/640?wx_fmt=png)

从代码层面看，psexec 从资源文件中提取出了一个服务，并开始创建且运行了该服务程序。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1Pp7oeFBTOH54YgjTrxVkHySPkKcryD9rQqjPo1lOWWxwTKhC13VlxQ/640?wx_fmt=png)

接着查看 openservicew request 的数据包，发现攻击机开始远程调用 svcctl 协议并打开 psexesvc 服务（psexec 必须调用 svcctl 协议，否则 psexesvc 服务无法启动）

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1x5qg37nc1mg3N4eLRO55WCWQApo28pczBjKYsibYmJPBJDWsD85DdPg/640?wx_fmt=png)

从代码层面看到，还需要创建与服务端通信的管道名。PsExec 使用命名管道可在同一台计算机的不同进程之间或在跨越一个网络的不同计算机的不同进程之间，支持可靠的、单向或双向的数据通信。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1sbCmib6Y1gQOedIngRS3zmZts6WFw4icehicvRNiaqZuib32iclDhicyrqM0w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL19ro1eZ6WKGZDOXUNwOM3Jk0OseG18UUs2ticAdXZXjY8BxYWNHDWruA/640?wx_fmt=png)

从数据包层发现开始创建 psexesvc、stdin、stdout、stderr 4 个命名管道。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL16e7WLqF9pF5bUavibfBrCRkSdSOhEUQWev0jE5C3LH4nZkFHJ1SFBwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1Fnvk04YstJ5AicRdMOGyTKdLnHtOVazOf22bqjWBtLkhjMRER7uJWwg/640?wx_fmt=png)

管道创建成功，psexec 可以正常使用，已成功连上目标机器 cmd。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1ia2mVDfqAG1n4Ng8604par1vQagC87DkTUVwtvwO1xPlOGXrtQj9pCA/640?wx_fmt=png)

在连接过程中，攻击机会每隔 30s 向目标机器发送一次 TCP-keep-alive 数据包，保持 TCP 心跳连接。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1nuyxicOHn4BSfzpIaVLNy2SZLYgUYK7Po5dxJTwleWYfNLibHuI2K4Qg/640?wx_fmt=png)

攻击机退出远程连接时，tcp 四次挥手关闭连接，psexesvc、stdin、stdout、stderr4 个管道也会关闭，会话结束。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1y5T8EjdbcwPGanJPe4yrEicI0ibx4licGictBAGBn96IpfFJnea1AYqMmg/640?wx_fmt=png)

psexec 成功登录退出后，会在目标机器的安全日志中产生 Event 4624、4628、4634，在系统日志中产生 Event 7045（记录 PSEXESVC 安装）、Event 7036（记录 PSEXESVC 服务状态）。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1SjpG6I263sNcERicfCxticTaMH6Wx64gPCzCIficXMqLNYQ7oykqa0Etw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1GXcb5fiad0tpXO2s3lfn7a8Q88UuhrywNcgI5ibZ95AThNe9EJjwZWKg/640?wx_fmt=png)

另外，当 psexec 远控目标机器时，可执行程序 PSEXESVC.EXE 被提取至目标机器的 C:\\Windows 目录下，然后再执行远程操作命令，psexec 断开后，目标机器 C:\\Windows 目录下的 PSEXESVC.EXE 被删除。

pexec 连接成功，打开目标机器 cmd，可执行 cmd 相关命令，还有其它相关命令：

```
psexec \\\\\\\\ip -u administrator -p 123456 -d -s calc
```

运行 calc 后返回，目标机器上会有一个 calc 进程，-s 意思是以系统身份运行。窗口是看不到的，如果需要目标机器看到这个窗口，需要加参数 - i。

```
psexec \\\\\\\\ip -u administrator -p 123456 -d calc
```

以当前身份运行 calc，然后返回。

```
psexec \\\\\\\\ip -u administrator -p 123456 -i -d cmd /c start http://www.baidu.com
```

以目标机器当前用户身份打开百度网页，并让他看到这个网页。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1RGXF73OS9dUI4QzCPrfmHfsqGaoMH6ibhEcJL1YrUoVibKAkv6NicK70A/640?wx_fmt=png)

**结尾**

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56IS7Izs1VxXnU7RFjoiadL1g8A9PaAQ92FHvayoYMZQq3QkicxQXuTHwyNuBDsyQNLMnoLJXAtztBA/640?wx_fmt=png)

如果运营过程发现安全设备有 psexec 相关告警，检查的时候围绕着 psexec 的特性针对性地对数据包的检查，发现误报及时添加相关白名单过滤持续性的安全运营，能显著地提高安全运营能力。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s55hGyuQ3vjH8LnEch46xIsCCA3vKcviaWaGVPbPagAMEfvLDVPic3Otn6qW0tI3dtusOFmBb4BjznvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibiaRBNn0A5YI88OyFjU8fn2Isf9bat4vQn18NwG6cXxVOSuKiapNm2nibQ/640?wx_fmt=png)