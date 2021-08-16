> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4hnDRGRhvg_PpcZih5mS5Q)

**高质量的安全文章，安全 offer 面试经验分享**

**尽在 # 掌控安全 EDU #**

**作者：学员昵称 -“**混吃等死”****

这里就简单聊一下学完正式课，去 hvv 的一些实战分享吧，这里运气比较好，去了两次还都算有所收获。

由于有些涉及单位的信息，而且图有些也没截取全，部分地方就用语言简单描述一下，主要还是分享一下思路。

案例
--

首先这里的 hvv 是给了我们对应目标资产的，就我们这个单位给了一个门户网站的地址。

  
目前该网站已经访问不了了，在整改。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFQ7vicktibNQEVvu0LwicR6pQs5IpjnGdfoPXTicfDLNHVBVOYT4S3K4nbQ/640?wx_fmt=png)

#### 1. 信息收集

就复现一下当时的渗透思路，步骤吧。首先拿到一个网站，我开始做的是通过站长工具对其进行进行 ping 检测，发现各地返回的地址都是相同，说明基本是一个实际 IP。

然后用 Nmap 对其进行全端口扫描挂着，nmap -Pn -p 1-65535 ip

扫描结束后，发现只开了公网的 http，并没有其他什么东西。

那么就基本要从这个 web 应用来开刀了，先简单粗暴，信息收集。

随便点了点，发现有类似传参的东西，http://********/search.aspx?key=1

试了之前学的判断是否存在 SQL 注入的方法，发现具有有 WAF，那没辙了，毕竟比较菜，不会绕，而且绕过了拿到里面的数据，也得先找后台是吧，hvv 目标是拿下目标服务器，并进行内网渗透。

在官网我们用风哥给我们的 7kbscan-WebPathBrute。

把目标网址直接填进去开始开扫，先整理一波网站目录。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFnzMu2qxTtv56HmkuTuGcUzicSjnf7vLqwhyxjicNvbibn0Dib6tsmKlyWA/640?wx_fmt=png)

这里直接扫描到了他的后台 http://*********/admin/Admin_Login.aspx

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFwokhU4fSPOiaDdTxOuED9KcoGRmUNI4EZ4LiaQGTzPy45dfnhoV9lTBw/640?wx_fmt=png)

本来想爆破，一看有验证码，这不完了蛋了吗。

试了一下简单的 admin/123456、admin/admin 也没有进入成功，先放着，看看有没有其他的问题。

  
不过这里我发现验证码可以绕过，这里的问题是验证码可以重放。

截了个包，在 reprater 模块按了几次 send 后发现返回包还是提示用户名或密码错误。

那就放着爆破（没爆出来，看来不是弱密码）。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFmxEyz3vYe3rcaOFOtkU3WqpAlfulpJgKkcFHpgtQE95rBCBmibO5SVw/640?wx_fmt=png)

那好办了，先爆破一波，用户名 admin，然后弱密码走你，梭哈！

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFXiaO13Ta8oW9lZOSEvbkDUxa61XuZmmgjRYBlsWLUyRhDC2188q8z4A/640?wx_fmt=png)

在工具帮你爆破的时候也别闲着，看看主站有没有什么地方可以搞的。

  
这里我看着看着，主站有一个链接，写着信息 ** 系统。

我点了一下，是个登录界面，而且这个登录界面还有个帮助信息。

我点开来看了一下，由于现在关站了，当时没有截图，我就口述一下

大致这样：用户名由姓全拼，名简写组成，如张小丽，用户名就为 zhangxl，初始密码 111111。

  
好家伙，这不就来了吗，看到这个一想到总会有人不改密码的，但是我好像没有姓全拼，名简写的字典啊....

求助风哥, 给的是姓名首字母的简拼。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaF5OxSNeBGtmJzRB6FfU3diauaOfGEz3kCV2oKMr2q59LdHM53hRBaYYw/640?wx_fmt=png)

这里我找网友要来了一个字段生成工具，生成了我们想要的字典  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFIicfZicf0L6xPRzzX4vp9PsJIprgLicdxll2GSreTmjTlaMfFa5t1dGVw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFZ2qzVAEcicxgBYthLmCWCfR5sWpvIuJaO4qBQeulZTk8HgBich9B4UGQ/640?wx_fmt=png)

这就舒服了呀

  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFmfRABtRtBqHxr3guyBvmkJv50fWURPI73CQnDBicWmRvr1OZicekOxqQ/640?wx_fmt=png)

  
他那个后台同理有验证码重放的问题，直接用户名上字段，默认密码 111111 开跑。

#### 2. 进后台找上传点

运气不错，直接跑到了很多默认密码，估计这系统好久没用了吧

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaF12ibU56AjcCj7LydIJX0tuUCBCOOXU6doVRwvGGUicg2gBkWKf8ZxCEg/640?wx_fmt=png)

那么我们来看看这里面有没有什么我们可以利用的功能呢？

比如找个上传点。

  
这里找到一个，投稿的地方，有个上传图传的功能。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFkn9Oia0UUxXbgMiaiaQgSOaIRNxM46Vs98JdgOlHdCcYHOtQViaQIQ8Ttw/640?wx_fmt=png)

这边比较简单，直接 burp 改对应后缀就能上传成功了。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaF3BiaibutpLxericPEaSABfzjOr9MrR4nHSpUk81ZaaR3fsoTVW5sZbXXg/640?wx_fmt=png)

然后右键图片，复制图片地址，找到路径，蚁剑连接 TMD

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFosnTrC6icTia9FGbn6zZfIFwd2p36sWQMbgZJ6trtalRWCVqOESPicrnw/640?wx_fmt=png)

#### 3. 提权杀入内网

打开虚拟终端，看看，发现不是 administrator 用户，那么就想提权。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFxjAoV9X4AVg4YNMialWwDqw5uvVDQjDRF6NcPMNhGXTCnuicX3FtnLbg/640?wx_fmt=png)

用学院之前给的魔改版烂土豆，发现死活传不上去，可能有什么拦截掉了。

试了半天反正没搞定，只能找一起打 HW 的小伙伴帮忙了。

让他用冰蝎连接目标服务器，然后上传文件。

冰蝎好像是传输加密的，所以无法检测到？

我不太清楚，总之他用冰蝎连接成功了。

  
直接给他新增一个管理员账户 admin。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFqZtibU07QJGPSS8vKc7ibFVrK2T6icbJ1IB6RzaboHD9M2ZGjU3nrUMuQ/640?wx_fmt=png)

然后发现，好像公网没有开对应的远程端口。

继续求助小伙伴，他用 CS 挂了一个代理。这里的代理必须是目标机器能访问公网，因为我们是公网的代理服务器。

我目前还不会![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaF5r4TOrnbibNWLulg49PqI4Kkm2C23TCLQZ5p1vcicPzpeVL61TQvm2kA/640?wx_fmt=png)

  
（学院出内网渗透课，必买）

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFCVsX1dXVWGyauFwjZyWKTVkMu7zGLdiakWlGjtEhtTWt8xFZenzQYUw/640?wx_fmt=png)

代理挂好后，用 Proxifier 工具连接代理

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFiaHQib1I9Dz0Yv4ea7U5jAwIKTLFHNrwwPPlicoz2gUkadw0RAnKxJTSg/640?wx_fmt=png)

这个正式课里面的福利课里有涉及，具体端口 ip 和你挂代理的设置有关

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFAiccSqjWMWcJU8XJnsxSib7TF7EYjDZQ59QLw79VccyuOP8dywjFn8Rw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFFibRITvnhejGz5N7O8Gpj1xdvaAzgia9NmdaBvQOdQPSyf9QarbUNteQ/640?wx_fmt=png)

这不成了么，我直接在本机就能访问到目标服务器了

  
然后使用 netstat -ano 发现这家伙居然没有远程 3389，好家伙，跑机房运维的？？？

  
直接百度，cmd 命令开始 RDP 远程的命令：

REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal” “Server /v fDeny

TSConnections /t REG_DWORD /d 0 /f

然后用之前增加的 admin 账户远程

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFw84M2KCDUvlrj5jLnybKzajkTwLbmxJFEdW0aUGZc8kmQyHZlicDVLA/640?wx_fmt=png)

成功了，成功了，杀入人家内网了。

  
这里我特地去看了一下，为啥没开远程，好家伙原来 administrator 是空密码

所以用 “猕猴桃” 也没抓出密码来（本来想用这个工具试试内网其他的服务器是不是相同密码），然后他们是用 TeamViewer 远程的。

#### 4. 拿下内网各种东西的权限

**4.1 本机的东西**

首先现在本机找点有用的东西，毕竟 hvv 是看你拿下的权限有多少

首先直接 netstat -ano 发现有个 6379 的端口，那默认是 redis 数据库

  
发现该数据库是空口令，直接进去了。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFqxj7S9v9vf26ByDHMj7PekbqcibX8EfuD1mybbbXvEB5Qu1ULo0SumQ/640?wx_fmt=png)

然后去源代码里面找到数据库的连接帐号密码

  
好家伙，密码设置的还挺复杂

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFhK8IgtXmLaE1pvR7v6pFNQS3QaQu0Jy9kiagB6icquOWmuLtIoj8iakbg/640?wx_fmt=png)

连接成功

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFdKV7IfdSZGNBmu5CupAeT4apibhqxUJlSzxNNTibgY35VFiaibEcKcHicCA/640?wx_fmt=png)

然后可以找对对应 SQL Server 数据库中存储的内容，可查询到对应系统的用户名和口令

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFib1gtS0iaCbnSnO0ic4ETONjfl1yLEdZ6TwKSfG1iaTX2fW9Nny6GMpLzQ/640?wx_fmt=png)

**4.2 内网中其他东西**

我们这里就直接在他机子上装了个 nmap，用这台机子直接对它的内网 C 段进行扫描。

发现了一个应用系统，弱密码 admin/admin，管理员权限

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFB0LiaqP5mub6dc3ZjmvmAkRmuPzBtgodhq6F7jiaEiaI3ibDItXEPhDibRg/640?wx_fmt=png)

拿下好多默认密码的华为视频会议系统

admin\Change_Me

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFkA6M2p6Sa38ibOVNZibVibbhDEib2IcwwltQOLAibZz9L7qG2pjyIzyJ4uQ/640?wx_fmt=png)

还有各式各样一些其他的东西（打印机、什么记录仪等）

比较杂，应用这块我就不举例了，进了内网后搞的东西可多了。

**4.3 拿下某管控平台权限**

重点来了，在我们拿下的那台服务器上，我们发现了他是一台服务器的管理端，有个 8443 端口。

好家伙，这玩意管理端没装杀毒？我们这个烂土豆都没杀掉？

（之后的 hw 碰到有杀毒的都凉了，没有免杀的，好气，提不了权）

我们找到控制台的登录界面。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFXjbLD5gLVib72JanaIic0jMd8o7MqmADoJa5icbK0owAzHhianN8ZqtJpg/640?wx_fmt=png)

试了一下，不是弱密码，还有登录失败锁定，这可怎么搞？  

经过仔细的排查，发现最近删除掉了一个叫 TQ 的 txt 文件，然后那个文件正好可以恢复，里面居然有他的密码。真的是鸿运当头。

然后成功登陆。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFAfLx2VSQWcWCxUtibdj98ib5UYeJUXClhC3TibS2iacA3icHiazqgGXVM8uA/640?wx_fmt=png)

重点来了，重点来了，这个软件有个下发文件的功能

  
我们直接上传 CS 马，并分发执行

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFcP4NUMXZLrMGCYAS2EkLDJiciaw5jB6OUQEIOQ5lxSyFsboE6KmcnByg/640?wx_fmt=png)

好家伙，装了这个软件的全部中招，这权限是真的大

这里服务器 + 终端总计 78 台

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFTe9XY0YiciciaaymZ4S5gO6sqUtn8KN1KTqPyMr5NlhZN3MuaV07GpsTA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaF3cQsGdQQIiaiaH7KEgl5IcmzjpuuMNAYiaviaqY9coWIpTfcsPt6FxlAew/640?wx_fmt=png)

为了避免发马过多，我们这里就选了一台作为测试

  
这些我不会，都是小伙伴做的后面

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFRqAGVG1POkZiaEM9rcS4V7R3Cx0pWrib9OWwmuicUvicNOficCC1hyIEssQ/640?wx_fmt=png)

运行 “猕猴桃”，找到了这台的密码，还挺有复杂度的，一般猜不到

  
直接远程拿下

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFRqAGVG1POkZiaEM9rcS4V7R3Cx0pWrib9OWwmuicUvicNOficCC1hyIEssQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFBofKWlrSWuBib8wszZibpdBYvyOKAGYLfrbnATrfQY0gnYZ1AQJGrImA/640?wx_fmt=png)

建议对这种分发功能应该设置二级密码进行限制

拿下的这台服务器里面还有应用系统，可以找他数据库密码

#### **5. 总结**

这是我第一次 hvv，根据学到的知识，应该说成功的开了个口子，

**总体大概：**  
根据提示存在默认密码和知道用户名命名规则 → 制作对应字典爆破系统

  
进入系统后台，找对应上传点 → 上传马，拿下目标服务器

  
在目标服务器上挂代理，杀入内网 → 进一步进行内网渗透

  
进到了目标服务器，后面才会有更大的动作。

因为内网渗透不太会，基本叫好兄弟帮忙搞的。

总之运气还是不错的，刚好没装杀毒，不然很可能提权那一步就出问题了。

而且针对有 WAF 的站点，目前能力有限，直接放弃，通过信息收集来获取进入后台的权限。

  
感觉挺好玩的，有空去了解一下免杀 ，如果有搞个免杀版烂土豆就舒服了。

  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpgPk47BrsibJNObVETT5qiaFNRHnObDHx14WTEFRrQkHeW5zSQg2jV6ET7fiaclVBF9brsCWfMutsibw/640?wx_fmt=png)

  

**回顾往期内容**

[Xray 挂机刷漏洞](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247504665&idx=1&sn=eb88ca9711e95ee8851eb47959ff8a61&chksm=fa6baa68cd1c237e755037f35c6f74b3c09c92fd2373d9c07f98697ea723797b73009e872014&scene=21#wechat_redirect)  

[POC 批量验证 Python 脚本编写](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247504664&idx=1&sn=e88c77671f252631de939c154de075db&chksm=fa6baa69cd1c237f1c1f35f8b434874341f7fe077452834dac0e289addf9ac56fcbf7df5a8a1&scene=21#wechat_redirect)

[实战纪实 | SQL 漏洞实战挖掘技巧](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247497717&idx=1&sn=34dc1d10fcf5f745306a29224c7c4008&chksm=fa6b8e84cd1c0792f0ec433310b24b4ccbe53354c11f334a1b0d5f853d214037bdba7ea00a9b&scene=21#wechat_redirect)  

[渗透工具 | 红队常用的那些工具分享](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247495811&idx=1&sn=122c664b1178d563ef5e071e0bfd7e28&chksm=fa6b89f2cd1c00e4327d6516c25fcfd2616cf7ae8ddef2a6e869b4a6ab6afad2a6788bf0d04a&scene=21#wechat_redirect)  

[代码审计 | 这个 CNVD 证书拿的有点轻松](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503150&idx=1&sn=189d061e1f7c14812e491b6b7c49b202&chksm=fa6bb45fcd1c3d490cdfa59326801ecb383b1bf9586f51305ad5add9dec163e78af58a9874d2&scene=21#wechat_redirect)

 [代理池工具撰写 | 只有无尽的跳转，没有封禁的 IP！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503462&idx=1&sn=0b696f0cabab0a046385599a1683dfb2&chksm=fa6bb717cd1c3e01afc0d6126ea141bb9a39bf3b4123462528d37fb00f74ea525b83e948bc80&scene=21#wechat_redirect)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif)

扫码白嫖视频 + 工具 + 进群 + 靶场等资料

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpx1Q3Jp9iazicHHqfQYT6J5613m7mUbljREbGolHHu6GXBfS2p4EZop2piaib8GgVdkYSPWaVcic6n5qg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWFyt1RHHuwgcQ5iat5ZXkETlp2icotQrCMuQk8HSaE9gopITwNa8hfI7A/640?wx_fmt=png)

 **扫码白嫖****！**

 **还有****免费****的配套****靶场****、****交流群****哦！**