> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/4547.html)

> 文章目录 0X00 背景 0×01 基础原理 0×02 关于破戒 Exit 暗桩 0×03 CDN + 反代隐藏 TeamserverProxy0×04 DNS 上线一个未填的坑 DN…

文章目录

*   0X00 背景
*   0×01 基础原理
*   0×02 关于破戒
    *   Exit 暗桩
*   0×03 CDN + 反代隐藏 Teamserver
    *   Proxy
*   0×04 DNS 上线
    *   一个未填的坑
    *   DNS Listener 特性
*   0×05 结语

0X00 背景
-------

**最近在做渗透测试相关的工作，因工作需要准备用 Cobalt Strike，老早都知道这款神器，早几年也看过官方的视频教程，但英文水平太渣当时很多都没听懂，出于各种原因后来也没怎么深入了解，所以一直都是处在大概了解的层面上。直到现在有需求了才开始研究，过程中体会也是蛮深，技术这东西真的不能只停留在知道和了解这个层面，就像学一门语言一样需要多动手去实践才能熟练运用的。**

当然在深入研究某一门技术的过程中难免遇到各种各样的问题，一步一步解决这些问题才是真正学习的过程。对 Cobalt strike 的学习和研究中我也同样遇到很多的问题，幸得一些素不相识的师傅无私帮助，才解决掉所有的问题，这里把过程中一些问题和解决办法记录下来，以便以后查阅，同时也希望对刚接触 Cobatl strike 的朋友有所帮助。

0×01 基础原理
---------

基础使用和原理网上有大把的文章和教程，我这里只阐述我个人理解的几个基本点，先说 stage 和 stager, 在传统的远程控制类软件我们都是直接生成一个完整功能的客户端 (其中包含了各种远控所需功能代码)，比如灰鸽子（这里年龄已暴露)，然后将客户端以各种方式上传至目标机器然后运行，运行后目标机器与我们控制端点对点的通讯。而 Cobalt strike 把这部分拆解为两部(stage 和 stager)，stager 是一个小程序，通常是手工优化的汇编指令，用于下载 stage、把它注入内存中运行。stage 则就是包含了很多功能的代码块，用于接受和执行我们控制端的任务并返回结果。stager 通过各种方式(如 http、dns、tcp 等) 下载 stage 并注入内存运行这个过程称为 Payload Staging。同样 Cobalt strike 也提供了类似传统远控上线的方式，把功能打包好直接运行后便可以与 teamserver 通讯，这个称为 Payload Stageless，生成 Stageless 的客户端可以在 Attack->Package->Windows Executeable（s）下生成。这部分我也是在研究 dns 上线时候才算分清楚，这里需要感谢 B0y1n4o4 师傅的帮助

0×02 关于破戒
---------

目前网上公布版本大多为官方试用版破戒而来且最高版为 3.14（5 月 4 号）版，我托朋找了一份 3.14 官方原版的来，原版的本身没有试用版那么多限制，破戒也相对容易，只需绕过 license 认证即可，这里在文件 common/Authorization.class 的构造函数中。

```
public Authorization() {
```

这里需要把该类的 `watermark`、`licensekey`、`validto`、`valid ` 实例域手动设置即可，如下面代码

```
String str = CommonUtils.canonicalize("cobaltstrike.auth");
```

剩余下其他关于 listener 个数限制可以参考 ssooking 师傅的博客查看。

### Exit 暗桩

剩下的一个就是 Cobalt strike 作者留下的一个 exit 的暗桩问题，这里还是请教了 ssooking 师傅才知道，否则我这 Java 渣渣不知道要找到什么时候，这里再次对 ssooking 师傅的帮助表示感谢。

作者在程序里留了个验证 jar 文件完整性的功能，如果更改了 jar 包的文件 这个完整性就遭到破坏，作者会在目标上线 30 分钟后，在此以后添加的命令任务后门加一个 exit 的指令，目标的 beacon 就自动断开了，如下图。

![](https://image.3001.net/images/20200310/1583843827_5e6789f3444a0.jpg!small)

代码在 `beacon/BeaconC2.class`

```
if (!(new File(str)).exists())
```

```
try {
```

再看 beacon/BeaconData.class

```
File file = new File(getClass().getProtectionDomain().getCodeSource().getLocation().toURI());
```

破戒方法是直接更改 `shouldPad` 方法中的 `this.shouldPad = paramBoolean;` 为 `this.shouldPad = false;`

0×03 CDN + 反代隐藏 Teamserver
--------------------------

这部分原理参考垃圾桶师傅的文章 ([点这里])，这里帮垃圾桶师傅填一个他在文章中说遇到的坑。

![](https://image.3001.net/images/20200310/1583844068_5e678ae4cfcf7.png!small)

![](https://image.3001.net/images/20200310/1583844092_5e678afc35b8b.png!small)

这里垃圾桶师傅在添加 Listener 的时候 Host 填写的是 CDN 的地址，在使用 powershell 下载 `stager` 运行，`stager` 再去下载 `stage` 的时候就是直接访问 cdn 的地址下载，但是 `malleable profile` 没有配置制定 stager 的行为，所以无法正常回源到 teamserver 下载，这里只需要在 profile 文件中配置 `http-stager` 模块，像 http-get 一样指定好 Host 即可从 CDN 访问到 teamserver 下载 `stage` 了。

### Proxy

反向代理原理这里借用垃圾桶师傅的图, 我就不具体再阐述，垃圾桶师傅已经讲得很明白。

![](https://image.3001.net/images/20200310/1583844128_5e678b20f2306.png!small)

我使用的是 Nginx 做的反向代理，这里如果刚研究这个的朋友可能会遇到客户端上线后 IP 是 Nginx 服务器 IP，走 CDN 的时候显为 CDN 节点 IP 的情况，这里有两个解决办法，先看看 `server/ServerUtils.class` 类中代码：

```
if (file.getName().toLowerCase().endsWith(".jar"))
```

这里 Cobatl Strike 可以从 `HttpHeader` 中的 `REMOTE_ADDRESS` 和 `X-Forwarded-For` 中取得 IP，我们要么在 Nginx 反向代理的时候设置 `REMOTE_ADDRESS` 值，要么在 profile 的配置文件中的 `http-config` 模块设置 `trust_x_forwarded_for` 值为 `true`，这也是看了代码从知道有这个配置，英文渣渣表示很惭愧，官方写得很详细。

这里有个问题就是反向代理时候自定义 `REMOTE_ADDRESS` 时候往往无效，不知道具体啥情况，我之前在另外的机器上都有测试成功过。

0×04 DNS 上线
-----------

### 一个未填的坑

这个坑是研究和使用 Cobalt Strike 来最大一个坑，至发文今日都没有解决。问题是出在使用 DNS 的 listener 不管是 `beacon_dns/reverse_http` 还是 `beacon_dns/reverse_dns_txt` 时候，若使用 `staging` 方式 `stager` 在下载 `stage` 注入到内存中的时候崩掉，如下图。

![](https://image.3001.net/images/20200310/1583844208_5e678b70c8166.png!small)

而若使用 `beacon_dns/reverse_http` 时候，选用非纯 dns 模式就没问题，非纯 dns 模式状态下 stager 在下载 stage 时候使用 http 方式，stage 只要成功下载注入内存后便可以 mode 改用 dns 方式来通讯了，要是有师傅知道怎么回事还赐教。

### DNS Listener 特性

最后经 B0y1n4o4 师傅指点，改用 stageless 方式上线就没有问题了。但是在使用 dns 上线的时候还需要注意个问题。在添加 Listener 的时候 `beacon_dns/reverse_http` 和 `beacon_dns/reverst_dns_txt` 都需要填写端口信息，如下图。

![](https://image.3001.net/images/20200310/1583844443_5e678c5ba9338.png!small)

如果端口使用 80 的情况下，上线之后的通讯优先使用 http 方式，若想用纯 dns 通讯的话就需要在上线之后首先使用 `mode` 指令切换至 dns、dns-txt 或者 dns6 模式。添加 listener 自定一个非 80 的端口上线之后所以的通讯都将默认采用 dns 方式，且不能使用 mode 切换成 http 模式。

0×05 结语
-------

以上均为我个人一些研究测试结论，有不到之处还请多多指正，Cobalt Strike 确实是一个蛮强大的工具，还有很多内容和技术有待研究，本人也正在学习 Java，争取早日通读内核代码。

*** 本文作者：Ca3tie1，转载请注明来自[林哲博客](https://www.lz80.com/category/zydq/asmr)**