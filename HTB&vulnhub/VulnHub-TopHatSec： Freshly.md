> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OF2cXc8fQ1NmHGzM1WoKbw)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **47** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.vulnhub.com/entry/tophatsec-freshly,118/

靶机难度：初级（CTF）

靶机发布日期：2015 年 2 月 18 日

靶机描述：

这项挑战的目标是通过网络闯入机器并找到隐藏在敏感文件中的秘密。如果可以找到秘密，请给我发送电子邮件进行验证。:)

您可以使用几种不同的方法。祝好运！

目标：得到 root 权限

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ZWuiawgD1ibGg7clricuxxJWvd3BX51uOT1iacjaPxvp07icnPric574yJwtDZmdcO26iahfEEp4JiaNBS8UuTKmHSryUw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_svg/kiaXicXJs2M4e1LWaSZxYGicECrUwR1s7o8UiblW7CyI0t5KzkBruwlYfgG40XPhDhlBibFriasnSQia9NxU7KZWhPDWZeWJcJr2CH5/640?wx_fmt=svg)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibic12u5slaUkPUfgOYax0LS9M0a0Gfj7iczj6cj4k0fCCVaibZ6dliccFwQ/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibibMtaLIYXjlgqbkicra9992AGFYmPOibb5ib1VqY5SV7Xm7HO1ic0W4Da4A/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：

```
192.168.56.137
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibY3UJVnkY4PfScrte0QO6oibNgMF5qAqIC0jmziaWia47eTIu5xZZibqLpg/640?wx_fmt=png)

nmap 扫出 80、443、8080 端口开放着...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibw2rLaLyIdnGFibwpibzSRbC2YxiczKzBOWNZqPO2EbNqicEodQY3ialVdMA/640?wx_fmt=png)

直接 nikto 漏扫...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibex9Zpa2WVLUvicibdEOiayEKv9CticWicQiaibGQJLz64w5vicIIU5P3ftGOYw/640?wx_fmt=png)

发现了 / login.php 和 / phpmyadmin / 目录... 访问...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ib9l6ia0cXiakkLoulxpESxfOT1HgyHCH2RAJnv4SFYic1ibVHnzHRBclbiag/640?wx_fmt=png)

使用 admin 默认账号密码失败...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibtiavIEq3Q1ByodWQ0Z9EYDQs7tibiad4tVdfia2cwZOEwMGoAAgiawPl18A/640?wx_fmt=png)

我尝试 sql 注入输入 `' or 1=1 -- -` 发现输出 1，说明成功登陆了... 存在 SQL 注入漏洞...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibpibPwfoQq1TWeicte1R9vdpzXHvdtteUfmhWIBWa217zalKX8icDiczVLA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibBTzvb9kccEdz62yanPmpqtFGFE974rUJBOibJcORhaZ8JtUYqzfTaLw/640?wx_fmt=png)

```
sqlmap -u "http://192.168.56.137/login.php" --form --level 3 --dbs
输入user=1*&password=1&s=Submit
```

发现 WordPress8080 库...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibXpM49bmMdZ86rcpY4FPlyykomOz5oP9ZibjPK9NMQwlvDuL68KPkb8w/640?wx_fmt=png)

```
sqlmap --url "http://192.168.56.137/login.php" --data "user=1*&password=1&s=Submit" --smart --batch --dump -T users -D wordpress8080
```

找到 wordpress 的用户名和密码：SuperSecretPassword

登陆 http://192.168.56.137/login.php 没成功...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibtWLfg1W0MbcLLkjFBvmQJsqxgtaQpcZrpNqUbccSicm2qUbr40Uosxw/640?wx_fmt=png)

前面 nmap 发现还有个 8080 开放着... 是 wordpress 框架...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibibdKkKiakSCsAdCVKGcra4FhkiaXIU9Py4cUXo0KjEVJkImmTPictPWvOQ/640?wx_fmt=png)

都知道 wordpress 都存在 wp-login.php 目录登陆页面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibRBkShjBPYCibRujUxyicDtluQwaJ2K1jAatEXUC1JhicrSOZDNoWZNicew/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_svg/kiaXicXJs2M4e1LWaSZxYGicECrUwR1s7o8UiblW7CyI0t5KzkBruwlYfgG40XPhDhlBibFriasnSQia9NxU7KZWhPDWZeWJcJr2CH5/640?wx_fmt=svg)

二、提权

使用 admin/SuperSecretPassword 成功登陆...

到这里就很简单了，直接上传 shell 或者改模板即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ib4iaQaKuAGIX2VwzicEMZq7BsqQwWaKXzfcU0AnUVUagDmicCUd3v94R5A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibaXUVGBnym5eCb5SjVnqqjlESc7b4CUnbCVy1Uf9YhMbDqLjPCusV9w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibxIGBvWhF7jy9th2q8hQufyfvydGkwDcNLqg16TLX4koTdBKq7c1dUQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibyAWiaPO53ju31sJwKL7xuriaNSKPHSBNyOZZCy5iaprtgccFQ2pLNMicCQ/640?wx_fmt=png)

这里我就不多介绍了，前面至少讲了 10 多次的文件上传文章....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ib86ktnoBhU96830KfWJRpM2oib7ibjtY4HCGpxiaQ97uMPH4Q5vmKYSiaUA/640?wx_fmt=png)

直接 su 提权即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMhvUmJ5U8KGaNALTCp9M9ibGaa2DjdiabebOUKTFiap8Vibrxa9uh3sMH3A3V94NwQTUmMvE2ZrMgQicA/640?wx_fmt=png)

成功提权...

这里肯定还有很多种方法提权... 这里还有几个用户没参与进来... 还有对 8080 端口进行 nikto 后，还存在 http://192.168.56.137:8080/img / 目录，下面很多文件... 还有 80 页面的 GIF 图也可以分析...mysql 数据库底层也没查看... 主要今天太累了，连续肝了三台靶机... 休息了...

由于我们已经成功得到 root 权限，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/ZWuiawgD1ibGg7clricuxxJWvd3BX51uOT1iacjaPxvp07icnPric574yJwtDZmdcO26iahfEEp4JiaNBS8UuTKmHSryUw/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)