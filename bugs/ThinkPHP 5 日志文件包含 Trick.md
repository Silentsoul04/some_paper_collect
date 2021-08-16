> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/kdEbtt4SjyuDXMCnOyQ9CA)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

原文地址：  

*   https://www.cnblogs.com/r00tuser/p/14344922.html
    

**0x01 前言**

本文源于实战场景，以下所有测试均基于 Fastadmin 前台模版 getshell 漏洞环境。  

**环境：**

```
Win10 
Phpstudy 2018  
PHP-7.0.12 NTS+Apache
Fastadmin V1.2.0.20210125_full 
ThinkPHP 5.0.24
Fastadmin默认配置 (不开启app_debug和app_trace)
```

**0x02 正文**

我们知道在 Thinkphp5 没有开启 app_debug 的时候，能够写入日志文件的信息很少而且只有触发报错的时候才会写入部分日志信息，如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhZ9QERIuOacEWBxjQwpXSgZc3WAzwrwSwDDiazIpib2fXyAyjDtMtGBlA/640?wx_fmt=png)

而直接用 url 传入 php 代码，空格会被 urlencode。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhjWk4ASLicNoE4N4Po1UREFia5LqGYMBb7BHdZoLyl6Yuqt06sliadU64w/640?wx_fmt=png)

观察日志信息，与及分析代码，可控有蓝色框的请求 IP 地址，红色圆圈的请求方法，与及后面的 host 和请求 uri，对应代码：

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhGlAtYCNKBEFHZ9G0jcAGC3MKyJOJkMwnXS9VlrfVK69ia2nfV5ibuh8w/640?wx_fmt=png)

一个个分析一下，ip 可以用 X-Forwarded-For 等，但最后都过滤了。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhgODAMHTHYwqMNPw58pSlYpmtBXIIVibvF5pnuTicnCgIXz6jxLia4ydXg/640?wx_fmt=png)

**method：**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJh1bFDOL3QSJiaCCYFY6LZr661X5PLHTounh5WBPmwzxN2r6IwHoZ3OoQ/640?wx_fmt=png)

**host：**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhMINs8zga2ocOhJtus29XqHJZBxyrSKGB8lTTJcK0iaQ55gP7V89FxPA/640?wx_fmt=png)

**uri：**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJh4icibw3j17qEwfTlmYwBEa3x5KaZN5LGskp7xkibib1R4VPjOSuLDLEsRQ/640?wx_fmt=png)

可以发现可用的选择还挺多的，method 可以用 X-HTTP-METHOD-OVERRIDE 头，host 可以用 X-REAL-HOST，uri 可以用：X-REWRITE-URL。

```
X-REAL-HOST: <?php phpinfo();?>
X-REWRITE-URL: <?php phpinfo();?>
X-HTTP-METHOD-OVERRIDE: <?php phpinfo();?>
```

**一一对应：**

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJh7n3Z0VevqYOPficVa64wVsmpsEs7ibWE8fk2fyibagajxicibeLia3YPWWpA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhWiaILyLvxEJraSeblxq0nM2PGzsJPC8soPAwjHCTiaEfhzrM00L7wUtQ/640?wx_fmt=png)

有一点需要注意，看上图，用 method 头会换成大写，PHP 马写进去之后解析可能会出问题，所以建议还是用 host 和 url 的两个头

**实战场景：**Fastadmin 普通用户可以登陆，有模版渲染漏洞，没有开 app_debug，无法修改头像，用模版渲染日志文件 getshell

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhk3YYRib9pXobLHlS6NuNvjrD37HcLlXcluls10xcbjm1T5Vg7m6kicMA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhhTyXHibDnKiaPD7RnnlBmgYNib1uwIoa9hX1D9ZH32ibYVyqwibvnQuXdVw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfmMddD6jhzA4CsgGJnhQJhPNmJoD3Q5CkmtibSFkhLiazLRm3RdZxcgW0zU6dRibBlIZQGG7icXicUZ8w/640?wx_fmt=png)

**0x03 总结**

遇到类似的场景时，基于 tp5 的文件包含、模板渲染写入 PHP 代码时可尝试用上述的请求头。  

关注公众号回复 “9527” 可免费获取一套 HTB 靶场文档和视频，“1120” 安全参考等安全杂志 PDF 电子版，“1208” 个人常用高效爆破字典，“0221”2020 年酒仙桥文章打包，还在等什么？赶紧点击下方名片关注学习吧！

公众号

* * *

**推 荐 阅 读**

  

  

  

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcAcRDPBsTMEQ0pGhzmYrBp7pvhtHnb0sJiaBzhHIILwpLtxYnPjqKmibA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247487086&idx=1&sn=37fa19dd8ddad930c0d60c84e63f7892&chksm=cfa6aa7df8d1236bb49410e03a1678d69d43014893a597a6690a9a97af6eb06c93e860aa6836&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcIJDWu9lMmvjKulJ1TxiavKVzyum8jfLVjSYI21rq57uueQafg0LSTCA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486961&idx=1&sn=d02db4cfe2bdf3027415c76d17375f50&chksm=cfa6a9e2f8d120f4c9e4d8f1a7cd50a1121253cb28cc3222595e268bd869effcbb09658221ec&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf8eyzKWPF5pVok5vsp74xolhlyLt6UPab7jQddW6ywSs7ibSeMAiae8TXWjHyej0rmzO5iaZCYicSgxg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

**欢 迎 私 下 骚 扰**

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XOPdGZ2MYOdSMdwH23ehXbQrbUlOvt6Y0G8fqI9wh7f3J29AHLwmxjIicpxcjiaF2icmzsFu0QYcteUg93sgeWGpA/640?wx_fmt=jpeg)