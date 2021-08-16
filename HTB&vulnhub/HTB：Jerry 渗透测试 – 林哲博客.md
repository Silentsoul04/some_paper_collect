> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/22294.html)

> 基础信息 简介：Hack The Box 是一个在线渗透测试平台。

简介：Hack The Box 是一个在线渗透测试平台。可以帮助你提升渗透测试技能和黑盒测试技能，平台环境都是模拟的真实环境，有助于自己更好的适应在真实环境的渗透  
链接：https://www.hackthebox.eu/home/machines/profile/144  
描述：  
![](https://image.3001.net/images/20210508/1620482450_60969992d5ec815a8afd0.png!small)

本次演练使用 kali 系统按照渗透测试的过程进行操作，通过 nmap 发现端口开放了 tomcat8080 端口, 使用密码登录 tomcat，使用 msfvenom 生成 war 文件上传到 tomcat 中，通过反弹获取 shell。

##### 1、靶机 ip

IP 地址为：10.10.10.95  
![](https://image.3001.net/images/20210508/1620482807_60969af7bbd1ad463c784.png!small)

##### 2、靶机端口与服务

```
nmap -sV -sS -T4 -O -A 10.10.10.95
```

![](https://image.3001.net/images/20210508/1620483045_60969be517b71aa090607.png!small)

```
PORT     STATE SERVICE VERSION
```

##### 3、网站信息收集

查看 8080 端口  
![](https://image.3001.net/images/20210508/1620483158_60969c56127314e1ef025.png!small)  
通过 dirsearch 进行目录扫描获取网站目录  
![](https://image.3001.net/images/20210508/1620486151_6096a807cd65b35ae4818.png!small)  
这里我列出了部分需要的目录

```
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
```

##### 1、生成木马文件

进入 / host-manager/html 目录发现需要账号密码  
![](https://image.3001.net/images/20210508/1620486685_6096aa1d876a58f4cf3cb.png!small)  
点击取消后，再返回界面发现账号与密码  
![](https://image.3001.net/images/20210508/1620486776_6096aa785629a6aaedc13.png!small)  
username=”tomcat”  
password=”s3cret”  
使用该密码进行登录，返回 403 页面无法正常进行登录  
![](https://image.3001.net/images/20210508/1620486915_6096ab038674005c0fe91.png!small)  
最后在 / manager / 目录下成功登录  
![](https://image.3001.net/images/20210508/1620487059_6096ab9314792340f4aef.png!small)  
发现可以上传 war 格式文件  
使用 msfvenom 生成 shell 文件

```
|_http-server-header: Apache-Coyote/1.1
```

![](https://image.3001.net/images/20210508/1620487229_6096ac3da144bddcb02b9.png!small)

##### 2、使用 msf 获取 shell

首先设置好 msf，使用 msf 监听 4444 端口获取反弹的 shell  
![](https://image.3001.net/images/20210508/1620487531_6096ad6bb2389cfaea16c.png!small)  
在登陆页面上传 war 文件  
![](https://image.3001.net/images/20210508/1620487705_6096ae1949f02c6550816.png!small)  
上传成功后点击 shell  
![](https://image.3001.net/images/20210508/1620487748_6096ae4435b834b310407.png!small)  
成功获取到 shell

![](https://image.3001.net/images/20210508/1620487781_6096ae653ae8c407e7394.png!small)

使用这种方式所获取的直接是 authority 权限可直接查看 root.txt  
![](https://image.3001.net/images/20210508/1620487869_6096aebd959b7681f0680.png!small)  
进入 \Users\Administrator\Desktop 目录发现没有 root.txt 但是存在 flags  
![](https://image.3001.net/images/20210508/1620488338_6096b0925bdb23f5a113b.png!small)  
进入 flags 目录使用 type 命令查看 2 for the price of 1.txt 文件  
获取到 root.txt 与 user.txt  
![](https://image.3001.net/images/20210508/1620488539_6096b15bdcb63eb48fc7e.png!small)