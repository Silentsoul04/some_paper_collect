> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.elextec.com](https://www.elextec.com/company-c-4783.html)

> 博智安全专注于网络信息安全领域, 服务逾千家党政军, 政府机关, 企事业单位和高校客户, 获得了众多用户的肯定。

一、前言  
工业控制系统离不开上位机监控系统和下位机控制器即 PLC，上位机软件相对比较容易获得，比如本文采用 kingview6.53，但 PLC 的获得相对来说就没有那么轻松，考虑这种情况，本文借助一款模拟工业自动化环境的开源软件 OpenPLC 基于 modbus 协议实现与组态王的通讯仿真。同时基于环境进行模拟测试，对组态王的组态画面、OpenPLC Editor 梯形图简单编程进行简单介绍，亲测效果不错。  
二、环境准备  
(1)Kali linux 虚拟机 (IP:192.168.180.146) 安装 OpenPLC  
安装过程参考 https://github.com/thiagoralves/OpenPLC_v3，安装后，Kali linux 虚拟机内置浏览器输入如下地址，账户 / 密码：openplc/openplc。  
  ![](https://www.elextec.com/resource/default/image/20200702/20200702132231_326.png)  
(2)Kali linux 虚拟机 (IP:192.168.180.146) 安装 OpenPLC_Editor，安装过程参考 https://github.com/thiagoralves/OpenPLC_Editor，安装后，在应用里面搜索 OpenPLC_Editor，打开后，编写简单程序如下，下载链接：openplc_test.st。

 ![](https://www.elextec.com/resource/default/image/20200702/20200702132245_967.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702132259_496.png)

   
(3)winxp sp3 虚拟机 (IP:192.168.180.157) 安装组态王 6.53，新建 test 工程下载链接 https://github.com/sxd0216/kingview--test，并按下图所示添加 OpenPLC 设备 OpenPLC_test。

 ![](https://www.elextec.com/resource/default/image/20200702/20200702132622_167.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702132632_616.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702132641_125.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702132652_393.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702132701_302.png)

   
(4) 设定变量并和 OpenPLC_test 连接

 ![](https://www.elextec.com/resource/default/image/20200702/20200702132856_746.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702132925_227.png)

   
(5) 组态简单画面，点击 Start，电机运行，点击 Stop，电机停止  
 ![](https://www.elextec.com/resource/default/image/20200702/20200702132939_257.png)  
三、仿真通讯  
(1)Kali linux 虚拟机 (IP:192.168.180.146) 中运行 OpenPLC，导入 OpenPLC_Editor 编辑好的程序 openplc_test.st。

 ![](https://www.elextec.com/resource/default/image/20200702/20200702133010_404.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702133039_20.png)

   
待程序编译好后，Go to Dashboard，然后 Start PLC

 ![](https://www.elextec.com/resource/default/image/20200702/20200702133126_990.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702133146_120.png)

   
待 PLC 出现 Running 后，进入 Monitoring

 ![](https://www.elextec.com/resource/default/image/20200702/20200702133201_325.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702133430_936.png)

   
(2)winxp sp3 虚拟机 (IP:192.168.180.157) 中运行 test 工程

 ![](https://www.elextec.com/resource/default/image/20200702/20200702133458_19.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702133512_770.png)

   
(3) 通过信息窗口查看，已经通讯成功  
 ![](https://www.elextec.com/resource/default/image/20200702/20200702133525_719.png)  
(4) 点击 Start 后，指示灯亮，电机启动，进入 PLC，发现 Start 和 MV1 变量值已经变为 TRUE，对比图如下：  
  ![](https://www.elextec.com/resource/default/image/20200702/20200702133555_69.png)  ![](https://www.elextec.com/resource/default/image/20200702/20200702133607_615.png)  
   
(5) 点击 Stop 后，指示灯灭，电机停止，进入 PLC，发现 Start 和 MV1 变量值已经变为 FALSE,Stop 变量值变为 TRUE，对比图如下：  
  ![](https://www.elextec.com/resource/default/image/20200702/20200702133630_554.png)  
   
四、模拟测试  
(1)winxp sp3 虚拟机 (IP:192.168.180.157) 中利用 wireshark 抓取 03. 仿真通讯中 Start(14 帧、16 帧)和 Stop(19 帧、21 帧)的数据包，下载链接 https://github.com/sxd0216/attack-packets。

 ![](https://www.elextec.com/resource/default/image/20200702/20200702133737_510.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702133749_83.png)

   
查看数据包，搜索 modbus 协议”05″功能码 Write Coil，捕获到 Start(14 帧、16 帧) 和 Stop(19 帧、21 帧) 的攻击数据包，modbus 协议常用功能码如下：  
01 ：读取线圈状态  
02：读取输入状态  
03：保持型寄存器读取  
05：写单一线圈  
06：写单一寄存器  
(2) 基于 wireshark 捕获的 Write Coil，编写 Python 攻击包，下载链接 https://github.com/sxd0216/attack-packets。

![](https://www.elextec.com/resource/default/image/20200702/20200702133936_499.png)

![](https://www.elextec.com/resource/default/image/20200702/20200702133948_328.png)

(3)利用攻击包也可以达到 (4) 和(5)的效果  
五、总结  
本文主要利用 OpenPLC 模拟 modbus 协议实现了 OpenPLC 实现了与 Kingview 的通讯仿真与模拟测试，大家如果感兴趣也可以基于 OpenPLC 模拟其他协议，进而仿真其他环境；同时本文也对 kingview 如何建立工程、组态画面，OpenPLC Editor 编辑简单梯形图程序进行了简单介绍，希望对热爱工控的人士有所帮助。