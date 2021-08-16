> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ZgA6PELbMUcZ3vRpr4oRlw)

**1. 简介**

在安全厂商日趋成熟的背景下，编写免杀马的难度和成本日益增长。好用新兴的开源项目在短时间内就被分析并加入特征库。笔者调研了部分开源项目，其中也有项目做了类似的分析 [1]，目前能够免杀的项目初步统计，其特征一是 star 数不过千，二是发布时间不会很长。尽管以上开源测试项目已经无法免杀，也有两种可以发展的方向，一个是学习其思想，自己实现并去特征免杀；二是改造原有项目，自己查特征、去特征，经过测试也能达到免杀。  

免杀方法和思路很多，但据笔者观察，目前免杀分为两大流派。一是二进制流，利用汇编配合上 C++，调用系统底层函数进内核的方式免杀。杀软如果直接在用户态检测其行为特征会比较困难。二是新工具新项目、小众工具流，其主要思想是寻找反病毒厂商未覆盖的方法和工具，一个是寻找新的语言工具和项目，跟厂商比速度。另一个是偏僻语言，用户量小，厂商一直并未发现或者工作重心不在上面。举个例子，可以用各种语言二次编译，配合上一些语言特性如 python 反序列化达成免杀。通过现有工具的组合有效提高复杂度，在反病毒人员的盲区里进行。两种各有优劣，后续若有文章会测试更多的免杀用例。这里推荐一个静态免杀学习项目 [2] 和 52pojie 上的免杀项目, 利用了大部分后文中提到的免杀技术 [3]。

2. 杀毒软件检测方法

以下的内容多为前人总结。因为云查杀本质上也是基于特征查杀，顾不单独列出。

2.1 特征码检测

  

对文件或内存中存在的特征做检测，一般的方法是做模糊哈希或者机器学习跑模型，优点是准确度高，缺点是对未知木马缺乏检测能力。所以目前依赖厂商的更新，厂商做的更新及时能有效提高杀软的防护水平。目前一些杀软对相似的病毒有一定的检测能力，猜测是基于模糊哈希做的。部分杀软同样对于加壳也有检测能力，对于不同的厂家有不同的策略，有些会对文件进行标记，而某数字会直接告警。

2.1.1 关联检测

  

检测的特征不仅仅是恶意 payload 的特征，也可能是一组关联的代码，把一组关联信息作为特征。比如在使用加载器加载 shellcode 时，需要开辟内存，将 shellcode 加载进内存，最后执行内存区域 shellcode。这些步骤就被反病毒人员提取出来作为特征，在调用了一组开辟内存的函数比如 virtualAlloc 之后对该内存使用 virtualProtect 来更改标示位为可执行并且对该内存进行调用就会触发报毒。以上只是一个简单的例子，具体情况具体分析，部分厂商对其进行了扩展，所以现在使用另外几个函数进行调用也无法免杀。不过其本质还是黑名单，还存在没有被覆盖到的漏网之鱼。

2.2 行为检测

  

行为检测通过 hook 关键 api，以及对各个高危的文件、组件做监控防止恶意程序对系统修改。只要恶意程序对注册表、启动项、系统文件等做操作就会触发告警。最后，行为检测也被应用到了沙箱做为动态检测，对于避免沙箱检测的办法有如下几个：

*   延时，部分沙箱存在运行时间限制
    
*   沙箱检测，对诸如硬盘容量、内存、虚拟机特征做检测
    
*   部分沙箱会对文件重命名，可以检测自身文件名是否被更改
    

2.3 小结

以上是对杀软检测做的一个小结，目前学术界对恶意代码的检测集中在机器学习上，已经有部分杀软已经应用落地了，如微软。对杀软检测手法更多的了解有助于我们写免杀马。  

3. 绕过技术

目前，随着 cs 的流行，越来越多的人使用 cs 的 shellcode，而放弃了自己开发编写的木马，或者使用改造的 msf 马。本篇文章也会以 shellcode 加载器作为例子。后续文章将会涉及更深入的内容。

3.1 经典技术

  

经典免杀技术如下，由于篇幅所限，本篇只含部分免杀技术。

*   特征码修改
    
*   花指令免杀
    
*   加壳免杀
    
*   内存免杀
    
*   二次编译
    
*   分离免杀
    
*   资源修改
    
*   白名单免杀
    

3.2 修改特征

  

一个加载器存在两个明显的特征，一个是 shellcode 和硬编码字符串。我们需要消除这些特征，比较方便的方案，使用 base64 等对上述特征进行编码，最好使用多种编码手段。对于 shellcode，使用 base64 并不安全，所以更安全的方案是加密，一个简单的异或加密就能消除 shellcode 的特征。第二个是加载器的关联特征也需要消除，对于代码中出现连续调用的 virtualAlloc，virtualProtect 进行插入花指令，通过加入无意义的代码干扰反病毒引擎。

笔者的一点想法，进一步混淆源代码，在不加壳的情况下稍微增加静态分析难度。也有论文提出可以使用 ROP 来提高代码的分析难度，因为现存的代码分析引擎对间接跳转和调用的支持存在瑕疵，复杂逻辑的代码更需要人工分析 [12]。

3.3 内存免杀  

  

shellcode 直接加载进内存，避免文件落地，可以绕过文件扫描。但是针对内存的扫描还需对 shellcode 特征做隐藏处理。对 windows 来说，新下载的文件和从外部来的文件，都会被 windows 打上标记，会被优先重点扫描。而无文件落地可以规避这一策略。同时申请内存的时候采用渐进式申请，申请一块可读写内存，再在运行改为可执行。最后，在执行时也要执行分离免杀的策略。

3.4 修改资源  

  

杀软在检测程序的时候会对诸如文件的描述、版本号、创建日期作为特征检测 [7]。可用 restorator 对目标修改资源文件。

3.5 隐藏 IAT

每调用一个系统函数就会在导入表中存在，这对于反病毒人员是个很好的特征，直接通过检测导入表中有没有调用可疑函数。这里就需要隐藏我们的导入函数。一个比较通用的办法是直接通过 getProcessAddress 函数获取所需要函数的地址。知道地址也就能直接调用，这样整个程序内除了 getProcessAddress 其他函数都不会出现在 IAT 表中。尽管这样已经能绕过上面的检测，但还有种更保险的做法，用汇编从 Teb 里找到 kernel32.dll 的地址，再从其导出表中获取所需系统函数。  

![](https://mmbiz.qpic.cn/mmbiz_png/PvqUNuD3SmTsiat9za8kB4TZR4iaxcJuPzSMlr6jrQpL1vmLjPiarVE4r3qwgg0xyPJmspyiaMUOzg7Kiav48y1n5TA/640?wx_fmt=png)

出自（https://www.52pojie.cn/thread-1360548-1-1.html）

3.6 分离免杀

整个 shellcode 加载器分为两个部分，分离下载 shellcode 和执行。加载器处在 stage0 阶段，其作用除了加载大马外并无其他作用。但是直接执行大马会被检测到，所以需要用到分离免杀。

![](https://mmbiz.qpic.cn/mmbiz_png/PvqUNuD3SmTsiat9za8kB4TZR4iaxcJuPzKUg8w6gZbpArGbSML5oC2GspiaEPFhkdA17ViaP1hL6fsZ9l37X4iaGLQ/640?wx_fmt=png)

出自（https://www.anquanke.com/post/id/190354）

通常杀软只检测一个进程的行为，所以如果存在两个恶意进程通过进程间通信就能逃过检测、达到免杀。

分离免杀的方法多种多样，既可以用 windows 的管道 [4][6]，也可以用 socket 通信 [5]。

3.7 二次编译免杀

像 msf 或者 cs 的 shellcode 在各个厂商里都盯的比较严，对于这些 shellcode 已经提取好特征只要使用就会被检测出。所以会使用各种编码器进行免杀。编码器有很多种，这里仅推荐 msf 的 shikata_ga_nai，是一种多态编码器，每次生成的 payload 都不一样。  

3.7.1 其他语言编译免杀

   

因为各种语言特性不同，对于不同语言编写的加载器厂商不一定第一时间跟进，导致了一段时间内可以绕过。在 2020 年初的时候，使用 python 作为加载器 [11] 免杀一阵，现在针对这类加载器逐渐严格，导致直接加载报毒，需要更多的混淆还改进。在 2020 年 5 月，奇安信红队出过一篇文章，利用 python 反序列化来加载 python 加载器 [8]，目前截止本文测试 2021 年 1 月已经无法使用了，是个比较好的思路。

3.8 系统函数白名单免杀 - uuid 方式  

  

Gamma 实验室在 2021 年 2 月 3 号发布了一篇微信公众号的文章 [9]，分析了 Check Point Research 研究的 apt 攻击的文章。其中的亮点在内存中 shellcode 的编码方式和调用都没有使用传统编码和调用的方式，利用了系统函数的特性，这次的例子是 uuid。使用的是系统给 UuidFromStringA 函数将 payload 的 uuid 数组转化为 shellcode 加载进内存，其特点就是程序中存在大量硬编码的 uuid。另一个，调用使用的是 EnumSystemLocalesA 函数，它的第一个参数是回调函数指针，也就意味着参数一只要传入 shellcode 首地址就会执行恶意命令。现已被杀，但是这里给出一个比较重要的思路，还有其他可以利用的 Windows 系统函数可以利用。另外现已经有项目实现了使用调用 guid 来进行免杀。

3.9 某数字公布的 stage uri 检测  

  

因为 cs 密钥都是硬编码的，被逆向出来后，只要使用 stage 分阶段的方式加载 cs，其流量都会被解密并能检测其特征 [10]。对抗的方式，二次打包改密钥，或更改 cs 的配置文件使得关闭 stage。同时使用 stageless，也得更换自己的 dll，cs 的 beacon.dll 同样在检测列表中。其实用改造过的 msf 马也不错，这也就是上文提到的过的开源项目，目前这个项目已经被某数字检测了，所以需要对项目进行改造。

4. 总结

以上总结了主流的免杀方式，后文的免杀就是以上技术的混合使用。本文还未涉及到诸如加壳，dll 以及使用 powershell 免杀等，这些会在之后的文章中提出。  

5. 参考文献

[1]     https://github.com/TideSec/BypassAntiVirus

[2]     https://github.com/Rvn0xsy/BadCode

[3]     https://www.52pojie.cn/thread-1360548-1-1.html

[4]     https://payloads.online/archivers/2019-11-10/4

[5]     https://payloads.online/archivers/2019-11-10/5

[6]     https://www.anquanke.com/post/id/190354

[7]     https://cloud.tencent.com/developer/article/1512006

[8]     [https://mp.weixin.qq.com/s/gZ28MvCPTQbTAVtQjO7T8w](https://mp.weixin.qq.com/s?__biz=MzU2NTc2MjAyNg==&mid=2247483980&idx=1&sn=e97ffdb7b184852ca6c6f2ac1864d851&scene=21#wechat_redirect)

[9]     [https://mp.weixin.qq.com/s/1DvYNDiZc2iV1pXEn7GZEA](https://mp.weixin.qq.com/s?__biz=Mzg2NjQ2NzU3Ng==&mid=2247487017&idx=1&sn=fddcd7fd3d5176f1cafe9bdffb45b13f&scene=21#wechat_redirect)

[10]   [https://mp.weixin.qq.com/s/fhcTTWV4Ddz4h9KxHVRcnw](https://mp.weixin.qq.com/s?__biz=MzU2NTc2MjAyNg==&mid=2247484689&idx=1&sn=8cf9c031f3d926c155ee5c018941b416&scene=21#wechat_redirect)

[11]   https://www.secpulse.com/archives/151899.html

[12]   https://arxiv.org/pdf/2012.06658.pdf

作者：icecream, 现在 b 站安全负责红蓝对抗蓝军方向工作

**推荐阅读**[**![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyIDG0WicDG27ztM2s7iaVSWKiaPdxYic8tYjCatQzf9FicdZiar5r7f7OgcbY4jFaTTQ3HibkFZIWEzrsGg/640?wx_fmt=png)**](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247494261&idx=1&sn=8e008b655fe25c2304edb0cfa1cdf264&chksm=9acd3aeaadbab3fcf857620968368e8fd39e1bf3f9767abb5b370c72558fb2f9ac2f5e234734&scene=21#wechat_redirect)  

**点击关注乌雲安全**

公众号

**觉得不错点个 **“赞”**、“在看”，支持下小编****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**