> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/YILNRQpm0A4b3b2Iab9mmQ)

大家好，我们是想要为亿人提供安全的亿人安全，这是我们自己想要做的事情，也是做这个公众号的初衷。希望以干货的方式，让大家多多了解这个行业，从中学到对自己有用的知识。

**靶机描述：**

下载地址：http://www.vulnhub.com/entry/westwild-11,338

级别：中级  

**信息收集**  

靶机 ip 扫描

```
arp-scan -l
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXzZQ51XcadWqXbwKWXSKttp83AtphRtW9Op3s44aJPibMibnRhnJ9Bxgw/640?wx_fmt=png)

```
kali攻击机:192.168.86.138
靶机：192.168.86.167
```

查看开放端口及版本、脚本信息探测

```
nmap -sC -sV -p- 192.168.86.167  --min-rate=2000 
-sC 脚本探测
-sV 版本探测
-p- 全端口探测
--min-rate 带宽配置 加速扫描
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXKRrcIayQzvN4IoUNPqzTIA5vAIIBVcnbIYpL45picFh9qgJxStkQ90g/640?wx_fmt=png)

查看 80 端口页面和目录扫描均没有得到有价值信息，只能由 445 端口入手，它作用是实现一些共享文件夹以及一些共享打印机的访问工作，使用 enum4liux 进行枚举查看可用信息，找到共享目录 wave

```
enum4linux 192.168.86.167
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXibMy9h5yB9XDr2qATNfd7AE8cwTdM3M3cBNXibhC6icicO2H90Zsj1Ofsg/640?wx_fmt=png)

使用 smbclient(samba client) 命令让 Linux 系统显示 Windows 系统所分享的资源

```
smbclient -L \\192.168.86.167
 -L：显示服务器端所分享出来的所有资源
 密码为空密码
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVX4XGbXpBBroClXMRcNrTh7YzsnkxobbMDmP4YqoCY4HIpMoWJVkd9JA/640?wx_fmt=png)

**FLAG1**
---------

进入共享文件夹查看信息

```
smbclient //192.168.86.167/wave
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXydUmlicl2icWzxnvzBgO3HauFyV81YN3y5uI2dqZIyw68YNOyWvWvpXg/640?wx_fmt=png)

在 kali 里查看信息，拿到 FLAG1

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXP96oiatm6D3JIRukpbtMqfqSqjUOodCkxtI4aScI1hEibtxxdyPYDaRg/640?wx_fmt=png)

使用 base64 进行解码，得到账号密码

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVX5QcGu1ia2MuS1PeG0vfccl53xoYynLktibIG38K3GhB5dVTc6Wf6VbMA/640?wx_fmt=png)

使用 ssh 连接成功

```
wavex/door+open
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXHy7YbDPrLm9vczZYqh9RnxXiaOysfN23jU2sThX5iafbfQUus7fpgwCg/640?wx_fmt=png)

**提权**
------

尝试切换 root 权限失败，使用 find 命令查找可读可写可执行文件，找到用户名 aveng 和密码

```
find / -type f -perm 0777  2>/dev/null
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXw0EzUVhqP2AZ31wPGQaxG6hFlUxvHdIqUnmia0ThObmvzU9hf4JdCMA/640?wx_fmt=png)

使用 ssh 连接成功

```
aveng/kaizen+80
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXsFvuoTnxBsXiaTMXoJks5M6GX2D3wicLM4IY9lOAibRPIMlflViaEgr2Ig/640?wx_fmt=png)

**FLAG2**
---------

切换 root 成功

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXicvDyrBlV4LWRjibTTALcIVS9t34PRJhY1aJQ1GMtDzibEB0Avn8ibbjgg/640?wx_fmt=png)

拿到 FLAG2

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpD84NDvY73wCFoB9jCFeVXthoVXWwsffaPsibB64ZlhQvjib1bkNXibicGhzJpasZbcQ7cL22mTUCZpg/640?wx_fmt=png)