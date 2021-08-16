> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BcxCXRgIcCzQ6XLZHDq-BQ)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fuBhZCW25hNtiawibXa6jdibJO1LiaaYSDECImNTbFbhRx4BTAibjAv1wDBA/640?wx_fmt=png)

扫码领资料

获黑客教程

免费 & 进群

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFJNibV2baHRo8G34MZhFD1sjTz4LHLiaKG9208VTU6pdTIEpC9jlW6UVfhIb9rHorCvvMsdiaya4T6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png)

一. Httrack：侦察工具
===============

*   对目标网站进行克隆复制，存放在本地，减少与目标系统交互，可用作钓鱼网站。
    

### **1. 安装 Httrack**

`apt-get install httrack`

### 2. 基本使用

*   在 Kali2021 中，默认是以 kali 用户进行登录的，而不是以往的 root 用户，这里需要注意。
    

##### **2.1）在 kali 目录下创建一个目录**

`mkdir /home/kali/wtcms/`

注意：如果需要创建多个目录，加参数 -r 即可。

##### **2.2）启动**

`httrack`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjcLOVUOozjOwcTITEDMLSMzib0Xl1siaicC00vUhM8VtrPgA7UxDaN0d1w/640?wx_fmt=png)

##### **2.3）输入项目的名字，这里输入 wtcms**

`wtmcs`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjtrObybDNS5y1RXTyeBEn5FGr1oxBATxSrzWSRibic1k3FaT1GRDw2Uiag/640?wx_fmt=png)

##### **2.4）输入 wtcms 项目存储的路径：**

`/home/kali/wtcms/`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjk3fRa9r9wWrNRtbwIKLMqiay0aFKzk6ibS5pDub0Zw8L50ib3LpShtgIA/640?wx_fmt=png)

##### **2.5）输入需要克隆的 URL**

`http://192.168.126.128/`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjIQibYTxRSft62UDk8dgnLqvlj6WhIDKSTs8pKW69DdUpBz2xjfGsMNQ/640?wx_fmt=png)

**解释**

1.  直接镜像站点
    
2.  用向导完成镜像
    
3.  只 get 某种特定的文件
    
4.  镜像在这个 url 下所有的链接
    
5.  测试在这个 url 下的链接
    
6.  退出
    

##### **2.6）此处选择 2**

`2`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjd0Xve7WBIWgzhycAyvwhkbib55Y7hib21nyibKKOXgwFkAJckMJcdpuRQ/640?wx_fmt=png)

##### **2.7）此处询问是否使用代理，**若开启了代理软件，则可以输入 127.0.0.1, 端口为 8888 （8888 是代理软件启动的端口），在这里直接 Enter 回车，不适用代理。

`enter`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbj1ESYheAR3Bib9Oxs1SGv3pqRKCdmXOB0ZvEfbHuiacRw4ApyniajSwbXQ/640?wx_fmt=png)

##### **2.8）指定需要克隆的范围，这里输入，是通配符的一种，表示所有**

`*`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjHa9uyaYqTInEOUe41qiagDxxbicuUR6go3dkSbAlqUTy74eibxEGkvq1g/640?wx_fmt=png)

##### **2.9）是否需要进行额外的选项，比如 - r，可以指定爬取深度**

`-r 4`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjKPGlpWZicnVricoSiaA3iaYdlyTHppAWZFfdR0P5gfrbdWBm6k4ZrcZiauQ/640?wx_fmt=png)

##### **2.10）是否准备启动**

`Y`

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjsKicsjgSnJ8Snjhrwb4vrdlUkhFfiauJ7103ZUTVNsGn6BgsRiajRDC1A/640?wx_fmt=png)

**2.11）完成，可看到克隆的资源**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEarCPgSd5meGe4fejblEbjSEAlykvIcNsCNYfMVBzHibmPugia5qF2ibHoQJfCktHsZbYuFCo6mkVsQ/640?wx_fmt=png)

### 3. 注意

1. 传统的在网站主机上浏览网页，你浏览的和摸索的时间越多，活动可能被网站跟踪，哪怕是随意的浏览网站，也会被记录踪迹。

2. 在没有进行授权的时候千万不要使用该软件进行镜像网站上的网页，像部署了安全狗或者其他防火墙的专业软件可能会记录这种行为，为攻击性质。

@

**欢迎加我微信：zkaq99、**实时分享安全动态

* * *

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSE8r6UDibLl3oFOu6cEZPryVrS6n7TfhmDVMfKfIfc7nicyXQ0r0CjPZxPIACeen4QF4fuLwsRBhzMw/640?wx_fmt=jpeg)

![图片](https://mmbiz.qpic.cn/mmbiz_png/sSriaaVicBMGmevib5nPyT8m3VxloX9cCQVGymXpYDibVwwMOmSaPtE9ib02ic3rRqoJH3Tics60h67X4ASIkXDowHV5A/640?wx_fmt=png)

往期回顾

[

黑客派出 58 岁亲妈帮他入侵监狱的安全系统，居然成功了???

2021-02-26

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFM95hxJHsvHXnAlZHet4qQv4BMQd95waIMVELrq2yyib0AAibsYYFNATqcv75vMmCicHeZIEJiaE8Mvw/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247510187&idx=1&sn=507e9f57a5b0064d352cb96b13138be6&chksm=ebeae186dc9d68905c038597dd0a89167a0d93eda1c88969745c8a12abd2d76e17d62eda72d2&scene=21#wechat_redirect)

[

三步拿 shell | 绕 waf 拿下赌博网站

2021-02-22

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSGh0vbLxS3s6cs5TAroBgmYyMnL9EI5DNV8lVUQgHleU298vjr8mVV8G34hJqsBqn5zGhN9pcXlwg/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247509625&idx=1&sn=772f295b20644d2aef4d7a0f2ec42b3c&chksm=ebeaef54dc9d6642d4e165807b5355a3058b51e8f18d13c8a376ddadca5b8b89ec316cdfb484&scene=21#wechat_redirect)

[

手把手教你编写远控工具及检测思路

2021-02-03

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEEHPrLl9fpG8PhTPKJibyBcByQtUrYWbqdZaXbltkD6boEvInuyy2LKlvMmAMCK7y3YLHu129WcdA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247507761&idx=1&sn=7f0f7c316be024e7db5461ebbe0dc64d&chksm=ebea961cdc9d1f0aff7341d07ab9aa7c9b248eb54df34649153eef08026d0d056ff3fc0c304a&scene=21#wechat_redirect)

[

记一次有趣的裸聊渗透测试

2021-03-01

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFUGlD0DibmykbJ8yTRy2wrFPek4DUaH1XxtZlDickQNttia6l1rwbMe2HFoxGeIWVrK80pia87CicjfGA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247510322&idx=1&sn=47599befa4014a292c1da5ac81aaf11e&chksm=ebeae01fdc9d6909513ce5c5cdcea1a5f31c3d966ea21be4bb2cfc8f1746d4b4838a9a5cb128&scene=21#wechat_redirect)

[

Fofa+Xray 联合实现批量挖洞

2021-02-01

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFvCmthyMgtPZH5h5GzzhoGoMY2eVhVq0iaAJjac8v8aiad2OdL5kEhrrlfxHuDXObicNQWArFbJMkgA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247507465&idx=1&sn=33bbfae575988803225594e26f5391e8&chksm=ebea9724dc9d1e3263f7403fbb34e14424311d0e9145b7d48d0815fef5ce54f6c81a164a0cce&scene=21#wechat_redirect)

[

实战纪实 | 一次护网中的漏洞渗透过程

2021-01-22

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEHCS6SZNzRp5K0yiclsOT1KkibxsVpiaXE0nPAq1fRvL9gKHYmqFNFH67yTogicvUvnMOjk1hMaBrEyA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247506677&idx=1&sn=0ef01f90372ff0fc62d5c4835950a73c&chksm=ebea93d8dc9d1ace1eb67e44d4016c12303e0938b4d6e5e66b0442ae40be70173511a18305a6&scene=21#wechat_redirect)