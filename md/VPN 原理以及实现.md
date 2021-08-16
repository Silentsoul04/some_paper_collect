> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/uM2U4RAJCGdxYvtsrHPbYg)

![](https://mmbiz.qpic.cn/mmbiz_gif/3k9IT3oQhT09IJjs3wGQbICd50va8zMqfnXZfD5LGdibcuOrtia3P4DpMAVfibZ8J4MsbHt0JW20QL8Wh0SO8zpyA/640?wx_fmt=gif)

**作者：0x7F@知道创宇 404 实验室  
时间：2021 年 7 月 21 日**

 **0x00 前言** 

最近在工作中遇到 VPN 的相关问题，之前一直对 VPN 的原理存在一些疑惑，借此机会学习一下 VPN 的原理以及进行实现验证。

由于 VPN 在不同系统下的实现方式不同，为了便于学习和理解，这里我们选择 `Linux` 环境，我本地测试环境使用的是 `Ubuntu 18.04 x64`。

本文从 TUN/TAP 出发，逐步理解 VPN 中的技术细节；并结合 simpletun 源码，进行 VPN 的原理验证。

 **0x01 VPN 是什么** 

VPN 全称为虚拟私人网络 (Virtual Private Network)，常用于连接中、大型企业或团体间私人网络的通讯方法，利用隧道协议（Tunneling Protocol）来达到发送端认证、消息保密与准确性等功能。

比如多地办公的公司，可以使用 VPN 将不同地区连接在同一内网下；或者在家办公的时候也可以通过 VPN 接入公司内网中。

VPN 以 CS 架构运行，工作流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELpEgStj1ic8bMNGBSy6TmxQCn22vXuCLehRHeWUxLPgmL3LGZUiaN6Vpqw/640?wx_fmt=png)

1.VPN 工作流程

在外网的用户可以使用 `vpn client` 连接组织搭建的 `vpn server` 以建立通信隧道，随后便建立了虚拟的私人网络，处于外网的 `worker` 和内网中的 `server` 可以相互通信。

那么我们可以简单理解 VPN，由 `VPN client` 捕获用户发出的报文，封装报文后通过物理网络通信链路将报文发给 `VPN server`，`VPN server` 接收到报文后进行解包，再将其转发给实际的目标，反之同理；VPN 在逻辑层面构建了虚拟网络。

 **0x02 TUN/TAP** 

那么在代码层面 VPN 是如何实现的呢？我们可以先来看看 TUN/TAP。

TUN/TAP 是操作系统内核中的虚拟网络设备，由软件进行实现，向操作系统和应用程序提供与硬件网络设备完全相同的功能。其中 TAP 是以太网设备 (二层设备)，操作和封装以太网数据帧，TUN 则是网络层设备 (三层设备)，操作和封装网络层数据帧。

当应用程序发出报文后，报文将通过操作系统协议栈处理，到达网络设备，硬件网络设备将收到的报文转化为电信号发出，而虚拟网络设备 (TUN/TAP) 不具备实际的物理功能，报文需要上层应用进行处理，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELpXfx0QU0ibhktZVqXySxmtEhVZOpqibp8Hb0vfNo28cPcZZSqYurNr7yw/640?wx_fmt=png)

2. 硬件 / 虚拟网络设备

我们直接使用命令创建 TUN/TAP 设备，并进行测试：

```
# ip tuntap 创建名为 tun0 的 tun 设备
sudo ip tuntap add dev tun0 mod tun
# 为 tun0 配置 ip
sudo ifconfig tun0 192.168.0.10 netmask 255.255.255.0
# 查看 tun0 网卡
ifconfig tun0
```

如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELp1Jetic2ic1DZvl277MdHhDNj8CkF7TukuH888Z3tGYiaPe3603BqnE1iaw/640?wx_fmt=png)

3. 通过命令创建 TUN 设备

在 VPN 中我们可以借助 TUN/TAP 来捕获用户发出的报文。

 **0x03 虚拟通信链路** 

按照 TUN/TAP 的工作特性，我们可以编写程序直接读写虚拟网卡 (也就是物理网卡实际收发报文的过程)，来实现捕获用户数据以及传递用户数据。(TUN 和 TAP 有不同的应用场景，下文我们将以更简单的 TUN 作为例子)

随后，位于不同主机上的程序通过 socket 进行通信，将从虚拟网卡的接收的数据通过 socket 发送给对端，这就是一个 VPN 的雏形了，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELphfFiamqmJodTFUaV4BkhhViblWZ2Ao6tkjoMOPiaDpbCHTOIhFbbdENYQ/640?wx_fmt=png)

4. 虚拟通信链路工作流程

`simpletun` 是这种方案的最小实现 (源码仅 300+ 行，感兴趣的小伙伴可以自行学习)，在源码中实现了创建虚拟网络设备以及 socket 通信，借助 `simpletun` 可以帮助我们快速进行验证。

需要注意一点，`simpletun` 启动后需要我们手动配置虚拟网卡的 ip 地址，当 ip 地址未配置时，两端相互发送数据 (部分操作系统会自动发送) 会造成程序异常退出，所以在代码中添加一个 `sleep(30)` 便于我们配置 ip 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELpMoHicTXm47bbhCoKu5RhLym15n64rIlWLqskaiasRgp5oBVctu48VbYA/640?wx_fmt=png)

5. 在 simpletun 中添加 sleep

在两台 `Ubuntu` 测试环境下配置并进行验证：

```
# A主机
# 编译 simpletun
gcc simpletun.c -Wall -o vpn
# 作为 vpn server 启动，并开启 debug，默认监听 55555
sudo ./vpn -i tun0 -s -d
# 配置 tun 网卡地址
sudo ifconfig tun0 192.168.0.10 netmask 255.255.255.0

# B主机
# 编译 simpletun
gcc simpletun.c -Wall -o vpn
# 作为 vpn client 启动，连接 server，并开启 debug
sudo ./vpn -i tun0 -c 10.11.33.50 -d
# 配置 tun 网卡地址
sudo ifconfig tun0 192.168.0.11 netmask 255.255.255.0
```

此时两台主机位于 `192.168.0.0/24` 虚拟网络网段下，可以相互通信，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELpWwVbA2pFJp0RGZ97LrhQbrNynQ6ptunGiaA8b6tSDNkNnecOkqnicuyQ/640?wx_fmt=png)

6. 虚拟通信链路两端通信

 **0x04 访问内网网段** 

在上文的验证中，我们可以实现两端的虚拟网络搭建和通信，但实际 VPN 的使用场景是需要通过 VPN 访问整个内网网段，在这种使用场景下，VPN server 至少配置有两张物理网卡，其中一张接入内网网段，另一张则连接到互联网。

按照 `0x03 虚拟通信链路` 的链路，VPN client 发送报文到内网主机，VPN server 接收到该报文后，将其写入到虚拟网卡中，随后报文进入 TCP/IP 协议栈，但是由于 IP 地址不是 VPN server 自己，该报文会被丢弃，无法正常进行通信；这里我们需要借助「报文转发」，将内网报文从虚拟网卡转发到内网网卡上。其新的工作流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELp6XGlRAoF7BW9TSVFQjRjmfDxbItWHx7GGTGrkjVl3iblrOCE59fcXoA/640?wx_fmt=png)

7.VPN 访问内网网段

> VPN server 一般会作为内网网关，内网主机无需任何额外配置就可以在虚拟网段下正常工作。

我们按照该流程配置测试环境，复用 `0x03 虚拟通信链路` 中的环境，在 VPN server 上我们使用 docker 模拟内网网段和主机，其环境搭建如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELpQ6sM14pf9zVBSdyBn3ksnVQqfTS7aGHkEx7iaXjD1vhQzWQ9Y6vdoWw/640?wx_fmt=png)

8.VPN 测试环境搭建

然后按照 `0x03 虚拟通信链路` 中的方式，启动 `simpletun` 并使用 `ifconfig` 配置 ip 地址，创建虚拟通信链路；使用如下命令开启报文转发：

```
# 临时开启报文转发
echo "1" > /proc/sys/net/ipv4/ip_forward
```

> 实际上在该测试环境下，docker 会自动开启报文转发

再通过 `iptables` 配置转发策略，如下：

```
# 将入口网卡、来源ip为 192.168.0.0/24 转发至 docker0
sudo iptables -A FORWARD -i tun0 -s 192.168.0.0/24 -o docker0 -j ACCEPT
# 将入口网卡、目的ip为 192.168.0.0/24 转发至 tun0
sudo iptables -A FORWARD -i docker0 -d 192.168.0.0/24 -o tun0 -j ACCEPT
```

> 实际上在该测试环境下，第二条可以不用配置，因为 docker 会自动配置转发策略，会覆盖这条策略

除此之外，为了在 VPN client 可以访问到内网主机，需要手动添加路由：

```
# VPN client 添加内网网段路由，设置为虚拟网络设备 tun0
sudo route add -net 172.17.0.0/24 tun0
```

此时 VPN 配置完成，内网主机和 VPN client 相互连通：

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELpfMGuT5yymJG2mOLYyS22EY3w6L1NGkDXaWGe2WkZNTw7sC4c7iaTJXw/640?wx_fmt=png)

9. 内网主机访问以及验证

 **0x05 拓 展** 

上文中我们使用最小实现验证了 VPN 的工作原理，但是实际场景却比这个复杂很多，这里我们简单抛出一些问题作为拓展学习。

**1.VPN 作为网关？**  
VPN server 一般作为网关进行配置，内网主机不用进行额外配置，也可以把报文发送给 VPN server。

**2.UDP 通信链路？**  
在 `simpletun` 中 VPN server 和 client 之间使用 TCP 进行通信，但是在实际场景一般使用 UDP 进行通信。

当使用 TCP 作为通信隧道时，并且上层应用也使用 TCP，也就是 `tcp in tcp`，当出现丢包时，上层应用的 TCP 和 VPN 通信隧道的 TCP 都会进行重传，从而通信中出现大量的重传报文，降低通信效率；如果在这种情况下，以 UDP 作为通信隧道，`tcp in udp`，丢包后将只由上层应用的 TCP 进行重传。

**3.etc**

 **0x06 总 结** 

最后感谢 rook1e@知道创宇 404 实验室 小伙伴同我一起学习和研究，解决了诸多问题。  

本文从 VPN 原理出发，介绍了关键作用的 TUN/TAP 虚拟网络设备，并结合 `simpletun` 创建了两端的虚拟通信链路，最后配合报文转发，实现并验证了 VPN 的通信工作原理。

VPN 的实现较为简单，但涉及到各种细枝末节的网络知识；这里的最小验证，可以为我们实现更为复杂的 VPN 或基于 VPN 技术的其他项目提供参考。

References:  
https://zh.wikipedia.org/wiki/%E8%99%9B%E6%93%AC%E7%A7%81%E4%BA%BA%E7%B6%B2%E8%B7%AF  
https://zhaohuabing.com/post/2020-02-24-linux-taptun/  
https://www.cnblogs.com/sparkdev/p/9262825.html  
https://serverfault.com/questions/39307/linux-ip-forwarding-for-openvpn-correct-firewall-setup  
https://liuyehcf.github.io/2019/08/25/OpenVPN-%E8%BD%AC%E8%BD%BD/  
https://yunfwe.cn/2018/05/24/2018/%E4%B8%80%E8%B5%B7%E5%8A%A8%E6%89%8B%E5%86%99%E4%B8%80%E4%B8%AAVPN/  
https://github.com/gregnietsky/simpletun

![](https://mmbiz.qpic.cn/mmbiz_gif/3k9IT3oQhT0Z79Hq9GCticVica4ufkjk5xiarRicG97E3oEcibNSrgdGSsdicWibkc8ycazhQiaA81j3o0cvzR5x4kRIcQ/640?wx_fmt=gif)

**往 期 热 门**

(点击图片跳转)

[

![](https://mmbiz.qpic.cn/mmbiz_jpg/3k9IT3oQhT1V8TbxA178dE9IhSA97XJVkH1kx8HvkQKgaGyQwucT4Wl68KzviafEko8yehbjOGy7UBtd9gPhPgw/640?wx_fmt=jpeg)

CVE-2021-33514：Netgear 多款交换机命令注入漏洞







](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650947935&idx=1&sn=2fa12756c9e844ffc8098cab126f29bf&chksm=8079036db70e8a7b0d63a5a84ac2bc2ce46ce6d390192347ace1854e79d04a2e52070165f204&scene=21#wechat_redirect)

  

[

![](https://mmbiz.qpic.cn/mmbiz_jpg/3k9IT3oQhT3KhTHWZI36e63AAqCsMmiaQsIeZ0JBZqY3vKUtJ1WXE97577Rgjl2ZNETPeuN9IBxwtL2QqOEaLVg/640?wx_fmt=jpeg)

赠书 |《404 Paper 精粹》第一期发布啦！







](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650947894&idx=1&sn=7fe7a66b5b1daffeac64b73333e9db3f&chksm=80790304b70e8a128397f499834f42012938e4f09cc537539a9bc7f2e2f5312e5e7812eb81b7&scene=21#wechat_redirect)

  

[

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavzvjP02IScE6fMPaPqow2JZHdciaCMzFSvkABvRwDaictDR5uYIjEtYbOFKrdfNz0cSEMOzjIE7eh2A/640?wx_fmt=jpeg)

利用 Pocsuite3 框架编写 poc 实战案例







](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650947757&idx=1&sn=677b6c36c52ce55225ae30f136b7ba5b&chksm=8079029fb70e8b89ecc9506aff4852d15d1a0237b99f691864ba7328581dae66e78f5f4d6c02&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/3k9IT3oQhT0Z79Hq9GCticVica4ufkjk5xK8te0JrCrcOiatDWNPRndZzq1N80rlbyxU9bGuTvekqEGu5utyHqicicw/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT09IJjs3wGQbICd50va8zMqN2SkNrrQyWIiaCQvodo60ZfrQIhWic0TSeglsSGiboXx1wjbOxwdu5jQw/640?wx_fmt=jpeg)

**觉得不错点个 “在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**