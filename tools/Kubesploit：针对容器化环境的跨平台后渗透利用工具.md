> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NWVZC9YzBxRFc61C8sdpiw)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39zDym7ziaia2miaYIhvEW6PsbDKfk4lRpQmt68YH4zDRtS25JOY7pYsdIG8fIiafz132grLjGWEibpgIQ/640?wx_fmt=jpeg)

关于 Kubesploit
-------------

Kubesploit 是一个功能强大的跨平台后渗透漏洞利用 HTTP/2 命令 & 控制服务器和代理工具，该工具基于 Golang 开发，基于 Merlin 项目实现其功能，主要针对的是容器化环境的安全问题。

我们的主要目标是提高社区对容器化环境安全的认识，并改进各种网络中实施的缓解措施。所有这些都是通过一个框架来实现的，这个框架为 PT 团队和红队成员在这些环境中的活动提供了适当的工具。使用这些工具将帮助你评估这些环境的优势，并进行必要的更改以保护它们。

由于 C&C 和代理基础设施已经由 Merlin 完成，我们只需要继承 Go 解释器（“Yaegi”）就可以在服务器端和代理运行 Golang 代码了。

它允许我们在 Golang 中编写模块，为模块提供更大的灵活性，并动态加载新模块。这是一个正在进行的项目，我们计划在未来添加更多与 Docker 和 Kubernetes 相关的模块。

功能介绍
----

*   支持容器加载；
    
*   支持 sock；
    
*   利用 CVE-2019-5736 漏洞攻击容器；
    
*   扫描 Kubernetes 集群中的已知 CVE 漏洞；
    
*   端口扫描，重点是 Kubernetes 服务；
    
*   从容器内扫描 Kubernetes 服务；
    
*   使用 RCE 扫描容器；
    
*   扫描 Pods 和容器；
    
*   扫描所有可用容器中的令牌；
    
*   使用多个选项运行命令；
    

快速开始
----

我们在 Katacoda 中创建了一个专用的 Kubernetes 环境，可以帮助你使用 Kubesploit 对目标环境进行测试。在下面的演示样例中，我们演示了一整套完整的操作，并告诉大家如何使用 Kubesploit 的自动化功能。

**演示样例：**https://github.com/cyberark/kubesploit/raw/assets/kubesploit_demo_with_progress_bar.gif

项目构建
----

首先，我们需要使用下列命令将该项目源码克隆至本地：

```
git clone https://github.com/cyberark/kubesploit.git

```

然后切换到项目根目录，运行下列命令即可：

```
make

```

### 快速构建

如需在 Linux 系统下对项目进行快速构建，可以直接运行下列命令：

```
export PATH=$PATH:/usr/local/go/bin

go build -o agent cmd/merlinagent/main.go

go build -o server cmd/merlinserver/main.go

```

YARA 规则
-------

我们所创建的 YARA 规则可以帮助广大研究人员获取 Kubesploit 源码，规则编写在 kubesploit.yara 文件中。

代理记录
----

代理中加载的每一个 Go 模块都会在目标设备中被记录下来。

MITRA 地图
--------

我们还为 Kubesploit 中所使用的每一个攻击向量创建了 MITRE 地图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39zDym7ziaia2miaYIhvEW6PsbAIkXibiaTgrZ8ibKPruAKwZ8icD75f6hCVC7gURiaGkicC5IotQ2fVAOItvA/640?wx_fmt=jpeg)

许可证协议
-----

本项目的开发与发布遵循 GPL v3.0 开源许可证协议。

项目地址
----

**Kubesploit：**https://github.com/cyberark/kubesploit

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

  

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

  

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39cpPZAJWAzWjEQtKT2aO8ljsMhAvRdXgZIWNnyfHYhgBDFM7574D04oob78kxeocbSAl9GzNYibzA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651125480&idx=2&sn=1af44d66d0b2e445439e99bff2c2c0db&chksm=bd1f64638a68ed75dbdf9d3f04545f80f68a6d198693361470e9163d112b8f2385ce66583754&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39sqjaLlJOnNYV4AEasAdibTzrH7PyIuE8MbnS21dOWVXNguibdAWFTQSXMxjy2GSJodYHLFhQ1ficDQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486112&idx=1&sn=296cf1bc4e88502ec3c5f73199949135&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39dEsdO2GpOvH87GrfzuscAMuA4JpicWAFbJtfakgMF2hheeTcSSwguAbjO45btx8ws2etnvSJlOzQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486070&idx=1&sn=c6957ca2d1878f316b7947b5ff990a01&chksm=ce1cf0e9f96b79fff5b27a3c146f9e8828728c33625a97366b0cae3df1853dbeda368c59177f&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************