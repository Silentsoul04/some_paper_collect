> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9n9TheaNUh39FNttRM4kHA)

fofa 语句：  

title="无线 smartWeb-- 登录页面"

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8FwVgYkapmCc6SskEpUTPOBp414iaQfKia9yuvVHAxQZ0Nlic0qqcYnTVyA/640?wx_fmt=png)

我们随便打开一个  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8FMd8fgchdicPlyicdSvia0TibfV5xfEIU40s7hBap5rIIu2AjA5zmxEcKBA/640?wx_fmt=png)

用 guest/guest 登陆管理系统，登陆完后，刷新一些后台首页，看一下 network

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8Fkq0CrfbuspjrLEiaXwUN44J50TpJhicliclaKRxiaB3zy79m1Q6NJYibUUg/640?wx_fmt=png)

密码去 base64 解密一下就好  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8F6McdlGS0Vg3nW4AgOPHOtAm4Ft60hC2SfhiaIyichFwENH1L77tHIBOA/640?wx_fmt=png)

然后再去登陆，已经是管理员权限  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8FFXqdTyBpYsbF8XwAYmkO3qearict7YiaSTmB0wzvHTSOZyETySGQGTgw/640?wx_fmt=png)

当然我们还可以使用下面的 POC，一样会返回管理员密码  

http://xxx.xxx.xxx.xxx/web/xml/webuser-auth.xml

参考链接：  

https://mp.weixin.qq.com/s/nvk4Nu8q8AxeuPDPygllWA

当然更多渗透测试的内容，也可以到星球里学习  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8FKV3NktCRVJ7W14sblwk73stL4P86DViaCoG069BgrcIFVZShmW6ZbMA/640?wx_fmt=png)