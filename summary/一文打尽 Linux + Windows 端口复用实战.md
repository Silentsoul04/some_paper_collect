> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/w2ayq3i0xcKHHS2YgkeyuA)

> 本文作者：**Spark**（Ms08067 内网安全小组成员）

Spark 微信（欢迎骚扰交流）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xzs5KHPYp6lazzH2MyicaiaJibLUoTTgj9dPyF6iaSf4PguTTiboumy7RNiaQ/640?wx_fmt=jpeg)

  
       **定义**：端口复用是指不同的应用程序使用相同端口进行通讯。  

**场景**：内网渗透中，搭建隧道时，服务器仅允许指定的端口对外开放。利用端口复用可以将 3389 或 22 等端口转发到如 80 端口上，以便外部连接。

示意图：

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xBt59UfORCe07mr5tiaoG39icMC7icGnwoO9EdfgA7NQhDMz0bvicEbMLhw/640?wx_fmt=png)

功能：

*   端口复用可以更好地隐蔽攻击行为，提高生存几率。
    
*   端口复用有时也用作通道后门。
    

特点：

*   端口复用在系统已开放的端口上进行通讯，只对输入的信息进行字符匹配，不对网络数据进行任何拦截、复制类操作，所以对网络数据的传输性能几乎没有影响。
    

**一、Linux 端口复用**

**1. 概述**  

使用 iptables 实现端口复用，使用 socat 进行连接。

使用该方法开启的端口复用为有限的端口复用，复用后端口正常业务会受影响，仅用于后门留存，或者临时使用，不建议长期使用。

**2. 原理**

**(1) iptables**

iptables 是 linux 下的防火墙管理工具。

*   真正实现防火墙功能的是 netfilter，它是 linux 内核中实现包过滤的核心。
    
*   免费。
    
*   可实现封包过滤、**封包重定向**和**网路地址转换（NAT）**等功能。  
    

**(2) 数据通过防火墙流程**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xiaHM7sejfMHoPokEZgkYPicibs3v5hJDA78GuPX6tnGNkuWus46qmibVkg/640?wx_fmt=png)

**(3) 链**

链是一些按顺序排列的规则的列表。

*   PREROUTING 链——对数据包作路由选择前应用此链中的规则（所有的数据包进来的时侯都先由这个链处理）
    
*   INPUT 链——进来的数据包应用此规则链中的策略
    
*   OUTPUT 链——外出的数据包应用此规则链中的策略
    
*   FORWARD 链——转发数据包时应用此规则链中的策略
    
*   POSTROUTING 链——对数据包作路由选择后应用此链中的规则（所有的数据包出来的时侯都先由这个链处理）
    

**(4) 表**

表由一组预先定义的链组成。

*   filter 表——用于存放所有与防火墙相关操作的默认表。通常用于过滤数据包。
    
*   nat 表——用于网络地址转换
    
*   mangle 表——用于处理数据包
    
*   raw 表——用于配置数据包，raw 中的数据包不会被系统跟踪。
    

**(5) 链和表的关系及顺序**

*   PREROUTING: raw -> mangle -> nat
    
*   INPUT: mangle -> filter
    
*   FORWARD: mangle -> filter
    
*   OUTPUT: raw -> mangle -> nat -> filter
    
*   POSTROUTING: mangle -> nat
    

**(6) 表和链的关系**

实际使用中是从表作为操作入口：

*   raw 表：PREROUTING，OUTPUT
    
*   mangle 表：PREROUTING，INPUT，FORWARD，OUTPUT，POSTROUTING
    
*   nat 表：PREROUTING，OUTPUT，POSTROUTING
    
*   filter 表：INPUT，FORWARD，OUTPUT
    

**(7) 添加规则**

```
iptables ‐t 表名 <‐A/I/D/R> 规则链名 [规则号] <‐i/o 网卡名> ‐p 协议名 <‐s 源ip、源子网> ‐‐sport 源端口 <‐d 目标ip/目标子网> ‐‐dport 目标端口 ‐j 动作
```

**(8) socat**

Socat 是 linux 下的一个多功能的网络工具，名字来由是 socket cat。其功能与 netcat 类似，可以看做是 netcat 的加强版。

**3. 指令速查**

**(1) 配置端口复用及开关规则**  

*   目标主机上创建新转发链
    
*   设置复用规则（设置转发规则）
    
*   设置开关规则（接受约定字符后规则生效 / 失效）
    
*   约定字符尽量复杂
    

```
# 新建端口复用链
iptables -t nat -N LETMEIN
# 端口复用规则
iptables -t nat -A LETMEIN -p tcp -j REDIRECT --to-port 22
# 开启端口复用开关
iptables -A INPUT -p tcp -m string --string 'threathuntercoming' --algo bm -m recent --set --name letmein --rsource -j ACCEPT
# 关闭端口复用开关
iptables -A INPUT -p tcp -m string --string 'threathunterleaving' --algo bm -m recent --name letmein --remove -j ACCEPT
# 开启端口复用
iptables -t nat -A PREROUTING -p tcp --dport 8000 --syn -m recent --rcheck --seconds 3600 --name letmein --rsource -j LETMEIN
```

**(2) 使用 socat 连接**

使用 socat 发送约定口令至目标主机打开端口复用开关

```
echo threathuntercoming | socat ‐ tcp:192.168.245.135:8000
```

使用完毕后，发送约定关闭口令至目标主机目标端口关闭端口复用

```
echo threathunterleaving | socat ‐ tcp:192.168.245.135:8000
```

**4. 实验**

*   Target：Ubuntu 16.04 x64
    
*   IP：192.168.245.135
    
*   开启 8000 端口的 web 服务
    
*   开启 22 端口
    

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xA5KyglhJHc6ib50X9UlpicGHm0ibWHgWntODJxAmE8zEiaMIBSbzYUr4Sw/640?wx_fmt=png)

配置端口复用：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1x7UfibwT1fbncvI1EtM3NBR0ufykxLqrzGsmlNASBmjHibOkshxkWfGEA/640?wx_fmt=png)

*   Attacker：Kali 2020 x64
    
*   IP：192.168.245.130
    

使用 socat 连接：

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xYZElk9MV2JZRhrib4P3jwzAia1bgVpUJQQJO83vR95vcIIr4Q8q4iaaicg/640?wx_fmt=png)

此时 ssh 可以通过 8000 访问，但是 8000 端口的正常 web 业务受到影响：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xNKa5K9SXHz2XqbutDScGmopicUTZuAGxribpF6p8CJJzGSmgRKVNQrug/640?wx_fmt=png)

使用 socat 断开连接，ssh 无法再连接，但是 8000 端口回复正常：

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xrMmCaI0PAKXLUEabnQ9LyB5OtJNv9TvccTziaXTkA7XLicR4PGnfgndg/640?wx_fmt=png)

**5. 参考链接**

*   https://www.jianshu.com/p/12a24a95fe2c
    
*   https://www.freebuf.com/articles/network/137683.html
    

**二、Windows 端口复用**

**1. 概述**

使用 HTTP.sys 中的 Net.tcp Port Sharing 服务，配合 WinRM 实现端口复用。

*   优点：HTTP.sys 为 windows 原生机制，WinRM 为 windows 自带功能，动作较小，不易触发主动防御。
    
*   需要管理员权限。
    

**2. 原理**

**(1) HTTP.sys**

HTTP.sys 是 Microsoft Windows 处理 HTTP 请求的内核驱动程序。

*   为了优化 IIS 服务器性能
    
*   从 IIS6.0 引入（即 Windows Server 2003 及以上版本）
    
*   IIS 服务进程依赖 HTTP.sys
    

HTTP.sys 监听 HTTP 流量，然后根据 URL 注册的情况去分发，以实现多个进程在同一个端口监听 HTTP 流量。微软公开了 HTTP Server API 库，Httpcfg、Netsh 等都是基于它的。

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xaibJgyAWZGp0Z7BwLeCBJ8um8va7QaGnYkiarna7AqYmtsByQkxibQruQ/640?wx_fmt=png)

整个过程描述如下：

**Step 1. 注册：**IIS 或其他应用使用 HTTP Server API 时，需要先在 HTTP.sys 上面注册 url prefix，以监听请求路径。

**Step 2. 路由：**HTTP.sys 获取到 request 请求，并分发这个请求给注册当前 url 对应的应用。

**(2) Net.tcp Port Sharing**

Net.tcp Port Sharing 服务是 WCF（Windows Communication Foundation，微软的一个框架）中的一个新系统组件，这个服务会开启 Net.tcp 端口共享功能以达到在用户的不同进程之间实现端口共享。这个机制的最终是在 HTTP.sys 中实现的。目前将许多不同 HTTP 应用程序的流量复用到单个 TCP 端口上的 HTTP.sys 模型已经成为 windows 平台上的标准配置。

在以前的 web 应用中，一个 web 应用绑定一个端口，若有其他应用则需要绑定其他的端口才能实现监听。如下图所示，Web Application 1 绑定了 80 端口后，Web Application 2 再去绑定 80 端口会出错。

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1x2TTkS8juRMucfVOwibfXmJlYMicBC2jtxkQl3F4cu46LZlicQ2B77kAaA/640?wx_fmt=png)

现在使用微软提供的 NET.tcp Port Sharing 服务，只要遵循相关的开发接口规则，就可以实现不同的应用共享相同的 web 服务器端口。如下图中 Web Application 1 和 Web Application 2 同时绑定在 80 端口。

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xcXb5Lbt7ibZ2tzuI2pWia6s15UvibmvMdR6mbMPxz5ibgSmwPx6RBUxia9Q/640?wx_fmt=png)

**(3) WinRM**

WinRM 全称是 Windows Remote Management，是微软服务器硬件管理功能的一部分，能够对本地或远程的服务器进行管理。WinRM 服务能够让管理员远程登录 windows 操作系统，获得一个类似 telnet 的交互式命令行 shell，而底层通讯协议使用的正是 HTTP。

事实上，WinRM 已经在 HTTP.sys 上注册了名为 wsman 的 url 前缀，默认监听端口 5985。因此，在安装了 IIS 的边界 windows 服务器上，开启 WinRM 服务后修改默认 listener 端口为 80 或新增一个 80 端口的 listener 即可实现端口复用，可以直接通过 80 端口登录 windows 服务器。

**3. 指令速查**

查询当前注册 url 前缀：

```
netsh http show servicestate
```

**(1) 开启 winrm 服务**

Windows 2012 及以上：winrm 默认启动并监听了 5985 端口。

Windows 2008：需要手动启动 winrm。

```
winrm quickconfig ‐q
```

**(2) 增加 80 端口复用**

```
winrm set winrm/config/service @{EnableCompatibilityHttpListener="true"}
```

**(3) 更改 winrm 为 80 端口**

*   默认 5985 端口开启，不需要更改端口。
    
*   默认 5985 端口不开启，则更改 winrm 为 80 端口，否则会因端口改变而引起管理员关注。
    

```
winrm set winrm/config/Listener?Address=*+Transport=HTTP @{Port="80"}
```

**(4) 攻击机也需要启动 winrm 并设置信任连接**

```
# 启动winrm
winrm quickconfig ‐q
# 设置信任主机地址
winrm set winrm/config/Client @{TrustedHosts="*"}
```

**(5) 连接使用 winrs 命令接口连接远程 winrm 服务执行命令，并返回结果**

winrs，Windows Remote Shell，windows 远程 shell，是 winrm 的一个组件。

```
winrs ‐r:http://www.aabbcc.com ‐u:administrator ‐p:Password [命令]
```

**4. 实验**

*   Target：Windows Server 2008 R2 x64
    
*   IP：192.168.245.133
    

开启 IIS 服务。

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xXE7m77TNWrnkPTCDuBeibxyqYLPutocQMTVibOjMMucXcuGwibaqtDSqQ/640?wx_fmt=png)

先看一下当前注册的 url 前缀：

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1x2chc7aWGtwl7nuMfQ0rwfbjqW2yI4nicuWXjFZyJBetvtPRj4syKfuw/640?wx_fmt=png)

启动 winrm：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xib2RWxK29aJ3xeBSPIRGzINNr62KRSicAXx6fIPMFu6GcN3BnrmQrbBg/640?wx_fmt=png)

再看一下注册的 url 前缀，发现 winrm 已注册：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xQLjY43hmymYibxqhCFiclXw9WptlNC6QAaialNrPB8catRhmoZC7eYguQ/640?wx_fmt=png)

看一下端口情况：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xcpQlIhpKxSYpBfV42Vltqw8Y3s4W11h96QyhWW02e8S69zJNpEP4iaQ/640?wx_fmt=png)

增加 80 端口复用：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xbsa30d76dDJ8RYXIeTQ5sZPiac8ThEkGNBqX5F7gI2THLMcHEibwpPDw/640?wx_fmt=png)

更改 winrm 为 80 端口：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xpAINMpdBuhxtYA4ZKMjvGvL0HhqslbMzJSoHKy6fic6AhYxhcejDA0Q/640?wx_fmt=png)

再看一下端口情况，发现 5985 端口已关闭：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xeDQpltFKaxib2BQm28MJBfUYzuLCXZWbwTvfHESmAgribkMs8zbqU18w/640?wx_fmt=png)

*   Attacker：Windows Server 2008 R2 x64
    
*   IP：192.168.245.134
    

启动 winrm 并设置信任主机地址：

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xORNkKIB5W76VSnDDk6x3iboX8MOYAHxNQ7Ey1Llfl4icWjLXz45s7rgg/640?wx_fmt=png)

使用 winrs 远程执行命令：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xmUfkYmgOYC6arl7nQrO0ykJbnACYFalN0SwnhtO5uZ3OGk5Nyok2QA/640?wx_fmt=png)

执行 cmd 命令可获取交互式 shell：  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xbCic6bJdOhib0FoN8jKGKWkfFdC1Xvj71MTaR4ZHJtUicWSIhWCtcVLMQ/640?wx_fmt=png)

此时 IIS 的正常服务并未受到影响：

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xgCJla2icwH40XVYLwVf8T2ibIoT5v3g1ToBIL14AMFTZHRS47Ny5aXEw/640?wx_fmt=png)

**5. 提升权限（未亲测）**

WinRM 服务也是受 UAC 影响的，所以本地管理员用户组里面只有 administrator 可以登录，其他管理员用户是没法远程登录 WinRM 的。要允许本地管理员组的其他用户登录 WinRM，需要修改注册表设置。

```
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

修改后，普通管理员登录后也是高权限。

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xvPicuWX2NAibakeLcD0LB5JN2icXyuZgzs7AQRdagzmXBDQDAFpj3Bulw/640?wx_fmt=png)

**6.Hash 登录（未亲测）**

系统自带的 winrs 命令登录时需要使用明文账号密码，那很多场景下尤其是 windows 2012 以后，经常只能抓取到本地用户的 hash，无法轻易获得明文密码。因此需要实现一款支持使用 NTLM hash 登录的客户端，使用 python 来实现不难。

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9ZzfluJuFOwazw8MppYN1xbOsXOmaoYQc3kwmYz1iaztWw728HTKH4297o0lPbGhWo1DIjPic1N03w/640?wx_fmt=png)

**7. 参考链接**

*   https://www.freebuf.com/articles/web/142628.html
    
*   https://paper.seebug.org/1004/
    

**扫描二维码，****加入内网小组，一起畅游内网！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9l9FNcvFsJZ3BvAGHhTVOzKWkTtDU8iaQePhu5BbEibDolILr4Qrh9qa4f0xibBc0b9814Uiaq604kUQ/640?wx_fmt=png)

**扫描下方****二维码学习更多安全知识！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

 ![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)

**目前 30000 + 人已关注加入我们**

![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)