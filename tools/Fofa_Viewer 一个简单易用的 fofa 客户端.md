> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/n0Mg_TTog0SAu1WTYqKEGg)

Fofa_Viewer 🔗
==============

![](https://mmbiz.qpic.cn/mmbiz_svg/X4XEGYefSBTYjicxeu3gUgyTfRQhvTGmibAcVf9U8yw2YyWXzAvXfOohVruvT4VnAVvrJWJUTbgYIb6l2IYqbHPYhsnNgUpfCF/640?wx_fmt=svg) ![](https://mmbiz.qpic.cn/mmbiz_svg/X4XEGYefSBTYjicxeu3gUgyTfRQhvTGmibxjx5RI8D0auLFMBmsTWZsrEQKvyJQXRMbAMIAlz5icO4Df5TsqjTicqKOJSoTwctUu/640?wx_fmt=svg) ![](https://mmbiz.qpic.cn/mmbiz_svg/X4XEGYefSBTYjicxeu3gUgyTfRQhvTGmibEmatYFDYAFL8kxqAkpI4cM37ZCfce7OrJd2L6zZic5f5SBkd6iaIUcKKB27h9HFKHib/640?wx_fmt=svg) ![](https://mmbiz.qpic.cn/mmbiz_svg/X4XEGYefSBTYjicxeu3gUgyTfRQhvTGmibuWRlLZqOrSTpib7AHh6ewdCLK7zA9Q4JmTTicZSR8tLI6GVjQX4PlIia14icPs54qM4ic/640?wx_fmt=svg)

简介
--

Fofa_Viewer 一个简单易用的 fofa 客户端由 WgpSec 狼组安全团队 f1ashine 师傅主要编写 ，程序使用使用 javafx 编写，便于跨平台使用  

师傅们觉得好用请**点亮小心心**~![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBS2y6rUxP8WZ6icRpX3K443xYL2WSzdJIHbW6cG61BpfmoxBu4IKsago9tJtylibGqrP0HN8PwtNQQ/640?wx_fmt=png)

使用  

-----

下载最新版本包，修改`config.properties` 即可开始使用

MAC 用户可以参考 zhaodie 师傅的文章来配置快速启动

查询语法可参考 https://fofa.so/

若下载速度太慢可以使用

https://hub.fastgit.org/wgpsec/fofa_viewer (推荐)

https://gitee.com/wgpsec/fofa_viewer （镜像）

**下载 Release**！！！（有很多小伙伴都不知道怎么下载）

https://github.com/wgpsec/fofa_viewer/releases

**项目代码全部开源，使用 Github CI 自动化构建发布，放心使用**

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBS2y6rUxP8WZ6icRpX3K4438lNMiaWO9ia6moPxcl1sCcNa18CAYcia9icHic1FqJZnSbGhOPTXzacxskw/640?wx_fmt=png)

若双击无法打开且 error.log 无数据可尝试 java -jar 启动，有其他问题可加入交流群询问

💬 版本更新记录
---------

**Release 1.0.6**

🔨修复 bug：

1.  查询语句过长时弹窗 error 提示框，无法查询结果
    
2.  在查询一次结果后使用排除干扰查询时无结果回显
    
3.  设置 error.log 文件的自动创建
    

💡新增功能：

1.  添加查询历史记录
    
2.  滚动条滚动到底部时自动追加下一页数据
    

**Release 1.0.5**

🔥新功能：

1.  添加查询智能提示；
    
2.  添加 fofa 的蜜罐排除功能；
    
3.  添加查询框的删除按钮；
    
4.  添加 server 头显示
    

🍉优化：

1.  导出功能修改为导出 http 资产方便使用其他工具进行扫描；
    
2.  移除 fofa 查询结果中的重复项；
    
3.  优化显示界面的序号和端口号排序问题
    

.....

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBS2y6rUxP8WZ6icRpX3K443G9mjcqf9mr22Ria9iaEecWN1QibyjOsxnYayicp4ibsaFQpFgo8MjGlWowQ/640?wx_fmt=png) 功能
----------------------------------------------------------------------------------------------------------------------------------------------------

1.  多标签式查询结果展示
    
2.  丰富的右键菜单
    
3.  支持查询结果导出
    
4.  支持手动修改查询最大条数，方便非高级会员使用 (修改`config.properties`中的`maxSize`即可)
    
5.  支持证书转换 将证书序列填写入启动页框内可转换，再使用 `cert="计算出来的值"` 语法进行查询 [具体例子](https://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484874&idx=1&sn=8e7bb4b5d6862fd7dd078d3365086bfa&scene=21#wechat_redirect)
    
6.  支持输入智能提示
    
7.  支持 fofa 的一键排除干扰（蜜罐）功能。（注：需要高级会员才能使用，使用时会在 tab 页标记`(*)`）
    

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBS2y6rUxP8WZ6icRpX3K443AYk4AtxzWpCoxDfydCTL4UtRicvMWrXJIIhOlYTPwVERAbp3iaagFNRQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBS2y6rUxP8WZ6icRpX3K4434epu7Ym5RVEDtSj1dkUCb0yvMCkDEDuYsmb3lhE1XarcNDicaK5YIJQ/640?wx_fmt=png) 二次开发
--------------------------------------------------------------------------------------------------------------------------------------------------

```
git clone https://github.com/wgpsec/fofa_viewer.git
```

本项目使用 `maven-assembly-plugin`打包编译，可按照下图进行配置

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBS2y6rUxP8WZ6icRpX3K443xpLvzJ0y9M1dfGHkwddazkiaWWRaHOqUUXyP4ibuNJJKfSr1icHqXxWag/640?wx_fmt=png)

idea 打开项目，等待依赖包下载完毕后直接双击 Plugins-assembly-assembly:assembly，然后将 target 文件夹中带有 "with-dependencies" 的 jar 包拷贝到带有 config.propertiese 的文件夹再运行即可。

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBS2y6rUxP8WZ6icRpX3K443oJIENx6VvJuemtSHSmp0RzX1t0kL8oxQhfV48DVO2fI7MUDicPPtgSg/640?wx_fmt=png)

⚠️ 说明
-----

*   使用前需要在`config.properties`中配置`email`和`key`才能正常使用
    
*   项目中配置了 error.log，如果有需要提 bug，希望能带上这个截图，另外也欢迎提 issue 帮助改进。
    
*   关注公众号回复 “加群” 即可加入官方交流群
    

公众号