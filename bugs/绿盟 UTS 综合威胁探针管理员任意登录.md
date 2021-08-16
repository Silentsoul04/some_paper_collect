\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/y2AuWIU-butHdO2K9Ijd3A)

**点击蓝字 ·  关注我们**

**01**

漏洞标题  

绿盟 UTS 综合威胁探针管理员任意登录

**02**

漏洞详情

随着 5G、物联网和人工智能等新技术的全面普及，黑客的攻击手段层出不穷，为了应对大量不同类型的攻击，传统威胁检测方案面临诸多挑战，NSFOCUS 推出了绿盟综合威胁探针（简称 UTS）。UTS 是一款集 IDS、WAF、威胁情报和全量行为日志于一身, 支持对接第三方大数据平台的多功能融合探针。

UTS 通过搭载 IDS 和 WAF 双检测引擎系统，结合威胁情报、恶意文件检测、DDoS 检测、Webshell 检测和异常行为检测等手段能快速检测传统威胁和高级威胁，同时配合自身的阻断策略对威胁进行快速旁路阻断，缩短用户响应处置时间，此外还可通过输出标准化日志对接态势平台进行统一威胁呈现和回溯分析。

绿盟综合威胁探针设备版本 V2.0R00F02SP02 及之前存在此漏洞。

**03**

漏洞利用过程

**0x01**

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOFgWlYlI2N93uFfjticia07icfEha5FgIvKTiaVaREO7t05cDiacRGqicRY8sA/640?wx_fmt=png)

对响应包进行修改，将 false 更改为 true 的时候可以泄露管理用户的 md5 值密码

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOFibwibFlql8FOgfQokEof39pyEjxUu88MNUBofqa4b938WkfFwJZp9EDQ/640?wx_fmt=png)

**0x02**

利用渠道的 md5 值去登录页面

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOFsNgvuOQBCKRwwVyTZvtQJayeJCIAnrYrSsZQ5MhLZRSLWWziagndNVg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOFhxMvjzibyrIQ0KVYDATuMrdTu4yIHn3Dbfl7aOQsXl7YsV3MKyXvNDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOFtR6jsYvsCKqCibqA6jrPpGqibs77g0MkFxZwgYuuZheOx4y9RffwaqrQ/640?wx_fmt=png)

**0x03**

7ac301836522b54afcbbed714534c7fb

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOFzhydAEu71Bjib1FNy6Tzchf6ibziaksiagaQJUgW0YHmmgPtEAFE7bcnVw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOF77lsTia4Qddnaj9iajr2DIw2drEJbvpfHlJVKgJUh0jZShSibrlj50Pfg/640?wx_fmt=png)

**0x04**

成功登录，登录后通过管理员权限对设备进行管控，并且可以看到大量的攻击信息，泄露内部网络地址包括资产管理。

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOF77lsTia4Qddnaj9iajr2DIw2drEJbvpfHlJVKgJUh0jZShSibrlj50Pfg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgembVSibbq2J8IsYKB25TxrOF3tbibWtUqoic4PdeyKHB4X6JROCSeOgALIjibX4ibLHbGLE2ibkqw3yyLQQ/640?wx_fmt=png)

**04**

处置意见

建议尽快更新补丁至最新：

http://update.nsfocus.com/update/listBsaUtsDetail/v/F02

关注 EDI 安全，获取更多 HVV 情报

**EDI 安全**

![](https://mmbiz.qpic.cn/mmbiz_jpg/rJALXSMzgembVSibbq2J8IsYKB25TxrOFcXvDZ3VLtWSWS79p4cWzW6fdDEmticXGuIvLKNicfE4icSYU01mFj4YRA/640?wx_fmt=jpeg)

**扫二维码｜关注我们**

一个专注渗透实战经验分享的公众号