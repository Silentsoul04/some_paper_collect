> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/UcN271EOlhOLzXELLH7k7g)

 **可禁用 Windows Defender，防火墙，智能屏幕并执行有效负载**

### 用法 

1.  在此行上编辑 Defeat-Defender.bat https://github.com/swagkarna/Defeat-Defender/blob/93823acffa270fa707970c0e0121190dbc3eae89/Defeat-Defender.bat#L72 并替换有效负载的直接网址
    
2.  运行脚本 “run.vbs”。它将要求管理员权限。如果授予了权限，该脚本将在没有控制台窗口的情况下静默运行。
    

获得管理员权限后，它将禁用防御者
----------------

1.  PUA 保护
    
2.  自动送样
    
3.  Windows 防火墙
    
4.  Windows 智能屏幕（永久）
    
5.  禁用快速扫描
    
6.  将 exe 文件添加到防御者设置中的排除项
    
7.  禁用勒索软件保护
    

病毒总结果 [8/04/2021]
-----------------

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2WV7ncSMowoDmlbkaEOpiaFVbzdLS9TAObQLtBFJG8DseZLYlgC0lQe8hKLSfmoh6PXzKYJBYBNLQ/640?wx_fmt=png)

绕过 Windows Defender 技术：
-----------------------

        最近，Windows 引入了称为 “防篡改” 的新功能。该功能可防止禁用实时保护并使用 Powershell 或 cmd 修改防御者注册表项... 如果需要禁用实时保护，则需要手动执行...。但是我们将使用 NSudo 禁用实时保护，而不会触发 Windows Defender

运行 Defeat-Defender 脚本后
----------------------

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2WV7ncSMowoDmlbkaEOpiaF23dmGMNruibo4eyIwj2Z95icDZZKtvFtxbnaQvodaPjIjVPDYW6LgSrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2WV7ncSMowoDmlbkaEOpiaFMQv7FuOL9JrPdyxVkicicqE9icwHG5jz6QIz60kbibq29n0XEQoysyDavw/640?wx_fmt=png)

        执行 Batch 文件时要求管理员权限、获得管理员特权后，它开始禁用 Windows Defender 实时保护，防火墙，智能屏幕并开始从服务器下载我们的后门，并将它放置在启动文件夹中。已从服务器下载.. 并且将在系统启动时启动。

项目地址：

https://github.com/swagkarna/Defeat-Defender