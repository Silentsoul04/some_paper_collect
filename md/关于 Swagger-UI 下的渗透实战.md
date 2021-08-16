> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/n4W8VbEly1GzGZM6C4jHBg)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)

> **文章作者****：M1kh**
> 
> **博客链接：https://blog.m1kh.com/index.php/archives/403/**

**前言**
======

最近在测试时频繁遇到 Swagger UI，趁着有空，就记录下。

**如何发现 Swagger UI**
===================

正常情况下，当访问一个网页出现以下错误，可根据报错回显看出这是典型 Spring 的接口报错信息。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcINXxQM70yhAKkaXV8NrAOmYGa5ibruP4LHtt9q7e5WxvVdhUsqfT8gw/640?wx_fmt=png)

另外 Spring 站点的默认 logo 是片小绿叶，可以观察下报错页的 logo 有没有小绿叶，或者拼接 favicon.ico 访问，如果出现以下图标，则有很大可能是 Swagger UI 的站。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcdiaB4WqtHYLiaWpnB1XCesqsLXtwBjpVAD924ztv0QVtE0T5VGeVNZOQ/640?wx_fmt=png)

如果这些都无法实现，则可用字典去遍历 Swagger UI 路径，当一级目录不存在时，可尝试拼接二级目录进行访问。

常见路径

```
/v2/api-docs
/swagger-ui.html
/swagger
/api/swagger
/Swagger/ui/index
/api/swaggerui
/swagger/ui
/api/swagger/ui
/api/swagger-ui.html
/user/swagger-ui.html
/libs/swaggerui
/swagger/index.html
/swagger-resources/configuration/ui
/swagger-resources/configuration/security
/api.html
/druid/index.html
/sw/swagger-ui.html
/api/swagger-ui.html
/template/swagger-ui.html
/spring-security-rest/api/swagger-ui.html
/spring-security-oauth-resource/swagger-ui.html
/swagger/v1/swagger.json
/swagger/v2/swagger.json
/api-docs
/api/doc
/docs/
/doc.html
/v1/api-docs
/v3/api-docs
```

扫目录建议使用 burp，通过返回包可看到完整的数据。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc6ibPnA64nPrAbmg7pWwwkUdNnfo4jGZWibgmTYHqSFJdJY9AquQhCMJg/640?wx_fmt=png)

某些时候，swagger-ui.html 会被禁止访问，这时候可以尝试拼接 / v2/api-docs 进行访问，如果一级目录 404，可以尝试拼接二级目录访问，以此类推。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcSrJscQZVCVj7nHbVDp3rpiclTj1kicE3Hibh9WtC07Usr0HuicibAI96FpA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc1Q8rmE6iaXvQ3X7FbeJEZWq9bweOWHZNX1lDootqxvQpMgLQyeom0Pw/640?wx_fmt=png)

另外，有些时候在 js 和 HTML 源码里也能发现 Swagger UI 的身影，具体怎么去做，还是看个人的思路和站点架构而定。

**漏洞挖掘**
========

由于 Swagger UI 接口中有详细的参数介绍，所以可以直接在 swagger-ui.html 页面构造参数发包，如果该接口没有权限验证，则会造成严重的安全隐患。

发包示例：

先点击 try is out，然后构造参数，excute 可直接发包。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc4cZ1ZYFb96ao55uIdgYuAFPyhI1bZ8DibDM0LPFoDwN0OjL6Hkh7rTw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fcibk27oicPkNr9G0BHztNiadVtyicgwr9ER064fgalMxossAQYLKPRBCibCg/640?wx_fmt=png)

由于 Swagger UI 的安全性问题，延伸出了几个常见的漏洞。

**文件上传漏洞**
----------

类似于 Swagger UI 这种上传接口，如果在部署时没做限制，一般都是任意文件上传的，当然，我们可以找一些 temp、test 这类上传接口，因为此类接口多数是开发过程中用作测试的，这种接口几乎都是无限上传文件类型的。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcghTXSF3BZcc7zBMDvic2kY9beSh972ibCbWbcu6N4yNP0eWahG6NTnwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcMylUyu2SfT2r0VgXgFmRGtHBz0Pzz6SsSAicUAqJibP3jYhWS9AMdCiaA/640?wx_fmt=png)

实战的时候遇到过一种情况，先是扫目录扫出一个 uploadtest 上传接口，在本地构造表单进行上传，通过 fuzz 找到了上传参数，但是上传成功却没返回路径，只返回了一个 file_id 号，根据这个 id 号猜测后端是以 id + 上传文件的绝对路径保存在数据库中，之后通过挖掘 SQL 注入，利用 sqlnmap 的 --search -C 参数找到了文件路径。

**用户身份信息泄露**
------------

某次测试的时候在 Swagger UI 发现了一个 api 接口存在未授权访问，只要知道手机号就可得到用户身份证、真实姓名等信息，且此处存在用户名枚举漏洞，因为后台登录功能采用了手机号 + 身份证 + 密码登录的形式，所以我们可针对该 api 编写脚本进行利用，提取身份证号，和手机号，为后续爆破后台做准备。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc0wcL7IA2QOI4644EbPsRww7FtRf1ZbOvnRj5lcDz3tRZZFn869dVtA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcWFM1T4EX6Md5x9gurC0hzTOSnuDXEwcLTXV3yoDibniaVPWqElcXGfaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcoPEEFZaJuz8QIF7t6I0OrRWuIlDyTrPf4bWNrlIiagTTop3sqAoJIow/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcXU6tzuxx0SU3oNwuAlmAUFqPQalKlPDN5DGDUapiaSNc0rjTfXwr7PA/640?wx_fmt=png)

然后利用提取出来的 api 信息构造爆破字典成功登录后台。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcetR4pfkvV6mu8KX19FibSocaZhQc0ibW0qxBHpXqI4pfz3QkFoibO5FSQ/640?wx_fmt=png)

  
遇到过很多的 Swagger UI 接口，很多时候其 api 都是裸奔的，而不少的 api 往往都包含着大量的敏感信息，这时候我们就可利用其缺陷，进行更深入的渗透，来把漏洞危害最大化。

**sql 注入**
----------

关于注入遇到的比较少，我去 fofa 找了几个 Swagger UI 复现了下，发现跟普通注入都大同小异，只要得到注入点就能利用了。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcRpQFJ3TE0I6qElibFPxvAI3uKeJ3t5VGtiaoZaWKNxQqWAvjzZXmhYKQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcIMG9jib68KUnnnYmFygqCwFHUVIpxlKMRYyYjgSU29IcxpbujmCx0nA/640?wx_fmt=png)

**任意文件下载**
----------

这个漏洞在 Swagger UI 里也是非常常见的，可直接在 API 文档里搜索关键字，如：downLoad、filename、path 等，之后就是构造参数测试了，这个就不多说了。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcYfBYVtPoPTEbykbHPWIBBUdyYKyFBWMiaiamQmukoIsvAHRtYaNh79DQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc3Ohy3diaE4RFzrjxgyUwV5uiaX6e605bZfv5Vxgy86LkFI9k7CLicCCZQ/640?wx_fmt=png)

**实战举例**
--------

以近期做的一次项目举例。

这是个复测的站点，站点存在 waf，仅有一个登录页，验证码有效，且有多因子认证，同事在第一次测试的时候是无漏洞情况。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcDKAeNDRb3IicatBTJsoD5AkAd2ibaqZJ1icfq2jpmCFma5Gsdvhb7KXgw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcN5PscHnhzfxWHDoaEYlhKdQy7NRiazM5iakq27fibZjlyicpQX2em1BuGA/640?wx_fmt=png)

接手了这个项目，简单的信息收集一番后便开搞了，因为时间关系，便迅速展开测试了。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcXvEzKHYcwI6hhOfaiaN81BtcdfN5JYgv3ia1gW99paQIhb2lq9kYdA3w/640?wx_fmt=png)

站点只有一个登陆页，存在 waf，且验证码有效，登陆多因子认证，经过端口探测和前端漏洞挖掘没发现突破点，后来翻查了 js 也没能发现有用的东西，正在一筹莫展的时候，一个报错也引起了我的注意。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcNdFFvtWXLnXp8WvJ4nNQq0HUWk06vHuzibZbrKbcB1WdWXZb6cD8beQ/640?wx_fmt=png)

根据回显可以看出这是典型 Spring Swagger 的接口报错信息，拼接下 / favicon.ico，看到了熟悉的 logo。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcUdKQj7uvWquvKxlv8OGS9WMKgMFhE5Ixrmcd7RW36s72g4SpTI087A/640?wx_fmt=png)

之后都不带考虑的，直接拼接 / v2/api-docs 访问，然后就得到了一个 Swagger-UI api 文档。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcFso7UBF22QWCcwfhwt7TXebvenaX2OdqmWEsLEkDibPKqcj7HfuDJqg/640?wx_fmt=png)

由于这个 json 接口不能直接构造参数发包，为了测试便捷性，尝试利用 swagger-ui.html 进行参数传递，访问 / swagger-ui.html，但该页面做了限制，无法访问到，使用 /%20/swagger-ui.html 进行访问，但也失败了，之后使用字典去遍历 swagger-ui 路径，但也只得到了这个页面，没办法，还是得回到 jsonAPI 进行利用。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcibDmfkpgjCEpYepCPezicErJ3u4SicWjJ6J1Oia6Sa3CfGbGnjHpvbPAQQ/640?wx_fmt=png)

### **未授权漏洞**

通过翻查文档，得到一个查询用户的 api 接口，点击 parameters，即可得到该 api 接口的详细参数

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcrocZqxtia2Dyy5ERnnAQaL3pqQibUrA6M3wrUbeNzRtmSAr2Il1z3mGw/640?wx_fmt=png)

有了参数就好搞了，直接构造参数发包，开始之前，把无痕模式打开，以证明可以未授权访问，通过回显可以看到，得到了大量的用户信息，包含了手机号，邮箱等等。

```
/findxxxs.action?start=1&limit=99
```

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcSJIyvic3K0Mv3TPcgk55xRCC7u987OqiaicMdVgIVC5rG5CRQ86ypfo1w/640?wx_fmt=png)

虽然得到了部分用户信息但却没包含密码，构不成什么威胁，之后再次翻找和用户有关的 api，但也没多大效果。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fcoq8iajV6pVM2VIZkiaCkbx05yooeAYhCUBfJVw6JL60mQaaiaxianUEGjQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcEiaQyYHibjib5jicnBjV4SLkMmBVPBDRNoibk5xNW2yGicL29Xfu8svX54VQ/640?wx_fmt=png)

之后发现了个用户枚举漏洞，由于是 API，所以不用验证码验证，经过短暂的爆破，得到了几个存在的用户，但并没什么作用，因为在 findxxs.action API 中也能获取这些用户名，无奈只能寻找其它的 api 做突破点。

### **任意用户密码重置**

进一步查找 API 发现了个密码重置 API，由于这种 api 测试起来危害较大，为了自身安全，取得客户同意后，便进一步开始测试了。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcMYsaSGhYkwm0Vbe5mmeJ13g8ViaerVEwhcC7Wxicot4zZfiap5caV14tQ/640?wx_fmt=png)

经过测试，发现可利用泄露的 api 接口信息，构造 userId 参数来重置任意后台用户密码，且该 api 也是可未授权访问的，这样一来，漏洞危害便大大增加了。

利用 api 接口未授权重置 userid1，userid2 登录密码，此时我们还不知道 userid1 和 userid2 的用户名是什么，这时候就要用上 / findxxxs.action API 接口了，因为该接口回显的用户信息包含了 userid，可以根据这个 userid 去寻找用户名。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcVd1GB0mUIdHSUo6dJ8sziaIib3Yib7rqjcXyp4lr0nnpmvE7tQn32gR4w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcWIFIekBYowMEeMds0YPaMWrLWibCDB7w9aGAjILycO0pc3YwUn2ScBg/640?wx_fmt=png)

利用 / findxxxs.action API 接口寻找 username

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc3q2Fbc5h0WjYeRcHic5u1s6hOLlibO3ueGEQAu5UI7K1V33guBhnyClw/640?wx_fmt=png)

前面说到了该后台存在多因子认证的，这时候就算我们重置了用户密码，但是还是无法登陆后台的，想要登录后台，就必须把用户的手机号给重置掉，或者寻找注册 API 注册登陆。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcykCuWBMoibvB57o5GuFU2e9icyiaBFoz1jqI4Y5iadGlZWWXtW5dvag16w/640?wx_fmt=png)

### **任意用户信息修改**

接下来找到了一个修改用户参数的 API，根据 api 接口信息构造参数，尝试利用未授权漏洞重置用户手机号登录后台。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FclJPRBOs9C8Xfnk71D79slvPSJlHQvr9IBQ7XyIiaKrMmMjiaraBep0bw/640?wx_fmt=png)

构造参数发包，可以看到修改成功了，然后到登陆页输入重置后的账号密码登录，却发现重置后的手机号怎么都无法收到信息，后来利用 API 重置为另一个手机号，重新登录获取短信，却也是毫无反应，刚开始以为是重置后的手机号需要等待一会才能生效，后来等了大半个小时，再次重新获取短信，却也是毫无反应，到此，也不得不放弃重置手机号进后台这个突破点了。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcPgdOxXicCjtdHGAqZeMrmrNHMfxrgNlhHZCmdjRt7TW0nU3ntaia1Kww/640?wx_fmt=png)

### **用户身份信息泄露**

由于进不去后台，只能尽量在多挖掘漏洞证明其危害了，通过 api 漏洞挖掘，得到了一个未授权接口，能未授权获取到大量用户身份证号等敏感数据。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc8qgeO4onvp1D3STpJfCPsYmpLsFCstYdibia0snxPc7KsWok1JZRIxFA/640?wx_fmt=png)

再进一步挖掘，发现大量 API 可未授权访问。随后利用未授权获取了大量合同数据，最大化了漏洞危害，到此，本次测试也告一段落了。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcPA3VUgIVuibMu0u3LibkibJDplVdTP9EZ6GAjMAQibJGIOohp8yTGTJdBQ/640?wx_fmt=png)

**其它场景**
--------

大多数时候 api 是会存在认证限制，这时候我们可以寻找其它的 api 进行测试，Swagger UI 大多会存在数量庞大的 API 接口，除非所有的接口都存在认证限制，否则一旦有漏网之鱼，就能被我们所利用，可以重点关注 test、temp 之类接口。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc137eVkDfFXfHWSDDHMJZFwHNyibXpeB6lZwDiaYXlBoDdAPEU4lLoeLw/640?wx_fmt=png)

当得到一个存在权限认证的 Swagger UI 接口时，可关注下 Select a spec，因为有些时候会存在多个接口文档，有可能当前 API 存在权限校验，而另一个 API 文档则裸奔的情况。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5FcIwWI2z7NMRVvI8kk5ic2OOrMrcN8jM0RT9o9b6ar6BHZicVp8hAkee8Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib6UZwwwjOXEVQOkfGQf5Fc1XHibvpRESezuC2F0Xjc2ECTGS8OZicHo2wibanicUR6VquWapKFQz7nwA/640?wx_fmt=png)

存在 spring 框架的站点要关注下 druid，因为 Springboot 会集成 Druid，多半开发会混合着使用，在挖掘 Swagger UI 的同是也不要忘记关注 druid 未授权的突破点。

此前也写过关注 druid 的漏洞利用，可参考 druid 未授权利用。

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)