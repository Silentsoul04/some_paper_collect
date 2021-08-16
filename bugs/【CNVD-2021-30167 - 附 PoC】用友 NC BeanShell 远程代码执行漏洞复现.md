> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zBJl19bmXZg2kVsVNTIG6Q)

**![](https://mmbiz.qpic.cn/mmbiz_jpg/7D2JPvxqDTH23Qn1c9IEUuOiarb4EY5tcByibvUp1Im1t9ZQg6jQpTr0Fjv90Eq9eoAPGm8QNNictsWVwYgD82qZg/640?wx_fmt=jpeg)**

**0x01 简介**

该漏洞是由于用友 NC 对外开放了 BeanShell 接口，攻击者可以在未授权的情况下直接访问该接口，并构造恶意数据执行任意代码并获取服务器权限。

**0x02 影响版本**

```
NC6.5版本
```

**0x03 漏洞复现**

FOFA 语句：

```
icon_hash="1085941792"
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTH23Qn1c9IEUuOiarb4EY5tchySc8oxR1RY79J2GibSJgeYFkH3oUiaJCfhjkvEb9BibSt3oicNfUB8yUQ/640?wx_fmt=png)

**访问目标站点这个酱紫**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTH23Qn1c9IEUuOiarb4EY5tcv5jwrFRjTu9Ho1ZGzxKia5X5X1wW5DwOpIVWvpeiaeVzbwiaIu1j5YcEw/640?wx_fmt=png)

**执行命令**

**PoC:**

```
/servlet/~ic/bsh.servlet.BshServlet
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTH23Qn1c9IEUuOiarb4EY5tcHJceoaibVe8WCRribaS3lEUXeciaia4URLiaFL4UuGaDFib2UbptFoPc1tfg/640?wx_fmt=png)

**脚本验证**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTH23Qn1c9IEUuOiarb4EY5tcaNdzSCKJ6PzIV7yt0lek9ibaKpEQXGd3Db7B6BCv7GrKU7bUy2hHrCw/640?wx_fmt=png)

用脚写的太菜就不放出来见笑了![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTH23Qn1c9IEUuOiarb4EY5tcVjibZliaiboCcnIt7rbKswhzDzc1CicnrE9kzfqbYloh4vyS6aRs994Ffw/640?wx_fmt=png)

**0x04 修复方案**

```
官方已发布安全补丁，建议使用该产品的用户及时安装该漏洞补丁包。
下载链接：
http://umc.yonyou.com/ump/querypatchdetailedmng?PK=18981c7af483007db179a236016f594d37c01f22aa5f5d19
```

****【往期推荐】****  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - Shellcode Loader 原理 (python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - 分离 + 混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附 PoC】CVE-2021-2109 | Weblogic Server 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)

基于 Kali Linux 环境，从理论、应用和实践三个维度详解 Windows 渗透测试，通过 136 个操作实例手把手带领读者学习，详解环境搭建、主机发现、嗅探欺骗、密码攻击、漏洞扫描等。