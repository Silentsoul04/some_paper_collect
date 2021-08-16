> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ahq3hEzF3DsJRFdT0zC_dA)

做了不少qp，BC渗透了，通宵了2个晚上干了几个盘子，简略的说下过程，做一下总结。  
  
首先说一下qp， 以我的渗透成功案例来说的话首先信息收集必不可少的，qp的特点是什么呢？  
  
他的后台会在服务器域名的后面以不同的端口形式架设 如图：

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3CndrC0VBib7ibDr5lLAiaC3wbkBeAKjLcHhRGoThEEIYkZpASiayo2U1KeIx3vxgj9zlAqLJMhOke6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

关于端口可以发现，基础东西你们都懂。  
  
切入点：

  

在app里面抓包，查找邮箱，充值，的地方寻找sql注入或者意见反馈的位置XSS  
  
有一种情况是抓包显示127.0.0.1的 抓不到包的情况，这种情况多于大盘子，它不一定走的是TCP UDP协议。可以参考 T-ice 表哥说的 Proxifier全局代理  
  
有了后台之后可以目录fuzz一下，有些管理员会有备份的习惯没准能有新发现。  
  
相对来说qp还是挺简单的。    
  
那么来说说BC吧，看个昨晚的渗透的案例。  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

基本上大型的BC盘子都是各种防护+cdn 标配，毕竟别人赚了那么多钱也不在乎这点设备钱。。。。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  
注册了个号 发现没地方能打XSS的。。。。。作罢  
  
因为这种大盘子服务一般是挺到位的，牌面这块方方面面给你整的很高大上，什么导航啊，什么积分商城啊。。乱七八糟的应有具有，在他主站一个VIP查询页面确定了一处sql注入，而且是thinkphp的框架

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

thinkphp3.2.3的 ，因为有CDN不知道真实IP，所以后台是个很麻烦的事情，本想着看看数据库里面的log有没有啥发现

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

没啥鸟用。。尝试读取日志文件，没有。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

最后读取配置文件确定了一个很脑残的事情。。。可能通宵了之后人的脑子有点僵。我给忘了这种BC后台肯定都是分离的。。。。嗨。少熬夜。  
  
于是。。我就以以往的经验手动的在主域名前面加上了一些可能的参数。。admin.XXXX.com   agdw.xxxxx.com   ag.xxxxx.com   嗯。。。如图：  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

这套程序的盘子大概100多个吧，几乎都是一模一样的，随便找了几个  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

后台有个地方任意上传。结果。。  
  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

被杀了还是咋回事。。  
  
做个总结：  
  
像这类盘子都是包网的，大多数都是java开发的。那么BC盘子的切入点是哪些呢  
  
以我渗透成功的案例来总结：  
  
1. XSS  
2.注入  
3.历史遗留的资产  
  
主要还是信息收集，和耐心。 其实现在注入还是挺多的，只是很难发现和识别了。 同时还需要和各种防护对抗，有时候其实是个注入只是被防护拦了不确定的情况下很多人就放弃。

  

这种菠菜类的网站 大多数服务器都是防范级别很高的 都是包网 资产很多 而且前后端都是分离的 有的时候 没有思路的时候 可以从运维方面下手 有的运维安全意识不是很高 还可以从C段入手 因为有的菠菜资产分布在几个C段 主站上面 肯定很少漏洞 基本上可以说没有  还有就是从游戏接口入手 或者游戏逻辑入手 个人一点点粗见。

* * *

  

除过上述作者总结的几点，其他常见的切入点包括弱口令、代码审计。

  

作者：极安带头大哥  

来源：https://bbs.secgeeker.net

* * *

**关注公众号:HACK之道**  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

如文章对你有帮助，请支持点下“赞”“在看”