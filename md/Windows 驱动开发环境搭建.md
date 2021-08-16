> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wThom_U5RaiZPFxoN7tVww)

前言

    本节主要讲解一下驱动开发环境搭建和一些常见的问题如何解决。     

1. 环境搭建

    必要工具，windbg，VS13 或更高版本，WDK 驱动开发，KmdManager 和 DbgView

    KmdManager 和 DbgView 的安装包如下

 **链接：https://pan.baidu.com/s/1F0sHm6xVNCQ2S4fC_C16hQ**

 **提取码：9fd2**

    当 VS 编辑器和 WDK 驱动开发安装之后，如下图，成功出现 Windows Driver 则表示成功。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyxmqNia2trIMgeKb5ehYKGzYUvLl1vgI9tvxR5aKxpIVLnLtlJN0sChJPCNuFMkhQBYYKoBqIBGR1Q/640?wx_fmt=png)

2. 编写一个简单的驱动

    首先创建一个驱动项目

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyxmqNia2trIMgeKb5ehYKGzY1fiaRVGFUEa21ibqz5pichZbibshxxBZLEiaKStHDFvVA0MUxtbP6P5cLuQ/640?wx_fmt=png)

    接下来编写驱动，如下图，是一个简单的驱动，在 C 语言或 C++ 中，我们的入口点函数都是 main，而到了驱动中，**DriverEntry 函数则是入口点，driver->DriverUnload = Uploades; 这行代码指向的是我们的卸载函数。**

 **因为我们的驱动要，安装 运行 停止 卸载，当运行的时候，执行的是主函数，停止的时候，执行的是 Uploads 这个函数。**

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyxmqNia2trIMgeKb5ehYKGzYdGnKJld6jzcBRFzp32Hu2SQsATplg4CUVcNB0O1lvlctuiaiaZNRB80g/640?wx_fmt=png)

    生成编译之后放到虚拟机进行调试运行。KmdManager 和 DbgView 用管理员方式运行，当点 Run 的时候，会执行 **DriverEntry** 主函数，在点击 Stop 停止的时候，就会触发我们的回调函数 Uploades。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyxmqNia2trIMgeKb5ehYKGzYLIzkfuq08WibNAib3XudpWxbHn3GlF9UH40EsnCtEniay5BF1Jyaia9sVQ/640?wx_fmt=png)

3. 常见问题

    KmdManager 在运行的时候，要以管理员运行。

    DbgView 如何不显示打印信息，可以根据如下图进行更改

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyxmqNia2trIMgeKb5ehYKGzY9IF3YwDduO3uRtZroJbRiaWibp77tc1enlBR1ribf9lCygp9veUhClib8g/640?wx_fmt=png)

    同时 DbgView 需要注册注册表，将以下代码保存为 reg 格式文件直接运行即可。

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Debug Print Filter]
"DEFAULT"=dword:0000000f
```

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyxmqNia2trIMgeKb5ehYKGzYVpd5DMU3qO9iaEuCf1HjLkGp6VWDYSLRZmSOB8QKG5PKwk9T320Tx3Q/640?wx_fmt=png)

 **微信搜索关注 "安全族" 长期更新安全资料，扫一扫即可关注安全族！**

![](https://mmbiz.qpic.cn/mmbiz_jpg/8miblt1VEWywCsRiaweFhRW8aDdjtoCoSU2eQAJ6KxKAoP0PSHvjGJvTZcRRXTAeSd9Qyib0ynLnBUwdiahhhOaSDQ/640?wx_fmt=jpeg)