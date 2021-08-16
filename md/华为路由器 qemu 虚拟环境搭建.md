> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PUhGsm2GdjX0loD-8yL6yA)

本次针对的路由器固件是华为路由器

下载地址

链接：https://pan.baidu.com/s/1soGEzvU1dBHau-r0WTFaNQ

提取码：0000

使用 binwalk 提取文件系统
=================

binwalk –Vme HG532eV100R001C01B020_upgrade_packet.bin

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25TkYqRGH3JMJkql8EicKNzdWnDcORUXibQ3ibo63ib3XrIb7AI6OrColg3A/640?wx_fmt=png)

出现了报错，查看下报错内容：

是缺少 sasquatch

开始出现报错后解压出来的文件是空的

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25NDeVmKNsuxSiaF8XLLiaMhicKuZfRl87TISzM5RbmsPFg5ZLcicKhNZ42g/640?wx_fmt=png)

经过墨鱼实验室大佬的指点后添加了相应的功能

sudo apt-get install build-essential liblzma-dev liblzo2-dev zlib1g-dev

以及安装 sasquatch

https://github.com/devttys0/sasquatch

之后，完美运行

Squashfs 标准格式

Big endian 大端序

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25IfXeCWKiaDCx3PctRTTXefHHhW3iasX7Mq3fcs8pHI0vmauWYiaorZJzA/640?wx_fmt=png)

出现了文件系统

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq251wGVj6WLiaCCr6nLtllJseDW5nqUd5ylmJOLGibwySG6CS78bSxqibwLA/640?wx_fmt=png)

找到文件系统中的一个可执行文件进行文件属性查看

系统分为精简指令集以及复杂指令集

大端序和小端序

使用 file 命令进行查看

MIPS 属于 RISC 精简指令集的体系 (《解密家用路由器 0day 漏洞挖掘技术》1.2.1 章节)

这里是 MIPS32 位所以启动 qemu 的时候选择 32 位的启动命令

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25VAqRpQjoIzptQic0tRlFmOOIAhAYW26q2YIW3xmSqIaWickeGiaKb09Uw/640?wx_fmt=png)

接下来我们使用 qemu 进行虚拟环境搭建的时候需要选择 MIPS 的内核

https://people.debian.org/~aurel32/qemu/mips/

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25AlVlS4rGDTdibdY6ayYePfnJ9NzfVHxjECwaXMmCViadx2PV8fhuAUQw/640?wx_fmt=png)

这里需要配置网络环境，我们在虚拟机中的 ubuntu 系统搭建 qemu 虚拟环境

我们需要桥接一个网卡

目前是 ens33 网口

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25gLTHua9S8zoaWUrfvGqYRKWU93pcskT8rZtKUeMOefxZPpxSaAVuSQ/640?wx_fmt=png)

创建一个新的网口

Tunctl –t top0 –u root

Ifconfig top0 192.168.10.1/24

Ifconfig

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25xUPjAzYQYrb6obqYQn43CCjBMNGxa7fvRXpCibt1cUKQTtSY515Vniag/640?wx_fmt=png)

开启 qemu 虚拟环境

```
qemu-system-mips -M malta -kernel vmlinux-2.6.32-5-4kc-malta -hda debian_squeeze_mips_standard.qcow2 -append "root=/dev/sda1 console=tty0" -net nic -net tap,ifname=top0 -nographic
```

我们在末尾添加了网络配置让 qemu 虚拟机将 top 网卡作为网关

-nographic 不显示窗体

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25TLOFqvnVkHkgGuN4RoPYGZ0bE5efEQia7D2ovtOm1rzUpFfibSv1nibdg/640?wx_fmt=png)

正在启动

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq258pLBYqibTFdUds0BrzZibibRmNInVqZVKGgsG3m3mM9T9tS6bTUZAbxvw/640?wx_fmt=png)

Qemu 的登陆密码是 root/root

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25ZHQKiaKTayQsV6D2T7TsQwuEPVSibYjFLnic6TChe4F8SIIjhYryibttoA/640?wx_fmt=png)

我们进入虚拟机进行网络的配置

Ifconfig 查看当前的网络环境

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25LqGPvddanT1w0onhaUpcFiaAclFmbkicqbvPExxb8f5dUhIqZrAJiaPsg/640?wx_fmt=png)

配置到我们 top0 网络下，将 top0 作为虚拟机网络出口

Ifocnfig eth0 192.168.10.2/24

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25j0aV8ibPoAjTUHjoje6f0xUNpkPhHqSkicpRpxicGwZOS2WoAmCluJRhQ/640?wx_fmt=png)

测试一下网络

可以 ping 的通我们的虚拟机

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25llC7uVFsER0FfJR1FVxgttwbbKlDsXBkY1MTb8p6OtezGGiaQ7HwuMg/640?wx_fmt=png)

我们使用 scp 将系统文件上传到 qemu 虚拟机的 root 文件夹下

先打包成一个 tar 文件

tar -czvf 1.tar squashfs-root/

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25rd9f2bE9CgicpZtPVicngsWjO1HLUOsgQ3g9V1Sq5P7tVqDAeTxfmIxQ/640?wx_fmt=png)

打包完成后进行上传

scp 1.tar root@192.168.10.2:/root/

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25OPnAXq2Wh7JwUOkR3q4uI1jBM6NTns7S1weYBRiaUqWV5CBfZ1ibN5Lw/640?wx_fmt=png)

查看到 qemu 虚拟机内已经接受到 1.tar

进行解压

使用 tar –zxvf 1.tar 命令进行解压

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25EzfO0yvF79ZS1eDeMlFMGRlsXTicVwptib1YTaZdibFDianJuMVIGamWEw/640?wx_fmt=png)

进行文件的挂载

mount -o bind /dev ./squashfs-root/dev

mount -t proc /proc ./squashfs-root/proc/

挂载完成后启动 shell

chroot ./squashfs-root/ sh

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25POPicDRXZLSFbzwEYuiaerIffWkBT3tQSN6PgphVibRiafCtaAA7brVD6g/640?wx_fmt=png)

我们使用 ssh 直接连接到 qemu 虚拟

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25ASR0xCvhIFkicowlVrrsTDxzicYkGDpiaxTFcEspuBzMxHqrYD67Wr2Gw/640?wx_fmt=png)

再次启动 shell

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25B6DyNqVm3l9g9rOauAI0c8BUZf1krAQobeWpiaMFMTG3gVvSAss3h0g/640?wx_fmt=png)

下面开启路由器的环境

./bin/upnp

./bin/mic

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25XbXI4iaDbG8kZiaWv9zO8Iz5rfkIR0ROS9CRlS1CB0LicZJYOdqNbVWqA/640?wx_fmt=png)

路由器的虚拟机启动成功

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25ch7DwpYqFCvIJ98miawb9UaXEaGGw2ib2FhfsNxbGX13telvPb51O6dw/640?wx_fmt=png)

我们使用 nmap 进行端口的扫描

Nmap –Pn –T4 192.168.10.2

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25XOpI8j2uoXuvjuCic9xGtYg0sbXRbrpdtjtLS1tZVLHnuLUbNt5V1VA/640?wx_fmt=png)

登陆的密码在 tar 压缩包中的 txt 文件中

Admin

@Hua1234

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25zsibsYCwQMU3ycHZHEciatRfzMwOics70tnt8ry5ONlJpVTuOdH40gfgg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/KGyt5Iecd5FNUOyMNLibXs0iaooUAOnq25LM1Xiaic0CcWfdS9h8C6ic4ueugic2qZszEVFHGAvF7mlsCRFn0uFTg3sg/640?wx_fmt=png)