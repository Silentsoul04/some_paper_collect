> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6yAq3HzFKc4qefiL0bE0PA)

**描述**

smart-web2 是一套相对简单的 OA 系统；包含了流程设计器，表单设计器，权限管理，简单报表管理等功能，经过代码审计发现其存在未授权漏洞。

  

  

  

  

  

**影响范围**

  

smart-web2

  

  

  

  

  

**漏洞复现**

  

1、漏洞代码位置

```
cn.com.smart.web.interceptor.ACLInterceptor 
```

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibCKkJGNgw4Xz5Gficpxm4BHx0oDztgO1ibfqdQzB0xPnTPsFX1GbvDn2jzytylpDKoqJ4iciaU7LvLZEA/640?wx_fmt=png)

```
smart-web2.src.main.resources.spring-web-config.xml
```

返回为 True，则会走到 else，走到 else 则不需用户进行身份认证。

```
http://x.x.x.x:8080/sso/../user/list
```

```
excludeMaps来源于：
```

```
smart-web2.src.main.resources.spring-web-config.xml
```

```
继续追踪，内容为：
```

```
我们可以确定如果uri开头为excludeMaps中的内容则不需要认证，例如
```

```
http://x.x.x.x:8080/sso/../user/list
```

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibCKkJGNgw4Xz5Gficpxm4BHxp0lW7uLj4nLjG75nWZ18MyO4VlcJH3ukS575sibeXufHibJAPFYZx1Yw/640?wx_fmt=png)

最后再说一句：

现在的安全圈子歪风邪气越来越大，天天吃瓜，为了防止黑产份子非法利用漏洞，本号相关的 wiki.xypbk.com 站点将在后续进行权限控制，形式应该是账号密码访问限制，公众号留言获取或私聊单独授权，非邀请码注册类，永不割韭菜，永久免费检索，就用来小圈子授权使用了，坚决抵制安全圈的歪风邪气。

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/nMQkaGYuOibDavXvuud5F09Tjl7NMvU8Yzhia63knJ4QJFvO4WBfd6KQazjtuPC7uqNBt5gE06ia7GjOVn2RFOicNA/640?wx_fmt=jpeg)

扫取二维码获取

更多精彩

![](https://mmbiz.qpic.cn/mmbiz_png/TlgiajQKAFPtOYY6tXbF7PrWicaKzENbNF71FLc4vO5nrH2oxBYwErfAHKg2fD520niaCfYbRnPU6teczcpiaH5DKA/640?wx_fmt=png)

Qingy 之安全  

![](https://mmbiz.qpic.cn/mmbiz_png/Y8TRQVNlpCW6icC4vu5Pl5JWXPyWdYvGAyfVstVJJvibaT4gWn3Mc0yqMQtWpmzrxibqciazAr5Yuibwib5wILBINfuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3pKe8enqDsSibzOy1GzZBhppv9xkibfYXeOiaiaA8qRV6QNITSsAebXibwSVQnwRib6a2T4M8Xfn3MTwTv1PNnsWKoaw/640?wx_fmt=png)

点个在看你最好看