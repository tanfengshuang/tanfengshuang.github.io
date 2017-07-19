---
layout: post
title:  "Implementing Puppet(405-11)"
categories: Linux
tags: RHCA 405
---

### 实施 Puppet 宿主

Puppet 宿主为 Puppet 模块和报告提供一个集中的位置。Puppet 宿主可以根据 Puppet 宿主上存储的配置，为每一个 Puppet 节点构建目录。此节点分类用于将模块中的类分配到 Puppet 节点，从而能在多个系统上重复利用 Puppet 代码。这种集中存储 Puppet 模块的方式也提供了一个高效率途径，一次性更新代码并应用到多个 Puppet 节点上。

Puppet 可以在单机模式中运行；所有 Puppet 节点在本地将 Puppet 模块应用到系统中。但是，如果没有集中式 Puppet 宿主，在多台计算机之间一致地管理更改就会变得更加困难。系统管理员将遇到他们一直面临的相同问题：如何大规模地一致管理许多系统。这一原因再加上其他因素，我们建议使用 Puppet 宿主为所有 Puppet 节点存储和管理模块。

在单机 Puppet 配置中，系统管理员直接对 Puppet 烟雾测试文件（包含模块中各种不同的类）使用 puppet apply 命令来应用更改到节点。在服务器/客户端（宿主/代理）架构中，可以从 Puppet 节点上的命令行界面中运行 puppet agent 命令来立即应用分配到该节点的类。此外，Puppet 节点上应当启动并启用 puppet 服务，它默认为每 30 分钟运行一次 puppet agent。

###### 部署 Puppet 宿主

1. 若要部署 Puppet 宿主，首先要安装 puppet-server 软件包：

```
[root@host ~]# yum install -y puppet-server
```

2. 启动并启用 puppetmaster 服务：

```
[root@host ~]# systemctl start puppetmaster.service
[root@host ~]# systemctl enable puppetmaster.service
```

3. 确定 Puppet 宿主为来自 Puppet 客户端的连接而侦听的 TCP 端口。ss -tlp 命令显示 TCP 端口上侦听进程的信息。

```
[root@serverc ~]# ss -tlp | grep puppet
LISTEN     0      128                     *:8140                     *:*
 users:(("puppet",1465,9))
```

Puppet 侦听 TCP 端口 8140。

4. Puppet 宿主侦听 TCP/8140，因此要在防火墙中永久打开此端口。

```
[root@host ~]# firewall-cmd --permanent --add-port=8140/tcp
```


### 实施 Puppet 客户端 

虽然 Puppet 可以单机模式运行，所有 Puppet 客户端都在本地应用 Puppet 模块到系统中，但大部分系统管理员发现 Puppet 最适合利用中央化的 Puppet 宿主来工作。部署 Puppet 客户端的第一步是安装 puppet 软件包：

```
[root@node ~]# yum install -y puppet
```

安装了 puppet 软件包后，必须为 Puppet 客户端配置 Puppet 宿主的主机名。Puppet 宿主的主机名应当放在 /etc/puppet/puppet.conf 文件中的 [agent] 部分下。下方的代码片段演示了一个示例，其中的 Puppet 宿主命名为 puppet.lab.example.com：

```
[agent]
    server = puppet.lab.example.com
```

> 如果未定义明确的 Puppet 宿主，Puppet 使用默认的主机名 puppet。如果 DNS 搜索路径中包含名为 puppet 的主机，将自动使用这一主机。 

对 Puppet 客户端执行的最后一步是启动 Puppet 代理服务并将它配置为在引导时运行。

```
[root@node ~]# systemctl start puppet.service
[root@node ~]# systemctl enable puppet.service
ln -s '/usr/lib/systemd/system/puppet.service'
 '/etc/systemd/system/multi-user.target.wants/puppet.service'
```

Puppet 代理服务将生成主机证书，并向 Puppet 宿主发送证书签名请求。发送了客户端证书请求后，Puppet 宿主上便可看到要签署客户端证书的请求。

```
[root@master ~]# puppet cert list
  "node.example.com" (SHA256) 24:C6:06:B5:43:D8:4C:77:C8:F6:68:9B:4C:8D:A9:29:74:
 05:70:01:0D:DB:D8:C0:10:D2:4D:FF:F3:A2:3B:32
```

puppet cert sign 命令将在 Puppet 宿主上签署客户端证书：

```
[root@master ~]# puppet cert sign node.example.com.
Notice: Signed certificate request for node.example.com
Notice: Removing file Puppet::SSL::CertificateRequest node.example.com
at '/var/lib/puppet/ssl/ca/requests/node.example.com.pem'
```

puppet cert list 命令仅显示待处理的请求。因此，它应当报告没有要签名的待处理密钥。

```
[root@master ~]# puppet cert list
```

若要查看已签名的证书，可在命令后加上主机名。

```
[root@master ~]# puppet cert list node.example.com
+ "node.example.com" (SHA256) 64:C7:00:BB:DB:4C:74:1F:CD:D5:B2:83:87:53:65:28:32:
 32:25:A3:2B:A7:51:06:71:7E:A5:69:BB:5E:D3:33
```

Puppet 宿主为证书签名后，Puppet 代理将收到一个目录并应用所需的任何更改。 


### 练习：实施 Puppet 客户端

1. 若要安装 Puppet 客户端，首先要安装 Puppet 软件。以 root 身份登录 serverb，再安装 puppet 软件包。

```
[root@serverb ~]# yum -y install puppet
```

2. 将 Puppet 代理配置为引用 Puppet 宿主。编辑 /etc/puppet/puppet.conf，并将下列内容添加到配置文件的 [agent] 段中。

```
    # Define which Puppet master to use.
    server = serverc.lab.example.com
```

server 设置应当指向 Puppet 宿主的完全限定主机名；本例中为 serverc.lab.example.com。

3. 通常，您要启动 Puppet 代理服务并将它配置为在引导时运行。在本实践练习中，您将手动运行 Puppet 代理，从而能查看它与 Puppet 宿主的互动。

puppet agent 第一次在 Puppet 客户端上运行时，它将生成证书并将它发送到 Puppet 宿主。Puppet 宿主必须接受和签署此证书，并将它返回到客户端，然后客户端才会被允许从 Puppet 宿主获取目录。当 puppet agent 首次运行时，它将默认等待两分钟（120 秒），然后重新发送签署证书的请求。Puppet 宿主可以配置自动签署证书，但这会带来安全风险；因此，通常必须在 Puppet 宿主上手动签署客户端证书。

运行 puppet agent 命令，让它每隔 10 秒重新发送证书签名请求，直到它从 Puppet 宿主收到回复为止。--waitforcert 选项可以更改重新发送证书请求的间隔时间。

```
[root@serverb ~]# puppet agent --waitforcert 10
```

查看 /var/log/messages 可以发现代理的重试记录。

```
[root@serverb ~]# tail /var/log/messages
... Output omitted ...
Oct 26 18:48:25 localhost puppet-agent[1493]: Did not receive certificate
Oct 26 18:48:35 localhost puppet-agent[1493]: Did not receive certificate
Oct 26 18:48:45 localhost puppet-agent[1493]: Did not receive certificate
```

4. 以 root 身份登录 Puppet 宿主 serverc。使用 puppet cert list 命令列出来自 Puppet 节点的待处理证书签名请求。

```
[root@serverc ~]# puppet cert list
  "serverb.lab.example.com" (SHA256) 25:03:59:01:D9:33:7B:5D:8D:E0:C2:53:41:5D:CE:
 00:A1:0D:2D: CA:AA:08:7C:5F:AE:17:3C:37:14:6C:1A:62
```

5. 在 Puppet 宿主上签署客户端证书。

```
[root@serverc ~]# puppet cert sign serverb.lab.example.com
Notice: Signed certificate request for serverb.lab.example.com
Notice: Removing file Puppet::SSL::CertificateRequest serverb.lab.example.com at
'/var/lib/puppet/ssl/ca/requests/serverb.lab.example.com.pem'
```

6. 在 Puppet 宿主上列出待处理证书请求不会显示任何输出。添加 Puppet 客户端主机名到 puppet cert list 命令将列出其已签名证书的指纹。

```
[root@serverc ~]# puppet cert list
[root@serverc ~]# puppet cert list serverb.lab.example.com
+ "serverb.lab.example.com" (SHA256) 4B:9F:60:B0:A6:89:26:09:B0:72:6A:9A:FF:46:E5:
 EE:F7:E2:C8:F2:12:62:67:44:69:C7:10:D5:B0:D9:67:18
```

7. 在 Puppet 客户端 serverb 上，检查 /var/log/messages，并确认 Puppet 代理收到了已签名的证书。其输出应与以下内容相似：

```
[root@serverb ~]# tail /var/log/messages
... Output omitted ...
Oct 26 18:50:15 localhost puppet-agent[1493]: Did not receive certificate
Oct 26 18:50:25 localhost puppet-agent[1493]: Did not receive certificate
Oct 26 18:50:35 localhost puppet-agent[1493]: Starting Puppet client version 3.6.2
Oct 26 18:50:36 localhost puppet-agent[26188]: Finished catalog run in 0.05 seconds
```

收到证书后，原始的 puppet agent 进程将生成一个子进程，以接受并运行来自 Puppet 宿主的目录。 

8. 在 Puppet 客户端上，使用 --noop 选项运行 Puppet 代理，确保它可以搭配 Puppet 宿主工作。如果一切正常，您应当看到类似于下文的输出。

```
[root@serverb ~]# puppet agent --test --noop
Info: Retrieving plugin
Info: Caching catalog for serverb.lab.example.com
Info: Applying configuration version '1445899836'
Notice: Finished catalog run in 0.02 seconds
```

9. 最初启动的 Puppet 代理仍在后台运行。需要识别并终止它，才能启动 Puppet 代理服务。

```
[root@serverb ~]# ps -ef | grep puppet
root      1493     1  0 13:34 ?        00:00:04 /usr/bin/ruby /usr/bin/puppet agent --waitforcert 10
root     26431 25952  0 15:52 pts/0    00:00:00 grep --color=auto puppet
[root@serverb ~]# kill 1493
[root@serverb ~]# ps -ef | grep puppet
root     26435 25952  0 15:53 pts/0    00:00:00 grep --color=auto puppet
```

10. 使用 systemctl 命令启动并启用 puppet.service 服务。检查其状态，以确认服务正在运行。

```
[root@serverb ~]# systemctl start puppet.service
[root@serverb ~]# systemctl enable puppet.service
ln -s '/usr/lib/systemd/system/puppet.service' '/etc/systemd/system/multi-user.target.wants/puppet.service'
[root@serverb ~]# systemctl status puppet.service
puppet.service - Puppet agent
   Loaded: loaded (/usr/lib/systemd/system/puppet.service; enabled)
   Active: active (running) since Mon 2015-10-26 15:53:38 PDT; 12s ago
 Main PID: 26438 (puppet)
   CGroup: /system.slice/puppet.service
           └─26438 /usr/bin/ruby /usr/bin/puppet agent --no-daemonize
 
Oct 26 15:53:38 serverb.lab.example.com systemd[1]: Starting Puppet agent...
Oct 26 15:53:38 serverb.lab.example.com systemd[1]: Started Puppet agent.
Oct 26 15:53:39 serverb.lab.example.com puppet-agent[26438]: Starting Puppet ...
Oct 26 15:53:39 serverb.lab.example.com puppet-agent[26444]: Finished catalog...
Hint: Some lines were ellipsized, use -l to show in full.
```


### 重复利用 Puppet 类

部署了 Puppet 宿主后，可在 Puppet 宿主上使用 puppet module install 命令安装模块。模块中的类现在可以利用节点分类分配到 Puppet 节点。节点分类在 /etc/puppet/manifests/site.pp 文件中配置。此文件应当始终包含一个 default 部分，即使其中不定义任何类，这样才能避免分类错误。下例显示了包含有 test 类（假定已在安装的模块中定义）的 default 部分：

```
node default {
  include test
}
```

注意其语法比较眼熟，因为它使用名称和花括号 ({}) 来包含定义。可以指定节点名称（与证书上的主机名匹配）来包含类。下列演示了一个空的 default 节点定义，后跟一个使用 test 类的 node.example.com 定义：

```
node default {
}
node 'node.example.com' {
 include test
}
```

> 注意： default 名称是特殊关键字，不可用引号括起。主机名为字符串，始终应当用引号括起。

可以使用逗号分隔列表来指定多个节点：

```
node default {
}
node 'node1.example.com', 'node2.example.com', 'node3.example.com' {
 include test
}
```

> 注意： 尽量避免节点名称存在多个匹配项，因为这可能会造成意外的结果。节点分类使用第一个最具体的匹配项匹配，所以文件顺序或有影响。

也可以使用正则表达式匹配节点。通常使用词语匹配（^ 表示词首，$ 表示词尾）来限制匹配的范围。下例与上例中的三个节点匹配：

```
node default {
}
node /^node[1-3].example.com$/ {
 include test
}
```

可以使用选择器 \d 匹配任何数字。+ 修饰符可用于匹配一个或多个其前面的字符。因此，当它们一起使用时（如下例所示），其组合将匹配一个或多个数字：

```
node default {
}
node /^node\d+.example.com$/ {
 include test
}
```

此正则表达式将匹配下列（及更多）主机名：

*    node1.example.com
*    node2.example.com
*    node10.example.com
*    node42.example.com
*    node123456.example.com

Puppet 节点获得了节点分类后，下一次运行将执行必要的步骤，使该节点与目录一致。当 puppet 服务首先运行时，它将在 Puppet 宿主签入（必要时请求证书），此后每隔 30 分钟（默认值）签入一次。此 30 分钟默认间隔可以使用 /etc/puppet/puppet.conf 文件中的 runinterval 选项更改。其值通常以秒为单位给定，但可以附加 m（分钟）或 h（小时）等修饰符。下例显示了 /etc/puppet/puppet.conf 的一个代码片段，它使用的值为 15 分钟，而非默认的 30 分钟。

```
[agent]
  runinterval = 15m
```

设置了此选项后，Puppet 代理将每隔 15 分钟自动检测更改，并开始连接 Puppet 宿主；不需要通过重启服务来提取更改。若要验证配置，可以使用 puppet agent 命令的 --configprint 选项：

```
[root@host ~]# puppet agent --configprint runinterval
900
```

> 注意: 必须以 root 用户身份运行 puppet agent --configprint 命令，才能获取准确的值。如果以普通的非特权用户运行该命令，报告的将是内置的默认值，而非当前有效的值。 


### 练习：重复利用 Puppet 类

1. 在 servera 上，以 student 身份将 rht-ftpcustom 模块从 servera 复制到 serverc。

```
[student@servera ~]$ scp labwork/rht-ftpcustom/pkg/rht-ftpcustom-0.1.0.tar.gz root@serverc:
```

2. 在 serverc 上，以 root 身份安装 rht-ftpcustom 模块。

```
[root@serverc ~]# puppet module install rht-ftpcustom-0.1.0.tar.gz
```

3. 在 serverb 上，验证 ftpcustom 类将包含在 puppet agent 运行中，但不要应用更改。

```
[root@serverb ~]# puppet agent --test --noop
Info: Retrieving plugin
Info: Caching catalog for serverb.lab.example.com
... Output omitted ...
Notice: Class[Ftpcustom::Install]: Would have triggered 'refresh' from 1 events
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/vsftpd.conf]/ensure: current_value absent, should be file (noop)
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/custom-banner]/ensure: current_value absent, should be file (noop)
Notice: Class[Ftpcustom::Config]: Would have triggered 'refresh' from 2 events
Info: Class[Ftpcustom::Config]: Scheduling refresh of Class[Ftpcustom::Service]
Notice: Class[Ftpcustom::Service]: Would have triggered 'refresh' from 1 events
Info: Class[Ftpcustom::Service]: Scheduling refresh of Service[vsftpd]
Notice: /Stage[main]/Ftpcustom::Service/Service[vsftpd]/ensure: current_value stopped, should be running (noop)
Info: /Stage[main]/Ftpcustom::Service/Service[vsftpd]: Unscheduling refresh on Service[vsftpd]
Notice: Class[Ftpcustom::Service]: Would have triggered 'refresh' from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 3 events
Notice: Finished catalog run in 0.71 seconds
```

4. 在 serverc 上，再次编辑 /etc/puppet/manifests/site.pp 文件。注释掉 default 部分中的 ftpcustom 类，再将它包含在一个含有 servera 和 serverb 的新节点分类中。注意主机名应当用引号括起。该文件应当包含以下内容：

```
node default {
  #include ftpcustom
}

node 'servera.lab.example.com', 'serverb.lab.example.com' {
  include ftpcustom
}
```

5. 在 serverb 上，验证 ftpcustom 类将包含在 puppet agent 运行中，但不要应用更改。

```
[root@serverb ~]# puppet agent --test --noop
Info: Retrieving plugin
Info: Caching catalog for serverb.lab.example.com
... Output omitted ...
Notice: Class[Ftpcustom::Install]: Would have triggered 'refresh' from 1 events
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/vsftpd.conf]/ensure: current_value absent, should be file (noop)
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/custom-banner]/ensure: current_value absent, should be file (noop)
Notice: Class[Ftpcustom::Config]: Would have triggered 'refresh' from 2 events
Info: Class[Ftpcustom::Config]: Scheduling refresh of Class[Ftpcustom::Service]
Notice: Class[Ftpcustom::Service]: Would have triggered 'refresh' from 1 events
Info: Class[Ftpcustom::Service]: Scheduling refresh of Service[vsftpd]
Notice: /Stage[main]/Ftpcustom::Service/Service[vsftpd]/ensure: current_value stopped, should be running (noop)
Info: /Stage[main]/Ftpcustom::Service/Service[vsftpd]: Unscheduling refresh on Service[vsftpd]
Notice: Class[Ftpcustom::Service]: Would have triggered 'refresh' from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 3 events
Notice: Finished catalog run in 0.71 seconds
```

6. 在 serverc 上，最后一次编辑 /etc/puppet/manifests/site.pp 文件。这一次，将 serverc 包含在节点分类中。删除个体节点列表，再使用正则表达式来匹配所有三台服务器。该文件应当包含以下内容：

```
node default {
  #include ftpcustom
}

node /^server[a-c].lab.example.com$/ {
  include ftpcustom
}
```

7. 在 serverb 上，验证 ftpcustom 类包含在 puppet agent 运行中。这一次应用更改。

```
[root@serverb ~]# puppet agent --test
Info: Retrieving plugin
Info: Caching catalog for serverb.lab.example.com
...output omitted...
Notice: /Stage[main]/Ftpcustom::Install/Package[vsftpd]/ensure: created
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/vsftpd.conf]/content: 
--- /etc/vsftpd/vsftpd.conf	2014-03-07 04:58:21.000000000 -0500
+++ /tmp/puppet-file20151016-27916-17s23ed	2015-10-16 15:44:42.498593814 -0400
@@ -84,6 +84,7 @@
 #
 # You may fully customise the login banner string:
 #ftpd_banner=Welcome to blah FTP service.
+banner_file=/etc/vsftpd/custom-banner
 #
 # You may specify a file of disallowed anonymous e-mail addresses. Apparently
 # useful for combatting certain DoS attacks.

Info: FileBucket got a duplicate file {md5}c4072ca90053a6e86cf86850c343346d
Info: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/vsftpd.conf]: Filebucketed /etc/vsftpd/vsftpd.conf to puppet with sum c4072ca90053a6e86cf86850c343346d
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/vsftpd.conf]/content: content changed '{md5}c4072ca90053a6e86cf86850c343346d' to '{md5}81ee930dab4087a96ea4f55b699702d2'
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/custom-banner]/ensure: defined content as '{md5}61c72b9a1e7bf54059d3b680dd1a0929'
Info: Class[Ftpcustom::Config]: Scheduling refresh of Class[Ftpcustom::Service]
Info: Class[Ftpcustom::Service]: Scheduling refresh of Service[vsftpd]
Notice: /Stage[main]/Ftpcustom::Service/Service[vsftpd]/ensure: ensure changed 'stopped' to 'running'
Info: /Stage[main]/Ftpcustom::Service/Service[vsftpd]: Unscheduling refresh on Service[vsftpd]
Notice: Finished catalog run in 3.92 seconds
```

8. 验证 FTP 服务器的功能。

    a. 安装 FTP 客户端。

```
    [root@serverb ~]# yum install -y ftp
```

    b. 连接本地的 FTP 服务器。

```
    [root@serverb ~]# ftp ::1
    Connected to ::1 (::1).
    220-############################################################
    220-#################### Private FTP Server ####################
    220-############# Only authorized users permitted. #############
    220-############################################################
    220    
    Name (::1:root): 
```

    c. 确认显示了自定义横幅后，使用 Ctrl+C 中断连接并返回到 shell。 


### 实验：实施 Puppet

1. 以 student 身份登录 workstation，再运行实验设置脚本。它将确认 Puppet 宿主和客户端服务器的主机名和网络设置是否正确。

```
[student@workstation ~]$ lab puppet-master-client setup
```

2. 在 serverc 上以 root 登录，再使用 yum 安装 puppet-server 软件包。

```
[root@serverc ~]# yum -y install puppet-server
```

3. 启动适当的服务并将它配置为在引导时运行。确保防火墙端口已经开启，以便客户端能够连接 Puppet 服务。

    a. 使用 systemctl 命令启动 Puppet 宿主服务 puppetmaster.service。立即并且永久启动该服务。

```
    [root@serverc ~]# systemctl start puppetmaster.service
    [root@serverc ~]# systemctl enable puppetmaster.service
    ln -s '/usr/lib/systemd/system/puppetmaster.service'
     '/etc/systemd/system/multi-user.target.wants/puppetmaster.service'
```

    b. Puppet 服务器服务侦听 TCP 端口 8140。在防火墙中永久打开此端口，让 Puppet 客户端能够连接 Puppet 宿主。

```
    [root@serverc ~]# firewall-cmd --add-port=8140/tcp
    success
    [root@serverc ~]# firewall-cmd --permanent --add-port=8140/tcp
    success
```

4. 在 serverb 上安装 Puppet 代理软件。以 root 身份登录 serverb，再使用 yum 安装 puppet 软件包。

```
[root@serverb ~]# yum -y install puppet
```

5. 将 Puppet 代理配置为引用 Puppet 宿主 serverc.lab.example.com。编辑 /etc/puppet/puppet.conf 配置文件，并将下列内容添加到 [agent] 段中。

```
    # Define which Puppet master to use.
    server = serverc.lab.example.com
```

6. 启动适当的 Puppet 客户端守护进程，并将它配置为在引导时运行。它将生成客户端主机证书，并向 Puppet 宿主发送证书签名请求。

```
[root@serverb ~]# systemctl start puppet.service
[root@serverb ~]# systemctl enable puppet.service
ln -s '/usr/lib/systemd/system/puppet.service'
 '/etc/systemd/system/multi-user.target.wants/puppet.service'
```

7. 在 Puppet 宿主上签署客户端的证书。

    a. 返回到 Puppet 宿主，再使用 puppet cert list 命令列出来自 Puppet 客户端的待处理证书签名请求。

```
    [root@serverc ~]# puppet cert list
      "serverb.lab.example.com" (SHA256) 24:C6:06:B5:43:D8:4C:77:C8:F6:68:9B:4C:8D:
      A9:29:74:05:70:01:0D:DB:D8:C0:10:D2:4D:FF:F3:A2:3B:32
```

    b. 签署来自客户端 serverb 的证书签名请求。

```
    [root@serverc ~]# puppet cert sign serverb.lab.example.com
    Notice: Signed certificate request for serverb.lab.example.com
    Notice: Removing file Puppet::SSL::CertificateRequest serverb.lab.example.com
     at '/var/lib/puppet/ssl/ca/requests/serverb.lab.example.com.pem'
```

8. 确认 Puppet 客户端与 Puppet 宿主正常配合。 在 Puppet 客户端上，以 noop 模式运行 Puppet 代理，确保它可以搭配 Puppet 宿主工作。

```
[root@serverb ~]# puppet agent --test --noop
Info: Retrieving plugin
Info: Caching catalog for serverb.lab.example.com
Info: Applying configuration version '1443898587'
Notice: Finished catalog run in 0.06 seconds
```

9. 在 serverc 上，从 http://materials.example.com/modules/thoraxe-motd-0.1.1.tar.gz 下载 motd 模块。

```
[root@serverc ~]$ wget http://materials.example.com/modules/thoraxe-motd-0.1.1.tar.gz
```

10. 安装 thoraxe-motd 模块。

```
[root@serverc ~]# puppet module install thoraxe-motd-0.1.1.tar.gz
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└── thoraxe-motd (v0.1.1)
```

11. 在 serverc 上，创建 /etc/puppet/manifests/site.pp 文件，并且为 servera、serverb 和 serverc 添加包含有 thoraxe-motd 模块的 motd 类的节点分类。

可以通过多种方式完成此任务；本例中使用默认的节点定义，但您可以使用逗号分隔的服务器列表，或者使用与服务器名称匹配的正则表达式。该文件应当包含以下内容：

```
node default {
  include motd
}
```

12. 在 serverb 上，验证 motd 类将包含在 puppet agent 运行中，但不要应用更改。

```
[root@serverb ~]# puppet agent --test --noop
Info: Retrieving plugin
Info: Caching catalog for serverb.lab.example.com
Info: Applying configuration version '1445026071'
Notice: /Stage[main]/Motd/File[/etc/motd]/content: 
--- /etc/motd	2013-06-07 10:31:32.000000000 -0400
+++ /tmp/puppet-file20151016-28316-189cjl1	2015-10-16 16:07:52.092218383 -0400
@@ -0,0 +1 @@
+This is the default message 

Notice: /Stage[main]/Motd/File[/etc/motd]/content: current_value {md5}d41d8cd98f00b204e9800998ecf8427e, should be {md5}c01d1bd4d04a962e8364a4fb55e59f05 (noop)
Notice: Class[Motd]: Would have triggered 'refresh' from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Finished catalog run in 0.11 seconds
```

13. 应用 Puppet 类到 serverb。

```
[root@serverb ~]$ puppet agent --test
Info: Retrieving plugin
Info: Caching catalog for serverb.lab.example.com
Info: Applying configuration version '1445026071'
Notice: /Stage[main]/Motd/File[/etc/motd]/content: 
--- /etc/motd	2013-06-07 10:31:32.000000000 -0400
+++ /tmp/puppet-file20151016-28447-8tzqi3	2015-10-16 16:09:49.094928524 -0400
@@ -0,0 +1 @@
+This is the default message 

Info: /Stage[main]/Motd/File[/etc/motd]: Filebucketed /etc/motd to puppet with sum d41d8cd98f00b204e9800998ecf8427e
Notice: /Stage[main]/Motd/File[/etc/motd]/content: content changed '{md5}d41d8cd98f00b204e9800998ecf8427e' to '{md5}c01d1bd4d04a962e8364a4fb55e59f05'
Notice: Finished catalog run in 0.17 seconds
```

14. 验证对 serverb 的添加。

```
[root@serverb ~]$ cat /etc/motd
This is the default message 
```


### 总结

在本章中，您学到了：

*    必须安装 puppet-server 软件包才能实施 Puppet 宿主。
*    puppetmaster.service 服务管理 Puppet 宿主守护进程。
*    充当 Puppet 宿主的服务器上必须开启 TCP 端口 8140。
*    puppet 软件包含有实施 Puppet 客户端需要的所有软件。
*    /etc/puppet/puppet.conf 的 [agent] 部分的 server = FQDN-of-master 行使 Puppet 代理软件指向 Puppet 宿主。
*    puppet.service systemd 服务控制客户端上的 Puppet 代理软件。
*    当 Puppet 代理向 Puppet 宿主签入时，puppet cert sign FQDN-of-client 命令可签署客户端的 CSR，并在下一次签入时返回签名后的主机证书。
*    /etc/puppet/manifests/site.pp 中的节点定义指定将要在 Puppet 客户端上部署的 Puppet 类。 

