> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bylibrary.cn](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/)

> 白阁文库是白泽 Sec 团队维护的一个漏洞 POC 和 EXP 披露以及漏洞复现的开源项目，欢迎各位白帽子访问白阁文库并提出宝贵建议。

[](https://github.com/BaizeSec/bylibrary/blob/main/docs/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E.md "编辑此页")

#### ActiveMQ 任意文件上传漏洞 [¶](#activemq "Permanent link")

该漏洞出现在 fileserver 应用中，ActiveMQ 中的 fileserver 服务允许用户通过 HTTP PUT 方法上传文件到指定目录。Fileserver 支持写入文件 (不解析 jsp), 但是支持移动文件(Move) 我们可以将 jsp 的文件 PUT 到 Fileserver 下, 然后再通过 Move 指令移动到可执行目录下访问

使用 vulhub 一键搭建，靶机 kali：192.168.1771.37

环境搭建成功，浏览器访问：[http://IP:8161](http://ip:8161/)

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/1.png)

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/2.png)

登录 admin 账号：默认账号 admin/admin，抓包进行修改，使用 PUT 方法上传文件。ActiveMQ Web 控制台分为三个应用程序：其中 admin，api 和 fileserver，其中 admin 是管理员页面，api 是界面，fileserver 是用于存储文件的界面；admin 和 api 需要先登录才能使用，fileserver 不需要登录。

上传 jsp 文件（系统不稳定，有时成功有时失败），

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/3.png)

```
docker-compose up -d
```

使用 MOVE 方法移动文件，成功的包没截上，再次上传是 500 回显，文件已经上传成功，访问地址，成功解析 jsp

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/4.png)

使用 MOVE 方法移动文件，成功的包没截上，再次上传是 500 回显，文件已经上传成功，访问地址，成功解析 jsp

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/5.png)

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/6.png)

漏洞影响版本：Apache ActiveMQ 5.x ~ 5.14.0

1、ActiveMQ Fileserver 的功能在 5.14.0 及其以后的版本中已被移除。建议用户升级至 5.14.0 及其以后版本。

2、通过移除 `conf\jetty.xml` 的以下配置来禁用 ActiveMQ Fileserver 功能

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/ActiveMQ/ActiveMQ%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/7.png)

* * *

最后更新: 2021-03-24