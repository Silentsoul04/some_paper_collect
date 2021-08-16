> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247499508&idx=1&sn=cd248421577208748e4125b5ad15574f&chksm=ebeab7d9dc9d3ecff7d8fef4f41ec02c0300c2530b8f84f2484d239ef9fb8e60cfae641beaa3&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaacU9xqCuIQia8mZiaTMod2IN9AChx36cA68kz5OEIiafWY2ntLy087rDibJJtdhx6ewb2ico3hc2hl3HA/640?wx_fmt=png)

扫码领资料

获黑客教教程

免费 & 进群

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGJojZ4G6V9tecYcLCYmfcP4icJTZz3icxs8Roc0tiaazUmuU6icTHic4ntfOv0VLg5Xr7SBAHk0vibl6tg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaacU9xqCuIQia8mZiaTMod2INniajhTc48WZfuY7PCw9UA4DKrNZOicLsdP89LD2hpwGoI3umV1WxrkNw/640?wx_fmt=png)

#### 版本说明：`Burp Suite2.1`

#### 下载地址：

#### `链接：https://pan.baidu.com/s/1JPV8rRjzxCL-4ubj2HVsug`

#### `提取码：zkaq`

#### 使用环境：jre1.8 以上

`下载链接：https://pan.baidu.com/s/1wbS31_H0muBBCtOjkCm-Pg`

`提取码：zkaq`

#### 工具说明：`Burp Suite 是用于攻击web 应用程序的集成平台`

引言
--

在全球最受安全人员欢迎的工具榜单中，Burp Suite 这个工具排名第一，并且排名第二的工具和它的功能和作用是一样的，并且还是免费的。

burp 这个工具还是付费的，还能一直保持第一。

可见 burp 这个工具在我们安全圈他的地位。

  
Burp Suite 是一款信息安全从业人员必备的集成型的渗透测试工具，它采用自动测试和半自动测试的方式，

包含了 Proxy,Spider,Scanner,Intruder,Repeater,Sequencer,Decoder,Comparer 等工具模块。

通过拦截 HTTP/HTTPS 的 web 数据包，充当浏览器和相关应用程序的中间人，进行拦截、修改、重放数据包进行测试。

**所以本文会对 burp 的一些安装以及使用，进行一个系统的介绍。**

burp 的安装以及其配置
-------------

Burp Suite 是由 Java 语言编写而成，而 Java 自身的跨平台性，使得软件的学习和使用更加方 便。

Burp Suite 不像其他的自动化测试工具，它需要你手工的去配置一些参数，触发一些自动 化流程，然后它才会开始工作。

  
Burp Suite 可执行程序是 java 文件类型的 jar 文件。

因为 burp 这个工具是付费的。所以有很多大佬弄了这个工具的破解版，基本功能都是具备的，知识许多高级工具会受限制，无法使用。

  
在 burp 工具下载安装之前，要确保自己的电脑本地安装了 java 环境。

网站上面有很多教程，我们直接按步骤安装就可以了。

安装完 java 环境之后，打开 cmd 命令行，如果呈现出来下图所示的效果，就证明 java 环境已经安装好了。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eAcm68edVLCwSSpflDauAnxeVrHjPPx6W8sqhriaHKXKDwhqXtpqBMRw/640?wx_fmt=png)

*   Burp Suite 是一个无需安装软件，下载完成后，直接从命令行启用即可。
    
    这时，你只要在 cmd 里执行 java -jar / 你 burp 工具的路径 / burpSuite 名字. jar 即可启动 Burp Suite, 或者，在工具所在路径下面打开 cmd 命令行，输入 Java -jar burpSuite 名字. jar
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4edWNPE5TvOu2wVIqUwTBeC3sdiaTwoSwctTbbAjds7iaISRzCrQhDua7Q/640?wx_fmt=png)

burp 代理和浏览器设置
-------------

Burp Suite 代理工具是以拦截代理的方式，拦截所有通过代理的网络流量，如客户端的请求数据、服务器端的返回信息等。

Burp Suite 主要拦截 http 和 https 协议的流量，通过拦截，Burp Suite 以中间人的方式，可以对客户端请求数据、服务端返回做各种处理，以达到安全评估测 试的目的。

在日常工作中，我们最常用的 web 客户端就是的 web 浏览器，我们可以通过代理的设置，做到 对 web 浏览器的流量拦截，并对经过 Burp Suite 代理的流量数据进行处理。

当 Burp Suite 启动之后，默认拦截的代理地址和端口是 127.0.0.1 ：8080, 我们可以从 Burp Suite 的 proxy 选项卡的 options 上查看。如图：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eIHrIx6G3uUvFDGVdjy7CwnEjcgMqwt6NGibRpI2AtCxQXp4LvKu9C6A/640?wx_fmt=png)

  
FireFox 设置

*   系统代理
    

1.  **启动 FireFox 浏览器，点击右上角三条横线，点击【选项】。**
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eUXneAANMnuhsNFLGSZ129gZUrFaaxEsdJkSXNTTChuHgeplEtKrqNA/640?wx_fmt=png)

  
**2. 在出现的搜索框里输入代理，点击【设置】。**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eDQ5u8LN1ribWsJzal8icqiacgyPek0S2bZ3z6Yo2mgDmbcb5WeHxA4VNQ/640?wx_fmt=png)

**3. 勾选手动代理配置，找到 “http 代理”，填写 127.0.0.1，端口 填写 8080，最后点击【确认】保存参数设置，完成 FireFox 的代理配置。**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eDhopg2kianjeDfOl26xBQFSf1NVr5xEcglpLiaplg8PyHGibv6gHozAicw/640?wx_fmt=png)

*   扩展插件  
    FireFox 浏览器中，可以添加 FireFox 的扩展组件，对代理服务器进行管理。例如 FoxyProxy、FireX Proxy、Proxy Swither 都是很好用的组件，这里对 FoxyProxy 进行一个讲解。
    
      
    **1. 启动 FireFox 浏览器，点击右上角三条横线，点击【附加组件】。**
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4epicKcHgM0ZuTcpMQM1GwAHNXW750PrnVPsLqW3fBiaEgATbcicjaAgSzA/640?wx_fmt=png)  

**2. 在出现的搜索框里输入 proxy。**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eaenGkcia7VJxeiaibtZhWXTCiceU3nbwRw1c6q8wGWFyBTeLFVVy8pNGlg/640?wx_fmt=png)

1.  **呈现出的所有插件都是和代理相关的，选择你要安装的插件就可以了。**
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eQPSyYicMNdic3TeyTfRqyzLwSIRn2j6H0yrBYmvm417s2ib0zRIDIt8Qg/640?wx_fmt=png)

  
**4. 安装成功之后，会在浏览器右上角出现对应插件的标志。**

**点击插件，点击【选项】。**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eOKUZGv0LL5AB4VgxBjzAh8uIclicdcojaA4pwxeC3f6lr4UkP4fjriag/640?wx_fmt=png)  

**5. 对应把我们拦截的 IP, 拦截的端口添加上去。**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4e2L3WpUpf803iaF7GmB9cRCBBMibbTUEnEfFAsUYNpYVfbmQXUO78oOew/640?wx_fmt=png)  

**6. 使用哪一个代理的时候，就勾选对应的代理名称即可。**

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eyk4UaPwHbkU3aIQUvk3QriaubHtwpab6fYkfCBic9cxMMqicjEOiaZCRBw/640?wx_fmt=png)

如何使用 burp 代理
------------

Burp Proxy 是 Burp Suite 以用户驱动测试流程功能的核心，通过代理模式，可以让我们拦截、 查看、修改所有在客户端和服务端之间传输的数据包。

**使用 burp 的流程如下：**

1.  首先，再确认 burp 工具安装成功，能正常运行之后，并且已经完成浏览器的代理 服务器配置。
    
2.  打开 Proxy 功能中的 Intercept 选项卡，确认拦截功能也就是能够抓数据包为 “Interception is on” 状态，如果显示 为 “Intercept is off” 则点击它，打开拦截功能。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4e7twuLibbmSu1l07KDdBAmojUFH1TiayysSu3QJSex0cYtJnMD43pl3Hg/640?wx_fmt=png)

  
3. 打开浏览器，输入你需要访问的 URL  
（以靶场地址 http://59.63.200.79:8004/shownews.asp?id=171 为例）并回车，这时你 将会看到数据流量经过 Burp Proxy 并暂停，直到你点击【Forward】，才会继续传输下去。

如果你点击了【Drop】，则这次通过的数据将会被丢失，不再继续处理。

  
4. 当我们点击【Forward】之后，我们将看到这次请求返回的所有数据。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eAYIgSibBK5MV7L1RUsN2cdYzzXASgdsiawicjyRrqT4lWjAwIE4DuusAw/640?wx_fmt=png)

  
当 Burp Suite 拦截的客户端和服务器交互之后，我们可以在 Burp Suite 的消息分析选项卡中查看这次请求的实体内容、消息头、请求参数等信息。

消息分析选项视图主要包括以下四项：

*   Raw：web 请求的 raw 格式，包含请求地址、http 协议版本、主机头、浏 览器信息、Accept 可接受的内容类型、字符集、编码方式、cookie 等。我们可以手工去修改这些信息，对服务器端进行渗透测试。
    
*   params：客户端请求的参数信息、包括 GET 或者 POST 请求的参数、 Cookie 参数。渗透人员可以通过修改这些请求参数来完成对服务器端的渗透测试。
    
*   headers：与 Raw 显示的信息类似，只是在这里面展示得更直观。
    
*   Hex：这个视图显示的是 Raw 的二进制内容，渗透测试人员可以通过 hex 编辑器对请求的内容进行修改。
    

一般情况下，**Burp Proxy 只拦截请求的消息**，普通文件请求如 css、js、图片是不会被拦截 的，你可以**修改默认的拦截选项**来拦截这些静态文件

当然，你也可以通过修改拦截的作用域，参数或者服务器端返回的关键字来控制 Burp Proxy 的消息拦截。

所有流经 Burp Proxy 的消息，都会在 http history 记录下来，我们可以通过历史选项卡，查看传输的数据内容，对交互的数据进行测试和验证。

同时，对于**拦截到的消息和历史消息，都可以通过右****击弹出菜单**，发送到 Burp 的其他组件，如 Spider、Scanner、 Repeater、Intruder、Sequencer、Decoder、Comparer、Extender，进行进一步的测试。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4ekwuib2e9Vwt2naHG3kKtsJNy0w9vCiapeb9lIVjfzic22LBrErWHzE12w/640?wx_fmt=png)

对数据包的操作
-------

Burp Proxy 的拦截功能主要由 Intercept 选项卡中的 Forward、Drop、Interception is on/off、 Action、Comment 以及 Highlight 构成，**它们的功能分别是：**

*   Forward 的功能：当你查看过消 息或者重新编辑过消息之后，点击此按钮，将发送消息至服务器端。
    
*   Drop 的功能：你想丢失 当前拦截的消息，不再 forward 到服务器端。
    
*   Interception is on：表示拦截功能打开，拦截所有 通过 Burp Proxy 的请求据。
    
*   Interception is off 表示拦截功能关闭，不再拦截通过 Burp Proxy 的所有请求数据。
    
*   Action 的功能：除了将当前请求的消息传递到 Spider、Scanner、 Repeater、Intruder、Sequencer、Decoder、Comparer 组件外，还可以做一些请求消息的修改，如改变 GET 或者 POST 请求方式、改变请求 body 的编码，同时也可以改变请求消息的拦截设置，如不再拦截此主机的数据包、不再拦截此 IP 地址的消息、不再拦截此种文件类型的消息、不再拦截此目录的消息，也可以指定针对此消息拦截它的服务器端返回数据。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4epojGVWq32j7rDRAP1SRoUXLFHxkGkNOQX8qpHBr8S9zTn8icWxpsCLQ/640?wx_fmt=png)  
Comment 的功能：对拦截的消息添加备注，在一次渗透测试中，你通常会遇到一连串的请求消息，为了便于区分，在某个关键的请求消息上，你可以添加备注信息。

  
Highlight 的功能：与 Comment 功能有点类似，即对当前拦截的消息设置高亮，以便于其他的请求消息相区分。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4epojGVWq32j7rDRAP1SRoUXLFHxkGkNOQX8qpHBr8S9zTn8icWxpsCLQ/640?wx_fmt=png)

SSL 和 Proxy 高级选项
----------------

在前面一章的基础上，我们已经仅仅能够抓 HTTP 的数据包。

接下来我们继续学习如何抓 https 的包。

HTTPS 协议是为了数据传输安全的需要，在 HTTP 原有的基础上，加入了安全套接字层 SSL 协议，通过 CA 证书来验证服务器的身份，并对通信消息进行加密。

基于 HTTPS 协议这些特性， 我们在使用 Burp Proxy 代理时，需要增加更多的设置，才能拦截 HTTPS 的数据包。

### CA 证书的安装

我们都知道，在 HTTPS 通信过程中，一个很重要的介质是 CA 证书。

**一般来说，Burp Proxy 代理过程中的 CA 主要分为如下几个步骤（以火狐浏览器为例子）**  

1.  首先，根据前面学习的内容，我们已经已配置好 Burp Proxy 监听端口和浏览器代理服务器设置。
    
2.  在 burp 打开的情况下，在浏览器 URL 栏的地方输入，你所拦截的 IP 和你所拦截的端口。127.0.0.1:8080。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eOMaqkooQ0vYe0S4IdE8gHw2h730TibTRohXSiceqpDYhGC0epn9kdo8A/640?wx_fmt=png)  
3. 点击 CA 证书，点击【保存文件】。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4exWNY22bEON5Gr4zKCbzsDUlUNlzBoVQoLHVedsLbOFGKc7rUY77Oww/640?wx_fmt=png)  
CA 证书安装成功之后，就可以抓 https 的包了。

### Proxy 监听设置

当我们启动 Burp Suite 时，默认会监听本地回路地址的 8080 端口，除此之外，我们也可以在默 认监听的基础上，根据我们自己的需求，对监听端口和地址等参数进行自由设置。

特别是当 我们测试非浏览器应用时，无法使用浏览器代理的方式去拦截客户端与服务器端通信的数据 流量，这种情况下，我们会使用自己的 Proxy 监听设置，而不会使用默认设置。

*   Proxy 监听设置
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eOk0xjyCZKYfrPLUBQJEh76umgRjGZ2URHzMc5F1FaSUriaLXPichJeug/640?wx_fmt=png)当我们在实际使用中，可能需要同时测试不同的应用程序时，我们可以通过设置不同的代理 端口，来区分不同的应用程序，Proxy 监听即提供这样的功能设置。

点击图中的【Add】按钮，会弹出 Proxy 监听设置对话框，里面有更丰富的设置，满足我们不同的测试需求。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4e1FHibJO5ziadZicoZ58X2MT7Lykiaq0JAic0lwhEficTMECQIKV02aXfkJvQ/640?wx_fmt=png)

请求处理 Request Handling 请求处理主要是用来控制接受到 Burp Proxy 监听端口的请求 后，如果对请求进行处理的。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eqM7JJ8FSwEZ9MeqQzJqYElXjcEeGEKYsuPYFgEJBQ7ZVOJt82BLOng/640?wx_fmt=png)

Burp 工具之 Intruder 模块
--------------------

Burp Intruder 作为 Burp Suite 中一款功能极其强大的自动化测试工具，通常被系统安全渗透测试人员被使用在各种任务测试的场景中。

### Intruder 使用场景和操作步骤

在渗透测试过程中，我们经常使用 Burp Intruder，它的工作原理是：Intruder 在原始请求数据的基础上，通过修改各种请求参数，以获取不同的请求应答。

每一次请求中，Intruder 通常会携带一个或多个 Payload, 在不同的位置进行攻击重放，通过应答数据的比对分析来获得需要的特征数据。

Burp Intruder 通常被使用在以下场景：

1.  标识符枚举 Web 应用程序经常使用标识符来引用用户、账户、资产等数据信息。例如: 用户名，文件 ID 和密码等。
    
2.  提取更加精准，有用的数据 在某些场景下，而不是简单地识别有效标识符，你需要通过简单标识符提取一些其他的数据。
    
    比如说，你想通过用户的个人空间 id，获取所有用户在个人空间标准的昵称和年龄。
    
3.  模糊测试，如 SQL 注入，跨站点脚本和文件路径遍历可以通过请求参数提交各种测试字符串，并分析错误消息和其他异常情况，来对应用程序进行检测。
    
    由 于的应用程序的大小和复杂性，手动执行这个测试是一个耗时且繁琐的过程。
    
    这样的场景，您可以设置 Payload，通过 Burp Intruder 自动化地对 Web 应用程序进行模糊测试。
    

**使用 Burp Intruder 进行测试时，步骤为：**

1.  **确认 Burp Suite 安装正确并正常启动，且完成了浏览器的代理设置。**
    
2.  **进入 Burp Proxy 选项卡，并通过右击菜单，发送到 Intruder。**
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4ePBJib3gUBib3smnHzxvaszI9mQKvR3B2t7wFhmLLrpByaDQzTuHYVOsw/640?wx_fmt=png)**3. 进行 Intruder 选项卡，打开 Target 和 Positions 子选项卡。**

这时，你会看到上一步发送过来 的请求消息。

  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eK2d65mnKaKvP0nQNIN8DS0WTdPlddgglEThohvEibUjibEYFZKb3XdqQ/640?wx_fmt=png)

**4. 因为我们了解到 Burp Intruder 攻击的基础是围绕刚刚发送过来的原始请求信息，在原始信息指定的位置上设置一定数量的攻击 Payload，通过 Payload 来发送请求获取应答消息。**

默认情况下，Burp Intruder 会对请求参数和 Cookie 参数设置成 Payload position，前缀添加 $ 符合，当发送请求时，会将 $ 标识的参数替换为 Payload。

  
5. 在 Position 界面的右边，有【Add $】、【Clear $】、【Auto $】、【Refersh $】四个按 钮，是用来控制请求消息中的参数在发送过程中是否被 Payload 替换，如果不想被替换，则选择此参数，点击【Clear $】, 即将参数前缀 $ 去掉。

  
6. 当我们打开 Payload 子选项卡，选择 Payload 的生成或者选择策略，默认情况下选 择 “Simple list”, 当然你也可以通过下拉选择其他 Payload 类型或者手工添加。

  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eW8XlqSe6zqw1W7Zu9J6KNvglzWzv8dMCfKjEJ7nsGxlcOO9DrTwibQA/640?wx_fmt=png)

7. 此时在界面的右上角，点击【Start attack】，发起攻击。

  
8. 此时，Burp 会自动打开一个新的界面，包含攻击执行的情况、Http 状态码、长度等结果信息。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4e1GEbx8hv7icaaTC8Ct09g5WaDfucic1dV3guy7XVBaCNkv8rBvn7HKtQ/640?wx_fmt=png)

9. 我们可以选择其中的某一次通信信息，查看请求消息和应答消息的详细。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eicsib0CNvnqeC8ib8l42ALluD62mTzCVzHFBI5qm9cRhxbibhPhsLXt3wg/640?wx_fmt=png)观察那些长度异常的结果。

Burp 工具之 Repeater 模块
--------------------

Burp Repeater 作为 Burp Suite 中一款手工验证 HTTP 消息的测试工具，通常用于多次重放请求 响应和手工修改请求消息的修改后对服务器端响应的消息分析。

### Repeater 的使用

在渗透测试过程中，我们经常使用 Repeater 来进行请求与响应的消息验证分析，比如修改请 求参数，验证输入的漏洞；修改请求参数，验证逻辑越权；从拦截历史记录中，捕获特征性 的请求消息进行请求重放。Burp Repeater 的操作界面如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eXvRqTuI4KevtI9KicoLwVg2RKKGaKskHYsIStdDYenib2LnrFnonxHDA/640?wx_fmt=png)请求消息区为客户端发送的请求消息的详细信息，Burp Repeater 为每一个请求都做了请求编 号，当我们在请求编码的数字上双击之后，可以修改请求的名字，这是为了方便多个请求消 息时，做备注或区分用的。

在编号的下方，有一个【GO】按钮，当我们对请求的消息编辑完之后，点击此按钮即发送请求给服务器端。

服务器的请求域可以在 target 处进行修改，如上图所示。

应答消息区为对应的请求消息点击【GO】按钮后，服务器端的反馈消息。

通过修改请求消息的参数来比对分析每次应答消息之间的差异，能更好的帮助我们分析系统可能存在的漏洞。

在我们使用 Burp Repeater 时，通常会结合 Burp 的其他工具一起使用，

比如 Proxy 的历史记录，Scanner 的扫描记录、Target 的站点地图等，通过其他工具上的右击菜单，执行【Send to Repeater】，跳转到 Repeater 选项卡中，然后才是对请求消息的修改以及请求重放、数据分析与漏洞验证。

### 可选项设置（Options）

与 Burp 其他工具的设置不同，Repeater 的可选项设置菜单位于整个界面顶部的菜单栏中，如图所示：

  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4ebS0k1ekhl2d3QnRiaHjrp1QQ5fVGGUIibBt7L35lKkTy9FECBicspkwjA/640?wx_fmt=png)

### 可选项设置（Options）

与 Burp 其他工具的设置不同，Repeater 的可选项设置菜单位于整个界面顶部的菜单栏中，如 图所示：

  
**其设置主要包括以下内容：**

*   更新 Content-Length，这个选项是用于控制 Burp 是否自动更新请求消息头中的 Content-Length。
    
*   解压和压缩（Unpack gzip / deflate ）这个选项主要用于控制 Burp 是否自动解压或压缩服务器端响应的内容。
    
*   跳转控制（Follow redirections） 这个选项主要用于控制 Burp 是否自动跟随服务器端作请 求跳转，比如服务端返回状态码为 302，是否跟着应答跳转到 302 指向的 url 地址。它有 4 个选项，分别是永不跳转（Never），站内跳转（On-site only ）、目标域内跳转（In- scope only）、始终跳转（Always），其中永不跳转、始终跳转比较好理解，站内跳转 是指当前的同一站点内跳转；目标域跳转是指 target scope 中配置的域可以跳转。
    
*   跳转中处理 Cookie（Process cookies in redirections ） 这个选项如果选中，则在跳转过 程中设置的 Cookie 信息，将会被带到跳转指向的 URL 页面，可以进行提交。
    
*   视图控制（View） 这个选项是用来控制 Repeater 的视图布局。
    
*   其他操作（Action） 通过子菜单方式，指向 Burp 的其他工具组件中。
    

Burp 工具之 Target 模块
------------------

Burp Target 组件主要包含站点地图、目标域、Target 工具三部分组成，他们帮助渗透测试人员更好地了解目标应用的整体状况、当前的工作涉及哪些目标域、分析可能存在的攻击面等信息。

### 站点地图 Site Map

对封神台靶场进行抓包，进行渗透测试，通过浏览器浏览的历史记录在站点地图中的展现结果。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4ehQ4AVqHiboMG4hGkxCTqSS3EJQTGdIRf73HDN15bicnpcpgN4GbbVpRA/640?wx_fmt=png)从图中我们可以看出，Site Map 的左边为访问的 URL，按照网站的层级和深度，树形展示整个应用系统的结构和关联其他域的 url 情况；

右边显示的是某一个 url 被访问的明细列表，共访问哪些 url，请求和应答内容分别是什么，都有着详实的记录。

基于左边的树形结构，我们可以选择某个分支，对指定的路径进行扫描和抓取。

同时，我们也可以将某个域直接加入 Target Scope 中。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eJqibvaUibTFaB4ibejCRChOoLnxKyNybVickbnGRYO824Fr093w3IicH6Zg/640?wx_fmt=png)除了加入 Target Scope 外，从上图中我们也可以看到，对于站点地图的分层，可以通过折叠和展开操作，更好的分析站点结构。

### 目标域设置 Target Scope

Target Scope 中作用域的定义比较宽泛，通常来说，当我们对某个产品进行渗透测试时，可以通过域名或者主机名去限制拦截内容，这里域名或主机名就是我们说的作用域；

如果我们想 限制得更为细粒度化，比如，你只想拦截 login 目录下的所有请求，这时我们也可以在此设 置，此时，作用域就是目录。

总体来说，Target Scope 主要使用于下面几种场景中：

*   限制站点地图和 Proxy 历史中的显示结果
    
*   告诉 Burp Proxy 拦截哪些请求
    
*   Burp Spider 抓取哪些内容
    
*   Burp Scanner 自动扫描哪些作用域的安全漏洞
    
*   在 Burp Intruder 和 Burp Repeater 中指定 URL
    

通过 Target Scope 我们能方便地控制 Burp 的拦截范围、操作对象，减少无效的噪音。

在 Target Scope 的设置中，主要包含两部分功能：允许规则和去除规则。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4em7zqPFdRJ8s8yQsDHrb7MdxVSguMc9hIahvFuusH4lkBAefGyNTaHg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eOzogeWfPAMacB5IULfjVtmHBp1Rb0pXcrVqSic0uoqyEHqMGkjrxFiaQ/640?wx_fmt=png)  
其中允许规则很容易理解，即包含在此规则列表中的操作允许、有效。

如果此规则用于拦截，则请求消息匹配包含规则列表中的将会被拦截；否则，请求消息匹配去除列表中的将不会被拦截。

  
规则主要由协议、域名或 IP 地址、端口、文件名 4 个部分组成， 这就意味着我们可以从协议、域名或 IP 地址、端口、文件名 4 个维度去控制哪些消息出现在允 许或去除在规则列表中。

  
当我们设置了 Target Scope （默认全部为允许），使用 Burp Proxy 进行代理拦截

在渗透测 试中通过浏览器代理浏览应用时，Burp 会自动将浏览信息记录下来，包含每一个请求和应答的详细信息，保存在 Target 站点地图中。

### **如何使用 Target 工具**

*   手工获取站点地图  
    当我们手工获取站点地图时，
    
    需要遵循以下操作步骤：
    
    **1. 设置浏览器代理和 Burp Proxy 代理。**
    
    **2. 关闭 Burp Proxy 的拦截功能。**
    
    **3. 手工浏览网页，**这时，Target 会自动记录站点地图信息。这种方式的好处在于我们可以根据自己的需 要和分析，自主地控制访问内容，记录的信息比较准确。但缺点就是比自动抓取相比需要更长的时间进行渗透测试，对渗透人员付出的时间和精力是很大的。
    
*   站点比较  
    这是一个 Burp 提供给渗透测试人员对站点进行动态分析的利器，我们在比较帐号权限 时经常使用到它。
    
    当我们登陆应用系统，使用不同的帐号，帐号本身在应用系统中被赋予了 不同的权限，那么帐号所能访问的功能模块、内容、参数等都是不尽相同的，此时使用站点比较，能很好的帮助渗透测试人员区分出来。
    
    > 一般有三种场景:  
    > 同一个帐号，具有不同的权限，比较两次请求结果的差异;.  
    > 两个不同的帐号，具有不同的权限，比较两次请求结果的差异；  
    > 两个不同的帐号，具有相同的权限，比较两次请求结果的差异。
    

**如何对站点进行比较：**

1. 首先我们在需要进行比较的功能链接上右击， 找到站点比较的菜单，点击菜单进入下一步。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eTiafY1qQZVab3GrZQAZgiaZn2oDMiczp3EQAckkiayxepVe5vHvGVD5eYQ/640?wx_fmt=png)

2. 由于站点比较是在两个站点地图之间进行的，所以我们在配置过程中需要分别指定 Site Map 1 和 Site Map2。

通常情况下，Site Map 1 我们默认为当前会话。

如图所示，点击【Next】。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eN2clnAhQv34rw2qbWwMdrzdDRibnibZ0ZcYXEfHjnH49O8z0ICUAb16A/640?wx_fmt=png)  
  

3. 这时我们会进入 Site Map 1 设置页面，如果是全站点比较我们选择第一项，如果仅仅比较我们选中的功能，则选择第二项。

如下图，点击【Next】。如果全站点比较，且不想加载其他域时，我们可以勾选只选择当前域。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eJibIjQZ6dvib5GCE16k43cKB4cTA6l5rRzFuxSAVt7O2PSIw4gNfALQw/640?wx_fmt=png)  
  

4. 接下来就是 Site Map 2 的配置，对于 Site Map 2 我们同样有两种方式

第一种是之前我们已经保存下来的 Burp Suite 站点记录

第二种是重新发生一次请求作为 Site Map2. 这里，我们选择第二种方式。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4ewGZ8FasoChmH1dL8WibHs4gFNe1GcyL7RuCrqKpFuhPxqXoqOwJh0pA/640?wx_fmt=png)  
  

5. 如果上一步选择了第二种方式，则进入请求消息设置界面。

在这个界面，我们需要指定通信 的并发线程数、失败重试次数、暂停的间隙时间。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4ey4mIhhrFQQ1Cfa4PLFxiaqyVtX0ibKicO5r4NjfA695g6jAzDSCk10tzA/640?wx_fmt=png)  

6. 设置完 Site Map 1 和 Site Map 2 之后，将进入请求消息匹配设置。

在这个界面，我们可以通过 URL 文件路径、Http 请求方式、请求参数、请求头、请求 Body 来对匹配条件进行过滤。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eLIEXjtc99piaqiaJfawW6vV11B5HNFrISVdFEo7pxvAcCR720opBu3fA/640?wx_fmt=png)  

7. 设置请求匹配条件，接着进入应答比较设置界面。

在这个界面上，我们可以设置哪些内容我们指定需要进行比较的。

从下图我们可以看出，主要有响应头、form 表单域、空格、MIME 类型。

点击【Next】。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eWeediaZXh4EicIibpvS4JEJibTWThlojiaJSyS6auMCHlchklxhHM1vQ7Lg/640?wx_fmt=png)  
  

8. 如果我们之前是针对全站进行比较，且是选择重新发生一次作为 Site Map2 的方式，则界面加载过程中会不停提示你数据加载的进度

如果涉及功能请求的链接较少，则很快进入比较界面。如下图。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eniaSItuMW2JpZvMa7opdHArmAVhSk6zHw7DpooCWab16fdN2oXlq99g/640?wx_fmt=png)

9. 从上图我们可以看到，站点比较的界面上部为筛选过滤器，下部由左、中、右三块构成。

左边为请求的链接列表，中间为 Site Map 1 和 Site Map 2 的消息记录，右边为消息详细信息。

当我们选择 Site Map 1 某条消息记录时，默认会自动选择 Site Map 2 与之对应的记录，这是有右上角的【同步选择】勾选框控制的，同时，在右边的消息详细区域，会自动展示 Site Map 1 与 Site Map 2 通信消息的差异，包含请求消息和应答消息，存在差异的地方用底色标注出来。

  
![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4e6aB6nfJZjRllDxYf0SkubicWEmUsluE3dQnFggAyR2qz1PmClVmibdOA/640?wx_fmt=png)

*   **攻击面分析**  
    我们来看一下 Analyze Target 的使用：1. 首先，我们通过站点地图，打开 Analyze Target。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4esDexBc6npNEY3micAMicyjEF1cYqtBySF0SfTrZoISmvxWp9vT1JvkPg/640?wx_fmt=png)  
2. 在弹出的分析界面中，我们能看到概况、动态 URL、静态 URL、参数 4 个视图。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eB0J51CKlPS17MGmddqsHubaEUykrw9nldYibJvFqaAnhIDIRHh1zovA/640?wx_fmt=png)  
3. 概况视图主要展示当前站点动态 URL 数量、静态 URL 数量、参数的总数、唯一的参数名数目，通过这些信息，我们对当前站点的总体状况有粗线条的了解。

  
4. 动态 URL 视图展示所有动态的 URL 请求和应答消息，跟其他的工具类似，当你选中某一条消息时，下方会显示此消息的详细信息。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eSiaD75nZLcM77U088aB9sypWnicOvb1kQzzFFvbfv4RhYbYoL6Gwy43g/640?wx_fmt=png)  
5. 静态和动态的 URL 视图是类似的。

  
6. 参数视图有上中下三部分组成，上部为参数和参数计数统计区，你可以通过参数使用的次数进行排序，对使用频繁的参数进行分析；中部为参数对于的使用情况列表，记录对于的参数每一次的使用记录；下部为某一次使用过程中，请求消息和应答消息的详细信息。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFCdGXhicRgt81Vn0NE6iaC4eIiackAO4DGjwKJzoPqibqD98gaqVnWvjkr0aVnwp4gjHV4ibPicmofQOnA/640?wx_fmt=png)  
在使用攻击面分析功能时，需要注意，**此功能主要是针对站点地图中的请求 URL 进行分析，如果某些 URL 没有记录，则不会被分析到**。

同时，在实际使用中，**存在很点站点使用伪静态，如果请求的 URL 中不带有参数，则分析时无法区别，只能当做静态 URL 来分析。**

学习更多黑客技能！体验靶场实战练习  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSGknl6nDn5MaRdzyQMyPhYianWUJMpU15llQnriagO3LGFJs7ZcYkO4kVP2iau3jy2gAXRThy33ZakRA/640?wx_fmt=jpeg)

（黑客视频资料及工具）

![图片](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSHBJKticajXibyFhPZib93CYVV4AbIDmNFwsxeOgDP0p5MDdLQZtamZ6bP7MJ86g8UkYr1tc90kibk7pQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFTib5w98ocX6Sx1YcmgS0tfPOIyEmD8jse5YLoeZzDibM8rNrQibZPsibKXekZaR8FFV3flUT84nU0LQ/640?wx_fmt=png)

往期内容回顾

![图片](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFTib5w98ocX6Sx1YcmgS0tfPOIyEmD8jse5YLoeZzDibM8rNrQibZPsibKXekZaR8FFV3flUT84nU0LQ/640?wx_fmt=png)

[渗透某赌博网站杀猪盘的经历！](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247493616&idx=1&sn=0299bced5d71aa60c9cea5b77a3b8fe7&chksm=ebeaaedddc9d27cb805b5e970f5554c47c32a69fdb9ee39acd1642a163e55b17f429e5db06a7&scene=21#wechat_redirect)  

[手把手教你如何远程监听他人手机](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247489276&idx=1&sn=1ceeadb6a7867d352b8ad559df06e5cd&chksm=ebe95fd1dc9ed6c7df69833d35b0cdb306a3818a649432fba41c14cf1599c9ce0d12177403fc&scene=21#wechat_redirect)

[黑客网站大全！都在这了！速看被删就没了](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247493025&idx=1&sn=97a10a4eca361ad2f66435f89bdcf2a3&chksm=ebeaac8cdc9d259ac26623014a38181b60ba57af9577f4e062e0ed9f33baef84ff60e645a6e6&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/CBJYPapLzSFTib5w98ocX6Sx1YcmgS0tfPyjxT8Q78w0uBADoIltpF1KribvWfHicVlFwShJRIxZls99XR1jaEYow/640?wx_fmt=gif)
