> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LO9UWiOcV-jC2ReH3ufghg)

集成 POC 如下
---------

任意用户登录 POC: 4 个

SQL 注入 POC: 2 个

后台文件上传 POC: 3 个

本地文件包含 POC: 2 个

前台文件上传 POC(非 WEB 目录): 1 个

任意文件删除 POC: 1 个

工具面板截图
------

![](https://mmbiz.qpic.cn/mmbiz_png/TezRTl7qZQRtRX5aPCPz68mWmibV16o5ERfSSeibWyeRktjv1Ltukut8UL57wredNHghI9ibbemv6n4SZLAe5XXRg/640?wx_fmt=png)

工具利用流程
------

### 1. 优先利用本地文件包含漏洞

原因是本地文件包含漏洞, 配合前台文件上传可以直接 getshell, 无需获取有效 Cookie

### 2. 若本地文件包含漏洞利用失败, 其次利用任意用户登录漏洞与 SQL 注入漏洞

这两个漏洞的利用方式集成在了 "获取 Cookie" 按钮上

共计 6 个 POC, 其中任意一个 POC 利用成功

都会自动停止, 并自动填充有效的 Cookie 到工具上

获取有效 Cookie 后, 即可选择后台文件上传一键利用

**如目标存在弱口令, 可手动填写有效 Cookie 后配合****文件上传一键利用**

### 3. 特定版本 v11.6 存在任意文件删除漏洞的利用

当目标为 v11.6 版本时, 一键利用即可 (该漏洞利用存在一些风险).

代码实现流程如下:

1) 删除 auth.inc.php 文件

2) 上传 webshell

3) 上传 auth.inc.php 源文件

4) 上传处理文件 (移动 auth.inc.php 到原本位置, 删除自身)

5) 检测 auth.inc.php 源文件是否恢复

项目地址
----

https://github.com/xinyu2428/TDOA_RCE/releases

