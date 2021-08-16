\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/H3No6o4FPVwWyJzekGumcw)

这是 **酒仙桥六号部队** 的第 **94** 篇文章。

全文共计3006个字，预计阅读时长11分钟。

  

**前言**

在hw期间和客户聊天的时候，听到客户说他们在外网还开了一个禅道项目管理系统，但是在hw的期间关闭了，hw过了在开启。因为开在外网，而且禅道管理系统以前就爆过一些漏洞，于是询问客户是多少版本的禅道，客户说：不知道是多少版本的，反正开了几年了。几年······那就能很肯定是老版本了，于是协助客户帮忙测试了一下。

  

**查看版本**

Url: index.php?Mode=getconfig

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56uUrnO9K94ALDNF2gl5MeFib3cGSvvVb9tiaeoqag4zRSdI71u8xicn9ricnrbwlQg5GcEzfzoetu7iag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

源码分析：./index.php

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56uUrnO9K94ALDNF2gl5MeFCyf711zYxJfGtPOwvRk97KsWzdIq5fia7g1GXpE4xWicWq7CYe65RawQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56uUrnO9K94ALDNF2gl5MeFMohSclhpUDz2YGDrP0Yf6sQE1aqeJ2eje24IrN3wv9pq2Q5KCW9A0A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们成功获取到禅道系统的版本是7.3版，于是百度找一下7.3版本的历史漏洞。

  

**快速验证**

发现该版本存在一个前台sql注入漏洞，为了快速验证漏洞，给客户展示危害，直接利用网上的payload打过去：

```
Url: /block-main.html?mode=getblockdata&blockid=task&param=payload2
```

Payload2的值用如下代码生成：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

测试的时候，发现并没有任何反应，页面空白，开启debug模式再尝试：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

再一次测试payload，依然是空白页面，第一次测试失败告终。

为了验证漏洞的准确性，我用自己的电脑下载了一个7.3版本的环境，依然同上步骤进行测试，结果如下：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以看到，成功进行了报错，证明了这个漏洞还是真实存在的，但是为什么在客户系统上就是空白页面呢？

  

**问题排查**

这里猜想可能是因为打了补丁或者被安全软件拦截了已知exp导致了在客户系统上利用失败。

这里选择先从打补丁方向排除问题，通过搜索发现官方发布了7.3版本的漏洞修复补丁：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

修复后的证明就是在根目录下会存在一个ok.txt文件，客户系统上就存在一个ok.txt，证明漏洞已经修复了。

  

**漏洞及补丁分析**

#### 漏洞分析：

在block模块的main方法中进行了动态调用，而且将我们输入的数据赋值给了$this->params属性，方便在后续的调用中使用，代码如图：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

当我们通过url传入参数：?blockid=task的时候，就会去调用printtaskBlock()方法：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以看到函数将我们输入的数据作为第二个参数传递进了getUserTasks方法中去，查看该方法代码：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

看到很直白的就将我们输入的数据作为了字段拼接进了sql语句中去，造成了sql注入漏洞。

#### 补丁分析：

先对比一下补丁，尝试能不能绕过补丁再次进行注入：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

修复后的代码中添加了一个\_\_construct方法，并且进行了一个if(!$this->loadModel('sso')->checkKey()) die(''); 判断，如果条件满足就会进行退出操作，跟进查看checkkey()方法，看看是怎么判断的：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

判断$this->config->sso->turnon是否有值，如果没值就会进行退出，在这里调试输出看看是否存在值：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

调试发现这里默认是不存在值的，这里的功能是为了实现禅道和然之系统的集成，只有当你在后台集成了然之系统，这里才会有值，默认是不会集成的，所以这里也就是空数据，这也就导致我们没办法再访问block这个模块了。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

因为不能访问block模块了，所以其他版本在该模块里面的注入一概失败。

  

**另起灶炉**

既然网上爆出来的历史漏洞不能利用，那么尝试一下能不能挖掘到新的漏洞来证明其危害。因为禅道管理系统对每个用户的权限都进行了划分管理，大部分的功能都需要登录后才能访问，不需要登陆就可以访问的方法在isOpenMethod方法中定义：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里定义的方法不需要登录就可以访问，但是block模块进行了再次验证，以往存在高危漏洞的block模块就排除了，然后查看其他的方法中，发现并没有一个方法可以在不登录时造成高危漏洞，但是我们也不能直接放弃，给客户说系统绝对安全，没有问题。

#### 退而求其次：

因为没有不需要登录就能造成漏洞的地方，所以再次挖掘登录后能getshell的漏洞，因为这是项目管理系统，必然会存在很多用户账号，有的用户可能安全意识没有那么强，可能使用弱密码等，只要我们能获取到其中任意一个账号后能造成getshell漏洞，也能证明其系统存在一定风险性。

所以这里的目标就是挖掘一个任意账户的getshell漏洞。

先看看禅道管理系统用户等级共划分了11个等级：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

那么我们挖掘的漏洞就尽量等级越低越好，最好是guest组的用户也可以造成getshell，这样我们随便获取到一个能登录的账号就可以造成gehshell。

每个组用户对应的访问权限储存在zt\_grouppriv数据表中的：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Group=11代表的就是guest组的权限。

对于这种严格划分了权限的系统，我们可以尝试两种方法：

第一种：就是对照着每个组的权限，挨个查看其方法是否能造成漏洞，这种办法有点笨重，花费时间大，但是比较全面。

第二种：先查找造成漏洞的地方，然后再去查找对应的访问权限，这种办法相对灵活高效。

#### 漏洞挖掘：

我这里就使用的第二种方法，先去查找造成漏洞的地方，再去查找对应的权限。

直接通过搜索高危函数来进行初步定位：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

先假设这里的import方法直接就可以任意文件上传，但是最关键的是我们还需要验证guest组是否存在访问权限，通过查看zt\_grouppriv表来确定：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以看到并没有testcase模块的访问权限，遂放弃该模块，因为就算这个方法能造成漏洞，但是我们的guest组用户也没有权限操作，查看其他的模块：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以看到只有file模块，extension模块，testcase模式（上面已排除）这三个模块中存在上传等操作，依旧先对照zt\_grouppriv权限表，确定权限。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

通过对照zt\_grouppriv权限表发现这里只有file模块下的download存在权限，其他的方法都没有权限吗？

#### 柳暗花明：

当我在查看file模块的时候却发现存在这样的一个方法：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

file模块中存在一个名字叫做ajaxUpload()的方法，虽然我们在zt\_grouppriv权限表中没有设定对这个方法的访问权限，但是我们回过头去看最开始的不需要登录就可以访问的方法列表，isOpenMethod方法：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

该方法中设定了如果$method方法中存在ajax字符串，即可以跳过后续的验证直接访问执行，然后我们分析一下上面的if判断语句：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以看到只要有用户登录，而且用户登录名不是guest，（这里是账号名字，不是账号组），那么就可以满足条件。为了验证一下访问权限，这里创建了一个guest组的账号，然后利用该账号进行登录，登录成功后访问url: /file-ajaxupload

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

既然我们guest组的账号可以访问到该方法，接下来就该分析一下上传代码了。

#### 绕过黑名单上传：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里上传的最终文件名来自于$this->file->getUpload(‘imgFile’)的返回结果。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里关键的getExtension方法获取并且验证后缀。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以很容易看到这里采用的黑名单的验证模式，在Windows下，我们可以通过123.php::$DATA ，1.php\[\\x81-\\x99\]等文件名来绕过黑名单模式，达到上传php文件造成getshell漏洞。

#### 漏洞验证：

1.  注册一个任意组的账号并登陆。
    
2.  使用如下exp，上传一个文件名为1.php::$DATA的shell文件。
    
    Exp.html
    

```
`<!DOCTYPE html>``<html lang="en">``<head>` `<meta charset="UTF-8">` `<title>禅道9.2及之前getshell</title>``</head>``<body>``<form action="http://10.70.132.34:81/zentao/file-ajaxUpload" method="post" enctype="multipart/form-data">` `<input type="file" >` `<input type="submit" >``</form>``</body>``</html>`
```

利用结果如图所示：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

成功绕过了黑名单验证，进行了上传php文件，访问的时候，忽略掉最后的::$DATA即可。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

到这里我们就成功挖掘到了一个任意用户组的getshell漏洞，该漏洞影响版本<=9.2版本，在9.3中对上传使用了白名单验证。同时也给客户展示了打过补丁的低版本禅道系统开放在外网仍然存在一定的风险性，然后建议在不影响业务的情况下，还是尽量将系统放在内网中使用，因为难免会在长时间开放在外网的情况下，某用户的账号密码泄露导致系统被getshell。

  

**总结**

1.  最开始得知系统是低版本的禅道系统。
    
2.  到直接利用“前台sql注入漏洞”的payload进行验证的失败。
    
3.  在排查问题中发现是因为系统已经打过补丁了。
    
4.  然后再分析漏洞原理和补丁代码 尝试绕过补丁进行注入。
    
5.  绕过补丁失败，导致另起灶炉挖掘新的漏洞。
    
6.  在理清楚系统的用户组等级划分和权限控制后，结合前台开放方法进行组合利用。
    
7.  利用windows特性进行bypass黑名单上传到达getshell的目的。
    

整个流程走下来，从最开始的信息初探，到利用已知漏洞失败，然后查找问题，到最后自己挖掘新的漏洞，在这个过程中还是学习到了很多知识，也了解到了禅道系统，希望能通过一次次曲折的问题来快速提高能力。

  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)