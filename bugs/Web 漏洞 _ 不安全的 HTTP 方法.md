> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9b7k20m7cosgzk4ip1nKmw)

我们常见的 HTTP 请求方法是 GET、POST 和 HEAD。但是，其实除了这两个之外，HTTP 还有一些其他的请求方法。

**WebDAV** （Web-based Distributed Authoring and Versioning） 一种基于 HTTP 1.1 协议的通信协议。它扩展了 HTTP 1.1，在 GET、POST、HEAD 等几个 HTTP 标准方法以外添加了一些新的方法，使应用程序可对 Web Server 直接读写，并支持写文件锁定 (Locking) 及解锁(Unlock)，还可以支持文件的版本控制。

WebDAV 虽然方便了网站管理员对网站的管理，但是也带来了新的安全风险！

*   PUT：由于 PUT 方法自身不带验证机制，利用 PUT 方法可以向服务器上传文件，所以恶意攻击者可以上传木马等恶意文件。
    
*   DELETE：利用 DELETE 方法可以删除服务器上特定的资源文件，造成恶意攻击。
    
*   OPTIONS：将会造成服务器信息暴露，如中间件版本、支持的 HTTP 方法等。
    
*   TRACE：可以回显服务器收到的请求，主要用于测试或诊断，一般都会存在反射型跨站漏洞
    

以下是 WebDAV 支持的 HTTP 请求方法。

```
方法      描述
GET       Get长度限制为1024，特别快，不安全，在URL里可见，URL提交参数以？分隔，多个参数用&连接，请求指定的页面信息，并返回实体主体。
HEAD      类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
POST      长度一般无限制，由中间件限制，较慢，安全，URL里不可见。请求的参数在数据包的请求body中
PUT       向指定资源位置上传其最新内容
DELETE    请求服务器删除指定的页面。
CONNECT   HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。
OPTIONS   返回服务器针对特定资源所支持的HTTP请求方法。也可以利用向Web服务器发送'*'的请求来测试服务器的功能性。
TRACE     回显服务器收到的请求，主要用于测试或诊断。
```

  

我们可以将请求方法设置为 OPTIONS，来查看服务器支持的请求方法。

  

如下，我们查看一下 CSDN 支持的请求方法，从返回包我们可以看出支持 GET、POST、OPTIONS。这是安全的。

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2ec69p01oPK9icILxowmiaEe35iay8rkb9WjqR9mKg1fibnqoh8hWDrj8xe3vdgWZ2ZGJF24Vic9caAjgA/640?wx_fmt=png)

  

但是有些网站开启了 WebDAV，并且管理员配置不当，导致支持危险的 HTTP 方法，如下。该网站除了支持 GET、POST、OPTIONS、HEAD 之外，还支持 PUT、DELETE 请求方法。

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2ec69p01oPK9icILxowmiaEe3bw2KgH8aImcj4w3ISSSanlwGXVIu2e6IxhokFM6Ue3u3srvJGkV8UQ/640?wx_fmt=png)

  

**风险等级**：低风险 (具体风险视通过不安全的 HTTP 请求能获得哪些信息)

  

**修订建议：**如果服务器不需要支持 WebDAV，请务必禁用它，或禁止不必要的 HTTP 方法，只留下 GET、POST 方法！

  

* * *

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2eV9YtXGl85IabPhGezlOtib7CpNtFKkysI4DshBjazicCick4euYeSHmWd4MxkTbbBiaib4ukpc2rLKjQ/640?wx_fmt=gif)

  

来源：谢公子的博客

责编：Zuo

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640?wx_fmt=png)

如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)