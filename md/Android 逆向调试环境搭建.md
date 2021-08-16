> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/L9EKDdjVkOOq8fSag93VZA)

前言
--

市面的 Android 模拟器大多为游戏而定制，几乎没有使用原生 AOSP 并提供 root 和拓展性 (可安装 xposed 之类的)。于是尝试使用 Android Studio 自带的模拟器：Android Studio Virtual Device (AVD)

环境
--

Android Studio 4.1.2

Android 7.1.1 x86 (API 25)

macOS Big Sur 11.4 x86_64

搭建
--

### 创建虚拟机

在选择设备型号的时候，选不带 “Play Store” 的即可，之后选择镜像，根据自己的架构选择 x86 或 arm，这里选择的是 x86 Images

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFC5RLmQuTuI0L9R6YhwIZ9YFNgj8rS1dy22VQh3N3vYWSdzhOaCq6kUyXlT0aHH67Hviajm5tFZqvw/640?wx_fmt=png)images

值得一提的是，Android8.0 以上 (不含 8.0) 可能无法安装 SuperSU 来进行 root，可能需要 Magisk，而 xposed 对 x86_64 的系统支持不是特别好几乎都不支持，而带有 Google APIs 的镜像并不提供 root 权限，所以这里选择了 Android 7.1.1 x86，理论上也适用于 Android 8.0 x86。

### 获得 ROOT 权限

选择并下载好镜像后就可以进入系统了，进入系统并不能从 Android Studio 进入，我们需要用命令进入并附加`-writable-system`参数：

```
emulator -avd 模拟器名 -writable-system
```

接下来我们需要获取 root 权限:

系统自带的 root 权限只能从 adb 调用，并不能给里面的 app 使用，所以我们利用 adb 中的 root 权限刷入 SuperSU

首先以 root 身份运行 adb：

```
adb rootadb remount
```

之后再进入 adb shell，如无意外应该是获得 root 的 shell, 再执行 setenforce 0：

```
adb shellgeneric_x86:/ # setenforce 0
```

之后我们将 SuperSU 刷机包下 x86 目录里面的 su.pie 文件 push 到 / system/bin 和 system/xbin 目录下：

```
adb push su.pie /system/bin/suadb push su.pie /system/xbin/su
```

之后进入 adb shell(root)，将 su 文件设成可执行并安装 su：

```
chmod 0755 /system/bin/suchmod 0755 /system/xbin/susu --installsu --daemon&
```

之后退出 adb shell，安装 SuperSU 程序和 Xposed Installer：

```
adb install supersu.apkadb install xposedinstaller.apk
```

安装完成后运行 SuperSU，点击 New User，之后弹出的对话框询问时候更新二进制，点击取消即可正常使用 SuperSU 了。

### 安装 Xposed

我们已经获取了 Root 权限，接下来需要安装 Xposed。

打开 Xposed Installer，点击安装。这里可能会出现无法获取 zip 之类的错误，科学上网即可。

安装时需要 root 权限，允许即可，安装完成后重启就会发现 xposed 已经安装完成了：

![](https://mmbiz.qpic.cn/mmbiz_png/khmibjLuVibFC5RLmQuTuI0L9R6YhwIZ9YicInFfenib0HDP1nDoEcicVwkp0tmvplEItSAcZhRMyic5MzXRdBCNm2Pg/640?wx_fmt=png)finish

### 一些问题

这个 SuperSU 的 root 权限重启就会消失，消失后需要重新走一遍 root 流程，xposed 则不受影响。

总结
--

于是我们已经顺利搭建好环境了，再安装逆向或脱壳相关的模块就能开展工作了。

EOF