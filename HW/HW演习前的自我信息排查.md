> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247491257&idx=1&sn=0cdf847f4a2c33825c782d521952e6b9&chksm=f9e071e2ce97f8f486f9d56576201efe90cbcaa42b6dee75169d6822c7c97c234b0843ccf07e&scene=21#wechat_redirect)

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icnicODgklvYb1ZeFibpnUKrR3e2tgu38kerpdsaAmawjIB0sgmFZIHNg7jmaJehfT9PYWTETjSlNSow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)       
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  

---

信息收集概述
------

信息收集一般都是渗透测试前期用来收集，为了测试目标网站，不得不进行各种信息收集。根据个人渗透测试经验总结文章。本文只是抛砖引玉，希望可以给大家一个好的思路。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNZIszdufDbQZOzLPLRMdqXoQmCGw9ibepbhWqbPianD37ic23ibe9dibIXCw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**+**

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNDfJUNA9jX9rePrf1JWNZ0mdUg4Kd6KIg86BOxwut4p3oKRRDFDGia1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**+**  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

信息收集
----

### **1、robots.txt**

当一个搜索蜘蛛访问一个站点时，它会首先检查该站点根目录下是否存在`robots.txt`，如果存在，搜索机器人就会按照该文件中的内容来确定访问的范围；如果该文件不存在，所有的搜索蜘蛛将能够访问网站上所有没有被口令保护的页面。

`robots.txt`基本上每个网站都用，而且放到了网站的根目录下，任何人都可以直接输入路径打开并查看里面的内容，如http://127.0.0.1/robots.txt ，该文件用于告诉搜索引擎，哪些页面可以去抓取，哪些页面不要抓取。

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNNElZckCEaicLq2hmrretJbTgkt5WvnZj2hJN9ibCxsQHl4QY1Kp3fJqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为了让搜索引擎不要收录`admin`页面而在`robots.txt`里面做了限制规则。但是这个`robots.txt`页面未对用户访问进行限制，可任意访问，导致可通过该文件了解网站的结构，比如admin目录、user目录等等。  

怎样即使用`robots.txt`的屏蔽搜索引擎访问的功能，又不泄露后台地址和隐私目录呢？

有，那就是使用星号（/*）作为通配符。举例如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNCPVpU1zhics97eRjm3OAxxYmvjbmCoadO9FRwbicAgviacR2VUM4zyQuA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个设置，禁止所有的搜索引擎索引根目录下a开头的目录。当然如果你后台的目录是`admin`，还是有可以被人猜到，但如果你再把`admin`改为`adminzvdl`呢？  
  

### **2、网站备份压缩文件**

管理员在对网站进行修改、升级等操作前，可能会将网站或某些页面进行备份，由于各种原因将该备份文件存放到网站目录下，该文件未做任何访问控制，导致可直接访问并下载。可能为`.rar、zip、.7z、.tar.gz、.bak、.txt、.swp`等等，以及和网站信息有关的文件名`www.rar、web.rar`等等

  

漏洞利用工具：御剑

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwN8OMP84qj4I4a3bsKEv9YtH1T4chex3eG2tO2dP66yz1x2K1hricaqUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###   

###   

### **3、Git导致文件泄露**

由于目前的`web`项目的开发采用前后端完全分离的架构:前端全部使用静态文件，和后端代码完全分离，隶属两个不同的项目。表态文件使用 git 来进行同步发布到服务器，然后使用`nginx` 指向到指定目录，以达到被公网访问的目的。

在运行`git init`初始化代码库的时候，会在当前目录下面产生一个`.git`的隐藏文件，用来记录代码的变更记录等等。在发布代码的时候，把`.git`这个目录没有删除，直接发布了。使用这个文件，可以用来恢复源代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNu1equHq0qsecnLTSOBE8CkBdibPKlcaQARmWUx4vF9FpibvhW1Se8xqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

GitHack是一个.git泄露利用脚本，通过泄露的.git文件夹下的文件，还原重建工程源代码。  

**脚本的工作原理**：

1.  解析.git/index文件，找到工程中所有的：( 文件名，文件sha1 )
    
2.  去.git/objects/ 文件夹下下载对应的文件
    
3.  zlib解压文件，按原始的目录结构写入源代码
    

GitHack.py http://www.hoolai.com/.git/

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNtibIHFBbOfvpPERh8mfAEGKrb5H4oVcibs5FBRUXViceOKmTn5N2s1HFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 执行结果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNNzK4Qgagw3xYsD4ibQ87ybqOUgFjdoKFpP63OdclOiaFSEmG5YK2DibDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

### **4、DS_store导致文件泄露**

.DS_Store是Mac下Finder用来保存如何展示 文件/文件夹 的数据文件，每个文件夹下对应一个。

如果开发/设计人员将.DS_Store上传部署到线上环境，可能造成文件目录结构泄漏，特别是备份文件、源代码文件。

ds_store_exp 是一个.DS_Store 文件泄漏利用脚本，它解析.DS_Store文件并递归地下载文件到本地：https://github.com/lijiejie/ds_store_exp

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNiaB8ZLOW0LIpjhwEzxtDA4pOs7sPPwj9sav0W6UL5fg6FNTlJPXluRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以模仿一个环境，利用`phpstudy`搭建`PHP`环境，把`.DS_store`文件上传到相关目录。

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNAp1dDosyZ7PYCYAczw4icCjpicBLEZoJFWfUzRTOATFCH9Kx22SS5icicA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后利用工具进行相关检测

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNbcjatQtouS4iaJy2mrFia8FQN0PZWC5nQFFN3ibLUgw3ibviaUBKp02iaIwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

工具下载地址：https://github.com/lijiejie/ds_store_exp

为了让实验更真实，我们在本地搭建环境，然后建立一个文件夹为admin和一个hello文件夹，利用该工具运行完以后，查看工具文件夹查看有什么结果。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

此文件和我们在一个文件夹内，如果是苹果用户，把文件copy到相关服务器目录以后，都会默认带一个文件`.DS_Store`。首先访问`test.php`文件，查看环境是否成功。

![图片](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icmfFLLhZpibianTQpTN9GJbwNNsgibK56vT213ic4WEHCGNWdVicjR4or4voP6hhfkZmmJ2BXPMKW2N4Yg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

环境搭建成功

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我们利用工具进行测试，运行完如上图，运行完以后我们可以到工具目录进行查看。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这是一个`.DS_Store`文件泄漏利用脚本，它解析`.DS_Store`文件并递归地下载文件到本地。

  

### **5、SVN导致文件泄露**

`Subversion`，简称`SVN`，是一个开放源代码的版本控制系统，相对于的`RCS、CVS`，采用了分支管理系统，它的设计目标就是取代`CVS`。互联网上越来越多的控制服务从`CVS`转移到Subversion。  

`SVN漏洞在实际渗透测试过程中，利用到也比较多，由于一些开发管理员疏忽造成，原理类似DS_Store漏洞。我们这里不再进行搭建环境，给大家推荐工具，利用方法如下：`  
  

#### 利用

1、漏洞利用工具：Seay SVN漏洞利用工具

2、添加网站url

在被利用的网址后面加 /.svn/entries，列出网站目录，甚至下载整站。

  

在被利用的网址后面加 /.svn/entries，列出网站目录，甚至下载整站  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

下载地址：https://pan.baidu.com/s/1jGA98jG

深入探索：  
https://github.com/admintony/svnExploit/  
我们采用大佬的exp(python3环境)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### **6、WEB-INF/web.xml泄露**

`WEB-INF`是`Java`的WEB应用的安全目录。如果想在页面中直接访问其中的文件，必须通过web.xml文件对要访问的文件进行相应映射才能访问。

WEB-INF主要包含一下文件或目录：

**/WEB-INF/web.xml**：Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则。**/WEB-INF/classes/**：含了站点所有用的 class 文件，包括 servlet class 和非servlet class，他们不能包含在 .jar文件中**/WEB-INF/lib/**：存放web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如数据库驱动jar文件**/WEB-INF/src/**：源码目录，按照包名结构放置各个java文件。**/WEB-INF/database.properties**：数据库配置文件

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

#### 6.1、环境搭建

我们需要利用jsp源码给大家进行示范，所以前提需要下载一个jsp环境，这里我们选取jspstudy进行示范。下载地址：http://www.phpstudy.net/phpstudy/JspStudy.zip

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

另外一种方法就是直接下载webgoat然后执行文件中webgoat.bat文件即可。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

下载地址：http://sourceforge.net/projects/owasp/files/WebGoat/WebGoat5.2/

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

#### 6.2、访问页面

访问地址：`http://localhost/,`进入此页面，证明我们tomcat已经启动，我们查看一下web.xml目录在哪里，你可以练习此靶场，靶场在后续会进行讲解。这里我们只讲解此web.xml信息泄露漏洞。如果让用户设置权限不严格，造成一些目录列出，结果是非常严重，我们通过访问web.xml文件，可以查看一些敏感信息，如下图

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

#### 6.3、扫描

利用工具扫描，我们得知此目录下面有一些敏感文件，我们尝试去访问

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

#### 6.4、验证结果

首先是一些tomcat登录信息，我们尝试去访问一些其它文件，通过不断尝试目录，有发现了一个sql文件和xml文件。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

###   

###   

### **7、hg源码泄漏**

`Mercurial 是一种轻量级分布式版本控制系统，使用 hg init的时候会生成.hg。`

`漏洞利用工具：dvcs-ripper`

`github项目地址：https://github.com/kost/dvcs-ripper`

`用法示例：`

```
rip-hg.pl -v -u http://www.example.com/.hg/
```

###   

### **8、CVS泄露**

`CVS是一个C/S系统，多个开发人员通过一个中心版本控制系统来记录文件版本，从而达到保证文件同步的目的。主要是针对 CVS/Root以及CVS/Entries目录，直接就可以看到泄露的信息。`

```
http://url/CVS/Root 返回根信息  
http://url/CVS/Entries 返回所有文件的结构
```

`漏洞利用工具：dvcs-ripper`

`github项目地址：https://github.com/kost/dvcs-ripper.git`

`运行示例:`

```
rip-cvs.pl -v -u http://www.example.com/CVS/
```

  

### **9、Bazaar/bzr泄露**

`bzr也是个版本控制工具, 虽然不是很热门, 但它也是多平台支持, 并且有不错的图形界面。`

`运行示例：`

```
rip-bzr.pl -v -u http://www.example.com/.bzr/
```

  

### **10、SWP 文件泄露**

`swp即swap文件，在编辑文件时产生的临时文件，它是隐藏文件，如果程序正常退出，临时文件自动删除，如果意外退出就会保留，文件名为 .filename.swp。`

`漏洞利用：直接访问.swp文件，下载回来后删掉末尾的.swp，获得源码文件。`

  

### **11、Zoomeye搜索引擎使用**

`ZoomEye`支持公网设备指纹检索和 Web 指纹检索

网站指纹包括应用名、版本、前端框架、后端框架、服务端语言、服务器操作系统、网站容器、内容管理系统和数据库等。

设备指纹包括应用名、版本、开放端口、操作系统、服务名、地理位置等

#### 11、1、搜索规则

首先，我们讲解下相关的快捷键，提高使用效率

*   Shift //: 显示快捷帮助
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   Esc: 隐藏快捷帮助
    
*   Shift h: 回到首页
    
*   Shift s: 高级搜索
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   s: 聚焦搜索框
    

####   

#### 11.2、搜索技巧

在设备和网站结果间切换

ZoomEye 将默认搜索公网设备，搜索结果页面左上角有公网设备和 Web 服务两个连接。因此您可以快速切换两种结果。

在输入关键字时，自动展开的智能提示下拉框最底部有两个指定搜索的选项。用方向键选定其中一个，回车即可执行搜索。

`ZoomEye` 使用 `Xmap` 和 `Wmap` ：两个能获取 Web 服务 和公网设备指纹的强大的爬虫引擎定期全网扫描，抓取和索引公网设备指纹。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

#### 11.3、实战搜索

我们今天主要讲下如何使用他的语法规则去高级搜索，搜索有用信息。

*   主机设备搜索组件名称
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

例1：搜索使用iis6.0主机：`app:"Microsoft-IIS" ver"6.0"`，可以看到0.6秒搜索到41，781,210左右的使用iis6.0的主机。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

*   端口
    

port: 开放端口

搜索远程桌面连接：`port:3389  
`

例1：查询开放3389端口的主机：`port:3389`

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   IP 地址
    

ip: 搜索一个指定的 IP 地址

例：搜索指定ip信息，`ip:121.42.173.26`

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

*   Web应用搜索
    

这里只讲解Web应用的查询方法

site:网站域名。

例子：查询有关taobao.com域名的信息，`site:taobao.com`

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**  
12、bing搜索引擎使用**

filetype:仅返回以指定文件类型创建的网页。

若要查找以PDF格式创建的报表，请键入主题，后面加`filetype:pdf`

  
       inanchor:、inbody:、intitle: 这些关键字将返回元数据中包含指定搜索条件（如定位标记、正文或标题等）的网页。为每个搜索条件指定一个关键字，您也可以根据需要使用多个关键字。  若要查找定位标记中包含msn、且正文中包含seo和sem的网页，请键入  

  

  

site:返回属于指定网站的网页。若要搜索两个或更多域，请使用逻辑运算符OR对域进行分组。

您可以使用site:搜索不超过两层的Web域、顶级域及目录。您还可以在一个网站上搜索包含特定搜索字词的网页。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

url:检查列出的域或网址是否位于Bing索引中。

请键入`url:sec-redclub.com  
`

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

###   

### **13、Fofa搜索**

Fofa 地址：https://fofa.so/

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

###   

### **14、站长工具**

#### 14.1、站长工具Whois

使用站长工具Whois可以查询域名是否已经被注册，以及注册域名的详细信息的数据库（如域名所有人、域名注册商）

http://tool.chinaz.com/

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

####   

  

  

#### 14.2、站长工具tool

可以看到有些加密/解密功能，例如MD5、url、js、base64加解密等等

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

###   

### **15、whois查询网站及服务器信息**

如果知道目标的域名，你首先要做的就是通过Whois数据库查询域名的注册信息，Whois数据库是提供域名的注册人信息，包括联系方式，管理员名字，管理员邮箱等等，其中也包括DNS服务器的信息。

默认情况下，`Kali`已经安装了`Whois`。你只需要输入要查询的域名即可：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

利用以上收集到的邮箱、QQ、电话号码、姓名、以及服务商，可以针对性进行攻击，利用社工库进行查找相关管理员信息，另外也可以对相关DNS服务商进行渗透，查看是否有漏洞，利用第三方漏洞平台，查看相关漏洞。  
  

### **16、Dig使用**

下载地址  https://www.isc.org/downloads/

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以使用dig命令对DNS服务器进行挖掘。

Dig命令后面直接跟域名，回车即可

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#### Dig常用选项

*   **-c** 选项，可以设置协议类型（class），包括IN(默认)、CH和HS。
    
*   **-f** 选项，dig支持从一个文件里读取内容进行批量查询，这个非常体贴和方便。文件的内容要求一行为一个查询请求。来个实际例子吧：
    
*   **-4** 和**-6** 两个选项，用于设置仅适用哪一种作为查询包传输协议，分别对应着IPv4和IPv6。
    
*   **-t** 选项，用来设置查询类型，默认情况下是A，也可以设置MX等类型，来一个例子：
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#### 跟踪dig全过程

dig非常著名的一个查询选项就是`+trace`，当使用这个查询选项后，dig会从根域查询一直跟踪直到查询到最终结果，并将整个过程信息输出出来

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

###   

###   

### **17、Nslookup用法**

`nslookup`是站长较为常用的工具之一，它甚至比同类工具dig的使用人数更多，原因是它的运行环境是windows，并且不需要我们再另外安装什么东西。dig是在`linux`环境里运行的命令，不过也可以在windows环境里使用，只是需要安装dig windows版本的程序。

`Nslookup`命令以两种方式运行：非交互式和交互式。

本文第一次提到“交互式”的概念，简单说明：交互式系统是指执行过程中允许用户输入数据和命令的系统。而非交互式系统，是指一旦开始运行，不需要人干预就可以自行结束的系统。因此，`nslookup`以非交互式方式运行，就是指运行后自行结束。而交互式，是指开始运行后，会要求使用者进一步输入数据和命令。

#### 类型

*   A 地址记录
    
*   AAAA 地址记录
    
*   AFSDB Andrew文件系统数据库服务器记录
    
*   ATMA ATM地址记录
    
*   CNAME 别名记录
    
*   HINFO 硬件配置记录，包括CPU、操作系统信息
    
*   ISDN 域名对应的ISDN号码
    
*   MB 存放指定邮箱的服务器
    
*   MG 邮件组记录
    
*   MINFO 邮件组和邮箱的信息记录
    
*   MR 改名的邮箱记录
    
*   MX 邮件服务器记录
    
*   NS 名字服务器记录
    
*   PTR 反向记录
    
*   RP 负责人记录
    
*   RT 路由穿透记录
    
*   SRV TCP服务器信息记录
    
*   TXT 域名对应的文本信息
    
*   X25 域名对应的X.25地址记录
    

  

#### 例如

1.设置类型为ns

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

2.下面的例子查询baidu.com使用的DNS服务器名称:  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

3.下面的例子展示如何查询baidu.com的邮件交换记录：  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4.查看网站cname值。  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

5.查看邮件服务器记录（-qt=MX）  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

6.同样nslookup也可以验证是否存在域传送漏洞，步骤如下：  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

### **18、fierce工具**

在进行了基本域名收集以后，如果能通过主域名得到所有子域名信息，再通过子域名查询其对应的主机IP，这样我们能得到一个较为完整的信息。除了默认使用，我们还可以自己定义字典来进行域名爆破。

使用`fierce`工具，可以进行域名列表查询：`fierce -dns domainName`

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

### **19、theHarvester的使用**

`theHarvester`是一个社会工程学工具，它通过搜索引擎、PGP服务器以及SHODAN数据库收集用户的email，子域名，主机，雇员名，开放端口和banner信息。

*   **-d**  服务器域名
    
*   **-l**  限制显示数目
    
*   **-b**  调用搜索引擎（baidu,google,bing,bingapi,pgp,linkedin,googleplus,jigsaw,all）
    
*   **-f**  结果保存为HTML和XML文件
    
*   **-h**  使用傻蛋数据库查询发现主机信息
    

#### 实例1

`theHarvester -d sec-redclub.com -l 100 -b baidu`

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

### **20、DNS枚举工具DNSenum**

`DNSenum`是一款非常强大的域名信息收集工具。它能够通过谷歌或者字典文件猜测可能存在的域名，并对一个网段进行反向查询。它不仅可以查询网站的主机地址信息、域名服务器和邮件交换记录，还可以在域名服务器上执行axfr请求，然后通过谷歌脚本得到扩展域名信息，提取子域名并查询，最后计算C类地址并执行whois查询，执行反向查询，把地址段写入文件。

本小节将介绍使用DNSenum工具检查DNS枚举。在终端执行如下所示的命令：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

输出的信息显示了DNS服务的详细信息。其中，包括主机地址、域名服务地址和邮件服务地址，最后会尝试是否存在域传送漏洞。

使用DNSenum工具检查DNS枚举时，可以使用dnsenum的一些附加选项，如下所示。

*   **--threads [number]**：设置用户同时运行多个进程数。
    
*   **-r**：允许用户启用递归查询。
    
*   **-d**：允许用户设置WHOIS请求之间时间延迟数（单位为秒）。
    
*   **-o**：允许用户指定输出位置。
    
*   **-w**：允许用户启用WHOIS请求。
    

  

### **21、subDomainsbrute二级域名收集**

二级域名是指顶级域名之下的域名，在国际顶级域名下，它是指域名注册人的网上名称；在国家顶级域名下，它是表示注册企业类别的符号。

我国在国际互联网络信息中心（Inter NIC） 正式注册并运行的顶级域名是CN，这也是我国的一级域名。

在顶级域名之下，我国的二级域名又分为类别域名和行政区域名两类。类别域名共7个，包括用于科研机构的ac；国际通用域名com、top；用于教育机构的edu；用于政府部门的gov；用于互联网络信息中心和运行中心的net；用于非盈利组织的org。而行政区域名有34个，分别对应于我国各省、自治区和直辖市。

以上为工具默认参数，如果是新手，请直接跟主域名即可，不用进行其它设置。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

`Python subDomainsbrute.py sec-redclub.com`  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

就可以直接运行，等待结果，最后在工具文件夹下面存在txt文件，直接导入扫描工具就可以进行扫描了。  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### **22、layer子域名检测工具**

layer子域名检测工具主要是windows一款二级域名检测工具，利用爆破形式。

工具作者：http://www.cnseay.com/4193/

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  
  

### **23、Nmap**

`Nmap`是一个网络连接端口扫描软件，用来扫描网上电脑开放的网络连接端口。确定哪些服务运行在哪些连接端口，并且推断计算机运行哪个操作系统。它是网络管理员必用的软件之一，以及用以评估网络系统安全。

#### 功能

*   主机发现
    
*   端口扫描
    
*   版本侦测
    
*   OS侦测
    

#### 部署方式

*   Kail集成环境
    
*   单独安装（使用yum、apt-get工具直接安装）
    
*   PentestBox环境
    
*   Windows版等等
    

  

### **24、DirBuster**

DirBuster是一款路径及网页暴力破解的工具,可以破解出一直没有访问过或者管理员后台的界面路径。Java运行环境+DirBuster程序包

*   双击运行`DirBuster.jar`
    
*   在URL中输入目标URL或者主机IP地址
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  
**25、github信息收集**  
  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  
**26、网盘信息收集**  
https://www.lingfengyun.com/  
http://magnet.chongbuluo.com/  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  
**27、Google Hacking**

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

```
`参考链接：``https://cloud.tencent.com/developer/article/1482443``https://www.cnblogs.com/suendanny/p/9505102.html``https://www.landui.com/help/show-8660.html``https://blog.csdn.net/weixin_40224560/article/details/93189255`
```

  

  

如果觉得文章对你有帮助，请转发并点击右下角“在看”![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)