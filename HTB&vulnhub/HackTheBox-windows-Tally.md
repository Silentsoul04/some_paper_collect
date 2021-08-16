> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0Plm0jyP46DW8G_IFXvSig)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **16** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/113

靶机难度：中级（5.0/10）

靶机发布日期：2018 年 5 月 4 日

靶机描述：

Tally can be a very challenging machine for some. It focuses on many different aspects of real Windows environments and requires users to modify and compile an exploit for escalation. Not covered in this document is the use of Rotten Potato, which is an unintended alternate method for privilege escalation.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_gif/n4YCONia9OnVdcsxvENIJZhKuYmLVN6mgZ00dHibVm6LCuGf7OBFiaoCvgWU91ibFQDH99J95ZYybkRwP3Bk2IeroQ/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_gif/yQJ56734ZYGfTXlb9EfkjcOm3QV6qvXC55Xmjs4Ratr9cy6KXiadaRnpjvY67FtHd8iaWsyNBAASVrJ6INp7lgcg/640?wx_fmt=gif)

  

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWy0xkzr0G05SJzvGHb3oudaF163QUzS6ZEEEad5R40xiafmR7lU3AMYCQ/640?wx_fmt=png)可以看到靶机的 IP 是 10.10.10.59.....

Nmap 揭示了目标上运行的大量服务.... 最值得注意的是，有一台托管 Sharepoint 的 IIS 服务器....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWy1wgPumEwfys4ib2SaUaM0hBiccLQWbV3ZpKWkqGP2pwiaIbficLaLmwa9Q/640?wx_fmt=png)

访问 80 发现 HOME 需要登录密码才可以进去...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyribOJ5DQRiaEPJNkiamYMcMw0B1g6RWvS3pmjLY9tAavZSSiaZ9bbBictAw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyNNoTgsyTzSsXGj9q8icaTiaxS6WUYcML3iatN1QIib3sd13hflrecJ7QUw/640?wx_fmt=png)

利用 gobuster 扫描发现了 viewlsts.aspx...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWymLnxRESiaKj1bTiaoBCvAojYbYnj2BUBriaKfQWhDJscH9JfEkFoyXpkw/640?wx_fmt=png)

可以看到这是一个主页...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyiazreKAVGTnjcVw98l9mZRH29oyCCcIBEy0vvfYfpYgYRaYKGHRYQIg/640?wx_fmt=png)

列出了目录中网站的文档和页面....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyrFvzC1wIuP3UFwdRqRY7wQNbYCthCKCZ4urTpElWhKN3DCGJMs2BNw/640?wx_fmt=png)

可以看到下载了文档，查看了内容... 这是 FTP 的密码... 目前还不知道账号...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyAcicJWQ0ChGbiaFAceibdgMpicbIMmEY0jRGZtayHkt27y0eQ4dkAf6AkQ/640?wx_fmt=png)

在 SitePage 里发现了例外一个信息... 发现了 ftp 的用户名：ftp_user

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWy7cMFDISVpicswdUn79hvBlwO2ICkdYLr4DV2h38IadDhDaCEnF9oO8g/640?wx_fmt=png)

成功登录，在 Files 目录下发现两个文件，下载到本地分析看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWywTGrABnOKYM94RKnIrQRs8zZMYdnPoNiaerjbpMIhQqsgf8rPq1icY1g/640?wx_fmt=png)

知道 kdbx 是哈希密码，利用 keepass2john 编译，然后 john 来爆破即可知道哈希值...

可以看到已经破解了账号密码...

然后继续进行数据库登录了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWySibRcBc8pckCYOdibd8K9KJfnVVdXnsWCo5vJ5806Z74LU3Iln3MvEbg/640?wx_fmt=png)

成功登录数据库后，在 zz_Archived 文件夹中找到一些 SQL 连接信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyLCx29u8W6xeKzOOQU4zicyaF8mpO95d6TiaGgyEhABkBR20TdrEVnwdg/640?wx_fmt=png)

将下载到本地查看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWys9xHWGGTaa2ZDdu7qhYYj74BDgsYXHogficiaTD4FMjNII39SEQgKupw/640?wx_fmt=png)

又发现了密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyVSDyd3HTvSImEJO1wJL6UaRvsiciat2adltW5dYI6jAsoqaS4hibKJMJg/640?wx_fmt=png)

在 \ zz_Migration\Binaries 目录下也发现了 tester.exe 文件...

利用 shrings 查看了内容，发现了信息：

```
UID=sa;PWD=GWE3V65#6KFH93@4GWTG2G
```

![](https://mmbiz.qpic.cn/mmbiz_gif/yQJ56734ZYGfTXlb9EfkjcOm3QV6qvXC55Xmjs4Ratr9cy6KXiadaRnpjvY67FtHd8iaWsyNBAASVrJ6INp7lgcg/640?wx_fmt=gif)

  

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyjw4KtZHdiaMKhdp82eibHjFOkVRxTebyzajIMJRXza3hHg4PORvBSDsA/640?wx_fmt=png)

利用 metploit 暴力提权吧...GO，MSF 生成反向 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWy5wOfV0VVuMOlVgAFoDzZt9NntEYAInkAtaFia3WRzhUiafT5icsDC1EUA/640?wx_fmt=png)

成功通过 FTP 上传了反向 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWy3t4fgWCYhk9QbuGtVH7IUttx3Ku48A0f1Yqo3eibJSqpB2nsoNfeYEQ/640?wx_fmt=png)

直接利用 auxiliary/admin/mssql/mssql_exec 进行 mssql 的命令注入，然后成功提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWy59tRv5TMTUtf9W74pWezXNGVcJsMvMdYWAqgfQq8oBaoNMLxDfhuXQ/640?wx_fmt=png)

这里学习使用隐身模式和 RottenPotato...GO

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyXfvEQUjGicnd8vJqLTyr80zDpGDlRvn7IY7mP5OQicZ2RGibu4WazoGGw/640?wx_fmt=png)

通过 getprivs 命令，我们可以验证对当前进程启用的所有特权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyFkqK0hSaPXYk6M2CicGjO7ruMHzgml0iccrvkGoweibK0sVDYswqGjU3w/640?wx_fmt=png)

先活得 user.txt 凭证...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyrNibMibFWkowlzwFR2sNppltbbAFiarlj0lU4CwVdudnOVnhmzeUXHJdQ/640?wx_fmt=png)

下载 rottenpotato.exe 进行提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyO6b3D2GiajrZlsYniciaCxRThKGPp1jMXFfOh00V9iaKJ7f11ummWwk5Nw/640?wx_fmt=png)

上传 rottenpotato 到靶机桌面目录下....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWy5WdR1BibR9ytuejYS5zIqZeNoSJJUa2Kd1y54utLiabLkbDT4ZfM0LJw/640?wx_fmt=png)

执行 use incognito 命令，将模块加载到 Meterpreter 会话中...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyRkOSVAQ4iactibpO9cOEBdkozgFvYnMDgnO2rUlksuCwYQ6mNeMUcVPQ/640?wx_fmt=png)

这里看到有一个有效的管理员令牌，模拟该令牌以获取特权即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyeGDJcDVPmhamJjvGkPzM0vUPibib7j0kmtHQic2G710ia3QiaaPRkJtFqcQ/640?wx_fmt=png)

可以看到获得了 system 权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KM9icWtNibxJCgtSboQbOSFWyWDuLsexpRo7S3bfUIPeiaQHJGxzV1AdBC1HKFcfy8eAoyMHgjzcUPTw/640?wx_fmt=png)

成功获得 root 信息...

这里还有很多漏洞，可以利用 EXP... 最近时间没那么多些出来，我都自己记录在自己的文档里了...

感兴趣的可以继续深挖学习！！加油！！

提示：这里还可以利用 CVE-2017-0213、CVE:-2016-1960 等进行提权...

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_gif/n4YCONia9OnVdcsxvENIJZhKuYmLVN6mgZ00dHibVm6LCuGf7OBFiaoCvgWU91ibFQDH99J95ZYybkRwP3Bk2IeroQ/640?wx_fmt=gif)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)