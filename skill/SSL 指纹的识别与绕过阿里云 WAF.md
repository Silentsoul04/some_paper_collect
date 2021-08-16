> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4JY80P0NkxQ346l8Xlf3fA)

![](https://mmbiz.qpic.cn/mmbiz_png/TN05MmJLxMqMTrcicggdTRKW3Iod235KxvyRglNB6RS0salXVC4VZsT7syP2xm3O6VcwqHn2y3AGzODAAzgDibnQ/640?wx_fmt=png)

  

**在一次友情访问中，发现使用不同的客户端发起请求会得到不同的响应，就很好奇为什么会这样？**

![](https://mmbiz.qpic.cn/mmbiz_png/US10Gcd0tQFGib3mCxJr4oMx1yp1ExzTEVCfibVear8YU1iaINh9dfwzSpc5YOkov9iaFibH1rGpbP9CW1LjukkYOkA/640?wx_fmt=png)

  

如下面，使用 Burp 和 python 请求：  

    Burp：  

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ55h62zaPtxWthbtf2hpQTwiciaZianTLJsX0RGjBLzBkTiaictyuic6N3ddzg/640?wx_fmt=png)

python Requests：

    同样的请求复制到 python3 中用 requests 发包:

```
<body data-spm="7663354">
  <div data-spm="1998410538">
    <div class="header">
      <div class="container">
        <div class="message">
          很抱歉，由于您访问的URL有可能对网站造成安全威胁，您的访问被阻断。
          <div>您的请求ID是: <strong>
276aedd416186716424122798e3951</strong></div>
        </div>
      </div>
    </div>
    <div class="main">
      <div class="container">
```

    一样的请求地址一样的参数一样的 http header，burp 发送的请求正常响应，python 发送的被 waf 拦截，curl 模拟请求也被拦截。

    waf 是阿里云的 waf，dig 域名也能看出来 ，cname 解析到了上面。

    多地 ping 发现并没有 cdn，不是 cdn 的 waf。

**其实第一种解决方法已经出来了，直接 ping 域名获取真实 ip，request 直接请求 ip 地址，在 Header 中指定 Host 即可绕过 waf 的弱智拦截，但是如果有 CDN 怎么办呢？**

    本来以为是 ua 的问题，后来更换了 ua 发现并没有什么卵用

    问了问朋友，说 python 的 tls 握手有特征

    在网上搜了一下，发现确实有很多类似的问题

相关链接：  

*   https://stackoverflow.com/questions/64967706/python-requests-https-code-403-without-but-code-200-when-using-burpsuite
    
*   https://stackoverflow.com/questions/63343106/how-to-avoid-request-fingerprinted-in-python
    
*   https://stackoverflow.com/questions/60407057/python-requests-being-fingerprinted
    

  
**我们来看一下不同客户端的 clienthello 报文：**

    客户端发起 https 的请求第一步是向服务器发送 tls 握手请求，其中就包含了客户端的一些特征。

    相关内容在 tls 协议报文中 Client Hello 的 Transport Layer Security 当中。

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ52oicFWDWa7hzibZxiaiajyicmLCAJSyicKKFnrhILmibbmMDATMBKT32cCQ7g/640?wx_fmt=png)

对比一下 burp 和 requests 的 Client Hello 有什么区别：

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ5JibkX9TprLdFSZzLUntwv746iaFkNmic9vrb2C0A93z3gz00IcSM7acQg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ57gLqP7VeJOWrP9CiaZxHpicP6RPutbCkW5g11eREmPIkHSIBZ9WW08qg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ54VoC9HKr1FzBebM1dhAwBuiahXfRiaUVUcD0ozCIP2QzUr2xZ1Tem5aQ/640?wx_fmt=png)

一些不同的点：

<table><tbody><tr><td width="268" valign="top">Burp Suite</td><td width="268" valign="top">Requests</td></tr><tr><td width="268" valign="top"><p>TLSv1.2 Record Layer: Handshake&nbsp;</p><p>Protocol: Client Hello</p><p>Version: TLS 1.2 (0x0303)</p><p>Length: 466</p><p>Cipher Suites Length: 104</p><p>Cipher Suites (52 suites)</p><p>Cipher Suite: TLS_AES_128_GCM_SHA256 (0x1301)</p><p>Cipher Suite: TLS_AES_256_GCM_SHA384 (0x1302)</p><p>Cipher Suite:&nbsp;</p><p>TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (0xc02c)</p><p>…..</p><p>Extensions Length: 285</p><p>Extension: signature_algorithms (len=46)</p><p>Extension: supported_versions (len=11)</p></td><td width="268" valign="top"><p>TLSv1.2 Record Layer: Handshake&nbsp;</p><p>Protocol: Client Hello</p><p>Version: TLS 1.0 (0x0301)</p><p>Length: 338</p><p>Cipher Suites Length: 86</p><p>Cipher Suites (43 suites)</p><p>Cipher Suite:&nbsp;</p><p>TLS_AES_256_GCM_SHA384 (0x1302)</p><p>Cipher Suite:&nbsp;</p><p>TLS_CHACHA20_POLY1305_SHA256 (0x1303)</p><p>Cipher Suite:&nbsp;</p><p>TLS_AES_128_GCM_SHA256 (0x1301)</p><p>…..</p><p>Extensions Length: 175</p><p>Extension: signature_algorithms (len=48)</p><p>Extension: supported_versions (len=9)</p></td></tr></tbody></table>

可以看到很多地方都存在差异，主要为支持的协议，length，支持的加密套件，加密套件的排列方式等。  

多次请求可以发现相同的客户端发起请求，除了 Random 和 Session ID 之外其他内容是完全一样的。
-------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ5THbhT3iaU6dkCF0DbeCsrWplTfxCiclhC6wlKCG6F64AmIOev0Y8KAwQ/640?wx_fmt=png)

所以这里面固定的内容其实就可以作为指纹来进行识别

比如加密套件 13 02-00 ff 在 tcp 流当中就有固定的字节顺序

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ5UIOPuyXdHQUzIicxQztGiao1FRyNaHicnhkNic8WOfSaY2NrAKKgngxibqg/640?wx_fmt=png)

**TLS Fingerprint 识别原理**

相关链接：

*   https://idea.popcount.org/2012-06-17-ssl-fingerprinting-for-p0f/
    
*   https://github.com/salesforce/ja3
    

看了下 ja3 的指纹计算规则：

**The field order is as follows:

```
SSLVersion,Cipher,SSLExtension,EllipticCurve,EllipticCurvePointFormat
```

Example:  

```
769,47-53-5-10-49161-49162-49171-49172-50-56-19-4,0-10-11,23-24-25,0
```



**

    把版本，加密套件，扩展等内容按顺序排列然后计算 hash 值，便可得到一个客户端的 TLS FingerPrint，waf 防护规则其实就是整理提取一些常见的非浏览器客户端 requests，curl 的指纹然后在客户端发起 https 请求时进行识别并拦截

**Bypass**

**除了 TLS 指纹，对 User-Agent 也是有对应拦截，如果使用带有 UA 特征的客户端那么 UA 也是需要更改的**

**1、访问源 IP 指定 host 绕过 waf**  

    上面提到过，套了阿里云 waf 的服务器 cname 解析到了 yundunwaf3.com 的域名，这种情况可以直接 ping 域名获取真实 ip，然后请求地址设置为真实 ip 在 HTTP Header 的 Host 字段中指定域名即可绕过 waf 的防护

**2、代理中转请求**

    在本地启动代理服务器，如 Burp Suite，发起 http 请求时指定代理服务器为 burp 的地址，让 burp 来进行 TLS 握手，算是一种曲线救国的方法

```
import requests
proxies = {
'http': 'http://127.0.0.1:8080',
'https': 'http://127.0.0.1:8080'
}
rsp=requests.get(url,proxies=proxies)
```

    当然这种方案需要找一个不会被拦截的客户端代理才可以，试了几个 go 写的代理如 goproxy 发现仍然被拦截。

  

**3、更换 request 库**  

    Requests 其实是对 urllib3 的一个封装，那 python 有没有不用 urllib 的 http request 库呢？

    翻了翻 aiohttp 的源码发现貌似并没有用 urllib3，抓包发现 tls 指纹和 requests 也有着明显的差异，实际测试 aiohttp 确实没有被拦截

**4、魔改 request**

    从根本上解决问题，debug 跟踪到了几处可能可以修改 TLS 握手特征的代码

举例：  

/usr/local/lib/python3.9/site-packages/urllib3/util/ssl_.py

```
DEFAULT_CIPHERS = ":".join(
    [
        "ECDHE+AESGCM",
        "ECDHE+CHACHA20",
        "DHE+AESGCM",
        "DHE+CHACHA20",
        "ECDH+AESGCM",
        "DH+AESGCM",
        "ECDH+AES",
        "DH+AES",
        "RSA+AESGCM",
        "RSA+AES",
        "!aNULL",
        "!eNULL",
        "!MD5",
        "!DSS",
]
)
```

`DEFAULT_CIPHERS`中定义了一部分的加密套件，直接进行一个删除，当然其他能改的地方也很多

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ5ECoaYib4rhe1Z04fVkPL6uBu9iaNzBH3QnQicQVV0XJ4Sl5Q55jsibiaPcQ/640?wx_fmt=png)

成功绕过了阿里云的拦截

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibA45lkxSbhgQkb3dnyECzZ5HJH56nQia1KkCX0NkGecARYTa7jJaIice1CPHPbaRFQqV8CvscFfHS9Q/640?wx_fmt=png)

原本内容：

```
Cipher Suites Length: 86
Cipher Suites (43 suites)
```

修改后内容：

```
Cipher Suites Length: 80
Cipher Suites (40 suites)
```

加密套件的内容发生了变化，使得 Finger Print 和原本 requests 不一致。

理论上其他客户端也可以进行修改代码实现变更 TLS 指纹的操作，但是如 java,go 等编译型语言写的工具在没有源码的情况下修改会很麻烦。

                                                                                                技术出自 Ares'X Blog

  

![](https://mmbiz.qpic.cn/mmbiz_png/jaXicGP9Gute45GJV6IhEbKTialwib0DIMPOqPHicWLn4Q7ft1iayZBSUp9ATFGuIZ8lIQOaibpO6IKMhwzmRFzYsPvQ/640?wx_fmt=png)

  

  

  

  

往期精彩：

[【超详细 | 附 EXP】Weblogic 新的 RCE 漏洞 CVE-2021-2394](http://mp.weixin.qq.com/s?__biz=Mzg2OTU4OTM0Ng==&mid=2247486945&idx=1&sn=392083ccb576058771c2a295f2fb74b3&chksm=ce9b85ecf9ec0cfa82c3ba8390ec088d0a05c28fb166bba67a799450204f3eb9a17538dcc28e&scene=21#wechat_redirect)  

[【入侵检测】Linux 机器中马排查思路](http://mp.weixin.qq.com/s?__biz=Mzg2OTU4OTM0Ng==&mid=2247486733&idx=1&sn=3d05d9a69aed5b5f6e468e54aeb1125b&chksm=ce9b8500f9ec0c1605eb9c88606f1015b340946002fca101403bbeaca6b9739e3fef2d839bcb&scene=21#wechat_redirect)  

[【入侵检测】Windows 机器中马排查思路](http://mp.weixin.qq.com/s?__biz=Mzg2OTU4OTM0Ng==&mid=2247485509&idx=1&sn=f34a72d2fc5788e1fb72b7e6f2ed87e2&chksm=ce9b8048f9ec095ec5c297e47a8eadcc741f62522086f208064dff63ced2744780aa6e4774a8&scene=21#wechat_redirect)  

[【代码审计】对一套钓鱼网站的代码审计](http://mp.weixin.qq.com/s?__biz=Mzg2OTU4OTM0Ng==&mid=2247484822&idx=1&sn=96e5b2650c58f10cdeb0d67061c8c65c&chksm=ce9b8d9bf9ec048d26435dfb6c895331ff1dbe6a10f1f4c12ffa777e75c215b39747681e71b4&scene=21#wechat_redirect)

 [Web 安全——登录页面渗透测试思路整理](http://mp.weixin.qq.com/s?__biz=Mzg2OTU4OTM0Ng==&mid=2247484739&idx=1&sn=a75feb8c3d10c9799b515694e0c067de&chksm=ce9b8d4ef9ec04584bbd4ace52dbdb4e6147e08227dd1fb6714340785cca2e6c167a64929493&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/9PzRadicuYUallMqWQx9qjgpvutvJ4ys5nnfMia5hakf5pErfv5NFSEQ9EYccDqnBXlhjhnvkniaQ5nNxXHqM1lwQ/640?wx_fmt=png)

  

点个在看你最好看

  

![](https://mmbiz.qpic.cn/mmbiz_png/p7SdVOXjnELHTh2b4daFLRpUN0icsRjVSSjELS3Ty82PbKmvZppLzB13Z19zW2h0239l4w1quRCs1krCUuUCAxg/640?wx_fmt=png)际测试 aiohttp 确实没有被拦截 tp 确实没有被拦截