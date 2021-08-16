> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zXmgjHD0Jy0zaZG0usw1sQ)

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LebHs2DGyKAEgZupcIbXWAgnQlIoLerewyAX3c3bLLg0iaTpJeUuGKrSWsicRvLMXwCIbhkUC8GqGibg/640?wx_fmt=gif)

**原创稿件征集**

  

邮箱：edu@antvsion.com

QQ：3200599554

黑客与极客相关，互联网安全领域里

的热点话题

漏洞、技术相关的调查或分析

稿件通过并发布还能收获

200-800 元不等的稿酬  

##### 一、概述

`Jump Lists`是`Windows 7`开始引入的新功能，该功能允许用户查看固定在任务栏中程序最近打开的文件，如图所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0M2BchefScLNUX0ubXdOCqAM8tvkCEfbYOyjy3cnqGamzc0UdlYOzvaw/640?wx_fmt=png)

`Jump Lists`由应用软件或者系统创建，作用是方便用户可以直接跳转到最近打开的文件或文件夹。`Jump List`显示的列表数量是有限的，在`Windows 7/8`操作系统中，用户可以通过更改注册表来修改`Jump List`的条数，但在`Windows 10`中，这个数量被固定了，用户无法自行修改。  

##### 二、Jump List 文件存储和格式

`Jump List`文件是一种`OLE`文件，存储在`C:\Users\%USERNAME%\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations`和`C:\Users\%USERNAME%\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination`路径下。  

`Jump List`有两种类型，一种是存放在`AutomaticDestinations`路径中的`automaticDestinations-ms (autoDest)`文件，另一个是`CustomDestination`中的`customdestination-ms(custDest)`文件。  

`autoDest`（`*.automaticDestinations-ms`）文件是由系统`shell`在用户与操作系统交互时（如启动应用程序、访问文件等）自动创建的，这些文件遵循`Microsoft Compound File Binary（CFB）`复合文件格式结构，并且文件中的每个编号流都遵循 `MS-SHLLINK`（即`LNK`）二进制文件格式。  

`custDest`（`*.customDestinations-ms`）文件是在用户操作`固定`项目时创建的，比如拖到某个文件或应用程序固定到任务栏中，或在列表中固定某个项目。`custDest`文件只是一系列相互附加的`lnk`格式流组成。比`autoDest`文件更为简单。  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0M6V6QOTzypnpuic0gOee20oz7KmKAfyHocwO289XTX70lGE6husE5ReQ/640?wx_fmt=png)

每个列表文件名都是以一串长度为 16 位的字符串命令，后面跟上`.automaticDestinations-ms`或者.`customDestinations-m`为后缀。  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0M3tSWPloRmtnU1zhH8BJyDw6zNARIr3meHMicgYVDrCkxNl2BakmGLSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0M2ZtA4PQKNEENOM5GEpeWWxZHOrjvibAoqiaBD6DcriamYPW1eSYzib8NNw/640?wx_fmt=png)

这 16 位长度的十六进制字符串是`APPID`（应用程序标识符），用于标识特定的程序或文件。根据微软官方说明，`APPID`可以分为应用程序自定义（`Application-Defined`）和操作系统定义（`System-Defined`）两种。应用程序可自定义一个固定的`APPID`。如果没有，操作系统根据应用程序的路径使用`CRC-64`算法计算出`APPID`。  

关于`APPID`的计算可以参考：https://www.hexacorn.com/blog/2013/04/30/jumplists-file-names-and-appid-calculator/  

一些常见应用程序的`Jump List ID`可以在互联网上找到：

https://gist.github.com/atilaromero/2146441

https://community.malforensics.com/t/list-of-jump-list-ids/158/1  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0MKhZWsficP1Hzhib0ekpfehTPcVZPCM9DiaAsOvwY7cQXgXrviaAkynyMKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0MHUSeCjKBI01n1rLGTXRYSDic0ISHYkgD1YcQGL6tiaIbPjPqHbm0DdFw/640?wx_fmt=png)  
在`autoDest`文件中，除了`Destlist`流之外，其他流都是由`LNK`流组成。`Destlist`流遵循特定的结构，分别记录了作为`MRU`和`MFUlist`的文件访问顺序及文件访问次数，并且包含时间戳。在`Windows 7`和`Windows 8`中，`Destlist`流结构是一致的，但`Windows 10`中有所不同。

这里我们以`Windows 7`系统举例，通过 16 进制编辑器查看其结构组成，打开一个`autoDest`文件，其结构如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0MLumkdmiabnb0zgFLIRtBicqgVRaZuZnzsLI9bRJSSWVmhAQKlD14uFvg/640?wx_fmt=png)

`D0 CF 11 E0 A1 B1 1A E1`：`OLECF`文件标识符

`00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00`：类标识符 (GUID)

`3E 00`：文件版本（次版本）

`03 00`：文件版本（主版本）

`FF FF`：字节顺序标识符（`FF FF`为小端模式，`FF FE`为大端模式）

更多关于`OLECF`格式详情可以参考：https://github.com/libyal/libolecf/blob/main/documentation/OLE%20Compound%20File%20format.asciidoc#2-the-file-header

`Destlist`头结构：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0Mww02laKZdWayouoypEPDqiaD117mKPJgjf9MrA99UjrodkDBFHL6VQw/640?wx_fmt=png)

`01 00 00 00`：版本号（在 windows 7/8 中是版本 1，windows 10 1511 中是版本 3）

`11 00 00 00`：当前条目数

`00 00 00 00`："固定" 的条目数量

`33 33 E3 41`：计数器

`11 00 00 00 00 00 00 00`：上一次的条目数

`11 00 00 00 00 00 00 00`：添加 / 删除操作的数量 - 随着条目的添加和删除而增加。

`custDest`（`*.customDestinations-ms`）文件的结构就简单很多：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0MJ1aESyDyibYcCpLRL6QiaW3akIreO1o9ibaR7fTQ0rpaJRbbCbFrJ9bVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0MaiaHU5cy8xp09T6icuicibX8oAwC1euicYkokNqx6p1PVrKPeLjfQicSxO2A/640?wx_fmt=png)

前面`36 Bytes`是文件头，然后是`lnk`文件部分，然后是下一条`lnk`文件部分，然后是文件结尾`0xbabffbab`：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0MXkYhODRiaTWAywepsvpsib3oJk1TWTWs8C8RbsDTo0Lpv9ic3rI8Xk0CA/640?wx_fmt=png)

##### 三、Jump Lists 取证实战

来源：Cynet 应急响应挑战赛

题目描述：`GOT-Research Ltd`的 财务部总监 `Lord Varys` 发现，有关高级员工工资的某些信息被泄露，并传到了该组织的其他员工手中。此财务信息保存在网络共享文件夹中。此网络文件夹的权限已授予给 `Lord Petyr Baelish`和其他 2 名前雇员：`John Snow` 和 `Daenerys Targaryen`。`John` 和 `Daenerys` 现在都是 `GOT` 的外部顾问，不再是财务部门的一员。但他们对财务共享文件夹的权限尚未撤销，`Petyr Baelish`, `John`和 `Daenerys`这三个人平时关系并不好，`Varys`怀疑这是泄密的原因，他认为有人想要陷害`Petyr Baelish`，让别人以为是他泄露了信息。`GOT-Research Ltd` 的`CEO`针对上述事情询问过`John`和`Daenerys`，但他们都称近一年（自从离开财务部后）都没有访问过财务文件夹。  

现在`GOT`委托你作为调查人员查清`John`或`Daenerys`是否访问了财务数据，其中包括已泄露的 `Management-Salaries.xlsx` 文件。  

最终需要提交嫌疑人的名字和在嫌疑人主机上找到的可疑财务文件文件名、以及时间戳。

我们拿到的调查文件是`John`和`Daenerys`电脑上的`Jump Lists`文件：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0MXqPFccpOdHtREAbZViaUviaXcMDpuoAyqLUppib2VsF8cXjQIzhRRwHsA/640?wx_fmt=png)  

我们使用`JumpListExplorer`工具（下载地址：https://ericzimmerman.github.io/#!index.md）对文件进行解析和分析：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0M3TXbJeaGpYAO9FFCmmLh3CEfYreDiapkfoTpgLNwpgpqOiahlk2IaDOA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0M50qabsRxeDpmlIMaX1ob7dkPsLdoZkc3S7Rwh2FVfIXMOrOkBKEbDg/640?wx_fmt=png)

经过检查和分析，最终在`John`的电脑上发现了可疑痕迹，在`2020-02-07 00:03:54`用`WinRAR`打开了一个`Finance-Summary.rar`的文件。  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfEt4T0icjvOqDsDQMSqQk0M98zdor8pgrlicKgXxdiboETrzABxgM2CJsmiar8lPgzPfAJaxDbTUziarw/640?wx_fmt=png)

分析`Jump Lists`类似的工具还有`JLECmd.exe`、`Nirsoft JumpListsView`等。

##### 参考资料：

https://hal.inria.fr/hal-01988839/document

https://github.com/libyal/dtformats/blob/main/documentation/Jump%20lists%20format.asciidoc

https://github.com/EricZimmerman/JumpList/blob/master/JumpList/Resources/AppIDs.txt

https://cyberforensicator.com/wp-content/uploads/2017/01/1-s2.0-S1742287616300202-main.2-14.pdf

https://binaryforay.blogspot.com/2016/02/jump-lists-in-depth-understand-format.html

https://port139.hatenablog.com/entry/2018/04/22/153224

https://www.researchgate.net/publication/303018869_An_Overview_of_the_Jumplist_Configuration_File_in_Windows_7

https://windowsir.blogspot.com/2011/12/jump-list-analysis.html

**本文实操：数字取证之 Autopsy（复制链接体验吧）**  

https://www.hetianlab.com/expc.do?ec=ECID9a4a-6bc7-4926-8ff5-6fa9c74fe756&pk_campaign=weixin-wemedia#stu  

Autopsy Forensic Browser 是数字取证工具 - The Sleuth Kit（TSK）的图形界面，用于对文件系统和卷进行取证。通过本实验学习文件系统取证的思想与方法，掌握 Autopsy 的使用。  

**文末福利**  

![](https://mmbiz.qpic.cn/mmbiz_png/Tug5TABvIGKDonfKugKiaibseq0DNjrcPibZuH1ch6u8NdIn07JicX0USAwMdBicgogcnhCIpap0movLpR6ZQnd7LkQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LfbzQb75ZqoK2T2YO9XTQYD0aDUibvcxdbLRqzCwlkYcn0HppvXpZuenRzjX8ibhzcibJJge9Bw9xc8A/640?wx_fmt=gif)

戳

  

“阅读原文”

  

  

体验免费靶场！