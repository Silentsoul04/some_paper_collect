> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/K3d_gnwDOAtlf9JO_Wrz6w)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **89** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

靶机地址：https://www.hackthebox.eu/home/machines/profile/210

靶机难度：中级（4.3/10）

靶机发布日期：2020 年 2 月 7 日

靶机描述：

JSON is a medium difficulty Windows machine running an IIS server with an ASP.NET application.

The application is found to be vulnerable to .NET deserialization, which is exploited using

ysoserial.net. A custom .NET program is found to be installed, which on reverse engineering

reveals encrypted credentials for an administrator. These credentials can be decrypted and used

to gain access to the FTP folder

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KYp5y7HOezU3rxzB1RGPeAuNVZQEOiavZ9VMSFGoXycicD0LFSNxdbVLQ/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.158....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KSoL1z2VM4JdvI0rKMF1DSxxvV7J52oibExEtzibFzhxEPwUvZic8AV8Dw/640?wx_fmt=png)

Nmap 扫描发现运行了 FTP 和 IIS 服务器... 操作系统版本为 Windows 2008 R2 或 2012，WinRM 也已在端口上打开

5985，可能有助于以后的横向移动渗透...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KNxEiaWtKKoEof2chgQSUlzIq0eBBwexljKxqTZLhicRnsw3ldOSqQ4Bw/640?wx_fmt=png)

访问 80 是个登录页面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K4n5iboV7zFky9Qq8fHdUEd2hkdRCuWtvbgKlyKd3kn1akmucxbvhQiaQ/640?wx_fmt=png)

使用 admin 默认口令成功进来了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K8fUr5UE82ib10lfzY26VfDVNHSliaIzO7cyvz4TWiaKOEbLNv8XCRUhMw/640?wx_fmt=png)

进来后点击模块有的是返回了登录界面，有的是 404 报错... 我进行了 bursuit 分析...

可以看到请求包含一个带有 base64 编码值的 Bearer 标头...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KauEknBTZCv1MSWRvSaviaAueNwRzznXZwBGhwSUu5XcPma7XAnzF35Q/640?wx_fmt=png)

对 base64 进行解码... 这是 administrator 用户的登录信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Kq1fIF0XliaQZq34peL5IPfJIO0R4ulvib6fvDeqdEc29U6AOm2kuYPHQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KNgZqicPib0fhh9oVXmJoLpyz36TM7qeq3hAPDQJIrQ7ia7oj7lpzXAuEw/640?wx_fmt=png)

经过发现，admin 的 md5sum 值就此哈希值... 修改 Bearer 为 dayuxiyou 后，发现好像存在反序列化漏洞？？继续测试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KtteYEjQVFf2KTEVcJz77CUicdF0ZWDdmjng3RpIV3X2QHQLOsUJL2PA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KricDF5tEo29j2iazEPAP2QBgoauF5rB7KV0icgn4QfuwLZ77XneObdBnQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

服务器返回 500 内部服务器错误，指出 JSON.Net 对象反序列化漏洞... 该 API 是用 ASP.NET 编写的，并且

服务器反序列化接收到的 JSON 对象....google 搜索 json.net 相关的反序列化漏洞看看... 利用

```
[ysoserial](https://github.com/pwntester/ysoserial.net)
```

它可以生成. net 反序列化有效负载...  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KsRbVn1MJaNicm1jCwKnm7TLYdwD7Cc7ibLiaZNP0FcWT7NGdEQ2RxUXYA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K5bZDKlPRnGRE1dCNZ1HYv0IYr1c7BHhuW9XkiacaibGG09VsfhRZsYOQ/640?wx_fmt=png)

下载即可... 放到本地使用 windows 的 powershell 进行编码即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KDfw9307Lj3Am0veia9TvECaXgfbT4vzHXdIHw2bkobAhpp9PpPbU2WA/640?wx_fmt=png)

```
.\ysoserial.exe -f Json.Net -g ObjectDataProvider -o base64 -c "ping -n 2 10.10.14.51"
```

利用 ysoserial 对简单 ping 进行了. net 的输出... 利用试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Kbw51hPUGWfAicj1I3ZHqKpcTpNRSnjjcWiaJa8WAxRWRDmxa2OYapp0A/640?wx_fmt=png)

虽然注入后回的是 500 错误，但是不影响反序列化的利用... 本地监听产生了数据包...

这里就很多方法可以反弹 shell 了，开始把...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KAqibA8jmOES4ibNSvQEODEnZ7aXuazLdBKx7OYWoYk2fL71xjg2eX8tw/640?wx_fmt=png)

```
.\ysoserial.exe -f Json.Net -g ObjectDataProvider -o base64 -c ""
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Kria9sOZweUr0QKg35jXVBz80Aw0LpOhqUf7lCib3cYOLRqzOV5KA0BNQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KRUpFmYGAH2z41uibEVnjycFibmYd2z8gDwXEdkzFk6puL2OPb0yia5Xjg/640?wx_fmt=png)

可以看到通过 powershell 成功上传了 nishang 的 tcp.ps1，获得了反弹 shell...

或者使用 smb 共享也可以..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KnceBfIDpaqtC05KM37HOLco3iahoIkvAjZbMLsztjZM9yeu7iaZe9jibg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KUOT2q5s5DrPnNmh9e0oDywicdjuf75s3NC2GViaxeKHS7V6c3z6dLicKA/640?wx_fmt=png)

获得了 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/vekKnDibcricu2UWZUgqzbqic9EBkejl6uTaAp9pZqTSiaibKPpbJamzHXyE2iapH87vjcQHV7hz25QFcBibaMpyadLqg/640?wx_fmt=png)

三、提权 root

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4yo5kHOX9ibN8ibibPy7W2Hr5gIiaWyWEuIGKPDgfhHf0oA2dpjKy7LLyBHicoTtfRED9OyIK92hpd9GhGqx3iaLln2A/640?wx_fmt=png)

  

  

  

方法 1：

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KKZaj2TCvkh4Y02NRaIcwFODric2eEYD8RKWxQJICewmtUljYEAxn0fw/640?wx_fmt=png)  

SeImpersonatePrivilege：

可以利用 NTLM 中继到本地协商获得系统用户的令牌，可以使用开源工具 Lovely-Potato... 通过 WinAPI CreateProcessWithToken 创建新进程，引入系统用户的令牌具有 SeImpersonatePrivilege 权限，最后该令牌具有 system 权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KicS3uZVrfEFKiagY6ecvnGZmaSBIypXeqhUugRD6GNiaRBZ0ibibc8YNRibQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KIqLBtQpJs7g8S7vr8Od5hr8T5ojbu7XMTozzW1qML7MApIQIWAKRrg/640?wx_fmt=png)

CLSID：

```
https://github.com/ohpe/juicy-potato/tree/master/CLSID/Windows_Server_2012_Datacenter
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K81BwCUeI2tj4o3tCb0ncVb0vIHrLdSiaicqfSmj2cDcYYhgjE8yrRf8Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KGdvIZUkEo1knsrYJNXZepxuS2uJxVSlAKtia7qR3Xp4pZly5diazu7eQ/640?wx_fmt=png)

```
dayu.exe -t * -p dayu.bat -l 1337 -c {e60687f7-01a1-40aa-86ac-db1cbf673334}
```

这是个简单的方法，对于前面的靶机我写过一样的方式方法... 这就不多解释了... 直接 GO  

利用 nc 和 CLSID 进行成功提权... 获得了 root 信息...

  

  

  

方法 2：

  

  

  

和 NO.80 中方法类似，利用 Chisel 进行本地交互...  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KUuQCMmibtzQ43I8no274dg5CgDb58e0dqjsy8LEB4BXsicaBQfOZj1uQ/640?wx_fmt=png)

检查 Json 上的侦听端口，可看到有端口在 localhost 上运行着... 继续查找看看什么程序...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KcHwibwnyoz3UeBRrPVFYAs4OH49bibDXtu51pEC5XH876FVVtHVdibP7w/640?wx_fmt=png)

使用 tasklist pid 660 来查看，这是 FileZilla Server.exe... 管理界面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KkTLVoMMNLG6sOMIkSKa6bJIlV8YR9vGbGUP9s9D0TtfeLPtEn4BWUQ/640?wx_fmt=png)

通过查看该配置文件存在 passwd 哈希值... 这里可以突破到管理员权限的界面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Kpic6BINfPuTCBRWTiaGugAASDHKKedQehiafcvHh2LedR3qnC1j36segg/640?wx_fmt=png)使用 Chisel 在 localhost 上运行的服务交互，我已经坐在 Windows exe 的副本中 \ share，因此可以将其复制到 Json，然后进行连接

```
https://filezilla-project.org/download.php?type=server
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K6sB3Hn5IkPSISqYM4LIeETc8T3HIL46sQ9UFeAyuOqibkqnAkpZVyNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KA23KPicBpgGZSn0TueKxBqTia9iaa454j4nU4u4fFRYXgzBGAf9T3O4gg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Km2ggLtlF7l414MTH8vN5QLPCbrh520yglOSG11xcrc4icFhOPYWQhZA/640?wx_fmt=png)

这里用 windows10 的 openvpn 一直无法连接上 HTB... 就没进行下去了...

到了这一步很简单了，直接点击 connect 应该直接通过隧道登录进去... 根据前面. XML 的提示，会直接可以登录 FTP 服务器... 然后查看到 root 信息...

  

  

  

方法 3：

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KGgTQYNLg0WLkkV1bcOlDmFfGiay8ZGRCnwYFicMsPkxI8FPT3DwyzckQ/640?wx_fmt=png)  

在 userpool 的帐户枚举期间，我注意到 Program Files 处有一个可疑的服务 FilesToSync，以及一对加密的凭证...

该服务似乎通过 FTP 在两个位置之间同步文件...

这里的 user 和 passwd 的哈希值无法破解...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KBG9NPMrOeoCBiaVqRbV4SpM26RTWS6Wgxqmq8icmkcNibeZZ1GR2jhE1A/640?wx_fmt=png)

copy 了 SyncLocation.exe 到 windows 机器上以进行进一步分析...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KRgG0jllMmZ4xZg2j4tASZafttVZ53C2ZB5TQ0ZZG4UzeibYf7gXKoiaw/640?wx_fmt=png)

事实证明，这 SyncLocation.exe 是一个. Net 程序集可执行文件，使用 dnSpy 轻松将其反编译为其源代码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KR662moHCu0IT1Q8acPVboB24sbEdLgpsoWaG3ibYLuE6CicbicNS2WkIw/640?wx_fmt=png)

可以忽略该提示...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KhLcnIMWryiaRK8BDGwEsELgxD7ib25oF5ibj4EVSicCMIAx0341BRG9SQQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KnelibVAcnFicTKWNlckNTregtCnPRT6bk19GJecyhPH73pCZZJHGtT2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KIGspt8xIxj5iceOaiaV9IAyp8jboHoEAkzVeHKkTNP8VPGFrvZpia1R3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KbKefHFOiaqibRd55fhZnGP17dYWTwmugbUeBpmbSDseU2BcRGJOJZ9zg/640?wx_fmt=png)

经过修改源代码.. 初步测试了，可通过运行输出需要的文本信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KGibbt4SN2eW0ZrAmGMvlKmzI5rm80KmYB1iaDMOumS3bJ5VKuczS0pGA/640?wx_fmt=png)

通过 void 源码修改方框输出值，显示 user 和 passwd...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KJSdvVow3mprYKuRn3KDH6yatUZbAulYs3N4SIWtriaSoRG1bRUxyw5A/640?wx_fmt=png)

这里下面报错了两个，修改下空格即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K6rbicLLhk7FEef57kHlhS6YdQV0rAA8pLkPxdTicMzhK8jHjlKhic5eWQ/640?wx_fmt=png)

或者结合 SyncLocation.exe.config 文件的内容以及哈希值... 通过

```
[dotnetfiddle](https://dotnetfiddle.net/)
```

运行即可获得用户密码...  

```
获得用户密码：superadmin:funnyhtb
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KJyMiaMs2d70nA1uiacgTBRJibR08dwWORMGFT8JELqSHBURfCA9a2Tc3g/640?wx_fmt=png)

成功通过获得的账号密码登录 FTP 服务器，获取 root 信息...

  

SQL 注入 - base64 的各项转换 - 利用 ysoserial 工具注入 shell - 利用 SMB 共享上传和下载提权文件 - 隧道的搭建 - dnSpy 源码的编写 - FileZilla Server 服务的使用等等...

又学到了挺多技术和知识，虽然不是很深入的了解某一款工具，但是基础的使用已经学会了... 加油！！

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