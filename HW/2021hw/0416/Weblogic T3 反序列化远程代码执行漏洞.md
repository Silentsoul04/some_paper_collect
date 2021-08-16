# Weblogic T3 反序列化远程代码执行漏洞
漏洞概述
----

利用T3协议进行反序列化漏洞实现远程代码执行。

漏洞分析
----

[某weblogic的T3反序列化0day分析](../0418/%E6%9F%90weblogic%E7%9A%84T3%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%960day%E5%88%86%E6%9E%90.md)

修复方案 
-----

关闭T3协议或限制访问地址。 1、检查是否开启T3协议：查找应用服务器路径是否存在/bea\_wls\_deployment\_internal/loginUser.do （返回状态码是200为已开启） 2、关闭和限制方式：在连接筛选器中输入：weblogic.security.net.ConnectionFilterImpl，在连接筛选器规则中 输入： ip \* \* allow t3（允许通过T3协议访问的地址） 0.0.0.0/0 \* deny t3 t3s（限制所有地址通过T3访问） 示例： 192.168.0.1\* \* allow t3（白名单IP） 192.168.0.2 \* \* allow t3（白名单的IP） 127.0.0.1 \* \* allow t3 0.0.0.0/0 \* \* deny t3 t3s

![](Weblogic%20T3%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E8%BF%9C%E7%A8%8B%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E/11d222cdac967fc7.jpg)

![](Weblogic%20T3%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E8%BF%9C%E7%A8%8B%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E/2d19909d7308f1bf.png)