\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/4eBDfFtAmQJAJdFaPzIezw)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25Xca8f96PMSJFk6CNeiaTRb5O4wZcJzPREpjXWaiccqmdpNuNBuAGqhcjQ/640?wx_fmt=png)

最近几次测试 APP 时，遇到过几次非 http/https 通信的情况，burp、fiddler 等 http 代理工具都无法正常抓到包，经过分析发现 app 是通过 socket 通信的，所以写出来记录下。

### socket 与 websocket

先来区别下 socket 与 websocket，因为我们在使用 burpsuite 和 fiddler 时，发现 burp 和 fiddler 都是可以抓 websocket 的，所以有必要先区别一下，从本质上来说二者关系并不大，甚至说没啥关系，盗用一张图来说明下二者关系，读者可自行百度、谷歌检索二者关系。

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XFMicw6RfHm0lIZTvPHXojETz9Xqftj4JT4FuDpicBicnJCibKaogib8VqUg/640?wx_fmt=png)

### socket 抓包思路

为了方便理解，我们自己可以实现一个简单的通过 socket 通信的 APP 和与之其对应的 Server，实现一个简单功能，客户端 APP 发送 socket 消息，模拟平时项目中 APP 调用 socket 相关接口通信，同时接收服务端下发的 socket 消息，客户端 APP 运行如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XU9qG9vW0slY0N273xkQWW7Eu4JFr6X992dznibRjFq8pUklibQicC2CzA/640?wx_fmt=png)

服务端通过 ServerSocket 构造器实现 socket 监听绑定即可，运行如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XhIVIZRpXYyPFCbXZU4iaWb5ib23uuNAic5O3EWFxpBKD515XiaTst3oyAw/640?wx_fmt=png)

以上，就简单实现了一个通过 socket 通信的 c/s，通过这种方式发送的数据包，burp 和 fiddler 之类的代理工具是无法抓到的，因为他们本来就属于 http/https/websocket 代理工具，对 socket 是无能为力的，所以我们需要换些思路。

#### tcpdump+wireshark

这种方式抓包非常通用，不光针对 socket 方式，http/https 等等也是可以的，因为这些两种抓包工具都是直接对流经网卡的数据包进行捕获，不存在区别信息传递使用什么协议，可以通过 tcpdump 将数据包保存成 pcap 格式，然后用 wireshark 打开进行分析，来看下用 tcpdump 抓到的数据包：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XOYabYoBltuXr40jcgNw35mqfqJMJKXg2QxPziaicQficQUwNUplVia3MbA/640?wx_fmt=png)

客户端发送的数据包

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XnCv6L40icDt4Y6zHe94VIKWJOzfocnKeI6ym1oY9ZfEdQqMTFnnjVAA/640?wx_fmt=png)

服务端接收的数据包

tcpdump 显示数据包格式不是很友好，导入到 wireshark 或者可用 wireshark 直接抓包，分析起来就比较容易了，可以看到数据传输是通过 socket 传输的：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XdyytGJFlS4w60dzrga4MZGrOP3vQEEtpjPN3sqvLGBWQJAzk84Hshw/640?wx_fmt=png)

#### hook 方式抓包

上述方法虽然抓包很好，但是对于渗透测试来说，我们不仅仅要看到数据包内容，更重要的是还能修改数据包，所以这里还可以使用 hook 方式抓包，在实现 socket 通信的过程，客户端 (基于 android6.0 系统) 发送消息需要会调用`java.io.OutputStream.write()` 方法，也有可能是 `java.io.PrintWriter.write()` 方法，这里以`java.io.OutputStream.write` 方法为例，这个方法有三个重载，分别是 `write(byte[] buffer)`、`write(int oneByte)`、`write(byte[] buffer, int offset, int count`)，通过去阅读 android 源码，可以发现，三者调用关系为 `write(byte[] buffer)-> write(byte[] buffer, int offset, int count)-> write(int oneByte)`，这里直接把源码粘贴过来，方便大家看：

```
/\*
 \* Licensed to the Apache Software Foundation (ASF) under one or more
 \* contributor license agreements. See the NOTICE file distributed with
 \* this work for additional information regarding copyright ownership.
 \* The ASF licenses this file to You under the Apache License, Version 2.0
 \* (the "License"); you may not use this file except in compliance with
 \* the License. You may obtain a copy of the License at
 \*
 \*   http://www.apache.org/licenses/LICENSE-2.0
 \*
 \* Unless required by applicable law or agreed to in writing, software
 \* distributed under the License is distributed on an "AS IS" BASIS,
 \* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 \* See the License for the specific language governing permissions and
 \* limitations under the License.
 \*/
package java.io;
import java.util.Arrays;
/\*\*
 \* A writable sink for bytes.
 \*
 \* <p>Most clients will use output streams that write data to the file system
 \* ({@link FileOutputStream}), the network ({@link java.net.Socket#getOutputStream()}/{@link
 \* java.net.HttpURLConnection#getOutputStream()}), or to an in-memory byte array
 \* ({@link ByteArrayOutputStream}).
 \*
 \* <p>Use {@link OutputStreamWriter} to adapt a byte stream like this one into a
 \* character stream.
 \*
 \* <p>Most clients should wrap their output stream with {@link
 \* BufferedOutputStream}. Callers that do only bulk writes may omit buffering.
 \*
 \* <h3>Subclassing OutputStream</h3>
 \* Subclasses that decorate another output stream should consider subclassing
 \* {@link FilterOutputStream}, which delegates all calls to the target output
 \* stream.
 \*
 \* <p>All output stream subclasses should override both {@link
 \* #write(int)} and {@link #write(byte\[\],int,int) write(byte\[\],int,int)}. The
 \* three argument overload is necessary for bulk access to the data. This is
 \* much more efficient than byte-by-byte access.
 \*
 \* @see InputStream
 \*/
public abstract class OutputStream implements Closeable, Flushable {
  /\*\*
   \* Default constructor.
   \*/
  public OutputStream() {
  }
  /\*\*
   \* Closes this stream. Implementations of this method should free any
   \* resources used by the stream. This implementation does nothing.
   \*
   \* @throws IOException
   \*       if an error occurs while closing this stream.
   \*/
  public void close() throws IOException {
    /\* empty \*/
  }
  /\*\*
   \* Flushes this stream. Implementations of this method should ensure that
   \* any buffered data is written out. This implementation does nothing.
   \*
   \* @throws IOException
   \*       if an error occurs while flushing this stream.
   \*/
  public void flush() throws IOException {
    /\* empty \*/
  }
  /\*\*
   \* Equivalent to {@code write(buffer, 0, buffer.length)}.
   \*/
  public void write(byte\[\] buffer) throws IOException {
    write(buffer, 0, buffer.length);
  }
  /\*\*
   \* Writes {@code count} bytes from the byte array {@code buffer} starting at
   \* position {@code offset} to this stream.
   \*
   \* @param buffer
   \*      the buffer to be written.
   \* @param offset
   \*      the start position in {@code buffer} from where to get bytes.
   \* @param count
   \*      the number of bytes from {@code buffer} to write to this
   \*      stream.
   \* @throws IOException
   \*       if an error occurs while writing to this stream.
   \* @throws IndexOutOfBoundsException
   \*       if {@code offset < 0} or {@code count < 0}, or if
   \*       {@code offset + count} is bigger than the length of
   \*       {@code buffer}.
   \*/
  public void write(byte\[\] buffer, int offset, int count) throws IOException {
    Arrays.checkOffsetAndCount(buffer.length, offset, count);
    for (int i = offset; i < offset + count; i++) {
      write(buffer\[i\]);
    }
  }
  /\*\*
   \* Writes a single byte to this stream. Only the least significant byte of
   \* the integer {@code oneByte} is written to the stream.
   \*
   \* @param oneByte
   \*      the byte to be written.
   \* @throws IOException
   \*       if an error occurs while writing to this stream.
   \*/
  public abstract void write(int oneByte) throws IOException;
  /\*\*
   \* Returns true if this writer has encountered and suppressed an error. Used
   \* by PrintStreams as an alternative to checked exceptions.
   \*/
  boolean checkError() {
    return false;
  }
}

```

所以理论上我们去 hook `write(byte[] buffer)` 这个方法就可以了，这个 hook 代码代码非常简单，这里就不做展示了，可以看看 hook 结果：  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XYOycUzibX4zxMdABKTMuSofsGfcsWF9G4NicbkHOvPe493wics81YXQ1A/640?wx_fmt=png)

到这里，能够 hook 到，就可以按照我们的需求来修改数据包了，当然，我们也需要找一个 APP 来实战下，在市场上的 APP 是否真的有效。

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XTRFMPT3HUCKSlr4ySAsHYMyFhdfS52ZJKzOv5gr2xZzzrOPbRyEqMQ/640?wx_fmt=png)

#### objection

前一篇文章讲 objection 的使用，这里正好可以用 objection watch 一下 `java.io.OutputStream` 输出流，发现这里面有 close、flush、write 三个方法

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XPO3ia8RcFEmsuJDs38giaIZBHJoq1oyic7DZ3icVQVgxxNZjibpEDx7Bs0A/640?wx_fmt=png)

Watch 一下 write 方法，观察 app 在处理业务时，是否有该方法的调用

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25Xq8BR25ibYm9y4YAXAHTPToHdlLiaIPVAGupBmHJiazjDvG2LbK6LdJWVA/640?wx_fmt=png)

通过 objection 对 write 方法的跟踪，发现确实 socket 通信调用了 write 方法，而且通过堆栈信息，我们还发现了疑似发送数据包的方法，send、request，这里尝试 hook send 方法，发现果然是有用的，输出信息如下：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffBElb1k90SpRULMnbwm25XerAFvs8HeAG83Hq9hdUo1KqvvegFXcEqvRbiaXR9rNUUq04uVWxYCgg/640?wx_fmt=png)

综上就是最近遇到的关于 socket 抓包的一点想法和实践，虽然平时测试很少遇到 socket 通信的，但是遇到了，就需要解决不是么？不知道大佬们还有没有更好的思路，如果有，还请告诉我。

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfe3nBUEiaFQfDH9ZEHwj03Otp8ibk7CRf8nWnOVv8LfG7lc1qq4OpibSY5TT657aGhtTLdk3jpGKiaORw/640?wx_fmt=png)