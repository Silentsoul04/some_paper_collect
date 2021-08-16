> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gDMxs3XrG9BpDz1WTowcBA)

  

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgsKrE9hkWT8icicPpewKbU6xjRiboONOXtlb00ALtmyHY8yhns5ibVTBjq6ug8Nl4icDF2c2EU25zrmxA/640?wx_fmt=png)

**点击上方蓝字关注我们**

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgsKrE9hkWT8icicPpewKbU6x2ke1YJgviaYL1cutBlRcTcL4NxFIAezDOe0QGbUtAoT1nYN8yYw9ILQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgsKrE9hkWT8icicPpewKbU6xhtia0qIE6nHicZWoQSJ97qxzH90fuE9rEXFGT4r0esXQicOrUDPxhvOuQ/640?wx_fmt=png)

01

靶机介绍

这次的靶机是 Retro, 房间链接 https://tryhackme.com/room/retro 虽然难度为 Hard 但是如果能扩大利用条件的话其实该靶机还是比较简单的。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPXSoJYibAe031uYGOhfoVWXeNddlR3AruZuL8icak3eDjLhFv4TXEzibYA/640?wx_fmt=png)

02

信息收集

一开始还是先信息收集。这次不知道为啥 autorecon 联动的 nmap 居然扫不出 80 端口出来。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPYjnwwO0hfZgnWkhZKAmcypScC8s4wBibewTUtic1Y2OdKarDTmoHDv7w/640?wx_fmt=png)

但是浏览器访问 IP 地址可以正常运行

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPFia4LSfxq39zicNfCkHDSxYpbichSaU63sbYjslCuhTakU5OJj5UfRZOQ/640?wx_fmt=png)

由于此次 gobuster 运行不了只能改用 dirsearch 加载 medium 字典可以跑出有一个 /retro 目录 (IP 是昨天的，但内容基本上一样不会有太大差别)

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPprG0IWDkicDFXj9YkW0SDhxfMgw4UiaZZuxKlXAfCZzHraXeF1rAPlkw/640?wx_fmt=png)

继续访问页面正常回显。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zP8n7SuEMCaJuGBAsm75kY9R2512TcujwqYOtYr3gTdBotBVbiby9xTjg/640?wx_fmt=png)

03

Wordpress 网站渗透

这里了你可以选择继续对 /retro 目录扫描, 也可以用 whatweb 看看该网页是不是个 CMS。这次可以看到是个经典的 wordpress 站点。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPh4bzf8lOScD97YCaj77BV7nxg8icfTgtQoCBqn1ypj80DticrFIxLTYw/640?wx_fmt=png)

先用 wpscan 基础扫描扫到一个 wade 用户。

命令:wpscan –url ip -eu

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zP42Dibibsp03KW7mw0kc0pdsMCDJhlic42DxLnYKUxribwvKgJ0k5C5preQ/640?wx_fmt=png)

由于 /retro 的文字信息比较多可以考虑用 cewl 扒下来看看有没有敏感词语是跟密码有关的。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPEdibUCFndDfxnGnqNgug0aej8vQgMJF59pk6uiaDIJzibfe9P33ZIn5NQ/640?wx_fmt=png)

然后启动爆破跑出一个有效的密码出来.

命令:wpscan –url ip -U wade -P shouji.txt

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPAyDk35TiaHpkOc84ynZrsNJdoJDQ1eEOQibpzNq3BibwY1M37Oq7u9b4w/640?wx_fmt=png)

04

密码重用, 远程登录

这里呢按照正常的思路是到后台上传反弹 shell 的，但是呢在信息收集的时候 3389 RDP 是开放状态可以考虑用刚才得到的账号密码信息尝试登录。  

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPPxJrPicKDZofxBtJ8McNgjAGBaicdM7NHiaMibxXDz9soGT4LakGdicl5iaw/640?wx_fmt=png)

利用 xfreerdp 成功登录。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPp8MuxKwWaSiaB2EYyiaVIYd8RFibUiamONHCvJsSVfsmc9Nbiauyf5yYosw/640?wx_fmt=png)

05

提权

一开始传了个 PowerUp.ps1 进去，但发现没啥提权点，只能转换了思路看看系统信息看看有没有没打的补丁或其它漏洞利用

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPqPalia75j7pnVUPCEjnqfVicMmwSQAMJtCsec5qBxuE8fEOwUic8FYrpQ/640?wx_fmt=png)

接下来用在线提权辅助 https://i.hacking8.com/tiquan 把系统信息复制进去点查询，优先考虑有 payload 的，这里我选择 CVE-2017-0213

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPT7bFlibfxOibwicJsl43Jf9eZBe6WSBb8n1RtDlxPMhEicZQKNMFEs6odA/640?wx_fmt=png)

通过 CVE 编号搜到一个 Github 脚本

https://github.com/WindowsExploits/Exploits/tree/master/CVE-2017-0213

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPa2LSmp5nKIbzzTHpD7Q0LSR8Gtvw79ya8USeosEMp0FY6KENr2LleQ/640?wx_fmt=png)

下载到 kali 以后用 smbserver 开启共享模式

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPecOQopVica0jo7WgJs4SHYGkNdqI2y8zgeDW1FaoWQzQtL05bkuI5Uw/640?wx_fmt=png)

在 xfreerdp 远程桌面上使用 copy 大法把文件复制过来。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPRU4XBZppmF3BwicCpL4M0M9qqrlEnoZfJEqbGjicFqMSzlG0kjXz1daw/640?wx_fmt=png)

运行方法很简单直接运行 CVE-2017-0213_x64.exe 就可以自动权限升级了。

![](https://mmbiz.qpic.cn/mmbiz_png/tF1M75DDm9QzDpQdQP5kE5Sickib0B30zPblLvg2AIDpgBbt55dFTkOz31V3QqdzIUib7eZbKWu5GZM24cWtYHNvA/640?wx_fmt=png)