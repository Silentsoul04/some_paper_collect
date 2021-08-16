> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8103)

木马是黑客实施网络攻击的常用兵器之一，有些木马可以通过免杀技术的加持躲过杀毒软件的查杀。本文由锦行科技的安全研究团队提供（作者：t43M!ne），旨在通过剖析 CS 木马生成过程以及开发免杀工具，帮助大家更好地理解 CS 木马的 Artifact 生成机制。

什么是 Cobaltstrike？
-----------------

Cobaltstrike 是用于红队行动、APT 攻击模拟的软件，它具备很强大的协同能力和难以置信的可扩展性。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100155-f36d9fc4-e676-1.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100155-f36d9fc4-e676-1.jpg)  
无论是编写 shellcode，创建自定义的 C2 二进制可执行文件，还是修改代码来隐藏恶意程序，它们都是红队日常工作的一部分，阅读和理解成熟的 C2 框架代码也是理所当然的事情。

CobaltStrike 如何生成 ShellCode？
----------------------------

CS 是使用 Swing 进行 UI 开发的，代码中直接找对话框对应操作类。  
aggressor\dialogs\WindowsExecutableDialog.class  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100155-f3bdca12-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100155-f3bdca12-e676-1.png)

可以看到很清晰的生成逻辑。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100156-f3e15086-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100156-f3e15086-e676-1.png)

通过 DialogUtils.getStager() 获得生成的 stager 然后通过 saveFile 保存文件。

getStager() 方法调用了 aggressor\DataUtils.shellcode() ，而这里其实是 Stagers 的接口：

return Stagers.shellcode(s, "x86", b);

最终在 stagers\Stagers.shellcode() 根据监听器类型，

实例化了继承自的 GenericStager 的 stagers\GenericHTTPStager 类，并由 generate() 生成 shellcode  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4838036-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4838036-e676-1.png)

shellcode 生成时，读取了 resources/httpstager.bin，并根据监听器的 host 和 port 等值组合为 Packer，最终替换到多个 X、Y 占位的 bin 文件中，最后返回 bytes[] 类型的 shellcode

Patch Artifact
--------------

shellcode 生成完成后，回到原点，可以看到根据用户的选择，对不同的 artifact 模板进行 patch，以 x86 的模板为例。  
继续跟进 patchArtifact  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4a78904-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4a78904-e676-1.png)

common\BaseArtifactUtils.class  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4cb20c6-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4cb20c6-e676-1.png)

稍微看一下 fixChecksum，是通过 PE 编辑器修复了校验码。  
这里不赘述了，对编辑器实现感兴趣的可以去看看 pe\PEEditor.class  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4eb21be-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100157-f4eb21be-e676-1.png)

注意看这里 this._patchArtifact(array, s)，调用了同名方法，PS：差点以为在看 Python  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100158-f52d23d4-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100158-f52d23d4-e676-1.png)

读取了 resources 文件夹下的 artifact32.exe 作为模板文件，根据重复的 1024 个 A 来定位 shellcode 位置。

与生成 shellcode 时类似，使用 common/CommonUtils.replaceAt() 对 bytes 流转为的字符串进行编辑替换。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100158-f54e0cf2-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100158-f54e0cf2-e676-1.png)

使用 16 进制编辑器可以直接看到用于标志存放 shellcode 的位置。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100158-f581cf06-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100158-f581cf06-e676-1.png)

值得一提的是，替换 shellcode 之后的 pe 文件，因为 shellcode 长度没有完全覆盖到标识的 1024 个 A，一般生成的 exe 都会残留部分字符，当然这并不会影响 shellcode 的执行。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f5b68f66-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f5b68f66-e676-1.png)

Shellcode Launcher
------------------

利用加载器远程回连获取下一阶段 payload 加载到内存中执行以规避杀软的探测，这种 VirtualAlloc 到 WriteProcessMemory 的分配内存模式早已被众多远控木马软件广泛利用。

CS 开发者在其最新的介绍视频中披露了部分 artifact 的源码，并演示了如何通过修改加载器绕过了 Defender 的查杀。

他通过用 HeapAlloc 来代替 VitualAlloc，躲避了大部分的杀软。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f5e08c4e-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f5e08c4e-e676-1.png)

在这个基础上，我们添加了对 shellcode 进行异或加密的功能，显然一个非常精简的基于 c++ 的 shellcode 加载器就成形了。

然后参考 CS 的方式，在本应放置 shellcode 的 buf 中，置入大量重复的占位符作为定位。  
python -c "print(1024*'A')"  
用 VisualStudio 或 MingW 将其编译为 template.exe。

开发免杀小工具
-------

新建一个 JavaFx 的项目，样式与部分代码参考某 chaos 免杀小助手。

捋下流程，首先需要对 CS 或 MSF 的 shellcode 进行预处理，然后进行异或加密，读取模板文件，定位到 shellcode 位置，进行覆盖，最后保存。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f600360c-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f600360c-e676-1.png)

有很多类直接可以从 CS 复制过来就能用。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f629f532-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100159-f629f532-e676-1.png)  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100200-f652bfd0-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100200-f652bfd0-e676-1.png)  
重点看下 xor，为了跟 launcher 解密一致，需要先转换为 int 类型进行异或，然后再转回 hex，最终打包为 jar  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100200-f6835f28-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100200-f6835f28-e676-1.png)

生成 veil 类型的 payload，复制粘贴，生成，保存。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100200-f6b404c0-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100200-f6b404c0-e676-1.png)  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20200825100201-f6efe9ae-e676-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200825100201-f6efe9ae-e676-1.png)  
最终免杀效果取决于 Launcher 模板，作为一个非常精简、没什么改动的模板，效果已经出乎意料了。

毕竟目的并非追求免杀效果，而应注重于理解 CS 木马的 Artifact 生成机制。