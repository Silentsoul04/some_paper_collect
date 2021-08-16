> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JMI0WZ2i4f-hccxSpj22BQ)

![](https://mmbiz.qpic.cn/mmbiz_jpg/A1HKVXsfHNkvKNOtZ7lQhSwlnL7vKaZJByBGwXEgAHeicJxcKYiacyWhUicOVHxiaxBTO5L8yiaBV4ibroISsiccicJ16Q/640?wx_fmt=jpeg)

本指南旨在说明如何尽可能地加强 Linux 的安全性和隐私性，并且不限于任何特定的指南。

  

免责声明：如果您不确定自己在做什么，请不要尝试在本文中使用任何内容。

  

本指南仅关注安全性和隐私性，而不关注性能，可用性或其他内容。列出的所有命令都将需要 root 特权。以 “$” 符号开头的单词表示一个变量，不同终端之间可能会有所不同。

选择正确的 Linux 发行版

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

选择一个好的 Linux 发行版有很多因素。

  

*   避免分发冻结程序包，因为它们在安全更新中通常很落后。
    
*   不使用与 Systemd 机制的发行版。Systemd 包含许多不必要的攻击面；它尝试做的事情远远超出了必要，并且超出了初始化系统应做的事情。
    
*   使用 musl 作为默认的 C 库。Musl 专注于最小化，这会导致很小的攻击面，而其他 C 库（例如 glibc）过于复杂，容易产生漏洞。例如，与 musl 中的极少数漏洞相比，glibc 中的一百多个漏洞已被公开披露。尽管仅靠披露的 CVE 本身通常是不准确的统计信息，但有时这种情况有时可以用来表示过分的问题。Musl 还具有不错的漏洞利用缓解措施，尤其是其新的强化内存分配器。
    
*   最好默认情况下使用 LibreSSL 而不是 OpenSSL 的发行版。OpenSSL 包含大量完全不必要的攻击面，并且遵循不良的安全做法。例如，它仍然保持 OS / 2 和 VMS 支持这些已有数十年历史的古老操作系统。这些令人讨厌的安全做法导致了可怕的 Heartbleed 漏洞。LibreSSL 是 OpenBSD 团队的 OpenSSL 分支，它采用了出色的编程实践并消除了很多攻击面。在 LibreSSL 成立的第一年内，它缓解了许多漏洞，其中包括一些高严重性的漏洞。
    

  

用作强化操作系统基础的最佳发行版是 Gentoo Linux，因为它可以让您精确地配置系统，以达到理想的效果，这将非常有用，尤其是参考我们在后面的章节中使用更安全的编译标志。

  

但是，由于 Gentoo 的巨大可用性缺陷，它对于许多人来说可能并不顺手。在这种情况下，Void Linux 的 Musl 构建是一个很好的折衷方案。

内核

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

内核是操作系统的核心，不幸的是很容易受到攻击。正如 Brad Spengler 曾经说过的那样，可以将其视为系统上最大，最易受攻击的 setuid 根二进制文件。因此，对内核进行尽可能多的强化非常重要。

  

**Stable vs LTS 内核**

  

Linux 内核以两种主要形式发布：稳定和长期支持（LTS）。稳定版本是较新的版本，而 LTS 发行版本是较老的稳定版本，长期以来一直受支持。选择上述任何一个发行版本都有许多后果。

  

Linux 内核未使用 CVE 标识安全漏洞。这意味着大多数安全漏洞的修复程序不能向后移植到 LTS 内核。但是稳定版本包含到目前为止进行的所有安全修复。

  

但是，有了这些修复程序，稳定的内核将包含更多新功能，因此大大增加了内核的攻击面，并引入了大量新错误。相反，LTS 内核的受攻击面较小，因为这些功能没有被不断添加。

  

此外，稳定的内核还包括更新的强化功能，以减轻 LTS 内核没有的某些利用。此类功能的一些示例是 Lockdown LSM 和 STACKLEAK GCC 插件。

  

总而言之，在选择稳定或 LTS 内核时需要权衡取舍。LTS 内核具有较少的强化功能，并且并非当时所有的公共错误修复都已向后移植，但是通常它的攻击面更少，并且引入未知错误的可能性也较小。稳定的内核具有更多的强化功能，并且包括所有已知的错误修复，但它也具有更多的攻击面以及引入更多未知错误的机会更大。最后，最好使用较新的 LTS 分支（如 4.19 内核）。

  

**Sysctl**

  

Sysctl 是允许用户配置某些内核设置并启用各种安全功能或禁用危险功能以减少攻击面的工具。要临时更改设置，您可以执行：

```
sysctl -w $tunable = $value
```

要永久更改 sysctls，您可以将要更改的 sysctls 添加到 / etc/sysctl.conf 或 / etc/sysctl.d 中的相应文件，具体取决于您的 Linux 发行版。

  

以下是您应更改的建议 sysctl 设置。

  

**Kernel self-protection**

  

```
kernel.kptr_restrict=2
```

内核指针指向内核内存中的特定位置。这些在利用内核方面可能非常有用，但是默认情况下不会隐藏内核指针，例如，通过读取 / proc/kallsyms 的内容即可轻松发现它们。此设置旨在减轻内核指针泄漏。另外，您可以设置 kernel.kptr_restrict = 1 以仅从没有 CAP_SYSLOG 功能的进程中隐藏内核指针。

```
kernel.dmesg_restrict=1
```

dmesg 是内核日志，它公开了大量有用的内核调试信息，但这通常会泄漏敏感信息，例如内核指针。更改上述 sysctl 设置会将内核日志限制为 CAP_SYSLOG 功能。

```
kernel.printk=3 3 3 3
```

尽管 dmesg_restrict 的值，启动过程中内核日志仍将显示在控制台中。能够在引导过程中记录屏幕的恶意软件可能会滥用此恶意软件以获得更高的特权。此选项可防止这些信息泄漏。必须将其与下面描述的某些引导参数结合使用才能完全有效。

```
kernel.unprivileged_bpf_disabled=1  
net.core.bpf_jit_harden=2
```

eBPF 暴露了很大的攻击面，因此需加以限制。这些系统将 eBPF 限制为 CAP_BPF 功能（在 5.8 之前的内核版本上为 CAP_SYS_ADMIN），并启用 JIT 强化技术，例如常量绑定。

```
dev.tty.ldisc_autoload=0
```

这将加载 TTY 行规则限制为 CAP_SYS_MODULE 功能，以防止非特权的攻击者使用 TIOCSETD ioctl 加载易受攻击的线路规则，而该 TIOCSETD ioctl 之前已在许多漏洞利用中被滥用。

```
vm.unprivileged_userfaultfd=0
```

userfaultfd() 系统调用经常被滥用以利用 “事后使用(use-after-free)” 缺陷。因此，该 sysctl 用于将此 syscall 限制为 CAP_SYS_PTRACE 功能。

```
kernel.kexec_load_disabled=1
```

kexec 是一个系统调用，用于在运行时引导另一个内核。可以滥用此功能来加载恶意内核并在内核模式下获得任意代码执行能力，因此该 sysctl 设置将被禁用。

```
kernel.sysrq=4
```

SysRq 密钥向非特权用户公开了许多潜在的危险调试功能。与通常的假设相反，SysRq 不仅是物理攻击的问题，而且还可以远程触发。该 sysctl 的值使其可以使用户只能使用 SAK 密钥，这对于安全地访问 root 是必不可少的。或者，您可以简单地将值设置为 0 以完全禁用 SysRq。

```
kernel.unprivileged_userns_clone=0
```

用户名称空间是内核中的一项功能，旨在改善沙箱并使非特权用户易于访问它，但是，此功能公开了重要的内核攻击面，以进行特权升级，因此该 sysctl 将用户名称空间的使用限制为 CAP_SYS_ADMIN 功能。对于无特权的沙箱，建议使用具有很少攻击面的 setuid 二进制文件，以最大程度地减少特权升级的可能性。沙箱章节部分将进一步讨论此主题。

  

请注意，尽管该 sysctl 仅在某些 Linux 发行版中存在，因为它需要内核补丁。如果您的内核不包含此补丁，则可以通过设置 user.max_user_namespaces = 0 来完全禁用用户名称空间（包括 root 用户）。

```
kernel.perf_event_paranoid=3
```

性能事件会增加大量内核攻击面，并导致大量漏洞。此 sysctl 设置将性能事件的所有使用限制为 CAP_PERFMON 功能（5.8 之前的内核版本为 CAP_SYS_ADMIN）。

  

请注意，此 sysctl 设置需要在某些发行版中具备相关的内核补丁。否则，此设置等效于 kernel.perf_event_paranoid = 2，它仅限制此功能的子集。

  

**网络**

  

```
net.ipv4.tcp_syncookies=1
```

这有助于防止 SYN 泛洪攻击，这种攻击是拒绝服务攻击的一种形式，在这种攻击中，攻击者发送大量虚假的 SYN 请求，以尝试消耗足够的资源以使系统对合法流量不响应。

```
net.ipv4.tcp_rfc1337=1
```

这通过丢弃处于时间等待状态的套接字的 RST 数据包来防止 time-wait 状态。

```
net.ipv4.conf.all.rp_filter=1  
net.ipv4.conf.default.rp_filter=1
```

这些启用了源验证，以验证从计算机所有网络接口接收到的数据包。

```
net.ipv4.conf.all.accept_redirects=0  
net.ipv4.conf.default.accept_redirects=0  
net.ipv4.conf.all.secure_redirects=0  
net.ipv4.conf.default.secure_redirects=0  
net.ipv6.conf.all.accept_redirects=0  
net.ipv6.conf.default.accept_redirects=0  
net.ipv4.conf.all.send_redirects=0  
net.ipv4.conf.default.send_redirects=0
```

这些设置禁用了 ICMP 重定向，以防止中间人攻击并最大程度地减少信息泄露。

```
net.ipv4.icmp_echo_ignore_all=1
```

此设置使您的系统忽略所有 ICMP 请求，以避免 Smurf 攻击，使设备更难以在网络上枚举，并防止通过 ICMP 时间戳识别时钟指纹。

```
net.ipv4.conf.all.accept_source_route=0  
net.ipv4.conf.default.accept_source_route=0  
net.ipv6.conf.all.accept_source_route=0  
net.ipv6.conf.default.accept_source_route=0
```

源路由是一种允许用户重定向网络流量的机制。由于这可用于执行中间人攻击，在中间人攻击中，出于恶意目的将流量重定向，因此上述设置将会禁用此功能。

```
net.ipv6.conf.all.accept_ra=0  
net.ipv6.conf.default.accept_ra=0
```

恶意的 IPv6 路由广告可能会导致中间人攻击，因此应将其禁用。

```
net.ipv4.tcp_sack=0  
net.ipv4.tcp_dsack=0  
net.ipv4.tcp_fack=0
```

禁用 TCP SACK。ACK 通常被利用，并且在许多情况下是不必要的，因此如果您不需要它，则应将其禁用。

  

**用户空间**

  

```
kernel.yama.ptrace_scope=2
```

ptrace 是一个系统调用，它允许程序调试、修改和检查另一个正在运行的进程，从而使攻击者可以轻易修改其他正在运行的程序的内存。设置将 ptrace 的使用限制为仅具有 CAP_SYS_PTRACE 功能的进程。或者，将 sysctl 设置为 3 以完全禁用 ptrace。

```
vm.mmap_rnd_bits=32  
vm.mmap_rnd_compat_bits=16
```

ASLR 是一种常见的漏洞利用缓解措施，它可以使进程的关键部分在内存中的位置随机化。这可能会使各种各样的漏洞利用更困难，因为它们首先需要信息泄漏。上述设置增加了用于 mmap ASLR 的熵的位数，从而提高了其有效性。

  

这些 sysctls 的值必须根据 CPU 体系结构进行设置。以上值与 x86 兼容，但其他体系结构可能有所不同。

```
fs.protected_symlinks=1  
fs.protected_hardlinks=1
```

仅当在可全局写入的粘性目录之外，当符号链接和关注者的所有者匹配或目录所有者与符号链接的所有者匹配时，才允许遵循符号链接。这还可以防止没有对源文件的读 / 写访问权限的用户创建硬链接。这两者都阻止了许多常见的 TOCTOU 漏洞（time-of-check-to-time-of-use）。

```
fs.protected_fifos=2  
fs.protected_regular=2
```

这些阻止了在可能由攻击者控制的环境（例如，全局可写目录）中创建文件，从而使数据欺骗攻击更加困难。

  

**引导参数**

  

引导参数在引导时使用引导加载程序（bootloader）将设置传递给内核。类似于 sysctl，可以使用某些设置来提高安全性。引导加载程序通常在引导参数设置方式上有所不同。下面列出了一些示例，但是您应该研究特定 bootloader 的修改参数的必要步骤。

  

如果使用 GRUB 作为引导程序，请编辑 / etc /default/grub 并将参数添加到 GRUB_CMDLINE_LINUX_DEFAULT=line。

  

如果使用 Syslinux，请编辑 / boot/syslinux/syslinux.cfg 并将它们添加到 APPEND 行中。

  

如果使用 systemd-boot，请编辑您的加载程序条目，并将其附加到 linux 行的末尾。

  

建议使用以下设置以提高安全性。

  

**Kernel self-protection**

  

```
slab_nomerge
```

这将禁用 slab 合并，这将通过防止覆盖合并的缓存中的对象并使其更难以影响 slab 缓存的布局，从而大大增加了堆利用的难度。

```
slub_debug=FZ
```

这些启用健全性检查（F）和重新分区（Z）。健全性检查会添加各种检查，以防止某些 slab 操作中的损坏。重新分区会在 slab 周围添加额外的区域，以检测 slab 何时被覆盖超过其实际大小，从而有助于检测溢出。

```
init_on_alloc=1 init_on_free=1
```

这样可以在分配和空闲时间期间将内存清零，这可以帮助减轻使用后使用的漏洞并清除内存中的敏感信息。如果您的内核版本低于 5.3，则这些选项不存在。而是在上述 slub_debug 选项后面附加 “P”，以获得 slub_debug=FZP 并添加 page_poison=1。由于它们实际上是一种调试功能，刚好具有一些安全性，因此它们在释放时提供的内存擦除形式较弱。

```
page_alloc.shuffle=1
```

此选项使页分配器空闲列表随机化，从而通过降低页分配的可预测性来提高安全性，同时这也提高了性能。

```
pti=on
```

这将启用内核页表隔离，从而减轻崩溃并防止某些 KASLR 绕过。

```
vsyscall=none
```

这将禁用 vsyscall，因为它们已过时且已被 vDSO 取代。vsyscall 也在内存中的固定地址上，使其成为 ROP 攻击的潜在目标。

```
debugfs=off
```

这将禁用 debugfs，它会公开许多有关内核的敏感信息。

```
oops=panic
```

有时某些内核漏洞利用会导致所谓的 “oops”。此参数将引发内核对此类事件 panic，从而防止这些攻击。但是，有时错误的驱动程序会导致无害的操作，这会导致系统崩溃，这意味着此引导参数只能在某些硬件上使用。

```
module.sig_enforce=1
```

这仅允许加载已使用有效密钥签名的内核模块，使加载恶意内核模块更加困难。

  

这可以防止加载所有树外内核模块（包括 DKMS 模块），除非您已对其进行签名，这意味着诸如 VirtualBox 或 Nvidia 驱动程序之类的模块可能不可用，但根据您的设置可能并不重要。

```
lockdown=confidentiality
```

内核锁定 LSM 可以消除用户空间代码滥用以升级为内核特权并提取敏感信息的许多方法。为了在用户空间和内核之间实现清晰的安全边界，此 LSM 是必需的。上面的选项在 confidentiality 模式（最严格的选项）中启用此功能。这意味着 module.sig_enforce=1。

```
mce=0
```

这将导致内核对 ECC 内存中无法利用的错误 panic，而这些错误可能会被利用。对于没有 ECC 内存的系统，这是不必要的。

```
quiet loglevel=0
```

这些参数可防止引导期间信息泄漏，并且必须与上面的 kernel.printk sysctl 结合使用。

  

**CPU 缓解**

  

最好启用适用于您的 CPU 的所有 CPU 缓解措施，以确保您不受已知漏洞的影响。这是启用所有内置缓解措施的列表：

```
spectre_v2=on spec_store_bypass_disable=on tsx=off tsx_async_abort=full,nosmt mds=full,nosmt l1tf=full,force nosmt=force kvm.nx_huge_pages=force
```

您必须研究系统受其影响的 CPU 漏洞，并相应地选择上述缓解措施。请记住，您将需要安装微代码更新，以完全免受这些漏洞的影响。但所有这些操作都可能导致性能显着下降。

  

**结果**

  

如果遵循了以上所有建议（不包括特定的 CPU 缓解措施），则将具有：

```
slab_nomerge slub_debug=FZ init_on_alloc=1 init_on_free=1 page_alloc.shuffle=1 pti=on vsyscall=none debugfs=off oops=panic module.sig_enforce=1 lockdown=confidentiality mce=0 quiet loglevel=0
```

如果将 GRUB 用作引导加载程序，则可能需要重新生成 GRUB 配置文件才能应用这些文件。

  

**hidepid**

  

/proc 是一个伪文件系统，其中包含有关系统上当前正在运行的所有进程的信息。默认情况下，所有用户都可以访问此程序，这可能使攻击者可以窥探其他进程。要只允许用户看到自己的进程，而不能看到其他用户的进程，则必须使用 hidepid=2，gid=proc 挂载选项来挂载 / proc。gid=proc 将 proc 组从此功能中排除，因此您可以将特定的用户或进程列入白名单。添加这些选项的一种方法是编辑 / etc/fstab 并添加：

```
proc /proc proc nosuid,nodev,noexec,hidepid=2,gid=proc 0 0
```

systemd-logind 仍然需要查看其他用户的进程，因此，要使用户会话在 systemd 系统上正常工作，必须创建 / etc/systemd/system/systemd-logind.service.d/hidepid.conf 并添加：

```
[Service]  
SupplementaryGroups=proc
```

  

**减少内核攻击面**

  

最好禁用不是绝对必要的任何功能，以最大程度地减少潜在的内核攻击面。这些功能不必一定很危险，它们可以只是被删除以减少攻击面的良性代码。切勿禁用您不了解的随机事物。以下是一些可能有用的示例，具体取决于您的设置。

  

**引导参数**

  

引导参数通常可以用来减少攻击面，这样的例子之一是：

```
ipv6.disable=1
```

这将禁用整个 IPv6 堆栈，如果您尚未迁移到该堆栈，则可能不需要该堆栈。如果正在使用的 IPv6，请不要使用此引导参数。

  

**将内核模块列入黑名单**

  

内核允许非特权的用户通过模块自动加载来间接导致某些模块被加载。这使攻击者可以自动加载易受攻击的模块，然后加以利用。一个这样的示例是 CVE-2017-6074，其中攻击者可以通过启动 DCCP 连接来触发 DCCP 内核模块的加载，然后利用该内核模块中的漏洞。

  

可以通过将文件插入 / etc/modprobe.d 并将指定的内核模块列入黑名单的方法，将特定的内核模块列入黑名单。

  

Install 参数告诉 modprobe 运行特定命令，而不是像往常一样加载模块。/bin/false 是仅返回 1 的命令，该命令实际上不会执行任何操作。两者都告诉内核运行 / bin/false 而不是加载模块，这将防止攻击者利用该模块。以下是最有可能不需要的内核模块：

```
install dccp /bin/false  
install sctp /bin/false  
install rds /bin/false  
install tipc /bin/false  
install n-hdlc /bin/false  
install ax25 /bin/false  
install netrom /bin/false  
install x25 /bin/false  
install rose /bin/false  
install decnet /bin/false  
install econet /bin/false  
install af_802154 /bin/false  
install ipx /bin/false  
install appletalk /bin/false  
install psnap /bin/false  
install p8023 /bin/false  
install p8022 /bin/false  
install can /bin/false  
install atm /bin/false
```

特别是模糊的网络协议会增加大量的远程攻击面。此黑名单：

  

*   DCCP — Datagram Congestion Control Protocol
    
*   SCTP — Stream Control Transmission Protocol
    
*   RDS — Reliable Datagram Sockets
    
*   TIPC — Transparent Inter-process Communication
    
*   HDLC — High-Level Data Link Control
    
*   AX25 — Amateur X.25
    
*   NetRom
    
*   X25
    
*   ROSE
    
*   DECnet
    
*   Econet
    
*   af_802154 — IEEE 802.15.4
    
*   IPX — Internetwork Packet Exchange
    
*   AppleTalk
    
*   PSNAP — Subnetwork Access Protocol
    
*   p8023 — Novell raw IEEE 802.3
    
*   p8022 — IEEE 802.2
    
*   CAN — Controller Area Network
    
*   ATM
    

```
install cramfs /bin/false  
install freevxfs /bin/false  
install jffs2 /bin/false  
install hfs /bin/false  
install hfsplus /bin/false  
install squashfs /bin/false  
install udf /bin/false
```

将各种稀有文件系统列入黑名单。

```
install cifs /bin/true  
install nfs /bin/true  
install nfsv3 /bin/true  
install nfsv4 /bin/true  
install gfs2 /bin/true
```

如果不使用网络文件系统，也可以将其列入黑名单。

```
install vivid /bin/false
```

vivid driver 驱动程序仅用于测试目的，并且是特权提升漏洞的原因，因此应禁用它。

```
install bluetooth /bin/false  
install btusb /bin/false
```

禁用具有安全问题历史记录的蓝牙。

```
install uvcvideo /bin/false
```

这会禁用网络摄像头，以防止其被用来监视您。

  

您也可以将麦克风模块列入黑名单，但这在系统之间可能会有所不同。要查找模块的名称，请在 / proc/asound/modules 中查找并将其列入黑名单。例如，一个这样的模块是 snd_hda_intel。

  

请注意，尽管有时麦克风的内核模块与扬声器的模块相同。这意味着像这样禁用麦克风也可能会无意中禁用任何扬声器，虽然扬声器也有可能变成麦克风，所以这不一定是消极的结果。

  

最好从物理上删除这些设备，或者至少在 BIOS/UEFI 中禁用它们。禁用内核模块并不总是那么有效。

  

**rfkill**

  

可以通过 rfkill 将无线设备列入黑名单，以进一步减少远程攻击面。要将所有无线设备列入黑名单，请执行：

```
rfkill block all
```

WiFi 可以通过以下方式解锁：

```
rfkill unblock wifi
```

在使用 systemd 的系统上，rfkill 在所有会话中均保持不变，但是，在使用其他 init 系统的系统上，您可能必须创建一个 init 脚本以在引导时执行这些命令。

  

**其他内核指针泄漏**

  

前面的部分已经防止了一些内核指针泄漏，但是还有更多泄漏。

  

在文件系统上，/boot 中存在内核映像和 System.map 文件。/usr/src 和 /{,usr/} lib/modules 目录中还有其他敏感的内核信息。您应该限制这些目录的文件权限，以使它们只能由 root 用户读取。您还应该删除 System.map 文件，因为除高级调试外，它们都不需要。

  

此外，某些日志记录守护程序（例如 systemd 的 journalctl）包括内核日志，可用于绕过上述 dmesg_restrict 保护。从 adm 组中删除用户通常足以撤销对以下日志的访问：

```
gpasswd -d $user adm
```

  

**限制对 sysfs 的访问**

  

sysfs 是伪文件系统，可提供大量的内核和硬件信息。它通常安装在 / sys 上。sysfs 导致大量信息泄漏，尤其是内核指针泄漏。Whonix 的 security-misc 软件包包括 hide-hardware-info 脚本，该脚本限制访问此目录以及 / proc 中的一些脚本，以试图隐藏潜在的硬件标识符并防止内核指针泄漏。该脚本是可配置的，并允许基于组将特定的应用程序列入白名单。建议应用此方法，并使其在启动时使用 init 脚本执行。或者这样做成 systemd 服务 [1]。

  

为了使基本功能在使用 systemd 的系统上运行，必须将一些系统服务列入白名单。这可以通过创建 / etc/systemd/system/user@.service.d/sysfs.conf 并添加以下内容来完成：

```
[Service]  
SupplementaryGroups=sysfs}
```

但是，这不能解决所有问题。许多应用程序可能仍会中断，您需要将它们正确列入白名单。

  

**Linux 强化**

  

某些发行版（例如 Arch Linux）包括强化的内核程序包。它包含许多强化补丁程序和更注重安全性的内核配置。如果可能的话，建议安装它。

  

**Grsecurity**

  

Grsecurity 是一组内核修补程序，可以大大提高内核安全性。这些补丁曾经可以免费获得，但是现在需要购买了。如果可用，则强烈建议您获取它。Grsecurity 提供了最新的内核和用户空间保护。

  

**内核运行时防护**

  

Linux Kernel Runtime Guard（LKRG）是一个内核模块，可确保运行时内核的完整性并检测漏洞。它可以杀死整个类别的内核漏洞。但这并不是一个完美的缓解方法，因为 LKRG 在设计上可以绕开。它仅适用于现成的恶意软件。但是，尽管可能性不大，但 LKRG 本身可能会像其他任何内核模块一样公开新的漏洞。

  

**自编译内核**

  

建议编译您自己的内核，同时启用尽可能少的内核模块和尽可能多的安全性功能，以将内核的受攻击面保持在绝对最低限度。

  

另外，应用内核强化补丁，例如如上所述的 linux-hardened 或 grsecurity。

  

发行版编译的内核还具有公共内核指针 / 符号，这对于漏洞利用非常有用。编译自己的内核将为您提供独特的内核符号，连同 kptr_restrict，dmesg_restrict 和其他针对内核指针泄漏的强化措施，将使攻击者更加难以创建依赖于内核指针知识的漏洞利用程序。

  

您就可以从 Whonix 的强化内核中汲取灵感或使用它。

强制访问措施

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

强制访问控制（MAC）系统对程序可以访问的内容进行细粒度的控制。这意味着您的浏览器将无权访问您的整个主目录或类似目录。

  

最常用的 MAC 措施是 SELinux 和 AppArmor。SELinux 比 AppArmor 更安全，因为它的粒度更细。例如，它是基于 inode 而不是基于路径的，允许强制执行明显更严格的限制，可以过滤内核 ioctl 等。不幸的是，这是以难以使用和难以学习为代价的，因此某些人可能会首选 AppArmor。

  

要在内核中启用 AppArmor，必须设置以下引导参数：

```
apparmor=1 security=apparmor
```

要启用 SELinux，请设置以下参数：

```
selinux=1 security=selinux
```

请记住，仅启用 MAC 措施本身并不能神奇地提高安全性。您必须制定严格的政策才能充分利用它。例如，要创建 AppArmor 配置文件，请执行：

```
aa-genprof $path_to_program
```

打开程序，然后像往常一样开始使用它。AppArmor 将检测需要访问哪些文件，并将它们添加到配置文件中（如果您选择的话）。但是，仅凭这一点不足以提供高质量的配置文件。请参阅 AppArmor 文档 [2] 以获取更多详细信息。

  

如果您想更进一步，则可以通过实施 initramfs 勾子来设置一个完整的系统 MAC 策略，该策略限制每个单个用户空间进程，该挂钩对 init 系统强制实施 MAC 策略。这就是 Android 使用 SELinux 的方式，以及 Whonix 未来将如何使用 AppArmor 的方式。对于加强实施最小特权原则的强大安全模型是必要的。

沙箱

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

**应用沙箱**

  

沙箱可让您在隔离的环境中运行程序，该环境对系统的其余部分具有有限的访问权限或完全没有访问权限。您可以使用它们来保护应用程序安全或运行不受信任的程序。

  

建议与 AppArmor 或 SELinux 一起在单独的用户帐户中使用 Bubblewrap 到沙箱程序。您也可以考虑改用 gVisor，它的优点是为每个来宾提供了自己的内核。

这些方法中的任何一个都可以用来创建一个功能强大的沙箱，并且暴露的攻击面最小。如果您不想自己创建沙箱，请在完成后考虑使用 Whonix 的 sandbox-app-launcher。您不应该使用 Firejail。

  

诸如 Docker 和 LXC 之类的容器解决方案经常被误导为沙盒形式。它们太宽松了，无法广泛支持各种应用程序，因此不能认为它们是强大的应用程序沙箱。

  

**常见沙箱逃逸**

  

**PulseAudio**

  

PulseAudio 是一种常见的声音服务器，但在编写时并未考虑隔离或沙盒的问题，这使其成为重复出现的沙盒逃逸漏洞。为了防止这种情况，建议您从沙箱中阻止对 PulseAudio 的访问，或者从系统中完全卸载它。

  

**D-Bus**

  

D-Bus 是台式机 Linux 上最流行的进程间通信形式，但它也是沙箱逃逸的另一种常见途径，因为它允许与服务自由交互。这些漏洞的一个例子就是 Firejail。您应该从沙箱中阻止对 D-Bus 的访问，或者通过 MAC 以细粒度的规则进行调解。

  

**GUI 隔离**

  

任何 Xorg 窗口都可以访问另一个窗口。这允许琐碎的键盘记录或屏幕截图程序，甚至可以记录诸如 root 密码之类的内容。您可以使用嵌套的 X11 服务器（例如 Xpra 或 Xephyr 和 bubblewrap）将 Xorg 窗口沙箱化。默认情况下，Wayland 将窗口彼此隔离，这将是一个比 Xorg 更好的选择，尽管 Wayland 可能不如 Xorg 普遍可用，因为它在开发中较早。

  

**ptrace**

  

如前所述，ptrace 是一个系统调用，可能会被滥用破坏在沙箱外部运行的进程。为避免这种情况，您可以通过 sysctl 启用内核 YAMA ptrace 限制，也可以在 seccomp 过滤器中将 ptrace syscall 列入黑名单。

  

**TIOCSTI**

  

TIOCSTI 是一个 ioctl，它允许注入终端命令，并为攻击者提供了一种简单的机制，可以在同一用户会话内的其他进程之间横向移动。可以通过将 seccomp 过滤器中的 ioctl 列入黑名单或使用 bubblewrap 的 --new-session 参数来缓解这种攻击。

  

**Systemd 沙箱**

  

虽然不建议使用 systemd，但有些系统可能无法切换。这些人至少可以使用沙盒服务，因此他们只能访问所需的内容。这是一个沙箱化 systemd 服务的示例：

```
[Service]  
CapabilityBoundingSet=CAP_NET_BIND_SERVICE  
ProtectSystem=strict  
ProtectHome=true  
ProtectKernelTunables=true  
ProtectKernelModules=true  
ProtectControlGroups=true  
ProtectKernelLogs=true  
ProtectHostname=true  
ProtectClock=true  
ProtectProc=invisible  
ProcSubset=pid  
PrivateTmp=true  
PrivateUsers=yes  
PrivateDevices=true  
MemoryDenyWriteExecute=true  
NoNewPrivileges=true  
LockPersonality=true  
RestrictRealtime=true  
RestrictSUIDSGID=true  
RestrictAddressFamilies=AF_INET  
RestrictNamespaces=yes  
SystemCallFilter=write read openat close brk fstat lseek mmap mprotect munmap rt_sigaction rt_sigprocmask ioctl nanosleep select access execve getuid arch_prctl set_tid_address set_robust_list prlimit64 pread64 getrandom  
SystemCallArchitectures=native  
UMask=0077  
IPAddressDeny=any  
AppArmorProfile=/etc/apparmor.d/usr.bin.example
```

所有选项的说明：

  

*   CapabilityBoundingSet=— Specifies the capabilities the process is given.
    
*   ProtectHome=true— Makes all home directories inaccessible.
    
*   ProtectKernelTunables=true— Mounts kernel tunables such as those modified throughsysctlas read-only.
    
*   ProtectKernelModules=true— Denies module loading and unloading.
    
*   ProtectControlGroups=true— Mounts all control group hierarchies as read-only.
    
*   ProtectKernelLogs=true— Prevents accessing the kernel logs.
    
*   ProtectHostname=true— Prevents changes to the system hostname.
    
*   ProtectClock— Prevents changes to the system clock.
    
*   ProtectProc=invisible— Hides all outside processes.
    
*   ProcSubset=pid— Permits access to only the pid subset of/proc.
    
*   PrivateTmp=true— Mounts an empty tmpfs over/tmpand/var/tmp, therefore hiding their previous contents.
    
*   PrivateUsers=true— Sets up an empty user namespace to hide other user accounts on the system.
    
*   PrivateDevices=true— Creates a new/devmount with minimal devices present.
    
*   MemoryDenyWriteExecute=true— Enforces a memory W^X policy.
    
*   NoNewPrivileges=true— Prevents escalating privileges.
    
*   LockPersonality=true— Locks down thepersonality()syscall to prevent switching execution domains.
    
*   RestrictRealtime=true— Prevents attempts to enable realtime scheduling.
    
*   RestrictSUIDSGID=true— Prevents executing setuid or setgid binaries.
    
*   RestrictAddressFamilies=AF_INET— Restricts the usable socket address families to IPv4 only (AF_INET).
    
*   RestrictNamespaces=true— Prevents creating any new namespaces.
    
*   SystemCallFilter=...— Restricts the allowed syscalls to the absolute minimum. If you aren't willing to maintain your own custom seccomp filter, then systemd provides many predefined system call sets that you can use.@system-servicewill be suitable for many use cases.
    
*   SystemCallArchitectures=native— Prevents executing syscalls from other CPU architectures.
    
*   UMask=0077— Sets the umask to a more restrictive value.
    
*   IPAddressDeny=any— Blocks all incoming and outgoing traffic to/from any IP address. SetIPAddressAllow=to configure a whitelist. Alternatively, setup a network namespace withPrivateNetwork=true.
    
*   AppArmorProfile=...— Runs the process under the specified AppArmor profile.
    

您不能仅将此示例配置复制到您的配置中，每种服务的要求各不相同，并且必须针对每种服务微调沙箱。要了解有关您可以设置的所有选项的更多信息，请阅读 systemd.exec 手册页 [3]。

  

如果您使用的系统不是 systemd 而是 init，那么可以使用 bubblewrap 轻松复制所有这些选项。

  

**gVisor**

  

普通沙箱固有地与主机共享同一内核。您信任我们已经评估为不安全的内核，可以正确限制这些程序。由于主机内核的整个攻击面已完全暴露，因此沙盒中的内核利用程序可以绕过任何限制。已经进行了一些努力来限制使用 seccomp 的攻击面，但不足以完全解决此问题。

  

GVisor 是解决此问题的方法。它为每个应用程序提供了自己的内核，该内核以内存安全的语言重新实现了 Linux 内核的大部分系统调用，从而提供了明显更强的隔离性。

  

**虚拟机**

  

虽然不是传统的 “沙盒”，但虚拟机通过虚拟化全新系统来分离进程，从而提供了非常强大的隔离性。KVM 是内核模块，它允许内核充当管理程序，而 QEMU 是利用 KVM 的仿真器。Virt-manager 和 GNOME Boxs 都是良好且易于使用的 GUI，用于管理 KVM / QEMU 虚拟机。不建议使用 Virtualbox 的原因有很多。

强化内存分配器

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

hardened_malloc 是一种硬化的内存分配器，可为堆内存损坏漏洞提供实质性的保护。它很大程度上基于 OpenBSD 的 malloc 设计，但具有许多改进。

  

可以通过 LD_PRELOAD 环境变量针对每个应用程序使用 hardened_malloc。例如，假设您编译的库位于 / usr/lib/libhardened_malloc.so，则可以执行：

```
LD_PRELOAD="/usr/lib/libhardened_malloc.so" $program
```

通过全局预加载该库，也可以在系统范围内使用它，这是使用它的推荐方法。为此，请编辑 / etc/ld.so.preload 并插入：

```
/usr/lib/libhardened_malloc.so
```

尽管大多数应用程序都可以正常工作，但 hardened_malloc 可能会破坏某些应用程序。建议使用以下选项编译 hardened_malloc 以最大程度地减少损坏：

```
CONFIG_SLAB_QUARANTINE_RANDOM_LENGTH=0 CONFIG_SLAB_QUARANTINE_QUEUE_LENGTH=0 CONFIG_GUARD_SLABS_INTERVAL=8
```

您还应该使用 sysctl 设置以下内容，以适应 hardened_malloc 创建的大量保护页：

```
vm.max_map_count=524240
```

Whonix 项目为基于 Debian 的发行版提供了 hardened_malloc 软件包。

强化编译标志

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

编译自己的程序可以带来很多好处，因为它使您能够优化程序的安全性。但是，执行完全相反的操作并降低安全性很容易，如果您不确定自己在做什么，请跳过本节。在基于源的发行版（例如 Gentoo）上，这将是最简单的，但也可以在其他发行版上这样做。

  

某些编译选项可用于添加其他漏洞利用缓解措施，从而消除整个类别的常见漏洞。您可能听说过常规保护，例如位置独立可执行文件，堆栈粉碎保护程序，立即绑定，只读重定位和 FORTIFY_SOURCE，但是本节将不做介绍，因为它们已被广泛采用。相反，它将讨论诸如控制流完整性和影子堆栈之类的现代漏洞利用缓解措施。

  

本节涉及主要用 C 或 C ++ 编写的本机程序。您必须使用 Clang 编译器，因为这些功能在 GCC 上不可用。请记住，由于未广泛采用这些缓解措施，因此某些应用程序在启用它们后可能无法运行。

  

控制流完整性（CFI）是一种缓解漏洞利用的方法，旨在防止诸如 ROP 或 JOP 之类的代码重用攻击。由于更广泛采用的缓解措施（例如 NX）使过时的利用技术过时了，因此使用这些技术利用了很大一部分漏洞。Clang 支持细粒度的前沿 CFI，这意味着它可以有效缓解 JOP 攻击。Clang 的 CFI 本身并不能减轻 ROP；您还必须使用下面记录的单独机制。要启用此功能，必须应用以下编译标志：

```
-flto -fvisibility=hidden -fsanitize=cfi
```

影子堆栈通过将程序复制到其他隐藏堆栈中来保护程序的返回地址。然后比较主堆栈和影子堆栈中的返回地址，看两者是否不同。如果是这样，则表明存在攻击，程序将中止，从而减轻了 ROP 攻击。Clang 具有称为 ShadowCallStack 的功能，可以完成此操作，但是，仅在 ARM64 上可用。要启用此功能，必须应用以下编译标志：

```
-fsanitize=shadow-call-stack
```

如果上述 ShadowCallStack 不是一个选项，则可以选择使用具有相似目标的 SafeStack。但是，不幸的是，此功能有许多漏洞，因此效果不甚理想。如果仍然希望启用此功能，则必须应用以下编译标志：

```
-fsanitize=safe-stack
```

最常见的内存损坏漏洞之一是未初始化的内存。Clang 有一个选项可以使用零或特定模式自动初始化变量。建议将变量初始化为零，因为使用其他模式比利用漏洞缓解功能更适合发现错误。要启用此功能，必须应用以下编译标志：

```
-ftrivial-auto-var-init=zero -enable-trivial-auto-var-init-zero-knowing-it-will-be-removed-from-clang
```

但该选项的存在目前正在辩论 [4] 中。

内存安全语言

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

用内存安全语言编写的程序会自动受到保护，免受各种安全漏洞的影响，这些安全漏洞包括缓冲区溢出，未初始化的变量，售后使用等。Microsoft 和 Google 的安全研究人员进行的研究证明，已发现的大多数漏洞都是内存安全问题。这样的内存安全语言的示例包括 Rust，Swift 和 Java，而内存不安全语言的示例包括 C 和 C ++。如果可行，应使用内存安全替代品替换尽可能多的程序。

Root 账户

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

root 可以执行任何操作，并且可以访问您的整个系统。因此，应尽可能将其锁定，以使攻击者无法轻松获得 root 用户访问权限。

  

**/etc/securetty**

  

/etc/securetty 文件指定允许您以 root 用户身份登录的位置。该文件应保留为空，以便任何人都不能从终端上这样做。

  

**限制 su**

  

su 可让您从终端切换用户。默认情况下，它尝试以 root 用户身份登录。要将 su 的使用限制在 wheel 组中，请编辑 / etc/pam.d/su 和 / etc/pam.d/su-l 并添加：

```
auth required pam_wheel.so use_uid
```

您应该在 wheel 组中拥有尽可能少的用户。

  

**锁定 root 账户**

  

要锁定 root 帐户以防止任何人以 root 身份登录，请执行：

```
passwd -l root
```

在执行此操作之前，请确保您具有获取根的替代方法（例如，从活动 USB 引导并更改为文件系统的 chroot），以免您无意中将自己锁定在系统之外。

  

**拒绝通过 SSH 的远程 root 登陆**

  

为了防止某人通过 SSH 以 root 身份登录，请编辑 / etc/ssh/sshd_config 并添加：

```
PermitRootLogin no
```

  

**增加散列回合数**

  

您可以增加 shadow 使用的哈希回合数，从而通过迫使攻击者计算更多的哈希值来破解您的密码，从而提高哈希密码的安全性。默认情况下，shadow 使用 5000 次回合，但是您可以将其增加到任意数量。尽管配置的回合越多，登录速度就越慢。编辑 / etc/pam.d/passwd 并添加回合选项。

```
password required pam_unix.so sha512 shadow nullok rounds=65536
```

这使 shadow 执行 65536 次散列回合。

  

应用此设置后，密码不会自动重新加密，因此您需要使用以下方法重置密码：

```
passwd $username
```

  

**限制 Xorg root 访问**

  

默认情况下，某些发行版以 root 用户身份运行 Xorg，这是一个问题，因为 Xorg 包含大量古老而又复杂的代码，这增加了巨大的攻击面，并使其更有可能拥有可以获取 root 特权的漏洞利用程序。要阻止它作为 root 用户执行，请编辑 / etc/X11/Xwrapper.config 并添加：

```
needs_root_rights = no
```

  

**安全访问 root**

  

恶意软件可以使用多种方法来嗅探 root 帐户的密码。因此，访问根帐户的传统方式是不安全的，最好根本不访问根，但这实际上是不可行的。本节详细介绍了访问根帐户的最安全方法。在安装操作系统后，应立即应用这些说明，以确保该软件不含恶意软件。

  

您绝对不能使用普通用户帐户访问 root，因为 root 可能已被盗用。您也不能直接登录到根帐户。通过执行以下操作，创建一个单独的 “管理员” 用户帐户，该帐户仅用于访问 root 用户，而不能用于访问其他用户：

```
useradd admin
```

执行并来设置一个非常强的密码：

```
passwd admin
```

仅允许该帐户使用您首选的权限提升机制。例如，如果使用 sudo，则通过执行以下命令来添加 sudoers 异常：

```
visudo -f /etc/sudoers.d/admin-account
```

然后输入：

```
admin ALL=(ALL) ALL
```

  

确保没有其他帐户可以访问 sudo（或您的首选机制）。

  

现在，要实际登录到该帐户，请先重新启动 - 例如，这可以防止受损的窗口管理器执行登录欺骗。当提供登录提示时，请通过按键盘上的以下组合键来激活安全注意键：

```
Alt + SysRq + k
```

这将杀死当前虚拟控制台上的所有应用程序，从而克服登录欺骗攻击。现在，您可以安全地登录到您的管理员帐户，并使用 root 用户执行任务。完成后，注销管理员帐户，然后重新登录到非特权用户帐户。

防火墙

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

防火墙可以控制传入和传出的网络流量，并且可以用来阻止或允许某些类型的流量。除非有特殊原因，否则应始终阻止所有传入流量。建议设置严格的 iptables 或 nftables 防火墙。火墙必须针对您的系统进行微调，并且没有一个适合所有防火墙的规则集。建议您熟悉创建防火墙规则。Arch Wiki 和手册页 [5] 都是很好的资源。

  

这是基本 iptables 配置的示例，该配置禁止所有传入的网络流量：

```
*filter  
:INPUT DROP [0:0]  
:FORWARD DROP [0:0]  
:OUTPUT ACCEPT [0:0]  
:TCP - [0:0]  
:UDP - [0:0]  
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT  
-A INPUT -i lo -j ACCEPT  
-A INPUT -m conntrack --ctstate INVALID -j DROP  
-A INPUT -p udp -m conntrack --ctstate NEW -j UDP  
-A INPUT -p tcp --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j TCP  
-A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable  
-A INPUT -p tcp -j REJECT --reject-with tcp-reset  
-A INPUT -j REJECT --reject-with icmp-proto-unreachable  
COMMIT
```

但是，您不应尝试在实际系统上使用此示例。它仅适用于某些台式机系统。

身份标识

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

为了保护隐私，最好最大程度地减少可追溯到您的信息量。

  

**主机名和用户名**

  

请勿在主机名或用户名中添加唯一标识的内容。将它们保留为通用名称，例如 “host” 和“user”，以便它们无法识别您。

  

**Timezones / Locales / Keymaps**

  

如果可能，应将您的时区设置为 “UTC”，将区域设置和键盘映射设置为 “ US”。

  

**机器 ID**

  

一个独一无二的机器 ID 被存储在 / var/lib/dbus/machine-id （systemd 系统是保存在 / etc/machine-id）这些应编辑为通用名称，例如 Whonix ID：

```
b08dfa6083e7567a1921a715000001fb
```

  

**MAC 地址欺骗**

  

MAC 地址是分配给网络接口控制器（NIC）的唯一标识符。每次您连接到网络时（WIFI 或以太网）则您的 MAC 地址已暴露。这使人们可以使用它来跟踪您并在本地网络上唯一地标识您。

  

但您不应该完全随机化 MAC 地址。拥有完全随机的 MAC 地址是显而易见的，并且会对您脱颖而出的行为产生不利影响。

  

MAC 地址的 OUI（组织唯一标识符）部分标识芯片组的制造商。对 MAC 地址的这一部分进行随机化处理可能会为您提供以前从未使用过的 OUI，数十年来从未使用过的 OUI 或在您所在的地区极为罕见的 OUI，因此使您脱颖而出，很明显地表明您在欺骗 MAC 地址。

  

MAC 地址的末尾标识您的特定设备，并且可以用来跟踪您的设备。仅对 MAC 地址的这一部分进行随机化可防止您被跟踪，同时仍使 MAC 地址看起来可信。

  

要欺骗这些地址，请首先执行以下命令找出您的网络接口名称：

```
ip a
```

接下来，安装 macchanger 并执行：

```
macchanger -e $network_interface
```

要在每次引导时随机分配 MAC 地址，您应该为您的特定初始化系统创建一个初始化脚本。这是 systemd 的一个示例：

```
[Unit]  
Description=macchanger on eth0  
Wants=network-pre.target  
Before=network-pre.target  
BindsTo=sys-subsystem-net-devices-eth0.device  
After=sys-subsystem-net-devices-eth0.device  
  
[Service]  
ExecStart=/usr/bin/macchanger -e eth0  
Type=oneshot  
  
[Install]  
WantedBy=multi-user.target
```

上面的示例在启动时欺骗了 eth0 接口的 MAC 地址。将 eth0 替换为您的网络接口。

  

**时间攻击**

  

几乎每个系统都有不同的时间。这可用于时钟偏斜指纹攻击，几毫秒的差异足以使用户被暴露识别。

  

**ICMP 时间戳**

  

ICMP 时间戳会在查询答复中泄漏系统时间。阻止这些攻击的最简单方法是利用防火墙阻止传入连接，或者使内核忽略 ICMP 请求。

  

**TCP 时间戳**

  

TCP 时间戳也会泄漏系统时间。内核尝试通过对每个连接使用随机偏移量来解决此问题，但这不足以解决问题。因此应该禁用 TCP 时间戳，可以通过使用 sysctl 设置以下内容来完成：

```
net.ipv4.tcp_timestamps=0
```

  

**TCP 初始化序号**

  

TCP 初始序列号（ISN）是泄漏系统时间的另一种方法。为了减轻这种情况，您必须安装 tirdad 内核模块，该模块会生成用于连接的随机 ISN。

  

**时间同步**

  

时间同步对于匿名性和安全性至关重要。错误的系统时钟可能使您遭受时钟偏斜指纹攻击，或者可以用来为您提供过时的 HTTPS 证书，从而绕过证书到期或吊销。

  

最流行的时间同步方法 NTP 是不安全的，因为它未经加密和未经身份验证，因此攻击者可以轻易地拦截和修改请求。NTP 还会以 NTP 时间戳格式泄漏本地系统时间，该格式可用于时钟偏斜指纹识别，如前所述。

  

因此，您应该卸载所有 NTP 客户端并禁用 systemd-timesyncd（如果正在使用）。您可以通过安全连接（HTTPS 或最好是 Torion 服务）连接到受信任的网站，而不是 NTP，并从 HTTP 标头中提取当前时间。达到此目的的工具是 sdwdate 或我自己的安全时间同步工具。

  

**按键指纹**

  

可以通过他们在键盘上输入键的方式来对人进行指纹识别。您可以通过键入速度，在两次按键之间的暂停，每次按键被按下和释放的确切时间等方式来唯一地进行指纹识别。可以使用 KeyTrac 在线进行测试。

  

Kloak 是一种工具，旨在通过混淆按键和释放事件之间的时间间隔来克服这种跟踪方法。当按键被按下时，它会引入随机延迟，然后由应用程序选择。

文件权限

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

默认情况下，文件的权限是非常宽松的。您应该在整个系统中搜索权限不当的文件和目录，并对其进行限制。例如，在诸如 Debian 之类的某些发行版中，用户的 Home 目录是全局可读的。

  

这可以通过执行以下操作来限制：

```
chmod 700 /home/$user
```

另外一些示例是 / boot，/usr /src 和 / {,usr /} lib/modules 它们包含内核映像，System.map 和其他各种文件，所有这些文件都可能泄漏有关内核的敏感信息。

```
chmod 700 /boot /usr/src /lib/modules /usr/lib/modules
```

在基于 Debian 的发行版中，必须使用 dpkg-statoverride 保留文件许可权。否则，它们将在更新期间被覆盖。

  

Whonix 的 SUID Disabler 和 Permission Hardener 会自动应用本节中详细介绍的步骤。

  

**setuid / setgid**

  

Setuid / SUID 允许用户使用二进制文件所有者的特权执行二进制文件。这通常用于允许非特权用户使用通常仅为 root 用户保留的某些功能。因此，许多 SUID 二进制文件都有特权升级安全漏洞的历史记录。Setgid / SGID 类似，但适用于组而不是用户。要使用 setuid 或 setgid 位查找系统上的所有二进制文件，请执行：

```
find / -type f \( -perm -4000 -o -perm -2000 \)
```

然后，您应该删除不使用的程序上的所有不必要的 setuid / setgid 位，或将其替换为功能。要删除 setuid 位，请执行：

```
chmod u-s $path_to_program
```

要删除 setgid 位，执行：

```
chmod g-s $path_to_program
```

要向文件添加功能，请执行：

```
setcap $capability+ep $path_to_program
```

或者，要删除不必要的功能，请执行：

```
setcap -r $path_to_program
```

  

**umask**

  

umask 设置新创建文件的默认文件权限。默认的 umask 是 0022，它不是很安全，因为它为系统上的每个用户提供了对新创建文件的读取访问权限。要使所有者以外的任何人都不可读新文件，请编辑 / etc/profile 并添加：

```
umask 0077
```

  

核心转储

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

核心转储包含特定时间（通常是该程序崩溃时）该程序的已记录内存。它们可能包含敏感信息，例如密码和加密密钥，因此必须将其禁用。

  

禁用它们的方法主要有三种：sysctl，systemd 和 ulimit。

  

**sysctl**

  

通过 sysctl 设置以下设置：

```
kernel.core_pattern=|/bin/false
```

  

**systemd**

  

创建 / etc/systemd/coredump.conf.d/disable.conf 并添加如下内容：

```
[Coredump]  
Storage=none
```

  

**ulimit**

  

编辑 / etc/security/limits.conf 并添加如下内容：

```
* hard core 0
```

  

**setuid 进程**

  

即使在进行了这些设置之后，以提升的特权运行的进程仍可能会转储其内存。

  

为了防止他们这样做，请通过 sysctl 设置以下内容：

```
fs.suid_dumpable=0
```

  

Swap

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

与核心转储类似，交换或分页将部分内存复制到磁盘，其中可能包含敏感信息。应该将内核配置为仅在绝对必要时进行交换，相应的 sysctl 设置：

```
vm.swappiness=1
```

  

PAM

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

PAM 是用于用户身份验证的框架。这就是您登录时使用的机制。您可以通过要求使用强密码或在失败的登录尝试后强制执行延迟验证来使其更加安全。

  

要强制使用强密码，可以使用 pam_pwquality。它强制执行密码的可配置策略。例如，如果您希望密码至少包含 16 个字符（最小），与旧密码（difok）至少 6 个不同的字符，至少 3 个数字（dcredit），至少 2 个大写字母（ucredit），至少 2 个字符小写字母（lcredit）和至少 3 个其他字符（ocredit），然后编辑 / etc/pam.d/passwd 并添加：

```
password required pam_pwquality.so retry=2 minlen=16 difok=6 dcredit=-3 ucredit=-2 lcredit=-2 ocredit=-3 enforce_for_root  
password required pam_unix.so use_authtok sha512 shadow
```

要强制执行延迟验证，可以使用 pam_faildelay。要在两次失败的登录尝试之间添加至少 4 秒的延迟以阻止暴力破解尝试，请编辑 / etc/pam.d/system-login 并添加：

```
auth optional pam_faildelay.so delay=4000000
```

4000000 是 4 秒（以微秒为单位）。

Microcode 更新

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

Microcode 更新对于修复关键的 CPU 漏洞（如 Meltdown 和 Spectre 等）至关重要。大多数发行版都将这些发行版包含在其软件仓库中，例如 Arch Linux 和 Debian。

IPv6 隐私扩展

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

IPv6 地址是从计算机的 MAC 地址生成的，从而使您的 IPv6 地址是唯一的，并直接绑定到计算机。隐私扩展会生成一个随机的 IPv6 地址，以减轻这种形式的跟踪。请注意，如果您开启了 MAC 地址欺骗机制或禁用了 IPv6，则无需执行这些步骤。

  

要启用这些功能，请通过 sysctl 设置以下设置：

```
net.ipv6.conf.all.use_tempaddr=2  
net.ipv6.conf.default.use_tempaddr=2
```

  

**NetworkManager**

  

要为 NetworkManager 启用隐私扩展，请编辑 / etc/NetworkManager/NetworkManager.conf 并添加：

```
[connection]  
ipv6.ip6-privacy=2
```

  

**systemd-networkd**

  

要为 systemd-networkd 启用隐私扩展，请创建 / etc/systemd/network/ipv6-privacy.conf 并添加：

```
[Network]  
IPv6PrivacyExtensions=kernel
```

  

**分区和挂载选项**

  

文件系统应分为多个分区，以对其权限进行细粒度控制。可以添加不同的安装选项以限制可以执行的操作：

  

*   nodev - 禁止使用设备
    
*   nosuid - 禁止 setuid 或 setgid 位
    
*   noexec - 禁止执行任何二进制文件
    

这些安装选项应在 / etc/fstab 中尽可能设置。如果您不能使用单独的分区，请创建绑定挂载。一个更安全的 / etc/fstab 的示例：

```
/        /          ext4    defaults                              1 1  
/home    /home      ext4    defaults,nosuid,noexec,nodev          1 2  
/tmp     /tmp       ext4    defaults,bind,nosuid,noexec,nodev     1 2  
/var     /var       ext4    defaults,bind,nosuid                  1 2  
/boot    /boot      ext4    defaults,nosuid,noexec,nodev          1 2
```

请注意，可以通过 shell 脚本绕过 noexec。

熵

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

熵基本上反应操作系统信息收集的随机程度，对于诸如加密之类的事情至关重要。因此，最好通过安装其他随机数生成器（如 haveged 和 jitterentropy）从各种来源收集尽可能多的熵。

  

为了使 jitterentropy 正确运行，必须通过创建 / usr/lib/modules-load.d/jitterentropy.conf 并添加以下内容尽早加载内核模块：

```
jitterentropy_rng
```

  

**RDRAND**

  

RDRAN 是提供随机数的 CPU 指令。如果可用，内核会自动将其用作熵源。但是由于它是专有的并且是 CPU 本身的一部分，因此无法审核和验证其安全性。您甚至无法对代码进行反向工程。该 RNG 以前曾遭受过漏洞的攻击，其中有些可能是后门攻击。通过设置以下引导参数可以不信任此功能：

```
random.trust_cpu=off
```

  

以 root 身份编辑文件

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

建议不要以 root 用户身份运行普通的文本编辑器。大多数文本编辑器可以做的不仅仅是简单地编辑文本文件，而且还可以被利用。例如，以 root 身份打开 vi 并输入：sh。现在，您具有一个可以访问整个系统的 root shell，攻击者可以轻松利用该 shell。

  

解决方案是使用 sudoedit。这会将文件复制到一个临时位置，以普通用户身份打开文本编辑器，编辑该临时文件并以 root 用户身份覆盖原始文件。这样，实际的编辑器就不会以 root 身份运行。要使用 sudoedit，执行：

```
sudoedit $path_to_file
```

默认情况下，它使用 vi，但是可以通过 EDITOR 或 SUDO_EDITOR 环境变量来切换默认编辑器。例如，要使用 nano，请执行：

```
EDITOR=nano sudoedit $path_to_file
```

可以在 / etc/environment 中全局设置此环境变量。

特定发行版的安全强化

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

**HTTP 包管理器镜像**

  

默认情况下，Linux 发行版通常使用 HTTP 或 HTTP 和 HTTPS 镜像的混合来从其软件存储库下载软件包。人们认为这很好，因为程序包管理器会在安装前验证程序包的签名。但是，从历史上看，已经有很多绕过此方法的地方。您应将软件包管理器配置为从 HTTPS 镜像专门下载以进行深度防御。

  

**APT seccomp-bpf**

  

自软件包管理器 Debian Buster 以来，APT 已支持可选的 seccomp-bpf 过滤。这限制了允许执行 APT 的系统调用，这可能严重限制攻击者尝试利用 APT 中的漏洞时对系统造成危害的能力。要启用此功能，请创建 / etc/apt/apt.conf.d/40sandbox 并添加：

```
APT::Sandbox::Seccomp "true";
```

  

物理安全

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

全盘加密可确保对驱动器上的所有数据进行加密，并且不会被物理攻击者读取。大多数发行版都支持在安装过程中启用加密，请确保设置了强密码。您也可以使用 dm-crypt 手动加密驱动器。

  

请注意，全盘加密不包括 / boot，这样仍然可以修改内核、引导加载程序和其他关键文件。为了完全防止篡改，您还必须实施经过验证的引导。

  

**BIOS / UEFI 强化**

  

如果您仍在使用旧版 BIOS，则应迁移到 UEFI，以利用较新的安全功能。大多数 BIOS 或 UEFI 实现都支持设置密码。最好启用它并设置一个非常强壮的密码。虽然这是很弱的保护，因为重置密码很简单。它通常存储在易失性内存中，因此攻击者只需要能够卸下 CMOS 电池几秒钟，或者他们就可以使用某些主板上的跳线将其重置。

  

您还应该禁用所有未使用的设备和引导选项，例如 USB 引导，以减少攻击面。

  

别忽略 BIOS 或 UEFI 的更新，确保将其更新。将其与常规操作系统更新一样重要。

  

此外，请参阅《NSA 的硬件和固件安全指南 [6]》。

  

**Bootloader 密码**

  

引导加载程序会在引导过程的早期执行，并负责加载操作系统。保护它非常重要，否则，它可能会被篡改。例如，本地攻击者可以通过在启动时使用 init=/bin/bash 作为内核参数来轻松获得 root shell，该命令告诉内核执行 / bin/bash 而不是常规的 init 系统。您可以通过为引导加载程序设置密码来防止这种情况。仅设置引导程序密码不足以完全保护它。还必须按照以下说明设置经过验证的启动。

  

**Grub**

  

要为 GRUB 设置密码，请执行：

```
grub-mkpasswd-pbkdf2
```

输入您的密码，该密码将生成一个字符串。它将类似于 “grub.pbkdf2.sha512.10000.C4009...”。创建 / etc/grub.d/40_password 并添加：

```
set superusers="$username"  
password_pbkdf2 $username $password
```

用 grub-mkpasswd-pbkdf2 生成的字符串替换 “$username” 将用于被允许使用 GRUB 命令行，编辑菜单项和执行任何菜单项的超级用户。对于大多数人来说，这只是“root”。

  

重新生成您的配置文件，GRUB 现在将受到密码保护。

  

要仅限制编辑引导参数并访问 GRUB 控制台，同时仍然允许您引导，请编辑 /boot/grub/grub.cfg 并在 “menuentry '$OSName'”旁边添加 “ --unrestricted” 参数。

```
menuentry 'Arch Linux' --unrestricted
```

您将需要再次重新生成配置文件以应用此更改。

  

**Syslinux**

  

Syslinux 可以设置主密码或菜单密码。引导任何条目都需要主密码，而引导特定条目仅需要菜单密码。

  

要为 Syslinux 设置主密码，请编辑 / boot/syslinux/syslinux.cfg 并添加：

```
MENU MASTER PASSWD $password
```

要设置菜单密码，请编辑 / boot/syslinux/syslinux.cfg，并在带有您要密码保护的项目的标签内，添加：

```
MENU PASSWD $password
```

将 “$password” 替换为您要设置的密码。

  

这些密码可以是纯文本，也可以使用 MD5，SHA-1，SHA-256 或 SHA-512 进行散列。建议先使用强哈希算法（例如 SHA-256 或 SHA-512）对密码进行哈希处理，以避免将其存储为明文形式。

  

**systemd-boot**

  

systemd-boot 具有防止在引导时编辑内核参数的选项。在 loader.conf 文件中，添加：

```
editor no
```

systemd-boot 并不正式支持保护内核参数编辑器的密码，但是您可以使用 systemd-boot-password 来实现。

  

**验证引导**

  

经过验证的引导通过密码验证来确保引导链和基本系统的完整性。这可用于确保物理攻击者无法修改设备上的软件。

  

如果没有经过验证的引导，则一旦获得物理访问权限，就可以轻松绕过上述所有预防措施。经过验证的引导不仅像许多人认为的那样是为了物理安全。它还可以用于防止远程恶意软件持久化——如果攻击者设法破坏了整个系统并获得了很高的特权，则经过验证的引导将在重新引导后还原其更改，并确保它们无法持久化。

  

经过验证的最常见的引导实现是 UEFI 安全引导，但是它本身并不是一个完整的实现，因为它仅会验证引导加载程序和内核，这意味着可以通过以下方法：

  

*   仅 UEFI 安全启动就没有一成不变的信任根，因此物理攻击者仍然可以刷新设备的固件。为了减轻这种情况，请结合使用 UEFI 安全启动和 Intel Boot Guard 或 AMD Secure Boot。
    
*   远程攻击者（或不使用加密的物理攻击者）可以简单地修改操作系统的任何其他特权部分。例如，如果他们有修改内核的特权，那么他们也可以修改 / sbin/init 来有效地获得相同的结果。因此，仅验证内核和引导加载程序不会对远程攻击者产生任何影响。为了减轻这种情况，您必须使用 dm-verity 验证基本操作系统，尽管由于传统 Linux 发行版的布局，这非常困难且笨拙。
    

  

通常，很难在传统 Linux 上实现可靠的经过验证的引导实现。

  

**USBs**

  

USB 设备为物理攻击提供了重要的攻击面。例如 BadUSB 和 Stuxnet 是此类攻击的范例。最佳实践是禁止所有新连接的 USB 且仅将受信任设备列入白名单，USBGuard 对此非常有用。

  

您也可以将 nousb 用作内核引导参数，以禁用内核中的所有 USB 支持。可以 sysctl 设置 kernel.deny_new_usb=1

  

**DMA 攻击**

  

直接内存访问（DMA）攻击涉及通过插入某些物理设备来完全访问所有系统内存。这可以通过控制设备可访问的内存区域的 IOMMU 或将特别易受攻击的内核模块列入黑名单来缓解。

  

要启用 IOMMU，请设置以下内核引导参数：

```
intel_iommu=on amd_iommu=on
```

您只需要为特定的 CPU 制造商启用该选项，但同时启用这两个选项就没有问题。

```
efi=disable_early_pci_dma
```

通过在非常早的启动过程中禁用所有 PCI 桥接器上的 busmaster 位，此选项可修复上述 IOMMU 中的漏洞。

  

此外，Thunderbolt 和 FireWire 通常容易受到 DMA 攻击。要禁用它们，请将这些内核模块列入黑名单：

```
install firewire-core /bin/false  
install thunderbolt /bin/false
```

  

**冷启动攻击**

  

当攻击者在擦除 RAM 中的数据之前对其进行分析时，就会发生冷启动攻击。使用现代 RAM 时，冷启动攻击不太实用，因为 RAM 通常会在几秒钟或几分钟内清除，除非将其放入冷却液（如液氮或冷冻机）中。攻击者必须在几秒钟内将设备中的 RAM 棒拔出并将其暴露于液氮中，而且确保用户不会注意到。

  

如果冷启动攻击是威胁模型的一部分，请在关机后保护计算机几分钟，以确保没有人可以访问您的 RAM 记忆棒。您也可以将 RAM 棒焊接到主板上，以使其更难以卡住。如果使用笔记本电脑，请取出电池，然后直接用充电电缆供电。关机后请拔出电缆，以确保 RAM 彻底断电无法访问。

  

在内核自我保护启动参数部分中，空闲时内存清零选项将用零覆盖内存中的敏感数据。此外，强化的内存分配器可以通过 CONFIG_ZERO_ON_FREE 配置选项清除用户空间堆内存中的敏感数据。尽管如此，某些数据仍可能保留在内存中。

  

此外，现代内核还包括复位攻击缓解措施，该命令可命令固件在关机时擦除数据，尽管这需要固件支持。

  

确保正常关闭计算机，以使上述缓解措施可以开始。

  

如果以上都不适用您的威胁模型，则可以实施 Tails 的内存擦除过程，该过程将擦除大部分内存（视频内存除外），并且已被证明是有效的。

最佳实践

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

一旦对系统进行了尽可能多的加固，就应该遵循良好的隐私和安全性惯例：

  

*   禁用或删除不需要的东西以最小化攻击面。
    
*   保持更新。配置 cron 任务或 init 脚本以每天更新系统。
    
*   不要泄漏有关您或您的系统的任何信息，无论它看起来多么渺小。
    
*   遵循常规的安全和隐私建议
    

尽管已经进行了强化，但您必须记住 Linux 仍然是一个有缺陷的操作系统，没有任何强化可以完全修复它。

其他指南

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

您应该进行尽可能多的研究，而不要依赖单一的信息来源。最大的安全问题之一就是用户。这些是我认为有价值的其他指南的链接：

  

Arch Linux Security wiki page：https://wiki.archlinux.org/index.php/Security

  

Whonix Documentation：https://www.whonix.org/wiki/Documentation

  

NSA RHEL 5 Hardening Guide（稍有过时，但仍包含有用的信息）：https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm

  

KSPP recommended kernel settings：https://kernsec.org/wiki/index.php/Kernel_Self_Protection_Project/Recommended_Settings

  

kconfig-hardened-check：https://github.com/a13xp0p0v/kconfig-hardened-check/

术语

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

您可能需要重新生成 GRUB 配置，以应用对引导加载程序所做的某些更改。在不同的发行版之间，执行此操作的步骤有时可能会有所不同。例如，在诸如 Arch Linux 之类的发行版上，应通过执行以下命令来重新生成配置文件：

```
grub-mkconfig -o $path_to_grub_config
```

"$path_to_grub_config" 取决于您如何设置系统。它通常是 / boot/grub/grub.cfg 或 / boot/EFI/grub/grub.cfg，但是在执行此命令之前，请务必确保正确。

  

另外，在 Debian 或 Ubuntu 等发行版上，您应该执行以下命令：

```
update-grub
```

  

能力

![](https://mmbiz.qpic.cn/mmbiz_png/b2YlTLuGbKDsbJzupnILVFhPtMaRjmvPKYRqTMjibE9pnd8oiawLVrQbOHQe4wBXkBQkzpKCWPKBqWgOLgwccBug/640?wx_fmt=png)

在 Linux 内核中，“root 特权” 分为各种不同的能力（capabilities）。这在应用最小特权原则时很有帮助——可以给它们仅授予特定的子集，而不是授予进程总的 root 特权。例如，如果程序只需要设置系统时间，则只需要 CAP_SYS_TIME 而不是 root 所有能力。这会限制可能造成的损害，但是，您仍必须谨慎授予能力，因为无论如何，其中许多能力可能会被滥用以获取完整的 root 特权。

  

相关链接：

  

1.  https://github.com/Whonix/security-misc/blob/master/lib/systemd/system/hide-hardware-info.service
    
2.  https://gitlab.com/apparmor/apparmor/-/wikis/Documentation
    
3.  https://www.freedesktop.org/software/systemd/man/systemd.exec.html
    
4.  https://lists.llvm.org/pipermail/cfe-dev/2020-April/065221.html
    
5.  https://href.li/?https://linux.die.net/man/8/iptables
    
6.  https://github.com/nsacyber/Hardware-and-Firmware-Security-Guidance
    

译文链接：https://blog.gaochao.me/post/645734976535543808/linux 系统安全强化指南

* * *

关注乌雲安全，后台回复 “**b****urp5.1**” 获取渗透利器 burpsuite2021.5.1

**推荐阅读**[**![图片](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyIDG0WicDG27ztM2s7iaVSWKiaPdxYic8tYjCatQzf9FicdZiar5r7f7OgcbY4jFaTTQ3HibkFZIWEzrsGg/640?wx_fmt=png)**](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247494811&idx=1&sn=23ec661f57424184e43ea216a8398c58&chksm=9acd3c04adbab512e2a1c40156b05a5dc40ea07dacf239a9041235324a563d555a4e4625e988&scene=21#wechat_redirect)

公众号

**觉得不错点个 **“赞”**、“在看”，支持下小编****![图片](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**