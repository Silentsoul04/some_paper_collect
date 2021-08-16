> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GI3XNb8Dvidh2JV2Az4kaA)

跟着墨鱼实验室的大佬学习复现

_**参考原文：https://mp.weixin.qq.com/s/ew9SY8SMDvtMptF6k1TKdw**_

本次针对的路由器固件是华为路由器

下载地址

_**链接：https://pan.baidu.com/s/1soGEzvU1dBHau-r0WTFaNQ**_

_**提取码：0000**_

使用 binwalk 提取文件系统

```
binwalk –Vme HG532eV100R001C01B020_upgrade_packet.bin
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibb3FdrLuCgfDW01cnVewPg4PtsPat1rkxl5wKuNs84FnISLhhYEHbxA/640?wx_fmt=png)  

出现了报错，查看下报错内容：

是缺少 sasquatch

开始出现报错后解压出来的文件是空的

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnib0mNQIXUnlD3qd3dPZj7B4d0HrjP8yeSEcb6SDCiaOeaJrDO2ZZlEglw/640?wx_fmt=png)

经过墨鱼实验室大佬的指点后添加了相应的功能

```
sudo apt-get install build-essential liblzma-dev liblzo2-dev zlib1g-dev
```

以及安装 sasquatch

https://github.com/devttys0/sasquatch

之后，完美运行

Squashfs 标准格式

Big endian 大端序

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibxdjT0sGR6ZcxWkJibSA6fZZ3eUxicrPf6k0hiarpMdUDFpF6PNpjNjbDA/640?wx_fmt=png)

出现了文件系统

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnib5UTPkmuERLsia3Ta7GyvDk9CEjAhbCQ8bMVNJxtCFb9Cwh5FbPr7EbA/640?wx_fmt=png)

找到文件系统中的一个可执行文件进行文件属性查看

系统分为精简指令集以及复杂指令集

大端序和小端序

使用 file 命令进行查看

MIPS 属于 RISC 精简指令集的体系 (《解密家用路由器 0day 漏洞挖掘技术》1.2.1 章节)

这里是 MIPS32 位所以启动 qemu 的时候选择 32 位的启动命令

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cniblhtHibxEODicWcfa7qV0MrL2q9SktLHmICCksviaZXStY8sxpJ9dIZmPQ/640?wx_fmt=png)

接下来我们使用 qemu 进行虚拟环境搭建的时候需要选择 MIPS 的内核

https://people.debian.org/~aurel32/qemu/mips/

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibFmibg2y4ibJHCO3K69DlIdFuSoGrsBlibfKl5ibA0bichPAOFnTfZngTfaA/640?wx_fmt=png)

这里需要配置网络环境，我们在虚拟机中的 ubuntu 系统搭建 qemu 虚拟环境

我们需要桥接一个网卡

目前是 ens33 网口

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibnzCiaH13A0ictnFhicULLuKVYiarIfEQBy9aVTJniagJibpzc2Cttft75nog/640?wx_fmt=png)

创建一个新的网口

```
Tunctl –t top0 –u root
Ifconfig top0 192.168.10.1/24
Ifconfig
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibaZWuEyGv7HibWtopYGRyaSyZbXNupl2DhSvumTfDAhrkrOTYCibyE4nA/640?wx_fmt=png)

开启 qemu 虚拟环境

```
qemu-system-mips -M malta -kernel vmlinux-2.6.32-5-4kc-malta -hda debian_squeeze_mips_standard.qcow2 -append "root=/dev/sda1 console=tty0" -net nic -net tap,ifname=top0 -nographic
```

我们在末尾添加了网络配置让 qemu 虚拟机将 top 网卡作为网关  

-nographic 不显示窗体

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibdnHoolISPSWt858JKunq85ibVpd6EmuXMIJiaMGlymoXpL3ztKdEkjBA/640?wx_fmt=png)

正在启动

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibSPFIb9k9RyBEIqhbqb9QWaQleulUZBM7VsVOibxicaOGpsHZ13ky9RHw/640?wx_fmt=png)

Qemu 的登陆密码是 root/root

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibAetATQ3aiaau0SEZhE3oiatLYbJoyYL3nuQWrcic6h0iccwuXVv1xtCI4A/640?wx_fmt=png)

我们进入虚拟机进行网络的配置

Ifconfig 查看当前的网络环境

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibgiahAcSYgUogvBTh8LXZsJFBFp3ctN7PjexWicgToWYgMIo4pVRzawzg/640?wx_fmt=png)

配置到我们 top0 网络下，将 top0 作为虚拟机网络出口

```
Ifocnfig eth0 192.168.10.2/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibJRm3Bo1OGgOEL6uXag1RS85wJYQI5DP0htm9uceS5eBVYUV2eWE9lw/640?wx_fmt=png)  

测试一下网络

可以 ping 的通我们的虚拟机

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibSLdMMzqUUuYbXywzQw3T8M6MfULiaNsHl1BicCUsQtDT14hLa8xZIjtQ/640?wx_fmt=png)

我们使用 scp 将系统文件上传到 qemu 虚拟机的 root 文件夹下

先打包成一个 tar 文件

```
tar -czvf 1.tar squashfs-root/
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibqw1lv8QgCePY6L8xeuObzeh1p2YQE33tQKI39rpfAGDv4SibGibUQNuQ/640?wx_fmt=png)  

打包完成后进行上传

```
scp 1.tar root@192.168.10.2:/root/
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibUNDn1oxxafPiar0Dl5JVf53VIV8A7DgAl6iaXzIjFcib8B6mWVoBVOAVQ/640?wx_fmt=png)  

查看到 qemu 虚拟机内已经接受到 1.tar

进行解压

使用 tar –zxvf 1.tar 命令进行解压

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnib1OZuevlUwnnpftbpBKOXeYsiagvL7YH0rfDPuHq8BAyBEzZo0vnE6iag/640?wx_fmt=png)

进行文件的挂载

```
mount -o bind /dev ./squashfs-root/dev
mount -t proc /proc ./squashfs-root/proc/
```

挂载完成后启动 shell

```
chroot ./squashfs-root/ sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnib5v1eP45WABBpRslCcP1P4GuiaCXGFaK5iclMTKH4D7RXibVwNX6v7uSJg/640?wx_fmt=png)  

我们使用 ssh 直接连接到 qemu 虚拟

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibZ0icCMmweQ6ocYHwUIYSCoLuhqHibKVVcia4Q2W3Wsv78dqX1Yu1MM5iag/640?wx_fmt=png)  

再次启动 shell

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibwqgZF7kv2pKLqJP1JlphUAB9yCkGyZiaF6waFTDfaXGdwPp0D44apjw/640?wx_fmt=png)

下面开启路由器的环境

```
./bin/upnp
./bin/mic
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibUPhWdkEFLcY0oKQ6ugAe9FzSibNYELGcCnJbdpNNPmTqSduagmick3rg/640?wx_fmt=png)

路由器的虚拟机启动成功

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cniblXFdVZFhtUl58vdaSbswnI7Jiab8I5onWlL8gEib6icC6unyOshrsD3UQ/640?wx_fmt=png)

我们使用 nmap 进行端口的扫描

```
Nmap –Pn –T4 192.168.10.2
```

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibXcbBXQiau8esQNC67DCXQDibj09rmW2G6mzH37RfHwhK5ycm4HEKXjjg/640?wx_fmt=png)

登陆的密码在 tar 压缩包中的 txt 文件中

Admin

@Hua1234

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibNS1b6PjlxbUKicy6Gv2epNpia8Yd61Zibf03s8l0bRpzh9zXc8bxbdncw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/vxZrt4kAItnZIuFULzhbPY4fwibNf5cnibPRdCbx9O4CUF6MMAedlywibZ3EODsfydhThkVfkxQdwoemwKFKHEiaXA/640?wx_fmt=png)  

在最后再次感谢墨鱼实验室大佬的帮助

![](https://mmbiz.qpic.cn/mmbiz_jpg/vxZrt4kAItnZe81OLsXwqBQ5Xic67mHDUcKjZUh7VbuMUjia0slt3gUicGPuaRBTiaHLqwedR6XmD5BibpHdNNiahX3w/640?wx_fmt=jpeg)