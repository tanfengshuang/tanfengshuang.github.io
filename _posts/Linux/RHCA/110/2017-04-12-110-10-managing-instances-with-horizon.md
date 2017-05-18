---
layout: post
title:  "使用 Horizon 管理实例(110-10)"
categories: Linux
tags: RHCA 110
---

### 使用 cloud-init 自定义实例

###### cloud-init

cloud-init 是处理实例初期初始化的软件。安装此软件后，管理员可将它用于：

*    设定默认的区域设置。
*    更新实例主机名。
*    生成或注入 SSH 私钥。
*    设置临时挂载点。

cloud-init 可以通过 user-data 调用。user-data 是用户提供的数据，在实例启动时由 cloud-init 读取并解析，从而对实例进行自定义。OpenStack 通过 cloud-init 实现实例管理；用户可以生成实例，再使用 Horizon 中的创建后(Post-Creation)选项卡来指定要应用到实例的自定义。 OpenStack 随后将信息转换为可由 cloud-init 读取的格式。

###### cloud-init 支持多种格式：

cloud-init 格式

*    格式 	            描述
*    使用 gzip 压缩的数据 	先解压缩数据，然后读取。管理员可以使用 gzip 压缩的数据来发送大小超过 16384 字节（user-data 大小限制）的信息。
*    Mime 多部件存档 	    管理员可以使用多部件文件来指定多种类型的数据。例如，可以同时指定用户数据脚本和 cloud-config 类型。
*    user-data 脚本 	    以 #! 或 Content-Type: text/x-shellscript 开头的脚本。该脚本将于实例第一次引导期间在 rc.local 级别执行。
*    include 文件 	    数据声明必须以 #include 或 Content-Type: text/x-include-url 开头。此声明指定要包含的文件。文件中包含 URL 列表，每行一个。每一个 URL 都将被读取，其内容将通过同一组规则进行传递；即，从 URL 读取的内容可以是 gzip 压缩数据、MIME 多部件或纯文本。
*    cloud-config 数据 	数据声明必须以 #cloud-config 或 Content-Type: text/cloud-config 开头。
*    upstart 作业 	    数据声明必须以 #upstart-job 或 Content-Type: text/upstart-job 开头。该内容将放入 /etc/init 下的文件中，与任何其他的 upstart 脚本一起供 upstart 使用。
*    部件处理程序 	    数据声明必须以 #part-handler 或 Content-Type: text/part-handler 开头。该数据将根据其文件名写入到 /var/lib/cloud/data 下的文件中。此处理程序必须用 Python 编写。

###### user-data 脚本

user-data scripts 提供了一种便捷的方式，以供在实例创建时向实例发送指令集合。该脚本将在 rc.local 级别调用，这是最后一个级别。下列脚本将使用 Bash 脚本更改当日消息：

```
#!/bin/bash
echo "This instance has been customized by cloud-init at $(date -R)!" >> /etc/motd
```

管理员可以使用 user-data 脚本安装新的软件包，更改系统设置，更新系统，以及确保文件存在等。

###### cloud-config

尽管管理员可以使用脚本来自定义实例，cloud-config 语法包含了各种可以使用的指令。借助 cloud-config 语法，管理员可以用友好的格式指定指令。此类指令包括：

*    在第一次引导时使用 Yum 更新系统。
*    添加新的 Yum 存储库。
*    更新现有的 Yum 存储库。
*    导入 SSH 密钥。
*    创建用户.
*    etc.

> 文件必须使用 YAML 语法，以便它能够被 cloud-init 解析并执行。

文件必须使用有效的 YAML 语法。下例演示如何通过添加新的用户并运行一组命令来自定义系统。

```
#cloud-config
groups:
  - cloud-users: [john,doe]

users:
  - default
  - name: barfoo
    gecos: Bar B. Foo
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    ssh-import-id: None
    lock-passwd: true
    ssh-authorized-keys:
      - <SSH public key>

runcmd:
 - [ wget, "http://materials.example.com", -O, /tmp/index.html ]
 - [ sh, -xc, "echo $(date) ': hello world!'" ]
```

```
[student@workstation ~]$ ssh -i ~/Downloads/demo_key1.pem cloud-user@172.25.250.27
[cloud-user@demo-testing-script ~]$ systemctl status cloud-init
cloud-init.service - Initial cloud-init job (metadata service crawler)
   Loaded: loaded (/usr/lib/systemd/system/cloud-init.service; enabled)
   Active: active  (exited) since Wed 2015-11-11 12:30:31 EST; 1min 9s ago
  Process: 700 ExecStart=/usr/bin/cloud-init init (code=exited, status=0/SUCCESS)
 Main PID: 700 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/cloud-init.service

cloud-init 将其操作记录在日志文件 /var/log/cloud-init.log 中。打开该文件，显示到最后一行；应该会看到指示 httpd 已经安装的行。完成后，注销。
[cloud-user@demo-testing-script ~]$ tail /var/log/cloud-init.log
...
cloud-init: Installed:
cloud-init: httpd.x86_64 0:2.4.6-31.el7_1.1
cloud-init: Dependency Installed:
cloud-init: apr.x86_64 0:1.4.8-3.el7                   apr-util.x86_64 0:1.5.2-6.el7
cloud-init: httpd-tools.x86_64 0:2.4.6-31.el7_1.1      mailcap.noarch 0:2.1.41-2.el7
cloud-init: Complete!
[cloud-user@demo-testing-script ~]$ exit
```

### 验证实例自定义

实例生成之后，管理员可以确保 cloud-init 指令是否成功执行。自定义可能包括：

*    安装软件包。
*    移除软件包。
*    更新系统。
*    创建用户或组。
*    检索文件。
*    etc.

### 总结

在本章中，您可以学到：

*    cloud-init 是处理实例初期初始化的软件。安装该软件后，管理员可以用它来设置默认的区域设置，更新实例主机名，并且生成或注入 SSH 私钥等。
*    OpenStack 会将信息转换为可由 cloud-init 读取的格式；实例引导时将执行该指令。
*    cloud-init 支持各种格式，如 gzip 压缩数据、user-data 脚本和 cloud-config 数据，以及众多其他格式。
*    借助 cloud-config，管理员可以用户友好的格式指定指令。此类指令包括使用 yum 在第一次引导时更新系统，添加新的 yum 存储库，更新现有的 yum 存储库以及导入 SSH 密钥等。
*    cloud-init 会生成日志，可用于对实例引导过程中可能出现的任何问题进行故障排除。
