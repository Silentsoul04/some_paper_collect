> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/f3dHgrm70jVAX2EViAyC8A)

假设我们现在控制了机器 A，机器 A 是一台可以出外网的机器，已经在 CS 上线了。

然后我们通过内网扫描 ms17_010，扫描到了机器 B。exp 执行成功，但是没有上线。因为机器 B 无法直接连接互联网，马就算执行了，也连接不到 CS teaserver 上。

这种情况下，我们有两种办法来使 B 在 CS 上上线。

**1. 通过代理上线**

我们在机器 A 上 8888 端口启动 http 代理, 我使用的是

> https://github.com/snail007/goproxy/releases

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwG1YXNQn8iaylZomyLiaW60CChSqkG7YjdfphefS0nungiazf4JzHOKwlQ/640?wx_fmt=png)

你也可以使用 cs 自带的 socks 代理，但是我这边测试好像是有问题，无法正常上线，不推荐使用。

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwh7vxibEfjQ4TXj0ic36SyNSic4rsKFibBjjz6htW5jib48G68vys3Ng1KoA/640?wx_fmt=png)

，新建一个监听器，打码的地方，正常填写公网地址，最重要的是红框中的地址

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwJwTzicGIv77ptlHaiaodjBTTibKWQeWa7xoyFsCBVTh9kCm0yHYyHkrgw/640?wx_fmt=png)

然后生成后门，选择带 s 的那个，要不然会无法上线。

流量经过代理后，成功上线

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwcATOPPKTWH3zCbMlcVxS0DAc1iao5AxpxbFKiaebZa47533rl1x7pmTQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwZpGQALdalZeMSoZeK2eZ593tCttxk5n9EFRKCMgWOibbOibrUyJfVvibA/640?wx_fmt=png)

执行命令成功![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwebXBib2lfjRtDgdd2pk8Tqaa9uM1W3ibEyicibWT37IO5Vlwa4QdT8Yr6A/640?wx_fmt=png)

**2. 通过正向端口上线**

新建监听器 T3，4444 端口不要动

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwTnHdeA9WuKu9FtTwEGoszvDf6xM6uGAGhrBOTwKdjRumTkw1nCMdOQ/640?wx_fmt=png)

同样也是生成带 S 的马，在 B 机器上运行以后。查看端口状态，4444 端口已经监听成功

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwUL5tLRK5dVcTme28OH5xZ9ZRicS712OfXHyPdpicQ6CF8wfrQCuXuqdg/640?wx_fmt=png)

进入 A 机器的 beacon

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwtzDiacxPUveazqKfUuBm1d3h7O9ZqOhyNjI0dGYDLjibNaib6ia0UqTUTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwOtfCfwZl8lAjGPibeUvuhd3B2WIbf0aMLVcE65SLhuVba4gjNgSYPow/640?wx_fmt=png)

命令执行成功![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1e60q3SBGCqb1C4vic55BHUwyBtlATFBocVhtY4fJrEcibM6BPYbsw0frgm7z86dxy3kKSmCLMPT20A/640?wx_fmt=png)