\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/lGcGNlvhcvbvYXTuL8VQBQ)

**点击蓝字**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

**关注我们** 

  
![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

简介  

![](https://mmbiz.qpic.cn/mmbiz_jpg/F3c71CW7CseicWswxThGgktbF9PibicOuNkwLXiamQIic2StQ9nONia3icMLEG60vbW0HyaZic09FOH1chSJgMtiaqicueLg/640?wx_fmt=jpeg)

fofa，钟馗之眼，shodan 等等一系列的公网设备搜索引擎，其中 fofa 和 shodan 使用的最多，本文就来整理一些 shodan 的搜索语法

Shodan：www.shodan.io

Ps：均来自互联网搜集整理

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

工业控制系统

三星电子广告牌

"Server: Prismview Player"

加油站泵控制器

"in-tank inventory" port:10001

自动车牌阅读器

P372 "ANPR enabled"

交通信号灯控制器 / 红光摄像机

mikrotik streetlight

美国的投票机

"voter system serial" country:US

运营思科合法拦截窃听的电信公司

"Cisco IOS" "ADVIPSERVICESK9\_LI-M"  

Ps:  

思科在 RFC 3924 中概述的窃听机制：合法拦截是指合法授权的拦截和监视拦截对象的通信。术语 “拦截对象” 指其通信和 / 或拦截相关信息（IRI）已被合法授权拦截并交付给某些机构的电信服务的订户。

监狱专用电话

"\[2J\[H Encartele Confidential"

Tesla PowerPack 充电状态

http.title:"Tesla PowerPack System" http.component:"d3" -ga3ca4f2

电动汽车充电器

"Server: gSOAP/2.8" "Content-Length: 583"

海事卫星

Shodan 制作了一个非常漂亮的 Ship Tracker，它还可以实时绘制船舶位置地图！

"Cobham SATCOM" OR ("Sailor" "VSAT")

潜艇任务控制仪表板

title:"Slocum Fleet Mission Control"

CAREL PlantVisor 制冷机组  
"Server: CarelDataServer" "200 Document follows"

Nordex 风力发电机场  
http.title:"Nordex Control" "Windows 2000 5.0 x86" "Jetty/3.1 (JSP 1.1; Servlet 2.2; java 1.6.0\_14)"

C4 Max 商用车 GPS 追踪器  
"\[1m\[35mWelcome on console"  
示例：C4 Max Vehicle GPS

DICOM 医用 X 射线机  
幸运的是，默认情况下是安全的，但是这些 1,700 多台计算机仍然没有互联网上的业务。  
"DICOM Server Response" port:104

GaugeTech 电表  
"Server: EIG Embedded Web Server" "200 Document follows"  
示例：GaugeTech 电表

西门子工业自动化  
"Siemens, SIMATIC" port:161

西门子 HVAC 控制器  
"Server: Microsoft-WinCE" "Content-Length: 12581"

门 / 锁门禁控制器  
"HID VertX" port:4070

铁路管理  
"log off" "select the appropriate"  

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

远程桌面

未受保护的 VNC  
"authentication disabled" "RFB 003.008"  
顺便说一下，Shodan Images 是一个很棒的辅助工具，可以浏览屏幕截图！🔎→  
Windows RDP  
Windows 辅助登录屏幕可保护 99.99％的安全。  
"\\x03\\x00\\x00\\x0b\\x06\\xd0\\x00\\x00\\x124\\x00"

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

网络基础设施

编织范围仪表板  
Kubernetes Pod 和 Docker 容器内部的命令行访问以及整个基础架构的实时可视化 / 监视。  
title:"Weave Scope" http.favicon.hash:567176827  
示例：编织范围仪表板  
MongoDB  
默认情况下，旧版本不安全。非常吓人。  
"MongoDB Server Information" port:27017 -authentication  
示例：MongoDB  
Mongo Express Web GUI  
就像臭名昭著的 phpMyAdmin 一样，但适用于 MongoDB。  
"Set-Cookie: mongo-express=" "200 OK"  
示例：Mongo Express GUI  
詹金斯 CI  
"X-Jenkins" "Set-Cookie: JSESSIONID" http.title:"Dashboard"  
例如：Jenkins CI  
Docker API  
"Docker Containers:" port:2375  
Docker 私有注册中心  
"Docker-Distribution-Api-Version: registry" "200 OK" -gitlab  
Pi-hole 开放 DNS 服务器  
"dnsmasq-pi-hole" "Recursion: enabled"  
已经 root 通过 Telnet 登录  
"root@" port:23 -login -password -name -Session  
Android 根网桥  
Google 愚蠢的断裂式更新方法的切线结果。🙄 更多信息在这里。  
"Android Debug Bridge" "Device" port:5555  
Lantronix 串行到以太网适配器泄漏 Telnet 密码  
Lantronix password port:30718 -secured  
Citrix 虚拟应用程序  
"Citrix Applications:" port:1604  
示例：Citrix 虚拟应用程序  
思科智能安装  
易受攻击（有点 “设计使然”，但特别是暴露时）。  
"smart install client active"  
PBX IP 电话网关  
PBX "gateway console" -password port:23  
Polycom 视频会议  
http.title:"- Polycom" "Server: lighttpd"  
Telnet 配置：  
"Polycom Command Shell" -failed port:23  
示例：Polycom 视频会议  
Bomgar 帮助台门户  
"Server: Bomgar" "200 OK"  
英特尔主动管理 CVE-2017-5689  
"Intel(R) Active Management Technology" port:623,664,16992,16993,16994,16995  
HP iLO 4 CVE-2017-12542  
HP-ILO-4 !"HP-ILO-4/2.53" !"HP-ILO-4/2.54" !"HP-ILO-4/2.55" !"HP-ILO-4/2.60" !"HP-ILO-4/2.61" !"HP-ILO-4/2.62" !"HP-iLO-4/2.70" port:1900  
Outlook Web Access：  
Exchange 2007  
"x-owa-version" "IE=EmulateIE7" "Server: Microsoft-IIS/7.0"  
示例：用于 Exchange 2007 的 OWA  
Exchange 2010  
"x-owa-version" "IE=EmulateIE7" http.favicon.hash:442749392  
示例：用于 Exchange 2010 的 OWA  
Exchange 2013/2016  
"X-AspNet-Version" http.title:"Outlook" -"x-owa-version"  
示例：用于 Exchange 2013/2016 的 OWA  
Lync / Skype for Business  
"X-MS-Server-Fqdn"

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

网络附加存储（NAS）

SMB（Samba）文件共享  
产生约 500,000 个结果… 通过添加 “文档” 或“视频”等来缩小范围。  
"Authentication: disabled" port:445  
特别是域控制器：  
"Authentication: disabled" NETLOGON SYSVOL -unix port:445  
关于 QuickBooks 文件的默认网络共享：  
"Authentication: disabled" "Shared this folder to access QuickBooks files OverNetwork" -unix port:445  
具有匿名登录的 FTP 服务器  
"220" "230 Login successful." port:21  
艾美加 / LenovoEMC NAS 驱动器  
"Set-Cookie: iomega=" -"manage/login.html" -http.title:"Log In"  
示例：Iomega / LenovoEMC NAS 驱动器  
布法罗 TeraStation NAS 驱动器  
Redirecting sencha port:9000  
示例：Buffalo TeraStation NAS 驱动器  
罗技媒体服务器  
"Server: Logitech Media Server" "200 OK"  
示例：Logitech 媒体服务器  
Plex 媒体服务器  
"X-Plex-Protocol" "200 OK" port:32400  
Tautulli / PlexPy 仪表板  
"CherryPy/5.1.0" "/home"

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

网路摄影机

偏航摄像机  
"Server: yawcam" "Mime-Type: text/html"  
webcamXP / webcam7  
("webcam 7" OR "webcamXP") http.component:"mootools" -401  
Android IP 网络摄像头服务器  
"Server: IP Webcam Server" "200 OK"  
安全 DVR  
html:"DVR\_H264 ActiveX"

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

打印机和复印机

HP 打印机  
"Serial Number:" "Built:" "Server: HP HTTP"  
示例：HP 打印机  
施乐复印机 / 打印机  
ssl:"Xerox Generic Root"  
示例：施乐复印机 / 打印机  
爱普生打印机  
"SERVER: EPSON\_Linux UPnP" "200 OK"  
"Server: EPSON-HTTP" "200 OK"  
示例：爱普生打印机  
佳能打印机  
"Server: KS\_HTTP" "200 OK"  
"Server: CANON HTTP Server"

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

家用设备

雅马哈音响  
"Server: AV\_Receiver" "HTTP/1.1 406"  
示例：雅马哈立体声  
Apple AirPlay 接收器  
Apple TV，HomePods 等  
"\\x08\_airplay" port:5353  
Chromecast / 智能电视  
"Chromecast:" port:8008  
快思聪智能家居控制器  
"Model: PYNG-HUB"

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/MVPvEL7Qg0Egj76lSPicXbCTgBVIialdvwWxbm21OZytaf8P6xK86NglUjbFn8agwwLkdNhBNdSRvMLtiaXKjEj7A/640?wx_fmt=gif)

**杂项  
**

OctoPrint 3D 打印机控制器  
title:"OctoPrint" -title:"Login" http.favicon.hash:1307375944  
示例：OctoPrint 3D 打印机  
以太网矿工  
"ETH - Total speed"  
示例：以太网矿工  
Apache 目录列表  
.pem 用任何扩展名或文件名代替 phpinfo.php。  
http.title:"Index of /" http.html:".pem"  
WordPress 配置错误  
wp-config.php 包含数据库凭据的公开文件。  
http.html:"\* The wp-config.php creation script uses this file"  
Minecraft 服务器太多  
"Minecraft Server" "protocol 340" port:25565  
从字面上看朝鲜的一切🇰🇵  
net:175.45.176.0/22,210.52.109.0/24,77.94.35.0/24  
TCP 每日报价  
端口 17（RFC 865）具有奇异的历史……  
port:17 product:"Windows qotd"  
找工作！‍💼  
"X-Recruiting:"

![](https://mmbiz.qpic.cn/mmbiz_jpg/F3c71CW7CseicWswxThGgktbF9PibicOuNkd0E4VJsuTEPaTvsZsib9N1r9dRJiaKHhnk0lXU18jA8XpwFOiaTftbVKg/640?wx_fmt=jpeg)

**扫二维码｜关注我们**

**公众号：信安小屋**