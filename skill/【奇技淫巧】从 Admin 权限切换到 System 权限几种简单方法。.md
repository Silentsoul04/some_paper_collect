\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/xvqeygBbF0gwqUHBvgXusg)

**![](https://mmbiz.qpic.cn/mmbiz/Wqr9SokRcTWcKgdMIz6KrDJxEeibLFHr3yNgsRElic1xaian4Yp469kv3ZpkfOZeXORYpicEwAVoMqG7Mt7XFO1Jfg/640?wx_fmt=gif)**

1\. 通过创建服务获得 System 权限

```
列举token：
incognito.exe list\_tokens -u
复制token：
incognito.exe execute \[options\] <token> <command>
```

2\. 利用 psexec  

a. 从 https://download.sysinternals.com/files/PSTools.zip 下载 PsTools  
b. 解压后将 PsExec.exe 复制到 C:\\Windows\\System32 中。  
c. 以管理员身份打开命令提示符  
d. 使用 PsExec.exe 启动一个新的命令提示符。通过使用 PsExec.exe，您将在系统上下文中打开新的命令提示符，并且执行所有操作的账户将是 LOCAL SYSTEM 账户。这与 Specops Deploy App 在安装应用程序时使用的账户相同。  

  
命令：

```
psexec.exe -accepteula -s -d cmd.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTVpFLOchEAp99uuTgo3J1G0f1DnZNLrKpOG19cL915bm5Jkibeic4OSqstDk460Pcma9DwrTN6t1Jcw/640?wx_fmt=png)

  
3\. 利用 Meterpreter  
需要使用工具 getsystem-offline.exe 和 getsystem\_service.exe  
https://github.com/xpn/getsystem-offline(vs2019 可以编译)  

GetSystem-Offline
-----------------

This is a simple tool that spawns a SYSTEM command prompt on Windows.

Created as a demo of access token security.

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTVpFLOchEAp99uuTgo3J1G0JnCxsSPDJlcL3ESOzp6ESniaCGz6S0LHRwA6IiaNmvQepNJIlmpuXkMA/640?wx_fmt=png)

  
4\. 利用 token 复制获得 System 权限  

  
下载地址：

https://labs.mwrinfosecurity.com/assets/BlogFiles/incognito2.zip

参考手册：

http://labs.mwrinfosecurity.com/assets/142/mwri\_security-implications-of-windows-access-tokens\_2008-04-14.pdf

常见用法如下：

```
列举token：
incognito.exe list\_tokens -u
复制token：
incognito.exe execute \[options\] <token> <command>
```

  
命令：  

```
incognito.exe execute -c "NT AUTHORITY\\SYSTEM" cmd.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTVpFLOchEAp99uuTgo3J1G0g8P1ZdVKB1PmtbFCz8OBxsMKpJVbcsIDeXuRazK8fInricr6s6tWTyg/640?wx_fmt=png)

  
这个比较好用。  
这几种方法算比较简单的。有些比较复杂的方法等我测试过后再更新

**![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTWXcxZtiaMibnvovwBicjfhIibT2t5ty0s12WMUR6mvPjH8ibwXsF2bEt64NVvThjYgNvfctEOYB3UdYgA/640?wx_fmt=jpeg)**