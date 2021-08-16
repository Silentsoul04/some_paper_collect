> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/c3AouOZRqISMhaPko9SU7g)

定义
--

**RMI: 远程方法调用 (Remote Method Invocation)**，它支持存储于不同地址空间的程序级对象之间彼此进行通信，实现远程对象之间的无缝远程调用。

**Java RMI**: 用于不同虚拟机之间的通信，这些虚拟机可以在不同的主机上、也可以在同一个主机上；一个虚拟机中的对象调用另一个虚拟上中的对象的方法，只不过是允许被远程调用的对象要通过一些标志加以标识

java 本身提供了一种 RPC 框架——RMI（即 Remote Method Invoke 远程方法调用），在编写一个接口需要作为远程调用时，都需要继承了 Remote，Remote 接口用于标识其方法可以从非本地虚拟机上调用的接口，只有在 “远程接口”（扩展 java.rmi.Remote 的接口）中指定的这些方法才可远程使用，

底层协议
----

### RPC

RPC（Remote Procedure Call）—远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC 协议假定某些传输协议的存在，如 TCP 或 UDP，为通信程序之间携带信息数据。在 OSI 网络通信模型中，RPC 跨越了传输层和应用层。RPC 使得开发包括网络分布式多程序在内的应用程序更加容易。RPC 采用客户机 / 服务器模式。请求程序就是一个客户机，而服务提供程序就是一个服务器。

小总结：

在最原始数据通讯中其实还是归根到 TCP/UDP 协议，但是自使用了 RPC 后可以不需要了解底层网络协议，但是底层还是通过 TCP/UDP 去进行网络调用的。其实 RPC 也只是远程方法调用的统称，重点在于方法调用中。而 RMI 实现就是 Java 版的一个 RPC 实现。

### RPC 和 RMI 的区别

RPC（Remote Procedure Call Protocol）远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC 不依赖于具体的网络传输协议，tcp、udp 等都可以。

RPC 是跨语言的通信标准。

RMI 可以被看作 SUN 对 RPC 的 Java 版本的实现，当然也还有其他的 RPC

微软的 DCOM 就是建立在 ORPC 协议之上实现的 RPC。

RMI 集合了 **Java 序列化**和 **Java 远程方法协议 (Java Remote Method Protocol)**。这里的 Java 远程方法协议则是 JRMP。

RMI 远程调用步骤
----------

在客户端和服务器各有一个代理，客户端的代理叫 Stub（存根），服务端的代理叫 Skeleton（骨架），合在一起形成了 RMI 构架协议，负责网络通信相关的功能。**代理都是由服务端产生的，客户端的代理是在服务端产生后动态加载过去的。**

stub 担当远程对象的客户本地代表或代理人角色，负责把要调用的远程对象方法的方法名及其参数编组打包, 并将该包转发给远程对象所在的服务器。

![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7gNkNldOvonMsiaRBd27CdIHd0knpFiaqq67eYLLSYeaPWiau20E8iagJsAcvlTOjqYCdz01icaaWsfeeA/640?wx_fmt=png)

**总结大体的内容如下：**

```
客户端(Client)：服务调用方。
    
客户端存根(Client Stub)：存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端。
    
服务端存根(Server Stub)：接收客户端发送过来的请求消息并进行解包，然后再调用本地服务进行处理。

服务端(Server)：服务的真正提供者。
```

RMIRegistry 可以认为是一个服务，它提供了一个 hashmap，里面是 `public_name`, `Stub_object` 名值对。比如有一个远程服务对象叫做 `Scientific_Calculator`，然后想把这个服务对外公布为 calc，这样会在 server 上创建一个 stub 对象，把他注册到 RMIRegistry ，这样 client 就可以从 RMIRegistry 中得到这个 stub 对象了，可以使用一个工具类`java.rmi.Naming`来方便的操作注册和操作。

Java RMI 简单示例一
--------------

![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7gNkNldOvonMsiaRBd27CdIHbYIAtZkWP5jTTlX0MQexoVf6GvkiaBm3w4QsOcYSYMXGPs9O7JrOs7Q/640?wx_fmt=png)

以下用一个最简单的 Hello 的示例来介绍 RMI 的应用。

IHello.java

```
package com.evalshell.rmi;import java.rmi.Remote;public interface IHello extends Remote {    public String sayHello(String name) throws java.rmi.RemoteException;}
```

HelloImpl.java

```
package com.evalshell.rmi;import java.net.MalformedURLException;import java.rmi.RemoteException;import java.rmi.registry.LocateRegistry;import java.rmi.server.UnicastRemoteObject;public class HelloImpl extends UnicastRemoteObject implements IHello {    private static final long serialVersionUID = 7077319331699640332L;    // 这个实现必须有一个显式的构造函数，并且要抛出一个RemoteException异常      public HelloImpl() throws RemoteException {        super();    }    @Override    public String sayHello(String name) throws RemoteException {        return "hello" + name ;    }    public static void main(String[] args) {        try{            IHello hello = new HelloImpl();            LocateRegistry.createRegistry(11001);          //如果不想在控制台上开启RMI注册程序RMIRegistry的话，可在RMI服务类程序中添加LocateRegistry.createRegistry(11001);            java.rmi.Naming.rebind("rmi://127.0.0.1:11001/hello", hello);            System.out.println("ready");        }catch (RemoteException | MalformedURLException e) {            e.printStackTrace();        }    }}
```

Hello_RMI_Client.java

```
package com.evalshell.rmi;import com.evalshell.rmi.IHello;import java.rmi.Naming;public class Hello_RMI_Client {    public static void main(String[] args) {        try {            com.evalshell.rmi.IHello hello = (IHello) Naming.lookup("rmi://127.0.0.1:11001/hello");            System.out.println(hello.sayHello("fengxuan"));        }catch (Exception e){            e.printStackTrace();        }    }}
```

启动`HelloImpl`

![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7gNkNldOvonMsiaRBd27CdIHCSVEkaScWlyodsIqhwp28ub9Mgw4RjxDkR5pT9Wic4c0NtuOmkzYiamg/640?wx_fmt=png)

可以看到开启了 RMI 的注册服务。

运行`Hello_RMI_Client`

输出了 hello fengxuan

![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7gNkNldOvonMsiaRBd27CdIHqweFviccdLLjZWtxsDo9IKr7IJMCJk8muQweaJfpo0kth4URaQg8ckw/640?wx_fmt=png)

⼀个 RMI Server 分为三部分:

```
1. 一个继承了了`java.rmi.Remote`的接口，其中定义我们要远程调用的函数，⽐如这里的sayHello方法 
2. 一个实现了了此接口的接口实现类
3. 一个主类，⽤用来创建Registry，并将⾯面的类实例例化后绑定到⼀一个地址。这就是我们所谓的Server了。
```

查看网络请求得知：

通过 wireshark，我们可以发现，整个过程进行了了两次 TCP 握手，建立了链接。

![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7gNkNldOvonMsiaRBd27CdIHQAe5dlzy2V14gSuNvWxuKoMoElZdgOzDb8NfemR1VhJ8z3RkicicLntw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7gNkNldOvonMsiaRBd27CdIHiaibNPYHYnhad2w1Paibq0q3WxVvxmp2UyvvnrGicqbeuibSr1D13ial9oJw/640?wx_fmt=png)

总结
--

总体来说就是所以的远程对象都会在 server 的随机的端口上映射，RMIRegistry 也会进行映射，但是 RMIRegistry 映射的端口是 1099（默认是，可以修改）。远程对象进行映射的端口，client 是不知道的。但是客户端代理对象会知道。

简单的分析了一下 RMI 底层的架构，但是这些其实都仅仅是基于概念和理论层面的，具体的代码实现其实还没去看。在其中也是看得晕头转向的，部分也摘取了其他师傅们的文章内容，感觉已经总结很到位了，疯狂安利。摘取的内容下也贴出来 摘取内容的出处，感谢各位师傅们的详细讲解。

参考
--

https://github.com/phith0n