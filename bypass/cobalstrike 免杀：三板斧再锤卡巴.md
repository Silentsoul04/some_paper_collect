> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_sGI4qNezca61AXJq06GHA)

**cobalstrike 免杀：三板斧再锤卡巴**

![](https://mmbiz.qpic.cn/mmbiz_gif/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsJGZiattL6rduiabNibbf9yIhGYtZ3TZqfwWWl8fgpdNclWCRkE0O4uIhQ/640?wx_fmt=gif)

1

概述

这次我们一起来探究如何规避卡巴的内存扫描，使得 cobalstrike 的 beacon 能够存活，主要原理就是：在 beacon 发送完心跳包 sleep，对自身的内存属性进行修改去除可执行属性和相关加密，其实现的方式有以下三种：

1、VEH Hook

2、SMC

3、InlineHook

下面会详细讲述各种方法。同时感谢 BGWill 师傅的文章，让我拓宽了见识。

相关代码已上传到项目中：

https://github.com/mai1zhi2/CobaltstrikeSource/

2

前置操作

在项目使用了自定义资源，并把 beacon 放在项目的资源中，然后通过解析自身的 pe 结构找到自身的资源，从而进行加载，beacon 放在资源可以自己异或加密，放在 bmp 图片后也是可以的：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsLzKLz1icbyhOYIdZudLx9AYQ2TwvwgkxFuc96CPCcmC365Rh0cQagqw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsHq3Gah03JRfyd55gUm5EBjVpqbLCtXW5omfCwq4xFtUyjSJsia8toYA/640?wx_fmt=png)

3

VEHHook

VEHHook 是基于 windows 异常的一种 Hook，所以这里要介绍下 windows 异常处理机制。在 windows 中，当程序执行过程中发生异常时，系统内核的异常处理过程（nt!KiDispatchException）便开始工作。当在没有内核调试器存在且异常程序没有调试的情况下，windows 系统就会把异常处理过程转交给用户层的异常处理过程（ntdll!RtlDispatchException），用户层会查找异常程序中是否安装了 VEH、SHE、TopLevelExceptionHandler 等异常处理过程，如果已安装则交给其处理。

所以，我们需要先在 beacon 端安装相应的异常处理过程，然后在发送心跳包时硬编码写下 int3 断点指令，当程序执行到 int3 就会触发异常，就会在 windows 用户态触发相应的处理过程，VEH->SHE->TopLevelExceptionHandler。在这里我们使用 int3+VEH Hook，因为 VEH 是全局的、基于进程、优先级高。

先找到发送心跳包处:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsHTx8wy4ibD46ywfrWmGuwY25vUuJnfGZhZxlqw7Gjt9yYRdSavmQ5Nw/640?wx_fmt=png)

然后在对应 pe 文件中找到相应的位置，硬编码上 int3 即 CC：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs9wI07wBicJgxUgwCToxl0l8ibBWkicHFYiaTSh1rtlicvuBUleM0XvQmFJw/640?wx_fmt=png)

然后安装 VectoredHandler 过程，当程序执行到断定时产生异常，该异常会被 VectoredHandler 所捕获：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsib3Pdfe8KVMjiaAlMctHiaicDyXPlicvTlZIrs98skOc7LqdUhhu5K4LvEw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsRZ82KF349cNgLWibRq9le9iaeXruOgSpiaxONBJAOKRzmEuqZCEwEbsjw/640?wx_fmt=png)

VectoredHandler() 如下，在函数中我们需要判断发生异常的地址是否与预期一致，里面的 context 结构体有详细的栈、寄存器信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsYUwczkBchtHCTicsKTHR3tF5t403NgfriatXyAhpYnicVFkFvdgEloXvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsiaVIfiaXW1QlCicgJAuia3BjP7icOeqjUjUibKibAr1bDib8XmA4ZIIqVdY9wA/640?wx_fmt=png)

Win10x64 执行效果：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsicNeqNrNojdnwd7Rs3fXtMuZzJsbfib8UianeYVdqoLarAmToF4COuFvQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsKGm6ibz41nghlGGjVNbQZ8ZiabDCelmp86dXeseOyxkJsxdvkFFUYFoQ/640?wx_fmt=png)

Win7x86 执行效果：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqssS1W8tWvP52yTDicckBLNiclGPvwoKxP49JVZD5CdRv4FkntS9Cn10VA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsoMN9mdsndpPvHKNWB3iaDUhemhtWicueZWt45xrBlmuCrsvzmtUHcH1A/640?wx_fmt=png)

执行任务时内存属性:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs3ibEKSgTAeffpqyyI5e3FPA9W9AN9JHiaXYtaiaibeCluLibmnk1pC9ZPNw/640?wx_fmt=png)

内存中的断点位置：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs3LQLicfoUriaaNicAibf94aX4R0U2DeMnQ2qDEuJhTxea9fGuO1rpQsM5Q/640?wx_fmt=png)

Sleep 时的内存属性：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsge3Xhd6Cz5tvmQDYUic0ficZibtgq9Sdv6ENKuVEibW0wLMIEROujj2pEw/640?wx_fmt=png)

4

SMC

SMC（Self-ModifyingCode），即能修改自身代码，对自己的代码进行打内存补丁。在 beacon 中，我们需要对心跳包的 sleep() 进行打补丁，将其修改我们自定义的 MySleep()。

这里需要注意的操作有，

1）因为 CS 的 beacon 是反射注入的，其反射注入的函数会对其进行重定位，所以我们需要对其 pe 的重定位数据进行修改，否则，在打内存补丁后，又被反射注入函数进行重定位，MySleep 的函数地址就会出错。这也是不能使用 IineHookCall IAT 的原因。

经过 pe 下数两行半等一系列数手指操作，找到需要修改的重定位数据：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsOQ4PXy2NFEn6PWYzAFufUJjs0EwmJk9RqgpKfeFmoeEdWWv5texQCg/640?wx_fmt=png)

2）自定义的 MySleep() 函数是 stdcall 的，不然栈会不平衡：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs7oPRozsTjibFNXasZLibe9U0ZFkSFUcNBOwicGsjhT3hyPaIe8qBY6I7g/640?wx_fmt=png)

在 Mysleep() 中，需要找到反射注入 dll 所申请的内存区域，所以要从栈上找到返回地址，再去进行相应的判断，应该也能使用 egghunter 在内存中捞出所在区域。

3）修改资源中心跳包的 sleep() 函数地址：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsv9I5aficJpvOYMKSb21MNuUZDXsA8RcB7yCo5btfkvIS1ibUsw84bMhA/640?wx_fmt=png)

Win10x64 执行效果：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs3SexFjd93lHlLAl6UMgIxhuOP0WKQIZPo035Dmic7paovIE1qXPiaqdA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqshicU8SDV2OGHYD2GUuvAwAKyRaPR4Hxj2c2myx1VGowNnHfeemoKy6w/640?wx_fmt=png)

Win7x86 执行效果：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsYiaHhEpDhbiaMxOBc0CJtZ0E3IUXwJzm6icEh09f7Ch4OJdsVTNykll4Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsquBvaweQDH0vm5hIiaPgjlTuJf8iblU2JMEQPib4XmsJz34emic2bwcrrA/640?wx_fmt=png)

Sleep 时的内存属性：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqstAdweT0XaUYSUAwBgGqiabzRSL02BnkdE8tBlQutqaZ27iaiaBxMwLzBg/640?wx_fmt=png)

5

InlineHook

Inline Hook 是直接修改指令的 Hook，使得程序转移了所执行的流程，转移的方式也各异，主要有 jmpxxxxxxxx、pushxxxxxxxx/retn、moveax,xxxxxxxx/jmp eax、callHookAddr(输入表地址)、HotPatchHook。使用 InlineHook 要注意几个问题：

1、是当 Hook 操作时的线程安全问题，可以先暂停所有线程，再进行 Hook 操作，最后恢复线程去避免，而 IATHook 相当于原子操作，不存在这问题。因为我们这里是先 Hooksleep()，再进行反射注入，所以也不存在这问题。

2、Detour 函数重入，造成无限递归

3、Detour 函数的线程安全问题，避免使用全局变量，若使用则要上锁。

这里我们稍改了下教主的框架，没有使用微软的 InlineHook 框架。

这里先定义代替 Sleep() 函数的自定义函数，也即是 Detour():

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs52sz0tKXKK6fz58OZSGZ8brReVMBhgLkzZhjTEDstgft1oIiba9195A/640?wx_fmt=png)

里面的 OriginalSleep 函数是一条跳转回去执行系统的 Sleep() 通道, 里面需要恢复原来的指令，及绕过自己安装的 Hook：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsocB6zbJ857JgscB9wZq5R68BJoZhTpsczoYrCdQgUMicJu1p19NAdFw/640?wx_fmt=png)

获取到需要 Hook 函数的地址并记录，通过 LoadLibrary()、GetProcAddress() 获得系统 Sleep() 函数的地址，这里有个地方需要注意，或许因为编译版本不同，后面可能是 FF25Jmp 或 E9 call 或者其他，这里 debug 编译的是 FF25Jmp，我们需要获取到 Jmp 后面地址所指向的值，该值才是真正系统 Sleep() 函数位置：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs5zAsTKmIlzcHZBJFX4DelBFDuc8THEwY4IkmlDA3LSibIHudur3aS1g/640?wx_fmt=png)

在系统 Sleep() 地址上，写入自己的跳转代码，一个 E9Jmp 跳到先前自定义好的 Sleep()：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs2lD0oD91vRzXYGKQftozLUvUaUTHbkibOB3PKC9CAYBJ2PtFxWmjIug/640?wx_fmt=png)

可以看到系统的 Sleep() 函数已经被 Hook 了：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsDIMMamJ4uKmPXiaaicn7eDLCCaN7ibwmc9VND2CKH686gEkaMR4Q0V7xw/640?wx_fmt=png)

一个 Jmp 跳转到我们自定义的 MySleep() 中：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsrYia5vjJLk5Gaqpg2lruy3nabBcRbJJd6eoDPlNLoxnwYktKLUATLrw/640?wx_fmt=png)

其中有个 E84E ED FF FF 这个 Call 是跳转回 OriginalSleep 函数，即跳转回去执行系统的 Sleep() 通道：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqszlicgYfqVfTaUeAVSexyxH0rolpj20BeXwmiawW1AMWsVk9v3D2KticJw/640?wx_fmt=png)

OriginalSleep 函数中一个 Jmp，绕过了 Hook，执行回系统的 Sleep() 函数：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqstI8rmr5STicL1b90nD2A73I5watCo2xWuVArLgTlS8vhicsnyQKdtmng/640?wx_fmt=png)

至此 InlineHook 完成。

Win10x64 执行效果：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqs4dh0uuqRxreEMMQicxXUCxVbexM6XKVkJnHXJV2N02icnkzMbECnwxdg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsjicfLgQtxKKhJQeFgjOjtLtGr4A2YwQpk7aGBxm0CGibdQ6Y2IMErVJw/640?wx_fmt=png)

6

小结

这次我们通过相关 Hook 的方式，达到修改内存中 Beacon 的属性和数据等目的，但操作起来还是有点繁琐。谢谢大家观看，下次我们再继续深究。

参考：

https://xz.aliyun.com/t/9399#toc-1

加密与解密

7

关注

本公众号不定期更新安全类文章和视频 欢迎前来关注

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACicst7e2PusNqmnq7NnRsqsW1TKBQANichZ9HxiaibgfIvCRlrqBSibiboLWhpUCicPKpe1oVdXaMro2qsg/640?wx_fmt=jpeg)