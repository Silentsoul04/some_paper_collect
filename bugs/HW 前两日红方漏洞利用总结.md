\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/kHi9SdFDSArT4z2PjIMjaQ)

转自：提笔杂货铺  

今年护网高手过招，各大厂商 0day 满天飞。各位蓝队的小伙伴也要加油啊！截至 9 月 12 日，目前已知的安全漏洞以及 poc 总结。  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo5zlj0PW3QfPuic1fGgRTuynn0FepXztiaynacKGPzeDXltUzwPXXr3eA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDomcKyVkkxGwYquDnxER6f7GUzRW3IgdGRjgucIeDJLkTjJgUWStmk2g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo6H18HykPRTECkWL3TAwdYxQ9ny1ibtLo9YHvEdLmQKMFy4QYt5jyvDw/640?wx_fmt=jpeg)

**1**

  

**深信服 EDR 3.2.21 任意代码执行漏洞分析**

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo6H18HykPRTECkWL3TAwdYxQ9ny1ibtLo9YHvEdLmQKMFy4QYt5jyvDw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoiaah2lVsHycWZs6Mht8BAhvDZ20fRLsOIFc5dPXUxF1vtBzHAvnUj1A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoZwic1DS2GOAgyHCrpp5OiblI6YZBItrLichCaeZrum7Qceff2vaUSrQkQ/640?wx_fmt=png)

官方已发布补丁，注意更新！  

**2**

  

  

**绿盟 UTS 综合威胁探针管理员任意登录**

漏洞详情：

  绿盟全流量威胁分析解决方案针对原始流量进行采集和监控，对流量信息进行深度还原、存储、查询和分析，可以及时掌握重要信息系统相关网络安全威胁风险，及时检测漏洞、病毒木马、网络攻击情况，及时发现网络安全事件线索，及时通报预警重大网络安全威胁，调查、防范和打击网络攻击等恶意行为，保障重要信息系统的网络安全。

绿盟综合威胁探针设备版本 V2.0R00F02SP02 及之前存在此漏洞。

漏洞利用过程：

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoChUTlmjPuv2ubjMmbZf0GNNibH0BKT1Ul3l7dlvb8Anv9h2mJoWFODQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDojnfMVVdQdr1RKibjQjiaWyXibGGBdzXegkT5CL8Q4mvC0ibZ1IEZRtdrCA/640?wx_fmt=png)  

对响应包进行修改，将 false 更改为 true 的时候可以泄露管理用户的 md5 值密码

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo3DScicZpenwib9dOAP3uLYuOq1BRvWlGVzLnLEATe4KuAoujkPVn6dvQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDosQBRnK8AtA6icKTKHGhI4SabM43HYXCeVwxHuqzQzwW9Gsa1DVgUWRQ/640?wx_fmt=png)

利用渠道的 md5 值去登录页面  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDohb430PRrHEouOtVUAWCSfcBjYjYzvxLSAsicpjibLZGIst4RelyHHskA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo6H18HykPRTECkWL3TAwdYxQ9ny1ibtLo9YHvEdLmQKMFy4QYt5jyvDw/640?wx_fmt=jpeg)

 成功登录，登录后通过管理员权限对设备进行管控，并且可以看到大量的攻击信息，泄露内部网络地址包括资产管理。  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoVDibpbEvQ3CCaWmWibDSYqBdzw5wHjCEtbol4RUupNfONds3FExTeumA/640?wx_fmt=png)

处置意见:

建议尽快更新补丁至最新：http://update.nsfocus.com/update/listBsaUtsDetail/v/F02  

**3  
**

  

**齐治堡垒机前台远程命令执行漏洞**

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoBDDgeO2v6pXCxicLnEhLHWSk4zJy7TicNicxwVXaYQ4iaCDdGNKR7X3X4Q/640?wx_fmt=png)

**漏洞利用：**  

利用条件：无需登录：

第一 http://10.20.10.11/listener/cluster\_manage.php 返回 “OK”。  
第二，执行以下链接即可 getshell，执行成功后，生成 PHP 一句话马 / var/www/shterm/resources/qrcode/lbj77.php 密码 10086，使用 BASE64 进行编码。这里假设 10.20.10.10 为堡垒机的 IP 地址。  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo3icsibWRe2CRAqIl2hxPwE3SMh4YwFd8oLmoP5cafaup9MZbR605yjPw/640?wx_fmt=png)

特征：  
漏洞利用点：  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDot9BqLNrKY6IhnL3uoE3R5TI19veO4q1EqwsFDS011OHeH12lGI2WHw/640?wx_fmt=png)

**4  
**

  

**泛微 OA 云桥任意文件读取漏洞**  

  

  利用 / wxjsapi/saveYZJFile 接口获取 filepath, 返回数据包内出现了程序的绝对路径, 攻击者可以通过返回内容识别程序运行路径从而下载数据库配置文件危害可见。  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDovaujxAYmS0WKzLs2sDL3z9IJNSOFMYDEyia8rrpzYccyRMicSu4XdhxQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo6pIGtsevIklEArGJSzsgqqVIq49fsfTFanjqkqOjRVmVsWLpWmXmEw/640?wx_fmt=png)

漏洞利用工具下载：链接：[https://pan.baidu.com/s/1j5kugnCc1pq5IjjlFMnZLw](https://pan.baidu.com/s/1j5kugnCc1pq5IjjlFMnZLw)#

提取码：52ss#

**5  
**

  

**Exchange Server 远程代码执行漏洞**

  

  CVE-2020-16875: Exchange Server 远程代码执行漏洞（202009 月度漏洞）（POC 未验证）

**ps 版 POC：**https://srcincite.io/pocs/cve-2020-16875.ps1.txt

**py 版 POC：**https://srcincite.io/pocs/cve-2020-16875.py.txt  

**6  
**

  

  

**宝塔中间件解析漏洞  
**

**1\. 环境搭建**

Windows Server2012 R2 X64

宝塔 Windows6.5.0 版本

宝塔选择：MySQL+ PHP-5.4+IIS 8.5  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoibkMLria8hOOvWEyLVdZCl60eWPBRd85xOmAZTg8uHwJPgm9zWtlibpVw/640?wx_fmt=png)

源码使用公开的 PHP 上传源码:

https://www.runoob.com/wp-content/uploads/2013/08/runoob-file-uplaod-demo.zip。  

已做白名单限制，仅允许上传 .gif、.jpeg、.jpg、.png 文件，文件大小必须小于 200 kB  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoONyy83XOzu2fzicmBSOrFcRstkng7NsK9fjeqNhM55ELpbBGgnRe52Q/640?wx_fmt=png)

**2\. 漏洞复现**

配置好网站以后：  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDokjFCfWMErqQ2w2r6uNPoYJ1MzvrrSSsuBrWfHY6HZTEicyucAYKo1cw/640?wx_fmt=png)

本地写一个

<?php

phpinfo();

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDosTy0Hd1jibvda4ad9nvqhEn0vI5R8nhkAiaT9on57WyWo0oeDuRUYVQQ/640?wx_fmt=png)

另存为. jpg 格式  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoBK9CZBPxGEVupBfWK3LgBDlSrXgsCicH05NH8gNcpEdZRNnXWJaZibMQ/640?wx_fmt=png)

直接上传文件，不需要做任何修改：  

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo3Urs3ib4U2kSLdATWUDNTQqqcauoPnS9r8GWV2eGicfOva9DN3FwxF9A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDo2zAOACur6nXayUlrmyyC19MrTVNoWqSLBt6B04t23JdgKOjcqoyawA/640?wx_fmt=png)

访问上传文件地址：

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoNNcTkT1ZvdFD1DcDQwibCEbUK6h73s0k6nZDjuoiciaQwZ4G9MRnsjNhA/640?wx_fmt=png)

在 upload/1.jpg 后面加 /.php

![](https://mmbiz.qpic.cn/mmbiz_png/x1FY7hp5L8EnykD9smAy0C5tXUCTaHDoicxkGh8p0Cb6JBu5fdraTtxIvELx70has4OnZxILOlUIib9JhXcFNCHw/640?wx_fmt=png)

成功验证存在任意文件上传漏洞！

文章参考：[HW 第二天: 0day 快递盒在线送 day；圈子社区孙团长；](https://mp.weixin.qq.com/s?__biz=MzU4NDU3NTYyMQ==&mid=2247484415&idx=1&sn=ae7faf734b925f37986533d723c0e1df&chksm=fd96fc15cae175039aa3479c0934d38104e2ed1309f339d5a6a5b5000af2bbaf548c5f9b4a15&mpshare=1&scene=21&srcid=091220HLXxYB7fLNZHBE6ns0&sharer_sharetime=1599921240016&sharer_shareid=feb70bfade95f7b919e1580553155df2&key=7adf10a6617c631553491d94e407876db6ddfc20dbf44e909e3858445e833178d54c0a3e35e6aa62c5dc82d4baa42d69d93dc836810a928309cd7597eb3bfe1cc67263d41855c09b04dca1a5160232caeb3a0ccc1d7407ee972f2372535d6d29aa708a4def7bc9a290d7e9f3b23545342fa1d7e4125ec342c4377777bd8e7351&ascene=1&uin=NzU0MTEyMzQ1&devicetype=Windows 10 x64&version=62090538&lang=zh_CN&exportkey=AexYZ71liMuk9dEQKtlilfI=&pass_ticket=UU2xrO6DCCSJXEqaLIjCgpmCPiXsqug7AOaDSmbHy6mnMyQtRszWr8qPVmPnpHYr&wx_header=0#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/x1FY7hp5L8Hr4hmCxbekk2xgNEJRr8vlbLKbZjjWdV4eMia5VpwsZHOfZmCGgia9oCO9zWYSzfTSIN95oRGMdgAw/640?wx_fmt=gif)

[2020HW 红方漏洞利用总结（一）](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247486324&idx=1&sn=396ce93d26763da4cdffb209a767bb51&chksm=9acedbebadb952fd25a47a0c4dd5dfc5cf35021259f289fa423bad3dc5f9d24e275e89a7da56&scene=21#wechat_redirect)  

[2020HW 红方漏洞利用总结（二）](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247488734&idx=1&sn=06e62a6f53526d928126744bcd57e410&chksm=9acec441adb94d57c60d385cd0552336ec1293fc4a190db203439af7b72e3ceb8ca3cd6d52a2&scene=21#wechat_redirect)  

[2020HW 小技巧总结](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247488734&idx=2&sn=fb768d96c43aa14cf4ceb8abb6af6abf&chksm=9acec441adb94d57574c4dcf0c84499ad78a1e40f696051777b31da05e54ddb63eb78a7a2755&scene=21#wechat_redirect)  

[2020 护网参考学习   关于护网行动的总结](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247486325&idx=1&sn=6cbdcdd453a7cfa40f2619fcd1a716e9&chksm=9acedbeaadb952fcb9eb9bc993cb2850c9067bd1ad88f73499bb9583f6b30db7d0bedb1a756e&scene=21#wechat_redirect)  

[2019 护网行动防守总结](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247486338&idx=1&sn=ba9f86af4678be5d9111054a3f7fd1d2&chksm=9acedb1dadb9520bdf4428ba5b2c7432c8599d5b849ac8d64174b3961898e7da7feaf42244f0&scene=21#wechat_redirect)  

[护网行动 - 攻击方的 “秘密武器”](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247486391&idx=1&sn=7c62921d770a7ed451e47569879986b7&chksm=9acedb28adb9523e727fbd8b2861bd866750fb5bc0ceffbf99d6ca98546110374ae963a10e14&scene=21#wechat_redirect)  

[护网行动 | 网络攻防实战演习之蓝队指南](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247486471&idx=1&sn=6bc69c3ac91108889f816d5371d02718&chksm=9acedc98adb9558e241b30d00d0c8bad376131a65b40e20b10904f11659dff5ed72455255d8e&scene=21#wechat_redirect)  

****扫描关注乌云安全****  

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavz34wLFhdnrWgsQZPkEyKged4nfofK5RI5s6ibiaho43F432YZT9cU9e79aOCgoNStjmiaL7p29S5wdg/640?wx_fmt=jpeg)