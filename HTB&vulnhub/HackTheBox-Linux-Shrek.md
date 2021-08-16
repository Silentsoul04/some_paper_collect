> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xh0LddR1XYcf6f2hgvIDnQ)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **113** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/oIHlBAgsibibaZSBtkWaD9qOibMlGGKG1U6NJ4t4bgUXty0Q6AhXKFcYNEpsWiazic0HyVSyicaSjONOzZ6q74umEtwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nrhQN3cHv2mGJ5Zp11NuKa7IBOxh9iaVia3yXjAYFuTnjSpCS81z1gb5T803gZDe8jRRykjtd0Oic9xYejeugEyuw/640?wx_fmt=png)

  

靶机地址：https://www.hackthebox.eu/home/machines/profile/58

靶机难度：中级（0.0/10）

靶机发布日期：2017 年 10 月 19 日

靶机描述：

Shrek, while not the most realistic machine, touches on many different subjects and is definitely one of the more challenging machines on Hack The Box. This machine features several fairly uncommon topics and requires a fair bit of research to complete.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/A8ErDBmzzEsR7ggiaQSGgibribrg9F5UGDJHSzxDGGibMq2e9iaZoZ80WAoG5zC3erJpOOp8MVBso1u12B0fQOoUebA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0fb0Y1M6icJJia7t9xsBuUuxZQgOLeWHYicicRpfEiahMz3mlpK0icx8qLpfMLDojhD7IwSE2IalXVBBFs9E1Z88Ka3Q/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/PRVgXdHra5CzBfuOaOX4dpiaoOia6WZfdos1RiaJEZJG7nrnxTkXBoianpRmkQTmqkmW3zkbaQqjAu6WwBYAmyGibiaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8x6tJtIlqfkGUiauJv7gGyY0XTdqkHeK2pDoQBFTrMYWQiaHqS0bDzofA/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.47....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h89HwlWnxs9IB5dicY3VHJ4EX9OAyQ1MDibKjdc2Y4xwxqLb0TgVwmic4xg/640?wx_fmt=png)

nmap 发现开放了 ssh、ftp、apache 服务...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8UjM4VxCEdZ53iadKDLwPic0lGblsxy2k3ib4N5zaKC3Eicb4ZdMOc2eCCg/640?wx_fmt=png)

访问 web 没发有用的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8XAUsr0e5WfaSxHhIO6gF4bZknJ2Y2KhuFCvuExA6BN0Lg2gDn4NdKA/640?wx_fmt=png)

简单测试 robot 等目录... 存在 uploads... 里面很多熟悉的 php、elf、aspx 等文件，都是 shellcode 可写入的文件...

对于 update 也存在该目录... 里面存在文件上传功能，但是无法获得 shellcode... 一个坑

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8xuNVTGUfrBxckr559SRqMnkXXUK0GXRd2bhiaKRfcIwbp9LLjw0W9aQ/640?wx_fmt=png)

将文件全部下载后，检查发现 secret_ultimate.php 文件内容中提到了隐藏目录 secret_area_51/....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h83stG6JzrnEibcqRuU485bMMyh3noP6enbJbIfiaklkelQRPctl9bmkMA/640?wx_fmt=png)

可以访问... 存在文件

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h81QEBDGH3QPWjF2rOfAlorlmngNl0XYKicVZlM2sEERjYwcv3At46Xaw/640?wx_fmt=png)

查看发现是个音频文件... 前几章也做过类似的文件内容破解密匙...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8gboiafoI11r5D1q5JBN74ZdnCSylw800qH4mTiahtYIKrYLkddmI98iaw/640?wx_fmt=png)

利用 audacity 工具对音频文件进行播放，在 Calamity 也遇到过类似的音频解密，前面一直没获得有用的信息... 继续找下看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8oibHksEcfoCB9vatK4xgfgp0XtsJicL6K6qARI4GACkPR7QnRs9ia2pSA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8F97LmqGDtWG6Du4sBCmhiaeHQVIOvLfWGwGEq8jvqH7m86VS0lEU2wQ/640?wx_fmt=png)

设置高频...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8JicFiaWgD9tG4QG3xImibKibutaVBK2g055Qu4ib9FMRibYuWUeZP5na2eEg/640?wx_fmt=png)

在音频图中发现了一串密匙信息...（隐藏真深）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8MyUlcqy7xncyysmEicajx5UbFIRAFS4Q86YHgPG9Vwb0j6OONBrdMicw/640?wx_fmt=png)

通过密匙提示，发现是用户 + 密码...

通过 nmap 提示 FTP 和 ssh，测试 FTP 成功登录....

又发现了一大堆文件....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8owfYAiaQ8xROAoRFM7mLXOrHgQJxmncQvwM1dqwprXR7mTHU9Mq6Qhw/640?wx_fmt=png)

全部下载到本地分析....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8MDwibz5mVl5khzLmibw5qoIUrsZNSm6hlJRZ4k3QZMbTy2spnU2SFibkA/640?wx_fmt=png)

首先一个 key 是 ssh 的 RSA 密匙凭证... 这肯定又需要 ssh 登录了... 这里需要找到密码....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8gNblUEwuaB9zpCoibvtTyFPC3yiciaHNMEIxNVO2y9QzcPicNdv2twLtlg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h88ygBzDVrWnxdqeTrfibQ1sIyuyic7POPTb54wvsw8To00Ct4BTyWlGvw/640?wx_fmt=png)

通过 cat *.txt 查看所有的文本信息.... 有四处存在无字符空格现象....

查看后发现这是 base64 值.... 空字符只是区分的意思...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8CQ2H0IOFhl14fplGFTrYYAVoTPAr4B5RTZ47FuuTvRicPUwwDHYY1NQ/640?wx_fmt=png)

编译后发现了一连串的字符... 又要解密了....

这里遇到过一次在 vulnhub... 这是椭圆曲线密码术算法...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h85COOJZiblTPAY14KEibN9NP1aBEMjMLA7xg2CUkePdF1OEmgpNTLQUWg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8yTw3AR4wSFA2EjdmTzWayYBAcrsdKD9eOHEFZWGOEgGthKJJ0ibaP8w/640?wx_fmt=png)

```
https://github.com/bwesterb/py-seccure
```

google 搜索，可以看到利用 python seccure 可解密次字符串....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8rvXZZe9WMm1UUPPFrAMH2YJuylDEKJbdbSEXB38CicuIwgwcZce1CgQ/640?wx_fmt=png)

按照方法安装....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8CrFApF2icia5LKqKSqbQFL88WR6UsYib5Ayu5axo8ibC1sUyiamz0V4ib7pg/640?wx_fmt=png)

```
import seccure

dayu =
seccure.decrypt(dayu, b"PrinceCharming")
```

成功解密... 获得了 user 和 passwd 信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8QEVPA3P2xENUZNawZtlicJ29B78fwWgYwGkSVqAia0jdL1NfvyJHPxYw/640?wx_fmt=png)

通过 ssh 服务成功登录... 获得了 user_flag 信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8zWltWFAxicibcJUJPzOzAfEK64ZYicF1AvU7bmZUkZicsGQcULBH9H0aIA/640?wx_fmt=png)

上传 LinEnum.sh 枚举靶机信息... 这里发现 / use/bin/vi 可提权到 farquad 用户，提权后存在一个二进制 mirror 程序... 是个兔子洞... 无法提权 root...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8R68j9lQOoa2WHFWkB3c53lFgYSZ2AJKnItMZcfxSDnawlV0Vse9yTw/640?wx_fmt=png)

```
find / -type f -newermt 2017-08-20 ! -newermt 2017-08-24 -ls 2>/dev/null
```

通过查看靶机上线时间后几天的操作信息... 发现 thoughts.txt 存在引用字符漏洞？？

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8NFpJLsDWBb2p22UwtoaUBrLGluhnNnnfNeZGFwsubr6Enliat4P2xuw/640?wx_fmt=png)

创建个文件夹，简单测试，经过几分钟发现从 user 变成了 nobody 权限....

这类似的漏洞也遇到过，是通配符漏洞....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h83FYq1Gtc1wNsF29twJ9rcUfD2Uv2KFvZzPghfzm8sCRE0TgCbzZ7VA/640?wx_fmt=png)

```
https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt
```

通过 google 查看到这篇文章...nobody 权限漏洞利用方法...

创建文件 --reference=thoughts.txt，该目录内的所有文件将获得 root 执行权限，就像 thoughts.txt 由 root 拥有一样...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8iaTKkficjfeKWpk1zcSmKaP71ia9vPaiauo6qggq1zae2pusiakwR8eDaKA/640?wx_fmt=png)

创建文件 --reference=thoughts.txt，并写入 shellcode 程序... 编译后输出二进制文件设置为 setuid...

可看到当船舰 reference 后，dayu 文件夹经过几分钟变成了 root 权限... 等待几分钟....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOFJfavSibLHs7ju29WKn3h8Xo7gfn5DnefXAmAHh3PZWUAN5OptIwFU0UyWkjkJkrPEiaiaicic45j4Bg/640?wx_fmt=png)

通过时间推移，dayu1 二进制程序获得了 root 权限... 执行该程序...

![](https://mmbiz.qpic.cn/mmbiz_png/oIHlBAgsibibaZSBtkWaD9qOibMlGGKG1U6NJ4t4bgUXty0Q6AhXKFcYNEpsWiazic0HyVSyicaSjONOzZ6q74umEtwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nrhQN3cHv2mGJ5Zp11NuKa7IBOxh9iaVia3yXjAYFuTnjSpCS81z1gb5T803gZDe8jRRykjtd0Oic9xYejeugEyuw/640?wx_fmt=png)

  

获得了 root 权限，并获得 root_flag 信息....

信息收集 + 隐藏目录音频文件破译 + FTP_base64 值破译 + 椭圆曲线密码破译 + 通配符漏洞提权

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台中级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/A8ErDBmzzEsR7ggiaQSGgibribrg9F5UGDJHSzxDGGibMq2e9iaZoZ80WAoG5zC3erJpOOp8MVBso1u12B0fQOoUebA/640?wx_fmt=png)

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