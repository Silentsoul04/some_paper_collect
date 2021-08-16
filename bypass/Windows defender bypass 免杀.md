> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iP9Ykg1zYRjqDQhwWTdifA)

当前浏览器不支持播放音乐或语音，请在微信或其他浏览器中播放 想起了你 程响 - 想起了你 ![](http://res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/icon/appmsg/qqmusic/icon_qqmusic_source55871f.svg)   ![](https://y.gtimg.cn/music/photo_new/T002R90x90M000001hRg6L2eWevf.jpg)

官方文档
----

在制作免杀的过程中，翻找 Windows 官方对 Windows Defender 的介绍，发现有这样一个目录：Configure Microsoft Defender Antivirus exclusions on Windows Server（在 Windows server 中配置 defender 排除项）。

> https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/configure-server-exclusions-microsoft-defender-antivirus?view=o365-worldwide#list-of-automatic-exclusions

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgReNDxa5JWjzODTSsuFS5Ark7KcrkLLF9hpHlKCg7o4icm6bEObrLxic7F0JMoslvGByjwb2IgkJGLtA/640?wx_fmt=png)

简而言之就是在 Windows Server2016 和 2019 中，Windows Defender 默认存在一些排除项，在实时检测过程中会忽略这些排除项，但是主动扫描的时候不会排除。这就给 Bypass Windows Defender 提供了一个新思路。

通篇寻找可用的路径，最终发现几个 exe 路径：

<table><thead><tr><th>路径</th><th>用途</th></tr></thead><tbody><tr><td>%systemroot%\System32\dfsr.exe</td><td>文件复制服务</td></tr><tr><td>%systemroot%\System32\dfsrs.exe</td><td>文件复制服务</td></tr><tr><td>%systemroot%\System32\Vmms.exe</td><td>Hyper-V 虚拟机管理</td></tr><tr><td>%systemroot%\System32\Vmwp.exe</td><td>Hyper-V 虚拟机管理</td></tr><tr><td>%systemroot%\System32\ntfrs.exe</td><td>AD DS 相关支持</td></tr><tr><td>%systemroot%\System32\lsass.exe</td><td>AD DS 相关支持</td></tr><tr><td>%systemroot%\System32\dns.exe</td><td>DNS 服务</td></tr><tr><td>%SystemRoot%\system32\inetsrv\w3wp.exe</td><td>WEB 服务</td></tr><tr><td>%SystemRoot%\SysWOW64\inetsrv\w3wp.exe</td><td>WEB 服务</td></tr><tr><td>%SystemDrive%\PHP5433\php-cgi.exe</td><td>php-cgi 服务</td></tr></tbody></table>

在文件路径不冲突的情况下，将这 10 个路径的木马应当都具有 bypass Windows Defender 的效果。

实例
--

以最后一个 php-cgi.exe 为例，默认在 Windows Server 2019 中是没有此路径的，所以在实际使用过程中需新建此目录。

首先使用 msf 生成一个默认的 exe 木马，并下载到目标服务器中执行，发现 Windows Defender 发出警告：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgReNDxa5JWjzODTSsuFS5ArkytFwxtrZpMNuconAFWHXgjxwBYrVulEhz8HbhnB25bSGwBPOae0JlA/640?wx_fmt=png)

获得的 session 也是昙花一现：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgReNDxa5JWjzODTSsuFS5ArkVro8cIqbCY1DDC9ibsbVQzmibUXspthndOQyGI1fwUejfVvAYLicBIThg/640?wx_fmt=png)

新建 php5433 目录，并将木马更名为 php-cgi.exe，执行：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgReNDxa5JWjzODTSsuFS5ArkQHcLzzOowZN2OuH0t8nzNjmO550VoUfuth8ibz3icO6hf8DkrovrAdGg/640?wx_fmt=png)

木马正常上线：

![](https://mmbiz.qpic.cn/mmbiz_png/fZT30hrVgReNDxa5JWjzODTSsuFS5ArkKicoEMcnquZNXOTbwdJtDMPo0ZIot6jPJEuAfVqL83Sqszib3vG2QGpA/640?wx_fmt=png)

有态度，不苟同  

![](https://mmbiz.qpic.cn/mmbiz/yqVAqoZvDibEmUmjw1g1ln7tK24z5BNE2zPOdcP7L8fSb1Gib86QD302AjpzM3taqlCib3icuScdPzP6GOTjf5eWeA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPwwb5AxgeyO4QBNh18Fn6zdHoLUI5icibB4ibJKHvDsZTm7oBibUMPBk2ccibiawFdUyRsxdwsHjdAVjYuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPwFnSZ4ST9beGd5aICibCzeudnBgkU2jxkNicmkoJOqCRpRTuZ66zKQRXahaCXcwyxugx5paBygA1aw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPx2Cg8E6NUnTM2QWsLDFu3aJ7MIzUCkaMnjNPMaIS6ZibIjxiaXVUGicnfQBohqjtxicSMt55NMBfpTWQ/640?wx_fmt=png)
