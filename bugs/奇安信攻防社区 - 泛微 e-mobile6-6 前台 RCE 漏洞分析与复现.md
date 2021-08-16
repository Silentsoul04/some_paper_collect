> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [forum.butian.net](https://forum.butian.net/share/84)

0x01 漏洞分析
---------

### 0x0101 漏洞触发点 - 1

根据外部公开的漏洞信息得到漏洞触发点为`client.do`，找到漏洞位置  
`WEB-INF/classes/weaver/mobile/core/web/ClientAction.class`

![](https://shs3.b.qianxin.com/butian_public/ff741098c0b160421594242cb71a8d0d7.jpg)  
可以看到获取一个`method`参数做路由  
![](https://shs3.b.qianxin.com/butian_public/fc883eab44dd017c0e3f8a049922be0ca.jpg)  
跟踪`getupload`方法  
![](https://shs3.b.qianxin.com/butian_public/fe87e17275f321ad5dd95f0c38b18e4a1.jpg)  
![](https://shs3.b.qianxin.com/butian_public/f6da1c7458501c24b74c4c199e3e8ef65.jpg)  
![](https://shs3.b.qianxin.com/butian_public/ff45f1eeaa7fe861c97822eb1692d05fd.jpg)  
可以看到直接拼接了 sql 语句  
![](https://shs3.b.qianxin.com/butian_public/fc9aaf56a0f4ad7d53f98bff736091438.jpg)  
**因为泛微 emobile 是采用 hsql 数据库，所以有注入就可以直接执行 java 代码。**

### 0x0102 漏洞触发点 - 2

根据漏洞信息漏洞触发点：`messageType.do`找到漏洞位置

`WEB-INF/classes/weaver/mobile/core/web/MessageTypeAction.class`  
![](https://shs3.b.qianxin.com/butian_public/f5f4d69c62d59c09ef416c95bc19b2584.jpg)  
和上面一样 method 参数做路由`typeid`这里限制了为 int 类型跳过。  
![](https://shs3.b.qianxin.com/butian_public/f37490ed72ba0084b4623e3025ef466ff.jpg)  
当 method 参数等于`create`时`typeName`参数存在注入  
![](https://shs3.b.qianxin.com/butian_public/f37c07afe23be60a2bdc04e8efd5d3498.jpg)

0x02 漏洞复现
---------

### 0x0201 漏洞触发点 - 1 复现

![](https://shs3.b.qianxin.com/butian_public/f6f9ea88ba7b835840a127f323548e136.jpg)

### 0x0202 漏洞触发点 - 2 复现

![](https://shs3.b.qianxin.com/butian_public/fb6fc498014efa3a93fe349b3c0fcbf78.jpg)