> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ZgNpP2olfVgSx9dP41XJng)

dll 劫持
------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsHTxSuNelBSvvEU4Yrf4OE74Zd6y4PCHowcibDDaLXibASGjd2PKS0ZjA/640?wx_fmt=png)

1.dll 劫持产生条件
------------

```
1.dll能否被劫持：不再'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs'注册表中2.其dll是EXE程序首先加载的DLL，而不是依赖其他DLL加载的。3.DLL确实被加载进内存中
```

2. 判断 dll 是否可以劫持。
-----------------

### 2.1 手动方法

利用进程查看软件，查看 dll 是否存'KnownDlls'注册表中。进程查看工具：ProcessExplorer/ProcessMonitor / 火绒剑

```
ProcessExplorer下载地址：https://docs.microsoft.com/zh-cn/sysinternals/downloads/process-explorer
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsicbsibDVJ3CwXVofVh7ibujToHwiavpibryseMNdn9MD3wcTGH0NGn7DCWA/640?wx_fmt=png)  

```
ProcessMonitor下载地址：https://docs.microsoft.com/zh-cn/sysinternals/downloads/procmon
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXs5JXCDx1tbKRtbdwJVNRCoCA28UaxViaLNLJGgjAibmnXTt4QNcAexMNw/640?wx_fmt=png)

```
火绒剑<br style="box-sizing: border-box;">
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsTXewCXAIPic0JRL0tibwFtRr1gSfLhbuCQibFslm9uHzdcqZHxfeicIubA/640?wx_fmt=png)  

根据进程查看的 dll 和注册表进行对比  

```
win7及以上：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXstA5H2If18Eb7ENfd3hPwfd8XHib7rmLnkWnaFXRfibpzFOSFUDtjDMibg/640?wx_fmt=png)  

```
notepad为例：6版本之前的SciLexer.dll存在dll劫持。注册表中没有SciLexer.dll。进程notepad调用到了SciLexer.dll。说明SciLexer.dll可能存在dll劫持。
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsfwjHaV8OWjibof1VFQ8Pic6CCTcvv4AMZ3Lu2XR5dfphiccaqicsCxXXCA/640?wx_fmt=png)  

2.2 自动审计

常见的几种工具：

```
1.Ratter(虚拟机中出错不知道为啥）https://github.com/sensepost/rattler/releases
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXs7I2CRSFeGDgyA1PoV1WbibX5eUFzWX6UX0L62zsb6Uc95BibcthmXP8g/640?wx_fmt=png)  

```
2.DLL Hijack Auditor
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXssHZpBdVlftapo7p48CCYGbPsQCLe8K64pyWlVibpCXC4tqKYgCDNd7g/640?wx_fmt=png)  

```
3.dll_hijack_detecthttps://github.com/adamkramer/dll_hijack_detect/releases
```

```
4.ChkDllHijackhttps://github.com/anhkgg/anhkgg-tools
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsq7SjDBIkmG0kv49K0O6Fena6icGxBW4j0YwC97DqGdQayk22ib0hxrjg/640?wx_fmt=png)

2.3 测试  

利用自动化验证 dll 劫持发现 sxs.dll 可能存在劫持

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsgro2kmY5ASVEFVGLD0fYzJ3PtfS380Lb3jNElicqzrjp7icjicAlic2icjw/640?wx_fmt=png)

msf 生成 dll，弹出计算机。

```
msfvenom -p windows/exec CMD = calc.exe EXITFUNC=thread -f dll -o sxs.dll
```

### 1. 可以替换可能存在 dll 劫持的文件。

### 2. 可以使用 dll 注入工具, 进行劫持文件

2.1dll 注入工具

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsgJmySM4o7MuM4TpAZMBpaxg5F1AYpbgApkkYEhkILgdNA0XVCkTJUw/640?wx_fmt=png)

2.2 使用 InjectProc 注入

```
InjectProc --dll注入工具https://github.com/secrary/InjectProc/releasesInjectProc.exe dll_inj joker.dll notepad.exe
```

注入成功

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsBDyyiaG83XpnqTuW8jYib4jq0XubVqq6aK3WGt6fwKFEnotXE688SL1g/640?wx_fmt=png)

### 3.dll 注入 + 重新打包

```
采用方法：shellcode--dll--pe导入dll--nsis重新打包
```

### 3.1cs 生成 shellcode

利用攻击 -> 生成后门 -> 语言类后门 ->C 语言![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXseO15WQD952wwQE7ib53kQPnLm7wSuicfL1vUNdGLt8oq9icdLcIibOMNtw/640?wx_fmt=png)

### 3.2 利用 DLL 注入攻击工具

```
利用DLL注入攻击工具把shellcode生成dll文件。会在同目录下生成conf.inf 和wwwcomw.dll两个文件
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsth665LGyibehzWhFjs7XSlBNbII5F6ACG2BR8CmS2GB9UdqOQuHExiaQ/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsPGoML3tKNAhbyicRrgBYOHS5H3ibspcTr7Qic1sgKLxplabYicicvsqdosA/640?wx_fmt=png)

### 3.3 利用 PE 工具把 DLL 导入 EXE  

```
```把上述生成的文件放到需要劫持的目录下。采用PE查看器，把上述shellcode生成的dll中的函数导入到exePE表中。把EXE放入到PE查看中-->选择函数-->随便选择一个右键Add New ImportDLL选择->选择函数->添加到列表->输入表列表中选中刚刚添加到内容，最后点击添加即可。```
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsjDf1nnn5jepQ0HxTDqSAXQrlhLzCFc8iaZRHTGRqiaCqjXGiaJ7dl2ruw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsSbZaQF7vyH0304tuMjfV31N5N6jQKI89XD5FyUDjDzGveibUMwmXR8w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsedy79RT7n3CBKibenogW9ic7FERvia5ib9ZTdVcZ6QsZaklH3zib2kfbKmA/640?wx_fmt=png)

### 3.4 上线  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXstpZgEN8rDtNVdU2NYRWACcMiax9JVzXibWlccktRHgaSGle4gvcBLm7w/640?wx_fmt=png)  

### 4 nsis 重新打包

```
上述完成后，本地劫持完成，这里使用nsis重新打包，进行钓鱼。<br style="box-sizing: border-box;">
```

### 4.1 准备工作

### 源安装包

源文件的 ico 图标提取。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsX0IZ7x4zDtSnXrbDgUh0sH3NwKDuDdRuZO5WBKkSyibXUibpJTBduCxw/640?wx_fmt=png)

4.2 打包  

```
选择可视化脚步编辑器<br style="box-sizing: border-box;">
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXssGyjeMiajZqMTqUIQkhrJxWW7XibOE9TxO57ia5Mico1PJ5tUZicic2vvLjQ/640?wx_fmt=png)  

```
安装需求填写<br style="box-sizing: border-box;">
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXssZQsgsvcKG6JHCoqOlInibbZuZjW84Uj8icVhdaXJmSSEr6qwbA6IfUw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsljxrTPWUH3GVYAQE4xCCvWtKbhwqqTHa3UG5Kq5yu8mM3BZbiaxWlibw/640?wx_fmt=png)

```
应用程序默认目录，本地安装一次最好，看一下本地默认安装的目录例如：C:\Program Files (x86)\VPN（本地）然后修改本地安装了，防止冲突，设置成了VPN1
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsia2azMQHLdbww3zsmcCUEhEcuG0uSzAIoDKwxA9tkgyXWklTiaicE5JLw/640?wx_fmt=png)  

```
选择需要打包的目录，把每个字目录选中。<br style="box-sizing: border-box;">
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXs7hSibBvCBKDiahjB0TkLPz2TrzUtdsD20dfBR2wksoO4yJKHibpb7seHw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsOSc0AccN8eo0RYhelRAELYC1TRbwFfRkXAgo2RDBXYaINJzhulYHow/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsibdyt1wuURdBykORhVubWkoVHOiaWRzyKgzojZcekdJfgaMByoNkmgnQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsj6DmHR9iaYUOXEgLkCOl60eI15C8EfpcUS3chgaeicjAoCpaKkVUt4dQ/640?wx_fmt=png)

编译成功

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsDrrVkafMiceV8EskqTicT2DHPbyBeTiaBJkDVpTqxz3cu1PmDsmLj5Z6Q/640?wx_fmt=png)

```
打包成功<br style="box-sizing: border-box;">
```

```
对比一下，下面为最新打包，上面为源文件，文件大小差距。除了没有数字签名<br style="box-sizing: border-box;">
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsLEOOARiaaHvX1kq9UPsulvANc3iasbvmQdwtjBs7fxEniaia9zZtAmXGlw/640?wx_fmt=png)

打包之类成功安装。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXstxOTniaicWzVLg4eY2e2RhOtExes6ZRTk0cEdgem0NVBaSHh2ZQb0StA/640?wx_fmt=png)

```
成功上线。<br style="box-sizing: border-box;">
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSkBENehqaiaroq7CQQhy4kXsq0OIxwcRoreH6QPHbAyBehdp3QicVAp9a3cR1JRslpQPrp82GbtIOiaQ/640?wx_fmt=png)  

```
默认可过火绒，360动态被杀，需要在dll生成进行免杀。
```