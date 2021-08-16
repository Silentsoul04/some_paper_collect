> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CfkC4EG_ODL1nesK2DVong)

公众号

![](https://mmbiz.qpic.cn/mmbiz_jpg/nDMNE6lrvW6zdXpuHONZh5VrAH7U5xmntCKibCkDcuwtxUibu3A0E1chkXr7cxibwiaFzGDic7EtkuYI8sQ561ib2mqQ/640?wx_fmt=jpeg)

### 账号安全

**1、用户信息文件 /etc/passwd**

`# 格式：account:password:UID:GID:GECOS:directory:shell  
# 用户名：密码：用户ID：组ID：用户说明：家目录：登陆之后的 shell  
root:x:0:0:root:/root:/bin/bash  
# 查看可登录用户：  
cat /etc/passwd | grep /bin/bash  
# 查看UID=0的用户  
awk -F: '$3==0{print $1}' /etc/passwd  
# 查看sudo权限的用户  
more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL)"  
`

注意：无密码只允许本机登陆，远程不允许登陆

**2、文件：/etc/shadow**

`# 用户名：加密密码：密码最后一次修改日期：两次密码的修改时间间隔：密码有效期：密码修改到期到的警告天数：密码过期之后的宽限天数：账号失效时间：保留  
root:$6$oGs1PqhL2p3ZetrE$X7o7bzoouHQVSEmSgsYN5UD4.kMHx6qgbTqwNVC5oOAouXvcjQSt.Ft7ql1WpkopY0UV9ajBwUt1DpYxTCVvI/:16809:0:99999:7:::  
`

**3、查看当前登录用户及登录时长**

`who     # 查看当前登录系统的所有用户（tty 本地登陆  pts 远程登录）  
w       # 显示已经登录系统的所用用户，以及正在执行的指令  
uptime  # 查看登陆多久、多少用户，负载状态  
`

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFTxHL28vosOl7Beq5aIGIeRsZ51cF3U8MqbicZfxTdvOotR8qHIxVicrg/640?wx_fmt=png)

**4、排查用户登录信息**

查看最近登录成功的用户及信息

`# 显示logged in表示用户还在登录  
# pts表示从SSH远程登录  
# tty表示从控制台登录，就是在服务器旁边登录  
last  
`

查看最近登录失败的用户及信息：

`# ssh表示从SSH远程登录  
# tty表示从控制台登录  
sudo lastb  
`

显示所有用户最近一次登录信息：

`lastlog  
`

在排查服务器的时候，黑客没有在线，可以使用 last 命令排查黑客什么时间登录的有的黑客登录时，会将`/var/log/wtmp`文件删除或者清空，这样我们就无法使用 last 命令获得有用的信息了。

在黑客入侵之前，必须使用`chattr +a`对`/var/log/wtmp`文件进行锁定，避免被黑客删除

**5、sudo 用户列表**

`/etc/sudoers  
`

### 入侵排查

`# 查询特权用户特权用户(uid 为0)：  
awk -F: '$3==0{print $1}' /etc/passwd  
# 查询可以远程登录的帐号信息：  
awk '/\$1|\$6/{print $1}' /etc/shadow  
# 除root帐号外，其他帐号是否存在sudo权限。如非管理需要，普通帐号应删除sudo权限：  
more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL)"  
# 禁用或删除多余及可疑的帐号  
usermod -L user    # 禁用帐号，帐号无法登录，/etc/shadow 第二栏为 ! 开头  
userdel user       # 删除 user 用户  
userdel -r user    # 将删除 user 用户，并且将 /home 目录下的 user 目录一并删除  
`

通过. bash_history 文件查看帐号执行过的系统命令：

打开 /home 各帐号目录下的 .bash_history，查看普通帐号执行的历史命令。

为历史的命令增加登录的 IP 地址、执行命令时间等信息：

``# 1、保存1万条命令：  
sed -i 's/^HISTSIZE=1000/HISTSIZE=10000/g' /etc/profile  
# 2、在/etc/profile的文件尾部添加如下行数配置信息：  
USER_IP=`who -u am i 2>/dev/null | awk '{print $NF}' | sed -e 's/[()]//g'`  
if [ "$USER_IP" = "" ]  
then  
USER_IP=`hostname`  
fi  
export HISTTIMEFORMAT="%F %T $USER_IP `whoami` "  
shopt -s histappend  
export PROMPT_COMMAND="history -a"  
# 3、让配置生效  
source /etc/profile  
``

注意：历史操作命令的清除：history -c

该操作并不会清除保存在文件中的记录，因此需要手动删除. bash_profile 文件中的记录

检查端口连接情况：

`netstat -antlp | more  
`

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFO4zmZUHNK6g1Nndu4COVKBHuvvKJC6cYul4YlTZQN3aY2ALTR5Cukw/640?wx_fmt=png)

使用 ps 命令，分析进程，得到相应 pid 号:

`ps aux | grep 6666  
`

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFUXDj4DdYXK1G7XCKzM0CsYyRIJ5lFr3P80rDNltC0wYx092BFjyFOw/640?wx_fmt=png)

查看 pid 所对应的进程文件路径：

`# $PID 为对应的 pid 号  
ls -l /proc/$PID/exe 或 file /proc/$PID/exe  
`

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFUGhy56quURiaFmRhJMrRaua32v2rVLCzR1mgcic9mqCxu3lDj78KRgDg/640?wx_fmt=png)

分析进程：

`# 根据pid号查看进程  
lsof -p 6071  
# 通过服务名查看该进程打开的文件  
lsof -c sshd  
# 通过端口号查看进程：  
lsof -i :22  
`

查看进程的启动时间点：

根据 pid 强行停止进程：

`kill -9 6071  
`

注意：如果找不到任何可疑文件，文件可能被删除，这个可疑的进程已经保存到内存中，是个内存进程。这时需要查找 PID 然后 kill 掉

检查开机启动项：

系统运行级别示意图：

查看运行级别命令：

`runlevel  
`

开机启动配置文件:

`/etc/rc.local  
/etc/rc.d/rc[0~6].d  
`

启动 Linux 系统时，会运行一些脚本来配置环境——rc 脚本。在内核初始化并加载了所有模块之后，内核将启动一个守护进程叫做 init 或 init.d。这个守护进程开始运行 / etc/init.d/rc 中的一些脚本。这些脚本包括一些命令，用于启动运行 Linux 系统所需的服务

开机执行脚本的两种方法：

*   在 /etc/rc.local 的 exit 0 语句之间添加启动脚本。脚本必须具有可执行权限
    
*   用 update-rc.d 命令添加开机执行脚本
    

1、编辑修改 /etc/rc.local

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFK5dia2IoynsVwDrjRcROqlRrJPz5LdeBM8efWiasNWXaGiaODw89Iv11w/640?wx_fmt=png)

2、update-rc.d：此命令用于安装或移除 System-V 风格的初始化脚本连接。脚本是存放在 /etc/init.d/ 目录下的，当然可以在此目录创建连接文件连接到存放在其他地方的脚本文件。

此命令可以指定脚本的执行序号，序号的取值范围是 0-99，序号越大，越迟执行。

当我们需要开机启动自己的脚本时，只需要将可执行脚本丢在 / etc/init.d 目录下，然后在 / etc/rc.d/rc_.d 文件中建立软链接即可

语法：

update-rc.d 脚本名或服务

`#1、在/etc/init.d目录下创建链接文件到后门脚本：  
ln -s /home/b4yi/kali-6666.elf /etc/init.d/backdoor  
#2、用 update-rc.d 命令将连接文件 backdoor 添加到启动脚本中去  
sudo update-rc.d backdoor defaults 99  
`

开机即执行。

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFnIrwCXpJoGkeEFMX2KiccoZBDia1jEp0r2QaksdyyEjiaGPLUYJklM0pA/640?wx_fmt=png)

入侵排查：

`more /etc/rc.local  
/etc/rc.d/rc[0~6].d  
ls -l /etc/rc.d/rc3.d/  
`

计划任务排查：

需要注意的几处利用 cron 的路径：

`crontab -l  # 列出当前用户的计时器设置  
crontab -r  # 删除当前用户的cron任务  
`

上面的命令实际上是列出了 /var/spool/cron/crontabs/root 该文件的内容：

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFKNgEClJFXZkHiaRYo6J3kwRPq0DiakZQz2EXriaPS4QOOicibSibqzDmJI5Q/640?wx_fmt=png)

*   /etc/crontab 只允许 root 用户修改
    
*   /var/spool/cron / 存放着每个用户的 crontab 任务，每个任务以创建者的名字命名
    
*   /etc/cron.d / 将文件写到该目录下，格式和 / etc/crontab 相同
    
*   把脚本放在
    

/etc/cron.hourly/、/etc/cron.daily/、/etc/cron.weekly/、/etc/cron.monthly / 目录中，让它每小时 / 天 / 星期 / 月执行一次

小技巧：

`more /etc/cron.daily/*  查看目录下所有文件  
`

入侵排查：

重点关注以下目录中是否存在恶意脚本;

`/var/spool/cron/*   
/etc/crontab  
/etc/cron.d/*  
/etc/cron.daily/*   
/etc/cron.hourly/*   
/etc/cron.monthly/*  
/etc/cron.weekly/  
/etc/anacrontab  
/var/spool/anacron/*  
`

### 入侵排查

查询已安装的服务：  
RPM 包安装的服务：

`chkconfig  --list  查看服务自启动状态，可以看到所有的RPM包安装的服务  
ps aux | grep crond 查看当前服务  
系统在3与5级别下的启动项   
中文环境  
chkconfig --list | grep "3:启用\|5:启用"  
英文环境  
chkconfig --list | grep "3:on\|5:on"  
`

源码包安装的服务:

`查看服务安装位置 ，一般是在/user/local/  
service httpd start  
搜索/etc/rc.d/init.d/  查看是否存在  
`

异常文件检查：

按照三种方式查找修改的文件：

*   按照名称
    
*   依据文件大小
    
*   按照时间查找
    

根据名称查找文件

`find / -name a.Test  
# 如果文件名记不全，可使用通配符*来补全  
# 如果不区分大小写，可以将-name 替换为-iname  
`

依据文件大小查找

`find / -size +1000M  
# +1000M表示大于1000M的文件，-10M代表小于10M的文件  
`

依据时间查找

`# -atime 文件的访问时间  
# -mtime 文件内容修改时间  
# -ctime 文件状态修改时间（文件权限，所有者/组，文件大小等，当然文件内容发生改变，ctime也会随着改变）  
# 要注意：系统进程/脚本访问文件，atime/mtime/ctime也会跟着修改，不一定是人为的修改才会被记录  
# 查找最近一天以内修改的文件：  
find / -mtime -1 -ls  | more   
# 查找50天前修改的文件：  
find ./ -mtime +50 -ls  
`

根据属主和属组查找

`-user 根据属主查找  
-group 根据属组查找  
-nouser 查找没有属主的文件  
-nogroup 查找没有属组的文件  
  
# 查看属主是root的文件  
find ./ -user root -type f  
# -type f表示查找文件，-type d表示查找目录  
# 注意：系统中没有属主或者没有属组的文件或目录，也容易造成安全隐患，建议删除。  
`

按照 CPU 使用率从高到低排序

`ps -ef --sort -pcpu  
`

按照内存使用率从高到低排序

`ps -ef --sort -pmem  
`

补充：

1、查看敏感目录，如 / tmp 目录下的文件，同时注意隐藏文件夹，以 “..” 为名的文件夹具有隐藏属性。  
2、得到发现 WEBSHELL、远控木马的创建时间，如何找出同一时间范围内创建的文件？  
可以使用 find 命令来查找，如 find /opt -iname “*” -atime 1 -type f 找出 /opt 下一天前访问过的文件。  
3、针对可疑文件可以使用 stat 进行创建修改时间。

系统日志检查：

日志默认存放位置：/var/log/  
必看日志：secure、history  
查看日志配置情况：more /etc/rsyslog.conf

`/var/log/wtmp 登录进入，退出，数据交换、关机和重启纪录  
/var/log/lastlog 文件记录用户最后登录的信息，可用 lastlog 命令来查看。  
/var/log/secure 记录登入系统存取数据的文件，例如 pop3/ssh/telnet/ftp 等都会被记录。  
/var/log/cron 与定时任务相关的日志信息  
/var/log/message 系统启动后的信息和错误日志  
/var/log/apache2/access.log apache access log  
`

![](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEZFw3nvqH2gFSQ495HbP7ibFZP1hY15Y8xDzbNFIDI68oA7UtQ5ctQ6Xf5w9QYVsuMVSpqPZ3pYT1g/640?wx_fmt=png)

日志分析技巧：

`1、定位有多少IP在爆破主机的root帐号：  
grep "Failed password for root" /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr | more  
定位有哪些IP在爆破：  
grep "Failed password" /var/log/secure|grep -E -o "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"|uniq -c  
爆破用户名字典是什么？  
grep "Failed password" /var/log/secure|perl -e 'while($_=<>){ /for(.*?) from/; print "$1\n";}'|uniq -c|sort -nr  
2、登录成功的IP有哪些：  
grep "Accepted " /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr | more  
登录成功的日期、用户名、IP：  
grep "Accepted " /var/log/secure | awk '{print $1,$2,$3,$9,$11}'   
3、增加一个用户kali日志：  
Jul 10 00:12:15 localhost useradd[2382]: new group: name=kali, GID=1001  
Jul 10 00:12:15 localhost useradd[2382]: new user: name=kali, UID=1001, GID=1001, home=/home/kali  
, shell=/bin/bash  
Jul 10 00:12:58 localhost passwd: pam_unix(passwd:chauthtok): password changed for kali  
#grep "useradd" /var/log/secure   
4、删除用户kali日志：  
Jul 10 00:14:17 localhost userdel[2393]: delete user 'kali'  
Jul 10 00:14:17 localhost userdel[2393]: removed group 'kali' owned by 'kali'  
Jul 10 00:14:17 localhost userdel[2393]: removed shadow group 'kali' owned by 'kali'  
# grep "userdel" /var/log/secure  
5、su切换用户：  
Jul 10 00:38:13 localhost su: pam_unix(su-l:session): session opened for user good by root(uid=0)  
sudo授权执行:  
sudo -l  
Jul 10 00:43:09 localhost sudo:    good : TTY=pts/4 ; PWD=/home/good ; USER=root ; COMMAND=/sbin/shutdown -r now  
`

webshell 查杀:

河马 WebShell 查杀：http://www.shellpub.com

Linux 安全检查脚本:

https://github.com/grayddq/GScan  
https://github.com/ppabc/security_check  
https://github.com/T0xst/linux

来源：https://www.jianshu.com/p/afc845cf9cc9

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPwwb5AxgeyO4QBNh18Fn6zdHoLUI5icibB4ibJKHvDsZTm7oBibUMPBk2ccibiawFdUyRsxdwsHjdAVjYuw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPwFnSZ4ST9beGd5aICibCzeudnBgkU2jxkNicmkoJOqCRpRTuZ66zKQRXahaCXcwyxugx5paBygA1aw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3heAguJrdPzLicKwibCOrj4LSdPHyjzIeCec4cT7TKYicpltRA9sjls9gnl2G8aQ2xxbEMDPklOXS9Qq1PiaWicxcjA/640?wx_fmt=png)