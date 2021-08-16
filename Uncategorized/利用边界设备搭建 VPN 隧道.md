> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3kUup92p89CXjxn4ZwFGRw)

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RXu3bXekvbOVFvAicpfFJwIOcQOuakZ6jTmyNoeraLFgI4cibKrDRiaPAljUry4dy4e2zK8lUMyKfkGg/640?wx_fmt=png)

0x00 背景
-------

```
在最近几次演练中，在互联网获得了一些目标路由器、防火墙等网络设备的弱口令。
之前没有通过搭建VPN的方式得到过入口，所以对这方面知识进行了补充。

```

0x01 L2TP VPN
-------------

```
相对于IPsec，L2TP的应用场景更容易被利用，也更便于操作相对而言要节省更多的
时间。L2TP隧道是在LAC和LNS之间建立的，LNS为L2TP的网络服务器，在本次场景
中就是开放在互联网上的路由器、防火墙等。LAC由自己的PC担任，我们最终的目的
是为了让自己的PC进入内网。

```

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGib27M8kfoKol9HfE5ywcQGianibXAaYbicypfBGMyo4XQC55EbUW2ly2HA/640?wx_fmt=png)

L2TP 隧道示意图

0x02 H3C ER 系列路由器
-----------------

```
以H3C ER系列路由器为例子搭建L2TP隧道

```

（1）配置网络服务器（ER 路由器）地址池  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGPI9HBDz2RMiak7SyJiaJA4d1Py1LiarVmqssnJoc3PmfgFicosgIBckbsg/640?wx_fmt=png)（2）配置拨号认证账户密码  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGflLSA4BndNXCgick2mREOOV6BJD2QQQ4ulZeRB8LOnD3FWnebrpYVFg/640?wx_fmt=png)（3）配置 LAC 相关信息，VPN 登录认证的账号密码可与拨号认证账号密码一致，设置 L2TP 服务器地址, 也就是网络设备出口 IP 地址。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGibZuFJWUhcGGOWf7ibAHsbOBAMBxg2qfwUicLlqhlJSzmU0ZeSH3jUIWw/640?wx_fmt=png)（4）配置访问 LNS 内网的路由，本次目标内网地址为 192.168.98.0 段，选择出接口为 L2TP0。配置完后便可通过 PC 连接。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGbl4EpvHRKw942YYEh3zZk4dtgbsJCQvfVicOEVXaxUhPaIhX6ib57fAg/640?wx_fmt=png)(5) 配置 VPN 连接信息（mac）（windows 参照 vpn 连接方式）  
输入服务器 IP 地址（路由器上设置的）及账号。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGEjlLINXFiaVAkrXnc8tXyQjsjBksL8VIcmBm3vU7q0J5udAbib70BoKA/640?wx_fmt=png)（6）连接成功后，登录路由器查看发现 PC 已经处于连接状态，PC 端 IP 段和想要达到的内网网段已经可以 ping 通。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGXQHKq75YP93ibS0l5Z9ibzNxHxoO5XY2Q056IAoRQ6QY3YXuh9icgoJng/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGxTT9hq6h7lwc4rpzrG0aFpK6VNDahNaTpT0P0EpO57iazxcwyBAoaMA/640?wx_fmt=png)（7）输入目标网段内网地址也可访问到相关资源，至此 VPN 入口搭建完毕。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGmfUlUjSickTbLADONDa3mj4V5SfF6RTM9uFiatHiaIa7ZibshAS6j2NVdQ/640?wx_fmt=png)（8）部分问题，mac 提示 "IPSEC 共享密钥丢失"，可参考：

https://www.jianshu.com/p/3d1e1982ee05

0x03 锐捷 RG 系列
-------------

```
以锐捷 RG系列路由器为例子搭建L2TP隧道

```

（1）进入配置界面设置。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGiaI3ZYPg9ictSF2UW8CQp6wdaer3cJTotQJ8iawnv5RbduuhlhicRvwz1A/640?wx_fmt=png)（2）选择总部。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGlkibokEaIN1bOy5lquRND0xjLqhP8np6yxk4OX95nSSsdb7XFJ0jz3g/640?wx_fmt=png)（3）选择分支机构为移动用户。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGkic7iaCkCJ8m9TTwQ8zPqsHx6Fhyz9Pz2RvjZYekACRKN6I7lB7LC1nA/640?wx_fmt=png)（4）选择 L2TP，配置网络地址池。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGZyBWma53tibuEIr2asJrmgJiaebDCzXXmEGl4HgjWFWwtUIg0icyapWBg/640?wx_fmt=png)（5）配置拨号认证账户密码。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGw9oKs9WPGLOnCiaujeSpHHkiaVr2XdAmqxfeB14uyb05A6W0eRbwzbtQ/640?wx_fmt=png)（6）点击完成完成 L2TP 隧道搭建。  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGsiaPfFN4t4rz2KicqqfXlEqz34A4b2lUyEpQQajndYB8fWjp4aVGtEyA/640?wx_fmt=png)（7）配置 VPN 连接信息（mac）（windows 参照 vpn 连接方式）  
输入服务器 IP 地址及账号。  

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdGH7IjsoyhrHr5QMDSyhYprGh8rSJibC3ia5COxBKBLhk57gPoRjo4slAg/640?wx_fmt=png)

（8）成功连接 VPN，路由器已显示 PC 处于连接状态，PC 端已经可以连接到内网网段。  

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RX3GlmCPBBibhSpm0x0XjkdG1Zer1huicobYmfnO2A8w9kxWianjkTZ6KPaKqbST6OdStOK0XfAIUk9Q/640?wx_fmt=png)

0x04 总结
-------

```
在攻防演练中遇到网络设备弱口令并开放到互联网上的情况较少，一旦遇到构建好VPN
隧道便可以快捷的进入到目标内网环境中，并且可以绕过互联网大部分的安全设备防护，
是一条不错的攻击路径。

```

0x05 参考链接
---------

http://www.h3c.com/CN/D_201705/995158_30005_0.htm#_Ref460923080

https://blog.csdn.net/weixin_34366546/article/details/92300406

http://www.h3c.com/cn/d_201112/735209_30005_0.htm#_Toc310844882

E

N

D

**关**

**于**

**我**

**们**

Tide 安全团队正式成立于 2019 年 1 月，是新潮信息旗下以互联网攻防技术研究为目标的安全团队，团队致力于分享高质量原创文章、开源安全工具、交流安全技术，研究方向覆盖网络攻防、系统安全、Web 安全、移动终端、安全开发、物联网 / 工控安全 / AI 安全等多个领域。

团队作为 “省级等保关键技术实验室” 先后与哈工大、齐鲁银行、聊城大学、交通学院等多个高校名企建立联合技术实验室，近三年来在网络安全技术方面开展研发项目 60 余项，获得各类自主知识产权 30 余项，省市级科技项目立项 20 余项，研究成果应用于产品核心技术研究、国家重点科技项目攻关、专业安全服务等。对安全感兴趣的小伙伴可以加入或关注我们。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/rTicZ9Hibb6RX4MU7S4WB8R6vF3JbUjA7K0ZtOPxqGSo1HGPhTDicQibOro93UYNBOwRPd4EFseGTDsl1tan0ZXcmw/640?wx_fmt=gif)