> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cRlHoilJa8HPX9L2ZNE77A)

这是 **酒仙桥六号部队** 的第 **124** 篇文章。

全文共计 2938 个字，预计阅读时长 9 分钟。

在我们使用 cobaltstrike 的过程中，会涉及到二次开发，从而使其功能上更加的健壮，不至于碰到杀软就软了的地步，本文从 cobaltstrike 的快速反编译到二次开发环境的准备作为二次开发 cobaltstrike 的起步。

`IntelliJ IDEA`自带了一个反编译 java 的工具，有时候我们需要对`cobaltstrike`的整个`jar`包进行反编译，使用这个`IntelliJ IDEA`双击之类的反编译时要是对整个源码层面进行搜索并不是很方便，可使用其自带的反编译工具，可以做到批量的整个反编译。

**一 CobaltStrike 反编译**

这里先在`IntelliJ IDEA`安装目录找到`java-decompiler.jar`拷贝到一个准备好的目录，并且新建两个文件，一个`cs_bin`里面放未反编译的`cobaltstrike`再建一个`cs_src`文件，这个是空文件，是为了之后放反编译后的`cobaltstrike`

```
/Applications/IntelliJ IDEA.app/Contents/plugins/java-decompiler/lib/java-decompiler.jar  //找到decompiler文件
cp java-decompiler.jar /Users/name/Desktop/wen/学习资料/java/cstips001/  //拷贝到准备好的目录
```

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDvXkDn8MZR966pnv7KxZyF1YGA3GNgF7uBk3HGWQGOAeKwy4MlseMXw/640?wx_fmt=png)

在`java-decompiler`中找到 decompiler 的路径，提取出来如下：

```
org/jetbrains/java/decompiler/main/decompiler/
```

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbD5du2Z051McGQ187icSKK6icib7YKPvic8zTNmDyI5jLE8Lr6Yz7cXmnfMw/640?wx_fmt=png)

把路径提取出来后，把反斜杠全部替换成`.`随之再其后加上`ConsoleDecompilers`，如下就是提供反编译的这个类。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDIUiatXdqS3pJzYicE4Q5zoUJLJTYLj6nhlMIe5PUuGDic3JZ8VgnViaKicw/640?wx_fmt=png)

```
org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompile
```

因为`MANIFEST.MF`中没有`main class`属性，没有指定主类，因此不能直接使用`java -jar`，如果想要执行`java`包中具体的类，要使用`java -cp`输入如下命令：

```
java -cp java-decompiler.jar org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler
```

执行的时候会有提示。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDzTvMfv9UDMrlyribl43YdiaOcTHTvFQ3HvdenLRtQEo7P7h6ClKZX2hA/640?wx_fmt=png)

让你加上`-dgs=true`后跟需要反编译的`cobaltstrike`和反编译之后要把`cobaltstrike`放入的目录，就是我们最开始建立的`cs_src`这个是存放反编译后的`cobaltstrike`，运行这条命令即可对整个`jar`包开始反编译。

```
ava -cp java-decompiler.jar org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler -dgs=true cs_bin/cobaltstrike.jar cs_src/
```

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDUbkRjrrQOPgsxWIdCxNbia7Eh8V5wuDw4icZqIeU9eoHAAhmicze5VD7A/640?wx_fmt=png)

反编译后，会自动打包成`jar`包，右键解压后打开可以看到都是`.java`了，使用这个方法会非常方便，就不需要第三方工具, 这个反编译出来的就可以直接放入`IntelliJ IDEA`中，可直接实现代码搜索，相关的交叉引用。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbD8u5ZA4Mbk22OakHVcOl4uLMTtKq4a86bOK6UPagyXUGjIOyX4B91UA/640?wx_fmt=png)

**二 CobaltStrike 二次开发环境**

打开 `IntelliJ IDEA` 选择`Create New Project`一直选择 Next。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDIxfI1bZ8rkRyQl4LibZNXQ4iaBccfubzVGfOpojpwDgoDKx4uNdt5Picw/640?wx_fmt=png)

这里选择路径跟起个名。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDecqzTcp1AsEH7rN2oiaLiac9NDVkNt53ZCJQkdoQucU2kY4zkcryov7A/640?wx_fmt=png)

创建好后需要先建立两个文件夹，右击选择`New Directory`建立一个`decompiled_src`文件夹，之后再建立一个`lib`文件夹。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDa7lWDV2QqCWoPUWQxlrHkibNMjuT1PDUKeFWGciaWxOBp9CicauhICj2g/640?wx_fmt=png)

把在`CSTips001`中反编译好的`CobaltStrike`复制到`decompiled_src`中，然后把它解压出来，可看到一个完整的反编译后的目录。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbD3OX0gDeDGrNKBKx7jzyvyK4XNqk4VjZgk8Q0ASCPIFQrjWxiadPuLgQ/640?wx_fmt=png)

随后把原始的未编译的`CobaltStrike`放到刚刚新建的`lib`中去。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbD6peicVaiaiaRpsCUcDLudYJ55WzXGiaj5kCFsqpxGE3dwWjv6tacz6mVrA/640?wx_fmt=png)

接下来需要对这个项目进行设置, 点击`File`中的`Project Structure`在`Modules`对`Dependencies`进行设置。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDSOXU5V2upBTyqdbHD6Ljwq3vQC2QbGy3oErWAmAdx38vwbJUqqF1EA/640?wx_fmt=png)

选择 lib 中的 cobalt strike.jar，确认是 Compile 之后勾选一下，然后选择 Apply。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDFEetXADrKMcccia0xWk66OAUSRORRA6tUtTG2y9xyrVFUhZUF28AVsQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDVgzSqibVFNxBjw4PLgznTFn0OFsWQea1ia8srjeh84OIiaFsic9IISzWZA/640?wx_fmt=png)

至此依赖关系设置完了，现在进入`Artifacts——>JAR——>From modules with dependencies`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDL8JibcXXfJibWnbjOzSIES2ke6OW82Ijxtx8L07qVLzqkraj6ZLEHauA/640?wx_fmt=png)

这里需要一个填写一个`Main Class`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDkibIyvGwVqK8TDqPE2sMDoXPfCzrLuQFQ3YW3GEvfPKxP1J3CQxts4w/640?wx_fmt=png)

目前我们还不知道这个`Main Class`该填什么，可以点在 lib 中的 META-INF 里双击 MANIFEST.MF，我们可以看到 Main Class，复制`aggressor.Aggressor`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDa29XtEpHw6HvsUnCUsrsEDVSZMXyBcfuISPC8f1r54RpTmq2pjOcvA/640?wx_fmt=png)

再次打开`Artifacts——>JAR——>From modules with dependencies`在`Main Class`处填入`aggressor.Aggressor`选择 OK, 这里就设置完成了。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDic28iclGTSKWzpU9Koz3mfLwL1EYV0RHcqnSXFzGRatAw06oX8j8hBrw/640?wx_fmt=png)

接下来在`decompiled_src`目录中找到已经反编译完的`aggressor`主类, 右击选择`Refactor ——Copy File`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDscDEZ6Zeyib4uYdqc0FdkPVA1LvwGCYTcx5zFhB3ibChwxQFbia3YkeFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDN4QjCEhYEAaMVQn7fx4lwfx708W9iasyT32JkEWUzRvDeWhAeu7hslQ/640?wx_fmt=png)

在`To directory`点击添加，选择之前创建的`src`在其中添加一个`aggressor`名字要一致，最后点击`Refactor`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDcqwyckexA3MqmLVL9QzasAruIlUkXvwQjAuJeblmLkCsnxiaoD372LA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDzmibcQz4WGy9O1JBbSibdrchaZhvcbfB1YpLLhLgrGvA7tYBe40KSCEA/640?wx_fmt=png)

这样`aggressor`就自动的被拷贝到`src`目录里去了，这里可以看一下, 如图。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDzePocsxPAr1Mwr2rBcGc4BauDBzCdPPmXn0DftA8Gw41bicyLP6C3gQ/640?wx_fmt=png)

测试一下，修改文件，保存。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDpHTl2GsK8dG9wPib3EAdlBChGctvh5aoic22KsxkKsvqtf8mUJPctvXg/640?wx_fmt=png)

到这里我们的整个准备工作就完成了，之后就是我们要修改哪个文件，就可以在完整的源码中找到那个文件，然后右键`Refactor`然后`Copy File`到这个目录然后进行修改，修改完成之后就可以选`Build——>Build Artifacts ——>Build`进行编译。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDUWyL8gmhwgVgA6GPLRAmnBvEh8GlxZkib9e1GwxEPxloSiayiaNQnleYw/640?wx_fmt=png)

当提示`Build completed successfully in 4 s 227 ms (a minute ago)`的时候，会生成一个`out`文件夹, 其中可看我们的编译好的`MycustomCS.jar`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDtCXUtQUWqLjHw2K5bnws1p7sHkobc3yspE2r6eqQ4FfE9YxJDK0BCA/640?wx_fmt=png)

在每次调试运行的时候，不需要切换到命令行环境，可以直接选择`Run`中的`Profile`设置参数。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDh3QRNyBXmibE5guVOoYRwWsPQZtlc1kw50b5SoNTadhcPzYJQ0ckw8w/640?wx_fmt=png)

选择➕号，在`JAR Applic`添加一个配置文件。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDiam6OibHzdCVBrxTtyEcAA2zl89w5mTAStkItO1QQzYuCMeWg2p1Tqcg/640?wx_fmt=png)

在`Path to JAR`中选择`out`文件中我们修改并编译好的 jar 包，选择好后点击`Apply`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDBib3r62LiabpUJ9hWaQE46AC8EtYcjBqYIykLGkYoyC50Cl50Hv38xgw/640?wx_fmt=png)

最后在`Run`中选择`Run CustoomRUn`即可看到消息窗。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbD779fZXz4V2c6Y1Z5LZy8bteZ5NhM6eFQeRpbgnNb4oibgGqvnvkKrFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDib3D1PX6FCQUB0xFNLuiawQJUhpgWb0iaNkl2qa4IicGfS5LibS9fYFhOWA/640?wx_fmt=png)

点击确定，发现弹出提示，没关系，继续点击确定。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDibVYBoUia3rjWnhE0eglm3LhpZoxEY8dLYNjxZlzrdBMjkzp8VeR8pRQ/640?wx_fmt=png)

我们把`-XX:+AggressiveHeap`复制下来。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDicdJNJRc2NsiaougE5oLRXJ6PdH02mKICU4LLVtMj1VWGtzYrZIIr06w/640?wx_fmt=png)

再放到`Run`的`Profile`这里就直接选择之前创建的`CustomRun`填入`VM options` 最后选择`Apply`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDcHpKxKkwdstSPYPaT9iaibtJfeSGUeK2tRfiaVR5dsd9dsfE0ibYdYN4oQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDsItmibrFMKPicXlWNdt06RaFibG7T6t9ticVcSzRdcTGRpjib9CmTbLicDOg/640?wx_fmt=png)  

再次`Run`运行, 继续复制`-XX:+UseParallelGC`

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDfWkc0S0BlK3iaYsaia64iaKibDgic1aUIj5NymnQl5wicYzBZvYtWaUbF1iaA/640?wx_fmt=png)

继续在添加到`VM options`中，记得要用空格隔开。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDbEX9oSupQicJ7dBo9Pv41FsU5haMsu2R5126CzACX0awofu3ia2AVIkQ/640?wx_fmt=png)

再次运行，提示`auth`文件找不到。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDFNRtn5ia9LuEpRwTIU5hXPaE7dbaB11kcwu1XbaAHOXCCyc1uYoowIA/640?wx_fmt=png)

这里把初始的`cobaltstrike.auth`文件复制到`MycustomCS.jar`同目录下。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDsLpFbc5r5PdibcM9ia5j6iciakwxia8Ntp58PHMulls8go57uMyLdN0N27g/640?wx_fmt=png)

最后运行, 到这里我们需要进行二次开发的环境就搭建好了，接下来就可以根据我们的需要通过关键字在源码中定位到具体的功能实现的代码，从而进行修改，或作一些功能上的增强。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54bra8QV5ltkjJklMGicrzbDX3YiatMHtLKmGI5Yj1QF4U1JzU6pHKn1YpPjhAtIZiaEgmlichd6MrpsA/640?wx_fmt=png)

**三 总结**

本文使用 IntelliJ IDEA 自带的 java 反编译工具，对 cobaltstrike 进行了快速的反编译，并展示了修改后的效果，权当抛砖引玉，具体的插件开发各位师傅可以结合具体场景进行编写。

**参考资料**

红队学院 CSTips

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCib3YrXdutsRQR3FylKf61fVYyFn1oHibMtpxGDMO919ickU2vRTK8bnYWg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibiaRBNn0A5YI88OyFjU8fn2Isf9bat4vQn18NwG6cXxVOSuKiapNm2nibQ/640?wx_fmt=png)