> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/esAzEiqlVx5m51aE-_CgIA)

@byt3bl33d3r 师傅在 twitter 上发布了他最新的一个 GitHub 仓库，名为 OffensiveNim。该仓库使用介绍了一种新的编译型语言来写一些工具，比如 minidump、shellcode loader。

为什么选择 nim？
==========

1.  可以直接编译为 C、C++、Objective-C 和 Javascript。
    
2.  语法简单，不依赖运行时虚拟机。
    
3.  具有极其成熟的外部接口 API。
    
4.  跨平台交叉编译。
    
5.  可以将代码直接编译为 Javascript，甚至初步支持 WebAssembly
    

使用 OffensiveNim
===============

OffensiveNim 提供了几个代码 demo

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zleycDibFeibpKicnGL8yNWdQQcBV5Hrj3Wd8pnrWDVCOn3rkiahbKUyzEzDLtvFxOZGKjwAeC1YQHlR0c5ZiciaKnOQ/640?wx_fmt=png)

我们使用 kali 来编译 OffensiveNim 提供的 demo，先 git clone 下来

```
git clone https://github.com/byt3bl33d3r/OffensiveNim.git
```

安装 nim 编译器和 nimble 包管理器。

```
sudo su
apt update
apt install nim
```

如果你要编译出 Windows 下可以运行的 exe 和 dll 文件，那么必须安装 mingw

```
apt install mingw-w64
```

然后使用包管理器 nimble 安装 winim 包

```
nimble install winim
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zleycDibFeibpKicnGL8yNWdQQcBV5Hrj3Wibtpc5wicCgIRH9Gauf5peXZBGVy8B7seD5Y6SvkeAkC4rD0LfD5iaBBg/640?wx_fmt=png)

最后 cd 到 OffensiveNim 的根目录，运行 make 命令即可。

几分钟编译完之后会在 bin 目录下生成 exe 可执行文件，拖回本地执行即可。

加载 shellcode
============

拖回本地的文件落地就被 Windows defender 杀烂，根据特征应该是 msf 的 shellcode 特征。![](https://mmbiz.qpic.cn/sz_mmbiz_png/zleycDibFeibpKicnGL8yNWdQQcBV5Hrj3WAH9Y7Pff5giciaszTo52TwIP6xH0f3Kd5FrWdZggbtXHsI6zFKGGCnyQ/640?wx_fmt=png)

查看其代码在 shellcode_bin.nim 中写死了 shellcode。![](https://mmbiz.qpic.cn/sz_mmbiz_png/zleycDibFeibpKicnGL8yNWdQQcBV5Hrj3W8ob60NwgFs8HYXKicA5nuRp3GS2tfP0BqtpBPYkEib1nDFxiaWABry5cA/640?wx_fmt=png)

那就用 nim 写一个 shellcode 混淆的就可以了，实现的代码就不放了，就一个 xor 加密 shellcode。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zleycDibFeibpKicnGL8yNWdQQcBV5Hrj3W3LotM8ntDWpQRVMuPYrsVJJwOrxZfjwsuWyRXxTQEeibfospXGrQJibA/640?wx_fmt=png)

正常上线![](https://mmbiz.qpic.cn/sz_mmbiz_png/zleycDibFeibpKicnGL8yNWdQQcBV5Hrj3WJZQbQrb0hFgPyYH8PHfRuqK9nKG638bvyPYbkGFq17Hp6onT3e42RA/640?wx_fmt=png)

参考
==

1.  https://github.com/byt3bl33d3r/OffensiveNim/
    
2.  https://twitter.com/byt3bl33d3r/status/1330572114970042369
    
3.  https://nim-lang.org/
    

分享、点赞、在看就是对我们的一种支持！

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zleycDibFeibp8YjH4BpZodsIJmZOG8Cc3sbuM3IMcxPurjryDzTA8WTHZTNIXvP1SUVvWh0PzSTxssDxmwydNrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nzxUaDY8yDCu9vYaicsKXmibIlxHDeXmK8yoDsVrSMpI3RgS4JPtgGPdqXToibeNYGEMgk5WznIayx4hwMd8sVgJA/640?wx_fmt=jpeg)