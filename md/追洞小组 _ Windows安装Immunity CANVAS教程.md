> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DG7jR7XLM5E64grbuZDu8g)

****文章来源｜MS08067 WEB攻防知识星球****

> 本文作者：****Taoing****（Ms08067实验室追洞小组组长）

  

  

**漏洞复现分析  认准追洞小组  
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

**参考：**[https://mp.weixin.qq.com/s/7LyEou4ebdqIAI9UguawiQ](https://mp.weixin.qq.com/s?__biz=MzI1MDU5NjYwNg==&mid=2247488229&idx=1&sn=5eea68e5964480df767121d7cb3d90cf&scene=21#wechat_redirect)

  

2021年3月2号，Twitter上有安全研究员披露了ImmunityCANVAS 7.26工具源码泄露事件，安恒威胁情报中心猎影实验室第一时间对此事进行了追踪。

  

**0x01:准备**
-----------

**实验室公众号回复"CANVAS" 获取工具下载地址**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicfBPlyryhDFZM0uNtpLnMnkHm3Tao09yicbYqDbfAUBIwZZXWZ34ibIFReGd4xCn8tGFuiamTiaBFqFg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**0x02:安装：**
------------

1、进入Windows Dependency目录运行CANVAS_Dependency_Installer.exe等待就Ok了

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicfBPlyryhDFZM0uNtpLnMnNJswmS0FwyQibpib8hMF6jQRibIxOR9A5q7QmaqF5cbxDzCNwAzlXnFQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

安装完成下一步修改启动配置路径。

  

**0x03:配置启动路径**
---------------

进入ImmunityCanvas7.26文件下canvas.bat文件

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

我的修改：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**0x04:启动**
-----------

进入ImmunityCanvas7.26路径下，运行canvas.bat启动

如果遇到下边问题，重新运行CANVAS_Dependency_Installer.exe即可。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**0x05:完美运行**
-------------

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**0x06:功能测试**
-------------

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

根据官方说明，这个工具有以下几个功能特点：

**自动攻击：**

可以选择自动攻击功能，软件会自动对网络、操作系统、应用软件进行攻击。攻击成功后，会自动返回目标系统的控制权；

**漏洞攻击：**

该软件漏洞库每天进行更新。同时包含了大量的0day以及未公开的漏洞。可以指定漏洞对目标系统进行攻击；

**代码开放：**

平台和主要攻击包代码开放，可自由组合调整重新打包自有攻击检测漏洞包，也可以基于平台研发编辑自己的攻击包；

**集成：**

可集成数据库的弱点扫描探测， 渗透攻击工具， 保证数据库及其应用的安全，也可集成Web应用的弱点扫描探测，渗透攻击工具，保证Web应用的安全。

  
  

  

**【追洞计划】****顾名思义即追最新的漏洞，包括****2020&2021所有Apache漏洞+主流框架****，将会在星球内部招收感兴趣学员，纳入追洞小组。**

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**主讲导师介绍：**

**Taoing**：Ms08067安全实验室核心成员，现任安恒信息高级安全工程师，擅长web渗透测试，应急响应。

**TtssGkf：**Ms08067实验室核心成员，Defcon86021议题分享者，擅长领域：渗透测试，漏洞分析，代码审计，安全开发。

  

  

**扫描下方二维码加入星球学习**

**加入后会邀请你进入内部微信群，内部微信群永久有效！**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

**目前36000+人已关注加入我们  
  
**![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)