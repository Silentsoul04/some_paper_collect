> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3t4Yo3dXKWGg1_UUo7etKA)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **64** 篇文章，本公众号会记录一些红队攻击的案例，不定时更新

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

前言

用过云函数的 XD 们都知道，它可以用来帮助我们转发请求，但是这种方法已经被老外玩烂了，但是也很实用！由于它自带 CDN 这样我们每次请求 Webshell 的时候 IP 都是不同的，从而达到隐藏 RT 的效果！还是那句话，一个合格的 RedTeam 被溯源到是很可耻的！

**如何优雅的隐藏你连接 Webshell 的真实 IP**

**云函数隐藏 Webshell 真实 IP  
**

首先来到腾讯云后台找到云函数，我们使用自定义的模版：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJo1cGqib4GKqlU4NBDbQuEkoJXibqjKWIMQco9o5SibfemPeFS9Loq8fDzm0icq4UWqUt2orToBCGfcA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJo1cGqib4GKqlU4NBDbQuEkOAvrvAGKGJK7jkFia0gSUfqGuKtFzgZRm3xf6SgXbPNwDgdDjM5O5Uw/640?wx_fmt=png)

然后依次点击函数服务 -> 函数管理 -> 函数代码，然后将下面的代码粘贴到 index.py 中：

```
# -*- coding: utf8 -*-
import requests
import json
def geturl(urlstr):
    jurlstr = json.dumps(urlstr)
    dict_url = json.loads(jurlstr)
    return dict_url['u']
def main_handler(event, context):
    url = geturl(event['queryString'])
    postdata = event['body']
    headers=event['headers']
    resp=requests.post(url,data=postdata,headers=headers,verify=False)
    response={
        "isBase64Encoded": False,
        "statusCode": 200,
        "headers": {'Content-Type': 'text/html;charset='+resp.apparent_encoding},
        "body": resp.text
    }
    return response
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJo1cGqib4GKqlU4NBDbQuEkoVa5nCibR9V0icOcKefNx4x0JqtIqYRDSQ6z4ytBICaO2ZO1IBicjPiaSg/640?wx_fmt=png)

然后点击部署后，创建一个触发器：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJo1cGqib4GKqlU4NBDbQuEkIvuZ5l5BI1hAXsiagsXFuafsdalCGI00949hzYM47yBZgma6zIxA3EA/640?wx_fmt=png)

这里需要选择 API 网关触发：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJo1cGqib4GKqlU4NBDbQuEkqWuFUgRrbKNuEn6nakRIdgtib9CcEaiaibEaL0ib3yyibJqALKm4uDpzJMQ/640?wx_fmt=png)

然后就可以访问这个了：

```
https://service-gh2cn6ys-xxxxx.gz.apigw.tencentcs.com/release/saulGoodman
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJo1cGqib4GKqlU4NBDbQuEkhfY7uibuzicP86wPsOFGp7BJ18RgPic8Nwqdz1ybgbrfoTKnFyn8roTSQ/640?wx_fmt=png)

这个时候 u 参数后面就是你的一句话：http://111.111.111.111/saulGoodman.php

```
https://service-gh2cn6ys-xxxxx.gz.apigw.tencentcs.com/release/saulGoodman?u=http://111.111.111.111/saulGoodman.php
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJo1cGqib4GKqlU4NBDbQuEkjwYXJrl6qxTaId9axeJDtHe3soEiaia1BibdeYEjo42Poez1JNvHt2MMA/640?wx_fmt=png)

这个时候每次访问 webshell 的 IP 都不一样！从而隐藏了 RT 的真实 IP！

* * *

参考文章：  

https://blog.csdn.net/qq_41918771/article/details/114359458

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

渗透攻击红队

一个专注于渗透红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDdjBqfzUWVgkVA7dFfxUAATDhZQicc1ibtgzSVq7sln6r9kEtTTicvZmcw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDY9HXLCT5WoDFzKP1Dw8FZyt3ecOVF0zSDogBTzgN2wicJlRDygN7bfQ/640?wx_fmt=png)

点分享

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDRwPQ2H3KRtgzicHGD2bGf1Dtqr86B5mspl4gARTicQUaVr6N0rY1GgKQ/640?wx_fmt=png)

点点赞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDgRo5uRP3s5pLrlJym85cYvUZRJDlqbTXHYVGXEZqD67ia9jNmwbNgxg/640?wx_fmt=png)

点在看