> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_-g38T5G9Ra7Dk1E2LxVFA)

绕过安全狗文件保护+进程禁用cmd直接执行命令
-----------------------

1.当安全狗开启，文件保护，禁止使用cmd去执行命令
--------------------------

当前环境：

```
 php7  
    apache2.4  
    dog——server ：5.0
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/PZCtvaaOQSlQWIkhLYN5W6NhpaQEI49tqsK24zPxQPuHqFQZtt0GBOSGQsictyvzYicE9GSzbfy9CXFBQe18icUaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

发现一句话的模拟终端无法执行。

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

2.简单绕过，发现只有一句话中的模拟终端无法执行。几种方式：
------------------------------

1.  ### 调用php中system/shell_exec函数前台显示。
    

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

2.利用蚁剑更改终端环境

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

设置终端路径：

1.复制cmd到另外目录---安全狗的查杀是加载绝对路径的。cmd路径：C:/windows/system32/cmd.exe 复制到任意路径  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

```
`2.修改cmd名字，默认无法直接修改``C：/windows/system32/cmd.exe--需要复制出去后才能修改文件。  
`
```

2.关于cmd进程阻止。
------------

‍‍‍

  

测试安全狗5.0版本：  

php5.3       通过

php5.5及以上  未通过

‍‍‍

```
  

```

```
![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
  

```

```
php为7的时候，本地无法进行cmd运行。  
蚁剑模拟终端出错。

  

  

```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

```


测试 php为7以下，这里选php5测试


```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
本地还是无法进行cmd操作  

```

发现php5.5 也是无法进行cmd虚拟终端调用

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
php换成5.3版本发现模拟终端可以执行  

```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

```
关于修改成powershell 。测试发现和php版本有关系  
和上文相同  

```