\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/BNtfGytjlV7NjtjwNWgArg)

案例来自某次攻防演习的真实案例中....

当大佬已经在内网漫游时，而身外菜鸡的我还在外边徘徊为拿不到 shell 而苦恼中....

决定对还未攻下的目标单位进行资产复盘看看能不能找到一些软柿子捏一捏

通过在小程序中搜索目标单位名称，发现存在一个订餐系统

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKp7Xk8JVyqoQZMuEJEuhLpPY4bRJDV0hqbI1TayEpno5OClicx3u8rKQ/640?wx_fmt=png)

Burp 抓取小程序所发送的数据包，先康康请求的目标域名先。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKQQPaxibEEgGWTBhZPJLf2ictYQRDuPODuGypNvJY8Fhysfankm3AGbeA/640?wx_fmt=png)

拿到域名后扫一波端口，发现有开放 22 443 3306 5353 6379 8009 8082 端口，尝试一波弱口令后无果。  

访问 8082 端口时返回一个 tomcat 的 404 页面

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKThIyF1Fhia1WjfJTIwznKMzdHeQwqGKyYl5DmK5Nkyial7dW45s2ia5aA/640?wx_fmt=png)

前边微信小程序在发送请求时以 /dining/api 作为 API 接口前缀，所以这里我尝试访问了一下 /dining，然后得到了如下界面。  

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKXnZmPTCS6E9RQa1kRdKfz38hYOxfoH9xlcjNicQ2aVjLNeHsXz7ULicQ/640?wx_fmt=png)

尝试爆了一波弱口令无果... 缺少点欧气，于是翻了一下页面的 js，发现存在几个接口的路径。  

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKeVqpnksXwzibfyEyaUQiaO6zicBu2DuYOAS5T5Rpy0X1XsLe68PgeZFsA/640?wx_fmt=png)

尝试看看是否可以未授权访问，尝试不带 /dining/ 和带 /dining/ 两种前缀路径进行访问，放回两种不同的结果。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKIjDNgG90CKF4IUwHG0xqHxDseNvI6icnuI7aUB96XZnOyPNAlNn14kw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKicqMYcs6PNFvEYMGozCxdjZUxwFzLElNtGpyickfj9qmDZqjfvSxRHPQ/640?wx_fmt=png)

返回了一个 **应用不匹配**, 刚开始以为是没法进行未授权访问。  

后来我回去看了一下小程序端发送的数据包，发现小程序的数据包每个请求中都带有 AppId: dining。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKvIskqD6uydMMnbED8aI1uSH2ja88hk2mBbXyXghkvNHeicEaDHt9DuA/640?wx_fmt=png)

于是我将请求包中加上了该头，这时页面返回的结果不一样，返回了 **401 - 未经授权的**，证明了加上 AppId 的头是有作用的。  

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKXgib7kqXQBzbPX8lV7KHXiaqsicMBNrwsgXutEdZrv0t7tL5o0KgcPddw/640?wx_fmt=png)

但是这里提示未经授权，意味着我们还是需要登录之后才能访问到对应的接口，回看之前的小程序数据包会发现，**数据包中使用的是 JWT 方式对用户进行认证**。  

**于是我将小程序数据包中自己的 JWT 认证所用的 token 值拷到了测试的数据包中进行重放。**

o.O???? 既然就返回了很多用户的数据，越权访问啊！有点香!!

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKnpo595HFvWYbstrpiavV7fKPukLdhVYP7MLXDpoBQ5rF1zrY6k4ByQg/640?wx_fmt=png)

但是返回的数据包里并没有用户密码之类的，只能看到用户名的格式 aabbbbb ，于是跑了一波弱口令 123456，成功进入到了后台 (**其实这里后来想一想应该可以直接用小程序端中的 JWT 认真头来直接访问后台页面...**)  

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKQgwniaYvEC3KZCmOoleGlEOapH7PFroOicTTQTyV28d6TQ6zP0LdOLmQ/640?wx_fmt=png)

但是现在登录的仅仅是个普通用户，想要登录到管理员的账户。  

因为目标系统使用的是 Angular.Js 写的前端，所以有些数据是需要存储在浏览器，所以看了下 SessionStorage。

发现存在一个 roleId，当时的思路是想替换为管理员的 roleId，然后看看请求时是否有管理员的效果。(**但是后来想想这可能不是一个正确的思路，因为浏览器在发送请求时并不会自动携带 SessionStorage 中的数据回传给服务器。**)

但是 roleId 是一串无法预测的随机数值，所以没法进行遍历拿到 id 值。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKicevSeDibGEQw9wcic8HwicYCfGQQf79picJYJaLbCsfAPPMsZBv0IHxI2w/640?wx_fmt=png)

前边已知该系统可以对接口进行越权访问，尝试构造一些和角色相关的路径进行访问，如: `/roles` `/user-role` `/user-roles` 等组合的路径，看看是否能看到 roleId 值之类的。  

当直接访问 `/role` 时直接就拿到了数据，可以看到回显的数据包中有一项为 `organizationName` 的属性值，当时并没有在回显的数据包中看到有`管理员`相关的字样时...

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByK0dfCG207bsQricLADdgd7MUxic33Tuqnqpiabbfm5z62G8FPzhvsOFLUw/640?wx_fmt=png)

于是当时想了一下，要不试一试 / admin 看看有没什么东西。(**这里真的靠猜，感觉来了...**)  

结果就直接返回了管理员账号和密码.... o.O!!!，cmd5 解密一波。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByK2JLOx7ia66Q7qIONGThXrUJx69icAia5CZBSlRsI6XLKZk2ccA9bVEz6A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKAA0UkOqGROIziaeU2g5GSiaibKe4uSEI8icXn2h8QibqwD41zwS1vCNAJEQ/640?wx_fmt=png)

有管理员账号后，摸索一番，尝试了以下思路。  

*   文件上传，无果。（响应包中有回显的路径，并且上传的文件后缀为. jsp 但是访问都是 404...)
    
*   fastjson，无果。(这个并不是一开始就知道的思路，只是之后拿 shell 拿不动，然后回来尝试的思路)
    

之后我尝试了 SQL 注入，常规语句判断，无果！但是我发现响应包中回显了一个 `orderBy` 参数，Js 发送的请求包中并没有显示有该参数的存在。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKfYjtUdvmia1ic1ae7J4FZmlnYWJ8ONwQJCopmj9Qh2oTNFJ8Sb4oW8TA/640?wx_fmt=png)

这是一个大多数后台系统都会有的一个参数，而且也是比较容易出现注入的地方。于是我手动的将数据包中加上了`orderBy=asc`的参数，然后对数据包进行了重放。  

发现报错，得到两个信息。

*   物理路径，并且显示有`xxxMapper.xml`的字样, 那用的 mybatis 框架无疑。
    
*   报错的语句提示为: `Unknown column 'material.asc'`
    

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKRuux10AtpBxUG82RaRA81sicZVkLFjK9dyghnLeNsEAiaQWeY5Qz9FDA/640?wx_fmt=png)

思考，`orderBy=XXX`，`XXX`部分是否为 SQL 可控的部分？于是我又写了一些别的值，然后看了一下数据包报错的回显。  

证实 orderBy=**XXX** 部分被并接到了 material.**XXX** 的位置上

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKsU4dQvq6lg4MrsHiaknjzxnZUibq4KOichiboDrn3J1uewt9f8NwaS2cGA/640?wx_fmt=png)

将 SQL 语句拷贝出来，然后进行构造。(先知这里不知道为什么缩进体现不出来，所以我用 notepad 截了个图)

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKWo7CftxI375AXqAx3cn9XrHQkfrGR4K2mu9mkuFbkele1L5S9VbTLw/640?wx_fmt=png)

可以看到实际上这里是用了预编译的，但是 orderby 位置没有处理好。（**同时 like 处没有使用 concat 的安全写法，个人感觉应该可能也有注入的可能。大佬勿喷，这里纯属个人瞎哔哔**）

select  count(0) from (select material.id, material.name, material.code, material.description, material.status, material.type, material.unit, material.create\_time from dining\_material material WHERE (material.name like ? ) order by material.asc) tmp\_count

这里只要构造并闭合一下即可，加上页面会回显错误信息，可以进行报错注入。

id) tmp\_count union select updatexml(1,substr(current\_user(),1,10),1)#

这里因为报错导致泄露了物理路径，所以尝试进行写 shell，无果，响应包中的报错信息提示目录只读。(忘截图了这里)

之前发现该服务器有对外开放 3306 6379 22 等端口，所以这里想的是读取配置文件中的数据，然后连接数据库。

`前边报错信息得到绝对路径前半段/WEB-INF/classes/db.properties`

`前边报错信息得到绝对路径前半段/WEB-INF/classes/redis.properties`

下边只要构造语句进行读取即可 (路径和文件名靠猜，因为大多数 java 的项目都喜欢这样命名)。

但是在读取时发现读取不到，发现 WEB-INF 路径变成了很奇怪的路径。

`_w_e_b_-_i_`

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKVz0ic35FBb9bQyxp8ibSNNlVkiavMM9fqJ2zdMwfsueItTYwf7XBxRVyA/640?wx_fmt=png)

于是进行了一段时间的尝试（痛苦面具），一开始以为是有过滤啥的。  

试了一段时间后发现将路径小写时就不会导致被转成奇奇怪怪的路径。

**但是 Linux 中是对大小写敏感的，所以这里需要修改一下注入的 payload。**

id) tmp\_count union select load\_file(concat('前边报错信息得到绝对路径前半段',upper('web-inf'),'/classes/db.properties'))#

成功读取到文件内容，3306 对外开放，直接进行连接，是 root 权限的账号。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKhh4xkiaJouhSkuibTfl3kdAdnqibgODIicmibeBIb7uaxHMhiarcK98h6xuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKQ7FNzk6x0Tm8BicGgvS9AAZqnGwmV89KkuAnffpdp2JgwNotG5hdNag/640?wx_fmt=png)

但是这里 mysql 无法写入文件 (报错信息提示目录为只读权限)，所以尝试进行 redis 的提权。  

先使用 mysql，读取 redis.properties 配置文件信息，然后连接 6379。（**这里在对 redis 提权时最好先吧原先的配置记录下来，避免改回去时忘记值是多少。**）

尝试 redis 提权姿势

*   计划任务反弹 shell，无果。（弹不回来，原因不明）
    
*   直接写 shell，能写成功。但是 jsp 马的内容第一行被加入了奇奇怪怪的字符串，导致无法正常解析。
    
*   ssh 写公钥，对方服务器没有生成公钥和私钥文件。
    
*   主从复制，感觉有点麻烦。（想留到最后没招了再尝试）
    

最后尝试的是模块加载执行命令，但是就有一个核心问题，怎么样吧模块文件传上去？

可以尝试之前发现的文件上传，但是可能会出现文件并没有落地成功的问题，所以这里我用的是 mysql 的 dumpfile 功能，将 so 文件写入到了 / tmp / 目录下。

在 gayhub 下载了相应的 exp.so 后，使用 010editor 将 so 库文件的十六进制值拷贝出来。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByK53SH4cdaVZZW7RnuPz8OyFWX0G8nETvMQNMUODbRM3PaonxicKrnhgQ/640?wx_fmt=png)

```
set @test = 0x十六进制值;
select @test into dumpfile "/tmp/1.so";


```

连接上 redis，使用以下命令加载模块，然后进行命令执行。

```
module load /tmp/1.so
system.exec "whoami;id"

```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKibGPRzHqMHQ9LaSphtQAO84GEnV6yuRNnstHEEGGsDKlRWku6xWvYQw/640?wx_fmt=png)

当我正高兴可以命令执行时，意外发生了，**redis 突然卡住，然后连接不上了，但是端口一直是通的。**  

后来通过本地模拟加载`exp.so`进行命令执行的过程后发现了问题所在，在 gayhub 上所找到的`exp.so`存在问题，当我换了另外一个`exp.so`文件之后就不会出现该问题了 (本地模拟)。

我大 E 了，再提权之前没有好好测试之前在 gayhub 上找到的`exp.so`文件，导致这种问题出现。

于是当天想了其他办法也没能拿到 shell，等着看看第二天是否服务回恢复正常。

没错，很狗血的第二天真的恢复了 (redis 上原本的一些测试数据都不见了，可能管理员重启了，才让服务恢复)，重复之前的步骤，用测试过没问题的 exp.so 文件再次的进行了提权。

最终创建用户，通过 22 端口登录到目标服务器。

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwMsjHEQuXu14t8SA3vHByKpV5icLu8207AvGiamuzTRmQPFdiaVGvrGuwgIYgBEVRbjEAfOg6YickeKg/640?wx_fmt=png)

渗透测试过程中，要经常留意一些小细节上的东西，尽可能的将所收集到的所有信息都利用起来，同时对一些较为危险的操作能在本地复现环境的建议复现一次，不要大 E 了。

作者：RichardTang，文章来源：先知社区

****扫描关注乌云安全****

![](https://mmbiz.qpic.cn/mmbiz_jpg/bMyibjv83iavz34wLFhdnrWgsQZPkEyKged4nfofK5RI5s6ibiaho43F432YZT9cU9e79aOCgoNStjmiaL7p29S5wdg/640?wx_fmt=jpeg)

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**