\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/Pu5Xb3PzZgGh8N\_DSSuBzg)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **七** 篇文章，本公众号会记录一些我学习红队攻击的复现笔记（由浅到深），笔记复现来源于**《渗透攻击红队百科全书》**出自于 **亮神** ，每周一更

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

第一章：信息搜集

目标资产信息搜集的程度，决定渗透过程的复杂程度。

目标主机信息搜集的深度，决定后渗透权限持续把控。

渗透的本质是信息搜集，而信息搜集整理为后续的情报跟进提供了强大的保证。

  ----Micropoor

**常用漏洞情报网站**

  

---

**Exploit-DB**

* * *

ExploitDB 是一个面向全世界黑客的漏洞提交平台，该平台会公布最新漏洞的相关情况，这些可以帮助企业改善公司的安全状况，同时也以帮助安全研究者和渗透测试工程师更好的进行安全测试工作。Exploit-DB 提供一整套庞大的归档体系，其中涵盖了各类公开的攻击事件、漏洞报告、安全文章以及技术教程等资源。

官方网站：

```
Usage: searchsploit \[options\] term1 \[term2\] ... \[termN\]
 
==========
 Examples
==========
  searchsploit afd windows local
  searchsploit -t oracle windows
  searchsploit -p 39446
  searchsploit linux kernel 3.2 --exclude="(PoC)|/dos/"
 
  For more examples, see the manual: https://www.exploit-db.com/searchsploit/
 
=========
 Options
=========
   -c, --case     \[Term\]      区分大小写(默认不区分大小写)
   -e, --exact    \[Term\]      对exploit标题进行EXACT匹配 (默认为 AND) \[Implies "-t"\].
   -h, --help                 显示帮助
   -j, --json     \[Term\]      以JSON格式显示结果
   -m, --mirror   \[EDB-ID\]    把一个exp拷贝到当前工作目录,参数后加目标id
   -o, --overflow \[Term\]      Exploit标题被允许溢出其列
   -p, --path     \[EDB-ID\]    显示漏洞利用的完整路径（如果可能，还将路径复制到剪贴板），后面跟漏洞ID号
   -t, --title    \[Term\]      仅仅搜索漏洞标题（默认是标题和文件的路径）
   -u, --update               检查并安装任何exploitdb软件包更新（deb或git）
   -w, --www      \[Term\]      显示Exploit-DB.com的URL而不是本地路径（在线搜索）
   -x, --examine  \[EDB-ID\]    使用$ PAGER检查（副本）Exp
       --colour               搜索结果不高亮显示关键词
       --id                   显示EDB-ID
       --nmap     \[file.xml\]  使用服务版本检查Nmap XML输出中的所有结果（例如：nmap -sV -oX file.xml）
                                使用“-v”（详细）来尝试更多的组合
       --exclude="term"       从结果中删除值。通过使用“|”分隔多个值
                              例如--exclude=“term1 | term2 | term3”。
 
=======
 Notes
=======
 \* 你可以使用任意数量的搜索词。
 \* Search terms are not case-sensitive (by default), and ordering is irrelevant.
   \* 搜索术语不区分大小写(默认情况下)，而排序则无关紧要。
   \* 如果你想用精确的匹配来过滤结果，请使用用 -e 参数
 \* 使用' - t '将文件的路径排除，以过滤搜索结果
   \* 删除误报(特别是在搜索使用数字时 - i.e. 版本).
 \* 当更新或显示帮助时，搜索项将被忽略。
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH4libD5DDWiaA7WfMGteIILsds1CTnXCBq3ichAtULicgQicicnialrz7QdY64g/640?wx_fmt=png)  
命令参数介绍：  

```
searchsploit
```

  

* * *

  
在 Kali Linux 下自带了 Exploit-DB 搜索，我们可以输入命令：  

```
searchsploit
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH4ka78BlJBsHD8kH7sKWpSc5VObdwXUYfXUX7zRIaCs0nBEMgj8hzhgA/640?wx_fmt=png)

示例一：搜索有关于 Windows 提权漏洞

搜索命令：**searchsploit -t windows local**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH4oIamdIJic1CDAwz0CQ1azebtN4iacDRlTUOvDb1zpD2fKJBUKdStIlsw/640?wx_fmt=png)

示例二：搜索有关于 Linux 提权漏洞

搜索命令：**searchsploit -t linux local  
**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH42Y42RLmaGHibsyjSGcN0iaib6NanmRWQgrf3eDLUuqZFxzeSSu2RqV55Q/640?wx_fmt=png)

示例三：搜索有关于 Apache 相关漏洞

搜索命令：**searchsploit -t apache**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH4jJJAdPBAjejVNqMkjdgkgNrBPRwOhN6jVdpwIe6DXCibefKpjFceYiaw/640?wx_fmt=png)

示例四：：默认 Kali 下的 Exploit-DB 是没有更新的，如果想要更新那么就使用这条命令（更新速度取决于你的网速和你的源）  
更新命令：**searchsploit -u**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH4cUUZHYJwQRvf04ZOe74I7RRex5BTWHTTweicI7mKxiaib2Sl5BHJm2E7g/640?wx_fmt=png)

**在线接口**

一：The Web of WebScan（一个同 IP、旁站、C 段在线查询网址）

域名：**https://www.webscan.cc/**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH4W8btibfzribl8hdFyzZ6s2uicEB3j8tjc2uV2Dr9vdmAms5v4uB2a4JcQ/640?wx_fmt=png)

二：在线子域名查询 - 接口光速版

域名：**http://sbd.ximcx.cn/**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH4r25jPyhTxLypUFH8fdluB3LCgJyZ9yNibxfgviaPtw6SHYUT5ROWBNTQ/640?wx_fmt=png)

三：在线 cms 指纹识别

域名：**http://whatweb.bugscaner.com/look/**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKVhucMAdyFtAiaN8E5A5qH45SWUuiaegvhicYlv57V7JZyYBwNO2icZeBRkbibLLVuEdTq6IICPgHtzrg/640?wx_fmt=png)

还有以下网址：

域名：**https://url.fht.im/**

域名：**http://viewdns.info/  
**域名：**http://www.t1shopper.com/tools/port-scan/**

...... 等等  

* * *

参考文章：

https://www.freebuf.com/sectool/139685.html

https://blog.csdn.net/qq\_20336817/article/details/42320189

https://www.jianshu.com/p/ca234f8cd661

渗透攻击红队 发起了一个读者讨论 你学废了吗？

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

渗透攻击红队

一个专注于渗透红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDdjBqfzUWVgkVA7dFfxUAATDhZQicc1ibtgzSVq7sln6r9kEtTTicvZmcw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDY9HXLCT5WoDFzKP1Dw8FZyt3ecOVF0zSDogBTzgN2wicJlRDygN7bfQ/640?wx_fmt=png)

点分享

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDRwPQ2H3KRtgzicHGD2bGf1Dtqr86B5mspl4gARTicQUaVr6N0rY1GgKQ/640?wx_fmt=png)

点点赞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDgRo5uRP3s5pLrlJym85cYvUZRJDlqbTXHYVGXEZqD67ia9jNmwbNgxg/640?wx_fmt=png)

点在看