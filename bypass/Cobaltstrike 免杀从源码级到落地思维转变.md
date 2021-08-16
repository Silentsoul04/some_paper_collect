> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KWKQjTatbSyFpbOf8lGGhA)

文章来源；
-----

https://www.freebuf.com/articles/web/258988.html  

---------------------------------------------------

前言
--

距离上一篇文章《那些 FastJson 漏洞不为人知的事情（开发角度）》已经过去三天了，但在我的心中仿佛过了一年。这三天是我在分析 Cobaltstrike 源码的一个过程，阅尽代码冷暖，但我依然要说一句 Cobaltstrike 牛逼~

场景描述
----

最早我为了研究 MSF 的免杀，去看了 MSF 木马源码，其采用大马传小马的方式，运行时首先将自己在内存中循环调用三次，也就是为什么会有 spwan 这个值的原因。它代表将自己在内存中循环调用几次，随后在最后一次时与服务端建立连接，然后服务端将大马传回。后来依照这个逻辑，我自己编写了 MSF 的客户端对接服务端做到了 VT 全过。原本想着按照这个思路可以将 Cobaltstrike 的木马也进行相同逻辑，但随后发现 Cobaltstrike 并不能生成木马源码，只能生成 shellcode。后来听说了加载器的概念，编写一个可以运行 16 进制的代码无论语言。由于我是做 Java 的，首先就想到使用 Java 实现，后来查找了相关资料，运用了很多相关途径都没有找到 Java 可以运行 shellcode 的逻辑。后来想起当时开发时用过的关键字 Native，并且参考了 Python 实现。产生了以下思路: python 提供了 python 与 c 之间的数据类型转换库，可以直接与 c 对接，叫做 ctypes。这点 Java 却没有，Java 与 c 对接只有 Native，通过 JNI 的方式与 c 对接。但是这其中存在一个问题，通俗点说就是创建 c 程序去运行 shellcode，然后使用 java 去进行调用。也就是 Java 是程序的入口唤醒 c 程序，这样的实现丝毫没有任何意义，不如拿 c 直接去写。所以果断放弃了 Java，回归最开始的思路，采用源码级免杀。但不得不接受的一个现实就是我们并没有此源码，所以我找了很多朋友能把 shellcode 逆向成 Java 代码。结果也是不了了之，直到我狠下决心决定分析 Cobaltstrike 的源码，查找它生成木马的逻辑。这期间我有两种逻辑，也是我的猜测: 1.Cobaltstrike 中 shellcode 是写死的，整个 16 进制都是相同的，只有反弹 shell 的地址和端口发生变化。在生成时只会替换端口和 IP 改动所发生位置，然后进行 16 进制直接替换。所以可能分析整个源码也会找不到木马源码。2.Cobaltstrike 中存在源码，每次用户选择不同的端口和 IP 之后将其代入源码，然后将源码编译从而产生 shellcode, 这种情况源码有迹可查。通过上述两种推理从而产生了我今天的文章，下面是我推理源码的过程。

环境准备
----

如果需要分析源码则必须将 Jar 文件反编译成. java 文件，然后在编辑器中运行，这样方便调试。像从前一样我还是拿出了我的反编译工具 Jd-jui![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEzGwT8V5iaM2IDf5BT1j7LJggJh6v67WjWPk0Bkw7WAIPtUUxnVU7vjg/640)

没错，结果卡在了这个页面半个多小时。我以为我电脑的问题，找了几个朋友都是这种情况后来放弃了。猜测可能存在以下原因: 1.Cobaltstrike 有什么内部安全措施，不让进行反编译。2. 文件太大导致 jd-jui 直接卡死，毕竟 Cobaltstrike 功能实在太多。

最后在公众号中碰巧在我迷离之际碰到了这个大佬发的![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEv4WAj0lbVf3VDCCmkggyk7SXRTC39W6Bh9NicTZTwLVpf1OWfQSKlKA/640)

依照这篇文章我成功的在自己的 idea 上跑起来 Cobaltstrike![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEQGbibHjjibQtUJuYBwx9oqibW2glag0wUHBYfrep3Mn9ZRwh73DxbKAXg/640)

关键代码查找
------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eE2FibgejObvykgqznlx0OWcfTVh0ric6VYauHicbVMibIqWtnJ2LCCegVzw/640)

此处为生成 payload 的关键代码，该类提供了三个方法

第一个方法

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEqeeqEfFd2zm7OCHSQfPxazSPMMzNiazl9riajoSwIsZc7svuRd40Nfaw/640)

该方法为选择监听器所执行的逻辑处理

第二个方法

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEgFVibTkgekTkoDHkUwbUR3qm066ETdaF2EHlxN2jqdOLc5BVWbHAukw/640)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eExS7QhpdrQ01gqV8ElDDicYzsoq0PhFIWNueHSHOBYQXUbngmnpCPdPw/640)

该方法为生成 payload 的主要逻辑，通过用户所选语言生成 shellcode

第三个方法

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEElJbZuGrINVoT25XWxGbPfooRPm3a5pP2vGwKE2pZfo9H9aISzvicjA/640)

该方法为 UI 界面填值，![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEC7GgC28M3qaXh0cIXsEEC8qybWic1Rib31JeUIzibGFZMQB6ic931BjMtA/640)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEh3q2x5boO8oKwTxbZQ0Z1tmb60U5NfeH1kfvN9uxIvhSmGzfmOeMtg/640)

接下来我着重分析第一个方法和第二个方法

```
public void dialogAction(ActionEvent var1, Map var2) {
this.options = var2;
boolean var3 = DialogUtils.bool(var2, "x64");
String var4 = DialogUtils.string(var2, "listener");
this.stager = ListenerUtils.getListener(this.client, var4).getPayloadStager(var3 ? "x64" : "x86");
if (this.stager.length == 0) {
if (var3) {
DialogUtils.showError("No x64 stager for listener " + var4);
} else {
DialogUtils.showError("No x86 stager for listener " + var4);
}
} else {
Map var5 = DialogUtils.toMap("ASPX: aspx, C: c, C#: cs, HTML Application: hta, Java: java, Perl: pl, PowerShell: ps1, PowerShell Command: txt, Python: py, Raw: bin, Ruby: rb, COM Scriptlet: sct, Veil: txt, VBA: vba");
String var6 = DialogUtils.string(var2, "format");
String var7 = "payload." + var5.get(var6);
SafeDialogs.saveFile((JFrame)null, var7, this);
}
}
```

```
public void dialogResult(String var1) {
String var2 = DialogUtils.string(this.options, "format");
boolean var3 = DialogUtils.bool(this.options, "x64");
String var4 = DialogUtils.string(this.options, "listener");
if (var2.equals("C")) {
this.stager = Transforms.toC(this.stager);
} else if (var2.equals("C#")) {
this.stager = Transforms.toCSharp(this.stager);
} else if (var2.equals("Java")) {
this.stager = Transforms.toJava(this.stager);
} else if (var2.equals("Perl")) {
this.stager = Transforms.toPerl(this.stager);
} else if (var2.equals("PowerShell") && var3) {
this.stager = (new ResourceUtils(this.client)).buildPowerShell(this.stager, true);
} else if (var2.equals("PowerShell") && !var3) {
this.stager = (new ResourceUtils(this.client)).buildPowerShell(this.stager);
} else if (var2.equals("PowerShell Command") && var3) {
this.stager = (new PowerShellUtils(this.client)).buildPowerShellCommand(this.stager, true);
} else if (var2.equals("PowerShell Command") && !var3) {
this.stager = (new PowerShellUtils(this.client)).buildPowerShellCommand(this.stager, false);
} else if (var2.equals("Python")) {
this.stager = Transforms.toPython(this.stager);
} else if (!var2.equals("Raw")) {
if (var2.equals("Ruby")) {
this.stager = Transforms.toPython(this.stager);
} else if (var2.equals("COM Scriptlet")) {
if (var3) {
DialogUtils.showError(var2 + " is not compatible with x64 stagers");
return;
}
this.stager = (new ArtifactUtils(this.client)).buildSCT(this.stager);
} else if (var2.equals("Veil")) {
this.stager = Transforms.toVeil(this.stager);
} else if (var2.equals("VBA")) {
this.stager = CommonUtils.toBytes("myArray = " + Transforms.toVBA(this.stager));
}
}
CommonUtils.writeToFile(new File(var1), this.stager);
DialogUtils.showInfo("Saved " + var2 + " to\n" + var1);
}
```

此方法首先监听用户所选 x86 或者 x64 的操作系统然后注入到全局

```
protected byte[] stager = null;
```

此全局数组为用户所选的配置，如监听的方式，IP 端口等。用户所选的可变操作都会存放到此，简单来说就是 payload 生成的可变参数。然后判断如果配置错误则会弹出相应的内容，如果成功则将界面显示的语言值和程序内部实际要匹配的值进行转化。为下面第二个方法做铺垫，此时用户不管是 x86，x64 还是监听方式, 所选的生成语言，IP 端口都已经放到 stager 中。

第二个方法

```
public void dialogResult(String var1) {
String var2 = DialogUtils.string(this.options, "format");
boolean var3 = DialogUtils.bool(this.options, "x64");
String var4 = DialogUtils.string(this.options, "listener");
if (var2.equals("C")) {
this.stager = Transforms.toC(this.stager);
} else if (var2.equals("C#")) {
this.stager = Transforms.toCSharp(this.stager);
} else if (var2.equals("Java")) {
this.stager = Transforms.toJava(this.stager);
} else if (var2.equals("Perl")) {
this.stager = Transforms.toPerl(this.stager);
} else if (var2.equals("PowerShell") && var3) {
this.stager = (new ResourceUtils(this.client)).buildPowerShell(this.stager, true);
} else if (var2.equals("PowerShell") && !var3) {
this.stager = (new ResourceUtils(this.client)).buildPowerShell(this.stager);
} else if (var2.equals("PowerShell Command") && var3) {
this.stager = (new PowerShellUtils(this.client)).buildPowerShellCommand(this.stager, true);
} else if (var2.equals("PowerShell Command") && !var3) {
this.stager = (new PowerShellUtils(this.client)).buildPowerShellCommand(this.stager, false);
} else if (var2.equals("Python")) {
this.stager = Transforms.toPython(this.stager);
} else if (!var2.equals("Raw")) {
if (var2.equals("Ruby")) {
this.stager = Transforms.toPython(this.stager);
} else if (var2.equals("COM Scriptlet")) {
if (var3) {
DialogUtils.showError(var2 + " is not compatible with x64 stagers");
return;
}
this.stager = (new ArtifactUtils(this.client)).buildSCT(this.stager);
} else if (var2.equals("Veil")) {
this.stager = Transforms.toVeil(this.stager);
} else if (var2.equals("VBA")) {
this.stager = CommonUtils.toBytes("myArray = " + Transforms.toVBA(this.stager));
}
}
CommonUtils.writeToFile(new File(var1), this.stager);
DialogUtils.showInfo("Saved " + var2 + " to\n" + var1);
}
```

此方法判断用户所选择的生成语言进入不同的 shellcode 生成, 将第一个方法获取到的可变参数传入

进入![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eExDVSxYQDEKicMneJdBNSESO5RvxnrMEhW2oOdyEEhFgxLq7Q3614GkQ/640)

生成代码内部我一下子明白了很多，var0 是用户配置参数，最后却成为了最后生成 shellcode 的内容? 并且参数都是 var0，此处应该有一个观点就是所有语言生成的 shellcode 其实是同一套东西

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eETKm8JM84nTziaZoAict4Jf1huNFhUTyuhmyg8I1G2QicibP25EfB2UtfEQ/640)

只是不同的语言声明数组的方式不同，然后根据语言不同去凑编码。然后把内容给 Packer 对象赋值进行下一步处理。

这正验证了我分析前提出的第一种猜想，框架源码中根本没有木马源码。所以源码级免杀的想法到这里结束了。

思路转变
----

没有源码只能又回归刚开始从 shellcode 下手，但是 java 不行。又只能从 python 下手，根据网上的例子从加载器上入手做源码级免杀。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eE25D60ShIesjbFVdM5kxsYhjDraUwiayhnSGEIyiazp8ZCoZJ058qbWxQ/640)

把加载器的代码删删改改，各种方法抽离，各种变量名称更改，最后

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEoSV1mdm8tjn76GNyvZd3cERStS9FIlr9f62bibCyYaHnN8anjHIInBg/640)

然后下载了哈勃，360 等实战依然过了。

说明只要会代码，免杀随便玩。

总结：
---

从木马源码级免杀到最后走投无路只能又从加载器源码免杀入手，其实我想简单了，但又想复杂了。复杂在于目前国内的杀软依然停留在静态查杀的地位 (个人观点，可以交流但别喷) 只是更新特征库真的快, 造成这种原因可能是因为病毒和杀软之间杀软本身所处的地位不占优势，也就是被动，先功后防的状态。简单在于通过查看 Cobaltstrike 源码，他的逻辑和思路另我这个开发也自愧不如，虽然国内的 it 水平已经日益见长，但是进步空间真的很大。

谢谢大家，可以继续关注我，关注我的专辑

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/k89AZuPTXhw4YrRYB9q5zda3k1c4K0eEabLsUnC3NspcjSJDwcLVE9vQlHKNQeezWQuibx6DDjIXn7lAjqtqbIA/640)原文地址；

https://www.freebuf.com/articles/web/258988.html

原作者授权转发