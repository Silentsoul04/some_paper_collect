> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9eDuJdYJMSNGVanZPt6dNQ)

最近遇到个挺恼火的事，刚通过某漏洞拿下的 weblogic 服务器被应急响应的小同学给下线了。于是最近在思考，怎么才能更好地去隐藏我们的内存马，让应急响应的小同学无法查杀。为什么不用 javaagent？第一，麻烦，适配难度高。第二，很容易被应急响应查杀，只需要看一下 jvm 的启动参数即可知道是否存在 javaagent 的后门。

为啥公开了，因为我手里还有很多种方案的，包括 tomcat，weblogic 等等，发出来大家一起学习。大家努力，让应急响应的小同学今年全翻车。

看完各大安全厂商发布的蓝队锦囊就感觉挺扯淡，攻防本质在于人对于底层的理解，而不是夸夸其谈。

至于怎么查杀，琢磨琢磨，捉摸不透的话私信

1. startUpClass
---------------

简单来讲，就是 weblogic 在启动后，加载 webapp 之前，会根据你指定的 startUpClass 参数，去执行你指定的类。jar 包位置只要在 classpath 中即可。当然，不懂的同学，可以去 youtube 上看一下印度老哥的一篇介绍，咖喱味口语还是很好玩的。https://www.youtube.com/watch?v=u2O9PuofoUI

在我测试后发现，weblogic 在执行到我们的 startUpClass 的时候，各种 runtime 已经建立好了。也就是说，我们完全可以在 startUpClass 中植入基于 filter 的内存马，做到持久化方案。

所以，本小节的重点在于：

1.  怎么找到所有 context
    
2.  怎么植入内存马
    

### 1.1 找到所有 context

参考上一篇文章，在这里我就发一下代码了

```
java.lang.reflect.Method m = Class.forName("weblogic.t3.srvr.ServerRuntime").getDeclaredMethod("theOne");        m.setAccessible(true);        ServerRuntime serverRuntime = (ServerRuntime) m.invoke(null);        List<WebAppServletContext> list = new java.util.ArrayList();        for (weblogic.management.runtime.ApplicationRuntimeMBean applicationRuntime : serverRuntime.getApplicationRuntimes()) {            java.lang.reflect.Field childrenF = applicationRuntime.getClass().getSuperclass().getDeclaredField("children");            childrenF.setAccessible(true);            java.util.HashSet set = (java.util.HashSet) childrenF.get(applicationRuntime);            for (Object key : set) {                if (key.getClass().getName().equals("weblogic.servlet.internal.WebAppRuntimeMBeanImpl")) {                    Field contextF = key.getClass().getDeclaredField("context");                    contextF.setAccessible(true);                    WebAppServletContext context = (WebAppServletContext) contextF.get(key);                    list.add(context);                }            }        }        return list;
```

### 1.2 注册内存马

在这里我是用另外一种方法，通过 javaassist 去组装一个内存马的 filter 类。为什么使用 javaassist，主要基于以下几点考虑

1.  相对于 javaasm，javaassist 简单易懂容易学习，编写的代码可读性非常高
    
2.  相对于通过 classloader 直接加载类，javaasist 可以做到可读性十分好，后期修改方便
    

javaassist 组装类的代码如下，组装完成后，调用`context.registerFilter`将我们的内存马注入到相应的 web 应用中。

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdFKyyBJ60V7VXF0Kqhuye1iaNe3cQzVIOgqJl6DGoibxVRD9mVGCvPuhKW0kpicR5libHaFI16JI4bKiaw/640?wx_fmt=png)image-20210402154333144![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdFKyyBJ60V7VXF0Kqhuye1iafFmD8yfe2UvzX3231OAjyMg6xNxJiaDolK90Kuu2eWeDecbrKADMNDg/640?wx_fmt=png)image-20210402154350734

2. 如何添加修改 startUpClass？
-----------------------

这个功能原本在 console 控制台中，需要登陆才可以操作。但是既然我们已经有了 weblogic 服务器的权限，我们怎么可以跳过登录，直接操作修改某些参数呢？

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdFKyyBJ60V7VXF0Kqhuye1iaib6lYjZaYys4lxlXChiaTM1MXwj1ph0mRpjhzXMCXrLBsXPBvSVia1Gmw/640?wx_fmt=png)image-20210402154719355

我们来大致跟踪一下这块代码，在这里我以 weblogic 12.2.1.4 为例

这一块的处理逻辑在`com.bea.console.actions.core.classes.createclassdeployment.CreateClassDeployment#finish`中

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdFKyyBJ60V7VXF0Kqhuye1ia2A5iaHur6YvBf9gh2GQtAGYS3ez320P9H5FQkvEhNIksjzkQoTVLl9Q/640?wx_fmt=png)image-20210402154842674

获取 DomainBean，然后操作就可以了。但是，如果你根据图中函数的流程获取 domainBean，你大概率是无法操作的。原因有以下几点

1.  获取到的 domainBean 是动态代理的，很多方法都不支持调用
    
2.  不允许直接调用 createStartupClass，提示无权限
    

又回到上面那段万能的获取 context 的源码了，稍微改造一下，即可获取未经动态代理的 domainBean，让我们间接实现某些 console 控制台功能。再也不担心管理员删除 console 后无法操作某些功能了。这块代码大家用心去体会

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdFKyyBJ60V7VXF0Kqhuye1iajZUM0WClVIqAp7ZjrGSPmEsSPuRwXFfE3PqeFWPcUbEDZEiajFWgK9A/640?wx_fmt=png)image-20210402155629393

然后我们模拟创建 startUpClass 的流程，即可完成通过代码创建的工作。

3. 查杀
-----

这玩意肯定是将某些配置写入到配置文件了，自己琢磨一下，毕竟是蓝军反治。

4. 效果
-----

在这里我向所有的 web 应用都注入内存马了，防止失连掉线。效果如图

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdFKyyBJ60V7VXF0Kqhuye1ia48tsd1ibKCMyQWy5G06r6XQFBkCQh5jPFLZQBUnYKeztk9PvAsxIz6Q/640?wx_fmt=png)image-20210402160118153

工具在这里我就不放了，有兴趣的同学加入我们团队的知识星球来获取工具的下载

我正在「宽字节安全」和朋友们讨论有趣的话题，你⼀起来吧？https://t.zsxq.com/qJe2JEi

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGgXGuibZ56sAeSjVFPyWEw25uaZEmwaGKmltLREfSVu5J7C9y8q7qg7GoGW5iapmeHKPoFY74Ha1fA/640?wx_fmt=png)