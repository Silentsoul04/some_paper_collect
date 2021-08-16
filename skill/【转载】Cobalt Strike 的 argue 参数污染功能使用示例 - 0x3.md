> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.0x3.biz](http://www.0x3.biz/archives/837.html)

argue 参数污染

使用 adminstrator 或 system 权限

Use:

```
argue [command] [fake arguments]
```

注意：fake arguments 应该比真实的要长

**示例 1：powershell 一句话上线**

直接运行 powershell.exe 一句话上线命令，会直接被 360 拦截

![](http://0x3.biz/usr/uploads/2019/09/4230469366.png)

使用 processmonitor 检查，发现是传递的是真实参数

![](http://0x3.biz/usr/uploads/2019/09/113510917.png)

使用 cobalt strike 的 argue 参数污染：

```
argue powershell.exe xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

再运行 argue，检查污染结果

![](http://0x3.biz/usr/uploads/2019/09/1619789597.png)

execute 执行 powershell.exe（shell 命令不会成功，因为 shell 本质是 cmd.exe /c arguments）

![](http://0x3.biz/usr/uploads/2019/09/2557996347.png)

使用 processmonitor 检查，发现参数污染成功，且 360 未拦截 powershell.exe

![](http://0x3.biz/usr/uploads/2019/09/703783061.png)

成功上线！

![](http://0x3.biz/usr/uploads/2019/09/1212492366.png)

**示例 2：添加 guest 后门**

首先查看 guest 用户组权限为 Guests

![](http://0x3.biz/usr/uploads/2019/09/719750294.png)

添加 guest 为 administrators 组，发现被 360 拦截

![](http://0x3.biz/usr/uploads/2019/09/2326664917.png)

使用 argue 参数污染 net1 程序（注意是 net1，而不是 net，因为 net 还是会把真正的参数传递给 net1 的）

![](http://0x3.biz/usr/uploads/2019/09/4173040046.png)

在 cobaltstrike 上使用 execute 添加 guest 到 administrators 组

![](http://0x3.biz/usr/uploads/2019/09/3216868896.png)

检查 processmonitor 发现参数污染成功，且 360 未拦截

![](http://0x3.biz/usr/uploads/2019/09/4226297187.png)

成功添加 guest 到 administrators 组

![](http://0x3.biz/usr/uploads/2019/09/185225760.png)

不依赖 CobaltStrike，使用其他 argue 参数污染工具可以达到同样的效果。