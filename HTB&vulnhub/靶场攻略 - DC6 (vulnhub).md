> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/a-o0NEzIrB6WFnxXcLVAaQ)

**声明：**请勿利用文中相关技术非法测试，如因此产生的一切不良后果与文章作者及本公众号无关！

DC-6 是一个易受攻击的实验环境，最终目的是让入侵者获得 root 权限，并读取 flag。DC_6 使用的操作系统为 Debian 64 位，可以再 virtualBox VM ware 上直接运行。作者在后面列出了相关提示：  

```
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

```
Download: http://www.five86.com/downloads/DC-6.zip
Download (Mirror): https://download.vulnhub.com/dc/DC-6.zip
Download (Torrent): https://download.vulnhub.com/dc/DC-6.zip.torrent ( Magnet)
```

**靶场下载链接：**

```
wpscan --url http://wordy --enumerate vp --enumerate vt --enumerate t --enumerate u
```

01

  

环境搭建  

  

根据作者的安装说明，将压缩文件下载后，通过 vm 或者 virtualbox 打开即可，注意由于作者设置为桥接模式，为了试验方便此处可以人为改为 NAT 模式。

  

02

  

**主机发现**  

  

通过 arp(地址解析协议) 进行局域网内主机发现，arp 是根据 IP 地址获取物理地址的一个 TCP/IP 协议。主机发送信息时将包含目标 IP 地址的 ARP 请求广播到网络上的所有主机，并接收返回消息，以此确定目标的物理地址；收到返回消息后将该 IP 地址和物理地址存入本机 ARP 缓存中并保留一定时间，下次请求时直接查询 ARP 缓存以节约资源。此处利用 metasploit 工具下 auxiliary 模块，通过 arp 协议发现内网主机的 ip 地址为 192.168.71.132。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18YtRMyOLxqU9udrtHJaQFnwTxkvG7icsPoSt0F7ibdHtNbIkZGMCUgINFw/640?wx_fmt=png)

  

03

  

**端口探测**  

  

利用 nmap 工具进行端口探测，相信大家对于 nmap 已经相当熟悉了，下面列举了一些 nmap 常见参数：

| 

-sT       TCP 扫描

-sS       SYN 扫描

-sF        FIN 扫描

-sA        ACK 扫描

-sU        UDP 扫描

-sR        RPC 扫描

-sP        ICMP 扫描

-sn        ping 扫描

-iL         文件读取 ip 地址

-O          操作系统识别

-T4         级别越高，扫描速度越快

-sV        版本检测 (sV)

 | 

 -oN            标准保存

 -oX            XML 保存

 -oG           Grep 保存

 -oA            保存到所有格式

  --host-timeout   主机超时时间 通常为 18000

 --open               只显示开放端口

 |

有端口扫描结果可知系统开放了两个端口：22 端口 (ssh)，80 端口 (http)。此处有两个利用思路，第一种是通过 hydra 对 ssh 服务进行爆破，第二种思路则是通过 web 进行渗透。可以进一步发现 http 服务被重定向到了 http://wordy/  

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18Y2NWK3RcF7lZezicAzbJCn0czqH0fWV7sACdcPrWWmHnwSWXns5M8Cpg/640?wx_fmt=png)

04

  

访问 web 服务  

  

由于 web 服务被重定向到了 http://wordy/ 此时可以本地添加域名到主机文件，这样以后我们访问`http://wordy/`就相当于访问对应的 ip 地址。添加完域名及对应 ip 后，可以发现已经能够访问`http://wordy/`  
`vim /etc/hosts`

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qn44UmeawwVibUSzfneHO18Y4NLxbUahnvnTfpKm52yjOPNnYnqWibclhIOiaQlPR0EXyfTDGnhjbibKA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18YQlibPiaTYlzucs7kWAleysicGRoOkh6QK5szrkM6wRqHt0LkglRiaxfdbA/640?wx_fmt=png)

05

  

**利用工具对 wordpress 网站信息搜集**  

  

查看打开的页面为 wordpress 搭建的 web 环境，利用工具 wpscan 对该 web 环境进黑盒扫描 (WPScan 是一个扫描 WordPress 漏洞的黑盒子扫描器)，可以获取到 wordpress 的版本，主题，插件，后台用户以及后台用户密码等。执行过程如下：

```
http://wordy/xmlrpc.phphttp://wordy/readme.htmlhttp://wordy/wp-cron.php
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18YPnRnK3kJ14WpLuDAEBKNgvKLIxWG8TGAZdCQmozicibWOsvQ1nW20FBQ/640?wx_fmt=png)

获取到的可用信息如下：

```
admin
graham
mark
sarah
jens
```

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

```
Things to do:
- Restore full functionality for the hyperdrive (need to speak to Jens)
- Buy present for Sarah's farewell party
- Add new user: graham - GSo7isUM1D4 - done
- Apply for the OSCP course
- Buy new laptop for Sarah's replacement
```

```
wordpress版本为：5.1.1Author: the WordPress team
枚举出的用户姓名如下：
```

```
admin
graham
mark
sarah
jens
```

06

  

**利用已知提示登录系统**  

  

```
cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt
```

在 kali 系统执行词条语句，生成密码字典 passwords.txt

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18YrBRHdYficG6vfN0L62HFnbyqLHQFyk42meFZeHNvquvKesK7MWvBTcg/640?wx_fmt=png)

根据枚举出的用户名和密码进行登录（`http://wordy/wp-login.php`）可以看到如下图所示界面，通过 wordspress 管理后台分析，并没有发现可以利用的漏洞。此时尝试寻找是否存在插件漏洞：activity monitor

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18YOrpXPQDKJadm3Cepxf0Nvibcibd1M2Z67SNau8iblncqhMYaQwqalz0XQ/640?wx_fmt=png)

07

  

**activity monitor 漏洞利用**  

  

将 45274.html 中的内容根据如下格式进行修改：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18Ye9DuaZCZo0PA7uaDYLtaTeiafVLOuQtibicZGicPhwRWqBSMLhu8untYkA/640?wx_fmt=png)

`nc -l -v -p 9999`在本机进行监听，通过 web 访问 45274.html 文件，点击运行后。可以看到 nc 成功反弹，效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18Y12J24nJg8ZguibibLbFnkbJg7QeXcly4BChx8oIFek6T9IuJfB19NVSA/640?wx_fmt=png)

进一步执行如下指令获取完整的 shell:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18YKyLhjICeKDUphVyfs8sRPl3KUm7dZLM3VIibGV3eJLXPTDA3qolKSiaA/640?wx_fmt=png)

通过目录查看，发现在`/home/mark/stuff`目录下存在`thing-to-do.txt`文件，其内容为：

```
Things to do:
- Restore full functionality for the hyperdrive (need to speak to Jens)
- Buy present for Sarah's farewell party
- Add new user: graham - GSo7isUM1D4 - done
- Apply for the OSCP course
- Buy new laptop for Sarah's replacement
```

添加了一个用户 graham 口令为 GSo7isUM1D4 ，因为系统开放了 22 端口，此时可以通过 ssh 登录该用户。

08

  

****提权****  

  

```
ssh graham@192.168.1.103
输入登录口令：GSo7isUM1D4
```

ssh 成功进行了登录，此时登录用户为 graham。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qn44UmeawwVibUSzfneHO18YCTzH6CT8kzA1PRAk0U84y0RxJI9p40I0Iw5usJeucEfsCUKAOWcuVA/640?wx_fmt=png)

继续输入 `sudo -l` 查看可执行的操作。发现能够以 jens 用户，不使用口令执行情况下执行 backups.sh。打开文件 backups.sh 为一个文件减压的命令行，可将减压指令删除，换成`/bin/bash`以 jens 用户去执行操作。

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qn44UmeawwVibUSzfneHO18Y4ofLsPwvANR74ukTEXLhPfrgxWrd8DYOf38lS6IFVKJl8lwQ52T8Vw/640?wx_fmt=jpeg)

此时用户为 jens，进一步，继续执行`sudo -l`查询可执行的操作，发现能够以 root 用户，在不使用口令的的情况下执行 nmap。故可通过 nmap 指令调用自己设定好的脚本如执行 / bin/bash。新建一个 nmap 可执行的脚本 root.nse，输入如下内容：

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qn44UmeawwVibUSzfneHO18YibAkf2hDiawnP2SSwEjvfo9J9Xib9sko1JZ4wXp3Fg0U9kcWUNC8LNA2Q/640?wx_fmt=jpeg)

通过 nmap 运行该 root.nse 脚本，进入 root 用户。

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qn44UmeawwVibUSzfneHO18YLPdaySzZabLYdn07BqTh6aFwV7LEdZ8PcBicicCGrhc5gW7iaVr8x1lVg/640?wx_fmt=jpeg)

- 往期推荐 -

  

[靶场攻略 | Moriarty Corp (vulnhub)](http://mp.weixin.qq.com/s?__biz=Mzg4MzA4Nzg4Ng==&mid=2247491777&idx=1&sn=1db28744c8b9f182b6552a443231c0a7&chksm=cf4e6fa0f839e6b60b8456604ccd36c240775039a320655f4dc9d71ee4a97c98763b176154a8&scene=21#wechat_redirect)  

**【推荐书籍】**