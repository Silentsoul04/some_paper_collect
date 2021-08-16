\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/jGB1OD6i6aXVEjZs-g4LHQ)

**![](https://mmbiz.qpic.cn/mmbiz/Wqr9SokRcTWcKgdMIz6KrDJxEeibLFHr3yNgsRElic1xaian4Yp469kv3ZpkfOZeXORYpicEwAVoMqG7Mt7XFO1Jfg/640?wx_fmt=gif)**
==============================================================================================================================================

Bank-10.10.10.29  



Author:Beauty

致力于开新坑

涉及知识

DNS 域传送

Linux 文件权限与属性之 SUID

1\. 信息收集
--------

nmap 常规跑

```
nmap -sC -sV 10.10.10.29
22端口：openSSH    Ubuntu
53端口:53端口为DNS(Domain Name Server，域名服务器)服务器所开放，主要用于域名解析，DNS服务在NT系统中使用的最为广泛。通过DNS服务器可以实现域名与IP地址之间的转换，只要记住域名就可以快速访问网站。端口漏洞:如果开放DNS服务，可以通过分析DNS服务器而直接获取Web服务器等主机的IP地址，再利用53端口突破某些不稳定的防火墙，从而实施攻击。
80端口:Apache/2.4.7
```

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fPe9JbVibjNVQnsoAnhPGKh0Vn5QxlDojribiaLC9u6frYOEwt2P5hCiaeA/640?wx_fmt=png)

访问 80 端口 ------Apache 默认页面

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8faQtIibfVR4orb53h45B3NuLxMuvaa1utvprHoo4cnpqNuhp31pibXaQA/640?wx_fmt=png)

dirbuster 跑目录，没发现什么。

2\. 开始
------

考虑从 53 端口入手。dns 作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。

使用 dig 或者 nslookup 命令。 使用 nslookup 命令

```
nslookupSERVER 10.10.10.29bank.htb
```

![](投稿文章：记一次打 HTB 靶机过程/640)

其中 bank.htb 为常识，师父：一般靶机叫什么名，网站就叫什么名。佩服. jpg

检测到 bank.htb 确实存在。

使用 dig 命令 刚开始使用 dig 命令时，不知道 bank.htb，所以未成功

纠正后

```
dig axfr bank.htb @10.10.10.29
```

可以清楚的看到整个域下的域名解析信息，从而将整个域暴漏无遗

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fT5BCkDcicSkjvTgEKziaX9p9aMLvicmKAiaXdHaIkKojmvf4YKJnmTLaEQ/640?wx_fmt=png)

修改 resolv.conf 该文件是 DNS 客户机配置文件，用于设置 DNS 服务器的 IP 地址及 DNS 域名

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8f8NVpcicLoItkqDHNdjyqeoH6e3ibW65rBkrqfYBPQVtA0EuPZULA5zqA/640?wx_fmt=png)

nameserver 表示解析域名时使用该地址指定的主机为域名服务器。

最主要是 nameserver 关键字，如果没指定 nameserver 就找不到 DNS 服务器

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fmjn0kxk5jdLOoJYalicm16cImEsnUu6Pc3DABpEb4OM3wVrKFoFkoLA/640?wx_fmt=png)

修改后可通过 ping 域名来检测是否修改成功

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8flh83EVFjHnFxXkbF07ib1odUNxKIrA6REYyVCSqYbwJy4Xxu5dqrPjA/640?wx_fmt=png)

修改成功。

浏览器访问 \[url\]http://bank.htb\[/url\]

发现登录页面，尝试弱口令失败。

由于修改了 DNS 配置文件，所访问的域也就不同。

再次进行目录扫描

dirsearch 工具

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fSy9TqrjT8zab77qkQ157ib8F1BMmF7P5ZvyCVV5ZdibAianvyP0rZNX8g/640?wx_fmt=png)

字典不够强大~

发现最近扫目录，别的老板都可以扫出自己想要的东西，而我不能。o(╥﹏╥)o

突然发现自己的 kali 有点迷，在 resolv.conf 里添加 nameserver 的时候，必须要将 10.10.10.29 添加在最上面一行，他才会解析。

balance-transfer 目录发现许多 acc 文件。找到最突出的那个。257（师出反常必有妖）

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fSAWicM6uIej77qMlxrKDV9QWyb4WFb2LpRo48gNnj0ia5yJoAYn7iaZxg/640?wx_fmt=png)

发现账户以及密码

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fCoA2z9dZN6lUGowUdLogsbSiadGRVFJ2FibFy9fiaMbLtqicItfzD3ibialw/640?wx_fmt=png)

```
\--ERR ENCRYPT FAILED
+=================+
| HTB Bank Report |+
=================+
===UserAccount===
Full Name: Christos Christopoulos
Email: [email]chris@bank.htb[email]
Password: !##HTBB4nkP4ssw0rd!##
CreditCards: 5Transactions: 39
Balance: 8842803 .
===UserAccount===
```

用这个账户登录进去发现有上传文件的地方

support.php

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fXPA95Bbb7n5prAngCYy1PT6e2UxABeV1hFBwO70lpZEInb27TibCJjg/640?wx_fmt=png)

习惯性地查看前端代码，发现一句注释，大概意思就是，会把后缀为. htb 的文件当 php 解析。有点傻

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8f04NKLLtkibdmTWswHx6sbk4uOA4csOgxSfGCG45qy5nJcJPQ13nic5rA/640?wx_fmt=png)

上传冰蝎 php 脚本，burp 抓包，修改后缀，repeater 发现上传成功。

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fCiasvN9L0p7UmKq98ZC4Y57MX7bEvvIaefrFrNuaZoZpN5XgmwUoZyw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fm89sgw2z8B9kuDrIwKhvp1jZn3TltVxkymAnpDzxDHgwenDlq2ibldg/640?wx_fmt=png)

点击上传的文件，用冰蝎连接对应路径，出现 phpinfo, 成功。

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fXlhUqaicyNOBEJChdibaia7hF2QyO1IyYqvGpqlvfQouMfN3NPMibFvYvQ/640?wx_fmt=png)

whoami 后，不出意外的果然是普通权限

开始漫漫提权路。

3\. 提权
------

冰蝎 “给我连” 结合 msf paload 进行监听

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fr1WQ8cULeDGniatOjV0exAvRYibBeOTBV4pKXjib6z7xeqp1x97lMsibwQ/640?wx_fmt=png)

到 home 目录下，htb 的 user flag 一般都在 home 目录下。

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fqdXl892bLicC51pVuF9RV2Ob7g2GhqkiafS5CmLQ2rM9qrPuibVONV5iaA/640?wx_fmt=png)

user.txt user flag 具体是啥就不透露了 

接下来要找到 root.flag，尝试进一下 root 目录，被拒绝。

准备先在本机搭建一个小型服务器，以传输提权脚本

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fp07S4RU2QKJsWNzY3tF4TTRJXy31eRYMDpevpBOUe9G71LibgEZ6HnQ/640?wx_fmt=png)

shell 进入目标机，获取本机的提权建议脚本

其中要注意获取文件的路径

这里我是直接在提权脚本所在文件处搭建的小型服务器

所以 wget 的时候就难得再加路径了，简单点

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fJYm8CjaHXvbBjlbiaWQBXAFdy989JIDg36y05iaSCeC6FZbXPd1nCh7A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fgyeL4Rk9ZDp29ibXEPIRVPEjiajJqBPN2lzm1o5SS6WonRcQNq6ibCeBg/640?wx_fmt=png)

提权脚本给出了很多提权建议，其中还包括脏牛提权脚本，图中没列出来

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fGEibm5lz6evxQS4cYEhrGtQDgfRl72dFUJJbOvGntgVTZAJ2nMMT8ww/640?wx_fmt=png)

先试一试 “eBPF\_verifier“

下载它

我是在本机下载的，所以再把目标机上使用相同的方式获取该脚本 -----45010.c

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fy3YOcqFico4SnNnEQgNDsmYabOsy3ibias9RfgaR3jVuG0iaIs957qWykw/640?wx_fmt=png)

.c 文件可以使用 gcc 命令进行编译。但是在这台目标机上，该提权脚本失败了。还是换回脏牛。

upset，o(╥﹏╥)o 这台机器所给的权限都很低。找不到能执行文件的有效目录。很难受

转战 LinEnum，使用 LinEnum.sh

同样的方法传到目标机上，运行后，会发现该脚本给出了目标机很多信息

LinEnum 列出了一串 SUID files

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8ffxHSRLd4BZQSbFREianichGvOew68GVjm6Uu89b1E4QjCa02tk1qwrGQ/640?wx_fmt=png)

SetUID(或者 s 权限）：当一个具有执行权限的文件设置 SetUID 权限后，用户执行这个文件时将以文件所有者的身份执行。

在此 “文件所有者” 的第三位是 s 权限

进入到 /var/htb/bin 里面

执行 emergency 文件时，其他用户可以获取所有者身份，只是有时间限制。

然后 cd /root 

找到 root.txt get flag!

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fWFsuVkxIJFMy6siam4AMtdxKNkr11vqBYtMzGOAOaxyCOw0QFFzaQPg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTXCXJXqMBNkLPFBLk4icqK8fel3u65pdcRF7tbBh6c0SkyASjkwwyzx77NtTsJDVHyibechzuyFCxmg/640?wx_fmt=png)

4\. 总结
------

### Linux 文件权限

借鉴 https://www.cnblogs.com/Jimmy1988/p/7260215.html 的博客。学习学习

### 交互式 shell

在目标主机里，很多键和命令都不好使用 用以下命令。使目标主机有交互式 shell

```
$ which python/usr/bin/python$ python -c 'import pty;pty.spawn("/bin/bash")'$ ctrl + z$ stty raw -echo && fg$ export TERM=screen
```

**********![](https://mmbiz.qpic.cn/mmbiz_png/Wqr9SokRcTWXcxZtiaMibnvovwBicjfhIibT2t5ty0s12WMUR6mvPjH8ibwXsF2bEt64NVvThjYgNvfctEOYB3UdYgA/640?wx_fmt=jpeg)**********