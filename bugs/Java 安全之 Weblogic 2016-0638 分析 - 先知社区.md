> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8701)

0x00 前言
-------

续上篇文的初探 weblogic 的 T3 协议漏洞，再谈 CVE-2016-0638， CVE-2016-0638 是基于 CVE-2015-4852 漏洞的一个绕过。

[Java 安全之初探 weblogic T3 协议漏洞](https://www.cnblogs.com/nice0e3/p/14201884.html)

0x01 环境搭建
---------

### 补丁环境搭建

这里采用上次的 weblogic 环境，但是在这里还需要打一个补丁包，来修复 CVE-2015-4852 漏洞后，对该漏洞进行一个绕过。

CVE-2015-4852 的修复补丁为`p21984589_1036_Generic`, 由于在互联网上并没有找到该补丁包，只能通过官网下载，官网下载需要购买对应的服务，所以在这里找了`p20780171_1036_Generic`和`p22248372_1036012_Generic`这两个补丁包，`p21984589_1036_Generic`是前面这两个补丁包的集成。

因为前面搭建是 docker 的环境，需要将这两个补丁包上传到 docker 镜像里面去，然后进行安装。

命令整理：

```
docker cp ../p20780171_1036_Generic  weblogic1036jdk7u21:/p20780171_1036_Generic

docker cp ../p22248372_1036012_Generic  weblogic1036jdk7u21:/p22248372_1036012_Generic

docker exec -it weblogic1036jdk7u21 /bin/bash

cd /u01/app/oracle/middleware/utils/bsu

mkdir cache_dir

vi bsu.sh   编辑MEM_ARGS参数为1024

cp /p20780171_1036_Generic/* cache_dir/

./bsu.sh -install -patch_download_dir=/u01/app/oracle/middleware/utils/bsu/cache_dir/ -patchlist=EJUW -prod_dir=/u01/app/oracle/middleware/wlserver/

cp /p22248372_1036012_Generic/* cache_dir/

./bsu.sh -install -patch_download_dir=/u01/app/oracle/middleware/utils/bsu/cache_dir/ -patchlist=ZLNA  -prod_dir=/u01/app/oracle/middleware/wlserver/ –verbose


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145240-735e6e52-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145240-735e6e52-44eb-1.png)

重启 weblogic 服务。

```
/u01/app/oracle/Domains/ExampleSilentWTDomain/bin/startWebLogic.sh


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145303-80b83588-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145303-80b83588-44eb-1.png)

这里看到 weblogic 2015-4852 的 payload 打过去，并没有像以往一样，创建一个文件。那么就说明补丁已经打上了，已经能够修复该漏洞了。这里是切换了 JDK7u21 和 cc1 的利用链依旧没打成功。

### 远程调试

接下来还是需要将里面的依赖包给拷一下。

```
mkdir wlserver1036

mkdir coherence_3.7

docker cp weblogic1036jdk7u21:/u01/app/oracle/middleware/modules ./wlserver1036

docker cp weblogic1036jdk7u21:/u01/app/oracle/middleware/wlserver/server/lib ./wlserver1036

docker cp weblogic1036jdk7u21:/u01/app/oracle/middleware/coherence_3.7/lib ./coherence_3.7/lib


```

下面来对该补丁进行一个绕过。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145318-8a00edba-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145318-8a00edba-44eb-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145327-8f540978-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145327-8f540978-44eb-1.png)

0x02 补丁分析
---------

补丁作用位置：

```
weblogic.rjvm.InboundMsgAbbrev.class :: ServerChannelInputStream
weblogic.rjvm.MsgAbbrevInputStream.class
weblogic.iiop.Utils.class


```

在分析漏洞前，先来看到一下，上一个漏洞点的补丁是怎么进行修复的。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145340-96b57472-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145340-96b57472-44eb-1.png)

在这其实看到该`resolveClass`方法的位置，前面加多一个判断。

前面判断 className 是否为空，ClassName 的长度是否为零，但是重点是`ClassFilter.isBlackListed`方法。

这里先打一个 CVE-2015-4852 exp 过来, 在该位置打个断点，跟踪进该方法，查看怎么进行防护。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145352-9e51d52c-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145352-9e51d52c-44eb-1.png)

跟进进来后，先别急着看后面的，因为下面还有一个静态代码块，静态代码块中代码优先执行，需要先来查看静态代码块内容。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145403-a4d42d96-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145403-a4d42d96-44eb-1.png)

这里前面有两个判断，判断中都调用了两个方法，来看看这两个方法的实现。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145415-abaf1fb8-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145415-abaf1fb8-44eb-1.png)

一个是判断是否为`weblogic.rmi.disableblacklist`，一个是判断是否为`weblogic.rmi.disabledefaultblacklist`, 写法有点奇怪，可能是因为是 class 文件的缘故。

这两个判断为 true 的话，就会执行来到下一步调用`updateBlackList`将后面的一系列黑名单的类传入到里面去。

`updateBlackList`该方法从名字得知，就是一个黑名单列表添加的一个方法，将黑名单内容添加到一个`HashSet`里面去。查看具体实现。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145426-b264e266-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145426-b264e266-44eb-1.png)

StringTokenizer 构造方法：为指定的字符串构造一个字符串 tokenizer。

hasMoreTokens 方法：返回与 `hasMoreTokens`方法相同的值。

nextToken 方法：返回此字符串 tokenizer 字符串中的下一个令牌。

总体的来理解就是构造一个字符串，然后遍历里面的值，然后调用`processToken`方法将该值传递进去。

再来看到`processToken`方法。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145435-b7f2b4ba-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145435-b7f2b4ba-44eb-1.png)

里面判断如果开头是`+`号, 则截取第一位后面的值添加到黑名单的这个 HashSet 里面去。如果是`-`号则移除，如果开头不是前面的`+` `-`号则直接添加到黑名单里面去。

到这里静态代码块就已经分析完成了，总的来说其实就是将一些危险的类，添加到了黑名单里的一个步骤。

黑名单列表为：

```
+org.apache.commons.collections.functors,
+com.sun.org.apache.xalan.internal.xsltc.trax,
+javassist,+org.codehaus.groovy.runtime.ConvertedClosure,
+org.codehaus.groovy.runtime.ConversionHandler,
+org.codehaus.groovy.runtime.MethodClosure


```

返回刚刚的`ClassFilter.isBlackListed`方法进行跟踪

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145448-bf973632-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145448-bf973632-44eb-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145457-c4e0457a-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145457-c4e0457a-44eb-1.png)

最后这里调用了`contains`方法判断 这个 pkgName 存不存在黑名单中，存在的话这里返回 true。

返回到`resolveClass`方法可以看到这里为 true，就会直接抛异常。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145508-cb9bb94e-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145508-cb9bb94e-44eb-1.png)

如果不存在于黑名单中，会来到 else 这个分支的代码块中调用父类的`resolveClass`方法。

而这一个点，只是过滤的一个点，下面来看看过滤的点都有哪些。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145520-d293c7b4-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145520-d293c7b4-44eb-1.png)

再来看下一个点`MsgAbbrevInputStream`的位置

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145530-d8ac583c-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145530-d8ac583c-44eb-1.png)

这里也是调用`ClassFilter.isBlackListed`方法进行过滤，和前面的是一样的。以此类推。

0x03 工具分析
---------

在 CVE-2016-0638 里面用到了 weblogic_cmd 工具，[github 地址](https://github.com/5up3rc/weblogic_cmd)。

下面来看看该工具的实现，再谈漏洞的绕过方式。

下载该源码后，导入 IDEA 中，配置命令参数。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145601-eadf62a6-44eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145601-eadf62a6-44eb-1.png)

这里如果报错找不到 sun.tools.asm 包的话，需要将 Tools.jar 包手动添加一下。在这我是使用 jdk1.6 进行执行的，使用 1.8 版本会找不到`sun.org.mozilla.javascript.internal.DefiningClassLoader`类

在 Main 的类中打个断点进行执行。

前面都是代码都是进行一个配置，这里断点选择落在该方法中。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145701-0e87bf00-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145701-0e87bf00-44ec-1.png)

选择跟踪

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145715-17470902-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145715-17470902-44ec-1.png)

继续跟踪`WebLogicOperation.blindExecute`方法。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145733-21f79272-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145733-21f79272-44ec-1.png)

前面判断了服务器类型，重点在`SerialDataGenerator.serialBlindDatas`方法中，payload 由该方法进行生成。跟进查看一下该方法如何生成 payload。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145744-28a4db3e-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145744-28a4db3e-44ec-1.png)

在这先选择跟踪`blindExecutePayloadTransformerChain`方法。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145756-2fac9b56-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145756-2fac9b56-44ec-1.png)

在这里又看到了熟悉的面孔，CC 链的部分代码。

回到刚刚的地方，跟踪`serialData`方法

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145812-38f2b7e0-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145812-38f2b7e0-44ec-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145824-40685ebc-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145824-40685ebc-44ec-1.png)

在这里就看到了 CC 链后面的一段代码，这组合成了一条 CC1 利用链。但是在后面调用了`BypassPayloadSelector.selectBypass`方法来处理在原生的利用链中本该直接进行序列化的对象。

跟进该方法进行查看。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145835-46c7a48e-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145835-46c7a48e-44ec-1.png)

这里面还会去调用`Serializables.serialize`，依旧先跟踪最里层的方法。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145847-4dd13498-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145847-4dd13498-44ec-1.png)

这传入一个 obj 对象和 out 对象，进行了序列化操作。然后将序列化后的数据写到 out 对象中。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145857-53f84794-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145857-53f84794-44ec-1.png)

执行完成后，返回上一个点，刚才分析得知返回的是序列化后的数据。所以在处调用`streamMessageImpl`方法传递的参数也是序列化的数据。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145910-5bcfe896-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145910-5bcfe896-44ec-1.png)

跟踪查看。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145922-6290aff8-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145922-6290aff8-44ec-1.png)

内部是 new 了一个`weblogic.jms.common.StreamMessageImpl`的实例，然后调用`setDataBuffer`方法将序列化后的对象和序列化后的长度传递进去。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145933-690d55a2-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145933-690d55a2-44ec-1.png)

执行完这步后，回到这个地方

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223145956-7708f936-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223145956-7708f936-44ec-1.png)

后面的这个方法是进行序列化操作的，这里又对 `streamMessageImpl`的实例对象进行了一次序列化。该方法在前面查看过了，这里就不跟进去看了。

而最后来到了这里。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223150008-7e18bf36-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223150008-7e18bf36-44ec-1.png)

而后面这个方法就是构造特定的数据包，使用 T3 协议发送 payload。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223150024-8773898a-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223150024-8773898a-44ec-1.png)

0x04 漏洞分析
---------

那么如果需要绕过的话，我们需要找一个类，他的类在内部的`readObject`方法创建了自己的`InputStream`的对象, 但是又不能为黑名单里面过滤掉的`ServerChannelInputStream`和`MsgAbbrevInputStream`里面的`readObject`方法。然后调用该`readObject` 方法进行反序列化，这时候就可以达成一个绕过的效果。

在师傅们的挖掘中寻找到了`weblogic.jms.common.StreamMessageImpl#readExternal()`，`StreamMessageImpl`类中的`readExternal`方法可以接收序列化数据作为参数，而当`StreamMessageImpl`类的`readExternal`执行时，会反序列化传入的参数并调用该参数反序列化后对应类的这个`readObject`方法。

绕过原理如下：

```
将反序列化的对象封装进了 StreamMessageImpl，然后再对 StreamMessageImpl 进行序列化，生成 payload 字节码。反序列化时 StreamMessageImpl 不在 WebLogic 黑名单里，可正常反序列化，在反序列化时 StreamMessageImpl 对象调用 readObject 时对 StreamMessageImpl 封装的序列化对象再次反序列化，这样就逃过了黑名单的检查。


```

在此先再来思考一个问题，`weblogic.jms.common.StreamMessageImpl#readExternal()`该方法是怎么被调用的呢？在前面分析原生`readObject`方法的时候发现，其实`readObject`方法的底层还会去调用很多其他方法。

在 Weblogic 从流量中的序列化类字节段通过 readClassDesc-readNonProxyDesc-resolveClass 获取到普通类序列化数据的类对象后，程序依次尝试调用类对象中的 readObject、readResolve、readExternal 等方法。而在这里`readExternal`就会被调用。

那么下面来调试分析一下该漏洞。

还是先在`weblogic.rjvm.InboundMsgAbbrev#ServerChannelInputStream.resolveClass`地方打个断点，然后使用 weblogic_cmd 工具打一个 payload 过去，先来查看一下传输过来的数据。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223150050-973d6426-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223150050-973d6426-44ec-1.png)

这里可以看到获取到的 ClassName 是`weblogic.jms.common.StreamMessageImpl`的对象，而不在再是`AnnotationInvocationHandler`对象。`StreamMessageImpl`不在黑名单中，这里的判断不会进行抛异常。

下个断点直接落在`StreamMessageImpl.readExternal`中跟踪一下。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223150104-9f93662a-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223150104-9f93662a-44ec-1.png)

看到调用栈这里就应验了我们前面所提到的关于`StreamMessageImpl.readExternal`调用问题。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223150115-a61222f2-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223150115-a61222f2-44ec-1.png)

这里的 var4 为正常反序列化后的数据，而后面会 new 一个`ObjectInputStream`类传递 var4 参数进去。然后再调用`readObject`方法

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223150126-ac69da14-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223150126-ac69da14-44ec-1.png)

执行完这一步后，命令就已经执行成功。后面的是对 CC 链执行命令的一个基本认知，在此不做赘述。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201223150143-b712c250-44ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201223150143-b712c250-44ec-1.png)

### 参考文章

```
https://xz.aliyun.com/t/8443#toc-6

https://www.anquanke.com/post/id/224343#h3-6

```

0x05 结尾
-------

其实摸清楚补丁的一个套路过后，再去基于补丁分析后面的漏洞会比较清晰。因为补丁无非就是再从 ClassFilter 里面添加黑名单列表，这也是为什么 weblogic 修修补补又爆洞的原因，并没有从根本原因去进行修复。如有不对的地方，望师傅们指出。