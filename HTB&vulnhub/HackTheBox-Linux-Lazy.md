> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ztCqmMxa6mlnNcnIsPImYQ)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **94** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/Zo04aoPGhhftaGG0yuEeaxw97HRiaFa8WJW7libBkFeicrPny8KnvKmeezoNnqicGdpWHkOm3eGAIXwGohqRuZ6S6Q/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/18

靶机难度：中级（4.8/10）

靶机发布日期：2017 年 10 月 5 日

靶机描述：

Lazy mainly focuses on the use of padding oracle attacks, however there are several unintended workarounds that are relatively easier, and many users miss the intended attack vector. Lazy also touches on basic exploitation of SUID binaries and using environment variables to aid in privilege escalation.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/py7L4cx8wYvBYkElUsqDz94g2u3uiaKibfK2IkLjMkEBKezINP2n0PyX4GwcXC1vl0K8KWnITP6HhjIuhyUBIXbA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XrBsia6eKtTFtr4vwm8FVt5frF8ojc6Xtp0ChSOwic1tRYkxthCoB1v1SekZZzcuvLGhDnRCDt8IVxpHV9flfc9A/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PXWEicxy2xC9I5LF9rFMqYCphbH3cCW3vuSv6ic9WhOhzpoJibNkQLKy2DAWiazzOcIg1RLfgZiauLaLG8ucgHOJGdw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFfzsib6IXRIBLIjG3zIRWQ0qiaY0cicMdL26MlpjZMaiaRbRm8kx49M6gibA/640?wx_fmt=png)  

可以看到靶机的 IP 是 10.10.10.18....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVF6Hmib4d1wNcwewGoSibz4GIBTBq42el1nkPZFOe0NB3AMBP9kiaNpia75Q/640?wx_fmt=png)

Nmap 发现开放了 OpenSSH 和 Apache 服务....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVF0V1uVxlYvl68G4qlGGbm3giazB0F5GSxvqMibJSW6bycPwJib6pvLqdsw/640?wx_fmt=png)

登录进来有一个 Login and Register 的选项，创建一个新帐户...dayu

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFkTSQiasyW76w1akHhgxAyicmnib0UWUOrR32qNnhNZ7xiadr1K2nN9f9kw/640?wx_fmt=png)

这里我利用 Bp 拦截分析了 sql 注入... 发现存在注入... 直接测试后前几个就测试成功了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFe7puE0ibZe8WOSzUS9klyweIbn9JvYQNxTLgKCyrRZCmoBz2LbcYcWA/640?wx_fmt=png)

直接通过 admin = 注册账户后，直接登录到了 admin 管理员用户权限内.... 这里估计存在 BUG... 或者作者遗漏了什么重要的东西...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFnprThLwa8h7CgDKmuR8KreTcicaSDHOBpicXDjBHgdbrXA6JaYOrAkvg/640?wx_fmt=png)

进来后直接可以看到存在 rsa 密匙凭证... 那这里直接通过 nmap 发现 ssh 服务登录即可获得 user 信息...（这里和昨天的靶机一样，昨天文章靶机直接文件上传就提权到了 root，惊呆了...）

但是我还是想通过别的途径继续获得密匙... 继续深入看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFX8E1rzeV5NRfUz9oMq160qtsM1ibD42m8Qib2lrXHgl5Vqu1AYd4xuIA/640?wx_fmt=png)

这里前面就创建了 dayu 用户... 登录

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFAT9onibG5ZOGDbicuWAfhic6N4eq9EIIiagG0IrI4CicEcQLEvq6ZTiaOmNQ/640?wx_fmt=png)

继续 BP 拦截查看到了 cookie：auth 值...

这里存在漏洞，存在填充 oracle 攻击行为：

```
[维基百科](https://en.wikipedia.org/wiki/Padding_oracle_attack)
```

可以阅读下此漏洞... 现实会遇到的...  

填充 oracle 攻击是一种使用加密消息的填充验证来解密密文的攻击，在密码学中，通常必须填充（扩展）可变长度的明文消息与基础密码原语进行兼容，攻击依赖于具有 “填充预言” 的人员，该人员可以自由响应有关邮件是否正确填充的查询...（这是 google 翻译来的...）

吴翰清《白帽子讲 WEB 安全》书中也讲解了此漏洞... 开始吧，不懂的百度也能查看很多信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFXaiaw8Eib1wM2OCvos0PdYt5l7RWg4MCpuXJZxQ2SjEL3G1tQpBdC7zw/640?wx_fmt=png)

```
perl padBuster.pl http://10.10.10.18/index.php BYWO%2BQTJb7XTDybidv9P%2BkewvWVcRk8J 8 -cookies auth=BYWO%2BQTJb7XTDybidv9P%2BkewvWVcRk8J -plaintext user=admin
```

直接利用 cookie 截取的 auth 值进行...

这里利用 [padBuster.pl](https://github.com/AonCyberLabs/PadBuster) 获取 admin 的 auth 值...

经过一段时间，获取到了... 直接更换填入即可...

这里就没截图了，通过 BP 更换 auth，然后输出，页面会通过 admin 的 auth 跳转到 admin 权限页面... 从而获取 rsa 值...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFicxkFqPa685jMWrYv9R7uoBich55KnGlkwgDbmNJibChibmib4SwjJFgENQ/640?wx_fmt=png)

通过获取的 rsa 值，ssh 成功登陆.. 获取的 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFteF7ibQSeE3cwofic0YbDcuxKuTc6JcaqE25Nc0ThxQcWSBiaZZciauRicQ/640?wx_fmt=png)

在 mitsos 目录下还存在二进制 backup 程序... 分析看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFqbDu5mW7hXxHBv0ZibvIKVXVfPxW4Sm1nBRicj2KglhOf3ycsTFZVszQ/640?wx_fmt=png)

通过分析... 在备份实用程序上运行字符串会显示它引用了 cat 命令，而没有指向二进制文件的实际路径它显示 cat /etc/shadow 并且 cat 未指定完整路径....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFejakf12FguRaPiacKeKuTlrttibrTFudr0LoYNP0XfQeicic6BhCHicMPTA/640?wx_fmt=png)

检查 PATH，可以看到 cat 实际位置...

cat 在 / bin / 中，直接创建一个 cat 文件，写入 / bin/sh 并赋予权限... 在执行 backup 后会首先搜索 cat /tmp，一旦找到它将执行并以 root 用户身份给我 shell...

成功获得 root 权限...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMXAS5VWI7MeqnZC18IxcVFb4ic2pTdicXrhYf7PDbQ7iajpW7rjRIY5kSklghtEDSibkgN7F1ndxzFUg/640?wx_fmt=png)

通过读取方式获得了 root 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/Zo04aoPGhhftaGG0yuEeaxw97HRiaFa8WJW7libBkFeicrPny8KnvKmeezoNnqicGdpWHkOm3eGAIXwGohqRuZ6S6Q/640?wx_fmt=png)

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