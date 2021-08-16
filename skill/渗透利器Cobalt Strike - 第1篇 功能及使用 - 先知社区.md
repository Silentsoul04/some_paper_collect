> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/3975)

### Cobalt Strike 介绍

[Cobalt Strike](https://www.cobaltstrike.com): C/S架构的商业渗透软件，适合多人进行团队协作，可模拟APT做模拟对抗，进行内网渗透。

本文讲解3.12版本，该版本支持了Unicode编码。

Cobalt Strike整体功能了解参考[MITRE ATT&CK™](https://attack.mitre.org/software/S0154/)

#### Cobalt Strike的C/S架构

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190123145322-92feca28-1edb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190123145322-92feca28-1edb-1.png)

*   客户端(Client GUI)
    *   团队成员使用的图形化界面
*   服务器(Team Server)
    *   控制 - Team Server是Cobalt Strike中所有payload的主控制器，与victim的所有连接`bind/reverse`都由Team Server管理。
    *   日志记录 - Cobalt Strike中发生的所有事件 保存在`logs`文件夹
    *   信息搜集 - 收集在后渗透阶段发现的、或攻击者在目标系统上用于登录的所有凭据`credentials`
    *   自定义脚本 - `cat teamserver`可看到该文件是一个简单的bash脚本(可根据自己要求修改) 调用Metasploit RPC服务`msfrpcd`并启动服务器`cobaltstrike.jar`

#### 基本步骤

*   (可选步骤)选取C2域名
*   (可选步骤)扩展Team Server - 选取或自定义一个C2通信配置文件[Malleable C2 profile](https://www.cobaltstrike.com/help-malleable-c2) 可设置有效的SSL证书等
*   (可选步骤)扩展Client功能 - 使用[AggressorScripts](https://github.com/search?q=Aggressor+Script)修改或扩展Cobalt Strike 3.* 的客户端功能
*   启动团队服务器Team Server
*   Client 登录Team Server
*   启动监听器Listener
*   生成payload
*   (可选步骤)对payload进行免杀 尽量避免被victim的杀毒软件报毒
*   使用任意途径以实现victim主机执行payload
*   对victim主机所在网络进行后渗透操作

#### 运行环境

*   Team Server 推荐运行环境
    *   Kali Linux 1.0, 2.0 – i386 and AMD64
    *   Ubuntu Linux 12.04, 14.04 – x86, and x86_64

*   Client GUI 运行环境
    *   Windows 7 and above
    *   macOS X 10.10 and above
    *   Kali Linux 1.0, 2.0 – i386 and AMD64
    *   Ubuntu Linux 12.04, 14.04 – x86, and x86_64

#### 扩展性

**CS的自带的某些功能可能已过时，但其扩展性是非常强大，利用好CS的扩展性十分有用！**

*   Team Server 的扩展性
    *   C2通信配置文件 - [Malleable C2 profile](https://www.cobaltstrike.com/help-malleable-c2)
        *   定义C2的通信格式，修改CS默认的流量特征，以对抗流量分析
        *   使用前强烈推荐使用团队服务器上的脚本对配置文件进行本地的单元测试以检查语法`./c2lint my.profile`
        *   每个Cobalt Strike团队服务器只能加载一个配置文件，如果需要多个配置文件，可以启动多个团队服务器，每个都有自己的配置文件，可从同一个Cobalt Strike客户端连接到这些服务器
    *   外部C2（第三方C2） - [External C2 (Third-party Command and Control)](https://www.cobaltstrike.com/downloads/externalc2spec.pdf)
        *   C2 over chat protocols
        *   C2 over network covert channels
        *   C2 trough database fields
        *   C2 trough synced locations ([owncloud](https://github.com/owncloud/core) / RSYNC etc.)
        *   ...
*   Client GUI 的扩展性
    *   [AggressorScripts](https://github.com/search?q=Aggressor+Script)脚本 - 修改或扩展Cobalt Strike 3.* 的客户端功能(可实现自定义菜单创建，日志记录，权限维持等)，参考官方的[Aggressor Script Tutorial and Reference](https://www.cobaltstrike.com/aggressor-script/index.html)
        *   具体的脚本文件`scriptName.cna` - [harleyQu1nn/AggressorScripts](https://github.com/harleyQu1nn/AggressorScripts)

### 启动团队服务器Team Server

执行`sudo ./teamserver`看到如下说明:

```
[*] Generating X509 certificate and keystore (for SSL)
[*] ./teamserver <host> <password> [/path/to/c2.profile] [YYYY-MM-DD]

    <host> is the (default) IP address of this Cobalt Strike team server
    <password> is the shared password to connect to this server
    [/path/to/c2.profile] is your Malleable C2 profile
    [YYYY-MM-DD] is a kill date for Beacon payloads run from this server
```

*   启动参数`./teamserver <host> <password> [/path/to/c2.profile] [YYYY-MM-DD]`
    *   1 - 必填参数`host` 本服务器外网IP/域名
    *   2 - 必填参数`password` Client GUI连接时需要输入的密码
    *   3 - 可选参数`Malleable C2 communication profile` 指定C2通信配置文件 该功能体现了CS的强大扩展性
    *   4 - 可选参数`kill date` 指定所有payload的终止日期

```
# 启动Team Server
 # team server 必须以 root 权限运行 以便于监听端口号为0–1023的listener
 # 默认使用50050端口 监听来自团队成员CS Client的连接请求
 # 例
 sudo ./teamserver this.CShost.com pAsSXXXw0rd
```

### Client登录团队服务器

macOS下启动CS Client GUI `./Cobalt Strike`

即`java -XX:ParallelGCThreads=4 -XX:+AggressiveHeap -XX:+UseParallelGC -jar cobaltstrike.jar $*`

团队成员A首次登录，需要输入连接信息(会自动保存为一个`team server profile`下次可直接登录)，登录到团队服务器Team Server。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190123153818-da3f3b38-1ee1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190123153818-da3f3b38-1ee1-1.png)

可通过`Cobalt Strike - > Preferences - > Team Servers`维护本地的登录信息配置文件的列表`team server profiles`。

### Client GUI 图形界面

#### 概览

> 注：使用`Aggressor Script`可以修改或扩展Cobalt Strike 3.* 的客户端功能和界面

当你没有手动加载过任何`Aggressor Script`时，登录后的Client GUI默认界面如下图：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190123162601-84826ff6-1ee8-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190123162601-84826ff6-1ee8-1.png)

#### 逐个讲解

*   顶部菜单 - CS的所有功能和配置 主要使用`View`和`Attack`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190126153208-7c862e0e-213c-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190126153208-7c862e0e-213c-1.png)

*   toolbar - 工具栏为经常使用的功能提供了一键入口（toolbar中所有功能都可以在顶部菜单找到 所以可设置不显示toolbar)  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20190126153004-32f6c0a0-213c-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190126153004-32f6c0a0-213c-1.png)
    
*   sessions和targerts - 管理被控的目标网络的会话和主机 (概览图中红色部分)共有3种不同的显示视图
    
*   `View`Tabs - 查看相关结果 (概览图中绿色部分)

### 顶部菜单Cobalt Strike

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190124191049-b44ec17c-1fc8-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190124191049-b44ec17c-1fc8-1.png)

#### VPN Interface

[VPN Pivoting - Cobalt Strike](https://www.cobaltstrike.com/help-covert-vpn)

#### Visualization

三种视图

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190126153838-650a059c-213d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190126153838-650a059c-213d-1.png)

#### Listeners

如下图，CS 3.12版 有8种Listener

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190124154832-725f8ab0-1fac-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190124154832-725f8ab0-1fac-1.png)

*   Listeners
    *   windows/beacon_dns/reverse_dns_txt
    *   windows/beacon_dns/reverse_http
    *   windows/beacon_http/reverse_http
    *   windows/beacon_https/reverse_https
    *   windows/beacon_smb/bind_pipe 即 [SMB Beacon](https://www.cobaltstrike.com/help-smb-beacon)
    *   windows/foreign/reverse_http
    *   windows/foreign/reverse_https
    *   windows/foreign/reverse_tcp
*   CS 3.13新增了1个Listener
    *   `windows/beacon_tcp/bind_tcp`
        *   用于P2P通信 且支持linuxSSH会话
        *   使用 `connect IP`命令控制等待连接的TCP Beacon
        *   使用`unlink`命令断开TCP Beacon会话

beacon - CS自带的Listeners

foreign - 配合外部Listeners，使其他远控软件能够控制CS中的victim主机

reverse - 表示victim中招后主动先发出请求,与Team Server上的对应的Listener监听的指定端口建立连接

#### Script Manager

`Cobalt Strike -> Script Manager`加载脚本  
`scriptName.cna`

如加载`ProcessColor.cna`后，可见修改了ps回显结果：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190126162416-c4e9f890-2143-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190126162416-c4e9f890-2143-1.png)

### 顶部菜单Attack

#### 基础知识

*   stager
    *   定义 - 本身是一段尽可能小的手工优化过的汇编程序 [参考](https://blog.cobaltstrike.com/2013/06/28/staged-payloads-what-pen-testers-should-know/)
    *   功能 - 连接到主控端，按照主控端的要求对大的payload (即 stage)进行下载、注入内存、执行
*   payload (即 stage)
    *   定义 - 理论上是一段任意大小的、与位置无关的代码，被stager执行
    *   功能 - 具体功能的实现，Cobalt Strike的许多攻击和工作流程都将payload用多个stage实现投递

payload staging - 指将payload用多个stage实现投递

[stageless payload artifact](https://blog.cobaltstrike.com/2016/06/15/what-is-a-stageless-payload-artifact/) - 可理解为包含了payload的"全功能"被控端程序

#### 功能讲解

*   Packages
    *   HTML Application - 生成(executable/VBA/powershell)这3种原理不同的VBScript实现的`evil.hta`文件
    *   MS Office Macro - 生成恶意宏放入office文件 非常经典的攻击手法
    *   Payload Generator - 生成各种语言版本的payload 便于进行免杀
    *   USB/CD AutoPlay - 生成利用自动播放运行的被控端文件
    *   Windows Dropper - 捆绑器可将任意正常的文件(如1.txt)作为Embedded File)，捆绑生成Dropper.exe (免杀效果差，很容易被杀软的行为分析引擎报毒)
    *   Windows Executable - 可执行文件 默认x86 勾选x64表示包含x64 payload stage生成了artifactX64.exe(17kb) artifactX64.dll(17kb)
    *   [Windows Executable (Stageless)](https://www.cobaltstrike.com/help-staged-exe) - Stageless 表示把包含payload在内的"全功能"被控端都放入生成的可执行文件beconX64.exe(313kb) beconX64.dll(313kb) becon.ps1(351kb)
*   Web Drive-by 基于Web的功能
    *   Manage - 管理当前Team Server开启的所有web服务(以下Clone Site等功能开启的)
    *   Clone Site - 克隆某网站 可使用JavaScript记录victim在生成的钓鱼网站的按键记录
    *   Host File - 在Team Server的某端口提供Web以供下载某文件，可选择response的MIME(推荐将文件放到github等"白域名"下以对抗流量分析)
    *   Scripted Web Delivery - 为payload提供web服务以便于下载和执行，类似于msf的[Script Web Delivery](https://www.rapid7.com/db/modules/exploit/multi/script/web_delivery).(推荐将文件放到github等"白域名"下以对抗流量分析)
    *   [Signed Applet Attack](https://cobaltstrike.com/help-java-signed-applet-attack) - 启动一个Web服务以提供自签名Java Applet的运行环境，浏览器会要求victim授予applet运行权限，如果victim同意则实现控制。该攻击方法已过时。[Java Self-Signed Applet (Age: 1.7u51)](https://blog.cobaltstrike.com/2014/01/21/obituary-java-self-signed-applet-age-1-7u51/)
    *   Smart Applet Attack - 自动检测Java版本并l利用已知的exploits绕过security sandbox.[CS官方称该攻击的实现已过时，在现代环境中无效](https://cobaltstrike.com/help-java-smart-applet-attack)
    *   System Profiler 用来获取系统信息:系统版本，Flash版本，浏览器版本等
*   Spear Phish - 鱼叉钓鱼邮件功能

### 顶部菜单View

*   Event log - 事件日志
    *   victim主机 上线等记录
    *   团队成员 登录/注销/交流
    *   直接在Event  
        log中输入内容并回车，则所有成员可见
    *   输入`/msg neo 123`则发给成员neo一条消息 内容为123
*   Web log - 所有Web服务的日志
    *   如`Clone site`功能克隆网站后可在此看到web访问日志及在网站中的按键记录。
    *   如`Host File`功能在Team Server的某端口提供Web以供下载某文件，可在此看到web访问日志。
*   Credentials - 显示所有已获取的victim主机的凭证
    *   如hashdump
    *   如Mimikatz
*   Downloaded files - 显示所有已下载的文件
*   Targets - 显示所有victim主机
*   Proxy Pivots - 查看代理信息
*   Applications - 显示victim主机的应用信息
    *   如 浏览器及具体版本
    *   如 操作系统及具体版本
*   Keystrokes - 查看目标windows系统的键盘记录结果
    *   窗口名称及该窗口下的键盘记录结果
*   Screenshots - 查看所有屏幕截图
    *   victim信息(user|computer name|pid|when)及图片
*   Script Console - 在此加载第三方脚本以增强功能：CS`3.*`版本只支持[AggressorScripts](https://github.com/search?q=Aggressor+Script)

### 顶部菜单Reporting

CS 3.12 可导出6种报告

*   Activity report - 活动报告：红队活动timeline
    
*   Hosts report - 主机报告：每个主机的Hosts, services, credentials, sessions
    
*   Indicators of Compromise - IoC报告：类似于威胁情报报告中的附录IoC信息，  
    内容包括:[Malleable C2 profile](https://www.cobaltstrike.com/help-malleable-c2)配置文件的流量分析、C2域名和ip、你上传的所有文件的MD5 hashes
    
*   Sessions report - 会话报告：红队活动的完整信息。它捕获每个session，该session的communication path(通信路径)，在该session期间放置在目标上的MD5 hashes，并提供红队活动的日志。
    
*   Social engineering report - 社会工程学报告：包括鱼叉钓鱼邮件及点击记录
    
*   Tactics, Techniques, and Procedures - 战术技术及相关程序报告：  
    报告内容是您的Cobalt Strike行动对应的 [MITRE ATT&CK™](https://attack.mitre.org/)Matrix，可看到对每种战术的检测策略和缓解策略。
    

如下图，报告可导出为MS Word或PDF文档(勾选即可对其中的Email和password打码)：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20190124162326-52607d96-1fb1-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190124162326-52607d96-1fb1-1.png)

* * *

顶级菜单Reporting下最后的两个选项:

*   Reset data - 清空数据 不可恢复
*   Export data - 导出源数据(生成的报告是从这些源数据中获取对应内容填充报告)：导出以下文件
    
    ```
    activity.tsv
    campaigns.tsv
    sentemails.tsv
    targets.tsv
    applications.tsv
    credentials.tsv
    services.tsv
    tokens.tsv
    c2info.tsv
    events.tsv
    sessions.tsv
    webhits.tsv
    ```
    

### 对目标主机进行操作

当有victim主机以任何方式运行了生成的被控端，出现在主机列表，选中要操作的目标主机，右键`interact`进入交互命令界面，在此使用Beacon Commands对victim主机执行各种操作。

Beacon Commands是最全的，包含了图形化的控制功能。

#### Beacon Commands

```
beacon> help

Beacon Commands
===============

    Command                   Description
    -------                   -----------
    browserpivot              Setup a browser pivot session
    bypassuac                 Spawn a session in a high integrity process
    cancel                    Cancel a download that's in-progress
    cd                        Change directory
    checkin                   Call home and post data
    clear                     Clear beacon queue
    covertvpn                 Deploy Covert VPN client
    cp                        Copy a file
    dcsync                    Extract a password hash from a DC
    desktop                   View and interact with target's desktop
    dllinject                 Inject a Reflective DLL into a process
    dllload                   Load DLL into a process with LoadLibrary()
    download                  Download a file
    downloads                 Lists file downloads in progress
    drives                    List drives on target
    elevate                   Try to elevate privileges
    execute                   Execute a program on target (no output)
    execute-assembly          Execute a local .NET program in-memory on target
    exit                      Terminate the beacon session
    getprivs                  Enable system privileges on current token
    getsystem                 Attempt to get SYSTEM
    getuid                    Get User ID
    hashdump                  Dump password hashes
    help                      Help menu
    inject                    Spawn a session in a specific process
    jobkill                   Kill a long-running post-exploitation task
    jobs                      List long-running post-exploitation tasks
    kerberos_ccache_use       Apply kerberos ticket from cache to this session
    kerberos_ticket_purge     Purge kerberos tickets from this session
    kerberos_ticket_use       Apply kerberos ticket to this session
    keylogger                 Inject a keystroke logger into a process
    kill                      Kill a process
    link                      Connect to a Beacon peer over SMB
    logonpasswords            Dump credentials and hashes with mimikatz
    ls                        List files
    make_token                Create a token to pass credentials
    mimikatz                  Runs a mimikatz command
    mkdir                     Make a directory
    mode dns                  Use DNS A as data channel (DNS beacon only)
    mode dns-txt              Use DNS TXT as data channel (DNS beacon only)
    mode dns6                 Use DNS AAAA as data channel (DNS beacon only)
    mode http                 Use HTTP as data channel
    mode smb                  Use SMB peer-to-peer communication
    mv                        Move a file
    net                       Network and host enumeration tool
    note                      Assign a note to this Beacon       
    portscan                  Scan a network for open services
    powerpick                 Execute a command via Unmanaged PowerShell
    powershell                Execute a command via powershell.exe
    powershell-import         Import a powershell script
    ppid                      Set parent PID for spawned post-ex jobs
    ps                        Show process list
    psexec                    Use a service to spawn a session on a host
    psexec_psh                Use PowerShell to spawn a session on a host
    psinject                  Execute PowerShell command in specific process
    pth                       Pass-the-hash using Mimikatz
    pwd                       Print current directory
    reg                       Query the registry
    rev2self                  Revert to original token
    rm                        Remove a file or folder
    rportfwd                  Setup a reverse port forward
    run                       Execute a program on target (returns output)
    runas                     Execute a program as another user
    runasadmin                Execute a program in a high-integrity context
    runu                      Execute a program under another PID
    screenshot                Take a screenshot
    setenv                    Set an environment variable
    shell                     Execute a command via cmd.exe
    shinject                  Inject shellcode into a process
    shspawn                   Spawn process and inject shellcode into it
    sleep                     Set beacon sleep time
    socks                     Start SOCKS4a server to relay traffic
    socks stop                Stop SOCKS4a server
    spawn                     Spawn a session 
    spawnas                   Spawn a session as another user
    spawnto                   Set executable to spawn processes into
    spawnu                    Spawn a session under another PID
    ssh                       Use SSH to spawn an SSH session on a host
    ssh-key                   Use SSH to spawn an SSH session on a host
    steal_token               Steal access token from a process
    timestomp                 Apply timestamps from one file to another
    unlink                    Disconnect from parent Beacon
    upload                    Upload a file
    wdigest                   Dump plaintext credentials with mimikatz
    winrm                     Use WinRM to spawn a session on a host
    wmi                       Use WMI to spawn a session on a host
```

如执行cmd命令`shell ifconfig` 更多命令说明请看附件。

#### Beacon Commands的细节和缺点

保证操作安全OPSEC - [了解Beacon Commands实现原理](https://blog.cobaltstrike.com/2017/06/23/opsec-considerations-for-beacon-commands/)

screenshot只能截取x86进程的窗口截图（x64无效）

等等

#### 界面截图

*   键盘记录  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20190126172452-3c7bfce8-214c-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190126172452-3c7bfce8-214c-1.png)

*   远程桌面操作(VNC) - 考验网速的时候到了

[![](https://xzfile.aliyuncs.com/media/upload/picture/20190126172342-127ceaa6-214c-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190126172342-127ceaa6-214c-1.png)

等等

### 总结

Cobalt Strike是一款扩展性强、功能强大的渗透软件，值得研究。