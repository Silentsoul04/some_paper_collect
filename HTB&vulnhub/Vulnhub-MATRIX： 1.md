> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/x_cxJUK5b_yPs5T4Bi8EDw)

大家好，我们是想要为亿人提供安全的亿人安全，这是我们自己想要做的事情，也是做这个公众号的初衷。希望以干货的方式，让大家多多了解这个行业，从中学到对自己有用的知识。

**靶机描述：**

下载地址：http://www.vulnhub.com/entry/matrix-1,259/

级别：中级  

目标：得到 root 权限和打开 flag.txt

扫描靶机 ip

```
arp-scan -l
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LsYZ8BVyrGRlMGQDOsIFhH3Nzx7NZBGHLNncOtvJiaG8JpWUQEtsAt7Q/640?wx_fmt=png)

```
攻击机：192.168.86.138
靶机：192.168.86.162
```

查看开放端口及版本信息

```
nmap -sV -p- 192.168.86.162
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LBJosKiajgEDIVrPGoCSicXsfJojLW1ELHuFRAnA6K5Gqm0ibKWsq8vZtQ/640?wx_fmt=png)

访问 80 端口页面，Follow the White Rabbit(跟随白色兔子)，Welcome to the real world, Neo. I'm glad you're here(欢迎来到真实世界，尼奥。很高兴你能在这里)。(ps~ 黑客帝国帝国的片段)

```
http://192.168.86.162:80
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6L6LZnDEhm2drCax5faF5jfyCWL0XaoB6iafNIstESTYrOcPyp21G1heA/640?wx_fmt=png)

查看网页源码时发现发现图片链接

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LHoNEpoicQEQX6VAEtoHXTiasD48hErkASKz39z0ibHhAInIueicW0yqVtA/640?wx_fmt=png)

打开后是一只白兔，并图片命名为 31337.png，根据提示那重要信息应该在 31337 端口中

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LujOh5xSicbmX6abQVh2wibx78ZgaQpyYQTBoVSqdnPU0uNEDIYiamLuGw/640?wx_fmt=png)

打开 31337 端口页面，Cypher(暗号)，"You know.. I know this steak doesn't exist. I know when I put it in my mouth; the Matrix is telling my brain that it is juicy, and delicious. After nine years.. you know what I realize? Ignorance is bliss."(“我知道这块牛排不存在。我知道当我放进嘴里时，矩阵正在告诉我的大脑它是美味多汁的。九年之后，你知道我意识到什么了吗？无知是幸福的。”)

```
http://192.168.86.162:31337
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6L3PdCJge7l4U19kYoVXEdQ2qyqhWFpouSFs0F4Sw1UiaydWZYtGqU2iaw/640?wx_fmt=png)

查看网页源码，找到用 base64 编码的一段话

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6L5sxrnorayo5xLLNEgloS4ia8PSoOBjHDxnTicIeUxbRXiaRJF3DJM6hog/640?wx_fmt=png)

解码后，Then you'll see, that it is not the spoon that bends, it is only yourself.>Cypher.matrix(你将会看见，弯曲的不是汤勺，而是你的信念。)

```
https://base64.us/
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LwbPHc4GcXfDIWNFoTaZ5icR5nEbDH1iawvkQctqGOzlexmpicoN5qCFgw/640?wx_fmt=png)

尝试以 / Cypher.matrix 为路径打开网页，出现了下载文件提示

```
http://192.168.86.162:31337/Cypher.matrix
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LF9AxxJrLFuXp6s445fcNudOjFBKu5VFjqZsYric8NyPv2uicib1oCuDGg/640?wx_fmt=png)

下载文件查看，文件内容使用 brainfuch 编码加密，进行解码后得到提示，You can enter into matrix as guest, with password k1ll0rXX Note: Actually, I forget last two characters so I have replaced with XX try your luck and find correct string of password.(你可以以 guest 进入到矩阵，密码为 k1ll0rXX 提示：事实上，我忘记了最后两个字符，所以我用 XX 替代了它们，你够幸运的话去找到正确的密码字符串)

```
https://www.splitbrain.org/services/ook
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LgrTXNRUeUDbiariaz7HpXUgeSfUHdCVwByAiboLrHJQjeibwOJJ2zNLPJg/640?wx_fmt=png)

使用 crunch 字典生成工具生成密码字典，选择要使用的字符集范围，选择使用 lalpha-numeric 字符集

```
文件路径：/usr/share/crunch/charset.lst
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6L5FjxK0GuNfUxBUWxPsEiaK5kNSGB3VcViaDia0uS9QDDoMQG2fU0KrJcA/640?wx_fmt=png)

生成字符字典

```
crunch 8 8 -f /usr/share/crunch/charset.lst lalpha-numeric -t k1ll0r@@ > pwd.dic

8 8:表示密码的长度为8位
-f /usr/share/crunch/charset.lst lalpha-numeric:指定字符集
-t k1ll0r@@:指定模式，@@插入小写字符
> pwd.dic:生成字典文件
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6L3oPQRssAkPDJKt2cWDEC8ytwJVzB9x9UkpqyvgqdtFFTOsGaj6xNDA/640?wx_fmt=png)

得到字典文件后使用 hydra 通过连接 ssh 进行密码破解

```
hydra -l guest -P pwd.dic  -v 192.168.86.162 ssh 

-l:指定用户
-P:指定字典
-v:显示爆破详细信息
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LK0RcgGTetlGTuCKxsIErCEgicrwPvSyQGFR0pzSltiaUzv1D4BFYzh1A/640?wx_fmt=png)

登录 ssh 成功

```
ssh guest@192.168.86.162
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6L5A2aTicXF3iaFyjJh3tHBW1U1vTr4TH7TNu4vmTOtqC8HXzHPaibS7C9Q/640?wx_fmt=png)

由于 bash shell 功能受限，所以采取 rbash 绕过方式

```
参考文章：
https://www.freebuf.com/articles/system/188989.html
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LVdevtooUqwhXlfhqN1UeQhGPZxYmsUyhWyiaaLdEbnc6RxgFuiciaq21g/640?wx_fmt=png)

重新登录 ssh，成功绕过 rbash

```
ssh guest@192.168.86.162  -t "bash --noprofile"
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LrmEQgd3P2aGFNduPsQzlibDVwiaHDic5JDltblr0pfrOjogd9x8haNnxw/640?wx_fmt=png)

使用命令查看目前用户权限

```
sudo -l
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LuVQzicGVhCSq7G2oFwLzP0BfsYB1SZukU65ZToHRXSibFzY0Tfeje7Hg/640?wx_fmt=png)

成功切换到 root 用户，得到 flag.txt

```
sudo su root
```

![](https://mmbiz.qpic.cn/mmbiz_png/iar31WKQlTTpBDLn4FXltlicYMjmka2S6LEibh7WvuC3t946Ch0ZKssZiauBZa1cXqic2VsE2NSmR8sWcOAIBsyROTA/640?wx_fmt=png)