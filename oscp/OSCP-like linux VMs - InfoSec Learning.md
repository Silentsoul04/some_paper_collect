> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [whale3070.github.io](https://whale3070.github.io/oscp/2019/12/12/07-x/)

发表于 2019-12-12 | 分类于 [oscp](https://whale3070.github.io/category/#/oscp) | 阅读数 次

参考资料：

*   [https://github.com/ferreirasc/oscp](https://github.com/ferreirasc/oscp)

* * *

Kioptrix: Level 1 (#1)
----------------------

[writeup](https://whale3070.github.io/training/2017/10/01/kiotrix%E9%9D%B6%E6%9C%BA-139%E7%AB%AF%E5%8F%A3samba/)

渗透思路：samba 版本漏洞，远程缓冲区溢出，使用 msf to root

Kioptrix: Level 1.1 (#2)
------------------------

[writeup 上](https://whale3070.github.io/training/2017/11/01/kioptrix-level-2-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E/)

[writeup 下](https://whale3070.github.io/training/2017/11/01/level-2-nc%E4%B8%8E%E5%86%85%E6%A0%B8%E6%BC%8F%E6%B4%9E%E6%8F%90%E6%9D%83/)

渗透思路：linux + web 远程命令执行漏洞，获得 shell。linux 内核漏洞提权。

Kioptrix: Level 1.2 (#3)
------------------------

[writeup 上](https://whale3070.github.io/training/2017/11/02/%E8%BF%98%E6%98%AFkioptrix-LFI%E4%B8%8E%E6%B3%A8%E5%85%A5/)

[writeup 下](https://whale3070.github.io/training/2017/11/01/level-3-%E4%B8%8B%E9%9B%86-wget%E4%B8%8E%E6%8F%90%E6%9D%83/)

渗透思路：linux + web eval 函数代码注入，有 msf exp 可用，直接获取 root 权限。 或者 sql 报错注入，ssh 登陆，上传 msf 载荷，meterpreter 提权。

Kioptrix: Level 1.3 (#4)
------------------------

[writeup](https://whale3070.github.io/training/2017/11/01/level-4-%E5%B8%83%E5%B0%94%E7%9B%B2%E6%B3%A8/)

渗透思路：布尔盲注获得用户名密码，ssh 登陆，mysql 高权限执行系统命令提权

FristiLeaks: 1.3
----------------

[writeup](https://whale3070.github.io/training/2017/12/01/FristiLeaks-%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0+%E5%86%85%E6%A0%B8%E6%BC%8F%E6%B4%9E%E6%8F%90%E6%9D%83/)

渗透思路：找到源码中隐藏的用户密码，文件上传漏洞拿 webshell +dirtycow 漏洞提权。

PwnLab: init
------------

[writeup](https://whale3070.github.io/training/2017/11/04/Pwnlab-LFI-%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB-%E4%B8%8A%E4%BC%A0/)

渗透思路：web 文件包含漏洞，php 伪协议查看 web 源码。通过上传功能上传 jpg 后缀的 webshell，用文件包含漏洞执行该 webshell

提权方式与这篇[文章](https://whale3070.github.io/training/2019/03/09/x/)类似， 找到了 suid 权限的程序 msgmike，运行该程序，并且使用 strings 命令，可以得知该程序调用了 cat 命令。

于是操纵环境变量，以本地目录开始查找 cat，并且伪造一个 cat 命令。

`echo "/bin/sh" > cat`

`chmod +x cat` 于是运行该 suid 权限的程序，就会运行伪造的 cat 命令，调用了 / bin/sh。使用这种方法提权。

Pluck: 1
--------

[writeup](https://whale3070.github.io/training/2019/12/08/11-x/)

渗透思路：web 文件包含漏洞，查看备份文档，获取 ssh 登陆密钥获得 shell。dirty cow linux 漏洞获取最高权限。

省略没做的机器
-------

以下两个机器因为时间紧张，简单难度的就不做了。

*   W1R3S: 1.0.1
*   Stapler: 1

Kioptrix: 2014
--------------

[writeup](https://whale3070.github.io/training/2017/11/02/kioptrix-2014-level-5-LFI%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB-%E6%95%8F%E6%84%9F%E6%96%87%E4%BB%B6%E6%B3%84%E9%9C%B2/) 看了一眼 17 年写的 wp，惨不忍睹简直想删。。太菜了

[writeup2019](https://whale3070.github.io/training/2019/12/09/04-x/)

渗透思路：目录遍历获取敏感信息，修改浏览器头文件，进入 8080 端口。web exp 执行获得 web 权限；linux 内核 exp 执行获得 root 权限。

Mr-Robot: 1
-----------

[writeup](https://whale3070.github.io/training/2017/12/02/mr-robot-%E7%88%86%E7%A0%B4%E5%90%8E%E5%8F%B0+SUID%E6%8F%90%E6%9D%83/)

渗透思路：wordpress cms 爆破后台，弱密码登陆 web 后台，登陆后修改 404 页面的 php 文件，替换为 php reverse shell。suid 程序 nmap 提权。

HackLAB: Vulnix
---------------

[writeup 上](https://whale3070.github.io/training/2017/12/03/NFS%E7%BD%91%E7%BB%9C%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F-smtp%E4%BF%A1%E6%81%AF%E6%90%9C%E9%9B%86+ssh%E7%88%86%E7%A0%B4/)

[writeup 下](https://whale3070.github.io/2017/12/04/Vulnix-nfs%E6%8F%90%E6%9D%83/)

渗透思路：smtp 泄露用户名，结合 ssh 爆破弱密码获得 shell。nsf 提权，参考[这篇 OutOfBox 机器](https://whale3070.github.io/oscp/2019/12/06/02-x/)，已经讲过了。

uncompleted
-----------

Brainpan: 1 (Part 1 of BO is relevant to OSCP. egghunting is out of scope though)

*   Not so sure (Didn’t solve them yet):
*   VulnOS: 2 [ok]
*   SickOs: 1.2 [ok]
*   /dev/random: scream
*   pWnOS: 2.0
*   SkyTower: 1
*   IMF
*   Lord of the Root 1.0.1 [ok]
*   Tr0ll
*   Pegasus
*   SkyTower [ok]