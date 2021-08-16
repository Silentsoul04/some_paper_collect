> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/tMg6gPzIb6okEaPGrZ6LOQ)

**前****文介绍 ****Sqlmap 的基本用法及 CTF 实战，并且分享了请求参数设置的用法，包括设置 HTTP、POST 请求、参数分割、设置 Cookie 等。********这篇文章将详细讲解 Python 攻防，编写代码获取 Windows 主机信息，利用注册表获取主机名及 USB 历史痕迹、回收站文件等，这些知识广泛应用于电子取证、Web 渗透和攻击溯源领域，并且 USB 历史痕迹是本文的亮点。**

作者作为网络安全的小白，分享一些自学基础教程给大家，希望你们喜欢。同时，更希望你能与我一起操作深入进步，后续也将深入学习网络安全和系统安全知识并分享相关实验。此外，本文参考了 B 站、51CTO 学院和参考文献中的文章，并结合自己的经验进行撰写，也推荐大家阅读参考文献。

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.html

话不多说，让我们开始新的征程吧！您的点赞、评论、收藏将是对我最大的支持，感恩安全路上一路前行，如果有写得不好或侵权的地方，可以联系我删除。基础性文章，希望对您有所帮助，作者目的是与安全人共同进步，加油~

文章目录：

*   **一. 获取 Windows 主机信息**
    
*   **二. 获取 Windows 注册表信息**
    
    1. 注册表基本结构
    
    2. 注册表基本操作
    
    3. 获取用户账户信息
    
*   **三. 获取回收站内容**
    
*   **四. 获取 U 盘痕迹**
    
*   **五. 总结**  
    

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。

**一. 获取 Windows 主机信息**
======================

WMI(Windows Management Instrumentation) 是一项核心的 Windows 管理技术，WMI 模块可用于获取 Windows 内部信息。WMI 作为一种规范和基础结构，通过它可以访问、配置、管理和监视几乎所有的 Windows 资源，比如用户可以在远程计算机器上启动一个进程；设定一个在特定日期和时间运行的进程；远程启动计算机；获得本地或远程计算机的已安装程序列表；查询本地或远程计算机的 Windows 事件日志等等。

本文使用 Python 获取 Windows 系统上相关的信息可以使用 WMI 接口，安装调用 PIP 工具即可。

*   pip install wmi
    
*   import wmi
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GePvPJPb1xqXRNlLqUwuAAticqrhO0ibibiaJic8BnyGp6K14gH6pUzCGnNicA/640?wx_fmt=png)

下面的代码是获取 Windows 主机相关信息。

```
import wmi
import os
import socket

w = wmi.WMI()

#获取电脑使用者信息
for CS in w.Win32_ComputerSystem():
    #print(CS)
    print("电脑名称: %s" %CS.Caption)
    print("使用者: %s" %CS.UserName)
    print("制造商: %s" %CS.Manufacturer)
    print("系统信息: %s" %CS.SystemFamily)
    print("工作组: %s" %CS.Workgroup)
    print("机器型号: %s" %CS.model)
    print("")

#获取操作系统信息
for OS in w.Win32_OperatingSystem():
    #print(OS)
    print("操作系统: %s" %OS.Caption)
    print("语言版本: %s" %OS.MUILanguages)
    print("系统位数: %s" %OS.OSArchitecture)
    print("注册人: %s" %OS.RegisteredUser)
    print("系统驱动: %s" %OS.SystemDevice)
    print("系统目录: %s" %OS.SystemDirectory)
    print("")

#获取电脑IP和MAC信息
for address in w.Win32_NetworkAdapterConfiguration(ServiceName = "e1dexpress"):
    #print(address)
    print("IP地址: %s" % address.IPAddress)
    print("MAC地址: %s" % address.MACAddress)
    print("网络描述: %s" % address.Description)
    print("")

#获取电脑CPU信息
for processor in w.Win32_Processor():
    #print(processor)
    print("CPU型号: %s" % processor.Name.strip())
    print("CPU核数: %s" % processor.NumberOfCores)
    print("")

#获取BIOS信息
for BIOS in w.Win32_BIOS():
    #print(BIOS)
    print("使用日期: %s" %BIOS.Description)
    print("主板型号: %s" %BIOS.SerialNumber)
    print("当前语言: %s" %BIOS.CurrentLanguage)
    print("")

#获取内存信息
for memModule in w.Win32_PhysicalMemory():
    totalMemSize = int(memModule.Capacity)
    print("内存厂商: %s" %memModule.Manufacturer)
    print("内存型号: %s" %memModule.PartNumber)
    print("内存大小: %.2fGB" %(totalMemSize/1024**3))
    print("")

#获取磁盘信息
for disk in w.Win32_DiskDrive():
    diskSize = int(disk.size)
    print("磁盘名称: %s" %disk.Caption)
    print("硬盘型号: %s" %disk.Model)
    print("磁盘大小: %.2fGB" %(diskSize/1024**3))

#获取显卡信息
for xk in w.Win32_VideoController():
    print("显卡名称: %s" %xk.name)
    print("")
       
#获取计算机名称和IP
hostname = socket.gethostname()
ip = socket.gethostbyname(hostname)
print("计算机名称: %s" %hostname)
print("IP地址: %s" %ip)
```

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Getvpawy8EU4z9PXRz2HfRJvKAgRwO4nFnxbh2fLmKmZ3nR1KLsmmx3g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GedlibuA0eQTnCSO1ZXOoBefKYhZJraiarvE8mSRqtMHo4gRtniaTEDxleQ/640?wx_fmt=png)

**二. 获取 Windows 注册表信息**
=======================

**1. 注册表基本结构**
--------------

注册表（Registry）是 Windows 系统中一个重要的数据库，它用于存储有关应用程序、用户和系统信息。注册表的结构就像一颗树，树的顶级节点 (hive) 不能添加、修改和删除，如下图所示是 Windows 注册表的顶级节点。

![](https://mmbiz.qpic.cn/mmbiz_jpg/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeZHIzjBlnicbuz0tOhG62TsMtBfq1ibnRM3Lj2Bq2Y9yxslRjK3rcYWeQ/640?wx_fmt=jpeg)

在 C# 中对注册表进行操作，需要引用命名空间 using Microsoft.Win32。

*   **RegistryKey 类**：表示注册表中的顶级结点，此类是注册表的封装。
    
*   **Registry 类**：提供表示 Windows 注册表中的根项 RegistryKey 对象，并提供访问项 / 值的 static 方法。常用的 Registry 对象的顶级节点（蜂窝, hive）的属性如下表所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeSL4BbQVuart9rw2AcEQeu08ow4vHawmpzXriaBvrCTCXpJ8ACpiavyVw/640?wx_fmt=png)

注册表中常用的数据类型有：

*   **REG_SZ**：字符串数据的主要类型，用于存储固定长度的字符串或其他短文本值。我们在实际程序中常用这种数据类型，如果要保存布尔值时，将它表示成 0 或 1。
    
*   **REG_BINARY**：用于存储二进制数据。
    
*   **REGEXPANDSZ**：可扩展的字符串值，可以保存在运行时才解析的系统变量。
    
*   **REGMULTISZ**：以数组的格式保存多个文本字符串，每个字符串 "元素" 都以 null 字符结束。
    

**2. 注册表基本操作**
--------------

Python 注册表操作主要调用 winreg 扩展包。官方文档如下：

*   https://docs.python.org/3.0/library/winreg.html
    

基本操作函数如下：

**(1) 创建操作**

*   **winreg.ConnectRegistry(computer_name, key)** 
    
    与计算机的预定义注册表句柄建立连接
    
*   **winreg.CreateKey(key, sub_key)** 
    
    创建或打开指定的键
    

例如在 HKEYCURRENTUSER 下创建键 Eastmount，其中我们最常用的是在 \ Software 这个键下创建程序产品键，保存一些程序的配置在注册表中。如果 Software 中没有 Eastmount 键，则会先创建这个键及其子键，如果存在就不会重写。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeHgUibvI30BRXxtKaVBvQgg1Pg273DytQOqBnYb0tHCVc8FibMRaQ94iaA/640?wx_fmt=png)

运行结果如下:

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeciamUeSf82knkmlE8sshxsnYq7HG1I4UbiaqXnxEluK7kgPVTylW1IAg/640?wx_fmt=png)

**(2) 检索键值操作**

*   **winreg.QueryInfoKey(key)** 
    
    以元组形式返回键的信息
    
*   **winreg.QueryValue(key, sub_key)** 
    
    以字符串形式检索键的未命名值
    
*   **winreg.QueryValueEx(key, value_name)** 
    
    检索与打开注册表项关联的指定值名称的类型和数据
    

在 Eastmount 下面新建一个值 yxz，内容为 “hello na”，然后编写代码读取相关的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GemicU7AgW31Otx8IP03y6bZuQxZibC2eAg6XB9aXWLcE6VycrAMdYXNHw/640?wx_fmt=png)

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeGEBwnz62VUF71BGgu73kbAqHRgeufFIL2Che3X1OocgxbhmcfYoefg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeTrvqRxxdia6G7E9xeCkqfTDuAviaZkrzvS20sP9rGic1cK8WU6faPm5fQ/640?wx_fmt=png)

**(3) 创建键值操作**

*   **winreg.SetValue(key, sub_key, type, value)** 
    
    将值与指定的键关联
    
*   **winreg.SetValueEx(key, value_name, reserved, type, value)** 
    
    将数据存储在打开的注册表项 Value 字段中
    

创建键值代码如下，但会提示 PermissionError: [WinError 5] 拒绝访问错误。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Gev5NXGb6Pm9pauWkPGOJa2EYmensYyX5aQiaCHYc9ibT1tKXUo6SUkazA/640?wx_fmt=png)

**(4) 删除键值操作**

*   **winreg.DeleteKey(key, sub_key)** 
    
    删除指定的键
    
*   **winreg.DeleteValue(key, value)** 
    
    从注册表项中删除值
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Ge2hVWYEQ8yEWjhgVqfjkjUQZd17UO34awaWHa1KDCuManwojDen85zQ/640?wx_fmt=png)

成功删除键值，如下图所示。  

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Ge2wbE1HGeEkcwibjXqHNibvJaCXMdBjuJ3BMjRDSTcghq8orick86S1sUw/640?wx_fmt=png)

**(5) 其他操作**

*   **winreg.EnumKey(key, index)** 
    
    枚举打开注册表的键
    
*   **winreg.EnumValue(key, index)** 
    
    枚举打开注册表项的值
    
*   **winreg.OpenKey(key, subkey,sam=KEYREAD)** 
    
    打开指定键
    
*   **winreg.FlushKey(key)** 
    
    刷新注册表
    
*   **winreg.LoadKey(key, subkey, filename)** 
    
    在指定键下创建一个子键，并将注册信息从指定文件存储到该子键中
    

**3. 获取用户账户信息**
---------------

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GecOAaOMXxUjy8MDib5wecC5ozIjZ4WCSb81BuiatQibShw3ia4jibuYibF6Vg/640?wx_fmt=png)

获取用户名称的代码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeicvR3rsDPcHWZP5hZTiaZUwfZ1Sru8f1HXNCiaIk5Eouc8xxa1iakicdleg/640?wx_fmt=png)

执行结果如下，我们可以通过读取含有 Users 字段的数据，从而间接获取用户账户信息。

```
# encoding:utf-8
from winreg import *
import sys

usb_name = []
uid_flag = []
usb_path = []

#连接注册表根键 以HKEY_LOCAL_MACHINE为例
regRoot = ConnectRegistry(None, HKEY_LOCAL_MACHINE)

#检索子项
subDir = r"SYSTEM\CurrentControlSet\Enum\USBSTOR"

#获取指定目录下所有键的控制
keyHandle = OpenKey(regRoot, subDir)

#获取该目录下所有键的个数(0-下属键个数 1-当前键值个数)
count = QueryInfoKey(keyHandle)[0]
print(count)

#穷举USBSTOR键获取键名
for i in range(count):
    subKeyName = EnumKey(keyHandle, i)
    subDir_2 = r'%s\%s' % (subDir, subKeyName)
    #print(subDir_2)

    #根据获取的键名拼接之前的路径作为参数 获取当前键下所属键的控制
    keyHandle_2 = OpenKey(regRoot, subDir_2)
    num = QueryInfoKey(keyHandle_2)[0]
    #遍历子键内容
    for j in range(num):
        subKeyName_2 = EnumKey(keyHandle_2, j)
        #print(subKeyName_2)
        result_path = r'%s\%s' % (subDir_2, subKeyName_2)

        #获取具体键值内容并判断Service为disk
        keyHandle_3 = OpenKey(regRoot, result_path)
        numKey = QueryInfoKey(keyHandle_3)[1]
        for k in range(numKey):
            #获取USB名称
            name, value, type_ = EnumValue(keyHandle_3, k)
            if(('Service' in name) and ('disk'in value)):
                value,type_ = QueryValueEx(keyHandle_3,'FriendlyName')
                usb = value
                uid = subKeyName_2
                path = "USBSTOR" + "\\" + subKeyName + "\\" + subKeyName_2
                print(usb)
                print(uid)
                print(path)                
                print("")
    
#关闭键值
CloseKey(keyHandle)
CloseKey(regRoot)
```

**三. 获取回收站内容**
==============

为什么我们要去获取回收站文件呢？因为很多情况下调查取证需要获取远程目标的历史痕迹，回收站是重要的一个目标。在 Windows 操作系统中，回收站是一个专门用来存放被删除文件的特色文件夹。

在使用 FAT 文件系统的 Windows98 系统中，回收站目录通常是 C:\Recycled；在 Windows NT2000、Windows XP 在内支持的 NTFS 操作系统中，C:\Recycler；在 Windows Vista 和 Windows7 中，回收站目录是 C:\$Recycle.Bin。如下图所示，回收站中包含两个文件，分别位于桌面和 D 盘目录。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeSnmk4NTgIh3KgLC7x7VUUcOr0C2nSfWnhY0Ik2g60klibibecZmCA9Gg/640?wx_fmt=png)

**第一步，检测回收站目录是否存在。**

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeibKg0MGGhKeXichlibsTt3KU8M4IiaciccwopuibmlvrfVUSm3LET2EiaTnkg/640?wx_fmt=png)

Windows10 操作系统输出结果如下所示：

*   C:\$Recycle.Bin\
    

**第二步，找到回收站之后，检测其中的内容，如下图所示，字符串 SID 与用户账户名是对应的，比如 1001 结尾的 SID。**

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeuW8ibiaLmxNwBtDkGeBAxmR3N1zo0FjbIjhck5uHyqjIvk7kMiaEyhsww/640?wx_fmt=png)

**第三步，编写代码获取回收站文件夹所在目录。**

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeyNc8kcSwjzj1J1veVBz6X0A5wErNa7wDxawmgXVdpGcBAkgKGjicwhg/640?wx_fmt=png)

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeibuNDDzibSpAqpVQ30L6f6wic5ZIRJicIXPSib5P4ux2OEuJZHAxHiaSEIyg/640?wx_fmt=png)

**第四步，用 python 将用户的 SID 关联起来，使用 Windows 注册表将 SID 转化为一个准确的用户名。**

*   通过检查 Windows 注册表键值
    
*   HKEYLOCALMACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\ProfileImagePath
    

编写一个函数来将每一个 SID 转化为用户名，这个函数将打开注册便检查 ProfileImagePath 键值，找到其值并从中找到用户名。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Ge0UoD7iaLJFeia5IyMHG7tmE98SLWqdhialsm1VP4Rox63csrgulxk3MGA/640?wx_fmt=png)

如下图所示，用户名为 “xiuzhang”。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Ge1qb5DsCw5dC5XxMHkricmSuxX3tDkicRhpAAMnLgYORN2dGJ845sGUaA/640?wx_fmt=png)

**第五步，获取回收站所有内容，完整代码如下。**

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeibHnVFicVP1hnfFswIzkf31ZgE1NPk243QHn5TBgSKmTN02Bp0HiaEvRw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GenQO8apz9ia7tfMGFjMZuPPSwIL7FgUAicRcXY0ZbkbDV3MuppFhaRibdg/640?wx_fmt=png)

输出结果如下图所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeCmOhqAlvVhWTIQkZgLSPpACVnaUvFib8mtvGDyjSosRx5xQrKF2LOqw/640?wx_fmt=png)

对应的回收站内容如下，但非常可惜获取的值无法对应，why？后续作者会继续深入挖掘。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Geehs6YnM4hib9kHxOWOfh8WRDN5iboOrIa6LKeWKMmWbxeyaJIjQCcLRA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Gen9e8e3YC29v6R7covJNJxmHebUuDz0ZRwR1qSLmmwpEDjzED5C6G4w/640?wx_fmt=png)

**如果我们想把文件删除到回收站，又怎么解决呢？Python 删除文件一般使用 os.remove，但这样是直接删除文件，不删到回收站的，那么想删除文件到回收站怎么办？**

(1) 安装 pypiwin32 扩展包（含 win32api）。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeVEc1bkQTaKXob7rOZKDWUpeORP6UUV5Frc2ZYuiaWUCrNCicB3amI0Dw/640?wx_fmt=png)

(2) 调用 SHFileOperation 函数实现删除文件至回收站。

> 在 Windows 的 shellapi 文件中定义了一个名为 SHFileOperation 的外壳函数，用它可以实现各种文件操作，如文件的拷贝、删除、移动等，该函数使用起来非常简单，它只有一个指向 SHFILEOPSTRUCT 结构的参数。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeV9wzwmlsDhW3MSicu48nXqhoj67qE9vuOC9xpamO6DiaZJHD6xtpSiaZQ/640?wx_fmt=png)

最终效果如下图所示，可以看到 require.rb 文件被成功删除。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GegpWVakCqZYcMXT7oGCOFiavJvyHkS77lNB12icLwwSnCLZ1TGRlHD1Gw/640?wx_fmt=png)

注意，注册表操作可能会遇到 “PermissionError: [WinError 5] 拒绝访问” 问题，我们需要设置 Python.exe 用户名完全控制，并且用管理员方式打开即可解决。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeKqKwbEFZvPth8W66a7O9WaJicoxEHGv5dOuW2AW1pvU8oNDVTPBUcIA/640?wx_fmt=png)

**四. 获取 U 盘痕迹**
===============

**这部分我认为是本文最大的亮点。**在 Windows 系统中，当一个 USB 移动存储设备插入时，就会在注册表中留下痕迹。当移动设备插入计算机时，即插即用管理器 PnP(Plug and Play)接受该事件，并且在 USB 设备的固件 (Firewre information) 中查询有关该设备的描述信息(厂商、型号、序列号等)。当设备被识别后，在注册表中创建一个新的键值：

*   HKEYLOCALMACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR
    

在这个键值下，会看到类似下面的结构子键，该子键代表设备类标示符，用来标识设备的一个特定类。

*   Disk&Ven###&Prod###&Rev_###
    

其中，子键中 "###" 代表区域由 PnP 管理器依据在 USB 设备描述符中获取的数据填写。如下图所示：

*   Disk&Venaigo&ProdMiniking&Rev_8.07 是 Device class ID
    
*   Q0UKCH37&0 是 Unique instance ID(UID)
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeqNqv3E152icT2XEbiayzdicibia0WYvBJP7Gru4MNmvl3Ivh6q0E3v3MavA/640?wx_fmt=png)

注意需要判断 Service 值为 disk，即为磁盘的子项，光盘为 cdrom。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeEwibIFRzrTH8ibwlr1lNK9aapeuwvJS5FLBFoibn9ibnRzpuN9BOSokb9Q/640?wx_fmt=png)

如果使用 UVCView 工具可以看见 USB 设备描述内容，其中的信息都是相互对应的。设备类 ID 一旦建立，就需要建立一个特定唯一的 UID，它可以把具有同一设备类标识的多个存储设备区分。

**完整实现代码如下：**

```
# encoding:utf-8
from winreg import *
import sys
usb_name = []
uid_flag = []
usb_path = []
#连接注册表根键 以HKEY_LOCAL_MACHINE为例
regRoot = ConnectRegistry(None, HKEY_LOCAL_MACHINE)
#检索子项
subDir = r"SYSTEM\CurrentControlSet\Enum\USBSTOR"
#获取指定目录下所有键的控制
keyHandle = OpenKey(regRoot, subDir)
#获取该目录下所有键的个数(0-下属键个数 1-当前键值个数)
count = QueryInfoKey(keyHandle)[0]
print(count)
#穷举USBSTOR键获取键名
for i in range(count):
    subKeyName = EnumKey(keyHandle, i)
    subDir_2 = r'%s\%s' % (subDir, subKeyName)
    #print(subDir_2)
    #根据获取的键名拼接之前的路径作为参数 获取当前键下所属键的控制
    keyHandle_2 = OpenKey(regRoot, subDir_2)
    num = QueryInfoKey(keyHandle_2)[0]
    #遍历子键内容
    for j in range(num):
        subKeyName_2 = EnumKey(keyHandle_2, j)
        #print(subKeyName_2)
        result_path = r'%s\%s' % (subDir_2, subKeyName_2)
        #获取具体键值内容并判断Service为disk
        keyHandle_3 = OpenKey(regRoot, result_path)
        numKey = QueryInfoKey(keyHandle_3)[1]
        for k in range(numKey):
            #获取USB名称
            name, value, type_ = EnumValue(keyHandle_3, k)
            if(('Service' in name) and ('disk'in value)):
                value,type_ = QueryValueEx(keyHandle_3,'FriendlyName')
                usb = value
                uid = subKeyName_2
                path = "USBSTOR" + "\\" + subKeyName + "\\" + subKeyName_2
                print(usb)
                print(uid)
                print(path)                
                print("")
#关闭键值
CloseKey(keyHandle)
CloseKey(regRoot)
```

输出的 USB 记录键名如下图所示，包括三星、金盾、等 U 盘或硬盘型号。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeX8tqAUQWkxSqVd0tQoHrTINYicia3ibTsreJPfC4ic8EgibGgB81L3ReHsA/640?wx_fmt=png)

其中对应的注册表信息如下图所示，FriendlyName 即是输出的 USB 名称 “Kingston DataTraveler 2.0 USB Device”，UID 序号为 “C860008862F1EE501A0F0105&0”，搜索的 Service(服务) 为 disk(磁盘)的选项。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GegB1GSS41WJJw6fA4SwONGrq8Aiac1G9RR7msdricP0E35pB8QFjpzyYQ/640?wx_fmt=png)

**简单总结：**

个人感觉这方面的资料真心很少，文章博客也少，所以看起来操作似乎很简单，但真正实现起来还是令人深思的。然后就是其实存储 USB 记录的还有很多键值，如

*   HKEYLOCALMACHINE\SYSTEM\CurrentControlSet\Enum\USB 该键值中能看到厂商号 (VID)、厂商产品号 (PID)，还有 LocationInformation(端口号) Port#0001.Hub#0005 等。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeqyGO2eOia5BCOwdTpk3HQa7F0O3HAM5U98WwE7EZbsJXX9HC3MHAHFQ/640?wx_fmt=png)

*   HKEYLOCALMACHINE\SYSTEM\CurrentControlSet\Control\DeviceClasses 该键值下有两个设备类：{53F56307-B6BF-11D0-94F2-00A0C91EFB8B}{53F5630d-B6BF-11D0-94F2-00A0C91EFB8B}，可以通过他们获取 USB 最后接入系统时间。
    

接下来我想要完成的就是如何把这些键值联系起来，似乎要通过 Dictionary，同时怎样获取时间，怎样正确删除这些信息都值得深究。

**五. 总结**
=========

**这篇文章希望您喜欢。同时感觉自己要学习的知识好多，尤其是论文，如数家珍加油。这个世界厉害的人太多太多，作为初学者，我们可能有差距，不论你之前是什么方向，是什么工作，是什么学历，是大学大专中专，亦或是高中初中，只要你喜欢安全，喜欢渗透，就朝着这个目标去努力吧！有差距不可怕，我们需要的是去缩小差距，去战斗，况且这个学习的历程真的很美，安全真的有意思。但切勿去做坏事，我们需要的是白帽子，是维护我们的网络，安全路上共勉。**

天行健，君子以自强不息。  
地势坤，君子以厚德载物。

最后希望基础性文章对您有所帮助，作者也是这个领域的菜鸟一枚，希望与您共同进步，共勉。学安全一年，认识了很多安全大佬和朋友，希望大家一起进步。这篇文章中如果存在一些不足，还请海涵。作者作为网络安全和系统安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，深知自己很菜，得努力前行。编程没有捷径，逆向也没有捷径，它们都是搬砖活，少琢磨技巧，干就对了。什么时候你把攻击对手按在地上摩擦，你就赢了，也会慢慢形成了自己的安全经验和技巧。加油吧，少年希望这个路线对你有所帮助，共勉。

前文分享（下面的超链接可以点击喔）：

*   [[网络安全] 一. Web 渗透入门基础与安全术语普及](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483786&idx=1&sn=d9096e1e770c660c6a5f4943568ea289&chksm=cfccb147f8bb38512c6808e544e1ec903cdba5947a29cc8a2bede16b8d73d99919d60ae1a8e6&scene=21#wechat_redirect)
    
*   [[网络安全] 二. Web 渗透信息收集之域名、端口、服务、指纹、旁站、CDN 和敏感信息](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483849&idx=1&sn=dce7b63429b5e93d788b8790df277ff3&chksm=cfccb104f8bb38121c341a5dbc2eb8fa1723a7e845ddcbefe1f6c728568c8451b70934fc3bb2&scene=21#wechat_redirect)
    
*   [[网络安全] 三. 社会工程学那些事及 IP 物理定位](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483994&idx=1&sn=1f2fd6bea13365c54fec8e142bb48e1d&chksm=cfccb297f8bb3b8156a18ae7edaba9f0a4bd5e38966bdaceeff03a5759ebd216a349f430f409&scene=21#wechat_redirect)
    
*   [[网络安全] 四. 手工 SQL 注入和 SQLMAP 入门基础及案例分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484068&idx=1&sn=a82f3d4d121773fdaebf1a11cf8c5586&chksm=cfccb269f8bb3b7f21ecfb0869ce46933e236aa3c5e900659a98643f5186546a172a8f745d78&scene=21#wechat_redirect)
    
*   [[网络安全] 五. XSS 跨站脚本攻击详解及分类 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484381&idx=1&sn=a1d459a7457b56b02e217f39e5161338&chksm=cfccb310f8bb3a06442b001fc7b38a0363b9fbd4436f450b0ce6fa2eeb5c796fc936ceb5d6fa&scene=21#wechat_redirect)
    
*   [[网络安全] 六. XSS 跨站脚本攻击靶场案例九题及防御方法 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485174&idx=1&sn=245b812489c845e875cf4bc4763747b7&chksm=cfccb63bf8bb3f2d537f36093de80dbeed5a340b141001d3ef8a9ac9d6336e0aaf62b013a54c&scene=21#wechat_redirect)
    
*   [[网络安全] 七. Burp Suite 工具安装配置、Proxy 基础用法及暴库入门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485381&idx=1&sn=9a0230cf22eba0a24152cb0e73a37224&chksm=cfccb708f8bb3e1ecf68078746521191921f41d19a0b82cb3f097856dad7a85c4d9c34750b3f&scene=21#wechat_redirect)
    
*   [[网络安全] 八. Web 漏洞及端口扫描之 Nmap、ThreatScan 和 DirBuster](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485437&idx=1&sn=2a7179464207fa68b708297ec0db6f00&chksm=cfccb730f8bb3e2629edb5ca114de79723e323512be9538a4d512297f8728a3a9d7718389b60&scene=21#wechat_redirect)
    
*   [[网络安全] 九. Wireshark 安装入门及抓取网站用户名密码 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485465&idx=1&sn=8e7f1f5790bfe754affe0599a3fce1ee&chksm=cfccb8d4f8bb31c2ca36f6467d700f4e4d7821899a6d5173ac0b525f0f6227c8392252b5c775&scene=21#wechat_redirect)
    
*   [[网络安全] 十. Wireshark 抓包原理、ARP 劫持、MAC 泛洪及数据追踪](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485551&idx=1&sn=15f00e14f4376e179a558444de8ef0a5&chksm=cfccb8a2f8bb31b456499a937598e750661841b5ca166a12073e343a049737fa3131fd422dc5&scene=21#wechat_redirect)
    
*   [[网络安全] 十一. Shodan 搜索引擎详解及 Python 命令行调用](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485599&idx=1&sn=0c60c042911fc79287417c2385550430&chksm=cfccb852f8bb3144a89f6b0d0df6c185a208aa989d98f8c7e3b7d741dedc371b3ecb4e70a747&scene=21#wechat_redirect)
    
*   [[网络安全] 十二. 文件上传漏洞 (1) 基础原理及 Caidao 入门知识](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485787&idx=1&sn=0c75cf81c4234031273bced4dff0b25c&chksm=cfccb996f8bb3080fe9583043b43665095fd6935a4147a2bb0d1ab9b91a6cde99da4747c5201&scene=21#wechat_redirect)
    
*   [[网络安全] 十三. 文件上传漏洞 (2) 常用绕过方法及 IIS6.0 解析漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485833&idx=1&sn=a613116633338ca85dfd1966052b0b02&chksm=cfccb944f8bb305296a32dac7f0942e727d66dc9f710bfb82c3597500e97d39714ecd2ed18cf&scene=21#wechat_redirect)
    
*   [[网络安全] 十四. 文件上传漏洞 (3) 编辑器漏洞和 IIS 高版本漏洞及防御](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485871&idx=1&sn=e6d0248e483dea9616a5d615f852eccb&chksm=cfccb962f8bb3074516c1ef8e01c7cb00a174fa5b1a51de3a49b13fd8c7846deeaf6d0e24480&scene=21#wechat_redirect)
    
*   [[网络安全] 十五. 文件上传漏洞 (4)Upload-labs 靶场及 CTF 题目 01-10](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488340&idx=1&sn=5b7bf5602294586f819340bd6190a34d&chksm=cfcca399f8bb2a8f746fc09c7142facc8ea17c008ba46dee423b90ff6abb3cd4486edf52d201&scene=21#wechat_redirect)
    
*   [[网络安全] 十六. 文件上传漏洞 (5) 绕狗一句话原理和绕过安全狗](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488396&idx=1&sn=67c1b13f041040c09c236bba99edfe0a&chksm=cfcca341f8bb2a5729778490db7441a4ddfdfa05dcc5f6322b4860db7780056f9f05f5bc0b3d&scene=21#wechat_redirect)  
    
*   [[网络安全] 十八. Metasploit 技术之基础用法万字详解及 MS17-010 漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488255&idx=1&sn=28b1f54fd420a0145cb95b842a36c567&chksm=cfcca232f8bb2b243bf4cbf5c1741c6af2c1fc666985d34b4f6b4a6ee3161d18975bb5ea18fc&scene=21#wechat_redirect)
    
*   [[网络安全] 十九. Metasploit 后渗透技术之信息收集和权限提权](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488639&idx=1&sn=dddd54eb0ba7cfdf71113a1f4a5c6548&chksm=cfcca4b2f8bb2da44c975ca12f16b4b76af351be4711ac7e77ca8622450a15c3af0172be3f9e&scene=21#wechat_redirect)
    
*   [[网络安全] 二十. Metasploit 后渗透技术之移植漏洞、深度提权和后门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488738&idx=1&sn=8106362219d99ae6deb8aeca1f6b1dff&chksm=cfcca42ff8bb2d397c44b839700d92fd22e4ac60c403b96cba734bc523cb258dbd0db5309952&scene=21#wechat_redirect)
    
*   [[网络安全] 二十一. Chrome 密码保存渗透解析、Chrome 蓝屏漏洞及音乐软件漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488883&idx=1&sn=65c362cc4c3958aa747716d17b29eeb3&chksm=cfcca5bef8bb2ca895525a1964425d1dfe74001a33e3b59b18bf902539cfc4d941dd96c33863&scene=21#wechat_redirect)
    
*   [[网络安全] 二十二. Powershell 基础入门及常见用法 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489093&idx=1&sn=216374f1db9af3e1bb4f9431b66237a3&chksm=cfcca688f8bb2f9e9fc25c1d1e21d3bceae0a9ff026f57e6e6df2ffa20597aa8356c15ea2280&scene=21#wechat_redirect)
    
*   [[网络安全] 二十三. Powershell 基础入门之常见语法及注册表操作 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489150&idx=1&sn=969db0e97868fe64fb03776b77bf7d13&chksm=cfcca6b3f8bb2fa56d2c9e4b2bdbd5abcc04ee724ee6cd2abbb059fca9ae65d6595ca98c2624&scene=21#wechat_redirect)
    
*   [[网络安全] 二十四. Web 安全学习路线及木马、病毒和防御初探](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489258&idx=1&sn=0fcfeb9555982c10eca90d2a78c5b58f&chksm=cfcca627f8bb2f315c8b089fcbeded22ab3515980a618857349e606e049d6a8d73bf79743ffe&scene=21#wechat_redirect)
    
*   [[网络安全] 二十五. 虚拟机 VMware+Kali 安装入门及 Sqlmap 基本用法](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489455&idx=1&sn=3835d420386dfb1df32f0f550f21b0d8&chksm=cfcca762f8bb2e7493a99415b19145f8c35b9af19dd6904511d525485fb9f98e310eb12e3054&scene=21#wechat_redirect)
    
*   [[网络安全] 二十六. SQL 注入之揭秘 Oracle 数据库注入漏洞和致命问题（Cream 老师）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489552&idx=1&sn=9336824ddebde336766c51a3674cf764&chksm=cfcca8ddf8bb21cbd1e80f08b012f59cf3d1661e756cd44b335671f5d13392752e0f56f041c1&scene=21#wechat_redirect)
    
*   [[网络安全] 二十七. Vulnhub 靶机渗透之环境搭建及 JIS-CTF 入门和蚁剑提权示例 (1)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489805&idx=1&sn=89a3970bea60cc4792a3288b9250523d&chksm=cfcca9c0f8bb20d68828a34fcee212aabf869b3937d2cc12f3cf768dda5d1175fcce41ba32cf&scene=21#wechat_redirect)
    
*   [[网络安全] 二十八. Vulnhub 靶机渗透之 DC-1 提权和 Drupal 漏洞利用 (2)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247490070&idx=1&sn=f7060d391eae4c91901efb22d0f4f7ae&chksm=cfccaadbf8bb23cd6d87f2bf0095232f5872519fc04413bbcb3d1451a1a593d2d8f42e3df8ab&scene=21#wechat_redirect)
    
*   [[网络安全] 二十九. 小白渗透之路及 Web 渗透技术总结（i 春秋 YOU 老师）](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247490243&idx=1&sn=d88c090287117977d12db27dca221f95&chksm=cfccaa0ef8bb23185205d68f1080c0bb3a4db9f77b39fa5fdaeb7f261fe022b2f7c0ec21e6f6&scene=21#wechat_redirect)
    
*   [[网络安全] 三十. Vulnhub 靶机渗透之 bulldog 信息收集和 nc 反弹 shell(3)](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247490908&idx=1&sn=d11ddda2cf0691720a1c945fb6164499&chksm=cfccad91f8bb2487b0ec754d66bd803a797c8fa0e222993b029b4c02e35d899561279abf9b11&scene=21#wechat_redirect)
    
*   [[网络安全] 三十一. Sqlmap 基础用法、CTF 实战及请求参数设置万字详解](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247491106&idx=1&sn=1133bb304f172d75671a3b0dedfa333f&chksm=cfccaeeff8bb27f9eddf5b8c4133624c28540b16212721ec67579f4f149f239a492fd8ace46d&scene=21#wechat_redirect)
    
*   [网络安全] 三十二. Python 攻防之获取 Windows 主机信息、注册表、U 盘痕迹和回收站 (1)  
    

最后，真诚地感谢您关注 “娜璋之家” 公众号，也希望我的文章能陪伴你成长，希望在技术路上不断前行。文章如果对你有帮助、有感悟，就是对我最好的回报，且看且珍惜！再次感谢您的关注，也请帮忙宣传下“娜璋之家”，哈哈~ 初来乍到，还请多多指教。  

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6Ge2bH4MMXJ24KrXZz13Cv8OLk3GZYYggZgxDDUgEkTbGK0UiaopPG6tOg/640?wx_fmt=png)

**日日思君不见君，共看夕阳落。**

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRNqAfkMicNnZwWVyLokAM6GeLK9UKQIzbyGJduYLYL1oudKy3jQCQCrJIVQq60aNCZ9lMgW1OoQ5cg/640?wx_fmt=png)

(By: Eastmount 2021-05-11 夜于武汉)

**参考文献：**

*   《Python 绝技运用 Python 成为顶级黑客》TJ.O Connor
    
*   Python wmi 模块获取 windows 内部信息 - mingerlcm
    
*   SHFileOperation 的用法 - xiaodai0
    
*   https://blog.csdn.net/Eastmount/article/details/108020041
    
*   C# 系统应用之注册表使用详解 - Eastmount
    
*   https://docs.python.org/2/library/winreg.html