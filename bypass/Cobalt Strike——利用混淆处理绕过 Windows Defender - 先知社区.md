> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/2173)

原文：[http://www.offensiveops.io/tools/cobalt-strike-bypassing-windows-defender-with-obfuscation/](http://www.offensiveops.io/tools/cobalt-strike-bypassing-windows-defender-with-obfuscation/)

对于所有红队队员来说，在投递有效载荷的时候，总要面临绕过安全软件的检测的挑战。而且，就像其他安全解决方案一样，Windows Defender 在检测 Cobalt Strike 等工具生成的普通有效载荷方面，也变得越来越驾轻就熟了。

在本文中，我们将以 Cobalt Strike 生成的 PowerShell 载荷为例，讲解如何通过混淆处理，使其绕过 Windows 10 系统上的 Windows Defender。在绕过 Windows Defender 方面，虽然该方法算不上是最优雅或最简单的，但是在工作中我们一直都在使用这种方法，并且一直很有效。

创建有效载荷的过程如下所示：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318102242-3ca87c82-2a53-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318102242-3ca87c82-2a53-1.png)

这时，会生成一个包含 PowerShell 命令的 payload.txt 文件。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318102300-476b8d26-2a53-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318102300-476b8d26-2a53-1.png)

但是，如果直接将上述文件放到受害者机器上运行的话，就会被 Windows Defender 检测到安全威胁。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318102326-56eb62e4-2a53-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318102326-56eb62e4-2a53-1.png)

为了绕过 Windows Defender，首先需要了解一下 Cobalt Strike 生成有效载荷的机制，然后修改相应的签名，从而使 Windows Defender 误认为它是安全的。

首先，不难看出，这些有效载荷命令是 base64 编码的，我们通过格式就能看出来；此外，这也可以通过 PowerShell 的 - encodedcommand 选项检测出来。

为了对命令进行解码，我们需要通过下面的命令

```
powershell.exe -nop -w hidden -encodedcommand
```

删除一部分字符串，并保留其余部分。

然后，使用以下命令对剩余字符串进行解码。

echo 'base64 payload' | base64 -d

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318102644-ccbaa7c8-2a53-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318102644-ccbaa7c8-2a53-1.png)

上述命令解码后的字符串，实际上还是 base64 编码字符串，但尝试解码时，发现该方法无效了，因为得到的都是乱码——从 PowerShell 命令中的 IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s[IO.Compression.CompressionMode]::Decompress))).ReadToEnd() 部分来看，该字符串好像还经过了 Gzip 压缩处理 。

现在，我们需要了解这个命令的具体内容，因为这是实际触发 Windows Defender 的部分，即有效载荷部分。利用谷歌搜索，我发现一个正好可以用来解决这个问题的脚本，地址为： [http://chernodv.blogspot.com.cy/2014/12/powershell-compression-decompression.html](http://chernodv.blogspot.com.cy/2014/12/powershell-compression-decompression.html)

```
$data = [System.Convert]::FromBase64String('gzip base64')
$ms = New-Object System.IO.MemoryStream
$ms.Write($data, 0, $data.Length)
$ms.Seek(0,0) | Out-Null
$sr = New-Object System.IO.StreamReader(New-Object System.IO.Compression.GZipStream($ms, [System.IO.Compression.CompressionMode]::Decompress))
$sr.ReadToEnd() | set-clipboard
```

该脚本首先对 base64 字符串进行解码，然后进行解压，这样就能得到相应的代码。此外，它还会将输出的内容复制到剪贴板，以便将其粘贴到稍后用到的文本文件中。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318102754-f6e37714-2a53-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318102754-f6e37714-2a53-1.png)

$var_code 变量保存的就是 Windows Defender 正在检测的有效载荷，所以，为了绕过它，我们需要对这个变量做些手脚。

如果进一步对 $var_code 进行解码的话，会得到一串 ASCII 字符，但现在还无需进行彻底解码。

```
$enc=[System.Convert]::FromBase64String('encoded string')
```

现在，我们来读取部分内容，具体命令如下所示：

```
$readString=[System.Text.Encoding]::ASCII.GetString($enc)
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318102903-2013f762-2a54-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318102903-2013f762-2a54-1.png)

上图展示了与用户代理和攻击者 IP 有关的一些信息。

下面，我们要做的是读取相关有效载荷并进行混淆处理，以绕过 Windows Defender 的检测。进行这项工作的时候，最佳的工具恐怕就是 [Daniel Bohannon](https://twitter.com/danielhbohannon?lang=en "Daniel Bohannon") 提供的 Invoke-Obfuscation 了。该项目的 Github 页面可以在[这里](https://github.com/danielbohannon/Invoke-Obfuscation "这里")找到。

现在启动 Invoke-Obfuscation，具体命令如下所示：

```
Import-Module .\Invoke-Obfuscation.psd1
Invoke-Obfuscation
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103116-6ef44d8c-2a54-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103116-6ef44d8c-2a54-1.png)

现在，有效载荷的哪些部分需要进行混淆，我们必须规定好，具体可以通过以下命令完成

```
Set scriptblock 'final_base64payload'
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103317-b7828eec-2a54-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103317-b7828eec-2a54-1.png)

该工具将读取脚本代码，然后询问接下来的处理方式。 在本例中，我先选择了 COMPRESS，然后又选择 1。当然，这并不意味着其他选项将无法正常工作，但是至少在编写本文时，我的选择是能够绕过 Windows Defender 的。然后，Invoke-Obfuscation 会根据我们的选择进行相应的处理，并输出可以绕过 Windows Defender 的 PowerShell 命令。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103425-dfa64314-2a54-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103425-dfa64314-2a54-1.png)

然后，只需键入 Out 和 PowerShell 脚本的保存路径即可。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103444-eb301674-2a54-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103444-eb301674-2a54-1.png)

这样，就会得到经过压缩的有效载荷，具体如下所示。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103456-f2461044-2a54-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103456-f2461044-2a54-1.png)

实际上，我们最终的目的，就是用 Invoke-Obfuscation 新建的有效载荷替换 [Byte[]]$var_code = [System.Convert]::FromBase64String。为此，我定义了一个新的变量，我称之为 $evil，只是用于存放 Invoke-Obfuscation 的输出内容。

注意，对于 Invoke-Obfuscation 的输出结果，最后一个 | 之后部分是应该删除掉的，因为它是用于执行命令的命令。但是，我们无需担心这个问题，因为 Cobalt Strike 模板会替我们删除这些内容。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103511-fb008f52-2a54-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103511-fb008f52-2a54-1.png)

将编辑后的脚本保存到 PowerShell 文件中，并执行该文件。如果您使用的是 Cobalt Strike，将会看到一个 beacon；如果您使用的是 @sec_groundzero 的 [Aggressor Script](https://github.com/secgroundzero/CS-Aggressor-Scripts "Aggressor Script") 的候，会看到一个 Slack 通知。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103615-211132b4-2a55-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103615-211132b4-2a55-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103622-25c4abce-2a55-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103622-25c4abce-2a55-1.png)

如果我们用 Process Hacker 来观察 vanilla CS 有效载荷和修改后的 CS 有效载荷之间的变化的话，就会发现 beacon 的低层行为并没有发生任何变化。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20180318103704-3ed6dc04-2a55-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20180318103704-3ed6dc04-2a55-1.png)