---
layout: post
title:  "实施 playbook(407-3)"
categories: Linux
tags: RHCA 407
---


### 编写 YAML 文件。


###### YAML 语法

1. 文件开始和结束标志

```
---
...output omitted...
...
```

2. 字符串: YAML 中的字符串不要求放在引号里，即使字符串中包含空格。如果需要，字符串可以用双引号或单引号括起。 

编写多行字符串有两种方式

```
include_newlines: |
          Example Company
          123 Main Street
          Atlanta, GA 30303
          
fold_newlines: >
          This is
          a very long,
          long, long, long
          sentence.          
```

3. 字典

```
key: value
内嵌块格式: {name: Automation using Ansible, code: DO407}
```

4. 列表

```
 - red
 - green
 - blue
 
内嵌块格式: fruits:  [red, green, blue]
```

5. 注释: 注释也可以用于提高可读性。在 YAML 中，注释以一个井号字符 (#) 开头，可存在于任何行（空行或非空行）的末尾。如果在非空行中使用，则要在井号之前加一个空格。 

###### 验证 YAML 语法

1. Python

> python -c 'import yaml, sys; print yaml.load(sys.stdin)' < myyaml.yml

2. YAML Lint: 对于不熟悉 Python 的管理员，网上提供了许多 YAML 语法验证工具。其中一个例子是 YAML Lint网站。将 playbook 中的 YAML 内容复制并粘贴到首页上的表单中，然后提交该表单。网页将报告语法验证的结果，同时显示原始提交内容的格式化版本。 
3. Ansible 命令行工具: Ansible 提供了一个原生功能，供管理员用于验证 playbook 中的 YAML 语法。ansible-playbook 命令用于执行 playbook，它含有可检查语法错误的 --syntax-check 选项。 


### 实施 Ansible playbook。

###### 模块简介

Ansible 有三种类型的模块：

*    核心模块随 Ansible 提供，由 Ansible 开发团队编写和维护。核心模块是最为重要的模块，用于常见的管理任务。
*    附加模块目前随 Ansible 提供，但未来可能会升级为核心模块或者另外提供。负责维护它们的通常不是 Ansible 团队，而是社区。通常而言，这些模块实施用于管理 OpenStack 等更新技术的功能。
*    自定义模块是由最终用户开发的模块，不随 Ansible 提供。如果没有现有的模块适合某一项任务，管理员可以编写新的模块来实施。 

核心模块和附加模块始终都能使用。Ansible 在控制节点上由 $ANSIBLE_LIBRARY 环境变量定义的目录中查找自定义模块；或者，如果未设置此变量，则由当前 Ansible 配置文件中的 library参数定义。Ansible 也会在相对于 playbook 使用位置的 ./library 目录中查找模块。

> library = /usr/share/my_modules/


###### 模块类别

为了更好地进行组织和管理，Ansible 模块分组为下列功能类别。

*    云
*    群集
*    命令
*    数据库
*    文件
*    清单
*    消息
*    监控
*    网络
*    通知
*    打包
*    源控制
*    系统
*    实用程序
*    Web 基础架构
*    窗口 

在红帽企业 Linux 7 系统上，模块安装在 /usr/lib/python2.7/site-packages/ansible/modules 目录中。核心模块和附加模块存储在单独的目录中。这两个目录中的模块分别组织在类别子目录中。 

###### 模块文档

1. 可以查阅 Ansible 文档网站 docs.ansible.com。借助网站上的模块索引，管理员可以搜索适用于不同功能的模块。例如，适于用户和服务管理的模块可以在 Systems Modules 下找到，而适合数据库管理的模块则可在 Database Modules 下找到。 
2. 模块文档也在 Ansible 控制节点上本地可用，可以使用 ansible-doc 命令访问。

*    要查看控制节点的可用模块的列表，可运行 ansible-doc -l 命令。这将显示模块名称列表以及其功能的概要。
*    若要显示特定模块的详细文档，可将模块名称传递至 ansible-doc, 例如ansible-doc yum。
*    ansible-doc 命令还提供 -s（或 --snippet）选项，它会生成示例输出，可以充当如何在 playbook 中使用特定模块的示范。此输出可以作为起步模板，包含在实施该模块以执行任务的 playbook 中。输出中包含的注释，提醒管理员各个选项的用法。

###### 调用模块

1. 可以使用 ansible 命令，作为临时命令的一部分来调用模块。管理员可利用 -m 选项指定要使用的模块名称。以下命令使用 ping 模块测试与所有受管主机的连接。此模块连接受管主机并进行身份验证，然后检验是否满足 Ansible 的 Python 要求。

    $ ansible -m ping all
              
2. 也可以在 playbook 中作为 task 的一部分来调用模块。下列片段演示了如何将软件包名称及其所需状态作为参数来调用 yum 模块。

```
    tasks:
    - name: Installs a package
      yum:
        name: postfix
        state: latest
```

###### 演示：探索 Ansible 模块

本演示将使用 Ansible 临时命令来展示利用 yum、service 和 uri 核心模块进行的一些常见操作。

```
$ cat inventory
servera.lab.example.com

$ cat ansible.cfg
[defaults]
inventory=inventory
remote_user=devops

[privilege_escalation]
become=False
become_method=sudo
become_user=root
become_ask_pass=False

$ ansible servera.lab.example.com --list-hosts
  hosts (1):
    servera.lab.example.com
    
$ ansible servera.lab.example.com -m ping
servera.lab.example.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

$ ansible servera.lab.example.com -m yum -a "name=httpd state=latest"
servera.lab.example.com | FAILED! => {
    "changed": true,
    "failed": true,
    "msg": "You need to be root to perform this command.\n",
    "rc": 1,
    "results": [
        "Loaded plugins: langpacks, search-disabled-repos\n"
    ]
}

$ ansible servera.lab.example.com -m yum -a "name=httpd state=latest" -b
servera.lab.example.com | SUCCESS => {
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
...output omitted...
    ]
}

$ ansible servera.lab.example.com -m service -a "name=httpd enabled=yes state=started" -b
servera.lab.example.com | SUCCESS => {
    "changed": true,
    "enabled": true,
    "name": "httpd",
    "state": "started"
}

$ ansible localhost -m uri -a "url=http://materials.example.com/"
localhost | SUCCESS => {
"changed": false,
"content_length": "2457",
"content_location": "http://materials.example.com/",
"content_type": "text/html;charset=ISO-8859-1",
"date": "Mon, 18 Apr 2016 19:10:28 GMT",
"redirected": false,
"server": "Apache/2.4.6 (Red Hat Enterprise Linux)",
"status": 200
}
```

### 编写和执行 playbook。

###### Playbook 基础知识

Playbook 是以 YAML 格式编写的文本文件。与 Puppet 等其他配置管理工具采用的语言相比，playbook 中使用的语法更加容易编写和掌握。

Playbook 名称用的是运动类比，一个 playbook 中包含一组 play，每一个 play 是可以执行的标准化计划。Ansible playbook 也包含 play，每个 play 负责定义要对特定的一组受管主机执行的一系列操作。这些操作称为 任务，而受管主机则称为 主机。调用 Ansible 模块并向它们传递完成所需操作而需要的参数，从而执行任务。 

在编写 Ansible playbook 时应注意，如果受管主机已在正确的状态，则任务不去进行不必要的更改。换而言之，如果 playbook 运行一次即可将主机置于正确的状态，则该 playbook 的编写应当使它运行第二次也安全，不会对配置的系统进行进一步的更改。具有此属性的 playbook 是 幂等的。大部分 Ansible 模块是幂等的，所以确保符合此要求相对容易。

每一 playbook 可以包含一个或多个 play。因为一个 play 将一系列任务应用到一组主机，所以在需要对一组主机执行一系列任务，而对另一组主机执行另一系列的任务时，就需要多个 play 来实现。 

```
---
# This is a simple playbook with a single play
-  name: a simple play
   hosts: managedhost.example.com
   user: remoteuser
   become: yes
   become_method: sudo
   become_user: root
   tasks:
   - name: first task
     service: name=httpd enabled=true
   - name: second task
     service: name=sshd enabled=true
...
```

command、shell 和 raw 模块的用法貌似简单，但在可能时，应尽量避免在 playbook 中使用它们。因为它们可以取用任意命令，因此使用这些模块时很容易写出非幂等的 playbook。

例如，以下使用 shell 模块的任务为非幂等。每次运行 play 时，它都会重写 /etc/resolv.conf，即使它已经包含了行“nameserver 192.0.2.1”。

```
- name: Non-idempotent approach with shell module
  shell: echo "nameserver 192.0.2.1" > /etc/resolv.conf
```

以幂等方式使用 shell 模块可以执行许多事务，而且有时候进行这些更改并使用 shell 是最佳的做法。但更快的方案或许是使用 ansible-doc 的 copy 模块，再使用它获得所需的效果。 在下例中，如果 /etc/resolv.conf 文件已包含正确的内容，则不会重写该文件：

```
- name: Idempotent approach with copy module
  copy:
    dest: /etc/resolv.conf
    content: "nameserver 192.0.2.1\n"
```

copy 模块具有专门的用途，可以轻松测试来了解是否达到了需要的状态，如果已达到，则不进行任何更改。shell 模块容许非常大的灵活性，但需要格外小心，从而确保它以幂等方式运行。

幂等的 playbook 可以重复运行，确保系统处于特定的状态，而不会破坏状态已经正确的系统。

###### 执行 playbook

```
$ ansible-playbook webserver.yml

语法验证
$ ansible-playbook --syntax-check webserver.yml

执行空运行 -C 选项。这会使 Ansible 报告在执行该 playbook 时将会发生什么更改，但不会对受管主机进行任何实际的更改。 
$ ansible-playbook -C webserver.yml

逐步执行 - 用户可以选择“y”执行任务，选择“n”跳过任务，或者选择“c”退出逐步执行并以非交互方式执行剩余的任务。 
$ ansible-playbook --step webserver.yml

```

###### 演示：实施 Ansible Playbook

```
$ cat inventory
localhost
servera.lab.example.com

$ cat ansible.cfg
[defaults]
inventory=inventory
remote_user=devops

[privilege_escalation]
become=False
become_method=sudo
become_user=root
become_ask_pass=False

$ vim intranet.yml
---
- name: intranet services
  hosts: servera.lab.example.com
  become: yes
  tasks:
  - block:
    - name: latest httpd version installed
      yum:
        name: httpd
        state: latest
    - name: latest firewalld version installed
      yum:
        name: firewalld
        state: latest
  - block:
    - name: firewalld permits http service
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes
  - block:
    - name: httpd enabled and running
      service:
        name: httpd
        enabled: true
        state: started
    - name: firewalld enabled and running
      service:
        name: firewalld
        enabled: true
        state: started
  - block:
    - name: test html page
      copy:
        content: "Welcome to the example.com intranet!\n"
        dest: /var/www/html/index.html
- name: test
  hosts: localhost
  tasks:
  - name: connect to intranet
    uri:
      url: http://servera.lab.example.com
      status_code: 200

$ ansible-playbook --syntax-check intranet.yml
playbook: intranet.yml

$ ansible-playbook intranet.yml
```

### 实验

```
---
- name: internet services
  hosts: serverb.lab.example.com
  become: yes
  tasks:
  - block:
    - name: latest httpd version installed
      yum:
        name: httpd
        state: latest
    - name: latest firewalld version installed
      yum:
        name: firewalld
        state: latest
    - name: latest mariadb-server version installed
      yum:
        name: mariadb-server
        state: latest
    - name: latest php version installed
      yum:
        name: php
        state: latest
    - name: latest php-mysql version installed
      yum:
        name: php-mysql
        state: latest
  - block:
    - name: firewalld permits http service
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes
  - block:
    - name: httpd enabled and running
      service:
        name: httpd
        enabled: true
        state: started
    - name: mariadb enabled and running
      service:
        name: mariadb
        enabled: true
        state: started
    - name: firewalld enabled and running
      service:
        name: firewalld
        enabled: true
        state: started
  - block:
    - name: get test php page
      get_url:
        url: "http://materials.example.com/grading/var/www/html/index.php"
        dest: /var/www/html/index.php
        mode: 0644
- name: test
  hosts: localhost
  tasks:
  - name: connect to internet web server
    uri:
      url: http://serverb.lab.example.com
      status_code: 200
```

### Summary

*    Ansible playbook用于描述配置受管主机所需的标准任务
*    Ansible playbook 以 YAML 格式编写
*    YAML 文件的结构中使用空格缩进来代表数据层次结构
*    任务的实施使用标准化代码，这些代码打包为 Ansible 模块
*    ansible-doc 命令可以列出已安装的模块，同时提供相关文档以及如何在 playbook 中使用模块的示例代码片段
*    ansible-playbook 命令用于验证 playbook 语法并执行 playbook
