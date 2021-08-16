> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/k9OtJ-GZaNOGOsxL-EIBww)

01

靶机介绍

这次的靶机是 Try Hack Me:Relevant, 房间链接 https://tryhackme.com/room/relevant。难度其实还行不算太难可以尝试玩玩。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGziaicM9ArBmqqia2tlahrRfDUh23VYu4hLyIsCdGKjTlI0P62SbNFz4Fg/640?wx_fmt=png)

02

信息收集

前期还是得用 python autorecon 进行信息收集。这里我已经扫完了，先看看 nmap 报告, 这次应该是个 windows 靶机, 开的端口比较多。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGjpfPL8Vd5jicMSIFHhyYBkM35ffxaSD4TvvSsibdmbOOhTVf5tiaaLFPw/640?wx_fmt=png)

同时看了看 smbmap 的扫描报告发现可以空密码登录 SMB 端口同时有个目录是 nt4wrksv 下有个 passwords.txt  

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGNiafpKYRjrX4kTpiaRvQsFCly2L2ul6G3YCpeOLNlqbs4Deu89UsMFkg/640?wx_fmt=png)

03

SMB 端口文件获取

首先使用 smbclient //ip/ 目录 连接，密码为空。登录成功以后用 get 把文件下载到本地。使用 cat 可以看到有两个加密信息。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGkSHCicyp6DCD61pv8FQVhGmB0JKickDqhoeA48YoFXOLXoeY40tJdRfQ/640?wx_fmt=png)

用在线解密可知其实都是 base64 加密都可解密出来。然而我试了一下 xfreerdp 连不上。。。。。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGJic6j4hr28avKibJTBdI2B74dbmWjd3g3486pTM1j0ZD09cNo5fRuV9A/640?wx_fmt=png)

04

挖掘其它端口

这里看了一下 autorecon 还有一个 49663 端口的报告, 瞬间感到奇怪。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGgQyyB2RMRVV9euj69n4EIoDTmsibhJvuIQj0fmsUFlicLlXzMPdomCXQ/640?wx_fmt=png)

后来试了一下原来是通 SMB 端口 nt4wrksv 目录下的, 同时前台可以访问。既然我们可以空密码登录并控制 nt4wrksv 下的文件那么可以考虑文件上传漏洞。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGFeSOygibicv4mKq0oUZpnnEfQB3QA82kNc07X9AMibyj1HBibIYD455Fug/640?wx_fmt=png)

05

文件上传

文件上传首先需要制作一个控制 shell。可以用一句话木马但试了一下后面提权有些命令执行不了就有点小尴尬了。还是建议用回 msfvenom 方便点。后面尝试了一波发现是可执行.aspx 文件的。

如果不会生成的话可以看看这位大佬的文章把 -f exe 变成 aspx 就好了。文章 http://blog.chinaunix.net/uid-31140357-id-5835076.html

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGib7yIGrfyVpmS2AR4zaeNX2smJNib6STgnglE9NAGKqpia7I9JFQicZ6Eg/640?wx_fmt=png)

这里先重回 SMB 端口用 put 把文件上传。  

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMG1Yw3OZ1AQIEMgvXowXwXg5lWrq34botLjUXSZWLk5awIlknyOFCwLQ/640?wx_fmt=png)

然后启动 MSF 填入以下参数。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGMGz2NYicJh2QVmA5CZMxVfeHmlX5nicK6OYSh97DZvEAN5Sse3R0onUw/640?wx_fmt=png)

前台访问 http://10.10.95.60:49663/nt4wrksv/payload_test.aspx , 出现 meterpreter 代表成功。至于 flag 很容易找就不展示了。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGAAoZeVA4HibY09emPLaicwGsvKOVh4pdzzKUtQqbSylqCG9icXx2UU8hw/640?wx_fmt=png)

06

提权

这次的提权比较简单由于 SeImpersonatePrivilege 滥用所导致的。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGOuDt2COPRBFvUK2D66Y70lW5A5glg1wpLRDaTuNVS2e55iaVibCia6cRw/640?wx_fmt=png)

使用方法很简单可以看看暗月的这篇文章 https://www.moonsec.com/archives/2888 总的来讲只需要一个 PrintSpoofer 就能一件提权了。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMG1EBh7HXE6YEvX3vEtujNpTOGJicibJUevgFsxD70O6h4yXMRzk1JoRhQ/640?wx_fmt=png)

这里指路已经编译好的 printspoofer Github 地址 https://github.com/dievus/printspoofer 下载就好。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGMlE6G19GoyXWlS6MW1d7C3lZ2lTte2RcNGwRLcXRNBLOngCoMVcB9A/640?wx_fmt=png)

这里按住 ctrl + z 退回 meterpreter。上传这个 PrintSpoofer.exe

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMGPoeteznrf5LltKtp3DwibclUc2hTbxcxXubjiaicxibdxwqUUPCEibbQXiag/640?wx_fmt=png)

然后就一步提权成功。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9TBLbIia0FRxeT9zP3T96uMG6z3fkdI8jRPibiaGfmeVBAGcdMngCoTZkicpDxvllxrYL0GyF3g3AUshg/640?wx_fmt=png)