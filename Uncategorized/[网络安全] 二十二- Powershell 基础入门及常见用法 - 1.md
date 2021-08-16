> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sNSOFcV5nnzY4jUW9UO2pg)

作者前面详细介绍了三种类型的漏洞利用普及，包括 Chrome 浏览器和音乐软件。**这篇文章将讲解 Powershell 基础入门知识，包括常见的用法，涉及基础概念、管道和重定向、执行外部命令、别名用法、变量定义等。Powershell 被广泛应用于安全领域，甚至成为每一位 Web 安全必须掌握的技术。**

本文参考了 Bilibili 的老师的课程，同时也结合了作者之前的编程经验进行讲解。作者作为网络安全的小白，分享一些自学基础教程给大家，希望你们喜欢。同时，更希望你能与我一起操作深入进步，后续也将深入学习网络安全和系统安全知识并分享相关实验。总之，希望该系列文章对博友有所帮助，写文不容易，大神请飘过，不喜勿喷，谢谢！

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.html

话不多说，让我们开始新的征程吧！您的点赞、评论、收藏将是对我最大的支持，感恩安全路上一路前行，如果有写得不好或侵权的地方，可以联系我删除。基础性文章，希望对您有所帮助，作者目的是与安全人共同进步，加油~

文章目录：

*   **一. Powershell 初识**
    
    1. 基础概念
    
    2. 为什么强大？
    
    3. 控制台和快捷键
    
    4. 数学运算  
    
*   **二. Powershell 管道和重定向**
    
    1. 管道
    
    2. 重定向  
    
*   **三. Powershell 执行外部命令及命令集**
    
    1. 外部命令
    
    2. 命令集  
    
*   **四. Powershell 别名使用**
    
    1. 别名基本用法
    
    2. 自定义别名
    
*   **五. Powershell 变量基础**
    
    1. 基础用法
    
    2. 变量操作
    
    3. 自动化变
    
    4. 环境变量
    
*   **六. Powershell 调用脚本程序**
    
    1. 脚本文件执行策略
    
    2. 调用脚本程序
    
*   **七. 总结**  
    

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。

一. Powershell 初识
================

1. 基础概念
-------

Windows PowerShell 是一种命令行外壳程序和脚本环境，使命令行用户和脚本编写者可以利用 .NET Framework 的强大功能。它引入了许多非常有用的新概念，从而进一步扩展了您在 Windows 命令提示符和 Windows Script Host 环境中获得的知识和创建的脚本。

传统的 CMD 支持脚本编写，但扩展性不好，而 Powershell 类似于 Linux shell，具有更好的远程处理、工作流、可更新的帮助、预定任务（Scheduled Job）、CIM 等优点。

那么，如何进入 Powershell 呢？  
一种方法是在运行中直接输入 Powershell 打开，另一种方法是 CMD 中输入 Powershell 打开。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9hjiaYOGZqjoUrzcNuYIfuBXYoDZXyRFezbOJicr2mtLQby6wTbvq827A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT94QqcAuRMHPvn05eWjbibxicuDQgZ0RduK1fe7c3E8GU6BpIklOiax2PRA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9CbqjWic5FMcxsdJpNqF8R5ONBWeU1icHkjQdT4F61cMP1oZjgTfR4Tjg/640?wx_fmt=png)

不同操作系统内置的 Powershell 是不一样的，比如 win7 或 win2008，如何查看版本呢？

```
$psversiontable
```

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9d19Rvr5vyceUjIQX8dUzbaevMIibIAmbRFrRToELyzOb2icqk8oeawag/640?wx_fmt=png)

2. 为什么强大？
---------

首先，它可以进行计算任务，包括计算 1gb 大小（以字节为单位），还有基本的运算。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT99W24XdFz4CkG7gp41HSesia2NCk7djyJiauD0NJzFNX8WaJo1wDyLicQQ/640?wx_fmt=png)

其次，Powershell 可以获取计算机的服务详细信息、状态等。

```
get-service
```

其显示结果如下图所示，采用动词 + 名词方式命名，比较清楚。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9v3ZL2zIDqOqXUm4eVEDTibNEBicLf7NX5qVoMIzCjaarqPiaSf7bjBfuA/640?wx_fmt=png)

而 CMD 中无法获取 services 的（输入 services.msc），它是以图形化方式显示出来的。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9jd7PuYqQRCHrgp3GtribkXKxGmgqfy42LecFn2mFfibNRrqfF5Y4lrdA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9PwLtmwdDNlLwEYSIIpDE4U8gdg6QcyMoRq3HS5icw3ibEIS6ZQKSCq3A/640?wx_fmt=png)

最后，由于 Powershell 具有以下特点，它被广泛应用于安全领域，甚至成为每一位 Web 安全必须掌握的技术。

*   方便
    
*   支持面向对象
    
*   支持和. net 平台交互
    
*   强大的兼容性，和 cmd、vbs 相互调用
    
*   可扩展性好，它可以用来管理活动目录、虚拟机产品等平台
    

3. 控制台和快捷键
----------

鼠标右键属性，可以对 Powershell 控制台进行编辑，并且它支持两种编辑模式，快速编辑模式默认钩上的。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9RHjWRKQDvicO5LaV4ZZOSy1AXMyltdDjsI84TLawkxnReg1TrmJzybw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9eFTBdNb7py3PZhsHxIp2uXFpZWPbZSPS7roNzNSfibASaPymKu26gww/640?wx_fmt=png)

Powershell 快捷键包括：

```
ALT+F7      清楚命令的历史记录PgUp PgDn   翻页Enter       执行当前命令End         将光标移动至当前命令的末尾Del         从右开始删除输入的命令字符Esc         清空当前命令行F2          自动补充历史命令至指定字符处F4          删除命令行至光标右边指定字符处F7          对话框显示命令行历史记录F8          检索包含指定字符的命令行历史记录F9          根据命令行的历史记录编号选择命令，历史记录编号可以通过F7查看 左/右        左右移动光标上/下        切换命令行的历史记录Home        光标移至命令行字符最左端Backspace   从右删除命令行字符Ctrl+C      取消正在执行的命令Tab         自动补齐命令或文件名
```

例如，使用快捷键 Ctrl+C 打断了正在运行的 ping 指令；使用 tab 快捷键补齐了 service.msc 命令。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9hianreiaQ7dEUtxpVrDPicicDBMBxIQNWLKS2QzI0CDSQRw8aib5ksKAI3A/640?wx_fmt=png)

4. 数学运算
-------

Powershell 支持数学运算，比如：

```
PS C:\Users\yxz> 2+4
PS C:\Users\yxz> 4-2
PS C:\Users\yxz> 4*3
PS C:\Users\yxz> 9%2
PS C:\Users\yxz> (1+3*5)/2
PS C:\Users\yxz> 1gb/1mb
PS C:\Users\yxz> 1gb/1mb*18kb
PS C:\Users\yxz> 1gb -gt 1mb
True
PS C:\Users\yxz> 0xabcd
```

显示结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9wEGBxFeMEHvOUDSemA10GEGsibC0ibFZ4xgaJb9vb0uM0sbMbVIGOuVg/640?wx_fmt=png)

二. Powershell 管道和重定向
====================

1. 管道
-----

Powershell 管道旨在将上一条命令的输出作为下一条命令的输出。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9G1Nf6iaGchP5NFgxPcOyPjfVs2JP1KnR9AvOWzEian1TRoxa7eMKn6Pg/640?wx_fmt=png)

管道并不是什么新事物，以前的 Cmd 控制台也有重定向的命令，例如 Dir | More 可以将结果分屏显示。传统的 Cmd 管道是基于文本的，但是 Powershell 管道是基于对象。例如：

```
linux：ls
cmd：dir
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9ib4tHKNLjtBab7IHAHoUSkVUwyDcfHHpv4oFPedvd1VON6x1gaS8lNA/640?wx_fmt=png)

如果只获取其中的 name、mode 值，则使用如下指令。

```
ls | format-table name, mode
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9re94msLeGDhBBf4O3kZrla1D9YlJwWYNlhwibm61H4fAwuhPaK51KbA/640?wx_fmt=png)

2. 重定向
------

重定向旨在把命令的输出保存到文件中，‘>’为覆盖，’>>’追加。

```
ls | format-table name, mode > demo.txttype demo.txt
```

上面代码是将 ls 显示文件内容的 name 和 mode 信息存储至本地 demo.txt 文件夹中，再调用 “type demo.txt” 打印文件内容。如果两个 >> 它会在原来的基础上，再进行补充（类似 a+），而单个大于号是删除原来的写入（类似 w）。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9xfJYEU5fhCMWxibNo21OEj0JsUZt7SFicuehUUrOtpoUutEGym5BKKhg/640?wx_fmt=png)  
输出结果如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT95Im6np4KmibMYWic5IqVawzeEwR3bJEbp4dsTKoe6pwCpMic6eQd41CkA/640?wx_fmt=png)

三. Powershell 执行外部命令及命令集
========================

1. 外部命令
-------

Powershell 是 CMD 的一个扩展，仍然能够让 CMD 中的命令在 Powershell 中使用，Powershell 初始化时会加载 CMD 应用程序，所以 CMD 命令正常情况下在 Powershell 中都能使用，例如 ipconfig。

查看端口信息

```
netstat -ano
```

包括协议、本地地址、外部地址、状态、PID（进程号）。  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9iaCVVTThOn4eOtmZTYrHVte9ibzdr8JKSB9oDhQ62BMdYgojETx2xMxQ/640?wx_fmt=png)

查看网络配置信息

```
ipconfig
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT94iaCBspBatuFy3tF6WMdShvGkHE8cCRs4EsGQdnz59rN5UYSrTME57w/640?wx_fmt=png)

打印路由信息

```
route print
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9zjnbQnx1M8c7FQWg8lRh9eskkSzWDSoCM4ic0RoAZQwvBYWysdtWk1A/640?wx_fmt=png)

自定义文件路径，打开应用程序

```
start notepad
notepad
```

notepad 放在 C 盘下面的 Windows\System32 文件中，能够直接打开。  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9r2v51gOC8fc1YAicb6LCjeNibp2CHVQ3kFk6GIPBkJcee2rLZ4ia3XCdA/640?wx_fmt=png)

系统变量

```
$env:path
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9fIEhwnaffC0LicWMicCEPAMkyEyNWWhYbFJm4YKxiaQnjfyYycaqfAHOw/640?wx_fmt=png)

Python 可以直接打开，Wordpad 不能打开，需要添加环境变量中。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9Jfoq4OcKrYOXsIdfkGqngUibLqevvfABDUfQV43LYSl1YegLEjvGPEw/640?wx_fmt=png)

2. 命令集
------

通过 get-command 获取所有命令，通常是动名词的方式。

```
get-command
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9HT23ADJOFB915szsaTGQPDpEricicc8GB8BibqLeovlzDe3A0F6a1OIvw/640?wx_fmt=png)

获取其用法的命令如下，简称 gcm。

```
get-help get-command
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9kJYaI8hwGkAldpOTr5LeOKCibNpTib1V7fxHIo9wAHk84kFVbJXrTYeA/640?wx_fmt=png)

获取进程信息

```
get-process
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9Va115lZaFDQvFd2OOYXGrN9t982hghOFrFfUic8sgZtibAJdaRBKWiaxQ/640?wx_fmt=png)

获取当前会话的别名

```
get-alias
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9eNLUkEtOOaw7h8EOdTib9Co5SKtkbs9Or2PTjgxvdmib3iaW87vcDl3hw/640?wx_fmt=png)

获取输入的历史命令信息

```
get-history
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9jx0MxYQRMibldxjplaJWB76VORUiaicUmaIAPcU5gBWjsnWugsde52zcg/640?wx_fmt=png)

获取当前时间

```
get-date
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT95HFzvEfSIxAoKe1EDtJAPmib8sN3icxGjeA7RnbAejkgHrzzbZ7ccZpg/640?wx_fmt=png)

四. Powershell 别名使用
==================

1. 别名基本用法
---------

获取所有命令 get-command 可以用别名 gcm 替代。

```
get-commandgcm
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9HT23ADJOFB915szsaTGQPDpEricicc8GB8BibqLeovlzDe3A0F6a1OIvw/640?wx_fmt=png)

获取当前目录的所有文件信息 get-childitem，可以用 ls、dir 两个命令达到同样的效果。

```
get-childitemlsdir
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9ZoCWKK82hFZThDvxGTZrU2mGvoIk221z0GnicUBz1ozJwS83tY8ngiaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9m1ictCBQWMkflnzdXvRB5Fh6DBmnPdleyiaqBIkU1QBxSQ49savfIkGw/640?wx_fmt=png)

获取相关的帮助信息，其命令如下：

```
get-help get-childitem
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9iaRW42InYSEHfJ6YibyqeRAIA5L7tiaJ28v1O41aKSdCibd2wiaQ7Vze6Vw/640?wx_fmt=png)

获取别名所对应真实的命令

```
get-alias -name lsget-alias -name dir
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9bRovfWz8ibHksia2Z8FfPrda0BvibV7OCffic5ic569nGdwJu1iaWIahPiaiag/640?wx_fmt=png)

查找所有以 Remove 开头的别名

```
get-alias | where{$_.definition.startswith("Remove")}
```

其中，where 来做一个管道的筛选，$_表示当前的元素，definition 定义一个字符串数组类型。Powershell 支持. net 强大的类库，里面的 definition 包括字符串 startswith 操作，获取字符串开头函数。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9lAEfe0kpcIG7U7JjfyIRjmYtFf2gyIfRFhtJOCZDCMficLEAHdl0QVw/640?wx_fmt=png)

查找所有别名，并调用 sort 降序排序及计算排列。

```
get-alias | group-object definition | sort -descending Count
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9Qqhiam17sLflkFB2qUsjlOBUrVibPcnepgRBavXQWtKxQSdC3CfSHqoQ/640?wx_fmt=png)

注意：自定义别名是临时生效的，当关闭 Powershell 时就会失效。

2. 自定义别名
--------

设置别名，将 notepad 设置为新的别名 pad。pad 打开 notepad，表明我们的别名创建成功。

```
set-alias -name pad -value notepad
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT93JCtZJx7KgEX6h9jALttBxMhDteCIPa758YqgQCG5Aea0noYWWuNzg/640?wx_fmt=png)

别名是临时生成的，关掉 Powershell 即可失效，也可以撰写命令删除。

```
del alias:pad
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT94J62BljDO4ZLOvcYApgWvD4xQRU4wricT56H8dCJ30ZtQIry8eeicKDw/640?wx_fmt=png)

保存别名

```
export-alias demo.psdirtype demo.ps
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9FbRg8YUOpK6VCr2XbVHzGsjiaOKwzd5gDMXknGlMHA8AibJ4qTBTH8EA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9qJFm2mswJWgqn9nHBaYhpyDwLAGdBIAyz7ic4sm0icUBkhYLgn4oecLA/640?wx_fmt=png)

导入别名命令如下，其中 - force 表示强制导入。

```
import-alias -force demo.ps
```

五. Powershell 变量基础
==================

1. 基础用法
-------

Powershell 变量跟 PHP 很类似，如下所示。

```
$name='eastmount'$name$age=28$age
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT95RnTujCjRznTUm3GMMAfZWE9Aau4mtH2WEtZf0FZdfePGjzl59RupQ/640?wx_fmt=png)

Powershell 对大小写不敏感，$a 和 $A 一样。复杂变量用大括号引起来，但不建议同学们这里定义。

```
${"I am a" var ()}="yxz"${"I am a" var ()}$n=(7*6+8)/2$n=3.14
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9W6d1iaR2dOOwGAXgP2ibexOx5iaqIHS9KT9u8ZqsKpT7SMOYYUAWJ01wA/640?wx_fmt=png)

变量也可以设置等于命令。

```
$n=ls
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT95vxps5WkWgVWjqSXecwssljozu5FUj1TZgKpRia2iaJCAoKvuTvbS9ibA/640?wx_fmt=png)

变量多个同时赋值，但不建议这么写。

```
$n1=$n2=$n3=25$n1,$n2,$n3
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9nOFvI6KSR3op7IibZb3ia0wicMCHYyshVXMc0XjDVOico6sMHdSQVicF10w/640?wx_fmt=png)

2. 变量操作  

----------

变量的基本运算操作

```
$a=2$b=10$c=a+b$a,$b,$c
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9Bp4U9nLcC8Xfx3y3FYA48uTyhic02Zxv5p3woRXCYNPwTOttAODlvhg/640?wx_fmt=png)

传统变量交换方法

```
$num1=10$num2=20$temp=$num1$num1=$num2$num2=$temp$num1,$num2
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9rllHKCmbaVBrTxAaibYedm6Fe0lzI1jwtd1Fgwz6cuIGkkQB2Llv84A/640?wx_fmt=png)

现在变量交换的写法

```
$num1=10$num2=20$num1,$num2=$num2,$num1$num1,$num2
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9Pp3dEhGGMPQ8yCOIpNkzJqgXyVoqzGMvX7kGCc3TC5NyXyIBLic03tA/640?wx_fmt=png)

查看当前的变量

```
ls variable:
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9yoM9RVf7sSOq5J5N8eMhRn5PMfbzBRRlslYu4Tl0MtYDuUzswLfdDg/640?wx_fmt=png)

查找特定的变量值，星号表示代替所有的值（num 开头）。

```
ls variable:num*ls variable:num1
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9DTUicyBrFD9Yecxx71oKUJ1U7b6P6ZuvHFhP2klqDMnFHelE4YaoKHA/640?wx_fmt=png)

查找变量是否存在

```
test-path variable:num1test-path variable:num0
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9FaUKYxtCffZibc6qEZXhtbjuvUgeib4XFtywhSMeyZbZbQAMZf8g7mjA/640?wx_fmt=png)

删除变量

```
del variable:num1test-path variable:num1
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9bnA8x6Zy8hJVJlRXNVao5dkF4Melzx6lv5QC6CzhicNR0Yfey9dynNg/640?wx_fmt=png)

专用变量管理的命令

```
clear-variableremove-variablenew-variable
```

3. 自动化变量
--------

powershell 打开会自动加载变量，例如：窗口打开它会自动加载大小，再比如程序的配置信息自动加载。

根目录信息

```
$home
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9cOOF4hmhRhBcmxKNaFwyAaRdI4htibpMfuciafAajLs16xd467wZlaYg/640?wx_fmt=png)

当前进程的标志符，该自动化内置变量只能读取，不能写入。

```
$pid
$$
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9ibmUx0ZCxib91iaBOS07t7yba4QJibmtmyROGvYcoVMhGDsBHju88PwOicQ/640?wx_fmt=png)

4. 环境变量  

----------

查看当前环境变量

```
ls env:
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9F5HKNuho2Ea7D13ZS3SeGlEjqMZCe2QmObHMonJ6bicicDyrIPxO6kNg/640?wx_fmt=png)

打印某个环境变量的值

```
$env:windir
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9ZmVUUJkbNgMD1k3dX0fTXfY4qo27NnyX8wQy9Zz5AP2TTicfbkwxNXQ/640?wx_fmt=png)

创建新的环境变量

```
$env:name='eastmount'ls env:na*
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT985zKX0qRRqDdgwpnxTvFmPBcUCEiaPObBqSUgtyo44yMCpfsOTcct0A/640?wx_fmt=png)

删除环境变量

```
del env:namels env:na*
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT92jJWZibIdX11IsoMbFEoW0JT3PQKjib29zTE2Pwkic7hLGaEcOeWOHQ6A/640?wx_fmt=png)

更新环境变量，注意它只是临时生效，并不会记录到我们的系统中。

```
$env:OS$env:OS="Linux"$env:OS
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9cwt5NdUtDv8Avovp5LeGI7MbyueRN9fibibDvbd1PLGff6ic7hwicDL2Eg/640?wx_fmt=png)

永久生效如何实现呢？增加路径至环境变量 PATH 中，只对 User 用户生效。

```
[environment]::setenvironmentvariable("PATH","E:\","User")[environment]::getenvironmentvariable("PATH","User")
```

系统变量对所有用户都生效，用户变量只对当前用户生效。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT92XY6Xq2P6PBrjBIJvSQe5bezKkkRKdibvFFGsk89GAWw1NBNia8c7Hrw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9b6rdVficPPjFooWWN7YYw1LJgNDVO9G9xFFRELL5cJf2sF96IKsiagcQ/640?wx_fmt=png)

生效之后如下图所示，用户变量增加了相关值。  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9EWibHic6qFO9ibfYSup9rJzPoicOMjKmr5gZNMmicAxpdBYqovQyTRohS8g/640?wx_fmt=png)

六. Powershell 调用脚本程序
====================

1. 脚本文件执行策略
-----------

首先，发现我们的脚本文件是禁止执行的。

```
get-executionpolicy
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9wDMe8zRSCibmIdkkOzFssibTn1NISkv8RtLib4zAdoLg6V3hmyQ2KdrOw/640?wx_fmt=png)

接着，我们尝试获取策略帮助信息。

```
get-help set-executionpolicy
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT909OhwTEMemfO63ME7eXLepcqSzsxS4OxQzDd9bPx5wpDiao66vKcp4A/640?wx_fmt=png)

最后修改权限，让其能运行 Powershell 脚本文件。

```
set-executionpolicy RemoteSigned
```

它会提示你需要启动管理员身份运行。  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9T4eprTlVp4aybEMKaNWmWcdiaLiaaibcVt5eEwnGtLuJslcibkaFnrjRIA/640?wx_fmt=png)

通过管理员身份打开 CMD，再设置其权限即可，设置完成之后可以调用相关的脚本程序。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT962lUjajHz3wB0eGokkUIQs3icnE2owhvPEyh4gaMhicdnkGWEv0TibNUQ/640?wx_fmt=png)

2. 调用脚本程序
---------

(1) 定义一个 demo.bat 文件，其内容如下，关闭回写，打印 hello world。

```
@echo offecho hello world
```

运行命令打开：

```
cd desktop.\demo.bat
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9QkOfdHWEowQh4jpFGdOZwackluqoial23iaEJvRGAdMaJ4hrQkt6qiaaA/640?wx_fmt=png)

(2) 定义一个 demo.vbs 文件，内容如下：

```
msgbox "CSDN Eastmount"
```

运行命令打开：

```
cd desktop.\demo.vbs
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9Wo2TyI6W0edib3vSgx3JBicEltFyzu9GFuzBttZuMOVKOwc4NPibDMlDg/640?wx_fmt=png)

(3) 运行 Powershell 脚本文件也类似。

```
$number=49switch($number){	{($_ -lt 50) -and ($_ -gt 40)} {"此数值大于50且小于40"}	50 {"此数值等于50"}	{$_ -gt 50} {"此数值大于50"}}
```

运行结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9q4ZMW8s3jdsktPpzelqmt5gIzdNHibmnsr9Allcat8cl7dGJAqhkBiaw/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9lzatyHUlkJWZLdblA91PRic0Ybicicfz8cJ0Rdz8UfD9qEEHwmoSQRVRg/640?wx_fmt=png)

那么，如何在 CMD 中运行 Powershell 文件呢？  
我们将 demo.bat 修改为如下内容，其中 & 表示运行。

```
@echo offpowershell "&'C:\Users\yxz\Desktop\demo.ps1'"
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9zAk1LemFdhRlSnhuJUDLU8m8sG7H8C0dlWJ0M2KjJ1uMS7xAvtr5ibQ/640?wx_fmt=png)  
运行命令：

```
cd desktop.\demo.bat
```

下面方法也可以直接运行

```
start demo.batdemo.bat
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNAZm0UmEJbfRaTMhTzSTT9CWuXABNLHmgGR9GPZaN9CmqGnEJQzQkTsVkSNiaGT3YSod2OOcM8m6Q/640?wx_fmt=png)

七. 总结
=====

写到这里，这篇文章介绍结束，主要内容：

*   **一. Powershell 初识**
    
*   **二. Powershell 管道和重定向**
    
*   **三. Powershell 执行外部命令及命令集**
    
*   **四. Powershell 别名使用**
    
*   **五. Powershell 变量基础**
    
*   **六. Powershell 调用脚本程序**
    

如果你是一名新人，一定要踏踏实实亲自动手去完成这些基础的逆向和渗透分析，相信会让你逐步提升，过程确实很痛苦，但做什么事又不辛苦呢？加油！希望你能成长为一名厉害的系统安全工程师或病毒分析师，到时候记得回到这篇文章的起点，告诉你的好友。

学安全一年，认识了很多安全大佬和朋友，希望大家一起进步。这篇文章中如果存在一些不足，还请海涵。作者作为网络安全和系统安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，深知自己很菜，得努力前行。编程没有捷径，逆向也没有捷径，它们都是搬砖活，少琢磨技巧，干就对了。什么时候你把攻击对手按在地上摩擦，你就赢了，也会慢慢形成了自己的安全经验和技巧。加油吧，少年希望这个路线对你有所帮助，共勉。

前文分享（下面的超链接可以点击喔）：

*   [[网络安全] 一. Web 渗透入门基础与安全术语普及](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483786&idx=1&sn=d9096e1e770c660c6a5f4943568ea289&chksm=cfccb147f8bb38512c6808e544e1ec903cdba5947a29cc8a2bede16b8d73d99919d60ae1a8e6&scene=21#wechat_redirect)
    
*   [[网络安全] 二. Web 渗透信息收集之域名、端口、服务、指纹、旁站、CDN 和敏感信息](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483849&idx=1&sn=dce7b63429b5e93d788b8790df277ff3&chksm=cfccb104f8bb38121c341a5dbc2eb8fa1723a7e845ddcbefe1f6c728568c8451b70934fc3bb2&scene=21#wechat_redirect)
    
*   [[网络安全] 三. 社会工程学那些事及 IP 物理定位](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483994&idx=1&sn=1f2fd6bea13365c54fec8e142bb48e1d&chksm=cfccb297f8bb3b8156a18ae7edaba9f0a4bd5e38966bdaceeff03a5759ebd216a349f430f409&scene=21#wechat_redirect)
    
*   [[网络安全] 四. 手工 SQL 注入和 SQLMAP 入门基础及案例分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484068&idx=1&sn=a82f3d4d121773fdaebf1a11cf8c5586&chksm=cfccb269f8bb3b7f21ecfb0869ce46933e236aa3c5e900659a98643f5186546a172a8f745d78&scene=21#wechat_redirect)
    
*   [[网络安全] 五. XSS 跨站脚本攻击详解及分类 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484381&idx=1&sn=a1d459a7457b56b02e217f39e5161338&chksm=cfccb310f8bb3a06442b001fc7b38a0363b9fbd4436f450b0ce6fa2eeb5c796fc936ceb5d6fa&scene=21#wechat_redirect)
    
*   [[网络安全] 六. XSS 跨站脚本攻击靶场案例九题及防御方法 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485174&idx=1&sn=245b812489c845e875cf4bc4763747b7&chksm=cfccb63bf8bb3f2d537f36093de80dbeed5a340b141001d3ef8a9ac9d6336e0aaf62b013a54c&scene=21#wechat_redirect)
    
*   [[网络安全] 七. Burp Suite 工具安装配置、Proxy 基础用法及暴库入门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485381&idx=1&sn=9a0230cf22eba0a24152cb0e73a37224&chksm=cfccb708f8bb3e1ecf68078746521191921f41d19a0b82cb3f097856dad7a85c4d9c34750b3f&scene=21#wechat_redirect)
    
*   [[网络安全] 八. Web 漏洞及端口扫描之 Nmap、ThreatScan 和 DirBuster](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485437&idx=1&sn=2a7179464207fa68b708297ec0db6f00&chksm=cfccb730f8bb3e2629edb5ca114de79723e323512be9538a4d512297f8728a3a9d7718389b60&scene=21#wechat_redirect)
    
*   [[网络安全] 九. Wireshark 安装入门及抓取网站用户名密码 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485465&idx=1&sn=8e7f1f5790bfe754affe0599a3fce1ee&chksm=cfccb8d4f8bb31c2ca36f6467d700f4e4d7821899a6d5173ac0b525f0f6227c8392252b5c775&scene=21#wechat_redirect)
    
*   [[网络安全] 十. Wireshark 抓包原理、ARP 劫持、MAC 泛洪及数据追踪](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485551&idx=1&sn=15f00e14f4376e179a558444de8ef0a5&chksm=cfccb8a2f8bb31b456499a937598e750661841b5ca166a12073e343a049737fa3131fd422dc5&scene=21#wechat_redirect)
    
*   [[网络安全] 十一. Shodan 搜索引擎详解及 Python 命令行调用](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485599&idx=1&sn=0c60c042911fc79287417c2385550430&chksm=cfccb852f8bb3144a89f6b0d0df6c185a208aa989d98f8c7e3b7d741dedc371b3ecb4e70a747&scene=21#wechat_redirect)
    
*   [[网络安全] 十二. 文件上传漏洞 (1) 基础原理及 Caidao 入门知识](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485787&idx=1&sn=0c75cf81c4234031273bced4dff0b25c&chksm=cfccb996f8bb3080fe9583043b43665095fd6935a4147a2bb0d1ab9b91a6cde99da4747c5201&scene=21#wechat_redirect)
    
*   [[网络安全] 十三. 文件上传漏洞 (2) 常用绕过方法及 IIS6.0 解析漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485833&idx=1&sn=a613116633338ca85dfd1966052b0b02&chksm=cfccb944f8bb305296a32dac7f0942e727d66dc9f710bfb82c3597500e97d39714ecd2ed18cf&scene=21#wechat_redirect)
    
*   [[网络安全] 十四. 文件上传漏洞 (3) 编辑器漏洞和 IIS 高版本漏洞及防御](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485871&idx=1&sn=e6d0248e483dea9616a5d615f852eccb&chksm=cfccb962f8bb3074516c1ef8e01c7cb00a174fa5b1a51de3a49b13fd8c7846deeaf6d0e24480&scene=21#wechat_redirect)
    
*   [[网络安全] 十五. 文件上传漏洞 (4)Upload-labs 靶场及 CTF 题目 01-10](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488340&idx=1&sn=5b7bf5602294586f819340bd6190a34d&chksm=cfcca399f8bb2a8f746fc09c7142facc8ea17c008ba46dee423b90ff6abb3cd4486edf52d201&scene=21#wechat_redirect)
    
*   [[网络安全] 十六. 文件上传漏洞 (5) 绕狗一句话原理和绕过安全狗](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488396&idx=1&sn=67c1b13f041040c09c236bba99edfe0a&chksm=cfcca341f8bb2a5729778490db7441a4ddfdfa05dcc5f6322b4860db7780056f9f05f5bc0b3d&scene=21#wechat_redirect)  
    
*   [[网络安全] 十八. Metasploit 技术之基础用法万字详解及 MS17-010 漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488255&idx=1&sn=28b1f54fd420a0145cb95b842a36c567&chksm=cfcca232f8bb2b243bf4cbf5c1741c6af2c1fc666985d34b4f6b4a6ee3161d18975bb5ea18fc&scene=21#wechat_redirect)
    
*   [[网络安全] 十九. Metasploit 后渗透技术之信息收集和权限提权](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488639&idx=1&sn=dddd54eb0ba7cfdf71113a1f4a5c6548&chksm=cfcca4b2f8bb2da44c975ca12f16b4b76af351be4711ac7e77ca8622450a15c3af0172be3f9e&scene=21#wechat_redirect)
    
*   [[网络安全] 二十. Metasploit 后渗透技术之移植漏洞、深度提权和后门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488738&idx=1&sn=8106362219d99ae6deb8aeca1f6b1dff&chksm=cfcca42ff8bb2d397c44b839700d92fd22e4ac60c403b96cba734bc523cb258dbd0db5309952&scene=21#wechat_redirect)
    
*   [[网络安全] 二十一. Chrome 密码保存渗透解析、Chrome 蓝屏漏洞及音乐软件漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488883&idx=1&sn=65c362cc4c3958aa747716d17b29eeb3&chksm=cfcca5bef8bb2ca895525a1964425d1dfe74001a33e3b59b18bf902539cfc4d941dd96c33863&scene=21#wechat_redirect)
    
*   [网络安全] 二十二. Powershell 基础入门及常见用法 - 1
    

2020 年 8 月 18 新开的 “娜璋 AI 安全之家”，主要围绕 Python 大数据分析、网络空间安全、人工智能、Web 渗透及攻防技术进行讲解，同时分享 CCF、SCI、南核北核论文的算法实现。娜璋之家会更加系统，并重构作者的所有文章，从零讲解 Python 和安全，写了近十年文章，真心想把自己所学所感所做分享出来，还请各位多多指教，真诚邀请您的关注！谢谢。2021 年继续加油！

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZePZ27y7oibNu4BGibRAq4HydK4JWeQXtQMKibpFEkxNKClkDoicWRC06FHBp99ePyoKPGkOdPDezhg/640?wx_fmt=png)

(By:Eastmount 2021-03-14 周日夜于武汉)

**参考文献：**

*   https://www.bilibili.com/video/av66327436 [推荐 B 站老师视频]
    
*   《安全之路 Web 渗透技术及实战案例解析》陈小兵老师
    
*   https://baike.baidu.com/item/Windows Power Shell/693789
    
*   https://www.pstips.net/powershell-piping-and-routing.html
    
*   https://www.pstips.net/using-the-powershell-pipeline.html