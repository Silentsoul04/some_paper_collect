> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aZJGLf20T7Cep-9yH1DP7g)

> 日期：2021-05-08
> 
> 作者：pbfochk
> 
> 来源：宸极实验室
> 
>   介绍：本篇文章主要讲述常规反溯源技术中利用 `CDN+域名` 对自己的真实`VPS`地址进行隐藏。

0x00 前言
-------

红蓝对抗演练中，蓝方对红方 `IP` 地址的溯源可以对红方造成很大阻碍，红队有效地隐藏自己的真实地址在实战中起着重要作用。

本文讲述通过 `CDN+域名` 方式隐藏木马在回连过程中的真实地址，从而规避被蓝方溯源的风险。

0x01 域名准备
---------

### 1.1 注册域名

首先我们需要一个域名来与 `CDN` 相互配合， `VPS` 所映射的域名肯定不能用自己的域名，一不小心就来个当场被捕。这次推荐使用国外免费域名网站 _https://www.freenom.com_ ，随便注册一个 `.tk` 域名。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicWdfvYH4KCtT0LHpzhEvZ2uw6C0l6KbrFsGVbIyMEWhPN1PUjktiaibvA/640?wx_fmt=png)

### 1.2 更改 DNS

绑定自己 `VPS` 地址并测试可用后，需要更换 `DNS` 解析地址（名称服务器下文 `CDN` 会提到）。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xic4ialGon1ricBeEA26fBTsFVW25rbbH4Hr3Z8DMOedlvcvzpDKhCibfiakQ/640?wx_fmt=png)

0x02 CDN
--------

### 2.1 CDN 准备

`CDN` 的全称是 `Content Delivery Network` ，即内容分发网络，这里我们注册免费 `CDN` _https://dash.cloudflare.com_ ，注册完成后填写之前注册好的 `.tk` 域名。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicjuUtPre4gngZwvfgtKd9sZuN3POxOzAiadPvX48BZMhWlrjQt7ibZXKw/640?wx_fmt=png)

### 2.2 获取 DNS 地址  

进入 `CDN` 设置界面，在 `Cloudflare` 名称服务器处会有两条 `DNS` 服务器地址。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xiccMKfwy72O5rUQibptRPkibAray67qiadq6SmMLicwM009ZsJpvD0Eqe9EQ/640?wx_fmt=png)

### 2.3 域名更换 DNS 服务器  

返回 `freenom` 域名 `Nameservers` 处，更换 `Cloudflare` 名称服务器处的两条 `DNS` 服务器地址。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicE5Lra2XE2C3dQU3xkSSiaNLuudtCGuuuGFVlibED3icn1ibt1F5w6aoNyA/640?wx_fmt=png)

### 2.4 开启缓存

进入缓存设置处开启缓存，否则访问域名时会出现更新延迟或禁止访问。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicmRWS0pJp06LXOwMmgicDibHvGl0wGx7lgkRkb3dJ1fUGLonYkYBos7nQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicr4uv9g66NRWWfSdTOsiacibK1Nryicmxs3MrPjkN8nFp2X7mqFhldQWiaQ/640?wx_fmt=png)

0x03 cobalt strike 溯源测试
-----------------------

### 3.1 常规查询

（1）配置完域名与 `CDN` ，使用 `ping` 命令测试域名效果。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicvlABzj103Dghj6MlRxwh7el5iaY0N9yazCHlGBibwG1C3ggaM0sXnCag/640?wx_fmt=png)

（2）可以看到域名回显地址已经不是我们`VPS`的真实地址了，我们使用搜索引擎测试。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicEXMyVbsRQzKSj5o08MkrmSicVVsIvz7X52FeAESVZVGGIrRcFiaIERTQ/640?wx_fmt=png)

（3）因为是国外免备案域名，`whois` 查询域名查不到任何信息。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xic0dOg2Xn88SNr7M5G6nEdgo81WwCZxZ32RTKMOo4uPQgiaAqBVfnVDrw/640?wx_fmt=png)

### 3.2 Cobalt Strike 监听器

新建监听器 `windows/beacon_http/reverse_http` 或 `windows/beacon_https/reverse_https` 均可， `host` 与 `beacons` 设置为域名， `port` 设置为支持的端口，以下图为例。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicic9Zo9qmuZG5vgIHicbovuTjym8ib58hibusPLT8tZGVa2tJUHdE3K2Ixg/640?wx_fmt=png)

`Cloudflare` 支持的 HTTP 端口：

```
80,8080,8880,2052,2082,2086,2095;

```

`Cloudflare` 支持的 HTTPS 端口：

```
443,2053,2083,2087,2096,8443;

```

### 3.3 Cobalt Strike 回连对比

（1）首先测试常规情况下生成的 `payload` 连接行为。执行过程分析：

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicRr0kZrufVsPuEHbHQKkaDTRaCt3uVOSsXMib7snrO7NZrAPsjGiasWqA/640?wx_fmt=png)

网络行为分析：

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xic7BCvyW63IAYJaIAVHTaa2tZZPB0NPecVIvZx1GAQZ2sWTlhb1DicVbQ/640?wx_fmt=png)

可以看到通过普通的溯源手段即可获取 VPS 的真实地址。

（2）接下来测试域名与 `CDN` 流量转发后的 `payload` 连接行为。执行过程分析：

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicVrWWAJQZLJMFXcP2lgnT9hjETXsbALNpBVpfZxSy5picPV41iaoZIdLw/640?wx_fmt=png)

网络行为分析：

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicbBz8HjYX9n3Vh6J30tp5wSCB0mg2agZZDpaicm77UfT7Tib1JGNLYsPQ/640?wx_fmt=png)

这时指向地址显示的是我们之前设置好的域名，指向的`IP`为 `CDN` 的地址。

（3）生成的 `payload` 在上线机器的端口占用显示为 `CDN` 的 IP 地址。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xictzB7R08RibB9FRLWHBkt8PuOqIgicu8TNylu9UomeESepCYZwEhZeE6A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicZCFzzPN1b9cbukdOm3NV04o7FaOE4jlgZMjaT3s5Zefvt7ylukA6kw/640?wx_fmt=png)

（4）目标机器通过 `CDN` 转发上线。

![](https://mmbiz.qpic.cn/mmbiz_png/4136w7o9Jve6kw2vwuefhzGfoTsEu0xicQmeB0UNcwhiaeFiaeLLPe2fzAXu9aVhOn2iaEib7p1Ln5O7yf2f0C89cIQ/640?wx_fmt=png)

0x04 总结
-------

本文内容主要应用于通过 `CDN` 的内容分发来对自己真实的 `C2 IP` 地址进行隐藏，`CDN`隐藏方法相对于域前置技术来说更加方便部署，相对于重定向方法来说暴露风险更低，是反溯源手段中比较推荐的一种方式。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**