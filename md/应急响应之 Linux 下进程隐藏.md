> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9Dga9CZZnfesSmSpMVqWNw)

**概述**

当黑客获取系统 root 权限时，为了实现持久化控制往往会创建隐藏恶意进程，这给应急响应人员取证的时候带来了难度，隐藏进程的方法分为两类，一类是用户态隐藏，另一类是内核态隐藏。用户态常使用的方法有很多，例如劫持预加载动态链接库，一般通过设置环境变量 LD_PRELOAD 或者 /etc/ld.so.preload，过滤 /proc/pid 目录、修改进程 PID 等等。内核态隐藏进程一般是加载恶意的内核模块实现进程隐藏，本文抛砖引玉，介绍应急响应场景中遇到过的 Linux 操作系统进程隐藏的手段以及检测方法。

**劫持预加载动态链接库 LD_PRELOAD**

查看 Linux 操作系统正在运行的进程，一般会使用系统命令 ps、top 等，像 ps 这样的命令通常是读取了 /proc/ 目录下文件。Linux 操作系统上的 /proc 目录存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关操作系统硬件和当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。/proc 目录中包含许多以数字命名的子目录，这些数字代表操作系统当前正在运行进程的进程号（pid），每个数字文件夹里面包含对应进程的多个信息文件。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tae2rbUMejXByhMqcjVskdaEiaYr3bSdOThsR8QJNXYNFPaia6qNZ40JMQ/640?wx_fmt=png)

LD_PRELOAD 是 Linux 操作系统的一个环境变量，它允许定义在程序运行前优先加载的动态链接库，设置完成后立即生效。劫持预加载动态链接库的进程隐藏方式往往是过滤 ps 等命令从 /proc/ 获取的结果，而不是针对 /proc/ 文件系统生成本身。在应急的时候一般可以通过 strace 命令调试 ps 命令的所有系统调用以及这个进程所接收到的所有的信号量。当系统未设置了 ld.so.preload，ps 命令读取 /etc/ld.so.preload，返回值为 - 1，说明文件不存在。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tahPgjnUB6tnib4iaib64iaEwKCsLGicMAPOSxj0rl1bqwkb8fs0PMdQoCZYQ/640?wx_fmt=png)

当系统设置了 ld.so.preload，ps 命令读取 /etc/ld.so.preload，返回值为 0，说明文件存在。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taDdibnbhAvr4yr8icibT7N3wia7kIib2T1UABWvcBUdy2JB9CpK0vnGGEibiaQ/640?wx_fmt=png)

比如，我们要隐藏进程 threat.py，可以借助 libprocesshider 项目，通过修改 static const char* processtofilter = "threat.py"，隐藏指定进程。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taZ53YDFCCUGB3ZBQicYusQXAicRqCg2fC872lkmQGg7pySjIVKIThUI3w/640?wx_fmt=png)

编译后将生成的 .so 文件路径写入 /etc/ld.so.preload。运行该进程后可以看到 ps 命令未检测到隐藏进程。但是使用 busybox 可以看到隐藏进程的相关信息，那是因为 busybox ps 命令直接读取了 proc 目录的数字，不调用系统预加载库。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taGBfJ3Ex4k6Tt2qibrGkbHtSyDAu8ia37bWka59RUXhegup7ibia6DhZ7ZQ/640?wx_fmt=png)

测试使用的脚本名为 processhider.c，下载地址如下：

https://github.com/gianlucaborello/libprocesshider/。

**劫持预加载动态链接库 LD_AUDIT**

上面介绍了劫持 LDPRELOAD 隐藏进程，黑客往往常用这个技术拦截系统调用执行恶意代码。正如文档 ld.so 中内容，LDPRELOAD 在所有其他对象（附加的、用户指定、ELF 共享对象）之前加载，但实际上 LDPRELOAD 并非真的是首先加载，通过利用 LDAUDIT 环境变量可以实现优先于 LD_PRELOAD 加载。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2ta8syD6A2Khqh64FcbibEvXe7E7Tjjgiaa0Gsia6mBGq2vwz0zZfic3ibMn8w/640?wx_fmt=png)

首先我们验证下 LDPRELOAD、LDAUDIT 的加载顺序。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2ta60m7j4ewzXSTFia3V2PJE7IQYBB50H9kroCTK3e1iagCZqxreqG75nmg/640?wx_fmt=png)

编译 preloadlib.c、auditlib.c。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taMBAdhoO2qTH0bXNiawBwfjEMgqPicyfBj4EvLCEBJd7HAGWcXrh6Ufbw/640?wx_fmt=png)

从执行 whoami 的结果可以看到 LDAUDIT 优先于 LDPRELOAD。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tabqxWojlbKfXibfM1wSLBE1wuBCGwRicMbOWiboWayjthH95ycNShh4iamw/640?wx_fmt=png)

还以 libprocesshider 项目为例，我们想要隐藏运行的脚本 threat.py，如果直接编译使用 LD_AUDIT 加载 so 文件，可以发现并没有隐藏进程。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2ta5OxIYsADPBUZcic9BthYpNIEiaBGqffZUzIxyjerxEPWey7QZ9tF8ib9g/640?wx_fmt=png)

通过查询 rtld-audit (https://man7.org/linux/man-pages/man7/rtld-audit.7.html) 文档，可以看到调用该库需要两个函数 laobjopen 和 lasymbind64，当加载器找到并加载一个库时，将调用 rtld-audit 中的 laobjopen 函数，struct linkmap 指向要加载的库，cookie 声明一个指针指向该对象标识符，并传给 lasymbind64，lasymbind64 函数不仅可以提供信息，还可以修改程序行为。故可以在 libprocesshider.c 追加以下代码：

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2ta9npthAS9XZztV3jibicGAkiaWyLT97aAhdwicaQicI7SMO0HpEAicCfsJBYQ/640?wx_fmt=png)

重新编译后，可以看已经隐藏进程。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taYqJ7eI3KdKMp4BoHwrhsEvNluRiaq5mwAlQC5hLketGEicEysXlmI2Ww/640?wx_fmt=png)

  
**修改进程 pid**

通过查看 linux 内核源码，在 include/linux/threads.h 文件中可以看到 pid 最大值的为变量 PIDMAXDEFAULT，相关代码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tawvF70UibKiaCjItjP8icOB7qhtdB9UnxR4AcIOpoKZ965qg1X4pBBc83Q/640?wx_fmt=png)

如果编译内核时设置了 CONFIGBASESMALL 选项，则 pid 的最大值是 0x1000，即 4096 个，否则最大值是 0x8000，即 32768 个。pid 的最大值是可以修改的，但是可以修改的最大值是多少，这个是通过 PIDMAXLIMIT 限定的，从代码可知，如果编译内核时设置了 CONFIGBASESMALL 选项，则最大值就是  8 * PAGESIZE 个大小，否则就看 long 的大小，如果大于 4，也就是最大可以设置 4*1024*1024 个，也即是 4194304 个，否则最大只能设置 PIDMAX_DEFAULT 个了。

用户可以通过查看 /proc/sys/kernel/pid_max 文件获取当前操作系统的 pid 的最大值，例如 centos7 默认可以有 131072 个 pid。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tabBJ2autyDC0WqsVDjbLtMvOM93HgHykUTYdQELZL2Zua4ZR5Y3YNbw/640?wx_fmt=png)

当前进程创建后，获取其 pid 注册 proc 目录下，然后遍历 tasklist 以其 pid 作为主键来显示 proc 目录。从上面知道 centos7 下最多有 131072 个 pid 号。故我们通过内核模块修改进程 pid，然后用户空间由于权限的限制无法读取内核配置的 pid 外的数据实现隐藏。代码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taMibdprpQ3lEbSPLscZDSTiadjF2NQepU8twVT5LLYVz6kUogurp6CCiaw/640?wx_fmt=png)

执行恶意进程，获取其进程 pid 为 2895，运行脚本后可以看到进程已经被隐藏了。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tawkSV6Jx4S6lvjNyicB4V0rsYhhgJcSrAQG5ZlicdSzJGRTSkxTWZvdnA/640?wx_fmt=png)

安装 stap 后默认会在路径 /usr/share/systemtap/examples/ 下存储一些非常实用的脚本。例如 network- 目录下脚本主要查看系统中每个进程的网络传输情况，io 目录下的脚本主要查看进程对磁盘的读写情况。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2ta0icSzRSeHXEicr2sibTm9MR9zp6Uc4MH6p5RLTEWbkLqGXLYTP3UiaXiaCQ/640?wx_fmt=png)

虽然隐藏了进程，但是还存在网络连接。故可以通过 network 目录的脚本从内核层面检测当前操作系统的网络连接获取隐藏进程。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taHvwFC4YHDOicWLQKevlmibVEpRkXuY0qc4gwl03N5Q2yoqt00avich1Tw/640?wx_fmt=png)

另外大多数情况下，我们只使用 kill 命令来杀死一个进程，实际上 kill 命令只是向进程发送一个信号，可以到 https://github.com/torvalds/linux/blob/master/arch/x86/include/uapi/asm/signal.h 查看每个信号对应的功能，信号 20 表示停止进程的运行, 但该信号可以被处理和忽略。在 Linux 下，我们 kill 一个不存在的进程会返 "No such process"，kill 一个存在的进程返回结果为空，故通过信号也可以获取隐藏进程 pid。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2taptzXmIqtM8icIgY2mayiaGRUMW0HCzicflmQK3JcbUzKVnenkwYcuv2Rw/640?wx_fmt=png)

**伪造内核模块隐藏进程**

在 Linux 上，有许多内核进程被创建来帮助完成系统任务。这些进程可用于调度、磁盘 I/O 等。当使用像 ps 之类的命令显示当前运行的进程时，内核进程的周围有 [括号]，普通进程通常不会显示带方括号的进程。Linux 恶意软件使用各种技术来隐藏检测。其中一种为让进程名显示 [] 来模拟内核线程。

(1) /proc/maps 通常用来查看进程的虚拟地址空间是如何使用的。正常的内核进程 maps 内容为空的，伪装的内核进程是有内容标识的。如下 pid 为 2120 为系统正常的内核进程，pid 为 3195 是伪造的内核进程。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tafTzHTduvzk4aZnzaXkqichWqrcq5wPlicd5eEFbWF6G1RrnTYDQAC6PQ/640?wx_fmt=png)

(2) /proc/exe 指向运行进程的二进制程序链接，正常内核进程没有相应的二进制文件，恶意的内核进程有对应的二进制文件。

![](https://mmbiz.qpic.cn/mmbiz_png/pOGBCic4vYicZlFpFtS6WK8H6HicagYf2tagTdmo4iajS2CHaXibpyCbDZbJdZewGHQ2W9HBap7f2IzV6WePiacYZz9Q/640?wx_fmt=png)

**总结**

安全的本质是对抗，只有熟悉了黑灰产常用的攻击方式，才能做到遇事不慌。Linux 下进程隐藏的方式远远不止以上几种，笔者只是介绍了在应急响应场景下遇到的一些情况，想要了解更多，且听下回分解。
==============================================================================================

  

**参考链接**

https://github.com/gianlucaborello/libprocesshider
==================================================

https://github.com/torvalds/linux/blob/master/arch/x86/include/uapi/asm/signal.h

**━ ━ ━ ━ ━**

  

公众号内回复 “**隐藏**”，可获取 PDF 版报告。

  

**关于微步在线研究响应团队**

微步情报局，即微步在线研究响应团队，负责微步在线安全分析与安全服务业务，主要研究内容包括威胁情报自动化研发、高级 APT 组织 & 黑产研究与追踪、恶意代码与自动化分析技术、重大事件应急响应等。

微步情报局由精通木马分析与取证技术、Web 攻击技术、溯源技术、大数据、AI 等安全技术的资深专家组成，并通过自动化情报生产系统、云沙箱、黑客画像系统、威胁狩猎系统、追踪溯源系统、威胁感知系统、大数据关联知识图谱等自主研发的系统，对微步在线每天新增的百万级样本文件、千万级 URL、PDNS、Whois 数据进行实时的自动化分析、同源分析及大数据关联分析。微步情报局自设立以来，累计率先发现了包括数十个境外高级 APT 组织针对我国关键基础设施和金融、能源、政府、高科技等行业的定向攻击行动，协助数百家各个行业头部客户处置了肆虐全球的 WannaCry 勒索事件、BlackTech 定向攻击我国证券和高科技事件、海莲花长期定向攻击我国海事 / 高科技 / 金融的攻击活动、OldFox 定向攻击全国上百家手机行业相关企业的事件。

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/pOGBCic4vYicZnSxLEdVd7S098eib2I4wib6QibO5sfnlXUvvTPXhSQwlQ2bHwYiab3dUvkyzjRHaZm2xXmydX9BibXbA/640?wx_fmt=jpeg)

**微步在线**

**研究响应中心**

- 长按二维码关注我们 -