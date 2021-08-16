> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/t89BxUq1LOimhKZqjJ7DZw)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **102** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/Zo04aoPGhhftaGG0yuEeaxw97HRiaFa8WJW7libBkFeicrPny8KnvKmeezoNnqicGdpWHkOm3eGAIXwGohqRuZ6S6Q/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/26

靶机难度：中级（4.6/10）

靶机发布日期：2017 年 10 月 10 日

靶机描述：

Bank is a relatively simple machine, however proper web enumeration is key to finding the necessary data for entry. There also exists an unintended entry method, which many users find before the correct data is located.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/py7L4cx8wYvBYkElUsqDz94g2u3uiaKibfK2IkLjMkEBKezINP2n0PyX4GwcXC1vl0K8KWnITP6HhjIuhyUBIXbA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XrBsia6eKtTFtr4vwm8FVt5frF8ojc6Xtp0ChSOwic1tRYkxthCoB1v1SekZZzcuvLGhDnRCDt8IVxpHV9flfc9A/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PXWEicxy2xC9I5LF9rFMqYCphbH3cCW3vuSv6ic9WhOhzpoJibNkQLKy2DAWiazzOcIg1RLfgZiauLaLG8ucgHOJGdw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9HeuxUQxAn8dGbymPNf8oW13XffTxp0CjQhNxibrpvvLq5RskrNrVeIA/640?wx_fmt=png)  

可以看到靶机的 IP 是 10.10.10.29....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9yN2iborbiaI8uiaM1lLdOYcibzsvW7ibAZX7RGddrHzGdz0LpvhuIQS1w1A/640?wx_fmt=png)

Nmap 发现开放了 OpenSSH，DNS 服务和 Apache 服务...

Apache 服务正在运行默认网页，并且无法从 DNS 服务器获取任何信息，在这种情况下，Apache 使用虚拟主机路由流量.. 将 bank.htb 加入 hosts 域名中即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9ibeibLl09ZYjXCH6ia26RP9VWwbjiaaA1chicOU5XwhVW4Oxpf8q732eO2Q/640?wx_fmt=png)

加入后访问域名重定向到了用户名密码登录界面... 这里开始分析 sql 注入... 只要遇到这种账号密码的界面，我相信都存在注入...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9w9lhW5UbaUWWdSkldVOEVlGl1ZE5taPp6d9gJZibQlH3ZD2gBL30g7A/640?wx_fmt=png)

可以看到首先我利用 BP 拦截了 10.10.10.29 页面信息... 是 200OK 返回，提示 apche2 界面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9wfQYu1IRU02Y47wSnFibM9fYBELWa3ibrDnFxYkFbnyeNUbwicd84Txzw/640?wx_fmt=png)

但当我在拦截修改 hosts 为 bank.htb 后，发现反馈的前段代码尽然是 302 Found... 正常返回应该是 200 ok 到登录页面...

但是返回 302 Found 说明存在跳转页面... 跳转到了 302 错误的页面上... 爆破看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9xgEavdsiaxAQu4ZTNkPb7BA6NtfibRENuiaz8gMch9L8csXpjkNtSxWGw/640?wx_fmt=png)

```
https://github.com/maurosoria/dirsearch/blob/master/dirsearch.py
```

这里利用 dirsearch.py 爆破了目录，获得了两个有效的目录信息....index 和 support....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9EdLvOF2fW7N5m5dG18JIOAIXtq8bmNicS7yFw7TFksaokAyLJwMVCZg/640?wx_fmt=png)

通过访问 index.php，他立马重定向到了 bank.htb...

利用 BP 拦截了 index 访问的页面... 发现是 302Found，我听过修改 302 成 200 OK 后...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9zZBVaOqTaGIMTOKtPYwicZydS33A3KP3M7sWiald7JMjDUVgCEUuwtWQ/640?wx_fmt=png)

重定向到了此页面，这是一个银行账目的页面数据存放....

这里知道了，这是一个需要将 302 修改 200 http 传输的定向页面...BP 有功能停止 head 头部传输... 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9a3zdYBX8Wbam7icWAAqW19qcYGIvPhnABAAARUiafb4a24iakPs45m9Zw/640?wx_fmt=png)

当尝试打开 / index.php 一个 302 Found 重定向时，我们可以使用 burp 停止重定向，这里直接将 302 Found 以 200 Ok 对于开放 Proxy -> Options -> Match and Replace 改变即可...

然后刷新页面，可直接访问到了 index 页面... 说明成功停止了重定向操作...

继续访问 support.php 此页面可以进行文件上传.... 但是经过测试，此页面只能上传 htb 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9FAb64nguSvoRCdQlicibEmLicBCelcVQqTiaCWhcWL51qbsGKJic5KoYhqg/640?wx_fmt=png)

简单一句话命令... 然后文件名命令. htb 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9fS58cmR7I9XEZyyIichqcLcSp5P2BEiccTcVibZWcDfmwNndsGUDUicryA/640?wx_fmt=png)

上传了文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9icSeN0FGA1MWar9wQ2M0CPfRAZv2nB61Pk168P6mcIlUGXqAJhDZuQA/640?wx_fmt=png)

简单测试，成功有效的 shell... 发现这是 www 权限... 这就简单了... 直接利用反弹个 shell 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9pEEl8bDevgoIRibDUNU4phQiacJytqq8WFsAVZyGaWYSic7X604q6796w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9LECXUg5FV7xGf6wEiaGLVGnyfxMYlqFGoiavP226kRMFDEpaWXuMj9Ng/640?wx_fmt=png)

通过 nc 反弹 shell 后，获得了 user.txt 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9qIonLN1NeD061IrsrwWHyXP30nNfIFcb8HHQxVd8XPRWtiarApyqLcg/640?wx_fmt=png)

这里和之前连续几台 liunx 的靶机一样... 对于靶机利用找提权的漏洞... 直接 LinEnum 或者找 SUID，最后不行在找文件 php 和 txt 等等... 直接提权即可....

发现了 emergency 直接存在执行 root 的权限... 利用即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KN89iaUzP6kkQGeveb9q0mM9icibWbv5C1kTjqo4aIOQ7pgCX0oDJzBPZ5lRSa4b3tBIiayA0j9ZUXqHQ/640?wx_fmt=png)

执行获得了 root 权限... 获得了 root 的 flag 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/Zo04aoPGhhftaGG0yuEeaxw97HRiaFa8WJW7libBkFeicrPny8KnvKmeezoNnqicGdpWHkOm3eGAIXwGohqRuZ6S6Q/640?wx_fmt=png)

这台靶机在注入跳转重定向那块浪费了很多时间... 不停的测试... 后面提权非常简单的就得到了 root....

学习到了 BP 拦截停止重定向的注入方式方法.... 继续累积加油......

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台中等难度的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/py7L4cx8wYvBYkElUsqDz94g2u3uiaKibfK2IkLjMkEBKezINP2n0PyX4GwcXC1vl0K8KWnITP6HhjIuhyUBIXbA/640?wx_fmt=png)

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