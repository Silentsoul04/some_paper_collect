> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/M7ILKR9xRVqrcox5V60ehw)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)  

> **文章来****源：****程序员阿甘**

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28gB5GH6buBtTQxqHibKu4Ytu3Oia0FrQoqFx3DO3oztSwJPQMdhQ1qiahGRz3InfCn5PvVnSYTeAl1hg/640?wx_fmt=jpeg)  

来自量子位

明明下载的是一张图片，只需修改后缀名，图片就变成了一首歌，一串 Python 代码？

国外黑客 David Buchanan 利用 Twitter 的漏洞，可以用图片伪装的方式传输一份 “加密” 文件，前提是**不超过 3MB**。

他成功把这种藏匿文件的 GitHub 源代码压缩到图片中。

现在你只要去他的 Twitter，把这张图片下载下来，并将文件后缀名从**.png** 修改为**.zip**，即可解压为 Github 代码。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA2DNfGfFUToRfkHfvpnV19IT26u2vEQ56A1xgKDBDfdtobMQicWxtATlps6yTnzH6LWlaq9iaqKvWQ/640?wx_fmt=png)

（注：亲测 Mac 自带解压工具报错，第三方工具可正常解压。）

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA2DNfGfFUToRfkHfvpnV19aeibXBL2Iy71ia6XnsPZUvJ6kyVLLJ2VH0yOIGU39Q2yK6SiaTBNTkQrw/640?wx_fmt=png)

对于有十几年网龄的老网民来说，这并不是一项新技术。早年就有人将文本文件或种子文件藏匿在 jpg 图片中。

这种方法的特点在于，把文件打包到图片中并不影响正常显示，但一般来说文件大小不过几十 KB。

随着网络发展，越来越多的平台允许用户上传大尺寸无损图片，这就给藏匿大文件提供了契机。

Buchanan 的新方法现在将藏匿文件体积增加到 3MB，你甚至能放入一首歌。

Twitter 上就有现成的例子，Buchanan 放出了一张 **surprise.mp3** 的图片。如果后缀名修改为. mp3，就变成了一首歌。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA2DNfGfFUToRfkHfvpnV19icLzC7qqibWzt39yT1Is20cLcpEnABicUFdxCmgMvvQX8GY365wfqsddA/640?wx_fmt=png)

至于这个 surprise，自然毫无意外是 Rick Astley 的《Never Gonna Give You Up》这首歌。恭喜你，又被 “瑞克摇” 了。

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtA2DNfGfFUToRfkHfvpnV19TeIqRxXp8MOkGqwAdbhJaTfoCTVAFde3v5YoOZ3TZWuYAjkQaKIBJg/640?wx_fmt=gif)

Buchanan 的这一发现已经连续多天成为 GitHub 热门项目，最终在周末登上日榜第一。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA2DNfGfFUToRfkHfvpnV19HeTRRicia94mgfFGYWvYoG5YQq0RicBXnPvU3P6icibeRicXWBrwiaXwZ35YQ/640?wx_fmt=png)

使用方法很简单，只需要将 pack.py 文件下载到本地，运行以下代码：

```
python3 pack.py cover.png file.zip output.png
```

其中，cover.png 是封面图片，file.zip 是你要藏匿的文件，output.png 是输出结果的文件名。

从外观上来看，output.png 和 cover.png 是一样的，但多出一个压缩包的大小。

原理
--

用图片隐藏压缩包的原理并不复杂，png 图片文件的格式如下。在 Zlib 之后，有一片 IDAT 块的附加数据。藏匿数据就放在这里。

**![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA2DNfGfFUToRfkHfvpnV19XZHEyia8iaC0IibN0U7lvLOZRaNauMa5xLB2NbLpts4EcTCicu6icoMy8tA/640?wx_fmt=png)△**图片来自 Twitter 用户 @angealbertini

Twitter 通常会压缩图像并删除所有不必要的元数据，但是可以在 “DEFLATE” 的末尾添加更多数据。

如果整个图像文件符合避免重新编码的要求，压缩包内容就不会从 IDAT 块内的 DEFLATE 流中剥离。

这种方法不仅限于嵌入 zip、mp3 等文件，只要数据能压缩到 3MB 以内，都可以嵌入到 png 图片中。

Buchanan 表示，这种方法可能被黑客用于藏匿恶意代码，他本人已将该漏洞利用报告给 “漏洞赏金” 程序，但却被 Twitter 告知这不是 bug。

能传输 “加密” 文件，怎么能说是 bug 呢？应该是隐藏功能才对。（手动狗头）

带压缩包的图片地址：  
https://i.imgur.com/kNhGrN3.png

David Buchanan 的 Twitter：  
https://twitter.com/David3141593/status/1371974874856587268

项目地址：  
https://github.com/DavidBuchanan314/tweetable-polyglot-png

```
版权申明：内容来源网络，版权归原创者所有。除非无法确认，都会标明作者及出处，如有侵权烦请告知，我们会立即删除并致歉。本文仅限于技术讨论与分享，严禁用于非法途径。若读者因此作出任何危害网络安全行为后果自负，与本号及原作者无关。谢谢!
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

推荐文章 ++++

![](https://mmbiz.qpic.cn/mmbiz_jpg/US10Gcd0tQFGib3mCxJr4oMx1yp1ExzTETemWvK6Zkd7tVl23CVBppz63sRECqYNkQsonScb65VaG9yU2YJibxNA/640?wx_fmt=jpeg)

* [一款内网自动化横向工具：InScan 开源扫描器](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650508466&idx=3&sn=f2adba378836f1eaf1eab5f1f8261ff2&chksm=83bae956b4cd60400af4aa53dc690f966b9d4071eb303068ac056f317e285f360e60757b94b4&scene=21#wechat_redirect)  

*[DAMM - 开源内存分析工具](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650484019&idx=4&sn=a333ae5988c77bef62ca5bce1396b16a&chksm=83ba48d7b4cdc1c1e20bfdeec343f186a1a841c53628c48bba080ca8bb7a29997c18c8dc66c7&scene=21#wechat_redirect)

*[Superl-url 一款开源关键词 URL 采集工具](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650456529&idx=4&sn=3884ffef47ccff9fc7962b9f49658dd0&chksm=83bba435b4cc2d233f72932ba4187d74e75032b3947b829b21adeb7c31ce8ffcf58941186764&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)