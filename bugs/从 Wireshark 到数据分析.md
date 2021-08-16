\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/NFom2T7WBW-AaezT5BxNDw)

**0x00 写在前面**  

    工欲善其事必先利其器！熟悉掌握一种神器对以后的工作必然是有帮助的，下面我将从简单的描述 Wireshark 的使用和自己思考去写，若有错误或不足还请批评指正。

**0x01Wireshark 介绍**
====================

   Wireshark（前称 Ethereal）是一个网络封包分析软件。网络封包分析软件的功能是撷取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark 使用 WinPCAP 作为接口，直接与网卡进行数据报文交换。在过去，网络封包分析软件是非常昂贵的，或是专门属于盈利用的软件。Ethereal 的出现改变了这一切。在 GNUGPL 通用许可证的保障范围底下，使用者可以以免费的代价取得软件与其源代码，并拥有针对其源代码修改及客制化的权利。Ethereal 是全世界最广泛的网络封包分析软件之一。（百度百科）

  官网：https://www.wireshark.org

**0x02Wireshark 使用**
====================

   首先从官网根据自己的环境下载对应的软件版本，下一步安装即可，建议不要安装在 C 盘。

   接下来打开 Wireshark，可以看到设别的网卡信息，选择需要抓包的网卡双击即可。或者按 Ctrl+K，勾选需要抓包的网卡，一般情况都会选择 WLAN 点击 Start 开始抓包。从流量波形图可以看到弯曲起伏则表示有流量，直线则没有流量。

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dOicVSvKrZSia818ZZQz9vKAlvCfISyI4eOl1TlwgwASlayLQOkaVB7icw/640?wx_fmt=png)

**0x03 Wireshark 语法**
=====================

  1. 过滤 MAC 地址

eth.addr == 00:71:cc:9a:28:93// 过滤目标或源地址是 00:71:cc:9a:28:93 的数据包

eth.src == 00:71:cc:9a:28:93// 过滤源地址是 00:71:cc:9a:28:93 的数据包

eth.dst == 00:71:cc:9a:28:93// 过滤目标地址是 00:71:cc:9a:28:93 的数据包

  2. 过滤 VLAN

vlan.id == 1024        // 过滤 VLANID 为 1024 的数据包

vlan.id\_name ==yunzui // 过滤 VLAN 名为 1024 的数据包

  3.IP 过滤

// 源地址过滤

ip.src == 8.8.8.8

ip.src eq 8.8.8.8

// 目标地址过滤

ip.dst == 8.8.8.8

ip.dst eq 8.8.8.8

//ip 地址过滤。不论源还是目标

ip.addr == 8.8.8.8

ip.addr eq 8.8.8.8

  4. 端口过滤

tcp.port == 8888

udp.port eq 8888

tcp.dstport == 8888// 只显 tcp 协议的目标端口 8888

tcp.srcport == 8888// 只显 tcp 协议的来源端口 8888

// 过滤端口范围

tcp.port >= 1 andtcp.port <= 8888

  5. 常用协议过滤

tcp            // 只显示 TCP 协议的数据流

udp           // 只显示 UDP 协议的数据流

arp           // 只显示 ARP 协议的数据流

icmp          // 只显示 ICMP 协议的数据流

http          // 只显示 HTTP 协议的数据流

smtp         // 只显示 SMTP 协议的数据流

ftp           // 只显示 FTP 协议的数据流

dns           // 只显示 DNS 协议的数据流

……

排除 HTTP 包，如! http 或 not http

 **6.HTTP** **模式过滤**

http.request.method== “GET”

http.request.method== “POST”

http.request.uri ==“/img/logo-edu.gif”

http contains “GET”

http contains“HTTP/1.”

// GET 数据包

http.request.method== “GET” && http contains “Host: ”

http.request.method== “GET” && http contains “User-Agent: ”

// POST 数据包

http.request.method== “POST” && http contains “Host: ”

http.request.method== “POST” && http contains “User-Agent: ”

// HTTP 请求数据包

http.request.method== "POST" && http contains "Java/1.8.0\_121"

// HTTP 响应数据包

http contains“HTTP/1.1 200 OK” && http contains “Content-Type: ”

  7. 运算符

less than：lt

less and equal：le

equal：eq

great then：gt

great and equal：ge

not equal：ne

  8. 连接符

and，or

如 tcp.port == 8888 andip.addr = 88.88.88.88

**0x04 Wireshark 功能**
=====================

   1. 数据包的结构

第 1 行：数据包整体概述，内容比较多

第 2 行：数据链路层详细信息，主要为 mac 地址

第 3 行：网络层详细信息，主要的是双方的 IP 地址

第 4 行：传输层的详细信息，主要的是双方的端口号

第 5 行：TCP 或 UDP 是传输的 DATA，DNS 这是域名的相关信息

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dfB9JStIODWL4uGTxENscichupZEIAZSkTobq5yLAoR0nflDVwnHQQJA/640?wx_fmt=png)

    2.wireshark 着色规则

  在菜单栏中点开视图中的着色规则就可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dvNia180icTp2kYia0qGj1Iv4dxC1VSuONDNiciaBRyKzkgjTVYYl4ic7XzCg/640?wx_fmt=png)

    3. 数据包的统计分析

  协议分级统计功能可以查看所选包协议的分布情况，可以分析者帮助识别可疑协议，和不正常的网络应用程序，提高分析效率。

  在菜单栏中点开统计中的协议分级（P）就可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dpWOZvmT5kHP9t6NBBv9NA59PHOpY7E9LG6aaodxOD8AeMTO0mJaXZw/640?wx_fmt=png)

  在 Endpoints 窗口中，可以通过排序 Bytes 和 Tx Bytes 来判断占用带宽最大的主机

  在菜单栏中点开统计中的 Endpoints 就可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dSMxzQtibQbmc4ToicYbqzEUVFQlx1AUOeTdbs88WI7S289RC66pgSic1A/640?wx_fmt=png)

  Conversions 窗口可以看到两个主机之间发送 / 接收数据包的数量、字节大小以及数据的流向情况，也可以通过排序来判断占用最大带宽的主机。

  在菜单栏中点开统计中的 Conversions 就可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dYAJjq3dWbozVGZgJ9iaAoqwYzjmwRibqRVOrzJM4Vop2vcnIFaamQxHA/640?wx_fmt=png)

    4. 追踪数据流

  当分析到某条数据包对于的数据流查看。可以选中数据，右键选择追踪流。里面就会有 tcp 流、udp 流、ssl 流、http 流。数据包属于哪种流就选择对应的流。

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dtibqibib4NpzEIp6hWmczXSlfEKIXTNWwicCs1WCiaQ6Rict4gDQk5E5ntDQ/640?wx_fmt=png)

**0x05 实战分析**
=============

  攻防世界中级数据分析题

  题目：黑客通过 wireshark 抓到管理员登陆网站的一段流量包（管理员的密码即是答案)。flag 提交形式为 flag{XXXX}

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dbqics0GR4RV4Ol4fJtZCnxX3Lp7F4KPljNr79mGEVGUEuJDISVhIibZg/640?wx_fmt=png)

  下载题目数据包，根据题目要求对数据包进行分析

  提取题目 keyword ：网站（HTTP） 登陆（POST）

  打开数据包过滤对应数据流

```
http.request.method == "POST"

```

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dFdGPr3LTkfE77yXDnVhGianQ3WshBUuJ0QWXwOqKHum15AtQFia62L1A/640?wx_fmt=png)

  追踪 HTTP 数据流，获取管理员密码

  ffb7567a1d4f4abdffdb54e022f8facd

![](https://mmbiz.qpic.cn/mmbiz_png/MjmKb3ap0hDyLibAuZicOJZuh1oNadF79dTczT8AQnFNnFkH0ADNSeRe2dwOcFsicoOz6bSbIMkG4ntjQLBHv0XHg/640?wx_fmt=png)

**0x06 总结思考**
=============

  伴随着网络安全攻防对抗愈演愈烈，全流量分析显得尤为重要，在海量的大数据中提取关键信息，不仅是攻击者的思路更是分析人员的必修课。

  雁过留痕！Wireshark 从 TCP/IP 协议全过程捕获数据流，是一款非常不错的分析工具，这里只介绍了很少的功能，有兴趣的朋友可以继续深究！

**0x07 参考链接**
=============

https://www.jianshu.com/p/63f6f7d5deed

https://mp.weixin.qq.com/s/tKsOm-xxe7ZBqgkKccjjZg

https://adworld.xctf.org.cn/

**关注弥天安全实验室微信公众平台回复 “wireshark” 获取数据包**

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaqjXT4YxgHVARD1NNv0RvKtiaAvXhmruVqgavPY3stwrfvLKetGycKUfxIq3Xc6F6dhU7eb4oh2gg/640?wx_fmt=gif) 

知识分享完了

喜欢别忘了关注我们哦~

学海浩茫，

予以风动，

必降弥天之润！

   弥  天

安全实验室  

![](https://mmbiz.qpic.cn/mmbiz_jpg/MjmKb3ap0hDyTJAqicycpl7ZakwfehdOgvOqd7bOUjVTdwxpfudPLOJcLiaSZnMC7pDDdlIF4TWBWWYnD04wX7uA/640?wx_fmt=jpeg)