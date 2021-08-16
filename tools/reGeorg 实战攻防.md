> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Rkv5yGlJk25agGowWIqMQQ)

这是 **酒仙桥六号部队** 的第 **140** 篇文章。

全文共计 1620 个字，预计阅读时长 6 分钟。

**前言**

当我们已经通过各种操作 getshell 之后想要进行内网横向渗透，但因为目标 ACL 策略设置的比较严格，只允许 HTTP 协议和对应端口通过。我们无法通过使用端口转发或者是端口映射的方法来从外网访问到内网的其他机器。这时我们就会想到 reGeorg 这款工具，通过该工具代理进入内网，通过 HTTP 协议转发请求。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaH5ts7Xhm8l8aNzhyrHE32WkraKomlHfXe0QVqKGanLHj2ibmNrZ2ibCdw/640?wx_fmt=png)

这个工具创建之初本意并不是专门用来渗透内网，而是某些企业员工在外网的环境下想访问内网资源。所以这几个安全意识不太足的小哥们写了一个可以通过部署在边界上的网页来进行流量转发，从而访问内网的一个办公工具......

可以看到该工具的'斯搂梗'是说 "每个办公室都需要这样的工具"..

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s567qHhF22JtclMU8muN45iaHUTznaTvP6VY2KEr4b8TUUoWLuhbpaBdkjib2o7hhp8sTd9ouKY2axPQ/640?wx_fmt=jpeg)

**源码分析**

那么如此厉害的工具究竟是怎么实现的呢？我们一起来看下源码：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHlvCVESOsgmDmJSEDESMlOg7JGPLLibhSBsSHPQLFhN39KCwX7CdyJvA/640?wx_fmt=png)

从入口开始，就是标准的一套：LOGO + argparse 来进行参数的支持和解析，真正逻辑从 askGeorg 函数开始。这个函数是来测试远程代理服务器是否能够访问，我们来看下这个函数的具体内容：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHcgkFFHORUj4DQRdhLhKzTKt3CEs61pWVMdtl1Zxiafdpej9vgBpGGcw/640?wx_fmt=png)

可以看到内容基本就是判断是否为 HTTPS，然后使用哪个工具。用 GET 方法来请求，如果状态码为 200 且内容跟远程服务器中内容一样就认为是 OK 的。

比较的内容就是浏览器访问看到的那一句话：

Python 中：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaH0MeKrjicDcBxA3KWrU4ia1KgqG9iaiazR9L3WBt9r5CCcgbP6IHkLLUUOw/640?wx_fmt=png)

php 中：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHp650SKGgYXAzQfcRyZzvYaHC68Lvia5ADJHmxmL9HARfnEbKsmHvNSg/640?wx_fmt=png)

浏览器访问：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHtgo25Y50AHicwUiboUGZIg4CgaeY6qKsBOb9LvibmvD4AkibR0csfCO7XQ/640?wx_fmt=png)

我们来继续往下看：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHiajUpwVcTV8554HBG9NFiaibLjzHJ5cibicicc2SEsDMjVOzKO3xwz54JTww/640?wx_fmt=png)

监听了客户端的端口，并设置 TCP 的排队上限为 1000，这样的对普通情况来说是足够了。

后边是创建循环不停的接收报文，并且将接收到的传入 session 线程中并启动。

session 的构造比较简单：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaH4Ccz6SsfBWvppy4b7zheq9Y4Drb0FZN39T9Ffba5vngkXYMTAXauibg/640?wx_fmt=png)

我们来看下线程最重要 run 中的内容：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHribSqn92EjW330bIO0f8nCGJyWGAGzWdxEKKreajgJgfr3yKjvPa04w/640?wx_fmt=png)

内容不多，就是判断 Socks4 还是 5 并解析，之后是创建读写线程并 start。

判断 Socks4 还是 5：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHFicp1MgicJ7qs2tvGK3BAIhenJQLxiaxjYBSTburDLRAETbGRSwiatXgQg/640?wx_fmt=png)

Socks 代理至少三个字节的请求, 第一个字节一定为 5，如果是 Socks4，则第一个字节一定为 4。

parseSocks5 和 parseSocks4 为判断对应 Socks 的协议解析是否成功。

Reader：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHY8TGjrdAo98UpuhZYVeERBetQ5x1BQONVxh2s0jaZsAaUia8Ebhhxxg/640?wx_fmt=png)

Writer：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHFJpvJftpNROulFHiccMEfd4RvaoNtDGc8fk98Q5AaiaL5m2QIYgwDE8A/640?wx_fmt=png)

读写部分是一些转发的常规操作。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHjzBpjiaKnaOTZMJjtpZfvjBFdVVQwicUvNrzLkTNiahYlV70jJuv6kslw/640?wx_fmt=png)

**实战攻防**

在实战中使用可能会碰到一些特殊问题。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaH7X8YL0o3QStCOh2ANia18ZmIY0L1UXRjlRac18XmFghEVT1W7PXibuzw/640?wx_fmt=png)

比如在浏览器中访问可以出现熟悉的 "Georg says，'All seems fine'"，说明可以正常访问。但是使用 reGeorgSocksProxy 客户端的时候会报'未准备好，请检查 url'，这是为什么呢？

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHpl9BT5JLVrgQpt3dEtDSpwyv2DTwZMg5ial9HEUldE2Eksza9rIAQzQ/640?wx_fmt=png)

排查问题需要进行一些代码调试。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHfoP48vIMnjaP3nvhuViagE7skiamLHO32lvkFic0wdXtDicPOlAtX2HKXA/640?wx_fmt=png)

通过打印出的关键字搜索，可以看到是 askGeorg 这个函数返回了 False 导致了退出程序。

这时我们可以进行调试，使用 Debug 来跟进代码，一行一行看到底哪里出错了。当不具备调试环境时也可以使用打印的方法定位问题。

这里我们使用打印的方法来定位问题。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHyUibibwFaPg6FldcUxfBP23gkfcVorOO9Sp6sL70u0WqtTuASQAVWxOg/640?wx_fmt=png)

我们再尝试运行一下代码，看看哪里出错。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHLaxwGb6ibLuc0LhRZ9pPq2h2icsCoJK18f0S5VUnLeNXVZRwRJYoOr1A/640?wx_fmt=png)

可以看到返回的状态码为 403，也就是说可能被 WAF 或者其他安全设备拦截掉了。我们通过代码可以获知只有当状态码为 200 的时候才可以正常使用，并且我们使用浏览器直接打开是可以正常访问的。也就是说我们的问题出现在了 Python 脚本跟浏览器的请求差异上，比如一些常见的请求头 User-Agent 、Accept-Language 等，这些我们需要一一补上。

我们需要将每一个请求都加入浏览器所包含的请求头，所以我们将该过程提取出来作为函数使用。

修改后的代码：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHmQ3SsJHxhIogQyh9TicmFZr9yLIJAbQlxHMs4x17dVy5e8lYB811BtA/640?wx_fmt=png)

setupRemoteSession 中的 CONNECT：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHDWmtmbaSI82pXVYoTaWpicibMVy1DSnmkCprd6pZQ2b57FqWQu7cw7rA/640?wx_fmt=png)

closeRemoteSession 中的 DISCONNECT:

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHvpMIKBmoJbrCcHA4oeSXhuq8DnibQJ9DkIVwkGiblZEEg1KUdjsicYOOg/640?wx_fmt=png)

reader 中的 READ:

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHSwoA19tmjwBia8LiaLt65PaReLOoROcPwbiarB6JKYaXnMVA4beb25U3A/640?wx_fmt=png)

Writer 中的 FORWORD:

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaHnkhvLoQ4ciac9t1YWc6zeghtmqaWM0410QaiaICFusDa3dA4DUgk1Ikw/640?wx_fmt=png)

修改完代码后我们实际进行测试。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s567qHhF22JtclMU8muN45iaH2U2omqngfx9iaV8HXEQXKkwzibMvMLtAomcibqGty2P9m2zicIVSVN3fhg/640?wx_fmt=png)

可以看到返回码已经变为 200 并且打印出了熟悉的'All seems fine'，说明可以正常使用了。Happy~

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s567qHhF22JtclMU8muN45iaHGtMeO03RibMkug5ZibBygcurfZImMWBq4maibjjWGOkqjTGsib4w9gxtbQ/640?wx_fmt=jpeg)

**总结**

掌握调试 / 打印等方法不论是对代码审计和修改脚本都有很大的帮助和提升。我们在实战中会碰到各种各样的问题，这时候需要自己细心耐心以及编码修改能力来解决这些问题。这样我们才可以做到在这不断提升的攻防中稳步前行。

[![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s55IG2v03IEtCG9PygicozPdEiaubAZqKmrmjNQT5ozPnDI8lPOuEibzWOno66etfJBrteNfdnAuCorDQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA==&mid=2247488442&idx=2&sn=56bbdbf3f35b3781aabd65c0d4b2f75b&chksm=9b39350bac4ebc1d010cd744eec558c5562883dafe376e13c8c0b8b4f2fbbba0d81639d107d6&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s55IG2v03IEtCG9PygicozPdEuyFfIGjEibJjlf5eQCL56BefGSXvMLWeGXskC4tSvfuw7VjlHiciciaLmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibiaRBNn0A5YI88OyFjU8fn2Isf9bat4vQn18NwG6cXxVOSuKiapNm2nibQ/640?wx_fmt=png)