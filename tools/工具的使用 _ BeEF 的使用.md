> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247484515&idx=1&sn=42704db3c788bf347aad9160db953431&chksm=eaad845eddda0d48e4e8039f1fe46feabc56f081063cbdc190a9d61b8c3c765ffd169a5ae4a2&scene=21#wechat_redirect)

**目录**

  

BeEF 的简单介绍

BeEF-XSS 的使用

获取用户 Cookie 

网页重定向

社工弹窗

钓鱼网站 (结合 DNS 欺骗)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HzZrTT4UZAcukC3cKqetSHpGJABL8ZCM8yibLyNpvY2Zia3IAY3P6yE9A/640?wx_fmt=gif)

BeEF 的简单介绍

**BEEF (The Browser Exploitation Framework)**：一款浏览器攻击框架，用 Ruby 语言开发的，Kali 中默认安装的一个模块，用于实现对 XSS 漏洞的攻击和利用。

BeEF 主要是往网页中插入一段名为 hook.js 的 JS 脚本代码，如果浏览器访问了有 hook.js(钩子) 的页面，就会被 hook(勾住)，勾连的浏览器会执行初始代码返回一些信息，接着目标主机会每隔一段时间（默认为 1 秒）就会向 BeEF 服务器发送一个请求，询问是否有新的代码需要执行。BeEF 服务器本质上就像一个 Web 应用，被分为前端和后端。前端会轮询后端是否有新的数据需要更新，同时前端也可以向后端发送指示， BeEF 持有者可以通过浏览器来登录 BeEF 的后端，来控制前端 (用户的浏览器)。BeEF 一般和 XSS 漏洞结合使用。  
BeEF 的目录是： /usr/share/beef-xss/beef   

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPsvd2QIwyia7MJ5L4Na9TyaUZxPmichliaia8PMekujRoItib2H0T7icjfcFw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5Ae7TL0ribktKWDdib4Lv47xsog4G72hP4TiceEfibZ8KibSDBg2SRNvDzOa8abYBLdQQNRY3tkZMIrfg/640?wx_fmt=gif)

BeEF-XSS 的使用

在使用之前，先修改 /usr/share/beef-xss/config.yaml  配置文件，将 ip 修改成我们 kali 的 ip 地址。后续我们进行其他实验也是需要修改这个配置文件

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPgSPfibQcnMAu3CYxrK1kqMDyRvt0llqwH3HETdKxSlINJdZPTdmwdPw/640?wx_fmt=png)

**打开方式：**

· 直接点击桌面上的图标，过 5 秒左右，然后它自动会打开命令行和浏览器 beef 的登录框

· 任意目录，直接输入命令：**beef-xss** 打开 ，过 5 秒左右，然后它自动会打开命令行和浏览器 beef 的登录框

· 进入 / usr/share/beef-xss/，输入命令：**./beef-xss** 打开 ，然后手动打开浏览器链接

kali 已经把 beef-xss 做成服务了，我们也可以使用 systemctl 命令来启动或关闭 beef 服务

· systemctl start beef-xss.service         #开启 beef 服务

· systemctl stop beef-xss.service         #关闭 beef 服务

· systemctl restart beef-xss.service      #重启 beef 服务

我直接进入该目录，**./beef** 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jP0c31h5KkrzqLiagiagOicriavicpfBFErVfShr9oYwyOYJ3hiammVPL4Rkkg/640?wx_fmt=png)

手动打开浏览器，登录名和密码默认都是：beef

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPwI7IY7ZP2fYNxEJSAogD237nYDYsoXJ3V5doHobC589qRRhU4NREWw/640?wx_fmt=png)

登录成功后，这里会显示在线的主机和不在线的主机。在线的就是现在该主机浏览器执行了我们的 JS 脚本代码，不在线的就是该主机曾经执行过我们的 JS 脚本代码，但是现在叉掉了该页面

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPa5emr6rkYaIKYKiae8clDUrGftMAHpJNbMQbQWaTEolaIHKELektqyQ/640?wx_fmt=png)

我们点击当前在线的主机，然后右边会有选择框，我们点击 Current Browser **，**然后下面就有一些功能项：Details、Logs、Commands、Rider、XssRays、Ipec、Network、WebRTC

· Details 是浏览器信息详情

· Logs 能记录你在浏览器上的操作，点击，输入操作都能记录

· Commands 是你能对该浏览器进行哪些操作

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPCcBHngAVDl7kjAVgCU6u00BRVyuklkwfvNrUlZ2ghVaE0kF5Iorm3A/640?wx_fmt=png)

我们点击 Command，这里有一些我们可以使用的功能分类，一共有 12 个大的功能，括号里面的是每个功能分类里面的个数。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPelDkwSAQE7wyoWs6LvT3rbDPjNDf5qBwFVpa5cqibH7fp4XYfwh9C2g/640?wx_fmt=png)

我们随便点开一个看看， 发现有四种颜色的功能。

· 绿色的代表该功能有效，并且执行不会被用户所发现

· 橙色的代表该功能有效，但是执行会被用户所发现

· 白色的代表该功能不确定是否有效

· 红色的代表该功能无效

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPiaU5vicC7mYwuuC7u4bGVnia0GGFet8U14LxgInmooEMfykoLXvyTtzWg/640?wx_fmt=png)

### **获取用户 Cookie** 

我们点击 Browser—>Hooked Domain —>Get Cookie，然后点击右下角的 Execute

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPYLzvWK6JjsY4ibCk2iasSqpaRZicJYMubVdl9kvTEYrDIXlaRm8UhnyZg/640?wx_fmt=png)

然后点击我们执行的那条命令，右边就可以看到浏览器的 Cookie 了。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPTQfv6pq2RGycgktofXDHIZ0k9Z5UwaEKYfIBw9FOPoAHGuMreULfzQ/640?wx_fmt=png)

### **网页重定向**

我们点击 Browser—>Hooked Domain —>Redirect Browser，然后点击右下角的 Execute，然后用户的浏览器的该页面就会跳转到百度的页面了。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPWjyfevA1Wo8mEWYSkRRwTT8BaerItA39qXXicBrUhlQLaI4xHCTKw0w/640?wx_fmt=png)

### **社工弹窗**

我们点击 Social Engiineering——>Pretty Theft ，然后右上角选择弹窗的类型，右下角点击 Execute

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPiad9FTStA6uHWRLOl7rCibSiaJX00e2yEslxGtQ6nO8Thwf7d9ibcbBcvw/640?wx_fmt=png)

然后浏览器那边就会弹出框，如果你在框内输入了用户名和密码的话

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPppxIaicxMfibsib2RPFLbSNLMLde9hyRXtt89ARoTJ6OX6V3bhGBicCTKg/640?wx_fmt=png)

如果用户输入了用户名和密码，点击了 Log in 的话，我们后台是可以收到密码的

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPh4gcgQGutEmLtLjPnH3sEfQkCGerOuJNbus5dvsr3LEiazIncMaZLQQ/640?wx_fmt=png)

### **钓鱼网站 (结合 DNS 欺骗)**

进入 / usr/share/beef-xss/ 目录下，执行命令：**./beef**　启动 beef ，API Token 值

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPYOsGdKrfricdL01xCMmyUWr1kgXQINiaKmNmD8lMDJ7aQbuhj6qsKuibA/640?wx_fmt=png)

新打开一个页面，执行下面的命令

```
curl -H "Content-Type: application/json; charset=UTF-8" -d '{"url":"https://www.baidu.com/","mount":"/"}' -X POST http://192.168.10.25:3000/api/seng/clone_page?token=cc69c97f075be5d72675c04bbde77f747d36c6a9   #这里的地址token是我们上一步获取到的token
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPMHZB26wBLOcfBhzoM2Mms0vvLG67qK84NhRkAmlaFEvH81A8jdfnOA/640?wx_fmt=png)

我们克隆的网站在目录：/usr/share/beef-xss/extensions/social_engineering/web_cloner/cloned_pages 下

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPUicFrRWQOkZ0hSTut5MCWeLO5gtiaTCbibz74UU8mZWI5CBTx4HIRtY7Q/640?wx_fmt=png)

 我们访问：http://192.168.10.25:3000/ ， 可以看到和百度一模一样。只要别人访问了该链接，这个浏览器就可以被我们控制了！

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPeDfoXozHlKs48yGICcibPezSN0fXb2MO4SO9NrePvzYwWpU0rLticGrQ/640?wx_fmt=png) 

那么，如何让其他人访问我们的这个链接呢？我们可以结合 DNS 欺骗，将百度的地址解析到我们的这个链接上，这样，别人访问百度的时候就自动跳转到我们的这个页面了？

进行 DNS 欺骗之前，先去配置文件中把 3000 端口改成 80 端口

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPeqONicGPib5A9dAzcnMUyibYgKBYjNQDcjSkzPtcvNRfb7zHH0MnibYDTg/640?wx_fmt=png)

然后利用 bettercap 进行 DNS 欺骗。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPVODndgNAzytyhjibTGB6DIzLdv7IySpm3yOWicdlRaTaKmWGX2Aw9o6Q/640?wx_fmt=png)

然后重新打开 beef，然后克隆 www.baidu.com 网站

只要被欺骗的主机访问 www.baidu.com，其实跳转到了我们克隆的网站。这里百度的图片没加载出来，有点尴尬。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jP1QgheIoWsib6uW2Licc2Aa0V0kSrwUQF4K4uJnbv5b2bics7Ud0yJyxZA/640?wx_fmt=png)

更多的关于 BeEF 的使用，参考 Freebuf 大佬的文章，写的很详细，很好！传送门——>https://www.freebuf.com/sectool/178512.html 

相关文章：Bettercap2.X 版本的使用

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cZ3DhLYkUiaTPmyVTTmwdpaFvpBwKxOT3k4NJvb7HZDdHyeZkn632lxP4qs0dvu2Qdo3G6nVIo5jA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2ckkbwTsBvnDJpb89o8WMxvAKOaVnz60hOe7y3wAHiclddyK53lpEKIQlx4DKOq6EojHibVicgibDB2aQ/640)

来源：谢公子的博客

责编：梁粉

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640)

由于文章篇幅较长，请大家耐心。如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)