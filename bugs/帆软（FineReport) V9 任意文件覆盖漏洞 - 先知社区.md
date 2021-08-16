> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9578)

> 先知社区，先知安全技术社区

帆软（FineReport) V9 任意文件覆盖漏洞

[Infiltrator](https://xz.aliyun.com/u/36241) / 2021-05-25 17:40:24 / 浏览数 3678 [安全技术](https://xz.aliyun.com/tab/1) [漏洞分析](https://xz.aliyun.com/node/1) 顶 (0) 踩 (0)

* * *

该漏洞是在近期 HVV 中被披露的，由于在初始化`svg`文件时，未对传入的参数做限制，导致可以对已存在的文件覆盖写入数据，从而通过将木马写入 jsp 文件中获取服务器权限。

*   WebReport V9  
    # 漏洞分析  
    `fr-chart-9.0.jar`包中`com.fr.chart.web/ChartSvgInitService`类传递`op`参数的值`svginit`：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210517145921-6834256c-b6dd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210517145921-6834256c-b6dd-1.png)  
    漏洞主要出现在`fr-chart-9.0.jar`包中`com.fr.chart.web/ChartSaveSvgAction`类，通过`cmd`参数传递`design_save_svg`命令，利用`filePath`参数传递需要初始化的`svg`文件，将`filePath`参数传入的字符串中`chartmapsvg`及后边的所有字符串拼接到`WebReport`目录下`“WEB-INF/assets/”`之后，如果生成的字符串中包含. svg 就会创建该文件，然后将`var7`的内容写入创建的文件。如果不包含. svg 就会递归创建该目录，即传入的是 jsp 等非`svg`文件就会创建目录无法写入数据，但如果是存在的 jsp 文件，就可以覆盖文件内容。整个过程直接进行字符串拼接，未过滤`“../”`因此可以利用路径穿越漏洞在任意可写位置创建文件或覆盖 jsp 文件内容。  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210517150001-7ff4119e-b6dd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210517150001-7ff4119e-b6dd-1.png)  
    跟踪`getInputStream`方法可见，通过`__CONTENT__`参数传递文件内容即可：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210517150012-86c33888-b6dd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210517150012-86c33888-b6dd-1.png)  
    # 漏洞利用  
    由于 WebReport V9 在安装之后在 WebReport 目录下存在`update.jsp`和`update1.jsp`，因此可以构造 payload 直接覆盖这两个文件的内容，从而 GetShell。构造如下 Payload 覆盖 update.jsp 文件内容：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210517150022-8c59318a-b6dd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210517150022-8c59318a-b6dd-1.png)  
    访问 update.jsp，成功覆盖内容：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210517150034-93df76c6-b6dd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210517150034-93df76c6-b6dd-1.png)  
    将文件内容替换为冰蝎木马，需要将双引号转义：  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210517150045-99f41396-b6dd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210517150045-99f41396-b6dd-1.png)  
    通过冰蝎成功连接服务器:  
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20210517150053-9ef5a24c-b6dd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210517150053-9ef5a24c-b6dd-1.png)  
    # 修复方法  
    严格过滤 filePath 参数的值，或使用路径和文件后缀白名单，删除默认 update.jsp 和 update1.jsp 页面，升级 FineReport 到最新版。  
    # 批量漏洞检测工具  
    [https://github.com/NHPT/WebReportV9Exp/](https://github.com/NHPT/WebReportV9Exp/)

点击收藏 | 0 关注 | 1

[上一篇：以 Bypass 为中心谭谈 Fl...](https://xz.aliyun.com/t/9584 "以 Bypass 为中心谭谈 Flask-jinja2 SSTI 的利用") [下一篇：520_APK_HOOK](https://xz.aliyun.com/t/9588 "520_APK_HOOK")