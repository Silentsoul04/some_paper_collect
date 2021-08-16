> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gCZbVP2F5ZdQFfjjeIAWkg)

声明: 本文仅供参考学习, 本人及其平台不承担任何法律责任  

限于篇幅, 本文仅讨论 windows office 系列, wps 系列原理大致相同.

**1) 什么是宏:**

宏就是一些命令组织在一起，作为一个单独命令完成一个特定任务。Microsoft Word 中对宏定义为：“宏就是能组织到一起作为一独立的命令使用的一系列 word 命令，它能使日常工作变得更容易”。Word 使用宏语言 Visual Basic 将宏作为一系列指令来编写。

宏又分为局部宏和全局宏. 全局宏对所有文档都有效, 局部宏只对本文档有效.

注意: 在 office 系列中是可以直接使用宏的, wps 则需自己安装对应的软件.

**2) 如何录制一个宏**

(word 版本: 2007)

首先点击开发工具, 然后点击录制宏.

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbcP2fuyhaelASib9Q3mD5zFC8JOd56J8sC4guU7HX0NfvANExz9uOAuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbibcic5rJ6pcddnwdiaw1Axkw9f8GycoZUbsvLolZfNRW5t0XEuOvCEumg/640?wx_fmt=png)

首先输入宏名, 然后选择触发方式, 将宏指定到按钮可以理解为添加一个快捷方式

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbXYByYmOAOy1UBc5vToBuibpPxXxTYiauV1b9sD8cmVecJMnia1uibpJiccQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbXIY7aBKtqbBArJIXUv7ooicWD0GlXGKBex86kGKAPyntZTaOP0icykBw/640?wx_fmt=png)

当我们点击这个按钮的时候就会执行宏.

将宏指定到键盘相当于使用快捷键执行宏.

**3）使用宏隐藏文字**

在编写代码前, 首先要更改两个设置

_![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbab3r1BEX5EACTLicmgibOsl49yAOJhrChy5m2yQr4XfGMlDYCxA6sgUA/640?wx_fmt=png)_

将箭头所所指的两个选项的对勾取消, 否则隐藏字体将会失败.

Alt+F11 打开 VBA

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbYrXG0VgDOgMTPb0e5pPDWqcN4hPyjDrMSNAZyjZkiaqnrlcoG63NYCA/640?wx_fmt=png)

这里录制了一个名为 hidden 的宏, 用于隐藏文字

Vba 的语法请自行学习, 这里不再赘述. 关于 vba 的更多 API 及其属性请参考微软官方档案

```
https://docs.microsoft.com/zh-cn/office/vbaa/api/overview/
```

**4) 什么是宏病毒**

宏病毒是一种寄存在文档或模板的宏中的计算机病毒。一旦打开这样的文档，其中的宏就会被执行，于是宏病毒就会被激活，转移到计算机上，并驻留在 Normal 模板上。从此以后，所有自动保存的文档都会 “感染” 上这种宏病毒，而且如果其他用户打开了感染病毒的文档，宏病毒又会转移到他的计算机上。-------- 以上内容摘自百度百科

利用自动执行的宏达到取得控制权的目的, 自动执行的宏名也可以在微软档案中找到.

```
https://docs.microsoft.com/zhcn/office/vba/word/concepts/customizing-word/auto-macros
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbiaDfgDQ3YDPal8giaFOyozCuVBkJWBTAJvDNic1WmxSwoEaRD5GU8xTGA/640?wx_fmt=png)

编写宏如下

注意: 在 VBA 中, 不区分字母的大小写, AutoOpen 和 autoopen 等价.

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbjpgrHlYd1Lu2AdI1xB5Os7gTttjG5iaj6JokmDMf162AzUDTx80f0RQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMboPAyHrC1ET0BM2icL4gsmQ8uC66ogNRXKtNPhibJnF8EPR89tI4YbLXA/640?wx_fmt=png)

**5) 编写宏病毒**

典型案例 (CDO 自发邮箱获取系统用户名):

可以获取的信息有很多, 这里以获取系统用户名为例

代码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbryjEDvFXR9YmZorrGZcpX3cgl6q8VYCAaM5lEZWhdQEFkXyvutzyoA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbNh30tg43JapB8fcXUS85ic1vRPnnkhFVfAofHMibUmq4hkqvrriabYAPA/640?wx_fmt=jpeg)

**总结:**

 攻击思路: 在实战中有一个吸引人的文件名很重要, 同时最关键的是引诱他主动降低宏的安全性, 例如某个拙劣之际的 APT 组织对我国医疗机构的定向攻击.

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibaI3j7CiaOTkVP5QVV6ILMbF7urmszuhx0KhbibWiabIMOjFQIAjXzRmHIlsibeKoQUDZ11UOtCGmS5g/640?wx_fmt=png)

**防守思路:**

1: 在线沙箱检测文档是否宏病毒。

2: 开启禁用宏

3: 安装杀毒软件

宏病毒攻击方式多种多样 (邮箱钓鱼, 云宏病毒, 下载木马等等)

比起我写的拙作, 网上有很多值得分析的宏病毒样本，希望本文对如何编写宏病毒不是很了解的人有所帮助.

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

**[干货 | 恶意代码分析之 Office 宏代码分析](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247493020&idx=1&sn=a6e06e20e32fd14d7723ca014eac3a04&chksm=ec1cb0a3db6b39b5bd5c0bb1d77057be1faa86c3b5a8d1b7025220bbcbbe8872d828be18c024&scene=21#wechat_redirect)  
**

**[Office 如何快速进行宏免杀](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247492416&idx=1&sn=c444b28f7aa67e9ee15d42bc1aeef10d&chksm=ec1cb67fdb6b3f69d33753fd68cad86f401c07f5fddb3157c81468cab144978a5fb3840da037&scene=21#wechat_redirect)  
**

[**利用 DOCX 文档远程模板注入执行宏**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247487401&idx=1&sn=55821cd34f91e44b5b95934878f8430b&chksm=ec1f5a96db68d3807e3dc359870e30ef68cdca47ba206b3d7788d91ec57b98576586e99f5bea&scene=21#wechat_redirect)  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8Qfeuvouibfico2qhUHkxIvX2u13s7zzLMaFdWAhC1MTl3xzjjPth3bLibSZtzN9KGsEWibPgYw55Lkm5VuKthibQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247495879&idx=1&sn=ab05215b31822bee4461255e9fac3237&chksm=ec1ca5f8db6b2cee91b02eb6a70e5ed979c6e46ef40b548f8467affed9bc9de476854bc5a41a&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：Buffer

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)
