---
layout: post
title:  "实施任务控制(407-5)"
categories: Linux
tags: RHCA 407
---

### 构建流控制

###### Ansible 循环

1. 简单循环

```
- yum:
    name: "{{ item }}"
    state: latest
  with_items:
    - postfix
    - dovecot
```

```
vars:
  mail_services:
    - postfix
    - dovecot

tasks:
  - yum:
      name: "{{ item }}"
      state: latest
    with_items: "{{ mail_services }}"
```

2. 散列列表：将数组作为参数传递时，数组可以是散列列表。下列代码片段演示了如何将一个多维数组（具有键对值的数组）传递给 user 模块以自定义其名称及组： 

```
- user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  with_items:
    - { name: 'jane', groups: 'wheel' }
    - { name: 'joe', groups: 'root' }
```


3. 嵌套循环：管理员也可以使用嵌套循环，即在循环内部通过 with_nested 关键字调用的循环。使用嵌套循环时，Ansible 将迭代第一个数组，只要其中包含有值。例如，当多名用户需要多种 MySQL 特权时，管理员可以创建一个多维数组，并通过 with_nested 关键字调用。以下片段说明了如何在嵌套循环中使用 mysql_user 模块以确保两名用户拥有一组三项特权：

```
- mysql_user:
    name: "{{ item[0] }}"
    priv: "{{ item[1] }}.*:ALL"
    append_privs: yes
    password: redhat

  with_nested:
    - [ 'joe', 'jane' ]
    - [ 'clientdb', 'employeedb', 'providerdb' ]
```

*    循环关键字 	            描述
*    with_file 	            取控制节点文件名列表。item 设置为序列中每一文件的内容。
*    with_fileglob 	        取文件名通配模式。item 设置为控制节点上目录中符合该模式的各个文件，并且按顺序、非递归迭代。
*    with_sequence 	        以数字递增顺序生成一个项目序列。可以取 start 和 end 参数，参数具有十进制、八进制或十六进制整数值。
*    with_random_choices 	取一个列表。item 设置为列表项中随机的一项。


###### 条件

1. Ansible 条件运算符

*    运算符 	                示例
*    等于 	                "{{ max_memory }} == 512"
*    小于 	                "{{ min_memory }} < 128"
*    大于 	                "{{ min_memory }} > 256"
*    小于等于 	                "{{ min_memory }} <= 256"
*    大于等于 	                "{{ min_memory }} >= 512"
*    不等于 	                "{{ min_memory }} != 512"
*    变量存在 	                "{{ min_memory }} is defined"
*    变量不存在 	            "{{ min_memory }} is not defined"
*    变量设为 1、True 或 yes 	"{{ available_memory }}"
*    变量设为 0、False 或 no 	"not {{ available_memory }}"
*    值存在于某一变量或数组 	    "{{ users }} in users["db_admins"]"

###### Ansible when 语句

注意 when 语句的位置。由于 when 语句不是模块变量，它必须通过缩进到任务的最高级别，放置在模块的“外面”。when 语句不需要处于任务的顶部。“自上而下”排序断裂对 Ansible 来说正常。 

```
- name: Create the database admin
  user:
    name: db_admin
  when: inventory_hostname in groups["databases"]
```

1. 多个条件： 一个 when 语句可用于评估多个值。为此，可以使用 and 和 or 关键字组合条件，或者使用括号分组条件。

*    ansible_kernel == 3.10.0-327.el7.x86_64 and inventory_hostname in groups['staging']
*    ansible_distribution == "RedHat" or ansible_distribution == "Fedora"
*    (ansible_distribution == "RedHat" and ansible_distribution_major_version == 7) or (ansible_distribution == "Fedora" and ansible_distribution_major_version == 23)

2. 使用布尔运算

*    True 或 Yes 或 1
*    False 或 No 或 0

```
---
- hosts: all
  vars:
    my_service: True

  tasks:
    - name: Installs a package
      yum:
        name: httpd
      when: my_service
```

### 实施处理程序

###### 处理程序

有时候，在任务确实更改系统时，可能需要运行进一步的任务。例如，更改服务配置文件时可能要求重新加载该服务以便使更改的配置生效。

处理程序是响应由其他任务触发的通知的任务。每一处理程序具有全局唯一的名称，在 playbook 中任务块的末尾触发。如果没有任务通过其名称通知程序程序，它就不会运行。如果有一个或多个任务通知处理程序，它会在 play 中所有其他任务完成后仅运行一次。因为处理程序就是任务，所以管理员可以在处理程序中使用适用于任何其他任务的模块。通常而言，处理程序被用于重新引导主机和重新启动服务。

处理程序可视为非活动任务，只有在使用 notify 语句显式调用时才会被触发。在下列代码片段中，只有配置文件更新并且通知了该任务时，restart_apache 任务才会重新启动 Apache 服务器：

```
tasks:
  - name: copy demo.example.conf configuration template
    copy:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart_apache

handlers:
  - name: restart_apache
    service:
      name: httpd
      state: restarted
```

###### 使用处理程序

如 Ansible 文档中所述，使用处理程序时需要牢记几个重要事项：

*    处理程序始终按照它们在 play 的 handlers 部分中编写的顺序来运行，而不是按照特定任务中 notify 语句列出的顺序。
*    处理程序在相关 play 中的所有其他任务完成后运行。playbook 的 tasks: 部分中某一任务调用的处理程序，将等到 tasks: 下的所有任务都已处理后才会运行。
*    处理程序名称存在于全局命名空间中。如果两个处理程序被错误地给予相同的名称，则仅会运行一个。
*    在 include 内定义的处理程序无法获得通知。
*    即使有多个任务通知处理程序，该处理程序依然仅运行一次。如果没有任务通知处理程序，它就不会运行。
*    如果包含 notify 的任务没有执行（例如，软件包已经安装），则处理程序不会获得通知。处理程序将被跳过，直到有其他任务通知它。只有相关任务获得了 CHANGED 状态，Ansible 才会通知处理程序。


### 实施标记

###### 标记 Ansible 资源

对于篇幅较长的 playbook，如果能够运行 playbook 中任务的子集会很有帮助。为此，可以将 标记设置到特定的资源上作为文本标签。标记资源仅需要使用 tags 关键字，再加上要应用的标记列表。为 play 添加标记后，可将 --tags 选项用于 ansible-playbook，使 playbook 过滤为仅执行加有特定标记的 play。标记可用于下列资源：

1. 在 playbook 中，每一个任务都可使用 tags 关键字添加标记：

```
tasks:
  - name: Install {{ item }} package
    yum: name={{ item }} state=installed
    with_items:
      - postfix
      - mariadb-server
    tags:
      - packages
```

2. 当任务文件包含在 playbook 中时，该任务可以添加标记，允许管理员为 include 语句设置全局标记, 为某一角色或 include 语句添加标记时，它们定义的所有任务也都会加上标记。

```
- include: common.yml
  tags: [webproxy, webserver]
```

###### 管理标记的资源

当标记应用至资源后，可以将 ansible-playbook 命令与 --tags 或 --skip-tags 参数搭配使用，以执行带标记的资源或防止带标记的资源被包含在 play 中。以下 playbook 含有两项任务；第一项任务带有 production 标记，另一项任务则未关联任何标记:

```
---
- hosts: all
  tasks:
    - name: My tagged task
      yum:
        name: httpd
        state: latest
      tags: production

    - name: Installs postfix
      yum:
        name: postfix
        state: latest

如需运行第一项任务，可以使用 --tags 参数
$ ansible-playbook main.yml --tags 'production'
TASK [My tagged task] ***************************************************
ok: [demo.example.com]
... Output omitted ...

因为指定了 --tags 选项，playbook 仅运行一个任务，即带有 production 标记的任务。要跳过第一个任务而仅运行第二个任务，可使用 --skip-tags 选项
$ ansible-playbook main.yml --skip-tags 'production'
... Output omitted ...

TASK [Installs postfix] **************************************************
ok: [demo.example.com]
```

###### 特殊标记

Ansible 拥有一个可在 playbook 中分配的特殊标记：always。此标记使得任务始终被执行，即便是使用了 --skip-tags 选项，除非通过 --skip-tags always 选项来明确跳过。

可以在命令行中通过 --tags 选项使用三种特殊标记：

*    tagged 关键字用于运行任何带标记的资源。
*    untagged 关键字的作用与 tagged 关键字相反，从 play 中排除所有带标记的资源。
*    all 关键字允许管理员包含 play 中的所有任务。这是命令行的默认行为。


### 处理错误

###### play 中的错误

1. 忽略失败的任务：默认情况下，如果某一任务失败，play 将中止；不过，可以通过跳过失败的任务来覆盖此行为。为此，需要在任务中使用 ignore_errors 关键字。下列代码片段演示了如何使用 ignore_errors 即使在任务失败时也继续执行 playbook。例如，如果 notapkg 软件包不存在，yum 模块将失败，但若将 ignore_errors 设为 yes，则执行将继续。

```
- yum:
    name: notapkg
    state: latest
  ignore_errors: yes
```

2. 强制执行处理程序：默认情况下，如果通知处理程序的任务失败，则该处理程序也会被跳过。管理员可以在任务中使用 force_handlers 关键字来覆盖此行为。这会强制调用该处理程序，即使相关的任务失败。下列代码片段演示了在任务中使用 force_handlers 关键字，而从在任务失败时强制执行相应的处理程序：

```
---
- hosts: all
  force_handlers: yes
  tasks:
    - yum:
        name: notapkg
        state: latest
      notify: restart_database

  handlers:
    - name: restart_database
      service:
        name: mariadb
        state: restarted
```

3. 覆盖 failed 状态：任务本身可以成功，但管理员可能希望基于特定的标准将任务标记为失败。为此，可以将 failed_when 关键字用于任务。这通常用于执行远程命令并且捕获变量中的输出的模块。例如，管理员可以运行输出错误消息的脚本，并使用该消息定义任务的失败状态。下列代码片段演示了如何在任务中使用 failed_when 关键字：

```
tasks:
  - shell:
      cmd: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
```

4. 覆盖 changed 状态：当任务更新受管主机时，它将获取 changed 状态；但是，如果任务不在受管主机上执行任何更改，处理程序会被跳过。changed_when 关键字可用于覆盖触发 changed 状态的默认行为。例如，如果管理员希望在每次运行 playbook 时重新启动某一服务，可以将 changed_when 关键字添加到相应任务中。下列代码片段演示了如果通过强制 changed 状态来每次都触发处理程序：

```
tasks:
  - shell:
      cmd: /usr/local/bin/upgrade-database
    register: command_result
    changed_when: "'Success' in command_result.stdout"
    notify:
      - restart_database

handlers:
  - name: restart_database
     service:
       name: mariadb
       state: restarted
```

###### Ansible 块和错误处理

在 playbook 中， 块是囊括了任务的子句。块允许对任务进行逻辑分组，并可用于控制任务的执行方式。例如，管理员可以定义一组主要任务和一组附加任务，附加任务仅在第一组失败时执行。为此，可利用三个关键字在 playbook 中使用块：

*    block：定义要运行的主要任务
*    rescue：定义将在 block 子句中定义的任务失败时运行的任务
*    always：定义始终都独立运行的任务，不论 block 和 rescue 子句中定义的任务是成功还是失败。

下例演示了如何在 playbook 中实施块。即使 block 子句中定义的任务失败，rescue 和 always 子句中定义的任务都将执行：

```
tasks:
  - block:
      - shell:
          cmd: /usr/local/lib/upgrade-database

    rescue:
      - shell:
          cmd: /usr/local/lib/create-users

    always:
      - service:
          name: mariadb
          state: restarted
```

```
---
- hosts: mailservers
  vars:
    maildir_path: /home/john/Maildir
    maildir: /home/student/Maildir
    mail_package: postfix
    mail_service: postfix

  tasks:
    - block:
      - name: Create {{ maildir_path }}
        copy:
          src: "{{ maildir }}"
          dest: "{{ maildir_path }}"
          mode: 0755

      rescue:
      - name: Install mail packages
        yum:
          name: "{{ item }}"
          state: latest
        with_items:
          - "{{ mail_package }}"
          - dovecot

      always:
      - name: Start mail services
        service:
          name: "{{ item }}"
          state: started
        with_items:
          - "{{ mail_service }}"
          - dovecot
        register: command_result

    - debug:
        var: command_result
```

### Summary

*    循环可用于迭代一组值。它们可以是按顺序或随机项目的列表、文件、自动生成的数字序列，或其他事物
*    条件可用于仅在符合特定条件时执行任务或 play
*    条件可以通过各种运算符进行测试，如比较、数学运算符和布尔值等
*    处理程序是特殊的任务，它们在被其他任务通知时在 play 的末尾执行
*    标记用于标示特定的任务，从而根据任务所具有的标记来跳过或执行任务
*    可以对任务进行配置，从而通过忽略任务失败、在任务失败时强制调用处理程序来处理错误，即使任务成功时也将它标记为失败，或者覆盖导致任务被标记为已更改的行为
*    块可同于将任务分组为单元，并且根据块中的任务是否都已成功来执行其他任务
