> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.anquanke.com](https://www.anquanke.com/post/id/225027)

[![](https://p2.ssl.qhimg.com/t01507eb625b357516d.jpg)](https://p2.ssl.qhimg.com/t01507eb625b357516d.jpg)

背景：
---

基于现阶段红蓝对抗强度的提升，诸如 WAF 动态防御、态势感知、IDS 恶意流量分析监测、文件多维特征监测、日志监测等手段，能够及时有效地检测、告警甚至阻断针对传统通过文件上传落地的 Webshell 或需以文件形式持续驻留目标服务器的恶意后门。

结合当下形势，这次我们着重关注在 Tomcat 日志处理流程中，是否有可取巧的地方使得 Tomcat 不记录访问记录，进而达到隐匿行踪的效果。

声明 ：

由于传播或利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，此文仅作交流学习用途。

历史文章：

[Tomcat 容器攻防笔记之 Filter 内存马](https://mp.weixin.qq.com/s/nPAje2-cqdeSzNj4kD2Zgw)  
[Tomcat 容器攻防笔记之 Servlet 内存马](https://mp.weixin.qq.com/s/rEjBeLd8qi0_t_Et37rAng)  
[Tomcat 容器攻防笔记之 JSP 金蝉脱壳](https://www.anquanke.com/post/id/224698)

一、Tomcat 的日志类型
--------------

基于默认配置启动的 Tomcat 会在 logs 目录中产生以下五类日志：catalina、localhost、localhost_access、host-manager、manager

*   catalina：记录了 Catalina 引擎的日志文件
*   localhost：记录了 Tomcat 内部代码抛出的日志
*   localhost_access: 记录了 Tomcat 的访问日志
*   host-manager 以及 manager：记录的是 Tomcat 的 webapps 目录下 manager 的应用日志

既然是跟隐藏访问记录有关，本次对 localhost_access 日志的调用逻辑和调用流程进行调试学习

二、Tomcat 记录访问日志的流程细节？
---------------------

Tomcat 是对客户端的请求完成响应后，再进行访问日志记录的。具体实现在 CoyoteAdapter#service 方法，下图第二个红框处。

[![](https://p3.ssl.qhimg.com/t0126b3ccb74fce47b9.png)](https://p3.ssl.qhimg.com/t0126b3ccb74fce47b9.png)

此处的 Context 变量其实是 StandardContext，Host 变量是 StandardHost。然而，无论是 StandardHost 类还是 StandardContext 类，这两个容器实现类都继承于 ContainerBase 类。

[![](https://p4.ssl.qhimg.com/t015d9ba75a46a486f8.png)](https://p4.ssl.qhimg.com/t015d9ba75a46a486f8.png)

由于这两个子类，并没有重写自己的 logAccess 方法，因此这里调用的 logAccess(request, response, time ,false) 方法，其实是调用其父类 ContainerBase 的 logAccess 方法。

[![](https://p4.ssl.qhimg.com/t019391a824468c8fd8.png)](https://p4.ssl.qhimg.com/t019391a824468c8fd8.png)

代码逻辑很清晰，稍微说明一下调用顺序，Tomcat 组件的日志记录是逐层回溯，从下往上调用的。

首先，从 CoyoteAdapter#service() 方法中，先由调用 StandardContext 实例的 logAccess 方法，所以上图的 this 第一次指代的是 StandardContext 自身，通过 getAccessLog 方法，获取 StandardContext 的日志记录对象。再调用 log() 方法，记录 request、reponse、time 中的信息。

那么当 StandardContext 调用完成日志记录后，进入下一个 if 逻辑。

通过 StandContext.getParent 方法，获取上级容器实现类 StandardHost。如果有朋友好奇为什么是 StandardHost 的话，可以先了解一下 Tomcat 的 Container 架构，也可以阅读先前编写的文章。

当获取了上级容器实例后，再次调用 logAccess 方法，其实进入的是上图方法本身，直到达到最上级容器：this.getParent() == null 成立。

现在，我们了解了 Tomcat 调用日志记录的顺序，具体来看看细节。

[![](https://p4.ssl.qhimg.com/t0111276b8bdd13dd33.png)](https://p4.ssl.qhimg.com/t0111276b8bdd13dd33.png)

在 Tomcat 的 / conf/server.xml 的默认配置中，只存在 localhost_access_log.txt 用于记录请求的 IP 地址、时间、请求方式、URI、协议等信息。其中 pattern 字段决定日志的记录形式和记录内容。

对 pattern 字段内容感兴趣可查阅该官方链接：

[https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Access_Logging](https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Access_Logging)

该配置嵌于 Host 标签内，属于 StandardHost 类。可见默认情况，仅有 StandardHost 调用 getAccessLog 方法时返回日志记录对象。

[![](https://p0.ssl.qhimg.com/t017bb4cbc38c3d1c14.png)](https://p0.ssl.qhimg.com/t017bb4cbc38c3d1c14.png)

首先，this.accessLogScanComplete 判断是否已完成配置文件中日志记录配置的扫描加载，如果扫描完成，则返回 accessLog 对象。若未扫描，则进行扫描加载。

在 Tomcat 的容器中，都有一个管道 (PipeLine) 及若干个阀（Valve），它们是容器类必须具备的模块，在容器对象生成时自动产生，Pipeline 就像是每个容器的逻辑总线，在 Pipeline 上按照配置的顺序，加载各个 valve 并逐个调用，各个 valve 实现具体的功能逻辑。

而配置文件中的 “org.apache.catalina.valves.AccessLogValve”，就是一个阀，实现日志记录的功能逻辑。

this.getPipeLine().getVales() 方法获取当前管道中所有阀。通过 else 中的方法体，我们也可以了解到，可以自行编写继承于 ValveBase 类的阀，用于实现我们想要的功能，这里会通过判断阀的类型是否为 AccessLog 类型，获取管道中所有有关日志记录的阀实例，并保存到 accessLog 中，最后返回。

[![](https://p2.ssl.qhimg.com/t01aaddafe49818417a.png)](https://p2.ssl.qhimg.com/t01aaddafe49818417a.png)

当 this.getAccessLog() 的返回值 accesssLog 不为空时，是调用 log 方法实现日志记录。

[![](https://p3.ssl.qhimg.com/t018eaeb61ab9173d55.png)](https://p3.ssl.qhimg.com/t018eaeb61ab9173d55.png)

此处的 this 为 accessLog 自身，accessLog 的类为 AccessLogAdapter，真正的日记记录实现类，是其成员变量 logs 中的 AccessLogValve，见下图。

[![](https://p1.ssl.qhimg.com/t0137fd81f538fc7739.png)](https://p1.ssl.qhimg.com/t0137fd81f538fc7739.png)

由于 AccessLogValve 并没有实现自己的 log 方法，在 AccessLogAdapter#log 中的 log.log(request, response, time), 调用的，其实是 AccessLogValve 的父类 AbstractAccessLogValve 的 log 方法。（org.apache.catalina.valves.AbstractAccessLogValve）

[![](https://p2.ssl.qhimg.com/t01c070fbbef44a0b30.png)](https://p2.ssl.qhimg.com/t01c070fbbef44a0b30.png)

在 AbstractAccessLogValve#log 方法中，满足逻辑条件，则最终记录日志信息。要想隐藏访问，避免记录入日志中，就要令这个 log 方法逻辑条件不成立。

在第一个 IF 条件中，改动前三个条件，会令该日志记录实现类失效，进而影响了正常功能，不建议改动。但后续 2 个条件，跟具体的 request 有关，并且是 “或” 判断，意味着，单独更改该类的成员属性 condition 和 conditionIf 不影响该类正常工作。

于是，我们可以通过改动 this.condition 和 request.getAttribute(this.conditiion), 或者 this.conditionIf 和 request.getAttribute(this.conditiionIf)，令以上任一条件不成立，则第一个 IF 逻辑则无法进入，最终使得 Tomcat 不记录我们的访问记录。

三、实现行踪隐匿
--------

经过前面分析，我们可以知道，日志记录，是在请求完成响应之后实施的。那么我们可以从 Request 中的 MappingData 获取 StandardHost，通过 Standardhost 获取 accessLog。

阅读过先前讲解 Servlet 内存马的朋友可能会好奇为何 StandardService 有 MappingData，为何 Request 也有，MappingData 作为记录映射关系的实例，也会最终传递给 Request 对象供其调用。

[![](https://p2.ssl.qhimg.com/t01e144eb7b8bee599e.png)](https://p2.ssl.qhimg.com/t01e144eb7b8bee599e.png)

因而我们无论是通过 Filter、Servlet 还是 JSP，都拥有了 ServletRequest 对象。

但要注意的是，Tomcat 采用的设计模式是门面模式，为了提高系统的独立性，将 Request 对象转换成了 RequestFacade 对象，转换之后，Request 则不可见，用户操作的对象只能是 RequestFacade。以此，通过门面实现了系统内部和外部操作对象的分离。

但是，因为门面实际上是为复杂的子系统为一个类提供一个简单的接口，对于 RequestFacade 对象而言，实际上完成操作的，仍然是 Request 对象，因而 Request 对象，自然而然会作为成员变量保存在 RequestFacade 对象之中。既然保存在其中，我们就可以通过 Java 的反射机制，越过访问控制权限，动态获取运行中实例的属性。

按照惯例，先把要导入的包说明一下：

```
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.mapper.MappingData" %>
<%@ page import="org.apache.catalina.core.StandardHost" %>
<%@ page import="org.apache.catalina.AccessLog" %>
<%@ page import="org.apache.catalina.valves.AbstractAccessLogValve" %>
<%@ page import="org.apache.catalina.core.AccessLogAdapter" %>
```

获取 Request 对象。

```
Field requestF = request.getClass().getDeclaredField(“request”);
// requestFacade的request由protected修饰
requestF.setAccessible(true);
Request req = (Request) requestF.get(request);
```

获取 MappingData 和 StandardHost：

```
MappingData mappingData = req.getMappingData();
StandardHost standardHost = (StandardHost) mappingData.host;
```

获取 accesslog 并赋值 AccessLogValve.condition 和 Request.attributes :

```
AccessLogAdapter accessLog = (AccessLogAdapter) standardHost.getAccessLog();
    Field logsF = accessLog.getClass().getDeclaredField("logs");
    logsF.setAccessible(true);
    AccessLog[] logs = (AccessLog[]) logsF.get(accessLogAdapter);
    for( AccessLog log:logs ){
        ((AbstractAccessLogValve)log).setCondition("WhatEverYouWant");//任意填入
    }
request.setAttribute("WhatEverYouWant", "WhatEverYouWant");
```

PS：以上代码，可任意嵌入 Filter、Servlet、JSP 中，均可生效。

看看效果：

[![](https://p5.ssl.qhimg.com/t01f01039012325f75f.png)](https://p5.ssl.qhimg.com/t01f01039012325f75f.png)

[![](https://p2.ssl.qhimg.com/t01042c933cf600c4b5.png)](https://p2.ssl.qhimg.com/t01042c933cf600c4b5.png)

[![](https://p0.ssl.qhimg.com/t01fd59bd1dd7326eab.png)](https://p0.ssl.qhimg.com/t01fd59bd1dd7326eab.png)