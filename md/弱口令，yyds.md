> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/nJ-blIy0JTJ7Dz0WuqrEYQ)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9Zrec46UFTSjH6JUkm88icDRQc4AYtjsTtvQZw67ROszs8OP3l7HpIl8A/640?wx_fmt=png)

一些自己搜集的弱口令

在渗透企业资产时，弱口令往往可达到出奇制胜，事半功倍的效果！特别是内网，那家伙，一个 admin admin 或者 admin123 拿下一片，懂的都懂。

当你还在苦恼如何下手时，我却悄悄进了后台，getshell 了

以下是我实战中，经常遇到的一些口令，希望大家记好笔记，弱口永远的 0Day，文末有常用字典，和查询网站。

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9Zrec46UFTSjH6JUkm88icDRQc4AYtjsTtvQZw67ROszs8OP3l7HpIl8A/640?wx_fmt=png)

tomcat

重所周知，tomcat 有项目部署管理界面

```
 http://ip:8080/host-manager/html
```

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZOln5I1g0S94K36zZmSFetCzNYrMGKCWoaflpJxNLWEqI72wDmgVnJw/640?wx_fmt=png)

```
用户名：admin  密码：admin
用户名：tomcat 密码：tomcat
用户名：tomcat 密码：s3cret
```

Apache axis2

axis2 同样和 tomcat 有 web 控制台

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9Z9WiaHAvibiaIicC6ge7wrLmoyURMtcm5jOzq9ybojwn79vSDtJmibgIiasNw/640?wx_fmt=png)

```
用户名：admin   密码：axis2
```

websphere

这个很少见，不过还是被我见到了

9080 端口

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZLayqbzyGnGNaxlQnAtNu2PEGL5pNth657CD3hBEZ2jWzQKLIOLzauQ/640?wx_fmt=png)

```
system/manager
wasadmin/wasadmin
admin/admin
```

MapZone Server Login

MAPZONE Server 是公司自主开发的一套服务型 GIS 平台产品, 产品基于面向服务架构, 提供数据服务发布、功能服务发布、外部服务聚合、定制化服务等功能

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9Z0zlibBoa9k4P25cfgx5NnoKXebiaJxzasiaEFTN0jDhYbBL1bia2TkZHibg/640?wx_fmt=png)

```
MapZone/MapZone
```

HIKVISION

sese 发抖的摄 (se) 像头

HIKVISION 默认密码 12345，但是安装的时候都会提示更改默认密码，这个只能靠运气

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9Zibmz82bsk52DKQvoiaYq10sicyXPjj0SWoM4hsY1nicjXsk8GqUhXvp5icw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9Z7hvuL2k9o5FuG8g8oq0jHykSk9jq60baG1aJHw4CemvkzLBeAMqZIw/640?wx_fmt=png)

vmwarw harbor

Harbor 是一个用于存储和分发 Docker 镜像的企业级 Registry 服务器，通过添加一些企业必需的功能特性，例如安全、标识和管理等

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZxFy97DiafxAMSuRLYUvsRmd8RxWqaFqKnXj1Y9kibMqiaLOAFtia1AOU0A/640?wx_fmt=png)

```
账号admin 密码Harbor12345
```

深信服设备

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9Z1d9HhmDhicworfda02EZCVMfO1bTFyKmZUiaxUnL0LMdOrmotC5usSlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZcAzBj5q67zarABkMYO4CVym45Xq68XKtRh6t1fnEbAjvsrAw0geicfw/640?wx_fmt=png)

```
用户名：sangfor
密码：sangfor
             sangfor@2018
             sangfor@2019
             admin/admin
             admin/空
```

华为设备

华为交换机

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZM0z75PTwqjkWVSwTyNG4IR7icgRY2LylVUIQlMJBzLftg4e5pE6QFfw/640?wx_fmt=png)  

```
用户名：admin   
密码：admin@huawei
密码：admin@huawei.com
```

华为 USG2250

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZE9OvMXpW3ibHvvHiaeeITgcwybXOnhgib1iaPFqXPhibcP7Dyl0X6Y1qFcg/640?wx_fmt=png)

```
admin/Admin@123
```

zentao

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZDnvC8Um4Gqiasb817fUUL0tFGV8AoMvtERpMcDibIiaIMOPIl8pftOxKQ/640?wx_fmt=png)

```
zentao 用户名：admin   密码：123456
```

Apache ActiveMQ

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZsRMibNAUpcFTdsVlVbvXan71r6vNT7zTPSUco4dwKxBZ1xQbfo6l2ew/640?wx_fmt=png)

```
未授权/admin/connections.jsp
用户名：admin  密码：admin
```

zabbix

某 src 真实遇到过，这是一个基于 WEB 界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案

更可反弹 shell 拿下权限哦

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZEN4aUwyPWiaIDbI0wqnhQvgrON0htu34NfASVzy80jWjrBYJKOPQqaA/640?wx_fmt=png)

```
用户名：admin   密码：zabbix
```

RabbitMQ  

RabbitMQ 是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODuU33NY3phUc3ibu0W1DVT9ZRjCasZNCUGodRPicN2JUrS9UW3JA6F6YP71QWl74DvJeoc4g5zZKVxQ/640?wx_fmt=png)

```
默认账号密码guest/guest  admin/admin
```

**再奉上常见的弱口令查询网站（怎么叫弱口令勒？，应该叫默认密码和账号）：**

```
HUAWEI 默认账号/密码查询工具：
https://support.huawei.com/onlinetoolweb/pqt/index.jsp
路由器密码社区数据库：
http://www.routerpasswords.com
默认路由器密码列表：
https://portforward.com/router-password/

然后就是百度，google了，好不好使，懂的都懂嗷
```

再附上一波字典，大家直接复制粘贴保存吧：

```
深信服产品  sangfor  sangfor sangfor@2018 sangfor@2019
深信服科技 AD     dlanrecover
深信服负载均衡 AD 3.6  admin  admin
深信服WAC ( WNS V2.6)  admin  admin
深信服VPN  Admin  Admin
深信服ipsec-VPN (SSL 5.5)  Admin  Admin
深信服AC6.0  admin  admin
SANGFOR防火墙  admin  sangfor
深信服AF(NGAF V2.2)  admin  sangfor
深信服NGAF下一代应用防火墙(NGAF V4.3)  admin  admin
深信服AD3.9  admin  admin
深信服上网行为管理设备数据中心  Admin  密码为空
SANGFOR_AD_v5.1  admin  admin
网御漏洞扫描系统  leadsec  leadsec
天阗入侵检测与管理系统 V7.0  Admin  venus70
   Audit  venus70
   adm  venus70
天阗入侵检测与管理系统 V6.0  Admin  venus60
   Audit  venus60
   adm  venus60
网御WAF集中控制中心(V3.0R5.0)  admin  leadsec.waf
   audit  leadsec.waf
   adm  leadsec.waf
联想网御  administrator  administrator
网御事件服务器  admin  admin123
联想网御防火墙PowerV  administrator  administrator
联想网御入侵检测系统  lenovo  default
网络卫士入侵检测系统  admin  talent
网御入侵检测系统V3.2.72.0  adm  leadsec32
   admin  leadsec32
联想网御入侵检测系统IDS  root  111111
   admin  admin123
科来网络回溯分析系统  csadmin  colasoft
中控考勤机web3.0  administrator  123456
H3C iMC  admin  admin
H3C SecPath系列  admin  admin
H3C S5120-SI  test  123
H3C智能管理中心  admin  admin
H3C ER3100  admin  adminer3100
H3C ER3200  admin  adminer3200
H3C ER3260  admin  adminer3260
H3C  admin  adminer
   admin  admin
   admin  h3capadmin
   h3c  h3c
360天擎  admin  admin
网神防火墙  firewall  firewall
天融信防火墙NGFW4000  superman  talent
黑盾防火墙  admin  admin
   rule  abc123
   audit  abc123
华为防火墙  telnetuser  telnetpwd
   ftpuser  ftppwd
方正防火墙  admin  admin
飞塔防火墙  admin  密码为空
Juniper_SSG__5防火墙  netscreen  netscreen
中新金盾硬件防火墙  admin  123
kill防火墙(冠群金辰)  admin  sys123
天清汉马USG防火墙  admin  venus.fw
   Audit  venus.audit
   useradmin  venus.user
阿姆瑞特防火墙  admin  manager
山石网科  hillstone  hillstone
绿盟安全审计系统  weboper  weboper
   webaudit  webaudit
   conadmin  conadmin
   admin  admin
   shell  shell
绿盟产品     nsfocus123
TopAudit日志审计系统  superman  talent
LogBase日志管理综合审计系统  admin  safetybase
网神SecFox运维安全管理与审计系统  admin  !1fw@2soc#3vpn
天融信数据库审计系统  superman  telent
Hillstone安全审计平台  hillstone  hillstone
网康日志中心  ns25000  ns25000
网络安全审计系统（中科新业）  admin  123456
天玥网络安全审计系统  Admin  cyberaudit
明御WEB应用防火墙  admin  admin
   admin  adminadmin
明御攻防实验室平台  root  123456
明御安全网关  admin  adminadmin
明御运维审计与册风险控制系统  admin  1q2w3e
   system  1q2w3e4r
   auditor  1q2w3e4r
   operator  1q2w3e4r
明御网站卫士  sysmanager  sysmanager888
亿邮邮件网关  eyouuser  eyou_admin
   eyougw  admin@(eyou)
   admin  +-ccccc
   admin  cyouadmin
Websense邮件安全网关  administrator  admin
梭子鱼邮件存储网关  admin  admin
```

弱口令好不好用，谁用谁知道啊，实在不行，top1000 密码字典跑一波，跑不出来就算了！你这站不日也罢，大家都讲究佛系日站，命里有就有，没就没!  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png)

END

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtIZ5VYusLbEoY8iaTjibTWg6AKjAQiahf2fctN4PSdYm2O1Hibr56ia39iaJcxBoe04t4nlYyOmRvCr56Q/640?wx_fmt=gif)

**看完记得点赞，关注哟，爱您！**

**请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

关注此公众号，回复 "Gamma" 关键字免费领取一套网络安全视频以及相关书籍，公众号内还有收集的常用工具, 原创工具！

  

**在看你就赞赞我！**

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbThXaInFkmyjOOcBoNCXGun5icNbT4mjCjcREA3nMN7G8icS0IKM3ebuLA/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTkwLkofibxKKjhEu7Rx8u1P8sibicPkzKmkjjvddDg8vDYxLibe143CwHAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/96Koibz2dODuKK75wg0AnoibFiaUSRyYlmhIZ0mrzg9WCcWOtyblENWAOdHxx9BWjlJclPlVRxA1gHkkxRpyK2cpg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif)

扫码关注我们

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif)

扫码领 hacker 资料，常用工具，以及各种福利

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTnHS31hY5p9FJS6gMfNZcSH2TibPUmiam6ajGW3l43pb0ySLc1FibHmicibw/640?wx_fmt=gif)

转载是一种动力 分享是一种美德