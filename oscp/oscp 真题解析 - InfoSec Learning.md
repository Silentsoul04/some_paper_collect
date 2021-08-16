> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [whale3070.github.io](https://whale3070.github.io/oscp/2019/12/06/02-x/)

发表于 2019-12-06 | 分类于 [oscp](https://whale3070.github.io/category/#/oscp) | 阅读数 次

由于微信公众号文章的不稳定性，我收藏的某些文章竟然提示已经被作者删除。由于国内的网络安全法，所以遇到微信公众号上的好文章最好保存到本地，以免过段时间去看就已经被删掉了。

本地的学习资料是 oscp 泄露的[真题](https://whale3070.github.io/reference/2019/12/03/OSCP%25E7%259C%259F%25E9%25A2%2598%25E6%25B3%2584%25E9%259C%25B2/)（稳定性起见，已经放在我自己的博客上做个备份了，如有侵权请联系我进行删除），但在考试环境中已经移除，但是对于体验 oscp 考点还是比较有帮助。

**OutOfBox**
------------

### 参考资料：

*   [nfs 原理](https://www.cnblogs.com/me80/p/7464125.html)
*   [NFS 服务配置](https://wiki.jikexueyuan.com/project/linux/nfs.html)
*   [针对 NFS 的渗透测试](https://www.freebuf.com/articles/network/159468.html)
*   [linux-privilege-escalation-using-weak-nfs-permissions](https://haiderm.com/linux-privilege-escalation-using-weak-nfs-permissions)
*   [Linux NFS 提权](https://bbs.pediy.com/thread-222518.htm)

### 渗透思路

（80 端口）文件包含漏洞 +（22 端口）ssh 弱密码，拿到一个用户权限。 （2049 端口）nfs 共享的目录有**写权限**。通过写入一个调用 / bin/bash 的 c 代码，然后编译，添加 suid 权限，由于 nfs **权限配置不当**，是 no_root_squash，所以本地以 root 权限添加到共享目录的 c 程序，也具有 root 权限。运行该 c 程序，即可提权到最高权限，获得 root shell。

**nfs 是网络文件系统的英文缩写，nsf 协议能使 user 访问其他计算机犹如使用自己的计算机一样。更详细的资料请查看参考资料**

```
showmount 命令用于查询nsf服务器
showmount -e IP 列出指定nsf服务器输出目录列表
```

### 小实验

通过 shodan 查找 nfs 服务器，运用学到的命令查看服务器共享目录。

[![](https://raw.githubusercontent.com/Whale3070/Whale3070.github.io/master/images/12-06-02/1.PNG)](https://raw.githubusercontent.com/Whale3070/Whale3070.github.io/master/images/12-06-02/1.PNG)

mount.nfs: access denied by server 是什么原因。由于服务器经验不够，没遇到过类似的。也许是没有访问权限。

### nfs 协议可能的漏洞

与此台机器类似的 [Vulnix——nfs 提权](https://whale3070.github.io/2017/12/04/Vulnix-nfs%E6%8F%90%E6%9D%83/)，总结一下 nfs 协议常见的漏洞。

nsf 协议默认开放端口是 2049，并且还会随机开放小于 1024 的端口，用于文件传输等作用。也就说，文件传输协议会使用不止一个端口。

#### 1.nfs 版本漏洞

`searchsploit nfs | grep -v 'dos' | grep -v 'windows'`

#### 2. 权限配置不当

应该使用 root_squash 配置，而不是 no_root_squash。

**cat /etc/exports** 可以查看配置

no_root_squash ：加上这个选项后，root 用户就会对共享的目录拥有至高的权限控制，就像是对本机的目录操作一样。不安全，不建议使用

#### 3. 共享目录可写

如果共享. ssh 目录，就可以写入 ssh 密钥，从而无密码登陆

#### 4. 敏感信息泄露

泄露用户名信息可以结合暴力破解弱密码，从而登陆。

**Ph33r machine**
-----------------

clam av 存在代码执行漏洞，找到 msf exp，执行即可拿到 root 权限

**Admin-pc**
------------

windows 机器：ftp 匿名访问，获得敏感信息。用户名密码的 hash，这里考察 hash 破解。

通过用户名密码进入 web 后台，上传 php 反弹 shell，获得普通用户权限。提权：上传 jsp webshell 到 c:/xampp/tomcat/webapps/examples 目录，获得最高权限。

这里注意，不同目录，权限不同。

**unreal tournament**
---------------------

软件版本漏洞，找到 exp，修改 exp 并执行。考点：替换 shellcode

**UCAL**
--------

nikto 扫描到 web 后台，找到对应 exp，获得 shell。linux 本地漏洞提权。

**OFFENSIV-W2K3**
-----------------

web 存在 php 远程命令执行漏洞，反弹获得 shell。通过应用软件的配置文件获得口令。windows udf 提权。

总结
--

关于操作系统，oscp 考察 linux 和 windows，我猜应该是一半一半，linux 2 台 / windows 3 台或者反过来。

关于难度，比 hackthebox 一些主机要简单，至少用户名和密码相同这种漏洞，在 htb 我还没遇到过，也许是送分题吧。htb 考察的知识点也很典型，不会有特别偏和怪的 / 不常见的题。

关于复习，针对某些知识点需要彻底掌握，才能确保不丢分。例如 hash 破解、shellcode 阅读和修改、windows 和 linux 提权、常见服务可能出现的漏洞。

以上就是我对 oscp 真题的一些感受。