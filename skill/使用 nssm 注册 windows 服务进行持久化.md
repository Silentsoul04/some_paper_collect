> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jwVYOR8zbP_hsKVvzJYUOA)

前言
==

NSSM 是一款注册 Windows 系统服务的工具。将应用程序注册为 windows 系统服务可以使其在系统停止、重启等情况仍可自动运行，可用于持久化后门。

下载
==

http://www.nssm.cc/download

使用
==

解压并在 nssm.exe 目录打开 cmd

使用如下命令注册服务

```
nssm install 服务名
```

填写木马路径和服务名，可以填写欺骗性较强的服务名

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4EmYmb8a293cricwyOf8uh7FdrHBuUhZRykadSCamCu1CMCk0ribr6H0Q/640?wx_fmt=png)

在服务里设置自动启动

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4yZyFOG2fAjkD3sNEd5PiaB8TTFJ8pg1QSibibzVwhXibxI0icCfPj6Ly4dg/640?wx_fmt=png)

重启对方机器，在 msf 中开启监听，成功得到 shell

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4GiaicC9zQT5e9jx5CsbXg0xYr6oJADyp1Gu4JLtcnGic2xM3VsOCAoV2w/640?wx_fmt=png)

作者：Leticia's Blog ，详情点击阅读原文。

**推荐阅读**[**![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyIDG0WicDG27ztM2s7iaVSWKiaPdxYic8tYjCatQzf9FicdZiar5r7f7OgcbY4jFaTTQ3HibkFZIWEzrsGg/640?wx_fmt=png)**](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247496904&idx=1&sn=e6c717bc2709f7c4ec8523bc681f43f3&chksm=9acd2457adbaad4169ed38ebf0d969553b6cf4dee26307f2ce6a137c52849e4ffdf06325347e&scene=21#wechat_redirect)  

公众号

**觉得不错点个 **“赞”**、“在看”，支持下小编****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**