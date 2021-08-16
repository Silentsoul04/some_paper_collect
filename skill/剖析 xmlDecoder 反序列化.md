> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/svJQ6R-VMzlTMuvgwumQiQ)

这是 **酒仙桥六号部队** 的第 **130** 篇文章。

全文共计 1727 个字，预计阅读时长 6 分钟。

一直想写个代码审计的文章，和大腿们交流下思路，正好翻 xxe 的时候看到一个 jdk 自带的 xmlDecoder 反序列化，很具有代表性，就来写一下，顺带翻一下源码。

为什么选这个呢，因为 ta 让 weblogic 栽了俩跟头，其他都是手写了几个洞，被人发现了，weblogic 是调用的东西存在一些问题，有苦没处说啊，下面剖析下 xmlDecoder 是怎么反序列化的。

**前期准备**

这次使用的是 idea 来调试代码，下面是用到一些快捷键：

Idea 中用到的 debug 快捷键：

F7 进入到代码，

Alt+shift+F7 强制进入代码

Atl+F9 执行跳到下一个断点处

F8 下一步

代码中有提到 invoke(class, method) 方法：

拿例子说话：

methodName.invoke(owner,args)

其中 owner 为某个对象，methodName 为需要执行的方法名称，Object[]  args 执行方法参数列表。

楼主使用的 jdk 版本：

1.8.0_151

**敲黑板开始了**

先整一个完整的 xml 文件，注意箭头的地方，后面会是个小坑。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicujgW3nE5qu2QViaCclgsYsmEeNzibo0AC4Se3WzHHYCF2LKzC2ia08vbhA/640?wx_fmt=png)

使用 java 代码解析 xml 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuUhvqlJFjYBL9CCjia9fodK9SP5T56q1twMELJM9dd9NLYUSOjyQXX9A/640?wx_fmt=png)

重点在 Object s2 = xd.readObject(); 这行代码，打断点跟一下源码。

Debug 模式启动：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicurfEAmFrHueUtIWFLHOhhmM43R0iaKdmib0295TRMhXZR6eGWlav4Yc4w/640?wx_fmt=png)

进入方法，是个三目运算：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicujtnymKtMwkpJnAYhKy3f1ctDaYjRUicQtr5QiaeicGpwMg7zvlIjk1RuQ/640?wx_fmt=png)

进入方法：

注：从这里开始，可以进行打断点，第一次跟不对的时候，下次再 debug 的时间 alt+f9 快速跳到断点处。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuhJsss9W6LlNVRshUDkGpzS4cicibZvFfqnsicJpEz3FuWvRgPnzBrPmVA/640?wx_fmt=png)

打个断点，进入方法：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuRKGqh6kH4sK9dEnBMSwCwaBw8juic8lXqrZluF3ND7Sj2eyd2j8BwhQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuBOE67o2ub0VQ04rP1k4LQs4iay5xXrp7ngmzoHqOC1wa1XBtxQDTutw/640?wx_fmt=png)

SAXParserImpl 中有一些配置，其中的 xmlReader 是前面已经设置过了，是接口对象 new 的实现类，我们看的是实现类，这里有 idea 可以自动进行跳入对应的实现类的方法。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuZgUXu80MibHnIb35wXMZicRTxZCyw918kjWsIImgM9nF4h2nSRGbQDEw/640?wx_fmt=png)

父类：

注：Super：调用父类的写法，有 super 的类必定继承（extend）了其他类。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuzicbANibMjuibfoYoZh7tqCZZpm6LDY4XYbaOpCyDpjr2jf3icaCMpV2wQ/640?wx_fmt=png)

进入父类：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicua36zkHG9QoG3J4dga91gXxvosusKYxfCW7W5hWyicCK0FZdtZnyWJtw/640?wx_fmt=png)

跟进：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuguWlnS55Qu2JNicd1MR7Y3RibclrNpAXSIGybdSRWU9Vp3VvPSPWVTmQ/640?wx_fmt=png)

继续跟进，跳到 XML11Configuration 的 parse（）方法：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuj9bOic2icYw48pPIhKzN721rw6icW2YY3pNum94ibOwxb5TULRw4OUr0UQ/640?wx_fmt=png)

一些配置：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuaDWxPh8fmRsehicOajovcmdkRkM9cibic2aaLbGtvNJo2JDrxHiaVTuMmg/640?wx_fmt=png)

F7 继续跟进：会进入本类的 parse 方法。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuWQvmkueNLMibdkZN4e71PCC6q5rhwDbk1NQCqqGYvoMTe3ico6rvgXyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicu651n9DBBXsxmjf0JibzlTIIBXKudNjbPx6icGnzfPxia66dlxNsh8m41g/640?wx_fmt=png)

进入 XMLDocumentFragmentScannerImpl 后，会看到有方法中进行了 do{}while{} 方法，其中的 next 方法是重点。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuP1EnqEkHX6Z3O5UQWEsP76iau4R4kK5XZMGdYjbG2VPvvRfGK2PW5EQ/640?wx_fmt=png)

跟进，跳到 XMLDocumentScannerImpl 的 next()：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicugFDBT1ssoZpdKibLWImd0ibn5J4z4VYXOFP8KRnI1zP3jPyBYVvl1R5g/640?wx_fmt=png)

进入 next() 方法：

在 do{}while{} 里循环多次。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuGg51FWuMCl56EHxUabekqJHXW9HdfKrYrrXcQl1wiakaLkv01xSgUAA/640?wx_fmt=png)

注：下面的显示台有变量的值，可以看到代码中变量值的变化。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuiaXcyFumbPcGcOsAaVI7DkV7DcIfYeSbGFJeh9DfVZXKafiaSl7L1lKg/640?wx_fmt=png)

继续跟进：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuE028gyHLB5mRyYhibwibgHwynSPF4evsBMedbhxoAE85ZhxB8xJ3kyhg/640?wx_fmt=png)

可以看到解析 xml 文件的时候有解析到 calc 字符，继续跟进。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicukFnN8Nuo22SQkiaicXqgwFJicSQqf9uTyKnzP3QNfRv5bwuGCWBniapWOA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuNXAxkQiaMhj42IoVfJBIdhRvEVI8WadJQT8FQpZiaoEyY6cnx9cTypzg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuOReaxe6AzjrkiaZLDj5NESDAOhnmLfmFQAQUo2QqXZPib4lib4EmdYQoA/640?wx_fmt=png)

ProcessBuilder 这个类在 xml 文件中有申明，然后到了 invoke，成功执行命令。

这一块代码建议亲自跟一下，会跟到很底层的东西，楼主在这一块卡了好长时间。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuLXibjNRhAATAxk2HRLICocblUjk4IdzVseic1pVMMvib7MxnNJWFehbrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicubGQxEh4icusVmmtaVcb51A5EUxxC23Cibakmc7Efkjvypk7t2wau0Y4w/640?wx_fmt=png)

这次是借助了 idea 进行了代码的跟踪，待到能手点方法跟踪代码的那天就是楼主神功大成之日！嘎嘎嘎~

记得好多开发大牛说过，想进步，多看看 jdk 源码，看懂 ta，打遍天下无敌手！（后面一句我吹的）

代码审计的时候不一定能搭的起环境来，基本功还是很重要的，看 jdk 源码就是一个很好的练习的方法。

用 jdk 自带的洞来练习代码审计的好处就是，可以使用 idea 帮助寻找跳转方法，不会有跟不下去的时候，门槛会降低很多；再有 cms 会有很多奇奇怪怪的写法，出现了洞的话最后还是一些基本的写法，楼主建议还是从基础的洞来入手，没有那么高的复杂度。

**编辑利用程序**

写一个方法，将 xml 文件拼接起来，还记得开头提到的小坑么

注：楼主最喜欢这种洞了，就像网站本身就给开了个后门一样。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuatehibBHia4tnc7QPwy8yZfVWCQIB5luUibV9GFVaFCwLTQEzsBPccaxw/640?wx_fmt=png)

Main 方法调用，试试 ping 命令。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicuKVNPfkJYjPcAVuQp0aKfjlEua45apLDiaN62gF77JAHjRNXWg3rRswg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54icDgzjQ59KRLpDJia7m8vicu8fZJ67b8gkqXdbaEpG9S2xIPyYZajnFtFNKjLicicDbV96HJvHXOqibqQ/640?wx_fmt=png)

将方法中的代码放在 jsp 文件中，就可以接收请求参数了，楼主已经在用了，各位大佬可以定制下。

**更近一步**

数据不回显？

1.  dnslog 外带，这里有个坑，能带的字符串长度有限制，中间不能有特殊符号，可以在命令中对数据进行加密切割，分段传输。
    
    防御方法：对 doslog 进行域名加黑，在攻击者探测阶段就失败。
    
    绕过：自建 dns。
    
2.  在服务器开启 nc 监听，在目标服务器访问 nc 服务器的端口，进行 nc 通信，将信息外带出来。
    
    防御方法：在服务器监听新开启的通信。
    
3.  复写父类方法，使执行有输出（有难度）。
    

参考以上方法和冰蝎的方法可以编写定制化一个 webshell 工具。

**结尾**

想想类似的洞？嘿嘿嘿~

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibtwcMF4fPYGQBA8sQ9EaU0s9oA3Roma7fK7IhibdfSbVecfYTw0VkA7w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibiaRBNn0A5YI88OyFjU8fn2Isf9bat4vQn18NwG6cXxVOSuKiapNm2nibQ/640?wx_fmt=png)