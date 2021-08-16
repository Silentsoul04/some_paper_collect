> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-D_rLdRc11-yvHzpj4W0pg)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **119** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/112

靶机难度：中级（4.5/10）

靶机发布日期：2017 年 10 月 28 日

靶机描述：

Enterprise is one of the more challenging machines on Hack The Box. It requires a wide range of knowledge and skills to successfully exploit. It features a custom wordpress plugin and a buffer overflow vulnerability that can be exploited both locally and remotely.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/J3oCWSdkC78PfvFSez15Pqhj518XGYX9EmlmyPuUhmz4ECUuWPkaicgqibR1u4z8Ak4GZCc8oxF37tFpc5DNTPBw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7NCia6HrnS8X77g9nlDlz63tiaq9cWHm8ErUUdSNEqAGsGAlG1KslAHkQ/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.61....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7fkZygwprh8FwEhhSybSap4fAo73eKriahKOGyUNcvicnj1EEc4nl5XzA/640?wx_fmt=png)

Nmap 发现开放了 SSH 服务器，以及 Apache 的几种不同版本服务，以及端口 32812 上的未知服务，端口 80 上安装了 Wordpress，端口 8080 上安装了 Joomla....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7PnLLVQzUpJiclbiaCEenMMPKogaq4mMAFhkciaIphVU6PsFu6CXdu0SNA/640?wx_fmt=png)

直接目录爆破....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7v8Cm1KGuMXzxVF1HajDXyr6haVH9dl3hndOeCHcnjNoHsFibkbL0cIg/640?wx_fmt=png)

发现了个解压文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7aGkeSf01qmRB2Lwc71wm6a66GMQbDSdaZOYYnvH5UqWegQzgNKSm2g/640?wx_fmt=png)

下载到本地分析...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7pxiaAuAalqUSQH7h33mT0AMA5CCuNKeia2s4WDw7YicRH9lXhRokxOvGQ/640?wx_fmt=png)

第一个是包含 WordPress php 配置的 php 脚本，位于 / var/www/html/wp-config.php 中，这里可以利用 SQL Injection 漏洞进行 sqlmap 注入扫描....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7lT3HzHY4neFH7jRicg0okT3bk8oiceDLiapb95M5y78flN5PTg8W5W8Mw/640?wx_fmt=png)

在这里，脚本还是使用了 / var/www/html/wp-config.php 配置，并具有与上述相同的功能，但附加了 2 条 if 语句，这些附加语句可对 sql 注入进行额外检查，地址：

```
http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1....
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic77GDNfCtMiaV1Wxv4RicKqjr48kKIv5YnLSqOR9wH6cTuZ0dibyJ3ibD4Vw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7G7K0Q7lyL6VajqVby7EmIfic49kvqEKzmRvDkDYXSJaWQs3n8PNCiceg/640?wx_fmt=png)

```
sqlmap -u http://10.10.10.61/wp-content/plugins/lcars/lcars_db.php\?query\=1 --dbs
```

发现了数据库.... 继续 sql

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic72Bo2WEtNVO4wdXQichkRN4jN4fLoIXTgXoVBGicic1eMkmuUq2mXMLLicw/640?wx_fmt=png)

```
sqlmap -u http://10.10.10.61/wp-content/plugins/lcars/lcars_db.php\?query\=1 -D wordpress -T wp_users -C user_login,user_pass,user_email --dump --hex --threads 5
```

这里发现了 william.riker 用户信息... 但是我密码 john 爆破了，没有成功解密...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7GggCQd3ic6dwbDOM2ia5Xnmoq21K7ulvt5SiaicYBXnvvicQlria83giaVg0g/640?wx_fmt=png)

```
sqlmap -u "http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1" --random-agent --batch --thread 10 -D wordpress -T wp_posts -C post_content --dump
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7nWCE9NibvCQU05QIbs0Tj7tdicIsUED7ZXibEzvibalfJhJsuLeoNVQBIA/640?wx_fmt=png)

经过各种 sqlmap 注入... 在一篇文章中发现了四个密码信息.... 应该是隐藏文章，未发表的.. 扫出来了 (这里记得去掉 N）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7FiazQGcG875fRxhrzcWUXrY2dvhBs7XSflPjvAkuJ4bNaBhgibyAmLKg/640?wx_fmt=png)

```
wpscan --url http://enterprise.htb --usernames william.riker --passwords passwd.txt
```

由于需要登陆 wordpress 博客，将四个密码创建文本，直接利用 wpscan 爆破即可...

结果发现了登陆的密码信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7E4PDxNj2dJ0O8yvgBsHXGRy8NC40bjDMabibIft88GzRGvwHEuRvR3w/640?wx_fmt=png)

一句话 shell...（这里跳过了) 不懂得看前面文章....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7ibCUdQaWEyeiaEC0xSkeGXPaRz3ASO1rddZhrknw3CZRU8CoiawDLjh0g/640?wx_fmt=png)

成功获得了 shell_www-data 权限....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7lz7fN4fWRLKKGciaRRssCrS4A1KK6Xg6icunPfC796M9u9dG59iaXR5zQ/640?wx_fmt=png)

这里无法 wget 上传文件... 我利用本地监听 80 端口，直接执行 LinEnum 枚举了靶机...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7urGAf6nEtwl8lZvAq5vh5bibib0ZFVIn8L1ibjDNMYhaTPaSehklWRDNw/640?wx_fmt=png)

可以看到我们当前 IP 是 172.17.0.3，还发现了另外两个 IP 信息... 去查看下 hosts

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic739h4o90DtIFLjO8o6GDthyDMgY6NJiciabo6ZgXGJVuVaXibiaNgA2D5tA/640?wx_fmt=png)

查看发现点 2 是 mysql 数据库.... 这里可以利用 nc 或者 MSF_portfwd 进行端口映射然后登录 mysql 数据库查看底层信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7GI0QhY6ibZWpqRDMXBBUsaoEjCxk5PgAOM59WA4qicUj1okOWIdI4Eqg/640?wx_fmt=png)

这里试过了几种方法，权限都无法上传，我直接对 8080 端口进行继续信息收集了....

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/J3oCWSdkC78PfvFSez15Pqhj518XGYX9EmlmyPuUhmz4ECUuWPkaicgqibR1u4z8Ak4GZCc8oxF37tFpc5DNTPBw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic72L4icDJbicCiccEdLQ7nG3WEEttJWjL399ia9lxnqVMQ16brkYCewaf3pg/640?wx_fmt=png)

根据前面 sqlmap 还收集到了 geordi.la.forge 用户信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7k3sBdZu7qdfBFDyUQmibn74zQkbuN3ibk1wjaymCnnJ2wUCRw2JRb6Sw/640?wx_fmt=png)

利用 sqlmap 获得的四个密码，成功登录....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7mGYr9sQhNX45jibX26s80UEqho08f5s97VnTUX6mia7SG5Oicn6erRXMw/640?wx_fmt=png)

直接在 Extensions > Templates > Templates > Protostar 的 joomla 上编辑模板....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7jFW4JlydWGaSI3P4UtNLrRRW9kBanes6mGz0L13D6ep9YkwbCfnh0Q/640?wx_fmt=png)

继续利用一句话 shell 提权即可....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic75Ze4MJDibpNkeKsZxBf7BWOYhxz11gacpLwe7Wn4HA9zpSXgD45cmww/640?wx_fmt=png)

成功获得了反向外壳.... 查看了本用户 IP...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7kgrvRF3nYOSy3EYfQ4gZmN2mib9KvCGSNQjPjW6LU2bosOqia2CSmibYQ/640?wx_fmt=png)

从这里获得的 shell，还是限制一样，www 权限.... 晕了... 继续枚举了靶机

但是我检查挂在后，看到 files 被挂载在端口 443 上... 就是之前下载 zip 文件的地方...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7YWckBSiaRtKJKKVMUJljLkWHXozwPYJqaZowep3vv1AOZOASWKicqI6A/640?wx_fmt=png)

测试创建文件夹，果然是共享的目录... 直接上传 shell 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7XcuxUKDbtgLfU0ZElIwia7QLGrXiaabawMT7A8do8YFeLiaSRA75FQP1A/640?wx_fmt=png)

继续文件上传提权... 成功提权还是 WWW 权限... 吐了

但是可以执行查看到 user_flag 信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic74iankzGR9l9oicE1qibiccquElvIZjsV2TwPSK3B3osLzbf2lFGSWTib8rQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7HicsY5H3preeGp8GOykBtOFgIsTF7NiacBIEredSyIQqvsly90UgJpaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7EYBic6ssicVuwB2XdpNsFY9c7QjA1QLmkPCpy5J5MBVjic8V6qmIrURGA/640?wx_fmt=png)

输入是否存在一定限制以触发缓冲区溢出   picarda1

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7sUC6QJfibX3qrxicgPuuRIvPg2B6XclZf5EdM7vmRyuhTibN9BUv2dGkQ/640?wx_fmt=png)

picarda1

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7aYByy3vDiceiabSL2pMicZp3zlP5pnI81pOsWZxQKDNMZqdiaCqbFx3PDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7bKT0dKSA1mILowQ8VdLJoKiaGY7dMVyWu2H2FVWhfd4xku7SodhEhCg/640?wx_fmt=png)这里通过 1~7 测试，4 存在缓冲区溢出...

可以看到溢出显示的是 Disable Security Force Fields....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7RCF2Avt3GJHzMcSAdEjrKY0cECE0ddnumfsHdbWBdNO9E7CsTWMPkQ/640?wx_fmt=png)

查看所有函数情况...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7tHSElk8gviaxibAxCCUpJc4LLSVW7aa307XxHN4rVOKUVPuSO2GGqMDQ/640?wx_fmt=png)

```
set disassembly-flavor intel
disass disableForcefields
```

现在我们可以通过此处的内容计算 EIP 的大小了...

公式为减去 esp（sub esp）到添加 esp（add esp）之间大小的和（调用到__isoc99_scanf@plt 为止）....

0xd4 + 0xc - 0x10 + 0xc - 0x10 + 0xc - 0x10 + 0x8 = 0xd0 = 208...

所以需要覆盖 EIP 的总大小为：208 + 4 = 212...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7eRA3LkpmrjniawjYLfqiaGJDJ6qYhO72u8UzX4PQRFamt7jyqDBiaI8kw/640?wx_fmt=png)

```
print system
print exit
find &system,+9999999,"sh"
```

最后寻找 sys 和 exit 有效输出 sh 的负载地址... 列出了四个有效负载地址，直接利用即可....  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPgeeE5bvej7ssMsfudRFic7N0Xyhcz05tH1tjzBhnXL5jFmKp72CIycssv68tXZQStFLmmD49D4icA/640?wx_fmt=png)

简单编写 python EXP... 成功获得了 root 权限，并获得了 root_flag 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/AhUATAqa6tibYa4zTrlvc4l1rFIH7HV8c7ibcicw1jgibbwVW2zia9JeVCleEKLjkT0RO7sJS34DVSzMJ9sGsIAn5Fg/640?wx_fmt=png)

目录爆破 - 解析 zip_php 内容收集信息 - Sqlmap 注入获取 mysql 信息 - 文件上传提权 - 缓冲区溢出提权。

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台中级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/oQI1m5hhwD5Gicl7xUf6kh3ISTH6iacM05s8G12QVAykGzh7S5Po8EgeS5XJvZbiacbS8AuRQJ1VaRic18jlToOhVQ/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

随缘收徒中~~ **随缘收徒中~~** **随缘收徒中~~**

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)