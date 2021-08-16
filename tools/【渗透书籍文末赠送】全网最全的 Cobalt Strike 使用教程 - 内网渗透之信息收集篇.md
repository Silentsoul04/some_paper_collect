> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qcWiIfj8pwTncmyCVJ2qPA)

> **文章来源：国科漏斗社区  作者：**C1ay  
> 
> **文末粉丝福利，0 门槛送书**
> 
> **PS: 文章有点长，不感兴趣的粉粉们可以直奔书籍**  

**一、前言**  

从本篇文章开始，斗哥将向大家详细的介绍 cobalt Strike 这款工具在内网渗透中的具体使用方式，因为涉及的内容较多，大致会分为信息收集篇、横向渗透篇、域控攻击篇、权限维持篇，在本篇文章中将为大家介绍信息收集篇。

**二、内网信息收集篇**

测试环境：

<table width="677"><tbody><tr><td width="63.33333333333333"><p><strong>系统</strong></p></td><td width="113.33333333333333"><p><strong>服务</strong></p></td><td width="109.33333333333334"><p><strong>IP</strong><strong></strong><strong> 地址</strong></p></td></tr><tr><td width="66.33333333333333"><p>kali</p></td><td width="116.33333333333333"><p>外网 VPS（teamserver）</p></td><td width="109.33333333333334"><p>192.168.0.108</p></td></tr><tr><td width="66.33333333333333"><p>win2008</p></td><td width="116.33333333333333"><p>web 服务器（边界服务器）</p></td><td width="109.33333333333334"><p>192.168.0.109/1.1.1.10</p></td></tr><tr><td width="66.33333333333333"><p>win2012</p></td><td width="116.33333333333333"><p>域控（受害者）</p></td><td width="109.33333333333334"><p>1.1.1.2</p></td></tr><tr><td width="66.33333333333333"><p>win7sp1</p></td><td width="116.33333333333333"><p>域内主机（受害者）</p></td><td width="109.33333333333334"><p>1.1.1.23</p></td></tr><tr><td width="66.33333333333333"><p>m0n0wall</p></td><td width="116.33333333333333"><p>防火墙</p></td><td width="109.33333333333334"><p>1.1.1.1/192.168.0.105</p></td></tr></tbody></table>

#### **2.1 判断是否存在域**

具体方式如下：

1、ipconfig /all

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729Q7MuSzBpnYjvD66T9VbibRibxkdXM0L7IP0GlCnsGJwTk6C9dicYOpI9Q/640?wx_fmt=png)

2、systeminfo

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729l5RykpCqqOUBTUWbBSAqKpaS58icucLsEp4xGCgtLlw2iaLuPmNCAMEA/640?wx_fmt=png)

3、net config workstation

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729d26tqrynXqr7vvKPd5Micw91fmY23DM1Ja2ShPOKpyRicoWteNlD6fLw/640?wx_fmt=png)

4、net time /domain  

输入该命令可能存在如下三种情况：

●1. 存在域，当前用户不是域用户。

●2. 存在域，当前用户是域用户。

●3. 不存在域。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729U2nuJgSfSUX98TiczMfwb7N1QNwO2l6Umsvk2b8xuibeNHiazibPpTbukQ/640?wx_fmt=png)

通过 shell net view /domain 查看当前域。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729KDJNMHHMAG8BT1NcmibvdNEsmOtlQxtOiaZVmH4XJRRwCLjJWkiaSabYw/640?wx_fmt=png)

**2.2 域内存活主机探测**

1、利用 netbios 快速探测内网

工具：Nbtscan

使用方法：将该文件上传到目标机器上并执行 nbtscan.exe IP 即可

效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729bIynlPiabbOmRN5sHyuUiacBsOIYRuibYSks5Y6ibqfavOf5Fv0STLcUzA/640?wx_fmt=png)

2、通过 arp 扫描完整探测内网

方式一：工具：arp-scan

命令：arp.exe –t IP

效果如下:

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729uZdEjAt3XcHehvG6buYqakPxZrB5KF3qmW0vCYaECmukRwr2gJUHDg/640?wx_fmt=png)

方式二：通过 ARPScan 脚本

命令：

```
1.net view
```

效果如下:

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729eoJPhG3mfE5sIkC6asaYKaLYYeAHxLtQ4ktRt2HZvlziaFu8ictWUOcA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729tevMAnYSLtEh54tvNnsa6kOItTDcIemZ75Lxqz8Ld8s1aUDyic4QBfQ/640?wx_fmt=png)

3、利用常规 tcp / udp 端口扫描探测内网

工具：scanline

命令：

```
sl -h -t 22,80-89,110,389,445,3389,1099,1433,2049,6379,7001,8080,1521,3306,3389,5432 -u 53,161,137,139 -p 1.1.1.1-254 /b
```

效果如下:

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729JYHibRDJEH1lem84uCm4AlYjN2qBahZqVUAWZicVIibk5p7jr8iaGpAggA/640?wx_fmt=png)

可以通过判端口开放情况来确定域内存活主机情况。

#### **2.3 域内基础信息收集**

获取域内主机信息，命令如下：

```
1.net view
```

效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729VWWtXZWYeIYYjfzZT7bEOic9PbMRbmO9YmfIhwsFUCrLdLBJ8biaPiasA/640?wx_fmt=png)

获取所在域的域名，命令如下：

```
net view /domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729toOuIOVQcN3YTch9B0XicjibpctpYKqcUOO7pRB411J9Jfe67ckGnXCQ/640?wx_fmt=png)

在得到域名后，通过 shell net view /domain:hacker 查看当前域内主机。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729xx359XjD2H9qOl48qzN4cHNhuWPtqQHdaabbicPp7Z9FH2rqdNJXd4A/640?wx_fmt=png)

查询结果可以在 View->target 中进行查看。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729QtzDVwQ4ZyLicer0h4cG2aa977je0iclgcZx1JI9Vd2oxI5C0JiaXIULQ/640?wx_fmt=png)

通过 net group “domain computers” /domain 命令可以查看域内计算机。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729PpAOs9sawsOEFdLEARt7MQphD0ciboNvVhG4w0NiaZzaQnFU0zhFR8fQ/640?wx_fmt=png)

通过 shell nltest /domain_trusts 命令查看域信任关系。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729xicmVicDQhFC3tWKFS64SHcN0m5G23lqVTpJpsJpzBuI5xYITdS8ed7g/640?wx_fmt=png)

通过 shell net accounts /domain 命令查看域内账号密码信息。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729prVvZhIdBNsbEDvSt0eExibe9pGrxVXu11zd1PMAWKiaFokbMy4A5gvQ/640?wx_fmt=png)

#### **2.4 域内控制器的查找**

通过如下命令查看当前域的域控制器。

1、 shell nltest /dclist:hacker

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729RSFWPRy7XEcsiaKM6Zp38hDgbvyoicMnxCxgc4IFbOskF0VkRPvXmSpw/640?wx_fmt=png)

2、shell net time /domain

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729IIHdqwtr9ESwyWJswSP9ClDh4nc49PXVGicUcicse81EsUwZmsB4JSEw/640?wx_fmt=png)

3、net group “Domain Controllers” /domain

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729RocNNAH3U8jTibYePPLKUsGjjqa7bE0MSoKSbhqyrRTue7ArLTndiaDw/640?wx_fmt=png)

4、netdom query pdc

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S7294Ehw1Z4KVlfbsuEwzQfiaShdXhgHC0jmw5D0AaAJQS5BH7ta2TcZSIg/640?wx_fmt=png)

#### **2.5 定位域管理员工具**

##### **2.5.1 通过 psloggedon.exe**

该工具可以查看本地登录的用户和通过本地计算机或远程计算机的资源登录的用户。如果指定的是用户名而不是计算机名，该工具会搜索网上邻居中的计算机，并显示该用户当前是否已经登录。其原理是通过检查注册表 HKEY_USERS 项的 key 值 和 通过 NetSessionEnum API 来枚举网络会话，但是该工具的某些功能需要管理员权限才能使用

参数：

●-l：仅显示本地登录，不显示本地和网络资源登录

●-x：不显示登录时间

●\ 计算机名：指定要列出登录信息的计算机的名称

●用户名：指定用户名，在网络中搜索该用户登录的计算机，该功能实现有问题。

效果如下：

1、获取本地及远程登录的计算机名称。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S7294F1FVPVzZMFicsm7hz6cNbGQNGo1Yxicv7p8J33yDib69ksQutRcKFg2A/640?wx_fmt=png)

2、获取指定计算机名的用户。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S72928Nic7QvqfENXLIJ0XmiaBJmgNB7uowiarMtSk0zh49jYDVtdJWq7ptAQ/640?wx_fmt=png)

3、获取指定用户登录的计算机。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729AylgKUibGAicnFGkicObV4MXibu8k0a8iaQZYAjVsMwr4ZHvbSB3HAEbia1A/640?wx_fmt=png)

##### **2.5.2 通过 PVEFindADUser.exe**

该工具可用于查找活动目录用户登录的位置，枚举域用户，以及查找在特定计算机上登录的用户，包括 本地用户，通过 RDP 登录的用户、用于运行服务和计划任务的用户。该工具的运行不需要管理员权限， 只需要普通域用户即可。

参数：

●-h：显示帮助信息

●-u：检查程序是否有新版本

●-current ：如果仅指定了 - current 参数，将获取目标计算机上当前登录的所有用户。如果指定了用户名，则显示该用户登录的计算机

●-last：如果仅制定了 - last 参数，将获取目标计算机的最后一个登录用户。如果指定了用户名，则显 示此用户上次登录的计算机。根据网络的安全策略，可能会隐藏最后一个登录用户的用户名，此时 使用该工具可能无法得到该用户名。

●-noping：阻止该工具在尝试获取用户登录信息之前对目标计算机执行 ping 命令

●-target：可选参数，用于指定要查询的主机。如果未指定此参数，将查询当前域中的所有主机。如果指定了此参数，则后跟一个由逗号分隔的主机名列表，此功能实现由问题。

效果如下：

1、查询所有主机当前的登录用户。

直接运行 pveadfinduser.exe -current ，则可显示域中所有计算机上当前登录的所有用户。查询结果将 被输出到 report.csv 文件中。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S7298ibyiacmseFVsVKsoGjLQBY7twQIxSQdpasSdG4ncrOuJNBIIkfSP2zw/640?wx_fmt=png)

2、查询指定用户当前登录的主机。

```
shell PVEFindADUser.exe -current hacker\administrator
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729n6OQA3S5uXCSHflIeB0zlpMOOkSj4XHftaT8cF6uLR7MX81dGl99CA/640?wx_fmt=png)

3、查询指定主机当前的登录用户。

```
shell PVEFindADUser.exe -current -target DC
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729e2wzPuPSzgyboy7ZxHMqS34fpA4fdLtGKvkjTZ411gYtEe3xLGg7WQ/640?wx_fmt=png)

##### **2.5.3 通过 PowerSploit 的 PowerView**

PowerView.ps1 脚本的 Invoke-StealthUserHunter 和 Invoke-UserHunter 函数。

●Invoke-UserHunter：找到域内特定的用户群，接收用户名、用户列表和域组查询，接收一个主机 列表或查询可用的主机域名。它可以使用 Get-NetSessions 和 Get-NetLoggedon 扫描每台服务器并 对扫描结果进行比较，从而找到目标用户集，不需要管理员权限。

●Invoke-StealthUserHunter：只需要进行一次查询，就可以获取域内的所有用户。该函数从 user.HomeDirectories 中提取所有用户，并对每台服务器进行 Get-NetSessions 获取。因为不需 要使用 Invoke-UserHunter 对每台机器进行操作，所以这个方法的隐蔽性相对较高，但是扫描结果 不一定全面。PowerView 默认使用 Invoke-StealthUserHunter，如果找不到需要的信息，就使用 Invoke-UserHunter。

执行以下命令可用于定位指定域用户登录的主机和查看指定主机当前的登录用户。但是值得说明的是，查看指定主机当前登录的用户这个功能并不好用，仅仅能列出当前主机登录的用户，枚举其他主机时不显示。

```
powershell-import

powershell Invoke-UserHunter
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729Y797NYJxchib9nJsqG68Urv0nEBohKxOTW4urkiaV9XzOdzQewOU5hEw/640?wx_fmt=png)

5.1.5.4 通过 NetSess 工具

netsess.exe 的原理也是调用 NetSessionEnum API，并且无需管理员权限。

命令如下：

```
netsess.exe -h 机器名
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729lcdSuDMhzQN4doibvFnf4lk9TEgSpnS1KiaQMpmkvic1zTaOSwBwZBRhQ/640?wx_fmt=png)

还有其他很多方式，这里就不一一列举了。

#### **2.6 查找域管理进程**

当计算机加入到域后，默认将”Domain Admin” 组赋予了本地系统管理员的权限。也就是说，在计算机添加到域，成为域的成员主机的过程中，系统将会自动把”Domain Admin” 域组添加到本地的 Administrators 组中。因此，只要是 Domain Admin 组的成员均可访问本地计算机，而且具备” 完全控制” 的权限。

因此对于渗透测试人员来说，把”Domain Admin” 域组添加到本地的 Administrators 组中是他们模拟域管理员帐户操作的常用方式。不过前提是，他们需要知道这些进程正在运行的系统。在本文中，五种寻找 “Domain Admin” 运行的进程的方法，其中涉及的技术包括：

1. 本地检查；

2. 查询活动域用户会话的域控制器；

3. 扫描运行任务的远程系统；

4. 扫描 NETBIOS 协议信息的远程系统。

##### **2.6.1 本地检查**

首先检查最初被破坏的系统，如果你已经存在于域管理进程中，那么在网络上运行真的没什么意义了。以下是使用本机命令检查是否有任何域管理进程正在运行的简单方法：

1、运行以下命令获取域控制器列表。

```
shell net group "Domain Controllers" /domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729zshktqTYVbGcxkMOra2FerIc9IwX6cblJJ4NkuTds0JOicul6xZ6y4g/640?wx_fmt=png)

2、运行以下命令以获取域管理员列表。

```
shell net group "Domain Admins" /domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729Av73YY4HjOpA9I5y919SeUuIzhMTibCnrGaYlStahDibtr6uriaY3QiaDg/640?wx_fmt=png)

3、运行以下命令列出进程和进程用户，运行该过程的帐户应该在第 7 列。

```
shell Tasklist /v
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S7293JC45lh5Z04V9cI7uM0GPfXBibibjbbGjcmTuOMxxKEa9icia6nGicoZymw/640?wx_fmt=png)

也可以直接通过 CS 模块查看进程列表。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729NVEej5ZR4HYms4ko5ZICc1Ubsdw11wrdiarj2UaHRttec71DibgNfzQQ/640?wx_fmt=png)

4、将任务列表与域管理员列表交叉引用，查看你是否进入域管理进程中

如果域管理进程始终是在最初受到攻击的系统上运行，那就太好了，但这属于理想的情况所以接下来的几种技术将帮助你在各种情况下的远程域系统上找到域管理进程。

##### **2.6.2 查询域控制器的活动域用户会话**

这项技术是安全公司 Netspi 的原创技术。我们需要一种用来识别活动的域管理进程和登录，而不是在整个网络上执行 shell 喷洒或执行任何会引发 “入侵检测系统” 的扫描。最终，我发现只需简单地查询以获取一个活动域用户会话列表即可，然后将该列表与域管理列表交叉引用。唯一可能出现问题的环节，就是你必须查询所有的域控制器。下面是我提供的一些基本步骤，以获得具有域用户权限的活动域管理会话的系统列表。

1、使用 LDAP 查询或 net 命令从 “域控制器”OU 中收集域控制器的列表，以下是我用过的一个 net 命令：

```
net group “Domain Controllers” /domain
```

请注意：虽然 OU 是域控制器列表的最佳来源，但前提是，你要对受信任域完成枚举并监控这些域控制器的过程。或者，你可以通过 DNS 查找它们：Nslookup –type=SRV _ldap._tcp.

2、使用 LDAP 查询或 net 命令从 “域管理员” 组中收集域管理员的列表。以下是我用过的一个 net 命令：

```
net group “Domain Admins” /domain
```

3、通过使用 Netsess.exe 查询每个域控制器，收集所有活动域会话的列表。Netsess 工具是 Joe Richards 提供的一个很棒的工具，它里面包含了本地 Windows 函数 “netsession enum”。该函数可以返回活动会话的 IP 地址、域帐户、会话启动时间和空闲时间。以下是我用过的一个 net 命令：

```
shell netsess.exe -h dc
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729qRAssQkvviaTz4CibQaqtBzGzrG1LhBfdLLX7qldiaMAqib4Y9X1CA1moA/640?wx_fmt=png)

4、将 Domain Admin 列表与活动会话列表交叉引用，以确定哪些 IP 地址上有活动的域令牌。在更安全的环境中，你可能需要等待具有域管理员权限的域管理员或服务帐户在网络上执行此操作。下面是一个使用 netsess 的非常快速和具有攻击力的 Windows 命令行脚本。

```
FOR /F %i in (dcs.txt) do @echo [+] Querying DC %i && @netsess -h %i 2>nul > sessions.txt && FOR /F %a in (admins.txt) DO @type sessions.txt | @findstr /I %a
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729csN29VP5RTljnqVes33BRicB53icK9FQKZ3fgPSq86qvEDcFSm3Rh25A/640?wx_fmt=png)

##### **2.6.3 扫描 NetBIOS 信息的远程系统**  

某些版本的 Windows 操作系统允许用户通过 NetBIOS 查询已登录用户，下面这个命令就用于扫描远程系统中的管理会话。将目标域内主机 ip 保存为 ips.txt 文件, 域管理员保存为 admins.txt 文件。

```
for /F %i in (ips.txt) do @echo [+] Checking %i && nbt

stat -A %i 2>NUL >res.txt && FOR /F %n in (admins.txt) DO @type res.txt | findst

r /I %n > NUL && echo [!] %n w
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729ibgXnxCKECOG2EkGrpxDjcSKsFQyUxxNHGKbdIqyvRNvfF9yoduP1pg/640?wx_fmt=png)

##### **2.6.4 查询远程系统中运行的任务**

如果目标机器在域系统中是通过共享的本地管理员账号运行的，就可以使用下列脚本来查询系统中的域管理任务。将目标域内主机 ip 保存为 ips.txt 文件，域管理员保存为 admins.txt 文件。

```
FOR /F %i in (ips.txt) DO @echo [+] %i && @tasklist /v /S %i /U user /P password 2 > NUL > output.txt && FOR /F %n in (admins.txt) Do @Type output.txt | findstr %n > NUL && echo [!] %n was found running a process on %i && pause
```

#### **2.7 通过 PowerView 模块**

PowerView 是由 Will Schroeder 开发的 PowerShell 脚本，属于 PowerSploit 框架和 Empire 的一部分。该脚本完全依赖于 PowerShell 和 WMI（Windows Management Instrumentation）查询。

下载地址：PowerTools/PowerView at master · PowerShellEmpire/PowerTools · GitHub

使用前需要进行导入，命令如下：

```
powershell-import
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S7292OPkVglgNolhZzacgIhZYSK90wBicHo4DW7icicYvJ2u37EXGL6VAVmpA/640?wx_fmt=png)

导入以后就可以执行命令进行域内信息收集了，常用命令如下表所示，具体命令使用可以查阅官方手册。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729yhQm8EgzHdC6AiccvxjCsGWGmSgytJVf151AumjzjOMWKib6GicgBN3Dg/640?wx_fmt=png)

下面演示其中几个命令的使用，其他命令的使用方式与之类似，大家可以自行尝试。

##### **2.7.1 Get-Domain**

该命令可以获取域的信息。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729XlmAjcXNtpUhebLjPenpfEicFV7aicmnURezhY4Ln6gZPV3l039QWia8g/640?wx_fmt=png)

##### **2.7.2 Get-DomainController**

该命令可以查看域控信息。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729WBfVtwXCboMcureYPiaP2pSTKpVfhWOdrvyhYOOs6lNPsNQ5zfRyhKg/640?wx_fmt=png)

##### **2.7.3 Get-DomainComputer**

该命令可以查看域内主机。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729kSJIicoGJVv36cgFdzmWFUhSDFNu9SU4udpyGYxsunIL2Y0rpDxP5Gw/640?wx_fmt=png)

##### **2.7.4 Get-NetLocalGroup**

该命令可以查看本地组。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729vUibXSh8yqIknjC9kcHNWX11qicibqoziauQqWnqyclibEVxwiboY9C5fFog/640?wx_fmt=png)

##### **2.7.5 Invoke-ShareFinder**

该命令可以查看域内共享。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S72937D3knVaTRaVRc6eKNU3oWM41W3Zq8SXwsQawIib3eVLKdGsgZiaUXOw/640?wx_fmt=png)

#### **2.8 判断当前用户**

##### **2.8.1 判断当前域用户是否是域内主机的超管**

因为普通域用户在进行一些高级别操作的配置时通常是需要域管理员的账号和密码，这是很不方便的。因此有的时候就会将普通的域用户增加到目标主机的超级管理员组，那么再做配置的时候就不需要域的超级管理员账号和密码。

可以通过下面的方式进行判断：

1、通过 net view 查看域内主机。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729IHjlibmdEt2qRO8Qbd260Hr5TbaHBe83BwLeHyOqsP0Miaolpe7SqH5A/640?wx_fmt=png)

2、通过 shell dir \\ 目标机器名 \ C$ 查看是否能够成功访问。

这里以 WIN7-SP 为例。

```
shell dir \\WIN7-SP\C$
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S72972QCouUyla3icb6rHCGHUjTdRUfKxDp2iaTexI3AnZJr9kRicj9NVhPRQ/640?wx_fmt=png)

可以看到成功访问到了 WIN7-SP 的 C 盘。

现在再来看看能不能成功访问域控 DC 的 C 盘。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S7294zFiciaic9RXtQVKSvMib03jaA7JaQGbAPxPBU7CDjZgomiaE9eDYS3JDBw/640?wx_fmt=png)

可以看到权限不够，说明本机用户是 WIN7-SP 的本地超级管理员。

我们也可以通过 powershell 脚本查询 WIN7-SP 的信息，命令如下。

```
powershell Get-NetLocalGroup -HostName WIN7-SP
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729iaxXTlQjVrtdkdicicmTtanBReeiamoUzJ2NnDnhdibLkxnSC4gqgVqhpGg/640?wx_fmt=png)

也可以通过如下命令查看登录过目标主机的用户。

```
shell dir /S /B \\WIN7-SP\c$\Users\ > user.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729o90t1vXfy3RTVb9jbRJI6icQApwUicKiciaicDCOmEKibVRgOPHupnqefWRA/640?wx_fmt=png)

##### **2.8.2 判断当前域用户是否是域管理员名**

1、查看 enterprise admins 组内用户，命令如下：

```
shell net group "enterprise admins" /domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729pqWLMkO6LFHs6buVcicYDXP4MQaibd1nicnTic2pZia05egKxLXTqCyC7icQ/640?wx_fmt=png)

2、查看 domain admins 组内用户，命令如下：

```
shell net group "domain admins" /domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729iaEoBrpoTicAfd22vFlBQepocUHvTjibT3z6MQtic9iaM9afeq3eCFiaad0Q/640?wx_fmt=png)

3、查看本地管理员组内用户，命令如下：

```
shell net localgroup "administrators" /domain
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHjD9sJ7gJDzB1JWYxV1S729vhiarOhGyTVTn3aEt3EZmIJZSHUzSM9jRfuhXy9fIfWQT14cfKMzPTw/640?wx_fmt=png)

**三、后语**

在本篇文章中，斗哥为大家介绍了内网信息收集的一些方式，在下一篇文章中，斗哥将继续为大家介绍在进行信息收集后，我们该如何获取用户的凭证信息进行横向渗透。

**四、参考链接**

1、《内网安全攻防渗透测试实战指南》   
2、CobaltStrike 的使用

**五、送书环节  
**

**以上三本任选其一！！**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtkW3Uc2v8n4HLKly7s5gyHLDvFTnpsmuMRLQk6EiboymDkDO7p2jAty6mLhiaKtmm8squXT5NluVQ/640?wx_fmt=png)

****【往期推荐】****  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - Shellcode Loader 原理 (python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - 分离 + 混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附 PoC】CVE-2021-2109 | Weblogic Server 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)

[【漏洞分析 | 附 EXP】CVE-2021-21985 VMware vCenter Server 远程代码执行漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487906&idx=1&sn=e35998115108336f8b7c6679e16d1d0a&chksm=ea37eef8dd4067ee13470391ded0f1c8e269f01bcdee4273e9f57ca8924797447f72eb2656b2&scene=21#wechat_redirect)

[【CNVD-2021-30167 | 附 PoC】用友 NC BeanShell 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487897&idx=1&sn=6ab1eb2c83f164ff65084f8ba015ad60&chksm=ea37eec3dd4067d56adcb89a27478f7dbbb83b5077af14e108eca0c82168ae53ce4d1fbffabf&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)