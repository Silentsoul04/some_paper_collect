\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzIxOTAzMjUzOA==&mid=2247484133&idx=1&sn=cea5a69ccbe6743ad9df1cb76478faee&chksm=97e03c50a097b546b08daa0b00752efd9c69ce00e6b2f656515697a850be7255e42e98f01a41&mpshare=1&scene=1&srcid=1022UAEZpqqQAP5UGgmJFEcb&sharer\_sharetime=1603377901335&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=9d3ca08bb5be9f04b534cc41a285097010854e8dc9487eba1c1f5f30e866eb10b1cca854a9936a1d2d2274e9e94d53a5a9ad2cbe2fcf17694a05592c92fffbf874a5025a58f8b851580fd7c3ed5abdb47c8709d6132a882f6e60ed9eef82f5f29e3a9ee86c1d53125cc17e38ec1db7a6afcfc3a4074e109365803261d71adaa5&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=ARf0PYo0yrFV0Ed%2Fzwzpovs%3D&pass\_ticket=MIC5Ar%2FikzMcOH1F8HNnc411WxyFMo1Kw3L353SY3XmezYiEUuovrlDORbkreA49&wx\_header=0)

![](https://mmbiz.qpic.cn/mmbiz_gif/pajG4l5Gnt55HcTYDo5mBdiaV28ZRC2VhLwD8yr2ySHApdcEPddVhGu5jBNuSNY0go6HJMCbeibDMAUKiaUvSGucg/640?wx_fmt=gif)

在使用 java.lang.Runtime#exec() 执行命令时，为何有时候命令前缀需要加 cmd /c 或者 bash -c？今天就来一探究竟！

### **Java 执行命令的 3 种方法**

**首先了解下在 Java 中执行命令的方法：**

常用的是 java.lang.Runtime#exec() 和 java.lang.ProcessBuilder#start()，除此之外，还有更为底层的 java.lang.ProcessImpl#start()，他们的调用关系如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOyKBqYCodjbo1QZib5LLDiaRr9XMmITceeTUT8Lhhhib9KiaUiaSkqp29FDA/640?wx_fmt=png)

其中，ProcessImpl 类是 Process 抽象类的具体实现，且该类的构造函数使用 private 修饰，所以无法在 java.lang 包外直接调用，只能通过反射调用 ProcessImpl#start() 方法执行命令。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOMjOCrfTfc7rD9pFHIFVgvs3ojxOtrLtDeXAQic3mrjcRvpSlvKSqgGg/640?wx_fmt=png)

**这 3 种执行方法如下：**

### java.lang.Runtime

```
public static String RuntimeTest() throws Exception {    InputStream ins = Runtime.getRuntime().exec("whoami").getInputStream();    ByteArrayOutputStream bos = new ByteArrayOutputStream();byte\[\] bytes = new byte\[1024\];int size;while((size = ins.read(bytes)) > 0)        bos.write(bytes,0,size);return bos.toString();}
```

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBONQrDmLBgxdC5kTL6posicFgicJc26Zs63RASFGbFevvo5PJwWmoeicMFA/640?wx_fmt=png)

### java.lang.ProcessBuilder

```
public static String ProcessTest() throws Exception {
  String\[\] cmds = {"cmd","/c","whoami"};
    InputStream ins = new ProcessBuilder(cmds).start().getInputStream();
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
byte\[\] bytes = new byte\[1024\];
int size;
while((size = ins.read(bytes)) > 0)
        bos.write(bytes,0,size);
return bos.toString();
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOGMubCYRaHhiafNxUh46vtnLclopzyic7tyUjNl8VNUCpLzCxDBpAgUHw/640?wx_fmt=png)

### java.lang.ProcessImpl

```
public static String ProcessImplTest() throws Exception {
    String\[\] cmds = {"whoami"};
    Class clazz = Class.forName("java.lang.ProcessImpl");
    Method method = clazz.getDeclaredMethod("start", new String\[\]{}.getClass(),Map.class,String.class,ProcessBuilder.Redirect\[\].class,boolean.class);
    method.setAccessible(true);
    InputStream ins = ((Process) method.invoke(null,cmds,null,".",null,true)).getInputStream();
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
byte\[\] bytes = new byte\[1024\];
int size;
while((size = ins.read(bytes)) > 0)
      bos.write(bytes,0,size);
return bos.toString();
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOrpM6U0kvPF6k5Xz2iaBym8Vz6pCBF0kmj4IPicsYn71QiajKl9s6KunAg/640?wx_fmt=png)

问题：

当直接将命令字符 echo echo\_test > echo.txt 传给 java.lang.Runtime#exec() 执行时报错：

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOjOv3GqahFFUJrqHKfWMaicuEzG31wbrr3qSpQmMvUFd4fNcZl6K63nQ/640?wx_fmt=png)

加上 cmd /c 可以成功执行：

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOT6LgYH2zVJnl2pmDLHRk3eVUYcibvUVIpvichpYDEpcaa8wh7icHL0Ipw/640?wx_fmt=png)

我们跟进下代码看看是什么原因导致的？

命令执行解析流程：

传入命令字符串 echo echo\_test > echo.txt 进行调试，跟进 java.lang.Runtime#exec(String)，该方法又会调用 java.lang.Runtime#exec(String,String\[\],File)。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBORWusicFUqPAh7JIH2Z1afkTlba13hs5WQYp2OgibqCHl4jzG5VhpRZjA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOFunwHn4pAEiaFYY18r4GQ3CTLFsJFMYPpLRD6sEuH69nic6tP2knmppQ/640?wx_fmt=png)

在该方法中调用了 StringTokenizer 类，通过特定字符对命令字符串进行分割，本地测试如下：

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOfhHdB1JhXXdXn4IicbXHusSYdazlRUFqPmlawxQ2JQRVDlSAnX8X21g/640?wx_fmt=png)

所以命令字符串 echo echo\_test > echo.txt 经过 StringTokenizer 类处理后得到命令数组:{"echo","echo\_test",">","echo.txt"} 。另外 java.lang.Runtime#exec() 共有 6 个重载方法，代码如下：

```
public Process exec(String command) throws IOException {return exec(command, null, null);}public Process exec(String cmdarray\[\]) throws IOException {return exec(cmdarray, null, null);}  public Process exec(String command, String\[\] envp) throws IOException {return exec(command, envp, null);}public Process exec(String command, String\[\] envp, File dir)throws IOException {if (command.length() == 0)throw new IllegalArgumentException("Empty command");  StringTokenizer st = new StringTokenizer(command);  String\[\] cmdarray = new String\[st.countTokens()\];for (int i = 0; st.hasMoreTokens(); i++)    cmdarray\[i\] = st.nextToken();return exec(cmdarray, envp, dir);}public Process exec(String\[\] cmdarray, String\[\] envp) throws IOException {return exec(cmdarray, envp, null);}public Process exec(String\[\] cmdarray, String\[\] envp, File dir)throws IOException {return new ProcessBuilder(cmdarray)    .environment(envp)    .directory(dir)    .start();}
```

这 6 个重载函数根据参数不同进行区分，主要是传入字符串跟数组两种形式，但是最终调用的都是最后一个 exec(String\[\],String\[\],File)，在该函数内部首先调用 ProcessBuilder 类的构造函数创建 ProcessBuilder 对象，然后调用 start()，最终返回一个 Process 对象。

所以 Runtime#exec()底层还是调用的 ProcessBuilder#start(), 且传入构造函数的参数要求是数组类型 (如下图)，所以传给 Runtime#exec() 的命令字符串需要先使用 StringTokenizer 类分割为数组再传入 ProcessBuilder 类。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOL4Piaq06CIZcwnUokM6ILb5LIG9mv55PwWic8QWSdGERVrwTwPbzRibUg/640?wx_fmt=png)

接着跟进 java.lang.ProcessBuilder#start()，取出 cmdarray\[0\] 赋值给 prog, 如果安全管理器 SecurityManager 开启, 会调用 SecurityManager#checkExec() 对执行程序 prog 进行检查，之后调用 ProcessImpl#start()。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOrllqRcAtGqVFbuy1iaakbSdkTV8AR28AzuODdXpsglZNHaUmicMcVCiaQ/640?wx_fmt=png)

跟进 java.lang.ProcessImpl#start()，Windows 下会调用 ProcessImpl 类的构造方法，如果是 Linux 环境，则会调用 java.lang.UNIXProcess#init<>。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOCIEkWTib16CjicTadOydtylZ4L9YAhqm9fWD2QbDgrWZf6gELF4wd1Ww/640?wx_fmt=png)

**跟进 java.lang.ProcessImpl 的构造方法**

该方法内 allowAmbiguousCommands 变量为 "是否允许调用本地进程" 的开关，在安全管理器未开启且 jdk.lang.Process.allowAmbiguousCommands 不为 false 时，allowAmbiguousCommands 变量值才为 true。当系统允许调用本地进程时，进入 Legacy mode(传统模式)，会调用 needsEscaping()，当 prog 存在空格且未被双引号包裹时需要使用 quoteString() 进行处理，接着调用 createCommandLine() 将命令数组拼接为命令字符串，最后调用 create() 创建进程。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOOGrs91Kvyg3ia3K2jsWISxec97WHJydYxtzsQgvDGKtBBP2ibd29nw6w/640?wx_fmt=png)

传统模式下，当可执行程序 prog 存在 \\ t 或空格时，该函数返回 true，即需要双引号包裹处理。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOuBHzjcOokibgoiaZRz617ibMen5WvVR8uX0hnXdtyMwJATglp88qGm9bg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOv2oLcYDfccgpufqIprjtPGHibV2G6PLicGB3jqqNBnY3L4VfNUOK6otg/640?wx_fmt=png)

最后调用 ProcessImpl#create()，这是一个 native 方法，根据 JNI 命名规则，会调用到 ProcessImpl\_md.c 中的 Java\_Java\_lang\_ProcessImpl\_create()，该函数会调用 Windows 系统 API 函数：CreateProcessW()，用来创建一个新的 Windows 进程。创建成功后，将新进程的句柄返回给 ProcessImpl#create()。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOAsU2a6DyCyIdnrzicia1Qw1mPfoNt5l6nWOS7dt3MYzwsYAUyuIozPLA/640?wx_fmt=png)

看下 CreateProcessW()怎么处理我们传入的命令的：当第一个参数 (lpApplicationName) 为 0 时，第二个参数 pcmd(lpCommandLine)需要提供启动程序及所需参数，彼此间以空格隔开。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBODic9YyLCAFhTGBIt7mvCaswhz0T4Z4r28gl1tth6icFpukutW3X1UwXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOmzHOLLiazTDkUoawoV61tw8qufXiaVB2MrA9NrPavR0Ncd4dV24DibBnA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBO1R6qztBMUWtFebicuU7QyRGsc8sM35iaXzWicrNDfLKs0w7vosBiaOoeIg/640?wx_fmt=png)

测试 ProcessImpl#create() 方法：

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOYhhUSm15fKfOQ21dia8DcWhcBD4WOAdygyMVCv7gUEFq2qOMgIfUAXA/640?wx_fmt=png)

加上 cmd /c 之后，成功执行命令：

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOxnOliaj6UuOARzBXHPF14aWZIXHN6IUSdRF0pVqtMiczc9ZQg1UXGlzg/640?wx_fmt=png)

**需要添加 cmd /c 的原因:**

在传入 echo echo\_test > echo.txt 命令字符串时，出现错误 ("java.io.IOException: Cannot run program"echo": CreateProcess error=2, 系统找不到指定的文件。")。原因是 echo 为命令行解释器 cmd.exe 的内置命令，并不是一个单独可执行的程序 (如下图)，所以如果想执行 echo 命令写文件需要先启动 cmd.exe，然后将 echo 命令做为 cmd.exe 的参数进行执行。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOEEkY9pje8icliaYkrvWXYe0zicNbVM2Qq2FG7Q9sJC1hImPicWZtmicgPTA/640?wx_fmt=png)

另外关于 cmd 下的 /c 参数，当未指定时, 运行如下示例程序, 系统会启动一个 pid 为 8984 的 cmd 后台进程，由于 cmd 进程未终止导致 java 程序卡死。当指定 / c 时，cmd 进程会在命令执行完毕后成功终止。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBO5h6p4dxiaxG7xciaqvrtm9cP0hRiamG1Hia5Dj7qAJCHbBbM9qUwXqeJIQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBO6P4gnSGdHp2MtWTsQSkE9Fcn6LmCJRuYibPqNljeG7quDrzvGZ8mysw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBOns5X3NATqMhZXWzmohs3tiarpGZOTIXhz14ZQImVSIab5Pk8cdDQAzQ/640?wx_fmt=gif)

**所以在 Windows 环境下，使用 Runtime.getRuntime() 执行的命令前缀需要加上 cmd /c，使得底层 Windows 的 processthreadsapi.h#CreateProcessW() 方法在创建新进程时，可以正确识别 cmd 且成功返回命令执行结果。**

**未完待续...  
**

360BugCloud 开源漏洞响应平台，国内自主议价漏洞收录模式开拓者！聚焦收录未被披露的开源以及通用组件高危漏洞，致力于维护开源软件和供应链安全。平台采用入驻邀请制，只面向成功提交未被披露漏洞的安全研究员开放。360BugCloud 开源漏洞响应平台首创 “自主议价” 模式及 “第三方专家评审” 机制，先议价后交洞，仅需提交漏洞影响力描述即可进行议价，让安全研究员完全掌握漏洞提交主动权，高额奖金上不封顶，让漏洞价值得到充分保障与肯定。

**6** **步轻松实现在 360BugCloud 提交漏洞**

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt47NrPdS5Fjg9X5ELN8k4ERHSNkViaRyJQ223ic9W4VRobKYicfic4oaIhibuO1PBrbV5K5YrGBoye67IA/640?wx_fmt=png)

**360BugCloud 漏洞提交地址**

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt6MgPpQNjUVSTFfmoOribhBO9lz5OuynFTDtpGCZYjLxqjRyFqJTKRfpjDibuR0NISwgHyHyOIxnHhw/640?wx_fmt=png)

360BugCloud 开源漏洞响应平台秉承 “Trust 信任、Tenet 原则、Top 权威、Together 共建” 的 04T 宗旨，力争打造以技术为驱动、以安全专家为核心的应急响应平台，提升网络安全防护能力，为国家、企业、用户打造最安全的网络环境。

![](https://mmbiz.qpic.cn/mmbiz_png/pajG4l5Gnt55HcTYDo5mBdiaV28ZRC2Vh0Qzjvba44iaLcKJiaWbaWeQdbDoRRe0mGIspGYjQUTWoHicHbSia8icOFgA/640?wx_fmt=png)