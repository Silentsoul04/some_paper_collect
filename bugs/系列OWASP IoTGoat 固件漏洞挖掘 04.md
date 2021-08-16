> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gf-lLUPX4aRTNHMr1Fhlbg)

继续接着上一章，本章节主要从以下几个方面入手。

*   缺乏安全更新机制
    
*   使用不安全或过时的组件
    
*   配置不当的加密方式
    
*   缺乏设备管理
    

**1.   缺乏安全更新机制**  

**1.1  不安全的包更新配置**

**CVE-2020-79****82**

opkg 包管理器分叉中的错误阻止正确解析签名存储库索引中的嵌入式校验和，从而允许中间人攻击者注入任意包有效负载（未经验证安装）。

### **1.2 更新系统上的固件不安全**

##### **①  固件可否被修改**

    固件结构分析，查看固件是否加密或者做签名，能否进行伪造。

    经查看并没有对文件系统及关键文件进行签名，攻击者可以任意修改固件，达到植入后门的目的。

    网上找到修改固件脚本

```
#!/bin/sh
sudo echo "Starting..."
MKSQSHFS4='./bin/mksquashfs4'
PADJFFS2='./bin/padjffs2'
case "$1" in
'extract'|'e')
offset1=`grep -oba hsqs $2 | grep -oP '[0-9]*(?=:hsqs)'`
offset2=`wc -c $2 | grep -oP '[0-9]*(?= )'`
size2=`expr $offset2 - $offset1`
#echo $offset1 " " $offset2 " " $size2
dd if=$2 of=kernel.bin bs=1 ibs=1 count=$offset1
dd if=$2 of=secondchunk.bin bs=1 ibs=1 count=$size2 skip=$offset1
sudo rm -rf squashfs-root 2>&1
sudo unsquashfs -d squashfs-root secondchunk.bin
rm secondchunk.bin
;;
'create'|'c')
sudo $MKSQSHFS4 ./squashfs-root ./newsecondchunk.bin -nopad -noappend -root-owned -comp xz -Xpreset 9 -Xe -Xlc 0 -Xlp 2 -Xpb2 -b 256k -processors 1
sudo chown $USER ./newsecondchunk.bin
cat kernel.bin newsecondchunk.bin > $2
$PADJFFS2 $2
rm newsecondchunk.bin
;;
*)
echo 'run
"modify-firmware.sh extract firmware.bin"
You will find file "kernel.bin" and folder "squashfs-root".
Modify "squashfs-root" as you like,after everything is done,run
"modify-firmware.sh create newfirmware.bin"
And you will get a modified firmware named newfirmware.bin.
'
;;
esac
```

获得权限后可以更改固件，上传，植入自己的后门。

**注入后门方法**

①通过网页固件升级方式

②dd if = 固件 of = 文件系统分区 (绕过固件校验，获取 Shell 的前提)

**2.  使用不安全或过时的组件**  

Dnsmasq、pppd、Linux Kernel、BusyBox

*   Dnsmasq
    

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQzWSK1U18EFk6uib4ZklpAeicfibDnLDq1WpWMgdFYXaDicU9TZ3uu8TxUndjD77ic3u8G0q5DSLrn1KA/640?wx_fmt=png)

*   pppd
    

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQzWSK1U18EFk6uib4ZklpAewWRUSAtJqPoq23RZCddRibTQXA8Wjpwdz8QYcnMozW7DmCAzjliae1BA/640?wx_fmt=png)

*   kernel
    

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQzWSK1U18EFk6uib4ZklpAeGP1Clp1CsCOOBRZYh0DnOFz5fFXIXBSGvbrfrnSC0IicBY17RhZ2yFw/640?wx_fmt=png)

*   busybox
    

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQzWSK1U18EFk6uib4ZklpAeIbtAbqHA1n9HYBYzjbIoqBH0JAl47UYsOU6q6iaRL601bQx0BCQxquw/640?wx_fmt=png)

    查看对应程序版本

*   Dnsmasq(2.73) 缓冲区溢出漏洞
    
*   pppd(2.4.7) CVE-2020-8597~ppp 2.4.2 到 2.4.8 中 pppd 中的 eap.c 在 eap_request 和 eap_response 函数中存在 rhostname 缓冲区溢出
    
*   Linux kernel(4.14.95) CVE-2018-7566、CVE-2018-9858、CVE-2018-9865 等漏洞
    
*   Busybox(1.28.4) CVE-2018-1000500 wget 缺少 SSL 证书验证漏洞，该漏洞可导致任意代码执行
    

**3.  隐私保护不足**

### **3.1 网络流量隐私**

网路信息监控和窃取 -- 劫持流量

路由器安装 tcpdump

```
$ opkg install tcpdump
```

路由器中抓包获取一些敏感信息

```
ssh root@192.168.72.132 "tcpdump -i br-lan -s 0 -w -" | wireshark.exe -k -i -
```

登录界面使用 https，登录数据加密，导致无法获取数据 (尝试私钥导入，发现需要密码，目前还没找到较好的解决方案)

http 协议流量可以直接明文获取相关信息 (以下是用户常访问的网站 dns 解析报文)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQzWSK1U18EFk6uib4ZklpAeSnibDaXxM03f1zSakGy2TicjSicUmkABA4ibtnL1F5U1opuJWrsMBZPxhA/640?wx_fmt=png)

### **3.2 数据库隐私**

    查看本地有一个 sqlite 数据库并没有对该数据库文件进行加密 (sensordata.db)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnQzWSK1U18EFk6uib4ZklpAetjQ6KWs3GKIORhBc4uwJWsNo86vtyPDl4gB6fXb9zUDpx69EhP7eew/640?wx_fmt=png)  

**包含姓名、邮箱、出生日期等敏感信息**  

**3.3  缺乏设备管理**

**系统日志、监控或审计功能未启用**

没有查看系统命令 **logread** 以及系统日志进程 (syslogd,logd)

**总结**

    IoTGoat 漏洞挖掘序列文章在这就基本结束了。

    有感兴趣的小伙伴，可以尝试看看该固件还有哪些漏洞还没发现。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnTqMVczDE3GyGU1hPA7RQQlIESOibcZaWMeJVMicz1JUKnoSKhomypNO0J7q4BAxqjgxmpWYYe17ia2A/640?wx_fmt=png)

如果您有意向加入我们，请留言: )，

或邮件投递简历：akast@hillstonenet.com