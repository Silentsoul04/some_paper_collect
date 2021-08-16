\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/b0Le8u7RmyTx3EPUS0Fcdw)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)  

**Ubuntu 最新爆出的漏洞，**GitHub 安全研究员 Kevin Backhouse 发现的一个 Ubuntu 系统大漏洞。

  
**测试环境：Ubuntu 20.04**

当前用户为普通用户权限，没有 sudo 权限

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzdPD6hrvJbr6LRITReNoFo5b3QibDUPnw1jMMlHhvQVGulplicic6l294g/640?wx_fmt=jpeg)

1\. 在用户主目录创建一个软连接

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzms2ZyVU5bCJerawDGTSAVIHyMJmqTO4dBRp1ADrp0ZENicIeoA6II2Q/640?wx_fmt=jpeg)

2. 更改语言设置，测试最好默认为英语（美国）换成英语（英国），汉语转换没成功过，可以试一试

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzKhxVST3VImSwnW4iaQbctEhZDuZIv1YDhjhSvut2ZblV9Ul105Op9Sg/640?wx_fmt=jpeg)

点选择后会有明显的卡顿，对话框也关不掉

看一下进程

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzXrbibvnN99y5G3RXlX9KQUTPxxIuWZtor2ePTibcw8YDa9s8icEdicEcOQ/640?wx_fmt=jpeg)

会有一个 accounts-daemon 进程占用了接近百分百 cpu，记住它的进程号，我的是 631。

3\. 杀掉这个 SIGSTOP 信号进程

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzPqO5Zp0wDkyfzLYlGEkZ4Ly2kHqOEe5Znu4HRiayoby6FRC0PWoWYZg/640?wx_fmt=jpeg)

4\. 删去软连接

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzpG5THpqGyGCrBAMuo0bG5BdFpLlskJjXhNdMfiaK4EhSjEse3sRaBug/640?wx_fmt=jpeg)

5\. 设置一个计时器，在注销后重置 accounts-daemon，延迟 30 秒给该进程发送 sigsegv 和 sigcont 信号，注销账户，让 SIGSEGV 起效。

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzEPE5006cPUrdosfwiaWiadia4Uf7aNBLQw1hwVUxEsbDFy3c6pMHALSIw/640?wx_fmt=jpeg)

6\. 注销账户后，等待几秒，会进入初始界面，漏洞利用成功，一直前进

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzF6rPRVDMOctNxTwrGyyfLictLXj3MfEooHoPVPNECK3xzDcGq42gdJw/640?wx_fmt=jpeg)

创建一个新用户

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzGlroseG8YmOWoRW2fD0b9NiakicuDniat6UGrmCBEtnviaZl93mTbqUxkw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzqAFxwjbDNISSoqYTRiciavOP8eyUb4s3xyfD2dxpP6Yjn3KNYxSMYicng/640?wx_fmt=jpeg)

创建完新用户查看权限

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLic6fiaUMpcSOqGldTPsP3hyzzpSlcNtM65ic2sdiaHkoqFRnprqVE1qaOq0XQOojRME8TKZETia4vqXpg/640?wx_fmt=jpeg)

此时这个新用户具有管理员权限

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

推荐文章 ++++

![](https://mmbiz.qpic.cn/mmbiz_jpg/US10Gcd0tQFGib3mCxJr4oMx1yp1ExzTETemWvK6Zkd7tVl23CVBppz63sRECqYNkQsonScb65VaG9yU2YJibxNA/640?wx_fmt=jpeg)

\* [靶场：uWSGI 路径遍历漏洞复现（CVE-2018-7490）](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650480718&idx=4&sn=de4a9fa6c645d2081d32006df6f577fe&chksm=83ba45aab4cdccbc19ee1287e48431dce38d1daf018b32452a7df11ac516143449e00b47950c&scene=21#wechat_redirect)  

\*[Discuz! ML RCE 漏洞 getshell 复现](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650472220&idx=3&sn=a27fd7c7c048dd3293f3a62a0a270368&chksm=83bb9af8b4cc13ee9607858c56877daeea6d6f0644b474d4f7daae874dca23344102e219b60b&scene=21#wechat_redirect)

\*[CVE-2019-17671:wrodpress 未授权访问漏洞 - 复现](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650474560&idx=4&sn=cafddc2022bcd62c6ad3f374bafa6c2c&chksm=83ba6da4b4cde4b26405933c7fe6c974982a85ea1d08a67b8a1b001165bac607aa0939a476ea&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)