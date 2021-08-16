\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzI5MDQ2NjExOQ==&mid=2247493679&idx=1&sn=238e3f002e4d719e951d01d7bd16b8f5&chksm=ec1dd807db6a51115762173ba280cb2c439352d578aa785253289a72e803400f1cd8aa36ef1d&mpshare=1&scene=1&srcid=1012oBl7W8eJkdPMft8693tM&sharer\_sharetime=1602490460068&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=1fd465a69b407b1c1a1b92c496c9b8ff81cf333f2ec9beae7873fb5b86f95f78418e84e8c4b4ca534fd961ae3c29cbd84b890139ce050e4b0614da061b14b59769b760fc8960c81a59af4501a566192ac012e497990a75f1dc7d70fbca31b58338c0e0b426b3e10bdd45726d0cbe24e5302113fa2b42261b07efdc1a13582227&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=ARPdxl0r%2B6WviSDYICw5DDo%3D&pass\_ticket=2G6SwO4uyYCX4aTiQDJvW1D1IrAJXn1CnpH%2BbX1rykSOMZNKPaotYwa2vyHnTBud&wx\_header=0)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsbtpGoibk9sLYqrC1dOxQLpfwu6LMnh83c3Ytyb2RzCHvpHwKu5cSExA/640?wx_fmt=png)

so 文件是啥？so 文件是 elf 文件，elf 文件后缀名是`.so`，所以也被称之为`so 文件`, elf 文件是 linux 底下二进制文件，可以理解为 windows 下的`PE文件`，在 Android 中可以比作`dll`，方便函数的移植，在常用于保护 Android 软件，增加逆向难度。  

解析 elf 文件有啥子用？最明显的两个用处就是：1、so 加固；2、用于 frida(xposed) 的检测！ 

**本文使用 c 语言，编译器为 vscode。如有错误，还请斧正！！！**

### 一、SO 文件整体格式

so 文件大体上可分为四部分，一般来说从上往下是`ELF头部->Pargarm头部->节区(Section)->节区头`，其中，除了`ELF头部`在文件位置固定不变外，其余三部分的位置都不固定。整体结构图可以参考非虫大佬的那张图，图片如下：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsbNMMMSYBFia0mf6lozkggGYjJVPdbb213rQ8ZCwCCBPOcGpf3vRicmrQ/640?wx_fmt=png)

**解析语言之所以选择 c 语言，有两个原因：**

1、做 so 加固的时候可以需要用到，这里就干脆用 c 写成一个模板，哪里需要就哪里改，不像上次解析 dex 文件的时候用 python 写，结果后面写指令还原的时候需要用的时候在写一遍 c 版本代价太大了；

2、在安卓源码中，有个`elf.h`文件，这个文件定义了我们解析时需要用到的所有数据结构，并且给出了参考注释，是很好的参考资料。`elf.h`文件路径如下：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUs3acKa8KMUMXTzsYzAqndBhwKcJicKnrts8Q5dZBR9Y0asTXkCUU3epw/640?wx_fmt=png)

### 二、解析 ELF 头部

**ELF 头部数据格式在 elf.h 文件中已经给出，如下图所示：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsMLibUDesJz9I3mktKYX0iaA6fSm7EzV8vtiaDY409gia8eG2xLjR5yibWxg/640?wx_fmt=png)

**每个字段解释如下：**

1、e\_ident 数组：前 4 个字节为`.ELF`，是 elf 标志头，第 5 个字节为该文件标志符，为 1 代表这是一个 32 位的 elf 文件，后面几个字节代表版本等信息。

2、e\_type 字段：表示是可执行文件还是链接文件等，安卓上的 so 文件就是分享文件，一般该字段为 3，详细请看下图。

3、e\_machine 字段：该字段标志该文件运行在什么机器架构上，例如 ARM。

4、e\_version 字段：该字段表示当前 so 文件的版本信息，一般为 1

5、e\_entry 字段：该字段是一个偏移地址，为程序启动的地址。 

6、e\_phoff 字段：该字段也是一个偏移地址，指向程序头 (Pargram Header) 的起始地址。 

7、e\_shoff 字段：该字段是一个偏移地址，指向节区头 (Section Header) 的起始地址。

8、e\_flags 字段：该字段表示该文件的权限，常见的值有 1、2、4，分别代表 read、write、exec。 

9、e\_ehsize 字段：该字段表示 elf 文件头部大小，一般固定为 52.

10、e\_phentsize 字段：该字段表示程序头 (Program Header) 大小，一般固定为 32.

11、e\_phnum 字段：该字段表示文件中有几个程序头。 

12、e\_shentsize: 该字段表示节区头 (Section Header) 大小，一般固定为 40.

13、e\_shnum 字段：该字段表示文件中有几个节区头。

14、e\_shstrndx 字段：该字段是一个数字，这个表明了`.shstrtab 节区(这个节区存储着所有节区的名字，例如.text)`的节区头是第几个。 

**`e_type`具体值 (相关值后面有英文注释，这里就不再添加中文注释了)：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUs2cOiaF3UHkyIq3huuTJMFRxMpJFIcvFhFmMPCOgicAe3IDDofp1Jjw8w/640?wx_fmt=png)

**解析代码如下：**

```
struct DataOffest parseSoHeader(FILE \*fp,struct DataOffest off)
{
    Elf32\_Ehdr header;
    int i = 0;
    fseek(fp,0,SEEK\_SET);
    fread(&header,1,sizeof(header),fp);
    printf("ELF Header:\\n");
    printf("    Header Magic: ");
    for (i = 0; i < 16; i++)
    {
        printf("%02x ",header.e\_ident\[i\]);
    }
    printf("\\n");
    printf("    So File Type: 0x%02x",header.e\_type);
    switch (header.e\_type)
    {
    case 0x00:
        printf("(No file type)\\n");
        break;
    case 0x01:
        printf("(Relocatable file)\\n");
        break;
    case 0x02:
        printf("(Executable file)\\n");
        break;
    case 0x03:
        printf("(Shared object file)\\n");
        break;
    case 0x04:
        printf("(Core file)\\n");
        break;
    case 0xff00:
        printf("(Beginning of processor-specific codes)\\n");
        break;
    case 0xffff:
        printf("(Processor-specific)\\n");
        break;
    default:
        printf("\\n");
        break;
    }
    printf("    Required Architecture: 0x%04x",header.e\_machine);
    if (header.e\_machine == 0x28)
    {
        printf("(ARM)\\n");
    }
    else
    {
        printf("\\n");
    }
    printf("    Version: 0x%02x\\n",header.e\_version);
    printf("    Start Program Address: 0x%08x\\n",header.e\_entry);
    printf("    Program Header Offest: 0x%08x\\n",header.e\_phoff);
    off.programheadoffset = header.e\_phoff;
    printf("    Section Header Offest: 0x%08x\\n",header.e\_shoff);
    off.sectionheadoffest = header.e\_shoff;
    printf("    Processor-specific Flags: 0x%08x\\n",header.e\_flags);
    printf("    ELF Header Size: 0x%04x\\n",header.e\_ehsize);
    printf("    Size of an entry in the program header table: 0x%04x\\n",header.e\_phentsize);
    printf("    Program Header Size: 0x%04x\\n",header.e\_phnum);
    off.programsize = header.e\_phnum;
    printf("    Size of an entry in the section header table: 0x%04x\\n",header.e\_shentsize);
    printf("    Section Header Size: 0x%04x\\n",header.e\_shnum);
    off.sectionsize = header.e\_shnum;
    printf("    String Section Index: 0x%04x\\n",header.e\_shstrndx);
    off.shstrtabindex = header.e\_shstrndx;
    return off;
}
```

三、程序头 (Program Header) 解析  

**程序头在`elf.h`文件中的数据格式是`Elf32_Phdr`，如下图所示：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUs2lJxl7zmPpkmJt7032icME805zSbRPwNhW6WqPSEbkic5Hkyv8rz5ArQ/640?wx_fmt=png)

**每个字段解释如下：**

1、p\_type 字段：该字段表明了段 (Segment) 类型，例如`PT_LOAD`类型，具体值看下图，实在有点多，没办法这里写完。 

2、p\_offest 字段：该字段表明了这个段在该 so 文件的起始地址。 

3、p\_vaddr 字段：该字段指明了加载进内存后的虚拟地址，我们静态解析时用不到该字段。 

4、p\_paddr 字段：该字段指明加载进内存后的实际物理地址，跟上面的那个字段一样，解析时用不到。 

5、p\_filesz 字段：该字段表明了这个段的大小，单位为字节。 

6、p\_memsz 字段：该字段表明了这个段加载到内存后使用的字节数。 

7、p\_flags 字段：该字段跟 elf 头部的 e\_flags 一样，指明了该段的属性，是可读还是可写。 

8、p\_align 字段：该字段用来指明在内存中对齐字节数的。 

**`p_type`字段具体取值：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsHzHS0yMt08YviaiacgvdPSyEOwJzeIayKqUov0lwIXGicictrDX9Ig5iaOg/640?wx_fmt=png)

**解析代码：**

```
struct DataOffest parseSoPargramHeader(FILE \*fp,struct DataOffest off)
{
    Elf32\_Half init;
    Elf32\_Half addr;
    int i;
    Elf32\_Phdr programHeader;
    
    init = off.programheadoffset;
    for (i = 0; i < off.programsize; i++)
    {
        addr = init + (i \* 0x20);
        fseek(fp,addr,SEEK\_SET);
        fread(&programHeader,1,32,fp);
        switch (programHeader.p\_type)
        {
        case 2:
            off.dynameicoff = programHeader.p\_offset;
            off.dynameicsize = programHeader.p\_filesz;
            break;
        default:
            break;
        }
        printf("\\n\\nSegment Header %d:\\n",(i + 1));
        printf("    Type of segment: 0x%08x\\n",programHeader.p\_type);
        printf("    Segment Offset: 0x%08x\\n",programHeader.p\_offset);
        printf("    Virtual address of beginning of segment: 0x%08x\\n",programHeader.p\_vaddr);
        printf("    Physical address of beginning of segment: 0x%08x\\n",programHeader.p\_paddr);
        printf("    Num. of bytes in file image of segment: 0x%08x\\n",programHeader.p\_filesz);
        printf("    Num. of bytes in mem image of segment (may be zero): 0x%08x\\n",programHeader.p\_memsz);
        printf("    Segment flags: 0x%08x\\n",programHeader.p\_flags);
        printf("    Segment alignment constraint: 0x%08x\\n",programHeader.p\_align);
    }
    return off;
}
```

四、节区头 (Section Header) 解析  

**节区头在 elf.h 文件中的数据结构为`Elf32_Shdr`，如下图所示：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUstypbXMd903zRVibTeUaOmlcyuicNKYXxKIhf3co6UXI3mz4zaPGVQHEQ/640?wx_fmt=png)

**每个字段解释如下：**

1、sh\_name 字段：该字段是一个索引值，是`.shstrtab`表 (节区名字字符串表) 的索引，指明了该节区的名字。 

2、sh\_type 字段：该字段表明该节区的类型，例如值为`SHT_PROGBITS`, 则该节区可能是`.text`或者`.rodata`，至于具体怎么区分，当然看 sh\_name 字段。具体取值看下图。 

3、sh\_flags 字段：跟上面的一样，就不再细说了。 

4、sh\_addr 字段：该字段是一个地址，是该节区加载进内存后的地址。 

5、sh\_offset 字段：该字段也是一个地址，是该节区在该 so 文件中的偏移地址。 

6、sh\_size 字段：该字段表明了该节区的大小，单位是字节。 

7、sh\_link 和 sh\_info 字段：这两个字段只适用于少数节区，我们这里解析用不到，感兴趣的可以去看官方文档。 

8、sh\_addralign 字段：该字段指明在内存中的对齐字节。 

9、sh\_entsize 字段：该字段指明了该节区中每个项占用的字节数。 

**`sh_type`取值：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUs3uiaUVPcVuyicrZyFwQFibp7VSdAd9VrOibBvXGgVdiaUQqynS93OKccHFw/640?wx_fmt=png)

**解析代码：**

```
struct DataOffest parseSoSectionHeader(FILE \*fp,struct DataOffest off,struct ShstrtabTable StrList\[100\])
{
    Elf32\_Half init;
    Elf32\_Half addr;
    Elf32\_Shdr sectionHeader;
    int i,id,n;
    char ch;
    int k = 0;
    init = off.sectionheadoffest;
    for (i = 0; i < off.sectionsize; i++)
    {
        addr = init + (i \* 0x28);
        fseek(fp,addr,SEEK\_SET);
        fread(§ionHeader,1,40,fp); 
        switch (sectionHeader.sh\_type)
        {
        case 2:
            off.symtaboff = sectionHeader.sh\_offset;
            off.symtabsize = sectionHeader.sh\_size;
            break;
        case 3:
            if(k == 0)
            {
                off.stroffset = sectionHeader.sh\_offset;
                off.strsize = sectionHeader.sh\_size;
                k++;
            }
            else if (k == 1)
            {
                off.str1offset = sectionHeader.sh\_offset;
                off.str1size = sectionHeader.sh\_size;
                k++;
            }
            else
            {
                off.str2offset = sectionHeader.sh\_offset;
                off.str2size = sectionHeader.sh\_size;
                k++;
            }
            break;
        default:
            break;
        }
        id = sectionHeader.sh\_name;
        printf("\\n\\nSection Header %d\\n",(i + 1));
        printf("    Section Name: ");
        for (n = 0; n < 50; n++)
        {
            ch = StrList\[id\].str\[n\];
            if (ch == 0)
            {
                printf("\\n");
                break;
            }
            else
            {
                printf("%c",ch);
            }
        }
        printf("    Section Type: 0x%08x\\n",sectionHeader.sh\_type);
        printf("    Section Flag: 0x%08x\\n",sectionHeader.sh\_flags);
        printf("    Address where section is to be loaded: 0x%08x\\n",sectionHeader.sh\_addr);
        printf("    Offset: 0x%x\\n",sectionHeader.sh\_offset);
        printf("    Size of section, in bytes: 0x%08x\\n",sectionHeader.sh\_size);
        printf("    Section type-specific header table index link: 0x%08x\\n",sectionHeader.sh\_link);
        printf("    Section type-specific extra information: 0x%08x\\n",sectionHeader.sh\_info);
        printf("    Section address alignment: 0x%08x\\n",sectionHeader.sh\_addralign);
        printf("    Size of records contained within the section: 0x%08x\\n",sectionHeader.sh\_entsize);
    }
    return off;
}
```

五、字符串节区解析  

**PS: 从这里开始网上的参考资料很少了，特别是参考代码，所以有错误的地方还请斧正；因为以后的 so 加固等只涉及到几个节区，所以只解析了`.shstrtab`、`.strtab`、`.dynstr`、`.text`、`.symtab`、`.dynamic`节区！！！**

在 elf 头部中有个`e_shstrndx`字段，该字段指明了`.shstrtab`节区头部是文件中第几个节区头部，我们可以根据这找到`.shstrtab`节区的偏移地址，然后读取出来，就可以为每个节区名字赋值了，然后就可以顺着锁定剩下的两个字符串节区。

在 elf 文件中，字符串表示方式如下：字符串的头部和尾部用标示字节`00`标志，同时上一个字符串尾部标识符`00`作为下一个字符串头部标识符。例如我有两个紧邻的字符串分别是`a`和`b`，那么他们在 elf 文件中 16 进制为`00 97 00 98 00`。

解析代码如下 (PS: 因为编码问题，第一次打印字符串表没问题，但填充进 sh\_name 就乱码，所以这里只放上解析`.shstrtab`的代码，但剩下两个节区节区代码一样)：

```
void parseStrSection(FILE \*fp,struct DataOffest off,int flag)
{
    int total = 0;
    int i;
    int ch;
    int mark;
    Elf32\_Off init;
    Elf32\_Off addr;
    Elf32\_Word count;
    mark = 1;
    if (flag == 1)
    {
        count = off.strsize;
        init = off.stroffset;
    }
    else if (flag == 2)
    {
        count = off.str1size;
        init = off.str1offset;
    }
    else
    {
        count = off.str2size;
        init = off.str2offset;
    }     
    printf("String Address==>0x%x\\n",init);
    printf("String List %d:\\n\\t\[1\]==>",flag);
    for (i = 0; i < count; i++)
    {
        addr = init + (i \* 1);
        fseek(fp,addr,SEEK\_SET);
        fread(&ch,1,1,fp);
        if (i == 0 && ch == 0)
        {
            continue;
        }
        else if (ch != 0)
        {
            printf("%c",ch);
        }
        else if (ch == 0 && i !=0)
        {
            printf("\\n\\t\[%d\]==>",(++mark));
        }
    }
    printf("\\n");
    
}
```

六、.dynamic 解析  

**`.dynamic`在`elf.h`文件中的数据结构是`Elf32-Dyn`，如下图所示：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsiar1E5GrKGJAE0ibGRjm0CHgOGKMfwDYy6osN6vujOLmE4LBTj0F0lAw/640?wx_fmt=png)

**第一个字段表明了类型，占 4 个字节；第二个字段是一个共用体，也占四个字节，描述了具体的项信息。解析代码如下：**

```
void parseSoDynamicSection(FILE \*fp,struct DataOffest off)
{
    int dynamicnum;
    Elf32\_Off init;
    Elf32\_Off addr;
    Elf32\_Dyn dynamicData;
    int i;
    init = off.dynameicoff;
    dynamicnum = (off.dynameicsize / 8);
    printf("Dynamic:\\n");
    printf("\\t\\tTag\\t\\t\\tType\\t\\t\\tName/Value\\n");
    for (i = 0; i < dynamicnum; i++)
    {
        addr = init + (i \* 8);
        fseek(fp,addr,SEEK\_SET);
        fread(&dynamicData,1,8,fp);
        printf("\\t\\t0x%08x\\t\\tNOPRINTF\\t\\t0x%x\\n",dynamicData.d\_tag,dynamicData.d\_un);
    }
    
}
```

七、.symtab 解析

**该节区是该 so 文件的符号表，它在`elf.h`文件中的数据结构是`Elf32_Sym`，如下所示：**

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsFZu3icaAW24lA20vqcDpfBume0dmW5ibRhFW8YOeqiatR1X2cHaWViaL1Q/640?wx_fmt=png)

**每个字段解释如下：**

1、st\_name 字段：该字段是一个索引值，指明了该项的名字。 

2、st\_value 字段：该字段表明了相关联符号的取值。 

3、stz-size 字段：该字段指明了每个项所占用的字节数。 

4、st\_info 和 st\_other 字段：这两个字段指明了符号的类型。 

5、st\_shndx 字段：相关索引。 

**解析代码如下 (PS：由于乱码问题，索引手动固定了地址测试，有兴趣的挨个解析字符应该可以解决乱码问题)：**

```
void parseSoDynamicSection(FILE \*fp,struct DataOffest off)
{
    int dynamicnum;
    Elf32\_Off init;
    Elf32\_Off addr;
    Elf32\_Dyn dynamicData;
    int i;
    init = off.dynameicoff;
    dynamicnum = (off.dynameicsize / 8);
    printf("Dynamic:\\n");
    printf("\\t\\tTag\\t\\t\\tType\\t\\t\\tName/Value\\n");
    for (i = 0; i < dynamicnum; i++)
    {
        addr = init + (i \* 8);
        fseek(fp,addr,SEEK\_SET);
        fread(&dynamicData,1,8,fp);
        printf("\\t\\t0x%08x\\t\\tNOPRINTF\\t\\t0x%x\\n",dynamicData.d\_tag,dynamicData.d\_un);
    }
}
```

```
void parseSymtabSection(FILE \*fp,struct DataOffest off)
{
    Elf32\_Off init;
    Elf32\_Off addr;
    Elf32\_Word count;
    Elf32\_Sym symtabSection;
    int k,i;
    init = off.symtaboff;
    count = off.symtabsize;
    printf("SymTable:\\n");
    for (i = 0; i < count; i++)
    {
        addr = init + (i \* 16);
        fseek(fp,addr,SEEK\_SET);
        fread(&symtabSection,1,16,fp);
        printf("Symbol Name Index: 0x%x\\n",symtabSection.st\_name);
        printf("Value or address associated with the symbol: 0x%08x\\n",symtabSection.st\_value);
        printf("Size of the symbol: 0x%x\\n",symtabSection.st\_size);
        printf("Symbol's type and binding attributes: %c\\n",symtabSection.st\_info);
        printf("Must be zero; reserved: 0x%x\\n",symtabSection.st\_other);
        printf("Which section (header table index) it's defined in: 0x%x\\n",symtabSection.st\_shndx);
    }
    
}
```

八、.text 解析

**PS：这部分没代码了，只简单解析一下，因为解析 arm 指令太麻烦了，估计得写个半年都不一定能搞定，后续写了会同步更新在 github!!!**

`.text`节区存储着可执行指令，我们可以通过节区头部的名字锁定`.text`的偏移地址和大小，找到该节区后，我们会发现这个节区存储的就是 arm 机器码，直接照着指令集翻译即可，没有其他的结构。通过 ida 验证如下：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUssrjFBLYk0xeMsSxyIrxZTObMzjpEohDUrVzo511umzos7atMyHZJgw/640?wx_fmt=png)

### 九、代码测试相关截图

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsfaiaicFSiaczpMmOUqg9k4wfQhWA0bPNxBOyibj0iayQjZAbGj2GicdKTn5A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsxDgfdIGgjhTibdXBxlXtD09PadVOEZKIVeKADicoX58pviaOm4WdBnfUw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUs4h5Oib8zyhTcl242RBLfk8FEgn8KbfJkUAcaxvGqfXwV6UcicelFT1icA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsx1dYnHuZ7kY0D6MARjDn2kAYSYOjORrLgwZRNQn9RTDkylnKQ3dIew/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe8JGJK9fAA8xibRiadBvsDUsgyhOsHLYYUyC4Gzts6OtdySgxYU5q5hKHngic3LG4KTbzOib8xSYb8WQ/640?wx_fmt=png)

### 十、frida 反调试和后序

frida 反调试最简单的就是检查端口，检查进程名，检查 so 文件等，但最准确以及最复杂的是检查汇编指令，我们知道 frida 是通过一个大调整实现 hook，而跳转的指令就那么几条，我们是否可以通过检查每个函数第一条指令来判断是否有 frida 了！！！(ps：简单写一下原理，拉开写就太多了，这里感谢某大佬和我讨论的这个话题！！！)

**本来因为这个 so 文件解析要写到明年去了，没想到看起来代码量大，但实际要用到的地方代码量很少。。。**

**源码 github 链接：**

> https://github.com/windy-purple/parseso/

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe3nBUEiaFQfDH9ZEHwj03Otp8ibk7CRf8nWnOVv8LfG7lc1qq4OpibSY5TT657aGhtTLdk3jpGKiaORw/640?wx_fmt=png)