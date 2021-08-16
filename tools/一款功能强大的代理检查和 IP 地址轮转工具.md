> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/SU49cbUqHRLBcsRTdVtaEg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38Y9jTQZ2ldkWsLNu4eEODf2Zlcdib3oXgdibEgm9oKrxibypk5mVOUjKcdEict6PXwLq7bwGaHZpWBicw/640?wx_fmt=jpeg)

关于 mubeng
---------

mubeng 是一款功能强大的代理检查和 IP 地址轮转工具。该工具具备以下几种功能特性：

> 代理 IP 轮换：在每次发送请求之后变更你的 IP 地址。
> 
> 代理检测：检测你的代理 IP 是否仍然可用。
> 
> 支持所有的 HTTP/S 方法。
> 
> 支持传递所有的参数和 URI。
> 
> 支持 HTTP&Socksv5 代理协议。
> 
> 易于使用：你可以直接使用自己的代理文件来配置和运行 mubeng，并选择需要执行的操作。
> 
> 跨平台特性：无论你使用的是 Windows、Linux、macOS 或是树莓派，你都可以正常使用 mubeng。

工具安装
----

### 预编译源码安装

广大研究人员可以直接访问该项目的【Releases 页面】来获取预编译好的项目代码，下载之后即可直接运行。

### Docker 安装

直接运行下列命令即可将 mubeng 的 Docker 镜像拉取到本地：

```
▶ docker pull kitabisa/mubeng

```

### 源码安装

这里需要使用 Go 编译器（v1.15+）：

```
▶ GO111MODULE=on go get -u ktbs.dev/mubeng/cmd/mubeng

```

注意：上述命令也适用于工具更新。

或者，你也可以使用下列命令将源代码手动构建为可执行程序：

```
▶ git clone https://github.com/kitabisa/mubeng

▶ cd mubeng

▶ make build

▶ (sudo) mv ./bin/mubeng /usr/local/bin

▶ make clean

```

工具使用
----

该工具要求我们提供自己的代理列表，可以是需要检测的代理，或是用于代理 IP 轮转的代理地址池：

```
▶ mubeng [-c|-a :8080] -f file.txt [options...]

```

### 工具选项

下面给出的是该工具所有支持的选项参数：

```
▶ mubeng -h

```

<table><thead><tr><th><strong><strong>参数选项</strong></strong></th><th><strong><strong>描述</strong></strong></th></tr></thead><tbody><tr><td>-f, —file<file></file></td><td>代理文件</td></tr><tr><td>-a, —address<addr>:<port></port></addr></td><td>运行代理服务器</td></tr><tr><td>-d, —daemon</td><td>代理服务器守护程序</td></tr><tr><td>-c, —check</td><td>执行代理状态检测</td></tr><tr><td>-t, —timeout</td><td>代理服务器检测最大超时（默认为 30s）</td></tr><tr><td>-r, —rotate<after></after></td><td>每次请求后轮转代理 IP 地址（默认为 1）</td></tr><tr><td>-v, —verbose</td><td>导出 HTTP 请求 / 响应，或显示无响应的代理服务器</td></tr><tr><td>-o, —output</td><td>日志输出</td></tr><tr><td>-u, —update</td><td>更新 mubeng 至最新稳定版本</td></tr><tr><td>-V, —version</td><td>显示当前 mubeng 版本</td></tr></tbody></table>

工具使用样例
------

比如说，你有一个如下所示的代理列表：

```
▶ mubeng -f proxies.txt --check --output live.txt

```

### 代理检测

你可以在命令中传递—check 选项来执行代理检测：

```
▶ mubeng -a localhost:8089 -f live.txt -r 10

```

上述命令中还是用了—output 选项来将可用代理存储至 live.txt 文件中：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38Y9jTQZ2ldkWsLNu4eEODfVGgmdq5J5yVo4Cgy4BZWfpvQ8eCK7sxCuHcY1WefInYic9y7Z6puVrw/640?wx_fmt=jpeg)

### 代理 IP 轮转

如果你想轮转代理服务器 IP 地址的话，可以直接从 live.txt 中获取可用代理，此时你必须使用 - a 选项来运行代理服务器：

```
▶ mubeng -a localhost:8089 -f live.txt -r 10
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38Y9jTQZ2ldkWsLNu4eEODfRic0vLicDBlEfw3qT8aJ5qwDdBv46PSsYUE0V9X5GaNEIotaEjlDqnpQ/640?wx_fmt=jpeg)

### BurpSuite 代理

如果你想将 mubeng 作为 BurpSuite 中的上游代理使用的话，仅需按下图配置即可：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38Y9jTQZ2ldkWsLNu4eEODfx51H97rC6ZnOBmKYb8NEico5QKVuLMRyeNboNhLRlhc70Szw9nWbZzw/640?wx_fmt=jpeg)

项目地址
----

mubeng：【点击文末阅读原文】

许可证协议
-----

本项目的开发与发布遵循 Apache 开源许可证协议。

参考资料
----

https://golang.org/doc/install

https://pkg.go.dev/ktbs.dev/mubeng/pkg/mubeng#Transport

https://portswigger.net/burp/documentation/desktop/getting-started/installing-burp

https://www.zaproxy.org/download/

https://github.com/kitabisa/mubeng/blob/master/.github/CONTRIBUTING.md﻿

作者：Alpha_h4ck，转载于 freebuf。  

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC7IHABFmuMlWQkSSzOMicicfBLfsdIjkOnDvssu6Znx4TTPsH8yZZNZ17hSbD95ww43fs5OFEppRTWg/640?wx_fmt=gif)

●[干货 | 渗透学习资料大集合（书籍、工具、技术文档、视频教程）](http://mp.weixin.qq.com/s?__biz=MzIwMzIyMjYzNA==&mid=2247492287&idx=1&sn=d9a24bab1b095f95a688b1b151ec0b59&chksm=96d019baa1a790ac7c3104b792f4864f306dfd8c1d7548d3371dc78ac89550fb9fcb83a3cf99&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3b9EpYTX241vj3pudtgtj7V0XFvzpxP5tBHCOtpXZmmHcpPBq4STTxVe56CdHHUb8hmAD6fzRrpA/640?wx_fmt=jpeg)

**关注公众号: HACK 之道**

公众号

如文章对你有帮助，请支持点下 “赞”“在看”