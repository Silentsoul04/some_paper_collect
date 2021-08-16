> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/SrKQ-hYW06LPaNHYntkSGw)

**InScan - 开源扫描器**

工具简介
----

_本工具只可用于安全性测试，误用于非法用途！_

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXkRczFoXjHfJMbdDHEuKs4oaVibNy3yh0jYTdaIpWCvBsDRw3sVKZNyg/640?wx_fmt=png)

### 工具定位

边界打点后的自动化内网工具，完全与服务端脱离。服务端只用于生成 poc，网段信息等配置。

### 内网渗透痛点

目前已有的扫描器，依赖库较多，体积过于庞大，在内网渗透中，很多极端情况无法安装扫描器，使用 socks4/socks5 代理扫描的话，时间久，效率低。

### InScan 优点

*   多平台，单一的二进制文件，免依赖;
    
*   支持自动可视化多级隧道，通过后台按钮开关即可穿越多层网络;
    
*   支持 ipv6 的扫描器;
    
*   快速直观查看多网卡机器，方便快速定位能穿多层网络机器;
    
*   通过已知密码生成社工字典，快速横向内网;
    
*   内网 B/S 架构系统自动化爆破，验证码自动识别;
    
*   快速资产识别，站点截图;
    
*   通过扫描到的资产自动化进行网站目录扫描；
    

### InScan 支持平台

全平台支持，一个二进制文件，开箱即用。命令行启动

```
-pocPort int
        rpc端口，默认：8009 (default 8009)
  -rpcPort int
        rpc端口，默认：8008 (default 8008)
  -sysTime int
        系统信息上报时间，默认：15秒 (default 15)
  -webPort int
        web端口，默认：8080 (default 8080)
```

#### Windows 使用

推荐管理员权限打开 cmd，在 cmd 界面执行 inscan.exe（管理员权限可支持 icmp 快速探测存活）

#### Linux 使用

赋予文件执行权限

```
chmod +x inscan
```

```
./inscan &
```

```
wget 后台生成的arm架构程序
```

#### Android 近源攻击

破解 WiFi 密码，手机安装 Termux.apk，打开终端。（不要勾选 icmp 探测存活）

```
chmod +x inscan
```

赋予文件执行权限

```
./inscan &
```

后台执行

```
192.168.1.1/16,172.16.0.0/8
```

Termux 切后台，使用手机浏览器访问 即可开始扫描。

### 横向移动生成器

填写 ip 地址段或者与域名，开启自动化目录扫描、爆破、字典生成等功能。  
ip 地址逗号分隔或换行分隔  
示例：

```
192.168.1.1/16
172.16.0.0/8
```

或

```
192.168.1.1/16
172.16.0.0/8
```

```
弱口令字典生成
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXD5sdRjhBR9FXM6j2sz65ndW3XACoSBWiaWIia1X2y5fg9dCiaJjNkCSOw/640?wx_fmt=jpeg)

```
poc选择
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXv3kP5xPJLvght4z71qibgLxjPibQmbhozq8y1rlibU46veEtiaERwy0ImQ/640?wx_fmt=jpeg)

#### 精准的扫描方式

InScan 首先做端口扫描，然后把状态为打开的 TCP 或 UDP 的 IP + 端口传递给服务识别模块，这些 IP + 端口会并行做服务探测。一旦连接建立成功，InScan 会尝试超时等待，一些常见的服务，例如 FTP、SSH、SMTP、Telnet、POP3、IMAP 服务会对建立的连接发送一些欢迎的 banner 信息，这个过程没有发送任何的数据 (也就是只经过了 TCP 的三次握手)，在等待的时间内如果收到了数据，InScan 会将收到的 banner 信息和空探针的上千个指纹库进行匹配，假如服务和版本信息完全识别了，那么这个端口的服务识别就结束了。假如 InScan 探测到的端口为存活，但是没有获得 banner 数据，那么 InScan 会根据对应端口和优先级动态调整数据探针指纹策略继续进行扫描，直到完全识别到服务和版本。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXABficXXwIOvibWh4MXgibiaZxnExpr3FWxfxbafhZP30UttYjXkzpTk2Yg/640?wx_fmt=png)

### 扫描结果

多网卡流量监控

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXu3anib5W8DLullF46sqMywsdq07zl44RSNQPfp4w4iceUT6DuRejGmDA/640?wx_fmt=jpeg)

```
操作系统详情
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXLH8tn8D7ibJ4ib6OGFJEGQPI48tum03TwSvaUlc7ezYdx1aJS6CrDMJg/640?wx_fmt=jpeg)

```
网卡、cpu、内存详情
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXaHdrV5VlpBD8xhByicjA7ly6jGNfqUiaTQO2LibD2rpPlFVllWxkPjzSQ/640?wx_fmt=jpeg)

```
可视化扫描进度
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXWY7GF34kVoWicmPriaia85bTZicoBrbBTmCC9paWJbbR9EtOBRP7ydiaibEQ/640?wx_fmt=jpeg)

```
banner详情 
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXcmazVVibd28QVXrBke7TGRVp3ZoOrNMp61DCP9FNo7T5pPpOUwtxzcg/640?wx_fmt=jpeg)

```
banner详情
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXs1kDskTeQOAYgmsNdGkssNSZeq4oj9WIicHNGhZicXO5vbMfPH3aKMFw/640?wx_fmt=jpeg)

```
可视化网卡识别，精准定位多网卡机器。
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXvq9YkGfhvSn9GI7PSw4ic6QlwmIAf9g86NKD0VSMC0s5GgiaCjezz92g/640?wx_fmt=jpeg)

```
poc漏洞验证
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDX1icWxniae28V4awGHT25dIjnhbdrZLvcFKUam5ic2eE6ZN6p4wXXCNHUg/640?wx_fmt=jpeg)

#### web 管理 ssh

爆破成功后的 ssh 通过网页操作。  
开发中, 近期上线............

#### rdp 管理

爆破成功后的 rdp，通过网页操作。  
开发中, 近期上线............

#### 数据库管理

web 界面实现数据库的增删改查功能，以及打包下载。  
开发中, 近期上线............

#### web 目录扫描

开发中, 近期上线............

#### web 登陆框自动爆破

机器学习的验证码识别库，自动爆破内网可登陆的 web 系统。  
开发中, 近期上线............

### poc 管理

#### 提交格式

目前支持 xray、nuclei 等模板，后续支持更多。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXIRuTF0hk7M3kFy0o1N1eyia2j04RGvgJpuqHSnmjnAFf2ibDBlNEzMQg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXt78Z7elDGZibauy6UdlyhEHxzzyiabwDH5DqPgaMxFfeyqsAoTfia8dfg/640?wx_fmt=jpeg)

### cms 指纹管理

操作系统指纹、CMS 指纹等。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXH3PSA28jtDjHeA4icdRR6HKlF7ZQtPZUQ5flLZs6k4oZ4s0T9LCesRQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDX9Ha4AbjpeticoPVQITn1ibN6G8ySkoJxMWv4hWSeUXcVhOjfG3Nr1Wwg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXrT9zV2ouZL8z994iae9A79UcqtibgYxBqZ4kL794Zf8Q6NhdibPYxnCZw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXCQoCoBAvOL43BWHuS1ziaqHcq5OeL2dsJZTzV1RnX2OT5oS73A8ibKCg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXS4oVZCuAVw88v7wfdXsHHxeOoVrDicsatujYd5YMymC1IFw1SstIBicQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXLczARA6rSIAjtaMuVc8FddBibJZJbZ0Xj7rgic92bzeKhKfZDW0jBewA/640?wx_fmt=jpeg)

### 可视化隧道节点管理

使用隧道功能后，poc 获取控制权限或爆破成功，自动化传输隧道 agent，通过后台开关即可控制节点。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDX9zKkMouzKEKuhTic1CZD6lFGSj690VfcxSIR2w1SUanTiaOQqjNzzxfg/640?wx_fmt=jpeg)

### DNSLOG

直接生成域名即可。

### shellcode 自动化免杀

1.  采用国密算法加密的 shellcode，可过大部分杀软，满足后渗透的需求。
    
2.  如编译好的 exe 文件，也可以使用该功能进行混淆捆版。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXYiahE3uo3bUBOszAaFpCG7qVd3SV43BKYxlYRePariaM3FJpSX54uWRg/640?wx_fmt=jpeg)

### 提权辅助

windows 自动采集补丁信息，如当前非管理员权限，可自动化进行简单提权，同时也会列出可提权 exp，进行手工提权。同时也可粘贴其他地方的 systeminfo 进行提权查询。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LJgvpfzb7h9bOIdWGxlnmDXcELHHGNMTicj8EY6SwubMpMBTFgRrxtLyUUGqB8JIbKrzOX90Ieh0PA/640?wx_fmt=jpeg)

### 提交反馈

如有好的建议，以及发现 BUG。  
GitHub issue: https://github.com/inbug-team/InScan/issues

官网 (生成扫描器)： https://www.inbug.org