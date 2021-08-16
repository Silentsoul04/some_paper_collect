> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HR1YUy-9yeCIEalDJV92tQ)

前言
==

最近的 ctf 内卷起来了，好多 ctf 好哥哥们转头冲进了 src，可是并不熟悉渗透的基本流程啊，于是就有了这篇文章的由来。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5Ohiab7ypy9dG59sic8dIc0y8ghxj6E8D9tc7eQfszTqM52YbLJVtryqA/640?wx_fmt=png)

信息收集
====

**信息收集思路**

1. 确定站点

这里我是对点渗透的，直接百度搜索主站域名     

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen592piaOiasRVjHQ9hooenMJo3J1sCGiaZmfgUlTNEbCxsDBQCF7oAKL5iaQ/640?wx_fmt=png)

这里，拿到主站域名以后，扫一下子域，因为有的子域并不在主站的 ip 下，从其他 c 段打进去的可能性大大增加。有很多扫子域的工具，Layer 啊提莫啊一类的，但是我一般在指定目标的时候才会上大字典，刷 rank 还是得速度快啊。

https://phpinfo.me/domain/

推荐这个在线子域扫描，不知道站长是谁，没办法贴出来，站长看到可以私聊呀。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5fy7ASZsu5AaSImxiaL6sCP2l1tjHlEmy3gCvY8dc5Mo0LicMickibfibFsQ/640?wx_fmt=png)

这个在线网站的好处就是把查询到的子域和 ip 对应起来，方便的一批。

2.c 段收集

这里在推荐一个 fofa 采集工具。由 Uknow 师傅写的，能够快速的批量收集信息

当然，fofa 高级会员会吃香很多，普通会员 api 只能 100 条。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5gbQ4cFf3eO9CicYHNicgY0WSw875OI2z5qmJWvTbX08fz5yDwu6vHCMA/640?wx_fmt=png)

没有 fofa 会员的师傅们可以用别工具的收集 c 段，这里推荐小米范。扫描 c 段快的一批，还很舒服。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen58UgGLB7UYGzGib2W9fBHibqyOq0gj5wvqdjh019NNYsntoy1Km7AUI8w/640?wx_fmt=png)

如果想扫描的同时加上 poc 验证这里推荐 goby（缺点扫描速度太慢，但是漏洞验证贼强）  

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5tdxZ8VUwckpp8unZ9GuE6VVvic46K31rGrW032bV3qZsQ0FxmvcVicjw/640?wx_fmt=png)

找薄弱点
====

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5OEvmEibkpH27BaK1ict6lbic1rgYXssARa7aJ1bpMSWwnYq7YamFExWlA/640?wx_fmt=png)

一般找这种 title 是某某系统的，基本都有同类型站点。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5YSjJokC8ibM7nrbNuVZQbC2to2aNqMDKrSzCHgvAMcyJicticDkgwUCUw/640?wx_fmt=png)

这个站点的路径可真够深的，我的思路是往 xxx.aspx 的上一个目录 fuzz, 比如 https://xxxx.edu.cn/123/456/789/login.aspx, 就在 / 456/ 后边 fuzz 他的后台，这里也是成功的找到了后台地址

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5KcQclBUNGlbP7aWsVpl61vjJoM7jicQ2YQfYsATefAdM1APKBpRianfA/640?wx_fmt=png)

单引号报错，双引号正常，万能密码还进去了。直接上 sqlmap，os-shell 成功

Fofa 查询同类型站点，也算一个小通杀。相关漏洞厂家已修复

Ps: 一个 ip 不同端口可以分开交 edu。我恰了五个站，剩下的打包提交了并没有多给 rank

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5h0T1HtYnwn4q1VRicnOnlngbz3WBXhRhTkjR824x5udZKep9EdIYruw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5k6icdDdUegcJKia3xcnwZrhxzFiaMyp7Ku2ibtftZBPuZt3h42j6TBbdPg/640?wx_fmt=png)

Fofa 的好处是搜索 ip=”xxxx/24” 的同时还会吧子域名列出来

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5ZmUL6CyaRX7wqR9iaGica0Ia6siaEPfZquZyRz778fB8kvczzXbscoeKg/640?wx_fmt=png)

在一个子域名下，管理员把管理接口写到了底部

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5rhJLJXMmsgBic62CAzH5VV1hCKWD4rEXJvwNocAb1Sn2z5gF9N9SDWA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen55eVcy7EmuhxMvnBRiagQq3iceBahPJhy74eWpP8ZuJ65iax2PSa3WtwoA/640?wx_fmt=png)

讯易的 cms, 找了找漏洞网站并没有能复现成功的

抓包验证码不刷新，前端验证，上大字典冲他。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5PuvRZBIQibMXoibK6uEPl8FmZ58AcUb6Qbjh6V6FvVmJS2BCPEbnyWxg/640?wx_fmt=png)

好简陋的后台。拿到后台后开始基本测试流程，sql 注入，文件上传, 越权。因为这个系统后台功能特别少。只找到了一个上传点，就不做过多概述了

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5FbQh4xKAl9yNSUicSWuGic8aFo93oicTKIAEVBP2x4tI4x8WGlQl8qO9A/640?wx_fmt=png)

白名单的绕过异常艰难，耗时俩小时。常规手段用尽了。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5J3tgOZVCSYFdYsvTt3b6ovx9QiaXbl7ouhEV8zDktyBQoKKP47Zia9hg/640?wx_fmt=png)

经过很长一段时间的百度，找好哥哥，终于有了思路。此上传点目录可控，加上 %00 转码截断，终于不负众望上传了上去。（我太菜了，听说这是基本的绕过思路）

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5Beokx6ekkvHUTaGo13hoSqbrVoKichzjr7sOibapV1nteicLMjNMKTmZA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5F6oXUPjwibkMvOMr6uR2uJKkJW44mjIR1jJ6lJwOYZhIHGpvM9y4Yyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen55NsK3C1eSxp39dzw0hflkpSib1GgTg7icYnB6Pyp63qf63q7gEMKUTSQ/640?wx_fmt=png)

不知道算不算讯易后台 day 呢 [狗头] [狗头] [狗头]

此学校子域大部分是讯易得后台同样方法拿下了两个 shell 可惜没有未授权，不能像拿下大屏幕那样直接未授权拿下所有子域 0.0。

接着往下走![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5fR74FeCTdClSmib9tPv02Kb3gRMef2Tvrmt6icdTeKhpxeocAPBXQWag/640?wx_fmt=png)

思路是 fofa 搜索同类型站点，找用户手册，有没有默认弱口令，或越权或后台杀疯

弱口令找到了，可是依然是后台没东西，仅有一个 xss 索性直接问提交了弱口部分。写到此，笔者有点累了，正好女朋友叫去吃早饭（早上 10.27 分），就到这吧。

一个学校多多少少的恰了俩高危，一堆中危，几个低危，祝师傅们天天有 rank, 天天大牛子。

总结：基本的 edu 流程，没有啥新奇的。适合刚入 edu 的师傅看一看。啥时候我才能拿下学校大屏幕呢。

文章涉及工具请在公众号回复：工具   获取

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGY3vTlNVyNRFn6DKCZyTen5lyyNaXs23XMVYbbbriaNC4qlBEcciaz1KicKoqB3qZf730cmWJiaWP3iaOw/640?wx_fmt=png)

**如果对你有帮助的话  
那就长按二维码，关注我们吧！**  

![](https://mmbiz.qpic.cn/mmbiz_png/Qx4WrVJtMVKBxb9neP6JKNK0OicjoME4RvV4HnTL7ky0RhCNB0jrJ66pBDHlSpSBIeBOqCrOTaWZ2GNWv466WNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/EWF7rQrfibGYIzeAryXG89shFicuMUhR5eYdoSEffib7WmrGvGmSPpdvYfpGIA7YGKFMoF1IrXutHXuD8tBBbAYJg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/wKOZZiacmHTc9LIKRXddrzz6MosLdiaH4EQNQgzsrSXHObdAia8yeIlLz6MbK9FxNDr44G7FNb2DBufqkjpwiczAibA/640?wx_fmt=png)

**![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif)**  [经验分享 | 渗透笔记之 Bypass WAF](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247486210&idx=1&sn=5c0f6409e51c3c0cfb6bde43f2406409&chksm=c07fb0f6f70839e0e29f4ea9c8655d4ce7690c2a147aeeb74f2827aece58e3746f3f7c4ee562&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif)  [什么是 HTTP 和 HTTPS](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247486492&idx=3&sn=0a975b99a0351a95eef41d37813f7e5d&chksm=c07fb7e8f7083efe8054f864b5b25541fa3bf19ab311700f29254d03e45a4357069ee07c8802&scene=21#wechat_redirect)  

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif)  [实战 |  BYPASS 安全狗 - 我也很 “异或”](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247486492&idx=1&sn=fbd4ca8ed69ba6cb3adbc6ac8561d825&chksm=c07fb7e8f7083efef437eb3d685cc5bd6ac489629c613b5f2ce9ced8a7f8fcd335b6f91821a8&scene=21#wechat_redirect)

右下角求赞求好看，喵~