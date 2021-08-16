\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/-mo-/p/12150515.html)

### 0x01 Kerberos 简介

在 Kerberos 认证中，最主要的问题是如何证明 “你是你” 的问题，如当一个 Client 去访问 Server 服务器上的某服务时，Server 如何判断 Client 是否有权限来访问自己主机上的服务，同时保证在这个过程中的通讯内容即使被拦截或篡改也不影响通讯的安全性，这正是 Kerberos 解决的问题。在域渗透过程中 Kerberos 协议的攻防也是很重要的存在。

#### 1.1 Kerberos 协议框架

在 Kerberos 协议中主要是有三个角色的存在：

1\. 访问服务的 Client  
2\. 提供服务的 Server  
3.KDC（Key Distribution Center）密钥分发中心

其中 KDC 服务默认会安装在一个域的域控中，而 Client 和 Server 为域内的用户或者是服务，如 HTTP 服务， SQL 服务。在 Kerberos 中 Client 是否有权限访问 Server 端的服务由 KDC 发放的票据来决定。

kerberos 的简化认证认证过程如下图

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104193033936-1902959860.png)

如果把 Kerberos 中的票据类比为一张火车票，那么 Client 端就是乘客，Server 端就是火车，而 KDC 就是就是车站的认证系统。如果 Client 端的票据是合法的（由你本人身份证购买并由你本人持有）同时有访问 Server 端服务的权限（车票对应车次正确）那么你才能上车。当然和火车票不一样的是 Kerberos 中有存在两张票，而火车票从头到尾只有一张。

由上图中可以看到 KDC 又分为两个部分：

Authentication Server： AS 的作用就是验证 Client 端的身份（确定你是身份证上的本人），验证通过就会给一张 TGT（Ticket Granting Ticket）票给 Client 。

Ticket Granting Server： TGS 的作用是通过 AS 发送给 Client 的票（TGT）换取访问 Server 端的票（上车的票 ST）。ST（ServiceTicket）也有资料称为 TGS Ticket，为了和 TGS 区分，在这里就用 ST 来说明。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104193302209-94748933.png)

KDC 服务框架中包含一个 KRBTGT 账户，它是在创建域时系统自动创建的一个账号，可以暂时理解为他就是一个无法登陆的账号。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104193318866-917001200.png)

#### 1.2 Kerberos 认证流程

当 Client 想要访问 Server 上的某个服务时，需要先向 AS 证明自己的身份，然后通过 AS 发放的 TGT 向 Server 发起认证请求，这个过程分为三块：

```
The Authentication Service Exchange            Client与AS的交互
The Ticket-Granting Service (TGS) Exchange     Client与TGS的交互
The Client/Server Authentication Exchange      Client与Server的交互


```

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104193033936-1902959860.png)

```
1.ASREQ:   Client向KDC发起ASREQ,请求凭据是Client hash加密的时间戳
2.AS\_REP:  KDC使用Client hash进行解密，如果结果正确就返回用krbtgt hash加密的TGT票据，TGT里面包含PAC,PAC包含Client的sid，Client所在的组。
3.TGSREQ:  Client凭借TGT票据向KDC发起针对特定服务的TGSREQ请求
4.TGS\_REP: KDC使用krbtgt hash进行解密，如果结果正确，就返回用服务hash 加密的TGS票据(这一步不管用户有没有访问服务的权限，只要TGT正确，就返回TGS票据)
5.AP\_REQ:  Client拿着TGS票据去请求服务
6.AP\_REP:  服务使用自己的hash解密TGS票据。如果解密正确，就拿着PAC去KDC那边问Client有没有访问权限，域控解密PAC。获取Client的sid，以及所在的组，再根据该服务的ACL，判断Client是否有访问服务的权限。


```

#### 1.3 PAC

在 Kerberos 最初设计的几个流程里说明了如何证明 Client 是 Client 而不是由其他人来冒充的，但并没有声明 Client 有没有访问 Server 服务的权限，因为在域中不同权限的用户能够访问的资源是有区别的。

所以微软为了解决这个问题在实现 Kerberos 时加入了 PAC 的概念， PAC 的全称是 Privilege Attribute Certificate(特权属性证书)。可以理解为火车有一等座，也有二等座，而 PAC 就是为了区别不同权限的一种方式。

(1)PAC 的实现

当用户与 KDC 之间完成了认证过程之后， Client 需要访问 Server 所提供的某项服务时， Server 为了判断用户是否具有合法的权限需要将 Client 的 User SID 等信息传递给 KDC， KDC 通过 SID 判断用户的用户组信息，用户权限等， 进而将结果返回给 Server， Server 再将此信息与用户所索取的资源的 ACL 访问控制列表（Access Control Lists，ACL）进行比较，最后决定是否给用户提供相应的服务。

PAC 会在 AS\_REP 中 AS 放在 TGT 里加密发送给 Client，然后由 Client 转发给 TGS 来验证 Client 所请求的服务。

在 PAC 中包含有两个数字签名 PAC\_SERVER\_CHECKSUM 和 PAC\_PRIVSVR\_CHECKSUM ，这两个数字签名分别由 Server 端密码 HASH 和 KDC 的密码 HASH 加密。

同时 TGS 解密之后验证签名是否正确，然后再重新构造新的 PAC 放在 ST 里返回给客户端，客户端将 ST 发送给服务端进行验证。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104195141661-1455705539.png)

特别说明的是，PAC 对于用户和服务全程都是不可见的。只有 KDC 能制作和查看 PAC。

(2)Server 与 KDC

PAC 可以理解为一串校验信息，为了防止被伪造和篡改，原则上是存放在 TGT 里，并且 TGT 由 KDC hash 加密。同时尾部会有两个数字签名，分别由 KDC 密码和 server 密码加密，防止数字签名内容被篡改。

同时 PAC 指定了固定的 User SID 和 Groups ID，还有其他一些时间等信息，Server 的程序收到 ST 之后解密得到 PAC 会将 PAC 的数字签名发送给 KDC，KDC 再进行校验然后将结果已 RPC 返回码的形式返回给 Server。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104195321756-1630097602.png)

### 0x02 Kerberos 与 SPN

#### 2.1 SPN 简介

服务主体名称（SPN：ServicePrincipal Names）是服务实例（可以理解为一个服务，比如 HTTP、MSSQL）的唯一标识符。Kerberos 身份验证使用 SPN 将服务实例与服务登录帐户相关联。如果在整个林或域中的计算机上安装多个服务实例，则每个实例都必须具有自己的 SPN 。如果客户端可能使用多个名称进行身份验证，则给定服务实例可以具有多个 SPN 。SPN 始终包含运行服务实例的主机的名称，因此服务实例可以为其主机的每个名称或别名注册 SPN。

如果用一句话来说明的话就是如果想使用 Kerberos 协议来认证服务，那么必须正确配置 SPN。

#### 2.2 SPN 格式与配置：

在 SPN 的语法中存在四种元素，两个必须元素和两个额外元素:

```
<serviceclass>/<host>:<port>/<service name>


```

```
<service class>：标识服务类的字符串
<host>：服务所在主机名称
<port>：服务端口
<service name>：服务名称


```

CopyCopy

其中

`<service class>`

和

`<host>`

为必须元素

例：

如果我想把域中一台主机 S2 中的 MSSQL 服务注册到 SPN 中则可以使用命令:

```
Setspn-A MSSQLSvc/s2.yunying.lab:1433 tsvc


```

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104195719907-2083011880.png)

注册成功之后可以通过以下命令来查看已经注册的 SPN:

```
setspn -Tyunying.lab –q \*/\*
或
setspn –q \*/\*


```

SPN 在其注册的林中必须是唯一的。如果它不唯一，则身份验证将失败。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104195940744-375410440.png)

在注册 SPN 时，可以使用 NetBIOS 名称，如 s2。也可以使用 FQDN(FullyQualified Domain Name 全限定域名) ，如 s2.yunying.lab 。有可能存在某一种名称注册的 SPN 不能成功访问的情况，如果没有配置正确可以换一种名称试一试。

一般情况下基于主机的服务会省略后面两个组件，格式为 /：

```
MSSQLSvc/s2.yunying.lab


```

如果服务使用非默认端口或者此主机存在多个服务实例的情况下，需要包括端口号或服务名：

```
MSSQLSvc/ s2.yunying.lab:1433


```

#### 2.3 SPN 扫描

在了解了 Kerberos 和 SPN 之后我们可以通过 SPN 来获取我们想要的信息，比如想知道域内哪些主机安装了什么服务，我们就不需要再进行批量的网络端口扫描。

在一个大型域中通常会有不止一个的服务注册 SPN ，所以可以通过 “SPN 扫描” 的方式来查看域内的服务。相对于通常的网络端口扫描的优点是不用直接和服务主机建立连接，且隐蔽性更高。

(1) 扫描工具

扫描工具有多种，下面挑选几种较为常见的工具来说明一下：

```
Discover-PSMSSQLServers:  是 Powershell-AD-Recon 工具集中的一个工具，用来查询已经注册了的 MSSQL 类型的 SPN。
GetUserSPNs.ps1:          是 Kerberoast 工具集中的一个 powershell 脚本，用来查询域内注册的 SPN
PowerView.ps1:            在 Powersploit 和 Empire 工具里都有集成，PowerView 相对于上面几种是根据不同用户的 objectsid 来返回，返回的信息更加详细。


```

(2) 原理说明

在 SPN 扫描时我们可以直接通过脚本，或者命令去获悉内网已经注册的 SPN 内容。那如果想了解这个过程是如何实现的，就需要提到 LDAP 协议。

LDAP 协议全称是 LightweightDirectory Access Protocol，一般翻译成轻量目录访问协议。是一种用来查询与更新 Active Directory 的目录服务通信协议。AD 域服务利用 LDAP 命名路径（LDAP naming path）来表示对象在 AD 内的位置，以便用它来访问 AD 内的对象。

那些 Powershell 脚本其实主要就是通过查询 LDAP 的内容并对返回结果做一个过滤，然后展示出来。

### 0x03 Kerberoasting

在前面介绍 Kerberos 的认证流程时说到，在 TGS\_REP 中，TGS 会返回给 Client 一张票据 ST，而 ST 是由 Client 请求的 Server 端密码进行加密的。当 Kerberos 协议设置票据为 RC4 方式加密时，我们就可以通过爆破在 Client 端获取的票据 ST，从而获得 Server 端的密码。

下图为设置 Kerberos 的加密方式，在域中可以在域控的 “组策略管理” 中进行设置：设置完成之后运行里输入 “gpupdate” 刷新组策略，策略生效。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104200744534-1747549875.png)

#### 3.1 早期的 Kerberoasting

攻击流程：

1\. 在域内主机 s1 中通过 Kerberoast 中的 GetUserSPNs.ps1 或者 GetUserSPNs.vbs 进行 SPN 扫描。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104201019622-1230459103.png)

2\. 根据扫描出的结果使用微软提供的类 Kerberos Requestor Security Token 发起 kerberos 请求，申请 ST 票据。

```
PS C:\\> Add-Type -AssemblyNameSystem.IdentityModel
PS C:\\> New-ObjectSystem.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList"MSSQLSvc/s2:1433"


```

可以看到这个过程通过 AS-REQ、AS-REP、TGS-REQ、TGS-REP 这四个认证流程，获取到 RC4 方式加密的票据。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104201804785-1293962172.png)

3.Kerberos 协议中请求的票据会保存在内存中，可以通过 klist 命令查看当前会话存储的 kerberos 票据:

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104201900007-982063241.png)

这里可以借助 mimikatz 导出：

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104201932502-1233130076.png)

使用 kerberoast 工具集中的 tgsrepcrack.py 工具进行离线爆破，成功得到 tsvc 账号的密码 admin1234! ：

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104202039321-119232534.png)

#### 3.2 Kerberoasting 的 “新姿势”

攻击流程：

1\. 在之前的 Kerberoasting 中需要通过 mimikatz 从内存中导出票据，这里使用的是 Empire 中的 Invoke-Kerberoast.ps1；Invoke-Kerberoast 通过提取票据传输时的原始字节，转换成 John the Ripper 或者 HashCat 能够直接爆破的字符串。

```
Invoke-kerberoast –outputformat hashcat |fl


```

2\. 这里–outputformat 参数可以指定输出的格式，可选 John the Ripper 和 Hashcat 两种格式，这里以 Hashcat 做演示：

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104202315528-1808929428.png)

CopyCopy

这个脚本申请访问的是

`MSSQLSvc/s2.yunying.lab:1433`

这个 SPN ，查看数据包可以看到 Invoke-Kerberoast 输出的 Hash 值就是 TGS-REP 中返回的票据内容，然后拼接成了 Hashcat 可以直接爆破的格式（以

`$krb5tgs$23*`

开头的）

3\. 可以把内容保存至文档，也可以直接重定向到 TXT 文件：

```
PS C:> Invoke-Kerberoast-Outputformat Hashcat | fl > test1.txt


```

4\. 使用 HASHCAT 工具进行破解：

```
PSC:> hashcat64.exe –m 13100 test1.txt password.list --force


```

Copy

在这里

`–m`

表示选择不同的加密类型，其中 13100 对应的是 Kerberos 5 TGS-REP 类型的密文。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104202527173-2133046556.png)

可以看到这里已经离线破解成功，输出了 s2 的密码 admin1234!。

Kerberoasting 的本质是通过破解在 Kerberos 认证流程中的 TGS\_REP 这个过程中 TGS 返回给 Client 的票据内容来进行密码的获取，在一个大型的域中还是有一定的利用价值，并且这种方式是离线爆破，过程较为隐蔽。

### 0x04 MS14-068

当我们拿到了一个普通域成员的账号后，想继续对该域进行渗透，拿到域控服务器权限。如果域控服务器存在 MS14\_068 漏洞，并且未打补丁，那么我们就可以利用 MS14\_068 快速获得域控服务器权限。

MS14-068 编号 CVE-2014-6324，补丁为 3011780，如果自检可在域控制器上使用命令检测。

```
systeminfo |find "3011780"


```

如下，为空说明该服务器存在 MS14-068 漏洞。

![](https://img2018.cnblogs.com/blog/1561366/201911/1561366-20191119163917194-962719061.png)

这个之前有写过：[\[内网渗透\]MS14-068 复现 (CVE-2014-6324)](https://www.cnblogs.com/-mo-/p/11890539.html)

这个漏洞产生的原因主要是以下几个问题：

```
A、在域中默认允许设置 Include-pac 的值为 False（不能算漏洞，应该是微软对于某些特定场景的特殊考虑设计出的机制）
B、PAC 中的数字签名可以由 Client 端指定，并且Key的值可以为空
C、PAC 的加密方式也可以由 Client 指定，并且Key的值为 generate\_subkey 函数生成的16位随机数
D、构造的 PAC 中包含高权限组的 SID 内容


```

通过以上几点， Client 完全伪造了一个 PAC 发送给 KDC ，并且 KDC 通过 Client 端在请求中指定的加密算法来解密伪造的 PAC 以及校验数字签名，并验证通过。KDC 在根据对伪造的 PAC 验证成功之后，返回给 Client 端一个新的 TGT，也就是说这时 Client 已经获得了一张包含有高权限 PAC 内容的正常的 TGT 票据（564eab 开头）。

在这个漏洞中主要的问题是存在于 KDC 会根据客户端指定 PAC 中数字签名的加密算法，以及 PAC 的加密算法，来校验 PAC 的合法性。这使得攻击者可通过伪造 PAC，修改 PAC 中的 SID，导致 KDC 判断攻击者为高权限用户，从而导致权限提升漏洞的产生。

### 0x05 Golden Ticket

#### 5.1 简介

Golden Ticket（下面称为金票）是通过伪造的 TGT（Ticket Granting Ticket），因为只要有了高权限的 TGT ，那么就可以发送给 TGS 换取任意服务的 ST 。可以说有了金票就有了域内的最高权限。

制作金票的条件：

```
域名称
域的SID值
域的KRBTGT账户密码HASH
伪造用户名，可以是任意的


```

#### 5.2 实验流程

1\. 金票的生成需要用到 krbtgt 的密码 HASH 值，可以借助 mimikatz 来获取 krbtgt 的值：

```
lsadump::dcsync/domain:yunying.lab /user:krbtgt


```

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104204041548-1855717171.png)

如果已经通过其他方式获取到了 KRBTGT HASH 也可以直接进行下一步。

2\. 得到 KRBTGT HASH 之后使用 mimikatz 中的 kerberos::golden 功能生成金票 golden.kiribi，即为伪造成功的 TGT。

```
kerberos::golden /admin:administrator /domain:yunying.lab /sid:\*\*\* /krbtgt:\*\*\* /ticket:golden.kiribi


```

参数说明：

```
/admin:     伪造的用户名
/domain:    域名称
/sid:       SID值，注意是去掉最后一个-后面的值
/krbtgt:    krbtgt的HASH值
/ticket:    生成的票据名称


```

3\. 金票的使用:

通过 mimikatz 中的 kerberos::ptt 功能（Pass The Ticket）将 golden.kiribi 导入内存中:

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104204559527-1462572496.png)

4\. 已经可以通过 dir 成功访问域控的共享文件夹。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104204624365-1413845069.png)

这样的方式导入的票据 20 分钟之内生效，如果过期再次导入就可以，并且可以伪造任意用户。

### 0x06 Silver Tickets

#### 6.1 简介

Silver Tickets（下面称银票）就是伪造的 ST（Service Ticket），因为在 TGT 已经在 PAC 里限定了给 Client 授权的服务（通过 SID 的值），所以银票只能访问指定服务。

制作银票的条件：

```
域名称
域的SID值
域的服务账户的密码HASH（不是krbtgt，是域控）
伪造的用户名，可以是任意用户名，这里是silver


```

#### 6.2 实验流程

1\. 首先我们需要知道服务账户的密码 HASH ，这里同样拿域控来举例，通过 mimikatz 查看当前域账号 administrator 的 HASH 值。注意，这里使用的不是 Administrator 账号的 HASH，而是 DC$ 的 HASH。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104205038107-81911087.png)

2\. 通过 mimikatz 生成银票:

```
kerberos::golden /domain:yunying.lab /sid:\*\*\* /target:dc.yunying.lab /service:cifs /rc4:\*\*\* /user:silver /ptt


```

参数说明：

```
/domain:    当前域名称
/sid:       SID值，和金票一样取前面一部分
/target:    目标主机，这里是 dc.yunying.lab
/service:   服务名称，这里需要访问共享文件，所以是 cifs
/rc4:       目标主机的 HASH 值
/user:      伪造的用户名,这里是silver
/ptt:       表示的是 Pass The Ticket攻击，是把生成的票据导入内存，也可以使用/ticket导出之后再使用kerberos::ptt来导入


```

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104205400009-319231197.png)

3\. 这时通过 klist 查看本机的 kerberos 票据可以看到生成的票据:

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104205721512-836014394.png)

4\. 可以成功访问 DC 的共享文件夹。

```
dir \\\\dc.yunying.lab\\c$ 


```

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104205754194-877986285.png)

银票生成时没有 KRBTGT 的密码，所以不能伪造 TGT 票据，只能伪造由 Server 端密码加密的 ST 票据，只能访问指定的服务。

### 0x07 Enhanced Golden Tickets(增强型金票)

#### 7.1 简介

在 Golden Ticket 部分说明可利用 krbtgt 的密码 HASH 值生成金票，从而能够获取域控权限同时能够访问域内其他主机的任何服务。但是普通的金票不能够跨域使用，也就是说金票的权限被限制在当前域内。

#### 7.2 普通金票的局限性

为什么普通金票会被限制只能在当前域内使用？

在上一篇文章中说到了域树和域林的概念，同时说到 YUNYING.LAB 为其他两个域（NEWS.YUNYING.LAB 和 DEV.YUNYING.LAB）的根域，根域和其他域的最大的区别就是根域对整个域林都有控制权。而域正是根据 Enterprise Admins 组（下文会说明）来实现这样的权限划分。在一个域林中，域控权限不是终点，根域的域控权限才是域渗透的终点。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104210724889-1128727066.png)

#### 7.3 突破限制

普通的黄金票据被限制在当前域内，在 2015 年 Black Hat USA 中国外的研究者提出了突破域限制的增强版的黄金票据。通过域内主机在迁移时 SIDHistory 属性中保存的上一个域的 SID 值制作可以跨域的金票。这里没有迁移，直接拿根域的 SID 号做演示。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104211000229-204386879.png)

如果知道根域的 SID 那么就可以通过子域的 KRBTGT 的 HASH 值，使用 mimikatz 创建具有 Enterprise Admins 组权限（域林中的最高权限）的票据。环境与上文普通金票的生成相同。

1\. 通过 whoami /all 可以看到 YUNYING.LAB 中 Enterprise Admins 组的 SID 号是：

```
S-1-5-21-4249968736-1423802980-663233003-519


```

2\. 通过 klist purge 删除当前保存的 Kerberos 票据，也可以在 mimikatz 里通过 kerberos::purge 来删除。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104211235417-432103476.png)

3\. 然后通过 mimikatz 重新生成包含根域 SID 的新的金票:

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104211629831-856789567.png)

注意这里是不知道根域 YUNYING.LAB 的 krbtgt 的密码 HASH 的，使用的是子域 NEWS.YUNYING.LAB 中的 KRBTGT 的密码 HASH。

4\. 然后再通过 dir 访问 DC.YUNYING.LAB 的共享文件夹，发现已经可以成功访问。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104211724024-2108945265.png)

此时的这个票据票是拥有整个域林的控制权的。我们知道制作增强金票的条件是通过 SIDHistory ，那么防御方法就是在域内主机迁移时进行 SIDHistory 过滤，它会擦除 SIDHistory 属性中的内容。

### 0x08 委派

#### 8.1 简介

委派在域环境中其实是一个很常见的功能，对于委派的利用相较于先前说的几种攻击方式较为 “被动”，但是一旦利用也会有很大的危害。

先说一下委派是什么意思吧：在域中如果出现 A 使用 Kerberos 身份验证访问域中的服务 B，而 B 再利用 A 的身份去请求域中的服务 C ，这个过程就可以理解为委派。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104212135824-1443760404.png)

User 访问主机 s2 上的 HTTP 服务，而 HTTP 服务需要请求其他主机的 SQLServer 数据库，但是 S2 并不知道 User 是否有权限访问 SQLServer ，这时 HTTP 服务会利用 User 的身份去访问 SQLServer，只要 User 有权限访问 SQLServer 服务就能访问成功。

而委派主要分为非约束委派（Unconstraineddelegation）和约束委派（Constrained delegation）两个方式，下面分别介绍两种方式如何实现。

#### 8.2 非约束委派

非约束委派在 Kerberos 中实现时，User 会将从 KDC 处得到的 TGT 发送给访问的 service1（可以是任意服务），service1 拿到 TGT 之后可以通过 TGT 访问域内任意其他服务，所以被称为非约束委派。

非约束委派的设置：

Windows 域中可以直接在账户属性中设置:

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104212923790-1331088807.png)

流程图如下：

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104213014593-1531735669.png)

可以看到在前 5 个步骤中 User 向 KDC 申请了两个 TGT （步骤 2 和 4），一个用于访问 Service1 一个用于访问 Service2 ，并且会将这两个都发给 Service1 。并且 Service1 会将 TGT2 保存在内存中。

#### 8.3 约束委派

由于非约束委派的不安全性，微软在 windows2003 中发布了约束委派的功能。约束委派在 Kerberos 中 User 不会直接发送 TGT 给服务，而是对发送给 service1 的认证信息做了限制，不允许 service1 代表 User 使用这个 TGT 去访问其他服务。这里包括一组名为 S4U2Self（Service for User to Self）和 S4U2Proxy（Service forUser to Proxy）的 Kerberos 协议扩展。

从下图可以看到整个过程其实可以分为两个部分，第一个是 S4U2Self 的过程（流程 1-4），第二个是 S4U2Proxy 的过程（流程 5-10）。

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104213342492-751058795.png)

在这个过程中，S4U2Self 扩展的作用是让 Service1 代表用户向 KDC 验证用户的合法性，并且得到一个可转发的 ST1。S4U2Proxy 的作用可以说是让 Service1 代表用户身份通过 ST1 重新获取 ST2，并且不允许 Service1 以用户的身份去访问其他服务。同时注意 forwardable 字段，有 forwardable 标记为可转发的是能够通过 S4U2Proxy 扩展协议进行转发的，如果没有标记则不能进行转发。

约束委派的配置：

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200104213521151-927603071.png)

### 0x09 参考链接

[Windows 内网协议学习专栏—安全客](https://www.anquanke.com/subject/id/193604)

[Kerberos 协议探索系列之扫描与爆破篇](https://mp.weixin.qq.com/s?__biz=MzUyOTc3NTQ5MA==&mid=2247484224&idx=1&sn=28d155a62ac96e331b53f6c198574bfa&chksm=fa5aadadcd2d24bb2e95cb98dd9411c40e664b959e8181de07ba3fb9d2fb4bff6d8fd3885a3c&mpshare=1&scene=23&srcid=&sharer_sharetime=1577065203780&sharer_shareid=d32981e13d51bf06188894426d2a54e5#rd)

[Kerberos 协议探索系列之票据篇](https://mp.weixin.qq.com/s?__biz=MzUyOTc3NTQ5MA==&mid=2247484307&idx=1&sn=52115ec7df30ee0f69edd5a0ab1c0304&chksm=fa5aad7ecd2d24685774ee25c6a0eb7a2ef7f47685073eb44526110260510d6f27ef2301d7f2&mpshare=1&scene=23&srcid=&sharer_sharetime=1577065217335&sharer_shareid=d32981e13d51bf06188894426d2a54e5#rd)

[Kerberos 协议探索系列之委派篇](https://mp.weixin.qq.com/s/_wwTo7JcFV_lXaxhgMFJCQ)