\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/iHY7gl82TxxNpGPh\_Fn09A)

本地搭建实验环境时遇到了不少小问题

实验环境 2008 R2

宝塔搭建的 IIS discuz3.2X

手动上传 shell

冰蝎连接

（ps: 有表哥使用冰蝎的时候提示文件存在但是无法获取密钥，解决办法，使用最新版本的冰蝎即可，具体详情看更新日志）

下载地址：https://github.com/rebeyond/Behinder/releases/

连接上 shell 发现无法执行命令???

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwc1PicUtxlzy4Gwo0mnhN7kcnIBtLibTrKaz10hU2CMH2U3PPxBvr1d9w/640?wx_fmt=png)

查看 phpinfo 原来是禁用了函数…… 几乎能用的都禁用了

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwNqaVYIoSshAPNU4BdiaR7ASL19MVGvJG42KahZ2zADqFy8K9QWF2vow/640?wx_fmt=png)

想想很奇怪，刚搭建的网站那就是是默认值，为啥平时日站不是这样的，东查西查之后真的要好好感谢一下宝塔，太 sweetheart 了，部分默认禁用值截图如下。

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwS3z7JaklDiaQyiaRoc1I91J1uqO9qD0lEfSQ5EVibeTwxGRkY8uJ6sD2g/640?wx_fmt=png)

既然禁用了函数，那么我们本着没有解决不掉问题的想法，百度！

雷神众测公众号的一篇文章总结的超棒！打 call !!

第一趴, 常规绕过，看了看 phpinfo，就知道现在情况不常规。

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwIbfatDribO5FjefYdD5lF3NJOLjZH8gXFMkWKKgAw7mWOTalEaALxBw/640?wx_fmt=png)

第二趴，对不起，putenv 不可用

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwP8eBCzC2g4U9rhYDicoIEZ9icH56v4ZmY9tr80GrkT36OAYKRHHVxLbA/640?wx_fmt=png)

第三趴，不支持

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwjibP83YsqEw9wPRVnu69hLpMeYxMWfiaNtibNXibtaaF9X3QM2xwZDMg3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwQOantlVR30NhQ9Wpf8jPJtOTnLHQ9lxrxvGSzxOwG5NXPdUkxEvIzw/640?wx_fmt=png)

6-12 都看了一遍，同理如上。

不得不说，总结的太好了，但是 tm 都不能用啊，宝塔牛逼啊，一剑封喉啊

饶头……

问了问大佬们，又 get 一个解决方案，又可以继续百度啦！！！

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwKFnibN9nG305bYMxPGrQv2pR2WYibatykxEN1t1Qswq6ic8bZq3IyPAicg/640?wx_fmt=png)

Emmm… 不对，我不会溢出啊。

继续百度查看一下 Github 的 bypass 全家桶？？？康康康康

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwTJsVh8vchAE3XQrlmGGlCibhcuVlh4hxRPceaKuwossoO6NMYuOkK3Q/640?wx_fmt=png)

直接飘红

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwdN4WrMmN909Vrz8k8GLlxmE3g5T146mXib6jrqr1ShibYEvh3Ovricb9A/640?wx_fmt=png)

又试了几个都是如此（毕竟禁用了函数）

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwWQlx1gIDXgqhemmCJ4aM7pbpCAQkkhpicZC4M7sGmsJ7r2PV1uG07bg/640?wx_fmt=png)

这个时候一篇文章吸引了我（没办法了，只能看你了）

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwqN5ojVcKcicxTjzpys0W0MIXzDvw9e05oFHwhELkQ0RLogdIyH7aSWQ/640?wx_fmt=png)

作者说

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwvfh9IdL9CxiboJwwnQdkbICADwQb88hcicZT3taFibz9SL9kAialOia1a4g/640?wx_fmt=png)

那我们就按照他的方法来做

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpw7Pg2sYRmkV8V6J2sAbPpe5XSolfvGxcqHfvczXQxEogjeGribqNqWTw/640?wx_fmt=png)

可是留下的代码好像不太行

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpw807BhicPvib1TomEkP4zMVZ4u7DtWcIN4RcP69f9icepMUJl9qcXsIfJQ/640?wx_fmt=png)  

最后找了暗月的提权工具

可以正常使用了，选择对应的版本，导出 udf.dll 文件

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwfpsjSv94BsZ6xtJHHNddAp2NHg8gTibvTB7iaibibn3ojqowTBZZFb7icsw/640?wx_fmt=png)

Ps：

MYSQL <5.1 版本导出路径：

C:udf.dll    2000  
C:udf.dll 2003（有的系统被转义，需要改为 C:sudf.dll）

导出 DLL 文件，导出时请勿必注意导出路径（一般情况下对任何目录可写，无需考虑权限问题）

MYSQL>= 5.1，必须要把 udf.dll 文件放到 MYSQL 安装目录下的 lib\\plugin 文件夹下才能创建自定义函数

该目录默认是不存在的，这就需要我们使用 webshell 找到 MYSQL 的安装目录，并在安装目录下创建 lib\\plugin 文件夹，然后将 udf.dll 文件导出到该目录即可。

之后我们可以成功执行命令

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwnIIOcFrMic5256RCJJZ8AeGaJFQiaWS6gowVWAZianSCOPLLMibzic2sO9g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwZicZXE2rzUhL0ec2U1LUWJ2P0Cwc43JmPnxwAESibsn2ic3zBILJhBwcA/640?wx_fmt=png)

Ps：亲测添加管理员数据库会 down，表哥们实际环境注意安全。  

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwDiaiaoiaCXZX8ZIkGQnRkPDw2qYSTzibMuHF8MgXcc3Ux7YIxWCzXibhH3Q/640?wx_fmt=png)

看了看也是只能简单运行 sys\_eval

这就很挠头呀，难道我要上服务器把宝塔的设置关掉嘛（当作无事发生）

想了想用 CS 反弹出来再康康吧

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwEApiag1QzASja6u9FE4wSibUFXI5JBPK1r1aE8T0ZmWke9XAXV9OBvXg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwibDEkHSOiadvE2t45bnbtZVCrGpZl1iahV1icKq5hHJAn5uicUOnbKKbicwg/640?wx_fmt=png)

服务器 powershell 普通管理员权限执行

意外发现可以无限制执行命令（其实捣鼓了好一会 ==，开始用的 3.13/3.14 都不可以执行，最后尝试了 4.1 版本发现可以执行）

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwOtIuyzD4YHLsCVCTrpEgxiaibrFdGKkTrLoy0NSDpfxw4c2CIctQqo1Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwUcmWq4oMqaJSw7bZeYxOaQ8aRwZaHFibwn31Yic1ib1snyDhHYVZWpGYA/640?wx_fmt=png)

Mimikatz 查看密码

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwWOPJ4EEPjNm0nAjz8C7OEMyyJ9QuialuqicmQBfG4t2NYvwrdEMyyzEA/640?wx_fmt=png)  

？？？？看下本地情况

![](https://mmbiz.qpic.cn/mmbiz_png/ax4vPrQ4hXfVs8TgK6vzbEYXUicctJkpwSQ8cfiaQTReibVeNTSzkkKiatnsDzy1rE6iaXnpL7Rtp1o4cLbHY1sr5yA/640?wx_fmt=png)

本地服务器加域之后没有域管理员密码无法直接创建用户的额 ==

意外的无视了域控的策略 == 表哥们可以本地搭环境验证一下。  

虽然本次实验有很多 bug 的地方，不过其中的思路觉得值的记录一下，忽略一些细节观看体验更佳。（虽然到这里本来想做的实验一步都没做 ==，完全为了突破环境限制），因为要继续做实验，记录的比较凌乱，表哥们各取所需，有遇到类似情况的也可以私聊讨论。（ps: 整个过程比较懵逼）