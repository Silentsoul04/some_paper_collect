> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iWI5GfCPFaW6SvuPduX0BQ)

**WAF 简介**

WAF 对于一些常规漏洞（如注入漏洞、XSS 漏洞、命令执行漏洞、文件包含漏洞）的检测大多是基于 “正则表达式” 和“AI + 规则”的方法，因此会有一定的概率绕过其防御。

绕过 waf 的测试中，有很多方法可以使用，以下常用方法： 

```
大小写绕过
 注释符绕过
 编码绕过
 分块传输绕过
 使用空字节绕过
 关键字替换饶过
 http协议覆盖绕过
 白名单ip绕过
 真实ip绕过
 Pipline绕过
 参数污染
 溢出waf绕过
```

可以把 WAF 分为四类：

```
云WAF类
硬件WAF类
软件WAF类
网站内置WAF类
```

云 waf 基于云端的检测，安装简单，修改 DNS 解析或在服务器安装云 WAF 的模块即可。

硬件 WAF 串联在内网的交换机上，防护范围大。

软件 WAF 安装在服务器上根据网站流量决定占用的内存量。

网站内置 WAF 在系统后台内置一项安全功能以便管理者使用。在这些类别内，硬件 WAF 防护能力较强。

**软件型 WAF:**

```
D盾：http://www.d99net.net/
云锁：http://www.yunsuo.com.cn/
网防：http://www.weishi110.cn/static/index.html
安全狗：https://www.safedog.cn/
护卫神：https://www.hws.com/
智创：https://www.zcnt.com/
悬镜：https://www.xmirror.cn/
UPUPW:https://www.upupw.net/
WTS-WAF:https://www.west.cn/
安骑士：https://help.aliyun.com/product/28449.html
dotDefender:http://www.applicure.com/Products/
```

**硬件型 WAF:**

```
绿盟：https://www.nsfocus.com.cn/
安恒 https://www.dbappsecurity.com.cn/
铱(yi)迅https://www.yxlink.com/
天融信
深信服
启明星辰
知道创宇
F5 BIG-IP:https://www.f5.com/
```

**基于云 WAF:**

```
安全宝：
创宇盾：https://defense.yunaq.com/cyd/
玄武盾
腾讯云
百度云
西部数码
阿里云盾：
奇安信网站卫士:
```

**开源型 WAF**

```
Naxsi:https://github.com/nbs-system/naxsi
OpenRASP:https://github.com/baidu/openrasp
ModSecurity：https://github.com/SpiderLabs/ModSecurity
https://github.com/SpiderLabs/owasp-modsecurity-crs
```

**@TOC**

**WAF 比较常见的监测机制特点有以下几种。**

（1）异常检测协议：拒绝不符合 HTTP 标准的请求，也可以只允许符合 HTTP 协议的部分选项通过，也有一些 web 应用防火墙还可以限定 http 协议中那些过于松散或未被完全制定的选项。

（2）增强输入验证：增强输入验证，对恶意字符进行拦截。

（3）及时补丁：及时屏蔽掉新型漏洞，避免攻击者进行攻击，主要依靠 WAF 厂商对新型漏洞的及时响应速度

（4）基于规则的保护和基于异常的保护：基于规则的保护可以提供各种 web 应用的安全规则，waf 生产商会维护这个规则库，并及时为其更新。用户可以按照这些规则对应用进行全方面检测。还有的产品可以基于合法应用数据建立模型，并以此为依据判断应用数据的异常。但这需要对用户企业的应用具有十分透彻的了解才可能做到

（5）状态管理；能够判断用户是否是第一次访问，将请求重定向到默认登录页面并且记录事件，或对暴力破解行为进行拦截。

（6）其他防护技术：如隐藏表单域保护、抗入侵规避技术、响应监视和信息泄露保护。

（7）配置规则：可以自定义防护的规则，如是否允许 “境外 ip” 的访问

**常见 WAF 进程和服务**

**(1) D 盾**

服务名：d_safe

进程名：D_Safe_Manage.exe、d_manage.exe

**(2) 云锁**

服务端监听端口：5555

服务名：YunSuoAgent/JtAgent（云锁 Windows 平台代理服务）、YunSuoDaemon/JtDaemon（云锁 Windows 平台守护服务）

进程名：yunsuo_agent_service.exe、yunsuo_agent_daemon.exe、PC.exe

**(3) 阿里云盾**

服务名：Alibaba Security Aegis Detect Service、Alibaba Security Aegis Update Service、AliyunService

进程名：AliYunDun.exe、AliYunDunUpdate.exe、aliyun_assist_service.exe

**(4) 腾讯云安全**

进程名：BaradAgent.exe、sgagent.exe、YDService.exe、YDLive.exe、YDEdr.exe

**(5) 360 主机卫士**

服务名：QHWafUpdata  

进程名：360WebSafe.exe、QHSrv.exe、QHWebshellGuard.exe

**(6) 网站 / 服务器安全狗**

服务名：SafeDogCloudHelper、SafedogUpdateCenter、SafeDogGuardCenter（服务器安全狗守护中心）

进程名：SafeDogSiteApache.exe、SafeDogSiteIIS.exe、SafeDogTray.exe、SafeDogServerUI.exe、SafeDogGuardCenter.exe、CloudHelper.exe、SafeDogUpdateCenter.exe

**(7) 护卫神 · 入侵防护系统**

服务名：hws、hwsd、HwsHostEx/HwsHostWebEx（护卫神主机大师服务）  

进程名：hws.exe、hwsd.exe、hws_ui.exe、HwsPanel.exe、HwsHostPanel.exe/HwsHostMaster.exe（护卫神主机大师）

**(8) 网防 G01 政府网站综合防护系统（“云锁” 升级版）**

服务端监听端口：5555

服务名：YunSuoAgent、YunSuoDaemon（不知是否忘了替换了！） 

进程名：gov_defence_service.exe、gov_defence_daemon.exe

**WAF 识别**

WAF 绕过不仅要了解 WAF 检查的原理，还需要识别是什么类型的 WAF，不同类型，不同品牌的 waf 监测机制不一样，绕过的方式也不同。

**(1) wafw00f/WhatWaf**

利用 wafw00f 识别 WAF，可以在 WAF 指纹目录下自行编写脚本。这类 WAF 识别工具的原理基本都是根据 HTTP 头部信息、状态码以及 WAF 拦截页中的图片、文字做为特征来进行检测，如 wafw00f 工具中的 yunsuo.py 脚本就是根据 cookie 中的 security_session_verify 来检测的。

```
/usr/lib/python3/dist-packages/wafw00f/plugins
```

```
#!/usr/bin/env python

NAME = "Yunsuo"
def is_waf(self):
 if self.matchcookie("^security_session_verify"):
 return True
 return False
```

**(2) sqlmap -identify-waf**

利用 sqlmap -identify-waf 参数识别 WAF，一样可以在 WAF 指纹目录下根据原有脚本和 Awesome-WAF 项目自行编写 WAF 指纹识别脚本，但有时可能会因为 sqlmap 新老版本的原因而导致存放路径不一样。

```
更新前：/usr/share/sqlmap/waf
```

```
更新后：/usr/share/golismero/tools/sqlmap/waf
```

```
#!/usr/bin/env python
"""
Copyright (c) 2006-2013 sqlmap developers (http://sqlmap.org/)
See the file "doc/COPYING" for copying permission
"""
import re
from lib.core.enumsimport HTTP_HEADER
from lib.core.settingsimport WAF_ATTACK_VECTORS
__product__ ="ModSecurity: Open Source Web Application Firewall (Trustwave)"
defdetect(get_page):
 retval =False
for vectorin WAF_ATTACK_VECTORS:
 page, headers, code = get_page(get=vector)
 retval = code ==501and re.search(r"Reference #[0-9A-Fa-f.]+", page, re.I)isNone
 retval |= re.search(r"Mod_Security|NOYB", headers.get(HTTP_HEADER.SERVER,""), re.I)isnotNone
if retval:
break
return retval
```

**(3) 项目地址**

https://github.com/sqlmapproject/sqlmap

https://github.com/EnableSecurity/wafw00f

https://github.com/Ekultek/WhatWaf

https://github.com/0xInfection/Awesome-

```
原创作者：GuiltyFet
https://blog.csdn.net/weixin_51387754/article/details/116535215
欢迎关注作者博客。
```

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC7IHABFmuMlWQkSSzOMicicfBLfsdIjkOnDvssu6Znx4TTPsH8yZZNZ17hSbD95ww43fs5OFEppRTWg/640?wx_fmt=gif)

●[干货 | 渗透学习资料大集合（书籍、工具、技术文档、视频教程）](http://mp.weixin.qq.com/s?__biz=MzIwMzIyMjYzNA==&mid=2247490892&idx=2&sn=5820f8871f23ffc525a27e1c6ae1ae4c&chksm=96d3e649a1a46f5f88051b88fb05efd4cda4c885a4f47ac63795354cbfe5e3a93de747f3f10a&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3b9EpYTX241vj3pudtgtj7V0XFvzpxP5tBHCOtpXZmmHcpPBq4STTxVe56CdHHUb8hmAD6fzRrpA/640?wx_fmt=jpeg)

**关注公众号: HACK 之道**

公众号

如文章对你有帮助，请支持点下 “赞”“在看”