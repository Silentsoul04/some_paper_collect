> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dGeSj26QrjRg3I6vIREwwA)

你说什么最难受，是相爱的人见不了面，还是最爱的人在别人身边。。。

----  网易云热评

文章来源：蚂蚁非攻安全实验室 、先知白帽大会

一、序列化与反序列化 

1、定义：序列化是用于将对象转换成二进制串存储, 对应着 writeObject，反序列正好相反, 将二进制串转换成对象, 对应着 Freadobject

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdRib34VKm2j7CTshr34zE57yRCAKK0uwoVSbgcm66bCRQnxNyMGU4uGg/640?wx_fmt=png)

2、各编程语言都存在：

Java：java.io.Serializable 接口、fastjson、jackson、gson

PHP：serialize()、 unserialize() 

Python：pickle

3、使用场景

http 参数，cookie，sesion，存储方式可能是 base64(rO0），压缩后的 base64(H4s),MII 等

Servlets http,Sockets,Session 管理器，包含的协议就包括：JMX,RMI,JMS,JND1 等 (\xac\Xed)

xm lXstream,XmldEcoder 等（http Body:Content-type: application/xml）

json(jackson,fastjson)http 请求中包含

二、Java 反序列化过程 

1. 对象实例化

sun.misc.Unsafe#allocateInstance 

通过反射调用构造函数 

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPd4PRrNiaIibV7V2Jf7YpVSsyvHLqrAlHSia1NWj4jqVYuDt9EIMvazlb6Q/640?wx_fmt=png)

2. 成员变量还原

Setter 和 getter 方法

通过反射直接设置

成员变量的处理 (例如：PriorityQueue)

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdXyaiaRC7FZNrn4xjfjPoCRCmtvUFdu30nlngXNjVKdU3RyRiby4fMhibQ/640?wx_fmt=png)

三、Java 反序列化漏洞（PriorityQueue） 

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdZq451iaG4fH1oneJHBMic6ooxeMoGeyGAlic9Zibq8uY17Sx87jWMwqOIg/640?wx_fmt=png)

四、Java 反序列化漏洞挖掘

1、寻找一个类，通过构造一个对象，使其在被反序列化时能执行到危险（sink）方法。 

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdHGHXjC6jkBtI27oKiawNSppV2Vicicet4I7cTZoTDTqFh533vbEQpnm9w/640?wx_fmt=png)

2、寻找一个类，存在可能的执行路径，从反序 列化入口（source）方法执行到危险（sink）方法自动化搜索）

3、构造这个对象，使危险（sink）方法参数可控。（手工打造）

五、 自动化挖掘实现

1、在静态分析中，这是一个典型的可达性分析问题。

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdQVvmTM6NUc5tdGVDMpaSmSib4icyudogrVmImlaxg9GadQ46KvaYrI0Q/640?wx_fmt=png)

2、 可达性分析 - may 分析：无需绘制控制流图，只需搜素调用树。

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdcL73TH1bTe6vgqrj28Ifqrib013aXdEPtSE6n3QB2dHoGJU35btlEfQ/640?wx_fmt=png)

六、、 调用树搜索实现

1、深度优先搜索（DFS） vs 广度优先搜索（BFS）

调用路径越长，payload 越难构造 ；搜索深度有限 ；等价于搜索一个 n 叉树（n>100）的前几层；调用链的存储

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdFmj7dgXr0r2zAzfKOGveeM2WicMMgWicmiasMta4dTvNWVo2quTzgB8MQ/640?wx_fmt=png)

2、深度优先搜索（DFS）

搜索停止条件：到达指定深度；搜索到 sink 方法

搜索结果保存：使用 stack 保存路径

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdMqCUic3G3icIvZtnU3U7ZJlWPBkDD6qedEsw4MG69sTibP9lSEbDsD6YQ/640?wx_fmt=png)

七、搜索中的多态问题

1、由于面向对象中多态性的存在，只有在运行时 才能确定调用哪个子类的 eat 方法。

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdCLFPJSGiazUTf2GMROwQHqAkajO4Aic0rbkWQ4LvFQw0tYkMg0Y0Q3ibA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdFHbjWukDC5o7kiaFyTia7MkGp2eKPZQTxw3QrNvWP91uNQbPbGvLZF5g/640?wx_fmt=png)

2、多态的处理

构建类、接口和方法继承树（双向树）

寻找调用的方法的实现所在类的所有子类集合

在上述集合中寻找调用者类的子类的集合

这些子类中重写的方法即为所有可能调用的方法

八、路径成环

搜索到 CircleChain 的 hashCode 方法时，这个方法调用了 Object#hashCode 方法，寻找 Object 的子类会再找到 CircleChain

类，形成环。

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdnyEarItzjPpYQe1QKwW9WuiawrX2EbZIToA5qVSckiaPr0jQh6h6Vib2w/640?wx_fmt=png)

九、路径爆炸

以下方法的实现会造成路径爆炸

1、Java.util.List#get 方法 

2、Java.lang.Object#toString 方法

3、java.util.Iterator#hasNext 方法

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdwTQOtexEhR9Wa5nJaH5rMKmJVty9DTFOXrgliaGmcibeXJkfLDuOtkjw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdJTzNQdzJajQRbhT5IbLPGJicun8bDlVpFq5lJDy6YmARicQqNSr9wboQ/640?wx_fmt=png)

十、路径爆炸成环问题解决

1. 搜索深度限制（兜底）

2. 已搜索方法缓存

1. 先缓存、后搜索

2. 缓存方法 signature

4. 调用链缓存

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdSYGB0TVeHicpU5yuabqH77JziaHicDk0aj6RLrcGFZDm7ibQnUuy3ia4bnA/640?wx_fmt=png)

只需要构造 C 方法执行时的上下文，使其与链 2 一致即可

十一、Jackson 反序列化漏洞挖掘

1、简介

Jackson 是一个开源的 Java 序列化与反序列化工具，可以将 java 对象序列化为 xml 或 json 格式的字符串，或者反序列化回对应的对象，由于其使用简单，速度较快，且不依靠除

JDK 外的其他库，被众多用户所使用。

Jackson 也是 Spring MVC 默认的 json 解析库，打开多态之后，jackson 会根据 json 中传入的类名进行反序列化

相比其他后来开发的 json 解析库来说，jackson 有灵活的 API，可以很容易根据需要进行扩展和定制。

2、Jackson 历史漏洞

CVE-2017-7525：RCE

CVE-2017-17485：RCE

CVE-2018-14718：RCE

CVE-2019-12086：任意文件读取 

CVE-2019-12384：RCE（要求反序列化后再序列化 payload） 

CVE-2019-14379：RCE （要求反序列化后再序列化）

3、Jackson 反序列化过程

对象初始化：

调用类的无参初始化方法

调用包含一个基础类型参数的构造函数，并且这个参数可控

对象中成员变量赋值：

将 json 看成 key-value 对，key 与 field 不一定一一对应。

首先看 key 是否存在 setter 方法，如果存在 setter 方法，则会通过反射调用 setter 方法

否则看在这个类中是否存在与 key 同名的 field，如果存在，则通过反射直接赋值。

否则看是否存在对应的 getter 方法，且 getter 的返回值是 Collection 或者 Map 的子类，如果满足这个条件，则会调用这个 getter 方法

如果以上条件都不满足，则抛出异常

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPdd8v7TRsQIrLVZ22r43Y632yWEmeakmFoBPI8WlIRxibMEye3R6KyaoQ/640?wx_fmt=png)

4、反序列化的 source method

Jackson 反序列化显式调用的方法：仅包含一个基本类型参数的构造函数；Setter 方法；返回值是 Collection 或者 Map 的子类的 getter 方法；

反序列化过程中隐式调用的方法：hashCode；compare

5、Jackson 反序列化的 sink method

命令执行：

• java.lang.reflect.Method#invoke

• javax.naming.Context#lookup

• javax.naming.Context#bind

• java.lang.Runtime#exec

• java.lang.ProcessBuilder#ProcessBuilder

文件读取：

• java.sql.Driver#connect MySQL 客户端任意文件读取

• org.xml.sax.XMLReader#parse

• javax.xml.parsers.SAXParser#parse

• javax.xml.parsers.DocumentBuilder#parse

6、Jackson 反序列化漏洞搜索结果

CVE-2019-12086：

com.mysql.cj.jdbc.NonRegisteringDriver#connect(String, Properties)-->

com.mysql.cj.jdbc.admin.MiniAdmin#MiniAdmin(String, Properties)-->

com.mysql.cj.jdbc.admin.MiniAdmin#MiniAdmin(String)

CVE-2017-7525：

com.sun.jndi.toolkit.url.GenericURLContext#lookup(String)-->

javax.naming.InitialContext#lookup(String)-->

com.sun.rowset.JdbcRowSetImpl#connect()-->

com.sun.rowset.JdbcRowSetImpl#setAutoCommit(boolean)

javax.xml.parsers.SAXParser#parse(InputSource, DefaultHandler)-->

org.mortbay.xml.XmlParser#parse(InputSource)-->

org.mortbay.xml.XmlConfiguration#XmlConfiguration(String)

CVE-2019-12814

com.sun.xml.internal.fastinfoset.sax.SAXDocumentParser#parse(InputSource)-->

org.apache.xalan.processor.TransformerFactoryImpl#newTemplates(Source)-->

org.jdom.transform.XSLTransformer#XSLTransformer(Source)-->

org.jdom.transform.XSLTransformer#XSLTransformer(String)

禁止非法，后果自负

欢迎关注公众号：web 安全工具库

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPd9334VzZl1k8QJRIKthfF6t9SDjUibCraqviciaJibQTabby26VTlbib1pGA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibsYSVyL0Rbreapj5IWZahPddpG6656UDPQx4a7cd8LavuLS5o0dmypY55dE36rlnHib9daicerZ9qHQ/640?wx_fmt=jpeg)