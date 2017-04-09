---
layout: post
title:  "介绍 Ansible(407-1)"
categories: Linux
tags: RHCA 407
---

### Ansible 架构概述

Ansible 架构中有两种计算机类型，即 控制节点和 受管主机。Ansible 软件安装在控制节点上，其所有组件也在控制节点上维护。受管主机列在 主机清单中；这是位于控制节点上的一个文本文件，其中含有受管主机名称或 IP 地址的列表。

系统管理员登录控制节点并启动 Ansible，为它提供 playbook和要管理的目标主机。除了控制单个系统外，也可以指定主机组或通配符。Ansible 使用 SSH 作为网络传输方式与受管主机进行通信。playbook 中引用的模块复制到受管主机。然后，使用 playbook 中指定的参数，按照顺序执行它们。Ansible 用户可以根据需要自行编写 自定义模块，但 Ansible 附带的核心模块能够执行大部分系统管理任务。 

###### Ansible 控制节点组件

*    组件 	描述
*    Ansible 配置 	Ansible 通过配置设置定义它的行为。这些设置包括执行命令的远程用户，以及利用 sudo 执行远程命令时要提供的密码，等等。默认配置值可以由环境变量或配置文件中定义的值覆盖。
*    主机清单 	        Ansible 主机清单定义主机从属于的配置组。清单可定义 Ansible 与受管主机的通信方式，还可定义主机和组变量值。
*    核心模块        	核心模块是与 Ansible 捆绑提供的模块，共计有 400 多个核心模块。
*    自定义模块 	    用户可以自行编写模块并添加到 Ansible 库中，来扩展 Ansible 的功能。模块通常使用 Python 编写，但也可使用任何解释型编程语言编写，如 shell、Ruby 和 Python 等。
*    Playbook 	    Ansible playbook 是采用 YAML 语法编写的文件，它们通过参数定义要应用到受管节点的模块。它们声明需要执行的任务。
*    连接插件 	        插件实现与受管主机或云提供商的通信。其包括原生的 SSH、paramiko SSH 和 local。Paramiko 是 OpenSSH 面向红帽企业 Linux 6 的 Python 实施，提供可改进 Ansible 性能的 ControlPersist 设置。
*    插件 	        增强 Ansible 功能的扩展，如电子邮件通知和日志记录。 

### Ansible 部署概述

###### Ansible 编配方法

Ansible 常被用于完成应用服务器调配。例如，可以通过编写 playbook 在新安装的基本系统上执行下列步骤：

1. 配置软件存储库。
2. 安装应用。
3. 调节配置文件。从版本控制系统选择性下载内容。
4. 在防火墙中打开必要的服务端口。
5. 启动相关的服务。
6. 测试应用并确认其正常工作。
7. 编配零停机时间滚动更新

Ansible 也是用于并行更新应用的简单工具。例如，可以通过开发 playbook 在应用服务器上执行下列步骤：

1. 停止系统和应用监控。
2. 从负载平衡中移除服务器。
3. 停止相关的服务。
4. 部署或更新应用。
5. 启动相关的服务。
6. 确认服务可用，并将服务器重新添加到负载平衡中。
7. 启动系统和应用监控。

可以使用 serial 关键字来限制一次性运行 playbook 的主机数量。一旦这一部分的服务器完成部署并且功能正常，Ansible 将移到目标组中的另一批服务器。默认情况下，Ansible 将尝试并行应用 playbook 到目标受管服务器，其生成的并行进程的确切数量由适用 ansible.cfg 配置文件内 forks 指令控制。 

###### Ansible 连接插件

### 描述 Ansible 清单 

###### 静态主机清单

主机清单文件的默认位置是 /etc/ansible/hosts。ansible* 命令与 --inventory PATHNAME 选项（简写为 -i PATHNAME）搭配时，将使用不同的主机清单文件。 

```
[webservers]
localhost             ansible_connection=local
web1.example.com
web2.example.com:1234 ansible_connection=ssh ansible_user=ftaylor
192.168.3.7
 
[db-servers]
web1.example.com
db1.example.com
```

```
[olympia]
washington1.example.com
washington2.example.com
 
[salem]
oregon01.example.com
oregon02.example.com
 
[nwcapitols:children]
olympia
salem
```

###### 动态主机清单

也可以动态生成 Ansible 主机清单信息。动态清单信息的来源包括公共/私有云提供商、Cobbler 系统信息和 LDAP 数据库，或者配置管理数据库 (CMDB)。Ansible 含有处理来自最常见提供商的动态主机、组和变量信息的脚本，如 Amazon EC2、Cobbler、Rackspace Cloud 和 OpenStack 等提供商。对于云提供商，必须在脚本能够访问的文件中定义身份验证和访问权限信息。

### Summary

*    Ansible 是一款基于 Python 的无代理配置管理工具。
*    Ansible 及其配置文件仅需要放在用于运行 Ansible 命令的系统上，本课程中将此系统称为 控制节点。
*    Ansible 主机清单文件是类似于 INI 的文件，用于标识 Ansible 管理的主机和主机组。
*    Ansible 将 模块从控制节点复制到 受管主机上，并在那里按照 Ansible playbook中指定的顺序执行。
*    原生 SSH 是 Ansible 使用的首选连接插件，但 Paramiko 插件能够使红帽企业 Linux 6 控制节点与受管主机进行高效率通信。 
