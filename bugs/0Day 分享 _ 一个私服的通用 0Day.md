\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/tqL-3Rgs7TqIUBm-BKN55w)

**![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqczeflvHvDexuf2BhBEBYlJCdjJS6aVZ0w6ooY5QwK27L2khaJWEOVdw2kunkBTviakCv6QeGxYjHg/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbYIQLAevTMxqj1m8dokyA5jhA4m3Rkdk5RmKwjrnmmVAp0pDr9zhYVg/640?wx_fmt=png)  

先 admin 尝试  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbRfdoO9H3LZk8NjgXPFaJRFqPec5GyRrJ1RZSUrxxdnOUWGDZwgjcyw/640?wx_fmt=png)

其实当我在浏览网站的最下方时，我看到了这一信息。我的大脑就浮现出了一个 0day，应该泛滥了！但我打算发出来  

**http://url/admin** 尝试一下运气  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbNXCzhUgBTsJygzXcMavOoUvecDGiaYTOZd97URWUS3C8OMY3EX2a3icg/640?wx_fmt=png)

看到这个后台更加确定了我内心的想法，这是有 0day 可以利用的！  

**一个通用的万能账号弱口令：  
**

**账号是 lu123 密码 cui123  
**

我们来尝试一下这个是否正确  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbGzJdMXyeFzIqtEUiawabMHOJEPPb7YX1QgicAvvPA4ygtvnlWaUbibOdA/640?wx_fmt=png)

在这里提供一下 fofa dork：

> body="华宜网络"

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbzI7Ta6xswXW2uZVN6XgXzbp1erceZNicickAIVp6gLELX5TnQJKibqGibQ/640?wx_fmt=png)

目前的话可以自己去尝试挖掘 fofa 关键词，肯定不止这么些站点

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbY9ictJngJffthBib4hvp1Sjkj2LyUnF6E5ukCPNH8tmpx3LAllYC4XPw/640?wx_fmt=png)

至于源码，在官网的话是需要付费购买，我们可以百度找找看有没有公开的

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbmDticwjOQia9bvODVUjoicC2siaPzmSQG9hnbKqSyhIKQiciamhpOLv05nIQ/640?wx_fmt=png)

下载了很多源码，在本地搭建了很多，最终发现这个网站的后台和某宜互联是相同的，可以确定是这套 cms 的源码  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbElF199fNFia3OicUUk5J3e2cs6zib0SicbtgxRuSJVNeENYe6qOBpDoM5A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeqUUGng5pj6t0icJHu4DKibbxP0ZL12Ej0KRichibLto73hLEaaTQAWWDh8KtgCNaQMicYp7TxpLTBz0g/640?wx_fmt=png)

我在 data 目录下找到了 mdb 文件，查看了数据库文件里的密码文件，lu123 是系统自带的一个超级管理用户，和 admin 类似。  

**![](https://mmbiz.qpic.cn/mmbiz_jpg/RpxgdDjibJqffqIwYFgh4VWGevKuZ7puLT0ricCZJGMae7mUqA7E9pcqSDQPnRMiaribQPBMUsicflGmKqbicWrMCicOw/640?wx_fmt=jpeg)**

**可以联系微信，进行技术交流**  

![](https://mmbiz.qpic.cn/mmbiz_jpg/RpxgdDjibJqdBGakDD8I6dJUoPeMTDIPqO2LnCTf7Vib12N86uGSEpIMGI8rp77C4JCSTttEW6pqeJzaZZMiasgDA/640?wx_fmt=jpeg)

**如果想要学习的师傅，可以加入星球噢！会一直更新优质的资源，请相信我们安译 Sec**  
  

**END.**  

**欢迎转发~**

**欢迎关注~**

**欢迎点赞~**