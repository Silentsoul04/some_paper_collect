> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ARCZIR2C40KSu8SjLMYHSw)

![](https://mmbiz.qpic.cn/mmbiz_gif/ibicicIH182el5PaBkbJ8nfmXVfbQx819qWWENXGA38BxibTAnuZz5ujFRic5ckEltsvWaKVRqOdVO88GrKT6I0NTTQ/640?wx_fmt=gif)

**一****：漏洞描述🐑**

**飞鱼星 家用智能路由存在权限绕过，通过 Drop 特定的请求包访问未授权的管理员页面**

**二:  漏洞影响🐇**

**飞鱼星 家用智能路由**

**三:  漏洞复现🐋**

```
FOFA: title="飞鱼星家用智能路由"
```

**登录页面如下**  

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el4WtnXiaQtWgfvq4DhHUTibj484RcaUOxKDvlibmk1VqexnNicBUBPQliakPFJEB74IhKzFicTANOWE7DVw/640?wx_fmt=png)

**访问 index.html 时会请求 cookie.cgi**

```
http://xxx.xxx.xxx.xxx/index.html
```

**页面抓包 Drop 掉 cookie.cgi**

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el4WtnXiaQtWgfvq4DhHUTibj4prNV6Vj4RhT5r2qfSKmRPFWArUibAsNsvXiaOXr6yPdo2oWtUWT9rqtg/640?wx_fmt=png)

****跳转后台获取了权限****

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el4WtnXiaQtWgfvq4DhHUTibj4kOIF82AAjs8UMJfzagxxL6PA0jTyp1uXJdReyicpeQFHDxeGeePsGzg/640?wx_fmt=png)

```
其中很多产品都存在请求 cookie.cgi，同样的方法可以绕过  
```

 ****四:  关于文库🦉****

**在线文库：**

**http://wiki.peiqi.tech**

**Github：**

**https://github.com/PeiQi0/PeiQi-WIKI-POC**

**（文库暂时关闭一段时间，敏感问题解决后再次开放~）**

最后
--

> 下面就是文库的公众号啦，更新的文章都会在第一时间推送在交流群和公众号
> 
> 想要加入交流群的师傅公众号点击交流群加我拉你啦~
> 
> 别忘了 Github 下载完给个小星星⭐

公众号

**同时知识星球也开放运营啦，希望师傅们支持支持啦🐟**

**知识星球里会持续发布一些漏洞公开信息和技术文章~**

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el4WtnXiaQtWgfvq4DhHUTibj4kdCIpibz3T8kWS3Tt3RJWPGnvRI4fWu3xSSMIruSyl76vbyXTWDM4icA/640?wx_fmt=png)

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**PeiQi 文库 拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。**