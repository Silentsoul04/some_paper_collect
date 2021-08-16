> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_Xa_U_ohnjafXPFwm3B-0Q)

Android 虚拟机和类加载机制
=================

Android 虚拟机
-----------

Android 应用程序运行在 Art/dalvik 虚拟机，并且每个应用程序对应有一个单独的虚拟机实例，Dalvik 虚拟机也算是 Java 虚拟机，不过执行的文件不是 class 文件而是 dex 文件，

Dalvik 和 Art 虚拟机指令集基于寄存器的，Java 虚拟机指令集是基于栈的。

JavaVM 和 DVM/ArtVM 区别

1.  指令条数，前者需要操作数栈和局部变量表中加载和操作数据，后者只在寄存器上操作数据
    
2.  可移植性 ， 前者可移植性强，后者在不同平台有不同实现
    

Dalvik 和 ART
------------

![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6w8fdvabycfAMoF7niaFK3vW17xn8TtaiatfLaLibMjNtick0O42XTCznz5ib8FxFAPFH8XXn5UoPpJJXdg/640?wx_fmt=png)

Dalvik 虚拟机执行的是 dex 字节码，Android2.2 版本后，引入了 JIT（即时编译技术）编辑器，Dalvik 在每次运行 app 时，JIT 会对频繁执行的 dex 代码进行编译和优化，缓存到内存中，在下次调用相同代码块时直接调用缓存中的机器码，App 关闭后，缓存也将被清除。

Dalvik 虚拟机里的垃圾回收分别在 mark 跟 sweep 时终端了线程两次。

ART 虚拟机是在 Android4.4 中引入的开发者选项，在 Android5.0 之后默认使用 ART，ART 向下兼容 Dalvik 虚拟机，同样运行 dex 文件，Dalvik 开发的应用可以在 ART 环境中运行。

ART 在应用程序安装时，将应用 dex 字节码使用 dex2oat 预编译（AOT）成本地机器码，消耗更多的存储空间，但不超过应用代码包的 20%，应用的安装时间也被延长

在 Android N 后对前面两种方式做了新的改进，在安装时不进行预编译，在运行时，对经常执行的方法进行 JIT，将 JIT 编译后的方法记录到 profile 配置文件，在设备闲置或充电时，编译守护进程运行，根据 profile 配置文件对常用代码进行 AOT 编译。

#### CLASSLOADER 介绍![](https://mmbiz.qpic.cn/mmbiz_png/v6ap3LYR6w8fdvabycfAMoF7niaFK3vW11AnjpiabnTVmT2LfetD5BUeVQR0SIC3HL9mibiaUWRqSynUoNicGiaYp1qg/640?wx_fmt=png)

ClassLoad 具体实现类：

*   BootClassLoader : 用于加载 Android Framework 层 class 文件
    

*   PathClassLoader: Android 应用程序类加载期，可以加载执行的 dex，以及 jar、zip、apk 中的 classes.dex
    

*   DexClassLoader: 用于加载执行的 dex, 以及 jar、zip、apk 中的 classes.dex
    

PathClassLoader 与 DexClassLoader 的共同父类是 BaseDexClassLoader 。

```
public class DexClassLoader extends BaseDexClassLoader {

  public DexClassLoader(String dexPath, String optimizedDirectory,
          String libraryPath, ClassLoader parent) {
      super(dexPath, new File(optimizedDirectory), libraryPath, parent);
  }
}

public class PathClassLoader extends BaseDexClassLoader {

  public PathClassLoader(String dexPath, ClassLoader parent) {
      super(dexPath, null, null, parent);
  }


  public PathClassLoader(String dexPath, String libraryPath,
          ClassLoader parent) {
      super(dexPath, null, libraryPath, parent);
  }
}
```

两者都可以加载指定的 dex，以及 jar、zip、apk 中的 classes.dex

前面两个类都没有定义 classloader，看看他们的父类 BaseDexClassLoader 中的 classloader

```
public Class<?> loadClass(String name) throws ClassNotFoundException {
  return loadClass(name, false);
}
   
protected Class<?> loadClass(String name, boolean resolve)
      throws ClassNotFoundException
{
      // First, check if the class has already been loaded
      Class<?> c = findLoadedClass(name);
      if (c == null) {
          try {
              if (parent != null) {
                  c = parent.loadClass(name, false);
              } else {
                  c = findBootstrapClassOrNull(name);
              }
          } catch (ClassNotFoundException e) {
              // ClassNotFoundException thrown if class not found
              // from the non-null parent class loader
          }

          if (c == null) {
              // If still not found, then invoke findClass in order
              // to find the class.
              c = findClass(name);
          }
      }
      return c;
}


protected Class<?> findClass(String name) throws ClassNotFoundException {
      throw new ClassNotFoundException(name);
  }
```

检查这个类的 class 是否加载过，如果加载过就直接返回 class 文件，在判断父类 classloader 是否加载过，如果都没有加载过，在自己去加载。这就是双亲委派模型。

这里的 parent 是通过构造方法传进来的 classloader，系统在创建 pathclassloader 的时候传进来的是 BootClassLoader，而不是他们的父类 BaseClassLoader

> 双亲委派机制的好处：
> 
> 1.  避免重复加载
>     
> 2.  安全性考虑，防止核心 API 库被随意篡改
>