> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/V0dNaTCvkAHcvyt6QKJAFg)

**本文首发于****奇安信攻防社区**  

**社区有奖征稿**

· 基础稿费、额外激励、推荐作者、连载均有奖励，年度投稿 top3 还有神秘大奖！

· 将稿件提交至奇安信攻防社区（点击底部 阅读原文 ，加入社区）

[点击链接了解征稿详情](https://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489051&idx=1&sn=0f4d1ba03debd5bbe4d7da69bc78f4f8&scene=21#wechat_redirect)

**背景**
------

之前，在某鱼上钩买了一个苹果（原装）无线充电器，真的是出血、割肉了。突然萌生出一个想法为什么这个充电器那么贵，我们难道自己不能做出来吗？然后，根据一般的硬件研究过程——拆。想要弄清楚原理一定要，“打破沙锅问到底”。下面便是拆开后这个充电器的内在：  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5hACSzh3vZSRdN8FibnVDN1fnw4yB2p4ibJryCrLuYqe63WVkvJibXUEoA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5NwZ3Uq6PlT5niaUUHfSLmgM8Aic2AY26pALZrLX1RcObDyNyeniap6iaDg/640?wx_fmt=jpeg)  
看到了他复杂的 “内在”——双层 PCB 加上传输线圈（with 反射面）。线圈不重要，这是我们都了解这个必然具有的部分，主要看看天线回溯到信号输入这中间模块的功能分区。  
看看这个部分，通过 PCB 板子上写着的信息可以了解到有 GND、CC2、CC1、D+、D-、VIN。其中，cc2 端口为空，这个部分我们无须过多关注（后期我们用的接口类型为 USB）。  
再来是快充协议处理模块，三极管放大器，MCU（对输出信号进行编码），还有 4 个白色的谐振电容。如各位大佬想尝试破解快充协议，可以在 MCU、编码器上做文章。本文不会考虑这些，拆解主要是为了了解电路层面一个无线充电器必需的部分。为 “构造基础电路” 部分奠定基础。  
不妨示波器观察一下无线充电器的输出电压的时域波形：  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5OeJ2HGsHC1k5oSKziaQ0hcGupYE7GecS1FU5sicDu0r0ibY9RdqvTd6dA/640?wx_fmt=jpeg)  
总结一下，我们要向复刻一个无线充电器都需要加入什么部分：

1.  发射端：  
    1） 电源类型（蓄电池、干电池、220V 交流电等）  
    2） 传输线圈  
    3） 谐振回路（匹配电路）
    
2.  接收端：  
    1） 输入端口（了解输入端口相关协议，明确端口电压作用）  
    2） 输入电压处理模块（交流变直流）  
    3） 稳压器模块（匹配电路）  
    4） 认证电压构造、处理模块  
    5） 输出接口
    

**相关参数获取**
----------

这一步是比较重要的，光有接口，没有这些接口参数，便无法完成复刻。那么我们都需要什么基本参数呢？这要看苹果传输线的结构和 USB 接口的连接方式。（如下图）  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5YIfX0ByUgmFqbaqYSPyo75som2FicPoLlibpH1Z5cC3zAHU9WCwgga9Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5y3YFxqBkeibVib9ibmCHJGhzRsVsOB842gYpqtEbeDcXXwEiaDcPqhK0sQ/640?wx_fmt=jpeg)  
这两张图便可以很清楚的说明一些。先讲 USB 端口部分，将充电线比较完好的拆开，金属外壳去除之后，是胶质包裹在电路上起到保护作用。我这里是通过偏口钳慢慢剪开，防止灼烧方式损害原本电路。呈现出右图的样子。看到这里一些熟悉硬件接口的师傅们便可以得出 usb 端口这侧不存在认证过程，就是电源的直联。  
不放心或者不熟悉的小伙伴把这个包开的 usb 接口插回到原有充电器上用示波器万用表面可以验证。（这一点很关键，大大降低了我们复刻的难度值）  
再来是讲中间传输线，苹果的数据线与安卓线不同，中间有六股线（如左图），分别是红线 Vcc、白线 D-、绿线 D + 和三股裸线（GND）。左图里面呈现的 4 股线是我将他们穿绕在一起上锡的样子，一根裸线过于脆弱。  
其实，三根裸线的类型判定有两种验证方法：1、usb 接口端只有 4 种线，那么就意味着三根裸线必然是和右图的黑线等电势。2、最直接的 usb 上电，拿万用表一测便可。  
说到这里我们已经成功了 1/4, 下面就是电路的参数测定。最重要的 Vcc 的大小，这个看一下我们的原装充电器上的基本参数便可以得知——5V。（示波器验证)  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5ssfOwQ5ECWibONxHmBDW0MAar7vHrg1siccI7SNia75JFYnA5CfQDXOFA/640?wx_fmt=jpeg)

D-、D + 信号线的认证初始电压大小呢？两个都是 + 2.72V。（如下图）  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5A8sLicEzQicUXSoaQ2Z4oPtX14ffEku4mm0gamOsnqEOdRBxs8eB6EJQ/640?wx_fmt=jpeg)

到这里，便已经成功了一半了，有个这几组电压，有线充电的目标便可以完成，但我们是对自身有要求的怎么无线充电呢？想法就是通过电路的方式构造这几个电路。

**构造基本电路**
----------

### **理论部分**

首先，应假设这个电路的应用环境，要是还需要插在插销上便逊色不少，为此我们这个电路设计的输入额定电压为 5V～24V。保证我们的手机不仅可以通过干电池、铅蓄电池等一系列电池充电，还可以用车载电源充电 ······  
讲完输入一侧，再来讨论一下输出部分。这里的处理方案是将设计的电源处理模块连接 lightening 充电头给手机充电（类似无线充电卡贴）。  
这里要声明一点我这篇文章并不是要破解苹果原装无线充电器输出电磁波编码，而是从底层电路层面来复现这个无线充电的原理。

先放一个整体电路的实物连接图吧!  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT54A7rcwc6FYsEgkibJGzTLicxE5aJhESxWGR0J8hMzGZkibdxxGiaIqt3ow/640?wx_fmt=jpeg)

整体电路设计概念图：（其中电源是交流电，后续我会继续出直流转交流电路知识）  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT55HTFTm1iaThb6tu0BLFibQfJVssO2s97c7Lgut0J91C3wic3nkXrfSSXg/640?wx_fmt=jpeg)

稳压与认证信号电压处理部分：  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5FJu7OfR8V6WqE3hwOBHa32vgibpeKp6Q0miaRFy3ttdgvlAYMrg4uY1w/640?wx_fmt=jpeg)

### **实际部分**

需要一定的手工制作，手里功夫欠缺的师傅们，请使用钞能力或用机器制作线圈。

1.  loop antenna  
    这里的 loop 是用 AWG 24 的铜丝，在上图中摩丝瓶（大概直径 2.5cm）绕 20 圈。这里各位师傅可以比较随意，我仅仅是为了配合手里有的电容大小才这样选择。
    
2.  LC 谐振回路  
    这个部分是能量传输的关键部分，输出端和接收段的谐振频点要一致参考公式 f_0 = 1/(2pi* sqrt(LC))。阻抗匹配保证能量不存在反射，以确保充电效率。
    
3.  倍压整流模块  
    因为从接收线圈中得到的是交流电，是无法直接输入到手机当中的。为此要通过整流电路将将交流电转换成直流，再通过 5v 稳压器，输出。  
    注：这里使用三倍压整流器的原因是为了匹配负载以及接收线圈的损耗（手工线圈由于形状，loop 的波瓣图不是理想的）。加上设计的谐振频点对应的能量传输波长远大于环天线直径，天线方向性 D 进一步降低。  
    为此，在设计传输线圈是既要考虑尺寸，同时也不能忽略本身电子参数的选择。**一句话表述这里的设计准则，让传输电磁波波长与环天性尺寸差不多。**
    
4.  认证电压构造  
    所谓认证电压，便是 D+、D - 两条线的电压大小。通过实验证明当这两条线的电压值为 0，即不构造认证电压，是充不上电的。这一点是基于 lightening 线充电头中的电路逻辑处理得出。D-、D + 信号用作内部芯片的逻辑判断依据（会在后续 bypass 部分，进一步说明）。
    
5.  tips 分享  
    关于传输天线的一些 tips（天线大佬可忽略）  
    loop 天线的应用主要是尺寸比较小，同时在实际的物理充电情况下，可以手动控制两个线圈磁感线的发出接收比例，但是其物理的拓扑结构对波瓣图的影响比较大。为此，如果真的想要复刻出可以使用的无线充电器，各位还是乖乖地用机器来 “盘” 线圈。  
    这里的传输天线漆包线盘成的，在连接电路时务必要用打火机或者砂纸去除外面绝缘物质，否则线圈无法传输电力。  
    输出接口我是通过 4 个芯片夹，连接苹果 4 种数据线构成的。（如下图）  
    ![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5TxqTGZuPYRMlm8ibIGrmkqVPEyqERfV7oy3SxyVnDNMYyib40RXPJMIw/640?wx_fmt=jpeg)
    

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5GzvUIIeyliavJXYQLjGL2Z8ia6bbam7ySTVIYr5qpsBRWzjeJrqnjibRg/640?wx_fmt=jpeg)

**绕过 bypass 充电认证**
------------------

通过实验发现，这两个信号线承载的电压大小不仅仅具有认证作用，还决定着充电的效率。当这两条线的电压值小于 1.5V 或大于 4.1V 时，都可以通过 iPhone 5s、iPhone 8 的充电认证，但是充电效率与这个信号电压大小呈正相关关系。并且随着负载（充电手机）的充电量也接近 100%，这两路的电压将越高。  
实验中我们设定的 D-、D + 信号为 2.73V，在充 iPhone 8 时冲到 80% 便不再继续变化。为此后续我们将解析 lightening 充电头内部的芯片，理解 D-、D + 信号的具体作用。目前还停留在定性阶段，为各位提供一种 “自底向上” 的研究思路。

**实验电路图**
---------

5V 稳压器模块测试结果（PCB）  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5dcQ6OtGXn4vLcZ5TrtDkGlYm0SW9lmISiajZKMxMCOQC4W8QwWLZyCg/640?wx_fmt=jpeg)

无线充电基本电路（发射 & 接收）：  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5n2K1YZpxJCxlo9AZl243jP4Vt5ZPDTwhHOkAicyexUCb9UpQMvJQwcQ/640?wx_fmt=png)  
其中，L_1、L_2 为线圈，对应 c 的大小有交流电压源频率决定。R_L 为负载，之间是一个三倍压整流电路。

**最终实验**
--------

详情请见实验视频，分别进行了 iPhone 5s 和 iPhone 8 的测试，都成功充上电。  
iPhone 5s 测试视频链接（比较短）：  
https://www.bilibili.com/video/BV1TX4y1P7oA/  
iPhone 8 测试视频链接：  
https://www.bilibili.com/video/BV1k5411T7pf/

由于本次测试仅仅是个小雏形，在设计之初并没有考虑快充等一些列协议的应用，单从底层接口原理、电磁波特性层面来复刻, 为各位奋斗在破解无线充电奥秘的师傅们提供一些来自底层电路的启发。  
本电路中应用的传输线圈是我手工自制的，在性能上只起到了一个传输的基本要求，欢迎感兴趣的大佬们可以从中获得启发，进而改进这个电路，达成自己的攻击效果。  
关于上述防止其他品牌手机串用充电这个因素，仅需给我们自己的这个传输器在能量电路到输出线圈和接收线圈到输出电路之间加上一个特殊的编码器、解码器模块（PT2272 解码器、PT2262 编码器 & 多路开关），便可保护自己的品牌独立性了 hhhhh！

END

  

【版权说明】本作品著作权归 Mori 所有，授权补天漏洞响应平台独家享有信息网络传播权，任何第三方未经授权，不得转载。

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE4VLqd11icRfbVjdB7IzfcT5GFyUQWE4ic34RF05ajTUp9iavYLrdgcR3sPTcBibUzwgibaaObJKjebCOQ/640?wx_fmt=jpeg)

**Mori**

  

安全领域的小学生，Geek 精神的推崇者

**敲黑****板！转发≠学会，课代表给你们划重点了**

**复习列表**

  

  

  

  

  

[](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489568&idx=1&sn=56beddb5ef58d9556d75bbd8dd146dd2&chksm=eafa506cdd8dd97a9420b312770c8ea1bdeb0394ce38ef54834e60e91c0b404f934ac494ca3c&scene=21#wechat_redirect)[出浅入深玩转 SQL 注入](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490316&idx=1&sn=654d98486e86620a496f2a1f4eed1c62&chksm=eafa5340dd8dda566a38633df43fe2393a628965350555a96253d6788975338b604006a694a7&scene=21#wechat_redirect)

  

[某邮件系统后台管理员任意登录分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490009&idx=1&sn=ea120cc287ad0735237a831a7b4847cb&chksm=eafa5195dd8dd8837c3d39040368110d77d2b30f0251fa2f7e4d5e656779249b1d30fce329a0&scene=21#wechat_redirect)[](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489781&idx=1&sn=a2d0ccd466dfa95067f223c8318a316d&chksm=eafa50b9dd8dd9af45ef4fcf23074aeecc196dc72b3ff447282a9e6ea9904dcc08fe72430d30&scene=21#wechat_redirect)

  

[开源 USB 协议栈漏洞挖掘](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490303&idx=2&sn=4759723bcbdc6c074c224691d6a6df2e&chksm=eafa52b3dd8ddba547c95a119d6875516f0d978ea4fccf74e7b7ccb1d3c62b63be0a5f5b4592&scene=21#wechat_redirect)

  

[某 AOSP 框架层提权漏洞分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490251&idx=1&sn=6f3832784fc73563b8dac2f0fb2ea4b0&chksm=eafa5287dd8ddb918869107baab619d5a54a6d0a6f8c4626bcd3a980936d4ae05b02c2742c02&scene=21#wechat_redirect)

  

[记一次自动化渗透测试的学习研究](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490231&idx=1&sn=66e61ab5545a09d14b0f0a754611f53a&chksm=eafa52fbdd8ddbedcde5bc475ce00ce83a303f24e4729075edcf7d991af7df3fc80ae61a2623&scene=21#wechat_redirect)

  

[某内容管理系统 RCE 漏洞分析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247490180&idx=1&sn=14e85afb3ea561074d78d883330c6994&chksm=eafa52c8dd8ddbde157946b05bba4074c298e2d5aebf4437344e8d5c09c1ffe69da9cfe88efe&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6D8InhXuGX2q6Cbw7zhMJLFcmlcnz38EApnEkFiaISicklcwbo3gnI17t54PqyYOE8LV4yczIfjdqw/640?wx_fmt=png)  

  

分享、点赞、在看，一键三连，yyds。

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucChYUNicUaqntiamEgZ1ZJYzLRasq5S6zvgt10NKsVZhejol3iakHl3ItlFWYc8ZAkDa2lzDc5SHxmqjw/640?wx_fmt=gif)

  

点击阅读原文，加入社区，获取更多技术干货！