> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LrLPvlO3gEdtWswi7GYvQw)

**1.M****imikatz 源码下载**

首先在 github 下载 mimikatz 源码  
https://github.com/gentilkiwi/mimikatz  
使用 vs2017 打开工程  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sMKOf0L2cJLwia97WCRkNXFuEdl5Ik0ania8Bh435hIwgwicDIUwmJ9qEw/640)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5suOnUbWDDdLbsVic3gKsiaBEw09K4gJvEF8icJX3mzQl5wVyibx0PicYs5pQ/640)

**2. 替换 mimikatz 字符串**  

```
在程序没有运行的情况下，一般都是通过特征码判断的。而且Mimikatz这些知名工具的内容像mimikatz、作者信息之类的字符串就很容易被做为特征码识别。

通用阅读源码大体可以了解存在比较明显的关键字：mimikatz、MIMIKATZ以及mimikatz/mimikatz/pleasesubscribe.rc文件的一些内容。可以利用Visual Studio的替换功能实现关键字的处理。操作如下：

编辑 -> 查找和替换 -> 在文件中替换 -> 区分大小写

mimikatz替换为wooyun

MIMIKATZ替换为WOOYUN

将mimikatz.xx文件重命名为wooyun.xx（“xx“代表任意后缀）

编辑 mimikatz/mimikatz/wooyun.rc，将一些名称进行修改，还有种类编辑器注释作者名称。
```

按 ctrl+shift+f 替换所有文件中的 mimikatz 字符串  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sjFcT8EYsGyRvgrp0p1xnnHo1hVh0o3xLr1zQp8GntKCbaJQA3RNvVQ/640)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sxwPcsiaGYyCu2657xWdk0D7jxY2Jdmg4ib8fhINfGe9OuV2UiccKNlt6A/640)

**3. 解决方案配置**

首先勾选 release  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s6wpMAcD5sv89TSLPg4EbVecuKmGya74wDwxic1bibnH9rNfWnofMolhQ/640)然后右键 mimikatz 项目属性，在常规中 MFC 的使用中选择在静态库使用 MFC  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sm4SCovJAXiaSo1J0lv9DMeaDyGMmzLMN53Xicm0e3c6AxkSLo7ibKv3wg/640)在 c/c++ 中运行库选择多线程（/MT）  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sAJCfCiawO71o9S3vm6Da8gqYOUokiba8ibE5ln9JjquobwWia9PBDVFgWA/640)

**4. 解决报错**  

点击生成会发现两处报错都是找不到 wooyun.h  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sBT6R6IicThlVhWBJQ1dETMt6I0RxWYeC5uQkc86e94fFZI870VAStTA/640)双击这一行，会来到报错代码处  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5siad2EB4jUFNMsJAicQzwuhSUWZkt8txjA2xMkGhUK0oIEiaB0FbMBnicvQ/640)将 wooyun 重新改为 mimikatz  
还有一处一样操作  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sG7OU1YaUeYAz7KCklCicjsPqNSnvMfTjpHIAFiavBGibDFsD4rI5rIDHA/640)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sPeL8d8BzzEItReU7P2IDvn0wjdoia2MN70K5g47Fxj7q06ITic0VIH8Q/640)重新生成，依然是找不到文件错误  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5swQhMxtk0tD2Kwcibee1L1aZc8IhLkbSAgibAQuHbWiafPOPnxme9uO7icA/640)将 wooyun.ico 改为 mimikatz.ico, 重新生成  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5srscl40N2JWTltyoz8YeF6raf7Qqhic946iaC3FYx27wiaeZoBULQyacgQ/640)

**5. 删除默认的静态资源**

使用 360 查杀发现已经免杀  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sFC6zVvGWiaH76ZuRTTNj7qhdP4UBa7ibQpwq7xpBxquASRWYGV6hYF0w/640)  
但是发现打开 mimikatz 时会被 360 动态拦截  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5sK15EAmCEuvMibsGPF3CS4fbTGKkYaTlLCzEtGmKKn7ic7WDp1E6EDUibQ/640)使用 Restorator 2018 编辑静态资源  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s5EfvMibS6oWAdfiawcvqcl2ZBQste9gfialGc4pHeVVe1JMrTD3wG98eg/640)删除图标和界面风格  
打开 mimikatz 发现已经绕过动态查杀  
![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9HGeK56ich8WjeSR6yWSr5s2JOZuM8aKd5kHcwohcdmWNaGDiaLL7eJc3Mx7mJmxevDicic6HvBdxtrg/640)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

**分离免杀 mimikatz**
-----------------

https://github.com/lengjibo/RedTeamTools/tree/master/windows/mimikatz_bypassAV

**[内网渗透 | 了解和防御 Mimikatz 抓取密码的原理](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247497583&idx=1&sn=ad464674651228120c45c656f3beb46f&chksm=ec1ca250db6b2b463495c9c4366a7f4f78aff4dfd079e134cdfecd3222e3787561d50ec8c4a8&scene=21#wechat_redirect)**  

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8Qfeuvouibfico2qhUHkxIvX2u13s7zzLMaFdWAhC1MTl3xzjjPth3bLibSZtzN9KGsEWibPgYw55Lkm5VuKthibQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496998&idx=1&sn=da047300e19463fc88fcd3e76fda4203&chksm=ec1ca019db6b290f06c736843c2713464a65e6b6dbeac9699abf0b0a34d5ef442de4654d8308&scene=21#wechat_redirect)

**点赞，转发，在看**

原创作者：cwkiller

文章来源：www.cnblogs.com/cwkiller  

如有侵权，请联系删除

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)