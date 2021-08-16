> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/89b24_7NTaofE--wUY9ybw)

**前言**
======

      关于 DNS 隧道的一些简单研究和利用。

**DNS 协议基础**
------------

     域名系统（服务）协议（DNS）是一种分布式网络，主要用于域名与 IP 地址的相互转换。

### **DNS 域名解析流程**

     本地 DNS 缓存 - 递归查询 - 迭代查询

**本地 DNS 缓存**  
     包含浏览器缓存，本地 host 文件，系统 dns 缓存

**递归查询**  
     该模式下 DNS 服务器接收到客户机请求，必须使用一个准确的查询结果回复客户机。

**迭代查询**  
     本地域名服务器向根域名服务器查询，根域名服务器告诉它下一步到哪里去查询，然后它在根据结果逐层向下查询，直到得到最终结果。每次它都是以 dns 客户机的身份去各个服务器查询，即迭代查询是本地服务器进行的操作。

#### **基本解析流程**

     以访问 http://test.cseroad.space 进行举例

```
1、检查浏览器缓存，本地host文件和本机的dns缓存，失败后，向本地设置的dns服务器（如114.114.114.114）发送查询请求，dns服务器到自身解析数据库中查询，查询成功返回IP地址（此过程成为递归查询）查询失败则触发迭代查询过程。
2、本地dns服务器向根域名服务器发送关于space的查询请求。
3、根域名服务器接收到查询请求，并把查询结果返回给dns服务器。
4、本地dns服务器收到根域名服务器返回的顶级域名服务器的地址，并向它查询关于cseroad的域名服务器地址。
5、顶级域名服务器接收到请求，进行查询，并把查询结果返回到dns服务器。
6、本地dns服务器收到关于cseroad的权限域名服务器地址，并发起查询test的请求。
7、权限域名服务器收到请求，并把test对应的A记录的ip返回给dns服务器。
8、本地dns服务器把权限域名服务器返回的ip地址发送到个人电脑。
9、个人电脑成功解析到http://test.cseroad.space对应的ip地址，在浏览器中进行访问。
```

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChcBylYg9DNVFjjWic8YDhYpdhzaoacLS4WsjMwiaYJGAek08Vgu2t9fHQ/640?wx_fmt=png)

**DNS 隧道**
----------

     DNS 隧道(DNS Tunneling)是将其他协议的内容封装在 DNS 协议中，然后以 DNS 请求和响应包完成传输数据 (通信) 的技术。当前网络世界中的 DNS 是一项必不可少的服务，所以防火墙和入侵检测设备处于可用性和用户友好的考虑将很难做到完全过滤掉 DNS 流量，因此，攻击者可以利用它实现远程控制，文件传输等操作。

### **DNS 隧道的两大类型**

     直连隧道：用户端直接和指定的目标 DNS 服务器建立连接，然后将需要传输的数据编码封装在 DNS 协议中进行通信。这种方式的优点是具有较高速度，但蔽性弱、易被探测追踪的缺点也很明显。另外直连方式的限制比较多，如目前很多的企业网络为了尽可能的降低遭受网络攻击的风险，一般将相关策略配置为仅允许与指定的可信任 DNS 服务器之间的流量通过。

     中继隧道：通过 DNS 迭代查询而实现的中继 DNS 隧道，这种方式及其隐秘，且可在绝大部分场景下部署成功。但由于数据包到达目标 DNS 服务器前需要经过多个节点的跳转，数据传输速度和传输能力较直连会慢很多。

     在实战用到 DNS 隧道的场景，对隐蔽性要求很高，速度相对来说没那么重要，因此主要使用中继隧道。

### **技术要点**

DNS 缓存机制的规避  
     再使用中继隧道时，如果需要解析的域名在本地的 DNS Server 中已经有缓存时，本地的 DNS Server 就不会转发数据包。所以在构造的请求中，每次查询的域名都是不一样的。

DNS 载荷的编码  
     从高层来看，载荷只是客户端和服务器通信的正常流量。例如客户端发送一个 A 记录请求给服务器，查询的主机名为 2roAUSwVqwOWCaaDC.test.nuoyan.com, 其中 2roAUSwVqwOWCaaDc 则是客户端传递给服务器的信息，这串字符解码后的信息便是 DNS 隧道。

可利用 DNS 查询类型  
     DNS 的记录类型有很多，常见的有 A，AAAA,CNAME,MX,NS 等。DNS 隧道可以利用其中的一些记录类型来传输数据。例如 A，MX，CNAME,TXT 等。  

```
A       记录 指定主机名（或域名）对应的IPV4地址记录
AAAA    记录 指定主机名（或域名）对应的IPV6地址记录
NS      记录  指定该域名由哪个DNS服务器来进行解析
MX      记录  指向一个邮件服务器
PTR     记录  将一个IP地址映射到对应的域名，也可以看成是A记录的反向
CNAME   记录  允许将多个名字映射到同一台计算机
TXT     记录 一般指主机名或域名的说明
```

**主动连接**  
     内网客户端位于防火墙后方，服务端无法做到主动连接，因此大多的 dns 隧道工具，客户端会定时向服务端发送请求，保证二者之间的通信状态。

### **常用工具**

     dnscat2，iodine，DeNise，dns2tcp 等单独的 DNS 隧道工具。  
     cs，msf，empire 等集合了 DNS 隧道功能的安全工具。  
     本次分别演示使用 dnscat2 和 cs 的 DNS 隐藏隧道。

### **实验环境**

     内网 win7 虚拟机  
     阿里云 vps  
     py 来的域名  

**Dnscat2 搭建 DNS 隧道**
---------------------

####      1、安装环境依赖和服务端

```
apt-get install ruby ruby-dev git make g++ rubygems
gem update --system
gem install bundler
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
bundle install
```

     如下表示安装成功。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChsQoqD5lBsWXibVXsKJpc4WuLJVhWReY6ohqOZqXQHcKZtnMr7YZJUNA/640?wx_fmt=png)

####      2、客户端下载安装

     Dnscat2 支持跨平台  
     linux 客户端

```
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/client/
make
```

     windows 客户端  
     下载链接  
     下载 dnascat2 的 win32.zip

####      3、建立隧道

     因为要使用中继隧道，设置一个 NS 记录指向自己的子域名，再设置一个 A 记录指向自己部署 server 端的服务器地址。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChUZiau3vjn84VR8HqylkAk52fPQiaSNre2XZpjjX38HDWp1RVhHZziaKew/640?wx_fmt=png)

     使用 dig +trace dnsa.cseroad.space 查看域名详细解析过程，对应着迭代查询过程。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChsaBlZ3p5DGSyQ5v7uMWXpUX7icC6CM5ztPmhRrKu3KE9kVWwyg5ibXibg/640?wx_fmt=png)

```
ruby ./dnscat2.rb dnsa.cseroad.space --no-cache
```

     执行 dnscat2，输入配置的子域名 --no-cache 代表禁止缓存

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChMh29r0syPnv5rqKa3pkaXEARLwrkpHfhRt8m42MyickYR2bVcNw2ltg/640?wx_fmt=png)

     secret 为随机生成的密钥，友好的给出了 2 种连接方式，分别为中继和直连。  
     直接在客户端执行中继连接方式，session established 表示隧道建立成功。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChaJlNnibCGenC5riaawAHSaKy7L0ASD7En5ROz63lQ7OzMVJicpXRKX3OQ/640?wx_fmt=png)

     服务端接收到客户端的请求，创建了 session 1 会话。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChyS24wKPNVgLMYGXXBcavhKCht1hphuE68wAYWIxicZo0VzzPDkeS2hw/640?wx_fmt=png)

     进入这个 session

```
session -i 1
```

     查看工具支持的功能。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChTzKhK2xY8AX93WbbelB0laicXDia5pCPW6XxkHsFqMqZzBkl1Zk8OrNQ/640?wx_fmt=png)

     执行系统命令，更多的功能不再演示。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChskuuOtuMLFKkdMHhpWt0XDvznqozNic4Hyn1Lu0S5iclS53tBY9CDCibw/640?wx_fmt=png)

**cs 使用 DNS 隧道建立连接**
--------------------

     cs 作为当前最流行的安全工具肯定少不了 DNS 隧道功能。  
     cs 的安装过程不再赘述。

####      1、新建基于 DNS 隧道的监听器

     payload 选择 beacon_dns，host 填写 A 类解析的域名，端口随意。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChrzibRH4bbcqgxC9jJT3GGqgqGBYEXW1esRuox32MibmggN2YXW0dJic3Q/640?wx_fmt=png)

     输入设置的 ns 服务器域名

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChpS38lbVyCWcOLUEfpbNib48J7twAb0cG19c5qahva4depplKzqtlCwg/640?wx_fmt=png)

#### 2、基于监听器创建 payload

     推荐使用 powershell，powershell 不会产生文件落地，且免杀性操作性强。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChN8iann2fr9nAhpUDcGLlNkAwPY6UiaoWlWK5D2XichYHvQqQexBDSibGhQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChVB7uQa8hgu7fF4HFntoWCUSgcFIFpn9rnhqm6ZxoJ3tTo8e72WcwcQ/640?wx_fmt=png)

成功上线。

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvChuJ81SzibrYzoTOWloapoEKYJMlm3JzfKrDIkDosrLiaDpTlyibEHpv5lg/640?wx_fmt=png)

  
     通过 Wireshark 可以查看到 cs 的流量都是通过 DNS 发出的，使用的为 A 类查询进行数据传输。  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5ickD6Rmmlic7yicHmB3JKjyvCh2sxkFhp6mO6jxIn1uzHp1ePCwKKegeAQJPqIt5mh5E59e9Fia8h2wGg/640?wx_fmt=png)

**总结**
======

     DNS 隧道在隐蔽性，穿透性上仍然具备不小的优势。  
     在大多数情况下它不会是最优选择，但在某些情况下它会成为唯一选择。

```
参考资料：
利用DNS隧道构建隐蔽C&C信道
DNS 深度理解 [ 一 ]
对 Cobalt Strike DNS隧道的理解与实战
『DNS隧道工具』— iodine
DNS Tunneling及相关实现
利用DNS Tunnel传输数据
```

作者：TIDE_nuoyan  
链接：https://www.jianshu.com/p/84c526ad1548  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**