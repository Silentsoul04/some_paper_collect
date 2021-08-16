> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/11nH41dpX5EP-MbpEShnyw)

_**No.1  
**_

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.2  
**_

_**前言**_

这里将介绍 Cobalt Strike 中几种常见 Beacon 的特性及使用，具体有 HTTP Beacon、HTTPS Beacon、DNS Beacon、SMB Beacon。

由于笔者在学习 Cobalt Strike 过程中，所看的教程使用的是 3.x 版本的 Cobalt Strike ，而我使用的是 4.0 版本的 Cobalt Strike ，要是表哥发现文中错误的地方，欢迎留言指正。

**几种常见的 Beacon**

**- HTTP 和 HTTPS Beacon**  

HTTP Beacon 和 HTTPS Beacon 也可以叫做 Web Beacon。默认设置情况下 HTTP Beacon 和 HTTPS Beacon 通过 HTTP GET 请求来下载任务。这些 Beacon 通过 HTTP POST 请求传回数据。

```
windows/beacon_http/reverse_http
windows/beacon_https/reverse_https
```

**- DNS Beacon**

```
windows/beacon_dns/reverse_dns_txt
windows/beacon_dns/reverse_http
```

**- SMB Beacon**

SMB Beacon 也可以叫做 pipe beacon

```
windows/beacon_smb/bind_pipe
```

_**No.3  
**_

_**HTTP Beacon 和 HTTPS Beacon**_

点击 Cobalt Strike --> Listeners 打开监听器管理窗口，点击 Add，输入监听器的名称、监听主机地址，因为这里是要创建一个 HTTP Beacon，所以其他的默认就行，最后点击 Save。  

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaYtiaf4WHUu7aeicex1tXwV8JjEYJKrM6t3icHOwD2eC3XW1xKrYdTIfwg/640?wx_fmt=png)

此时可以测试一下刚才设置的监听器，点击 Attack --> Web Drive-by --> Scripted Web Delivery(s) 在弹出的窗口中选择刚才新添的 Listener，因为我的靶机是 64 位的，所以我把 Use x64 payload 也给勾选上了，最后点击 Launch。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRia6rqQMic5a5uQqUiabJUbxND6p3IMPYhSvkoQTgia8QNvlXZklgQb0RcGg/640?wx_fmt=png)

复制弹窗的命令，放到靶机中运行。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaoIgoIxcBO9DlLjPVy94FNjSB1LRvrPhJJnjGyzZV2k0hx9DHgB53TQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaQJicw0lradBKyROWLicFkIicPgbriaicdmD2ib30z3DlNIo90jx8jqbRzZIQ/640?wx_fmt=png)

此时，回到 Cobalt Strike ，就可以看到已经靶机上线了。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaKN5ZfAoIYEyUCRdDRQzb0jepfgtVC7ibHiat8jbHtAf91md566sRNZDQ/640?wx_fmt=png)

HTTPS Beaocn 和 HTTP Beacon 一样，使用了相同的 Malleable C2 配置文件，使用 GET 和 POST 的方式传输数据，不同点在于 HTTPS 使用了 SSL，因此 HTTPS Beacon 就需要使用一个有效的 SSL 证书，具体如何配置可以参考：

https://www.cobaltstrike.com/help-malleable-c2# validssl

_**No.4  
**_

_**DNS Beacon**_

DNS Beacon，顾名思义就是使用 DNS 请求将 Beacon 返回。这些 DNS 请求用于解析由你的 Cobalt Strike 团队服务器作为权威 DNS 服务器的域名。DNS 响应告诉 Beacon 休眠或是连接到团队服务器来下载任务。DNS 响应也告诉 Beacon 如何从你的团队服务器下载任务。  

在 Cobalt Strike 4.0 及之后的版本中，DNS Beacon 是一个仅 DNS 的 Payload，在这个 Payload 中没有 HTTP 通信模式，这是与之前不同的地方。

**DNS Beacon 的工作流程具体如下：**

首先， Cobalt Strike 服务器向目标发起攻击，将 DNS Beacon 传输器嵌入到目标主机内存中，然后在目标主机上的 DNS Beacon 传输器回连下载 Cobalt Strike 服务器上的 DNS Beacon 传输体，当 DNS Beacon 在内存中启动后就开始回连 Cobalt Strike 服务器，然后执行来自 CS 服务器的各种任务请求。

原本 DNS Beacon 可以使用两种方式进行传输，一种是使用 HTTP 来下载 Payload，一种是使用 DNS TXT 记录来下载 Payload，不过现在 4.0 版本中，已经没有了 HTTP 方式，CS4.0 以及未来版本都只有 DNS TXT 记录这一种选择了，所以接下来重点学习使用 DNS TXT 记录的方式。

根据作者的介绍，DNS Beacon 拥有更高的隐蔽性，但是速度相对于 HTTP Beacon 等会更慢。

**域名配置**

既然是配置域名，所以就需要先有个域名，这里就用我的博客域名作为示例：添加一条 A 记录指向 Cobalt Strike 服务器的公网 IP，再添加几条 ns 记录指向 A 记录域名即可。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaibGUW5RcZVMOLRVmlp49KnMEeIqriclYp9jgVK9cH2PN6WK1NJ9eZ09w/640?wx_fmt=png)

添加一个监听器，DNS Hosts 填写 NS 记录和 A 记录对应的名称，DNS Host 填写 A 记录对应的名称。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiagnfTXp2HMFtWRp0R808UH28q5KDxU8XEthqGCftA0vlnP0iavmnZ8ew/640?wx_fmt=png)

根据上一章的方法创建一个攻击脚本，放到目标主机中运行后，在 Cobalt Strike 客户端可以看到一个小黑框。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiazKiaPBRBnjDHyeHnGZ6qBjJKhNKoeFUykm7g1By2VXZPh7HDTgNSib6g/640?wx_fmt=png)

然后经过一段时间的等待，就可以目标主机发现已经上线了。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRia3czdUYlY7rEVLGY07Yeuh3HI6tA80oQLO1icw5wIeE9JSuu3nTEwhiaQ/640?wx_fmt=png)

_**No.5  
**_

_**SMB Beacon 简介**_

SMB Beacon 使用命名管道通过一个父 Beacon 进行通信。这种对等通信对同一台主机上的 Beacon 和跨网络的 Beacon 都有效。Windows 将命名管道通信封装在 SMB 协议中。因此得名 SMB Beacon。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRia89zN4lZ1OzUXhzFLwSIUuFvMfajaiavYVtjVmn3mI6SSnpytx8Mibzxw/640?wx_fmt=png)

因为链接的 Beacons 使用 Windows 命名管道进行通信，此流量封装在 SMB 协议中，所以 SMB Beacon 相对隐蔽，绕防火墙时可能发挥奇效 (系统防火墙默认是允许 445 的端口与外界通信的，其他端口可能会弹窗提醒，会导致远程命令行反弹 shell 失败)。

SMB Beacon 监听器对 “提升权限” 和“横向渗透”中很有用。

**SMB Beacon 配置**

首先需要一个上线的主机，这里我使用的 HTTP Beacon，主机上线后，新建一个 SMB Beacon，输入监听器名称，选择 Beacon SMB，管道名称可以直接默认，也可以自定义。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaCShKULo9icQ6ibXckq3VCHeHj0T9pCZbm8YXd8leFUhefGYwpedpUJIA/640?wx_fmt=png)

接下来在 Beacon 中直接输入 spawn SMB，这里的 SMB 指代的是创建的 SMB Beacon 的监听器名称，也可以直接右击 session，在 Spawn 选项中选择刚添加的 SMB Beacon。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaaO4vZ1s67wFOoAbM4DpIv71Tfa2WCvWISOaibbTxib6YohRpTyGXLI8Q/640?wx_fmt=png)

等待一会儿，就可以看到派生的 SMB Beacon，在 external 中可以看到 IP 后有个∞∞字符。

接下来我这里将 SMB Beacon 插入到进程中，以 vmtoolsed 进程为例。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiaC45Rib7xmyb3aouw6PjqbntTalN3VKDice7xLSbY6IQG4dSI9BpCgdSA/640?wx_fmt=png)

在 vmtoolsed 中插入 SMB Beacon 后，便能看到 process 为 vmtoolsed.exe 的派生 SMB Beacon。

当上线主机较多的时候，只靠列表的方式去展现，就显得不太直观了，通过 Cobalt Strike 客户端中的透视图便能很好的展现。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiavRQlbrIhm65LohPMyclAt7ztHicxTia3XNHvxfADhabZgSXKFaK0SD3A/640?wx_fmt=png)

在 Cobalt Strike 中，如果获取到目标的管理员权限，在用户名后会有 * 号标注，通过这个区别，可以判断出当前上线的 test 用户为普通权限用户。

_**No.6  
**_

_**参考链接**_

https://www.bilibili.com/video/BV16b411i7n5

https://klionsec.github.io/2017/09/23/cobalt-strike/

https://pythonpig.github.io/2018/01/17/Cobaltstrike-SMB-beacon/

https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf

_**招聘启事**_

安恒雷神众测 SRC 运营（实习生）  
————————  
【职责描述】  
1.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

设计师（实习生）

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
3、有品牌传播、产品设计或新媒体视觉工作经历；  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦 19 楼  
学历要求：本科及以上  
工作年限：1 年及以上，条件优秀者可放宽

简历投递至 

bountyteam@dbappsecurity.com.cn

安全招聘  
————————  
公司：安恒信息  
岗位：Web 安全 安全研究员  
部门：战略支援部  
薪资：13-30K  
工作年限：1 年 +  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
【岗位职责】  
1. 定期面向部门、全公司技术分享;  
2. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3. 负责完成部门渗透测试、红蓝对抗业务;  
4. 负责自动化平台建设  
5. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1. 至少 1 年安全领域工作经验；  
2. 熟悉 HTTP 协议相关技术  
3. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1. 具备良好的英语文档阅读能力；  
2. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5. 开发过安全相关的开源项目；  
6. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7. 个人技术博客；  
8. 在优质社区投稿过文章；

岗位：安全红队武器自动化工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

岗位：红队武器化 Golang 开发工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3. 了解 docker，能进行简单的项目部署；  
3. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4. 熟悉 TCP/IP 协议的基本运作原理；  
5. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。  
简历投递至 

bountyteam@dbappsecurity.com.cn

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUoQaib7hPc6PrvRQL3TXYRiatlz75pGODdnnY5Y4TsWFAicOHfAn7iaVVhqq14TIFfyLTpnsicrJ5CWuw/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUaP7DgO9Wloat1jvS9vibbhzL08w6p76hydoluxrxFuR8jibeypIwxSGFjHIR3Jicb2bEHTnWx40hRQ/640?wx_fmt=jpeg)