> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5Xhz8sNOmagQPZ_IaHkR8Q)

_**前言**_

  

**申明****：本次测试只作为学习用处，请勿未授权进行渗透测试，****切勿用于它用途！**

_**Part.1 漏洞描述**_

  

杭州法源软件开发有限公司开发的实践教学平台系统下的法律知识数据库系统存在通用 SQLi 漏洞，DBA 权限，可 Getshell。  

  

_**Part.2 漏洞影响**_

  

最新版本  

  

_**Part.3 FoFa 语法**_

  

```
title="实践教学平台 – 杭州法源软件开发有限公司"
```

  

_**Part.4 漏洞复现**_

  

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGYmoJ1EiboC3trNF5F57DY7p4xcu3VzoKzLuhygOa2eXXwUWYPQFezCAKvGYxNj9AhHclp5SXJ6hyg/640?wx_fmt=png)

**漏洞 url:**

```
http://xxxxxxx/JusRepos/ui/login.asp
```

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGYmoJ1EiboC3trNF5F57DY7pUibddEXLFI8mKC3m4tKJJWPwwU5LlWvmVFEmcwjwiaRXiciaQo82iaYfkwA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGYmoJ1EiboC3trNF5F57DY7peuhFD1g6JkCe1sTHIBY8TCD1JGMkMuQ4ia7QiaZCibic61CWwbHHJzFia0w/640?wx_fmt=png)

**注入参数为：**

```
txtUser=*
```

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGYmoJ1EiboC3trNF5F57DY7pkNnicaUIRYJdjUrg1f0C3IKficen6kQn1PnzfG02Qd32WmrZByXFOSrg/640?wx_fmt=png)

DBA 权限，可 OS-shell  

  

  

  

_**Part.5 活动  
**_

  

**公众号目前还在举办官网投稿送书活动，大家踊跃投稿呀！！！  
具体可以看公众号的近期推文。**       
**F12sec 官网：**  

> http://www.0dayhack.net

**给公众号点点关注吧  
**

**球球师傅们了**！  

![](https://mmbiz.qpic.cn/mmbiz_gif/z1BDHniaudwlfmWtOG25nMTy7Rm8fr8FiaZGaNFeOjQK1wfqWpcrZc3TPvRmyYN2r2qC8JFGuMFEXobVMYp9hQzQ/640?wx_fmt=gif)

扫码二维码

获取更多姿势

**F12sec**

![](https://mmbiz.qpic.cn/mmbiz_jpg/EWF7rQrfibGZjT4wKhhUiaY0Vfb11FayFhDumgDvFHln8q6rttXdllugQU7ibcLLOxp5H581iayytcnXEPwibEO8yqw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/eHpJZ2bXAXdly9aB5q6xe9xjE66TzF3GbwhdOYtfUyyejGYeOcS7L6yn8WP1LflIANPiafT4h0kghD7MGhJkqAA/640?wx_fmt=png)

  

往期推荐  

[实战 | 记一次利用 mssql 上线](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484628&idx=1&sn=2345aec9a4550a194dc5a28b0c5cd496&chksm=c07fbf20f708363614e51e5525c1aad9b4c8f5a50b391b0c83f11d1073a26d7bb8f4dc3a9fc8&scene=21#wechat_redirect)  
[漏洞复现 | 某系统通用（0day）](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484741&idx=1&sn=62989d6b4d1540a8ec829f665e42e033&chksm=c07fbeb1f70837a7577a86d3687a8c8fa177eb5115e14b7fe0727b80021baaad6378b3a62c76&scene=21#wechat_redirect)  

[实战 | 利用无线渗透加内网渗透进行钓妹子 (上篇)](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484701&idx=1&sn=39a82092e0c16d219b50d5ecd1274674&chksm=c07fbee9f70837ffe6f35b87bea98da39379b1e94d5cb4ce0cbc18c2b008b1bcef20954aefb6&scene=21#wechat_redirect)

[漏洞复现 | Microsoft Windows10 本地提权漏权 CVE-­2021­-1732  
](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484765&idx=1&sn=c454dc78e4340bcaae2f6cc1f7d944fa&chksm=c07fbea9f70837bf68bf4b37358bdbf9affb87e5fb077e55db2136da53303b697b96ead5ce10&scene=21#wechat_redirect)

[漏洞复现 | （通用 0day）金和 C6 协同 OA 管理平台后台存在水平越权漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484809&idx=1&sn=833a086a0d4a75e4bedbc4e2f7bcf19d&chksm=c07fbe7df708376befb44b4398a5c576d3c59f85304bdcf4c0d64764ecad4c50d518d926e846&scene=21#wechat_redirect)