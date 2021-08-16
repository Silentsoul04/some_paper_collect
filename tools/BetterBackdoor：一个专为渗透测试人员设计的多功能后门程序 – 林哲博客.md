> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/5511.html)

> 文章目录 BetterBackdoor 功能介绍 BetterBackdoor 运行机制工具要求兼容性工具下载与安装工具使用演示视频许可证协议项目地址

文章目录

*   BetterBackdoor
    *   功能介绍
    *   BetterBackdoor 运行机制
*   工具要求
    *   兼容性
    *   工具下载与安装
    *   工具使用
    *   演示视频
    *   许可证协议
*   项目地址

![](https://image.3001.net/images/20200111/1578737578_5e199faadc52f.jpg!small)

BetterBackdoor
--------------

**BetterBackdoor 是一款多功能的后门工具，广大安全研究人员可以利用 BetterBackdoor 来获取目标设备的远程访问权限。**

一般来说，后门工具会利用类似 NetCat 这样的实用工具来实现两大主要功能：使用 cmd 或 bash 来实现控制命令的远程传递并接收响应信息。这种方式实现起来很容易，但是也会受到各种因素的限制。而 BetterBackdoor 成功克服了这种限制，并引入了击键注入、获取屏幕截图、传输文件以及其他的渗透任务。

### 功能介绍

BetterBackdoor 可以直接帮助渗透测试人员创建并控制一个后门。

BetterBackdoor 创建的后门工具可以实现下列功能：

> 1、运行终端命令行控制指令
> 
> 2、运行 PowerShell 脚本
> 
> 3、运行 DuckyScripts 来注入键盘击键操作
> 
> 4、根据文件扩展名来提取文件
> 
> 5、提取 Microsoft Edge 密码以及 WiFi 密码
> 
> 6、向目标设备发送文件或接收目标设备发送过来的文件
> 
> 7、开启键盘记录器
> 
> 8、获取目标设备的屏幕截图
> 
> 9、获取目标设备的剪切板数据
> 
> 10、获取目标文件的内容（cat）

BetterBackdoor 创建的后门由一个客户端和一个服务器端组成，双方通过套接字链接通信。渗透测试的发起方需要开启一个服务器端，目标设备需要以客户端的形式跟这台服务器建立连接。连接建立成功之后，渗透测试人员就可以从服务器端向目标设备发送控制命令来管理和控制后门程序了。

### BetterBackdoor 运行机制

首先，BetterBackdoor 会创建一个 “run.jar” 文件，即后门 jar 文件，然后将其拷贝到 “backdoor” 目录中。接下来，将包含有服务器 IP 地址的文本文件添加进 “run.jar” 文件中，这里的 IP 地址是以明文形式写入的。

如果你想的话，你还可以将 Java 运行时环境拷贝至 “backdoor” 目录中，然后创建一个批处理文件 “run.bat” 来在封装的 Java 运行时环境中运行后门程序。

BetterBackdoor 支持在一个单一网络，局域网，或互联网（广域网）下工作。如果你想要在广域网上使用 BetterBackdoor，则必须进行端口转发。

若要使用广域网，必须在服务器端主机开启 TCP，并使用端口 1025 和 1026 来进行端口转发。完成此操作之后，即使目标设备和渗透发起设备位于不同的网络上，渗透测试人员也可以控制后门。

要在目标设备上启动后门，请将 “backdoor” 目录下的所有文件传输到目标设备中。如果后门文件内封装有 JRE 环境，那么直接运行 run.bat 即可，否则请运行 run.jar 文件。运行完成之后，后门便会在目标设备上启动。

工具要求
----

> 1、Java JDK >= 8
> 
> 2、生成后门与控制后门的设备必须是同一台，IP 地址必须是保持静态不变的。
> 
> 3、控制后门的设备必须关闭本机防火墙，如果在类 Unix 操作系统下运行的话，则需要使用 “sudo” 权限来运行 BetterBackdoor。

### 兼容性

BetterBackdoor 支持在 Windows、macOS 和 Linux 平台下运行，但生成的后门程序目前仅支持在 Windows 平台下工作。

### 工具下载与安装

使用下列命令将项目源码克隆至本地：

```
git clone https://github.com/ThatcherDev/BetterBackdoor.git
```

切换到项目所在的工作目录：

```
cd BetterBackdoor
```

使用 Maven 构建 BetterBackdoor，Windows 平台请运行下列命令：

```
mvnw.cmd clean package
```

Linux 和 macOS 环境请运行下列命令完成 BetterBackdoor 的构建：

```
sh mvnw clean package
```

### 工具使用

> java -jar betterbackdoor.jar

### 演示视频

> 视频地址：【点我观看】

### 许可证协议

> 本项目的开发与发布遵循 MIT 开源许可证协议。

项目地址
----

BetterBackdoor：【GitHub 传送门】

*** 参考来源：ThatcherDev，FB 小编 Alpha_h4ck 编译，转载请注明来自[林哲博客](https://www.lz80.com/category/zydq)**