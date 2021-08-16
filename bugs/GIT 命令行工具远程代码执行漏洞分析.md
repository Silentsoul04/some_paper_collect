> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/972fmkQM1YpKhFCeAj-lwg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMsCKzOcSia4CUbkMJiaFtvDiaGwnXTHzO9exWmQKljFzibl9lV2N3l3zwv9g/640?wx_fmt=jpeg)

介绍
--

在这篇文章中，我们将会详细介绍漏洞 CVE-2020-26233。这个漏洞将影响 Windows 平台下 GitHub CLI 工具中 Git 凭证管理器核心 v2.0.280 及其之前所有版本的 GIT 命令行工具（也被称为 gh），而且一旦成功利用，攻击者将能够在供应链攻击中使用该漏洞，并攻击全球数百万的软件开发人员。

问题描述
----

在此之前，我们曾讨论过 GitHub 桌面端的远程代码执行问题，但这一次受影响的组件则是 Git 凭证管理器核心。

默认配置下，当 Git 克隆带有子模块的代码库时，它首先克隆代码库的顶层（根目录），然后递归地克隆子模块。但是在这样做时，它会从顶级目录中启动一个新的 Git 进程。

如果一个名为 git.exe 的恶意程序被存放在了代码库根目录下，那么当程序尝试读取配置信息时，Git 凭证管理器核心将调用此二进制文件。克隆过程正常进行，并且没有可见的迹象表明运行了恶意二进制文件而不是原始 git 可执行文件。

自从我们在 2020 年 11 月发布第一份报告以来，Github 创建了一个 SafeExec 库，以减轻 Windows 中二进制文件搜索顺序不一致带来的风险。

简要回顾一下，Windows 首先检查当前文件夹中是否存在给定的二进制文件，只有在找不到该二进制文件时，才会遍历 %PATH% 环境变量中的目录，直到找到目标可执行文件。

在 gh 的 v1.2.1 版本中，引入了一个 safeexec.LookPath 函数，当通过滥用 Windows 路径搜索顺序克隆新存储库时，可以阻止远程代码执行。

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMs0yBc4t6Lo2EKQDOxHhvmyWqGGicTHb5xewgA8TzSOeYq5r5poCMnuCQ/640?wx_fmt=jpeg)

在仔细研究之后，我们的安全工程师 Vitor Fernandes 发现了一个绕过方法，这样就可以利用它来实现远程代码执行了。

在漏洞发现过程中，我们发现在 fork 一个新的私有存储库时，仍然可能出现远程代码执行场景。因为在克隆命令执行之后，并不会通过 safeexec.LookPath 函数来调用 “git.exe config credential.namespace”。因此，所以 Windows 将返回到其默认值并搜索 git.exe 文件当前克隆存储库中的二进制文件：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMsttWKURbLMg6a8XV8TvPtnibhjECsg8ekMyicGgUvd8MHYysDxMiatBJxw/640?wx_fmt=jpeg)

下面给出的是 src/shared/Microsoft.Git.CredentialManager/CommandContext.cs 中的代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMsltibp7yGvQAcJH9txLEkRX0vJPPjhUGObdbsreEI3aibbSUR80rYV2ZA/640?wx_fmt=jpeg)

我们可以看到，在第 89 行代码处，将创建一个新的进程来搜索 git.exe，而 “Environment.LocateExecutable(‘git.exe’)” 将作为目录路径参数传递给 GitProcess()函数。

下图显示的是 Environment.LocateExecutable() 函数代码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMsAFSxicBu6xscCA8vZhtD9lXcyoYxrbFyf980OxGkubE00AnFdCWnJzg/640?wx_fmt=jpeg)

/src/shared/Microsoft.Git.CredentialManager/EnvironmentBase.cs

函数 environment.TryLocateExecutable 的代码可以在【阅读原文】找到：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMs6181qOIARnLnOtCKYa3FHWtib2bwRYpbQ0YmcQLlC3k3IZRtQLJBFqA/640?wx_fmt=jpeg)

在使用 Windows 的实用工具 where.exe 时，它将会返回所有出现的文件或命令，包括 %PATH% 和当前目录的值。

漏洞利用
----

下面给出的是针对该漏洞的漏洞利用步骤：

> 创建一个新的代码库，或向现有代码库中添加文件；
> 
> 向这个代码库中上传一个 Windows 可执行文件，然后将其重命名为 exe；
> 
> 等待目标用户 fork 这个代码库；
> 
> 然后成功拿到 Shell；

在下面的例子中，我们将 calc.exe 重命名为了 git.exe，并将其上传到目标代码库中：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMsylUOob9A4vwGZoIZiaomnUhqcpHIHDx8Sw5Buu8bV7YCyUy6naIiaYQA/640?wx_fmt=jpeg)

Fork 代码库并执行 “gh repo fork REPOSITORY_NAME —clone” 命令之后，目标设备将弹出计算器程序：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR398q2nLwgDYls15FI9JWrMsEibCYic2iaFDrPbozKgDFZuOSpM6GVxXycQjoy6xcSLiaP3FvKqfAAdKSg/640?wx_fmt=jpeg)

参考资料
----

> https://github.com/microsoft/Git-Credential-Manager-Core/security/advisories/GHSA-2gq7-ww4j-3m76
> 
> https://blog.blazeinfosec.com/attack-of-the-clones-github-desktop-remote-code-execution
> 
> https://superuser.com/questions/897644/how-does-windows-decide-which-executable-to-run
> 
> https://stackoverflow.com/questions/304319/is-there-an-equivalent-of-which-on-the-windows-command-line
> 
> https://nvd.nist.gov/vuln/detail/CVE-2020-26233﻿

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3icpSmNbdiaVpmTEfDHJFoS2OIO0ibau3Xo0W3W5icSIT9hIQY4gmlK4nOY8jcVq2hngIe7Fug8w6lHyQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484287&idx=1&sn=16a9b2dc0e205a0e5fe86ae5cae9fe2e&scene=21#wechat_redirect)[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39823fgk2Py1fbU5wCoewwO0AKFIGmCLF6bY37GDicGMDRicgQf6xW1jtjY8Raby8RjiauX5205Zg8Dg/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484370&idx=1&sn=8b79701a2936e04e390f165344e5fcdc&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3ibSZod64tZYfVs9eOO83Wq83nUmS51lkhNxf89EtGvGDD3Dlqria56Wl73fmg1kGk4WNKVN8AXCuEQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485424&idx=1&sn=1d4409309a035cb6ffcbdff54cc7ab7b&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR395Z4CT37PeziaibYaIGkflMsWMmHkcFLhU4zwO7V5TrLCjyZtIkKvYTIrL4WYG2cs1wZdH3uKaKRDQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485262&idx=1&sn=3038b6d54c1a38213d660ca7b0000562&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR38jJpuKrr8kx7KiazujuhoibR00ibHanwiaWL3iacIL65dliaJaPRwUwL2DvOo9NL4UWva3EwF35bcflS0A/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485242&idx=1&sn=b189e16baeec14f28f55c690598ef020&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************