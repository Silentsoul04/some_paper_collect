> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IutPFguZmNvPZd5XrlK6Mg)

        [逆向智能门锁漏洞分析 PART（一）](http://mp.weixin.qq.com/s?__biz=MzIzMTc1MjExOQ==&mid=2247492107&idx=1&sn=f1add5d8c1868a516cd6b64c0cb5d436&chksm=e89dcad3dfea43c5741e3bcbc14163270b4bc85d0edd40b9219e045332b2a0c20607352f9df0&scene=21#wechat_redirect)后续篇

**BTSNOOP （安卓蓝牙 HCI 记录器）**

最终，BTSNOOP 必须成为我最大的发现之一，当想要捕捉一个完整的蓝牙会话之间的中央（手机）和外设（锁），我尝试各种方法来捕获我的 OTA 蓝牙会话，包括北欧的 nRF 嗅探器开发板 nRF52840-DK，Sena 的 UD100 加密狗，Ubertooth-One 和德州仪器 CC2540 加密狗。所有这些方法的问题在于，由于蓝牙低功耗 （BLE）通道跳跃，它们无法跟踪连接。北欧 nRF52840-DK 在与 Wireshark 和北欧的 BLE 嗅探器插件一起使用时非常接近，但不幸的是，数据包捕获是在链路层（而不是主机控制器接口层），导致无法解析的加密数据。  

由于它不是在 HCI 层捕获的，这意味着它容易受到 CCM AES-128 BLE 安全密钥交换协议握手的影响。根据我的判断，这意味着如果 nRF52840-DK 在配对时没有嗅探，它将完全错过安全握手，导致数据包没有解密。

**重要提示：NRF 嗅探器缺点！**

如果 KeyWe 锁在配对后执行通道跃点，但在 App 传输第 1 个数据包之前，nRF 嗅探器将错过初始 CCM AES-128 BLE 安全密钥交换。这将导致加密的 "无用的" 数据包。这是 nRF 嗅探器无法遵循通道图的一个缺陷。注意：CCM AES-128 BLE 安全密钥交换是所有蓝牙低功耗 OTA 无线连接中的安全协议，可防止 MiM 窃听。

**CCM AES-128 BLE 安全密钥交换：**

（这是与 KeyWe 项目无关的一般信息）

临时密钥在蓝牙配对过程中使用。短期密钥用作首次设备对加密连接的密钥。短期密钥使用三条信息生成：临时密钥和两个随机数，一个由从站生成，另一个由主项生成。

使用短期密钥加密连接后，其他密钥将分发。长期密钥替换加密连接的短期密钥。身份解析密钥用于隐私。连接签名密钥用于身份验证。

**幸运的是，有一个极好的方法来捕捉蓝牙流量使用您的 ANDROID 设备！**

在 Android 手机上

• 转到设置

• 如果未启用开发人员选项，请现在启用

• 转到开发人员选项

• 启用选项 启用蓝牙 HCI 窥探日志

• 执行需要捕获的操作（会话）

• 禁用选项 启用蓝牙 HCI 窥探日志

• 使用亚行（安卓调试桥）将文件复制到 PC

• 感兴趣的文件 btsnoop_hci.log

**注意：**通常，我会保留启用蓝牙 HCI 窥探日志的选项，因为它在我的根测试手机上

获取 btsnoop_hci.log 蓝牙会话的完整信息

**列出的 android 文件**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRbW00TK0ibYWYEAaibWp9jGZic8icm5xJDaL0Xw77Ir3zsoflISoLG7oxCw/640?wx_fmt=png)

**调出的 btsnoop_hci 日志文件**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRgibtq30lteYiaibmLTsn6ibAlrtgTQg0jugqD6ibIQVdz5tVfJ0dhlVN3Bg/640?wx_fmt=png)

**重命名 btsnoop_hci.log btsnoop_hci-07-31-20.log（与我的会话日期一起附加）**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRgJGC5t4AuE6eNbL9wDliabMaYBWrHYr8nswy4FnlF5IGLKAq7HmQ1YA/640?wx_fmt=png)

**WIRESHARK 分析**

将 btsnoop_hci.log 导入 Wireshark，我们可以看到 OTA 加密数据包交换。这与 Frida 函数十六进制提供了交叉引用用户会话活动的一种有价值的方法。从我在 KeyWe 锁研究期间生成的大量会话中，我可以确认这些数据包交换在每个会话中遵循相同的顺序，并且永远不会变化。在下面的示例中，我们可以看到由应用程序启动的打开密钥交换，然后是门。

**示例 1：应用程序发送应用程序编号 + 门返回门号 + 门发送 Hello**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRuaRJ8iaII0NwVZd4Gkc6AZpywyy1sEuibQAoFGyWAMhSjMe9kHlUIH7g/640?wx_fmt=png)

有趣的注意：（在上面的截图）：fb2b28c68b3f99c514b98fada4bf0b89 （传输 #2） 可以解密与通用键复制门号！

这可以通过使用免费的在线 AES-128 密码工具在这里验证：http://aes.online-domain-tools.com/

输入加密数据包 fb2b28c68b3f99c514b98fada4bf0b89 并输入密钥（通用键） c88ff4150f4ac27934a6c5e6741efac 后单击解密

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRAIv9YaD0dyfNujYgWIaPxKibb3KXRic8BsGI1RAzurWX2tY6XRYKq5Rg/640?wx_fmt=png)

使用在线 AES-128 密码工具是将 Wireshark 会话数据与 Frida 十六进制数据关联到的方便方法。接下来的几个屏幕截图显示了各种蓝牙 OTA 流量如何与应用程序的已知函数调用重合的示例。这为我们提供了大量有关程序流和执行的信息。

**示例 2：应用程序发送欢迎 + 门发送开始 + 应用程序和门两个交换门模式**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRfpMiaYgtOntYOS8MpobTtsrZU5tlicluh9s1Z8RgbRzHheWDwArXZaIQ/640?wx_fmt=png)

**示例 3：APP 发送 eKey = 门发送 eKey（身份验证和授权）**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRqZicxKj0ggXia3Y5OGo80SgIkA42gowmtEJZk9gyficEGRgLPgLQUbscw/640?wx_fmt=png)

**示例 4：门状态 = 门时间集交换**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRD2A6UyNcovqUDGVDibqPBjoLJTtBDug17ouwkEH50jCdrLfcHZ1VCKA/640?wx_fmt=png)

**示例 5：门状态交换**

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRiacDBjXibQWhhz3y7ibbSlTQqPHH1x1IIfvUc4qVwgAaf83icokz8yrroA/640?wx_fmt=png)

**重放攻击**

如屏幕截图所示（上图），可以使用 btsnoop_hci.log（捕获）在 Wireshark 中分析整个蓝牙会话，并与使用 Frida 工具找到的数据并排进行比较。另请注意，通过无线发送的每个加密数据包都可以通过确定传输方向（应用程序到门或门到应用程序）并使用适当的密钥（AppKey 或门键）进行解密。

现在，我们已经有了密钥，知道如何解释所有数据，我们可以尝试使用重放攻击来操作锁。同样，F-Secure 在他们的 Github 中提供了一个不错的工具，他们称之为 "open_from_pcap"。根据预先录制的 pcap 会话中的信息，此工具允许他们重放会话并操作锁。当然，当他们在脚本中重新编辑其脚本时，keys.py 无害。然而，正如我前面所说，我最终能够对功能进行逆向工程。因此，通过将 F-Secure 的 REDACTED keys.py 我自己的版本交换，它允许我在 open_from_pcap 上实现 "btsnoop_hci.log" 工具。

使用我的 Sena UD100 蓝牙 USB 适配器，运行 “open_from_pcap” 脚本的第一个结果如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRUVM4xRZ6np3iaOSWeMDIky9xLTPHTzve9AZEvnfY5nPGkjFARopg6TQ/640?wx_fmt=png)

显然，我修改 keys.py 正确确定了公钥、AppKey 和 DoorKey，但在 eKeyVerify 阶段失败。

在不深入地了解 F-Secure 的编码的情况下，open_from_pcap python 脚本在另一个脚本 （decode_from_pcap）中调用一个函数，该脚本据称会从会话 pcap 中检索 eKey。不幸的是，这对我来说并不适用。也许这与他们的 pcap 文件与我的 btsnoop_hci.log 文件的格式结构不同（从 Wireshark 会话中保存为 pcap）有关，不管怎样，我决定放弃破译他们的代码，而是修改了‘open_from_pcap’文件，使用我的硬编码 eKey，而不是尝试检索它。

顺便说一下，F-Secure 人员需要从 pcap 检索密钥的原因是，当创建帐户时，eKey 会在线存储，并且不存在于任何 OTA 传输中。但是，通过使用 Frida，它可以通过将 javascript 注入 eKeyVerify 函数来检测，该函数提供返回值的十六进制（见下文）。

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQREyl5LyRYMUl9bWVUHXibQp5tgiac2NvEEkbGfxe1pibbwJFXgHDLxZG4A/640?wx_fmt=png)

正如您从上面的屏幕截图中看到的，作为测试的一部分，我决定删除我的（旧）KeyWe 帐户并创建一个（新）帐户。通过这样做，我也许能够确定从一个用户帐户到另一个用户帐户发生哪些更改，以及 eKey 是否可预测。从我所看到的，没有明显的可检测模式。eKey（也称为用户密码）是在所有者创建其帐户时生成的，因此在帐户设置期间，密钥是完全随机的，并且可能从用户密码派生。此外，当我创建新帐户时，我故意只更改了原始密码中的一个字符。此处意在说明，这可能有助于我确定 eKey 是否仅派生自用户密码。

从上面的屏幕截图中，似乎很清楚的是，eKeyVerify 函数是由应用程序调用的，并且传递的参数是一个 6 字节的值 （eKey）。返回的值可以假定为由应用程序（使用 AppKey 加密）发送到门的修改结果 eKey。在不深入挖掘代码的情况下，我只能假设 6 字节值 （eKey） 提供了指向存储的实际（修改的） eKey 的链接。无论如何，这个 6 字节的 eKey 是完成重播会话所需的全部。

将 eKey 硬编码到 replay.py 脚本中，导致以下重播会话：

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRztVzicJ6u8wS2PytkfgsA8LLKPHV5ToWgNYBxeabl3VibX4sLiaboicchQ/640?wx_fmt=png)

**成功！！**

此重放是成功的，因为我能够获得 eKey（用于身份验证和授权）从我扎根的手机使用 Frida 提取它。就目前情况，由于合法所有者的 eKey 无法以这种方式访问，因此此重放攻击无法在野外工作。同样，正如我前面所说，eKey 不是传送 OTA 的。

考虑以下代码段捕获我的手机的 btsboop_hci.log 和相应的 Frida 十六进制的 eKeyVerify 函数：

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRwk4OLaj8Ql4Gdg2F0aq9RorO78rX1EJZeSL3iafwMDNAqgcu0TYwotw/640?wx_fmt=png)

Wireshark 捕获显示打开数据包传输（密钥交换、握手等）。密切关注此会话中的第 8 个数据包。它显示应用程序发送的加密数据包，其中包含修改后的 eKey 值。从 Frida 十六进制，另请注意，6 字节 eKey 值被枚举为修改后的 eKey 值的字节 （5：11）。

根据我们学到的关于此数据包在传输前如何加密的知识，我们知道这将是一个使用 AppKey 作为密钥的 AES-128 密码。

使用在线 AES-128 密码工具解密第 8 个 OTA 加密数据包：46402315a85a72e66e9671d0444b513af 使用 AppKey：e022c1193ebb3882efc9cf79b6e557d1 作为密钥， 正如我们所了解的，6 字节 eKey 值被枚举为解密修改的 eKey 值的字节 （5：11）。

（见下文）

![](https://mmbiz.qpic.cn/mmbiz_png/JG2DPHdT7vhUoMRSNgSFqkD6raSG5mQRm4gzbXS4KPDLzVG0vF4hC7B3WOZQVsaLericEZOkbYjoOkicu3Vibcpzg/640?wx_fmt=png)

这个小练习清楚地表明，如果我们可以在野外合法所有者的打开数据包交换的 OTA 捕获，我们将有我们需要的一切（包括他们的用户密码 + eKey）来危害他们的家庭安全，并解锁他们的门！

**总结**

我不得不承认，我花了大量的时间研究（按几个月的顺序）这个项目，主要是因为处理沉重的混淆的代码，但也由于学习曲线涉及学习任何新的工具。熟悉 Frida 工具及其许多功能和实现中有几个对我来说是这个项目的要点之一。此外，能够对 Android 应用程序进行逆向工程同样有益，同时意识到 F-Secure 通报发布以来已经一年一段时间了，供应商已经有足够的机会来减轻他们的影响。因此，我强烈同意 F-Secure 研究人员的发现，因为固件更新功能肯定会减轻他们的大量暴露，并且使用自定义（内部）加密算法一开始就不是非常好。

结尾之际，我还要处理一些未完成的研究，这是 OTA 捕获我的非根个人电话和 KeyWe 锁之间的会话，以创建一个重放攻击，将在野外工作。无论如何，为了减轻他们的曝光率，我强烈建议 Guardtec KeyWe 智能锁的当前所有者尽快将移动应用程序升级到最新版本（本文撰写时的版本 2.1.0）。

招新小广告

ChaMd5 Venom 招收大佬入圈

新成立组 IOT + 工控 + 样本分析 长期招新  

欢迎联系 admin@chamd5.org

  

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBR8nk7RR7HefBINILy4PClwoEMzGCJovye9KIsEjCKwxlqcSFsGJSv3OtYIjmKpXzVyfzlqSicWwxQ/640?wx_fmt=jpeg)