> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/m2tvGJdOmFOm-K5j-zUzQQ)

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U9urR0NDSva8zibsz2GSQhV40tnaYcEYkIF8icWMoaicpcpUeRKZknDB8lA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击上方蓝字关注我哦！

  

### **简介（Gamma实验室第二个开源工具）**

在日常渗透过程中我们经常遇到信息泄露出ALIYUN_ACCESSKEYID与ALIYUN_ACCESSKEYSECRET（阿里云API key），特别是laravel框架得debug信息。APP中也会泄露这些信息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U9TQttegumoUopUYPj8cznYVaToQxVFDO7JOQ8nuicqq0LGEoR8GUauLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

**！！！下载链接在文末！！！**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U9NKRxoun12cbGLrLeLfVP4m1R8E4ok6LkBs9P2T9qMuQZ6tD1YXNRfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **概述**

我们说下阿里API有什么用吧，以下是官方说明：

云服务器（Elastic Compute Service，ECS），可以调用API管理您的云上资源和开发自己的应用程序。

ECS API支持HTTP或者HTTPS网络请求协议，允许GET和POST方法。您可以通过以下方式调用ECS API

详情参考阿里云官方API文档：https://help.aliyun.com/document_detail/25484.html?spm=a2c4g.11186623.6.1276.12244f88jytZ8c

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U9NKRxoun12cbGLrLeLfVP4m1R8E4ok6LkBs9P2T9qMuQZ6tD1YXNRfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

### **开发思路**

#### **1、通过阿里云SDK使用**

SDK下载地址：https://github.com/aliyun/aliyun-openapi-python-sdk

  

pip安装

```
`# Install the core library``pip install aliyun-python-sdk-core` `# Install the ECS management library``pip install aliyun-python-sdk-ecs` `# Install the RDS management library``pip install aliyun-python-sdk-rds`
```

  

调用查询ecs主机

```
`#!/usr/bin/env python``#coding=utf-8``from aliyunsdkcore.client import AcsClient``from aliyunsdkcore.acs_exception.exceptions import ClientException``from aliyunsdkcore.acs_exception.exceptions import ServerException``from aliyunsdkecs.request.v20140526.DescribeInstancesRequest import DescribeInstancesRequest``client = AcsClient('<accessKeyId>', '<accessSecret>', 'cn-hangzhou')``request = DescribeInstancesRequest()``request.set_accept_format('json')``response = client.do_action_with_exception(request)``# python2:  print(response)` `print(str(response, encoding='utf-8'))`
```

  

创建命令

```
`#!/usr/bin/env python``#coding=utf-8``from aliyunsdkcore.client import AcsClient``from aliyunsdkcore.acs_exception.exceptions import ClientException``from aliyunsdkcore.acs_exception.exceptions import ServerException``from aliyunsdkecs.request.v20140526.CreateCommandRequest import CreateCommandRequest``client = AcsClient('<accessKeyId>', '<accessSecret>', 'cn-hangzhou')``request = CreateCommandRequest()``request.set_accept_format('json')``response = client.do_action_with_exception(request)``# python2:  print(response)` `print(str(response, encoding='utf-8'))`
```

  

这里会返回一个云助手命令id，返回结果：

```
`{``"RequestId": "E69EF3CC-94CD-42E7-8926-F133B86387C0",``"CommandId": "c-7d2a745b412b4601b2d47f6a768d3a14"``}`
```

  

执行命令

```
`#!/usr/bin/env python``#coding=utf-8``from aliyunsdkcore.client import AcsClient``from aliyunsdkcore.acs_exception.exceptions import ClientException``from aliyunsdkcore.acs_exception.exceptions import ServerException``from aliyunsdkecs.request.v20140526.InvokeCommandRequest import InvokeCommandRequest``client = AcsClient('<accessKeyId>', '<accessSecret>', 'cn-hangzhou')``request = InvokeCommandRequest()``request.set_accept_format('json')``response = client.do_action_with_exception(request)``# python2:  print(response)` `print(str(response, encoding='utf-8'))`
```

  

返回结果

```
`{``"RequestId": "E69EF3CC-94CD-42E7-8926-F133B86387C0",``"InvokeId": "t-7d2a745b412b4601b2d47f6a768d3a14"``}`
```

  

安全组部分就省略了，根据API文档

  

#### **2、通过GET/POST** **使用**

这里先说下公共请求参数

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U94m4icPGqDPYE9Af8qyicoCaxLwnfWzdE9XPXn3p1rPmB4TEXLhgho6BQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

2.1 GET请求

```
`https://ecs.aliyuncs.com/?Action=DescribeInstanceStatus``&RegionId=cn-hangzhou``&PageSize=1``&PageNumber=1``&InstanceId.1=i-bp1j4i2jdf3owlhe****``&<公共请求参数>`
```

XML返回格式：

```
`<DescribeInstanceStatusResponse>``<PageNumber>1</PageNumber>``<InstanceStatuses>``<InstanceStatus>``<Status>Running</Status>``<InstanceId>i-bp1j4i2jdf3owlhe****</InstanceId>``</InstanceStatus>``</InstanceStatuses>``<TotalCount>58</TotalCount>``<PageSize>1</PageSize>``<RequestId>746C3444-9A24-4D7D-B8A8-DCBF7AC8BD66</RequestId>``</DescribeInstanceStatusResponse>`
```

JSON返回格式

```
`{``"PageNumber": 1,``"InstanceStatuses": {``"InstanceStatus": [` `{``"Status": "Running",``"InstanceId": "i-bp1j4i2jdf3owlhe****"` `}` `]` `},``"TotalCount": 58,``"PageSize": 1,``"RequestId": "746C3444-9A24-4D7D-B8A8-DCBF7AC8BD66"``}`
```

  

2.2 POST请求

```
`POST / HTTP/1.1``Host: ecs.aliyuncs.com``User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:84.0) Gecko/20100101 Firefox/84.0``Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8``Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2``Accept-Encoding: gzip, deflate``Content-Type: application/x-www-form-urlencoded``Content-Length: 0``Action=DescribeInstanceStatus&RegionId=cn-hangzhou&PageSize=1&PageNumber=1&InstanceId.1=i-bp1j4i2jdf3owlhe****&<公共请求参数>`
```

返回跟GET方式一样

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U93Fm1tsiczScpdhQWJFiaZ0ar9DUEX5MGbeiaA5tt6kMibbTxPcVzf8eRpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U93Fm1tsiczScpdhQWJFiaZ0ar9DUEX5MGbeiaA5tt6kMibbTxPcVzf8eRpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U99l74wlgYW9J99Rn0jic7T2KuvzHngCUCjPv9fNr0aqqKzZOMibvr1I9Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **工具使用**

### 图形化界面，没什么说的。附一张截图相信大家都明白了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U90ibFglcvicUUiaMhiceH1icAvhHv8fDydlXoffpZ0wNR2tAzWdn3iafClicQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U9ZnGqmP59kF1HEc6VyT6iaz1vM3D68nF0N3XaLCLtia6DyQgNd7GyVbiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

### **结束**

阿里云为运维人员与开发人员提供了方便，但同时自身也要加强安全意识，注意自己的key不要泄露，不然直接接管阿里云所有esc主机，风险比一般高危漏洞都还要高。

  

  

  

  

### **下载链接及漏洞文章**

项目链接：https://github.com/mrknow001/aliyun-accesskey-Tools

工具下载链接：https://github.com/mrknow001/aliyun-accesskey-Tools/releases/download/1.0/Aliyun-.AK.Tools.exe

  

欢迎关注Gamma实验室,后续会推出更多实用方便的工具，爱您！

  

文章链接：https://www.freebuf.com/articles/web/255717.html

  

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuUG8XZ4x4lpqU9KhEiaQ6U9ia3ovj54GVvHRW2SkG6p8Mkum3qXpsIEaLqtZbbGD6LAE0bQ86ZgX2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

END

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtIZ5VYusLbEoY8iaTjibTWg6AKjAQiahf2fctN4PSdYm2O1Hibr56ia39iaJcxBoe04t4nlYyOmRvCr56Q/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

  

**看完记得点赞，关注哟，爱您！**

  

**请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

  

关注此公众号，回复"Gamma"关键字免费领取一套网络安全视频以及相关书籍，公众号内还有收集的常用工具！

  

**在看你就赞赞我！**

![图片](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbThXaInFkmyjOOcBoNCXGun5icNbT4mjCjcREA3nMN7G8icS0IKM3ebuLA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTkwLkofibxKKjhEu7Rx8u1P8sibicPkzKmkjjvddDg8vDYxLibe143CwHAw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/96Koibz2dODt7492lKcjVLXNwERFNUQJVkkKj3EYBiboRWmHfnymrDxeEVrYapXicBGbRLhPzWv5wbhXR59PDyC8Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

扫码关注我们

![图片](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

  

扫码领hacker资料，常用工具，以及各种福利

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTnHS31hY5p9FJS6gMfNZcSH2TibPUmiam6ajGW3l43pb0ySLc1FibHmicibw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

转载是一种动力 分享是一种美德

![](http://mmbiz.qpic.cn/mmbiz_png/BgtOBfjgBw2b45hFJofcljuYW2JjaEZtQWdw2XQFttVf4AichibuqymWOGV9IW8yTBZkYUGeAaoxxrItQzgIXvTA/640?wx_fmt=png&wxfrom=200) 交易担保 小互动助手 Gamma安全实验室 小程序

                 

免责声明：本站提供安全工具、程序(方法)可能带有攻击性，仅供安全研究与教学之用，风险自负!

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

  

订阅查看更多复现文章、学习笔记

thelostworld

安全路上，与你并肩前行！！！！

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/uljkOgZGRjeUdNIfB9qQKpwD7fiaNJ6JdXjenGicKJg8tqrSjxK5iaFtCVM8TKIUtr7BoePtkHDicUSsYzuicZHt9icw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

个人知乎：https://www.zhihu.com/people/fu-wei-43-69/columns

个人简书：https://www.jianshu.com/u/bf0e38a8d400

个人CSDN：https://blog.csdn.net/qq_37602797/category_10169006.html

个人博客园：https://www.cnblogs.com/thelostworld/

FREEBUF主页：https://www.freebuf.com/author/thelostworld?type=article

![图片](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcW6VR2xoE3js2J4uFMbFUKgglmlkCgua98XibptoPLesmlclJyJYpwmWIDIViaJWux8zOPFn01sONw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

欢迎添加本公众号作者微信交流，添加时备注一下“公众号”  

![图片](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3ibf9GUyuOCzpVJBq6z1Z60vzBjlEWLAu4gD9Lk4S57BcEiaGOibJfoXicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)