> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QNo5BEBGc9dDQ8B2wzVDsg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibjBuB7ibc0B2q8MTMx8911Jw2YaNE7TlPyHLmpiasuGpBeKLjCKHnweGCwyR9K8llzmKaSEI9OngoA/640?wx_fmt=jpeg)

CSSG
----

CSSG 是一款功能强大的 Cobalt Strike Shellcode 生成工具。本质上来说，CSSG 是一个具备攻击性的 Python 脚本，广大研究人员可以使用它来轻松生成并格式化信标 Shellcode。

该工具支持生成无阶段信标 Shellcode，并带有暴露的退出方法、额外的格式化、加密、编码、压缩和多行输出等功能。

注意：Shellcode 的转换通常需要按菜单顺序降序执行。

执行要求
----

可选的 AES 加密选项使用 / assets 文件夹中的 python 脚本实现。

具体取决于要安装的 pycryptodome 包来执行 AES 加密。

在使用 pip 命令安装 pycryptodome 包时，具体取决于你的 Python 环境：

```
python -m pip install pycryptodome

python3 -m pip install pycryptodome

py -3 -m pip install pycryptodome

py -2 -m pip install pycryptodome
```

我们可以在 pip 安装执行完成之后检测 pycryptodome 包的安装情况，使用命令如下：

```
python -m pip list | grep crypto
```

生成器将会使用系统默认的 “python” 命令来启动 AES 加密脚本。

工具下载
----

广大研究人员可以使用下列命令将该项目源码克隆至本地并使用：

```
git clone https://github.com/RCStep/CSSG.git
```

Shellcode 生成器选项
---------------

**监听器：**

使用”…” 按钮选择一个有效的监听器。Shellcode 将会根据选择的监听器来生成。

**发送器：**

无阶段（CSSG 是一款不支持阶段操作的 Shellcode 生成器）。

**退出方法：**

进程：当信标关闭之后，退出整个进程；

线程：当信标关闭之后，退出运行信标的线程；

**本地 Shellcode 选项：**

如果要从现有信标执行 Shellcode，则可以使用该选项。

生成一个信标 Shellcode Payload，该 Payload 可以从同一架构父信标继承关键函数指针。

**现有会话：**

Shellcode 会将会话元数据提取至父 Beacon 会话中。

Shellcode 将会在此信标会话中执行。

**x86 选项：**

生成 x86 Shellcode，默认生成 x64 Shellcode。

**使用 Shellcode 文件：**

使用外部生成的原始 Shellcode 文件代替生成信标 Shellcode。

这将允许我们使用以前导出的 Shellcode 文件或其他工具（Donut、msfvenom 等）的输出。

**格式化：**

元数据 - Shellcode 二进制源码输出，无格式化；

十六进制 - Shellcode 十六进制格式输出；

0x90,0x90,0x90 - Shellcode C# 风格字节数组输出；

\x90\x90\x90 - Shellcode C\C++ 风格字节数组输出；

b64 - Base64 编码选项；

**异或加密 Shellcode：**

勾选以对 Shellcode 进行异或加密。

**异或密钥：**

使用随机生成的或可编辑的异或密钥字符进行加密。

多个字符意味着多轮异或加密。

**AES 加密 Shellcode：**

勾选以启用对 Shellcode 的 AES 加密，加密类型可选。

使用一个 Python 脚本来执行 AES 分组密码 AES-CBC 加密。

Shellcode 将会填充 \ 0 值来满足分组大小要求。

除此之外，工具还会在在加密的 Shellcode 数据前面加上一个随机生成的向量。

**AES 密钥：**

用于加密的随机生成的可编辑 AES 密钥。

生成 32 字节的密钥，并优先用于 256 位加密强度。

接受的加密密钥字节长度为 16、24 和 32 位。

**编码和压缩：**

无编码 / 压缩 - 不对 Shellcode 进行编码和压缩。

b64 - 进行 Base64 编码。

Gzip + b64 - 先进行 gzip 压缩，然后进行 Base64 编码。

gzip - 进行 gzip 压缩。

b64 + gzip - 先进行 Base64 编码，然后进行 gzip 压缩。

工具运行截图
------

添加 Shellcode-Cobalt Strike 顶部菜单栏的 Shellcode 生成器：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibjBuB7ibc0B2q8MTMx8911JJ0iapZ8RyBLAxf4thokvZnnYK6dibibiahpKzj6A6ru2Iuic61uDJaqEiaiaw/640?wx_fmt=jpeg)

项目地址
----

CSSG：点击底部【阅读原文】  

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

  

  

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3ibjBuB7ibc0B2q8MTMx8911JkP8Bquuj2b0zbxP9zcKIicfjCZSt2Ab913DdM0bQ3kiaeDQs1SkIH73w/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485679&idx=1&sn=98231f665beaa2e51e310868a842bf49&scene=21#wechat_redirect)[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR38uc7ftXsPQPBCzNev5v295tWuoUWzRmpFgGEWkSMyRdicXrAbIHWIIuQNIBzdjfZfsNsJc8BbuNHw/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485648&idx=1&sn=5541b19c332754ec2073879966e6b181&scene=21#wechat_redirect)[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3ibySoIcGyJ8fyeXpPZqDCtqzgGnDIvw4TibfiamtaiaefSyadfFicibn6M3eKvI3R8zJVJHnz3KSakfTyA/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485625&idx=1&sn=26d16959e57236013380498fe93b1ddd&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************