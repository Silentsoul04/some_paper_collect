> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vi9t9_ogvta6FP4kWl9v9g)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.1  
**_

_**前言**_

"当拥有 shell 而无法获得用户密码的情况下，可以尝试一下以下的骚操作"

_**No.2  
**_

_**创建普通用户**_

首先用有足够权限的 shell 创建一个 Administritor 用户：

```
net user Administritor$ yourpasswrod /add
net localgroup administrators Administritor$ /add
```

_**No.3  
**_

_**创建影子用户**_

1. 登录创建的 Administritor$ 账户下

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibuXdWNENNibM8YO0uUq0lBepbC5zqicNvtV2RBHFJtX5E7icibh93uo8b4aw/640?wx_fmt=jpeg)

2. 打开注册表 HKEY_LOCAL_MACHINE\SAM\SAM，右击权限，找到 Administrators 勾选完全控制，关掉注册表

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibu7YDAibvQrYFnmUVDx6E7yuQMia9njB00ZmXmNwgxvyBc9iakwQsOBHyAw/640?wx_fmt=jpeg)

3. 重新打开注册表 HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names 下找到创建的账户

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibuz8ZVvntQ0DicuNotheLQepjIfYVwVNMgnyzvUJ5kV5NpUufE7Nhe2eg/640?wx_fmt=jpeg)

4. 获取默认类型，记下来，然后..\..\Names\Administritor$ 导出注册表为 1.reg  

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibujGlqZ0qCgqwcLn6uiaHGTQlDPwgh4boDJsJ0V0RdhdcXuibtIbJ6H5ibA/640?wx_fmt=jpeg)

5. 在..\..\Users 目录下找到 Administritor$ 账户对应的默认类型，导出为 2.reg，比如..\..\Users\000003EE

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibuiboZJdeALicFAbEecMDhEXcQMbAped4oJKib5USthibER7qlOIPvxHBcMg/640?wx_fmt=jpeg)

6. 接着导出你想要复制的账户（假设是 administrator）的默认类型为 3.reg，如 administrator 的..\..\Users\000003E8

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibuI0TqjMwTQos8G37VicepGaRTqoZaxRkibpmNwPF18qSZk6mBwzia6264Q/640?wx_fmt=jpeg)

7. 接下来修改 2.reg，把 3.reg 里的

 F

 对应的值替换给 2.reg 里的

 F

 对应的值，保存 2.reg，删除 3.reg  

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibul008xce0FCUMQFibYdoYd9ApH7v5yTkb30xLIVV95Vt4MByFe5MNTLw/640?wx_fmt=jpeg)

8. 在 shell 窗口下 删除 Administritor$ 账户，net user Administritor$ /del

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibukE9Wn5uHcbjatnMLjz8TMZM0h2sKscp8O6lr3So3EKsRN9FbWd80og/640?wx_fmt=jpeg)

9. 刷新注册表，并把 1.reg 和 2.reg 导入进注册表，删除 1.reg 和 2.reg，并清空回收站

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibubvYPQTvbgBTyQU0fksOdkC5hiaXj1K7qpGwrRzojkzDQLSKU1Jllp3Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibuvqVovhzfw91iaEW3BqAHlSJHiaxZA7aqkhBoKdaxIa28QrJsBWM0Smow/640?wx_fmt=jpeg)

10. 最后切换账户，重新登录创建的 Administritor$ 账户下，输入你创建的密码，就进去你想进去那个账户的界面了

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibuIE1tb7iaBcRGibXKaoCgUlZGyXzx4c0JFY8jPPLiaXBCe0W4UMFIMZUWg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibugCfNmKJTaicwiczPT9bbqoNRf59bOSVauafkDLKu6uiaF7LuH6FHxicB9g/640?wx_fmt=jpeg)

_**No.4  
**_

_**删除影子用户**_

删除注册表 HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\ 下对应帐户的键值 (共有两处)

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUIKgLKC7HeU64N4diaEluibuvqVovhzfw91iaEW3BqAHlSJHiaxZA7aqkhBoKdaxIa28QrJsBWM0Smow/640?wx_fmt=jpeg)

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWDJyW3UCBiaNuUteDwicCG6vlaJsxhBpV3EgjHXbn1DnTQHuoRhsTxnbPtWMib5KdDOJxV8TY3vZzVg/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWDJyW3UCBiaNuUteDwicCG6vicyFHiaicHSHKTE7GlEaPpq7EZ9PTyAEicFlB9Fj2rYShbHf3d4k748PUA/640?wx_fmt=jpeg)