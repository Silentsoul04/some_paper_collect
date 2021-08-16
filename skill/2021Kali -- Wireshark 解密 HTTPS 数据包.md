> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/kEszOL6IsQEVqFlrdnOXLw)

一、安装谷歌浏览器

1、下载安装包

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEevwTEOOibXurZs9IhtyPgR59O7uBE1VE2YUH6amyjsXSxbxkrCvT3RLA/640?wx_fmt=png)

2、安装 gdbit

```
gdebi google-chrome-stable_current_amd64.deb，提示安装gdbit，按y，继续安装
```

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEe56Wvf6KMxe3icicJOouiaibkxcwv7qBXO4olapNrFTLJReENswURnCjiarw/640?wx_fmt=png)

3、进入 root 模式，安装 

```
gdebi google-chrome-stable_current_amd64.deb
```

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEeOeyYp9TEJZYwRry2qWPSSH62sE6LhsexPI8ib7uIPicVdaO3G0yLxSuA/640?wx_fmt=png)

二、进入 root 模式启动 wireshark

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEebhoGpY6C5hphrWI68JRDRdzLUKic4WQaHHr5mc7ibW1z4KoX0Zx2wBrA/640?wx_fmt=png)

三、开启抓包

1、选择网卡

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEeIKD5tEDIQ6eUFhlcVO19Zwmmdzu06YjKOWZHFKhVpGqrZMset0Mryg/640?wx_fmt=png)

2、通过终端启动浏览器，访问百度，并过滤 ssl 的数据包，发现看到的都是些加密的内容

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEeWI1T7j5zH3FciaYyPGCicIoLbR1QHWspSAoh6QtOqEU5oTBRPYC3pf8w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEeeCaktlc4ibEzd3xVvbiaQXdatb8MHia5a7VQv6T19ia8dzyFkA1yLxr1Rg/640?wx_fmt=png)

四、重新打开一个终端，设置环境变量

1、新建一个 *.log 文件

2、设置变量

```
export SSLKEYLOGFILE=/home/aiyou/桌面/key.log
```

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEeB6NPunnho5CSwCiaheOcdg2GUpib3HC13tPOuHLtpe0amulicNYFOva3w/640?wx_fmt=png)

3、设置 wireshark，编辑 --》首选项 --》protocols

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEeNPfVscSniarl0gPlNSicicAQOyMHqcORv6V7jewnKRkT7z1s4rgEMl3hA/640?wx_fmt=png)

4、通过终端启动浏览器，重新抓包过滤 ssl，这些数据包就很容易看懂了

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuPg4icThxPZHtMZ6ZzQgKEeKWK9icFPoWsJqGBib4qqWQXzIohs1ucdqWoclf4Shqt0ggd17CURfNqA/640?wx_fmt=png)

禁止非法，后果自负

转自：web 安全工具库

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**