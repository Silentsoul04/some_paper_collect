> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cWiW_6OjlMQV7TM0uK1tHg)

来源：https://blog.csdn.net/nmask

我很早之前就想开发一款 app 玩玩，无奈对 java 不够熟悉，之前也没有开发 app 的经验，因此一直耽搁了。最近想到尝试用 python 开发一款 app，google 搜索了一番后，发现确实有路可寻，目前也有了一些相对成熟的模块，于是便开始了动手实战，过程中发现这其中有很多坑，好在最终依靠 google 解决了，因此小记一番。  

### 准备工作

利用 python 开发 app 需要用到 python 的一个模块–kivy，kivy 是一个开源的，跨平台的 Python 开发框架，用于开发使用创新的应用程序。简而言之，这是一个 python 桌面程序开发框架（类似 wxpython 等模块），强大的是 kivy 支持 linux、mac、windows、android、ios 平台，这也是为什么开发 app 需要用到这个模块。  

虽然 kivy 是跨平台的，但是想要在不同的平台使用 python 代码，还需要将 python 代码打包成对应平台的可执行程序，好在 kivy 项目下有个打包工具项目–buildozer，这是官方推荐的打包工具，因为相对比较简单，自动化程度高，其他项目比如：python-for-android 也能起到类似的作用，这里不展开介绍。

### 搭建 kivy 开发环境

需要在 pc 上安装 kivy 开发环境，这里演示下 mac 与 linux 下的安装过程。

#### install kivy for mac

安装一些依赖包：

```
brew install pkg-config sdl2 sdl2_image sdl2_ttf sdl2_mixer gstreamer
```

安装 cython 以及 kivy：

```
pip install cython==0.25
pip install kivy
```

如果安装 kivy 报错，则使用下面的方式安装 kivy：

```
git clone https://github.com/kivy/kivy
python setup.py install
```

安装后测试：

```
$python
Python 2.7.10 (default, Jul 15 2017, 17:16:57)
[GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.31)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>> import kivy
[INFO   ] [Logger      ] Record log in /Users/didi/.kivy/logs/kivy_18-05-08_4.txt
[INFO   ] [Kivy        ] v1.10.1.dev0, git-5f6c66e, 20180507
[INFO   ] [Python      ] v2.7.10 (default, Jul 15 2017, 17:16:57)
[GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.31)]
```

说明：导入 kivy 模块没有报错则说明安装成功。

#### install kivy for centos7

先安装依赖：

```
yum install \
make \
mercurial \
automake \
gcc \
gcc-c++ \
SDL_ttf-devel \
SDL_mixer-devel \
khrplatform-devel \
mesa-libGLES \
mesa-libGLES-devel \
gstreamer-plugins-good \
gstreamer \
gstreamer-python \
mtdev-devel \
python-devel \
python-pip \
java-devel
```

安装 cython 以及 kivy:

```
pip install Cython==0.20
pip install kivy
```

centos 安装 kivy 参考：https://kivy.org/docs/installation/installation-linux.html#using-software-packages

说明：其他安装 kivy 方式可移步：https://kivy.org/#download（需要某墙）

### 用 kivy 开发第一个 python app

安装完 kivy 就可以开发 app 程序了，这里演示下 hello-world 程序，关于 kivy 更复杂的用法不是本文重点，后面再成文介绍。  

1) 创建一个 main.py 文件，写入：

```
#! -*- coding:utf-8 -*-
from kivy.app import App
class HelloApp(App):
pass
if __name__ == '__main__':
HelloApp().run()
```

2) 创建一个 hello.kv 文件，写入：

```
Label:
text: 'Hello, World! I am nMask'
```

简单说明：main.py 是入口函数，定义了一个 HelloApp 类，该类继承 kivy.app；hello.kv 文件是 kivy 程序，相当于定义界面风格等，该文件命名规则为类名小写且去除 app。

### 运行第一个 python app

```
python main.py
```

运行结果：  
![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMmRwDzr7lsJVCicmjViaiafMR6DEy3W25hUbibj8NzJGhewVoWf5sgBrcnTwJLRxuZVmQUPgkJzo3pzSg/640?wx_fmt=jpeg)

### 安装 buildozer 工具

通过以上的编码，我创建了自己的第一个 python app 程序，该程序可以直接在 mac、linux、windows 平台下运行，那么如何让它在安卓或者苹果手机上运行呢？我们知道在安卓上运行，需要将其打包成 apk 安装程序，因此就需要用到前面提到过的 buildozer 工具，（buildozer 工具可以打包 kivy 程序，支持 android、ios 等），buildozer 的安装过程比较简单：

```
pip install buildozer
```

### 使用 buildozer 工具将 kivy 程序打包成 apk

在 python 项目目录下运行：

```
buildozer init
```

运行成功将会创建一个配置文件 buildozer.spec，可以通过修改配置文件更改 app 的名称等，然后运行：

```
buildozer android debug deploy run
```

运行以上命令将会生成跨平台的安装包，可适用安卓、ios 等，如果用于安卓，则是利用 python-for-android 项目。

在第一次运行以上命令的时候，会自动在系统中下载安卓 sdk 等必要文件，如下图。（过程需要翻墙，而且有很多依赖需要下载）  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMmRwDzr7lsJVCicmjViaiafMR6YgLDmWT7c6icy7ThrNhYx64ZZGxxnk0dd8VQmicq7dmwA4Jb0EgibUzvw/640?wx_fmt=jpeg)

说明：这里只演示打包成 apk 文件，iso 平台的可自行研究，参考文档：https://github.com/kivy/buildozer。

### python apk 程序测试

如果以上步骤都运行成功的话，应该会在项目目录下的 bin 目录下生成一个 apk 文件，类似如下：  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMmRwDzr7lsJVCicmjViaiafMR6X8nibibsLmawO4kDRbVlHPeARjLy5tWfCscf2wicAAIC9pxRGs7md0lQQ/640?wx_fmt=jpeg)

然后将 apk 下载到安卓系统的手机上，安装即可，测试效果如下：  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMmRwDzr7lsJVCicmjViaiafMR6ZfBxicNFngqcQUCkgafaLmRe7ibcvEaFFdtlHJ5yw3Haz6JDlo8gsMfw/640?wx_fmt=jpeg)  

打开 app  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMmRwDzr7lsJVCicmjViaiafMR67XWbcKFmKrqL9QkPh7lDDFjClBH5G21B2JKBSvmBklrxKhqKO7v73g/640?wx_fmt=jpeg)

### buildozer 使用说明

```
Usage:
buildozer [--profile <name>] [--verbose] [target] <command>...
buildozer --version
Available targets:
android        Android target, based on python-for-android project
ios            iOS target, based on kivy-ios project
android_old    Android target, based on python-for-android project (old toolchain)
Global commands (without target):
distclean          Clean the whole Buildozer environment.
help               Show the Buildozer help.
init               Create a initial buildozer.spec in the current directory
serve              Serve the bin directory via SimpleHTTPServer
setdefault         Set the default command to run when no arguments are given
version            Show the Buildozer version
Target commands:
clean      Clean the target environment
update     Update the target dependencies
debug      Build the application in debug mode
release    Build the application in release mode
deploy     Deploy the application on the device
run        Run the application on the device
serve      Serve the bin directory via SimpleHTTPServer
Target "android_old" commands:
adb                Run adb from the Android SDK. Args must come after --, or
use --alias to make an alias
logcat             Show the log from the device
Target "ios" commands:
list_identities    List the available identities to use for signing.
xcode              Open the xcode project.
Target "android" commands:
adb                Run adb from the Android SDK. Args must come after --, or
use --alias to make an alias
logcat             Show the log from the device
p4a                Run p4a commands. Args must come after --, or use --alias
to make an alias
```

### buildozer 打包过程中的坑点

如果在打包过程中遇到报错，可以修改 buildozer.spec 配置文件中的 log_level 为 2，然后重新运行，可以看具体的错误信息。

#### 报错：You might have missed to install 32bits libs

这个错是我在 centos7 上运行时报的错，大意是系统缺少了某些 32 位的依赖文件。  
解决方案：

```
yum -y install --skip-broken glibc.i686 arts.i686 audiofile.i686 bzip2-libs.i686 cairo.i686 cyrus-sasl-lib.i686 dbus-libs.i686 directfb.i686 esound-libs.i686 fltk.i686 freeglut.i686 gtk2.i686 hal-libs.i686 imlib.i686 lcms-libs.i686 lesstif.i686 libacl.i686 libao.i686 libattr.i686 libcap.i686 libdrm.i686 libexif.i686 libgnomecanvas.i686 libICE.i686 libieee1284.i686 libsigc++20.i686 libSM.i686 libtool-ltdl.i686 libusb.i686 libwmf.i686 libwmf-lite.i686 libX11.i686 libXau.i686 libXaw.i686 libXcomposite.i686 libXdamage.i686 libXdmcp.i686 libXext.i686 libXfixes.i686 libxkbfile.i686 libxml2.i686 libXmu.i686 libXp.i686 libXpm.i686 libXScrnSaver.i686 libxslt.i686 libXt.i686 libXtst.i686 libXv.i686 libXxf86vm.i686 lzo.i686 mesa-libGL.i686 mesa-libGLU.i686 nas-libs.i686 nss_ldap.i686 cdk.i686 openldap.i686 pam.i686 popt.i686 pulseaudio-libs.i686 sane-backends-libs-gphoto2.i686 sane-backends-libs.i686 SDL.i686 svgalib.i686 unixODBC.i686 zlib.i686 compat-expat1.i686 compat-libstdc++-33.i686 openal-soft.i686 alsa-oss-libs.i686 redhat-lsb.i686 alsa-plugins-pulseaudio.i686 alsa-plugins-oss.i686 alsa-lib.i686 nspluginwrapper.i686 libXv.i686 libXScrnSaver.i686 qt.i686 qt-x11.i686 pulseaudio-libs.i686 pulseaudio-libs-glib2.i686 alsa-plugins-pulseaudio.i686 python-matplotli
```

参考：https://ask.fedoraproject.org/en/question/9556/how-do-i-install-32bit-libraries-on-a-64-bit-fedora/

#### 报错：Error compiling Cython file

错误大意为 cython 文件出错，可能是 cython 模块没有安装，或者版本有问题。  
解决方案：

```
pip install cython==0.25
```

#### 报错：IOError: [Errno 2] No such file or directory…..

这是在打包的最后一步，将 apk 文件 copy 到项目 bin 目录下时报的错，是 buildozer 的一个 bug。  

解决方案：  

修改 / usr/local/lib/python2.7/dist-packages/buildozer/tagets/android.py 文件：  

(1) 在文件开头导入:

```
from distutils.version import LooseVersion
```

(2) 将 786 行: XXX found how the apk name is really built from the title 这一行以下的代码替换为：

```
__sdk_dir = self.android_sdk_dir
build_tools_versions = os.listdir(join(__sdk_dir, 'build-tools'))
build_tools_versions = sorted(build_tools_versions, key=LooseVersion)
build_tools_version = build_tools_versions[-1]
gradle_files = ["build.gradle", "gradle", "gradlew"]
is_gradle_build = any((exists(join(dist_dir, x)) for x in gradle_files)) and build_tools_version >= ’25.0'
```

### buildozer 虚拟机

kivy 官方推出了一个 buildozer 虚拟机镜像，已经安装好了 buildozer 以及一些依赖文件，为 buildozer 打包测试提供平台。由于之前我在 mac 上利用 buildozer 打包一直报错，后来换成 centos 也依然没有成功，因此便下载了此虚拟机，测试效果如下：  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rO1ibUkmNGMmRwDzr7lsJVCicmjViaiafMR6SnodMH8Im0iatJ2v3avzqOdTjiboEam48QaCAdz63STcVnA0ZxNgRtyQ/640?wx_fmt=jpeg)

虚拟机下载地址：http://txzone.net/files/torrents/kivy-buildozer-vm-2.0.zip

说明：对于无法解决依赖问题的朋友，可以使用此虚拟机进行程序打包，开发环境还是推荐用自己的本机。

### kivy 开发实例

因为本文重点在于介绍如何利用 kivy+buildozer 开发一款 python app，因此对于 kivy 的开发过程，以及 app 功能进行了最简化。想要学习如何开发更复杂的 app，可参考：https://muxuezi.github.io/posts/kivy-perface.html#

**2021 年，字节跳动扩招一万 Java 程序员！戳原文**
--------------------------------