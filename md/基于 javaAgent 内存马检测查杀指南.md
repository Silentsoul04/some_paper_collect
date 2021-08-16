> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6ZUjvAFvshA1WmYXlSPuHA)

新版冰蝎的内存马使用基于 javaagnet 技术实现，在昨天的文章中，我只谈了怎么还原被修改的内存马。但是在检测这块，我并没有详细说明，其实这块我也不太清楚。不过经过我昨日连夜分析 javaagent 相关技术资料，找到一种检测 javaagent 内存马比较通用的方法。分享给大家，希望让 hw 蓝队的小同学做应急响应的时候，不被甲方问倒。当然，清除 javaagent 内存马是一个比较有挑战的任务，如果您感觉在不重启目标服务器的情况下很难完成该项任务，请与我们联系。

1 基础
----

我们先介绍以下什么是 java agent，也就是 JVM Instrumentation 。

> JDK™5.0 中引入包`java.lang.instrument`。 该包提供了一个 Java 编程 API，可以用来开发增强 Java 应用程序的工具，例如监视它们或收集性能信息。使用  Instrumentation，开发者可以构建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM  上的程序，甚至能够替换和修改某些类的定义。有了这样的功能，开发者就可以实现更为灵活的运行时虚拟机监控和 Java  类操作了，这样的特性实际上提供了一种虚拟机级别支持的 AOP 实现方式，使得开发者无需对 JDK 做任何升级和改动，就可以实现某些 AOP  的功能了。

简单来讲，该项技术赋予我们可以动态修改 JVM 中已加载类的字节码的能力而不需要重启 JVM。内存马正是利用这点，修改中间件的处理逻辑。

通过 JVM Instrumentation 去修改一个已经加载的类的字节码，我们可以认为需要两个关键 API

*   retransformClasses(Class<?>… classes) throws UnmodifiableClassException;
    
*   void redefineClasses(ClassDefinition… definitions) throws ClassNotFoundException, UnmodifiableClassException;
    

这两个函数的主要区别，用人话来讲

retransform class 可以简单理解为回滚操作，具体回滚到哪个版本，这个需要看情况而定，下面不管那种情况都有一个前提，那就是 javaagent 已经要求要有 retransform 的能力了：

如果类是在第一次加载的的时候就做了 transform，那么做 retransform 的时候会将代码回滚到 transform 之后的代码 如果类是在第一次加载的的时候没有任何变化，那么做 retransform 的时候会将代码回滚到最原始的类文件里的字节码 如果类已经加载了，期间类可能做过多次 redefine(比如被另外一个 agent 做过)，但是接下来加载一个新的 agent 要求有 retransform 的能力了，然后对类做 redefine 的动作，那么 retransform 的时候会将代码回滚到上一个 agent 最后一次做 redefine 后的字节码。

redefineClasses 对参数代码的类进行重新定义。针对的是已经加载的类。

2 检测
----

冰蝎内存马通过修改`javax.servlet.http.HttpServlet#service`方法，添加自己的处理逻辑，也就是内存马。

我们整理一下冰蝎的添加流程

1.  通过 javaassist 获取`javax.servlet.http.HttpServlet`类的字节码
    
2.  向 service 方法添加字节码
    
3.  清除 javaassist 缓存，调用 redefineClass 重新定义修改后的 HttpServlet 字节码
    

这样，http 中间件在处理每个 http 链接的时候，就会调用修改后的 httpservlet 方法。如果发现处理的 url 为内存马需要响应的 url，则执行 webshell 处理流程，否则隐藏不执行任何操作。

冰蝎在添加完内存马后，会将已经落地的 javaagent 文件删除。这也给我们检测代码了很大的挑战。

添加完内存马后在控制台打印的内容![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYPGj6MfDAbcn0ovv2LWTcgSJt40L8BrFIlDtfLR1WUfjhibRSZsicEcudw/640?wx_fmt=png)

在 JVM 中，同一个类是允许被 javaagent 动态修改多次的。而且在 java 中，在 retransform 方法被调用的时候，jvm 会将类的字节码作为参数传递给用户的 Transform 转换类。

在这里有一个大坑，也就是在调用 retransformClass 方法的时候参数中的字节码并不是调用 redefineClass 后被修改的类的字节码。对于冰蝎来讲，我们根本无法获取被冰鞋修改后类的字节码，这一点才是冰蝎最骚的地方。

我们自己写 javaagent 清除内存马的时候，同样也是无法获取到被 redefineClass 修改后的字节码，只能获取到被 retransformClass 修改后的字节码。通过 javaassist 等 asm 工具获取到类的字节码，也只是读取磁盘上响应类的字节码，而不是 jvm 中的字节码。这也就是我说清除容易，检测难的缘由。

在应急响应的时候，客户需要我们有确切的证据去证明服务器被注入内存马。没有证据证明服务器被植入内存马，客户难道允许我们直接运行清除内存马的工具？

所以这个检测的重点在于，怎么安全地获取到 JVM 运行的类的字节码以供我们分析。(在这里我们只谈 oracle jvm，其他 jvm 暂且不谈，有需要的同学后台联系我)

幸好我发现了`sa-jdi.jar`这个 jvm 提供的小工具。

> sa-jdi.jar  Hotspot 的 Serviceability Agent  是个很强大的 JVM 监控工具集合 (这里简称 SA), 我们可以通过 SA 提供的 Java API（sa-jdi.jar）做事情。

使用方法很简单，打开，输入 jvm 的进程，点击菜单栏的 tools-class broswer 查看当前 jvm 中已经加载并被 java Instrumentation 修改后的类。然后搜索你认为可能被植入内存马的类

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYPWSkoStsk4kAHE5S8nsTAOibSlYwrJdbq71LvIUfIEibfIOp1ZxZIjAkw/640?wx_fmt=png)

点击你关心的方法，比如 service 方法，就可以查看该方法的字节码。

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYPvBGeltXGThZNFicknUwLv6tx946XWFXUK1V0PkXxm7V7Xe151n91kQA/640?wx_fmt=png)

如图是冰蝎的内存马，修改后的类的字节码。我们可以发现可读性很差，别急，我们可以使用 https://github.com/hengyunabc/dumpclass 工具来 dump 类至指定的文件夹。运行如图

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYPGH0ticoLlwGtyXNDXA90pic8pIYUll1Wb7HW3XcCg1eLU7TYwkJudtiaw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYPPE0ibI0jO0vamjzTlyzJJAZ0DPYaU3H1rk9icibUCeHlSYn7sJPNnScdw/640?wx_fmt=png)

现在可以做真正地溯源工作了。

3 清除
----

清楚工作是最简单的，在前面我们也说过，通过 javaassist 就可以很方便地获取磁盘上未经内存马修改的类的字节码。通过 retransformClass 方法重新定义类即可。

我们首先需要编写一个 javaagent，然后注入到目标中间件即可。在这里不分中间件类型，只需要知道被修改的类名即可

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYPlicDswzg6vDAbjy6UjAAPgZH1cCN9dLibqv4ic9MicL7PT0gXnUAMo0RzQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYP0OUudPAKMia315JQPTPOgMyGLq1QnpBPpehGePepBByiagPLE5F06dUw/640?wx_fmt=png)

#### 3.1 周瑜内存马的清理

项目地址。https://github.com/threedr3am/ZhouYu

该类内存马的特征在于阻止后续 javaagent 加载的方式，防止 webshell 被查杀。我们来看一下代码

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdHibl8aNLNVLcMG1A5Kb7cYPPborjKOwbMB5DRcoiavMFdogOtXfFVFRLRcBibyQL44FYtibq2YWPoBYQ/640?wx_fmt=png)

在这里其实不影响随后 javaagent 加载的。原因在于，javaagent 修改类的字节码的关键在于用户需要编写继承自`java.lang.instrument.ClassFileTransformer`，去完成修改字节码的工作。而周瑜内存马的方法在于，如果发现某个类继承自`ClassFileTransformer`，则将其字节码修改为空。但是在这里并不会影响 JVM 加载一个新的 javaagent。周瑜内存马该功能只会破坏 rasp 的正常工作。

周瑜内存马正常通过 javaagent 加载并查杀即可，不会受到任何影响的。或者，我们也可以通过 redefineClass 的方法去修改类的字节码。

相关工具稍后上传 GitHub，后台回复内存马清理即可获取链接

> 注意，检测方法不会影响线上业务。向线上业务加载 javaagent 有可能会直接导致目标 JVM 出现异常而导致服务宕机，请慎重实用工具

写了这么多，难道你不应该加我的知识星球来支持我一下吗，你的支持就是我最大的动力

我正在「宽字节安全」和朋友们讨论有趣的话题，你⼀起来吧？https://t.zsxq.com/qJe2JEi

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGgXGuibZ56sAeSjVFPyWEw25uaZEmwaGKmltLREfSVu5J7C9y8q7qg7GoGW5iapmeHKPoFY74Ha1fA/640?wx_fmt=png)