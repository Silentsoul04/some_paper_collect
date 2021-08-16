> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_N0r_VAKfBia9G7t5vZtkA)

**高质量的安全文章，安全 offer 面试经验分享**

**尽在 # 掌控安全 EDU #**

**作者：掌控安全学院核心成员 - ho****lic**

0x01. 前言
========

时隔应该快一年了吧，具体 ghost 这个漏洞出来我也忘记了，由于我最近无聊，然后想起我使用的 tomcat 有没有漏洞，  

 于是我就来试了试，顺便分析一下这段已经时隔许久的漏洞，依稀记得上次的文章是简单的复现~~ 反正是闲的无聊

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAyIeG3Yruw2v8TebUVdcI75AH38TKuIwXiciawHZU77CQcdW90UiakfVqQ/640?wx_fmt=png)

0x02. 环境部署
==========

idea2020.2 + tomcat7.0.99+jdk1.8  

具体参考下面的文章，idea 导入 Tomcat 源码：  
**环境与 exp 打包好，在附件中~**

**后台回复：“附件” 即可**

https://blog.csdn.net/u013268035/article/details/81349341

https://www.cnblogs.com/r00tuser/p/12343153.html

0x03. 漏洞的基础
===========

Tomcat 部署的时候会有两个重要的文件  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA8fPNL1BnU0ydUYzmF4xqcZFdicEoYL5Rkj39eRb2tRFKZZG2cot3jIg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAp5MKE5QM5u7ibgnz1TgD3lOpGVREpicn88c5s825mY4twHwvJ9etc2Hw/640?wx_fmt=png)

而 Tomcat 在 `server.xml`中配置了两种连接器。

其中包含的是 AJP Connector 和 HTTP Connector

AJP Connector 说明启用了 8080 和 8009 端口，这时候我们可以看一下

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA1ibpYXBQY0xltZF1tZO58f0uoXTXk7wKQnjiaOibJgib4Nxx3wkiauZGNhQ/640?wx_fmt=png)  

而相对于，8080 我们是可以访问的

  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAxLjxrgWA8Fp4l56C8yXiaDJP9hwEO3L4tiaykfyPT0S24PYO1CnXmFUg/640?wx_fmt=png)

也就是说，8080 是负责 Http 协议，8009 负责接收 ajp 协议；

**提问 1：AJP 协议和 HTTP 协议有什么区别吗？**

**HTTP 协议：**负责接收建立 HTTP 数据包，成为一个 web 服务器，处理 HTTP 协议的同时还额外可处理`Servlet`和 jsp

**AJP 协议：** 负责和其他的 HTTP 服务器建立连接， 通过 AJP 协议和另一个 web 容器进行交互

Web 用户访问 Tomcat 服务器的两种方式  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA0FVsrZ56ia9icMJzh2naffFicN3rlEforTmNDpVxts31wZnNiciaujURUmQ/640?wx_fmt=png)

而一个 Tomcat 就是一个 server，其中包含多个 service；

而每个 service 由 **Connector、Container**、Jsp 引擎、日志等组件构成，造成漏洞的关键地方是 **Connector、Container**

**Connector** 上面已经说过分别是 **AJP Connector** 和 **HTTP Connector** ，是用来接受客户端的请求，请求中的数据包在被 Connector 解析后就会由 Container 处理。这个过程大致如下图：  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAjEDzYFJfmll6ZbXIzoa3W9DekA8BMSmNLqlRicKTbCel6O2o7OUtl1g/640?wx_fmt=png)  
一次请求的处理可以划分为 **Connector** 及 **Container** 进行处理，经历的过程大致如下：

*   一个 TCP/IP 数据包发送到目标服务器，被监听此端口的 Tomcat 获取到。
    
*   处理这个 Socket 网络连接，使用 Processor 解析及包装成 request 和 response 对象，并传递给下一步处理。
    
*   Engine 来处理接下来的动作，匹配虚拟主机 Host、上下文 Context、Mapping Table 中的 servlet。
    
*   Servlet 调用相应的方法（service/doGet/doPost…）进行处理，并将结果逐级返回。
    
    **小结
    
    * * *
    
    **
    ---------------
    

对于使用 HTTP 协议或 AJP 协议进行访问的请求来讲，在解析包装成为 request 和 response 对象之后的流程都是一样的

主要的区别就是对 **socket 流量的处理以及使用 Processor 进行解析的过程的不同**

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaArX0LM0fTxxO6y6icvbWfDKomqU6WMmicBm4HkS3YFzsFDic6iaDQ4icic1eQ/640?wx_fmt=png)

也就是第二步的地方出现问题

**0x04. 源码分析**
==============

通过第三步中，我们知道是 Processor 的解析过程不同，而提供这部分功能的接口，在 `org.apache.coyote.Processor`，主要负责请求的预处理。  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAXeGHtVBqnuFtuscI1RGiblu1OMcXul5fLgG0pmOpXszCCmxTE8elJzg/640?wx_fmt=png)  
而此处 AjpProcessor 则是处理 ajp 协议的请求，并通过它将请求转发给 Adapter，针对不用的协议则具有不同的实现类。

我们从上面 wirshark 抓包可以看出  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA1lxFDKtTU1RLFcKibApEbqfm0GTb4LKrGrgxk5Fy46E6BstValtouWw/640?wx_fmt=png)  
可以看出这边是设置了三个莫名其妙的东西  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAzaq3ia14wvgtPQ9F25quO65iaRibJxdWv8mzvvNrY8VFmMqCLusn0Te4Q/640?wx_fmt=png)

通过 exp 源码可以看到这边是设置了三个域对象的值  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAXUZA6WyJeEzOh9yrAPPYa4RTpQUrg5lvcmIvDpsSAj3SjPGlowg7fg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAOficSYYcA1KyewbPEpSia2ACbyn6TA824k19RyYy12I4N6ibShGVv7BEw/640?wx_fmt=png)  
可以看出，我们这边是执行成功了，由于 wirshark 抓包没有去截图，又关闭了，所以就不截图了

**这边可以假设**：请求参数时是写死，也就是 xxx.jsp 文件，而 jsp 后缀等等原因，然后成功进行了文件包含

**那么我们的 test.txt 是怎么识别的？**

我们继续往下看，来解决种种疑惑  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAhmv2yyZNfyKcfR5vKb6tD8zLdPTiaW79Sz2LIWItKkTtbksqORp7nmg/640?wx_fmt=png)  
知道是这个 **prepareRequest()** 问题，我们 f7 跟进去看看

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAfmecbE6ia2FLgo2lBKTmHJq5leIliaGpzdxKVCohdLJrCkRJjhE1BF6A/640?wx_fmt=png)  
进入了该方法的内部，我们继续跟进查看，这边进行判断，获取到了请求参数为 GET  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAicfCDvl6ib0dxvL12thP1wDsDPmrERfCOaBsIkGItOKHUfwjibia7mlPVA/640?wx_fmt=png)  
首先是一些解析数据包读取字节的操作  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAaMLt14g8Kic8LZDv8UicobZW1e2u2pDooRuhmZ6Je22DqHY2dt2Jz8rw/640?wx_fmt=png)  
while 循环获取，switch 判断  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAIw2C9icOzK81uORNxhyYr8ibYNFib2Bo2bBtp9icFs0c1icHHgq8BUjIKeg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAkcENE8QLgibc6W22zzm1RjNhM6Vnyzibgrxmj4jTAiarF64HAv68tlia0Q/640?wx_fmt=png)

当 **att****ributeCode=1**0 时，则进入第一条分支，继续 f8 进去

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAxrrCiawCjlxdELPZnowRrkVvhKQ1EQ7zcpEduckSbVvp7lTpcZIEIdg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA6FicmSnUW2B7icWKSmCJzX5hjby5NhL1kQ8U0FZnsRIaVC1iaHD7xNl1w/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAI9lw9N6fiaku4grjevnn2wmKC0EAILsnib0p7qD9micxNBwialExPauoTA/640?wx_fmt=png)  
是不是有了有种眼熟的感觉~~~ 其实就是往域对象中存值

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA8ANLDFaktbKyxl3sqhvOclc3kP3lF74JHhTtvdEDomU3lmoEM0GdKQ/640?wx_fmt=png)

由于 if 都不满足，直接进入了最后这个比进的 else 分支

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAV90kac2yqoacr124X7qgotPyEEoTuzSOIhN3UeMk0azNEcI8k1AV9A/640?wx_fmt=png)  
最后进入第二个步骤的最后一小步，f8 继续下一步走，在预处理完了 request headers 之后，在 adapter 里面处理 request，然后调用 Adapter 将请求交给 Container 处理

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA9t7XnJptExIuVetZtskoBOS0hd34PDLc12bFv1DW0YGmhcAYpdlh6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAlnia09Q6aQicbEgN2JYdpyOdvumN2llrN5Xp4PAvs5Qib1cdXtbTBROgw/640?wx_fmt=png)  
进入 service 方法，然后 f8 一直跟进

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAL9yOmVBPxPyqIlCPvEsnWvolsPfxycosnjiaGBdZ068CBichWSlS7iavg/640?wx_fmt=png)

直到跟进此处，请求会发送到对应的 servlet，我们请求的是一个 jsp 文件，根据 tomcat 的默认 web.xml 文件

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAEXFNClslyIhjJ9ZUsdadqj2sz5fwMNgtjgEgCBjXf02JNXTJmqSk0Q/640?wx_fmt=png)

通过上面**假设**，tomcat 默认将 jsp/jspx 结尾的请求交给`org.apache.jasper.servlet.JspServlet`处理，它的`service()`方法如下：

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAwEL3Ae2qWTekrwBZ2IYXiakQu6eiaQFt8TRAoU8rCz0QqSM04z53xIYQ/640?wx_fmt=png)  
而 jspUri 等于 null，满足条件，则进入该 if 条件分支  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAaIujOK3gmMB1ky0f4nNkCAMBusnwYED5ByEAnPqpJV4FOYzCB2ZzzQ/640?wx_fmt=png)

由于 **javax.servlet.include.servlet_path** 可控制，通过 **getAttribute** 去域对象中获取属性名为 **javax.servlet.include.servlet_path** 的值，得到值为 test.txt

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaApED3yRzMeBZu2rEHApZeFOCg0nZbL4f6S41ZTl2sCAgKgPJl4UPDGQ/640?wx_fmt=png)  
此时，**jspUri** 的值为 **/test.txt**  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAl6plQIExiaU43SV5NIM6UO3zMq5tmF3OkM32as1jl8XYDd35aFkQTgw/640?wx_fmt=png)  
**pathInfo** 的值此时为空

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAxyWDCxl6xPefpgfARbagT0GyxO9wHuhe29sdaebz7BtBsSHKU41vGA/640?wx_fmt=png)  
最后一直到下面，传入 serviceJspFile 方法中

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaAibUJibIzlTnQVoHKZkhoOeuVsf6zjq0putXd3RP0o98XrjcxpMtyp4HA/640?wx_fmt=png)

继续跟进，会先判断文件是否存在，如果存在，随后才会初始化 **wrapper**，最后调用 **JspServletWrapper** 的 **service** 方法来解析，从而导致本地文件包含  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcr4ibZ8HBbabGZpfCP6jn7iaA5vxOROgf0q56EAZsQlSsbtUHsT3u034ADkicjyqJI6dQE3Thqx2ruIA/640?wx_fmt=png)

****环境与 exp 打包好，****后台回复：“附件” 即可****
====================================

**参考**
======

https://gitee.com/wdragondragon/javasec/blob/master/tomcat%20ajp%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.md  

https://www.cnblogs.com/r00tuser/p/12343153.html

https://xz.aliyun.com/t/7325#toc-6

https://zhishihezi.net/b/5d644b6f81cbc9e40460fe7eea3c7925#

  

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