> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OyXONTmjDJME8x6o2-hXOA)

  

  

    在内网渗透过程中常常会碰到当前跳板机有 RDP 的连接记录，有些管理员会勾选保存密码，这个时候就可以通过 mimikatz 来获取明文凭证；但有些管理员就不会勾选保存密码，这个时候我们如何获取到 RDP 的连接凭证？

**内网渗透中如何获取到明文凭证‍**

**SharpRDPThief**

SharpRDPThief 是 RDPThief 的 C# 实现。它使用 EasyHook 将一个 DLL 注入 mstsc.exe，然后它会挂钩 CryptProtectMemory api 调用。hook 将从传递给 CryptProtectMemory 的地址中获取密码，然后通过 EasyHook 的 IPC 服务器将其发送到主进程。

目前这只是概念实现的证明，需要 RDPHook.dll 与 SharpRDPThief.exe 位于同一目录中。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIu0Au1wRhzeUhr8CDW96hOibeUicDoqIATfzOYAQKRhmUZILQBgNWdsQ2rDIqlvaVr4VhC7IibaDMdA/640?wx_fmt=png)

此时如果客户端使用了 mstsc 并输入了 user、pass ：（此时是未勾选保存凭据）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIu0Au1wRhzeUhr8CDW96hOiaXq2Xw1pajZougSg8Mc0Wibnj1B3UWqAEFcIR6F5yw72p2skkeZAQrg/640?wx_fmt=png)

SharpRDPThief  就会把 RDPhook.dll 注入到 mstsc 进程从而获取到 user、pass：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIu0Au1wRhzeUhr8CDW96hOaN9R3LfCkPlNNhyp9gBxfpNO1OfRatQ9mbEuRa0nxpS8picVjVwduPQ/640?wx_fmt=png)

**RDPThief**

RdpThief 本身是一个独立的 DLL，当它被注入到 mstsc.exe 进程中时，将执行 API 挂钩，提取明文凭据并将它们保存到文件中。

当在 Cobalt Strike 上加载 aggressor 脚本时，有三个命令可用：

```
rdpthief_enable – 启用新 mstsc.exe 进程的心跳检查并注入它们。
rdpthief_disable – 禁用新 mstsc.exe 的心跳检查，但不会卸载已加载的 DLL。
rdpthief_dump – 打印提取的凭据（如果有就会打印出来）
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIu0Au1wRhzeUhr8CDW96hOHhCCyH5cg2JNnPkibTHp8dU1HoOyCnlQtbzZ581uPEXFu7LsKUHUbIA/640?wx_fmt=png)

可能是我 CS 版本问题，把 rdpthief_dump 的结果复制到文本就能看到 ip、user、pass 了：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIu0Au1wRhzeUhr8CDW96hOoeoENDTulUBXpc5uyXcf8ftaOukg5DyvibIOkd5icpib3YYNhCvTHT0hQ/640?wx_fmt=png)

它实际把凭据保存到了 **%temp%\data.bin** 文件里：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIu0Au1wRhzeUhr8CDW96hOwGDzS40XFWqzCXewjiayiaBL1WwtVId2el3Bic61KwFMgkiamFq6wbgGFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIu0Au1wRhzeUhr8CDW96hObAOuRpUr1bu3QxGMW5dVIxDhdsoQwp18UXFBB8RediaUbYgnEPKel4w/640?wx_fmt=png)

实际情况下如果发现目标机器上有很多 mstsc 连接记录，此时我们就可以注入到 mstsc 进程，耐心等待一两天猎物上钩，可能会收获不少！当然这也靠很大部分的运气成分在里面，不说了，祝兄弟们好运！

* * *

参考文章：  

https://www.mdsec.co.uk/2019/11/rdpthief-extracting-clear-text-credentials-from-remote-desktop-clients/

https://github.com/0x09AL/RdpThief

https://github.com/passthehashbrowns/SharpRDPThief

****【往期推荐】****  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - Shellcode Loader 原理 (python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - 分离 + 混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附 PoC】CVE-2021-2109 | Weblogic Server 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)

[【漏洞分析 | 附 EXP】CVE-2021-21985 VMware vCenter Server 远程代码执行漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487906&idx=1&sn=e35998115108336f8b7c6679e16d1d0a&chksm=ea37eef8dd4067ee13470391ded0f1c8e269f01bcdee4273e9f57ca8924797447f72eb2656b2&scene=21#wechat_redirect)

[【CNVD-2021-30167 | 附 PoC】用友 NC BeanShell 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487897&idx=1&sn=6ab1eb2c83f164ff65084f8ba015ad60&chksm=ea37eec3dd4067d56adcb89a27478f7dbbb83b5077af14e108eca0c82168ae53ce4d1fbffabf&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)

一如既往的学习，一如既往的整理，一如即往的分享。![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“**如侵权请私聊公众号删文**”

公众号