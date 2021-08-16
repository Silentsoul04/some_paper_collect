> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BaFP7CtUCKxmkcADTkIjew)

**高质量的安全文章，安全 offer 面试经验分享**

**尽在 # 掌控安全 EDU #**

**作者：掌控安全 - mss**

### 一. 网络相关指令

```
ipconfig -------------------- 查看本机ip
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NlQiaTeuFdiajd99KxGvBlhh50Bdpud8bJCj76ZicBRyibyAcgRSE8OjwUg/640?wx_fmt=png)

```
ipconfig /all -------------------- 查看ip地址等网卡配置
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NfWjFxg11tQjsnwk9gr7aUqAKybydVus7Bdo67vwZrnqicQNhHqGmAgg/640?wx_fmt=png)

```
ipconfig /flushdns ---------------- 清除本地 DNS 缓存
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NIk27eWQRdwq9qoHQdEicUIicwKySMBgXRFNVcgR2QXjXzxMZWXGX8xPw/640?wx_fmt=png)

```
ping 目标ip地址 ---------------- 检测主机是否能通信
（比如ping一下百度 ping www.baidu.com）
ping ip/域名 ---------------- 查看访问该ip/域名的延迟和丢包率  
ping ip/域名 -n 5 ---------------- Ping ip/域名 5 次  
ping ip/域名 -t ---------------- 一直ping 一个ip/域名  
tracert ip/域名 ---------------- 路由追踪
(路由追踪Tracert主要是用来确定ip报文访问目标的时候所经过的路径,比如你的主机去ping百度的服务器，那么你的报文经过了那几个路由器，都是可以通过tracert去确定的)
telnet ip/域名 端口号 ----------- 测试远程主机端口是否能够正常通信 
(使用前先在Windows功能里面开启"Telnet客户端"或者"Telnet Client"这个功能 例如：telnet 192.168.0.55 8080)
wf.msc ----------------  设置防火墙规则 
(会打开防火墙设置窗口)
```

```
whoami -------------------- 查看当前用户及权限
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NLQLou1CIyr1C9DCMnJSkzuicXbpibAiaibPVBjZ2q9DZiaAVgCU29klzVQA/640?wx_fmt=png)

```
systeminfo -------------------- 查看计算机信息（版本，位数，补丁情况）
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NYG61ViatgZ3MkRgybu4rhmytasUrpMMqW8mDoC8f2rxnJKH0g2MggAA/640?wx_fmt=png)

```
ver -------------------- 查看计算机操作系统版本
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NSjjEPFwykkcJic9S0aW1QB4A13ZOUNcibcMUrkIWlOZV9DooafOqEb4A/640?wx_fmt=png)

### 二. 用户和组

```
net user -------------------- 查看当前系统有哪些用户
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1N5xnFv6qn8OCNib9m4thddgJibJ7eS5A8ZYWkLOCBOUicKOGficVNyOjLYA/640?wx_fmt=png)

```
net user 用户名 -------------------- 查看用户的基本信息，所属组
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1N7ktyCG4St0YaIe8rcYj2B2jxAgjfjuZJ8iatCHw1sDVO1Ux4vBQia6Bw/640?wx_fmt=png)

```
net user 用户名 密码 /add -------------------- 添加新用户并设置密码
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NUY5zhhr17jFoHBMoSIgG8fftE3GIAum1h2J63778jVicWHXudyZy3aA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NNwvgafpDqgXF2MT54BSzqztzxSGPE8XoSHKr54jOkXwGPu6IJY2X8Q/640?wx_fmt=png)

```
net user 用户名 /del -------------------- 删除用户
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NxZ7pFAc3ha7zfKl1QWP0BRia3hsmA3XGddSOibcv2URgwYZRVOlW9OWA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1Nl83JmPd6AILRFFkJx9rnkCZf254k7r1aUXUn2R8xWbxsuulzgL18Ig/640?wx_fmt=png)

```
query user -------------------- 查看当前在线的用户
(仅Windows Server支持)
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1Npjs9AuYK9D1sWToG5BibOqaib6zaBxicxSYokPmeGeiazOYHUkoYuzd0dQ/640?wx_fmt=png)

```
net localgroup -------------------- 查看所有的本地组
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NUUOqaV3gbpcIficvKZxGSspo3e3anVGMcVpf9kYcJdy1vKuserG0ySQ/640?wx_fmt=png)

```
net localgroup administrators -------------------- 查看administrators组中有哪些用户
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NHfhBKMLxm8iagkQ1kScXiaa2v4G4icKib03z3QOoOb6UibFcEgDKyM3BAoQ/640?wx_fmt=png)

```
net localgroup administrators lisi /add -------------------- 将用户lisi添加到本地管理员(administrators)组
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NVTn7gugwvcrNIWKlX6OtuqART4Cy27SGU3zj8ew7PTiaRWZDecoAUVg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NnuxAEeuG2icz4Sm4fymIibxRBBXeIQOhkcgribvHDK17FK4CCzBeIotSw/640?wx_fmt=png)

```
net user /domain -------------------- 该参数仅在 Windows NT Server 域成员的 Windows NT Workstation 计算机上可用。由此可以此判断当前用户是否是域成员。
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1N2AbWFQRUicWEOXN6ZyZSR0CbOUNEpqfYby4hg7nu3ibcuibWpFbruWRIw/640?wx_fmt=png)

如果用户在域成员中时可使用一下命令：

> netstat -aon|findstr “80” —————————— 查看 80 端口
> 
> tasklist |findstr “4680” —————————— 查看 4680 进程
> 
> net group /domain —————————— 查看域中的组
> 
> net group “组名” /domain —————————— 查看域组”Domain Users” 中的用户成员

### 三. powershell

`powershell`可以在渗透中提供强大的助力，下面这些脚本使用的时候记得修改 ip 地址

扫描存活 ip，最前面的`1..255`是 ip 地址的 d 段，最后范围是 192.168.0.1-255，判断和修改方式下同

```
1..255 | % {echo "192.168.0.$_"; ping -n 1 -w 100 192.168.0.$_} | Select-String ttl
```

判断主机类型，根据 ttl 值判断，范围 192.168.0.1-255

```
1..255 | % {echo "192.168.0.$_"; ping -n 1 -w 100 192.168.0.$_} | Select-String ttl |% { if ($_ -match "ms") { $ttl = $_.line.split('=')[2] -as [int]; if ($ttl -lt 65) { $os = "linux"} elseif ($ttl -gt 64 -And $ttl -lt 129) { $os = "windows"} else {$os = "cisco"}; write-host "192.168.0.$_ OS:$os" ; echo "192.168.0.$_" >> scan_results.txt }}
```

扫描端口

```
24..25 | % {echo ((new-object Net.Sockets.TcpClient).Connect("192.168.1.119",$_)) "Port $_ is open!"} 2>$null24..25 |% {echo "$_ is "; Test-NetConnection -Port $_ -InformationLevel "Quiet" 192.168.1.119}2>null
```

扫描指定端口的 ip

```
foreach ($ip in 1..20) {Test-NetConnection -Port 80 -InformationLevel "Detailed" 192.168.0.$ip}
```

下载文件

```
start powershell ---------------- 启动powershell$client = new-object System.Net.WebClient$client.DownloadFile('#1', '#2') ---------------- #1填写文件的下载地址，#2填写保存文件的路径和保存文件的文件名和文件类型$client.DownloadFile('https://i.zkaq.org/','D:\zkaq\1.txt')下载https://i.zkaq.org/网页到D盘的zkaq文件夹中的1.txt文件中（1.txt不存在会自动创建）保存的文件类型可以是任意文件类型
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NuUaC7IfiaoIa2BfvFTj83jRogklaMGicLeXXQKCRwMAxjUZvW0wpBeHA/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NNZKicW8KnznVIT7vP0ss5kHiaeSibRdEcCicHkZtYtX1hry2Qic1svDUhhA/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1Nic0ribIMl08KFibwPZT61kgtoed9g6tpkshemGghLhCQUjuBfaferR4JQ/640?wx_fmt=png)

### 四. telnet

`telnet`常规使用是和服务器建立连接，也开业用来探测端口是否开放

用法：`telnet 主机 端口`，如：`telnet dc 3389`。

注意：不是所有机器都安装了此服务。

### 五. wmic

WMIC 扩展 WMI（Windows Management Instrumentation，Windows 管理工具） ，提供了从命令行接口和批命令脚本执行系统管理的支持。

在 cmd 中有些时候查到的数据不全，如某些进程的 pid，这时可以使用 wmic 进行操作，WMIC 提供了大量的全局开关、别名、动词、命令和丰富的命令行帮助增强用户接口。

`wmic startup get command,caption`, 查看启动程序信息

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NVp4xFlqmjPibbicds8HgLmDBK8NrKoKloRvtNDwJoVLXfxLKnWILGrQA/640?wx_fmt=png)  
`wmic service list brief`, 查询本机服务信息

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NsXMfANQaMc1jHTpxicylv8ecL8GaXlV83y4MRA0CwjncDm0AnJATztQ/640?wx_fmt=png)  
还可以使用`tasklist` 查询进程信息  
`schtasks /query /fo LIST /V`, 查看计划任务

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1Nybkwn5teJzStMjSAfqkQRibMfGapYIrxib7hBKoaNVerswMrfEcibVaZg/640?wx_fmt=png)  
`netstat -ano`查看端口列表

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NEhkgn6fQOos75slnv2OFazHx3ApGbT3V1WRcrgBBic3llTMakrric4Rw/640?wx_fmt=png)  
注意，一般查看进程端口，先查进程 pid，在根据 pid 查端口

### 六. 文件和文件夹

#### **文件夹**

```
d: ------------------  进入d盘  
cd /d c:/test ------------------ 切换磁盘和目录
 （进入 c 盘的 test 文件夹） 
cd \test1\test2------------------  进入文件夹（进入 test2 文件夹） 
cd \ ------------------ 返回根目录  
cd .. ------------------ 回到上级目录  
md test ------------------ 新建文件夹test
dir ------------------ 显示目录中文件列表   
tree c:\test ------------------ 显示目录结构（
c 盘 test 目录）
cd ------------------ 显示当前目录位置  
cd d: ------------------ 切换指定磁盘的当前目录位置
dir -------------- 查看当前目录文件
（类似于linux下的ls命令，如果是需要查看隐藏文件的或者更多操作的话，可以使用dir /?来查看其它用法）
md 文件夹名----------  创建文件夹（目录名）
rd 文件夹 -------------- 删除文件夹
copy 路径\文件名 路径\文件名 ------------------  复制文件
（把一个文件拷贝到另一个地方）
move 路径\文件名 路径\文件名 ------------------ 移动文件
 把一个文件移动（就是剪切+复制）到另一个地方 
del 文件名 ------------------ 删除文件
 （这个是专门删除文件的，不能删除文件夹）
```

```
cd. > 文件名.txt ---------------- 创建空文件 （在当前目录写入一个空txt文件）cd. > a.txtcd. 表示改变当前目录为当前目录，即等于没改变；而且此命令不会有输出。> 表示把命令输出写入到文件。后面跟着a.txt，就表示写入到a.txt。而此例中命令不会有输出，所以就创建了没有内容的空文件。
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NF4YQVfusH62NvtvwxaOaX7y17OIgoF5d671Zqn34ZjOWN7MTxt3DQg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NYtiaExPcSanK7a6CJIichw0e2Yic61YmC1QuKQciaVR9X8JgvMkgic3pWfw/640?wx_fmt=png)

#### **文件**

```
cd D:\文件夹名 > 文件名.txt ---------------- 在文件夹中创建空文件（在D盘的文件夹下写入一个空txt文件）cd D:\zkaq > a.txt在D盘zkaq文件夹写入空文件 a.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NIckw74o34IA9BlyhM4PQu1t50pbSmQSeR9nrvRPJho7YfvwzWgaloA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NMHHtbZ7ZmK0YGcWLE9ORJgluA1vfzLiah1FX0GNmUdyKxWpAyrkaYFA/640?wx_fmt=png)

```
echo abc > 1.txt ---------------- 写入文本 （在当前目录的1.txt文件中写入文本abc,覆盖1.txt原内容）echo 输出echo abc 输出abc输出abc文本到当前目录的1.txt文件中，如果1.txt文件不存在则会创建1.txt文件
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NovNG9vSqZTkhpu7J2hAfqMT3qiboialw6mYyZ1FJ6zDGmfHuN3HBGMgw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1N35c5RTqPUtLPl2PU3ib3GibS7Il1NWWR0xkD2ia6fXLDUF68oLNe1ZcKQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NGVcjkgmiaWYjZJUHkCwk9Qhah6QvB52PXBOvOF65rZXWBFmGlmWIrkw/640?wx_fmt=png)

```
echo 123 >> 1.txt ---------------- 文本追加写入 （在当前目录的1.txt文件中追加写入文本123，不覆盖1.txt原内容）>> 追加
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NHwJejyIHoM57qjhicSqcZdyI0a51qA3NbQqcuJwFjjj4yqM2t7wwhcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1N461bcG9EeluTuWKR79UN7rPAS0H96p6NsRnhiaG7fu3u5IZNI8pficDg/640?wx_fmt=png)

```
echo 12345 > 8.txt:9.txt ---------------- 创建隐藏文件9.txt（把文本内容12345写入到当前目录依赖于8.txt文件的9.txt文件中，被依赖文件8.txt文件不存在会自动创建）: 冒号后面的文件可以是任意文件类型，包括文件夹
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NiaicxSgTIe7cmy6B3XxoJxGFK6MHIHByx5FwQrHibYCApzia96ByKxNfEw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1Nic5R2hAJ54KbZibAcVW3sZEM2ZNpQVnlrLUu9eIQeI8iaYeMkouI4XsjA/640?wx_fmt=png)

```
notepad 8.txt:9.txt ---------------- 查看隐藏文件9.txtnotepad 打开记事本
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NsnwGKhq0oMibibKFYhTd0AxmBiaH6MPCibVwzAwZ1khsbkcVPmI8IiaaiaWQ/640?wx_fmt=png)

```
copy C.txt/b+B.txt D.txt ---------------- 复制文件 (把B.txt文件和C.txt中的内容合并复制到D.txt中，D.txt不存在会自动创建，文件类型可以是任意文件类型)copy 命令的作用是将一个或多个已存在的文件复制到其他位置，或者将多个文件合并为一个文件，或者创建一个批处理文件。
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1Nf33fyBepCnzJCqN1LRoS0pf6MFbBFkLNIxcWoaNjKWZz2cU1IDUTiaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NhibpItddgfaouticyywicNLLoxjPtSyxoc7iaWBBo6sic7JUxgsMXiaanviaQ/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NpgKd0hGPuza14iaGJ41N50ibw6fctUN34xLiaibEk4WTB4OFF5VD1zoiaaw/640?wx_fmt=png)

```
del D.txt ---------------- 删除文件D.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NuXQQok1dAIfqKMSadepickuz0kT8ZflXOP7eUfzcxdWhkeVSNZNJpKA/640?wx_fmt=png)

```
type B.txt ---------------- 读取文件 读取当前文件夹中的B.txt文件的内容
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqiaibaTLWLIdUIlt0D5xZb1NaCv7lUTpickiaNMuiaIlJshictnxWAocfU3B8drYJ6J0ntPBhu1jribZxJQ/640?wx_fmt=png)

### 七. CMD 主机管理命令

```
shutdown /s ---------------- 关机 
 shutdown /r ---------------- 重启  
 shutdown /L ---------------- 注销(小写的L也可以) 
 shutdown /h /f ---------------- 休眠  
 shutdown /a ---------------- 取消关机  
 shutdown /s /t 3600 ---------------- 定时关机（3600 秒后关机） 
 cls ----------------  清除CMD的屏幕(类似于linux下的clear) 
help ----------------  查看命令帮助
(使用这个命令之后，我们可以看到所有的dos命令，并且后面还有中文的解释，这样我们就可以根据自己的需求要找到想要使用的命令。)
```

### 八. CMD 进程相关操作命令

```
tasklist ---------------- 显示当前正在运行的进程  
start 程序名/程序所在路径 ---------------- 运行程序或命令 
taskkill /im 进程名.exe ---------------- 按名称结束进程
 （taskkill /im notepad.exe 关闭记事本）
taskkill /pid号 ---------------- 按PID结束进程
 (staskkill /pid 1234 关闭 PID 为 1234 的进程）
```

### 九. CMD 其他基础命令

```
gpedit.msc-----组策略 
 Nslookup-------IP地址侦测器 
（是一个 监测网络中 DNS 服务器是否能正确实现域名解析的命令行工具。）
 explorer-------打开资源管理器 
 lusrmgr.msc----本机用户和组 
 services.msc---本地服务设置 
 notepad--------打开记事本 
 cleanmgr-------垃圾整理 
 net start messenger----开始信使服务 
 compmgmt.msc---计算机管理 
 net stop messenger-----停止信使服务 
 conf-----------启动netmeeting
 dvdplay--------DVD播放器 
 charmap--------启动字符映射表 
 diskmgmt.msc---磁盘管理实用程序 
 calc-----------启动计算器 
 chkdsk.exe-----Chkdsk磁盘检查 
 devmgmt.msc--- 设备管理器 
 rononce -p----15秒关机 
 dxdiag---------检查DirectX信息 
 regedt32-------注册表编辑器 
 Msconfig.exe---系统配置实用程序 
 rsop.msc-------组策略结果集 
 regedit.exe----注册表 
 progman--------程序管理器 
 winmsd---------系统信息 
 perfmon.msc----计算机性能监测程序 
 winver---------检查Windows版本 
 sfc /scannow-----扫描错误并复原 
 taskmgr-----任务管理器（2000/xp/2003） 
 wmimgmt.msc----打开windows管理体系结构(WMI) 
 wupdmgr--------windows更新程序 
 wscript--------windows脚本宿主设置 
 write----------写字板 
 wiaacmgr-------扫描仪和照相机向导 
 mspaint--------画图板 
 mstsc----------远程桌面连接 
 magnify--------放大镜实用程序 
 mmc------------打开控制台 
 mobsync--------同步命令 
 iexpress-------木马捆绑工具，系统自带 
 fsmgmt.msc-----共享文件夹管理器 
 utilman--------辅助工具管理器 
 dcomcnfg-------打开系统组件服务 
 ddeshare-------打开DDE共享设置 
 osk------------打开屏幕键盘 
 odbcad32-------ODBC数据源管理器 
 oobe/msoobe /a----检查XP是否激活 
 ntbackup-------系统备份和还原 
 narrator-------屏幕“讲述人” 
 netstat -an----(TC)命令检查接口 
 syncapp--------创建一个公文包 
 sysedit--------系统配置编辑器 
 sigverif-------文件签名验证程序 
 ciadv.msc------索引服务程序 
 shrpubw--------创建共享文件夹 
 secpol.msc-----本地安全策略 
 syskey---------系统加密
 （一旦加密就不能解开，保护windows xp系统的双重密码）
 services.msc---本地服务设置 
 Sndvol32-------音量控制程序 
 sfc.exe--------系统文件检查器 
 sfc /scannow---windows文件保护 
 taskmgr--------任务管理器 
 eventvwr-------事件查看器 
 eudcedit-------造字程序 
 compmgmt.msc---计算机管理 
 packager-------对象包装程序 
 perfmon.msc----计算机性能监测程序 
 charmap--------启动字符映射表 
 cliconfg-------SQL SERVER 客户端网络实用程序 
 Clipbrd--------剪贴板查看器 
 conf-----------启动netmeeting
 certmgr.msc----证书管理实用程序
```

未完待续  

  

**回顾往期内容**

[Xray 挂机刷漏洞](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247504665&idx=1&sn=eb88ca9711e95ee8851eb47959ff8a61&chksm=fa6baa68cd1c237e755037f35c6f74b3c09c92fd2373d9c07f98697ea723797b73009e872014&scene=21#wechat_redirect)  

[POC 批量验证 Python 脚本编写](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247504664&idx=1&sn=e88c77671f252631de939c154de075db&chksm=fa6baa69cd1c237f1c1f35f8b434874341f7fe077452834dac0e289addf9ac56fcbf7df5a8a1&scene=21#wechat_redirect)

[实战纪实 | SQL 漏洞实战挖掘技巧](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247497717&idx=1&sn=34dc1d10fcf5f745306a29224c7c4008&chksm=fa6b8e84cd1c0792f0ec433310b24b4ccbe53354c11f334a1b0d5f853d214037bdba7ea00a9b&scene=21#wechat_redirect)  

[渗透工具 | 红队常用的那些工具分享](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247495811&idx=1&sn=122c664b1178d563ef5e071e0bfd7e28&chksm=fa6b89f2cd1c00e4327d6516c25fcfd2616cf7ae8ddef2a6e869b4a6ab6afad2a6788bf0d04a&scene=21#wechat_redirect)  

[代码审计 | 这个 CNVD 证书拿的有点轻松](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503150&idx=1&sn=189d061e1f7c14812e491b6b7c49b202&chksm=fa6bb45fcd1c3d490cdfa59326801ecb383b1bf9586f51305ad5add9dec163e78af58a9874d2&scene=21#wechat_redirect)

 [代理池工具撰写 | 只有无尽的跳转，没有封禁的 IP！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503462&idx=1&sn=0b696f0cabab0a046385599a1683dfb2&chksm=fa6bb717cd1c3e01afc0d6126ea141bb9a39bf3b4123462528d37fb00f74ea525b83e948bc80&scene=21#wechat_redirect)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif)

扫码白嫖视频 + 工具 + 进群 + 靶场等资料

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpx1Q3Jp9iazicHHqfQYT6J5613m7mUbljREbGolHHu6GXBfS2p4EZop2piaib8GgVdkYSPWaVcic6n5qg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWFyt1RHHuwgcQ5iat5ZXkETlp2icotQrCMuQk8HSaE9gopITwNa8hfI7A/640?wx_fmt=png)

 **扫码白嫖****！**

 **还有****免费****的配套****靶场****、****交流群****哦！**
