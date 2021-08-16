> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qwvs2c__XntvH_uXYJzXiw)

> Immunity Canvas是ImmunitySec出品的一款商业化、跨平台的漏洞检测工具，目前有800+个漏洞利用程序，由于商业性质在国内使用并不是很广泛。

2021年3月2号，安全研究员Ege BalcI在Twitter上有披露了有关Immunity CANVAS 7.26工具源码泄露事件。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/5y3Pga6Bj5rkIOUyDibqbZibXictVXeEDPfa4w2ItyUoDkPHys7vuumcTW04OCgbRjTElzECbSSp8xDG294gZ6QGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

武器库分类大致如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/5y3Pga6Bj5rkIOUyDibqbZibXictVXeEDPfdAfbvsx4PtmbGKWolXYD9fHnZYIic8GI4KuiaSERicRBEcobQlkUfKQXg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

部分EXP：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/5y3Pga6Bj5rkIOUyDibqbZibXictVXeEDPf8JrgsgxLs60JbxUOB1zicRLqmgWY9xQN64CFc57kHyfHac80GBHwrFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

下载地址：
-----

Canvas主程序 链接：https://share.weiyun.com/aN3jLp9u 密码：g6nt7r

windows依赖（linux运行无需下载）:

链接：https://share.weiyun.com/jFBphkp5 密码：ysewgn

解压密码：www.1937cn.com

测试环境：

```
Ubuntu 18.04
```

Linux安装
-------

安装依赖：

```
sudo apt-get update  
sudo apt-get -y install python-pip  
sudo apt-get -y install gtk2.0  
sudo apt-get -y install python-glade2  
sudo apt-get -y install python-nacl python-bcrypt  
sudo pip install pycrypto  
sudo pip install pyasn1  
sudo pip install diskcache==4.1.0  
sudo pip install asn1tools  
sudo apt-get install -y python-pycurl  
sudo apt-get install -y libcanberra-gtk-module  
sudo pip install pycurl  
sudo pip install requests  

sudo pip install pygame

  



```

进入Canvas目录执行:

sudo bash installer/linux_installer.sh

  

安装完成后进入Canvas主目录执行：

sudo python runcanvas.py

  

完成后如下所示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

主界面： 

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Windows安装
---------

基础环境：

```
* Python 2.6 or 2.7  
* GTK  
* PyCrypto (some modules)  
* Py-GTK and its associated libraries  
* Pyasn1  
* CANVAS STRATEGIC: ZeroMQ/PyZMQ  
* Pynacl  
* Bcrypt  

* Asn1tools

  



```

安装依赖：

管理员权限运行CANVAS_Dependency_Installer,pycurl-7.43.0.win32-py2.7（打包目录下包含，直接运行即可）

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

安装成功后如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

使用指南：

http://www.immunityinc.com/downloads/documentation/tutorials/canvas101-part1.pdf http://www.immunityinc.com/downloads/documentation/tutorials/canvas101-part2.pdf http://www.immunityinc.com/downloads/documentation/tutorials/HCN-Part1.pdf

ps:现已加入黑客画像渗透工具包

原文地址：http://www.1937cn.com/thread-248-1-1.html

  

有问题请在社区交流