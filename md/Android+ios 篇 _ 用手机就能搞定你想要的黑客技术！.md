\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/v8tzVx8J1G8b\_-v3jDHiFA)

渗透测试人员都习惯一台笔记本走天下，但有的时候笔记本还是太大，一些地方用笔记本做渗透测试还是太招摇，而且有的时候也不会随身都带笔记本。这时如果可以利用随身携带的手机进行渗透，想想都很酷。众所周知，手机版的 kali 就是 Kali NetHunter，但这神器一是要刷机，二是适配的手机非常少，三是即使刷成功了，那你手机上原来的各种软件就不那么好用了。今天跟大家分享一下如何在手机（Android&IOS）上不刷机、免 root 安装 nmap、sqlmap、msf 等工具，将手机改造成移动渗透利器。

Android 篇
---------

### 0x01 安装 Termux

Termux 是一款开源且不需要 root，运行在 Android 终端上极其强大的 linux 模拟器，支持 apt 管理软件包，完美支持 python,ruby,go,nodejs。termux 下载: https://github.com/termux/termux-app   termux 官网：https://termux.com/   安装第一次打开会显示下图：

![](https://mmbiz.qpic.cn/mmbiz_png/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TN06ziadqBc9ibtn4Ok3pCoAvsYIicRXvjw1HnzXUibdRNdlsbkMxvv4LQw/640?wx_fmt=png)

注意，安装完成后要进行权限设置，Termux 只有一个存储权限，记得打开，否则 Termux 会一直如上图一样旋转；

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TibTetJttBmSBicF54U8ty1HQzNo3CtAylicA5gWfOIwI4ypBKxn8W8PTg/640?wx_fmt=jpeg)

安装完毕，Termux 登场：

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TX81XhzgtrwwmjvIC6jVL7ias686kgAp6Y5EsS1RWicqNGIqBeFChOOSg/640?wx_fmt=jpeg)

### 0x02 Termux 基本使用

Termux 界面长按屏幕，显示菜单项（包括返回、复制、粘贴、更多），此时屏幕出现可选择的复制光标。

Termux 界面从左向右滑动，显示隐藏式导航栏，可以新建、切换、重命名会话 session 和调用弹出输入法  

常用快捷键：

```
音量-键(Ctrl)+L                清除屏幕内容
音量-键(Ctrl)+C                终止当前操作
音量-键(Ctrl)D                 退出当前会话session
音量+键+D                      Tab键（可自动补全命令或文件名）
音量+键+W                      方向键 上（可显示前一条命令）
音量+键+S                      方向键 下（可显示后一条命令）
音量+键+A                      方向键 左（可左移动光标）
音量+键+D                      方向键 右（可右移动光标）
音量+键+Q                      显示或关闭扩展键（ESC、插入链接CTR、ALT、TAB、-、/、|以及左滑扩展键一栏可切换到全功能支持手机输入法的输入框）
```

常用命令（和 linux 基本类似）：

```
apt update                    更新源
apt search <query>            全文搜索可安装包
apt install <package>         安装软件包
apt upgrade                   升级软件包
apt show  <package>           显示软件包的信息
apt list \[--installed\]        列出所有（或已安装）的软件包信息
apt remove <package>          删除软件包
chmod                         修改文件权限
chown                         修改文件归属
...
```

### 0x03 打造 Android 渗透神器

1、更新源：

```
apt update && apt upgrade   
cd ..
cd usr/etc/apt
vim sources.list
```

修改源：

```
deb \[arch=all,aarch64\] http://mirrors.tuna.tsinghua.edu.cn/termux stable main
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TQicx3vLmho9yYdzfibuAGMZwUOMohxNJr4ZBDK14ghywDSZyJGBQ0AcQ/640?wx_fmt=jpeg)

```
apt update
```

2、安装 nmap

```
apt install nmap
```

很方便，完成后在手机上出现熟悉界面：

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TnFe7UDbj58q456pKgADgNNkjkE3rSVbBh5J0ResUBDYPXFZ63GV3icA/640?wx_fmt=jpeg)

3、安装 sqlmap

首先安装运行 sqlmap 所需要的包 python2，以及 clone GitHub 包的 git 包

```
apt install python2 git
git clone https://github.com/sqlmapproject/sqlmap
```

时间略长，完成后在手机上出现熟悉界面：

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TY7asTwnRicve1eHSw1xzVvYwfD1yBgzOkCBKM51X7VDt3QVVXGIC20w/640?wx_fmt=jpeg)

4、安装 msf   首先安装 wget：

```
pkg install wget
```

下载 msf 安装脚本：

```
wget https://Auxilus.github.io/metasploit.sh
```

运行安装脚本：

```
sh metasploit.sh
```

这个过程比较慢，大概需要 40 分钟左右，成功后，在手机中出现熟悉的界面：

![](https://mmbiz.qpic.cn/mmbiz_png/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TFuV3PhoNIricCNNQGT4MYmuBhIjo79iatJbFiahGBy0rV63h1MIQOcl9A/640?wx_fmt=png)

IOS 篇
-----

### 0x01  安装 iSH

iSH 是一个使用 usermode x86 模拟器将 Linux shell 引入 IOS 设备的工具，基于 Alpine Linux，该程序占用空间小，具备一定的安全性且易于上手。不过目前 iSH 还处于测试阶段，部分功能还不完善。

iSH github 地址：https://github.com/tbodt/ish

由于目前 iSH 还是 beta 版，所以想要在 IOS 设备上安装 iSH，首先需要安装 APP TestFlight，它可以帮助开发人员测试 Beta 版 App。TestFlight 运行环境要求：iOS 8 或更高版本的 iPhone、iPad 或 iPod touch。

![](https://mmbiz.qpic.cn/mmbiz_png/msM7PibnMwSCxquqqE9Duo58TkCRgdH9THO7B6z37ID2O7o6Pvz7LeXfhubWJkuibibUwPM1EnTOvbxzWvfsQnEKg/640?wx_fmt=png)

安装 TestFlight 后，打开链接：https://testflight.apple.com/join/97i7KM8O ，然后点击 “开始测试”，如图所示，就可以打开 TestFlight 并收到加入 iSH 测试版的邀请了。

![](https://mmbiz.qpic.cn/mmbiz_png/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TiaLnSA3NsNbicUP05jFaaReEyptUldrdg1yiaFhb4mcOiaib1UbfnATGIjQ/640?wx_fmt=png)

安装 iSH 完毕后，出现 iSH 界面：

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TdMYzUf3E7pfjwgKg5icGo4oYuW6O8xdmKf2pJiaHIIt5PNB4nwwEEqlA/640?wx_fmt=jpeg)

### 0x02 iSH 基本使用

iSH 自带了多功能键盘：

![](https://mmbiz.qpic.cn/mmbiz_png/msM7PibnMwSCxquqqE9Duo58TkCRgdH9Tj3mss7hLxibTS8vYoE3RPJKNabSpibHCcvF3ML4woicwVKag1PBD52j8A/640?wx_fmt=png)

上图中的四个图标分为是：TAB 键、Shift 键、ESC 键以及可以滑动的方向键，结合手机的键盘，基本可以满足 shell 的一些操作。

常用命令：

```
apk update                    更新源
apk search <query>            全文搜索可安装包
apk add <package>             安装软件包
apk upgrade                   升级软件包
apk list \[--installed\]        列出所有（或已安装）的软件包信息
apk del <package>             删除软件包
chmod                         修改文件权限
chown                         修改文件归属
...
```

### 0x03 打造 iOS 渗透神器

1、更新源：

```
apk update
apk upgrade
```

2、安装 nmap

```
apk add nmap
```

很方便，完成后在 iphone 手机上出现熟悉界面：

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TEdJtyOHVULG8xSKI6W9U39hFf9Tko6Xvicv9bTvG2vVIdUaSrKaWzXA/640?wx_fmt=jpeg)

需要注意的是，在安装过程中，iphone 或者 ipad 不能锁屏，需要在设置 -> 显示与亮度 -> 自动锁定 设置为为永不锁定，否则会安装失败报错。

![](https://mmbiz.qpic.cn/mmbiz_jpg/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TFibLicfGvqI13a4ZDzywp8lAoQYLhg8EkFNQl7GJu20U08sc9MkduYpA/640?wx_fmt=jpeg)

3、安装 sqlmap

首先安装运行 sqlmap 所需要的包 python2，以及 clone GitHub 包的 git 包

```
apk add python2 git
git clone https://github.com/sqlmapproject/sqlmap
```

时间略长，完成后在 ipad 上出现熟悉界面：

![](https://mmbiz.qpic.cn/mmbiz_png/msM7PibnMwSCxquqqE9Duo58TkCRgdH9TpqxPPoibx2pEOjsgNE08xIWuyFntdocibdhhkwmMhXD0LvfibG8SVJVwg/640?wx_fmt=png)

其他
--

如果对手机的键盘不太适应，可以搭配购买便携式的蓝牙键盘，操作起来更加顺手，携带也很方便，可以说是一机在手，天下我有~

![](https://mmbiz.qpic.cn/mmbiz_jpg/83e7tQTo0wM6SgnYKEicwwCndwJt4qpAmIJAYUOiadv1hoBLckjCp3ianmaKsIGt3y3pTcnrvvLa2lpO8jvatvGbA/640?wx_fmt=jpeg)

说明，本教程文章仅限用于学习和研究目的，请勿用于非法用途。

文章来自 https://cloud.tencent.com/developer/article/1541004

### **声明：****本人分享该教程是希望大家，通过这个教程了解信息安全并提高警惕！本教程仅限于教学使用，不得用于其他用途触犯法律，本人一概不负责，请知悉！**

### **免责声明：本文旨在传递更多市场信息，不构成任何投资建议和其他非法用途。文章仅代表作者观点，****以上文章之对于正确的用途，仅适用于学习。**

![](https://mmbiz.qpic.cn/mmbiz_png/83e7tQTo0wO7OjZo110ia3ial18mgQbCwFzxQicmSFicxeBRicakYQAHJ4QsBtT4j6RuZkWyKS1o6U2ufeg6k7ia6quQ/640?wx_fmt=png)****![](https://mmbiz.qpic.cn/mmbiz_png/83e7tQTo0wPqZBQGoIee5SUPvSghllp2hRZ6cX9rViaT6ibibciaiamMicNoVsmAqgwhbQ2vvvq7eSGxTlX0g6qEFVaw/640?wx_fmt=png)****

↑↑↑**长按**图片**识别二维码**关註↑↑↑