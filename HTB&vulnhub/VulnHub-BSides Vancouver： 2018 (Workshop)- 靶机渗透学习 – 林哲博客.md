> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/21109.html)

> 靶机地址：https://www.vulnhub.com/entry/bsides-vancouver-2018-workshop%2C231/ 靶机难度：中级（CTF） 靶机发布日期：2018 年 3 月…......

**靶机地址：https://www.vulnhub.com/entry/bsides-vancouver-2018-workshop%2C231/**

**靶机难度：中级（CTF）**

**靶机发布日期：2018 年 3 月 21 日**

**靶机描述：**

**Boot2root 挑战旨在创建一个安全的环境，您可以在该环境中对（故意）易受攻击的目标执行真实的渗透测试。**

**该研讨会将为您提供定制的 VM，目标是在其上获得根级别的访问权限。**

**对于那些想进入渗透测试却又不知道从哪里开始的人来说，这是一个很大的机会。***

**如果这听起来令人生畏，请不要担心！在研讨会期间，我们将在渗透测试的每个步骤中讨论各种方法，常见的陷阱和有用的工具。**

**要求：**

**能够运行两个虚拟机并具有 USB 端口的笔记本电脑。**

**至少 20GB 的可用空间。**

**已预先安装 VirtualBox。**

**kali-VM**

**熟悉 CLI**

**是的，这是另一个基于 WordPress 的 VM（尽管只有我的第二个）—- 谷歌翻译**

**目标：得到 root 权限 & 找到 flag.txt**

**作者：DXR 嗯嗯呐**

信息收集
----

nmap 扫描靶机 IP

![](https://image.3001.net/images/20210121/1611212769_600927e19fd0d3af95c53.png!small?1611212770611)

nmap 端口扫描

![](https://image.3001.net/images/20210121/1611212774_600927e6addf9fac81af3.png!small?1611212775722)

21  ftp 可以匿名灯登陆

22 ssh

80  http

开始访问 21 端口，将 users.txt.bk 文件下载到本地

![](https://image.3001.net/images/20210121/1611212782_600927ee27e3fc700a238.png!small?1611212783158)

![](https://image.3001.net/images/20210121/1611212808_600928087ede67bd3d0d9.png!small?1611212809251)

![](https://image.3001.net/images/20210121/1611212817_6009281188d84eefc4113.png!small?1611212818410)

发现文件是几个用户名

![](https://image.3001.net/images/20210121/1611212851_600928338dbf9912d41d6.png!small?1611212852359)

22 端口没有密码，只能继续访问 80 端口

![](https://image.3001.net/images/20210121/1611213240_600929b8422614ee91594.png!small?1611213242540)

前面 nmap 扫描端口是发现了 robots.txt 和一个目录

![](https://image.3001.net/images/20210121/1611213245_600929bdeae3f7d103a24.png!small?1611213246741)

访问这个目录看看

![](https://image.3001.net/images/20210121/1611213252_600929c4da519631f71a6.png!small?1611213254003)

看来这是个 wordpress，先使用 gobuster 爆破一下目录

> gobuster dir -u [http://192.168.16.150/backup_wordpress/](https://www.lz80.com/go?_=6bf612ab82aHR0cDovLzE5Mi4xNjguMTYuMTUwL2JhY2t1cF93b3JkcHJlc3Mv)-t30 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt  -s 200,301,302

![](https://image.3001.net/images/20210121/1611213316_60092a040ea3b97014028.png!small?1611213317434)

发现两个有趣的目录，一个文件目录，一个登陆界面

![](https://image.3001.net/images/20210121/1611213323_60092a0b9b1df621ebb69.png!small?1611213324570)

![](https://image.3001.net/images/20210121/1611213342_60092a1e3745506d4d7be.png!small?1611213344124)

先使用 wpscan 枚举一下 wordpreess 账号

> wpscan –url ‘[http://192.168.16.150/backup_wordpress/](https://www.lz80.com/go?_=6bf612ab82aHR0cDovLzE5Mi4xNjguMTYuMTUwL2JhY2t1cF93b3JkcHJlc3Mv)‘ –api-token ‘API’ –enumerate u

![](https://image.3001.net/images/20210121/1611213349_60092a255228ddd17ed56.png!small?1611213350394)

继续使用 wpscan 爆破一下密码

> wpscan –url ‘[http://192.168.16.150/backup_wordpress/](https://www.lz80.com/go?_=6bf612ab82aHR0cDovLzE5Mi4xNjguMTYuMTUwL2JhY2t1cF93b3JkcHJlc3Mv)‘ –api-token ‘API’ -U wordpressusers.txt -P /usr/share/wordlists/rockyou.txt

![](https://image.3001.net/images/20210121/1611213366_60092a36b48ded4c56a46.png!small?1611213367879)

爆破出账号密码

john / enigma

登陆账号，寻找注入点

![](https://image.3001.net/images/20210121/1611214160_60092d501d25bbdfba9e2.png!small?1611214161220)

上传反弹 shell

![](https://image.3001.net/images/20210121/1611214165_60092d55af6ce2f3b5945.png!small?1611214167120)

在 media 模块看到上传的 php 文件

![](https://image.3001.net/images/20210121/1611214172_60092d5c73de5c4ad8a86.png!small?1611214173389)

点击获得文件的访问路径

![](https://image.3001.net/images/20210121/1611214182_60092d666d7251ec84f15.png!small?1611214183814)

提权
--

kail 上监听 1234 端口，访问反弹 shell, 返回交换窗口

![](https://image.3001.net/images/20210121/1611214187_60092d6ba6c9913982643.png!small?1611214190112)

查看 home 下的用户

![](https://image.3001.net/images/20210121/1611214192_60092d706dbcb31324ebd.png!small?1611214193393)

有几个用户，测试发现只有 anne 用户可以登录

![](https://image.3001.net/images/20210121/1611214198_60092d76a7525aaf09a6c.png!small?1611214200347)

使用 hydra 爆破一下密码，爆破出来了密码

> hydra -l anne -P /usr/share/wordlists/rockyou.txt -t 4 ssh://192.168.16.151

![](https://image.3001.net/images/20210121/1611214205_60092d7dcfe3f931269b2.png!small?1611214206909)

login: anne   password: princess

登录发现具备 sudo 权限，直接 sudo su 提权到 root 权限

![](https://image.3001.net/images/20210121/1611214242_60092da28ac5cbc404548.png!small?1611214243581)

**完成！！！**

总结
--

这个应该有很多种方法，不然这有点愧对中级了，之后在慢慢探索吧。

1、使用 nmap 扫描靶机 IP 和端口信息，匿名访问 FTP 获得 users 文件, 访问 80 端口 robots 文件，获得目录。

2、使用 gobuster 爆破目录，获得 wordpress 登陆界面，使用 wpscan 爆破账号密码。登陆 john 用户，上传反弹 shell, 获得交换 shell。

3、使用 hydra 爆破 anne 用户，ssh 登陆，用户具备 sudo 权限，直接 sudo 切换到 root 用户上，获得 root 用户权限。