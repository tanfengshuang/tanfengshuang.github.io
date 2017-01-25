---
layout: post
title:  "Configuring System Logging(413-13)"
categories: Linux
tags: 413 
---

### Configuring rsyslog Log File Management

Linux上通常可以通过rsyslog来实现系统日志的集中管理，这种情况下通常会有一个日志服务器，然后每个机器配置自己日志通过rsyslog来写到远程的日志服务器上。

*    Package rsyslog
*    Configure file: /etc/rsyslog.conf  /etc/rsyslog.d/*.conf

###### Remote Logging with Encrypted TCP

1. Clock First: 检查两台机器上的时钟是否一致
2. Certificates - TLS(Transport Layer Security 安全传输层协议)
3. Server Configuration

   *    rsyslog: open a TCP port to accept encrypted connection
   *    Install rsyslog-gnutls package to provide the rsyslog plugins that support TLS encryption
   *    Server certificate, Server private key, CA certificate

4. Client Configuration
   *    CA certificate
   *    Send (all) messages to log Server
   *    Install rsyslog-gnutls package to provide the rsyslog plugins that support TLS encryption
   
###### Using Rulesets to Divide Local and Remote Logs

```
# vim /etc/syslog.conf
*.* @@(o) Server-IP:portN

# vim /etc/rsyslog.d/1.conf
:fromhost-ip, isequal, "Client-IP1"     /var/log/dir1/messages
:fromhost-ip, isequal, "Client-IP2"     /var/log/dir1/messages
```

###### Performance Checklist: Secure Logging with rsyslog

```
[server] # date
Sun Jan 22 22:19:36 EST 2017

[client]# date
Sun Jan 22 22:19:36 EST 2017
[client]# certtool --generate-privkey --outfile ca-key.pem
Generating a 2048 bit RSA private key...
[client]# ll ca-key.pem
-rw-------. 1 root root 1679 Jan 22 22:21 ca-key.pem
[client]# chmod 400 ca-key.pem
[client]# ll ca-key.pem
-r--------. 1 root root 1679 Jan 22 22:21 ca-key.pem
[client]# certtool --generate-self-signed --load-privkey ca-key.pem  --outfile ca.pem
[client]# ll ca.pem
-rw-r--r--. 1 root root 1281 Jan 22 22:27 ca.pem
[client]# file ca.pem 
ca.pem: ASCII text

[client]# certtool --generate-privkey --outfile server-key.pem              -> 生成server上的key，可以在任意机器上申请
Generating a 2048 bit RSA private key...
[client]# certtool --generate-request --load-privkey server-key.pem --outfile server-request.pem
...
Is this a TLS web client certificate? (y/N): y
Is this also a TLS web server certificate? (y/N): y
[client]# certtool --generate-certificate --load-request server-request.pem --outfile server-cert.pem --load-ca-certificate ca.pem --load-ca-privkey ca-key.pem
...
Is this a TLS web client certificate? (y/N): y
Is this also a TLS web server certificate? (y/N): y
[client]# ll
total 20
-r--------. 1 root root 1679 Jan 22 22:21 ca-key.pem
-rw-r--r--. 1 root root 1281 Jan 22 22:27 ca.pem
-rw-r--r--. 1 root root 1334 Jan 25 02:19 server-cert.pem
-rw-------. 1 root root 1675 Jan 25 02:13 server-key.pem
-rw-r--r--. 1 root root 2404 Jan 25 02:15 server-request.pem
[client]# scp ca.pem server-cert.pem server-key.pem root@server
[client]# mkdir /etc/rsyslog-keys
[client]# cp ca.pem /etc/rsyslog-keys
[client]# ll /etc/rsyslog-keys
total 4
-rw-r--r--. 1 root root 1281 Jan 25 02:32 ca.pem
[client]# yum -y install rsyslog-gnutls
[client]# vim /etc/rsyslog.d/logging-client.conf
# certificate files - just CA for a client
$DefaultNetstreamDriverCAFile /etc/rsyslog-keys/ca.pem

# set up the action
$DefaultNetstreamDriver gtls            # use gtls netstream driver
$ActionSendStreamDriverMode 1           # require TLS for the connection
$ActionSendStreamDriverAuthMode anon    # server is NOT authenticated
*.info @@(o)serverX.example.com:6514    # send (all) messages of info and above priority
[client]# service rsyslog restart
Shutting down system logger: [  OK  ]
Starting system logger: [  OK  ]
[client]# logger "test1111111111111111111111111111111111"
[client]# logger "test2222222222222222222222222222222222"
[client]# tail -f /var/log/messages
Jan 25 02:54:35 cloud-qe-16-vm-04 root: test1111111111111111111111111111111111
Jan 25 02:55:29 cloud-qe-16-vm-04 root: test2222222222222222222222222222222222



[server]# mkdir /etc/rsyslog-keys
[server]# ll /etc/rsyslog-keys/
total 12
-rw-r--r--. 1 root root 1281 Jan 25 02:21 ca.pem
-rw-r--r--. 1 root root 1334 Jan 25 02:21 server-cert.pem
-rw-------. 1 root root 1675 Jan 25 02:21 server-key.pem
[server]# yum install -y rsyslog-gnutls
[server]# yum info rsyslog-gnutls
Loaded plugins: product-id, search-disabled-repos, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Installed Packages
Name        : rsyslog-gnutls
Arch        : x86_64
Version     : 5.8.10
Release     : 10.el6_6
Size        : 31 k
Repo        : installed
From repo   : beaker-Server
Summary     : TLS protocol support for rsyslog
URL         : http://www.rsyslog.com/
License     : (GPLv3+ and ASL 2.0)
Description : The rsyslog-gnutls package contains the rsyslog plugins that provide the
            : ability to receive syslog messages via upcoming syslog-transport-tls
            : IETF standard protocol.
[server]# rpm -ql rsyslog-gnutls
/lib64/rsyslog/lmnsd_gtls.so
[server]# man 5 rsyslog.conf
[server]# vim /etc/rsyslog.d/logging-server.conf
# make gtls driver the default
$DefaultNetstreamDriver gtls

# certificate files
$DefaultNetstreamDriverCAFile /etc/rsyslog-keys/ca.pem
$DefaultNetstreamDriverCertFile /etc/rsyslog-keys/server-cert.pem
$DefaultNetstreamDriverKeyFile /etc/rsyslog-keys/server-key.pem
$ModLoad imtcp                                  # load TCP listener
$InputTCPServerStreamDriverMode 1

# run driver in TLS-only mode
$InputTCPServerStreamDriverAuthMode anon        # client is NOT authenticated
$InputTCPServerRun 6514                         # listen on port 6514
[server]# service rsyslog restart
Shutting down system logger: [  OK  ]
Starting system logger: [  OK  ]
[server]# netstat -antlp | grep 6514
tcp        0      0 0.0.0.0:6514                0.0.0.0:*                   LISTEN      8374/rsyslogd       
tcp        0      0 :::6514                     :::*                        LISTEN      8374/rsyslogd 
[server]# tail -f /var/log/messages 
Jan 25 02:54:35 cloud-qe-16-vm-04 root: test1111111111111111111111111111111111
Jan 25 02:55:29 cloud-qe-16-vm-04 root: test2222222222222222222222222222222222

指定server端log输出到文件/var/log/client/messages， 并且不再输出在/var/log/messages 
[server]# vim /etc/rsyslog.d/remote-filter.conf
:fromhost-ip, isequal, "10.16.98.105"   /var/log/client/messages
:fromhost-ip, isequal, "10.16.98.105"   ~
[server]# service rsyslog restart
Shutting down system logger: [  OK  ]
Starting system logger: [  OK  ]
```

### Managing Log File Rotation

```
[server]# cat /etc/logrotate.conf | egrep -v "#|^$"
weekly
rotate 4
create
dateext
include /etc/logrotate.d
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}
/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

[server]# vim /etc/logrotate.d/client
/var/log/client/* {
    rotate 4
    size 2k
    compress
    missingok
}

[server]# cat /etc/cron.daily/logrotate 
#!/bin/sh
/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0

[server]# ls /var/log/client/
messages
[server]# /etc/cron.daily/logrotate 

```

###### Performance Checklist: Adjusting Log File Rotation Policy


### Unit Test: Implementing a Secure Log Server



### rsyslog

在 Linux 系统中，日志文件记录了系统中包括内核、服务和其它应用程序等在内的运行信息。在我们解决问题的时候，日志是非常有用的，它可以帮助我们快速的定位遇到的问题。

在 RHEL 6中，日志是使用rsyslogd守护进程进行管理的，该进程是之前版本的系统中syslogd的升级版，对原有的日志系统进行了功能的扩展，提供了诸如过滤器，日志加密保护，各种配置选项，输入输出模块，支持通过 TCP 或者 UDP 协议进行传输等。

rsyslog的配置文件为 /etc/rsyslog.conf, 大多数日志文件都位于 /var/log/ 目录中，在该目录中，你可能注意到很多日志文件末尾包含一串数字（如 maillog-20150301），这说明这些日志文件经过了日志转储，这样可以避免日志文件过大。

在软件包logrotate中包含了一个定时任务，根据/etc/logrotate.conf文件和/etc/logrotate.d/目录中的的配置定期的转储日志文件。

在配置文件中, 通过配置 filter 以及 action 对日志进行管理， rsyslog发现符合 filter 规则的日志后，会将日志发送到 action 指定的动作进行处理。

> filter        action

###### Filter

*    基于设施/优先级的过滤器 (Facility/Priority-based filters)
*    基于属性的过滤器
*    基于表达式的过滤器： 基于表达式的过滤器使用了rsyslog自定义的脚本语言RainerScript构建复杂的filter，这里暂时不对这种方法进行讲述。

######### 基于设施/优先级的过滤器

> FACILITY.PRIORITY

*    FACILITY指定了产生日志消息的子系统，可选值为 auth, authpriv, cron, daemon, kern, lpr, mail, news, syslog, user, ftp, uucp, local0 ~ local7
*    PRIORITY指定了日志消息的优先级，可用的优先级包含 debug (7), info (6), notice (5), warning (4), err (3), crit (2), alert (1), emerg (0)

```
前置符号 = 表明只有该优先级的消息会被捕获; ! 表明除了该优先级的消息之外的优先级会被捕获; 除了前置符号外，可以使用符号 *, 表示所有的设施或者优先级; 对优先级部分使用 none 关键字会捕获所有没有指定优先级的消息
定义多个设施或者优先级使用 , 分隔，如果是多个 filter 的话，则使用 ; 进行分隔。

kern.*                    # 选择所有优先级的内核日志
mail.crit                 # 选择所有mail 的优先级高于crit的日志
cron.!info,!debug         # 选择除了 info 和 debug 优先级的 cron 日志

*.info;mail.none;authpriv.none;cron.none
```

######### 基于属性的过滤器

> :PROPERTY, [!]COMPARE_OPERATION, "STRING"

> :PROPERTY是要比较的日志属性，COMPARE_OPERATION 为要执行的比较操作，这里的 ! 表示取反的意思，"STRING"为比较的值。

可以使用的比较操作：

|比较操作 	|描述                                                           |
|-----------|--------------------------------------------------------------|
|contains 	|匹配提供的字符串值是否是属性的一部分，如果不区分大小写, 使用contains_i |
|isequal 	|比较属性和值是否相等                                             |
|startswith |属性是否以指定字符串开始(startswith_i)                           |
|regex 	    |正则表达式(POSIX BRE 基本正则)匹配                               |
|ereregex 	|正则表达式(POSIX ERE 扩展正则)匹配                               |
|isempty 	|判断属性是否为空，不需要 value                                    |


```
:msg, contains, "error"
:hostname, isequal, "host1"
:msg, !regex, "fatal .* error"

```

###### Action 

Action定义了当匹配指定的 filter 的时候，执行什么操作。如果要指定多个 ACTION， 使用 & 连接多个 ACTION。

```
kern.=crit user1
& ^test-program;temp            -> 在 ACTION 后面追加 ;模板名称 可以为指定的 action 使用该模板格式化日志。
& @192.168.0.1
```

> 这里的 ;temp 指定了传递日志给 test-program 程序时（^ 开头表明日志发送给该可执行文件），使用它 temp 模板格式化日志。


######### 保存日志到日志文件

1. 静态记录日志

> FILTER    PATH

*    PATH 指定了日志要保存到的文件, 例如 cron.* /var/log/cron.log 指定了所有的定时任务日志都写入到/var/log/cron.log文件。
*    默认情况下，每次生成 syslog 的时候，日志信息会同步到日志文件。可以在文件路径前使用 - 指定忽略同步（如果系统崩溃，会丢失日志，但是这样可以提高日志性能）。

2. 动态生成的日志文件

> FILTER     ?DynamicFile

*    DynamicFile是预定义的输出路径模板，后面会讲到

3. 通过网络发送rsyslog

rsyslog可以使用网络将日志消息发送或者接受日志，使用这个特性，可以实现使用单一的日志服务器统一管理多台服务器日志。

> @[(z<NUMBER>,o)]HOST:[PORT]

*    @告诉syslog使用 UDP 协议发送日志，要使用 TCP 的话，使用 @@
*    可选值 zNUMBER 设置了是否允许使用zlib对日志压缩（压缩级别1-9）
*    可选值 o, This option is experimental. Use at your own risk and only if you know why you need it! If in doubt, do NOT turn it on.

```
*.* @192.168.0.1        # 使用 UDP 发送，默认端口514
*.* @@example.com:18    # 使用 TCP 发送到端口18， 默认10514
*.* @(z9)[2001:db8::1]  # UDP, ipv6，使用zlib级别9压缩
*.* @@(o,z9)192.168.0.1:1470     #In this example, messages are forwarded via plain TCP with experimental framing and maximum compression to the host 192.168.0.1 at port 1470.
```

4. 丢弃日志

要丢弃日志消息，使用~动作。

> FILTER    ~

```
cron.* ~
```


###### 模板

任何rsyslog生成的日志都可以根据需要使用模板进行格式化，要创建模板，使用如下指令

> $template TEMPLATE_NAME, "text %PROPERTY% more text", [OPTION]

*    $template指令表明了接下来的内容定义了一个模板
*    TEMPLATE_NAME 是模板的名称
*    接下来双引号之间的内容为模板的内容
*    OPTION, 它指定了模板的功能，支持选项为 sql 和 stdsql，在使用数据库存储的时候会用到

######### 生成动态文件名

模板可以用来生成动态文件名，就如之前所述，在使用动态文件名的时候，需要在 ACTION 中的模板名称前增加 ? 表明该文件名是动态生成的。

```
$template DynamicFile,"/var/log/test_logs/%timegenerated%-test.log"
*.* ?DynamicFile
```

> timegenerated 属性从日志信息中提取出消息的时间戳，这样可以为每个日志生成唯一文件名称。
    
######### 属性

在模板中使用的属性是在 % 之间的内容，使用属性可以访问日志消息中的内容。

> %PROPERTY_NAME[:FROM_CHAR:TO_CHAR:OPTION]%

可用的属性列表见 man rsyslog.conf

###### 全局指令

全局指令是rsyslogd守护进程的配置指令。所有的全局指令必须以 $ 开始，每行只能有一个指令，例如：

> $MainMsgQueueSize 50000

在新的配置格式中(rsyslog v6)，已经不在使用这种方式的指令，但是它们仍然是可用的。


[Viewing and Managing Log Files](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/ch-Viewing_and_Managing_Log_Files.html)
[Configuration](http://www.rsyslog.com/doc/v8-stable/configuration/)









