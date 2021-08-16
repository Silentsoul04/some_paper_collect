> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/c0y6nk3U77EOjYSatpnRbw)

点击上方 网络工程师笔记，选择 设为星标

优质文章，及时送达

公众号

#### **加群交流在后台回复 “****加群****”，添加小编微信**

![图片](https://mmbiz.qpic.cn/mmbiz_png/PuI1aiczpHCMV4NbC6UMxVU5DgSpkXsEdkicsH91WjCRSUYlsuR0VJe12F56icG2GVlnuic4FUPADRkKMFTgmTkArA/640?wx_fmt=png)

**1. 常用命令视图**

| 

**常用视图**





 | 

**进入视图**





 | 

**视图功能**





 |
| 

用户视图





 | 

用户从终端成功登录至设备即进入用户视图，在屏幕上显示：

<HUAWEI





 | 

在用户视图下，用户可以完成查看运行状态和统计信息等功能。





 |
| 

系统视图





 | 

在用户视图下，输入命令 system-view 后回车，进入系统视图。

<HUAWEI> system-view

[HUAWEI





 | 

在系统视图下，用户可以配置系统参数以及通过该视图进入其他的功能配置视图。





 |
| 

接口视图



 | 

使用 interface 命令并指定接口类型及接口编号可以进入相应的接口视图。

[HUAWEI] interface gigabitethernet X/Y/Z

[HUAWEI-GigabitEthernetX/Y/Z]



 | 

配置接口参数的视图称为接口视图。在该视图下可以配置接口相关的物理属性、链路层特性及 IP 地址等重要参数。



 |
| 

路由协议视图



 | 

在系统视图下，使用路由协议进程运行命令可以进入到相应的路由协议视图。

[HUAWEI]OSPF

[HUAWEI-ospf-1]



 | 

路由协议的大部分参数是在相应的路由协议视图下进行配置的。例如 IS-IS 协议视图、OSPF 协议视图、RIP 协议视图。



 |

**2. 创建 VLAN**

<Huawei>    // 用户视图，一般 display 命令查看信息比较多。

<Huawei>system-view   // 准备进入系统视图。

[Huawei]vlan 100   // 创建 vlan 100。[Huawei-vlan100]quit   // 退回系统视图。

**3. 将端口加入到 vlan 中**

Huawei] interface

GigabitEthernet2/0/1  //(10G 光口) 

[Huawei-GigabitEthernet2/0/1]port link-

type access  // 定义端口传输模式 

[Huawei-GigabitEthernet2/0/1]portdefault vlan 100  // 将端口加入 vlan100 

[Huawei- GigabitEthernet2/0/1] quit    // 回到接口视图   

[Huawei] interface GigabitEthernet1/0/0    // 进入 1 号插槽上的第一个千兆网口接口视图中。0 代表 1 号口

[Huawei- GigabitEthernet1/0/0] port link-type access  // 定义端口传输模式 

[Huawei- GigabitEthernet2/0/1] port 

default vlan 10  // 将这个端口加入到 vlan10 中

[Huawei- GigabitEthernet2/0/1] quit 

**4. 将多个端口加入到 VLAN 中**

<Huawei>system-view   
[Huawei]vlan 10

[Huawei-vlan10]portGigabitEthernet 1/0/0 to 1/0/29  // 将 0 到 29 号口加入到 vlan10 中

 [Huawei-vlan10]quit

**5. 交换机配置 IP 地址**

[Huawei] interface Vlanif100   // 进入

vlan100 接口视图与 vlan 100 命令进入的地方不同 

[Huawei-Vlanif100] ip address 

192.168.1.1 255.255.255.0// 定义 vlan100

管理 IP 三层 交换网关路由 

[Huawei-Vlanif100] quit    // 返回视图

**6. 配置默认网关**

[Huawei]ip route-static 0.0.0.0 0.0.0.0 

192.168.1.254 // 配置默认网关。

**7. 交换机保存设置和重置命令**

<Huawei>save    // 保存配置信息 

<Huawei>reset saved-configuration// 重置交换机的配置  

<Huawei>reboot   // 重新启动交换机

**8 交换机常用的显示命令**

用户视图模式下：  

<Huawei>display current-

configuration   // 显示现在交换机正在运行的配置明细

<Huawei>display device   // 显示各设备状态 

<Huawei>display interface xxx  // 显示个端口状态，用？可以查看后边跟的选项 

<Huawei>display version   // 查看交换机固件版本信息 

<Huawei>display vlan xxx   // 查看 vlan 的配置信息

![图片](https://mmbiz.qpic.cn/mmbiz_png/PuI1aiczpHCMV4NbC6UMxVU5DgSpkXsEd6grq7KWwuCqu2TzqzrKplmuibexrlq6gqJXGSuvhNjJ3pDsTPiaPcjEg/640?wx_fmt=png)

**1. 基本配置**

<H3C>      // 用户直行模式提示符, 用户视图   

<H3C>system-view    //** 进入系统视图 ** 

[H3C] sysname xxx   // 设置主机名成为 xxx 这里使用修改特权用户密码

**2. 用户配置**

<H3C>system-view  

[H3C]super password H3C     // 设置用户分级密码

[H3C]undo superpassword     // 删除用户分级密码

[H3C]localuser bigheap 1234561   //Web 网管用户设置, 1 为管理级用户

[H3C]undo localuser bigheap // 删除 Web 网管用户

[H3C]user-interface aux 0     // 只支持 0

[H3C-Aux]idle-timeout 250     // 设置超时为 2 分 50 秒, 若为 0 则表示不超时, 默认为 5 分钟

[H3C-Aux]undoidle-timeout     // 恢复默认值

[H3C]user-interface vty 0     // 只支持 0 和 1

[H3C-vty]idle-timeout 250     // 设置超时为 2 分 50 秒, 若为 0 则表示不超时, 默认为 5 分钟

[H3C-vty]undoidle-timeout     // 恢复默认值

[H3C-vty]set authentication 

password123456     // 设置 telnet 密码, 必须设置

[H3C-vty]undo set authentication 

password   // 取消密码

[H3C]displayusers     // 显示用户

[H3C]displayuser-interface     // 用户界面状态

**3.vlan 配置**

[H3C]vlan 2         **// 创建 VLAN2**

[H3C]undo vlan all     // 删除除缺省 VLAN 外的所有 VLAN, 缺省 VLAN 不能被删除

[H3C-vlan2]port Ethernet 0/4 to 

Ethernet0/7     // 将 4 到 7 号端口加入到

VLAN2 中, 此命令只能用来加 access 端口, 不能用来增加 trunk 或者 hybrid 端口

[H3C-vlan2]port-isolate enable    // 打开 VLAN 内端口隔离特性，不能二层转发,** 默认不启用该功能 **

[H3C-Ethernet0/4]port-isolate uplink-portvlan 2    // 设置 4 为 VLAN2 的 ** 隔离上行端口 **，用于转发二层数据, 只能配置一个上行端口, 若为 trunk, 则建议允许所有 VLAN 通过, 隔离不能与汇聚同时配置

[H3C]display vlan all     //** 显示所有 VLAN 的详细信息 **

[H3C]user-group 20     // 创建 user-group 20，默认只存在 user-group 1

[H3C-UserGroup20]port Ethernet 0/4 toEthernet 0/7     //** 将 4 到 7 号端口加入到 VLAN20 中，** 初始时都属于 user-group 1 中

[H3C]display user-group 20     // 显示 user-group 20 的相关信息

**4. 交换机 IP 配置**

[H3C]vlan 20        //** 创建 vlan**

[H3C]management-vlan 20     // 管理 vlan

[H3C]interface vlan-interface 20      //** 进入并管理 vlan20**

[H3C]undo interface vlan-interface 20      // 删除管理 VLAN 端口

[H3C-Vlan-interface20]ip address192.168.1.2 255.255.255.0    //** 配置管理 VLAN 接口静态 IP 地址 **

[H3C-Vlan-interface20]undo ipaddress      // 删除 IP 地址

[H3C-Vlan-interface20]ip gateway 192.168.1.1      // 指定缺省网关 (默认无网关地址)

[H3C-Vlan-interface20]undo ip gateway

[H3C-Vlan-interface20]shutdown   //** 关闭接口 **

[H3C-Vlan-interface20]undo shutdown  // 开启

[H3C]display ip      // 显示管理 VLAN 接口 IP 的相关信息

[H3C]display interface vlan-interface20      // 查看管理 VLAN 的接口信息

<H3C>debugging ip      // 开启 IP 调试功能

<H3C>undo debugging ip

**5.DHCP 客户端配置**

[H3C-Vlan-interface20]ip address dhcp-alloc     // 管理 VLAN 接口 ** 通过 DHCP 方式获取 IP 地址 **  

[H3C-Vlan-interface20]undo ip address dhcp-alloc     // 取消

[H3C]display dhcp     // 显示 DHCP 客户信息 <H3C>debugging dhcp-alloc      // 开启 DHCP 调试功能

<H3C>undo debugging dhcp-alloc

**6. 端口配置**

[H3C]interface Ethernet0/3     // 进入端口

[H3C-Ethernet0/3]shutdown    // 关闭端口

[H3C-Ethernet0/3]speed 100      // 速率可为 10,100,1000 和 auto(缺省)

[H3C-Ethernet0/3]duplexfull      //** 双工, 可 ** 为 half,full 和 auto，光口和汇聚后不能配置

[H3C-Ethernet0/3]flow-control     //** 开启流控，默认为关闭 **

[H3C-Ethernet0/3]broadcast-suppression 20      // 设置抑制广播百分比为 20%, 可取 5,10,20,100, 缺省为 100, 同时组播和未知单播也受此影响

[H3C-Ethernet0/3]loopback internal      // 内环测试

[H3C-Ethernet0/3]port link-type trunk    // 设置链路的 ** 类型为 trunk**

[H3C-Ethernet0/3]port trunk pvid vlan 20      // 设置 20 为该 trunk 的缺省 VLAN，默认为 1(trunk 线路两端的 PVID 必须一致)

[H3C-Ethernet0/3]port access vlan 20      // 将当前 **access 端口加入指定的 VLAN**  

[H3C-Ethernet0/3]port trunk permit vlan all      // 允许 ** 所有的 VLAN 通过当前的 trunk 端口,** 可多次使用该命令  

[H3C]link-aggregation Ethernet 0/1 to Ethernet 0/4      //** 将 1-4 口加入汇聚组,**1 为主端口, 两端需要同时配置, 设置了端口镜像以及端口隔离的端口无法汇聚  

[H3C]undo link-aggregation Ethernet 0/1   // 删除该汇聚组

[H3C]link-aggregation mode egress 

// 配置端口汇聚模式为根据目的 MAC 地址进行负荷分担, 可选为 ingress,egress 和 both, 缺省为 both

[H3C]monitor-port Ethernet 0/2      //** 将该端口设置为镜像端口 **, 必须先设置镜像端口, 删除时必须先删除被镜像端口, 而且它们不能同在一个端口, 该端口不能在汇聚组中, 设置新镜像端口时, 新取代旧, 被镜像不变

[H3C]mirroring-port Ethernet 0/3 toEthernet 0/4 both      // 将 ** 端口 3 和 4 设置为被镜像端口 **,both 为同时监控接收和发送的报文, inbound 表示仅监控接收的报文, outbound 表示仅监控发送的报文

[H3C]display mirror

[H3C]display interface Ethernet 0/3

<H3C>resetcounters      //** 清除所有端口的统计信息 **  

![图片](https://mmbiz.qpic.cn/mmbiz_png/PuI1aiczpHCMV4NbC6UMxVU5DgSpkXsEdCEDJp0QEaBr8M3g5yQyAyLcKYIENBbQb9tqLM3kuX3IibH0dgEicX8zw/640?wx_fmt=png)

**1. 基本命令**

>Enable    // 进入特权模式  

#Exit      // 返回上一级操作模式

#End      // 返回到特权模式

#copy running-config startup-config // 保存配置文件#  

del flash:config.text  // 删除配置文件 (交换机及 1700 系列路由器)

#erase startup-config  // 删除配置文件 (2500 系列路由器)

#del flash:vlan.dat  // 删除 Vlan 配置信息（交换机）

#Configure terminal  // 进入全局配置模式

(config)# hostname switchA  // 配置设备名称为 switchA

(config)#banner motd &    // 配置每日提示信息 & 为终止符

(config)#enable secret level 1 0 star  // 配置远程登陆密码为 star

(config)#enable secret level 15 0 star  // 配置特权密码为 star

Level 1 为普通用户级别，可选为 1~15，15 为最高权限级别；0 表示密码不加密  

(config)#enable services web-server // 开启交换机 WEB 管理功能

Services 可选以下：web-server(WEB 管理)、telnet-server(远程登陆) 等

**2. 查看信息**

#show running-config    // 查看当前生效的配置信息  

#show interface fastethernet 0/3  // 查看 F0/3 端口信息

#show interface serial 1/2 // 查看 S1/2 端口信息

#show interface        // 查看所有端口信息

#show ip interface brief     // 以简洁方式汇总查看所有端口信息

#show ip interface     // 查看所有端口信息

#show version        // 查看版本信息

#show mac-address-table    // 查看交换机当前 MAC 地址表信息

#show running-config    // 查看当前生效的配置信息

#show vlan         // 查看所有 VLAN 信息

#show vlan id 10     // 查看某一 VLAN (如 VLAN10) 的信息

#show interface fastethernet 0/1  // 查看某一端口模式 (如 F 0/1)

#show aggregateport 1 summary  // 查看聚合端口 AG1 的信息

#show spanning-tree   // 查看生成树配置信息

#show spanning-tree interface 

fastethernet 0/1  // 查看该端口的生成树状态

#show port-security   // 查看交换机的端口安全配置信息

#show port-security address   // 查看地址安全绑定配置信息

#show ip access-lists listname  // 查看名为 listname 的列表的配置信息

**3. 端口基本配置**

(config)#Interface fastethernet 0/3     // 进入 F0/3 的端口配置模式  
(config)#interface rangefa 0/1-2,0/5,0/7-9   // 进入 F0/1、F0/2、F0/5、F0/7、F0/8、F0/9 的端口配置模式  

(config-if)#speed 10   // 配置端口速率为 10M, 可选 10,100,auto

(config-if)#duplex full   // 配置端口为全双工模式, 可选 full(全双工),half(半双式),auto(自适应)

(config-if)#no shutdown          // 开启该端口

(config-if)#switchport access vlan 10   // 将该端口划入 VLAN10 中, 用于 VLAN

(config-if)#switchport mode trunk   // 将该端口设为 trunk 模式, 可选模式为 access , trunk

(config-if)#port-group 1   // 将该端口划入聚合端口 AG1 中, 用于聚合端口  

**4. 端口聚合配置**

(config)# interface aggregateport 1   // 创建聚合接口 AG1  

(config-if)# switchport mode trunk   // 配置并保证 AG1 为 trunk 模式  

(config)#int f0/23-24  
(config-if-range)#port-group 1   // 将端口（端口组）划入聚合端口 AG1 中

**5. 生成树**

配置多生成树协议:

switch(config)#spanning-tree        // 开启生成树协议  

switch(config)#spanning-tree mst 

configuration   // 建立多生成树协议

switch(config-mst)#name ruijie           // 命名为 ruijie

switch(config-mst)#revision 1     // 设定校订本为 1

switch(config-mst)#instance0 vlan 10,20   // 建立实例 0

switch(config-mst)#instance 1 vlan 30,40   // 建立实例 1

switch(config)#spanning-tree mst 0 priority 4096  // 设置优先级为 4096

switch(config)#spanning-tree mst 1 priority 8192  // 设置优先级为 8192

switch(config)#interface vlan 10

switch(config-if)#vrrp1ip 192.168.10.1 // 此为 vlan 10 的 IP 地址

switch(config)#interface vlan 20

switch(config-if)#vrrp1ip 192.168.20.1 // 此为 vlan 20 的 IP 地址

switch(config)#interface vlan 30

switch(config-if)#vrrp2ip 192.168.30.1 //

此为 vlan 30 的 IP 地址 (另一三层交换机)

switch(config)#interface vlan 40

switch(config-if)#vrrp2ip 192.168.40.1 //  

此为 vlan 40 的 IP 地址 (另一三层交换机)

**6.vlan 的基本配置**

(config)#vlan 10    // 创建 VLAN10

(config-vlan)#name vlanname   // 命名 VLAN 为 vlanname

(config-if)#switchport access vlan 10   // 将该端口划入 VLAN10 中

某端口的接口配置模式下进行

(config)#interface vlan 10     // 进入 VLAN 10 的虚拟端口配置模式

(config-if)# ip address 192.168.1.1 

255.255.255.0   // 为 VLAN10 的虚拟端口配置 IP 及掩码，二层交换机只能配置一个 IP，此 IP 是作为管理 IP 使用，例如，使用 Telnet 的方式登录的 IP 地址

(config-if)# no shutdown    // 启用该端口

**7. 端口安全**

(config)# interface fastethernet 0/1  // 进入一个端口  

(config-if)# switchport port-security   // 开启该端口的安全功能  

a、配置最大连接数限制

(config-if)# switchport port-secruity maxmum 1 // 配置端口的最大连接数为 1，最大连接数为 128

(config-if)# switchport port-secruity violation shutdown

// 配置安全违例的处理方式为 shutdown，可选为 protect (当安全地址数满后，将未知名地址丢弃)、restrict(当违例时，发送一个 Trap 通知)、shutdown(当违例时将端口关闭，并发送 Trap 通知，可在全局模式下用 errdisable recovery 来恢复)  

b、IP 和 MAC 地址绑定

(config-if)#switchport port-security mac-address xxxx.xxxx.xxxx ip-address 172.16.1.1

// 接口配置模式下配置 MAC 地址 xxxx.xxxx.xxxx 和 IP172.16.1.1 进行绑定 (MAC 地址注意用小写)

**8. 三层路由功能 (针对三层交换机)**

(config)# ip routing     // 开启三层交换机的路由功能  

(config)# interface fastethernet 0/1  (config-if)# no switchport  // 开启端口的三层路由功能 (这样就可以为某一端口配置 IP

(config-if)# ip address 192.168.1.1 

255.255.255.0 

(config-if)# no shutdown

**9. 三层交换机路由协议**

(config)# ip route 172.16.1.0 

255.255.255.0 172.16.2.1  // 配置静态路由  

注: 172.16.1.0 255.255.255.0     // 为目标网络的网络号及子网掩码

172.16.2.1 为下一跳的地址，也可用接口表示, 如 ip route 172.16.1.0 255.255.255.0 serial 1/2(172.16.2.0 所接的端口)  

(config)# router rip   // 开启 RIP 协议进程

(config-router)# network 172.16.1.0   // 宣告本设备的直连网段信息

(config-router)# version 2    // 开启 RIP V2，可选为 version 1(RIPV1)、version 2(RIPV2) 

(config-router)# no auto-summary  // 关闭路由信息的自动汇总功能 (只有在 RIPV2 支持)  

(config)# router ospf  // 开启 OSPF 路由协议进程（针对 1762，无需使用进程 ID）

(config)# router ospf 1  // 开启 OSPF 路由协议进程（针对 2501，需要加 OSPF 进程 ID）

(config-router)# network 192.168.1.0 0.0.0.255 area 0     

// 宣告直连网段信息，并分配区域号 (area0 为骨干区域)

![图片](https://mmbiz.qpic.cn/mmbiz_png/PuI1aiczpHCMV4NbC6UMxVU5DgSpkXsEdGJNT6ZUsCYRjeWtZGiahNnkT2ibA3gLIHbKt1N6CjmPe7odCutlKvkDQ/640?wx_fmt=png)

**向上滑动阅览**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/PuI1aiczpHCMV4NbC6UMxVU5DgSpkXsEd2oPbzwFKP3dedFaZDsywvjtpqkCyz03wAj3aWB0XDdTqsyLxEylfGw/640?wx_fmt=jpeg)

  

**后台回复 “网络工程师****” 获取计算机网络资料**

扫描二维码

获取更多精彩

网络工程师

![](https://mmbiz.qpic.cn/mmbiz_jpg/iawyCGLKzL2ugapBqToXkicoIY4ic0ibx01xWiahduYVb1rVcyPrxEmDyHMGjVJVGyPm2EmVBkTMBac6BOGVOCMMWmw/640?wx_fmt=jpeg)

  

**更多文章**

  

[精选！计算机网络基础学习指南](http://mp.weixin.qq.com/s?__biz=MzIwOTcyNjA3Mw==&mid=2247496445&idx=1&sn=1d7220bdcc0c443d6eaa7d89927ca587&chksm=976dcdeba01a44fdb54ba430e775505e5ee7338d13a7b37011624aa81b7dbd887513d9697e7b&scene=21#wechat_redirect)  

[万字长文！你应该知道的计算机网络基础知识](http://mp.weixin.qq.com/s?__biz=MzIwOTcyNjA3Mw==&mid=2247496319&idx=1&sn=45b95536f595171e3df8b4faca9417d9&chksm=976dcd69a01a447fdd1b76bbdc2eccc3f77cd0fec2a9ecd0a5b6d179c89cc07013cd856e2f06&scene=21#wechat_redirect)  

[案例 | 某园区办公网络网速慢、访问网页的故障处理](http://mp.weixin.qq.com/s?__biz=MzIwOTcyNjA3Mw==&mid=2247491807&idx=1&sn=c1e071bb3bebcca2017898295b7fdfa1&chksm=976ddfc9a01a56df9bfd1ff0ddfb82663ba87feeffecffae077112407848e4d2da02c90e7c8c&scene=21#wechat_redirect)  

  

文章都看完了![](https://mmbiz.qpic.cn/mmbiz_gif/wsJLEGLdzraibqsEMRDEV73MbibIngowdd7uO70ezh44Xa5QfeL0VKA7ZVRDDq8nrKaHOfxhuHniahZeyyBcC2yYw/640?wx_fmt=gif)不点个![](https://mmbiz.qpic.cn/mmbiz_png/PdyqW8iauQhlKJl4aACAiaBibrm26sK7EtbG1b64T1l1t3zXg14lZdbgCA1afqO1wqBicsU7hBQRMRcIM3ZgsVyiaPw/640?wx_fmt=png) 吗