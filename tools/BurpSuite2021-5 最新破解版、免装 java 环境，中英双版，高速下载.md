> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hD0HfqSPgQfDUT4Nb1CcXQ)

双击 Burp Suite Professional 直接启动，jre 使用了程序中的相对路径，不用再安装 java 环境了，所以不用配置 java 环境，直接启动

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM5d4pZnsPlDhcq2NGvdRsfnQ1SOptcJhibA4xKqLPOjes1hxDI1ibeZPsVPOUiaplmldic9J8l9HlrPBg/640?wx_fmt=png)

目录结构  

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM5d4pZnsPlDhcq2NGvdRsfnjgYnLUoL5XAjT7ukHJwhskWkIiaoxeCYdCVU9Qs3QucDPQLaZnySGCA/640?wx_fmt=png)

完美支持中英双版

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM5d4pZnsPlDhcq2NGvdRsfnGPmlruu2fdtb3dTdLmkABtTzM8QIWy03ItbaPMVre4BXXsaoojNYLA/640?wx_fmt=png)

下载地址（已高速）
---------

https://cloud.189.cn/t/IF7JBjjMNB7b

改进  

-----

*   现在，当 Burp 使用 HTTP / 2 成功通信时，它会在请求和响应中提供反馈。您发送到服务器的第一个请求将显示 HTTP / 1。但是，一旦 Burp 确定网站支持 HTTP / 2，所有后续消息将分别在请求行和状态行中表明这一点。有关 Burp 的实验性 HTTP / 2 支持的更多信息，请参考文档。
    
*   实验性的基于浏览器的扫描功能的性能已得到改善。
    
*   嵌入式浏览器已升级到 Chromium 84。
    
*   修复了之前版本的许多 BUG
    

Bug 修复
------

*   `Cookie` 现在，多个 标题已正确显示在 “参数” 选项卡中。
    
*   我们还修复了一个通过我们的漏洞赏金计划报告的安全漏洞。通过大量的用户交互，攻击者可能会从本地文件系统中窃取逗号分隔的文件。攻击者必须诱使用户访问恶意网站，将请求复制为 curl 命令，然后通过命令行执行该请求。
    
*   修复了之前版本许多 BUG
    

BurpSuite2021.5 校验码
-------------------

```
MD5: 16d84e62bdd32c16b0311772e803e2a1
SHA1: 5c63b6a72498e724574d87de9680f10d1745cf5e
SHA256: e7c25ba1c4e0b968fa28d98d6a9484f1e67c4a90f3f7ec89e67e20893c150f0c
CRC32: 2d99e1d5
```

公众号

最后  

-----

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**无害实验室拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**