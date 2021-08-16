> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jTKN6eiSbCt672PP6RKvEA)

**0X00    前言**

Traccar 是一个开源的 GPS 跟踪系统。此存储库包含基于 Java 的后端服务。它支持 170 多种 GPS 协议和 1500 多种型号的 GPS 跟踪设备。Traccar 可以与任何主要的 SQL 数据库系统一起使用

开源地址：https://github.com/traccar/traccar

官网地址：https://www.traccar.org/  

这款开源的 GPS 追踪系统，实测后效果不错，精度在 10 米左右。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKTN6ApiaKGah9jPPDxwVTVblj7iaN7z296I1aL2pibzEI0YnNiasHVqfx0Q/640?wx_fmt=png)

以及官网支持手机或者 GPS 的定位器相应的型号  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEK6oB9Nkok7tOoAs7MpI6qPoT7DZ1TbxvibvMGqBjobSuOtUpyagBFYwA/640?wx_fmt=png)

 **0X00    Traccar 是什么？**

**Traccar 是一个免费的开源现代 GPS 跟踪系统，支持 170 多种 GPS 协议和超过 1500 种型号的 GPS 跟踪设备。**  

可以满足

*   出租车，货车，卡车 / 拖车
    
*   农用设备，车队，集装箱，船舶，全地形车
    
*   专人跟踪，个人车辆，手机
    

等追踪定位需求。

Traccar 的功能非常多，可**切换卫星、街景地图，****追踪运动轨迹，****追踪行程**，**停留点**等

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKN1gXa1EuR05HlAffPMNawDboTuubxALuPytLVzgxFbAgBxRdL1I6EA/640?wx_fmt=png)

**追踪行程**  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKicSPuhibyic4aGoqgzw2aA5zztfLB9pjM8DBz9fE3hxJuYQgmCBITparQ/640?wx_fmt=png)

**停留点**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKHKq3H1lB4Sa8ic4RSibfFv2icNjK5zsOcmp62cm51WNDktokq0icESJJKA/640?wx_fmt=png)

**0X01    搭建 Traccar 服务端**  

我用的是阿里云香港 ECS 的云服务器，教程算是非常傻瓜了，需要有那么一点 Linux 基础，不懂的可以楼下问或者度娘谷歌，安装好之后就可以登陆了

使用宝塔面板或者 Centos 等其他 Linux 或者 Windows 都是可以  

只需要有 JAVA 环境和 MYSQL 环境即可

Ubuntu 16.04 x64 系统，1 CPU，25 GB SSD

先使用 SSH 连接到云服务器，然后 APT-GET 更新

```
apt-get update
```

安装 Java 和 MySQL 服务器

```
apt-get install unzip default-jre mysql-server
```

中途会让设置 MySQL 密码（回车则默认为 root ，为了安全建议自己设置）

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKicmhrz1dQnTHH6rMswaspbpBhpfkGvplibiaoH8QiaS7dqCyUmkibHZzqQw/640?wx_fmt=png)

再次输入确认密码

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKfJeFR4h4IID8ew38fKuyicICsLpd18CgFqPuAZ2kjYLBng2RNH5r7wQ/640?wx_fmt=png)

创建一个新的数据库  “traccar”  ，使用上一步设置的 MySQL 密码登陆

```
echo "create database traccar" | mysql -u root -p
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEK10wZZLRtGah8pHhDufhx9qvH5aZia36eavlsiaYg9eUmFmh7PvIkXa3Q/640?wx_fmt=png)

下载 Traccar 安装压缩包  

```
wget https://github.com/traccar/traccar/releases/download/v4.12/traccar-linux-64-4.12.zip
```

解压压缩包  

```
unzip traccar-linux-*.zip
```

安装 Traccar 服务端

```
./traccar.run
```

创建配置文件 “traccar.xml”

```
vim traccar.xml
```

编辑完按 ESC，然后输入**:wq**，保存退出即可  

输入内容（红色区域换成自己 MySQL 密码）

```
cp traccar.xml /opt/traccar/conf/
```

PS：上述 8082 端口也可以自行设置，以及数据库用户名和密码，如果是云厂商的服务器，记得在云面板放行开启的端口哦！

替换默认配置文件

```
systemctl start traccar
```

启动 Traccar 服务

```
systemctl status traccar
```

检查 Traccar 是否正确启动

```
systemctl status traccar
```

如果看到 active 即表示启动成功

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKNomdnGicQGvTuiaesZsID4sicNXf1QpUMMlPwRXmz5NFBx1NWciazdMadw/640?wx_fmt=png)

**0X02    登陆 Traccar Web 管理**  

浏览器输入   (http:// 服务器 IP:8082)，语言选择中文即可

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKeCYfGBuiaNxY5nWsRNIqicfGG8AYbbUibBXK00O7EibTcyMicgkMSl7aFOg/640?wx_fmt=png)

**初始用户名和密码都是 ：** admin

（为了安全登录后，登录后记得更改密码）

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKxhWIKp5vFsURVdYVtibYqP5NSsn6COhQrCicibgN4RT9n9FAw62DnzW7g/640?wx_fmt=png)

添加设备和设备编码  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKghedGHltZiaI3OfgkZfKYFELYUYibfia5If6zcblglMUoFfMKHYUplCsg/640?wx_fmt=png)

**0X03    安装 Traccar 客户端**  

=============================

安卓的可以直接 Google Play 商城搜索 traccar 下载或者 apkcombo.com 上搜 traccar

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEK6QYqNoL5zGoFTWmaD2pxpWI4lm9ZJAaN3An7XTO2spc4MxoVsGPFlg/640?wx_fmt=png)

Andorid 和 IOS 均可以上官网下载安装

https://www.traccar.org/client/  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKniaX3B3KyGiaccpyyksFcM5GFGBEnTFZfkwGNibGOPuE02iaLsmt2GB9tA/640?wx_fmt=png)

安装打开，给予权限

然后设置

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKXoibqF3icNxwcKj7VTMCks7c5eS8iaIK48PGAqbM73thzSicFctFFoOD3A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKEqbPxae5MiclGgbrmIvuDvTp9JbobFR5icP6JETlESdPFMWBJhnVnVfA/640?wx_fmt=png)

**然后打开 Traccar Web 管理界面点击跟踪就可以定位到你的位置了**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKCZokFicgwUVSOuxlHLFCfTVxzcKP1zJ6hAbmkeKyPGO5oI0Or5FW9dg/640?wx_fmt=png)

Web 端查看设备信息  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKNafW6hfq7qyplxv9UOtibZ72m8zmUnp1iavwAogT1DZy57g5jibZP4C2Q/640?wx_fmt=png)

点击设备名称  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKE7XSkUebcrr2AaXW79iaRxBSicxfsv6KWFrMQICMkea3VGqfIlGgV4XQ/640?wx_fmt=png)

地图服务器选择  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKuQ5MbusYz0IdqIhVjsrAviaA2qk64qOaGuUx81KYO7PTDeIgAxGA3rQ/640?wx_fmt=png)

管理界面还有很多设置，小伙伴们可以自行探索。

给女朋友装上再也不怕女朋友出门了![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouibgt3bGyUL3FGiadA36NNWEKKAahlYNCGs2icpPbCaxdUA8yuib85GVlgO2jjurlxqn5lUo7XtjiaQECA/640?wx_fmt=png)，实时监控着。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动

[![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibHHibNbEpsAMia19jkGuuz9tTIfiauo7fjdWicOTGhPibiat3Kt90m1icJc9VoX8KbdFsB6plzmBCTjGDibQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496352&idx=1&sn=df6ddbf35ac56259299ce37681d56e5b&chksm=ec1ca79fdb6b2e8946f91d54722a7abb04f83111f9d348090167b804bc63b40d3efeb9beabbe&scene=21#wechat_redirect)

**点赞，转发，在看**

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)