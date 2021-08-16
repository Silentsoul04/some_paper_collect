> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0yeEvE-VrixIhgBRTjimxQ)

概述  



---------

Nefilim 勒索软件出现于 2020 年 3 月，与另一个勒索软件家族 Nemty 共享一部分代码。Nefilim 利用 Citrix 网关设备中的漏洞 (如 CVE-2019-11634 和 CVE-2019-19781) 进行攻击，成功入侵后会窃取目标网络中的数据并加密，从而要求公司支付赎金，否则将泄漏窃取的数据。

技术细节  



-----------

**初始访问**

Nefilim 勒索软件会暴力破解 RDP 远程桌面协议进行分发，并利用 Citrix 网关设备中的已知漏洞实现初始访问。在目标系统中建立初始立足点后，攻击者便会释放 Nefilim 勒索软件并执行。

**横向移动**

攻击者使用 PsExec 等工具在受害者的网络中远程执行命令，并使用 Mimikatz、LaZagne 和 NetPass 等工具收集凭证。攻击者会利用窃取的凭证在受害网络中进行横向移动，试图发现更多敏感数据，执行的某些命令如下所示：

```
Start copy kill.bat \destinationip\c$\windows\temp
```

```
Start psexec.exe \destinationip -u domain\username\ -p password -d -h -r mstdc -s -accepteula -nobanner c:\windows\teamp\Kill.bat
```

```
Start psexec.exe -accepteula \destinationip -u domain\username\ -p password reg add HKLM\software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /F
```

```
WMIC /node: \destinationip /username:”domain\username” /password:”password” process CALL CREATE “cmd.exe /c copy \sourceip\c$\windows\temp C:\WINDOWS\TEMP\kill.bat"
```

```
WMIC /node: \destinationip /username:”domain\username” /password:”password” process CALL CREATE “cmd.exe /c C:\WINDOWS\TEMP\kill.bat"
```

如下图所示，攻击者使用使用 bat 脚本停止服务和杀死进程：

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZwRMgGLn7pj9RYibRSrDFIQsdmPwU6XNSMceE3tTV8MnsGMjh7QYI7TXdIKhmcic1icFC0myeZ9kLIA/640?wx_fmt=png)

图 1. 停止的服务  

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZwRMgGLn7pj9RYibRSrDFIQaj7eEuh5HNem5devcRysMlbbJlRhJPSYuRHuA7ySFe2CaBiaUbwIOiaA/640?wx_fmt=png)

图 2. 终止的进程  

**数据窃取**

Nefilim 会将服务器 / 共享目录中的数据复制本地目录下，并使用释放的 7zip 二进制文件进行压缩。它还会释放 MegaSync 并安装，以窃取数据。

**勒索软件执行**

Nefilim 恶意软件使用 AES-128 加密锁定文件，赎金是通过电子邮件支付的。加密后，它将释放名为 “NEFILIM-DECRYPT.txt” 的勒索说明文件。加密后文件的扩展名会被修改为. NEFILIM，且文件末尾被附加 AES 加密密钥。之后，将使用勒索软件可执行文件中的嵌入的 RSA-2048 公钥对其进行加密。除了加密的 AES 密钥外，勒索软件在所有加密文件中添加 “NEFILIM”字符串作为标记。

该勒索软件还调用 IsDebuggerPresent 函数进行反调试，该函数会检查用户模式调试器是否正在调试该进程。它还利用 GetTickCount/QueryPerformanceCounter API 获取系统重启时间与当前时间的相差毫秒数。

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZwRMgGLn7pj9RYibRSrDFIQBquteXcjicQjKLUAkeEhu0x2uh6L4CvyTfraRqdpl5ziaSYdhNFfHnag/640?wx_fmt=png)

图 3. 反调试 API  

成功感染后，Nefilim 会调用 ShellExecute API 以将自身从目标系统中删除。

```
"C:\Windows\System32\cmd.exe" /c timeout /t 3 /nobreak && del "C:\Users\admin\Download{ransomware_filename}.exe" /s /f /q
```

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakZwRMgGLn7pj9RYibRSrDFIQdTFodbibibSWQrXQZM2d6MC756e0a6uibnXTx2RYNlXeWfACIb6WepnnA/640?wx_fmt=png)

图 4. 删除自身

IoC  



----------

```
文件哈希：





8be1c54a1a4d07c84b7454e789a26f04a30ca09933b41475423167e232abea2b
b8066b7ec376bc5928d78693d236dbf47414571df05f818a43fb5f52136e8f2e
3080b45bab3f804a297ec6d8f407ae762782fa092164f8ed4e106b1ee7e24953
7de8ca88e240fb905fc2e8fd5db6c5af82d8e21556f0ae36d055f623128c3377
b227fa0485e34511627a8a4a7d3f1abb6231517be62d022916273b7a51b80a17
3bac058dbea51f52ce154fed0325fd835f35c1cd521462ce048b41c9b099e1e5
353ee5805bc5c7a98fb5d522b15743055484dc47144535628d102a4098532cd5
5ab834f599c6ad35fcd0a168d93c52c399c6de7d1c20f33e25cb1fdb25aec9c6
52e25bdd600695cfed0d4ee3aca4f121bfebf0de889593e6ba06282845cf39ea
35a0bced28fd345f3ebfb37b6f9a20cc3ab36ab168e079498f3adb25b41e156f
7a73032ece59af3316c4a64490344ee111e4cb06aaf00b4a96c10adfdd655599
08c7dfde13ade4b13350ae290616d7c2f4a87cbeac9a3886e90a175ee40fb641
D4492a9eb36f87a9b3156b59052ebaf10e264d5d1ce4c015a6b0d205614e58e3
B8066b7ec376bc5928d78693d236dbf47414571df05f818a43fb5f52136e8f2e
fcc2921020690a58c60eba35df885e575669e9803212f7791d7e1956f9bf8020
```

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**