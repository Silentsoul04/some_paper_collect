> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vGf1Y8MK9912IyTZjCMGTQ)

![](https://mmbiz.qpic.cn/mmbiz_png/gicQ7o2bUTaxXgDML1Cvs0YYYGg7otpOAQ2gzVSZojZ3bqfyfLiaHJU0UXotWCThVwW9wP9AcebCAmGiahgVrFibKQ/640?wx_fmt=png)

点击上方蓝字关注我

![](https://mmbiz.qpic.cn/mmbiz_png/RQeIt3Pib9ACndicibtRFhb6kvGnco1ruEg1kd4dx35GUUAl2ia08ib3usxsUJZP5smvZh9N1zg8uQ5mgibwn34gxHhA/640?wx_fmt=png)

前言

DNS 的全称是 Domain Name System（网络名称系统）, 它作为将域名和 IP 地址相互映射, 使人更方便地访问互联网.

当用户输入某一网址如 luomiweixiong.com, 网络上的 DNS Server 会将该域名解析, 并找到对应的真实 IP 如 127.0.0.1, 使用户可以访问这台服务器上相应的服务.

DNSlog 就是存储在 DNS Server 上的域名信息, 它记录着用户对域名 leishianquan.com 等的访问信息, 类似日志文件.

原理图：

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlVn1grPHeXktwcyvibVaqeCLND3jHz4aniasGe8If6dNI79ejLdhTReiaQ/640?wx_fmt=png)

举个栗子

比如说, 我注册了一个为 luomiweixiong.com 的域名, 我将 它的 a 记录泛解析到 139.x.x.x 上, 这样就实现了无论我记录值填什么他都有解析, 并且都指向 139.x.x.x, 当我向 dns 服务器发起 test.luomiweixiong.com 的解析请求时,DNSlog 中会记录下他给 test.luomiweixiong.com 解析, 解析值为 139.x.x.x.

部署

一、域名解析配置

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlbCBAtiaGHibkeWNPtgeHfm1RUUASVWHj01cc22ibOD8O5HtJVibMyF4TDg/640?wx_fmt=png)

添加一个 A 记录与 2 个 ns 记录. 其中 A 记录指向服务器 IP 地址,NS 记录指向 A 记录的域名地址.

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlbQkvxGnQDgZnMVSootMEc6raY3g9icvz6ia9tzh2IPF0eEtdsNibjP13w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIl2gKzHD41Cgnz3NicCpPCctfxkC26crA9uibk8fwZMIOGQnibcsmeur4uw/640?wx_fmt=png)

二、自定义一个 dns host

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlsaGtLQ0fqeaYnT2s189e182Zsiaw6ib8Mp3wOV00EIQa1iaoHIEWkbTYQ/640?wx_fmt=png)

三、项目部署

```
https://github.com/lanyi1998/DNSlog-GO
```

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIloQicAFHMjk2oJhibDzh6SIW4Ah9f9ocaJnzCFCwyBdYeXllJFFXsCYmQ/640?wx_fmt=png)

1、该项目是由 GO 语言编写的, 所以部署的时候需要用到 GO 语言的环境.

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIljTpyEqP2BhPkEQJsmT4TVffo8yhQYPIVgPX5ewiaHsfp24v0v0p1ibzQ/640?wx_fmt=png)2、DNS 使用的是 53 端口, 记得 53 端口的放行.

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlr6Ob24MwQs9VEtiaZQbo4yVckmFzpFTC0Q57bUpQmib0HjqmzDORMAwA/640?wx_fmt=png)

3、配置文件 config.ini 的修改 (我这里前端采用的是 8000 端口, 也记得要放行)

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIl3etcVWIkceaJ9UgQdXNEXnDDkdM6R3qVdIBYbnrep1vWn70tkTVZmg/640?wx_fmt=png)

四、启动  

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlBzjXKbY8hNd13Q5Jq2KAm9taV7iaXs0T7opOl26IPxeqmRXjF6lyN5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlBwWMxiavyMQhQP9vSEF4ML8s9hZ2ssnc1zk9DzNq8GdzkpEch8MarWQ/640?wx_fmt=png)

访问出现需要输入 Token 就需要填入上面配置文件 config.ini 中的 Token "luomweixiong"

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlHfX2cFPN9Xgpzox3I6Y8FYCDenE1mAHTYZk6YRibSDvQ5vqMsPibt7AA/640?wx_fmt=png)

五、环境验证

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlDsqh1GlybGFFbpzYhKacEUr41hcjpOetYtpBF8tPNj7ibnafg8tVH6w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwOMJ0w2LzqnUib3Sh4KJicSIlE1Fl7YqgsMh7hoCGdPK8VFwx91U4mE9IM9t0DicSQI6WhpXRPQSicclQ/640?wx_fmt=png)

六、搭建成功

总结

一、在配置 A 记录与 DNS 的时候需要稍等片刻等生效, 不要怀疑没有配置成功.

二、尽量使用自己搭建 dnslog, 避免信息泄漏, 或者被别人捕获信息.

往期推荐

[未授权访问漏洞汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484804&idx=2&sn=519ae0a642c285df646907eedf7b2b3a&chksm=ea37fadedd4073c87f3bfa844d08479b2d9657c3102e169fb8f13eecba1626db9de67dd36d27&scene=21#wechat_redirect)

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)  

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)  

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)

[

](http://mp.weixin.qq.com/s?__biz=MzIzODE0NDc3OQ==&mid=2247483787&idx=1&sn=7bee29f9a0a1e06d60ad6b28c44cabf5&chksm=e93c9defde4b14f9418e5a25d39b55f0576fc19479f74df36d25402c9cfb508358e3c70b4d3f&scene=21#wechat_redirect)  

  

  

点个在看 你最好看![](https://mmbiz.qpic.cn/mmbiz_png/ImtD1PjRzRibmwqBpXL6icIKqbwdwwR26NfB89hJ09AJCorfLHxNdGlIIKr02IiajJ3O6t3qzXFXcJZ1lUxUnibTIA/640?wx_fmt=png)