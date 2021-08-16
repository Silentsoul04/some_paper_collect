> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ud53rhgyHIxw6KtDCk0b_A)

Part 01

**引言**

    此次为授权渗透，但客户就丢了一个链接啥都没了。这种情况不好搞，分享这篇文章的原因主要是过程曲折，给大家提供下一些思路，当然大佬有更好的思路也可以分享下。

Part 02

**正文**  

打开就是登录页面

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7iaNPsKKFgJoW7BuW0N1HfeHEcgpibr9B6Layib1RgM4nibJIrt3KSZacYg/640?wx_fmt=png)

打开登录页面，第一反应是先试试错误次数有没有限制，用户名可否枚举。然后有个记住我，尝试下 shiro。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7fib221aqY0UnibKf9996v5SEL7h460qaPgkpdiaJ6K80mS3mL87IG8xaA/640?wx_fmt=png)

提示用户名或密码错误，多错误几次也没弹窗验证，Intruder 安排上，直接爆破的是 admin 账号，不然交叉爆破太费时间，爆破同时 shiro 也去试试。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7rfp2VfsTZnx9ff06OKzQrk3wQnSEO5fbf0bxSoDFWXOYLLrPGicEflQ/640?wx_fmt=png)

确实是使用的 shiro，虽然没有爆破出来，但有这个也是不错的。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7KvKWaaS6z2jX9m30U3xOqzKj3udhmhM6vib3LiaghVw3NbhfrtjmLqvA/640?wx_fmt=png)

毫不客气的打开了紫霞大佬的 shiro 工具。

换了个 shiro 工具也没跑出来 key。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7uRo3uPNAn4JId1QSyKmqCnz7ChAKEVc2nbo6VtggHAnceHayGkeOug/640?wx_fmt=png)

爆破不行，shiro 反序列化也没有，也没看出来是什么 cms 特征。

到这点上了，直接宣布渗透结束？要是这样那我岂不是被开除了？

先收集一波信息吧。

查看 js 发现如果账号密码正确会跳转到 index 这个页面

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7Cc5v9tp8Ea3iaOzs5tbCN1FN5a6kKKXMAxUDSibZPb9F7Ax0XRFIz48A/640?wx_fmt=png)

直接访问 index 页面，提示 302 跳转

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7I6gAib5zqdoy0muw1FZk6G6Ikw0vmEKUmCr5EglLOicrX1OvNWBzfIfw/640?wx_fmt=png)

看到这里我突然想起了 shiro 还有个权限绕过，反手就是一个 /;/index

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7zQdZXHI3XdzYKn3vBOTNDEQvicfyIicOvHibSUMaaSx9UG67ESr4IuLng/640?wx_fmt=png)

果然 shiro 默认 key 是改了但权限绕过还是存在，随知而来得还有一个坏消息站点得目录怎么获得呢？

把 js 找遍了也没有看见其他路径了，最后掏出目录爆破工具试试，加上 /;/ 去爆破。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7tAG4tAsLJP4JLT20g1uaTQOQqALrqT4ZfNRcd37TfwKiat0lpaRBibJw/640?wx_fmt=png)

这站的 swagger 既然对外网开放，真是绝处逢生。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7fqARsW9BRb0breWicujfmAPZXicaR5d5K1vlWicbLD1JYcMEyYWz1K3eg/640?wx_fmt=png)

之前有朋友问 swagger 是什么，我这里就顺便说下。

```
Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化RESTful 风格
的Web 服务。 总体目标是使客户端和文件系统作为服务器以同样的速度来更新。 文
件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。 
Swagger 让部署管理和使用功能强大的API从未如此简单
```

Swagger 里面没有找到能 getshell 的接口，只能从后台下手，swagger 只是一些 api 接口信息，而且增加用户修改密码这些接口会验证登录账号也无法使用，在后台或许还能有机会。

果然皇天不负有心人，最后找到了用户名和密码。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7sL9QbVJoykvrGrkgHUVuicMIk0gibAWYyVUxONlRQfprgeiclyqEMGhQQ/640?wx_fmt=png)

把加密的密码拿去各个网站解都无果（一开始以为是强口令，没有去尝试爆破）

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7yg14NDRdeXzxEhf9wicC0DlTsdoDCMLEEGPqiczZtzknzABZsHjUXk2w/640?wx_fmt=png)

 然后去日志查询接口逛了一圈，确定了管理员账号是 jwj。并不是 admin，难怪最开始跑字典没跑出来，估计这也是一个原因，只能再试试爆破了。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7s6bWfMYQ2E2p3emVLe9GkuPVuh8GicYf2pkq58fZialDS3q25ggMoricQ/640?wx_fmt=png)

果然爆破出来了，密码八个八，成功进入后台。

【后台特征太多，无截图】

进入后台翻了个遍，找到个上传，而且也只是个前端过滤，那不得送他个好的马子。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7t8HamgjedORdJOVH4Rs7icl7ZqtkPa4ibvuC9fPJ9B0FE91Mk93LCEZw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7LhAKicicvlS7H7JIxr3gx9gvlI2GMeadW144S7iafLJVWQEvyqtDvGcag/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicwsibic72TPLrVDF1G83YyqL7Gex7Vzcq8THd2Jsdp8Y6VdGkHofNxIia5HBkbX6Megzyn4uicBB7eqfw/640?wx_fmt=png)

蚁剑连上，完工。

Part 03

**总结**

此次渗透虽然技术含量不好，但非常有趣。从爆破后台账号密码开始，兜兜转转又回到了爆破后台账号密码。重新验证了那句话，渗透的本质就是信息收集。

Part 04

**免责声明**

    本项目仅进行信息搜集，漏洞探测工作，无漏洞利用、攻击性行为，发文初衷为仅为方便安全人员对授权项目完成测试工作和学习交流使用。请使用者遵守当地相关法律，勿用于非授权测试，勿用于非授权测试，勿用于非授权测试~~

**【往期推荐】**  

[未授权访问漏洞汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484804&idx=2&sn=519ae0a642c285df646907eedf7b2b3a&chksm=ea37fadedd4073c87f3bfa844d08479b2d9657c3102e169fb8f13eecba1626db9de67dd36d27&scene=21#wechat_redirect)

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)  

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)