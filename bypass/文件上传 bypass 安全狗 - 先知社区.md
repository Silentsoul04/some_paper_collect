> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9507)

*   0x00 前言

我们知道 WAF 分为软 WAF，如某狗，某盾等等；云 WAF，如阿里云 CDN，百度云 CDN 等等；硬 WAF，如天融信，安恒等等，无论是软 WAF、云 WAF 还是硬 WAF，总体上绕过的思路都是让 WAF 无法获取到文件名或者其他方式无法判断我们上传的木马（PHP、JSP、ASP、ASPX 等等）。

这里总结下关于软 waf 中那些绕过文件上传的姿势和尝试思路，这里选择绕过的软 waf 为某狗 4.0，可能其他软 waf 在拦截关键字方面可能会有差异，但绕过软 waf 的大体思想都是相同的，如果文章中有错误，欢迎师傅们斧正。

*   0x01 初探原理

写这篇文章时想过一个问题，如何总结哪些属于文件上传 Bypass 的范畴？打个比方：

```
上传正常.jpg的图片 #成功
上传正常.php #拦截
绕过.php文件的filename后进行上传 #成功
使用绕过了filename的姿势上传恶意.php #拦截
以上这么个逻辑通常来讲是waf检测到了正文的恶意内容。再继续写的话就属于免杀的范畴了，过于模糊并且跑题了，并不是真正意义上的文件上传Bypass，那是写不完的。
```

上传文件时 waf 会检查哪里？

```
请求的url
Boundary边界
MIME类型
文件扩展名
文件内容
```

常见扩展名黑名单：

```
asp|asa|cer|cdx|aspx|ashx|ascx|asax
php|php2|php3|php4|php5|asis|htaccess
htm|html|shtml|pwml|phtml|phtm|js|jsp
vbs|asis|sh|reg|cgi|exe|dll|com|bat|pl|cfc|cfm|ini
```

测试时的准备工作：

```
什么语言？什么容器？什么系统？都什么版本？
上传文件都可以上传什么格式的文件？还是允许上传任意类型？
上传的文件会不会被重命名或者二次渲染？
```

*   0x02 环境介绍

实验环境：mysql + apache +php

waf：某狗 4.0

这里写了一个简单的上传页面判断，观察代码可以发现只允许上传 Content-Type 为 image/gif、image/jpeg、image/pjpeg 三种形式的文件

```
<html>
<body>

<form action="upload.php" method="post"
enctype="multipart/form-data">
<label for="file">Filename:</label>
<input type="file"  /> 
<br />
<input type="submit"  />
</form>

</body>
</html>
<?php
error_reporting(0);
if ((($_FILES["file"]["type"] == "image/gif")
|| ($_FILES["file"]["type"] == "image/jpeg")
|| ($_FILES["file"]["type"] == "image/pjpeg")))
//&& ($_FILES["file"]["size"] < 20000))
  {
  if ($_FILES["file"]["error"] > 0)
    {
    echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
    }
  else
    {
    echo "Upload: " . $_FILES["file"]["name"] . "<br />";
    echo "Type: " . $_FILES["file"]["type"] . "<br />";
    echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
    echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

    if (file_exists("upload/" . $_FILES["file"]["name"]))
      {
      echo $_FILES["file"]["name"] . " already exists. ";
      }
    else
      {
      move_uploaded_file($_FILES["file"]["tmp_name"],
      "upload/" . $_FILES["file"]["name"]);
      echo "Stored in: " . "upload/" . $_FILES["file"]["name"];
      }
    }
  }
else
  {
  echo "Invalid file";
  }
?>


```

*   0x03 实验 bypass

先上传一个 asp，看一下返回值

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100422-6055d2ea-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100422-6055d2ea-ae0f-1.png)

这里看到了 404xxxdog 的页面，那应该是拦截了，我这里先放过去看看，果然是某狗拦截了

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100423-609f1e78-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100423-609f1e78-ae0f-1.png)

开始尝试绕 waf

这里我先把 Content-Type 改成 image/gif 通用图片类型

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100423-60d219e0-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100423-60d219e0-ae0f-1.png)

0x03.1 增大文件大小

测试发现 waf 对于 Content-Disposition 字段的 长度验证不是很准确，因为我们可以想到它进行拦截的规则肯定是基于正则，那么我们想办法让安全狗拦截的正则匹配不到即可

这里附一个对 Content-Disposition 字段的解释

```
在常规的 HTTP 应答中，Content-Disposition 响应头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。

在 multipart/form-data 类型的应答消息体中，Content-Disposition 消息头可以被用在 multipart 消息体的子部分中，用来给出其对应字段的相关信息。各个子部分由在Content-Type 中定义的分隔符分隔。用在消息体自身则无实际意义。

Content-Disposition 消息头最初是在 MIME 标准中定义的，HTTP 表单及 POST 请求只用到了其所有参数的一个子集。只有 form-data 以及可选的 name 和 filename 三个参数可以应用在HTTP场景中。

```

这里对这个字段的长度进行篡改，绕过成功

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-61089696-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-61089696-ae0f-1.png)

0x03.2 对文件名修改（卒）

我们在上传时候通常会把文件名后缀和解析漏洞，那么 waf 对于 filename 参数后的值的文件名后缀肯定是要正则去匹配的 这样正常上传肯定不行

那么绕过之前我们猜想，第一个它可能是对 filename 这样的键值对进行匹配，例如 "ket = val" 这样的键值对，那么这里我们就是 filename=“shell.php”

那这里把双引号去除，扰乱匹配，发现不行

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-614af9dc-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-614af9dc-ae0f-1.png)

那么我们可不可以多一个 filename，因为文件在接收上传文件名时取的是最后一个 filename，那么我们在最后一个 filename 参数前加一些干扰的 filename 参数试试

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-61757018-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-61757018-ae0f-1.png)

发现还是不行 那么这里就知道他是对所有 filename 参数进行检测 那么我们能不能把前面的 filename 参数去掉值呢

```
Content-Disposition: form-data; 

```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-61945aa0-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100424-61945aa0-ae0f-1.png)

结果对文件名进行修改全卒，在之前版本的某狗在 filename= ; 是可以进行绕过的，4.0 版本文件名修改全卒

0x03.3 修改文件名后缀

经典的 apache 解析漏洞尝试，拦截

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-61aeced0-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-61aeced0-ae0f-1.png)

可以在文件名中间加符号扰乱某狗匹配，经测试 ";" """ ' " 均可

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-61cf5ef2-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-61cf5ef2-ae0f-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-61ec1b64-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-61ec1b64-ae0f-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-622ff410-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100425-622ff410-ae0f-1.png)

0x03.4 对 filename 动手脚

这里可以让 waf 对 filename 这个字符串匹配不到，但是服务器又可以接收，加入换行这类的干扰

先测试单个字符进行换行，都失败

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-624d8df4-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-624d8df4-ae0f-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-627bb5e4-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-627bb5e4-ae0f-1.png)

切断 filename= 和 之后的值，则可以绕过

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-62a786d8-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-62a786d8-ae0f-1.png)

文件名换行，即 hex 加入 0a，也可以绕过

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-62c696ae-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100426-62c696ae-ae0f-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-62e44be0-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-62e44be0-ae0f-1.png)

0x03.5 修改匹配字段（卒）

我们的 filename 参数是在 post 包中的 Content-Disposition 字段，那么 waf 也是先匹配到这个 http 头在对内容进行检测，我们可以尝试对这个头的特征进行修改

我们尝试去掉这个 form-data (form-data; 的意思是内容描述，form-data 的意思是来自表单的数据，但是即使不写 form-data，apache 也接受。)

```
Content-Disposition: 

```

发现失败，之前 3.0 版本可以绕，4.0 卒

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-62ff3fb8-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-62ff3fb8-ae0f-1.png)

对 Content-Disposition 进行参数污染，拦截

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-631996ba-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-631996ba-ae0f-1.png)

对 Content-Disposition 进行大小写混淆，拦截

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-633a694e-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100427-633a694e-ae0f-1.png)

加上额外的 Content-Type 进行干扰，拦截

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-6369a1fa-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-6369a1fa-ae0f-1.png)

加上 filename 进行参数污染，拦截

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-6399fbc0-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-6399fbc0-ae0f-1.png)

加一个额外的 Content-Length 头，拦截

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-63ba4e2a-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-63ba4e2a-ae0f-1.png)

0x03.6 多个等号

经测试两个 = 或者三个 = 都可以达到绕过的效果

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-63e5ccb2-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100428-63e5ccb2-ae0f-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100429-64015fd6-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100429-64015fd6-ae0f-1.png)

0x03.7 %00 截断

%00 截断产生的原因是 0x00 为十六进制的表示方法，ASCII 码里就为 0，而有些函数在进行处理的时候会把这个当作结束符

这里直接尝试在文件名后面加上 %00 形成 00 截断，成功绕过

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210506100429-641e452e-ae0f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210506100429-641e452e-ae0f-1.png)

*   0x04 后记

某狗 4.0 版本对于 3.0 版本又有了一定的改进，对于之前的文件名修改和修改匹配字段已经不能够绕过 waf，但是对于绕软 waf 的思想总结起来可以有如下几点：

大小写转换、干扰字符污染、字符编码、拼凑法、各种换行符

总体来说，这些思想不仅在文件上传 bypass 中有用，对于 sql 注入绕 waf 等一系列地方思想都大体相同，师傅们可自行拓展，文章中若有错或不足的地方欢迎师傅们交流。