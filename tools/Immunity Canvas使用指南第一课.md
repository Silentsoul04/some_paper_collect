> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/UgyIacdcccdHLEUYiQGkWA)

**点击蓝字**

![图片](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**关注我们**

  

  

**_声明  
_**

本文作者：北美第一突破手  
本文字数：2500

阅读时长：30分钟

附件/链接：点击查看原文下载

声明：**本文仅供学习参考，请勿用作违法用途，否则后果自负**

本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCRdEUkWjnADMBeY9cRyYo7UwyjdSU8dcm3AzduFQpjTX8Cta5xQVGAdjdaNiae0yof2agC45uR2pg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

![](http://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDjy8pCtpvJKBibCLXQDm14MbdlTqXYESXADHkVpL6f81Z4TVFOGQMjBjgxPpUcYnzahRhibQUdcKzQ/0?wx_fmt=png)WgpSec狼组安全团队推荐搜索安全研究漏洞复现安全工具

  

  

**_前言_**

  

    Immunity CANVAS是Immunity公司的一款商业级漏洞利用和渗透测试工具，此工具并不开源，其中文版介绍如下：  
    “Canvas是ImmunitySec出品的一款安全漏洞检测工具。它包含几百个以上的漏洞利用。对于渗透测试人员来说，Canvas是比较专业的安全漏洞利用工具。Canvas 也常被用于对IDS和IPS的检测能力的测试。Canvas目前已经使用超过700个的漏洞利用，在兼容性设计也比较好，可以使用其它团队研发的漏洞利用工具，例如使用Gleg, Ltd’s VulnDisco 、 the Argeniss Ultimate0day 漏洞利用包。”

社区链接：【 https://c.wgpsec.org/p/10056 】（阅读原文一起参与讨论）

一、

**_工具安装_**

该工具(文末附工具下载地址)支持在Windwos和Ubuntu上进行安装使用，但是windwos上环境搭建起来非常的麻烦，所以就直接安装在Ubuntu上，可以直接去官网直接下载镜像，关于镜像的选择，个人建议选择`Ubuntu18.04`的版本，最新的版本在安装的时候会非常的卡，甚至加载不到安装页面：  
Ubuntu镜像下载页面

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在安装虚拟机的时候需要选择高级安装的方式进行安装，方便自己自定义操作，安装的流程不在多说：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

安装好镜像之后，需要修改一下下载源，这里有一个坑，我在使用阿里源和清华源的时候会显示定位不到安装包，换成中科大的源就没有问题，当换源之后就可以开始安装了。

将工具放置到`Ubuntu`中解压之后有下面几个目录：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在`ubantu`上进行安装只需要使用我们框出来的两个东西，其中的txt 是环境安装的命令：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

先安装以下的环境：

```


```
`sudo apt-get update``sudo apt-get -y install python-pip``sudo apt-get -y install gtk2.0``sudo apt-get -y install python-glade2``sudo apt-get -y install python-nacl python-bcrypt``sudo pip install pycrypto``sudo pip install pyasn1``sudo pip install diskcache==4.1.0``sudo pip install asn1tools``sudo apt-get install -y python-pycurl``sudo apt-get install -y libcanberra-gtk-module``sudo pip install pycurl``sudo pip install requests``sudo pip install pygame`
```




```

全部复制然后等待安装完成，安装完成以后直接进入`Install`目录中，先给`linux_installer.sh` 加上权限，然后进行安装，安装完成以后就可以直接进入工具的目录运行

运行的时候按照他的说明是直接运行`runcanvas.py`，但是会发现非常的慢，甚至十几分钟都打不开环境，但是可以使用`Ctrl + C`，不加载对所有的模块。但是事实上还有一个`runcanvas.sh`，使用这个启动的话只要话费几秒就可以直接打开服务，并且加载所有的模块：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

接着是语言的问题，很同学安装之后的界面显示的语言都是中文的语言，非常的不友好，看着很别扭，那么只需要将`Ubuntu`的界面语言改为英文就可以完全解决这个问题：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

当完成这些以后，就安装好了这个工具，鉴于安装步骤太多，大家可以下载打包好的工具，直接导入虚拟机就可以使用，虚拟机环境来自【J0o1ey师傅的团队成员】，工具目录在`/home/CANVAS/`下，改一下默认界面语言就可以使用了。

二、

**_界面介绍与工具使用_**

  

界面介绍

打开工具的大致布局如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### Moudles && Search

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   Moudles
    

这个地方是指改工具的攻击模块，也就是他能够做什么，他的攻击模块很多，包括主机发现，漏洞扫描，漏洞利用，他的漏洞模块目录分的很清楚，按照操作系统进行的划分：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

比如上面的这个目录结构，使用EXP，然后选择的是Local，接着你可以选择平台，windwos还是Unix，选择完成以后可以选择具体哪种类型的主机，比如这里我选择的Windows7，个人感觉非常的人性化，如果你要是找不到你想要的，还可以在副页进行搜索：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

### Node tree && EXP Description

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   Node tree
    

这个界面是是可以显示我们当前的回连主机和攻陷的主机，这个大红色的圆圈就是当前的回连主机，绿色就是攻陷成功的主机，所有的攻击得到的会话会返回到这个地方来，我们可以将一台被攻陷的主机作为回连的主机，继续攻击其他主机，你可以右键这个大红圆圈查看一下信息：  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

包括`target host` 和网卡信息，IP地址等，`knowledge`界面可以添加目标和删除目标

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

选择攻陷的主机作为回连主机：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这个时候可以观察到上面的 `Current Callback`已经改变了：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

接着开始攻击其他主机，攻击成功以后如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   Word map
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

它是用来显示你攻击成功的IP地址的大致地理位置的，使用python的库进行操作，按照他的要求我们可以到`CANAVS\gui\wordmap\`中查看到他的介绍：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

按照他的方式我尝试进行安装，但是没法下载他的`IP`数据库，显示404,：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我也尝试百度下载的`IP`数据库，但是安装上去后就会报错，估计是版本不合适

*   Cmd line
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

命令行输入，也就是和 `CS` 中的无图像化界面的操作差不多的东西，可以使用它提供的命令完成我们之前的一些操作，输入`help`就可以显示相关信息，比如我们使用`help load`:

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
会告诉你相关的信息，以及他的解释

*   EXP Description
    

显示你选择的EXP的详细信息，这里我选择一下`ms17-010`这个EXP，然后就可以查看到相关的信息：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Consle

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   Current status
    

用于显示我们攻击的日志，分为几种颜色，蓝色代表攻击结束，灰色代表攻击中，红色代表攻击失败：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   Canvnas log && Debug log 这里是日输出的位置，实时显示我们的相关日志和报错日志：
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

顶部信息

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   Target host
    

添加一个攻击目标：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

*   stop EXP
    

停止攻击

*   config
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
这个是后续外网搭建的相关配置，之后会写

三、

  

**_实战_**

  

> 首先先吹一波他的 `ms17-010`，是真的非常好用，在我自己测试的时候，没有发现蓝屏，很稳定，并且使用起来方便，快捷

这里我的环境是`ms17010`的漏洞环境，首先我添加一下目标：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

扫描一下开放了哪些端口，模拟一下真实环境:

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

发现开放了445端口：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

那么我们就可以尝试使用一下`MS17-010`，搜索一下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

选第二个，进行攻击，攻击成功后会返回一个 `Beacon`：  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

返回的`Beacon`的一些介绍：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

> 硬盘改为显示当前目录文件。。。

尝试执行命令看看：  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

查看一下目录信息：  
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

然后我们关闭这个`Beacon`，想要再次打开可以点击一下你想要的攻陷主机，然候在`modules`中选择 `Listen`就可以再次打开，选中主机时会高亮显示：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

想要对攻陷的主机进行其他模块攻击也是一样的操作

  

**_后记_**

  

        文章有写的不正确的地方希望各位师傅斧正，因为是自己翻译的和看官方文档，难免会有差错，师傅们也可以查看官方文档进行学习，但是他们的文档确实有点离谱，视频还是`SWP`是在太难顶了：  

*   http://www.immunityinc.com/products/canvas/documentation.html
    
*   http://www.immunityinc.com/downloads/documentation/tutorials/canvas101-part1.pdf
    
*   http://www.immunityinc.com/downloads/documentation/tutorials/canvas101-part2.pdf
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在实战中我们常见的几种利用`17010`的方式都太过于危险和不稳定，这个工具的exp完美解决这个问题，利用率高，在后面，我们也会在出一份远程服务器登录的教程，希望  
最后感谢**J0师傅**打包好的工具！

各位师傅求点亮【**点赞**】【**转发**】【**在看**】

（可以的话再点点文中广告）

文中提到安装包下载

**关注【WgpSec狼组安全团队】****微信公众号**

**回复【 canvas】下载**

  

**团队师傅【北美第一突破手】的微信**  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)