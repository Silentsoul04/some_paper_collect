> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CsJzMFm4qaGLmF67jbjhXw)

这里以 Jellyfin 任意文件读取漏洞为例子

一、如何用 goby 自定义 poc

启动 goby，填写漏洞名称，fofa 查询语句  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJfbiaciazb7eboOwWXbFuX4JYxv5LEToPfym92eYyrgqZjaV9Yhx7yN7g/640?wx_fmt=png)

点一下提交，然后点测试，到这里开始编写 poc，测试 URL 那里填写 URI，可以是 GET/POST，请求头填写主要的，也就是关键的

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJ6rPAJ4MW9Sc58zxHgcuJwgpLx7WFVcISM3smSghibs0SOgjKmvKPKIQ/640?wx_fmt=png)

往下拉，填写返回包信息，这里依旧是填写关键的，有标准性，能决定是否存在漏洞的，这里返回状态码要 200，HTTP 正文填写 font，因为这个漏洞会返回一个下载文件，而且 response 包也包含 font，可以作为是否存在漏洞的标记，方便下一步更准确的利用漏洞  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJvBEWjLcm5ic0icuQ1IhRyHYicuq71ZFLu7NjmWV5ngWEylbZgW7HxoiaBQ/640?wx_fmt=png)

写完之后，导出 poc，没错是一个 json 文件  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJFr9jUZGbkgnIIAabbibLMVgeGnVsalcM8Tn1HxEUFVlkGOwLciaicZu1Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJlXdow3YOQZgEgW9ESvVRxJKc2jyAc1c60xbHPm0jmkftYib6FBQJENQ/640?wx_fmt=png)

打开后是这个样子  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJwDANDmcOB23HqiaVwr0YaurLOB4UpSPI6PE9tfxj9BzEWibm9qMerBNw/640?wx_fmt=png)

当然如果遇到 RCE 漏洞，我们可以在 poc 里添加漏洞利用步骤，内容跟原 poc 相差无几，实际上是传递了要执行命令的变量  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJy19JZMSicjuCfY4CMHCvXibj2SsYeJ3wDYibjXGyg8xmIeTEKdIt7ibhyg/640?wx_fmt=png)

二、如何利用导入 poc 并且结合 goby 进行批量漏洞扫描  

先导入一下 poc  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJ9FkLqazvZucQQIcIsjTmaPSvkcm31fWodraGWWK2uN9TRbwF8qU1jA/640?wx_fmt=png)

导入完后，回到扫描页面，打开 fofa  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJTTB3cO0Xpqu8wLibF8uZ3D4ia2D3yjXFGrFfsj82XccicyCQziblKPL5zg/640?wx_fmt=png)

输入 fofa 查询语句，点查询，查询完后，再点一下导入当前页

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJ6cCZHEt8xvVmmcPu8ico2Z1rlic1RiaKPa7EzQAn1LcicavIjOHcFjvS0g/640?wx_fmt=png)

然后就是开始扫描  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJ2qo5SwpmuHibf347xeYoiaEjJZG7mhlFuHyPtfPX6WJxicNJum6Hib2aHQ/640?wx_fmt=png)

批量扫描结束后，会显示存在漏洞的 ip  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJ4FGZ3zJRF4wczAABRQoWd4J3AYiaZibehdcG6wmekENdEslfKs8Tkwiaw/640?wx_fmt=png)

接着就是验证是否真的存在漏洞  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv04ONlbqbzYTGz7MHomyPEJHduXUrfoOic07wmngv9zERKB55jFzOtguHiaGZmYvenAUcbaQtAguJeQ/640?wx_fmt=png)

到这里就结束了  

更多渗透测试内容，可以到星球学习  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8FKV3NktCRVJ7W14sblwk73stL4P86DViaCoG069BgrcIFVZShmW6ZbMA/640?wx_fmt=png)