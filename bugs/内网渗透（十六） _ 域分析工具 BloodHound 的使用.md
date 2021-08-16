> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247494062&idx=1&sn=0c486f53daca08ee61abc51926db2b96&chksm=fc7bed73cb0c646557767e2e21d3c0113df80f831263dea4ce45bd6969da44f27ee700518427&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFhLz1H5qAkgh9wkAnWtKQNJd5gpJXE7XFR5qAuM2JpmdfLVUoDkug3r0BJF0TiaMK5vyiaYCEzwqeag/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490017&idx=1&sn=426336dfeeda818b0772b3c44703e173&chksm=fc781d3ccb0f942a7c07662752bb2f6983eb9c249c0d6b833f058b1d95fc7080d2d2598054ac&scene=21#wechat_redirect)

作者：谢公子

CSDN 安全博客专家，擅长渗透测试、Web 安全攻防、红蓝对抗。其自有公众号：谢公子学安全

免责声明：本公众号发布的文章均转载自互联网或经作者投稿授权的原创，文末已注明出处，其内容和图片版权归原网站或作者本人所有，并不代表 安世加 的观点，若有无意侵权或转载不当之处请联系我们处理，谢谢合作！

**欢迎各位添加微信号：qinchang_198231** 

**加入 安世加 交流群 和大佬们一起交流安全技术**

**BloodHound**

BloodHound 是一款域内免费是分析工具。BloodHound 通过图与线的形式，将域内用户、计算机、组、会话、ACL 之间的关系呈现出来。BloodHound 使用图形理论，在活动目录环境中自动理清大部分人员之间的关系和防护。攻击者使用 BloodHound 可以快速、深入地了解活动目录中用户之间的关系。BloodHound 可以在域内导出相关信息，将采集的数据导入本地 Neo4j 数据库，并进行展示和分析。Neo4j 是一款 NoSQL 图形数据库，它将结构化数据存储在网络内而不是表中。BloodHound 正是利用 Neo4j 这种特性，通过合理的分析，直观地以节点空间的形式表达相关数据。

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDKQB1KSQA7ciaEmgpib2IIjlWo4tmHficuryibBJchFjbibobmlYqxvhLC4w/640?wx_fmt=gif)

**BloodHound 的安装**  

我这里以 64 位的 windows10 系统为例

**1：安装 Neo4j 数据库**

Neo4j 数据库的运行需要 Java 环境的支持，我这里已经安装好了 Java 环境。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDQ2KNT407QF7GLClg6ia7XRPuM5EV8Tq7MHibY6GhGSr4uSSvU41P7VfA/640?wx_fmt=png)

去 Neo4j 官网下载社区免费版：https://neo4j.com/download-center/#community

下载后，将安装包解压，然后打开命令行窗口，进入解压后的 bin 目录，输入命令：neo4j.bat console，启动 Neo4j 服务。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDfEBP2kwDKcS17y86HA6iaU2AER8KG44I3tT9jdaJCv3PXIricX13mFYw/640?wx_fmt=png)

服务启动后，浏览器输入：http://127.0.0.1:7474/browser/ ，然后在打开的页面中输入用户名和密码即可。

Host 默认是：bolt://127.0.0.1:7687 ，账号和密码默认是：neo4j

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDuHUqx9ubAEA7ej9X4DlHZP2A9BPEIDEeDpYPa7m0icNNbKducn53YWw/640?wx_fmt=png)

点击 Connect，连接成功后，第一次会让我们修改密码，我这里将密码修改为 123456。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDpBAH9YicCJ1P9iaeiakECgFJkW0g7CTldouhLUKWtGALGBkjgRnFA2FNw/640?wx_fmt=png)

充值完密码之后，就可以正常使用 Neo4j 了

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDLDKpPicLbHx6xrazsCQEQgVial7EkXpWOZnWxXiaSib53UM7Q7h9lxKHWg/640?wx_fmt=png)

**2：安装 BloodHound**

下载 BloodHound：https://github.com/BloodHoundAD/BloodHound/releases/download/2.0.4/BloodHound-win32-x64.zip

下载后解压，进入解压目录，找到 BloodHound.exe，双击。这里输入我们上面的账号密码登录即可。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDBN5Nl3rdfYib2vdzv1C9RVnyNJeEySSzZ8DJCK8TZg2NDFL5lqqUPBQ/640?wx_fmt=png)

登录成功后的截图

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDAPSQH6MlHI503U8pzSm2yU5iaFW5mkAqgjT7Wg4b6w6RXuM9nYy9HKQ/640?wx_fmt=png)

界面左上角是菜单按钮和搜素栏。三个选项卡分别是数据库信息 (Database Info)、节点信息(Node Info) 和查询(Queries)。数据库信息选显卡中显示了所分析域的用户数量、计算机数量、组数量、会话数量、ACL 数量、关系等信息，用户可以在此处执行基本的数据库管理操作，包括注销和切换数据库，以及清除当前加载的数据库。节点信息选项卡中显示了用户在图表中单击的节点的信息。查询选项卡中显示了 BloodHound 预置的查询请求和用户自己构建的查询请求。

界面左上角是设置区。

第一个是刷新功能，BloodHound 将重新计算并绘制当前显示的图形；

第二个是导出图形功能，可以将当前绘制的图形导出为 JSON 或 PNG 文件；

第三个是导入图形功能，可以导入 JSON 文件；

第四个是上传数据功能，BloodHound 将对上传的文件进行自动检测，然后获取 CSV 格式的数据；

第五个是更改布局类型功能，用于在分层和强制定向图布局之间切换；

第六个是设置功能，可以更改节点的折叠行为，以及在不同的细节模式之间切换。

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDKQB1KSQA7ciaEmgpib2IIjlWo4tmHficuryibBJchFjbibobmlYqxvhLC4w/640?wx_fmt=gif)

**采集数据**  

在使用 BloodHound 进行分析时，需要调用来自活动目录的三条信息：

1.  哪些用户登录了哪些机器？  
    
2.  哪些用户拥有管理员权限？  
    
3.  哪些用户和组属于哪些组？  
    

BloodHound 需要的这三条信息依赖于 PowerView.ps1 脚本的 BloodHound。BloodHound 分为两部分：

1.  一是 PowerShell 采集器脚本 (旧版本叫做 BloodHound_Old.ps1，新版本叫做 SharpHound.ps1)，
    
2.  二是可执行文件 SharpHound.exe。
    

执行以下命令用于收集信息，不需要管理员权限。

```
SharpHound.exe -c all
```

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDmiayBImdVevlJauq5DfCdm8wbDv5XXqoFBLdxuWJ6P0icrvvyDMK7FGA/640?wx_fmt=png)

  

执行完成后，会在当前目录生成 时间戳_BloodHound.zip 的文件

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDKQB1KSQA7ciaEmgpib2IIjlWo4tmHficuryibBJchFjbibobmlYqxvhLC4w/640?wx_fmt=gif)

**导入数据**  

BloodHound 支持通过界面上传单个文件和 ZIP 文件，最简单的方法是将压缩文件放到界面上节点信息选项卡以外的任意空白位置。文件导入后，即可查看内网的相关信息。

如图，该内网有 7 个用户、4 台主机、41 个组，2 个 sessions，411 条 ACLs、472 个关系。

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDVFuGeb8YYITS7Vs7R6HvE3rPlGXUrzw5lzjBwoRrj2ULBUia7eANvMw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDKQB1KSQA7ciaEmgpib2IIjlWo4tmHficuryibBJchFjbibobmlYqxvhLC4w/640?wx_fmt=gif)

**查询信息**  

进入查询模块，可以看到预定义的 12 个常用查询条件

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDdj8ia4Y1DrQuDu6FicyMF3rh27icJay8THJIZdYQ7XYsHhneIibvnrDrnw/640?wx_fmt=png)

1.  Find all Domain Admins：查询所有域管理员
    
2.  Find Shortest Paths to Domain Admins：查找到达域管理员的最短路径
    
3.  Find Principals with DCSync Rights：查找具有 DCSync 权限的主体
    
4.  Users with Foreign Domain Group Membership：具有外部域组成员身份的用户
    
5.  Groups with Foreign Domain Group Membership：具有外部域组成员身份的组
    
6.  Map Domain Trusts：映射域信任
    
7.  Shortest Paths to Unconstrained Delegation Systems：无约束委托系统的最短路径
    
8.  Shortest Paths from Kerberoastable Users：Kerberoastable 用户的最短路径
    
9.  Shortest Paths to Domain Admins from kerberoastable Users：从 Kerberoathable 用户到域管理员的最短路径
    
10.  Shortest Path from Owned Principals：拥有主体的最短路径
    
11.  Shortest Paths to Domain Admins from Owned Principals：从所属主体到域管理员的最短路径
    
12.  Shortest Paths to High Value Targets：高价值目标的最短路径  
    

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFhiaADKtzrxwOADv1wWJmfoDGY2kuckbeII73trVyRGTaBu2rfh9p35LnDqGYDxsicm5AMtCVarvG0A/640?wx_fmt=png)

**本文所需工具，关注公众号：asjeiss ，后台回复：BloodHound 即可获取下载链接**

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFjWI9QibTmpF13L33cHIh2bSMLAI4tW7sTgTkzh4lRcZ6JR7SrOibCTYUEsg8ZsmyKnUBm7h4J5klZw/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MzA3NzM2MjAzMg==&mid=2657228904&idx=1&sn=aa0d7a52864f19cbd6245a46ce162a1f&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFgag1b8Ubew6DhiceZ9tKengA7WrOUhVx2wCKjHy6GSFbZ3YLKjy6N7LB9p9p7eRNiapvEiax7b5N2Jg/640?wx_fmt=png)

[内网渗透 | 红蓝对抗：Windows 利用 WinRM 实现端口复用打造隐蔽后门](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247493916&idx=2&sn=eacc42e5f8f68fc65dae1c8a1201f014&chksm=fc7bedc1cb0c64d7115c0c3bf84410e29102a25627891c9eb85cba2026b7bc3622a9ebb5e2b2&scene=21#wechat_redirect)  

[内网渗透（十五） | psexec 工具使用浅析 (admin$)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247493004&idx=1&sn=e908ac6ef03c0b5ae0cc5a2cabba7ebb&chksm=fc7be151cb0c684789bbc5f6a54e5906a3fed929eeffdba7667839491826fbb029bba295ef82&scene=21#wechat_redirect)  

[内网渗透（十四） | 工具的使用 | Impacket 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247492731&idx=1&sn=570c1d9e12ef39709e289b5cc9e2447f&chksm=fc7be0a6cb0c69b0d94c41408b862214beaa631b04ba32f2307819a8b3ac445726849cb24e7f&scene=21#wechat_redirect)

[内网渗透（十三） | WinRM 远程管理工具的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247492427&idx=1&sn=af3a862d78184e93b6e9377f12bce354&chksm=fc7be796cb0c6e80a057dff2a7d67e3483c33e8da2d3a7acb84d04fdd997f28cb89f1fd617fd&scene=21#wechat_redirect)  

[内网渗透（十二） | 利用委派打造隐蔽后门 (权限维持)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247491363&idx=1&sn=e5d6670b0f76299d92110d7b679ad70b&chksm=fc781bfecb0f92e8aacaa6f4f7788ed48577e25f943d92073b1b26e68bfbc8f505b2dd2fa4d8&scene=21#wechat_redirect)  

[内网渗透（十一） | 哈希传递攻击 (Pass-the-Hash,PtH)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490908&idx=1&sn=97594fbbef40346d07b5a6e5185ce77e&chksm=fc781981cb0f9097d18f4b32ff39f59b3512cedd35f0810ad5f61b661e631153f8c4e157d875&scene=21#wechat_redirect)  

[技术干货 | 工具：Social engineering tookit 钓鱼网站](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490513&idx=2&sn=10afb29a20f37df05ebb12ea4d540e1f&chksm=fc781f0ccb0f961a85e646dd54e977dbcaeb5569be6701db4c29b9e204d964bab3ded6bf1999&scene=21#wechat_redirect)

[技术干货 | 工具的使用：CobaltStrike 上线 Linux 主机 (CrossC2)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490608&idx=1&sn=f2b2ea93b109447aa8cc2c872aa87c52&chksm=fc7818edcb0f91fbf85fa53f71e9967fc29fc93f6a783eed154707ca2dec24ca7f419fde5705&scene=21#wechat_redirect)

[内网渗透（十） | 票据传递攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490376&idx=2&sn=c070dd4c761b49d3fabd573cc9c96b5a&chksm=fc781f95cb0f9683b0f6c64f5db5823973c1b10e87b1452192bbed6c1159eccf6e8f2fd0290b&scene=21#wechat_redirect)  

[内网渗透（九） | Windows 域的管理](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490197&idx=1&sn=4682065ddcab00b584918bc267e33f53&chksm=fc781e48cb0f975eddc44d77698fbb466d0eac7d745a6e5bbaf131560b3d4f9e22c1a359d241&scene=21#wechat_redirect)  

[内网渗透（八） | 内网转发工具的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490042&idx=1&sn=136d4057044a7d6f6cb5b57d20f7954a&chksm=fc781d27cb0f9431ec590662ab4e6bcd31b303e7caa20a2b116fd9a9b97e9e3be0bc34408490&scene=21#wechat_redirect)  

[内网渗透 | 域内用户枚举和密码喷洒攻击 (Password Spraying)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489985&idx=1&sn=0b7bce093e501b9817f263c24e0ed5b8&chksm=fc781d1ccb0f940aad0c9b2b06b68c7a58b0b4c513fe45f7da6e6438cac76d4778e61122faf8&scene=21#wechat_redirect)  

[内网渗透（七） | 内网转发及隐蔽隧道：网络层隧道技术之 ICMP 隧道 (pingTunnel/IcmpTunnel)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489736&idx=2&sn=0cb551ee520860878c2c33108033c00c&chksm=fc781c15cb0f9503f672aa0bd18cb13fef4c60124ba5978ab947c34272b2d8a28c584a99219d&scene=21#wechat_redirect)  

[内网渗透（六） | 工作组和域的区别](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489205&idx=1&sn=24f9a2e0e6b92a167f3082bb6e09c734&chksm=fc781268cb0f9b7e3c11d19a9fb41567124055eb0e8dd526cbbaf1e9393ff707f9fa9d10c32b&scene=21#wechat_redirect)  

[内网渗透（五） | AS-REP Roasting 攻击](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489128&idx=1&sn=dac676323e81307e18dd7f6c8998bde7&chksm=fc7812b5cb0f9ba3a63c447468b7e1bdf3250ed0a6217b07a22819c816a8da1fdf16c164fce2&scene=21#wechat_redirect)

[内网渗透 | 内网穿透工具 FRP 的使用](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489057&idx=3&sn=f81ef113f1f136c2289c8bca24c5deb1&chksm=fc7812fccb0f9beaa65e5e9cf40cf9797d207627ae30cb8c7d42d8c12a2cb0765700860dab84&scene=21#wechat_redirect)  

[内网渗透（四） | 域渗透之 Kerberoast 攻击_Python](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488972&idx=1&sn=87a6d987de72a03a2710f162170cd3a0&chksm=fc781111cb0f98070f74377f8348c529699a5eea8497fd40d254cf37a1f54f96632da6a96d83&scene=21#wechat_redirect)  

[内网渗透（三） | 域渗透之 SPN 服务主体名称](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)  

[内网渗透（二） | MSF 和 CobaltStrike 联动](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[内网渗透 | 域内认证之 Kerberos 协议详解](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)  

[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)