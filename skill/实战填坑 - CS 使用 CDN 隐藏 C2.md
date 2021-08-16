> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/B30Unfh5yAN4A151P1gsMQ)

**CS 的基础用法、修改端口、密码的教程网上很多，此处不再赘述。**

但在搭建域名 + CDN 隐藏版 c2 时楼主遇到了不少的坑，在这里顺着搭建的思路慢慢把踩的坑填上。

**1.CS 证书特征配置**
===============

Cobalt Strike 是一款美国 Red Team 开发的渗透测试神器，常被业界人称为 CS。

**1.1 去除证书特征：基于 keytool 生成自签证书**
--------------------------------

用 JDK 自带的 keytool 证书工具即可生成新证书：

_keytool 命令:_

```
-certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令
```

例如：

**国内 baidu**

```
keytool -keystore cobaltStrike.store -storepass 123456 -keypass 123456 -genkey -keyalg RSA -alias baidu.com -dname "CN=ZhongGuo, OU=CC, O=CCSEC, L=BeiJing, ST=ChaoYang, C=CN"
```

**国外 gmail：**  

```
keytool -keystore cobaltstrike.store -storepass 123456 -keypass 123456 -genkey -keyalg RSA -alias gmail.com -dname "CN=gmail.com, OU=Google Mail, O=Google GMail, L=Mountain View, ST=CA, C=US"
```

（Windows 版也可使用 java 安装目录下自带工具 <JAVA_HOME>\bin\keytool.exe）

然后使用 keytool 工具可查看生成的证书：

```
keytool -list -v -keystore cobaltstrike.store
```

**其中的坑：**  

要么生成 cobaltstrike.store 替换默认位置对应文件，要么在 teamserver 启动文件中指定，例如生成 baidu.store，就要修改 teamserver 为：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w05loQDZibdu5iaibic4DiaDPbuQSd9ibLCWopFwarvQ8Q9vW1Xp3iaBNSYAfXQ/640?wx_fmt=png)

**1.2 去除证书特征：基于 openssl 生成域名证书**
--------------------------------

这里有两个思路，一是申请域名后使用 certbot 生成对应证书；二是申请域名后修改 ns 记录，由托管服务商签发。

这里都需要申请域名，可**百度 freenom 申请域名的教程**（楼主申请失败了，无法接收邮件，使用插件也不行，所以算是一个坑）。

**填坑方式: 推荐** **https://www.namesilo.com****，**申请个冷门的也并不贵，才几块钱就可以用一年，还可微信支付。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0iaezK1p8xdvULRWKLeUOicsiax3HVh0ZicsayYdZ8pjFvJaaWiaEX6CGsBA/640?wx_fmt=png)

### 证书签发思路一 certbot：

假如你申请域名为：+++.tk，那么在 vps 上安装 certbot ，然后生成证书：

```
certbot certonly  -d +++.tk  -d *.+++.tk --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0AsDP0FNzXicg8QXLnKdWgzAS41aTAQxTXBW9PfFA9FE3oiby4yEAibwcA/640?wx_fmt=png)

需要你在 ns 服务商处添加两条 txt 记录。以 namesilo 为例

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0rv6t2V35GibHzC6BDUeLjvPSEkWGuTDUdfjibpuqW5SHnHdDhXVoMOdg/640?wx_fmt=png)

选择 txt 记录插入即可。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w01icT5sxo9Ct8G1B4iaZl4MQQa0J1icRoia7kPU6JOuTdJtDBu8tRqfHvew/640?wx_fmt=png)

freenome 一样道理，而后会让你插入第二条，确认后等待一会，生效后回车确认，即可在当前目录生成域名证书：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0u2VMcIdlGWiaBtxpjeYrQy39mf3lezwj5HgLJRD7GsPxVDPEosYiaEdA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0M6IzuGRqYQLNO6UsYwpULAZtPdn9nG2DWicEgkGy6agcz6d55c8SIKA/640?wx_fmt=png)

**这里的坑：**两次添加 txt 记录后需要等待一点时间才能解析成功，可另外开启 bash 使用 dig 命令：

```
dig -t txt _acme-challenge.+++.tk  @8.8.8.8
```

测试是否成功，成功获取 txt 内容再在 certbot 点击回车，没有 dig 可使用 yum install bind-utils 命令安装，否则生成失败还要重新认证，重新添加 TXT 记录。  

然后基于 openssl 生成为 p12 文件

```
openssl pkcs12 -export -in ./fullchain.pem -inkey ./privkey.pem -out +++.tk.p12 -name +++.tk -passout pass:123456
```

最后使用 keytool 生成 store：  

```
keytool -importkeystore -deststorepass 123456 -destkeypass 123456 -destkeystore +++.tk.store -srckeystore +++.tk.p12 -srcstoretype PKCS12 -srcstorepass 123456 -alias +++.tk
```

### 证书签发思路二 cloudflare：

申请域名后可在 cloudflare 申请免费账户，更改 NS 服务器地址，托管域名。然后在 cloudflare 设置即可：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0A4icEJhacQmuFMHCPwyLSL8OfKpMgsOYKcWicwOstJlxrrnGgvbbTg3g/640?wx_fmt=png)

下一步就可以从 cloudflare 一键导出证书：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0Ftiachu2fO8hrJP9TtotkDe845FojoIXfJDSWsGNjgboibmBlICRImVw/640?wx_fmt=png)

依然是使用 openssl 生成 p12，然后 store 文件，具体操作参考上一个思路。

**2. 服务器特征配置**
==============

**2.1 隐藏服务器：CDN 加速**
--------------------

在 cloudflarr 注册域名后，将 NS 记录指向 alice.ns.cloudflare.com 和 chase.ns.cloudflare.com 即可选择使用 cdn 加速即可开启。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0NXVDuN84vMn4ic3GiaOCclDnOYP52UbIHichlTicOM4B5pbRtgteAfxFeQ/640?wx_fmt=png)

本地使用 ping 测试，为 cdn 的 ip，而非你在域名服务商登记的真实 ip 就达到目的了：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0JM9BeiaiaohOasecc8XZ3e7RDrJLuwNoUicHwPCwVdaD5QDIWIL7DE1pA/640?wx_fmt=png)

这里还需要注意一点，要想实时返回命令结果还需要关闭缓存

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0I5NefdDCPliatFsiahQQUVu6XJXYC0rSdXZkqoOpX1FQfFG9MCO5984A/640?wx_fmt=png)

虽然 cloudflare 可以随时清除，但不能手动去做，可开启页面规则，绕过所有缓存。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0YicFoaOsPCN8zWNnKJLTK7Rg9fibg4XIhmYEb4EdGw1s6QcZIyRJwt1Q/640?wx_fmt=png)

编辑缓存级别为绕过即可。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0iaiasMXXdOpKz6zpQafXCrnUO4VoH9G95xtJr7gysZgwPLAE4wwxvJVQ/640?wx_fmt=png)

**这里的坑：**

不是设置地址为 www.+++.tk 就完事了，楼主单纯写完域名后去解析，死活无法上线，直接报错 520error，还 520，我还 521 呢我。

**填坑方式：**

域名后的内容根据 malleable.profile 规则设定，例如 jQuery-2.2.4 的所有请求包均为 *.js，在页面规则中域名配置后跟  *.js 即可，如果为其他内容，例如 amazon.profile，gmail.profile，都要跟 * 字符，意为域名后请求所有目录均绕过缓存。

**2.2 隐藏流量特征：profile**
----------------------

Malleable C2 profile 作为 CS 的配置文件，可以配置通信流量的特征，用来隐藏自己的行踪，以 Malleable-C2-Profiles 为例：

https://github.com/rsmudge/Malleable-C2-Profiles

官方参考地址：

https://www.cobaltstrike.com/help-malleable-c2

**填坑 1：**生成 shellcode 或可执行文件时渗透时，是在目标机放一个小的 payload，然后由这个小的 payload 去下载大马，这个过程是个分段过程，不是一次下载回来的，其中下载请求相关的流量特征，可以通过 http-stager 来定义：

```
http-stager {
  set uri_x86 "/get32.gif";
  set uri_x64 "/get64.gif";
  client {
    parameter "id" "1234";
    header "Cookie" "SomeValue";
  }
  server {
    header "Content-Type" "image/gif";
    output {
      prepend "GIF89a";
      print;
    }
  }
}
```

**填坑 2：**使用 cloudflare 隐藏 c2 还要设置 profile 中的 head 的 mime-type，具体为：需要在 http-config 将头设置为 header "Content-Type" "application/*; charset=utf-8"，不然可能会出现能上线但是无法回显命令的情况：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0BzaTgrUY9cmh3vBqgStnT4z9quSYQp1SElHoUC5wUL1ibYj9KIWjMqg/640?wx_fmt=png)

**填坑 3：**在 profile 中设置 user-agent 可避免各种被检测，同时也是 https 反向代理的有力识别标志。

例如将 ua 设置为 Mozilla/5.0 (Windows NT 6.1; Trident/8.0; rv:12.0)：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0E5A6mzNYCQKKuGE4HINTYgcuQWltZ6WFRj9oS8YRuw8TUplGKGXLvg/640?wx_fmt=png)

在使用 nginx 反向代理时即可过滤：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0skLX88p3PjErCmjqJEaEcNGcVNd1ZwLQM3zeB86kgdbHtzCnJlWhUA/640?wx_fmt=png)

此时如果有多个工具生成 shellcode 上线 ua 不同，可设置多条件过滤，满足多人运动的需求：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w07F59Pdf6bhiaeApFoKoh83wB69gz3z0fibJjvQ9RFulb6iaobCcPKuAxA/640?wx_fmt=png)

最后在 https-certificate 配置中还要对 https 证书进行声明：

```
https-certificate {
set keystore “api.xxx.com.store”;
set password “123456”;
}
```

最后检查配置文件有效性：  

```
./c2lint malleable.profile
```

即可在 teamserver 启动时加载 profile：./teamserver 1.1.1.1(你的 ip) ******(密码)

```
malleable.profile
```

2.3 服务器反向代理限制访问  

最重要的一个知识点就是使用反向代理限制你的 c2 被别人发现，例如配置：

```
location ~*/jquery {
if ($http_user_agent != "Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko") {
return 302 $REDIRECT_DOMAIN$request_uri;
}
proxy_pass          https://127.0.0.1:5553;
}
```

就可以很好地隐藏自己，但这里有一些坑点，楼主是踩了又跳出来;  

**填坑 1：**在 nginx 配置信息中 location ~*/ 位置，需配置 x-forword 信息，同时在 profile 设置，否则上线的外网 ip 为自己的 vps，或 cdn 地址，无法获取外部信息：

nginx.conf 文件：

```
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0DVj3NRWppDvVQcBOSrXUWsgdnXLvUhktcEoIH4wGzToEWqJ2TW1Wtg/640?wx_fmt=png)profile 文件：

```
http-config {
    set trust_x_forwarded_for "true";
}
```

**填坑 2：**在 nginx 反向代理配置中，启动监听器时 http host 可为域名地址，bind port 可另选一个，作为 proxy_pass 内容：  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8cvQtyplN5ydfjo6Eeu9w0f2icbEkibOqAt403hZgmVqlUQdXeBibic8Zy1qD5mENT53v2iboq6dhvwNw/640?wx_fmt=png)

其中 bind port 可在本机使用防火墙屏蔽，只允许内部访问：

```
iptables -I INPUT -p tcp --dport 45559 -j DROP
iptables -I INPUT -s 127.0.0.1 -p tcp --dport 45559 -j ACCEPT
```

但 proxy_pass http://127.0.0.1:45559; 的地址选择也有门道，**reverse http 类型可填写任意本机真实 ip 信息**，如 localhost，127.0.0.1，甚至有外网网卡的直接外网真实 ip 都可以正常上线。  

**但 reverse https 类型只能填写 127.0.0.1**，如使用真实 ip，如 99.199.99.199，proxy_pass https://99.199.99.199.:45559 的请求 cdn 会一直超时，造成无法上线。

**CS4.3 下载地址：**

微信公众号：HACK 学习呀

**后台回复：CS4.3**

即可获得下载地址

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

**[CS 如何配置通过 CDN 上线](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247490277&idx=1&sn=62528cb1168e28003a59d693ec44006e&chksm=ec1f4fdadb68c6cc111edb7fa3b33c8e868c294e6eaee2127e912f24db8636be3e800e53a2d1&scene=21#wechat_redirect)  
**

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8Qfeuvouibfico2qhUHkxIvX2u13s7zzLMaFdWAhC1MTl3xzjjPth3bLibSZtzN9KGsEWibPgYw55Lkm5VuKthibQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496352&idx=1&sn=df6ddbf35ac56259299ce37681d56e5b&chksm=ec1ca79fdb6b2e8946f91d54722a7abb04f83111f9d348090167b804bc63b40d3efeb9beabbe&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：伞

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)