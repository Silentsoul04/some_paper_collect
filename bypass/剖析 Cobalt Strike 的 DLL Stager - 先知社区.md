> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9512)

NVISO 最近监测到了针对其金融部门客户的作为目标的行动。根据员工关于可疑邮件的报告，该攻击在其开始阶段就被发现。虽然没有造成任何伤害，但我们通常会确定任何与之相关的指标，以确保对实施此行为者进行额外的监视。

被报告邮件是申请公司其中一个公开招聘的职位，并试图发送恶意文档。除了利用实际的工作 Offer, 还引起我们注意的是恶意文档中还存在 [execution-guardrails](https://attack.mitre.org/techniques/T1480/) 。通过对该文档分析，发现了通过 [Component Object Model Hijacking](https://attack.mitre.org/techniques/T1546/015/)（组件对象模型劫持) 来维持 Cobalt Strike Stager 的意图

在我空闲的时间里，我很享受分析 NVISO 标记的野外样本, 因此进一步解剖 Cobalt Strike DLL payload，这篇博客文章将介绍有效载荷的结构，设计选择，并重点介绍如何减少日志足迹和缩短 Shellcode 的时间窗口。

分析执行流
-----

为了了解，恶意代码是如何工作的，我们必须去分析其从开始到结束的行为。在本节中，我们将介绍一下流程。

1. 通过`DllMain`初始化执行

2. 通过`WriteBufferToPipe`, 将加密 shellcode 发送到命名管道

3. 通过管道进行读取，通过`PipeDecryptExec`, 解密 shellcode 并执行

如前所述，恶意文档的 DLL 的载荷意图伪装为 [COM in-process server](https://docs.microsoft.com/en-us/windows/win32/com/inprocserver32)。有了这些认识，我们可以着眼与 DLL 公开的一些已知的入口点。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164313-c505de32-a7fd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164313-c505de32-a7fd-1.png)

从技术层面来说，恶意代码可以发生在该 8 个函数中任意一个，但恶意代码通常驻留在`DllMain`给定的函数中，除了 [TLS callbacks](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#tls-callback-functions), 它是最可能执行的函数。

> `DllMain`：动态链接库（DLL）的可选入口点。当系统启动或终止进程或线程时，它将使用进程的第一个线程为每个已加载的 DLL 调用入口点函数。当使用`LoadLibrary`和`FreeLibrary`函数加载或卸载 DLL 时，系统也会调用 DLL 的入口点函数。
> 
> [docs.microsoft.com/zh-CN/windows/win32/dlls/dllmain](https://docs.microsoft.com/en-us/windows/win32/dlls/dllmain)

DllMain 入口点
-----------

从下面的捕获结果可以看到，该`DllMain`函数只是通过创建一个新线程来简单执行另一个函数。该线程函数我们命名为`DllMainThread`, 它不需要提供任何参数即可执行。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164330-cea4bc74-a7fd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164330-cea4bc74-a7fd-1.png)

分析`DllMainThread`函数发现其实它是对我们将发现的恶意载荷的解密和执行函数的一个附加的包装.(被保护函数在捕获中被称为`DecryptBufferAndExec`)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164351-db5b13aa-a7fd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164351-db5b13aa-a7fd-1.png)

进一步深入, 我们可以看到恶意逻辑的开始。具有 Cobalt Strike 经验的分析师会立马意识到这个众所周知的`MSSE-%d-server`特征。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164428-f17cdf06-a7fd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164428-f17cdf06-a7fd-1.png)

上面代码中发生了几件事:

1. 该示例开始通过 [GetTickCount](https://docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-gettickcount) 获取 tick 计数, 然后将其除以 0x26AA。尽管获取的计数通常是时间的度量，但下一个操作仅仅将其作为随机数来使用

2. 然后该示例继续调用在`sprintf`函数周围的包装器。它的作用是格式化字符串为`PipeName`的缓冲区。如可以观察到，格式化字符串将是`\\.\pipe\MSSE-%d-server`其中`%d`为前面的除法计算的结果。(例如:`\\.\pipe\MSSE-1234-server`)。这个管道格式是有据可查的 Cobalt Strike 威胁指标。

3. 通过在全局变量定义管道的名称，恶意代码将创建一个新线程以运行`WriteBufferToPipeThread`. 此函数使我们接下来即将分析的。

4. 最后，在新的线程运行时, 代码跳转到`PipeDecryptExec`例程。

到目前为止, 我们有了线性的从 DllMain 入口点到`DecryptBufferAndExec`函数的执行过程，我们可以绘制如下流程:

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164722-59062736-a7fe-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164722-59062736-a7fe-1.png)

如我们所见，两个线程现在将同时运行。让我们集中于其中写内容到管道 (`WriteBufferToPipeThread`) 的线程，其次是与之对应`PipeDecryptExec`的内容。

WriteBufferToPipe 线程
--------------------

写入生成的管道的线程是在`DecryptBufferAndExec`没有任何其他参数的情况下启动的。通过进入该函数，我们可以观察到它只是一个`WriteBufferToPipe`前的简单的包装器，此外传递如下从全局`Payload`变量 (。(由`pPayload`指针指向)) 恢复的参数。

1.shellcode 的大小, 存储在 offset=0x4 处

2. 指向包含加密 shellcode 缓冲区的指针，该缓冲区存储在 offset=0x14 处

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164740-63b4e500-a7fe-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164740-63b4e500-a7fe-1.png)

在`WriteBufferToPipe`函数中，我们可以注意到代码是通过创建新管道开始的。管道的名称是从`PipeName`全局变量中恢复的，如果您还记得的话，该全局变量先前是由`sprintf`函数填充的。

代码创建了单个实例，通过调用 [CreateNamedPipeA](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createnamedpipea) 导出管道 ((`PIPE_ACCESS_OUTBOUND`)), 然后通过调用 [ConnectNamedPipe](https://xz.aliyun.com/t/ConnectNamedPipe) 将其连接到该实例。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164804-720cde6e-a7fe-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164804-720cde6e-a7fe-1.png)

如果连接成功,`WriteBufferToPipe`函数只要有 shellcode 的字节需要写入到管道就继续循环调用`WriteFile` 来实现写入。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164842-8918a8ea-a7fe-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164842-8918a8ea-a7fe-1.png)

值得注意的一个重要细节是, 一旦 shellcoode 写到了管道中, 先前打开管道的句柄就通过 [CloseHandle](https://docs.microsoft.com/en-us/windows/win32/api/handleapi/nf-handleapi-closehandle) 来关闭。这表明管道的唯一目的就是为了传输加密的 shellcode。

一旦`` `WriteBufferToPipe`` ` 函数执行完毕, 线程终止。总体而言，执行流程非常简单，可以如下绘制:

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164852-8f23d926-a7fe-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164852-8f23d926-a7fe-1.png)

PipeDecryptExec 流程
------------------

作为快速恢复部分, 该`PipeDecryptExec`流程在创建`WriteBufferToPipe`线程后被立即执行。执行的一个任务是分配一个内存区域，接收要通过命名管道传输的 shellcode。为此，将存储在全局 Payload 变量偏移 0x4 处的 shellcode 大小作为参数来执行 [malloc](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/malloc?view=msvc-160) 调用。

一旦缓冲区分配完成，代码将休眠 1024 毫秒 (0x400), 并将缓冲区位置和大小作为参数调用`FillBufferFromPipe`。如果该函数调用失败则返回`FALSE` (`0`), 那么代码会再次循环至该 [Sleep](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-sleep) 调用，并再次尝试操作，直到操作成功为止。这些调用和循环是必需的, 因为多线程示例必须等待至 shellcode 写入到了管道中。

一旦将 shellcode 写入到了分配的缓冲区中，`PipeDecryptExec`最终会通过`XorDecodeAndCreateThread`来解密并执行 shellcode。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164911-9a036000-a7fe-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164911-9a036000-a7fe-1.png)

要将加密的 shellcode 从管道传输到分配的缓冲区中, `FillBufferFromPipe`通过 [CreateFileA](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea) 以只读的方式 (`GENERIC_READ`)) 打开管道. 就像创建管道一样，从全局`PipeName`变量中获取名称。如果访问管道失败，则函数将继续返回`FALSE` (`0`), 而导致上述`Sleep`并重试的循环。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428164941-abe4fa18-a7fe-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428164941-abe4fa18-a7fe-1.png)

一旦管道以只读模式打开，`FillBufferFromPipe`函数接着会继续复制 shellcode 直到分配缓冲区使用 ReadFile 填满为止。缓冲区填满后，`CloseHandle`关闭命名管道的句柄, 并且`FillBufferFromPipe`函数返回`TRUE` (`1`).

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165259-21cb9034-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165259-21cb9034-a7ff-1.png)

一旦`FillBufferFromPipe`成功完成，命名管道已经完成了它的任务和加密的 shellcode 已经从一个存储区域移动到另一个。

回到调用者`PipeDecryptExec`函数中，一旦`FillBufferFromPipe`函数调用返回`TRUE`，`XorDecodeAndCreateThread`将使用以下参数进行调用：

1.  包含复制的 shellcode 的缓冲区。
2.  shellcode 的长度，存储在全局`Payload`变量的 offset=`0x4`处。
3.  对称 XOR 解密密钥，存储在全局`Payload`变量的 offset=`0x8`处。

调用后，该`XorDecodeAndCreateThread`函数首先使用 [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) 分配另一个内存区域。分配的区域具有读 / 写权限 ([PAGE_READWRITE](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants)), 但不能执行。通过不同时具有可写和可执行权限，这个示例可能是尝试躲避一些只寻找 [PAGE_EXECUTE_READWRITE](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants) 区域的安全解决方案。

一旦这个区域被分配, 函数就会在 Shellcode 缓冲区上循环并使用简单的 xor 操作将每个字节解密

分配到新的内存区域中。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165315-2bd3ad3c-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165315-2bd3ad3c-a7ff-1.png)

解密完成后，`GetModuleHandleAndGetProcAddressToArg`函数被调用。它的作用放置指向两个有用的函数指针到内存:`GetModuleHandleA` and `GetProcAddress`。这些函数能够允许 shellcode

解析其他过程，而不必依赖于它们的导入。在存储这些指针之前，该`GetModuleHandleAndGetProcAddressToArg`函数首先确保特定值不是`FALSE`（`0`）。令人惊讶的是，存储在全局变量（此处称为`zero`）中的该值始终为`FALSE`，因此指针从未被存储。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165327-32aaf84a-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165327-32aaf84a-a7ff-1.png)

回到调用者函数，`XorDecodeAndCreateThread`使用`VirtualProtect`更改 shellcode 的存储区域为可执行权限（`PAGE_EXECUTE_READ`），最终创建一个新线程。该线程从`JumpToParameter`函数开始，该函数充当 shellcode 的简单包装，shellcode 作为参数提供。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165339-3a2dc994-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165339-3a2dc994-a7ff-1.png)

从这里开始，执行先前的加密 Cobalt Strike shellcode stager ，解析 [WinINet](https://docs.microsoft.com/en-us/windows/win32/wininet/about-wininet) 过程, 下载最终的信标然后执行它。我们不会在这篇文章中介绍 shellcode 的分析，因为它应该用一篇专属的文章来分析。

尽管最后一个流程包含了更多的分支和逻辑，但总体流程图仍然非常简单。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165355-436e2292-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165355-436e2292-a7ff-1.png)

内存流分析
-----

在上述分析中，最令人惊讶的是存在一个众所周知的命名管道。通过在管道出口处解密 shellcode 或进行进程间通信，可以将管道用作防御逃避机制。

但在我们的案例中，它只是充当 [memcpy](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/memcpy-wmemcpy?view=msvc-160) 将加密的 Shellcode 从 DLL 移至另一个缓冲区的作用。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165413-4e513e56-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165413-4e513e56-a7ff-1.png)

那么为什么要使用这种开销呢？ 正如另以为同事指出，答案在于 Artifact Kit，这是 Cobalt Strike 的依赖项：

> Cobalt Strike uses the Artifact Kit to generate its executables and DLLs. The Artifact Kit is a source code framework to build executables and DLLs that evade some anti-virus products. […] One of the techniques [see: `src-common/bypass-pipe.c` in the Artifact Kit] generates executables and DLLs that serve shellcode to themselves over a named pipe. If an anti-virus sandbox does not emulate named pipes, it will not find the known bad shellcode.
> 
> [cobaltstrike.com/help-artifact-kit](https://www.cobaltstrike.com/help-artifact-kit)

正如我们在上图中所看到的, 在`malloc`缓冲区加密 shellcode 的 stageing 为了躲避检测会产生大量开销。`XorDecodeAndCreateThread`直接从初始加密的 Shellcode 中读取则可以避免这些操作。如下图所示，避免使用命名管道将进一步消除对循环`Sleep`调用的需求，因为数据将随时可用。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165432-59943cd2-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165432-59943cd2-a7ff-1.png)

看来我们找到了一种减少得到 shellcode 时间的方法。但是流行的防病毒解决方案是否被命名管道所欺骗？

修补执行流程
------

为了检验该推测， 让我们改进恶意执行流程。对于初学者，我们可以跳过与管道毋庸的调用，而直接在`DllMainThread`函数调用`PipeDecryptExec`，从而绕过管道的创建和编写。汇编级的修补方式执行过程超出了本文的讨论范围，这里我们仅感兴趣于流程的抽象。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165503-6bffa956-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165503-6bffa956-a7ff-1.png)

该`PipeDecryptExec`功能还需要打补丁以跳过`malloc`分配 \ 读取管道，并确保它能够提供`XorDecodeAndCreateThread`需要的 DLL 的加密 shellcode 而不是现在不存在的重复区域。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165518-74e5dd06-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165518-74e5dd06-a7ff-1.png)

修补我们的执行流程后，如果安全解决方案将这些未使用的指令作为检测基础，则我们可以将其都清空。

应用补丁后， 们最终得到了一条线性且较短，直到执行 Shellcode 的路径。下图专注于此修补路径，不包括下面的分支`WriteBufferToPipeThread.`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165533-7da8345c-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165533-7da8345c-a7ff-1.png)

由于我们也弄清楚了 shellcode 如何进行加密（我们拥有`xor`密钥）一样，我们修改了两个示例以修改 C2 的地址，因为它可用于识别我们的目标客户。

为确保 shellcode 不依赖任何绕过的调用，我们启动了一个快速的 Python HTTPS 服务器，并确保经过编辑的 domain 解析为`127.0.0.1`。然后，我们可以通过 [rundll32.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/rundll32) 调用原始 DLL 和修补的 DLL，并观察 shellcode 如何去试图检索 Cobalt Strike Beacon，以证明我们的补丁程序没有影响到 shellcode。我们调用的导出的`StartW` 函数是在`Sleep`调用周围的简单包装。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165602-8f4572e2-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165602-8f4572e2-a7ff-1.png)

防病毒审查
-----

那么，命名管道实际上是否可以用作防御逃避机制？尽管有有效的方法来衡量补丁程序的影响（例如：比较多个沙盒解决方案），但 VirusTotal 确实提供了快速的初步评估。因此，我们向 VirusTotal 提交了以下版本重新编辑 C2 的样本：

*   `wpdshext.dll.custom.vir` 这是经过编辑的 Cobalt Strike DLL。
*   `wpdshext.dll.custom.patched.vir` 这是我们未命名的补丁和编辑过的 Cobalt Strike DLL。

由于原始的 Cobalt Strike 包含可识别的特征（命名管道），因此我们希望修补后的版本具有较低的检测率，即使 Artifact Kit 不这样认为。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165627-9ded981a-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165627-9ded981a-a7ff-1.png)

正如我们预期的那样，Cobalt Strike 所利用的命名管道开销实际上是作为检测基础。从以上截图中可以看出，原始版本（左）仅获得 [17 次检测](https://www.virustotal.com/gui/file/a01ebc2be23ba973f5393059ea276c245e6cea1cd1dc3013548c059e810b83e6/detection)，而修补版本（右）获得的共 [16 次检测](https://www.virustotal.com/gui/file/e9dc6d7ac7659e99d2149f4ee5f6fb9fb5f873efd424d5f5572d93dee7958346/detection)少了一个。在给出的解决方案中，我们注意到 ESET 和 Sophos 未能检测到无管道版本，而 ZoneAlarm 无法识别原始版本。

一个值得注意的观察结果是，一个适配流程处于中间过程的补丁，但未对未使用的代码清 0，结果发现它是检测到最多的版本，总共有 [20 个匹配](https://www.virustotal.com/gui/file/f2458d8d9c86a8cb4a5ef09ad4213419f70728f69f207464c4b3c423ba7ae3c4/detection)。出现更高检测率是因为此修补程序允许不认识管道的防病毒提供商也仍然可以使用与管道相关的操作签名去定位 shellcode。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210428165653-ad86751c-a7ff-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210428165653-ad86751c-a7ff-1.png)

尽管这些测试针对的是默认的 Cobalt Strike 缺乏命名管道的行为。但有人可能会争辩，定制的命名管道模式将具有最好的效果。尽管我们在最初的测试中没有想到这种变体，但我们在次日提交了一个版本 (改变了管道名称为`NVISO-RULES-%d`而不是`MSSE-%d-server`), 然后获得了 [18 个检测结果](https://www.virustotal.com/gui/file/5f2b3f855ffb78d91fc2e35377f50c579d31956bf0e39d97e36fbec968fdb7aa/detection)。作为比较，我们另外两个样本的检测率在一夜之间增加到 30+。但是，我们必须考虑这 18 个检测结果受初始 shellcode 的影响。

结论
--

事实证明，逆向恶意的 Cobalt Strike DLL 比预期的要有趣。总体而言，我们注意到存在嘈杂的操作，这些操作的使用不是功能要求，甚至可以充当检测基础。为了证实我们的假设，我们修补了执行流程，并观察了简化版本如何以较低的检测率（几乎未更改）到达 C2 服务器。

因此，这个分析为什么很重要？

蓝队
--

首先，最重要的是，这个载荷的分析突出显示了常见的 Cobalt Strike DLL 特征，使我们可以进一步微调检测规则。虽然这个 Stager 是第一个被分析的 DLL, 但我们确实寻找了其他 Cobalt Strike 格式，比如默认信标和可以用的[可延展的 C2](https://www.cobaltstrike.com/help-malleable-c2)，包括动态链接库和可移植的可执行文件。出乎意料的是，所有格式都共享了这个常见的[文档记录的](https://blog.cobaltstrike.com/2021/02/09/learn-pipe-fitting-for-all-of-your-offense-projects/)`MSSE-%d-server`的管道名称，并且对[开源检测规则](https://grep.app/search?q=MSSE-&case=true)的快速[搜索](https://grep.app/search?q=MSSE-&case=true)显示了它被寻找的东西很少。

红队
--

除了对 NVISO 的防御行动有所帮助外，这项研究还使我们的进攻团队在选择使用定制交付机制方面感到更加安慰。更重要的是，遵循我们记录的设计选择。在针对成熟环境的操作中使用命名管道更有可能引发危险信号，并且到目前为止，至少在不更改生成方式的情况下，似乎仍无法提供任何可逃避的优势。

对于下一个针对我们客户的参与者: 我期待着修改您的样本并测试更改后的管道名称的有效性。

> 译者注: 修改管道名称还是蛮有用的...
> 
> 本文未翻译文章，原文链接:[https://blog.nviso.eu/2021/04/26/anatomy-of-cobalt-strike-dll-stagers/](https://blog.nviso.eu/2021/04/26/anatomy-of-cobalt-strike-dll-stagers/)