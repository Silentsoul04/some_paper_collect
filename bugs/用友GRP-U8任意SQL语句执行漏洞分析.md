\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/FliS-RfLAL0mtnODGyBh1Q)

**![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlVMDtusKZ8CjWnMVoiaxxicASL7LmM9AcDIlsFHDcFBGf93HfztrVaw6g8ZQzQF1rbCGbf7gjHONfEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  

**1、前言**
--------

用友GRP-U8 R10行政事业财务管理软件是用友公司专注于电子政务事业，基于云计算技术所推出的新一代产品，是我国行政事业财务领域专业的财务管理软件。**近日，百度云安全团队监测到有研究人员披露了用友GRP-U8任意SQL语句执行漏洞的POC，并可利用SQL SERVER数据库特性执行系统命令，我们对漏洞进行了复现和分析，发现该漏洞危害严重，请广大用户及时进行升级修复。**

  

**2、环境搭建**
----------

  

通过搜索发现用友GRP-U8存在3个版本，分别为B版、C版、G版，其中B版，C版和G版模块数量、结构不一样，B版和C版是CS结构，G版可以用浏览器登录。我们需要下载G版进行安装。

  

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlU2uQyAVRzSRfqfIaib09n9TxCic8ft0c2Gdj1Paq4IYCCfjic0LzhCKPKXSVCeDNmq83QE2BKe2ZXyg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

安装完成后需要利用自带的工具配置SQL SERVER数据库，并且设置Tomcat的端口等信息并启动。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

安装完成后，目录结构如下，其中webserver为tomcat目录，webapps为Web目录。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**3、漏洞分析**  

  

我们根据网上披露的POC来进一步分析代码，先到webapps/WEB-INF/web.xml中到”Proxy” servlet对应的类为com.anyi.midas.MidasProxy。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

跟进到com.anyi.midas.MidasProxy，POST请求进入doPost方法，并随后调用了Dispatcher的Process方法处理请求。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

跟进到Dispatcher类的Process方法，37行创建了RequestInfo的对象rqi，并在45行调用了rqi对象的processRequestInfo方法来处理请求。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

进入processRequestInfo方法，73行获取请求中ec参数，当ec为空时，加密选项为false；因此，为了方便编写POC，ec参数需要置空，97行调用XMLTools类将dp参数中的xml内容进行解析，方便后续直接获取xml内容中各个参数的值。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

返回到Dispatcher类的Process方法继续往下看，解析xml后可通过rqi.getFunctionName()获取标签<R9FUNCTION>下NAME标签的值作为函数名，并且根据不同的函数名进入不同的条件语句，而POC中的函数名为AS\_DataRequest，理所当然的跟进到63行，接下来程序调用了com.anyi.midas.access.DataModule的as\_DataRequest方法。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

跟进看看as\_DataRequest方法是如何定义的，其中参数providerName来自XML的PARAM标签下ProviderName标签的值，参数data则为Data标签的值，均为外部可控值。64行调用了ProviderFactory类的getProvider方法。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

跟进到ProviderFactory类，POC中ProviderName值为DataSetProviderData，因此返回了QueryProvider类的对象；实际上，经过分析SQLProvider类也可导致任意SQL语句执行，此处ProviderName可为SQLProvider、DataSetProviderData或者置空（为空也会返回SQLProvider对象）。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

跟进到QueryProvider类的dataRequest方法，该方法调用了DBTools类的executeSQL方法。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

跟进executeSQL方法发现，145行调用了rqi对象中java.sql.Connection对象创建SQL连接对象stmt，sqlType默认为”query”进入147行，调用了executeQueryAction\_Buffer方法。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

跟进到executeQueryAction\_Buffer方法，199行判断数据库是否为oracle，GRP-U8默认为SQL SERVER数据库，因此进入else语句后，206行通过stmt对象调用了executeQuery方法执行了可控的sql参数，并且整个过程中没有SQL语句拼接，可导致任意SQL语句执行。

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

而SQL SERVER可通过xp\_cmdshell语句扩展存储过程将命令字符串作为操作系统命令shell执行，**因此该漏洞危害严重，建议用户及时修复。**

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**4、安全产品解决方案**
--------------

  

百度度御关WAF、高级威胁感知系统，以及智能安全一体化产品已支持该漏洞的检测和拦截，有需要的用户可以访问anquan.baidu.com联系我们。

  

_参考链接：_

_https://nosec.org/home/detail/4561.html_

  

**推荐阅读**

**[通达OA 11.5版本某处SQL注入漏洞复现分析](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650494316&idx=1&sn=bdbf4281f9d0b9cedd8659c93a1dd86e&chksm=beb3e32c89c46a3a5691ce5cbb13b4f6b94ed2c3b5e7fbd6e44b2dd633042dfbbabf4b8d08f8&scene=21#wechat_redirect)  
**

**[从CVE-2020-8816聊聊shell参数扩展](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650493205&idx=1&sn=1e3d4ccba11cc2b708f0fdec82d72587&chksm=beb3e75589c46e43271ec0c997d53ffabcdc8596476718d2aff510fc55b6c01040b5c481fe8b&scene=21#wechat_redirect)  
**

**[Shiro rememberMe反序列化攻击检测思路](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650493186&idx=1&sn=39b99df8a3d82534cb9fd255f9e0059d&chksm=beb3e74289c46e5455698fd3c1758b865dbd10fd7b513b3e9f19cce4c5b62d5b0887fcd028b0&scene=21#wechat_redirect)  
**

**[Spring Boot + H2 JNDI注入漏洞复现分析](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650492993&idx=2&sn=0b27352db1d34ba1b077ae397d1f08a5&chksm=beb3e60189c46f17b60ee16fd1460679986064211caaeace1e0f6fbfde2a629b17aa43434fe6&scene=21#wechat_redirect)  
**

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)