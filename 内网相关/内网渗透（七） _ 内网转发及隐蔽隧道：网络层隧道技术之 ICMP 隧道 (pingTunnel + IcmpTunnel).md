> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489736&idx=2&sn=0cb551ee520860878c2c33108033c00c&chksm=fc781c15cb0f9503f672aa0bd18cb13fef4c60124ba5978ab947c34272b2d8a28c584a99219d&scene=21#wechat_redirect)

**欢迎各位添加微信号：qinchang_198231**

**加入安全 + 交流群**

**和大佬们一起交流安全技术**

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWStK7P85dEbzW3T8Vq05ZCIoViaUWibLxVZcEibYZnvYRe15arfFxEBiblA/640?wx_fmt=gif)

网络层隧道技术之 ICMP 隧道 (pingTunnel/IcmpTunnel)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWStK7P85dEbzW3T8Vq05ZCIoViaUWibLxVZcEibYZnvYRe15arfFxEBiblA/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HaqYkzRr4G8N4wkuSTEL7RPTPQ32BdslJufe96oiatuv6ZZduQGafZ0w/640?wx_fmt=png)

目录

    ICMP 隧道

        使用 ICMP 搭建隧道 (PingTunnel)

        使用 ICMP 搭建隧道 (Icmptunnel)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HaqYkzRr4G8N4wkuSTEL7RPTPQ32BdslJufe96oiatuv6ZZduQGafZ0w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7rcIPiaOpGhxC0LicZoAT7vX9vXicvL86eYMxClIadcXxMJ6YrZHMkVAeu0QFJgnFsJqHm0Ohn1ZVbg/640?wx_fmt=png)

ICMP 隧道

  

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7rcIPiaOpGhxC0LicZoAT7vXP4kmnG0ITetrvpfxbsHWClNnbEDw4YnibREnpzCP0k1XAKeLqCZDGTg/640?wx_fmt=png)

ICMP 隧道简单实用，是一个比较特殊的协议。在一般的通信协议里，如果两台设备要进行通信，肯定需要开放端口，而在 ICMP 协议下就不需要。最常见的 ping 命令就是利用的 ICMP 协议，攻击者可以利用命令行得到比回复更多的 ICMP 请求。在通常情况下，每个 ping 命令都有相应的回复与请求。

在一些网络环境中，如果攻击者使用各类上层隧道 (例如：HTTP 隧道、DNS 隧道、常规正 / 反向端口转发等) 进行的操作都失败了，常常会通过 ping 命令访问远程计算机，尝试建立 ICMP 隧道，将 TCP/UDP 数据封装到 ICMP 的 ping 数据包中，从而穿过防火墙(防火墙一般不会屏蔽 ping 的数据包)，实现不受限制的访问访问。

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7rcIPiaOpGhxC0LicZoAT7vX9vXicvL86eYMxClIadcXxMJ6YrZHMkVAeu0QFJgnFsJqHm0Ohn1ZVbg/640?wx_fmt=png)

使用 ICMP 搭建隧道 (PingTunnel)

  

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7rcIPiaOpGhxC0LicZoAT7vXP4kmnG0ITetrvpfxbsHWClNnbEDw4YnibREnpzCP0k1XAKeLqCZDGTg/640?wx_fmt=png)

PingTunnel 是一款常用的 ICMP 隧道工具，可以跨平台使用，为了避免隧道被滥用，还可以为隧道设置密码。

**拓扑图如下：**

192.168.10.X 模拟公网地址，Web 服务器模拟企业对外提供 Web 服务的机器，该机器可以通内网，同时向公网提供服务。内网存在一台 WIndows 机器，Web 服务器可以与该机器连接。现在我们获取到了 Web 服务器的权限，想用 ICMP 搭建通往内网的隧道，连接内网 Windows 的 3389 端口。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWib4IiaQRtXB7HRDybxns2Gicaibsuhdb6M3pYrrLoXKmyh2j1ibpkQwklXg/640?wx_fmt=png)

**PingTunnel 的安装**

```
#安装libpcap的依赖环境
yum -y install byacc
yum -y install flex bison
 
#安装libpcap依赖库
wget http://www.tcpdump.org/release/libpcap-1.9.0.tar.gz
tar -xzvf libpcap-1.9.0.tar.gz
cd libpcap-1.9.0
./configure
make && make install
 
#安装PingTunnel
wget http://www.cs.uit.no/~daniels/PingTunnel/PingTunnel-0.72.tar.gz
tar -xzvf PingTunnel-0.72.tar.gz
cd PingTunnel
make && make install
```

**在 Web 服务器的操作**

```
ptunnel -x shuteer     #-x 指定连接密码
```

**![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWOcQib76Rd5hicnDZL2xuxIb9qb0PCtYgic8aRkwrXUUc1JdnavmkOkjHw/640?wx_fmt=png)**

**VPS 的操作**

```
ptunnel -p 192.168.10.129 -lp 1080 -da 10.10.10.10 -dp 3389 -x shuteer
    -p 指定ICMP隧道另一端的IP
    -lp：指定本地监听的端口
    -da：指定要转发的目标机器的IP
    -dp：指定要转发的目标机器的端口
    -x：指定连接密码
```

**![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWJ065Wz9fPVvJibUwB1ejXaAQHFbggibuUUb8rzszQ7purNgc29Pe1mpA/640?wx_fmt=png)**

然后我们只需要远程连接 192.168.10.129 的 1080 端口就相当于连接了 10.10.10.10 的 3389 端口了。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWsVkYfSNQDBvdBCBsOv18kGCXK2818KlxobhEWXOy1l6cfRLibQ21aHw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjW9bmXnYNNJN3C87XQZSFIRFBe5QYN7OLzy69U1ibPhTSD5T7PLNuiawOA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7rcIPiaOpGhxC0LicZoAT7vX9vXicvL86eYMxClIadcXxMJ6YrZHMkVAeu0QFJgnFsJqHm0Ohn1ZVbg/640?wx_fmt=png)

使用 ICMP 搭建隧道 (Icmptunnel)

  

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7rcIPiaOpGhxC0LicZoAT7vXP4kmnG0ITetrvpfxbsHWClNnbEDw4YnibREnpzCP0k1XAKeLqCZDGTg/640?wx_fmt=png)

适用场景：目标机器是 Linux 服务器的情况

*   攻击机：Redhat7  192.1.68.10.20
    
*   靶机：Centos7  192.168.10.13
    

ICMP 隧道是指将 TCP 连接通过 ICMP 包进行隧道传送的一种方法。

icmptunnel 是一个将 IP 流量封装到 ICMP echo 请求和回复（ping）包中的隧道工具，是在允许 ping 的网络中进行拓展、绕过防火墙的一种半隐蔽方式。虽然 ICMP echo 流量在网络边界通常会被过滤，但这种方法仍然可能对从企业内网出连到互联网的技术有一定帮助。

**icmptunnel 的安装**

分别在服务端和客户端安装 icmptunnel，安装过程如下

```
git clone https://github.com/jamesbarlow/icmptunnel.git
make
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWxaiaeT94WzBCicUib6FWnl0AMFzROqZ17MbXMibboalgq6ykA04kPAhurw/640?wx_fmt=png)

**服务端的操作：**

```
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all      #禁用icmp回复
./icmptunnel -s                                       #监听
重新打开一个命令行窗口
ifconfig tun0 10.0.0.1 netmask 255.255.255.0          #添加tun0网卡，分配隧道地址10.0.0.1/24
```

**![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWjUM0YWpfHC924Wkz6jGeeW0YicMf0QyK5WSxdPzuGUXbeUF3wgDwlKw/640?wx_fmt=png)**

**![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWP9HDzSHcrt2FPBuwejW96Wv9Wj4lCja5EkyIDzBiawMIKgmhZtWz2WQ/640?wx_fmt=png)**

**客户端的操作：**

```
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all      #禁用icmp回复
./icmptunnel 192.168.10.20                            #连接服务端
重新打开一个命令行窗口
ifconfig tun0 10.0.0.2 netmask 255.255.255.0          #添加tun0网卡，分配隧道地址10.0.0.2/24
```

**![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWOat5N8dRQHZRwoP0OrH3EMiaPAicNJcKYjga8t8ICxkCb0VbpmjGD4rw/640?wx_fmt=png)**

至此，客户端和服务端已经打通了一条 ICMP 隧道。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWnkgHgibh7HdZ3wlicdkib3x6FeUuRXlOLja2H2lv426Mz1mUSozwUgnQA/640?wx_fmt=png)

接下来，在客户端和服务端分别用隧道的 ip 地址进行 SSH 连接。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWVQhJiaPaPCfbiabaAwMqsnfbMFk2N1ySp9vFxsmLCgHfUY1D7Gyc5CRA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWnRnX5pgxzEqglAdSAyXpsibeeoSiccS139qicytktP7wRZTPJ0icOIeCLA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWhahicKJiaaScf1o1l9R4ST9k2Juhib1Mqe4FS94mSXQQnuJeYlUsrBmuA/640?wx_fmt=png)

责编：Vivian

来源：谢公子博客

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFh6DjmDpKicfAIGpl3KlQu836ZrYWP3g2wg6QgtZMqfPCJxskmzMPuI26YzibgcyUXvVEqLicyXORGdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFh6DjmDpKicfAIGpl3KlQu83dmrribxWJ5Q0Culb5RicgtfQ8wYT6icvHptHmcEaQtpB1ib3Vyyscuxovg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFh6DjmDpKicfAIGpl3KlQu83ianSic5ut6IwHRl5soWKiba6Xtnt0kWMNb1Jke8afZFSt9vLpZA6rBRZQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFjGibCQezQKY4NzE1WGn6FBCbq3pQVl0oONnYXT354mlVw0edib6X6flYib9JRTic4DTibgib15WZC7sDUA/640?wx_fmt=png)

[内网渗透（六） | 工作组和域的区别](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489205&idx=1&sn=24f9a2e0e6b92a167f3082bb6e09c734&chksm=fc781268cb0f9b7e3c11d19a9fb41567124055eb0e8dd526cbbaf1e9393ff707f9fa9d10c32b&scene=21#wechat_redirect)  

[内网渗透（五） | AS-REP Roasting 攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489128&idx=1&sn=dac676323e81307e18dd7f6c8998bde7&chksm=fc7812b5cb0f9ba3a63c447468b7e1bdf3250ed0a6217b07a22819c816a8da1fdf16c164fce2&scene=21#wechat_redirect)

[内网渗透 | 内网穿透工具 FRP 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489057&idx=3&sn=f81ef113f1f136c2289c8bca24c5deb1&chksm=fc7812fccb0f9beaa65e5e9cf40cf9797d207627ae30cb8c7d42d8c12a2cb0765700860dab84&scene=21#wechat_redirect)  

[内网渗透（四） | 域渗透之 Kerberoast 攻击_Python](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488972&idx=1&sn=87a6d987de72a03a2710f162170cd3a0&chksm=fc781111cb0f98070f74377f8348c529699a5eea8497fd40d254cf37a1f54f96632da6a96d83&scene=21#wechat_redirect)  

[内网渗透（三） | 域渗透之 SPN 服务主体名称](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)  

[内网渗透（二） | MSF 和 CobaltStrike 联动](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[内网渗透 | 域内认证之 Kerberos 协议详解](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)  

[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)