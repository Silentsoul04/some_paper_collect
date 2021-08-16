\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/wo7ppyly9YTKFaGZVEv3iQ)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ok4fxxCpBb7zdDQqQotVhJhDz7nicvGPuibicUFRZcGUJ1FkHY2vTKMvLucM0cmGH9meG5VkLxdibibDpq03VFq6BVA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

作者：ale\_wong@云影实验室

  

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

前言

迟到的Fastjson反序列化漏洞分析，按照国际惯例这次依旧没有放poc。道理还是那个道理，但利用方式多种多样。除了之前放出来用于文件读写的利用方式以外其实还可以用于SSRF。

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

漏洞概述

在之前其他大佬文章中，我们可以看到的利用方式为通过清空指定文件向指定文件写入指定内容(用到第三方库)。当gadget是继承的第一个类的子类的时候，满足攻击fastjson的条件。此时寻找到的需要gadget满足能利用期望类绕过checkAutoType。

本文分析了一种利用反序列化指向fastjson自带类进行攻击利用，可实现文件读取、SSRF攻击等。

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

调试分析

### **1\. 漏洞调试**

从更新的补丁中可以看到expectClass类新增了三个方法分别为：

java.lang.Runnable、java.lang.Readable、java.lang.AutoCloseable

首先，parseObject方法对传入的数据进行处理。通过词法解析得到类型名称，如果不是数字则开始checkAutoType检查。

```
`if (!allDigits) {` `clazz = config.checkAutoType(typeName, null, lexer.getFeatures());` `}`
```

当传入的数据不是数字的时候，默认设置期望类为空，进入checkAutoType进行检查传入的类。

```


```
`final boolean expectClassFlag;``if (expectClass == null) {` `expectClassFlag = false;``} else {` `if (expectClass == Object.class` `|| expectClass == Serializable.class` `|| expectClass == Cloneable.class` `|| expectClass == Closeable.class` `|| expectClass == EventListener.class` `|| expectClass == Iterable.class` `|| expectClass == Collection.class` `) {` `expectClassFlag = false;` `} else {` `expectClassFlag = true;` `}``}`
```




```

判断期望类，此时期望类为null。往下走的代码中，autoCloseable 满足不在白名单内,不在黑名单内，autoTypeSupport没有开启，expectClassFlag为false。

其中：

A.计算哈希值进行内部白名单匹配

B.计算哈希值进行内部黑名单匹配

C.非内部白名单且开启autoTypeSupport或者是期望类的，进行hash校验白名单acceptHashCodes、黑名单denyHashCodes。如果在acceptHashCodes内则进行加载( defaultClassLoader),在黑名单内则抛出 autoType is not support。

```


```
clazz = TypeUtils.getClassFromMapping(typeName);
```




```

满足条件C后来到clazz的赋值，解析来的代码中对clazz进行了各种判断

```


```
clazz = TypeUtils.getClassFromMapping(typeName);
```




```

从明文缓存中取出autoCloseable赋值给 clazz

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```


```
`clazz = TypeUtils.getClassFromMapping(typeName);``if (clazz == null) {` `clazz = deserializers.findClass(typeName);``}``if (clazz == null) {` `clazz = typeMapping.get(typeName);``}``if (internalWhite) {` `clazz = TypeUtils.loadClass(typeName, defaultClassLoader, true);``}``if (clazz != null) {` `if (expectClass != null` `&& clazz != java.util.HashMap.class` `&& !expectClass.isAssignableFrom(clazz)) {` `throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());` `}` `return clazz;`
```




```

当clazz不为空时，expectClassFlag为空不满足条件，返回clazz,至此，第一次的checkAutoType检查完毕。

```


```
`ObjectDeserializer deserializer = config.getDeserializer(clazz);``Class deserClass = deserializer.getClass();``if (JavaBeanDeserializer.class.isAssignableFrom(deserClass)` `&& deserClass != JavaBeanDeserializer.class` `&& deserClass != ThrowableDeserializer.class) {` `this.setResolveStatus(NONE);``} else if (deserializer instanceof MapDeserializer) {` `this.setResolveStatus(NONE);``}``Object obj = deserializer.deserialze(this, clazz, fieldName);``return obj;`
```




```

将检查完毕的autoCloseable进行反序列化，该类使用的是JavaBeanDeserializer反序列化器，从MapDeserializer中继承

```


```
`public <T> T deserialze(DefaultJSONParser parser, Type type, Object fieldName) {` `return deserialze(parser, type, fieldName, 0);``}``public <T> T deserialze(DefaultJSONParser parser, Type type, Object fieldName, int features) {` `return deserialze(parser, type, fieldName, null, features, null);``}//进入后代码如下`
```




```

```


```
`if ((typeKey != null && typeKey.equals(key))` `|| JSON.DEFAULT_TYPE_KEY == key) {` `lexer.nextTokenWithColon(JSONToken.LITERAL_STRING);` `if (lexer.token() == JSONToken.LITERAL_STRING) {` `String typeName = lexer.stringVal();` `lexer.nextToken(JSONToken.COMMA);` `if (typeName.equals(beanInfo.typeName)|| parser.isEnabled(Feature.IgnoreAutoType)) {` `// beanInfo.typeName是autoCloseable ，但IgnoreAutoType没有开启``          if (lexer.token() == JSONToken.RBRACE) {` `lexer.nextToken();` `break;` `}` `continue;` `}//不满足条件所以这块代码被跳过了`
```




```

JSON.DEFAULT\_TYPE\_KEY 为@type ,并给它赋值传入的key @type ,将第二个类也就是这次 的gadget传入

```


```
`if (deserializer == null) {` `Class<?> expectClass = TypeUtils.getClass(type);` `userType = config.checkAutoType(typeName, expectClass, lexer.getFeatures());` `deserializer = parser.getConfig().getDeserializer(userType);``}`
```




```

期望类在这里发生了变化，expectClass的值变为java.lang.AutoCloseable，typeName为gadget，

```


```
`boolean jsonType = false;` `InputStream is = null;` `try {` `String resource = typeName.replace('.', '/') + ".class";` `if (defaultClassLoader != null) {` `is = defaultClassLoader.getResourceAsStream(resource);` `} else {` `is = ParserConfig.class.getClassLoader().getResourceAsStream(resource);` `//开了一个class文件的输入流` `}` `if (is != null) {` `ClassReader classReader = new ClassReader(is, true);//new reader工具` `TypeCollector visitor = new TypeCollector("<clinit>", new Class[0]);` `classReader.accept(visitor);` `jsonType = visitor.hasJsonType();` `}` `} catch (Exception e) {` `// skip` `} finally {` `IOUtils.close(is);//关闭流 JarURLConnection$JarURLInputStream` `}`
```




```

来到JSONType注解，取typename gadget转换变为路径，resource通过将 “.” 替换为”/“得到路径 。其实已经开始读取gadget了，它本意应该是加载AutoCloseable。

```


```
`public ClassReader(InputStream is, boolean readAnnotations) throws IOException {` `this.readAnnotations = readAnnotations;` `{` `ByteArrayOutputStream out = new ByteArrayOutputStream();` `byte[] buf = new byte[1024];` `for (; ; ) {` `int len = is.read(buf);` `if (len == -1) {` `break;` `}` `if (len > 0) {` `out.write(buf, 0, len);` `}` `}` `is.close();` `this.b = out.toByteArray();` `}`
```




```

可以看到这里有读取文件的功能。所以之前网传的POC可能是利用这里这个特性(?)留意一下以后研究…

```
`if (autoTypeSupport || jsonType || expectClassFlag) {` `boolean cacheClass = autoTypeSupport || jsonType;` `clazz = TypeUtils.loadClass(typeName, defaultClassLoader, cacheClass);` `//开始加载gadget``}``if (expectClass != null) {` `if (expectClass.isAssignableFrom(clazz)) {//判断里面的类是否为继承类` `TypeUtils.addMapping(typeName, clazz);` `return clazz;` `} else {` `throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());` `}``}`
```

isAssignableFrom()这个方法用于判断里面的类是否为继承类，当利用了java.lang.AutoCloseable这个方法去攻击fastjson，那么后续反序列化的链路需要是继承于该类的子类。

TypeUtils.addMapping(typeName, clazz)这一步成功把gadget加入缓存中并返回被赋值gadget的clazz.

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

checkAutoType正式检查完毕，此时用deserializer = parser.getConfig().getDeserializer(userType); userType既gadget进行反序列化。

```


```
`private void xxTryOnly(boolean isXXXXeconnect, Properties mergedProps) throws ``  XXXException {` `Exception connectionNotEstablishedBecause = null;` `try {` `coreConnect(mergedProps);` `this.connectionId = this.io.getThreadId();` `this.isClosed = false;`
```




```

进入coreConnect()

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在这里进行连接。至此漏洞利用完结。

### **2\. 总结：**

在本次反序列化漏洞中，笔者认为关键点在于找到合适并且可利用的常用jar包中的gadget。gadget在被反序列化后即可执行类里的恶意的功能(不仅限于RCE还包括任意文件读取/创建,SSRF等)。也可以使本漏洞得到最大化的利用。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

参考链接

https://b1ue.cn/archives/348.html

https://daybr4ak.github.io/2020/07/20/fastjson%201.6.68%20autotype%20bypass/

（请点击“阅读原文”查看链接）

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "虚线阴影分割线")

  

  

\- End -

精彩推荐

[安全客季刊正式发布 | 戳这里、读季刊赢好礼吧~](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649733249&idx=1&sn=0fe01b8d897e58469422f2e6c94ae925&chksm=888c88eebffb01f8cf4518b2ba6a2219e8ef2bb27ec4460ef33cd83ab6e382344a4981e4de6c&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[通过HackerOne漏洞报告学习PostMessage漏洞实战场景中的利用与绕过](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649733211&idx=1&sn=26d9fdbb40f88a53a1a99b486c8bdfe1&chksm=888c8834bffb01220a8a753e960e94017ec49c735ad349a9c696bd0a228c6ea8b83a22dc3f01&scene=21#wechat_redirect)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[使用无括号的XSS绕过CSP策略研究](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649733148&idx=1&sn=68792baa173530bc3fce1d6be4cc8a0b&chksm=888c8873bffb01654b4a47be657344213027dbe4b717b6bedc79c5f0922bb5809238749b0fe2&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[攻防演练中防守方的骚姿势](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649733170&idx=1&sn=97f261b77fbf92350bab10c1b58ec309&chksm=888c885dbffb014ba94ba4ce13ca96456c0ec72de2c5c18cac09a28735accebafc6c6bd76839&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "虚线阴影分割线")

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**戳“阅读原文”查看更多内容**