> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VH8XBueLuA4bSbTfV-FNOg)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fuBhZCW25hNtiawibXa6jdibJO1LiaaYSDECImNTbFbhRx4BTAibjAv1wDBA/640?wx_fmt=png)

扫码领资料

获黑客教程

免费 & 进群

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFJNibV2baHRo8G34MZhFD1sjTz4LHLiaKG9208VTU6pdTIEpC9jlW6UVfhIb9rHorCvvMsdiaya4T6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png)

Gargamel
--------

Gargamel 是一款基于 Rust 开发的信息安全取证工具，**广大研究人员可以使用 Gargamel 来完成日常的信息取证任务。**

工具下载
----

广大研究人员可以使用下列命令将该项目源码克隆至本地：

```
git clone https://github.com/Lifars/gargamel.git
```

项目编译
----

假设你已经在本地设备上安装并配置好了 Rust v1.41+，打开终端窗口，并切换到项目目录下，输入下列命令即可编译项目：

```
cargo build --release
```

我们可以使用下列命令编译调试构建：

```
cargo build
```

已编译好的可执行文件可以在 target/release/gargamel.exe 或 target/debug/gargamel.exe 路径下找到。

设置日志等级
------

我们可以按照下列方式修改工具的日志记录等级：

> 打开 src/main.rs；
> 
> 在第 42 和 43 行，将 LevelFilter::Info 修改为 LevelFilter::Trace 即可查看更多详细日志信息；
> 
> 注意，LevelFilter::Trace 将会记录下包括密码在内的所有内容；

用户指南
----

现在，这款应用程序仅支持在 Windows 系统上运行，目标设备必须是 Windows 或 Linux 系统。

你还需要确保下列程序已经存储在了跟 Gargamel 相同的目录之中：

> psexec：
> 
> 【https://docs.microsoft.com/en-us/sysinternals/downloads/psexec】
> 
> paexec：【https://www.poweradmin.com/paexec/】
> 
> winpmem：【https://github.com/Velocidex/c-aff4/releases】
> 
> plink 和 pscp：【https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html】
> 
> SharpRDP：【https://github.com/vildibald/SharpRDP/releases/tag/v1.0.0】
> 
> WMImplant：【https://github.com/vildibald/WMImplant】
> 
> exe：【https://www.7-zip.org/download.html】

Gargamel 的使用
------------

Gargamel 需要在具备高级权限的终端中启动才能完全发挥其功能。目前，它不支持 UAC 对话框，也不支持在有限权限下运行时的任何类型的通知。当以有限的用户权限运行时，一些操作（如目标内存转储）将不起作用。

### 基础使用

假设你想要连接到一台带有下列参数的计算机：

> 地址：192.168.42.47
> 
> 用户名：Jano
> 
> 密码：nbusr123

下列命令将利用 PsExec 方法获取防火墙状态、网络状态、登录用户、运行进程、活动网络连接、注册表、系统 & 应用事件日志。获取到的取证信息将存储在 Gargamel 的 testResults 目录下：

```
gargamel.exe -c 192.168.42.47 -u Jano --psexec -o testResults
```

Gargamel 将会询问输入远程用户的密码，我们这里的密码为 nbusr123。注意，密码在输入过程中是隐藏的。

我们还可以直接在命令行参数中指定用户名和密码：

```
gargamel.exe -c 192.168.42.47 -u Jano --psexec -p nbusr123 -o testResults
```

### 域使用

假设你想要连接到域中一台带有下列参数的计算机：

> 域：WORKSPACE
> 
> 计算机名：JanovPC
> 
> 用户名：Jano
> 
> 密码：nbusr123

下列命令将利用 PsExec 方法获取防火墙状态、网络状态、登录用户、运行进程、活动网络连接、注册表、系统 & 应用事件日志：

```
gargamel.exe -c JanovPC -u Jano -d WORKSPACE --psexec -o testResults
```

或者，直接在命令行参数中指定目标设备信息：

```
gargamel.exe -c JanovPC -u Jano -d WORKSPACE --psexec -p nbusr123 -o testResults
```

### 其他连接方式

PsExec 是其中一种支持的连接方法，我们可以将 --psexec 替换为下列选项：

> --psexec
> 
> --psrem
> 
> --rdp
> 
> --wmi
> 
> --ssh

我们也可以一次使用多种方法。比如说，同时使用 PsExec 和 RDP：

```
gargamel.exe -c 192.168.42.47 -u Jano --psexec --rdp -o testResults
```

### 获取内存

为了获取内存导转储，可以直接在参数后添加 - m 选项：

```
gargamel.exe -c 192.168.42.47 -u Jano --psexec -o testResults -m
```

如果你只需要获取内存转储而不需要其他取证信息，可以直接使用下列命令：

```
gargamel.exe -c 192.168.42.47 -u Jano --psexec -o testResults -m --no-events-search --no-evidence-search --no-registry-search
```

这个功能目前仅支持目标为 Windows 系统的主机。

### 运行自定义命令

Gargamel 可以在远程主机中运行自定义 Windows CMD 或 Linux Shell 命令。

我们需要使用下列内容创建一个 custom-commands.txt 文件：

```
# Will be run using any method

ipconfig

# Will run only when launching with at least one of --all, --psexec, --wmi methods

:psexec:wmi ipconfig -all
```

接下来，我们就可以使用 - e 选项来运行上述命令了：

```
gargamel.exe -c 192.168.42.47 -u Jano --psexec -o testResults -e custom-commands.txt
```

### 下载自定义文件

Gargamel 能够下载远程文件，首先我们需要使用下列内容创建一个 custom-files.txt 文件：

```
C:\Users\Public\sss*

C:\Users\Jano\danove.pdf

# This line and the next one will be ignored

# C:\Users\Jano\somBajecny.pptx  
```

接下来，我们就可以使用 - s 选项来运行上述命令了：

```
gargamel.exe -c 192.168.42.47 -u Jano --psexec -o testResults -s custom-files.txt
```

所有选项
----

```
USAGE:

    gargamel.exe [FLAGS] [OPTIONS] --user <user>

 

FLAGS:

    -a, --all                   Acquire evidence from Windows machine using all supported methods (PsExec, PsRemote,

                                WMI, RDP).

        --no-events-search      Disables Windows event logs acquisition.

        --no-evidence-search    Disables acquisition of evidence that can be usually downloaded quickly (like ipconfig,

                                firewall status etc..)

        --no-registry-search    Disables target registry acquisition.

    -h, --help                  Prints help information

    -m, --mem-image             Optional: Memory dump of a target Windows machine.

        --local                 Acquire evidence from local machine.

        --nla                   Optional: Use network level authentication when using RDP. (Windows targets only)

        --no-7z                 Optional: Disable 7zip compression for registry & memory images.This will significantly

                                decrease the running time, but WMI and RDP connections will probably not work properly.

                                    (Windows targets only)

        --psexec                Acquire evidence from Windows machine using PsExec. Requires both PsExec64.exe and

                                paexec.exe in the current directory or in the path.

        --psrem                 Acquire evidence from Windows machine using PowerShell. Requires both PsExec64.exe and

                                paexec.exe in the current directory or in the path.

        --rdp                   Acquire evidence from Windows machine using RDP. Requires SharpRDP.exe in the current

                                directory or in the path.

        --ssh                   Acquire evidence from Linux machine using SSH. Requires both plink.exe and pscp.exe in

                                the current directory or in the path.

    -V, --version               Prints version information

        --wmi                   Acquire evidence from Windows machine using WMI. Requires WMImplant.ps1 in the current

                                directory or in the path and PowerShell 3.0+ on the host machine.Note: It is necessary

                                to disable Windows Defender real-time protection (other AVs not tested).

 

OPTIONS:

    -c, --computer <computer>                        Remote computer address/name. [default: 127.0.0.1]

    -u, --user <user>                                Remote user name

    -d, --domain <domain>                            Optional: Remote Windows domain

    -o, --output <local-store-directory>

            Name of local directory to store the evidence [default: evidence-output]

 

    -p, --password <password>

            Optional: Remote user password. Skipping this option will prompt a possibility to put a password in hidden

            way.To specify an empty password use `-p ""`

 

        --redownload <re-download>

            Optional: Download and DELETE specified file from target computer. Use this in case of previous failed

            partially completed operation. For just downloading a file (without deleting it) please use a `search`

            switch. If you specify a 7zip chunk (.7z.[chunk-number], e.g. .7z.004), then it will also automatically try to

            download subsequent chunks.Use also with --psexec --psrem, --rdp, --wmi, --all

 

    -r, --remote-storage <remote-store-directory>

            Name of remote directory to be used as a temporary storage. (Windows targets only) [default:

            C:\Users\Public]

 

    -e, --commands <custom-command-path>             Optional: File with custom commands to execute on remote computer

 

    -s, --search <search-files-path>

            Optional: File with files names to be searched on remote computer. File names supports also `*` and `?`

            wildcards on file names (but not yet parent directories).

 

        --key <ssh-key>                              Optional: Name/path of SSH private key file. (Linux target only)

 

        --timeout <timeout>

            Optional: Timeout in seconds for long running operations.This option is a workaround for a bug in

            WMImplant.ps1 amd SharpRDP.exe where finishing of a long running operation cannot sometimes properly close

            the connection leaving the Gargamel in seemingly frozen state or executing the next operation with the

            previous one unfinished on target site.Increasing this timeout may solve issues when acquiring registry or

            memory image from target machine. [default: 300]
```

存在的问题
-----

> WMI 无法将输出写入至包含 “_” 符号的路径 / 文件名中。

项目地址  

-------

**Gargamel：https://github.com/Lifars/gargamel**

转自：https://www.freebuf.com/articles/system/265703.html

```
学习更多黑客技能！体验靶场实战练习


（黑客视频资料及工具）


往期回顾
```

[

实战 | 看我如何干掉裸聊 APP

2021-03-14

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSGiaCSfHWIuyhV56kVtRhwVS3h7DDG8y7lrV6y1ia1CTNLHgM2ibtIatqFicdynEzAFydcwhSP5ibzv89Q/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247511422&idx=1&sn=3305fe35eafe6f14523dbfd854c9e64f&chksm=ebeae453dc9d6d45b9ca7cb18e12d3611e0b46825939445487c097d376469b54fbe2a8e794a9&scene=21#wechat_redirect)

[

无线渗透！菠萝派一个 wifi 神器

2020-09-28

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEcvsPiaIBuGf3z4bibVd7ibuUkoo3QkN1yWGgwq5CgaUtWG1OeodDmCEfYx9J2Fnbm9lPZGf7RL4kfA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247494169&idx=1&sn=239349214c12ff53d1e3e17750ae6771&chksm=ebeaa334dc9d2a22dde2c04b29cf344a5faead28f731a4a7dd57c633a5a04fc94d02f9304722&scene=21#wechat_redirect)

[

get 新技能 ! 用手机也能秒变黑客

2020-08-21

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSGFX06lQLY3jibA59V5OdTbg18IGvico5VZdRImsFOf24FD0cibd3yQm2DnXyeRhbuR2UPIFRkic2WSXA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247492144&idx=1&sn=496cbb580cd168e6eb1110dbc13b4730&chksm=ebeaab1ddc9d220b89caf041855d799671dfc21b7f490c0f1b40b4f6b796cf1c4bb47e7b11b9&scene=21#wechat_redirect)

[

一波三折，成功拿下迷妹学校的服务器

2020-08-12

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHictAKXtqpskZBAxGqDF1QRa3exaM1aU6dPrkVASq2KJ8icnZpbnXNZuPnL0n7qmpfOOL9b9BvfxGA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247491526&idx=1&sn=f542f8f5164d0803b0fd884e2fa63038&chksm=ebe956ebdc9edffd85b532bd430156f7734ea2484d65265a78cc776ebdfd3911b230ec4712c2&scene=21#wechat_redirect)

[

骚操作 ！花式钓鱼攻击实战教程

2020-07-14

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSF2ibTjhCf06zKtZqZAFPVUtW0lPzDj3R3cBl9ZFiacw9o9mCK9XaGEeupRGW8MxENCJcFicvQPKld0g/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247489488&idx=1&sn=7fc630e36df3a3c28b378b385b95df07&chksm=ebe95efddc9ed7ebafa5d47a72d4e0ef75811b5dc504321f5301f47f7a4c6b0a3e777ad601b4&scene=21#wechat_redirect)

[

快嫖！免费的文件恢复工具

2020-06-28

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFqiccBZEDqkkyLcicltIcl6dSMtPr8QPye0M54PmTMxar9sPq9lSicicvHo4t21QuOgM7KpwjHuGd7PA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247488671&idx=2&sn=1845e9dad614168819ecaab7098ac133&chksm=ebe95db2dc9ed4a4fea93a7e93e9782af5f567814c0665eb3c058824b5afdfc5dda2769f9a0b&scene=21#wechat_redirect)