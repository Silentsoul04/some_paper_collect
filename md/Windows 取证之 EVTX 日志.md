> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HpoQynX0HkXozDpQebZpdw)

  
![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LebHs2DGyKAEgZupcIbXWAgnQlIoLerewyAX3c3bLLg0iaTpJeUuGKrSWsicRvLMXwCIbhkUC8GqGibg/640?wx_fmt=gif)

**原创稿件征集**

  

邮箱：edu@antvsion.com

QQ：3200599554

黑客与极客相关，互联网安全领域里

的热点话题

漏洞、技术相关的调查或分析

稿件通过并发布还能收获

200-800 元不等的稿酬

##### 0x0、概述  

`evtx`文件是微软从 `Windows NT 6.0`(Windows Vista 和 Server 2008) 开始采用的一种全新的日志文件格式。在此之前的格式是 `evt` 。`evtx`由`Windows`事件查看器创建，包含 Windows 记录的事件列表，以专有的二进制 XML 格式保存。

##### 0x1、EVTX 文件结构

evtx 文件主要由三部分组成：

*   file header （文件头）
    
*   chunks （数据块）
    
*   trailing empty values （尾部填充空值）
    

**File Header（文件头）：**

文件头长度为 4KB（4096bytes），其结构如下：

<table width="758"><thead><tr><th>偏移</th><th>长度（Bytes）</th><th>值</th><th>描述</th></tr></thead><tbody><tr><td>0x00</td><td>8</td><td>"ElfFile\x00"</td><td>标志位 / 签名</td></tr><tr><td>0x08</td><td>8</td><td><br></td><td>第一个区块编号（存在时间最久的区块编号）</td></tr><tr><td>0x10</td><td>8</td><td><br></td><td>当前区块编号（块的编号从 0 开始）</td></tr><tr><td>0x18</td><td>8</td><td><br></td><td>下一条事件记录的 ID</td></tr><tr><td>0x20</td><td>4</td><td>128</td><td>文件头有效部分的大小</td></tr><tr><td>0x24</td><td>2</td><td>1</td><td>次要版本</td></tr><tr><td>0x26</td><td>2</td><td>3</td><td>主要版本</td></tr><tr><td>0x28</td><td>2</td><td>4096</td><td>文件头的大小</td></tr><tr><td>0x2A</td><td>2</td><td><br></td><td>区块的数量</td></tr><tr><td>0x2C</td><td>76</td><td><br></td><td><strong>未知 (空值)</strong></td></tr><tr><td>0x78</td><td>4</td><td><br></td><td>文件标志</td></tr><tr><td>0x7C</td><td>4</td><td><br></td><td>文件头前 120 bytes 的 CRC32 校验和</td></tr><tr><td>0x80</td><td>3968</td><td><br></td><td><strong>未知 (空值)</strong></td></tr></tbody></table>

我们可以使用 Hex 编辑器打开一个`evtx`文件查看一下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsAz0Ff2WZ475Dic39yxJFyN3KFXALA0qgqLmqTsR4Iyt1DwUAOx1D2IA/640?wx_fmt=png)

**Chunk(块)：**

每个块的大小是 65536 bytes（64KB），主要由三部分组成：

*   chunk header 块头
    
*   array of event records 事件记录组
    
*   unused space 未使用的空间
    

`chunk`头长度为 512bytes，其结构如下：

<table width="758"><thead><tr><th>偏移</th><th>长度（Bytes）</th><th>值</th><th>描述</th></tr></thead><tbody><tr><td>0x00</td><td>8</td><td>"ElfChnk\x00"</td><td>标志位 / 签名</td></tr><tr><td>0x08</td><td>8</td><td><br></td><td>基于日志编号的第一条日志记录的 ID</td></tr><tr><td>0x10</td><td>8</td><td><br></td><td>基于日志编号的最后一条日志记录的 ID</td></tr><tr><td>0x18</td><td>8</td><td><br></td><td>基于文件编号的第一条日志记录的 ID</td></tr><tr><td>0x20</td><td>8</td><td><br></td><td>基于文件编号的最后一条日志记录的 ID</td></tr><tr><td>0x28</td><td>4</td><td>128</td><td>chunk 头大小</td></tr><tr><td>0x2C</td><td>4</td><td><br></td><td>最后一条日志记录的偏移量（相对于块头的起始偏移量）</td></tr><tr><td>0x30</td><td>4</td><td><br></td><td>下一条日志记录的偏移量（相对于块头的起始偏移量）</td></tr><tr><td>0x34</td><td>4</td><td><br></td><td>事件记录数据的 CRC32 校验和</td></tr><tr><td>0x38</td><td>64</td><td><br></td><td><strong>Unknown (空值)</strong></td></tr><tr><td>0x78</td><td>4</td><td><br></td><td><strong>Unknown (flags?)</strong></td></tr><tr><td>0x7C</td><td>4</td><td><br></td><td>块头 CRC32 校验和（块头前 120 个字节和 128 至 512 字节的数据的 CRC32 校验和）</td></tr></tbody></table>

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsSSfS3yYfjG11Su6GMjTYwrsibflFa5OUybenQFb0fWvaUUHciaksScsg/640?wx_fmt=png)

**Event record(事件记录)：**

事件记录的长度非固定长度，其结构如下：

<table width="758"><thead><tr><th>偏移</th><th>长度（Bytes）</th><th>值</th><th>描述</th></tr></thead><tbody><tr><td>0x00</td><td>4</td><td>"\x2a\x2a\x00\x00"</td><td>标志位 / 签名</td></tr><tr><td>0x04</td><td>4</td><td><br></td><td>事件记录的长度</td></tr><tr><td>0x08</td><td>8</td><td><br></td><td>记录 ID</td></tr><tr><td>0x10</td><td>8</td><td><br></td><td>日志记录的写入时间（FILETIME）</td></tr><tr><td>0x18</td><td>不确定</td><td><br></td><td>基于二进制 XML 编码的信息</td></tr><tr><td>不确定</td><td>4</td><td><br></td><td>记录长度（副本）</td></tr></tbody></table>

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsB5ia5nNZFGlRmKrkjE1QMYn2rEB6cQqtaA6yib8SupjfhGsclq98JSnw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgs1bJaAOPy8de29gm3IsgWFDa77tLJHsUsNECmay3bibcUSmv7BJ4afHQ/640?wx_fmt=png)

由上面的信息，可知`evtx`日志文件包含一个 4KB 的文件头加后面一定数量的 64KB 大小的块，一个块中记录一定数量（大约 100 条）的事件记录。每个块是独立的，不受其他块影响。不会出现一条事件记录的数据存在于两个块中。每条记录包含一个基于二进制 XML 编码的信息。每条事件记录包含其创建时间与事件 ID（可以用于确定事件的种类），因此可以反映某个特定的时间发生的特定的操作，取证人员可以根据日志文件来发现犯罪的过程。

`evtx`日志文件大概的结构如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgs9QfV9RGPzDdTj9cQona13u0fpricA2aPO0TjHQ2iaZHQWFnkoBUHguicg/640?wx_fmt=png)

在 windows 事件查看器中查看：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsahyevicUn2J4ibGfzOSfQKkvGQ7nE4SCty0icsaUfXIGibMbOw8UWibwHWg/640?wx_fmt=png)

##### 0x2、EVTX 文件的存储

Windows 事件日志文件保存在`%SystemRoot%\System32\Winevt\Logs`路径中。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsHe8icmGk8uYOyPy4TiaC7QJLUuzqbvHVllWia1p0328rAmzhOMtlRkkxA/640?wx_fmt=png)

常见日志文件主要有三个，分别是：`System.evtx` 、`Application.evtx` 和`Security.evtx`。分别是系统日志、应用程序日志和安全日志。

*   **System.evtx**
    
    记录操作系统自身组件产生的日志事件，比如驱动、系统组件和应用软件的崩溃以及数据丢失错误等等。
    
*   **Application.evtx**
    
    记录应用程序或系统程序运行方面的日志事件，比如数据库程序可以在应用程序日志中记录文件错误，应用的崩溃记录等。
    
*   **Security.evtx**
    
    记录系统的安全审计日志事件，比如登录事件、对象访问、进程追踪、特权调用、帐号管理、策略变更等。`Security.evtx`也是取证中最常用到的。
    

默认情况下，当一个`evtx`文件的记录满了，日志服务会覆盖最开始的记录，从头开始写入新的记录。也就是相当于一个循环记录的缓存文件。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgscGWoFwh3DEdS3ZUITicRFaaKx5NTg7zrGANSMuCoDvPII7YrUJRM2vg/640?wx_fmt=png)

##### 0x3、Evtx 日志分析

`Windows` 用 `Event ID`来标识事件的不同含义，拿`Security`日志来说，一些常见的`Event ID` 如下：

<table width="758"><thead><tr><th>事件 ID</th><th>描述</th></tr></thead><tbody><tr><td>4608</td><td>Windows 启动</td></tr><tr><td>4609</td><td>Windows 关机</td></tr><tr><td>4616</td><td>系统时间发生更改</td></tr><tr><td>4624</td><td>用户成功登录到计算机</td></tr><tr><td>4625</td><td>登录失败。使用未知用户名或密码错误的已知用户名尝试登录。</td></tr><tr><td>4634</td><td>用户注销完成</td></tr><tr><td>4647</td><td>用户启动了注销过程</td></tr><tr><td>4648</td><td>用户在以其他用户身份登录时，使用显式凭据成功登录到计算机</td></tr><tr><td>4703</td><td>令牌权限调整</td></tr><tr><td>4704</td><td>分配了用户权限</td></tr><tr><td>4720</td><td>已创建用户账户</td></tr><tr><td>4725</td><td>账户被禁用</td></tr><tr><td>4768</td><td>请求 Kerberos 身份验证票证（TGT）</td></tr><tr><td>4769</td><td>请求 Kerberos 服务票证</td></tr><tr><td>4770</td><td>已续订 Kerberos 服务票证</td></tr><tr><td>4779</td><td>用户在未注销的情况下断开了终端服务器会话</td></tr></tbody></table>

1、通过 Windows 事件查看器分析日志

通过`Windows`事件查看器可以查看当前主机的事件日志，也可以打开保存的 `evtx`文件。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsicswRLhGwTgYDTv33nscRzSWYicpWjxA7hROBYNBsnPbjJicX93rWftWw/640?wx_fmt=png)

可以通过点击、筛选、查找等多种方式查看事件日志

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsykeWibJo2Kb0qC4xfEgLt9ibQ3jaWJXl0P8U11G4XhWd3HjBJiaSpuIeA/640?wx_fmt=png)

筛选器提供了丰富的筛选方式：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgslB6KVeudRiaQyUGqT4Exnr5LxUfL94UO6ibup8M3jzqya0rdjfUlCBJQ/640?wx_fmt=png)

2、通过工具分析 Evtx

**Log Parser**

`Log Parser`（是微软公司自己开发的日志分析工具，它功能强大，使用简单，可以分析基于文本的日志文件、`XML` 文件、`CSV`（逗号分隔符）文件，以及操作系统的事件日志、注册表、文件系统、Active Directory。它使用类似 SQL 语句一样查询分析这些数据，还可以把分析结果以图表的形式展现出来。

`Log Parser`下载地址：https://www.microsoft.com/en-us/download/details.aspx?id=24659

使用方法:  

```
logparser -i:输入文件的格式 -o:输出文件的格式 "查询语句 和文件路径"
```

```
LogParser.exe -i:EVT -o:DATAGRID  "SELECT *  FROM E:\Security.evtx where EventID=4624"
```

查询登录成功的事件：

```
>LogParser.exe

Microsoft (R) Log Parser Version 2.2.10
Copyright (C) 2004 Microsoft Corporation. All rights reserved.

Usage:   LogParser [-i:<input_format>] [-o:<output_format>] <SQL query> |
                   file:<query_filename>[?param1=value1+...]
                   [<input_format_options>] [<output_format_options>]
                   [-q[:ON|OFF]] [-e:<max_errors>] [-iw[:ON|OFF]]
                   [-stats[:ON|OFF]] [-saveDefaults] [-queryInfo]

         LogParser -c -i:<input_format> -o:<output_format> <from_entity>
                   <into_entity> [<where_clause>] [<input_format_options>]
                   [<output_format_options>] [-multiSite[:ON|OFF]]
                   [-q[:ON|OFF]] [-e:<max_errors>] [-iw[:ON|OFF]]
                   [-stats[:ON|OFF]] [-queryInfo]

 -i:<input_format>   :  one of IISW3C, NCSA, IIS, IISODBC, BIN, IISMSID,
                        HTTPERR, URLSCAN, CSV, TSV, W3C, XML, EVT, ETW,
                        NETMON, REG, ADS, TEXTLINE, TEXTWORD, FS, COM (if
                        omitted, will guess from the FROM clause)
 -o:<output_format>  :  one of CSV, TSV, XML, DATAGRID, CHART, SYSLOG,
                        NEUROVIEW, NAT, W3C, IIS, SQL, TPL, NULL (if omitted,
                        will guess from the INTO clause)
 -q[:ON|OFF]         :  quiet mode; default is OFF
 -e:<max_errors>     :  max # of parse errors before aborting; default is -1
                        (ignore all)
 -iw[:ON|OFF]        :  ignore warnings; default is OFF
 -stats[:ON|OFF]     :  display statistics after executing query; default is
                        ON
 -c                  :  use built-in conversion query
 -multiSite[:ON|OFF] :  send BIN conversion output to multiple files
                        depending on the SiteID value; default is OFF
 -saveDefaults       :  save specified options as default values
 -restoreDefaults    :  restore factory defaults
 -queryInfo          :  display query processing information (does not
                        execute the query)


Examples:
 LogParser "SELECT date, REVERSEDNS(c-ip) AS Client, COUNT(*) FROM file.log
            WHERE sc-status<>200 GROUP BY date, Client" -e:10
 LogParser file:myQuery.sql?myInput=C:\temp\ex*.log+myOutput=results.csv
 LogParser -c -i:BIN -o:W3C file1.log file2.log "ComputerName IS NOT NULL"

Help:
 -h GRAMMAR                  : SQL Language Grammar
 -h FUNCTIONS [ <function> ] : Functions Syntax
 -h EXAMPLES                 : Example queries and commands
 -h -i:<input_format>        : Help on <input_format>
 -h -o:<output_format>       : Help on <output_format>
 -h -c                       : Conversion help
```

```
EvtxECmd.exe -f 日志文件 --xml 输出路径
```

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsHYA6HoVSWI2z6pvhSl1jmnTnbv2ebX4vLOKvAap6nia78ZUuWAwoMcg/640?wx_fmt=png)

还有其他的语法，具体可以查看其帮助信息

```
>LogParser.exe
Microsoft (R) Log Parser Version 2.2.10
Copyright (C) 2004 Microsoft Corporation. All rights reserved.
Usage:   LogParser [-i:<input_format>] [-o:<output_format>] <SQL query> |
                   file:<query_filename>[?param1=value1+...]
                   [<input_format_options>] [<output_format_options>]
                   [-q[:ON|OFF]] [-e:<max_errors>] [-iw[:ON|OFF]]
                   [-stats[:ON|OFF]] [-saveDefaults] [-queryInfo]
         LogParser -c -i:<input_format> -o:<output_format> <from_entity>
                   <into_entity> [<where_clause>] [<input_format_options>]
                   [<output_format_options>] [-multiSite[:ON|OFF]]
                   [-q[:ON|OFF]] [-e:<max_errors>] [-iw[:ON|OFF]]
                   [-stats[:ON|OFF]] [-queryInfo]
 -i:<input_format>   :  one of IISW3C, NCSA, IIS, IISODBC, BIN, IISMSID,
                        HTTPERR, URLSCAN, CSV, TSV, W3C, XML, EVT, ETW,
                        NETMON, REG, ADS, TEXTLINE, TEXTWORD, FS, COM (if
                        omitted, will guess from the FROM clause)
 -o:<output_format>  :  one of CSV, TSV, XML, DATAGRID, CHART, SYSLOG,
                        NEUROVIEW, NAT, W3C, IIS, SQL, TPL, NULL (if omitted,
                        will guess from the INTO clause)
 -q[:ON|OFF]         :  quiet mode; default is OFF
 -e:<max_errors>     :  max # of parse errors before aborting; default is -1
                        (ignore all)
 -iw[:ON|OFF]        :  ignore warnings; default is OFF
 -stats[:ON|OFF]     :  display statistics after executing query; default is
                        ON
 -c                  :  use built-in conversion query
 -multiSite[:ON|OFF] :  send BIN conversion output to multiple files
                        depending on the SiteID value; default is OFF
 -saveDefaults       :  save specified options as default values
 -restoreDefaults    :  restore factory defaults
 -queryInfo          :  display query processing information (does not
                        execute the query)
Examples:
 LogParser "SELECT date, REVERSEDNS(c-ip) AS Client, COUNT(*) FROM file.log
            WHERE sc-status<>200 GROUP BY date, Client" -e:10
 LogParser file:myQuery.sql?myInput=C:\temp\ex*.log+myOutput=results.csv
 LogParser -c -i:BIN -o:W3C file1.log file2.log "ComputerName IS NOT NULL"
Help:
 -h GRAMMAR                  : SQL Language Grammar
 -h FUNCTIONS [ <function> ] : Functions Syntax
 -h EXAMPLES                 : Example queries and commands
 -h -i:<input_format>        : Help on <input_format>
 -h -o:<output_format>       : Help on <output_format>
 -h -c                       : Conversion help
```

**Log Parser Studio**

`logparser`的 GUI 版本。

下载地址：https://techcommunity.microsoft.com/t5/exchange-team-blog/log-parser-studio-2-0-is-now-available/ba-p/593266

其界面如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgskuq4duPLgE1E3Gicicxib7lVxBVzBPMXURF8iabFROp3eBenFbicP100T8w/640?wx_fmt=png)

**Event Log Explorer**

Event Log Explorer 是一个非常好用的 Windows 日志分析工具，下载地址：https://eventlogxp.com/

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsDbjTvR0nX2TtaREueZlmDRbPJQJMVNiadxK0FHoZA80obUhaIaJTqjQ/640?wx_fmt=png)

**LogParser Lizard**

LogParser Lizard 是一个功能丰富的 Windows 日志分析软件，可以通过类似 SQL 查询语句对日志筛选查询进行分析。

下载地址：https://lizard-labs.com/log_parser_lizard.aspx

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsWYWOu9M5Nyicd6EwXacfWLQetylrKBbz87fHqicuiahXVlhrZMRwNic9PA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsaBI4bOicF4kjrozvTKLENwib3dLozDpicQjB4QKvMM72qSn9z9YRpZfYg/640?wx_fmt=png)

**Evtx Explorer/EvtxECmd**

具有标准化 CSV、XML 和 json 输出的事件日志 (Evtx) 解析器！

下载地址：https://ericzimmerman.github.io/#!index.md

使用方法：

```
EvtxECmd.exe -f 日志文件 --xml 输出路径
```

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgs913JYY67SHEQ1IfJVNFk0nL7o6ugF8YxWgCNPdGzz81lRA97PTc4hQ/640?wx_fmt=png)

解析的 xml 文件结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsS5S4TOTicboz1Fv1GQC3X53dCVcy05xPCzxTejp39SIO5WbLWzlb2uQ/640?wx_fmt=png)

##### 0x4、Evtx 取证实战

题目来源：Cynet 应急响应挑战赛

描述：`GOT Ltd` 的人力资源主管`King-Slayer`认为他的电脑上有可疑活动。

2020 年 2 月 8 日，15:00 左右，他发现桌面上出现了一个带有 `kiwi`标志的文件。据他描述，该文件首次出现在他的桌面后不久就突然消失了。那天晚些时候，他开始收到消息告诉他需要重新激活 `Windows Defender`。他激活了 `Windows Defender`，几个小时后又收到了同样的消息。

他决定将这件事告诉他在 IT 部门的朋友——`Chris`。`Chris`立即将此事报告给了 `GOT` 的网络安全部门。

该公司的 CISO 立即打电话求助我们，GOT 有限公司总部设在瑞士，`CISO` 向我们发送了来自 `King-Slayer`的 `PC` 和域控制器的所有事件日志文件。他希望我们查出异常：

提示：

*   用户帐户 (KingSlayer) 是他电脑上的本地管理员。
    
*   域名 -> GOT.Com
    
*   DC 服务器名称 -> WIN-IL7M7CC6UVU
    
*   Jaime(King Slayer) 的主机名 ->DESKTOP-HUB666E(172.16.44.135)
    

提交攻击者使用的域用户帐户 (King-Slayer 除外) 以及他使用此用户帐户访问的主机的 IP 地址。

我们拿到的文件包括 DC 服务的日志和主机日志文件：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgs1aibzRGhxAichsibes6andbN8Wja6urflcsY64N46ibCRsBwzUcTWQ8HDQ/640?wx_fmt=png)

给出的文件还有一个提示就是`PassTheHash` , 表明攻击者使用了该技术。

> 传递哈希是一种黑客技术，它允许攻击者使用用户密码的基础 NTLM 或 LanMan 哈希对远程服务器或服务进行身份验证，而不是像通常情况下那样要求使用关联的明文密码。它取代了仅窃取哈希值并使用该哈希值进行身份验证而窃取明文密码的需要。--via 维基百科

通过日志交叉比对和筛选查找，我们确定了在`2020-2-9 21:59`左右，有异常登录行为

注意：Windows EVTX 的`FILETIME` 是 UTC 时间，注意转化为瑞士当地时间。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgs1icv7YArsVibVIUtvdt02ryXjpxt8OzJX9kPFoOxkjxTErftrjJuibQrA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfG0V0sINMyGa4nkWr2SOgsyxSEENiciaTrFrNH1VDnxrN7XZO65AzxL6icauTaLZLK5pAPruFsWlh7Q/640?wx_fmt=png)

我们发现用户`Daenerys`在 2020 年 2 月 9 日 21:59 (当地时间 15：59) 通过`SMB`协议登录到`WIN-IL7M7CC6UVU`(域控制器)，而且使用了`PSExec.exe` 利用`Deanerys`用户登录了域控服务器。攻击者可能使用了`Mimikatz`拿到了`Daenerys`用户的哈希，然后用于横向移动渗透到 DC。

**参考资料**

https://github.com/williballenthin/python-evtx

Windows EVTX 日志恢复与取证技术研究 ：https://xuewen.cnki.net/CMFD-1018252760.nh.html

**实操推荐：windows 日志查看与清理**
------------------------

https://www.hetianlab.com/expc.do?ec=6f9e720b-9bfb-488f-b348-9ca91cca8828&pk_campaign=weixin-wemedia#stu

通过本实验掌握常见服务的日志记录位置，阅读常见服务产生的日志，掌握系统日志、服务日志和计划任务日志的清理方法。  

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LfbzQb75ZqoK2T2YO9XTQYD0aDUibvcxdbLRqzCwlkYcn0HppvXpZuenRzjX8ibhzcibJJge9Bw9xc8A/640?wx_fmt=gif)

  

**戳**

**“阅读原文”**

**体验免费靶场！**