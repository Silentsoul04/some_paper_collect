\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/ajB1zXQ2nmIA\_E7DINKa0Q)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqczeflvHvDexuf2BhBEBYlJCdjJS6aVZ0w6ooY5QwK27L2khaJWEOVdw2kunkBTviakCv6QeGxYjHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
===============================================================================================================================================================================

分享3个burpsuite插件，希望对大家有所帮助！  
    //使用方法很简单，瞄一眼readme就行啦  

    **未授权检测：**  
    https://github.com/theLSA/burp-unauth-checker

    **敏感参数提取：**  
    https://github.com/theLSA/burp-sensitive-param-extractor

    信息提取：  
    https://github.com/theLSA/burp-info-extractor

下面从需求，实现上简单介绍这三个插件

  

===

**burp-unauth-checker**
=======================

**需求**
------

> 自动化检测未授权访问  
> autorize  
> authz  
> authmatrix  
> auto repeater  
> 上几个插件都挺好，但是还是不太符合，想要的是在浏览的时候就能自动检测是否有未授权访问漏洞。

  

**实现**  

---------

python编写，实现IScannerCheck接口doPassiveScan方法

authParams.cfg文件存储授权的参数，如**token,cookie**

默认过滤后缀列表filterSuffixList = **"jpg,jpeg,png,gif,ico,bmp,svg,js,css,html,avi,mp4,mkv,mp3,txt"**

应对一些特殊情况，设置了排除的授权参数列表excludeAuthParamsList

**onlyIncludeStatusCode：**设置检测的响应码，比如只检测200的响应  
  

原本想直接取消掉授权参数，但是可能造成响应失败，所以把授权参数值替换成自定义的数据，如：**cookie:\[空\]，token=unauthp**  
  

**sendUnauthenticatedRequest**发送替换了授权参数值的请求

授权参数分两种  

> 1-在http的header中，如cookie,authorization等  
> 2-在http参数中，如post数据中的token等

**实现**  

> 1-直接将header的授权参数值替换即可：  
> if headerName.lower() in self.authParamsList:  
> header = headerName + ": " + newAuthHeaderVal  
> 2-  
> get/post请求的参数，常规操作buildParameter，updateParameter即可  
> json参数，直接将body数据解析为字典再替换授权参数值  

```


  



```

```
 `if paramType == 6:` `paramKey = para.getName()` `paramValue = para.getValue()` `print paramKey + ":" + paramValue` `reqJsonBodyOffset = self._helpers.analyzeRequest(requestResponse).getBodyOffset()` `reqJsonBodyString = requestResponse.getRequest().tostring()[reqJsonBodyOffset:]` `print reqJsonBodyString` `reqJsonBodyStringDict = json.loads(reqJsonBodyString)` `#reqJsonBodyStringDict = ast.literal_eval(reqJsonBodyString)` `for authParamName in authParamsList:` `if authParamName in reqJsonBodyStringDict.keys():` `reqJsonBodyStringDict[authParamName] = newAuthParamValue`
```

```
最后再对比原请求和替换了授权参数值的响应body  

```

```
`compareResponses：``if (str(nBody).split() == str(oBody).split()):`
```

```
一致则为未授权访问漏洞。
```

  

---

**效果图**
-------

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**burp-sensitive-param-extractor**
==================================

**需求**
------

检测并提取请求参数中的敏感参数名，如userid，username，方便测试越权漏洞，并形成敏感参数字典。  
  

**实现**
------

使用python开发，实现IHttpListener接口，processHttpMessage方法。

param-regular.cfg：参数正则配置文件，id表示检测请求参数中包含id的参数，如userid，idcard等

支持4种参数检测：

> self.requestParamDict\['urlParams'\] = \[\]  
> self.requestParamDict\['BodyParams'\] = \[\]  
> self.requestParamDict\['cookieParams'\] = \[\]  
> self.requestParamDict\['jsonParams'\] = \[\]

先获取请求的所有类型参数，放到**requestParamDict再findSensitiveParam**最后**write2file**。  

  

**效果图**
-------

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

===

**burp-info-extractor**
=======================

**需求**
------

快速提取数据中有价值的信息，如HTTP响应数据包中的用户名，密码等。

比如一个api（/user/list）返回大量用户名/密码，大多数是json格式（jsonarray），就可以使用此工具快速提取信息。  

**实现**
------

使用java开发，实现IContextMenuFactory接口createMenuItems方法。

采用两种提取方式：  
**1.json格式提取  
2.正则提取**

> 1.json格式提取：  
>     采用google gson
> 
> 2.正则提取  
>     re库常规操作即可

  

**效果图**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

欢迎师傅们联系微信，多多交流

**END.**

  

* * *

  

**欢迎转发~**

**欢迎关注~  
**

**欢迎点赞~**