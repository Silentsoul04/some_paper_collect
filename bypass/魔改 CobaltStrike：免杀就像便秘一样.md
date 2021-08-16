> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/M673ni4Mt8VkW5gJ3Tl9Ig)

魔改 CobaltStrike：免杀就像便秘一样

  

1

一、概述

最近快开展护网行动，陆续有老哥问是否有免杀的马子。遂本篇文章简要介绍 CobaltStrike 的免杀方式。若有什么错误之处，请大伙指出，谢谢。下面主要分三部分来展开。

环境配置如下：

受害端：192.168.202.143、192.168.202.1

控制端：192.168.202.1

Teamserver:192.168.202.1

2

二、Stager 执行返回 Beacon 时的网络特征及规避网络测绘

当 CobaltStrike 的 stager 在受害端运行时，会请求 TeamServer 端拉取 Beacon 进行在内存中反射注入运行，有关 stager 端的详细分析可以看之前我所写的文章，https://www.52pojie.cn/thread-1334525-1-1.html。

先运行 stage，打开 wireshark 抓包：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOma5BIWOI3BPgKast5eMO80icPoWm61Sklysvykgguafj6CvDFUsSjEA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QO4zQ8PCktHCI8XzbZRibbOYXxTuu1CdhPjfWOABl7Uu5Dv8iaibedBBtwQ/640?wx_fmt=png)

可见受害端访问 teamserverhttp://192.168.202.1/7v9v，返回 206k 的数据包，结合之前上文的 CS 的 stager 分析文章可以得出响应得数据就是经过异或后的 Beacon 数据。

先来具体看看 stager 是如何生成相关 URI 的：调用 MSFURI()，传入参数 4：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QObicOOMlZ3mUqLV9zD6eOWWbxsN7MEUXkkYf5TW4ibmHibLtaViaQIhyRGw/640?wx_fmt=png)

跟入，可见传入的参数 4 表示所生成的 URI 为”/” 再假上 4 个字符串组成的，最后传入 checksum8() 后并判断结果是否为 92：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOLnYm5bNovkwGKyKaMO7EfOicjUd8OuA9Sic9QsBliaR9CDVcTj9yR3fPg/640?wx_fmt=png)

字符串由 var1 传入 pick() 方法来随机生成：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOia32UYOrCUTaibmess25nSQqCUxSufWiaXsEuUR3IDPibB6oOibhLKc2YzA/640?wx_fmt=png)

跟入 checksum8() 方法，该方法就是计算传入的 URI 字符（不包括 /）的 asc 值的总和，再把和与 256 进行求余：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOOMpTic5kV9mfKCs3HDib79fd39OiayMjenUxbvCXceicbVujmB9biazbSzg/640?wx_fmt=png)

所以，URI 的规则就是四个字符串的 asc 码之和，再把和值与 256 求余，若求余后的值等于 92 则生成该 URI 值。

现在来观察 TeamServer 是如何响应 stage 并返回 Beacon 的:

现跟入 NanoHTTPD 对应的线程中先接受请求头的数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOugTibUQdPiapbW4UOkg0IhMtxdYZeqgEYWJNq7RjymsJicKI1kRxk6CxQ/640?wx_fmt=png)

再将请求头的数据、请求方式和 URI 传入_serve() 方法，在该方法中先对 useragent 进行判断：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QO4mG4FqHiaLx4lXbAjmog3ONFmHTvTxC48gcm6Lq9bRxuVP1Ro0ZGkoA/640?wx_fmt=png)

紧接着把 URI 传入 checksum8() 函数进行规则判断：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOfwtsw8BkkHm9wlpdIcaWYEmia8JcRqSxasDib7UeSicAuES17x2t3SKKg/640?wx_fmt=png)

在 isStager() 中调用 checksum8() 函数对传入的 URI 进行判断，checksum8（）上述已介绍过：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOKYWxUyichBZDyuW0DUAQEicKJ9CILlP7uXI5ibNiaVaKIhHYGGmTQibeMTg/640?wx_fmt=png)

拼接相应的响应头和响应数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOKrjiciaa25h1zJn2tgCRDiaDXhgDomIicYwZsPc1ziaTgG0CHLDHzJtKGMg/640?wx_fmt=png)

最后调用 sendResponse() 发送数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOaYTmibFC30JO7955VicwX8GhXM4NCh2AQIb8ngSg1CicRTbXficnNs8Ocw/640?wx_fmt=png)

在该方法中每次发送 2048 个字节数据：  
![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOU47sgA9azFXSn6oJZEZg14x05uqbIpshJvA8MhlmISuZsV2xeYqDcA/640?wx_fmt=png)

直接使用浏览器访问该 URI，能返回相应的 Beacon:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOvHSRZuiba88wyMCias69Cbys3s3YGlbMaiapLYjonsdmDGR7Rz6KRM4ibg/640?wx_fmt=png)

所以目前有网络测绘技术对这种特征根据相应的算法拼接 URI 在公网上进行扫描，所以我们需要对其修改，需要修改的点主要有两个：

1、当控制端 Aggressor 生成 stage 时，调用 MSFURI() 生成请求 URI 这个函数后，再把生成的 URI 写入 stage；

2、WebServer 中的响应请求头的函数，当调用 isStager() 时会验证传入的 URI，所以可以修改 isStage() 该函数来个改变对 URI 的校验；

修改的方法有很多是比较灵活的，可以直接写定某个 URI，也可以换一种生成的规则，只要能避免网络测绘就可以了。

3

三、Shellcode 的转换为 C 代码

CobaltStrike 所生成的 shellcode 其实是一个使用 wininet 库及其对应函数的加载器，其作用就是用来下载对应的 stage，并在内存中反射注入。所以我们完全可以把对应 shellcode 转为 c 代码，然后进行一系列混淆等操作来实现免杀，这里混淆或加密就比较多发挥的空间了。

我直接把 C 代码所生成的 shellcode 写入，都没有任何加任何垃圾代码或者混淆和加密编码等操作就已经免杀了：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOnpeRMr4TMJGmBpictAQLtYLjiayX9ib5hf1GXJFJwia8eicpbzvedgO3EXQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOpseesxzaGOEHgoZhqTCWLJyCNlHPRIMkBNaNjITMt9LrccevntYG3Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOGGibbpVBGOs15XeFhrmCg6HtEYArt8LwKC1SGtQxpZvC036hqvKW37Q/640?wx_fmt=png)

执行上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOkVOgSQVdYGtcia313RVNUo9SvTXk79vVnhzoVJRwK8DLv9yx3ia0VN5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOeazPP6BI0XWskBMsqC07u2du5BN40Z1ibibWsPice4g2ob6jn3R0ianEFg/640?wx_fmt=png)

代码功能就是一个连接 Teamserver 端并下载 stage，并跳转执行，这里截取部分代码：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOykvoibgZWXCE6x5l3HXpTicgoDIcGcSBf7Eic8wdsYjiaiaibStfRl1WeDyg/640?wx_fmt=png)

另外我再用另一个库 winhttp 实现了该功能：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOWUiaAnSUSwI2F8TqqoLu4Peo0zLwaLmFVGwBNuCcciaQyPPib6pUNuHDw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOJTnEicrHyGZy1jVolrlia2exEh34fVvIXQ3EzUgZqmZ6ia798I2QSfEkg/640?wx_fmt=png)

均能正常上线的，这两份转 shellcode 为 C 的代码，上传到之前 CS 源码的地址，混淆可以自行加上去：

https://github.com/mai1zhi2/CobaltstrikeSource

4

四、Beacon 的内存查杀特征

虽然上述的 stager 已能免杀绝大部分的杀软，但是卡巴对内存反射注入加载的 beacon 查杀比较厉害，DOS 头的花指令对卡巴作用不大：

这里 DOS 头的前面是一串不定长的花指令，这串花指令应该是用来干扰 360/ 火绒内存查杀的，我们也可以 nop 掉，：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QO6T2IQIF7okKIc7AlLOibdbGED1JKhyEGegmQq5Fn4sBRVLia1SJwZWCw/640?wx_fmt=png)

花指令的后面就是异或解密 dll 的操作了，具体异或解密算法可以看我之前发的 CS 分析的帖子：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOncbMo9yLia430DGNjguxcUJKxWKefQEJGjXMNIJuIPFTaWT7gOOksGA/640?wx_fmt=png)

这里有妹纸分享无阶段生成的 beacon.exe 特征的帖子，https://xz.aliyun.com/t/9224?page=1，里面总结了 beacon 端被查杀的几个特征:

1.  读取了默认的 default. profile 内容，可以看看之前我发的帖子后门生成 beacon 部分里有提及，所以不要使用默认的 default. profile，需要加载自己 profile。
    
2.  导出反射注入的函数名字，默认生成是 ReflectiveLoader，这个也是需要在 profile 替换的。
    
3.  就是查杀 IAT 表，可以在 profile 配置 set obfuscate "true"
    

但有点比较遗憾的是，我复现该方法时，卡巴和火绒的静态和实时监控都过不了，可能是我操作步骤问题抑或是加了其他壳，同样我在 CS4.1 解密出的 dll 也是报毒的：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOFurQokjqH00BSKro0QNZsry6KrUiaibvsN0l2PQ3naOt7BooJOmgkMYg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOib9FVxZkHDGt6bLyLbvHZUib34y1Ux7GqCKIkgea61GBnlrDORvfHO2A/640?wx_fmt=png)

但 360 是能过掉的：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AAYVhncGibf79TcGXGd3v7QOyG2pa9YcL2Z6uvh2RRQoDhPKEYTOL28zicWkTmxXGiadInSLicyXp9ic2g/640?wx_fmt=png)

5

五、总结

如果能从规避网络测绘 ->shellcode 混淆 ->beacon 去特征（成功的话）这几步做下来，应该是能规避市面上一些杀软，其实在没有源码的情况下能做的动作还是比较有限和麻烦的，去特征码、加资源、伪造签名、搞 IAT 表、搞入口点等，小弟功力尚浅，希望老哥能发表高见，请老哥指导，谢谢大家观看。