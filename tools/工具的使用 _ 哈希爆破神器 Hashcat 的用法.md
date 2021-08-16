> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247484188&idx=1&sn=89efce85027e9a73a4e1ccfd8886aaf4&chksm=eaad8321ddda0a37a03ba907c989234d17c1b131a8931fad68d223bbd32ff9a898afec2c2a16&scene=21#wechat_redirect)

**目录**

  

HashCat

HshCat 的使用

使用 Hashcat 生成字典

使用 Hashcat 破解 NTLMv2

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HzZrTT4UZAcukC3cKqetSHpGJABL8ZCM8yibLyNpvY2Zia3IAY3P6yE9A/640?wx_fmt=gif)

  

  

HashCat 系列软件在硬件上支持使用 CPU、NVIDIA GPU、ATI GPU 来进行密码破解。在操作系统上支持 Windows、Linux 平台，并且需要安装官方指定版本的显卡驱动程序，如果驱动程序版本不对，可能导致程序无法运行。

HashCat 主要分为三个版本：Hashcat、oclHashcat-plus、oclHashcat-lite。这三个版本的主要区别是：HashCat 只支持 CPU 破解。oclHashcat-plus 支持使用 GPU 破解多个 HASH，并且支持的算法高达 77 种。oclHashcat-lite 只支持使用 GPU 对单个 HASH 进行破解，支持的 HASH 种类仅有 32 种，但是对算法进行了优化，可以达到 GPU 破解的最高速度。如果只有单个密文进行破解的话，推荐使用 oclHashCat-lite。

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3omibErD0ylUqAdVrtJNSatMGHTmp1EJfkFZ3oOOtVyayHwic46picqfC9z4AN9xsMosJbw1WFDQIguaA/640?wx_fmt=png)

HshCat 的使用

由于笔者穷逼一个，所以使用最简单 cpu 破解。

```
-m   指定哈希类型

-a   指定攻击模式，有5中模式

    0 Straight（字典破解）

    1 Combination（组合破解）

    3 Brute-force（掩码暴力破解）

    6 Hybrid dict + mask（混合字典+掩码）

    7 Hybrid mask + dict（混合掩码+字典）

-o   输出文件

-stdout  指定基础文件

-r  指定规则文件

-V   打印出版本

-h   查看帮助
```

-m 参数的一些哈希类型

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPByklBtNm47bj0b8mEV3ywtia7H4kiaeOYTr9qticibvp0Mx0eKfWjb5kGQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

使用 Hashcat 生成字典

rules 目录下存放着生成字典的各种规则

我们在当前目录下将基础信息保存在 base.txt 文件中

输出成 test.txt 文件

```
hashcat64.exe --stdout base.txt -r C:\Users\17250\Desktop\hashcat-4.1.0\rules\dive.rule -o test.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPuPFiaT4maauZaEd4097GClMSJFIluL3aawtnzLmznhk4VkK7ENCwh2g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC5x6JawVlxYwrsf4OxhIz1HoaHEjBLqmAGrZlH8BTIAaGKt4xLxqt7gEL9Jj00Y7u9ic8Xy6EYiaVBQ/640?wx_fmt=gif)

使用 Hashcat 破解 NTLMv2

hashcat64.exe -m 5600 Net-NTLM-Hash  password.txt

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPcuWSq0SQ5ccCmneMT9vSCicYgUEJUFZRvLbnzp9Hbb0usBXeWbjD9jA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2e4moqygI59agYAD3Btf8jPdDnhrE2FDiaxLV3cOXBHz2WEQSHwgjfcN1bLyWZtVW7C1TjIJaaegNA/640?wx_fmt=png)

                  

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2ckkbwTsBvnDJpb89o8WMxvAKOaVnz60hOe7y3wAHiclddyK53lpEKIQlx4DKOq6EojHibVicgibDB2aQ/640)

来源：谢公子的博客

责编：梁粉

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640)

由于文章篇幅较长，请大家耐心。如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640)