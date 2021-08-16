> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BdmrN-8lR9AXM8jX3yxJAA)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0f37879QPsePTXHmLMD7lX7qSLLyiaZN9DjST4F0lntFrFkiaqmSqGWOPQ/640?wx_fmt=png)

**0x0 前言**
----------

简单叙述下自己研究 DLL 劫持的原因。

*   0. 学习 window 的库加载思想
    
*   1. 为了在 sangforSRC 挖个洞去长沙打个卡。
    
*   2. 免杀及其权限持久化维持的一个思路
    

**0x1 DLL 是什么**
---------------

动态链接库 (Dynamic-Link-Library, 缩写 dll), 是微软公司在微软视窗操作系统中实现共享函数库概念的一种实现方式。这些库函数的扩展名是. DLL、.OCX(包含 ActiveX 控制的库) 或者. DRV(旧式的系统的驱动程序)

> 所谓动态链接, 就是把一些经常会共享的代码 (静态链接的 OBJ 程序库) 制作成 DLL 档, 当可执行文件调用到 DLL 档内的函数时，Windows 操作系统才会把 DLL 档加载进存储器内，DLL 档本身的结构就是可执行档，当程序有需求时函数才进行链接。通过动态链接方式，存储器浪费的情形将可大幅降低。静态链接库则是直接链接到可执行文件
> 
> DLL 的文件格式与视窗 EXE 文件一样——也就是说，等同于 32 位视窗的可移植执行文件（PE）和 16 位视窗的 New Executable（NE）。作为 EXE 格式，DLL 可以包括源代码、数据 &action=edit&redlink=1) 和资源 &action=edit&redlink=1) 的多种组合。
> 
> 还有更广泛的定义, 这个没必要去理解了。

一些与之相关的概念:

> 静态库与动态库的比较
> 
> 静态库被链接后直接嵌入可执行文件中
> 
> 好处: 不需要外部函数支持, 无环境依赖, 兼容性好。
> 
> 坏处: 容易浪费空间，不方便修复 bug。
> 
> 动态库的好处与静态库相对。
> 
> Linux 下静态库名字一般是: libxxx.a window 则是: *.lib、*.h
> 
> Linux 下动态库名字一般是: libxxx.so window 则是: .dll、.OCX(..etc)

**0x2 DLL 的用途**
---------------

DLL 动态链接库，是程序进行动态链接时加载的库函数。

故动态链接最直接的好处是磁盘和内存的消耗减少，这也是 dll 最初的目的。

不过，dll 也有缺点，就是容易造成版本冲突, 比如不同的应用程序共享同一个 dll, 而它们需求的是不同的版本，这就会出现矛盾, 解决办法是把不同版本的 dll 放在不同的文件夹中。

**0x3 入门 DLL 的使用**
------------------

### 0x3.1 编写 TestDll.dll

1. 采用 vs2017 新建 DLL 项目

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fFHE3Jun6Bt4QvIrqib1eFHt1GLTkk3K6lic5e3pBJc1k4971ocM2j78g/640?wx_fmt=png)

2. 分析 DLL 的组成

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fvGqTt0qQg5sXy99jO6ASKhMBv89lCoiaYExRAkia4sv0hZTOMgKHAsjQ/640?wx_fmt=png)

其中 dllmain.cpp 代码如下

每个 DLL 都可以有一个入口点函数 DllMain, 系统会在不同时刻调用此函数。

```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"

BOOL APIENTRY DllMain( HMODULE hModule, // 模块句柄
                       DWORD  ul_reason_for_call, // 调用原因
                       LPVOID lpReserved // 参数保留
                     )
{
    switch (ul_reason_for_call) // 根据调用原因选择不不同的加载方式
    {
    case DLL_PROCESS_ATTACH: // DLL被某个程序加载
    case DLL_THREAD_ATTACH: // DLL被某个线程加载
    case DLL_THREAD_DETACH: // DLL被某个线程卸载
    case DLL_PROCESS_DETACH: //DLL被某个程序卸载
        break;
    }
    return TRUE;
}
```

我们可以在该文件下引入 Windows.h 库, 然后编写一个 msg 的函数。

```
#include <Windows.h>

void msg() {
    MessageBox(0, L"Dll-1 load  succeed!", 0, 0);
}

接下来在解决方案资源管理下的项目下打开头文件中的framework.h来导出msg函数.
#pragma once

#define WIN32_LEAN_AND_MEAN             // 从 Windows 头文件中排除极少使用的内容
// Windows 头文件
#include <windows.h>

extern "C" __declspec(dllexport) void msg(void);
```

然后点击生成中的重新生成解决方案编译得到 TestDll.dll 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fdopOZWJOEdf5MQg4DGAOEZL8uz4ZuqiaR2oMGxnp9YbORB7GN6Tb4dw/640?wx_fmt=png)

可以用 16 进制文件查看下 dll 的文件头，正如上面所说的一样，和 exe 是一样的。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0f3gdPqqFcB83Nr1TAb2YHU8ibT71IOCn9Bnemgtoh4xyYlj4C6ItU9Cw/640?wx_fmt=png)

### 0x3.2 调用 dll 文件

解决方案处右键新建一个项目，选择 > 控制台应用取名 hello

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fCl3Xnocd6vEXry0rVfM6tiaR0vFJWvDYRgtMt0owaR6Oet1AmTZkYXg/640?wx_fmt=png)

修改 hello.cpp 的文件内容如下:

```
#include <iostream>
#include <Windows.h>
using namespace std;

int main()
{
    // 定义一个函数类DLLFUNC
    typedef void(*DLLFUNC)(void);
    DLLFUNC GetDllfunc = NULL;
    // 指定动态加载dll库
    HINSTANCE hinst = LoadLibrary(L"TestDll.dll");
    if (hinst != NULL) {
        // 获取函数位置
        GetDllfunc = (DLLFUNC)GetProcAddress(hinst, "msg");
    }
    if (GetDllfunc != NULL) {
        //运行msg函数
        (*GetDllfunc)();
    }
}
```

然后 ctrl+F5, 运行调试。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fCl3Xnocd6vEXry0rVfM6tiaR0vFJWvDYRgtMt0owaR6Oet1AmTZkYXg/640?wx_fmt=png)

可以看到成功加载了我们写的 msg 函数。

有关代码中更多的细节的解释可以参考: C++ 编写 DLL 文件

**0x4 DLL 劫持漏洞**
----------------

### 0x4.1 原理简述

什么是 DLL 劫持漏洞 (DLL Hijacking Vulnerability)?

> 如果在进程尝试加载一个 DLL 时没有并没有指定 DLL 的绝对路径，那么 Windows 会尝试去按照顺序搜索这些特定目录来查找这个 DLL, 如果攻击者能够将恶意的 DLL 放在优先于正常 DLL 所在的目录，那么就能够欺骗系统去加载恶意的 DLL，形成” 劫持”,CWE 将其归类为 UntrustedSearch Path Vulnerability, 比较直译的一种解释。

### 0x4.2 查找 DLL 目录的顺序

正如动态链接库安全 、动态链接库搜索顺序微软的官方文档所说,

在 Windows XP SP2 之前 (不包括), 默认未启用 DLL 搜索模式。

Windows 查找 DLL 目录及其顺序如下:

> 1.  The directory from which the application loaded.
>     
> 2.  The current directory.
>     
> 3.  The system directory. Use the GetSystemDirectory function to get the path of this directory.
>     
> 4.  The 16-bit system directory. There is no function that obtains the path of this directory, but it is searched.
>     
> 5.  The Windows directory. Use the GetWindowsDirectory function to get the path of this directory.
>     
> 6.  The directories that are listed in the PATH environment variable. Note that this does not include the per-application path specified by the App Paths registry key. The App Paths key is not used when computing the DLL search path.
>     

在 Windows 下, 几乎每一种文件类型都会关联一个对应的处理程序。

首先 DLL 会先尝试搜索启动程序所处的目录 (1)，没有找到，则搜索被打开文件所在的目录 (2), 若还没有找到, 则搜索系统目录 (3), 若还没有找到, 则向下搜索 16 位系统目录，…Windows 目录… Path 环境变量的各个目录。

这样的加载顺序很容易导致一个系统 dll 被劫持，因为只要攻击者将目标文件和恶意 dll 放在一起即可, 导致恶意 dll 先于系统 dll 加载，而系统 dll 是非常常见的，所以当时基于这样的加载顺序，出现了大量受影响软件。

后来为了减轻这个影响, 默认情况下，从 Windows XP Service Pack 2（SP2）开始启用安全 DLL 搜索模式。

> 1.  The directory from which the application loaded.
>     
> 2.  The system directory. Use the GetSystemDirectory function to get the path of this directory.
>     
> 3.  The 16-bit system directory. There is no function that obtains the path of this directory, but it is searched.
>     
> 4.  The Windows directory. Use the GetWindowsDirectory function to get the path of this directory.
>     
> 5.  The current directory.
>     
> 6.  The directories that are listed in the PATH environment variable. Note that this does not include the per-application path specified by the App Paths registry key. The App Paths key is not used when computing the DLL search path.
>     

可以看到当前目录被放置在了后面, 对系统 dll 起到一定的保护作用。

注:

> 强制关闭 SafeDllSearchMode 的方法:
> 
> 创建注册表项:
> 
> ```
> Include the following filters:
> Operation is CreateFile
> Operation is LoadImage
> Path contains .cpl
> Path contains .dll
> Path contains .drv
> Path contains .exe
> Path contains .ocx
> Path contains .scr
> Path contains .sys
> 
> Exclude the following filters:
> Process Name is procmon.exe
> Process Name is Procmon64.exe
> Process Name is System
> Operation begins with IRP_MJ_
> Operation begins with FASTIO_
> Result is SUCCESS
> Path ends with pagefile.sys
> ```
> 
> 值为 0

不过从上面分析可以知道, 系统 dll 应该是经常调用的, 如果我们对程序安装的目录拥有替换权限，比如装在了非系统盘，那么我们同样可以利用加载顺序的 (1) 来劫持系统的 DLL。

从 Windows7 之后, 微软为了更进一步的防御系统的 DLL 被劫持，将一些容易被劫持的系统 DLL 写进了一个注册表项中，那么凡是此项下的 DLL 文件就会被禁止从 EXE 自身所在的目录下调用，而只能从系统目录即 SYSTEM32 目录下调用。注册表路径如下：

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs
```

win10 的键值项, 如图:

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fyiccpHmnicDnUabrIFe8P7pXkt4PRgUcHGZbKIHABsw0jia1CZZgA2Srw/640?wx_fmt=png)

这样子就进一步保护了系统 dll, 防止这些常用的 dll 被劫持加载。

但是如果开发者滥用 DLL 目录，依然会导致 DLL 劫持问题。(开发真难… orz)

### 0x4.3 防御思路

*   调用第三方 DLL 时, 使用绝对路径
    
*   调用 API SetDllDirectory(L”“) 将当前目录从 DLL 加载顺序中移除
    
*   开发测试阶段，可以采用 Process Monitor 进行黑盒复测
    

**0x5 实例演示**
------------

这里我们需要使用一个工具:Process Monitor v3.60

操作过程如动态链接库安全所说:

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0f71DJAM76hdO7R6yVQiclXfYqFvLCpYtaia31rtbic6l2lP514Jzzh9afQ/640?wx_fmt=png)

打开进程监视器的时候, 会要求填入过滤器。

一次填好即可 (通过上面的配置，我们可以过滤大量无关的信息, 快速定位到 DLL 确实的路径)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fbSlsV1oWEvFrhVMVFsXsxq7AnibUOicGYF6icysxjCrmK7Tehl10l2fSg/640?wx_fmt=png)

然后我们随便打开一个程序, 这里我使用的是深 x 服的 EasyConnectInstaller:

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fePicQ0MrojFAl3dcSwAjicEy7MwE0SKEibicU1YliaiaB9Esf0bBFJU9HoIQ/640?wx_fmt=png)

可以看到这里最终会去尝试加载当前目录的一些 dll, 这里可以尝试进行替换 rattler 中的 payload.dll 名字即可, 点击执行就可以弹出 calc 了。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fibickoGL6O4hrL8WMfnnFEdiaEoOiaQQZviaejG0Pygr8H9NfwudUcyAv2w/640?wx_fmt=png)

**0x6 自动化挖掘**
-------------

### 0x6.1 Ratter

1. 下载地址:https://github.com/sensepost/rattler/releases/

2. 使用

Rattler_x64.exe NDP461-KB3102438-Web.exe 1

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fLyzn38oVbZRuxxicWVKRpHeD2lGuoNIkw7R9RJziaAJDFBicVu5dOo9sw/640?wx_fmt=png)

结果发现这个并没有检测出来, 可能是 calc.exe 启动失败的原因, 个人感觉这个工具并不是很准确。

### 0x6.2 ChkDllHijack

1. 下载地址:[https://github.com/anhkgg/anhkgg-tools]

2. 使用 windbg 导出 module

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fNYCBBpwvn3CWfKq7CJ3DRxjHzMEpQiaQibPbjKJalYjURlMeQB4oPwOQ/640?wx_fmt=png)

然后打开 chkDllHijack, 粘贴处要验证的 DLL 内容

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fTrZwqYADGxPKpYSVEeaYM6L4XDEGsh99ZUWE3IichXaxhPgTdafG41A/640?wx_fmt=png)

然后让他自己跑完即可, 如果成功下面就会出现结果。

否则就是失败:

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5puVQSH7Pmh4hXKXvUjL0fYt2TBABOj5oBAPgdem3pG2ElE68RRceJNBfTU7OSgHVlCRc0G1Uczg/640?wx_fmt=png)

**0x7 总结**
----------

综合来说, 我个人还是比较推荐采用 Process monitor 作为辅助工具, 然后自己手工验证这种挖掘思路的, 不过自动化的确挺好的，可以尝试自己重新定制下检测判断规则。本文依然倾向于入门的萌新选手, 后面可能会回归 DLL 代码细节和免杀利用方面来展开 (这个过程就比较需要耗时间 Orz, 慢慢填坑吧)。

**0x8 参考链接**
------------

.dll 文件编写和使用

DLL 劫持 - 免杀

DLL 劫持漏洞自动化识别工具 Rattler 测试

注入技术系列：一个批量验证 DLL 劫持的工具

恶意程序研究之 DLL 劫持

（点击 “阅读原文” 查看链接）  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)
----------------------------------------------------------------------------------------------------------------------------------------------

  

- End -  

精彩推荐

[渗透测试中的 Exchange](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649736957&idx=1&sn=ccbf22ab5e3576c28bf65b549e96801a&chksm=888cf692bffb7f84a811f0ea7cb15d6d954d29938a3ca6818073d1bdc2a51d15fed7ee03d691&scene=21#wechat_redirect)  

[HTTP 协议攻击方法汇总（下）](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649736836&idx=1&sn=fdd6150c8b5981b4de1ea4f7825f7de4&chksm=888cf6ebbffb7ffd0886d857a75cb49cc0edeef0bffc969f82f031db9ab0cfcdb4df2b5f80d4&scene=21#wechat_redirect)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[HTTP 协议攻击方法汇总（上）](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649736739&idx=1&sn=27599630789af348b407455ee36a5ba2&chksm=888cf64cbffb7f5a14f5d4029debea23c0157ec01bb1554dcf9402d1e95a0199c01366555adb&scene=21#wechat_redirect)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[强网杯 2020 决赛 RealWord 的 Chrome 逃逸——GOOexec（GOO）](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649736649&idx=1&sn=52bdb674b4460950e2302cf070e725f1&chksm=888cf5a6bffb7cb06d5ef37ebbde1e40ede7071b1aa55891b171a7f249dc5b28d90bc1e07692&scene=21#wechat_redirect)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  
![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)  

---------------------------------------------------------------------------------------------------------------------------------------------------

**戳 “阅读原文” 查看更多内容**