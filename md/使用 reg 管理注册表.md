> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/duYwm3VtlCmyO7UcDTiGUg)

**目录**

注册表

注册表结构

reg

增 

删

改

查

注册表  

======

Windows 注册表就相当于 Windows 系统的数据库，系统和软件的配置信息放在注册表里面。如果注册表出现了问题，可能导致系统崩溃。我们平时是使用 regedit.exe 命令来使用图形化界面管理注册表的。而在很多时候，使用图形化界面管理注册表很麻烦。所以今天介绍一种使用纯命令行的工具 (reg.exe) 来管理注册表。使用 reg.exe 可以对注册表进行添加、删除、修改、查看等操作。

注册表结构
-----

注册表有四个关键术语：键、值、值类型、数据

值的类型有六种，分别为：

*   REG_BINARY
    
*   REG_DWORD
    
*   REG_EXPAND_SZ
    
*   REG_MULTI_SZ
    
*   REG_QWORD
    
*   REG_SZ
    

reg
---

reg(控制台注册表编辑器)，默认文件路径为：C:\Windows\System32\reg.exe 。执行 reg /? 可以查看 reg 的帮助。如果使用 reg 对注册表进行增删改查的话，需要管理员权限。

```
QUERY
ADD
DELETE
COPY
SAVE
LOAD
UNLOAD
RESTORE
COMPARE
EXPORT
IMPORT
FLAGS
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIAGiabiaMiaVibFJ4EficBErTiciapVdc82pd661a3GWzeE3E9T2E38Ga47yqw/640?wx_fmt=png)

使用 reg 修改注册表，有几个简写

```
HKCR    HKEY_CLASSES_ROOT
HKCU   HKEY_CURRENT_USER
HKLM    HKEY_LOCAL_MACHINE 
HKU     HKEY_USERS
HKCC    HKEY_CURRENT_CONFIG
```

### 增

使用如下命令在 HKEY_CURRENT_USER 下新建一个 test 键，值为 hello，值的类型为 REG_SZ 。

/v 后面跟需要创建的值的名称，/t 后面是值的类型，/d 后面是这个值的数据，/f 是强制不提示

```
reg add hkcu\test /v hello /t REG_SZ /d "this is test!" /f
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIu5KqibjM2YY8SyMkA7BrzibfMXGzJBv36B04OW2F5gic8CjrKoaunVODg/640?wx_fmt=png)

### 删

```
删除HKEY_CURRENT_USER下的test键的hello值
reg delete hkcu\test /v hello /f

删除HKEY_CURRENT_USER下的test键
reg delete hkcu\test /f
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIyyQkvBeicHXZbhrz26ksA2mOAMudkkSiaHuJD7fU3j4lAd0BSjJCZQuA/640?wx_fmt=png)

### 改

修改 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\ 的 Terminal Server 键 的 fDenyTSConnections 值的数据为 0 (0x00000000)。

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI7A7QWKezDJEg6awG6CHPQicYNqVsHrI8Ivd8IhwQjgtQgQOr3WXr1Ww/640?wx_fmt=png)

### 查

```
reg query HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEII2zChD4BWsmFGXtnOAvyxcHvWJiaaHI89aMRUCb4IMp8Va4gkKdriaTg/640?wx_fmt=png)