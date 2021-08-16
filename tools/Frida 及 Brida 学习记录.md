> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fCaX-QekyR0CQI0shw0-Qg)

前言
--

之前看了 ESE 大表哥的`Brida插件加解密实战`, 借用 ESE 表哥的资料的基础上对 Frida 和 Brida 插件的一些知识学习和使用做了一个简单的记录。

一、Frida
-------

关于 Frida 的介绍和安装这里就不在重复阐述了，可以参考 ESE 表哥简述的这篇文章 https://www.jianshu.com/p/c349471bdef7

#### 1.1Frida 使用

对于 Frida 的使用方法将以一个编写的 DEMO 来进行展示。在正式使用前需要解释下 python 下的 frida 模块的部分函数名的作用和意义。  
Python 的 frida 模块提供了 frida 使用的所有命令的接口函数，下面截图将解一下几个基本函数。第一个是 frida.get_usb_device() 函数，这个函数是用于获取相应的 usb 口连接的设备。这边有个坑，大家实体机的时候可能会碰到，取决于设备和数据线的性能，大家可以先踩坑，直接说可能映像不深 (滑稽脸)，如下图所示：  
![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzDuG0geNwUAvhNAj193lY3dmkJUgR7sffMsCgY9zPz0n8icN44jWk7sg/640?wx_fmt=png)

赋值的 device 为一个类对象，该类有如下的属性方法，其中最常用的是 attach 方法，通过 attach 方法获取 session 对象来附加到目标进程中：  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzpL0qGFpn0atE4l5jmgeuVKETia2dEfT58ynLcIgL6ia1qLpQFjkR57eA/640?wx_fmt=png)

attach 方法使用返回一个类对象，该类有如下方法，还是挑一个最常用的 create_script 方法进行讲解，该方法用于创建 js 脚本代码并将代码注入到目标进程中：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzMLmLUGlIh5ibQ7yLn91huhia4ZuDria770wa2bfQMN764VibZ4HQp6wElg/640?wx_fmt=png)

调用 create_script 方法如下所示，script 是 frida.core.Script 的一个对象常用 on 和 load 方法，就成功把 js 代码注入到 com.android.chrome 进程中了，如果进程调用 open 函数，就会通过 js 代码中的 send 函数发回 message：  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMz8FeiaddjJhGPwic3sIQMGUicEps0NQahdwuBOM4CiahKlicEAcaFYzgH3Uw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzlfMmFqticibdyshRdGIdVtaibV6tdlULIB0Xj3Kx1lr9C3djzOEoVRKZA/640?wx_fmt=png)

然后就是 frida 的相关 HOOK 脚本的编写，HOOK 脚本的编写套路按下面截图所示就行，需要掌握一点 javascript 语法知识：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMznTnql2yVKLpkYiatq96ciadhewicYTg5ubqiblWbcgdgiaUicwYCMPpu6ySw/640?wx_fmt=png)

以上是 frida 模块使用的简单使用，下面开始介绍一下使用 frida 进行 hook 加解密方法，实验环境准备了一个简单的 apk 安装包 “eseBrida.apk” 和 phpstudy 搭建的服务端，进行通信演示。  
在手机上安装了 eseBrida.apk,PC 端用 phpstudy 搭建了相应的服务端，尝试使用 Burpsuit 抓包看看相应的数据包内容，如下所示：  
在 DEMO 的 app 中输入相应的用户名密码，点击登录使用 BurpSuit 查看发现请求包和响应包的数据都是加密的：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzauyOZm7BL7qcHkAMD6L9zbQ0LrsC1yricv97ol10USKCEZFdLYeQ94Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzMwsYS30zW4sV9bdMfeAHnP2EllrWe4sOtbTkkLUILYD0VlGDiaUlgEA/640?wx_fmt=png)

使用 Jeb 反编译相关 apk 的源码查找到在传送数据包时用于加解密的类方法，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzibPibrDuop4brM5Eu1FYkpHcLfgf3p6Zlx8CPLem9pu3tib6jwCdvthTA/640?wx_fmt=png)

点击进入函数得到其调用的是这个包”com.ese.http.encrypt“下的 AesEncryptionBase64 类中的 encrypt 和 decrypt，故根据上述信息编写相应用于加密解密的 HOOK 脚本，由上述 JEB 中加解密函数分析可以知道，app 在调用加解密函数时会传入两个参数其中第一个即为 key。利用此可以 HOOK app 中的加密方法直接打印出相应的加解密 key，即使对代码做了混淆等也可以直接输出。调用加密方法的 HOOK 脚本如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzHTnW16xsyFuVfnluXF7YfzRumrwauGMhldIjgGImBCyjhItUGlQySg/640?wx_fmt=png)

然后启动在手机中启动 frida-server, 利用 python 的 frida 模块编写相应的进行 hook 交互的脚本，python 脚本如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzjqlJ5HpmSFVaTUaq6pLhJiaZl0vHIpTRBkWy110KO2jU4dibxkttxuXQ/640?wx_fmt=png)

运行 python 的交互式 HOOK 脚本，在 app 中的登录框中输入相应的登录名和密码进行登录尝试，app 客户端会调用上述的加密方法加密对输入的用户名和密码进行加密，HOOK 方法直接 HOOK 到相应的加密方法，输出传入的 key 和加密参数以及加密结果，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzFtZkP1eKkicYGibqGjbGX0lyQ1iaRvXRfHnfXia58QI8Fhw1lvloDCiaOeA/640?wx_fmt=png)

运行的 python 交互式 HOOK 加密函数的脚本输出结果如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzWJia9x4koY1tO8WZxHtzVxptwiaTckiciaKWCTnwmUylCB9ia3df7ibVbXvA/640?wx_fmt=png)

同理 HOOK 解密函数，其 HOOK 脚本内容如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzzvX19nCcaKX1eFM8icwfFiaJtxRaVaRFDdlczL754NEOH8EbRtxRTibuQ/640?wx_fmt=png)

Python 的交互式输出内容如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzGlysQUMiaJVRczbjgDt14bHN9rgVlwQOY8iamqNuUWwQsTpLicqdprDWQ/640?wx_fmt=png)

以上就是，Frida 内容的简单介绍，下一章节就是 Brida 插件的使用介绍。

二、Brida
-------

#### 2.1 Brida 插件使用准备

Brida 上文也说到了是一个 BurpSuit 插件用于连接 Frida 和 BrupSuit，对 app 进行 HOOK 其相关函数，重写该函数供 BurpSuit 进行例如解包拼包操作。该插件虽由 java 编写，但其核心功能即 HOOK 和 app 数据通信是用 python 来实现的，下载源码从源码中加载的 python 脚本文件可知，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMztPeyyqNrUDXu7z28uE2gtXsKZDoMcRBSzXSDJwqCibANtypwtrXHYZQ/640?wx_fmt=png)

查看 java 文件调用相应的 python 脚本内容，可以发现其主要依赖于 pyro4 和 firda 两个模块，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzjHMwvWCxWwI2LS9TqORzlRlJgw2tsSPv1fE628iceWibPgKlAjiaQOt7w/640?wx_fmt=png)

Frida 模块上面已经说过即用来调用 frida 进行 HOOK，Pyro4 是 python 的 RPC 框架即远程过程调用，用 Burpsuit 的图形化按钮来调用 HOOK 脚本函数。

Burpsuit 中导入安装 Brida 插件，由上分析可知插件要能正常使用必须依赖于 python 环境和 python 的 frida、pyro4 模块，故首先要安装好上述环境。导入插件，其展示界面如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMz26NnG5Boe3Gj7M4S8siamWxMM5nkg1NVeibNLbgojYUxpls6y0fQIU2g/640?wx_fmt=png)

Brida 的界面如上所示，主要分为三个部分：console 输出框用于输出插件启动，调用 app，以及运行报错等信息：控制按钮用于用户启动 \ 终止服务，启动或结束 app，载入 HOOK js 脚本等作用；功能选项中最重要的就是 configurations 和 Excute method 两个，configurations 为插件正常运行所需的环境参数配置，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzN5iahnvpprrf7frugoMdAUdyptibJ6x73EkuSnoc4qQZkoPHk5lMzNTg/640?wx_fmt=png)

其中选择连接方式就是 PC 机和手机使用 USB 线连的还是用远程端口转发的方式连的。查看插件源码也可以看到，当选择 remote 选项时即调用为 python 中 firda 模块的 get_remote_device() 方法，local 选项即调用了 get_usb_device() 的方法，脚本源码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzW8QvzdOjHtAfjONaMo0fj5nFS7qP0Dib36RicNt3kNMKwBX0dAAsSOXg/640?wx_fmt=png)

然后时 Excute method 功能选项，该功能是提供了一个能够执行 HOOK 脚本中定义函数的接口界面，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzamicuI3eyEDUpn1qa8mrQOU9RU01icYpVWHlMVF7Bkn6byAYafuXJVXA/640?wx_fmt=png)

执行的结果输出至 console 输出台中。

#### 2.2RPC HOOK 脚本

由于此处 Brida 插件通过 RPC 的方式来调用 Frida HOOK 出的方法，故此处需要编写相应的 rpc 调用用于 hook js 脚本，其基本格式在 Brida 源码中有相应的模板，在 RPC 函数中写相应的 HOOK 函数内容就行，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzNANSibziahaTZwOV6Pl8V7bMahVzNfXImeDeXpzc5pSGNCVcqcAtvEyQ/640?wx_fmt=png)

其中默认有四个定义函数分别对应 Brida 插件的四个快捷键，contextcustom1、contextcustom2 用加密解密请求包，contextcustom3、contextcustom4 用于加解密相应包：  
对应于 Brida 的如下快捷键，如下所示：  
在 BurpSuit 的 Repeater 中的请求包快捷如下图所示，Brida Custom1 即对应 contextcustom1，Brida Custom2 即对应 contextcustom2：  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMz6rtSSVYDiblPY3bE92b74k7Ap51k1UhkoP0Zojx3c7RaRk8SqgYu9kw/640?wx_fmt=png)

相应包也同理：  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzmZ5lKQt2VhnOiaWezXOpRHj9FVoApTQ3zwN8L4LG6gH1jQuBnDFE3Ug/640?wx_fmt=png)

除了上述默认的函数外，也可以自己定义函数只是没有相关快捷键，可以在 Brida 插件中的 Excute Method 功能中调用执行，结果在 console 台上输出：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzHcCNlrokSGHgWOp8G42RzfWdeVrAkicZu1iay6gIecnl4ianXuDvX1t5A/640?wx_fmt=png)

在使用 Brida 插件使用 RPC 调用 HOOK 函数输出到 Burpsuit 的界面中有一个需要注意的是，BurpSuit 输入输出接口中的数据是以 16 进制编码后的字符串来传递的，因此 RPC 脚本中需要定义字符转 16 进制字符的函数和相应反转函数，这个在 Brida 给的脚本模板中有编写了相应的函数（转换函数只作用于默认快捷键函数）  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzWOUiblKr0fSy21J2ciaibz94V1QnUAzbUVMXqz2ZxF4nicyBjK2lpgvKPA/640?wx_fmt=png)

如果为图片等字节流，可采用 byte 转成 16 进制函数，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMz53KV5yTGXJ7cHk96dibYPicaaHV7w7YicJibpttcxu4zcic1CJ9op7GdWLg/640?wx_fmt=png)

#### Brida 使用

根据上述介绍，在 Brida 插件的 configurations 中输入相应的配置信息，包括系统 python 执行路径、rpc 启动地址和端口、frida 的 HOOK 脚本路径、app 包名、获取方式，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzNQuXg0QG16jFTicdA2YJ1Bn72voWiaXic3Jib0M5udEIuXFYibFprjJbZxw/640?wx_fmt=png)

编写的相关 HOOK 脚本函数 contextcustom1、contextcustom2，contextcustom3、contextcustom4 同理如下所示：  
加密函数：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzxicHvjpic6plv3dknsIHLYjeGvwIDQBeKO5xqvfPADRicKjJibFP2VVB5Q/640?wx_fmt=png)

解密函数：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzT5GnrZmibUIVdMicXhhW6vmoxu1DWXwvS7ToHmguHDVPeSH98XfiaicxiaQ/640?wx_fmt=png)

点击 start server 启动服务，在点击 spawn application 启动 app，在点击 reload JS 将脚本导入 Frida 中进行 HOOK，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzTMjlJIGDmmk8vYec17S6NF7axUxAnUy1PUl4sR6XROicgHIgiaQmCwWg/640?wx_fmt=png)

Brida 点击 Spawn 自动打开了相关 app，在 app 中输入用户名、密码 BrupSuit 抓到相应数据包，发送至 repeate 如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzChayds9YP8HWbGkerXnaFl30ET1TaUiajRy5PcAv3kW3OxLymbyN4AA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzv00VMWdHkSWxldnDB9aoXibHISMyz30TdykE4J7426xEoeW0YU46YPA/640?wx_fmt=png)

选中要解密的内容，调用 Brida custom2 快捷键成功解密请求包内容，如下所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzB4p4AB1ZebXdqZXDf5llRJ24fBkjvCkKTtJic38ia2OGBE6cajKkS1Yw/640?wx_fmt=png)

选中要加密的内容，使用 Brida Custom1 快捷键即调用加密函数将相应内容加密，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMzzvpEBoTzvYTe4PnhsiaibzHx9wnVK9Ah7UX5OhEAvAgvHEhHQg5gCDrA/640?wx_fmt=png)

加解密返回包也是同样的原理，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic28j3K2KpgBhZUFic6NMtJMznFZfzVIs8GWsWRUd6V33bIKNvIuMwjicGGFoiatANeBcVDK6vNdvibYWg/640?wx_fmt=png)

**参考链接**

*   https://github.com/federicodotta/Brida/releases
    
*   https://blog.csdn.net/xiaolewennofollow/article/details/52155457
    
*   https://www.jianshu.com/p/c349471bdef7
    
*   https://frida.re/docs/
    
*   https://bbs.pediy.com/thread-248977.htm
    
*   https://www.freebuf.com/sectool/143360.html
    

### 作者：spider，文章来源：先知社区

**关注公众号: HACK 之道**  

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3qL1R1NCVbY1ElanNngBlMTUKUibAUoQNQuufs7QibuMXoBHX5ibneNiasMzdthUAficktvRzexoRTXuw/640?wx_fmt=jpeg)