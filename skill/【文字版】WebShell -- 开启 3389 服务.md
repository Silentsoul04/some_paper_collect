> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zGnrOaRAoucf3WeeXxVBPQ)

世间最好的默契，并非是有人懂你说出的故事，而是有人懂你说不出的心事。。。

----  网易云热评

一、查询远程连接的端口

1、命令查询，win7/win10/win2003 都可以使用

```
REG query HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server\WinStations\RDP-Tcp /v PortNumber
```

0xd3d 转换十进制是 3389  

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvwt0lb2rOUdfQptaItLgWH8Pr4YP0KXgXdO83YFRaX7uv4prPOOlSbw/640?wx_fmt=png)

2、注册表查询

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvPGehAL2zoyKWUwdXbs7IMSQBpUW2ZyLFySM6aWymL8NibYTrlTib6esQ/640?wx_fmt=png)

二、开启 3389 端口的方法

1、工具开启

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvfCTZicPALX7YsXD49KcM9xavtqPKWeAnGoQEhT5wjZCIbEYHzKibhRMw/640?wx_fmt=png)

2、批处理开启，将下面代码复制到 bat 文件，双击自行开启远程连接

```
echo Windows Registry Editor Version 5.00>>3389.reg
echo [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server]>>3389.reg
echo "fDenyTSConnections"=dword:00000000>>3389.reg
echo [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\Wds\rdpwd\Tds\tcp]>>3389.reg
echo "PortNumber"=dword:00000d3d>>3389.reg
echo [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp]>>3389.reg
echo "PortNumber"=dword:00000d3d>>3389.reg
regedit /s 3389.reg
del 3389.reg
```

三、当 3389 端口被修改后，如何查找

1、通过大马的读注册表、找到 3389 端口，然后读取

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvrjEiaxyr8fxCaFoxlrVaTStaC0ujRLHx0BAk0sFH8WS7UZmSGoaYRHQ/640?wx_fmt=png)

2、通过大马的 cmd 命令

tasklist /svc, 获取 termservice 的 pid 号

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvxDzP04m7Rn0XYsgLcpric20tet3uda3dQiczUsTqibq56OXGYoq4NZmjg/640?wx_fmt=png)

再执行 netstat -ano 查找 2684 对应的的端口就是远程连接端口

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvAaicxa7HO6EXmkynEGyDiaTIQnmmbUb5f8ibo93lvbiaAXCMZib7QI0TzIw/640?wx_fmt=png)

3、通过工具扫描

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvgfers1ia0l4JbhOaBibVMxicdqicjOe1Vn43crjBlebqP57MG9RlrbYEXw/640?wx_fmt=png)

工具下载方式：

1、关注视频号：之乎者也吧，私信

2、关注公众号：回复 20210314

注意：软件均来自网络，不能保证安全性

禁止非法，后果自负

欢迎关注公众号：web 安全工具库

欢迎关注视频号：之乎者也吧

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvqMfw8sUto4vjoibJxn8mA1syFElJeRTN58UXua3J3Fb0U2OOG7tIBaA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3Uibv9kCibPXonqiawmT8UpRPRxvicAiaoW4QGKIZC04kqiapXtcctqeibc2Sokh2QPgs1l69Ibz22VeKafeNw/640?wx_fmt=jpeg)