> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/U8DoH9Blesb-t5v5onxccg)

作者：漠效  

https://blog.csdn.net/GX_1_11_real/article/details/84571303

**前言**  

=========

无论在哪个系统中，删除文件都是必须谨慎的操作。

因为如果不小心删除了重要文件，就会导致个人或公司出现重大的损失。

类似于 windows 系统误删了文件，可以使用一些软件进行恢复操作。Linux 也是有几款软件可以做到误删恢复的。

注意事项：虽然有软件可以对误删的数据进行恢复，但是完全恢复数据的概率并不是百分百的。

因此，使用 rm 命令删除文件的时候，一定要小心；重要的数据一定要有备份；并且恢复删除的数据前，删除文件的目录内不能往进存放新东西，否则覆盖掉的信息无法找回。

下面介绍的就是对 Linux 中误删文件的恢复操作。

**1、lsof**
==========

**原理：**  
这个命令实际上并不能直接用来恢复文件，不过它可以列出被各种进程打开的文件信息。

配合其他命令，从 / proc 目录下的信息中恢复 “文件已删除，但进程仍保持打开该文件的状态” 的文件。

/proc 目录是挂载的是在内存中所映射的一块区域，当我们对这些文件进行读取和写入时，实际上是在从内存中获取相关信息。

因此，当我们对文件进行读取或写入时 (即有进程正使用文件时)，哪怕硬盘中的该文件已删除，还可以从内存中的信息恢复文件。

**注意：  
**必须以 root 用户的权限运行， 因为 lsof 需要访问核心内存和各种文件。

  
只能恢复 “文件已删除，但进程仍保持打开该文件的状态” 的文件。

如果误删了目录，目录中的其他文件未被进程打开，没有进行使用的文件将无法使用此方法恢复。

**lsof 输出信息的意义：**

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyZO6QtBVlxq4cuEByEkORq1qQntgfLSmAcweehm2f8iaUzCAUywMYqhA/640?wx_fmt=png)

```
COMMAND       进程的PID(进程标识符)
USER          进程所有者
FD            用来识别该文件(文件描述符)
DEVICE        指定磁盘的名称
SIZE          文件的大小
NODE          索引节点(文件在磁盘上的标识)
NAME          打开文件的确切名称
```

`**最常用参数:**`

```
-c       显示某进程现在打开的文件
 -p       显示哪些文件被某pid进程打开
 -g       显示归属某gid的进程情况
 -d       显示目录下被进程开启的文件
 -d       显示使用fd为4的进程
 -i:80    显示打开80端口的进
```

**恢复文件操作**
----------

环境：  
在 / mnt 下有一些文件，其中一个文件 train.less 正在被查看，然后另一个终端将其删除

**【1】lsof 查看**

查看正在使用删除文件的进程号

`

lsof /mnt

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyqIsfbwDo849Kl7QXG9Rm9CRrmjqpBcfuCYr9wgLeaQv8wQFD4gbZlA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYya0eEQgASW0QeIIt5XRpOPdE8TSCBIN0e8pPaoj89x3p3xYQ49bvIDA/640?wx_fmt=png)  

**【2】恢复**

切换到 / proc 下，删除文件对应的进程的 pid 下的文件描述符中的目录中；将对应的内容重定向或 cp 到其他文件中  
重点关注：PID 与 FD

```
cd /proc/31284/fd/
   cat 4 > /mnt/ferris_train.less
```

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyZ2gZwamQTJhM2hRQv5YdhmTSPJBVxhxtPsv3agLrgmcE6zWv9OibeYQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYykC2OUtYWqLay1ckHyJX53Cm6hGevRicsxxFTBoGf0uib0ENZfbHt1tHg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYymQjZxZ1Ks6IiarXO9juQ1Geic4hFtr3us2ecu78RpGHC9NU3iaF9Pcb8g/640?wx_fmt=png)

**2、extundelete**
=================

**原理：  
**使用存储在分区日志中的信息，尝试恢复已从 ext3 或 ext4 的分区中删除的文件

**优点**：  
相比于 ext3grep 只能恢复 ext3 文件系统的文件，其适用范围更广，恢复速度更快

**extundelete 官方地址 (官方文档)：  
**http://extundelete.sourceforge.net

**extundelete 下载地址：  
**http://downloads.sourceforge.net/project/extundelete/extundelete/0.2.4/extundelete-0.2.4.tar.bz2  
(最新版本的 extundelete 是 0.2.4，于 2013 年 1 月发布)

**注意：**

*   在数据删除之后，要卸载被删除数据所在的磁盘或是分区
    
*   如果是系统根分区遭到误删除，就要进入单用户模式, 将根分区以只读的方式挂载, 尽可能避免数据被覆盖
    
*   数据被覆盖后无法找回
    
*   恢复仍有一定的机率失败，平时应对重要数据作备份，小心使用 rm
    

**安装**
------

**1、依赖安装**

```
centos安装操作
yum install e2fsprogs-devel   e2fsprogs* gcc*

ubuntu安装操作
apt-get install build-essential  e2fslibs-dev  e2fslibs-dev
```

**2、编译安装**

```
wget http://downloads.sourceforge.net/project/extundelete/extundelete/0.2.4/extundelete-0.2.4.tar.bz2
tar xf  extundelete-0.2.4.tar.bz2
cd  extundelete-0.2.4
./configure
make
make install
```

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyTJx3cstsiaicq5zS9HhBoPicicLGhO5y7NeDvpBYiaqiaSgzVtkW5aMLGcAQ/640?wx_fmt=png)

`

cd /root/extundelete-0.2.4/src

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyRa7RXNj2icclW3vmIZE26gbuXNxqcAJoAQS86UWCQibmBWgZQH07Of6w/640?wx_fmt=png)  

`

extundelete -v

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYypf4Bk3w0icAd1D1ibTZ2mSCxp7gpphQjLMC23nSQKj9fOSWqDLlicYWibA/640?wx_fmt=png)

执行 make 命令会在 src 目录下生成 extundelete 可执行文件，可在此直接执行恢复命令。

  
执行 make install 会将程序安装在 / usr/local/bin / 下

**恢复文件操作**
----------

执行 extundelete 命令的当前目录必须是可写的。

**1、查看要恢复文件的分区的文件系统**

`

df  -Th

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyFd7axlavaA1Uof8YdOMqUtM6SnAia9EIGsCjJfzuYfmxGzhmeibr5xYA/640?wx_fmt=png)

**2、对要恢复文件的分区解除挂载**

`

umount /mnt

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyvgYJZvC5nbJic1rwlTMbLasibzQVju5IKQYp2iaQrj07AkIdtZbIB46IQ/640?wx_fmt=png)  

**3、查看可以恢复的数据**

指定误删文件的分区进行查找  
最后一列标记为 Deleted 的文件，即为删除了的文件

`

extundelete /dev/vdb1 --inode 2 （根分区的inode值是2）

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyZzuYd7YicLkU1wyv8zMSbVamHnHSLJZ4E46j7iaOBfLvqjfr8fD0xosg/640?wx_fmt=png)  

**4、恢复单个目录**

指定要恢复的目录名  
如果是空目录，则不会恢复

`

extundelete /dev/vdb1 --restore-directory  ferris

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyrkw2RQtsic584TcPfOaI5a4pibDX1UUlpmlPDwEyJjyuscbCaqCfjgPg/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYydAw52iaXFL0y0QNb6dp4ffDp2rJpJGB4XqoaIafCibeotN8GPia3kCHicg/640?wx_fmt=png)

当执行恢复文件的命令后，会在执行命令的当前的目录下生成 RECOVERED_FILES 目录，恢复的文件都会放入此目录中。如未生成目录，即为失败。

**5、恢复单个文件**

指定要恢复的文件名  
如果几 k 大小的小文件，有很大几率恢复失败

`

extundelete /dev/vdb1 --restore-file openssh-7.7p1.tar.gz

`

![](https://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0DAFPX7EWXb2BDW26A8SYyOtn0IypeYMvvDn13Llo7libcUzecGO2m26MicuHdCzWoOYicKfg1WQoQg/640?wx_fmt=png)

**6、恢复全部删除的文件**

无需指定文件名或目录名，恢复全部删除的数据

extundelete /dev/vdb1 --restore-all

**版权申明：内容来源网络，版权归原创者所有。除非无法确认，都会标明作者及出处，如有侵权烦请告知，我们会立即删除并表示歉意。祝愿每一位读者生活愉快！谢谢!**