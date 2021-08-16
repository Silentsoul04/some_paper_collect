> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/O9xlHJUNuDhSAG6koOhA-g)

**文章来****源：** **LemonSec**

 **最近在做系统安全基线检查相关的，网上找了一些脚本以及群友分享的。整理下分享给大家：**  

**首先是 Linux 的 shell 加固脚本  
**

```
#!/bin/bash

#设置密码复杂度
if [ -z "`cat /etc/pam.d/system-auth | grep -v "^#" | grep "pam_cracklib.so"`" ];then
  sed -i '/password    required      pam_deny.so/a\password    required      pam_cracklib.so  try_first_pass minlen=8 ucredit=-1   lcredit=-1   ocredit=-1 dcredit=-1 retry=3 difok=5' /etc/pam.d/system-auth
fi
#密码输入失败3次，锁定5分钟
sed -i 's#auth        required      pam_env.so#auth        required      pam_env.so\nauth       required       pam_tally.so  onerr=fail deny=3 unlock_time=300\nauth           required     /lib/security/$ISA/pam_tally.so onerr=fail deny=3 unlock_time=300#' /etc/pam.d/system-auth

#修改默认访问权限
sed -i '/UMASK/s/077/027/' /etc/login.defs

#设置重要文件目录权限
chmod 644 /etc/passwd  
chmod 600 /etc/xinetd.conf 
chmod 600 /etc/inetd.conf  
chmod 644 /etc/group  
chmod 000 /etc/shadow  
chmod 644 /etc/services  
chmod 600 /etc/security
#chmod 750 /etc/        #启动了nscd服务导致设置权限以后无法登陆 #系统默认755可以接受  #不能修改，如果修改polkit的服务就启动不了
chmod 750 /etc/rc6.d  
chmod 750 /tmp  
chmod 750 /etc/rc0.d/  
chmod 750 /etc/rc1.d/  
chmod 750 /etc/rc2.d/  
chmod 750 /etc/rc4.d  
chmod 750 /etc/rc5.d/  
chmod 750 /etc/rc3.d  
chmod 750 /etc/rc.d/init.d/  
chmod 600 /etc/grub.conf
chmod 600 /boot/grub/grub.conf
chmod 600 /etc/lilo.conf

#检查用户umask设置
sed -i '/umask/s/002/077/' /etc/csh.cshrc
sed -i '/umask/s/002/077/' /etc/bashrc
sed -i '/umask/s/002/077/' /etc/profile
csh_login=`cat /etc/csh.login | grep -i "umask"`
if [ -z "$csh_login" ];then
  echo -e "/numask 077" >>/etc/csh.login
fi


#FTP安全设置 #如果安装了FTP服务 可以进行这个设置
vsftpd_conf=`find /etc/ -maxdepth 2 -name vsftpd.conf`
if [ ! -z "$vsftpd_conf" ];then
  sed -i '/anonymous_enable/s/YES/NO/' $vsftpd_conf
fi

ftpuser=`find /etc/ -maxdepth 2 -name ftpusers`
if [ ! -z "$ftpuser" ] && [ -z "`cat $ftpuser | grep -v "^#" | grep root`"];then
  echo "root" >>$ftpuser
fi

sed -i '/^ftp/d' /etc/passwd

#重要文件属性设置
chattr +i /etc/passwd
chattr +i /etc/shadow
chattr +i /etc/group
chattr +i /etc/gshadow
chattr +a /var/log/messages
#chattr +i /var/log/messages.*

#检查core dump 设置
chk_core=`grep core /etc/security/limits.conf | grep -v "^#"`
if [ -z "$chk_core" ];then
  echo "*               soft    core            0"  >> /etc/security/limits.conf
  echo "*               hard    core            0"  >> /etc/security/limits.conf
fi

#删除潜在危险文件 可以先检查一下是否有危险文件，如果没有的话，就不需要执行这个
hosts_equiv=`find / -maxdepth 3 -name hosts.equiv 2>/dev/null`
if [ ! -z "$hosts_equiv" ];then
  mv "$hosts_equiv" "$hosts_equiv".bak
fi

_rhosts=`find / -maxdepth 3 -name .rhosts 2>/dev/null`
if [ ! -z "$_rhosts" ];then
  mv "$_rhosts" "$_rhosts".bak
fi

_netrc=`find / -maxdepth 3 -name .netrc 2>/dev/null`
if [ ! -z "$_netrc" ];then
  mv "$_netrc" "$_netrc".bak
fi

#检查系统内核参数配置,修改只当次生效，重启需重新设置
sysctl -w net.ipv4.conf.all.accept_source_route="0"
sysctl -w net.ipv4.conf.all.accept_redirects="0"
sysctl -w net.ipv4.icmp_echo_ignore_broadcasts="1"
sysctl -w net.ipv4.conf.all.send_redirects="0"
sysctl -w net.ipv4.ip_forward="0"

#打开syncookie，缓解syn fiood攻击
echo 1 > /proc/sys/net/ipv4/tcp_syncookies

#不响应ICMP请求
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

#防syn攻击优化，提高未连接队列大小
sysctl -w net.ipv4.tcp_max_syn_backlog="2048"

#检查拥有suid和sgid权限文件并修改文件权限为755 目前这些不需要改变权限，需要定期巡检
find /usr/bin/chage /usr/bin/gpasswd /usr/bin/wall /usr/bin/chfn /usr/bin/chsh /usr/bin/newgrp /usr/bin/write /usr/sbin/usernetctl /bin/mount /bin/umount /bin/ping /sbin/netreport -type f -perm /6000 | xargs chmod 755
```

然后是 Windows 的批处理脚本：（这里收集了两个，可根据自身情况修改整合）  

1、一键加固  

```
echo 现在开始Windows安全加固，确认请按任意键
pause
echo [version] >account.inf REM帐户口令授权配置模块
echo signature="$CHICAGO$" >>account.inf
echo [System Access] >>account.inf
echo MinimumPasswordLength=6 >>account.inf REM 修改帐户密码最小长度为6
echo PasswordComplexity=1 >>account.inf REM 开启帐户密码复杂性要求
echo MaximumPasswordAge=90 >>account.inf REM 修改帐户密码最长留存期为90天
echo PasswordHistorySize=5 >>account.inf REM 修改强制密码历史为5次
echo EnableGuestAccount=0 >>account.inf REM 禁用Guest帐户
echo LockoutBadCount=6 >>account.inf REM 设定帐户锁定阀值为6次
secedit /configure /db account.sdb /cfg account.inf /log account.log
del account.*

echo [version] >rightscfg.inf
REM 授权配置
echo signature="$CHICAGO$" >>rightscfg.inf
echo [Privilege Rights] >>rightscfg.inf
echo seremoteshutdownprivilege=Administrators >>rightscfg.inf
REM从远端系统强制关机只指派给Administrators组
echo seshutdownprivilege=Administrators >>rightscfg.inf
REM关闭系统仅指派给Administrators组
echo setakeownershipprivilege=Administrators >>rightscfg.inf
REM 取得文件或其它对象的所有权仅指派给Administrators
echo seinteractivelogonright=Administrators >> rightscfg.inf
REM 在本地登陆权限仅指派给Administrators
echo senetworklogonright=Administrators >>rightscfg.inf
REM只允许Administrators从网络访问
secedit /configure /db rightscfg.sdb /cfg rightscfg.inf /log rightscfg.log /quiet
del rightscfg.*

echo [version] >audit.inf REM 日志配置
echo signature="$CHICAGO$" >>audit.inf
echo [Event Audit] >>audit.inf
echo AuditSystemEvents=3 >>audit.inf REM
开启审核系统事件
echo AuditObjectAccess=3 >>audit.inf
REM 开启审核对象访问
echo AuditPrivilegeUse=3 >>audit.inf
REM 开启审核特权使用
echo AuditPolicyChange=3 >>audit.inf
REM 开启审核策略更改
echo AuditAccountManage=3 >>audit.inf
REM 开启审核帐户管理
echo AuditProcessTracking=3 >>audit.inf
REM 开启审核过程跟踪
echo AuditDSAccess=3 >>audit.inf
REM 开启审核目录服务访问
echo AuditLogonEvents=3 >>audit.inf
REM 开启审核登陆事件
echo AuditAccountLogon=3 >>audit.inf
REM 开启审核帐户登陆事件
echo AuditLog >>audit.inf
echo MaximumLogSize=8192 >>logcfg.inf REM 设置应用日志文件最大8192KB
echo AuditLogRetentionPeriod=0 >>logcfg.inf REM设置当达到最大的日志尺寸时按需要改写事件
echo RestrictGuestAccess=1 >>logcfg.inf REM设置限制GUEST访问应用日志
echo [Security Log] >>logcfg.inf REM设置安全日志
echo MaximumLogSize=8192 >>logcfg.inf REM 设置安全日志文件最大8192KB
echo AuditLogRetentionPeriod=0 >>logcfg.inf REM设置当达到最大的日志尺寸时按需要改写事件
echo RestrictGuestAccess=1 >>logcfg.inf REM设置限制GUEST访问安全日志
echo [Application Log] >>logcfg.inf REM设置应用日志
echo MaximumLogSize=8192 >>logcfg.inf 设置安全日志文件最大8192KB
echo AuditLogRetentionPeriod=0 >>logcfg.inf REM设置当达到最大的日志尺寸时按需要改写事件
echo RestrictGuestAccess=1 >>logcfg.inf REM设置限制GUEST访问安全日志
secedit /configure /db audit.sdb /cfg audit.inf /log audit.log /quiet
del audit.*

REM 共享配置
REM 清除admin$共享
net share admin$ /del 
REM 清除ipc$共享
net share ipc$ /del
REM 清除C盘共享
net share c$ /del   
REM 清除D盘共享
net share d$ /del   

REM IP协议配置
REM 启用SYN攻击保护
@echo Windows Registry Editor Version 5.00>>SynAttack.reg 
@echo [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services]>>SynAttack.reg 
@echo "SynAttackProtect"=dword:2>>SynAttack.reg
@echo "TcpMaxPortsExhausted"=dword:5>>SynAttack.reg
@echo "TcpMaxHalfOpen"=dword:500>>SynAttack.reg
@echo "TcpMaxHalfOpenRetried"=dword:400>>SynAttack.reg
@regedit /s SynAttack.reg
@del SynAttack.reg

REM 启用屏幕保护程序
@echo Windows Registry Editor Version 5.00>>scrsave.reg 
@echo [HKEY_CURRENT_USER\Control Panel\Desktop]>>scrsave.reg 
@echo "ScreenSaveActive"="1">>scrsave.reg
@echo "ScreenSaverIsSecure"="1">>scrsave.reg
@echo "ScreenSaveTimeOut"="300">>scrsave.reg
@echo "SCRNSAVE.EXE"="d:\\WINDOWS\\system32\\logon.scr">>scrsave.reg
@regedit /s scrsave.reg
@del scrsave.reg

REM “Microsoft网络服务器”设置为“在挂起会话之前所需的空闲时间”为15分钟
@echo Windows Registry Editor Version 5.00>>lanmanautodisconn.reg 
@echo [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters]>>lanmanautodisconn.reg 
@echo "autodisconnect"=dword:0000000f>>lanmanautodisconn.reg 
@regedit /s lanmanautodisconn.reg
@del lanmanautodisconn.reg

REM 关闭自动播放
@echo Windows Registry Editor Version 5.00>>closeautorun.reg
@echo [HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer]>>closeautorun.reg
@echo  "NoDriveTypeAutoRun"=dword:000000ff>>closeautorun.reg
@regedit /s closeautorun.reg
@del closeautorun.reg
```

2、加固  

```
@echo off

chcp 936

echo TANOVO-CHECK>%~dp0\JianCha@1.txt

echo  ---user list status--- >>%~dp0\JianCha@1.txt
net user & net localgroup administrators >>%~dp0\JianCha@1.txt

echo.>>%~dp0\JianCha@1.txt
echo ---------------------------------passowrd policy----------------------- >>%~dp0\JianCha@1.txt
net accounts>>%~dp0\JianCha@1.txt

echo.>>%~dp0\JianCha@1.txt
echo ---------------------------------share------------------------------->>%~dp0\JianCha@1.txt
net share>>%~dp0\JianCha@1.txt

REM XP系统不支持
echo ---------------------------------hotfix InstalledOn--------------------------->>%~dp0\JianCha@1.txt
wmic qfe get hotfixid,InstalledOn,InstalledBy>>%~dp0\JianCha@1.txt

REM REM XP系统使用
REM echo ---------------------------------hotfix InstalledOn--------------------------->>%~dp0\JianCha@1.txt
REM wmic qfe >>%~dp0\JianCha@1.txt


echo ---------------------------------port LISTEN---------------------------------->>%~dp0\JianCha@1.txt
netstat -ano | find "LISTEN">>%~dp0\JianCha@1.txt

echo ---------------------------------system policy-------------------------------->>%~dp0\JianCha@1.txt
Auditpol.exe /get /category:*>>%~dp0\JianCha@1.txt

REM 英文版本
echo ---------------------------------user,password Expired------------------------
echo administrator's user,password   
net user administrator|find "expires" 
echo guest's user,password
net user guest|find "expires"
REM 英文版本
echo ---------------------------------user active----------------------------------
echo administrator's status
net user administrator|find "active"
echo guest's status
net user guest|find "active"

REM REM 中文版本
REM echo ---------------------------------user,password Expired------------------------>>%~dp0\JianCha@1.txt
REM echo administrator's user,password>>%~dp0\JianCha@1.txt
REM net user administrator|find "到期" >>%~dp0\JianCha@1.txt
REM echo guest's user,password>>%~dp0\JianCha@1.txt
REM net user guest|find "到期">>%~dp0\JianCha@1.txt
REM REM 中文版本
REM echo ---------------------------------user active---------------------------------->>%~dp0\JianCha@1.txt
REM echo administrator's status>>%~dp0\JianCha@1.txt
REM net user administrator|find "启用">>%~dp0\JianCha@1.txt
REM echo guest's status>>%~dp0\JianCha@1.txt
REM net user guest|find "启用">>%~dp0\JianCha@1.txt

echo ---------------------------------system install------------------------------->>%~dp0\JianCha@1.txt
systeminfo|find "OS">>%~dp0\JianCha@1.txt

echo ---------------------------------service-------------------------------------->>%~dp0\JianCha@1.txt
net start>>%~dp0\JianCha@1.txt

echo ---------------------------------screen_Saver--------------------------------->>%~dp0\JianCha@1.txt
echo if screensaveractive == 1 , screen_Saver is active>>%~dp0\JianCha@1.txt
reg query "HKEY_CURRENT_USER\Control Panel\Desktop" /v ScreenSaveaAtive>>%~dp0\JianCha@1.txt
reg query "HKEY_CURRENT_USER\Control Panel\Desktop" /v ScreenSaverIsSecure>>%~dp0\JianCha@1.txt
reg query "HKEY_CURRENT_USER\Control Panel\Desktop" /v ScreenSaveTimeOut>>%~dp0\JianCha@1.txt

secedit /export /cfg LocalGroupPolicy&type LocalGroupPolicy >>%~dp0\JianCha@1.txt


echo -----firewall-in-rules----- >>%~dp0\JianCha@1.txt
netsh advfirewall firewall show rule name=all dir=in type=dynamic status=enabled >>%~dp0\JianCha@1.txt
echo -----firewall-out-rules----- >>%~dp0\JianCha@1.txt
netsh advfirewall firewall show rule name=all dir=out type=dynamic status=enabled >>%~dp0\JianCha@1.txt


pause
```

最后在分享两个表格：  

**Linux 安全检查**

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPxvvDvQN6BiadWZKJa3VuPnct0qLiahRePrjVrDeRvPe7LIUQzu88hPbz5Ld0qjjMI8ib7oIeLrMIe7Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPxvvDvQN6BiadWZKJa3VuPncy2iaXb8icPx5XhQA0HJzDw6H3S6jZMw2OylCjydGH5MBRE3x0Dys593Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPxvvDvQN6BiadWZKJa3VuPncOF5w0VvGYpN5CIpVm9LQpbovw0ldyxWqfFxjVibPUSbD2KadbFGXFSw/640?wx_fmt=png)

**Windows 安全检查**

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPxvvDvQN6BiadWZKJa3VuPncXeEGVuF5oA65ttGVtbLfRjRMQpIS2hP6DMl915GzaXn7mRZTwjJqibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPxvvDvQN6BiadWZKJa3VuPnciaXTUOFuicPia4rrqVnHBs0rFnkhNLKiaZ0g94aBRSCN0UUibrF7Ucg31Aw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPxvvDvQN6BiadWZKJa3VuPncICRcIRe0P7t5iaxzWXkNwrR1KXZ9JyqibSYKs5qDHBChHP4OMNNsWERQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPxvvDvQN6BiadWZKJa3VuPncno9dfDn0xPpW7koabC8a5C3BemkZM5d7qNTKhjnU2Z7Grmjiba57RLA/640?wx_fmt=png)

_版权申明：内容来源网络，版权归原创者所有。除非无法确认，都会标明作者及出处，如有侵权，烦请告知，我们会立即删除并致歉!_

公众号