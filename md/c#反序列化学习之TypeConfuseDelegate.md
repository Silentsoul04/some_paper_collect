> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2s457-XCm4XSCjK5NsMv6g)

反序列化漏洞在很多语言中都存在。之前一直在研究 java 的反序列化漏洞，但是想检验一下对于 java 反序列化漏洞的理解，于是我学习 c# 反序列化漏洞。当然，我也不是专业 c# 开发，没有正规接触过 c#，所以有些地方在所难免有纰漏。

1. 反序列化基础
---------

对象的序列化与反序列化的作用在于将一个对象的状态从当前 CLR（java 叫 JVM）传输至另外一端 CLR。该功能一般与 rpc 远程调用所搭配以降低开发难度。比如 weblogic 的 T3 rmi 协议，就是客户端将对象通过序列化传输到服务端。然后调用对象的某些方法。序列化的过程中，只存储对象的属性。例如一个学生类。

```
class student{    public String name;    public int age;    public String getName(){        return name;    }}
```

在这里我们先暂时不考虑继承自 serializable 接口。序列化的话，只会传输 name 与 age 这两个属性。至于方法则不会传输。在反序列化的时候，读取到 name 与 age 的属性，并将这两个属性赋值给相应的类生成的对象。你可以认为反序列化的代码如下

```
Class clazz = readClassNameFromTxt();
Student s = clazz.newInstance();
s.name = readStringFromTxt();
s.age = readIntFromTxt()
```

至于怎么存储对象的值，则由序列化的方式决定。例如 xml 序列化组件，fastjson 序列化组件，ObjectInputStream 序列化等等。在 c# 则是 BinaryFormatter 等，原理都是一样。

那么你想问，既然在序列化的过程中传输对象的属性，那么是怎样造成反序列化漏洞的？在反序列化过程中，一般都会调用某个特殊方法以满足特殊的反序列化需求。在 java 中就是 readObject 方法，c# 就是 OnDeserialization 方法。在这些特殊的方法中又调用了某些类的操作，一步一步最终执行代码。这就是反序列化漏洞

和 java 一样，c# 序列化种类需要继承自 ISerializable 接口才可以被序列化。为了保证序列化中还原的对象相同，一般在双方会在反序列化的过程中再次确认被反序列化的对象的类是否是同一个。在 java 是 serialVersionUID 判断，c# 类型版本由 SerializationInfo 中保存的 AssemblyName 决定，其规则遵循 clr 默认程序集发现和加载策略

2. 委托
-----

基本原理懂了，那我们拿`TypeConfusedDelegate`这个 gadget 来分析。在学习这个 gadget 之前，我们了解一下 c# 委托。C# 中的委托（Delegate）类似于 C 或 C++ 中函数的指针。**委托（Delegate）** 是存有对某个方法的引用的一种引用类型变量。引用可在运行时被改变。示例代码如下

```
static int num = 10;      public static int AddNum(int p)      {         num += p;         return num;      }      public static int getNum()      {         return num;      }      static void Main(string[] args)      {         // 创建委托实例         NumberChanger nc1 = new NumberChanger(AddNum);         // 使用委托对象调用方法         nc1(25);         Console.WriteLine("Value of Num: {0}", getNum());      }
```

其实委托就是 c++ 的函数的指针。c# 为了照顾以前 c++ 程序员的开发习惯，但是又不想在 c# 中直接让程序员操作函数指针，于是搞了委托。查看 c# 的字节码，可以看到委托其实是生成一个类，继承自 System.muitiDelegate。调用委托，则是执行这个类的 Invoke 方法。

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdE5rpSiafSjk37diaKxoUWiaR1hxbWZTHkdJfL5iaqAlic9AejOqOYFlKlRWLzRJSkSEeZl2cpWUullFqw/640?wx_fmt=png)

至于这个 Invoke 方法则是运行的时候动态生成的。但是我在查阅 msdn 中，发现一篇 2001 年的杂志介绍了委托的实现原理。

https://docs.microsoft.com/en-us/archive/msdn-magazine/2001/june/net-delegates-part-2

Invoke 方法的伪代码如下

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdE5rpSiafSjk37diaKxoUWiaR1rq106Ggj0xXFUK53f9VT38dX1vO6iaXdT4VUl2pR4w6hlpOia04yOXfA/640?wx_fmt=png)

其实就是执行函数指针所指向的函数。

c# 还支持多重委托。也就是在一个委托中，按顺序执行其中的多个委托。

委托其实与面向对象的多态在软件设计的作用都差不多，大家可以思考一下 c# 与 java 的区别。

3. SortedSet
------------

c# SortedSet 是一个很重要的数据结构。看字面意思就是一个有序的集合。也就是说，再向 SortedSet 添加元素的时候，SortedSet 会调用排序算法并将你的所添加的元素添加到合适的位置。既然这里，你肯定会问，是怎么排序的？所以 c# 的 SortedSet 的构造函数中，有一个参数就是传入排序算法的委托。

代码如下

```
Comparison<string> da = new Comparison<string>(String.Compare);
IComparer<string> comp = Comparer<string>.Create(da);
var sortedSet = new SortedSet<string>(comp);
```

在向 SortedSet 添加数据的时候，排序算法会调用 String.Compare 方法。

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdE5rpSiafSjk37diaKxoUWiaR1sU8kIEUgxyvc3pFSRiaVMtD3ARib4M4x380ahfbn58atTWfRfsAJKzxA/640?wx_fmt=png)

也就是遍历 SortedSet 中所有元素，并一一与添加的元素通过 Compare 对比。

下面我们看一下 SortedSet 反序列化的过程。在 OnDeserialization 方法中，首先在序列化流中还原 Compare，然后再还原 SortedSet 的每个元素，并调用 Add 添加到实例化后的 SortedSet 中。

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdE5rpSiafSjk37diaKxoUWiaR1zUKfUxHYwV2picy5NrPuYFBUUmQn8NeIUiczvOZBNJjsTFvTZ1IprRdQ/640?wx_fmt=png)

到现在我们明白了，既然在 c# 中`Comparison`是一个委托。那么按照 java 中 ysoserial 的流程，首先将正常委托添加到 SortedSet，防止在添加数据的时候不小心触发恶意委托。然后再通过反射修改成恶意委托，将这个 SortedSet 序列化，在反序列化的过程中不就可以触发恶意委托了吗。原理如此，我们继续往下分析反序列化 payload 生成

4. 反序列化 payload 生成
------------------

我们看一下委托的数据结构

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdE5rpSiafSjk37diaKxoUWiaR1QYt1IMrxOquGJZ4F0f2DYBf5SBE7fy1TF17mele8E7VXB9ic3ttN1xA/640?wx_fmt=png)

我们只需要通过反射替换 Method 即可。但是失败了，因为 c# 通过反射去修改委托实在是太麻烦了。稍有不慎就异常退出。但是我们通过 c# 修改多重委托就很方便。在调用多重委托时，会按顺序执行多种委托中的每一个委托。多重委托的数据结构如下

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdE5rpSiafSjk37diaKxoUWiaR1gDDyauSWficicWmqrKTM7G5JfDibq9H1yj8SM2rMfFxT6gLURRQHfC34A/640?wx_fmt=png)

很简单，我们只需要在 sortedSet 添加完数据后，再通过反射，将_invocationList 的中任意一个委托修改成恶意委托即可。c# 中有一个万能委托，可以指向任何函数。只需要替换成他即可。代码如下

```
FieldInfo fi =            typeof(MulticastDelegate).GetField("_invocationList", BindingFlags.NonPublic | BindingFlags.Instance);        object[] invoke_list = d.GetInvocationList() ;        // Modify the invocation list to add Process::Start(string, string)        invoke_list[1] = new Func<string, string, Process>(Process.Start);        fi.SetValue(d, invoke_list);
```

最终成功弹出计算器，堆栈如下

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdE5rpSiafSjk37diaKxoUWiaR1wGnFIWhibAaiaLCXPv8MSibmnB4gLicRttPheW8L5zal5xe1BVQSWSDhew/640?wx_fmt=png)

相关学习资料稍后上传至星球

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdEjSODheHaAokPQRjTKp7tI2r4IYIUKcqDicftqmvObxd3vkwRhaODMias2tsGEt2InTSWd4p8sPezQ/640?wx_fmt=png)