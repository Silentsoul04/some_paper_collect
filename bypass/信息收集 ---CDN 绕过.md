> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gfBARgh2C9dlphmQacPP8w)

当老板给了我们一个目标网址时，我们第一时间应该做的就是对目标网址的信息收集，信息收集中又包含了许多复杂而又繁琐的步骤。接下来由我简单的介绍一下信息收集中的 CDN 绕过吧！有写的不好的或者不对地方还请大佬们随时指出呀！

**什么是 CDN？**

CDN 即内容分发网络，主要解决因传输距离和不同运营商节点造成的网络速度性能低下的问题。

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3Vs8gDB0B9IeO9PMbgrUQJlgMb4tv5icFIbyB670WwosXQTBFhurTLicQ/640?wx_fmt=png)

#**目前常见**的 CDN 绕过技术有哪些?

1. 子域名查询

原理：站长没有对子域名（也就是主站的同一网段下的子网站）加 CDN 技术

2. 邮件服务查询

原理：①一个正常的公司是没有必要对邮件服务器加 CDN 技术的，经济问题

②涉及到一个主动联系的问题，发邮件给我就会暴露他们的真实 IP

3. 国外地址请求

原理：如果一个网站主要是针对国内人员，那在国外访问国内就没有必要加 CDN 技术

说白了也还是经济成本的问题，加 CDN 是要钱的！！！

4. 遗留文件，扫描全网

原理：phpinfo.php 里面有特定的参数可以看真实 IP

5. 黑暗引擎搜索特定文件

shodan、zoomeye、fofa

特定文件：文件唯一的 HASH 值、MD5 值，搜索网站的 ico 文件，也就是图标文件

6.dns 历史记录 or DDOS 攻击 (违法🈲用)

DNS 历史记录：查网站以前可能没有使用过 CDN 时，解析的 IP 记录，也许是真实 IP

这里需要说一下，其实加 CDN 节点也是为了防御 DDOS 攻击的，攻与防之间，看谁损耗的成本更低，谁就是胜利者！**防御的目的也是为了提高攻击的成本**

一、先判断目标网址有无 CDN 节点
==================

**验证 CDN 的存在：****https://www.wepcc.com/**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI31dLfGQqIryrurJ4Vf61vJFdZCWjNa30PaQibLicoqiaBibvLv1mrIc9pyw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3l7ictxZvPkXibWDYn1TbXdhGdfFQmsWgPa4Q5yH1PKgTs4oAV0DYJZ7w/640?wx_fmt=png)

**如果目标网址解析出来的 IP 像图中一样，并不是那么统一，那目标网址就是加了 CDN 节点**

**以下这个是没有 CDN 节点的，因此解析出来的 IP 就是此目标网址的真实 IP**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3naC9v27OCscPickblH5BXVNDuKAEm6HPibA3piawcibFlwx6OkkicpbfKQw/640?wx_fmt=png)

**现在考虑的是有 CDN 节点，我们该如何绕过 CDN, 寻找真实的 IP 地址呢？**

二、绕过 CDN 节点, 寻找真实 IP
====================

★利用子域名请求获取真实 IP
---------------

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI37UDmF3IvrwQI4BoPic6gXzOdtBG3DRtXS49mCzQ0yUq0OhTCTlICCyg/640?wx_fmt=png)

去掉 WWW 后，我们再次寻找：

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3cdjPIMZQDNPobGQXOgFgtF9s5xHObQwFBm2ibOVj6aILj7piccUN8yYA/640?wx_fmt=png)

只有最后一位不同，利用 FOFA，对 71、72 进行 IP 反查域名：

124.*.*.71，登进去后，没有数字签名的认证，打开网页需要我下载数字证书，但至少 HTTP 的状态码是 200 的

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3pzyMBpa4FxbpW39RKicLf8ymeYg0PjWOukZokkKJvxzvM6tRyB52W3w/640?wx_fmt=png)

124.*.*.72，可以完全打开网页，没有出现意外情况，状态码显示 301，说明网页做了一个重定向的跳转。

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3vyrH4m5ib5ja4meBqMR3gpxPVluvG5Aj3dINCxsmO74I4vXNCZNFXYA/640?wx_fmt=png)

那为什么会出现这种情况呢？

这也许是因为站长在做 CDN 节点的时候选了 www.****.com 作为分流网站，其实只要把 www 改为”*”，就可以完美解决这个问题了。

★利用 DNS 解析的历史记录
---------------

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3j2CkhsEAXmMWYhgMdczh5qfLUveOw2ubaWwTVtjWrs7AWhKqxEbZPQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3aQiaJ5Wm9FerUWh2UeiaiaPEyFqeSrsBsrmzRtltTr4K4FAoJvItwY06A/640?wx_fmt=png)

此结果仅供参考

历史解析出最早的记录是在 2012 年，有可能在 2012 年的时候，网站并没有使用 CDN

那解析出来的 IP 就是真实 IP, 这仅仅是其中一种情况!!!

第二种，即使在 2012 年网站没有使用 CDN，但是在这几年过程中，网站的服务器也许会被更换到另一个地方！就是 2012 年的 IP 是真实 IP, 那也不是如今的真实 IP 地址了！！！

★利用国外地址请求获取真实 IP
----------------

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3bQ0epk5lKGx8gOSp6UgCvpold366aEcAppM13JT0f9Xib59Cc6e4LCA/640?wx_fmt=png)

https://asm.ca.com/en/ping.php

原本这个网站是国外在线代理网站，有很多全球不同地方的 ping 服务，有一定的机率可以帮助我们找出子域名的真实 IP。

但是不知道为什么这个网站的 tools 服务关闭了，一进去需要登录，而且无法注册账号，网上找了好久也没有找到解决的办法！

其实还有一种方法跟这个类似，只不过要一个个的来切换代理 IP，尽量选择一些不知名国家的代理，更容易获取到真实 IP

★利用邮件服务器接口获取真实 IP
-----------------

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3CCDcAhAv2iaNBZX1dk7BTfKEAC5icicClSIZ7KeGHRkG7sc2tDGnWamIw/640?wx_fmt=png)

去掉 www，发现还是有 CDN

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3MEzFTf39RkWDSNJMJm3G6yIw7RXFuXUEhx2havXFskKkYECia1u5m4g/640?wx_fmt=png)

打开邮件查看源码，这里可以看到 from www.***.com 后面附带 IP 地址

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3yQD6hFdBylNxO5rkskic5rbiaBRTM6DgId35aicFmr8fbWJUMAOdfuYew/640?wx_fmt=png)

验证此 IP 是否为真实 IP，修改 C:\Windows\System32\drivers\etc 下的 hosts 的文件

先修改与上面不同的 IP：

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3O6MVpLN3kU7FQw0RT8OLhWlWWc3HokRUmmBDw00ib0xaU1mOm2W4iceg/640?wx_fmt=png)

访问此网站：

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI32fnB2smZkRxIUz1Y4EmUt3yCvap8cMpibgFoJaIspxsczEpDJzeTw3A/640?wx_fmt=png)

检测 219.153.49.169 是否为真实 IP

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3TcBgBpDZicapcHKKictQUq9HdE9fwdaWgNwtqPyAlj8BMiaULa0U0M9ww/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3xZ99M5dLnEqSEmuiaZlRHjp7iaMlicEuWVDQNDPyorzVjwblzCaglle2Q/640?wx_fmt=png)

访问成功

到这里大家可能会有一个疑问，如果我将 IP 修改成 CDN 节点的 IP，是否会访问成功呢？

答案是可以访问成功的，既然可以访问成功，那我做这个实验岂不是毫无意义？

其实并不是，因为我们验证了 219.153.49.169 这个 IP 是可以访问网页，并且这个 IP 是邮件服务器发送过来的！具有很强的可靠性！！

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3euZCxaKAQTiane8cbkRsXxZ2wmQa5gKsPgtlSgic9x4SKQafSMhEqtcw/640?wx_fmt=png)

看备案号：

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3SFia8CbKOFwQQ2t3uptST19m366xRUHOJOJiaKLloOxy4FUwzibUsB0jQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI370Han989I2RYe8NMniagI3qiarhKRDr59ibSNyQUT9FWKTFN62IP1RWQQ/640?wx_fmt=png)

按常理来说，这么近的地方，是没有必要放 CDN 的，经济成本

★利用黑暗引擎搜索特定文件获取真实 IP
--------------------

原理：黑暗搜索引擎有过滤缓存的机制，确保搜出来的结构大部分都是真是存在的

而 ico 图标文件的 hash 值是唯一的，用 shodan 去检索自己所爬取到的 ico 文件的 hash 值，如果我们提供的 hash 值与 shodan 缓存的一致，IP 地址便会被搜索到！！！

鄙人不才，找到一个 bocai 网站，它就刚刚好有 ico 文件：

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3IGIP78PVaOs2heCT5wLVmMOB3sFLpJyoQ8VluBfag0uFhevCqo425w/640?wx_fmt=png)

将上图红框内的网址放在以下这段代码中，注：在 python2.7 环境下执行这段代码

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3aMzhAFykxuo7qzseJzAMvBb3Y3picmHpZ9R0xz5FtxT7wPQ9POsz2mA/640?wx_fmt=png)运行后得 favicon.ico 的 hash 值，符合 shodan 语法，去 shodan 搜索：http.favicon.hash:613098635

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3Zw4GxkCPdOowK8UhianiaGyfgyQOnS9coOiaTCTV33p6tDlN2De0mhsgQ/640?wx_fmt=png)

直接吐了，解析出来 10 个 IP 地址，日本占了七成！！！都是分布在其它地方的 CDN 节点，发现还是不行啊！能力有限，还是要继续学习 ----

俗话说环境搭半年，实验半小时！！！如今真是感同身受，这一步的环境弄了我前后几个小时！

到最后却还是抓不了 baocai 网站的真实 IP，而且后面还要看能不能搞到主站，一步更比一步艰辛！师傅说了，菜是原罪，菜就菜，没有什么可以解释的—_—！

★Censys 查询 SSL 证书找到真实 IP
------------------------

原理：就是搜集 SSL 证书 Hash，然后遍历 IP 去查询证书 Hash，如果匹配到相同的，证明这个 IP 就是那个**域名同根证书**的服务器真实 IP。简单来说，就是遍历 0.0.0.0/0:443，通过 IP 连接 Https 时，会显示证书。

我命由我不由天，我决定再用另一种方法搞搞这个 bocai 网站！

输入域名，在 Censys 上查找与域名相关的证书，找到四个

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI38rL9gQzYJuPh5YhCd2JMYcLUiaa7uRtktJLbrCvEWWBeIcOsI2HfB4Q/640?wx_fmt=png)

点进去，在页面右侧 Explore(), 并点击 IPv4，运气好可以找到真实 IP

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3XtTcTibPwJpY04jFCtXIghiaFjYtwAicRafibxgZqyzgwNcZ2ddlg3lyMg/640?wx_fmt=png)

我已经等不及啦，我们一起来看看结果吧：

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI31MRAP8B5jMoOjhAhVQID7thDBL9zia44ldLRQicy2oK01m9Q3cngbhNA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3rMOGHQECXLIEiaa3INwZibOiafAjjNsl2x21a9cWPLUMwFkeJdXRtibbtg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3nSFlommAjn9zU5z0L1nuAQRibfGCWnqDOqXsicoibL5Zn4asTZQgXgeibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3n96n3pGlFV5X6Z1jxPhqU8fxJM32yL8aY35x1ibaGG1pPuG80YoLwWg/640?wx_fmt=png)

好家伙，四个证书没有一个可以找得到 IP 的，我彻底服了

**总结****：没有哪一个方法是万能的，每个方法都要试一遍，直至找到我们所需要的信息，以上只是简略的列出了我平时所用的几个方法，还有一些工具比如 fuckcdn(易语言)、w8fuckcdn(python)、zmap 也是可以绕过 CDN 寻找到真实 IP**

**小彩蛋：**

**今天在做 TCP/IP 其中的 HTTP 作业时，发现了我们学校的邮件网站有个特别意思的现像：**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3CYzomEhGwC5JkBOpVc46ibibIG0cw0bkViavbMiayooCEgMN1Ig5MMyg0Q/640?wx_fmt=png)

**登进去可以看到背后的邮件，但是需要我们强制修改密码后，才可以看**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3qicz6kgJPDJxTxlicXMeCTatdEXy24nCy0qm5ibJrkHYn5AqyJMLbfwyg/640?wx_fmt=png)

**这时我打开 F12，选取②框**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3mA0VMyjAJ63YBcYva2ibP6QPDBZJjSZibPFuoIiaiavSc0Vqq6LbUWYa3g/640?wx_fmt=png)

**然后再到③右键，编辑 HTML，**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3giapWmic7OJicicE3sRA95wibBibmRdz56iaUXeKT6moiaRJZyJDia32EOOo1Sg/640?wx_fmt=png)

**Ctrl+A 全选，然后全部删除**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3U4TiaR6smzHljwkBo72BwlEdcElrURJ8j2Sq1bOoMeL99OtjgOgIvRA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI31H1ibobOELUC3dWrj0PgFj638PODd1RlOm2lrj8NaNhDcT15PZJsrvQ/640?wx_fmt=png)

**鼠标在空白框外点击，然后看效果：**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3MVUcaw0YB1GRbyr1LGCQPyOH2RMqrONCkEIcIm1AfrUUunK9oyYNlg/640?wx_fmt=png)

**虽然强制修改密码没有了，但是因为这个灰色的界面还是无法点击，按上面的操作再来一遍，删去灰色的界面就可以了**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3gS0mfUk8sQ02LEjJIdCEQFlKoDYaRYibZ6Unu2Y2U1aXGeSric8DgUNA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3iciaUhJ9UbTCBAlXlvKaJTnOkWicjiapQar00DtPwwBXpVOWf82Q908j7A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3S2tfIbgpOFfZvbPkCWqoacdFCgr2Gclve8icwuoMMcDUmNicdwp4IxmQ/640?wx_fmt=png)

**搞定啦！！PS：自由是人的天性，建议还是修改密码的好！哈哈哈**

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFAaHOnU6LzBzD8pibdqic6aI3AkMiaXdbQQzHtssiaVib5pTXU8EicowMTNhVCfluUY59eSYEsaAD8tcLUQ/640?wx_fmt=png)