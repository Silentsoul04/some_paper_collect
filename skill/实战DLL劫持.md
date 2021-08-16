> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/liv4S4rqQwiD0yecnNisNA)

 ![安全鸭](http://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNakoPXq9aPYuhOj3SrCc1eLXB37lY487tvRfKhZWGIojsRzJDDibZQEZ3ic0l8tg5GcMS6qgu4zzDhw/0?wx_fmt=png) ** 安全鸭 ** 回头下望人寰处，不见长安见尘雾 19篇原创内容   公众号

0x01 前言  

==========

最近看到cnvd上很多高危漏洞，都是dll劫持漏洞。为此我也结合以前了解的知识，学习并实战通过dll劫持对指定应用进行内存修改。

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibiciasic5Zxic1OkMmOuueZvay97Evaol54Ex5n3ULgn6kasMUHG1FIVOXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

0x02 思考
=======

2.1 为什么可以进行DLL劫持
----------------

这部分主要取决于windows系统dll加载机制。dll文件的加载顺序为：

(1)EXE所在目录；

(2)当前目录；

(3)系统目录；

(4)WINDOWS目录；

(5)环境变量 PATH 所包含的目录。

这就导致我们可以让我们自己编写的同名dll在正常dll之前进行进行加载。

2.2 为什么有的dll无法进行劫持
------------------

主要分三种情况：

1.dll采用绝对路径方式加载（这种方式极少）

2.windows新系统采用knowndlls方式保护的dll，可以通过以下路径查看被保护的dll是哪些

计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs

Windows10保护的dll如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibtkoFAq7Q8bWMqMj3sNUq7ne874rNTPv8vb65pibVxSiaicKhsticSbNdBQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3.非主程序直接调用的dll可能无法劫持。

  

2.3 劫持dll编写原理
-------------

1.获取被劫持dll的输出表

2.编写劫持dll，模拟和被劫持dll相同的输出函数，然后把对应函数的处理方式设置为转发到真正的dll函数地址

0x03 实战编写劫持DLL
==============

我们准备了一个windows应用来测试劫持：GDAE3.90.pro.exe,如下图:

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibHfTqSkdm4YgtIQY3ias3icxFyyniaSCBwaQzzIIB8LtJ46lzey14PXRicw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3.1 确定劫持dll目标
-------------

我们必须先知道这个应用在执行的时候加载了哪些dll，这里我们通过火绒剑的系统监控功能进行监控，打开后运行程序知道完全跑起来：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibdlpfOrQUlNiaxMLWshyU3R4mibhSrHhibn2vqGpFEib7SVSdjfNIibhs6ow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样我们就找到了所有应用跑起来时加载的dll文件。按从上往下的顺序，排除knowdlls记录的dll后，我们可以把其他的dll拷贝的应用的目录下，再重新监控一次加载的dll文件：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibQEtnCQ1WqQl1sdOZkk6H6FO2iapqNPEtEanyZQjUrnmqfSgiaZBE0p9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以看到wininet.dll和wldp.dll就可以是我们用来劫持的dll了。

3.2 编写劫持补丁
----------

我们需要2个工具，一个是微软的vs，一个是网上高手写好的用于劫持dll自动生成代码的工作，网上有很多，我比较推荐以下这个：

https://github.com/strivexjun/AheadLib-x86-x64

这里我以劫持wininet.dll进行举例：

用aheadlib打开wininet.dll，并生成一个劫持的源码：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibza5crIO4ajaicjss3G6J4Xf3niaCXHTlgTfFN0BzjSLsRhGaGfz5fiavg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

打开vs，选择新建一个动态链接库

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibO3u8G1gD5ickE0UWyx1icibgvdeTJpnic9MQsF2icAHpCbn7ia5ZeBdwhrhw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后删除几个自动生成的项目文件：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibKDfAz8rDaeLlNABhOCnXlAJRibibQAPFVrlgtAMnfFZHq1zrIGQGicSQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在把用aheadlib生成的wininet.cpp拷贝到项目跟目录下，并添加到vs的ide项目的源文件里面：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHiba8siaibPavUELSa4kgyhy0GlXhWJHJX24xpS4hH2Nr5rc6zrfWEuDKkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

找到源码中的以下部分，需要我们自行修改：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibajU435l8YamQclRpVpq9icKYY4GYePcJ7SBOGGW3zLodMnEPhSyYeVQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中宿主进程名改为我们运行的程序名称，上面的ThreadProc函数就为我们劫持成功后想自己干的事情了，比如通过内存修改主程序代码，比如加载shellcode上线主机等。这里我弹出一个消息框，我们代码修改后如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibEKnEviaJ4KxItsM5SHQrlWr6rowvKdRMIk1cvf5eI1CvlVfjK5VLiaMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这时候我们还需要对项目进行一些配置，否则无法编译通过：

1.配置属性--高级--字符集--改为“使用多字节字符集”

2.配置属性--C/C++--预编译头--改为“不使用预编译头”

3.配置属性--常规--目标文件名--改为“wininet.dll”

这时候就可以编译了，成功编译后把生成的wininet.dll文件拷贝到应用的目录下，再打开应用：

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNaOWVdnjVme8LSyavriczpHibBltgnSNmINF1BicUcJniciaAfhMFaOrP6ibHt1stD1gO09Q7H74hvJo79w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

成功进行劫持，弹出消息框。

 ![安全鸭](http://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNakoPXq9aPYuhOj3SrCc1eLXB37lY487tvRfKhZWGIojsRzJDDibZQEZ3ic0l8tg5GcMS6qgu4zzDhw/0?wx_fmt=png) ** 安全鸭 ** 回头下望人寰处，不见长安见尘雾 19篇原创内容   公众号