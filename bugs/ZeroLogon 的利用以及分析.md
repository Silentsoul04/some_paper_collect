\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/S9Hwb1-lLhh4QfI4b551SQ)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wkV2bs7wnUlOsibEM98KsUuvLBcicmbUNj0dMB6ZtdRsNHvJ3rdVdkhXQ/640?wx_fmt=png)

作者：daiker & wfox @360Linton-Lab

在今年 9 月份，国外披露了 CVE-2020-1472(又被叫做 ZeroLogon) 的漏洞详情，网上也随机公开了 Exp。是近几年 windows 上比较重量级别的一个漏洞。通过该漏洞，攻击者只需能够访问域控的 445 端口，在无需任何凭据的情况下能拿到域管的权限。该漏洞的产生来源于 Netlogon 协议认证的加密模块存在缺陷，导致攻击者可以在没有凭证的情况情况下通过认证。该漏洞的最稳定利用是调用 netlogon 中 RPC 函数 NetrServerPasswordSet2 来重置域控的密码，从而以域控的身份进行 Dcsync 获取域管权限。

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png)

0x00 漏洞的基本利用

首先来谈谈漏洞的利用。

### **1\. 定位域控**

在我们进入内网之后，首先就是快速定位到域控所在的位置。下面提供几种方法

1、批量扫描 389 端口。

如果该机器同时开放着 135,445,53 有很大概率就是域控了，接下来可以通过 nbtscan，smbverion，oxid，ldap 来佐证

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wK5IziaAp2nMRIY4yC3eJe9EGrmQSTOYfuOHkmy5UxVice6vbryyUQiaew/640?wx_fmt=png)

2、如果知道域名的话，可以尝试通过 dns 查询

当然这种也有很大的偶然性，需要跟域共享一套 DNS，在实战中有些企业内网会这样部署，可以试试。

Linux 下命令有

```
net time /domainnet group "Domain controllers" /domain dsquery server -o rdnadfind -sc dclistNltest /dclist:域名
```

Windows 下命令有

```
python secretsdump.py   test.local/DC2016\\$@DC2016    -dc-ip  192.168.110.16   -just-dc-user test\\\\administrator -hashes 31d6cfe0d16ae931b73c59d7e0c089c0:31d6cfe0d16ae931b73c59d7e0c089c0
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wAGtzIeFOH5clBcrk6hia5HhSBctSddy2M8cdCzgREEF3ibNf8hQGzeuQ/640?wx_fmt=png)

3、如果我们控制了一台域成员机器，可以直接查询。

以下是一些常见的查询命令

```
\# Section 3.1.4.4.1
def ComputeNetlogonCredentialAES(inputData, Sk):
    IV='\\x00'\*16
    Crypt1 = AES.new(Sk, AES.MODE\_CFB, IV)
    return Crypt1.encrypt(inputData)
```

### **2\. 重置域控密码**

这里利用 CVE-2020-1472 来重置域控密码。注意，这里是域控密码，不是域管的密码。是域控这个机器用户的密码。可能对域不是很熟悉的人对这点不是很了解。在域内，机器用户跟域用户一样，是域内的成员，他在域内的用户名是机器用户 +$(如 DC2016\\$)，在本地的用户名是 SYSTEM。

机器用户也是有密码的，只不过这个密码我们正常无感，他是随机生成的，密码强度是 120 个字符，高到无法爆破，而且会定时更新。

我们通过`sekurlsa::logonPasswords`就可以看到机器用户的密码

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wRFTwm7PrrTERUDD69c4FLial31dmIywqJ6UmTEAxptj7gFkBguWvROA/640?wx_fmt=png)

在注册表`HKLM\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters`

`DisablePasswordChange`决定机器用户是否定时更新密码，默认是 0，定时更新

`MaximumPasswordAge`决定机器用户更新的时间，默认是 30 天。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wUlnXaHmF5OiaWLHum3iceEia7pPWaWnEjaMAvIBefksR3vy3G7Sva7LIg/640?wx_fmt=png)

接下来开始利用

命令在`python cve-2020-1472-exploit.py 机器名 域控IP`

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wvCT838aJTiaKkChHAic4hqRMewiaQCZOsJef5z1ibqeEXA1JiaaL6qkVBsw/640?wx_fmt=png)

这一步会把域控 DC2016(即 DC2016\\$ 用户) 的密码置为空，即 hash 为`31d6cfe0d16ae931b73c59d7e0c089c0`

接下来使用空密码就可以进行 Dcsync(直接登录不行吗？在拥有域控的机器用户密码的情况下，并不能直接使用该密码登录域控，因为机器用户是不可以登录的，但是因为域控的机器用户具备 Dcsync 特权，我们就可以滥用该特权来进行 Dcsync)

这里面我们使用 impacket 套件里面的`secretsdump`来进行 Dcsync。

```
NTSTATUS NetrServerPasswordSet2(
   \[in, unique, string\] LOGONSRV\_HANDLE PrimaryName,
   \[in, string\] wchar\_t\* AccountName,
   \[in\] NETLOGON\_SECURE\_CHANNEL\_TYPE SecureChannelType,
   \[in, string\] wchar\_t\* ComputerName,
   \[in\] PNETLOGON\_AUTHENTICATOR Authenticator,
   \[out\] PNETLOGON\_AUTHENTICATOR ReturnAuthenticator,
   \[in\] PNL\_TRUST\_PASSWORD ClearNewPassword
);
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wOBqOTHjpGjFMelPPInsgK37knjmuHSLh56zEoeObDfQQtWibyMB6A5g/640?wx_fmt=png)

### **3\. 恢复脱域的域控**

在攻击过程中，我们将机器的密码置为空，这一步是会导致域控脱域的，具体原因后面会分析。其本质原因是由于机器用户在 AD 中的密码 (存储在 ntds.dic) 与本地的注册表 / lsass 里面的密码不一致导致的。所以要将其恢复，我们将 AD 中的密码与注册表 / lsass 里面的密码保持一致就行。这里主要有三种方法

1、从注册表 / lsass 里面读取机器用户原先的密码，恢复 AD 里面的密码

我们直接通过`reg save`命令 将注册表里面的信息拿回本地，通过 secretsdump 提取出里面的 hash。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wUO7cucWUWMRkSooibgLglthpdchaDhbxrVhD0zCQECj8NSyK4VsXUibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wiaLrAH1LNV5IewtmbrjnunticnL2nnRhuV27JNuqPA7ezJ4oiaGLUZWqA/640?wx_fmt=png)

或者使用 mimikatz 的`sekurlsa::logonpassword`从 lsass 里面进行抓取

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wI02Tqf1llsOVUMHP4mQvp2icftkwgHC4ZXHIVg1b6684jemhzUSNz1g/640?wx_fmt=png)

可以使用 CVE-2020-1472 底下的 restorepassword.py 来恢复

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wAoMKWiazHDS0YJDWTxjLyicmrQv4lkytVibt7uJo37ibIAaSSIf8Il3n4Q/640?wx_fmt=png)

也可使用 zerologon 底下的 reinstall\_original\_pw.py 来恢复，这个比较暴力，再打一次，计算密码的时候使用了空密码的 hash 去计算 session\_key。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wg3VO8CooNcGiaC4EyjoZbAkksdY7eGCFFW6Fic2CP3dUxbpwlpdSibeuw/640?wx_fmt=png)

可以发现 AD 里面的密码已经恢复如初了

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wzmAJAIsy6WDhot5iaZOH9R67rgAy6sYhz3yPp370IBZHRl9iboeicvNkg/640?wx_fmt=png)

2、从 ntds.dict 里面读取 AD 历史密码，然后恢复 AD 里面的密码

只需要加 secretsdump 里面加`-history`参数就行

这个不太稳定，我本地并没有抓到历史密码

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wFL2LqaFHeHKw7RrYHMicPG95tJDfARqNszCbxBibUIURd4N4ZTeQNn6w/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wicq4icto4zQnQibbiciabI1abj2AaQuwL052CWVeF6kKXZbO1PRqSgHAkCw/640?wx_fmt=png)

3、一次性重置计算机的机器帐户密码。(包括 AD，注册表，lsass 里面的密码)。

这里使用一个 powershell 的 cmdlet`Reset-ComputerMachinePassword`, 他是微软在计算机脱域的情况下给出的一种解决方案。

可以一次性重置计算机的机器帐户密码。(包括 AD，注册表，lsass 里面的密码)。

我们用之前 dcsync 获取的域管权限登录域控。

执行`powershell Reset-ComputerMachinePassword`

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wJTyQibibMgO2Y52cy92cZ474mFFnMfSfx9EcSkBhQemeNCgicBVHM3WbQ/640?wx_fmt=png)

可以看到三者的 hash 已经保持一致了

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wtz8QIl6CSZgIAPS2Kf9oZ1ibQQVCMEsG5bzyqgibibmWvPdNrZlxswEzg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wfw4ctSbcLiaZzNFrWGfHbmzZfwQQfItjHM3yEicfbXwdUQfMqia4UPAfw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wxicqpfS57g3XKfrhIGdCVH0CvphUURIZbwRQa9keHkKB8nAEoRd4pVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png)

0x01 漏洞分析

### **1、netlogon 用途**

Netlogon 是 Windows Server 进程，用于对域中的用户和其他服务进行身份验证。由于 Netlogon 是服务而不是应用程序，因此除非手动或由于运行时错误而停止，否则 Netlogon 会在后台连续运行。Netlogon 可以从命令行终端停止或重新启动。其他机器与域控的 netlogon 通讯使用 RPC 协议 MS-NRPC。

MS-NRPC 指定了 Netlogon 远程协议，主要功能有基于域的网络上的用户和计算机身份验证；为早于 Windows 2000 备份域控制器的操作系统复制用户帐户数据库；维护从域成员到域控制器，域的域控制器之间以及跨域的域控制器之间的域关系；并发现和管理这些关系。

我们在 MS-NRPC 的文档里面可以看到为了维护这些功能所提供的 RPC 函数。机器用户访问这些 RPC 函数之前会利用本身的 hash 进行校验，这次的问题就出现在认证协议的校验上。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83weXlIbUYSibaNnRTib8e2lCTGnoWESnoRjgazlfhlPhcjYbVQbSTgkvfQ/640?wx_fmt=png)

### **2、IV 全为 0 导致的 AES\_CFB8 安全问题**  

来看下 AES\_CFB8 算法的一个安全问题。

首先说下 CFB 模式的加解密

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wejSAicwtr8ibjC2a5LLPNwicTPfV9LZRNa2frFrxU00CPAhlLKN6JyBqQ/640?wx_fmt=png)

CFB 是一种分组密码，可以将块密码变为自同步的流密码。

其加解密公式如下

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wbbNM14jrQlaL7rg7nkMWQpWIh3uWhJX48Q8Z4M1hiazb1e8GXnpTpdg/640?wx_fmt=png)

既将明文拆分为 N 份，C1，C2，C3。  

每一轮的密文的计算是，先将上一轮的密文进行加密 (在 AES\_CFB 里面是使用 AES 进行加密)，然后异或明文，得到新一轮的密文。

这里需要用到上一轮的密文，由于第一轮没有上一轮。所以就需要一个初始向量参与运算，这个初始向量我们成为 IV。

下面用一张图来具体讲解下。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83w25d3zCnM6zq3icJaoBTvXHRzs3J2YIWkqFSstiaFL7JtL6z3MAPFEN5w/640?wx_fmt=png)

这里的 IV 是`fab3c65326caafb0cacb21c3f8c19f68`

明文是`0102030405060708`

第一轮没有上一轮，需要 IV 参与运算。那么第一轮的运算就是。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wv34ZlvXJdOXL54XuKViaLw3cziakuPKp5712y73vaNGNFVpGCCUoDbQQ/640?wx_fmt=png)

E(`fab3c65326caafb0cacb21c3f8c19f68`) = `e2xxxxxxxxxxxxxxxxxxxxxxxxxxxxx`  

然后 e2 与明文 01 异得到密文 e3。

第二轮的密文计算是，先将第一轮的密文进行 AES 加密，然后异或明文，密文。

第一轮的密文就是`(没有fa了)b3c65326caafb0cacb21c3f8c19f68`+`e2`\=`b3c65326caafb0cacb21c3f8c19f68e2`

E(`b3c65326caafb0cacb21c3f8c19f68e2`\=`9axxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

然后 91 与明文异或得到密文 98

我们用一个表格来表示这个过程 (为什么 E(fab3c65326caafb0cacb21c3f8c19f68)=\`e2xxxxxxxxxxxxxxxxxxxxxxxxxxxxx？这点大家不用关心，AES key 不一定，计算的结果也不一定，这里是假设刚好存在某个 key 使得这个结果成立)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wK57ib4HiasXyhCOAOo9OCZYPViaAHVKmUxXvq4l3Atzr8PGz7jjib6kerA/640?wx_fmt=png)

最后就是明文是`0102030405060708`经过八轮运算之后得到`e39855xxxxxxxxxxxxxxxxxxxxxxxxx`  

这里有个绕的点是，每一轮计算的值是 8 位，既 0x01,0x02。(每个 16 进制数 4 位)。因为是 AES\_CFB8。

而每轮 AES 运算的是 128 位 (既 16 字节), 因为这里是 AES128。

我们观察每轮`参与AES运算的上一轮密文`。

第一轮是`` `fab3c65326caafb0cacb21c3f8c19f68``。第二轮的时候是往后移八位，既减去`fa`得到`b3c65326caafb0cacb21c3f8c19f68`，再加上第一轮加密后的密码`e3`得到`b3c65326caafb0cacb21c3f8c19f68`。

这个时候我们考虑一种极端的情况。

当 IV 为 8 个字节的 0 的时候，既 IV=`000000000000000000000000000000`

那么新的运算就变成

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wvy0uWsZn0XkFiaU5aAT3LjzZ2ibRdy7CSwbiaURvb8k85xibd0w7b3N5ag/640?wx_fmt=png)

只要 key 固定，那么 E(X) 的值一定是固定的。大家可以看到`参与AES运算的上一轮密文`的值是不断减去最前面的 00，不断加入密文。  

那么是不是在 key 固定的情况下，只要我保证`参与AES运算的上一轮密文`是固定的，那么`E(参与AES运算的上一轮密文)`一定是固定的。

`参与AES运算的上一轮密文`每轮是怎么变化的。

`000000000000000000000000000000` -> `0000000000000000000000000000a4` -> `00000000000000000000000000a489`。

前面的 00 不断减少，后面不断加进密文。

那么我是不是只需要保证不断加进来的值是 00,`参与AES运算的上一轮密文`就一直是`000000000000000000000000000000`。也就是说现在只要保证每一轮`加密后的密文`是`00`, 那么整个表格就不会变化。最后得到的密文就是`000000000000000000000000000000`.。

要保证每一轮`加密后的密文`是`00`, 只需要每一轮的`明文内容`和`E(参与AES运算的上一轮密文)的前面8位`一样就行。(两个一样的的数异或为 0)

我们来看下这个表格。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83w6ibKKicib8a3xRc1HhGicEGGRe968lXJRm5aGVVBqKCc7qL3n4nsIMdicHA/640?wx_fmt=png)

这个地方我们不敢保证，但是前面八位的可能性有 2\*\*8=256(`00-FF`)，因为每一位都可能是 0 或者 1。那么也就是说我们运行一次，在不知道 key 的情况下，`E(000000000000000000000000000000)`的前面 8 位一定和明文一样的概率是 1/256, 我们可以通过不断的增加尝试次数，运行到 2000 次的时候，至少有一次命中的概率已经有 99.6% 了。(具体怎么算。文章后面会介绍)。由于在 key 固定的情况下，E(000000000000000000000000000000) 的值固定，所以`E(参与AES运算的上一轮密文)的前面8位`是固定的，而每一轮的`明文内容`和`E(参与AES运算的上一轮密文)的前面前面8位`一样。所以每一轮的明文内容就必须要一样。所以要求明文的格式就是`XYXYXYXYXYXYXY`这种格式。那么还剩下最后一个问题。假设我们可以控制明文，那么在不知道 key 的情况下，我们怎么保证`E(000000000000000000000000000000)`的前面 8 位一定和明文一样呢。  

所以我们最后下一个结论。

在 AES\_CFB8 算法中，如果 IV 为全零。只要我们能控制明文内容为 XYXYXYXY 这种格式 (X 和 Y 可以一样，既每个字节的值都是一样的)，那么一定存在一个 key, 使得 AES\_CFB8(XYXYXYXY)=00000000。

### **3、netlogon 认证协议绕过**

说完 IV 全为 0 导致的 AES\_CFB8 安全问题，我们来看看 netlogon 认证协议。

继续看图

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wFibSRNLvoBibNeaqaNBVicptI1BPkkLQOBVTt7dSMIPtTCzSYGXLH59pA/640?wx_fmt=png)

1、客户端调用 NetrServerReqChallenge 向服务端发送一个 ClientChallenge

2、服务端向客户端返回送一个 ServerChallenge

3、双方都利用 client 的 hash、ClientChallenge、ServerChallenge 计算一个 session\_key。

4、客户端利用 session\_key 和 ClientChallenge 计算一个 ClientCredential。并发送给服务端进行校验。

5、服务端也利用 session\_key 和 ClientChallenge 去计算一个 ClientCredential，如果值跟客户端发送过来的一致，就让客户端通过认证。

这里的计算 ClientChallenge 使用 ComputeNetlogonCredential 函数。

有两种算法，分别采用 DES\_ECB 和 AES\_CFB。可以通过协商 flag 来选择哪一种加密方式。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wDkmTI1hEnq3IJzicetVlgJibOaOMc2LmW1ZntOOoyAJdv7Irn0NfcesQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wzO2l0loumPtlHILEsE7hmChAqK8pATbolibvcT6DShBY71jLbIGyueA/640?wx_fmt=png)

这里存在问题的是 AES\_CFB8。为了方便理解，我们用一串 python 代码来表示这个加密过程。

```
indata = b'\\x00' \* (512-len(self.\_\_password)) + self.\_\_password + pack('<L', len(self.\_\_password))
request\['ClearNewPassword'\] = nrpc.ComputeNetlogonCredential(indata, self.sessionKey)
```

使用 AES\_CFB8，IV 是’\\x00’\*16，明文密码是 ClientChallenge，key 是 session\_key，计算后的密文是 ClientCredential。

这里 IV 是’\\x00’\*16，我们上面一节得出一个结论。在 AES\_CFB8 算法中，如果 IV 为全零。只要我们能控制明文内容为 XYXYXYXY 这种格式 (X 和 Y 可以一样，既每个字节的值都是一样的)，那么一定存在一个 key, 使得 AES\_CFB8(XYXYXYXY)=00000000。

这里 ClientChallenge 我们是可以控制的，那么一定就存在一个 key，使得 ClientCredential 为`00000000000000`

那么我们就可以。

1、向服务端发送一个 ClientChallenge`00000000000000`(只要满足 XYXYXYXY 这种格式就行)

2、循环向服务端发送 ClientCredential 为`00000000000000`，直达出现一个 session\_key, 使得服务端生成的 ClientCredential 也为`00000000000000`。

还有一个需要注意的环节。

认证的整个协议包里面，默认会增加签名校验。这个签名的值是由 session\_key 进行加密的。但是由于我们是通过让服务端生成的 ClientCredential 也为`00000000000000`来绕过前面的认证，没有 session\_key。所以这个签名我们是无法生成的。但是我们是可以取消设置对应的标志位来关闭这个选项的。

在 _NegotiateFlags_ 中。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83w7uCyBxngu2ymqvku3iaUsKX3wSQqayItQehWYItyVFaLY12kC6SXfiaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83w75aDe0anJ5rt3eL08mfmaJkmNlL5xgCW9TYnbkIEOXL2RaUHXESntQ/640?wx_fmt=png)

所以在 Poc 里面作者将 flag 位设置为 0x212fffff

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wxSPFmvcdNV27D08jQeFLlPFIbz5SG15ia3oCI4hAIgNqeWTKSyJjlfw/640?wx_fmt=png)

在 NetrServerAuthenticate 里面并没有提供传入`NegotiateFlags`的参数，因此这里我们使用 NetrServerAuthenticate3。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wpcX5nzsNaDIyl0dtE3ic1uBxAUxicOpqwibbTYjFH3PiaXNNslS8ec8noQ/640?wx_fmt=png)

### **4、重置密码利用分析**

前面的认证都通过之后，我们就可以利用改漏洞来重置密码，为啥一定是该漏洞，有没有其他的方法，后面会介绍。这里着重介绍重置密码的函数。

在绕过认证之后，我们就可以调用 RPC 函数了。作者调用的是 RPC 函数 NetrServerPasswordSet2。

```
def connect(self):
        if self.rpc\_con == None:
            print('Performing authentication attempts...')
            for attempt in range(0, self.MAX\_ATTEMPTS):
                self.rpc\_con = try\_zero\_authenticate(self.dc\_handle, self.dc\_ip, self.target\_computer)
                if self.rpc\_con == None:
                    print('=', end='', flush=True)
                else:
                    break
        return self.rpc\_con
```

调用这个函数需要注意两个地方。  

1)、一个是 Authenticator。

如果我们去看 NRPC 里面的函数，会发现很多函数都需要这个参数。这个参数也是一个校验。在前面的校验通过，建立通道之后，还会校验 Authenticator。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wG2MfAHib5x0QjibiaphuqK4aCWPwEiaaSicL3frIcQ1RogXc7cV1ialOGcgQ/640?wx_fmt=png)

我们去文档看看 Authenticator 怎么生成的

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wTJboZCsLCZITib0fibWAHHnEvowJHiciaHrAcO1fhhYR4ibrpPcXAP41DWQ/640?wx_fmt=png)

这里面我们不可控的参数是是使用 ComputeNetlogonCredential 计算 ClientStoreCredentail+TimeNow, 这这里的 ComputeNetlogonCredential 跟之前一样，之前我们指定了 AES\_CFB8，这里也就是 AES\_CFB8。而 ClientStoreCredentail 的值我们是可控的，TimeNow 的值我们也是可控的。我们只要控制其加起来的值跟我们之前指定的 ClientChallenge 一样 (session\_key 跟之前的一样，之前指定的是`00000000000000`)，就可以使得最后的 Authenticator 为`0000000000000000`，最后我们指定 Authenticator 为`0000000000000000`就可以绕过 Authenticator 的校验。

2)、另外一个是 ClearNewPassword

我们用一段代码来看看他是怎么计算的

```
def update\_authenticator(self):
        authenticator = nrpc.NETLOGON\_AUTHENTICATOR()
        # authenticator\['Credential'\] = nrpc.ComputeNetlogonCredential(self.clientStoredCredential, self.sessionKey)
        # authenticator\['Timestamp'\] = 10
        authenticator\['Credential'\] = b'\\x00' \* 8
        authenticator\['Timestamp'\] = 0
        return authenticator
```

也是使用之前的 ComputeNetlogonCredential 来计算的。密码结构包含 516 个字节，最后的 4 个字节指明了密码长度。前面的 512 字节是填充值加密码。这里的填充值是’\\x00’，事实上，这个是任意的。我们只要控制 indata 的值跟我们之前指定的 ClientChallenge 一样 (session\_key 跟之前的一样，其实也不是完全一样，最后的 ClearNewPassword 跟之前的 ClientCredential 长度不一样，所以 indata 也得是 (len(ClearNewPassword)/len(ClientChallenge))_ClientChallenge, 之前指定的 ClientChallenge 为__`00000000000000`，这里也就是 \`‘\\x00’\\_516\`)，就可以使得最后的 ClearNewPassword 全为 0。  

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC5BBRiccicQkicWD6n2u8JzHItRdz6v3aT1CtLryat1lBwGv3QiaOHgGXFqaNpCiaTvanXwKr8tbBrYvxw/640?wx_fmt=png)

0x02 常见的几个问题

### **1、 为何机器用户修改完密码之后会脱域**

dirkjanm 在 https://twitter.com/\_dirkjan/status/1306280553281449985 已经说的很清楚了。最主要的原因是 AD 里面存储的机器密码跟本机的 Lsass 里面存储的密码不一定导致的。这里简单翻译一下。

正常情况下，AD 运行正常。有一个 DC 和一个服务器。他们彼此信任是因为他们有一个共享的 Secret：机器帐户密码。他们可以使用它彼此通讯并建立加密通道。两台机器上的共享 Secret 是相同的。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83ww3FdeHBic6FPe5k7GCOaMZIHgicpNdedF07UIRxSj2xsiaNBKiaVM4lJtg/640?wx_fmt=png)

尝试登录服务器的用户可以通过带有服务票证的 Kerberos 进行登录。该服务票证由 DC 使用机器帐户密码加密。  

服务器具有相同的 Secret，可以解密票证并知道其合法性。用户获得访问权限。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wudWXwXDxOCzxm4ZmvmtI7XhMKM5BgiaalKSVh5B5uibWDDfgKVqKUcicA/640?wx_fmt=png)

借助 Zerologon 攻击，攻击者可以更改 AD 中计算机帐户的密码，从而在一侧更改 Secret。

现在，服务器无法再在域上登录。在大多数情况下，服务器仍将具有有效的 Kerberos 票证，因此某些登录仍将起作用。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wYeZfcVKNlr0pLk6BVUvBTWXtxPnGU8M0cu42iaIVhecpPlveN2eJARA/640?wx_fmt=png)

在漏洞利用之前发出的 Kerberos 票证仍然可以使用，但是新的票证将由 AD 使用新密钥（以蓝色显示）进行加密。服务器无法解密 (因为使用了 Lsass 里面的密码 hash 去进行解密，这个加密用的不一致) 这些文件并抛出错误。后续 Kerberos 登录也随即无效。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wBQ9dTg9UO6EU7hLkP8PTx6NORBicMx8b20o9AAhvPblyzogsR4Libf0Q/640?wx_fmt=jpeg)

NTLM 的登录也不行，因为使用 AD 帐户登录已通过安全通道（通过相同的 netlogon 协议 zerologon 滥用）在 DC 上进行了验证。

但是无法建立此通道，因为信任中断，并且服务器再次引发错误。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wMHHx6wA4cEVp6eiamyicXPeYGcjLjr3X8LiaZrWu4MEQkAnG53R9D2SvQ/640?wx_fmt=png)

但是，在最常见的特权升级中，将目标 DC 本身而不是另一台服务器作为目标。这很有趣，因为现在它们都在单个主机上运行。

但这并没有完全不同，因为 DC 也有多个存储凭据的位置。

像服务器一样，DC 拥有一个带有密码的机器帐户，该帐户以加密方式存储在注册表中。引导时将其加载到 lsass 中。如果我们使用 Zerologon 更改密码，则仅 AD 中的密码会更改，而不是注册表或 lsass 中的密码。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wQb6AnuNfRgrp46FRnKmaLzFPHYeQic7ozykH3A43yfjlsbc3N9pbUWA/640?wx_fmt=png)

利用后，每当发出新的 Kerberos 票证时，我们都会遇到与服务器相同的问题。DC 无法使用 lsass 中的机器帐户密码来解密服务票证，并且无法使用 Kerberos 中断身份验证。  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wzLGe9j1l9LiaQcbThcM8JF1gQfhtvET9UyRm1BnCk9CKvH8A1Ay1FVg/640?wx_fmt=png)

对于 NTLM，则有所不同。在 DC 上，似乎没有使用计算机帐户，但是通过另一种方式（我尚未调查过）验证了 NTLM 登录，该方式仍然有效。  

这使您可以使用 DC 计算机帐户的空 NT 哈希值进行 DCSync。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wZUicPJmcWqPTLsgzUNYma9GVBt5vHunsAErSlH0SNrK2Y53BtOjCwjA/640?wx_fmt=png)

如果您真的想使用 Kerberos，我想（未经测试）它可以与 2 个 DC 一起使用。DC 之间的同步可能会保持一段时间，因为 Kerberos 票证仍然有效。  

因此，一旦将 DC1 的新密码同步到 DC2，就可以使用 DC1 的帐户与 DC1 同步。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wAD7SEDWzVicGiajdezxyzsr8pHibpTL9tH1xfibhm02smuS3HibFtEXBjlw/640?wx_fmt=png)

之所以起作用，是因为 DC2 的票证已使用 DC2 机器帐户的 kerberos 密钥进行了加密，而密钥没有更改。  

### **2、 脚本里面 2000 次失败的概率是 0.04 是怎么算的**

在作者的利用脚本里面，我们注意到这个细节。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83w4MnczY2HMbfC6pYAFHyGIEtVTsoDXdEyjGQ2E6ZEANrqUL8xZopicPQ/640?wx_fmt=png)

作者说平均 256 次能成功，最大的尝试次数是 2000 次，失败的概率是 0.04。那么这个是怎么算出来的呢。

一个基本的概率问题。每一次成功的概率都是 1/256，而且每一次之间互不干扰。那么运行 N 次，至少一次成功的概率就是`1-(255/256)**N`

那么运行 256 次成功的概率就是

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83woKAZOYZSROoJGtBbnMdFOQkazS2N77Dqxxta6RWCamwFhhOtHtCm6Q/640?wx_fmt=png)

运行 2000 次成功的概率就是

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wkYic5t1scph4z2OIzcgkUjMBrLYowNViafCPTSFogNUS4u0XBQJXK0Ng/640?wx_fmt=png)

### **3、NRPC 那么多函数，是不是一定得重置密码**

已经绕过了 netlogon 的权限校验，那么 netlogon 里面的 RPC 函数那么多，除了重置密码，有没有其他更优雅的函数可以用来利用呢。

我们可以在 API 文档里面开始寻觅。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wqwIBLkRzjqEtBve5QLuqerwwTt8nEaE3gwsLmTdIAjibuodTUUfRicTw/640?wx_fmt=png)

事实上，在 impacket 里面的`impacket/tests/SMB_RPC/test_nrpc.py`里面已经基本实现了调用的代码，我们只要小做修改，就可以调用现有的代码来做测试。

我们将认证部分，从账号密码登录替换为我们的 Poc

```
def connect(self):
        if self.rpc\_con == None:
            print('Performing authentication attempts...')
            for attempt in range(0, self.MAX\_ATTEMPTS):
                self.rpc\_con = try\_zero\_authenticate(self.dc\_handle, self.dc\_ip, self.target\_computer)
                if self.rpc\_con == None:
                    print('=', end='', flush=True)
                else:
                    break
        return self.rpc\_con
```

将 Authenticator 的实现部分也替换下就行。

```
def update\_authenticator(self):
        authenticator = nrpc.NETLOGON\_AUTHENTICATOR()
        # authenticator\['Credential'\] = nrpc.ComputeNetlogonCredential(self.clientStoredCredential, self.sessionKey)
        # authenticator\['Timestamp'\] = 10
        authenticator\['Credential'\] = b'\\x00' \* 8
        authenticator\['Timestamp'\] = 0
        return authenticator
```

然后其他地方根据报错稍微修改就可以了。我们开始一个个做测试。  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83ww1WyRewG9ZJKSQlXHGPxYS063rXkKsvFxz6YEX5KCtuCTuSDEUtiatw/640?wx_fmt=png)

这块基本是查看信息。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wtFrhCobCVPLLbDuXg837o8l87GVcl0C1JJd2arP8IZueBxwiaV0Uoxg/640?wx_fmt=png)

虽然可以调用成功，但是对我们的利用帮助不大。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83w94hvznHPpwOXu8EQSEdHM4pHBmaFFnDwDDYNswVPibc366cgho4jQiaw/640?wx_fmt=png)

这块基本是是建立安全通道的，设置密码已经使用了，除去认证，设置密码，还有一个查看密码。遗憾的是 EncryptedNtOwfPassword 是使用 sesson\_key 参与加密的，我们不知道 sesson\_key，也就无法解密。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5otyMkgzRw69swH7a5S83wVSgdI5ZHmxOtrdKIEkWhgnrlzysWRNIBTibaI6hC6Bzpciby0Z4J3FEg/640?wx_fmt=png)

其他的函数整体试了下，也没有找到几个比较方便直接提升到域管权限的。大家可以自行寻觅。

事实上，dirkjanm 也研究了一种无需重置密码，借助打印机漏洞 relay 来利用改漏洞的方法，但是由于 Rlay 在实战中的不方便性，整体来说并不比重置密码好用，这里不详细展开，大家可以自行查看文章 A different way of abusing Zerologon (CVE-2020-1472)。

### **4、是不是只有 IV 全为零才是危险的**

在之前的分析中，只要`参与AES运算的上一轮密文`每一轮保存不变就行，第一轮的`参与AES运算的上一轮密文`就是 IV。也就是说，存在一个 IV，只要他能够保持最前面 8 位不断移到最后，如 AA(XXXXXX) -> (XXXXXX)AA，值保持不变，就一定存在一个 key, 使得 AES\_CFB8(XYXYXYXY)=`IV*(len(明文)/len(IV))`(这里乘以`(len(明文)/len(IV)`是因为密文长度跟明文一样，不一定跟 IV 一样)。显然 IV 全为零满足这个条件，但是不止是 IV 全为零才有这个安全问题。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

\- End -

精彩推荐

[RTL-SDR 接收 NOAA 气象卫星](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649732860&idx=1&sn=175c6b23ea8af32edf73f4f1bfe85ea6&chksm=888c8693bffb0f8578422cb767a92803b57ce9bce59412dd7f39ab68b785f9c7509258a13fc6&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[ShadowMove：隐蔽的横向移动策略](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649732789&idx=1&sn=16cde6178e78a4b92c623f449b29efc8&chksm=888c86dabffb0fccf64fa8161063a1caa006963531c00cc407d51aaf3a4001025996b40c33f6&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[杀毒软件 McAfee 创始人 McAfee 在西班牙被捕](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649732709&idx=1&sn=043cb6e148bcaed2dc39e3a4027d4b71&chksm=888c860abffb0f1cc54f6cdcf4bce080331221f5c96c8c2baf4d79d244bfc1a1a0ec1ddcf80c&scene=21#wechat_redirect)

[TISC 2020 CTF 题目分析及 writeups](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649732701&idx=1&sn=e922523d09e49b413de3eb2c931e699b&chksm=888c8632bffb0f24e2fb7d6882ae14ba58543ff2c2d23e8c93736e2d6a4778bba06cee2be046&scene=21#wechat_redirect)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)

**戳 “阅读原文” 查看更多内容**