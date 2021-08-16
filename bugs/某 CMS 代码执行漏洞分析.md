> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5_HxHEFrCxOCagGOQPOCDw)

**本文首发于****奇安信攻防社区**  

**社区有奖征稿**

· 基础稿费、额外激励、推荐作者、连载均有奖励，年度投稿 top3 还有神秘大奖！

· 将稿件提交至奇安信攻防社区（点击底部 阅读原文 ，加入社区）

[点击链接了解征稿详情](https://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489051&idx=1&sn=0f4d1ba03debd5bbe4d7da69bc78f4f8&scene=21#wechat_redirect)

**0x1 前言**
==========

对 opensns 的一次代码审计，涉及到一些 tp 框架的方法利用, 以及对 tp 框架进行审计时经常忽略的点。

**0x2 漏洞分析**
============

官网下载源码：http://os.opensns.cn/product/index/download  
解压 打开

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8b6PnlBxS3CDOye7fCelSYfvllqdc6OLoiaxsOicswzJm3xIargyeYCZvw/640?wx_fmt=png)

很典型的一个使用 tp 框架的 cms，控制器都在 application 目录下  
然后粗略的看了一下所有能直接访问的控制器，没发现有明显漏洞的地方，但是发现了个比较可疑的方法。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8byAN2Z1Iq11EMibc6EjKWZ37p4ibXwnTcJPaeZDo4ibHibmmOdu2yhPv8BA/640?wx_fmt=png)

在 Weibo/ShareController 控制器中有一个 shareBox 方法 其中获取了 query 参数 然后 url 解码 在 parse_str 将 $query 解析成数组，然后 assign 成模板变量 最后 display 模板。这里并没有对获取的参数进行操作，那就可能在模板里面对参数进行操作了，去看下模板内容

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bRFPcentN6LhrKianzmrhGF0yd2RCe2EqUPxMIc2V2GI7sYd6mkKgL7w/640?wx_fmt=png)

文件位于 Weibo/View/default/Widget/share/sharebox.html  
看到用了 {:W()} 这种写法  
W 方法位于 Thinkphp/common/function.php

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bGvgFffTscYPfhNrvNqA7xTrDOWM1E0hOv4OMpUw37mI55rgWmXfdog/640?wx_fmt=png)

备注解释是渲染输出 调用了 R 方法 继续看一下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bicLicepMBHJkcicLFgiabVHHBQfxZj9X7ibuLuibnCvxBTlZgMPLszk1UQow/640?wx_fmt=png)

远程调用控制器的方法  
{:W(‘Weibo/Share/fetchShare’,array(‘param’=>$parse_array))}  
那这行代码就是调用 fetchShare 方法，参数也就是之前获取的 $query 解析成得数组  
那去看一下 fetchShare 方法, 位于 / Weibo/Widget/ShareWidget.class.php

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bschj9DD4N5ybiaQoMpy42WMVVmO6ZwUdLJbO2LCxCpDic0Qs75ty0c5Q/640?wx_fmt=png)

接着调用了 assginFetch 方法，我们看下 D 方法

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8b4HlJaMBnQscKPGAEVIaG2xlfibCA6CStB5oZbEO2akFH2FibxiblV8ezw/640?wx_fmt=png)

实例化模型类，那就是在 Weibo/Model/ShareMode.class.php，然后又调用了 getInfo 方法

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8b2IeWawy2ia9DT2NV30gRZHSuwRXs3Ws08I1jiaiaq1CrkHGlOia5NGciaaw/640?wx_fmt=png)

这里又调用了 D 方法，并且调用了有一个参数的方法。根据上面的了解，D 方法可以实例化 Model 类，那可利用的范围就变大了，去找一下可以利用的方法，只要满足两个条件。  
1. 为 Model 类  
2. 方法只能有一个传入的参数

看下 tp 框架本身自带的 Model 类。/ThinkPHP/Library/Think/Model.class.php

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bZojPdIwiaw3Bhzf9s1pGh5vibqMOHsgyXEyVMGQ79ylI4TDsuOGD4Bqg/640?wx_fmt=png)

找到一个可以实现 sql 注入的一个方法。但是我们可以尝试去寻找能实现代码执行的方法。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8byDgwqc47F3Zm7XciaEE3pVjDy84BbiawBZf2rBL9icnfZhdAG346eEbcg/640?wx_fmt=png)

同文件下有个_validationFieldItem 方法里面有 call_user_func_array 方法，如果能调用这个方法 并且两个参数都可控 那么就能实现代码执行。根据现在已知的条件，还不能利用 可以先记录一下。

找到一个可以利用的类，这是 cms 自己写的类。/Application/Common/Model/ScheduleModel.class.php

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bEhmlRPWePUeRE1KVWU4NpnnnJkVm3kkTGK4ibSsf2m6ZwaAwCC6njmA/640?wx_fmt=png)

一个参数，为 Model，满足这两个条件，然后看到又调用了 D 方法来实例化 Model 类，但是调用的方法为两个参数，结合上面找到的_validationFieldItem 方法。按照流程构造 poc，就能实现代码执行。

梳理一下漏洞触发流程：  
1.ShareController.shareBox->  
2.ShareWidget. fetchShare->  
3.ShareWidget.assginFetch->  
4.ShareModel.getInfo(这里控制 D 方法生成 ScheduleModel 类，并调用传入一个参数的方法)->  
5.ScheduleModel.runSchedule(这里控制 D 方法生成 Model 类，并调用传入两个参数的方法)->  
6.Model._validationFieldItem

**0x3 漏洞验证**
============

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bvwp2wna5SyXw35HX6WIdtrUJPGicH1icEJjEYibyBBmF5oxrIX4EzyMxQ/640?wx_fmt=png)

**0x4 总结**
==========

涉及到 tp 框架一些方法的调用应该还有其他的调用链，或者还能使用魔术方法来实现利用大家有兴趣可以找一找顺便可以熟悉下 tp 框架。

END

  

【版权说明】本作品著作权归 moonv 所有，授权补天漏洞响应平台独家享有信息网络传播权，任何第三方未经授权，不得转载。

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE50ibBLqIsKDichl7HIUWuQ8bG6v5PsrK5hEIaiafHbHiatePCfiaHDSCakVt95kAwlBhOSEwCGrPZWHaA/640?wx_fmt=jpeg)

**moonv**

  

一个神秘且优秀的补天白帽子

**敲黑****板！转发≠学会，课代表给你们划重点了**

**复习列表**

  

  

  

  

  

[出浅入深玩转 SQL 注入](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490316&idx=1&sn=654d98486e86620a496f2a1f4eed1c62&chksm=eafa5340dd8dda566a38633df43fe2393a628965350555a96253d6788975338b604006a694a7&scene=21#wechat_redirect)

  

[开源 USB 协议栈漏洞挖掘](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490303&idx=2&sn=4759723bcbdc6c074c224691d6a6df2e&chksm=eafa52b3dd8ddba547c95a119d6875516f0d978ea4fccf74e7b7ccb1d3c62b63be0a5f5b4592&scene=21#wechat_redirect)[](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489781&idx=1&sn=a2d0ccd466dfa95067f223c8318a316d&chksm=eafa50b9dd8dd9af45ef4fcf23074aeecc196dc72b3ff447282a9e6ea9904dcc08fe72430d30&scene=21#wechat_redirect)

  

[某 AOSP 框架层提权漏洞分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490251&idx=1&sn=6f3832784fc73563b8dac2f0fb2ea4b0&chksm=eafa5287dd8ddb918869107baab619d5a54a6d0a6f8c4626bcd3a980936d4ae05b02c2742c02&scene=21#wechat_redirect)

  

[记一次自动化渗透测试的学习研究](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490231&idx=1&sn=66e61ab5545a09d14b0f0a754611f53a&chksm=eafa52fbdd8ddbedcde5bc475ce00ce83a303f24e4729075edcf7d991af7df3fc80ae61a2623&scene=21#wechat_redirect)

  

[某内容管理系统 RCE 漏洞分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490180&idx=1&sn=14e85afb3ea561074d78d883330c6994&chksm=eafa52c8dd8ddbde157946b05bba4074c298e2d5aebf4437344e8d5c09c1ffe69da9cfe88efe&scene=21#wechat_redirect)  

  

[某邮件系统后台管理员任意登录分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490009&idx=1&sn=ea120cc287ad0735237a831a7b4847cb&chksm=eafa5195dd8dd8837c3d39040368110d77d2b30f0251fa2f7e4d5e656779249b1d30fce329a0&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6D8InhXuGX2q6Cbw7zhMJLFcmlcnz38EApnEkFiaISicklcwbo3gnI17t54PqyYOE8LV4yczIfjdqw/640?wx_fmt=png)  

  

分享、点赞、在看，一键三连，yyds。

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucChYUNicUaqntiamEgZ1ZJYzLRasq5S6zvgt10NKsVZhejol3iakHl3ItlFWYc8ZAkDa2lzDc5SHxmqjw/640?wx_fmt=gif)

  

点击阅读原文，加入社区，获取更多技术干货！