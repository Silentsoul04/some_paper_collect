\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/A-VndozeSzXvFdgPmGFDCw)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaa62yRh8ZMicGSoozvuoh0ibFQGj4hjhLCxqwV8T0z3NBPjjnvfZcyObNAEnOib0bH4lRT5dj1Rawd7g/640?wx_fmt=png)

点击蓝字关注我们吧！

抓 包

  

1、修改网络
------

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxMlgLa90fExoVibLLL7KXOiaMib4Y7grncwricMHYMrp7hgc8M48HDnMshA/640?wx_fmt=png)

与本机 IP 一致，打开 BurpSutie

选择：Proxy-Optinos-Proxy Lisiteners

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxxw0bwvul5OBQ83NG6DWeTVw33Xdzo5MGiaHKTvevguXXnUKLibZP0nxg/640?wx_fmt=png)

**2、导入证书**
----------

打开手机浏览器输入：https://burp

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxENyttvNOdbSrwRDqhicBcAgxx42MGYibDAXtiathAs4vq498UPjey1PQQ/640?wx_fmt=png)

点击右上角的：CA Certificate  下载完后，把证书的格式改成 Cer

导入证书

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxVLib444ewfAADHbMuIycWibdDygicYS6clic4MHDibjVS5Xwib9sAKicoOs5g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxPgm8s8p9vocez4OTBPMtmk5MooLwEAhqZaEBickSkQKTU0O2aRLsHDg/640?wx_fmt=png)

导入就完事了。

**3、抓包测试**
----------

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxrCTSBf0X1jAZ1b7jMhL4NMgx7AGWKSNoM1W6DRrSPsYpIscbvJkr2A/640?wx_fmt=png)

好的，没有问题嘿嘿嘿～

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxFiaY5KJJF65ucrhOGNfBvpxFavomjY0bUHfxObG413nWRY1YJrms0zw/640?wx_fmt=jpeg)

常规 APP 安全测试方式

  

##### 工具以及环境：

**SDK**：Java JDK

**工具**：7-zip、dex2jar、jd-gui、apktool、IDA Pro、IDEA/Eclipse、010 editor、SQLite、ApkIDEA、BurpSutie/Fiddler.....

  

1、客户端程序简单的安全测试
--------------

###### **1.1 数字签名检测**

C:\\ProgramFiles\\Java\\jdk1.8.0\_144\\bin>jarsigner.exe-verify APK   文件地址

结果为 “jar“表示签名没问题

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxeuCdUJe4SDjHWa8marmu7H9j6Q8gkX57dnHJtXYhHCftib9nY9icVcBw/640?wx_fmt=png)

###### **1.2 反编译检测**

选择 APK - 右键 - 打开压缩包

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7ux3b9WtpmvNQgib6URIS1W62Nw1icU9OWSIYBqmGnia8VAmDV28jz1rDFQw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxZIrI4Of4bQA2Blzsyx0VGy54IasfF4BOGxn4oPFZciabH7cibXAV3fgg/640?wx_fmt=png)

下面就不多说了，大家懂得都懂，直接上 dex2jar 都是老套路了嗷。

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ082noUlrjkQtMyVHBfyY7ux2da2gSpQOsA22ia6qyic5O9FrzXZibicqkcHKfNVla1hXH2VUhIsZ4l5jQ/640?wx_fmt=jpeg)

用 jd-gui 打开 (我用虚拟机操作的，可能有点不太清楚)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxJZLp6EDVuA7EPnxeqKSvm0ziciaAGNW4y8MsiapGRwe6VY34sl8nH5YMQ/640?wx_fmt=png)

还有一种方式比较快捷，

使用 Android Killer 工具，不过这个工具可能有点老，大家可以自行选择

直接把 apk 文件拖进去即可，内置 dex2jar 和 jd-gui 工具

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxlyIFHVNwrSibW4ZAbu6nQWibXyNRRicZTjupvEZ0BMP3y3HDia6ad2MwQQ/640?wx_fmt=jpeg)

###### **1.3 目录结构**

我个人比较习惯先看一下 xml 文件，里面是获取那些权限之类的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxx2sWJkm43iaeaT2zghicVVDlVJRu71WKYms0ApiagLDvdTxjUKG94eQIw/640?wx_fmt=jpeg)

<table width="680"><thead><tr><th>文件夹</th><th>作用</th></tr></thead><tbody><tr><td>asset 文件夹</td><td>资源目录 1：asset 和 res 都是资源目录但有所区别，见下面说明</td></tr><tr><td>lib 文件夹</td><td>so 库存放位置，一般由 NDK 编译得到，常见于使用游戏引擎或 JNI native 调用的工程中</td></tr><tr><td>META-INF 文件夹</td><td>存放工程一些属性文件，例如 Manifest.MF</td></tr><tr><td>res 文件夹</td><td>资源目录 2：asset 和 res 都是资源目录但有所区别，见下面说明</td></tr><tr><td>AndroidManifest.xml</td><td>Android 工程的基础配置属性文件</td></tr><tr><td>classes.dex</td><td>Java 代码编译得到的 DalvikVM 能直接执行的文件，下面有介绍</td></tr><tr><td>resources.arsc</td><td>对 res 目录下的资源的一个索引文件，保存了原工程中 strings.xml 等文件内容</td></tr><tr><td><br></td><td><br></td></tr></tbody></table>

###### **1.4 组件安全测试**

使用 APKTool 或者 dex2jar 解包，打开目录中的 AndroidManifest.xml

声明：android:exported="true" 可导出

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxDsul9TIumiaYbrickpKUn0salfH4l8TSccicg2Pq9Ql8pYsmNFFIgbEbA/640?wx_fmt=jpeg)

声明：android:exported="false" 不可导出

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ082noUlrjkQtMyVHBfyY7uxopIsOEmuKkfn81pbib6TV2RCpLAKRxsScraIMBRPaVOrHFIVSeVap4w/640?wx_fmt=png)

未显示声明：android:exported：  

组件不是：Content Provider：

###### **1.5 app 四大组件介绍**

1、Activity

  
(1)  一个 Activity 通常就是一个单独窗口

(2) Activity 之间通过 lntent 进行通信

(3) Android 应用中每一恶搞 Activity 都必须要在 androidmanifest.xml 配置文件声明，否则系统将不识别也不执行该 Activity

2、service  

(1)service 用于在后台完成用户指定的操作，service 有两种

(2)started(启动)：当应用程序组件 调用 startService() 方法启动服务时，服务器处于 started 状态

(3)bound(绑定)：当应用程序组件 调用 bindService() 方法绑定到服务时，服务处于 bound 状态

3、content provider  

(1)ContentProvider（数据提供者）是在应用程序间共享数据的一种接口机制 ContentProvider 提供. 了更为高级的数据共享方法

4、broadcast receiver

BroadCastReceiver(广播接收者) 作为四大组件，起到了在各组件中或不同进程中传递消息的功能，

在 Android 中的广播主要由三部分组成：广播发送者，广播（需要传递的消息），广播接收者三部分组成。

###### **1.6 应用权限测试**

*   用反编译工具
    
*   打开源码后，检查 AndoridManifest.xml 文化将应用权限和业务功能需要权限做对比，检查申请应用权限是否大于业务需要权限，有即存在安全隐患。
    
    或者 python manitree.py -f AndroidManifest.xml
    

**manitree 项目地址：**https://github.com/antitree/manitree

**对于 APP 安全测试我了解的也不是很多，近期也会在 app 安全上多花点时间去研究它，给大家带来更好更优质的文章。  
**

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09UMHicQKdkBOjyEkfcuwk5ia3Dibpwr50IrKagljyPaooxhib1hwHPywcIk0ib6Bhbick09RJhicwXP2juA/640?wx_fmt=png)