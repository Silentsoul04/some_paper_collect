> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KgCQ5I_ZQTlExZtaW9BLnA)

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LebHs2DGyKAEgZupcIbXWAgnQlIoLerewyAX3c3bLLg0iaTpJeUuGKrSWsicRvLMXwCIbhkUC8GqGibg/640?wx_fmt=gif)

原创稿件征集

  

邮箱：edu@antvsion.com

QQ：3200599554

黑客与极客相关，互联网安全领域里

的热点话题

漏洞、技术相关的调查或分析

稿件通过并发布还能收获

200-800 元不等的稿酬

曾经的我，以为挖掘 cnvd 是一个非常遥不可及的事情，注册资产 5000 万黑盒 10 案例（目前还不会代码审计吖，后续会努力学习吖！）这个门槛就已经难住我了（脚本小子的自述）。

事情的转机在于某次挖掘 edu 的过程中, 且漏洞都非常小白，无任何技术含量，师傅们看个热闹就好了，在校学生只想赚点生活费。呜呜呜呜~~~~~

主页界面常规漏洞测试一波，如弱口令，以及登陆框注入，均无果，发现还存在一处密码找回功能。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uQCgmPTfzRkdk1iarT39BSAnzVoyB2ibLVIpsB3IMEdloIw40t6d0ftlQ/640?wx_fmt=png)

找回密码界面。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40ubDEH2oLjI7ibMEWRxJfvee6WCw3Ltpia8ryE9yPIiatf1oUvR2PlHN4jQ/640?wx_fmt=png)

每个参数挨个测试一下最简单的注入加一个单引号’ 当然也仅仅限于为我这种偷懒型选手了，对于 int 型注入点如果做了报错处理可能无法探测，各位师傅们挖洞的时候不要学我一样偷懒哦。

运气果然很重要啊，阿巴阿巴，直接出货，且目标不存在 waf 直接上 sqlmap 跑了

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uEVIPhpOiaIbpVO2bRTyxMIXWZNUc1PPmkp3flyT6RNE1wdkcvh7vO2Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40usjQJwwUxVSroOwRYHO71WG1vLf4HFlEn5Jbkpic1BKs8bOoh01YYa8Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uib7m2uLnvPdSLK1clOAlEsUfdTY2BBelSJJLMrCHbWLQsW6ib5P5yolQ/640?wx_fmt=png)

Dba 权限 sqlserver 数据库，直接起飞，难得找绝对路径写 shell 了直接 os-shell 成功获取权限，而且可回显。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uOpYC0h0dIgn3pHyjEFyKgsxQL2pc6dTfDRAr0UOLfcKEjpH41LTGWQ/640?wx_fmt=png)

通过企查查查询了一下开发单位, 发现资产过 5000 万，接下来只要案例满足 10 例就可了，fofa 查询相关关键字，收集一波站点。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40ufliamvZuOicO6Iq0F9gOXkYoDqMtcBgVSDDx4Xwqyw8badFnzZSvibQfA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40u8hIS6CCQ3m0dDRFMYVeRSowW3Mk4dw0jf19O4hw4cRN5o425ib3Xk6g/640?wx_fmt=png)  

还算不错，阿巴阿巴，整理整理继续肝，一个良好的开端从有洞开始，  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40unVgnh2tZjo31xEJw5hXTf8fpnTfU4qbxtMCtS1mn9JgPTTDMRicQdoA/640?wx_fmt=png)

于是想着写一波脚本，批量跑一下 admin 账户的弱口令，这么多站点，我不相信没有一个弱口令，大概看了几个站点，发现一模一样的站，于是直接在某个站点本地 F12 看他登陆请求包，然后写脚本，批量跑一下  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uQOzzS9I0fUVtPjTIFMSQ9cob6uvdajytuMBaptibGv8vXOg8IvS0gcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uJ6XUE3PckkC2ibYcVVk5FL9crZS3xPD7ibz7wjtdibfDsJEJbAByYC1fg/640?wx_fmt=png)

又尝试了另一种方式，感谢 fengxsone 老表给我说的这个 selenium 自动化测试模块，阿巴阿巴，顺便拿来练练手，这里简单介绍一下呀, 百度无所不能。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40u7gtJbwyywAuuwicpja6basMSVGHD5kPj6T3pF8iaqzErVNE9Pua1WdaQ/640?wx_fmt=png)

通过 pip install selenium 或者在 pycharm 中搜索安装模块即可

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40u8CwXBztnJXiaBsDDWlUmgKia1KNNDibVeYSJhXR0h8G6Gqw7Z8ptNJesQ/640?wx_fmt=png)

这个模块呢，非常非常简单哈，安装完成之后，需要下载对应浏览器对应版本的驱动，之后就可以愉快的玩耍了，有兴趣的师傅，记得看开发手册吖，我觉得我讲的太 low 还是打扰了

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uAjzElFu9z13yD4ib9xbaxZEoCy9dVlBAd3qMmuWPTSRh1tkE5At071A/640?wx_fmt=png)

Run 一下之后就是一个慢慢等待的过程咯

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uRGjpJwQ9R3hhd4E9auEicK18obv9Kn9fNdXTfZKZdicT5dj80vXAZ12g/640?wx_fmt=png)

大约跑了 20 几个站点之后，成功使用 admin—123456 进入后台。我的脚本也 gg 了，没有写跳转的操作，我菜，阿巴阿巴

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uqTsRaQoGswVHcL6ibLQN3z5nuXWt1OvrzCd2Lbxd3yllKbibg8Jxl6Iw/640?wx_fmt=png)

进入后台后，一通操作，找到一处上传点，but 上传到的路径居然传到一台文件服务器上面去了，虽然能拿到一个文件服务器的 shell，但是估计其他站的文件也是传在这个服务器上面，溜了溜了。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40usqC5fUg3gb4tFkibXeibdiaJiabS2BjNql7n8b2pkubocmwHr16ibOyUUqw/640?wx_fmt=png)

继续 fuzz，注销登陆，之后重新登录同时使用 burp 抓包，单独拉出来放包，请叫我打码小王子。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uiacIMYDhlCzY5Smss1xQRcleV5gzIqKb9ZTC5vno6iar6uwm7AsqLiafA/640?wx_fmt=png)

发现并未存在 session 或者 token 参数。于是直接到其他站点，随意输入账户密码，替换返回包。直接越权到 admin 账户。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uCypQDw5J3WNTthUsawwP5d3bQbdakG5JlP42FOR6axOB1DcicInHfEQ/640?wx_fmt=png)

接着 fuzz，该测的测了，但在密码重置处倒是有点东东，身份证只要输入存在的账户，手机号输入自己的即可将验证码发送到自己的手机

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40usViarGJQAfRW4ntict6mvHN22aDT1kuOtqnRcgfsjo3rOZCAWYy4IaiaQ/640?wx_fmt=png)

所以说这里我们只需要获取到可登陆的身份证即可用自己的手机号接受到验证码，于是登陆，后台，在用户管理界面抓包。之后就是慢慢删除 cookie 的过程了，这里推荐一下 burp 的这个功能，贼好用

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40u0KAayzBvnbJzJgMBduvZbNNJxojU6gkzJ3NG3b3doxeW47hBiavy4hA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40um0oYnfqgNptBUmmZPXM1PSp4qOKa3Z6BWkGG3Rj7j1icCgmTjPSeeEg/640?wx_fmt=png)

在这个功能模块中，可以快速的 fuzz cookie 简直不要太舒服，最后发现只需要  

一个 schoolcon 参数，即可未授权访问，而且 schoocan 参数在加载页面时就已经有了  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uONW2bLd3TQuKtoBVqzSDCTYntBK1yMRL4ZFKM16Dn6fPFRhiczicemTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40uNibp2xibOeFbJVKP27IAnle9VyeMpJ2GH5ibk6d750BBRYoEakojhGWvw/640?wx_fmt=png)

整理开交，阿巴阿巴。

假装一段时间后~~~~~~~~~~~~~~~~~~~~~~ 证书下来了，嚯嚯，起飞。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfCluqPt6ZMQZ0VMfwAv40umBCtZxFlZDAEASHH8eBWOW5owgo34eDQQ0nIHVqtFtNlsP6WLeOIUw/640?wx_fmt=png)

阿巴阿巴，太没有技术含量了，各位师傅们见谅，才入门还要学习的东西太多了，有想要一起学习互相监督。

  
**本文推荐靶场实操：利用 sqlmap 辅助手工注入（复制链接做实验）  
**

https://www.hetianlab.com/expc.do?ec=ECID172.19.104.182015011915533100001&pk_campaign=weixin-wemedia#stu  

（本实验主要介绍了利用 sqlmap 辅助手工注入，通过本实验的学习，你能够了解 sqlmap，掌握 sqlmap 的常用命令，学会使用 sqlmap 辅助手工完成注入。）  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LePLZAbjmQXX3nEX3ZthS07AZ28WhA6ibrSdzypbyhH8cxZ23opOJYpV8TgkjG45SPLN5wzzBqx3Gw/640?wx_fmt=png)  

**0.02 元报名↓↓↓**

《渗透测试实战训练营》  

今天开课

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9Le0lxb4ntiagSXQ8xrfIWdJ2KsIBf9ia9ys7ibMuiaQeHDvkbnazEZ98s8fMQKZHQ63237cnU6W2wbXvw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LeNciacycV6QWlzNIldujmKCMJeicmGQgqATiaX6jPyT5RlUdXUkkt8sd4Sn1j5eXIqz179LBiabwv4Ow/640?wx_fmt=gif)

戳 “阅读原文” 报名渗透测试实战训练营