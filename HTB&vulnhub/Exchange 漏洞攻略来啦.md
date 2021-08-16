> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/EIiYn4cr_PmPT8YgiDAfaQ)

一、发现 Exchange

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/U5As78gibMD4uCNPb7ibV7x2TF1lFAeCyu7ngRB5njicztqI0DTVxfuo9rMpB0uYYqT7mg6xPic3NKb4e9S79qG5bQ/640?wx_fmt=png)

  

在渗透测试中, 当进行信息收集与环境侦察时, 发现与识别 Exchange 及其相关服务, 可以有多种方法与途径。

**1、地址遍历**

在公网上寻找 Exchange 邮件服务器可以通过访问目标域名的邮箱地址来寻找查看。或者通过 ZoomEye、showdan 等进行针对性查找。

对内网环境中的 Exchange 可以尝试遍历 ip 地址, 收集 https:\\ip\owa 的返回信息判断。

**2、端口与服务**

Exchange 的正常运行需要多个服务与功能组件之间相互依赖与协调, 因此, 安装了 Exchange 的服务器上会开放某些端口对外提供服务, 不同的服务与端口可能取决于服务器所安装的角色、服务器进行的配置、以及网络环境与访问控制的安全配置等。通过端口发现服务, 来识别确认服务器上安装了 Exchange , 是最常规也是最简易的方法。

但是此方法不推荐使用端口扫描容易流量异常被发现, 尤其是使用 nmap。

**3、SPNs 名称查询**

SPN(Service Principal Name), 是 Kerberos 认证中不可缺少的, 每一个启用 Kerberos 认证的服务都拥有一个 SPN, 如文件共享服务的 SPN 为 cifs/domain_name,LDAP 服务的 SPN 为 ldap/domain_name, 在 Kerberos 认证过程, 客户端通过指定 SPN 让 KDC 知晓客户端请求访问的是哪个具体服务, 并使用该服务对应的服务账号的密钥来对最终票据进行加密。

在活动目录数据库中, 每一个计算机对象有一个属性名为 servicePrincipalName, 该属性的值是一个列表, 存储着该计算机启用 Kerberos 认证的每一个服务名称。安装在 Windows 域环境中的 Exchange 服务同样会接入 Kerberos 认证, 因此, Exchange 相关的多个服务, 应该都可以从该属性中找到对应的 SPN。

执行 SPN 名称查找的工具和方法有很多, 直接以域内的一台工作机, 通过 setspn 查询获得。

SPN 是启用 Kerberos 的服务所注册的便于 KDC 查找的服务名称, 这些 SPN 名称信息被记录在活动目录数据库中, 只要服务安装完成, 这些 SPN 名称就已经存在, 除非卸载或删除, SPN 名称查询与当前服务是否启动没有关系（如 Exchange 服务器的 IMAP/POP 等部分服务默认是不启动的, 但其 SPN 名称同样存在）。

二、暴力破解

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/U5As78gibMD4uCNPb7ibV7x2TF1lFAeCyu7ngRB5njicztqI0DTVxfuo9rMpB0uYYqT7mg6xPic3NKb4e9S79qG5bQ/640?wx_fmt=png)

针对 Exchange 服务的利用, 包括各类漏洞在内, 都有一个很重要的前提, 就是必须要有一个有效的可登录用户账户。因此, 在发现 Exchange 服务之后, 最重要的一步就是获得用户账户。

**1、owa 登录爆破**

通常情况下, Exchange 系统是不会对邮箱登录次数做限制的, 因此, 利用大字典来进行爆破, 是最为简易和最为常见的一种突破方法。

Exchange 邮箱的登录账号分为三种形式, 分别为 “domain\username”、“username” 和“user@domain（邮件地址）”, 这三种方式可以并存使用, 也可以限制具体一种或两种使用。

具体使用哪一种用户名登录可以根据登录口的提示确定, 但这并不百分百准确, 管理员通过修改配置或者登录页面, 可以自行设置登录方式, 和提示说明。因此如果直接使用 owa 页面爆破, 用户名需要尝试全部三种方式。

爆破方式使用 burp 即可, 通过返回包长短即可判断成功与否。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9ywvMzxUQkHeGeTCHzSg56ebnPZXE7pxZrYDFj6Gsc2v2iaJIpnu9KNwg/640?wx_fmt=png)

**2、特殊接口爆破**

对于某些限制登录次数的网站, 还可以尝试对其 NTLM 验证接口进行爆破, 最常见的就是 ews 接口, 但除 ews 接 E 以外, 还有以下接口地址。

```
/Autodiscover/Autodiscover.xml
/Microsoft-Server-ActiveSync/default.eas
/Microsoft-Server-ActiveSync
/Autodiscover
/Rpc/
/EWS/Exchange.asmx
/EWS/Services.wsdl
/EWS/
/OAB/
/Mapi
```

| 

API 接口

 | 

说明

 |
| 

/autodiscover

 | 

自 Exchange Server 2007 开始推出的一项自动服务, 用于自动配置用户在 Outlook 中邮箱的相关设置, 简化用户登陆使用邮箱的流程。

 |
| 

/ecp “Exchange Control Panel”

 | 

Exchange 管理中心, 管理员用于管理组织中的 Exchange 的 Web 控制台

 |
| 

/ews “Exchange Web Services”

 | 

Exchange Web Service, 实现客户端与服务端之间基于 HTTP 的 SOAP 交互

 |
| 

/mapi

 | 

Outlook 连接 Exchange 的默认方式, 在 2013 和 2013 之后开始使用, 2010 sp2 同样支持

 |
| 

/Microsoft-Server-ActiveSync

 | 

用于移动应用程序访问电子邮件

 |
| 

/OAB “Offline Address Book”

 | 

用于为 Outlook 客户端提供地址簿的副本, 减轻 Exchange 的负担

 |
| 

/owa “Outlook Web APP”

 | 

Exchange owa 接口, 用于通过 web 应用程序访问邮件、日历、任务和联系人等

 |
| 

/powershell

 | 

用于服务器管理的 Exchange 管理控制台

 |
| 

/RPC

 | 

早期的 Outlook 还使用称为 Outlook Anywhere 的 RPC 交互

 |

注意, 使用常规的抓包软件并不能直接爆破 NTLM 验证, 但可以使用脚本来进行批量爆破。例如使用 Ruby 脚本写的简易工具：NTLM 登陆爆破。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9yfITpZ5pd8PhXfVARwUz7atWLOUkrr6lrZbW1TrelYwmBJvN2eibSJ6A/640?wx_fmt=png)

三、获取全局通讯录 GlobalAddressList

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/U5As78gibMD4uCNPb7ibV7x2TF1lFAeCyu7ngRB5njicztqI0DTVxfuo9rMpB0uYYqT7mg6xPic3NKb4e9S79qG5bQ/640?wx_fmt=png)

在获得一个有效账户后, 为了长期控制, 或者更全面的控制, 一般会选择获取邮箱全部邮件地址列表, 即全局通讯录 GlobalAddressList。

Exchange GlobalAddressList(全局地址列表) 包含 Exchange 组织中所有邮箱用户的邮件地址, 只要获得 Exchange 组织内任一邮箱用户的凭据, 就能够通过 GlobalAddressList 导出其他邮箱用户的邮件地址。

以下内容主要参考 3gstuden 大佬文章。

**1、通过 Outlook Web Access(OWA)**

需要获得邮件用户的明文口令, 登录 OWA 后, 选择联系人 ->All Users。

```
https://domainname/owa/#path=/people
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9yYDlvl2ElQcfIReksFwVAB6YkdiauT5lTAaQAicGFljQZOXGUMJLNsHiaQ/640?wx_fmt=png)

使用该目录获取通讯录列表, 可以通过 burp 修改返回邮件地址数量导出。之后使用正则匹配即可, 但操作相对繁琐。一般情况下, 当条数超过 1000 条之后, 返回数据包大小超过 5M。因此, 并不推荐使用。

**2、通过 Exchange Web Service(EWS)**

通过 EWS 接口, 可以实现客户端与服务端之间基于 HTTP 的 SOAP 交互。很多针对 Exchange 的二次开发, 都是基于该端口进行开发。通过该端口, 可以基本实现用户 web 接口（owa）全部操作。因此, 在 ews 接口开放的前提下, 可以使用该接口检索通讯录, 或下载邮件。使用该接口下载邮件时, 还可以不触发 已读 / 未读 标签变更。

微软官方说明中, 对 ews 语法功能修改有三个版本, 分别为 exchange server 2007、exchange server 2010、exchange server 2013。由于 2007 基本已经不再使用, 不过多讨论, 2013 及以上版本目前使用的与 2013 版本相同。

**2013 及以上**

对于 Exchange 2013 及更高版本, 无法使用查看文件夹的方式直接导出全部通讯录, 但是可以使用 FindPeople 操作实现。参考资料

需要注意, FindPeople 操作时必须指定搜索条件, 无法通过通配符直接获取所有结果, 因此只能通过遍历数字 0-9 和字母 a-z 作为指定搜索条件的方式, 覆盖全部结果, 之后去重即可。

但是需要注意的是, 使用 CVE-2018-8581 漏洞时, 一定概率并不能读取指定用户通讯录列表。

**2010 版本**

对于 Exchange 2010 及更低版本, 只能使用 ResolveName 操作。参考资料

这里需要注意, ResolveName 操作每次最多只能获得 100 个结果, 如果 GlobalAddressList 中的邮箱用户大于 100, 那么无法直接获得完整结果。

因此在使用 ResolveName 操作时, 可以加入搜索条件, 确保每次获得的结果能够少于 100, 通过多次搜索实现对全部结果的覆盖。

通常使用的方法：

搜索条件为任意两个字母的组合, 例如 aa、ab、ac….zz, 总共搜索 26*26=676 次, 一般情况下能够覆盖所有结果。

**3、通过 Outlook 客户端使用的协议**

Outlook 客户端通常使用的协议为 RPC、RPC over HTTP(也称作 Outlook Anywhere) 和 MAPI over HTTP。

在登录用户, 选择联系人 -> 通讯簿, 即可查看并导出完整的 GlobalAddressList 列表。

需要注意的是, MAPI over HTTP 是 Exchange Server 2013 Service Pack 1 (SP1) 中实现的新传输协议, 用来替代 RPC OVER HTTP (也称作 Outlook Anywhere)。但是在 Exchange2013 中默认没有启用 MAPI OVER HTTP , 而是使用的 RPC OVER HTTP , 需要手动开启, 而 Exchange2016 默认启用 MAPI OVER HTTP 的。

**1.MAPI OVER HTTP**

通过 MAPI OVER HTTP 读取 GlobalAddressList 可以使用 ruler , 但是该工具目前暂不支持 RPC over HTTP。

**2.RPC over HTTP**

通过 RPC over HTTP 读取 GlobalAddressList 可使用 ptswarm 的 Exchanger.py。  

流程如下：

```
# 获得 All Users 对应的guid
python exchanger.py $IP/$USERNAME:$PASSWORD@$DOMAINNAME nspi list-tables


# 读取 AddressList
python exchanger.py $IP/$USERNAME:$PASSWORD@$DOMAINNAME nspi dump-tables -guid $GUID
```

**4、通过 Offline Address Book (OAB)**

OAB, 即脱机通讯簿, 是缓存到 Outlook 客户端本地的通讯簿集副本, 以便 Outlook 用户在与服务器断开连接时可以访问通讯簿。为减轻 Exchange 服务器上的工作负载, 用户在使用 outlook 缓存模式时, 客户端将优先查询本地 OAB 。但是 OAB 本身存在一定滞后性, 默认每隔 480 分钟更新一次。因此, 新装环境是不存在该目录的。

手工更新方法, 在 Exchange Management Shell 中输入

```
Get-OfflineAddressBook | Update-OfflineAddressBook
```

可以手工更新, 或者在 ecp 中修改 OAB 更新时间后等待。  

**1. 读取 Autodiscover 配置信息**

访问的 URL：

```
https://domain/autodiscover/autodiscover.xml
# 不能直接通过 web 访问,只会看到报错页面。需要构造访问包。具体请参考https://www.4hou.com/posts/62jl
```

**2. 读取 OAB 文件列表**

访问的 URL：

```
https://<domain>/OABUrl/oab.xml
```

返回结果中包括多个 OAB 文件的列表。

**3. 下载 lzx 文件**

访问的 URL：

```
OABUrl/xx.lzx
```

以上步骤可以使用 3gstudent 的工具 checkAutodiscoverEX.py 。  

**4. 解码 lzx 文件**

对 lzx 文件解码, 需要使用工具 oabextract。  

下载后需要进行安装, 编译好可在 Kali 下直接使用的版本下载地址：  

http://x2100.icecube.wisc.edu/downloads/python/python2.6.Linux-x86_64.gcc-4.4.4/bin/oabextract

将 lzx 文件转换为 oab 文件的命令示例：

```
oabextract 4667c322-5c08-4cda-844a-253ff36b4a6a-data-5.lzx gal.oab
```

提取出 GAL 的命令示例：

```
strings gal.oab|grep SMTP
```

**5、域用户查询**

由于在 Exchange 中, 默认情况下, 所有的邮箱用户都会有一个与之对应的域用户, 因此通过其他手段直接获取域用户列表, 也可以同步获得邮箱用户列表。

注：所有邮箱用户都有对应的域用户, 但域用户不一定拥有邮箱, 需要管理员主动开启设置。

**1.ldap 查询**

ldap 轻型目录访问协议, 在 windows 系统中, 可以通过 ldap 获取域用户基本信息。同时, 如果 ldap 配置不当, 存在未授权访问漏洞, 可以直接通过 389 端口获取用户列表。  

可以在获得有效账户后使用工具获取用户列表.

Kali 系统通过 ldapsearch 获取所有用户邮件地址

```
ldapsearch -x -H ldap://$IP:389 -D "CN=$username,CN=Users,DC=gfinger,DC=com" -w $password -b "DC=gfinger,DC=com" |grep mail:
```

Windows 系统通过 PowerView 获取所有用户邮件地址

```
$uname=$username
$pwd=ConvertTo-SecureString $password -AsPlainText –Force
$cred=New-Object System.Management.Automation.PSCredential($uname,$pwd)
Get-NetUser -Domain gfinger.com -DomainController $IP -ADSpath "LDAP://DC=gfinger,DC=com" -Credential $cred | fl mail
```

**2. 域内查询**

域内查询可以使用传统的内网渗透方式导出域用户。也可以使用域管直接远程操作 Exchange 导出邮箱地址。

```
$User = "gfinger\$username"
$Pass = ConvertTo-SecureString -AsPlainText $password -Force
$Credential = New-Object System.Management.Automation.PSCredential -ArgumentList $User,$Pass
$Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri http://Exchange.gfinger.com/PowerShell/ -Authentication Kerberos -Credential $Credential
Import-PSSession $Session -AllowClobber
Get-Mailbox|fl PrimarySmtpAddress
Remove-PSSession $Session
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9yezNEF6ib0YjTaOn3WftoqaRevzAcyiaJ39P7XxdOwCyAticMlgcmdma7Q/640?wx_fmt=png)

四、NTLM 中继

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/U5As78gibMD4uCNPb7ibV7x2TF1lFAeCyu7ngRB5njicztqI0DTVxfuo9rMpB0uYYqT7mg6xPic3NKb4e9S79qG5bQ/640?wx_fmt=png)

**1、用户中继**

NTLM 中继攻击在 SMB、HTTP 协议中的应用讨论得比较多, 其实质是应用协议通过 NTLM 认证的方式进行身份验证, 因此, 利用 NTLM 进行认证的应用都可能遭受 NTLM 中继攻击。Exchange 服务器提供 RPC/HTTP、MAPI/HTTP、EWS 等接口, 都是基于 HTTP 构建的上层协议, 其登陆方式通过 NTLM 进行, 因此, NTLM 中继同样适用与 Exchange。

Exchange 的 NTLM 中继攻击由 William Martin 于 Defcon26 的演讲中提出并实现了利用工具 ExchangeRelayx 。

ExchangeRelayx 由 python 实现, 依赖安装完成并启动后, 会启动 SMB 服务和 2 个 HTTP 服务, SMB 服务和监听在 80 端口的 HTTP 服务用于接收受害者主机发送的认证, 监听在 8000 端口的 HTTP 服务是一个管理后台, 用于管理重放攻击成功的 Exchange 会话。该工具实现了将获取到的 Net-NTLM 哈希重放到真实 Exchange 服务器的 EWS 接口进行认证, 通过 EWS 获取用户邮箱的邮件信息、附件下载、创建转发规则、查询 GAL 等。

**2、域内提权**

由于 Exchange 本身机制问题, 在内网中, 使用 CVE-2018-8581 漏洞, 将管理员的 NTLM hash 中继到域控服务器上, 就可以获得域管权限, 实现域内提权。但是这种利用方式存在一定限制, 较为不易实现, 但仍不失为一种有效的提权方式。

五、Exchange Admin Center（ecp）管理

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/U5As78gibMD4uCNPb7ibV7x2TF1lFAeCyu7ngRB5njicztqI0DTVxfuo9rMpB0uYYqT7mg6xPic3NKb4e9S79qG5bQ/640?wx_fmt=png)

exchange server 默认将其管理页面入口 Exchange Admin Center（ecp）和其正常邮箱登录口 Outlook Web Access（owa）一同发布。默认登陆地址为

https://domain/ecp/

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9yPj4ZOBFIFbmUwZ3tpDOHbA5P8FxshXv7FaIfWia8poCw906ib5huQJJg/640?wx_fmt=png)

对于普通权限的用户, 该登录口无利用价值, 但管理员用户在该口登陆, 可完成权限内任意操作, 包括增、删、改邮箱, 添加规则, 设置代收, 修改用户权限等全部操作。

**1、邮箱托管**

Exchange 邮件服务存在一种机制, 可以设置权限将邮箱委托给指定用户管理使用。

这种委托可以是全局的委托, 可以通过后台修改；也可以是对单独文件夹进行委托, 用户自行对文件夹设置。

因此, 当 ecp 可登录且拥有管理员权限时, 就可以通过添加邮箱委托的方式, 实现邮箱控制。在默认情况下, 某些管理员在配置时, 组用户会默认拥有对组内用户的委托管理权限。

添加委托：

```
ecp ——> 收件人 ——> 目标用户 ——> 邮件委托 ——> 完全访问添加指定用户
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9y0aKeuaL6CZ1CnTYezLll2HibnC4jicSfpaQIWicE5fPlP0ePz0BPVmAfQ/640?wx_fmt=png)

添加完成后, 使用指定用户登录正常 owa 页面, 选择打开其他邮箱即可。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9y0L3h5kNWpoRqibGRACEK69aA5yPHQ6K11ia8ZAhgVjvERclCGSichkhfA/640?wx_fmt=png)

另一种邮箱文件夹的权限委托, 相对隐蔽, 在用户的指定文件夹上设置权限, 即可使其他用户具有访问操作权限。主要利用可以参考 CVE-2018-8581, 可以通过 ews 接口实现以上操作。

首先在目标用户文件夹添加指定用户权限。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9y0CO5pzUdKoQyrG6GJa0SNBiclYFvBRIdp93jPmlYa1YIiaplXmOBdBGg/640?wx_fmt=png)

在指定用户文件夹下添加共享文件用户。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9yZu17HEfUNc29iax0XJTq7Hetq5YguU09x1sJ3qahy894DfibNZAXX04g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9ysA0suGBdQTn3sS8bNRKFrUBD95EsKiaGEHpqkO6tYR1MvUEwh1nh7CQ/640?wx_fmt=png)

**2、邮箱管理员**

在 ecp 中也可以实现添加邮箱管理员权限。

注：域管 administrator 默认为邮箱管理员, 但邮箱管理员和域管其实并无关系。添加邮箱管理员不会修改用户域内权限。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9ySjZVwWyIpibKnqGU8P7ZnBDyGvX7fkZD2P8CtSYic1bPCySzMUp6exbA/640?wx_fmt=png)

**3、邮件检索**

在后台管理中, 还有一项多邮箱检索邮件的功能, 但较为耗时, 对于体量较大的邮件系统不建议使用。

```
合规性管理 ——> 就地电子数据展示和保留 ——> 添加规则
```

**4、全局规则**

在 ecp 后台, 可以添加全局规则, 此处的规则只是简单利用, 例如新建规则代收邮件, 将全部带有关键词 password 的邮件抄送指定邮箱一份。

六、规则同步

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/U5As78gibMD4uCNPb7ibV7x2TF1lFAeCyu7ngRB5njicztqI0DTVxfuo9rMpB0uYYqT7mg6xPic3NKb4e9S79qG5bQ/640?wx_fmt=png)

对于 Exchange 规则的利用其实还有很多其他方法。利用规则配合 Outlook 客户端可以实现对用户主机的入侵。

Outlook 是 Office 办公软件中用于管理电子邮件的专用软件, Exchange 邮箱用户使用 Outlook 进行邮件管理可以体验 Exchange 专用的各种功能, 也是应用非常广泛的办公软件之一。Outlook 功能非常强大, 其中的一些合法功能由于其特殊性, 当攻击者利用一些灵活的技巧往往可达成意想不到的效果。

**规则和通知功能的滥用**

Outlook 提供了一项 “规则和通知”（Rules and Alerts）的功能, 可以设置邮件接收和发送的策略, 分为规则条件和动作, 即用户定义当邮件满足某些条件时（如邮件主题包含特定词语）, 触发一个特定的动作, 这个动作可以是对邮件的管理、处置, 甚至是启动应用程序。

当攻击者拥有合法邮箱用户凭证的情况下, 可以利用该功能在正常用户收到符合某种条件的邮件时执行特定的命令, 例如反弹一个 shell。该利用方法需要注意：

*   攻击者已拥有有效的邮箱用户凭证；
    
*   当触发动作为启动应用程序时, 只能直接调用可执行程序, 如启动一个 exe 程序, 但无法为应用程序传递参数, 即无法利用 powershell 执行一句话代码进行反弹 shell（因为只能执行 powershell.exe 而无法传递后面的命令行参数）；
    
*   用户需要在开启 Outlook 的情况下触发规则条件才有效, 在未使用 Outlook 的情况下无法触发动作；但是, 用户通过其他客户端（如 OWA ）接收浏览了该邮件, 而后打开了 Outlook, 仍然可以触发该动作发生（只要这封邮件没有在打开 Outlook 之前删除）；
    
*   规则和通知可以通过 Outlook 进行创建、管理和删除, OWA 对规则和通知的操作可用项较少（无法创建 “启动应用程序” 的动作）；
    

该功能可以实现根据邮件主题或内容匹配启动指定应用程序, 因此, 可以作为一个合适的攻击面, 在满足一定条件的情况下进行利用。总结一下该攻击需要满足的条件：

*   攻击者需要拥有合法的邮箱用户凭证, 且该用户使用 Outlook 进行邮件管理；
    
*   攻击者需要通过 Outlook 登陆用户邮箱, 然后为其创建一条合适的规则, 将要执行的应用程序要么位于用户使用 Outlook 的主机上, 要么位于主机可访问到的位置（如内网共享文件夹、WebDAV 目录下等）；
    
*   Ruler 也提供了利用上述规则和通知功能, 可以通过命令行创建规则、发送邮件触发规则。通过结合 Empire、共享文件夹、ruler, 对该功能进行利用。
    

但是需要注意的是, 使用这种规则同步的方法依旧会触发杀软。例如在下载木马至本机这一行为会同时受到浏览器和杀软的同步检查, 成功几率偏低。

**主页设置功能的滥用**

在 Outlook 中, 提供了一个功能允许用户在使用 Outlook 的时候设置收件箱界面的主页, 可以通过收件箱的属性来设置加载外部 URL, 渲染收件箱界面。

收件箱主页 URL 作为收件箱的设置属性, 会在客户端 Outlook 和 Exchange 服务端之间进行同步, 而通过 MAPI/HTTP 协议与 Exchange 服务端的交互, 可以直接设置该属性。因此, 当已拥有合法邮箱凭证的前提下, 可以利用该功能, 为邮箱用户设置收件箱主页 URL 属性, 将其指向包含恶意代码的页面, 当用户在 Outlook 中浏览刷新收件箱时, 将触发加载恶意页面, 执行恶意脚本代码, 形成远程命令执行。

Outlook 收件箱主页指向的 URL 在 Outlook 中通过 iframe 标签加载, 其执行 wscript 或 vbscript 受沙箱环境限制, 无法使用脚本代码创建敏感的恶意对象, 即无法直接通过 CreateObject(Wscript.Shell) 的方式执行命令。但是, 此处可以通过载入与 Outlook 视图相关的 ActiveX 组件, 然后获取 ViewCtl1 对象, 通过该对象获取应用程序对象 OutlookApplication, 该对象即表示整个 Outlook 应用程序, 从而逃出 Outlook 沙箱的限制, 接着, 就可以直接通过 Outlook 应用程序对象调用 CreateObject 方法, 来创建新的应用程序对象 Wscript.Shell, 执行任意命令。

```
Set Application = ViewCtl1.OutlookApplication           # 取得顶层的Outlook应用程序对象,实现逃逸
Set cmd = Application.CreateObject("Wscript.Shell")     # 利用Outlook应用程序对象创建新的对象,执行系统命令
cmd.Run("cmd.exe")
```

实现该攻击需要的前提条件：

*   攻击者需要拥有合法的邮箱用户凭证, 且该用户使用 Outlook 进行邮件管理；
    
*   攻击者通过 Outlook 登陆用户邮箱, 为其收件箱属性设置主页 URL, 指向包含恶意脚本代码的页面；
    
*   ruler 提供了通过 MAPI/HTTP 的协议交互, 利用合法的邮箱凭证向服务端写入收件箱主页 URL 属性, 当用户使用 Outlook 并从 Exchange 服务端同步该设置时, 其随后对收件箱的刷新浏览将触发加载恶意网页, 并执行恶意代码。
    

七、其他

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/U5As78gibMD4uCNPb7ibV7x2TF1lFAeCyu7ngRB5njicztqI0DTVxfuo9rMpB0uYYqT7mg6xPic3NKb4e9S79qG5bQ/640?wx_fmt=png)

**隐藏文件夹**

对于 Exchange 用户邮箱, 将文件夹的扩展属性 PidTagAttributeHidden(0x10F4000B) 设置为 true 时, 该文件夹对于用户不可见, 但只要知道了隐藏文件夹的 Id, 依旧能够通过程序进行数据交互。利用这一特性, 可以将隐藏文件夹构造成文件信息中转地。

**邮件伪造**

传统套路, 不赘述。

对于 Exchange 邮箱系统, 拥有 Domain admin 权限的域用户, 可通过 outlook 直接指定发件人, 伪造任意发件人发送邮件。伪造邮件的方式十分简单, 且邮件头无法显示真实 IP。

*   使用 Outlook2013 客户端指定发件人发送邮件, 接收邮件直接显示伪造人的名字, 伪造成功。
    
*   使用 Outlook2016 客户端测试, 邮件接收方的发件人位置显示 "XXX 代表 XXX", 伪造失败。
    
      
    

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ib9NfrmtOiagPvKChepMzW9yoRAWtFVHgwefrIY9AibTVzAlfDJSRZr6xvlnpje4IJWHXfQSqVtb1jw/640?wx_fmt=png)