> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LHLQdV4bMgq1ew4wdqGX-w)

![](https://mmbiz.qpic.cn/mmbiz_png/OhKLyqyFoP9mJwX65uY3o0wwuMo2eWPeFuDIhxJlAjMcIicKFSYLVZ6fjicY0dNle24gfmiaVpwCcP2PeZuZyaRzw/640?wx_fmt=png)点击上方蓝字关注我们

![](https://mmbiz.qpic.cn/mmbiz_jpg/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRUfShSwl3q9xPKTKjGCZ29FibxIYvr9bDtUr9NVjVljySy3nz3NoTbJyA/640?wx_fmt=jpeg)

本文主要介绍一种可以在不知道用户账号密码的情况下提升权限的方法。

**Windows 凭据管理器**

Windows 凭据管理器是是一个预安装在 Windows 中的简单密码管理器，可以在其中保存用户和网站的凭据。

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRUCehJ7ABDbvPbJry6qibdSY8KibZvUuovU6mb3LmhPW6Tzx5NrGUyEO7g/640?wx_fmt=png)

在 Windows 凭据管理器中，可以选择添加 “Windows 凭据”、“基于证书的凭据”、“普通凭据” 这三种类型的凭据。相对来说，“Windows 凭据”和 “普通凭据” 使用较多，“基于证书的凭据”几乎很少使用。

现在让我们看看如何保存我们的凭据。

打开搜索栏并输入凭据管理器。

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRU8SEGEe9ePtJVEeiaJ9RGtgNx4KpdPtaTTg4ALO1HTCBy8ib6qnCfxvfw/640?wx_fmt=png)

然后尝试添加一个新凭据。

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRUMwqsBwhK7epNwq74Gzoh6ktE9DGhx5wicVwrhiaYLTuXnKtiaHXNDOwFA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRU5eZdd8TQN2ofiaIBqTnCPcbxwpZ0ZHNTpgJCX8cu4nFpOiawosUibGKpw/640?wx_fmt=png)

我们可以通过运行以下命令来确认凭据已保存。

```
cmdkey /list
```

cmdkey 是凭据管理器命令行实用工具，用于创建、列出和删除存储的用户名和密码或凭据。

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRUtHT1ABxFYnO2sY6dmnlyiaeDrbho16RBhmeJFMBKs6twyAsibRruRG9A/640?wx_fmt=png)

完美，下一步就是下载 **empire** 脚本，链接地址：

https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/dumpCredStore.ps1

该工具使用一组 windows32 API 来检索有关所有已保存凭据的信息。安装完成后，运行以下命令：

```
powershell Import-module <File-Path> ; Enum-Creds
```

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRUib6aN422Y46OnKt4m5O2Ilh1ptAqIEO2biaI3AHWIB0q2Hg70aCeolOQ/640?wx_fmt=png)

注意：有时可能会出现错误，说由于执行策略无法执行 PowerShell 文件。为了避免出现这一错误，可以在 cmd 中输入以下命令。

```
powershell Set-ExecutionPolicy -Scope CurrentUser Unrestricted
```

最后，成功获得用户凭据。

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRUWJ9eY8j1LiaLxMrx6YqPibZk8nCzj2MFoLQhEdusdhjkopLRLsHL5icwQ/640?wx_fmt=png)

现在我们已经知道了用户的登录凭据，接下来需要做的就是运行 runas 命令，从而以该用户身份运行命令。

```
runas /user:<USER> <YOUR_COMMANDS>
```

runas 允许用户用其他权限运行指定的工具和程序，冒充其他用户。

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZqeDS53tp4JytFAxibE7KRU7AmHqtdeLvZQS9EhShaL6dSygQvaxNk2LCib6FKvNChX6Uy06wnsvJw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RQoDdorCu0V5znWFiaMBVWiaibdvAvmGeUvfC5LJ60x1Kq5wiaQ5UtMKEDcwQJ3ibicBdGBKxGs1V2AuZcg3ISoDto1g/640?wx_fmt=png)

  

END

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/DQk5QiaQiciakbdd2suianExpqsibsgicicdZwFxkJcfjEwbj52sZXoSCavNWvQUkIvibRllnaWa1aTtwMORO4R4Bzph9w/640?wx_fmt=jpeg)

好文！必须在看