> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Bmn4w_OGMnC4PFKJX85p8A)

![](https://mmbiz.qpic.cn/mmbiz_gif/ibicicIH182el5PaBkbJ8nfmXVfbQx819qWWENXGA38BxibTAnuZz5ujFRic5ckEltsvWaKVRqOdVO88GrKT6I0NTTQ/640?wx_fmt=gif)

**一****：漏洞描述🐑**

**X 天 高级可持续威胁安全检测系统 存在越权访问漏洞，攻击者可以通过工具修改特定的返回包导致越权后台查看敏感信息**

**二:  漏洞影响🐇**

**X 天 高级可持续威胁安全检测系统**

**三:  漏洞复现🐋**

**登录页面如下**

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el71T1D9H7abWI76W4qH8CmuXMGe3wQ4HB9ClfUJNbp1ibTKX5oQDLGuWWvV6JRibOLPFvrFOian4UxLA/640?wx_fmt=png)

**其中抓包过程中发现请求的一个身份验证 Url**  

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el71T1D9H7abWI76W4qH8CmuURmFgoeJERTcW1gDYI0XDA7OvZgZnibCTuzk2pXRicn5umMUALMuWr6A/640?wx_fmt=png)

```
{"role": "", "login_status": false, "result": "ok"}
```

**其中 **login_status 为 false**, 将参数使用 Burp 替换响应包为 **true****

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el71T1D9H7abWI76W4qH8CmutQJIXZc9VhvJ8DaKh18A1TPAOs0d63vINpkb8VsvpHvRrmWiabQicPmA/640?wx_fmt=png)

**请求 **/api/user/islogin** 时成功越过身份验证**

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el71T1D9H7abWI76W4qH8CmuLExxbVuI7WJDPib0rzPwvNWKC0Ng9Jk2epC39LbJguNriaYCEwyiatIYg/640?wx_fmt=png)

**再次访问首页验证越权漏洞**

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el71T1D9H7abWI76W4qH8Cmu1jlzI618ZZicIXqT0ygK1UsAnH7MZIibxbGACqjHicaOTAk82QWTLOQjw/640?wx_fmt=png)

 ****四:  关于文库🦉****

****（文库暂时关闭一段时间，敏感问题解决后再次开放~）****

**在线文库：**

**http://wiki.peiqi.tech**

**Github：**

**https://github.com/PeiQi0/PeiQi-WIKI-POC**

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

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el7iafXcY0OcGbVuXIcjiaBXZuHPQeSEAhRof2olkAM9ZghicpNv0p8rRbtNCZJL4t82g15Va8iahlCWeg/640?wx_fmt=png)

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**PeiQi 文库 拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。**