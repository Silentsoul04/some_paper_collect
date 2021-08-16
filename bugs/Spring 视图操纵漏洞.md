\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/pJGE7nS2zg-tuz4YPf7Xgw)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**_**No.1  
**_**_

_**概述**_

知识点起源于一个小哥的 GitHub 的 demo，已经在 **R****eference** 提及了，核心观点想要表达如果能操纵 spring 的视图（view）是一件很危险的事情，紧接着用到了 **Thymeleaf** 这个模版来举例子。

_**_**No.2  
**_**_

_**跟踪过程**_

小哥在代码中举了两个例子，这两个例子分别是由相同的特征，返回的内容可被攻击者操纵。

```
//GET /path?lang=en HTTP/1.1
    //GET /path?lang=\_\_$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22id%22).getInputStream()).next()%7d\_\_::.x
    @GetMapping("/path")
    public String path(@RequestParam String lang) {
        return "user/" + lang + "/welcome"; //template path is tainted
    }

    //GET /fragment?section=main
    //GET /fragment?section=\_\_$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22touch%20executed%22).getInputStream()).next()%7d\_\_::.x
    @GetMapping("/fragment")
    public String fragment(@RequestParam String section) {
        return "welcome :: " + section; //fragment is tainted
    }
```

这里先慢慢看，spring 的模版处理在这里 **org.springframework.web.servlet.ViewView# render** ，根据注释可以知道这个地方是个接口，要实现需要到相关模版渲染引擎单中去实现。

```
/\*\*
   \* Render the view given the specified model.
   \* <p>The first step will be preparing the request: In the JSP case, this would mean
   \* setting model objects as request attributes. The second step will be the actual
   \* rendering of the view, for example including the JSP via a RequestDispatcher.
   \* @param model a Map with name Strings as keys and corresponding model
   \* objects as values (Map can also be {@code null} in case of empty model)
   \* @param request current HTTP request
   \* @param response he HTTP response we are building
   \* @throws Exception if rendering failed
   \*/
  void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
      throws Exception;

}
```

而实际上的处理过程是在 **org.springframework.web.servlet.DispatcherServlet.render**

```
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    
  ...

    View view;
    String viewName = mv.getViewName();
    if (viewName != null) {
      // We need to resolve the view name.
      view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
      if (view == null) {
        throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
            "' in servlet with name '" + getServletName() + "'");
      }
    }
    
    ...
      
    try {
      if (mv.getStatus() != null) {
        response.setStatus(mv.getStatus().value());
      }
      view.render(mv.getModelInternal(), request, response);
    }
```

跟一下流程，首先在 String viewName = mv.getViewName(); 的过程中就获取到我们传入的 POC。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLI382HQe3SdyvibTnwGlu5yRc2zeNS1XBgibMeib5M13iaNTKvWBE69bzbw/640?wx_fmt=png)

紧接着进行 view = resolveViewName(viewName, mv.getModelInternal(), locale, request); 处理，这个 **resolveViewName** ，当从英文翻译就知道它大概要**解析视图名字** ，当然本着严谨的角度还是需要看代码的，跟进来之后会来到 **ContentNegotiatingViewResolver# resolveViewName** 当中，关注一下 **getCandidateViews**

```
public View resolveViewName(String viewName, Locale locale) throws Exception {
    RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
    Assert.state(attrs instanceof ServletRequestAttributes, "No current ServletRequestAttributes");
    List<MediaType> requestedMediaTypes = getMediaTypes(((ServletRequestAttributes) attrs).getRequest());
    if (requestedMediaTypes != null) {
      List<View> candidateViews = getCandidateViews(viewName, locale, requestedMediaTypes);
      View bestView = getBestView(candidateViews, requestedMediaTypes, attrs);
      if (bestView != null) {
        return bestView;
      }
    }
```

先跟进来 **getCandidateViews** ，这玩意会循环当前的 **this.viewResolvers** 内容，并且进行处理。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLiaic2aeJ8YAtZXJnGIb6Wu7aTME4C9Ppib3ImsTn0fibOp3KIgYibnp7PHA/640?wx_fmt=png)

当解析到 **ThymeleafViewResolver** 这个方法的时候，会进入 **createView** ，在这个方法中根据几个条件进行判断，比如 redirect: 或者 forward: 等等。因此我们的 **ViewName** 不满足，自然是在 **resolveViewName** 处理之后返回了 **ThymeleafView** 。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLQdFErzNYy9Vib4aHQH3Jibib8hKFJpOVVVPAHRwHD29QtKD1xJgibRqBkw/640?wx_fmt=png)

同理在 **UrlBasedViewResolver# createView** 方法当中也是大概根据 redirect: 或者 forward: 进行判断，因此在 **resolveViewName** 处理之后返回了 **InternalResourceView** 。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLIKSRHqBTFWKT1WqQPOSfEdls8VHLyKbvNIv02Edfylp4nb7mIWZ2iaw/640?wx_fmt=png)

再看 **getBestView**，我们当前的 **candidateViews** 这个 **list** 当中有两个 view 分别是 **InternalResourceView** 和 **ThymeleafView** ，进入处理之后返回的是 **ThymeleafView** 。也就是说经过 view = resolveViewName(viewName, mv.getModelInternal(), locale, request); 处理之后，返回了我们的视图方法是 **ThymeleafView** ，紧接着就是 view.render(mv.getModelInternal(), request, response); 来到相关的视图方法中进行解析了。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLP0lRKDpApw5CpxIO6Makkk96N7LUmAEkq9tpNkNYm8BmGOXmQuOuAQ/640?wx_fmt=png)

当然最后的模版解析过程在 **org.thymeleaf.spring5.expression.SPELVariableExpressionEvaluator# evaluate** 。

```
exec:347, Runtime (java.lang)
...
evaluate:263, SPELVariableExpressionEvaluator (org.thymeleaf.spring5.expression)
executeVariableExpression:166, VariableExpression (org.thymeleaf.standard.expression)
executeSimple:66, SimpleExpression (org.thymeleaf.standard.expression)
execute:109, Expression (org.thymeleaf.standard.expression)
execute:138, Expression (org.thymeleaf.standard.expression)
preprocess:91, StandardExpressionPreprocessor (org.thymeleaf.standard.expression)
parseExpression:120, StandardExpressionParser (org.thymeleaf.standard.expression)
parseExpression:62, StandardExpressionParser (org.thymeleaf.standard.expression)
parseExpression:44, StandardExpressionParser (org.thymeleaf.standard.expression)
renderFragment:278, ThymeleafView (org.thymeleaf.spring5.view)
render:189, ThymeleafView (org.thymeleaf.spring5.view)
```

_**_**No.3  
**_**_

_**区分**_

**1、加上 @ResponseBody 注释**  

```
//GET /fragment?section=main
    //GET /fragment?section=\_\_$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22touch%20executed%22).getInputStream()).next()%7d\_\_::.x
    @GetMapping("/fragment")
    public String fragment(@RequestParam String section) {
        return "welcome :: " + section; //fragment is tainted
    }


@GetMapping("/safe/fragment")
    @ResponseBody
    public String safeFragment(@RequestParam String section) {
        return "welcome :: " + section; //FP, as @ResponseBody annotation tells Spring to process the return values as body, instead of view name
    }
```

首先需要了解 **@ResponseBody** 注释的作用。

```
1、该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；
2、再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。
```

而实际上在 **RequestMappingHandlerAdapter# handleInternal** 返回 view 的对象的时候返回空，找不到模版对象，自然也就没有下面的模版解析 **render** 的渲染过程了。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwL7tLJGqFcoFpEaPMFRNcXOV2zHE8rXicdhvszZ5dJticIBL1hkVVmtgIQ/640?wx_fmt=png)

**2、redirect: 或者 forward: 前缀**

```
@GetMapping("/safe/redirect")
public String redirect(@RequestParam String url) {
    return "redirect:" + url; //FP as redirects are not resolved as expressions
}

@GetMapping("/safe/forward")
public String forward(@RequestParam String url) {
    return "forward:" + url; //FP as redirects are not resolved as expressions
}
```

实际上我们前面也聊过了在 **AbstractCachingViewResolver# resolveViewName** 当中，当调用 view = createView(viewName, locale); 会进入每个模版的 **createView** 方法，而这个方法根据前缀 redirect: 或者 forward: 进行匹配，返回的是 **RedirectView** 或者 **InternalResourceView** ，所以自然不会进行模版解析。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLq991fibpHqiaia0dKUj41CpOeXiczjhibK8nDRTrkyJk3GawTzVJFBhM0qQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwL1PPMcuGSI47UwsuBckuRabqoeO5TF0jhcX009I4w7uviaWDCKKq7I0A/640?wx_fmt=png)

**3、HttpServletResponse 标记**

在构造方法中指定 HttpServletResponse response 的 **org/springframework/web/servlet/DispatcherServlet# doDispatch** 方法之后，实际上就不返回 **modelandView ，**也就是说实际上没有进行模版解析。

```
@GetMapping("/doc/{document}")
    public void getDocument(@PathVariable String document) {
        log.info("Retrieving " + document);
        //returns void, so view name is taken from URI
    }
  
  @GetMapping("/safe/doc/{document}")
    public void getDocument(@PathVariable String document, HttpServletResponse response) {
        log.info("Retrieving " + document); //FP
    }
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLAv0sghN31GQDKBZdqRrTQpLgubZ1uQxYbxweib8NBiaZjnR9tU4N6QTg/640?wx_fmt=png)

_**_**No.4  
**_**_

_**后话**_

那么 **velocity** 和 **freemarker** 有这个问题吗，目前我还没发现，似乎传入进来不会做表达式解析，可以顺着这个思路再看看，总体来说还是挺有趣的。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLBnC8FyNrgC5VQhgC36k26me1LHdUNB9a0oKNRCmkB996AaI9F8xibfg/640?wx_fmt=png)

而 **Thymeleaf** 传入进来之后在 **ThymeleafView# renderFragment** 会针对传入的进行表达式解析。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLjI3HHQO4kgwJjkIaeuFBmeTPHib5fmorXONTQTSkJvqicuUVkpDbRc8g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwLZmZ27EGicbVyIhvOA5O5H95ialrbmG6wwwdMCT1ZDmiaFYWtjWoicDEZ0Q/640?wx_fmt=png)

当然一般开发的时候，好像是不会这么处理视图的，因此这么写的实际面应该不会太大，当然和 **spring** 解析过程有关方法调试断点可以下在 **DispatcherServlet# doDispatch** ，一般都是可以命中的。

_**_**No.5  
**_**_

_**Reference**_

https://github.com/veracode-research/spring-view-manipulation

_**招聘启事**_

安恒雷神众测 SRC 运营（实习生）  
————————  
【职责描述】  
1\.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2\.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3\.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4\.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5\.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

设计师（实习生）

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
3、有品牌传播、产品设计或新媒体视觉工作经历；  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦 19 楼  
学历要求：本科及以上  
工作年限：1 年及以上，条件优秀者可放宽

简历投递至 

bountyteam@dbappsecurity.com.cn

安全招聘  
————————  
公司：安恒信息  
岗位：Web 安全 安全研究员  
部门：战略支援部  
薪资：13-30K  
工作年限：1 年 +  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
【岗位职责】  
1\. 定期面向部门、全公司技术分享;  
2\. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3\. 负责完成部门渗透测试、红蓝对抗业务;  
4\. 负责自动化平台建设  
5\. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1\. 至少 1 年安全领域工作经验；  
2\. 熟悉 HTTP 协议相关技术  
3\. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4\. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5\. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6\. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7\. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8\. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1\. 具备良好的英语文档阅读能力；  
2\. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3\. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4\. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5\. 开发过安全相关的开源项目；  
6\. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7\. 个人技术博客；  
8\. 在优质社区投稿过文章；

岗位：安全红队武器自动化工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1\. 负责红蓝对抗中的武器化落地与研究；  
2\. 平台化建设；  
3\. 安全研究落地。  
【岗位要求】  
1\. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2\. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4\. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5\. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1\. 有高并发 tcp 服务、分布式等相关经验者优先；  
2\. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4\. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5\. 具备良好的英语文档阅读能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

岗位：红队武器化 Golang 开发工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1\. 负责红蓝对抗中的武器化落地与研究；  
2\. 平台化建设；  
3\. 安全研究落地。  
【岗位要求】  
1\. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2\. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3\. 了解 docker，能进行简单的项目部署；  
3\. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4\. 熟悉 TCP/IP 协议的基本运作原理；  
5\. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1\. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2\. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4\. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5\. 具备良好的英语文档阅读能力。  
简历投递至 

bountyteam@dbappsecurity.com.cn

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JVkBq7E0Tkmasfkxw0T0pwL8EXia2ET23jIInniaRfeLkKbAwMWSKHqYMy96usaJG8cPoVSGXTo5utQ/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JVdTmjjQz1QkuJq91GHX8kt6R5ibFR9JBHpEN0pXofqyLu46cnwCt7NsXywrcadGQRNgpTFfKoYa2Q/640?wx_fmt=jpeg)