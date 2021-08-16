> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JaVPRvbfE9aY1GpLj4-HQA)

**本篇文章由 ChaMd5 安全团队开发小组投稿**

**写在前面**  

  LBS 的场景大家遇到的不是一两回了，很多人遇到能出外网的可以轻松搞定，遇到那种节点机器不出网的就人傻了，今天咱们就拿捏一下。  

**本文阅读时间可能较久，适合反复阅读。这篇文章不删，请放心阅读。**

**正文**

  负载均衡 (Load Balance) 是一种廉价的扩容的方案，它的概念不是本文的重点，不知道的可以去查资料学习。实现负载均衡的方式有很多种，比如 DNS 方式、HTTP 重定向方式、IP 负载均衡方式、反向代理方式等等。

比如 DNS 方式就是这种:

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNTxnCfU3pXf5LGnPya30iaUVDrbm05wZPRKUVBKC516Fr5cjMWZFIibVw/640?wx_fmt=png)

再比如用 ipvsadm 来做的 IP 负载均衡:

```
ipvsadm -a -t 30.0.30.10:80 -r 172.16.1.2:8080 -m
ipvsadm -a -t 30.0.30.10:80 -r 172.16.1.3:8080 -m
ipvsadm -a -t 30.0.30.10:80 -r 172.16.1.4:8080 -m
```

再比如反向代理的负载均衡：  

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNZGLo1NUB0Dic5R6UAeQNicRLZzGNDCIKOSRgJeXZlYyU91jwfKJZOoIg/640?wx_fmt=png)

其中像 HTTP 重定向方式、DNS 方式等能够直接访问到单一机器的情况，不在我们本文讨论范围内。连接的时候，URL 处按 IP 格式来填，然后把域名加在 Host 头处，就完事了。**我们重点讨论不能直接访问到跑着具体业务的某个节点的情况，比如说「反向代理方式」。**  

反向代理方式其中比较流行的方式是用 nginx 来做负载均衡。我们先简单的介绍一下 nginx 支持的几种策略：

| 

**名称**

 | 

策略

 |
| 

轮询（默认）

 | 

按请求顺序逐一分配

 |
| 

weight

 | 

根据权重分配

 |
| 

ip_hash

 | 

根据客户端 IP 分配

 |
| 

least_conn

 | 

根据连接数分配

 |
| 

fair (第三方)

 | 

根据响应时间分配

 |
| 

url_hash (第三方)

 | 

根据 URL 分配

 |

其中 ip_hash、url_hash 这种能固定访问到某个节点的情况，我们也不讨论，跟单机没啥区别么不是。  

我们以默认的「轮询」方式来做演示。演示的环境已经上传至 AntSword-Labs，有兴趣的朋友可以自行去尝试。  

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNeMX1B3E3XSoGbsNnOHUnsiapFyT8fod72xdtNlTexTJvkX2Dg72Yyog/640?wx_fmt=png)

为了方便解释，我们只用两个节点，启动之后，看到有 3 个容器（你想像成有 3 台服务器就成）。

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNgfSbtib9oibKZkmIJibE87mumShujTQZSsRiboQbsPiaQibichNWM1IcaSbMw/640?wx_fmt=png)

现在整个架构长这个样子：

```
┌─────────────┐
                          │             │
                   ┌──────►  LBSNode 1  │
┌─────────┐        │      │             │
│         │        │      └─────────────┘
│  Nginx  ├────────┤
│         │        │      ┌─────────────┐
└─────────┘        │      │             │
                   └──────►  LBSNode 2  │
                          │             │
                          └─────────────┘
```

Node1 和 Node2 均是 tomcat 8 ，在内网中开放了 8080 端口，我们在外部是没法直接访问到的。

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNic96NF8UicJwW42zNn60Qqza0drZib5G11xe491qO5bFWwh1gXWJGqaicg/640?wx_fmt=png)

我们只能通过 nginx 这台机器访问。nginx 的配置如下：

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNusS1JczAmLQueQ7NBQY1YZqr5tkjVvmyIn38EzghyibUSbibnRhROKzg/640?wx_fmt=png)

 **场景描述**   

OK，我们假定在真实的业务系统上，存在一个 RCE 漏洞，可以让我们获取 WebShell。  

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNhCQfonOLmqfiaiaDD4wRZ93IjnHxPZb0ZF7GNZtnOIGiauBV5G8hib95WA/640?wx_fmt=png)

我们先按常规操作在蚁剑里添加 Shell  

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNOQibHvOWicfJfria3NkC6TZu6j94j54RB2rZ3vIH4uFdwK47A0KxtEW3A/640?wx_fmt=png)

然后连接目标，因为两台节点都在相同的位置存在 ant.jsp，所以连接的时候也没出现什么异常。

**难点一**：我们需要在**每一台节点**的**相同位置**都上传**相同内容的 WebShell**

一旦有一台机器上没有，那么在请求轮到这台机器上的时候，就会出现 404 错误，影响使用。**是的，这就是你出现一会儿正常，一会儿错误的原因。**

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNAVGm00dGDXdh06ibicRqTCumpwiaPJd9TUtw3JIu8ibicPK4SQPLcKX30XA/640?wx_fmt=png)

**难点二**：我们在执行命令时，**无法知道下次的请求交给哪台机器去执行**。

我们执行 ip addr 查看当前执行机器的 ip 时，可以看到一直在飘，因为我们用的是轮询的方式，还算能确定，一旦涉及了权重等其它指标，就让你好好体验一波什么叫飘乎不定。  

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNic7W85J9rMGX47SBusREF2oV2e2zERusY9XukTqLibB5RRiaBuEaPgLGg/640?wx_fmt=png)

**难点三：**当我们需要**上****传一些工具**时，麻烦来了**：**  

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNyVXibvISujGl256tgRc9DKLNmaO6R3OMZY847gXLibJD6QXAQ6icDIpiaA/640?wx_fmt=png)

我们本地的 111.png 大小是 2117006, 由于 antSword 上传文件时，采用的分片上传方式，把一个文件分成了多次 HTTP 请求发送给了目标，所以尴尬的事情来了，两台节点上，各一半，而且这一半到底是怎么组合的，取决于 LBS 算法，这可怎么办？

**难点四：由于目标机器不能出外网，想进一步深入，只能使用 reGeorg/HTTPAbs 等 HTTP Tunnel，可在这个场景下，这些 tunnel 脚本全部都失灵了。**  

如果说前面三个难点还可以忍一忍，那第四个难点就直接劝退了。这还怎么深入内网？

 **Plan A**  **关****掉其中一台机器** **（作死）**  

是的，首先想到的第一个方案是关机 / 停服，只保留一台机器，因为健康检查机制的存在，很快其它的节点就会被 nginx 从池子里踢出去，那么妥妥的就能继续了。  

这个方案实在是「**老寿星上吊——活腻了**」，影响业务，还会造成灾难，直接 Pass 不考虑。（实验环境下，权限够的时候是可以测试可行性的）。  

****综合评价**：真实环境下千万不要尝试！！！！**

 **Plan B**  **执行前先判断要不要执行** 

我们既然无法预测下一次是哪台机器去执行，那我们的 Shell 在执行 Payload 之前，先判断一下要不要执行不就行了？  

以执行命令时 Bash 为例，在执行前判断一下 IP：

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNBQY4wxs5gHF3icDBM0UicCLNmibK4O8qlLnCl5qJyiadeOZnBMGdhiaShRA/640?wx_fmt=png)

效果大概就是这个样子：  

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNKG3ib88dmy6YTRNISoEZZnJNsAuIOFnT2VR9vQjDH2EguuWDER5P2YA/640?wx_fmt=png)

这样一来，确实是能够保证执行的命令是在我们想要的机器上了，可是这样执行命令，**不够丝滑**，一点美感都没有。**另外，上传文件、HTTP 隧道 这些要怎么解决？**

**综合评价**：该方案 「**勉强能用****」**，仅适合在执行命令的时候用用，不够优雅。

 **Plan C**  **在** **Web 层做一次 HTTP 流量转发** **（重点）**

没错，我们用 AntSword 没法直接访问 LBSNode1 内网 IP(172.23.0.2) 的 8080 端口，但是有人能访问呀，除了 nginx 能访问之外，**LBSNode2 这台机器也是可以访问 Node1 这台机器的 8080 端口**的。

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNribLVJiaGiaSplEJzAKRBLYKnC1r0dQDZXbeyJicLvdnoxh1zVNQQbRFKg/640?wx_fmt=png)

还记不记得 「PHP Bypass Disable Function」 这个插件，我们在这个插件加载 so 之后，本地启动了一个 httpserver，然后我们用到了 HTTP 层面的流量转发脚本 「**antproxy.php**」, 我们放在这个场景下看：

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNR3dxJjLkxvsdoViaO0MKFicjCoW8686PgGNQGNvNmTBWBx48Fw6f8aMg/640?wx_fmt=png)

我们一步一步来看这个图，我们的目的是：**所有的数据包都能发给「LBSNode 1」这台机器。**

首先是 第 1 步，我们请求 /antproxy.jsp，这个请求发给 nginx

nginx 接到数据包之后，会有两种情况：

我们先看黑色线，第 2 步把请求传递给了目标机器，请求了 Node1 机器上的 /antproxy.jsp，接着 第 3 步，/antproxy.jsp 把请求重组之后，传给了 Node1 机器上的 /ant.jsp，成功执行。

再来看红色线，第 2 步把请求传给了 Node2 机器, 接着第 3 步，Node2 机器上面的 /antproxy.jsp 把请求重组之后，传给了 Node1 的 /ant.jsp，成功执行。

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy060lXtb8YklejbUq0pljIExfycMZyTCNMysg0kO2PibsPSzsCWdIPiclsIt22OfyWmYtwRUqbhe9aw/640?wx_fmt=png)

**完美**

**我们看看怎么具体操作**

**1.** **创建 antproxy.jsp 脚本**

**修改转发地址**，转向**目标 Node** 的 内网 IP 的 **目标脚本** 访问地址。

**注意：不仅仅是 WebShell 哟，还可以改成 reGeorg 等脚本的访问地址。**

我们将 target 指向了 LBSNode1 的 ant.jsp

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNBgFicxvjgwLTN4fS3pnBvzGHOCmibWVK323HicansBxseGibPRx2UTYaTw/640?wx_fmt=png)

**注意:** 

**a) 不要使用上传功能**，上传功能会分片上传，导致分散在不同 Node 上。

**b)** 要保证每一台 Node 上都有相同路径的 antproxy.jsp, 所以我疯狂保存了很多次，保证每一台都上传了脚本

**2. 修改 Shell 配置, 将 URL 部分填写为 antproxy.jsp 的地址，其它配置不变**

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNWNytyqDwGRBHicax3u9vwWf5d4gSCXcX9QOaPZOohBiaDIgF6HiaXeDbw/640?wx_fmt=png)

**3. 测试执行命令, 查看 IP**

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNRNm31rTltPibpmdZylQPSyyyOKWb7OE64zf4guToBFUv3F2uy4Z5ZFA/640?wx_fmt=png)

可以看到 IP 已经固定, 意味着请求已经固定到了 LBSNode1 这台机器上了。此时使用分片上传、HTTP 代理，都已经跟单机的情况没什么区别了。

查看一下 Node1 上面的 tomcat 的日志, 可以看到收束的过程：

![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYNw1Hkibfiapq1UicudZ5mkB1nVpglHgvGL7PNto24YGgotPwASWgiazDVpQ/640?wx_fmt=png)

Node1 和 Node2 交叉着访问 Node1 的 /ant.jsp 文件，符合 nginx 此时的 LBS 策略。  

**优点：**

*   低权限就可以完成，如果权限高的话，还可以通过端口层面直接转发，不过这跟 Plan A 的关服务就没啥区别了
    
*   流量上，只影响访问 WebShell 的请求，其它的正常业务请求不会影响。
    
*   适配更多工具
    

**缺点：**  

*   该方案需要「目标 Node」和「其它 Node」 之间内网互通，如果不互通就凉了（敲黑板：加固方案快记下来）
    

**写在最后**  

      学过了思路，就剩下课后作业了，antproxy.php 大家都比较熟悉了，剩下的几种脚本语言的 Proxy 脚本目前还没有。马上就 4 月了，**快去艾特你的好基友和你一起写这个 HTTP 转发脚本吧～ ![](https://mmbiz.qpic.cn/mmbiz_png/lkcJVly3Wy3wic9XX592fic7lx4iaBGSCYN6RM1MS2xyabxwghQqV1Kg0so0b0KdqZ1iaJBUlBdxnYyicoxIFpYD0TA/640?wx_fmt=png)** 

end

  

招新小广告

ChaMd5 Venom 招收大佬入圈

新成立组 IOT + 工控 + 样本分析 长期招新  

欢迎联系 admin@chamd5.org

  
  

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBR8nk7RR7HefBINILy4PClwoEMzGCJovye9KIsEjCKwxlqcSFsGJSv3OtYIjmKpXzVyfzlqSicWwxQ/640?wx_fmt=jpeg)