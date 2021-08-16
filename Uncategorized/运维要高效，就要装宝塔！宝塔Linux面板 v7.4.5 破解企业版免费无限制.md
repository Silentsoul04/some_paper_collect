> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/R7ou5EvTygkupcdsszlqYQ)

 ![释然IT杂谈](http://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPyicuicp7dubykiaj03kmHDugpzquGPeqiaEdNqn3ycJicLTQGWT1BaG033CabB1sY8FmW5BtFBcbyw5PQ/0?wx_fmt=png) ** 释然IT杂谈 ** 本公众号专注于分享网络工程（思科、华为），系统运维（Linux）、以及安全等方面学习资源，以及相关技术文章、学习视频和学习书籍等。期待您的加入~~~关注回复“724”可领取免费学习资料（含有书籍）。 185篇原创内容   公众号

有句俗话说运维要高效，就要装宝塔。   

那么，什么是宝塔？

宝塔面板（有Linux和Window两个版本）是非常优秀的PHP集成环境管理工具，可以让用户轻松选择Apache/Nginx，MySQL，PHP等不同版本安装与切换。

宝塔Linux面板是提升运维效率的服务器管理软件，支持一键LAMP/LNMP/集群/监控/网站/FTP/数据库/JAVA等100多项服务器管理功能。

宝塔面板自发布以来，因其功能强、易部署、易用性、安全性等方面，受到个人站长的欢迎，符合国人使用习惯，近几年迅速普及开来。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzPgzToEVkiaoGhaojtVgH7xSib0EE0QVnIxajVmE5ej4pDcQ80xD3AvAvsAqcc2cyTymLtkiayjqvtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**新版特性：**

  

  

  

宝塔Linux面板安装教程 – 2021年1月20日更新 – 7.5.1正式版

https://www.bt.cn/bbs/thread-19376-1-1.html

宝塔linux面板命令大全 – 宝塔面板

https://www.bt.cn/btcode.html

  

  

  

**版本区别：**

宝塔免费版：

**近200个免费插件，常用有：网站管理、系统安全、系统监控、计划任务、文件管理、软件管理、一键部署等等。通过免费版就可以实现点击鼠标轻松建站的目的。比以前的命令行面板容易多了，对新手特别友好。**

宝塔专业版：

**专业版提供了很多付费插件，包括宝塔系统加固、网站防篡改程序、网站监控报表、Apache防火墙、Nginx防火墙、宝塔负载均衡、MySQL主从复制、宝塔任务管理器、异常监控推送、微信小程序、宝塔数据同步工具等等插件，可以从安全、维护、性能等等多方面维护网站和服务器平稳运行。**

宝塔企业版：

**企业版插件除了包含以上内容之外，额外还包括多用户管理、socks5代理服务器，另外有专属官方QQ群提供一对一服务，还有专属运维顾问提供快速响应服务，助力企业业务平稳运行。**

**破解说明：**

  

  

  

  

  

*   * 去除面板与宝塔官方的所有通信、数据上报、下发接口；全部官方插件免费无限制使用
    
*   安装脚本后无需注册宝塔账号，默认为宝塔企业版，宝塔开心版可以使用所有付费插件！
    

2021.03.02 Update7.sh

  

  

  

  

  

  

  

1.  同步官网最新版本
    
2.  完全去掉上报功能，接口调用注释
    
3.  修改登录接口,不验证，不记录， 默认返回绑定号码18988889999
    
4.  默认关闭活动推荐及在线客服功能
    
5.  修复升级通道，支持后续纯净版升级
    
6.  修复面版的修复功能
    
7.  增加三方组件安装，免费版全支持，收费版三方组件 部份已支持免费下载安装使用，包括但不限于：Cloudflare解析，CloudFlare 批量设置ip，站长管理工具，站群管理工具，内网穿透神器，宝塔API调用BOT等…三方组件仅测试安装、可用，并未测试实际功能。
    

支持Centos,  Debian，理论上宝塔支持的系统，宝塔净化版脚本都支持！！！

  

  

**安装命令：**

  

  

  

  

v7.5.1 纯净版内测版本（极速安装方式 (安装时间1至10秒)，升级后可能需要重启面板）  

如果已经升级或安装官方v7.5.1正式版，再执行此脚本即可一键升级净化升级为宝塔纯净版；

此升级安装过程，不影响网站数据，不宕机，不修改任何网站数据，无缝切换，建议先测试

  

  

**Centos/Debian升级命令：**

```
wget -O /home/update7.sh http://download.hostcli.com/install/update7.sh && bash /home/update7.sh
```

**Centos安装命令：**

根据系统执行框内命令开始安装（大约2分钟完成面板安装）  

```
yum install -y wget && wget -O install.sh http://download.hostcli.com/install/install_6.0.sh && sh install.sh
```

**Debian安装命令：**

```
wget -O install.sh http://download.hostcli.com/install/install_6.0.sh && bash install.sh
```

**还原命令：**

如需还原官方版本，在SSH窗口执行下列命令：

```
curl http://download.bt.cn/install/update6.sh|bash
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzPgzToEVkiaoGhaojtVgH7xWXR9K6rH5o4FqV3ZSzVNaRBq3dbcVA3zKXEFK25LN63Bcpx2heBGicg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzPgzToEVkiaoGhaojtVgH7xLSDqXRo1zVHYCDgrfaksmBziajxfRVvrngh8Twpw7COKuKibPNApyeTQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzPgzToEVkiaoGhaojtVgH7xuUeyttKdDpFicMWMWwE2TAYvMsvgB45pWib6gPTJBZmTVOaek40J0Eiag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

           

更多推荐

           [

  

干货！常用安全标准又更新了V5和V1.5版



](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247492953&idx=1&sn=273df2c754b1bda10b5560c0b6604138&chksm=9758a020a02f2936953a2bb57ee8b69bf0ccf8a8f594b8d80a7945ac65ee2a30ea5da71203c5&scene=21#wechat_redirect)[

  

网工必备网络排错管理工具之IP链路测试工具



](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247492940&idx=1&sn=8fed62092e66a5aff71e1fbd7a6e91e7&chksm=9758a035a02f2923479efee325345171fce1acd9ab6a795b18eb3ee7fc33d88c7472d9297177&scene=21#wechat_redirect)[

  

福利又来了，35本精选书籍任你挑选！



](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247492966&idx=1&sn=d88ac6a86a5a7e256c9cecad80a6d963&chksm=9758a01fa02f29096b9b93ed05776f58cd18d8a865760813e0aaa81e6500fbe325faaad2076d&scene=21#wechat_redirect)[

  

渗透测试在线工具集合



](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247492886&idx=1&sn=6ff8dd497d126187f9657b088e903427&chksm=9758a06fa02f29799e590c44d4276ba564195b1c17cb58f1e469b1ad0babf1796c2db0f74b02&scene=21#wechat_redirect)[

  

实战干货-现网环境中，路由重分发的标准解决方案



](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247492828&idx=1&sn=14295ffba73ffe5f23ba50a74d1f82ac&chksm=9758a1a5a02f28b30f7d154a6bd7a2de6dae86ede8cd302ebf34e84be80ba51afcafca0873a1&scene=21#wechat_redirect)

[  
](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247492814&idx=1&sn=c9c3f428ec3f566457d60ff6dcd7ee49&chksm=9758a1b7a02f28a192838ea18c355391ba637488f992478508c64bd153b20231bc8554b415b2&scene=21#wechat_redirect)

[这些黑客经常挂在最边的网全“黑话”，你知道多少？](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247492814&idx=1&sn=c9c3f428ec3f566457d60ff6dcd7ee49&chksm=9758a1b7a02f28a192838ea18c355391ba637488f992478508c64bd153b20231bc8554b415b2&scene=21#wechat_redirect) 

【更多精彩扫描加入】

![图片](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzCWice1Y8p1bdRNj1ia9mnqXmk7sSHiasP3r4pRyV3v3WxCT1UIFqKmD1F4g64cjHbgPhmkj6vbK1FA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)