> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/URTsHxKgpTvRONSMvvJDLA)

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LebHs2DGyKAEgZupcIbXWAgnQlIoLerewyAX3c3bLLg0iaTpJeUuGKrSWsicRvLMXwCIbhkUC8GqGibg/640?wx_fmt=gif)

原创稿件征集

  

邮箱：edu@antvsion.com

QQ：3200599554

黑客与极客相关，互联网安全领域里

的热点话题

漏洞、技术相关的调查或分析

稿件通过并发布还能收获

200-800 元不等的稿酬

**前言**  

前一段时间都在挖 edu src，为了混几个证书，中间陆陆续续也挖到好几枚系统的通杀吧，不过资产都不多，都是黑盒测试出来的，没啥技术含量。只有这次挖到的这枚通杀稍微有那么一点点价值，从外网 web 一步步深入最后服务器提权，拿下整台服务器桌面权限。

**1. 信息搜集**

日常广撒网挖通杀，常规流程，上 fofa 搜索关键字，xx 大学 xx 系统，xx 大学 xx 平台，一般就是这几个关键词，或者是直接搜 body=”xx 公司”，xx 公司一定要是经常给学校做开发的，往往都是好几所学校用同一家公司的产品。然后就找到了这样一个系统

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEppbe6SxUWNdiae0GkE5kTjNc71I4ShzUs86M2KKP8a4icUt0X3zy4aOA/640?wx_fmt=png)  

查了下归属，归属是某某学院，教育资产，通过各种语法，信息搜集，找到大概十多所学校都在用这个系统，因为语法太多了，这里随便搜了搜。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEVw8HXUgicv6XPnNRsjnHmBLDTOCJ00cNZz5nR9Dflxwicia89uEHoRxCg/640?wx_fmt=png)

**  
2. 四处碰壁**

正常的黑盒测试流程，看一下啥语言写的，ASP+IIS，很常规的配置，edu 一般除了 jsp 就是 asp 了，很少见到 php 站，iis 的站，若后续有文件上传的点，可以测测 iis 解析漏洞，老版本的 iis 洞还是挺多的。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEkW1PRKEql582c4qnES7ZPIIInsTicLfxZVjPAPphibtyw05DfibmqiaoiaA/640?wx_fmt=png)  

既然是 asp 的站，那就上御剑，先来一顿目录爆破，asp、aspx 勾选上，80w 的大字典开跑，一杯茶的功夫，目录爆破完毕，果不其然，啥也没跑出来。  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEAjQwWQWXuSlhPasu27rjktRYVlCzZDHbFHw9usXAMEJle1VeGjdj0w/640?wx_fmt=png)

一般这种情况的话，可以换一换要跑目录，因为它整个系统可能架设在一个特定命名的目录下，这里因为时间关系，就没跑了。

既然目录爆破不行，这系统打开就是登录点，那就爆破登陆点试一试，各种用户名都爆破了一遍  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEaPn5LYamTnAF8nibPjEeu08hSticYgiaSWBZsNpEzCmwnwZlzgThFsDwA/640?wx_fmt=png)

还是失败了，一个弱口令都没爆破出来，学号，工号爆破都试过了，没有一个成功的，目前为止，目录爆破，密码爆破都走不通。

Sql 注入，post 注入，常规操作，果然。。。。又是一片红，必然做了过滤，简单的 fuzz 了下 sql 语句饶了绕，还是失败。  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTE0ewoDhyjQNyvvHocRvh1MuopBSPHicHoBcxQiblsbnQAMMib7XRgxdaOg/640?wx_fmt=png)

各种操作都来了一波，啥也没挖到，在挖 edu 的这段时间里，经常遇到这种情况，都习惯了。

既然注入也没有，还有过滤，那就测测逻辑漏洞，右下角找回密码，我可太喜欢找回密码了，找回密码处就是逻辑漏洞的高发地点，一打一个准，  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEibBppRy65u4CndyZTMmBN7upgK1IiakwdtUQQGicOgmO2ibfvs4FoGoKww/640?wx_fmt=png)  

点进去是这样一个页面，挺简陋，越是简陋，就越好打，果断输入答案，抓包。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEtibSAbnugGToKOp9icBaIGQpdoiaw4GbynPKYd5iaa0nd9ACZj9pVHa05A/640?wx_fmt=png)  

没啥好看的，要是返回包里是 json 格式的话，那还有得玩。反正我遇上的逻辑漏洞，都是前端验证传回来的 json 参数，改 json 实现绕过。  

**3. 柳暗花明（发现 sql 注入）**

Sql 注入，爆破，弱口令，逻辑漏洞都试过了，都失败了，正准备放弃的时候，我发现找回密码的时候，他这个系统有个特点，只要你一输入要找回的账户然后再换行，本来它设置问题那一栏是空的，在你输入完账号再换行时，它问题那一栏自动就出现了验证问题。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEY0XUnPm0XFEsribNRNcj7qE1tfPwJe63V96QM8kQTeCpoU3eRtBlb0A/640?wx_fmt=png)

所以我推断，在用户输入完账号之后换行就触发了一个动作，这个动作会自动将用户输入的账号带入到后台，从后端获取这个账号的问题，然后再显示在前端，必然有数据交互的一个过程，既然有交互，那么这个点也可能存在注入的可能。  

想到这里，打开 burp，输入完账号之后不换行，切换至 burp，抓包，然后再换行，触发动作，果然抓到了一个 post 包，请求内容正是账号

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEicOn44RKYib5iaWRaRdlAv1EZZpfwewuic2os2Gy174yPgv7uO650xqIPQ/640?wx_fmt=png)

输入一个单引号，发现报错了, 存在注入无疑了，这系统普通的登录点卡的死死的，还是被找到注入了，只不过这个注入的位置太奇葩了，一般人遇上 waf 就放弃了。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEeFZSgnJvWrmvbVFl3ic3gfafLcaiagxiaEyYTgvUAwnHou46RcUZ3yXUA/640?wx_fmt=png)

Sqlmap 一把梭，发现是 mssql，还是 dba 权限，不用想了，mssql+dba 权限 = xp_cmdshell，都不用进后台了。--os-shell

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEpPVRqzR65mJVKCHmWVebsUuYeKJLkSBAcwq5aXnGCf7jjNiaxfVRFDA/640?wx_fmt=png)  

**4.bypass 上线 cs 并提权**

过程就不放图了，简单描述一下，用的是 certutil.exe -urlcache -split -f 下载 cs 马，cs 马在我的服务器上，刚开始下载文件的时候，报错，whoami 一看，数据库权限，只读权限，没有写文件的权限，这可麻烦了

最后解决办法是，把 cs 马下载到 mssqlserver 用户的桌面目录下，其他路径没有执行下载的权限，在自己用户的桌面目录从有写文件的权限了吧？执行 cs 马，cs 上线！

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEgIlqwGAceGzrcw6tvYXj5NlJibiaNZJfMNUXXlsuqGvLRwxAibTYiavoMA/640?wx_fmt=png)

虽然拿到了 shell，不过这个 shell 的权限实在太低了，dumphash 报错，操作注册表就各种报错，反正啥操作的报错，因为权限太低

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEI1Aw8ibXUOJR5AAIbDpkSrv5YKcn1FkRTxO8VDVmwzZJomhjeMahE2Q/640?wx_fmt=png)

如今当务之急就是提权，先执行一下 systeminfo、tasklist 看看啥情况

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEpMRN28kcaALiaZdrO8ZTicqOH4AADyCfjbdiaLdCmbFhFHDKom2MdmTxA/640?wx_fmt=png)

Server 2012 的机器，补丁实在打的有点多，吓人。Tasklist 里也没发现有杀毒软件，估计是云 waf

2012 的机器内核漏洞算是最多的了，来来回回试了几个 MS16-032/016, 全打上补丁了，最后一个 MS16-075, 一把打穿，成功拿下 system 权限，2012 的机器还是好提。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEKTX7vpwicItGiacJVB6n47oYhUic0S51iayS97T7uUGZ8hGiay4kC2GLz5A/640?wx_fmt=png)

**Bypass 远程桌面组获取桌面控制权**

执行一下 netstat -ano 发现开了 3389 端口，net user 发现一堆的用户，这里就不放图了，不然篇幅实在太长了，简单的信息搜集之后，开始办正事，目标是桌面控制，上神器 Mimikatz，抓一抓明文密码。这里稍微提一下，2021 的机器是可以通过改注册表直接获取明文密码的，一抓发现管理员上次是 5.3 登录的，没抓到密码，只有 hash

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEYssIVsxjFxIAzYJasL6j4zVoMXiag8QdsHMIAQ7wvUtj21BGFbicSBTg/640?wx_fmt=png)

抓不到明文密码，那就新建用户，net user admin 123456 /add 新建用户，新建了一个 admin 密码 123456 的用户。远程桌面连一下试试。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEXpyF9ibq209iavIK3XLd3XnwcXVXeobyHxibUyUyJkXas7Ux0lekea9iaA/640?wx_fmt=png)  

出现报错：“连接被拒绝，因为没有授权此用户帐户进行远程登录！以为就要成功了，这一个报错就像是当头一棒，找了找原因，是因为我新建的用户没有加入到远程桌面组，所以无法登录，

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTE2GIOlfKO5K86u4P3dzic5SRds6nyOwCTQKATRFAEOTp8jFONczj2icoA/640?wx_fmt=png)  

用 net user 把 admin 加入到远程桌面组之后，还是报错，我又修改注册表把防火墙关了，RDP 规矩也放行了，无果.. 我猜可能是修改完配置之后要重启才会生效，我要是重启的话，这台服务器上的这套系统必然会瘫痪，重启是肯定不可取的。

在思想斗争了半天之后，我想到 guest 用户应该是默认就在远程桌面组的，我只要激活 guest 用户，那我就可以不重启就连 3389 了。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEuZ1XcXxB7cwjpLVNx7mGNicjJlvEDc20PvfYgiapOv1JNPJ2wgy4YnTw/640?wx_fmt=png)

激活 guest 用户成功，密码 123456，远程连接一下

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEAchsXHkNQ2icug4IzeEp7ruIDFvHTbXTNIoeugzZdpGbEDolsuZibdyQ/640?wx_fmt=png)

一看到这个正在配置远程会话就知道稳了，3389 成功上了桌面，guest 权限，加了个隐藏账户，并手动加入到远程桌面组

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEqVV6bUdiahJGmAac7EctADILHQ5UxUZqqgeOAkRjev7pTP2L9rWibydA/640?wx_fmt=png)

5.RDP 劫持失败
----------

我们的目标是 administrator 的桌面控制权，但是密码抓不到，又不能重置 administrator 的密码，怎么拿下它的桌面？

这里我用了 RDP 劫持，上传一个 psexec 工具，然后获取一个 system 权限的 cmd，因为只有 system 权限的命令行才能进行接管会话

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEbjdVnibJiasufrfq7mhuDDGgWGjZgxZcRzUsO2anBQ03b9JtVnREwicibA/640?wx_fmt=png)  

首先 query user 查看会话 ID（这里的图是我写文章的时候截的，所以登录时间是 6-1）

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEicdOgGh5wfkJVZNjksLXxHm4woiaOwOY1PsEMxVc7QMDGk8WdRMWOylA/640?wx_fmt=png)

然后再在 system 权限的命令行中执行 tscon 2，发现失败，因为上次登录的时间已经超过三天了，凭证过期，无法劫持会话

****6.PTH**** ****攻击实现********利用** **hash** **登录****
----------------------------------------------------

最后通过 pth 攻击 hash 传递攻击拿下了 administrator 的桌面权限，具体如下

mimikatz 命令：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEbegiamOpojsFyWLcQShiayk1RUibS5SHLCSGDqHm4EkC5guFzsaJEw0uA/640?wx_fmt=png)执行后弹出远程登录界面，选择连接，成功实现无密码登录 administrator  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTEpP01ibFNkFbAbXQxAoSx0QbZmRhIW3pgQ0tYCVWhkCic1QIt5NBGfBMQ/640?wx_fmt=png)

桌面长这样，mssql 数据库管理页面还没退出

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfnJw4RrIa46zJhsyzycMTErmK7vZfRsUcYE30ep4nOYATsfLauPLibBPLUxicEgoMJGLSjRJTejicrw/640?wx_fmt=png)

**结尾**
======

梳理一下过程：1. 从外网信息搜集—2. 到发现 sql 注入—3. 到绕过权限上马—4. 再到低权限提权—5. 最后通过 pth 实现无密码登录 administrator 桌面，整个过程没有什么技术含量，都是很基本的操作，但是能学到很多，求各位大师傅轻喷，我觉得从发现问题到解决问题是一个很享受的过程，还有，最后拿到了程序的源码，审计后又发现了一处注入和未授权进后台，因为篇幅问题就不说了，漏洞已经打包提交至平台，最后，网安学习这条路任重道远，希望自己能走下去，少一点花里胡哨，踏踏实实学东西才是最重要的，不能觉得自己学了点皮毛就四处炫耀，保持适当的谦卑。

****本********文推荐实操：SQL 注入原理与实践 （复制链接做实验）  
****https://www.hetianlab.co/expc.do?ec=ECIDee9320adea6e062017112114390500001&pk_campaign=weixin-wemedia#stu  

本实验介绍了 SQL 注入原理，解释了简单判断一个参数是否存在注入的原理，能够利用简单的 SQL 注入获取其他敏感数据。

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC6iavic0tIJIoZCwKvUYnFFiaibgSm6mrFp1ZjAg4ITRicicuLN88YodIuqtF4DcUs9sruBa0bFLtX59lQQ/640?wx_fmt=gif)

戳 “阅读原文” 体验免费靶场!