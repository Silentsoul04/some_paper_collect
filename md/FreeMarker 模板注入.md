> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wkixsXKHXZREHPqmhxFp9g)

一、简介

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFO9ZDGBVzx8o8TaWYPTDChFEpgt5Mo01VLHrmyFP4ERZ25kGuA4iaYz8fMjAoOeUQUu0biakaBS02otCr8rmycg/640?wx_fmt=png)

FreeMarker 是一款模板引擎, 通过 Java 类库引入, 模板文件简称为 FTL（后缀可能也为这个）。输出方式为 MVC（模型, 视图, 控制器）模式, 适用于 Web 开发框架生成 html 页面。所以此类库经常应用于 MVC 开发模式的 Java Web 程序。  

二、利用与发掘

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFO9ZDGBVzx8o8TaWYPTDChFEpgt5Mo01VLHrmyFP4ERZ25kGuA4iaYz8fMjAoOeUQUu0biakaBS02otCr8rmycg/640?wx_fmt=png)

既然简介为模板引擎, 那么就一定有可以动态利用的地方。FreeMarker 动态处理变量为 **${}** 格式, 当然还有标签格式, 这个稍后讲解。看到这里, 是不是很像 el、spel 引擎模板的解析风格, 不过确实有相似部分, 当然也有区别。接下来, 通过代码分析利用方式或常见格式：  

```
Configuration configuration = new Configuration();
String templateContent = "${1+1}";
Template tpl = new Template(null, templateContent, configuration);
StringWriter writer = new StringWriter();
tpl.process(null, writer);
System.out.println(writer);
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08ECUozE6jHDVe5pz9Nz1d5rNvESauibiaUibElTWHoQyshKMjZdF1h2Sc9CmfCJAOOFl59NiaA8G3B9Q/640?wx_fmt=png)

可以看到成功执行表达式并输出结果, FreeMarker 与 el、spel 的区别这时就体现了, 虽然检测漏洞时都可以利用类似表达式, 但是执行命令时又有不同, FreeMarker 在 process 执行时必须继承 **freemarker.template.TemplateModel**, 如下图报错所示：

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08ECUozE6jHDVe5pz9Nz1d5FaWz9yajD6qbHBAJ79IHpWjn1ayiabzJWeBAicgHjTQC6DeomXicvZJmg/640?wx_fmt=png)

那么 FreeMarker 执行命令的方式有哪些呢, 可以直接通过 TemplateModel 反推继承链, 看看哪些类中的方法存在可以利用的方式, 从上图报错中可以回溯执行的调用过程

```
at freemarker.core.Expression.eval(Expression.java:78)
at freemarker.core.MethodCall._eval(MethodCall.java:55)
at freemarker.core.Expression.eval(Expression.java:78)
at freemarker.core.Expression.evalAndCoerceToString(Expression.java:82)
at freemarker.core.DollarVariable.accept(DollarVariable.java:41)
at freemarker.core.Environment.visit(Environment.java:324)
at freemarker.core.Environment.process(Environment.java:302)
at freemarker.template.Template.process(Template.java:325)
at com.cc.demo.spel.main(spel.java:37)
```

跟踪到 MethodCall 中_eval 方法, 在继承了 TemplateModel 类时最终会执行到对应类的 exec 方法中, 这里直接贴出可利用的类（既继承 TemplateModel 类又存在 exec 方法, 方法中存在执行命令）得到：

```
freemarker.template.utility.Execute
freemarker.template.utility.ObjectConstructor
freemarker.template.utility.JythonRuntime
```

当然这是一种执行方式, 还有官方

（https://freemarker.apache.org/docs/ref_builtins_expert.html）的标签执行。

当然还有一些变量参考

（https://freemarker.apache.org/docs/ref_specvar.html）。官方文档还是很管用的。  

三、限制

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFO9ZDGBVzx8o8TaWYPTDChFEpgt5Mo01VLHrmyFP4ERZ25kGuA4iaYz8fMjAoOeUQUu0biakaBS02otCr8rmycg/640?wx_fmt=png)

既然官方已知含有危险方法并可执行, 那么就一定会有限制解决的方案, 在 2.3.17 版本中加入了 setNewBuiltinClassResolver 方法, 此方法可以限制最后的执行类, 有多种格式可以选择, 例如

```
configuration = new Configuration(Configuration.DEFAULT_INCOMPATIBLE_IMPROVEMENTS);
configuration.setDefaultEncoding(CommonConstants.DEFAULT_CHARSET_NAME);
configuration.setTemplateUpdateDelayMilliseconds(0);
configuration.setAPIBuiltinEnabled(false);
configuration.setNewBuiltinClassResolver(TemplateClassResolver.ALLOWS_NOTHING_RESOLVER);
configuration.setLogTemplateExceptions(false);
```

第一行设置 config 版本（可以忽略不设置）, 第二行设置编码格式, 第三行设置模板更新缓存延迟, 第四行设置 API 权限, 第五行设置可以加载类, 第六行设置异常 log 输出。重点在第四行与第五行, 第四行中的 api 可访问 Java Api FreeMarker 中 object_wrapper（根据环境不同, 可能存在利用方式）, 第五行中

```
ALLOWS_NOTHING_RESOLVER
```

为禁止解析任何类, 对应还有  

```
UNRESTRICTED_RESOLVE
```

等于

```
ClassUtil.forName(className)
```

和 SAFER_RESOLVER 禁止解析上述命令执行的三个类。

四、突破

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFO9ZDGBVzx8o8TaWYPTDChFEpgt5Mo01VLHrmyFP4ERZ25kGuA4iaYz8fMjAoOeUQUu0biakaBS02otCr8rmycg/640?wx_fmt=png)

上述讲解了限制, 既然有限制, 必然就要想到应对之策, 从官方的变量参考中, 找到了一些可以利用的点, 例如 **get_optional_template** 特殊变量, 官方文档中描述：

This method returns a hash that contains the following entries:

```
exists: A boolean that tells if the template was found.
```

```
include: A directive that, when called, includes the template. Calling this directive is similar to calling the include directive, but of course with this you spare looking up the template again. This directive has no parameters, nor nested content. If exists is false, this key will be missing; see later how can this be utilized with the exp!default operator.
```

```
import: A method that, when called, imports the template, and returns the namespace of the imported template. Calling this method is similar to calling the import directive, but of course with this you spare looking up the template again, also, it doesn't assign the namespace to anything, just returns it. The method has no parameters. If exists is false, this key will be missing; see later how can this be utilized with the exp!default operator.
```

变量可以包含模板文件以及 import method, 类似 PHP 中的 include 方法, 这时利用方式就有很多了, 可以发散思维到 PHP 文件包含漏洞等等, 利用方式可以参考官方, 也可更新一下使用

```
<#assign optTemp = .get_optional_template('some.ftl')>
<#if optTemp.exists>
Template was found:
<@optTemp.include />
<#else>
Template was missing.
</#if>
```

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZ7rRw3kbvpDFicsKcLcp9Q7tYiaMwLANvcHAoByTiaGaus4HBukgfIXt9g/640?wx_fmt=png)