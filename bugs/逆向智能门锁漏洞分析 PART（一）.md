> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMTc1MjExOQ==&mid=2247492107&idx=1&sn=f1add5d8c1868a516cd6b64c0cb5d436&chksm=e89dcad3dfea43c5741e3bcbc14163270b4bc85d0edd40b9219e045332b2a0c20607352f9df0&scene=21#wechat_redirect)

**简介**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tRTtlxM78K05gPlG4TqSnWrdU2pckicskriaWg3og7ia2ibiblJzQ3icQJyzQ/640?wx_fmt=png)

最近，我有机会研究一家知名安全公司（F-Secure）发现的 **Guardtec KeyWe 智能锁漏洞**。F-Secure 人员发现，由于设计缺陷，攻击者可以拦截和解密来自锁的合法所有者的流量。他们的博客（发布于 2019 年 12 月）非常吸引人，信息量很大。我立马就想看看能否重复他们的努力，意识到 F-Secure 已经发布了一个通报，并且供应商有机会减轻他们的曝光率。不幸的是，由于他们没有固件更新功能，它们的缓解选项非常有限。相反，他们选择在他们的 Android 应用程序中使用模糊处理，试图隐藏更相关的代码部分（他们做的相当不错，我可能会添加）。  

虽然 F-Secure 已经打下了基础，但他们小心翼翼地不透露太多信息，甚至修改了自己的一些工具，从而保留了他们说的 "王国的钥匙"。在我的旅途中，我发现自己经常回到他们的博客，特别是当我发现了新的相关信息。这个博客的目的是，不仅仅是整理我的笔记和记录我的研究，但也许通知其他人一些很酷的工具和方法，用于逆向工程 Android/iOS 应用程序。

收到我的 KeyWe 智能锁的货件并创建一个测试夹具来安装它后，我下载了 Android 应用程序到我的手机，并创建了我的帐户。我熟悉了锁的功能和移动应用程序的外观和感觉。我还运行了我的 Nordic nRF Connect 移动应用程序（可在谷歌 Play 商店免费）来获取有关我的锁的有用信息，如蓝牙地址，主要服务 UUID，特性等。

**注意：**了解 BLE 的 GATT 和 GAP 超出了这篇文章的范围，但是，如果您想了解更多 BLE 规范可访问 https://www.bluetooth.com/specifications/。

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tpRICYLL0UtRwXq9547ZR0asghS4NLP4gMKAuSaRnYSLgjUu3iagZHxQ/640?wx_fmt=png)

**有用的信息**  

KeyWe（锁定）和（移动）应用程序之间的通信通过蓝牙低功耗 （BLE） 数据包传输，这些数据包使用标准 ECB AES-128 密码进行加密，以防止第三方窃听。消息通道的安全性仅基于用于加密和解密 OTA（无线）AES-128 数据包的 3 个密钥。

• 用于初始密钥交换的公共密钥（公共秘钥）

• 用于加密应用程序发送到门的数据包的应用程序密钥（AppKey）

• 用于加密门发送到应用程序的数据包的门钥匙（门键）

**密钥生成**

CommonKey 完全基于静态 16 字节值，该值只需要设备蓝牙地址的最后 5 个字节来枚举，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tjkGlqgJg9HJmnETByJyQbRWqhJRj7G1XGFjncaWuOGZk5gu5uCrDmw/640?wx_fmt=png)

在进一步检查多个 KeyWe 设备上的 CommonKey 后，所有检查的设备之间唯一的区别是设备蓝牙地址的最后两个字节！在我的情况下，值 4C 和 93 是我的设备所独有。这表明 CommonKey 具有高度可预测性，并且仅基于每个设备 16 字节静态值内的两个字节！  

AppKey 和 DoorKey 由两种算法（并且经过大量模糊处理）方法创建 AppKey 和 DoorKey。F-Secure 人员创建了一个不错的工具，通过模拟这两种方法的操作来生成三个密钥（尽管他们对他们的工作进行编辑和模糊处理，使其通常无害）。然而，经过相当长的时间后，我能够反向工程这两种方法的功能，我自己使用一个方便的开源工具称为 Frida（稍后将介绍这一点）。

AppKey 和 DoorKey 创建包括将两个参数传递到 makeAppKey 和 makedoorKey 函数。这两个参数分别是 AppNumber 和 DoorNumbe 。

AppNumber 是一个静态（硬编码） 12 字节值（填充四个零字节）：92 4b 03 5f bd a5 6a e5 ef 0e d0 5a 00 00 00 00 注意：AppNumber 使用公钥加密并发送到门（锁定）作为用户会话的第一个数据包，从而启动会话。

门号是门（锁）生成的动态（更改每个新会话）12 字节值（用四个零字节填充）。此值（也使用公钥加密）将发送到应用程序，以响应接收 AppNumber。（见下图）

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8taibRxS5ib5Lg6g5dS9f4SHwVmFDdZrQKaJyhozCFvjFWyecCnUia1CyRA/640?wx_fmt=png)

注意：这两个 "打开" 传输（AppNumber 和 DoorNumber）完成了初始密钥交换过程，并允许创建 AppKey 和 DoorKey，这将提供应用程序和门之间的安全 OTA 通信。  

现在，AppNumber 和 DoorNumber 已经创建和交换了，我们有生成其余两个密钥（AppKey 和 DoorKey）所需的两个组件。这是通过调用 makeAppKey 和 makeDoorKey 函数以 AppNumber 和 DoorNumber 作为参数来完成的。这在固件内部完成，而不是发送 OTA。

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tkolff7kDgSjL3UickibFOBfO7IBMhnFyYuXtnxgAKXGWHZjmiazgOFctg/640?wx_fmt=png)

**应用程序流程图**  

现在，我们已经生成和交换了 AppKey 和 DoorKey，现在每一方都可以加密 / 解密发送或接收的数据包。应用程序传输的所有数据包都将使用 AppKey 加密，门传输的所有数据包都将使用 DoorKey 加密。双方都知道对方的加密方案，因此可以解密数据包。

下图演示典型用户会话的事件顺序：

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tURFRK25kfQXZdWvFNcAoEBEPMtlqoJ9QdibdsIiaLuvyHJIvv5dbUyNA/640?wx_fmt=png)

应该注意的是，所有这些数据包在每个用户会话开始时都传输 OTA，并且所有 13 个数据包都根据传输方向使用 AppKey 或 DoorKey 加密。  

**逆向安卓 APK**  

所有移动应用程序都下载为 APK（安卓应用程序包）文件。APK 文件以 ZIP 格式保存，通常直接下载到 Android 设备，通常通过谷歌 Play 商店，但也可以在其他网站上找到。当反向工程 Android APK 时， 我发现使用第三方网站搜索旧版本的 APK 通常很有帮助。我最喜欢的是 https://apkpure.com/。

**软件要求**

Java 版本 1.8.0_251

ADB（android debug bridge）版本 1.0.41

APKStudio （wrapper for Apktool） 版本 2.4.1

Dex2Jar

Jadx 版本 1.1.0  

Frida 版本 12.8.9

**硬件要求**

**Root 的智能手机**

典型的 APK 包含一些非常有用的内容, 例如 AndroidManifest.xml/classes.dex 和 resource.arsc 文件以及 Meta-INF 和 res 文件夹。有几种不同的方法来打开驻留在您的 PC 上的 APK。显然，因为它是一个 ZIP 文件，任何各种 UNZIP 提取器将工作得很好，但是使用像 Dex2Jar、Apktool 和 Jadx（例如几例）这样的工具提供了一些额外的优势，例如将. dex 文件转换为 java 代码，以便更好地读取和 GUI 支持代码的导航。

DEX 文件（Dalvik 可执行文件）是用于初始化和执行 Android 移动平台应用程序的开发人员文件。像 Apktool 这样的工具可以将 DEX（机器语言）文件编译到 Smali（汇编语言）文件中。我们还可以使用 dex2jar 等工具将 DEX 文件转换为 JAR （java） 文件，并使用 jadx GUI 将 JAR 文件作为 java 源代码打开。Java 源代码比 Smali 源代码更容易阅读。有许多选项可用于导航 Android APK，包括我最喜欢的， APKStudio。

有了许多可用的选项，描述实现这些工具中的任何一个所涉及的各种步骤将超出本文的范围。我建议下载它们，并尝试几种技术，以找到最适合你。有很多有用的教程在那里。

**使用 FRIDA**  

F-Secure 的研究人员在他们的博客中说，他们能够使用一种叫做 Frida 的工具拦截 Android 应用程序中的函数调用。我不知道这个工具，所以我决定搜索一下。这个工具是惊人的！了解如何实现 Frida 超出了此编写的范围。但是，只要说，Frida 允许研究人员附加到应用程序中的现有函数，并动态转储参数和返回值。这是绝对值得一看的！https://frida.re/docs/home/

**Frida w/toolkit 安装**

• pip install frida

• pip install frida-tools

**要求：**

•frida server 和 adb

• 用于调试的 apk 的手机 （我用一个旧的三星 Gs5**）**

**在 root 的安卓手机上安装 frida server**

可到 https://github.com/frida/frida/releases 安装服务器并下载用于使用的特定电话平台的适当文件。

（如果您不确定手机的架构，请下载并运行 Droid 硬件信息（来自 Google Play 商店）

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8trM4cxeBRXafiaCx0ibZI9R4dhoiceaVwCd464jGynLibvp5Pr0MhLOfogg/640?wx_fmt=png)

• 将下载的文件复制到项目目录  

• 导航到项目目录

• 使用 ：xz - d - k frida-server-12.9.8-android-arm. xz 解压缩. xz 文件

• 使用 adb 工具将提取的文件推送到根手机：

**初始设置：**（在根手机上安装 frida-server）  

·$ adb push frida-server-12.9.4-android-arm ·data/local/tmp

· $ adb shelll

· $ su

· # cd /data/local/tmp

· # chmod 777 frida-server

· # ./frida-server &

在有根手机上安装 frida-server 之后，按照以下方式开始一个新会话:

· $ adb shell

· $ su

· # cd /data/local/tmp

· # ./frida-server &

最初，我很难协调新工具（Frida）的学习，而要找到与 F-Secure 人在他们的博客中引用的相同功能调用，这又增加了一个负担。由于 F-Secure 的通报，供应商严重混淆了最新版本，这意味着不再附加 "makeAppKey" 或 "makeDoorKey" 功能。我还发现，供应商采用了安全措施，以防止在根手机上运行 KeyWe 应用程序。F-Secure 的研究人员创建了一个很酷的工具，使用 python 和 Frida javascript 附加到违规方法，并注入 java 代码，以始终将 false 返回到 "isRooted"。  

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tEjuvvUgT6FZY2hwgvia6BhUarNuOtLE2tP18tUzMUICuvwq2B0XmSSA/640?wx_fmt=png)

不幸的是，由于我较新版本的 Android APK 严重混淆，"RootTool" 类并不存在，我被迫搜索我的 APK 代码，试图找到等价的代码。经过相当长的时间搜索代码，我最终找到了根检查方法。原来 "RootTool" 类现在被引用为 "n"，"iSRooted" 函数现在被引用为函数 "b"。  

**根据我的搜索结果修改了 F-Secure 的 KeyWe-Tool 脚本**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tgjds00QSSk5PvMzxPD5MUWJLWRJ0aP8z24dgrzK7F3A7WkyziaJuyEw/640?wx_fmt=png)

**结果成功逃避根检测！**  

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tWB64ZSiaQ0WzCVM9BlPfWQqxBI9I7ZM4cjeIcnj2SjKjZScFtjeVItw/640?wx_fmt=png)

继续前进， 我认为使用最新版本的 Android 应用程序是更多的麻烦远比它的价值。混淆是巨大的，并变得极其乏味和沮丧，试图找到其名称已改为单个字母的函数。幸运的是，大约这一次，一位同事给了我一个链接，一些旧的 KeyWe android APK https://apkpure.com/keywe-for-a-smarter-life/com.guardtec.keywe/versions。事实证明，这是一个伟大的发现，因为现在我可以抢得一个版本，所有引用的功能仍然完好无损。我证明了这一点，返回到 F-Secure 根逃避脚本的原始（未修改）版本，并且成功运行！同时，我了解到，trace_java_functions''工具现在也工作，为我提供了大量的明确数据。  

在下面的 Frida 十六进制示例中，我们可以看到对 AES-128 密码函数（由应用程序生成）传递两个 byte_array 参数（AppNumber，CommonKey） 的调用。这显然是加密应用程序编号与通用键的 OTA 数据包传输到门，启动事件的会话序列。

同样，此示例还显示对 AES-128 密码函数进行的另一个调用，将参数（DoorNumber，CommonKey）传递到将 OTA 传输的门号加密到应用程序。

最后，在此示例中，Frida 允许我们查看传递的参数以及生成 AppKey 和 DoorKey 的两个内部函数调用的返回值。

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tsDQNTy5CI7hhCHfpXdj65sr4shzHaxeAgIW1a5K71aNReYgrOZz6cQ/640?wx_fmt=png)

基于捕获的许多 Frida 会话和数据破译，我很快就非常熟悉 KeyWe 应用程序，并且很好地了解了重要密钥的生成方式和位置，以及启动时的事件序列。此外，我了解到，在活动会话期间，门的状态不断通过应用程序和门之间交换的信息进行监控和更新。我的会话包括登录、解锁和使用手机上的应用程序锁定门。  

**F-Secure Frida java scripts：**

逃避根检测：disable_root_detection.js

跟踪注入的 java 函数：trace_java_functions.js（原始）

**会话包括：**

• 登录，等待连接和锁定（红色）状态

• 单击解锁，等待几秒钟，单击锁定，等待几秒钟

• 点击解锁，等待自动锁定，断开连接

C:\Users\rayfe\keywe-tooling\frida>start_root.py                                                           

C:\Users\rayfe\keywe-tooling\frida>keywe_inject.py trace_java_functions.js

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tXb98EAJXEbhUicBtP8m7akoHUZACwXwGMwsMXE25XhvmFfEAIQd8YmQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/JG2DPHdT7viaHUc0gY4hHsT0nDTj2Qn8tBjUm1g407pGuQiaAd8j6EFyvLrZw5dq3aqcFQnnCX7k2ogBHFjhHqxQ/640?wx_fmt=gif)

未完待续 ······

招新小广告

ChaMd5 Venom 招收大佬入圈

新成立组 IOT + 工控 + 样本分析 长期招新  

欢迎联系 admin@chamd5.org

  

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBR8nk7RR7HefBINILy4PClwoEMzGCJovye9KIsEjCKwxlqcSFsGJSv3OtYIjmKpXzVyfzlqSicWwxQ/640?wx_fmt=jpeg)