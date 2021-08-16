\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/LaVFzvrHFWKNLgDZscFGag)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)  

PE 文件是 Windows 操作系统下使用的可执行文件的格式，可执行系列：EXE、SCR，库系列：DLL、OCX、CPL、DRV，驱动程序：SYS、VXD 等都是 PE 文件

  
在 Windows 系统中，PE 文件被系统加载器映射到内存中，每一个程序都有自己的虚拟空间，这个虚拟空间的内存地址称为虚拟地址（Virtual Address\\VA）

  
相对虚拟地址（Relative Virtual Address\\RVA）是一个简单的，相对于 PE 文件载入地址的偏移位置，它是一个相对的地址（偏移）  
当 PE 文件在磁盘中时，某个数据位置相对于文件头的偏移量称为文件偏移地址（File Offset）  
所有 PE 文件以 64 字节 DOS 头开始，DOS 头只是为了兼容早期操作系统  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SIHadZLaHc1TRQ87y7l84iaebX89ELLTDAFxKOS3cCg1X11xA23icS5qA/640?wx_fmt=png)   
e\_magic：0x5A4D，MZ 标志

  
e\_lfanew：0x000000E8，NT 头偏移量  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4S6vIHEd6RPVqTHALwTSz8LUNkIGMxkrVeWiaVaN15oInEko8DZaVOyRg/640?wx_fmt=png)   
#define IMAGE\_DOS\_SIGNATURE         0x5A4D      // MZ  
这里采用的小端序存储数据，地址高位存储数据的高位，地址低位存储数据的低位，是一种逆序存储方式  
000000E8 就是 NT 头的位置，与 DOS 头之间隔了一段 DosStub 数据，这个数据是可变的，所以需要 e\_lfanew 来指定 NT 头的位置  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SAKlkDvt8ibjk30vu1JygFqeBnoqypBH6qUPjLJdvQeD7t89Q3IsvPwg/640?wx_fmt=png)   
NT 头定义如下  

```
typedef struct \_IMAGE\_NT\_HEADERS {
    DWORD Signature;        // PE标识
    IMAGE\_FILE\_HEADER FileHeader;        //文件头
    IMAGE\_OPTIONAL\_HEADER32 OptionalHeader;        //可选头
} IMAGE\_NT\_HEADERS32, \*PIMAGE\_NT\_HEADERS32;

Signature类型为DWORD，占用4个字节
#define IMAGE\_NT\_SIGNATURE           0x00004550   // PE00
```

  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SibqpIslaqxgCDOYC10ic3CTtTfmsuJq1ISr8Evo53KcAlCjqQchhs2NA/640?wx_fmt=png)   
文件头是表示文件大致属性的 IMAGE\_FILE\_HEADER 结构体  

```
typedef struct \_IMAGE\_FILE\_HEADER {
    WORD    Machine;        // 2个字节，指明支持的CPU类型
    WORD    NumberOfSections;        // 2个字节，指明节区数量
    DWORD   TimeDateStamp;                // 4字节，编译器生成这个PE文件的时间
    DWORD   PointerToSymbolTable;        // 4字节，调试符号相关，不作研究
    DWORD   NumberOfSymbols;        // 4字节，调试符号相关，不作研究
    WORD    SizeOfOptionalHeader;        // 2个字节，指明可选头 IMAGE\_OPTIONAL\_HEADER32 大小
    WORD    Characteristics;        // 2个字节，表明文件属性，如DLL文件，SYS文件等
} IMAGE\_FILE\_HEADER, \*PIMAGE\_FILE\_HEADER;


IMAGE\_OPTIONAL\_HEADER32是PE头结构体中最大的，定义如下

typedef struct \_IMAGE\_OPTIONAL\_HEADER {
    //
    // Standard fields.
    //

    WORD    Magic;        // 普通可执行文件为0x010B，PE32 （64位）值为0x020B
    BYTE    MajorLinkerVersion;
    BYTE    MinorLinkerVersion;
    DWORD   SizeOfCode;        // 代码区块的大小，通常是.text区块大小
    DWORD   SizeOfInitializedData;
    DWORD   SizeOfUninitializedData;
    DWORD   AddressOfEntryPoint;        // 程序入口点，RVA值
    DWORD   BaseOfCode;        // 代码段的起始地址，RVA值，通常是.text区块的起始RVA
    DWORD   BaseOfData;        // 数据段的起始地址，RVA值，通常是.data区块的起始RVA

    //
    // NT additional fields.
    //

    DWORD   ImageBase;                // PE文件在内存中首选的装载基地址
    DWORD   SectionAlignment;        // PE文件装载到内存时区块的对齐大小，假设.text区块的大小为0x7748，而SectionAlignment的大小为0x1000，那么对齐后的大小为0x8000字节
    DWORD   FileAlignment;                // 磁盘上PE文件中区块的对齐大小，对齐方式类似SectionAlignment
    WORD    MajorOperatingSystemVersion;
    WORD    MinorOperatingSystemVersion;
    WORD    MajorImageVersion;
    WORD    MinorImageVersion;
    WORD    MajorSubsystemVersion;
    WORD    MinorSubsystemVersion;
    DWORD   Win32VersionValue;
    DWORD   SizeOfImage;        // PE文件被装载到内存空间后总的大小，指从ImageBase到最后一个区块的大小
    DWORD   SizeOfHeaders;                // Dos头、DosStub、PE头以及区块头的总大小，并进行FileAlignment对齐后的大小
    DWORD   CheckSum;        // 校验和，一般的EXE文件通常为0，判断文件是否被修改
    WORD    Subsystem;                // 子系统，区分系统驱动文件与普通可执行文件
    WORD    DllCharacteristics;
    DWORD   SizeOfStackReserve;
    DWORD   SizeOfStackCommit;
    DWORD   SizeOfHeapReserve;
    DWORD   SizeOfHeapCommit;
    DWORD   LoaderFlags;
    DWORD   NumberOfRvaAndSizes;        // 指定最后一个成员是数组个数
    IMAGE\_DATA\_DIRECTORY DataDirectory\[IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES\];
} IMAGE\_OPTIONAL\_HEADER32, \*PIMAGE\_OPTIONAL\_HEADER32;
```

  
写一个程序，关掉随机基址  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SAyAZ5WsvoibPrc3ZL2Uia1fYjicyprvVjga4axaqESa9wC4BU3iaqa1GxQ/640?wx_fmt=png)   
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SjXB8kkRx8IiccJBVgb1s3HLbpYr6oxFxiatL9I0RQLDfibRpI57QLF98w/640?wx_fmt=png)   
AddressOfEntryPoint 是一个 RVA 值，所以当程序实际被加载到内存时，对应的入口点地址的虚拟地址

(VA) = ImageBase+AddressOfEntryPoint = 0x00400000+0x000012DA = 0x004012DA  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SjTKue8pONuzA3zALMT9ScuPBmNc1Xx5gLoFchvulTPSdDSZxI2Cia9A/640?wx_fmt=png)   
子系统查看  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SLrGUQEf0TVY3SzlkgVhQpeNSz6lZZJTuI0dGCAHB8xCjz2wawgHy1w/640?wx_fmt=png)   
SizeOfImage 验证  
DWORD   SizeOfImage;     

 // PE 文件被装载到内存空间后总的大小，指从 ImageBase 到最后一个区块的大小  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SLcpkabibfwIOdUfL5aNtjYeiadVBU39v138To9g5gMSsM57vARru6qcA/640?wx_fmt=png)   
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SuPwLN1gJ9zMtI7jIHKltj3ia4Yj5XV3mj8yZAibBtxZ59RVYZ17I0DdA/640?wx_fmt=png)   
OPTIONAL\_HEADER 末尾是一个数据目录表数组，数组元素个数为 16  
各元素对应表项：

  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SmMV3pWjDdtPgia67hgiboFclXG0NRagwP6Dysicvbmic0MSI2t7FJTpXow/640?wx_fmt=png)   
IMAGE\_DATA\_DIRECTORY 定义如下：  

```
typedef struct \_IMAGE\_DATA\_DIRECTORY {
    DWORD   VirtualAddress;        // 数据块的起始RVA地址
    DWORD   Size;                // 数据块的长度
} IMAGE\_DATA\_DIRECTORY, \*PIMAGE\_DATA\_DIRECTORY;
```

  
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SjPkmPaQvX1dZbdibIRuJVw1zVZ1cyvicdKL96axcBVucsRcAwkxcUUbw/640?wx_fmt=png)   
 ![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4SxDicZjUpoagicAIWkPVsqcjjIVvgRnqBUORDsibiaDicDZ85OCYvcW6iblhg/640?wx_fmt=png)   
![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL8XZgpPmJmgqDHSZIrQial4Sicpw4MvnWatiaXsjMM5qkxCbSK2P0jJtwI7X6VDuONecTdygymXQp47Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)