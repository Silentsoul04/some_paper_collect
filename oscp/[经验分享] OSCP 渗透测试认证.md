> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/dhcn/p/7850730.html)

http://f4l13n5n0w.github.io/blog/2015/05/05/jing-yan-fen-xiang-oscp-shen-tou-ce-shi-ren-zheng/

“120 天的旅程即将结束，以一场历时 24 小时没有选择题的考试，收获屠龙路上第一座里程碑。…” 这是我通过 OSCP 认证考试时，第一时间的感受。自豪和欣喜之情不亚于 2008 年我拿下 CCIE R&S 的时候。 关于 PWK (Pentesting with Kali Linux) 和 OSCP (Offensive Security Certified Professional)，我想很多人会觉着陌生。但说起 Offensive Security，BackTrack，Kali，NetHunter 和 Metasploit，圈里的朋友应该就熟悉多了。

[](http://f4l13n5n0w.github.io/downloads/oscp-certs.png)[![](http://f4l13n5n0w.github.io/downloads/oscp-certs.png)](http://f4l13n5n0w.github.io/downloads/oscp-certs.png)

作为在老外圈子里备受推崇的渗透测试技术类认证，国外从业人员对它的介绍和评价已经足够丰富了：  
[http://netsec.ws/?p=398](http://netsec.ws/?p=398)  
[http://www.jasonbernier.com/oscp-review/](http://www.jasonbernier.com/oscp-review/)  
[http://leonjza.github.io/blog/2014/11/22/trying-harder-oscp-and-me/](http://leonjza.github.io/blog/2014/11/22/trying-harder-oscp-and-me/)  
[http://www.securitysift.com/offsec-pwb-oscp/](http://www.securitysift.com/offsec-pwb-oscp/)  
[http://wp.sgrosshome.com/2014/07/22/offensive-security-certified-professional-my-experience-and-review/](http://wp.sgrosshome.com/2014/07/22/offensive-security-certified-professional-my-experience-and-review/)  
[http://www.primalsecurity.net/0x2-course-review-penetration-testing-with-kali-linux-oscp/](http://www.primalsecurity.net/0x2-course-review-penetration-testing-with-kali-linux-oscp/)  
[https://www.rcesecurity.com/2013/05/oscp-course-and-exam-review/](https://www.rcesecurity.com/2013/05/oscp-course-and-exam-review/)  
[http://buffered.io/posts/oscp-and-me/](http://buffered.io/posts/oscp-and-me/)  
[http://www.hackingtheperimeter.com/2014/12/introduction-id-wanted-to-take-on-oscp.html](http://www.hackingtheperimeter.com/2014/12/introduction-id-wanted-to-take-on-oscp.html)

然而，遗憾的发现，并没有关于这个认证的中文介绍。也许是因为语言问题，也许是因为这个认证还在起步阶段没有宣传吧。但是不同于 CEH 等一系列的纸上谈兵，这是一个真枪实弹的学习经历。不同于现在火热的 Web 攻击，这个认证更偏重于传统的内网渗透技术。

关于教材
----

300 多页的教材涵盖了所有从基础知识到进阶的渗透测试技巧（当然不是深入）。所以对于没有渗透经验的朋友（但还是需要基本计算机和网络的知识，如果有编程基础会更好但不是必须），这是一个很好的入门级课程。关于这个课程的具体内容可以查看[这里](https://www.offensive-security.com/documentation/penetration-testing-with-kali.pdf)。

关于 PWK Labs
-----------

相对于其他认证课程来说，OSCP 与众不同也是最吸引人的是它设计详实的 Online Lab（包括 4 个子网，57 台主机 / 服务器，每一台主机 / 服务器都包含 Offensive Security 团队在真实工作环境中遇到的漏洞）。我的起点是 VPN 连接到 Public Network 中的一台主机，终点是渗透进入到 Admin Network。 57 台设备（除掉网关 / 防火墙）都是目标，都存在漏洞可以被攻破并获得最高权限。所以获取所有设备的最高权限（ROOT/SYSTEM）并读取证明文件（proof.txt 或者 network-secret.txt）也就是终极目标。在实现这个终极目标中，你不仅会学习各种 exploits 利用，弱密码，Web 漏洞以及一系列提权技巧和漏洞利用。除了技术层面，更重要的是你能学到耐心和思路。漏洞就在那里，如果没找到只能是因为不够仔细和耐心，”TRY HARDER” 是整个课程的标语。每攻破一个主机，都需要在后边自主的学习，查资料，有时还需要灵光一闪的运气。更难得的是，lab 里边 50 多台主机服务器不是独立无关的。有时候你需要攻破特定主机并从中获取相应信息才能帮助你攻破其他的主机。而整个 Lab 里的高潮恐怕要算是端口转发（port forwarding）和 Pivoting（实在不会翻译，类似于利用跳板进入不可路由网络）。

关于考试
----

OSCP 的认证考试也是另类的存在，考生拥有 24 小时的时间（实际是 23 小时 45 分钟）去完成考试，具体如何分配时间由考生自己决定。题目是 5 台主机（随机抽取），目标是攻入并拿到最高权限（ROOT/SYSTEM）。战利品有 local.txt（一般权限可得）和 proof.txt（最高权限可得），分别对应不同的分数。满分 100 分，70 分可以通过考试。24 小时结束之后，你还有 24 小时去完成并提交考试报告（需要详细说明攻击步骤和里程碑截屏来证明确实攻破并获得相应权限）。

我的经验
----

首先，虽然课程本身是没有门槛的，但是购买 Lab 机时还是不便宜的。对渗透了解的越少你所需要的时间就越多，所以我还是建议先对渗透测试有一定了解之后再选择这个课程，有几个非常好的热身网站推荐给大家：  
[http://vulnhub.com/](http://vulnhub.com/)  
[https://www.pentesterlab.com/](https://www.pentesterlab.com/)

下面就是我的 120 天学习经历

由于不知道水深水浅，一开始买了 60 天的 Lab 时间。拿到教材和视频之后，大概用了一周多的时间把教材和视频过完，并完成课后练习题（这个时间取决于你的基础有多少，每天 4-6 个小时学习的话应该最多不会超过 3 周）。

终于可以摩拳擦掌进入在线 Lab 了。我的起点是 Public Network 里的一台电脑，利用课程里学到的知识对目标网络进行信息收集和枚举（enumeration）。Nmap 是用的最多的工具之一。目标主机 / 服务器有着不同的难度，所以我的策略是先从头开始一个一个过，拿下简单的，如果没思路了就留下，开始下一个。过完一遍之后，再翻过头来做更仔细的枚举（enumeration）和上网查资料，直到发现漏洞。几乎所有的主机都存在 “已知” 漏洞，所以剩下的工作就是仔细挖掘把它找出来。然而现实并不像想象的那么简单，很多时候的感觉就是 “你明明知道漏洞就在那里却找不到” 的崩溃。每一台攻破的机器都包含一个 proof.txt 战利品文件用来证明该主机已经被征服，有时候还会发现 network-secrets.txt 文件用来在 control panel 里解开不同的子网（IT, DEV and Admin）。有些主机里还包含攻克其他主机的线索，所以对每台被攻破的主机都需要仔细检查一番，搜集尽可能多的线索。

很快的 60 天的 Lab 时间就用完了，我只拿下了 30 几台主机并解锁了 2 个子网（IT 和 DEV）。我显然还没有准备好考试，于是又续了 30 天的 Lab。这 30 天里剩下的都是一些硬骨头了，更多的的利用 Pivoting 拿下 IT 和 DEV 子网里的主机，并解锁了 Admin 子网。时间又一次接近尾声，还剩下不到 10 台主机（其中包括 “著名的”pain，sufferance 和 humble）。我再一次续了 30 天，我想拿下所有的主机。最终，除了 humble 和 Admin 子网里的 Jack（获得 shell 但没能提权）没有拿下，我攻破并提权了其他所有的 50 多台主机。我想，我应该去考试了。之后花了我大概一周的时间整理出 Lab 实验报告，一共是 369 页。

我约的考试是从周五晚上 7 点开始，时长 23 小时 45 分钟。一共五台目标机器跟 Lab 里的一样包含已知漏洞。总分 100 分，70 分过。

晚上 8 点 27 分，拿下第一台主机。 10 点，第二台主机。 凌晨 1 点 27 分，第三台主机也被攻破。 然后一直到凌晨 5 点多，再没有建树。困得不行的我，决定小睡三小时，闹铃上到早上 8 点半起。截止到这时候，我只有 50 分。 8 点半起来继续研究，终于 10 点 52 分，拿下第四台主机（我认为是最难的一台）。 这时已经拿到 75 分，可以过了。中午简单的吃了些午饭，回来继续最后一个主机。

又过了几个小时的网上搜索，修改和测试，终于在下午 2 点 35 分的时候攻破最后一台主机。这时我拿到了 100 分。在反复检查和做好所有截屏工作之后，我决定先去好好的睡一觉。24 小时的考试结束后，我还有 24 小时的时间整理和提交考试报告。 一觉睡到晚上 9 点多，起来写报告。因为只有五台主机，又有了之前 Lab 报告的经验，截屏和记录工作做的还不错，报告花了我大概 2 两个多小时就结束了。之后就是打包上传，然后再去睡个好觉 :)

过了一天，收到了下边的邮件，写下了开头的那句话。

[](http://f4l13n5n0w.github.io/downloads/oscp.png)[![](http://f4l13n5n0w.github.io/downloads/oscp.png)](http://f4l13n5n0w.github.io/downloads/oscp.png)最后，感谢 IRC 里的 admin 们和所有聊过天，帮助过我的朋友：

*   goodbestguy
*   haken29a
*   loneferret
*   bolexxx
*   g0tmi1k_
*   stormtide31
*   Zipkoppie
*   B34

最最后，发几个对 Lab 和考试有帮助的总结帖子（cheat sheets）:

*   [Reverse shell cheat sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
*   [Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
*   [Creating Metasploit Payloads](http://netsec.ws/?p=331)
*   [OSCP Handy Tips and Tricks](https://sathisharthars.wordpress.com/2015/01/28/oscp-offensive-security-certified-professional-handy-tips-and-tricks/)