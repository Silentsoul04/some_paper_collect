\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/iaMrql\_gJf0Su2jnh93FYQ)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaa62yRh8ZMicGSoozvuoh0ibFQGj4hjhLCxqwV8T0z3NBPjjnvfZcyObNAEnOib0bH4lRT5dj1Rawd7g/640?wx_fmt=png)

点击蓝字关注我们吧！

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyogBy7xAQibchicFl7vsK9o0N08trysCI6Ux2fBQkRpfZNehBJLTaib0v0vBhpOIcdw67iapXZE6RLkAg/640?wx_fmt=png)

1.Windows Hook 的概念

  

*   Windows 操作系统的图形用户界面（Graphic User Interface）以事件驱动 （Event Driven）的方式工作
    
*   在操作系统中键盘、鼠标，选择菜单、按钮，以及移动鼠标、改变窗口大小 与位置等都是事件
    
*   发生这样的事件时，OS 会把事先定义好的消息发送给相应的应用程序，应用 程序分析收到的信息后执行相应动作（上述过程在《Windows 程序设计》一书中有详尽说明）
    
*   也就是说，敲击键盘时，消息会从操作系统内核移动到应用程序。所谓的 “消息钩子” 就在此间偷看这些信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyogBy7xAQibchicFl7vsK9o0N08trysCI6Ux2fBQkRpfZNehBJLTaib0v0vBhpOIcdw67iapXZE6RLkAg/640?wx_fmt=png)

2\. 常规 Windows 消息流

*   发生键盘输入事件时，WM\_KEYDOWN 消息 被添加到 \[OS message queue\]
    
*   OS 判断哪个应用程序中发生了事件，然后从 \[OS message queue\] 取出消息，添加到相应应 用程序的 \[application message queue\] 中
    
*   应用程序（如记事本）监视自身的 \[application message queue\]，发现新添加的 WM\_KEYDOWN 消息后，调用相应的事件处理程序处理
    

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCKg6uZYhOPhnj6ic7gicCnvBuuzqFPOEicLaUYuC3rtkTSIhfezqx6XiaibXA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyogBy7xAQibchicFl7vsK9o0N08trysCI6Ux2fBQkRpfZNehBJLTaib0v0vBhpOIcdw67iapXZE6RLkAg/640?wx_fmt=png)

3\. 被 Hook 后的消息流

*   OS 消息队列与应用程序消息队列之间存在一条 “钩链”（Hook Chain），设置好键盘消息钩子之后，处于“钩链” 中的键盘消息钩子会比应用程序先看到相应信息。
    
    在键盘消息钩子函数的内部，除了可以查看消息之外，还可以修改消息本身，而且还能对消息实施拦截，阻止消息传递
    
*   可以同时设置多个相同的键盘消息钩子，这些钩子按照设置的先后顺序被触发，最近安装的钩子放在链的开始，而最早安装的钩子放在最后，也就是后加入的先获得控制权
    

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCKibvaCljoJFiboyIImsRD6eTarJekfibcKQiae9XDmza9YpNV6AsFrLr9og/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyogBy7xAQibchicFl7vsK9o0N08trysCI6Ux2fBQkRpfZNehBJLTaib0v0vBhpOIcdw67iapXZE6RLkAg/640?wx_fmt=png)

4.SetWindowsHookEx()

*   使用 SetWindowsHookEx( ) API 设置消息钩子 - **若 dwThreadID 参数被设置为 0**，
    
    则安装的钩子为 “全局钩子”（GlobalHook），
    
    它会影响到运行中的（以及以后要运行的）所有进程。
    
*   使用 SetWindowsHookExO 设置好钩子之后，在某个进程中生成指定消息时，操作系统会将相关的 DLL 文件强制注入到相应进程然后调用注册的钩子
    

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyogBy7xAQibchicFl7vsK9o0N08trysCI6Ux2fBQkRpfZNehBJLTaib0v0vBhpOIcdw67iapXZE6RLkAg/640?wx_fmt=png)

5.Hook 键盘输入

程序代码由两部分组成

*   1\. **HookMain**：用于加载 DLL 文件并控制 Hook 的设置和取消
    
    编译后为一个**.exe** 可执行文件
    
*   2\. **\- KeyHook**：Hook 相关的主要代码
    
    编译后为一个.**dll** 文件
    

**5.1.HookMain**  

-------------------

相当于一个 DLL 加载器

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCK5fADmxeQCBlJ9kCUceicxfFGI0ojMrqibwHzuMiaIyVUG2pozPiawR3jHQ/640?wx_fmt=png)

**5.2.KeyHook**
---------------

KeyHook.dll 的核心代码

安装好键盘 “钩子” 后，无论哪个进程，只要发生键盘输入事件，

OS 就会强制将 KeyHook.dll 注入相应进程。

加载了 KeyHook.dll 的进程中，发生键盘事件时会首先调用执行 KeyboardProc( )

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCK5Q30pRndCYe7hVy7JtCh92NibIRmXsLlE4wcPWzq7iaoY5HExRwfDGnA/640?wx_fmt=png)

**5.3. 在编译时注意选择合适的版本：**
-----------------------

Release x64
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCKF7CaTw4mmDM2yGU6ODme4UgzNCibeDjtfruWx0S9AYuibfTV9pbQwRVw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCK3Aof2mbe9t5dwwIicbNc1DNfhvoM1PmtFXaU21ia88icUPRkXXpPpdLqw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCK3Aof2mbe9t5dwwIicbNc1DNfhvoM1PmtFXaU21ia88icUPRkXXpPpdLqw/640?wx_fmt=png)

**5.4. 打开 HookMain.exe**
------------------------

无法在 1.txt 中输入任何内容

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ09zSRD770a8JwXfov4ZlcCKZibY2J4icLMXGGxzpjzf52iakH4GSa5tvYj4v17ZI8FH9MBFCGH2k7sqg/640?wx_fmt=png)

**5.5.**
--------

在 exe 中输入 q 退出后，又可以在 1.txt 中输入了
-------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyogBy7xAQibchicFl7vsK9o0N08trysCI6Ux2fBQkRpfZNehBJLTaib0v0vBhpOIcdw67iapXZE6RLkAg/640?wx_fmt=png)

6\. 其他

**6.1.Hook 还有很多种实现方式**
----------------------

\- 注入：inline、DLL

\- 调试：通过调试相关的 API 进行 Hook

**6.2.Hook 的目标也有多种**
--------------------

\- IAT、代码、EAT

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibiarFfCb4DNv9CkaFnPsQtetjvmfJOuDSkOiabHLmyOxzsicSRHsmqFmRovFic9TT0iaY4P6mSxSwu9Rw/640?wx_fmt=png)