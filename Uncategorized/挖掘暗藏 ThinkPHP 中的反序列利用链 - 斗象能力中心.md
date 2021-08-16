\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.riskivy.com\](https://blog.riskivy.com/%e6%8c%96%e6%8e%98%e6%9a%97%e8%97%8fthinkphp%e4%b8%ad%e7%9a%84%e5%8f%8d%e5%ba%8f%e5%88%97%e5%88%a9%e7%94%a8%e9%93%be/)

**前言**
------

在 Blackhat2018，来自 Secarma 的安全研究员 Sam Thomas 讲述了一种攻击 PHP 应用的新方式，使用 phar 伪协议可以在不使用 unserialize() 函数的情况下触发 PHP 反序列化漏洞，极大地扩展了 PHP 反序列化的攻击面并且开源了新工具 PHPGGC,PHPGGC 可以针对十数个 PHP 流行框架进行了反序列化利用链输出。

于是本文由此对中国最流行的 PHP 框架之一 Thinkphp 进行了反序列化利用链挖掘。

**预备知识**
--------

#### 1.PHP 反序列化原理

PHP 反序列化就是在读取一段字符串然后将字符串反序列化成 php 对象。

#### 2\. 在 PHP 反序列化的过程中会自动执行一些魔术方法

<table><thead><tr><th>方法名</th><th>调用条件</th></tr></thead><tbody><tr><td>__call</td><td>调用不可访问或不存在的方法时被调用</td></tr><tr><td>__callStatic</td><td>调用不可访问或不存在的静态方法时被调用</td></tr><tr><td>__clone</td><td>进行对象 clone 时被调用，用来调整对象的克隆行为</td></tr><tr><td>__constuct</td><td>构建对象的时被调用；</td></tr><tr><td>__debuginfo</td><td>当调用 var_dump() 打印对象时被调用（当你不想打印所有属性）适用于 PHP5.6 版本</td></tr><tr><td>__destruct</td><td>明确销毁对象或脚本结束时被调用；</td></tr><tr><td>__get</td><td>读取不可访问或不存在属性时被调用</td></tr><tr><td>__invoke</td><td>当以函数方式调用对象时被调用</td></tr><tr><td>__isset</td><td>对不可访问或不存在的属性调用 isset() 或 empty() 时被调用</td></tr><tr><td>__set</td><td>当给不可访问或不存在属性赋值时被调用</td></tr><tr><td>__set_state</td><td>当调用 var_export() 导出类时，此静态方法被调用。用__set_state 的返回值做为 var_export 的返回值。</td></tr><tr><td>__sleep</td><td>当使用 serialize 时被调用，当你不需要保存大对象的所有数据时很有用</td></tr><tr><td>__toString</td><td>当一个类被转换成字符串时被调用</td></tr><tr><td>__unset</td><td>对不可访问或不存在的属性进行 unset 时被调用</td></tr><tr><td>__wakeup</td><td>当使用 unserialize 时被调用，可用于做些对象的初始化操作</td></tr></tbody></table>

#### 3\. 反序列化的常见起点

\_\_wakeup 一定会调用

\_\_destruct 一定会调用

\_\_toString 当一个对象被反序列化后又被当做字符串使用

#### 4\. 反序列化的常见中间跳板:

\_\_toString 当一个对象被当做字符串使用

\_\_get 读取不可访问或不存在属性时被调用

\_\_set 当给不可访问或不存在属性赋值时被调用

\_\_isset 对不可访问或不存在的属性调用 isset() 或 empty() 时被调用

形如 $this->$func();

#### 5\. 反序列化的常见终点:

\_\_call 调用不可访问或不存在的方法时被调用

call\_user\_func 一般 php 代码执行都会选择这里

call\_user\_func\_array 一般 php 代码执行都会选择这里

#### 6.Phar 反序列化原理以及特征

phar:// 伪协议会在多个函数中反序列化其 metadata 部分  
受影响的函数包括不限于如下:

```
copy,file\_exists,file\_get\_contents,file\_put\_contents,file,fileatime,filectime,filegroup,
fileinode,filemtime,fileowner,fileperms,
fopen,is\_dir,is\_executable,is\_file,is\_link,is\_readable,is\_writable,
is\_writeable,parse\_ini\_file,readfile,stat,unlink,exif\_thumbnailexif\_imagetype,
imageloadfontimagecreatefrom,hash\_hmac\_filehash\_filehash\_update\_filemd5\_filesha1\_file,
get\_meta\_tagsget\_headers,getimagesizegetimagesizefromstring,extractTo


```

(Thinkphp 框架中暂未发现, 略有遗憾)

**漏洞挖掘**
--------

#### 1\. 安装 Thinkphp 5.1.37 环境

首先去 github 下载 Thinkphp 的源码, 现在 Thinkphp 已经分为 2 个部分,  
https://github.com/top-think/framework/tags  
https://github.com/top-think/thinkphp/tags  
下载 5.1.37(最新版) 对应的版本号  
将 framework 改名为为 thinkphp 放到 think-5.1.37 中  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/99f4996dc97731de0cf348841cf347c5.png)

#### 2\. 寻找反序列化的起始点

使用 idea 打开该文件夹, 开启 xdebug  
直接 Ctrl+Shift+F 搜索 “\_\_destruct(” 看到此处有其他方法调用, 我们继续跟进  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/476d582a2006fec44984117828f14c20.png)  
发现 Windows->removeFiles(); 中使用了 file\_exists 方法, 而且 $files 可控

```
class Windows extends Pipes
{

 /\*\* @var array \*/
 private $files = \[\];
.....
public function \_\_destruct()
{
 $this->close();
 $this->removeFiles();
}
/\*\*
 \* 删除临时文件
 \*/
private function removeFiles()
{
 foreach ($this->files as $filename) {
 if (file\_exists($filename)) {
 @unlink($filename);
 }
 }
 $this->files = \[\];
}


```

查看 file\_exists 的定义可以知道,$filename 会被当做字符串处理, 那么 $filename->\_\_toString() 方法就会被调用  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/cf745a643c4ff7c4679ef2db47e925c8.png)

#### 3\. 寻找反序列化的中间跳板

下面就要求寻找一个实现了\_\_toString() 方法的对象来作为跳板  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/5e18f0a0252d3620f011fcdd550a8401.png)

此处 thinkphp\\library\\think\\model\\concern\\Conversion.php 存在跳板可能

```
trait Conversion
{
 protected $visible = \[\];
 protected $hidden = \[\];
 protected $append = \[\];
....

public function \_\_toString()
{
 return $this->toJson();
}
.....

public function toJson($options = JSON\_UNESCAPED\_UNICODE)
{
 return json\_encode($this->toArray(), $options);
}
.......


```

toArray() 函数中寻找一个满足条件的:  
$ 可控变量 -> 方法 (参数可控)  
这样可以去触发某个类的\_\_call 方法,  
找到符合条件的一处, 其中 “$relation” 和 “$name” 都是可控变量,$name 需要为数组

`$relation->visible($name);`

![](https://blog.riskivy.com/wp-content/uploads/2019/07/bfe49d8c75a401e64a49d79849dc4b6d.png)

#### 4\. 寻找反序列化代码执行点

下面我们需要寻找一个类满足以下 2 个条件

1\. 该类中没有”visible” 方法

2\. 实现了\_\_call 方法

直接查找 “public function \_\_call”

一般 PHP 中的\_\_call 方法都是用来进行容错或者是动态调用, 所以一般会在\_\_call 方法中使用

\_\_call\_user\_func($method, $args)

\_\_call\_user\_func\_array(\[$obj,$method\], $args)  
但是 public function \_\_call($method, $args) 我们只能控制 $args, 所以很多类都不可以用  
经过查找发现 think-5.1.37/thinkphp/library/think/Request.php 中的 \_\_call 使用了一个 array 取值的

```
public function \_\_call($method, $args)
{
 if (array\_key\_exists($method, $this->hook)) {
 array\_unshift($args, $this);
 return call\_user\_func\_array($this->hook\[$method\], $args);
 }

 throw new Exception('method not exists:' . static::class . '->' . $method);
}


```

这里的 $hook 是我们可控的, 所以我们可以设计一个数组 $hook= {“visable”=>” 任意 method”}  
但是这里有个 array\_unshift($args, $this); 会把 $this 放到 $arg 数组的第一个元素这样我们只能

```
call\_user\_func\_array(\[$obj,"任意方法"\],\[$this,任意参数\])
也就是
$obj->$func($this,$argv)


```

如下图  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/d1a2c3268ea960ab09e7950fe3deaaf4.png)  
这种情况是很难执行命令的, 但是 Thinkphp 作为一个 web 框架,  
Request 类中有一个特殊的功能就是过滤器 filter(ThinkPHP 的多个远程代码执行都是出自此处)  
所以可以尝试覆盖 filter 的方法去执行代码  
寻找使用了过滤器的所有方法  
发现 input() 函数满足条件  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/b83f72d62291d3b2df8180389bcd9d98.png)

```
    public function input($data = \[\], $name = '', $default = null, $filter = '')
    {
        if (false === $name) {
            // 获取原始数据
            return $data;
        }

        $name = (string) $name;
        if ('' != $name) {
            // 解析name
            if (strpos($name, '/')) {
                list($name, $type) = explode('/', $name);
            }

            $data = $this->getData($data, $name);

            if (is\_null($data)) {
                return $default;
            }

            if (is\_object($data)) {
                return $data;
            }
        }

        // 解析过滤器
        $filter = $this->getFilter($filter, $default);

        if (is\_array($data)) {
            array\_walk\_recursive($data, \[$this, 'filterValue'\], $filter);
            if (version\_compare(PHP\_VERSION, '7.1.0', '<')) {
                // 恢复PHP版本低于 7.1 时 array\_walk\_recursive 中消耗的内部指针
                $this->arrayReset($data);
            }
        } else {
            $this->filterValue($data, $name, $filter);
        }

        if (isset($type) && $data !== $default) {
            // 强制类型转换
            $this->typeCast($data, $type);
        }

        return $data;
    }



```

但是这个方法不能直接使用,$name 是一个数组 (由于前面判断条件 is\_array($name)),`(string)$name` 会报错终止程序, 所以不能直接使用这个函数  
继续查找调用 input 方法的的函数  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/3c012d10f2ae62748e73de6408badc39.png)  
这里发现一个函数 public function param($name =”, $default = null, $filter = ”), 如果能满足 $name 为字符串, 就可以控制变量代码执行了  
所以继续向上查找使用了 param 的方法  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/46fbba82650acf18c0a53b0526f68cb4.png)  
但是 PHP 有个特性, 一个函数可以接收任意数量参数, 超出的部分可以自动忽略  
这里就发现 isAjax/isPjax 方法可以满足 param 的第一个参数为字符串, 因为 $this->config 也是可控的

```
    public function isAjax($ajax = false)
    {
        $value  = $this->server('HTTP\_X\_REQUESTED\_WITH');
        $result = 'xmlhttprequest' == strtolower($value) ? true : false;

        if (true === $ajax) {
            return $result;
        }

        $result           = $this->param($this->config\['var\_ajax'\]) ? true : $result;
        $this->mergeParam = false;
        return $result;
    }


```

#### 5\. 构造反序列化利用链

攻击链如下图所示  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/5fa463f1eb8b8c17f8066f9700c11596.png)

#### 6\. 漏洞利用条件

使用的 ThinkPHP 5.1.X 框架的程序中, 满足以下任意条件:  
1\. 未经过滤直接使用反序列化操作  
2\. 可以文件上传且文件操作函数的参数可控，且:、/、phar 等特殊字符没有被过滤  
![](https://blog.riskivy.com/wp-content/uploads/2019/07/ce2e441b17933256e9d3ae0f527c5c13.png)

POC:  
此漏洞仅影响 Thinkphp 5.1.X

```
漏洞还未修复，已提交至官方，poc暂不披露。


```

**参考**
------

https://github.com/ambionics/phpggc  
https://www.cnblogs.com/iamstudy/articles/thinkphp\_5\_x\_rce\_1.html  
https://www.cnblogs.com/iamstudy/articles/unserialize\_in\_php\_inner\_class.html  
https://p0sec.net/index.php/archives/114/  
https://paper.seebug.org/680/

作者：斗象能力中心 TCC – 小胖虎