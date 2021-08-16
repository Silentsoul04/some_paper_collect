> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VU2iBJNNI06oN5NaZcn-Sw)

  
        一款轻量级、高性能、功能强大的内网穿透反向代理服务器。目前支持 tcp、udp 流量转发，可支持任何 tcp、udp 上层协议（访问内网网站、本地支付接口调试、ssh 访问、远程桌面，内网 dns 解析等，此外还支持内网 http 代理、内网 socks5 代理、p2p 等，并带有功能强大的 web 管理端。

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3PPhMF434C3Qjcpf3z4eSRRHSotsWzTicNdGibZiag4N8lyZ1SFEfhkCug/640?wx_fmt=png)

nps 下载地址：https://github.com/ehang-io/nps/releases/tag/v0.26.8

有多个不同系统的服务端和客户端，可根据现实环境下载。  

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3Mfu38P3J8w773IDWJOpp0dfx8mB2m6cz2hjyNcWdBTqZsnsJsNzbug/640?wx_fmt=png)

实验环境：  
一台公网服务器 (vps/linux) 下载服务端  
一台内网服务器 (windows) 下载客户端

nps  

1、从 vps 上下载 nps 服务端

```
wget https://github.com/ehang-io/nps/releases/download/v0.26.8/linux_amd64_server.tar.gz
```

2、解压

```
tar-zxvf linux_amd64_server.tar.gz
```

3、安装 

```
nsp install
```

4、启动

```
nsp start
```

5、查看端口 nsp 默认 web 界面端口是 8080（可以在配置文件里修改默认端口）

```
http://xxx:8080
```

6、默认密码为 admin/123（可以在配置文件里修改默认密码）

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3tTLMlUQphlaUicKEZXQ9KiaAcibpWPiacBUo9HvN3s65sc8cJwxqqRCdeA/640?wx_fmt=png)

7、新建一个客户端设置用户名密码，密钥  

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3NNrP6mviaR8DTyOOrMLXz7YeRRp9smicmhZibWSkK6bbHcqMiaicj628Fpg/640?wx_fmt=png)

8、选择 socks 代理 ID 必须要和客户端 ID 一致  

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3CwHpF5XUAaZ92HX3B8XiclodXkAnK389r5r2xTLMzfzdRRTbcKfk9qw/640?wx_fmt=png)

9、下载客户端到内网的 windows 机器上

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo36nV7ygM8bNYYPS42TJpj1AciaYGHqt7vTJEfwLW7a2iaBXqXrtTLib4fg/640?wx_fmt=png)

10、一切就绪、直接复制命令在内网机器上运行

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3Ka07aFgdpiahpc3iaYic7YLl7IpoWtr0pd546RGC1Micj0hEgTIXC8TywQ/640?wx_fmt=png)

11、此时 windows 机器已经跑起来了、web 页面提示已上线  

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3vvjh0aBfUibPxUZ9doD0SygZf3D5HrtibpqMNbmUfngHvNhSORibEYHCA/640?wx_fmt=png)

12、我们无法从本机通到内网机器、使用 Socks5 客户端软件把流量带出来

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo30usTNwKC2ZhTIHano0fhwJcVOz1SlhwRx59V0NWdZSbWHntj54QEHg/640?wx_fmt=png)

设置我们的账号密码

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3xWP06bymg8pmMsUdfHmokXpdMEvYnetukqGPpb4DPicYnnH7j8QYibNQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3jQaWibxRmMlG537lxf283F6ImKXJENPPdwoH4utXH33bKQaevtRjcAQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV1eBYK4kibQHHZGTD0oo4ibo3KPiaFsicYo5M5iaElncjBDXmESddXgBgXcQcgadqzt9FcFwL9pSxc8dtg/640?wx_fmt=png)

当然也可以实现 tpc、udp、http 等隧道

详情了解：https://github.com/ehang-io/nps/

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTkwLkofibxKKjhEu7Rx8u1P8sibicPkzKmkjjvddDg8vDYxLibe143CwHAw/640?wx_fmt=png)

**【往期推荐】**  

[未授权访问漏洞汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484804&idx=2&sn=519ae0a642c285df646907eedf7b2b3a&chksm=ea37fadedd4073c87f3bfa844d08479b2d9657c3102e169fb8f13eecba1626db9de67dd36d27&scene=21#wechat_redirect)

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)  

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)  

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)