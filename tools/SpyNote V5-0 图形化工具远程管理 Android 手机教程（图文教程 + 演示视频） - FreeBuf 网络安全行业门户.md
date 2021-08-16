> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.freebuf.com](https://www.freebuf.com/sectool/164077.html)

*** 本文原创作者：艾登——皮尔斯，本文属 FreeBuf 原创奖励计划，未经许可禁止转载**  

前言
--

**本篇文章主要以图文教程和视频演示详细地教你如何快速学会使用 SpyNote 5.0 图形化工具来穿透内网远程控制 Android 手机。本教程有一定的攻击性，请各位 Freebuf 小粉合理使用。(切勿用于违法，否则因用于违法产生的一切法律纠纷，由使用者自行承担，与本文作者无关)**

![](https://image.3001.net/images/20180304/15200935222671.png!small)

Freebuf 之前相关 SpyNote 的介绍文章

> [小心，Android 木马工具 SpyNote 免费啦！远程监听就是这么简单](http://www.freebuf.com/news/110776.html)  
> 
> [当心，安卓远控（spynote）升级了……](http://www.freebuf.com/news/110776.html)

**所需环境:**
---------

> Windows 7/8/10 系统
> 
> Java 环境
> 
> Microsoft .NET Framework 4.0 框架
> 
> SpyNote 5.0 （[百度云传送门提取码 nhq6](https://pan.baidu.com/s/1BkMm-iEs__f7wZALMOZztg), [官方下载地址](http://www.iq-team.org/spy-note/))
> 
> 网络通 (用于端口映射到公网来达到穿透内网[传送门](http://www.youtusoft.com/))

**教程开始:**  

------------

1. 下载并且安装 Java[](https://www.java.com/zh_CN/)，已经安装了 Java 可以跳过

![](https://image.3001.net/images/20180304/15200939462033.png!small)  

2. 如果你的计算机没有安装 Microsoft .NET Framework 4.0 框架，请去百度搜索下载安装，已经安装了 Microsoft .NET Framework 4.0 可以跳过。  

![](https://image.3001.net/images/20180304/1520094006786.png!small)  

3. 下载安装网络通给接下来的内网穿透做铺垫  

![](https://image.3001.net/images/20180304/15200940816393.png!small)  

下载进行安装，如果你没有网络通的帐号就去注册一个帐号，并且登录。

![](https://image.3001.net/images/20180304/15200940953023.png!small)  

然后打开 CMD 输入 ipconfig, 我的内网 IP 地址是: 192.168.1.152  

![](https://image.3001.net/images/20180304/15200941052071.png!small)  

然后点击添加映射, 免费用户线路就选择美国 1，名称你可以随便输入，内网 IP 就输入自己的内网 IP 地址，内网端口可以任意，但是不能冲突。  

![](https://image.3001.net/images/20180304/15200941182056.png!small)  

点击确定后，鼠标移到线路，右击选择复制外网地址  

![](https://image.3001.net/images/20180304/15200941285574.png!small)  

由上图可以得知，我的外网端口是 29035  

4. 打开 SpyNote5.0 点击 Listen Port(监听端口)  

![](https://image.3001.net/images/20180304/1520094164414.png!small)  

输入 2222 添加监听端口 --->Add, 再输入 29035 添加上线地址的端口 --->Add，再点击 OK  

![](https://image.3001.net/images/20180304/15200941838867.png!small)  

当如下图所示时，控制端就已经处于监听状态。  

![](https://image.3001.net/images/20180304/15200942433507.png!small)  

5. 点击 BuildClient 生成木马。  

(1)Client Info 设置木马的图标，版本号，名称：  

![](https://image.3001.net/images/20180304/15200942745615.png!small)  

(2)Dynamic DNS 设置上线地址：

![](https://image.3001.net/images/20180304/15200942909320.png!small)  

(3)Properties 设置对生成的木马进行一些特殊设置  

![](https://image.3001.net/images/20180304/15200943915101.png!small) 

> Hide application 是隐藏应用程序
> 
> Wi-fi Wakelock 是锁定 WIF
> 
> CPU WakelockI 是常驻后台
> 
> Permission Root SuperSu 是主动申请 Root 权限
> 
> Device Administration 是请求激活设备管理器 (防普通方式卸载)
> 
> Accessibility(Keylogger) 是用于键盘记录
> 
> Set a Repeating Alarm 是设置报警 (应该是上线提示)

这里我就全选  

(4)Merging App(捆绑 APK)，就是和其他 apk 安装包进行捆绑

![](https://image.3001.net/images/20180304/15200944851208.png!small)  

因为捆绑有几率会因为被捆绑文件加固而捆绑失败，所以这里我就不捆绑。

(5) 设置好一切后点击左上角的 Build--->Build APK 创建木马, 会弹出一个这个  

![](https://image.3001.net/images/20180304/15200946165776.png!small)  

(6) 我们就找到你 SpyNote5.0 的所在路径的根目录有个 Patch 的文件夹  

![](https://image.3001.net/images/20180304/1520094638552.png!small)  

(7) 点击进入选择 Patch-StaminaMode-release，然后点击打开  

![](https://image.3001.net/images/20180304/15200946504028.png!small)  

(8) 你点击打开之后下面就会出现一个绿色的进度条  

![](https://image.3001.net/images/20180304/15200946597939.png!small)  

(9) 生成完毕会自动打开一个文件夹里面有一个名字为 client 的 apk 文件就是你刚刚生成的木马。  

![](https://image.3001.net/images/20180304/15200946716679.png!small)  

(10)现在我用我的手机 (魅蓝 5) 安装刚刚生成木马，然后等待上线，安装打开后控制端这边就已经提示上线了，接下来就可以对这个手机进行一个远程的控制了。

![](https://image.3001.net/images/20180304/15200947299988.png!small)  

**主要功能:**
---------

> File Manager：读取手机文件
> 
> SMS Manager：读取手机 SMS 短信
> 
> Calls Manager：读取手机通讯录
> 
> Contacts Manager：读取联系人
> 
> Location Manager：读取 GPS 位置
> 
> Account Manager：不详
> 
> Camera Manager：利用摄像头进行拍照
> 
> Audio Recorder：录音
> 
> Shell Terminal：不详
> 
> Applications：应用管理
> 
> Keylogger：键盘记录
> 
> Settings：设置
> 
> Phone：手机管理
> 
> Client：客户端管理
> 
> Chat：进行文本对话

下面是 SpyNote5.0 生成教程和演示视频:

**防范方法：**
---------

> (1) 安装个手机杀毒软件，定期更新病毒库，定期杀毒
> 
> (2) 提高自身的安全意识
> 
> (3) 不下载安装第三方来历不明的文件，下载应用最好去应用商店下载
> 
> (4) 对于不必要的权限建议你禁止其 APP 获取权限

**小结：**
-------

这款工具因为其功能强大而闻名，且上线是比 Metasploit 生成的 payload 上线要稳定很多，还会自动弹出激活设备管理器窗口，来提升自身的权限，以防被普通卸载方式卸载，但是没有提及如何地详细使用，所以鄙人就弄了一个详细的图文 + 视频的教学献给忠实的 Freebuf 小粉，就当作是元宵节鄙人送给你们的礼物吧！希望 Freebuf 小粉们喜欢！

![](https://image.3001.net/images/20180304/15200975192932.png!small)

*** 本文原创作者：艾登——皮尔斯，本文属 FreeBuf 原创奖励计划，未经许可禁止转载**