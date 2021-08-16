> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aO4POHsTOUClRRXAvN-VXw)

**魔改 CobaltStrike：beacon 浅析 (下)**

![](https://mmbiz.qpic.cn/mmbiz_gif/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPynawVCT63huHO7Cs4DT5bYSa16rYdxAgJjwLkGAmdZiaaClqLgV1BLQ/640?wx_fmt=gif)

**1** **概述**

这次书接上回，[魔改 CobaltStrike：命由己造（上）](http://mp.weixin.qq.com/s?__biz=MzAwMjc0NTEzMw==&mid=2653572872&idx=1&sn=dbd20c14633aa9671b563e76d4483090&chksm=811b594ab66cd05ca7202aed2d8499bff1fa611c5beff51d270227fbda473f3e310d3998927e&scene=21#wechat_redirect)之前分析了 1-50 号功能，下面来分析 50-100 功能号，若有错还请大伙指出，谢谢大伙。

**2****50-100 功能号分析**

Rportfwd

任务号 50，端口转发数据

先获取到监听的端口号：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPMb8otarfGJG9PfX4mJG0fN5JNysEicDdGbZSQuict6yw7C83yIcdcfrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPXJBiarZ0YpQiaM8jh2EISHUNEG0ZbtcEKnz2hRHfbQicjmCmCKIo2Ud9g/640?wx_fmt=png)

然后 bindsockattr 0.0.0.0:8888 和开始 listen:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP0fwJdkB4nM21nakbAFg3VZnqGuYwgINpQTctmPDlVI3a5ow00QauqQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPKDqGib3OVEtGJBqxr4TtvaBWbMOVMjH3HXIe50MME2cQNJNhIt4Cibsw/640?wx_fmt=png)

任务号 51，rportfwdstop port 停止指定端口转发

有关于 rportfwd 所涉及到的任务号有 15、16、50、51，cs 和 msf 应该是根据 portfwd 开源项目 https://github.com/rssnsj/portfwd/ 进行整合的。

**Ls**

任务号 53，调用 FindFirstFileA()、FindNextFileA() 相关 Api 遍历当前文件夹，调用 FileTimeToSystemTime() 获取文件的文件修改时间大小等信息并通过 SystemTimeToTzSpecificLocalTime() 转换，最后进行相应的拼接返回给 teamserver 端：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPDHwUoUEce50xEhPohoMJ4v9XFrVHxMVoQpgIjLHXHEAbdtSneK3lsg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPugKErjicCEBHXSv5cyhkhibjxBdFOSUefG17obcFF0SHpobTj4SyqfHQ/640?wx_fmt=png)

把获取到的相关文件信息返回给 teamserver：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPicvBZttQicTXZicltfiaxufMFr9n5UNe8qEj6ER7J6dcEqM44YhxOP0lRQ/640?wx_fmt=png)

**Mkdir**

任务号 54，createDirectoryA() 创建目录

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPXvf1UaV4iabFBLpjQS66icWkcXKtvwJjeJTfz9Vb2f3ep9U6kdbWabHg/640?wx_fmt=png)

drives

任务号 55，GetLogicalDrivers() 列出目标机上所有的磁盘盘符

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPseTYYroJEFmzNDWxxoDmOYxKlV47kcibouBBqMcehHfhQVrJtB4MrVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPU3kgAkASRtBh7z6EXAlPZ3GVFmzFJwxR9EwDUZaW81WK6LfL372qcQ/640?wx_fmt=png)

Rm

任务号 56，删除文件

先用 GetFileAttributesA() 获取到文件属性，进行判断是文件还是文件夹：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPial6oiagicmpbc4KgFU2pz3jHXZIsf1SwkcxLZFEAiaicwTByh6G6TkcNHw/640?wx_fmt=png)

如果是文件夹就遍历删除里面文件再删文件夹：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPkFCK9KDWibkENVPwBF2icibIegxaQdKohVhhVQeap9YLvG2Skia5PhG5xg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPMzEDksQtgic2fbJFxVVVAu67Un1ktoPkhPiawShEFtYMK23RYbyhLMOQ/640?wx_fmt=png)

如果是文件则直接删：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPTes1FgsOj3nvLJKacoXFh8PuDIXm5tAYia6w64Bb9HWe5Z60gyibdIrw/640?wx_fmt=png)

One-liner

任务号 59，简单来说就是开启服务器，让执行相应的 ps 命令，返回一个会话。

绑定端口，新建线程进行监听：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPXs3qauoEeJCPQicQmNnmqKEldKaW9LDnaNZlhMxx58FYRkQT8Iib1vXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP2e6IkriaVd8ibE8apobNdoHpXTDMm86DyNtgettyTGdXp5E2TibmTGICA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPdfw5k0qicRldXwGbULlHiaicD0L8lBia1ibn9h34UEzypniblAL4VusNpNuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPEyVJfBiby0m2bjaaQKniaG3sy7xStgT4KnuBf0CeaWQswC08icHKbKzaA/640?wx_fmt=png)

执行 ps 脚本返回会话：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPG0rXV4PQW4DlL9yYGaZZD7tPPiaOFo5JnyRW3aqCFmJWBuGkkw5AacA/640?wx_fmt=png)

用管道执行 %COMSPEC% /c echo 配合 pthhash 发送给对应 dll

首先调用 CreateaNamePipeA() 创建管道：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPicIl4rbB8vSzsEDXocT4dziaUIlJW2OlfXebTWC1V1NibVzPedHWMGgPw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP6Nz5tdT8Xdm2SYVTUl3QZ0xmVibJnlxboGCFge1epKIB3ZibLY1SHLxg/640?wx_fmt=png)

创建新线程，等待另一端连接该管道：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPQqZnvMXKzg3OUTlOC3R9dtxflvYANgWERaTSpciczwFfJAlwlibiaUGmg/640?wx_fmt=png)

调用 connectNamePipe() 等待另一端来连接该管道，同时线程阻塞：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPmj5r6RrHfkFuqWta4XPdLiag3sHzcy7E4BBKNCFmQgE5vQbHIWnT5Rg/640?wx_fmt=png)

接着 aggressor 端会发送 mimikatz.dll 进行挂起注入并调用执行，过程与 spawn 一致，下面是 mimikatz.dll 内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPVQwoQLSOUJLCwdvSCfsBKV3qnrHcsuUfscDVLlYD6PwhUHNsbWiaK7w/640?wx_fmt=png)

mimikatz.dll 进行挂起注入并调用执行：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP640C7Yv8MJ1icDicDqVxKOu4DjngDqlZibgDlJdTRSRbqy7nibWBQzyfibg/640?wx_fmt=png)

恢复线程并执行：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPUZia7rAnGxnKnicTxApzzeDicTa94kaPme84OtZveGRnmhbMJ881BnpnQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPhGhLN9utz8icqTSagwictmLr0uJsGEq3Apm8klPZk0BOrdtCNM8Dyz4Q/640?wx_fmt=png)

在内存中 Dump 下来 mimikatz.dll，里面保存着两条管道名字，负责 mimikatz 与 beacon 通信用的：

\pipe\8f5879 是 beacon 生成的，mimikatz 调用 peekNamedPipe 去连接：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPicm98dsFFSQpdlszJ9GnXMT5lFfsQPNoibRNI3YIIVk4ZGFXuibyrNT5w/640?wx_fmt=png)

\pipe\2c67bfba 是 mimikatz 生成的，beacon 调用 peekNamedPipe 去连接：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPXYP6Cx3VDXY78xPplAF7HIXGb6E3A3KdobbLh5YMB9uEEpuohPhk9A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPktjZptL8bFBa0yTBNp8fLAX9tOD5tnxgbvID7ic8ys76018emkZnBqg/640?wx_fmt=png)

aggressor 会发送任务号 61 来建立进行与 mimikatz.dll 的通信，并进行后续的模拟高权限 token。

mimikatz 执行后 aggressor 回传通信管道信息，任务号 61：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPnbzW78vzp9zLeoU2NRGG2icQEPEzEMf6KdozL6W2ow6fTCAXsED3Oeg/640?wx_fmt=png)

调用 peekNamedPipe 链接 mimikatz 生成的管道 \pipe\2c67bfba：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPHm3BgflO92XV7OpOBHWx3x1Q7n1e0qTRoic7mvxa5y71YWof8y9POsA/640?wx_fmt=png)

一旦停止阻塞，就会调用 ImpersonateNamedPipeClient 来模拟高权限客户端的 token，并调用 openTreadToken() 获得当前线程 token：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPzicp1hfMbhvDJmCO3CEiaJelAw6NsrI2bfyiaOvN4pUHcqJ7j0AnkA9Gw/640?wx_fmt=png)

ImpersonateLoggedOnUser() 让当前线程模拟登陆用户进行操作，并保存 token 值为全局 tokenHandle：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPe4qPnobbvate0vbCyia2FhxUxiciaVMfYaBic1uVibrnMm2AxSe4Fx1TZkA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPjTib37YqKFocKJn7VAh9d2Pv0icJniaFpZicJianCZ9aDdeSYjnolYpqp2A/640?wx_fmt=png)

MimikatzJob 执行后数据的回传

任务号 62，流程与任务号 40 一样。

link 连接 SMBBeacon

任务号 68，link 通过管道连接 SMB 形式的 Beacon

Aggress 端在生成 link 方式的 stagless 时，会加载 windows/beacon_bind_pipe 中 pviot.dll 的数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPkjqK8VGVzskn4nul3C2P9mtWSgNzh7z7X70jALhhWkTDtJ7gBFoUZA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPHicJQVqWnFfEibDukr0cWgsOyoy6S1zOXiarhob26kGl8Yf2GHic79CPaw/640?wx_fmt=png)

在 link 之前，需要建立连接：

shellnet use \\10.10.10.165\c$ /user:"administrator" "!@#Q123"

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPx3RicGg62YLpURllnbhseYrR3qdty4A5MZv7W02qBDFRZXZrhcmcV2Q/640?wx_fmt=png)

再执行 jumppsexec64 10.10.10.165 Test_SMB：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPxVslBpw7fM7GRzegarxTxIzQNXjgxZm1HbPPTGyibcZTnKFldehJfOw/640?wx_fmt=png)

在执行 jump 命令过程中会调用到 link，下面看 jump 指令的执行过程分析：

先调用任务号 9upload 上传文件：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPdzsoNvlMzjsfM130J3pPP8luGbLnylicGq0icb9t9JRKYSt5gtriaqE5A/640?wx_fmt=png)

将文件保存在目标 ip 的 ADMIN\$ 下，名字问 f9febc5.exe

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPNzf7eYQE7eAeC7ciabKibqUORktorGPnlkbqicvsgwxPXYFIKwibvno9LA/640?wx_fmt=png)

然后通过任务号 100inline-execute 调用该文件。

再调用 55 任务号 rm 去删除上传的文件：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP25HCyqqB3RfE44GRZfaLaIrShkKXKYk6npibnkpLXxdmDnvib2HtUVvg/640?wx_fmt=png)

接着再调用 47 任务号 pause 暂停 1 秒：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPWtPmCC9H88A8bB4m2RSWAAYHCyKSH0iaVu4w8TqHl5ee8NPiaDRZspjw/640?wx_fmt=png)

最后调用任务号 68link Beacon:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPS6DYjAUxHlEhic94N7MWoJN5PTCFRbyKqezr6HMVroPYudcQQ0vJq5A/640?wx_fmt=png)

调用 createFIle() 打开命名管道：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP3jsPM2FAemK74Sj6HuxNnBtGoQTwxFwnwObuByq1xOPo1JnsMWBfSw/640?wx_fmt=png)

调用 SetNamedPipeHandleState() 切换管道为读模式：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPpMSicFkT53saiay4vHmaZUficAEO3PApNoTJ33uEQiaEkhL3tIfgA1RDVQ/640?wx_fmt=png)

循环调用 ReadFile() 读服务端返回的数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPR5ypQZVblwPh9sdDnhibUz4tMpwjccj1ibJIHnmibdUaRUKoiaI6Ysbabw/640?wx_fmt=png)

将数据返回给 teamserver 端上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP3ibFuYspqUDibu1H7aUPauvsolXcCD45VtzCZ4iau6K3Y2Wfpcc2aXQxA/640?wx_fmt=png)

Spawnto(x64)

任务号 69，spawntox64，设置 Beacon 派生会话时使用的程序，执行流程与任务号 13spawnto x86 一样。

execute-assembly(x86)

任务号 70，内存执行 C#的可执行文件，

首先传输 invokeassembly.dll 并注入到 rundll32.exe，实现了在其内存中创建 CLR 环境，然后通过管道再将 C#可执行文件读取到内存中, 最后执行。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPncUCrUlBiclthLvfIgCXomib5oUf4ptaT0LQpr4bicWQFlcJW7n9EjN5Q/640?wx_fmt=png)

Msf、cs 的 execute-assembly 与开源的 https://github.com/b4rtik/metasploit-execute-assembly/ 大同小异，我们大概了解下其执行流程：

当 execute-assembly 的 dll 注入到相应进程后

先后调用 ICLRMetaHost::EnumerateLoadedRuntimes、ICLRMetaHost::GetRuntime 方法以获取有效的 ICLRRuntimeInfo 指针：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPrrI03KwSQfUd9pxzx1RdsMSKSC3iabcZAL8H8vN9GIbZIeg3w4Feddg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPjcEYBJdyOFGxiakanW19XK68hGibuVClmupHibD2Mc08RpEPJ2Xpo9TQQ/640?wx_fmt=png)

调用 ICLRRuntimeInfo::GetInterface 方法获取接口并使用：

这里使用的是 ICorRuntimeHost：

· 需指定 CLSID_CorRuntimeHost 为 rclsid 参数

· 需指定 IID_ICorRuntimeHost 为 RIID 参数

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPCD5icib8Qr43IYzmJ6WcTXq3JcTOeguG4qxyI8hrqVbwaicL5531cjuXA/640?wx_fmt=png)

当获取到_AppDomainPtr 后，使用其 Load_3(...) 从内存中读取并加载.NET 程序集

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPEBVJsGLRDfnnHUicViadbJrwJ0HibsEia00SoC6zRANKyQRqQs6VicEEyiaQ/640?wx_fmt=png)

获取参数后调用静态方法。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPiczReWGeibBau1icKGjqRIVwW99Am9qUNRLWUWRpuf2lFU09eTibzJgxzA/640?wx_fmt=png)

execute-assembly(x64)

任务号 71，流程与任务号 70 一样，区别只是传输 dll 的版本是 x64 的。

Setenv

任务号 72，putenv() 设置环境变量

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPU0KQp02CnITfJ0a3adnib4L8sv6ZSTdcyEW5a8ic36YIOHh0er5H9OmQ/640?wx_fmt=png)

Cp

任务号 73，调用 CopyFileA() 复制文件

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPXONwSpWwAnyjVxFknVtkWTrg6Xqcu5IFbBhxfFib5icsMsk9NAmodpCg/640?wx_fmt=png)

Mv

任务号 74，调用 MoveFileA() 移动文件:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPia4ib5uaZHaWo9UgkuqQYOC949eeJEYvCbCUMvCCHQlCIlc0IFMR6EibA/640?wx_fmt=png)

ppid

任务号 75，伪造子进程（jobs）的父进程为指定进程，jobs 是指 portscan 等操作，不是派生会话操作。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPIWBkDrEZpaczmG3IAxeg2F6rJfjTrOO6KLQHJB12HycXmW6xG5hSmg/640?wx_fmt=png)

首先我们设置指定 jobs 的父进程为 18360“

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPVDzUv1BXPFUTZyag39K2YEOic0XHsQPKkMbFX2o3fPWcjrapiaCeX6rw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP3x1wrrnXT4gUqPVsn4wV0Y7iciaNITQib5gVKHmaChxd2p555o1Ymfvow/640?wx_fmt=png)

我们设置了注入 calc.exe，该进程 pid 为 35752：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPZGas6mKEAqNKn3D9ibWjDRiao1WoTu1qCjsOhBDUUfZW2pns5x09ygVw/640?wx_fmt=png)

查询 calc.exe 的父进程 pid 为我们设置好的父进程：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPznwg8ndgUic4vFw4MolS7oibpr2UQicziaicZ7gPaO3G5ADYIdGFNHqoVRg/640?wx_fmt=png)

runu

任务号 76，父进程欺骗，即指定所创建程序的父进程 pid 值，并执行该进程：

我们设置伪装的父进程 pid18360，先用 openprocess() 以 PROCESS_ALL_ACCESS 全部权限根据 pid 打开要伪装的父进程，获取其进程句柄：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPhSibYa4DIWXibGyCnmxNN49SXGxia7aqvyiaU2m9naGu51BiaoYGfHSgMXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPzNNewb3XGxvTO6rnnxpcLvBbyvTWqP6YuLibrEZQJr34IFiarq65Hk7Q/640?wx_fmt=png)

调用 UpdateProcThreadAttribute()，传入父进程的句柄，指定可继承的句柄：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPQ2Z5Picq2WvdqKDl5R9CRdiahz7iczY6ZhaednZhKMOiaZL76f7aAEVYKQ/640?wx_fmt=png)

接着用已构造的属性结构体更新属性表：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPB28Z9HtX5vQe6QrGqSZFG50srrL9xbD8oM98MG960blAlFNP4rTxAA/640?wx_fmt=png)

最后使用 createprocess() 传入已跟换父进程属性表 STARTUPINFOEX 来创建新程序：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPMCGicv6AAxbONxkpEZ8uylsiaKUJY0BEeSftYyseCQ8ibC7PZgJrGs9rQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPibkcqdknjlVdIvrnlFY6t29BibfN0Nh18MpHH7vlyseq7UDIic5n2dAaA/640?wx_fmt=png)

getprivs

任务号 77，getprivs，启用当前访问令牌所拥有的特权

命令窗口输入 getprivs 后，受控端会受到一系列的特权信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPkgX68UCiaml8lOL8abWM2svxMFu09qpia16SAy5SM2cKEueZN18ewBhA/640?wx_fmt=png)

当使用 hash 传递后，tokenhandle 会有相应的令牌信息，否则走 else 流程，流程中先使用 GetCurrentProcess() 获取当前进程的句柄，再使用 OpenProcessToken() 打开与进程相关的令牌：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPhbFarxIrlicGrutVM0D6UC93QtHczuBlbJibvgFVbicYvdnuiaaSeAM2qw/640?wx_fmt=png)

传入指定特权的名称调用 LookupPrivilegeValue() 函数查看系统权限的特权值，函数调用成功后，信息存入第三个类型为 LUID 的结构体中，并且函数返回非 0。接着调用 AdjustTokenPrivileges() 并传入新特权信息的指针 **PTOKEN_PRIVILEGES** 启用当前访问令牌所拥有的特权。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPqQMJVdrCVGNNk4tjw8arxnbRxv0KPpHxdSQ0MiazicZgapBNnxqKB7Ug/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPcIwzXa7ucDibR3ic0O8K1cibC7RfJ99OTsw4gYqeic7cPpwZ6n9LH14yPQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPkUyiaHcgdawhGUdu1ESbYxB7Shw9DsHhIa0j6iboWcYia5Nptb5PibG3zQ/640?wx_fmt=png)

Run/shell

任务号 78，在目标上执行程序（输出回显）

调用 createProcess() 或 CreateProcessAsUserA() 创建程序，当相关令牌 token 时 (该 token 是在 pth 或 AdjustTokenPrivileges() 提权后获得的)，会调用 CreateProcessAsUserA()，否则调用 createProcess()，

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPdtOrMzyMRolKrAfLKVw22ibtj10yzjiaD2FAPibzsK45v2b7KSoEiajTXg/640?wx_fmt=png)

shell 会在启动程序的命令前加上 cmd.exe：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPoT3B5j1OmtcJNZvTb4q1wjwE5ea5fzAggqfAMbibOPygsKIlF7o2vYA/640?wx_fmt=png)

执行完返回的数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPUQvXCWCRBFgic9vLlgKgbjIEfgaDdBIDVjdFvxr7sLDupxYq3M94ebg/640?wx_fmt=png)

没有 token 时调用 createProcess():

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPz7JPH0Crbiaic0NGQYFdJCt82taYkI6nZWgZXnOMPicPJCxS2025iaFTlQ/640?wx_fmt=png)

BeaconTCP PIVOT 将受感染系统用作内网中其他 Beacon 会话的中转器。

任务号 82，当在 Pivoting->Listener（ReserveTCP Beacon）或 oneliner，创建完成后再执行一条命令 rportfwd4444 windows/beacon_reverse_tcp，会调用此任务号，原理与 rportfwd 一致：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPYHRMUKW5UQnKYueGEluUDZ23J5vl7aNCmt4PXAV66PU0w8pVvIMyvg/640?wx_fmt=png)

调用此任务后，会监听相应的端口：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPKrkUcuoBfwsT3dlQmUUNVoocxE0ohgiajyRgs5etwGf8yfPibkm7PGMQ/640?wx_fmt=png)

生成 stagless 方式的后门时，能选择相应的 listen:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPphztV3KezJh7Tx4eFCeB6ZALRcDUrfnoGmpmibyOxImVA5rcNUm9s1Q/640?wx_fmt=png)

选择了 windows/beacon_reverse_tcp 后，agressor 端会读取 pviot.dll:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPGDCG06q73WeKSkneLCHGIvTc8CbezJ4gd2rAMRG3n7GanR2O4DZxmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPS5JKkyhDLyhawNtUoicXT9XGaR9LxrsbkHdhicnZdOcP0XH7icQjCMqjg/640?wx_fmt=png)

读取到的 pviot.dll 与 beacon.dll 是有一定区别的。

执行上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPPCWvQl3H0EL0zMmHa4bv9G9WMqNibWtmiaKVkxnmYK0ol2KeZ17WQ8YQ/640?wx_fmt=png)

Argue

进程参数欺骗，

任务号 83 是增加欺骗的参数，

任务号 84 是删除欺骗的参数，

任务号 85 是查询设置了哪些参数，

我们主要看增加的操作：

先调用 ExpandEnvironmentString() 扩展环境变量字符串并以定义值替换：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPBicj9XWTCicY9RnCzSXKU1ib9PaNrUO9Y28gG5ORlPruTib3iccso7WLvVg/640?wx_fmt=png)

当启动程序进行挂起，然后会替换对应程序的命令：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPiagg4bLhqaLEWZrtZ328I9Ix8mjrWyIibHFXibOrT17pnEwY2xsWebyog/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPuCfjucB5G0c9DOtcwM6jwibQnmdMKt9me3AOiciavUHhKfTxcKdCIwO4g/640?wx_fmt=png)

然后修改该程序进程块的信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPn1ibFDK5Tpl1jbibOsFZuAcxF6fBDUf634Z7YdsbWbntUUnxgr0pbTng/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPYibLsicwNw5DbI7YXW2oNtlgF3ibdXibfhuaXn3IAlw7fqrdyjcibCyNxiaA/640?wx_fmt=png)

Connect

任务号 86，Connect 连接 bind 形式的 TCPBeacon

Aggress 端在生成 bind 方式的 stagless 时，会加载 windows/beacon_bind_tcp 中 pviot.dll 的数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPQLXNciaRu2hG5udQE5fRvZLwLniahB5MWP5SDia4ldlSxZ0hOsUexeNPw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPfQ5ZJ6Yjz8ib2svcvwfUg0chfJXmhtXF6mASIJEC0peUflyUlicgrXLQ/640?wx_fmt=png)

当受到端受到 connect 指令后，会调用 connect 函数去连接相应地址及端口：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPISmPuOjoJYGAGAaIcFD02icTKjF17FKvIdlT6Wn9qGpnevsIQk0Hic5A/640?wx_fmt=png)

Bind 设置监听的端口：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPFhcib45poKtNo0Fh5vibK8O3VTnNsJDopIoichZtsW9YbYL7YFLXuSDiag/640?wx_fmt=png)

连接目标 ip 的 bind 端口：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPj8qChOKibdK32514zezng7jVduFtgXTwec3YrS1RKDCBVeTxut0rM2Q/640?wx_fmt=png)

执行上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPYzZFxO4yX8QalvWok6S9RJLjibiaGp27QxBNwiaicIWLHkialiavLYFP8Hsw/640?wx_fmt=png)

execute-assembly(x86)

任务号 87

流程与任务号 70 一样，作用也是内存执行 C#的可执行文件，流程一样：首先传输 invokeassembly.dll 并注入到 rundll32.exe，实现了在其内存中创建 CLR 环境，然后通过管道再将 C#可执行文件读取到内存中，最后调用执行。区别是，不使用 ImpersonateLoggedOnUser() 当前线程模拟登陆用户进行操作，TokenHandle 要在 pth 后才会生成并保存：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPYJg8C2IEH7kxAtyoIxVCkS9Lx1MN6pUbWNic0ZPgXh0apAhU6Vic0t1w/640?wx_fmt=png)

execute-assembly(x64)

任务号 88

流程与任务号 71 一样，区别是不使用 ImpersonateLoggedOnUser() 当前线程模拟登陆用户进行操作，TokenHandle 要在 pth 后才会生成并保存:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPYJg8C2IEH7kxAtyoIxVCkS9Lx1MN6pUbWNic0ZPgXh0apAhU6Vic0t1w/640?wx_fmt=png)

Portscan 等传输反射 dll（x86）

任务号 89，传输并执行 portscan、hashdump 等反射 dll，即 job，原理也是挂起线程 rundll32 线程注入 dll，流程也是一样的，区别是不使用 ImpersonateLoggedOnUser() 当前线程模拟登陆用户进行操作，TokenHandle 要在 pth 后才会生成并保存:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP2c8ITDXJAibbjJUw11T9l279StiaiaCe1R8n1ia8lcNgCPAYicTm8R98z2Q/640?wx_fmt=png)

当相应的 job 执行完后，会通过管道把数据返回 beacon，触发 40、62 任务号，后面数据传输详见 40 任务号。

Portscan 等传输反射 dll（x64）

任务号 90，同 89 任务号流程一样的，只是在 Syswow64 的在 rundll32.exe：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPNqZZnJ9nKcF7CHVF5JwibNmw8XVadicW2wS6xY9A3FvtLhOsF49I4SHg/640?wx_fmt=png)

desktopVNC

任务号 91，x64desktop VNC 远程桌面（不注入进程），需要由 vnc 的 dll（我们没有这个 dll）

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP5RkMUKg7y46rYLEkW2KumdBB5bCaadLice1W31FgeIQsHtvPZ6AopSQ/640?wx_fmt=png)

Blockdlls

任务号 92，设置相关策略使创建的子进程加载非微软签名的 dll 时会被阻止。（win10 才生效）

Spawnas(x86)

任务号 93，以其他用户身份派生会话，具体流程与任务号 1spawn (x86) 一致，区别在注入时启动进程用了 CreateProcessWithLogonW() 可以指定其他用户身份：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP7pFUaf22CrWfoibFpcEBTVmNj5IQNuUZZQbFoUialX1G2HxXn1hibWKGg/640?wx_fmt=png)

新进程在指定凭据（用户，域和密码）的安全上下文中运行指定的可执行文件:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPv5icqRSEne85o1luCiawCgwbJEdibv1ZbO5r0Q9E2XDicqlCTBHgrw6edw/640?wx_fmt=png)

Spawnas(x64)

任务号 94，以其他用户身份派生会话，具体流程与任务号 93spawn(x86) 一致，也是在注入时启动进程用了 CreateProcessWithLogonW() 可以指定其他用户身份，区别只是启动程序的位数不同。

Spawnu(x86)

任务号 98，spawnu 指令是在指定派生会话进程的父进程，作用与 runu 指令类似，容易与 spawnto 混淆了。总体流程时先用 openprocess 传入要设置为父进程 pid 值获得相应的进程句柄，然后再用 UpdateProcThreadAttribute() 更新属性表：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPxUcWMhYqbibB7PrJNwjfp9andFN2QU5fAHQdq2em9u6YdBxTqL8yNbg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPdnrotzmYv1CYSnWRj8SHEzJUwuPic7vhTzt5yEr4Hy0Z0f89N7eOQgw/640?wx_fmt=png)

先用 openprocess() 以 PROCESS_ALL_ACCESS 全部权限根据 pid 打开要伪装的父进程，获取其进程句柄：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPGYF1MLXeJxSHdhQBbzNibOX6v5aqvBLSMJycYxVb5O6AuicJ2JycKjyg/640?wx_fmt=png)

调用 UpdateProcThreadAttribute()，传入父进程的句柄，指定可继承的句柄：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPGdhg3KTO1VtMx861mWPWgFBFwMJvKYhaPqvnwzqSlj1jED40dNdzhg/640?wx_fmt=png)

调用 DuplicateHandle() 复制句柄的目的主要是对句柄权限赋值和修改:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPAx71Px38UTWv92WF1ftyCw4ic2lnVF2Q34BBzOzwVMcyuEl2x3EgMMQ/640?wx_fmt=png)

最后调用 createProcess() 启动 rundll32.exe 后续进行相应的注入：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPr24GsAJcd42vH0q2lGHYusleGia9PQYKUgE9ickRmpgt96ibn1dEk3Sdg/640?wx_fmt=png)

派生后的会话进程信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPicYR9EzyuITQJothjYG5XicLqMiapuKGsD5AnMmoPbxL6xaX8lOStQp1Q/640?wx_fmt=png)

派生后的会话进程的父进程信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIP52CG8YXKia2nfTw8NFQqR0KC3zLke0XS5GicL3AfNicCDbZv3LcSEIefg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPiau53hL36GakxPOaOw7LUmFKcjxszhAVfQwGtSwWkUzDcicoRKEorHWg/640?wx_fmt=png)

可见新创建的 rundll32.exe 的父进程指向了所指定的进程。

Spawnu(x64)

任务号 99，流程与任务号 98 一样，只是传输的 x64 的反射 dll。

inline-execute

任务号 100, 在 Beacon 会话中执行 BeaconObject File (BOF), obj File 也就是编译后但未链接的目标文件,CobaltStrike 会先对这个 obj 文件进行一些处理，比如解析 obj 文件中一些需要的段.text，.data，在处理一些表比如 IMAGE_RELOCATION，IMAGE_SYMBOL 等等，然后在经过一系列的处理后，会把需要的部分按照一定格式打包起来随后在发送给 Beacon，这时 Beacon 接收到的是 CobaltStrike 已经解析处理过的 obj 文件数据，并非是原本的 obj 文件，所以 Beacon 主要做的是必须是在进程内才能确定并完成的事情比如处理重定位，填充函数指针等等，最后去执行 go 入口点。

关于 inline-execute 执行 BOF 可参考 wbglil 的文章 https://wbglil.gitbook.io/cobalt-strike/cobalt-strike-yuan-li-jie-shao/untitled-3。

**3** **小结**

这次我们分析了 beacon 约 100 个功能号，下次我们再一起来重写 beacon。若有错误还请师傅指出，最后谢谢大家观看。

**4** **关注公众号**

本公众号更新安全类文章 欢迎长期关注  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6AC7aeO7Ek6Jh6SdHfg96UIPdrGe5FVYf3KEFD7vqyHSBeicFliauickz8Oojv5ibrrp48xeXxKjLyIvxA/640?wx_fmt=jpeg)