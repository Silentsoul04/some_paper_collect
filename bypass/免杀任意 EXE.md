> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rmGW1S-YyJEalDW_oIf9cg)

实现一个 PE 文件加载器
-------------

PE 文件
-----

### PE 文件简述

PE 文件的全称是 Portable Executable，意为可移植的可执行的文件，常见的 EXE、DLL、OCX、SYS、COM 都是 PE 文件，PE 文件是微软 Windows 操作系统上的程序文件（可能是间接被执行，如 DLL），这篇文章主要讲对 EXE 文件进行内存加载并运行的方法，代码实现，和完成一个 GUI 加载器工具的全过程。

### 文件结构

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdGPfpTtbeLjNIVIWpDq6YQU44FZFls4r0Sx8XUibhfOZJvkTs7ZZoWjVnj7sLb2zEOLRTorKKur0xQ/640?wx_fmt=jpeg)

  

由上图可以

1.  Dos Header
    
    是用来兼容 MS-DOS 操作系统的，目的是当这个文件在 MS-DOS 上运行时提示一段文字，大部分情况下是：This program cannot be run in DOS mode. 还有一个目的，就是指明 NT 头在文件中的位置。
    
2.  NT Header
    
    包含 windows PE 文件的主要信息，其中包括一个‘PE’字样的签名，PE 文件头（IMAGE_FILE_HEADER）和 PE 可选头（IMAGE_OPTIONAL_HEADER32）。
    
3.  Section Table
    
    是 PE 文件后续节的描述，windows 根据节表的描述加载每个节。
    
4.  Section
    
    每个节实际上是一个容器，可以包含代码、数据等等，每个节可以有独立的内存权限，比如代码节默认有读 / 执行权限，节的名字和数量可以自己定义。
    

无论 PE 文件在磁盘中还是在内存中，都少不了地址的概念，理解以下几个概念很重要。

*   虚拟地址 (Virtual Address)：在一个程序运行起来的时候，会被加载到内存中，并且每个进程都有自己的 4GB，这个 4GB 当中的某个位置叫做**虚拟地址**，由物理地址映射过来的，4GB 的空间并没有全部被用到。
    
*   基地址 (Image Base): 磁盘中的文件加载到内存当中的时候可以加载到任意位置，而这个位置就是程序的基址。EXE 默认的加载基址是 400000h,DLL 文件默认基址是 10000000h。需要注意的是基地址不是程序的入口点。
    
*   相对虚拟地址 (Relative Virtual Address): 为了避免 PE 文件中有确定的内存地址，引入了相对虚拟地址的概念。RVA 是在内存中相对与载入地址 (基地址）的偏移量，所以你可以发现前三个概念的关系：虚拟地址 = 基地址 + 相对虚拟地址
    
*   文件偏移地址 (FOA)：当 PE 文件储存在某个磁盘当中的时候，某个数据的位置相对于文件头的偏移量。
    
*   入口点 (OEP)：首先明确一个概念就是 OEP 是一个相对虚拟地址 (Relative Virtual Address)，然后使用 OEP + Image Base == 入口点的虚拟地址 (Virtual Address)，通常情况下，OEP 指向的程序真实的入口点，而不是 main 函数。
    

### 执行流程

这里有大佬做的 windows 执行 PE 文件全流程图，下面是地址：

https://github.com/corkami/pics/blob/master/binary/pe101/pe101l.png

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGPfpTtbeLjNIVIWpDq6YQU4zlPGIB2lxXEq7Uu0q2lW7OwtJzdesI4deVnRG1N6v2rC21GcQ6cvA/640?wx_fmt=png)

大概流程如下：

1.  加载 PE 文件：判断是否为 PE 文件，然后将要加载的文件读取到内存中，并且对齐。
    

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGPfpTtbeLjNIVIWpDq6YQU9vchMUqn2wRl9bfHguxPAgUy58mr86bGhicXibFGictEDMlibSvjicevFVA/640?wx_fmt=png)

  

2.  进行重定位：如果当前加载到内存当中的基址与 Option Header 的 Image Base 一样，即在理想基址中展开了，或重定位表 data[5] 的长度为 0，则不需要重定位。重定位表的 sizeOfBlock 是加上块头部 8 字节的大小。重定位元素也很简单，以 WORD 为单位，但要注意高 4 位为 0x3 才有效，修复重定位表时要检查该位是否有效。
    

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGPfpTtbeLjNIVIWpDq6YQUg0KkJ9CAiaWK2paw80KaR4q8FGxD0Ra6o1X1gtYg7ojWwCJA1YJxib0g/640?wx_fmt=png)

  

4.  构建导入表：通过偏移 + 内存基址，获取导入表第一个 dll 的数据，按照导入的 dll 逐个遍历，直到当前导入表的 OriginalFirstThunk 为 0，即遍历完毕。
    

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGPfpTtbeLjNIVIWpDq6YQUpicVFhicGUGwr0TnNDgBjSibMcqBibj02ddJD9vSbJKibbRqe9tVCDtPzBQ/640?wx_fmt=png)

  

PE 加载器
------

PE 加载器，就是将一个 PE 文件映射到自己的内存，然后启动其 main 函数运行程序。一个 PELoader 的实现，需要有几个注意点：内存对齐，修复 IAT 表，修复重定位表，将内存属性改为可执行。

### 内存对齐

根据 exe 文件在加载到内存中对齐粒度进行对齐

```
LPVOID MapImageToMemory(LPVOID base_addr){ LPVOID mem_image_base = NULL; PIMAGE_DOS_HEADER raw_image_base = (PIMAGE_DOS_HEADER)base_addr; FuVirtualAlloc MyVirtualAlloc = (FuVirtualAlloc)GetProcAddress(hKernel32, "VirtualAlloc");  if (IMAGE_DOS_SIGNATURE != raw_image_base->e_magic) {  return NULL; }  PIMAGE_NT_HEADERS nt_header = (PIMAGE_NT_HEADERS)(raw_image_base->e_lfanew + (UINT_PTR)raw_image_base); if (IMAGE_NT_SIGNATURE != nt_header->Signature) {  return NULL; }  if (nt_header->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR].VirtualAddress) {  return NULL; }  PIMAGE_SECTION_HEADER section_header = (PIMAGE_SECTION_HEADER)(raw_image_base->e_lfanew + sizeof(*nt_header) + (UINT_PTR)raw_image_base);  mem_image_base = MyVirtualAlloc((LPVOID)(nt_header->OptionalHeader.ImageBase), nt_header->OptionalHeader.SizeOfImage, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);  if (NULL == mem_image_base) {  mem_image_base = MyVirtualAlloc(NULL, nt_header->OptionalHeader.SizeOfImage, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE); }  if (NULL == mem_image_base) {  return NULL; }  memcpy(mem_image_base, (LPVOID)raw_image_base, nt_header->OptionalHeader.SizeOfHeaders);  for (int i = 0; i < nt_header->FileHeader.NumberOfSections; i++) {  memcpy((LPVOID)(section_header->VirtualAddress + (UINT_PTR)mem_image_base), (LPVOID)(section_header->PointerToRawData + (UINT_PTR)raw_image_base), section_header->SizeOfRawData);  section_header++; } return mem_image_base;}
```

### 修复 IAT 表

根据 PE 结构的导入表，加载所需的 dll，并获取导入函数的地址并写入导入表中

```
VOID FixImageIAT(PIMAGE_DOS_HEADER dos_header, PIMAGE_NT_HEADERS nt_header){ DWORD op; DWORD iat_rva; SIZE_T iat_size; HMODULE import_base; PIMAGE_THUNK_DATA thunk; PIMAGE_THUNK_DATA fixup; PIMAGE_IMPORT_DESCRIPTOR import_table = (PIMAGE_IMPORT_DESCRIPTOR)(nt_header->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress + (UINT_PTR)dos_header); DWORD iat_loc = (nt_header->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IAT].VirtualAddress) ? IMAGE_DIRECTORY_ENTRY_IAT : IMAGE_DIRECTORY_ENTRY_IMPORT;  iat_rva = nt_header->OptionalHeader.DataDirectory[iat_loc].VirtualAddress; iat_size = nt_header->OptionalHeader.DataDirectory[iat_loc].Size;  LPVOID iat = (LPVOID)(iat_rva + (UINT_PTR)dos_header);  FuVirtualProtect myVirtualProtect = (FuVirtualProtect)GetProcAddress(hKernel32, "VirtualProtect"); FuLoadLibraryA myLoadLibraryA = (FuLoadLibraryA)GetProcAddress(hKernel32, "LoadLibraryA");  myVirtualProtect(iat, iat_size, PAGE_READWRITE, &op);  while (import_table->Name) {  import_base = myLoadLibraryA((LPCSTR)(import_table->Name + (UINT_PTR)dos_header));  fixup = (PIMAGE_THUNK_DATA)(import_table->FirstThunk + (UINT_PTR)dos_header);  if (import_table->OriginalFirstThunk)  {   thunk = (PIMAGE_THUNK_DATA)(import_table->OriginalFirstThunk + (UINT_PTR)dos_header);  }  else  {   thunk = (PIMAGE_THUNK_DATA)(import_table->FirstThunk + (UINT_PTR)dos_header);  }  while (thunk->u1.Function)  {   PCHAR func_name;   if (thunk->u1.Ordinal & IMAGE_ORDINAL_FLAG64)   {    fixup->u1.Function = (UINT_PTR)GetProcAddress(import_base, (LPCSTR)(thunk->u1.Ordinal & 0xFFFF));   }   else   {    func_name = (PCHAR)(((PIMAGE_IMPORT_BY_NAME)(thunk->u1.AddressOfData))->Name + (UINT_PTR)dos_header);    fixup->u1.Function = (UINT_PTR)GetProcAddress(import_base, func_name);   }   fixup++;   thunk++;  }  import_table++; } return;}
```

### 修复重定位表

直接申请当前 exe 的 ImageBase 地址，如果加载到内存当中的基址与 Option Header 的 Image Base 一样，就相当于在理想基址中展开，不需要修复重定位表。但是这种方法一般用于 x64 的程序，因为 x86 程序的 Image Base 较低，被占用导致无法正常执行的几率很高。

```
mem_image_base = MyVirtualAlloc((LPVOID)(nt_header->OptionalHeader.ImageBase), nt_header->OptionalHeader.SizeOfImage, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

if (NULL == mem_image_base)
{
	mem_image_base = MyVirtualAlloc(NULL, nt_header->OptionalHeader.SizeOfImage, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
}
```

### 判断 C# 程序

可以通过 DataDirectory 的第 15 项，**IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR** 中 VirtualAddress 是否为空，来判断是否为 C# 程序，如果是 C# 程序，程序使用了 Donut 项目将 C# 程序转换为 shellcode，Donut 是一个 shellcode 生成工具，它可以从. NET 程序集中创建与位置无关的 shellcode payloads。此 shellcode 可用于将程序集注入任意 Windows 进程。给定一个任意. NET 程序集，参数和入口点（如 Program.Main），Donut 就可为我们生成一个与位置无关的 shellcode，并从内存加载它。项目地址如下：

https://github.com/TheWover/donut

判断是否为 PE 程序，并且判断程序位数以及是否为 C# 程序：

```
int CMFCLoaderDlg::checkBit(TCHAR* filePath,int& BIT,int& TYPE){ IMAGE_DOS_HEADER myDosHeader; IMAGE_NT_HEADERS myNTHeader; IMAGE_NT_HEADERS64 myNTHeader64; LONG e_lfanew; errno_t err; FILE* pfile = NULL;  if ((err = _wfopen_s(&pfile, filePath, L"rb")) != 0) {  MessageBox(_T("File open error!"), NULL, MB_ICONERROR);  return 0; } fread(&myDosHeader, 1, sizeof(IMAGE_DOS_HEADER), pfile); if (myDosHeader.e_magic != 0x5A4D) {  MessageBox(_T("Not a PE file!"), NULL, MB_ICONERROR);  fclose(pfile);  return 0; } e_lfanew = myDosHeader.e_lfanew; fseek(pfile, e_lfanew, SEEK_SET); fread(&myNTHeader, 1, sizeof(IMAGE_NT_HEADERS), pfile); switch (myNTHeader.FileHeader.Machine) {  case 0x014c:  {   BIT = 32;   if (myNTHeader.OptionalHeader.DataDirectory[0x0e].VirtualAddress)   {    TYPE = 1;   }   break;  }   case 0x8664:  {   BIT = 64;   fseek(pfile, e_lfanew, SEEK_SET);   fread(&myNTHeader64, 1, sizeof(IMAGE_NT_HEADERS64), pfile);      if (myNTHeader64.OptionalHeader.DataDirectory[0x0e].VirtualAddress)   {    TYPE = 1;   }   break;  }  default:   break; } return 0;}
```

### 程序加密

使用了 RC4 加密算法，将要加载的 PE 文件加密并存放在资源段中，这种方法其实免杀效果并不好，接下来可以考虑其他将 Loader 和 payload 分离的其他方法。

```
int commanMake(TCHAR* filePath, TCHAR* outfilePath, int BIT){ TCHAR* DATfilename; if (BIT == 32) {  DATfilename = L"x32PEloader.DAT"; } else if (BIT == 64) {  DATfilename = L"x64PEloader.DAT"; } else {  return 0; } HANDLE hPE = CreateFile(filePath, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL); if (hPE == INVALID_HANDLE_VALUE) {  wprintf(L"[!]  Unable to Open FIle %s\n", filePath);  CloseHandle(hPE);  return 0; } unsigned char* key = GeneratePassword(128);  int peSize = GetFileSize(hPE, NULL); PBYTE shellcode = (PBYTE)malloc(peSize + StreamKeyLenth); if (shellcode == NULL) {  return 0; } memcpy(shellcode, key, StreamKeyLenth); DWORD lpNumberOfBytesRead; PWCHAR fileName = outfilePath; int ret = ReadFile(hPE, shellcode + StreamKeyLenth, peSize, &lpNumberOfBytesRead, NULL); if (ret == 0) {  return 0; } StreamCrypt(shellcode + StreamKeyLenth, peSize, key, StreamKeyLenth);  if (CopyFile(DATfilename, fileName, FALSE) == 0) {  wprintf(L"[!]  Unable to Open FIle PEloader.DAT\n");  return 0; }  HANDLE  hResource = BeginUpdateResource(fileName, FALSE);  if (NULL != hResource) {  if (UpdateResource(hResource, RT_RCDATA, MAKEINTRESOURCE(404), MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT), (LPVOID)shellcode, peSize + sizeof(key)) != FALSE)  {   EndUpdateResource(hResource, FALSE);   wprintf(L"[+]  Successfully generated %s\n", fileName);  } } free(shellcode); CloseHandle(hPE); return 1;}
```

### 成品效果

对 x64 的 mimikatz 进行加载器生成，功能正常。

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGPfpTtbeLjNIVIWpDq6YQUiaQ3gyAt4tteS53wQVWwMJIQ1eUzVQlwINVyfhdkibM3fYe6Nxv4Qa9A/640?wx_fmt=png)

  

上传 VT 结果，免杀效果还凑活，针对国外杀软需要继续改进，目前免杀一些常用的提权工具完全够用。相关代码及工具上传至知识星球。

![](https://mmbiz.qpic.cn/mmbiz_png/cOCqjucntdGPfpTtbeLjNIVIWpDq6YQU3kFIgHPvHnrZdibHuCMwhnicA6HMQ7nwSs9KsPLdyPicKmt6aHOtebW6Q/640?wx_fmt=png)

  

参考文章和项目
-------

https://blog.csdn.net/kclax/article/details/93727011

https://bbs.pediy.com/thread-249133.htm

https://github.com/TheWover/donut

https://www.cnblogs.com/onetrainee/p/12938085.html

号外  

-----

宽字节安全团队第一期线下网络安全就业班 7 月 1 日开班了，由宽字节安全团队独立运营，一线红队大佬带队，有丰富的漏洞研究、渗透测试、应急响应的经验与沉淀，干货多多，欢迎添加客服咨询。

[点击查看详情](https://mp.weixin.qq.com/s?__biz=MzUzNTEyMTE0Mw==&mid=2247484744&idx=1&sn=705508138f99f87f5111289e5e68a344&scene=21#wechat_redirect)

客服微信：unicodesec

![](https://mmbiz.qpic.cn/mmbiz_jpg/cOCqjucntdEhsMUjTPslVricKT94iaKpb5sL2PolmEf1WwcEEuwFaIGL9U3ePh1KXDDK8yggpMPHwDUibcn5b17wg/640?wx_fmt=jpeg)