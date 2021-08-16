> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NIy0omE7yTyUZEX3FyyFZA)

![](https://mmbiz.qpic.cn/mmbiz_gif/ll2PVky3MmaYz8UWHicLDSC4e3NqcibOLicCiaYtmQm9A0Ied3OLs4Wibj6bLz8jmf0gB3k0h1aQNNVKNqjKFcZV0Mg/640?wx_fmt=gif)

IDA Pro 是 Hex-Rayd 公司销售的静态反编译软件，全称为交互式反汇编器专业版（Interactive Disassembler Professional），人们常称其为 IDA Pro。IDA Pro 首先是一个反汇编器，可以显示二进制汇编代码（可执行文件或 DLL），它提供的某些高级功能使操作者可以更容易理解汇编代码。其次，它又是一个调试器，通过它可以逐条调试二进制文件中的指令，从而确定当前正在执行指令及执行顺序等。

作为最优秀的反汇编软件之一，通常情况下人们利用 IDA Pro 进行挖掘、研究软件中的漏洞、逆向分析恶意代码及使用 IDC 脚本语言自动执行各项任务。也可以利用 IDA Pro 调试软件，修改软件中的数据或执行流程，从而达到改变软件功能的目的。

**********工业网络安全领域中，在没有源代码的情况下，人们通常利用 IDA Pro 分析恶意软件，了解恶意软件的运行机制，及时防御攻击。也可以利用 IDA Pro 进行漏洞分析（挖掘漏洞、分析漏洞、开发破解程序）等。**********

**本文先介绍一下 IDA PRO 的常用功能，后期安帝科技会继续带来使用 IDA PRO 更改简单 C++ 程序的过程分享。**

**什么是 IDA PRO**

**01**

**常用快捷键**

*   a：将数据转换为字符串；
    
*   F5：一键反汇编；
    
*   Esc：回退键，能够倒回上一部操作的视图；
    
*     ：空格键，切换 IDA-View 窗口中的图形视图和文本视图；
    
*   Shift+F12：可以打开 string 窗口，一键找出所有的字符串；
    
*   ctrl+w：保存 IDA 数据库；
    
*   ctrl+s：选择某个数据段，直接进行跳转；
    
*   x：对着某个函数、变量按该快捷键，可以查看它的交叉引用；
    
*   g：跳转到某个地址；
    
*   n：更改变量的名称；
    
*   y：更改变量的类型。
    

**02**

**窗口简介**

在 IDA PRO 中可以通过 View→Open Subviews 菜单打开用户需要显示的窗口。

**（1）IDA-View 窗口**

IDA-View 窗口即反汇编窗口是操作和分析二进制文件的主要工具。IDA-View 窗口有两种显示模式：默认的基于图形视图和面向文本的列表视图，用户可根据自己的喜好选择适合的视图模式。

*   **图形视图**
    

图形视图将一个函数分解为很多基本块，可以通过图形视图清楚地了解函数中的逻辑及控制流程。

图形视图使用不同颜色的箭头区分函数块中的执行方向。绿色线代表当条件为 true 时的执行分支，红色线代表当条件为 false 时的执行分支，蓝色线代表下一个即将执行块。

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYC9MEkX3keibj7rzv5sKIvIl52utCxia32kLprzvQA96sXaOQ1DOmUehcQ/640?wx_fmt=png)

*   **文本视图**
    

文本视图会呈现一个程序完整的反汇编代码清单，只有通过这个窗口才能查看一个二进制文件的数据部分。

可以通过 Options→General 命令打开 IDA 的常规选项，在这里可以在视图中添加偏移地址、栈指针、反汇编注释等。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYCIQgrYcOFuHuR2cMYmIkSQ9zWuutO3tmfJ22wTdJFD1QcszIt6NvovA/640?wx_fmt=jpeg)

在显示窗口的左侧提供箭头窗口，描述函数中的非线性流程。实线代表非条件跳转，虚线代表条件跳转。如果一个跳转将控制权转交给程序中某个以前的地址，这时会使用粗线进行表示，出现这类逆向流程，通常表示程序中存在循环逻辑。

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYC5DNhHCRgv27Q2xD5XVibnEIW67HWibW4tGjoZ3gzj7qnJE0djnqiblTiaQ/640?wx_fmt=png)

**（2）函数窗口**

函数窗口包括函数名、段名、起始地址、长度、参数信息等。

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYCibNcuicqmiciaX6xqIwtQD2CibkqiaCliaI5iaviaFPzlTsaIxTnicVtOaaviaMnQ/640?wx_fmt=png)

**（3）十六进制窗口**

十六进制窗口显示程序内容和列表的标准十六进制代码，以及其对应的 ASCII 字符。右击选择 “Edit” 选项或按下 F2 快捷键可进入十六进制编辑器，在编辑器修改对应数据后，右击选择 “Apply changes” 或按下 F2 提交数据，从而达到修改应用程序的效果。

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYCAMOujMEA6tMCiayAuEoD63WPrlqeibzvH3pBRBxcj9iaYAXtlTOOmhiaBg/640?wx_fmt=png)

**（4）结构体窗口**

结构体窗口用于显示 IDA 决定在一个二进制文件中使用任何复杂的数据结构布局。双击数据结构的名称，IDA 将展开该结构，可以查看该结构的详细布局，包括每个字段的名称和大小。可以在此窗口中进行创建和修改结构体的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYCCaBWKx98FQdvSGfs5WlcH5cPbPUUY0lMNfoklgtsku9bnzxuzkbibKA/640?wx_fmt=png)

**（5）枚举窗口**

如果 IDA 检测到标准枚举数据类型，将在枚举窗口中列出该数据类型。可以使用枚举来代替整数常量，提高反汇编代码的可读性。在枚举窗口中也可以定义自己的枚举类型，并将其用在经过反汇编的二进制代码中。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYCvMUMmUHbpmZKDEiayOByRdzegXcjI5o6CEayaTRIuACSEHyyIEricxvQ/640?wx_fmt=jpeg)

**（6）String 窗口**

String 窗口中显示的是从二进制文件中提取出来的所有字符串，以及每个字符串所在的地址。该窗口有助于通过程序运行输出逆向找出对应的代码片段。 

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYCcRvjZicVQJGtPn1G1icekFGeq4UmXet0TDRicSuAibatZBAFfmlYwKFGLg/640?wx_fmt=png)

**（7）函数调用窗口**

函数调用窗口展示了一个函数中调用了哪些其他函数，以及自身被哪些函数调用。

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmbHQOH9usfp1GtXeDoKQfYCpXIO6icteAymP8S9vrJtJicmSTz6u8KLGyEHA7oKIfHoQOCKovypPh9A/640?wx_fmt=png)

如上图所示：函数 main 在函数_scrt_common_main_seh(void) 中调用 1 次，main 函数本身调用了其他 9 个函数。

初次接触时，IDA Pro 中似乎有很多窗口，上述的 7 个窗口是反编译时常用的几个。具体区别为：

*   IDA-View 窗口主要用于操作、分析二进制文件；
    
*   函数窗口主要用于列举 IDA 在数据库中的函数；
    
*   十六进制窗口主要展示程序内容和列表的十六进制代码；
    
*   结构体窗口主要展示二进制文件中的复杂数据结构；
    
*   枚举窗口主要展示 IDA 检测到的标准枚举数据类型；
    
*   String 窗口主要展示二进制文件中的所有字符串；
    
*   函数调用窗口主要展示一个函数的上下级调用关系。
    

除了上述窗口，IDA Pro 中还有其他窗口，如：导入窗口、导出窗口、Names 窗口、段窗口、签名窗口、类型库窗口等。

上述窗口的理论知识已为大家进行了洗白，**《利用 IDA PRO 反汇编 C++ 程序之操作篇》即将上映，欢迎大家持续关注安帝科技，我们下周一见~~~**

**往期精选**

![](https://mmbiz.qpic.cn/mmbiz_gif/ll2PVky3MmYrt8sZBqiah8aNInQspzccnPAAhd0OKLQ5Ziad4ovQEohiamyrpYjQcwVAWROpYWmMXia1YRKPiczFSQw/640?wx_fmt=gif)

[安帝科技畅谈工业互联网安全解决之道](http://mp.weixin.qq.com/s?__biz=MzU3ODQ4NjA3Mg==&mid=2247493505&idx=1&sn=cd76dc9de8d8ffab5b3b70b0f452c9f3&chksm=fd760dd6ca0184c08d9b721cafcda5b0bde36b460425d5036f7fe244bf0d86249701116a6401&scene=21#wechat_redirect)  

[安帝科技煤炭行业工业网络安全态势感知系统解决方案](http://mp.weixin.qq.com/s?__biz=MzU3ODQ4NjA3Mg==&mid=2247493470&idx=1&sn=68a02791549db1382952edb2320be8a8&chksm=fd760d09ca01841fc388425555c7b70c4ffceb901e661c0a0e64e4abb95812c4a5d7555bd0ec&scene=21#wechat_redirect)  

[安帝科技智慧炼化态势感知平台建设探索](http://mp.weixin.qq.com/s?__biz=MzU3ODQ4NjA3Mg==&mid=2247493441&idx=1&sn=2b7d875a54bbc77405f5b1d9c3114f9b&chksm=fd760d16ca01840056ad1a285e86ef41f5e5f2b872c22f2a5eca916ff5f376028f5b561c350b&scene=21#wechat_redirect)  

  

  

**安帝科技**丨 **ANDISEC**

北京安帝科技有限公司是新兴的工业网络安全能力供应商，专注于网络化、数字化、智能化背景下的工业网络安全技术、产品、服务的探索和实践，创新应用网络空间行为学及工业网络行为验证，构建了工业大数据深度分析、威胁情报共享、威胁感知和协同响应等核心能力优势，为电力、石油石化、煤炭、烟草、轨道交通、智能制造等关键信息基础设施行业提供安全产品、服务和综合解决方案，工业网络安全态势感知平台已部署 3600 余家电厂。  

![](https://mmbiz.qpic.cn/mmbiz_gif/ll2PVky3MmaYz8UWHicLDSC4e3NqcibOLicM3zL7JicjErwAgYHma0KEtaf9jaFs0y5UT09nqabt8oIql9NUhs9L9A/640?wx_fmt=gif)  

**点击 “在看” 鼓励一下吧**

![](https://mmbiz.qpic.cn/mmbiz_png/ll2PVky3MmZOFFgf4oTyfUTx7y0ZBV9yIg8qmaSwAvbjiba2KQn7VgmXYIriagc9H0hMu1u0UIZr98JYSmyG9tibg/640?wx_fmt=png)