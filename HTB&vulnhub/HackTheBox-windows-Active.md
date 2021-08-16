> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XsPBKqgJmDbvjWuv2Ar6kw)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **69** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

靶机地址：https://www.hackthebox.eu/home/machines/profile/148

靶机难度：中级（4.8/10）

靶机发布日期：2018 年 12 月 4 日

靶机描述：

Active is an easy to medium difficulty machine, which features two very prevalent techniques to gain privileges within an Active Directory environment.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

  

![](https://mmbiz.qpic.cn/mmbiz_png/nKibbsr7q5Uoic4HqaOR77KgQOr062ubgGR7k9HhTqwJWan2KibZRiczhxkEzyKMBGO4LQDicBMFMPcJgp3RI6ia8IzA/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9sA5M7xetyFVsZEgPtpBUh2Yyjibyt5MBYlFGAZfPnXbBg154dxjRH4w/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.100.....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ99ibwl1n8l85NDJtzLFL6iaJC5B3Wp7Ur8dsiar3BgSyXkiaMfrtxtv0Zmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9iahicoxgcpUvcU6Q5BaicUjUQJbLhWSaKD3GiaEKyL0s8jc9gzPHy4gL5A/640?wx_fmt=png)

Nmap 揭示了一个域为 Active Directory 安装的 active.htb，Microsoft DNS 6.1 正在运行，它允许 nmap 将域控制器指纹识别为 Windows Server 2008 R2 SP1， 端口 445 也打开着，继续进一步的执行 nmap SMB 脚本查看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ93zB7KWPGdcice2xtrtMibzBrHW7kQ79YdI4MLlbXyTuvLibazYv83vaDg/640?wx_fmt=png)

```
nmap --script=safe -p 445 10.10.10.100
```

可以看到 SMB2 版本正在运行，并且连接到它的所有客户端都已启用，还需要消息签名，试试 SMB 中继攻击...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ93UtibHsXj5ic9na87fSfgFm8ygUzZpSlMqwxDHIZ8I1HjAqKzbMbhCCQ/640?wx_fmt=png)

前面 nmap139 上看到打开了 netbios-ssn，利用 smbmap 查看存在的目录情况，然后以匿名方式访问共享 Replication，但是目录太大，我通过别的方式查看所有文件信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9Jq6qy4jDj0oM3yJMqckgKkpTEvwcXjcOBbRdLq0gxfGs7lyfCeV8pw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9jIwP7sS9Ra2icwFdeUelkIic6PGtnRiaWGmjFyBj7ibnmXkruMMGEeTVFg/640?wx_fmt=png)

```
\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\
```

在 gropu 目录下发现了 Groups.xml ，文件是可逆格式存储着，存在用户名密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9lnleNMWRz0OVTN5g2XxmXG4PrVa51XPGLeJiarh1C7R0NwZTXKG8l0w/640?wx_fmt=png)

```
smbmap -R Replication -H 10.10.10.100 -A Groups.xml -q
```

通过命令下载，然后查看找到一个用户名和一个加密的密码...

![](https://mmbiz.qpic.cn/mmbiz_png/nKibbsr7q5Uoic4HqaOR77KgQOr062ubgGR7k9HhTqwJWan2KibZRiczhxkEzyKMBGO4LQDicBMFMPcJgp3RI6ia8IzA/640?wx_fmt=png)

二、提权

cpassword 是组策略密码，学习链接：

```
https://adsecurity.org/?p=2288  （简称GPP）
```

这里需要解密 gpp，使用 gpp-decrypt 工具即可

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9ibY9ibz21otSdc6orh6elTq8LHe0DkG6EyWpEXSXTDIvfk0dCbQH3B2w/640?wx_fmt=png)

解密成功...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ99tZuSSuCtYL8XunCy3F6OzK3pXe7wrZUPqOibCSf126u2VFbA8PuhJQ/640?wx_fmt=png)

通过凭证登陆到 users 目录下，查看到 user 内容...

回看前面 nmap 发现开放了 88 端口运行了 kerberoas 服务，下面运行 kerberoasting 技术进行获取 root 信息...

```
https://room362.com/post/2016/kerberoast-pt1/
https://room362.com/post/2016/kerberoast-pt2/
https://room362.com/post/2016/kerberoast-pt3/   kerberoasting学习资料
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9R4iagYN0ic1H67OUhHlgiamiafibibcqMqgmzNeB4q553K9cX9kuPxJwWQuA/640?wx_fmt=png)

利用 GetUserSPNs.py 从 impacket 得到管理员 Kerberos 凭证...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9hO0ICsdBBjmrdptBvDEkFEOdaNIkl4MvItquD3y7Ex5icAeNAs5DibibQ/640?wx_fmt=png)

```
python GetUserSPNs.py -request active.htb/SVC_TGS
```

获得了密匙凭证，直接通过 john 爆破即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9d38QQ1anqrKQg89CzGdbfME37p7xicskqeo0j40hHCxCwRAibhdSd0cQ/640?wx_fmt=png)

```
john dayu.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

通过爆破获得了管理员密码...

![](https://mmbiz.qpic.cn/mmbiz_jpg/2iaOMskBibMM4mNsp9A5G4u3Ev6zqassh3abNdVibWQe9H3ugibS1g34X7kn0Nibp23jchf2sWCdR4aS9aXSMI4LJiaw/640?wx_fmt=jpeg)

方法 1：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ93bVXUKwQZToBaHYeaRYklo4jicmdEf5ouUYkich2IKjpDZvgMicBkVicsw/640?wx_fmt=png)

```
smbclient -U administrator //10.10.10.100/Users
```

很简单，知道了管理员密码，直接利用 smbclient 登陆即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9Cic1s2agiaLNyEoaH5JjGuuwMH7yp6XmsK0nibydjMFRdfc0u1pmRKHiaA/640?wx_fmt=png)

登陆后在桌面查看到了 root 信息...

![](https://mmbiz.qpic.cn/mmbiz_jpg/2iaOMskBibMM4mNsp9A5G4u3Ev6zqassh3abNdVibWQe9H3ugibS1g34X7kn0Nibp23jchf2sWCdR4aS9aXSMI4LJiaw/640?wx_fmt=jpeg)

方法 2：

当然，一路走下来，我们都是利用 smb 共享的方式攻破了这台靶机，shell 都没用上...

这边可以利用 psexec 和 wmiexec 都可以用于获取具有管理员级别访问权限的系统上的 Shell...

演示个：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9KNafYvMqvXNdSRP4AONC1ibNGcqnsOcP6P6JY4zwd8qSUOnm3fsbhFQ/640?wx_fmt=png)

成功获得 shell，进入到计算机中...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9YM1q4RQQOKSUmkp8WH6MlrOZrUN9GarOcBevW5Qfk34J9TcI3RDJUg/640?wx_fmt=png)

轻松获得了 shell.... 获得 shell 后能进行什么操作就不多说了，进去看看就好...

提示下，psexec 是个 python 文件，在本地直接使用我报了很多错，所以才直接利用 msf 的，一般在本地就可执行... 不知道是不是新版 kali 的原因...

![](https://mmbiz.qpic.cn/mmbiz_jpg/2iaOMskBibMM4mNsp9A5G4u3Ev6zqassh3abNdVibWQe9H3ugibS1g34X7kn0Nibp23jchf2sWCdR4aS9aXSMI4LJiaw/640?wx_fmt=jpeg)

方法 3：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ9WNhXJp4BGPJTjRpuzTiaoXricLLz4DdBfICzTY1tSPGjGz6EiatBjf1Sw/640?wx_fmt=png)

```
python3 wmiexec.py active.htb/Administrator:Ticketmaster1968@10.10.10.100
```

利用 wmiexec 进行获得 shell... 

![](https://mmbiz.qpic.cn/mmbiz_jpg/2iaOMskBibMM4mNsp9A5G4u3Ev6zqassh3abNdVibWQe9H3ugibS1g34X7kn0Nibp23jchf2sWCdR4aS9aXSMI4LJiaw/640?wx_fmt=jpeg)

方法 4：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMqIuIVZrhKwqXjIySHQRJ91PJCxxjOHiaCc1GxlsZPsJLQoBRqsCFPmqibyfcxe3I7ogtiaxmAmhaSA/640?wx_fmt=png)

```
mount -t cifs //10.10.10.100/Users /mnt/smb -v -o user=SVC_TGS,pass=GPPstillStandingStrong2k18
```

这里利用挂载的方式在本地直接可以共享访问对方靶机的所有权限信息...

方法太多了这里... 列举四个就好了...

  

通过 SMB 匿名访问重定位文件共享，到组策略首选项泄露... 到 Kerberoasting 获得管理员凭证...

SMB 共享在生活中太常见了... 建议，删除对复制共享的匿名访问，删除密钥加密的旧密码，Kerberoasting 它是 Windows 的一项功能，而不是 Windows 的一个 bug，但仍需要一个强大的密码策略，以使散列更难用普通的单词列表破解，建议长度至少为 28 个字符，并且每六个月轮换一次，或者用最低特权策略进行限制减轻攻击....

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了中等靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

  

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

**随缘收徒中~~ **随缘收徒中~~** **随缘收徒中~~****

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)