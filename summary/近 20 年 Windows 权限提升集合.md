\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/Q4j4hM0TUdRsjwNixOoIjg)

> 来源：https://github.com/Ascotbe/Kernelhub
> 
> 来源：https://www.ascotbe.com/2020/08/10/KernelHub/#%E5%88%A9%E7%94%A8%E6%96%B9%E5%BC%8F-4
> 
> 前言

该项目是一个 Windows 提权搜集项目，除未通过测试 EXP 都有详细说明以及演示 GIF 图，如果项目中的代码有您的代码，本人为标注来源的请提交 Issues

> 未测试成功编号

下列编号都是在筛选后未能通过复现测试的 CVE，附带未成功原因，欢迎提交 PR

<table width="NaN"><thead><tr><th>SecurityBulletin</th><th>Remarks</th></tr></thead><tbody><tr><td>CVE-2015-0002</td><td>有源码未能测试成功</td></tr><tr><td>CVE-2015-0062</td><td>有源码和 EXP 未能测试成功</td></tr><tr><td>CVE-2015-1725</td><td>有源码未知编译方式</td></tr><tr><td>CVE-2016-3309</td><td>有源码和 EXP 未能测试成功</td></tr><tr><td>CVE-2014-6321</td><td>只有 winshock_test.sh 文件</td></tr><tr><td>CVE-2019-0859</td><td>需要安装 windows7 sp1 x64 需要更新 2019 年 3 月份的补丁</td></tr><tr><td>CVE-2018-8440</td><td><br></td></tr><tr><td>CVE-2018-1038</td><td>有源码未知编译方式</td></tr><tr><td>CVE-2013-5065</td><td>缺少 NDProxy 环境</td></tr><tr><td>CVE-2013-0008</td><td><br></td></tr><tr><td>CVE-2009-0079</td><td>未能利用</td></tr><tr><td>CVE-2011-0045</td><td>未能找到可用 EXP</td></tr><tr><td>CVE-2010-2554</td><td>未能找到可用 EXP</td></tr><tr><td>CVE-2005-1983</td><td>有源码和 EXP 未能测试成功</td></tr><tr><td>CVE-2012-0002</td><td>蓝屏漏洞无实际利用价值</td></tr><tr><td>CVE-2010-0020</td><td>未能找到可用 EXP</td></tr><tr><td>CVE-2014-6324</td><td><br></td></tr><tr><td>CVE-2018-0743</td><td>未能找到利用 POC</td></tr></tbody></table>

### 编号列表

<table width="NaN"><thead><tr><th align="left">SecurityBulletin</th><th align="center">Description</th><th align="center">OperatingSystem</th></tr></thead><tbody><tr><td align="left">CVE-2020-1472</td><td align="center">Netlogon Elevation of Privilege</td><td align="center">Windows 2008/2012/2016/2019/1903/1909/2004</td></tr><tr><td align="left">CVE-2020-0796</td><td align="center">SMBv3 Remote Code Execution</td><td align="center">Windows 1903/1909</td></tr><tr><td align="left">CVE-2020-0787</td><td align="center">Windows Background Intelligent Transfer Service</td><td align="center">Windows 7/8/10/2008/2012/2016/2019</td></tr><tr><td align="left">CVE-2019-1458</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/2016</td></tr><tr><td align="left">CVE-2019-1388</td><td align="center">Windows Certificate Dialog Elevation of Privilege</td><td align="center">Windows 7/8/2008/2012/2016/2019</td></tr><tr><td align="left">CVE-2019-0859</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/2016/2019</td></tr><tr><td align="left">CVE-2019-0803</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/2016/2019</td></tr><tr><td align="left">CVE-2018-8639</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/2016/2019</td></tr><tr><td align="left">CVE-2018-8453</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/2016/2019</td></tr><tr><td align="left">CVE-2018-8440</td><td align="center">Windows ALPC Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/2016</td></tr><tr><td align="left">CVE-2018-8120</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/2008</td></tr><tr><td align="left">CVE-2018-1038</td><td align="center">Windows Kernel Elevation of Privilege</td><td align="center">Windows 7/2008</td></tr><tr><td align="left">CVE-2018-0743</td><td align="center">Windows Subsystem for Linux Elevation of Privilege</td><td align="center">Windows 10/2016</td></tr><tr><td align="left">CVE-2018-0833</td><td align="center">SMBv3 Null Pointer Dereference Denial of Service</td><td align="center">Windows 8/2012</td></tr><tr><td align="left">CVE-2017-8464</td><td align="center">LNK Remote Code Execution</td><td align="center">Windows 7/8/10/2008/2012/2016</td></tr><tr><td align="left">CVE-2017-0213</td><td align="center">Windows COM Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/2016</td></tr><tr><td align="left">CVE-2017-0143</td><td align="center">Windows Kernel Mode Drivers</td><td align="center">Windows 7/8/10/2008/2012/2016/Vista</td></tr><tr><td align="left">CVE-2017-0101</td><td align="center">GDI Palette Objects Local Privilege Escalation</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2016-7255</td><td align="center">Windows Kernel Mode Drivers</td><td align="center">Windows 7/8/10/2008/2012/2016/Vista</td></tr><tr><td align="left">CVE-2016-3371</td><td align="center">Windows Kernel Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2016-3309</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2016-3225</td><td align="center">Windows SMB Server Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2016-0099</td><td align="center">Secondary Logon Handle</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2016-0095</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2016-0051</td><td align="center">WebDAV Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2016-0041</td><td align="center">Win32k Memory Corruption Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2015-2546</td><td align="center">Win32k Memory Corruption Elevation of Privilege</td><td align="center">Windows 7/8/10/2008/2012/Vista</td></tr><tr><td align="left">CVE-2015-2387</td><td align="center">ATMFD.DLL Memory Corruption</td><td align="center">Windows 7/8/2003/2008/2012/Vista/Rt</td></tr><tr><td align="left">CVE-2015-2370</td><td align="center">Windows RPC Elevation of Privilege</td><td align="center">Windows 7/8/10/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2015-1725</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/10/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2015-1701</td><td align="center">Windows Kernel Mode Drivers</td><td align="center">Windows 7/2003/2008/Vista</td></tr><tr><td align="left">CVE-2015-0062</td><td align="center">Windows Create Process Elevation of Privilege</td><td align="center">Windows 7/8/2008/2012</td></tr><tr><td align="left">CVE-2015-0057</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2015-0003</td><td align="center">Win32k Elevation of Privilege</td><td align="center">Windows 7/8/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2015-0002</td><td align="center">Microsoft Application Compatibility Infrastructure Elevation of Privilege</td><td align="center">Windows 7/8/2003/2008/2012</td></tr><tr><td align="left">CVE-2014-6324</td><td align="center">Kerberos Checksum Vulnerability</td><td align="center">Windows 7/8/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2014-6321</td><td align="center">Microsoft Schannel Remote Code Execution</td><td align="center">Windows 7/8/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2014-4113</td><td align="center">Win32k.sys Elevation of Privilege</td><td align="center">Windows 7/8/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2014-4076</td><td align="center">TCP/IP Elevation of Privilege</td><td align="center">Windows 2003</td></tr><tr><td align="left">CVE-2014-1767</td><td align="center">Ancillary Function Driver Elevation of Privilege</td><td align="center">Windows 7/8/2003/2008/2012/Vista</td></tr><tr><td align="left">CVE-2013-5065</td><td align="center">NDProxy.sys</td><td align="center">Windows XP/2003</td></tr><tr><td align="left">CVE-2013-1345</td><td align="center">Kernel Driver</td><td align="center">Windows 7/8/2003/2008/2012/Vista/Rt/Xp</td></tr><tr><td align="left">CVE-2013-1332</td><td align="center">DirectX Graphics Kernel Subsystem Double Fetch</td><td align="center">Windows 7/8/2003/2008/2012/Vista/Rt</td></tr><tr><td align="left">CVE-2013-0008</td><td align="center">Win32k Improper Message Handling</td><td align="center">Windows 7/8/2008/2012/Vista/Rt</td></tr><tr><td align="left">CVE-2012-0217</td><td align="center">Service Bus</td><td align="center">Windows 7/2003/2008/Xp</td></tr><tr><td align="left">CVE-2012-0002</td><td align="center">Remote Desktop Protocol</td><td align="center">Windows 7/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2011-2005</td><td align="center">Ancillary Function Driver Elevation of Privilege</td><td align="center">Windows 2003/Xp</td></tr><tr><td align="left">CVE-2011-1974</td><td align="center">NDISTAPI Elevation of Privilege</td><td align="center">Windows 2003/Xp</td></tr><tr><td align="left">CVE-2011-1249</td><td align="center">Ancillary Function Driver Elevation of Privilege</td><td align="center">Windows 7/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2011-0045</td><td align="center">Windows Kernel Integer Truncation</td><td align="center">Windows Xp</td></tr><tr><td align="left">CVE-2010-4398</td><td align="center">Driver Improper Interaction with Windows Kernel</td><td align="center">Windows 7/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2010-3338</td><td align="center">Task Scheduler</td><td align="center">Windows 7/2008/Vista</td></tr><tr><td align="left">CVE-2010-2554</td><td align="center">Tracing Registry Key ACL</td><td align="center">Windows 7/2008/Vista</td></tr><tr><td align="left">CVE-2010-1897</td><td align="center">Win32k Window Creation</td><td align="center">Windows 7/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2010-0270</td><td align="center">SMB Client Transaction</td><td align="center">Windows 7/2008</td></tr><tr><td align="left">CVE-2010-0233</td><td align="center">Windows Kernel Double Free</td><td align="center">Windows 2000/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2010-0020</td><td align="center">SMB Pathname Overflow</td><td align="center">Windows 7/2000/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2009-2532</td><td align="center">SMBv2 Command Value</td><td align="center">Windows 2008/Vista</td></tr><tr><td align="left">CVE-2009-0079</td><td align="center">Windows RPCSS Service Isolation</td><td align="center">Windows 2003/Xp</td></tr><tr><td align="left">CVE-2008-4250</td><td align="center">Server Service</td><td align="center">Windows 2000/2003/Vista/Xp</td></tr><tr><td align="left">CVE-2008-4037</td><td align="center">SMB Credential Reflection</td><td align="center">Windows 2000/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2008-3464</td><td align="center">AFD Kernel Overwrite</td><td align="center">Windows 2003/Xp</td></tr><tr><td align="left">CVE-2008-1084</td><td align="center">Win32.sys</td><td align="center">Windows 2000/2003/2008/Vista/Xp</td></tr><tr><td align="left">CVE-2006-3439</td><td align="center">Remote Code Execution</td><td align="center">Windows 2000/2003/Xp</td></tr><tr><td align="left">CVE-2005-1983</td><td align="center">PnP Service</td><td align="center">Windows 2000/Xp</td></tr><tr><td align="left">CVE-2003-0352</td><td align="center">Buffer Overrun In RPC Interface</td><td align="center">Windows 2000/2003/Xp/Nt</td></tr></tbody></table>

### 所需环境

*   测试目标系统
    
    ```
    #Windows 7 SP1 X64
    ed2k://|file|cn\_windows\_7\_home\_premium\_with\_sp1\_x64\_dvd\_u\_676691.iso|3420557312|1A3CF44F3F5E0BE9BBC1A938706A3471|/
    #Windows 7 SP1 X86
    ed2k://|file|cn\_windows\_7\_home\_premium\_with\_sp1\_x86\_dvd\_u\_676770.iso|2653276160|A8E8BD4421174DF34BD14D60750B3CDB|/
    #Windows Server 2008 R2 SP1 X64
    ed2k://|file|cn\_windows\_server\_2008\_r2\_standard\_enterprise\_datacenter\_and\_web\_with\_sp1\_x64\_dvd\_617598.iso|3368839168|D282F613A80C2F45FF23B79212A3CF67|/
    #Windows Server 2003 R2 SP2 x86
    ed2k://|file|cn\_win\_srv\_2003\_r2\_enterprise\_with\_sp2\_vl\_cd1\_X13-46432.iso|637917184|284DC0E76945125035B9208B9199E465|/
    #Windows Server 2003 R2 SP2 x64
    ed2k://|file|cn\_win\_srv\_2003\_r2\_enterprise\_x64\_with\_sp2\_vl\_cd1\_X13-47314.iso|647686144|107F10D2A7FF12FFF0602FF60602BB37|/
    #Windows Server 2008 SP2 x86
    ed2k://|file|cn\_windows\_server\_standard\_enterprise\_and\_datacenter\_with\_sp2\_x86\_dvd\_x15-41045.iso|2190057472|E93B029C442F19024AA9EF8FB02AC90B|/
    #Windows Server 2000 SP4 x86
    ed2k://|file|ZRMPSEL\_CN.iso|402690048|00D1BDA0F057EDB8DA0B29CF5E188788|/
    #Windows Server 2003 SP2 x86
    thunder://QUFodHRwOi8vcy5zYWZlNS5jb20vV2luZG93c1NlcnZlcjIwMDNTUDJFbnRlcnByaXNlRWRpdGlvbi5pc29aWg==
    #Windows 8.1 x86
    ed2k://|file|cn\_windows\_8\_1\_enterprise\_x86\_dvd\_2972257.iso|3050842112|6B60ABF8282F943FE92327463920FB67|/
    #Windows 8.1 x64
    ed2k://|file|cn\_windows\_8\_1\_x64\_dvd\_2707237.iso|4076017664|839CBE17F3CE8411E8206B92658A91FA|/
    #Windows 10 1709 x64
    ed2k://|file|cn\_windows\_10\_multi-edition\_vl\_version\_1709\_updated\_dec\_2017\_x64\_dvd\_100406208.iso|5007116288|317BDC520FA2DD6005CBA8293EA06DF6|/
    
    ```
    
*   Linux 编译环境
    
    ```
    sudo vim /etc/apt/sources.list
    #在sources.list末尾添加deb http://us.archive.ubuntu.com/ubuntu trusty main universe
    sudo apt-get update
    sudo apt-get install mingw32 mingw32-binutils mingw32-runtime
    sudo apt-get install gcc-mingw-w64-i686 g++-mingw-w64-i686 mingw-w64-tools
    
    ```
    
*   Windows 编译环境
    
    ```
    VS2019（内置V142、V141、V120、V110、V100、V141\_xp、V120\_xp、V110\_xp、MFC）
    
    
    ```
    

### 关于错误

由于项目内容较多，难免有些错别字或者遗漏的 CVE 编号等问题，如果您发现了错误，还望提交 Issues 来帮助我维护该项目。

### 免责声明

本项目仅面向合法授权的企业安全建设行为，在使用本项目进行检测时，您应确保该行为符合当地的法律法规，并且已经取得了足够的授权。

如您在使用本项目的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。

在使用本项目前，请您务必审慎阅读、充分理解各条款内容，限制、免责条款或者其他涉及您重大权益的条款可能会以加粗、加下划线等形式提示您重点注意。除非您已充分阅读、完全理解并接受本协议所有条款，否则，请您不要使用本项目。您的使用行为或者您以其他任何明示或者默示方式表示接受本协议的，即视为您已阅读并同意本协议的约束。

### 参考项目 & 网站

*   windows-kernel-exploits
    
*   WindowsExploits
    
*   Exploits
    
*   CVE
    
*   CVE Details
    

![](https://mmbiz.qpic.cn/mmbiz_png/fQ4PtepQmkrY3S6FKNBia3twAdhCz1P5iaqsKMc4AVRibFWDS1lYFdHt7SULsljEluUHOhBDsOu7DO9TicCX4meuBQ/640?wx_fmt=jpeg)