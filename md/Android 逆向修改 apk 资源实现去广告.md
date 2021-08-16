> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4PEH8tKbrivKSVTVJ9heOg)

本篇是《Android 逆向入门教程》的第二章第 6 节，更多章节详细内容及实验材料可通过加入底部免费的【Android 逆向成长计划】星球获得！

**0x00 前言**  

在我们使用 app 的时候，经常会遇到开屏广告和弹窗广告，顾名思义，开屏广告是打开 app 就会出现的广告页面，弹窗广告就是打开 app 后弹出的广告弹窗。要是一不小心点到广告，还会去跳转到相关页面下载，令人极其讨厌，本篇教程就针对部分 app 广告问题来进行破解去除。  

**0x01 实验一：开屏广告去除**  

开屏广告选取的 apk 是火柴人，初次安装上 apk, 打开之后截图如下

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3gl8mR7d1WyFibYia9wnaOhp8pV7kSyrVWyc0jhKkA3Woa1BicMVW49prQ/640)

  

不难发现，初次打开 apk 的界面并不是我们游戏界面，而是广告界面。对于这种开屏广告的分析，首先我们可以将 apk 拖入 AndroidKiller 中，查看 AndroidManifest.xml 配置文件，

在这里我们需要学习一个知识点，apk 的启动界面是在 AndroidManifest.xml 配置声明的，他的配置声明位置如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3Sv2OicD1TepGPBa6GiaymibdnJzGQEMa5v8In64LWNKZqUrtEZjibtDdGA/640)

  

我们只需要修改 apk 主界面的 activity 为我们的初次打开界面即可。在模拟器上打开我们的 app, 进入主界面，使用命令 adb shell dumpsys activity | findstr "mFocusedActivity"，查看当前界面的组件名为 org.cocos2dx.lua.AppActivity，将其所在的界面设置为初始界面即可完成去除开屏广告。

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL353FY7lYtNZ1OynAHibnCIG4R2dNjhBMQQxSW6D4I0UZfOFbicryPjLcg/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3ibDR42CSwCLZO3Wzr05lHxPb3cHiaLGO7QUOfgIVyQ0iaWBfhpDpMWibYg/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3yj9IibLpDeqxCfRTmLtgDstKotrT70eJb869kIqRK5HYazcblGHyib3w/640)

  

然后保存，回编译，安装，打开 app 发现开屏广告成功去除。

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL353hgia5aibADZK3aTicGosfoTJKzY9awm6pN3h1R7fOA20S7QoW9RThUg/640)

  

**0x02 实验二：弹窗广告去除**  

弹窗广告选取的 apk 是 laserdraw，初次安装上 apk 之后，在我们进行绘画的时候总是会出现一个弹窗广告，该 apk 的最下面也会有广告显示，而且每次打开一个界面都会弹出一个广告，效果图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3H5a1dApzeagScXuUTxfL4VtNgznQicycGsCehq25Ivue9BS8rRX0z5g/640)

  

本次实验就是要去除掉该 apk 的弹窗广告。将该 apk 拖入 AndroidKiller 中反编译，打开 AndroidManifest.xml，找到修改 user-permission 标签，删除掉关于网络权限配置声明。  

主要是删除掉这以下几个：  

```
android.permission.INTERNET，访问网络连接，可能产生GPRS流
android.permission.CHANGE_WIFI_STATE  Wifi 改变状态
android.permission.ACCESS_WIFI_STATE WiFi 状态
android.permission.ACCESS_NETWORK_STATE 网络状态
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3FFouPyibv71FvFOzLK34icHpUt9xZhEky4NC0bw9FWxiasvFkUKtEp89g/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3QIwuvmPCuCkehUcHZLoo0SafjcGcsdwAKuC4pm2YujtuicTKgIWNkWw/640)

  

然后保存，回编译，安装，发现弹窗广告成功去除。

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWian6UnictNgCmvnjuCSPiaS0u1YPCFicbku7m7WhOmDV4ONPzu5m0RMXAJk5jK9sJZyYRhFAEHUNW3HQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3PhO5LB8RkJGnC8mUjRbmVhLnicywXCcysLWRZR0zcSKZr4ibJ2ItRkCQ/640)

  

**0x03 知识点小结**

*   修改入口广告
    

```
activity标签中带有：
<actionandroid:/>
<categoryandroid:/>
main和launcher属性结尾的是当前的入口界面
然后通过命令获取到主页activity,修改其为入口界面即可
命令为 adb shell dumpsys activity | findstr"mFocusedActivity"
```

*   弹窗广告修改
    

```
删除user-permission标签中有change_network_statechange_wifi_state
access_network_state access_wifi_state
注意：android.permission.internet不要删除
```

  

  

团队公开知识库链接：  

https://www.yuque.com/whitecatanquantuandui/xkx7k2

知识星球：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL3yZrl9a0iawR9iaCCDKWlSHaaF0R38ibN0a1aES6oajjQsfJJJAy9Kloyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu4icQTSibbYk5v7jyDLWapbL38ribbkhygJGFKOw0G0gKiamFNAsh9EGvUTiaWR80uiclJMskMN0bQuKrtA/640?wx_fmt=png)

**往期经典**

  

[Android 逆向入门成长计划【免费知识星球 + 微信交流群】](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247485929&idx=1&sn=1b74ef1fc771be6ff32fb56b6ab8f560&chksm=9b3701ddac4088cb2fd25fd7bc3356f7a9710460a804360a955f036b7398e6a3c47c18377541&scene=21#wechat_redirect)  

[《从入门到秃头之 PWN 蛇皮走位》](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247484643&idx=1&sn=a8effd61504fc574ee7089f9ce8440af&chksm=9b370cd7ac4085c102e5f3ee1bd1fcb6f61fc56559edef3e4eb33d934c499c541d56c87ae7cd&scene=21#wechat_redirect)  

[漏洞挖掘｜条件竞争在漏洞挖掘中的妙用](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247484441&idx=1&sn=2f24cfe9e648118a4e537c0446e98119&chksm=9b370c2dac40853bf285a7e3fcbb8d83cba2aa34e82d81346af29d38c2855269dbc094aaab1e&scene=21#wechat_redirect)  

[漏洞笔记 | 记一次与 XXE 漏洞的爱恨纠缠](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483808&idx=1&sn=e49283ccb0de1a3b7ac89e9cdb6d0e2f&chksm=9b370994ac4080829721246426d4dee351a4b7bdc2ac737f1f5fe1eb1a69eeb8a1dcaeb50b13&scene=21#wechat_redirect)  

[移动安全 - APP 渗透进阶之 AppCan 本地文件解密](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247484404&idx=1&sn=184cb740fcdcdccc6f41f81c9abbc008&chksm=9b370bc0ac4082d6ef87004b1ce7e7c5e4d618a8e4696021d805e90a31c6a32b23c4139c5b50&scene=21#wechat_redirect)  

[内网渗透之从信息收集到横向独家姿势总结 - linux 篇](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247484666&idx=1&sn=5b0cf18e99d5e7cb9ff2b4a105251557&chksm=9b370cceac4085d8792838dde9359371751fad09200df6fd7ccf98a3a0230b71dbcd73cc595b&scene=21#wechat_redirect)  

[HVV 前奏｜最新版 AWVS&Nessus 破解及批量脚本分享](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247484137&idx=2&sn=286c87de932713c7e0951e9633a9fda9&chksm=9b370addac4083cb868bed7429c420f7feb101abd69550cd8cc4645f31e18b29d5d7f93cb3f7&scene=21#wechat_redirect)  

[Android 抓包总结 - HTTPS 单向认证 & 双向认证突破](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247484224&idx=1&sn=c3af9883d69de8e7c57d9f9503e63685&chksm=9b370b74ac408262fa670b3917250d509f77bd5de87dd4fef463358a6e18f1dcad517280c9bd&scene=21#wechat_redirect)

[图形验证码绕过新姿势之深度学习与 burp 结合](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247484598&idx=1&sn=997f5329edb5ad2850e8e129da32f175&chksm=9b370c82ac4085941fe53b84ec5fa61961cbb4ef406ae2013773d6144a28a665be9f7e80a283&scene=21#wechat_redirect)  

安

全

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/Ov836aiagXu6QjcUJ7PANdN0pQJFo6ZOMYtCDjiaB5dg0TXT5vL4ldibEiacDdz9EL4s83YH3k0UibCRDvw69eWoQdA/640?wx_fmt=jpeg)

**扫描二维码 ｜****关注我们**

       微信号 : WhITECat_007  ｜  名称：WhITECat 安全团队