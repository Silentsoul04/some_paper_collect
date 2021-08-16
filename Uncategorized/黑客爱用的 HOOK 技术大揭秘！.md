\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/JRH318f7DTzPYloU5rb3gw)

> **来自公众号：小白学黑客**

什么是 HOOK 技术？
------------

病毒木马为何惨遭杀软拦截？

商业软件为何频遭免费破解？

系统漏洞为何能被补丁修复？

这一切的背后到底是人性的扭曲，还是道德的沦丧，尽请收看今天的专题文章：《什么是 HOOK 技术？》

上面是开个玩笑，言归正传，今天来聊的话题就是安全领域一个非常重要的技术：**HOOK 技术**。

HOOK，英文意思是 “钩子”

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLkXNDscHfIlpqXPcm1DRInl0no3xCGRkZfRlyhibNElopOxiaOwQRahpw/640?wx_fmt=png)

在计算机编程中，HOOK 是一种「劫持」程序原有执行流程，添加额外处理逻辑的一种技术。

按照这个定义，其实我们 Python 中的**装饰器**和 Java 中的**注解**，这种面向切面编程的手法在某种程度上来说，也算是 HOOK。

**不同的是，本文要探讨的 HOOK 并非属于程序原有的逻辑，而是在程序已经编译成可执行文件甚至已经在运行中的时候，如何劫持和修改程序的流程。**

按照劫持的目标不同，常见的 HOOK 有以下这些类型：

> *   Inline HOOK
>     
> *   IAT HOOK
>     
> *   C++ virtable HOOK
>     
> *   SEH HOOK
>     
> *   IDT HOOK
>     
> *   SSDT HOOK
>     
> *   IRP HOOK
>     
> *   TDI HOOK && NDIS HOOK
>     
> *   Windows Message HOOK
>     

接下来，咱们挨个来看一下。

Inline HOOK
-----------

程序和代码是给程序员们看的，计算机要运行，最终是要编译成 CPU 的机器指令才能执行。

**Inline HOOK** 的目标就是直接修改程序编译后的指令，属于最基础也最常见的 HOOK 技术。

下面我们以一个实例来感受一下 Inline HOOK 的效果：

```
void functionA() { cout << "this is function A" << endl;}void hookFunction() { cout << "this is hookFunction" << endl;}int main() { cout << "before hook" << endl; functionA(); // prepare hook unsigned char code\[5\] = { 0xe9, 0x00, 0x00, 0x00, 0x00 }; unsigned int offset = (unsigned int)hookFunction - ((unsigned int)functionA + 5); \*(unsigned int\*)&code\[1\] = offset;  // install hook unsigned long old = 0;  VirtualProtect(functionA, 0x1000, PAGE\_EXECUTE\_READWRITE, &old); memcpy(functionA, code, 5); cout << "after hook" << endl; functionA(); return 0;}
```

输出：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibL7RqhJtAXTpoMaTeGst2c8jc5XsBy8FPk7O25h8Jwu74Gtmia0p3nz5w/640?wx_fmt=png)

代码中定义了目标函数 functionA 和 hook 函数 hookFunction。

第一次调用，输出显示调用了原函数。

然后安装一个 HOOK，准备了一条 **jmp** 指令，覆盖函数入口处的指令。此时观察覆盖前后的函数指令变化对比：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLUOlUyQbnvdEKNC5RPLCMkliaP4qzpfWILFK08AdicTBnWRmAbsyyiavrg/640?wx_fmt=png)

再次调用该函数，则一进入就发生跳转，我们安装的 HOOK 函数得到了执行。

所以第二次输出显示 HOOK 函数得到了调用。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLLVzb7MfrphXWZ5ic04Pg8wyJeicKZhVWsqDCh1c51MSHHQxM44C34ARw/640?wx_fmt=png)

大部分情况下，我们习惯于在函数入口处执行 HOOK，但这并不是绝对的，还需要具体问题具体分析。比如如果我们需要等待函数执行完毕时拿到返回值才能介入处理，这个时候就需要在函数 return 的地方进行 HOOK。甚至有可能需要在函数中途某个地方介入，这个时候就需要更进一步的对函数的反编译指令进行分析，确定 HOOK 的点位和处理逻辑。

执行 Inline HOOK 非常关键的几点：

> *   指令所在的内存页是否允许写入操作，若只读，须先添加写入权限
>     
> *   需要动态解析目标位置处的指令，不能像上面那样暴力覆盖，否则会影响原来函数的执行逻辑
>     
> *   如果在 HOOK 处理函数中需要调用原函数，注意别陷入死循环
>     
> *   如果有参数，需要处理好堆栈平衡
>     

Inline HOOK 由于是直接在 CPU 机器指令层面上的操作，所以首先是无法做到跨平台。同时，还需要对 CPU 的指令集有一定程序的了解，具备一定的汇编语言功底。

好在，IT 行业从来不缺造轮子的人，已经有不少优秀的开源项目将 Inline HOOK 封装成库，比如：**Detours**。

* * *

除了直接修改函数的机器指令，还有一类 HOOK，它们修改的是某些重要的**函数指针**，从而达到劫持执行的目的。

形形色色的函数指针，就衍生出各式各样的 HOOK 技术。

IAT HOOK
--------

一个程序的所有代码一般不会全部都编译到一个模块中，分拆到不同的模块既有利于合作开发，也有利于代码管理，降低耦合。

动态链接库就提供了这样的能力，将不同的模块编译成一个个的动态库文件，在使用时引入调用。

在 Windows 平台上，动态链接库一般以 DLL 文件的形式存在，主程序模块一般是 EXE 文件形式存在。无论是 EXE 还是 DLL，都是属于 PE 文件。

一个模块引用了哪些模块的哪些函数，是被记录在 PE 文件的**导入表 IAT** 中。这个表格位于 PE 文件的头部，里面记录了模块的名字，函数的名字。

在模块加载时，模块加载器将解析对应函数的实际地址，填入到导入表中。

通过修改导入表 IAT 中函数的地址，这种 HOOK 叫 **IAT HOOK**。

SEH HOOK
--------

SEH 是 Windows 操作系统上**结构化异常处理**的缩写，在代码中通过 try/except 来捕获异常时，操作系统将会在线程的栈空间里安置一个异常处理器（其实就是一个数据结构），里面定义了发生异常时该去执行哪里的代码处理异常。

异常处理可以多级嵌套，那多个异常处理就构成了一个链表，存在于栈空间之上。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLFp5xwibcezfapFUTI6zWgJo3o3XdDhI5ukOKMs2IDMs3HYVaKUHgeWg/640?wx_fmt=png)

当发生异常时，操作系统系统就从最近的异常处理器进行寻求处理，如果能处理则罢了，不能处理就继续寻求更上一级的异常处理器，直到找到能处理的异常处理器。如果都没法处理，那对不起，只好弹出那个经典的报错对话框，进程崩溃。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLaYSbRepInRCemqzwwYpDEPwLqPR4uzPvZhPwSwx5Yc3m0iaNg1lcEew/640?wx_fmt=png)

**SEH HOOK** 针对的目标就是修改这些异常处理器中记录的函数指针，当异常发生时就能获得执行，从而劫持到执行流！因为这些异常处理器都位于线程的栈空间，修改起来并非难事。

C++ virtable HOOK
-----------------

C++ 是一门面向对象的编程语言，支持面向对象的三大特性：**封装性**、**继承性**、**多态性**。

其中的多态性，各个 C++ 编译器基本上都是通过一种叫**虚函数表**的机制来实现。

下面通过一个实际的例子来感受一下虚函数表在 C++ 多态性上发挥的作用。

```
#include <iostream>using namespace std;class Animal {public: virtual void breathe() {  cout << "Animal breathe" << endl; } virtual void eat() {  cout << "Animal eat" << endl; }};class Fish : public Animal {public: virtual void breathe() {  cout << "Fish breathe" << endl; } virtual void eat() {  cout << "Fish eat" << endl; }};class Cat : public Animal {public: virtual void breathe() {  cout << "Cat breathe" << endl; } virtual void eat() {  cout << "Cat eat" << endl; }};int main() { Animal\* animal = nullptr; Fish\* fish = new Fish(); Cat\* cat = new Cat(); animal = fish; animal->breathe(); animal->eat(); cout << "--------------" << endl; cout << "sizeof(fish) = " << sizeof(fish) << endl; cout << "sizeof(cat) = " << sizeof(cat) << endl; cout << "--------------" << endl; animal = cat; animal->breathe(); animal->eat(); delete fish; delete cat; return 0;}
```

输出：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLiaQ6FfgZe2raU0p2XbV3x8wdf20M1a0A4ibJpIfLPib39vl19HLoz4uPg/640?wx_fmt=png)

通过上面的输出，可以看到，fish 和 cat 对象都只占据 4 个字节。因为这两个类都没有成员变量，唯一需要存储的就是一个虚函数表指针。

以 cat 对象为例，看一下它的地址，是 0x005cfc30：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLmLZ0B8WjFrp2bEqficib9hPPrwEScjF4tko8tKfebvclUtibLpDS2Lnow/640?wx_fmt=png)

看一下这个地址起始的 4 个字节，是什么：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLZjkdWk2zVvWvMShsWmICG2u4aqN4E0Kgg8Ih6SVJcuqhgm3nuNtGjg/640?wx_fmt=png)

虚表指针是 0x000d9b90（这里需要注意字节顺序）。

通过这个地址，找到虚函数表，里面有两个函数地址：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLnVCUs2jxIVdKa8P6YOibcORguvfzQW1lAI9ePAO75gs6iacnQsUqY3Ng/640?wx_fmt=png)

查看这两个地址，都是指向了一个 jmp 指令，分别跳到了 Cat 类的两个虚函数。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLE5RxNicq7kiaqLjgduBC1cJvnhaE9xyHs1C2LQxd5oiaAbV9Fxo2vsqfg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLiagvVUlS20X2ngBQ0GYfJDglJ28OeboWhJfNrBUqoh4ibiajHRibWdasJQ/640?wx_fmt=png)

通过上面的实例，总结一下对象、虚函数表和虚函数代码之间的关系如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLg3IgTDS3zd02z5Gcia7rUc3miaVJ38ZlOXPRdtON21VjNLy7rbsg1MDQ/640?wx_fmt=png)

每个包含虚函数的类对象，在内存中都有一个指针，位于对象头部，指向的是一个虚函数表，表中的每一项都是虚函数地址。

类继承后，如果重写了父类的虚函数，子类对象指向的表格中对应函数的地址将会更新为子类的函数。

这样，使用父类指针指向子类对象，通过指针调用虚函数时，就能调用到子类重写的虚函数，从而实现**多态性**。

既然是记录函数地址的表格，那就有存在被篡改的可能，这就是 **C++ virtable HOOK**。

通过篡改对应虚函数的地址，实现对相应函数调用的拦截。

实施这种 HOOK，需要逆向分析目标 C++ 对象的结构，掌握虚函数表中各个函数的位置，才能精准打击。

* * *

上面几种 HOOK，修改的都是应用层的函数指针，而操作系统内核中还有一些非常重要的表格，它们的表项中记录了一些更加关键的函数，HOOK 这些表格中的函数是非常高危的操作，操作不当将导致操作系统崩溃。当然，高风险高回报，HOOK 这些函数，能实现一些非常强大的功能，是病毒、木马、安全软件非常爱干的事情。

SSDT HOOK
---------

**系统调用**是操作系统提供给应用程序的编程接口 API，应用程序通过这些 API 得以操作计算机的资源（如进程、网络、文件等）。

执行系统调用的时候，CPU 将从用户模式切换到内核模式，进入内核后，将会根据系统调用的 API 编号，去找到对应的系统服务函数，实现对应 API 的功能。

操作系统将所有的系统服务函数地址，存放在了一个表格中，这个表格就是**系统服务描述符表**。在 Linux 上，这个表格的名字叫 **sys\_call\_table**，在 Windows 上，它叫 **KeServiceDescriptorTable**，简称 **SSDT**。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLpwvkomsm1lYwssicpgu9viaiaicOsF0VzgQfJaaNSq9qOTSgjx8E8olPiaQ/640?wx_fmt=png)

Windows 上的 SSDT 向来是兵家必争之地，安全软件为了监控应用程序的行为，通常都会替换 SSDT 表格中的系统服务函数地址为它们的函数。当系统调用触发时，安全软件将会及时知晓，并通过应用程序的参数来判定是否 “放行” 这次调用。

IDT HOOK
--------

内核中除了记录系统服务的 SSDT，还有一个非常重要的表格：**中断描述符表 IDT**。

IDT 用于记录 CPU 执行过程中遇到**中断**、**异常**等情况时，该转向哪里去处理这些情况的函数地址。

HOOK IDT 有一个注意事项，不同于 SSDT 是全局唯一的，IDT 是与 CPU 核心紧密相关的，对于多核处理器，会对应多个 IDT 表。如果想通过 HOOK IDT 中的函数来搞事情的话，可能需要同时处理多个表。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLGR4NocJwbzSGFBZJzia888oFCka4AWN7tmjYibibibZf9kHrdDUZ7N1AZg/640?wx_fmt=png)

除了直接修改函数指令和修改函数指针之外，还有一类特殊的 HOOK，它们通过系统提供的机制拦截某些消息、通知，从而有机会介入监听、拦截。

IRP HOOK
--------

在 Windows 系统上，用户程序和内核驱动之间的交互是通过一种称为 **IRP** 的数据结构实现的，你可以简单将其理解为应用程序发送了一个消息下去，这个消息就是一个 IRP。

而接收消息的目标，是驱动程序创建的**设备 Device**。注意，这个设备不一定是物理设备，也可能完全不存在的虚拟设备，驱动程序可以任意创建一个不存在的设备。

Windows 内核中提供了驱动设备的挂载操作，允许别的驱动程序对指定设备进行挂载，从而可以截获发送给该设备的 “消息”，这种 HOOK 方式被称为 **IRP HOOK**。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhdfjTIGNbo4B4tvY9XASZibLJHTJIcBavTtcrof5Mx3YrxK1co3MoRsC7InHhqviaxIohuVeSbU1dLw/640?wx_fmt=png)

国内一些安全软件为了互相攻击，经常用这种方式拦截对方驱动程序的消息，从而可以 “保护” 自己不被对方干掉。

TDI HOOK && NDIS HOOK
---------------------

这两种 HOOK 方式与 Windows 内核中的网络子系统密切相关。

Windows 内核中的网络结构是分层式设计。从最上层的 API socket 层、到 TCP/IP 协议栈层、再到底层的网卡驱动程序，分了很多个层次。

而层与层之间的交互，是通过一系列标准接口来实现的，其中最重要的两个接口标准就是 **TDI** 和 **NDIS**。TDI 封装了不同协议栈的差异（Windows 不止支持 TCP/IP 协议栈）提供给上层统一的调用接口。NDIS 则封装了底层不同网卡的驱动程序接口差异，提供给上层统一的收发数据包接口。

Windows 为了扩展性支持，允许类似防火墙之类的软件通过这些接口接入，从而能够截获到网络通信流量，进行安全审计。

既然开了这些接口，一些流氓软件和木马病毒也就盯上了它们，通过这些接口就能轻松监听、篡改网络数据，达到邪恶的目的。

Windows Message HOOK
--------------------

Windows 操作系统的 UI 交互是以**消息**来驱动的，用户的键盘输入、鼠标操作都会被操作系统以消息的形式发送到各个应用程序处理。

Windows 提供了 API 接口，可以被程序用于捕获这些消息，从而实现一些特定的功能。

```
HHOOK SetWindowsHookEx(  int       idHook,  HOOKPROC  lpfn,  HINSTANCE hmod,  DWORD     dwThreadId);
```

这种机制叫做 Windows 消息钩子，最常见的就要数键盘钩子了，在十多年前流氓软件和木马病毒大行其道的时候，这些恶意软件经常喜欢通过这种方式来**监听**用户的键盘输入，从而来盗取 QQ 密码（当然，现在肯定是不行的了）。

总结
--

以上就是要介绍的全部 HOOK 技术了。当然有 HOOK，就有反 HOOK，很多安全软件都会检查关键的位置是否被篡改。不仅如此，因为流氓软件随意修改系统，Windows 从 Win7 x64 开始加入了 **PatchGuard** 机制，针对操作系统核心数据结构都加入了定时检测机制，一旦发现被篡改，立刻蓝屏给你看，而且在随着系统升级换代，这个检查的粒度和强度变得越来越强。

最后来回到文章开头的几个问题：

**病毒木马为何惨遭杀软拦截？**

因为安全软件在内核中 HOOK 了大量的关键位置，病毒木马的进程、文件、网络行为都将受到监控，一举一动都难逃杀软的眼睛，想要拦截易如反掌。

**商业软件为何频遭免费破解？**

通过逆向分析加上 Inline HOOK，破解者可以篡改掉商业软件的注册校验机制，让校验函数返回成功，绕开软件的限制。

**系统漏洞为何能被补丁修复？**

统一通过 Inline HOOK，操作系统能够修改原来有 bug 的代码，转而执行修复后的新版本，解决系统漏洞。

**你看，技术就是一柄双刃剑，善或恶，一念之间。**

**推荐↓↓↓**

![](https://mmbiz.qpic.cn/mmbiz_jpg/NVvB3l3e9aG5kWic5P8XOwFOhXKjibAt6Yfb1QuqSRZaV5QGHtqqXZFWkia50TDjpWTBqG8Huj3aMlA6cOE9cBVkQ/640?wx_fmt=jpeg)

**Linux 学习**