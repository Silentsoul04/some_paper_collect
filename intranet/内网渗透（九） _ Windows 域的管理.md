> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490197&idx=1&sn=4682065ddcab00b584918bc267e33f53&chksm=fc781e48cb0f975eddc44d77698fbb466d0eac7d745a6e5bbaf131560b3d4f9e22c1a359d241&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFgwuEp9SUZPx1nFQ8GW7lWHnnImWeVFF9wBDK21ecqM7sOIV7WVEKzkhHy3nsLFOIx8lkWp4BIQRQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490017&idx=1&sn=426336dfeeda818b0772b3c44703e173&chksm=fc781d3ccb0f942a7c07662752bb2f6983eb9c249c0d6b833f058b1d95fc7080d2d2598054ac&scene=21#wechat_redirect)

**欢迎各位添加微信号：qinchang_198231** 

 **加入安全 + 交流群 和大佬们一起交流安全技术**

Windows 域的管理

目录

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3omM3I3SHqpC3fX8KDoYpANd5TENeGKkGoNfKK0FKj3UEgriaYHvYaRIamze2SWj2vdLqPJBAzZ7HLg/640?wx_fmt=png)

  

域的管理

默认容器

组织单位的管理

添加额外域控制器

卸载域控服务器

组策略应用

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3omM3I3SHqpC3fX8KDoYpANdYKk11PfY9RktpP9RlIuWtuag5B0HJrDcBnzmsmTqpvFeMSXJyPjSFg/640?wx_fmt=png)

### **域的管理**

**域用户账户的管理**

*   创建域用户账户
    
*   配置域用户账户属性
    
*   验证用户的身份
    
*   授权或拒绝对域资源的访问
    

**组的管理**

**组的类型**

*   安全组：安全组有安全标识（SID），能够给其授权访问本地资源或网络资源。即能授权访问资源，也可以利用其群发电子邮件
    
*   通讯组：通迅组没有安全标识（SID），不能授权其访问资源，只能用来群发电子邮件
    

**组的作用域**

*   本地域组：代表的是对某个资源的访问权限。只能授权其访问本域资源，其他域中的资源不能授权其访问。
    
*   全局组：创建全局组是为了合并工作职责相似的用户的账户，只能将本域的用户和组添加到全局组。在多域环境中不能合并其他域中的用户。
    
*   通用组：和全局组的作用一样，目的是根据用户的职责合并用户。与全局组不同的是，在多域环境中它能够合并其他域中的域用户帐户，比如可以把两个域中的经理帐户添加到一个通用组。
    

点击 Users 容器，然后在右边框中可对已存在的用户修改属性，可以修改组的属性，也可以右键，然后新建用户。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2djo88sJ9C76ee5icXJNVXdGjclQPpLdKtgFZ0C3LJXpGCd6W241mvruDC1hnG17bJ7gr71E0cS54g/640?wx_fmt=png)

**组织单位 OU 的管理**

*   OU 的概念
    
*   OU 的应用
    

Active Directory 域内的资源是以对象 (Object) 的形式存在，例如用户、计算机都是对象，而对象是通过属性（Attribute)来描述其特征的，也就是对象本身是一些属性的集合

**容器与组织单位**

*   容器（Container）与对象相似，它有自己的名称，也是一些属性的集合，不过容器内可以包含其他对象（例如用户、计算机等对象)，还可以包含其他容器。
    
*   组织单位（Organization Units，OU) 是一个比较特殊的容器，其中除了可以包含其他对象与组织单位之外，还有组策略（Group Policy）的功能。
    

### **默认容器和组织单位**

*   Builtin 容器：Builtin 容器是 Active Driectory 默认创建的第一个容器，主要用于保存域中本地安全组。
    
*   Computers 容器：Computers 容器是 Active Driectory 默认创建的第 2 个容器，用于存放 Windows Server 2008 域内所有成员计算机的计算机账号。
    
*   Domain Controllers 组织单位：Domain Controllers 是一个特殊的容器，主要用于保存当前域控制器下创建的所有子域和辅助域。
    
*   Users 容器：Users 容器主要用于保存安装 Active Driectory 时系统自动创建的用户和登录到当前域控制器的所有用户账户。
    

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2djo88sJ9C76ee5icXJNVXdGdYJ6hqFAUU5aJ5JuzmRsQXmib0FNYwss8YpXosYnJLGwRHcSmcGyWCg/640?wx_fmt=png)

### **组织单位的管理**

OU 的概念：OU 是 AD 中的容器，可在其中存放用户、组、计算机和其他 OU，而且可以设置组策略

*   创建 OU：基于部门，如行政部、人事部；基于地理位置，如北京、上海；基于对象，如用户、计算机
    
*   删除 OU：取消 “防止对象被意外删除”
    

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2djo88sJ9C76ee5icXJNVXdGmwwbiaJUljqlYx6rFe0ibSVkeYw1nibYptkqvibn4W1DCibqEj3gJ3tUyYg/640?wx_fmt=png)

比如上图中的 Student 就是我们自己新建的组织单位 OU，我们可以把相关的用户或主机加入到 Student 组织单位中，然后对该组织单位设置组策略，那么就可以对其内的所有用户或主机生效 

### **添加额外域控制器**

在一个域中，一般域控服务器至少有 2 个或者以上。所以，我们得添加额外的域控服务器。在任何一台域控制器上都可以修改 AD 中的内容，每台域控制器上 AD 中的内容都是同步的

**添加额外域控制器的条件**

*   具有域管理员权限
    
*   计算机 TCP/IP 参数配置正确 IP、DNS 服务器地址
    
*   操作系统版本必须受当前域功能级别支持
    

**添加额外域控制器的步骤**

*   查看当前域功能级别
    
*   将计算机加入到当前域
    
*   运行 dcpromo 命令安装活动目录
    

### **卸载域控服务器**

运行 dcpromo 命令进行常规卸载，如果该域内还有其他域控制器，该域控制器会被降级为成员服务器，如果是域内最后一个域控制器，该域控制器会被降级为独立服务器。

**卸载域控制器的注意事项**

确认所有域控制器都处于联机状态，确认还有其他域控制器承担着 “全局编录” 角色，在 “Active Directory 站点和服务” 控制台中，查看并手工转移 “全局编录” 角色

### **组策略应用**

计算机配置成域控服务器后，或计算机加入一个域后，会发现本地的安全策略已经呈灰色的，配置不了了。在一个域中，通过在域控服务器上配置组策略，来对域中的主机或域中的用户去设置策略

组策略：Windows 操作系统中的组策略是管理员为用户或计算机定义并控制程序、网络资源和操作系统行为的主要工具。通过使用组策略可以对计算机或者用户设置相应的策略

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2djo88sJ9C76ee5icXJNVXdGsuj0IN7mDInJSgAnqqf9oBN0wg7CBfJibWnWmzsgpju8ibuIMumkMqDw/640?wx_fmt=png)

**组策略的功能**

*   账户策略的设置
    
*   本地策略的设置
    
*   脚本的设置
    
*   用户工作环境的设置
    
*   软件的安装与删除
    
*   限制软件运行
    
*   文件夹的重定向
    
*   限制访问可移动设备
    

**组策略优点**

减小管理成本，只需设置一次，相应的计算机或用户即可应用，减小用户单独配置错误的可能性，可以针对特定对象设置特定的策略

**组策略对象**

GPO （Group Policy Object）的概念：存储组策略的所有配置信息，AD 中的一种特殊对象

默认 GPO：默认域策略、默认域控制器策略

GPO 链接：只能链接到站点、域、OU

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2djo88sJ9C76ee5icXJNVXdGQnA7Ymh3Sol7kwQTlkTt3ZVkEUbyUrBuG4z7kJTkbk44Vicn2iaibDT8A/640?wx_fmt=png)

**组策略的应用规则**

*   策略继承与阻止：下级容器可以继承或阻止应用其上级容器的 GPO 设置
    
*   策略累加与冲突：多个 GPO 设置可以累加或发生冲突被覆盖
    
*   策略强制生效：使下级容器强制执行其上级容器的 GPO 设置
    
*   筛选：阻止一个容器内的用户或计算机应用其 GPO 设置
    

**策略继承与阻止**

下级容器默认会继承来自上级容器的 GPO ，子容器可以阻止继承上级容器的 GPO ，右击容器→阻止继承

**策略累加与冲突**

如果多个组策略设置不冲突，则最终的有效策略是所有组策略设置的累加 如果多个组策略设置冲突，则后应用的组策略覆盖先应用的组策略

**组策略应用顺序**

组策略应用顺序：

*   首先应用本地组策略
    
*   如果有站点组策略，则应用
    
*   接着应用域策略
    
*   最后应用 OU 上的策略
    
*   如果同一个 OU 上链接了多个 GPO，则按照链接顺序从高到低逐个应用
    

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2djo88sJ9C76ee5icXJNVXdGCnTKCd4tEAuQcHwKB7dhblSdmBLbdlxQTjUbNVHnNlJWJDhBn1Je9g/640?wx_fmt=png)

策略强制生效：强制生效是上级容器强制下级容器执行其 GPO 设置

筛选：筛选可以阻止一个 GPO 应用于容器内的特定计算机或用户 委派→权限设置

打开本地组策略：WIN+R 键打开运行窗口，然后输入：gpedit.msc

打开组策略：管理工具 --> 组策略管理

![](https://mmbiz.qpic.cn/mmbiz_png/YUyZ7AOL3omM3I3SHqpC3fX8KDoYpANdEjSAenLicg6l3iaUZszkjc7wGJJX38AbjrOaHGiadorWuCGBgNWtcxqiaw/640?wx_fmt=png)

END

责编：vivian

来源：谢公子博客

[![](https://mmbiz.qpic.cn/mmbiz_jpg/UZ1NGUYLEFiaJeGjd04dibz7iah6JTVeLsicT9kVuXfNXdqGhfhvhCZicafopwTts4dZoF9icPAFJh9RQ9omsbplQLTA/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=MjM5NDY1OTA1NQ==&mid=2650791647&idx=1&sn=400183ea5bea0a3156a2b3a016a0f4d7&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/UZ1NGUYLEFjGibCQezQKY4NzE1WGn6FBCbq3pQVl0oONnYXT354mlVw0edib6X6flYib9JRTic4DTibgib15WZC7sDUA/640?wx_fmt=png)

[**内网渗透（八） | 内网转发工具的使用**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247490042&idx=1&sn=136d4057044a7d6f6cb5b57d20f7954a&chksm=fc781d27cb0f9431ec590662ab4e6bcd31b303e7caa20a2b116fd9a9b97e9e3be0bc34408490&scene=21#wechat_redirect)  

[**内网渗透 | 域内用户枚举和密码喷洒攻击 (Password Spraying)**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489985&idx=1&sn=0b7bce093e501b9817f263c24e0ed5b8&chksm=fc781d1ccb0f940aad0c9b2b06b68c7a58b0b4c513fe45f7da6e6438cac76d4778e61122faf8&scene=21#wechat_redirect)  

[**内网渗透（七） | 内网转发及隐蔽隧道：网络层隧道技术之 ICMP 隧道 (pingTunnel/IcmpTunnel)**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489736&idx=2&sn=0cb551ee520860878c2c33108033c00c&chksm=fc781c15cb0f9503f672aa0bd18cb13fef4c60124ba5978ab947c34272b2d8a28c584a99219d&scene=21#wechat_redirect)  

[**内网渗透（六） | 工作组和域的区别**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489205&idx=1&sn=24f9a2e0e6b92a167f3082bb6e09c734&chksm=fc781268cb0f9b7e3c11d19a9fb41567124055eb0e8dd526cbbaf1e9393ff707f9fa9d10c32b&scene=21#wechat_redirect)  

[**内网渗透（五） | AS-REP Roasting 攻击**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489128&idx=1&sn=dac676323e81307e18dd7f6c8998bde7&chksm=fc7812b5cb0f9ba3a63c447468b7e1bdf3250ed0a6217b07a22819c816a8da1fdf16c164fce2&scene=21#wechat_redirect)

[**内网渗透 | 内网穿透工具 FRP 的使用**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247489057&idx=3&sn=f81ef113f1f136c2289c8bca24c5deb1&chksm=fc7812fccb0f9beaa65e5e9cf40cf9797d207627ae30cb8c7d42d8c12a2cb0765700860dab84&scene=21#wechat_redirect)  

[**内网渗透（四） | 域渗透之 Kerberoast 攻击_Python**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488972&idx=1&sn=87a6d987de72a03a2710f162170cd3a0&chksm=fc781111cb0f98070f74377f8348c529699a5eea8497fd40d254cf37a1f54f96632da6a96d83&scene=21#wechat_redirect)  

[**内网渗透（三） | 域渗透之 SPN 服务主体名称**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488936&idx=1&sn=82c127c8ad6d3e36f1a977e5ba122228&chksm=fc781175cb0f986392b4c78112dcd01bf5c71e7d6bdc292f0d8a556cc27e6bd8ebc54278165d&scene=21#wechat_redirect)  

[**内网渗透（二） | MSF 和 CobaltStrike 联动**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488905&idx=2&sn=6e15c9c5dd126a607e7a90100b6148d6&chksm=fc781154cb0f98421e25a36ddbb222f3378edcda5d23f329a69a253a9240f1de502a00ee983b&scene=21#wechat_redirect)  

[**内网渗透 | 域内认证之 Kerberos 协议详解**](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488900&idx=3&sn=dc2689efec7757f7b432e1fb38b599d4&chksm=fc781159cb0f984f1a44668d9e77d373e4b3bfa25e5fcb1512251e699d17d2b0da55348a2210&scene=21#wechat_redirect)  

**[内网渗透（一） | 搭建域环境](http://mp.weixin.qq.com/s?__biz=MzU2MTQwMzMxNA==&mid=2247488866&idx=2&sn=89f9ca5dec033f01e07d85352eec7387&chksm=fc7811bfcb0f98a9c2e5a73444678020b173364c402f770076580556a053f7a63af51acf3adc&scene=21#wechat_redirect)**