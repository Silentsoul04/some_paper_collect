> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0fcxGYK--bYdtEvkuZBb2A)

一、基本命令：(## 注意 Empire 下命令区分大小写 ##)

1、启动 Empire 后，首先查看一下命令帮助信息

```
help
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBTnfFia8swptedZMiao55dN3DOGH5DPuRzHj7h45mXTiafl8Ik3YflM4iaQ/640?wx_fmt=png)

二、设置监听器：

与 coblatstrike 类似，也是要先创建 Linstener，然后后续操作选择生成的监听器即可，与 CS 不同的是：当开启多个监听时，必须使用不同的名称、不同的端口

1、进入 listeners 界面：

```
listeners
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBpwWtKJ7BKZwR7EDQO4Mib3NbicflUp2NsNPN6YiclhXxDNr2mAu689f3A/640?wx_fmt=png)

查看命令帮助

```
help
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBkPF96gdAQolAzH0tWsS3bMebsE3n5K1SY7jFJJ9Lm3IApIDqqdrDqg/640?wx_fmt=png)

2、创建 Listener：

**uselistene****r** 命令，可以通过空格 + 双击 tab 键，看到一共有 7 种 Listener，一般常用的是：**http、http_foreign、meterpreter、redirector**，其他的不怎么用

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBOWqGmRuMcJIURYyYNhBtHvBhO3oKr0iaqEQQXCbqxibNeh2yKFxCMKfw/640?wx_fmt=png)

创建 Listener：

①选择监听器类型

```
uselistener <监听器类型>
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBrQHSmrRNLwakLcyqFx7JEjiav28X4eBYjvR0mmw0WiaiaEqEDuMIibLWqQ/640?wx_fmt=png)

②查看所需配置

```
info
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBN62nO1iaTibz1PhmGvYKqFlzmlXgxRK4BqAeI7RoUvh3x8GD57ZHLbdw/640?wx_fmt=png)

③设置相关配置并启动监听

```
set Name <自定义名称>
set Host <服务器ip>
set Port <监听端口>
execute    #执行监听器
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBWTnmU3PCGORibRQn0sKXibleBZI7O37eZYRYBlv7fbtmCxOtOqzdw02g/640?wx_fmt=png)

back 返回 listeners 界面即可看到监听器列表

```
list
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBJR5nF5T99xIHFDMc8mwFsWEIvicCkuIPfia0lmCgN1aiaTYxCwaibEmQlw/640?wx_fmt=png)

④删除监听器

```
kill <监听器名称>
```

三、生成木马 stagers：

在 listeners 界面下操作，通过 usestager，空格 + 双击 tab 键可以看到支持的所有 stagers，

multi 是通用模块、osx 是 mac 操作系统、然后就是 windows 的模块

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBcibfQnSW9E8VjN5P7O0uJnw2cKaXUBHaiaavnW20udNcWUZXwGNRxazQ/640?wx_fmt=png)

①生成 stager：

```
usestager <模块名称>
set listener <监听器名称>
execute    #执行生成
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBNwgkoibmia0ibD2yHzkq85bPyuglxnYLmmCWr5fqia4POPNXM3sv9g9BPw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBeNNpfD3MMRmdroeyvCf0icOZttmXRk4nvFKDe4z2omZgh9kIiaGDyBXQ/640?wx_fmt=png)

然后会在 / tmp / 目录下生成木马文件，在目标主机运行就会上线

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBbcWYbo8xSSDgZmYPc2NlugtJIbd9vpjfZicmLNmticOWOdP7N0o1fWpQ/640?wx_fmt=png)

②使用 launcher 命令：

```
launcher <语言> <监听器名称>
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJB7wfF0TsPd5qrbGoHjbLo9JmCheKWy9kP5babXDsicPskB7R1l1YlXlA/640?wx_fmt=png)

将生成的这段 payload 在有 powershell 的目标机上执行，就会上线

③操作 agents(会话)：

1. 查看会话列表：

```
agents    #相当于msf的sessions命令
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBzwFDtm0QJWyt04sXJIMGPZSrtFQFbbfWH6v7XbzRmCl6021iaWVCBNA/640?wx_fmt=png)

重命名 agents：

```
rename <会话名称> <新的名称>
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBSjVe38HT9ZmmXVZPP9nZUn4NTCa14ZKwZCeUTDesLKpF39zJV30tKA/640?wx_fmt=png)

2. 进入会话：

```
interact <会话名称>
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJB70TnRh2HWm7TBcbGLbg3DIFaxUWoVY92QzCFqoOTyGFsfsaFDeKJkw/640?wx_fmt=png)

back 命令就可以实现将会话放置后台了  

3. 杀死会话：

```
kill <会话名称>  先执行
remove <会话名称>   再执行
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJB31I6URSbqgsibUfB72NshatcdCegIGIXa2xkuAibGGAibl7490icpiaUqZg/640?wx_fmt=png)

4.agents 具体命令操作：

很多命令和 MSF 的 meterpreter 会话操作差不多

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBToxqwuwvA9ORumEQuaz7OiaqY8WqucibpfnqUJRISxeEeSCrDw9PqYvQ/640?wx_fmt=png)

常用命令：

```
help agentcmds
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBfUVNg6vxzo5zh9HYT8hTe25nBMGqryEZNeqJBgbTKyEIEEWluycwvg/640?wx_fmt=png)

执行 cmd 命令：

```
shell +命令
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBd52PIoavqx1yXdu79ticqG6gd2tXUknXIVdhLbwickWzK4c0Hmdm2icSw/640?wx_fmt=png)

四、进程注入：

```
ps    #查看进程PID
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJB0afJbMGoD5GNtVlIpnwQ77WsF06icibibpMR6TAe3kf9jzUtY8p39JGicA/640?wx_fmt=png)

注入进程：

```
usemodule management/psinject
set ProcId
set Listener <监听器名称>
execute
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW6zPNUNkrCM6q2A1bwzceJBRAU0sIufGPpQmGQtYWDHnjibvkn7ia1PZh8wTNZP4rg8kxjr1EJ1uclg/640?wx_fmt=png)

注入成功会返回一个新的 agent 会话

**点个赞和在看吧，欢迎转发！**

**点个赞和在看吧，欢迎转发！**

**点个赞和在看吧，欢迎转发！**

![](https://mmbiz.qpic.cn/mmbiz_gif/ehibzaP4CvW5hb2Px7LJVkWEktazM0liacYxsJOVsyUz8lx6MSWyGTmJyJsPsgj9sOSueI5JRuQLTCPW5njR68aA/640?wx_fmt=gif)