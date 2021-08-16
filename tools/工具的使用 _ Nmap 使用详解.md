> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247483849&idx=1&sn=7448559219c08db3913d027064af17da&chksm=eaad81f4ddda08e24435273ce2ce762060197223ca3719a38589299bbe5b3471e36531201380&scene=21#wechat_redirect)

Nmap 使用详解

  

  

  

*   探索目标主机是否在线
    
*       当探测公网 ip 时
    
*           nmap -sn
    
*           nmap  -PE/-PP/-PM
    
*       当探测内网 ip 时
    
*           nmap -sn
    
*           nmap  -PE/-PP/-PM
    
*   端口扫描及其原理
    
*   端口扫描用法
    
*       简单扫描 (nmap ip)
    
*       全面扫描 (nmap -A ip)
    
*       探测指定端口的开放状态
    
*       探测 N 个最有可能开放的端口
    
*   版本侦测
    
*       版本侦测原理
    
*       版本侦测用法
    
*   OS 侦测
    
*       OS 侦测原理
    
*       OS 侦测用法
    
*   Nmap 高级用法
    
*       防火墙 / IDS 规避
    
*           分片
    
*           IP 诱骗 (IP decoys)
    
*           IP 伪装
    
*           指定源端口
    
*           扫描延时
    
*           其他技术
    
*       NSE 脚本引擎
    
*   Zenmap 的使用
    

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC6KuYg7GdjHmJvggbicJmdtVDjgErQQgEFQApSaLHNRKIiakaAwFdbNDcluLTdOXjbNia2xQcjZShRjQ/640?wx_fmt=gif)

Nmap 是一款开源免费的网络发现（Network Discovery）和安全审计（Security Auditing）工具。软件名字 Nmap 是 Network Mapper 的简称。Nmap 最初是由 Fyodor 在 1997 年开始创建的。随后在开源社区众多的志愿者参与下，该工具逐渐成为最为流行安全必备工具之一。官网为：www.nmap.org。 

一般情况下，Nmap 用于列举网络主机清单、管理服务升级调度、监控主机或服务运行状况。Nmap 可以检测目标机是否在线、端口开放情况、侦测运行的服务类型及版本信息、侦测操作系统与设备类型等信息。

Nmap 的优点：

*   灵活。支持数十种不同的扫描方式，支持多种目标对象的扫描
    
*   强大。Nmap 可以用于扫描互联网上大规模的计算机
    
*   可移植。支持主流操作系统：Windows/Linux/Unix/MacOS 等等；源码开放，方便移植
    
*   简单。提供默认的操作能覆盖大部分功能，基本端口扫描 nmap targetip，全面的扫描 nmap –A targetip
    
*   自由。Nmap 作为开源软件，在 GPL License 的范围内可以自由的使用
    
*   文档丰富。Nmap 官网提供了详细的文档描述。Nmap 作者及其他安全专家编写了多部 Nmap 参考书籍
    
*   社区支持。Nmap 背后有强大的社区团队支持
    

Nmap 包含四项基本功能：

*   主机发现 (Host Discovery)
    
*   端口扫描 (Port Scanning)
    
*   版本侦测 (Version Detection)
    
*   操作系统侦测 (Operating System Detection)
    

而这四项功能之间，又存在大致的依赖关系 (通常情况下的顺序关系，但特殊应用另外考虑)，首先需要进行主机发现，随后确定端口状态，然后确定端口上运行的具体应用程序和版本信息，然后可以进行操作系统的侦测。而在这四项功能的基础上，nmap 还提供防火墙和 IDS 的规避技巧，可以综合运用到四个基本功能的各个阶段。另外 nmap 还提供强大的 NSE(Nmap  Scripting Language) 脚本引擎功能，脚本可以对基本功能进行补充和扩展。

先整理一些 nmap 参数及其意义

```
nmap –iflist : 查看本地主机的接口信息和路由信息
-A ：选项用于使用进攻性方式扫描
-T4： 指定扫描过程使用的时序，总有6个级别（0-5），级别越高，扫描速度越快，但也容易被防火墙或IDS检测并屏蔽掉，在网络通讯状况较好的情况下推荐使用T4
-oX test.xml： 将扫描结果生成 test.xml 文件，如果中断，则结果打不开
-oA test.xml:  将扫描结果生成 test.xml 文件，中断后，结果也可保存
-oG test.txt:  将扫描结果生成 test.txt 文件
-sn : 只进行主机发现，不进行端口扫描
-O : 指定Nmap进行系统版本扫描
-sV: 指定让Nmap进行服务版本扫描
-p <port ranges>: 扫描指定的端口
-sS/sT/sA/sW/sM:指定使用 TCP SYN/Connect()/ACK/Window/Maimon scans的方式来对目标主机进行扫描
-sU: 指定使用UDP扫描方式确定目标主机的UDP端口状况
-script <script name> : 指定扫描脚本
-Pn ： 不进行ping扫描
-sP :  用ping扫描判断主机是否存活，只有主机存活，nmap才会继续扫描，一般最好不加，因为有的主机会禁止ping
-PI :  设置这个选项，让nmap使用真正的ping(ICMP echo请求)来扫描目标主机是否正在运行。
-iL 1.txt : 批量扫描1.txt中的目标地址
 
-sL: List Scan 列表扫描，仅将指定的目标的IP列举出来，不进行主机发现
-sY/sZ: 使用SCTP INIT/COOKIE-ECHO来扫描SCTP协议端口的开放的情况
-sO: 使用IP protocol 扫描确定目标机支持的协议类型
-PO : 使用IP协议包探测对方主机是否开启 
-PE/PP/PM : 使用ICMP echo、 ICMP timestamp、ICMP netmask 请求包发现主机
-PS/PA/PU/PY : 使用TCP SYN/TCP ACK或SCTP INIT/ECHO方式进行发现
-sN/sF/sX: 指定使用TCP Null, FIN, and Xmas scans秘密扫描方式来协助探测对方的TCP端口状态
-e eth0：指定使用eth0网卡进行探测
-f : --mtu <val>: 指定使用分片、指定数据包的 MTU.
-b <FTP relay host>: 使用FTP bounce scan扫描方式
-g： 指定发送的端口号
-r: 不进行端口随机打乱的操作（如无该参数，nmap会将要扫描的端口以随机顺序方式扫描，以让nmap的扫描不易被对方防火墙检测到）
-v 表示显示冗余信息，在扫描过程中显示扫描的细节，从而让用户了解当前的扫描状态
-n : 表示不进行DNS解析；
-D  <decoy1,decoy2[,ME],...>: 用一组 IP 地址掩盖真实地址，其中 ME 填入自己的 IP 地址
-R ：表示总是进行DNS解析。 
-F : 快速模式，仅扫描TOP 100的端口 
-S <IP_Address>: 伪装成其他 IP 地址
--ttl <val>: 设置 time-to-live 时间
--badsum: 使用错误的 checksum 来发送数据包（正常情况下，该类数据包被抛弃，如果收到回复，说明回复来自防火墙或 IDS/IPS）
--dns-servers  : 指定DNS服务器
--system-dns : 指定使用系统的DNS服务器   
--traceroute : 追踪每个路由节点 
--scanflags <flags>: 定制TCP包的flags
--top-ports <number> :扫描开放概率最高的number个端口
--port-ratio <ratio>: 扫描指定频率以上的端口。与上述--top-ports类似，这里以概率作为参数
--version-trace: 显示出详细的版本侦测过程信息
--osscan-limit: 限制Nmap只对确定的主机的进行OS探测（至少需确知该主机分别有一个open和closed的端口）
--osscan-guess: 大胆猜测对方的主机的系统类型。由此准确性会下降不少，但会尽可能多为用户提供潜在的操作系统
--data-length <num>: 填充随机数据让数据包长度达到 Num
--ip-options <options>: 使用指定的 IP 选项来发送数据包
--spoof-mac <mac address/prefix/vendor name> : 伪装 MAC 地址
--version-intensity <level>: 指定版本侦测强度（0-9），默认为7。数值越高，探测出的服务越准确，但是运行时间会比较长。
--version-light: 指定使用轻量侦测方式 (intensity 2)
--version-all: 尝试使用所有的probes进行侦测 (intensity 9)
--version-trace: 显示出详细的版本侦测过程信息
nmap 192.168.1.0/24 -exclude 192.168.1.10  #扫描除192.168.1.0外的该网段的其他地址
nmap 192.168.1.0/24 -excludefile f:/1.txt  #扫描除给定文件中的地址以外的其他地址
nmap -sF -T4 192.168.1.0 #探测防火墙状态
```

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjHB7C85hnZBxEdY7XfdialFSs2sqkhU0hAQGG4vDn1nDOdXUGDicfaJ1rTeQDPNJRciaedicADLANghw/640)

一：探索目标主机是否在线

![](https://mmbiz.qpic.cn/mmbiz_png/uN1LIav7oJ8xLmmCxUAIxzgp52kq6Au35axTbLzLuPia6AjIgpPUctQZibSSGePqTcUvoicibQnIsaibicys2ib15oCNg/640?wx_fmt=png)

主机发现的原理与 Ping 命令类似，发送探测包到目标主机，如果收到回复，那么说明目标主机是开启的。Nmap 支持十多种不同的主机探测方式，用户可以在不同的条件下灵活选用不同的方式来探测目标机。主机发现常用参数如下。

```
-sn: Ping Scan 只进行主机发现，不进行端口扫描。
-PE/PP/PM: 使用ICMP echo、 ICMP timestamp、ICMP netmask 请求包发现主机。
-PS/PA/PU/PY[portlist]: 使用TCP SYN/TCP ACK或SCTP INIT/ECHO方式进行发现。 
 
-sL: List Scan 列表扫描，仅将指定的目标的IP列举出来，不进行主机发现。 
-Pn: 将所有指定的主机视作开启的，跳过主机发现的过程。
-PO[protocollist]: 使用IP协议包探测对方主机是否开启。  
-n/-R: -n表示不进行DNS解析；-R表示总是进行DNS解析。  
--dns-servers <serv1[,serv2],...>: 指定DNS服务器。   
--system-dns: 指定使用系统的DNS服务器   
--traceroute: 追踪每个路由节点
```

当探测公网 ip 时
----------

### nmap -sn

Nmap 会发送四种不同类型的数据包来探测目标主机是否在线。

1.  ICMP echo request
    
2.  a TCP SYN packet to port 443(https)
    
3.  a TCP ACK packet to port 80(http)
    
4.  an ICMP timestamp request
    

```
例：  nmap  -sn   133.133.100.30
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzVV0t0HCWEYJqOcBreYib3fR9Gibvd7e89cict6LpfvMXfvzeMGn03icibFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUznfibOtWrD0Zr9GdHRCbaKde5aWn5vf8TtXc08qWzYEUDbia1nYlDUXfg/640?wx_fmt=png)

依次发送四个报文探测目标机是否开启。只要收到其中一个包的回复，那就证明目标机开启。使用四种不同类型的数据包可以避免因防火墙或丢包造成的判断错误

通常主机发现并不单独使用，而只是作为端口扫描、版本侦测、OS 侦测先行步骤。而在某些特殊应用（例如确定大型局域网内活动主机的数量），可能会单独专门使用主机发现功能来完成。

### nmap  -PE/-PP/-PM

*   -PE 的 ICMP Echo 扫描简单来说是通过向目标发送 ICMP Echo 数据包来探测目标主机是否存活，但由于许多主机的防火墙会禁止这些报文，所以仅仅 ICMP 扫描通常是不够的。                                                nmap   -PE   133.133.100.30
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzXFvUdmsL9UB1RJCzvCzdMgmZC6JB10ZVaiapB17Zzje3nzNlKSKQcpg/640?wx_fmt=png)
    
*   -PP 的 ICMP time stamp 时间戳扫描在大多数防火墙配置不当时可能会得到回复，可以以此方式来判断目标主机是否存活。倘若目标主机在线，该命令还会探测其开放的端口以及运行的服务！
    
    nmap  nmap  -PP  133.133.100.30
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUz5UianYdicfZN4TCkicnXJiaqiae6b4UDOibs7en0T9icehMCZ7UWQ8v8zM5Pw/640?wx_fmt=png)
    
*   -PM 的 ICMP address maskPing 地址掩码扫描会试图用备选的 ICMP 等级 Ping 指定主机，通常有不错的穿透防火墙的效果       nmap  -PM  133.133.100.30
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzOVWQxyxNlTQfxHgMlDk0MS2lyhWNLBmdUoyqCgBxulonwXXt8q5sQg/640?wx_fmt=png)
    
*   -PS 的 TCP SYN 扫描
    

**当探测内网 ip 时**

### nmap -sn

使用 nmap  -sn  内网 ip   这个命令会发送 arp 请求包探测目标 ip 是否在线，如果有 arp 回复包，则说明在线。此命令可以探测目标主机是否在线，如果在线，还可以得到其 MAC 地址。但是不会探测其开放的端口号。

### nmap  -PE/-PP/-PM

使用 nmap  -PE/PP/PM  内网 ip 探测主机的开启情况，使用的是 ARP 请求报文，如果有 ARP 回复报文，说明主机在线。-PP/PE/PM 命令探测到主机在线后，还会探测主机的端口的开启状态以及运行的服务，其探测端口状态原理在下一节中有介绍。

探测该主机所在网段内所有主机的在线情况，使用的是  nmap  -sn   网段 / 子网掩码 。

```
例：nmap  -sn  10.96.10.0/24  或  nmap  -sn  10.96.10.100-200
```

探测 10.96.10.0 这个网段内主机的在线情况，返回在线主机的 ip 和 MAC 地址

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUz1ogBdicznQz8N7EbfSuEyq0q6xKKtIMxNiaVJSgwPlP7WEM1jbYtricmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjHB7C85hnZBxEdY7XfdialFSs2sqkhU0hAQGG4vDn1nDOdXUGDicfaJ1rTeQDPNJRciaedicADLANghw/640)

二：端口扫描及其原理

![](https://mmbiz.qpic.cn/mmbiz_png/uN1LIav7oJ8xLmmCxUAIxzgp52kq6Au35axTbLzLuPia6AjIgpPUctQZibSSGePqTcUvoicibQnIsaibicys2ib15oCNg/640?wx_fmt=png)

端口扫描是 Nmap 最基本最核心的功能，用于确定目标主机的 TCP/UDP 端口的开放情况。

默认情况下，Nmap 会扫描 1000 个最有可能开放的 TCP 端口

Nmap 通过探测将端口划分为 6 个状态：

1.  open：端口是开放的。
    
2.  closed：端口是关闭的。
    
3.  filtered：端口被防火墙 IDS/IPS 屏蔽，无法确定其状态。
    
4.  unfiltered：端口没有被屏蔽，但是否开放需要进一步确定。
    
5.  open|filtered：端口是开放的或被屏蔽，Nmap 不能识别。
    
6.  closed|filtered ：端口是关闭的或被屏蔽，Nmap 不能识别
    

**TCP SYN 扫描 (-sS)**

        这是 Nmap 默认的扫描方式，通常被称作半开放扫描。该方式发送 SYN 到目标端口，如果收到 SYN/ACK 回复，那么可以判断端口是开放的；如果收到 RST 包，说明该端口是关闭的。如果没有收到回复，那么可以判断该端口被屏蔽了。因为该方式仅发送 SYN 包对目标主机的特定端口，但不建立完整的 TCP 连接，所以相对比较隐蔽，而且效率比较高，适用范围广。

**TCP connent 扫描 (-sT)**

      TCP connect 方式使用系统网络 API connect 向目标主机的端口发起连接，如果无法连接，说明该端口关闭。该方式扫描速度比较慢，而且由于建立完整的 TCP 连接会在目标主机上留下记录信息，不够隐蔽。所以，TCP connect 是 TCP SYN 无法使用才考虑使用的方式

**TCP ACK 扫描 (-sA)**

      向目标主机的端口发送 ACK 包，如果收到 RST 包，说明该端口没有被防火墙屏蔽；没有收到 RST 包，说明被屏蔽。该方式只能用于确定防火墙是否屏蔽某个端口，可以辅助 TCP SYN 的方式来判断目标主机防火墙的状况

**TCP FIN/Xmas/NULL 扫描 (-sN/sF/sX)**

      这三种扫描方式被称为秘密扫描，因为相对比较隐蔽。FIN 扫描向目标主机的端口发送的 TCP FIN 包或 Xmas tree 包或 NULL 包，如果收到对方的 RST 回复包，那么说明该端口是关闭的；没有收到 RST 包说明该端口可能是开放的或者被屏蔽了。其中 Xmas tree 包是指 flags 中 FIN URG PUSH 被置为 1 的 TCP 包；NULL 包是指所有的 flags 都为 0 的 TCP 包。

**UDP 扫描 (-sU)**

      UDP 扫描用于判断 UDP 端口的情况，向目标主机的 UDP 端口发送探测包，如果收到回复 ICMP port unreachable 就说明该端口是关闭的；如果没有收到回复，那说明该 UDP 端口可能是开放的或者屏蔽的。因此，通过反向排除法的方式来判断哪些 UDP 端口是可能处于开放状态的。

**其他方式 (-sY/-sZ)**

     除了以上几种常用的方式外，Nmap 还支持多种其他的探测方式。例如使用 SCTP INIT/Cookie-ECHO 方式是来探测 SCTP 的端口开放情况；使用 IP protocol 方式来探测目标主机支持的协议类型 (tcp/udp/icmp/sctp 等等)；使用 idle scan 方式借助僵尸主机来扫描目标主机，以达到隐蔽自己的目的；或者使用 FTP bounce scan，借助 FTP 允许的代理服务扫描其他的主机，同样达到隐蔽自己的目的

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjHB7C85hnZBxEdY7XfdialFSs2sqkhU0hAQGG4vDn1nDOdXUGDicfaJ1rTeQDPNJRciaedicADLANghw/640)

三：端口扫描用法

![](https://mmbiz.qpic.cn/mmbiz_png/uN1LIav7oJ8xLmmCxUAIxzgp52kq6Au35axTbLzLuPia6AjIgpPUctQZibSSGePqTcUvoicibQnIsaibicys2ib15oCNg/640?wx_fmt=png)

扫描方式选项

```
-sS/sT/sA/sW/sM:指定使用 TCP SYN/Connect()/ACK/Window/Maimon scans的方式来对目标主机进行扫描。
 
-sU: 指定使用UDP扫描方式确定目标主机的UDP端口状况。
 
-sN/sF/sX: 指定使用TCP Null, FIN, and Xmas scans秘密扫描方式来协助探测对方的TCP端口状态。
 
--scanflags <flags>: 定制TCP包的flags。
 
-sI <zombiehost[:probeport]>: 指定使用idle scan方式来扫描目标主机（前提需要找到合适的zombie host）
 
-sY/sZ: 使用SCTP INIT/COOKIE-ECHO来扫描SCTP协议端口的开放的情况。
 
-sO: 使用IP protocol 扫描确定目标机支持的协议类型。
 
-b <FTP relay host>: 使用FTP bounce scan扫描方式
```

端口参数与扫描顺序

```
-p <port ranges>: 扫描指定的端口
 
实例: -p 22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080,S:9（其中T代表TCP协议、U代表UDP协议、S代表SCTP协议）
 
-F: Fast mode – 快速模式，仅扫描TOP 100的端口
 
-r: 不进行端口随机打乱的操作（如无该参数，nmap会将要扫描的端口以随机顺序方式扫描，以让nmap的扫描不易被对方防火墙检测到）。
 
--top-ports <number>:扫描开放概率最高的number个端口（nmap的作者曾经做过大规模地互联网扫描，以此统计出网络上各种端口可能开放的概率。以此排列出最有可能开放端口的列表，具体可以参见文件：nmap-services。默认情况下，nmap会扫描最有可能的1000个TCP端口）
 
--port-ratio <ratio>: 扫描指定频率以上的端口。与上述--top-ports类似，这里以概率作为参数，让概率大于--port-ratio的端口才被扫描。显然参数必须在在0到1之间，具体范围概率情况可以查看nmap-services文件
```

简单扫描 (nmap ip)

```
nmap   202.207.236.2
```

例如：nmap  202.207.236.2    这个命令会按照 nmap-services 文件中指定的端口进行扫描，然后列出目标主机开放的端口号，以及端口号上运行的服务。在一次简单扫描中，Nmap 会以默认 TCP SYN 扫描方式进行，仅判断目标端口是否开放，若开放，则列出端口对应的服务名称。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzUXQX5HQCvNbBiapwnz2ESq2ryZp1ib2ib9mGORuPeMq0dYSDqEBAKweRw/640?wx_fmt=png)

**探测端口开放过程**：  确定主机在线之后，nmap 会按照 nmap-services 文件中的端口号发送 TCP SYN 报文给主机相应的端口，如果主机回复一个包含 TCP SYN、ACK 的报文，则说明该端口号开放。nmap 会再回复一个 TCP RST 清除连接复位。下面的截图是 nmap 是和目标主机的 80 号端口的探测过程，由此可见，目标主机的 22 号端口属于开放状态！

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzuYHUgQT7mdP2QgdKWRib2BlGKprQlZY0evVB8oA2tlsg3zzibyR0XAaQ/640?wx_fmt=png)

全面扫描 (nmap -A ip)

```
nmap   -A  202.207.236.2
```

例如：nmap  -A  202.207.236.2    这个命令不仅列出目标主机开放的端口号，对应的服务，还较为详细的列出了服务的版本，其支持的命令, 到达目标主机的每一跳路由等信息。在进行完全扫描时，扫描机与目标主机之间存在大量的数据流量交互，扫描时长随之增加。完全扫描不仅仅是 TCP 协议上的通信交互，还有例如 ICMP、HTTP、NBSS、TDS、POP 等等协议的交互，这些协议的交互是因为在完全扫描开始时首先对目标主机的开放端口进行了确认，之后再根据不同对应的不同服务进行服务版本信息探测、账户信息等信息的探测！

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzHAqBjib189LeATz5fKjfDgSYFJ45s6mLCURURXyiaV10ZG6RhuKSbcmw/640?wx_fmt=png)

*   探测主机是否在线：全面扫描时探测主机是否在线和简单扫描完全一致
    
*   探测端口是否打开：全面扫描时探测主机端口开放和简单扫描完全一致
    
*   探测端口服务具体版本：每个协议都不一样，总之就是确定端口开放了之后，和该端口进行更多的数据交互，以获得更多的信息。在下一节的版本探测中有更深入的研究
    
*   探测主机系统：在下一节的系统探测中有更深入的研究。
    

```
nmap -T4 -A -v xx.xx.xx.xx
```

例如：nmap -T4 -A -v 10.96.10.246

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzAzFrkhrPUd7w7e6HaORjAicoHW1LjxJt5shlb0f5PD5pavC8ia9ibL5EQ/640?wx_fmt=png)

全面扫描时数据流量包的截图，确定了哪些端口的协议开启了之后，进行更加深入的探测！

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzBBFkzo1VrbGOjMibC8ibZvzicGEicibsL8n7YPL51QDE96Y8kx1hgibA2Bng/640?wx_fmt=png)

探测指定端口的开放状态
-----------

在默认情况下，Nmap 对端口的扫描方式是从小到大进行的，或者是参照 nmap-services 中文件列出的端口进行扫描。-p 选项可以指定一个端口号或者一个端口范围。若既想扫描 TCP 端口又想扫描 UDP 端口，则需要在端口号前加上 T：或 U：来分别代表 TCP 和 UDP 协议。注意，要既扫描 TCP 又扫描 UDP，则需要指定 - sU 及至少一个 TCP 扫描类型（-sS(半连接扫描)，-sT(全连接扫描) 等），如果没有给定协议限定符，端口号会被加到所有协议列表。

例： nmap  -p  80-445  10.96.10.246     扫描目标主机的 80-445 端口的开放情况

### ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzEVxlLiaBpak0icrNYIvy3gJQStkNaT97bjdGQpQj5zCXBqibOyKpBgINg/640?wx_fmt=png)

从上面的图中可以看到，若只简单的指定一个端口范围，Nmap 会默认以 TCP SYN 方式扫描目标端口，若既想扫描目标 TCP 端口又想扫描 UDP 的端口，则需要指定扫描方式以及端口。

例：nmap  -sS   -sU  -p  T:80,U:445   10.96.10.246     以半连接的 TCP SYN 方式扫描目标主机的 80 端口，以 UDP 方式扫描目标主机的 445 端口

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUz9JasSuLXtVUjgaDBtXVojueibnPL3WmrFmg7fZGwb1tibvdU3EvGGiaXw/640?wx_fmt=png)

探测 N 个最有可能开放的端口
---------------

```
例：nmap -sS -sU --top-ports 100 10.96.10.246
```

参数 - sS 表示使用 TCP SYN 方式扫描 TCP 端口；-sU 表示扫描 UDP 端口；--top-ports 100 表示扫描最有可能开放的 100 个端口（TCP 和 UDP 分别 100 个端口）。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzQQeCFJ4C1mgj66L50bp4awFiabN0hkQYcgAkfvFd0Lgj6hgjFS5T1dw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjHB7C85hnZBxEdY7XfdialFSs2sqkhU0hAQGG4vDn1nDOdXUGDicfaJ1rTeQDPNJRciaedicADLANghw/640)

四：版本侦测

![](https://mmbiz.qpic.cn/mmbiz_png/uN1LIav7oJ8xLmmCxUAIxzgp52kq6Au35axTbLzLuPia6AjIgpPUctQZibSSGePqTcUvoicibQnIsaibicys2ib15oCNg/640?wx_fmt=png)

版本侦测，用于确定目标主机开放端口上运行的具体的应用程序及版本信息。

Nmap 提供的版本侦测具有如下的优点：

*   高速。并行地进行套接字操作，实现一组高效的探测匹配定义语法。
    
*   尽可能地确定应用名字与版本名字。
    
*   支持 TCP/UDP 协议，支持文本格式与二进制格式。
    
*   支持多种平台服务的侦测，包括 Linux/Windows/Mac OS/FreeBSD 等系统。
    
*   如果检测到 SSL，会调用 openSSL 继续侦测运行在 SSL 上的具体协议（如 HTTPS/POP3S/IMAPS）。
    
*   如果检测到 SunRPC 服务，那么会调用 brute-force RPC grinder 进一步确定 RPC 程序编号、名字、版本号。
    
*   支持完整的 IPv6 功能，包括 TCP/UDP，基于 TCP 的 SSL。
    
*   通用平台枚举功能（CPE）
    
*   广泛的应用程序数据库（nmap-services-probes）。目前 Nmap 可以识别几千种服务的签名，包含了 180 多种不同的协议。
    

版本侦测原理
------

版本侦测主要分为以下几个步骤：

1.  首先检查 open 与 open|filtered 状态的端口是否在排除端口列表内。如果在排除列表，将该端口剔除。
    
2.  如果是 TCP 端口，尝试建立 TCP 连接。尝试等待片刻（通常 6 秒或更多，具体时间可以查询文件 nmap-services-probes 中 Probe TCP NULL q|| 对应的 totalwaitms）。通常在等待时间内，会接收到目标机发送的 “WelcomeBanner” 信息。nmap 将接收到的 Banner 与 nmap-services-probes 中 NULL probe 中的签名进行对比。查找对应应用程序的名字与版本信息。
    
3.  如果通过 “Welcome Banner” 无法确定应用程序版本，那么 nmap 再尝试发送其他的探测包（即从 nmap-services-probes 中挑选合适的 probe），将 probe 得到回复包与数据库中的签名进行对比。如果反复探测都无法得出具体应用，那么打印出应用返回报文，让用户自行进一步判定。
    
4.  如果是 UDP 端口，那么直接使用 nmap-services-probes 中探测包进行探测匹配。根据结果对比分析出 UDP 应用服务类型。
    
5.  如果探测到应用程序是 SSL，那么调用 openSSL 进一步的侦查运行在 SSL 之上的具体的应用类型。
    
6.  如果探测到应用程序是 SunRPC，那么调用 brute-force RPC grinder 进一步探测具体服务。
    

版本侦测用法
------

比如目标主机把 SSH 的 22 号端口改成了 2222 端口，那么如果使用普通扫描只会发现 2222 端口是开启的，并不能知道 2222 号端口上运行的程序，通过加参数  -sV  进行版本扫描，可以探测到目标主机上 2222 端口运行的是 SSH 服务

```
-sV: 指定让Nmap进行版本侦测
 
--version-intensity <level>: 指定版本侦测强度（0-9），默认为7。数值越高，探测出的服务越准确，但是运行时间会比较长。
 
--version-light: 指定使用轻量侦测方式 (intensity 2)
 
--version-all: 尝试使用所有的probes进行侦测 (intensity 9)
 
--version-trace: 显示出详细的版本侦测过程信息
```

```
例:  nmap  -sV  10.96.10.246
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzLOg8xOmgzdrtUKIM0xjB9Z9T5fs2R8icwNDjgKzNUTU99lJ6dZpwdHg/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjHB7C85hnZBxEdY7XfdialFSs2sqkhU0hAQGG4vDn1nDOdXUGDicfaJ1rTeQDPNJRciaedicADLANghw/640)

五：OS 侦测

![](https://mmbiz.qpic.cn/mmbiz_png/uN1LIav7oJ8xLmmCxUAIxzgp52kq6Au35axTbLzLuPia6AjIgpPUctQZibSSGePqTcUvoicibQnIsaibicys2ib15oCNg/640?wx_fmt=png)

操作系统侦测用于检测目标主机运行的操作系统类型及设备类型等信息。

Nmap 拥有丰富的系统数据库 nmap-os-db，目前可以识别 2600 多种操作系统与设备类型。

OS 侦测原理
-------

Nmap 使用 TCP/IP 协议栈指纹来识别不同的操作系统和设备。在 RFC 规范中，有些地方对 TCP/IP 的实现并没有强制规定，由此不同的 TCP/IP 方案中可能都有自己的特定方式。Nmap 主要是根据这些细节上的差异来判断操作系统的类型的。

具体实现方式如下：

1.  Nmap 内部包含了 2600 多已知系统的指纹特征（在文件 nmap-os-db 文件中）。将此指纹数据库作为进行指纹对比的样本库。
    
2.  分别挑选一个 open 和 closed 的端口，向其发送经过精心设计的 TCP/UDP/ICMP 数据包，根据返回的数据包生成一份系统指纹。
    
3.  将探测生成的指纹与 nmap-os-db 中指纹进行对比，查找匹配的系统。如果无法匹配，以概率形式列举出可能的系统。
    

OS 侦测用法
-------

```
-O: 指定Nmap进行OS侦测。
 
--osscan-limit: 限制Nmap只对确定的主机的进行OS探测（至少需确知该主机分别有一个open和closed的端口）。
 
--osscan-guess: 大胆猜测对方的主机的系统类型。由此准确性会下降不少，但会尽可能多为用户提供潜在的操作系统
```

```
例： nmap -O 10.96.10.246
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzVHCFXqr6t8PYxticK0gGsPsouBicGQc5TSNPicdR9wQwoOck1vlYuWkHA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjHB7C85hnZBxEdY7XfdialFSs2sqkhU0hAQGG4vDn1nDOdXUGDicfaJ1rTeQDPNJRciaedicADLANghw/640)

六：Nmap 高级用法

![](https://mmbiz.qpic.cn/mmbiz_png/uN1LIav7oJ8xLmmCxUAIxzgp52kq6Au35axTbLzLuPia6AjIgpPUctQZibSSGePqTcUvoicibQnIsaibicys2ib15oCNg/640?wx_fmt=png)

防火墙 / IDS 规避
------------

防火墙与 IDS 规避为用于绕开防火墙与 IDS 的检测与屏蔽，以便能够更加详细地发现目标主机的状况。nmap 提供了多种规避技巧通常可以从两个方面考虑规避方式：数据包的变换 (Packet Change) 和时序变换(Timing Change)

### 分片

将可疑的探测包进行分片处理 (例如将 TCP 包拆分成多个 IP 包发送过去)，某些简单的防火墙为了加快处理速度可能不会进行重组检查，以此避开其检查

### IP 诱骗 (IP decoys)

在进行扫描时，将真实 IP 地址在和其他主机的 IP 地址混合使用 (其他主机需要在线，否则目标主机将回复大量数据包到不存在的数主机，从而实质构成了 DOS 攻击)，以此让目标主机的防火墙或 IDS 追踪大量的不同 IP 地址的数据包，降低其追查到自身的概率。但是，某些高级的 IDS 系统通过统计分析仍然可以追踪出扫描者真实的 IP 地址

### IP 伪装

IP 伪装就是将自己发送的数据包中的 IP 地址伪装成其他主机的地址，从而目标机认为是其他主机与之通信。需要注意的是，如果希望接收到目标主机的回复包，那么伪装的 IP 需要位于统一局域网内。另外，如果既希望隐蔽自己的 IP 地址，又希望收到目标主机的回复包，那么可以尝试使用 idle scan 或匿名代理等网络技术

### 指定源端口

某些目标主机只允许来自特定端口的数据包通过防火墙。例如，FTP 服务器的配置为允许源端口为 21 号的 TCP 包通过防火墙与 FTP 服务器通信，但是源端口为其他的数据包被屏蔽。所以，在此类情况下，可以指定数据包的源端口

### 扫描延时

某些防火墙针对发送过于频繁的数据包会进行严格的侦查，而且某些系统限制错误报文产生的频率。所以，我们可以降低发包的频率和发包延时以此降低目标主机的审查强度

### 其他技术

nmap 还提供其他多种规避技巧，比如指定使用某个网络接口来发送数据包、指定发送包的最小长度、指定发包的 MTU、指定 TTL、指定伪装的 MAC 地址，使用错误检查。

```
-f; --mtu <val>: 指定使用分片、指定数据包的 MTU.
-D <decoy1,decoy2[,ME],...>: 用一组 IP 地址掩盖真实地址，其中 ME 填入自己的 IP 地址。
-S <IP_Address>: 伪装成其他 IP 地址
-e <iface>: 使用特定的网络接口
-g/--source-port <portnum>: 使用指定源端口
--data-length <num>: 填充随机数据让数据包长度达到 Num。
--ip-options <options>: 使用指定的 IP 选项来发送数据包。
--ttl <val>: 设置 time-to-live 时间。
--spoof-mac <mac address/prefix/vendor name>: 伪装 MAC 地址
--badsum: 使用错误的 checksum 来发送数据包（正常情况下，该类数据包被抛弃，如果收到回复，
说明回复来自防火墙或 IDS/IPS）
```

实例：

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzhxsJ8b9ZsKKpyH2ZKPtrgBjgYeiagZho4bspjq1hibDevr2lrgFHItyw/640?wx_fmt=png)

```
nmap -F -Pn -D 10.96.10.100,10.96.10.110,ME  -e eth0  -g 5555 202.207.236.3
 
-F参数表示快速扫描100个端口，-Pn不进行ping扫描，-D表示使用ip诱骗方式掩饰真实ip，使用的是10.96.10.100和10.96.10.110，ME表示自己真实的ip，这里是10.96.10.234，-e 参数指定eth0网卡发送数据包，-g参数指定发送的端口号
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzw6epKfGgJQAELUpepstBZQL4b6YwkTcsLL1eDjVMNticQqWqXvzibaNw/640?wx_fmt=png)

NSE 脚本引擎
--------

NSE 脚本引擎 (Nmap Scripting Engine) 是 nmap 最强大，最灵活的功能之一，允许用户自己编写脚本来执行自动化的操作或者扩展 nmap 的功能。

nmap 的脚本库的路径：/usr/share/nmap/scripts  或  /xx/nmap/scripts/ ，该目录下的文件都是 nse 脚本

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzIGI5SIMJicV2tbRZHOYOVibPaomIKTokJuicZIGbQxs93tIlyWaEYRm3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzUhjSFc1tVWYrA8VZBfVT056CWqwI9LwmeT8q0aybFpfdULgg8jptqg/640?wx_fmt=png)

NSE 使用 Lua 脚本语言，并且默认提供了丰富的脚本库，目前已经包含了 14 个类别的 350 多个脚本。NSE 的设计初衷主要考虑以下几个方面

*   网络发现（Network Discovery）
    
*   更加复杂的版本侦测（例如 skype 软件）
    
*   漏洞侦测 (Vulnerability Detection)
    
*   后门侦测 (Backdoor Detection)
    
*   漏洞利用 (Vulnerability Exploitation)
    

**Nmap 的脚本主要分为以下几类：**

*   Auth：负责处理鉴权证书 (绕过鉴权) 的脚本
    
*   Broadcast：在局域网内探查更多服务去开启情况，如 DHCP/DNS 等
    
*   Brute：针对常见的应用提供暴力破解方式，如 HTTP/HTTPS
    
*   Default：使用 - sC 或 - A 选项扫描时默认的脚本，提供基本的脚本扫描能力
    
*   Discovery：对网络进行更多的信息搜集，如 SMB 枚举，SNMP 查询等
    
*   Dos：用于进行拒绝服务攻击
    
*   Exploit：利用已知的漏洞入侵系统
    
*   External：利用第三方的数据库或资源 。如，进行 whois 解析
    
*   Fuzzer：模糊测试脚本，发送异常的包到目标机，探测出潜在漏洞
    
*   Intrusive：入侵性的脚本，此类脚本可能引发对方的 IDS/IPS 的记录或屏蔽
    
*   Malware：探测目标是否感染了病毒，开启后门等
    
*   Safe：与 Intrusive 相反，属于安全性脚本
    
*   Version：负责增强服务与版本扫描功能的脚本
    
*   Vuln：负责检查目标机是否有常见漏洞，如 MS08-067
    

```
例如：
nmap -script   smb-vuln-ms17-010  192.168.10.34     #可以探测该主机是否存在ms17_010漏洞
nmap --max-parallelism 800 --script http-slowloris scanme.nmap.org  #可以探测该主机是否存在http拒绝服务攻击漏洞
nmap -script http-iis-short-name-brute 192.168.10.34  #探测是否存在IIS短文件名漏洞
nmap -script mysql-empty-password 192.168.10.34       #验证mysql匿名访问
nmap -p 443 -script ssl-ccs-injection 192.168.10.34   #验证是否存在openssl CCS注入漏洞
 
--script=http-waf-detect            #验证主机是否存在WAF
--script=http-waf-fingerprint       #验证主机是否存在WAF
 
nmap --script-brute 192.168.1.1     #nmap可对数据库、SMB、SNMP等进行简单密码的暴力破解
nmap --script-vuln  192.168.1.1     #扫描是否有常见漏洞
 
--script-updatedb         #更新脚本数据库
--script-help             #输入脚本对应的使用方法
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUzib8U105JkIf6cJL9ib7f4aCmqTXoYX3ebQCice0ibKNf7ZvzJ0b6kNbu1g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjHB7C85hnZBxEdY7XfdialFSs2sqkhU0hAQGG4vDn1nDOdXUGDicfaJ1rTeQDPNJRciaedicADLANghw/640)

七：Zenmap 的使用

![](https://mmbiz.qpic.cn/mmbiz_png/uN1LIav7oJ8xLmmCxUAIxzgp52kq6Au35axTbLzLuPia6AjIgpPUctQZibSSGePqTcUvoicibQnIsaibicys2ib15oCNg/640?wx_fmt=png)

Zenmap 是 Nmap 官方提供的图形界面，通常随 Nmap 的安装包发布。Zenmap 是用 Python 语言编写而成的开源免费的图形界面，能够运行在不同操作系统平台上

（Windows/Linux/Unix/Mac OS 等）。Zenmap 旨在为 nmap 提供更加简单的操作方式。简单常用的操作命令可以保存成为 profile，用户扫描时选择 profile 即可；可以方便地比较不同的扫描结果；

| 

Intense scan

 | 

(nmap -T4 -A -v)

 | 

一般来说，Intense scan 可以满足一般扫描

 |
|   
 | 

-T4

 | 

 加快执行速度

 |
|   
 | 

-A

 | 

 操作系统及版本探测

 |
|   
 | 

-v

 | 

 显示详细的输出

 |
| 

Intense scan plus UDP

 | 

(nmap -sS -sU -T4 -A -v)

 | 

即 UDP 扫描

 |
|   
 | 

-sS

 | 

  TCP SYN 扫描

 |
|   
 | 

-sU

 | 

  UDP 扫描

 |
| 

Intense scan,all TCP ports

 | 

(nmap -p 1-65536 -T4 -A -v)

 | 

扫描所有 TCP 端口，范围在 1-65535，试图扫描所有端口的开放情况，速度比较慢。

 |
|   
 | 

-p

 | 

 指定端口扫描范围

 |
| 

Intense scan,no ping

 | 

(nmap -T4 -A -v -Pn)

 | 

非 ping 扫描

 |
|   
 | 

-Pn

 | 

 非 ping 扫描

 |
| 

Ping scan

 | 

(nmap -sn)

 | 

Ping 扫描

优点：速度快。

缺点：容易被防火墙屏蔽，导致无扫描结果

 |
|   
 | 

-sn

 | 

 ping 扫描

 |
| 

Quick scan

 | 

(nmap -T4 -F)

 | 

快速的扫描

 |
|   
 | 

-F

 | 

 快速模式。

 |
| 

Quick scan plus

 | 

(nmap -sV -T4 -O -F --version-light)

 | 

快速扫描加强模式

 |
|   
 | 

-sV

 | 

 探测端口及版本服务信息。

 |
|   
 | 

-O

 | 

 开启 OS 检测

 |
|   
 | 

--version-light

 | 

 设定侦测等级为 2。

 |
| 

Quick traceroute

 | 

(nmap -sn --traceroute)

 | 

路由跟踪

 |
|   
 | 

-sn Ping

 | 

扫描，关闭端口扫描

 |
|   
 | 

-traceroute

 | 

 显示本机到目标的路由跃点。

 |
| 

Regular scan

 | 

规则扫描

 |   
 |
| 

Slow comprehensive scan

 | 

(nmap -sS -sU -T4 -A -v -PE -PP -PS80,443,-PA3389,PU40125 -PY -g 53 --script all)

 | 

慢速全面扫描。

 |

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4bPsKsflkyUIGQBtwbrVON6aKpFXZCqiaJxibicEDVk4vC5BLSRDlk2ksibJPZxwdAWC17hqrUP1qptQ/640?wx_fmt=gif)

END

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4bPsKsflkyUIGQBtwbrVON6aKpFXZCqiaJxibicEDVk4vC5BLSRDlk2ksibJPZxwdAWC17hqrUP1qptQ/640?wx_fmt=gif)

来源：谢公子博客

责编：Vivian

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC60d89fUar3V8e8MgHOumHtqBIO7yiaIAv9MzD1ckia3I7Gp8rJfIMejTZoE2lk3VWX40TzSGcib930g/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC60d89fUar3V8e8MgHOumHtqBIO7yiaIAv9MzD1ckia3I7Gp8rJfIMejTZoE2lk3VWX40TzSGcib930g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dFib5g1fbicqX8MgAr9JCHUz5QSTtbncwyPspv21Dn2OyhRRDaegJGy4k6G68wgdkiaRSuneKTjkialA/640?wx_fmt=png)

由于文章篇幅较长，请大家耐心。如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC60d89fUar3V8e8MgHOumHtc4SX1PlfxlEOmmRNjRoiaKsVyuia3SDOsP4apib12WsxcQuNicsQRbgfVg/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC60d89fUar3V8e8MgHOumHtqBIO7yiaIAv9MzD1ckia3I7Gp8rJfIMejTZoE2lk3VWX40TzSGcib930g/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC60d89fUar3V8e8MgHOumHtqBIO7yiaIAv9MzD1ckia3I7Gp8rJfIMejTZoE2lk3VWX40TzSGcib930g/640?wx_fmt=png)