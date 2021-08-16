\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzUyMTA0MjQ4NA==&mid=2247495187&idx=3&sn=0226ff6c0e6495cfe71853adb8521f7d&chksm=f9e38148ce94085e5243c88e98cbffbcb96f1ff69c7ba506069717078f67417c33d88c3516ab&mpshare=1&scene=1&srcid=1019gZcPkU1yolxccHoiKkUC&sharer\_sharetime=1603067369877&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=d1b2c53fcb9e77eefc605e31d69b037f35cfaf910bd703d6aa2863e2b3ddbca798bbe75d6e0631c02a37d2f18a44a4296648ebd497b8dc75f796ee41690892b5d619c7a331d197adb32d5a7a7e7b0b060b4c963a4faade21cde03eafebb5386ee0449cf89795bc99e537afcd9b8b78ed141d8f3f4448a4022bb12b569bb5b572&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=ATDymLJWi%2BpSo1ZRNGzKyp4%3D&pass\_ticket=fNc1mNErgeHhn4jm0DcjBlD5hkXepEyD08VA%2B16wYw5QmvtETgayFa%2BrZuz3ot9i&wx\_header=0)

转发自：计算机与网络安全

  

**1、关于WAF**

  

WAF（Web Application Firewall，Web应用防火墙）是通过执行一系列针对HTTP/HTTPS的安全策略来专门为Web应用提供保护的一款产品。

  

WAF基本上可以分为以下几类。

  

**（1）软件型WAF**

  

以软件形式装在所保护的服务器上的WAF，由于安装在服务器上，所以可以接触到服务器上的文件，直接检测服务器上是否存在WebShell、是否有文件被创建等。

  

**（2）硬件型WAF**

  

以硬件形式部署在链路中，支持多种部署方式，当串联到链路中时可以拦截恶意流量，在旁路监听模式时只记录攻击不进行拦截。

  

**（3）云WAF**  

  

一般以反向代理的形式工作，通过配置NS记录或CNAME记录，使对网站的请求报文优先经过WAF主机，经过WAF主机过滤后，将认为无害的请求报文再发送给实际网站服务器进行请求，可以说是带防护功能的CDN。

  

**（4）网站系统内置的WAF**  

  

网站系统内置的WAF也可以说是网站系统中内置的过滤，直接镶嵌在代码中，相对来说自由度高，一般有以下这几种情况。

  

● 输入参数强制类型转换（intval等）。

  

● 输入参数合法性检测。

  

● 关键函数执行（SQL执行、页面显示、命令执行等）前，对经过代码流程的输入进行检测。

  

● 对输入的数据进行替换过滤后再继续执行代码流程（转义/替换掉特殊字符等）。

  

网站系统内置的WAF与业务更加契合，在对安全与业务都比较了解的情况下，可以更少地收到误报与漏报。

  

**2、WAF判断**

  

下面介绍判断网站是否存在WAF的几种方法。

  

**（1）SQLMap**  

  

使用SQLMap中自带的WAF识别模块可以识别出WAF的种类，但是如果按下面装的WAF并没有什么特征，SQLMap就只能识别出类型是Generic。

  

下面以某卫士官网为例，在SQLMap中输入以下命令，结果如图1所示。

  

sqlmap.py –u "http://xxx.com"--identify-waf--batch

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/VcRPEU1K2odJCDYibCEG1NbAjrTL4FQtzl30uhSUJFYZQHUEKtOvHYm3toAp6sYcPHGu736OcEI4LAqc1My3qJQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1  使用SQLMap识别WAF

  

可以看到识别出WAF的类型为XXX Web Application Firewall。

  

要想了解详细的识别规则可以查看SQLMap的WAF目录下的相关脚本，也可以按照其格式自主添加新的WAF识别规则，写好规则文件后直接放到WAF目录下即可。

  

**（2）手工判断**  

  

这个也比较简单，直接在相应网站的URL后面加上最基础的测试语句，比如union select 1,2,3%23，并且放在一个不存在的参数名中，本例里使用的是参数aaa，如图2所示，触发了WAF的防护，所以网站存在WAF。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/VcRPEU1K2odJCDYibCEG1NbAjrTL4FQtzNkHZcVGhqPMoia9HROAguHZ1vWBPpALOicrn5txSKIMT81X4icePhSWCg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2  WAF拦截了非法请求

  

因为这里选取了一个不存在的参数，所以实际并不会对网站系统的执行流程造成任何影响，此时被拦截则说明存在WAF。

  

被拦截的表现为（增加了无影响的测试语句后）：页面无法访问、响应码不同、返回与正常请求网页时不同的结果等。

  

**3、一些WAF的绕过方法**

  

**（1）大小写混合**

  

在规则匹配时只针对了特定大写或特定小写的情况，在实战中可以通过混合大小写的方式进行绕过（现在几乎没有这样的情况），如下所示。

  

uNion sElEct 1,2,3,4,5

  

**（2）URL编码**

  

极少部分的WAF不会对普通字符进行URL解码，如下所示。

  

union select 1,2,3,4,5

  

上述命令将被编码为如下所示的命令。

  

%75%6E%69%6F%6E%20%73%65%6C%65%63%74%20%31%2C%32%2C%33%2C%34%2C%35

  

还有一种情况就是URL二次编码，WAF一般只进行一次解码，而如果目标Web系统的代码中进行了额外的URL解码，即可进行绕过。

  

union select 1,2,3,4,5

  

上述命令将被编码为如下所示的命令。

  

%2575%256E%2569%256F%256E%2520%2573%2565%256C%2565%2563%2574%2520%2531%252C%2532% 252C%2533%252C%2534%252C%2535

  

**（3）替换关键字**  

  

WAF采用替换或者删除select/union这类敏感关键词的时候，如果只匹配一次则很容易进行绕过。

  

union select 1,2,3,4,5

  

上述命令将转换为如下所示的命令。

  

unuionion selselectect 1,2,3,4,5

  

**（4）使用注释**

  

注释在截断SQL语句中用得比较多，在绕过WAF时主要使用其替代空格（/\*任意内容\*/），适用于检测过程中没有识别注释或替换掉了注释的WAF。

  

Union select 1,2,3,4,5

  

上述命令将转换为如下所示的命令。

  

union/\*2333\*/select/\*aaaa\*/1,2,3,4,5

  

还可以使用内联注释尝试绕过WAF的检测。

  

**（5）多参数请求拆分**

  

对于多个参数拼接到同一条SQL语句中的情况，可以将注入语句分割插入。

  

例如请求URL时，GET参数为如下格式。

  

a=\[input1\]&b=\[input2\]

  

将GET的参数a和参数b拼接到SQL语句中，SQL语句如下所示。

  

and a=\[input1\]and b=\[input2\]

  

这时就可以将注入语句进行拆分，如下所示。

  

a=union/\*&b=\*/select 1,2,3,4

  

最终将参数a和参数b拼接，得到的SQL语句如下所示。

  

and a=union /\*and b=\*/select 1,2,3,4

  

**（6）HTTP参数污染**

  

HTTP参数污染是指当同一参数出现多次，不同的中间件会解析为不同的结果，具体如表1所示（例子以参数color=red&color=blue为例）。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/VcRPEU1K2odJCDYibCEG1NbAjrTL4FQtz6eJUvod8fNcdwI3Biapd3Epo9P8VQ46RqLs156k0XHjS3FibYoKry93w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

表1  HTTP参数污染

  

在上述提到的中间线中，IIS比较容易利用，可以直接分割带逗号的SQL语句。在其余的中间线中，如果WAF只检测了同参数名中的第一个或最后一个，并且中间件特性正好取与WAF相反的参数，则可成功绕过。下面以IIS为例，一般的SQL注入语句如下所示。

  

Inject=union select 1,2,3,4

  

将SQL注入语句转换为以下格式。

  

Inject=union/\*&inject=\*/select/\*&inject=\*/1&inject=2&inject=3&inject=4

  

最终在IIS中读入的参数值将如下所示。

  

Inject=union/\*,\*/select/\*,\*/1,2,3,4

  

**（7）生僻函数**

  

使用生僻函数替代常见的函数，例如在报错注入中使用polygon（）函数替换常用的updatexml（）函数，如下所示。

  

SELECT polygon（（select\*from（select\*from（select@@version）f）x））;

  

**（8）寻找网站源站IP**

  

对于具有云WAF防护的网站而言，只要找到网站的IP地址，然后通过IP访问网站，就可以绕过云WAF的检测。

  

常见的寻找网站IP的方法有下面这几种。

  

● 寻找网站的历史解析记录。

  

● 多个不同区域ping网站，查看IP解析的结果。

  

● 找网站的二级域名、NS、MX记录等对应的IP。

  

● 订阅网站邮件，查看邮件发送方的IP。

  

**（9）注入参数到cookies中**

  

某些程序员在代码中使用$\_REQUEST获取参数，而$\_REQUEST会依次从GET/POST/cookie中获取参数，如果WAF只检测了GET/POST而没有检测cookie，可以将注入语句放入cookie中进行绕过。

  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/ffq88LJJ8oPhzuqa2g06cq4ibd8KROg1zLzfrh8U6DZtO1oWkTC1hOvSicE26GgK8WLTjgngE0ViaIFGXj2bE32NA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_gif/x1FY7hp5L8Hr4hmCxbekk2xgNEJRr8vlbLKbZjjWdV4eMia5VpwsZHOfZmCGgia9oCO9zWYSzfTSIN95oRGMdgAw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

[ctf系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247493664&idx=1&sn=40df204276e9d77f5447a0e2502aebe3&chksm=f9e3877bce940e6d0e26688a59672706f324dedf0834fb43c76cffca063f5131f87716987260&scene=21#wechat_redirect)

[日志安全系列-安全日志](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494122&idx=1&sn=984043006a1f65484f274eed11d8968e&chksm=f9e386b1ce940fa79b578c32ebf02e69558bcb932d4dc39c81f4cf6399617a95fc1ccf52263c&scene=21#wechat_redirect)

[【干货】流量分析系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494242&idx=1&sn=7f102d4db8cb4dddb5672713803dc000&chksm=f9e38539ce940c2f488637f312fb56fd2d13a3dd57a3a938cd6d6a68ebaf8806b37acd1ce5d0&scene=21#wechat_redirect)

[【干货】超全的 渗透测试系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494408&idx=1&sn=75b61410ecc5103edc0b0b887fd131a4&chksm=f9e38453ce940d450dc10b69c86442c01a4cd0210ba49f14468b3d4bcb9d634777854374457c&scene=21#wechat_redirect)

[【干货】持续性更新-内网渗透测试系列文章](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494623&idx=1&sn=f52145509aa1a6d941c5d9c42d88328c&chksm=f9e38484ce940d920d8a6b24d543da7dd405d75291b574bf34ca43091827262804bbef564603&scene=21#wechat_redirect)  

[【干货】android安全系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494707&idx=1&sn=5b2596d41bda019fcb15bbfcce517621&chksm=f9e38368ce940a7e95946b0221d40d3c62eeae515437c040afd144ed9d499dcf9cc67f2874fe&scene=21#wechat_redirect)  

  

* * *

****扫描关注LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)