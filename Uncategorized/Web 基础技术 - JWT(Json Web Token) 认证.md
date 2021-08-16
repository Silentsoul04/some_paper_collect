> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/V03jAx1x1WprkhdiMrag3A)

目录

*   ```
    JWT简介
    ```
    
*   ```
    JWT数据结构
    ```
    

*   ```
    JWT头部
    ```
    
*   ```
    JWT有效载荷
    ```
    
*   ```
    JWT签名
    ```
    

*   ```
    JWT用法
    ```
    
*   ```
    JWT验证流程
    ```
    
*   ```
    JWT问题与趋势
    ```
    
*   ```
    JWT安全风险
    ```
    

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2eyg6YoWDSGUEHCfaibpicAoJ7LKMsn651kOMWajlUvsULqtMCDLb8jXCSOvEITYgj15YP1nwMibXiamw/640?wx_fmt=gif)

**JWT 简介  
**

```
Web服务使用最多的认证方式是基于Session的认证

https://blog.csdn.net/qq_36119192/article/details/84977902

但是这种模式最大的问题是，没有分布式架构，无法支持横向扩展。
```

```
如果使用一个服务器，该模式完全没有问题。

但是，如果它是服务器集群或面向服务的跨域体系结构的话，

则需要一个统一的session数据库来保存session会话数据实现共享，

这样负载均衡下的每个服务器才可以正确的验证用户身份。

那现在有这么一个需求：站点A和站点B提供同一公司的相关服务。

现在要求用户只需要登录其中一个网站，

然后它就会自动登录到另一个网站。怎么做？

一种解决方案是听过持久化session数据，写入数据库或文件持久层等。

收到请求后，验证服务从持久层请求数据。

该解决方案的优点在于架构清晰，而缺点是架构修改比较费劲，

整个服务的验证逻辑层都需要重写，工作量相对较大。

而且由于依赖于持久层的数据库或者问题系统，会有单点风险，

如果持久层失败，整个认证体系都会挂掉。

那么，JWT(Json Web Token)诞生了！


JWT的原则是在服务器身份验证之后，

将生成一个JSON对象并将其发送回用户，如下所示。

```Swift
{
    "UserName": "admin",
    "Role": "0",
    "Expire": "2019-08-26 12:25:36"
}
```

之后，当用户与服务器通信时，客户在请求中发回JSON对象。

服务器仅依赖于这个JSON对象来标识用户。

为了防止用户篡改数据，服务器将在生成对象时添加签名。

这样，服务器不保存任何会话数据，即服务器变为无状态，

使其更容易扩展。

JWT数据结构

JWT由三部分构成，header（头部）、payload（载荷）和 signature（签名）

头部


JWT头部分是一个描述 JWT 元数据的JSON对象，通常如下所示。

```Swift
{
    "alg": "HS256",
    "typ": "JWT"
}
```
在上面的代码中，alg 属性表示签名使用的算法，默认为HMAC SHA256（写为HS256）；
typ 属性表示令牌的类型，JWT令牌统一写为 JWT。


最后，使用Base64 URL算法将上述JSON对象转换为字符串保存。


有效载荷


有效载荷部分，是 JWT 的主体内容部分，也是一个JSON对象，

包含需要传递的数据。JWT指定七个默认字段供选择。

- iss：发行人
- exp：到期时间
- sub：主题
- aud：用户
- nbf：在此之前不可用
- iat：发布时间
- jti：JWT ID用于标识该JWT
除以上默认字段外，我们还可以自定义私有字段，如下例：
```Swift
{
    "name": "admin",
    "role": 0
}
```


请注意，默认情况下JWT是未加密的，任何人都可以解读其内容，

因此不要构建隐私信息字段，存放保密信息，以防止信息泄露。

JSON对象也使用 Base64 URL算法转换为字符串保存。

签名


签名哈希部分是对上面两部分数据签名，通过指定的算法生成哈希，

以确保数据不会被篡改。

首先，需要指定一个密码（secret）。

该密码仅仅为保存在服务器中，并且不能向用户公开。

然后，使用标头中指定的签名算法（默认情况下为HMAC SHA256）

根据以下公式生成签名。

```Swift
HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload),secret)
```


在计算出签名哈希后，JWT头，

有效载荷和签名哈希的三个部分组合成一个字符串，

每个部分用"."分隔，就构成整个JWT对象。

**Base64URL算法**


如上所述，JWT头和有效载荷序列化的算法都用到了Base64URL。

该算法和常见Base64算法类似，稍有差别。

Base64中用的三个字符是"+"，"/"和"="，由于在URL中有特殊含义

，因此Base64URL中对他们做了替换：

"="去掉，"+"用"-"替换，"/"用"_"替换，这就是Base64URL算法。

JWT用法

客户端接收服务器返回的JWT，将其存储在Cookie或localStorage中。


此后，客户端将在与服务器交互中都会带JWT。

如果将它存储在Cookie中，就可以自动发送，但是不会跨域，

因此一般是将它放入HTTP请求的Header Authorization字段中.

当跨域时，也可以将JWT被放置于POST请求的数据主体中。
如下是放在 X-Access-Token字段中。

JWT验证流程

后端服务器收到客户端发来的JWT数据的话，

根据Base64 URL算法将header部分还原，取出加密算法。

根据加密算法、payload、secret 进行重新签名，

并且比对签名值来判断该字符串是否被篡改。

 Reserved claims 也会被用来进行校验jwt字符串，

下面我们来一一列举。

- iss(Issuser)：如果签发的时候这个claim的值是“a.com”，验证的时候如果这个claim的值不是“a.com”就属于验证失
- sub(Subject)：如果签发的时候这个claim的值是“admin”，验证的时候如果这个claim的值不是“admin”就属于验证失败
- aud(Audience)：如果签发的时候这个claim的值是 “['b.com','c.com']” ，验证的时候这个claim的值至少要包含b.com，c.com的其中一个才能验证通过；
- exp(Expiration time)：如果验证的时候超过了这个claim指定的时间，就属于验证失败；
- iat(Issued at)：它可以用来做一些maxAge之类的验证，假如验证时间与这个claim指定的时间相差的时间大于通过maxAge指定的一个值，就属于验证失败；

JWT问题与趋势

1、JWT默认不加密，但可以加密。生成原始令牌后，

可以使用改令牌再次对其进行加密。

2、当JWT未加密方法是，一些私密数据无法通过JWT传输。


3、JWT不仅可用于认证，还可用于信息交换。

善用JWT有助于减少服务器请求数据库的次数。

4、JWT的最大缺点是服务器不保存会话状态，

所以在使用期间不可能取消令牌或更改令牌的权限。

也就是说，一旦JWT签发，在有效期内将会一直有效。

5、JWT本身包含认证信息，因此一旦信息泄露，

任何人都可以获得令牌的所有权限。为了减少盗用，JWT的有效期不宜设置太长。

对于某些重要操作，用户在使用时应该每次都进行进行身份验证。

6、为了减少盗用和窃取，JWT不建议使用HTTP协议来传输代码，

而是使用加密的HTTPS协议进行传输。

JWT安全问题

- 由于JWT传输过程中的加密方法是Base64URL，而Base64 URL能够轻易解码，所以如果敏感数据在JWT中，是非常危险的。
- 未校验签名。某些服务端并未校验 JWT 签名。所以，可以尝试修改 token 后直接发给服务端，查看结果。
- 一些JWT库支持 none 算法，即没有签名算法，当 alg 为none时后端不会进行签名校验。将alg修改为none后，去掉JWT中的signature数据（仅剩header + '.' + payload + '.'）然后提交到服务端即可
- HS256（对称加密）密钥暴力破解、




JWT在线网站：https://jwt.io/#debugger

关于对JWT攻击的文章：https://mp.weixin.qq.com/s/GwNLPLwBW4IOJ-X5P0yyNA如果想跟我一起讨论，那快加入我的知识星球吧！




来源：谢公子的博客

责编：浮夸



由于文章篇幅较长，请大家耐心。如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。
最后，欢迎加入谢公子的小黑屋（安全交流群2）(QQ群：932811543)
```