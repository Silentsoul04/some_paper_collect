\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/ai8IFbSlr7gkLRj8xNaESg)

**FRP 工具的使用**  

FRP 官方文档: https://gofrp.org/docs

### **一、FRP 工具的介绍**

#### **1\. 为什么需要内网穿透**

我们的物理机、服务器可能处于路由器后或者处于内网之中。如果我们想直接访问到这些设备（远程桌面、远程文件、SSH 等等），一般来说要通过一些转发或者 P2P(端到端) 组网软件的帮助。

其实，对于 FRP 穿透工具来说，它和端口转发有所不同，端口转发是只会进行单个端口的流量转发，但是这在渗透中往往是不行的，我们通过通过 FRP 进行内网的全流量的数据代理，像 FRP 可以代理全端口、全流量的数据，这样我们就可以使用 SocksCap 或者 Proxifier 等工具进行连接。

#### **2.FRP 介绍**

frp 是一个可用于内网穿透的高性能的反向代理应用，支持 TCP、UDP 协议，为 HTTP 和 HTTPS 应用协议提供了额外的能力，且尝试性支持了点对点穿透。frp 采用 go 语言开发。更多的人使用 frp 是为了进行反向代理，满足通过公网服务器访问处于内网的服务，如访问内网 web 服务，远程 ssh 内网服务器，远程控制内网 NAS 等，实现类似花生壳、ngrok 等功能。而对于内网渗透来讲，这种功能恰好能够满足我们进行内网渗透的流量转发。FRP 最大的一个特点是使用 SOCKS 代理，而 SOCKS 是加密通信的，类似于做了一个加密的隧道，可以把外网的流量，通过加密隧道穿透到内网。效果有些类似于 VPN。

#### **3\. 为什么使用 FRP**

通过在具有公网 IP 的节点上部署 frp 服务端，可以轻松地将内网服务穿透到公网，同时提供诸多专业的功能特性，这包括：

*   客户端服务端通信支持 TCP、KCP 以及 Websocket 等多种协议。
    
*   采用 TCP 连接流式复用，在单个连接间承载更多请求，节省连接建立时间。
    
*   代理组间的负载均衡。
    
*   端口复用，多个服务通过同一个服务端端口暴露。
    
*   多个原生支持的客户端插件 (静态文件查看，HTTP、SOCK5 代理等)，便于独立使用 frp 客户端完成某些工作。
    
*   高度扩展性的服务端插件系统，方便结合自身需求进行功能扩展。
    
*   服务端和客户端 UI 页面。
    

### **二、FRP 工具原理**

#### **1.FRP 实现原理**

**frp 主要由客户端 (frpc) 和服务端 (frps) 组成，服务端通常部署在具有公网 IP 的机器上，客户端通常部署在需要穿透的内网服务所在的机器上。**内网服务由于没有公网 IP，不能被非局域网内的其他用户访问。隐藏用户通过访问服务端的 frps，由 frp 负责根据请求的端口或其他信息将请求路由到对应的内网机器，从而实现通信。

#### **2.FRP 图示**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuD6RialQ8IG3iciccqM61oIwTIgLZzp1ZrV2HNjKHrcn1HgfCHmTgALiaicQ/640?wx_fmt=png)

如上图所示，在目标内网 (192.168.1.0 网段) 中，一共有多台内网服务器(使用三台进行简要说明)，其中内网 WEB 服务器和内网数据库服务器的端口被映射到 Nginx 反向代理服务器上，这样直接访问 Nginx 反向代理服务器的 IP 加上对应的端口即可访问到内网 WEB 服务器和内网数据库服务器。

现在存在这样的一种情况，由于内网 WEB 服务器的端口被映射到了公网 (也就是 Nginx 反向代理服务器的 80 端口上)，因此可以通过访问 Nginx 服务器的 80 端口就直接可以访问到内网的 WEB 服务器上的业务，如果发现 WEB 主机上存在漏洞，通过漏洞拿到了 WEB 服务器的 shell，注意这个 shell 并不是 Nginx 服务器的 shell，而是内网 WEB 服务器的 shell。

如果我们需要进行内网渗透，有两种思路，第一种是在内网 WEB 服务器上安装 nmap、masscan 这类工具进行扫描，第二种就是使用 frp 等工具进行代理，使用代理进行扫描。这里面就牵扯到两个问题，一个问题是如果拿下的这台主机是 windows 主机，但是在内网里面又发现了一台主机有 web 服务，这样怎么办？我们可以在这台主机上安装 Burp 和浏览器，进行抓包渗透，但是这个前提是你要可以连接 RDP，也就是说内网 WEB 服务器的 3389 也是映射到公网上的，可以直接连接进行渗透；还有一个问题是如果拿下的主机是 linux 主机呢？怎么在 linux 主机上安装 burp 呢？这显然是不合理的，但是我们可以在拿下的这台 linux 主机上开启 SOCKS 代理，然后在本地使用 SOCKS 代理去连接，但是显然比较麻烦。因此，很多人在内网渗透中可能会选择使用 FRP 内网穿透工具来进行内网中的全流量代理，FRP 是一个全流量代理，在本地可以使用 SocksCap、Proxifier 等工具进行连接。下面来进行说明。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuKialbzOHV8knYpuSIfibtbbmVXiadpTeeThuCdjoF4HArEp9ib6YcOsotw/640?wx_fmt=png)

首先，我们需要将 FRP 的服务端部署到公网 VPS 上，然后在我们拿下的那台内网主机上部署 FRP 的客户端，FRP 在 github 上有多种版本，有些是有配置文件的，这样的话就需要在客户端和服务器端分别配置两个文件，客户端的文件分别是 frpc 文件和 frpc.ini 文件；服务器端的文件分别是 frps 文件和 frps.ini 文件。还有不需要配置文件的，这是作者经过二次开发的文件

下载链接:

https://github.com/uknowsec/frpModify。

#### **3.FRP 工作原理介绍**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuGia5j0Rtlicmib1NV1SXK8pFRdrYibGJl9bLhQWxV7s0VJsGTcLKYq4sqA/640?wx_fmt=png)

(1): 首先启动 frpc，frpc 启动后会向 frps 注册，也就是内网 WEB 服务器会向 VPS 请求注册。

(2): 客户端请求 frps，也就是当我们的攻击机去访问 frps。

(3):frps 告知 frpc 有新请求，需要建立连接，也就是 VPS 告知内网 WEB 服务器，需要建立连接。

(4):frps 收到 frpc 的请求，建立新的连接，也就是 VPS 接收到了内网 WEB 服务器的请求，建立了新的连接。

(5):frps 吧 frpc 和攻击机的流量互相转发，将 frps 服务器当成流量中转站，也就是 VPS 将攻击机的流量转发给内网 WEB 服务器，把内网 WEB 服务器的流量转发给攻击机。

#### **4.FRP 配置文件**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuGzO24l000ARSagwC9VCPtiaiaxgRrnahykSiaNnZ3X835Srt69OcqYPDA/640?wx_fmt=png)

(1): 完整的服务器端配置文件

```
\# \[common\] 是必需的
\[common\]
# ipv6的文本地址或主机名必须括在方括号中
# 如"\[::1\]:80", "\[ipv6-host\]:http" 或 "\[ipv6-host%zone\]:80"
bind\_addr = 0.0.0.0
bind\_port = 7000

# udp nat 穿透端口
bind\_udp\_port = 7001

# 用于 kcp 协议 的 udp 端口，可以与 "bind\_port" 相同
# 如果此项不配置, 服务端的 kcp 将不会启用 
kcp\_bind\_port = 7000

# 指定代理将侦听哪个地址，默认值与 bind\_addr 相同
# proxy\_bind\_addr = 127.0.0.1

# 如果要支持虚拟主机，必须设置用于侦听的 http 端口（非必需项）
# 提示：http端口和https端口可以与 bind\_port 相同
vhost\_http\_port = 80
vhost\_https\_port = 443

# 虚拟 http 服务器的响应头超时时间（秒），默认值为60s
# vhost\_http\_timeout = 60

# 设置 dashboard\_addr 和 dashboard\_port 用于查看 frps 仪表盘
# dashboard\_addr 默认值与 bind\_addr 相同
# 只有 dashboard\_port 被设定，仪表盘才能生效
dashboard\_addr = 0.0.0.0
dashboard\_port = 7500

# 设置仪表盘用户密码，用于基础认证保护，默认为 admin/admin
dashboard\_user = admin
dashboard\_pwd = admin

# 仪表板资产目录(仅用于 debug 模式下)
# assets\_dir = ./static
# 控制台或真实日志文件路径，如./frps.log
log\_file = ./frps.log

# 日志级别，分为trace（跟踪）、debug（调试）、info（信息）、warn（警告）、error（错误） 
log\_level = info

# 最大日志记录天数
log\_max\_days = 3

# 认证 token
token = 12345678

# 心跳配置, 不建议对默认值进行修改
# heartbeat\_timeout 默认值为 90
# heartbeat\_timeout = 90

# 允许 frpc(客户端) 绑定的端口，不设置的情况下没有限制
allow\_ports = 2000-3000,3001,3003,4000-50000

# 如果超过最大值，每个代理中的 pool\_count 将更改为 max\_pool\_count
max\_pool\_count = 5

# 每个客户端可以使用最大端口数，默认值为0，表示没有限制
max\_ports\_per\_client = 0

# 如果 subdomain\_host 不为空, 可以在客户端配置文件中设置 子域名类型为 http 还是 https
# 当子域名为 test 时, 用于路由的主机为 test.frps.com
subdomain\_host = frps.com

# 是否使用 tcp 流多路复用，默认值为 true
tcp\_mux = true

# 对 http 请求设置自定义 404 页面
# custom\_404\_page = /path/to/404.html
```

(2): 完整的客户端配置文件  

```
\# \[common\] 是必需的
\[common\]
# ipv6的文本地址或主机名必须括在方括号中
# 如"\[::1\]:80", "\[ipv6-host\]:http" 或 "\[ipv6-host%zone\]:80"
server\_addr = 0.0.0.0
server\_port = 7000

# 如果要通过 http 代理或 socks5 代理连接 frps，可以在此处或全局代理中设置 http\_proxy
# 只支持 tcp协议
# http\_proxy = http://user:passwd@192.168.1.128:8080
# http\_proxy = socks5://user:passwd@192.168.1.128:1080

# 控制台或真实日志文件路径，如./frps.log
log\_file = ./frpc.log

# 日志级别，分为trace（跟踪）、debug（调试）、info（信息）、warn（警告）、error（错误）
log\_level = info

# 最大日志记录天数
log\_max\_days = 3

# 认证 token
token = 12345678

# 设置能够通过 http api 控制客户端操作的管理地址
admin\_addr = 127.0.0.1
admin\_port = 7400
admin\_user = admin
admin\_pwd = admin

# 将提前建立连接，默认值为 0
pool\_count = 5

# 是否使用 tcp 流多路复用，默认值为 true，必需与服务端相同
tcp\_mux = true

# 在此处设置用户名后，代理名称将设置为  {用户名}.{代理名}
user = your\_name

# 决定第一次登录失败时是否退出程序，否则继续重新登录到 frps
# 默认为 true
login\_fail\_exit = true

# 用于连接到服务器的通信协议
# 目前支持 tcp/kcp/websocket, 默认 tcp
protocol = tcp

# 如果 tls\_enable 为 true, frpc 将会通过 tls 连接 frps
tls\_enable = true

# 指定 DNS 服务器
# dns\_server = 8.8.8.8

# 代理名, 使用 ',' 分隔
# 默认为空, 表示全部代理
# start = ssh,dns

# 心跳配置, 不建议对默认值进行修改
# heartbeat\_interval 默认为 10 heartbeat\_timeout 默认为 90
# heartbeat\_interval = 30
# heartbeat\_timeout = 90

# 'ssh' 是一个特殊代理名称
\[ssh\]
# 协议 tcp | udp | http | https | stcp | xtcp, 默认 tcp
type = tcp
local\_ip = 127.0.0.1
local\_port = 22
# 是否加密, 默认为 false
use\_encryption = false
# 是否压缩
use\_compression = false
# 服务端端口
remote\_port = 6001
# frps 将为同一组中的代理进行负载平衡连接
group = test\_group
# 组应该有相同的组密钥
group\_key = 123456
# 为后端服务开启健康检查, 目前支持 'tcp' 和 'http' 
# frpc 将连接本地服务的端口以检测其健康状态
health\_check\_type = tcp
# 健康检查连接超时
health\_check\_timeout\_s = 3
# 连续 3 次失败, 代理将会从服务端中被移除
health\_check\_max\_failed = 3
# 健康检查时间间隔
health\_check\_interval\_s = 10

\[ssh\_random\]
type = tcp
local\_ip = 127.0.0.1
local\_port = 22
# 如果 remote\_port 为 0 ,frps 将为您分配一个随机端口
remote\_port = 0

# 如果要暴露多个端口, 在区块名称前添加 'range:' 前缀
# frpc 将会生成多个代理，如 'tcp\_port\_6010', 'tcp\_port\_6011'
\[range:tcp\_port\]
type = tcp
local\_ip = 127.0.0.1
local\_port = 6010-6020,6022,6024-6028
remote\_port = 6010-6020,6022,6024-6028
use\_encryption = false
use\_compression = false

\[dns\]
type = udp
local\_ip = 114.114.114.114
local\_port = 53
remote\_port = 6002
use\_encryption = false
use\_compression = false

\[range:udp\_port\]
type = udp
local\_ip = 127.0.0.1
local\_port = 6010-6020
remote\_port = 6010-6020
use\_encryption = false
use\_compression = false

# 将域名解析到 \[server\_addr\] 可以使用 http://web01.yourdomain.com 访问 web01
\[web01\]
type = http
local\_ip = 127.0.0.1
local\_port = 80
use\_encryption = false
use\_compression = true
# http 协议认证
http\_user = admin
http\_pwd = admin
# 如果服务端域名为 frps.com, 可以通过 http://test.frps.com 来访问 \[web01\] 
subdomain = web01
custom\_domains = web02.yourdomain.com
# locations 仅可用于HTTP类型
locations = /,/pic
host\_header\_rewrite = example.com
# params with prefix "header\_" will be used to update http request headers
header\_X-From-Where = frp
health\_check\_type = http
# frpc 将会发送一个 GET http 请求 '/status' 来定位http服务
# http 服务返回 2xx 状态码时即为存活
health\_check\_url = /status
health\_check\_interval\_s = 10
health\_check\_max\_failed = 3
health\_check\_timeout\_s = 3

\[web02\]
type = https
local\_ip = 127.0.0.1
local\_port = 8000
use\_encryption = false
use\_compression = false
subdomain = web01
custom\_domains = web02.yourdomain.com
# v1 或 v2 或 空
proxy\_protocol\_version = v2

\[plugin\_unix\_domain\_socket\]
type = tcp
remote\_port = 6003
plugin = unix\_domain\_socket
plugin\_unix\_path = /var/run/docker.sock

\[plugin\_http\_proxy\]
type = tcp
remote\_port = 6004
plugin = http\_proxy
plugin\_http\_user = abc
plugin\_http\_passwd = abc

\[plugin\_socks5\]
type = tcp
remote\_port = 6005
plugin = socks5
plugin\_user = abc
plugin\_passwd = abc

\[plugin\_static\_file\]
type = tcp
remote\_port = 6006
plugin = static\_file
plugin\_local\_path = /var/www/blog
plugin\_strip\_prefix = static
plugin\_http\_user = abc
plugin\_http\_passwd = abc

\[plugin\_https2http\]
type = https
custom\_domains = test.yourdomain.com
plugin = https2http
plugin\_local\_addr = 127.0.0.1:80
plugin\_crt\_path = ./server.crt
plugin\_key\_path = ./server.key
plugin\_host\_header\_rewrite = 127.0.0.1

\[secret\_tcp\]
# 如果类型为 secret tcp, remote\_port 将失效
type = stcp
# sk 用来进行访客认证
sk = abcdefg
local\_ip = 127.0.0.1
local\_port = 22
use\_encryption = false
use\_compression = false

# 访客端及服务端的用户名应该相同
\[secret\_tcp\_visitor\]
# frpc role visitor -> frps -> frpc role server
role = visitor
type = stcp
# 要访问的服务器名称
server\_name = secret\_tcp
sk = abcdefg
# 将此地址连接到访客 stcp 服务器
bind\_addr = 127.0.0.1
bind\_port = 9000
use\_encryption = false
use\_compression = false

\[p2p\_tcp\]
type = xtcp
sk = abcdefg
local\_ip = 127.0.0.1
local\_port = 22
use\_encryption = false
use\_compression = false

\[p2p\_tcp\_visitor\]
role = visitor
type = xtcp
server\_name = p2p\_tcp
sk = abcdefg
bind\_addr = 127.0.0.1
bind\_port = 9001
use\_encryption = false
use\_compression = false
```

(3): 客户端和服务器端部分配置文件  

这个是下载下来的配置文件，主要看一下 frpc.ini 和 frps.ini 这两个配置文件

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuIP5eJ15Xp9XRq2RZLf4I30qoUMQ8oica3zRs4ge3iacibGGhwreCKficcg/640?wx_fmt=png)

介绍一下这个配置文件的参数

frps.ini 的配置文件如下:

```
\[common\]
bind\_port = 7000
```

\[common\] 部分是必须有的配置，其中 bind\_port 是自己设定的 frp 服务端端口，用于和 frp 服务器端进行通信。

token 用于验证连接，只有服务端和客户端 token 相同的时候才能正常访问。如果不使用 token，那么所有人都可以直接连接上，所以我建议大家在使用的时候还是把 token 加上。  

frpc.ini 配置文件如下:

```
\[common\]
server\_addr = VPS IP
server\_port = 7000

\[ssh\]
type = tcp
local\_ip = 127.0.0.1
local\_port = 22
remote\_port = 6000
```

\[common\] 部分是必须有的配置

其中 server addr 是 VPS 的 IP

server port 是用来和 frp 客户端通信的端口，必须和 frp 服务器端端口一致，默认是 7000。

\[ssh\] 可以修改成任意名称，local ip 是本地的 IP，local port 是本地进行监听的端口，remote port 是访问 VPS 时需要使用的端口，也就是说访问到这个 remote port 端口时才能访问到内网。  

### **三、FRP 使用场景**

#### **1.windows 主机**

*   第一种情况 (3389 出网)
    

当我们拿下的这台 windows 主机开启 3389 端口时，那就比较好办了，我们可以直接拿到管理员 hash 解密或者新建用户等直接进行连接，这样我们可以在该主机上执行安装 goby 等常用的扫描工具，但是这样可能会被管理员发现；当然，我们也可以使用 frp 进行内网代理，在物理机上通过代理进行扫描。

*   第二种情况 (3389 不出网)
    

当我们拿下的这台内网主机是 windows 时，也就是内网 WEB 服务器是 windows 时，如果这台主机的 3389 端口没有映射到 Nginx 服务器上，那么，我们就不能使用远程桌面进行连接，如果需要扫描内网最好的办法就是使用 frp 等代理工具进行全流量代理，然后使用 SocksCap 或 Proxifier 等工具进行代理。

#### **2.Linux 主机**

当我们拿下的主机是 linux 主机的时候，我们想要在该主机上安装一些扫描工具，可以使用命令行安装 nmap、masscan 等工具，但是相对来说容易被发现，因此也可以使用 frp 来进行代理，并且也不能进行远程桌面，因此，使用 frp 是相对来说比较好的方法。

### **四、FRP 工具的使用**

#### **1.FRP 下载**

FRP 使用 Golang 编写，可以直接下载相应的文件。

有配置文件:

https://github.com/fatedier/frp/releases

不需要配置文件:

https://github.com/uknowsec/frpModify

#### **2\. 部署**

解压缩下载的压缩包，将其中的 frpc 拷贝到内网服务所在的机器上，将 frps 拷贝到具有公网 IP 的机器上，放置在任意目录。

#### **3\. 常用命令**

(1): 启动 frp 客户端

```
./frpc -c ./frpc.ini
```

(2): 后台启动 frp 客户端  

```
nohup ./frpc -c ./frpc.ini &
```

(3): 启动 frp 服务器端  

```
./frps -c ./frps.ini
```

(4): 后台启动 frp 服务器端  

```
nohub ./frps -c ./frps.ini &
```

(5): 控制台  

```
wget https://github.com/fatedier/frp/releases/download/v0.9.3/frp\_0.34.1\_linux\_386.tar.gz
tar xzf frp\_0.34.1\_linux\_386.tar.gz
mv frp\_0.34.1\_linux\_386 frp
cd frp
vi frp.ini
```

\# 添加如下内容  

```
dashboard\_port = 7500
dashboard\_user = admin
dashboard\_pwd = admin
```

启动 frps

```
./frps -c ./frps.ini
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuESC28Rbn5ww3AnPG5P4ib9kUQT0UpAx7vxbRI8lkPibiaTEiaIcIrPShWw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuTKxdl1jA0EWJSTJq9AytLXztK93lTyDxoo9qPL0TRh9ZgCQl7kHUew/640?wx_fmt=png)

但是这个功能一般没不太会用到。

### **五、FRP 工具使用案例**

#### **1\. 内网环境搭建**

VPS(FRP 服务器端):116.62.106.123

攻击机 (物理机 win10):115.171.90.105

内网 WEB 服务器 (虚拟机 win7):192.168.223.151

**注意: 内网 WEB 服务器必须要出网，这样才能连接 VPS 的 7000 端口。**

#### **2\. 使用有配置文件的 FRP**

(1): 在 VPS 上下载 FRP 服务器端

这个类型的 FRP

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuRdGrp3KzVfDBM0ibYQrH0ExnzC4vM7pGZdtR2XrGe1RGKcicznz6DgvQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxugciaricdfMia538rUrj75FicAKOxR8GBrVKTHaVjciaRaL6WfZRcFobxybg/640?wx_fmt=png)

(2): 在内网服 WEB 务器 (192.168.223.151) 上下载 frp 客户端

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxu2culc0iasctA5ChyhrXZMwlLvIzEq5aDxby4cfGiaiclJl9riagC1cXNlA/640?wx_fmt=png)

(3): 修改 FRP 配置文件如下所示

```
\[common\]
server\_addr = 116.62.106.123
server\_port = 7000

\[3389\]
type = tcp
local\_ip = 127.0.0.1
local\_port = 3389
remote\_port = 6000
plugin = socks5
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuaPJ2ibYXQO4KT9c9L84AflT7utvSwb2PEZSpN0dFXSDWYU5tuJ5VKqw/640?wx_fmt=png)  

注意: 说两个需要注意的地方:

第一个: local\_port 这块的端口经过测试，可以设置成任意端口，目前在 windows 主机上测试了 3389、22222 和 22 端口都成功了。

第二个: 需要添加上 plugin = socks5 这一行，因为经过测试，如果不加这一行，使用 SocksCap 会显示协商代理认证方式失败，因此需要加上这一行。如下图:

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxubSibcGYDxAJfQuz5sSzoBRFZ5MuLh8RoMC2hRkujQLQXXFMnuJq74yw/640?wx_fmt=png)

(4): 先在 VPS(FRP 服务器端) 执行如下命令

./frps -c frps.ini

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuqb0zQD2NbobmnhJBGcbwfyonnB3tpM9WiaHUu9Dj9FKIS8CqzstV9tA/640?wx_fmt=png)

(5): 然后在 FRP 客户端执行如下命令

frpc -c ./frpc.ini

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxucPYbQYiayuibWVr9SZplNSA9JqdeFiczte3UwHKG8CTSFj4diaWwxpibxjg/640?wx_fmt=png)

(6): 设置浏览器代理为 VPS 的 6000 端口

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuCb8JAibrFHibNPZbSYeqOh8Zia0JvxA55W3Y9Uk6Vs9uD7zibT4VKptD3w/640?wx_fmt=png)

(7): 成功访问到内网主机

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuulbyia8zKibrMJbuv1F8B7kPySNNYOiaVNdtTUJdoaJvycTB7BkIlZUZw/640?wx_fmt=png)

#### 3\. 使用没有配置文件的 FRP

https://github.com/uknowsec/frpModify

(1): 在 VPS 上上传 frp 服务器端

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuWscCph1wHzric8xWwdBm9Tgmd1pXyS7CUccojWic5qNWKLVXug7icqa4Q/640?wx_fmt=png)

(2): 在内网 WEB 服务器 (192.168.223.151) 上上传 frp 客户端

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuWReodo79gDOJ4LEWzHRBibw1NTsxxw7CibHHxic3fBnWSKMiavSunPwElw/640?wx_fmt=png)

(3): 在 VPS 上添加配置文件

配置文件名称如下:

```
frps.ini
\[common\]
bind\_port = 2333
token = uknowsec
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxugdsKe16wrEG9ZmPrWWVSY4HN6RfXGpylhOpgf70z6j9snyC1HOfgBA/640?wx_fmt=png)

(4): 在 VPS 上执行如下命令

```
./frps -c frps.ini
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuuwwUQDh9bcnZw81Vqj04eemIlhBMh5hjAP4MdJRxEeqYKgm7dKvlxQ/640?wx_fmt=png)  

(5): 在内网 WEB 服务器上执行如下命令

```
frpc.exe -t VPSIP -p 2333
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxujianGDRbJmvC3jf9TsmefeRa867ODnw4orIMFANUPnh7S2ufFxbcMSw/640?wx_fmt=png)  

这样 FRP 就配置成功了。

#### **4\. 使用工具通过 FRP 访问内网**

为了证明 FRP 的联通性，下面的操作都是在另一台电脑上进行操作的。

(1): 使用浏览器设置代理访问

①: 设置浏览器代理

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxu6DoXcGuBHibsdQTgjcpoqiaDBTtJcxaIlPz54PIiaXSQIVcriad3sHkficg/640?wx_fmt=png)

②: 使用代理进行访问

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuWWop6MOmWj1Pe4IqicibVsrc1asibAI2ziaEfFNgiaScYsctvOjibXc7MRaA/640?wx_fmt=png)

(2): 使用 SocksCap 进行访问

①: 设置 SocksCap

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuWobJJ45icGj2dKjribmmamZHicovhvubILk5hHcb9SvKXL8e7Fx1mdUFA/640?wx_fmt=png)

②: 使用 SocksCap 进行访问

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuUt2hC6tbf1RQR9R41ns77C5M8ib7LeGnQRlSgargCJwWh3jtTROLI1w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxu0368DPvwASS02zRxVeuEImE7VAaeOI95licsYZGxibYWWLFWeG3Q2dGA/640?wx_fmt=png)

(3): 使用 Proxifier 进行访问

①: 设置 Proxifier

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxu6cnWMfIt9blGCYQhwBp3j0Pvdc3ceMMrwEER4OxYqY5nZfiakoMXgicg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxu3dWu1dP4bSMN50BmwiauL7N78V0b8iaRZkX74mEQq0xVUbAwegURXKsA/640?wx_fmt=png)

②: 所有的应用程序都会走 Proxifier 代理

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibWJgkNvmvYe4Lxnib4gzyxuuEeEJrxiaiaA2gOBrMC54Xib2JT9v2cegSMrEmT3r86hWBGoVYWibjHcyw/640?wx_fmt=png)

**(4): 三种方式的优缺点:**

①: 使用浏览器代理比较方便，设置简单，但是只能访问内网中的 web 服务。

②: 使用 SocksCap 时，需要使用哪种工具就把该工具导入进去，相对来说比较方便，但是如果需要使用多个工具，每个导入会比较麻烦。

③: 使用 Proxifier 时，所有的应用都会走代理，这样就会导致有些应用你不想让它走代理也没有办法做到。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)**推荐阅读：****[内网渗透 | 常用的内网穿透工具使用](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247492155&idx=1&sn=3e0a217a784eb250cad2f2f41a488f68&chksm=ec1cb704db6b3e12f9840f33deccebc488f0369cd5e0cd46286c89fbac39f41b52e2436af845&scene=21#wechat_redirect)**

[![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou80h6Jor7Py4sKIwfiaowozsMP0Yjn9RcoJAmPMKa5hQVczeXoDxIic2QaZYKKrLDlJFT5v6EpREmjg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247492102&idx=1&sn=aa09a4f38ae21b73a1d3a938d97aae20&chksm=ec1cb739db6b3e2f7d7edc43d338e9f2dc4563edc768a34fb4214618f5107ecfc89f9d7b802c&scene=21#wechat_redirect)

**点赞 在看 转发**

原创投稿作者：想走安全的小白

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)