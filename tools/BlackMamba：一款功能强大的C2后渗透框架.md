> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-rD-RI5nn9rimPoEbfWtcQ)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38aadGAnQic8Czao3L0wOiak2fZqUwrrvjMQlfcY5rxX70hEohtB1IJJt6bToo0uHU4icncibY7iaTyaUw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

来自 | FreeBuf

BlackMamba
----------

BlackMamba是一款多客户端C2/后渗透框架，并且还支持某些网络间谍软件的功能。该工具基于Python 3.8.6和QT框架开发，可以在渗透测试任务中为广大研究人员提供帮助。

**BlackMamba的功能如下：**

> 多客户端支持：支持同时连接多个客户端；
> 
> 实时通信更新：支持客户端和服务器端之间的实时通信和更新；
> 
> 通信加密：支持对除了屏幕视频流之外的所有通信信息进行加密；
> 
> 截屏收集：从客户端获取实时截屏；
> 
> 视频流：实时查看客户端屏幕视频流；
> 
> 客户端锁定：锁定和解锁客户端设备；
> 
> 文件传输加密（上传/下载）：可从客户端下载文件，或向客户端上传文件；
> 
> 键盘记录：记录客户端键盘按键信息；
> 
> Web下载器：支持从URL下载文件；

工具安装-服务器端
---------

首先，使用下列命令将该项目源码克隆至本地：

```
git clone https://github.com/loseys/BlackMamba.git
```

接下来，安装PIP包：

接下来，在网关或路由器打开端口65000和65005，具体端口号可选。为BlackMamba创建防火墙例外规则，或直接禁用防火墙。

打开“BlackMamba/bin/profile/socket.txt”文件，然后输入打开的端口号：

打开BlackMamba目录，然后打开“keygen.py”文件。拷贝结果密钥并拷贝到“BlackMamba/bin/profile/crypt_key.py”文件中。

返回BlackMamba根目录，然后打开“main.py”文件：

### WINDOWS

```
python main.py
```

### GNU/LINUX

点击一个人形和加号的按钮，输入待创建Python文件的路径，输入端口号和主机的IP地址，然后点击“创建”按钮。

工具安装-客户端
--------

创建好客户端脚本之后，你将需要在目标主机上运行该脚本。

### Windows

```
python script.py
```

### GNU/Linux

下载代码包：

然后运行下列命令：

注意事项：客户端脚本并不会实现持久化运行，如果你想实现持久化，你得需要自行动手实现。除此之外，客户端脚本可能会有几秒钟或几分钟的延迟，具体取决于通信连接的质量。

工具运行截图
------

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38aadGAnQic8Czao3L0wOiak2J4TmUdrCgzE45A9gRblaxiaTDZO8WTfD6Riayltpb1msnjExrGXMLOiaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38aadGAnQic8Czao3L0wOiak2ha85NZkW9KNCMuLZZuP8TVHzdBFBVVRKkM1eulYnt0Xq2BMKn1OL5w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38aadGAnQic8Czao3L0wOiak2e7FxRskp5XxDSljH4YakNfHhWDWDuTrNnzvVBl3gk64YXyCxRfic2bQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38aadGAnQic8Czao3L0wOiak2LvjslEVicWUCBicLAcCB31j5yxu6Tv9lMuz2OdTkDNXEe4IQAhxPGW0g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38aadGAnQic8Czao3L0wOiak29tvc7icrXMmdFOH0u3mPkiawJ3PDOORcHTmYfjiax0cqQM8tjVCJB6D2w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38aadGAnQic8Czao3L0wOiak2icXribIbzYpCZSPX19E4XeGDDVr9GsLEEt4DyibYQDy3BUcwSzh0e6BMA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

项目地址
----

BlackMamba：点击底部【阅读原文】获取

  

```
版权申明：内容来源网络，版权归原创者所有。除非无法确认，都会标明作者及出处，如有侵权烦请告知，我们会立即删除并表示歉意。谢谢!

```