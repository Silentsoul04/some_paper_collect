> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wA4Wp9YnFsvFvsa-V79xpw)

  
![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LebHs2DGyKAEgZupcIbXWAgnQlIoLerewyAX3c3bLLg0iaTpJeUuGKrSWsicRvLMXwCIbhkUC8GqGibg/640?wx_fmt=gif)

**原创稿件征集**

  

邮箱：edu@antvsion.com

QQ：3200599554

黑客与极客相关，互联网安全领域里

的热点话题

漏洞、技术相关的调查或分析

稿件通过并发布还能收获

200-800 元不等的稿酬

##### 0x0、概述

`ShellBags`是一组用来记录文件夹（包括挂载网络驱动器文件夹和挂载设备的文件夹）的名称、大小、图标、视图、位置的注册表项，或称为`BagMRU`。每次对文件夹的操作，`ShellBags`的信息都会更新，而且包含时间戳信息。是`Windows`系统改善用户体验的功能之一。即使删除文件夹后，`ShellBags`仍然会保留文件夹的信息。因此可以用来揭示用户的活动。

##### 0x1、ShellBags 的用途和取证中的价值

微软从`Windows 7`开始引入`ShellBags`，虽然在`Windows xp`中也存在，但是其文件格式发生了很大的改变。并在后续的系统上一直使用。`ShellBags`用来保存用户浏览文件夹时的偏好信息，比如文件夹的排列方式，文件夹显示图标的大小等，比如将文件夹的显示方式从 "大图标" 模式改为 "详细信息模式"，`ShellBags`会立即创建或更新记录，当你打开、关闭或者右键单击、或者重命名文件夹时，也会创建或更新`ShellBags`记录。这意味着：

1、如果在`Windows ShellBags`记录了某个文件夹，那么表示它一定在某个时间出现过在该系统中，包括压缩文件在内的本地文件系统、网络位置和外接设备（如 U 盘、移动硬盘等）上的文件夹，即使它现在已经不存在了。

2、由于这些对文件夹的操作和查看首选项与该用户的注册表配置单元（`registry hives`）相关联。所以我们可以将特定用户和特定的文件夹相关联，甚至，还可以从`ShellBags`包含的`MAC`时间戳中获取文件夹的访问时间信息。

##### 0x2、`ShellBags`的存储

在`Windows XP`中，存储在`NTUSER.dat`文件中，在系统注册表中的位置分别是：

*   记录网络路径文件夹访问的记录在：`\Software\Microsoft\Windows\Shell`
    
*   记录本地文件夹访问的记录在：`\Software\Microsoft\Windows\ShellNoRoam`
    
*   可移动存储器文件夹访问的记录在：`\Software\Microsoft\Windows\StreamMRU`
    

但是从`Windows 7`开始，已经发生了很大的变化，`Windows 7`新增了一个用户特定的注册表配置单元：`USRCLASS.dat`。这个配置单元支持新的用户访问控制（UAC）和强制访问控制完整性级别。它用于记录来自无权写入标准注册表配置单元的用户进程的配置信息。所以如果要获取完整的`ShellBags`信息，需要为每个用户解析`NTUSER.dat` 和 `USRCLASS.dat`这两个文件。这些文件可以在`%userprofile%`、`%userprofile%\AppData\Local\Microsoft\Windows`路径中找到。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0xc7LECKGV6iaic7UZGTAcZ9sZCCm5fRxR1ibwW5l0Yx87tzyXKVDt2wSw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0dliaibcibiagLpsxKp6UywXiaTQBKYKGp6icC8L7wxiaUqE9icl0A6cMZdunVA/640?wx_fmt=png)

具体的注册表位置为：

记录最近通过资源管理器访问的文件夹的信息：

`USRCLASS.DAT`：`HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`

`USRCLASS.DAT`：`HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\Bags`

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0KGowJaAtpjibUdLKXR89F739QEkVrFub3DvcibT0IEzwm0cSrXplPeFw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0lTTusdt2gwXHmxRLjbXsl2q2PicOoNiaaicH1jGcTD0yIuIWhaqKsk8vA/640?wx_fmt=png)

记录从桌面访问的文件夹信息：

`NTUSER.DAT`：`HKCU\Software\Microsoft\Windows\Shell\BagMRU`

`NTUSER.DAT`：`HKCU\Software\Microsoft\Windows\Shell\Bags`

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0ibZYBhFE9vSAYH3ic3L6NGYsGmYwYTVGMF9mEicGTb1SSUjv01GG7GoDA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0AsEXbfjyURRUic8ciaA47aS7SbibpjvoraLIaXh0CDntkVsCJLc0Dw8UA/640?wx_fmt=png)  

还有其他一些注册表位置：（因为系统版本不同，可能有一些附加的注册表项目）

`NTUSER.DAT`：`HKCU\Software\Microsoft\Windows\ShellNoRoam\BagMRU`

`NTUSER.DAT`：`HKCU\Software\Microsoft\Windows\ShellNoRoam\Bags`

`USRCLASS.DAT`：`HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\ShellNoRoam\BagMRU`

`USRCLASS.DAT`：`HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\ShellNoRoam\Bags`

`USRCLASS.DAT`：`HKCU\Software\Classes\Wow6432Node\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`

`USRCLASS.DAT`：`HKCU\Software\Classes\Wow6432Node\Local Settings\Software\Microsoft\Windows\Shell\Bags`

可以发现`ShellBags`的数据主要存在两个注册表键值中，分别是`BagMRU`和`Bags`中。

`BagMRU`：用于记录文件夹名称和文件夹路径

`Bags`：记录文件夹的视图配置，比如窗口的大小，位置，排序方式等视图模式信息。

##### 0x3、ShellBags 的结构分析

当用户通过`Windows`资源管理器浏览文件系统的时，首次打开个文件夹时，系统会创建`ShellBags`条目。每个文件夹都有一个编号，编号从`0`开始记录。当打开子路径，会在相应的`ShellBags`条目右侧添加一个条目，并分配一个编号，编号每次递增`1`。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0IElbowbZ3oianOw1FpiawNCu05Fib4uo9bPvMdkQ1SaTj17ab8MXqZ9eA/640?wx_fmt=png)

当用户对文件夹进行操作，`ShellBag`条目会立即更新。这意味着相应的注册表项最后修改时间可能也是最后操作文件夹的时间。

**`BagMRU`键值：**

`ShellBags`的根目录，它的子键包含了`ShellBags`子条目和子目录。由数字编号`0,1,2...`组成。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0icrIcEOqaZMkEpweDALd5LCYtXe3zWUN0icuBr0Eic0MbKRwNOGZ2SiaEw/640?wx_fmt=png)

而他们的二进制类型的值项（`value name`）记录着文件夹的路径和文件夹的长短名称。因此，我们可以根据这个结构还原文件系统的目录结构。

我们可以通过`Nirsoft`的注册表修改监视工具和`Shell bags view`工具结合注册表查看其创建和变化过程。

首先在`C`盘下创建一个`shellbagstest`的文件夹

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0tuVUoOzTVCdMWoePmibqQPIxEpTanCvod9s7dkUbWlLJOPU9XDP6efQ/640?wx_fmt=png)

查看注册表修改：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB062Xs7eGAWpa29JHOdabqduqFRbLx00JmX37zZibqg1AiaDNpiauKMDn3g/640?wx_fmt=png)

在注册表中查看：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0ibLdYdKdiazicx7CU5GPZlsViaL1kdldcxjfQNK62NafwEDA4z6XMSuOjg/640?wx_fmt=png)

可以看到`HKEY_CURRENT_USER\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\BagMRU\2\0`键的值项`"11"`的内容记录了文件夹的名称，并与其子键`"11"`对应。  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0OqJ4Nc3dHOyhF50sWZ7cTkJ3icIibXsuTpqEdGr7zPV4tTwwAjr9XhnA/640?wx_fmt=png)

`HKEY_CURRENT_USER\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\BagMRU\2\0\11`键的`NodeSlot`项值即为`HKEY_CURRENT_USER\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\Bags`键中的`ID`(`Bags`键的子健名称）。

还有一个`MRUListEx`，这个记录了上次访问了哪个文件夹。根据这个可以解析出文件夹的访问次序。当对应文件夹下没有子文件夹或者子文件夹未被访问过，其值为`ff ff ff ff`。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0odBsj53BMN58c0ibNiarnXwquy4wcCjwibfhCyh2dTa0NRuCwGnBMEjfQ/640?wx_fmt=png)

当我们进入这个文件夹并添加一个子文件夹后，再次查看注册表修改：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0124jlpEcEhKibI5kkWRIPibeNBnGHbLFmDhFQAt7KNxmVSfJhfvWicaIg/640?wx_fmt=png)

可以看到新增了多个键值项，`MRUListEx`也发生了变化。其中`"1"`为子文件夹`"subdir"`。而`"0"`是新建文件夹时候的`"新建文件夹"`。  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0bqFqma1tF8wXIvn3Yy9gjhFFK74nQ4RxnXMk0YkRIRDXCcWbmqktpw/640?wx_fmt=png)

**关于`MRUListEx`值项**

它是记录文件夹访问次序的，每 4 个字节记录一个文件夹的数字编号，新增访问记录，原记录往右边移。拿刚刚这个举例：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0C5WvwpNdIyzxmWEH6L2pkOnSQmwOASuurMmDgpu4ECNXx3WaNA5OkA/640?wx_fmt=png)

最左侧`0x00000001`表示最近访问过的是`shellbagstest`文件夹下数字序号为`"1"`的文件夹，也就是`subdir`这个文件夹。如果再增加一个文件夹`subdir2`：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB07fs9w7sqck4VGDbGJhbL58Q05Vq5a3jbMahLM1c7pSJehRaLAMicNVw/640?wx_fmt=png)

`MRUListEX`的值则变成如下数值：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0p1uBKS6qOHN78n11ibhMkg3zPzfNb2DQojB17iaYZDEXSuz3xX5icrHVA/640?wx_fmt=png)

**`Bags`键值**：

`Bags`键值由数字命名的子健组成，每个子健的数字编号对应特定的文件夹。其下包含有一个`Shell` 的键用于存储与文件夹相关的视图配置信息，比如位置、大小、排序方式等。所以我们可以根据`BagMRU`键值获得特定文件夹在`Bags`键下对应的数字序号后, 即可`Bags`键中定位相应子键, 进而查看文件夹的视图配置信息。

比如刚刚我们创建的文件夹：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0lm5iaNCGBRq4RviaMcV52kjxLZATu0WcyRdcIKohU2qJjShD9no0TbnA/640?wx_fmt=png)

##### 0x4、取证实战

案情：在之前的调查（Windows 取证之注册表）中，我们证明了`Theon`趁`Podrick`不在的时候把`U`盘（2020 年 2 月 3 日下午 12 点 15 分 - 12 点 45 分之间）插进了他的电脑。`Podrick`称他电脑的一些文件 / 文件夹发生的更改。他认为是`Theon`干的。`Podrick`希望我们帮忙找出是哪些文件发生了更改。主要关注桌面上被清空的`Projects`文件夹。

提交`Projects`文件夹被`Theon`重新创建的时间。

提供给我们的文件包括`NTUSER.DAT` 和 `UsrClass.dat`。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0AQTP7jRMM7h07aFZfKU5Nt90cmhicJ0HFSHrBvIgatrGw1mwKKQO0Rw/640?wx_fmt=png)

我们可以借助`ShellBags`解析工具，如`Shell Bag Explorer`：https://f001.backblazeb2.com/file/EricZimmermanTools/ShellBagsExplorer.zip  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0XKMQ3tRJ1OPxKEvRb5JCMpsRibW2gj9iaCNZYvfUICZ26tvvfhpGdWicw/640?wx_fmt=png)

使用工具加载`USRCLASS.DAT`文件

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB0TD9tib8FsZB3NluUQvVC9Sj9zkQspcDEOU4ZQqS5IxWwc1wLWMq9r5g/640?wx_fmt=png)

找到`Projects`文件夹，可以看到文件夹的创建时间是`12:41:26` ，这个时间点正好是`Podrick`不在电脑旁边的时间。结合之前的调查，即可确定这个时间就是文件夹重新创建的时间。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfPnRIKxicsJ0TzYQCdf8GB04ib4afJqRbzqriaK33YAHjfpMXEmGmWkiaUd5FUlz4CKOtNzGlgicgL0Sg/640?wx_fmt=png)

参考资料：

基于注册表中的 ShellNo-Roam 表键解析文件夹操作行为：http://www.xsjs-cifs.com/article/2014/1008-3650-39-3-42.html

Computer Forensic Artifacts: Windows 7 Shellbags ：https://www.sans.org/blog/computer-forensic-artifacts-windows-7-shellbags/

Explaining the Bags/BagMRU registry tree (trying) - Tielen Consultancy ：https://www.jeroentielen.nl/explaining-the-bagsbagmru-registry-tree-trying/

**相关实验：内存镜像取证**  

https://www.hetianlab.com/expc.do?ec=ECID6a2f-ed6f-4f85-9363-731535a5c3c4&pk_campaign=weixin-wemedia#stu  

了解常用的内存镜像取证工具的使用，包括 Dumplt、FTK Imager、Belkasoft RAM Capture 和 Dump 镜像内存提取工具。  

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LfbzQb75ZqoK2T2YO9XTQYD0aDUibvcxdbLRqzCwlkYcn0HppvXpZuenRzjX8ibhzcibJJge9Bw9xc8A/640?wx_fmt=gif)

  

戳

  

“阅读原文”

  

  

体验免费靶场！