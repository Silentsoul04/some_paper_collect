> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Gud431quGtOHSoTMr9_KXQ)

目录：

        漏洞成因  

        漏洞复现

        场景提出及 EXP 编写

作者：水木逸轩 @深蓝攻防实验室

**01**

漏洞成因

该漏洞为 SSRF+XmlDecoder 组合形成的攻击链，首先看出现漏洞的 SSRF 漏洞的 jsp，文件路径：/sys/ui/extend/varkind/custom.jsp。

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAswuXjKlMHtnTGgkpd8EzYhJZg33znG3ZyC8ibibmS3TU769IPHXyKyHgA/640?wx_fmt=png)

c:import 为文件资源导入标签，根据标签解释从 HTTP 请求中获取 var 参数的值，然后解析 json 获取 body 的值，将 body 违 URL 进行资源导入 c:param 为参数传递标签，若 c:import 中 url 的属性为 http://www.baidu.com 然后通过 c:param 进行参数传递，若参数为 name=admin，则最后发送的请求为 http://www.baidu.com?name=admin 以上为 SSRF 漏洞的分析。

接着对 xmlDecoder 漏洞的路由进行分析：

在 struts.xml 文件中找到操作映射，寻找 com.landray.kmss.sys.search.actions.SysSearchMainAction 类

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsjwJdlibowicPIle9dbrdJVaUR2USA1fUGx0sEUzvBl0vIEJJ26DpdEDg/640?wx_fmt=png)

首先看 ActionForm 参数，ActionForm 为从 HTTP 请求中获取的参数，首先判断是否能获得参数中 fdParemNames 的值

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsBh5Ang4bP4Sc18InLuITc1IXcMQy2YYQDURYklWmu9oR1rvOIjjBAA/640?wx_fmt=png)

如果获取不到，调用 setParametersToSearchConditionInfo 方法

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAs6yLJibGoU4eG7qHqAIvU0dGxv4z1ldCS0OEJm4t26TiaNlYqkg7Osyzw/640?wx_fmt=png)

这里触发 xmlDecoder 反序列化漏洞

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsNWILgjzA8LzZSKNpFOQdkBtqbUmsy8Y5QIvtODbkWaQDelf2ONwn1w/640?wx_fmt=png)

**02**

 漏洞复现

访问蓝凌目标：

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsIhYzQyj7xpH7S1bSGlU9EOd7WLMDQceZl6TUT3Irx4o33Libibr6rvVg/640?wx_fmt=png)

POC 如下，使用 bsh.Interpreter 调用 java 函数，执行 Java 代码：

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAskkReRVeffjl8W2jvz4CHP7N0dP1ndgdVtKrhFOC8D6Av88icNwGia7Hw/640?wx_fmt=png)

登录目标远程机器，弹个计算器：

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsO29PrNk3mYickq6KxSbXbfLtRauOXJBQ9gkbVKz9vfzszEj9h9ZmB6w/640?wx_fmt=png)

**03**

场景提出及 EXP 编写

因为可能存在写入 jsp 文件被文件落地设备检测的危险，所以直接写入一个 war 包，而且 jsp 和 java 文件不落地，只落地. war，.xml，.class 文件，所以尝试直接注册 filter 写入 cmd 马。

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsXElr7FhvdLrMmWh6GfO7vH9UrxHwkZlp8ehbJGAicCV72RUU0DlJHeQ/640?wx_fmt=png)

配置好 web.xml 文件，使用 IDEA 导出 war 包

此处注意，login 类需要在 jdk7 下面编译。

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsibRBjIFcQFJQLAia8oUwcVic9NfYpPxqyWfp6uaRibSLlQl4rRRXcVKcdw/640?wx_fmt=png)

使用 Java 代码读取 war 包的 byte 格式，使用文件写入函数，写入到目标环境的 tomcat webapps 目录。

最后改变 POC，写入 war 文件

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsp7EQxUBCc1d5WfBPOM7Q55oChpFVT0vf61oibywPXLzgEfZf1dq7kcg/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAsBRPmumcmWzX82KDZ4B6vlBRqNRUlu9zcDic3xqtJx99yxZsyMnX6nDw/640?wx_fmt=png)

尝试命令执行：

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2fcxIjboInwZrzanG2icoyAslOFL9Q5Znq9uQImWsU2d5y0sefib4MicibkOjrywKNrGibNbCd0S6SKVsg/640?wx_fmt=png)

执行成功，编写漏洞利用脚本

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2ead0u4lVLGAjV59OJcFCArrswoQCkbbMeH27HMPpbJvWrSkuBreUDF15RanewaicqCF4R6iaUVX12g/640?wx_fmt=png)

测试脚本是否成功

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2ead0u4lVLGAjV59OJcFCArz3WAibVXlyoticNtvuJE13alVWQdch2CVOBYDWzgrMGDd3yknyVCxU4Q/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2ead0u4lVLGAjV59OJcFCArAH3Jv1k0iaNdSQ2vX1LD9eJvqAaoic2yOupgOWFGN7p4P6Cr1DooNbFQ/640?wx_fmt=png)

如果想跟我一起讨论，那快加入我的知识星球吧！https://t.zsxq.com/7MnIAM7

![图片](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2eKvY8jwoT7yxMvHfscqNQUJ2ed5fxYvws9QrsiaaXtMqRxaiaWFryhXYVpiaDxVUPA2vBQvj0G0uKicQ/640?wx_fmt=png)