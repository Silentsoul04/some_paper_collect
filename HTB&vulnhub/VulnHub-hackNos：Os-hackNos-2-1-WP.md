> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cN2Tpyt1BFVlLvgnw6avDQ)

**靶机描述：**

Difficulty : Easy to Intermediate

Flag : 2 Flag first user And second root

Learning : Web Application | Enumeration | Password Cracking

大家好，我们是想要为亿人提供安全的亿人安全，这是我们自己想要做的事情，也是做这个公众号的初衷。希望以干货的方式，让大家多多了解这个行业，从中学到对自己有用的芝士~

**靶机描述**

**一、信息收集**
----------

1. 首先通过命令 arp-scan-l 确定靶机 ip

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkEByY86fibTrzDlNxPfv5tLfSVYanIyW6apNcLaFricaqroNRpiaG2E8ZA/640?wx_fmt=png)

2. 然后使用 nmap 进行端口扫描，并且用 dirb 对其进行目录爆破

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvksvEeCBDft03ib9CjSezUMpeBNfjP34dWPs5gF02BGj8Qz2vbQE8XzicA/640?wx_fmt=png)

3. 然后 dirb，扫到了 tsweb 目录，然后访问看看。

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkyhxP1pF5SHGbdqSafVSMflkVrRZIzehujsrvZK5dLGsIcnu8MGc9qA/640?wx_fmt=png)

4. 然后使用 wpscan 工具进行扫描，发现了 gracemedia-media-player 插件

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkmtJ2nnbLxOibkGX2W8ID9XuqWGM5ZZSa7ZibWfebNVtEfUkKZ4fWkQfQ/640?wx_fmt=png)

5. 使用 Google 搜索看看，有什么好东西。

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkg9fLGbK69I9h2icwvZLNgXAcONIO45FcqEzIWAazZ25e8yPF5mWQcSg/640?wx_fmt=png)

6. 可以利用 CVE-2019-9618 漏洞渗透

.

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkQjgzQdk1hJLrMtM0jrUqXsEtcBib8yo8hcmbiciaiblicqr4XuydFLiceIZQ/640?wx_fmt=png)

7. 执行之后找到了 flag1

flag:$1$flag$vqjCxzjtRc7PofLYS2lWf/

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkswyIflkFe7Pt1v7mWFdgibklMMnP6t2V2dic0ib1fIdmpu9ETa9VPzPjQ/640?wx_fmt=png)

8. 然后使用 john 进行爆破，得到了密码 topsecret

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkzM6Q0IzO9T3F1O43ngBJ6Hoe34AylVZKyJt1QHibrpiabLWQN2UT405Q/640?wx_fmt=png)

**二：提权**

1. 登录之后在 backups 下看到了一个可能是密码… 发现了 MD5 值。

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkpl3GektRiaYUibZGqGM419BagqniacvSGibhOnMYDuFHThZAtAmSiawQSQw/640?wx_fmt=png)

$1$rohit$01Dl0NQKtgfeL08fGrggi0

2. 利用 john 解密试试

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkzVj1e5Cf8qsQicKBJQ2ZoGSlCT59mmOXveiaUV9oOT1qXVYz6TxibRtVQ/640?wx_fmt=png)

3.john 爆破密码：!%hack41

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkTegZJ3LYrsicarh06fxy8ibPQLueS1LaoUzt2FRXqCTknA0k4XDWKxFg/640?wx_fmt=png)

4. 拿到了 flag，然后提权

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkoIqTwibJkjyONAaTBGLn7WkuvEEaMGtxtvNpJia55Nic4cpRVgDibOmHyA/640?wx_fmt=png)

5. 发现可以使用 sudo 提权，然后再次输入之前的密码，成功获得 root 权限。找到了第二个 flag

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpGfMNVibwSbG2Vjz37SDIvkrme1gEqrdmz4iaCAdUibuzB0JfZOD9guFsDsibM8aCZnVB0lnVM1FelLw/640?wx_fmt=png)

**三：总结**: 这台靶机还有别的提权方式，希望有时间的朋友可以试试。希望这篇文章可以帮助到你，我也是初学者，希望会的大佬勿喷，如果有错误的地方请指出来，我一定虚心改正。如果有好的方法，大家可以分享出来，多多交流。