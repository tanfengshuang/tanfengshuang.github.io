---
layout: post
title:  "管理变量和 Inclusion（包含）(407-4)"
categories: Linux
tags: RHCA 407
---

### 管理变量

三个基本的范围级别：

*    全局范围：从命令行或 Ansible 配置设置的变量
*    Play 范围：在 play 和相关结构中设置的变量
*    主机范围：由清单、事实收集或注册的任务，在主机组和个别主机上设置的变量

如果在多个级别上定义了相同名称的变量，则采用级别高的变量。因此，由清单定义的变量将被 playbook 定义的变量覆盖，后者将被命令行中定义的变量覆盖。

###### playbook 中的变量

在编写 playbook 时，管理员可以使用自己的变量并在任务中调用它们

Playbook 变量可以通过多种方式定义。最简单的方式是将它放在 playbook 开头的 vars 块中：

```
- hosts: all
  vars:
    user: joe
    home: /home/joe
```

也可以在外部文件中定义 playbook 变量。此时不使用 vars，可以改为使用 vars_files 指令，后面跟上相对于 playbook 而言是外部的、应当要读取的变量文件列表：

```
- hosts: all
  vars_files:
    - vars/users.yml

$ cat vars/users.yml
user: joe
home: /home/joe
```

###### 主机变量和组变量

直接应用到主机的清单变量可以归为两个宽泛的类别： 主机变量应用到特定的主机，而 组变量则应用到某一或某组主机组中的所有主机。主机变量优先于组变量，但 playbook 中定义的变量的优先级比这两者更高。

1. 若要定义主机变量和组变量，一种方法是直接在清单文件中定义。这是较旧的做法，不建议采用，但用户可能会遇到. 此做法存在诸多缺点；例如，它使得清单文件更难以处理，在同一文件中混合提供了主机和变量信息，采用的也是过时的语法。
    
```
以下是主机变量 ansible_user，为主机 demo.example.com 而定义。
[servers]
demo.example.com  ansible_user=joe

在本例中，组变量 user 是为 servers 组定义的。

[servers]
demo1.example.com
demo2.example.com

[servers:vars]
user=joe
```

2. 使用 group_vars 和 host_vars 目录: 首选的做法, 是在与清单文件或目录相同的工作目录中，创建两个目录 group_vars 和 host_vars。这两个目录分别包含用于定义组变量和主机变量的文件。

```
$ cat ~/project/inventory
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenters:children]
datacenter1
datacenter2

如果需要为两个数据中心的所有服务器定义一个通用值，可以为 datacenters 设置一个组变量：
$ cat ~/project/group_vars/datacenters
package: httpd

如果要为两个数据中心定义不同的值，可以为每个数据中心设置组变量：
$ cat ~/project/group_vars/datacenter1
package: httpd
$ cat ~/project/group_vars/datacenter2
package: apache

如果要为每个数据中心中的每一主机定义不同的值，则建议使用主机变量：
$ cat ~/project/host_vars/demo1.example.com
package: httpd
$ cat ~/project/host_vars/demo2.example.com
package: apache
$ cat ~/project/host_vars/demo3.example.com
package: mariadb-server
$ cat ~/project/host_vars/demo4.example.com
package: mysql-server
```

###### 覆盖来自命令行的变量

清单变量可被 playbook 中设置的变量覆盖，这两种变量又可通过在命令行中传递参数到 ansible 或 ansible-playbook 命令来覆盖。如果在某一次运行 playbook 时需要针对单个主机覆盖为变量定义的值，可以使用这种做法。例如：

```
$ ansible-playbook demo2.example.com main.yml -e "package=apache"
```

###### 变量和数组

```
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
```

然后使用下列变量来访问用户：
1. users.bjones.first_name      # Returns 'Bob'
2. users.acook.home_dir         # Returns '/users/acook'

由于变量被定义为 Python 字典，因此也可使用另一种语法。
1. users['bjones']['first_name']        # Returns 'Bob'
2. users['acook']['home_dir']           # Returns '/users/acook'


###### 注册的变量

管理员可以使用 register 语句捕获命令的输出。输出保存在一个变量中，稍后可用于调试用途或者达成其他目的，例如基于命令输出的特定配置。

以下 playbook 演示了如何为调试用途捕获命令的输出, debug 模块用于将 install_result 注册变量的值转储到终端。

```
---
- name: Installs a package and prints the result
  hosts: all
  tasks:
    - name: Install the package
      yum:
        name: httpd
        state: installed
      register: install_result

    - debug: var=install_result
```


### 管理事实

###### Ansible 事实

Ansible 事实是 Ansible 从受管主机自动探查到的变量。事实由 setup 模块调取，其中包含的有用信息存储到可供管理员重复使用的变量中。Ansible 事实可以成为 playbook 的一部分，依据受管主机的值放置在条件、循环或任何其他动态语句中；例如：

*    可以根据当前内核版本来重新启动服务器。
*    可以根据可用的内存来自定义 MySQL 配置文件。
*    可以根据主机名称来创建用户。 

借助 Ansible 事实，可以方便地检索受管节点的状态，并根据其状态决定要执行的操作。事实提供与如下相关的信息

*    主机名称
*    内核版本
*    网络接口
*    IP 地址
*    操作系统版本
*    各种环境变量
*    CPU 数量
*    提供的或可用的内存
*    可用磁盘空间

```
$ ansible demo1.example.com -m setup
ansible demo1.example.com -m setup
demo1.example.com | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.25.250.10"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::5054:ff:fe00:fa0a"
        ],
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "01/01/2011",
        "ansible_bios_version": "0.5.1",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/boot/vmlinuz-3.10.0-327.el7.x86_64",
            "LANG": "en_US.UTF-8",
            "console": "ttyS0,115200n8",
            "crashkernel": "auto",
            "net.ifnames": "0",
            "no_timer_check": true,
            "ro": true,
            "root": "UUID=2460ab6e-e869-4011-acae-31b2e8c05a3b"
        }
... Output omitted ...
```

Ansible 事实:

*    事实 	变量
*    主机名称 	                                    {{ ansible_hostname }}
*    主要 IPv4 地址（基于路由） 	                    {{ ansible_default_ipv4.address }}
*    主磁盘第一分区大小（基于磁盘名称，如 vda、vdb，等等） 	{{ ansible_devices.vda.partitions.vda1.size }}
*    DNS 服务器 	                                    {{ ansible_dns.nameservers }}
*    内核版本 	                                    {{ ansible_kernel }}

###### 事实过滤器

Ansible 事实包含与系统相关的广泛信息。管理员可以使用 Ansible 过滤器，在从受管节点收集事实时限制返回的结果。过滤器可用于：

*    仅检索与网卡相关的信息。
*    仅检索与磁盘相关的信息。
*    仅检索与用户相关的信息。

若要使用过滤器，需要利用 -a 'filter=EXPRESSION' 将表达式作为选项来传递。例如，若要仅返回关于 eth0 的信息，可以对 ansible_eth0 元素应用过滤器： 

```
$ ansible demo1.example.com -m setup -a 'filter=ansible_eth0'
demo1.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "ansible_eth0": {
            "active": true,
            "device": "eth0",
            "ipv4": {
                "address": "172.25.250.10",
                "broadcast": "172.25.250.255",
                "netmask": "255.255.255.0",
                "network": "172.25.250.0"
            },
            "ipv6": [
                {
                    "address": "fe80::5054:ff:fe00:fa0a",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "52:54:00:00:fa:0a",
            "module": "virtio_net",
            "mtu": 1500,
            "pciid": "virtio0",
            "promisc": false,
            "type": "ether"
        }
    },
    "changed": false
}
```

如果管理了大量服务器，管理员可以手动禁用受管主机的事实。要禁用事实，可在 playbook 中将 gather_facts 设为 no：

```
---
- hosts: large_farm
  gather_facts: no
```

###### 自定义事实

管理员可以自行创建事实，将它们推送到受管节点。创建后，自定义事实将由 setup 模块集成和读取。自定义事实可用于：

*    基于自定义脚本定义系统的特定值。
*    基于程序执行定义值。

自定义事实的文件保存在 /etc/ansible/facts.d 目录中，文件的扩展名必须为 .fact。事实文件是采用 INI 或 JSON 格式的纯文本文件。INI 事实文件包含由一部分定义的顶层值，后跟用于待定义的事实的键值对：

```
[packages]
web_package = httpd
db_package = mariadb-server

[users]
user1 = joe
user2 = jane
```

Ansible 对两种格式返回相同结果，而且结果置于 ansible_local 层中。接着可利用过滤器确保成功安装并检索了自定义事实：

```
$ ansible demo1.example.com -m setup -a 'filter=ansible_local'
demo1.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "custom": {
                "packages": {
                    "db_package": "mariadb-server",
                    "web_package": "httpd"
                },
                "users": {
                    "user1": "joe",
                    "user2": "jane"
                }
            }
        }
    },
    "changed": false
}
```

自定义事实的使用方式与 playbook 中的默认事实一致。

```
---
- hosts: all
  tasks:
  - name: Prints various Ansible facts
    debug:
      msg: The package to install on {{ ansible_fqdn }} is {{ ansible_local.custom.packages.web_package }}
```

### 管理包含

1. 可以通过 include 指令将外部文件中的任务包含到 playbook 中。

```
tasks:
  - name: Include tasks to install the database server
    include: tasks/db_server.yml
```

2. include_vars 模块可以包含 JSON 或 YAML 文件中定义的变量，覆盖已定义的主机变量和 playbook 变量。

```
 tasks:
  - name: Include the variables from a YAML or JSON file
    include_vars: vars/variables.yml
```

###### 包含任务

在 playbook 中使用 include 指令来指定将什么任务文件包含在 playbook 中的哪一个点上。可以使用 vars 指令设置由任务文件使用的变量，并覆盖 playbook 变量、清单变量、注册的变量和任务文件中定义的事实。

```
$ cat environment.yml
- name: Installs the {{ package }} package
  yum:
    name: "{{ package }}"
    state: latest

- name: Starts the {{ service }} service
  service:
    name: "{{ service }}"
    state:  "{{ state }}"
    

$ cat playbook.yml
---
- name: Install, start, and enable services
  hosts: all
  tasks:
  - name: Includes the tasks file and defines the variables
    include: environment.yml
    vars:
      package: mariadb-server
      service: mariadb
      state: started
    register: output

  - name: Debugs the included tasks
    debug:
      var: output
```

###### 包含变量

一些设置变量的方式包括：

*    在清单文件或 host_vars 和 group_vars 目录的外部文件中定义清单变量
*    事实和注册的变量
*    通过 vars 在 playbook 文件中定义 playbook 变量，或者通过 vars_files 在外部文件中定义
*    include_vars 模块是另一种方式，可以在 playbook 中设置来自外部文件的变量。此方式的特别之处在于，它是通过模块来执行的，而且会覆盖利用上述方法设定的任何值。在结合使用设置了您想要覆盖的变量值的任务文件时，或者联合条件执行来仅在一定条件下设置特定值时，可以使用这种方式。

```
$ cat variables.yml
---
packages:
  web_package: httpd
  db_package: mariadb-server
  
$ cat playbook.yml
---
- name: Install web application packages
  hosts: all
  tasks:
  - name: Includes the tasks file and defines the variables
    include_vars: variables.yml

  - name: Debugs the variables imported
    debug:
      msg: >
        "{{ packages['web_package'] }} and {{ packages.db_package }} 
        have been imported"  
```


### Summary

*    Ansible 变量让管理员能够在整个 Ansible 项目中跨文件重复利用值
*    变量的名称由字符串组成，它必须以字母开头，并且只能含有字母、数字和下划线
*    可以在清单中为主机和主机组定义变量，可以为 playbook 定义变量，还可以通过事实和外部文件以及从命令行定义变量
*    清单变量最好存储在相对于清单的 host_vars 和 group_vars 目录内的文件中，而不要存储在清单文件本身中
*    Ansible 事实是 Ansible 从受管主机自动探查到的变量
*    在 playbook 中，当变量用作值的开头时，必须要使用引号
*    register 关键字可用于在变量中捕获命令的输出。
*    include 和 include_vars 模块可用于将 YAML 或 JSON 格式的任务或变量文件包含在 playbook 中。
















