> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/v0rpjwLYHzHf1gVAIwoMgg)

目录  

  

  

  

WPScan 的使用  

    扫描指定的 WordPress 站点  

    主题扫描  

    扫描主题中存在的漏洞  

    简单扫描 WordPress 插件  

    完整扫描 WordPress 插件

    枚举 WordPress 用户名  

    暴力破解  

    介绍    

  

  

  

WPScan 是 Kali Linux 默认自带的一款漏洞扫描工具，它采用 Ruby 编写，能够扫描 WordPress 网站中的多种安全漏洞，其中包括 WordPress 本身的漏洞、插件漏洞和主题漏洞。最新版本 WPScan 的数据库中包含超过 18000 种插件漏洞和 2600 种主题漏洞，并且支持最新版本的 WordPress。值得注意的是，它不仅能够扫描类似 robots.txt 这样的敏感文件，而且还能够检测当前已启用的插件和其他功能。

WordPress 是全球流行的博客网站，全球有上百万人使用它来搭建博客。他使用 PHP 脚本和 Mysql 数据库来搭建网站。

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

### WPScan 的使用

  

由于 Kali 中自带了 WPScan，所以怎么安装就不讲了，直接说说怎么使用。

```
wpscan -h  #查看参数以及意义
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReRtaKuB2cgfsKzTNEMNicZ8rkmGxLvk77AN0cLB7Re8GppthJI8BS9EQ/640?wx_fmt=png)

首次打开，我们可以先**更新其漏洞库**

```
wpscan -update  #升级漏洞库
```

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

 **扫描指定的 WordPress 站点**  

  

  

```
wpscan -u http://192.168.10.44   #扫描WordPress站点，可以使用 -u 或者 --url 参数都可
```

它会扫描给定的 WordPress 站点的一些信息，并且列出可能是漏洞的地方。注意，这里 wpscan 判断是否有漏洞，是根据 wordpress 的版本判定的，只要你的版本低于存在漏洞的版本，那么，它就认为存在漏洞，所以，这个没有太多的参考性。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReA5q3HWiaUvibbl3meC8JWicu8cXze0Nd4LbnKjrEFBKQ4T0QIx6FnQeyA/640?wx_fmt=png)

> 注意：以下扫描的结果都是只看进度条之下的

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

**主题扫描**

  

  

```
wpscan -u http://192.168.10.44 --enumerate t  #主题扫描
```

一共扫描了数据库中的 411 个主题，发现了 3 个主题

**![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReusT57nrpKCOibh1CUdGTTYvyXvcEvXUnaZTTkk64105LmXGoUhS0tXg/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

  

**扫描主题中存在的漏洞**

  

```
wpscan -u http://192.168.10.44 --enumerate vt  #主题扫描
```

可以看到，这里没发现主题中存在漏洞的 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReWNszB0sook5Jiav062b3OoWandQu6uicjVt11G5Irc87fHqgPHqAjV9g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

**简单扫描 WordPress 插件**

  

```
wpscan -u http://192.168.10.44 --enumerate p  #插件扫描
```

一共扫描了数据库中的 1494 个插件，其中发现了 1 个插件，并且又漏洞 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRebW8ITXdYVoJk9VpibJ9UPeiaNNribiaMd3rEI8jfs44jEVGdeQaVuEFq8A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

**完整扫描 WordPress 插件**  

  

```
wpscan -u http://192.168.10.44 --enumerate ap  #扫描插件中的漏洞
```

可以看到，扫描乐乐 77803 个插件，发现了 3 个插件，其中一个有漏洞 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRec5frtKUHufY7nxM2AhJhndZxN5oeYKxXz4uMv5bO6xTdibsra59DkibA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

**枚举 WordPress 用户名**

  

  

```
wpscan -u http://192.168.10.44 --enumerate u  #枚举用户
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRejPaXMxGY2ZCGpAu5VowOr4TObSAPKdgGmoSHna6EMjFZoVw2kM9qOA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

**暴力破解密码**  

  

```
wpscan –u http://192.168.10.44 --wordlist /root/Desktop/dict.txt --username admin --threads 100 #指定用户名为admin，密码为 /root/Desktop/dict.txt 字典文件中的数据
```

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2ckkbwTsBvnDJpb89o8WMxvAKOaVnz60hOe7y3wAHiclddyK53lpEKIQlx4DKOq6EojHibVicgibDB2aQ/640?wx_fmt=gif)

来源：谢公子的博客

责编：浮夸

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640?wx_fmt=png)

如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)