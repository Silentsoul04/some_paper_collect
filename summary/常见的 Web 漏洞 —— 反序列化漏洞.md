> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sLUEKLW2Qm9AayyGhCBk8A)

前段时间因各种原因，终于下定决心好好总结一下反序列化漏洞，耗时两周，且看且珍惜！  

DESERIALIZATION.TIME.

**#1**

**漏洞简介**

Vulnerability Introduction

*   **序列化**：把对象转换为字节序列的过程，即把对象转换为可以存储或传输的数据的过程。例如将内存中的对象转换为二进制数据流或文件，在网络传输过程中，可以是字节或是 XML 等格式。  
    
*   **反序列化**：把字节序列恢复为对象的过程，即把可以存储或传输的数据转换为对象的过程。例如将二进制数据流或文件加载到内存中还原为对象  
    

反序列化漏洞首次出现在 2015 年。虽然漏洞较新，但利用十分热门，主要原因还是太过信任客户端提交的数据，容易被开发者忽略，该漏洞一般都可执行任意命令或代码，造成的影响较大。

**#2**

**漏洞成因  
**

Causes Of Vulnerability

在身份验证，文件读写，数据传输等功能处，未对反序列化接口做访问控制，未对序列化数据做加密和签名，加密密钥使用硬编码（如 Shiro 1.2.4），使用不安全的反序列化框架库（如 Fastjson 1.2.24）或函数的情况下，由于序列化数据可被用户控制，攻击者可以精心构造恶意的序列化数据（执行特定代码或命令的数据）传递给应用程序，在应用程序反序列化对象时执行攻击者构造的恶意代码，达到攻击者的目的。  

**#3**

**漏洞位置**

Vulnerability Location

1.  解析认证 token、session 的位置  
    
2.  将序列化的对象存储到磁盘文件或存入数据库后反序列化时的位置，如读取 json 文件，xml 文件等
    
3.  将对象序列化后在网络中传输，如传输 json 数据，xml 数据等
    
4.  参数传递给程序
    
5.  使用 RMI 协议，被广泛使用的 RMI 协议完全基于序列化
    
6.  使用了不安全的框架或基础类库，如 JMX 、Fastjson 和 Jackson 等
    
7.  自定义协议用来接收与发送原始的 java 对象  
    

**#4**

**漏洞原理及实验**

Vulnerability Principle

在 Python 和 PHP 中，一般通过构造一个包含魔术方法（在发生特定事件或场景时被自动调用的函数，通常是构造函数或析构函数）的类，然后在魔术方法中调用命令执行或代码执行函数，接着实例化这个类的一个对象并将该对象序列化后传递给程序，当程序反序列化该对象时触发魔术方法从而执行命令或代码。在 Java 中没有魔术方法，但是有**反射（reflection）机制**：在程序的运行状态中，可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法，这种动态获取程序信息以及动态调用对象的功能称为 Java 语言的反射机制。一般利用反射机制来构造一个执行命令的对象或直接调用一个具有命令执行或代码执行功能的方法实现任意代码执行。

**01**

**Python 反序列化漏洞实验**

以 pickle 模块为例，假设浏览器传递序列化后的 Cookie 给服务器保存，服务器经过一些处理后反序列化还原 Cookie：  

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRTC1ia3ucTnuIj2VscMgHAs5YzNiaUaO3GQTOOtc5iaKseY74OHeZrmbibQ/640?wx_fmt=png)

程序正常运行时，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRA9eOstEUGCpH5vznic9f8kiaWiciazg0NmgQoDFEcfT2QfMYIVYfvVosLQ/640?wx_fmt=png)

  
利用 pickle 模块和魔术方法__reduce__生成执行命令的 Payload：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRHCkD1Xg4sGResFfciaVwTqmjLUS7ib3KnQGSMVefk9yiaJDENurc99b0g/640?wx_fmt=png)

  
生成执行 whoami 命令的 Payload，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRL36HQalghiaBA9PiaJIQABXOlnRTl7I5bIYSeib5byuFpZSIn6JA5NGvQ/640?wx_fmt=png)

  
使用执行 whoami 命令的 Payload 替换序列化后的 Cookie 的值模拟 RCE 漏洞利用，当正常程序反序列化 Cookie 值时生成包含__reduce__函数的 exec 类，从而执行命令：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRkDaBn7SUicZvXTlrcyDorXfoicEBDGGqlzJVCQqvaQKXI2FibbJsxp96w/640?wx_fmt=png)

  
程序运行结果，如图：  

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRZVlFqicrZHhUqjPw6HAxk6cvr3diapvZKc1OmlD9t5jRBBSsEV4X4TMQ/640?wx_fmt=png)

**02**

**PHP 反序列化漏洞实验**

PHP 中通常使用 serialize 函数进行序列化，使用 unserialize 函数进行反序列化。

**serialize 函数输出格式**

*   NULL 被序列化为：N  
    
*   Boolean 型数据序列化为：b:1，b:0，分别代表 True 和 False
    
*   Integer 型数据序列化为：i: 数值
    
*   String 型数据序列化为：s: 长度:” 值”
    
*   对象序列化为：O: 类名长度: 类名: 字段数: 字段
    

输出的数字基本都是代表长度，在构造 Payload 时需要注意修改长度。

**PHP 中常用魔术方法**

*   __construct：当对象被创建时调用
    
*   __destruct：当对象被销毁前调用
    
*   __sleep：执行 serialize 函数前调用
    
*   __wakeup：执行 unserialize 函数前调用
    
*   __call：在对象中调用不可访问的方法时调用
    
*   __callStatic：用静态方法调用不可访问方法时调用
    
*   __get：获得类成因变量时调用
    
*   __set：设置类成员变量时调用
    

使用下面代码创建一个类 A 并实例化一个对象 a，然后输出序列化对象 a 后的值：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRANdW9icPUenRcZbAjJcicM7DD9ZEs2ibn7A0CEEGDxYrAiaF5wLWtOVz0w/640?wx_fmt=png)

  
序列化对象 a，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRzGibhu87nm9yiaJCiaQ65EMrn0BYGAFtAqWM2ic8BsU7GLF4fQeAjTL0fA/640?wx_fmt=png)

  
PHP 中序列化后的数据中并没有像 Python 一样包含函数__construct 和 print 的信息，而仅仅是类名和成员变量的信息。因此，在 unserialize 函数的参数可控的情况下，还需要代码中包含魔术方法才能利用反序列化漏洞。

使用下面代码定义一个包含魔术方法__destruct 的类 A，然后实例化一个对象 a 并输出序列化后的数据，在对象销毁的时候程序会调用 system 函数执行 df 命令，然后通过 GET 方法传递参数 arg 的值给服务器进行反序列化：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjR1qTod7fyz5h5SrtRuzOK3qvDvWCDhbAiadJoBNntYEOL9vlBjPicNrRg/640?wx_fmt=png)

  
不传入 arg 参数时，服务器返回对象 a 序列化后的数据和 df 命令执行的结果，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRiaJDr3VjmGCnsxHnlROIwjzYkSt2C40ntjDS9pLibMhLwDrJt0Ticbp4A/640?wx_fmt=png)

  
利用对象 a 序列化后的值构造执行 id 命令的 Payload：O"A"{s"test";s"id";}，通过 arg 参数提交之后，在反序列化的过程中成功覆盖变量 test 的值为 id，并在对象销毁时执行命令，如图：

  
当然，现实环境中几乎没有这样方便的攻击链，需要花不少时间去寻找 POP 链。

可参考

*   PHP 反序列化入门之寻找 POP 链（一）
    
*   PHP 反序列化入门之寻找 POP 链（二）
    

**03**

**JAVA 反序列化漏洞实验**

Java 中通常使用 Java.io.ObjectOutputStream 类中的 writeObject 方法进行序列化，java.io.ObjectInputStream 类中的 readObject 方法进行反序列化。使用下面代码将字符串进行序列化和反序列化：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRcz4eue0dBO2WH7stIS7UQk7vkyAAe1Q7dteOOXCLO440xLGHhQgpkQ/640?wx_fmt=png)

  
程序执行后生成 a.ser 文件，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjR41KPkw72HVichEmNZ0wiaHiaLKzjdyFCySoVLNSc88g7lsDnM7Ln92orw/640?wx_fmt=png)

  
以十六进制查看 a.ser 文件内容，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRVyhlvKfHdLqibhia1aIal8jyBnPoTXvOH30vicw23tjnIXFG2T8yaXdsw/640?wx_fmt=png)

  
Java 序列化数据格式始终以双字节的十六进制 0xAC ED 作为开头，Base64 编码之后为 rO0。之后的两个字节是版本号，通常为 0x00 05。

  
一个 Java 类的对象要想序列化成功，必须满足两个条件：

1.  该类必须实现 java.io.Serializable 接口。
    
2.  该类的所有属性必须是可序列化的，如果有一个属性不是可序列化的，则该属性必须注明是短暂的。
    

使用下面代码将对象序列化后存储到 a.ser 文件：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRhC1UJdgiay34ODwXHh0aqFwatF9A0ib9gJrpL4cHPzicnNoiboEuaxiaibRw/640?wx_fmt=png)

  
执行程序后生成 a.ser 文件，以十六进制格式查看文件内容，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRwyoSn6PibSicQmzAcpNUKAFia7xPUNTPQmVWCjmsQb2K1pibicicQcMicaz4Q/640?wx_fmt=png)

  
最后 5 个字节分别为字符串长度和 calc 的 ASCII 值。因此，修改文件为下图所示，即 notepad 的 ASCII 值和长度：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjR7ssH3OL0Md0q9WBMicQjPrYT07mH83qPBHYqibic9jwpRsj5YDnJia72IQ/640?wx_fmt=png)

  
使用下面代码进行反序列化对象：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRKySBibsOWXYeRibpmHfRUFyfn7LfqHe0owibUlneKcVM8nI8fBSUshNEg/640?wx_fmt=png)

  
程序执行后成功运行 notepad，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjR7UKL1gCSSZbKvqic95IrIGwbyLZpnnnfiajmXn3SnoURC5Yd8SHqfu0A/640?wx_fmt=png)

  
现实环境中也没有这样方便的攻击链，需要去寻找 POP 链。

可参考

从反序列化到命令执行 – Java 中的 POP 执行链

**04**

**FastJson 反序列化漏洞实验**

FastJson 作为史上最快的 Json 解析库应用也十分广泛，在 1.2.69 版本以下，其 AutoType 特性在反序列化过程中会导致反序列化漏洞，这个特性就是：在对 JSON 字符串进行反序列化的时候，会读取 @type 参数指定的类，然后把 JSON 内容反序列化为此类的对象，并且会调用这个类的设置（setter）方法。

**实验环境**

*   前端采用 json 提交用户名密码
    
*   后台使用 fastjson 1.2.24 版本
    
*   源码和 WAR 包 GitHub 地址
    

创建一个 User 类，用于查看序列化数据格式，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjROoiaJONGmE5MQGibcSEKiajRYh1XCrxa1xwhpEHCEXsnd2ibMHoBSMqTHQ/640?wx_fmt=png)

  
创建一个 home 类用于输出 user 对象的序列化数据，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRAoFJiajl9E1WufrHOsQ4jKRTHia0ibMPhVfLbKH7bEJxuVLJ9fw8TiaF1Q/640?wx_fmt=png)

  
创建一个 login 类用于获取前端页面提交的 json 格式用户名和密码数据，并使用 JSON.parseObject 方法进行反序列化解析 json 数据，在后台可看到提交的数据，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRYfggT8ph1wcTpowtsibjthIY2fYPm3vWXl4ItZmRmIsicF8OO1icN7CJg/640?wx_fmt=png)

  
访问 home 页面可直接获取 user 对象序列化后的结果，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjR2PTkh4wMs6iaAftRI6HdtD9kR3MskGSW64iaicmsEFouGNLoCaC9HjlyQ/640?wx_fmt=png)

  
@type 的值为对象所属的类，user 和 passwd 分别为对象的用户名属性和密码属性。因此可以利用 AutoType 特性，构造一个使用 @type 参数指定一个攻击类库，包含类属性或方法的 JSON 字符串提交到服务器，在反序列化时调用这个类的方法达到执行代码的目的。通常使用 java.net.Inet4Address 类或 java.net.Inet6Address 类，通过 val 参数传递域名，利用 DnsLog 进行漏洞检测，即：{"@type":"java.net.Inet4Address","val":"DnsLog"}。在登录页面输入用户名和密码提交，拦截数据包，修改提交的 Json 数据，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRHLSJY99xOtjUWPm8L4brVVGenmfGyicWribl2X8FXlUlU4fZ9pJyYnjQ/640?wx_fmt=png)

虽然服务器返回错误信息，但 Payload 仍然被成功执行，在 DnsLog 网站可以看到解析记录，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRma5Kcjy4zfEwILvAL7k0JlWd1FLPy2OlMPHsDbKvyLoXHQHANYHE4A/640?wx_fmt=png)

  
要执行命令需要构造新的 POP 链，常用的 POP 链：

*   基于 JNDI 注入
    
*   基于 ClassLoader
    
*   基于 TemplatesImpl
    

由于本实验仅使用最小依赖编写，此处不再详细分析 POP 链。

可参考

*   Java 安全之 FastJson JdbcRowSetImpl 链分析
    
*   Fastjson 反序列化之 TemplatesImpl 调用链
    

**05**

**ASP.NET 反序列化漏洞实验**

.NET 框架包含多个序列化类：

*   BinaryFormatter
    
*   JavaScriptSerializer
    
*   XmlSerializer
    
*   DataContractSerializer
    
*   。。。  
    

本实验以 XML 序列化和反序列化为例。  

**实验环境**

*   采用 Xml 提交数据
    
*   使用. NET Framework 4.6.1
    
*   完整源码 GitHub 地址
    

使用下面代码定义一个 Test 类，包含执行 ipconfig 命令并返回执行结果的函数 Run，使用 XmlSerializer 类将对象序列化后输出到页面：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRrWan6jsMInHheLiaxNgbQm2Xq16eu68X3BEMamkYAFm9kfdzZfsp4cg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjR7Tric1rOyic4DI177rtwzyKBIricibSykXbTX4WkyquUNhXuIOvd50zqQg/640?wx_fmt=png)

  
使用下面代码将提交的 XML 数据反序列化，并执行对象的 Run 函数：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRkgkxG7Z6C6Aew6G2ia7ichicribn3u0BicXmlxlWrYc69UcYw7G5vkhvbpQ/640?wx_fmt=png)

  
正常情况下访问页面，返回序列化后的数据，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRIX1Jgml2mtM2yGaUwy3xwZuGxRMvM16YWhFXDRCjB7vaTEXJStKZpg/640?wx_fmt=png)

  
点击查看 IP 按钮后，客户端提交数据，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRTm2Bqiajf6jzDGo6HKbefWI7p5rn1NXJC8X9qMb4OP6TvCAUawHbktQ/640?wx_fmt=png)

  
服务器执行命令后返回到客户端，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRnejhxjyKNh6SLoxJmn4TnwibXG1Jce1YHpkKL0EskNpnkQDibAj2m0Bg/640?wx_fmt=png)

  
如果攻击者将传输的 XML 数据进行篡改，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRyhdWicABA4HrhgWXxSqmK358Z3sico9KLcQg7iaWNbFWRmBWlGB1Yfo0Q/640?wx_fmt=png)

  
服务器在反序列化后执行 whoami 命令，如图：  

![](https://mmbiz.qpic.cn/mmbiz_png/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRufoML55qPiaE0XeiaUV66LNlibWv3bvno5O8meASQdnBicAWic5dlXqVJAw/640?wx_fmt=png)

**#5**

**防御方法**

Defense Method

*   对反序列数据加密或签名，且加密密钥和签名密钥不要使用硬编码
    
*   对反序列化接口添加认证授权
    
*   设置反序列化服务仅在本地监听或者设置相应防火墙策略
    
*   禁止使用存在漏洞的第三方框架库
    
*   过滤、禁用危险函数
    
*   过滤 T3 协议或限定可连接的 IP
    
*   设置 Nginx 反向代理，实现 t3 协议和 http 协议隔离
    

**#6**

**常用工具**

Common Tools

*   Java 反序列化工具 YSoSerial.jar
    
*   PHP 反序列化工具 PHPGGC
    
*   .NET 反序列化工具 YSoSerial.NET
    

  

  

**参考文章**

  

1.  深入理解 JAVA 反序列化漏洞
    
2.  Java 反序列化漏洞从入门到深入
    
3.  Java 反序列化漏洞分析
    
4.  从反序列化到命令执行 – Java 中的 POP 执行链
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/ylnlTiawybAdhlTwONB6icP1tARyiaBGjjRFB5hibnKiatEIvhcee9F9MJIy8v081jae7BrUwfnufgIqqiccjMDLGpng/640?wx_fmt=jpeg)

DESERIALIZATION.TIME.

**微信公众号** **|** 墨守安全

**官方网站 |**moushou.org.cn