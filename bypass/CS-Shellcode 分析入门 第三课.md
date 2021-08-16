> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pcDzINtxJOgKnHUlyvlvZQ)

**点击蓝字**

![](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif)

**关注我们**

  

**_声明  
_**

本文作者：Gality

本文字数：3100

阅读时长：20~30 分钟

附件 / 链接：点击查看原文下载

**本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

![](http://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDjy8pCtpvJKBibCLXQDm14MbdlTqXYESXADHkVpL6f81Z4TVFOGQMjBjgxPpUcYnzahRhibQUdcKzQ/0?wx_fmt=png)WgpSec 狼组安全团队推荐搜索安全工具安全研究漏洞复现

  

**_前言_**

  

        本文是 CS 的 shellcode 分析的第三篇文章，该系列文章旨在帮助具有一定二进制基础的朋友看懂 cs 的 shellcode 的生成方式，进而可以达到对 shellcode 进行二进制层面的改变与混淆，用于免杀相关的研究。

**实现的一个免杀加载器 https://github.com/wgpsec/CS-Avoid-killing**

[CS-Shellcode 分析系列 第一课](http://mp.weixin.qq.com/s?__biz=MzIyMjkzMzY4Ng==&mid=2247487086&idx=1&sn=281e468c8ba38d53a50bf9eae24772fe&chksm=e824a9b7df5320a1c105dd5fe484ff44f62a1421a1e1686a90cb5d7604bbb6532f3b45006457&scene=21#wechat_redirect)  

[CS-Shellcode 分析入门 第二课](http://mp.weixin.qq.com/s?__biz=MzIyMjkzMzY4Ng==&mid=2247487765&idx=1&sn=4aa17ec86305ebc3832becc1ed057144&chksm=e824b6ccdf533fdaaecb93f7e9a7b7f0992c745918f07670ebf1e33cb1fcadd7746d5bdeb8a0&scene=21#wechat_redirect)  

一、

**_Shellcode 分析_**

同样是接上文，上文提到`726774C`是函数`LoadLibraryA`的特征值，这一点我们也可以在动态调试中验证

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JTicc3JoicaWGLxV4U1R2sIcKuUaGAlVX6Cr7hfveZhYH5rFruo1q2X0Q/640?wx_fmt=png)

可以看到 rsi 指向的就是`LoadLibraryA`这个函数名，此时我们已经有了函数地址，可以进行函数调用了，我们看接下来的操作

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JtDlF0TZicyJ2g5kncDW8GCVZddvMLyNdCucxFiaMB2d1ibfjEDicRHH7ng/640?wx_fmt=png)

将之前的栈顶的值 pop 到 rax 中，也就是 dll 的导出表的地址，[rax+24]则是 AddressOfNameOrdinals 的值，该值是储存函数序号的 RVA，那么同样，加 rdx(dll 基地址)得到实际内存中的地址，我们之前说过 AddressOfNameOrdinals 表的元素宽度为 2，所以 [r8+rcx*2] 取到了`LoadLibraryA`对应的序号，至于为什么要取到函数序号呢，这里要讲一下导出函数在 dll 中是什么存储的

> 引用博客：https://blog.csdn.net/Apollon_krj/article/details/77337333
> 
> 导出表有三个子表，也就是 AddressOfFunctions， AddressOfNames 和 AddressOfNameOrdinals 这三个，AddressOfFunctions 中存储的是导出函数的地址表，每一项为 4 字节，AddressOfNames 中储存的是函数名字符串的地址，每一项也是 4 字节，AddressOfNameOrdinals 中存储的是函数序号的地址，每一项为 2 字节，这三个子表中存储的地址都是 RVA，也就是说要加上 dll 的基地址才是在内存中的真实地址。这三个子表中的名字表和序号表是相互对应的，但是地址表和另外两个不一定是对应的，导出函数一定有地址，但是不一定有名字（声明为 noname 的），但一个函数也可以对应多个名字，不过一般情况下，名字表均不会大于地址表。
> 
> 比如我们自定义一个 dll 库，在导出时采用. def 文件的方式导出：
> 
> ```
> void InternetOpenA(
>   LPCSTR lpszAgent,
>   DWORD  dwAccessType,
>   LPCSTR lpszProxy,
>   LPCSTR lpszProxyBypass,
>   DWORD  dwFlags
> );
> 
> ```
> 
> 导出的函数有两个函数我们声明为 noname，故其在导出表中不存在名字。则其导出表的 NumberOfNames = 2，NumberOfFunctions = （6-1+1） = 6。即地址表长度为 6 宽度为 4，Size 为 24；名字表长度为 2，宽度为 4，Size 为 8；序号表长度为 2，宽度为 2，Size 为 4。对应的内存图如下（名字表的地址是按照从小到大排的，地址表有与我们. def 中指定了序号，因此是乱序的，如果不指定，编译器自动分配的一般也是有序的）
> 
> ![这里写图片描述]![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JvlFiaHxI9jlYK7dWd4e2FHWCm2Zib9Ir3LZEQBtDwib8f0n3aW5icT90JA/640?wx_fmt=png)
> 
> 有一点需要注意 AddressOfNameOrdinals 指向的序号表中的值是非准确的，应该均加上 Base 才是真正的序号 (Base 等于序号表中最小的值)

取到了函数序号后，就可以找到函数的真实地址了，但还要找到地址表的地址，故`mov r8d, [rax+1Ch]`取到地址表 RVA 后，再`add r8, rdx`得到真实地址，后面再`mov eax, [r8+rcx*4]`将`LoadLibraryA`的地址 (RVA) 放入 eax 中,`add rax, rdx`将函数的实际地址存入 rax

接着是一连串的 pop，这里的操作其实就是在布置函数参数了，因为后面`jmp rax`相当于一个函数调用，只需要根据 x64 的调用约定及函数定义将参数手动布置好，就相当于是完成一次函数调用了。

> 在 x64 中，参数的传递法则为：第一个参数放`rcx`，第二个参数放`rdx`，然后依次为`r8`, `r9`, 更多的参数放堆栈

那我们看他传递的参数是什么，我们只需反向找 push 就可以了，中间只有 push 和 pop 操作会影响到栈，这个师傅们可以自行寻找，我直接说了，这里的 pop 和上面最开始的 push 是一一对应的，由于`LoadLibraryA`其实只需一个参数，所以真正有用的其实就是 rcx 寄存器，对应`wininet`的字符串，所以此时相当于是执行`LoadLibraryA('wininet')`, 将 wininet.dll 加载进内存， `r10`中存储的是返回到![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JyrJG5jlsia9P0hjNLSykQk7Z4WsCFt6u2IyyFVPicSABKHN4AGQW3llQ/640?wx_fmt=png)

的返回地址，这一步 push+jmp 相当于是手动 call 了，所以`LoadLibraryA`执行完会返回到箭头处，而我们可以发现，到`call rbp`这一部分的代码和上面的其实差不多，同样是遍历 dll，不同的是这次遍历的 dll 中多了 wininet.dll， 这次寻找的函数的特征值是`0A779563Ah`, 对应着`InternetOpenA`函数，函数原型如下：

```
void InternetConnectW(
  HINTERNET     hInternet,
  LPCWSTR       lpszServerName,
  INTERNET_PORT nServerPort,
  LPCWSTR       lpszUserName,
  LPCWSTR       lpszPassword,
  DWORD         dwService,
  DWORD         dwFlags,
  DWORD_PTR     dwContext
);

```

传递的参数值均为 0， 根据微软的描述，该函数是应用程序调用的第一个 WinINet 函数。它告诉 Internet DLL 初始化内部数据结构，并为将来来自应用程序的调用做准备。当应用程序完成 Internet 功能使用后，应调用 InternetCloseHandle 释放句柄和所有相关资源。

如果成功调用，则返回传递给后续 WinINet 函数的有效句柄，否则返回 NULL，执行完成后返回到箭头位置，返回值通过 rax 返回

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JgJWd5VYDeZqoAu7Zic1ib74zIz9RXwVUSreW4njHlS5S95XiaUokRAspw/640?wx_fmt=png)

再然后就是一路跳转到这里

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JMmJiaNHhjQMkZ7LxibcfoToFDsCGCC0WKvkWo9v4ODxQf6MpO13sRCtg/640?wx_fmt=png)

大眼一看，这不是类似的套路嘛，我们看这次的参数，`pop rdx`是刚刚 call 过来的返回地址，其实就是我们 shellcode 中设置的 IP 地址

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JtGjeiaxRVrmdbHryFdqez9msUnAw4xLD2aQFyWibiajg7oyXWJMrQLjwg/640?wx_fmt=png)

rcx 对应刚刚`InternetOpenA`执行后返回的资源句柄，r8d 是`20020`（端口号），栈上是 0，0，3，0，0 执行函数的特征值是 0C69F8957h，该特征值对应的函数为`InternetConnectW`，函数原型如下：

```
void HttpOpenRequestW(
  HINTERNET hConnect,
  LPCWSTR   lpszVerb,
  LPCWSTR   lpszObjectName,
  LPCWSTR   lpszVersion,
  LPCWSTR   lpszReferrer,
  LPCWSTR   *lplpszAcceptTypes,
  DWORD     dwFlags,
  DWORD_PTR dwContext
);

```

返回值：如果连接成功，则返回会话的有效句柄，否则返回 NULL

传递的参数为：

```
BOOLAPI HttpSendRequestA(
  HINTERNET hRequest,
  LPCSTR    lpszHeaders,
  DWORD     dwHeadersLength,
  LPVOID    lpOptional,
  DWORD     dwOptionalLength
);

```

（这里是从动态调试中看的，至于为什么是从 rsp+28 开始布置栈上参数没有找到相关的资料 = = 有知道的师傅请务必留言赐教 Orz）

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JNBFLDYdRmemq3guOnfOhNUcicRCYSI5k4GftLUy3MK3Yf0mtG70qpqw/640?wx_fmt=png)

同样的套路，布置参数，通过特征值找到函数并调用，布置的参数我们后面一起说

`3B2E55EBh`对应的函数为 HttpOpenRequestW，函数原型为：

该函数用于创建一个 http 请求句柄，返回值：如果成功为 http 请求句柄，否则为 NULL

> HttpOpenRequest 函数创建一个新的 HTTP 请求句柄并将指定的参数存储在该句柄中。HTTP 请求句柄保存要发送到 HTTP 服务器的请求，并包含要作为请求的一部分发送的所有 RFC822 / MIME / HTTP 标头。

传递的参数为：

```
1: rcx InternetConnectW返回的HTTP session句柄
2: rdx 0000000000000000 0的话默认使用GET请求
3: r8 0000021003970186 "/qq6E"   一个描述你请求资源的字符串，当请求一个默认页面时令这个参数指向一个空串
4: r9 0000000000000000  HTTP 版本，这个参数为 NULL 时，默认使用""HTTP/1.1""
5: [rsp+28] 0000000000000000 
6: [rsp+30] 0000000000000000 
7: [rsp+38] FFFFFFFF84400200 
8: [rsp+40] 0000000000000000


```

然后就是下面的代码：

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JrF7V5MsQnP6XWOuHYjoXlyicTLOCBsdDuOOibMr6oR57DHrfI3we1iavg/640?wx_fmt=png)

`add rbx, 50h`之后，rbx 就指向了 user-agent：

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0Jw6oSib5xPpaV01rxvbxBXiaOPtPsbVD58ZWYicwUiateSKwHd18k9mm5pw/640?wx_fmt=png)

然后常规操作不再细说，`7B18062Dh`对应着 HttpSendRequestA ，函数原型如下：

该函数将之前的请求发送到 http 服务器上，成功则返回 True，失败返回 False，布置的参数为：

```
1: rcx 0000000000CC000C 
2: rdx 00000210039701D6 "User-Agent: Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)\r\n"
3: r8 FFFFFFFFFFFFFFFF 
4: r9 0000000000000000 
5: [rsp+28] 00000210039701D6 "User-Agent: Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)\r\n"



```

然后就是根据是否成功发送请求进行分支跳转，如果 没有成功发送请求，则执行如下代码重试，最多重试 10 次，然后就会进这里

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0Jd05tLZMOuhLicfeJjpiaUItoyibmuMlbcDbGnPibqKE4BWlfNM3IHZnEkw/640?wx_fmt=png)

我们看由于没有给 r10 赋值，所以会触发异常，程序就中断了

而如果成功执行的话，会执行如下代码：

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JXMYhRtkECplfyRpjx0QtxbdJO6x6Jmm36605Foqicfz9ic4kIqfDCr6w/640?wx_fmt=png)

这里看这个参数，如果做过手动加载 shellcode 的师傅估计一眼就能看出来这是 virtualAlloc 函数的参数，该函数的函数原型如下：

```
LPVOID VirtualAlloc(
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  flAllocationType,
  DWORD  flProtect
);


```

> 在调用进程的虚拟地址空间中保留，提交或更改页面区域的状态。此功能分配的内存将自动初始化为零。如果成功执行，则返回申请的空间页的基地址，否则返回 NULL

函数参数为：

```
1: rcx 0000000000000000 
2: rdx 0000000000400000 
3: r8 0000000000001000 
4: r9 0000000000000040


```

申请过空间后，就是

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0JibflbJ8Q0sTveicwqgicPIqq4qqfc17wAYgYjL4FHsibHGH4GuOfKdCN3A/640?wx_fmt=png)

调用 InternetReadFile 函数，原型：

```
BOOLAPI InternetReadFile(
  HINTERNET hFile,
  LPVOID    lpBuffer,
  DWORD     dwNumberOfBytesToRead,
  LPDWORD   lpdwNumberOfBytesRead
);


```

> 从 InternetOpenUrl，FtpOpenFile 或 HttpOpenRequest 函数打开的句柄中读取数据, 成功则返回 True，失败则为 false

参数为：

```
1: rcx 之前的资源句柄
2: rdx 000001C58E180000 
3: r8 0000000000002000 
4: r9 000000822B32F250


```

然后就是执行复制出来的代码了

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzB97zOOUMGXcib83AwHF8R0Jok03KOyiaNAWuKSgzGHDSdYTMqWYrKjicyPKOlxtd4O2lyXWytn2tuUQ/640?wx_fmt=png)

整个 cs 自动生成的 shellcode 分析就结束了，我们可以发现其实这段 shellcode 只是完成了从 cs 的 teamserver 上下载代码执行的功能，真正的恶意代码还需要接着分析，当然通过已有的分析，我们已经可以做到对 shellcode 做到二进制层面的修改混淆，甚至可以加任意的无关代码，想怎样就怎么，极大程度的规避杀软通过特征码识别恶意程序的查杀方式, 至于真正的恶意代码，我们后续在接着分析。

**_后记_**

  

有想一起研究免杀技术或者二进制技术的师傅萌，简历请砸 Gality@wgpsec.org，欢迎各位师傅一起交流学习

公众号

  

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec 狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

![](https://mmbiz.qpic.cn/mmbiz_jpg/4LicHRMXdTzBhAsD8IU7jiccdSHt39PeyFafMeibktnt9icyS2D2fQrTSS7wdMicbrVlkqfmic6z6cCTlZVRyDicLTrqg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/gdsKIbdQtWAicUIic1QVWzsMLB46NuRg1fbH0q4M7iam8o1oibXgDBNCpwDAmS3ibvRpRIVhHEJRmiaPS5KvACNB5WgQ/640?wx_fmt=gif)