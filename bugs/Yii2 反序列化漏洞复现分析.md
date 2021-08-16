> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yj1LRYG0HwJ3-armOUQJWA)

**![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlVMDtusKZ8CjWnMVoiaxxicASL7LmM9AcDIlsFHDcFBGf93HfztrVaw6g8ZQzQF1rbCGbf7gjHONfEg/640?wx_fmt=png)**
------------------------------------------------------------------------------------------------------------------------------------------------

**1、漏洞描述**

Yii 是一套基于组件、用于开发大型 Web 应用的高性能 PHP 框架。Yii2 2.0.38 之前的版本存在反序列化漏洞，程序在调用 unserialize() 时，攻击者可通过构造特定的恶意请求执行任意命令。

**2、环境安装**  

到 github 上下载 yii2 的 2.0.37 版本

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBoicXia1ymKmjVMIAuicGR0Ial7BYIELaWhKlmIy6Sym1NEWsDiaibY4GJYA/640?wx_fmt=png)

解压之后，修改 / config/web.php 文件 17 行 cookieValidationKey, 可以随便定义

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBqVe4TaPXOXXc5HxSUKdADkHKhf0LbGHGqUSxvyakThv3ia72GW7dcgA/640?wx_fmt=png)

然后进入目录 php yii serve 启动

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBKKSEibFaQZPS7BDzlGmUI6ngyU2oIUpkAJktyuyfLn83VlkJrWNjUlg/640?wx_fmt=png)

接着添加一个存在漏洞的 Action

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBGrCibib1wFpO8wI1ria3LKhuWbBqmFRwh0ndTXtqt1JDtVZLuvxb4vibGQ/640?wx_fmt=png)

生成 poc，poc.php

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBJocBWDGwvJtxLFbXdQQEyMLTO41AeGOLthnGQ1vpzk5pJCLSGqEiafg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrByoSCk3S7MfuQCGsiaqsbWQ3KmQOneaSvIakkwwUMM9mtpqiamReeibEzQ/640?wx_fmt=png)

发送请求，环境搭建成功

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrB5ictP3f14vPmYPczawJf7J3icticrCHjPiasG0ibrlduuYaGiaTOebMr07hw/640?wx_fmt=png)

**3、漏洞分析：**

根据 POC 得知利用链为：  
yii\db\BatchQueryResult -> Faker\Generator ->yii\rest\CreateAction 

先分析 yii\db\BatchQueryResult basic/

vendor/yiisoft/yii2/db/BatchQueryResult.php

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrB9uHbbG41Ueq1cz7lUl1BIAJrdc0jHw8QAWpjmFQMVPLnBrcb5DtWRg/640?wx_fmt=png)

BatchQueryResult 中有个__destruct, 调用了 rest 方法，最后调用了 close 方法。根据 POC 知道下一个链为 Faker\Generator，跟进 Generator basic/vendor/fzaninotto/faker/src/Faker/Generator.php

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBbvabHu0gX9910uva5bj9hhmmElz5EyMZc5AibpBZuqianmaDHfOW46dg/640?wx_fmt=png)

发现该文件并没有 close 方法，只有__get，__call，__destruct 3 个魔术方法, get 和 destruct 感觉都没有什么问题，查看 php 手册 call 方法介绍如下：

当对象调用一个不存在的方法时候触发:

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBLDjfN0JCVsLbDvcticpRLAU0YCrT2iaLCGQrFIehud0Zz6iaTCkoXFcnw/640?wx_fmt=png)

所以 Generator 调用 close 方法时候会去调用__call 方法，跟进该方法：  

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBPkyF1cmAYSxb7Rc9e9xUyXKsNegs5SywWymNy1tialUBlq52RpyxLWg/640?wx_fmt=png)

Call 方法有 2 个参数，$method 为调用的方法值，为 close，$attributes 为调用方法所传递的参数为空，跟进 format 方法

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBrauQtQCRficsReGRj53aKmJ7fe7cFZs2xufPJ2Oj6qHcNSHTndibAgpA/640?wx_fmt=png)

format 调用了 call_user_func_array 方法，该方法有两个参数，$formatter 为 close, $arguments 为空，分析 getFormatter 方法，发现 $this->formatters[$formatter] 可控，需要为 $this->formatters[“close”], 该参数为 call_user_func_array 第一个参数，这个时候利用该类可以调用任意类的无参方法。根据 POC 得知，下一环调用了 yii\rest\CreateAction 的 run 方法，查看该方法：

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBwKz5LvCG2QBg6HpL1VR5cjGWf7bFUKdqFL8Ru1a3Z5VYQVCfyOQbHg/640?wx_fmt=png)

Run 方法调用了 call_user_func 方法，先判断 checkAccess 是否存在，存在则调用 call_user_func 方法。

checkAccess 为 rest/Action.php 的公共属性，

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBLakLy3oiauKiapwFiaXPWvqLzZJMlyqdakc6Eygvw0o0qNe20uD9bbUicw/640?wx_fmt=png)

Id 为 base/Action 的公共属性

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlXOibfDt3cibX8UNltru4JDrBXiaqouuibbEia3yBKz9owWSsglxfOicUOjASAv1ly3yKkBAVwqRR0wtDicg/640?wx_fmt=png)

CreateAction 继承 rest/Action.php，rest/Action.php 继承 base/Action。所以只需要在序列化 CreateAction 类的时候指定 checkAccess 和 id 参数的值即可调用 call_user_func 方法，所以整体利用链如下：

1.yii\db\BatchQueryResult-> __destruct->rest

2.Faker\Generator->call->format->call_user_func_array

3.yii\rest\CreateAction->run-> call_user_func

**4、安全产品解决方案**

百度度御关 WAF、高级威胁感知系统，以及智能安全一体化产品已支持该漏洞的检测和拦截，有需要的用户可以访 anquan.baidu.com 联系我们。

参考链接：

https://github.com/0xkami/cve-2020-15148

https://xz.aliyun.com/t/8307

**推荐阅读**

**[Nette 框架未授权任意代码执行漏洞分析](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650495438&idx=1&sn=082eab1e40706c5890f6bf4f5438db3d&chksm=beb3ef8e89c46698461e85d3534ff0a97c6edf07d25576731426b90321e36dc13aacd7901c75&scene=21#wechat_redirect)  
**

**[从 CVE-2020-8816 聊聊 shell 参数扩展](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650493205&idx=1&sn=1e3d4ccba11cc2b708f0fdec82d72587&chksm=beb3e75589c46e43271ec0c997d53ffabcdc8596476718d2aff510fc55b6c01040b5c481fe8b&scene=21#wechat_redirect)  
**

**[Shiro rememberMe 反序列化攻击检测思路](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650493186&idx=1&sn=39b99df8a3d82534cb9fd255f9e0059d&chksm=beb3e74289c46e5455698fd3c1758b865dbd10fd7b513b3e9f19cce4c5b62d5b0887fcd028b0&scene=21#wechat_redirect)  
**

**[Spring Boot + H2 JNDI 注入漏洞复现分析](http://mp.weixin.qq.com/s?__biz=MjM5MTAwNzUzNQ==&mid=2650492993&idx=2&sn=0b27352db1d34ba1b077ae397d1f08a5&chksm=beb3e60189c46f17b60ee16fd1460679986064211caaeace1e0f6fbfde2a629b17aa43434fe6&scene=21#wechat_redirect)  
**

![](https://mmbiz.qpic.cn/mmbiz_png/dyDu14T9ZVAiaRR2BruXhgDFianSKOoyx14RX8gIW1cu1CdOsY6bqT2a8E2T4QLM5My3WR8cdpJtNak6yH54Q45A/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/nTljOhrUdlVMDtusKZ8CjWnMVoiaxxicAS1E7h3CG7OPHGDl3vN6eYBgs1Udz9HqSYfEmLO2HH6AlGJTLdBGrLow/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_gif/ibJWc0l6oDVsYiaBFJFcicqfzH4coqQicXgUdmXkm7jN1ZLjibDSGYoX7x09r8D5doW2OEUQRictauEvrYJ9aOgQcvTw/640?wx_fmt=gif)