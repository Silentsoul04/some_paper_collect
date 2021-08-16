> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/5052.html)

> 文章目录 一、设置网络代理 & 安装好 CA 证书 二、模拟器安装 Xposed 框架 和 JustTrustMe 模块重启手机 三、开始渗透测试 1. 短信验证码安全性测试 2. 任意文件上传测试 3. 订单支付…

文章目录

*   一、设置网络代理 & 安装好 CA 证书
*   二、模拟器安装 Xposed 框架 和 JustTrustMe 模块重启手机
*   三、开始渗透测试
    *   1. 短信验证码安全性测试
    *   2. 任意文件上传测试
    *   3. 订单支付绕过漏洞
    *   4. 代码安全 - APP 客户端 root 检测绕过 (以中国建设银行客户端为例)
*   总结：

**前言：前些阵子因为疫情蔓延的关系，大家基本都在家自我隔离 & 远程办公。微信好友张三让我帮忙测试他们公司开发的房地产工程建设相关的 APP 客户端等相关测试项目。看看客户端的安全性可以过关不。顺便也写出自己之前渗透测试一些其他厂商的 APP 客户端的过程，双倍稿费不多，所以一定要趁早。本文测试流程仅为个人的思路想法。本文会以这个 APP 为出发点、也会拓扑出一些其他的移动客户端测试小技巧分支出来。希望各位大佬喜欢！**

准备工具：

> Android Studio、MT 管理器、
> 
> 抓包工具：Burp Suite or Fiddler
> 
> 环境：逍遥模拟器 (Andorid 7)
> 
> Xposed 框架、JustTrustMe(关闭 SSL 证书验证)

一、设置网络代理 & 安装好 CA 证书
--------------------

1. 逍遥模拟器 => 设置 =>WLAN=> 长按热点名称 => 修改网络 配置如下图所示。

![](https://image.3001.net/images/20200306/1583506965_5e626615a70a1.png!small)

2. 设置代理监听端口

![](https://image.3001.net/images/20200306/1583507491_5e626823de9fa.png!small)3. 访问 http://burp 下载 CA 证书安装，安装证书需要设置模拟器 PIN 码锁。安装证书之后访问 baidu.com 。在 Burp Suite 就可以看到已经可以成功抓到了 https 的数据包。

![](https://image.3001.net/images/20200306/1583508447_5e626bdf61233.png!small)

![](https://image.3001.net/images/20200306/1583507445_5e6267f50244b.png!small)二、模拟器安装 Xposed 框架 和 JustTrustMe 模块重启手机
--------------------------------------------------------------------------------------------------------------------

![](https://image.3001.net/images/20200306/1583507581_5e62687ddad73.png!small)Android 7.1 64 位不能正常安装 Xposed，请选择 Android 7.1(预览版)

![](https://image.3001.net/images/20200306/1583507671_5e6268d734a90.png!small)重启安卓模拟器设备设备

![](https://image.3001.net/images/20200306/1583507715_5e626903c0608.png!small)

框架安装完成之后，安装 JustTrustMe 模块 (禁止 SSL 验证) 打勾模块然后重启模拟器

![](https://image.3001.net/images/20200306/1583507739_5e62691b7a7c1.png!small)

**三、开始渗透测试**
------------

![](https://image.3001.net/images/20200307/1583518136_5e6291b8b8900.png!small)

模拟器安装 apk 之后打开 apk，打开 app 可以正常抓到数据包，因为这个 app 有 SSL 验证所以不安装 JustTrustMe 模块禁用 SSL 可能会抓不到数据包。

### ![](https://image.3001.net/images/20200306/1583508802_5e626d42a7a72.png!small)1. 短信验证码安全性测试

(1) 漏洞 1 - 短信验证码轰炸漏洞

没有设置图形验证码 / 滑动验证码等 ，只在后端进行了限制发送次数，可以通过手机尾号 +”/n”，修改 cookie 来实现绕过。

![](https://image.3001.net/images/20200307/1583517772_5e62904c2cde5.png!small)

(2) 漏洞 1 - 任意用户注册

![](https://image.3001.net/images/20200306/1583509238_5e626ef668221.png!small)

可以清楚看到返回数据包中的字典”num” 就是验证码信息。

![](https://image.3001.net/images/20200307/1583519099_5e62957bd5a4b.png!small)危害：可以导致批量注册任意用户 (一堆僵尸账号)

(3) 漏洞 2 - 任意用户密码重置

![](https://image.3001.net/images/20200306/1583509650_5e627092d9389.png!small)

重置密码验证码直接回显在返回的数据包中，导致任意用户密码可以被重置。

### 2. 任意文件上传测试

(1) 曲折的用户头像任意文件上传

![](https://image.3001.net/images/20200307/1583512117_5e627a35af10d.png!small)

(正常头像上传)

上传文件没有地址路径回显很尴尬

![](https://image.3001.net/images/20200307/1583512737_5e627ca1e70ab.png!small)

(phpinfo 文件成功上传 后端没有对上传文件后缀名黑名单验证？？？喵喵喵)

修改上传文件名和文件内容 重新上传成功，但是不知道路径有点尴尬。

![](https://image.3001.net/images/20200307/1583513270_5e627eb6188c2.png!small)

后来发现天无绝人之路，可以明显看到用户上传的头像会被重命名，20200306235208248.png 这个名字一看，用脚趾头想都知道是按照时间戳来重命名了用户上传的文件。为了方便快捷验证文件上传漏洞，这里用 python 写了一个工具来跑这个 phpinfo(); 地址, 代码也非常简单：

```
import requests
```

跑了大概 30 分钟找到了 phpinfo 地址: http://************.com/****/upload/**************/20200306235314618.php

![](https://image.3001.net/images/20200307/1583515701_5e6288357d98c.png!small)

![](https://image.3001.net/images/20200307/1583514629_5e628405a0c7f.png!small)

项目测试时候尽量别上传一句话 / webshell 免到时候你洗不干净

### 3. 订单支付绕过漏洞

(1)APP 订单支付绕过![](https://image.3001.net/images/20200307/1583526269_5e62b17df1fc9.png!small)

这个漏洞位置主要发生在： 新用户完成初始化注册的时候、完善个人资料，完善个人资料可以获得积 200 的积分，只要不断完善个人资料就能源源不断获取积分。然后积分可以代替法币来开通会员。

![](https://image.3001.net/images/20200307/1583520386_5e629a8293b61.png!small)

![](https://image.3001.net/images/20200307/1583539005_5e62e33dc4e6d.png!small)

![](https://image.3001.net/images/20200307/1583520579_5e629b43bce85.png!small)

17296 月 / 24 月 = 720 年

2022+720=2742 年 (可以送走好几代人)

![](https://image.3001.net/images/20200307/1583520729_5e629bd909a62.png!small)

(2)H5 商城订单支付绕过 (H5 商城未必真的安全)

这里需要使用另外一个抓包工具 (Fiddler https://www.telerik.com/fiddler 可以抓比较完整的信息流)。这种的订单支付漏洞比较特殊，他漏洞不存在于你部署的支付接口的 SDK 接口中，你修改什么参数都不会绕过去，什么数量 \ 价格为负数，等等都不会绕过。但是这还是存在着漏洞，但是这种漏洞一般作用于虚拟商品，比如 PornHub 的收费影片、喜马拉雅的收费音频、知乎的 Live、[林哲博客](https://www.lz80.com/category/zydq)的精品公开课、某出版社的电子书等等。对实体商品影响不大。

事情是因为疫情期间在家学习，发现有个出版社可以领取优惠卷免费购买电子书

![](https://image.3001.net/images/20200307/1583523665_5e62a75122d1e.png!small)一个账号只能购买 5 本电子书。且不能再电脑端阅读，影响阅读体验。为此就出此下策。

打开 Fiddler 进行相关设置 ：

> Tools ==>Capture Https CONNECTs(✔)/Decrypt HTTPS traffic(✔) 

![](https://image.3001.net/images/20200307/1583522342_5e62a226ddb81.png!small)PC 端登录微信；点击商城链接；

![](https://image.3001.net/images/20200307/1583522658_5e62a3623287e.png!small)打开后是这样子的:

![](https://image.3001.net/images/20200307/1583522716_5e62a39cce950.png!small)

我们随便打开一本电子书，点击购买

![](https://image.3001.net/images/20200307/1583523164_5e62a55c467a3.png!small)

![](https://image.3001.net/images/20200307/1583523239_5e62a5a786be4.png!small)(你尝试修改啥都没用，你想到的别人都替你想到了)

![](https://image.3001.net/images/20200307/1583522957_5e62a48d08267.png!small)这个请求会返回包含表单内所有书单书名的 JSON 数据包；

![](https://image.3001.net/images/20200307/1583526395_5e62b1fb307b9.png!small)

发现这个请求是读取电子书的目录

![](https://image.3001.net/images/20200307/1583524468_5e62aa742e220.png!small)每一本书都有一个指定的 book_id

> bizData[ebook_id]=e_5dfc36588ff4a_K4N5RBZ6

当测试人员点击” 立即阅读” 的时候，会向 xxxxxx.com//get_ebook_content POST 当前这本书的 book_id。来获取这本书的目录。以及资源文件。

![](https://image.3001.net/images/20200307/1583525047_5e62acb754d76.png!small)

book_id 拼接后 的下载链接如下：

> http://**************.com/******************/ebook/[book_id]/1576810082/OPS/copyright.xhtml

![](https://image.3001.net/images/20200307/1583525552_5e62aeb0c6e8a.png!small)![](https://image.3001.net/images/20200307/1583525602_5e62aee27e498.png!small)漏洞验证：

![](https://image.3001.net/images/20200307/1583525675_5e62af2b2bbb0.png!small)

![](https://image.3001.net/images/20200307/1583525810_5e62afb25daf5.png!small)这样子就可以愉快在电脑上面阅读了我购买的五本的电子书了。同时也发现只要有付费书籍的 book_id 也可以进行爬取，这就导致了测试者在测试商城的时候可以不用购买书籍也可以阅读书籍。导致另类的订单支付接口漏洞。

漏洞分析：

> 这个漏洞主要发生在厂商的微信公众号上的 H5 平台上和支付接口 SDK 没有任何关系，只是对资源文件的保护没有做好，而且商城 H5 书单没有对敏感目录设置 token 口令访问验证 ，导致可以任意爬取商城内的所有书籍。相同的这个类型的漏洞也发生在一些知名的互联网厂商企业身上，当然名字不能说，虾仁猪心。

### 4. 代码安全 - APP 客户端 root 检测绕过 (以中国建设银行客户端为例)

部分 APP 不能在 root 设备上面运行，比如中国建设银行客户端就不能在 root 设备上运行，为此来以此 “中国建设银行客户端” 为案例进行剖析，![](https://image.3001.net/images/20200307/1583527818_5e62b78a03c43.png!small)

(加固信息 / 疑似伪加固)

点击查看 =>Dex 编辑器 ++++=> 检索关键词 “root”

![](https://image.3001.net/images/20200307/1583528070_5e62b886c4faa.png!small)

留意到最后一个

![](https://image.3001.net/images/20200307/1583528091_5e62b89b1551c.png!small)![](https://image.3001.net/images/20200307/1583528123_5e62b8bb3622b.png!small)

package com.secneo.apkwrapper;

```
id = 20200306235208248while True:
```

![](https://image.3001.net/images/20200307/1583528693_5e62baf53f94b.png!small)第一行是知名 root 管理的软件包名，第二行是 root 获取的软件包名。第三行第四行均是相关只有 root 权限才能访问的目录。

那么如何绕过中国建设银行客户端的 root 检测，让中国建设银行客户端也可以在已 root 设备上面正常运行呢???

第一种方法，MT 管理器 访问手机目录”/system/xbin/”。

![](https://image.3001.net/images/20200307/1583528980_5e62bc1470235.png!small)

删除 su 文件，然后重启手机再重新打开即可绕过中国建设银行的 root 检测，因为 su 文件不存在，但是也会导致你无法重新访问手机根目录 (慎用)。

第二种方法、Hook 方式绕过中国建设银行 root 检测

HookMain2 的方法代码：

```
url = "http://www.**************.com/*******/upload/userInfoCover/{}.php".format(id)
```

这个是在进程中捕获中国建设银行客户端的包名 (com.chinamworld.main)、如果捕获到相关包名进程则 调用 getClassLoader 的方法代码：

```
id += 1r=requests.get(url=url,headers={"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36"})
```

hookMainActivity 核心代码

```
if int(r.status_code)== int(200):
```

replaceHookedMethod 会完全替换返回 checkRoot 原方法的值为 null，则 root 状态为 false ，否则为 ture，弹出设备已 root 警告 => 强制退出客户端

**Xposed 日志：**

 **![](https://image.3001.net/images/20200307/1583530627_5e62c283711ab.png!small) ![](https://image.3001.net/images/20200307/1583530654_5e62c29e8adc7.png!small)** 

可以看到已经成功绕过中国建设银行 root 检测，至于危害性有多大需要根据攻击者的自身的实力来断定；攻击者可以利用绕过中国建设银行的噱头来骗取小白安装，然后在模块中嵌入一个后门 (https://blog.csdn.net/ALDYS4/article/details/102878693)，实现反弹 shell ，Metasplot 植入 Xposed 模块反弹 shell 

![](https://image.3001.net/images/20200307/1583538826_5e62e28a3bf0d.png!small)

(物理机真机测试)

> 视频效果：传送门 

危害有多大取决于那个人的知识面有多广，如果他愿意他甚至可以 Hook 你的键盘读取的密码，让你蒙受经济损失 。

总结：
---

> 我个人觉得 APP 测试在一定程度上和 Web 测试差不多，毕竟很多都是 H5 写的。其次就是思维挺重要，虽然一些支付接口 SDK 坚不可摧，但是可以换一种姿势来挖掘、前后端校验都需要检查仔细、其次是 Token 口令的生存日期以前访问权限要控制好，最后是代码安全，虽然不知道有时候你 root 检测反而会给不发分子打开一个大门。移动端还有很多地方可以测试，比如小程序、微信公众号、老版本客户端等等。

**参考链接：**

> https://www. [林哲博客](https://www.lz80.com/category/zydq) /articles/terminal/114910.html
> 
> https://www. [林哲博客](https://www.lz80.com/category/zydq) /sectool/167274.html
> 
> https://blog.csdn.net/ALDYS4/article/details/102878693
> 
> https://blog.csdn.net/ALDYS4/article/details/100851377

*** 本文原创作者：艾登——皮尔斯，本文属于[林哲博客](https://www.lz80.com/category/zydq)原创奖励计划，未经许可禁止转载**