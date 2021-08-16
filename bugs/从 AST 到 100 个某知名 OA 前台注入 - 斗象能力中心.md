\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.riskivy.com\](https://blog.riskivy.com/%e4%bb%8east%e5%88%b0100%e4%b8%aa%e6%9f%90%e7%9f%a5%e5%90%8doa%e5%89%8d%e5%8f%b0%e6%b3%a8%e5%85%a5/)

> 2019 年 2 月在写这篇文章 [挖掘暗藏 ThinkPHP 中的反序列利用链](https://blog.riskivy.com/%e6%8c%96%e6%8e%98%e6%9a%97%e8%97%8fthinkphp%e4%b8%ad%e7%9a%84%e5%8f%8d%e5%ba%8f%e5%88%97%e5%88%a9%e7%94%a8%e9%93%be/) , 寻找 PHP 反序列化的`POP Chain`时, 我就在想这种纯粹的体力劳动可不可以更现代化一点, 不仅仅是`Ctrl+Shift+F`这种机械重复的体力劳动, 当时了解了一些相关的项目 / 论文, 包括不限于`Navex`, `Prvd`, `Cobra`, `Codeql`. 鉴于 Cobra 代码开源, 也相对简单, 后来有一阵子某知名 OA 漏洞爆发, 于是参考了`Cobra`的`PHP Parser`尝试实现一个通过遍历 Java AST(抽象语法树) 进行漏洞挖掘的工具, 没想到效果出奇的好, 筛选出 160 个前台注入点, 手工编写了约 50 个前台注入 EXP.
> 
> 文中涉及的漏洞均为`workflowcentertreedata`通告的相似漏洞研究, [补丁版本](https://www.weaver.com.cn/cs/package/Ecology_security_20200102_v10.24.zip)之后均已失效

预备知识
----

### 某知名 OA 介绍

某知名 OA 是使用 Java 编写的一个 OA 套件, 代码相对古老, 其中 sql 查询语句多是拼接, 且代码中没有过滤, 其过滤是通过统一的`Filter`实现的, 存在一些绕过的情况.

某知名 OA 的主体功能是通过 JSP 实现的, 这里是目前只有`PMD`支持解析, 但是没有尝试, 从 idea 的的解析结果来看, 大概是解析不到具体函数逻辑的, 好在 JSP 可以编译成`Java Servlet`, 某知名 OA 使用的`Resin Server` 也会缓存编译好的`Java Servlet`, 这里倒是省了不少麻烦.

### 编译原理基础

了解过编译原理的同学都知道, 一般语言的编译都是通过 `词法分析`,`语法分析`, 然后解析成`AST(抽象语法树)`, 这里包含了一个程序源文件的所有结构化信息, 通过遍历 AST 的方式, 我们可以精确的取出我们需要的信息, 而不是笨拙的使用全局搜索, 正则表达式这种会丢失上下文信息的方式.

一般的编译过程如下图所示

![](https://blog.riskivy.com/wp-content/uploads/2020/05/def77df2aed03a84041a8955cb555a2a.png)

环境准备
----

首先这里需要搭建某知名 OA 的环境, 这里以`某知名OA 8`为例, 可以去百度下载`Ecology8.100.0531`

默认配置安装完成就 OK 了

### 遍历某知名 OA 的 JSP 文件路径

先使用 Python 获取到某知名 OA 文件夹中的 JSP 文件路径, 这里可以自己过滤一下

```
#python 遍历文件夹
import os
def get\_files(path=r"D:\\WEAVER\\ecology\\"):
    g = os.walk(path)
    result = \[\]
    for path, d, file\_list in g:
        for filename in file\_list:
            full\_path = os.path.join(path, filename)
            result.append(\[full\_path, filename\])
    return result


```

然后通过`burp intruder`的方式遍历某知名 OA 的 JSP 在前台的可访问性, 这里使用 Python 访问也行

获取到如下列表

```
Request Payload    Status Error  Timeout    Length Comment
7373   workflow/request/WorkflowViewRequestDetailBodyAction.jsp   200    false  false  73584  
7319   workflow/request/WorkflowManageRequestBody.jsp 200    false  false  71216  
7359   workflow/request/WorkflowSignInput.jsp 200    false  false  69746  
6445   web/workflow/request/WorkflowAddRequestBody.jsp    200    false  false  69080  
7372   workflow/request/WorkflowViewRequestDetailBody.jsp 200    false  false  66718  
7297   workflow/request/WorkflowAddRequestBodyDataCenter.jsp  200    false  false  64160  
7322   workflow/request/WorkflowManageRequestBodyDataCenter.jsp   200    false  false  64098  
7301   workflow/request/WorkflowAddRequestFormBody.jsp    200    false  false  62012  
3499   hrm/report/resource/HrmConstRpDataDefine.jsp   200    false  false  61648  
6923   workflow/request/BillBudgetExpenseDetail.jsp   200    false  false  61272  
7295   workflow/request/WorkflowAddRequestBody.jsp    200    false  false  60130  
7370   workflow/request/WorkflowViewRequestBody.jsp   200    false  false  59860    
.....
2368    formmode/import/ProcessOperation.jsp    200 false   false   218 
7378    workflow/request/WorkflowViewSign.jsp   0   false   false   5   
6419    web/WebBBSDsp.jsp   0   false   false   0   
6421    web/WebDsp.jsp  0   false   false   0   
6422    web/WebJournalDsp.jsp   0   false   false   0   
6426    web/WebListDspSecond.jsp    0   false   false   0   


```

![](https://blog.riskivy.com/wp-content/uploads/2020/05/a61568ab99d868e38e4ab247c6f42250.png)

### 获取 Resin 生成的 Servlet.java

获取到 JSP 文件的访问权限列表的同时, 某知名 OA 的目录`D:\WEAVER\ecology\WEB-INF\work\_jsp`中也生成了对应的 JSP Servlet

![](https://blog.riskivy.com/wp-content/uploads/2020/05/96c07c92efb11cf493de36b1430a594e.png)

然后把`_jsp`目录复制出来, 某知名 OA 的准备过程就到这里结束了

参考`Cobra`的`PHP Parser`
----------------------

### Cobra 源码理解

[cobra/parser.py](https://github.com/WhaleShark-Team/cobra/blob/master/cobra/parser.py)

```
\# -\*- coding: utf-8 -\*-

"""
    parser
    ~~~~~~

    Implements Code Parser

    :author:    BlBana <635373043@qq.com>
    :homepage:  https://github.com/WhaleShark-Team/cobra
    :license:   MIT, see LICENSE for more details.
    :copyright: Copyright (c) 2018 Feei. All rights reserved
"""
from phply.phplex import lexer  # 词法分析
from phply.phpparse import make\_parser  # 语法分析
from phply import phpast as php
from .log import logger

with\_line = True
scan\_results = \[\]  # 结果存放列表初始化
repairs = \[\]  # 用于存放修复函数


def export(items):
    result = \[\]
    if items:
        for item in items:
            if hasattr(item, 'generic'):
                item = item.generic(with\_lineno=with\_line)
            result.append(item)
    return result


def export\_list(params, export\_params):
    """
    将params中嵌套的多个列表，导出为一个列表
    :param params:
    :param export\_params:
    :return:
    """
    for param in params:
        if isinstance(param, list):
            export\_params = export\_list(param, export\_params)

        else:
            export\_params.append(param)

    return export\_params


def get\_all\_params(nodes):  # 用来获取调用函数的参数列表，nodes为参数列表
    """
    获取函数结构的所有参数
    :param nodes:
    :return:
    """
    params = \[\]
    export\_params = \[\]  # 定义空列表，用来给export\_list中使用
    for node in nodes:
        if isinstance(node.node, php.FunctionCall):  # 函数参数来自另一个函数的返回值
            params = get\_all\_params(node.node.params)

        else:
            if isinstance(node.node, php.Variable):
                params.append(node.node.name)

            if isinstance(node.node, php.BinaryOp):
                params = get\_binaryop\_params(node.node)
                params = export\_list(params, export\_params)

            if isinstance(node.node, php.ArrayOffset):
                param = get\_node\_name(node.node.node)
                params.append(param)

            if isinstance(node.node, php.Cast):
                param = get\_cast\_params(node.node.expr)
                params.append(param)

            if isinstance(node.node, php.Silence):
                param = get\_silence\_params(node.node)
                params.append(param)

    return params


def get\_silence\_params(node):
    """
    用来提取Silence类型中的参数
    :param node:
    :return:
    """
    param = \[\]
    if isinstance(node.expr, php.Variable):
        param = get\_node\_name(node.expr)

    if isinstance(node.expr, php.FunctionCall):
        param.append(node.expr)

    if isinstance(node.expr, php.Eval):
        param.append(node.expr)

    if isinstance(node.expr, php.Assignment):
        param.append(node.expr)

    return param


def get\_cast\_params(node):
    """
    用来提取Cast类型中的参数
    :param node:
    :return:
    """
    param = \[\]
    if isinstance(node, php.Silence):
        param = get\_node\_name(node.expr)

    return param


def get\_binaryop\_params(node):  # 当为BinaryOp类型时，分别对left和right进行处理，取出需要的变量
    """
    用来提取Binaryop中的参数
    :param node:
    :return:
    """
    logger.debug('\[AST\] Binaryop --> {node}'.format(node=node))
    params = \[\]
    buffer\_ = \[\]

    if isinstance(node.left, php.Variable) or isinstance(node.right, php.Variable):  # left, right都为变量直接取值
        if isinstance(node.left, php.Variable):
            params.append(node.left.name)

        if isinstance(node.right, php.Variable):
            params.append(node.right.name)

    if not isinstance(node.right, php.Variable) or not isinstance(node.left, php.Variable):  # right不为变量时
        params\_right = get\_binaryop\_deep\_params(node.right, params)
        params\_left = get\_binaryop\_deep\_params(node.left, params)

        params = params\_left + params\_right

    params = export\_list(params, buffer\_)
    return params


def get\_binaryop\_deep\_params(node, params):  # 取出right，left不为变量时，对象结构中的变量
    """
    取出深层的变量名
    :param node: node为上一步中的node.left或者node.right节点
    :param params:
    :return:
    """
    if isinstance(node, php.ArrayOffset):  # node为数组，取出数组变量名
        param = get\_node\_name(node.node)
        params.append(param)

    if isinstance(node, php.BinaryOp):  # node为BinaryOp，递归取出其中变量
        param = get\_binaryop\_params(node)
        params.append(param)

    if isinstance(node, php.FunctionCall):  # node为FunctionCall，递归取出其中变量名
        params = get\_all\_params(node.params)

    return params


def get\_expr\_name(node):  # expr为'expr'中的值
    """
    获取赋值表达式的表达式部分中的参数名-->返回用来进行回溯
    :param node:
    :return:
    """
    param\_lineno = 0
    is\_re = False
    if isinstance(node, php.ArrayOffset):  # 当赋值表达式为数组
        param\_expr = get\_node\_name(node.node)  # 返回数组名
        param\_lineno = node.node.lineno

    elif isinstance(node, php.Variable):  # 当赋值表达式为变量
        param\_expr = node.name  # 返回变量名
        param\_lineno = node.lineno

    elif isinstance(node, php.FunctionCall):  # 当赋值表达式为函数
        param\_expr = get\_all\_params(node.params)  # 返回函数参数列表
        param\_lineno = node.lineno
        is\_re = is\_repair(node.name)  # 调用了函数，判断调用的函数是否为修复函数

    elif isinstance(node, php.BinaryOp):  # 当赋值表达式为BinaryOp
        param\_expr = get\_binaryop\_params(node)
        param\_lineno = node.lineno

    else:
        param\_expr = node

    return param\_expr, param\_lineno, is\_re


def get\_node\_name(node):  # node为'node'中的元组
    """
    获取Variable类型节点的name
    :param node:
    :return:
    """
    if isinstance(node, php.Variable):
        return node.name  # 返回此节点中的变量名


def is\_repair(expr):
    """
    判断赋值表达式是否出现过滤函数，如果已经过滤，停止污点回溯，判定漏洞已修复
    :param expr: 赋值表达式
    :return:
    """
    is\_re = False  # 是否修复，默认值是未修复
    for repair in repairs:
        if expr == repair:
            is\_re = True
            return is\_re
    return is\_re


def is\_sink\_function(param\_expr, function\_params):
    """
    判断自定义函数的入参-->判断此函数是否是危险函数
    :param param\_expr:
    :param function\_params:
    :return:
    """
    is\_co = -1
    cp = None
    if function\_params is not None:
        for function\_param in function\_params:
            if param\_expr == function\_param:
                is\_co = 2
                cp = function\_param
                logger.debug('\[AST\] is\_sink\_function --> {function\_param}'.format(function\_param=cp))
    return is\_co, cp


def is\_controllable(expr):  # 获取表达式中的变量，看是否在用户可控变量列表中
    """
    判断赋值表达式是否是用户可控的
    :param expr:
    :return:
    """
    controlled\_params = \[
        '$\_GET',
        '$\_POST',
        '$\_REQUEST',
        '$\_COOKIE',
        '$\_FILES',
        '$\_SERVER',
        '$HTTP\_POST\_FILES',
        '$HTTP\_COOKIE\_VARS',
        '$HTTP\_REQUEST\_VARS',
        '$HTTP\_POST\_VARS',
        '$HTTP\_RAW\_POST\_DATA',
        '$HTTP\_GET\_VARS'
    \]
    if expr in controlled\_params:
        logger.debug('\[AST\] is\_controllable --> {expr}'.format(expr=expr))
        return 1, expr
    return -1, None


def parameters\_back(param, nodes, function\_params=None):  # 用来得到回溯过程中的被赋值的变量是否与敏感函数变量相等,param是当前需要跟踪的污点
    """
    递归回溯敏感函数的赋值流程，param为跟踪的污点，当找到param来源时-->分析复制表达式-->获取新污点；否则递归下一个节点
    :param param:
    :param nodes:
    :param function\_params:
    :return:
    """
    expr\_lineno = 0  # source所在行号
    is\_co, cp = is\_controllable(param)

    if len(nodes) != 0 and is\_co == -1:
        node = nodes\[len(nodes) - 1\]

        if isinstance(node, php.Assignment):  # 回溯的过程中，对出现赋值情况的节点进行跟踪
            param\_node = get\_node\_name(node.node)  # param\_node为被赋值的变量
            param\_expr, expr\_lineno, is\_re = get\_expr\_name(node.expr)  # param\_expr为赋值表达式,param\_expr为变量或者列表

            if param == param\_node and is\_re is True:
                is\_co = 0
                cp = None
                return is\_co, cp, expr\_lineno

            if param == param\_node and not isinstance(param\_expr, list):  # 找到变量的来源，开始继续分析变量的赋值表达式是否可控
                is\_co, cp = is\_controllable(param\_expr)  # 开始判断变量是否可控

                if is\_co != 1:
                    is\_co, cp = is\_sink\_function(param\_expr, function\_params)

                param = param\_expr  # 每次找到一个污点的来源时，开始跟踪新污点，覆盖旧污点

            if param == param\_node and isinstance(param\_expr, list):
                for expr in param\_expr:
                    param = expr
                    is\_co, cp = is\_controllable(expr)

                    if is\_co == 1:
                        return is\_co, cp, expr\_lineno

                    \_is\_co, \_cp, expr\_lineno = parameters\_back(param, nodes\[:-1\], function\_params)

                    if \_is\_co != -1:  # 当参数可控时，值赋给is\_co 和 cp，有一个参数可控，则认定这个函数可能可控
                        is\_co = \_is\_co
                        cp = \_cp

        if is\_co == -1:  # 当is\_co为True时找到可控，停止递归
            is\_co, cp, expr\_lineno = parameters\_back(param, nodes\[:-1\], function\_params)  # 找到可控的输入时，停止递归

    elif len(nodes) == 0 and function\_params is not None:
        for function\_param in function\_params:
            if function\_param == param:
                is\_co = 2
                cp = function\_param

    return is\_co, cp, expr\_lineno


def get\_function\_params(nodes):
    """
    获取用户自定义函数的所有入参
    :param nodes: 自定义函数的参数部分
    :return: 以列表的形式返回所有的入参
    """
    params = \[\]
    for node in nodes:

        if isinstance(node, php.FormalParameter):
            params.append(node.name)

    return params


def anlysis\_function(node, back\_node, vul\_function, function\_params, vul\_lineno):
    """
    对用户自定义的函数进行分析-->获取函数入参-->入参用经过赋值流程，进入sink函数-->此自定义函数为危险函数
    :param node:
    :param back\_node:
    :param vul\_function:
    :param function\_params:
    :param vul\_lineno:
    :return:
    """
    global scan\_results
    try:
        if node.name == vul\_function and int(node.lineno) == int(vul\_lineno):  # 函数体中存在敏感函数，开始对敏感函数前的代码进行检测
            for param in node.params:
                if isinstance(param.node, php.Variable):
                    analysis\_variable\_node(param.node, back\_node, vul\_function, vul\_lineno, function\_params)

                if isinstance(param.node, php.FunctionCall):
                    analysis\_functioncall\_node(param.node, back\_node, vul\_function, vul\_lineno, function\_params)

                if isinstance(param.node, php.BinaryOp):
                    analysis\_binaryop\_node(param.node, back\_node, vul\_function, vul\_lineno, function\_params)

                if isinstance(param.node, php.ArrayOffset):
                    analysis\_arrayoffset\_node(param.node, vul\_function, vul\_lineno)

    except Exception as e:
        logger.debug(e)


# def analysis\_functioncall(node, back\_node, vul\_function, vul\_lineno):
#     """
#     调用FunctionCall-->判断调用Function是否敏感-->get params获取所有参数-->开始递归判断
#     :param node:
#     :param back\_node:
#     :param vul\_function:
#     :param vul\_lineno
#     :return:
#     """
#     global scan\_results
#     try:
#         if node.name == vul\_function and int(node.lineno) == int(vul\_lineno):  # 定位到敏感函数
#             for param in node.params:
#                 if isinstance(param.node, php.Variable):
#                     analysis\_variable\_node(param.node, back\_node, vul\_function, vul\_lineno)
#
#                 if isinstance(param.node, php.FunctionCall):
#                     analysis\_functioncall\_node(param.node, back\_node, vul\_function, vul\_lineno)
#
#                 if isinstance(param.node, php.BinaryOp):
#                     analysis\_binaryop\_node(param.node, back\_node, vul\_function, vul\_lineno)
#
#                 if isinstance(param.node, php.ArrayOffset):
#                     analysis\_arrayoffset\_node(param.node, vul\_function, vul\_lineno)
#
#     except Exception as e:
#         logger.debug(e)


def analysis\_binaryop\_node(node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    """
    处理BinaryOp类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
    :param node:
    :param back\_node:
    :param vul\_function:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    logger.debug('\[AST\] vul\_function:{v}'.format(v=vul\_function))
    params = get\_binaryop\_params(node)
    params = export\_list(params, export\_params=\[\])

    for param in params:
        is\_co, cp, expr\_lineno = parameters\_back(param, back\_node, function\_params)
        set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)


def analysis\_arrayoffset\_node(node, vul\_function, vul\_lineno):
    """
    处理ArrayOffset类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
    :param node:
    :param vul\_function:
    :param vul\_lineno:
    :return:
    """
    logger.debug('\[AST\] vul\_function:{v}'.format(v=vul\_function))
    param = get\_node\_name(node.node)
    expr\_lineno = node.lineno
    is\_co, cp = is\_controllable(param)

    set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)


def analysis\_functioncall\_node(node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    """
    处理FunctionCall类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
    :param node:
    :param back\_node:
    :param vul\_function:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    logger.debug('\[AST\] vul\_function:{v}'.format(v=vul\_function))
    params = get\_all\_params(node.params)
    for param in params:
        is\_co, cp, expr\_lineno = parameters\_back(param, back\_node, function\_params)
        set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)


def analysis\_variable\_node(node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    """
    处理Variable类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
    :param node:
    :param back\_node:
    :param vul\_function:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    logger.debug('\[AST\] vul\_function:{v}'.format(v=vul\_function))
    params = get\_node\_name(node)
    is\_co, cp, expr\_lineno = parameters\_back(params, back\_node, function\_params)
    set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, params, vul\_lineno)


def analysis\_if\_else(node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    nodes = \[\]
    if isinstance(node.node, php.Block):  # if语句中的sink点以及变量
        analysis(node.node.nodes, vul\_function, back\_node, vul\_lineno, function\_params)

    if node.else\_ is not None:  # else语句中的sink点以及变量
        if isinstance(node.else\_.node, php.Block):
            analysis(node.else\_.node.nodes, vul\_function, back\_node, vul\_lineno, function\_params)

    if len(node.elseifs) != 0:  # elseif语句中的sink点以及变量
        for i\_node in node.elseifs:
            if i\_node.node is not None:
                if isinstance(i\_node.node, php.Block):
                    analysis(i\_node.node.nodes, vul\_function, back\_node, vul\_lineno, function\_params)

                else:
                    nodes.append(i\_node.node)
                    analysis(nodes, vul\_function, back\_node, vul\_lineno, function\_params)


def analysis\_echo\_print(node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    """
    处理echo/print类型节点-->判断节点类型-->不同If分支回溯判断参数是否可控-->输出结果
    :param node:
    :param back\_node:
    :param vul\_function:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    global scan\_results

    if int(vul\_lineno) == int(node.lineno):
        if isinstance(node, php.Print):
            if isinstance(node.node, php.FunctionCall):
                analysis\_functioncall\_node(node.node, back\_node, vul\_function, vul\_lineno, function\_params)

            if isinstance(node.node, php.Variable) and vul\_function == 'print':  # 直接输出变量信息
                analysis\_variable\_node(node.node, back\_node, vul\_function, vul\_lineno, function\_params)

            if isinstance(node.node, php.BinaryOp) and vul\_function == 'print':
                analysis\_binaryop\_node(node.node, back\_node, vul\_function, vul\_lineno, function\_params)

            if isinstance(node.node, php.ArrayOffset) and vul\_function == 'print':
                analysis\_arrayoffset\_node(node.node, vul\_function, vul\_lineno)

        elif isinstance(node, php.Echo):
            for k\_node in node.nodes:
                if isinstance(k\_node, php.FunctionCall):  # 判断节点中是否有函数调用节点
                    analysis\_functioncall\_node(k\_node, back\_node, vul\_function, vul\_lineno, function\_params)  # 将含有函数调用的节点进行分析

                if isinstance(k\_node, php.Variable) and vul\_function == 'echo':
                    analysis\_variable\_node(k\_node, back\_node, vul\_function, vul\_lineno), function\_params

                if isinstance(k\_node, php.BinaryOp) and vul\_function == 'echo':
                    analysis\_binaryop\_node(k\_node, back\_node, vul\_function, vul\_lineno, function\_params)

                if isinstance(k\_node, php.ArrayOffset) and vul\_function == 'echo':
                    analysis\_arrayoffset\_node(k\_node, vul\_function, vul\_lineno)


def analysis\_eval(node, vul\_function, back\_node, vul\_lineno, function\_params=None):
    """
    处理eval类型节点-->判断节点类型-->不同If分支回溯判断参数是否可控-->输出结果
    :param node:
    :param vul\_function:
    :param back\_node:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    global scan\_results

    if vul\_function == 'eval' and int(node.lineno) == int(vul\_lineno):
        if isinstance(node.expr, php.Variable):
            analysis\_variable\_node(node.expr, back\_node, vul\_function, vul\_lineno, function\_params)

        if isinstance(node.expr, php.FunctionCall):
            analysis\_functioncall\_node(node.expr, back\_node, vul\_function, vul\_lineno, function\_params)

        if isinstance(node.expr, php.BinaryOp):
            analysis\_binaryop\_node(node.expr, back\_node, vul\_function, vul\_lineno, function\_params)

        if isinstance(node.expr, php.ArrayOffset):
            analysis\_arrayoffset\_node(node.expr, vul\_function, vul\_lineno)


def analysis\_file\_inclusion(node, vul\_function, back\_node, vul\_lineno, function\_params=None):
    """
    处理include/require类型节点-->判断节点类型-->不同If分支回溯判断参数是否可控-->输出结果
    :param node:
    :param vul\_function:
    :param back\_node:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    global scan\_results
    include\_fs = \['include', 'include\_once', 'require', 'require\_once'\]

    if vul\_function in include\_fs and int(node.lineno) == int(vul\_lineno):
        logger.debug('\[AST-INCLUDE\] {l}-->{r}'.format(l=vul\_function, r=vul\_lineno))

        if isinstance(node.expr, php.Variable):
            analysis\_variable\_node(node.expr, back\_node, vul\_function, vul\_lineno, function\_params)

        if isinstance(node.expr, php.FunctionCall):
            analysis\_functioncall\_node(node.expr, back\_node, vul\_function, vul\_lineno, function\_params)

        if isinstance(node.expr, php.BinaryOp):
            analysis\_binaryop\_node(node.expr, back\_node, vul\_function, vul\_lineno, function\_params)

        if isinstance(node.expr, php.ArrayOffset):
            analysis\_arrayoffset\_node(node.expr, vul\_function, vul\_lineno)


def set\_scan\_results(is\_co, cp, expr\_lineno, sink, param, vul\_lineno):
    """
    获取结果信息-->输出结果
    :param is\_co:
    :param cp:
    :param expr\_lineno:
    :param sink:
    :param param:
    :param vul\_lineno:
    :return:
    """
    results = \[\]
    global scan\_results

    result = {
        'code': is\_co,
        'source': cp,
        'source\_lineno': expr\_lineno,
        'sink': sink,
        'sink\_param:': param,
        'sink\_lineno': vul\_lineno
    }
    if result\['code'\] != -1:  # 查出来漏洞结果添加到结果信息中
        results.append(result)
        scan\_results += results


def analysis(nodes, vul\_function, back\_node, vul\_lineo, function\_params=None):
    """
    调用FunctionCall-->analysis\_functioncall分析调用函数是否敏感
    :param nodes: 所有节点
    :param vul\_function: 要判断的敏感函数名
    :param back\_node: 各种语法结构里面的语句
    :param vul\_lineo: 漏洞函数所在行号
    :param function\_params: 自定义函数的所有参数列表
    :return:
    """
    buffer\_ = \[\]
    for node in nodes:
        if isinstance(node, php.FunctionCall):  # 函数直接调用，不进行赋值
            anlysis\_function(node, back\_node, vul\_function, function\_params, vul\_lineo)

        elif isinstance(node, php.Assignment):  # 函数调用在赋值表达式中
            if isinstance(node.expr, php.FunctionCall):
                anlysis\_function(node.expr, back\_node, vul\_function, function\_params, vul\_lineo)

            if isinstance(node.expr, php.Eval):
                analysis\_eval(node.expr, vul\_function, back\_node, vul\_lineo, function\_params)

            if isinstance(node.expr, php.Silence):
                buffer\_.append(node.expr)
                analysis(buffer\_, vul\_function, back\_node, vul\_lineo, function\_params)

        elif isinstance(node, php.Print) or isinstance(node, php.Echo):
            analysis\_echo\_print(node, back\_node, vul\_function, vul\_lineo, function\_params)

        elif isinstance(node, php.Silence):
            nodes = get\_silence\_params(node)
            analysis(nodes, vul\_function, back\_node, vul\_lineo)

        elif isinstance(node, php.Eval):
            analysis\_eval(node, vul\_function, back\_node, vul\_lineo, function\_params)

        elif isinstance(node, php.Include) or isinstance(node, php.Require):
            analysis\_file\_inclusion(node, vul\_function, back\_node, vul\_lineo, function\_params)

        elif isinstance(node, php.If):  # 函数调用在if-else语句中时
            analysis\_if\_else(node, back\_node, vul\_function, vul\_lineo, function\_params)

        elif isinstance(node, php.While) or isinstance(node, php.For):  # 函数调用在循环中
            if isinstance(node.node, php.Block):
                analysis(node.node.nodes, vul\_function, back\_node, vul\_lineo, function\_params)

        elif isinstance(node, php.Function) or isinstance(node, php.Method):
            function\_body = \[\]
            function\_params = get\_function\_params(node.params)
            analysis(node.nodes, vul\_function, function\_body, vul\_lineo, function\_params=function\_params)

        elif isinstance(node, php.Class):
            analysis(node.nodes, vul\_function, back\_node, vul\_lineo, function\_params)

        back\_node.append(node)


def scan\_parser(code\_content, sensitive\_func, vul\_lineno, repair):
    """
    开始检测函数
    :param code\_content: 要检测的文件内容
    :param sensitive\_func: 要检测的敏感函数,传入的为函数列表
    :param vul\_lineno: 漏洞函数所在行号
    :param repair: 对应漏洞的修复函数列表
    :return:
    """
    try:
        global repairs
        global scan\_results
        repairs = repair
        scan\_results = \[\]
        parser = make\_parser()
        all\_nodes = parser.parse(code\_content, debug=False, lexer=lexer.clone(), tracking=with\_line)
        for func in sensitive\_func:  # 循环判断代码中是否存在敏感函数，若存在，递归判断参数是否可控;对文件内容循环判断多次
            back\_node = \[\]
            analysis(all\_nodes, func, back\_node, int(vul\_lineno), function\_params=None)
    except SyntaxError as e:
        logger.warning('\[AST\] \[ERROR\]:{e}'.format(e=e))

    return scan\_results


```

### 数据流分析基础知识

使用数据流分析进行漏洞挖掘一般知道 4 个关键词就可以了

*   `sink`: 污点函数, 敏感函数, 比如
    *   `PHP`: `mysqli_query`, `system`, `shell_exec`, `unserialize`
    *   `Java` : `executeSql`, `GroovyShell.evaluate()`, `Runtime.getRuntime().exec()`,`unserialize`
*   `source`: 输入来源, 通常为用户可控的来源, 比如
    *   `PHP`: `$_GET`, `$_POST`, `$_REQUEST`, `$_COOKIE`, `$_FILES`, `$_SERVER`, `$HTTP_POST_FILES`, `$HTTP_COOKIE_VARS`, `$HTTP_REQUEST_VARS`, `$HTTP_POST_VARS`, `$HTTP_RAW_POST_DATA`, `$HTTP_GET_VARS`
    *   `JAVA`: `request.getParameter`, `request.getparametermap`
*   `repair`/`sanitizer`: 修复函数 / 清理函数, 通常为恶意输入过滤, hash 或者强制类型转换, 比如
    *   `PHP`: `md5`, `addslashes`, `mysqli_real_escape_string`, `mysql_escape_string`
    *   `Java` : `Integer.parseInt`, Java 中更多是开发者自己实现的函数, 例如某知名 OA 中的 `null2int`, `getIntValue`
*   `DataFlow`: 数据流, 变量在代码中的传递路径, 是`Static Analysis`中很重要的知识点, 这里先不考虑`ControlFlow`

了解了以上知识点, 结合`Cobra`的`PHP Parser`, 总结一下大概逻辑

1.  定义`sink`, `source`, `repair`
    1.  一组敏感函数`sensitive_func`, 例如`mysqli_query`
    2.  一组修复函数 `repair`, 例如`mysqli_real_escape_string`
    3.  一组预置的可控输入`source`, 例如`_GET`
2.  查找`mysqli_query`所在代码文件`vul_file`的行数`sink_lineno`
3.  `Cobra` 的逻辑是自上而下遍历 PHP 文件, 直到匹配`vul_file`的`sink_lineno`, 递归寻找变量传递过程, 是否能传达到可控输入`source`(这里的`source`也可以是函数定义的形参, 这样可以发现漏洞函数, 作为二次 sink 进行新的漏洞发掘)
    
4.  若果传递过程中没有经过修复函数`repair`的处理, 即可认为这里存在漏洞
    

实现 Java 的 AST 处理器
-----------------

其实大部分语言到了 AST 层面, 结构都差不多, 到了 IR 阶段 (`Intermediate Representation`) 就基本没有区别了

(很多代码审计软件都会先把源文件转换成 IR 再进行处理, 用 AST 其实一样处理, 只是 IR 更加通用, 常见的 IR 有三地址码形式)

所以从 PHP 的处理器到 Java 的处理器的基本功能是差不多实现的.

这里我们只需要把 Java 代码转换成 AST 的形式就足够满足需求了

Java AST 解析器选择 Python 的`javalang`库

安装方法: `pip install javalang`

这边我之前整理`phply`和`javalang`结构对照的表格, 可能有所疏漏, 但是基本覆盖了常用的一些对象

<table><thead><tr><th>phply</th><th>javalang</th><th>解释</th><th>可迭代 / 参数</th><th>类型递归</th></tr></thead><tbody><tr><td>php.Variable</td><td>MemberReference</td><td>变量引用 member</td><td></td><td></td></tr><tr><td>php.FunctionCall</td><td>MethodInvocation</td><td>函数直接调用 member arguments</td><td>arguments</td><td></td></tr><tr><td>php.BinaryOp</td><td>BinaryOperation</td><td>二元操作 operandl operandr operator</td><td></td><td>operandl operandr</td></tr><tr><td></td><td>ArrayInitializer</td><td>数组初始化</td><td></td><td></td></tr><tr><td>php.ArrayOffset</td><td>ArraySelector</td><td>数组赋值操作 / 不需要</td><td>children</td><td></td></tr><tr><td>php.Block</td><td>BlockStatement</td><td>一些局部语句块,{} statements</td><td>statements</td><td></td></tr><tr><td>php.Print</td><td></td><td>Java 中应当没有, 应该是函数调用 sout</td><td></td><td></td></tr><tr><td>php.Assignment</td><td>Assignment</td><td>赋值语句</td><td></td><td>expressionl</td></tr><tr><td>php.Eval</td><td></td><td>这个 java 里没有, 有就是 beanshell/jshell</td><td></td><td></td></tr><tr><td>php.Silence</td><td></td><td>准备执行函数调用而不显示错误消息 https://www.php.net/manual/en/internals2.opcodes.begin-silence.php</td><td></td><td></td></tr><tr><td>php.Echo</td><td></td><td>Java 中应当没有, 应该是函数调用 sout</td><td></td><td></td></tr><tr><td>php.Include</td><td></td><td>import 暂不考虑</td><td></td><td></td></tr><tr><td>php.Require</td><td></td><td>import 暂不考虑</td><td></td><td></td></tr><tr><td>php.While</td><td>WhileStatement</td><td>body.statements condition</td><td>body.statements</td><td></td></tr><tr><td>php.For</td><td>ForStatement</td><td></td><td></td><td><blockstatement>body</blockstatement></td></tr><tr><td>php.Function</td><td>MethodDeclaration</td><td>phply: 函数名称 java 没有</td><td>body</td><td></td></tr><tr><td>php.Method</td><td>MethodDeclaration</td><td>phply: 类名称与函数名称 java 类方法</td><td>body</td><td></td></tr><tr><td>php.Class</td><td>ClassDeclaration</td><td>类定义</td><td>body</td><td></td></tr><tr><td>php.Cast</td><td>Cast</td><td>强制类型转换 $foo = (int) $bar;</td><td></td><td></td></tr><tr><td>php.If</td><td>IfStatement</td><td>then_statement else_statement</td><td></td><td>then_statement else_statement</td></tr><tr><td></td><td>DoStatement</td><td>do{}While 结构, 基本等同 While 处理</td><td>body.statements</td><td></td></tr><tr><td></td><td>Statement</td><td></td><td></td><td>expression</td></tr><tr><td></td><td>CompilationUnit</td><td>整个树</td><td>children[-1]</td><td></td></tr><tr><td></td><td>StatementExpression</td><td>是直接赋值给变量 (没变量类型声明开头) (代指一行?</td><td></td><td>expression</td></tr><tr><td></td><td>LocalVariableDeclaration</td><td>声明变量且初始化</td><td>declarators</td><td>declarators[0].initializer</td></tr><tr><td></td><td>This</td><td>代指当前类 / 类变量也是 This 的实例</td><td></td><td></td></tr><tr><td></td><td>SwitchStatement</td><td></td><td>cases:[SwitchStatementCase]</td><td></td></tr><tr><td></td><td>SwitchStatementCase</td><td></td><td>statements</td><td></td></tr><tr><td>php.Block</td><td>BlockStatement</td><td></td><td>statements</td><td></td></tr></tbody></table>

### `scan_parser`配置`sink`, `repair`启动扫描

```
def scan\_parser(self, code\_content, sensitive\_func, vul\_lineno, repair):
    """
    先从 sensitive\_func 中提取敏感函数 func 循环查询AST
    ->进入analysis中查询 vul\_lineno 所在行的敏感函数调用
    :param code\_content: 要检测的文件内容
    :param sensitive\_func: 要检测的敏感函数,传入的为函数列表
    :param vul\_lineno: 漏洞函数所在行号
    :param repair: 对应漏洞的修复函数列表
    :return:
    """
    try:
        # global repairs
        # global scan\_results
        self.repairs = repair
        self.scan\_results = \[\]
        tree = javalang.parse.parse(code\_content)
        all\_nodes = tree.children\[-1\]
        for func in sensitive\_func:  # 循环判断代码中是否存在敏感函数，若存在，递归判断参数是否可控;对文件内容循环判断多次
            back\_node = \[\]
            self.analysis(all\_nodes, func, back\_node, int(vul\_lineno), function\_params=None)
    except SyntaxError as e:
        print('\[AST\] \[ERROR\]:{e}'.format(e=e))

    return self.scan\_results


```

### `analysis`分析器主函数

```
def analysis(self, nodes, vul\_function, back\_node, vul\_lineo, function\_params=None):
    """
    总体的思路是遍历所有节点且放入back\_nodes中
    -> 查找所有的 MethodInvocation 直到找到匹配 vul\_lineo 的那一个
    -> 然后在函数调用中查找出来涉及的变量
    ( anlysis\_function 就是进入函数体进行敏感函数查找而已,可以优化 )
    ( analysis\_functioncall\_node 就是取出敏感函数的参数(变量)进行 parameters\_back )

    :param nodes: 所有节点
    :param vul\_function: 要判断的敏感函数名
    :param back\_node: 各种语法结构里面的语句
    :param vul\_lineo: 漏洞函数所在行号
    :param function\_params: 自定义函数的所有参数列表
    :return:
    """
    buffer\_ = \[\]
    for node in nodes:
        if isinstance(node, MethodInvocation):
            # 从原文的意思看,这里是检测到函数调用,去找这个方法的MethodDeceleration,如果这个函数里面有敏感操作,就爆有问题
            self.anlysis\_function(node, back\_node, vul\_function, function\_params, vul\_lineo)

        elif isinstance(node, StatementExpression):
            if isinstance(node.expression, MethodInvocation):
                self.anlysis\_function(node.expression, back\_node, vul\_function, function\_params, vul\_lineo)

            elif isinstance(node.expression, Assignment):
                if isinstance(node.expression.value, MethodInvocation):
                    self.anlysis\_function(node.expression.value, back\_node, vul\_function, function\_params,
                                          vul\_lineo)
        # todo 这里还有 binop 的操作
        elif isinstance(node, LocalVariableDeclaration):
            for declarator in node.declarators:
                if isinstance(declarator.initializer, MethodInvocation):
                    self.anlysis\_function(declarator.initializer, back\_node, vul\_function, function\_params,
                                          vul\_lineo)


        elif isinstance(node, IfStatement):  # 函数调用在if-else语句中时
            self.analysis\_if\_else(node, vul\_function, back\_node, vul\_lineo, function\_params)

        elif isinstance(node, TryStatement):  # 函数调用在try-catch-finally语句中时
            # print(back\_node)
            self.analysis(node.block, vul\_function, back\_node, vul\_lineo, function\_params)
            # analysis(node.catches, back\_node, vul\_function, vul\_lineo, function\_params)
            # analysis(node.finally\_block, back\_node, vul\_function, vul\_lineo, function\_params)

        elif isinstance(node, WhileStatement):
            self.analysis(node.body.statements, vul\_function, back\_node, vul\_lineo, function\_params)


        elif isinstance(node, ForStatement):
            if isinstance(node.body, BlockStatement):
                self.analysis(node.body, vul\_function, back\_node, vul\_lineo, function\_params)


        elif isinstance(node, MethodDeclaration):
            function\_body = \[node\]
            function\_params = self.get\_function\_params(node.parameters)
            self.analysis(node.body, vul\_function, function\_body, vul\_lineo, function\_params=function\_params)


        elif isinstance(node, ClassDeclaration):
            self.analysis(node.body, vul\_function, back\_node, vul\_lineo, function\_params)
        # if back\_node == "executeSql":
        #     print(back\_node)
        back\_node.append(node)


```

### `anlysis_function`分析函数调用

```
def anlysis\_function(self, node, back\_node, vul\_function, function\_params, vul\_lineno):
    """
    对用户自定义的函数进行分析-->获取函数入参-->入参用经过赋值流程，进入sink函数-->此自定义函数为危险函数
    最终目的是分析函数调用
    :param node: 传入一个 MethodDeclaration 类型节点
    :param back\_node: 传入 back\_nodes
    :param vul\_function: 存在漏洞的函数名
    :param function\_params: 函数的形参(从 MethodDeceleration 节点进来的话)
    :param vul\_lineno:
    :return:
    """
    global scan\_results
    # try:
    if node.member == vul\_function and int(node.position.line) == int(vul\_lineno):  # 函数体中存在敏感函数，开始对敏感函数前的代码进行检测
        for param in node.arguments:
            if isinstance(param, MemberReference):
                self.analysis\_variable\_node(param, back\_node, vul\_function, vul\_lineno, function\_params)

            elif isinstance(param, MethodInvocation):
                self.analysis\_functioncall\_node(param, back\_node, vul\_function, vul\_lineno, function\_params)

            elif isinstance(param, BinaryOperation):
                self.analysis\_binaryop\_node(param, back\_node, vul\_function, vul\_lineno, function\_params)

    # except Exception as e:
    #     print(e)


```

### `analysis_variable_node`分析变量节点

```
def analysis\_variable\_node(self, node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    """
    处理Variable类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
    这里直接将最后一步回溯到的变量写入全局结果表中,并不包含路径
    :param node:
    :param back\_node:
    :param vul\_function:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    # print('\[AST\] vul\_function:{v}'.format(v=vul\_function))
    param = self.get\_node\_name(node)
    is\_co, cp, expr\_lineno = self.parameters\_back(param, back\_node, function\_params)
    self.set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)


```

### `get_expr_name`获取赋值表达式中的参数名

```
def get\_expr\_name(self, node):  # expr为'expr'中的值
    """
    获取赋值表达式的表达式部分中的参数名(变量名)-->返回用来进行回溯
    :param node: 输入一个节点(要求是一个表达式的右值), 检测表达式包含的所有变量
    :return param\_expr: 返回表达式中涉及的所有变量的列表 \[\]
    :return param\_lineno: 返回当前表达式所在行 int
    :return is\_re: 返回是否已经修复  boolean
    """
    # todo 这里有个坑. javalang有position缺失的情况.可能会发生变量回溯丢失
    param\_lineno = 0
    is\_re = False
    param\_expr = None


    if isinstance(node, MemberReference):  # 当赋值表达式为变量
        param\_expr = node.member  # 返回变量名
        param\_lineno = node.position.line

    elif isinstance(node, MethodInvocation):  # 当赋值表达式为函数
        param\_expr = self.get\_all\_params(node.arguments)  # 返回函数参数列表
        param\_lineno = node.position.line
        # function\_name = node.qualifier + "." + node.member
        is\_re = False
        # 调用了函数，判断调用的函数是否为修复函数
        for func in self.get\_all\_funcs(node):
            if self.is\_repair(func):
                is\_re = True
                break

    elif isinstance(node, BinaryOperation):  # 当赋值表达式为BinaryOp
        param\_expr = self.get\_binaryop\_params(node)
        # todo 需要修复javalang的 position 丢失的问题 这里先硬编码一下
        # param\_lineno = node.position.line
        param\_lineno = 7

    elif isinstance(node, Assignment):  # 当赋值表达式为Assignment
        param\_expr, param\_lineno, is\_re = self.get\_expr\_name(node.value)
        # param\_lineno = node.position.line

    elif isinstance(node, This):  # 当赋值表达式为 This
        for selector in node.selectors:
            param\_expr, param\_lineno, is\_re = self.get\_expr\_name(selector)
            if is\_re:
                return param\_expr, param\_lineno, is\_re
    else:
        param\_expr = node
        # print(param\_expr)
    # print(param\_expr)
    return param\_expr, param\_lineno, is\_re


```

### `get_node_name`获取变量节点的变量名

```
def get\_node\_name(self, node):  # node为'node'中的元组
    """
    获取MemberReference类型节点的name
    :param node: 一般是MemberReference,字面量啥的不需要跟踪
    :return: MemberReference.member
    """
    if isinstance(node, MemberReference):
        return node.member  # 返回此节点中的变量名
    elif isinstance(node, VariableDeclarator):
        return node.name  # 返回此节点中的变量名


```

### `parameters_back`实现变量回溯

```
    def parameters\_back(self, param, nodes, function\_params=None, node\_lineno=-1):  # 用来得到回溯过程中的被赋值的变量是否与敏感函数变量相等,param是当前需要跟踪的污点
        """
        递归回溯敏感函数的赋值流程，param为跟踪的污点，当找到param来源时-->分析复制表达式-->获取新污点；否则递归下一个节点
        :param param: 输入一个变量名
        :param nodes: nodes 也就是之前访问的back\_nodes,里面基本都是LocalVariableDeclaration/StatementExpression/IFxxx
        :param function\_params: 递归过程中保持函数的形参,如果变量是从形参获得也认为可控
        :return is\_co, cp, expr\_lineno: 可控返回1 , 可控的变量名, 变量所在行
        """
        # node\_lineno = -1
        # print(node\_lineno)
        if len(nodes) > 0 and node\_lineno == -1:
            node\_lineno = nodes\[0\].position.line  # source所在行号
        expr\_lineno = 0
        is\_re = False
        is\_co, cp = self.is\_controllable(param)

        if len(nodes) != 0 and is\_co == -1:
            node = nodes\[len(nodes) - 1\]
            # if isinstance(node, LocalVariableDeclaration):
            tnodes = \[\]
            if isinstance(node, LocalVariableDeclaration):  # 回溯的过程中，对出现赋值情况的节点进行跟踪
                if isinstance(node, LocalVariableDeclaration):
                    tnodes = \[\[declarator, declarator.initializer\] for declarator in node.declarators\]
            elif isinstance(node, StatementExpression):
                if isinstance(node.expression, Assignment):
                    tnodes = \[\[node.expression.expressionl, node.expression.value\]\]

            for left\_var, right\_var in tnodes:
                param\_node = self.get\_node\_name(left\_var)
                # param\_expr为赋值表达式,param\_expr为变量或者列表
                param\_expr, expr\_lineno, is\_re = self.get\_expr\_name(right\_var)

                if param == param\_node and is\_re is False and isinstance(right\_var, MethodInvocation):
                    funcs = self.get\_all\_funcs(right\_var)
                    # print(funcs)
                    if not is\_re:
                        for func in funcs:
                            is\_co, cp = self.is\_controllable(func)
                            if is\_co == 1:
                                return is\_co, cp, expr\_lineno

                if param == param\_node and is\_re is True:
                    is\_co = 0
                    cp = None
                    return is\_co, cp, expr\_lineno

                if param == param\_node and not isinstance(param\_expr, list):  # 找到变量的来源，开始继续分析变量的赋值表达式是否可控
                    is\_co, cp = self.is\_controllable(param\_expr)  # 开始判断变量是否可控

                    if is\_co != 1:
                        is\_co, cp = self.is\_sink\_function(param\_expr, function\_params)

                    param = param\_expr  # 每次找到一个污点的来源时，开始跟踪新污点，覆盖旧污点

                if param == param\_node and isinstance(param\_expr, list):
                    for expr in param\_expr:
                        param = expr
                        is\_co, cp = self.is\_controllable(expr)

                        if is\_co == 1:
                            return is\_co, cp, expr\_lineno

                        \_is\_co, \_cp, expr\_lineno = self.parameters\_back(param, nodes\[:-1\], function\_params, node\_lineno)

                        if \_is\_co != -1:  # 当参数可控时，值赋给is\_co 和 cp，有一个参数可控，则认定这个函数可能可控
                            is\_co = \_is\_co
                            cp = \_cp

            if is\_co == -1:  # 当is\_co为True时找到可控，停止递归
                is\_co, cp, expr\_lineno = self.parameters\_back(param, nodes\[:-1\], function\_params, node\_lineno)  # 找到可控的输入时，停止递归

        # 如果是变量来源在函数的形参中,其实需要获取到函数名/函数所在行
        elif len(nodes) == 0 and function\_params is not None:
            for function\_param in function\_params:
                if function\_param == param:
                    is\_co = 2
                    cp = function\_param
                    expr\_lineno = node\_lineno

        return is\_co, cp, expr\_lineno


```

### `analysis_functioncall_node`处理函数调用节点

```
def analysis\_functioncall\_node(self, node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    """
    处理FunctionCall类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
    :param node:
    :param back\_node:
    :param vul\_function:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    # print('\[AST\] vul\_function:{v}'.format(v=vul\_function))
    params = set(list(self.get\_all\_params(node.arguments)))
    for param in params:
        is\_co, cp, expr\_lineno = self.parameters\_back(param, back\_node, function\_params)
        self.set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)


```

### `get_function_params`提取函数的参数

```
def get\_function\_params(self, nodes):
    """
    获取用户自定义函数的所有入参
    :param nodes: 自定义函数的参数部分
    :return params: 以列表的形式返回所有的入参
    """
    params = \[\]
    for node in nodes:
        if isinstance(node, FormalParameter):
            params.append(node.name)
    return list(set(params))


```

### `get_all_params`获取函数的参数列表

```
def get\_all\_params(self, nodes):  # 用来获取调用函数的参数列表，nodes为参数列表
    """
    获取函数结构的所有参数
    :param nodes: 输入MethodInvocation.arguments 作为nodes
    :return params: 返回这个函数参数列表中涉及的全部变量
    """
    params = \[\]
    export\_params = \[\]  # 定义空列表，用来给export\_list中使用
    for node in nodes:
        if isinstance(node, MethodInvocation):  # 函数参数来自另一个函数的返回值
            params = self.get\_all\_params(node.arguments)
        else:
            if isinstance(node, MemberReference):
                params.append(node.member)
            elif isinstance(node, BinaryOperation):
                params = self.get\_binaryop\_params(node)
                params = self.export\_list(params, export\_params)
    return list(set(params))


```

### `get_all_funcs`获取节点下所有函数调用

```
def get\_all\_funcs(self, node, tmp=\[\]):
    funcs = \[node.member\]
    export\_funcs = \[\]  # 定义空列表，用来给export\_list中使用
    for node in node.arguments:
        if isinstance(node, MethodInvocation):  # 函数参数来自另一个函数的返回值
            funcs.append(node.member)
            funcs = list(self.export\_list(funcs, export\_funcs))
        # if isinstance(node, MethodInvocation)
        # return get\_all\_funcs(node)
    return list(set(funcs))


```

### `analysis_binaryop_node`处理二元运算

```
def analysis\_binaryop\_node(self, node, back\_node, vul\_function, vul\_lineno, function\_params=None):
    """
    处理BinaryOp类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
    :param node:
    :param back\_node:
    :param vul\_function:
    :param vul\_lineno:
    :param function\_params:
    :return:
    """
    # print('\[AST\] vul\_function:{v}'.format(v=vul\_function))
    export\_params = \[\]
    params = self.get\_binaryop\_params(node)
    params = self.export\_list(params, export\_params)

    for param in params:
        is\_co, cp, expr\_lineno = self.parameters\_back(param, back\_node, function\_params)
        self.set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)


```

### `get_binaryop_deep_params`处理多层二元运算

```
def get\_binaryop\_deep\_params(self, node, params):  # 取出right，left不为变量时，对象结构中的变量
    """
    递归取出深层的变量名
    :param node: node为 get\_binaryop\_params 中的 node.operandl 或者 node.operandr 节点
    :param params: 传进来之前的参数
    :return params: 返回深层的参数列表
    """
    if isinstance(node, BinaryOperation):  # node为BinaryOp，递归取出其中变量
        param = self.get\_binaryop\_params(node)
        params.append(param)
    if isinstance(node, MethodInvocation):  # node为FunctionCall，递归取出其中变量名
        params = self.get\_all\_params(node.arguments)
    return params


```

### `get_binaryop_params`提取二元运算涉及的变量

```
def get\_binaryop\_params(self, node):  # 当为BinaryOp类型时，分别对left和right进行处理，取出需要的变量
    """
    用来提取Binaryop中的参数
    :param node: 输入一个BinaryOperation节点
    :return params: 返回当前节点涉及的变量列表
    """
    # print('\[AST\] Binaryop --> {node}'.format(node=node))
    params = \[\]
    buffer\_ = \[\]

    if isinstance(node.operandl, MemberReference) or isinstance(node.operandr,
                                                                MemberReference):  # left, right都为变量直接取值
        if isinstance(node.operandl, MemberReference):
            params.append(node.operandl.member)

        if isinstance(node.operandr, MemberReference):
            params.append(node.operandr.member)

    if not isinstance(node.operandl, MemberReference) or not isinstance(node.operandr,
                                                                        MemberReference):  # right不为变量时
        params\_right = self.get\_binaryop\_deep\_params(node.operandr, params)
        params\_left = self.get\_binaryop\_deep\_params(node.operandl, params)

        params = params\_left + params\_right

    params = self.export\_list(params, buffer\_)
    return params


```

### `analysis_if_else`分析判断语句

```
def analysis\_if\_else(self, node, vul\_function, back\_node, vul\_lineno, function\_params=None):
    nodes = \[\]
    if isinstance(node.then\_statement, BlockStatement):
        self.analysis(node.then\_statement.statements, vul\_function, back\_node, vul\_lineno, function\_params)

    if isinstance(node.else\_statement, BlockStatement):
        self.analysis(node.else\_statement.statements, vul\_function, back\_node, vul\_lineno, function\_params)

    if isinstance(node.else\_statement, IfStatement):
        self.analysis\_if\_else(node.else\_statement, vul\_function, back\_node, vul\_lineno, function\_params)


```

### `is_sink_function`判断函数入参是否进入

```
def is\_sink\_function(self, param\_expr, function\_params):
    """
    判断指定函数函数的入参-->判断此函数是否是危险函数
    :param param\_expr: 传入一个变量名
    :param function\_params: 该函数的入参
    :return: 如果该变量名在函数定义的入参中,也认为可控返回True
    """
    is\_co = -1
    cp = None
    if function\_params is not None:
        for function\_param in function\_params:
            if param\_expr == function\_param:
                is\_co = 2
                cp = function\_param
                # print('\[AST\] is\_sink\_function --> {function\_param}'.format(function\_param=cp))
    return is\_co, cp


```

### `is_controllable`判断复制表达式是否可控

```
def is\_controllable(self, expr):  # 获取表达式中的变量，看是否在用户可控变量列表中
    """
    判断赋值表达式是否是用户可控的
    :param expr: 传入一个函数名
    :return 1, expr: 如果该函数是敏感函数就返回 1,函数名
    """
    controlled\_params = \[
        'getParameter'
        # '$\_GET',
        # '$\_POST',
        # '$\_REQUEST',
        # '$\_COOKIE',
        # '$\_FILES',
        # '$\_SERVER',
        # '$HTTP\_POST\_FILES',
        # '$HTTP\_COOKIE\_VARS',
        # '$HTTP\_REQUEST\_VARS',
        # '$HTTP\_POST\_VARS',
        # '$HTTP\_RAW\_POST\_DATA',
        # '$HTTP\_GET\_VARS'
    \]
    if expr in controlled\_params:
        # print('\[AST\] is\_controllable --> {expr}'.format(expr=expr))
        return 1, expr
    return -1, None


```

### `is_repair`判断赋值表达式中是否有过滤函数

```
def is\_repair(self, expr):
    """
    判断赋值表达式是否出现过滤函数，如果已经过滤，停止污点回溯，判定漏洞已修复
    :param expr: 这里应该是函数名称
    :return is\_re: 返回是否已经修复 boolean
    """
    is\_re = False  # 是否修复，默认值是未修复
    for repair in self.repairs:
        if expr == repair:
            is\_re = True
            return is\_re
    return is\_re

def is\_sink\_function(self, param\_expr, function\_params):
    """
    判断指定函数函数的入参-->判断此函数是否是危险函数
    :param param\_expr: 传入一个变量名
    :param function\_params: 该函数的入参
    :return: 如果该变量名在函数定义的入参中,也认为可控返回True
    """
    is\_co = -1
    cp = None
    if function\_params is not None:
        for function\_param in function\_params:
            if param\_expr == function\_param:
                is\_co = 2
                cp = function\_param
                # print('\[AST\] is\_sink\_function --> {function\_param}'.format(function\_param=cp))
    return is\_co, cp


```

### `set_scan_results`存储结果

```
def set\_scan\_results(self, is\_co, cp, expr\_lineno, sink, param, vul\_lineno):
    """
    获取结果信息-->输出结果
    :param is\_co:
    :param cp:
    :param expr\_lineno:
    :param sink:
    :param param:
    :param vul\_lineno:
    :return:
    """
    results = \[\]
    # global scan\_results

    result = {
        'code': is\_co,
        'source': cp,
        'source\_lineno': expr\_lineno,
        'sink': sink,
        'sink\_param:': param,
        'sink\_lineno': vul\_lineno
    }
    # for scan\_result in scan\_results:
    #     if

    if result\['code'\] != -1:  # 查出来漏洞结果添加到结果信息中
        results.append(result)
        self.scan\_results += results


```

测试代码
----

### 测试文件

历史漏洞: [某知名 OA e-cology WorkflowCenterTreeData 前台接口 SQL 注入漏洞复现_数据库_小龙人 - CSDN 博客](https://blog.csdn.net/zycdn/article/details/102494037)

`java_src/_workflowcentertreedata__jsp.java`

```
/\*
 \* JSP generated by Resin-3.1.8 (built Mon, 17 Nov 2008 12:15:21 PST)
 \*/

package \_jsp.\_mobile.\_browser;

import javax.servlet.\*;
import javax.servlet.jsp.\*;
import javax.servlet.http.\*;

import org.json.\*;
import weaver.general.Util;

import java.util.\*;

import weaver.workflow.workflow.WorkTypeComInfo;

public class \_workflowcentertreedata\_\_jsp extends com.caucho.jsp.JavaPage {
    private static final java.util.HashMap<String, java.lang.reflect.Method> \_jsp\_functionMap = new java.util.HashMap<String, java.lang.reflect.Method>();
    private boolean \_caucho\_isDead;

    public void
    \_jspService(javax.servlet.http.HttpServletRequest request,
                javax.servlet.http.HttpServletResponse response)
            throws java.io.IOException, javax.servlet.ServletException {
        javax.servlet.http.HttpSession session = request.getSession(true);
        com.caucho.server.webapp.WebApp \_jsp\_application = \_caucho\_getApplication();
        javax.servlet.ServletContext application = \_jsp\_application;
        com.caucho.jsp.PageContextImpl pageContext = \_jsp\_application.getJspApplicationContext().allocatePageContext(this, \_jsp\_application, request, response, null, session, 8192, true, false);
        javax.servlet.jsp.PageContext \_jsp\_parentContext = pageContext;
        javax.servlet.jsp.JspWriter out = pageContext.getOut();
        final javax.el.ELContext \_jsp\_env = pageContext.getELContext();
        javax.servlet.ServletConfig config = getServletConfig();
        javax.servlet.Servlet page = this;
        response.setContentType("application/x-json;charset=UTF-8");
        request.setCharacterEncoding("UTF-8");
        try {
            out.write(\_jsp\_string0, 0, \_jsp\_string0.length);
            weaver.conn.RecordSet rs;
            rs = (weaver.conn.RecordSet) pageContext.getAttribute("rs");
            if (rs == null) {
                rs = new weaver.conn.RecordSet();
                pageContext.setAttribute("rs", rs);
            }
            out.write(\_jsp\_string1, 0, \_jsp\_string1.length);
            weaver.conn.RecordSet rsIn;
            rsIn = (weaver.conn.RecordSet) pageContext.getAttribute("rsIn");
            if (rsIn == null) {
                rsIn = new weaver.conn.RecordSet();
                pageContext.setAttribute("rsIn", rsIn);
            }
            out.write(\_jsp\_string2, 0, \_jsp\_string2.length);

            String node = Util.null2String(request.getParameter("node"));
            String arrNode\[\] = Util.TokenizerString2(node, "\_");
            String type = arrNode\[0\];
            String value = arrNode\[1\];

            String flowids = "";
            ArrayList flowidList = new ArrayList();

            String scope = Util.null2String(request.getParameter("scope"));
            String initvalue = Util.null2String(request.getParameter("initvalue"));
            String formids = Util.null2String(request.getParameter("formids"));

            rs.executeSql("select \* from mobileconfig where mc\_type=5 and mc\_scope=" + scope + " and mc\_);
            if (rs.next()) {
                flowids = Util.null2String(rs.getString("mc\_value"));
            }

            if (initvalue != null && !"".equals(initvalue)) {
                flowids += "," + initvalue;
                flowidList = Util.TokenizerString(flowids, ",");
            }

            JSONArray jsonArrayReturn = new JSONArray();

            if ("root".equals(type)) { //\\u4e3b\\u76ee\\u5f55\\u4e0b\\u7684\\u6570\\u636e
                WorkTypeComInfo wftc = new WorkTypeComInfo();
                while (wftc.next()) {
                    JSONObject jsonTypeObj = null;
                    String wfTypeId = wftc.getWorkTypeid();
                    String wfTypeName = wftc.getWorkTypename();
                    //if("1".equals(wfTypeId)) continue;
                    rs.executeSql("select id,workflowname from workflow\_base where isvalid='1' and workflowtype=" + wfTypeId + " and  ( isbill=0 or (isbill=1 and formid<0) or (isbill=1 and formid in (" + formids + ")))");
                    while (rs.next()) {
                        jsonTypeObj = new JSONObject();
                        String wfId = Util.null2String(rs.getString("id"));
                        if (flowidList.contains(wfId)) {
                            jsonTypeObj.put("expanded", true);
                            break;
                        }

                    }
                    if (jsonTypeObj != null) {
                        jsonTypeObj.put("id", "wftype\_" + wfTypeId);
                        jsonTypeObj.put("text", wfTypeName);
                        jsonTypeObj.put("checked", false);
                        jsonTypeObj.put("draggable", false);
                        jsonTypeObj.put("leaf", false);
                        jsonArrayReturn.put(jsonTypeObj);
                    }
                }
            } else if ("wftype".equals(type)) {
                rs.executeSql("select id,workflowname from workflow\_base where isvalid='1' and workflowtype=" + value + " and ( isbill=0 or (isbill=1 and formid<0) or (isbill=1 and formid in (" + formids + ")))");

                while (rs.next()) {

                    JSONObject jsonWfObj = new JSONObject();
                    String wfId = Util.null2String(rs.getString("id"));
                    String wfName = Util.null2String(rs.getString("workflowname"));
                    jsonWfObj.put("id", "wf\_" + wfId);
                    jsonWfObj.put("text", wfName);
                    jsonWfObj.put("draggable", false);

                    if (!flowidList.contains(wfId)) {
                        jsonWfObj.put("checked", false);
                    } else {
                        jsonWfObj.put("checked", true);
                        jsonWfObj.put("expanded", true);
                    }
                    jsonWfObj.put("leaf", true);
                    jsonArrayReturn.put(jsonWfObj);
                }
            }
            out.println(jsonArrayReturn.toString());

            out.write(\_jsp\_string1, 0, \_jsp\_string1.length);
        } catch (java.lang.Throwable \_jsp\_e) {
            pageContext.handlePageException(\_jsp\_e);
        } finally {
            \_jsp\_application.getJspApplicationContext().freePageContext(pageContext);
        }
    }

    private java.util.ArrayList \_caucho\_depends = new java.util.ArrayList();

    public java.util.ArrayList \_caucho\_getDependList() {
        return \_caucho\_depends;
    }

    public void \_caucho\_addDepend(com.caucho.vfs.PersistentDependency depend) {
        super.\_caucho\_addDepend(depend);
        com.caucho.jsp.JavaPage.addDepend(\_caucho\_depends, depend);
    }

    public boolean \_caucho\_isModified() {
        if (\_caucho\_isDead)
            return true;
        if (com.caucho.server.util.CauchoSystem.getVersionId() != 1886798272571451039L)
            return true;
        for (int i = \_caucho\_depends.size() - 1; i >= 0; i--) {
            com.caucho.vfs.Dependency depend;
            depend = (com.caucho.vfs.Dependency) \_caucho\_depends.get(i);
            if (depend.isModified())
                return true;
        }
        return false;
    }

    public long \_caucho\_lastModified() {
        return 0;
    }

    public java.util.HashMap<String, java.lang.reflect.Method> \_caucho\_getFunctionMap() {
        return \_jsp\_functionMap;
    }

    public void init(ServletConfig config)
            throws ServletException {
        com.caucho.server.webapp.WebApp webApp
                = (com.caucho.server.webapp.WebApp) config.getServletContext();
        super.init(config);
        com.caucho.jsp.TaglibManager manager = webApp.getJspApplicationContext().getTaglibManager();
        com.caucho.jsp.PageContextImpl pageContext = new com.caucho.jsp.PageContextImpl(webApp, this);
    }

    public void destroy() {
        \_caucho\_isDead = true;
        super.destroy();
    }

    public void init(com.caucho.vfs.Path appDir)
            throws javax.servlet.ServletException {
        com.caucho.vfs.Path resinHome = com.caucho.server.util.CauchoSystem.getResinHome();
        com.caucho.vfs.MergePath mergePath = new com.caucho.vfs.MergePath();
        mergePath.addMergePath(appDir);
        mergePath.addMergePath(resinHome);
        com.caucho.loader.DynamicClassLoader loader;
        loader = (com.caucho.loader.DynamicClassLoader) getClass().getClassLoader();
        String resourcePath = loader.getResourcePathSpecificFirst();
        mergePath.addClassPath(resourcePath);
        com.caucho.vfs.Depend depend;
        depend = new com.caucho.vfs.Depend(appDir.lookup("mobile/browser/WorkflowCenterTreeData.jsp"), -7926612934612916794L, false);
        com.caucho.jsp.JavaPage.addDepend(\_caucho\_depends, depend);
    }

    private final static char\[\] \_jsp\_string0;
    private final static char\[\] \_jsp\_string1;
    private final static char\[\] \_jsp\_string2;

    static {
        \_jsp\_string0 = "\\r\\n\\r\\n\\r\\n\\r\\n\\r\\n\\r\\n".toCharArray();
        \_jsp\_string1 = "\\r\\n".toCharArray();
        \_jsp\_string2 = "\\r\\n\\r\\n".toCharArray();
    }
}


```

### 分析代码

`java_parser_class.py`

```
\# -\*- coding: utf-8 -\*-
import os
from functools import reduce

from javalang.parse import parse
from javalang.tree import \*
import javalang
import copy

fp = open("res\_test.txt", 'a+')
# fp.write("type\\tfilename\\tparam\_line\\tsink\_line\\n")
class JavaParse():

    def \_\_init\_\_(self, filename):
        self.filename = filename  # r"java\_src\\\_workflowcentertreedata\_\_jsp.java"
        self.src = open(self.filename, 'r', encoding='utf8', errors='ignore').read()

        self.with\_line = True
        self.scan\_results = \[\]  # 结果存放列表初始化
        self.repairs = \[\]  # 用于存放修复函数

    def export(self, items):
        """
        #todo 暂时不知道干啥的,好像是用来打印的
        :param items:
        :return:
        """
        result = \[\]
        if items:
            for item in items:
                if hasattr(item, 'generic'):
                    item = item.generic(with\_lineno=self.with\_line)
                result.append(item)
        return result

    def export\_list(self, params1, export\_params1):
        """
        将params中嵌套的多个列表，导出为一个列表
        :param params: 输入一个嵌套类的参数列表
        :param export\_params: 要合并且输出的列表
        :return export\_params: 输出一个没有嵌套的列表
        """

        params = copy.deepcopy(params1)
        export\_params = copy.deepcopy(export\_params1)
        # print(params)
        # print(export\_params)
        for param in params:
            if isinstance(param, list):
                # print(1)
                export\_params = self.export\_list(param, export\_params)
            else:
                # print(2)
                export\_params.append(param)
                # print(export\_params)
        # print("return")
        return list(set(export\_params))

    def get\_all\_funcs(self, node, tmp=\[\]):
        funcs = \[node.member\]
        export\_funcs = \[\]  # 定义空列表，用来给export\_list中使用
        for node in node.arguments:
            if isinstance(node, MethodInvocation):  # 函数参数来自另一个函数的返回值
                funcs.append(node.member)
                funcs = list(self.export\_list(funcs, export\_funcs))
            # if isinstance(node, MethodInvocation)
            # return get\_all\_funcs(node)
        return list(set(funcs))

    # def get\_all\_funcs(node):
    #     funcs = \[node.qualifier + "." + node.member\]
    #     export\_funcs = \[\]  # 定义空列表，用来给export\_list中使用
    #     for node in node.arguments:
    #         if isinstance(node, MethodInvocation):  # 函数参数来自另一个函数的返回值
    #             funcs.append(node.qualifier + "." + node.member)
    #             funcs = export\_list(funcs, export\_funcs)
    #             # return get\_all\_funcs(node)
    #     return funcs

    def get\_all\_params(self, nodes):  # 用来获取调用函数的参数列表，nodes为参数列表
        """
        获取函数结构的所有参数
        :param nodes: 输入MethodInvocation.arguments 作为nodes
        :return params: 返回这个函数参数列表中涉及的全部变量
        """
        params = \[\]
        export\_params = \[\]  # 定义空列表，用来给export\_list中使用
        for node in nodes:
            if isinstance(node, MethodInvocation):  # 函数参数来自另一个函数的返回值
                params = self.get\_all\_params(node.arguments)
            else:
                if isinstance(node, MemberReference):
                    params.append(node.member)
                elif isinstance(node, BinaryOperation):
                    params = self.get\_binaryop\_params(node)
                    params = self.export\_list(params, export\_params)
        return list(set(params))

    def get\_binaryop\_params(self, node):  # 当为BinaryOp类型时，分别对left和right进行处理，取出需要的变量
        """
        用来提取Binaryop中的参数
        :param node: 输入一个BinaryOperation节点
        :return params: 返回当前节点涉及的变量列表
        """
        # print('\[AST\] Binaryop --> {node}'.format(node=node))
        params = \[\]
        buffer\_ = \[\]

        if isinstance(node.operandl, MemberReference) or isinstance(node.operandr,
                                                                    MemberReference):  # left, right都为变量直接取值
            if isinstance(node.operandl, MemberReference):
                params.append(node.operandl.member)

            if isinstance(node.operandr, MemberReference):
                params.append(node.operandr.member)

        if not isinstance(node.operandl, MemberReference) or not isinstance(node.operandr,
                                                                            MemberReference):  # right不为变量时
            params\_right = self.get\_binaryop\_deep\_params(node.operandr, params)
            params\_left = self.get\_binaryop\_deep\_params(node.operandl, params)

            params = params\_left + params\_right

        params = self.export\_list(params, buffer\_)
        return params

    def get\_binaryop\_deep\_params(self, node, params):  # 取出right，left不为变量时，对象结构中的变量
        """
        递归取出深层的变量名
        :param node: node为 get\_binaryop\_params 中的 node.operandl 或者 node.operandr 节点
        :param params: 传进来之前的参数
        :return params: 返回深层的参数列表
        """
        if isinstance(node, BinaryOperation):  # node为BinaryOp，递归取出其中变量
            param = self.get\_binaryop\_params(node)
            params.append(param)
        if isinstance(node, MethodInvocation):  # node为FunctionCall，递归取出其中变量名
            params = self.get\_all\_params(node.arguments)
        return params

    # todo
    def get\_expr\_name(self, node):  # expr为'expr'中的值
        """
        获取赋值表达式的表达式部分中的参数名(变量名)-->返回用来进行回溯
        :param node: 输入一个节点(要求是一个表达式的右值), 检测表达式包含的所有变量
        :return param\_expr: 返回表达式中涉及的所有变量的列表 \[\]
        :return param\_lineno: 返回当前表达式所在行 int
        :return is\_re: 返回是否已经修复  boolean
        """
        # todo 这里有个坑. javalang有position缺失的情况.可能会发生变量回溯丢失
        param\_lineno = 0
        is\_re = False
        param\_expr = None


        if isinstance(node, MemberReference):  # 当赋值表达式为变量
            param\_expr = node.member  # 返回变量名
            param\_lineno = node.position.line

        elif isinstance(node, MethodInvocation):  # 当赋值表达式为函数
            param\_expr = self.get\_all\_params(node.arguments)  # 返回函数参数列表
            param\_lineno = node.position.line
            # function\_name = node.qualifier + "." + node.member
            is\_re = False
            # 调用了函数，判断调用的函数是否为修复函数
            for func in self.get\_all\_funcs(node):
                if self.is\_repair(func):
                    is\_re = True
                    break

        elif isinstance(node, BinaryOperation):  # 当赋值表达式为BinaryOp
            param\_expr = self.get\_binaryop\_params(node)
            # todo 需要修复javalang的 position 丢失的问题 这里先硬编码一下
            # param\_lineno = node.position.line
            param\_lineno = 7

        elif isinstance(node, Assignment):  # 当赋值表达式为Assignment
            param\_expr, param\_lineno, is\_re = self.get\_expr\_name(node.value)
            # param\_lineno = node.position.line

        elif isinstance(node, This):  # 当赋值表达式为 This
            for selector in node.selectors:
                param\_expr, param\_lineno, is\_re = self.get\_expr\_name(selector)
                if is\_re:
                    return param\_expr, param\_lineno, is\_re
        else:
            param\_expr = node
            # print(param\_expr)
        # print(param\_expr)
        return param\_expr, param\_lineno, is\_re

    def get\_node\_name(self, node):  # node为'node'中的元组
        """
        获取MemberReference类型节点的name
        :param node: 一般是MemberReference,字面量啥的不需要跟踪
        :return: MemberReference.member
        """
        if isinstance(node, MemberReference):
            return node.member  # 返回此节点中的变量名
        elif isinstance(node, VariableDeclarator):
            return node.name  # 返回此节点中的变量名

    def is\_repair(self, expr):
        """
        判断赋值表达式是否出现过滤函数，如果已经过滤，停止污点回溯，判定漏洞已修复
        :param expr: 这里应该是函数名称
        :return is\_re: 返回是否已经修复 boolean
        """
        is\_re = False  # 是否修复，默认值是未修复
        for repair in self.repairs:
            if expr == repair:
                is\_re = True
                return is\_re
        return is\_re

    def is\_sink\_function(self, param\_expr, function\_params):
        """
        判断指定函数函数的入参-->判断此函数是否是危险函数
        :param param\_expr: 传入一个变量名
        :param function\_params: 该函数的入参
        :return: 如果该变量名在函数定义的入参中,也认为可控返回True
        """
        is\_co = -1
        cp = None
        if function\_params is not None:
            for function\_param in function\_params:
                if param\_expr == function\_param:
                    is\_co = 2
                    cp = function\_param
                    # print('\[AST\] is\_sink\_function --> {function\_param}'.format(function\_param=cp))
        return is\_co, cp

    def is\_controllable(self, expr):  # 获取表达式中的变量，看是否在用户可控变量列表中
        """
        判断赋值表达式是否是用户可控的
        :param expr: 传入一个函数名
        :return 1, expr: 如果该函数是敏感函数就返回 1,函数名
        """
        controlled\_params = \[
            'getParameter'
            # '$\_GET',
            # '$\_POST',
            # '$\_REQUEST',
            # '$\_COOKIE',
            # '$\_FILES',
            # '$\_SERVER',
            # '$HTTP\_POST\_FILES',
            # '$HTTP\_COOKIE\_VARS',
            # '$HTTP\_REQUEST\_VARS',
            # '$HTTP\_POST\_VARS',
            # '$HTTP\_RAW\_POST\_DATA',
            # '$HTTP\_GET\_VARS'
        \]
        if expr in controlled\_params:
            # print('\[AST\] is\_controllable --> {expr}'.format(expr=expr))
            return 1, expr
        return -1, None

    def parameters\_back(self, param, nodes, function\_params=None, node\_lineno=-1):  # 用来得到回溯过程中的被赋值的变量是否与敏感函数变量相等,param是当前需要跟踪的污点
        """
        递归回溯敏感函数的赋值流程，param为跟踪的污点，当找到param来源时-->分析复制表达式-->获取新污点；否则递归下一个节点
        :param param: 输入一个变量名
        :param nodes: nodes 也就是之前访问的back\_nodes,里面基本都是LocalVariableDeclaration/StatementExpression/IFxxx
        :param function\_params: 递归过程中保持函数的形参,如果变量是从形参获得也认为可控
        :return is\_co, cp, expr\_lineno: 可控返回1 , 可控的变量名, 变量所在行
        """
        # node\_lineno = -1
        # print(node\_lineno)
        if len(nodes) > 0 and node\_lineno == -1:
            node\_lineno = nodes\[0\].position.line  # source所在行号
        expr\_lineno = 0
        is\_re = False
        is\_co, cp = self.is\_controllable(param)

        if len(nodes) != 0 and is\_co == -1:
            node = nodes\[len(nodes) - 1\]
            # if isinstance(node, LocalVariableDeclaration):
            tnodes = \[\]
            if isinstance(node, LocalVariableDeclaration):  # 回溯的过程中，对出现赋值情况的节点进行跟踪
                if isinstance(node, LocalVariableDeclaration):
                    tnodes = \[\[declarator, declarator.initializer\] for declarator in node.declarators\]
            elif isinstance(node, StatementExpression):
                if isinstance(node.expression, Assignment):
                    tnodes = \[\[node.expression.expressionl, node.expression.value\]\]

            for left\_var, right\_var in tnodes:
                param\_node = self.get\_node\_name(left\_var)
                # param\_expr为赋值表达式,param\_expr为变量或者列表
                param\_expr, expr\_lineno, is\_re = self.get\_expr\_name(right\_var)

                if param == param\_node and is\_re is False and isinstance(right\_var, MethodInvocation):
                    funcs = self.get\_all\_funcs(right\_var)
                    # print(funcs)
                    if not is\_re:
                        for func in funcs:
                            is\_co, cp = self.is\_controllable(func)
                            if is\_co == 1:
                                return is\_co, cp, expr\_lineno

                if param == param\_node and is\_re is True:
                    is\_co = 0
                    cp = None
                    return is\_co, cp, expr\_lineno

                if param == param\_node and not isinstance(param\_expr, list):  # 找到变量的来源，开始继续分析变量的赋值表达式是否可控
                    is\_co, cp = self.is\_controllable(param\_expr)  # 开始判断变量是否可控

                    if is\_co != 1:
                        is\_co, cp = self.is\_sink\_function(param\_expr, function\_params)

                    param = param\_expr  # 每次找到一个污点的来源时，开始跟踪新污点，覆盖旧污点

                if param == param\_node and isinstance(param\_expr, list):
                    for expr in param\_expr:
                        param = expr
                        is\_co, cp = self.is\_controllable(expr)

                        if is\_co == 1:
                            return is\_co, cp, expr\_lineno

                        \_is\_co, \_cp, expr\_lineno = self.parameters\_back(param, nodes\[:-1\], function\_params, node\_lineno)

                        if \_is\_co != -1:  # 当参数可控时，值赋给is\_co 和 cp，有一个参数可控，则认定这个函数可能可控
                            is\_co = \_is\_co
                            cp = \_cp

            if is\_co == -1:  # 当is\_co为True时找到可控，停止递归
                is\_co, cp, expr\_lineno = self.parameters\_back(param, nodes\[:-1\], function\_params, node\_lineno)  # 找到可控的输入时，停止递归

        # 如果是变量来源在函数的形参中,其实需要获取到函数名/函数所在行
        elif len(nodes) == 0 and function\_params is not None:
            for function\_param in function\_params:
                if function\_param == param:
                    is\_co = 2
                    cp = function\_param
                    expr\_lineno = node\_lineno

        return is\_co, cp, expr\_lineno

    def get\_function\_params(self, nodes):
        """
        获取用户自定义函数的所有入参
        :param nodes: 自定义函数的参数部分
        :return params: 以列表的形式返回所有的入参
        """
        params = \[\]
        for node in nodes:
            if isinstance(node, FormalParameter):
                params.append(node.name)
        return list(set(params))

    def anlysis\_function(self, node, back\_node, vul\_function, function\_params, vul\_lineno):
        """
        对用户自定义的函数进行分析-->获取函数入参-->入参用经过赋值流程，进入sink函数-->此自定义函数为危险函数
        最终目的是分析函数调用
        :param node: 传入一个 MethodDeclaration 类型节点
        :param back\_node: 传入 back\_nodes
        :param vul\_function: 存在漏洞的函数名
        :param function\_params: 函数的形参(从 MethodDeceleration 节点进来的话)
        :param vul\_lineno:
        :return:
        """
        global scan\_results
        # try:
        if node.member == vul\_function and int(node.position.line) == int(vul\_lineno):  # 函数体中存在敏感函数，开始对敏感函数前的代码进行检测
            for param in node.arguments:
                if isinstance(param, MemberReference):
                    self.analysis\_variable\_node(param, back\_node, vul\_function, vul\_lineno, function\_params)

                elif isinstance(param, MethodInvocation):
                    self.analysis\_functioncall\_node(param, back\_node, vul\_function, vul\_lineno, function\_params)

                elif isinstance(param, BinaryOperation):
                    self.analysis\_binaryop\_node(param, back\_node, vul\_function, vul\_lineno, function\_params)

        # except Exception as e:
        #     print(e)

    def analysis\_binaryop\_node(self, node, back\_node, vul\_function, vul\_lineno, function\_params=None):
        """
        处理BinaryOp类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
        :param node:
        :param back\_node:
        :param vul\_function:
        :param vul\_lineno:
        :param function\_params:
        :return:
        """
        # print('\[AST\] vul\_function:{v}'.format(v=vul\_function))
        export\_params = \[\]
        params = self.get\_binaryop\_params(node)
        params = self.export\_list(params, export\_params)

        for param in params:
            is\_co, cp, expr\_lineno = self.parameters\_back(param, back\_node, function\_params)
            self.set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)

    def analysis\_functioncall\_node(self, node, back\_node, vul\_function, vul\_lineno, function\_params=None):
        """
        处理FunctionCall类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
        :param node:
        :param back\_node:
        :param vul\_function:
        :param vul\_lineno:
        :param function\_params:
        :return:
        """
        # print('\[AST\] vul\_function:{v}'.format(v=vul\_function))
        params = set(list(self.get\_all\_params(node.arguments)))
        for param in params:
            is\_co, cp, expr\_lineno = self.parameters\_back(param, back\_node, function\_params)
            self.set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)

    def analysis\_variable\_node(self, node, back\_node, vul\_function, vul\_lineno, function\_params=None):
        """
        处理Variable类型节点-->取出参数-->回溯判断参数是否可控-->输出结果
        这里直接将最后一步回溯到的变量写入全局结果表中,并不包含路径
        :param node:
        :param back\_node:
        :param vul\_function:
        :param vul\_lineno:
        :param function\_params:
        :return:
        """
        # print('\[AST\] vul\_function:{v}'.format(v=vul\_function))
        param = self.get\_node\_name(node)
        is\_co, cp, expr\_lineno = self.parameters\_back(param, back\_node, function\_params)
        self.set\_scan\_results(is\_co, cp, expr\_lineno, vul\_function, param, vul\_lineno)

    def analysis\_if\_else(self, node, vul\_function, back\_node, vul\_lineno, function\_params=None):
        nodes = \[\]
        if isinstance(node.then\_statement, BlockStatement):
            self.analysis(node.then\_statement.statements, vul\_function, back\_node, vul\_lineno, function\_params)

        if isinstance(node.else\_statement, BlockStatement):
            self.analysis(node.else\_statement.statements, vul\_function, back\_node, vul\_lineno, function\_params)

        if isinstance(node.else\_statement, IfStatement):
            self.analysis\_if\_else(node.else\_statement, vul\_function, back\_node, vul\_lineno, function\_params)

    def set\_scan\_results(self, is\_co, cp, expr\_lineno, sink, param, vul\_lineno):
        """
        获取结果信息-->输出结果
        :param is\_co:
        :param cp:
        :param expr\_lineno:
        :param sink:
        :param param:
        :param vul\_lineno:
        :return:
        """
        results = \[\]
        # global scan\_results

        result = {
            'code': is\_co,
            'source': cp,
            'source\_lineno': expr\_lineno,
            'sink': sink,
            'sink\_param:': param,
            'sink\_lineno': vul\_lineno
        }
        # for scan\_result in scan\_results:
        #     if

        if result\['code'\] != -1:  # 查出来漏洞结果添加到结果信息中
            results.append(result)
            self.scan\_results += results

    def analysis(self, nodes, vul\_function, back\_node, vul\_lineo, function\_params=None):
        """
        总体的思路是遍历所有节点且放入back\_nodes中
        -> 查找所有的 MethodInvocation 直到找到匹配 vul\_lineo 的那一个
        -> 然后在函数调用中查找出来涉及的变量
        ( anlysis\_function 就是进入函数体进行敏感函数查找而已,可以优化 )
        ( analysis\_functioncall\_node 就是取出敏感函数的参数(变量)进行 parameters\_back )

        :param nodes: 所有节点
        :param vul\_function: 要判断的敏感函数名
        :param back\_node: 各种语法结构里面的语句
        :param vul\_lineo: 漏洞函数所在行号
        :param function\_params: 自定义函数的所有参数列表
        :return:
        """
        buffer\_ = \[\]
        for node in nodes:
            if isinstance(node, MethodInvocation):
                # 从原文的意思看,这里是检测到函数调用,去找这个方法的MethodDeceleration,如果这个函数里面有敏感操作,就爆有问题
                self.anlysis\_function(node, back\_node, vul\_function, function\_params, vul\_lineo)

            elif isinstance(node, StatementExpression):
                if isinstance(node.expression, MethodInvocation):
                    self.anlysis\_function(node.expression, back\_node, vul\_function, function\_params, vul\_lineo)

                elif isinstance(node.expression, Assignment):
                    if isinstance(node.expression.value, MethodInvocation):
                        self.anlysis\_function(node.expression.value, back\_node, vul\_function, function\_params,
                                              vul\_lineo)
            # todo 这里还有 binop 的操作
            elif isinstance(node, LocalVariableDeclaration):
                for declarator in node.declarators:
                    if isinstance(declarator.initializer, MethodInvocation):
                        self.anlysis\_function(declarator.initializer, back\_node, vul\_function, function\_params,
                                              vul\_lineo)


            elif isinstance(node, IfStatement):  # 函数调用在if-else语句中时
                self.analysis\_if\_else(node, vul\_function, back\_node, vul\_lineo, function\_params)

            elif isinstance(node, TryStatement):  # 函数调用在try-catch-finally语句中时
                # print(back\_node)
                self.analysis(node.block, vul\_function, back\_node, vul\_lineo, function\_params)
                # analysis(node.catches, back\_node, vul\_function, vul\_lineo, function\_params)
                # analysis(node.finally\_block, back\_node, vul\_function, vul\_lineo, function\_params)

            elif isinstance(node, WhileStatement):
                self.analysis(node.body.statements, vul\_function, back\_node, vul\_lineo, function\_params)


            elif isinstance(node, ForStatement):
                if isinstance(node.body, BlockStatement):
                    self.analysis(node.body, vul\_function, back\_node, vul\_lineo, function\_params)


            elif isinstance(node, MethodDeclaration):
                function\_body = \[node\]
                function\_params = self.get\_function\_params(node.parameters)
                self.analysis(node.body, vul\_function, function\_body, vul\_lineo, function\_params=function\_params)


            elif isinstance(node, ClassDeclaration):
                self.analysis(node.body, vul\_function, back\_node, vul\_lineo, function\_params)
            # if back\_node == "executeSql":
            #     print(back\_node)
            back\_node.append(node)

    def scan\_parser(self, code\_content, sensitive\_func, vul\_lineno, repair):
        """
        先从 sensitive\_func 中提取敏感函数 func 循环查询AST
        ->进入analysis中查询 vul\_lineno 所在行的敏感函数调用
        :param code\_content: 要检测的文件内容
        :param sensitive\_func: 要检测的敏感函数,传入的为函数列表
        :param vul\_lineno: 漏洞函数所在行号
        :param repair: 对应漏洞的修复函数列表
        :return:
        """
        try:
            # global repairs
            # global scan\_results
            self.repairs = repair
            self.scan\_results = \[\]
            tree = javalang.parse.parse(code\_content)
            all\_nodes = tree.children\[-1\]
            for func in sensitive\_func:  # 循环判断代码中是否存在敏感函数，若存在，递归判断参数是否可控;对文件内容循环判断多次
                back\_node = \[\]
                self.analysis(all\_nodes, func, back\_node, int(vul\_lineno), function\_params=None)
        except SyntaxError as e:
            print('\[AST\] \[ERROR\]:{e}'.format(e=e))

        return self.scan\_results

    def run(self):
        code\_lines = self.src.split('\\n')
        run\_function = lambda x, y: x if y in x else x + \[y\]

        for i in range(code\_lines.\_\_len\_\_()):
            line = code\_lines\[i\]
            if 'executeSql' in line:
                print("\*" \* 50)
                print("executeSql in " + self.filename + ":" + str(i + 1))
                res = self.scan\_parser(self.src, \['executeSql'\], i + 1, \['null2int', 'getIntValue'\])
                res = reduce(run\_function, \[\[\], \] + res)
                print(res)
                for x in res:
                    print("##" \* 20 + "found sqli in " + self.filename + "##" \* 20)
                    if x\['code'\] > 0:
                        sink\_line = x\['sink\_lineno'\] - 1
                        source\_lineno = x\['source\_lineno'\] - 1
                        print("注入参数: ", x\['source\_lineno'\], " | ", code\_lines\[source\_lineno\].strip(" \\t"))
                        print("------------>")
                        print("注入点: ", x\['sink\_lineno'\], " | ", code\_lines\[sink\_line\].strip(" \\t"))
                        record = "%d\\t%s\\t%d\\t%d\\t%s\\n" % (x\['code'\], self.filename, x\['source\_lineno'\],  x\['sink\_lineno'\], code\_lines\[source\_lineno\].strip(" \\t"))
                        fp.write(record)
                        fp.flush()
                print("\\n")




import sys
import time
t = time.time()
if \_\_name\_\_ == '\_\_main\_\_':
    filename = "java\_src/Sqli.java"
    filename = r"java\_src/\_workflowcentertreedata\_\_jsp.java"
    # filename = sys.argv\[1\]
    print(filename)
    a = JavaParse(filename)
    a.run()
    print(time.time() - t)


# fp = open("res.txt", 'a+')


```

### 分析结果

可以很明显的看出, 存在如下注入点

`_workflowcentertreedata__jsp.java` -> `/mobile/browser/WorkflowCenterTreeData.jsp`

*   注入参数: line: 62 | `String scope = Util.null2String(request.getParameter("scope"));`
*   注入参数: line: 64 | `String formids = Util.null2String(request.getParameter("formids"));`
    
*   注入参数: line: 54 | `String node = Util.null2String(request.getParameter("node"));`

```
java\_src/\_workflowcentertreedata\_\_jsp.java
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
executeSql in java\_src/\_workflowcentertreedata\_\_jsp.java:66
\[{'code': 1, 'source': 'getParameter', 'source\_lineno': 62, 'sink': 'executeSql', 'sink\_param:': 'scope', 'sink\_lineno': 66}\]
########################################found sqli in java\_src/\_workflowcentertreedata\_\_jsp.java########################################
注入参数:  62  |  String scope = Util.null2String(request.getParameter("scope"));
------------>
注入点:  66  |  rs.executeSql("select \* from mobileconfig where mc\_type=5 and mc\_scope=" + scope + " and mc\_);


\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
executeSql in java\_src/\_workflowcentertreedata\_\_jsp.java:85
\[{'code': 1, 'source': 'getParameter', 'source\_lineno': 64, 'sink': 'executeSql', 'sink\_param:': 'formids', 'sink\_lineno': 85}\]
########################################found sqli in java\_src/\_workflowcentertreedata\_\_jsp.java########################################
注入参数:  64  |  String formids = Util.null2String(request.getParameter("formids"));
------------>
注入点:  85  |  rs.executeSql("select id,workflowname from workflow\_base where isvalid='1' and workflowtype=" + wfTypeId + " and  ( isbill=0 or (isbill=1 and formid<0) or (isbill=1 and formid in (" + formids + ")))");


\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
executeSql in java\_src/\_workflowcentertreedata\_\_jsp.java:105
\[{'code': 1, 'source': 'getParameter', 'source\_lineno': 64, 'sink': 'executeSql', 'sink\_param:': 'formids', 'sink\_lineno': 105}, {'code': 1, 'source': 'getParameter', 'source\_lineno': 54, 'sink': 'executeSql', 'sink\_param:': 'value', 'sink\_lineno': 105}\]
########################################found sqli in java\_src/\_workflowcentertreedata\_\_jsp.java########################################
注入参数:  64  |  String formids = Util.null2String(request.getParameter("formids"));
------------>
注入点:  105  |  rs.executeSql("select id,workflowname from workflow\_base where isvalid='1' and workflowtype=" + value + " and ( isbill=0 or (isbill=1 and formid<0) or (isbill=1 and formid in (" + formids + ")))");
########################################found sqli in java\_src/\_workflowcentertreedata\_\_jsp.java########################################
注入参数:  54  |  String node = Util.null2String(request.getParameter("node"));
------------>
注入点:  105  |  rs.executeSql("select id,workflowname from workflow\_base where isvalid='1' and workflowtype=" + value + " and ( isbill=0 or (isbill=1 and formid<0) or (isbill=1 and formid in (" + formids + ")))");


0.2094409465789795


```

总体分析结果
------

### 过滤后结果

结合前台访问响应码为 200 的 jsp 文件列表, 且直接为注入点, 不包含`二次sink`注入的注入点, 一个文件多个注入点没有去重, 共计 **160 处注入点**

![](https://blog.riskivy.com/wp-content/uploads/2020/05/45f5b07f30d683053fd37324ce7ceba2.png)

### 手工构造注入 EXP

经过手工构造注入, 去掉`某知名OA中表不存在`, `del语句注入`, `同一个文件不同注入点`, 剩余 **48 个成功 EXP**

![](https://blog.riskivy.com/wp-content/uploads/2020/05/1173238690ced8df35b69ef8ee4f32c0.png)

PS. 由于漏洞过多, /weaver / 接口下面映射 Servlet 就没有再继续分析, 欢迎一起研究自动化代码审计

优缺点分析
-----

### 优点

1\. 相比正则匹配漏洞, 通过遍历 AST 抽象语法树的形式, 能够获得代码中的上下文关系, 可以更准确的定位漏洞  
2\. 操作 AST 语法树, 可以更灵活的进行代码分析, 格式化的代码可以更好的为其他分析手段提供支撑, 比如机器学习分析 AST/CFG/IR

### 缺点

1.AST 处理的性能消耗较大  
2\. 目前的代码不能很好的跨文件处理, 仅限于单个文件, 虽然有办法可以二次解析  
3\. 目前没有覆盖所有的 Java Token, 存在遍历对象缺失的情况  
4.AST 所包含的信息维度不够, 编写代码难度不小, 也不够通用, 一个引擎只能分析一种语言  
5\. 市面上的这类工具已经不少了: `Fortify`,`CheckMarx` , `SonarQube` , `Codeql`, `Joern` 效果各有千秋, 但绝不是银弹

> 本文只是`Static Analysis`的一次浅显尝试, 虽说效果不错, 能看出来有很多地方写的很粗糙, 后面会使用更先进的技术改善这里的缺点.
> 
> `Static Analysis`不是银弹, 也有着自己的局限性, 也不能全指望着`Static Analysis`能够覆盖所有的漏洞点, 毕竟一个即`Sound`又`Complete`的分析是不存在的.

作者: 斗象能力中心 TCC – 小胖虎

> TCC Team 长期招聘，包含各细分领域安全研究员 \[Web / 网络攻防 / 逆向\]、机器学习、数据分析等职位。感兴趣不妨发简历联系我们。  
> Email: alex.xu@tophant.com。