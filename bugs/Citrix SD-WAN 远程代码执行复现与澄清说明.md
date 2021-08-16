\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/ylKR7zdefIsSJ8y8Mi6KMw)

1 介绍

Citrix SD-WAN 作为软件、虚拟设备和硬件
--------------------------

Citrix SD-WAN 有可用的软件、虚拟或硬件设备版本。企业通过使用 Citrix 的产品将 SDN 和 NFV 引入其广域网，使其更具可扩展性、成本效益更好，同时确保强大的应用程序性能。Citrix SD-WAN 还可以通过集成路由、防火墙和 WAN 优化功能帮助企业简化其分支网络。

Citrix SD-WAN 安全功能
------------------

该产品附带的 Citrix 防火墙达到他们公司防火墙硬件标准，同时也已经通过了 ICSA 实验室的认证。根据 ICSA 的一份报告，Citrix SD-WAN 410 设备起初因为防火墙没有为记录的事件提供足够的信息。 最初未能满足 set 安全性和功能性要求。但后来的版本通过固件升级把两个问题都解决了。

通过 Citrix 的合作伙伴（如 Palo Alto Networks 或 Zscaler）合作，Citrix SD-WAN 编排服务可以在分支站点和基于公共云的安全网关之间创建 IPsec 隧道。提供针对高级威胁保护和威胁情报的优质服务。

大概长这个样子  

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iadHz4YCpfgPiahzQQCKjYEmUU1b5pSFIRrDvPPiaTibSnkuNpOMg8PwbIQ/640?wx_fmt=png)

2 影响范围

**影响范围** : Citrix SD-WAN 11.2 before 11.2.2

Citrix SD-WAN 11.1 before 11.1.2b

Citrix SD-WAN 10.2 before 10.2.8

3 漏洞复现

使用福林表哥提供的脚本 脚本暂时不予提供 各位师傅可以去 github 找找  

python url 'whoami'  

![](https://mmbiz.qpic.cn/mmbiz_jpg/7XAvvlbibo1QLSJeewYic79537eaQABI0iaibk3jUibKfNFwqqawhOgn0QlQtnXakC3IvBhkmkbdtxxU0ROtiatDJ7GA/640?wx_fmt=jpeg)

4 修复方案

Citrix 公司已经针对该漏洞发布了更新，请访问以下链接并升级版本

https://support.citrix.com/article/CTX285061=

澄清说明

阿乐你好公众号一直与零组团队一起战斗 这个复现其实昨天都搞好了 因为这个憨批让我 tm 没心情发了  造谣一张嘴 辟谣跑断腿

暂时没有盈利例如公众号接一些广告 知识星球 昨天就出了一个神人 喷我公众号接盈利性广告  还有 2 个啥都不知道就就在那跟风 既然说从我公众号进来的 

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iagPZMRCLlibZPqRPEkFXd0D1eQvZic8b8qq73IEmFOXWJuq5ykdrtCL6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iafwER4NtL7FATUtFyyZEFFJPIfXibwicclsRsyndkEOZtLf9wrLU569Kg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iaG9rORTwlNrVheUzIQwkh7lrgl5Ric2y25lls55guYwsCECh1WfpANLA/640?wx_fmt=png)

微信公众号有一个功能 就是被删的文章 也有记录 我就全部截图出来

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iaicicUhRUMA1VKVBibU7qibWLsaWia4uMxibJNVPX0PgZ5TuU1Ukf55pozpag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0ia7FGKoENS9icBGCibpLdwFTYAeUqU3wypX7hQk5DDFw5WvAibEUTRvnGeg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0ia2255x3urVzho9qt8tpzm3kQiaGbk4K1UftXEiaT9II5uj1iajicMd4655A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iaBfhzgic46ILfIYv047WXb3FPUG2HkPVsD9XyMz7TjiaQgshuraEsfH5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0ia5NAWVAnKfMOEOJMULYbcuzwpPzGzKVd6yypZCse8TJXGicDMzDEfWdg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0ianU6vBaphv0txM8cv76VLX9Vyia0eYTxnM2giag99ovByQnmtHVlmSENw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iaEQnVS0sbQw1ojo7R3Raata31w9Np755tvicicgaNppM3JX7ia34icInkibQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iaZx9YqUOvlJOYvGbK9iaGN01DMlWsPAnd98xjx3dTH3qYYHyqZrpiaxhQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iaMYz6cQjOePrMHvfrYyVEZOH2sZU9tyxa2mMPjXNPeE60AjpKn5DRkw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7XAvvlbibo1QLSJeewYic79537eaQABI0iaqE7Isw6HRe1xVno02ZpNQz016I11wQKOuxc2aJYp26DHrN9F98zpJg/640?wx_fmt=png)

借用乌云的一句话  

与其相信谣言 不如相信阿乐