> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TEAyiPUdhSlpVT_djQmiCw)

**本文首发于先知社区，点击原文链接可查看原文**

##### **0x00：背景**

shiro 反序列化 RCE 是在实战中一个比较高频且舒适的漏洞，shiro 框架在 java web 登录认证中广泛应用，每一次目标较多的情况下几乎都可以遇见 shiro，而因其 payload 本身就是加密的，无攻击特征，所以几乎不会被 waf 检测和拦截。  

##### **0x01：shiro 反序列化的原理**

先来看看 shiro 在 1.2.4 版本反序列化的原理，这里是 shiro 拿到 cookie 后的关键代码，先 decrypt 再反序列化。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfjax2Pmx5NcFSh98O3BP4IdXAf1NnYP37CSySdgjBDEWsO00CuhsgIg/640?wx_fmt=png)

跟到 decrypt 方法

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfHg5pXufLVfSwxUicq3zH4r3iaEicEtHz54eHSIR2fNuNibLgOgMib5ibvibfA/640?wx_fmt=png)

调用具体的 cipherService，传入加密后的数据和 cipherKey 进行解密

getDeryptionCipherKey() 获取的值也就是这个 DEFALUT key，硬编码在程序中。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfqIRNUsfRCMv3qE5XnMAawof701jEYYia6cNyZpXr1qj7dic5W0xx1zGw/640?wx_fmt=png)

查看 CipherService 接口的继承关系，发现其仅有一个实现类 JcaCipherService（静态可以这样看，动态调试会直接跟进去）。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfMT05Rc045fCDcnJz6KQHpybfCQb6J8kjIXD0R33Dp67GoibB8hvOrsQ/640?wx_fmt=png)

查看实现类的 decrypt 方法，可以看到 iv 即 ciphertext 的前 16 个字节，因为 iv 由我们随机定义并附加在最后的 ciphertext 前面，也就是说只要知道 key 即可构造反序列化 payload。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxf8l7OUbeicOYJzjtdFhbfbcg6U94VP7RcmUFiaWuzTXiaCaRdENzjqbqQA/640?wx_fmt=png)

key 是硬编码，后续官方修改为获得随机 key，但正如一开始所说，存在开源框架配置硬编码 key，因此在 1.4.2 以前很多 shiro 都可以通过略微修改脚本遍历高频 key 的方式去攻击，下图举例，github 上有很多类似这样的代码。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfORZDS1VULxHZOvIFic5ibA1cQ7DJl0EKXHJlHgVmPVXcu7kcXSibJ8Wfg/640?wx_fmt=png)

CBC 算法的 shiro 生成 payload 的关键代码如下，也就是我们通用的生成 shiro 攻击代码，python 中有实现 aes-cbc 的算法，通过指定 mode 为 AES-CBC，遍历 key，随机生成 iv，配合 ysoserial 的 gadget 即可生成 payload。

| 

```
BS   = AES.block_size
    pad = lambda s: s + ((BS - len(s) % BS) * chr(BS - len(s) % BS)).encode()
    mode =  AES.MODE_CBC
    iv   =  uuid.uuid4().bytes
    file_body = pad(file_body)
    encryptor = AES.new(base64.b64decode(key), mode, iv)
    base64_ciphertext = base64.b64encode(iv + encryptor.encrypt(file_body))
    return base64_ciphertext

```

 |

##### **0x02：高版本下的利用**

而在 1.4.2 以后由于 padding oracle 的影响，shiro 官方把加密方式改为了 GCM，所以我们需要更改脚本，添加 GCM 下的攻击方式去攻击高版本的 shiro，通过跟踪代码动态调试可以看出确实是使用 GCM 加密。  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfia9lVFJIBPd70crYkAWD6xicBcgcxxkj9Owetv5GyEZlMUyQuzAt7YCg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfEjA7OLYmGWKZn0bvZibrsbpY3ZJApzvlJMS52do3ibqah4FKmm3ffldQ/640?wx_fmt=png)

所以 shiro 的攻击脚本中的核心代码我们来修改一下, GCM 加密不需要 padding，但需要一个 MAC 值（也就是我代码里的 tag），这块可以自己跟一下源码，核心代码如下：

| 

```
iv = os.urandom(16)
    cipher = AES.new(base64.b64decode(key), AES.MODE_GCM, iv)          
    ciphertext, tag = cipher.encrypt_and_digest(file_body) 
    ciphertext = ciphertext + tag   
    base64_ciphertext = base64.b64encode(iv + ciphertext)
    return base64_ciphertext

```

 |

##### **0x03：获得 key & 回显 & 内存 shell**

今年从三月到八月有很多师傅写了很多文章集中在 shiro 的利用上，我也综合了各位师傅的想法和思路搞了一些脚本。

**1.key 检测：**

以前的脚本批量检测 shiro 的存在或者说获得 key 我们用到最多的还是 ysoserial 中的 urldns 模块，不过这个有一个问题就是如果不出网的话会有问题，所以这里还有一个新的办法就是使用一个空的 SimPlePrincpalCollection 作为我们要序列化的对象，也就是构造一个正确的 rememberMe 触发反序列化，如果 key 是正确的，响应包中不会返回 rememberMe=deleteMe，这里一开始我想在脚本中通过检测返回包中是否有 deleteMe 关键字来做判断，结果发现有一些环境本身就会返回 rememberMe=deleteMe，因此最终选择了一个比较暴力的方式，关键代码如下  

| 

```
r1 = requests.get(target, cookies={'rememberMe': "123"}, timeout=10, proxies=PROXY,
                         verify=False, headers=myheader,allow_redirects=False)
    rsp1=len(str(r1.headers))
 #这里我先给一个肯定不正确的rememberMe   
    try:
        for key in keys:
            print("[-] start key: {0}".format(key))
            if ciphertype == 'CBC':
                payload = CBCCipher(key,base64.b64decode(checkdata))
            if ciphertype == 'GCM':
                payload = GCMCipher(key,base64.b64decode(checkdata))

            payload = payload.decode()
        #print(payload)
            r = requests.get(target, cookies={'rememberMe': payload}, timeout=10, proxies=PROXY,
                         verify=False, headers=myheader,allow_redirects=False)  # 发送验证请求
            rsp = len(str(r.headers))
            if rsp1 != rsp and r.status_code != 400:
            #在这里和肯定不正确的返回包header的lenth做比较，如果有差异，则是正确的的key
                print("!! Get key: {0}".format(key))
                exit()

```

 |

当然这样也有可能存在误报，大家可以酌情修改，用起来是这样。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxf55RricdbLQ4rEG1cibnpdLKlv2QoeOxRNDxo7TibzgIuxu2AvqXqOadKw/640?wx_fmt=png)  

**2. 回显:**

回显这个点也是有很多师傅发了很多文章，从一开始 linux 下的半回显到 kingkk 师傅的 tomcat 通用半回显再到 c0ny1 师傅的半自动化挖掘再到 fnmsd 师傅的通用回显，中间也穿插着其他师傅的文章这里就不列出来了，当然最终我用的也是 fnmsd 师傅的代码，一路跟了这些文章过来不得不感慨师傅们真是 tql 以及这种不断提出新思路不断完善的过程实在令本菜鸡拍断大腿。

fnmsd 师傅用的其实也是 c0ny1 师傅的思路，就是在当前线程对象里搜索 request 对象，判断请求头中是否存在指定请求头，再从 response 对象里获取输出。

fnmsd 师傅也对代码里存在的几个小问题做了好几个版本的修改，最终我在实际运用的时候依然存在一点点问题就是师傅设置的深度优先搜索是 52 层，结果测试时还是遇到了 52 层没有搜到的情况，结果随意改成 100 层就找到了.... 以及在用 ysoserial 里面有的 cc 链比较长生成出来的 payload 差不多逼近了 tomact 默认对 header 的限制长度，有点危险。nginx 就不说了直接凉，如果遇到 nginx 需要换个思路，比如可以尝试先分段注入，最后再执行。

代码就不贴了，列一下师傅的文章，他有给出代码

fnmsd 通用回显

fnmsd 修复通用回显

先跑一下测试能不能回显，效果如图：

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfaE8fqL81v0U9eJUWSpPCH5jic0lxQPyVnDpH9VHuUZ8pFnNrTpVRYAw/640?wx_fmt=png)

再把输出的 payload 粘贴到 burp 中执行命令：

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxflKrNuiaVMxG8sib4haj3E91e1zl95ANRtUqKMpz8dHS6gFiceHOO0Zwfg/640?wx_fmt=png)

**3. 内存 webshell**

其实严格来说在回显里提到的 kingkk 师傅做的是内存 webshell，一开始是有点混杂的，但其实跟一路会发现内存和 webshell 到最后利用的点是不一样的，内存 webshell 的思路是注册 filter（当然还有观星的师傅写的通杀 spring 的，思路是注册 controller，链接在此 基于内存 Webshell 的无文件攻击技术研究)，其实在有了 fnmsd 师傅思路比较好的回显之后，内存 webshell 可能需求就不太大了，但有时候红队可能需要一个内存 shell 来维持权限 (?) 或者需要配合 reGeorg 代理进内网，所以这个东西还是有必要的

一开始看到 threedr3am 师傅的基于 tomcat 的内存 Webshell 无文件攻击技术文章，解决了 shiro 下的利用，也能做到勉强通杀 tomcat（除了 tomcat6，原因可看评论区，这个我没测试），但是遇到 sihro 有一个最大的问题就是长度，没错，这个长度超大发了。菜鸡如我也上蹿下跳试图通过修改 ysoserial 和 payload 的代码缩短 payload，事实证明我确实不行，长亭师傅的解决办法看了一眼感觉不是特别好用，再加上后来又继续去跟了回显，就没管这个了。后来又看到了涙笑师傅的思路，解决了长度的问题。  

涙笑师傅的文章：

[Java 代码执行漏洞中类动态加载的应用](https://mp.weixin.qq.com/s?__biz=MzAwNzk0NTkxNw==&mid=2247484622&idx=1&sn=8ec625711dcf87f0b6abe67483f0534d&scene=21#wechat_redirect)

其实原理就是在 cookie 中反序列执行的代码只是插入了一个我们自定义的 ClassLoader，这块长度很小。在这个 ClassLoader 中反射调用 defineClass 方法，这个方法的 byte 参数从 POST 参数中取出来。也就是把我们注册 filter 的这段 payload 写在 static 代码块里，编译后的 byte 再通过 POST 传递，自定义 ClassLoader 加载时会自动执行 static 代码块的代码。

攻击的时候先生成 payload，上面是最终要执行的类，下面是 rememberMe 中要执行的插入 classloader 的 payload。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfpVQ07jhq5lZnGHYTyEG8T7N44bvsxGmDBz8oLiclLFiaTBKibymZaFrdw/640?wx_fmt=png)

在 burp 里是这样：

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfhVcPVLnicXubC5QdQuNXfXAo29NvTQjywKicLSdkbmKTc2G5pc0wPeFA/640?wx_fmt=png)

要注意 classData 一定要 url 编码，我刚开始测试没注意这点一直不成功。涙笑师傅的文中给出了配合 reGerog 的 payload，这里就不粘贴了。

**4. 内存 webshell 适配：**

后面还有一个问题就是内存 webshell 怎么适配冰蝎，因为冰蝎用的 pageContext 类在 springboot 里面是没有的，所以解决方案一个是反编译后修改冰蝎的代码重新打包，这一块虽然能做出来，但是不通用，而且比较麻烦，有师傅在先知发过文，可以参考冰蝎改造之适配基于 tomcat Filter 的无文件 webshell  

另外一个思路就是不改冰蝎的服务端，改内存 webshell，也看到有师傅发的文，思路是沿用涙笑师傅通过自定义 classloader，接着先注入一个 pageContext 类，再注入内存 webshell，链接在此 [冰蝎改造之不改动客户端 => 内存马，有兴趣的师傅可以去研究一下。](https://mp.weixin.qq.com/s?__biz=MzU2NTc2MjAyNg==&mid=2247484318&idx=1&sn=ece9e52218be0ea84ef166c3bfd20f23&scene=21#wechat_redirect)

然而不知道是我学艺不精还是师傅的文章写的有点模糊（划掉），我总感觉在类加载机制下，这样做好像有点不对劲的地方，而我也确实没搞出来，一直报错，希望这位师傅如果看到这篇文章可以联系一下弟弟，ddddhm，我找了半天也不知道怎么联系上您。

##### **0x04：后记**

上文涉及到的所有脚本和我修改后的 ysoserial 包将在 github 中放出，才疏学浅，也是第一次做 java 安全的研（zong）究（jie）和分享，其他的工作也很多导致 shiro 研究的断断续续，文章也写得断断续续，时间跨度拉的很大，长达好几个月，后面写的时候很多的东西都快忘记了，因此文章中可能存在错误和疏漏，希望师傅们不吝指出。

最后感谢上文中提到的每一位师傅和部分没有提到的师傅，感谢你们无私的分享，才会让菜鸡如我学习到这么多知识。