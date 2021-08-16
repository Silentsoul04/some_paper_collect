\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/aikongmeng/articles/10657479.html)

[逆向神器之 IDA 的使用](https://www.freebuf.com/column/157939.html)
-----------------------------------------------------------

逆向工程作为一个新兴的领域，在软件维护中有着重要的作用。充分利用逆向工程技术就可以对现有系统进行改造，减少开发强度，提高软件开发效率，降低项目开发的经济成本，提高经济效益，并在一定程度上保证软件开发和利用的延续性，而 IDA 在逆向分析有着非常重要的作用。

### IDA pro 7.0 版本

用到的工具有 IDA pro 7.0  ，被反汇编的是百度云（BaiduNetdisk\_5.6.1.2.exe）。

首先，IDA pro 的长相如下：

[![](https://image.3001.net/images/20171221/15138426523644.png!small)](https://image.3001.net/images/20171221/15138426523644.png)

共有（File , Edit , Jump , Search , View , Debugger , Options , Windows , Help）9 个模块，还有下面的诸多小菜单。

现在我们点击 File，选择 Open 打开一个文件，这里我们选择百度云盘 PC 端安装程序，出现如下图示：

[![](https://image.3001.net/images/20171221/15138430897407.png!small)](https://image.3001.net/images/20171221/15138430897407.png)

这里我们直接默认 OK 即可。

此时，我们看到的视图是这样的：

[![](https://image.3001.net/images/20171221/15138433035385.png!small)](https://image.3001.net/images/20171221/15138433035385.png)

然后我们对各个部分进行标号，单独进行介绍：

[![](https://image.3001.net/images/20171221/15138437034339.png!small)](https://image.3001.net/images/20171221/15138437034339.png)

第一部分表示的是对不同代码块使用不同的颜色进行区分，我们可以直接点击相应的颜色块进行不同代码块的定位。

蓝色：表示代码段。

棕色：表示数据段。

红色：表示内核。

第二部分表示该程序的函数表，双击后可查看详细信息。

[![](https://image.3001.net/images/20171221/15138445267921.png!small)](https://image.3001.net/images/20171221/15138445267921.png)

该函数对应的 IDA View-A 如下：

[![](https://image.3001.net/images/20171221/15138446727920.png!small)](https://image.3001.net/images/20171221/15138446727920.png)

第三部分对应的就是整体程序或者某个函数的图标概述形式，可以大体把握功能和结构的走向。对整体的脱壳逆向有很大的帮助。

[![](https://image.3001.net/images/20171221/15138450057437.png!small)](https://image.3001.net/images/20171221/15138450057437.png)

第四部分主要可以显示以下 6 部分信息：

> （1）IDA View-A
> 
> （2）Hex View-1
> 
> （3）Structures
> 
> （4）Enums
> 
> （5）Imports
> 
> （6）Exports

其中 IDA View-A 表示的就是某个函数的图标架构，可以查看程序的逻辑树形图，把程序的结构更人性化地显示出来，方便我们的分析。

具体表示形式，上文中有截图可参考。

在 Hex View-1 中可以查看 16 进制代码，方便定位代码后使用其他工具修改，具体表示如下图所示：

[![](https://image.3001.net/images/20171221/15138457269437.png!small)](https://image.3001.net/images/20171221/15138457269437.png)

在 Stuuctures 中可以查看程序的结构体：

[![](https://image.3001.net/images/20171221/15138459472752.png!small)](https://image.3001.net/images/20171221/15138459472752.png)

在 Enums 中可以查看枚举信息：

[![](https://image.3001.net/images/20171221/15138460185110.png!small)](https://image.3001.net/images/20171221/15138460185110.png)

在 Imports 中可以查看到输入函数，导入表即程序中调用到的外面的函数：

[![](https://image.3001.net/images/20171221/15138461514350.png!small)](https://image.3001.net/images/20171221/15138461514350.png)

在 Exports 中可以查看到输出函数：

[![](https://image.3001.net/images/20171221/15138462178028.png!small)](https://image.3001.net/images/20171221/15138462178028.png)

以上就是 IDA 主面板中的各个部分的功能介绍了。

接下来我们介绍 9 个菜单模块，即：

> File , Edit , Jump , Search , View , Debugger , Options , Windows , Help

[![](https://image.3001.net/images/20171221/15138465548768.png!small)](https://image.3001.net/images/20171221/15138465548768.png)

1.File 是用来打开，新建，装载一个应用程序的，这大家都知道的。

2.Edit 是用来编辑反汇编代码的，可以复制，筛选什么的。

3.Jump 是用来跳转的，可以有很多种类型的跳转，比如跳转到上一个位置或者下一个位置，跳转到某个指定的地址。还可以根据名字，函数来进行跳转，跳转到一个新的窗口，跳转某一个偏移量等等，总之很多了，具体大家可以慢慢积累了。这个模块就比较重要了。

4.Serach 是用来搜索的。

5.View 是用来选择显示方式的，或者显示某一特定模块信息的。比如以树形逻辑图显示，或者 16 进制形式显示。还可以单独显示某一特定信息，比如输入或者输出表等。

6.Debugger ，调试器被集成在 IDA 中，首先我们使用 IDA 装入文件，来生成数据库，用户可以使用反汇编功能，查看所有反汇编信息，这些均可以在调试器中进行和使用。

[![](https://image.3001.net/images/20171221/15138477568732.png!small)](https://image.3001.net/images/20171221/15138477568732.png)

7.Options ，在这里可以进行一下常规性的设置。

8.Windows，

9.Help，使用 IDA 的一些帮助文档，检查更新等等。

* * *

使用 IDA 的一个大体步骤：

1\. 装入文件或程序

2\. 指令断点

3\. 程序运行

4\. 分析堆栈

5\. 添加监视

6\. 进行地址分析

7\. 单步跟踪

8\. 找到 bug

9\. 使用硬件断点进行 bug 确认

* * *

本期就介绍到这里了，IDA 有很多高级功能，会在之后的实战文章中进行讲解。

下一期我们将讲解《逆向动态调试之 Ollydbg 的使用》。

学完汇编，IDA，Ollydug 我们就可以进行简单的逆向分析了。