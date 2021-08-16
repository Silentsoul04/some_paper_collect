> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/v2I2BcLVoVL0zP1JVWKZhQ)

> **来自公众号：小白学黑客**

CPU 起飞了
-------

最近有朋友在群里反馈，自己服务器的 CPU 一直处于高占用状态，但用 **top**、**ps** 等命令却一直找不到是哪个进程在占用，怀疑中了**挖矿病毒**，急的团团转。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyES70icWWt87HB0PSFfzvvyYJq3LV43GKzWVrREMBSqHgdAKG8M6sLWp9A/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESZVCmOmKDoDKnmqQiabXfHaSqKXZIJ4zQHbO1eUOChw0Yy6R4C5b2xXg/640?wx_fmt=png)

根据经验，我赶紧让他看一下当前服务器的网络连接，看看有没有可疑连接，果然发现了有点东西：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESrZicgxOJdzMicyFicGeZNKf5kvibCpQnw0hGrNYiaNHSvCKbIhRUwvqkA9w/640?wx_fmt=png)

上 **Shodan** 查一下这 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESEbsIgmuRfbmuicgdh4H9dRJSJ8vjvEnf73f97pA7ay93bJia1WDjHreA/640?wx_fmt=png)

反向查找，发现有诸多域名曾经解析到这个 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESjrekMF8sHc0nTfHhic6A071ADSEzkt7fG5uZ9NuicKic4Yl4GQAIq6aPQ/640?wx_fmt=png)

这是一个位于德国的 IP 地址，开放了`4444`,`5555`,`7777`等数个特殊的服务端口：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESEN88wTjfOicd4Zqw5x0nqAvsPa5YK6ViafPYogHr2sicribyYFbsTOxrtQ/640?wx_fmt=png)

其中这位朋友服务器上发现的连接到的是 7777 端口，**钟馗之眼**显示，这是一个 HTTP 服务的端口，直接访问返回的信息如下：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyES0zmiaFxmcjj73APCNjWDD7clsnXhqbZM2qdu12SzjTLTfyrKWtY0BbA/640?wx_fmt=png)

**mining pool!**，服务器正在挖矿实锤了！

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESY2mRejGbzLoJFzP90qaYPx5uoPicgHp5MxM1NjDcusumfTZXbT3o0SQ/640?wx_fmt=png)

但神奇的是，这个进程像是隐身了一般，找不到存在的任何痕迹。

进程如何隐藏
------

现在说回到本文的正题：**Linux 操作系统上，进程要隐藏起来，有哪些招数？**

要回答这个问题，先来知道 ps、top 等命令枚举系统的进程列表的原理。

Linux 的设计哲学是：**一切皆文件！**

进程也不例外， Linux 系统中有一个特殊的目录：**/proc/**，这个目录下的内容，不是硬盘上的文件系统，而是操作系统内核暴露出的内核中进程、线程相关的数据接口，也就是 **procfs**，里面记录了系统上正在运行的进程和线程信息，来查看一下：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyES4F71kKwiaiadIVTj0Jn7qaT5xw0pbcZJsKUj6OTNA0qSTwRicU6HdE1zA/640?wx_fmt=png)

这些以数字命名的目录，就是一个进程的 PID，里面记录了该进程的详细信息。

而 ps、top 等命令的工作原理，实质上就是遍历这个目录。

知道了原理，想实现隐藏就有以下几个思路：

### 命令替换

直接替换系统中的 ps、top 命令工具。可以从 GitHub 上下载它们的源码，加入对应的过滤逻辑，在遍历进程的时候，剔除挖矿进程，实现隐藏的目的。

### 模块注入

编写一个动态链接库 so 文件，在 so 中，HOOK 遍历相关的函数（**readdir/readdir64**），遍历的时候，过滤挖矿进程。

通过修改 **LD_PRELOAD** 环境变量或 / etc/ld.so.preload 文件，配置动态链接库，实现将其注入到目标进程中。

### 内核级隐藏

模块注入的方式是在应用层执行函数 HOOK，隐藏挖矿进程，更进一步，可以通过加载驱动程序的方式在内核空间 HOOK 相应的系统调用来实现隐藏。不过这对攻击者的技术要求也更高，遇到这样的病毒清理起来挑战也更大了。

揪出挖矿进程
------

通过上面的进程隐藏原理看得住来，都是想尽办法隐藏 / proc 目录下的内容，类似于 “**障眼法**”，所以包含 **ps**、**top**、**ls** 等等在内的命令，都没办法看到挖矿进程的存在。

但蒙上眼不代表不存在，有一个叫 **unhide** 的工具，就能用来查看隐藏进程。

我让这位朋友安装这个工具来查找隐藏的进程，但奇怪的是，一执行 **yum install** 安装，远程连接的 SSH 会话就立刻断开。

于是退而求其次，选择通过源码安装，又是一直各种报错 ···

因为我没办法亲自操作这台服务器，沟通起来比较麻烦，于是我决定研究下这个 unhide 工具的源码，然后编一个 python 脚本发给他执行。

源码地址：`https://github.com/YJesus/Unhide-NG/blob/master/unhide-linux.c`

在查找隐藏进程模块，其大致使用了如下的方法：

> 挨个访问 **/proc/pid/** 目录，其中，pid 从 1 到到 max_pid 累加
> 
> *   如果目录不存在，跳过
>     
> *   如果是 unhide 自己的进程，跳过
>     
> *   如果在 ps 命令中能看到，跳过
>     
> *   剩下的，既不是自己，也不在 ps 命令输出中，则判定为隐藏进程
>     

按照这个思路，我编写了一个 Python 脚本发给这位朋友，执行后果然发现了隐藏的进程：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyES5Wvm9S0kJRqrWtq9zWHfO8y2ibsuC5HrezIGpQ6b5fKMrWVfHr5tasA/640?wx_fmt=png)

别着急，不是真的有这么多进程，这里是把所有的线程 ID 列举出来了。随便挑选了一个看一下：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESCOuggngia1W2Avm5sRj0DdzlBIWmJOXqY6l8aYKzET34he0xD9m3E9A/640?wx_fmt=png)

还记得前面通过 **netstat** 命令看到挖矿进程建立了一个网络连接吗？Linux 一切皆文件，在 **/proc/pid/fd** 目录下有进程打开的文件信息：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESsmsb9A1f70x9DhO1icKiaYHicMse9LEVXOA9ztJwYnoSyWGGKOwwlbCicg/640?wx_fmt=png)

这里发现这个进程打开了一个 socket，后面的 10212 是 inode id，再通过下面的命令看一下这个 socket 到底是什么：

> cat /proc/net/tcp | grep 10212

输出了四元组信息：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESiaZL3zZiauq0CnRPI2QibQcmHEtKQicmdWyPzibYOOsQIEG0w0ia4ql8mepw/640?wx_fmt=png)

左边是源 IP 地址：源端口，右边是目的 IP 地址：目的端口

目的端口 1E61 就是 7777！！！

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESGEPbFeEqALsqUXgdZ21iaGgUEItIhb02V6ZWJsZM8XlI8iaSl8icwACEA/640?wx_fmt=png)

找到了，就是这货！

再次查看 **cat /proc/pid/environ**，定位到进程的可执行文件：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESzKSwQPaiaK3vQ1NCncPM0mcjiaDrcmrgj4aR5TR7yVjXHE1Tl6KoVWcA/640?wx_fmt=png)

总算把这家伙找到了：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESTCa2dFJ97pQymCkpG4qtXQrU4hfFkQa2RXx9W1uIb1IbUJdTSPceyw/640?wx_fmt=png)

网上一搜这家伙，看来是惯犯了：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyES1qARPaOLibUYiagwIZpTRgzxx6Hl3oblCtbSdPPKgeDp0YssT2ZaXwsw/640?wx_fmt=png)

挖矿病毒分析
------

把这个挖矿木马下载下来，反汇编引擎中查看，发现加壳了。

脱壳后，在 IDA 中现出了原形，不禁倒吸了一口凉气，居然悄悄修改`/root/.ssh/authorized_keys`文件，添加了 RSA 密钥登录方式，留下这么一个后门，随时都能远程登录进来。

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESs307E4Fn5sLWTlCYx6iapwJ2iagOQVf4IzbfT7ib0Y7xvBuSncWZWdTNw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESoicyDvS0vHcptZWlTx5U8HxYGcks326uFX5X1w6nv1sSgeEAjaBg2BQ/640?wx_fmt=png)

除此之外，还发现了病毒尝试连接的大量域名：

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESiaZfYe2Uia2YQ9u3LeheP7wxIjof3XrLNWtQRW53lRRQENbxCxaLqd9w/640?wx_fmt=png)

看到这里简直可怕！自己的服务器被病毒按在地上摩擦啊！

![](https://mmbiz.qpic.cn/mmbiz_png/H1LuiaOqzMhfXes0pHtcY6WicXaMiaPOyESJe5xlhT8ibR4m4HuMqyUGFNU6MKGowQo4Xv4vawxDtEdjpLficb6fEvw/640?wx_fmt=png)

清除建议
----

> *   开启 SELinux
>     
> *   杀掉挖矿进程
>     
> *   删除病毒程序（注意 rm 命令是否被替换）
>     
> *   删除病毒驱动程序（注意 rm 命令是否被替换）
>     
> *   删除病毒添加的登录凭据
>     
> *   防火墙封禁 IP、端口
>     

这个病毒到底是怎么植入进来的呢？？？

咱们下回分解～

**推荐↓↓↓**

![](https://mmbiz.qpic.cn/mmbiz_jpg/NVvB3l3e9aG5kWic5P8XOwFOhXKjibAt6YNpz8iaibic57tSkQa7tQ68ibqN2PG0Pz7ReF8CnwcVL7AZlJyHnbn4xVMg/640?wx_fmt=jpeg)

**运维**