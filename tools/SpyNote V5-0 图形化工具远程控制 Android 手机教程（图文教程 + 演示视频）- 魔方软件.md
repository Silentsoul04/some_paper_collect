> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.morefunsoft.com](http://www.morefunsoft.com/az/43543.html)

SpyNote V5.0 图形化工具远程控制 Android 手机教程（图文教程 + 演示视频）

2018-03-14 08:30 来源: 黑客与极客

原标题：SpyNote V5.0 图形化工具远程控制 Android 手机教程（图文教程 + 演示视频）

* 本文原创作者：艾登——皮尔斯，本文属 FreeBuf 原创奖励计划，未经许可禁止转载

前言

本篇文章主要以图文教程和视频演示详细地教你如何快速学会使用 SpyNote 5.0 图形化工具来穿透内网远程控制 Android 手机。本教程有一定的攻击性，请各位 Freebuf 小粉合理使用。(切勿用于违法，否则因用于违法产生的一切法律纠纷，由使用者自行承担，与本文作者无关)

![](http://www.morefunsoft.com/uploads/allimg/200112/060302aM_0.jpeg)

小心，Android 木马工具 SpyNote 免费啦！远程监听就是这么简单

当心，安卓远控（spynote）升级了……

小心，Android 木马工具 SpyNote 免费啦！远程监听就是这么简单

当心，安卓远控（spynote）升级了……

Windows 7/8/10 系统

Java 环境

Microsoft .NET Framework 4.0 框架

SpyNote 5.0 （百度云传送门提取码 nhq6, 官方下载地址)

网络通 (用于端口映射到公网来达到穿透内网传送门)

Windows 7/8/10 系统

Java 环境

Microsoft .NET Framework 4.0 框架

SpyNote 5.0 （百度云传送门提取码 nhq6, 官方下载地址)

展开全文

网络通 (用于端口映射到公网来达到穿透内网传送门)

1. 下载并且安装 Java，已经安装了 Java 可以跳过

![](http://www.morefunsoft.com/uploads/allimg/200112/06015X3G_0.jpeg)

2. 如果你的计算机没有安装 Microsoft .NET Framework 4.0 框架，请去百度搜索下载安装，已经安装了 Microsoft .NET Framework 4.0 可以跳过。

![](http://www.morefunsoft.com/uploads/allimg/200112/060214Y96_0.jpeg)

3. 下载安装网络通给接下来的内网穿透做铺垫

![](http://www.morefunsoft.com/uploads/allimg/200112/06023044J_0.jpeg)

下载进行安装，如果你没有网络通的帐号就去注册一个帐号，并且登录。

![](http://www.morefunsoft.com/uploads/allimg/200112/0602462093_0.jpeg)

然后打开 CMD 输入 ipconfig, 我的内网 IP 地址是: 192.168.1.152

![](http://www.morefunsoft.com/uploads/allimg/200112/060142P50_0.jpeg)

然后点击添加映射, 免费用户线路就选择美国 1，名称你可以随便输入，内网 IP 就输入自己的内网 IP 地址，内网端口可以任意，但是不能冲突。

![](http://www.morefunsoft.com/uploads/allimg/200112/06012A548_0.jpeg)

点击确定后，鼠标移到线路，右击选择复制外网地址

![](http://www.morefunsoft.com/uploads/allimg/200112/060110B60_0.jpeg)

由上图可以得知，我的外网端口是 29035

4. 打开 SpyNote5.0 点击 Listen Port(监听端口)

![](http://www.morefunsoft.com/uploads/allimg/200112/0600545T5_0.jpeg)

输入 2222 添加监听端口—>Add, 再输入 29035 添加上线地址的端口—>Add，再点击 OK

![](http://www.morefunsoft.com/uploads/allimg/200112/0600061011_0.jpeg)

当如下图所示时，控制端就已经处于监听状态。

![](http://www.morefunsoft.com/uploads/allimg/200112/0600223a3_0.jpeg)

5. 点击 BuildClient 生成木马。

(1)Client Info 设置木马的图标，版本号，名称：

![](http://www.morefunsoft.com/uploads/allimg/200112/06003TB7_0.jpeg)

(2)Dynamic DNS 设置上线地址：

![](http://www.morefunsoft.com/uploads/allimg/200112/055934E25_0.jpeg)

(3)Properties 设置对生成的木马进行一些特殊设置

![](http://www.morefunsoft.com/uploads/allimg/200112/0559505T8_0.jpeg)

Hide application 是隐藏应用程序

Wi-fi Wakelock 是锁定 WIF

CPU WakelockI 是常驻后台

Permission Root SuperSu 是主动申请 Root 权限

Device Administration 是请求激活设备管理器 (防普通方式卸载)

Accessibility(Keylogger) 是用于键盘记录

Set a Repeating Alarm 是设置报警 (应该是上线提示)

Hide application 是隐藏应用程序

Wi-fi Wakelock 是锁定 WIF

CPU WakelockI 是常驻后台

Permission Root SuperSu 是主动申请 Root 权限

Device Administration 是请求激活设备管理器 (防普通方式卸载)

Accessibility(Keylogger) 是用于键盘记录

Set a Repeating Alarm 是设置报警 (应该是上线提示)

这里我就全选

(4)Merging App(捆绑 APK)，就是和其他 apk 安装包进行捆绑

![](http://www.morefunsoft.com/uploads/allimg/200112/055T63W2_0.jpeg)

因为捆绑有几率会因为被捆绑文件加固而捆绑失败，所以这里我就不捆绑。

(5) 设置好一切后点击左上角的 Build—>Build APK 创建木马, 会弹出一个这个

![](http://www.morefunsoft.com/uploads/allimg/200112/055Z23D8_0.jpeg)

(6) 我们就找到你 SpyNote5.0 的所在路径的根目录有个 Patch 的文件夹

![](http://www.morefunsoft.com/uploads/allimg/200112/05591U062_0.jpeg)

![](http://www.morefunsoft.com/uploads/allimg/200112/055KR935_0.jpeg)

(8) 你点击打开之后下面就会出现一个绿色的进度条

![](http://www.morefunsoft.com/uploads/allimg/200112/055Q45191_0.jpeg)

(9) 生成完毕会自动打开一个文件夹里面有一个名字为 client 的 apk 文件就是你刚刚生成的木马。

![](http://www.morefunsoft.com/uploads/allimg/200112/055S05E5_0.jpeg)