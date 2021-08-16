> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LhDyq9T7EHnjf2d9Qdfgdw)

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrWvy6u5Wib0RibO61K1Mbr0Rz5JwDs32ia6b2JGociawXxMt649icOC1GmQQ/640?wx_fmt=png)

QEMU 是目前最先进的动态二进制翻译跨平台仿真软件，它可以模拟 x86、ARM、ARM64、MIPS、PowerPC 等架构。QEMU 的原理主要是**将 ELF 格式的可执行文件翻译成中间形式**，然后根据中间形式，拷贝编译好的微操作代码，**形成目标基本块**，最后再执行此基本块。它的总体结构如图所示  
  

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrD4zdh02NnvdhS6B786rk271khzOVVlNreJGZN3ulZ63BfgXmjd0y9A/640?wx_fmt=png)

QEMU 主要有两种仿真方式：  

*   用户模式仿真：允许一个（Linux）进程执行在不同架构的 CPU 上，该模式下，QEMU 可以作为进程级虚拟机
    
*   系统模式仿真：允许仿真完整的系统，包括处理器和配套的外设，该模式下，QEMU 也可以作为系统虚拟机
    

接下来我们从 QEMU 的安装开始来讲解这两种仿真方式，需要用到的固件可以在后台**回复「QEMU」**获取  

1 安装 QEMU

```
sudo apt-get update
sudo apt-get install qemu-system-mips
sudo apt-get install qemu-user
sudo apt-get install qemu-user-static
sudo apt-get install qemu-utils
```

2 QEMU 系统模式仿真

首先我们需要从 debian 官网下载 kernel 和 image，地址如下：

```
https://people.debian.org/~aurel32/qemu/mipsel/
```

「为什么我们这里知道使用 mipsel 呢，你可以在文件系统内随便找一个 ELF 文件，然后使用 file 命令查看一下」

将目录中的所有文件下载到一个 kernel 内即可，同时也将固件解压放到同一目录  

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrKWS9ULXIsibNz8P1e6GN3gdzaedjbzaCf525AFpe80kdfrhaoAKP33A/640?wx_fmt=png)

首先安装虚拟网络设备 tun

```
sudo apt-get install uml-utilities
```

为 root 用户添加网卡 tap0  

```
sudo tunctl -t tap0 -u root
```

设置 IP 地址  

```
sudo ifconfig tap0 192.168.3.1/24
```

查看一下我们设置的 IP 地址  

```
ifconfig
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXr84LE8K6wuoRO5CdcRzUmtibQpfhU6IPZXsqfDuVe1JqQXplL6GUoDKg/640?wx_fmt=png)

进入 kernel 目录，并使用如下命令启动 qemu：  

```
sudo qemu-system-mipsel -M malta -kernel ./vmlinux-3.2.0-4-4kc-malta -hda ./debian_wheezy_mipsel_standard.qcow2 -append "root=/dev/sda1 console=tty0" -net nic -net tap,ifname=tap0,script=no,downscript=no -nographic -s
```

命令解析如下  

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrEXK8us6PjkUyOcaeMG8Aoibu59MIZUks48EAs5wwINWMdcaibIOxv4zQ/640?wx_fmt=png)

效果如图所示  

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrNptqCydeXBdEeoqYcQVzb4ShtwKkGotiaXjCnFLYG88OCt6momu3Sdw/640?wx_fmt=png)

账号密码均为 root

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXriaaAhKGIibx8B57coO9Fj0hINibrvSwbthiabEYWY5ZDlIw8W6TdqbqXIQ/640?wx_fmt=png)

使用 ifconfig 配置仿真机的 eth0 网卡为 192.168.3.2  

```
ifconfig eth0 192.168.3.2
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXr25ZhVN81QbtQc3u7ku1nNGaDJiaxz7I5PnGqdKYfNI1YSOTF0txQbBg/640?wx_fmt=png)

使用物理机测试与仿真机之间的连通性  

```
ping 192.168.3.2
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrUmvwZAUUrRvibBpoNbnicwlMf2hBn2ibLtl5ZzJXObxaiaIicbHZI6B8QYg/640?wx_fmt=png)

使用 scp 命令将 squashfs.tar 传入仿真机

```
scp squashfs.tar root@192.168.3.2:/root
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXr4HBic6ia9Ahduu8RXYATNe7oAsvxa0rr6d5JBdibqHa31ZzMn9H3y29tA/640?wx_fmt=png)

在仿真机内使用 tar 命令解压  

```
tar -zxvf squashfs.tar
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXr0lSeCwVzDqZIia7tOltdjeheWSbRodZ4KNianuZRmqrtiaGCgMGIP5qWg/640?wx_fmt=png)

挂载固件文件系统中的 proc 目录和 dev 目录到 chroot 环境，**因为 proc 中存储着进程所需的文件，比如 pid 文件等等，而 dev 中存储着相关的设备**  

```
mount -o bind /dev ./squashfs-root/dev
mount -t proc /proc ./squashfs-root/proc/
```

使用 chroot 更改 root 目录，系统的目录结构将以 squashfs-root 作为根  

```
chroot ./squashfs-root/ sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrgGIcmSdHpmM8LVBosSTUeoboFJmmY0U3x4oMjXR8E1k29TdHZYicm5A/640?wx_fmt=png)

至此，我们就可以运行该文件系统中的程序啦  

3 QEMU 用户模式仿真

  
因为我们要运行的是 mipsel 的程序，所以这里我们使用 qemu-mipsel 来执行  

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrh3ibTdibxr3eZBX3pliaRewrLvMQibicR3RPiaJn5aPPd6cqCw1eWibxMJiaNQ/640?wx_fmt=png)

但如果有的时候目标程序使用了动态链接库就会导致我们执行失败，这个时候我们只要配合 chroot 使用即可，首先将 qemu-mipsel 拷贝到 squashfs-root 目录  

```
cp $(which qemu-mipsel-static) .
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrmAxHC7cXHDBXXnibO3mxzk0R1pXpEgx1y7icibVxqXBpx1XsurdPVvAnQ/640?wx_fmt=png)

```
sudo chroot . ./qemu-mipsel-static ./www/api
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbtsyz8hiap3Nk8TibmKHjGXrJFpsKx6y97bOgh2erdeicoMf0Nqf0TpVM6sYlibiaYUKTG0qWozrYyAfw/640?wx_fmt=png)  

4 总结  

QEMU 的出现为我们这些测试人员节约了大量的成本。我们可以在没有开发板的情况下进行测试、调试和运行，大大提高了效率  

参考引用：  

```
https://swordfaith.github.io/2019/09/24/QEMU%20%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/
基于QEMU的嵌入式系统仿真环境的构建（陈宇星）
```

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuOLmpibBstibso6rfWkEJFDwibcW4QbNcHtf2GCjia9DQiafAr9nbQnufE5Ow/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuOu5t0fBMsCxzDQnk766HhG7jB3rsQictYQjAdKx8Vv1iaDn97KIZLmS8w/640?wx_fmt=png)[![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuO7PBBlnU8bHMbauw0vriaef9Q24JynYXMwcLlvVn8rFGTXeU8tjBdvGQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzUzMjcxMzg5Mg==&mid=2247488135&idx=1&sn=e0d2834854ecd8b7a2259b2b8c39c7b9&chksm=faae4a4ccdd9c35a3f203671921d09e9bca0134541958e0129ff26882b2c42c38b77136e0971&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuOwABcJoQRcGEbGyTo9y77oDsdVdIm6qjvSPSvDAtGgLdzUOFhzhyxjw/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzUzMjcxMzg5Mg==&mid=2247486554&idx=1&sn=230088994b318cb213e41f23a7093982&chksm=faae5491cdd9dd87521f796e9c1dfdebdb9df6082e623f5c7bd9c2d68056df36cf6beccec093&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuOBRnCibOatR02XnTiaJ7y7JDIiaM3VAJGCicov5bgdJKfsE1KvUJzNpVwfg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzUzMjcxMzg5Mg==&mid=2247485825&idx=2&sn=29ce2ce1183af1204be56d82f0311c94&chksm=faae514acdd9d85c055e9684d74099ff22435858120f9ed759629eeb3d0e1eb05165c0d4242e&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuOKyaOEmT4tk7NYSgpyUb8WnBbzq3GyuRHV2Sc6riaQ9CyCbpFwjVUdDg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzUzMjcxMzg5Mg==&mid=2247485561&idx=2&sn=b41a34ca87b871f70c8b7ab43fb7dc30&chksm=faae50b2cdd9d9a426b5f4cf4c78aaf1671944de79f5e771e920973db78c8e22ff562d03e408&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuOYHBgCMibUV4PbGl9qvVyuTCHAmtZUl4gq6tdaxUvOmvr36QMbibH0UkA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzUzMjcxMzg5Mg==&mid=2247485455&idx=1&sn=a5df769f0231bfcbf54aadf8efa84ecc&chksm=faae50c4cdd9d9d2107bcea1a2b58d81310eaf9fdd8b24d4ff9db7e75cb0eba3b5bd151599c0&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/djiam4RadAPbBHRXaTibGjQ4tCPAAAXOuOff9szInMwnjic9wDFQryX3VdHLr5r1VepL0Wkcpl2uIwGj4GxvqCdWA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzUzMjcxMzg5Mg==&mid=2247485445&idx=1&sn=94649507c498c9e6ab9ea82be6fdad46&chksm=faae50cecdd9d9d8e60734cd8d4222604341ba4ded3fc4d45206f549f628a37bb490f117fd18&scene=21#wechat_redirect)
