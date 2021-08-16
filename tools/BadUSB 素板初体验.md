> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MPpDtBLqcCmjPbS3-Ar7vA)

****文章源自【字节脉搏社区】- 字节脉搏实验室****

**作者 - K.Fire**

**扫描下方二维码进入社区：**

**![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnK3Fc7MgHHCICGGSg2l58vxaP5QwOCBcU48xz5g8pgSjGds3Oax0BfzyLkzE9Z6J4WARvaN6ic0GRQ/640?wx_fmt=png)**

**简介**

**BadUSB 攻击是一种利用 USB 固件中的固有漏洞的攻击，将一个写入了恶意代码的定制 USB 设备，例如 U 盘，插入受害者电脑，它会伪装成 HID 设备（Human InterfaceDevice，是计算机直接与人交互的设备，例如键盘、鼠标等）进行操作。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**硬件**

**![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1ynyekmAic6Zxk6W4NIgB8czvASXLkEYxcpbjWjQyMjmaZgpso6kQkHVw/640?wx_fmt=png)**

**本文使用的是 TB 上二十块还包邮的 Arduino Micro， 长上面这个样子**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**IDE**

**![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1yGicYrficP3A6XX8yQhsZAl1l3ypffnN4JuaKNMAHrazKEUURJQ3c3QCA/640?wx_fmt=png)**

**Arduino IDE 1.8.13**

**下载地址：https://www.arduino.cc/en/software**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**Cobaltstrike 上线**

**1.cobaltstrike 生成上线脚本**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1yxicXRCpt2KOiaL5ScYuqiaSgcMnrictuKD6dr5oY7dY5NwN9oiaZUaXbEyQ/640?wx_fmt=png)

**使用 powershell 远程加载:**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1ywdaajmjSTthEWlibXL4qRZMEWBfdQlEictuib8ZcrjEDchQuNibhibXjIPg/640?wx_fmt=png)

**在命令行中直接运行就会直接上线**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1yqD4yCiab6aTCjSh7nDW6ibgoMZ66IibxibOJafLCQCOCnjeiaT5QFLEUthQ/640?wx_fmt=png)

**2. 写入代码到 badusb**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1yv2zuDLtTT84Y0ibIcZw5iaosPURiaA0jH1MU67kJ2wwVwwjzUWq4xGuLA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1yia2DzUtWcfqVheEOU0v25UPOeNcmt5eNBicF1WLqribcPUQanwCcngLdw/640?wx_fmt=png)

**写入成功后，插上 badusb，就会自动使用 “运行” 执行以上 powershell 命令。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1yh9T1Ms4QfRv2vhwq1dHRdMzgNibFmdzOJbYdvAkgUCbrLwZU0zRicbfA/640?wx_fmt=png)

**过 UAC 防护**

**模拟键盘按下左方向键和回车即可，代码如下**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnKCWzDyYg3zXNr5MiaLdGJ1ySOBtoTQdiad7PzmjU24At1jAvolKmwhss0G27frw4VMsQaka2h46tmg/640?wx_fmt=png)

**其他**

**可以模拟键盘输入将 payload 加入开机启动项，这样也就同时实现了权限维持  
**

**绕过中文输入法：例如开启大写输入模式，或者模拟切换输入法，得具体情况具体分析**

**遇到杀毒软件拦截时，某篇文章提出可以通过模拟鼠标点击允许的方法绕过，但经过测试，这种类型的板子无法做到在所有的终端上模拟鼠标移动十分精确，所以这条路行不通**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**结语**

**目前，市面上主流的安全软件都能拦截 badusb 进行远程下载文件等一系列操作，网上绕过的方法也并不是很多，例如使用文件共享来实现 payload 的下载，但适用范围较小。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**通知！**

**公众号招募文章投稿小伙伴啦！只要你有技术有想法要分享给更多的朋友，就可以参与到我们的投稿计划当中哦~ 感兴趣的朋友公众号首页菜单栏点击【商务合作 - 我要投稿】即可。期待大家的参与~**

**![](https://mmbiz.qpic.cn/mmbiz_jpg/ia3Is12pQKnKRau1qLYtgUZw8e6ENhD9UWdh6lUJoISP3XJ6tiaibXMsibwDn9tac07e0g9X5Q6xEuNUcSqmZtNOYQ/640?wx_fmt=jpeg)**

**记得扫码**

**关注我们**